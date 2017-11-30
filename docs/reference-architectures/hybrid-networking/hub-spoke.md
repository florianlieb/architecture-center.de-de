---
title: Implementieren einer Hub-Spoke-Netzwerktopologie in Azure
description: Vorgehensweise zum Implementieren einer Hub-Spoke-Netzwerktopologie in Azure
author: telmosampaio
ms.date: 05/05/2017
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: e6f07a7962dd5728226b023700268340590d97a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
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

## <a name="architecture"></a>Architektur

Diese Architektur besteht aus den folgenden Komponenten:

* **Lokales Netzwerk**: Ein privates lokales Netzwerk innerhalb einer Organisation.

* **VPN-Gerät**: Ein Gerät oder ein Dienst, der externe Konnektivität mit dem lokalen Netzwerk bereitstellt. Bei dem VPN-Gerät kann es sich um ein Hardwaregerät oder eine Softwarelösung wie den Routing- und RAS-Dienst (RRAS) unter Windows Server 2012 handeln. Eine Liste der unterstützten VPN-Appliances und Informationen zur Konfiguration ausgewählter VPN-Geräte für die Verbindung mit Azure finden Sie unter [Informationen zu VPN-Geräten und IPsec-/IKE-Parametern für VPN-Gatewayverbindungen zwischen Standorten][vpn-appliance].

* **VPN-Gateway für ein virtuelles Netzwerk oder ExpressRoute-Gateway**: Über das Gateway für virtuelle Netzwerke kann das VNET zur Konnektivität mit Ihrem lokalen Netzwerk eine Verbindung mit dem VPN-Gerät oder eine ExpressRoute-Verbindung herstellen. Weitere Informationen finden Sie unter [Verbinden eines lokalen Netzwerks mit einem virtuellen Microsoft Azure-Netzwerk][connect-to-an-Azure-vnet].

> [!NOTE]
> In den Bereitstellungsskripts für diese Referenzarchitektur werden ein VPN-Gateway für die Konnektivität und ein VNET in Azure verwendet, um Ihr lokales Netzwerk zu simulieren.

* **Hub-VNET**: Das Azure-VNET, das als Hub in der Hub-Spoke-Topologie verwendet wird. Der Hub ist der zentrale Konnektivitätspunkt für Ihr lokales Netzwerk, mit dem Sie Dienste hosten, die von den verschiedenen in den Spoke-VNETs gehosteten Workloads genutzt werden können.

* **Gatewaysubnetz**: Die Gateways für die virtuellen Netzwerke befinden sich im selben Subnetz.

* **Subnetz für gemeinsame Dienste**: Ein Subnetz im Hub-VNET, das für das Hosten von Diensten verwendet wird, die von allen Spokes gemeinsam genutzt werden können (z.B. DNS oder AD DS).

* **Spoke-VNETs**: Ein oder mehrere Azure-VNETs, die als Spokes in der Hub-Spoke-Topologie verwendet werden. Spokes können verwendet werden, um Workloads in ihren eigenen VNETs, die getrennt von anderen Spokes verwaltet werden, zu isolieren. Jede Workload kann mehrere Schichten umfassen, wobei mehrere Subnetze über Azure Load Balancer verbunden sind. Weitere Informationen zur Anwendungsinfrastruktur finden Sie unter [Ausführen von Windows-VM-Workloads][windows-vm-ra] und [Ausführen von Linux-VM-Workloads][linux-vm-ra].

* **VNET-Peering**: Über eine [Peeringverbindung][vnet-peering] können zwei VNETs in derselben Azure-Region miteinander verbunden werden. Peeringverbindungen sind nicht-transitive Verbindungen zwischen VNETs mit niedrigen Latenzen. Sobald eine Peeringverbindung hergestellt wurde, tauschen die VNETs ohne Einsatz eines Routers Datenverkehr über den Azure-Backbone aus. In einer Hub-Spoke-Netzwerktopologie wird durch VNET-Peering eine Verbindung zwischen dem Hub und den einzelnen Spokes hergestellt.

