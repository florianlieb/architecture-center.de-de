---
title: "Kriterien für die Auswahl eines Datenspeichers"
description: "Übersicht über Azure-Computeoptionen"
author: MikeWasson
ms.openlocfilehash: 7fb75cd334438c5b985fa04ad8afe3236f2391f8
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="criteria-for-choosing-a-data-store"></a><span data-ttu-id="fefbd-103">Kriterien für die Auswahl eines Datenspeichers</span><span class="sxs-lookup"><span data-stu-id="fefbd-103">Criteria for choosing a data store</span></span>

<span data-ttu-id="fefbd-104">Azure unterstützt zahlreiche Typen von Datenspeicherlösungen mit jeweils unterschiedlichen Features und Funktionen.</span><span class="sxs-lookup"><span data-stu-id="fefbd-104">Azure supports many types of data storage solutions, each providing different features and capabilities.</span></span> <span data-ttu-id="fefbd-105">In diesem Artikel werden die Vergleichskriterien beschrieben, die Sie bei der Bewertung eines Datenspeichers anwenden sollten.</span><span class="sxs-lookup"><span data-stu-id="fefbd-105">This article describes the comparison criteria you should use when evaluating a data store.</span></span> <span data-ttu-id="fefbd-106">Sie sollen Ihnen dabei helfen, zu bestimmen, welche Datenspeichertypen die Anforderungen Ihrer Lösung erfüllen können.</span><span class="sxs-lookup"><span data-stu-id="fefbd-106">The goal is to help you determine which data storage types can meet your solution's requirements.</span></span>

## <a name="general-considerations"></a><span data-ttu-id="fefbd-107">Allgemeine Überlegungen</span><span class="sxs-lookup"><span data-stu-id="fefbd-107">General Considerations</span></span>

<span data-ttu-id="fefbd-108">Sammeln Sie für den Vergleich zunächst möglichst viele der folgenden Informationen zu den Anforderungen für Ihre Daten.</span><span class="sxs-lookup"><span data-stu-id="fefbd-108">To start your comparison, gather as much of the following information as you can about your data needs.</span></span> <span data-ttu-id="fefbd-109">Anhand dieser Informationen können Sie bestimmen, welche Datenspeichertypen Ihren Anforderungen gerecht werden.</span><span class="sxs-lookup"><span data-stu-id="fefbd-109">This information will help you to determine which data storage types will meet your needs.</span></span>

### <a name="functional-requirements"></a><span data-ttu-id="fefbd-110">Funktionsanforderungen</span><span class="sxs-lookup"><span data-stu-id="fefbd-110">Functional requirements</span></span>

- <span data-ttu-id="fefbd-111">**Datenformat:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-111">**Data format**.</span></span> <span data-ttu-id="fefbd-112">Welche Arten von Daten möchten Sie speichern?</span><span class="sxs-lookup"><span data-stu-id="fefbd-112">What type of data are you intending to store?</span></span> <span data-ttu-id="fefbd-113">Zu den allgemeinen Typen gehören Transaktionsdaten, JSON-Objekte, Telemetriedaten, Suchindizes oder Flatfiles.</span><span class="sxs-lookup"><span data-stu-id="fefbd-113">Common types include transactional data, JSON objects, telemetry, search indexes, or flat files.</span></span>
- <span data-ttu-id="fefbd-114">**Datengröße:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-114">**Data size**.</span></span> <span data-ttu-id="fefbd-115">Wie umfangreich sind die Entitäten, die Sie speichern möchten?</span><span class="sxs-lookup"><span data-stu-id="fefbd-115">How large are the entities you need to store?</span></span> <span data-ttu-id="fefbd-116">Müssen diese Entitäten als einzelnes Dokument beibehalten werden, oder können sie auf mehrere Dokumente, Tabellen, Sammlungen usw. aufgeteilt werden?</span><span class="sxs-lookup"><span data-stu-id="fefbd-116">Will these entities need to be maintained as a single document, or can they be split across multiple documents, tables, collections, and so forth?</span></span>
- <span data-ttu-id="fefbd-117">**Skalierung und Struktur:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-117">**Scale and structure**.</span></span> <span data-ttu-id="fefbd-118">Welche Gesamtmenge an Speicherkapazität benötigen Sie?</span><span class="sxs-lookup"><span data-stu-id="fefbd-118">What is the overall amount of storage capacity you need?</span></span> <span data-ttu-id="fefbd-119">Gehen Sie davon aus, dass Sie die Daten partitionieren?</span><span class="sxs-lookup"><span data-stu-id="fefbd-119">Do you anticipate partitioning your data?</span></span> 
- <span data-ttu-id="fefbd-120">**Datenbeziehungen:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-120">**Data relationships**.</span></span> <span data-ttu-id="fefbd-121">Müssen Ihre Daten 1:n- oder m:n-Beziehungen unterstützen?</span><span class="sxs-lookup"><span data-stu-id="fefbd-121">Will your data need to support one-to-many or many-to-many relationships?</span></span> <span data-ttu-id="fefbd-122">Sind die Beziehungen selbst ein wichtiger Teil der Daten?</span><span class="sxs-lookup"><span data-stu-id="fefbd-122">Are relationships themselves an important part of the data?</span></span> <span data-ttu-id="fefbd-123">Müssen Sie Daten innerhalb des gleichen DataSets oder aus externen DataSets verknüpfen oder auf andere Weise kombinieren?</span><span class="sxs-lookup"><span data-stu-id="fefbd-123">Will you need to join or otherwise combine data from within the same dataset, or from external datasets?</span></span> 
- <span data-ttu-id="fefbd-124">**Konsistenzmodell:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-124">**Consistency model**.</span></span> <span data-ttu-id="fefbd-125">Wie wichtig ist es, dass in einem Knoten durchgeführte Aktualisierungen in anderen Knoten angezeigt werden, damit weitere Änderungen vorgenommen werden können?</span><span class="sxs-lookup"><span data-stu-id="fefbd-125">How important is it for updates made in one node to appear in other nodes, before further changes can be made?</span></span> <span data-ttu-id="fefbd-126">Können Sie letztlich Konsistenz akzeptieren?</span><span class="sxs-lookup"><span data-stu-id="fefbd-126">Can you accept eventual consistency?</span></span> <span data-ttu-id="fefbd-127">Benötigen Sie ACID-Garantien für Transaktionen?</span><span class="sxs-lookup"><span data-stu-id="fefbd-127">Do you need ACID guarantees for transactions?</span></span>
- <span data-ttu-id="fefbd-128">**Schemaflexibilität:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-128">**Schema flexibility**.</span></span> <span data-ttu-id="fefbd-129">Welche Art von Schemas wenden Sie auf Ihre Daten an?</span><span class="sxs-lookup"><span data-stu-id="fefbd-129">What kind of schemas will you apply to your data?</span></span> <span data-ttu-id="fefbd-130">Verwenden Sie ein festes Schema, ein Schema bei Schreibvorgängen oder ein Schema bei Lesevorgängen?</span><span class="sxs-lookup"><span data-stu-id="fefbd-130">Will you use a fixed schema, a schema-on-write approach, or a schema-on-read approach?</span></span>
- <span data-ttu-id="fefbd-131">**Parallelität:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-131">**Concurrency**.</span></span> <span data-ttu-id="fefbd-132">Welche Art von Parallelitätsmechanismus möchten Sie beim Aktualisieren und Synchronisieren von Daten verwenden?</span><span class="sxs-lookup"><span data-stu-id="fefbd-132">What kind of concurrency mechanism do you want to use when updating and synchronizing data?</span></span> <span data-ttu-id="fefbd-133">Werden in der Anwendung viele Aktualisierungen durchgeführt, die potenziell zu Konflikten führen?</span><span class="sxs-lookup"><span data-stu-id="fefbd-133">Will the application perform many updates that could potentially conflict.</span></span> <span data-ttu-id="fefbd-134">Wenn dies der Fall ist, benötigen Sie möglicherweise die Datensatzsperrung und Steuerung für pessimistische Parallelität.</span><span class="sxs-lookup"><span data-stu-id="fefbd-134">If so, you may requiring record locking and pessimistic concurrency control.</span></span> <span data-ttu-id="fefbd-135">Können Sie alternativ die Steuerung für optimistische Parallelität unterstützen?</span><span class="sxs-lookup"><span data-stu-id="fefbd-135">Alternatively, can you support optimistic concurrency controls?</span></span> <span data-ttu-id="fefbd-136">Wenn ja, ist eine einfache zeitstempelbasierte Parallelitätssteuerung ausreichend. Oder benötigen Sie die erweiterte Funktion der Parallelitätssteuerung mit mehreren Versionen?</span><span class="sxs-lookup"><span data-stu-id="fefbd-136">If so, is simple timestamp-based concurrency control enough, or do you need the added functionality of multi-version concurrency control?</span></span>
- <span data-ttu-id="fefbd-137">**Datenverschiebung:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-137">**Data movement**.</span></span> <span data-ttu-id="fefbd-138">Müssen in Ihrer Lösung ETL-Aufgaben durchgeführt werden, um Daten in andere Speicher oder Data Warehouses zu verschieben?</span><span class="sxs-lookup"><span data-stu-id="fefbd-138">Will your solution need to perform ETL tasks to move data to other stores or data warehouses?</span></span>
- <span data-ttu-id="fefbd-139">**Datenlebenszyklus:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-139">**Data lifecycle**.</span></span> <span data-ttu-id="fefbd-140">Werden die Daten einmal geschrieben und häufig gelesen?</span><span class="sxs-lookup"><span data-stu-id="fefbd-140">Is the data write-once, read-many?</span></span> <span data-ttu-id="fefbd-141">Können sie in Cool oder Cold Storage verschoben werden?</span><span class="sxs-lookup"><span data-stu-id="fefbd-141">Can it be moved into cool or cold storage?</span></span>
- <span data-ttu-id="fefbd-142">**Andere unterstützte Features:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-142">**Other supported features**.</span></span> <span data-ttu-id="fefbd-143">Benötigen Sie andere spezifische Features, z.B. Schemaüberprüfung, Aggregation, Indizierung, Volltextsuche, MapReduce oder andere Abfragefunktionen?</span><span class="sxs-lookup"><span data-stu-id="fefbd-143">Do you need any other specific features, such as schema validation, aggregation, indexing, full-text search, MapReduce, or other query capabilities?</span></span>

### <a name="non-functional-requirements"></a><span data-ttu-id="fefbd-144">Nicht funktionsbezogene Anforderungen</span><span class="sxs-lookup"><span data-stu-id="fefbd-144">Non-functional requirements</span></span>

- <span data-ttu-id="fefbd-145">**Leistung und Skalierbarkeit:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-145">**Performance and scalability**.</span></span> <span data-ttu-id="fefbd-146">Wie lauten Ihre Anforderungen an die Datenleistung?</span><span class="sxs-lookup"><span data-stu-id="fefbd-146">What are your data performance requirements?</span></span> <span data-ttu-id="fefbd-147">Haben Sie spezielle Anforderungen an Datenerfassungs- und Datenverarbeitungsraten?</span><span class="sxs-lookup"><span data-stu-id="fefbd-147">Do you have specific requirements for data ingestion rates and data processing rates?</span></span> <span data-ttu-id="fefbd-148">Welche zulässigen Antwortzeiten für die Abfrage und Aggregation der Daten nach der Erfassung werden benötigt?</span><span class="sxs-lookup"><span data-stu-id="fefbd-148">What are the acceptable response times for querying and aggregation of data once ingested?</span></span> <span data-ttu-id="fefbd-149">Auf welche Größe muss der Datenspeicher zentral hochskaliert werden können?</span><span class="sxs-lookup"><span data-stu-id="fefbd-149">How large will you need the data store to scale up?</span></span> <span data-ttu-id="fefbd-150">Ist Ihre Workload eher leseintensiv oder schreibintensiv?</span><span class="sxs-lookup"><span data-stu-id="fefbd-150">Is your workload more read-heavy or write-heavy?</span></span>
- <span data-ttu-id="fefbd-151">**Zuverlässigkeit:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-151">**Reliability**.</span></span> <span data-ttu-id="fefbd-152">Welche Gesamt-SLA muss unterstützt werden?</span><span class="sxs-lookup"><span data-stu-id="fefbd-152">What overall SLA do you need to support?</span></span> <span data-ttu-id="fefbd-153">Welche Fehlertoleranzebene müssen Sie für Datenconsumer bereitstellen?</span><span class="sxs-lookup"><span data-stu-id="fefbd-153">What level of fault-tolerance do you need to provide for data consumers?</span></span> <span data-ttu-id="fefbd-154">Welche Art von Sicherungs-und Wiederherstellungsfunktionen benötigen Sie?</span><span class="sxs-lookup"><span data-stu-id="fefbd-154">What kind of backup and restore capabilities do you need?</span></span> 
- <span data-ttu-id="fefbd-155">**Replikation:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-155">**Replication**.</span></span> <span data-ttu-id="fefbd-156">Müssen Ihre Daten auf mehrere Replikate oder Regionen verteilt werden?</span><span class="sxs-lookup"><span data-stu-id="fefbd-156">Will your data need to be distributed among multiple replicas or regions?</span></span> <span data-ttu-id="fefbd-157">Welche Art von Datenreplikationsfunktionen benötigen Sie?</span><span class="sxs-lookup"><span data-stu-id="fefbd-157">What kind of data replication capabilities do you require?</span></span> 
- <span data-ttu-id="fefbd-158">**Einschränkungen:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-158">**Limits**.</span></span> <span data-ttu-id="fefbd-159">Entsprechen die Einschränkungen eines bestimmten Datenspeichers Ihren Anforderungen an die Skalierung, die Anzahl der Verbindungen und den Durchsatz?</span><span class="sxs-lookup"><span data-stu-id="fefbd-159">Will the limits of a particular data store support your requirements for scale, number of connections, and throughput?</span></span> 

### <a name="management-and-cost"></a><span data-ttu-id="fefbd-160">Verwaltung und Kosten</span><span class="sxs-lookup"><span data-stu-id="fefbd-160">Management and cost</span></span>

