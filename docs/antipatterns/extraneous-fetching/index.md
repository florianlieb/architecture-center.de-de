---
title: Antimuster „Extraneous Fetching“ (Irrelevante Abrufe)
description: Das Abrufen von mehr Daten, als für einen Geschäftsvorgang erforderlich sind, kann zu unnötigem E/A-Mehraufwand und einer Reduzierung der Reaktionsfähigkeit führen.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 7a72bfd3e4b2e206f3266a046fac2083224ecb4f
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="extraneous-fetching-antipattern"></a><span data-ttu-id="4b103-103">Antimuster „Extraneous Fetching“ (Irrelevante Abrufe)</span><span class="sxs-lookup"><span data-stu-id="4b103-103">Extraneous Fetching antipattern</span></span>

<span data-ttu-id="4b103-104">Das Abrufen von mehr Daten, als für einen Geschäftsvorgang erforderlich sind, kann zu unnötigem E/A-Mehraufwand und einer Reduzierung der Reaktionsfähigkeit führen.</span><span class="sxs-lookup"><span data-stu-id="4b103-104">Retrieving more data than needed for a business operation can result in unnecessary I/O overhead and reduce responsiveness.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="4b103-105">Problembeschreibung</span><span class="sxs-lookup"><span data-stu-id="4b103-105">Problem description</span></span>

<span data-ttu-id="4b103-106">Dieses Antimuster kann auftreten, wenn die Anwendung versucht, die Anzahl von E/A-Anforderungen gering zu halten, indem alle *ggf.* benötigten Daten abgerufen werden.</span><span class="sxs-lookup"><span data-stu-id="4b103-106">This antipattern can occur if the application tries to minimize I/O requests by retrieving all of the data that it *might* need.</span></span> <span data-ttu-id="4b103-107">Dies ist häufig das Ergebnis einer Überkompensation für das Antimuster [Chatty I/O][chatty-io] (Sehr hohe Zahl von E/A-Vorgängen).</span><span class="sxs-lookup"><span data-stu-id="4b103-107">This is often a result of overcompensating for the [Chatty I/O][chatty-io] antipattern.</span></span> <span data-ttu-id="4b103-108">Es kann beispielsweise sein, dass eine Anwendung die Details für jedes Produkt einer Datenbank abruft.</span><span class="sxs-lookup"><span data-stu-id="4b103-108">For example, an application might fetch the details for every product in a database.</span></span> <span data-ttu-id="4b103-109">Der Benutzer benötigt aber ggf. nur einen Teil dieser Details (da einige für Kunden nicht relevant sind) und muss vermutlich nicht *alle* Produkte auf einmal sehen können.</span><span class="sxs-lookup"><span data-stu-id="4b103-109">But the user may need just a subset of the details (some may not be relevant to customers), and probably doesn't need to see *all* of the products at once.</span></span> <span data-ttu-id="4b103-110">Auch wenn sich der Benutzer den gesamten Katalog ansieht, ist es sinnvoll, die Ergebnisse nach Seiten aufzuteilen und beispielsweise jeweils nur 20 anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="4b103-110">Even if the user is browsing the entire catalog, it would make sense to paginate the results &mdash; showing 20 at a time, for example.</span></span>

<span data-ttu-id="4b103-111">Ursachen dieses Problems können eine schlechte Programmierung oder ein unzureichender Entwurf sein.</span><span class="sxs-lookup"><span data-stu-id="4b103-111">Another source of this problem is following poor programming or design practices.</span></span> <span data-ttu-id="4b103-112">Im folgenden Code wird beispielsweise Entity Framework verwendet, um die gesamten Details zu jedem Produkt abzurufen.</span><span class="sxs-lookup"><span data-stu-id="4b103-112">For example, the following code uses Entity Framework to fetch the complete details for every product.</span></span> <span data-ttu-id="4b103-113">Anschließend werden die Ergebnisse gefiltert, um nur eine Teilmenge der Felder zurückzugeben, und die restlichen Daten werden verworfen.</span><span class="sxs-lookup"><span data-stu-id="4b103-113">Then it filters the results to return only a subset of the fields, discarding the rest.</span></span> <span data-ttu-id="4b103-114">Das vollständige Beispiel finden Sie [hier][sample-app].</span><span class="sxs-lookup"><span data-stu-id="4b103-114">You can find the complete sample [here][sample-app].</span></span>

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

<span data-ttu-id="4b103-115">Im nächsten Beispiel ruft die Anwendung Daten zum Durchführen eines Aggregationsvorgangs ab, der stattdessen von der Datenbank übernommen werden könnte.</span><span class="sxs-lookup"><span data-stu-id="4b103-115">In the next example, the application retrieves data to perform an aggregation that could be done by the database instead.</span></span> <span data-ttu-id="4b103-116">Die Anwendung berechnet den Gesamtumsatz, indem alle Datensätze für alle bearbeiteten Bestellungen abgerufen werden und anschließend die Summe für diese Datensätze gebildet wird.</span><span class="sxs-lookup"><span data-stu-id="4b103-116">The application calculates total sales by getting every record for all orders sold, and then computing the sum over those records.</span></span> <span data-ttu-id="4b103-117">Das vollständige Beispiel finden Sie [hier][sample-app].</span><span class="sxs-lookup"><span data-stu-id="4b103-117">You can find the complete sample [here][sample-app].</span></span>

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

<span data-ttu-id="4b103-118">Im nächsten Beispiel ist ein kleineres Problem dargestellt, das durch die Art und Weise entsteht, wie LINQ to Entities von Entity Framework verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="4b103-118">The next example shows a subtle problem caused by the way Entity Framework uses LINQ to Entities.</span></span> 

```csharp
var query = from p in context.Products.AsEnumerable()
            where p.SellStartDate < DateTime.Now.AddDays(-7) // AddDays cannot be mapped by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

<span data-ttu-id="4b103-119">Die Anwendung versucht, Produkte mit einem Verkaufsstartdatum (`SellStartDate`) zu ermitteln, das länger als eine Woche zurückliegt.</span><span class="sxs-lookup"><span data-stu-id="4b103-119">The application is trying to find products with a `SellStartDate` more than a week old.</span></span> <span data-ttu-id="4b103-120">In den meisten Fällen übersetzt LINQ to Entities eine `where`-Klausel in eine SQL-Anweisung, die von der Datenbank ausgeführt wird.</span><span class="sxs-lookup"><span data-stu-id="4b103-120">In most cases, LINQ to Entities would translate a `where` clause to a SQL statement that is executed by the database.</span></span> <span data-ttu-id="4b103-121">Hier kann LINQ to Entities die `AddDays`-Methode aber nicht SQL zuordnen.</span><span class="sxs-lookup"><span data-stu-id="4b103-121">In this case, however, LINQ to Entities cannot map the `AddDays` method to SQL.</span></span> <span data-ttu-id="4b103-122">Stattdessen wird jede Zeile der Tabelle `Product` zurückgegeben, und die Ergebnisse werden im Arbeitsspeicher gefiltert.</span><span class="sxs-lookup"><span data-stu-id="4b103-122">Instead, every row from the `Product` table is returned, and the results are filtered in memory.</span></span> 

<span data-ttu-id="4b103-123">Der Aufruf von `AsEnumerable` ist ein Hinweis darauf, dass ein Problem vorliegt.</span><span class="sxs-lookup"><span data-stu-id="4b103-123">The call to `AsEnumerable` is a hint that there is a problem.</span></span> <span data-ttu-id="4b103-124">Bei dieser Methode werden die Ergebnisse in eine `IEnumerable`-Schnittstelle konvertiert.</span><span class="sxs-lookup"><span data-stu-id="4b103-124">This method converts the results to an `IEnumerable` interface.</span></span> <span data-ttu-id="4b103-125">`IEnumerable` unterstützt zwar die Filterung, aber der Filtervorgang wird aufseiten des *Clients* durchgeführt, nicht auf der Datenbankseite.</span><span class="sxs-lookup"><span data-stu-id="4b103-125">Although `IEnumerable` supports filtering, the filtering is done on the *client* side, not the database.</span></span> <span data-ttu-id="4b103-126">Standardmäßig nutzt LINQ to Entities die `IQueryable`-Schnittstelle, mit der die Zuständigkeit für die Filterung an die Datenquelle übergeben wird.</span><span class="sxs-lookup"><span data-stu-id="4b103-126">By default, LINQ to Entities uses `IQueryable`, which passes the responsibility for filtering to the data source.</span></span> 

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="4b103-127">Beheben des Problems</span><span class="sxs-lookup"><span data-stu-id="4b103-127">How to fix the problem</span></span>

<span data-ttu-id="4b103-128">Vermeiden Sie es, große Datenvolumen abzurufen, die schnell veraltet sind oder verworfen werden. Rufen Sie nur die Daten ab, die für den durchzuführenden Vorgang benötigt werden.</span><span class="sxs-lookup"><span data-stu-id="4b103-128">Avoid fetching large volumes of data that may quickly become outdated or might be discarded, and only fetch the data needed for the operation being performed.</span></span> 

<span data-ttu-id="4b103-129">Anstatt jede Spalte aus einer Tabelle abzurufen und dann zu filtern, ist es ratsam, die benötigten Spalten der Datenbank auszuwählen.</span><span class="sxs-lookup"><span data-stu-id="4b103-129">Instead of getting every column from a table and then filtering them, select the columns that you need from the database.</span></span>

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

<span data-ttu-id="4b103-130">Führen Sie außerdem die Aggregation in der Datenbank und nicht im Anwendungsspeicher durch.</span><span class="sxs-lookup"><span data-stu-id="4b103-130">Similarly, perform aggregation in the database and not in application memory.</span></span>

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

<span data-ttu-id="4b103-131">Stellen Sie bei Verwendung von Entity Framework sicher, dass LINQ-Abfragen mit der `IQueryable`-Schnittstelle und nicht mit `IEnumerable` aufgelöst werden.</span><span class="sxs-lookup"><span data-stu-id="4b103-131">When using Entity Framework, ensure that LINQ queries are resolved using the `IQueryable`interface and not `IEnumerable`.</span></span> <span data-ttu-id="4b103-132">Unter Umständen müssen Sie die Abfrage so anpassen, dass nur Funktionen verwendet werden, die der Datenquelle zugeordnet werden können.</span><span class="sxs-lookup"><span data-stu-id="4b103-132">You may need to adjust the query to use only functions that can be mapped to the data source.</span></span> <span data-ttu-id="4b103-133">Das vorherige Beispiel kann umgestaltet werden, um die `AddDays`-Methode aus der Abfrage zu entfernen, damit der Filtervorgang von der Datenbank durchgeführt werden kann.</span><span class="sxs-lookup"><span data-stu-id="4b103-133">The earlier example can be refactored to remove the `AddDays` method from the query, allowing filtering to be done by the database.</span></span>

```csharp
DateTime dateSince = DateTime.Now.AddDays(-7); // AddDays has been factored out. 
var query = from p in context.Products
            where p.SellStartDate < dateSince // This criterion can be passed to the database by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

## <a name="considerations"></a><span data-ttu-id="4b103-134">Überlegungen</span><span class="sxs-lookup"><span data-stu-id="4b103-134">Considerations</span></span>

- <span data-ttu-id="4b103-135">In einigen Fällen können Sie die Leistung verbessern, indem Sie die Daten horizontal partitionieren.</span><span class="sxs-lookup"><span data-stu-id="4b103-135">In some cases, you can improve performance by partitioning data horizontally.</span></span> <span data-ttu-id="4b103-136">Wenn unterschiedliche Vorgänge auf verschiedene Attribute der Daten zugreifen, kann eine horizontale Partitionierung bewirken, dass weniger Konflikte auftreten.</span><span class="sxs-lookup"><span data-stu-id="4b103-136">If different operations access different attributes of the data, horizontal partitioning may reduce contention.</span></span> <span data-ttu-id="4b103-137">Da die meisten Vorgänge häufig nur für eine kleinere Teilmenge der Daten ausgeführt werden, lässt sich die Leistung verbessern, indem diese Last verteilt wird.</span><span class="sxs-lookup"><span data-stu-id="4b103-137">Often, most operations are run against a small subset of the data, so spreading this load may improve performance.</span></span> <span data-ttu-id="4b103-138">Informationen hierzu finden Sie unter [Datenpartitionierung][data-partitioning].</span><span class="sxs-lookup"><span data-stu-id="4b103-138">See [Data partitioning][data-partitioning].</span></span>

- <span data-ttu-id="4b103-139">Für Vorgänge, mit denen uneingeschränkte Abfragen unterstützt werden, sollten Sie die Paginierung implementieren und jeweils nur eine begrenzte Anzahl von Entitäten abrufen.</span><span class="sxs-lookup"><span data-stu-id="4b103-139">For operations that have to support unbounded queries, implement pagination and only fetch a limited number of entities at a time.</span></span> <span data-ttu-id="4b103-140">Wenn sich ein Kunde beispielsweise einen Produktkatalog ansieht, können Sie jeweils eine Seite mit Ergebnissen anzeigen.</span><span class="sxs-lookup"><span data-stu-id="4b103-140">For example, if a customer is browsing a product catalog, you can show one page of results at a time.</span></span>

- <span data-ttu-id="4b103-141">Nutzen Sie nach Möglichkeit Features, die in den Datenspeicher integriert sind.</span><span class="sxs-lookup"><span data-stu-id="4b103-141">When possible, take advantage of features built into the data store.</span></span> <span data-ttu-id="4b103-142">SQL-Datenbanken enthalten in der Regel beispielsweise Aggregatfunktionen.</span><span class="sxs-lookup"><span data-stu-id="4b103-142">For example, SQL databases typically provide aggregate functions.</span></span> 

- <span data-ttu-id="4b103-143">Wenn Sie einen Datenspeicher verwenden, der eine bestimmte Funktion nicht unterstützt, z.B. die Aggregration, können Sie das berechnete Ergebnis an einem anderen Ort speichern und den Wert aktualisieren, wenn Datensätze hinzugefügt oder aktualisiert werden. Die Anwendung muss den Wert dann nicht jedes Mal neu berechnen, wenn er benötigt wird.</span><span class="sxs-lookup"><span data-stu-id="4b103-143">If you're using a data store that doesn't support a particular function, such as aggregration, you could store the calculated result elsewhere, updating the value as records are added or updated, so the application doesn't have to recalculate the value each time it's needed.</span></span>

- <span data-ttu-id="4b103-144">Wenn Sie sehen, dass Anforderungen eine große Anzahl von Feldern abrufen, sollten Sie den Quellcode untersuchen und ermitteln, ob alle Felder auch wirklich erforderlich sind.</span><span class="sxs-lookup"><span data-stu-id="4b103-144">If you see that requests are retrieving a large number of fields, examine the source code to determine whether all of these fields are actually necessary.</span></span> <span data-ttu-id="4b103-145">Diese Anforderungen können auch eine Folge einer schlecht entworfenen `SELECT *`-Abfrage sein.</span><span class="sxs-lookup"><span data-stu-id="4b103-145">Sometimes these requests are the result of poorly designed `SELECT *` query.</span></span> 

- <span data-ttu-id="4b103-146">Ebenso können Anforderungen, bei denen eine große Zahl von Entitäten abgerufen wird, ein Zeichen dafür sein, dass Daten von der Anwendung nicht richtig gefiltert werden.</span><span class="sxs-lookup"><span data-stu-id="4b103-146">Similarly, requests that retrieve a large number of entities may be sign that the application is not filtering data correctly.</span></span> <span data-ttu-id="4b103-147">Vergewissern Sie sich, dass alle Entitäten auch wirklich benötigt werden.</span><span class="sxs-lookup"><span data-stu-id="4b103-147">Verify that all of these entities are actually needed.</span></span> <span data-ttu-id="4b103-148">Nutzen Sie nach Möglichkeit die Filterung auf Datenbankseite, indem Sie beispielsweise `WHERE`-Klauseln in SQL verwenden.</span><span class="sxs-lookup"><span data-stu-id="4b103-148">Use database-side filtering if possible, for example, by using `WHERE` clauses in SQL.</span></span> 

- <span data-ttu-id="4b103-149">Das Auslagern der Verarbeitung in die Datenbank ist nicht immer die beste Vorgehensweise.</span><span class="sxs-lookup"><span data-stu-id="4b103-149">Offloading processing to the database is not always the best option.</span></span> <span data-ttu-id="4b103-150">Verwenden Sie diese Strategie nur, wenn die Datenbank dafür entworfen bzw. optimiert wurde.</span><span class="sxs-lookup"><span data-stu-id="4b103-150">Only use this strategy when the database is designed or optimized to do so.</span></span> <span data-ttu-id="4b103-151">Die meisten Datenbanksysteme sind speziell im Hinblick auf bestimmte Funktionen optimiert, aber nicht dafür ausgelegt, als Anwendungsmodule für allgemeine Zwecke zu fungieren.</span><span class="sxs-lookup"><span data-stu-id="4b103-151">Most database systems are highly optimized for certain functions, but are not designed to act as general-purpose application engines.</span></span> <span data-ttu-id="4b103-152">Weitere Informationen finden Sie unter [Antimuster „Busy Database“ (Ausgelastete Datenbank)][BusyDatabase].</span><span class="sxs-lookup"><span data-stu-id="4b103-152">For more information, see the [Busy Database antipattern][BusyDatabase].</span></span>


## <a name="how-to-detect-the-problem"></a><span data-ttu-id="4b103-153">Erkennen des Problems</span><span class="sxs-lookup"><span data-stu-id="4b103-153">How to detect the problem</span></span>

<span data-ttu-id="4b103-154">Zu den Symptomen des „Extraneous Fetching“ (Irrelevante Abrufe) gehören lange Wartezeiten und ein niedriger Durchsatz.</span><span class="sxs-lookup"><span data-stu-id="4b103-154">Symptoms of extraneous fetching include high latency and low throughput.</span></span> <span data-ttu-id="4b103-155">Wenn die Daten aus einem Datenspeicher abgerufen werden, steigt auch die Konfliktwahrscheinlichkeit.</span><span class="sxs-lookup"><span data-stu-id="4b103-155">If the data is retrieved from a data store, increased contention is also probable.</span></span> <span data-ttu-id="4b103-156">Endbenutzer berichten vermutlich von längeren Antwortzeiten oder Fehlern aufgrund von Diensten, bei denen ein Timeout auftritt. In diesen Fällen können Fehler vom Typ „HTTP 500 (Interner Server)“ oder „HTTP 503 (Dienst nicht verfügbar)“ zurückgegeben werden.</span><span class="sxs-lookup"><span data-stu-id="4b103-156">End users are likely to report extended response times or failures caused by services timing out. These failures could return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="4b103-157">Überprüfen Sie die Ereignisprotokolle für den Webserver. Sie enthalten wahrscheinlich ausführlichere Informationen zu den Ursachen und Umständen der Fehler.</span><span class="sxs-lookup"><span data-stu-id="4b103-157">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="4b103-158">Die Symptome dieses Antimusters und einige ermittelte Telemetriedaten können den Daten des [Antimusters „Monolithic Persistence“ (Monolithische Persistenz)][MonolithicPersistence] stark ähneln.</span><span class="sxs-lookup"><span data-stu-id="4b103-158">The symptoms of this antipattern and some of the telemetry obtained might be very similar to those of the [Monolithic Persistence antipattern][MonolithicPersistence].</span></span> 

<span data-ttu-id="4b103-159">Sie können die folgenden Schritte ausführen, um die Ursache zu ermitteln:</span><span class="sxs-lookup"><span data-stu-id="4b103-159">You can perform the following steps to help identify the cause:</span></span>

1. <span data-ttu-id="4b103-160">Identifizieren Sie langsame Workloads oder Transaktionen, indem Sie Auslastungstests, eine Prozessüberwachung und andere Verfahren zur Erfassung von Instrumentierungsdaten durchführen.</span><span class="sxs-lookup"><span data-stu-id="4b103-160">Identify slow workloads or transactions by performing load-testing, process monitoring, or other methods of capturing instrumentation data.</span></span>
2. <span data-ttu-id="4b103-161">Beobachten Sie alle Verhaltensmuster des Systems.</span><span class="sxs-lookup"><span data-stu-id="4b103-161">Observe any behavioral patterns exhibited by the system.</span></span> <span data-ttu-id="4b103-162">Bestehen bestimmte Beschränkungen in Bezug auf die Transaktionen pro Sekunde oder das Benutzervolumen?</span><span class="sxs-lookup"><span data-stu-id="4b103-162">Are there particular limits in terms of transactions per second or volume of users?</span></span>
3. <span data-ttu-id="4b103-163">Korrelieren Sie die Instanzen von langsamen Workloads mit Verhaltensmustern.</span><span class="sxs-lookup"><span data-stu-id="4b103-163">Correlate the instances of slow workloads with behavioral patterns.</span></span>
4. <span data-ttu-id="4b103-164">Identifizieren Sie die verwendeten Datenspeicher.</span><span class="sxs-lookup"><span data-stu-id="4b103-164">Identify the data stores being used.</span></span> <span data-ttu-id="4b103-165">Führen Sie für jede Datenquelle die Erfassung von Telemetriedaten auf niedriger Ebene aus, um das Verhalten von Vorgängen beobachten zu können.</span><span class="sxs-lookup"><span data-stu-id="4b103-165">For each data source, run lower level telemetry to observe the behavior of operations.</span></span>
6. <span data-ttu-id="4b103-166">Identifizieren Sie alle langsamen Abfragen, für die auf diese Datenquellen verwiesen wird.</span><span class="sxs-lookup"><span data-stu-id="4b103-166">Identify any slow-running queries that reference these data sources.</span></span>
7. <span data-ttu-id="4b103-167">Führen Sie eine ressourcenspezifische Analyse der langsamen Abfragen durch, und verfolgen Sie, wie die Daten verwendet und verbraucht werden.</span><span class="sxs-lookup"><span data-stu-id="4b103-167">Perform a resource-specific analysis of the slow-running queries and ascertain how the data is used and consumed.</span></span>

<span data-ttu-id="4b103-168">Suchen Sie nach folgenden Symptomen:</span><span class="sxs-lookup"><span data-stu-id="4b103-168">Look for any of these symptoms:</span></span>

- <span data-ttu-id="4b103-169">Häufige, umfangreiche E/A-Anforderungen, die an dieselbe Ressource oder einen Datenspeicher gesendet werden.</span><span class="sxs-lookup"><span data-stu-id="4b103-169">Frequent, large I/O requests made to the same resource or data store.</span></span>
- <span data-ttu-id="4b103-170">Auftreten von Konflikten in einer freigegebenen Ressource oder in einem Datenspeicher.</span><span class="sxs-lookup"><span data-stu-id="4b103-170">Contention in a shared resource or data store.</span></span>
- <span data-ttu-id="4b103-171">Ein Vorgang, für den häufig große Datenmengen über das Netzwerk eingehen.</span><span class="sxs-lookup"><span data-stu-id="4b103-171">An operation that frequently receives large volumes of data over the network.</span></span>
- <span data-ttu-id="4b103-172">Anwendungen und Dienste, für die sehr lange auf den Abschluss von E/A-Vorgängen gewartet wird.</span><span class="sxs-lookup"><span data-stu-id="4b103-172">Applications and services spending significant time waiting for I/O to complete.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="4b103-173">Beispieldiagnose</span><span class="sxs-lookup"><span data-stu-id="4b103-173">Example diagnosis</span></span>    

<span data-ttu-id="4b103-174">In den folgenden Abschnitten werden diese Schritte auf die vorherigen Beispiele angewendet.</span><span class="sxs-lookup"><span data-stu-id="4b103-174">The following sections apply these steps to the previous examples.</span></span>

### <a name="identify-slow-workloads"></a><span data-ttu-id="4b103-175">Identifizieren von langsamen Workloads</span><span class="sxs-lookup"><span data-stu-id="4b103-175">Identify slow workloads</span></span>

<span data-ttu-id="4b103-176">Mit diesem Graphen werden Leistungsergebnisse eines Auslastungstests dargestellt, bei denen bis zu 400 gleichzeitige Benutzer simuliert wurden, die die oben beschriebene `GetAllFieldsAsync`-Methode ausführen.</span><span class="sxs-lookup"><span data-stu-id="4b103-176">This graph shows performance results from a load test that simulated up to 400 concurrent users running the `GetAllFieldsAsync` method shown earlier.</span></span> <span data-ttu-id="4b103-177">Der Durchsatz nimmt langsam ab, wenn die Auslastung steigt.</span><span class="sxs-lookup"><span data-stu-id="4b103-177">Throughput diminishes slowly as the load increases.</span></span> <span data-ttu-id="4b103-178">Die durchschnittliche Reaktionszeit verlängert sich, wenn die Workload zunimmt.</span><span class="sxs-lookup"><span data-stu-id="4b103-178">Average response time goes up as the workload increases.</span></span> 

![Auslastungstestergebnisse für die GetAllFieldsAsync-Methode][Load-Test-Results-Client-Side1]

<span data-ttu-id="4b103-180">Bei einem Auslastungstest für den `AggregateOnClientAsync`-Vorgang zeigt sich ein ähnliches Muster.</span><span class="sxs-lookup"><span data-stu-id="4b103-180">A load test for the `AggregateOnClientAsync` operation shows a similar pattern.</span></span> <span data-ttu-id="4b103-181">Die Menge der Anforderungen ist relativ stabil.</span><span class="sxs-lookup"><span data-stu-id="4b103-181">The volume of requests is reasonably stable.</span></span> <span data-ttu-id="4b103-182">Die durchschnittliche Reaktionszeit nimmt mit der Workload zu, aber langsamer als im vorherigen Graphen.</span><span class="sxs-lookup"><span data-stu-id="4b103-182">The average response time increases with the workload, although more slowly than the previous graph.</span></span>

![Auslastungstestergebnisse für die AggregateOnClientAsync-Methode][Load-Test-Results-Client-Side2]

### <a name="correlate-slow-workloads-with-behavioral-patterns"></a><span data-ttu-id="4b103-184">Korrelieren von langsamen Workloads mit Verhaltensmustern</span><span class="sxs-lookup"><span data-stu-id="4b103-184">Correlate slow workloads with behavioral patterns</span></span>

<span data-ttu-id="4b103-185">Jede Korrelation zwischen normalen Zeiträumen mit hoher Nutzung und langsamer Leistung können Anzeichen für Probleme sein.</span><span class="sxs-lookup"><span data-stu-id="4b103-185">Any correlation between regular periods of high usage and slowing performance can indicate areas of concern.</span></span> <span data-ttu-id="4b103-186">Sehen Sie sich das Leistungsprofil von Funktionen, für die Leistungsprobleme vermutet werden, genau an. So können Sie bestimmen, ob das Profil zum zuvor durchgeführten Auslastungstest passt.</span><span class="sxs-lookup"><span data-stu-id="4b103-186">Closely examine the performance profile of functionality that is suspected to be slow running, to determine whether it matches the load testing performed earlier.</span></span>

<span data-ttu-id="4b103-187">Führen Sie Auslastungstests für dieselben Funktionen durch, indem Sie Benutzerlasten Schritt für Schritt anwenden, um den Punkt zu ermitteln, an dem die Leistung erheblich absinkt oder ganz ausfällt.</span><span class="sxs-lookup"><span data-stu-id="4b103-187">Load test the same functionality using step-based user loads, to find the point where performance drops significantly or fails completely.</span></span> <span data-ttu-id="4b103-188">Falls dieser Punkt innerhalb der Grenzen Ihres Nutzungsbereichs aus der Praxis liegt, sollten Sie untersuchen, wie die Funktionen implementiert wurden.</span><span class="sxs-lookup"><span data-stu-id="4b103-188">If that point falls within the bounds of your expected real-world usage, examine how the functionality is implemented.</span></span>

