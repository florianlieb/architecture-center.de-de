---
title: "Antimuster „Kein Caching“"
description: "Durch wiederholtes Abrufen der gleichen Daten können Leistung und Skalierbarkeit verringert werden."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 8a2bc3b473a30536cc1bef9e1dcad87acb46c4a9
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/05/2018
---
# <a name="no-caching-antipattern"></a><span data-ttu-id="ee01b-103">Antimuster „Kein Caching“</span><span class="sxs-lookup"><span data-stu-id="ee01b-103">No Caching antipattern</span></span>

<span data-ttu-id="ee01b-104">In einer Cloudanwendung, die viele gleichzeitige Anforderungen verarbeitet, können durch wiederholtes Abrufen der gleichen Daten Leistung und Skalierbarkeit sinken.</span><span class="sxs-lookup"><span data-stu-id="ee01b-104">In a cloud application that handles many concurrent requests, repeatedly fetching the same data can reduce performance and scalability.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="ee01b-105">Problembeschreibung</span><span class="sxs-lookup"><span data-stu-id="ee01b-105">Problem description</span></span>

<span data-ttu-id="ee01b-106">Wenn die Daten nicht zwischengespeichert werden, kann eine Reihe unerwünschter Verhaltensweisen auftreten, wie z.B. folgende:</span><span class="sxs-lookup"><span data-stu-id="ee01b-106">When data is not cached, it can cause a number of undesirable behaviors, including:</span></span>

- <span data-ttu-id="ee01b-107">Wiederholtes Abrufen der gleichen Informationen aus einer Ressource, deren Zugriff viel E/A-Overhead oder Latenz kostet</span><span class="sxs-lookup"><span data-stu-id="ee01b-107">Repeatedly fetching the same information from a resource that is expensive to access, in terms of I/O overhead or latency.</span></span>
- <span data-ttu-id="ee01b-108">Wiederholtes Erstellen der gleichen Objekte oder Datenstrukturen für mehrere Anforderungen</span><span class="sxs-lookup"><span data-stu-id="ee01b-108">Repeatedly constructing the same objects or data structures for multiple requests.</span></span>
- <span data-ttu-id="ee01b-109">Übermäßig viele Abrufe für einen Remotedienst, für den ein Dienstkontingent gilt und der Clients ab einem bestimmten Grenzwert drosselt</span><span class="sxs-lookup"><span data-stu-id="ee01b-109">Making excessive calls to a remote service that has a service quota and throttles clients past a certain limit.</span></span>

<span data-ttu-id="ee01b-110">Diese Probleme können zu langen Antwortzeiten, vermehrten Konflikten im Datenspeicher und unzureichender Skalierbarkeit führen.</span><span class="sxs-lookup"><span data-stu-id="ee01b-110">In turn, these problems can lead to poor response times, increased contention in the data store, and poor scalability.</span></span>

<span data-ttu-id="ee01b-111">Das folgende Beispiel verwendet Entity Framework, um eine Verbindung mit einer Datenbank herzustellen.</span><span class="sxs-lookup"><span data-stu-id="ee01b-111">The following example uses Entity Framework to connect to a database.</span></span> <span data-ttu-id="ee01b-112">Jede Clientanforderung führt zu einem Aufruf in der Datenbank, selbst wenn mehrere Anforderungen exakt die gleichen Daten abrufen.</span><span class="sxs-lookup"><span data-stu-id="ee01b-112">Every client request results in a call to the database, even if multiple requests are fetching exactly the same data.</span></span> <span data-ttu-id="ee01b-113">Die Kosten wiederholter Anforderungen hinsichtlich E/A-Overhead und Datenzugriffsgebühren können sich schnell summieren.</span><span class="sxs-lookup"><span data-stu-id="ee01b-113">The cost of repeated requests, in terms of I/O overhead and data access charges, can accumulate quickly.</span></span>

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

<span data-ttu-id="ee01b-114">Das vollständige Beispiel finden Sie [hier][sample-app].</span><span class="sxs-lookup"><span data-stu-id="ee01b-114">You can find the complete sample [here][sample-app].</span></span>

<span data-ttu-id="ee01b-115">Dieses Antimuster tritt üblicherweise aus folgenden Gründen auf:</span><span class="sxs-lookup"><span data-stu-id="ee01b-115">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="ee01b-116">Wenn Sie keinen Cache verwenden, ist die Implementierung einfacher, und das Verfahren funktioniert gut bei geringen Lasten.</span><span class="sxs-lookup"><span data-stu-id="ee01b-116">Not using a cache is simpler to implement, and it works fine under low loads.</span></span> <span data-ttu-id="ee01b-117">Die Verwendung eines Caches macht den Code komplizierter.</span><span class="sxs-lookup"><span data-stu-id="ee01b-117">Caching makes the code more complicated.</span></span> 
- <span data-ttu-id="ee01b-118">Die Vor- und Nachteile eines Caches sind nicht klar.</span><span class="sxs-lookup"><span data-stu-id="ee01b-118">The benefits and drawbacks of using a cache are not clearly understood.</span></span>
- <span data-ttu-id="ee01b-119">Es gibt Bedenken hinsichtlich des Overheads für die Sicherstellung der Genauigkeit und Aktualität der zwischengespeicherten Daten.</span><span class="sxs-lookup"><span data-stu-id="ee01b-119">There is concern about the overhead of maintaining the accuracy and freshness of cached data.</span></span>
- <span data-ttu-id="ee01b-120">Eine Anwendung wurde aus einem lokalen System migriert, in dem die Netzwerklatenz kein Problem war und das System auf teurer Hochleistungshardware ausgeführt wurde, daher wurde im ursprünglichen Entwurf kein Caching berücksichtigt.</span><span class="sxs-lookup"><span data-stu-id="ee01b-120">An application was migrated from an on-premises system, where network latency was not an issue, and the system ran on expensive high-performance hardware, so caching wasn't considered in the original design.</span></span>
- <span data-ttu-id="ee01b-121">Entwicklern ist nicht bewusst, dass Caching in einem bestimmten Szenario eine Möglichkeit sein kann.</span><span class="sxs-lookup"><span data-stu-id="ee01b-121">Developers aren't aware that caching is a possibility in a given scenario.</span></span> <span data-ttu-id="ee01b-122">Beispielsweise denken Entwickler beim Implementieren einer Web-API möglicherweise nicht daran, ETags zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="ee01b-122">For example, developers may not think of using ETags when implementing a web API.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="ee01b-123">Beheben des Problems</span><span class="sxs-lookup"><span data-stu-id="ee01b-123">How to fix the problem</span></span>

<span data-ttu-id="ee01b-124">Die beliebteste Cachingstrategie ist die *bedarfsbasierte* oder *cachefremde* Strategie.</span><span class="sxs-lookup"><span data-stu-id="ee01b-124">The most popular caching strategy is the *on-demand* or *cache-aside* strategy.</span></span>

- <span data-ttu-id="ee01b-125">Während eines Lesevorgangs versucht die Anwendung, die Daten aus dem Cache zu lesen.</span><span class="sxs-lookup"><span data-stu-id="ee01b-125">On read, the application tries to read the data from the cache.</span></span> <span data-ttu-id="ee01b-126">Wenn sich die Daten nicht im Cache befinden, ruft die Anwendung sie aus der Datenquelle ab und fügt sie dem Cache hinzu.</span><span class="sxs-lookup"><span data-stu-id="ee01b-126">If the data isn't in the cache, the application retrieves it from the data source and adds it to the cache.</span></span>
- <span data-ttu-id="ee01b-127">Bei einem Schreibvorgang schreibt die Anwendung die Änderung direkt in die Datenquelle und entfernt den alten Wert aus dem Cache.</span><span class="sxs-lookup"><span data-stu-id="ee01b-127">On write, the application writes the change directly to the data source and removes the old value from the cache.</span></span> <span data-ttu-id="ee01b-128">Wenn die Daten das nächste Mal benötigt werden, werden sie abgerufen und dem Cache hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="ee01b-128">It will be retrieved and added to the cache the next time it is required.</span></span>

<span data-ttu-id="ee01b-129">Diese Vorgehensweise eignet sich für Daten, die sich häufig ändern.</span><span class="sxs-lookup"><span data-stu-id="ee01b-129">This approach is suitable for data that changes frequently.</span></span> <span data-ttu-id="ee01b-130">Hier sehen Sie das vorherige Beispiel, das aktualisiert wurde und jetzt das [cachefremde][cache-aside] Muster verwendet.</span><span class="sxs-lookup"><span data-stu-id="ee01b-130">Here is the previous example updated to use the [Cache-Aside][cache-aside] pattern.</span></span>  

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

<span data-ttu-id="ee01b-131">Beachten Sie, dass die `GetAsync`-Methode jetzt die `CacheService`-Klasse aufruft, anstatt die Datenbank direkt aufzurufen.</span><span class="sxs-lookup"><span data-stu-id="ee01b-131">Notice that the `GetAsync` method now calls the `CacheService` class, rather than calling the database directly.</span></span> <span data-ttu-id="ee01b-132">Die `CacheService`-Klasse versucht zuerst, das Element aus Azure Redis Cache abzurufen.</span><span class="sxs-lookup"><span data-stu-id="ee01b-132">The `CacheService` class first tries to get the item from Azure Redis Cache.</span></span> <span data-ttu-id="ee01b-133">Wenn der Wert in Redis Cache nicht gefunden wird, ruft `CacheService` eine Lambdafunktion auf, die vom Aufrufer an sie übergeben wurde.</span><span class="sxs-lookup"><span data-stu-id="ee01b-133">If the value isn't found in Redis Cache, the `CacheService` invokes a lambda function that was passed to it by the caller.</span></span> <span data-ttu-id="ee01b-134">Die Lambdafunktion ist dafür zuständig, die Daten aus der Datenbank abzurufen.</span><span class="sxs-lookup"><span data-stu-id="ee01b-134">The lambda function is responsible for fetching the data from the database.</span></span> <span data-ttu-id="ee01b-135">Diese Implementierung entkoppelt das Repository von der jeweiligen Cachinglösung und den `CacheService` von der Datenbank.</span><span class="sxs-lookup"><span data-stu-id="ee01b-135">This implementation decouples the repository from the particular caching solution, and decouples the `CacheService` from the database.</span></span> 

## <a name="considerations"></a><span data-ttu-id="ee01b-136">Überlegungen</span><span class="sxs-lookup"><span data-stu-id="ee01b-136">Considerations</span></span>

- <span data-ttu-id="ee01b-137">Wenn der Cache nicht verfügbar ist – möglicherweise aufgrund eines vorübergehenden Fehlers – geben Sie keinen Fehler an den Client zurück.</span><span class="sxs-lookup"><span data-stu-id="ee01b-137">If the cache is unavailable, perhaps because of a transient failure, don't return an error to the client.</span></span> <span data-ttu-id="ee01b-138">Rufen Sie die Daten stattdessen aus der ursprünglichen Datenquelle ab.</span><span class="sxs-lookup"><span data-stu-id="ee01b-138">Instead, fetch the data from the original data source.</span></span> <span data-ttu-id="ee01b-139">Denken Sie jedoch daran, dass während der Wiederherstellung des Caches der ursprüngliche Datenspeicher mit Anforderungen überschwemmt werden könnte, sodass Zeitüberschreitungen und Verbindungsfehler die Folge wären.</span><span class="sxs-lookup"><span data-stu-id="ee01b-139">However, be aware that while the cache is being recovered, the original data store could be swamped with requests, resulting in timeouts and failed connections.</span></span> <span data-ttu-id="ee01b-140">(Dies ist schließlich einer der Hauptgründe, aus denen ein Cache überhaupt verwendet wird.) Verwenden Sie eine Technik wie z.B. das [Sicherungsmuster][circuit-breaker], um eine übermäßige Beanspruchung der Datenquelle zu vermeiden.</span><span class="sxs-lookup"><span data-stu-id="ee01b-140">(After all, this is one of the motivations for using a cache in the first place.) Use a technique such as the [Circuit Breaker pattern][circuit-breaker] to avoid overwhelming the data source.</span></span>

- <span data-ttu-id="ee01b-141">Anwendungen, die nicht statische Daten zwischenspeichern, sollte für die Unterstützung der letztlichen Konsistenz entworfen werden.</span><span class="sxs-lookup"><span data-stu-id="ee01b-141">Applications that cache nonstatic data should be designed to support eventual consistency.</span></span>

- <span data-ttu-id="ee01b-142">Bei Web-APIs können Sie das clientseitige Cachen unterstützen, indem Sie einen Cache-Control-Header in die Anforderungs- und Antwortnachricht einschließen und ETags zum Identifizieren der Version von Objekten verwenden.</span><span class="sxs-lookup"><span data-stu-id="ee01b-142">For web APIs, you can support client-side caching by including a Cache-Control header in request and response messages, and using ETags to identify versions of objects.</span></span> <span data-ttu-id="ee01b-143">Weitere Informationen finden Sie unter [API-Implementierung][api-implementation].</span><span class="sxs-lookup"><span data-stu-id="ee01b-143">For more information, see [API implementation][api-implementation].</span></span>

- <span data-ttu-id="ee01b-144">Sie müssen keine vollständigen Entitäten zwischenspeichern.</span><span class="sxs-lookup"><span data-stu-id="ee01b-144">You don't have to cache entire entities.</span></span> <span data-ttu-id="ee01b-145">Wenn der größte Teil einer Entität statisch ist und sich nur ein kleiner Teil häufig ändert, speichern Sie die statischen Elemente zwischen, und rufen Sie die dynamischen Elemente aus der Datenquelle ab.</span><span class="sxs-lookup"><span data-stu-id="ee01b-145">If most of an entity is static but only a small piece changes frequently, cache the static elements and retrieve the dynamic elements from the data source.</span></span> <span data-ttu-id="ee01b-146">Mit dieser Vorgehensweise können Sie die Menge an E/A-Vorgängen reduzieren, die für die Datenquelle ausgeführt werden.</span><span class="sxs-lookup"><span data-stu-id="ee01b-146">This approach can help to reduce the volume of I/O being performed against the data source.</span></span>

- <span data-ttu-id="ee01b-147">In einigen Fällen, wenn flüchtige Daten kurzlebig sind, kann es nützlich sein, diese zwischenzuspeichern.</span><span class="sxs-lookup"><span data-stu-id="ee01b-147">In some cases, if volatile data is short-lived, it can be useful to cache it.</span></span> <span data-ttu-id="ee01b-148">Nehmen wir als Beispiel ein Gerät, das kontinuierlich Statusupdates sendet.</span><span class="sxs-lookup"><span data-stu-id="ee01b-148">For example, consider a device that continually sends status updates.</span></span> <span data-ttu-id="ee01b-149">Es kann sinnvoll sein, diese Informationen bei Eingang zwischenzuspeichern und gar nicht erst in den dauerhaften Speicher zu schreiben.</span><span class="sxs-lookup"><span data-stu-id="ee01b-149">It might make sense to cache this information as it arrives, and not write it to a persistent store at all.</span></span>  

- <span data-ttu-id="ee01b-150">Um zu verhindern, dass Daten veralten, unterstützen viele Cachinglösungen konfigurierbare Ablaufzeiträume, sodass diese Daten nach dem angegebenen Zeitraum automatisch aus dem Cache entfernt werden.</span><span class="sxs-lookup"><span data-stu-id="ee01b-150">To prevent data from becoming stale, many caching solutions support configurable expiration periods, so that data is automatically removed from the cache after a specified interval.</span></span> <span data-ttu-id="ee01b-151">Möglicherweise müssen Sie die Ablaufzeit für Ihr Szenario anpassen.</span><span class="sxs-lookup"><span data-stu-id="ee01b-151">You may need to tune the expiration time for your scenario.</span></span> <span data-ttu-id="ee01b-152">Daten, die in hohem Maß statisch sind, können länger im Cache verbleiben als flüchtige Daten, die schnell veralten.</span><span class="sxs-lookup"><span data-stu-id="ee01b-152">Data that is highly static can stay in the cache for longer periods than volatile data that may become stale quickly.</span></span>

- <span data-ttu-id="ee01b-153">Wenn die Cachinglösung keinen integrierten Ablauf bietet, müssen Sie möglicherweise einen Hintergrundprozess implementieren, der den Cache gelegentlich leert, damit er nicht ins Unendliche anwächst.</span><span class="sxs-lookup"><span data-stu-id="ee01b-153">If the caching solution doesn't provide built-in expiration, you may need to implement a background process that occasionally sweeps the cache, to prevent it from growing without limits.</span></span> 

- <span data-ttu-id="ee01b-154">Sie können nicht nur Daten aus einer externen Datenquelle zwischenspeichern, Sie können den Cache auch zum Speichern der Ergebnisse komplexer Berechnungen verwenden.</span><span class="sxs-lookup"><span data-stu-id="ee01b-154">Besides caching data from an external data source, you can use caching to save the results of complex computations.</span></span> <span data-ttu-id="ee01b-155">Bevor Sie den Cache jedoch zu diesem Zweck nutzen, instrumentieren Sie die Anwendung, um zu ermitteln, ob sie tatsächlich CPU-gebunden ist.</span><span class="sxs-lookup"><span data-stu-id="ee01b-155">Before you do that, however, instrument the application to determine whether the application is really CPU bound.</span></span>

- <span data-ttu-id="ee01b-156">Es kann nützlich sein, den Cache vorzubereiten, wenn die Anwendung startet.</span><span class="sxs-lookup"><span data-stu-id="ee01b-156">It might be useful to prime the cache when the application starts.</span></span> <span data-ttu-id="ee01b-157">Füllen Sie den Cache mit den Daten auf, die am wahrscheinlichsten verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="ee01b-157">Populate the cache with the data that is most likely to be used.</span></span>

- <span data-ttu-id="ee01b-158">Schließen Sie immer eine Instrumentierung ein, die Cachetreffer und Cachefehler erkennt.</span><span class="sxs-lookup"><span data-stu-id="ee01b-158">Always include instrumentation that detects cache hits and cache misses.</span></span> <span data-ttu-id="ee01b-159">Verwenden Sie diese Informationen, um Cachingrichtlinien anzupassen, beispielsweise dahingehend, welche Daten zwischengespeichert werden und wie lange Daten im Cache aufbewahrt werden, bevor sie ablaufen.</span><span class="sxs-lookup"><span data-stu-id="ee01b-159">Use this information to tune caching policies, such what data to cache, and how long to hold data in the cache before it expires.</span></span>

- <span data-ttu-id="ee01b-160">Wenn unzureichendes Caching zu Engpässen führt, kann durch vermehrtes Caching die Menge an Anforderungen so weit steigen, dass das Web-Front-End überlastet wird.</span><span class="sxs-lookup"><span data-stu-id="ee01b-160">If the lack of caching is a bottleneck, then adding caching may increase the volume of requests so much that the web front end becomes overloaded.</span></span> <span data-ttu-id="ee01b-161">Clients erhalten möglicherweise HTTP 503-Fehler (Dienst nicht verfügbar).</span><span class="sxs-lookup"><span data-stu-id="ee01b-161">Clients may start to receive HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="ee01b-162">Diese sind ein Hinweis darauf, dass Sie das Front-End horizontal hochskalieren sollten.</span><span class="sxs-lookup"><span data-stu-id="ee01b-162">These are an indication that you should scale out the front end.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="ee01b-163">Erkennen des Problems</span><span class="sxs-lookup"><span data-stu-id="ee01b-163">How to detect the problem</span></span>

<span data-ttu-id="ee01b-164">Sie können die folgenden Schritte ausführen, um herauszufinden, ob ein unzureichendes Caching Leistungsprobleme verursacht:</span><span class="sxs-lookup"><span data-stu-id="ee01b-164">You can perform the following steps to help identify whether lack of caching is causing performance problems:</span></span>

1. <span data-ttu-id="ee01b-165">Überprüfen Sie den Anwendungsentwurf.</span><span class="sxs-lookup"><span data-stu-id="ee01b-165">Review the application design.</span></span> <span data-ttu-id="ee01b-166">Erfassen Sie den Bestand aller Datenspeicher, die von der Anwendung verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="ee01b-166">Take an inventory of all the data stores that the application uses.</span></span> <span data-ttu-id="ee01b-167">Ermitteln Sie für jeden Datenspeicher, ob die Anwendung einen Cache verwendet.</span><span class="sxs-lookup"><span data-stu-id="ee01b-167">For each, determine whether the application is using a cache.</span></span> <span data-ttu-id="ee01b-168">Sofern möglich, ermitteln Sie, wie häufig sich die Daten ändern.</span><span class="sxs-lookup"><span data-stu-id="ee01b-168">If possible, determine how frequently the data changes.</span></span> <span data-ttu-id="ee01b-169">Gute Anfangskandidaten für das Caching sind Daten, die sich nur langsam ändern, und statische Verweisdaten, die häufig gelesen werden.</span><span class="sxs-lookup"><span data-stu-id="ee01b-169">Good initial candidates for caching include data that changes slowly, and static reference data that is read frequently.</span></span> 

2. <span data-ttu-id="ee01b-170">Instrumentieren Sie die Anwendung, und überwachen Sie das Livesystem, um herauszufinden, wie häufig die Anwendung Daten abruft oder Informationen berechnet.</span><span class="sxs-lookup"><span data-stu-id="ee01b-170">Instrument the application and monitor the live system to find out how frequently the application retrieves data or calculates information.</span></span>

3. <span data-ttu-id="ee01b-171">Erstellen Sie ein Profil der Anwendung in einer Testumgebung, um Low-Level-Metriken zum Overhead zu erfassen, der mit Datenzugriffsvorgängen oder anderen häufig ausgeführten Berechnungen in Verbindung steht.</span><span class="sxs-lookup"><span data-stu-id="ee01b-171">Profile the application in a test environment to capture low-level metrics about the overhead associated with data access operations or other frequently performed calculations.</span></span>

4. <span data-ttu-id="ee01b-172">Führen Sie einen Auslastungstest in einer Testumgebung durch, um zu ermitteln, wie das System unter normaler und schwerer Workload reagiert.</span><span class="sxs-lookup"><span data-stu-id="ee01b-172">Perform load testing in a test environment to identify how the system responds under a normal workload and under heavy load.</span></span> <span data-ttu-id="ee01b-173">Der Auslastungstest sollte das Datenzugriffsmuster simulieren, das in der Produktionsumgebung mit realistischen Workloads beobachtet wurde.</span><span class="sxs-lookup"><span data-stu-id="ee01b-173">Load testing should simulate the pattern of data access observed in the production environment using realistic workloads.</span></span> 

5. <span data-ttu-id="ee01b-174">Untersuchen Sie die Datenzugriffsstatistiken für die zugrunde liegenden Datenspeicher, und untersuchen Sie, wie häufig die gleichen Datenanforderungen wiederholt werden.</span><span class="sxs-lookup"><span data-stu-id="ee01b-174">Examine the data access statistics for the underlying data stores and review how often the same data requests are repeated.</span></span> 


## <a name="example-diagnosis"></a><span data-ttu-id="ee01b-175">Beispieldiagnose</span><span class="sxs-lookup"><span data-stu-id="ee01b-175">Example diagnosis</span></span>

<span data-ttu-id="ee01b-176">In den folgenden Abschnitten werden diese Schritte auf die zuvor beschriebene Beispielanwendung angewendet.</span><span class="sxs-lookup"><span data-stu-id="ee01b-176">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-the-application-and-monitor-the-live-system"></a><span data-ttu-id="ee01b-177">Instrumentieren der Anwendung und Überwachen des Livesystems</span><span class="sxs-lookup"><span data-stu-id="ee01b-177">Instrument the application and monitor the live system</span></span>

<span data-ttu-id="ee01b-178">Instrumentieren Sie die Anwendung, und überwachen Sie sie, um Informationen zu den spezifischen Anforderungen zu erhalten, die Benutzer während der Produktionsphase der Anwendung senden.</span><span class="sxs-lookup"><span data-stu-id="ee01b-178">Instrument the application and monitor it to get information about the specific requests that users make while the application is in production.</span></span> 

<span data-ttu-id="ee01b-179">Die folgende Abbildung zeigt die Überwachung der Daten, die von [New Relic][NewRelic] während eines Auslastungstests erfasst wurden.</span><span class="sxs-lookup"><span data-stu-id="ee01b-179">The following image shows monitoring data captured by [New Relic][NewRelic] during a load test.</span></span> <span data-ttu-id="ee01b-180">In diesem Fall ist `Person/GetAsync` der einzige ausgeführte HTTP GET-Vorgang.</span><span class="sxs-lookup"><span data-stu-id="ee01b-180">In this case, the only HTTP GET operation performed is `Person/GetAsync`.</span></span> <span data-ttu-id="ee01b-181">In einer Liveproduktionsumgebung kann die Kenntnis der Häufigkeit, mit der jede Anforderung ausgeführt wird, Ihnen Einblicke darin geben, welche Ressourcen zwischengespeichert werden sollten.</span><span class="sxs-lookup"><span data-stu-id="ee01b-181">But in a live production environment, knowing the relative frequency that each request is performed can give you insight into which resources should be cached.</span></span>

![New Relic mit Serveranforderungen aus der CachingDemo-Anwendung][NewRelic-server-requests]

<span data-ttu-id="ee01b-183">Wenn Sie eine detailliertere Analyse benötigen, können Sie einen Profiler verwenden, um in einer Testumgebung (nicht im Produktionssystem) Leistungsdaten auf niedriger Ebene zu erfassen.</span><span class="sxs-lookup"><span data-stu-id="ee01b-183">If you need a deeper analysis, you can use a profiler to capture low-level performance data in a test environment (not the production system).</span></span> <span data-ttu-id="ee01b-184">Sehen Sie sich Metriken wie E/A-Anforderungsraten, Arbeitsspeichernutzung und CPU-Auslastung an.</span><span class="sxs-lookup"><span data-stu-id="ee01b-184">Look at metrics such as I/O request rates, memory usage, and CPU utilization.</span></span> <span data-ttu-id="ee01b-185">Diese Metriken zeigen möglicherweise eine große Anzahl von Anforderungen im Datenspeicher oder Dienst oder eine wiederholte Verarbeitung, die die gleiche Berechnung ausführt.</span><span class="sxs-lookup"><span data-stu-id="ee01b-185">These metrics may show a large number of requests to a data store or service, or repeated processing that performs the same calculation.</span></span> 