- <span data-ttu-id="fefbd-161">**Verwalteter Dienst:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-161">**Managed service**.</span></span> <span data-ttu-id="fefbd-162">Verwenden Sie nach Möglichkeit einen verwalteten Datendienst, es sei denn, Sie benötigen bestimmte Funktionen, die nur in einem IaaS-gehosteten Datenspeicher enthalten sind.</span><span class="sxs-lookup"><span data-stu-id="fefbd-162">When possible, use a managed data service, unless you require specific capabilities that can only be found in an IaaS-hosted data store.</span></span>
- <span data-ttu-id="fefbd-163">**Regionale Verfügbarkeit:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-163">**Region availability**.</span></span> <span data-ttu-id="fefbd-164">Ist der Dienst im Fall von verwalteten Diensten in allen Azure-Regionen verfügbar?</span><span class="sxs-lookup"><span data-stu-id="fefbd-164">For managed services, is the service available in all Azure regions?</span></span> <span data-ttu-id="fefbd-165">Muss Ihre Lösung in bestimmten Azure-Regionen gehostet werden?</span><span class="sxs-lookup"><span data-stu-id="fefbd-165">Does your solution need to be hosted in certain Azure regions?</span></span>
- <span data-ttu-id="fefbd-166">**Übertragbarkeit:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-166">**Portability**.</span></span> <span data-ttu-id="fefbd-167">Müssen Ihre Daten zu lokalen oder externen Rechenzentren oder zu anderen Cloudhostingumgebungen migriert werden?</span><span class="sxs-lookup"><span data-stu-id="fefbd-167">Will your data need to migrated to on-premises, external datacenters, or other cloud hosting environments?</span></span>
- <span data-ttu-id="fefbd-168">**Lizenzierung:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-168">**Licensing**.</span></span> <span data-ttu-id="fefbd-169">Bevorzugen Sie einen proprietären gegenüber einem OSS-Lizenztyp?</span><span class="sxs-lookup"><span data-stu-id="fefbd-169">Do you have a preference of a proprietary versus OSS license type?</span></span> <span data-ttu-id="fefbd-170">Gibt es andere externe Einschränkungen im Hinblick auf den zu verwendenden Lizenztyp?</span><span class="sxs-lookup"><span data-stu-id="fefbd-170">Are there any other external restrictions on what type of license you can use?</span></span>
- <span data-ttu-id="fefbd-171">**Gesamtkosten:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-171">**Overall cost**.</span></span> <span data-ttu-id="fefbd-172">Wie hoch sind die Gesamtkosten für die Verwendung des Diensts in Ihrer Lösung?</span><span class="sxs-lookup"><span data-stu-id="fefbd-172">What is the overall cost of using the service within your solution?</span></span> <span data-ttu-id="fefbd-173">Wie viele Instanzen müssen ausgeführt werden, um Ihren Anforderungen an die Betriebszeit und den Durchsatz gerecht zu werden?</span><span class="sxs-lookup"><span data-stu-id="fefbd-173">How many instances will need to run, to support your uptime and throughput requirements?</span></span> <span data-ttu-id="fefbd-174">Berücksichtigen Sie bei dieser Berechnung auch die Betriebskosten.</span><span class="sxs-lookup"><span data-stu-id="fefbd-174">Consider operations costs in this calculation.</span></span> <span data-ttu-id="fefbd-175">Ein Grund für verwaltete Dienste sind die reduzierten Betriebskosten.</span><span class="sxs-lookup"><span data-stu-id="fefbd-175">One reason to prefer managed services is the reduced operational cost.</span></span>
- <span data-ttu-id="fefbd-176">**Kosteneffizienz:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-176">**Cost effectiveness**.</span></span> <span data-ttu-id="fefbd-177">Können Sie Ihre Daten partitionieren, um sie kosteneffektiver zu speichern?</span><span class="sxs-lookup"><span data-stu-id="fefbd-177">Can you partition your data, to store it more cost effectively?</span></span> <span data-ttu-id="fefbd-178">Können beispielsweise umfangreiche Objekte aus einer kostenintensiven relationalen Datenbank in einen Objektspeicher verschoben werden?</span><span class="sxs-lookup"><span data-stu-id="fefbd-178">For example, can you move large objects out of an expensive relational database into an object store?</span></span>

### <a name="security"></a><span data-ttu-id="fefbd-179">Sicherheit</span><span class="sxs-lookup"><span data-stu-id="fefbd-179">Security</span></span>

- <span data-ttu-id="fefbd-180">**Sicherheit**.</span><span class="sxs-lookup"><span data-stu-id="fefbd-180">**Security**.</span></span> <span data-ttu-id="fefbd-181">Welche Art von Verschlüsselung benötigen Sie?</span><span class="sxs-lookup"><span data-stu-id="fefbd-181">What type of encryption do you require?</span></span> <span data-ttu-id="fefbd-182">Benötigen Sie die Verschlüsselung ruhender Daten?</span><span class="sxs-lookup"><span data-stu-id="fefbd-182">Do you need encryption at rest?</span></span> <span data-ttu-id="fefbd-183">Welche Authentifizierungsmethode möchten Sie zur Herstellung einer Verbindung mit den Daten verwenden?</span><span class="sxs-lookup"><span data-stu-id="fefbd-183">What authentication mechanism do you want to use to connect to your data?</span></span>
- <span data-ttu-id="fefbd-184">**Überwachung:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-184">**Auditing**.</span></span> <span data-ttu-id="fefbd-185">Welche Art von Überwachungsprotokoll müssen Sie generieren?</span><span class="sxs-lookup"><span data-stu-id="fefbd-185">What kind of audit log do you need to generate?</span></span>
- <span data-ttu-id="fefbd-186">**Netzwerkanforderungen:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-186">**Networking requirements**.</span></span> <span data-ttu-id="fefbd-187">Muss der Zugriff auf Ihre Daten über andere Netzwerkressourcen beschränkt oder auf andere Weise verwaltet werden?</span><span class="sxs-lookup"><span data-stu-id="fefbd-187">Do you need to restrict or otherwise manage access to your data from other network resources?</span></span> <span data-ttu-id="fefbd-188">Darf nur innerhalb der Azure-Umgebung auf die Daten zugegriffen werden?</span><span class="sxs-lookup"><span data-stu-id="fefbd-188">Does data need to be accessible only from inside the Azure environment?</span></span> <span data-ttu-id="fefbd-189">Müssen die Daten über bestimmte IP-Adressen oder Subnetze zugänglich sein?</span><span class="sxs-lookup"><span data-stu-id="fefbd-189">Does the data need to be accessible from specific IP addresses or subnets?</span></span> <span data-ttu-id="fefbd-190">Müssen sie über Anwendungen oder Dienste zugänglich sein, die lokal oder in anderen externen Rechenzentren gehostet werden?</span><span class="sxs-lookup"><span data-stu-id="fefbd-190">Does it need to be accessible from applications or services hosted on-premises or in other external datacenters?</span></span>

### <a name="devops"></a><span data-ttu-id="fefbd-191">DevOps</span><span class="sxs-lookup"><span data-stu-id="fefbd-191">DevOps</span></span>

- <span data-ttu-id="fefbd-192">**Fähigkeiten:**</span><span class="sxs-lookup"><span data-stu-id="fefbd-192">**Skill set**.</span></span> <span data-ttu-id="fefbd-193">Ist Ihr Team besonders vertraut mit der Verwendung bestimmter Programmiersprachen, Betriebssysteme oder anderen Technologien?</span><span class="sxs-lookup"><span data-stu-id="fefbd-193">Are there particular programming languages, operating systems, or other technology that your team is particularly adept at using?</span></span> <span data-ttu-id="fefbd-194">Stellen andere dagegen potenziell eine Schwierigkeit für das Team dar?</span><span class="sxs-lookup"><span data-stu-id="fefbd-194">Are there others that would be difficult for your team to work with?</span></span>
- <span data-ttu-id="fefbd-195">**Clients:** Besteht eine gute Clientunterstützung für Ihre Entwicklungssprachen?</span><span class="sxs-lookup"><span data-stu-id="fefbd-195">**Clients** Is there good client support for your development languages?</span></span>

<span data-ttu-id="fefbd-196">In den folgenden Abschnitten werden verschiedene Datenspeichermodelle in Bezug auf das Workloadprofil, die Datentypen und Beispiele für Anwendungsfälle verglichen.</span><span class="sxs-lookup"><span data-stu-id="fefbd-196">The following sections compare various data store models in terms of workload profile, data types, and example use cases.</span></span>

