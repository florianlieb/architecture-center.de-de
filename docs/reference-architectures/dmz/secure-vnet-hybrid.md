---
title: Implementieren einer sicheren hybriden Netzwerkarchitektur in Azure
description: Implementieren einer sicheren hybriden Netzwerkarchitektur in Azure
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.prev: ./index
pnp.series.next: secure-vnet-dmz
cardTitle: DMZ between Azure and on-premises
ms.openlocfilehash: 81dea2e4439d5a01ebb88ab86dc0a59609bb7bc3
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="dmz-between-azure-and-your-on-premises-datacenter"></a>DMZ zwischen Azure und Ihrem lokalen Datencenter

Diese Referenzarchitektur zeigt ein sicheres Hybridnetzwerk, das ein lokales Netzwerk in Azure erweitert. Die Architektur implementiert eine DMZ, auch als *Umkreisnetzwerk* bezeichnet, zwischen dem lokalen Netzwerk und einem virtuellen Azure-Netzwerk (VNet). Die DMZ umfasst virtuelle Netzwerkgeräte (Network Virtual Appliances, NVAs), die Sicherheitsfunktionen wie z.B. Firewalls und Paketüberprüfung implementieren. Der gesamte ausgehende Datenverkehr vom VNet wird zwangsweise durch das lokale Netzwerk in das Internet getunnelt, sodass er überwacht werden kann.

[![0]][0] 

*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*

Für diese Architektur ist eine Verbindung mit Ihrem lokalen Rechenzentrum erforderlich, entweder mithilfe eines [VPN-Gateways][ra-vpn] oder einer [ExpressRoute][ra-expressroute]-Verbindung. Typische Einsatzmöglichkeiten für diese Architektur sind:

* Hybridanwendungen, in denen Workloads teilweise lokal und teilweise in Azure ausgeführt werden.
* Infrastruktur, die eine präzise Kontrolle des aus einem lokalen Rechenzentrum in ein Azure VNet eingehenden Verkehrs erfordert.
* Anwendungen, die ausgehenden Verkehr überwachen müssen. Dies ist ein oftmals auf behördliche Bestimmungen zurückgehendes Erfordernis vieler kommerzieller Systeme und kann dabei helfen, die Offenlegung privater Informationen zu verhindern.

## <a name="architecture"></a>Architecture

Die Architektur umfasst die folgenden Komponenten.

* **Lokales Netzwerk**. Ein privates lokales Netzwerk innerhalb einer Organisation.
* **Azure Virtual Network (VNet)**. Das VNet hostet die Anwendung und weitere Ressourcen, die in Azure ausgeführt werden.
* **Gateway**. Das Gateway stellt Verbindungen zwischen den Routern im lokalen Netzwerk und dem VNet bereit.
* **Virtuelles Netzwerkgerät**. Virtuelles Netzwerkgerät ist ein generischer Begriff zur Beschreibung eines virtuellen Computers, der Aufgaben wie das Erteilen oder Ablehnen von Zugriff als Firewall, das Optimieren von WAN-Vorgängen (Wide Area Network) (einschließlich Netzwerkkomprimierung), benutzerdefiniertes Routing oder andere Netzwerkfunktionen ausführt.
* **Subnetze für die Webschicht, die Geschäftsschicht und die Datenschicht**. Subnetze, in denen die VMs und Dienste gehostet sind, die eine 3-schichtige Beispielanwendung implementieren, die in der Cloud ausgeführt wird. Weitere Informationen finden Sie unter [Ausführen von Windows-VMs für eine n-schichtige Architektur in Azure][ra-n-tier].
* **Benutzerdefinierte Routen (UDR)**. [Benutzerdefinierte Routen][udr-overview] definieren den Fluss von IP-Datenverkehr innerhalb von Azure-VNets.

    > [!NOTE]
    > Je nach den Anforderungen Ihrer VPN-Verbindung können Sie BGP-Routen (Border Gateway Protocol) anstelle von UDRs konfigurieren, um die Weiterleitungsregeln zu implementieren, die Datenverkehr zurück durch das lokale Netzwerk leiten.
    > 
    > 

* **Verwaltungssubnetz** Dieses Subnetz enthält VMs, die Verwaltungs- und Überwachungsfunktionen für die Komponenten implementieren, die im VNet ausgeführt werden.

## <a name="recommendations"></a>Empfehlungen

Die folgenden Empfehlungen gelten für die meisten Szenarios. Sofern Sie keine besonderen Anforderungen haben, die Vorrang haben, sollten Sie diese Empfehlungen befolgen. 

### <a name="access-control-recommendations"></a>Empfehlungen für die Zugriffssteuerung

Verwenden Sie [rollenbasierte Zugriffssteuerung][rbac] (Role-Based Access Control, RBAC), um die Ressourcen in Ihrer Anwendung zu verwalten. Ziehen Sie die Erstellung der folgenden [benutzerdefinierten Rollen][rbac-custom-roles] in Erwägung:

- Eine DevOps-Rolle mit Berechtigungen zum Verwalten der Infrastruktur für die Anwendung, zum Bereitstellen der Anwendungskomponenten und zum Überwachen und Neustarten von VMs.  

- Eine zentrale IT-Administratorrolle zum Verwalten und Überwachen von Netzwerkressourcen.

- Eine IT-Sicherheitsadministratorrolle zum Verwalten der sicheren Netzwerkressourcen, wie etwa der NVAs. 

Die DevOps- und IT-Administratorrollen sollten keinen Zugriff auf die NVA-Ressourcen besitzen. Dieser sollte auf die IT-Sicherheitsadministratorrolle beschränkt sein.

### <a name="resource-group-recommendations"></a>Empfehlungen für Ressourcengruppen

Azure-Ressourcen, wie etwa VMs, VNets und Lastenausgleichsmodule, können durch Zusammenfassung in Ressourcengruppen komfortabel verwaltet werden. Weisen Sie jeder Ressourcengruppe RBAC-Rollen zu, um den Zugriff einzuschränken.

Es empfiehlt sich, die folgenden Ressourcengruppen zu erstellen:

* Eine Ressourcengruppe, die das VNet (ohne die VMs), NSGs, und die Gatewayressourcen für die Verbindung mit dem lokalen Netzwerk enthält. Weisen Sie dieser Ressourcengruppe die zentrale IT-Administratorrolle zu.
* Eine Ressourcengruppe, die die VMs für die NVAs (einschließlich des Lastenausgleichsmoduls), die Jumpbox und anderen Verwaltungs-VMs und die UDR für das Gatewaysubnetz enthält, das den gesamten Datenverkehr zwangsweise durch die NVAs leitet. Weisen Sie dieser Ressourcengruppe die IT-Sicherheitsadministratorrolle zu.
* Getrennte Ressourcengruppen für jede Logikschicht, die das Lastenausgleichsmodul und die VMs enthalten. Beachten Sie, dass diese Ressourcengruppen nicht die Subnetze für die einzelnen Schichten enthalten sollten. Weisen Sie dieser Ressourcengruppe die DevOps-Rolle zu.

### <a name="virtual-network-gateway-recommendations"></a>Empfehlungen für das Gateway des virtuellen Netzwerks

Der lokale Netzwerkverkehr wird durch ein virtuelles Netzwerkgateway in das VNet übermittelt. Zu diesem Zweck empfiehlt sich ein [Azure VPN-Gateway][guidance-vpn-gateway] oder ein [Azure ExpressRoute-Gateway][guidance-expressroute].

### <a name="nva-recommendations"></a>Empfehlungen für virtuelle Netzwerkgeräte

Virtuelle Netzwerkgeräte stellen verschiedene Dienste zum Verwalten und Überwachen des Netzwerkverkehrs bereit. Im [Azure Marketplace][azure-marketplace-nva] werden verschiedene virtuelle Netzwerkgeräte von Drittanbietern angeboten, die Sie verwenden können. Wenn keins dieser Drittanbieter-NVAs Ihren Anforderungen entspricht, können Sie mithilfe von virtuellen Computern ein benutzerdefiniertes virtuelles Netzwerkgerät erstellen. 

Beispielsweise implementiert die Lösungsbereitstellung für diese Referenzarchitektur ein virtuelles Netzwerkgerät mit folgenden Funktionen in einem virtuellen Computer:

* Der Datenverkehr wird mithilfe von [IP-Weiterleitung][ip-forwarding] zu den NVA-Netzwerkschnittstellen (NICs) geroutet.
* Die Durchleitung von Datenverkehr durch das virtuelle Netzwerkgerät wird nur zugelassen, wenn dies angemessen ist. Alle NVA-VMs in der virtuellen Referenzarchitektur stellen einfache Linux-Router dar. Eingehender Datenverkehr kommt an der Netzwerkschnittstelle *eth0* an, und ausgehender Datenverkehr, der den in benutzerdefinierten Skripts definierten Regeln entspricht, wird durch die Netzwerkschnittstelle *eth1* versendet.
* Die NVAs können nur aus dem Verwaltungssubnetz konfiguriert werden. 
* Durch das Verwaltungssubnetz gerouteter Datenverkehr durchläuft die NVAs nicht. Andernfalls würde beim Ausfall der NVAs keine Route mehr zu ihrer Instandsetzung bestehen.  
* Die VMs für das NVA werden in einer [Verfügbarkeitsgruppe][availability-set] hinter einem Lastenausgleichsmodul platziert. Die UDR im Gatewaysubnetz leitet NVA-Anforderungen an das Lastenausgleichsmodul.

Schließen Sie ein Schicht-7-NVA ein, um Anwendungsverbindungen auf der NVA-Ebene zu beenden und die Affinität mit den Back-End-Schichten aufrechtzuerhalten. Dies garantiert symmetrische Konnektivität, bei der Antwortdatenverkehr von den Back-End-Schichten durch das NVA zurückgegeben wird.

Eine andere Möglichkeit besteht darin, mehrere NVAs in Serie zu verbinden, wobei jedes NVA eine spezialisierte Sicherheitsaufgabe ausführt. Dadurch kann jede Sicherheitsfunktion getrennt auf NVA-Basis verwaltet werden. Beispielsweise kann ein NVA, das eine Firewall implementiert, in Serie mit einem NVA platziert werden, das Identitätsdienste ausführt. Die komfortable Verwaltung wird um den Preis zusätzlicher Netzwerkhops erkauft, die die Latenz erhöhten können – stellen Sie also sicher, dass sich dies nicht nachteilig auf die Leistung Ihrer Anwendung auswirkt.


### <a name="nsg-recommendations"></a>NSG-Empfehlungen

Das VPN-Gateway macht eine öffentliche IP-Adresse für die Verbindung mit dem lokalen Netzwerk verfügbar. Es wird empfohlen, eine Netzwerksicherheitsgruppe (NSG) für das eingehende NVA-Subnetz zu erstellen, mit Regeln zum Blockieren jeglichen Datenverkehrs, der nicht aus dem lokalen Netzwerk stammt.

Außerdem werden NSGs für jedes Subnetz empfohlen, um eine zweite Schutzebene vor eingehendem Verkehr bereitzustellen, der ein fehlerhaft konfiguriertes oder deaktiviertes NVA umgeht. Beispielsweise implementiert das Subnetz der Webschicht in der Referenzarchitektur eine NSG mit einer Regel zum Ignorieren aller Anforderungen, die nicht vom lokalen Netzwerk (192.168.0.0/16) oder vom VNet empfangen werden, und eine weitere Regel ignoriert alle Anforderungen, die nicht an Port 80 erfolgen.

### <a name="internet-access-recommendations"></a>Empfehlungen zum Zugriff auf das Internet

[Zwangsweises Tunneln][azure-forced-tunneling] des gesamten ausgehenden Internetdatenverkehrs durch Ihr lokales Netzwerk mithilfe des Site-zu-Site-VPN-Tunnels und Routen in das Internet mithilfe von Netzwerkadressenübersetzung (Network Address Translation, NAT). Dadurch wird die versehentliche Offenlegung jeglicher vertraulicher Informationen verhindert, die in Ihrer Datenschicht gespeichert sind, und zugleich die Untersuchung und Überwachung des gesamten ausgehenden Verkehrs ermöglicht.

> [!NOTE]
> Blockieren Sie nicht den gesamten Internetverkehr von den Logikschichten, da diese Schichten dadurch auch an der Verwendung von Azure PaaS-Diensten gehindert werden, die auf öffentlichen IP-Adressen aufbauen, wie etwa VM-Diagnoseprotokollierung, Herunterladen von VM-Erweiterungen und weitere Funktionen. Für Azure-Diagnose ist außerdem erforderlich, dass Komponenten Lese- und Schreibzugriff auf ein Azure Storage-Konto besitzen.
> 
> 

Überprüfen Sie, ob der ausgehende Internetdatenverkehr ordnungsgemäß zwangsweise getunnelt wird. Wenn Sie eine VPN-Verbindung mit dem [Routing- und RAS-Dienst][routing-and-remote-access-service] auf einem lokalen Server verwenden, setzen Sie ein Tool wie [WireShark][wireshark] oder [Microsoft Message Analyzer](https://www.microsoft.com/download/details.aspx?id=44226) ein.

### <a name="management-subnet-recommendations"></a>Empfehlungen für das Verwaltungssubnetz

Das Verwaltungssubnetz enthält eine Jumpbox, die Verwaltungs- und Überwachungsfunktionen ausführt. Schränken Sie die Ausführung aller sicheren Verwaltungsaufgaben auf die Jumpbox ein.
 
Erstellen Sie für die Jumpbox keine öffentliche IP-Adresse. Erstellen Sie stattdessen eine Route für den Zugriff auf die Jumpbox durch das Eingangsgateway. Erstellen Sie NSG-Regeln, damit das Verwaltungssubnetz nur auf Anforderungen von der zugelassenen Route reagiert.

## <a name="scalability-considerations"></a>Überlegungen zur Skalierbarkeit

Die Referenzarchitektur verwendet ein Lastenausgleichsmodul, um lokalen Netzwerkverkehr zu einem Pool von NVA-Geräten zu leiten, die das Routen des Verkehrs übernehmen. Die NVAs sind in einer [Verfügbarkeitsgruppe][availability-set] platziert. Diese Auslegung ermöglicht es Ihnen, den Durchsatz der NVAs im zeitlichen Ablauf zu überwachen, und als Reaktion auf einen Anstieg der Last NVA-Geräte hinzuzufügen.

Das SKU-VPN-Standardgateway unterstützt den anhaltenden Durchsatz von bis zu 100 Mbit/s. Die SKU für hohe Leistung bietet bis zu 200 Mbit/s. Für größere Bandbreiten können Sie das Upgrade auf ein ExpressRoute-Gateway in Erwägung ziehen. ExpressRoute stellt bis zu 10 Gb/s Bandbreite mit geringerer Latenz als eine VPN-Verbindung zur Verfügung.

Weitere Informationen zur Skalierbarkeit von Azure-Gateways finden Sie im Abschnitt mit den Überlegungen zur Skalierbarkeit in [Implementieren einer sicheren Hybrid-Netzwerkarchitektur mit Azure und lokalem VPN][guidance-vpn-gateway-scalability] und [Implementieren einer Hybrid-Netzwerkarchitektur mit Azure ExpressRoute][guidance-expressroute-scalability].

## <a name="availability-considerations"></a>Überlegungen zur Verfügbarkeit

Wie bereits erwähnt, verwendet die Referenzarchitektur einen Pool von NVA-Geräten hinter einem Lastenausgleichsmodul. Das Lastenausgleichsmodul verwendet eine Integritätssonde zum Überwachen der einzelnen NVA und entfernt alle nicht reagierenden NVAs aus dem Pool.

Wenn Sie Azure ExpressRoute verwenden, um Konnektivität zwischen dem VNet und dem lokalen Netzwerk bereitzustellen, [konfigurieren Sie ein VPN-Gateway zum Bereitstellen von Failover][ra-vpn-failover] für den Fall, dass die ExpressRoute-Verbindung nicht verfügbar ist.

Spezifische Informationen zum Aufrechterhalten der Verfügbarkeit von VPN- und ExpressRoute-Verbindungen finden Sie in den Überlegungen zur Verfügbarkeit in [Implementieren einer sicheren Hybrid-Netzwerkarchitektur mit Azure und lokalem VPN][guidance-vpn-gateway-availability] und [Implementieren einer Hybrid-Netzwerkarchitektur mit Azure ExpressRoute][guidance-expressroute-availability]. 

## <a name="manageability-considerations"></a>Überlegungen zur Verwaltbarkeit

Die gesamte Anwendungs- und Ressourcenüberwachung sollte durch die Jumpbox im Verwaltungssubnetz erfolgen. Je nach Ihren Anwendungsanforderungen benötigen Sie möglicherweise zusätzliche Überwachungsressourcen im Verwaltungssubnetz. In diesem Fall sollte der Zugriff auf diese Ressourcen über die Jumpbox erfolgen.

Wenn keine Gatewaykonnektivität zwischen Ihrem lokalen Netzwerk und Azure besteht, können Sie weiterhin auf die Jumpbox zugreifen, indem Sie eine öffentliche IP-Adresse bereitstellen, diese der Jumpbox hinzufügen und sich dann remote über das Internet anmelden.

Das Subnetz jeder einzelnen Schicht in der Referenzarchitektur wird durch NSG-Regeln geschützt. Möglicherweise müssen Sie eine Regel zum Öffnen von Port 3389 für RDP-Zugriff (Remote Desktop Protocol) auf Windows-VMs oder Port 22 für SSH-Zugriff (Secure Shell) auf Linux-VMs erstellen. Andere Verwaltungs- und Überwachungstools erfordern möglicherweise Regeln zum Öffnen zusätzlicher Ports.

Wenn Sie die Konnektivität zwischen Ihrem lokalen Rechenzentrum und Azure mithilfe von ExpressRoute bereitstellen, verwenden Sie das [Azure Connectivity Toolkit (AzureCT)][azurect] für die Überwachung und Problembehandlung von Verbindungsproblemen.

Sie finden weitere Informationen speziell zum Überwachen und Verwalten von VPN- und ExpressRoute-Verbindungen in den Artikeln [Implementieren einer sicheren Hybrid-Netzwerkarchitektur mit Azure und lokalem VPN][guidance-vpn-gateway-manageability] und [Implementieren einer Hybrid-Netzwerkarchitektur mit Azure ExpressRoute][guidance-expressroute-manageability].

## <a name="security-considerations"></a>Sicherheitshinweise

Diese Referenzarchitektur implementiert mehrere Sicherheitsebenen.

### <a name="routing-all-on-premises-user-requests-through-the-nva"></a>Routen aller lokalen Benutzeranforderungen durch das NVA
Die UDR im Gatewaysubnetz blockiert alle Benutzeranfragen, die nicht aus dem lokalen Netzwerk stammen. Die UDR leitet zulässige Anfragen an die NVAs im privaten DMZ-Subnetz weiter, und diese Anfragen werden an die Anforderungen weitergeleitet, wenn sie aufgrund der NVA-Regeln zugelassen werden. Sie können der UDR andere Routen hinzufügen, achten Sie jedoch darauf, dass die NVAs nicht unabsichtlich umgangen werden und Verwaltungsdatenverkehr für das Verwaltungssubnetz dadurch nicht blockiert wird.

Das Lastenausgleichsmodul vor den NVAs fungiert außerdem als Sicherheitsgerät, indem Datenverkehr an Ports ignoriert wird, die in den Regeln für den Lastenausgleich nicht als offen definiert sind. Die Lastenausgleichsmodule in der Referenzarchitektur lauschen nur auf HTTP-Anforderungen an Port 80 und auf HTTPS-Anforderungen an Port 443. Dokumentieren Sie alle zusätzlichen Regeln, die Sie den Lastenausgleichsmodulen hinzufügen, und überwachen Sie den Datenverkehr, um sicherzustellen, dass keine Sicherheitsprobleme vorliegen.

### <a name="using-nsgs-to-blockpass-traffic-between-application-tiers"></a>Verwenden Sie NSGs, um den Datenverkehr zwischen Logikschichten zu blockieren/übergeben
Der Datenverkehr zwischen den Schichten wird mithilfe von NSGs eingeschränkt. Die Geschäftsschicht blockiert jeglichen Verkehr, der nicht aus der Webschicht stammt, und die Datenschicht blockiert jeglichen Verkehr, der nicht aus der Geschäftsschicht stammt. Wenn bei Ihnen das Erfordernis besteht, die NSG-Regeln zu erweitern, um einen breiteren Zugriff auf diese Schichten zuzulassen, wägen Sie diese Erfordernisse gegen die Sicherheitsrisiken ab. Jeder neue eingehende Kommunikationsweg stellt eine Gelegenheit für die zufällige oder absichtliche Offenlegung von Daten oder Beschädigung von Anwendungen dar.

### <a name="devops-access"></a>DevOps-Zugriff
Verwenden Sie [RBAC][rbac], um die Vorgänge einzuschränken, die DevOps in den einzelnen Schichten ausführen können. Verwenden Sie beim Erteilen von Berechtigungen den [Ansatz der geringsten Rechte][security-principle-of-least-privilege]. Protokollieren Sie alle Verwaltungsvorgänge, und führen Sie regelmäßig Überwachungen durch, um sicherzustellen, dass alle Konfigurationsänderungen auf Planung beruhen.

## <a name="solution-deployment"></a>Bereitstellung von Lösungen

Eine Bereitstellung für eine Referenzarchitektur, die diese Empfehlungen implementiert, steht auf [GitHub][github-folder] zur Verfügung. Die Referenzarchitektur kann gemäß den folgenden Anweisungen bereitgestellt werden:

1. Klicken Sie auf diese Schaltfläche:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-hybrid%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Nachdem der Link im Azure-Portal geöffnet wurde, müssen Sie Werte für einige Einstellungen eingeben:   
   * Der Name der **Ressourcengruppe** ist bereits in der Parameterdatei definiert. Wählen Sie also **Neu erstellen**, und geben Sie im Textfeld `ra-private-dmz-rg` ein.
   * Wählen Sie im Dropdownfeld **Standort** die Region aus.
   * Lassen Sie die Textfelder für den **Vorlagenstamm-URI** bzw. **Parameterstamm-URI** unverändert.
   * Überprüfen Sie die allgemeinen Geschäftsbedingungen, und aktivieren Sie dann das Kontrollkästchen **Ich stimme den oben genannten Geschäftsbedingungen zu**.
   * Klicken Sie auf die Schaltfläche **Kaufen**.
3. Warten Sie, bis die Bereitstellung abgeschlossen ist.
4. Die Parameterdateien enthalten einen hartcodierten Administratorbenutzernamen und das zugehörige Kennwort für alle VMs. Es wird dringend empfohlen, beides sofort zu ändern. Wählen Sie jeden virtuellen Computer in der Bereitstellung im Azure-Portal aus, und klicken Sie dann auf dem Blatt **Support und Problembehandlung** auf **Kennwort zurücksetzen**. Wählen Sie im Dropdownfeld **Modus** die Option **Kennwort zurücksetzen** und dann einen neuen **Benutzernamen** und ein **Kennwort** aus. Klicken Sie auf die Schaltfläche **Aktualisieren**, um Ihre Änderungen zu speichern.

## <a name="next-steps"></a>Nächste Schritte

* Erfahren Sie, wie Sie eine [DMZ zwischen Azure und dem Internet](secure-vnet-dmz.md) implementieren.
* Erfahren Sie, wie Sie eine [hoch verfügbare Hybrid-Netzwerkarchitektur][ra-vpn-failover] implementieren.
* Weitere Informationen zum Verwalten der Netzwerksicherheit mit Azure finden Sie unter [Microsoft-Clouddienste und Netzwerksicherheit][cloud-services-network-security].
* Ausführliche Informationen über den Schutz von Ressourcen in Azure finden Sie unter [Erste Schritte mit Microsoft Azure-Sicherheit][getting-started-with-azure-security]. 
* Weitere Details zur Behandlung von Sicherheitsrisiken über Azure-Gatewayverbindungen finden Sie unter [Implementieren einer sicheren Hybrid-Netzwerkarchitektur mit Azure und lokalem VPN][guidance-vpn-gateway-security] und [Implementieren einer Hybrid-Netzwerkarchitektur mit Azure ExpressRoute][guidance-expressroute-security].
  > 

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[azure-forced-tunneling]: https://azure.microsoft.com/en-gb/documentation/articles/vpn-gateway-forced-tunneling-rm/
[azure-marketplace-nva]: https://azuremarketplace.microsoft.com/marketplace/apps/category/networking
[cloud-services-network-security]: https://azure.microsoft.com/documentation/articles/best-practices-network-security/
[getting-started-with-azure-security]: /azure/security/azure-security-getting-started
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-hybrid
[guidance-expressroute]: ../hybrid-networking/expressroute.md
[guidance-expressroute-availability]: ../hybrid-networking/expressroute.md#availability-considerations
[guidance-expressroute-manageability]: ../hybrid-networking/expressroute.md#manageability-considerations
[guidance-expressroute-security]: ../hybrid-networking/expressroute.md#security-considerations
[guidance-expressroute-scalability]: ../hybrid-networking/expressroute.md#scalability-considerations
[guidance-vpn-gateway]: ../hybrid-networking/vpn.md
[guidance-vpn-gateway-availability]: ../hybrid-networking/vpn.md#availability-considerations
[guidance-vpn-gateway-manageability]: ../hybrid-networking/vpn.md#manageability-considerations
[guidance-vpn-gateway-scalability]: ../hybrid-networking/vpn.md#scalability-considerations
[guidance-vpn-gateway-security]: ../hybrid-networking/vpn.md#security-considerations
[ip-forwarding]: /azure/virtual-network/virtual-networks-udr-overview#ip-forwarding
[ra-expressroute]: ../hybrid-networking/expressroute.md
[ra-n-tier]: ../virtual-machines-windows/n-tier.md
[ra-vpn]: ../hybrid-networking/vpn.md
[ra-vpn-failover]: ../hybrid-networking/expressroute-vpn-failover.md
[rbac]: /azure/active-directory/role-based-access-control-configure
[rbac-custom-roles]: /azure/active-directory/role-based-access-control-custom-roles
[routing-and-remote-access-service]: https://technet.microsoft.com/library/dd469790(v=ws.11).aspx
[security-principle-of-least-privilege]: https://msdn.microsoft.com/library/hdb58b2f(v=vs.110).aspx#Anchor_1
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview
[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx
[wireshark]: https://www.wireshark.org/
[0]: ./images/dmz-private.png "Sichere Hybrid-Netzwerkarchitektur"
