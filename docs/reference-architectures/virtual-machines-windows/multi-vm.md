---
title: "Ausführen von VMs mit Lastenausgleich in Azure zur Steigerung von Skalierbarkeit und Verfügbarkeit"
description: "Vorgehensweise zum Ausführen mehrerer Windows-VMs in Azure zur Steigerung von Skalierbarkeit und Verfügbarkeit"
author: telmosampaio
ms.date: 09/07/2017
pnp.series.title: Windows VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
ms.openlocfilehash: d38cfb41255c547f1f1e87ef289c7a79033df778
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="run-load-balanced-vms-for-scalability-and-availability"></a>Ausführen von VMs mit Lastenausgleich zur Steigerung von Skalierbarkeit und Verfügbarkeit

Diese Referenzarchitektur zeigt eine Reihe von bewährten Methoden für die Ausführung mehrerer virtueller Windows-Computer (VMs) in einer Skalierungsgruppe hinter einem Lastenausgleich, um die Verfügbarkeit und Skalierbarkeit zu verbessern. Diese Architektur kann für jede zustandslose Workload (z.B. einen Webserver) verwendet werden und stellt damit einen Baustein für die Bereitstellung von n-schichtigen Anwendungen dar. [**Stellen Sie diese Lösung bereit**.](#deploy-the-solution)

![[0]][0]

*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*

## <a name="architecture"></a>Architektur

Diese Architektur basiert auf der in [Ausführen eines virtuellen Windows-Computers in Azure][single vm] gezeigten Architektur. Die dort gegebenen Empfehlungen gelten auch für diese Architektur.

In dieser Architektur wird eine Workload auf mehrere VM-Instanzen verteilt. Es gibt eine einzelne öffentliche IP-Adresse, und der Internetdatenverkehr wird durch einen Lastenausgleich auf die VMs verteilt. Diese Architektur kann für eine einschichtige Anwendung (z.B. eine zustandslose Webanwendung) verwendet werden.

Diese Architektur besteht aus den folgenden Komponenten:

* **Ressourcengruppe:** [*Ressourcengruppen*][resource-manager-overview] dienen zum Gruppieren von Ressourcen, damit sie nach Lebensdauer, Besitzer und anderen Kriterien verwaltet werden können.
* **Virtuelles Netzwerk (VNet) und Subnetz_** Jede VM in Azure wird in einem VNet bereitgestellt, das weiter in Subnetze unterteilt wird.
* **Azure Load Balancer**: Beim [Lastenausgleich] werden die eingehenden Internetanforderungen an die VM-Instanzen verteilt. 
* **Öffentliche IP-Adresse**: Beim Lastenausgleich ist für den Empfang von Internetdatenverkehr eine öffentliche IP-Adresse erforderlich.
* **VM-Skalierungsgruppe**: Eine [VM-Skalierungsgruppe][vm-scaleset] ist eine Gruppe von identischen VMs, die für das Hosten einer Workload verwendet werden. Skalierungsgruppen ermöglichen das horizontale Hoch- oder Herunterskalieren der Anzahl der VMs – und zwar sowohl manuell als auch basierend auf vordefinierten Regeln.
* **Verfügbarkeitsgruppe**. Die [Verfügbarkeitsgruppe][availability set] enthält die VMs und berechtigt die VMs damit für eine höhere [Vereinbarung zum Servicelevel (SLA)][vm-sla]. Damit eine höhere SLA angewendet werden kann, muss die Verfügbarkeitsgruppe mindestens zwei VMs enthalten. Verfügbarkeitsgruppen sind in Skalierungsgruppen implizit. Wenn Sie VMs außerhalb einer Skalierungsgruppe erstellen, müssen Sie die Verfügbarkeitsgruppe separat erstellen.
* **Verwaltete Datenträger**: Azure Managed Disks verwaltet die VHD-Dateien (virtuelle Festplatte) für die VM-Datenträger. 
* **Speicher**: Erstellen Sie ein Azure Storage-Konto zum Speichern der Diagnoseprotokolle für die VMs.

## <a name="recommendations"></a>Recommendations

Ihre Anforderungen können von der hier beschriebenen Architektur abweichen. Verwenden Sie diese Empfehlungen als Ausgangspunkt. 

### <a name="availability-and-scalability-recommendations"></a>Empfehlungen zur Verfügbarkeit und Skalierbarkeit

Eine Möglichkeit zur Steigerung der Verfügbarkeit und Skalierbarkeit stellt die Verwendung einer [VM-Skalierungsgruppe][vmss] dar. VM-Skalierungsgruppen ermöglichen die Bereitstellung und Verwaltung einer Gruppe von identischen VMs. Skalierungsgruppen unterstützen die automatische Skalierung basierend auf Leistungsmetriken. Mit zunehmender Last auf den VMs werden dem Lastenausgleich automatisch zusätzliche VMs hinzugefügt. Erwägen Sie die Verwendung von Skalierungsgruppen, wenn Sie VMs schnell horizontal hochskalieren müssen oder eine automatische Skalierung erforderlich ist.

Standardmäßig verwenden Skalierungsgruppen eine „Überbereitstellung“. Das bedeutet, dass die Skalierungsgruppe anfänglich mehr VMs bereitstellt, als Sie anfordern, und diese zusätzlichen VMs später löscht. Hierdurch wird eine insgesamt bessere Bereitstellung von VMs sichergestellt. Wenn Sie keine [verwalteten Datenträger](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks) verwenden, werden bei aktivierter Überbereitstellung nicht mehr als 20 VMs pro Speicherkonto bzw. bei deaktivierter Überbereitstellung nicht mehr als 40 VMs empfohlen.

Es gibt zwei grundlegende Methoden für die Konfiguration von VMs in einer Skalierungsgruppe:

- Verwenden Sie Erweiterungen, um die VM zu konfigurieren, nachdem diese bereitgestellt wurde. Bei diesem Ansatz dauert das Starten neuer VM-Instanzen länger als bei VMs ohne Erweiterungen.

- Stellen Sie einen [verwalteten Datenträger](/azure/storage/storage-managed-disks-overview) mit einem benutzerdefinierten Datenträgerimage bereit. Diese Option kann möglicherweise schneller bereitgestellt werden. Allerdings müssen Sie das Image auf dem neuesten Stand halten.

Weitere Überlegungen finden Sie unter [Überlegungen zum Entwurf von Skalierungsgruppen][vmss-design].

> [!TIP]
> Wenn Sie eine Lösung für die automatische Skalierung verwenden, testen Sie diese im Voraus mit Workloads auf Produktionsebene.

Wenn Sie keine Skalierungsgruppe verwenden, sollten Sie zumindest eine Verfügbarkeitsgruppe verwenden. Erstellen Sie zur Unterstützung der [Verfügbarkeits-SLA für Azure-VMs][vm-sla] mindestens zwei VMs in der Verfügbarkeitsgruppe. Azure Load Balancer erfordert auch, dass VMs mit Lastenausgleich derselben Verfügbarkeitsgruppe angehören.

Für jedes Azure-Abonnement gelten Standardeinschränkungen, die auch eine maximale Anzahl von VMs pro Region vorschreiben. Sie können den Grenzwert erhöhen, indem Sie eine Supportanfrage einreichen. Weitere Informationen finden Sie unter [Einschränkungen für Azure-Abonnements und Dienste, Kontingente und Einschränkungen][subscription-limits].

### <a name="network-recommendations"></a>Netzwerkempfehlungen

Platzieren Sie die VMs im gleichen Subnetz. Machen Sie die VMs nicht direkt über das Internet verfügbar, sondern weisen Sie jeder VM stattdessen eine private IP-Adresse zu. Clients verwenden für Verbindungen die öffentliche IP-Adresse des Lastenausgleichs.

Wenn Sie sich bei VMs hinter dem Lastenausgleich anmelden möchten, erwägen Sie das Hinzufügen einer einzelnen VM als geschützten Host/Jumpbox mit einer öffentlichen IP-Adresse, auf der Sie sich anmelden können. Melden Sie sich dann auf den VMs hinter dem Lastenausgleich von der Jumpbox aus an. Alternativ können Sie für denselben Zweck NAT-Eingangsregeln im Lastenausgleich konfigurieren. Wenn Sie n-schichtige Workloads oder mehrere Workloads hosten, stellt eine Jumpbox jedoch eine bessere Lösung dar.

### <a name="load-balancer-recommendations"></a>Empfehlungen für den Lastenausgleich

Fügen Sie alle VMs in der Verfügbarkeitsgruppe dem Back-End-Adresspool des Lastenausgleichs hinzu.

Definieren Sie Lastenausgleichsregeln, um Netzwerkdatenverkehr an die VMs weiterzuleiten. Um beispielsweise HTTP-Datenverkehr zu aktivieren, erstellen Sie eine Regel, die Port 80 aus der Front-End-Konfiguration Port 80 im Back-End-Adresspool zuordnet. Wenn ein Client eine HTTP-Anforderung an Port 80 sendet, wählt der Lastenausgleich mithilfe eines [Hashalgorithmus][load balancer hashing], der die Quell-IP-Adresse enthält, eine Back-End-IP-Adresse aus. Auf diese Weise werden die Clientanforderungen auf die VMs verteilt.

Verwenden Sie zum Weiterleiten von Datenverkehr an eine bestimmte VM NAT-Regeln. Erstellen Sie z.B. zum Aktivieren von RDP auf den VMs eine separate NAT-Regel für jede VM. Jede Regel sollte Port 3389, dem Standardport für RDP, eine eigene Portnummer zuordnen. Verwenden Sie z.B. Port 50001 für „VM1“, Port 50002 für „VM2“ usw. Weisen Sie die NAT-Regeln den Netzwerkkarten der VMs zu.

### <a name="storage-account-recommendations"></a>Empfehlungen für Speicherkonten

Erstellen Sie separate Azure Storage-Konten für jede VM, um die virtuellen Festplatten (VHDs) zu speichern und das Überschreiten der [IOPS-Grenzwerte][vm-disk-limits] (Input/Output Operations Per Second) für Speicherkonten zu vermeiden.

Wir empfehlen die Verwendung von [verwalteten Datenträgern](/azure/storage/storage-managed-disks-overview) mit [Storage Premium][premium]. Bei verwalteten Datenträgern ist kein Speicherkonto erforderlich. Sie geben einfach die Größe und den Typ des Datenträgers an, woraufhin dieser mit Hochverfügbarkeit bereitgestellt wird.

Erstellen Sie ein Speicherkonto für Diagnoseprotokolle. Dieses Speicherkonto kann von allen VMs gemeinsam verwendet werden. Dabei kann es sich um ein nicht verwaltetes Speicherkonto mit Standarddatenträgern handeln.

## <a name="availability-considerations"></a>Überlegungen zur Verfügbarkeit

Die Verfügbarkeitsgruppe macht Ihre Anwendung gegenüber geplanten und ungeplanten Wartungsereignisse stabiler.

* *Geplante Wartungen* treten auf, wenn Microsoft die zugrunde liegende Plattform aktualisiert. Dabei müssen VMs in einigen Fällen neu gestartet werden. Azure stellt sicher, dass die VMs in einer Verfügbarkeitsgruppe nicht alle gleichzeitig neu gestartet werden. Mindestens eine VM wird weiter ausgeführt, während die anderen neu gestartet werden.
* *Ungeplante Wartungen* treten bei einem Hardwarefehler auf. Azure stellt sicher, dass die VMs in einer Verfügbarkeitsgruppe über mehrere Serverracks bereitgestellt wurden. Dies trägt dazu bei, die Auswirkungen von Hardwarefehlern, Netzwerkausfällen, Unterbrechungen der Stromversorgung usw. zu reduzieren.

Weitere Informationen finden Sie unter [Verwalten der Verfügbarkeit virtueller Windows-Computer in Azure][availability set]. Das folgende Video bietet einen guten Überblick über Verfügbarkeitsgruppen: [How Do I Configure an Availability Set to Scale VMs][availability set ch9] (Wie konfiguriere ich eine Verfügbarkeitsgruppe zum Skalieren von VMs?).

> [!WARNING]
> Sie müssen die Verfügbarkeitsgruppe unbedingt bei der Bereitstellung der VM konfigurieren. Derzeit gibt es keine Möglichkeit, einer Verfügbarkeitsgruppe nach dem Bereitstellen der VM eine Resource Manager-VM hinzuzufügen.

Der Lastenausgleich verwendet [Integritätstests] zur Überwachung der Verfügbarkeit von VM-Instanzen. Wenn eine Instanz bei einem Test nicht innerhalb eines Timeoutzeitraums erreicht werden kann, beendet der Lastenausgleich das Senden von Datenverkehr an diese VM. Der Lastenausgleich führt aber weiterhin Tests durch, und wenn die VM wieder erreichbar ist, sendet der Lastenausgleich auch wieder Datenverkehr an diese VM.

Im Folgenden werden einige Empfehlungen für Integritätstests durch den Lastenausgleich vorgestellt:

* Die Tests können für HTTP oder TCP durchgeführt werden. Wenn Ihre VMs einen HTTP-Server ausführen, erstellen Sie einen HTTP-Test. Erstellen Sie andernfalls einen TCP-Test.
* Geben Sie für einen HTTP-Test den Pfad zu einem HTTP-Endpunkt an. Der Test überprüft, ob von diesem Pfad eine HTTP 200-Antwort empfangen wird. Dies kann den Stammpfad („/“) oder ein Endpunkt zur Integritätsüberwachung sein, der benutzerdefinierte Logik zum Überprüfen der Integrität der Anwendung implementiert. Der Endpunkt muss anonyme HTTP-Anforderungen zulassen.
* Der Test wird von einer [bekannten][health-probe-ip] IP-Adresse (168.63.129.16) gesendet. Stellen Sie sicher, dass Datenverkehr an diese oder von dieser IP-Adresse in keinen Firewallrichtlinien oder NSG-Regeln (Netzwerksicherheitsgruppe) blockiert wird.
* Verwenden Sie die [Protokolle der Integritätstests][health probe log], um den Status der Integritätstests anzuzeigen. Aktivieren Sie die Protokollierung für jeden Lastenausgleich im Azure-Portal. Die Protokolle werden in den Azure Blob Storage geschrieben. Die Protokolle zeigen, wie viele VMs am Back-End aufgrund fehlerhafter Testantworten keinen Netzwerkdatenverkehr empfangen.

## <a name="manageability-considerations"></a>Überlegungen zur Verwaltbarkeit

Bei mehreren VMs ist es wichtig, Prozesse zu automatisieren, damit sie zuverlässig und wiederholbar sind. Sie können [Azure Automation][azure-automation] verwenden, um die Bereitstellung, Betriebssystempatches und andere Aufgaben zu automatisieren. Für diesen Zweck können Sie [Azure Automation][azure-automation] verwenden, ein auf Windows PowerShell basierender Automatisierungsdienst. Beispiele für Automatisierungsskripts sind im [Runbookkatalog] auf TechNet verfügbar.

## <a name="security-considerations"></a>Sicherheitshinweise

Virtuelle Netzwerke stellen in Azure eine Isolationsbegrenzung für Datenverkehr dar. VMs in einem VNET können nicht direkt mit VMs in einem anderen VNET kommunizieren. VMs im gleichen VNET können miteinander kommunizieren, sofern Sie den Datenverkehr nicht durch die Erstellung von [Netzwerksicherheitsgruppen][nsg] (NSGs) beschränken. Weitere Informationen finden Sie unter [Microsoft-Clouddienste und Netzwerksicherheit][network-security].

Für eingehenden Internetdatenverkehr definieren die Lastenausgleichsregeln, welcher Datenverkehr an das Back-End weitergeleitet wird. Allerdings unterstützen Lastenausgleichsregeln keine Listen sicherer IP-Adressen. Wenn Sie also einer Liste sicherer IP-Adressen bestimmte öffentliche IP-Adressen hinzufügen möchten, fügen Sie dem Subnetz eine NSG hinzu.

## <a name="deploy-the-solution"></a>Bereitstellen der Lösung

Eine Bereitstellung für diese Architektur ist auf [GitHub][github-folder] verfügbar. Folgendes wird bereitgestellt:

  * Ein virtuelles Netzwerk mit einem einzelnem Subnetz namens **web**, das zum Hosten der VMs verwendet wird
  * Eine VM-Skalierungsgruppe, die VMs mit der neuesten Version von Windows Server 2016 Datacenter Edition enthält. Die automatische Skalierung ist aktiviert.
  * Ein Lastenausgleich, der sich vor der VM-Skalierungsgruppe befindet
  * Eine NSG mit Regeln für den eingehenden Datenverkehr, die HTTP-Datenverkehr an die VM-Skalierungsgruppe zulässt

### <a name="prerequisites"></a>Voraussetzungen

Bevor Sie die Referenzarchitektur in Ihrem eigenen Abonnement bereitstellen können, müssen Sie die folgenden Schritte durchführen.

1. Klonen oder forken Sie das GitHub-Repository [AzureCAT-Referenzarchitekturen][ref-arch-repo], oder laden Sie die zugehörige ZIP-Datei herunter.

2. Vergewissern Sie sich, dass Azure CLI 2.0 auf Ihrem Computer installiert ist. Um die CLI zu installieren, folgen Sie den Anweisungen unter [Installieren von Azure CLI 2.0][azure-cli-2].

3. Installieren Sie das npm-Paket mit den [Azure-Bausteinen][azbb].

4. Melden Sie sich über eine Eingabeaufforderung, eine bash-Eingabeaufforderung oder die PowerShell-Eingabeaufforderung bei Ihrem Azure-Konto an. Verwenden Sie dazu die unten aufgeführten Befehle, und befolgen Sie die Anweisungen.

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>Bereitstellen der Lösung mit azbb

Gehen Sie zum Bereitstellen der einzelnen VM-Beispielworkloads folgendermaßen vor:

1. Navigieren Sie zum Ordner `virtual-machines\multi-vm\parameters\windows` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.

2. Öffnen Sie die Datei `multi-vm-v2.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen an, und speichern Sie die Datei.

  ```bash
  "adminUsername": "",
  "adminPassword": "",
  ```

3. Führen Sie `azbb` aus, um die VMs wie unten dargestellt bereitzustellen.

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p multi-vm-v2.json --deploy
  ```

Weitere Informationen zum Bereitstellen dieser Beispielreferenzarchitektur finden Sie in unserem [GitHub-Repository][git].

<!-- Links -->
[github-folder]: http://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[n-tier-linux]: ../virtual-machines-linux/n-tier.md
[n-tier-windows]: n-tier.md
[single vm]: single-vm.md
[premium]: /azure/storage/common/storage-premium-storage
[naming conventions]: /azure/guidance/guidance-naming-conventions
[vm-scaleset]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[availability set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[availability set ch9]: https://channel9.msdn.com/Series/Microsoft-Azure-Fundamentals-Virtual-Machines/08
[azure-automation]: https://azure.microsoft.com/documentation/services/automation/
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-automation]: /azure/automation/automation-intro
[bastion host]: https://en.wikipedia.org/wiki/Bastion_host
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[health probe log]: /azure/load-balancer/load-balancer-monitor-log
[Integritätstests]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[Lastenausgleich]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load balancer hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[network-security]: /azure/best-practices-network-security
[nsg]: /azure/virtual-network/virtual-networks-nsg
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[Runbookkatalog]: /azure/automation/automation-runbook-gallery#runbooks-in-runbook-gallery
[subscription-limits]: /azure/azure-subscription-service-limits
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[vmss-quickstart]: https://azure.microsoft.com/documentation/templates/?term=scale+set
[VM-sizes]: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
[0]: ./images/multi-vm-diagram.png "Architektur einer Lösung mit mehreren VMs in Azure bestehend aus einer Verfügbarkeitsgruppe mit zwei VMs und einem Lastenausgleich"
[azure-cli-2]: /azure/install-azure-cli?view=azure-cli-latest
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Template-Building-Blocks-Version-2-(Windows)
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm