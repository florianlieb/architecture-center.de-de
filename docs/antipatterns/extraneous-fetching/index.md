---
title: "Antimuster „Extraneous Fetching“ (Irrelevante Abrufe)"
description: "Das Abrufen von mehr Daten, als für einen Geschäftsvorgang erforderlich sind, kann zu unnötigem E/A-Mehraufwand und einer Reduzierung der Reaktionsfähigkeit führen."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 8a808dce62a1c80c126b7b1df536f74c46726ea1
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="extraneous-fetching-antipattern"></a>Antimuster „Extraneous Fetching“ (Irrelevante Abrufe)

Das Abrufen von mehr Daten, als für einen Geschäftsvorgang erforderlich sind, kann zu unnötigem E/A-Mehraufwand und einer Reduzierung der Reaktionsfähigkeit führen. 

## <a name="problem-description"></a>Problembeschreibung

Dieses Antimuster kann auftreten, wenn die Anwendung versucht, die Anzahl von E/A-Anforderungen gering zu halten, indem alle *ggf.* benötigten Daten abgerufen werden. Dies ist häufig das Ergebnis einer Überkompensation für das Antimuster [Chatty I/O][chatty-io] (Sehr hohe Zahl von E/A-Vorgängen). Es kann beispielsweise sein, dass eine Anwendung die Details für jedes Produkt einer Datenbank abruft. Der Benutzer benötigt aber ggf. nur einen Teil dieser Details (da einige für Kunden nicht relevant sind) und muss vermutlich nicht *alle* Produkte auf einmal sehen können. Auch wenn sich der Benutzer den gesamten Katalog ansieht, ist es sinnvoll, die Ergebnisse nach Seiten aufzuteilen und beispielsweise jeweils nur 20 anzuzeigen.

Ursachen dieses Problems können eine schlechte Programmierung oder ein unzureichender Entwurf sein. Im folgenden Code wird beispielsweise Entity Framework verwendet, um die gesamten Details zu jedem Produkt abzurufen. Anschließend werden die Ergebnisse gefiltert, um nur eine Teilmenge der Felder zurückzugeben, und die restlichen Daten werden verworfen. Das vollständige Codebeispiel finden Sie [hier][sample-app].

```csharp
public async Task<IHttpActionResult> GetAllFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Execute the query. This happens at the database.
        var products = await context.Products.ToListAsync();

        // Project fields from the query results. This happens in application memory.
        var result = products.Select(p => new ProductInfo { Id = p.ProductId, Name = p.Name });
        return Ok(result);
    }
}
```

Im nächsten Beispiel ruft die Anwendung Daten zum Durchführen eines Aggregationsvorgangs ab, der stattdessen von der Datenbank übernommen werden könnte. Die Anwendung berechnet den Gesamtumsatz, indem alle Datensätze für alle bearbeiteten Bestellungen abgerufen werden und anschließend die Summe für diese Datensätze gebildet wird. Das vollständige Codebeispiel finden Sie [hier][sample-app].

```csharp
public async Task<IHttpActionResult> AggregateOnClientAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Fetch all order totals from the database.
        var orderAmounts = await context.SalesOrderHeaders.Select(soh => soh.TotalDue).ToListAsync();

        // Sum the order totals in memory.
        var total = orderAmounts.Sum();
        return Ok(total);
    }
}
```

Im nächsten Beispiel ist ein kleineres Problem dargestellt, das durch die Art und Weise entsteht, wie LINQ to Entities von Entity Framework verwendet wird. 

```csharp
var query = from p in context.Products.AsEnumerable()
            where p.SellStartDate < DateTime.Now.AddDays(-7) // AddDays cannot be mapped by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

Die Anwendung versucht, Produkte mit einem Verkaufsstartdatum (`SellStartDate`) zu ermitteln, das länger als eine Woche zurückliegt. In den meisten Fällen übersetzt LINQ to Entities eine `where`-Klausel in eine SQL-Anweisung, die von der Datenbank ausgeführt wird. Hier kann LINQ to Entities die `AddDays`-Methode aber nicht SQL zuordnen. Stattdessen wird jede Zeile der Tabelle `Product` zurückgegeben, und die Ergebnisse werden im Arbeitsspeicher gefiltert. 

Der Aufruf von `AsEnumerable` ist ein Hinweis darauf, dass ein Problem vorliegt. Bei dieser Methode werden die Ergebnisse in eine `IEnumerable`-Schnittstelle konvertiert. `IEnumerable` unterstützt zwar die Filterung, aber der Filtervorgang wird aufseiten des *Clients* durchgeführt, nicht auf der Datenbankseite. Standardmäßig nutzt LINQ to Entities die `IQueryable`-Schnittstelle, mit der die Zuständigkeit für die Filterung an die Datenquelle übergeben wird. 

## <a name="how-to-fix-the-problem"></a>Beheben des Problems

Vermeiden Sie es, große Datenvolumen abzurufen, die schnell veraltet sind oder verworfen werden. Rufen Sie nur die Daten ab, die für den durchzuführenden Vorgang benötigt werden. 

Anstatt jede Spalte aus einer Tabelle abzurufen und dann zu filtern, ist es ratsam, die benötigten Spalten der Datenbank auszuwählen.

```csharp
public async Task<IHttpActionResult> GetRequiredFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Project fields as part of the query itself
        var result = await context.Products
            .Select(p => new ProductInfo {Id = p.ProductId, Name = p.Name})
            .ToListAsync();
        return Ok(result);
    }
}
```

Führen Sie außerdem die Aggregation in der Datenbank und nicht im Anwendungsspeicher durch.

```csharp
public async Task<IHttpActionResult> AggregateOnDatabaseAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Sum the order totals as part of the database query.
        var total = await context.SalesOrderHeaders.SumAsync(soh => soh.TotalDue);
        return Ok(total);
    }
}
```

Stellen Sie bei Verwendung von Entity Framework sicher, dass LINQ-Abfragen mit der `IQueryable`-Schnittstelle und nicht mit `IEnumerable` aufgelöst werden. Unter Umständen müssen Sie die Abfrage so anpassen, dass nur Funktionen verwendet werden, die der Datenquelle zugeordnet werden können. Das vorherige Beispiel kann umgestaltet werden, um die `AddDays`-Methode aus der Abfrage zu entfernen, damit der Filtervorgang von der Datenbank durchgeführt werden kann.

```csharp
DateTime dateSince = DateTime.Now.AddDays(-7); // AddDays has been factored out. 
var query = from p in context.Products
            where p.SellStartDate < dateSince // This criterion can be passed to the database by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

## <a name="considerations"></a>Überlegungen

- In einigen Fällen können Sie die Leistung verbessern, indem Sie die Daten horizontal partitionieren. Wenn unterschiedliche Vorgänge auf verschiedene Attribute der Daten zugreifen, kann eine horizontale Partitionierung bewirken, dass weniger Konflikte auftreten. Da die meisten Vorgänge häufig nur für eine kleinere Teilmenge der Daten ausgeführt werden, lässt sich die Leistung verbessern, indem diese Last verteilt wird. Informationen hierzu finden Sie unter [Datenpartitionierung][data-partitioning].

- Für Vorgänge, mit denen uneingeschränkte Abfragen unterstützt werden, sollten Sie die Paginierung implementieren und jeweils nur eine begrenzte Anzahl von Entitäten abrufen. Wenn sich ein Kunde beispielsweise einen Produktkatalog ansieht, können Sie jeweils eine Seite mit Ergebnissen anzeigen.

- Nutzen Sie nach Möglichkeit Features, die in den Datenspeicher integriert sind. SQL-Datenbanken enthalten in der Regel beispielsweise Aggregatfunktionen. 

