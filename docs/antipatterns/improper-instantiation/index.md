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
# <a name="improper-instantiation-antipattern"></a><span data-ttu-id="16eff-103">Antimuster für ungeeignete Instanziierung</span><span class="sxs-lookup"><span data-stu-id="16eff-103">Improper Instantiation antipattern</span></span>

<span data-ttu-id="16eff-104">Es kann für die Leistung nachteilhaft sein, immer wieder neue Instanzen eines Objekts zu erzeugen, das einmal erstellt und dann freigegeben werden soll.</span><span class="sxs-lookup"><span data-stu-id="16eff-104">It can hurt performance to continually create new instances of an object that is meant to be created once and then shared.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="16eff-105">Problembeschreibung</span><span class="sxs-lookup"><span data-stu-id="16eff-105">Problem description</span></span>

<span data-ttu-id="16eff-106">Viele Bibliotheken stellen Abstraktionen von externen Ressourcen bereit.</span><span class="sxs-lookup"><span data-stu-id="16eff-106">Many libraries provide abstractions of external resources.</span></span> <span data-ttu-id="16eff-107">Intern verwalten diese Klassen in der Regel ihre eigenen Verbindungen mit der Ressource. Dabei fungieren sie als Broker, mit denen Clients auf die Ressource zugreifen können.</span><span class="sxs-lookup"><span data-stu-id="16eff-107">Internally, these classes typically manage their own connections to the resource, acting as brokers that clients can use to access the resource.</span></span> <span data-ttu-id="16eff-108">Im Folgenden werden einige Beispiele für Brokerklassen aufgeführt, die für Azure-Anwendungen relevant sind:</span><span class="sxs-lookup"><span data-stu-id="16eff-108">Here are some examples of broker classes that are relevant to Azure applications:</span></span>

- <span data-ttu-id="16eff-109">`System.Net.Http.HttpClient`(Fixierte Verbindung) festgelegt ist(Fixierte Verbindung) festgelegt ist.</span><span class="sxs-lookup"><span data-stu-id="16eff-109">`System.Net.Http.HttpClient`.</span></span> <span data-ttu-id="16eff-110">Kommuniziert über HTTP mit einem Webdienst.</span><span class="sxs-lookup"><span data-stu-id="16eff-110">Communicates with a web service using HTTP.</span></span>
- <span data-ttu-id="16eff-111">`Microsoft.ServiceBus.Messaging.QueueClient`(Fixierte Verbindung) festgelegt ist(Fixierte Verbindung) festgelegt ist.</span><span class="sxs-lookup"><span data-stu-id="16eff-111">`Microsoft.ServiceBus.Messaging.QueueClient`.</span></span> <span data-ttu-id="16eff-112">Stellt Nachrichten für eine Service Bus-Warteschlange bereit, und empfängt diese.</span><span class="sxs-lookup"><span data-stu-id="16eff-112">Posts and receives messages to a Service Bus queue.</span></span> 
- <span data-ttu-id="16eff-113">`Microsoft.Azure.Documents.Client.DocumentClient`(Fixierte Verbindung) festgelegt ist(Fixierte Verbindung) festgelegt ist.</span><span class="sxs-lookup"><span data-stu-id="16eff-113">`Microsoft.Azure.Documents.Client.DocumentClient`.</span></span> <span data-ttu-id="16eff-114">Stellt eine Verbindung mit einer Cosmos DB-Instanz her.</span><span class="sxs-lookup"><span data-stu-id="16eff-114">Connects to a Cosmos DB instance</span></span>
- <span data-ttu-id="16eff-115">`StackExchange.Redis.ConnectionMultiplexer`(Fixierte Verbindung) festgelegt ist(Fixierte Verbindung) festgelegt ist.</span><span class="sxs-lookup"><span data-stu-id="16eff-115">`StackExchange.Redis.ConnectionMultiplexer`.</span></span> <span data-ttu-id="16eff-116">Stellt eine Verbindung mit Redis, einschließlich Azure Redis Cache, her.</span><span class="sxs-lookup"><span data-stu-id="16eff-116">Connects to Redis, including Azure Redis Cache.</span></span>

<span data-ttu-id="16eff-117">Diese Klassen sind dafür gedacht, einmal instanziiert und über die gesamte Lebensdauer einer Anwendung hinweg wiederverwendet zu werden.</span><span class="sxs-lookup"><span data-stu-id="16eff-117">These classes are intended to be instantiated once and reused throughout the lifetime of an application.</span></span> <span data-ttu-id="16eff-118">Es ist jedoch ein weit verbreiteter Irrtum, dass diese Klassen nur bei Bedarf erworben und schnell freigegeben werden sollen.</span><span class="sxs-lookup"><span data-stu-id="16eff-118">However, it's a common misunderstanding that these classes should be acquired only as necessary and released quickly.</span></span> <span data-ttu-id="16eff-119">(Die hier aufgelisteten Klassen beziehen sich zufälligerweise auf .NET-Bibliotheken, aber das Muster ist nicht eindeutig für .NET.)</span><span class="sxs-lookup"><span data-stu-id="16eff-119">(The ones listed here happen to be .NET libraries, but the pattern is not unique to .NET.)</span></span>

<span data-ttu-id="16eff-120">Im folgenden ASP.NET-Beispiel wird eine Instanz von `HttpClient` für die Kommunikation mit einem Remotedienst erstellt.</span><span class="sxs-lookup"><span data-stu-id="16eff-120">The following ASP.NET example creates an instance of `HttpClient` to communicate with a remote service.</span></span> <span data-ttu-id="16eff-121">Das vollständige Beispiel finden Sie [hier][sample-app].</span><span class="sxs-lookup"><span data-stu-id="16eff-121">You can find the complete sample [here][sample-app].</span></span>

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

<span data-ttu-id="16eff-122">In einer Webanwendung ist diese Methode nicht skalierbar.</span><span class="sxs-lookup"><span data-stu-id="16eff-122">In a web application, this technique is not scalable.</span></span> <span data-ttu-id="16eff-123">Für jede Benutzeranforderung wird ein neues `HttpClient`-Objekt erstellt.</span><span class="sxs-lookup"><span data-stu-id="16eff-123">A new `HttpClient` object is created for each user request.</span></span> <span data-ttu-id="16eff-124">Bei hoher Auslastung kann durch den Webserver die Anzahl der verfügbaren Sockets ausgeschöpft werden, was zu `SocketException`-Fehlern führt.</span><span class="sxs-lookup"><span data-stu-id="16eff-124">Under heavy load, the web server may exhaust the number of available sockets, resulting in `SocketException` errors.</span></span>

<span data-ttu-id="16eff-125">Dieses Problem ist nicht auf die `HttpClient`-Klasse beschränkt.</span><span class="sxs-lookup"><span data-stu-id="16eff-125">This problem is not restricted to the `HttpClient` class.</span></span> <span data-ttu-id="16eff-126">Andere Klassen, die Ressourcen umschließen oder deren Erstellung kostspielig ist, können ähnliche Probleme verursachen.</span><span class="sxs-lookup"><span data-stu-id="16eff-126">Other classes that wrap resources or are expensive to create might cause similar issues.</span></span> <span data-ttu-id="16eff-127">Im folgenden Beispiel wird eine Instanz der `ExpensiveToCreateService`-Klasse erstellt.</span><span class="sxs-lookup"><span data-stu-id="16eff-127">The following example creates an instances of the `ExpensiveToCreateService` class.</span></span> <span data-ttu-id="16eff-128">Hier ist die Erschöpfung von Sockets nicht unbedingt problematisch, sondern lediglich die Dauer für die Erstellung der einzelnen Instanzen.</span><span class="sxs-lookup"><span data-stu-id="16eff-128">Here the issue is not necessarily socket exhaustion, but simply how long it takes to create each instance.</span></span> <span data-ttu-id="16eff-129">Das kontinuierliche Erstellen und Zerstören von Instanzen dieser Klasse kann die Skalierbarkeit des Systems beeinträchtigen.</span><span class="sxs-lookup"><span data-stu-id="16eff-129">Continually creating and destroying instances of this class might adversely affect the scalability of the system.</span></span>

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

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="16eff-130">Beheben des Problems</span><span class="sxs-lookup"><span data-stu-id="16eff-130">How to fix the problem</span></span>

<span data-ttu-id="16eff-131">Wenn die Klasse, die die externe Ressource umschließt, gemeinsam nutzbar und threadsicher ist, erstellen Sie eine gemeinsame Singletoninstanz oder einen Pool von wiederverwendbaren Instanzen der Klasse.</span><span class="sxs-lookup"><span data-stu-id="16eff-131">If the class that wraps the external resource is shareable and thread-safe, create a shared singleton instance or a pool of reusable instances of the class.</span></span>

<span data-ttu-id="16eff-132">Im folgenden Beispiel wird eine statische `HttpClient`-Instanz verwendet. Dadurch wird die Verbindung für alle Anforderungen freigegeben.</span><span class="sxs-lookup"><span data-stu-id="16eff-132">The following example uses a static `HttpClient` instance, thus sharing the connection across all requests.</span></span>

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

## <a name="considerations"></a><span data-ttu-id="16eff-133">Überlegungen</span><span class="sxs-lookup"><span data-stu-id="16eff-133">Considerations</span></span>

- <span data-ttu-id="16eff-134">Das Schlüsselelement dieses Antimusters ist das wiederholte Erstellen und Zerstören von Instanzen eines *gemeinsam nutzbaren* Objekts.</span><span class="sxs-lookup"><span data-stu-id="16eff-134">The key element of this antipattern is repeatedly creating and destroying instances of a *shareable* object.</span></span> <span data-ttu-id="16eff-135">Wenn eine Klasse nicht gemeinsam nutzbar (nicht threadsicher) ist, dann wird dieses Antimuster nicht angewendet.</span><span class="sxs-lookup"><span data-stu-id="16eff-135">If a class is not shareable (not thread-safe), then this antipattern does not apply.</span></span>

- <span data-ttu-id="16eff-136">Der Typ der freigegebenen Ressource kann darüber entscheiden, ob Sie ein Singleton verwenden oder einen Pool erstellen sollten.</span><span class="sxs-lookup"><span data-stu-id="16eff-136">The type of shared resource might dictate whether you should use a singleton or create a pool.</span></span> <span data-ttu-id="16eff-137">Die `HttpClient`-Klasse ist für die gemeinsame Verwendung konzipiert, nicht für das Zusammenfassen in einem Pool.</span><span class="sxs-lookup"><span data-stu-id="16eff-137">The `HttpClient` class is designed to be shared rather than pooled.</span></span> <span data-ttu-id="16eff-138">Andere Objekte können das Pooling unterstützen, sodass das System die Workload auf mehrere Instanzen verteilen kann.</span><span class="sxs-lookup"><span data-stu-id="16eff-138">Other objects might support pooling, enabling the system to spread the workload across multiple instances.</span></span>

- <span data-ttu-id="16eff-139">Objekte, die Sie über mehrere Anforderungen hinweg freigeben, *müssen* threadsicher sein.</span><span class="sxs-lookup"><span data-stu-id="16eff-139">Objects that you share across multiple requests *must* be thread-safe.</span></span> <span data-ttu-id="16eff-140">Die `HttpClient`-Klasse ist zwar für diese Art der Verwendung konzipiert, allerdings kann es sein, dass andere Klassen keine gleichzeitigen Anforderungen unterstützen. Sehen Sie daher in der verfügbaren Dokumentation nach.</span><span class="sxs-lookup"><span data-stu-id="16eff-140">The `HttpClient` class is designed to be used in this manner, but other classes might not support concurrent requests, so check the available documentation.</span></span>

- <span data-ttu-id="16eff-141">Einige Ressourcentypen sind knapp und sollten nicht beibehalten werden.</span><span class="sxs-lookup"><span data-stu-id="16eff-141">Some resource types are scarce and should not be held onto.</span></span> <span data-ttu-id="16eff-142">Dies gilt beispielsweise für Datenbankverbindungen.</span><span class="sxs-lookup"><span data-stu-id="16eff-142">Database connections are an example.</span></span> <span data-ttu-id="16eff-143">Das Aufrechterhalten einer offenen Datenbankverbindung, die nicht erforderlich ist, kann andere gleichzeitige Benutzer daran hindern, auf die Datenbank zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="16eff-143">Holding an open database connection that is not required may prevent other concurrent users from gaining access to the database.</span></span>

- <span data-ttu-id="16eff-144">Im .NET Framework werden viele Objekte, die Verbindungen mit externen Ressourcen herstellen, mit statischen Factorymethoden anderer Klassen erstellt, die diese Verbindungen verwalten.</span><span class="sxs-lookup"><span data-stu-id="16eff-144">In the .NET Framework, many objects that establish connections to external resources are created by using static factory methods of other classes that manage these connections.</span></span> <span data-ttu-id="16eff-145">Diese Factoryobjekte sind für die Speicherung und Wiederverwendung bestimmt, nicht für die Löschung und Wiederherstellung.</span><span class="sxs-lookup"><span data-stu-id="16eff-145">These factories  objects are intended to be saved and reused, rather than disposed and recreated.</span></span> <span data-ttu-id="16eff-146">Beispielsweise wird im Azure Service Bus das `QueueClient`-Objekt durch ein `MessagingFactory`-Objekt erstellt.</span><span class="sxs-lookup"><span data-stu-id="16eff-146">For example, in Azure Service Bus, the `QueueClient` object is created through a `MessagingFactory` object.</span></span> <span data-ttu-id="16eff-147">Intern verwaltet `MessagingFactory` Verbindungen.</span><span class="sxs-lookup"><span data-stu-id="16eff-147">Internally, the `MessagingFactory` manages connections.</span></span> <span data-ttu-id="16eff-148">Weitere Informationen finden Sie unter [Bewährte Methoden für Leistungsoptimierungen mithilfe von Service Bus-Messaging][service-bus-messaging].</span><span class="sxs-lookup"><span data-stu-id="16eff-148">For more information, see [Best Practices for performance improvements using Service Bus Messaging][service-bus-messaging].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="16eff-149">Erkennen des Problems</span><span class="sxs-lookup"><span data-stu-id="16eff-149">How to detect the problem</span></span>

<span data-ttu-id="16eff-150">Zu den Symptomen dieses Problems zählen neben einem Rückgang des Durchsatzes oder einer erhöhten Fehlerrate einer oder mehrere der folgenden Punkte:</span><span class="sxs-lookup"><span data-stu-id="16eff-150">Symptoms of this problem include a drop in throughput or an increased error rate, along with one or more of the following:</span></span> 

- <span data-ttu-id="16eff-151">Eine Zunahme von Ausnahmen, die auf die Erschöpfung von Ressourcen wie Sockets, Datenbankverbindungen, Dateihandles usw. hinweist</span><span class="sxs-lookup"><span data-stu-id="16eff-151">An increase in exceptions that indicate exhaustion of resources such as sockets, database connections, file handles, and so on.</span></span> 
- <span data-ttu-id="16eff-152">Erhöhter Speicherverbrauch und Garbage Collection</span><span class="sxs-lookup"><span data-stu-id="16eff-152">Increased memory use and garbage collection.</span></span>
- <span data-ttu-id="16eff-153">Eine Zunahme der Netzwerk-, Festplatten- oder Datenbankaktivität</span><span class="sxs-lookup"><span data-stu-id="16eff-153">An increase in network, disk, or database activity.</span></span>

<span data-ttu-id="16eff-154">Sie können die folgenden Schritte durchführen, um dieses Problem zu identifizieren:</span><span class="sxs-lookup"><span data-stu-id="16eff-154">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="16eff-155">Führen Sie eine Prozessüberwachung des Produktionssystems durch, um Punkte zu identifizieren, an denen Antwortzeiten verlangsamt werden oder aufgrund mangelnder Ressourcen ein Fehler im System auftritt.</span><span class="sxs-lookup"><span data-stu-id="16eff-155">Performing process monitoring of the production system, to identify points when response times slow down or the system fails due to lack of resources.</span></span>
2. <span data-ttu-id="16eff-156">Untersuchen Sie die an diesen Punkten erfassten Telemetriedaten, um festzustellen, welche Vorgänge ressourcenverbrauchende Objekte erstellen und zerstören könnten.</span><span class="sxs-lookup"><span data-stu-id="16eff-156">Examine the telemetry data captured at these points to determine which operations might be creating and destroying resource-consuming objects.</span></span>
3. <span data-ttu-id="16eff-157">Führen Sie für jeden vermuteten Vorgang in einer kontrollierten Testumgebung einen Auslastungstest durch (nicht im Produktionssystem).</span><span class="sxs-lookup"><span data-stu-id="16eff-157">Load test each suspected operation, in a controlled test environment rather than the production system.</span></span>
4. <span data-ttu-id="16eff-158">Überprüfen Sie den Quellcode, und untersuchen Sie, wie Brokerobjekte verwaltet werden.</span><span class="sxs-lookup"><span data-stu-id="16eff-158">Review the source code and examine the how broker objects are managed.</span></span>

<span data-ttu-id="16eff-159">Schauen Sie sich Stapelüberwachungen für Vorgänge an, die langsam ablaufen oder Ausnahmen erzeugen, wenn das System ausgelastet ist.</span><span class="sxs-lookup"><span data-stu-id="16eff-159">Look at stack traces for operations that are slow-running or that generate exceptions when the system is under load.</span></span> <span data-ttu-id="16eff-160">Anhand dieser Informationen können Sie erkennen, wie diese Vorgänge Ressourcen verwenden.</span><span class="sxs-lookup"><span data-stu-id="16eff-160">This information can help to identify how these operations are utilizing resources.</span></span> <span data-ttu-id="16eff-161">Mithilfe von Ausnahmen kann festgestellt werden, ob Fehler dadurch verursacht werden, dass die gemeinsam genutzten Ressourcen ausgeschöpft sind.</span><span class="sxs-lookup"><span data-stu-id="16eff-161">Exceptions can help to determine whether errors are caused by shared resources being exhausted.</span></span> 

## <a name="example-diagnosis"></a><span data-ttu-id="16eff-162">Beispieldiagnose</span><span class="sxs-lookup"><span data-stu-id="16eff-162">Example diagnosis</span></span>

<span data-ttu-id="16eff-163">In den folgenden Abschnitten werden diese Schritte auf die zuvor beschriebene Beispielanwendung angewendet.</span><span class="sxs-lookup"><span data-stu-id="16eff-163">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slow-down-or-failure"></a><span data-ttu-id="16eff-164">Identifizieren der Punkte der Verlangsamung oder des Ausfalls</span><span class="sxs-lookup"><span data-stu-id="16eff-164">Identify points of slow down or failure</span></span>

<span data-ttu-id="16eff-165">Die folgende Abbildung zeigt mit [New Relic APM][new-relic] erzeugte Ergebnisse, die Vorgänge mit einer schlechten Antwortzeit zeigen.</span><span class="sxs-lookup"><span data-stu-id="16eff-165">The following image shows results generated using [New Relic APM][new-relic], showing operations that have a poor response time.</span></span> <span data-ttu-id="16eff-166">In diesem Fall lohnt es sich, sich mit der `GetProductAsync`-Methode im `NewHttpClientInstancePerRequest`-Controller näher auseinanderzusetzen.</span><span class="sxs-lookup"><span data-stu-id="16eff-166">In this case, the `GetProductAsync` method in the `NewHttpClientInstancePerRequest` controller are worth investigating further.</span></span> <span data-ttu-id="16eff-167">Beachten Sie, dass sich auch die Fehlerrate erhöht, wenn diese Vorgänge ausgeführt werden.</span><span class="sxs-lookup"><span data-stu-id="16eff-167">Notice that the error rate also increases when these operations are running.</span></span> 

![New Relic-Überwachungsdashboard mit der Beispielanwendung für die Erstellung einer neuen HttpClient-Objektinstanz für die einzelnen Anforderungen][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="16eff-169">Untersuchen der Telemetriedaten und Ermitteln von Korrelationen</span><span class="sxs-lookup"><span data-stu-id="16eff-169">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="16eff-170">Die nächste Abbildung zeigt Daten, die über denselben Zeitraum wie bei der vorherigen Abbildung mit der Threadprofilerstellung erfasst wurden.</span><span class="sxs-lookup"><span data-stu-id="16eff-170">The next image shows data captured using thread profiling, over the same period corresponding as the previous image.</span></span> <span data-ttu-id="16eff-171">Das Öffnen von Socketverbindungen durch das System dauert sehr lange. Das Schließen und Verarbeiten von Socketausnahmen dauert sogar noch länger.</span><span class="sxs-lookup"><span data-stu-id="16eff-171">The system spends a significant time opening socket connections, and even more time closing them and handling socket exceptions.</span></span>

![New Relic-Threadprofilersteller mit der Beispielanwendung für die Erstellung einer neuen HttpClient-Objektinstanz für die einzelnen Anforderungen][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a><span data-ttu-id="16eff-173">Durchführen von Auslastungstests</span><span class="sxs-lookup"><span data-stu-id="16eff-173">Performing load testing</span></span>

<span data-ttu-id="16eff-174">Simulieren Sie mithilfe von Auslastungstests typische Vorgänge, die von Benutzern ausgeführt werden könnten.</span><span class="sxs-lookup"><span data-stu-id="16eff-174">Use load testing to simulate the typical operations that users might perform.</span></span> <span data-ttu-id="16eff-175">Hierdurch können besser die Teile eines Systems ermittelt werden, bei denen unter wechselnder Belastung Ressourcenauslastung auftritt.</span><span class="sxs-lookup"><span data-stu-id="16eff-175">This can help to identify which parts of a system suffer from resource exhaustion under varying loads.</span></span> <span data-ttu-id="16eff-176">Führen Sie diese Tests in einer kontrollierten Umgebung und nicht im Produktionssystem durch.</span><span class="sxs-lookup"><span data-stu-id="16eff-176">Perform these tests in a controlled environment rather than the production system.</span></span> <span data-ttu-id="16eff-177">Das folgende Diagramm zeigt den Durchsatz der Anforderungen, die vom `NewHttpClientInstancePerRequest`-Controller bearbeitet werden, wenn die Benutzerauslastung auf 100 gleichzeitige Benutzer steigt.</span><span class="sxs-lookup"><span data-stu-id="16eff-177">The following graph shows the throughput of requests handled by the `NewHttpClientInstancePerRequest` controller as the user load increases to 100 concurrent users.</span></span>

![Durchsatz der Beispielanwendung für die Erstellung einer neuen HttpClient-Objektinstanz für die einzelnen Anforderungen][throughput-new-HTTPClient-instance]

<span data-ttu-id="16eff-179">Die Anzahl der pro Sekunde bearbeiteten Anforderungen steigt zunächst mit zunehmender Workloadanzahl.</span><span class="sxs-lookup"><span data-stu-id="16eff-179">At first, the volume of requests handled per second increases as the workload increases.</span></span> <span data-ttu-id="16eff-180">Bei ca. 30 Benutzern erreicht die Anzahl der erfolgreichen Anforderungen jedoch eine Begrenzung, und das System beginnt, Ausnahmen zu generieren.</span><span class="sxs-lookup"><span data-stu-id="16eff-180">At about 30 users, however, the volume of successful requests reaches a limit, and the system starts to generate exceptions.</span></span> <span data-ttu-id="16eff-181">Von da an nimmt die Anzahl der Ausnahmen mit der Benutzerauslastung allmählich zu.</span><span class="sxs-lookup"><span data-stu-id="16eff-181">From then on, the volume of exceptions gradually increases with the user load.</span></span> 

<span data-ttu-id="16eff-182">Beim Auslastungstest wurden diese Fehler als „HTTP 500 (Interner Serverfehler)“-Fehler gemeldet.</span><span class="sxs-lookup"><span data-stu-id="16eff-182">The load test reported these failures as HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="16eff-183">Die Überprüfung der Telemetriedaten ergab, dass diese Fehler dadurch verursacht wurden, dass die Socketressourcen durch das System ausgeschöpft wurden, je mehr `HttpClient`-Objekte erstellt wurden.</span><span class="sxs-lookup"><span data-stu-id="16eff-183">Reviewing the telemetry showed that these errors were caused by the system running out of socket resources, as more and more `HttpClient` objects were created.</span></span>

<span data-ttu-id="16eff-184">Das nächste Diagramm zeigt einen ähnlichen Test für einen Controller, der das benutzerdefinierte `ExpensiveToCreateService`-Objekt erstellt.</span><span class="sxs-lookup"><span data-stu-id="16eff-184">The next graph shows a similar test for a controller that creates the custom `ExpensiveToCreateService` object.</span></span>

![Durchsatz der Beispielanwendung für die Erstellung einer neuen ExpensiveToCreateService-Objektinstanz für die einzelnen Anforderungen][throughput-new-ExpensiveToCreateService-instance]

<span data-ttu-id="16eff-186">Dieses Mal generiert der Controller keine Ausnahmen, aber der Durchsatz befindet sich nach wie vor im Stillstand, während die durchschnittliche Reaktionszeit um den Faktor 20 zunimmt.</span><span class="sxs-lookup"><span data-stu-id="16eff-186">This time, the controller does not generate any exceptions, but throughput still reaches a plateau, while the average response time increases by a factor of 20.</span></span> <span data-ttu-id="16eff-187">(Das Diagramm verwendet eine logarithmische Skalierung für die Antwortzeit und den Durchsatz.) Die Telemetriedaten zeigen, dass die Erstellung neuer `ExpensiveToCreateService`-Instanzen die Hauptursache des Problems war.</span><span class="sxs-lookup"><span data-stu-id="16eff-187">(The graph uses a logarithmic scale for response time and throughput.) Telemetry showed that creating new instances of the `ExpensiveToCreateService` was the main cause of the problem.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="16eff-188">Implementieren der Lösung und Überprüfen des Ergebnisses</span><span class="sxs-lookup"><span data-stu-id="16eff-188">Implement the solution and verify the result</span></span>

<span data-ttu-id="16eff-189">Nach dem Wechsel der `GetProductAsync`-Methode zur Freigabe einer einzelnen `HttpClient`-Instanz zeigte ein zweiter Auslastungstest, dass die Leistung verbessert wurde.</span><span class="sxs-lookup"><span data-stu-id="16eff-189">After switching the `GetProductAsync` method to share a single `HttpClient` instance, a second load test showed improved performance.</span></span> <span data-ttu-id="16eff-190">Es wurden keine Fehler gemeldet, und das System war in der Lage, eine steigende Last von bis zu 500 Anforderungen pro Sekunde zu bewältigen.</span><span class="sxs-lookup"><span data-stu-id="16eff-190">No errors were reported, and the system was able to handle an increasing load of up to 500 requests per second.</span></span> <span data-ttu-id="16eff-191">Die durchschnittliche Antwortzeit wurde im Vergleich zum vorherigen Test halbiert.</span><span class="sxs-lookup"><span data-stu-id="16eff-191">The average response time was cut in half, compared with the previous test.</span></span>

![Durchsatz der Beispielanwendung für die Erstellung derselben HttpClient-Objektinstanz für die einzelnen Anforderungen][throughput-single-HTTPClient-instance]

<span data-ttu-id="16eff-193">Zum Vergleich: Die folgende Abbildung zeigt die Telemetriedaten der Stapelüberwachung.</span><span class="sxs-lookup"><span data-stu-id="16eff-193">For comparison, the following image shows the stack trace telemetry.</span></span> <span data-ttu-id="16eff-194">Dieses Mal wendet das System den Großteil der Zeit für die Durchführung realer Aufgaben statt für das Öffnen und Schließen von Sockets.</span><span class="sxs-lookup"><span data-stu-id="16eff-194">This time, the system spends most of its time performing real work, rather than opening and closing sockets.</span></span>

![New Relic-Threadprofilersteller mit der Beispielanwendung für die Erstellung einer einzelnen HttpClient-Objektinstanz für alle Anforderungen][thread-profiler-single-HTTPClient-instance]

<span data-ttu-id="16eff-196">Das nächste Diagramm zeigt einen ähnlichen Auslastungstest mit einer freigegebenen Instanz des `ExpensiveToCreateService`-Objekts.</span><span class="sxs-lookup"><span data-stu-id="16eff-196">The next graph shows a similar load test using a shared instance of the `ExpensiveToCreateService` object.</span></span> <span data-ttu-id="16eff-197">Auch hier steigt die Anzahl der bearbeiteten Anforderungen mit der Benutzerauslastung, während die durchschnittliche Antwortzeit gering bleibt.</span><span class="sxs-lookup"><span data-stu-id="16eff-197">Again, the volume of handled requests increases in line with the user load, while the average response time remains low.</span></span> 

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
