---
title: "Antimuster „Synchrone E/A-Vorgänge“"
description: "Ein Blockieren des aufrufenden Threads, während ein E/A-Vorgang abgeschlossen wird, kann die Leistung senken und sich auf die vertikale Skalierbarkeit auswirken."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: d5b3635565c6b71ef7716f54ee8cccc76093c3a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="synchronous-io-antipattern"></a><span data-ttu-id="33d72-103">Antimuster „Synchrone E/A-Vorgänge“</span><span class="sxs-lookup"><span data-stu-id="33d72-103">Synchronous I/O antipattern</span></span>

<span data-ttu-id="33d72-104">Ein Blockieren des aufrufenden Threads, während ein E/A-Vorgang abgeschlossen wird, kann die Leistung senken und sich auf die vertikale Skalierbarkeit auswirken.</span><span class="sxs-lookup"><span data-stu-id="33d72-104">Blocking the calling thread while I/O completes can reduce performance and affect vertical scalability.</span></span>

## <a name="problem-description"></a><span data-ttu-id="33d72-105">Problembeschreibung</span><span class="sxs-lookup"><span data-stu-id="33d72-105">Problem description</span></span>

<span data-ttu-id="33d72-106">Ein synchroner E/A-Vorgang blockiert den aufrufenden Thread, während der E/A-Vorgang abgeschlossen wird.</span><span class="sxs-lookup"><span data-stu-id="33d72-106">A synchronous I/O operation blocks the calling thread while the I/O completes.</span></span> <span data-ttu-id="33d72-107">Der aufrufende Thread wird in einen Wartezustand versetzt und kann in diesem Intervall keine sinnvollen Vorgänge ausführen, wodurch Verarbeitungsressourcen verschwendet werden.</span><span class="sxs-lookup"><span data-stu-id="33d72-107">The calling thread enters a wait state and is unable to perform useful work during this interval, wasting processing resources.</span></span>

<span data-ttu-id="33d72-108">Häufige Beispiele für E/A-Vorgänge sind etwa folgende:</span><span class="sxs-lookup"><span data-stu-id="33d72-108">Common examples of I/O include:</span></span>

- <span data-ttu-id="33d72-109">Abrufen oder dauerhaftes Speichern von Daten in einer Datenbank oder einer anderen Art persistentem Speicher</span><span class="sxs-lookup"><span data-stu-id="33d72-109">Retrieving or persisting data to a database or any type of persistent storage.</span></span>
- <span data-ttu-id="33d72-110">Senden einer Anforderung an einen Webdienst</span><span class="sxs-lookup"><span data-stu-id="33d72-110">Sending a request to a web service.</span></span>
- <span data-ttu-id="33d72-111">Übermitteln einer Nachricht oder Abrufen einer Nachricht aus einer Warteschlange</span><span class="sxs-lookup"><span data-stu-id="33d72-111">Posting a message or retrieving a message from a queue.</span></span>
- <span data-ttu-id="33d72-112">Schreiben in eine lokale Datei oder Lesen aus einer lokalen Datei</span><span class="sxs-lookup"><span data-stu-id="33d72-112">Writing to or reading from a local file.</span></span>

<span data-ttu-id="33d72-113">Dieses Antimuster tritt üblicherweise aus folgenden Gründen auf:</span><span class="sxs-lookup"><span data-stu-id="33d72-113">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="33d72-114">Es ist anscheinend die intuitivste Art, einen Vorgang auszuführen.</span><span class="sxs-lookup"><span data-stu-id="33d72-114">It appears to be the most intuitive way to perform an operation.</span></span> 
- <span data-ttu-id="33d72-115">Die Anwendung erfordert eine Antwort aus einer Anforderung.</span><span class="sxs-lookup"><span data-stu-id="33d72-115">The application requires a response from a request.</span></span>
- <span data-ttu-id="33d72-116">Die Anwendung verwendet eine Bibliothek, die nur synchrone Methoden für E/A-Vorgänge bereitstellt.</span><span class="sxs-lookup"><span data-stu-id="33d72-116">The application uses a library that only provides synchronous methods for I/O.</span></span> 
- <span data-ttu-id="33d72-117">Eine externe Bibliothek führt intern synchrone E/A-Vorgänge aus.</span><span class="sxs-lookup"><span data-stu-id="33d72-117">An external library performs synchronous I/O operations internally.</span></span> <span data-ttu-id="33d72-118">Ein einziger synchroner E/A-Aufruf kann eine vollständige Aufrufkette blockieren.</span><span class="sxs-lookup"><span data-stu-id="33d72-118">A single synchronous I/O call can block an entire call chain.</span></span>

<span data-ttu-id="33d72-119">Der folgende Code lädt eine Datei in einen Azure-Blobspeicher hoch.</span><span class="sxs-lookup"><span data-stu-id="33d72-119">The following code uploads a file to Azure blob storage.</span></span> <span data-ttu-id="33d72-120">Es gibt zwei Stellen, an denen der Code blockiert wird, weil er auf die synchrone E/A-Verarbeitung wartet: die `CreateIfNotExists`-Methode und die `UploadFromStream`-Methode.</span><span class="sxs-lookup"><span data-stu-id="33d72-120">There are two places where the code blocks waiting for synchronous I/O, the `CreateIfNotExists` method and the `UploadFromStream` method.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

container.CreateIfNotExists();
var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    blockBlob.UploadFromStream(fileStream);
}
```

<span data-ttu-id="33d72-121">Hier sehen Sie ein Beispiel für das Warten auf die Antwort eines externen Diensts.</span><span class="sxs-lookup"><span data-stu-id="33d72-121">Here's an example of waiting for a response from an external service.</span></span> <span data-ttu-id="33d72-122">Die `GetUserProfile`-Methode ruft einen Remotedienst auf, der ein `UserProfile` zurückgibt.</span><span class="sxs-lookup"><span data-stu-id="33d72-122">The `GetUserProfile` method calls a remote service that returns a `UserProfile`.</span></span>

```csharp
public interface IUserProfileService
{
    UserProfile GetUserProfile();
}

public class SyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public SyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is a synchronous method that calls the synchronous GetUserProfile method.
    public UserProfile GetUserProfile()
    {
        return _userProfileService.GetUserProfile();
    }
}
```

<span data-ttu-id="33d72-123">Den vollständigen Code für beide Beispiele finden Sie [hier][sample-app].</span><span class="sxs-lookup"><span data-stu-id="33d72-123">You can find the complete code for both of these examples [here][sample-app].</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="33d72-124">Beheben des Problems</span><span class="sxs-lookup"><span data-stu-id="33d72-124">How to fix the problem</span></span>

<span data-ttu-id="33d72-125">Ersetzen Sie synchrone E/A-Vorgänge durch asynchrone Vorgänge.</span><span class="sxs-lookup"><span data-stu-id="33d72-125">Replace synchronous I/O operations with asynchronous operations.</span></span> <span data-ttu-id="33d72-126">Dadurch wird der aktuelle Thread wieder frei und kann weiterhin sinnvolle Tasks ausführen, anstatt zu blockieren. So lässt sich die Auslastung der Computeressourcen verbessern.</span><span class="sxs-lookup"><span data-stu-id="33d72-126">This frees the current thread to continue performing meaningful work rather than blocking, and helps improve the utilization of compute resources.</span></span> <span data-ttu-id="33d72-127">Die asynchrone Ausführung von E/A-Vorgängen ist insbesondere dann effizient, wenn ein unerwarteter Anstieg der Anforderungen von Clientanwendungen verarbeitet werden muss.</span><span class="sxs-lookup"><span data-stu-id="33d72-127">Performing I/O asynchronously is particularly efficient for handling an unexpected surge in requests from client applications.</span></span> 

<span data-ttu-id="33d72-128">Viele Bibliotheken bieten sowohl synchrone als auch asynchrone Versionen von Methoden.</span><span class="sxs-lookup"><span data-stu-id="33d72-128">Many libraries provide both synchronous and asynchronous versions of methods.</span></span> <span data-ttu-id="33d72-129">Verwenden Sie nach Möglichkeit die asynchronen Versionen.</span><span class="sxs-lookup"><span data-stu-id="33d72-129">Whenever possible, use the asynchronous versions.</span></span> <span data-ttu-id="33d72-130">Hier ist die asynchrone Version des vorherigen Beispiels, das eine Datei in einen Azure-Blobspeicher hochlädt.</span><span class="sxs-lookup"><span data-stu-id="33d72-130">Here is the asynchronous version of the previous example that uploads a file to Azure blob storage.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

await container.CreateIfNotExistsAsync();

var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

<span data-ttu-id="33d72-131">Der `await`-Operator gibt die Steuerung an die aufrufende Umgebung zurück, während der asynchrone Vorgang ausgeführt wird.</span><span class="sxs-lookup"><span data-stu-id="33d72-131">The `await` operator returns control to the calling environment while the asynchronous operation is performed.</span></span> <span data-ttu-id="33d72-132">Der Code nach dieser Anweisung fungiert als Fortsetzung, die ausgeführt wird, wenn der asynchrone Vorgang abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="33d72-132">The code after this statement acts as a continuation that runs when the asynchronous operation has completed.</span></span>

<span data-ttu-id="33d72-133">Ein sorgfältig entworfener Dienst sollte auch asynchrone Vorgänge bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="33d72-133">A well designed service should also provide asynchronous operations.</span></span> <span data-ttu-id="33d72-134">Hier sehen Sie eine asynchrone Version des Webdiensts, der Benutzerprofile zurückgibt.</span><span class="sxs-lookup"><span data-stu-id="33d72-134">Here is an asynchronous version of the web service that returns user profiles.</span></span> <span data-ttu-id="33d72-135">Die `GetUserProfileAsync`-Methode benötigt eine asynchrone Version des Benutzerprofildiensts.</span><span class="sxs-lookup"><span data-stu-id="33d72-135">The `GetUserProfileAsync` method depends on having an asynchronous version of the User Profile service.</span></span>

```csharp
public interface IUserProfileService
{
    Task<UserProfile> GetUserProfileAsync();
}

public class AsyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public AsyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is an synchronous method that calls the Task based GetUserProfileAsync method.
    public Task<UserProfile> GetUserProfileAsync()
    {
        return _userProfileService.GetUserProfileAsync();
    }
}
```

<span data-ttu-id="33d72-136">Bei Bibliotheken, die keine asynchronen Versionen von Vorgängen bereitstellen, ist es unter Umständen möglich, asynchrone Wrapper für ausgewählte synchrone Methoden zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="33d72-136">For libraries that don't provide asynchronous versions of operations, it may be possible to create asynchronous wrappers around selected synchronous methods.</span></span> <span data-ttu-id="33d72-137">Gehen Sie dabei sehr vorsichtig vor.</span><span class="sxs-lookup"><span data-stu-id="33d72-137">Follow this approach with caution.</span></span> <span data-ttu-id="33d72-138">Mit diesem Ansatz lässt sich zwar möglicherweise die Reaktionsfähigkeit in dem Thread verbessern, der den asynchronen Wrapper aufruft, aber es werden auch mehr Ressourcen verbraucht.</span><span class="sxs-lookup"><span data-stu-id="33d72-138">While it may improve responsiveness on the thread that invokes the asynchronous wrapper, it actually consumes more resources.</span></span> <span data-ttu-id="33d72-139">Möglicherweise wird ein gesonderter Thread erstellt, und durch die Synchronisierung der von diesem Thread ausgeführten Verarbeitung entsteht Overhead.</span><span class="sxs-lookup"><span data-stu-id="33d72-139">An extra thread may be created, and there is overhead associated with synchronizing the work done by this thread.</span></span> <span data-ttu-id="33d72-140">In diesem Blogbeitrag werden einige Nachteile erläutert: [Should I expose asynchronous wrappers for synchronous methods?][async-wrappers] (Sollen asynchrone Wrapper für synchrone Methoden verfügbar gemacht werden?)</span><span class="sxs-lookup"><span data-stu-id="33d72-140">Some tradeoffs are discussed in this blog post: [Should I expose asynchronous wrappers for synchronous methods?][async-wrappers]</span></span>

<span data-ttu-id="33d72-141">Hier sehen Sie ein Beispiel eines asynchronen Wrappers um eine synchrone Methode.</span><span class="sxs-lookup"><span data-stu-id="33d72-141">Here is an example of an asynchronous wrapper around a synchronous method.</span></span>

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

<span data-ttu-id="33d72-142">Jetzt kann der aufrufende Code im Wrapper warten:</span><span class="sxs-lookup"><span data-stu-id="33d72-142">Now the calling code can await on the wrapper:</span></span>

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a><span data-ttu-id="33d72-143">Überlegungen</span><span class="sxs-lookup"><span data-stu-id="33d72-143">Considerations</span></span>

- <span data-ttu-id="33d72-144">E/A-Vorgänge, die voraussichtlich sehr kurzlebig sind und keine Konflikte verursachen, können als synchrone Vorgänge leistungsfähig sein.</span><span class="sxs-lookup"><span data-stu-id="33d72-144">I/O operations that are expected to be very short lived and are unlikely to cause contention might be more performant as synchronous operations.</span></span> <span data-ttu-id="33d72-145">Ein Beispiel ist das Lesen kleiner Dateien auf einem SSD-Laufwerk.</span><span class="sxs-lookup"><span data-stu-id="33d72-145">An example might be reading small files on an SSD drive.</span></span> <span data-ttu-id="33d72-146">Der Overhead durch das Umleiten eines Tasks in einen anderen Thread und das Synchronisieren mit diesem Thread nach Abschluss des Tasks kann die Vorteile der asynchronen E/A-Verarbeitung überwiegen.</span><span class="sxs-lookup"><span data-stu-id="33d72-146">The overhead of dispatching a task to another thread, and synchronizing with that thread when the task completes, might outweigh the benefits of asynchronous I/O.</span></span> <span data-ttu-id="33d72-147">Diese Fälle sind jedoch relativ selten, und die meisten E/A-Vorgänge lassen sich asynchron ausführen.</span><span class="sxs-lookup"><span data-stu-id="33d72-147">However, these cases are relatively rare, and most I/O operations should be done asynchronously.</span></span>

- <span data-ttu-id="33d72-148">Durch Verbesserung der E/A-Leistung werden möglicherweise andere Teile des Systems zu Engpässen.</span><span class="sxs-lookup"><span data-stu-id="33d72-148">Improving I/O performance may cause other parts of the system to become bottlenecks.</span></span> <span data-ttu-id="33d72-149">Dadurch, dass die Blockierung von Threads aufgehoben wird, kann eine höhere Anzahl gleichzeitiger Anforderungen an gemeinsam genutzte Ressourcen entstehen, wodurch wiederum möglicherweise nicht genügend Ressourcen verfügbar sind oder Ressourcen gedrosselt werden müssen.</span><span class="sxs-lookup"><span data-stu-id="33d72-149">For example, unblocking threads might result in a higher volume of concurrent requests to shared resources, leading in turn to resource starvation or throttling.</span></span> <span data-ttu-id="33d72-150">Wenn dies zu einem Problem wird, müssen Sie möglicherweise Ihre Webserver horizontal hochskalieren oder Datenspeicher partitionieren, um Konflikte zu reduzieren.</span><span class="sxs-lookup"><span data-stu-id="33d72-150">If that becomes a problem, you might need to scale out the number of web servers or partition data stores to reduce contention.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="33d72-151">Erkennen des Problems</span><span class="sxs-lookup"><span data-stu-id="33d72-151">How to detect the problem</span></span>

<span data-ttu-id="33d72-152">Aus Benutzersicht reagiert die Anwendung nicht.</span><span class="sxs-lookup"><span data-stu-id="33d72-152">For users, the application may seem unresponsive or appear to hang periodically.</span></span> <span data-ttu-id="33d72-153">Bei der Anwendung können Timeoutausnahmen auftreten.</span><span class="sxs-lookup"><span data-stu-id="33d72-153">The application might fail with timeout exceptions.</span></span> <span data-ttu-id="33d72-154">In diesen Fällen können auch HTTP 500-Fehler (interner Server) zurückgegeben werden.</span><span class="sxs-lookup"><span data-stu-id="33d72-154">These failures could also return HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="33d72-155">Auf dem Server werden eingehende Clientanforderungen blockiert, bis ein Thread verfügbar wird, sodass die Anforderungswarteschlangen zu lang werden, was sich in HTTP 503-Fehlern (Dienst nicht verfügbar) manifestiert.</span><span class="sxs-lookup"><span data-stu-id="33d72-155">On the server, incoming client requests might be blocked until a thread becomes available, resulting in excessive request queue lengths, manifested as HTTP 503 (Service Unavailable) errors.</span></span>

<span data-ttu-id="33d72-156">Sie können die folgenden Schritte ausführen, um das Problem zu identifizieren:</span><span class="sxs-lookup"><span data-stu-id="33d72-156">You can perform the following steps to help identify the problem:</span></span>

1. <span data-ttu-id="33d72-157">Überwachen Sie das Produktionssystem, und ermitteln Sie, ob blockierte Workerthreads den Durchsatz einschränken.</span><span class="sxs-lookup"><span data-stu-id="33d72-157">Monitor the production system and determine whether blocked worker threads are constraining throughput.</span></span>

2. <span data-ttu-id="33d72-158">Wenn Anforderungen aufgrund einer zu geringen Anzahl von Threads blockiert werden, überprüfen Sie die Anwendung, um zu ermitteln, welche Vorgänge synchrone E/A-Vorgänge ausführen.</span><span class="sxs-lookup"><span data-stu-id="33d72-158">If requests are being blocked due to lack of threads, review the application to determine which operations may be performing I/O synchronously.</span></span>

3. <span data-ttu-id="33d72-159">Führen Sie kontrollierte Auslastungstests jedes Vorgangs durch, der synchrone E/A-Vorgänge ausführt, um herauszufinden, ob diese Vorgänge die Systemleistung beeinträchtigen.</span><span class="sxs-lookup"><span data-stu-id="33d72-159">Perform controlled load testing of each operation that is performing synchronous I/O, to find out whether those operations are affecting system performance.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="33d72-160">Beispieldiagnose</span><span class="sxs-lookup"><span data-stu-id="33d72-160">Example diagnosis</span></span>

<span data-ttu-id="33d72-161">In den folgenden Abschnitten werden diese Schritte auf die zuvor beschriebene Beispielanwendung angewendet.</span><span class="sxs-lookup"><span data-stu-id="33d72-161">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-web-server-performance"></a><span data-ttu-id="33d72-162">Überwachen der Webserverleistung</span><span class="sxs-lookup"><span data-stu-id="33d72-162">Monitor web server performance</span></span>

<span data-ttu-id="33d72-163">Bei Azure-Webanwendungen und -Webrollen lohnt es sich, die Leistung des IIS-Webservers zu überwachen.</span><span class="sxs-lookup"><span data-stu-id="33d72-163">For Azure web applications and web roles, it's worth monitoring the performance of the IIS web server.</span></span> <span data-ttu-id="33d72-164">Achten Sie besonders auf die Länge der Anforderungswarteschlange, um herauszufinden, ob Anforderungen in Zeiträumen hoher Aktivität dadurch blockiert werden, dass sie auf verfügbare Threads warten.</span><span class="sxs-lookup"><span data-stu-id="33d72-164">In particular, pay attention to the request queue length to establish whether requests are being blocked waiting for available threads during periods of high activity.</span></span> <span data-ttu-id="33d72-165">Sie können diese Informationen erfassen, indem Sie die Azure-Diagnose aktivieren.</span><span class="sxs-lookup"><span data-stu-id="33d72-165">You can gather this information by enabling Azure diagnostics.</span></span> <span data-ttu-id="33d72-166">Weitere Informationen finden Sie unter:</span><span class="sxs-lookup"><span data-stu-id="33d72-166">For more information, see:</span></span>

- <span data-ttu-id="33d72-167">[Überwachen von Apps in Azure App Service][web-sites-monitor]</span><span class="sxs-lookup"><span data-stu-id="33d72-167">[Monitor Apps in Azure App Service][web-sites-monitor]</span></span>
- <span data-ttu-id="33d72-168">[Erstellen und Verwenden von Leistungsindikatoren in einer Azure-Anwendung][performance-counters]</span><span class="sxs-lookup"><span data-stu-id="33d72-168">[Create and use performance counters in an Azure application][performance-counters]</span></span>

<span data-ttu-id="33d72-169">Instrumentieren Sie die Anwendung, um festzustellen, wie Anforderungen verarbeitet werden, sobald sie akzeptiert wurden.</span><span class="sxs-lookup"><span data-stu-id="33d72-169">Instrument the application to see how requests are handled once they have been accepted.</span></span> <span data-ttu-id="33d72-170">Durch Nachverfolgen eines Anforderungsflows kann ermittelt werden, ob die Anforderung langsame Aufrufe ausführt und den aktuellen Thread blockiert.</span><span class="sxs-lookup"><span data-stu-id="33d72-170">Tracing the flow of a request can help to identify whether it is performing slow-running calls and blocking the current thread.</span></span> <span data-ttu-id="33d72-171">Mithilfe eines Threadprofils lassen sich Anforderungen markieren, die blockiert werden.</span><span class="sxs-lookup"><span data-stu-id="33d72-171">Thread profiling can also highlight requests that are being blocked.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="33d72-172">Auslastungstest der Anwendung</span><span class="sxs-lookup"><span data-stu-id="33d72-172">Load test the application</span></span>

<span data-ttu-id="33d72-173">Das folgende Diagramm zeigt die Leistung der oben gezeigten synchronen `GetUserProfile`-Methode unter verschiedenen Auslastungen mit bis zu 4.000 gleichzeitigen Benutzern.</span><span class="sxs-lookup"><span data-stu-id="33d72-173">The following graph shows the performance of the synchronous `GetUserProfile` method shown earlier, under varying loads of up to 4000 concurrent users.</span></span> <span data-ttu-id="33d72-174">Es handelt sich um eine ASP.NET-Anwendung, die in einer Azure-Clouddienst-Webrolle ausgeführt wird.</span><span class="sxs-lookup"><span data-stu-id="33d72-174">The application is an ASP.NET application running in an Azure Cloud Service web role.</span></span>

![Leistungsdiagramm für die Beispielanwendung mit synchronen E/A-Vorgängen][sync-performance]

<span data-ttu-id="33d72-176">Für den synchronen Vorgang wurde eine Wartezeit von 2 Sekunden hartcodiert, um eine synchrone E/A-Verarbeitung zu simulieren, daher liegt die Mindestantwortzeit bei knapp über 2 Sekunden.</span><span class="sxs-lookup"><span data-stu-id="33d72-176">The synchronous operation is hard-coded to sleep for 2 seconds, to simulate synchronous I/O, so the minimum response time is slightly over 2 seconds.</span></span> <span data-ttu-id="33d72-177">Wenn die Auslastung ca. 2.500 Benutzer erreicht, stabilisiert sich die durchschnittliche Antwortzeit auf einem bestimmten Niveau, obwohl die Menge an Anforderungen pro Sekunde weiter steigt.</span><span class="sxs-lookup"><span data-stu-id="33d72-177">When the load reaches approximately 2500 concurrent users, the average response time reaches a plateau, although the volume of requests per second continues to increase.</span></span> <span data-ttu-id="33d72-178">Beachten Sie, dass die Skalierung für diese beiden Messungen logarithmisch ist.</span><span class="sxs-lookup"><span data-stu-id="33d72-178">Note that the scale for these two measures is logarithmic.</span></span> <span data-ttu-id="33d72-179">Die Anzahl von Anforderungen pro Sekunde verdoppelt sich zwischen diesem Punkt und dem Ende des Tests.</span><span class="sxs-lookup"><span data-stu-id="33d72-179">The number of requests per second doubles between this point and the end of the test.</span></span>

<span data-ttu-id="33d72-180">In einer isolierten Umgebung ergibt sich aus diesem Test nicht eindeutig, ob die synchrone E/A-Verarbeitung ein Problem darstellt.</span><span class="sxs-lookup"><span data-stu-id="33d72-180">In isolation, it's not necessarily clear from this test whether the synchronous I/O is a problem.</span></span> <span data-ttu-id="33d72-181">Bei größerer Auslastung erreicht die Anwendung möglicherweise einen kritischen Punkt, an dem der Webserver Anforderungen nicht mehr rechtzeitig verarbeiten kann, sodass Clientanwendungen Timeoutausnahmen empfangen.</span><span class="sxs-lookup"><span data-stu-id="33d72-181">Under heavier load, the application may reach a tipping point where the web server can no longer process requests in a timely manner, causing client applications to receive time-out exceptions.</span></span>

<span data-ttu-id="33d72-182">Eingehende Anforderungen werden vom IIS-Webserver in die Warteschlange eingereiht und an einen Thread weitergeleitet, der im ASP.NET-Threadpool ausgeführt wird.</span><span class="sxs-lookup"><span data-stu-id="33d72-182">Incoming requests are queued by the IIS web server and handed to a thread running in the ASP.NET thread pool.</span></span> <span data-ttu-id="33d72-183">Da jeder Vorgang die E/A-Verarbeitung synchron ausführt, wird der Thread blockiert, bis der Vorgang abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="33d72-183">Because each operation performs I/O synchronously, the thread is blocked until the operation completes.</span></span> <span data-ttu-id="33d72-184">Wenn die Workload steigt, werden schlussendlich alle ASP.NET-Threads im Threadpool zugewiesen und blockiert.</span><span class="sxs-lookup"><span data-stu-id="33d72-184">As the workload increases, eventually all of the ASP.NET threads in the thread pool are allocated and blocked.</span></span> <span data-ttu-id="33d72-185">An diesem Punkt müssen alle weiteren eingehenden Anforderungen in der Warteschlange auf einen verfügbaren Thread warten.</span><span class="sxs-lookup"><span data-stu-id="33d72-185">At that point, any further incoming requests must wait in the queue for an available thread.</span></span> <span data-ttu-id="33d72-186">Wenn die Warteschlange immer länger wird, treten bei Anforderungen Timeouts auf.</span><span class="sxs-lookup"><span data-stu-id="33d72-186">As the queue length grows, requests start to time out.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="33d72-187">Implementieren der Lösung und Überprüfen des Ergebnisses</span><span class="sxs-lookup"><span data-stu-id="33d72-187">Implement the solution and verify the result</span></span>

<span data-ttu-id="33d72-188">Das nächste Diagramm zeigt die Ergebnisse des Auslastungstests für die asynchrone Version des Codes.</span><span class="sxs-lookup"><span data-stu-id="33d72-188">The next graph shows the results from load testing the asynchronous version of the code.</span></span>

![Leistungsdiagramm für die Beispielanwendung mit asynchronen E/A-Vorgängen][async-performance]

<span data-ttu-id="33d72-190">Der Durchsatz ist wesentlich höher.</span><span class="sxs-lookup"><span data-stu-id="33d72-190">Throughput is far higher.</span></span> <span data-ttu-id="33d72-191">Im gleichen Zeitraum wie beim vorherigen Test verarbeitet das System erfolgreich nahezu das 10-Fache des Durchsatzes (gemessen in Anforderungen pro Sekunde).</span><span class="sxs-lookup"><span data-stu-id="33d72-191">Over the same duration as the previous test, the system successfully handles a nearly tenfold increase in throughput, as measured in requests per second.</span></span> <span data-ttu-id="33d72-192">Darüber hinaus ist die durchschnittliche Antwortzeit relativ konstant und bleibt etwa 25-mal kürzer als im vorherigen Test.</span><span class="sxs-lookup"><span data-stu-id="33d72-192">Moreover, the average response time is relatively constant and remains approximately 25 times smaller than the previous test.</span></span>


[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO


[async-wrappers]: http://blogs.msdn.com/b/pfxteam/archive/2012/03/24/10287244.aspx
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg



