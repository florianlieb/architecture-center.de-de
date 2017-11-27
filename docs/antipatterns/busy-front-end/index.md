---
title: "Antimuster für ausgelastete Front-Ends"
description: "Die Ausführung asynchroner Arbeiten in einer großen Anzahl von Hintergrundthreads kann Vordergrundaufgaben von Ressourcen blockieren."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: cedb80ddac5ceb1eb901455df3165993fd28a138
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="busy-front-end-antipattern"></a>Antimuster für ausgelastete Front-Ends

Die Ausführung asynchroner Arbeiten in einer großen Anzahl von Hintergrundthreads kann andere gleichzeitig ausgeführte Vordergrundaufgaben von Ressourcen blockieren und die Antwortzeiten dadurch auf ein inakzeptables Niveau reduzieren.

## <a name="problem-description"></a>Problembeschreibung

Ressourcenintensive Aufgaben können die Antwortzeiten für Benutzeranforderungen erhöhen und zu langen Wartezeiten führen. Eine Möglichkeit zur Verbesserung der Antwortzeiten ist die Auslagerung ressourcenintensiver Aufgaben in einen separaten Thread. Durch diesen Ansatz kann die Anwendung reaktionsfähig bleiben, während die Verarbeitung im Hintergrund erfolgt. Aufgaben, die in einem Hintergrundthread ausgeführt werden, verbrauchen jedoch weiterhin Ressourcen. Wenn zu viele dieser Aufgaben vorhanden sind, können sie die Threads blockieren, die Anforderungen verarbeiten.

> [!NOTE]
> Der Begriff *Ressource* kann vieles umfassen, beispielsweise die CPU-Auslastung, die Belegung von Speicher und die Netzwerk- oder Datenträger-E/A-Vorgänge.

Dieses Problem tritt in der Regel auf, wenn eine Anwendung als monolithischer Code entwickelt und die gesamte Geschäftslogik in einer einzelnen, für die Darstellungsschicht freigegebenen Ebene zusammengefasst wird.

Das folgende Beispiel mit ASP.NET veranschaulicht das Problem. Das vollständige Codebeispiel finden Sie [hier][code-sample].

```csharp
public class WorkInFrontEndController : ApiController
{
    [HttpPost]
    [Route("api/workinfrontend")]
    public HttpResponseMessage Post()
    {
        new Thread(() =>
        {
            //Simulate processing
            Thread.SpinWait(Int32.MaxValue / 100);
        }).Start();

        return Request.CreateResponse(HttpStatusCode.Accepted);
    }
}

public class UserProfileController : ApiController
{
    [HttpGet]
    [Route("api/userprofile/{id}")]
    public UserProfile Get(int id)
    {
        //Simulate processing
        return new UserProfile() { FirstName = "Alton", LastName = "Hudgens" };
    }
}
```

- Die `Post`-Methode im `WorkInFrontEnd`-Controller implementiert einen HTTP POST-Vorgang. Dieser Vorgang simuliert eine CPU-intensive Aufgabe mit langer Ausführungszeit. Die Arbeit erfolgt in einem separaten Thread, um einen schnellen Abschluss des POST-Vorgangs zu ermöglichen.

- Die `Get`-Methode im `UserProfile`-Controller implementiert einen HTTP GET-Vorgang. Diese Methode ist deutlich weniger CPU-intensiv.

Das vorrangige Problem sind die Ressourcenanforderungen der `Post`-Methode. Obwohl die Arbeit in einem Hintergrundthread ausgeführt wird, kann sie erhebliche CPU-Ressourcen beanspruchen. Diese Ressourcen werden für andere Vorgänge freigegeben, die von anderen gleichzeitigen Benutzern ausgeführt werden. Wenn eine moderate Anzahl von Benutzern diese Anforderung zur gleichen Zeit sendet, wird die Gesamtleistung vermutlich darunter leiden, sodass alle Vorgänge verlangsamt werden. Benutzer können beispielsweise eine wesentliche Wartezeit bei der `Get`-Methode feststellen.

