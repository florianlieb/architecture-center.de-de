---
title: Interaktive Datenuntersuchung
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 20740a8fe912a63526c847416b832941f4ac33ec
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/03/2018
---
# <a name="interactive-data-exploration"></a>Interaktive Datenuntersuchung

In vielen Business Intelligence-Lösungen (BI) von Unternehmen werden Berichte und Semantikmodelle von BI-Experten erstellt und zentral verwaltet. Immer häufiger möchten Organisationen es Benutzern aber ermöglichen, datengestützte Entscheidungen zu treffen. Außerdem stellen immer mehr Unternehmen *Data Scientists* oder *Datenanalysten* ein, deren Aufgabe darin besteht, Daten interaktiv zu untersuchen und Statistikmodelle und Analyseverfahren anzuwenden, um in den Daten Trends und Muster zu ermitteln. Bei der interaktiven Datenuntersuchung sind Tools und Plattformen erforderlich, die für Ad-hoc-Abfragen und Datenvisualisierungen eine Verarbeitung mit kurzen Wartezeiten ermöglichen.

![](./images/data-exploration.png)

## <a name="self-service-bi"></a>Self-Service-Business Intelligence

Self-Service-Business Intelligence (Self-Service-BI) ist die Bezeichnung des modernen Ansatzes zum Treffen von Entscheidungen in Unternehmen, bei dem Benutzer in die Lage versetzt werden, aus Daten gewonnene Erkenntnisse zu ermitteln, zu untersuchen und für das gesamte Unternehmen bereitzustellen. Hierfür muss die Datenlösung mehrere Anforderungen erfüllen:

* Ermittlung von Unternehmensdatenquellen über einen Datenkatalog
* Masterdatenverwaltung zur Sicherstellung der Einheitlichkeit von Datenentitätsdefinitionen und -werten
* Tools für die interaktive Datenmodellierung und Visualisierung für geschäftliche Benutzer

In einer Self-Service-BI-Lösung suchen und verwenden Benutzer normalerweise Datenquellen, die für ihren jeweiligen Geschäftsbereich relevant sind, und nutzen intuitive Tools und Produktivitätsanwendungen zum Definieren von persönlichen Datenmodellen und Berichten, die sie für Kollegen bereitstellen können.

In Frage kommender Azure-Dienst:

- [Azure Data Catalog](/azure/data-catalog/data-catalog-what-is-data-catalog)
- [Microsoft Power BI](https://powerbi.microsoft.com/)

## <a name="data-science-experimentation"></a>Data Science-Experimente
Wenn eine Organisation Advanced Analytics und die Vorhersagemodellierung benötigt, wird die anfängliche Vorbereitung normalerweise von Data Science-Experten durchgeführt. Ein Data Scientist untersucht die Daten und wendet Verfahren der statistischen Analyse an, um Beziehungen zwischen den *Features* der Daten und den gewünschten vorhergesagten *Bezeichnungen* zu ermitteln. Für die Datenuntersuchung werden meist Programmiersprachen eingesetzt, z.B. Python oder R, die über eine native Unterstützung der statistischen Modellierung und Visualisierung verfügen. Die für die Datenuntersuchung verwendeten Skripts werden normalerweise in speziellen Umgebungen gehostet, z.B. Jupyter-Notebooks. Mit diesen Tools können Data Scientists die Daten programmgesteuert untersuchen und die gefundenen Erkenntnisse dokumentieren und bereitstellen.

In Frage kommender Azure-Dienst:

- [Azure Notebooks](https://notebooks.azure.com/)
- [Azure Machine Learning Studio](/azure/machine-learning/studio/what-is-ml-studio)
- [Azure Machine Learning-Experimentieren-Dienste](/azure/machine-learning/preview/experimentation-service-configuration)
- [Einführung in Azure Data Science Virtual Machine für Linux und Windows](/azure/machine-learning/data-science-virtual-machine/overview)

## <a name="challenges"></a>Herausforderungen

- **Einhaltung des Datenschutzes:** Gehen Sie mit Bedacht vor, wenn Sie für Benutzer persönliche Daten für die Self-Service-Analyse und -Berichterstellung bereitstellen. Aufgrund von Organisationsrichtlinien und gesetzlichen Bestimmungen können sich hierbei Compliance-Konflikte ergeben. 

- **Datenvolumen:** Es kann zwar nützlich sein, Benutzern den Zugriff auf die gesamte Datenquelle zu gewähren, aber dies kann zu Excel- oder Power BI-Vorgängen mit sehr langer Ausführungsdauer oder zu Spark SQL-Abfragen führen, für die sehr viele Clusterressourcen verwendet werden.

- **Benutzerwissen:** Benutzer erstellen ihre eigenen Abfragen und Aggregationen, um fundierte Geschäftsentscheidungen treffen zu können. Sind Sie sicher, dass die Benutzer über die erforderlichen Analyse- und Abfragefähigkeiten verfügen, die Sie zum Erzielen genauer Ergebnisse benötigen?

- **Freigabe von Ergebnissen:** Es kann Auswirkungen auf die Sicherheit haben, wenn Benutzer Berichte oder Datenvisualisierungen erstellen und freigeben können.

## <a name="architecture"></a>Architecture

Das Ziel bei diesem Szenario ist zwar die Unterstützung der interaktiven Datenanalyse, aber die Data Science-Aufgaben zur Datenbereinigung, Stichprobenerstellung und Strukturierung umfassen häufig Prozesse mit langer Ausführungsdauer. Aus diesem Grund ist die Verwendung einer Architektur mit [Batchverarbeitung](../big-data/batch-processing.md) sinnvoll.

## <a name="technology-choices"></a>Auswahl der Technologie

Die folgenden Technologiekomponenten sind für die interaktive Datenuntersuchung in Azure zu empfehlen.

### <a name="data-storage"></a>Datenspeicher

- **Azure Storage Blob-Container** oder **Azure Data Lake Store**: Data Scientists arbeiten im Allgemeinen mit Rohquelldaten, um sicherzustellen, dass sie Zugriff auf alle möglichen Features, Ausreißer und Fehler in den Daten haben. Bei einem Big Data-Szenario handelt es sich bei diesen Daten normalerweise um Dateien in einem Datenspeicher.

Weitere Informationen finden Sie im Artikel zur [Datenspeicherung](../technology-choices/data-storage.md).

### <a name="batch-processing"></a>Batchverarbeitung

- **R Server** oder **Spark**: Die meisten Data Scientists nutzen Programmiersprachen mit starker Unterstützung mathematischer und statistischer Pakete, z.B. R oder Python. Beim Arbeiten mit großen Datenmengen können Sie die Wartezeit reduzieren, indem Sie Plattformen verwenden, mit denen für diese Sprachen die verteilte Verarbeitung möglich ist. R Server kann allein oder zusammen mit Spark verwendet werden, um R-Verarbeitungsfunktionen horizontal hochzuskalieren, und Spark umfasst die native Unterstützung von Python für ähnliche Funktionen für das horizontale Hochskalieren in dieser Sprache.
- **Hive**: Hive ist eine gute Wahl zum Transformieren von Daten mit SQL-ähnlicher Semantik. Benutzer können mithilfe von HiveQL-Anweisungen Tabellen erstellen und laden, die in semantischer Hinsicht SQL ähneln.

Weitere Informationen finden Sie im Artikel zur [Batchverarbeitung](../technology-choices/batch-processing.md).

### <a name="analytical-data-store"></a>Analysedatenspeicher

- **Spark SQL**: Spark SQL ist eine API, die auf Spark basiert und die Erstellung von Datenrahmen und Tabellen unterstützt, für die das Abfragen per SQL-Syntax möglich ist. Unabhängig davon, ob die zu analysierenden Datendateien Rohquelldateien oder neue Dateien sind, die per Batchprozess bereinigt und vorbereitet wurden, können Benutzer für weitere Abfragen und Analysen Spark SQL-Tabellen dafür definieren. 
- **Hive**: Zusätzlich zur Batchverarbeitung von Rohdaten per Hive können Sie eine Hive-Datenbank erstellen, die Hive-Tabellen und -Ansichten basierend auf den Ordnern enthält, in denen die Daten gespeichert sind. Auf diese Weise werden interaktive Abfragen für Analyse- und Berichterstellungszwecke ermöglicht. HDInsight enthält einen Interactive Hive-Clustertyp, für den die speicherinterne Zwischenspeicherung verwendet wird, um die Antwortzeiten für Hive-Abfragen zu reduzieren. Benutzer, die sich mit SQL-ähnlicher Syntax auskennen, können Interactive Hive zum Untersuchen von Daten nutzen.

Weitere Informationen finden Sie unter [Analysedatenspeicher](../technology-choices/analytical-data-stores.md).

### <a name="analytics-and-reporting"></a>Analysen und Berichte

- **Jupyter**: Jupyter-Notebooks verfügen über eine browserbasierte Oberfläche zum Ausführen von Code in Sprachen wie R, Python oder Scala. Bei Verwendung von R Server oder Spark für die Batchverarbeitung von Daten oder von Spark SQL zum Definieren eines Schemas mit Tabellen für Abfragen kann Jupyter eine gute Wahl sein, um Abfragen für die Daten durchzuführen. Bei der Nutzung von Spark können Sie die Spark Dataframe-Standard-API oder die Spark SQL-API sowie eingebettete SQL-Anweisungen verwenden, um die Daten abzufragen und Visualisierungen zu erstellen.
- **Interactive Hive-Clients**: Bei Verwendung eines Interactive Hive-Clusters zum Abfragen der Daten können Sie die Hive-Ansicht im Ambari-Cluster-Dashboard, das Befehlszeilentool Beeline oder ein beliebiges ODBC-basiertes Tool (mit dem Hive ODBC-Treiber) nutzen, z.B. Microsoft Excel oder Power BI.

Weitere Informationen finden Sie im Artikel zur [Technologie für die Datenanalyse und Berichterstellung](../technology-choices/analysis-visualizations-reporting.md).