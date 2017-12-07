---
title: "Verbinden eines lokalen Netzwerks mit Azure über ein VPN"
description: "Erläutert, wie Sie eine sichere Netzwerkarchitektur zwischen Standorten implementieren, die ein virtuelles Azure-Netzwerk und ein lokales Netzwerk umfasst, die über ein VPN verbunden sind."
author: RohitSharma-pnp
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute
pnp.series.prev: ./index
cardTitle: VPN
ms.openlocfilehash: 66b2605c551148fadcdee6808c4e85940089f1e5
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a>Verbinden eines lokalen Netzwerks mit Azure über ein VPN-Gateway

Diese Referenzarchitektur zeigt, wie Sie ein lokales Netzwerk auf Azure ausdehnen, indem Sie ein VPN (virtuelles privates Netzwerk) zwischen Standorten verwenden. Der Datenverkehr zwischen dem lokalen Netzwerk und dem virtuellen Azure-Netzwerk (VNet) wird durch einen IPSec-VPN-Tunnel übertragen. [**Stellen Sie diese Lösung bereit**.](#deploy-the-solution)

![[0]][0]

*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*

## <a name="architecture"></a>Architektur 

Die Architektur umfasst die folgenden Komponenten.

* **Lokales Netzwerk**. Ein in einer Organisation betriebenes privates lokales Netzwerk.

* **VPN-Gerät**. Ein Gerät oder ein Dienst, das bzw. der externe Konnektivität mit dem lokalen Netzwerk bereitstellt. Bei dem VPN-Gerät kann es sich um ein Hardwaregerät oder eine Softwarelösung, z.B. den Routing- und RAS-Dienst unter Windows Server 2012, handeln. Eine Liste unterstützter VPN-Geräte und Informationen zu ihrer Konfiguration für die Verbindung mit einem Azure-VPN-Gateway finden Sie in den Anweisungen für das ausgewählte Gerät unter [Informationen zu VPN-Geräten für VPN-Gatewayverbindungen zwischen Standorten][vpn-appliance].

* **Virtuelles Netzwerk (VNet)**. Die Cloudanwendung und die Komponenten des Azure-VPN-Gateways befinden sich im selben [VNet][azure-virtual-network].

* **Azure-VPN-Gateway**. Der Dienst [VPN Gateway][azure-vpn-gateway] ermöglicht dem VNet, über ein VPN-Gerät eine Verbindung mit dem lokalen Netzwerk herzustellen. Weitere Informationen finden Sie unter [Verbinden eines lokalen Netzwerks mit einem Microsoft Azure Virtual Network][connect-to-an-Azure-vnet]. Das VPN-Gateway enthält die folgenden Elemente:
  
  * **Gateway des virtuellen Netzwerks**. Eine Ressource, die dem VNet ein virtuelles VPN-Gerät bereitstellt. Dieses ist für die Weiterleitung des Datenverkehrs vom lokalen Netzwerk zum VNet zuständig.
  * **Gateway des lokalen Netzwerks**. Eine Abstraktion des lokalen VPN-Geräts. Netzwerkdatenverkehr aus der Cloudanwendung zum lokalen Netzwerk wird durch dieses Gateway geleitet.
  * **Verbindung**. Die Verbindung verfügt über Eigenschaften, die den Verbindungstyp (IPSec) und den Schlüssel angeben, der für das lokale VPN-Gerät freigegeben wird, um Datenverkehr zu verschlüsseln.
  * **Gatewaysubnetz**. Das virtuelle Netzwerkgateway befindet sich in einem eigenen Subnetz, das verschiedenen Anforderungen unterliegt, die im Abschnitt „Empfehlungen“ weiter unten beschrieben werden.

* **Cloudanwendung**. Die in Azure gehostete Anwendung. Sie kann mehrere Schichten umfassen, wobei mehrere Subnetze über Azure Load Balancer verbunden sind. Weitere Informationen zur Anwendungsinfrastruktur finden Sie unter [Ausführen von Windows-VM-Workloads][windows-vm-ra] und [Ausführen von Linux-VM-Workloads][linux-vm-ra].

* **Interner Load Balancer**. Netzwerkverkehr aus dem VPN-Gateway wird über einen internen Load Balancer an die Cloudanwendung weitergeleitet. Der Load Balancer befindet sich im Front-End-Subnetz der Anwendung.

## <a name="recommendations"></a>Empfehlungen

Die folgenden Empfehlungen gelten für die meisten Szenarien. Sofern Sie keine besonderen Anforderungen haben, die Vorrang haben, sollten Sie diese Empfehlungen befolgen.

### <a name="vnet-and-gateway-subnet"></a>VNet und Gatewaysubnetz

Erstellen Sie ein Azure VNet mit einem Adressraum, der für alle erforderlichen Ressourcen ausreichend groß ist. Stellen Sie sicher, dass der VNet-Adressraum genügend Platz für Wachstum bietet, falls in Zukunft weitere VMs benötigt werden. Der Adressraum des VNet darf sich nicht mit dem des lokalen Netzwerks überschneiden. Im obigen Diagramm wird beispielsweise der Adressraum 10.20.0.0/16 für das VNet verwendet.

Erstellen Sie ein Subnetz mit dem Namen *GatewaySubnet* mit dem Adressbereich „/27“. Dieses Subnetz ist für das Gateway für virtuelle Netzwerke erforderlich. Durch Zuweisung von 32 Adressen zu diesem Subnetz wird eine künftige Überschreitung von Beschränkungen der Gatewaygröße verhindert. Vermeiden Sie außerdem, dieses Subnetz in der Mitte des Adressraums zu platzieren. Eine empfehlenswerte Vorgehensweise besteht darin, den Adressraum für das Gatewaysubnetz am oberen Ende des VNet-Adressraums festzulegen. In dem im Diagramm gezeigten Beispiel wird „10.20.255.224/27“ verwendet.  Es folgt eine schnelle Methode zur [CIDR]-Berechnung:

1. Legen Sie die variablen Bits im Adressraum des VNet bis zu den Bits, die vom Gatewaysubnetz verwendet werden, auf 1 und dann die restlichen Bits auf 0 fest.
2. Konvertieren Sie die resultierenden Bits in Dezimalzahlen, und drücken Sie sie als Adressraum aus, wobei die Präfixlänge auf die Größe des Subnetzes des Gateways festgelegt wird.

Beispielsweise ändert sich der IP-Adressbereich „10.20.0.0.0/16“ eines VNet nach Anwenden von Schritt 1 oben in „10.20.0b11111111.0b11100000“.  Nach der Konvertierung in Dezimalzahlen und deren Ausdruck als Adressraum ergibt sich „10.20.255.255.224/27“. 

> [!WARNING]
> Stellen Sie im Gatewaysubnetz keine VMs bereit. Weisen Sie außerdem diesem Subnetz keine Netzwerksicherheitsgruppe (NSG) zu, da das Gateway sonst nicht mehr funktioniert.
> 
> 

### <a name="virtual-network-gateway"></a>Gateway des virtuellen Netzwerks

Ordnen Sie dem Gateway für virtuelle Netzwerke eine öffentliche IP-Adresse zu.

Erstellen Sie das Gateway für virtuelle Netzwerke im Gatewaysubnetz, und weisen Sie ihm die neu zugeordnete öffentliche IP-Adresse zu. Verwenden Sie den Gatewaytyp, der Ihren Anforderungen am ehesten entspricht und der von Ihrem VPN-Gerät unterstützt wird:

- Erstellen Sie ein [richtlinienbasiertes Gateway ][policy-based-routing], wenn Sie die Weiterleitung von Anforderungen anhand von Richtlinienkriterien wie Adresspräfixen genau steuern müssen. Richtlinienbasierte Gateways arbeiten mit statischem Routing und funktionieren nur mit Verbindungen zwischen Standorten.

- Erstellen Sie ein [routenbasiertes Gateway][route-based-routing], wenn Sie eine Verbindung mit dem lokalen Netzwerk über RRAS herstellen, Verbindungen mit mehreren Standorten oder regionsübergreifende Verbinden unterstützen oder VNet-zu-VNet-Verbindungen implementieren (einschließlich Routen, die mehrere VNets durchqueren). Routenbasierte Gateways verwenden dynamisches Routing, um den Datenverkehr zwischen Netzwerken zu lenken. Sie können Ausfälle im Netzwerkpfad besser verkraften als statische Routen, weil sie alternative Routen ausprobieren können. Routenbasierte Gateways können auch den Verwaltungsaufwand reduzieren, da Routen nicht manuell aktualisiert werden müssen, wenn sich Netzwerkadressen ändern.

Eine Liste der unterstützten VPN-Geräte finden Sie unter [Informationen zu VPN-Geräten für VPN Gateway-Verbindungen zwischen Standorten][vpn-appliances].

> [!NOTE]
> Wenn Sie nach Erstellung des Gateways die Gatewaytypen ändern möchten, müssen Sie das Gateway löschen und neu erstellen.
> 
> 

Wählen Sie die Azure VPN Gateway SKU, die Ihren Durchsatzanforderungen am ehesten entspricht. Azure VPN Gateway ist in drei SKUs verfügbar, die in der folgenden Tabelle gezeigt werden. 

| SKU | VPN-Durchsatz | Max. IPsec-Tunnel |
| --- | --- | --- |
| Basic |100 MBit/s |10 |
| Standard |100 MBit/s |10 |
| High Performance |200 MBit/s |30 |

> [!NOTE]
> Die SKU „Basic“ ist nicht mit Azure ExpressRoute kompatibel. Sie können [die SKU ändern][changing-SKUs], nachdem das Gateway erstellt wurde.
> 
> 

Gebühren fallen basierend auf der Bereitstellungs- und Verfügbarkeitsdauer des Gateways an. Siehe [VPN Gateway – Preise][azure-gateway-charges].

Erstellen Sie Routingregeln für das Gatewaysubnetz, die eingehenden Anwendungsdatenverkehr vom Gateway zum internen Load Balancer leiten, anstatt Anforderungen direkt an die Anwendungs-VMs weiterleiten zu lassen.

### <a name="on-premises-network-connection"></a>Lokale Netzwerkverbindung

Erstellen Sie ein Gateway für das lokale Netzwerk. Geben Sie die öffentliche IP-Adresse des lokalen VPN-Geräts und den Adressraum des lokalen Netzwerks an. Beachten Sie, dass das lokale VPN-Gerät über eine öffentliche IP-Adresse verfügen muss, auf die das lokale Netzwerkgateway in Azure VPN Gateway zugreifen kann. Das VPN-Gerät darf sich nicht hinter einem Gerät für Netzwerkadressenübersetzung (Network Address Translation, NAT) befinden.

Richten Sie für das virtuelle und das lokale Netzwerkgateway eine Verbindung zwischen Standorten ein. Wählen Sie den Typ der Verbindung zwischen Standorten (IPSec) aus, und geben Sie den gemeinsam verwendeten Schlüssel an. Die Verschlüsselung zwischen Standorten mit dem Azure-VPN-Gateway basiert auf dem IPSec-Protokoll, wobei zur Authentifizierung vorinstallierte Schlüssel verwendet werden. Sie können den Schlüssel angeben, wenn Sie das Azure-VPN-Gateway erstellen. Sie müssen das VPN-Gerät, das lokal ausgeführt wird, mit demselben Schlüssel konfigurieren. Andere Authentifizierungsmechanismen werden derzeit nicht unterstützt.

Stellen Sie sicher, dass die lokale Routinginfrastruktur so konfiguriert ist, dass Anforderungen für Adressen im Azure VNet an das VPN-Gerät weitergeleitet werden.

Öffnen Sie im lokalen Netzwerk alle Ports, die von der Cloudanwendung benötigt werden.

Testen Sie die Verbindung, um zu überprüfen, ob:

* Das lokale VPN-Gerät den Datenverkehr über das Azure-VPN-Gateway ordnungsgemäß an die Cloudanwendung weiterleitet.
* Das VNet den Datenverkehr ordnungsgemäß zum lokalen Netzwerk zurückleitet.
* Unzulässiger Datenverkehr in beide Richtungen ordnungsgemäß blockiert wird.

## <a name="scalability-considerations"></a>Überlegungen zur Skalierbarkeit

Sie können eine begrenzte vertikale Skalierbarkeit erreichen, indem Sie von den VPN Gateway SKUs „Basic“ oder „Standard“ zur VPN SKU „High Performance“ wechseln.

Für VNets, die ein hohes Volumen an VPN-Datenverkehr erwarten, sollten Sie das Verteilen der verschiedenen Workloads in separate kleinere VNets und die Konfiguration eines VPN-Gateways für jedes einzelne davon in Betracht ziehen.

Das VNet kann entweder horizontal oder vertikal partitioniert werden. Verschieben Sie zum horizontalen Partitionieren einige VM-Instanzen von jeder Ebene in Subnetze des neuen VNet. Das Ergebnis ist, dass jedes VNet dieselbe Struktur und Funktionalität aufweist. Um eine vertikale Partitionierung vorzunehmen, müssen Sie jede Ebene neu gestalten, um die Funktionalität in verschiedene logische Bereiche zu unterteilen (z.B. Auftragsbearbeitung, Fakturierung, Kundenbetreuung usw.). Jeder Funktionsbereich kann dann in einem eigenen VNet platziert werden.

Die Replikation eines lokalen Active Directory-Domänencontrollers im VNet und die Implementierung von DNS im VNet kann dazu beitragen, einen Teil des sicherheitsrelevanten und administrativen Datenverkehrs zu reduzieren, der von den lokalen Systemen in die Cloud fließt. Weitere Informationen finden Sie unter [Erweitern von Active Directory Domain Services (AD DS) auf Azure][adds-extend-domain].

## <a name="availability-considerations"></a>Überlegungen zur Verfügbarkeit

Wenn Sie sicherstellen müssen, dass das lokale Netzwerk für das Azure-VPN-Gateway verfügbar bleibt, implementieren Sie für das lokale VPN-Gateway einen Failovercluster.

Wenn Ihre Organisation über mehrere lokale Standorte verfügt, richten Sie [Verbindungen dieser Standorte][vpn-gateway-multi-site] mit einem oder mehreren Azure VNets ein. Dieser Ansatz erfordert dynamisches (routenbasiertes) Routing. Stellen Sie daher sicher, dass das lokale VPN-Gateway dies unterstützt.

Weitere Informationen zu Vereinbarungen zum Servicelevel finden Sie unter [SLA für VPN Gateway][sla-for-vpn-gateway]. 

## <a name="manageability-considerations"></a>Überlegungen zur Verwaltbarkeit

Überwachen Sie von lokalen VPN-Geräten stammende Diagnoseinformationen. Dieser Vorgang hängt von den Funktionen ab, die das VPN-Gerät bietet. Wenn Sie z.B. den Routing- und RAS-Dienst unter Windows Server 2012 verwenden, wählen Sie die [RRAS-Protokollierung][rras-logging].

Verwenden Sie die [Azure VPN Gateway-Diagnose][gateway-diagnostic-logs] zum Erfassen von Informationen zu Verbindungsproblemen. Diese Protokolle können verwendet werden, um Informationen wie Quelle und Ziele von Verbindungsanforderungen nachzuverfolgen, welches Protokoll verwendet wurde und wie die Verbindung aufgebaut wurde (oder warum der Versuch fehlgeschlagen ist).

Überwachen Sie die Betriebsprotokolle des Azure-VPN-Gateways mithilfe der Überwachungsprotokolle, die im Azure-Portal verfügbar sind. Für das lokale Netzwerkgateway, das Azure-Netzwerkgateway und die Verbindung sind separate Protokolle verfügbar. Diese Informationen können verwendet werden, um jegliche Änderungen am Gateway nachzuverfolgen, und sind ggf. nützlich, wenn ein zuvor funktionierendes Gateway aus irgendeinem Grund nicht mehr funktioniert.

![[2]][2]

Überwachen Sie Verbindungen, und verfolgen Sie Verbindungsfehlerereignisse nach. Sie können ein Überwachungspaket wie [Nagios][nagios] verwenden, um diese Informationen zu erfassen und zu melden.

## <a name="security-considerations"></a>Sicherheitshinweise

Generieren Sie für jedes VPN-Gateway einen anderen gemeinsam verwendeten Schlüssel. Wählen Sie einen sicheren gemeinsam verwendeten Schlüssel, um Brute-Force-Angriffe abzuwehren.

> [!NOTE]
> Derzeit können Sie Azure Key Vault nicht für das Vorinstallieren von Schlüsseln für das Azure-VPN-Gateway nutzen.
> 
> 

Stellen Sie sicher, dass das lokale VPN-Gerät eine Verschlüsselungsmethode verwendet, die [mit dem Azure-VPN-Gateway kompatibel][vpn-appliance-ipsec] ist. Für richtlinienbasiertes Routing unterstützt das Azure-VPN-Gateway die Verschlüsselungsalgorithmen AES256, AES128 und 3DES. Routenbasierte Gateways unterstützen AES256 und 3DES.

Wenn sich Ihr lokales VPN-Gerät in einem Umkreisnetzwerk (DMZ) befindet, das eine Firewall zwischen dem Umkreisnetzwerk und dem Internet aufweist, müssen Sie möglicherweise [zusätzliche Firewallregeln][additional-firewall-rules] konfigurieren, um die VPN-Verbindung zwischen Standorten zuzulassen.

Wenn die Anwendung im VNet Daten an das Internet sendet, erwägen Sie [die Implementierung der Tunnelerzwingung][forced-tunneling], um den gesamten internetgebundenen Datenverkehr durch das lokale Netzwerk zu leiten. Auf diese Weise können Sie aus der lokalen Infrastruktur ausgehende Anforderungen der Anwendung überprüfen.

> [!NOTE]
> Die Tunnelerzwingung kann die Konnektivität mit Azure-Diensten (z.B. Azure Storage) und dem Windows-Lizenz-Manager beeinträchtigen.
> 
> 


## <a name="troubleshooting"></a>Problembehandlung 

Allgemeine Informationen zur Behandlung gängiger Fehler im Zusammenhang mit einem VPN finden Sie im Blogbeitrag [Troubleshooting common VPN related errors][troubleshooting-vpn-errors] (Problembehandlung bei gängigen VPN-bezogenen Fehlern).

Die folgenden Empfehlungen sind nützlich, um festzustellen, ob Ihr lokales VPN-Gerät ordnungsgemäß funktioniert.

- **Überprüfen Sie die vom VPN-Gerät generierten Protokolldateien auf Fehler oder Ausfälle.**

    Dies hilft Ihnen festzustellen, ob das VPN-Gerät ordnungsgemäß funktioniert. Der Speicherort dieser Informationen variiert abhängig von Ihrer Anwendung. Wenn Sie beispielsweise RRAS unter Windows Server 2012 verwenden, können Sie den folgenden PowerShell-Befehl verwenden, um Fehlerereignisinformationen für den RRAS-Dienst anzuzeigen:

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    Die *Message*-Eigenschaft jedes Eintrags enthält eine Beschreibung des Fehlers. Hier einige typische Beispiele:

        - Inability to connect, possibly due to an incorrect IP address specified for the Azure VPN gateway in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {41, 3, 0, 0}
        Index              : 14231
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: The network connection between your computer and
                             the VPN server could not be established because the remote server is not responding. This could
                             be because one of the network devices (for example, firewalls, NAT, routers, and so on) between your computer
                             and the remote server is not configured to allow VPN connections. Please contact your
                             Administrator or your service provider to determine which device may be causing the problem.
        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, The network connection between
                             your computer and the VPN server could not be established because the remote server is not
                             responding. This could be because one of the network devices (for example, firewalls, NAT, routers, and so on)
                             between your computer and the remote server is not configured to allow VPN connections. Please
                             contact your Administrator or your service provider to determine which device may be causing the
                             problem.}
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:26:02 PM
        TimeWritten        : 3/18/2016 1:26:02 PM
        UserName           :
        Site               :
        Container          :
        ```

        - The wrong shared key being specified in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {233, 53, 0, 0}
        Index              : 14245
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: Internet key exchange (IKE) authentication credentials are unacceptable.

        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, IKE authentication credentials are
                             unacceptable.
                             }
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:34:22 PM
        TimeWritten        : 3/18/2016 1:34:22 PM
        UserName           :
        Site               :
        Container          :
        ```

    Sie können auch Ereignisprotokollinformationen zu Verbindungsversuchen über den RRAS-Dienst mithilfe des folgenden PowerShell-Befehls abrufen: 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    Bei einem Verbindungsfehler enthält dieses Protokoll Fehler, die wie folgt aussehen:

    ```
    EventID            : 20227
    MachineName        : on-prem-vm
    Data               : {}
    Index              : 4203
    Category           : (0)
    CategoryNumber     : 0
    EntryType          : Error
    Message            : CoId={B4000371-A67F-452F-AA4C-3125AA9CFC78}: The user SYSTEM dialed a connection named
                         AzureGateway that has failed. The error code returned on failure is 809.
    Source             : RasClient
    ReplacementStrings : {{B4000371-A67F-452F-AA4C-3125AA9CFC78}, SYSTEM, AzureGateway, 809}
    InstanceId         : 20227
    TimeGenerated      : 3/18/2016 1:29:21 PM
    TimeWritten        : 3/18/2016 1:29:21 PM
    UserName           :
    Site               :
    Container          :
    ```

