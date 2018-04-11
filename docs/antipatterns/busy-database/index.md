---
title: Antimuster „Ausgelastete Datenbank“
description: Das Auslagern der Verarbeitung an einen Datenbankserver kann zu Leistungs- und Skalierbarkeitsproblemen führen.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 9fdbde0731a1be570ef611894a9d23a1be87f4e7
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="busy-database-antipattern"></a><span data-ttu-id="164fc-103">Antimuster „Ausgelastete Datenbank“</span><span class="sxs-lookup"><span data-stu-id="164fc-103">Busy Database antipattern</span></span>

<span data-ttu-id="164fc-104">Das Auslagern der Verarbeitung an einen Datenbankserver kann dazu führen, dass dieser Server mehr mit der Ausführung von Code beschäftigt ist, anstatt auf Anforderungen zum Speichern und Abrufen von Daten zu reagieren.</span><span class="sxs-lookup"><span data-stu-id="164fc-104">Offloading processing to a database server can cause it to spend a significant proportion of time running code, rather than responding to requests to store and retrieve data.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="164fc-105">Problembeschreibung</span><span class="sxs-lookup"><span data-stu-id="164fc-105">Problem description</span></span>

<span data-ttu-id="164fc-106">Viele Datenbanksysteme können Code ausführen.</span><span class="sxs-lookup"><span data-stu-id="164fc-106">Many database systems can run code.</span></span> <span data-ttu-id="164fc-107">Hierzu gehören z.B. gespeicherte Prozeduren und Trigger.</span><span class="sxs-lookup"><span data-stu-id="164fc-107">Examples include stored procedures and triggers.</span></span> <span data-ttu-id="164fc-108">Häufig ist es effizienter, diese Verarbeitung in der Nähe der Daten auszuführen, als die Daten zur Verarbeitung an eine Clientanwendung zu übertragen.</span><span class="sxs-lookup"><span data-stu-id="164fc-108">Often, it's more efficient to perform this processing close to the data, rather than transmitting the data to a client application for processing.</span></span> <span data-ttu-id="164fc-109">Eine übermäßige Verwendung dieser Features kann jedoch aus verschiedenen Gründen die Leistung beeinträchtigen:</span><span class="sxs-lookup"><span data-stu-id="164fc-109">However, overusing these features can hurt performance, for several reasons:</span></span>

- <span data-ttu-id="164fc-110">Der Datenbankserver wendet zu viel Zeit für die Verarbeitung auf, anstatt neue Clientanforderungen zu akzeptieren und Daten abzurufen.</span><span class="sxs-lookup"><span data-stu-id="164fc-110">The database server may spend too much time processing, rather than accepting new client requests and fetching data.</span></span>
- <span data-ttu-id="164fc-111">Eine Datenbank ist üblicherweise eine gemeinsam genutzte Ressource, daher kann sie in Zeiträumen intensiver Nutzung zu einem Engpass werden.</span><span class="sxs-lookup"><span data-stu-id="164fc-111">A database is usually a shared resource, so it can become a bottleneck during periods of high use.</span></span>
- <span data-ttu-id="164fc-112">Die Laufzeitkosten können übermäßig hoch sein, wenn die Abrechnung für den Datenspeicher auf Verbrauchseinheiten basiert.</span><span class="sxs-lookup"><span data-stu-id="164fc-112">Runtime costs may be excessive if the data store is metered.</span></span> <span data-ttu-id="164fc-113">Dies gilt insbesondere für verwaltete Datenbankdienste.</span><span class="sxs-lookup"><span data-stu-id="164fc-113">That's particularly true of managed database services.</span></span> <span data-ttu-id="164fc-114">Azure SQL-Datenbank z.B. berechnet [Datenbanktransaktionseinheiten][dtu] (Database Transaction Units, DTUs).</span><span class="sxs-lookup"><span data-stu-id="164fc-114">For example, Azure SQL Database charges for [Database Transaction Units][dtu] (DTUs).</span></span>
- <span data-ttu-id="164fc-115">Datenbankkapazitäten lassen sich nicht endlos zentral hochskalieren, und die horizontale Skalierung einer Datenbank ist nicht gerade einfach.</span><span class="sxs-lookup"><span data-stu-id="164fc-115">Databases have finite capacity to scale up, and it's not trivial to scale a database horizontally.</span></span> <span data-ttu-id="164fc-116">Daher kann es sinnvoller sein, die Verarbeitung an eine Computeressource auszulagern, z.B. eine VM oder eine App Service-App, die sich problemlos horizontal hochskalieren lässt.</span><span class="sxs-lookup"><span data-stu-id="164fc-116">Therefore, it may be better to move processing into a compute resource, such as a VM or App Service app, that can easily scale out.</span></span>

<span data-ttu-id="164fc-117">Dieses Antimuster tritt üblicherweise aus folgenden Gründen auf:</span><span class="sxs-lookup"><span data-stu-id="164fc-117">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="164fc-118">Die Datenbank wird als Dienst betrachtet, nicht als Repository.</span><span class="sxs-lookup"><span data-stu-id="164fc-118">The database is viewed as a service rather than a repository.</span></span> <span data-ttu-id="164fc-119">Eine Anwendung verwendet den Datenbankserver möglicherweise, um Daten zu formatieren (z.B. zum Konvertieren in XML), Zeichenfolgendaten zu ändern oder komplexe Berechnungen auszuführen.</span><span class="sxs-lookup"><span data-stu-id="164fc-119">An application might use the database server to format data (for example, converting to XML), manipulate string data, or perform complex calculations.</span></span>
- <span data-ttu-id="164fc-120">Entwickler versuchen, Abfragen zu schreiben, deren Ergebnisse den Benutzern direkt angezeigt werden können.</span><span class="sxs-lookup"><span data-stu-id="164fc-120">Developers try to write queries whose results can be displayed directly to users.</span></span> <span data-ttu-id="164fc-121">Eine Abfrage kann beispielsweise Felder kombinieren oder Datums-, Uhrzeit- und Währungswerte entsprechend dem Gebietsschema formatieren.</span><span class="sxs-lookup"><span data-stu-id="164fc-121">For example a query might combine fields, or format dates, times, and currency according to locale.</span></span>
- <span data-ttu-id="164fc-122">Entwickler versuchen, das Antimuster [Irrelevante Abrufe][ExtraneousFetching] zu korrigieren, indem sie Berechnungen in die Datenbank verschieben.</span><span class="sxs-lookup"><span data-stu-id="164fc-122">Developers are trying to correct the [Extraneous Fetching][ExtraneousFetching] antipattern by pushing computations to the database.</span></span>
- <span data-ttu-id="164fc-123">Gespeicherte Prozeduren werden verwendet, um Geschäftslogik zu kapseln, möglicherweise, weil sie als einfacher zu verwalten und zu aktualisieren angesehen werden.</span><span class="sxs-lookup"><span data-stu-id="164fc-123">Stored procedures are used to encapsulate business logic, perhaps because they are considered easier to maintain and update.</span></span>

<span data-ttu-id="164fc-124">Das folgende Beispiel ruft die 20 Bestellungen mit dem höchsten Wert für ein bestimmtes Vertriebsgebiet ab und formatiert die Ergebnisse als XML.</span><span class="sxs-lookup"><span data-stu-id="164fc-124">The following example retrieves the 20 most valuable orders for a specified sales territory and formats the results as XML.</span></span>
<span data-ttu-id="164fc-125">Es verwendet Transact-SQL-Funktionen, um die Daten zu analysieren und die Ergebnisse zu XML zu konvertieren.</span><span class="sxs-lookup"><span data-stu-id="164fc-125">It uses Transact-SQL functions to parse the data and convert the results to XML.</span></span> <span data-ttu-id="164fc-126">Das vollständige Codebeispiel finden Sie [hier][sample-app].</span><span class="sxs-lookup"><span data-stu-id="164fc-126">You can find the complete sample [here][sample-app].</span></span>

```SQL
SELECT TOP 20
  soh.[SalesOrderNumber]  AS '@OrderNumber',
  soh.[Status]            AS '@Status',
  soh.[ShipDate]          AS '@ShipDate',
  YEAR(soh.[OrderDate])   AS '@OrderDateYear',
  MONTH(soh.[OrderDate])  AS '@OrderDateMonth',
  soh.[DueDate]           AS '@DueDate',
  FORMAT(ROUND(soh.[SubTotal],2),'C')
                          AS '@SubTotal',
  FORMAT(ROUND(soh.[TaxAmt],2),'C')
                          AS '@TaxAmt',
  FORMAT(ROUND(soh.[TotalDue],2),'C')
                          AS '@TotalDue',
  CASE WHEN soh.[TotalDue] > 5000 THEN 'Y' ELSE 'N' END
                          AS '@ReviewRequired',
  (
  SELECT
    c.[AccountNumber]     AS '@AccountNumber',
    UPPER(LTRIM(RTRIM(REPLACE(
    CONCAT( p.[Title], ' ', p.[FirstName], ' ', p.[MiddleName], ' ', p.[LastName], ' ', p.[Suffix]),
    '  ', ' '))))         AS '@FullName'
  FROM [Sales].[Customer] c
    INNER JOIN [Person].[Person] p
  ON c.[PersonID] = p.[BusinessEntityID]
  WHERE c.[CustomerID] = soh.[CustomerID]
  FOR XML PATH ('Customer'), TYPE
  ),

  (
  SELECT
    sod.[OrderQty]      AS '@Quantity',
    FORMAT(sod.[UnitPrice],'C')
                        AS '@UnitPrice',
    FORMAT(ROUND(sod.[LineTotal],2),'C')
                        AS '@LineTotal',
    sod.[ProductID]     AS '@ProductId',
    CASE WHEN (sod.[ProductID] >= 710) AND (sod.[ProductID] <= 720) AND (sod.[OrderQty] >= 5) THEN 'Y' ELSE 'N' END
                        AS '@InventoryCheckRequired'

  FROM [Sales].[SalesOrderDetail] sod
  WHERE sod.[SalesOrderID] = soh.[SalesOrderID]
  ORDER BY sod.[SalesOrderDetailID]
  FOR XML PATH ('LineItem'), TYPE, ROOT('OrderLineItems')
  )

FROM [Sales].[SalesOrderHeader] soh
WHERE soh.[TerritoryId] = @TerritoryId
ORDER BY soh.[TotalDue] DESC
FOR XML PATH ('Order'), ROOT('Orders')
```

<span data-ttu-id="164fc-127">Dies ist zweifellos eine komplexe Abfrage.</span><span class="sxs-lookup"><span data-stu-id="164fc-127">Clearly, this is complex query.</span></span> <span data-ttu-id="164fc-128">Wie wir später sehen werden, benötigt sie eine erhebliche Menge an Verarbeitungsressourcen auf dem Datenbankserver.</span><span class="sxs-lookup"><span data-stu-id="164fc-128">As we'll see later, it turns out to use significant processing resources on the database server.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="164fc-129">Beheben des Problems</span><span class="sxs-lookup"><span data-stu-id="164fc-129">How to fix the problem</span></span>

<span data-ttu-id="164fc-130">Verschieben Sie Verarbeitungsvorgänge vom Datenbankserver auf andere Logikschichten.</span><span class="sxs-lookup"><span data-stu-id="164fc-130">Move processing from the database server into other application tiers.</span></span> <span data-ttu-id="164fc-131">Idealerweise sollten Sie die Datenbank auf die Ausführung von Datenzugriffsvorgängen beschränken, sodass nur die Funktionalitäten verwendet werden, für die die Datenbank optimiert wurde – z.B. die Aggregation in einem Managementsystem für relationale Datenbanken (RDBMS).</span><span class="sxs-lookup"><span data-stu-id="164fc-131">Ideally, you should limit the database to performing data access operations, using only the capabilities that the database is optimized for, such as aggregation in an RDBMS.</span></span>

<span data-ttu-id="164fc-132">Der oben stehende Transact-SQL-Code kann z.B. durch eine Anweisung ersetzt werden, die einfach die zu verarbeitenden Daten abruft.</span><span class="sxs-lookup"><span data-stu-id="164fc-132">For example, the previous Transact-SQL code can be replaced with a statement that simply retrieves the data to be processed.</span></span>

```SQL
SELECT
soh.[SalesOrderNumber]  AS [OrderNumber],
soh.[Status]            AS [Status],
soh.[OrderDate]         AS [OrderDate],
soh.[DueDate]           AS [DueDate],
soh.[ShipDate]          AS [ShipDate],
soh.[SubTotal]          AS [SubTotal],
soh.[TaxAmt]            AS [TaxAmt],
soh.[TotalDue]          AS [TotalDue],
c.[AccountNumber]       AS [AccountNumber],
p.[Title]               AS [CustomerTitle],
p.[FirstName]           AS [CustomerFirstName],
p.[MiddleName]          AS [CustomerMiddleName],
p.[LastName]            AS [CustomerLastName],
p.[Suffix]              AS [CustomerSuffix],
sod.[OrderQty]          AS [Quantity],
sod.[UnitPrice]         AS [UnitPrice],
sod.[LineTotal]         AS [LineTotal],
sod.[ProductID]         AS [ProductId]
FROM [Sales].[SalesOrderHeader] soh
INNER JOIN [Sales].[Customer] c ON soh.[CustomerID] = c.[CustomerID]
INNER JOIN [Person].[Person] p ON c.[PersonID] = p.[BusinessEntityID]
INNER JOIN [Sales].[SalesOrderDetail] sod ON soh.[SalesOrderID] = sod.[SalesOrderID]
WHERE soh.[TerritoryId] = @TerritoryId
AND soh.[SalesOrderId] IN (
    SELECT TOP 20 SalesOrderId
    FROM [Sales].[SalesOrderHeader] soh
    WHERE soh.[TerritoryId] = @TerritoryId
    ORDER BY soh.[TotalDue] DESC)
ORDER BY soh.[TotalDue] DESC, sod.[SalesOrderDetailID]
```

<span data-ttu-id="164fc-133">Die Anwendung verwendet dann die .NET Framework-`System.Xml.Linq`-APIs, um die Ergebnisse als XML zu formatieren.</span><span class="sxs-lookup"><span data-stu-id="164fc-133">The application then uses the .NET Framework `System.Xml.Linq` APIs to format the results as XML.</span></span>

```csharp
// Create a new SqlCommand to run the Transact-SQL query
using (var command = new SqlCommand(...))
{
    command.Parameters.AddWithValue("@TerritoryId", id);

    // Run the query and create the initial XML document
    using (var reader = await command.ExecuteReaderAsync())
    {
        var lastOrderNumber = string.Empty;
        var doc = new XDocument();
        var orders = new XElement("Orders");
        doc.Add(orders);

        XElement lineItems = null;
        // Fetch each row in turn, format the results as XML, and add them to the XML document
        while (await reader.ReadAsync())
        {
            var orderNumber = reader["OrderNumber"].ToString();
            if (orderNumber != lastOrderNumber)
            {
                lastOrderNumber = orderNumber;

                var order = new XElement("Order");
                orders.Add(order);
                var customer = new XElement("Customer");
                lineItems = new XElement("OrderLineItems");
                order.Add(customer, lineItems);

                var orderDate = (DateTime)reader["OrderDate"];
                var totalDue = (Decimal)reader["TotalDue"];
                var reviewRequired = totalDue > 5000 ? 'Y' : 'N';

                order.Add(
                    new XAttribute("OrderNumber", orderNumber),
                    new XAttribute("Status", reader["Status"]),
                    new XAttribute("ShipDate", reader["ShipDate"]),
                    ... // More attributes, not shown.

                    var fullName = string.Join(" ",
                        reader["CustomerTitle"],
                        reader["CustomerFirstName"],
                        reader["CustomerMiddleName"],
                        reader["CustomerLastName"],
                        reader["CustomerSuffix"]
                    )
                   .Replace("  ", " ") //remove double spaces
                   .Trim()
                   .ToUpper();

               customer.Add(
                    new XAttribute("AccountNumber", reader["AccountNumber"]),
                    new XAttribute("FullName", fullName));
            }

            var productId = (int)reader["ProductID"];
            var quantity = (short)reader["Quantity"];
            var inventoryCheckRequired = (productId >= 710 && productId <= 720 && quantity >= 5) ? 'Y' : 'N';

            lineItems.Add(
                new XElement("LineItem",
                    new XAttribute("Quantity", quantity),
                    new XAttribute("UnitPrice", ((Decimal)reader["UnitPrice"]).ToString("C")),
                    new XAttribute("LineTotal", RoundAndFormat(reader["LineTotal"])),
                    new XAttribute("ProductId", productId),
                    new XAttribute("InventoryCheckRequired", inventoryCheckRequired)
                ));
        }
        // Match the exact formatting of the XML returned from SQL
        var xml = doc
            .ToString(SaveOptions.DisableFormatting)
            .Replace(" />", "/>");
    }
}
```

