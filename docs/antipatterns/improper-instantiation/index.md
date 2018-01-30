---
title: "Antimuster für ungeeignete Instanziierung"
description: "Vermeiden Sie es, ständig neue Instanzen eines Objekts zu erstellen, das einmal erstellt und dann freigegeben werden soll."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 4b217f7fc644901eb5c3e77319d151caed30eef1
ms.sourcegitcommit: cf207fd10110f301f1e05f91eeb9f8dfca129164
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/29/2018
---
# <a name="improper-instantiation-antipattern"></a>Antimuster für ungeeignete Instanziierung

Es kann für die Leistung nachteilhaft sein, immer wieder neue Instanzen eines Objekts zu erzeugen, das einmal erstellt und dann freigegeben werden soll. 

## <a name="problem-description"></a>Problembeschreibung

Viele Bibliotheken stellen Abstraktionen von externen Ressourcen bereit. Intern verwalten diese Klassen in der Regel ihre eigenen Verbindungen mit der Ressource. Dabei fungieren sie als Broker, mit denen Clients auf die Ressource zugreifen können. Im Folgenden werden einige Beispiele für Brokerklassen aufgeführt, die für Azure-Anwendungen relevant sind:

- `System.Net.Http.HttpClient`(Fixierte Verbindung) festgelegt ist(Fixierte Verbindung) festgelegt ist. Kommuniziert über HTTP mit einem Webdienst.
- `Microsoft.ServiceBus.Messaging.QueueClient`(Fixierte Verbindung) festgelegt ist(Fixierte Verbindung) festgelegt ist. Stellt Nachrichten für eine Service Bus-Warteschlange bereit, und empfängt diese. 
- `Microsoft.Azure.Documents.Client.DocumentClient`(Fixierte Verbindung) festgelegt ist(Fixierte Verbindung) festgelegt ist. Stellt eine Verbindung mit einer Cosmos DB-Instanz her.
- `StackExchange.Redis.ConnectionMultiplexer`(Fixierte Verbindung) festgelegt ist(Fixierte Verbindung) festgelegt ist. Stellt eine Verbindung mit Redis, einschließlich Azure Redis Cache, her.

Diese Klassen sind dafür gedacht, einmal instanziiert und über die gesamte Lebensdauer einer Anwendung hinweg wiederverwendet zu werden. Es ist jedoch ein weit verbreiteter Irrtum, dass diese Klassen nur bei Bedarf erworben und schnell freigegeben werden sollen. (Die hier aufgelisteten Klassen beziehen sich zufälligerweise auf .NET-Bibliotheken, aber das Muster ist nicht eindeutig für .NET.)

Im folgenden ASP.NET-Beispiel wird eine Instanz von `HttpClient` für die Kommunikation mit einem Remotedienst erstellt. Das vollständige Beispiel finden Sie [hier][sample-app].

```csharp
public class NewHttpClientInstancePerRequestController : ApiController
{
    // This method creates a new instance of HttpClient and disposes it for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        using (var httpClient = new HttpClient())
        {
            var hostName = HttpContext.Current.Request.Url.Host;
            var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
            return new Product { Name = result };
        }
    }
}
```

In einer Webanwendung ist diese Methode nicht skalierbar. Für jede Benutzeranforderung wird ein neues `HttpClient`-Objekt erstellt. Bei hoher Auslastung kann durch den Webserver die Anzahl der verfügbaren Sockets ausgeschöpft werden, was zu `SocketException`-Fehlern führt.

Dieses Problem ist nicht auf die `HttpClient`-Klasse beschränkt. Andere Klassen, die Ressourcen umschließen oder deren Erstellung kostspielig ist, können ähnliche Probleme verursachen. Im folgenden Beispiel wird eine Instanz der `ExpensiveToCreateService`-Klasse erstellt. Hier ist die Erschöpfung von Sockets nicht unbedingt problematisch, sondern lediglich die Dauer für die Erstellung der einzelnen Instanzen. Das kontinuierliche Erstellen und Zerstören von Instanzen dieser Klasse kann die Skalierbarkeit des Systems beeinträchtigen.

```csharp
public class NewServiceInstancePerRequestController : ApiController
{
    public async Task<Product> GetProductAsync(string id)
    {
        var expensiveToCreateService = new ExpensiveToCreateService();
        return await expensiveToCreateService.GetProductByIdAsync(id);
    }
}

public class ExpensiveToCreateService
{
    public ExpensiveToCreateService()
    {
        // Simulate delay due to setup and configuration of ExpensiveToCreateService
        Thread.SpinWait(Int32.MaxValue / 100);
    }
    ...
}
```

## <a name="how-to-fix-the-problem"></a>Beheben des Problems

Wenn die Klasse, die die externe Ressource umschließt, gemeinsam nutzbar und threadsicher ist, erstellen Sie eine gemeinsame Singletoninstanz oder einen Pool von wiederverwendbaren Instanzen der Klasse.

Im folgenden Beispiel wird eine statische `HttpClient`-Instanz verwendet. Dadurch wird die Verbindung für alle Anforderungen freigegeben.

```csharp
public class SingleHttpClientInstanceController : ApiController
{
    private static readonly HttpClient HttpClient;

    static SingleHttpClientInstanceController()
    {
        HttpClient = new HttpClient();
    }

    // This method uses the shared instance of HttpClient for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        var hostName = HttpContext.Current.Request.Url.Host;
        var result = await HttpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
        return new Product { Name = result };
    }
}
```

## <a name="considerations"></a>Überlegungen

- Das Schlüsselelement dieses Antimusters ist das wiederholte Erstellen und Zerstören von Instanzen eines *gemeinsam nutzbaren* Objekts. Wenn eine Klasse nicht gemeinsam nutzbar (nicht threadsicher) ist, dann wird dieses Antimuster nicht angewendet.

- Der Typ der freigegebenen Ressource kann darüber entscheiden, ob Sie ein Singleton verwenden oder einen Pool erstellen sollten. Die `HttpClient`-Klasse ist für die gemeinsame Verwendung konzipiert, nicht für das Zusammenfassen in einem Pool. Andere Objekte können das Pooling unterstützen, sodass das System die Workload auf mehrere Instanzen verteilen kann.

- Objekte, die Sie über mehrere Anforderungen hinweg freigeben, *müssen* threadsicher sein. Die `HttpClient`-Klasse ist zwar für diese Art der Verwendung konzipiert, allerdings kann es sein, dass andere Klassen keine gleichzeitigen Anforderungen unterstützen. Sehen Sie daher in der verfügbaren Dokumentation nach.

- Einige Ressourcentypen sind knapp und sollten nicht beibehalten werden. Dies gilt beispielsweise für Datenbankverbindungen. Das Aufrechterhalten einer offenen Datenbankverbindung, die nicht erforderlich ist, kann andere gleichzeitige Benutzer daran hindern, auf die Datenbank zuzugreifen.

- Im .NET Framework werden viele Objekte, die Verbindungen mit externen Ressourcen herstellen, mit statischen Factorymethoden anderer Klassen erstellt, die diese Verbindungen verwalten. Diese Factoryobjekte sind für die Speicherung und Wiederverwendung bestimmt, nicht für die Löschung und Wiederherstellung. Beispielsweise wird im Azure Service Bus das `QueueClient`-Objekt durch ein `MessagingFactory`-Objekt erstellt. Intern verwaltet `MessagingFactory` Verbindungen. Weitere Informationen finden Sie unter [Bewährte Methoden für Leistungsoptimierungen mithilfe von Service Bus-Messaging][service-bus-messaging].

## <a name="how-to-detect-the-problem"></a>Erkennen des Problems

Zu den Symptomen dieses Problems zählen neben einem Rückgang des Durchsatzes oder einer erhöhten Fehlerrate einer oder mehrere der folgenden Punkte: 

- Eine Zunahme von Ausnahmen, die auf die Erschöpfung von Ressourcen wie Sockets, Datenbankverbindungen, Dateihandles usw. hinweist 
- Erhöhter Speicherverbrauch und Garbage Collection
- Eine Zunahme der Netzwerk-, Festplatten- oder Datenbankaktivität

Sie können die folgenden Schritte durchführen, um dieses Problem zu identifizieren:

1. Führen Sie eine Prozessüberwachung des Produktionssystems durch, um Punkte zu identifizieren, an denen Antwortzeiten verlangsamt werden oder aufgrund mangelnder Ressourcen ein Fehler im System auftritt.
2. Untersuchen Sie die an diesen Punkten erfassten Telemetriedaten, um festzustellen, welche Vorgänge ressourcenverbrauchende Objekte erstellen und zerstören könnten.
3. Führen Sie für jeden vermuteten Vorgang in einer kontrollierten Testumgebung einen Auslastungstest durch (nicht im Produktionssystem).
4. Überprüfen Sie den Quellcode, und untersuchen Sie, wie Brokerobjekte verwaltet werden.

Schauen Sie sich Stapelüberwachungen für Vorgänge an, die langsam ablaufen oder Ausnahmen erzeugen, wenn das System ausgelastet ist. Anhand dieser Informationen können Sie erkennen, wie diese Vorgänge Ressourcen verwenden. Mithilfe von Ausnahmen kann festgestellt werden, ob Fehler dadurch verursacht werden, dass die gemeinsam genutzten Ressourcen ausgeschöpft sind. 

## <a name="example-diagnosis"></a>Beispieldiagnose

In den folgenden Abschnitten werden diese Schritte auf die zuvor beschriebene Beispielanwendung angewendet.

### <a name="identify-points-of-slow-down-or-failure"></a>Identifizieren der Punkte der Verlangsamung oder des Ausfalls

Die folgende Abbildung zeigt mit [New Relic APM][new-relic] erzeugte Ergebnisse, die Vorgänge mit einer schlechten Antwortzeit zeigen. In diesem Fall lohnt es sich, sich mit der `GetProductAsync`-Methode im `NewHttpClientInstancePerRequest`-Controller näher auseinanderzusetzen. Beachten Sie, dass sich auch die Fehlerrate erhöht, wenn diese Vorgänge ausgeführt werden. 