- **Überprüfen Sie die Konnektivität und Weiterleitung im gesamten VPN-Gateway.**

    Das VPN-Gerät leitet möglicherweise den Datenverkehr nicht ordnungsgemäß durch das Azure-VPN-Gateway weiter. Verwenden Sie ein Tool wie[PsPing][psping], um die Konnektivität und Weiterleitung im gesamten VPN-Gateway zu überprüfen. Um beispielsweise die Konnektivität eines lokalen Computers mit einem Webserver im VNet zu testen, führen Sie den folgenden Befehl aus (ersetzen Sie `<<web-server-address>>` durch die Adresse des Webservers):

    ```
    PsPing -t <<web-server-address>>:80
    ```

    Wenn der lokale Rechner Datenverkehr an den Webserver weiterleiten kann, sollten Sie eine Ausgabe ähnlich der folgenden sehen:

    ```
    D:\PSTools>psping -t 10.20.0.5:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.0.5:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.0.5:80 (warmup): 6.21ms
    Connecting to 10.20.0.5:80: 3.79ms
    Connecting to 10.20.0.5:80: 3.44ms
    Connecting to 10.20.0.5:80: 4.81ms

      Sent = 3, Received = 3, Lost = 0 (0% loss),
      Minimum = 3.44ms, Maximum = 4.81ms, Average = 4.01ms
    ```

    Wenn der lokale Computer nicht mit dem angegebenen Ziel kommunizieren kann, werden Meldungen wie diese angezeigt:

    ```
    D:\PSTools>psping -t 10.20.1.6:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.1.6:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.1.6:80 (warmup): This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80:
      Sent = 3, Received = 0, Lost = 3 (100% loss),
      Minimum = 0.00ms, Maximum = 0.00ms, Average = 0.00ms
    ```

- **Stellen Sie sicher, dass die lokale Firewall VPN-Datenverkehr passieren lässt und die richtigen Ports geöffnet sind.**

- **Prüfen Sie, ob das lokale VPN-Gerät eine Verschlüsselungsmethode verwendet, die [mit dem Azure-VPN-Gateway kompatibel][vpn-appliance] ist.** Für richtlinienbasiertes Routing unterstützt das Azure-VPN-Gateway die Verschlüsselungsalgorithmen AES256, AES128 und 3DES. Routenbasierte Gateways unterstützen AES256 und 3DES.

Die folgenden Empfehlungen sind nützlich, um festzustellen, ob ein Problem mit dem Azure-VPN-Gateway vorliegt:

- **Untersuchen Sie [Diagnoseprotokolle für das Azure-VPN-Gateway][gateway-diagnostic-logs] auf potenzielle Probleme.**

- **Stellen Sie sicher, dass das Azure-VPN-Gateway und das lokale VPN-Gerät mit demselben gemeinsam genutzten Authentifizierungsschlüssel konfiguriert sind.**

    Sie können den vom Azure-VPN-Gateway gespeicherten gemeinsamen Schlüssel mithilfe des folgenden Azure CLI-Befehls einsehen:

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    Verwenden Sie den Befehl entsprechend Ihrem lokalen VPN-Gerät, um den gemeinsam genutzten Schlüssel anzuzeigen, der für dieses Gerät konfiguriert wurde.

    Vergewissern Sie sich, dass das Subnetz *GatewaySubnet*, in dem sich das Azure-VPN-Gateway befindet, keiner Netzwerksicherheitsgruppe zugeordnet ist.

    Sie können die Details des Subnetzes mit dem folgenden Azure CLI-Befehl einsehen:

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    Stellen Sie sicher, dass es kein Datenfeld mit dem Namen *Network Security Group id* gibt. Das folgende Beispiel zeigt die Ergebnisse für eine Instanz von *GatewaySubnet*, der eine Netzwerksicherheitsgruppe (*VPN-Gateway-Group*) zugewiesen ist. Wenn für diese Netzwerksicherheitsgruppe Regeln definiert sind, kann dies das ordnungsgemäße Funktionieren des Gateways verhindern.

    ```
    C:\>azure network vnet subnet show -g profx-prod-rg -e profx-vnet -n GatewaySubnet
        info:    Executing command network vnet subnet show
        + Looking up virtual network "profx-vnet"
        + Looking up the subnet "GatewaySubnet"
        data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/virtualNetworks/profx-vnet/subnets/GatewaySubnet
        data:    Name                            : GatewaySubnet
        data:    Provisioning state              : Succeeded
        data:    Address prefix                  : 10.20.3.0/27
        data:    Network Security Group id       : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/networkSecurityGroups/VPN-Gateway-Group
        info:    network vnet subnet show command OK
    ```

- **Vergewissern Sie sich, dass die virtuellen Computer im Azure VNet so konfiguriert sind, dass sie Datenverkehr von außerhalb des VNet zulassen.**

    Überprüfen Sie jegliche NSG-Regeln, die mit Subnetzen verknüpft sind, die diese virtuellen Computer enthalten. Sie können alle NSG-Regeln mit dem folgenden Azure CLI-Befehl einsehen:

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- **Stellen Sie sicher, dass das Azure-VPN-Gateway verbunden ist.**

    Mit dem folgenden Azure PowerShell-Befehl können Sie den aktuellen Status der Azure-VPN-Verbindung überprüfen. Der Parameter `<<connection-name>>` ist der Name der Azure-VPN-Verbindung, die das virtuelle Netzwerkgateway mit dem lokalen Gateway verknüpft.

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    In den folgenden Codeausschnitte ist die Ausgabe hervorgehoben, die erzeugt wird, wenn das Gateway verbunden (das erste Beispiel) und getrennt (das zweite Beispiel) ist:

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : Connected
    EgressBytesTransferred     : 55254803
    IngressBytesTransferred    : 32227221
    ProvisioningState          : Succeeded
    ...
    ```

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection2 -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : NotConnected
    EgressBytesTransferred     : 0
    IngressBytesTransferred    : 0
    ProvisioningState          : Succeeded
    ...
    ```

