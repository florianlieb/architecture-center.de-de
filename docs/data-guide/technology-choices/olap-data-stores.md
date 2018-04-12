---
title: Auswählen eines OLAP-Datenspeichers
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: f3041b95696c9408a2c9ab747fe1ec3041db0743
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/31/2018
---
# <a name="choosing-an-olap-data-store-in-azure"></a>Auswählen eines OLAP-Datenspeichers in Azure

Die analytische Onlineverarbeitung (Online Analytical Processing, OLAP) ist eine Technologie, die große Geschäftsdatenbanken organisiert und komplexe Analysen unterstützt. In diesem Thema werden die Optionen für OLAP-Lösungen in Azure verglichen.

> [!NOTE]
> Weitere Informationen dazu, wann Sie einen OLAP-Datenspeicher verwenden sollten, finden Sie unter [Online analytical processing](../scenarios/online-analytical-processing.md) (Analytische Onlineverarbeitung).

## <a name="what-are-your-options-when-choosing-an-olap-data-store"></a>Welche Optionen stehen Ihnen bei der Auswahl eines OLAP-Datenspeichers zur Verfügung?

In Azure erfüllen alle folgenden Datenspeicher die grundlegenden Anforderungen für OLAP:

- [SQL Server mit Columnstore-Indizes](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)
- [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [SQL Server Analysis Services (SSAS)](/sql/analysis-services/analysis-services)

SQL Server Analysis Services (SSAS) bietet OLAP- und Data Mining-Funktionen für Business Intelligence-Anwendungen. Sie können SSAS auf lokalen Servern installieren oder auf einer VM in Azure hosten. Azure Analysis Services ist ein vollständig verwalteter Dienst, der die gleichen Hauptfunktionen wie SSAS bereitstellt. Azure Analysis Services unterstützt das Herstellen von Verbindungen mit [verschiedenen Datenquellen](/azure/analysis-services/analysis-services-datasource) in der Cloud und lokalen Datenquellen in der Organisation.

Gruppierte Columnstore-Indizes sind in SQL Server 2014 und höher sowie Azure SQL-Datenbank verfügbar und eignen sich ideal für OLAP-Workloads. Ab SQL Server 2016 (einschließlich Azure SQL-Datenbank) können Sie jedoch mithilfe von aktualisierbaren nicht gruppierten Columnstore-Indizes die hybride Verarbeitung von Transaktionen und Analysen (Hybrid Transactional and Analytical Processing, HTAP) nutzen. HTAP ermöglicht Ihnen die OLTP- und OLAP-Verarbeitung auf der gleichen Plattform. Dadurch entfällt die Notwendigkeit, mehrere Kopien Ihrer Daten zu speichern sowie separate OLTP- und OLAP-Systeme zu verwenden. Weitere Informationen finden Sie unter [Erste Schritte mit Columnstore für operative Echtzeitanalyse](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics).

## <a name="key-selection-criteria"></a>Wichtige Auswahlkriterien

Beantworten Sie die folgenden Fragen, um die Auswahl einzuschränken:

- Möchten Sie einen verwalteten Dienst verwenden, anstatt Ihre eigenen Server zu verwalten?

- Benötigen Sie eine sichere Authentifizierung mit Azure Active Directory (Azure AD)?

- Möchten Sie Echtzeitanalysen ausführen? Ist dies der Fall, beschränken Sie sich auf die Optionen, die Echtzeitanalysen unterstützten. 

    *Echtzeitanalyse* bezieht sich in diesem Kontext auf eine einzelne Datenquelle, beispielsweise eine ERP-Anwendung (Enterprise Resource Planning), die sowohl auf einer operativen als auch einer analytischen Workload ausgeführt wird. Wenn Sie Daten aus mehreren Quellen integrieren müssen oder eine sehr hohe Analyseleistung durch Verwendung von vorab aggregierten Daten wie Cubes erforderlich ist, benötigen Sie möglicherweise trotzdem ein separates Data Warehouse.

- Müssen Sie vorab aggregierte Daten verwenden, beispielsweise zum Bereitstellen von semantischen Modellen, die Geschäftsbenutzern die Verwendung von Analysen erleichtern? Ist dies der Fall, wählen Sie eine Option aus, die mehrdimensionale Cubes oder tabellarische Semantikmodelle unterstützt. 

    Die Bereitstellung von Aggregaten kann Benutzern die konsistente Berechnung von Datenaggregaten ermöglichen. Bei der Arbeit mit mehreren Spalten und vielen Zeilen können vorab aggregierte Daten zudem die Leistung erheblich steigern. Daten können in mehrdimensionalen Cubes oder tabellarischen Semantikmodellen vorab aggregiert werden.

- Müssen Sie Daten aus mehreren Quellen (abgesehen von Ihrem OLTP-Datenspeicher) integrieren? Ist dies der Fall, sollten Sie Optionen erwägen, die eine einfache Integration mehrerer Datenquellen ermöglichen.

## <a name="capability-matrix"></a>Funktionsmatrix

In den folgenden Tabellen sind die Hauptunterschiede in Bezug auf die Funktionen zusammengefasst.

### <a name="general-capabilities"></a>Allgemeine Funktionen

| | Azure Analysis Services | SQL Server Analysis Services | SQL Server mit Columnstore-Indizes | Azure SQL-Datenbank mit Columnstore-Indizes |
| --- | --- | --- | --- | --- |
| Verwalteter Dienst | Ja | Nein  | Nein  | Ja |
| Unterstützung mehrdimensionaler Cubes | Nein  | Ja | Nein  | Nein  |
| Unterstützung tabellarischer Semantikmodelle | Ja | Ja | Nein  | Nein  |
| Einfache Integration mehrerer Datenquellen | Ja | Ja | Nein <sup>1</sup> | Nein <sup>1</sup> |
| Unterstützung von Echtzeitanalysen | Nein  | Nein  | Ja | Ja |
| Prozess zum Kopieren von Daten aus Quellen erforderlich | Ja | Ja | Nein  | Nein  |
| Azure AD-Integration | Ja | Nein  | Nein<sup>2</sup> | Ja |

[1] Obwohl SQL Server und Azure SQL-Datenbank nicht zum Abfragen und Integrieren mehrerer externer Datenquellen verwendet werden können, können Sie zu diesem Zweck eine Pipeline mit [SSIS](/sql/integration-services/sql-server-integration-services) oder [Azure Data Factory](/azure/data-factory/) erstellen. Eine auf einer Azure-VM gehostete SQL Server-Instanz bietet zusätzliche Optionen, beispielsweise Verbindungsserver und [PolyBase](/sql/relational-databases/polybase/polybase-guide). Weitere Informationen finden Sie unter [Pipeline orchestration, control flow, and data movement](../technology-choices/pipeline-orchestration-data-movement.md) (Pipelineorchestrierung, Ablaufsteuerung und Datenverschiebung).

[2] Das Herstellen einer Verbindung mit einer SQL Server-Instanz, die auf einer Azure-VM ausgeführt wird, wird für ein Azure AD-Konto nicht unterstützt. Verwenden Sie stattdessen ein Active Directory-Domänenkonto.

### <a name="scalability-capabilities"></a>Skalierbarkeitsfunktionen

| | Azure Analysis Services | SQL Server Analysis Services | SQL Server mit Columnstore-Indizes | Azure SQL-Datenbank mit Columnstore-Indizes |
| --- | --- | --- | --- | --- |
| Redundante regionale Server für Hochverfügbarkeit  | Ja | Nein | Ja | Ja |
| Unterstützung des horizontalen Hochskalierens von Abfragen  | Ja | Nein | Ja | Nein  |
| Dynamische Skalierbarkeit (zentrales Hochskalieren)  | Ja | Nein | Ja | Nein  |