- Wenn Sie einen Datenspeicher verwenden, der eine bestimmte Funktion nicht unterstützt, z.B. die Aggregration, können Sie das berechnete Ergebnis an einem anderen Ort speichern und den Wert aktualisieren, wenn Datensätze hinzugefügt oder aktualisiert werden. Die Anwendung muss den Wert dann nicht jedes Mal neu berechnen, wenn er benötigt wird.

- Wenn Sie sehen, dass Anforderungen eine große Anzahl von Feldern abrufen, sollten Sie den Quellcode untersuchen und ermitteln, ob alle Felder auch wirklich erforderlich sind. Diese Anforderungen können auch eine Folge einer schlecht entworfenen `SELECT *`-Abfrage sein. 

- Ebenso können Anforderungen, bei denen eine große Zahl von Entitäten abgerufen wird, ein Zeichen dafür sein, dass Daten von der Anwendung nicht richtig gefiltert werden. Vergewissern Sie sich, dass alle Entitäten auch wirklich benötigt werden. Nutzen Sie nach Möglichkeit die Filterung auf Datenbankseite, indem Sie beispielsweise `WHERE`-Klauseln in SQL verwenden. 

- Das Auslagern der Verarbeitung in die Datenbank ist nicht immer die beste Vorgehensweise. Verwenden Sie diese Strategie nur, wenn die Datenbank dafür entworfen bzw. optimiert wurde. Die meisten Datenbanksysteme sind speziell im Hinblick auf bestimmte Funktionen optimiert, aber nicht dafür ausgelegt, als Anwendungsmodule für allgemeine Zwecke zu fungieren. Weitere Informationen finden Sie unter [Antimuster „Busy Database“ (Ausgelastete Datenbank)][BusyDatabase].


## <a name="how-to-detect-the-problem"></a>Erkennen des Problems

Zu den Symptomen des „Extraneous Fetching“ (Irrelevante Abrufe) gehören lange Wartezeiten und ein niedriger Durchsatz. Wenn die Daten aus einem Datenspeicher abgerufen werden, steigt auch die Konfliktwahrscheinlichkeit. Endbenutzer berichten vermutlich von längeren Antwortzeiten oder Fehlern aufgrund von Diensten, bei denen ein Timeout auftritt. In diesen Fällen können Fehler vom Typ „HTTP 500 (Interner Server)“ oder „HTTP 503 (Dienst nicht verfügbar)“ zurückgegeben werden. Überprüfen Sie die Ereignisprotokolle für den Webserver. Sie enthalten wahrscheinlich ausführlichere Informationen zu den Ursachen und Umständen der Fehler.

Die Symptome dieses Antimusters und einige ermittelte Telemetriedaten können den Daten des [Antimusters „Monolithic Persistence“ (Monolithische Persistenz)][MonolithicPersistence] stark ähneln. 

Sie können die folgenden Schritte ausführen, um die Ursache zu ermitteln:

1. Identifizieren Sie langsame Workloads oder Transaktionen, indem Sie Auslastungstests, eine Prozessüberwachung und andere Verfahren zur Erfassung von Instrumentierungsdaten durchführen.
2. Beobachten Sie alle Verhaltensmuster des Systems. Bestehen bestimmte Beschränkungen in Bezug auf die Transaktionen pro Sekunde oder das Benutzervolumen?
3. Korrelieren Sie die Instanzen von langsamen Workloads mit Verhaltensmustern.
4. Identifizieren Sie die verwendeten Datenspeicher. Führen Sie für jede Datenquelle die Erfassung von Telemetriedaten auf niedriger Ebene aus, um das Verhalten von Vorgängen beobachten zu können.
6. Identifizieren Sie alle langsamen Abfragen, für die auf diese Datenquellen verwiesen wird.
7. Führen Sie eine ressourcenspezifische Analyse der langsamen Abfragen durch, und verfolgen Sie, wie die Daten verwendet und verbraucht werden.

Suchen Sie nach folgenden Symptomen:

- Häufige, umfangreiche E/A-Anforderungen, die an dieselbe Ressource oder einen Datenspeicher gesendet werden.
- Auftreten von Konflikten in einer freigegebenen Ressource oder in einem Datenspeicher.
- Ein Vorgang, für den häufig große Datenmengen über das Netzwerk eingehen.
- Anwendungen und Dienste, für die sehr lange auf den Abschluss von E/A-Vorgängen gewartet wird.

## <a name="example-diagnosis"></a>Beispieldiagnose    

In den folgenden Abschnitten werden diese Schritte auf die vorherigen Beispiele angewendet.

### <a name="identify-slow-workloads"></a>Identifizieren von langsamen Workloads

Mit diesem Graphen werden Leistungsergebnisse eines Auslastungstests dargestellt, bei denen bis zu 400 gleichzeitige Benutzer simuliert wurden, die die oben beschriebene `GetAllFieldsAsync`-Methode ausführen. Der Durchsatz nimmt langsam ab, wenn die Auslastung steigt. Die durchschnittliche Reaktionszeit verlängert sich, wenn die Workload zunimmt. 

![Auslastungstestergebnisse für die GetAllFieldsAsync-Methode][Load-Test-Results-Client-Side1]

Bei einem Auslastungstest für den `AggregateOnClientAsync`-Vorgang zeigt sich ein ähnliches Muster. Die Menge der Anforderungen ist relativ stabil. Die durchschnittliche Reaktionszeit nimmt mit der Workload zu, aber langsamer als im vorherigen Graphen.

![Auslastungstestergebnisse für die AggregateOnClientAsync-Methode][Load-Test-Results-Client-Side2]

### <a name="correlate-slow-workloads-with-behavioral-patterns"></a>Korrelieren von langsamen Workloads mit Verhaltensmustern

Jede Korrelation zwischen normalen Zeiträumen mit hoher Nutzung und langsamer Leistung können Anzeichen für Probleme sein. Sehen Sie sich das Leistungsprofil von Funktionen, für die Leistungsprobleme vermutet werden, genau an. So können Sie bestimmen, ob das Profil zum zuvor durchgeführten Auslastungstest passt.

Führen Sie Auslastungstests für dieselben Funktionen durch, indem Sie Benutzerlasten Schritt für Schritt anwenden, um den Punkt zu ermitteln, an dem die Leistung erheblich absinkt oder ganz ausfällt. Falls dieser Punkt innerhalb der Grenzen Ihres Nutzungsbereichs aus der Praxis liegt, sollten Sie untersuchen, wie die Funktionen implementiert wurden.

Ein langsamer Vorgang ist nicht unbedingt ein Problem, wenn Folgendes gilt: Er fällt nicht in Zeiten, in denen das System eine hohe Auslastung aufweist, es ist kein zeitkritischer Vorgang, und er wirkt sich nicht negativ auf die Leistung anderer wichtiger Vorgänge aus. Die Generierung von monatlichen Statistiken zum Betrieb kann beispielsweise ein Vorgang mit langer Ausführungsdauer sein, aber er kann häufig als Batchprozess und Auftrag mit niedriger Priorität durchgeführt werden. Es ist aber ein kritischer Geschäftsvorgang, wenn Kunden den Produktkatalog durchsuchen. Konzentrieren Sie sich auf die Telemetriedaten, die von diesen wichtigen Vorgängen generiert werden, um ermitteln zu können, wie die Leistung in Zeiten mit hoher Auslastung variiert.

### <a name="identify-data-sources-in-slow-workloads"></a>Identifizieren von Datenquellen bei langsamen Workloads

Wenn Sie vermuten, dass ein Dienst aufgrund der Art des Datenempfangs eine schlechte Leistung aufweist, sollten Sie untersuchen, wie die Anwendung mit den verwendeten Repositorys interagiert. Überwachen Sie das Livesystem, um Informationen dazu zu erhalten, auf welche Quellen in Zeiten mit schlechter Leistung zugegriffen wird. 