Die folgenden Empfehlungen sind nützlich, um festzustellen, ob es Probleme mit der Host-VM-Konfiguration, der Auslastung von Netzwerkbandbreite oder der Anwendungsleistung gibt:

- **Überprüfen Sie, ob die Firewall im Gastbetriebssystem, die auf den Azure-VMs im Subnetz ausgeführt wird, ordnungsgemäß konfiguriert ist, um zulässigen Datenverkehr aus den lokalen IP-Bereichen zuzulassen.**

- **Stellen Sie sicher, dass das Datenverkehrsvolumen nicht nahe an der Grenze der Bandbreite liegt, die dem Azure-VPN-Gateway zur Verfügung steht.**

    Wie Sie dies überprüfen können, hängt vom VPN-Gerät ab, das lokal ausgeführt wird. Wenn Sie z.B. RRAS unter Windows Server 2012 verwenden, können Sie mit Systemmonitor die Menge der Daten verfolgen, die über die VPN-Verbindung empfangen und übertragen werden. Wählen Sie unter Verwendung des Objekts *RAS insgesamt* die Leistungsindikatoren *Empfangene Bytes/Sek.* und *Übertragene Bytes/Sek.*:

    ![[3]][3]

    Vergleichen Sie die Ergebnisse mit der Bandbreite, die dem VPN-Gateway zur Verfügung steht (100 Mbit/s für die SKUs „Basic“ und „Standard“ und 200 Mbit/s für die SKU „High Performance“):

    ![[4]][4]