![New Relic-Überwachungsdashboard mit der Beispielanwendung für die Erstellung einer neuen HttpClient-Objektinstanz für die einzelnen Anforderungen][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a>Untersuchen der Telemetriedaten und Ermitteln von Korrelationen

Die nächste Abbildung zeigt Daten, die über denselben Zeitraum wie bei der vorherigen Abbildung mit der Threadprofilerstellung erfasst wurden. Das Öffnen von Socketverbindungen durch das System dauert sehr lange. Das Schließen und Verarbeiten von Socketausnahmen dauert sogar noch länger.

![New Relic-Threadprofilersteller mit der Beispielanwendung für die Erstellung einer neuen HttpClient-Objektinstanz für die einzelnen Anforderungen][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a>Durchführen von Auslastungstests

Simulieren Sie mithilfe von Auslastungstests typische Vorgänge, die von Benutzern ausgeführt werden könnten. Hierdurch können besser die Teile eines Systems ermittelt werden, bei denen unter wechselnder Belastung Ressourcenauslastung auftritt. Führen Sie diese Tests in einer kontrollierten Umgebung und nicht im Produktionssystem durch. Das folgende Diagramm zeigt den Durchsatz der Anforderungen, die vom `NewHttpClientInstancePerRequest`-Controller bearbeitet werden, wenn die Benutzerauslastung auf 100 gleichzeitige Benutzer steigt.

![Durchsatz der Beispielanwendung für die Erstellung einer neuen HttpClient-Objektinstanz für die einzelnen Anforderungen][throughput-new-HTTPClient-instance]

Die Anzahl der pro Sekunde bearbeiteten Anforderungen steigt zunächst mit zunehmender Workloadanzahl. Bei ca. 30 Benutzern erreicht die Anzahl der erfolgreichen Anforderungen jedoch eine Begrenzung, und das System beginnt, Ausnahmen zu generieren. Von da an nimmt die Anzahl der Ausnahmen mit der Benutzerauslastung allmählich zu. 

Beim Auslastungstest wurden diese Fehler als „HTTP 500 (Interner Serverfehler)“-Fehler gemeldet. Die Überprüfung der Telemetriedaten ergab, dass diese Fehler dadurch verursacht wurden, dass die Socketressourcen durch das System ausgeschöpft wurden, je mehr `HttpClient`-Objekte erstellt wurden.

Das nächste Diagramm zeigt einen ähnlichen Test für einen Controller, der das benutzerdefinierte `ExpensiveToCreateService`-Objekt erstellt.

![Durchsatz der Beispielanwendung für die Erstellung einer neuen ExpensiveToCreateService-Objektinstanz für die einzelnen Anforderungen][throughput-new-ExpensiveToCreateService-instance]

Dieses Mal generiert der Controller keine Ausnahmen, aber der Durchsatz befindet sich nach wie vor im Stillstand, während die durchschnittliche Reaktionszeit um den Faktor 20 zunimmt. (Das Diagramm verwendet eine logarithmische Skalierung für die Antwortzeit und den Durchsatz.) Die Telemetriedaten zeigen, dass die Erstellung neuer `ExpensiveToCreateService`-Instanzen die Hauptursache des Problems war.

### <a name="implement-the-solution-and-verify-the-result"></a>Implementieren der Lösung und Überprüfen des Ergebnisses

Nach dem Wechsel der `GetProductAsync`-Methode zur Freigabe einer einzelnen `HttpClient`-Instanz zeigte ein zweiter Auslastungstest, dass die Leistung verbessert wurde. Es wurden keine Fehler gemeldet, und das System war in der Lage, eine steigende Last von bis zu 500 Anforderungen pro Sekunde zu bewältigen. Die durchschnittliche Antwortzeit wurde im Vergleich zum vorherigen Test halbiert.

![Durchsatz der Beispielanwendung für die Erstellung derselben HttpClient-Objektinstanz für die einzelnen Anforderungen][throughput-single-HTTPClient-instance]

Zum Vergleich: Die folgende Abbildung zeigt die Telemetriedaten der Stapelüberwachung. Dieses Mal wendet das System den Großteil der Zeit für die Durchführung realer Aufgaben statt für das Öffnen und Schließen von Sockets.

![New Relic-Threadprofilersteller mit der Beispielanwendung für die Erstellung einer einzelnen HttpClient-Objektinstanz für alle Anforderungen][thread-profiler-single-HTTPClient-instance]

Das nächste Diagramm zeigt einen ähnlichen Auslastungstest mit einer freigegebenen Instanz des `ExpensiveToCreateService`-Objekts. Auch hier steigt die Anzahl der bearbeiteten Anforderungen mit der Benutzerauslastung, während die durchschnittliche Antwortzeit gering bleibt. 

![Durchsatz der Beispielanwendung für die Erstellung derselben HttpClient-Objektinstanz für die einzelnen Anforderungen][throughput-single-ExpensiveToCreateService-instance]



[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ImproperInstantiation
[service-bus-messaging]: /azure/service-bus-messaging/service-bus-performance-improvements
[new-relic]: https://newrelic.com/application-monitoring
[throughput-new-HTTPClient-instance]: _images/HttpClientInstancePerRequest.jpg
[dashboard-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestWebTransactions.jpg
[thread-profiler-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestThreadProfile.jpg
[throughput-new-ExpensiveToCreateService-instance]: _images/ServiceInstancePerRequest.jpg
[throughput-single-HTTPClient-instance]: _images/SingleHttpClientInstance.jpg
[throughput-single-ExpensiveToCreateService-instance]: _images/SingleServiceInstance.jpg
[thread-profiler-single-HTTPClient-instance]: _images/SingleHttpClientInstanceThreadProfile.jpg