<span data-ttu-id="4b103-189">Ein langsamer Vorgang ist nicht unbedingt ein Problem, wenn Folgendes gilt: Er fällt nicht in Zeiten, in denen das System eine hohe Auslastung aufweist, es ist kein zeitkritischer Vorgang, und er wirkt sich nicht negativ auf die Leistung anderer wichtiger Vorgänge aus.</span><span class="sxs-lookup"><span data-stu-id="4b103-189">A slow operation is not necessarily a problem, if it is not being performed when the system is under stress, is not time critical, and does not negatively affect the performance of other important operations.</span></span> <span data-ttu-id="4b103-190">Die Generierung von monatlichen Statistiken zum Betrieb kann beispielsweise ein Vorgang mit langer Ausführungsdauer sein, aber er kann häufig als Batchprozess und Auftrag mit niedriger Priorität durchgeführt werden.</span><span class="sxs-lookup"><span data-stu-id="4b103-190">For example, generating monthly operational statistics might be a long-running operation, but it can probably be performed as a batch process and run as a low priority job.</span></span> <span data-ttu-id="4b103-191">Es ist aber ein kritischer Geschäftsvorgang, wenn Kunden den Produktkatalog durchsuchen.</span><span class="sxs-lookup"><span data-stu-id="4b103-191">On the other hand, customers querying the product catalog is a critical business operation.</span></span> <span data-ttu-id="4b103-192">Konzentrieren Sie sich auf die Telemetriedaten, die von diesen wichtigen Vorgängen generiert werden, um ermitteln zu können, wie die Leistung in Zeiten mit hoher Auslastung variiert.</span><span class="sxs-lookup"><span data-stu-id="4b103-192">Focus on the telemetry generated by these critical operations to see how the performance varies during periods of high usage.</span></span>

### <a name="identify-data-sources-in-slow-workloads"></a><span data-ttu-id="4b103-193">Identifizieren von Datenquellen bei langsamen Workloads</span><span class="sxs-lookup"><span data-stu-id="4b103-193">Identify data sources in slow workloads</span></span>

<span data-ttu-id="4b103-194">Wenn Sie vermuten, dass ein Dienst aufgrund der Art des Datenempfangs eine schlechte Leistung aufweist, sollten Sie untersuchen, wie die Anwendung mit den verwendeten Repositorys interagiert.</span><span class="sxs-lookup"><span data-stu-id="4b103-194">If you suspect that a service is performing poorly because of the way it retrieves data, investigate how the application interacts with the repositories it uses.</span></span> <span data-ttu-id="4b103-195">Überwachen Sie das Livesystem, um Informationen dazu zu erhalten, auf welche Quellen in Zeiten mit schlechter Leistung zugegriffen wird.</span><span class="sxs-lookup"><span data-stu-id="4b103-195">Monitor the live system to see which sources are accessed during periods of poor performance.</span></span> 

<span data-ttu-id="4b103-196">Instrumentieren Sie für jede Datenquelle das System, um Folgendes zu erfassen:</span><span class="sxs-lookup"><span data-stu-id="4b103-196">For each data source, instrument the system to capture the following:</span></span>

- <span data-ttu-id="4b103-197">Die Häufigkeit, mit der auf die einzelnen Datenspeicher zugegriffen wird.</span><span class="sxs-lookup"><span data-stu-id="4b103-197">The frequency that each data store is accessed.</span></span>
- <span data-ttu-id="4b103-198">Die Datenmengen, die im Datenspeicher abgelegt werden und diesen verlassen.</span><span class="sxs-lookup"><span data-stu-id="4b103-198">The volume of data entering and exiting the data store.</span></span>
- <span data-ttu-id="4b103-199">Den zeitlichen Ablauf dieser Vorgänge, vor allem die Wartezeit der Anforderungen.</span><span class="sxs-lookup"><span data-stu-id="4b103-199">The timing of these operations, especially the latency of requests.</span></span>
- <span data-ttu-id="4b103-200">Die Art und Häufigkeit der Fehler, die auftreten, während bei typischer Auslastung auf die einzelnen Datenspeicher zugegriffen wird.</span><span class="sxs-lookup"><span data-stu-id="4b103-200">The nature and rate of any errors that occur while accessing each data store under typical load.</span></span>

<span data-ttu-id="4b103-201">Vergleichen Sie diese Informationen mit dem Datenvolumen, das von der Anwendung an den Client zurückgegeben wird.</span><span class="sxs-lookup"><span data-stu-id="4b103-201">Compare this information against the volume of data being returned by the application to the client.</span></span> <span data-ttu-id="4b103-202">Verfolgen Sie das Verhältnis des zurückgegebenen Datenvolumens für den Datenspeicher und des an den Client zurückgegebenen Datenvolumens nach.</span><span class="sxs-lookup"><span data-stu-id="4b103-202">Track the ratio of the volume of data returned by the data store against the volume of data returned to the client.</span></span> <span data-ttu-id="4b103-203">Falls ein großer Unterschied zu erkennen ist, sollten Sie den Vorgang untersuchen und ermitteln, ob von der Anwendung nicht benötigte Daten abgerufen werden.</span><span class="sxs-lookup"><span data-stu-id="4b103-203">If there is any large disparity, investigate to determine whether the application is fetching data that it doesn't need.</span></span>

<span data-ttu-id="4b103-204">Unter Umständen können Sie diese Daten erfassen, indem Sie das Livesystem überwachen und den Lebenszyklus jeder Benutzeranforderung nachverfolgen, oder Sie können eine Serie von synthetischen Workloads modellieren und für ein Testsystem ausführen.</span><span class="sxs-lookup"><span data-stu-id="4b103-204">You may be able to capture this data by observing the live system and tracing the lifecycle of each user request, or you can model a series of synthetic workloads and run them against a test system.</span></span>

<span data-ttu-id="4b103-205">Mit den folgenden Graphen werden Telemetriedaten dargestellt, die per [New Relic APM][new-relic] während eines Auslastungstests der `GetAllFieldsAsync`-Methode erfasst wurden.</span><span class="sxs-lookup"><span data-stu-id="4b103-205">The following graphs show telemetry captured using [New Relic APM][new-relic] during a load test of the `GetAllFieldsAsync` method.</span></span> <span data-ttu-id="4b103-206">Beachten Sie den Unterschied zwischen der Datenmenge, die von der Datenbank empfangen wird, und den dazugehörigen HTTP-Antworten.</span><span class="sxs-lookup"><span data-stu-id="4b103-206">Note the difference between the volumes of data received from the database and the corresponding HTTP responses.</span></span>

![Telemetriedaten für die GetAllFieldsAsync-Methode][TelemetryAllFields]

<span data-ttu-id="4b103-208">Für jede Anforderung hat die Datenbank 80.503 Byte zurückgegeben, aber die Antwort an den Client hat nur 19.855 Byte enthalten. Dies sind ca. 25% der Größe der Datenbankantwort.</span><span class="sxs-lookup"><span data-stu-id="4b103-208">For each request, the database returned 80,503 bytes, but the response to the client only contained 19,855 bytes, about 25% of the size of the database response.</span></span> <span data-ttu-id="4b103-209">Die Größe der an den Client zurückgegebenen Daten kann je nach Format variieren.</span><span class="sxs-lookup"><span data-stu-id="4b103-209">The size of the data returned to the client can vary depending on the format.</span></span> <span data-ttu-id="4b103-210">Für diesen Auslastungstest hat der Client JSON-Daten angefordert.</span><span class="sxs-lookup"><span data-stu-id="4b103-210">For this load test, the client requested JSON data.</span></span> <span data-ttu-id="4b103-211">Separate Tests mit XML (nicht dargestellt) verfügten über eine Antwortgröße von 35.655 Byte oder 44% der Größe der Datenbankantwort.</span><span class="sxs-lookup"><span data-stu-id="4b103-211">Separate testing using XML (not shown) had a response size of 35,655 bytes, or 44% of the size of the database response.</span></span>

<span data-ttu-id="4b103-212">Beim Auslastungstest für die `AggregateOnClientAsync`-Methode sind die Ergebnisse weniger gemäßigt.</span><span class="sxs-lookup"><span data-stu-id="4b103-212">The load test for the `AggregateOnClientAsync` method shows more extreme results.</span></span> <span data-ttu-id="4b103-213">In diesem Fall wurde für jeden Test eine Abfrage durchgeführt, bei der jeweils mehr als 280 KB an Daten aus der Datenbank abgerufen wurden, während die JSON-Antwort nur 14 Byte umfasst.</span><span class="sxs-lookup"><span data-stu-id="4b103-213">In this case, each test performed a query that retrieved over 280Kb of data from the database, but the JSON response was a mere 14 bytes.</span></span> <span data-ttu-id="4b103-214">Es kommt zu diesem großen Unterschied, weil die Methode ein aggregiertes Ergebnis aus einer großen Datenmenge berechnet.</span><span class="sxs-lookup"><span data-stu-id="4b103-214">The wide disparity is because the method calculates an aggregated result from a large volume of data.</span></span>

![Telemetriedaten für die AggregateOnClientAsync-Methode][TelemetryAggregateOnClient]

### <a name="identify-and-analyze-slow-queries"></a><span data-ttu-id="4b103-216">Identifizieren und Analysieren von langsamen Abfragen</span><span class="sxs-lookup"><span data-stu-id="4b103-216">Identify and analyze slow queries</span></span>

<span data-ttu-id="4b103-217">Suchen Sie nach den Datenbankabfragen, die die meisten Ressourcen verbrauchen und deren Ausführung am längsten dauert.</span><span class="sxs-lookup"><span data-stu-id="4b103-217">Look for database queries that consume the most resources and take the most time to execute.</span></span> <span data-ttu-id="4b103-218">Sie können eine Instrumentierung hinzufügen, um die Start- und Endzeiten für viele Datenbankvorgänge zu ermitteln.</span><span class="sxs-lookup"><span data-stu-id="4b103-218">You can add instrumentation to find the start and completion times for many database operations.</span></span> <span data-ttu-id="4b103-219">Viele Datenspeicher liefern zudem eingehende Informationen dazu, wie Abfragen durchgeführt und optimiert werden.</span><span class="sxs-lookup"><span data-stu-id="4b103-219">Many data stores also provide in-depth information on how queries are performed and optimized.</span></span> <span data-ttu-id="4b103-220">Im Bereich „Abfrageleistung“ im Verwaltungsportal von Azure SQL-Datenbank können Sie beispielsweise eine Abfrage auswählen und ausführliche Informationen zur Laufzeitleistung anzeigen.</span><span class="sxs-lookup"><span data-stu-id="4b103-220">For example, the Query Performance pane in the Azure SQL Database management portal lets you select a query and view detailed runtime performance information.</span></span> <span data-ttu-id="4b103-221">Dies ist die Abfrage, die vom `GetAllFieldsAsync`-Vorgang generiert wird:</span><span class="sxs-lookup"><span data-stu-id="4b103-221">Here is the query generated by the `GetAllFieldsAsync` operation:</span></span>

![Bereich mit Abfragedetails im Verwaltungsportal von Windows Azure SQL-Datenbank][QueryDetails]


## <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="4b103-223">Implementieren der Lösung und Überprüfen des Ergebnisses</span><span class="sxs-lookup"><span data-stu-id="4b103-223">Implement the solution and verify the result</span></span>

<span data-ttu-id="4b103-224">Nach dem Umstellen der `GetRequiredFieldsAsync`-Methode auf die Verwendung einer SELECT-Anweisung auf Datenbankseite wurden für den Auslastungstest die folgenden Ergebnisse angezeigt:</span><span class="sxs-lookup"><span data-stu-id="4b103-224">After changing the `GetRequiredFieldsAsync` method to use a SELECT statement on the database side, load testing showed the following results.</span></span>

![Auslastungstestergebnisse für die GetRequiredFieldsAsync-Methode][Load-Test-Results-Database-Side1]

<span data-ttu-id="4b103-226">Für diesen Auslastungstest wurden dieselbe Bereitstellung und dieselbe simulierte Workload mit 400 gleichzeitigen Benutzern wie zuvor verwendet.</span><span class="sxs-lookup"><span data-stu-id="4b103-226">This load test used the same deployment and the same simulated workload of 400 concurrent users as before.</span></span> <span data-ttu-id="4b103-227">Der Graph zeigt eine viel niedrigere Wartezeit.</span><span class="sxs-lookup"><span data-stu-id="4b103-227">The graph shows much lower latency.</span></span> <span data-ttu-id="4b103-228">Die Antwortzeiten verkürzen sich unter Last auf ca. 1,3 Sekunden, verglichen mit vier Sekunden im vorherigen Fall.</span><span class="sxs-lookup"><span data-stu-id="4b103-228">Response time rises with load to approximately 1.3 seconds, compared to 4 seconds in the previous case.</span></span> <span data-ttu-id="4b103-229">Der Durchsatz ist mit 350 Anforderungen pro Sekunde gegenüber 100 Anforderungen zuvor ebenfalls höher.</span><span class="sxs-lookup"><span data-stu-id="4b103-229">The throughput is also higher at 350 requests per second compared to 100 earlier.</span></span> <span data-ttu-id="4b103-230">Die Datenmenge, die aus der Datenbank abgerufen wird, liegt nun in der Nähe der Größe der HTTP-Antwortnachrichten.</span><span class="sxs-lookup"><span data-stu-id="4b103-230">The volume of data retrieved from the database now closely matches the size of the HTTP response messages.</span></span>

![Telemetriedaten für die GetRequiredFieldsAsync-Methode][TelemetryRequiredFields]

<span data-ttu-id="4b103-232">Beim Auslastungstest mit der `AggregateOnDatabaseAsync`-Methode werden die folgenden Ergebnisse generiert:</span><span class="sxs-lookup"><span data-stu-id="4b103-232">Load testing using the `AggregateOnDatabaseAsync` method generates the following results:</span></span>

![Auslastungstestergebnisse für die AggregateOnDatabaseAsync-Methode][Load-Test-Results-Database-Side2]

<span data-ttu-id="4b103-234">Die durchschnittliche Antwortzeit ist jetzt sehr niedrig.</span><span class="sxs-lookup"><span data-stu-id="4b103-234">The average response time is now minimal.</span></span> <span data-ttu-id="4b103-235">Dies ist eine deutliche Verbesserung der Leistung, die hauptsächlich durch die starke Reduzierung der E/A-Vorgänge für die Datenbank erzielt wurde.</span><span class="sxs-lookup"><span data-stu-id="4b103-235">This is an order of magnitude improvement in performance, caused primarily by the large reduction in I/O from the database.</span></span> 

<span data-ttu-id="4b103-236">Hier sind die entsprechenden Telemetriedaten für die `AggregateOnDatabaseAsync`-Methode dargestellt.</span><span class="sxs-lookup"><span data-stu-id="4b103-236">Here is the corresponding telemetry for the `AggregateOnDatabaseAsync` method.</span></span> <span data-ttu-id="4b103-237">Die aus der Datenbank abgerufene Datenmenge wurde stark reduziert, und zwar von über 280 KB pro Transaktion auf 53 Byte.</span><span class="sxs-lookup"><span data-stu-id="4b103-237">The amount of data retrieved from the database was vastly reduced, from over 280Kb per transaction to 53 bytes.</span></span> <span data-ttu-id="4b103-238">Aus diesem Grund ist die maximale dauerhafte Anzahl von Anforderungen pro Minute von ca. 2.000 auf über 25.000 angestiegen.</span><span class="sxs-lookup"><span data-stu-id="4b103-238">As a result, the maximum sustained number of requests per minute was raised from around 2,000 to over 25,000.</span></span>

![Telemetriedaten für die AggregateOnDatabaseAsync-Methode][TelemetryAggregateInDatabaseAsync]


## <a name="related-resources"></a><span data-ttu-id="4b103-240">Zugehörige Ressourcen</span><span class="sxs-lookup"><span data-stu-id="4b103-240">Related resources</span></span>

- <span data-ttu-id="4b103-241">[Antimuster „Busy Database“][BusyDatabase] (Ausgelastete Datenbank)</span><span class="sxs-lookup"><span data-stu-id="4b103-241">[Busy Database antipattern][BusyDatabase]</span></span>
- <span data-ttu-id="4b103-242">[Antimuster „Chatty I/O“][chatty-io] (Sehr hohe Zahl von E/A-Vorgängen)</span><span class="sxs-lookup"><span data-stu-id="4b103-242">[Chatty I/O antipattern][chatty-io]</span></span>
- <span data-ttu-id="4b103-243">[Data partitioning][data-partitioning] (Datenpartitionierung)</span><span class="sxs-lookup"><span data-stu-id="4b103-243">[Data partitioning best practices][data-partitioning]</span></span>


[BusyDatabase]: ../busy-database/index.md
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