> [!NOTE]
> <span data-ttu-id="164fc-134">Dieser Code ist einigermaßen komplex.</span><span class="sxs-lookup"><span data-stu-id="164fc-134">This code is somewhat complex.</span></span> <span data-ttu-id="164fc-135">In einer neuen Anwendung sollten Sie eher eine Serialisierungsbibliothek verwenden.</span><span class="sxs-lookup"><span data-stu-id="164fc-135">For a new application, you might prefer to use a serialization library.</span></span> <span data-ttu-id="164fc-136">Hier geht es jedoch darum, dass das Entwicklungsteam ein Refactoring für eine vorhandene Anwendung durchführt. Daher muss die Methode exakt das gleiche Format zurückgeben wie der ursprüngliche Code.</span><span class="sxs-lookup"><span data-stu-id="164fc-136">However, the assumption here is that the development team is refactoring an existing application, so the method needs to return the exact same format as the original code.</span></span>

## <a name="considerations"></a><span data-ttu-id="164fc-137">Überlegungen</span><span class="sxs-lookup"><span data-stu-id="164fc-137">Considerations</span></span>

- <span data-ttu-id="164fc-138">Viele Datenbanksysteme sind in hohem Maß für bestimmte Arten der Datenverarbeitung optimiert, beispielsweise für die Berechnung von Aggregatwerten in großen Datasets.</span><span class="sxs-lookup"><span data-stu-id="164fc-138">Many database systems are highly optimized to perform certain types of data processing, such as calculating aggregate values over large datasets.</span></span> <span data-ttu-id="164fc-139">Lagern Sie diese Datenverarbeitungsarten nicht aus der Datenbank aus.</span><span class="sxs-lookup"><span data-stu-id="164fc-139">Don't move those types of processing out of the database.</span></span>

- <span data-ttu-id="164fc-140">Verlagern Sie Verarbeitungsprozesse nicht, wenn in diesem Fall die Datenbank weitaus mehr Daten über das Netzwerk übertragen müsste.</span><span class="sxs-lookup"><span data-stu-id="164fc-140">Do not relocate processing if doing so causes the database to transfer far more data over the network.</span></span> <span data-ttu-id="164fc-141">Weitere Informationen finden Sie unter [Antimuster „Irrelevante Abrufe“][ExtraneousFetching].</span><span class="sxs-lookup"><span data-stu-id="164fc-141">See the [Extraneous Fetching antipattern][ExtraneousFetching].</span></span>

- <span data-ttu-id="164fc-142">Wenn Sie die Verarbeitung auf eine Logikschicht verlagern, muss diese Schicht möglicherweise horizontal hochskaliert werden, um die zusätzliche Verarbeitung auszuführen.</span><span class="sxs-lookup"><span data-stu-id="164fc-142">If you move processing to an application tier, that tier may need to scale out to handle the additional work.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="164fc-143">Erkennen des Problems</span><span class="sxs-lookup"><span data-stu-id="164fc-143">How to detect the problem</span></span>

<span data-ttu-id="164fc-144">Zu den Symptomen einer ausgelasteten Datenbank gehört ein unverhältnismäßiges Absinken der Durchsatzwerte und Antwortzeiten bei Vorgängen, die auf die Datenbank zugreifen.</span><span class="sxs-lookup"><span data-stu-id="164fc-144">Symptoms of a busy database include a disproportionate decline in throughput and response times in operations that access the database.</span></span> 

<span data-ttu-id="164fc-145">Sie können die folgenden Schritte ausführen, um dieses Problem zu identifizieren:</span><span class="sxs-lookup"><span data-stu-id="164fc-145">You can perform the following steps to help identify this problem:</span></span> 

1. <span data-ttu-id="164fc-146">Verwenden Sie die Leistungsüberwachung, um zu ermitteln, wie viel Zeit das Produktionssystem für die Ausführung von Datenbankaktivitäten aufwendet.</span><span class="sxs-lookup"><span data-stu-id="164fc-146">Use performance monitoring to identify how much time the production system spends performing database activity.</span></span>

2. <span data-ttu-id="164fc-147">Untersuchen Sie die Vorgänge, die in diesen Zeiträumen von der Datenbank ausgeführt werden.</span><span class="sxs-lookup"><span data-stu-id="164fc-147">Examine the work performed by the database during these periods.</span></span>

3. <span data-ttu-id="164fc-148">Wenn Sie den Verdacht hegen, dass bestimmte Vorgänge übermäßige Datenbankaktivitäten nach sich ziehen, führen Sie in einer kontrollierten Umgebung einen Auslastungstest durch.</span><span class="sxs-lookup"><span data-stu-id="164fc-148">If you suspect that particular operations might cause too much database activity, perform load testing in a controlled environment.</span></span> <span data-ttu-id="164fc-149">Jeder Test sollte eine Kombination der verdächtigen Vorgänge mit einer variablen Benutzerauslastung ausführen.</span><span class="sxs-lookup"><span data-stu-id="164fc-149">Each test should run a mixture of the suspect operations with a variable user load.</span></span> <span data-ttu-id="164fc-150">Untersuchen Sie die Telemetriedaten der Auslastungstests, um herauszufinden, wie die Datenbank verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="164fc-150">Examine the telemetry from the load tests to observe how the database is used.</span></span>

4. <span data-ttu-id="164fc-151">Wenn die Datenbankaktivität eine erhebliche Verarbeitungslast, aber wenig Datenverkehr zeigt, untersuchen Sie den Quellcode, um festzustellen, ob die Verarbeitung besser an anderer Stelle erfolgen sollte.</span><span class="sxs-lookup"><span data-stu-id="164fc-151">If the database activity reveals significant processing but little data traffic, review the source code to determine whether the processing can better be performed elsewhere.</span></span>

<span data-ttu-id="164fc-152">Wenn der Umfang der Datenbankaktivität gering ist oder die Antwortzeiten relativ kurz sind, ist eine ausgelastete Datenbank wahrscheinlich kein Leistungsproblem.</span><span class="sxs-lookup"><span data-stu-id="164fc-152">If the volume of database activity is low or response times are relatively fast, then a busy database is unlikely to be a performance problem.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="164fc-153">Beispieldiagnose</span><span class="sxs-lookup"><span data-stu-id="164fc-153">Example diagnosis</span></span>

<span data-ttu-id="164fc-154">In den folgenden Abschnitten werden diese Schritte auf die zuvor beschriebene Beispielanwendung angewendet.</span><span class="sxs-lookup"><span data-stu-id="164fc-154">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-the-volume-of-database-activity"></a><span data-ttu-id="164fc-155">Überwachen des Umfangs der Datenbankaktivität</span><span class="sxs-lookup"><span data-stu-id="164fc-155">Monitor the volume of database activity</span></span>

<span data-ttu-id="164fc-156">Das folgende Diagramm zeigt die Ergebnisse eines Auslastungstests, der für die Beispielanwendung mit einer Schrittauslastung von bis zu 50 gleichzeitigen Benutzern ausgeführt wurde.</span><span class="sxs-lookup"><span data-stu-id="164fc-156">The following graph shows the results of running a load test against the sample application, using a step load of up to 50 concurrent users.</span></span> <span data-ttu-id="164fc-157">Der Umfang der Anforderungen erreicht schnell einen Maximalwert und bleibt auf diesem Niveau. Die durchschnittliche Antwortzeit dagegen steigt stetig.</span><span class="sxs-lookup"><span data-stu-id="164fc-157">The volume of requests quickly reaches a limit and stays at that level, while the average response time steadily increases.</span></span> <span data-ttu-id="164fc-158">Beachten Sie, dass für diese beiden Metriken eine logarithmische Skalierung verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="164fc-158">Note that a logarithmic scale is used for those two metrics.</span></span>

![Ergebnisse des Auslastungstests für die Verarbeitung in der Datenbank][ProcessingInDatabaseLoadTest]

<span data-ttu-id="164fc-160">Das nächste Diagramm zeigt CPU-Auslastung und DTUs als Prozentsatz des Dienstkontingents.</span><span class="sxs-lookup"><span data-stu-id="164fc-160">The next graph shows CPU utilization and DTUs as a percentage of service quota.</span></span> <span data-ttu-id="164fc-161">DTUs bieten ein Maß dafür, wie viel Verarbeitung die Datenbank leistet.</span><span class="sxs-lookup"><span data-stu-id="164fc-161">DTUs provides a measure of how much processing the database performs.</span></span> <span data-ttu-id="164fc-162">Das Diagramm zeigt, dass sowohl die CPU- als auch die DTU-Auslastung schnell 100 % erreichten.</span><span class="sxs-lookup"><span data-stu-id="164fc-162">The graph shows that CPU and DTU utilization both quickly reached 100%.</span></span>

![Azure SQL-Datenbankmonitor zeigt die Leistung der Datenbank während der Verarbeitung][ProcessingInDatabaseMonitor]

### <a name="examine-the-work-performed-by-the-database"></a><span data-ttu-id="164fc-164">Untersuchen der von der Datenbank ausgeführten Vorgänge</span><span class="sxs-lookup"><span data-stu-id="164fc-164">Examine the work performed by the database</span></span>

<span data-ttu-id="164fc-165">Bei den von der Datenbank ausgeführten Tasks kann es sich um echte Datenzugriffsvorgänge handeln, nicht um Verarbeitungstasks. Daher müssen Sie die SQL-Anweisungen verstehen, die ausgeführt werden, während die Datenbank beschäftigt ist.</span><span class="sxs-lookup"><span data-stu-id="164fc-165">It could be that the tasks performed by the database are genuine data access operations, rather than processing, so it is important to understand the SQL statements being run while the database is busy.</span></span> <span data-ttu-id="164fc-166">Überwachen Sie das System, um den SQL-Datenverkehr zu erfassen, und korrelieren Sie die SQL-Vorgänge mit Anwendungsanforderungen.</span><span class="sxs-lookup"><span data-stu-id="164fc-166">Monitor the system to capture the SQL traffic and correlate the SQL operations with application requests.</span></span>

<span data-ttu-id="164fc-167">Wenn es sich bei den Datenbankvorgängen um reine Zugriffsvorgänge handelt, ohne große Verarbeitungslast, kann das Problem durch [irrelevante Abrufe][ExtraneousFetching] verursacht werden.</span><span class="sxs-lookup"><span data-stu-id="164fc-167">If the database operations are purely data access operations, without a lot of processing, then the problem might be [Extraneous Fetching][ExtraneousFetching].</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="164fc-168">Implementieren der Lösung und Überprüfen des Ergebnisses</span><span class="sxs-lookup"><span data-stu-id="164fc-168">Implement the solution and verify the result</span></span>

<span data-ttu-id="164fc-169">Das folgende Diagramm zeigt einen Auslastungstest mit dem aktualisierten Code.</span><span class="sxs-lookup"><span data-stu-id="164fc-169">The following graph shows a load test using the updated code.</span></span> <span data-ttu-id="164fc-170">Der Durchsatz ist erheblich höher: über 400 Anforderungen pro Sekunde im Vergleich zu 12 in der vorherigen Version.</span><span class="sxs-lookup"><span data-stu-id="164fc-170">Throughput is significantly higher, over 400 requests per second versus 12 earlier.</span></span> <span data-ttu-id="164fc-171">Die durchschnittliche Antwortzeit ist ebenfalls wesentlich niedriger: knapp über 0,1 Sekunden im Vergleich zu 4 Sekunden.</span><span class="sxs-lookup"><span data-stu-id="164fc-171">The average response time is also much lower, just above 0.1 seconds compared to over 4 seconds.</span></span>

![Ergebnisse des Auslastungstests für die Verarbeitung in der Datenbank][ProcessingInClientApplicationLoadTest]

<span data-ttu-id="164fc-173">Die CPU- und DTU-Auslastung zeigt, dass es trotz erhöhten Durchsatzes länger gedauert hat, bis das System ausgelastet war.</span><span class="sxs-lookup"><span data-stu-id="164fc-173">CPU and DTU utilization shows that the system took longer to reach saturation, despite the increased throughput.</span></span>

![Azure SQL-Datenbankmonitor zeigt die Leistung der Datenbank während der Verarbeitung in der Clientanwendung][ProcessingInClientApplicationMonitor]

## <a name="related-resources"></a><span data-ttu-id="164fc-175">Zugehörige Ressourcen</span><span class="sxs-lookup"><span data-stu-id="164fc-175">Related resources</span></span> 

- <span data-ttu-id="164fc-176">[Antimuster „Irrelevante Abrufe“][ExtraneousFetching]</span><span class="sxs-lookup"><span data-stu-id="164fc-176">[Extraneous Fetching antipattern][ExtraneousFetching]</span></span>


[dtu]: /sql-database/sql-database-what-is-a-dtu
[ExtraneousFetching]: ../extraneous-fetching/index.md
[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/BusyDatabase

[ProcessingInDatabaseLoadTest]: ./_images/ProcessingInDatabaseLoadTest.jpg
[ProcessingInClientApplicationLoadTest]: ./_images/ProcessingInClientApplicationLoadTest.jpg
[ProcessingInDatabaseMonitor]: ./_images/ProcessingInDatabaseMonitor.jpg
[ProcessingInClientApplicationMonitor]: ./_images/ProcessingInClientApplicationMonitor.jpg
