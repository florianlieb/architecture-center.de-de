---
title: Auswählen eines Analysedatenspeichers
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: cdc32c16e30aec5e1c0cb6959182215f99d56b56
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="choosing-an-analytical-data-store-in-azure"></a>Auswählen eines Analysedatenspeichers in Azure

In einer [Big Data](../big-data/index.md)-Architektur wird häufig ein Analysedatenspeicher benötigt, der verarbeitete Daten in einem strukturierten Format bereitstellt, das Abfragen mit Analysetools ermöglicht. Analysedatenspeicher, die sowohl das Abfragen von Hot-Path- als auch von Cold-Path-Daten unterstützen, werden zusammenfassend als Bereitstellungsebene oder Datenbereitstellungsspeicher bezeichnet.

Die Bereitstellungsebene wird für verarbeitete Hot-Path- und Cold-Path-Daten verwendet. In der [Lambda-Architektur](../big-data/index.md#lambda-architecture) ist die Bereitstellungsebene in zwei Ebenen unterteilt: eine Ebene für die _schnelle Bereitstellung_, auf der inkrementell verarbeitete Daten gespeichert werden, und eine Ebene für die _Batchbereitstellung_, die die Ausgabe der Batchverarbeitung enthält. Für die Bereitstellungsebene ist eine starke Unterstützung für zufällige Lesevorgänge mit kurzer Wartezeit erforderlich. Der Datenspeicher für die Geschwindigkeitsebene sollte außerdem zufällige Schreibvorgänge unterstützen, da es beim Batchladen von Daten in diesen Speicher zu unerwünschten Verzögerungen kommen kann. Andererseits wird für die Datenspeicherung für die Batchebene keine Unterstützung von zufälligen Schreibvorgängen benötigt, sondern von Batchschreibvorgängen.

Es gibt keine Lösung für die Datenverwaltung, die für alle Datenspeicheraufgaben am besten geeignet ist. Verschiedene Datenverwaltungslösungen sind für unterschiedliche Aufgaben optimiert. Die meisten Cloud-Apps und Big Data-Prozesse aus der Praxis verfügen über viele verschiedene Datenspeicheranforderungen, und häufig wird eine Kombination von Datenspeicherlösungen genutzt.

## <a name="what-are-your-options-when-choosing-an-analytical-data-store"></a>Welche Möglichkeiten haben Sie bei der Auswahl eines Analysedatenspeichers?

Es gibt mehrere Optionen für die Datenbereitstellungsspeicherung in Azure. Dies richtet sich nach Ihren jeweiligen Anforderungen:

- [SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [Azure SQL-Datenbank](/azure/sql-database/)
- [SQL Server auf einer Azure-VM](/sql/sql-server/sql-server-technical-documentation)
- [HBase/Phoenix in HDInsight](/azure/hdinsight/hbase/apache-hbase-overview)
- [Hive LLAP in HDInsight](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)
- [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [Azure Cosmos DB](/azure/cosmos-db/)

Im Rahmen dieser Optionen gibt es verschiedene Datenbankmodelle, die für unterschiedliche Arten von Aufgaben optimiert sind:

- [Schlüssel-Wert](https://msdn.microsoft.com/library/dn313285.aspx#sec7)-Datenbanken enthalten für jeden Schlüsselwert ein serialisiertes Objekt. Diese eignen sich gut zum Speichern von großen Datenmengen, bei denen Sie ein Element für einen bestimmten Schlüsselwert abrufen möchten und keine Abfrage basierend auf anderen Eigenschaften des Elements durchführen müssen.
- [Dokument](https://msdn.microsoft.com/library/dn313285.aspx#sec8)datenbanken sind Schlüssel-Wert-Datenbanken, in denen die Werte *Dokumente* sind. Ein „Dokument“ ist in diesem Zusammenhang eine Sammlung mit benannten Feldern und Werten. In der Datenbank werden die Daten normalerweise in einem bestimmten Format gespeichert, z.B. XML, YAML, JSON oder BSON, aber es kann auch Nur-Text verwendet werden. In Dokumentdatenbanken können Abfragen für Felder durchgeführt werden, bei denen es sich nicht um Schlüsselfelder handelt, und es können sekundäre Indizes definiert werden, um das Abfragen effizienter zu gestalten. Eine Dokumentdatenbank ist daher besser für Anwendungen geeignet, für die Daten basierend auf Kriterien abgerufen werden müssen, die komplexer als der Wert des Dokumentschlüssels sind. Sie können beispielsweise Abfragen für Felder wie Produkt-ID, Kunden-ID oder Kundenname durchführen.
- [Column-Family](https://msdn.microsoft.com/library/dn313285.aspx#sec9)-Datenbanken sind Schlüssel-Wert-Datenspeicher, bei denen die Datenspeicherung in Sammlungen mit verwandten Spalten strukturiert ist, die als „Spaltenfamilien“ (Column Families) bezeichnet werden. Eine Datenbank für eine Erhebung kann beispielsweise eine Gruppe mit Spalten für den Namen einer Person (Vorname, Zweiter Vorname, Nachname), eine Gruppe für die Adresse der Person und eine Gruppe für die Profilinformationen der Person (Geburtsdatum, Geschlecht) enthalten. In der Datenbank kann jede Spaltenfamilie auf einer separaten Partition gespeichert werden, während alle Daten für eine Person demselben Schlüssel zugeordnet bleiben. Eine Anwendung kann eine einzelne Spaltenfamilie lesen, ohne sich alle Daten einer Entität durchzulesen.
- In [Diagramm](https://msdn.microsoft.com/library/dn313285.aspx#sec10)datenbanken werden Informationen als Sammlung mit Objekten und Beziehungen gespeichert. Eine Diagrammdatenbank kann auf effiziente Weise Abfragen durchführen, die das Netzwerk der Objekte und die dazugehörigen Beziehungen durchlaufen. Die Objekte können beispielsweise Mitarbeiter in einer Personalverwaltungsdatenbank sein, und Sie können Abfragen der Art „Alle Mitarbeiter ermitteln, die direkt oder indirekt für Stephan arbeiten“ durchführen.

## <a name="key-selection-criteria"></a>Wichtige Auswahlkriterien

Beantworten Sie die folgenden Fragen, um die Auswahl einzuschränken:

- Benötigen Sie Bereitstellungsspeicher, der als langsamster Pfad (Hot Path) für Ihre Daten dienen kann? Wenn ja, können Sie sich auf die Optionen beschränken, die für eine Ebene für die schnelle Bereitstellung optimiert sind.

- Muss die hochgradig parallelisierte Verarbeitung (Massively Parallel Processing, MPP) unterstützt werden, bei der Abfragen automatisch auf mehrere Prozesse oder Knoten verteilt werden? Wenn ja, sollten Sie eine Option wählen, die das horizontale Hochskalieren für Abfragen unterstützt.

- Bevorzugen Sie die Verwendung eines relationalen Datenspeichers? Wenn ja, können Sie sich auf die Optionen mit einem relationalen Datenbankmodell beschränken. Beachten Sie aber, dass einige nicht relationale Speicher die SQL-Syntax für Abfragen unterstützen und dass Tools wie PolyBase genutzt werden können, um nicht relationale Datenspeicher abzufragen.

## <a name="capability-matrix"></a>Funktionsmatrix

In den folgenden Tabellen sind die Hauptunterschiede in Bezug auf die Funktionen zusammengefasst.

### <a name="general-capabilities"></a>Allgemeine Funktionen

| | SQL-Datenbank | SQL Data Warehouse | HBase/Phoenix in HDInsight | Hive LLAP in HDInsight | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| Verwalteter Dienst | Ja | Ja | Ja<sup>1</sup> | Ja<sup>1</sup> | Ja | Ja |
| Primäres Datenbankmodell | Relational (Spaltenformat bei Verwendung von Columnstore-Indizes) | Relationale Tabellen mit Speicherung in Spalten | Wide Columnstore | Hive/In-Memory | Tabellarisch/MOLAP-Semantikmodelle | Dokumentspeicher, Diagramm, Schlüssel-Wert-Speicherung, Wide Columnstore |
| SQL-Sprachunterstützung | Ja | Ja | Ja (mit [Phoenix](http://phoenix.apache.org/)-JDBC-Treiber) | Ja | Nein | Ja |
| Optimiert für Ebene für schnelle Bereitstellung | Ja<sup>2</sup> | Nein  | Ja | Ja | Nein | Ja |

[1] Mit manueller Konfiguration und Skalierung

[2] Mit speicheroptimierten Tabellen und Hashindex oder nicht gruppierten Indizes
 
### <a name="scalability-capabilities"></a>Skalierbarkeitsfunktionen

|                                                  | SQL-Datenbank | SQL Data Warehouse | HBase/Phoenix in HDInsight | Hive LLAP in HDInsight | Azure Analysis Services | Cosmos DB |
|--------------------------------------------------|--------------|--------------------|----------------------------|------------------------|-------------------------|-----------|
| Redundante regionale Server für Hochverfügbarkeit |     Ja      |        Ja         |            Ja             |           Nein            |           Nein             |    Ja    |
|             Unterstützung des horizontalen Hochskalierens von Abfragen             |      Nein       |        Ja         |            Ja             |          Ja           |           Ja           |    Ja    |
|          Dynamische Skalierbarkeit (zentrales Hochskalieren)          |     Ja      |        Ja         |             Nein              |           Nein            |           Ja           |    Ja    |
|        Unterstützung der speicherinternen Zwischenspeicherung von Daten        |     Ja      |        Ja         |             Nein             |          Ja           |           Ja           |    Nein      |

### <a name="security-capabilities"></a>Sicherheitsfunktionen

| | SQL-Datenbank | SQL Data Warehouse | HBase/Phoenix in HDInsight | Hive LLAP in HDInsight | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| Authentifizierung  | SQL/Azure Active Directory (Azure AD) | SQL/Azure AD | Lokal/Azure AD <sup>1</sup> | Lokal/Azure AD <sup>1</sup> | Azure AD | Datenbankbenutzer/Azure AD per Zugriffssteuerung (IAM) |
| Datenverschlüsselung ruhender Daten | Ja<sup>2</sup> | Ja<sup>2</sup> | Ja<sup>1</sup> | Ja<sup>1</sup> | Ja | Ja |
| Sicherheit auf Zeilenebene | Ja | Nein  | Ja<sup>1</sup> | Ja <sup>1</sup> | Ja (per Sicherheit auf Objektebene im Modell) | Nein  |
| Unterstützung von Firewalls | Ja | Ja | Ja<sup>3</sup> | Ja<sup>3</sup> | Ja | Ja |
| Dynamische Datenmaskierung | Ja | Nein  | Ja<sup>1</sup> | Ja * | Nein  | Nein  |

[1] Erfordert die Verwendung eines [in die Domäne eingebundenen HDInsight-Clusters](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] Verwendung von Transparent Data Encryption (TDE) zum Verschlüsseln und Entschlüsseln von ruhenden Daten erforderlich

[3] Bei Verwendung in einem Azure Virtual Network. Siehe [Erweitern von Azure HDInsight per Azure Virtual Network](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).
