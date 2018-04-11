---
title: Checkliste für Resilienz für Azure-Dienste
description: Checkliste mit einer Anleitung zur Resilienz für verschiedene Azure-Dienste.
author: petertaylor9999
ms.date: 03/02/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: 25d961d6bb753b1f515fc073e51bbb912cc59db7
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/08/2018
---
# <a name="resiliency-checklist-for-specific-azure-services"></a>Checkliste für Resilienz für bestimmte Azure-Dienste

Resilienz ist die Fähigkeit des Systems, nach Ausfällen für ein System eine Wiederherstellung durchzuführen und die Betriebsbereitschaft sicherzustellen. Sie gehört zu den [Säulen der Softwarequalität](../guide/pillars.md). Jede Technologie verfügt über ihre eigenen speziellen Fehlermodi, die Sie beim Entwerfen und Implementieren Ihrer Anwendung berücksichtigen müssen. Verwenden Sie diese Checkliste, um die Resilienzaspekte für bestimmte Azure-Dienste zu überprüfen. Verwenden Sie auch die [allgemeine Checkliste zur Resilienz](./resiliency.md).

## <a name="app-service"></a>App Service

**Verwenden Sie den Standard- oder Premium-Tarif.** Diese Tarife unterstützen Stagingslots und automatisierte Sicherungen. Weitere Informationen hierzu finden Sie unter [Azure App Service-Pläne – Detaillierte Übersicht](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/).

**Vermeiden Sie zentrales Hoch- oder Herunterskalieren.** Wählen Sie stattdessen einen Tarif und eine Instanzgröße aus, der bzw. die Ihre Leistungsanforderungen unter typischer Last erfüllt, und [skalieren Sie die Instanzen horizontal hoch](/azure/app-service-web/web-sites-scale/), um Änderungen beim Datenverkehrsvolumen zu bewältigen. Durch zentrales Hoch- oder Herunterskalieren kann ein Neustart der Anwendung ausgelöst werden.  

**Speichern Sie die Konfiguration als App-Einstellungen.** Verwenden Sie App-Einstellungen, um die Konfigurationseinstellungen als App-Einstellungen zu speichern. Definieren Sie die Einstellungen in den Resource Manager-Vorlagen oder mithilfe von PowerShell, sodass Sie sie als Teil einer automatisierten Bereitstellung/Aktualisierung anwenden können, um die Zuverlässigkeit zu erhöhen. Weitere Informationen finden Sie unter [Konfigurieren von Web-Apps in Azure App Service](/azure/app-service-web/web-sites-configure/).

**Erstellen Sie separate App Service-Pläne für die Produktion und für Tests.** Verwenden Sie keine Slots Ihrer Produktionsbereitstellung zu Testzwecken.  Alle Apps in einem App Service-Plan nutzen die gleichen VM-Instanzen. Wenn Sie Produktions- und Testbereitstellungen in den gleichen Plan aufnehmen, kann sich dies negativ auf die Produktionsbereitstellung auswirken. Auslastungstests können z.B. den aktiven Produktionsstandort beeinträchtigen. Wenn Sie Testbereitstellungen in einen separaten Plan aufnehmen, isolieren Sie sie von der Produktionsversion.  

**Trennen Sie Web-Apps von Web-APIs.** Wenn zu Ihrer Lösung ein Web-Front-End und eine Web-API gehören, sollten Sie sie in getrennte App Service-Apps aufnehmen. Dieser Entwurf erleichtert es, die Lösung nach Workloads zu zerlegen. Sie können dann die Web-App und die API in separaten App Service-Plänen ausführen, sodass sie unabhängig skaliert werden können. Wenn Sie diesen Grad an Skalierbarkeit anfänglich nicht benötigen, können Sie die Apps im gleichen Plan bereitstellen und sie später bei Bedarf in separate Pläne verschieben.

**Vermeiden Sie die Verwendung der App Service-Sicherungsfunktion, um Azure SQL-Datenbanken zu sichern.** Verwenden Sie stattdessen [automatisierte SQL-Datenbanksicherungen][sql-backup]. Bei der App Service-Sicherung wird die Datenbank in eine SQL-BACPAC-Datei exportiert, und dies kostet DTUs.  

**Führen Sie die Bereitstellung in einem Stagingslot durch.** Erstellen Sie einen Bereitstellungsslot für das Staging. Stellen Sie Anwendungsupdates im Stagingslot bereit, und überprüfen Sie die Bereitstellung, bevor Sie sie in die Produktion übertragen. Dies reduziert die Wahrscheinlichkeit fehlerhafter Updates in der Produktion. Darüber hinaus wird sichergestellt, dass alle Instanzen vorbereitet sind, bevor sie in die Produktion übertragen werden. Viele Anwendungen weisen eine erhebliche Aufwärmdauer und Kaltstartzeit auf. Weitere Informationen finden Sie unter [Einrichten von Stagingumgebungen für Web-Apps in Azure App Service](/azure/app-service-web/web-sites-staged-publishing/).

**Erstellen Sie einen Bereitstellungsslot, um die letzte als funktionierend bekannte (LKG) Bereitstellung zu speichern.** Wenn Sie ein Update in der Produktion bereitstellen, verschieben Sie die vorherige Produktionsbereitstellung in den LKG-Slot. Dies erleichtert die Zurücksetzung einer fehlerhaften Bereitstellung. Wenn Sie später ein Problem feststellen, können Sie schnell die LKG-Version wiederherstellen. Weitere Informationen finden Sie unter [Einfache Webanwendung](../reference-architectures/app-service-web-app/basic-web-app.md).

**Aktivieren Sie die Diagnoseprotokollierung**, einschließlich der Anwendungsprotokollierung und der Webserverprotokollierung. Protokollierung ist wichtig für die Überwachung und die Diagnose. Weitere Informationen finden Sie unter [Aktivieren der Diagnoseprotokollierung für Web-Apps in Azure App Service](/azure/app-service-web/web-sites-enable-diagnostic-log/).

**Speichern Sie Protokolle im Blobspeicher.** Dies erleichtert das Erfassen und Analysieren der Daten.

**Erstellen Sie ein separates Speicherkonto für Protokolle.** Verwenden Sie nicht das gleiche Speicherkonto für Protokolle und Anwendungsdaten. Dadurch wird verhindert, dass die Protokollierung die Anwendungsleistung verringert.

**Überwachen Sie die Leistung.** Verwenden Sie einen Leistungsüberwachungsdienst wie [New Relic](http://newrelic.com/) oder [Application Insights](/azure/application-insights/app-insights-overview/), um Leistung und Verhalten der Anwendung unter Last zu überwachen.  Durch die Leistungsüberwachung erhalten Sie in Echtzeit Einblicke in die Anwendung. Dadurch können Sie Probleme diagnostizieren und eine Ursachenanalyse von Fehlern durchführen.

## <a name="application-gateway"></a>Application Gateway

**Stellen Sie mindestens zwei Instanzen bereit.** Stellen Sie Application Gateway mit mindestens zwei Instanzen bereit. Eine einzelne Instanz ist ein Single Point of Failure. Verwenden Sie zwei oder mehr Instanzen für Redundanz und Skalierbarkeit. Um sich für die [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/) zu qualifizieren, müssen Sie zwei oder mehr mittelgroße oder größere Instanzen bereitstellen.

## <a name="cosmos-db"></a>Cosmos DB

**Replizieren Sie die Datenbank zwischen Regionen.** Mit Cosmos DB können Sie einem Cosmos DB-Datenbankkonto eine beliebige Anzahl von Azure-Regionen zuordnen. Eine Cosmos-DB-Datenbank kann eine Schreibregion und mehrere Leseregionen aufweisen. Bei einem Fehler in der Schreibregion können Sie ein anderes Replikat für Lesezugriff verwenden. Das Client-SDK verarbeitet dies automatisch. Sie können auch ein Failover der Schreibregion auf eine andere Region ausführen. Weitere Informationen finden Sie unter [Wie werden Daten mit Azure Cosmos DB global verteilt?](/azure/cosmos-db/distribute-data-globally).

## <a name="event-hubs"></a>Event Hubs

**Verwenden Sie Prüfpunkte.**  Ein Ereignisconsumer sollte seine aktuelle Position in vordefinierten Intervallen in einen beständigen Speicher schreiben. Falls für den Consumer ein Fehler auftritt (z.B. ein Absturz des Consumers oder Ausfall des Hosts), kann eine neue Instanz das Lesen des Datenstroms dann ab der zuletzt aufgezeichneten Position fortsetzen. Weitere Informationen finden Sie unter [Ereignisconsumer](/azure/event-hubs/event-hubs-features#event-consumers).

**Regeln Sie den Umgang mit doppelten Nachrichten.** Wenn ein Ereignisconsumer ausfällt, wird die Nachrichtenverarbeitung ab dem letzten aufgezeichneten Prüfpunkt fortgesetzt. Alle Nachrichten, die nach dem letzten Prüfpunkt bereits verarbeitet wurden, werden erneut verarbeitet. Aus diesem Grund muss Ihre Logik für die Nachrichtenverarbeitung idempotent sein, oder die Anwendung muss eine Deduplizierung von Nachrichten durchführen können.

**Behandeln Sie Ausnahmen.** Ein Ereignisconsumer verarbeitet einen Batch mit Nachrichten normalerweise in einer Schleife. Es ist ratsam, Ausnahmen in dieser Verarbeitungsschleife zu verarbeiten, damit kein gesamter Batch mit Nachrichten verloren geht, wenn eine einzelne Nachricht eine Ausnahme verursacht.

**Verwenden Sie eine Warteschlange für unzustellbare Nachrichten.** Wenn die Verarbeitung einer Nachricht zu einem nicht vorübergehenden Fehler führt, sollten Sie die Nachricht in eine Warteschlange für unzustellbare Nachrichten einreihen, damit Sie den Status nachverfolgen können. Je nach Szenario können Sie versuchen, die Nachricht später noch einmal zu verarbeiten, eine kompensierende Transaktion anzuwenden oder eine andere Aktion durchzuführen. Beachten Sie, dass Event Hubs nicht über eine integrierte Funktionalität für Warteschlangen für unzustellbare Nachrichten verfügt. Sie können Azure Queue Storage oder Service Bus verwenden, um eine Warteschlange für unzustellbare Nachrichten zu implementieren, oder Azure Functions oder einen anderen Mechanismus für die Ereignisverarbeitung nutzen.  

**Implementieren Sie die Notfallwiederherstellung per Failover zu einem sekundären Event Hubs-Namespace.** Weitere Informationen finden Sie unter [Georedundante Notfallwiederherstellung in Azure Event Hubs](/azure/event-hubs/event-hubs-geo-dr).

## <a name="redis-cache"></a>Redis-Cache

**Konfigurieren der Georeplikation.** Die Georeplikation bietet einen Mechanismus zum Verknüpfen von zwei Azure Redis Cache-Instanzen im Premium-Tarif. In den primären Cache geschriebene Daten werden in einen sekundären schreibgeschützten Cache repliziert. Weitere Informationen finden Sie unter [Konfigurieren der Georeplikation für Azure Redis Cache](/azure/redis-cache/cache-how-to-geo-replication).

**Konfigurieren der Redis-Datenpersistenz.** Mithilfe der Redis-Persistenz können Sie die in Redis gespeicherten Daten dauerhaft speichern. Sie können zudem Momentaufnahmen erstellen und die Daten sichern, die Sie dann im Fall eines Hardwarefehlers laden können. Weitere Informationen finden Sie unter [Konfigurieren von Datenpersistenz für Azure Redis Cache vom Typ „Premium“](/azure/redis-cache/cache-how-to-premium-persistence).

Wenn Sie Redis Cache als temporären Zwischenspeicher für Daten und nicht als permanenten Speicher verwenden, sind diese Empfehlungen möglicherweise nicht relevant. 

## <a name="search"></a>Suchen,

**Stellen Sie mehr als ein Replikat bereit.** Verwenden Sie mindestens zwei Replikate für hohe Verfügbarkeit für Lesezugriff oder drei für hohe Verfügbarkeit für Lese-/Schreibzugriff.

**Konfigurieren Sie Indexer für Bereitstellungen in mehreren Regionen.** Für eine Bereitstellung in mehreren Regionen sollten Sie die Optionen für Kontinuität bei der Indizierung in Betracht ziehen.

  * Wenn die Datenquelle georepliziert wird, sollten Sie im Allgemeinen die Indexer der einzelnen regionalen Azure Search-Dienste auf das eigene lokale Datenquellenreplikat verweisen. Dieser Ansatz wird jedoch für große Datasets, die in Azure SQL-Datenbank gespeichert sind, nicht empfohlen. Der Grund ist, dass Azure Search keine inkrementelle Indizierung für sekundäre SQL-Datenbankreplikate ausführen kann. Dies ist nur für primäre Replikate möglich. Verweisen Sie stattdessen alle Indexer auf das primäre Replikat. Verweisen Sie den Azure Search-Indexer nach einem Failover auf das neue primäre Replikat.  
  * Wenn die Datenquelle nicht georepliziert wird, verweisen Sie mehrere Indexer auf die gleiche Datenquelle, damit Azure Search-Dienste in mehreren Regionen kontinuierlich und unabhängig die Datenquelle indizieren. Weitere Informationen finden Sie unter [Überlegungen zur Leistung und Optimierung von Azure Search][search-optimization].

## <a name="storage"></a>Speicher

**Verwenden Sie für Anwendungsdaten georedundanten Speicher mit Lesezugriff (RA-GRS).** Bei einem RA-GRS-Speicher werden die Daten in eine sekundäre Region repliziert, und er bietet schreibgeschützten Zugriff aus der sekundären Region. Wenn in der primären Region der Speicher ausfällt, kann die Anwendung die Daten über die sekundäre Region lesen. Weitere Informationen finden Sie unter [Azure Storage-Replikation](/azure/storage/storage-redundancy/).

**Verwenden Sie Managed Disks für VM-Datenträger.** [Managed Disks][managed-disks] ermöglicht eine höhere Zuverlässigkeit für VMs in einer Verfügbarkeitsgruppe, da die Datenträger ausreichend voneinander isoliert sind, um Single Points of Failure zu vermeiden. Zudem gelten für Managed Disks keine IOPS-Grenzwerte von VHDs, die in einem Speicherkonto erstellt wurden. Weitere Informationen finden Sie unter [Verwalten der Verfügbarkeit virtueller Windows-Computer in Azure][vm-manage-availability].

**Erstellen Sie für Queue Storage eine Sicherungswarteschlange in einer anderen Region.** Für Queue Storage hat ein schreibgeschütztes Replikat eingeschränkten Nutzen, da Sie Elemente nicht in die Warteschlange aufnehmen oder daraus entfernen können. Erstellen Sie stattdessen eine Sicherungswarteschlange in einem Speicherkonto in einer anderen Region. Bei einem Speicherausfall kann die Anwendung die Sicherungswarteschlange verwenden, bis die primäre Region wieder verfügbar ist. Auf diese Weise kann die Anwendung weiterhin neue Anforderungen verarbeiten.  

## <a name="sql-database"></a>SQL-Datenbank

**Verwenden Sie den Standard- oder Premium-Tarif.** Diese Tarife bieten einen längeren Zeitraum für die Point-in-Time-Wiederherstellung (35 Tage). Weitere Informationen finden Sie unter [SQL-Datenbankoptionen und -leistung](/azure/sql-database/sql-database-service-tiers/).

**Aktivieren Sie die SQL-Datenbanküberwachung.** Die Überwachung kann verwendet werden, um böswillige Angriffe oder Benutzerfehler zu diagnostizieren. Weitere Informationen finden Sie unter [Erste Schritte mit der SQL-Datenbanküberwachung](/azure/sql-database/sql-database-auditing-get-started/).

**Verwenden Sie die aktive Georeplikation**, um ein lesbares sekundäres Replikat in einer anderen Region zu erstellen.  Falls Ihre primäre Datenbank ausfällt oder einfach offline geschaltet werden muss, können Sie ein manuelles Failover auf die sekundäre Datenbank ausführen.  Bis Sie ein Failover ausführen, bleibt die sekundäre Datenbank schreibgeschützt.  Weitere Informationen finden Sie unter [Aktive Georeplikation in Azure SQL-Datenbank](/azure/sql-database/sql-database-geo-replication-overview/).

**Verwenden Sie Sharding.** Erwägen Sie die Verwendung von Sharding, um die Datenbank horizontal zu partitionieren. Sharding kann Fehlerisolation bereitstellen. Weitere Informationen finden Sie unter [Horizontales Hochskalieren mit Azure SQL-Datenbank](/azure/sql-database/sql-database-elastic-scale-introduction/).

**Verwenden Sie die Point-in-Time-Wiederherstellung für eine Wiederherstellung nach einem menschlichen Fehler.**  Mit der Point-in-Time-Wiederherstellung wird Ihre Datenbank zu einem früheren Zeitpunkt wiederhergestellt. Weitere Informationen finden Sie unter [Wiederherstellen einer Azure SQL-Datenbank mit automatisierten Datenbanksicherungen][sql-restore].

**Verwenden Sie die Geowiederherstellung für die Wiederherstellung nach einem Dienstausfall.** Mit der Geowiederherstellung wird eine Datenbank auf der Basis einer georedundanten Sicherung wiederhergestellt.  Weitere Informationen finden Sie unter [Wiederherstellen einer Azure SQL-Datenbank mit automatisierten Datenbanksicherungen][sql-restore].

## <a name="sql-server-running-in-a-vm"></a>SQL Server auf einem virtuellen Computer

**Replizieren Sie die Datenbank.** Verwenden Sie SQL Server AlwaysOn-Verfügbarkeitsgruppen, um die Datenbank zu replizieren. Dadurch erreichen Sie hohe Verfügbarkeit, wenn eine SQL Server-Instanz ausfällt. Weitere Informationen finden Sie unter [Ausführen von Windows-VMs für eine n-schichtige Anwendung](../reference-architectures/virtual-machines-windows/n-tier.md).

**Sichern Sie die Datenbank**. Wenn Sie bereits [Azure Backup](https://azure.microsoft.com/documentation/services/backup/) zum Sichern Ihrer VMs verwenden, erwägen Sie die Nutzung von [Azure Backup für SQL Server-Workloads mit DPM](/azure/backup/backup-azure-backup-sql/). Bei dieser Vorgehensweise gibt es eine Sicherungsadministratorrolle für die Organisation und ein einheitliches Wiederherstellungsverfahren für VMs und SQL Server. Verwenden Sie andernfalls die [verwaltete SQL Server-Sicherung in Microsoft Azure](https://msdn.microsoft.com/library/dn449496.aspx).

## <a name="traffic-manager"></a>Traffic Manager

**Führen Sie manuelle Failbacks aus.** Führen Sie nach einem Traffic Manager-Failover ein manuelles Failback und kein automatische Failback aus. Überprüfen Sie vor einem Failback, ob alle Subsysteme der Anwendung fehlerfrei sind.  Andernfalls könnte eine Situation eintreten, bei der die Anwendung zwischen den Rechenzentren hin und her wechselt. Weitere Informationen finden Sie unter [Ausführen von VMs in mehreren Regionen für Hochverfügbarkeit](../reference-architectures/virtual-machines-windows/multi-region-application.md).

**Erstellen Sie einen Endpunkt für Integritätstests.** Erstellen Sie einen benutzerdefinierten Endpunkt, der die Gesamtintegrität der Anwendung angibt. Dies ermöglicht Traffic Manager, ein Failover auszuführen, wenn ein kritischer Pfad und nicht nur das Front-End ausfällt. Der Endpunkt sollte einen HTTP-Fehlercode zurückgeben, wenn eine kritische Abhängigkeit fehlerhaft oder nicht erreichbar ist. Melden Sie jedoch keine Fehler für nicht kritische Dienste. Andernfalls könnte der Integritätstest ein Failover auslösen, wenn es nicht erforderlich ist, und falsch positive Ergebnisse erstellen. Weitere Informationen finden Sie unter [Traffic Manager-Endpunktüberwachung und -failover](/azure/traffic-manager/traffic-manager-monitoring/).

## <a name="virtual-machines"></a>Virtual Machines

**Vermeiden Sie es, eine Produktionsworkload auf einer einzigen VM auszuführen.** Die Bereitstellung einer einzigen VM ist bei geplanten oder ungeplanten Wartungen nicht zuverlässig. Nehmen Sie stattdessen mehrere VMs in eine Verfügbarkeitsgruppe oder eine [VM-Skalierungsgruppe](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/) auf, der sie einen Lastenausgleich vorschalten.

**Geben Sie bei der Bereitstellung der VM eine Verfügbarkeitsgruppe an.** Derzeit gibt es keine Möglichkeit, einer Verfügbarkeitsgruppe nach dem Bereitstellen der VM eine VM hinzuzufügen. Sie müssen beim Hinzufügen einer neuen VM zu einer vorhandenen Verfügbarkeitsgruppe eine NIC für die VM erstellen und die NIC dem Back-End-Adresspool des Lastenausgleichs hinzufügen. Andernfalls leitet der Lastenausgleich den Netzwerkdatenverkehr nicht an die VM weiter.

**Nehmen Sie jede Logikschicht in eine separate Verfügbarkeitsgruppe auf.** Fügen Sie in einer Anwendung mit n-Schichten VMs aus verschiedenen Schichten nicht in die gleiche Verfügbarkeitsgruppe ein. VMs in einer Verfügbarkeitsgruppe werden über Fehlerdomänen (FDs) und Updatedomänen (UD) verteilt. Um den Redundanzvorteil von FDs und UDs nutzen zu können, muss jedoch jede VM in der Verfügbarkeitsgruppe die gleichen Clientanforderungen verarbeiten können.

**Wählen Sie die richtige VM-Größe basierend auf den Leistungsanforderungen aus.** Beginnen Sie beim Verlagern einer vorhandenen Workload in Azure mit der VM-Größe, die Ihren lokalen Servern am ehesten entspricht. Messen Sie dann die Leistung Ihrer tatsächlichen Workload hinsichtlich CPU, Arbeitsspeicher und Datenträger-IOPS, und passen Sie die Größe bei Bedarf an. Dadurch wird sichergestellt, dass sich die Anwendung in einer Cloudumgebung wie erwartet verhält. Wenn Sie mehrere NICs benötigen, beachten Sie zudem den NIC-Grenzwert für jede Größe.

**Verwenden Sie Managed Disks für VHDs.** [Managed Disks][managed-disks] ermöglicht eine höhere Zuverlässigkeit für VMs in einer Verfügbarkeitsgruppe, da die Datenträger ausreichend voneinander isoliert sind, um Single Points of Failure zu vermeiden. Zudem gelten für Managed Disks keine IOPS-Grenzwerte von VHDs, die in einem Speicherkonto erstellt wurden. Weitere Informationen finden Sie unter [Verwalten der Verfügbarkeit virtueller Windows-Computer in Azure][vm-manage-availability].

**Installieren Sie Anwendungen auf einem Datenträger für Daten und nicht auf dem Datenträger für das Betriebssystem.** Andernfalls kann der Grenzwert für die Datenträgergröße erreicht werden.

**Verwenden Sie Azure Backup zum Sichern von VMs.** Sicherungen schützen vor versehentlichen Datenverlusten. Weitere Informationen finden Sie unter [Schützen von Azure-VMs mit einem Recovery Services-Tresor](/azure/backup/backup-azure-vms-first-look-arm/).

**Aktivieren Sie Diagnoseprotokolle**, z.B. grundlegende Integritätsmetriken, Infrastrukturprotokolle und die [Startdiagnose][boot-diagnostics]. Startdiagnosen dienen dazu, einen Fehler beim Startvorgang zu untersuchen, wenn sich Ihre VM in einem nicht startfähigen Zustand befindet. Weitere Informationen finden Sie unter [Übersicht über Azure-Diagnoseprotokolle][diagnostics-logs].

**Verwenden Sie die AzureLogCollector-Erweiterung.** (Nur Windows-VMs.) Diese Erweiterung aggregiert Azure Platform-Protokolle und lädt sie in Azure-Speicher hoch, ohne dass eine Remoteanmeldung des Operators auf der VM erforderlich ist. Weitere Informationen finden Sie unter [AzureLogCollector-Erweiterung](/azure/virtual-machines/virtual-machines-windows-log-collector-extension/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="virtual-network"></a>Virtuelles Netzwerk

**Um öffentliche IP-Adressen einer Whitelist hinzuzufügen oder zu sperren, fügen Sie dem Subnetz eine NSG hinzu.** Blockieren Sie den Zugriff von böswilligen Benutzern, oder lassen Sie nur Benutzer, die über Zugriffsberechtigungen verfügen, auf die Anwendung zugreifen.  

**Erstellen Sie einen benutzerdefinierten Integritätstest.** Integritätstests durch den Lastenausgleich können für HTTP oder TCP durchgeführt werden. Wenn eine VM auf einem HTTP-Server ausgeführt wird, ist der HTTP-Test ein besserer Indikator für den Integritätsstatus als ein TCP-Test. Verwenden Sie für einen HTTP-Test einen benutzerdefinierten Endpunkt, der die Gesamtintegrität der Anwendung meldet, einschließlich aller kritischen Abhängigkeiten. Weitere Informationen finden Sie unter [Übersicht über Azure Load Balancer](/azure/load-balancer/load-balancer-overview/).

**Blockieren Sie den Integritätstest nicht.** Der Integritätstests durch den Lastenausgleich wird von einer bekannten IP-Adresse (168.63.129.16) gesendet. Blockieren Sie den Datenverkehr zu oder von dieser IP-Adresse nicht in Firewallrichtlinien oder in NSG-Regeln (Netzwerksicherheitsgruppe). Wenn der Integritätstest blockiert wird, wird die VM vom Lastenausgleich aus der Rotation entfernt.

**Aktivieren Sie die Protokollierung für den Lastenausgleich.** Die Protokolle zeigen, wie viele VMs am Back-End aufgrund fehlerhafter Testantworten keinen Netzwerkdatenverkehr empfangen. Weitere Informationen finden Sie unter [Protokollanalysen für Azure Load Balancer](/azure/load-balancer/load-balancer-monitor-log/).

<!-- links -->
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[diagnostics-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs/
[managed-disks]: /azure/storage/storage-managed-disks-overview
[search-optimization]: /azure/search/search-performance-optimization/
[sql-backup]: /azure/sql-database/sql-database-automated-backups/
[sql-restore]: /azure/sql-database/sql-database-recovery-using-backups/
[vm-manage-availability]: /azure/virtual-machines/windows/manage-availability#use-managed-disks-for-vms-in-an-availability-set
