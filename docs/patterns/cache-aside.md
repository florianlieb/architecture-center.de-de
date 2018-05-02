---
title: Cachefremd
description: Daten bei Bedarf aus einem Datenspeicher in einen Cache laden
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: d4d7c9dcd612c780e3e494509a57b6b4a0144423
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/16/2018
---
# <a name="cache-aside-pattern"></a><span data-ttu-id="4fc26-104">Cachefremdes Muster</span><span class="sxs-lookup"><span data-stu-id="4fc26-104">Cache-Aside pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="4fc26-105">Laden Sie Daten bei Bedarf aus einem Datenspeicher in einen Cache.</span><span class="sxs-lookup"><span data-stu-id="4fc26-105">Load data on demand into a cache from a data store.</span></span> <span data-ttu-id="4fc26-106">Dies kann die Leistung verbessern und trägt zudem dazu bei, die Konsistenz zwischen den Daten im Cache und den Daten im zugrunde liegenden Datenspeicher aufrechtzuerhalten.</span><span class="sxs-lookup"><span data-stu-id="4fc26-106">This can improve performance and also helps to maintain consistency between data held in the cache and data in the underlying data store.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="4fc26-107">Kontext und Problem</span><span class="sxs-lookup"><span data-stu-id="4fc26-107">Context and problem</span></span>

<span data-ttu-id="4fc26-108">Anwendungen verwenden einen Cache, um wiederholte Zugriffe auf Informationen in einem Datenspeicher zu verbessern.</span><span class="sxs-lookup"><span data-stu-id="4fc26-108">Applications use a cache to improve repeated access to information held in a data store.</span></span> <span data-ttu-id="4fc26-109">Allerdings kann nicht erwartet werden, dass zwischengespeicherte Daten immer vollständig konsistent mit den Daten im Datenspeicher sind.</span><span class="sxs-lookup"><span data-stu-id="4fc26-109">However, it's impractical to expect that cached data will always be completely consistent with the data in the data store.</span></span> <span data-ttu-id="4fc26-110">Anwendungen sollten eine Strategie implementieren, mit der sichergestellt wird, dass die Daten im Cache so aktuell wie möglich sind. Sie sollten jedoch auch Situationen erkennen und handhaben können, die auftreten, wenn die Daten im Cache veraltet sind.</span><span class="sxs-lookup"><span data-stu-id="4fc26-110">Applications should implement a strategy that helps to ensure that the data in the cache is as up-to-date as possible, but can also detect and handle situations that arise when the data in the cache has become stale.</span></span>

## <a name="solution"></a><span data-ttu-id="4fc26-111">Lösung</span><span class="sxs-lookup"><span data-stu-id="4fc26-111">Solution</span></span>

<span data-ttu-id="4fc26-112">Viele kommerzielle Cachesysteme bieten Read-Through- und Write-Through-/Write-Behind-Vorgänge.</span><span class="sxs-lookup"><span data-stu-id="4fc26-112">Many commercial caching systems provide read-through and write-through/write-behind operations.</span></span> <span data-ttu-id="4fc26-113">In diesen Systemen ruft eine Anwendung Daten mithilfe von Verweisen auf den Cache ab.</span><span class="sxs-lookup"><span data-stu-id="4fc26-113">In these systems, an application retrieves data by referencing the cache.</span></span> <span data-ttu-id="4fc26-114">Wenn die Daten nicht im Cache enthalten sind, werden sie aus dem Datenspeicher abgerufen und dem Cache hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="4fc26-114">If the data isn't in the cache, it's retrieved from the data store and added to the cache.</span></span> <span data-ttu-id="4fc26-115">Änderungen an den im Cache gespeicherten Daten werden automatisch auch in den Datenspeicher zurückgeschrieben.</span><span class="sxs-lookup"><span data-stu-id="4fc26-115">Any modifications to data held in the cache are automatically written back to the data store as well.</span></span>

<span data-ttu-id="4fc26-116">Bei Caches, die diese Funktionalität nicht bereitstellen, ist es die Aufgabe der Anwendungen, die den Cache verwenden, die Daten zu verwalten.</span><span class="sxs-lookup"><span data-stu-id="4fc26-116">For caches that don't provide this functionality, it's the responsibility of the applications that use the cache to maintain the data.</span></span>

