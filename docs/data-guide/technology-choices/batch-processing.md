---
title: "Auswählen einer Batchverarbeitungstechnologie"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: bfb850ee8e9d8fd41927b4ca3b612e15b5ae6b11
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-batch-processing-technology-in-azure"></a>Auswählen einer Batchverarbeitungstechnologie in Azure

Big Data-Lösungen nutzen häufig Batchaufträge mit langer Ausführungszeit, um Daten für die Analyse zu filtern, zu aggregieren oder anderweitig vorzubereiten. In der Regel umfassen diese Aufträge das Lesen von Quelldateien aus skalierbarem Speicher (etwa HDFS, Azure Data Lake Store und Azure Storage), das Verarbeiten dieser Dateien und das Schreiben der Ausgabe in neue Dateien in skalierbarem Speicher. 

Die wichtigste Anforderung bei diesen Batchverarbeitungs-Engines ist die Möglichkeit zur Skalierung von Berechnungen, um große Datenvolumen zu verarbeiten. Im Gegensatz zur Echtzeitverarbeitung werden bei der Batchverarbeitung Wartezeiten (die Zeit zwischen der Datenerfassung und der Berechnung eines Ergebnisses) erwartet, die in Minuten oder Stunden angegeben werden.

## <a name="what-are-your-options-when-choosing-a-batch-processing-technology"></a>Welche Batchverarbeitungstechnologien stehen zur Verfügung?

In Azure erfüllen alle folgenden Datenspeicher die grundlegenden Anforderungen für die Batchverarbeitung:

- [Azure Data Lake Analytics](/azure/data-lake-analytics/)
- [Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [HDInsight mit Spark](/azure/hdinsight/spark/apache-spark-overview)
- [HDInsight mit Hive](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [HDInsight mit Hive LLAP](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

## <a name="key-selection-criteria"></a>Wichtige Auswahlkriterien

Beantworten Sie die folgenden Fragen, um die Auswahl einzuschränken:

- Möchten Sie einen verwalteten Dienst verwenden, anstatt Ihre eigenen Server zu verwalten?

- Möchten Sie Batchverarbeitungslogik deklarativ oder imperativ erstellen?

- Führen Sie Batchaufträge schubweise aus? Falls ja, ziehen Sie Optionen in Erwägung, bei denen Sie den Cluster anhalten können bzw. bei denen die Kosten pro Batchauftrag abgerechnet werden.

- Müssen Sie bei der Batchverarbeitung auch relationale Datenspeicher abfragen, etwa zum Nachschlagen von Referenzdaten? Falls ja, ziehen Sie Optionen in Erwägung, die das Abfragen von externen relationalen Speichern ermöglichen.

## <a name="capability-matrix"></a>Funktionsmatrix

In der folgenden Tabellen sind die Hauptunterschiede der Funktionen zusammengefasst: 

### <a name="general-capabilities"></a>Allgemeine Funktionen

| | Azure Data Lake Analytics | Azure SQL Data Warehouse | HDInsight mit Spark | HDInsight mit Hive | HDInsight mit Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| Verwalteter Dienst | Ja | Ja | Ja <sup>1</sup> | Ja <sup>1</sup> | Ja <sup>1</sup> |
| Unterstützt das Anhalten von Computevorgängen | Nein  | Ja | Nein  | Nein  | Nein  |
| Relationaler Datenspeicher | Ja | Ja | Nein  | Nein  | Nein  |
| Programmierbarkeit | U-SQL | T-SQL | Python, Scala, Java, R | HiveQL | HiveQL |
| Programmierparadigma | Mischung aus deklarativ und imperativ  | Deklarativ | Mischung aus deklarativ und imperativ | Deklarativ | Deklarativ | 
| Preismodell | Pro Batchauftrag | Nach Clusterstunde | Nach Clusterstunde | Nach Clusterstunde | Nach Clusterstunde |  

[1] Mit manueller Konfiguration und Skalierung
 
### <a name="integration-capabilities"></a>Integrationsfunktionen
| | Azure Data Lake Analytics | SQL Data Warehouse | HDInsight mit Spark | HDInsight mit Hive | HDInsight mit Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| Zugriff über Azure Data Lake Store | Ja | Ja | Ja | Ja | Ja |
| Abfragen über Azure Storage | Ja | Ja | Ja | Ja | Ja |
| Abfragen über externe relationale Speicher | Ja | Nein | Ja | Nein  | Nein  |

### <a name="scalability-capabilities"></a>Skalierbarkeitsfunktionen
| | Azure Data Lake Analytics | SQL Data Warehouse | HDInsight mit Spark | HDInsight mit Hive | HDInsight mit Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| Granularität bei der horizontalen Skalierung  | Pro Auftrag | Pro Cluster | Pro Cluster | Pro Cluster | Pro Cluster |
| Schnelles horizontales Hochskalieren (weniger als 1 Minute) | Ja | Ja | Nein  | Nein  | Nein  |
| Speicherinternes Zwischenspeichern | Nein  | Ja | Ja | Nein | Ja | 

### <a name="security-capabilities"></a>Sicherheitsfunktionen
| | Azure Data Lake Analytics | SQL Data Warehouse | HDInsight mit Spark | Apache Hive in HDInsight | Hive LLAP in HDInsight |
| --- | --- | --- | --- | --- | --- |
| Authentifizierung  | Azure Active Directory (Azure AD) | SQL/Azure AD | Nein  | lokal/Azure AD <sup>1</sup> | lokal/Azure AD <sup>1</sup> |
| Autorisierung  | Ja | Ja| Nein  | Ja <sup>1</sup> | Ja <sup>1</sup> |
| Überwachung  | Ja | Ja | Nein  | Ja <sup>1</sup> | Ja <sup>1</sup> |
| Datenverschlüsselung ruhender Daten | Ja| Ja <sup>2</sup> | Ja | Ja | Ja |
| Sicherheit auf Zeilenebene | Nein  | Ja | Nein  | Ja <sup>1</sup> | Ja <sup>1</sup> |
| Unterstützung von Firewalls | Ja | Ja | Ja | Ja <sup>3</sup> | Ja <sup>3</sup> |
| Dynamische Datenmaskierung | Nein  | Nein  | Nein  | Ja <sup>1</sup> | Ja <sup>1</sup> |

[1] Verwendung eines [in die Domäne eingebundenen HDInsight-Clusters](/azure/hdinsight/domain-joined/apache-domain-joined-introduction) erforderlich

[2] Verwendung von Transparent Data Encryption (TDE) zum Verschlüsseln und Entschlüsseln von ruhenden Daten erforderlich

[3] Unterstützt bei [Verwendung in einem virtuellen Azure-Netzwerk](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)