- **Vergewissern Sie sich, dass Sie die richtige Anzahl und Größe von VMs für Ihre Anwendungslast bereitgestellt haben.**

    Stellen Sie fest, ob virtuelle Computer im Azure VNet langsam arbeiten. Falls ja, können sie überlastet sein, es können zu wenige sein, um die Last zu bewältigen, oder die Load Balancer sind möglicherweise nicht richtig konfiguriert. Um dies zu ermitteln, [müssen Sie Diagnoseinformationen aufzeichnen und analysieren][azure-vm-diagnostics]. Sie können die Ergebnisse im Azure-Portal einsehen. Es stehen aber auch viele Tools von Drittanbietern zur Verfügung, die detaillierte Einblicke in die Leistungsdaten ermöglichen.

- **Vergewissern Sie sich, dass die Anwendung Cloudressourcen effizient nutzt.**

    Instrumentieren Sie Anwendungscode, der auf jeder VM ausgeführt wird, um festzustellen, ob Anwendungen Ressourcen optimal nutzen. Dafür können Sie Tools wie [Application Insights][application-insights] verwenden.

## <a name="deploy-the-solution"></a>Bereitstellen der Lösung


**Voraussetzungen**. Sie müssen über eine lokale Infrastruktur verfügen, die bereits mit einem geeigneten Netzwerkgerät konfiguriert ist.

Führen Sie die folgenden Schritte aus, um die Lösung bereitzustellen.

1. Klicken Sie auf die Schaltfläche unten:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Warten Sie, bis der Link im Azure-Portal geöffnet wird, und gehen Sie dann folgendermaßen vor: 
   * Der Name der **Ressourcengruppe** ist bereits in der Parameterdatei definiert. Wählen Sie also **Neu erstellen**, und geben Sie im Textfeld `ra-hybrid-vpn-rg` ein.
   * Wählen Sie im Dropdownfeld **Standort** die Region aus.
   * Lassen Sie die Textfelder für den **Vorlagenstamm-URI** bzw. **Parameterstamm-URI** unverändert.
   * Überprüfen Sie die allgemeinen Geschäftsbedingungen, und aktivieren Sie dann das Kontrollkästchen **Ich stimme den oben genannten Geschäftsbedingungen zu**.
   * Klicken Sie auf die Schaltfläche **Kaufen**.
3. Warten Sie, bis die Bereitstellung abgeschlossen ist.



<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[expressroute]: ../hybrid-networking/expressroute.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[naming conventions]: /azure/guidance/guidance-naming-conventions

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[arm-templates]: /azure/resource-group-authoring-templates
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-portal]: /azure/azure-portal/resource-group-portal
[azure-powershell]: /azure/powershell-azure-resource-manager
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[network-security-group]: /azure/virtual-network/virtual-networks-nsg
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/v1_2/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[azure-vpn-gateway-diagnostics]: http://blogs.technet.com/b/keithmayer/archive/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell.aspx
[ping]: https://technet.microsoft.com/library/ff961503.aspx
[tracert]: https://technet.microsoft.com/library/ff961507.aspx
[psping]: http://technet.microsoft.com/sysinternals/jj729731.aspx
[nmap]: http://nmap.org
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: http://blogs.technet.com/b/keithmayer/archive/2015/12/07/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs.aspx
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[create-on-prem-network]: https://technet.microsoft.com/library/dn786406.aspx#routing
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[application-insights]: /azure/application-insights/app-insights-overview-usage
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
<!--[solution-script]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/Deploy-ReferenceArchitecture.ps1-->
<!--[solution-script-bash]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/deploy-reference-architecture.sh-->
<!--[virtualNetworkGateway-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/parameters/virtualNetworkGateway.parameters.json-->
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[0]: ./images/vpn.png "Hybrides Netzwerk, das sich über die lokale und Azure-Infrastruktur erstreckt"
[2]: ../_images/guidance-hybrid-network-vpn/audit-logs.png "Überwachungsprotokolle im Azure-Portal"
[3]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png "Leistungsindikatoren zum Überwachen von VPN-Netzwerkdatenverkehr"
[4]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png "Beispieldiagramm für die Leistung eines VPN-Netzwerks"