<span data-ttu-id="4fc26-117">Eine Anwendung kann die Funktionalität eines Read-Through-Cache durch Implementieren einer Strategie mit dem cachefremden Muster emulieren.</span><span class="sxs-lookup"><span data-stu-id="4fc26-117">An application can emulate the functionality of read-through caching by implementing the cache-aside strategy.</span></span> <span data-ttu-id="4fc26-118">Mit dieser Strategie werden Daten bei Bedarf in den Cache geladen.</span><span class="sxs-lookup"><span data-stu-id="4fc26-118">This strategy loads data into the cache on demand.</span></span> <span data-ttu-id="4fc26-119">Die Abbildung veranschaulicht die Verwendung des cachefremden Musters zum Speichern von Daten im Cache.</span><span class="sxs-lookup"><span data-stu-id="4fc26-119">The figure illustrates using the Cache-Aside pattern to store data in the cache.</span></span>

![Verwendung des cachefremden Musters zum Speichern von Daten im Cache](./_images/cache-aside-diagram.png)


<span data-ttu-id="4fc26-121">Aktualisiert eine Anwendung Informationen, kann sie die Write-Through-Strategie einsetzen, indem sie die Änderung am Datenspeicher vornimmt und das entsprechende Element im Cache ungültig macht.</span><span class="sxs-lookup"><span data-stu-id="4fc26-121">If an application updates information, it can follow the write-through strategy by making the modification to the data store, and by invalidating the corresponding item in the cache.</span></span>

<span data-ttu-id="4fc26-122">Wenn das Element das nächste Mal benötigt wird, führt die Strategie mit dem cachefremden Muster dazu, dass die aktualisierten Daten aus dem Datenspeicher abgerufen und wieder zum Cache hinzugefügt werden.</span><span class="sxs-lookup"><span data-stu-id="4fc26-122">When the item is next required, using the cache-aside strategy will cause the updated data to be retrieved from the data store and added back into the cache.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="4fc26-123">Probleme und Überlegungen</span><span class="sxs-lookup"><span data-stu-id="4fc26-123">Issues and considerations</span></span>

<span data-ttu-id="4fc26-124">Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:</span><span class="sxs-lookup"><span data-stu-id="4fc26-124">Consider the following points when deciding how to implement this pattern:</span></span> 

<span data-ttu-id="4fc26-125">**Lebensdauer der zwischengespeicherten Daten**.</span><span class="sxs-lookup"><span data-stu-id="4fc26-125">**Lifetime of cached data**.</span></span> <span data-ttu-id="4fc26-126">Viele Caches implementieren eine Ablaufrichtlinie, mit der die Daten ungültig gemacht und aus dem Cache entfernt werden, wenn für einen angegebenen Zeitraum nicht darauf zugegriffen wurde.</span><span class="sxs-lookup"><span data-stu-id="4fc26-126">Many caches implement an expiration policy that invalidates data and removes it from the cache if it's not accessed for a specified period.</span></span> <span data-ttu-id="4fc26-127">Damit das cachefremde Muster wirksam ist, stellen Sie sicher, dass die Ablaufrichtlinie zum Zugriffsmuster für Anwendungen passt, die die Daten verwenden.</span><span class="sxs-lookup"><span data-stu-id="4fc26-127">For cache-aside to be effective, ensure that the expiration policy matches the pattern of access for applications that use the data.</span></span> <span data-ttu-id="4fc26-128">Legen Sie keinen zu kurzen Ablaufzeitraum fest. Dies könnte dazu führen, dass Anwendungen Daten kontinuierlich aus dem Datenspeicher abrufen und dem Cache hinzufügen.</span><span class="sxs-lookup"><span data-stu-id="4fc26-128">Don't make the expiration period too short because this can cause applications to continually retrieve data from the data store and add it to the cache.</span></span> <span data-ttu-id="4fc26-129">Legen Sie auch keinen zu langen Ablaufzeitraum fest, damit Sie keine veralteten Daten im Cache haben.</span><span class="sxs-lookup"><span data-stu-id="4fc26-129">Similarly, don't make the expiration period so long that the cached data is likely to become stale.</span></span> <span data-ttu-id="4fc26-130">Denken Sie daran, dass das Zwischenspeichern für relativ statische Daten oder für Daten, die häufig gelesen werden, am effektivsten ist.</span><span class="sxs-lookup"><span data-stu-id="4fc26-130">Remember that caching is most effective for relatively static data, or data that is read frequently.</span></span>

<span data-ttu-id="4fc26-131">**Entfernen von Daten**.</span><span class="sxs-lookup"><span data-stu-id="4fc26-131">**Evicting data**.</span></span> <span data-ttu-id="4fc26-132">Die meisten Caches haben im Vergleich mit dem Datenspeicher, aus dem die Daten stammen, eine beschränkte Größe, und sie müssen Daten ggf. entfernen.</span><span class="sxs-lookup"><span data-stu-id="4fc26-132">Most caches have a limited size compared to the data store where the data originates, and they'll evict data if necessary.</span></span> <span data-ttu-id="4fc26-133">In den meisten Caches werden dann die am längsten nicht mehr verwendeten Elemente entfernt, aber dies kann möglicherweise angepasst werden.</span><span class="sxs-lookup"><span data-stu-id="4fc26-133">Most caches adopt a least-recently-used policy for selecting items to evict, but this might be customizable.</span></span> <span data-ttu-id="4fc26-134">Konfigurieren Sie die globale Ablaufeigenschaft und andere Eigenschaften des Cache sowie das Ablaufdatum der einzelnen Elemente im Cache, um sicherzustellen, dass der Cache kostengünstig ist.</span><span class="sxs-lookup"><span data-stu-id="4fc26-134">Configure the global expiration property and other properties of the cache, and the expiration property of each cached item, to ensure that the cache is cost effective.</span></span> <span data-ttu-id="4fc26-135">Es ist nicht immer angebracht, eine globale Entfernungsrichtlinie auf jedes Element im Cache anzuwenden.</span><span class="sxs-lookup"><span data-stu-id="4fc26-135">It isn't always appropriate to apply a global eviction policy to every item in the cache.</span></span> <span data-ttu-id="4fc26-136">Wenn es z.B. mit viel Aufwand verbunden ist, ein Element im Cache aus dem Datenspeicher abzurufen, kann es von Vorteil sein, dieses Element im Cache zu belassen und stattdessen häufiger verwendete, jedoch weniger aufwändige Elemente zu löschen.</span><span class="sxs-lookup"><span data-stu-id="4fc26-136">For example, if a cached item is very expensive to retrieve from the data store, it can be beneficial to keep this item in the cache at the expense of more frequently accessed but less costly items.</span></span>

<span data-ttu-id="4fc26-137">**Vorbereiten des Cache**.</span><span class="sxs-lookup"><span data-stu-id="4fc26-137">**Priming the cache**.</span></span> <span data-ttu-id="4fc26-138">Viele Lösungen füllen den Cache vorab mit den Daten auf, die eine Anwendung wahrscheinlich als Teil der Verarbeitung beim Starten benötigt.</span><span class="sxs-lookup"><span data-stu-id="4fc26-138">Many solutions prepopulate the cache with the data that an application is likely to need as part of the startup processing.</span></span> <span data-ttu-id="4fc26-139">Das cachefremde Muster kann dennoch nützlich sein, wenn einige dieser Daten abgelaufen sind oder entfernt werden.</span><span class="sxs-lookup"><span data-stu-id="4fc26-139">The Cache-Aside pattern can still be useful if some of this data expires or is evicted.</span></span>

<span data-ttu-id="4fc26-140">**Konsistenz**.</span><span class="sxs-lookup"><span data-stu-id="4fc26-140">**Consistency**.</span></span> <span data-ttu-id="4fc26-141">Durch Implementieren des cachefremden Musters ist die Konsistenz zwischen dem Datenspeicher und dem Cache nicht garantiert.</span><span class="sxs-lookup"><span data-stu-id="4fc26-141">Implementing the Cache-Aside pattern doesn't guarantee consistency between the data store and the cache.</span></span> <span data-ttu-id="4fc26-142">Ein Element im Datenspeicher kann jedoch jederzeit von einem externen Prozess geändert werden, und diese Änderung wird möglicherweise erst im Cache wiedergegeben, wenn das Element das nächste Mal geladen wird.</span><span class="sxs-lookup"><span data-stu-id="4fc26-142">An item in the data store can be changed at any time by an external process, and this change might not be reflected in the cache until the next time the item is loaded.</span></span> <span data-ttu-id="4fc26-143">In einem System, das Daten über Datenspeicher repliziert, kann dies ein ernsthaftes Problem werden, wenn die Synchronisierung häufig auftritt.</span><span class="sxs-lookup"><span data-stu-id="4fc26-143">In a system that replicates data across data stores, this problem can become serious if synchronization occurs frequently.</span></span>

<span data-ttu-id="4fc26-144">**Lokales (speicherinternes) Zwischenspeichern**.</span><span class="sxs-lookup"><span data-stu-id="4fc26-144">**Local (in-memory) caching**.</span></span> <span data-ttu-id="4fc26-145">Ein Cache kann für eine Anwendungsinstanz lokal und im Speicher gespeichert sein.</span><span class="sxs-lookup"><span data-stu-id="4fc26-145">A cache could be local to an application instance and stored in-memory.</span></span> <span data-ttu-id="4fc26-146">Das cachefremde Muster kann in dieser Umgebung nützlich sein, wenn eine Anwendung wiederholt auf die gleichen Daten zugreift.</span><span class="sxs-lookup"><span data-stu-id="4fc26-146">Cache-aside can be useful in this environment if an application repeatedly accesses the same data.</span></span> <span data-ttu-id="4fc26-147">Allerdings ist ein lokaler Cache privat. Daher können verschiedene Anwendungsinstanzen jeweils über eine Kopie der gleichen zwischengespeicherten Daten verfügen.</span><span class="sxs-lookup"><span data-stu-id="4fc26-147">However, a local cache is private and so different application instances could each have a copy of the same cached data.</span></span> <span data-ttu-id="4fc26-148">Diese Daten können schnell zwischen Caches inkonsistent werden, sodass es möglicherweise erforderlich ist, dass Daten in einem privaten Cache ablaufen und häufiger aktualisiert werden.</span><span class="sxs-lookup"><span data-stu-id="4fc26-148">This data could quickly become inconsistent between caches, so it might be necessary to expire data held in a private cache and refresh it more frequently.</span></span> <span data-ttu-id="4fc26-149">In diesen Szenarien sollten Sie die Verwendung eines freigegebenen oder verteilten Mechanismus zum Zwischenspeichern prüfen.</span><span class="sxs-lookup"><span data-stu-id="4fc26-149">In these scenarios, consider investigating the use of a shared or a distributed caching mechanism.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="4fc26-150">Verwendung dieses Musters</span><span class="sxs-lookup"><span data-stu-id="4fc26-150">When to use this pattern</span></span>

<span data-ttu-id="4fc26-151">Verwenden Sie dieses Muster in folgenden Fällen:</span><span class="sxs-lookup"><span data-stu-id="4fc26-151">Use this pattern when:</span></span>

- <span data-ttu-id="4fc26-152">Ein Cache stellt keine nativen Read-Through- und Write-Through-Vorgänge bereit.</span><span class="sxs-lookup"><span data-stu-id="4fc26-152">A cache doesn't provide native read-through and write-through operations.</span></span>
- <span data-ttu-id="4fc26-153">Der Ressourcenbedarf ist nicht vorhersehbar.</span><span class="sxs-lookup"><span data-stu-id="4fc26-153">Resource demand is unpredictable.</span></span> <span data-ttu-id="4fc26-154">Mit diesem Muster können Anwendungen Daten bei Bedarf laden.</span><span class="sxs-lookup"><span data-stu-id="4fc26-154">This pattern enables applications to load data on demand.</span></span> <span data-ttu-id="4fc26-155">Es werden im Voraus keine Annahmen darüber getroffen, welche Daten eine Anwendung benötigt.</span><span class="sxs-lookup"><span data-stu-id="4fc26-155">It makes no assumptions about which data an application will require in advance.</span></span>

<span data-ttu-id="4fc26-156">Dieses Muster ist in folgenden Fällen möglicherweise nicht geeignet:</span><span class="sxs-lookup"><span data-stu-id="4fc26-156">This pattern might not be suitable:</span></span>

- <span data-ttu-id="4fc26-157">Wenn das zwischengespeicherte Dataset statisch ist.</span><span class="sxs-lookup"><span data-stu-id="4fc26-157">When the cached data set is static.</span></span> <span data-ttu-id="4fc26-158">Wenn die Daten in den verfügbaren Cachespeicher passen, bereiten Sie den Cache beim Start mit den Daten vor, und wenden Sie eine Richtlinie an, die verhindert, dass die Daten ablaufen.</span><span class="sxs-lookup"><span data-stu-id="4fc26-158">If the data will fit into the available cache space, prime the cache with the data on startup and apply a policy that prevents the data from expiring.</span></span>
- <span data-ttu-id="4fc26-159">Zum Zwischenspeichern von Sitzungszustandsinformationen in einer Webanwendung, die in einer Webfarm gehostet wird.</span><span class="sxs-lookup"><span data-stu-id="4fc26-159">For caching session state information in a web application hosted in a web farm.</span></span> <span data-ttu-id="4fc26-160">In dieser Umgebung sollten Sie vermeiden, dass Abhängigkeiten basierend auf der Client/Server-Affinität entstehen.</span><span class="sxs-lookup"><span data-stu-id="4fc26-160">In this environment, you should avoid introducing dependencies based on client-server affinity.</span></span>

## <a name="example"></a><span data-ttu-id="4fc26-161">Beispiel</span><span class="sxs-lookup"><span data-stu-id="4fc26-161">Example</span></span>

<span data-ttu-id="4fc26-162">In Microsoft Azure können Sie Azure Redis Cache verwenden, um einen verteilten Cache zu erstellen, der von mehreren Instanzen einer Anwendung gemeinsam genutzt werden kann.</span><span class="sxs-lookup"><span data-stu-id="4fc26-162">In Microsoft Azure you can use Azure Redis Cache to create a distributed cache that can be shared by multiple instances of an application.</span></span> 

<span data-ttu-id="4fc26-163">Rufen Sie die statische `Connect`-Methode auf, und übergeben Sie die Verbindungszeichenfolge, um eine Verbindung mit einer Azure Redis Cache-Instanz herzustellen.</span><span class="sxs-lookup"><span data-stu-id="4fc26-163">To connect to an Azure Redis Cache instance, call the static `Connect` method and pass in the connection string.</span></span> <span data-ttu-id="4fc26-164">Die Methode gibt ein `ConnectionMultiplexer`-Element zurück, das die Verbindung darstellt.</span><span class="sxs-lookup"><span data-stu-id="4fc26-164">The method returns a `ConnectionMultiplexer` that represents the connection.</span></span> <span data-ttu-id="4fc26-165">Ein Ansatz zur Freigabe einer `ConnectionMultiplexer` -Instanz in Ihrer Anwendung ist das Verwenden einer statischen Eigenschaft, die wie im folgenden Beispiel eine verbundene Instanz zurückgibt.</span><span class="sxs-lookup"><span data-stu-id="4fc26-165">One approach to sharing a `ConnectionMultiplexer` instance in your application is to have a static property that returns a connected instance, similar to the following example.</span></span> <span data-ttu-id="4fc26-166">Dieser Ansatz ist eine threadsichere Möglichkeit, um nur eine einzelne verbundene Instanz zu initialisieren.</span><span class="sxs-lookup"><span data-stu-id="4fc26-166">This approach provides a thread-safe way to initialize only a single connected instance.</span></span>

```csharp
private static ConnectionMultiplexer Connection;

// Redis Connection string info
private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
{
    string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
    return ConnectionMultiplexer.Connect(cacheConnection);
});

public static ConnectionMultiplexer Connection => lazyConnection.Value;
```

<span data-ttu-id="4fc26-167">Die `GetMyEntityAsync`-Methode im folgenden Codebeispiel zeigt eine Implementierung des cachefremden Musters basierend auf Azure Redis Cache.</span><span class="sxs-lookup"><span data-stu-id="4fc26-167">The `GetMyEntityAsync` method in the following code example shows an implementation of the Cache-Aside pattern based on Azure Redis Cache.</span></span> <span data-ttu-id="4fc26-168">Diese Methode ruft mit dem Read-Through-Ansatz ein Objekt aus dem Cache ab.</span><span class="sxs-lookup"><span data-stu-id="4fc26-168">This method retrieves an object from the cache using the read-through approach.</span></span>

<span data-ttu-id="4fc26-169">Ein Objekt wird mit einer ganzzahligen ID als Schlüssel identifiziert.</span><span class="sxs-lookup"><span data-stu-id="4fc26-169">An object is identified by using an integer ID as the key.</span></span> <span data-ttu-id="4fc26-170">Die `GetMyEntityAsync`-Methode versucht, ein Element mit diesem Schlüssel aus dem Cache abzurufen.</span><span class="sxs-lookup"><span data-stu-id="4fc26-170">The `GetMyEntityAsync` method tries to retrieve an item with this key from the cache.</span></span> <span data-ttu-id="4fc26-171">Wenn ein übereinstimmendes Element gefunden wird, wird es zurückgegeben.</span><span class="sxs-lookup"><span data-stu-id="4fc26-171">If a matching item is found, it's returned.</span></span> <span data-ttu-id="4fc26-172">Wenn im Cache keine Übereinstimmung vorhanden ist, ruft die `GetMyEntityAsync`-Methode das Objekt aus einem Datenspeicher ab, fügt es dem Cache hinzu und gibt es zurück.</span><span class="sxs-lookup"><span data-stu-id="4fc26-172">If there's no match in the cache, the `GetMyEntityAsync` method retrieves the object from a data store, adds it to the cache, and then returns it.</span></span> <span data-ttu-id="4fc26-173">Der Code, der die Daten tatsächlich aus dem Datenspeicher liest, ist hier nicht dargestellt, da er vom Datenspeicher abhängt.</span><span class="sxs-lookup"><span data-stu-id="4fc26-173">The code that actually reads the data from the data store is not shown here, because it depends on the data store.</span></span> <span data-ttu-id="4fc26-174">Beachten Sie, dass für das zwischengespeicherte Element konfiguriert ist, dass es abläuft. Dadurch wird verhindert, dass es veraltet ist, wenn es an anderer Stelle aktualisiert wird.</span><span class="sxs-lookup"><span data-stu-id="4fc26-174">Note that the cached item is configured to expire to prevent it from becoming stale if it's updated elsewhere.</span></span>


```csharp
// Set five minute expiration as a default
private const double DefaultExpirationTimeInMinutes = 5.0;

public async Task<MyEntity> GetMyEntityAsync(int id)
{
  // Define a unique key for this method and its parameters.
  var key = $"MyEntity:{id}";
  var cache = Connection.GetDatabase();
  
  // Try to get the entity from the cache.
  var json = await cache.StringGetAsync(key).ConfigureAwait(false);
  var value = string.IsNullOrWhiteSpace(json) 
                ? default(MyEntity) 
                : JsonConvert.DeserializeObject<MyEntity>(json);
  
  if (value == null) // Cache miss
  {
    // If there's a cache miss, get the entity from the original store and cache it.
    // Code has been omitted because it's data store dependent.  
    value = ...;

    // Avoid caching a null value.
    if (value != null)
    {
      // Put the item in the cache with a custom expiration time that 
      // depends on how critical it is to have stale data.
      await cache.StringSetAsync(key, JsonConvert.SerializeObject(value)).ConfigureAwait(false);
      await cache.KeyExpireAsync(key, TimeSpan.FromMinutes(DefaultExpirationTimeInMinutes)).ConfigureAwait(false);
    }
  }

  return value;
}
```

>  <span data-ttu-id="4fc26-175">Die Beispiele verwenden die Azure Redis Cache-API, um auf den Speicher zuzugreifen und Informationen aus dem Cache abzurufen.</span><span class="sxs-lookup"><span data-stu-id="4fc26-175">The examples use the Azure Redis Cache API to access the store and retrieve information from the cache.</span></span> <span data-ttu-id="4fc26-176">Weitere Informationen finden Sie unter [Verwenden von Azure Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) und [Gewusst wie: Erstellen einer Web-App mit Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto)</span><span class="sxs-lookup"><span data-stu-id="4fc26-176">For more information, see [Using Microsoft Azure Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) and [How to create a Web App with Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto)</span></span>

<span data-ttu-id="4fc26-177">Die unten gezeigte `UpdateEntityAsync`-Methode veranschaulicht, wie ein Objekt im Cache für ungültig erklärt wird, wenn der Wert von der Anwendung geändert wird.</span><span class="sxs-lookup"><span data-stu-id="4fc26-177">The `UpdateEntityAsync` method shown below demonstrates how to invalidate an object in the cache when the value is changed by the application.</span></span> <span data-ttu-id="4fc26-178">Der Code aktualisiert den ursprünglichen Datenspeicher und entfernt dann das zwischengespeicherte Element aus dem Cache.</span><span class="sxs-lookup"><span data-stu-id="4fc26-178">The code updates the original data store and then removes the cached item from the cache.</span></span>

```csharp
public async Task UpdateEntityAsync(MyEntity entity)
{
    // Update the object in the original data store.
    await this.store.UpdateEntityAsync(entity).ConfigureAwait(false); 

    // Invalidate the current cache object.
    var cache = Connection.GetDatabase();
    var id = entity.Id;
    var key = $"MyEntity:{id}"; // The key for the cached object.
    await cache.KeyDeleteAsync(key).ConfigureAwait(false); // Delete this key from the cache.
}
```

> [!NOTE]
> <span data-ttu-id="4fc26-179">Die Reihenfolge der Schritte ist wichtig.</span><span class="sxs-lookup"><span data-stu-id="4fc26-179">The order of the steps is important.</span></span> <span data-ttu-id="4fc26-180">Aktualisieren Sie den Datenspeicher, *bevor* Sie das Element aus dem Cache entfernen.</span><span class="sxs-lookup"><span data-stu-id="4fc26-180">Update the data store *before* removing the item from the cache.</span></span> <span data-ttu-id="4fc26-181">Wenn Sie zuerst das zwischengespeicherte Element entfernen, entsteht ein kleines Zeitfenster, in dem ein Client das Element abrufen kann, bevor der Datenspeicher aktualisiert wird.</span><span class="sxs-lookup"><span data-stu-id="4fc26-181">If you remove the cached item first, there is a small window of time when a client might fetch the item before the data store is updated.</span></span> <span data-ttu-id="4fc26-182">Dies führt zu einem Cachefehler (da das Element aus dem Cache entfernt wurde). Dadurch wird die frühere Version des Elements aus dem Datenspeicher abgerufen und wieder im Cache hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="4fc26-182">That will result in a cache miss (because the item was removed from the cache), causing the earlier version of the item to be fetched from the data store and added back into the cache.</span></span> <span data-ttu-id="4fc26-183">Das Ergebnis sind veraltete Cachedaten.</span><span class="sxs-lookup"><span data-stu-id="4fc26-183">The result will be stale cache data.</span></span>


## <a name="related-guidance"></a><span data-ttu-id="4fc26-184">Verwandte Leitfäden</span><span class="sxs-lookup"><span data-stu-id="4fc26-184">Related guidance</span></span> 

<span data-ttu-id="4fc26-185">Die folgenden Informationen sind unter Umständen auch relevant, wenn dieses Muster implementiert wird:</span><span class="sxs-lookup"><span data-stu-id="4fc26-185">The following information may be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="4fc26-186">[Caching Guidance (Leitfaden zum Caching)](https://docs.microsoft.com/azure/architecture/best-practices/caching).</span><span class="sxs-lookup"><span data-stu-id="4fc26-186">[Caching Guidance](https://docs.microsoft.com/azure/architecture/best-practices/caching).</span></span> <span data-ttu-id="4fc26-187">Enthält weitere Informationen zum Zwischenspeichern von Daten in einer Cloudlösung und die Probleme, die Sie bedenken sollten, wenn Sie einen Cache implementieren.</span><span class="sxs-lookup"><span data-stu-id="4fc26-187">Provides additional information on how you can cache data in a cloud solution, and the issues that you should consider when you implement a cache.</span></span>

- <span data-ttu-id="4fc26-188">[Data Consistency Primer (Grundlagen der Datenkonsistenz)](https://msdn.microsoft.com/library/dn589800.aspx).</span><span class="sxs-lookup"><span data-stu-id="4fc26-188">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="4fc26-189">Cloudanwendungen verwenden in der Regel Daten, die auf Datenspeicher verteilt sind.</span><span class="sxs-lookup"><span data-stu-id="4fc26-189">Cloud applications typically use data that's spread across data stores.</span></span> <span data-ttu-id="4fc26-190">Das Verwalten und Erhalten der Datenkonsistenz in dieser Umgebung ist ein wichtiger Aspekt des Systems, insbesondere die Probleme mit Parallelität und Dienstverfügbarkeit, die auftreten können.</span><span class="sxs-lookup"><span data-stu-id="4fc26-190">Managing and maintaining data consistency in this environment is a critical aspect of the system, particularly the concurrency and availability issues that can arise.</span></span> <span data-ttu-id="4fc26-191">Dieser Artikel erläutert Probleme im Zusammenhang mit der Konsistenz verteilter Daten und fasst zusammen, wie eine Anwendung letztlich Konsistenz implementieren kann, um die Verfügbarkeit von Daten beizubehalten.</span><span class="sxs-lookup"><span data-stu-id="4fc26-191">This primer describes issues about consistency across distributed data, and summarizes how an application can implement eventual consistency to maintain the availability of data.</span></span>
