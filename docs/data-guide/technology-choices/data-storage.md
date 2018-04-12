---
title: Auswählen einer Datenspeichertechnologie
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: b14611a2dc34bcb145cf420441795d4124e7baeb
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="choosing-a-big-data-storage-technology-in-azure"></a>Auswählen einer Big Data-Speichertechnologie in Azure

In diesem Thema werden Datenspeicheroptionen für Big Data-Lösungen verglichen – insbesondere Datenspeicher für Massendatenerfassung und Batchverarbeitung (im Gegensatz zu [Analysedatenspeichern](./analytical-data-stores.md) oder [Echtzeit-Streamingerfassung](./real-time-ingestion.md)).

## <a name="what-are-your-options-when-choosing-data-storage-in-azure"></a>Welche Datenspeicheroptionen stehen in Azure zur Verfügung?

Daten können auf verschiedene Arten in Azure erfasst werden. Für welche Option Sie sich entscheiden, hängt ganz von Ihren Anforderungen ab.

**File Storage**

- [Azure Storage-Blobs](/azure/storage/blobs/storage-blobs-introduction)
- [Azure Data Lake Store](/azure/data-lake-store/)

**NoSQL-Datenbanken**

- [Azure Cosmos DB](/azure/cosmos-db/)
- [HBase in HDInsight](http://hbase.apache.org/)

## <a name="azure-storage-blobs"></a>Azure Storage-Blobs

Azure Storage ist ein verwalteter, hochverfügbarer, sicherer, stabiler, skalierbarer und redundanter Speicherdienst. Microsoft übernimmt die Wartung und behandelt kritische Probleme für Sie. Die große Menge von Diensten und Tools, die mit dieser Lösung verwendet werden können, macht Azure Storage zur am weitesten verbreiteten Speicherlösung von Azure.

Für die Datenspeicherung stehen verschiedene Azure Storage-Dienste zur Verfügung. Die flexibelste Option zum Speichern von Blobs aus einer Reihe von Datenquellen ist [Blob Storage](/azure/storage/blobs/storage-blobs-introduction). Blobs sind im Grunde Dateien. Sie eignen sich unter anderem zum Speichern von Bildern, Dokumenten, HTML-Dateien, virtuellen Festplatten (Virtual Hard Disks, VHDs) und Big Data wie Protokollen und Datenbanksicherungen. Blobs werden in Containern gespeichert, die Ordnern ähneln. Ein Container dient zum Gruppieren mehrerer Blobs. Ein Speicherkonto kann eine unbegrenzte Anzahl von Containern enthalten, und in einem Container kann eine unbegrenzte Anzahl von Blobs gespeichert werden.

Azure Storage ist flexibel, hochverfügbar und kostengünstig – und somit eine gute Wahl für Big Data- und Analyselösungen. Die Lösung bietet eine heiße und eine kalte Speicherebene sowie eine Archivspeicherebene für verschiedene Anwendungsfälle. Weitere Informationen finden Sie unter [Azure Blob Storage: Speicherebenen „Heiß“ (Hot), „Kalt“ (Cool) und „Archiv“](/azure/storage/blobs/storage-blob-storage-tiers).

Auf Azure Blob Storage kann über Hadoop (verfügbar über HDInsight) zugegriffen werden. In HDInsight kann ein Blobcontainer in Azure Storage als Standarddateisystem für den Cluster verwendet werden. Über eine durch einen WASB-Treiber bereitgestellte HDFS-Schnittstelle (Hadoop Distributed File System) können sämtliche Komponenten in HDInsight direkt mit strukturierten oder unstrukturierten Daten arbeiten, die als Blobs gespeichert sind. Auf Azure Blob Storage kann auch über das PolyBase-Feature von Azure SQL Data Warehouse zugegriffen werden.

Darüber hinaus sprechen folgende Features für Azure Storage:

- [Mehrere Parallelitätsstrategien](/azure/storage/common/storage-concurrency?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
- [Optionen für Notfallwiederherstellung und Hochverfügbarkeit](/azure/storage/common/storage-disaster-recovery-guidance?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
- [Verschlüsselung ruhender Daten](/azure/storage/common/storage-service-encryption?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
- [Rollenbasierte Zugriffssteuerung (Role-Based Access Control, RBAC)](/azure/storage/common/storage-security-guide?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#management-plane-security) zum Steuern des Zugriffs mithilfe von Azure Active Directory-Benutzern und -Gruppen

## <a name="azure-data-lake-store"></a>Azure Data Lake Store

[Azure Data Lake Store](/azure/data-lake-store/) ist ein unternehmensweites riesiges Repository für Big Data-Analyseworkloads. Mit Data Lake können Sie Daten von beliebiger Größe, Art und Erfassungsgeschwindigkeit zur Durchführung operativer und explorativer Analysen an einem zentralen, [sicheren](/azure/data-lake-store/data-lake-store-overview#DataLakeStoreSecurity) Ort erfassen.

Bei Data Lake Store gelten keinerlei Einschränkungen für die Kontogröße, die Dateigröße oder die Menge an Daten, die in einem Data Lake gespeichert werden kann. Daten werden dauerhaft gespeichert, indem mehrere Kopien erstellt werden, und können für unbegrenzte Zeit im Data Lake verbleiben. Zusätzlich zur Erstellung mehrerer Kopien zum Schutz vor unerwarteten Ausfällen verteilt Data Lake auch Teile einer Datei auf mehrere einzelne Speicherserver. Dies verbessert den Lesedurchsatz, wenn die Datei zum Ausführen von Datenanalysen parallel gelesen wird.

Auf Data Lake Store kann über Hadoop (verfügbar über HDInsight) unter Verwendung der WebHDFS-kompatiblen REST-APIs zugegriffen werden. Dies ist ggf. eine geeignete Alternative zu Azure Storage, wenn die Größe individueller Dateien oder aller Dateien die von Azure Storage unterstützte Größe übersteigt. Wenn Sie Data Lake Store als primären Speicher für einen HDInsight-Cluster verwenden, sollten Sie allerdings [diese Richtlinien zur Leistungsoptimierung](/azure/data-lake-store/data-lake-store-performance-tuning-guidance#optimizing-io-intensive-jobs-on-hadoop-and-spark-workloads-on-hdinsight) berücksichtigen – insbesondere die spezifischen Richtlinien für [Spark](/azure/data-lake-store/data-lake-store-performance-tuning-spark), [Hive](/azure/data-lake-store/data-lake-store-performance-tuning-hive), [MapReduce](/azure/data-lake-store/data-lake-store-performance-tuning-mapreduce) und [Storm](/azure/data-lake-store/data-lake-store-performance-tuning-storm). Informieren Sie sich außerdem über die [regionale Verfügbarkeit](https://azure.microsoft.com/regions/#services) von Data Lake Store, da diese Lösung in weniger Regionen verfügbar ist als Azure Storage und sich in der gleichen Region befinden muss wie Ihr HDInsight-Cluster.

In Verbindung mit Azure Data Lake Analytics wurde Data Lake Store speziell für die Analyse der gespeicherten Daten konzipiert und für Datenanalyseszenarien optimiert. Auf Data Lake Store kann auch über das PolyBase-Feature von Azure SQL Data Warehouse zugegriffen werden.

## <a name="azure-cosmos-db"></a>Azure Cosmos DB

[Azure Cosmos DB](/azure/cosmos-db/) ist eine global verteilte Datenbank von Microsoft mit mehreren Modellen. Cosmos DB garantiert Wartezeiten im einstelligen Millisekundenbereich im 99. Perzentil an jedem Ort der Welt, bietet mehrere gut definierte Konsistenzmodelle zur Optimierung der Leistung und garantiert Hochverfügbarkeit mit Multihostingfunktionen.

Azure Cosmos DB ist schemaunabhängig. Die Lösung indiziert automatisch alle Daten, sodass Sie sich nicht mit der Schema- und Indexverwaltung befassen müssen. Außerdem unterstützt sie nativ mehrere Datenmodelle wie Dokumente, Schlüssel-Wert-Paare, Diagramme und spaltenbasierte Daten. 

Features von Azure Cosmos DB:

- [Georeplikation](/azure/cosmos-db/distribute-data-globally)
- [Flexible Skalierung für Durchsatz und Speicher](/azure/cosmos-db/partition-data) weltweit
- [Fünf wohl definierte Konsistenzebenen](/azure/cosmos-db/consistency-levels)

## <a name="hbase-on-hdinsight"></a>HBase in HDInsight

[Apache HBase](http://hbase.apache.org/) ist eine Open-Source-NoSQL-Datenbank, die auf Hadoop basiert und nach dem Vorbild von Google BigTable erstellt wurde. HBase bietet wahlfreien Zugriff und starke Konsistenz für große Mengen unstrukturierter und teilweise strukturierter Daten in einer schemalosen Datenbank, die nach Spaltenfamilien gegliedert ist.

Daten werden in den Zeilen einer Tabelle gespeichert und die Daten in einer Zeile zu einer Spaltenfamilie zusammengefasst. HBase ist insofern schemalos, als weder die Spalten noch der Typ der darin gespeicherten Daten vor der Verwendung definiert werden müssen. Der Open-Source-Code lässt sich linear skalieren, sodass Petabytes von Daten auf Tausenden von Knoten verarbeitet werden können. HBase nutzt Datenredundanz, Stapelverarbeitung und andere Funktionen, die von verteilten Anwendungen im Hadoop-Ökosystem zur Verfügung gestellt werden.

Die [HDInsight-Implementierung](/azure/hdinsight/hbase/apache-hbase-overview) nutzt die Architektur mit horizontaler Skalierung von HBase für automatisches Sharding von Tabellen, für starke Konsistenz bei Lese- und Schreibvorgängen sowie für automatisches Failover. Die Leistung wird durch speicherinterne Zwischenspeicherung für Lesevorgänge und Schreibvorgänge mit hohem Durchsatz optimiert. In den meisten Fällen sollten Sie den [HBase-Cluster in einem virtuellen Netzwerk erstellen](/azure/hdinsight/hbase/apache-hbase-provision-vnet), damit andere HDInsight-Cluster und Anwendungen direkt auf die Tabellen zugreifen können.

## <a name="key-selection-criteria"></a>Wichtige Auswahlkriterien

Beantworten Sie die folgenden Fragen, um die Auswahl einzuschränken:

- Benötigen Sie verwalteten, cloudbasierten Hochgeschwindigkeitsspeicher für Text- oder Binärdaten? Falls ja, verwenden Sie eine der Dateispeicheroptionen.

- Benötigen Sie Dateispeicher, der für parallele Analyseworkloads und hohen Durchsatz/hohe IOPS optimiert ist? Falls ja, entscheiden Sie sich für eine Option, deren Leistung für Analyseworkloads optimiert ist.

- Müssen Sie unstrukturierte oder teilweise strukturierte Daten in einer schemalosen Datenbank speichern? Falls ja, entscheiden Sie sich für eine der nicht relationalen Optionen. Vergleichen Sie die Optionen für die Indizierung und die Datenbankmodelle. Abhängig von der Art der Daten, die Sie speichern möchten, sind die primären Datenbankmodelle unter Umständen der wichtigste Faktor.

- Können Sie den Dienst in Ihrer Region verwenden? Überprüfen Sie die regionale Verfügbarkeit der einzelnen Azure-Dienste. Weitere Informationen finden Sie unter [Verfügbare Produkte nach Region](https://azure.microsoft.com/regions/services/).

## <a name="capability-matrix"></a>Funktionsmatrix

In den folgenden Tabellen sind die Hauptunterschiede der Funktionen zusammengefasst:

### <a name="file-storage-capabilities"></a>Dateispeicherfunktionen

|  | Azure Data Lake Store | Azure Blob Storage-Container |
| --- | --- | --- |
| Zweck | Optimierter Speicher für Big Data-Analyseworkloads |Universell einsetzbarer Objektspeicher für eine Vielzahl von Speicherszenarien |
| Anwendungsfälle | Batch-, Streaming Analytics- und Machine Learning-Daten wie Protokolldateien, IoT-Daten, Clickstreams, große Datasets | Jede Art von Text- oder Binärdaten, beispielsweise Daten des Anwendungs-Back-Ends, Sicherungsdaten, Medienspeicher für Streaming und universelle Daten |
| Strukturdefinition | Hierarchisches Dateisystem | Objektspeicher mit flachem Namespace |
| Authentifizierung | Basierend auf [Azure Active Directory-Identitäten](/azure/active-directory/active-directory-authentication-scenarios) | Basierend auf gemeinsamen Geheimnissen – [Kontozugriffsschlüssel](/azure/storage/common/storage-create-storage-account#manage-your-storage-account), [Shared Access Signature-Schlüssel](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) und [rollenbasierte Zugriffssteuerung (Role-Based Access Control, RBAC)](/azure/security/security-storage-overview) |
| Authentifizierungsprotokoll | OAuth 2.0. Aufrufe müssen ein gültiges, über Azure Active Directory ausgestelltes JWT (JSON-Webtoken) enthalten. | Hashbasierter Nachrichtenauthentifizierungscode (Hashed Message Authentication Code, HMAC). Aufrufe müssen einen Base64-codierten SHA-256-Hash über einen Teil der HTTP-Anforderung enthalten. |
| Autorisierung | POSIX-Zugriffssteuerungslisten (Access Control Lists, ACLs). Auf Azure Active Directory-Identitäten basierende ACLs können auf Datei- und Ordnerebene festgelegt werden. | Verwenden Sie [Zugriffsschlüssel](/azure/storage/common/storage-create-storage-account#manage-your-storage-account) für die Autorisierung auf Kontoebene. Verwenden Sie [Shared Access Signature-Schlüssel](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) für die Konto-, Container- oder Blobautorisierung. |
| Überwachung | Verfügbar.  |Verfügbar |
| Verschlüsselung ruhender Daten | Transparent, serverseitig | Transparent, serverseitig; clientseitige Verschlüsselung |
| Entwickler-SDKs | .NET, Java, Python, Node.js | .Net, Java, Python, Node.js, C++, Ruby |
| Leistung von Analyseworkloads | Optimierte Leistung für parallele Analyseworkloads, hohen Durchsatz und hohe IOPS | Nicht für Analyseworkloads optimiert. |
| Größenbeschränkungen | Keine Beschränkungen für Kontogrößen, Dateigrößen oder die Anzahl von Dateien. | Die geltenden Einschränkungen sind [hier](/azure/azure-subscription-service-limits#storage-limits) |
| Georedundanz | Lokal redundant (mehrere Kopien von Daten in einer Azure-Region) | Lokal redundant (LRS), global redundant (GRS), global redundant mit Lesezugriff (RA-GRS). Weitere Informationen finden Sie [hier](/azure/storage/common/storage-redundancy) . |

### <a name="nosql-database-capabilities"></a>NoSQL-Datenbankfunktionen

|                                    |                                           Azure Cosmos DB                                           |                                                             HBase in HDInsight                                                             |
|------------------------------------|-----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
|       Primäres Datenbankmodell       |                      Dokumentspeicher, Diagramm, Schlüssel-Wert-Speicherung, Wide Columnstore                      |                                                             Wide Columnstore                                                              |
|         Sekundäre Indizes          |                                                 Ja                                                 |                                                                     Nein                                                                      |
|        SQL-Sprachunterstützung        |                                                 Ja                                                 |                                     Ja (mit dem [Phoenix](http://phoenix.apache.org/)-JDBC-Treiber)                                      |
|            Konsistenz             |                   Stark, begrenzte Veraltung, Sitzung, Präfixkonsistenz, letztlich                   |                                                                   STARK (Strong)                                                                   |
| Native Azure Functions-Integration |                        [Ja](/azure/cosmos-db/serverless-computing-database)                        |                                                                     Nein                                                                      |
|   Automatische globale Verteilung    |                          [Ja](/azure/cosmos-db/distribute-data-globally)                           | Nein. Die [HBase-Clusterreplikation kann so konfiguriert werden](/azure/hdinsight/hbase/apache-hbase-replication), dass sie sich über Regionen mit letztlicher Konsistenz erstreckt. |
|           Preismodell            | Flexibel skalierbare Anforderungseinheiten (Request Units, RUs), die nach Bedarf pro Sekunde berechnet werden; flexibel skalierbarer Speicher |                              Minutenpreise für HDInsight-Cluster (horizontale Skalierung von Knoten), Speicher                               |