> [!NOTE]
> In diesem Artikel werden ausschließlich [Resource Manager](/azure/azure-resource-manager/resource-group-overview)-Bereitstellungen behandelt, Sie können jedoch auch eine Verbindung zwischen einem klassischen VNET und einem Resource Manager-VNET im selben Abonnement herstellen. Auf diese Weise können Ihre Spokes klassische Bereitstellungen hosten und trotzdem von gemeinsamen Diensten im Hub profitieren.


## <a name="recommendations"></a>Recommendations

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

Eine Bereitstellung für diese Architektur ist auf [GitHub][ref-arch-repo] verfügbar. Bei dieser werden zum Testen der Konnektivität Ubuntu-VMs in jedem VNET verwendet. Es gibt keine tatsächlichen Dienste, die im Subnetz für **gemeinsame Dienste** im **Hub-VNET** gehostet werden.

### <a name="prerequisites"></a>Voraussetzungen

Bevor Sie die Referenzarchitektur in Ihrem eigenen Abonnement bereitstellen können, müssen Sie die folgenden Schritte durchführen.

1. Klonen oder forken Sie das GitHub-Repository [AzureCAT-Referenzarchitekturen][ref-arch-repo], oder laden Sie die zugehörige ZIP-Datei herunter.

2. Wenn Sie stattdessen die Azure CLI verwenden möchten, vergewissern Sie sich, dass Azure CLI 2.0 auf Ihrem Computer installiert ist. Um die CLI zu installieren, folgen Sie den Anweisungen unter [Installieren von Azure CLI 2.0][azure-cli-2].

3. Wenn Sie PowerShell bevorzugen, stellen Sie sicher, dass auf Ihrem Computer das neueste PowerShell-Modul für Azure installiert ist. Um das neueste Azure PowerShell-Modul zu installieren, folgen Sie den Anweisungen unter [Installieren von PowerShell für Azure][azure-powershell].

4. Melden Sie sich über eine Eingabeaufforderung, eine bash-Eingabeaufforderung oder die PowerShell-Eingabeaufforderung bei Ihrem Azure-Konto an. Verwenden Sie dazu die unten aufgeführten Befehle, und befolgen Sie die Anweisungen.

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>Bereitstellen des simulierten lokalen Rechenzentrums

Führen Sie die folgenden Schritte durch, um das simulierte lokale Rechenzentrum als Azure-VNET bereitzustellen.

1. Navigieren Sie zum Ordner `hybrid-networking\hub-spoke\onprem` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.

2. Öffnen Sie die Datei `onprem.vm.parameters.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 11 und 12 an, und speichern Sie die Datei.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um die simulierte lokale Umgebung als VNET in Azure bereitzustellen. Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.

  ```bash
  sh ./onprem.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `ra-onprem-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.

4. Warten Sie, bis die Bereitstellung abgeschlossen ist. Diese Bereitstellung erstellt ein virtuelles Netzwerk, einen virtuellen Ubuntu-Computer und ein VPN-Gateway. Die Erstellung des VPN-Gateways kann mehr als 40 Minuten in Anspruch nehmen.

### <a name="azure-hub-vnet"></a>Azure-Hub-VNET

Führen Sie die folgenden Schritte durch, um das Hub-VNET bereitzustellen und eine Verbindung mit dem oben erstellten simulierten lokalen VNET herzustellen.

1. Navigieren Sie zum Ordner `hybrid-networking\hub-spoke\hub` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.

2. Öffnen Sie die Datei `hub.vm.parameters.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 11 und 12 an, und speichern Sie die Datei.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Öffnen Sie die Datei `hub.gateway.parameters.json`, geben Sie einen gemeinsam verwendeten Schlüssel wie unten dargestellt zwischen den Anführungszeichen in Zeile 23 an, und speichern Sie die Datei. Notieren Sie sich diesen Wert, da Sie ihn später für die Bereitstellung benötigen.

  ```bash
  "sharedKey": "",
  ```

4. Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um die simulierte lokale Umgebung als VNET in Azure bereitzustellen. Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.

  ```bash
  sh ./hub.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus
  ```

  ```powershell
  ./hub.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus
  ```
  > [!NOTE]
  > Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `ra-hub-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.

5. Warten Sie, bis die Bereitstellung abgeschlossen ist. Diese Bereitstellung erstellt ein virtuelles Netzwerk, einen virtuellen Ubuntu-Computer, ein VPN-Gateway und eine Verbindung mit dem im vorherigen Abschnitt erstellten Gateway. Die Erstellung des VPN-Gateways kann mehr als 40 Minuten in Anspruch nehmen.

### <a name="connection-from-on-premises-to-the-hub"></a>Verbindung zwischen lokalen Rechenzentren und dem Hub

Führen Sie die folgenden Schritte durch, um eine Verbindung zwischen dem simulierten lokalen Rechenzentrum und dem Hub-VNET herzustellen.

1. Navigieren Sie zum Ordner `hybrid-networking\hub-spoke\onprem` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.

2. Öffnen Sie die Datei `onprem.connection.parameters.json`, geben Sie einen gemeinsam verwendeten Schlüssel wie unten dargestellt zwischen den Anführungszeichen in Zeile 9 an, und speichern Sie die Datei. Der Wert dieses gemeinsam verwendeten Schlüssels muss mit dem des für das lokale, zuvor bereitgestellte Gateway verwendeten Schlüssels identisch sein.

  ```bash
  "sharedKey": "",
  ```

3. Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um die simulierte lokale Umgebung als VNET in Azure bereitzustellen. Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.

  ```bash
  sh ./onprem.connection.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.connection.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `ra-onprem-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.

4. Warten Sie, bis die Bereitstellung abgeschlossen ist. Diese Bereitstellung stellt eine Verbindung zwischen dem VNET für die Simulation eines lokalen Rechenzentrums und dem Hub-VNET her.

### <a name="azure-spoke-vnets"></a>Azure-Spoke-VNETs

Führen Sie die folgenden Schritte durch, um die Spoke-VNETs bereitzustellen und eine Verbindung mit dem oben erstellten Hub-VNET herzustellen.

1. Wechseln Sie in den Ordner `hybrid-networking\hub-spoke\spokes` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.

2. Öffnen Sie die Datei `spoke1.web.parameters.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 53 und 54 an, und speichern Sie die Datei.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Wiederholen Sie den vorherigen Schritt für die Datei `spoke2.web.parameters.json`.

4. Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um den ersten Spoke bereitzustellen und ihn mit dem Hub zu verbinden. Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```
  > [!NOTE]
  > Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `ra-spoke1-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.

5. Warten Sie, bis die Bereitstellung abgeschlossen ist. Diese Bereitstellung erstellt ein virtuelles Netzwerk, ein Lastenausgleichsmodul mit drei virtuellen Computern, auf denen Ubuntu und Apache ausgeführt werden, sowie eine VNET-Peeringverbindung mit dem im vorherigen Abschnitt erstellten Hub-VNET. Diese Bereitstellung kann mehr als 20 Minuten in Anspruch nehmen.

6. Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um den ersten Spoke bereitzustellen und ihn mit dem Hub zu verbinden. Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```
  > [!NOTE]
  > Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `ra-spoke2-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.

5. Warten Sie, bis die Bereitstellung abgeschlossen ist. Diese Bereitstellung erstellt ein virtuelles Netzwerk, ein Lastenausgleichsmodul mit drei virtuellen Computern, auf denen Ubuntu und Apache ausgeführt werden, sowie eine VNET-Peeringverbindung mit dem im vorherigen Abschnitt erstellten Hub-VNET. Diese Bereitstellung kann mehr als 20 Minuten in Anspruch nehmen.

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a>Herstellen einer Peeringverbindung zwischen Azure-Hub-VNETs und Spoke-VNETs

Führen Sie die folgenden Schritte durch, um VNET-Peeringverbindungen für das Hub-VNET herzustellen.

1. Wechseln Sie in den Ordner `hybrid-networking\hub-spoke\hub` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.

2. Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um die Peeringverbindung für den ersten Spoke herzustellen. Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 1
  ```

2. Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um die Peeringverbindung für den zweiten Spoke herzustellen. Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 2
  ```

### <a name="test-connectivity"></a>Testen der Konnektivität

Führen Sie die folgenden Schritte durch, um zu überprüfen, ob die mit einer lokalen Rechenzentrumsbereitstellung verbundene Hub-Spoke-Topologie funktioniert.

1. Stellen Sie über das [Azure-Portal][Portal] eine Verbindung mit Ihrem Abonnement her, und navigieren Sie in der Ressourcengruppe `ra-onprem-rg` zum virtuellen Computer `ra-onprem-vm1`.

2. Beachten Sie auf dem Blatt `Overview` `Public IP address` für die VM.

3. Stellen Sie über einen SSH-Client eine Verbindung mit der oben genannten IP-Adresse her, indem Sie den Benutzernamen und das Kennwort verwenden, den bzw. das Sie bei der Bereitstellung angegeben haben.

4. Führen Sie in der Eingabeaufforderung in der VM, mit der Sie eine Verbindung hergestellt haben, den folgenden Befehl aus, um die Konnektivität zwischen dem lokalen VNET und dem VNET von Spoke 1 zu testen.

  ```bash
  ping 10.1.1.37
  ```

### <a name="add-connectivity-between-spokes"></a>Herstellen von Konnektivität zwischen Spokes

Wenn zwischen Spokes eine Verbindung hergestellt werden soll, müssen Sie für jeden Spoke UDRs bereitstellen, der den für andere Spokes vorgesehenen Datenverkehr an das Gateway im Hub-VNET weiterleitet. Führen Sie die folgenden Schritte durch, um zu prüfen, dass Sie derzeit keine Verbindung zwischen Spokes herstellen können. Geben Sie anschließend die UDRs an, und testen Sie erneut die Konnektivität.

1. Wiederholen Sie die Schritte 1 bis 4 oben, wenn Sie nicht mehr mit der Jumpbox-VM verbunden sind.

2. Stellen Sie eine Verbindung mit einem der Webserver in Spoke 1 her.

  ```bash
  ssh 10.1.1.37
  ```

3. Testen Sie die Konnektivität zwischen Spoke 1 und Spoke 2. Bei dem Test sollte ein Fehler auftreten.

  ```bash
  ping 10.1.2.37
  ```

4. Kehren Sie zur Eingabeaufforderung Ihres Computers zurück.

5. Wechseln Sie in den Ordner `hybrid-networking\hub-spoke\spokes` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.

6. Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um eine UDR für den ersten Spoke bereitzustellen. Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```

7. Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um eine UDR für den zweiten Spoke bereitzustellen. Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```

8. Kehren Sie zurück zum SSH-Terminal zurück.

9. Testen Sie die Konnektivität zwischen Spoke 1 und Spoke 2. Der Test sollte erfolgreich abgeschlossen werden.

  ```bash
  ping 10.1.2.37
  ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azure-powershell]: /powershell/azure/install-azure-ps?view=azuresmps-3.7.0
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

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Hub-Spoke-Topologie in Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Hub-Spoke-Topologie in Azure mit transitivem Routing"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Hub-Spoke-Topologie in Azure mit transitivem Routing unter Verwendung eines NVA"
[3]: ./images/hub-spokehub-spoke.svg "Hub-Spoke-Hub-Spoke-Topologie in Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
