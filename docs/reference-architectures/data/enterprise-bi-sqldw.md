---
title: Enterprise BI mit SQL Data Warehouse
description: Verwenden von Azure, um aus lokal gespeicherten relationalen Daten Einblicke in Geschäftsvorgänge zu gewinnen
author: alexbuckgit
ms.date: 04/13/2018
ms.openlocfilehash: b5e5aa32fc9cc8c7b8b5a42c9a4fc3e0216b2f72
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/16/2018
---
# <a name="enterprise-bi-with-sql-data-warehouse"></a>Enterprise BI mit SQL Data Warehouse
 
Diese Referenzarchitektur implementiert eine [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt)-Pipeline (Extract-Load-Transform), die Daten aus einer lokalen SQL Server-Datenbank in SQL Data Warehouse verschiebt und die Daten für die Analyse transformiert. [**So stellen Sie diese Lösung bereit**.](#deploy-the-solution)

![](./images/enterprise-bi-sqldw.png)

**Szenario**: Ein großes OLTP-Dataset einer Organisation ist in einer SQL Server-Datenbank lokal gespeichert. Die Organisation möchte mittels SQL Data Warehouse eine Analyse mit Power BI ausführen. 

Diese Referenzarchitektur ist für einmalige oder bedarfsgesteuerte Aufträge bestimmt. Wenn Sie fortlaufend (stündlich oder täglich) Daten verschieben müssen, sollten Sie mit Azure Data Factory einen automatisierten Workflow definieren.

## <a name="architecture"></a>Architecture

Die Architektur umfasst die folgenden Komponenten.

**SQL Server**. Die Quelldaten befinden sich in einer lokalen SQL Server-Datenbank. Um die lokale Umgebung zu simulieren, stellen die Bereitstellungsskripts für diese Architektur eine VM in Azure bereit, auf der SQL Server installiert ist. 

**Blobspeicher**. Blobspeicher wird als Stagingbereich zum Kopieren der Daten vor dem Laden in SQL Data Warehouse verwendet.

**Azure SQL Data Warehouse**. [SQL Data Warehouse](/azure/sql-data-warehouse/) ist ein verteiltes System für die Analyse großer Datenmengen. Es unterstützt massive Parallelverarbeitung (Massive Parallel Processing, MPP), die die Ausführung von Hochleistungsanalysen ermöglicht. 

**Azure Analysis Services**: [Analysis Services](/azure/analysis-services/) ist ein vollständig verwalteter Dienst, der Datenmodellierungsfunktionen ermöglicht. Verwenden Sie Analysis Services zum Erstellen eines semantischen Modells, das Benutzer abfragen können. Analysis Services ist in einem Szenario mit BI-Dashboard besonders nützlich. In dieser Architektur liest Analysis Services Daten aus dem Data Warehouse, um das semantische Modell zu verarbeiten, und bedient Dashboardabfragen effizient. Darüber hinaus unterstützt der Dienst auch die elastische Parallelität durch zentrales Hochskalieren von Replikaten zur schnelleren Abfragenverarbeitung.

Zurzeit unterstützt Azure Analysis Services tabellarische Modelle, aber keine mehrdimensionalen Modelle. Tabellarische Modelle verwenden relationale Modellierungskonstrukte (Tabellen und Spalten), wohingegen mehrdimensionale Modelle OLAP-Modellierungskonstrukte (Cubes, Dimensionen und Measures) verwenden. Wenn Sie mehrdimensionale Modelle benötigen, verwenden Sie SQL Server Analysis Services (SSAS). Weitere Informationen finden Sie unter [Vergleichen von tabellarischen und mehrdimensionalen Lösungen](/sql/analysis-services/comparing-tabular-and-multidimensional-solutions-ssas).

**Power BI**: Power BI ist eine Suite aus Business Analytics-Tools zum Analysieren von Daten für Einblicke in Geschäftsvorgänge. In dieser Architektur dient sie zum Abfragen des in Analysis Services gespeicherten semantischen Modells.

**Azure Active Directory** (Azure AD) authentifiziert Benutzer, die über Power BI eine Verbindung mit dem Analysis Services-Server herstellen.

## <a name="data-pipeline"></a>Data Pipeline
 
Diese Referenzarchitektur verwendet die [WorldWideImporters](/sql/sample/world-wide-importers/wide-world-importers-oltp-database)-Beispieldatenbank als Datenquelle. Die Phasen der Datenpipeline sind:

1. Exportieren der Daten aus SQL Server in Flatfiles (BCP-Hilfsprogramm).
2. Kopieren der Flatfiles in Azure Blob Storage (AzCopy).
3. Laden der Daten in SQL Data Warehouse (PolyBase).
4. Transformieren der Daten in ein Sternschema (T-SQL).
5. Laden eines Semantikmodells in Analysis Services (SQL Server Data Tools).

![](./images/enterprise-bi-sqldw-pipeline.png)
 
> [!NOTE]
> Erwägen Sie für die Schritte 1 &ndash; 3 die Verwendung von Redgate Data Platform Studio. Data Platform Studio wendet optimal abgestimmte Kompatibilitätspatches und Optimierungen an und ermöglicht so den schnellsten Einstieg in die Verwendung von SQL Data Warehouse. Weitere Informationen finden Sie unter [Laden von Daten mit Redgate Data Platform Studio](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate). 

In den folgenden Abschnitten werden diese Phasen ausführlicher beschrieben.

### <a name="export-data-from-sql-server"></a>Exportieren von Daten aus SQL Server

Das Hilfsprogramm [BCP](/sql/tools/bcp-utility) (Bulk Copy Program) ist eine schnelle Möglichkeit zum Erstellen von Textflatfiles aus SQL-Tabellen. In diesem Schritt wählen Sie die Spalten, die Sie exportieren möchten, jedoch nicht die zu transformierenden Daten. Alle Datentransformationen sollten in SQL Data Warehouse erfolgen.

**Empfehlungen**

Planen Sie die Datenextrahierung nach Möglichkeit außerhalb der Spitzenzeiten, um Ressourcenkonflikte in der Produktionsumgebung zu minimieren. 

Führen Sie BCP nicht auf dem Datenbankserver aus. Führen Sie BCP stattdessen auf einem anderen Computer aus. Schreiben Sie die Dateien auf ein lokales Laufwerk. Stellen Sie sicher, dass genügend E/A-Ressourcen für gleichzeitige Schreibvorgänge bereitstehen. Um die Leistung zu optimieren, exportieren Sie die Dateien auf dedizierte schnelle Speicherlaufwerke.

Sie können die Netzwerkübertragung beschleunigen, indem Sie die exportierten Daten im komprimierten Gzip-Format speichern. Allerdings ist das Laden komprimierter Dateien in Warehouse langsamer als das Laden dekomprimierter Dateien, sodass ein Kompromiss zwischen schneller Netzwerkübertragung und schnellerem Laden getroffen werden muss. Wenn Sie die Gzip-Komprimierung verwenden möchten, erstellen Sie keine einzelne Gzip-Datei. Teilen Sie die Daten stattdessen auf mehrere komprimierte Dateien auf.

### <a name="copy-flat-files-into-blob-storage"></a>Kopieren von Flatfiles in Blobspeicher

Das Hilfsprogramm [AzCopy](/azure/storage/common/storage-use-azcopy) ist für das Hochleistungskopieren von Daten in den Azure-Blobspeicher bestimmt.

**Empfehlungen**

Erstellen Sie das Speicherkonto in einer Region, die sich in der Nähe des Quelldatenspeicherorts befindet. Stellen Sie das Speicherkonto und die SQL Data Warehouse-Instanz in der gleichen Region bereit. 

Führen Sie AzCopy und Ihre Produktionsworkloads nicht auf dem gleichen Computer aus, da CPU- und E/A-Verbrauch die Produktionsworkloads beeinträchtigen können. 

Testen Sie den Upload zuerst, um die Uploadgeschwindigkeit zu ermitteln. Sie können in AzCopy mit der Option „/NC“ die Anzahl der gleichzeitigen Kopiervorgänge angeben. Beginnen Sie mit dem Standardwert, und experimentieren Sie mit dieser Einstellung, um die Leistung zu optimieren. In einer Umgebung mit geringer Bandbreite können zu viele gleichzeitige Vorgänge die Netzwerkverbindung überlasten, sodass die Vorgänge nicht erfolgreich abgeschlossen werden können.  

AZCopy verschiebt Daten über das öffentliche Internet in den Speicher. Wenn dies nicht schnell genug ist, sollten Sie die Einrichtung einer [ExpressRoute](/azure/expressroute/)-Verbindung erwägen. ExpressRoute ist ein Dienst, der Ihre Daten über eine dedizierte private Verbindung zu Azure weiterleitet. Wenn Ihre Netzwerkverbindung zu langsam ist, können Sie die Daten auch physisch auf einem Datenträger an ein Azure-Rechenzentrum senden. Weitere Informationen finden Sie unter [Übertragen von Daten in und aus Azure](/azure/architecture/data-guide/scenarios/data-transfer).

Während eines Kopiervorgangs erstellt AzCopy eine temporäre Journaldatei, mit der AzCopy den Vorgang bei einer Unterbrechung (z.B. aufgrund eines Netzwerkfehlers) neu starten kann. Stellen Sie sicher, dass auf dem Datenträger genügend Speicherplatz zum Speichern der Journaldateien vorhanden ist. Mit der Option „/Z“ können Sie angeben, wohin die Journaldateien geschrieben werden.

### <a name="load-data-into-sql-data-warehouse"></a>Laden von Daten in SQL Data Warehouse

Laden Sie die Dateien mit [PolyBase](/sql/relational-databases/polybase/polybase-guide) aus dem Blobspeicher in das Data Warehouse. PolyBase ist dafür ausgelegt, die MPP-Architektur (Massively Parallel Processing) von SQL Data Warehouse zu nutzen, und bietet damit die schnellste Möglichkeit, Daten in SQL Data Warehouse zu laden. 

Das Laden der Daten ist ein zweistufiger Prozess:

1. Erstellen eines Satzes externer Tabellen für die Daten. Eine externe Tabelle ist eine Tabellendefinition, die auf außerhalb des Warehouse gespeicherte Daten zeigt &mdash; in diesem Fall die Flatfiles im Blobspeicher. Dieser Schritt verschiebt keine Daten in das Warehouse.
2. Erstellen von Stagingtabellen und Laden der Daten in die Stagingtabellen. Dieser Schritt kopiert die Daten in das Warehouse.

**Empfehlungen**

Sie sollten SQL Data Warehouse verwenden, wenn Sie große Datenmengen (mehr als 1 TB) haben und eine Analyseworkload ausführen, die von der Parallelität profitiert. SQL Data Warehouse ist nicht ideal für OLTP-Workloads oder kleinere Datasets (< 250GB). Verwenden Sie für Datasets unter 250GB Azure SQL-Datenbank oder SQL Server. Weitere Informationen finden Sie unter [Data Warehousing und Data Marts](../../data-guide/relational-data/data-warehousing.md).

Erstellen Sie die Stagingtabellen als Heaptabellen, die nicht indiziert werden. Die Abfragen, die die Produktionstabellen erstellen, resultieren in einem vollständigen Tabellenscan, sodass die Stagingtabellen nicht Indiziert werden müssen.

PolyBase nutzt automatisch die Vorteile der Parallelität im Warehouse. Die Ladeleistung wird skaliert, indem Sie die DWUs heraufsetzen. Die beste Leistung erzielen Sie mit einem einzelnen Ladevorgang. Die Aufteilung der Eingabedaten in Blöcke und das Ausführen mehrerer paralleler Ladevorgänge bringt keinen Leistungsvorteil.

PolyBase kann mit GZip komprimierte Dateien lesen. Allerdings wird nur ein einziger Leser pro komprimierter Datei verwendet, weil das Dekomprimieren der Datei ein Singlethread-Vorgang ist. Vermeiden Sie daher, eine einzelne große komprimierte Datei zu laden. Teilen Sie die Daten stattdessen in mehrere komprimierte Dateien auf, um den Vorteil der Parallelität zu nutzen. 

Bedenken Sie dabei folgende Einschränkungen:

- PolyBase unterstützt eine maximale Spaltengröße von `varchar(8000)`, `nvarchar(4000)` oder `varbinary(8000)`. Wenn Ihre Daten diese Grenzen überschreiten, können Sie die Daten in Blöcke unterteilen, wenn Sie sie exportieren, und die Blöcke nach dem Import wieder zusammensetzen. 

- PolyBase verwendet das feste Zeilenabschlusszeichen „\n“ oder einen Zeilenvorschub. Dies kann Probleme verursachen, wenn das Zeilenumbruchzeichen in den Quelldaten vorkommt.

- Ihr Quelldatenschema könnte Datentypen enthalten, die in SQL Data Warehouse nicht unterstützt werden.

Um diese Einschränkungen zu umgehen, können Sie eine gespeicherte Prozedur erstellen, die die erforderlichen Konvertierungen ausführt. Verweisen Sie auf diese gespeicherte Prozedur, wenn Sie BCP ausführen. Alternativ konvertiert [Redgate Data Platform Studio](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate) automatisch Datentypen, die in SQL Data Warehouse nicht unterstützt werden.

Weitere Informationen finden Sie in den folgenden Artikeln:

- [Bewährte Methoden zum Laden von Daten in Azure SQL Data Warehouse](/azure/sql-data-warehouse/guidance-for-loading-data)
- [Migrieren Ihrer Schemas nach SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-migrate-schema)
- [Leitfaden zum Definieren von Datentypen für Tabellen in SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types)

### <a name="transform-the-data"></a>Transformieren der Daten

Transformieren Sie die Daten, und verschieben Sie sie in Produktionstabellen. In diesem Schritt werden die Daten in ein Sternschema mit Dimensions- und Faktentabellen umgewandelt, sodass sie für die semantische Modellierung geeignet sind.

Erstellen Sie die Produktionstabellen mit gruppierten Columnstore-Indizes, die beste Abfragegesamtleistung bieten. Columnstore-Indizes sind für Abfragen optimiert, die viele Datensätze überprüfen. Columnstore-Indizes sind nicht für Singleton-Suchvorgänge (d.h. Suchen in einer einzelnen Zeile) geeignet. Wenn Sie häufig Singleton-Suchvorgänge ausführen müssen, können Sie einen nicht gruppierten Index einer Tabelle hinzufügen. Singleton-Suchvorgänge können mit einem nicht gruppierten Index erheblich schneller ausgeführt werden. Jedoch sind Singleton-Suchvorgänge in der Regel in Data Warehouse-Szenarien weniger gebräuchlich als OLTP-Workloads. Weitere Informationen finden Sie unter [Indizieren von Tabellen in SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-tables-index).

> [!NOTE]
> Gruppierte Columnstore-Tabellen unterstützen nicht die Datentypen `varchar(max)`, `nvarchar(max)` oder `varbinary(max)`. Ziehen Sie in diesem Fall einen Heap- oder gruppierten Index in Erwägung. Sie können diese Spalten in eine separate Tabelle einfügen.

Da die Beispieldatenbank nicht sehr groß ist, haben wir replizierte Tabellen ohne Partitionen erstellt. Bei Produktionsworkloads verbessert die Verwendung von verteilten Tabellen wahrscheinlich die Abfrageleistung. Siehe [Leitfaden für das Entwerfen verteilter Tabellen in Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-tables-distribute). Unsere Beispielskripts führen die Abfragen mithilfe einer statischen [Ressourcenklasse](/azure/sql-data-warehouse/resource-classes-for-workload-management) aus.

### <a name="load-the-semantic-model"></a>Laden des semantischen Modells

Laden Sie die Daten in ein tabellarisches Modell in Azure Analysis Services. In diesem Schritt erstellen Sie ein semantisches Datenmodell mithilfe von SQL Server Data Tools (SSDT). Sie können auch ein Modell erstellen, indem Sie es aus einer Power BI Desktop-Datei importieren. Da SQL Data Warehouse keine Fremdschlüssel unterstützt, müssen Sie dem Semantikmodell die Beziehungen hinzufügen, damit Sie eine tabellenübergreifende Verknüpfung durchführen können.

### <a name="use-power-bi-to-visualize-the-data"></a>Verwenden von Power BI zum Visualisieren von Daten

Power BI unterstützt zwei Optionen zum Herstellen einer Verbindung mit Azure Analysis Services:

- Importieren. Die Daten werden in das Power BI-Modell importiert.
- Liveverbindung. Daten werden direkt per Pull aus Analysis Services abgerufen.

Verwenden Sie die Liveverbindung, da sie kein Kopieren von Daten in das Power BI-Modell erfordert. Auch stellt die Verwendung von DirectQuery sicher, dass die Ergebnisse immer mit den neuesten Quelldaten konsistent sind. Weitere Informationen finden Sie unter [Herstellen einer Verbindung mit Power BI](/azure/analysis-services/analysis-services-connect-pbi).

**Empfehlungen**

Vermeiden Sie, BI-Dashboardabfragen direkt im Data Warehouse auszuführen. BI-Dashboards erfordern sehr kurze Antwortzeiten, die direkte Warehouse-Abfragen möglicherweise nicht leisten können. Darüber hinaus schränkt das Aktualisieren des Dashboards die Anzahl gleichzeitiger Abfragen ein, was sich auf die Leistung auswirken könnte. 

Azure Analysis Services dient zum Behandeln der Abfrageanforderungen eines BI-Dashboards, daher ist die empfohlene Vorgehensweise die Abfrage von Analysis Services über Power BI.

## <a name="scalability-considerations"></a>Überlegungen zur Skalierbarkeit

### <a name="sql-data-warehouse"></a>SQL Data Warehouse

Sie können mit SQL Data Warehouse Serverressourcen bedarfsabhängig horizontal hochskalieren. Das Abfragemodul optimiert Abfragen für die parallele Verarbeitung basierend auf der Anzahl von Serverknoten und verschiebt Daten nach Bedarf zwischen Knoten. Weitere Informationen finden Sie unter [Verwalten von Computeressourcen in Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview).

### <a name="analysis-services"></a>Analysis Services

Bei Produktionsworkloads sollten Sie den Standard-Tarif für Azure Analysis Services nutzen, da er Partitionierung und DirectQuery unterstützt. Innerhalb eines Tarifs bestimmt die Größe der Instanz Arbeitsspeicher und Verarbeitungsleistung. Die Verarbeitungsleistung wird in Query Processing Units (QPUs) gemessen. Beobachten Sie Ihre QPU-Nutzung, um die geeignete Größe auszuwählen. Weitere Informationen finden Sie unter [Überwachen von Servermetriken](/azure/analysis-services/analysis-services-monitor).

Bei hoher Last kann die Abfrageleistung durch parallele Abfragen beeinträchtigt werden. Sie können Analysis Services horizontal hochskalieren, indem Sie einen Pool von Replikaten zum Verarbeiten von Abfragen erstellen, sodass mehrere Abfragen gleichzeitig ausgeführt werden können. Die Verarbeitung des Datenmodells erfolgt immer auf dem primären Server. Standardmäßig behandelt der primäre Server auch Abfragen. Optional können Sie den primären Server ausschließlich zum Ausführen der Verarbeitung festlegen, sodass der Abfragepool alle Abfragen verarbeitet. Wenn Sie hohe Anforderungen an die Verarbeitung stellen, sollten Sie die Verarbeitung vom Abfragepool trennen. Bei hohen Abfragelasten und relativ unaufwändiger Verarbeitung können Sie den primären Server in den Abfragepool einschließen. Weitere Informationen finden Sie unter [Horizontales Hochskalieren von Azure Analysis Services](/azure/analysis-services/analysis-services-scale-out). 

Um unnötigen Verarbeitungsaufwand zu verringern, sollten Sie das tabellarische Modell mit Partitionen in logische Bereiche unterteilen. Jede Partition kann separat verarbeitet werden. Weitere Informationen finden Sie unter [Partitionen](/sql/analysis-services/tabular-models/partitions-ssas-tabular).

## <a name="security-considerations"></a>Überlegungen zur Sicherheit

### <a name="ip-whitelisting-of-analysis-services-clients"></a>IP-Positivlisten von Analysis Services-Clients

Erwägen Sie, mit dem Firewallfeature von Analysis Services Positivlisten von Client-IP-Adressen zu erstellen. Bei Aktivierung blockiert die Firewall alle Clientverbindungen, die nicht in den Firewallregeln angegeben sind. Die Standardregeln setzen den Power BI-Dienst auf die Positivliste, aber Sie können diese Regel falls gewünscht deaktivieren. Weitere Informationen finden Sie unter [Härtung von Azure Analysis Services mit der neuen Firewallfunktion](https://azure.microsoft.com/blog/hardening-azure-analysis-services-with-the-new-firewall-capability/).

### <a name="authorization"></a>Autorisierung

Azure Analysis Services authentifiziert mit Azure Active Directory (Azure AD) Benutzer, die eine Verbindung mit dem Analysis Services-Server herstellen. Sie können einschränken, welche Daten ein bestimmter Benutzer anzeigen kann, indem Sie Rollen erstellen und dann Azure AD-Benutzer oder Gruppen diesen Rollen zuweisen. Für jede Rolle können Sie: 

- Tabellen oder einzelne Spalten schützen. 
- Einzelne Zeilen basierend auf Filterausdrücken schützen. 

Weitere Informationen finden Sie unter [Verwalten von Datenbankrollen und Benutzern](/azure/analysis-services/analysis-services-database-users).

## <a name="deploy-the-solution"></a>Bereitstellen der Lösung

Eine Bereitstellung für diese Referenzarchitektur ist auf [GitHub][ref-arch-repo-folder] verfügbar. Folgendes wird bereitgestellt:

  * Eine Windows-VM, um einen lokalen Datenbankserver zu simulieren. Sie enthält SQL Server 2017 und zugehörige Tools zusammen mit Power BI Desktop.
  * Ein Azure Storage-Konto, das Blobspeicher zum Speichern von Daten bereitstellt, die aus SQL Server-Datenbank exportiert wurden.
  * Eine Instanz von Azure SQL Data Warehouse.
  * Eine Azure Analysis Services-Instanz.

### <a name="prerequisites"></a>Voraussetzungen

1. Klonen oder forken Sie das GitHub-Repository [Azure-Referenzarchitekturen][ref-arch-repo], oder laden Sie die zugehörige ZIP-Datei herunter.

2. Installieren Sie die [Azure-Bausteine][azbb-wiki] (azbb).

3. Melden Sie sich über eine Eingabeaufforderung, eine Bash-Eingabeaufforderung oder die PowerShell-Eingabeaufforderung bei Ihrem Azure-Konto an. Verwenden Sie hierzu den unten aufgeführten Befehl, und befolgen Sie die Anweisungen.

  ```bash
  az login  
  ```

### <a name="deploy-the-simulated-on-premises-server"></a>Bereitstellen des simulierten lokalen Servers

Zuerst stellen Sie einen virtuellen Computer als simulierten lokalen Server bereit, auf dem SQL Server 2017 und die zugehörigen Tools vorhanden sind. Dieser Schritt lädt auch die [Wide World Importers-Beispieldatenbanken für Microsoft SQL](/sql/sample/world-wide-importers/wide-world-importers-oltp-database) in SQL Server.

1. Navigieren Sie zum Ordner `data\enterprise-bi-sqldw\onprem\templates` des Repositorys, das Sie in den Voraussetzungen oben heruntergeladen haben.

2. Ersetzen Sie in der `onprem.parameters.json`-Datei die Werte für `adminUsername` und `adminPassword`. Ändern Sie auch die Werte im Abschnitt `SqlUserCredentials` mit entsprechendem Benutzernamen und Kennwort. Beachten Sie das `.\\`-Präfix in der userName-Eigenschaft.
    
    ```bash
    "SqlUserCredentials": {
      "userName": ".\\username",
      "password": "password"
    }
    ```

3. Führen Sie `azbb` wie unten dargestellt aus, um den lokalen Server bereitzustellen.

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <location> -p onprem.parameters.json --deploy
    ```

4. Die Bereitstellung kann inklusive der Ausführung des [DSC](/powershell/dsc/overview)-Skripts zum Installieren der Tools und Wiederherstellen der Datenbank 20 bis 30 Minuten dauern. Überprüfen Sie die Bereitstellung im Azure-Portal, indem Sie die Ressourcen in der Ressourcengruppe überprüfen. Die VM `sql-vm1` und die zugehörigen Ressourcen sollten angezeigt werden.

### <a name="deploy-the-azure-resources"></a>Bereitstellen der Azure-Ressourcen

Dieser Schritt stellt Azure SQL Data Warehouse und Azure Analysis Services zusammen mit einem Storage-Konto bereit. Wenn Sie möchten, können Sie diesen Schritt parallel zu dem vorherigen Schritt ausführen.

1. Navigieren Sie zum Ordner `data\enterprise-bi-sqldw\azure\templates` des Repositorys, das Sie in den Voraussetzungen oben heruntergeladen haben.

2. Führen Sie den folgenden Azure CLI-Befehl zum Erstellen einer Ressourcengruppe aus, wobei die angegebenen Parameter in Klammern ersetzt werden. Beachten Sie, dass Sie zum Bereitstellen eine andere Ressourcengruppe verwenden können als die, die Sie im vorherigen Schritt für den lokalen Server verwendet haben. 

    ```bash
    az group create --name <resource_group_name> --location <location>  
    ```

3. Führen Sie den folgenden Azure CLI-Befehl zum Bereitstellen der Azure-Ressourcen aus, wobei die angegebenen Parameter in Klammern ersetzt werden. Beim `storageAccountName`-Parameter müssen Sie die [Benennungsregeln](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) für Speicherkonten befolgen. Verwenden Sie für den `analysisServerAdmin`-Parameter Ihren Azure Active Directory-Benutzerprinzipalnamen (User Principal Name, UPN).

    ```bash
    az group deployment create --resource-group <resource_group_name> --template-file azure-resources-deploy.json --parameters "dwServerName"="<server_name>" "dwAdminLogin"="<admin_username>" "dwAdminPassword"="<password>" "storageAccountName"="<storage_account_name>" "analysisServerName"="<analysis_server_name>" "analysisServerAdmin"="user@contoso.com"
    ```

4. Überprüfen Sie die Bereitstellung im Azure-Portal, indem Sie die Ressourcen in der Ressourcengruppe überprüfen. Ein Speicherkonto, die Azure SQL Data Warehouse-Instanz und die Analysis Services-Instanz sollten angezeigt werden.

5. Rufen Sie den Zugriffsschlüssel für das Speicherkonto über das Azure-Portal ab. Wählen Sie das Speicherkonto aus, um es zu öffnen. Wählen Sie unter **Einstellungen** die Option **Zugriffsschlüssel** aus. Kopieren Sie den Primärschlüsselwert. Sie werden ihn im nächsten Schritt verwenden.

### <a name="export-the-source-data-to-azure-blob-storage"></a>Exportieren der Quelldaten in Azure Blob Storage 

In diesem Schritt führen Sie ein PowerShell-Skript aus, das mit BCP die SQL-Datenbank in Flatfiles auf dem virtuellen Computer exportiert und diese Dateien anschließend mit AzCopy in Azure Blob Storage kopiert.

1. Stellen Sie mit Remotedesktop eine Verbindung mit dem simulierten lokalen virtuellen Computer her.

2. Während Sie bei der VM angemeldet sind, führen Sie die folgenden Befehle in einem PowerShell-Fenster aus.  

    ```powershell
    cd 'C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\onprem'

    .\Load_SourceData_To_Blob.ps1 -File .\sql_scripts\db_objects.txt -Destination 'https://<storage_account_name>.blob.core.windows.net/wwi' -StorageAccountKey '<storage_account_key>'
    ```

    Ersetzen Sie für den `Destination`-Parameter `<storage_account_name>` mit dem Namen des Speicherkontos, das Sie zuvor erstellt haben. Verwenden Sie für den `StorageAccountKey`-Parameter den Zugriffsschlüssel für das Speicherkonto.

3. Stellen Sie im Azure-Portal sicher, dass die Quelldaten in den Blobspeicher kopiert wurden, indem zum Speicherkonto navigieren, den Blobdienst auswählen und den `wwi`-Container öffnen. Daraufhin sollte eine Liste von Tabellen mit vorangestelltem `WorldWideImporters_Application_*` angezeigt werden.

### <a name="execute-the-data-warehouse-scripts"></a>Ausführen der Data Warehouse-Skripts

1. Starten Sie SQL Server Management Studio (SSMS) über die Remotedesktopsitzung. 

2. Herstellen einer Verbindung mit SQL Data Warehouse

    - Servertyp: Datenbank-Engine
    
    - Servername: `<dwServerName>.database.windows.net`, wobei `<dwServerName>` der Name ist, den Sie beim Bereitstellen der Azure-Ressourcen angegeben haben. Sie können diesen Namen im Azure-Portal abrufen.
    
    - Authentifizierung: SQL Server-Authentifizierung. Verwenden Sie die Anmeldeinformationen, die Sie beim Bereitstellen der Azure-Ressourcen angegeben haben, in den Parametern `dwAdminLogin` und `dwAdminPassword`.

2. Navigieren Sie auf der VM zum Ordner `C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\azure\sqldw_scripts`. Führen Sie die Skripts in diesem Ordner in numerischer Reihenfolge aus, `STEP_1` bis `STEP_7`.

3. Wählen Sie die `master`-Datenbank in SSMS aus, und öffnen Sie das `STEP_1`-Skript. Ändern Sie den Wert des Kennworts in der folgenden Zeile, und führen Sie das Skript aus.

    ```sql
    CREATE LOGIN LoaderRC20 WITH PASSWORD = '<change this value>';
    ```

4. Wählen Sie die `wwi`-Datenbank in SSMS aus. Öffnen Sie das `STEP_2`-Skript, und führen Sie das Skript aus. Wenn Sie eine Fehlermeldung erhalten, stellen Sie sicher, dass Sie das Skript für die Datenbank `wwi` und nicht `master` ausführen.

5. Öffnen Sie eine neue Verbindung mit SQL Data Warehouse unter Verwendung des `LoaderRC20`-Benutzers und des im `STEP_1`-Skript angegebenen Kennworts.

6. Öffnen Sie mithilfe dieser Verbindung das `STEP_3`-Skript. Legen Sie die folgenden Werte im Skript fest:

    - SECRET: Verwenden Sie den Zugriffsschlüssel für Ihr Speicherkonto.
    - LOCATION: Verwenden Sie den Namen des Speicherkontos wie folgt: `wasbs://wwi@<storage_account_name>.blob.core.windows.net`.

7. Führen Sie mithilfe derselben Verbindung die Skripts `STEP_4` bis `STEP_7` sequenziell aus. Stellen Sie sicher, dass jedes Skript vor der Ausführung des nächsten erfolgreich abgeschlossen wird.

In SMSS sollte eine Reihe von `prd.*`-Tabellen in der `wwi`-Datenbank angezeigt werden. Führen Sie die folgende Abfrage aus, um sicherzustellen, dass die Daten generiert wurden: 

```sql
SELECT TOP 10 * FROM prd.CityDimensions
```

### <a name="build-the-azure-analysis-services-model"></a>Erstellen des Azure Analysis Services-Modells

In diesem Schritt erstellen Sie ein tabellarisches Modell, das Daten aus dem Data Warehouse importiert. Sie werden dann das Modell für Azure Analysis Services bereitstellen.

1. Starten Sie SQL Server Data Tools 2015 über die Remotedesktopsitzung.

2. Wählen Sie **Datei** > **Neu** > **Projekt**.

3. Wählen Sie im Dialogfeld **Neues Projekt** unter **Vorlagen** die Option **Business  Intelligence** > **Analysis Services** > **Analysis Services-Projekt für tabellarische Modelle** aus. 

4. Benennen Sie das Projekt, und klicken Sie auf **OK**.

5. Wählen Sie im Dialogfeld **Designer für tabellarische Modelle** die Option **Integrierter Arbeitsbereich** aus, und legen Sie für **Kompatibilitätsgrad** `SQL Server 2017 / Azure Analysis Services (1400)` fest. Klicken Sie auf **OK**.

6. Klicken Sie im Fenster **Tabellarischer Modell-Explorer** mit der rechten Maustaste auf das Projekt, und wählen Sie **Aus Datenquelle importieren** aus.

7. Wählen Sie **Azure SQL Data Warehouse** aus, und klicken Sie auf **Verbinden**.

8. Geben Sie für **Server** den vollqualifizierten Namen Ihres Azure SQL Data Warehouse-Servers ein. Geben Sie für **Datenbank** `wwi` ein. Klicken Sie auf **OK**.

9. Wählen Sie im nächsten Dialogfeld **Datenbank**-Authentifizierung aus, geben Sie Ihren Benutzernamen und Ihr Kennwort für Azure SQL Data Warehouse ein, und klicken Sie auf **OK**.

10. Aktivieren Sie im Dialogfeld **Navigator** die Kontrollkästchen für **prd.CityDimensions**, **prd.DateDimensions** und **prd.SalesFact**. 

    ![](./images/analysis-services-import.png)

11. Klicken Sie auf **Laden**. Wenn die Verarbeitung abgeschlossen ist, klicken Sie auf **Schließen**. Jetzt sollte eine tabellarische Ansicht der Daten angezeigt werden.

12. Klicken Sie im Fenster **Tabellarischer Modell-Explorer** mit der rechten Maustaste auf das Projekt, und wählen Sie **Modellansicht** > **Diagrammansicht** aus.

13. Ziehen Sie das Feld **[prd.SalesFact].[WWI City ID]** auf das Feld **[prd.CityDimensions].[WWI City ID]**, um eine Beziehung zu erstellen.  

14. Ziehen Sie das Feld **[prd.SalesFact].[Invoice Date Key]** auf das Feld **[prd.DateDimensions].[Date]**.  
    ![](./images/analysis-services-relations.png)

15. Klicken Sie im Menü **Datei** auf **Alle speichern**.  

16. Klicken Sie im **Projektmappen-Explorer** mit der rechten Maustaste auf das Projekt, und wählen Sie **Eigenschaften** aus. 

17. Geben Sie unter **Server** die URL Ihrer Azure Analysis Services-Instanz ein. Diesen Wert können Sie im Azure-Portal ermitteln. Wählen Sie im Portal die Analysis Services-Ressource aus, klicken Sie auf den Bereich „Übersicht“, und suchen Sie nach der Eigenschaft **Servername**. Sie ähnelt `asazure://westus.asazure.windows.net/contoso`. Klicken Sie auf **OK**.

    ![](./images/analysis-services-properties.png)

18. Klicken Sie im **Projektmappen-Explorer** mit der rechten Maustaste auf das Projekt, und wählen Sie **Bereitstellen** aus. Melden Sie sich bei Azure an, wenn Sie aufgefordert werden. Wenn die Verarbeitung abgeschlossen ist, klicken Sie auf **Schließen**.

19. Zeigen Sie im Azure-Portal die Details für Ihre Azure Analysis Services-Instanz an. Stellen Sie sicher, dass Ihr Modell in der Liste der Modelle angezeigt wird.

    ![](./images/analysis-services-models.png)

### <a name="analyze-the-data-in-power-bi-desktop"></a>Analysieren der Daten in Power BI Desktop

In diesem Schritt verwenden Sie Power BI zum Erstellen eines Berichts aus den Daten in Analysis Services.

1. Starten Sie Power BI Desktop über die Remotedesktopsitzung.

2. Klicken Sie auf dem Begrüßungsbildschirm auf **Daten abrufen**.

3. Wählen Sie **Azure** > **Azure Analysis Services-Datenbank** aus. Klicken Sie auf das Menü **Verbinden**

    ![](./images/power-bi-get-data.png)

4. Geben Sie die URL Ihrer Analysis Services-Instanz ein, und klicken Sie auf **OK**. Melden Sie sich bei Azure an, wenn Sie aufgefordert werden.

5. Erweitern Sie im Dialogfeld **Navigator** das Projekt für tabellarische Modelle, das Sie bereitgestellt haben, wählen Sie das Modell aus, das Sie erstellt haben, und klicken Sie auf **OK**.

2. Wählen Sie im Bereich **Visualisierungen** das Symbol **Gestapeltes Balkendiagramm** aus. Vergrößern Sie in der Berichtsansicht die Visualisierung.

6. Erweitern Sie im Bereich **Felder** **prd.CityDimensions**.

7. Ziehen Sie **prd.CityDimensions** > **WWI City ID** auf **Achse**.

8. Ziehen Sie **prd.CityDimensions** > **City** auf **Legende**.

9. Erweitern Sie im Bereich „Felder“ **prd.SalesFact**.

10. Ziehen Sie **prd.SalesFact** > **Total Excluding Tax** auf **Wert**.

    ![](./images/power-bi-visualization.png)

11. Wählen Sie unter **Filter auf visueller Ebene** die Option **WWI City ID** aus.

12. Legen Sie für **Filtertyp** `Top N` fest und für **Elemente anzeigen** `Top 10`.

13. Ziehen Sie **prd.SalesFact** > **Total Excluding Tax** auf **Nach Wert**.

    ![](./images/power-bi-visualization2.png)

14. Klicken Sie auf **Filter anwenden**. Die Visualisierung zeigt die obersten 10 Gesamtumsätze nach Stadt.

    ![](./images/power-bi-report.png)

Weitere Informationen zu Power BI Desktop finden Sie unter [Erste Schritte mit Power BI Desktop](/power-bi/desktop-getting-started).

## <a name="next-steps"></a>Nächste Schritte

- Weitere Informationen zu dieser Referenzarchitektur finden Sie in unserem [GitHub-Repository][ref-arch-repo-folder].
- Informieren Sie sich über die [Azure-Bausteine][azbb-repo].

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw

