---
title: Wiederherstellung nach Datenbeschädigung oder unbeabsichtigtem Löschen
description: In diesem Leitfaden erfahren Sie, wie Sie nach Datenbeschädigung oder versehentlichem Löschen eine Wiederherstellung ausführen, ausfallsichere, hochverfügbare und fehlertolerante Anwendungen erstellen und die Notfallwiederherstellung planen
author: MikeWasson
ms.date: 01/10/2018
ms.openlocfilehash: 76d2f996750d5a67b67bd5dc4977580f3b8abbc3
ms.sourcegitcommit: 3d6dba524cc7661740bdbaf43870de7728d60a01
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/11/2018
---
# <a name="recover-from-data-corruption-or-accidental-deletion"></a>Wiederherstellung nach Datenbeschädigung oder unbeabsichtigtem Löschen 

Teil eines zuverlässigen Plans für Geschäftskontinuität sind Maßnahmen für den Fall, dass Daten beschädigt oder versehentlich gelöscht werden. Im Anschluss finden Sie Informationen zur Wiederherstellung von Daten, die durch Anwendungs- oder Bedienerfehler beschädigt oder versehentlich gelöscht wurden.

## <a name="virtual-machines"></a>Virtual Machines

Verwenden Sie [Azure Backup](/azure/backup/), um virtuelle Azure-Computer vor Anwendungsfehlern oder versehentlichem Löschen zu schützen. Mit Azure Backup können Sie über mehrere VM-Datenträger hinweg konsistente Sicherungen erstellen. Darüber hinaus kann der Sicherungstresor über Regionen hinweg repliziert werden, um bei Ausfall einer Region eine Wiederherstellung zu ermöglichen.

## <a name="storage"></a>Speicher

Azure Storage bietet Datenresilienz durch automatisierte Replikate. Allerdings verhindert dies nicht die Beschädigung von Daten durch Anwendungscode oder Benutzer, ob versehentlich oder mit böser Absicht. Um Datengenauigkeit bei Anwendungs- oder Benutzerfehlern zu gewährleisten, sind weitergehende Methoden erforderlich – beispielsweise das Kopieren der Daten an einen sekundären Speicherort mit einem Überwachungsprotokoll. 

- **Blockblobs.** Erstellen Sie eine Zeitpunkt-Momentaufnahme von jedem Blockblob. Weitere Informationen finden Sie unter [Erstellen einer Momentaufnahme eines Blobs](/rest/api/storageservices/creating-a-snapshot-of-a-blob). Bei jeder Momentaufnahme wird Ihnen nur die erforderliche Speicherkapazität berechnet, die Sie benötigen, um die Veränderungen innerhalb des Blobs seit der letzten Momentaufnahme zu speichern. Die Momentaufnahmen sind von der Existenz des zugrunde liegenden Originalblobs abhängig, sodass ein Kopiervorgang in ein anderes Blob oder sogar in ein anderes Speicherkonto empfohlen wird. Dadurch sind die Sicherungsdaten dann ordnungsgemäß vor versehentlichem Löschen geschützt. Verwenden Sie [AzCopy](/azure/storage/common/storage-use-azcopy) oder [Azure PowerShell](/azure/storage/common/storage-powershell-guide-full), um die Blobs in ein anderes Speicherkonto zu kopieren.

- **Dateien.** Verwenden Sie [Freigabemomentaufnahmen (Vorschauversion)](/azure/storage/files/storage-how-to-use-files-snapshots) oder AzCopy bzw. PowerShell, um die Dateien in ein anderes Speicherkonto zu kopieren.

- **Tabellen.** Verwenden Sie AzCopy, um die Tabellendaten in ein anderes Speicherkonto in einer anderen Region zu exportieren.

## <a name="database"></a>Datenbank

### <a name="azure-sql-database"></a>Azure SQL-Datenbank 

SQL-Datenbank führt automatisch eine Kombination aus wöchentlichen vollständigen Datenbanksicherungen, stündlichen differenziellen Datenbanksicherungen sowie Transaktionsprotokollsicherungen im Abstand von fünf bis zehn Minuten durch, um Ihr Unternehmen vor Datenverlusten zu schützen. Verwenden Sie die Point-in-Time-Wiederherstellung, um eine Datenbank zu einem früheren Zeitpunkt wiederherzustellen. Weitere Informationen finden Sie unter 

- [Wiederherstellen einer Azure SQL-Datenbank mit automatisierten Datenbanksicherungen](/azure/sql-database/sql-database-recovery-using-backups)

- [Übersicht über die Geschäftskontinuität mit Azure SQL-Datenbank](/azure/sql-database/sql-database-business-continuity)

### <a name="sql-server-on-vms"></a>SQL Server auf virtuellen Computern

Für SQL Server auf virtuellen Computern gibt es zwei Optionen: herkömmliche Sicherungen und Protokollversand. Herkömmliche Sicherungen ermöglichen die Wiederherstellung auf einen bestimmten Zeitpunkt – allerdings ist der Wiederherstellungsprozess langsam. Um herkömmliche Sicherungen wiederherzustellen, müssen Sie mit einer vollständigen Erstsicherung beginnen und dann alle nachfolgend durchgeführten Sicherungen anwenden. Bei der zweiten Option wird eine Protokollversandsitzung konfiguriert, um die Wiederherstellung von Protokollsicherungen zu verzögern (beispielsweise um zwei Stunden). In dem entstandenen Zeitfenster können Fehler in der primären Datenbank behoben werden.

### <a name="azure-cosmos-db"></a>Azure Cosmos DB

Azure Cosmos DB erstellt in regelmäßigen Abständen automatisch Sicherungen. Sicherungskopien werden in einem anderen Speicherdienst getrennt gespeichert, und diese Sicherungen werden zum besseren Schutz vor regionalen Ausfällen global repliziert. Falls Sie Ihre Datenbank oder -sammlung versehentlich löschen, können Sie ein Supportticket anfordern oder den Azure Support bitten, die Daten aus der letzten automatischen Sicherung wiederherzustellen. Weitere Informationen finden Sie unter [Automatische Onlinesicherung und -wiederherstellung mit Azure Cosmos DB](/azure/cosmos-db/online-backup-and-restore).

### <a name="azure-database-for-mysql-azure-database-for-postresql"></a>Azure Database for MySQL, Azure Database for PostgreSQL

Bei der Verwendung von Azure Database for MySQL oder Azure Database for PostgreSQL erstellt der Datenbankdienst automatisch alle fünf Minuten eine Sicherung des Diensts. Mithilfe dieses automatischen Sicherungsfeatures können Sie einen früheren Zustand des Servers und aller seiner Datenbanken wiederherstellen. Weitere Informationen finden Sie unter 

- [Sichern und Wiederherstellen eines Servers in Azure Database for MySQL mit dem Azure-Portal](/azure/mysql/howto-restore-server-portal)

- [Sichern und Wiederherstellen eines Servers in Azure Database for PostgreSQL mit dem Azure-Portal](/azure/postgresql/howto-restore-server-portal)

