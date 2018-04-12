---
title: Data Warehousing und Data Marts
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 552cdfad2d571c93f83bc1e4ff0d09ac12d0b6a4
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="data-warehousing-and-data-marts"></a>Data Warehousing und Data Marts

Ein Datawarehouse ist ein zentrales, organisatorisches, relationales Repository für integrierte Daten aus einer einzelnen Quelle oder aus mehreren unterschiedlichen Quellen, die sich über mehrere oder alle Themenbereiche erstrecken. Data Warehouses speichern aktuelle und historische Daten und werden zur Erstellung verschiedener Datenberichte und -analysen verwendet.

![Data Warehousing in Azure](../images/data-warehousing.png)

Für ein Data Warehouse vorgesehene Daten werden in regelmäßigen Abständen aus verschiedenen Quellen mit wichtigen Unternehmensdaten extrahiert. Die Daten können im Zuge der Verschiebung formatiert, bereinigt, validiert, zusammengefasst und neu strukturiert werden. Alternativ können die Daten in der niedrigsten Detailebene gespeichert werden – mit aggregierten Ansichten, die im Warehouse zur Berichterstellung zur Verfügung stehen. In beiden Fällen fungiert das Data Warehouse als permanenter Speicherplatz für Daten, die für Berichte, zur Analyse und zum Treffen wichtiger geschäftlicher Entscheidungen unter Verwendung von BI-Tools (Business Intelligence) verwendet werden.

## <a name="data-marts-and-operational-data-stores"></a>Data Marts und Speicher für operative Daten

Die bedarfsgerechte Verwaltung von Daten ist eine komplexe Angelegenheit, und es wird immer seltener ein einzelnes Data Warehouse verwendet, das alle Daten des gesamten Unternehmens darstellt. Stattdessen erstellen Organisationen kleinere, spezifischere Data Warehouses. Diese werden als *Data Marts* bezeichnet und machen die gewünschten Daten für die Analyse verfügbar. Ein Orchestrierungsprozess füllt die Data Marts mit Daten aus einem Speicher für operative Daten. Der Speicher für operative Daten fungiert als Vermittler zwischen dem Quelltransaktionssystem und dem Data Mart. Bei den Daten, die durch den Speicher für operative Daten verwaltet werden, handelt es sich um eine bereinigte Version der Daten aus dem Quelltransaktionssystem und in der Regel um eine Teilmenge der historischen Daten, die vom Data Warehouse oder Data Mart verwaltet werden. 

## <a name="when-to-use-this-solution"></a>Verwendung dieser Lösung

Entscheiden Sie sich für ein Data Warehouse, wenn Sie große Mengen von Daten aus operativen Systemen in ein leicht verständliches, aktuelles und exaktes Format bringen müssen. Data Warehouses müssen nicht die gleiche knappe Datenstruktur verwenden, die ggf. in Ihren operativen Datenbanken/OLTP-Datenbanken zur Anwendung kommt. Sie können Spaltennamen verwenden, die für geschäftliche Benutzer und Analytiker sinnvoll sind, das Schema zur Vereinfachung von Datenbeziehungen umstrukturieren und mehrere Tabellen in einer einzelnen Tabelle zusammenfassen. Dadurch unterstützen Sie Benutzer bei der Erstellung von Ad-hoc-Berichten bzw. bei der Erstellung von Berichten und der Analyse der Daten in BI-Systemen, sodass sie diese Schritte ohne die Hilfe von einem Datenbankadministrator (DBA) oder Datenentwickler ausführen können.

Ziehen Sie die Verwendung eines Data Warehouse in Betracht, wenn historische Daten aus Leistungsgründen von den Quelltransaktionssystemen getrennt bleiben müssen. Data Warehouses bieten einen zentralen Ort mit gängigen Formaten, Schlüsseln, Datenmodellen und Zugriffsmethoden, um den Zugriff auf historische Daten von mehreren Orten zu vereinfachen.

Data Warehouses sind für Lesezugriff optimiert, was eine schnellere Berichterstellung ermöglicht (verglichen mit der Ausführung von Berichten für das Quelltransaktionssystem). Darüber hinaus bieten Data Warehouses folgende Vorteile:

* Alle historischen Daten aus mehreren Quellen können über ein Data Warehouse gespeichert und als alleingültige Quelle genutzt werden.
* Sie können die Qualität der Daten verbessern, indem Sie sie beim Importieren in das Data Warehouse bereinigen sowie präzisere Daten und konsistente Codes und Beschreibungen bereitstellen.
* Berichtstools konkurrieren nicht mit den Quelltransaktionssystemen um Abfrageverarbeitungszyklen. Mit einem Data Warehouse kann sich das Transaktionssystem vorwiegend um die Behandlung von Schreibvorgängen kümmern, während das Data Warehouse den Großteil der Leseanforderungen bewältigt.
* Ein Data Warehouse kann zur Konsolidierung von Daten aus anderer Software verwendet werden.
* Mit Data Mining-Tools lassen sich automatische Methoden auf in Ihrem Warehouse gespeicherte Daten anwenden, um verborgene Muster zu finden.
* Data Warehouses vereinfachen die Bereitstellung von sicherem Zugriff für autorisierte Benutzer sowie die Beschränkung des Zugriffs für andere. Die geschäftlichen Benutzer benötigen keinen Zugriff auf die Quelldaten, wodurch ein potenzieller Angriffsvektor für Transaktionssysteme in der Produktionsumgebung beseitigt wird.
* Data Warehouses vereinfachen die Erstellung von Business Intelligence-Lösungen, die auf den Daten aufbauen (beispielsweise [OLAP-Cubes](online-analytical-processing.md)).

## <a name="challenges"></a>Herausforderungen

Benutzer, die ein Data Warehouse ordnungsgemäß für ihre geschäftlichen Anforderungen konfigurieren möchten, sehen sich unter Umständen mit einigen der folgenden Herausforderungen konfrontiert:

* Zeitaufwand für die ordnungsgemäße Modellierung der Geschäftskonzepte: Dies ist ein wichtiger Aspekt, da Data Warehouses informationsgesteuert sind und der Rest des Projekts auf der Konzeptzuordnung basiert. Dieser Schritt umfasst die Standardisierung geschäftsspezifischer Begriffe und allgemeiner Formate (etwa für Währung und Datumsangaben) sowie die Strukturierung des Schemas in einer Weise, die für geschäftliche Benutzer sinnvoll ist, ohne jedoch die Präzision von Datenaggregaten und Beziehungen zu beeinträchtigen.
* Planung und Einrichtung Ihrer Datenorchestrierung: Hierbei ist zu berücksichtigen, wie Daten aus dem Quelltransaktionssystem in das Data Warehouse kopiert und wann historische Daten aus Ihren Speichern für operative Daten in das Warehouse verschoben werden sollen.
* Gewährleistung oder Optimierung der Datenqualität durch Bereinigung der Daten beim Importieren in das Warehouse

## <a name="data-warehousing-in-azure"></a>Data Warehousing in Azure

In Azure können einzelne oder mehrere Datenquellen vorhanden sein, die entweder auf Kundentransaktionen oder auf verschiedenen Geschäftsanwendungen basieren, die von unterschiedlichen Abteilungen verwendet werden. Diese Daten sind üblicherweise in mindestens einer [OLTP](online-transaction-processing.md)-Datenbank gespeichert. Die Daten können auf anderen Speichermedien wie Netzwerkfreigaben, in Azure Storage-Blobs oder in einem Data Lake gespeichert werden. Die Daten können auch vom Data Warehouse selbst oder in einer relationalen Datenbank wie Azure SQL-Datenbank gespeichert werden. Die Analysedatenspeicher-Ebene dient zum Abwickeln von Abfragen, die von Analyse- und Berichtstools für das Data Warehouse oder für den Data Mart ausgegeben werden. In Azure kann für diese Analysespeicherfunktion Azure SQL Data Warehouse oder Azure HDInsight mit Hive oder Interactive Query verwendet werden. Darüber hinaus benötigen Sie ein gewisses Maß an Orchestrierung, um Daten in regelmäßigen Abständen aus dem Datenspeicher in das Data Warehouse zu verschieben oder zu kopieren. Hierzu können Sie Azure Data Factory oder Oozie in Azure HDInsight verwenden.

Ein Data Warehouse kann in Azure auf verschiedene Arten implementiert werden. Für welche Option Sie sich entscheiden, hängt ganz von Ihren Anforderungen ab. Die folgenden Listen sind in zwei Kategorien unterteilt: [symmetrisches Multiprocessing](https://en.wikipedia.org/wiki/Symmetric_multiprocessing) (SMP) und [Massively Parallel Processing](https://en.wikipedia.org/wiki/Massively_parallel) (MPP). 

SMP:

- [Azure SQL-Datenbank](/azure/sql-database/)
- [SQL Server auf einem virtuellen Computer](/sql/sql-server/sql-server-technical-documentation)

MPP:

- [Azure Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [Apache Hive in HDInsight](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [Interactive Query (Hive LLAP) in HDInsight](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

Allgemein gilt: SMP-basierte Warehouses eignen sich am besten für kleine bis mittelgroße Datasets (4 bis 100 TB), während MPP häufig für Big Data verwendet wird. Die Abgrenzung zwischen kleinen/mittelgroßen Daten und Big Data hängt zum Teil mit der Definition und der unterstützenden Infrastruktur Ihres Unternehmens zusammen. (Weitere Informationen finden Sie unter [Choosing an OLTP data store](online-transaction-processing.md#scalability-capabilities) (Auswählen eines OLTP-Datenspeichers).) 

Abgesehen davon hat die Art des Workloadmusters wahrscheinlich einen größeren Einfluss auf die Entscheidung als die Datengröße. So können beispielsweise komplexe Abfragen für eine SMP-Lösung zu langsam sein und die Verwendung einer MPP-Lösung erforderlich machen. Bei MPP-basierten Systemen ist mit Leistungseinbußen bei geringen Datengrößen zu rechnen, was auf die knotenübergreifende Verteilung und Konsolidierung der Aufträge zurückzuführen ist. Wenn die Größe Ihrer Daten bereits 1 TB übersteigt und voraussichtlich weiter zunimmt, empfiehlt sich die Verwendung einer MPP-Lösung. Und auch wenn Ihre Daten kleiner sind, Ihre Workloads aber die verfügbaren Ressourcen Ihrer SMP-Lösung übersteigen, ist MPP wahrscheinlich die beste Wahl.

Die Daten, auf die Ihr Data Warehouse zugreift bzw. die von Ihrem Data Warehouse gespeichert werden, können aus verschiedensten Datenquellen stammen – unter anderem auch aus einem Data Lake wie [Azure Data Lake Store](/azure/data-lake-store/). Unter [Azure Data Lake and Azure Data Warehouse: Applying Modern Practices to Your App](https://azure.microsoft.com/resources/videos/build-2016-azure-data-lake-and-azure-data-warehouse-applying-modern-practices-to-your-app/) (Azure Data Lake und Azure Data Warehouse: moderne Methoden für Ihre App) finden Sie ein Video, in dem die verschiedenen Stärken von für Azure Data Lake geeigneten MPP-Diensten miteinander verglichen werden.

SMP-Systeme zeichnen sich durch eine einzelne Instanz eines Managementsystems für relationale Datenbanken aus, die sämtliche Ressourcen (CPU/Arbeitsspeicher/Datenträger) gemeinsam nutzen. Ein SMP-System kann zentral hochskaliert werden. Für SQL Server auf einem virtuellen Computer können Sie die VM-Größe zentral hochskalieren. Für Azure SQL-Datenbank können Sie zum zentralen Hochskalieren einen anderen Diensttarif auswählen. 

MPP-Systeme können durch Hinzufügen weiterer Computeknoten (mit eigener CPU, eigenem Arbeitsspeicher und eigenen E/A-Subsystemen) horizontal hochskaliert werden. Aufgrund physischer Einschränkungen kann ein Server nur bis zu einem gewissen Punkt zentral hochskaliert werden. Danach ist abhängig von der Workload horizontales Hochskalieren vorzuziehen. Für MPP-Lösungen werden aufgrund von Abweichungen bei Abfragen, Modellierung, Datenpartitionierung und anderen speziellen Aspekten der parallelen Verarbeitung jedoch andere Kompetenzen vorausgesetzt. 

Der Abschnitt [Genauere Betrachtung von Azure SQL-Datenbank und SQL Server auf Azure Virtual Machines](/azure/sql-database/sql-database-paas-vs-sql-server-iaas#a-closer-look-at-azure-sql-database-and-sql-server-on-azure-vms) enthält hilfreiche Informationen zur Wahl einer SMP-Lösung. 

Azure SQL Data Warehouse kann auch für kleine und mittelgroße Datasets mit rechen- und speicherintensiven Workloads verwendet werden. Weitere Informationen zu SQL Data Warehouse-Mustern und gängigen Szenarien finden Sie unter den folgenden Link:

- [SQL Data Warehouse Patterns and Anti-Patterns](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/) (SQL Data Warehouse: Muster und Antimuster)
- [SQL Data Warehouse Loading Patterns and Strategies](https://blogs.msdn.microsoft.com/sqlcat/2017/05/17/azure-sql-data-warehouse-loading-patterns-and-strategies/) (SQL Data Warehouse: Lademuster und -strategien)
- [Migrieren von Daten zu Azure SQL Data Warehouse](https://blogs.msdn.microsoft.com/sqlcat/2016/08/18/migrating-data-to-azure-sql-data-warehouse-in-practice/)
- [Common ISV application patterns using Azure SQL Data Warehouse](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/common-isv-application-patterns-using-azure-sql-data-warehouse/) (Gängige ISV-Anwendungsmuster mit Azure SQL Data Warehouse)

## <a name="key-selection-criteria"></a>Wichtige Auswahlkriterien

Beantworten Sie die folgenden Fragen, um die Auswahl einzuschränken:

- Möchten Sie einen verwalteten Dienst verwenden, anstatt Ihre eigenen Server zu verwalten?

- Arbeiten Sie mit sehr großen Datasets oder mit sehr komplexen Abfragen mit langer Ausführungszeit? Falls ja, empfiehlt sich die Verwendung einer MPP-Option. 

- Ist die Datenquelle bei einem großen Dataset strukturiert oder unstrukturiert? Unstrukturierte Daten müssen ggf. in einer Big Data-Umgebung wie Spark in HDInsight, Azure Databricks, Hive LLAP in HDInsight oder Azure Data Lake Analytics verarbeitet werden. Alle diese Optionen können als ELT-Modul (Extrahieren, Laden, Transformieren) sowie als ETL-Modul (Extrahieren, Transformieren, Laden) fungieren. Die verarbeiteten Daten können als strukturierte Daten ausgegeben werden, um sie leichter in SQL Data Warehouse oder in eine der anderen Optionen laden zu können. Für strukturierte Daten bietet SQL Data Warehouse die Leistungsstufe „Optimiert für Compute“, die für rechenintensive Workloads mit sehr hohen Leistungsanforderungen konzipiert ist.

- Möchten Sie Ihre historischen Daten und Ihre aktuellen operativen Daten trennen? Falls ja, wählen Sie eine der Optionen, die eine [Orchestrierung](../technology-choices/pipeline-orchestration-data-movement.md) erfordern. Hierbei handelt es sich um eigenständige Warehouses, die für intensiven Lesezugriff optimiert und am besten als separater Speicher für historische Daten geeignet sind.

- Müssen Sie Daten aus mehreren Quellen (abgesehen von Ihrem OLTP-Datenspeicher) integrieren? Falls ja, empfiehlt sich die Verwendung einer Option mit einfacher Integration mehrerer Datenquellen. 

- Sind Sie auf Mehrinstanzenfähigkeit angewiesen? Falls ja, ist SQL Data Warehouse nicht die ideale Lösung. Weitere Informationen finden Sie unter [SQL Data Warehouse Patterns and Anti-Patterns](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/) (SQL Data Warehouse: Muster und Antimuster).

- Bevorzugen Sie einen relationalen Datenspeicher? Falls ja, grenzen Sie Ihre Optionen auf Lösungen mit einem relationalen Datenspeicher ein. Bedenken Sie dabei jedoch, dass Sie nicht relationale Datenspeicher bei Bedarf mithilfe eines Tools wie PolyBase abfragen können. Sollten Sie sich für die Verwendung von PolyBase entscheiden, führen Sie anhand Ihrer unstrukturierte Datasets Leistungstests für Ihre Workload aus.

- Benötigen Sie Echtzeitberichte? Falls Sie bei Abfragen mit vielen Singleton-Einfügungen auf schnelle Antworten angewiesen sind, grenzen Sie Ihre Optionen auf Lösungen ein, die Echtzeitberichte unterstützen.

- Müssen Sie eine große Anzahl von gleichzeitigen Benutzern und Verbindungen unterstützen? Die Unterstützung einer Reihe von gleichzeitigen Benutzern/Verbindungen hängt von mehreren Faktoren ab. 

    - Informieren Sie sich für Azure SQL-Datenbank über die [dokumentierten Ressourcenlimits](/azure/sql-database/sql-database-resource-limits) für Ihre Dienstebene. 
    
    - Bei SQL Server sind maximal 32.767 Benutzerverbindungen zulässig. Bei Ausführung auf einem virtuellen Computer hängt die Leistung von der Größe des virtuellen Computers sowie von anderen Faktoren ab. 
    
    - Bei SQL Data Warehouse ist die Anzahl gleichzeitiger Abfragen und gleichzeitiger Verbindungen beschränkt. Weitere Informationen finden Sie unter [Parallelitäts- und Workloadverwaltung in SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-develop-concurrency). Die Einschränkungen von SQL Data Warehouse lassen sich bei Bedarf mit zusätzlichen Diensten wie [Azure Analysis Services](/azure/analysis-services/analysis-services-overview) kompensieren.

- Welche Art von Workload verwenden Sie? Im Allgemeinen eignen sich MPP-basierte Warehouse-Lösungen am besten für analytische, batchorientierte Workloads. Wenn es sich bei Ihrer Workload um eine Transaktionsworkload mit zahlreichen kleinen Lese-/Schreibvorgängen oder mehreren zeilenweisen Vorgängen handelt, empfiehlt sich die Verwendung einer SMP-Option. Hiervon ausgenommen sind Szenarien, in denen eine Streamverarbeitung für einen HDInsight-Cluster (beispielsweise Spark Streaming) verwendet wird und die Daten in einer Hive-Tabelle gespeichert werden.

## <a name="capability-matrix"></a>Funktionsmatrix

In den folgenden Tabellen sind die Hauptunterschiede der Funktionen zusammengefasst:

### <a name="general-capabilities"></a>Allgemeine Funktionen

| | Azure SQL-Datenbank | SQL Server (VM) | SQL Data Warehouse | Apache Hive in HDInsight | Hive LLAP in HDInsight |
| --- | --- | --- | --- | --- | --- | -- |
| Verwalteter Dienst | Ja | Nein | Ja | Ja<sup>1</sup> | Ja<sup>1</sup> |
| Erfordert Datenorchestrierung (Speicherung von Datenkopie/historischen Daten) | Nein  | Nein  | Ja | Ja | Ja |
| Einfache Integration mehrerer Datenquellen | Nein  | Nein  | Ja | Ja | Ja |
| Unterstützung des Anhaltens von Computevorgängen | Nein  | Nein  | Ja | Nein<sup>2</sup> | Nein<sup>2</sup> |
| Relationaler Datenspeicher | Ja | Ja |  Ja | Nein  | Nein  |
| Echtzeitberichte | Ja | Ja | Nein  | Nein  | Ja |
| Flexible Sicherungswiederherstellungspunkte | Ja | Ja | Nein<sup>3</sup> | Ja<sup>4</sup> | Ja<sup>4</sup> |
| SMP/MPP | SMP | SMP | MPP | MPP | MPP |

[1] Manuelle Konfiguration und Skalierung.

[2] Nicht benötigte HDInsight-Cluster können gelöscht und später erneut erstellt werden. Fügen Sie an Ihren Cluster einen externen Datenspeicher an, damit Ihre Daten beim Löschen des Clusters erhalten bleiben. Zur Automatisierung des Clusterlebenszyklus können Sie mithilfe von Azure Data Factory einen bedarfsgesteuerten HDInsight-Cluster für die Verarbeitung Ihrer Workload erstellen und ihn nach Abschluss der Verarbeitung wieder löschen.

[3] Mit SQL Data Warehouse können Sie eine Datenbank auf der Grundlage eines beliebigen verfügbaren Wiederherstellungspunkts der letzten sieben Tage wiederherstellen. Momentaufnahmen werden alle vier bis acht Stunden gestartet und sind sieben Tage lang verfügbar. Wenn eine Momentaufnahme älter als sieben Tage ist, läuft sie ab, und ihr Wiederherstellungspunkt ist nicht mehr verfügbar.

[4] Verwenden Sie ggf. einen [externen Hive-Metastore](/azure/hdinsight/hdinsight-hadoop-provision-linux-clusters#use-hiveoozie-metastore), der nach Bedarf gesichert und wiederhergestellt werden kann. Für die Daten können die standardmäßigen Sicherungs- und Wiederherstellungsoptionen für Blob Storage oder Data Lake Store verwendet werden. Wenn Sie mehr Flexibilität und Komfort benötigen, können Sie aber auch HDInsight-Sicherungs- und Wiederherstellungslösungen von Drittanbietern (beispielsweise [Imanis Data](https://azure.microsoft.com/blog/imanis-data-cloud-migration-backup-for-your-big-data-applications-on-azure-hdinsight/)) verwenden.

### <a name="scalability-capabilities"></a>Skalierbarkeitsfunktionen

| | Azure SQL-Datenbank | SQL Server (VM) |  SQL Data Warehouse | Apache Hive in HDInsight | Hive LLAP in HDInsight |
| --- | --- | --- | --- | --- | --- | -- |
| Redundante regionale Server für Hochverfügbarkeit  | Ja | Ja | Ja | Nein  | Nein  |
| Unterstützung des horizontalen Hochskalierens für Abfragen (verteilte Abfragen)  | Nein  | Nein  | Ja | Ja | Ja |
| Dynamische Skalierbarkeit (zentrales Hochskalieren)  | Ja | Nein  | Ja<sup>1</sup> | Nein  | Nein  |
| Unterstützung der speicherinternen Zwischenspeicherung von Daten | Ja |  Ja | Nein | Ja | Ja |

[1] SQL Data Warehouse ermöglicht zentrales Hoch- und Herunterskalieren durch Anpassung der Anzahl von Data Warehouse-Einheiten (Data Warehouse Units, DWUs). Weitere Informationen finden Sie unter [Verwalten von Computeleistung in Azure SQL Data Warehouse (Übersicht)](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview).

### <a name="security-capabilities"></a>Sicherheitsfunktionen

|                         |           Azure SQL-Datenbank            |  SQL Server auf einem virtuellen Computer  | SQL Data Warehouse |   Apache Hive in HDInsight    |    Hive LLAP in HDInsight     |
|-------------------------|-----------------------------------------|-----------------------------------|--------------------|-------------------------------|-------------------------------|
|     Authentifizierung      | SQL/Azure Active Directory (Azure AD) | SQL/Azure AD/Active Directory |   SQL/Azure AD   | Lokal/Azure AD <sup>1</sup> | Lokal/Azure AD <sup>1</sup> |
|      Autorisierung      |                   Ja                   |                Ja                |        Ja         |              Ja              |       Ja<sup>1</sup>        |
|        Überwachung         |                   Ja                   |                Ja                |        Ja         |              Ja              |       Ja<sup>1</sup>        |
| Datenverschlüsselung ruhender Daten |            Ja<sup>2</sup>             |         Ja<sup>2</sup>          |  Ja<sup>2</sup>  |       Ja<sup>2</sup>        |       Ja<sup>1</sup>        |
|   Sicherheit auf Zeilenebene    |                   Ja                   |                Ja                |        Ja         |              Nein                |       Ja<sup>1</sup>        |
|   Unterstützung von Firewalls    |                   Ja                   |                Ja                |        Ja         |              Ja              |       Ja<sup>3</sup>        |
|  Dynamische Datenmaskierung   |                   Ja                   |                Ja                |        Ja         |              Nein                |       Ja<sup>1</sup>        |

[1] Erfordert die Verwendung eines [in die Domäne eingebundenen HDInsight-Clusters](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] Erfordert die Verwendung von Transparent Data Encryption (TDE) zum Verschlüsseln und Entschlüsseln ruhender Daten.

[3] Unterstützt bei [Verwendung in einem virtuellen Azure-Netzwerk](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)

Weitere Informationen zum Schutz Ihres Data Warehouse finden Sie hier:

* [Sichern der SQL-Datenbank](/azure/sql-database/sql-database-security-overview#connection-security)
* [Sichern einer Datenbank in SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-manage-security)
* [Erweitern von Azure HDInsight per Azure Virtual Network](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)
* [Einführung in die Hadoop-Sicherheit mit in die Domäne eingebundenen HDInsight-Clustern](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)

