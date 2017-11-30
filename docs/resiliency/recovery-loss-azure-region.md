---
title: Wiederherstellen nach Ausfall einer Azure-Region
description: "Artikel über die Grundlagen und den Entwurf von robusten, hoch verfügbaren, fehlertoleranten Anwendungen sowie die Planung der Notfallwiederherstellung."
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: 42a7d865e101b43279f3198f3dd75df1b15a8565
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
[!INCLUDE [header](../_includes/header.md)]
# <a name="azure-resiliency-technical-guidance-recovery-from-a-region-wide-service-disruption"></a>Technischer Leitfaden zur Resilienz in Azure: Wiederherstellung nach einer regionsweiten Dienstunterbrechung
Azure ist physisch und logisch in Einheiten unterteilt, die als Regionen bezeichnet werden. Eine Region besteht aus einem Rechenzentrum oder mehreren in unmittelbarer Nähe zueinander befindlichen Rechenzentren. 

In seltenen Fällen kann es vorkommen, dass auf Einrichtungen in einer gesamten Region nicht zugegriffen werden kann, beispielsweise aufgrund von Netzwerkfehlern. Naturkatastrophen können dazu führen, dass eine Region vollständig ausfällt. Dieser Artikel erläutert die Funktionen von Azure, mit denen Sie Anwendungen erstellen können, die über mehrere Regionen verteilt werden. Diese Verteilung bewirkt, dass es sehr unwahrscheinlich ist, dass ein Fehler in einer Region sich auf andere Regionen auswirkt.

## <a name="cloud-services"></a>Clouddienste
### <a name="resource-management"></a>Ressourcenverwaltung
Sie können Compute-Instanzen auf mehrere Regionen verteilen, indem Sie in jeder Zielregion einen separaten Clouddienst erstellen und das Bereitstellungspaket dann in jedem Clouddienst veröffentlichen. Beachten Sie jedoch, dass die Verteilung des Datenverkehrs auf Clouddienste in verschiedenen Regionen durch den Anwendungsentwickler oder mithilfe eines Diensts für die Datenverkehrsverwaltung implementiert werden muss.

Ein wichtiger Aspekt der Kapazitätsplanung ist es, die Anzahl von Reserverolleninstanzen zu bestimmen, die zum Zweck der Notfallwiederherstellung im Voraus bereitgestellt werden sollen. Eine vollständige sekundäre Bereitstellung stellt sicher, dass bei Bedarf sofort Kapazität zur Verfügung steht, verdoppelt jedoch auch die Kosten. Eine häufige Vorgehensweise ist es, eine kleine sekundäre Bereitstellung einzurichten, die gerade groß genug ist, um wichtige Dienste auszuführen. Diese kleine sekundäre Bereitstellung ist empfehlenswert, um sowohl für Reservekapazität zu sorgen als auch die Konfiguration der sekundären Umgebung zu testen.

> [!NOTE]
> Das Abonnementkontingent ist keine Kapazitätsgarantie. Bei diesem Kontingent handelt es sich lediglich um ein Kreditlimit. Um eine bestimmte Kapazität zu garantieren, muss die erforderliche Anzahl von Rollen im Dienstmodell definiert werden, und die Rollen müssen bereitgestellt werden.
> 
> 

