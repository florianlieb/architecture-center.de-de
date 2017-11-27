---
title: Antimuster der monolithischen Persistenz
description: "Das Speichern sämtlicher Daten einer Anwendung in einem einzigen Datenspeicher kann die Leistung beeinträchtigen."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 7f04b9f0805c281068b6b2edaf040683773e6f6e
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="monolithic-persistence-antipattern"></a><span data-ttu-id="09564-103">Antimuster der monolithischen Persistenz</span><span class="sxs-lookup"><span data-stu-id="09564-103">Monolithic Persistence antipattern</span></span>

<span data-ttu-id="09564-104">Das Speichern sämtlicher Daten einer Anwendung in einem einzigen Datenspeicher kann die Leistung beeinträchtigen, weil es entweder zu Ressourcenkonflikten führt oder weil der Datenspeicher sich für einige Daten nicht gut eignet.</span><span class="sxs-lookup"><span data-stu-id="09564-104">Putting all of an application's data into a single data store can hurt performance, either because it leads to resource contention, or because the data store is not a good fit for some of the data.</span></span>

## <a name="problem-description"></a><span data-ttu-id="09564-105">Problembeschreibung</span><span class="sxs-lookup"><span data-stu-id="09564-105">Problem description</span></span>

<span data-ttu-id="09564-106">Früher verwendeten Anwendungen häufig einen einzigen Datenspeicher, unabhängig von den unterschiedlichen Datentypen, die eine Anwendung speichern muss.</span><span class="sxs-lookup"><span data-stu-id="09564-106">Historically, applications have often used a single data store, regardless of the different types of data that the application might need to store.</span></span> <span data-ttu-id="09564-107">Der Grund dafür war üblicherweise, dass der Anwendungsentwurf vereinfacht werden sollte. Eine andere Ursache waren häufig die technischen Fähigkeiten des Entwicklungsteams.</span><span class="sxs-lookup"><span data-stu-id="09564-107">Usually this was done to simplify the application design, or else to match the existing skill set of the development team.</span></span> 

<span data-ttu-id="09564-108">Moderne cloudbasierte Systeme weisen meist zusätzliche funktionelle und nicht funktionelle Anforderungen auf und müssen viele heterogene Arten von Daten speichern – z.B. Dokumente, Bilder, zwischengespeicherte Daten, Nachrichten in Warteschlangen, Anwendungsprotokolle und Telemetriedaten.</span><span class="sxs-lookup"><span data-stu-id="09564-108">Modern cloud-based systems often have additional functional and nonfunctional requirements, and need to store many heterogenous types of data, such as documents, images, cached data, queued messages, application logs, and telemetry.</span></span> <span data-ttu-id="09564-109">Das Verfolgen des herkömmlichen Ansatzes und Speichern all dieser Informationen im gleichen Datenspeicher kann hauptsächlich aus zwei Gründen die Leistung beeinträchtigen:</span><span class="sxs-lookup"><span data-stu-id="09564-109">Following the traditional approach and putting all of this information into the same data store can hurt performance, for two main reasons:</span></span>

- <span data-ttu-id="09564-110">Das Speichern und Abrufen großer Mengen nicht in Zusammenhang stehender Daten im gleichen Datenspeicher kann zu Konflikten führen, was wiederum lange Antwortzeiten und Verbindungsfehler zur Folge hat.</span><span class="sxs-lookup"><span data-stu-id="09564-110">Storing and retrieving large amounts of unrelated data in the same data store can cause contention, which in turn leads to slow response times and connection failures.</span></span>
- <span data-ttu-id="09564-111">Unabhängig davon, welcher Datenspeicher ausgewählt wird, er wird sich vermutlich nicht ideal für all die verschiedenen Arten von Daten eignen oder nicht für die Vorgänge optimiert sein, die von der Anwendung ausgeführt werden.</span><span class="sxs-lookup"><span data-stu-id="09564-111">Whichever data store is chosen, it might not be the best fit for all of the different types of data, or it might not be optimized for the operations that the application performs.</span></span> 

