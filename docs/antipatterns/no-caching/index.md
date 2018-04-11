---
title: Antimuster „Kein Caching“
description: Durch wiederholtes Abrufen der gleichen Daten können Leistung und Skalierbarkeit verringert werden.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 8a2bc3b473a30536cc1bef9e1dcad87acb46c4a9
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/05/2018
---
# <a name="no-caching-antipattern"></a>Antimuster „Kein Caching“

In einer Cloudanwendung, die viele gleichzeitige Anforderungen verarbeitet, können durch wiederholtes Abrufen der gleichen Daten Leistung und Skalierbarkeit sinken. 

## <a name="problem-description"></a>Problembeschreibung

Wenn die Daten nicht zwischengespeichert werden, kann eine Reihe unerwünschter Verhaltensweisen auftreten, wie z.B. folgende:

- Wiederholtes Abrufen der gleichen Informationen aus einer Ressource, deren Zugriff viel E/A-Overhead oder Latenz kostet
- Wiederholtes Erstellen der gleichen Objekte oder Datenstrukturen für mehrere Anforderungen
- Übermäßig viele Abrufe für einen Remotedienst, für den ein Dienstkontingent gilt und der Clients ab einem bestimmten Grenzwert drosselt

Diese Probleme können zu langen Antwortzeiten, vermehrten Konflikten im Datenspeicher und unzureichender Skalierbarkeit führen.

Das folgende Beispiel verwendet Entity Framework, um eine Verbindung mit einer Datenbank herzustellen. Jede Clientanforderung führt zu einem Aufruf in der Datenbank, selbst wenn mehrere Anforderungen exakt die gleichen Daten abrufen. Die Kosten wiederholter Anforderungen hinsichtlich E/A-Overhead und Datenzugriffsgebühren können sich schnell summieren.

```csharp
public class PersonRepository : IPersonRepository
{
    public async Task<Person> GetAsync(int id)
    {
        using (var context = new AdventureWorksContext())
        {
            return await context.People
                .Where(p => p.Id == id)
                .FirstOrDefaultAsync()
                .ConfigureAwait(false);
        }
    }
}
```

Das vollständige Beispiel finden Sie [hier][sample-app].

Dieses Antimuster tritt üblicherweise aus folgenden Gründen auf:

- Wenn Sie keinen Cache verwenden, ist die Implementierung einfacher, und das Verfahren funktioniert gut bei geringen Lasten. Die Verwendung eines Caches macht den Code komplizierter. 
- Die Vor- und Nachteile eines Caches sind nicht klar.
- Es gibt Bedenken hinsichtlich des Overheads für die Sicherstellung der Genauigkeit und Aktualität der zwischengespeicherten Daten.
- Eine Anwendung wurde aus einem lokalen System migriert, in dem die Netzwerklatenz kein Problem war und das System auf teurer Hochleistungshardware ausgeführt wurde, daher wurde im ursprünglichen Entwurf kein Caching berücksichtigt.
- Entwicklern ist nicht bewusst, dass Caching in einem bestimmten Szenario eine Möglichkeit sein kann. Beispielsweise denken Entwickler beim Implementieren einer Web-API möglicherweise nicht daran, ETags zu verwenden.

## <a name="how-to-fix-the-problem"></a>Beheben des Problems

Die beliebteste Cachingstrategie ist die *bedarfsbasierte* oder *cachefremde* Strategie.

- Während eines Lesevorgangs versucht die Anwendung, die Daten aus dem Cache zu lesen. Wenn sich die Daten nicht im Cache befinden, ruft die Anwendung sie aus der Datenquelle ab und fügt sie dem Cache hinzu.
- Bei einem Schreibvorgang schreibt die Anwendung die Änderung direkt in die Datenquelle und entfernt den alten Wert aus dem Cache. Wenn die Daten das nächste Mal benötigt werden, werden sie abgerufen und dem Cache hinzugefügt.

Diese Vorgehensweise eignet sich für Daten, die sich häufig ändern. Hier sehen Sie das vorherige Beispiel, das aktualisiert wurde und jetzt das [cachefremde][cache-aside] Muster verwendet.  

