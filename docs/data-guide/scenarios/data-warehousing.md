---
title: Data Warehousing und Data Marts
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: eec883c68cf94637c3061814d0841c73b58d7e52
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="data-warehousing-and-data-marts"></a>Data Warehousing und Data Marts

Ein Datawarehouse ist ein zentrales, organisatorisches, relationales Repository für integrierte Daten aus einer einzelnen Quelle oder aus mehreren unterschiedlichen Quellen, die sich über mehrere oder alle Themenbereiche erstrecken. Data Warehouses speichern aktuelle und historische Daten und werden zur Erstellung verschiedener Datenberichte und -analysen verwendet.

![Data Warehousing in Azure](./images/data-warehousing.png)

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

Verwandte Dienste:

* [Azure SQL-Datenbank](/azure/sql-database/)
* [SQL Server auf einem virtuellen Computer](/sql/sql-server/sql-server-technical-documentation)
* [Azure Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
* [Apache Hive in HDInsight](/azure/hdinsight/hadoop/hdinsight-use-hive)
* [Interactive Query (Hive LLAP) in HDInsight](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)


## <a name="technology-choices"></a>Auswahl der Technologie

- [Data Warehouses](../technology-choices/data-warehouses.md)
- [Pipelineorchestrierung](../technology-choices/pipeline-orchestration-data-movement.md)

