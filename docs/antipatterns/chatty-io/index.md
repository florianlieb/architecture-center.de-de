---
title: "Antimuster „Zu viele E/A-Vorgänge“"
description: "Eine sehr hohe Anzahl von E/A-Anforderungen kann die Leistung und Reaktionsfähigkeit beeinträchtigen."
author: dragon119
ms.openlocfilehash: 50001316939b56c9b57a119f6ae20f0878f54c0f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="chatty-io-antipattern"></a><span data-ttu-id="95551-103">Antimuster „Zu viele E/A-Vorgänge“</span><span class="sxs-lookup"><span data-stu-id="95551-103">Chatty I/O antipattern</span></span>

<span data-ttu-id="95551-104">Der kumulative Effekt einer hohen Anzahl von E/A-Anforderungen kann sich massiv auf die Leistung und die Reaktionsfähigkeit auswirken.</span><span class="sxs-lookup"><span data-stu-id="95551-104">The cumulative effect of a large number of I/O requests can have a significant impact on performance and responsiveness.</span></span>

## <a name="problem-description"></a><span data-ttu-id="95551-105">Problembeschreibung</span><span class="sxs-lookup"><span data-stu-id="95551-105">Problem description</span></span>

<span data-ttu-id="95551-106">Netzwerkaufrufe und andere E/A-Vorgänge sind im Vergleich zu Computingtasks von Natur aus langsamer.</span><span class="sxs-lookup"><span data-stu-id="95551-106">Network calls and other I/O operations are inherently slow compared to compute tasks.</span></span> <span data-ttu-id="95551-107">Jede E/A-Anforderung führt in der Regel zu einem erheblichen Overhead, und der kumulative Effekt einer hohen Zahl von E/A-Vorgängen kann das System verlangsamen.</span><span class="sxs-lookup"><span data-stu-id="95551-107">Each I/O request typically has significant overhead, and the cumulative effect of numerous I/O operations can slow down the system.</span></span> <span data-ttu-id="95551-108">Im Folgenden finden Sie einige häufige Gründe für eine zu hohe Anzahl von E/A-Vorgängen.</span><span class="sxs-lookup"><span data-stu-id="95551-108">Here are some common causes of chatty I/O.</span></span>

### <a name="reading-and-writing-individual-records-to-a-database-as-distinct-requests"></a><span data-ttu-id="95551-109">Lesen und Schreiben einzelner Datensätze in einer Datenbank in unterschiedlichen Anforderungen</span><span class="sxs-lookup"><span data-stu-id="95551-109">Reading and writing individual records to a database as distinct requests</span></span>

<span data-ttu-id="95551-110">Das folgende Beispiel liest Daten aus einer Produktdatenbank.</span><span class="sxs-lookup"><span data-stu-id="95551-110">The following example reads from a database of products.</span></span> <span data-ttu-id="95551-111">Es gibt drei Tabellen: `Product`, `ProductSubcategory` und `ProductPriceListHistory`.</span><span class="sxs-lookup"><span data-stu-id="95551-111">There are three tables, `Product`, `ProductSubcategory`, and `ProductPriceListHistory`.</span></span> <span data-ttu-id="95551-112">Der Code ruft mithilfe einer Reihe von Abfragen alle Produkte in einer Unterkategorie zusammen mit den zugehörigen Preisinformationen ab:</span><span class="sxs-lookup"><span data-stu-id="95551-112">The code retrieves all of the products in a subcategory, along with the pricing information, by executing a series of queries:</span></span>  

1. <span data-ttu-id="95551-113">Die Unterkategorie wird aus der Tabelle `ProductSubcategory` abgefragt.</span><span class="sxs-lookup"><span data-stu-id="95551-113">Query the subcategory from the `ProductSubcategory` table.</span></span>
2. <span data-ttu-id="95551-114">Durch Abfragen der Tabelle `Product` werden alle Produkte in dieser Unterkategorie gesucht.</span><span class="sxs-lookup"><span data-stu-id="95551-114">Find all products in that subcategory by querying the `Product` table.</span></span>
3. <span data-ttu-id="95551-115">Für jedes Produkt werden die Preisdaten aus der Tabelle `ProductPriceListHistory` abgefragt.</span><span class="sxs-lookup"><span data-stu-id="95551-115">For each product, query the pricing data from the `ProductPriceListHistory` table.</span></span>

<span data-ttu-id="95551-116">Die Anwendung verwendet das [Entity Framework][ef] zum Abfragen der Datenbank.</span><span class="sxs-lookup"><span data-stu-id="95551-116">The application uses [Entity Framework][ef] to query the database.</span></span> <span data-ttu-id="95551-117">Das vollständige Codebeispiel finden Sie [hier][code-sample].</span><span class="sxs-lookup"><span data-stu-id="95551-117">You can find the complete sample [here][code-sample].</span></span> 

```csharp
public async Task<IHttpActionResult> GetProductsInSubCategoryAsync(int subcategoryId)
{
    using (var context = GetContext())
    {
        // Get product subcategory.
        var productSubcategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subcategoryId)
                .FirstOrDefaultAsync();

        // Find products in that category.
        productSubcategory.Product = await context.Products
            .Where(p => subcategoryId == p.ProductSubcategoryId)
            .ToListAsync();

        // Find price history for each product.
        foreach (var prod in productSubcategory.Product)
        {
            int productId = prod.ProductId;
            var productListPriceHistory = await context.ProductListPriceHistory
                .Where(pl => pl.ProductId == productId)
                .ToListAsync();
            prod.ProductListPriceHistory = productListPriceHistory;
        }
        return Ok(productSubcategory);
    }
}
```

<span data-ttu-id="95551-118">Dieses Beispiel veranschaulicht das Problem explizit, aber manchmal maskiert eine objektrelationale Abbildung (Object Relational Mapping, O/RM) das Problem, wenn untergeordnete Datensätze implizit nacheinander abgerufen werden.</span><span class="sxs-lookup"><span data-stu-id="95551-118">This example shows the problem explicitly, but sometimes an O/RM can mask the problem, if it implicitly fetches child records one at a time.</span></span> <span data-ttu-id="95551-119">Diese wird als „N+1-Problem“ bezeichnet.</span><span class="sxs-lookup"><span data-stu-id="95551-119">This is known as the "N+1 problem".</span></span> 

### <a name="implementing-a-single-logical-operation-as-a-series-of-http-requests"></a><span data-ttu-id="95551-120">Implementieren eines einzigen logischen Vorgangs als Serie von HTTP-Anforderungen</span><span class="sxs-lookup"><span data-stu-id="95551-120">Implementing a single logical operation as a series of HTTP requests</span></span>

<span data-ttu-id="95551-121">Dies passiert häufig, wenn Entwickler versuchen, einem objektorientierten Paradigma zu folgen und Remoteobjekte so behandeln, als wären es lokale Objekte im Arbeitsspeicher.</span><span class="sxs-lookup"><span data-stu-id="95551-121">This often happens when developers try to follow an object-oriented paradigm, and treat remote objects as if they were local objects in memory.</span></span> <span data-ttu-id="95551-122">Die kann zu einer zu hohen Anzahl von Netzwerkroundtrips führen.</span><span class="sxs-lookup"><span data-stu-id="95551-122">This can result in too many network round trips.</span></span> <span data-ttu-id="95551-123">Ein Beispiel: Die folgende Web-API macht die einzelnen Eigenschaften von `User`-Objekten über einzelne HTTP GET-Methoden verfügbar.</span><span class="sxs-lookup"><span data-stu-id="95551-123">For example, the following web API exposes the individual properties of `User` objects through individual HTTP GET methods.</span></span> 

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}/username")]
    public HttpResponseMessage GetUserName(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/gender")]
    public HttpResponseMessage GetGender(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/dateofbirth")]
    public HttpResponseMessage GetDateOfBirth(int id)
    {
        ...
    }
}
```

<span data-ttu-id="95551-124">Rein technisch ist dieser Ansatz völlig in Ordnung, allerdings werden die meisten Clients wahrscheinlich verschiedene Eigenschaften für jedes `User`-Objekt abrufen müssen, sodass der Clientcode in etwa folgendermaßen aussieht.</span><span class="sxs-lookup"><span data-stu-id="95551-124">While there's nothing technically wrong with this approach, most clients will probably need to get several properties for each `User`, resulting in client code like the following.</span></span> 

```csharp
HttpResponseMessage response = await client.GetAsync("users/1/username");
response.EnsureSuccessStatusCode();
var userName = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/gender");
response.EnsureSuccessStatusCode();
var gender = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/dateofbirth");
response.EnsureSuccessStatusCode();
var dob = await response.Content.ReadAsStringAsync();
```

### <a name="reading-and-writing-to-a-file-on-disk"></a><span data-ttu-id="95551-125">Lesen und Schreiben in einer Datei auf einem Datenträger</span><span class="sxs-lookup"><span data-stu-id="95551-125">Reading and writing to a file on disk</span></span>

<span data-ttu-id="95551-126">Datei-E/A-Vorgänge umfassen das Öffnen einer Datei und das Springen an den geeigneten Punkt, bevor Daten gelesen oder geschrieben werden.</span><span class="sxs-lookup"><span data-stu-id="95551-126">File I/O involves opening a file and moving to the appropriate point before reading or writing data.</span></span> <span data-ttu-id="95551-127">Wenn der Vorgang abgeschlossen wurde, kann die Datei geschlossen werden, um Systemressourcen einzusparen.</span><span class="sxs-lookup"><span data-stu-id="95551-127">When the operation is complete, the file might be closed to save operating system resources.</span></span> <span data-ttu-id="95551-128">Eine Anwendung, die kontinuierlich kleine Mengen von Informationen in einer Datei liest oder schreibt, generiert einen erheblichen E/A-Overhead.</span><span class="sxs-lookup"><span data-stu-id="95551-128">An application that continually reads and writes small amounts of information to a file will generate significant I/O overhead.</span></span> <span data-ttu-id="95551-129">Schreibanforderungen für geringe Datenmengen können auch zur Fragmentierung von Dateien führen und damit nachfolgende E/A-Vorgänge noch weiter verlangsamen.</span><span class="sxs-lookup"><span data-stu-id="95551-129">Small write requests can also lead to file fragmentation, slowing subsequent I/O operations still further.</span></span> 

<span data-ttu-id="95551-130">Das folgende Beispiel verwendet ein `FileStream`-Objekt, um ein `Customer`-Objekt in eine Datei zu schreiben.</span><span class="sxs-lookup"><span data-stu-id="95551-130">The following example uses a `FileStream` to write a `Customer` object to a file.</span></span> <span data-ttu-id="95551-131">Durch Erstellen des `FileStream`-Objekts wird die Datei geöffnet, durch Löschen des Objekts wird sie wieder geschlossen.</span><span class="sxs-lookup"><span data-stu-id="95551-131">Creating the `FileStream` opens the file, and disposing it closes the file.</span></span> <span data-ttu-id="95551-132">(Die `using`-Anweisung löscht das `FileStream`-Objekt automatisch.) Wenn die Anwendung diese Methode wiederholt aufruft, wenn neue Kunden hinzugefügt werden, kann der E/A-Overhead schnell anwachsen.</span><span class="sxs-lookup"><span data-stu-id="95551-132">(The `using` statement automatically disposes the `FileStream` object.) If the application calls this method repeatedly as new customers are added, the I/O overhead can accumulate quickly.</span></span>

```csharp
private async Task SaveCustomerToFileAsync(Customer cust)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        byte [] data = null;
        using (MemoryStream memStream = new MemoryStream())
        {
            formatter.Serialize(memStream, cust);
            data = memStream.ToArray();
        }
        await fileStream.WriteAsync(data, 0, data.Length);
    }
}
```

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="95551-133">Beheben des Problems</span><span class="sxs-lookup"><span data-stu-id="95551-133">How to fix the problem</span></span>

<span data-ttu-id="95551-134">Reduzieren Sie die Anzahl von E/A-Anforderungen, indem Sie die Daten in größere, weniger häufig auftretende Anforderungen packen.</span><span class="sxs-lookup"><span data-stu-id="95551-134">Reduce the number of I/O requests by packaging the data into larger, fewer requests.</span></span>

<span data-ttu-id="95551-135">Rufen Sie Daten in einer einzigen Abfrage aus einer Datenbank ab, nicht in mehreren kleinen Abfragen.</span><span class="sxs-lookup"><span data-stu-id="95551-135">Fetch data from a database as a single query, instead of several smaller queries.</span></span> <span data-ttu-id="95551-136">Im Folgenden sehen Sie eine überarbeitete Version des Codes, der Produktinformationen abruft.</span><span class="sxs-lookup"><span data-stu-id="95551-136">Here's a revised version of the code that retrieves product information.</span></span>

```csharp
public async Task<IHttpActionResult> GetProductCategoryDetailsAsync(int subCategoryId)
{
    using (var context = GetContext())
    {
        var subCategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subCategoryId)
                .Include("Product.ProductListPriceHistory")
                .FirstOrDefaultAsync();

        if (subCategory == null)
            return NotFound();

        return Ok(subCategory);
    }
}
```

<span data-ttu-id="95551-137">Befolgen Sie die REST-Entwurfsprinzipien für Web-APIs.</span><span class="sxs-lookup"><span data-stu-id="95551-137">Follow REST design principles for web APIs.</span></span> <span data-ttu-id="95551-138">Nachfolgend sehen Sie eine überarbeitete Version der Web-API aus dem vorherigen Beispiel.</span><span class="sxs-lookup"><span data-stu-id="95551-138">Here's a revised version of the web API from the earlier example.</span></span> <span data-ttu-id="95551-139">Statt separater GET-Methoden für jede Eigenschaft wird eine einzige GET-Methode verwendet, die das `User`-Objekt zurückgibt.</span><span class="sxs-lookup"><span data-stu-id="95551-139">Instead of separate GET methods for each property, there is a single GET method that returns the `User`.</span></span> <span data-ttu-id="95551-140">Dadurch wird für jede Anforderung ein längerer Antworttext zurückgegeben, aber die einzelnen Clients werden wahrscheinlich weniger API-Aufrufe durchführen.</span><span class="sxs-lookup"><span data-stu-id="95551-140">This results in a larger response body per request, but each client is likely to make fewer API calls.</span></span>

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}")]
    public HttpResponseMessage GetUser(int id)
    {
        ...
    }
}

// Client code
HttpResponseMessage response = await client.GetAsync("users/1");
response.EnsureSuccessStatusCode();
var user = await response.Content.ReadAsStringAsync();
```

<span data-ttu-id="95551-141">Erwägen Sie bei Datei-E/A-Vorgängen, Daten im Arbeitsspeicher zu puffern und die gepufferten Daten dann in einem einzigen Vorgang in eine Datei zu schreiben.</span><span class="sxs-lookup"><span data-stu-id="95551-141">For file I/O, consider buffering data in memory and then writing the buffered data to a file as a single operation.</span></span> <span data-ttu-id="95551-142">Mit dieser Vorgehensweise reduzieren Sie den Overhead, der durch häufiges Öffnen und Schließen der Datei entsteht, und verringern zudem die Fragmentierung der Datei auf dem Datenträger.</span><span class="sxs-lookup"><span data-stu-id="95551-142">This approach reduces the overhead from repeatedly opening and closing the file, and helps to reduce fragmentation of the file on disk.</span></span>

```csharp
// Save a list of customer objects to a file
private async Task SaveCustomerListToFileAsync(List<Customer> customers)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        foreach (var cust in customers)
        {
            byte[] data = null;
            using (MemoryStream memStream = new MemoryStream())
            {
                formatter.Serialize(memStream, cust);
                data = memStream.ToArray();
            }
            await fileStream.WriteAsync(data, 0, data.Length);
        }
    }
}

// In-memory buffer for customers.
List<Customer> customers = new List<Customers>();

// Create a new customer and add it to the buffer
var cust = new Customer(...);
customers.Add(cust);

// Add more customers to the list as they are created
...

// Save the contents of the list, writing all customers in a single operation
await SaveCustomerListToFileAsync(customers);
```

## <a name="considerations"></a><span data-ttu-id="95551-143">Überlegungen</span><span class="sxs-lookup"><span data-stu-id="95551-143">Considerations</span></span>

- <span data-ttu-id="95551-144">Die ersten beiden Beispiele führen *weniger* E/A-Aufrufe durch, jedes empfängt dadurch aber *mehr* Informationen.</span><span class="sxs-lookup"><span data-stu-id="95551-144">The first two examples make *fewer* I/O calls, but each one retrieves *more* information.</span></span> <span data-ttu-id="95551-145">Diese beiden Faktoren müssen gegeneinander abgewogen werden.</span><span class="sxs-lookup"><span data-stu-id="95551-145">You must consider the tradeoff between these two factors.</span></span> <span data-ttu-id="95551-146">Die richtige Antwort hängt von den tatsächlichen Nutzungsmustern ab.</span><span class="sxs-lookup"><span data-stu-id="95551-146">The right answer will depend on the actual usage patterns.</span></span> <span data-ttu-id="95551-147">Im Web-API-Beispiel könnte sich etwa herausstellen, dass Clients häufig nur den Benutzernamen benötigen.</span><span class="sxs-lookup"><span data-stu-id="95551-147">For example, in the web API example, it might turn out that clients often need just the user name.</span></span> <span data-ttu-id="95551-148">In diesem Fall ist es sinnvoll, diesen mit einem separaten API-Aufruf verfügbar zu machen.</span><span class="sxs-lookup"><span data-stu-id="95551-148">In that case, it might make sense to expose it as a separate API call.</span></span> <span data-ttu-id="95551-149">Weitere Informationen finden Sie im Antimuster [Irrelevante Abrufe][extraneous-fetching].</span><span class="sxs-lookup"><span data-stu-id="95551-149">For more information, see the [Extraneous Fetching][extraneous-fetching] antipattern.</span></span>

- <span data-ttu-id="95551-150">Entwerfen Sie beim Lesen von Daten keine zu umfangreichen E/A-Anforderungen.</span><span class="sxs-lookup"><span data-stu-id="95551-150">When reading data, do not make your I/O requests too large.</span></span> <span data-ttu-id="95551-151">Eine Anwendung sollte nur die Daten abrufen, die sie wahrscheinlich benötigt.</span><span class="sxs-lookup"><span data-stu-id="95551-151">An application should only retrieve the information that it is likely to use.</span></span> 

- <span data-ttu-id="95551-152">Manchmal hilft es, die Informationen für ein Objekt in zwei Blöcke zu unterteilen: *Daten, auf die häufig zugegriffen wird* und die für die meisten Anforderungen relevant sind, und *Daten, auf die weniger häufig zugegriffen wird* und die entsprechend seltener benötigt werden.</span><span class="sxs-lookup"><span data-stu-id="95551-152">Sometimes it helps to partition the information for an object into two chunks, *frequently accessed data* that accounts for most requests, and *less frequently accessed data* that is used rarely.</span></span> <span data-ttu-id="95551-153">Häufig machen die Daten, auf die am häufigsten zugegriffen wird, einen relativ geringen Teil der Gesamtdatenmenge für ein Objekt aus, sodass sich der E/A-Overhead erheblich senken lässt, indem nur dieser Teil zurückgegeben wird.</span><span class="sxs-lookup"><span data-stu-id="95551-153">Often the most frequently accessed data is a relatively small portion of the total data for an object, so returning just that portion can save significant I/O overhead.</span></span>

- <span data-ttu-id="95551-154">Vermeiden Sie es beim Schreiben von Daten, Ressourcen länger als notwendig zu sperren, um das Risiko von Konflikten während eines Vorgangs mit langer Ausführungsdauer zu mindern.</span><span class="sxs-lookup"><span data-stu-id="95551-154">When writing data, avoid locking resources for longer than necessary, to reduce the chances of contention during a lengthy operation.</span></span> <span data-ttu-id="95551-155">Wenn ein Schreibvorgang mehrere Datenspeicher, Dateien oder Dienste umfasst, wenden Sie einen letztlich konsistenten Ansatz an.</span><span class="sxs-lookup"><span data-stu-id="95551-155">If a write operation spans multiple data stores, files, or services, then adopt an eventually consistent approach.</span></span> <span data-ttu-id="95551-156">Weitere Informationen finden Sie unter [Data Consistency guidance][data-consistency-guidance] (Leitfaden zur Datenkonsistenz).</span><span class="sxs-lookup"><span data-stu-id="95551-156">See [Data Consistency guidance][data-consistency-guidance].</span></span>

- <span data-ttu-id="95551-157">Wenn Sie Daten vor dem Schreiben im Arbeitsspeicher puffern, sind die Daten gefährdet, wenn der Prozess abstürzt.</span><span class="sxs-lookup"><span data-stu-id="95551-157">If you buffer data in memory before writing it, the data is vulnerable if the process crashes.</span></span> <span data-ttu-id="95551-158">Wenn die Datenrate in der Regel verhältnismäßig gering ist oder Lastspitzen aufweist, ist es möglicherweise sicherer, die Daten in einer externen dauerhaften Warteschlange wie z.B. [Event Hubs](http://azure.microsoft.com/en-us/services/event-hubs/) zu puffern.</span><span class="sxs-lookup"><span data-stu-id="95551-158">If the data rate typically has bursts or is relatively sparse, it may be safer to buffer the data in an external durable queue such as [Event Hubs](http://azure.microsoft.com/en-us/services/event-hubs/).</span></span>

- <span data-ttu-id="95551-159">Erwägen Sie, Daten, die Sie aus einem Dienst oder einer Datenbank abrufen, zwischenzuspeichern.</span><span class="sxs-lookup"><span data-stu-id="95551-159">Consider caching data that you retrieve from a service or a database.</span></span> <span data-ttu-id="95551-160">So lässt sich die Menge an E/A-Vorgängen verringern, da wiederholte Anforderungen für die gleichen Daten vermieden werden.</span><span class="sxs-lookup"><span data-stu-id="95551-160">This can help to reduce the volume of I/O by avoiding repeated requests for the same data.</span></span> <span data-ttu-id="95551-161">Weitere Informationen finden Sie in den [bewährten Methoden für das Caching][caching-guidance].</span><span class="sxs-lookup"><span data-stu-id="95551-161">For more information, see [Caching best practices][caching-guidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="95551-162">Erkennen des Problems</span><span class="sxs-lookup"><span data-stu-id="95551-162">How to detect the problem</span></span>

<span data-ttu-id="95551-163">Zu den Symptomen einer zu hohen Anzahl von E/A-Vorgängen gehören eine hohe Latenz und ein geringer Durchsatz.</span><span class="sxs-lookup"><span data-stu-id="95551-163">Symptoms of chatty I/O include high latency and low throughput.</span></span> <span data-ttu-id="95551-164">Endbenutzer berichten vermutlich von längeren Antwortzeiten oder Fehlern aufgrund von Diensten, bei denen ein Timeout auftritt. Diese Probleme entstehen durch einen immer größeren Konflikt bei E/A-Ressourcen.</span><span class="sxs-lookup"><span data-stu-id="95551-164">End users are likely to report extended response times or failures caused by services timing out, due to increased contention for I/O resources.</span></span>

<span data-ttu-id="95551-165">Sie können die folgenden Schritte ausführen, um die Ursache solcher Probleme zu identifizieren:</span><span class="sxs-lookup"><span data-stu-id="95551-165">You can perform the following steps to help identify the causes of any problems:</span></span>

1. <span data-ttu-id="95551-166">Führen Sie eine Prozessüberwachung des Produktionssystems durch, um Vorgänge mit unzureichenden Antwortzeiten zu identifizieren.</span><span class="sxs-lookup"><span data-stu-id="95551-166">Perform process monitoring of the production system to identify operations with poor response times.</span></span>
2. <span data-ttu-id="95551-167">Führen Sie für jeden im vorherigen Schritt identifizierten Vorgang einen Auslastungstest durch.</span><span class="sxs-lookup"><span data-stu-id="95551-167">Perform load testing of each operation identified in the previous step.</span></span>
3. <span data-ttu-id="95551-168">Sammeln Sie während der Auslastungstests Telemetriedaten über die Datenzugriffsanforderungen jedes Vorgangs.</span><span class="sxs-lookup"><span data-stu-id="95551-168">During the load tests, gather telemetry data about the data access requests made by each operation.</span></span>
4. <span data-ttu-id="95551-169">Erfassen Sie detaillierte Statistiken für jede an einen Datenspeicher gesendete Anforderung.</span><span class="sxs-lookup"><span data-stu-id="95551-169">Gather detailed statistics for each request sent to a data store.</span></span>
5. <span data-ttu-id="95551-170">Erstellen Sie ein Profil für die Anwendung in der Testumgebung, um herauszufinden, wo möglich E/A-Engpässe auftreten können.</span><span class="sxs-lookup"><span data-stu-id="95551-170">Profile the application in the test environment to establish where possible I/O bottlenecks might be occurring.</span></span> 

<span data-ttu-id="95551-171">Suchen Sie nach folgenden Symptomen:</span><span class="sxs-lookup"><span data-stu-id="95551-171">Look for any of these symptoms:</span></span>

- <span data-ttu-id="95551-172">Eine große Anzahl von kleinen E/A-Anforderungen für die gleiche Datei</span><span class="sxs-lookup"><span data-stu-id="95551-172">A large number of small I/O requests made to the same file.</span></span>
- <span data-ttu-id="95551-173">Eine große Anzahl von kleinen Netzwerkanforderungen, die von einer Anwendungsinstanz an den gleichen Dienst gesendet werden</span><span class="sxs-lookup"><span data-stu-id="95551-173">A large number of small network requests made by an application instance to the same service.</span></span>
- <span data-ttu-id="95551-174">Eine große Anzahl von kleinen Anforderungen, die von einer Anwendungsinstanz an den gleichen Datenspeicher gesendet werden</span><span class="sxs-lookup"><span data-stu-id="95551-174">A large number of small requests made by an application instance to the same data store.</span></span>
- <span data-ttu-id="95551-175">Anwendungen und Dienste, die zunehmend E/A-gebunden sind</span><span class="sxs-lookup"><span data-stu-id="95551-175">Applications and services becoming I/O bound.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="95551-176">Beispieldiagnose</span><span class="sxs-lookup"><span data-stu-id="95551-176">Example diagnosis</span></span>

<span data-ttu-id="95551-177">In den folgenden Abschnitten werden diese Schritte auf das oben gezeigte Beispiel angewendet, das eine Datenbank abfragt.</span><span class="sxs-lookup"><span data-stu-id="95551-177">The following sections apply these steps to the example shown earlier that queries a database.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="95551-178">Auslastungstest der Anwendung</span><span class="sxs-lookup"><span data-stu-id="95551-178">Load test the application</span></span>

<span data-ttu-id="95551-179">Dieses Diagramm zeigt die Ergebnisse der Auslastungstests.</span><span class="sxs-lookup"><span data-stu-id="95551-179">This graph shows the results of load testing.</span></span> <span data-ttu-id="95551-180">Der Medianwert der Antwortzeit wird in Zehntelsekunden pro Anforderung gemessen.</span><span class="sxs-lookup"><span data-stu-id="95551-180">Median response time is measured in 10s of seconds per request.</span></span> <span data-ttu-id="95551-181">Das Diagramm zeigt eine sehr hohe Latenz.</span><span class="sxs-lookup"><span data-stu-id="95551-181">The graph shows very high latency.</span></span> <span data-ttu-id="95551-182">Bei einer Last von 1.000 Benutzern muss ein Benutzer möglicherweise fast eine Minute auf die Ergebnisse einer Abfrage warten.</span><span class="sxs-lookup"><span data-stu-id="95551-182">With a load of 1000 users, a user might have to wait for nearly a minute to see the results of a query.</span></span> 

![Ergebnisse der Auslastungstests für wichtige Leistungsindikatoren der Beispielanwendung für eine zu hohe Anzahl von E/A-Vorgängen][key-indicators-chatty-io]

> [!NOTE]
> <span data-ttu-id="95551-184">Die Anwendung wurde als Azure App Service-Web-App bereitgestellt und verwendet Azure SQL-Datenbank.</span><span class="sxs-lookup"><span data-stu-id="95551-184">The application was deployed as an Azure App Service web app, using Azure SQL Database.</span></span> <span data-ttu-id="95551-185">Der Auslastungstest wurde mit einer simulierten schrittbasierten Workload von bis zu 1.000 gleichzeitigen Benutzern durchgeführt.</span><span class="sxs-lookup"><span data-stu-id="95551-185">The load test used a simulated step workload of up to 1000 concurrent users.</span></span> <span data-ttu-id="95551-186">Die Datenbank war mit einem Verbindungspool konfiguriert, der bis zu 1.000 gleichzeitige Verbindungen unterstützte, um das Risiko zu reduzieren, dass Verbindungskonflikte sich auf die Ergebnisse auswirken.</span><span class="sxs-lookup"><span data-stu-id="95551-186">The database was configured with a connection pool supporting up to 1000 concurrent connections, to reduce the chance that contention for connections would affect the results.</span></span> 

### <a name="monitor-the-application"></a><span data-ttu-id="95551-187">Überwachen der Anwendung</span><span class="sxs-lookup"><span data-stu-id="95551-187">Monitor the application</span></span>

<span data-ttu-id="95551-188">Sie können ein Paket für die Überwachung der Anwendungsleistung (Application Performance Monitoring, APM) verwenden, um die wichtigsten Metriken zu erfassen und zu analysieren, die eine zu hohe Anzahl von E/A-Vorgängen identifizieren können.</span><span class="sxs-lookup"><span data-stu-id="95551-188">You can use an application performance monitoring (APM) package to capture and analyze the key metrics that might identify chatty I/O.</span></span> <span data-ttu-id="95551-189">Welche Metriken wichtig sind, richtet sich nach der E/A-Workload.</span><span class="sxs-lookup"><span data-stu-id="95551-189">Which metrics are important will depend on the I/O workload.</span></span> <span data-ttu-id="95551-190">In diesem Beispiel waren die Datenbankabfragen die interessantesten E/A-Anforderungen.</span><span class="sxs-lookup"><span data-stu-id="95551-190">For this example, the interesting I/O requests were the database queries.</span></span> 

<span data-ttu-id="95551-191">Die folgende Abbildung zeigt Ergebnisse, die mithilfe von [New Relic APM][new-relic] generiert wurden.</span><span class="sxs-lookup"><span data-stu-id="95551-191">The following image shows results generated using [New Relic APM][new-relic].</span></span> <span data-ttu-id="95551-192">Der Spitzenwert der durchschnittlichen Datenbankantwortzeit lag während der maximalen Workload bei ca. 5,6 Sekunden pro Anforderung.</span><span class="sxs-lookup"><span data-stu-id="95551-192">The average database response time peaked at approximately 5.6 seconds per request during the maximum workload.</span></span> <span data-ttu-id="95551-193">Das System konnte während des gesamten Tests durchschnittlich 410 Anforderungen pro Minute unterstützen.</span><span class="sxs-lookup"><span data-stu-id="95551-193">The system was able to support an average of 410 requests per minute throughout the test.</span></span>

![Übersicht über den Datenverkehr in der AdventureWorks2012-Datenbank][databasetraffic]

### <a name="gather-detailed-data-access-information"></a><span data-ttu-id="95551-195">Erfassen detaillierter Informationen über den Datenzugriff</span><span class="sxs-lookup"><span data-stu-id="95551-195">Gather detailed data access information</span></span>

<span data-ttu-id="95551-196">Ein genauerer Blick auf die Überwachungsdaten zeigt, dass die Anwendung drei verschiedene SQL SELECT-Anweisungen ausgeführt hat.</span><span class="sxs-lookup"><span data-stu-id="95551-196">Digging deeper into the monitoring data shows the application executes three different SQL SELECT statements.</span></span> <span data-ttu-id="95551-197">Diese entsprechen den Anforderungen, die von Entity Framework generiert wurden, um Daten aus den Tabellen `ProductListPriceHistory`, `Product` und `ProductSubcategory` abzurufen.</span><span class="sxs-lookup"><span data-stu-id="95551-197">These correspond to the requests generated by Entity Framework to fetch data from the `ProductListPriceHistory`, `Product`, and `ProductSubcategory` tables.</span></span>
<span data-ttu-id="95551-198">Darüber hinaus ist die Abfrage, die Daten aus der Tabelle `ProductListPriceHistory` abruft, die bei weitem am häufigsten ausgeführte SELECT-Anweisung.</span><span class="sxs-lookup"><span data-stu-id="95551-198">Furthermore, the query that retrieves data from the `ProductListPriceHistory` table is by far the most frequently executed SELECT statement, by an order of magnitude.</span></span>

![Von der Beispielanwendung im Test ausgeführte Abfragen][queries]

<span data-ttu-id="95551-200">Es zeigt sich, dass die oben gezeigte `GetProductsInSubCategoryAsync`-Methode 45 SELECT-Abfragen durchführt.</span><span class="sxs-lookup"><span data-stu-id="95551-200">It turns out that the `GetProductsInSubCategoryAsync` method, shown earlier, performs 45 SELECT queries.</span></span> <span data-ttu-id="95551-201">Jede Abfrage führt dazu, dass die Anwendung eine neue SQL-Verbindung öffnet.</span><span class="sxs-lookup"><span data-stu-id="95551-201">Each query causes the application to open a new SQL connection.</span></span>

![Abfragestatistiken für die Beispielanwendung im Test][queries2]

> [!NOTE]
> <span data-ttu-id="95551-203">Diese Abbildung zeigt Ablaufverfolgungsinformationen für die langsamste Instanz des `GetProductsInSubCategoryAsync`-Vorgangs im Auslastungstest.</span><span class="sxs-lookup"><span data-stu-id="95551-203">This image shows trace information for the slowest instance of the `GetProductsInSubCategoryAsync` operation in the load test.</span></span> <span data-ttu-id="95551-204">In einer Produktionsumgebung ist es sehr hilfreich, die Ablaufverfolgungen der langsamsten Instanzen zu untersuchen, um festzustellen, ob ein Muster vorliegt, das auf ein Problem hinweist.</span><span class="sxs-lookup"><span data-stu-id="95551-204">In a production environment, it's useful to examine traces of the slowest instances, to see if there is a pattern that suggests a problem.</span></span> <span data-ttu-id="95551-205">Wenn Sie nur die Durchschnittswerte betrachten, entgehen Ihnen möglicherweise Probleme, die unter hoher Last erheblich schlimmer werden.</span><span class="sxs-lookup"><span data-stu-id="95551-205">If you just look at the average values, you might overlook problems that will get dramatically worse under load.</span></span>

<span data-ttu-id="95551-206">Die nächste Abbildung zeigt die tatsächlich ausgegebenen SQL-Anweisungen.</span><span class="sxs-lookup"><span data-stu-id="95551-206">The next image shows the actual SQL statements that were issued.</span></span> <span data-ttu-id="95551-207">Die Abfrage, die Preisinformationen abruft, wird für jedes einzelne Produkt in der Unterkategorie der Produkte ausgeführt.</span><span class="sxs-lookup"><span data-stu-id="95551-207">The query that fetches price information is run for each individual product in the product subcategory.</span></span> <span data-ttu-id="95551-208">Mithilfe eines Joins ließe sich die Anzahl von Datenbankaufrufen deutlich reduzieren.</span><span class="sxs-lookup"><span data-stu-id="95551-208">Using a join would considerably reduce the number of database calls.</span></span>

![Abfragedetails für die Beispielanwendung im Test][queries3]

<span data-ttu-id="95551-210">Wenn Sie eine objektrelationale Abbildung wie z.B. Entity Framework verwenden, kann eine Ablaufverfolgung der SQL-Abfragen Einblicke dazu liefern, wie die objektrelationale Abbildung die programmgesteuerten Aufrufe in SQL-Anweisungen übersetzt. Gleichzeitig können Bereiche aufgezeigt werden, in denen sich der Datenzugriff optimieren lässt.</span><span class="sxs-lookup"><span data-stu-id="95551-210">If you are using an O/RM, such as Entity Framework, tracing the SQL queries can provide insight into how the O/RM translates programmatic calls into SQL statements, and indicate areas where data access might be optimized.</span></span> 

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="95551-211">Implementieren der Lösung und Überprüfen des Ergebnisses</span><span class="sxs-lookup"><span data-stu-id="95551-211">Implement the solution and verify the result</span></span>

<span data-ttu-id="95551-212">Ein Umschreiben des Aufrufs von Entity Framework generiert die folgenden Ergebnisse.</span><span class="sxs-lookup"><span data-stu-id="95551-212">Rewriting the call to Entity Framework produced the following results.</span></span>

![Ergebnisse der Auslastungstests für wichtige Leistungsindikatoren für die Chunky-API in der Beispielanwendung für eine zu hohe Anzahl von E/A-Vorgängen][key-indicators-chunky-io]

<span data-ttu-id="95551-214">Dieser Auslastungstest wurde mit dem gleichen Auslastungsprofil in der gleichen Bereitstellung ausgeführt.</span><span class="sxs-lookup"><span data-stu-id="95551-214">This load test was performed on the same deployment, using the same load profile.</span></span> <span data-ttu-id="95551-215">Dieses Mal zeigt das Diagramm eine viel niedrigere Latenz.</span><span class="sxs-lookup"><span data-stu-id="95551-215">This time the graph shows much lower latency.</span></span> <span data-ttu-id="95551-216">Die durchschnittliche Anforderungszeit bei 1.000 Benutzern liegt zwischen 5 und 6 Sekunden – zuvor lag sie bei fast einer Minute.</span><span class="sxs-lookup"><span data-stu-id="95551-216">The average request time at 1000 users is between 5 and 6 seconds, down from nearly a minute.</span></span>

<span data-ttu-id="95551-217">Dieses Mal unterstützte das System durchschnittlich 3.970 Anforderungen pro Minute, im Vergleich zu 410 im vorherigen Test.</span><span class="sxs-lookup"><span data-stu-id="95551-217">This time the system supported an average of 3,970 requests per minute, compared to 410 for the earlier test.</span></span>

![Transaktionsübersicht für die Chunky-API][databasetraffic2]

<span data-ttu-id="95551-219">Die Ablaufverfolgung der SQL-Anweisung zeigt, dass alle Daten in einer einzigen SELECT-Anweisung abgerufen werden.</span><span class="sxs-lookup"><span data-stu-id="95551-219">Tracing the SQL statement shows that all the data is fetched in a single SELECT statement.</span></span> <span data-ttu-id="95551-220">Diese Abfrage ist zwar deutlich komplexer, wird dafür aber nur einmal pro Vorgang ausgeführt.</span><span class="sxs-lookup"><span data-stu-id="95551-220">Although this query is considerably more complex, it is performed only once per operation.</span></span> <span data-ttu-id="95551-221">Und während komplexe Joins teuer werden können, sind relationale Datenbanksysteme für diese Art von Abfragen optimiert.</span><span class="sxs-lookup"><span data-stu-id="95551-221">And while complex joins can become expensive, relational database systems are optimized for this type of query.</span></span>  

![Abfragedetails für die Chunky-API][queries4]

## <a name="related-resources"></a><span data-ttu-id="95551-223">Zugehörige Ressourcen</span><span class="sxs-lookup"><span data-stu-id="95551-223">Related resources</span></span>

- <span data-ttu-id="95551-224">[API Design best practices][api-design] (Bewährte Methoden für den API-Entwurf)</span><span class="sxs-lookup"><span data-stu-id="95551-224">[API Design best practices][api-design]</span></span>
- <span data-ttu-id="95551-225">[Caching best practices][caching-guidance] (Bewährte Methoden für das Caching)</span><span class="sxs-lookup"><span data-stu-id="95551-225">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="95551-226">[Data Consistency Primer][data-consistency-guidance] (Grundlagen der Datenkonsistenz)</span><span class="sxs-lookup"><span data-stu-id="95551-226">[Data Consistency Primer][data-consistency-guidance]</span></span>
- <span data-ttu-id="95551-227">[Antimuster „Irrelevante Abrufe“][extraneous-fetching]</span><span class="sxs-lookup"><span data-stu-id="95551-227">[Extraneous Fetching antipattern][extraneous-fetching]</span></span>
- <span data-ttu-id="95551-228">[Antimuster „Kein Caching“][no-cache]</span><span class="sxs-lookup"><span data-stu-id="95551-228">[No Caching antipattern][no-cache]</span></span>

[api-design]: ../../best-practices/api-design.md
[caching-guidance]: ../../best-practices/caching.md
[code-sample]:  https://github.com/mspnp/performance-optimization/tree/master/ChattyIO
[data-consistency-guidance]: http://https://msdn.microsoft.com/library/dn589800.aspx
[ef]: /ef/
[extraneous-fetching]: ../extraneous-fetching/index.md
[new-relic]: https://newrelic.com/application-monitoring
[no-cache]: ../no-caching/index.md

[key-indicators-chatty-io]: _images/ChattyIO.jpg
[key-indicators-chunky-io]: _images/ChunkyIO.jpg
[databasetraffic]: _images/DatabaseTraffic.jpg
[databasetraffic2]: _images/DatabaseTraffic2.jpg
[queries]: _images/DatabaseQueries.jpg
[queries2]: _images/DatabaseQueries2.jpg
[queries3]: _images/DatabaseQueries3.jpg
[queries4]: _images/DatabaseQueries4.jpg

