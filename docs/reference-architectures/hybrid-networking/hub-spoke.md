---
title: Implementieren einer Hub-Spoke-Netzwerktopologie in Azure
description: Vorgehensweise zum Implementieren einer Hub-Spoke-Netzwerktopologie in Azure
author: telmosampaio
ms.date: 04/09/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 3b19526a9ed77c1605325a9eec101ffbee7c8401
ms.sourcegitcommit: 3846a0ab2b2b2552202a3c9c21af0097a145ffc6
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/29/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a>Implementieren einer Hub-Spoke-Netzwerktopologie in Azure

Diese Referenzarchitektur zeigt, wie eine Hub-Spoke-Topologie in Azure implementiert wird. Bei einem *Hub* handelt es sich um ein virtuelles Netzwerk (VNET) in Azure, das als zentraler Konnektivitätspunkt für Ihr lokales Netzwerk fungiert. *Spokes* sind VNETs, die eine Peeringverbindung mit dem Hub herstellen und zur Isolierung von Workloads verwendet werden können. Der Datenverkehr wird über eine ExpressRoute- oder VPN-Gatewayverbindung zwischen dem lokalen Rechenzentrum und dem Hub weitergeleitet.  [**Stellen Sie diese Lösung bereit**](#deploy-the-solution).

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

* **Gatewaysubnetz**: Die Gateways für virtuelle Netzwerke befinden sich in demselben Subnetz.

* **Spoke-VNETs**: Ein oder mehrere Azure-VNETs, die als Spokes in der Hub-Spoke-Topologie verwendet werden. Spokes können verwendet werden, um Workloads in ihren eigenen VNETs, die getrennt von anderen Spokes verwaltet werden, zu isolieren. Jede Workload kann mehrere Schichten umfassen, wobei mehrere Subnetze über Azure Load Balancer verbunden sind. Weitere Informationen zur Anwendungsinfrastruktur finden Sie unter [Ausführen von Windows-VM-Workloads][windows-vm-ra] und [Ausführen von Linux-VM-Workloads][linux-vm-ra].

* **VNET-Peering**: Über eine [Peeringverbindung][vnet-peering] können zwei VNETs in derselben Azure-Region miteinander verbunden werden. Peeringverbindungen sind nicht-transitive Verbindungen zwischen VNETs mit niedrigen Latenzen. Sobald eine Peeringverbindung hergestellt wurde, tauschen die VNETs ohne Einsatz eines Routers Datenverkehr über den Azure-Backbone aus. In einer Hub-Spoke-Netzwerktopologie wird durch VNET-Peering eine Verbindung zwischen dem Hub und den einzelnen Spokes hergestellt.

> [!NOTE]
> In diesem Artikel werden ausschließlich [Resource Manager](/azure/azure-resource-manager/resource-group-overview)-Bereitstellungen behandelt, Sie können jedoch auch eine Verbindung zwischen einem klassischen VNET und einem Resource Manager-VNET im selben Abonnement herstellen. Auf diese Weise können Ihre Spokes klassische Bereitstellungen hosten und trotzdem von gemeinsamen Diensten im Hub profitieren.

## <a name="recommendations"></a>Empfehlungen

Die folgenden Empfehlungen gelten für die meisten Szenarios. Sofern Sie keine besonderen Anforderungen haben, die Vorrang haben, sollten Sie diese Empfehlungen befolgen.

### <a name="resource-groups"></a>Ressourcengruppen

Das Hub-VNET und die einzelnen Spoke-VNETs können in verschiedenen Ressourcengruppen und sogar in verschiedenen Abonnements implementiert werden, sofern sie demselben Azure Active Directory-Mandanten (Azure AD) in der gleichen Azure-Region angehören. Dies ermöglicht nicht nur eine dezentralisierte Verwaltung der einzelnen Workloads, sondern auch die Verwaltung gemeinsamer Dienste im Hub-VNET.

### <a name="vnet-and-gatewaysubnet"></a>VNET und GatewaySubnet

Erstellen Sie ein Subnetz mit dem Namen *GatewaySubnet* mit dem Adressbereich /27. Dieses Subnetz ist für das Gateway für virtuelle Netzwerke erforderlich. Durch Zuweisung von 32 Adressen zu diesem Subnetz wird eine künftige Überschreitung von Beschränkungen der Gatewaygröße verhindert.

Weitere Informationen zum Einrichten des Gateways finden Sie abhängig von Ihrem Verbindungstyp in den folgenden Referenzarchitekturen:

- [Hybrides Netzwerk mit ExpressRoute][guidance-expressroute]
- [Hybrides Netzwerk mit einem VPN-Gateway][guidance-vpn]

Um eine höhere Verfügbarkeit zu erzielen, können Sie ExpressRoute mit einem VPN für Failoverzwecke kombinieren. Weitere Informationen finden Sie unter [Verbinden eines lokalen Netzwerks mit Azure unter Verwendung von ExpressRoute mit VPN-Failover][hybrid-ha].

Eine Hub-Spoke-Topologie kann auch ohne Gateway verwendet werden, wenn keine Konnektivität mit Ihrem lokalen Netzwerk erforderlich ist. 

### <a name="vnet-peering"></a>VNet-Peering

Beim VNET-Peering wird eine nicht-transitive Beziehung zwischen zwei VNETs hergestellt. Wenn Sie eine Verbindung zwischen Spokes herstellen möchten, sollten Sie eventuell eine separate Peeringverbindung zwischen diesen Spokes herstellen.

Wenn jedoch zwischen mehreren Spokes eine Verbindung hergestellt werden muss, werden die möglichen Peeringverbindungen sehr schnell zur Neige gehen, da [die Anzahl der VNET-Peerings pro VNET begrenzt ist][vnet-peering-limit]. In diesem Szenario sollten Sie die Verwendung von benutzerdefinierten Routen (User Defined Routes, UDRs) in Erwägung ziehen, um zu erzwingen, dass der für einen Spoke vorgesehene Datenverkehr an eine NVA, die als Router im Hub-VNET fungiert, gesendet wird. Hierdurch können die Spokes miteinander verbunden werden.

Sie können Spokes auch für die Kommunikation mit Remotenetzwerken über das Gateway für das Hub-VNET konfigurieren. Damit der Gatewaydatenverkehr zwischen den Spokes und dem Hub weitergeleitet und eine Verbindung mit Remotenetzwerken hergestellt werden kann, müssen Sie folgende Schritte durchführen:

  - Konfigurieren Sie die VNET-Peeringverbindung im Hub dahingehend, dass **Gatewaytransit zugelassen** wird.
  - Konfigurieren Sie die VNET-Peeringverbindung in den einzelnen Spokes dahingehend, dass **Remotegateways verwendet werden**.
  - Konfigurieren Sie alle VNET-Peeringverbindungen dahingehend, dass **weitergeleiteter Datenverkehr zugelassen** wird.

## <a name="considerations"></a>Überlegungen

### <a name="spoke-connectivity"></a>Konnektivität zwischen Spokes

Wenn Konnektivität zwischen Spokes hergestellt werden muss, sollten Sie eine NVA für das Routing im Hub implementieren und UDRs im Spoke zur Weiterleitung des Datenverkehrs an den Hub verwenden.

![[2]][2]

In diesem Szenario müssen Sie die Peeringverbindungen dahingehend konfigurieren, dass **weitergeleiteter Verkehr zugelassen** wird.

### <a name="overcoming-vnet-peering-limits"></a>Umgehung von VNET-Peeringbeschränkungen

Stellen Sie sicher, dass Sie die [Begrenzung der Anzahl von VNET-Peerings pro VNET][vnet-peering-limit] in Azure einhalten. Falls Sie mehr Spokes als die maximal zulässige Anzahl benötigen, sollten Sie eventuell eine Hub-Spoke-Hub-Spoke-Topologie erstellen, bei der die erste Ebene von Spokes auch als Hub fungiert. Im folgenden Diagramm wird diese Vorgehensweise veranschaulicht.

![[3]][3]

Berücksichtigen Sie auch die Dienste, die im Hub gemeinsam genutzt werden, um sicherzustellen, dass sich der Hub für eine größere Anzahl von Spokes skalieren lässt. Wenn Ihr Hub beispielsweise Firewalldienste bereitstellt, sollten Sie beim Hinzufügen mehrerer Spokes die Bandbreitenbeschränkungen Ihrer Firewalllösung berücksichtigen. Es wird empfohlen, einige dieser gemeinsamen Dienste auf eine zweite Hubebene zu verlagern.

## <a name="deploy-the-solution"></a>Bereitstellen der Lösung

Eine Bereitstellung für diese Architektur ist auf [GitHub][ref-arch-repo] verfügbar. Bei dieser werden zum Testen der Konnektivität VMs in jedem VNET verwendet. Es gibt keine tatsächlichen Dienste, die im Subnetz für **gemeinsame Dienste** im **Hub-VNET** gehostet werden.

Die Bereitstellung erstellt die folgenden Ressourcengruppen in Ihrem Abonnement:

- hub-nva-rg
- hub-vnet-rg
- onprem-jb-rg
- onprem-vnet-rg
- spoke1-vnet-rg
- spoke2-vent-rg

Die Vorlagenparameterdateien beziehen sich auf diese Namen, d.h. wenn Sie sie ändern, müssen Sie die Parameterdateien entsprechend aktualisieren.

### <a name="prerequisites"></a>Voraussetzungen

1. Klonen oder Forken Sie das GitHub-Repository [Referenzarchitekturen][ref-arch-repo], oder laden Sie die entsprechende ZIP-Datei herunter.

2. Installieren Sie [Azure CLI 2.0][azure-cli-2].

3. Installieren Sie das npm-Paket mit den [Azure Bausteinen][azbb].

4. Melden Sie sich über eine Eingabeaufforderung, eine Bash-Eingabeaufforderung oder die PowerShell-Eingabeaufforderung bei Ihrem Azure-Konto an. Verwenden Sie hierzu den unten aufgeführten Befehl.

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>Bereitstellen des simulierten lokalen Rechenzentrums

Führen Sie die folgenden Schritte aus, um das simulierte lokale Rechenzentrum als Azure-VNET bereitzustellen:

1. Navigieren Sie zum `hybrid-networking/hub-spoke`-Ordner des Repositorys für Referenzarchitekturen.

2. Öffnen Sie die Datei `onprem.json` . Ersetzen Sie die Werte für `adminUsername` und `adminPassword`.

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. (Optional) Legen Sie für eine Linux-Bereitstellung `osType` auf `Linux` fest.

4. Führen Sie den folgenden Befehl aus:

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. Warten Sie, bis die Bereitstellung abgeschlossen ist. Bei dieser Bereitstellung werden ein virtuelles Netzwerk, ein virtueller Computer und ein VPN-Gateway erstellt. Das Erstellen des VPN-Gateways kann etwa 40 Minuten dauern.

### <a name="deploy-the-hub-vnet"></a>Bereitstellen des Hub-VNet

Führen Sie die folgenden Schritte aus, um das Hub-VNet bereitzustellen.

1. Öffnen Sie die Datei `hub-vnet.json` . Ersetzen Sie die Werte für `adminUsername` und `adminPassword`.

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. (Optional) Legen Sie für eine Linux-Bereitstellung `osType` auf `Linux` fest.

3. Geben Sie für `sharedKey` einen gemeinsam genutzten Schlüssel für die VPN-Verbindung ein. 

    ```bash
    "sharedKey": "",
    ```

4. Führen Sie den folgenden Befehl aus:

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. Warten Sie, bis die Bereitstellung abgeschlossen ist. Bei dieser Bereitstellung werden ein virtuelles Netzwerk, ein virtueller Computer, ein VPN-Gateway und eine Verbindung mit dem Gateway erstellt.  Das Erstellen des VPN-Gateways kann etwa 40 Minuten dauern.

### <a name="test-connectivity-with-the-hub"></a>Testen der Konnektivität mit dem Hub

Testen Sie die Konnektivität zwischen der simulierten lokalen Umgebung und dem Hub-VNET.

**Windows-Bereitstellung**

1. Suchen Sie im Azure-Portal die VM namens `jb-vm1` in der `onprem-jb-rg`-Ressourcengruppe.

2. Klicke Sie auf `Connect`, um eine Remotedesktopsitzung mit der VM zu öffnen. Verwenden Sie das Kennwort, das Sie in der `onprem.json`-Parameterdatei angegeben haben.

3. Öffnen Sie auf der VM eine PowerShell-Konsole, und verwenden Sie das `Test-NetConnection`-Cmdlet, um sicherzustellen, dass Sie eine Verbindung mit der Jumpbox-VM im Hub-VNet herstellen können.

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
Die Ausgabe sollte in etwa wie folgt aussehen:

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> Standardmäßig lassen Windows Server-VMs in Azure keine ICMP-Antworten zu. Wenn Sie `ping` zum Testen der Konnektivität nutzen möchten, müssen Sie für jede VM ICMP-Datenverkehr in der erweiterten Windows-Firewall aktivieren.

**Linux-Bereitstellung**

1. Suchen Sie im Azure-Portal die VM namens `jb-vm1` in der `onprem-jb-rg`-Ressourcengruppe.

2. Klicken Sie auf `Connect`, und kopieren Sie den im Portal angezeigten `ssh`-Befehl. 

3. Führen Sie in einer Linux-Eingabeaufforderung `ssh` zum Herstellen der Verbindung mit der simulierten lokalen Umgebung aus. Verwenden Sie das Kennwort, das Sie in der `onprem.json`-Parameterdatei angegeben haben.

4. Verwenden Sie den Befehl `ping`, um die Konnektivität mit der Jumpbox-VM im Hub-VNet zu testen:

   ```bash
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a>Bereitstellen der Spoke-VNets

Führen Sie die folgenden Schritte aus, um die Spoke-VNETs bereitzustellen.

1. Öffnen Sie die Datei `spoke1.json` . Ersetzen Sie die Werte für `adminUsername` und `adminPassword`.

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. (Optional) Legen Sie für eine Linux-Bereitstellung `osType` auf `Linux` fest.

3. Führen Sie den folgenden Befehl aus:

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. Wiederholen Sie die Schritte 1 bis 2 für die `spoke2.json`-Datei.

5. Führen Sie den folgenden Befehl aus:

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. Führen Sie den folgenden Befehl aus:

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a>Testen der Konnektivität

Testen Sie die Konnektivität zwischen der simulierten lokalen Umgebung und den Spoke-VNets.

**Windows-Bereitstellung**

1. Suchen Sie im Azure-Portal die VM namens `jb-vm1` in der `onprem-jb-rg`-Ressourcengruppe.

2. Klicke Sie auf `Connect`, um eine Remotedesktopsitzung mit der VM zu öffnen. Verwenden Sie das Kennwort, das Sie in der `onprem.json`-Parameterdatei angegeben haben.

3. Öffnen Sie auf der VM eine PowerShell-Konsole, und verwenden Sie das `Test-NetConnection`-Cmdlet, um sicherzustellen, dass Sie eine Verbindung mit der Jumpbox-VM im Hub-VNet herstellen können.

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

**Linux-Bereitstellung**

Führen Sie die folgenden Schritte aus, um die Konnektivität der simulierten lokalen Umgebung mit den Spoke-VNETs mit Linux-VMs zu testen:

1. Suchen Sie im Azure-Portal die VM namens `jb-vm1` in der `onprem-jb-rg`-Ressourcengruppe.

2. Klicken Sie auf `Connect`, und kopieren Sie den im Portal angezeigten `ssh`-Befehl. 

3. Führen Sie in einer Linux-Eingabeaufforderung `ssh` zum Herstellen der Verbindung mit der simulierten lokalen Umgebung aus. Verwenden Sie das Kennwort, das Sie in der `onprem.json`-Parameterdatei angegeben haben.

5. Verwenden Sie den Befehl `ping`, um die Konnektivität mit den Jumpbox-VMs in den einzelnen Spokes zu testen:

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a>Herstellen von Konnektivität zwischen Spokes

Dieser Schritt ist optional. Wenn Sie zulassen möchten, dass Spokes Verbindungen miteinander herstellen können, müssen Sie ein virtuelles Netzwerkgerät als Router im Hub-VNet verwenden. Außerdem müssen Sie erzwingen, dass Datenverkehr von den Spokes über den Router verläuft, wenn versucht wird, eine Verbindung mit anderen Spokes herzustellen. Führen Sie die folgenden Schritte aus, um ein einfaches Beispiel-Netzwerkgerät als einzelne VM und benutzerdefinierte Routen bereitzustellen, um die Verbindungsherstellung zwischen den zwei Spoke-VNETs zu ermöglichen:

1. Öffnen Sie die Datei `hub-nva.json` . Ersetzen Sie die Werte für `adminUsername` und `adminPassword`.

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. Führen Sie den folgenden Befehl aus:

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Hub-Spoke-Topologie in Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Hub-Spoke-Topologie in Azure mit transitivem Routing"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Hub-Spoke-Topologie in Azure mit transitivem Routing unter Verwendung eines NVA"
[3]: ./images/hub-spokehub-spoke.svg "Hub-Spoke-Hub-Spoke-Topologie in Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
