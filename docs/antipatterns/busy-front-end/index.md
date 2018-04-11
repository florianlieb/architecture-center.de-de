---
title: Antimuster für ausgelastete Front-Ends
description: Die Ausführung asynchroner Arbeiten in einer großen Anzahl von Hintergrundthreads kann Vordergrundaufgaben von Ressourcen blockieren.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: cedb80ddac5ceb1eb901455df3165993fd28a138
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="busy-front-end-antipattern"></a><span data-ttu-id="262b6-103">Antimuster für ausgelastete Front-Ends</span><span class="sxs-lookup"><span data-stu-id="262b6-103">Busy Front End antipattern</span></span>

<span data-ttu-id="262b6-104">Die Ausführung asynchroner Arbeiten in einer großen Anzahl von Hintergrundthreads kann andere gleichzeitig ausgeführte Vordergrundaufgaben von Ressourcen blockieren und die Antwortzeiten dadurch auf ein inakzeptables Niveau reduzieren.</span><span class="sxs-lookup"><span data-stu-id="262b6-104">Performing asynchronous work on a large number of background threads can starve other concurrent foreground tasks of resources, decreasing response times to unacceptable levels.</span></span>

## <a name="problem-description"></a><span data-ttu-id="262b6-105">Problembeschreibung</span><span class="sxs-lookup"><span data-stu-id="262b6-105">Problem description</span></span>

<span data-ttu-id="262b6-106">Ressourcenintensive Aufgaben können die Antwortzeiten für Benutzeranforderungen erhöhen und zu langen Wartezeiten führen.</span><span class="sxs-lookup"><span data-stu-id="262b6-106">Resource-intensive tasks can increase the response times for user requests and cause high latency.</span></span> <span data-ttu-id="262b6-107">Eine Möglichkeit zur Verbesserung der Antwortzeiten ist die Auslagerung ressourcenintensiver Aufgaben in einen separaten Thread.</span><span class="sxs-lookup"><span data-stu-id="262b6-107">One way to improve response times is to offload a resource-intensive task to a separate thread.</span></span> <span data-ttu-id="262b6-108">Durch diesen Ansatz kann die Anwendung reaktionsfähig bleiben, während die Verarbeitung im Hintergrund erfolgt.</span><span class="sxs-lookup"><span data-stu-id="262b6-108">This approach lets the application stay responsive while processing happens in the background.</span></span> <span data-ttu-id="262b6-109">Aufgaben, die in einem Hintergrundthread ausgeführt werden, verbrauchen jedoch weiterhin Ressourcen.</span><span class="sxs-lookup"><span data-stu-id="262b6-109">However, tasks that run on a background thread still consume resources.</span></span> <span data-ttu-id="262b6-110">Wenn zu viele dieser Aufgaben vorhanden sind, können sie die Threads blockieren, die Anforderungen verarbeiten.</span><span class="sxs-lookup"><span data-stu-id="262b6-110">If there are too many of them, they can starve the threads that are handling requests.</span></span>

> [!NOTE]
> <span data-ttu-id="262b6-111">Der Begriff *Ressource* kann vieles umfassen, beispielsweise die CPU-Auslastung, die Belegung von Speicher und die Netzwerk- oder Datenträger-E/A-Vorgänge.</span><span class="sxs-lookup"><span data-stu-id="262b6-111">The term *resource* can encompass many things, such as CPU utilization, memory occupancy, and network or disk I/O.</span></span>

<span data-ttu-id="262b6-112">Dieses Problem tritt in der Regel auf, wenn eine Anwendung als monolithischer Code entwickelt und die gesamte Geschäftslogik in einer einzelnen, für die Darstellungsschicht freigegebenen Ebene zusammengefasst wird.</span><span class="sxs-lookup"><span data-stu-id="262b6-112">This problem typically occurs when an application is developed as monolithic piece of code, with all of the business logic combined into a single tier shared with the presentation layer.</span></span>

<span data-ttu-id="262b6-113">Das folgende Beispiel mit ASP.NET veranschaulicht das Problem.</span><span class="sxs-lookup"><span data-stu-id="262b6-113">Here’s an example using ASP.NET that demonstrates the problem.</span></span> <span data-ttu-id="262b6-114">Das vollständige Codebeispiel finden Sie [hier][code-sample].</span><span class="sxs-lookup"><span data-stu-id="262b6-114">You can find the complete sample [here][code-sample].</span></span>

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

- <span data-ttu-id="262b6-115">Die `Post`-Methode im `WorkInFrontEnd`-Controller implementiert einen HTTP POST-Vorgang.</span><span class="sxs-lookup"><span data-stu-id="262b6-115">The `Post` method in the `WorkInFrontEnd` controller implements an HTTP POST operation.</span></span> <span data-ttu-id="262b6-116">Dieser Vorgang simuliert eine CPU-intensive Aufgabe mit langer Ausführungszeit.</span><span class="sxs-lookup"><span data-stu-id="262b6-116">This operation simulates a long-running, CPU-intensive task.</span></span> <span data-ttu-id="262b6-117">Die Arbeit erfolgt in einem separaten Thread, um einen schnellen Abschluss des POST-Vorgangs zu ermöglichen.</span><span class="sxs-lookup"><span data-stu-id="262b6-117">The work is performed on a separate thread, in an attempt to enable the POST operation to complete quickly.</span></span>

- <span data-ttu-id="262b6-118">Die `Get`-Methode im `UserProfile`-Controller implementiert einen HTTP GET-Vorgang.</span><span class="sxs-lookup"><span data-stu-id="262b6-118">The `Get` method in the `UserProfile` controller implements an HTTP GET operation.</span></span> <span data-ttu-id="262b6-119">Diese Methode ist deutlich weniger CPU-intensiv.</span><span class="sxs-lookup"><span data-stu-id="262b6-119">This method is much less CPU intensive.</span></span>

<span data-ttu-id="262b6-120">Das vorrangige Problem sind die Ressourcenanforderungen der `Post`-Methode.</span><span class="sxs-lookup"><span data-stu-id="262b6-120">The primary concern is the resource requirements of the `Post` method.</span></span> <span data-ttu-id="262b6-121">Obwohl die Arbeit in einem Hintergrundthread ausgeführt wird, kann sie erhebliche CPU-Ressourcen beanspruchen.</span><span class="sxs-lookup"><span data-stu-id="262b6-121">Although it puts the work onto a background thread, the work can still consume considerable CPU resources.</span></span> <span data-ttu-id="262b6-122">Diese Ressourcen werden für andere Vorgänge freigegeben, die von anderen gleichzeitigen Benutzern ausgeführt werden.</span><span class="sxs-lookup"><span data-stu-id="262b6-122">These resources are shared with other operations being performed by other concurrent users.</span></span> <span data-ttu-id="262b6-123">Wenn eine moderate Anzahl von Benutzern diese Anforderung zur gleichen Zeit sendet, wird die Gesamtleistung vermutlich darunter leiden, sodass alle Vorgänge verlangsamt werden.</span><span class="sxs-lookup"><span data-stu-id="262b6-123">If a moderate number of users send this request at the same time, overall performance is likely to suffer, slowing down all operations.</span></span> <span data-ttu-id="262b6-124">Benutzer können beispielsweise eine wesentliche Wartezeit bei der `Get`-Methode feststellen.</span><span class="sxs-lookup"><span data-stu-id="262b6-124">Users might experience significant latency in the `Get` method, for example.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="262b6-125">Beheben des Problems</span><span class="sxs-lookup"><span data-stu-id="262b6-125">How to fix the problem</span></span>

<span data-ttu-id="262b6-126">Verschieben Sie Prozesse, die erhebliche Ressourcen beanspruchen, auf ein separates Back-End.</span><span class="sxs-lookup"><span data-stu-id="262b6-126">Move processes that consume significant resources to a separate back end.</span></span> 

<span data-ttu-id="262b6-127">Bei diesem Ansatz reiht das Front-End ressourcenintensive Aufgaben in eine Nachrichtenwarteschlange ein.</span><span class="sxs-lookup"><span data-stu-id="262b6-127">With this approach, the front end puts resource-intensive tasks onto a message queue.</span></span> <span data-ttu-id="262b6-128">Das Back-End wählt die Aufgaben zur asynchronen Verarbeitung aus.</span><span class="sxs-lookup"><span data-stu-id="262b6-128">The back end picks up the tasks for asynchronous processing.</span></span> <span data-ttu-id="262b6-129">Die Warteschlange fungiert auch als Lastenausgleich, da sie Anforderungen für das Back-End puffert.</span><span class="sxs-lookup"><span data-stu-id="262b6-129">The queue also acts as a load leveler, buffering requests for the back end.</span></span> <span data-ttu-id="262b6-130">Wenn die Warteschlange zu lang wird, können Sie die automatische Skalierung konfigurieren, um das Back-End horizontal hochzuskalieren.</span><span class="sxs-lookup"><span data-stu-id="262b6-130">If the queue length becomes too long, you can configure autoscaling to scale out the back end.</span></span>

<span data-ttu-id="262b6-131">Hier sehen Sie eine überarbeitete Version des obigen Codes.</span><span class="sxs-lookup"><span data-stu-id="262b6-131">Here is a revised version of the previous code.</span></span> <span data-ttu-id="262b6-132">In dieser Version reiht die `Post`-Methode eine Nachricht in eine Service Bus-Warteschlange ein.</span><span class="sxs-lookup"><span data-stu-id="262b6-132">In this version, the `Post` method puts a message on a Service Bus queue.</span></span> 

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

<span data-ttu-id="262b6-133">Das Back-End pullt Nachrichten aus der Service Bus-Warteschlange und führt die Verarbeitung aus.</span><span class="sxs-lookup"><span data-stu-id="262b6-133">The back end pulls messages from the Service Bus queue and does the processing.</span></span>

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

## <a name="considerations"></a><span data-ttu-id="262b6-134">Überlegungen</span><span class="sxs-lookup"><span data-stu-id="262b6-134">Considerations</span></span>

- <span data-ttu-id="262b6-135">Dieser Ansatz erhöht die Komplexität der Anwendung zusätzlich.</span><span class="sxs-lookup"><span data-stu-id="262b6-135">This approach adds some additional complexity to the application.</span></span> <span data-ttu-id="262b6-136">Sie müssen das Einreihen in die und Entfernen aus der Warteschlange sicher behandeln, damit im Fall eines Fehlers keine Anforderungen verloren gehen.</span><span class="sxs-lookup"><span data-stu-id="262b6-136">You must handle queuing and dequeuing safely to avoid losing requests in the event of a failure.</span></span>
- <span data-ttu-id="262b6-137">Die Anwendung ist von einem zusätzlichen Dienst für die Nachrichtenwarteschlange abhängig.</span><span class="sxs-lookup"><span data-stu-id="262b6-137">The application takes a dependency on an additional service for the message queue.</span></span>
- <span data-ttu-id="262b6-138">Die Verarbeitungsumgebung muss ausreichend skalierbar sein, um die erwartete Arbeitsauslastung zu bewältigen und die erforderlichen Durchsatzziele zu erfüllen.</span><span class="sxs-lookup"><span data-stu-id="262b6-138">The processing environment must be sufficiently scalable to handle the expected workload and meet the required throughput targets.</span></span>
- <span data-ttu-id="262b6-139">Dieser Ansatz sollte zwar die allgemeine Reaktionsfähigkeit verbessern, die Ausführung der auf das Back-End verschobenen Aufgaben kann jedoch mehr Zeit in Anspruch nehmen.</span><span class="sxs-lookup"><span data-stu-id="262b6-139">While this approach should improve overall responsiveness, the tasks that are moved to the back end may take longer to complete.</span></span> 

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="262b6-140">Erkennen des Problems</span><span class="sxs-lookup"><span data-stu-id="262b6-140">How to detect the problem</span></span>

<span data-ttu-id="262b6-141">Zu den Symptomen eines ausgelasteten Front-Ends zählt die lange Wartezeit bei der Ausführung ressourcenintensiver Aufgaben.</span><span class="sxs-lookup"><span data-stu-id="262b6-141">Symptoms of a busy front end include high latency when resource-intensive tasks are being performed.</span></span> <span data-ttu-id="262b6-142">Endbenutzer berichten vermutlich von längeren Antwortzeiten oder Fehlern aufgrund von Diensten, bei denen ein Timeout auftritt. In diesen Fällen können auch Fehler vom Typ „HTTP 500 (interner Server)“ oder „HTTP 503 (Dienst nicht verfügbar)“ zurückgegeben werden.</span><span class="sxs-lookup"><span data-stu-id="262b6-142">End users are likely to report extended response times or failures caused by services timing out. These failures could also return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="262b6-143">Überprüfen Sie die Ereignisprotokolle für den Webserver. Sie enthalten wahrscheinlich ausführlichere Informationen zu den Ursachen und Umständen der Fehler.</span><span class="sxs-lookup"><span data-stu-id="262b6-143">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="262b6-144">Sie können die folgenden Schritte ausführen, um dieses Problem zu identifizieren:</span><span class="sxs-lookup"><span data-stu-id="262b6-144">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="262b6-145">Führen Sie eine Prozessüberwachung des Produktionssystems durch, um Punkte zu identifizieren, an denen Antwortzeiten verlangsamt werden.</span><span class="sxs-lookup"><span data-stu-id="262b6-145">Perform process monitoring of the production system, to identify points when response times slow down.</span></span>
2. <span data-ttu-id="262b6-146">Untersuchen Sie die an diesen Punkten erfassten Telemetriedaten, um die ausgeführte Kombination von Vorgängen und die verwendeten Ressourcen zu ermitteln.</span><span class="sxs-lookup"><span data-stu-id="262b6-146">Examine the telemetry data captured at these points to determine the mix of operations being performed and the resources being used.</span></span> 
3. <span data-ttu-id="262b6-147">Suchen Sie nach Korrelationen zwischen langen Antwortzeiten und der Anzahl sowie den Kombinationen von Vorgängen, die zu diesen Zeitpunkten ausgeführt wurden.</span><span class="sxs-lookup"><span data-stu-id="262b6-147">Find any correlations between poor response times and the volumes and combinations of operations that were happening at those times.</span></span>
4. <span data-ttu-id="262b6-148">Führen Sie einen Auslastungstest für jeden „verdächtigen“ Vorgang aus, um herauszufinden, welche Vorgänge Ressourcen verbrauchen und andere Vorgänge blockieren.</span><span class="sxs-lookup"><span data-stu-id="262b6-148">Load test each suspected operation to identify which operations are consuming resources and starving other operations.</span></span> 
5. <span data-ttu-id="262b6-149">Überprüfen Sie den Quellcode für diese Vorgänge, um zu ermitteln, weshalb sie einen übermäßigen Ressourcenverbrauch verursachen könnten.</span><span class="sxs-lookup"><span data-stu-id="262b6-149">Review the source code for those operations to determine why they might cause excessive resource consumption.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="262b6-150">Beispieldiagnose</span><span class="sxs-lookup"><span data-stu-id="262b6-150">Example diagnosis</span></span> 

<span data-ttu-id="262b6-151">In den folgenden Abschnitten werden diese Schritte auf die zuvor beschriebene Beispielanwendung angewendet.</span><span class="sxs-lookup"><span data-stu-id="262b6-151">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slowdown"></a><span data-ttu-id="262b6-152">Identifizieren der Punkte, an denen eine Verlangsamung auftritt</span><span class="sxs-lookup"><span data-stu-id="262b6-152">Identify points of slowdown</span></span>

<span data-ttu-id="262b6-153">Instrumentieren Sie jede Methode, um die Dauer der einzelnen Anforderungen und die von ihnen verbrauchten Ressourcen nachzuverfolgen.</span><span class="sxs-lookup"><span data-stu-id="262b6-153">Instrument each method to track the duration and resources consumed by each request.</span></span> <span data-ttu-id="262b6-154">Überwachen Sie die Anwendung anschließend in der Produktionsumgebung.</span><span class="sxs-lookup"><span data-stu-id="262b6-154">Then monitor the application in production.</span></span> <span data-ttu-id="262b6-155">Dies kann Ihnen einen allgemeinen Überblick darüber bieten, wie Anforderungen miteinander um Ressourcen konkurrieren.</span><span class="sxs-lookup"><span data-stu-id="262b6-155">This can provide an overall view of how requests compete with each other.</span></span> <span data-ttu-id="262b6-156">In Zeiten hoher Belastung beeinträchtigen ressourcenintensive Anforderungen mit langer Ausführungszeit wahrscheinlich andere Vorgänge. Dieses Verhalten kann durch eine Überwachung des Systems und der Leistungsabnahme beobachtet werden.</span><span class="sxs-lookup"><span data-stu-id="262b6-156">During periods of stress, slow-running resource-hungry requests will likely impact other operations, and this behavior can be observed by monitoring the system and noting the drop off in performance.</span></span>

<span data-ttu-id="262b6-157">Der folgende Screenshot zeigt ein Überwachungsdashboard.</span><span class="sxs-lookup"><span data-stu-id="262b6-157">The following image shows a monitoring dashboard.</span></span> <span data-ttu-id="262b6-158">(Wir haben [AppDynamics] für unsere Tests verwendet.) Zu Beginn ist die Auslastung des Systems gering.</span><span class="sxs-lookup"><span data-stu-id="262b6-158">(We used [AppDynamics] for our tests.) Initially, the system has light load.</span></span> <span data-ttu-id="262b6-159">Dann beginnen Benutzer, die `UserProfile`-GET-Methode anzufordern.</span><span class="sxs-lookup"><span data-stu-id="262b6-159">Then users start requesting the `UserProfile` GET method.</span></span> <span data-ttu-id="262b6-160">Die Leistung ist einigermaßen gut, bis andere Benutzer Anforderungen an die `WorkInFrontEnd`-POST-Methode ausgeben.</span><span class="sxs-lookup"><span data-stu-id="262b6-160">The performance is reasonably good until other users start issuing requests to the `WorkInFrontEnd` POST method.</span></span> <span data-ttu-id="262b6-161">An diesem Punkt nehmen die Antwortzeiten drastisch zu (erster Pfeil).</span><span class="sxs-lookup"><span data-stu-id="262b6-161">At that point, response times increase dramatically (first arrow).</span></span> <span data-ttu-id="262b6-162">Die Antwortzeiten verbessern sich erst, nachdem die Anzahl von Anforderungen an den `WorkInFrontEnd`-Controller abgenommen hat (zweiter Pfeil).</span><span class="sxs-lookup"><span data-stu-id="262b6-162">Response times only improve after the volume of requests to the `WorkInFrontEnd` controller diminishes (second arrow).</span></span>

![Der AppDynamics-Bereich für die Geschäftstransaktionen zeigt die Auswirkungen der Antwortzeiten aller Anforderungen, wenn der WorkInFrontEnd-Controller verwendet wird.][AppDynamics-Transactions-Front-End-Requests]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="262b6-164">Untersuchen der Telemetriedaten und Ermitteln von Korrelationen</span><span class="sxs-lookup"><span data-stu-id="262b6-164">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="262b6-165">Die nächste Abbildung zeigt einige der Metriken, die zum Überwachen der Ressourcenverwendung im gleichen Zeitintervall gesammelt wurden.</span><span class="sxs-lookup"><span data-stu-id="262b6-165">The next image shows some of the metrics gathered to monitor resource utilization during the same interval.</span></span> <span data-ttu-id="262b6-166">Zunächst greifen nur wenig Benutzer auf das System zu.</span><span class="sxs-lookup"><span data-stu-id="262b6-166">At first, few users are accessing the system.</span></span> <span data-ttu-id="262b6-167">Sobald weitere Benutzer eine Verbindung herstellen, steigt die CPU-Auslastung erheblich (100 %).</span><span class="sxs-lookup"><span data-stu-id="262b6-167">As more users connect, CPU utilization becomes very high (100%).</span></span> <span data-ttu-id="262b6-168">Beachten Sie auch, dass die Netzwerk-E/A-Rate bei der Zunahme der CPU-Auslastung anfänglich steigt.</span><span class="sxs-lookup"><span data-stu-id="262b6-168">Also notice that the network I/O rate initially goes up as CPU usage rises.</span></span> <span data-ttu-id="262b6-169">Nachdem die CPU-Auslastung den Höchstpunkt erreicht hat, nimmt die Netzwerk-E/A-Rate aber sogar ab.</span><span class="sxs-lookup"><span data-stu-id="262b6-169">But once CPU usage peaks, network I/O actually goes down.</span></span> <span data-ttu-id="262b6-170">Dies liegt daran, dass das System nur eine relativ kleine Anzahl von Anforderungen verarbeiten kann, sobald die CPU voll ausgelastet ist.</span><span class="sxs-lookup"><span data-stu-id="262b6-170">That's because the system can only handle a relatively small number of requests once the CPU is at capacity.</span></span> <span data-ttu-id="262b6-171">Wenn Benutzer die Verbindung trennen, nimmt die CPU-Auslastung ab.</span><span class="sxs-lookup"><span data-stu-id="262b6-171">As users disconnect, the CPU load tails off.</span></span>

![AppDynamics-Metriken zur CPU- und Netzwerkauslastung][AppDynamics-Metrics-Front-End-Requests]

<span data-ttu-id="262b6-173">An diesem Punkt ist die `Post`-Methode im `WorkInFrontEnd`-Controller anscheinend ein erstklassiger Kandidat für eine genauere Prüfung.</span><span class="sxs-lookup"><span data-stu-id="262b6-173">At this point, it appears the `Post` method in the `WorkInFrontEnd` controller is a prime candidate for closer examination.</span></span> <span data-ttu-id="262b6-174">Zur Bestätigung dieser Hypothese sind weitere Schritte in einer kontrollierten Umgebung erforderlich.</span><span class="sxs-lookup"><span data-stu-id="262b6-174">Further work in a controlled environment is needed to confirm the hypothesis.</span></span>

### <a name="perform-load-testing"></a><span data-ttu-id="262b6-175">Durchführen von Auslastungstests</span><span class="sxs-lookup"><span data-stu-id="262b6-175">Perform load testing</span></span> 

<span data-ttu-id="262b6-176">Der nächste Schritt ist die Ausführung von Tests in einer kontrollierten Umgebung.</span><span class="sxs-lookup"><span data-stu-id="262b6-176">The next step is to perform tests in a controlled environment.</span></span> <span data-ttu-id="262b6-177">Führen Sie beispielsweise eine Reihe von Auslastungstests durch, bei denen jede Anforderung nacheinander einbezogen und dann ausgelassen wird, um die Auswirkungen anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="262b6-177">For example, run a series of load tests that include and then omit each request in turn to see the effects.</span></span>

<span data-ttu-id="262b6-178">Das folgende Diagramm zeigt die Ergebnisse eines Auslastungstests für eine identische Bereitstellung des in den vorherigen Tests verwendeten Clouddiensts.</span><span class="sxs-lookup"><span data-stu-id="262b6-178">The graph below shows the results of a load test performed against an identical deployment of the cloud service used in the previous tests.</span></span> <span data-ttu-id="262b6-179">Beim Test wurden eine konstante Last von 500 Benutzern, die den `Get`-Vorgang im `UserProfile`-Controller ausführen, sowie eine schrittweise Last von Benutzern, die den `Post`-Vorgang im `WorkInFrontEnd`-Controller ausführen, verwendet.</span><span class="sxs-lookup"><span data-stu-id="262b6-179">The test used a constant load of 500 users performing the `Get` operation in the `UserProfile` controller, along with a step load of users performing the `Post` operation in the `WorkInFrontEnd` controller.</span></span> 

![Anfängliche Testergebnisse für den WorkInFrontEnd-Controller][Initial-Load-Test-Results-Front-End]

<span data-ttu-id="262b6-181">Anfangs beträgt die schrittweise Last 0, d. h. nur die aktiven Benutzer führen die `UserProfile`-Anforderungen aus.</span><span class="sxs-lookup"><span data-stu-id="262b6-181">Initially, the step load is 0, so the only active users are performing the `UserProfile` requests.</span></span> <span data-ttu-id="262b6-182">Das System kann auf ca. 500 Anforderungen pro Sekunde reagieren.</span><span class="sxs-lookup"><span data-stu-id="262b6-182">The system is able to respond to approximately 500 requests per second.</span></span> <span data-ttu-id="262b6-183">Nach 60 Sekunden beginnt eine Last von 100 zusätzlichen Benutzern, POST-Anforderungen an den `WorkInFrontEnd`-Controller zu senden.</span><span class="sxs-lookup"><span data-stu-id="262b6-183">After 60 seconds, a load of 100 additional users starts sending POST requests to the `WorkInFrontEnd` controller.</span></span> <span data-ttu-id="262b6-184">Die an den `UserProfile`-Controller gesendete Arbeitsauslastung sinkt nahezu sofort auf ungefähr 150 Anforderungen pro Sekunde.</span><span class="sxs-lookup"><span data-stu-id="262b6-184">Almost immediately, the workload sent to the `UserProfile` controller drops to about 150 requests per second.</span></span> <span data-ttu-id="262b6-185">Dies ist auf die Funktionsweise des Auslastungstests zurückzuführen.</span><span class="sxs-lookup"><span data-stu-id="262b6-185">This is due to the way the load-test runner functions.</span></span> <span data-ttu-id="262b6-186">Er wartet vor dem Senden der nächsten Anforderung auf eine Antwort. Je länger es dauert, eine Antwort zu empfangen, desto niedriger ist folglich die Anforderungsrate.</span><span class="sxs-lookup"><span data-stu-id="262b6-186">It waits for a response before sending the next request, so the longer it takes to receive a response, the lower the request rate.</span></span>

<span data-ttu-id="262b6-187">Wenn weitere Benutzer POST-Anforderungen an den `WorkInFrontEnd`-Controller senden, nimmt die Antwortrate des `UserProfile`-Controllers weiter ab.</span><span class="sxs-lookup"><span data-stu-id="262b6-187">As more users send POST requests to the `WorkInFrontEnd` controller, the response rate of the `UserProfile` controller continues to drop.</span></span> <span data-ttu-id="262b6-188">Beachten Sie jedoch, dass die Anzahl der vom `WorkInFrontEnd`-Controller verarbeiteten Anforderungen relativ konstant bleibt.</span><span class="sxs-lookup"><span data-stu-id="262b6-188">But note that the volume of requests handled by the `WorkInFrontEnd`controller remains relatively constant.</span></span> <span data-ttu-id="262b6-189">Die Sättigung des Systems wird deutlich, sobald die Gesamtrate beider Anforderungen einen stabilen, aber niedrigen Grenzwert erreicht.</span><span class="sxs-lookup"><span data-stu-id="262b6-189">The saturation of the system becomes apparent as the overall rate of both requests tends towards a steady but low limit.</span></span>


### <a name="review-the-source-code"></a><span data-ttu-id="262b6-190">Überprüfen des Quellcodes</span><span class="sxs-lookup"><span data-stu-id="262b6-190">Review the source code</span></span>

<span data-ttu-id="262b6-191">Der letzte Schritt besteht darin, den Quellcode zu überprüfen.</span><span class="sxs-lookup"><span data-stu-id="262b6-191">The final step is to look at the source code.</span></span> <span data-ttu-id="262b6-192">Das Entwicklungsteam wusste, dass die `Post`-Methode viel Zeit in Anspruch nehmen könnte, und hat in der ursprünglichen Implementierung daher einen separaten Thread verwendet.</span><span class="sxs-lookup"><span data-stu-id="262b6-192">The development team was aware that the `Post` method could take a considerable amount of time, which is why the original implementation used a separate thread.</span></span> <span data-ttu-id="262b6-193">Dadurch wurde das unmittelbare Problem gelöst, da die `Post`-Methode nicht durch das Warten auf den Abschluss einer Aufgabe mit langer Ausführungsdauer blockiert wurde.</span><span class="sxs-lookup"><span data-stu-id="262b6-193">That solved the immediate problem, because the `Post` method did not block waiting for a long-running task to complete.</span></span>

<span data-ttu-id="262b6-194">Die von dieser Methode ausgeführte Arbeit verbraucht jedoch nach wie vor CPU-Zeit, Arbeitsspeicher und andere Ressourcen.</span><span class="sxs-lookup"><span data-stu-id="262b6-194">However, the work performed by this method still consumes CPU, memory, and other resources.</span></span> <span data-ttu-id="262b6-195">Wenn die asynchrone Ausführung dieses Prozesses ermöglicht wird, kann die Leistung beeinträchtigt werden, da Benutzer auf unkontrollierte Weise eine große Anzahl dieser Vorgänge gleichzeitig auslösen können.</span><span class="sxs-lookup"><span data-stu-id="262b6-195">Enabling this process to run asynchronously might actually damage performance, as users can trigger a large number of these operations simultaneously, in an uncontrolled manner.</span></span> <span data-ttu-id="262b6-196">Die Anzahl von Threads, die von einem Server ausgeführt werden können, ist begrenzt.</span><span class="sxs-lookup"><span data-stu-id="262b6-196">There is a limit to the number of threads that a server can run.</span></span> <span data-ttu-id="262b6-197">Wird dieser Grenzwert überschritten, wird bei dem Versuch, einen neuen Thread zu starten, wahrscheinlich eine Ausnahme in der Anwendung ausgelöst.</span><span class="sxs-lookup"><span data-stu-id="262b6-197">Past this limit, the application is likely to get an exception when it tries to start a new thread.</span></span>

> [!NOTE]
> <span data-ttu-id="262b6-198">Dies bedeutet nicht, dass Sie asynchrone Vorgänge vermeiden sollten.</span><span class="sxs-lookup"><span data-stu-id="262b6-198">This doesn't mean you should avoid asynchronous operations.</span></span> <span data-ttu-id="262b6-199">Das Ausführen eines asynchronen Wartevorgangs für einen Netzwerkaufruf ist eine empfohlene Vorgehensweise.</span><span class="sxs-lookup"><span data-stu-id="262b6-199">Performing an asynchronous await on a network call is a recommended practice.</span></span> <span data-ttu-id="262b6-200">(Siehe das Antimuster für [synchrone E/A][sync-io].) Hier besteht das Problem darin, dass CPU-intensive Vorgänge in einem anderen Thread erzeugt wurden.</span><span class="sxs-lookup"><span data-stu-id="262b6-200">(See the [Synchronous I/O][sync-io] antipattern.) The problem here is that CPU-intensive work was spawned on another thread.</span></span> 

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="262b6-201">Implementieren der Lösung und Überprüfen des Ergebnisses</span><span class="sxs-lookup"><span data-stu-id="262b6-201">Implement the solution and verify the result</span></span>

<span data-ttu-id="262b6-202">Die folgende Abbildung zeigt die Leistungsüberwachung nach dem Implementieren der Lösung.</span><span class="sxs-lookup"><span data-stu-id="262b6-202">The following image shows performance monitoring after the solution was implemented.</span></span> <span data-ttu-id="262b6-203">Die Auslastung war mit der zuvor gezeigten Auslastung vergleichbar, die Antwortzeiten für den `UserProfile`-Controller sind jedoch deutlich schneller.</span><span class="sxs-lookup"><span data-stu-id="262b6-203">The load was similar to that shown earlier, but the response times for the `UserProfile` controller are now much faster.</span></span> <span data-ttu-id="262b6-204">Die Anzahl von Anforderungen hat über den gleichen Zeitraum von 2.759 auf 23.565 zugenommen.</span><span class="sxs-lookup"><span data-stu-id="262b6-204">The volume of requests increased over the same duration, from 2,759 to 23,565.</span></span> 

![Der AppDynamics-Bereich für die Geschäftstransaktionen zeigt die Auswirkungen der Antwortzeiten aller Anforderungen, wenn der WorkInBackground-Controller verwendet wird.][AppDynamics-Transactions-Background-Requests]

<span data-ttu-id="262b6-206">Beachten Sie, dass der `WorkInBackground`-Controller auch eine deutlich größere Anzahl von Anforderungen behandelt hat.</span><span class="sxs-lookup"><span data-stu-id="262b6-206">Note that the `WorkInBackground` controller also handled a much larger volume of requests.</span></span> <span data-ttu-id="262b6-207">In diesem Fall ist allerdings kein direkter Vergleich möglich, da sich die vom Controller ausgeführte Arbeit erheblich vom ursprünglichen Code unterscheidet.</span><span class="sxs-lookup"><span data-stu-id="262b6-207">However, you can't make a direct comparison in this case, because the work being performed in this controller is very different from the original code.</span></span> <span data-ttu-id="262b6-208">Die neue Version reiht eine Anforderung einfach in die Warteschlange ein, anstatt eine zeitaufwändige Berechnung durchzuführen.</span><span class="sxs-lookup"><span data-stu-id="262b6-208">The new version simply queues a request, rather than performing a time consuming calculation.</span></span> <span data-ttu-id="262b6-209">Entscheidend ist, dass diese Methode bei hoher Last nicht mehr die Leistung des gesamten Systems beeinträchtigt.</span><span class="sxs-lookup"><span data-stu-id="262b6-209">The main point is that this method no longer drags down the entire system under load.</span></span>

<span data-ttu-id="262b6-210">Die verbesserte Leistung ist auch an der CPU- und Netzwerkauslastung zu erkennen.</span><span class="sxs-lookup"><span data-stu-id="262b6-210">CPU and network utilization also show the improved performance.</span></span> <span data-ttu-id="262b6-211">Die CPU-Auslastung hat nie 100 % erreicht. Die Anzahl verarbeiteter Netzwerkanforderungen war weitaus höher als zuvor und ist erst bei der Abnahme der Arbeitsauslastung gesunken.</span><span class="sxs-lookup"><span data-stu-id="262b6-211">The CPU utilization never reached 100%, and the volume of handled network requests was far greater than earlier, and did not tail off until the workload dropped.</span></span>

![AppDynamics-Metriken zur CPU- und Netzwerkauslastung für den WorkInBackground-Controller][AppDynamics-Metrics-Background-Requests]

<span data-ttu-id="262b6-213">Das folgende Diagramm zeigt die Ergebnisse eines Auslastungstests.</span><span class="sxs-lookup"><span data-stu-id="262b6-213">The following graph shows the results of a load test.</span></span> <span data-ttu-id="262b6-214">Die Gesamtanzahl verarbeiteter Anforderungen hat im Vergleich zu den früheren Tests erheblich zugenommen.</span><span class="sxs-lookup"><span data-stu-id="262b6-214">The overall volume of requests serviced is greatly improved compared to the the earlier tests.</span></span>

![Ergebnisse des Auslastungstests für den BackgroundImageProcessing-Controller][Load-Test-Results-Background]

## <a name="related-guidance"></a><span data-ttu-id="262b6-216">Verwandte Anweisungen</span><span class="sxs-lookup"><span data-stu-id="262b6-216">Related guidance</span></span>

- <span data-ttu-id="262b6-217">[Autoscaling best practices][autoscaling] (Bewährte Methoden für die automatische Skalierung)</span><span class="sxs-lookup"><span data-stu-id="262b6-217">[Autoscaling best practices][autoscaling]</span></span>
- <span data-ttu-id="262b6-218">[Background jobs best practices][background-jobs] (Bewährte Methoden für Hintergrundaufträge)</span><span class="sxs-lookup"><span data-stu-id="262b6-218">[Background jobs best practices][background-jobs]</span></span>
- <span data-ttu-id="262b6-219">[Queue-Based Load Leveling pattern][load-leveling] (Warteschlangenbasiertes Lastenausgleichsmuster)</span><span class="sxs-lookup"><span data-stu-id="262b6-219">[Queue-Based Load Leveling pattern][load-leveling]</span></span>
- <span data-ttu-id="262b6-220">[Web Queue Worker architecture style][web-queue-worker] (Webwarteschlangen-Workerarchitektur)</span><span class="sxs-lookup"><span data-stu-id="262b6-220">[Web Queue Worker architecture style][web-queue-worker]</span></span>

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


