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
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a><span data-ttu-id="bae7d-103">Auswählen einer Lösung zum Herstellen einer Verbindung zwischen einem lokalen Netzwerk und Azure</span><span class="sxs-lookup"><span data-stu-id="bae7d-103">Choose a solution for connecting an on-premises network to Azure</span></span>

<span data-ttu-id="bae7d-104">Dieser Artikel vergleicht Optionen zum Herstellen einer Verbindung zwischen einem lokalen Netzwerk und einem virtuellen Azure-Netzwerk (VNET).</span><span class="sxs-lookup"><span data-stu-id="bae7d-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="bae7d-105">Für jede Option stellen wir eine Referenzarchitektur und eine zur Bereitstellung geeignete Lösung vor.</span><span class="sxs-lookup"><span data-stu-id="bae7d-105">We provide a reference architecture and a deployable solution for each option.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="bae7d-106">VPN-Verbindung</span><span class="sxs-lookup"><span data-stu-id="bae7d-106">VPN connection</span></span>

<span data-ttu-id="bae7d-107">Verwenden Sie ein virtuelles privates Netzwerk (VPN), um über einen IPSec-VPN-Tunnel eine Verbindung zwischen Ihrem lokalen Netzwerk und einem Azure-VNET herzustellen.</span><span class="sxs-lookup"><span data-stu-id="bae7d-107">Use a virtual private network (VPN) to connect your on-premises network with an Azure VNet through an IPSec VPN tunnel.</span></span>

<span data-ttu-id="bae7d-108">Diese Architektur eignet sich für hybride Anwendungen, bei denen wahrscheinlich nur wenig Datenverkehr zwischen der lokalen Hardware und der Cloud stattfindet. Sie ist auch dann geeignet, wenn Sie eine geringfügig erhöhte Latenz in Kauf nehmen können, um von der Flexibilität und der Verarbeitungsleistung der Cloud zu profitieren.</span><span class="sxs-lookup"><span data-stu-id="bae7d-108">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

<span data-ttu-id="bae7d-109">**Vorteile**</span><span class="sxs-lookup"><span data-stu-id="bae7d-109">**Benefits**</span></span>

- <span data-ttu-id="bae7d-110">Diese Architektur lässt sich leicht konfigurieren.</span><span class="sxs-lookup"><span data-stu-id="bae7d-110">Simple to configure.</span></span>

<span data-ttu-id="bae7d-111">**Herausforderungen**</span><span class="sxs-lookup"><span data-stu-id="bae7d-111">**Challenges**</span></span>

- <span data-ttu-id="bae7d-112">Es ist ein lokales VPN-Gerät erforderlich.</span><span class="sxs-lookup"><span data-stu-id="bae7d-112">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="bae7d-113">Microsoft garantiert zwar für jedes VPN-Gateway eine Verfügbarkeit von 99,9 %, diese SLA gilt aber nur für das VPN-Gateway, nicht für Ihre Netzwerkverbindung zum Gateway.</span><span class="sxs-lookup"><span data-stu-id="bae7d-113">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this SLA only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="bae7d-114">Eine VPN-Verbindung über ein Azure VPN-Gateway unterstützt zurzeit eine Bandbreite von maximal 200 MBit/s.</span><span class="sxs-lookup"><span data-stu-id="bae7d-114">A VPN connection over Azure VPN Gateway currently supports a maximum of 200 Mbps bandwidth.</span></span> <span data-ttu-id="bae7d-115">Möglicherweise müssen Sie Ihr virtuelles Azure-Netzwerk über mehrere VPN-Verbindungen hinweg partitionieren, wenn Sie damit rechnen, diesen Durchsatz zu überschreiten.</span><span class="sxs-lookup"><span data-stu-id="bae7d-115">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

<span data-ttu-id="bae7d-116">**[Weitere Informationen...][vpn]**</span><span class="sxs-lookup"><span data-stu-id="bae7d-116">**[Read more...][vpn]**</span></span>

## <a name="azure-expressroute-connection"></a><span data-ttu-id="bae7d-117">Azure ExpressRoute-Verbindung</span><span class="sxs-lookup"><span data-stu-id="bae7d-117">Azure ExpressRoute connection</span></span>

<span data-ttu-id="bae7d-118">ExpressRoute-Verbindungen nutzen eine dedizierte private Verbindung über einen Drittanbieter für die Konnektivität.</span><span class="sxs-lookup"><span data-stu-id="bae7d-118">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="bae7d-119">Die private Verbindung erweitert Ihr lokales Netzwerk auf Azure.</span><span class="sxs-lookup"><span data-stu-id="bae7d-119">The private connection extends your on-premises network into Azure.</span></span> 

<span data-ttu-id="bae7d-120">Diese Architektur eignet sich für hybride Anwendungen, die umfangreiche geschäftskritische Workloads ausführen, für die ein hohes Maß an Skalierbarkeit erforderlich ist.</span><span class="sxs-lookup"><span data-stu-id="bae7d-120">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span> 

<span data-ttu-id="bae7d-121">**Vorteile**</span><span class="sxs-lookup"><span data-stu-id="bae7d-121">**Benefits**</span></span>

- <span data-ttu-id="bae7d-122">Es steht eine wesentlich höhere Bandbreite zur Verfügung – bis zu 10 GBit/s, je nach Konnektivitätsanbieter.</span><span class="sxs-lookup"><span data-stu-id="bae7d-122">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="bae7d-123">Die Architektur unterstützt eine dynamische Skalierung der Bandbreite, um in Zeiträumen mit geringerer Nachfrage die Kosten zu senken.</span><span class="sxs-lookup"><span data-stu-id="bae7d-123">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="bae7d-124">Allerdings steht diese Option nicht bei allen Konnektivitätsanbietern zur Verfügung.</span><span class="sxs-lookup"><span data-stu-id="bae7d-124">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="bae7d-125">Möglicherweise erhält Ihre Organisation direkten Zugriff auf landesweite Clouds – dies hängt vom Konnektivitätsanbieter ab.</span><span class="sxs-lookup"><span data-stu-id="bae7d-125">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="bae7d-126">Es gilt für die gesamte Verbindung eine Verfügbarkeits-SLA von 99,9 %.</span><span class="sxs-lookup"><span data-stu-id="bae7d-126">99.9% availability SLA across the entire connection.</span></span>

<span data-ttu-id="bae7d-127">**Herausforderungen**</span><span class="sxs-lookup"><span data-stu-id="bae7d-127">**Challenges**</span></span>

- <span data-ttu-id="bae7d-128">Die Einrichtung kann sehr komplex sein.</span><span class="sxs-lookup"><span data-stu-id="bae7d-128">Can be complex to set up.</span></span> <span data-ttu-id="bae7d-129">Zum Erstellen einer ExpressRoute-Verbindung müssen Sie mit einem Drittanbieter für die Konnektivität zusammenarbeiten.</span><span class="sxs-lookup"><span data-stu-id="bae7d-129">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="bae7d-130">Der Anbieter ist für die Bereitstellung der Netzwerkverbindung verantwortlich.</span><span class="sxs-lookup"><span data-stu-id="bae7d-130">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="bae7d-131">Die Architektur erfordert lokale Router mit hoher Bandbreite.</span><span class="sxs-lookup"><span data-stu-id="bae7d-131">Requires high-bandwidth routers on-premises.</span></span>

<span data-ttu-id="bae7d-132">**[Weitere Informationen...][expressroute]**</span><span class="sxs-lookup"><span data-stu-id="bae7d-132">**[Read more...][expressroute]**</span></span>

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="bae7d-133">ExpressRoute mit VPN-Failover</span><span class="sxs-lookup"><span data-stu-id="bae7d-133">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="bae7d-134">Diese Option kombiniert die beiden vorherigen: Unter normalen Bedingungen wird eine ExpressRoute-Verbindung verwendet, aber im Fall eines Konnektivitätsverlusts in der ExpressRoute-Leitung erfolgt ein Failover auf eine VPN-Verbindung.</span><span class="sxs-lookup"><span data-stu-id="bae7d-134">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="bae7d-135">Diese Architektur eignet sich für hybride Anwendungen, die sowohl die höhere Bandbreite von ExpressRoute als auch Netzwerkkonnektivität mit hoher Verfügbarkeit benötigen.</span><span class="sxs-lookup"><span data-stu-id="bae7d-135">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span> 

<span data-ttu-id="bae7d-136">**Vorteile**</span><span class="sxs-lookup"><span data-stu-id="bae7d-136">**Benefits**</span></span>

- <span data-ttu-id="bae7d-137">Bei einem Ausfall der ExpressRoute-Leitung ist eine hohe Verfügbarkeit gewährleistet, auch wenn die Fallbackverbindung über ein Netzwerk mit geringerer Bandbreite erfolgt.</span><span class="sxs-lookup"><span data-stu-id="bae7d-137">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

<span data-ttu-id="bae7d-138">**Herausforderungen**</span><span class="sxs-lookup"><span data-stu-id="bae7d-138">**Challenges**</span></span>

- <span data-ttu-id="bae7d-139">Die Konfiguration ist komplex.</span><span class="sxs-lookup"><span data-stu-id="bae7d-139">Complex to configure.</span></span> <span data-ttu-id="bae7d-140">Sie müssen sowohl eine VPN-Verbindung als auch eine ExpressRoute-Leitung einrichten.</span><span class="sxs-lookup"><span data-stu-id="bae7d-140">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="bae7d-141">Es sind redundante Hardwarekomponenten (VPN-Geräte) und eine redundante Azure VPN-Gatewayverbindung erforderlich, wofür Gebühren anfallen.</span><span class="sxs-lookup"><span data-stu-id="bae7d-141">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

<span data-ttu-id="bae7d-142">**[Weitere Informationen...][expressroute-vpn-failover]**</span><span class="sxs-lookup"><span data-stu-id="bae7d-142">**[Read more...][expressroute-vpn-failover]**</span></span>

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md