## <a name="relational-database-management-systems-rdbms"></a><span data-ttu-id="fefbd-197">Managementsysteme für relationale Datenbanken (RDBMS)</span><span class="sxs-lookup"><span data-stu-id="fefbd-197">Relational database management systems (RDBMS)</span></span>

<table>
<tr><td><span data-ttu-id="fefbd-198">**Workload**</span><span class="sxs-lookup"><span data-stu-id="fefbd-198">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-199">Die Erstellung neuer Datensätze sowie die Aktualisierung vorhandener Daten werden regelmäßig durchgeführt.</span><span class="sxs-lookup"><span data-stu-id="fefbd-199">Both the creation of new records and updates to existing data happen regularly.</span></span></li>
            <li><span data-ttu-id="fefbd-200">Mehrere Vorgänge müssen in einer einzelnen Transaktion abgeschlossen werden.</span><span class="sxs-lookup"><span data-stu-id="fefbd-200">Multiple operations have to be completed in a single transaction.</span></span></li>
            <li><span data-ttu-id="fefbd-201">Aggregationsfunktionen sind für Kreuztabellen erforderlich.</span><span class="sxs-lookup"><span data-stu-id="fefbd-201">Requires aggregation functions to perform cross-tabulation.</span></span></li>
            <li><span data-ttu-id="fefbd-202">Starke Integration in Berichtstools ist erforderlich.</span><span class="sxs-lookup"><span data-stu-id="fefbd-202">Strong integration with reporting tools is required.</span></span></li>
            <li><span data-ttu-id="fefbd-203">Beziehungen werden mithilfe von Datenbankeinschränkungen erzwungen.</span><span class="sxs-lookup"><span data-stu-id="fefbd-203">Relationships are enforced using database constraints.</span></span></li>
            <li><span data-ttu-id="fefbd-204">Die Abfrageleistung wird mithilfe von Indizes optimiert.</span><span class="sxs-lookup"><span data-stu-id="fefbd-204">Indexes are used to optimize query performance.</span></span></li>
            <li><span data-ttu-id="fefbd-205">Ermöglicht den Zugriff auf bestimmte Teilmengen von Daten.</span><span class="sxs-lookup"><span data-stu-id="fefbd-205">Allows access to specific subsets of data.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-206">**Datentyp**</span><span class="sxs-lookup"><span data-stu-id="fefbd-206">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-207">Daten sind stark normalisiert.</span><span class="sxs-lookup"><span data-stu-id="fefbd-207">Data is highly normalized.</span></span></li>
            <li><span data-ttu-id="fefbd-208">Datenbankschemas sind erforderlich und werden erzwungen.</span><span class="sxs-lookup"><span data-stu-id="fefbd-208">Database schemas are required and enforced.</span></span></li>
            <li><span data-ttu-id="fefbd-209">m:n-Beziehungen zwischen Datenentitäten in der Datenbank.</span><span class="sxs-lookup"><span data-stu-id="fefbd-209">Many-to-many relationships between data entities in the database.</span></span></li>
            <li><span data-ttu-id="fefbd-210">Einschränkungen werden im Schema definiert und gelten für alle Daten in der Datenbank.</span><span class="sxs-lookup"><span data-stu-id="fefbd-210">Constraints are defined in the schema and imposed on any data in the database.</span></span></li>
            <li><span data-ttu-id="fefbd-211">Daten erfordern eine hohe Integrität.</span><span class="sxs-lookup"><span data-stu-id="fefbd-211">Data requires high integrity.</span></span> <span data-ttu-id="fefbd-212">Indizes und Beziehungen müssen genau verwaltet werden.</span><span class="sxs-lookup"><span data-stu-id="fefbd-212">Indexes and relationships need to be maintained accurately.</span></span></li>
            <li><span data-ttu-id="fefbd-213">Daten erfordern eine starke Konsistenz.</span><span class="sxs-lookup"><span data-stu-id="fefbd-213">Data requires strong consistency.</span></span> <span data-ttu-id="fefbd-214">Transaktionen werden so durchgeführt, dass sichergestellt wird, dass alle Daten für alle Benutzer und Prozesse 100 % konsistent sind.</span><span class="sxs-lookup"><span data-stu-id="fefbd-214">Transactions operate in a way that ensures all data are 100% consistent for all users and processes.</span></span></li>
            <li><span data-ttu-id="fefbd-215">Die Größe der einzelnen Dateneinträge soll klein bis mittelgroß sein.</span><span class="sxs-lookup"><span data-stu-id="fefbd-215">Size of individual data entries is intended to be small to medium-sized.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-216">**Beispiele**</span><span class="sxs-lookup"><span data-stu-id="fefbd-216">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-217">Branchenspezifisch (Personalverwaltung, Customer Relationship Management, Enterprise Resource Planning)</span><span class="sxs-lookup"><span data-stu-id="fefbd-217">Line of business  (human capital management, customer relationship management, enterprise resource planning)</span></span></li>
            <li><span data-ttu-id="fefbd-218">Bestandsverwaltung</span><span class="sxs-lookup"><span data-stu-id="fefbd-218">Inventory management</span></span></li>
            <li><span data-ttu-id="fefbd-219">Berichtsdatenbank</span><span class="sxs-lookup"><span data-stu-id="fefbd-219">Reporting database</span></span></li>
            <li><span data-ttu-id="fefbd-220">Buchhaltung</span><span class="sxs-lookup"><span data-stu-id="fefbd-220">Accounting</span></span></li>
            <li><span data-ttu-id="fefbd-221">Asset-Management</span><span class="sxs-lookup"><span data-stu-id="fefbd-221">Asset management</span></span></li>
            <li><span data-ttu-id="fefbd-222">Fondsmanagement</span><span class="sxs-lookup"><span data-stu-id="fefbd-222">Fund management</span></span></li>
            <li><span data-ttu-id="fefbd-223">Bestellungsverwaltung</span><span class="sxs-lookup"><span data-stu-id="fefbd-223">Order management</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="document-databases"></a><span data-ttu-id="fefbd-224">Dokumentdatenbanken</span><span class="sxs-lookup"><span data-stu-id="fefbd-224">Document databases</span></span>

<table>
<tr><td><span data-ttu-id="fefbd-225">**Workload**</span><span class="sxs-lookup"><span data-stu-id="fefbd-225">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-226">Allgemeiner Zweck.</span><span class="sxs-lookup"><span data-stu-id="fefbd-226">General purpose.</span></span></li>
            <li><span data-ttu-id="fefbd-227">Einfüge- und Aktualisierungsvorgänge werden häufig durchgeführt.</span><span class="sxs-lookup"><span data-stu-id="fefbd-227">Insert and update operations are common.</span></span> <span data-ttu-id="fefbd-228">Die Erstellung neuer Datensätze sowie die Aktualisierung vorhandener Daten werden regelmäßig durchgeführt.</span><span class="sxs-lookup"><span data-stu-id="fefbd-228">Both the creation of new records and updates to existing data happen regularly.</span></span></li>
            <li><span data-ttu-id="fefbd-229">Keine objektrelationalen Impedanzabweichungen.</span><span class="sxs-lookup"><span data-stu-id="fefbd-229">No object-relational impedance mismatch.</span></span> <span data-ttu-id="fefbd-230">Dokumente können besser mit den im Anwendungscode verwendeten Objektstrukturen abgeglichen werden.</span><span class="sxs-lookup"><span data-stu-id="fefbd-230">Documents can better match the object structures used in application code.</span></span></li>
            <li><span data-ttu-id="fefbd-231">Optimistische Parallelität wird häufiger verwendet.</span><span class="sxs-lookup"><span data-stu-id="fefbd-231">Optimistic concurrency is more commonly used.</span></span></li>
            <li><span data-ttu-id="fefbd-232">Daten müssen durch die verarbeitende Anwendung geändert und verarbeitet werden.</span><span class="sxs-lookup"><span data-stu-id="fefbd-232">Data must be modified and processed by consuming application.</span></span></li>
            <li><span data-ttu-id="fefbd-233">Für die Daten ist ein Index für mehrere Felder erforderlich.</span><span class="sxs-lookup"><span data-stu-id="fefbd-233">Data requires index on multiple fields.</span></span></li>
            <li><span data-ttu-id="fefbd-234">Einzelne Dokumente werden abgerufen und als einzelner Block geschrieben.</span><span class="sxs-lookup"><span data-stu-id="fefbd-234">Individual documents are retrieved and written as a single block.</span></span></li>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-235">**Datentyp**</span><span class="sxs-lookup"><span data-stu-id="fefbd-235">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-236">Daten können auf denormalisierte Weise verwaltet werden.</span><span class="sxs-lookup"><span data-stu-id="fefbd-236">Data can be managed in de-normalized way.</span></span></li>
            <li><span data-ttu-id="fefbd-237">Die Größe der einzelnen Dokumentdaten ist relativ gering.</span><span class="sxs-lookup"><span data-stu-id="fefbd-237">Size of individual document data is relatively small.</span></span></li>
            <li><span data-ttu-id="fefbd-238">Für jeden Dokumenttyp kann ein eigenes Schema verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="fefbd-238">Each document type can use its own schema.</span></span></li>
            <li><span data-ttu-id="fefbd-239">Dokumente können optionale Felder enthalten.</span><span class="sxs-lookup"><span data-stu-id="fefbd-239">Documents can include optional fields.</span></span></li>
            <li><span data-ttu-id="fefbd-240">Dokumentdaten sind teilweise strukturiert, d.h., die Datentypen der einzelnen Felder sind nicht streng definiert.</span><span class="sxs-lookup"><span data-stu-id="fefbd-240">Document data is semi-structured, meaning that data types of each field are not strictly defined.</span></span></li>
            <li><span data-ttu-id="fefbd-241">Die Datenaggregation wird unterstützt.</span><span class="sxs-lookup"><span data-stu-id="fefbd-241">Data aggregation is supported.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-242">**Beispiele**</span><span class="sxs-lookup"><span data-stu-id="fefbd-242">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-243">Produktkatalog</span><span class="sxs-lookup"><span data-stu-id="fefbd-243">Product catalog</span></span></li>
            <li><span data-ttu-id="fefbd-244">Benutzerkonten</span><span class="sxs-lookup"><span data-stu-id="fefbd-244">User accounts</span></span></li>
            <li><span data-ttu-id="fefbd-245">Stückliste</span><span class="sxs-lookup"><span data-stu-id="fefbd-245">Bill of materials</span></span></li>
            <li><span data-ttu-id="fefbd-246">Personalisierung</span><span class="sxs-lookup"><span data-stu-id="fefbd-246">Personalization</span></span></li>
            <li><span data-ttu-id="fefbd-247">Content Management</span><span class="sxs-lookup"><span data-stu-id="fefbd-247">Content management</span></span></li>
            <li><span data-ttu-id="fefbd-248">Betriebsdaten</span><span class="sxs-lookup"><span data-stu-id="fefbd-248">Operations data</span></span></li>
            <li><span data-ttu-id="fefbd-249">Bestandsverwaltung</span><span class="sxs-lookup"><span data-stu-id="fefbd-249">Inventory management</span></span></li>
            <li><span data-ttu-id="fefbd-250">Transaktionsverlaufsdaten</span><span class="sxs-lookup"><span data-stu-id="fefbd-250">Transaction history data</span></span></li>
            <li><span data-ttu-id="fefbd-251">Materialisierte Sicht anderer NoSQL-Datenspeicher.</span><span class="sxs-lookup"><span data-stu-id="fefbd-251">Materialized view of other NoSQL stores.</span></span> <span data-ttu-id="fefbd-252">Ersetzt Datei-/Blob-Indizierung.</span><span class="sxs-lookup"><span data-stu-id="fefbd-252">Replaces file/BLOB indexing.</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="keyvalue-stores"></a><span data-ttu-id="fefbd-253">Schlüssel-Wert-Speicher</span><span class="sxs-lookup"><span data-stu-id="fefbd-253">Key/value stores</span></span>

<table>
<tr><td><span data-ttu-id="fefbd-254">**Workload**</span><span class="sxs-lookup"><span data-stu-id="fefbd-254">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-255">Die Identifizierung und der Zugriff auf Daten erfolgen mithilfe eines einzigen ID-Schlüssels, z.B. über ein Wörterbuch.</span><span class="sxs-lookup"><span data-stu-id="fefbd-255">Data is identified and accessed using a single ID key, like a dictionary.</span></span></li>
            <li><span data-ttu-id="fefbd-256">Extrem skalierbar.</span><span class="sxs-lookup"><span data-stu-id="fefbd-256">Massively scalable.</span></span></li>
            <li><span data-ttu-id="fefbd-257">Keine Verknüpfungen, Sperren oder Unions sind erforderlich.</span><span class="sxs-lookup"><span data-stu-id="fefbd-257">No joins, lock, or unions are required.</span></span></li>
            <li><span data-ttu-id="fefbd-258">Keine Aggregationsmechanismen werden verwendet.</span><span class="sxs-lookup"><span data-stu-id="fefbd-258">No aggregation mechanisms are used.</span></span></li>
            <li><span data-ttu-id="fefbd-259">Sekundäre Indizes werden generell nicht verwendet.</span><span class="sxs-lookup"><span data-stu-id="fefbd-259">Secondary indexes are generally not used.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-260">**Datentyp**</span><span class="sxs-lookup"><span data-stu-id="fefbd-260">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-261">Die Datengröße ist tendenziell umfangreich.</span><span class="sxs-lookup"><span data-stu-id="fefbd-261">Data size tends to be large.</span></span></li>
            <li><span data-ttu-id="fefbd-262">Jeder Schlüssel ist einem einzelnen Wert zugeordnet, d.h. einem nicht verwalteten Datenblob.</span><span class="sxs-lookup"><span data-stu-id="fefbd-262">Each key is associated with a single value, which is an unmanaged data BLOB.</span></span></li>
            <li><span data-ttu-id="fefbd-263">Keine Schemas werden erzwungen.</span><span class="sxs-lookup"><span data-stu-id="fefbd-263">There is no schema enforcement.</span></span></li>
            <li><span data-ttu-id="fefbd-264">Keine Beziehungen zwischen Entitäten.</span><span class="sxs-lookup"><span data-stu-id="fefbd-264">No relationships between entities.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-265">**Beispiele**</span><span class="sxs-lookup"><span data-stu-id="fefbd-265">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-266">Datenzwischenspeicherung</span><span class="sxs-lookup"><span data-stu-id="fefbd-266">Data caching</span></span></li>
            <li><span data-ttu-id="fefbd-267">Sitzungsverwaltung</span><span class="sxs-lookup"><span data-stu-id="fefbd-267">Session management</span></span></li>
            <li><span data-ttu-id="fefbd-268">Benutzereinstellungs- und Benutzerprofilverwaltung</span><span class="sxs-lookup"><span data-stu-id="fefbd-268">User preference and profile management</span></span></li>
            <li><span data-ttu-id="fefbd-269">Produktempfehlungen und Ad-Serving</span><span class="sxs-lookup"><span data-stu-id="fefbd-269">Product recommendation and ad serving</span></span></li>
            <li><span data-ttu-id="fefbd-270">Wörterbücher</span><span class="sxs-lookup"><span data-stu-id="fefbd-270">Dictionaries</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="graph-databases"></a><span data-ttu-id="fefbd-271">Diagrammdatenbanken</span><span class="sxs-lookup"><span data-stu-id="fefbd-271">Graph databases</span></span>

