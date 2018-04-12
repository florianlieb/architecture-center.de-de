---
title: Technischer Leitfaden – Wiederherstellung nach lokalen Ausfällen in Azure
description: Artikel über die Grundlagen und den Entwurf von robusten, hoch verfügbaren, fehlertoleranten Anwendungen sowie die Planung der Notfallwiederherstellung nach lokalen Fehlern in Azure.
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: 5fc929bd1affe3dd6616f908bae0e7d2fefb89d5
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-local-failures-in-azure"></a>Technischer Leitfaden zur Resilienz in Azure – Wiederherstellung nach lokalen Ausfällen in Azure

Es gibt zwei primäre Bedrohungen der Anwendungsverfügbarkeit:

* Ausfall von Geräten, z.B. Festplatten und Servern
* Volle Auslastung kritischer Ressourcen, z.B. von Computeressourcen bei Spitzenlast

Um unter solchen Umständen für Hochverfügbarkeit zu sorgen, bietet Azure eine Kombination aus Ressourcenverwaltung, Elastizität, Lastenausgleich und Partitionierung. Einige dieser Funktionen werden automatisch für alle Azure-Dienste ausgeführt. In einigen Fällen muss der Anwendungsentwickler aber zusätzliche Schritte ausführen, um davon zu profitieren.

## <a name="cloud-services"></a>Cloud Services
Azure Cloud Services besteht aus Sammlungen mit mindestens einer Web- oder Workerrolle. Eine oder mehrere Instanzen einer Rolle können gleichzeitig ausgeführt werden. Die Konfiguration bestimmt die Anzahl von Instanzen. Rolleninstanzen werden mithilfe einer Komponente überwacht und verwaltet, die als Fabric Controller bezeichnet wird. Der Fabric Controller erkennt sowohl Software- als auch Hardwarefehler automatisch und reagiert darauf.

Jede Rolleninstanz wird in einem eigenen virtuellen Computer ausgeführt und kommuniziert über einen Gast-Agent mit dem Fabric Controller. Der Gast-Agent sammelt Ressourcen- und Knotenmetriken, einschließlich VM-Nutzung, Status, Protokolle, Ressourcenauslastung, Ausnahmen und Fehlerbedingungen. Der Fabric Controller fragt den Gast-Agent in konfigurierbaren Intervallen ab und startet die VM neu, wenn der Gast-Agent nicht antwortet. Bei einem Hardwarefehler verschiebt der zugehörige Fabric Controller alle betroffenen Rolleninstanzen in einen neuen Hardwareknoten und konfiguriert das Netzwerk neu, um den Datenverkehr an diesen Knoten zu leiten.

Um von diesen Funktionen profitieren zu können, sollten Entwickler sicherstellen, dass Dienstrollen keinen Statusinformationen in den Rolleninstanzen speichern. Stattdessen sollte der Zugriff auf alle persistenten Daten über einen permanenten Speicher erfolgen, beispielsweise über Azure Storage oder Azure SQL-Datenbank. So wird allen Rollen die Behandlung von Anforderungen ermöglicht. Das bedeutet auch, dass Rolleninstanzen jederzeit ausfallen können, ohne Inkonsistenzen im vorübergehenden oder permanenten Status des Diensts zu verursachen.

Die Anforderung, den Status außerhalb der Rollen zu speichern, hat verschiedene Auswirkungen. Es sollten z.B. alle zugehörigen Änderungen an einem Azure-Tabellenspeicher nach Möglichkeit in einer einzigen Entitätsgruppentransaktion ausgeführt werden. Es ist allerdings nicht immer möglich, alle Änderungen in einer einzigen Transaktion auszuführen. Sie sollten sorgfältig darauf achten, dass Rolleninstanzfehler keine Probleme verursachen, wenn sie langfristige Vorgänge unterbrechen, die zwei oder mehr Updates am persistenten Status des Diensts umfassen. Wenn eine andere Rolle versucht, einen solchen Vorgang erneut auszuführen, sollte sie den Fall voraussehen, dass die Arbeit nur teilweise abgeschlossen wurde, und entsprechend reagieren.