<span data-ttu-id="09564-112">Das folgende Beispiel zeigt einen ASP.NET-Web-API-Controller, der einen neuen Datensatz zu einer Datenbank hinzufügt und das Ergebnis in einem Protokoll aufzeichnet.</span><span class="sxs-lookup"><span data-stu-id="09564-112">The following example shows an ASP.NET Web API controller that adds a new record to a database and also records the result to a log.</span></span> <span data-ttu-id="09564-113">Das Protokoll wird in der gleichen Datenbank gespeichert wie die Geschäftsdaten.</span><span class="sxs-lookup"><span data-stu-id="09564-113">The log is held in the same database as the business data.</span></span> <span data-ttu-id="09564-114">Das vollständige Codebeispiel finden Sie [hier][sample-app].</span><span class="sxs-lookup"><span data-stu-id="09564-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public class MonoController : ApiController
{
    private static readonly string ProductionDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        await DataAccess.LogAsync(ProductionDb, LogTableName);
        return Ok();
    }
}
```

<span data-ttu-id="09564-115">Die Geschwindigkeit, mit der Protokollzeilen generiert werden, wird sich wahrscheinlich auf die Leistung der Geschäftsvorgänge auswirken.</span><span class="sxs-lookup"><span data-stu-id="09564-115">The rate at which log records are generated will probably affect the performance of the business operations.</span></span> <span data-ttu-id="09564-116">Und wenn eine andere Komponente, z.B. ein Anwendungsprozessmonitor, die Protokolldaten regelmäßig liest und verarbeitet, kann dies die Geschäftsvorgänge ebenfalls beeinträchtigen.</span><span class="sxs-lookup"><span data-stu-id="09564-116">And if another component, such as an application process monitor, regularly reads and processes the log data, that can also affect the business operations.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="09564-117">Beheben des Problems</span><span class="sxs-lookup"><span data-stu-id="09564-117">How to fix the problem</span></span>

<span data-ttu-id="09564-118">Trennen Sie die Daten je nach Verwendung.</span><span class="sxs-lookup"><span data-stu-id="09564-118">Separate data according to its use.</span></span> <span data-ttu-id="09564-119">Wählen Sie für jedes Dataset den Datenspeicher aus, der sich am besten für den Verwendungszweck des jeweiligen Datasets eignet.</span><span class="sxs-lookup"><span data-stu-id="09564-119">For each data set, select a data store that best matches how that data set will be used.</span></span> <span data-ttu-id="09564-120">Im vorherigen Beispiel sollte die Anwendung die Protokollierung in einem anderen Speicher ausführen als in der Datenbank, die die Geschäftsdaten enthält:</span><span class="sxs-lookup"><span data-stu-id="09564-120">In the previous example, the application should be logging to a separate store from the database that holds business data:</span></span> 

```csharp
public class PolyController : ApiController
{
    private static readonly string ProductionDb = ...;
    private static readonly string LogDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        // Log to a different data store.
        await DataAccess.LogAsync(LogDb, LogTableName);
        return Ok();
    }
}
```

## <a name="considerations"></a><span data-ttu-id="09564-121">Überlegungen</span><span class="sxs-lookup"><span data-stu-id="09564-121">Considerations</span></span>

- <span data-ttu-id="09564-122">Trennen Sie die Daten nach der Art der Verwendung und des Zugriffs.</span><span class="sxs-lookup"><span data-stu-id="09564-122">Separate data by the way it is used and how it is accessed.</span></span> <span data-ttu-id="09564-123">Speichern Sie beispielsweise Protokollinformationen und Geschäftsdaten nicht im gleichen Datenspeicher.</span><span class="sxs-lookup"><span data-stu-id="09564-123">For example, don't store log information and business data in the same data store.</span></span> <span data-ttu-id="09564-124">Diese Datentypen weisen sehr unterschiedliche Anforderungen und Zugriffsmuster auf.</span><span class="sxs-lookup"><span data-stu-id="09564-124">These types of data have significantly different requirements and patterns of access.</span></span> <span data-ttu-id="09564-125">Protokolleinträge sind von Natur aus sequenziell, während Geschäftsdaten wahrscheinlich einen zufallsbasierten Zugriff erfordern und häufig relational sind.</span><span class="sxs-lookup"><span data-stu-id="09564-125">Log records are inherently sequential, while business data is more likely to require random access, and is often relational.</span></span>

- <span data-ttu-id="09564-126">Berücksichtigen Sie die Datenzugriffsmuster für jeden Datentyp.</span><span class="sxs-lookup"><span data-stu-id="09564-126">Consider the data access pattern for each type of data.</span></span> <span data-ttu-id="09564-127">Speichern Sie z.B. formatierte Berichte und Dokumente in einer Dokumentdatenbank wie [Cosmos DB][CosmosDB], und verwenden Sie [Azure Redis Cache][Azure-cache] zum Zwischenspeichern temporärer Daten.</span><span class="sxs-lookup"><span data-stu-id="09564-127">For example, store formatted reports and documents in a document database such as [Cosmos DB][CosmosDB], but use [Azure Redis Cache][Azure-cache] to cache temporary data.</span></span>

- <span data-ttu-id="09564-128">Wenn Sie diese Leitlinien befolgen, die Datenbank aber dennoch ihre Grenze erreicht, müssen Sie die Datenbank möglicherweise zentral hochskalieren.</span><span class="sxs-lookup"><span data-stu-id="09564-128">If you follow this guidance but still reach the limits of the database, you may need to scale up the database.</span></span> <span data-ttu-id="09564-129">Erwägen Sie auch, die Datenbank horizontal zu skalieren und die Last auf verschiedenen Datenbankserver zu partitionieren.</span><span class="sxs-lookup"><span data-stu-id="09564-129">Also consider scaling horizontally and partitioning the load across database servers.</span></span> <span data-ttu-id="09564-130">Für die Partitionierung muss jedoch möglicherweise der Anwendungsentwurf geändert werden.</span><span class="sxs-lookup"><span data-stu-id="09564-130">However, partitioning may require redesigning the application.</span></span> <span data-ttu-id="09564-131">Weitere Informationen finden Sie unter [Data partitioning][DataPartitioningGuidance] (Datenpartitionierung).</span><span class="sxs-lookup"><span data-stu-id="09564-131">For more information, see [Data partitioning][DataPartitioningGuidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="09564-132">Erkennen des Problems</span><span class="sxs-lookup"><span data-stu-id="09564-132">How to detect the problem</span></span>

<span data-ttu-id="09564-133">Das System wird sich wahrscheinlich erheblich verlangsamen und letztendlich ausfallen, da nicht genügend Ressourcen wie z.B. Datenbankverbindungen vorhanden sind.</span><span class="sxs-lookup"><span data-stu-id="09564-133">The system will likely slow down dramatically and eventually fail, as the system runs out of resources such as database connections.</span></span>

<span data-ttu-id="09564-134">Sie können die folgenden Schritte ausführen, um die Ursache zu ermitteln.</span><span class="sxs-lookup"><span data-stu-id="09564-134">You can perform the following steps to help identify the cause.</span></span>

1. <span data-ttu-id="09564-135">Instrumentieren Sie das System, um die wichtigsten Leistungsstatistiken aufzuzeichnen.</span><span class="sxs-lookup"><span data-stu-id="09564-135">Instrument the system to record the key performance statistics.</span></span> <span data-ttu-id="09564-136">Erfassen Sie Zeitinformationen für jeden Vorgang sowie die Punkte, an denen die Anwendung Daten liest und schreibt.</span><span class="sxs-lookup"><span data-stu-id="09564-136">Capture timing information for each operation, as well as the points where the application reads and writes data.</span></span>
1. <span data-ttu-id="09564-137">Überwachen Sie nach Möglichkeit das ausgeführte System einige Tage lang in einer Produktionsumgebung, um herauszufinden, wie das System in der Praxis verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="09564-137">If possible, monitor the system running for a few days in a production environment to get a real-world view of how the system is used.</span></span> <span data-ttu-id="09564-138">Wenn dies nicht möglich ist, führen Sie skriptbasierte Auslastungstests mit einer realistischen Menge an virtuellen Benutzern durch, die eine typische Reihe von Vorgängen ausführen.</span><span class="sxs-lookup"><span data-stu-id="09564-138">If this is not possible, run scripted load tests with a realistic volume of virtual users performing a typical series of operations.</span></span>
2. <span data-ttu-id="09564-139">Verwenden Sie die Telemetriedaten, um Zeiträume unzureichender Leistung zu identifizieren.</span><span class="sxs-lookup"><span data-stu-id="09564-139">Use the telemetry data to identify periods of poor performance.</span></span>
3. <span data-ttu-id="09564-140">Ermitteln Sie, auf welche Datenspeicher während dieser Zeiträume zugegriffen wurde.</span><span class="sxs-lookup"><span data-stu-id="09564-140">Identify which data stores were accessed during those periods.</span></span>
4. <span data-ttu-id="09564-141">Identifizieren Sie die Datenspeicherressourcen, bei denen möglicherweise Konflikte auftreten.</span><span class="sxs-lookup"><span data-stu-id="09564-141">Identify data storage resources that might be experiencing contention.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="09564-142">Beispieldiagnose</span><span class="sxs-lookup"><span data-stu-id="09564-142">Example diagnosis</span></span>

<span data-ttu-id="09564-143">In den folgenden Abschnitten werden diese Schritte auf die zuvor beschriebene Beispielanwendung angewendet.</span><span class="sxs-lookup"><span data-stu-id="09564-143">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-and-monitor-the-system"></a><span data-ttu-id="09564-144">Instrumentieren und Überwachen des Systems</span><span class="sxs-lookup"><span data-stu-id="09564-144">Instrument and monitor the system</span></span>

<span data-ttu-id="09564-145">Das folgende Diagramm zeigt die Ergebnisse des Auslastungstests der oben beschriebenen Beispielanwendung.</span><span class="sxs-lookup"><span data-stu-id="09564-145">The following graph shows the results of load testing the sample application described earlier.</span></span> <span data-ttu-id="09564-146">Bei diesem Test wurde eine Schrittauslastung von bis zu 1.000 gleichzeitigen Benutzern verwendet.</span><span class="sxs-lookup"><span data-stu-id="09564-146">The test used a step load of up to 1000 concurrent users.</span></span>

![Leistungsergebnisse des Auslastungstests für den SQL-basierten Controller][MonolithicScenarioLoadTest]

<span data-ttu-id="09564-148">Wenn die Auslastung auf 700 Benutzer steigt, steigt auch der Durchsatz.</span><span class="sxs-lookup"><span data-stu-id="09564-148">As the load increases to 700 users, so does the throughput.</span></span> <span data-ttu-id="09564-149">An diesem Punkt sinkt der Durchsatz, und das System wird anscheinend mit maximaler Kapazität ausgeführt.</span><span class="sxs-lookup"><span data-stu-id="09564-149">But at that point, throughput levels off, and the system appears to be running at its maximum capacity.</span></span> <span data-ttu-id="09564-150">Die durchschnittliche Antwortzeit steigt langsam mit der Benutzerauslastung, was zeigt, dass das System mit den Anforderungen nicht mithalten kann.</span><span class="sxs-lookup"><span data-stu-id="09564-150">The average response gradually increases with user load, showing that the system can't keep up with demand.</span></span>

### <a name="identify-periods-of-poor-performance"></a><span data-ttu-id="09564-151">Identifizieren von Zeiträumen unzureichender Leistung</span><span class="sxs-lookup"><span data-stu-id="09564-151">Identify periods of poor performance</span></span>

<span data-ttu-id="09564-152">Wenn Sie das Produktionssystem überwachen, stellen Sie möglicherweise Muster fest.</span><span class="sxs-lookup"><span data-stu-id="09564-152">If you are monitoring the production system, you might notice patterns.</span></span> <span data-ttu-id="09564-153">Möglicherweise sinken Antwortzeiten zu jeder Tageszeit erheblich.</span><span class="sxs-lookup"><span data-stu-id="09564-153">For example, response times might drop off significantly at the same time each day.</span></span> <span data-ttu-id="09564-154">Dies könnte durch eine regelmäßige Workload oder einen geplanten Batchauftrag verursacht werden oder einfach daran liegen, dass sich zu bestimmten Zeiten mehr Benutzer im System befinden.</span><span class="sxs-lookup"><span data-stu-id="09564-154">This could be caused by a regular workload or scheduled batch job, or just because the system has more users at certain times.</span></span> <span data-ttu-id="09564-155">Sie sollten sich auf die Telemetriedaten für diese Ereignisse konzentrieren.</span><span class="sxs-lookup"><span data-stu-id="09564-155">You should focus on the telemetry data for these events.</span></span>

<span data-ttu-id="09564-156">Suchen Sie nach Zusammenhängen zwischen erhöhten Antwortzeiten und vermehrten Datenbankaktivitäten oder E/A-Vorgängen in gemeinsam genutzten Ressourcen.</span><span class="sxs-lookup"><span data-stu-id="09564-156">Look for correlations between increased response times and increased database activity or I/O to shared resources.</span></span> <span data-ttu-id="09564-157">Wenn solche Zusammenhänge zu erkennen sind, kann dies bedeuten, dass die Datenbank einen Engpass darstellt.</span><span class="sxs-lookup"><span data-stu-id="09564-157">If there are correlations, it means the database might be a bottleneck.</span></span>

### <a name="identify-which-data-stores-are-accessed-during-those-periods"></a><span data-ttu-id="09564-158">Ermitteln, auf welche Datenspeicher während dieser Zeiträume zugegriffen wurde</span><span class="sxs-lookup"><span data-stu-id="09564-158">Identify which data stores are accessed during those periods</span></span>

<span data-ttu-id="09564-159">Das nächste Diagramm zeigt die DTU-Nutzung (Database Throughput Units, Datenbankdurchsatzeinheiten) während des Auslastungstests.</span><span class="sxs-lookup"><span data-stu-id="09564-159">The next graph shows the utilization of database throughput units (DTU) during the load test.</span></span> <span data-ttu-id="09564-160">(Eine DTU ist ein Maß der verfügbaren Kapazität und setzt sich aus CPU-Auslastung, Arbeitsspeicherzuweisung und E/A-Rate zusammen.) Die Auslastung der DTUs erreichte schnell 100 %.</span><span class="sxs-lookup"><span data-stu-id="09564-160">(A DTU is a measure of available capacity, and is a combination of CPU utilization, memory allocation, I/O rate.) Utilization of DTUs quickly reached 100%.</span></span> <span data-ttu-id="09564-161">Dies ist ungefähr der Punkt, an dem der Durchsatz im vorherigen Diagramm den Spitzenwert erreichte.</span><span class="sxs-lookup"><span data-stu-id="09564-161">This is roughly the point where throughput peaked in the previous graph.</span></span> <span data-ttu-id="09564-162">Die Datenbankauslastung blieb bis zum Ende des Tests sehr hoch.</span><span class="sxs-lookup"><span data-stu-id="09564-162">Database utilization remained very high until the test finished.</span></span> <span data-ttu-id="09564-163">Gegen Ende ist ein leichtes Abfallen zu beobachten, das durch Drosselung, konkurrierende Datenbankverbindungen oder andere Faktoren entstanden sein kann.</span><span class="sxs-lookup"><span data-stu-id="09564-163">There is a slight drop toward the end, which could be caused by throttling, competition for database connections, or other factors.</span></span>

![Datenbankmonitor im klassischen Azure-Portal mit der Ressourcennutzung der Datenbank][MonolithicDatabaseUtilization]

### <a name="examine-the-telemetry-for-the-data-stores"></a><span data-ttu-id="09564-165">Untersuchen der Telemetrie für die Datenspeicher</span><span class="sxs-lookup"><span data-stu-id="09564-165">Examine the telemetry for the data stores</span></span>

<span data-ttu-id="09564-166">Instrumentieren Sie die Datenspeicher, um die Details der Aktivität auf niedriger Ebene zu erfassen.</span><span class="sxs-lookup"><span data-stu-id="09564-166">Instrument the data stores to capture the low-level details of the activity.</span></span> <span data-ttu-id="09564-167">In der Beispielanwendung zeigten die Datenzugriffsstatistiken eine große Menge an Einfügevorgängen, die sowohl für die Tabelle `PurchaseOrderHeader` als auch für die Tabelle `MonoLog` ausgeführt wurden.</span><span class="sxs-lookup"><span data-stu-id="09564-167">In the sample application, the data access statistics showed a high volume of insert operations performed against both the `PurchaseOrderHeader` table and the `MonoLog` table.</span></span> 

![Datenzugriffsstatistiken für die Beispielanwendung][MonolithicDataAccessStats]

### <a name="identify-resource-contention"></a><span data-ttu-id="09564-169">Identifizieren von Ressourcenkonflikten</span><span class="sxs-lookup"><span data-stu-id="09564-169">Identify resource contention</span></span>

<span data-ttu-id="09564-170">An diesem Punkt können Sie den Quellcode überprüfen, und sich dabei auf die Punkte konzentrieren, an denen die Anwendung auf Ressourcen mit Konflikten zugegriffen hat.</span><span class="sxs-lookup"><span data-stu-id="09564-170">At this point, you can review the source code, focusing on the points where contended resources are accessed by the application.</span></span> <span data-ttu-id="09564-171">Suchen Sie nach Situationen wie etwa den folgenden:</span><span class="sxs-lookup"><span data-stu-id="09564-171">Look for situations such as:</span></span>

- <span data-ttu-id="09564-172">Daten, die logisch getrennt sind, werden in den gleichen Speicher geschrieben.</span><span class="sxs-lookup"><span data-stu-id="09564-172">Data that is logically separate being written to the same store.</span></span> <span data-ttu-id="09564-173">Daten wie Protokolle, Berichte und Nachrichten in Warteschlangen sollten nicht in der gleichen Datenbank gespeichert werden wie Geschäftsdaten.</span><span class="sxs-lookup"><span data-stu-id="09564-173">Data such as logs, reports, and queued messages should not be held in the same database as business information.</span></span>
- <span data-ttu-id="09564-174">Der Datenspeicher und die Art der Daten, die darin gespeichert werden sollen, stimmen nicht überein, beispielsweise werden große Blobs oder XML-Dokumente in einer relationalen Datenbank gespeichert.</span><span class="sxs-lookup"><span data-stu-id="09564-174">A mismatch between the choice of data store and the type of data, such as large blobs or XML documents in a relational database.</span></span>
- <span data-ttu-id="09564-175">Daten mit erheblich unterschiedlichen Nutzungsmustern befinden sich im gleichen Speicher, z.B. werden Daten mit hohen Schreib-, aber geringen Leseanforderungen zusammen mit Daten mit geringen Schreib-, aber hohen Leseanforderungen gespeichert.</span><span class="sxs-lookup"><span data-stu-id="09564-175">Data with significantly different usage patterns that share the same store, such as high-write/low-read data being stored with low-write/high-read data.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="09564-176">Implementieren der Lösung und Überprüfen des Ergebnisses</span><span class="sxs-lookup"><span data-stu-id="09564-176">Implement the solution and verify the result</span></span>

<span data-ttu-id="09564-177">Die Anwendung wurde geändert und schreibt Protokolle jetzt in einen separaten Datenspeicher.</span><span class="sxs-lookup"><span data-stu-id="09564-177">The application was changed to write logs to a separate data store.</span></span> <span data-ttu-id="09564-178">Hier sind die Ergebnisse des Auslastungstests:</span><span class="sxs-lookup"><span data-stu-id="09564-178">Here are the load test results:</span></span>

![Leistungsergebnisse des Auslastungstests mit dem mehrsprachigen Controller][PolyglotScenarioLoadTest]

<span data-ttu-id="09564-180">Das Durchsatzmuster ähnelt dem vorherigen Diagramm, aber der Punkt, an dem die Leistung ihren Spitzenwert erreicht, liegt etwa 500 Anforderungen pro Sekunde höher.</span><span class="sxs-lookup"><span data-stu-id="09564-180">The pattern of throughput is similar to the earlier graph, but the point at which performance peaks is approximately 500 requests per second higher.</span></span> <span data-ttu-id="09564-181">Die durchschnittliche Antwortzeit ist geringfügig geringer.</span><span class="sxs-lookup"><span data-stu-id="09564-181">The average response time is marginally lower.</span></span> <span data-ttu-id="09564-182">Diese Statistiken erzählen jedoch nicht die ganze Geschichte.</span><span class="sxs-lookup"><span data-stu-id="09564-182">However, these statistics don't tell the full story.</span></span> <span data-ttu-id="09564-183">Die Telemetrie für die Geschäftsdatenbank zeigt, dass die DTU-Auslastung ihren Spitzenwert bei etwa 75 % erreicht, nicht bei 100 %.</span><span class="sxs-lookup"><span data-stu-id="09564-183">Telemetry for the business database shows that DTU utilization peaks at around 75%, rather than 100%.</span></span>

![Datenbankmonitor im klassischen Azure-Portal mit der Ressourcennutzung der Datenbank im mehrsprachigen Szenario][PolyglotDatabaseUtilization]

<span data-ttu-id="09564-185">Ebenso erreicht die maximale DTU-Auslastung der Protokolldatenbank nur etwa 70 %.</span><span class="sxs-lookup"><span data-stu-id="09564-185">Similarly, the maximum DTU utilization of the log database only reaches about 70%.</span></span> <span data-ttu-id="09564-186">Die Datenbanken sind nicht mehr der Faktor, der die Leistung des Systems einschränkt.</span><span class="sxs-lookup"><span data-stu-id="09564-186">The databases are no longer the limiting factor in the performance of the system.</span></span>

![Datenbankmonitor im klassischen Azure-Portal mit der Ressourcennutzung der Protokolldatenbank im mehrsprachigen Szenario][LogDatabaseUtilization]


## <a name="related-resources"></a><span data-ttu-id="09564-188">Zugehörige Ressourcen</span><span class="sxs-lookup"><span data-stu-id="09564-188">Related resources</span></span>

- <span data-ttu-id="09564-189">[Choose the right data store][data-store-overview] (Auswählen des richtigen Datenspeichers)</span><span class="sxs-lookup"><span data-stu-id="09564-189">[Choose the right data store][data-store-overview]</span></span>
- <span data-ttu-id="09564-190">[Criteria for choosing a data store][data-store-comparison] (Kriterien für die Auswahl eines Datenspeichers)</span><span class="sxs-lookup"><span data-stu-id="09564-190">[Criteria for choosing a data store][data-store-comparison]</span></span>
- <span data-ttu-id="09564-191">[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence][Data-Access-Guide] (Datenzugriff für hoch skalierbare Lösungen: Verwenden von SQL, NoSQL und mehrsprachiger Persistenz)</span><span class="sxs-lookup"><span data-stu-id="09564-191">[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence][Data-Access-Guide]</span></span>
- <span data-ttu-id="09564-192">[Data partitioning][DataPartitioningGuidance] (Datenpartitionierung)</span><span class="sxs-lookup"><span data-stu-id="09564-192">[Data partitioning][DataPartitioningGuidance]</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/MonolithicPersistence
[CosmosDB]: http://azure.microsoft.com/services/cosmos-db/
[Azure-cache]: /azure/redis-cache/
[Data-Access-Guide]: https://msdn.microsoft.com/library/dn271399.aspx
[DataPartitioningGuidance]: ../../best-practices/data-partitioning.md
[data-store-overview]: ../../guide/technology-choices/data-store-overview.md
[data-store-comparison]: ../../guide/technology-choices/data-store-comparison.md

[MonolithicScenarioLoadTest]: _images/MonolithicScenarioLoadTest.jpg
[MonolithicDatabaseUtilization]: _images/MonolithicDatabaseUtilization.jpg
[MonolithicDataAccessStats]: _images/MonolithicDataAccessStats.jpg
[PolyglotScenarioLoadTest]: _images/PolyglotScenarioLoadTest.jpg
[PolyglotDatabaseUtilization]: _images/PolyglotDatabaseUtilization.jpg
[LogDatabaseUtilization]: _images/LogDatabaseUtilization.jpg