```csharp
public class CachedPersonRepository : IPersonRepository
{
    private readonly PersonRepository _innerRepository;

    public CachedPersonRepository(PersonRepository innerRepository)
    {
        _innerRepository = innerRepository;
    }

    public async Task<Person> GetAsync(int id)
    {
        return await CacheService.GetAsync<Person>("p:" + id, () => _innerRepository.GetAsync(id)).ConfigureAwait(false);
    }
}

public class CacheService
{
    private static ConnectionMultiplexer _connection;

    public static async Task<T> GetAsync<T>(string key, Func<Task<T>> loadCache, double expirationTimeInMinutes)
    {
        IDatabase cache = Connection.GetDatabase();
        T value = await GetAsync<T>(cache, key).ConfigureAwait(false);
        if (value == null)
        {
            // Value was not found in the cache. Call the lambda to get the value from the database.
            value = await loadCache().ConfigureAwait(false);
            if (value != null)
            {
                // Add the value to the cache.
                await SetAsync(cache, key, value, expirationTimeInMinutes).ConfigureAwait(false);
            }
        }
        return value;
    }
}
```

Beachten Sie, dass die `GetAsync`-Methode jetzt die `CacheService`-Klasse aufruft, anstatt die Datenbank direkt aufzurufen. Die `CacheService`-Klasse versucht zuerst, das Element aus Azure Redis Cache abzurufen. Wenn der Wert in Redis Cache nicht gefunden wird, ruft `CacheService` eine Lambdafunktion auf, die vom Aufrufer an sie übergeben wurde. Die Lambdafunktion ist dafür zuständig, die Daten aus der Datenbank abzurufen. Diese Implementierung entkoppelt das Repository von der jeweiligen Cachinglösung und den `CacheService` von der Datenbank. 

## <a name="considerations"></a>Überlegungen

- Wenn der Cache nicht verfügbar ist – möglicherweise aufgrund eines vorübergehenden Fehlers – geben Sie keinen Fehler an den Client zurück. Rufen Sie die Daten stattdessen aus der ursprünglichen Datenquelle ab. Denken Sie jedoch daran, dass während der Wiederherstellung des Caches der ursprüngliche Datenspeicher mit Anforderungen überschwemmt werden könnte, sodass Zeitüberschreitungen und Verbindungsfehler die Folge wären. (Dies ist schließlich einer der Hauptgründe, aus denen ein Cache überhaupt verwendet wird.) Verwenden Sie eine Technik wie z.B. das [Sicherungsmuster][circuit-breaker], um eine übermäßige Beanspruchung der Datenquelle zu vermeiden.

- Anwendungen, die nicht statische Daten zwischenspeichern, sollte für die Unterstützung der letztlichen Konsistenz entworfen werden.

- Bei Web-APIs können Sie das clientseitige Cachen unterstützen, indem Sie einen Cache-Control-Header in die Anforderungs- und Antwortnachricht einschließen und ETags zum Identifizieren der Version von Objekten verwenden. Weitere Informationen finden Sie unter [API-Implementierung][api-implementation].

- Sie müssen keine vollständigen Entitäten zwischenspeichern. Wenn der größte Teil einer Entität statisch ist und sich nur ein kleiner Teil häufig ändert, speichern Sie die statischen Elemente zwischen, und rufen Sie die dynamischen Elemente aus der Datenquelle ab. Mit dieser Vorgehensweise können Sie die Menge an E/A-Vorgängen reduzieren, die für die Datenquelle ausgeführt werden.

- In einigen Fällen, wenn flüchtige Daten kurzlebig sind, kann es nützlich sein, diese zwischenzuspeichern. Nehmen wir als Beispiel ein Gerät, das kontinuierlich Statusupdates sendet. Es kann sinnvoll sein, diese Informationen bei Eingang zwischenzuspeichern und gar nicht erst in den dauerhaften Speicher zu schreiben.  

- Um zu verhindern, dass Daten veralten, unterstützen viele Cachinglösungen konfigurierbare Ablaufzeiträume, sodass diese Daten nach dem angegebenen Zeitraum automatisch aus dem Cache entfernt werden. Möglicherweise müssen Sie die Ablaufzeit für Ihr Szenario anpassen. Daten, die in hohem Maß statisch sind, können länger im Cache verbleiben als flüchtige Daten, die schnell veralten.

- Wenn die Cachinglösung keinen integrierten Ablauf bietet, müssen Sie möglicherweise einen Hintergrundprozess implementieren, der den Cache gelegentlich leert, damit er nicht ins Unendliche anwächst. 

- Sie können nicht nur Daten aus einer externen Datenquelle zwischenspeichern, Sie können den Cache auch zum Speichern der Ergebnisse komplexer Berechnungen verwenden. Bevor Sie den Cache jedoch zu diesem Zweck nutzen, instrumentieren Sie die Anwendung, um zu ermitteln, ob sie tatsächlich CPU-gebunden ist.

- Es kann nützlich sein, den Cache vorzubereiten, wenn die Anwendung startet. Füllen Sie den Cache mit den Daten auf, die am wahrscheinlichsten verwendet werden.

- Schließen Sie immer eine Instrumentierung ein, die Cachetreffer und Cachefehler erkennt. Verwenden Sie diese Informationen, um Cachingrichtlinien anzupassen, beispielsweise dahingehend, welche Daten zwischengespeichert werden und wie lange Daten im Cache aufbewahrt werden, bevor sie ablaufen.

- Wenn unzureichendes Caching zu Engpässen führt, kann durch vermehrtes Caching die Menge an Anforderungen so weit steigen, dass das Web-Front-End überlastet wird. Clients erhalten möglicherweise HTTP 503-Fehler (Dienst nicht verfügbar). Diese sind ein Hinweis darauf, dass Sie das Front-End horizontal hochskalieren sollten.

## <a name="how-to-detect-the-problem"></a>Erkennen des Problems

Sie können die folgenden Schritte ausführen, um herauszufinden, ob ein unzureichendes Caching Leistungsprobleme verursacht:

1. Überprüfen Sie den Anwendungsentwurf. Erfassen Sie den Bestand aller Datenspeicher, die von der Anwendung verwendet werden. Ermitteln Sie für jeden Datenspeicher, ob die Anwendung einen Cache verwendet. Sofern möglich, ermitteln Sie, wie häufig sich die Daten ändern. Gute Anfangskandidaten für das Caching sind Daten, die sich nur langsam ändern, und statische Verweisdaten, die häufig gelesen werden. 

2. Instrumentieren Sie die Anwendung, und überwachen Sie das Livesystem, um herauszufinden, wie häufig die Anwendung Daten abruft oder Informationen berechnet.

3. Erstellen Sie ein Profil der Anwendung in einer Testumgebung, um Low-Level-Metriken zum Overhead zu erfassen, der mit Datenzugriffsvorgängen oder anderen häufig ausgeführten Berechnungen in Verbindung steht.

4. Führen Sie einen Auslastungstest in einer Testumgebung durch, um zu ermitteln, wie das System unter normaler und schwerer Workload reagiert. Der Auslastungstest sollte das Datenzugriffsmuster simulieren, das in der Produktionsumgebung mit realistischen Workloads beobachtet wurde. 

5. Untersuchen Sie die Datenzugriffsstatistiken für die zugrunde liegenden Datenspeicher, und untersuchen Sie, wie häufig die gleichen Datenanforderungen wiederholt werden. 


## <a name="example-diagnosis"></a>Beispieldiagnose

In den folgenden Abschnitten werden diese Schritte auf die zuvor beschriebene Beispielanwendung angewendet.

### <a name="instrument-the-application-and-monitor-the-live-system"></a>Instrumentieren der Anwendung und Überwachen des Livesystems

Instrumentieren Sie die Anwendung, und überwachen Sie sie, um Informationen zu den spezifischen Anforderungen zu erhalten, die Benutzer während der Produktionsphase der Anwendung senden. 

Die folgende Abbildung zeigt die Überwachung der Daten, die von [New Relic][NewRelic] während eines Auslastungstests erfasst wurden. In diesem Fall ist `Person/GetAsync` der einzige ausgeführte HTTP GET-Vorgang. In einer Liveproduktionsumgebung kann die Kenntnis der Häufigkeit, mit der jede Anforderung ausgeführt wird, Ihnen Einblicke darin geben, welche Ressourcen zwischengespeichert werden sollten.

![New Relic mit Serveranforderungen aus der CachingDemo-Anwendung][NewRelic-server-requests]

Wenn Sie eine detailliertere Analyse benötigen, können Sie einen Profiler verwenden, um in einer Testumgebung (nicht im Produktionssystem) Leistungsdaten auf niedriger Ebene zu erfassen. Sehen Sie sich Metriken wie E/A-Anforderungsraten, Arbeitsspeichernutzung und CPU-Auslastung an. Diese Metriken zeigen möglicherweise eine große Anzahl von Anforderungen im Datenspeicher oder Dienst oder eine wiederholte Verarbeitung, die die gleiche Berechnung ausführt. 

### <a name="load-test-the-application"></a>Auslastungstest der Anwendung

Das folgende Diagramm zeigt die Ergebnisse des Auslastungstests der Beispielanwendung. Der Auslastungstest simuliert eine Schrittauslastung von bis zu 800 Benutzern, die eine typische Reihenfolge von Vorgängen ausführen. 