Stellen Sie sich beispielsweise einen Dienst vor, bei dem Daten über mehrere Speicher hinweg partitioniert werden. Wenn eine Workerrolle ausfällt, während ein Shard neu angeordnet wird, wird dieser Vorgang unter Umständen nicht abgeschlossen. Es kann auch sein, dass die Neuanordnung von einer anderen Workerrolle von Beginn an wiederholt wird. Dies kann unter Umständen zu verwaisten oder beschädigten Daten führen. Zur Verhinderung von Problemen müssen Vorgänge mit langer Ausführungsdauer mindestens eine der beiden folgenden Eigenschaften haben:

* *Idempotent*: Eine Wiederholung ist ohne Nebeneffekte möglich. Um idempotent zu sein, sollte ein langfristiger Vorgang immer die gleiche Auswirkung haben, und zwar unabhängig davon, wie oft er ausgeführt wird und ob er unterbrochen wird.
* *Inkrementell erneut startbar*: Der Vorgang sollte ab der letzten Fehlerquelle fortgesetzt werden können. Um dies zu ermöglichen, sollte ein Vorgang mit langer Ausführungsdauer aus einer Abfolge von kleineren „atomaren“ Vorgängen bestehen. Außerdem sollte der Status in permanentem Speicher aufgezeichnet werden, damit der Vorgang bei jedem nachfolgenden Aufruf an der Stelle fortgesetzt wird, an der der Vorgänger den Vorgang beendet hat.

Alle langfristigen Vorgänge sollten wiederholt aufgerufen werden, bis sie erfolgreich abgeschlossen werden. Ein Bereitstellungsvorgang kann beispielsweise in einer Azure-Warteschlange platziert und von einer Workerrolle erst nach erfolgreicher Durchführung aus der Warteschlange entfernt werden. Zum Bereinigen von Daten, die durch unterbrochene Vorgänge erstellt wurden, ist möglicherweise eine Garbage Collection erforderlich.

### <a name="elasticity"></a>Elastizität
Die anfängliche Anzahl von Instanzen, die für jede Rolle ausgeführt werden, ist in der Konfiguration jeder Rolle festgelegt. Administratoren sollten zu Beginn jede Rolle so konfigurieren, dass sie je nach erwarteter Auslastung mit mindestens zwei Instanzen ausgeführt wird. Sie können Rolleninstanzen aber problemlos nach oben oder unten skalieren, wenn sich Nutzungsmuster ändern. Dies ist manuell im Azure-Portal möglich, oder Sie können den Prozess automatisieren, indem Sie Windows PowerShell, die Dienstverwaltungs-API oder Drittanbietertools verwenden. Weitere Informationen finden Sie unter [Automatisches Skalieren einer Anwendung](/azure/cloud-services/cloud-services-how-to-scale/).

### <a name="partitioning"></a>Partitionierung
Der Azure Fabric Controller verwendet zwei Arten von Partitionen:

* Eine *Updatedomäne* wird zum Aktualisieren der Rolleninstanzen eines Diensts in Gruppen verwendet. Azure stellt Dienstinstanzen in mehreren Updatedomänen bereit. Für ein direktes Update fährt der Fabric Controller alle Instanzen in einer Updatedomäne herunter, aktualisiert sie und startet sie neu. Anschließend fährt der Fabric Controller mit der nächsten Updatedomäne fort. Dieser Ansatz verhindert, dass während des Aktualisierungsvorgangs der komplette Dienst nicht verfügbar ist.
* Eine *Fehlerdomäne* definiert potenzielle Hardware- oder Netzwerkfehlerstellen. Bei Rollen mit mehr als einer Instanz stellt der Fabric Controller sicher, dass die Instanzen auf mehrere Fehlerdomänen verteilt werden, um zu verhindern, dass isolierte Hardwarefehler zu Dienstunterbrechungen führen. Sämtliche Risiken hinsichtlich Server- und Clusterfehlern werden durch Fehlerdomänen abgedeckt.

Gemäß der [Azure-Vereinbarung zum Servicelevel (SLA)](https://azure.microsoft.com/support/legal/sla/) garantiert Microsoft eine externe Konnektivität von mindestens 99,95%, wenn zwei oder mehr Webrolleninstanzen in verschiedenen Fehler- und Upgradedomänen bereitgestellt werden. Im Gegensatz zu Updatedomänen gibt es keine Möglichkeit, die Anzahl von Fehlerdomänen zu steuern. Azure weist Fehlerdomänen automatisch zu und verteilt Rolleninstanzen auf diese. Mindestens die ersten beiden Instanzen jeder Rolle werden in verschiedenen Fehler- und Upgradedomänen platziert, um sicherzustellen, dass jede Rolle mit mindestens zwei Instanzen die SLA erfüllt. Dies ist im folgenden Diagramm dargestellt:

![Vereinfachte Darstellung der Fehlerdomänenisolation](./images/technical-guidance-recovery-local-failures/partitioning-1.png)

### <a name="load-balancing"></a>Lastenausgleich
Jeglicher eingehender Datenverkehr einer Webrolle wird durch einen zustandslosen Lastenausgleich geleitet, der Clientanforderungen zwischen den Rolleninstanzen verteilt. Einzelne Rolleninstanzen besitzen keine öffentlichen IP-Adressen und sind nicht direkt aus dem Internet erreichbar. Webrollen sind zustandslos, sodass alle Clientanforderungen an jede Rolleninstanz weitergeleitet werden können. Alle 15 Sekunden wird ein [StatusCheck](https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleenvironment.statuscheck.aspx) -Ereignis ausgelöst. Sie können es verwenden, um anzugeben, ob die Rolle für den Empfang von Datenverkehr bereit ist oder ob sie beschäftigt ist und aus der Lastenausgleichsrotation herausgenommen werden sollte.

## <a name="virtual-machines"></a>Virtual Machines
Virtuelle Azure-Computer unterscheiden sich von PaaS-Computerollen (Platform as a Service) im Zusammenhang mit Hochverfügbarkeit in mehreren Punkten. In einigen Fällen müssen Sie zusätzliche Aufgaben ausführen, um Hochverfügbarkeit sicherzustellen.

### <a name="disk-durability"></a>Dauerhaftigkeit von Datenträgern
Im Gegensatz zu PaaS-Rolleninstanzen sind auf virtuellen Computern gespeicherte Daten persistent, auch wenn der virtuelle Computer verschoben wird. Virtuelle Azure-Computer verwenden VM-Datenträger, die in Azure Storage als Blobs vorhanden sind. Aufgrund der Verfügbarkeitsmerkmale von Azure Storage sind die Daten, die auf Laufwerken virtueller Computer gespeichert werden, ebenfalls hoch verfügbar.

Beachten Sie, dass das Laufwerk D (auf Windows-VMs) eine Ausnahme von dieser Regel darstellt. Das Laufwerk D ist eigentlich ein physischer Speicher auf dem Rackserver, der den virtuellen Computer hostet, und die Daten dieses Laufwerks gehen verloren, wenn der virtuelle Computer recycelt wird. Das Laufwerk D ist nur für die temporäre Speicherung vorgesehen. Unter Linux macht Azure den lokalen temporären Datenträger „normalerweise“ (aber nicht immer) als /dev/sdb-Blockgerät verfügbar. Häufig wird er vom Azure Linux-Agent als „/mnt/resource“- oder „/mnt“-Bereitstellungspunkt bereitgestellt (konfigurierbar über „/etc/waagent.conf“).

### <a name="partitioning"></a>Partitionierung
Azure bietet native Unterstützung für die Ebenen einer PaaS-Anwendung (Webrolle und Workerrolle) und kann diese daher richtig auf Fehler- und Updatedomänen verteilen. Die Ebenen in einer IaaS-Anwendung (Infrastructure as a Service) müssen dagegen manuell über Verfügbarkeitsgruppen definiert werden. Für ein IaaS-SLA sind Verfügbarkeitsgruppen erforderlich.

![Verfügbarkeitsgruppen für virtuelle Azure-Computer](./images/technical-guidance-recovery-local-failures/partitioning-2.png)

Im vorherigen Diagramm sind die IIS-Ebene (Internet Information Services), die als Web-App-Ebene fungiert, und die SQL-Ebene, die als Datenebene fungiert, verschiedenen Verfügbarkeitsgruppen zugewiesen. Indem virtuelle Computer auf verschiedene Fehlerdomänen verteilt werden, wird sichergestellt, dass alle Instanzen jeder Ebene über Hardwareredundanz verfügen und während einer Aktualisierung nicht ganze Ebenen heruntergefahren werden.

### <a name="load-balancing"></a>Lastenausgleich
Wenn der Datenverkehr auf die virtuellen Computer verteilt werden soll, müssen Sie die virtuellen Computer in einer Anwendung gruppieren und den Lastenausgleich über einen bestimmten TCP- oder UDP-Endpunkt ausführen. Weitere Informationen finden Sie unter [Lastenausgleich zwischen virtuellen Computern](/azure/virtual-machines/virtual-machines-linux-load-balance/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Wenn die virtuellen Computer Eingaben aus einer anderen Quelle erhalten (beispielsweise einem Warteschlangenmechanismus), ist kein Lastenausgleich erforderlich. Der Lastenausgleich verwendet eine grundlegende Integritätsprüfung, um zu ermitteln, ob der Datenverkehr an den Knoten gesendet werden soll. Sie können auch selbst Tests erstellen, um anwendungsspezifische Integritätsmetriken zu implementieren, die bestimmen, ob der virtuelle Computer Datenverkehr empfangen soll.

## <a name="storage"></a>Speicher
Azure Storage ist der grundlegende permanente Datendienst für Azure. Er stellt Blob-, Tabellen-, Warteschlangen- und VM-Datenträgerspeicher bereit. Storage verwendet eine Kombination aus Replikation und Ressourcenverwaltung, um innerhalb eines einzelnen Rechenzentrums Hochverfügbarkeit bereitzustellen. Mit der Vereinbarung zum Servicelevel (SLA) zur Verfügbarkeit von Azure Storage wird gewährleistet, dass mindestens 99,9% der Zeit Folgendes sichergestellt ist:

* Richtig formatierte Anforderungen zum Hinzufügen, Aktualisieren, Lesen und Löschen von Daten werden erfolgreich und korrekt verarbeitet.
* Speicherkonten verfügen über eine Verbindung mit dem Internetgateway.

### <a name="replication"></a>Replikation
Die Dauerhaftigkeit von Daten für Azure Storage wird ermöglicht, indem mehrere Kopien aller Daten auf verschiedenen Laufwerken gespeichert werden, die sich auf voneinander vollständig unabhängigen physischen Speichersubsystemen innerhalb der Region befinden. Daten werden synchron repliziert, und für alle Kopien wird ein Commit ausgeführt, bevor ein Schreibvorgang bestätigt wird. Azure Storage weist eine hohe Konsistenz auf, was bedeutet, dass Lesevorgänge garantiert die neuesten Schreibvorgänge widerspiegeln. Darüber hinaus werden Kopien der Daten fortlaufend überprüft, um Datenverfall zu erkennen und zu beheben – eine häufig übersehene Bedrohung der Integrität gespeicherter Daten.

Dienste profitieren bereits durch die reine Verwendung von Azure Storage von der Replikation. Der Dienstentwickler muss für die Wiederherstellung nach einem lokalen Ausfall keine zusätzlichen Schritte ausführen.

### <a name="resource-management"></a>Ressourcenverwaltung
Speicherkonten, die nach Mai 2014 erstellt wurden, können auf bis zu 500 TB skaliert werden (zuvor lag die Höchstgrenze bei 200 TB). Wenn zusätzlicher Speicherplatz erforderlich ist, müssen Anwendungen so entworfen werden, dass sie mehrere Speicherkonten verwenden.

### <a name="virtual-machine-disks"></a>VM-Datenträger
Ein VM-Datenträger wird als Seitenblob in Azure Storage gespeichert und erhält damit die gleichen Dauerhaftigkeits- und Skalierbarkeitseigenschaften wie Blobspeicher. Aufgrund dieses Designs sind die Daten auf dem Datenträger eines virtuellen Computers persistent. Dies gilt selbst dann, wenn der Server ausfällt, auf dem der virtuelle Computer ausgeführt wird, und der virtuelle Computer auf einem anderen Server neu gestartet werden muss.

## <a name="database"></a>Datenbank
### <a name="sql-database"></a>SQL-Datenbank
Azure SQL-Datenbank ermöglicht „Database as a Service“. Hierbei können Anwendungen relationale Datenbanken schnell bereitstellen, Daten einfügen und die Datenbanken abfragen. Azure SQL-Datenbank stellt viele der vertrauten SQL Server-Features und -Funktionen bereit und entbindet Sie gleichzeitig vom Aufwand für Hardware, Konfiguration, Patching und Resilienz.

> [!NOTE]
> Azure SQL-Datenbank weist keine 1:1-Featureübereinstimmung mit SQL Server auf. Es sollen andere Anforderungen erfüllt werden, und zwar speziell für Cloudanwendungen (elastische Skalierung, „Database as a Service“ zur Reduzierung von Wartungskosten usw.). Weitere Informationen finden Sie unter [Wählen Sie eine SQL Server-Cloudoption: Azure SQL-Datenbank (PaaS) oder SQL Server auf Azure-VMs (IaaS)](/azure/sql-database/sql-database-paas-vs-sql-server-iaas/).
> 
> 

#### <a name="replication"></a>Replikation
Azure SQL-Datenbank stellt integrierte Resilienz gegenüber Fehlern auf Knotenebene bereit. Alle Schreibvorgänge in eine Datenbank werden automatisch mithilfe eines Commitverfahrens per Quorum auf mindestens zwei Hintergrundknoten repliziert. (Der primäre Knoten und mindestens ein sekundärer Knoten müssen bestätigen, dass die Aktivität in das Transaktionsprotokoll geschrieben wird, bevor die Transaktion als erfolgreich betrachtet und zurückgegeben wird.) Im Fall eines Knotenfehlers führt die Datenbank automatisch ein Failover auf eins der sekundären Replikate durch. Dies verursacht eine vorübergehende Unterbrechung der Verbindung für Clientanwendungen. Aus diesem Grund müssen alle Azure SQL-Datenbank-Clients die Verarbeitung vorübergehender Verbindungen in irgendeiner Form implementieren. Weitere Informationen finden Sie unter [Anleitung zu dienstspezifischen Wiederholungsmechanismen](/azure/best-practices-retry-service-specific/).

#### <a name="resource-management"></a>Ressourcenverwaltung
Jede Datenbank wird bei der Erstellung mit einer Größenbeschränkung konfiguriert. Die derzeit verfügbare maximale Größe beträgt 1 TB (Größenbeschränkungen variieren je nach Dienstebene, siehe [Tarife und Leistungsstufen von Azure SQL-Datenbanken](/azure/sql-database/sql-database-resource-limits/#service-tiers-and-performance-levels)). Wenn eine Datenbank den oberen Grenzwert erreicht, werden weitere INSERT- oder UPDATE-Befehle zurückgewiesen. (Das Abfragen und Löschen von Daten ist weiterhin möglich.)

Innerhalb einer Datenbank verwendet Azure SQL-Datenbank ein Fabric zur Verwaltung von Ressourcen. Anstelle eines Fabric Controllers wird jedoch eine Ringtopologie eingesetzt, um Fehler zu erkennen. Jedes Replikat in einem Cluster besitzt zwei Nachbarn und ist dafür zuständig zu erkennen, wenn diese ausfallen. Wenn ein Replikat ausfällt, lösen seine Nachbarn einen Reconfiguration Agent aus, um das Replikat auf einem anderen Computer neu zu erstellen. Mithilfe einer Engine-Drosselung wird sichergestellt, dass ein logischer Server nicht zu viele Ressourcen auf einem Computer verwendet oder die physischen Grenzwerte des Computers überschreitet.

### <a name="elasticity"></a>Elastizität
Wenn für die Anwendung mehr Speicherplatz als das von der Datenbank vorgegebene Limit von 1 TB erforderlich ist, muss ein Verfahren zur horizontalen Hochskalierung implementiert werden. Sie führen die horizontale Hochskalierung mit Azure SQL-Datenbank durch, indem Sie Daten über mehrere SQL-Datenbanken hinweg manuell partitionieren (auch als Sharding bezeichnet). Ein solcher Ansatz ermöglicht es, mithilfe der Skalierung einen nahezu linearen Kostenanstieg zu erreichen. Durch elastisches Wachstum bzw. bedarfsgesteuerte Kapazität können Systeme mit inkrementellem Kostenanstieg wachsen, da Datenbanken basierend auf der pro Tag tatsächlich durchschnittlich genutzten Größe berechnet werden, nicht basierend auf der möglichen Maximalgröße.

## <a name="sql-server-on-virtual-machines"></a>SQL Server auf virtuellen Computern
Durch das Installieren von SQL Server auf virtuellen Azure-Computern (Version 2014 oder höher) können Sie die herkömmlichen Verfügbarkeitsfunktionen von SQL Server nutzen. Beispiele hierfür sind AlwaysOn-Verfügbarkeitsgruppen oder die Datenbankspiegelung. Beachten Sie, dass für virtuelle Computer, Speicher und Netzwerkressourcen von Azure andere Betriebsmerkmale gelten als für eine lokale, nicht virtualisierte IT-Infrastruktur. Für eine erfolgreiche Implementierung einer SQL Server-Lösung für Hochverfügbarkeit und Notfallwiederherstellung in Azure sollten Sie diese Unterschiede kennen und Ihre Lösung daran anpassen.

### <a name="high-availability-nodes-in-an-availability-set"></a>Hochverfügbarkeitsknoten in einer Verfügbarkeitsgruppe
Wenn Sie eine Hochverfügbarkeitslösung in Azure implementieren, können Sie mithilfe einer Verfügbarkeitsgruppe in Azure Hochverfügbarkeitsknoten in separaten Fehler- und Upgradedomänen platzieren. Bei einer Verfügbarkeitsgruppe handelt es sich um ein Azure-Konzept. Sie sollten diese bewährte Methode befolgen, um sicherzustellen, dass Ihre Datenbanken tatsächlich hoch verfügbar sind – unabhängig davon, ob Sie AlwaysOn-Verfügbarkeitsgruppen, Datenbankspiegelung oder andere Verfahren verwenden. Wenn Sie diese bewährte Methode nicht befolgen, gehen Sie unter Umständen fälschlicherweise davon aus, dass Ihr System hoch verfügbar ist. In Wahrheit können Ihre Knoten jedoch alle gleichzeitig ausfallen, weil sie zufällig in der gleichen Fehlerdomäne in der Azure-Region platziert wurden.

Diese Empfehlung gilt nicht im gleichen Ausmaß für den Protokollversand. Es handelt sich hierbei um eine Funktion der Notfallwiederherstellung. Sie sollten sicherstellen, dass die Server in separaten Azure-Regionen ausgeführt werden. Der Definition nach handelt es sich bei diesen Regionen um separate Fehlerdomänen.

Damit sich Azure Cloud Services-VMs, die über das klassische Portal bereitgestellt werden, in derselben Verfügbarkeitsgruppe befinden, müssen Sie sie in derselben Cloud Service-Instanz bereitstellen. Für VMs, die mit dem Azure Resource Manager (aktuelles Portal) bereitgestellt werden, besteht diese Einschränkung nicht. Für VMs, die mit dem klassischen Portal in Azure Cloud Services bereitgestellt werden, können nur Knoten in derselben Cloud Services-Instanz an derselben Verfügbarkeitsgruppe beteiligt sein. Darüber hinaus sollten sich die Cloud Services-VMs in demselben virtuellen Netzwerk befinden, um sicherzustellen, dass sie auch nach einer Dienstreparatur ihre IPs beibehalten. So können Störungen beim DNS-Update vermieden werden.

### <a name="azure-only-high-availability-solutions"></a>Nur in Azure: Lösungen mit hoher Verfügbarkeit
Sie können eine Lösung mit hoher Verfügbarkeit für SQL Server-Datenbanken in Azure mit AlwaysOn-Verfügbarkeitsgruppen oder per Datenbankspiegelung umsetzen.

Das folgende Diagramm veranschaulicht die Architektur von AlwaysOn-Verfügbarkeitsgruppen, die auf Azure Virtual Machines ausgeführt werden. Das Diagramm stammt aus dem detaillierten Artikel zu diesem Thema: [Hochverfügbarkeit und Notfallwiederherstellung für SQL Server auf virtuellen Azure-Computern](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

![AlwaysOn-Verfügbarkeitsgruppen in Microsoft Azure](./images/technical-guidance-recovery-local-failures/high_availability_solutions-1.png)

Sie können eine AlwaysOn-Verfügbarkeitsgruppe auch als automatische End-to-End-Bereitstellung auf virtuellen Azure-Computern implementieren, indem Sie die AlwaysOn-Vorlage im Azure-Portal verwenden. Weitere Informationen finden Sie unter [SQL Server AlwaysOn Offering in Microsoft Azure Portal Gallery](https://blogs.technet.microsoft.com/dataplatforminsider/2014/08/25/sql-server-alwayson-offering-in-microsoft-azure-portal-gallery/)(in englischer Sprache).

Das folgende Diagramm veranschaulicht die Verwendung der Datenbankspiegelung auf Azure Virtual Machines. Auch dieses Diagramm stammt aus dem detaillierten Artikel [Hochverfügbarkeit und Notfallwiederherstellung für SQL Server auf virtuellen Azure-Computern](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

![Datenbankspiegelung in Microsoft Azure](./images/technical-guidance-recovery-local-failures/high_availability_solutions-2.png)

> [!NOTE]
> Für beide Architekturen ist ein Domänencontroller erforderlich. Bei der Datenbankspiegelung können jedoch Serverzertifikate verwendet werden, sodass kein Domänencontroller benötigt wird.
> 
> 

## <a name="other-azure-platform-services"></a>Weitere Azure Platform-Dienste
Anwendungen, die unter Azure erstellt werden, profitieren von Plattformfunktionen für die Wiederherstellung nach lokalen Ausfällen. In einigen Fällen können Sie bestimmte Aktionen durchführen, um die Verfügbarkeit für Ihr jeweiliges Szenario zu erhöhen.

### <a name="service-bus"></a>SERVICE BUS
Um die Risiken durch einen vorübergehenden Ausfall von Azure Service Bus zu minimieren, können Sie eine dauerhafte clientseitige Warteschlange erstellen. Diese Warteschlange verwendet vorübergehend einen alternativen, lokalen Speichermechanismus, um Nachrichten zu speichern, die der Service Bus-Warteschlange nicht hinzugefügt werden können. Die Anwendung kann entscheiden, wie die temporär gespeicherten Nachrichten verarbeitet werden sollen, nachdem der Dienst wiederhergestellt wurde. Weitere Informationen finden Sie unter [Bewährte Methoden für Leistungsoptimierungen mithilfe von Service Bus-Brokermessaging](/azure/service-bus-messaging/service-bus-performance-improvements/) sowie unter [Service Bus (Notfallwiederherstellung)](recovery-loss-azure-region.md#other-azure-platform-services).

### <a name="hdinsight"></a>HDInsight
Die mit Azure HDInsight verknüpften Daten werden standardmäßig in Azure Blob Storage gespeichert. Azure Storage stellt Eigenschaften für hohe Verfügbarkeit und Dauerhaftigkeit für Blob Storage bereit. Die Verarbeitung über mehrere Knoten hinweg, die für Hadoop MapReduce-Aufträge erforderlich ist, erfolgt auf einem vorübergehenden Hadoop Distributed File System (HDFS), das bei Bedarf für HDInsight bereitgestellt wird. Die Ergebnisse eines MapReduce-Auftrags werden standardmäßig auch in Azure Blob Storage gespeichert, sodass die verarbeiteten Daten dauerhaft und hoch verfügbar sind, nachdem die Bereitstellung des Hadoop-Clusters aufgehoben wurde. Weitere Informationen finden Sie unter [HDInsight (Notfallwiederherstellung)](recovery-loss-azure-region.md#other-azure-platform-services).

## <a name="checklists-for-local-failures"></a>Prüflisten für lokale Fehler
### <a name="cloud-services"></a>Cloud Services
1. Lesen Sie den Abschnitt zu Cloud Services in diesem Dokument.
2. Konfigurieren Sie mindestens zwei Instanzen für jede Rolle.
3. Speichern Sie den Status in einem permanenten Speicher, nicht in Rolleninstanzen.
4. Verarbeiten Sie das StatusCheck-Ereignis richtig.
5. Fassen Sie aufeinander bezogene Änderungen in Transaktionen zusammen, sofern dies möglich ist.
6. Überprüfen Sie, ob die Workerrollenaufgaben idempotent sind und neu gestartet werden können.
7. Rufen Sie Vorgänge so lange auf, bis sie erfolgreich sind.
8. Erwägen Sie die Verwendung von Strategien für eine automatische Skalierung.

### <a name="virtual-machines"></a>Virtual Machines
1. Lesen Sie den Abschnitt zu Virtual Machines in diesem Dokument.
2. Verwenden Sie Laufwerk D nicht zum dauerhaften Speichern.
3. Gruppieren Sie Computer einer Dienstebene in einer Verfügbarkeitsgruppe.
4. Konfigurieren Sie den Lastenausgleich und optionale Tests.

### <a name="storage"></a>Speicher
1. Lesen Sie den Abschnitt zu Storage in diesem Dokument.
2. Verwenden Sie mehrere Speicherkonten, wenn Daten oder Bandbreite das Kontingent überschreiten.

### <a name="sql-database"></a>SQL-Datenbank
1. Lesen Sie den Abschnitt zu SQL-Datenbank in diesem Dokument.
2. Implementieren Sie eine Wiederholungsrichtlinie zum Verarbeiten von vorübergehenden Fehlern.
3. Verwenden Sie Partitionierung/Sharding als Strategie zum horizontalen Hochskalieren.

### <a name="sql-server-on-virtual-machines"></a>SQL Server auf virtuellen Computern
1. Lesen Sie den Abschnitt zu SQL Server auf Virtual Machines in diesem Dokument.
2. Befolgen Sie die vorherigen Empfehlungen für Virtual Machines.
3. Verwenden Sie die Hochverfügbarkeitsfunktionen von SQL Server, z.B. AlwaysOn.

### <a name="service-bus"></a>SERVICE BUS
1. Lesen Sie den Abschnitt zu Service Bus in diesem Dokument.
2. Erstellen Sie ggf. eine dauerhafte clientseitige Warteschlange.

### <a name="hdinsight"></a>HDInsight
1. Lesen Sie den Abschnitt zu HDInsight in diesem Dokument.
2. Es sind keine zusätzlichen Schritte erforderlich, um die Verfügbarkeit bei lokalen Fehlern sicherzustellen.

