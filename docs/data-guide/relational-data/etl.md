---
title: Extrahieren, Transformieren und Laden (ETL)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 1879b649fa3dfdf5c00f8ee30e53b83f7139fbf0
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/03/2018
---
# <a name="extract-transform-and-load-etl"></a>Extrahieren, Transformieren und Laden (ETL)

Viele Organisationen fragen sich, wie sie Daten aus mehreren Datenquellen und in verschiedenen Formaten erfassen und in einzelne oder mehrere Datenspeicher verschieben können. Das Ziel ist möglicherweise nicht die gleiche Art von Datenspeicher wie die Quelle, und häufig unterscheidet sich auch das Format, oder die Daten müssen vor dem Laden in das endgültige Ziel strukturiert oder bereinigt werden.

Zur Bewältigung dieser Herausforderungen wurden im Laufe der Jahre verschiedene Tools, Dienste und Prozesse entwickelt. Unabhängig vom verwendeten Prozess muss im Allgemeinen die Arbeit koordiniert und ein gewisses Maß an Datentransformation innerhalb der Datenpipeline angewendet werden. In den folgenden Abschnitten werden gängige Methoden für die Durchführung dieser Aufgaben erläutert.

## <a name="extract-transform-and-load-etl"></a>Extrahieren, Transformieren und Laden (ETL)

Extrahieren, Transformieren und Laden (ETL) ist eine Datenpipeline und wird verwendet, um Daten aus verschiedenen Quellen zu sammeln, gemäß den Geschäftsregeln zu transformieren und in einen Zieldatenspeicher zu laden. Die Transformation in ETL findet in einem speziellen Modul statt und beinhaltet häufig die Verwendung von Stagingtabellen zur vorübergehenden Speicherung von Daten, während diese transformiert und schließlich in das Ziel geladen werden.

Die durchgeführte Datentransformation umfasst in der Regel verschiedene Vorgänge wie Filtern, Sortieren, Aggregieren, Verknüpfen, Bereinigen, Deduplizieren und Validieren von Daten.

![ETL-Prozess (Extrahieren, Transformieren, Laden)](../images/etl.png)

Die drei ETL-Phasen werden häufig parallel ausgeführt, um Zeit zu sparen. Während der Datenextraktion kann also beispielsweise ein Transformationsprozess für bereits empfangene Daten ausgeführt werden, um die Daten für das Laden vorzubereiten, und ein Ladevorgang kann mit der Arbeit an den vorbereiteten Daten beginnen, anstatt auf den Abschluss des gesamten Extraktionsprozesses zu warten.

In Frage kommender Azure-Dienst:
- [Azure Data Factory v2](https://azure.microsoft.com/services/data-factory/)

Weitere Tools:
- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

## <a name="extract-load-and-transform-elt"></a>Extrahieren, Laden und Transformieren (ELT)

Extrahieren, Laden und Transformieren (ELT) unterscheidet sich von ETL lediglich darin, an welcher Stelle die Transformation erfolgt. In der ELT-Pipeline findet die Transformation im Zieldatenspeicher statt. Die Daten werden mithilfe der Verarbeitungsfunktionen des Zieldatenspeichers transformiert, anstatt ein separates Transformationsmodul zu verwenden. Dadurch wird das Transformationsmodul aus der Pipeline entfernt, was die Architektur vereinfacht. Ein weiterer Vorteil dieses Ansatzes ist, dass durch Skalieren des Zieldatenspeichers auch die Leistung der ELT-Pipeline skaliert wird. ELT setzt jedoch voraus, dass das Zielsystem über genügend Leistung verfügt, um die Daten effizient transformieren zu können.

![ELT-Prozess (Extrahieren, Laden, Transformieren)](../images/elt.png)

ELT kommt üblicherweise in Big Data-Szenarien zum Einsatz. So können Sie beispielsweise zunächst alle Quelldaten in Flatfiles in einem skalierbaren Speicher wie HDFS (Hadoop Distributed File System) oder Azure Data Lake Store extrahieren. Anschließend können Sie die Quelldaten mit Technologien wie Spark, Hive oder PolyBase abfragen. Entscheidend bei ELT ist, dass der für die Transformation verwendete Datenspeicher der gleiche Datenspeicher ist, in dem die Daten später auch genutzt werden. Dieser Datenspeicher liest direkt aus dem skalierbaren Speicher, anstatt die Daten in seinen eigenen proprietären Speicher zu laden. Bei diesem Ansatz wird also der Datenkopierschritt aus ELT übersprungen, der bei umfangreichen Datasets sehr zeitaufwendig sein kann.

In der Praxis ist der Zieldatenspeicher ein [Data Warehouse](./data-warehousing.md) mit einem Hadoop-Cluster (Hive oder Spark) oder ein SQL Data Warehouse. Im Allgemeinen wird bei der Abfrage ein Schema auf die Flatfiledaten angewendet und als Tabelle gespeichert, wodurch die Daten wie jede andere Tabelle im Datenspeicher abgefragt werden können. Diese Tabellen werden als externe Tabellen bezeichnet, da sich die Daten nicht in dem Speicher befinden, der vom Datenspeicher selbst verwaltet wird, sondern in einem externen skalierbaren Speicher. 

Der Datenspeicher verwaltet nur das Schema der Daten und wendet es beim Lesen an. Ein Hadoop-Cluster mit Hive beschreibt beispielsweise eine Hive-Tabelle, in der die Datenquelle im Grunde ein Pfad zu einer Gruppe von Dateien in HDFS ist. In SQL Data Warehouse lässt sich mit PolyBase das gleiche Ergebnis erzielen: Es wird eine Tabelle für Daten erstellt, die außerhalb der eigentlichen Datenbank gespeichert sind. Nach dem Laden der Quelldaten können die in externen Tabellen enthaltenen Daten mithilfe der Funktionen des Datenspeichers verarbeitet werden. In Big Data-Szenarien muss der Datenspeicher für MPP (Massively Parallel Processing) geeignet sein. Dabei werden die Daten in kleinere Blöcke aufgeteilt, die dann parallel von mehreren Computern verarbeitet werden.

In der letzten Phase der ELT-Pipeline werden die Quelldaten üblicherweise in ein endgültiges Format transformiert, das besser für die Arten von Abfragen geeignet ist, die unterstützt werden müssen. So können die Daten beispielsweise partitioniert werden. ELT kann außerdem optimierte Speicherformate wie Parquet verwendet, das zeilenorientierte Daten spaltenförmig speichert und eine optimierte Indizierung bietet. 

In Frage kommender Azure-Dienst:

- [Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [HDInsight mit Hive](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [Azure Data Factory v2](https://azure.microsoft.com/services/data-factory/)
- [Oozie in HDInsight](/azure/hdinsight/hdinsight-use-oozie-linux-mac)

Weitere Tools:

- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

## <a name="data-flow-and-control-flow"></a>Datenfluss und Ablaufsteuerung

Im Datenpipelinekontext sorgt die Ablaufsteuerung für eine geordnete Verarbeitung einer Reihe von Tasks. Zur Erzwingung der korrekten Verarbeitungsreihenfolge dieser Tasks werden Rangfolgeneinschränkungen verwendet. Diese Einschränkungen können Sie sich als Connectors in einem Workflowdiagramm vorstellen, wie in der folgenden Abbildung zu sehen. Jeder Task hat ein Ergebnis wie „Erfolgreich“, „Fehler“ oder „Abschluss“. Die Verarbeitung nachfolgender Tasks wird erst initiiert, wenn der vorherige Task mit einem dieser Ergebnisse abgeschlossen wurde.

Ablaufsteuerungen führen Datenflüsse als Task aus. In einem Datenflusstask werden Daten aus einer Quelle extrahiert, transformiert oder in einen Datenspeicher geladen. Die Ausgabe eines einzelnen Datenflusstasks kann als Eingabe für den nächsten Datenflusstask verwendet werden, und Datenflüsse können parallel ausgeführt werden. Im Gegensatz zu Ablaufsteuerungen können zwischen Tasks in einem Datenfluss keine Einschränkungen hinzugefügt werden. Sie können jedoch einen Daten-Viewer hinzufügen, um die Daten zu beobachten, die durch die einzelnen Tasks verarbeitet werden.

![Datenfluss, der als Task in einer Ablaufsteuerung ausgeführt wird](../images/control-flow-data-flow.png)

Das obige Diagramm enthält mehrere Tasks innerhalb der Ablaufsteuerung. Einer davon ist ein Datenflusstask. Einer der Tasks ist in einen Container geschachtelt. Container ermöglichen die Strukturierung von Tasks, um eine Arbeitseinheit bereitzustellen. Ein Beispiel wäre etwa die Wiederholung von Elementen in einer Sammlung (beispielsweise Dateien in einem Ordner oder Datenbankanweisungen).

In Frage kommender Azure-Dienst:
- [Azure Data Factory v2](https://azure.microsoft.com/services/data-factory/)

Weitere Tools:
- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

## <a name="technology-choices"></a>Auswahl der Technologie

- [OLTP-Datenspeicher (Online Transaction Processing)](./online-transaction-processing.md#oltp-in-azure)
- [OLAP-Datenspeicher (Online Analytical Processing)](./online-analytical-processing.md#olap-in-azure)
- [Data Warehouses](./data-warehousing.md)
- [Pipelineorchestrierung](../technology-choices/pipeline-orchestration-data-movement.md)
