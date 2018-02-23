---
title: "Erweitern lokaler Datenlösungen auf die Cloud"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 66fc225dc123202ba587d82f15ea0883e1bbf3b5
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="extending-on-premises-data-solutions-to-the-cloud"></a>Erweitern lokaler Datenlösungen auf die Cloud

Wenn Organisationen Workloads und Daten in die Cloud verlagern, spielen ihre lokalen Datencenter häufig weiterhin eine wichtige Rolle. Der Begriff *Hybrid Cloud* bezieht sich auf eine Kombination aus Public Cloud und lokalen Rechenzentren, um eine integrierte IT-Umgebung zu schaffen, die beides umfasst. Manche Organisationen verwenden Hybrid Cloud, um ihr gesamtes Datencenter nach und nach in die Cloud zu migrieren. Andere Organisationen nutzen Clouddienste zur Erweiterung ihrer vorhandenen lokalen Infrastruktur. 

In diesem Artikel werden einige Überlegungen und bewährte Methoden im Zusammenhang mit der Verwaltung von Daten in einer Hybrid Cloud-Lösung beschrieben.

## <a name="when-to-use-a-hybrid-solution"></a>Szenarien für eine Hybridlösung

Eine Hybridlösung kann in folgenden Szenarien verwendet werden:

* Als Übergangsstrategie während einer längerfristigen Migration zu einer vollständig nativen Cloudlösung
* Wenn bestimmte Daten oder Workloads aufgrund von Bestimmungen oder Richtlinien nicht in die Cloud verlagert werden dürfen
* Für Notfallwiederherstellung und Fehlertoleranz durch Replikation von Daten und Diensten zwischen lokalen und cloudbasierten Umgebungen
* Zur Verringerung der Wartezeit zwischen Ihrem lokalen Rechenzentrum und Remotestandorten durch Hosten eines Teils Ihrer Architektur in Azure

## <a name="challenges"></a>Herausforderungen

* Einrichtung einer Umgebung mit konsistenter Sicherheit, Verwaltung und Entwicklung sowie Vermeidung der mehrfachen Ausführung von Arbeiten

* Einrichtung einer zuverlässigen, sicheren Datenverbindung mit geringer Wartezeit zwischen lokaler und cloudbasierter Umgebung

* Replikation Ihrer Daten und Anpassung von Anwendungen und Tools zur Verwendung der korrekten Datenspeicher innerhalb der jeweiligen Umgebung

* Schutz und Verschlüsselung von in der Cloud gehosteten Daten, auf die über die lokale Umgebung zugegriffen wird (oder umgekehrt)

## <a name="on-premises-data-stores"></a>Lokale Datenspeicher

Zu lokalen Datenspeichern zählen Datenbanken und Dateien. Es sind verschiedene Gründe denkbar, warum diese in der lokalen Umgebung verbleiben müssen. Bestimmte Daten oder Workloads dürfen möglicherweise aufgrund von Bestimmungen oder Richtlinien nicht in die Cloud verlagert werden. Eine lokale Platzierung kann aufgrund von Bedenken im Bereich Datenhoheit, Datenschutz oder Datensicherheit bevorzugt werden. Während einer Migration sollen möglicherweise einige Daten für eine noch nicht migrierte Anwendung in der lokalen Umgebung verbleiben.

Berücksichtigen Sie folgende Aspekte, wenn Sie Anwendungsdaten in einer Public Cloud platzieren möchten:

* **Kosten:** Die Kosten für Speicher in Azure können deutlich geringer sein als die Kosten für Speicher mit ähnlichen Eigenschaften in einem lokalen Datencenter. Da viele Unternehmen natürlich bereits in High-End-SANs investiert haben, kommen diese Kostenvorteile unter Umständen erst so richtig zur Geltung, wenn die vorhandene Hardware älter wird.
* **Elastische Skalierung:** Die Planung und Bewältigung von Datenkapazitätszuwächsen in einer lokalen Umgebung kann eine echte Herausforderung sein. Dies gilt insbesondere dann, wenn das Datenwachstum nur schwer vorhersagbar ist. In diesem Fall können Anwendungen von der bedarfsgerechten Kapazität sowie vom nahezu unbegrenzten Speicherplatz in der Cloud profitieren. Für Anwendungen mit Datasets, deren Größe sich nicht sonderlich stark verändert, ist dieser Aspekt weniger relevant.
* **Notfallwiederherstellung:** In Azure gespeicherte Daten können automatisch innerhalb einer Azure-Region und über verschiedene geografische Regionen hinweg repliziert werden. In Hybridumgebungen können diese Technologien für die Replikation zwischen lokalen und cloudbasierten Datenspeichern verwendet werden.

## <a name="extending-data-stores-to-the-cloud"></a>Erweitern von Datenspeichern auf die Cloud

Lokale Datenspeicher können auf verschiedene Weise auf die Cloud erweitert werden. Eine Option ist die Verwendung lokaler und cloudbasierter Replikate. Dadurch kann zwar eine hohe Fehlertoleranz erreicht werden, ggf. sind jedoch Änderungen an Anwendungen erforderlich, damit im Falle eines Failovers eine Verbindung mit dem richtigen Datenspeicher hergestellt wird.

Eine andere Möglichkeit ist die Verlagerung eines Teils der Daten in den Cloudspeicher, während die aktuelleren oder häufiger verwendeten Daten in der lokalen Umgebung verbleiben. Diese Methode kann eine kostengünstigere Option für die langfristige Speicherung darstellen sowie die Antwortzeiten beim Datenzugriff durch Verkleinerung Ihres operativen Datasets verbessern.

Eine dritte Möglichkeit wäre, alle Daten in der lokalen Umgebung zu belassen und Cloud Computing für das Hosten von Anwendungen zu verwenden. In diesem Fall wird die Anwendung in der Cloud gehostet und über eine sichere Verbindung mit Ihrem lokalen Datenspeicher verknüpft. 

## <a name="azure-stack"></a>Azure Stack

[Microsoft Azure Stack](/azure/azure-stack/) kann als umfassende Hybrid Cloud-Lösung verwendet werden. Azure Stack ist eine Hybrid Cloud-Plattform, mit der Sie Azure-Dienste aus Ihrem Datencenter bereitstellen können. Dies trägt zur Wahrung der Konsistenz zwischen lokaler Umgebung und Azure bei, da hierbei identische Tools verwendet werden und keine Codeänderungen erforderlich sind. 

Im Anschluss folgen einige Anwendungsfälle für Azure und Azure Stack:

* **Edgelösungen und nicht vernetzte Lösungen:** Werden Sie den Anforderungen im Hinblick auf Wartezeit und Konnektivität gerecht, indem Sie Daten lokal in Azure Stack verarbeiten und dann zur weiteren Analyse in Azure aggregieren, und verwenden Sie dabei in beiden Systemen eine gemeinsame Anwendungslogik. 
* **Cloudanwendungen, die verschiedene Bestimmungen erfüllen:** Entwickeln Sie Anwendungen in Azure, und stellen Sie sie dort bereit – mit der Flexibilität, die gleichen Anwendungen lokal in Azure Stack bereitzustellen, um gesetzliche oder richtlinienbasierte Anforderungen zu erfüllen.
* **Lokales Cloudanwendungsmodell:** Verwenden Sie Azure, um vorhandene Anwendungen zu aktualisieren und zu erweitern oder um neue Anwendungen zu erstellen. Nutzen Sie konsistente DevOps-Prozesse in Azure (in der Cloud) sowie lokal (in Azure Stack).

## <a name="sql-server-data-stores"></a>SQL Server-Datenspeicher

Wenn Sie SQL Server lokal ausführen, können Sie den Microsoft Azure Blob Storage-Dienst zum Sichern und Wiederherstellen verwenden. Weitere Informationen finden Sie unter [SQL Server-Sicherung und -Wiederherstellung mit dem Microsoft Azure Blob Storage Service](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service). Dadurch erhalten Sie unbegrenzten externen Speicherplatz und können die gleichen Sicherungen sowohl für die lokal ausgeführte SQL Server-Instanz als auch für eine auf einem virtuellen Computer in Azure ausgeführte SQL Server-Instanz verwenden. 

[Azure SQL-Datenbank](/azure/sql-database/) ist eine verwaltete relationale Datenbank als Dienst (Database-as-a-Service, DaaS). Da Azure SQL-Datenbank das Microsoft SQL Server-Modul verwendet, können Anwendungen mit beiden Technologien auf die gleiche Weise auf Daten zugreifen. Azure SQL-Datenbank kann praktischerweise auch mit SQL Server kombiniert werden. So kann eine Anwendung beispielsweise mit dem Feature [SQL Server Stretch Database](/sql/sql-server/stretch-database/stretch-database) auf ein Element zugreifen, das wie eine einzelne Tabelle in einer SQL Server-Datenbank aussieht, während einige oder alle Zeilen dieser Tabelle unter Umständen in Azure SQL-Datenbank gespeichert sind. Daten, die für einen bestimmten Zeitraum nicht genutzt wurden, werden von dieser Technologie automatisch in die Cloud verschoben. Für Anwendungen, die diese Daten lesen, ist nicht feststellbar, dass Daten in die Cloud verschoben wurden.

Die Verwaltung von Datenspeichern in der lokalen Umgebung und in der Cloud ist nicht immer einfach, wenn die Daten synchronisiert bleiben sollen. Hierzu können Sie [SQL-Datensynchronisierung](/azure/sql-database/sql-database-sync-data) verwenden – einen Dienst, der auf Azure SQL-Datenbank basiert und mit dem Sie die ausgewählten Daten bidirektional über mehrere Azure SQL-Datenbanken und SQL Server-Instanzen hinweg synchronisieren können. Mit der Datensynchronisierung können Sie zwar problemlos Daten in verschiedenen Datenspeichern auf dem neuesten Stand halten, sie sollte jedoch nicht für die Notfallwiederherstellung oder für die Migration von der lokalen SQL Server-Instanz zu Azure SQL-Datenbank verwendet werden.

Für die Notfallwiederherstellung und Geschäftskontinuität können Sie [AlwaysOn-Verfügbarkeitsgruppen](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server) verwenden, um Daten zwischen mehreren Instanzen von SQL Server zu replizieren, die ggf. auf virtuellen Azure-Computern in einer anderen geografischen Region ausgeführt werden.

## <a name="network-shares-and-file-based-data-stores"></a>Netzwerkfreigaben und dateibasierte Datenspeicher

In einer Hybrid Cloud-Architektur behalten Organisation neuere Dateien meist in der lokalen Umgebung und archivieren ältere Dateien in der Cloud. Dies wird gelegentlich als Datei-Tiering bezeichnet. Hierbei kann nahtlos auf beide Sätze von Dateien (lokal und in der Cloud gehostet) zugegriffen werden. Dieser Ansatz minimiert die Auslastung der Netzwerkbandbreite und beschleunigt den Zugriff auf neuere Dateien, die wahrscheinlich häufiger genutzt werden. Gleichzeitig profitieren Sie bei archivierten Daten von den Vorteilen des cloudbasierten Speichers. 

Manche Organisationen möchten ggf. auch ihre Netzwerkfreigaben vollständig in die Cloud verlagern. Dies kann beispielsweise wünschenswert sein, wenn sich die Anwendungen, die darauf zugreifen, ebenfalls in der Cloud befinden. Zur Implementierung können Tools für die [Datenorchestrierung](../technology-choices/pipeline-orchestration-data-movement.md) verwendet werden.


[Azure StorSimple](/azure/storsimple/) bietet die umfassendste integrierte Speicherlösung zur Verwaltung von Speicheraufgaben zwischen lokalen Geräten und Azure-Cloudspeicher. StorSimple ist eine effiziente, kostengünstige und einfach zu verwaltende Storage Area Network (SAN)-Lösung, mit der viele der Probleme und Kosten vermieden werden, die mit der Datenspeicherung und dem Datenschutz im Unternehmen verbunden sind. Die Lösung verwendet das proprietäre Gerät der StorSimple 8000-Serie, kann in Clouddienste integriert werden und stellt einen Satz integrierter Verwaltungstools bereit.

Eine weitere Möglichkeit zur Verwendung lokaler Netzwerkfreigaben mit cloudbasiertem Dateispeicher ist [Azure Files](/azure/storage/files/storage-files-introduction). Azure Files bietet vollständig verwaltete Dateifreigaben, auf die Sie über das standardmäßige [SMB-Protokoll](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx?f=255&MSPPError=-2147217396) (Server Message Block; manchmal auch CIFS genannt) zugreifen können. Azure Files kann als Dateifreigabe auf Ihrem lokalen Computer eingebunden oder mit vorhandenen Anwendungen verwendet werden, die auf lokale Dateien oder auf Dateien in einer Netzwerkfreigabe zugreifen.

Dateifreigaben in Azure Files können mithilfe von [Azure File Sync](/azure/storage/files/storage-sync-files-planning) mit Ihren lokalen Windows-Servern synchronisiert werden. Mit Azure File Sync können Sie Dateien zwischen Ihrem lokalen Dateiserver und Azure Files aufteilen, was ein großer Vorteil ist. So können Sie sicherstellen, dass nur die neuesten und zuletzt verwendeten Dateien in der lokalen Umgebung verbleiben. 

Weitere Informationen finden Sie unter [Entscheidung zwischen Azure-Blobs, Azure Files und Azure-Datenträger](/azure/storage/common/storage-decide-blobs-files-disks).

## <a name="hybrid-networking"></a>Hybrid Networking

In diesem Artikel standen Hybriddatenlösungen im Mittelpunkt. Eine weitere Überlegung ist jedoch, wie Sie Ihr lokales Netzwerk auf Azure erweitern können. Weitere Informationen zu diesem Aspekt von Hybridlösungen finden Sie in den folgenden Themen:

- [Auswählen einer Lösung zum Herstellen einer Verbindung zwischen einem lokalen Netzwerk und Azure](../../reference-architectures/hybrid-networking/considerations.md)
- [Verbinden eines lokalen Netzwerks mit Azure](../../reference-architectures/hybrid-networking/index.md)