### <a name="load-test-the-application"></a><span data-ttu-id="ee01b-186">Auslastungstest der Anwendung</span><span class="sxs-lookup"><span data-stu-id="ee01b-186">Load test the application</span></span>

<span data-ttu-id="ee01b-187">Das folgende Diagramm zeigt die Ergebnisse des Auslastungstests der Beispielanwendung.</span><span class="sxs-lookup"><span data-stu-id="ee01b-187">The following graph shows the results of load testing the sample application.</span></span> <span data-ttu-id="ee01b-188">Der Auslastungstest simuliert eine Schrittauslastung von bis zu 800 Benutzern, die eine typische Reihenfolge von Vorgängen ausführen.</span><span class="sxs-lookup"><span data-stu-id="ee01b-188">The load test simulates a step load of up to 800 users performing a typical series of operations.</span></span> 

![Leistungstestergebnisse für das Szenario ohne Cache][Performance-Load-Test-Results-Uncached]

<span data-ttu-id="ee01b-190">Die Anzahl von pro Sekunden ausgeführten erfolgreichen Tests stabilisiert sich auf einem Niveau, und dadurch werden weitere Anforderungen verlangsamt.</span><span class="sxs-lookup"><span data-stu-id="ee01b-190">The number of successful tests performed each second reaches a plateau, and additional requests are slowed as a result.</span></span> <span data-ttu-id="ee01b-191">Die durchschnittliche Testzeit steigt stetig mit der Workload.</span><span class="sxs-lookup"><span data-stu-id="ee01b-191">The average test time steadily increases with the workload.</span></span> <span data-ttu-id="ee01b-192">Die Antwortzeit sinkt, sobald die Benutzerauslastung einen Spitzenwert erreicht.</span><span class="sxs-lookup"><span data-stu-id="ee01b-192">The response time levels off once the user load peaks.</span></span>

### <a name="examine-data-access-statistics"></a><span data-ttu-id="ee01b-193">Untersuchen der Datenzugriffsstatistiken</span><span class="sxs-lookup"><span data-stu-id="ee01b-193">Examine data access statistics</span></span>

<span data-ttu-id="ee01b-194">Datenzugriffsstatistiken und andere von einem Datenspeicher bereitgestellte Informationen, z.B. dazu, welche Abfragen am häufigsten wiederholt werden, können sehr nützlich sein.</span><span class="sxs-lookup"><span data-stu-id="ee01b-194">Data access statistics and other information provided by a data store can give useful information, such as which queries are repeated most frequently.</span></span> <span data-ttu-id="ee01b-195">In Microsoft SQL Server bietet die `sys.dm_exec_query_stats`-Verwaltungssicht statistische Informationen zu kürzlich ausgeführten Abfragen.</span><span class="sxs-lookup"><span data-stu-id="ee01b-195">For example, in Microsoft SQL Server, the `sys.dm_exec_query_stats` management view has statistical information for recently executed queries.</span></span> <span data-ttu-id="ee01b-196">Der Text für jede Abfrage ist in der `sys.dm_exec-query_plan`-Sicht verfügbar.</span><span class="sxs-lookup"><span data-stu-id="ee01b-196">The text for each query is available in the `sys.dm_exec-query_plan` view.</span></span> <span data-ttu-id="ee01b-197">Sie können ein Tool wie z.B. SQL Server Management Studio verwenden, um die folgende SQL-Abfrage auszuführen und zu ermitteln, wie häufig Abfragen ausgeführt werden.</span><span class="sxs-lookup"><span data-stu-id="ee01b-197">You can use a tool such as SQL Server Management Studio to run the following SQL query and determine how frequently queries are performed.</span></span>

```SQL
SELECT UseCounts, Text, Query_Plan
FROM sys.dm_exec_cached_plans
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
CROSS APPLY sys.dm_exec_query_plan(plan_handle)
```

<span data-ttu-id="ee01b-198">Die `UseCount`-Spalte in den Ergebnissen gibt an, wie häufig jede Abfrage ausgeführt wird.</span><span class="sxs-lookup"><span data-stu-id="ee01b-198">The `UseCount` column in the results indicates how frequently each query is run.</span></span> <span data-ttu-id="ee01b-199">Die folgende Abbildung zeigt, dass die dritte Abfrage über 250.000-mal ausgeführt wurde – deutlich mehr als jede andere Abfrage.</span><span class="sxs-lookup"><span data-stu-id="ee01b-199">The following image shows that the third query was run more than 250,000 times, significantly more than any other query.</span></span>

![Ergebnisse der Abfragen der dynamischen Verwaltungssichten in SQL Server Management Server][Dynamic-Management-Views]

<span data-ttu-id="ee01b-201">Dies ist die SQL-Abfrage, die so viele Datenbankanforderungen verursacht:</span><span class="sxs-lookup"><span data-stu-id="ee01b-201">Here is the SQL query that is causing so many database requests:</span></span> 

```SQL
(@p__linq__0 int)SELECT TOP (2)
[Extent1].[BusinessEntityId] AS [BusinessEntityId],
[Extent1].[FirstName] AS [FirstName],
[Extent1].[LastName] AS [LastName]
FROM [Person].[Person] AS [Extent1]
WHERE [Extent1].[BusinessEntityId] = @p__linq__0
```

<span data-ttu-id="ee01b-202">Dies ist die Abfrage, die Entity Framework in der oben gezeigten `GetByIdAsync`-Methode generiert.</span><span class="sxs-lookup"><span data-stu-id="ee01b-202">This is the query that Entity Framework generates in `GetByIdAsync` method shown earlier.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="ee01b-203">Implementieren der Lösung und Überprüfen des Ergebnisses</span><span class="sxs-lookup"><span data-stu-id="ee01b-203">Implement the solution and verify the result</span></span>

<span data-ttu-id="ee01b-204">Nachdem Sie einen Cache implementiert haben, wiederholen Sie die Auslastungstests, und vergleichen Sie die Ergebnisse mit den vorherigen Auslastungstests ohne Cache.</span><span class="sxs-lookup"><span data-stu-id="ee01b-204">After you incorporate a cache, repeat the load tests and compare the results to the earlier load tests without a cache.</span></span> <span data-ttu-id="ee01b-205">Dies sind die Ergebnisse des Auslastungstests nach dem Hinzufügen eines Caches zur Beispielanwendung.</span><span class="sxs-lookup"><span data-stu-id="ee01b-205">Here are the load test results after adding a cache to the sample application.</span></span>

![Leistungstestergebnisse für das Szenario mit Cache][Performance-Load-Test-Results-Cached]

<span data-ttu-id="ee01b-207">Der Menge an erfolgreichen Tests stabilisiert sich weiterhin auf einem Niveau, jedoch bei einer höheren Benutzerauslastung.</span><span class="sxs-lookup"><span data-stu-id="ee01b-207">The volume of successful tests still reaches a plateau, but at a higher user load.</span></span> <span data-ttu-id="ee01b-208">Die Anforderungsrate bei dieser Auslastung ist deutlich höher als vorher.</span><span class="sxs-lookup"><span data-stu-id="ee01b-208">The request rate at this load is significantly higher than earlier.</span></span> <span data-ttu-id="ee01b-209">Die durchschnittliche Testzeit steigt weiterhin mit der Auslastung, aber die maximale Antwortzeit beträgt 0,05 ms, im Vergleich zu 1 ms &mdash;eine 20&times; fache Verbesserung.</span><span class="sxs-lookup"><span data-stu-id="ee01b-209">Average test time still increases with load, but the maximum response time is 0.05 ms, compared with 1ms earlier &mdash; a 20&times; improvement.</span></span> 

## <a name="related-resources"></a><span data-ttu-id="ee01b-210">Zugehörige Ressourcen</span><span class="sxs-lookup"><span data-stu-id="ee01b-210">Related resources</span></span>

- <span data-ttu-id="ee01b-211">[API implementation best practices][api-implementation] (Bewährte Methoden für die API-Implementierung)</span><span class="sxs-lookup"><span data-stu-id="ee01b-211">[API implementation best practices][api-implementation]</span></span>
- <span data-ttu-id="ee01b-212">[Cache-Aside Pattern][cache-aside-pattern] (Cachefremdes Muster)</span><span class="sxs-lookup"><span data-stu-id="ee01b-212">[Cache-Aside Pattern][cache-aside-pattern]</span></span>
- <span data-ttu-id="ee01b-213">[Caching best practices][caching-guidance] (Bewährte Methoden für das Caching)</span><span class="sxs-lookup"><span data-stu-id="ee01b-213">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="ee01b-214">[Circuit Breaker pattern][circuit-breaker] (Schutzschaltermuster)</span><span class="sxs-lookup"><span data-stu-id="ee01b-214">[Circuit Breaker pattern][circuit-breaker]</span></span>

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