<table>
<tr><td><span data-ttu-id="fefbd-272">**Workload**</span><span class="sxs-lookup"><span data-stu-id="fefbd-272">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-273">Die Beziehungen zwischen Datenelementen sind sehr komplex und erfordern viele Hops zwischen zugehörigen Datenelementen.</span><span class="sxs-lookup"><span data-stu-id="fefbd-273">The relationships between data items are very complex, involving many hops between related data items.</span></span></li>
            <li><span data-ttu-id="fefbd-274">Die Beziehungen zwischen Datenelementen sind dynamisch und ändern sich mit der Zeit.</span><span class="sxs-lookup"><span data-stu-id="fefbd-274">The relationship between data items are dynamic and change over time.</span></span></li>
            <li><span data-ttu-id="fefbd-275">Beziehungen zwischen Objekten sind bevorzugte Beziehungen, bei denen keine Fremdschlüssel und Verknüpfungen durchlaufen werden müssen.</span><span class="sxs-lookup"><span data-stu-id="fefbd-275">Relationships between objects are first-class citizens, without requiring foreign-keys and joins to traverse.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-276">**Datentyp**</span><span class="sxs-lookup"><span data-stu-id="fefbd-276">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-277">Daten bestehen aus Knoten und Beziehungen.</span><span class="sxs-lookup"><span data-stu-id="fefbd-277">Data is comprised of nodes and relationships.</span></span></li>
            <li><span data-ttu-id="fefbd-278">Knoten ähneln Tabellenzeilen oder JSON-Dokumenten.</span><span class="sxs-lookup"><span data-stu-id="fefbd-278">Nodes are similar to table rows or JSON documents.</span></span></li>
            <li><span data-ttu-id="fefbd-279">Beziehungen sind genau so wichtig wie Knoten und werden direkt in der Abfragesprache verfügbar gemacht.</span><span class="sxs-lookup"><span data-stu-id="fefbd-279">Relationships are just as important as nodes, and are exposed directly in the query language.</span></span></li>
            <li><span data-ttu-id="fefbd-280">Zusammengesetzte Objekte, z.B. eine Person mit mehreren Telefonnummern, werden meist in separate kleinere Knoten unterteilt und mit durchlaufbaren Beziehungen kombiniert.</span><span class="sxs-lookup"><span data-stu-id="fefbd-280">Composite objects, such as a person with multiple phone numbers, tend to be broken into separate, smaller nodes, combined with traversable relationships</span></span> </li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-281">**Beispiele**</span><span class="sxs-lookup"><span data-stu-id="fefbd-281">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-282">Organigramme</span><span class="sxs-lookup"><span data-stu-id="fefbd-282">Organization charts</span></span></li>
            <li><span data-ttu-id="fefbd-283">Social Graphs</span><span class="sxs-lookup"><span data-stu-id="fefbd-283">Social graphs</span></span></li>
            <li><span data-ttu-id="fefbd-284">Betrugserkennung</span><span class="sxs-lookup"><span data-stu-id="fefbd-284">Fraud detection</span></span></li>
            <li><span data-ttu-id="fefbd-285">Analyse</span><span class="sxs-lookup"><span data-stu-id="fefbd-285">Analytics</span></span></li>
            <li><span data-ttu-id="fefbd-286">Empfehlungsmodule</span><span class="sxs-lookup"><span data-stu-id="fefbd-286">Recommendation engines</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="column-family-databases"></a><span data-ttu-id="fefbd-287">Column-Family-Datenbanken</span><span class="sxs-lookup"><span data-stu-id="fefbd-287">Column-family databases</span></span>

<table>
<tr><td><span data-ttu-id="fefbd-288">**Workload**</span><span class="sxs-lookup"><span data-stu-id="fefbd-288">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-289">In den meisten Column-Family-Datenbanken werden Schreibvorgänge extrem schnell durchgeführt.</span><span class="sxs-lookup"><span data-stu-id="fefbd-289">Most column-family databases perform write operations extremely quickly.</span></span></li>
            <li><span data-ttu-id="fefbd-290">Aktualisierungs- und Löschvorgänge sind selten.</span><span class="sxs-lookup"><span data-stu-id="fefbd-290">Update and delete operations are rare.</span></span></li>
            <li><span data-ttu-id="fefbd-291">Bietet Zugriff mit hohem Durchsatz und niedriger Latenz.</span><span class="sxs-lookup"><span data-stu-id="fefbd-291">Designed to provide high throughput and low-latency access.</span></span></li>
            <li><span data-ttu-id="fefbd-292">Unterstützt den einfachen Abfragezugriff auf eine bestimmte Gruppe von Feldern in einem wesentlich größeren Datensatz.</span><span class="sxs-lookup"><span data-stu-id="fefbd-292">Supports easy query access to a particular set of fields within a much larger record.</span></span></li>
            <li><span data-ttu-id="fefbd-293">Extrem skalierbar.</span><span class="sxs-lookup"><span data-stu-id="fefbd-293">Massively scalable.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-294">**Datentyp**</span><span class="sxs-lookup"><span data-stu-id="fefbd-294">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-295">Daten werden in Tabellen gespeichert, die aus einer Schlüsselspalte und einer oder mehreren Spaltenfamilien bestehen.</span><span class="sxs-lookup"><span data-stu-id="fefbd-295">Data is stored in tables consisting of a key column and one or more column families.</span></span></li>
            <li><span data-ttu-id="fefbd-296">Bestimmte Spalten können nach einzelnen Zeilen variieren.</span><span class="sxs-lookup"><span data-stu-id="fefbd-296">Specific columns can vary by individual rows.</span></span></li>
            <li><span data-ttu-id="fefbd-297">Der Zugriff auf einzelne Zellen erfolgt über get- und put-Befehle.</span><span class="sxs-lookup"><span data-stu-id="fefbd-297">Individual cells are accessed via get and put commands</span></span></li>
            <li><span data-ttu-id="fefbd-298">Mehrere Zeilen werden unter Verwendung eines scan-Befehls zurückgegeben.</span><span class="sxs-lookup"><span data-stu-id="fefbd-298">Multiple rows are returned using a scan command.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-299">**Beispiele**</span><span class="sxs-lookup"><span data-stu-id="fefbd-299">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-300">Recommendations</span><span class="sxs-lookup"><span data-stu-id="fefbd-300">Recommendations</span></span></li>
            <li><span data-ttu-id="fefbd-301">Personalisierung</span><span class="sxs-lookup"><span data-stu-id="fefbd-301">Personalization</span></span></li>
            <li><span data-ttu-id="fefbd-302">Sensordaten</span><span class="sxs-lookup"><span data-stu-id="fefbd-302">Sensor data</span></span></li>
            <li><span data-ttu-id="fefbd-303">Telemetrie</span><span class="sxs-lookup"><span data-stu-id="fefbd-303">Telemetry</span></span></li>
            <li><span data-ttu-id="fefbd-304">Nachrichten</span><span class="sxs-lookup"><span data-stu-id="fefbd-304">Messaging</span></span></li>
            <li><span data-ttu-id="fefbd-305">Analysen sozialer Medien</span><span class="sxs-lookup"><span data-stu-id="fefbd-305">Social media analytics</span></span></li>
            <li><span data-ttu-id="fefbd-306">Webanalysen</span><span class="sxs-lookup"><span data-stu-id="fefbd-306">Web analytics</span></span></li>
            <li><span data-ttu-id="fefbd-307">Aktivitätsüberwachung</span><span class="sxs-lookup"><span data-stu-id="fefbd-307">Activity monitoring</span></span></li>
            <li><span data-ttu-id="fefbd-308">Wetter- und andere Zeitreihendaten</span><span class="sxs-lookup"><span data-stu-id="fefbd-308">Weather and other time-series data</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="search-engine-databases"></a><span data-ttu-id="fefbd-309">Suchmaschinen-Datenbanken</span><span class="sxs-lookup"><span data-stu-id="fefbd-309">Search engine databases</span></span>

<table>
<tr><td><span data-ttu-id="fefbd-310">**Workload**</span><span class="sxs-lookup"><span data-stu-id="fefbd-310">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-311">Indizierung von Daten aus mehreren Quellen und Diensten.</span><span class="sxs-lookup"><span data-stu-id="fefbd-311">Indexing data from multiple sources and services.</span></span></li>
            <li><span data-ttu-id="fefbd-312">Abfragen werden ad hoc durchgeführt und können komplex sein.</span><span class="sxs-lookup"><span data-stu-id="fefbd-312">Queries are ad-hoc and can be complex.</span></span></li>
            <li><span data-ttu-id="fefbd-313">Aggregation ist erforderlich.</span><span class="sxs-lookup"><span data-stu-id="fefbd-313">Requires aggregation.</span></span></li>
            <li><span data-ttu-id="fefbd-314">Volltextsuche ist erforderlich.</span><span class="sxs-lookup"><span data-stu-id="fefbd-314">Full text search is required.</span></span></li>
            <li><span data-ttu-id="fefbd-315">Ad-hoc-Self-Service-Abfragen sind erforderlich.</span><span class="sxs-lookup"><span data-stu-id="fefbd-315">Ad hoc self-service query is required.</span></span></li>
            <li><span data-ttu-id="fefbd-316">Datenanalyse mit Index für alle Felder ist erforderlich.</span><span class="sxs-lookup"><span data-stu-id="fefbd-316">Data analysis with index on all fields is required.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-317">**Datentyp**</span><span class="sxs-lookup"><span data-stu-id="fefbd-317">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-318">Teilweise strukturiert oder unstrukturiert</span><span class="sxs-lookup"><span data-stu-id="fefbd-318">Semi-structured or unstructured</span></span></li>
            <li><span data-ttu-id="fefbd-319">Text</span><span class="sxs-lookup"><span data-stu-id="fefbd-319">Text</span></span></li>
            <li><span data-ttu-id="fefbd-320">Text mit Verweis auf strukturierte Daten</span><span class="sxs-lookup"><span data-stu-id="fefbd-320">Text with reference to structured data</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-321">**Beispiele**</span><span class="sxs-lookup"><span data-stu-id="fefbd-321">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-322">Produktkataloge</span><span class="sxs-lookup"><span data-stu-id="fefbd-322">Product catalogs</span></span></li>
            <li><span data-ttu-id="fefbd-323">Websitesuche</span><span class="sxs-lookup"><span data-stu-id="fefbd-323">Site search</span></span></li>
            <li><span data-ttu-id="fefbd-324">Protokollierung</span><span class="sxs-lookup"><span data-stu-id="fefbd-324">Logging</span></span></li>
            <li><span data-ttu-id="fefbd-325">Analyse</span><span class="sxs-lookup"><span data-stu-id="fefbd-325">Analytics</span></span></li>
            <li><span data-ttu-id="fefbd-326">Shopping-Websites</span><span class="sxs-lookup"><span data-stu-id="fefbd-326">Shopping sites</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="data-warehouse"></a><span data-ttu-id="fefbd-327">Data Warehouse</span><span class="sxs-lookup"><span data-stu-id="fefbd-327">Data warehouse</span></span>

<table>
<tr><td><span data-ttu-id="fefbd-328">**Workload**</span><span class="sxs-lookup"><span data-stu-id="fefbd-328">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-329">Datenanalysen</span><span class="sxs-lookup"><span data-stu-id="fefbd-329">Data analytics</span></span></li>
            <li><span data-ttu-id="fefbd-330">Enterprise BI</span><span class="sxs-lookup"><span data-stu-id="fefbd-330">Enterprise BI</span></span>   </li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-331">**Datentyp**</span><span class="sxs-lookup"><span data-stu-id="fefbd-331">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-332">Verlaufsdaten aus mehreren Quellen.</span><span class="sxs-lookup"><span data-stu-id="fefbd-332">Historical data from multiple sources.</span></span></li>
            <li><span data-ttu-id="fefbd-333">In der Regel denormalisiert in einem „Stern“- oder „Schneeflocken“-Schema, das aus Fakten- und Dimensionstabellen besteht.</span><span class="sxs-lookup"><span data-stu-id="fefbd-333">Usually denormalized in a "star" or "snowflake" schema, consisting of fact and dimension tables.</span></span></li>
            <li><span data-ttu-id="fefbd-334">Wird normalerweise mit neuen Daten auf Basis eines Zeitplans geladen.</span><span class="sxs-lookup"><span data-stu-id="fefbd-334">Usually loaded with new data on a scheduled basis.</span></span></li>
            <li><span data-ttu-id="fefbd-335">Dimensionstabellen enthalten häufig mehrere Verlaufsversionen einer Entität, die als *langsam veränderliche Dimension* bezeichnet wird.</span><span class="sxs-lookup"><span data-stu-id="fefbd-335">Dimension tables often include multiple historic versions of an entity, referred to as a *slowly changing dimension*.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-336">**Beispiele**</span><span class="sxs-lookup"><span data-stu-id="fefbd-336">**Examples**</span></span></td>
    <td><span data-ttu-id="fefbd-337">Enterprise Data Warehouse, das Daten für analytische Modelle, Berichte und Dashboards umfasst</span><span class="sxs-lookup"><span data-stu-id="fefbd-337">An enterprise data warehouse that provides data for analytical models, reports, and dashboards.</span></span>
    </td>
</tr>
</table>


## <a name="time-series-databases"></a><span data-ttu-id="fefbd-338">Zeitreihendatenbanken</span><span class="sxs-lookup"><span data-stu-id="fefbd-338">Time series databases</span></span>

