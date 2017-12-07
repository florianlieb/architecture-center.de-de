---
title: "Verbinden eines lokalen Netzwerks mit Azure über ExpressRoute"
description: "Erläutert, wie Sie eine sichere Netzwerkarchitektur zwischen Standorten implementieren, die ein virtuelles Azure-Netzwerk und ein lokales Netzwerk umfasst, die über ExpressRoute verbunden werden."
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute-vpn-failover
pnp.series.prev: vpn
cardTitle: ExpressRoute
ms.openlocfilehash: 671be5118faaefab5ba5348de81642d8a8124b59
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute"></a>Verbinden eines lokalen Netzwerks mit Azure über ExpressRoute

Diese Referenzarchitektur zeigt, wie man mithilfe von [Azure ExpressRoute][expressroute-introduction] ein lokales Netzwerk mit virtuellen Azure-Netzwerken verbindet. ExpressRoute-Verbindungen nutzen eine dedizierte private Verbindung über einen Drittanbieter für die Konnektivität. Die private Verbindung erweitert Ihr lokales Netzwerk auf Azure. [**So stellen Sie diese Lösung bereit**.](#deploy-the-solution)

![[0]][0]

*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*

## <a name="architecture"></a>Architektur

Die Architektur umfasst die folgenden Komponenten.

* **Lokales Unternehmensnetzwerk**. Ein privates lokales Netzwerk innerhalb einer Organisation.

* **ExpressRoute-Verbindung**. Eine vom Konnektivitätsanbieter bereitgestellte Layer 2- oder Layer 3-Verbindung, die das lokale Netzwerk über die Edgerouter mit Azure verbindet. Für die Verbindung wird die vom Konnektivitätsanbieter verwaltete Hardwareinfrastruktur verwendet.

* **Lokale Edgerouter**. Route, die das lokale Netzwerk mit der über den Anbieter verwalteten Verbindung verbinden. Je nachdem, wie Ihre Verbindung bereitgestellt wird, müssen Sie möglicherweise die öffentlichen IP-Adressen angeben, die von den Routern verwendet werden.
* **Microsoft Edge-Router**. Zwei Router in einer Aktiv/Aktiv-Konfiguration mit Hochverfügbarkeit. Diese Router ermöglichen es einem Konnektivitätsanbieter, eine direkte Verbindung zwischen ihren Leitungen und dem Rechenzentrum herzustellen. Je nachdem, wie Ihre Verbindung bereitgestellt wird, müssen Sie möglicherweise die öffentlichen IP-Adressen angeben, die von den Routern verwendet werden.

* **Virtuelle Azure-Netzwerke (VNETs)**. Jedes VNET befindet sich in einer einzelnen Azure-Region und kann mehrere Anwendungsebenen hosten. Anwendungsebenen können mithilfe von Subnetzen in jedem VNET segmentiert werden.

* **Öffentliche Azure-Dienste**. Azure-Dienste, die innerhalb einer Hybridanwendung genutzt werden können. Diese Dienste stehen auch über das Internet zur Verfügung, aber der Zugriff über eine ExpressRoute-Leitung bietet niedrige Latenz und eine besser vorhersagbare Leistung, da der Datenverkehr nicht über das Internet geleitet wird. Verbindungen werden über das [öffentliche Peering][expressroute-peering] hergestellt. Die hierbei verwendeten Adressen gehören entweder der Organisation oder werden vom Konnektivitätsanbieter bereitgestellt.

* **Office 365-Dienste**. Die öffentlich verfügbaren Office 365-Anwendungen und -Dienste, die von Microsoft bereitgestellt werden. Verbindungen werden über das [Microsoft-Peering][expressroute-peering] hergestellt. Die hierbei verwendeten Adressen gehören entweder der Organisation oder werden vom Konnektivitätsanbieter bereitgestellt. Es ist auch möglich, per Microsoft-Peering eine direkte Verbindung mit Microsoft CRM Online herzustellen.

* **Konnektivitätsanbieter** (nicht gezeigt). Unternehmen, die Layer 2- oder Layer 3-Konnektivität zwischen Ihrem Rechenzentrum und einem Azure-Rechenzentrum bereitstellen.

## <a name="recommendations"></a>Empfehlungen

Die folgenden Empfehlungen gelten für die meisten Szenarios. Sofern Sie keine besonderen Anforderungen haben, die Vorrang haben, sollten Sie diese Empfehlungen befolgen.

### <a name="connectivity-providers"></a>Konnektivitätsanbieter

Wählen Sie einen geeigneten ExpressRoute-Konnektivitätsanbieter für Ihren Standort aus. Verwenden Sie den folgenden Azure PowerShell-Befehl, um eine Liste der Konnektivitätsanbieter abzurufen, die an Ihrem Standort verfügbar sind:

```powershell
Get-AzureRmExpressRouteServiceProvider
```

ExpressRoute-Konnektivitätsanbieter verbinden Ihr Rechenzentrum über die folgenden Methoden mit Microsoft:

* **Per Co-Location und Cloud Exchange**. Wenn Sie sich in einer Einrichtung mit Cloud Exchange befinden, können Sie virtuelle Querverbindungen mit Azure über den Ethernet Exchange des Co-Location-Anbieters anfordern. Co-Location-Anbieter stellen entweder Layer 2-Querverbindungen oder verwaltete Layer 3-Querverbindungen zwischen Ihrer Infrastruktur in der Co-Location-Einrichtung und Azure bereit.
* **Point-to-Point-Ethernet-Verbindungen**. Sie können Ihre lokalen Rechenzentren/Büros über Point-to-Point-Ethernet-Leitungen mit Azure verbinden. Point-to-Point-Ethernet-Anbieter können Layer 2-Verbindungen oder verwaltete Layer 3-Verbindungen zwischen Ihrem Standort und Azure bereitstellen.
* **Any-to-Any-Netzwerke (IPVPN)**. Sie können Ihr Fernnetz (WAN) in Azure integrieren. IPVPN-Anbieter (normalerweise MPLS VPN) stellen Any-to-Any-Konnektivität zwischen Ihren Niederlassungen und den Rechenzentren bereit. Azure kann mit Ihrem WAN verbunden werden, sodass es wie eine normale Niederlassung erscheint. WAN-Anbieter stellen in der Regel verwaltete Layer 3-Konnektivität bereit.

Weitere Informationen zu Konnektivitätsanbietern finden Sie der [ExpressRoute-Übersicht][expressroute-introduction].

### <a name="expressroute-circuit"></a>ExpressRoute-Verbindung

Stellen Sie sicher, dass Ihre Organisation die [ExpressRoute-Voraussetzungen][expressroute-prereqs] zum Herstellen der Verbindung mit Azure erfüllt.

Sofern nicht bereits geschehen, fügen Sie ein Subnetz namens `GatewaySubnet` zu Ihrem Azure-VNET hinzu, und erstellen Sie mit dem Azure-VPN-Gatewaydienst ein virtuelles ExpressRoute-Gateway. Weitere Informationen zur Vorgehensweise finden Sie unter [ExpressRoute-Workflows für die Verbindungsbereitstellung und Verbindungszustände][ExpressRoute-provisioning].

Gehen Sie folgendermaßen vor, um eine ExpressRoute-Verbindung zu erstellen:

1. Führen Sie den folgenden PowerShell-Befehl aus:
   
    ```powershell
    New-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>> -Location <<location>> -SkuTier <<sku-tier>> -SkuFamily <<sku-family>> -ServiceProviderName <<service-provider-name>> -PeeringLocation <<peering-location>> -BandwidthInMbps <<bandwidth-in-mbps>>
    ```
2. Senden Sie den `ServiceKey` für die neue Verbindung an den Dienstanbieter.

3. Warten Sie die Bereitstellung der Verbindung durch den Anbieter ab. Überprüfen Sie mit dem folgenden PowerShell-Befehl den Bereitstellungszustand einer Verbindung:
   
    ```powershell
    Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    ```

    Das Feld `Provisioning state` im Abschnitt `Service Provider` der Ausgabe ändert sich von `NotProvisioned` in `Provisioned`, wenn die Verbindung bereit ist.

    > [!NOTE]
    > Wenn Sie eine Layer 3-Verbindung verwenden, sollte die Konfiguration und Verwaltung für das Routing über den Anbieter erfolgen. Sie teilen dem Anbieter die erforderlichen Informationen zum Implementieren der geeigneten Routen mit.
    > 
    > 

4. Bei Verwendung einer Layer 2-Verbindung:

    1. Reservieren Sie zwei /30-Subnetze mit gültigen öffentlichen IP-Adressen für jeden Peeringtyp, den Sie implementieren möchten. Über diese /30-Subnetze werden IP-Adressen für die Router bereitgestellt, die für die Verbindung verwendet werden. Wenn Sie privates, öffentliches und Microsoft-Peering verwenden, benötigen Sie 6 /30-Subnetze mit gültigen öffentlichen IP-Adressen.     

    2. Konfigurieren Sie das Routing für die ExpressRoute-Verbindung. Führen Sie die folgenden PowerShell-Befehle für jeden Peeringtyp aus, den Sie konfigurieren möchten (privat, öffentlich, Microsoft). Weitere Informationen finden Sie unter [Erstellen und Ändern des Peerings für eine ExpressRoute-Verbindung mithilfe von PowerShell][configure-expressroute-routing].
   
        ```powershell
        Set-AzureRmExpressRouteCircuitPeeringConfig -Name <<peering-name>> -Circuit <<circuit-name>> -PeeringType <<peering-type>> -PeerASN <<peer-asn>> -PrimaryPeerAddressPrefix <<primary-peer-address-prefix>> -SecondaryPeerAddressPrefix <<secondary-peer-address-prefix>> -VlanId <<vlan-id>>

        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit <<circuit-name>>
        ```

    3. Reservieren Sie einen weiteren Pool mit gültigen öffentlichen IP-Adressen für die Netzwerkadressübersetzung (NAT) für das öffentliche Peering und das Microsoft-Peering. Es wird empfohlen, für jeden Peeringtyp einen eigenen Pool zu verwenden. Teilen Sie den Pool Ihrem Konnektivitätsanbieter mit, damit dieser BGP-Ankündigungen (Border Gateway Protocol) für diese Bereiche konfigurieren kann.

5. Führen Sie die folgenden PowerShell-Befehle aus, um Ihre privaten VNETs mit der ExpressRoute-Verbindung zu verknüpfen. Weitere Informationen finden Sie unter [Verbinden eines virtuellen Netzwerks mit einer ExpressRoute-Verbindung][link-vnet-to-expressroute].

    ```powershell
    $circuit = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $gw = Get-AzureRmVirtualNetworkGateway -Name <<gateway-name>> -ResourceGroupName <<resource-group>>
    New-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> -ResourceGroupName <<resource-group>> -Location <<location> -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
    ```

Sie können mehrere VNETs in unterschiedlichen Regionen mit derselben ExpressRoute-Leitung verbinden, solange VNETs und ExpressRoute-Leitung sich innerhalb derselben geopolitischen Region befinden.

### <a name="troubleshooting"></a>Problembehandlung 

Wenn über eine zuvor funktionsfähige ExpressRoute-Leitung jetzt keine Verbindung mehr hergestellt werden kann, obwohl weder lokal noch in Ihrem privaten VNET Konfigurationsänderungen durchgeführt wurden, müssen Sie sich möglicherweise mit dem Konnektivitätsanbieter in Verbindung setzen und gemeinsam an der Problemlösung arbeiten. Überprüfen Sie mithilfe der folgenden PowerShell-Befehle, ob die ExpressRoute-Verbindung bereitgestellt wurde:

```powershell
Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

Die Ausgabe dieses Befehls zeigt verschiedene Verbindungseigenschaften, darunter `ProvisioningState`, `CircuitProvisioningState` und `ServiceProviderProvisioningState`, wie unten gezeigt.

```
ProvisioningState                : Succeeded
Sku                              : {
                                     "Name": "Standard_MeteredData",
                                     "Tier": "Standard",
                                     "Family": "MeteredData"
                                   }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : NotProvisioned
```

Wenn `ProvisioningState` nicht auf `Succeeded` festgelegt ist, nachdem Sie versucht haben, eine neue Verbindung zu erstellen, entfernen Sie die Verbindung über den folgenden Befehl. Versuchen Sie anschließend erneut, die Verbindung zu erstellen.

```powershell
Remove-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

Wenn Ihr Anbieter die Verbindung bereits bereitgestellt hat und `ProvisioningState` auf `Failed` festgelegt ist oder `CircuitProvisioningState` nicht `Enabled` lautet, wenden Sie sich an Ihren Anbieter, um weitere Unterstützung zu erhalten.

## <a name="scalability-considerations"></a>Überlegungen zur Skalierbarkeit

ExpressRoute-Verbindungen bieten einen Pfad mit hoher Bandbreite zwischen Netzwerken. Allgemein gilt: Je größer die Bandbreite, desto höher die Kosten. 

ExpressRoute bietet zwei [Tarife][expressroute-pricing] für Kunden: einen Volumentarif und eine Datenflatrate. Die Gebühren richten sich nach der Verbindungsbandbreite. Die verfügbare Bandbreite variiert je nach Anbieter. Verwenden Sie das Cmdlet `Get-AzureRmExpressRouteServiceProvider`, um die verfügbaren Anbieter in Ihrer Region sowie die angebotenen Bandbreiten anzuzeigen.
 
Eine einzelne ExpressRoute-Verbindung kann eine bestimmte Anzahl von Peerings und VNET-Verbindungen unterstützen. Weitere Informationen finden Sie unter [ExpressRoute-Limits](/azure/azure-subscription-service-limits).

Das ExpressRoute Premium-Add-On bietet gegen eine zusätzliche Gebühr einige zusätzliche Funktionen:

* Erhöhung der Routengrenzwerte für öffentliches und privates Peering. 
* Eine höhere Anzahl von VNET-Verbindungen pro ExpressRoute-Leitung. 
* Globale Konnektivität für Dienste.

Ausführliche Informationen finden Sie unter [ExpressRoute – Preise][expressroute-pricing]. 

ExpressRoute-Verbindungen sind so konzipiert, dass Sie das erworbene Bandbreitenlimit für kurzfristige Spitzen im Netzwerkdatenverkehr um das Doppelte überschreiten können, ohne dass zusätzliche Kosten anfallen. Dies wird durch den Einsatz redundanter Verbindungen erreicht. Allerdings wird dieses Feature nicht von allen Konnektivitätsanbietern unterstützt. Stellen Sie sicher, dass Ihr Konnektivitätsanbieter dieses Feature unterstützt, bevor Sie sich darauf verlassen.

Wenngleich einige Anbieter eine Änderung der Bandbreite zulassen, sollten Sie eine anfängliche Bandbreite auswählen, die Ihre Anforderungen übersteigt und einen Puffer für künftiges Wachstum bietet. Wenn Sie zu einem späteren Zeitpunkt mehr Bandbreite benötigen, haben Sie zwei Optionen:

- Erhöhen Sie die Bandbreite. Diese Option sollte nach Möglichkeit vermieden werden, weil nicht alle Anbieter eine dynamische Erhöhung der Bandbreite unterstützen. Wenn jedoch eine Bandbreitenerhöhung benötigt wird, fragen Sie bei Ihrem Anbieter an, ob das Ändern der ExpressRoute-Bandbreiteneigenschaften über PowerShell-Befehle unterstützt wird. Falls dem so ist, führen Sie die nachstehenden Befehle aus.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $ckt.ServiceProviderProperties.BandwidthInMbps = <<bandwidth-in-mbps>>
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    Sie können die Bandbreite ohne Konnektivitätsverlust erhöhen. Ein Downgrade der Bandbreite führt zu einer Konnektivitätsunterbrechung, weil Sie die Verbindung löschen und mit der neuen Konfiguration neu erstellen müssen.

- Stellen Sie Ihren Tarif um, und/oder führen Sie ein Upgrade auf Premium durch. Führen Sie zu diesem Zweck die folgenden Befehle aus. Die Eigenschaft `Sku.Tier` kann `Standard` oder `Premium` lauten, die Eigenschaft `Sku.Name` kann auf `MeteredData` oder `UnlimitedData` festgelegt werden.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>

    $ckt.Sku.Tier = "Premium"
    $ckt.Sku.Family = "MeteredData"
    $ckt.Sku.Name = "Premium_MeteredData"

    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    > [!IMPORTANT]
    > Stellen Sie sicher, dass die Eigenschaft `Sku.Name` dem `Sku.Tier` und der `Sku.Family` entspricht. Wenn Sie Familie und Tarif ändern, aber nicht den Namen, wird Ihre Verbindung deaktiviert.
    > 
    > 

    Ein SKU-Upgrade kann ohne Unterbrechung durchgeführt werden, aber Sie können nicht von der Datenflatrate auf den Volumentarif umstellen. Bei einem Downgrade der SKU muss Ihr Bandbreitenverbrauch innerhalb des Standardlimits für die Standard-SKU liegen.

## <a name="availability-considerations"></a>Überlegungen zur Verfügbarkeit

ExpressRoute unterstützt keine Protokolle für die Routerredundanz – wie z.B. HSRP (Hot Standby Routing Protocol) und VRRP (Virtual Router Redundancy Protocol) – für die Implementierung von Hochverfügbarkeit. Stattdessen wird ein redundantes BGP-Sitzungspaar pro Peering verwendet. Um hochverfügbare Verbindungen mit Ihrem Netzwerk zu ermöglichen, stellt Azure zwei redundante Ports auf zwei Routern (Microsoft Edge) in einer Aktiv/Aktiv-Konfiguration bereit.

BGP-Sitzungen verwendet standardmäßig einen Leerlauftimeoutwert von 60 Sekunden. Wenn für eine Sitzung drei Mal ein Timeout auftritt (180 Sekunden gesamt), wird der Router als nicht verfügbar markiert, und der gesamte Datenverkehr wird auf den verbleibenden Router umgeleitet. Dieses 180-Sekunden-Timeout kann für kritische Anwendungen zu lang sein. Wenn dies der Fall ist, können Sie die BGP-Timeouteinstellungen auf dem lokalen Router auf einen kürzeren Zeitraum festlegen.

Sie können für Ihre Azure-Verbindung auf unterschiedliche Weise Hochverfügbarkeit konfigurieren – je nachdem, welchen Anbieter Sie verwenden und wie viele ExpressRoute-Verbindungen und VNET-Gatewayverbindungen Sie konfigurieren möchten. Nachfolgend werden die verfügbaren Optionen zusammengefasst:

* Wenn Sie eine Layer 2-Verbindung verwenden, stellen Sie redundante Router in Ihrem lokalen Netzwerk in einer Aktiv/Aktiv-Konfiguration bereit. Verbinden Sie die primäre Verbindung mit dem ersten und die zweite Verbindung mit dem zweiten Router. Auf diese Weise erhalten Sie eine hochverfügbare Verbindung auf beiden Seiten der Leitung. Dies ist erforderlich, wenn Sie die Vereinbarung zum Servicelevel für ExpressRoute (SLA) benötigen. Ausführliche Informationen finden Sie unter [SLA für Azure ExpressRoute][sla-for-expressroute].

    Das folgende Diagramm zeigt eine Konfiguration mit redundanten lokalen Routern, die mit der primären und sekundären Leitung verbunden sind. Jede Leitung verarbeitet den Datenverkehr für ein öffentliches Peering und ein privates Peering (jedes Peering umfasst ein dediziertes Paar aus /30-Adressräumen, wie im vorherigen Abschnitt beschrieben).

    ![[1]][1]

* Überprüfen Sie bei Verwendung einer Layer 3-Verbindung, ob redundante BGP-Sitzungen zur Sicherstellung der Verfügbarkeit bereitgestellt werden.

* Verbinden Sie das VNET mit mehreren ExpressRoute-Leitungen, die von unterschiedlichen Dienstanbietern bereitgestellt werden. Diese Strategie bietet zusätzliche Hochverfügbarkeit und Notfallwiederherstellungsfunktionen.

* Konfigurieren Sie ein Site-to-Site-VPN als Failoverpfad für ExpressRoute. Weitere Informationen finden Sie unter [Verbinden eines lokalen Netzwerks mit Azure unter Verwendung von ExpressRoute mit VPN-Failover][highly-available-network-architecture].
 Diese Option gilt nur für das private Peering. Für Azure- und Office 365-Dienste ist das Internet der einzige Failoverpfad. 

## <a name="manageability-considerations"></a>Überlegungen zur Verwaltbarkeit

Sie können mit dem [Azure Connectivity Toolkit (AzureCT)][azurect] die Konnektivität zwischen Ihrem lokalen Rechenzentrum und Azure überwachen. 

## <a name="security-considerations"></a>Sicherheitshinweise

Sie können die Sicherheitsoptionen für Ihre Azure-Verbindung abhängig von Ihren Anforderungen im Hinblick auf Sicherheit und Compliance auf unterschiedliche Weise konfigurieren. 

ExpressRoute arbeitet auf Layer 3. Bedrohungen in der Anwendungsebene können durch den Einsatz einer Netzwerksicherheitsappliance verhindert werden, die den Datenverkehr auf autorisierte Ressourcen beschränkt. Darüber hinaus können ExpressRoute-Verbindungen mit öffentlichem Peering nur lokal initiiert werden. Dadurch wird verhindert, dass ein nicht autorisierter Dienst aus dem Internet auf lokale Daten zugreift und diese kompromittiert.

Um die Sicherheit zu maximieren, platzieren Sie Netzwerksicherheitsappliances zwischen dem lokalen Netzwerk und den Edgeroutern des Anbieters. Dies trägt dazu bei, nicht autorisierten Datenverkehr aus dem VNET zu unterbinden:

![[2]][2]

Im Rahmen von Überwachung oder Compliance kann es notwendig sein, den direkten Zugriff von VNET-Komponenten auf das Internet zu verbieten und eine [Tunnelerzwingung][forced-tuneling] zu implementieren. In diesem Fall sollte der Internetdatenverkehr an einen lokal ausgeführten Proxy umgeleitet werden, wo er überwacht werden kann. Der Proxy kann so konfiguriert werden, dass er nicht autorisierten ausgehenden Datenverkehr blockiert und potenziell schädlichen eingehenden Datenverkehr filtert.

![[3]][3]

Um die Sicherheit zu maximieren, aktivieren Sie keine öffentliche IP-Adresse für Ihre VMs, und stellen Sie mithilfe von Netzwerksicherheitsgruppen (NSGs) sicher, dass diese VMs nicht öffentlich zugänglich sind. VMs sollten nur über die interne IP-Adresse erreichbar sein. Diese Adressen können über das ExpressRoute-Netzwerk verfügbar gemacht werden, sodass DevOps-Mitarbeiter vor Ort eine Konfiguration oder Wartung durchführen können.

Wenn Sie Verwaltungsendpunkte für VMs gegenüber einem externen Netzwerk offenlegen müssen, verwenden Sie NSGs oder Zugriffssteuerunglisten (ACLs), um die Sichtbarkeit dieser Ports auf eine Whitelist mit IP-Adressen oder Netzwerken zu beschränken.

> [!NOTE]
> Standardmäßig weisen Azure-VMs, die über das Azure-Portal bereitgestellt wurden, eine öffentliche IP-Adresse mit Anmeldezugriff auf.  
> 
> 


## <a name="deploy-the-solution"></a>Bereitstellen der Lösung

**Voraussetzungen** Sie müssen über eine lokale Infrastruktur verfügen, die bereits mit einer geeigneten Netzwerkappliance konfiguriert ist.

Führen Sie die folgenden Schritte aus, um die Lösung bereitzustellen.

1. Klicken Sie auf die Schaltfläche unten:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Warten Sie, bis der Link im Azure-Portal geöffnet wird, und gehen Sie dann folgendermaßen vor:
   * Der Name der **Ressourcengruppe** ist bereits in der Parameterdatei definiert. Wählen Sie also **Neu erstellen**, und geben Sie im Textfeld `ra-hybrid-er-rg` ein.
   * Wählen Sie im Dropdownfeld **Standort** die Region aus.
   * Lassen Sie die Textfelder für den **Vorlagenstamm-URI** bzw. **Parameterstamm-URI** unverändert.
   * Überprüfen Sie die allgemeinen Geschäftsbedingungen, und aktivieren Sie dann das Kontrollkästchen **Ich stimme den oben genannten Geschäftsbedingungen zu**.
   * Klicken Sie auf die Schaltfläche **Kaufen**.
3. Warten Sie, bis die Bereitstellung abgeschlossen ist.
4. Klicken Sie auf die Schaltfläche unten:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Warten Sie, bis der Link im Azure-Portal geöffnet wird, und gehen Sie dann folgendermaßen vor:
   * Wählen Sie im Abschnitt **Ressourcengruppe** die Option **Vorhandene verwenden** aus, und geben Sie im Textfeld `ra-hybrid-er-rg` ein.
   * Wählen Sie im Dropdownfeld **Standort** die Region aus.
   * Lassen Sie die Textfelder für den **Vorlagenstamm-URI** bzw. **Parameterstamm-URI** unverändert.
   * Überprüfen Sie die allgemeinen Geschäftsbedingungen, und aktivieren Sie dann das Kontrollkästchen **Ich stimme den oben genannten Geschäftsbedingungen zu**.
   * Klicken Sie auf die Schaltfläche **Kaufen**.
6. Warten Sie, bis die Bereitstellung abgeschlossen ist.


<!-- links -->
[forced-tuneling]: ../dmz/secure-vnet-hybrid.md
[highly-available-network-architecture]: ./expressroute-vpn-failover.md

[expressroute-technical-overview]: /azure/expressroute/expressroute-introduction
[expressroute-prereqs]: /azure/expressroute/expressroute-prerequisites
[configure-expressroute-routing]: /azure/expressroute/expressroute-howto-routing-arm
[sla-for-expressroute]: https://azure.microsoft.com/support/legal/sla/expressroute/v1_0/
[link-vnet-to-expressroute]: /azure/expressroute/expressroute-howto-linkvnet-arm
[ExpressRoute-provisioning]: /azure/expressroute/expressroute-workflows
[expressroute-introduction]: /azure/expressroute/expressroute-introduction
[expressroute-peering]: /azure/expressroute/expressroute-circuit-peerings
[expressroute-pricing]: https://azure.microsoft.com/pricing/details/expressroute/
[expressroute-limits]: /azure/azure-subscription-service-limits#networking-limits
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-architectures.vsdx
[er-circuit-parameters]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/parameters/expressRouteCircuit.parameters.json
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[0]: ./images/expressroute.png "Hybrid-Netzwerkarchitektur mit Azure ExpressRoute"
[1]: ../_images/guidance-hybrid-network-expressroute/figure2.png "Einsatz redundanter Router mit primären und sekundären ExpressRoute-Verbindungen"
[2]: ../_images/guidance-hybrid-network-expressroute/figure3.png "Hinzufügen von Sicherheitsgeräten zum lokalen Netzwerk"
[3]: ../_images/guidance-hybrid-network-expressroute/figure4.png "Einsatz der Tunnelerzwingung zum Überwachen des Internetdatenverkehrs"
[4]: ../_images/guidance-hybrid-network-expressroute/figure5.png "Ermitteln des ServiceKey einer ExpressRoute-Verbindung"  