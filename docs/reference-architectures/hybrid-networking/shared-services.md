---
title: Implementieren einer Hub-Spoke-Netzwerktopologie mit gemeinsamen Diensten in Azure
description: Es wird beschrieben, wie Sie eine Hub-Spoke-Netzwerktopologie mit gemeinsamen Diensten in Azure implementieren.
author: telmosampaio
ms.date: 02/25/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: c0fb1d1ddd7c70ed914d58e7c73b10475b91aedf
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/08/2018
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a>Implementieren einer Hub-Spoke-Netzwerktopologie mit gemeinsamen Diensten in Azure

Diese Referenzarchitektur baut auf der [Hub-Spoke][guidance-hub-spoke]-Referenzarchitektur auf, um gemeinsame Dienste in den Hub einzubinden, die von allen Spokes genutzt werden können. Als ersten Schritt zur Migration eines Rechenzentrums zur Cloud und Erstellung eines [virtuellen Rechenzentrums] müssen Sie zunächst die Dienste für Identität und Sicherheit freigeben. Anhand dieser Referenzarchitektur wird veranschaulicht, wie Sie Ihre Active Directory-Dienste aus Ihrem lokalen Rechenzentrum auf Azure erweitern und ein virtuelles Netzwerkgerät (Network Virtual Appliance, NVA) hinzufügen, das in einer Hub-Spoke-Topologie als Firewall fungieren kann.  [**Stellen Sie diese Lösung bereit**](#deploy-the-solution).

![[0]][0]

*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*

Diese Topologie bietet unter anderem folgende Vorteile:

* **Kosteneinsparungen** durch die Zentralisierung von Diensten, die von mehreren Workloads (z.B. Network Virtual Appliances, kurz NVAs, und DNS-Servern) an einem einzigen Standort gemeinsam genutzt werden können
* **Umgehung von Abonnementbeschränkungen** durch die Herstellung von Peeringverbindungen zwischen VNETs verschiedener Abonnements und dem zentralen Hub
* **Trennung der Belange** zwischen zentralen IT-Vorgängen (SecOps, InfraOps) und Workloads (DevOps)

Typische Einsatzmöglichkeiten für diese Architektur sind Folgende:

* Workloads, die in verschiedenen Umgebungen wie Entwicklungs-, Test- und Produktionsumgebungen eingesetzt werden und gemeinsame Dienste wie DNS, IDS, NTP oder AD DS erfordern Gemeinsame Dienste werden im Hub-VNET platziert, während die einzelnen Umgebungen in einem Spoke bereitgestellt werden, um die Isolation beizubehalten.
* Workloads, bei denen keine Konnektivität untereinander bestehen muss, die jedoch Zugriff auf gemeinsame Dienste erfordern
* Unternehmen, die auf eine zentrale Steuerung von Sicherheitsmechanismen (z.B. eine Firewall im Hub als DMZ) und eine getrennte Verwaltung von Workloads in den einzelnen Spokes angewiesen sind

## <a name="architecture"></a>Architecture

Die Architektur umfasst die folgenden Komponenten.

* **Lokales Netzwerk**. Ein in einer Organisation betriebenes privates lokales Netzwerk.

* **VPN-Gerät**: Ein Gerät oder ein Dienst, der externe Konnektivität mit dem lokalen Netzwerk bereitstellt. Bei dem VPN-Gerät kann es sich um ein Hardwaregerät oder eine Softwarelösung wie den Routing- und RAS-Dienst (RRAS) unter Windows Server 2012 handeln. Eine Liste der unterstützten VPN-Appliances und Informationen zur Konfiguration ausgewählter VPN-Geräte für die Verbindung mit Azure finden Sie unter [Informationen zu VPN-Geräten und IPsec-/IKE-Parametern für VPN-Gatewayverbindungen zwischen Standorten][vpn-appliance].

* **VPN-Gateway für ein virtuelles Netzwerk oder ExpressRoute-Gateway**: Über das Gateway für virtuelle Netzwerke kann das VNET zur Konnektivität mit Ihrem lokalen Netzwerk eine Verbindung mit dem VPN-Gerät oder eine ExpressRoute-Verbindung herstellen. Weitere Informationen finden Sie unter [Verbinden eines lokalen Netzwerks mit einem virtuellen Microsoft Azure-Netzwerk][connect-to-an-Azure-vnet].

> [!NOTE]
> In den Bereitstellungsskripts für diese Referenzarchitektur werden ein VPN-Gateway für die Konnektivität und ein VNET in Azure verwendet, um Ihr lokales Netzwerk zu simulieren.

* **Hub-VNET**: Das Azure-VNET, das als Hub in der Hub-Spoke-Topologie verwendet wird. Der Hub ist der zentrale Konnektivitätspunkt für Ihr lokales Netzwerk, mit dem Sie Dienste hosten, die von den verschiedenen in den Spoke-VNETs gehosteten Workloads genutzt werden können.

* **Gatewaysubnetz**: Die Gateways für die virtuellen Netzwerke befinden sich im selben Subnetz.

* **Subnetz für gemeinsame Dienste**: Ein Subnetz im Hub-VNET, das für das Hosten von Diensten verwendet wird, die von allen Spokes gemeinsam genutzt werden können (z.B. DNS oder AD DS).

* **DMZ-Subnetz**: Ein Subnetz im Hub-VNET zum Hosten von NVAs, die als Sicherheitseinrichtungen, z.B. Firewalls, dienen können.

* **Spoke-VNETs**: Ein oder mehrere Azure-VNETs, die als Spokes in der Hub-Spoke-Topologie verwendet werden. Spokes können verwendet werden, um Workloads in ihren eigenen VNETs, die getrennt von anderen Spokes verwaltet werden, zu isolieren. Jede Workload kann mehrere Schichten umfassen, wobei mehrere Subnetze über Azure Load Balancer verbunden sind. Weitere Informationen zur Anwendungsinfrastruktur finden Sie unter [Ausführen von Windows-VM-Workloads][windows-vm-ra] und [Ausführen von Linux-VM-Workloads][linux-vm-ra].

* **VNET-Peering**: Über eine [Peeringverbindung][vnet-peering] können zwei VNETs in derselben Azure-Region miteinander verbunden werden. Peeringverbindungen sind nicht-transitive Verbindungen zwischen VNETs mit niedrigen Latenzen. Sobald eine Peeringverbindung hergestellt wurde, tauschen die VNETs ohne Einsatz eines Routers Datenverkehr über den Azure-Backbone aus. In einer Hub-Spoke-Netzwerktopologie wird durch VNET-Peering eine Verbindung zwischen dem Hub und den einzelnen Spokes hergestellt.

> [!NOTE]
> In diesem Artikel werden ausschließlich [Resource Manager](/azure/azure-resource-manager/resource-group-overview)-Bereitstellungen behandelt, Sie können jedoch auch eine Verbindung zwischen einem klassischen VNET und einem Resource Manager-VNET im selben Abonnement herstellen. Auf diese Weise können Ihre Spokes klassische Bereitstellungen hosten und trotzdem von gemeinsamen Diensten im Hub profitieren.

## <a name="recommendations"></a>Empfehlungen

Alle Empfehlungen für die [Hub-Spoke][guidance-hub-spoke]-Referenzarchitektur gelten auch für die Referenzarchitektur für gemeinsame Dienste. 

Außerdem gelten die folgenden Empfehlungen für die meisten Szenarien mit gemeinsamen Diensten. Sofern Sie keine besonderen Anforderungen haben, die Vorrang haben, sollten Sie diese Empfehlungen befolgen.

### <a name="identity"></a>Identity

Die meisten Organisationen großer Unternehmen verfügen im lokalen Rechenzentrum über eine AD DS-Umgebung (Active Directory Domain Services). Um die Verwaltung von Assets zu ermöglichen, die aus Ihrem lokalen Netzwerk nach Azure verschoben werden und von AD DS abhängig sind, ist es ratsam, AD DS-Domänencontroller in Azure zu hosten.

Falls Sie Gruppenrichtlinienobjekte nutzen, die Sie für Azure und Ihre lokale Umgebung separat steuern möchten, sollten Sie für jede Azure-Region einen anderen AD-Standort verwenden. Ordnen Sie Ihre Domänencontroller in einem zentralen VNET (Hub) an, auf das abhängige Workloads zugreifen können.

### <a name="security"></a>Sicherheit

Wenn Sie Workloads aus Ihrer lokalen Umgebung nach Azure verschieben, müssen einige dieser Workloads auf VMs gehostet werden. Aus Gründen der Konformität müssen Sie für Datenverkehr, der diese Workloads durchläuft, unter Umständen Einschränkungen erzwingen. 

Sie können virtuelle Netzwerkgeräte in Azure verwenden, um unterschiedliche Arten von Sicherheits- und Leistungsdiensten zu hosten. Wenn Sie derzeit mit einem bestimmten Satz von lokalen Geräten vertraut sind, ist es ratsam, jeweils die gleichen virtualisierten Geräte in Azure zu verwenden (falls zutreffend).

> [!NOTE]
> Für die Bereitstellungsskripts dieser Referenzarchitektur wird eine Ubuntu-VM mit aktivierter IP-Weiterleitung verwendet, um ein virtuelles Netzwerkgerät zu imitieren.

## <a name="considerations"></a>Überlegungen

### <a name="overcoming-vnet-peering-limits"></a>Umgehung von VNET-Peeringbeschränkungen

Stellen Sie sicher, dass Sie die [Begrenzung der Anzahl von VNET-Peerings pro VNET][vnet-peering-limit] in Azure einhalten. Falls Sie mehr Spokes als die maximal zulässige Anzahl benötigen, sollten Sie eventuell eine Hub-Spoke-Hub-Spoke-Topologie erstellen, bei der die erste Ebene von Spokes auch als Hub fungiert. Im folgenden Diagramm wird diese Vorgehensweise veranschaulicht.

![[3]][3]

Berücksichtigen Sie auch die Dienste, die im Hub gemeinsam genutzt werden, um sicherzustellen, dass sich der Hub für eine größere Anzahl von Spokes skalieren lässt. Wenn Ihr Hub beispielsweise Firewalldienste bereitstellt, sollten Sie beim Hinzufügen mehrerer Spokes die Bandbreitenbeschränkungen Ihrer Firewalllösung berücksichtigen. Es wird empfohlen, einige dieser gemeinsamen Dienste auf eine zweite Hubebene zu verlagern.

## <a name="deploy-the-solution"></a>Bereitstellen der Lösung

Eine Bereitstellung für diese Architektur ist auf [GitHub][ref-arch-repo] verfügbar. Bei dieser werden zum Testen der Konnektivität Ubuntu-VMs in jedem VNET verwendet. Es gibt keine tatsächlichen Dienste, die im Subnetz für **gemeinsame Dienste** im **Hub-VNET** gehostet werden.

### <a name="prerequisites"></a>Voraussetzungen

Bevor Sie die Referenzarchitektur in Ihrem eigenen Abonnement bereitstellen können, müssen Sie die folgenden Schritte ausführen.

1. Klonen oder Forken Sie das GitHub-Repository [AzureCAT-Referenzarchitekturen][ref-arch-repo], oder laden Sie die zugehörige ZIP-Datei herunter.

2. Vergewissern Sie sich, dass Azure CLI 2.0 auf Ihrem Computer installiert ist. Anweisungen zur CLI-Installation finden Sie unter [Installieren von Azure-CLI 2.0][azure-cli-2].

3. Installieren Sie das npm-Paket mit den [Azure Bausteinen][azbb].

4. Melden Sie sich über eine Eingabeaufforderung, eine Bash-Eingabeaufforderung oder die PowerShell-Eingabeaufforderung an Ihrem Azure-Konto an. Verwenden Sie hierzu den unten aufgeführten Befehl, und befolgen Sie die Anweisungen.

  ```bash
  az login
  ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a>Bereitstellen des simulierten lokalen Rechenzentrums mit azbb

Führen Sie die folgenden Schritte aus, um das simulierte lokale Rechenzentrum als Azure-VNET bereitzustellen:

1. Navigieren Sie zum Ordner `hybrid-networking\shared-services-stack\` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.

2. Öffnen Sie die Datei `onprem.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 45 und 46 ein, und speichern Sie die Datei.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Führen Sie `azbb` aus, um die simulierte lokale Umgebung wie unten gezeigt bereitzustellen.

  ```bash
  azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
  ```
  > [!NOTE]
  > Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `onprem-vnet-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.

4. Warten Sie, bis die Bereitstellung abgeschlossen ist. Bei dieser Bereitstellung werden ein virtuelles Netzwerk, ein virtueller Windows-Computer und ein VPN-Gateway erstellt. Die Erstellung des VPN-Gateways kann mehr als 40 Minuten in Anspruch nehmen.

### <a name="azure-hub-vnet"></a>Azure-Hub-VNET

Führen Sie die folgenden Schritte durch, um das Hub-VNET bereitzustellen und eine Verbindung mit dem oben erstellten simulierten lokalen VNET herzustellen.

1. Öffnen Sie die Datei `hub-vnet.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 50 und 51 ein, und speichern Sie die Datei.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. Geben Sie in Zeile 52 unter `osType` entweder `Windows` oder `Linux` ein, um Windows Server 2016 Datacenter bzw. Ubuntu 16.04 als Betriebssystem für die Jumpbox zu installieren.

3. Geben Sie einen gemeinsam verwendeten Schlüssel wie unten dargestellt zwischen den Anführungszeichen in Zeile 83 ein, und speichern Sie die Datei.

  ```bash
  "sharedKey": "",
  ```

4. Führen Sie `azbb` aus, um die simulierte lokale Umgebung wie unten gezeigt bereitzustellen.

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
  ```
  > [!NOTE]
  > Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `hub-vnet-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.

5. Warten Sie, bis die Bereitstellung abgeschlossen ist. Bei dieser Bereitstellung werden ein virtuelles Netzwerk, ein virtueller Computer, ein VPN-Gateway und eine Verbindung mit dem im vorherigen Abschnitt erstellten Gateway erstellt. Die Erstellung des VPN-Gateways kann mehr als 40 Minuten in Anspruch nehmen.

### <a name="adds-in-azure"></a>AD DS in Azure

Führen Sie die folgenden Schritte aus, um die AD DS-Domänencontroller in Azure bereitzustellen.

1. Öffnen Sie die Datei `hub-adds.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 14 und 15 ein, und speichern Sie die Datei.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. Führen Sie `azbb` aus, um die AD DS-Domänencontroller wie unten gezeigt bereitzustellen.

  ```bash
  azbb -s <subscription_id> -g hub-adds-rg - l <location> -p hub-adds.json --deploy
  ```
  
  > [!NOTE]
  > Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `hub-adds-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.

  > [!NOTE]
  > Dieser Teil der Bereitstellung kann mehrere Minuten dauern, da die beiden VMs in die Domäne eingebunden werden müssen, die im simulierten lokalen Rechenzentrum gehostet wird, und anschließend AD DS darauf installiert werden muss.

### <a name="nva"></a>NVA

Führen Sie die folgenden Schritte aus, um eine NVA im Subnetz `dmz` bereitzustellen:

1. Öffnen Sie die Datei `hub-nva.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 13 und 14 ein, und speichern Sie die Datei.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```
2. Führen Sie `azbb` aus, um die NVA-VM und die benutzerdefinierten Routen bereitzustellen.

  ```bash
  azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
  ```
  > [!NOTE]
  > Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `hub-nva-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.

### <a name="azure-spoke-vnets"></a>Azure-Spoke-VNETs

Führen Sie die folgenden Schritte aus, um die Spoke-VNETs bereitzustellen.

1. Öffnen Sie die Datei `spoke1.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 52 und 53 ein, und speichern Sie die Datei.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. Geben Sie in Zeile 54 unter `osType` entweder `Windows` oder `Linux` ein, um Windows Server 2016 Datacenter bzw. Ubuntu 16.04 als Betriebssystem für die Jumpbox zu installieren.

3. Führen Sie `azbb` aus, um die erste Spoke-VNET-Umgebung wie unten gezeigt bereitzustellen.

  ```bash
  azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
  ```
  
  > [!NOTE]
  > Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `spoke1-vnet-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.

3. Wiederholen Sie den obigen Schritt 1 für die Datei `spoke2.json`.

4. Führen Sie `azbb` aus, um die zweite Spoke-VNET-Umgebung wie unten gezeigt bereitzustellen.

  ```bash
  azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
  ```
  > [!NOTE]
  > Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `spoke2-vnet-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a>Herstellen einer Peeringverbindung zwischen Azure-Hub-VNETs und Spoke-VNETs

Führen Sie die folgenden Schritte aus, um eine Peeringverbindung vom Hub-VNET mit den Spoke-VNETs zu erstellen.

1. Öffnen Sie die Datei `hub-vnet-peering.json`, und stellen Sie sicher, dass der Ressourcengruppenname und der Name des virtuellen Netzwerks für die einzelnen VNET-Peerings ab Zeile 29 korrekt sind.

2. Führen Sie `azbb` aus, um die erste Spoke-VNET-Umgebung wie unten gezeigt bereitzustellen.

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
  ```

  > [!NOTE]
  > Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `hub-vnet-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[guidance-hub-spoke]: ./hub-spoke.md
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[virtuellen Rechenzentrums]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "Topologie mit gemeinsamen Diensten in Azure"
[3]: ./images/hub-spokehub-spoke.svg "Hub-Spoke-Hub-Spoke-Topologie in Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
