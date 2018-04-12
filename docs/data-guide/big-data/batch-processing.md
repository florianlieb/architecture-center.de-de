---
title: Batchverarbeitung
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: d6843bf4e20c3eb26e61cfa09300ad533e969c2e
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/03/2018
---
# <a name="batch-processing"></a>Batchverarbeitung

Ein verbreitetes Big Data-Szenario ist die Batchverarbeitung ruhender Daten. In diesem Szenario werden die Quelldaten in einen Datenspeicher geladen – entweder durch die Quellanwendung selbst oder durch einen Orchestrierungsworkflow. Anschließend werden die Daten direkt durch einen parallelisierten Auftrag verarbeitet. Dieser Auftrag kann ebenfalls durch den Orchestrierungsworkflow initiiert werden. Die Verarbeitung kann mehrere iterative Schritte umfassen. Die transformierten Ergebnisse werden schließlich in einen Analysedatenspeicher geladen, der von Analyse- und Berichterstellungskomponenten abgefragt werden kann.

So können beispielsweise die Protokolle eines Webservers in einen Ordner kopiert und über Nacht verarbeitet werden, um tägliche Berichte zur Webaktivität zu generieren.

![](./images/batch-pipeline.png)

## <a name="when-to-use-this-solution"></a>Verwendung dieser Lösung

Die Batchverarbeitung kommt in verschiedensten Szenarien zum Einsatz – von der einfachen Datentransformation bis hin zur umfassenderen ETL-Pipeline (Extrahieren, Transformieren, Laden). In einem Big Data-Kontext kann die Batchverarbeitung auf sehr große Datasets angewendet werden, bei denen die Berechnung viel Zeit in Anspruch nimmt. (Weitere Informationen finden Sie beispielsweise unter [Lambda-Architektur](../big-data/index.md#lambda-architecture).) Die Batchverarbeitung führt in der Regel zu weiteren interaktiven Untersuchungen, stellt die modellierungsbereiten Daten für Machine Learning bereit oder schreibt die Daten in einen für die Analyse und Visualisierung optimierten Datenspeicher.

Ein Beispiel für die Batchverarbeitung ist die Transformation zahlreicher teilweise strukturierter CSV- oder JSON-Flatfiles in ein schematisiertes und strukturiertes Format, das für weitere Abfragen genutzt werden kann. Die Daten werden in der Regel aus den bei der Erfassung verwendeten Rohformaten (beispielsweise CSV) in Binärformate konvertiert, mit denen sich bei Abfragen eine bessere Leistung erzielen lässt, da sie Daten in einem spaltenförmigen Format speichern und häufig Indizes und Inlinestatistiken für die Daten bereitstellen.

## <a name="challenges"></a>Herausforderungen

- **Datenformat und -codierung**: Einige der kompliziertesten Probleme treten bei Dateien mit unerwartetem Format oder unerwarteter Codierung auf. So können Quelldateien beispielsweise eine Mischung aus UTF-16- und UTF-8-Codierung, unerwartete Trennzeichen (Leerzeichen anstelle von Tabstopps) oder unerwartete Zeichen enthalten. Ein weiteres Beispiel sind Textfelder mit Tabstopps, Leerzeichen oder Kommas, die als Trennzeichen interpretiert werden. Die Logik für das Laden und Analysieren der Daten muss über die nötige Flexibilität verfügen, um diese Probleme erkennen und bewältigen zu können.

- **Orchestrieren von Zeitsegmenten**: Quelldaten werden häufig in einer Ordnerhierarchie platziert, die Verarbeitungsfenster darstellt, welche nach Jahr, Monat, Tag, Stunde usw. strukturiert sind. Manchmal treffen Daten unter Umständen verspätet ein. Ein Beispiel: Angenommen, ein Webserver fällt aus, und die Protokolle für den 7. März landen erst am 9. März in dem Ordner für die Verarbeitung. Werden sie einfach ignoriert, da sie zu spät eingetroffen sind? Kommt die Downstreamverarbeitungslogik mit außerordentlichen Datensätzen zurecht?

## <a name="architecture"></a>Architecture

Eine Batchverarbeitungsarchitektur verfügt über folgende logische Komponenten, die auch im obigen Diagramm dargestellt sind:

- **Datenspeicher**: In der Regel ein verteilter Datenspeicher, der als Repository für zahlreiche große Dateien in verschiedenen Formaten fungieren kann. Diese Art von Speicher wird allgemein häufig als Data Lake bezeichnet. 

- **Batchverarbeitung**: Aufgrund des großen Umfangs von Big Data müssen Lösungen oftmals Datendateien mithilfe von Batchaufträgen mit langer Ausführungszeit verarbeiten, um die Daten zu filtern, zu aggregieren und anderweitig auf die Analyse vorzubereiten. Diese Aufträge beinhalten in der Regel das Lesen von Quelldateien, ihre Verarbeitung und das Schreiben der Ausgabe in neue Dateien. 

- **Analysedatenspeicher**: Viele Big Data-Lösungen bereiten Daten für die Analyse vor und stellen die verarbeiteten Daten dann in einem strukturierten Format bereit, das mithilfe von Analysetools abgefragt werden kann. 

- **Analysen und Berichte**: Ziel der meisten Big Data-Lösungen ist es, über Analysen und Berichte Einblicke in die Daten zu bieten. 

- **Orchestrierung**: In Verbindung mit der Batchverarbeitung ist in der Regel auch ein gewisses Maß an Orchestrierung erforderlich, um die Daten in Ihren Datenspeicher, in die Batchverarbeitung, in den Analysedatenspeicher und in die Berichterstellungsebenen zu kopieren bzw. zu migrieren.

## <a name="technology-choices"></a>Auswahl der Technologie

Für Batchverarbeitungslösungen in Azure werden folgende Technologien empfohlen:

### <a name="data-storage"></a>Datenspeicher

- **Azure Storage Blob-Container**: Da Azure Blob Storage bereits von vielen Azure-Geschäftsprozessen genutzt wird, ist diese Option eine gute Wahl für einen Big Data-Speicher.
- **Azure Data Lake Store**. Der nahezu unbegrenzte Speicher für jegliche Dateigröße sowie umfassende Sicherheitsoptionen machen Azure Data Lake Store zu einer guten Wahl für besonders umfangreiche Big Data-Lösungen, die einen zentralen Speicher für Daten in heterogenen Formaten benötigen.

Weitere Informationen finden Sie im Artikel zur [Datenspeicherung](../technology-choices/data-storage.md).

### <a name="batch-processing"></a>Batchverarbeitung

- **U-SQL**: U-SQL ist die von Azure Data Lake Analytics verwendete Abfrageverarbeitungssprache. Sie kombiniert den deklarativen Charakter von SQL mit der prozeduralen Erweiterbarkeit von C# und den Vorteilen der Parallelität, um eine effiziente Verarbeitung umfangreicher Daten zu ermöglichen.
- **Hive**: Hive ist eine SQL-ähnliche Sprache, die von den meisten Hadoop-Distributionen (einschließlich HDInsight) unterstützt wird. Sie kann zum Verarbeiten von Daten aus einem beliebigen HDFS-kompatiblen Speicher (einschließlich Azure Blob Storage und Azure Data Lake Store) verwendet werden.
- **Pig**: Pig ist eine deklarative Big Data-Verarbeitungssprache, die in vielen Hadoop-Distributionen (einschließlich HDInsight) zum Einsatz kommt. Sie eignet sich besonders gut für die Verarbeitung unstrukturierter oder teilweise strukturierter Daten.
- **Spark**: Das Spark-Modul unterstützt Batchverarbeitungsprogramme in verschiedenen Programmiersprachen (beispielsweise Python, Java und Scala). Spark verwendet eine verteilte Architektur, um Daten parallel auf mehreren Workerknoten zu verarbeiten.

Weitere Informationen finden Sie im Artikel zur [Batchverarbeitung](../technology-choices/batch-processing.md).

### <a name="analytical-data-store"></a>Analysedatenspeicher

- **SQL Data Warehouse**: Azure SQL Data Warehouse ist ein verwalteter, auf SQL Server-Datenbanktechnologien basierender Dienst und für die Unterstützung umfangreicher Data Warehousing-Workloads optimiert.
- **Spark SQL**: Spark SQL ist eine API, die auf Spark basiert und die Erstellung von Datenrahmen und Tabellen unterstützt, für die das Abfragen per SQL-Syntax möglich ist.
- **HBase**: HBase ist ein NoSQL-Datenspeicher mit geringer Wartezeit und einer flexiblen Hochleistungsoption für die Abfrage strukturierter und teilweise strukturierter Daten.
- **Hive**: Hive ist nicht nur bei der Batchverarbeitung hilfreich, sondern bietet auch eine Datenbankarchitektur, die ähnlich konzeptioniert ist wie ein typisches Managementsystem für relationale Datenbanken. Dank Verbesserungen der Hive-Abfrageleistung durch Innovationen wie dem Tez-Modul und der Stinger-Initiative können Hive-Tabellen in einigen Szenarien effektiv als Quellen für analytische Abfragen verwendet werden.

Weitere Informationen finden Sie im Artikel zu [Analysedatenspeichern](../technology-choices/analytical-data-stores.md).

### <a name="analytics-and-reporting"></a>Analysen und Berichte

- **Azure Analysis Services**: Viele Big Data-Lösungen emulieren herkömmliche Business Intelligence-Architekturen für Unternehmen durch die Einbeziehung eines zentralisierten OLAP-Datenmodells (Online Analytical Processing, analytische Onlineverarbeitung; häufig als Cube bezeichnet), das als Grundlage für Berichte, Dashboards und interaktive Analysen („Slice and Dice“) verwendet werden kann. Azure Analysis Services unterstützt die Erstellung mehrdimensionaler und tabellarischer Modelle, um diese Anforderung zu erfüllen.
- **Power BI**: Mit Power BI können Datenanalysten interaktive Datenvisualisierungen auf der Grundlage von Datenmodellen in einem OLAP-Modell oder direkt auf der Grundlage eines Analysedatenspeichers erstellen.
- **Microsoft Excel**: Microsoft Excel ist eine der meistverwendeten Softwareanwendungen der Welt und bietet ein breites Spektrum an Funktionen für die Analyse und Visualisierung von Daten. Mit Excel können Datenanalysten Dokumentdatenmodelle auf der Grundlage von Analysedatenspeichern erstellen oder Daten aus OLAP-Datenmodellen in interaktive PivotTables und Diagramme abrufen.

Weitere Informationen finden Sie im Artikel zu [Analysen und Berichten](../technology-choices/analysis-visualizations-reporting.md).

### <a name="orchestration"></a>Orchestrierung

- **Azure Data Factory** Mit Azure Data Factory-Pipelines kann eine Abfolge von Aktivitäten definiert und für wiederkehrende temporale Fenster geplant werden. Diese Aktivitäten können Datenkopiervorgänge sowie Hive-, Pig-, MapReduce- oder Spark-Aufträge in bedarfsgesteuerten HDInsight-Clustern, U-SQL-Aufträge in Azure Data Lake Analytics und gespeicherte Prozeduren in Azure SQL Data Warehouse oder Azure SQL-Datenbank initiieren.
- **Oozie** und **Sqoop**: Oozie ist ein Auftragsautomatisierungsmodul für das Apache Hadoop-Ökosystem und kann verwendet werden, um Datenkopiervorgänge und Hive-, Pig- und MapReduce-Aufträge zum Verarbeiten von Daten sowie Sqoop-Aufträge zum Kopieren von Daten zwischen HDFS und SQL-Datenbanken zu initiieren.

Weitere Informationen finden Sie im Artikel zur [Pipelineorchestrierung](../technology-choices/pipeline-orchestration-data-movement.md).
