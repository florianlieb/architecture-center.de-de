---
title: Auswählen einer Datenübertragungstechnologie
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: bb0732b0f771a4c9e1a4e565875576c08484490a
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="transferring-data-to-and-from-azure"></a>Übertragen von Daten in und aus Azure

Daten können auf verschiedene Arten in und aus Azure übertragen werden. Für welche Option Sie sich entscheiden, hängt ganz von Ihren Anforderungen ab.

## <a name="physical-transfer"></a>Physische Übertragung

In folgenden Fällen empfiehlt sich die Verwendung physischer Hardware für die Übertragung von Daten in Azure:

- Ihr Netzwerk ist langsam oder unzuverlässig.
- Zusätzliche Netzwerkbandbreite ist zu teuer.
- Im Zusammenhang mit sensiblen Daten können aufgrund von Sicherheits- oder Organisationsrichtlinien keine ausgehenden Verbindungen verwendet werden. 

Wenn es Ihnen hauptsächlich auf die Übertragungsgeschwindigkeit Ihrer Daten ankommt, sollten Sie testen, ob die Netzwerkübertragung tatsächlich langsamer ist als der physische Transport.

Für den physischen Transport von Daten in Azure stehen zwei Hauptoptionen zur Verfügung:
- **Azure Import/Export:** Der [Azure Import/Export-Dienst](/azure/storage/common/storage-import-export-service) ermöglicht die sichere Übertragung großer Datenmengen in Azure Blob Storage oder Azure Files durch den Versand interner SATA-HDDs oder -SDDs an ein Azure-Datencenter. Mit diesem Dienst können Sie auch Daten aus Azure Storage auf Festplattenlaufwerke übertragen und sich diese zusenden lassen, um sie lokal zu laden.

- **Azure Data Box:** [Azure Data Box](https://azure.microsoft.com/services/storage/databox/) ist eine von Microsoft bereitgestellte Appliance, die ganz ähnlich funktioniert wie der Azure Import/Export-Dienst. Microsoft schickt Ihnen eine proprietäre, sichere und manipulationsgeschützte Übertragungsappliance und kümmert sich um die gesamte Logistik, was Sie über das Portal verfolgen können. Ein Vorteil von Azure Data Box ist die hohe Benutzerfreundlichkeit. Sie müssen nicht mehrere Festplatten kaufen, vorbereiten und die Dateien auf die einzelnen Festplatten übertragen. Azure Data Box wird von zahlreichen branchenführenden Azure-Partnern unterstützt, um in ihren Produkten den nahtlosen Offlinetransport in die Cloud zu erleichtern. 

## <a name="command-line-tools-and-apis"></a>Befehlszeilentools und APIs

Erwägen Sie die Verwendung dieser Optionen, wenn Sie eine skript- und programmgesteuerte Datenübertragung benötigen.

- **Azure-Befehlszeilenschnittstelle:** Die [Azure-Befehlszeilenschnittstelle](/azure/hdinsight/hdinsight-upload-data#commandline) ist ein plattformübergreifendes Tool, mit dem Sie Azure-Dienste verwalten und Daten in Azure Storage hochladen können. 

- **AzCopy:** Verwenden Sie AzCopy an einer Befehlszeile unter [Windows](/azure/storage/common/storage-use-azcopy?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) oder [Linux](/azure/storage/common/storage-use-azcopy-linux?toc=%2fazure%2fstorage%2fblobs%2ftoc.json), um Daten ganz einfach und mit optimaler Leistung aus bzw. in Azure Blob Storage, Azure File Storage oder Azure Table Storage zu kopieren. AzCopy unterstützt Nebenläufigkeit und Parallelität sowie die Fortsetzung unterbrochener Kopiervorgänge. Darüber hinaus ist AzCopy schneller als die meisten anderen Optionen. Für den programmgesteuerten Zugriff nutzt AzCopy die [Microsoft Azure Storage Data Movement-Bibliothek](/azure/storage/common/storage-use-data-movement-library) als Kernframework. Diese wird als .NET Core-Bibliothek bereitgestellt. 

- **PowerShell:** Das [PowerShell-Cmdlet `Start-AzureStorageBlobCopy`](/powershell/module/azure.storage/start-azurestorageblobcopy?view=azurermps-5.0.0) ist eine Option für Windows-Administratoren mit PowerShell-Erfahrung.  

- **AdlCopy:** [AdlCopy](/azure/data-lake-store/data-lake-store-copy-data-azure-storage-blob) ermöglicht das Kopieren von Daten aus Azure Storage Blob-Instanzen in Data Lake Store. Diese Option kann auch zum Kopieren von Daten zwischen zwei Azure Data Lake Store-Konten verwendet werden. Sie kann jedoch nicht verwendet werden, um Daten aus Data Lake Store in Speicherblobs zu kopieren.

- **Distcp:** Bei einem HDInsight-Cluster mit Data Lake Store-Zugriff können Sie Tools aus dem Hadoop-Ökosystem (beispielsweise [Distcp](/azure/data-lake-store/data-lake-store-copy-data-wasb-distcp)) verwenden, um Daten aus einem HDInsight-Clusterspeicher (WASB) in ein Data Lake Store-Konto zu kopieren (und umgekehrt).

- **Sqoop:** [Sqoop](/azure/hdinsight/hadoop/hdinsight-use-sqoop) ist ein Apache-Projekt und Teil des Hadoop-Ökosystems. Es ist auf allen HDInsight-Clustern vorinstalliert. Mit Sqoop können Sie Daten zwischen einem HDInsight-Cluster und relationalen Datenbanken wie SQL, Oracle und MySQL übertragen. Bei Sqoop handelt es sich um eine Sammlung verwandter Tools (einschließlich Import und Export). Sqoop kann mit HDInsight-Clustern verwendet werden – entweder unter Verwendung von Azure Storage Blob-Instanzen oder unter Verwendung von angefügtem Data Lake Store-Speicher.

- **PolyBase:** [PolyBase](/sql/relational-databases/polybase/get-started-with-polybase) ist eine Technologie, die über die T-SQL-Sprache auf Daten außerhalb der Datenbank zugreift. In SQL Server 2016 können Sie mit PolyBase Abfragen für externe Daten in Hadoop ausführen oder Daten aus Azure Blob Storage importieren/exportieren. In Azure SQL Data Warehouse können Sie Daten aus Azure Blob Storage und Azure Data Lake Store importieren/exportieren. PolyBase ist derzeit die schnellste Methode, um Daten in SQL Data Warehouse zu importieren.

- **Hadoop-Befehlszeile:** Wenn Sie über Daten verfügen, die sich im Hauptknoten eines HDInsight-Clusters befinden, können Sie diese mithilfe des Befehls `hadoop -copyFromLocal` in den angefügten Speicher Ihres Clusters (beispielsweise Azure Storage Blob oder Azure Data Lake Store) kopieren. Um den Hadoop-Befehl verwenden zu können, müssen Sie zunächst eine Verbindung mit dem Hauptknoten herstellen. Anschließend können Sie eine Datei in den Speicher hochladen.

## <a name="graphical-interface"></a>Grafische Benutzeroberfläche

Erwägen Sie die Verwendung folgender Optionen, wenn Sie nur wenige Dateien oder Datenobjekte übertragen und den Vorgang nicht automatisieren müssen.

- **Azure Storage-Explorer:** Der [Azure Storage-Explorer](https://azure.microsoft.com/features/storage-explorer/) ist ein plattformübergreifendes Tool zur Verwaltung der Inhalte Ihrer Azure-Speicherkonten. Mit diesem Tool können Sie Blobs, Dateien, Warteschlangen, Tabellen und Azure Cosmos DB-Entitäten hochladen, herunterladen und verwalten. Verwenden Sie es zusammen mit Blob Storage, um Blobs und Ordner zu verwalten und Blobs zwischen Ihrem lokalen Dateisystem und Blob Storage oder zwischen Speicherkonten hoch- und herunterzuladen.

- **Azure-Portal:** Sowohl Blob Storage als auch Data Lake Store verfügt über eine webbasierte Schnittstelle zum Erkunden von Dateien sowie zum Hochladen einzelner neuer Dateien. Diese Option empfiehlt sich, wenn Sie Ihre Dateien schnell erkunden möchten, ohne Tools zu installieren oder Befehle auszuführen, oder einfach einige neue Dateien hochladen möchten.

## <a name="data-pipeline"></a>Datenpipeline

**Azure Data Factory:** Der verwaltete Dienst [Azure Data Factory](/azure/data-factory/) eignet sich am besten zur regelmäßigen Übertragung von Dateien – zwischen einer Reihe von Azure-Diensten, lokal oder zwischen einer Kombination aus beidem. Mit Azure Data Factory können Sie datengesteuerte Workflows (so genannte Pipelines) erstellen und planen, die Daten aus unterschiedlichen Datenspeichern erfassen. Die Anwendung kann diese Daten mithilfe von Compute Services wie Azure HDInsight Hadoop, Spark, Azure Data Lake Analytics und Azure Machine Learning verarbeiten und transformieren. Erstellen Sie datengesteuerte Workflows zur [Orchestrierung](../technology-choices/pipeline-orchestration-data-movement.md) und Automatisierung der Verschiebung und Transformation von Daten.

## <a name="key-selection-criteria"></a>Wichtige Auswahlkriterien

Die folgenden Fragen unterstützen Sie bei der Ermittlung eines geeigneten Systems für Ihre Datenübertragungsszenarien:

- Müssen Sie sehr große Datenmengen übertragen, was über eine Internetverbindung zu lange dauern würde oder unzuverlässig/zu teuer wäre? Falls ja, empfiehlt sich die physische Übertragung.

- Bevorzugen Sie die Verwendung skriptgesteuerter (und somit wiederverwendbarer) Datenübertragungsaufgaben? Falls ja, entscheiden Sie sich für eine der Befehlszeilenoptionen oder für Azure Data Factory.

- Müssen Sie eine große Datenmenge über eine Netzwerkverbindung übertragen? Falls ja, wählen Sie eine für Big Data optimierte Option.

- Müssen Sie Daten in eine relationale Datenbank oder aus einer relationalen Datenbank übertragen? Falls ja, entscheiden Sie sich für eine Option, die mindestens eine relationale Datenbank unterstützt. Beachten Sie, dass einige dieser Optionen auch einen Hadoop-Cluster erfordern.

- Benötigen Sie eine automatisierte Datenpipeline oder Workfloworchestrierung? Falls ja, empfiehlt sich die Verwendung von Azure Data Factory.

## <a name="capability-matrix"></a>Funktionsmatrix

In den folgenden Tabellen sind die Hauptunterschiede der Funktionen zusammengefasst:

### <a name="physical-transfer"></a>Physische Übertragung

| | Azure Import/Export-Dienst | Azure Data Box |
| --- | --- | --- |
| Formfaktor | Interne SATA-HDDs oder -SDDs | Einzelne sichere und manipulationsgeschützte Hardwareappliance |
| Von Microsoft verwaltete Versandlogistik | Nein | Ja |
| Integration in Partnerprodukte | Nein | Ja |
| Angepasste Appliance | Nein | Ja |

### <a name="command-line-tools"></a>Befehlszeilentools

**Hadoop/HDInsight**

| | Distcp | Sqoop | Hadoop-Befehlszeilenschnittstelle |
| --- | --- | --- | --- |
| Für Big Data optimiert | Ja | Ja |  Ja |
| Kopieren in relationale Datenbank |  Nein | Ja | Nein |
| Kopieren aus relationaler Datenbank |  Nein | Ja | Nein |
| Kopieren in Blob Storage |  Ja | Ja | Ja |
| Kopieren aus Blob Storage | Ja |  Ja | Nein |
| Kopieren in Data Lake Store | Ja | Ja | Ja |
| Kopieren aus Data Lake Store | Ja | Ja | Nein |

**Andere**

| | Azure-Befehlszeilenschnittstelle | AzCopy | PowerShell | AdlCopy | PolyBase |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Kompatible Plattformen | Linux, OS X, Windows | Linux, Windows | Windows | Linux, OS X, Windows | SQL Server, Azure SQL Data Warehouse | 
| Für Big Data optimiert | Nein | Nein | Nein | Ja<sup>1</sup> | Ja<sup>2</sup> |
| Kopieren in relationale Datenbank | Nein | Nein | Nein | Nein | Ja | 
| Kopieren aus relationaler Datenbank | Nein | Nein | Nein | Nein | Ja | 
| Kopieren in Blob Storage | Ja | Ja | Ja | Nein | Ja | 
| Kopieren aus Blob Storage | Ja | Ja | Ja | Ja | Ja |
| Kopieren in Data Lake Store | Nein | Nein | Ja | Ja |  Ja | 
| Kopieren aus Data Lake Store | Nein | Nein | Ja | Ja | Ja | 


[1] AdlCopy ist bei Verwendung mit einem Data Lake Analytics-Konto für die Übertragung von Big Data optimiert.

[2] Für PolyBase [kann die Leistung verbessert werden](/sql/relational-databases/polybase/polybase-guide#performance), indem die Berechnung mithilfe von Push an Hadoop übertragen und durch die Verwendung von [PolyBase-Erweiterungsgruppen](/sql/relational-databases/polybase/polybase-scale-out-groups) die parallele Datenübertragung zwischen SQL Server-Instanzen und Hadoop-Knoten ermöglicht wird.

### <a name="graphical-interface-and-azure-data-factory"></a>Grafische Benutzeroberfläche und Azure Data Factory

| | Azure Storage-Explorer | Azure-Portal* | Azure Data Factory |
| --- | --- | --- | --- |
| Für Big Data optimiert | Nein | Nein | Ja | 
| Kopieren in relationale Datenbank | Nein | Nein | Ja |
| Kopieren in relationale Datenbank | Nein | Nein | Ja |
| Kopieren in Blob Storage | Ja | Nein | Ja |
| Kopieren aus Blob Storage | Ja | Nein | Ja |
| Kopieren in Data Lake Store | Nein | Nein | Ja |
| Kopieren aus Data Lake Store | Nein | Nein | Ja |
| Hochladen in Blob Storage | Ja | Ja | Ja |
| Hochladen in Data Lake Store | Ja | Ja | Ja |
| Orchestrieren von Datenübertragungen | Nein | Nein | Ja |
| Benutzerdefinierte Datentransformationen | Nein | Nein | Ja |
| Preismodell | Kostenlos | Kostenlos | Nutzungsbasierte Bezahlung |

\* Azure-Portal bedeutet in diesem Fall die Verwendung der webbasierten Erkundungstools für Blob Storage und Data Lake Store.