Instrumentieren Sie für jede Datenquelle das System, um Folgendes zu erfassen:

- Die Häufigkeit, mit der auf die einzelnen Datenspeicher zugegriffen wird.
- Die Datenmengen, die im Datenspeicher abgelegt werden und diesen verlassen.
- Den zeitlichen Ablauf dieser Vorgänge, vor allem die Wartezeit der Anforderungen.
- Die Art und Häufigkeit der Fehler, die auftreten, während bei typischer Auslastung auf die einzelnen Datenspeicher zugegriffen wird.

Vergleichen Sie diese Informationen mit dem Datenvolumen, das von der Anwendung an den Client zurückgegeben wird. Verfolgen Sie das Verhältnis des zurückgegebenen Datenvolumens für den Datenspeicher und des an den Client zurückgegebenen Datenvolumens nach. Falls ein großer Unterschied zu erkennen ist, sollten Sie den Vorgang untersuchen und ermitteln, ob von der Anwendung nicht benötigte Daten abgerufen werden.

Unter Umständen können Sie diese Daten erfassen, indem Sie das Livesystem überwachen und den Lebenszyklus jeder Benutzeranforderung nachverfolgen, oder Sie können eine Serie von synthetischen Workloads modellieren und für ein Testsystem ausführen.

Mit den folgenden Graphen werden Telemetriedaten dargestellt, die per [New Relic APM][new-relic] während eines Auslastungstests der `GetAllFieldsAsync`-Methode erfasst wurden. Beachten Sie den Unterschied zwischen der Datenmenge, die von der Datenbank empfangen wird, und den dazugehörigen HTTP-Antworten.

![Telemetriedaten für die GetAllFieldsAsync-Methode][TelemetryAllFields]

Für jede Anforderung hat die Datenbank 80.503 Byte zurückgegeben, aber die Antwort an den Client hat nur 19.855 Byte enthalten. Dies sind ca. 25% der Größe der Datenbankantwort. Die Größe der an den Client zurückgegebenen Daten kann je nach Format variieren. Für diesen Auslastungstest hat der Client JSON-Daten angefordert. Separate Tests mit XML (nicht dargestellt) verfügten über eine Antwortgröße von 35.655 Byte oder 44% der Größe der Datenbankantwort.

Beim Auslastungstest für die `AggregateOnClientAsync`-Methode sind die Ergebnisse weniger gemäßigt. In diesem Fall wurde für jeden Test eine Abfrage durchgeführt, bei der jeweils mehr als 280 KB an Daten aus der Datenbank abgerufen wurden, während die JSON-Antwort nur 14 Byte umfasst. Es kommt zu diesem großen Unterschied, weil die Methode ein aggregiertes Ergebnis aus einer großen Datenmenge berechnet.

![Telemetriedaten für die AggregateOnClientAsync-Methode][TelemetryAggregateOnClient]

### <a name="identify-and-analyze-slow-queries"></a>Identifizieren und Analysieren von langsamen Abfragen

Suchen Sie nach den Datenbankabfragen, die die meisten Ressourcen verbrauchen und deren Ausführung am längsten dauert. Sie können eine Instrumentierung hinzufügen, um die Start- und Endzeiten für viele Datenbankvorgänge zu ermitteln. Viele Datenspeicher liefern zudem eingehende Informationen dazu, wie Abfragen durchgeführt und optimiert werden. Im Bereich „Abfrageleistung“ im Verwaltungsportal von Azure SQL-Datenbank können Sie beispielsweise eine Abfrage auswählen und ausführliche Informationen zur Laufzeitleistung anzeigen. Dies ist die Abfrage, die vom `GetAllFieldsAsync`-Vorgang generiert wird:

![Bereich mit Abfragedetails im Verwaltungsportal von Windows Azure SQL-Datenbank][QueryDetails]