<table>
<tr><td><span data-ttu-id="fefbd-339">**Workload**</span><span class="sxs-lookup"><span data-stu-id="fefbd-339">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-340">Der überwiegende Anteil der Vorgänge sind Schreibvorgänge (95 bis 99 %).</span><span class="sxs-lookup"><span data-stu-id="fefbd-340">An overwhelmingly proportion of operations (95-99%) are writes.</span></span></li>
            <li><span data-ttu-id="fefbd-341">Datensätze werden generell sequenziell in zeitlicher Reihenfolge angefügt.</span><span class="sxs-lookup"><span data-stu-id="fefbd-341">Records are generally appended sequentially in time order.</span></span></li>
            <li><span data-ttu-id="fefbd-342">Aktualisierungen sind selten.</span><span class="sxs-lookup"><span data-stu-id="fefbd-342">Updates are rare.</span></span></li>
            <li><span data-ttu-id="fefbd-343">Löschvorgänge werden massenweise und in zusammenhängenden Blöcken oder Datensätzen durchgeführt.</span><span class="sxs-lookup"><span data-stu-id="fefbd-343">Deletes occur in bulk, and are made to contiguous blocks or records.</span></span></li>
            <li><span data-ttu-id="fefbd-344">Leseanforderungen können größer als der verfügbare Arbeitsspeicher sein.</span><span class="sxs-lookup"><span data-stu-id="fefbd-344">Read requests can be larger than available memory.</span></span></li>
            <li><span data-ttu-id="fefbd-345">Üblicherweise erfolgen mehrere Lesevorgänge gleichzeitig.</span><span class="sxs-lookup"><span data-stu-id="fefbd-345">It's common for multiple reads to occur simultaneously.</span></span></li>
            <li><span data-ttu-id="fefbd-346">Daten werden sequenziell in aufsteigender oder absteigender zeitlicher Reihenfolge gelesen.</span><span class="sxs-lookup"><span data-stu-id="fefbd-346">Data is read sequentially in either ascending or descending time order.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-347">**Datentyp**</span><span class="sxs-lookup"><span data-stu-id="fefbd-347">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-348">Ein Zeitstempel wird als primärer Schlüssel und Sortiermechanismus verwendet.</span><span class="sxs-lookup"><span data-stu-id="fefbd-348">A time stamp that is used as the primary key and sorting mechanism.</span></span></li>
            <li><span data-ttu-id="fefbd-349">Messungen ab dem Eintrag oder Beschreibungen des Eintrags.</span><span class="sxs-lookup"><span data-stu-id="fefbd-349">Measurements from the entry or descriptions of what the entry represents.</span></span></li>
            <li><span data-ttu-id="fefbd-350">Mit Tags werden zusätzliche Informationen zum Typ oder Ursprung sowie andere Informationen zum Eintrag definiert.</span><span class="sxs-lookup"><span data-stu-id="fefbd-350">Tags that define additional information about the type, origin, and other information about the entry.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-351">**Beispiele**</span><span class="sxs-lookup"><span data-stu-id="fefbd-351">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-352">Überwachungs- und Ereignistelemetrie</span><span class="sxs-lookup"><span data-stu-id="fefbd-352">Monitoring and event telemetry.</span></span></li>
            <li><span data-ttu-id="fefbd-353">Sensor- oder andere IoT-Daten</span><span class="sxs-lookup"><span data-stu-id="fefbd-353">Sensor or other IoT data.</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="object-storage"></a><span data-ttu-id="fefbd-354">Objektspeicher</span><span class="sxs-lookup"><span data-stu-id="fefbd-354">Object storage</span></span>

<table>
<tr><td><span data-ttu-id="fefbd-355">**Workload**</span><span class="sxs-lookup"><span data-stu-id="fefbd-355">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-356">Identifizierung erfolgt nach Schlüssel.</span><span class="sxs-lookup"><span data-stu-id="fefbd-356">Identified by key.</span></span></li>
            <li><span data-ttu-id="fefbd-357">Auf Objekte kann öffentlich oder privat zugegriffen werden.</span><span class="sxs-lookup"><span data-stu-id="fefbd-357">Objects may be publicly or privately accessible.</span></span></li>
            <li><span data-ttu-id="fefbd-358">Inhalte sind normalerweise Ressourcen, z.B. eine Tabelle, ein Bild oder eine Videodatei.</span><span class="sxs-lookup"><span data-stu-id="fefbd-358">Content is typically an asset such as a spreadsheet, image, or video file.</span></span></li>
            <li><span data-ttu-id="fefbd-359">Inhalte müssen dauerhaft (persistent) sein und sich außerhalb von Anwendungsebenen oder virtuellen Computern befinden.</span><span class="sxs-lookup"><span data-stu-id="fefbd-359">Content must be durable (persistent), and external to any application tier or virtual machine.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-360">**Datentyp**</span><span class="sxs-lookup"><span data-stu-id="fefbd-360">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-361">Die Datengröße ist umfangreich.</span><span class="sxs-lookup"><span data-stu-id="fefbd-361">Data size is large.</span></span></li>
            <li><span data-ttu-id="fefbd-362">Blobdaten</span><span class="sxs-lookup"><span data-stu-id="fefbd-362">Blob data.</span></span></li>
            <li><span data-ttu-id="fefbd-363">Wert ist nicht transparent.</span><span class="sxs-lookup"><span data-stu-id="fefbd-363">Value is opaque.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-364">**Beispiele**</span><span class="sxs-lookup"><span data-stu-id="fefbd-364">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-365">Bilder, Videos, Office-Dokumente, PDF-Dateien</span><span class="sxs-lookup"><span data-stu-id="fefbd-365">Images, videos, office documents, PDFs</span></span></li>
            <li><span data-ttu-id="fefbd-366">CSS, Skripts, CSV</span><span class="sxs-lookup"><span data-stu-id="fefbd-366">CSS, Scripts, CSV</span></span></li>
            <li><span data-ttu-id="fefbd-367">Statisches HTML, JSON</span><span class="sxs-lookup"><span data-stu-id="fefbd-367">Static HTML, JSON</span></span></li>
            <li><span data-ttu-id="fefbd-368">Protokoll- und Überwachungsdateien</span><span class="sxs-lookup"><span data-stu-id="fefbd-368">Log and audit files</span></span></li>
            <li><span data-ttu-id="fefbd-369">Datenbanksicherungen</span><span class="sxs-lookup"><span data-stu-id="fefbd-369">Database backups</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="shared-files"></a><span data-ttu-id="fefbd-370">Freigegebene Dateien</span><span class="sxs-lookup"><span data-stu-id="fefbd-370">Shared files</span></span>

<table>
<tr><td><span data-ttu-id="fefbd-371">**Workload**</span><span class="sxs-lookup"><span data-stu-id="fefbd-371">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-372">Migration aus vorhandenen Apps, die mit dem Dateisystem interagieren.</span><span class="sxs-lookup"><span data-stu-id="fefbd-372">Migration from existing apps that interact with the file system.</span></span></li>
            <li><span data-ttu-id="fefbd-373">Erfordert eine SMB-Schnittstelle.</span><span class="sxs-lookup"><span data-stu-id="fefbd-373">Requires SMB interface.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-374">**Datentyp**</span><span class="sxs-lookup"><span data-stu-id="fefbd-374">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-375">Dateien in einer hierarchischen Gruppe von Ordnern.</span><span class="sxs-lookup"><span data-stu-id="fefbd-375">Files in a hierarchical set of folders.</span></span></li>
            <li><span data-ttu-id="fefbd-376">Zugänglich über E/A-Standardbibliotheken.</span><span class="sxs-lookup"><span data-stu-id="fefbd-376">Accessible with standard I/O libraries.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="fefbd-377">**Beispiele**</span><span class="sxs-lookup"><span data-stu-id="fefbd-377">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="fefbd-378">Legacydateien</span><span class="sxs-lookup"><span data-stu-id="fefbd-378">Legacy files</span></span></li>
            <li><span data-ttu-id="fefbd-379">Freigegebener Inhalt zugänglich innerhalb verschiedener virtueller Computer oder App-Instanzen</span><span class="sxs-lookup"><span data-stu-id="fefbd-379">Shared content accessible among a number of VMs or app instances</span></span></li>
        </ul>
    </td>
</tr>
</table>