### <a name="load-balancing"></a>Lastenausgleich
Für den Lastenausgleich des Datenverkehrs über mehrere Regionen hinweg ist eine Lösung für die Datenverkehrsverwaltung erforderlich. Azure stellt zu diesem Zweck den [Azure Traffic Manager](https://azure.microsoft.com/services/traffic-manager/)bereit. Sie können auch Drittanbieterdienste nutzen, die ähnliche Funktionen für die Verwaltung des Datenverkehrs bieten.

### <a name="strategies"></a>Strategien
Zum Implementieren verteilter Computeressourcen über mehrere Regionen hinweg stehen viele alternative Strategien zur Verfügung. Diese müssen auf die jeweiligen Geschäftsanforderungen und die besonderen Bedingungen der Anwendung angepasst werden. Die Vorgehensweisen können grob in die drei folgenden Kategorien unterteilt werden:

* **Erneute Bereitstellung bei einem Notfall**: Hierbei wird die Anwendung bei einem Notfall von Grund auf neu bereitgestellt. Diese Vorgehensweise eignet sich für nicht-kritischen Anwendungen, die keine garantierte Wiederherstellungszeit erfordern.
* **Warmspare (aktiv/passiv)**: Ein sekundärer gehosteter Dienst wird in einer alternativen Region erstellt, und Rollen werden bereitgestellt, um eine minimale Kapazität zu garantieren. Die Rollen empfangen jedoch keinen Produktionsdatenverkehr. Diese Vorgehensweise eignet sich für Anwendungen, die nicht dafür entworfen wurden, Datenverkehr über Regionen hinweg zu verteilen.
* **Hotspare (aktiv/aktiv)**: Die Anwendung wurde dafür entworfen, Produktionslasten in mehreren Regionen zu empfangen. Die Clouddienste in jeder Region können zur Notfallwiederherstellung für eine höhere Kapazität als erforderlich konfiguriert werden. Alternativ dazu können die Clouddienste auch horizontal hochskaliert werden, wenn es zum Zeitpunkt eines Notfalls und für ein Failover erforderlich ist. Für diese Vorgehensweise sind beträchtliche Investitionen in den Anwendungsentwurf erforderlich, aber sie bietet auch erhebliche Vorteile: eine kurze und garantierte Wiederherstellungszeit, kontinuierliche Tests aller Wiederherstellungsstandorte und eine effiziente Nutzung der Kapazität.

Eine vollständige Beschreibung des verteilten Entwurfs würde den Rahmen dieses Dokuments sprengen. Weitere Informationen finden Sie unter [Notfallwiederherstellung und hohe Verfügbarkeit für Azure-Anwendungen](https://aka.ms/drtechguide).

## <a name="virtual-machines"></a>Virtual Machines
Die Wiederherstellung von IaaS-VMs (Infrastructure as a Service) ähnelt in vielerlei Hinsicht der Wiederherstellung von PaaS-Computeressourcen (Platform as a Service). Es gibt jedoch wesentliche Unterschiede aufgrund der Tatsache, dass eine IaaS-VM sowohl aus der VM als auch aus dem VM-Datenträger besteht.

* **Verwenden von Azure Backup zum Erstellen von anwendungskonsistenten, regionsübergreifenden Sicherungen**:
  [Azure Backup](https://azure.microsoft.com/services/backup/) ermöglicht es Kunden, anwendungskonsistente Sicherungen über mehrere VM-Datenträger hinweg zu erstellen und die Replikation von Sicherungen über Regionen hinweg zu unterstützen. Sie erreichen dies, indem Sie zum Zeitpunkt der Erstellung die Georeplikation des Sicherungstresors auswählen. Beachten Sie, dass die Replikation des Sicherungstresors zum Zeitpunkt der Erstellung konfiguriert werden muss. Eine spätere Einrichtung ist nicht möglich. Wenn eine Region ausfällt, stellt Microsoft den Kunden die Sicherungen zur Verfügung. Kunden können ihre Systeme auf jeden ihrer konfigurierten Wiederherstellungspunkte wiederherstellen.
* **Trennen des Datenträgers mit Daten vom Datenträger mit dem Betriebssystem**: Ein wichtiger Aspekt bei IaaS-VMs ist, dass Sie den Betriebssystemdatenträger nicht ändern können, ohne die VM neu zu erstellen. Dies ist kein Problem, wenn Ihre Wiederherstellungsstrategie darin besteht, das System nach einem Notfall erneut bereitzustellen. Es kann jedoch ein Problem sein, wenn Sie den Warmspare-Ansatz zum Reservieren von Kapazität verwenden. Um diesen Ansatz richtig zu implementieren, muss der richtige Betriebssystemdatenträger sowohl am primären als auch am sekundären Standort bereitgestellt werden, und die Anwendungsdaten müssen auf einem separaten Laufwerk gespeichert werden. Verwenden Sie nach Möglichkeit eine standardmäßige Betriebssystemkonfiguration, die an beiden Standorten bereitgestellt werden kann. Nach einem Failover müssen Sie das Datenlaufwerk wieder an die vorhandenen IaaS-VMs im sekundären Rechenzentrum anfügen. Verwenden Sie AzCopy, um Momentaufnahmen des Datenträgers bzw. der Datenträger mit den Daten in einen Remotestandort zu kopieren.
* **Potenzielle Konsistenzprobleme nach einem Geofailover mehrerer VM-Datenträger**: VM-Datenträger werden als Azure Storage-Blobs implementiert und weisen die gleichen Georeplikationsmerkmale auf. Da die Georeplikation asynchron verläuft und unabhängige Replikate erzeugt, kann die Konsistenz zwischen Datenträgern nur bei Verwendung von [Azure Backup](https://azure.microsoft.com/services/backup/) garantiert werden. Für einzelne VM-Datenträger kann ein absturzkonsistenter Status nach einem geografischen Failover garantiert werden, aber es gibt keine Garantie für eine datenträgerübergreifende Konsistenz. Dies kann in einigen Fällen zu Problemen führen (beispielsweise beim Datenträgerstriping).

## <a name="storage"></a>Speicher
### <a name="recovery-by-using-geo-redundant-storage-of-blob-table-queue-and-vm-disk-storage"></a>Wiederherstellung mithilfe von georedundanter Speicherung von Blob-, Tabellen-, Warteschlangen- und VM-Datenträgerspeichern
In Azure werden Blobs, Tabellen, Warteschlangen und VM-Datenträger standardmäßig geografisch repliziert. Dies wird als GRS bezeichnet (Geo-Redundant Storage, georedundanter Speicher). GRS repliziert Speicherdaten in ein gekoppeltes Rechenzentrum, das sich Hunderte von Kilometern entfernt innerhalb einer bestimmten geografischen Region befindet. GRS sorgt für zusätzliche Dauerhaftigkeit, sollte eine schwerwiegender Rechenzentrumsnotfall eintreten. Microsoft steuert, wann ein Failover durchgeführt wird. Failover sind auf schwerwiegende Notfälle begrenzt, bei denen der ursprüngliche primäre Standort als nicht innerhalb eines angemessenen Zeitrahmens wiederherstellbar erachtet wird. In einigen Szenarien kann es sich hierbei um mehrere Tage handeln. Daten werden üblicherweise innerhalb weniger Minuten repliziert, auch wenn das Synchronisierungsintervall noch nicht durch eine Vereinbarung zum Servicelevel abgedeckt ist.

Im Fall eines geografischen Failovers ändert sich die Art des Zugriffs auf das Konto nicht (URL und Kontoschlüssel bleiben unverändert). Das Speicherkonto befindet sich nach dem Failover jedoch in einer anderen Region. Dies kann sich auf Anwendungen auswirken, für die eine regionale Affinität zu ihrem Speicherkonto erforderlich ist. Auch bei Diensten und Anwendungen, für die sich das Speicherkonto nicht im gleichen Rechenzentrum befinden muss, können die rechenzentrumsübergreifenden Latenzzeiten und Bandbreitenkosten überzeugende Gründe dafür sein, den Datenverkehr temporär in die Failoverregion zu verschieben. All dies sollte bei der Gesamtstrategie für die Notfallwiederherstellung berücksichtigt werden.

Zusätzlich zum automatischen Failover durch GRS hat Azure einen Dienst eingeführt, der Ihnen Lesezugriff auf die Kopie Ihrer Daten im sekundären Speicherort bietet. Dieser Dienst wird als RA-GRS (Read-Access Geo-Redundant Storage, georedundanter Speicher mit Lesezugriff) bezeichnet.

Weitere Informationen zur Speicherung mit GRS und RA-GRS finden Sie unter [Azure Storage-Replikation](/azure/storage/storage-redundancy/).

### <a name="geo-replication-region-mappings"></a>Regionszuordnungen für die Georeplikation:
Es ist wichtig zu erfahren, wo Ihre Daten geografisch repliziert werden. Sie müssen wissen, wo die anderen Instanzen Ihrer Daten bereitgestellt werden sollen, die regionale Affinität mit Ihrem Speicher erfordern. Weitere Informationen finden Sie unter [Azure-Regionspaare](/azure/best-practices-availability-paired-regions).

### <a name="geo-replication-pricing"></a>Georeplikation – Preise:
Die Georeplikation ist in den aktuellen Preisen für Azure Storage enthalten. Dies wird als georedundanter Speicher (GRS) bezeichnet. Wenn Sie nicht möchten, dass Ihre Daten geografisch repliziert werden, können Sie die Georeplikation für Ihr Konto deaktivieren. Dies wird als lokal redundanter Speicher bezeichnet und zu einem geringeren Preis als GRS berechnet.

### <a name="determining-if-a-geo-failover-has-occurred"></a>Ermitteln, ob ein geografisches Failover stattgefunden hat
Wenn ein Geofailover stattgefunden hat, wird dies auf dem [Dashboard zur Azure-Dienstintegrität](https://azure.microsoft.com/status/) angezeigt. Anwendungen können jedoch eine automatisierte Methode zur Erkennung eines geografischen Failovers implementieren, indem sie die geografische Region für ihr Speicherkonto überwachen. Eine solche Konfiguration kann dazu verwendet werden, weitere Wiederherstellungsvorgänge auszulösen, z.B. die Aktivierung von Computeressourcen in der geografischen Region, in die der Speicher verschoben wurde. Hierfür können Sie eine Abfrage über die Dienstverwaltungs-API durchführen, indem Sie die [Get Storage Account Properties](https://msdn.microsoft.com/library/ee460802.aspx)-Anforderung verwenden. Die relevanten Eigenschaften lauten:

    <GeoPrimaryRegion>primary-region</GeoPrimaryRegion>
    <StatusOfPrimary>[Available|Unavailable]</StatusOfPrimary>
    <LastGeoFailoverTime>DateTime</LastGeoFailoverTime>
    <GeoSecondaryRegion>secondary-region</GeoSecondaryRegion>
    <StatusOfSecondary>[Available|Unavailable]</StatusOfSecondary>

### <a name="vm-disks-and-geo-failover"></a>VM-Datenträger und geografisches Failover
Wie bereits im Abschnitt zu VM-Datenträgern erläutert, gibt es keine Garantien für VM-übergreifende Datenkonsistenz nach einem Failover. Zur Gewährleistung der Richtigkeit von Sicherungen sollte eine Sicherungsanwendung wie Data Protection Manager verwendet werden, um Anwendungsdaten zu sichern und wiederherzustellen.

## <a name="database"></a>Datenbank
### <a name="sql-database"></a>SQL-Datenbank
Azure SQL-Datenbank stellt zwei Arten von Wiederherstellung bereit: geografische Wiederherstellung und aktive Georeplikation.

#### <a name="geo-restore"></a>Geografische Wiederherstellung
[Geografische Wiederherstellung](/azure/sql-database/sql-database-recovery-using-backups/#geo-restore) steht auch für Basic-, Standard- und Premium-Datenbanken zur Verfügung. Sie stellt die Standardoption für die Wiederherstellung dar, wenn die Datenbank aufgrund eines Vorfalls in der Region, in dem die Datenbank gehostet wird, nicht verfügbar ist. Ähnlich wie die Point-in-Time-Wiederherstellung ist die Geowiederherstellung von Datenbanksicherungen in georedundantem Azure-Speicher abhängig. Die Wiederherstellung erfolgt aus der geografisch replizierten Sicherungskopie und ist daher in Bezug auf Speicherausfälle in der primären Region flexibel. Ausführlichere Informationen finden Sie unter [Wiederherstellen einer Azure SQL-Datenbank oder Failover auf eine sekundäre Datenbank](/azure/sql-database/sql-database-disaster-recovery/).

#### <a name="active-geo-replication"></a>Aktive Georeplikation
[Aktive Georeplikation](/azure/sql-database/sql-database-geo-replication-overview/) ist für alle Datenbanktarife verfügbar. Sie ist für Anwendungen vorgesehen, für die umfangreichere Wiederherstellungsanforderungen erforderlich sind, als die Geowiederherstellung bieten kann. Bei der aktiven Georeplikation können Sie bis zu vier lesbare sekundäre Replikate auf Servern in verschiedenen Regionen erstellen. Sie können ein Failover auf ein beliebiges sekundäres Replikat initiieren. Darüber hinaus kann die aktive Georeplikation verwendet werden, um Anwendungsupgrades oder die räumliche Verlegung von Anwendungen zu unterstützen, sowie als Lastenausgleich für schreibgeschützte Arbeitsauslastungen. Weitere Informationen finden Sie unter [Konfigurieren der Georeplikation](/azure/sql-database/sql-database-geo-replication-portal/) und [Failover zur sekundären Datenbank](/azure/sql-database/sql-database-geo-replication-failover-portal/). Einzelheiten zum Entwerfen und Implementieren von Anwendungen und Anwendungsupgrades ohne Ausfallzeiten finden Sie unter [Entwerfen einer Anwendung für die cloudbasierte Notfallwiederherstellung mithilfe der aktiven Georeplikation in SQL-Datenbank](/azure/sql-database/sql-database-designing-cloud-solutions-for-disaster-recovery/) sowie unter [Verwalten von parallelen Upgrades von Cloudanwendungen mithilfe der aktiven Georeplikation von SQL-Datenbank](/azure/sql-database/sql-database-manage-application-rolling-upgrade/).

### <a name="sql-server-on-virtual-machines"></a>SQL Server auf virtuellen Computern
Für die Wiederherstellung und hohe Verfügbarkeit für SQL Server 2012 (und höher) auf Azure Virtual Machines steht eine Vielzahl von Optionen zur Verfügung. Weitere Informationen finden Sie unter [Hohe Verfügbarkeit und Notfallwiederherstellung für SQL Server auf virtuellen Azure-Computern](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

## <a name="other-azure-platform-services"></a>Weitere Azure Platform-Dienste
Wenn Sie Ihre Clouddienste in mehreren Azure-Regionen ausführen möchten, müssen Sie die Auswirkungen für alle Abhängigkeiten bedenken. In den folgenden Abschnitten wird bei den dienstspezifischen Anleitungen davon ausgegangen, dass Sie den gleichen Azure-Dienst in einem alternativen Azure-Rechenzentrum verwenden müssen. Hierzu gehören sowohl Konfigurations- als auch Datenreplikationsaufgaben.

> [!NOTE]
> In einigen Fällen können diese Schritte dabei helfen, das Risiko eines dienstspezifischen Ausfalls zu verringern, nicht eines Ereignisses des gesamten Rechenzentrums. Aus Sicht der Anwendung kann ein dienstspezifischer Ausfall die gleichen Einschränkungen zur Folge haben und die temporäre Migration des Diensts in eine alternative Azure-Region erfordern.
> 
> 

### <a name="service-bus"></a>Service Bus
Azure Service Bus verwendet einen eindeutigen Namespace, der nicht mehrere Azure-Regionen umfasst. Die erste Anforderung besteht also darin, die erforderlichen Service Bus-Namespaces in der alternativen Region einzurichten. Es müssen jedoch auch Überlegungen bezüglich der Beständigkeit der Nachrichten in der Warteschlange angestellt werden. Es gibt verschiedene Strategien zur Replikation von Nachrichten über Azure-Regionen hinweg. Details zu diesen Replikationsstrategien und weiteren Strategien für die Notfallwiederherstellung finden Sie unter [Bewährte Methoden zum Schützen von Anwendungen vor Service Bus-Ausfällen und Notfällen](/azure/service-bus-messaging/service-bus-outages-disasters/). Weitere Verfügbarkeitsaspekte finden Sie unter [Service Bus (Verfügbarkeit)](recovery-local-failures.md#other-azure-platform-services).

### <a name="app-service"></a>App Service
Um eine Azure App Service-Anwendung, z.B. eine Web-App oder Mobile App, in eine sekundäre Azure-Region zu migrieren, müssen Sie über eine Sicherung der Website verfügen, die veröffentlicht werden kann. Wenn der Ausfall nicht das gesamte Azure-Rechenzentrum betrifft, können Sie eine aktuelle Sicherung des Websiteinhalts möglicherweise über FTP herunterladen. Erstellen Sie dann, sofern nicht bereits erfolgt, in der alternativen Region eine neue App, um Kapazität zu reservieren. Veröffentlichen Sie die Website in der neuen Region, und nehmen Sie ggf. notwendige Konfigurationsänderungen vor. Bei diesen Änderungen kann es sich um Verbindungszeichenfolgen für die Datenbank oder andere regionsspezifische Einstellungen handeln. Sofern notwendig, fügen Sie das SSL-Zertifikat der Website hinzu, und ändern Sie den DNS CNAME-Eintrag, sodass der benutzerdefinierte Domänenname auf die URL der neu bereitgestellten Azure-Web-App verweist.

### <a name="hdinsight"></a>HDInsight
Die mit HDInsight verknüpften Daten werden standardmäßig in Azure Blob Storage gespeichert. Für HDInsight muss sich ein Hadoop-Cluster zur Verarbeitung von MapReduce-Aufträgen in der gleichen Region befinden wie das Speicherkonto mit den zu analysierenden Daten. Sofern Sie das Georeplikationsfeature für Azure Storage verwenden, können Sie auf die Daten in der sekundären Region zugreifen, in die die Daten repliziert wurden, falls aus irgendeinem Grund die primäre Region nicht mehr verfügbar ist. Sie können einen neuen Hadoop-Cluster in der Region erstellen, in die die Daten repliziert wurden, und mit der Verarbeitung der Daten fortfahren. Weitere Verfügbarkeitsaspekte finden Sie unter [HDInsight (Verfügbarkeit)](recovery-local-failures.md#other-azure-platform-services).

### <a name="sql-reporting"></a>SQL Reporting
Derzeit erfordert die Wiederherstellung nach dem Verlust einer Azure-Region mehrere SQL Reporting-Instanzen in verschiedenen Azure-Regionen. Diese SQL Reporting-Instanzen sollten auf die gleichen Daten zugreifen, und für diese Daten sollte ein eigener Wiederherstellungsplan für den Notfall bereitstehen. Sie können auch externe Sicherungskopien der RDL-Datei für jeden Bericht speichern.

### <a name="media-services"></a>Media Services
Azure Media Services verfügt über eine andere Wiederherstellungsstrategie für Codierung und Streaming. In der Regel ist das Streaming während eines regionalen Ausfalls wichtiger. Zur Vorbereitung auf einen solchen Fall sollten Sie über ein Media Services-Konto in zwei verschiedenen Azure-Regionen verfügen. Die codierten Inhalte sollten sich in beiden Regionen befinden. Bei einem Ausfall können Sie den Streamingdatenverkehr an die alternative Region umleiten. Die Codierung kann in jeder beliebigen Azure-Region erfolgen. Wenn die Codierung zeitkritisch ist, beispielsweise während der Verarbeitung von Liveereignissen, müssen Sie darauf vorbereitet sein, bei einem Ausfall Aufträge an ein alternatives Rechenzentrum zu übermitteln.

### <a name="virtual-network"></a>Virtuelles Netzwerk
Konfigurationsdateien stellen die schnellste Methode zum Einrichten eines virtuellen Netzwerks in einer alternativen Azure-Region dar. Nachdem Sie das virtuelle Netzwerk in der primären Azure-Region konfiguriert haben, [exportieren Sie die Einstellungen des virtuellen Netzwerks](/azure/virtual-network/virtual-networks-create-vnet-classic-portal/) in eine Netzwerkkonfigurationsdatei. Bei einem Ausfall in der primären Region [stellen Sie das virtuelle Netzwerk](/azure/virtual-network/virtual-networks-create-vnet-classic-portal/) aus der gespeicherten Konfigurationsdatei wieder her. Konfigurieren Sie dann Clouddienste, virtuelle Computer oder standortübergreifende Einstellungen für das neue Netzwerk.

## <a name="checklists-for-disaster-recovery"></a>Prüflisten für die Notfallwiederherstellung
## <a name="cloud-services-checklist"></a>Prüfliste – Cloud Services
1. Lesen Sie den Abschnitt zu Cloud Services in diesem Dokument.
2. Erstellen Sie eine regionsübergreifende Strategie für die Notfallwiederherstellung.
3. Informieren Sie sich über die Nachteile, die durch das Reservieren von Kapazität in alternativen Regionen entstehen können.
4. Verwenden Sie Tools für das Datenverkehrsrouting, beispielsweise Azure Traffic Manager.

## <a name="virtual-machines-checklist"></a>Prüfliste – Virtual Machines
1. Lesen Sie den Abschnitt zu Virtual Machines in diesem Dokument.
2. Verwenden Sie [Azure Backup](https://azure.microsoft.com/services/backup/) , um anwendungskonsistente, regionsübergreifende Sicherungen zu erstellen.

## <a name="storage-checklist"></a>Prüfliste – Storage
1. Lesen Sie den Abschnitt zu Storage in diesem Dokument.
2. Deaktivieren Sie nicht die Georeplikation von Speicherressourcen.
3. Informieren Sie sich über alternative Regionen für die Georeplikation im Fall eines Failovers.
4. Erstellen Sie benutzerdefinierte Sicherungsstrategien für benutzergesteuerte Failoverstrategien.

## <a name="sql-database-checklist"></a>Prüfliste – SQL-Datenbank
1. Lesen Sie den Abschnitt zu SQL-Datenbank in diesem Dokument.
2. Verwenden Sie je nach Bedarf die [Geowiederherstellung](/azure/sql-database/sql-database-recovery-using-backups/#geo-restore) oder die [Georeplikation](/azure/sql-database/sql-database-geo-replication-overview/).

## <a name="sql-server-on-virtual-machines-checklist"></a>Prüfliste – SQL Server auf Virtual Machines
1. Lesen Sie den Abschnitt zu SQL Server auf Virtual Machines in diesem Dokument.
2. Verwenden Sie regionsübergreifende AlwaysOn-Verfügbarkeitsgruppen oder die Datenbankspiegelung.
3. Alternativ dazu können Sie Daten auch in einem Blobspeicher sichern und wiederherstellen.

## <a name="service-bus-checklist"></a>Prüfliste – Service Bus
1. Lesen Sie den Abschnitt zu Service Bus in diesem Dokument.
2. Konfigurieren Sie einen Service Bus-Namespace in einer alternativen Region.
3. Erwägen Sie benutzerdefinierte Strategien zur regionsübergreifenden Replikation von Nachrichten.

## <a name="app-service-checklist"></a>App Service-Checkliste
1. Lesen Sie den Abschnitt zu App Service in diesem Dokument.
2. Speichern Sie Websitesicherungen außerhalb der primären Region.
3. Versuchen Sie bei einem partiellen Ausfall, die aktuelle Website über FTP abzurufen.
4. Planen Sie die Bereitstellung der Website-Inhalte in einer neuen oder vorhandenen Website in einer alternativen Region.
5. Planen Sie Konfigurationsänderungen sowohl für die Anwendung als auch für DNS CNAME-Einträge.

## <a name="hdinsight-checklist"></a>Prüfliste – HDInsight
1. Lesen Sie den Abschnitt zu HDInsight in diesem Dokument.
2. Erstellen Sie einen neuen Hadoop-Cluster in der Region mit den replizierten Daten.

## <a name="sql-reporting-checklist"></a>Prüfliste – SQL Reporting
1. Lesen Sie den Abschnitt zu SQL Reporting in diesem Dokument.
2. Verwalten Sie eine alternative SQL Reporting-Instanz in einer anderen Region.
3. Erstellen Sie einen separaten Plan zur Replikation des Ziels in dieser Region.

## <a name="media-services-checklist"></a>Prüfliste – Media Services
1. Lesen Sie den Abschnitt zu Media Services in diesem Dokument.
2. Erstellen Sie ein Media Services-Konto in einer alternativen Region.
3. Codieren Sie den gleichen Inhalt in beiden Regionen zur Unterstützung eines Streamingfailovers.
4. Übermitteln Sie Codierungsaufträge bei einer Dienstunterbrechung an eine alternative Region.

## <a name="virtual-network-checklist"></a>Prüfliste – Virtual Network
1. Lesen Sie den Abschnitt zu Virtual Network in diesem Dokument.
2. Verwenden Sie exportierte Einstellungen für das virtuelle Netzwerk, um es in einer anderen Region neu zu erstellen.

