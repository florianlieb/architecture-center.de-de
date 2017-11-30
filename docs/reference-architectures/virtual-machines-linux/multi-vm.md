---
title: "Ausführen von VMs mit Lastenausgleich in Azure zur Steigerung von Skalierbarkeit und Verfügbarkeit"
description: "Ausführen mehrerer virtueller Linux-Computer in Azure zur Steigerung von Skalierbarkeit und Verfügbarkeit"
author: telmosampaio
ms.date: 09/07/2017
pnp.series.title: Linux VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
ms.openlocfilehash: 0f772cdc596ea96073da3b9b2a2709ce3041889d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="run-load-balanced-vms-for-scalability-and-availability"></a>Ausführen von VMs mit Lastenausgleich zur Steigerung von Skalierbarkeit und Verfügbarkeit

Diese Referenzarchitektur zeigt eine Reihe von bewährten Methoden für die Ausführung mehrerer virtueller Linux-Computer (VMs) in einer Skalierungsgruppe hinter einem Lastenausgleich, um die Verfügbarkeit und Skalierbarkeit zu verbessern. Diese Architektur kann für jede zustandslose Workload (z.B. einen Webserver) verwendet werden. Sie stellt damit einen grundlegender Baustein für die Bereitstellung von n-schichtigen Anwendungen dar. [**Stellen Sie diese Lösung bereit**.](#deploy-the-solution)

![[0]][0]

*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*

## <a name="architecture"></a>Architektur

Diese Architektur basiert auf der in [Ausführen eines virtuellen Linux-Computers in Azure][single vm] gezeigten Architektur. Die dort gegebenen Empfehlungen gelten auch für diese Architektur.

In dieser Architektur wird eine Workload auf mehrere VM-Instanzen verteilt. Es gibt eine einzelne öffentliche IP-Adresse, und der Internetdatenverkehr wird mit einem Lastenausgleich auf die virtuellen Computer verteilt. Diese Architektur kann für Anwendungen mit einer Ebene (z.B. zustandslosen Webanwendungen) verwendet werden.

Die Architektur besteht aus den folgenden Komponenten:

* **Ressourcengruppe:** [*Ressourcengruppen*][resource-manager-overview] dienen zum Gruppieren von Ressourcen, damit sie nach Lebensdauer, Besitzer und anderen Kriterien verwaltet werden können.
* **Virtuelles Netzwerk (VNet) und Subnetz_** Jede VM in Azure wird in einem VNet bereitgestellt, das weiter in Subnetze unterteilt wird.
* **Azure Load Balancer:** Beim [Lastenausgleich] werden die eingehenden Internetanforderungen an die VM-Instanzen verteilt. 
* **Öffentliche IP-Adresse:** Beim Lastenausgleich ist für den Empfang von Internetdatenverkehr eine öffentliche IP-Adresse erforderlich.
* **VM-Skalierungsgruppe:** Eine [VM-Skalierungsgruppe][vm-scaleset] ist ein Satz von identischen virtuellen Computern, die für das Hosten einer Workload verwendet werden. Skalierungsgruppen ermöglichen das horizontale Hoch- oder Herunterskalieren der Anzahl der VMs – und zwar sowohl manuell als auch auf vordefinierten Regeln basierend.
* **Verfügbarkeitsgruppe**. Die [Verfügbarkeitsgruppe][availability set] enthält die virtuellen Computer und berechtigt die virtuellen Computer damit für eine höhere [Vereinbarung zum Servicelevel (SLA)][vm-sla]. Damit eine höhere SLA angewendet werden kann, muss die Verfügbarkeitsgruppe mindestens zwei virtuelle Computer enthalten. Verfügbarkeitsgruppen sind in Skalierungsgruppen implizit. Wenn Sie virtuelle Computer außerhalb einer Skalierungsgruppe erstellen, müssen Sie die Verfügbarkeitsgruppe separat erstellen.
* **Verwaltete Datenträger:** Azure Managed Disks verwaltet die VHD-Dateien (virtuelle Festplatte) für die VM-Datenträger. 
* **Speicher:** Erstellen Sie ein Azure Storage-Konto zum Speichern der Diagnoseprotokolle für die virtuellen Computer.

## <a name="recommendations"></a>Recommendations

Ihre Anforderungen können von der hier beschriebenen Architektur abweichen. Verwenden Sie diese Empfehlungen als Startpunkt. 

### <a name="availability-and-scalability-recommendations"></a>Empfehlungen für Verfügbarkeit und Skalierbarkeit

Eine Option für mehr Verfügbarkeit und Skalierbarkeit stellt die Verwendung einer [VM-Skalierungsgruppe][vmss] dar. VM-Skalierungsgruppen ermöglichen die Bereitstellung und Verwaltung einer Gruppe von identischen VMs. Skalierungsgruppen unterstützen die automatische Skalierung basierend auf Leistungsmetriken. Mit zunehmender Last auf den virtuellen Computern werden dem Lastenausgleich automatisch zusätzliche virtuelle Computer hinzugefügt. Erwägen Sie die Verwendung von Skalierungsgruppen, wenn Sie VMs schnell horizontal hochskalieren müssen oder eine automatische Skalierung benötigen.

Standardmäßig verwenden Skalierungsgruppen eine „Überbereitstellung“. Das bedeutet, dass die Skalierungsgruppe anfänglich mehr virtuelle Computer bereitstellt, als Sie anfordern, und diese zusätzlichen virtuellen Computer später löscht. Dies verbessert die allgemeine Erfolgsrate bei der Bereitstellung von VMs. Wenn Sie [Managed Disks](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks) nicht verwenden, werden bei aktivierter Überbereitstellung nicht mehr als 20 virtuelle Computer pro Speicherkonto bzw. bei deaktivierter Überbereitstellung nicht mehr als 40 VMs empfohlen.

Es gibt zwei grundlegende Methoden für das Konfigurieren von virtuellen Computern in einer Skalierungsgruppe:

- Verwenden Sie die Erweiterungen, um den virtuellen Computer zu konfigurieren, nachdem er bereitgestellt wurde. Bei diesem Ansatz dauert das Starten neuer VM-Instanzen länger als bei virtuellen Computern ohne Erweiterungen.

- Stellen Sie einen [verwalteten Datenträger](/azure/storage/storage-managed-disks-overview) mit einem benutzerdefinierten Datenträgerimage bereit. Diese Option kann möglicherweise schneller bereitgestellt werden. Allerdings müssen Sie das Abbild auf dem neuesten Stand halten.

Weitere Überlegungen finden Sie unter [Überlegungen zum Entwurf von Skalierungsgruppen][vmss-design].

> [!TIP]
> Wenn Sie eine Lösung für die automatische Skalierung verwenden, testen Sie diese im Voraus mit Workloads auf Produktionsebene.

Wenn Sie keine Skalierungsgruppe verwenden, sollten Sie zumindest eine Verfügbarkeitsgruppe verwenden. Erstellen Sie zur Unterstützung der [Verfügbarkeits-SLA für Azure-VMs][vm-sla] mindestens zwei VMs in der Verfügbarkeitsgruppe. Azure Load Balancer erfordert auch, dass VMs mit Lastenausgleich derselben Verfügbarkeitsgruppe angehören.

Jedes Azure-Abonnement verfügt über Standardeinschränkungen, zu denen auch eine maximale Anzahl von virtuellen Computern pro Region gehört. Sie können den Grenzwert erhöhen, indem Sie eine Supportanfrage einreichen. Weitere Informationen finden Sie unter [Grenzwerte für Azure-Abonnements, -Dienste und -Kontingente sowie allgemeine Beschränkungen][subscription-limits].

### <a name="network-recommendations"></a>Netzwerkempfehlungen

Platzieren Sie die virtuellen Computer im gleichen Subnetz. Machen Sie die virtuellen Computer nicht direkt über das Internet verfügbar, sondern weisen Sie jedem virtuellen Computer stattdessen eine private IP-Adresse zu. Clients verwenden für Verbindungen die öffentliche IP-Adresse des Lastenausgleichs.

Wenn Sie sich auf virtuellen Computern im Lastenausgleich anmelden möchten, erwägen Sie das Hinzufügen eines einzelnen virtuellen Computers als geschützten Host/Jumpbox mit einer öffentlichen IP-Adresse, auf dem Sie sich anmelden können. Melden Sie sich dann auf den virtuellen Computern im Lastenausgleich von der Jumpbox aus an. Alternativ können Sie im Lastenausgleich eingehende NAT-Regeln für denselben Zweck konfigurieren. Eine Jumpbox stellt jedoch eine bessere Lösung dar, wenn Sie n-schichtige Workloads oder mehrere Workloads hosten.

### <a name="load-balancer-recommendations"></a>Empfehlungen für den Lastenausgleich

Fügen Sie alle virtuellen Computer in der Verfügbarkeitsgruppe dem Back-End-Adresspool des Lastenausgleichs hinzu.

Definieren Sie Lastenausgleichsregeln, um Netzwerkdatenverkehr an die virtuellen Computer weiterzuleiten. Um beispielsweise HTTP-Datenverkehr zu aktivieren, erstellen Sie eine Regel, die Port 80 aus der Front-End-Konfiguration Port 80 im Back-End-Adresspool zuordnet. Wenn ein Client eine HTTP-Anforderung an Port 80 sendet, wählt der Lastenausgleich mithilfe eines [Hashalgorithmus][load balancer hashing], der die Quell-IP-Adresse enthält, eine Back-End-IP-Adresse aus. Auf diese Weise werden die Clientanforderungen auf die virtuellen Computer verteilt.

Verwenden Sie zum Weiterleiten von Datenverkehr an einen bestimmten virtuellen Computer NAT-Regeln. Erstellen Sie z.B. zum Aktivieren von RDP auf den virtuellen Computern eine separate NAT-Regel für jeden virtuellen Computer. Jede Regel sollte Port 3389, dem Standardport für RDP, eine eigene Portnummer zuordnen. Verwenden Sie z.B. Port 50001 für „VM1“, Port 50002 für „VM2“ usw. Weisen Sie die NAT-Regeln den Netzwerkkarten der virtuellen Computer zu.

### <a name="storage-account-recommendations"></a>Empfehlungen für Speicherkonten

Erstellen Sie separate Azure Storage-Konten für jeden virtuellen Computer, um die virtuellen Festplatten (VHDs) zu speichern und das Überschreiten der [IOPS-Grenzwerte][vm-disk-limits] (Input/Output Operations Per Second) für Speicherkonten zu vermeiden.

Wir empfehlen die Verwendung von [verwalteten Datenträgern](/azure/storage/storage-managed-disks-overview) mit [Storage Premium][premium]. Bei verwalteten Datenträgern ist kein Speicherkonto erforderlich. Sie geben einfach die Größe und den Typ des Datenträgers an, und dieser wird mit Hochverfügbarkeit bereitgestellt.

Erstellen Sie ein Speicherkonto für die Diagnoseprotokolle. Dieses Speicherkonto kann von allen virtuellen Computer gemeinsam verwendet werden. Dabei kann es sich um ein nicht verwaltetes Speicherkonto mit Standarddatenträgern handeln.

## <a name="availability-considerations"></a>Überlegungen zur Verfügbarkeit

Die Verfügbarkeitsgruppe macht Ihre Anwendung gegenüber geplanten und ungeplanten Wartungsereignissen stabiler.

* *Geplante Wartungen* treten auf, wenn Microsoft die zugrunde liegende Plattform aktualisiert. Dabei müssen virtuelle Computer in einigen Fällen neu gestartet werden. Azure stellt sicher, dass die virtuellen Computer in einer Verfügbarkeitsgruppe nicht alle gleichzeitig neu gestartet werden. Mindestens eine VM wird weiter ausgeführt, während die anderen neu gestartet werden.
* *Ungeplante Wartungen* treten bei einem Hardwarefehler auf. Azure stellt sicher, dass die virtuellen Computer in einer Verfügbarkeitsgruppe über mehrere Serverracks bereitgestellt wurden. Dies trägt dazu bei, die Auswirkungen von Hardwarefehlern, Netzwerkausfällen, Unterbrechungen der Stromversorgung usw. zu reduzieren.

Weitere Informationen finden Sie unter [Verwalten der Verfügbarkeit virtueller Computer][availability set]. Das folgende Video enthält eine gute Übersicht über Verfügbarkeitsgruppen: [How Do I Configure an Availability Set to Scale VMs][availability set ch9] (Wie konfiguriere ich eine Verfügbarkeitsgruppe zum Skalieren virtueller Computer?).

> [!WARNING]
> Sie müssen die Verfügbarkeitsgruppe unbedingt bei der Bereitstellung der VM konfigurieren. Derzeit gibt es keine Möglichkeit, einer Verfügbarkeitsgruppe nach dem Bereitstellen des virtuellen Computers eine Resource Manager-VM hinzuzufügen.

Der Lastenausgleich verwendet [Integritätstests] zur Überwachung der Verfügbarkeit von VM-Instanzen. Wenn eine Instanz bei einem Test nicht innerhalb eines Timeoutzeitraums erreicht werden kann, beendet der Lastenausgleich das Senden von Datenverkehr an diesen virtuellen Computer. Der Lastenausgleich führt aber weiterhin Tests durch, und wenn der virtuelle Computer wieder erreichbar ist, sendet der Lastenausgleich auch wieder Datenverkehr an diese VM.

Empfehlungen für Integritätstests durch den Lastenausgleich:

* Die Tests können für HTTP oder TCP durchgeführt werden. Wenn auf Ihren virtuellen Computern ein HTTP-Server ausgeführt wird, erstellen Sie einen HTTP-Test. Erstellen Sie andernfalls einen TCP-Test.
* Geben Sie für einen HTTP-Test den Pfad zu einem HTTP-Endpunkt an. Der Test überprüft, ob von diesem Pfad eine HTTP-200-Antwort empfangen wird. Dies kann den Stammpfad („/“) oder ein Endpunkt zur Integritätsüberwachung sein, der benutzerdefinierte Logik zum Überprüfen der Integrität der Anwendung implementiert. Der Endpunkt muss anonyme HTTP-Anforderungen zulassen.
* Der Test wird von einer [bekannten][health-probe-ip] IP-Adresse (168.63.129.16) gesendet. Stellen Sie sicher, dass Datenverkehr zu oder von dieser IP-Adresse in keiner Firewall-Richtlinie und keinen NSG-Regeln (Netzwerksicherheitsgruppe) blockiert wird.
* Verwenden Sie die [Protokolle der Integritätstests][health probe log], um den Status der Integritätstests anzuzeigen. Aktivieren Sie die Protokollierung für jeden Lastenausgleich im Azure-Portal. Die Protokolle werden in Azure Blob Storage geschrieben. Die Protokolle zeigen, wie viele virtuelle Computer am Back-End aufgrund fehlerhafter Testantworten keinen Netzwerkdatenverkehr empfangen.

## <a name="manageability-considerations"></a>Überlegungen zur Verwaltbarkeit

Bei mehreren virtuellen Computern ist es wichtig, die Prozesse zu automatisieren, damit sie zuverlässig und reproduzierbar sind. Sie können [Azure Automation][azure-automation] verwenden, um die Bereitstellung, Betriebssystempatches und andere Aufgaben zu automatisieren.

## <a name="security-considerations"></a>Sicherheitshinweise

Virtuelle Netzwerke stellen in Azure eine Isolationsbegrenzung für Datenverkehr dar. Virtuelle Computer in einem VNET können nicht direkt mit virtuellen Computern in einem anderen VNET kommunizieren. VMs im selben VNET können miteinander kommunizieren, sofern Sie den Datenverkehr nicht durch die Erstellung von [Netzwerksicherheitsgruppen][nsg] (NSGs) beschränken. Weitere Informationen finden Sie unter [Microsoft Cloud Services und Netzwerksicherheit][network-security].

Für eingehenden Internetdatenverkehr definieren die Lastenausgleichsregeln, welcher Datenverkehr an das Back-End weitergeleitet wird. Allerdings unterstützen Lastenausgleichsregeln keine IP-Sicherheitslisten. Wenn Sie also einer Sicherheitsliste bestimmte öffentliche IP-Adressen hinzufügen möchten, fügen Sie dem Subnetz eine NSG hinzu.

## <a name="deploy-the-solution"></a>Bereitstellen der Lösung

Eine Bereitstellung für diese Architektur ist auf [GitHub][github-folder] verfügbar. Sie enthält ein VNET, eine NSG und drei virtuelle Computer in einer Skalierungsgruppe hinter einem Lastenausgleich, der unten beschrieben wird:

  * Ein virtuelles Netzwerk mit einzelnem Subnetz namens **web** hostet die virtuellen Computer.
  * Eine VM-Skalierungsgruppe enthält die virtuellen Computer, auf denen die neueste Version von Ubuntu (16.04.3 LTS) ausgeführt wird. Die automatische Skalierung ist aktiviert.
  * Vor der-Skalierungsgruppe befindet sich ein Lastenausgleich.
  * Eine NSG mit Regeln für den eingehenden Datenverkehr lässt HTTP-Datenverkehr an die VM-Skalierungsgruppe zu.

### <a name="prerequisites"></a>Voraussetzungen

Bevor Sie die Referenzarchitektur in Ihrem eigenen Abonnement bereitstellen können, müssen Sie die folgenden Schritte ausführen.

1. Klonen oder Forken Sie das GitHub-Repository [AzureCAT-Referenzarchitekturen][ref-arch-repo], oder laden Sie die zugehörige ZIP-Datei herunter.

2. Vergewissern Sie sich, dass Azure CLI 2.0 auf Ihrem Computer installiert ist. Um die Befehlszeilenschnittstelle zu installieren, befolgen Sie die Anweisungen unter [Installieren von Azure CLI 2.0][azure-cli-2].

3. Installieren Sie das npm-Paket mit den [Azure Bausteinen][azbb].

4. Melden Sie sich über eine Eingabeaufforderung, eine bash-Eingabeaufforderung oder die PowerShell-Eingabeaufforderung bei Ihrem Azure-Konto an. Verwenden Sie dazu die unten aufgeführten Befehle, und befolgen Sie die Anweisungen.

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>Bereitstellen der Lösung mit azbb

Zum Bereitstellen der einzelnen VM-Beispielworkload gehen Sie folgendermaßen vor:

1. Navigieren Sie zum Ordner `virtual-machines\multi-vm\parameters\linux` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.

2. Öffnen Sie die Datei `multi-vm-v2.json`, geben Sie einen Benutzernamen und den SSH-Schlüssel wie unten dargestellt zwischen den Anführungszeichen an, und speichern Sie die Datei.

  ```bash
  "adminUsername": "",
  "sshPublicKey": "",
  ```

3. Führen Sie `azbb` aus, um die virtuellen Computer wie unten dargestellt bereitzustellen.

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
[Runbook Gallery]: /azure/automation/automation-runbook-gallery#runbooks-in-runbook-gallery
[subscription-limits]: /azure/azure-subscription-service-limits
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[vmss-quickstart]: https://azure.microsoft.com/documentation/templates/?term=scale+set
[VM-sizes]: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
[0]: ./images/multi-vm-diagram.png "Architektur einer Lösung mit mehreren virtuellen Computern in Azure bestehend aus einer Verfügbarkeitsgruppe mit zwei virtuellen Computern und einem Lastenausgleich"
[azure-cli-2]: /azure/install-azure-cli?view=azure-cli-latest
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Template-Building-Blocks-Version-2-(Linux)
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm