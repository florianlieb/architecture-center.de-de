---
title: Auswählen einer Lösung zum Herstellen einer Verbindung zwischen einem lokalen Netzwerk und Azure
description: Dieser Artikel vergleicht Referenzarchitekturen zum Herstellen einer Verbindung zwischen einem lokalen Netzwerk und Azure.
author: telmosampaio
ms.date: 04/06/2017
ms.openlocfilehash: 274b9df1817632a7f3eaafa8bf02e965fdc3feea
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a>Auswählen einer Lösung zum Herstellen einer Verbindung zwischen einem lokalen Netzwerk und Azure

Dieser Artikel vergleicht Optionen zum Herstellen einer Verbindung zwischen einem lokalen Netzwerk und einem virtuellen Azure-Netzwerk (VNET). Für jede Option stellen wir eine Referenzarchitektur und eine zur Bereitstellung geeignete Lösung vor.

## <a name="vpn-connection"></a>VPN-Verbindung

Verwenden Sie ein virtuelles privates Netzwerk (VPN), um über einen IPSec-VPN-Tunnel eine Verbindung zwischen Ihrem lokalen Netzwerk und einem Azure-VNET herzustellen.

Diese Architektur eignet sich für hybride Anwendungen, bei denen wahrscheinlich nur wenig Datenverkehr zwischen der lokalen Hardware und der Cloud stattfindet. Sie ist auch dann geeignet, wenn Sie eine geringfügig erhöhte Latenz in Kauf nehmen können, um von der Flexibilität und der Verarbeitungsleistung der Cloud zu profitieren.

**Vorteile**

- Diese Architektur lässt sich leicht konfigurieren.

**Herausforderungen**

- Es ist ein lokales VPN-Gerät erforderlich.
- Microsoft garantiert zwar für jedes VPN-Gateway eine Verfügbarkeit von 99,9 %, diese SLA gilt aber nur für das VPN-Gateway, nicht für Ihre Netzwerkverbindung zum Gateway.
- Eine VPN-Verbindung über ein Azure VPN-Gateway unterstützt zurzeit eine Bandbreite von maximal 200 MBit/s. Möglicherweise müssen Sie Ihr virtuelles Azure-Netzwerk über mehrere VPN-Verbindungen hinweg partitionieren, wenn Sie damit rechnen, diesen Durchsatz zu überschreiten.

**[Weitere Informationen...][vpn]**

## <a name="azure-expressroute-connection"></a>Azure ExpressRoute-Verbindung

ExpressRoute-Verbindungen nutzen eine dedizierte private Verbindung über einen Drittanbieter für die Konnektivität. Die private Verbindung erweitert Ihr lokales Netzwerk auf Azure. 

Diese Architektur eignet sich für hybride Anwendungen, die umfangreiche geschäftskritische Workloads ausführen, für die ein hohes Maß an Skalierbarkeit erforderlich ist. 

**Vorteile**

- Es steht eine wesentlich höhere Bandbreite zur Verfügung – bis zu 10 GBit/s, je nach Konnektivitätsanbieter.
- Die Architektur unterstützt eine dynamische Skalierung der Bandbreite, um in Zeiträumen mit geringerer Nachfrage die Kosten zu senken. Allerdings steht diese Option nicht bei allen Konnektivitätsanbietern zur Verfügung.
- Möglicherweise erhält Ihre Organisation direkten Zugriff auf landesweite Clouds – dies hängt vom Konnektivitätsanbieter ab.
- Es gilt für die gesamte Verbindung eine Verfügbarkeits-SLA von 99,9 %.

**Herausforderungen**

- Die Einrichtung kann sehr komplex sein. Zum Erstellen einer ExpressRoute-Verbindung müssen Sie mit einem Drittanbieter für die Konnektivität zusammenarbeiten. Der Anbieter ist für die Bereitstellung der Netzwerkverbindung verantwortlich.
- Die Architektur erfordert lokale Router mit hoher Bandbreite.

**[Weitere Informationen...][expressroute]**

## <a name="expressroute-with-vpn-failover"></a>ExpressRoute mit VPN-Failover

Diese Option kombiniert die beiden vorherigen: Unter normalen Bedingungen wird eine ExpressRoute-Verbindung verwendet, aber im Fall eines Konnektivitätsverlusts in der ExpressRoute-Leitung erfolgt ein Failover auf eine VPN-Verbindung.

Diese Architektur eignet sich für hybride Anwendungen, die sowohl die höhere Bandbreite von ExpressRoute als auch Netzwerkkonnektivität mit hoher Verfügbarkeit benötigen. 

**Vorteile**

- Bei einem Ausfall der ExpressRoute-Leitung ist eine hohe Verfügbarkeit gewährleistet, auch wenn die Fallbackverbindung über ein Netzwerk mit geringerer Bandbreite erfolgt.

**Herausforderungen**

- Die Konfiguration ist komplex. Sie müssen sowohl eine VPN-Verbindung als auch eine ExpressRoute-Leitung einrichten.
- Es sind redundante Hardwarekomponenten (VPN-Geräte) und eine redundante Azure VPN-Gatewayverbindung erforderlich, wofür Gebühren anfallen.

**[Weitere Informationen...][expressroute-vpn-failover]**

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md