![Leistungstestergebnisse für das Szenario ohne Cache][Performance-Load-Test-Results-Uncached]

Die Anzahl von pro Sekunden ausgeführten erfolgreichen Tests stabilisiert sich auf einem Niveau, und dadurch werden weitere Anforderungen verlangsamt. Die durchschnittliche Testzeit steigt stetig mit der Workload. Die Antwortzeit sinkt, sobald die Benutzerauslastung einen Spitzenwert erreicht.

### <a name="examine-data-access-statistics"></a>Untersuchen der Datenzugriffsstatistiken

Datenzugriffsstatistiken und andere von einem Datenspeicher bereitgestellte Informationen, z.B. dazu, welche Abfragen am häufigsten wiederholt werden, können sehr nützlich sein. In Microsoft SQL Server bietet die `sys.dm_exec_query_stats`-Verwaltungssicht statistische Informationen zu kürzlich ausgeführten Abfragen. Der Text für jede Abfrage ist in der `sys.dm_exec-query_plan`-Sicht verfügbar. Sie können ein Tool wie z.B. SQL Server Management Studio verwenden, um die folgende SQL-Abfrage auszuführen und zu ermitteln, wie häufig Abfragen ausgeführt werden.

```SQL
SELECT UseCounts, Text, Query_Plan
FROM sys.dm_exec_cached_plans
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
CROSS APPLY sys.dm_exec_query_plan(plan_handle)
```

Die `UseCount`-Spalte in den Ergebnissen gibt an, wie häufig jede Abfrage ausgeführt wird. Die folgende Abbildung zeigt, dass die dritte Abfrage über 250.000-mal ausgeführt wurde – deutlich mehr als jede andere Abfrage.

![Ergebnisse der Abfragen der dynamischen Verwaltungssichten in SQL Server Management Server][Dynamic-Management-Views]

Dies ist die SQL-Abfrage, die so viele Datenbankanforderungen verursacht: 

```SQL
(@p__linq__0 int)SELECT TOP (2)
[Extent1].[BusinessEntityId] AS [BusinessEntityId],
[Extent1].[FirstName] AS [FirstName],
[Extent1].[LastName] AS [LastName]
FROM [Person].[Person] AS [Extent1]
WHERE [Extent1].[BusinessEntityId] = @p__linq__0
```

Dies ist die Abfrage, die Entity Framework in der oben gezeigten `GetByIdAsync`-Methode generiert.

### <a name="implement-the-solution-and-verify-the-result"></a>Implementieren der Lösung und Überprüfen des Ergebnisses

Nachdem Sie einen Cache implementiert haben, wiederholen Sie die Auslastungstests, und vergleichen Sie die Ergebnisse mit den vorherigen Auslastungstests ohne Cache. Dies sind die Ergebnisse des Auslastungstests nach dem Hinzufügen eines Caches zur Beispielanwendung.

![Leistungstestergebnisse für das Szenario mit Cache][Performance-Load-Test-Results-Cached]

Der Menge an erfolgreichen Tests stabilisiert sich weiterhin auf einem Niveau, jedoch bei einer höheren Benutzerauslastung. Die Anforderungsrate bei dieser Auslastung ist deutlich höher als vorher. Die durchschnittliche Testzeit steigt weiterhin mit der Auslastung, aber die maximale Antwortzeit beträgt 0,05 ms, im Vergleich zu 1 ms &mdash;eine 20&times; fache Verbesserung. 

## <a name="related-resources"></a>Zugehörige Ressourcen

- [API implementation best practices][api-implementation] (Bewährte Methoden für die API-Implementierung)
- [Cache-Aside Pattern][cache-aside-pattern] (Cachefremdes Muster)
- [Caching best practices][caching-guidance] (Bewährte Methoden für das Caching)
- [Circuit Breaker pattern][circuit-breaker] (Schutzschaltermuster)

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/NoCaching
[cache-aside-pattern]: /azure/architecture/patterns/cache-aside
[caching-guidance]: ../../best-practices/caching.md
[circuit-breaker]: ../../patterns/circuit-breaker.md
[api-implementation]: ../../best-practices/api-implementation.md#optimizing-client-side-data-access
[NewRelic]: http://newrelic.com/azure
[NewRelic-server-requests]: _images/New-Relic.jpg
[Performance-Load-Test-Results-Uncached]:_images/InitialLoadTestResults.jpg
[Dynamic-Management-Views]: _images/SQLServerManagementStudio.jpg
[Performance-Load-Test-Results-Cached]: _images/CachedLoadTestResults.jpg