## <a name="implement-the-solution-and-verify-the-result"></a>Implementieren der Lösung und Überprüfen des Ergebnisses

Nach dem Umstellen der `GetRequiredFieldsAsync`-Methode auf die Verwendung einer SELECT-Anweisung auf Datenbankseite wurden für den Auslastungstest die folgenden Ergebnisse angezeigt:

![Auslastungstestergebnisse für die GetRequiredFieldsAsync-Methode][Load-Test-Results-Database-Side1]

Für diesen Auslastungstest wurden dieselbe Bereitstellung und dieselbe simulierte Workload mit 400 gleichzeitigen Benutzern wie zuvor verwendet. Der Graph zeigt eine viel niedrigere Wartezeit. Die Antwortzeiten verkürzen sich unter Last auf ca. 1,3 Sekunden, verglichen mit vier Sekunden im vorherigen Fall. Der Durchsatz ist mit 350 Anforderungen pro Sekunde gegenüber 100 Anforderungen zuvor ebenfalls höher. Die Datenmenge, die aus der Datenbank abgerufen wird, liegt nun in der Nähe der Größe der HTTP-Antwortnachrichten.

![Telemetriedaten für die GetRequiredFieldsAsync-Methode][TelemetryRequiredFields]

Beim Auslastungstest mit der `AggregateOnDatabaseAsync`-Methode werden die folgenden Ergebnisse generiert:

![Auslastungstestergebnisse für die AggregateOnDatabaseAsync-Methode][Load-Test-Results-Database-Side2]

Die durchschnittliche Antwortzeit ist jetzt sehr niedrig. Dies ist eine deutliche Verbesserung der Leistung, die hauptsächlich durch die starke Reduzierung der E/A-Vorgänge für die Datenbank erzielt wurde. 

Hier sind die entsprechenden Telemetriedaten für die `AggregateOnDatabaseAsync`-Methode dargestellt. Die aus der Datenbank abgerufene Datenmenge wurde stark reduziert, und zwar von über 280 KB pro Transaktion auf 53 Byte. Aus diesem Grund ist die maximale dauerhafte Anzahl von Anforderungen pro Minute von ca. 2.000 auf über 25.000 angestiegen.

![Telemetriedaten für die AggregateOnDatabaseAsync-Methode][TelemetryAggregateInDatabaseAsync]


## <a name="related-resources"></a>Zugehörige Ressourcen

- [Antimuster „Busy Database“][BusyDatabase] (Ausgelastete Datenbank)
- [Antimuster „Chatty I/O“][chatty-io] (Sehr hohe Zahl von E/A-Vorgängen)
- [Data partitioning][data-partitioning] (Datenpartitionierung)


[BusyDatabase]: ../busy-database/index.md
[chatty-io]: ../chatty-io.md
[data-partitioning]: ../../best-practices/data-partitioning.md
[new-relic]: https://newrelic.com/application-monitoring

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ExtraneousFetching

[chatty-io]: ../chatty-io/index.md
[MonolithicPersistence]: ../monolithic-persistence/index.md
[Load-Test-Results-Client-Side1]:_images/LoadTestResultsClientSide1.jpg
[Load-Test-Results-Client-Side2]:_images/LoadTestResultsClientSide2.jpg
[Load-Test-Results-Database-Side1]:_images/LoadTestResultsDatabaseSide1.jpg
[Load-Test-Results-Database-Side2]:_images/LoadTestResultsDatabaseSide2.jpg
[QueryPerformanceZoomed]: _images/QueryPerformanceZoomed.jpg
[QueryDetails]: _images/QueryDetails.jpg
[TelemetryAllFields]: _images/TelemetryAllFields.jpg
[TelemetryAggregateOnClient]: _images/TelemetryAggregateOnClient.jpg
[TelemetryRequiredFields]: _images/TelemetryRequiredFields.jpg
[TelemetryAggregateInDatabaseAsync]: _images/TelemetryAggregateInDatabase.jpg