## <a name="how-to-fix-the-problem"></a>Beheben des Problems

Verschieben Sie Prozesse, die erhebliche Ressourcen beanspruchen, auf ein separates Back-End. 

Bei diesem Ansatz reiht das Front-End ressourcenintensive Aufgaben in eine Nachrichtenwarteschlange ein. Das Back-End wählt die Aufgaben zur asynchronen Verarbeitung aus. Die Warteschlange fungiert auch als Lastenausgleich, da sie Anforderungen für das Back-End puffert. Wenn die Warteschlange zu lang wird, können Sie die automatische Skalierung konfigurieren, um das Back-End horizontal hochzuskalieren.

Hier sehen Sie eine überarbeitete Version des obigen Codes. In dieser Version reiht die `Post`-Methode eine Nachricht in eine Service Bus-Warteschlange ein. 

```csharp
public class WorkInBackgroundController : ApiController
{
    private static readonly QueueClient QueueClient;
    private static readonly string QueueName;
    private static readonly ServiceBusQueueHandler ServiceBusQueueHandler;

    public WorkInBackgroundController()
    {
        var serviceBusConnectionString = ...;
        QueueName = ...;
        ServiceBusQueueHandler = new ServiceBusQueueHandler(serviceBusConnectionString);
        QueueClient = ServiceBusQueueHandler.GetQueueClientAsync(QueueName).Result;
    }

    [HttpPost]
    [Route("api/workinbackground")]
    public async Task<long> Post()
    {
        return await ServiceBusQueuehandler.AddWorkLoadToQueueAsync(QueueClient, QueueName, 0);
    }
}
```

Das Back-End pullt Nachrichten aus der Service Bus-Warteschlange und führt die Verarbeitung aus.

```csharp
public async Task RunAsync(CancellationToken cancellationToken)
{
    this._queueClient.OnMessageAsync(
        // This lambda is invoked for each message received.
        async (receivedMessage) =>
        {
            try
            {
                // Simulate processing of message
                Thread.SpinWait(Int32.Maxvalue / 1000);

                await receivedMessage.CompleteAsync();
            }
            catch
            {
                receivedMessage.Abandon();
            }
        });
}
```

## <a name="considerations"></a>Überlegungen

- Dieser Ansatz erhöht die Komplexität der Anwendung zusätzlich. Sie müssen das Einreihen in die und Entfernen aus der Warteschlange sicher behandeln, damit im Fall eines Fehlers keine Anforderungen verloren gehen.
- Die Anwendung ist von einem zusätzlichen Dienst für die Nachrichtenwarteschlange abhängig.
- Die Verarbeitungsumgebung muss ausreichend skalierbar sein, um die erwartete Arbeitsauslastung zu bewältigen und die erforderlichen Durchsatzziele zu erfüllen.
- Dieser Ansatz sollte zwar die allgemeine Reaktionsfähigkeit verbessern, die Ausführung der auf das Back-End verschobenen Aufgaben kann jedoch mehr Zeit in Anspruch nehmen. 

## <a name="how-to-detect-the-problem"></a>Erkennen des Problems

Zu den Symptomen eines ausgelasteten Front-Ends zählt die lange Wartezeit bei der Ausführung ressourcenintensiver Aufgaben. Endbenutzer berichten vermutlich von längeren Antwortzeiten oder Fehlern aufgrund von Diensten, bei denen ein Timeout auftritt. In diesen Fällen können auch Fehler vom Typ „HTTP 500 (interner Server)“ oder „HTTP 503 (Dienst nicht verfügbar)“ zurückgegeben werden. Überprüfen Sie die Ereignisprotokolle für den Webserver. Sie enthalten wahrscheinlich ausführlichere Informationen zu den Ursachen und Umständen der Fehler.

Sie können die folgenden Schritte ausführen, um dieses Problem zu identifizieren:

1. Führen Sie eine Prozessüberwachung des Produktionssystems durch, um Punkte zu identifizieren, an denen Antwortzeiten verlangsamt werden.
2. Untersuchen Sie die an diesen Punkten erfassten Telemetriedaten, um die ausgeführte Kombination von Vorgängen und die verwendeten Ressourcen zu ermitteln. 
3. Suchen Sie nach Korrelationen zwischen langen Antwortzeiten und der Anzahl sowie den Kombinationen von Vorgängen, die zu diesen Zeitpunkten ausgeführt wurden.
4. Führen Sie einen Auslastungstest für jeden „verdächtigen“ Vorgang aus, um herauszufinden, welche Vorgänge Ressourcen verbrauchen und andere Vorgänge blockieren. 
5. Überprüfen Sie den Quellcode für diese Vorgänge, um zu ermitteln, weshalb sie einen übermäßigen Ressourcenverbrauch verursachen könnten.

## <a name="example-diagnosis"></a>Beispieldiagnose 

In den folgenden Abschnitten werden diese Schritte auf die zuvor beschriebene Beispielanwendung angewendet.

### <a name="identify-points-of-slowdown"></a>Identifizieren der Punkte, an denen eine Verlangsamung auftritt

Instrumentieren Sie jede Methode, um die Dauer der einzelnen Anforderungen und die von ihnen verbrauchten Ressourcen nachzuverfolgen. Überwachen Sie die Anwendung anschließend in der Produktionsumgebung. Dies kann Ihnen einen allgemeinen Überblick darüber bieten, wie Anforderungen miteinander um Ressourcen konkurrieren. In Zeiten hoher Belastung beeinträchtigen ressourcenintensive Anforderungen mit langer Ausführungszeit wahrscheinlich andere Vorgänge. Dieses Verhalten kann durch eine Überwachung des Systems und der Leistungsabnahme beobachtet werden.

Der folgende Screenshot zeigt ein Überwachungsdashboard. (Wir haben [AppDynamics] für unsere Tests verwendet.) Zu Beginn ist die Auslastung des Systems gering. Dann beginnen Benutzer, die `UserProfile`-GET-Methode anzufordern. Die Leistung ist einigermaßen gut, bis andere Benutzer Anforderungen an die `WorkInFrontEnd`-POST-Methode ausgeben. An diesem Punkt nehmen die Antwortzeiten drastisch zu (erster Pfeil). Die Antwortzeiten verbessern sich erst, nachdem die Anzahl von Anforderungen an den `WorkInFrontEnd`-Controller abgenommen hat (zweiter Pfeil).

![Der AppDynamics-Bereich für die Geschäftstransaktionen zeigt die Auswirkungen der Antwortzeiten aller Anforderungen, wenn der WorkInFrontEnd-Controller verwendet wird.][AppDynamics-Transactions-Front-End-Requests]

### <a name="examine-telemetry-data-and-find-correlations"></a>Untersuchen der Telemetriedaten und Ermitteln von Korrelationen

Die nächste Abbildung zeigt einige der Metriken, die zum Überwachen der Ressourcenverwendung im gleichen Zeitintervall gesammelt wurden. Zunächst greifen nur wenig Benutzer auf das System zu. Sobald weitere Benutzer eine Verbindung herstellen, steigt die CPU-Auslastung erheblich (100 %). Beachten Sie auch, dass die Netzwerk-E/A-Rate bei der Zunahme der CPU-Auslastung anfänglich steigt. Nachdem die CPU-Auslastung den Höchstpunkt erreicht hat, nimmt die Netzwerk-E/A-Rate aber sogar ab. Dies liegt daran, dass das System nur eine relativ kleine Anzahl von Anforderungen verarbeiten kann, sobald die CPU voll ausgelastet ist. Wenn Benutzer die Verbindung trennen, nimmt die CPU-Auslastung ab.

![AppDynamics-Metriken zur CPU- und Netzwerkauslastung][AppDynamics-Metrics-Front-End-Requests]

An diesem Punkt ist die `Post`-Methode im `WorkInFrontEnd`-Controller anscheinend ein erstklassiger Kandidat für eine genauere Prüfung. Zur Bestätigung dieser Hypothese sind weitere Schritte in einer kontrollierten Umgebung erforderlich.

### <a name="perform-load-testing"></a>Durchführen von Auslastungstests 

Der nächste Schritt ist die Ausführung von Tests in einer kontrollierten Umgebung. Führen Sie beispielsweise eine Reihe von Auslastungstests durch, bei denen jede Anforderung nacheinander einbezogen und dann ausgelassen wird, um die Auswirkungen anzuzeigen.

Das folgende Diagramm zeigt die Ergebnisse eines Auslastungstests für eine identische Bereitstellung des in den vorherigen Tests verwendeten Clouddiensts. Beim Test wurden eine konstante Last von 500 Benutzern, die den `Get`-Vorgang im `UserProfile`-Controller ausführen, sowie eine schrittweise Last von Benutzern, die den `Post`-Vorgang im `WorkInFrontEnd`-Controller ausführen, verwendet. 

![Anfängliche Testergebnisse für den WorkInFrontEnd-Controller][Initial-Load-Test-Results-Front-End]

Anfangs beträgt die schrittweise Last 0, d. h. nur die aktiven Benutzer führen die `UserProfile`-Anforderungen aus. Das System kann auf ca. 500 Anforderungen pro Sekunde reagieren. Nach 60 Sekunden beginnt eine Last von 100 zusätzlichen Benutzern, POST-Anforderungen an den `WorkInFrontEnd`-Controller zu senden. Die an den `UserProfile`-Controller gesendete Arbeitsauslastung sinkt nahezu sofort auf ungefähr 150 Anforderungen pro Sekunde. Dies ist auf die Funktionsweise des Auslastungstests zurückzuführen. Er wartet vor dem Senden der nächsten Anforderung auf eine Antwort. Je länger es dauert, eine Antwort zu empfangen, desto niedriger ist folglich die Anforderungsrate.

Wenn weitere Benutzer POST-Anforderungen an den `WorkInFrontEnd`-Controller senden, nimmt die Antwortrate des `UserProfile`-Controllers weiter ab. Beachten Sie jedoch, dass die Anzahl der vom `WorkInFrontEnd`-Controller verarbeiteten Anforderungen relativ konstant bleibt. Die Sättigung des Systems wird deutlich, sobald die Gesamtrate beider Anforderungen einen stabilen, aber niedrigen Grenzwert erreicht.


### <a name="review-the-source-code"></a>Überprüfen des Quellcodes

Der letzte Schritt besteht darin, den Quellcode zu überprüfen. Das Entwicklungsteam wusste, dass die `Post`-Methode viel Zeit in Anspruch nehmen könnte, und hat in der ursprünglichen Implementierung daher einen separaten Thread verwendet. Dadurch wurde das unmittelbare Problem gelöst, da die `Post`-Methode nicht durch das Warten auf den Abschluss einer Aufgabe mit langer Ausführungsdauer blockiert wurde.

Die von dieser Methode ausgeführte Arbeit verbraucht jedoch nach wie vor CPU-Zeit, Arbeitsspeicher und andere Ressourcen. Wenn die asynchrone Ausführung dieses Prozesses ermöglicht wird, kann die Leistung beeinträchtigt werden, da Benutzer auf unkontrollierte Weise eine große Anzahl dieser Vorgänge gleichzeitig auslösen können. Die Anzahl von Threads, die von einem Server ausgeführt werden können, ist begrenzt. Wird dieser Grenzwert überschritten, wird bei dem Versuch, einen neuen Thread zu starten, wahrscheinlich eine Ausnahme in der Anwendung ausgelöst.

> [!NOTE]
> Dies bedeutet nicht, dass Sie asynchrone Vorgänge vermeiden sollten. Das Ausführen eines asynchronen Wartevorgangs für einen Netzwerkaufruf ist eine empfohlene Vorgehensweise. (Siehe das Antimuster für [synchrone E/A][sync-io].) Hier besteht das Problem darin, dass CPU-intensive Vorgänge in einem anderen Thread erzeugt wurden. 

### <a name="implement-the-solution-and-verify-the-result"></a>Implementieren der Lösung und Überprüfen des Ergebnisses

Die folgende Abbildung zeigt die Leistungsüberwachung nach dem Implementieren der Lösung. Die Auslastung war mit der zuvor gezeigten Auslastung vergleichbar, die Antwortzeiten für den `UserProfile`-Controller sind jedoch deutlich schneller. Die Anzahl von Anforderungen hat über den gleichen Zeitraum von 2.759 auf 23.565 zugenommen. 

![Der AppDynamics-Bereich für die Geschäftstransaktionen zeigt die Auswirkungen der Antwortzeiten aller Anforderungen, wenn der WorkInBackground-Controller verwendet wird.][AppDynamics-Transactions-Background-Requests]

Beachten Sie, dass der `WorkInBackground`-Controller auch eine deutlich größere Anzahl von Anforderungen behandelt hat. In diesem Fall ist allerdings kein direkter Vergleich möglich, da sich die vom Controller ausgeführte Arbeit erheblich vom ursprünglichen Code unterscheidet. Die neue Version reiht eine Anforderung einfach in die Warteschlange ein, anstatt eine zeitaufwändige Berechnung durchzuführen. Entscheidend ist, dass diese Methode bei hoher Last nicht mehr die Leistung des gesamten Systems beeinträchtigt.

Die verbesserte Leistung ist auch an der CPU- und Netzwerkauslastung zu erkennen. Die CPU-Auslastung hat nie 100 % erreicht. Die Anzahl verarbeiteter Netzwerkanforderungen war weitaus höher als zuvor und ist erst bei der Abnahme der Arbeitsauslastung gesunken.

![AppDynamics-Metriken zur CPU- und Netzwerkauslastung für den WorkInBackground-Controller][AppDynamics-Metrics-Background-Requests]

Das folgende Diagramm zeigt die Ergebnisse eines Auslastungstests. Die Gesamtanzahl verarbeiteter Anforderungen hat im Vergleich zu den früheren Tests erheblich zugenommen.

![Ergebnisse des Auslastungstests für den BackgroundImageProcessing-Controller][Load-Test-Results-Background]

## <a name="related-guidance"></a>Verwandte Anweisungen

- [Autoscaling best practices][autoscaling] (Bewährte Methoden für die automatische Skalierung)
- [Background jobs best practices][background-jobs] (Bewährte Methoden für Hintergrundaufträge)
- [Queue-Based Load Leveling pattern][load-leveling] (Warteschlangenbasiertes Lastenausgleichsmuster)
- [Web Queue Worker architecture style][web-queue-worker] (Webwarteschlangen-Workerarchitektur)

[AppDyanamics]: https://www.appdynamics.com/
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[background-jobs]: /azure/architecture/best-practices/background-jobs
[code-sample]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[fullDemonstrationOfSolution]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[load-leveling]: /azure/architecture/patterns/queue-based-load-leveling
[sync-io]: ../synchronous-io/index.md
[web-queue-worker]: /azure/architecture/guide/architecture-styles/web-queue-worker

[WebJobs]: http://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx
[ComputePartitioning]: https://msdn.microsoft.com/library/dn589773.aspx
[ServiceBusQueues]: https://msdn.microsoft.com/library/azure/hh367516.aspx
[AppDynamics-Transactions-Front-End-Requests]: ./_images/AppDynamicsPerformanceStats.jpg
[AppDynamics-Metrics-Front-End-Requests]: ./_images/AppDynamicsFrontEndMetrics.jpg
[Initial-Load-Test-Results-Front-End]: ./_images/InitialLoadTestResultsFrontEnd.jpg
[AppDynamics-Transactions-Background-Requests]: ./_images/AppDynamicsBackgroundPerformanceStats.jpg
[AppDynamics-Metrics-Background-Requests]: ./_images/AppDynamicsBackgroundMetrics.jpg
[Load-Test-Results-Background]: ./_images/LoadTestResultsBackground.jpg


