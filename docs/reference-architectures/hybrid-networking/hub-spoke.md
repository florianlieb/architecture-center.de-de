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
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="08558-103">Implementieren einer Hub-Spoke-Netzwerktopologie in Azure</span><span class="sxs-lookup"><span data-stu-id="08558-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="08558-104">Diese Referenzarchitektur zeigt, wie eine Hub-Spoke-Topologie in Azure implementiert wird.</span><span class="sxs-lookup"><span data-stu-id="08558-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="08558-105">Bei einem *Hub* handelt es sich um ein virtuelles Netzwerk (VNET) in Azure, das als zentraler Konnektivitätspunkt für Ihr lokales Netzwerk fungiert.</span><span class="sxs-lookup"><span data-stu-id="08558-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="08558-106">*Spokes* sind VNETs, die eine Peeringverbindung mit dem Hub herstellen und zur Isolierung von Workloads verwendet werden können.</span><span class="sxs-lookup"><span data-stu-id="08558-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="08558-107">Der Datenverkehr wird über eine ExpressRoute- oder VPN-Gatewayverbindung zwischen dem lokalen Rechenzentrum und dem Hub weitergeleitet.</span><span class="sxs-lookup"><span data-stu-id="08558-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="08558-108">[**Stellen Sie diese Lösung bereit**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="08558-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="08558-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="08558-109">![[0]][0]</span></span>

<span data-ttu-id="08558-110">*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*</span><span class="sxs-lookup"><span data-stu-id="08558-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="08558-111">Diese Topologie bietet unter anderem folgende Vorteile:</span><span class="sxs-lookup"><span data-stu-id="08558-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="08558-112">**Kosteneinsparungen** durch die Zentralisierung von Diensten, die von mehreren Workloads (z.B. Network Virtual Appliances, kurz NVAs, und DNS-Servern) an einem einzigen Standort gemeinsam genutzt werden können</span><span class="sxs-lookup"><span data-stu-id="08558-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="08558-113">**Umgehung von Abonnementbeschränkungen** durch die Herstellung von Peeringverbindungen zwischen VNETs verschiedener Abonnements und dem zentralen Hub</span><span class="sxs-lookup"><span data-stu-id="08558-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="08558-114">**Trennung der Belange** zwischen zentralen IT-Vorgängen (SecOps, InfraOps) und Workloads (DevOps)</span><span class="sxs-lookup"><span data-stu-id="08558-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="08558-115">Typische Einsatzmöglichkeiten für diese Architektur sind Folgende:</span><span class="sxs-lookup"><span data-stu-id="08558-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="08558-116">Workloads, die in verschiedenen Umgebungen wie Entwicklungs-, Test- und Produktionsumgebungen eingesetzt werden und gemeinsame Dienste wie DNS, IDS, NTP oder AD DS erfordern</span><span class="sxs-lookup"><span data-stu-id="08558-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="08558-117">Gemeinsame Dienste werden im Hub-VNET platziert, während die einzelnen Umgebungen in einem Spoke bereitgestellt werden, um die Isolation beizubehalten.</span><span class="sxs-lookup"><span data-stu-id="08558-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="08558-118">Workloads, bei denen keine Konnektivität untereinander bestehen muss, die jedoch Zugriff auf gemeinsame Dienste erfordern</span><span class="sxs-lookup"><span data-stu-id="08558-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="08558-119">Unternehmen, die auf eine zentrale Steuerung von Sicherheitsmechanismen (z.B. eine Firewall im Hub als DMZ) und eine getrennte Verwaltung von Workloads in den einzelnen Spokes angewiesen sind</span><span class="sxs-lookup"><span data-stu-id="08558-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="08558-120">Architektur</span><span class="sxs-lookup"><span data-stu-id="08558-120">Architecture</span></span>

<span data-ttu-id="08558-121">Diese Architektur besteht aus den folgenden Komponenten:</span><span class="sxs-lookup"><span data-stu-id="08558-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="08558-122">**Lokales Netzwerk**:</span><span class="sxs-lookup"><span data-stu-id="08558-122">**On-premises network**.</span></span> <span data-ttu-id="08558-123">Ein privates lokales Netzwerk innerhalb einer Organisation.</span><span class="sxs-lookup"><span data-stu-id="08558-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="08558-124">**VPN-Gerät**:</span><span class="sxs-lookup"><span data-stu-id="08558-124">**VPN device**.</span></span> <span data-ttu-id="08558-125">Ein Gerät oder ein Dienst, der externe Konnektivität mit dem lokalen Netzwerk bereitstellt.</span><span class="sxs-lookup"><span data-stu-id="08558-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="08558-126">Bei dem VPN-Gerät kann es sich um ein Hardwaregerät oder eine Softwarelösung wie den Routing- und RAS-Dienst (RRAS) unter Windows Server 2012 handeln.</span><span class="sxs-lookup"><span data-stu-id="08558-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="08558-127">Eine Liste der unterstützten VPN-Appliances und Informationen zur Konfiguration ausgewählter VPN-Geräte für die Verbindung mit Azure finden Sie unter [Informationen zu VPN-Geräten und IPsec-/IKE-Parametern für VPN-Gatewayverbindungen zwischen Standorten][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="08558-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="08558-128">**VPN-Gateway für ein virtuelles Netzwerk oder ExpressRoute-Gateway**:</span><span class="sxs-lookup"><span data-stu-id="08558-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="08558-129">Über das Gateway für virtuelle Netzwerke kann das VNET zur Konnektivität mit Ihrem lokalen Netzwerk eine Verbindung mit dem VPN-Gerät oder eine ExpressRoute-Verbindung herstellen.</span><span class="sxs-lookup"><span data-stu-id="08558-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="08558-130">Weitere Informationen finden Sie unter [Verbinden eines lokalen Netzwerks mit einem virtuellen Microsoft Azure-Netzwerk][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="08558-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="08558-131">In den Bereitstellungsskripts für diese Referenzarchitektur werden ein VPN-Gateway für die Konnektivität und ein VNET in Azure verwendet, um Ihr lokales Netzwerk zu simulieren.</span><span class="sxs-lookup"><span data-stu-id="08558-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="08558-132">**Hub-VNET**:</span><span class="sxs-lookup"><span data-stu-id="08558-132">**Hub VNet**.</span></span> <span data-ttu-id="08558-133">Das Azure-VNET, das als Hub in der Hub-Spoke-Topologie verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="08558-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="08558-134">Der Hub ist der zentrale Konnektivitätspunkt für Ihr lokales Netzwerk, mit dem Sie Dienste hosten, die von den verschiedenen in den Spoke-VNETs gehosteten Workloads genutzt werden können.</span><span class="sxs-lookup"><span data-stu-id="08558-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="08558-135">**Gatewaysubnetz**:</span><span class="sxs-lookup"><span data-stu-id="08558-135">**Gateway subnet**.</span></span> <span data-ttu-id="08558-136">Die Gateways für die virtuellen Netzwerke befinden sich im selben Subnetz.</span><span class="sxs-lookup"><span data-stu-id="08558-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="08558-137">**Subnetz für gemeinsame Dienste**:</span><span class="sxs-lookup"><span data-stu-id="08558-137">**Shared services subnet**.</span></span> <span data-ttu-id="08558-138">Ein Subnetz im Hub-VNET, das für das Hosten von Diensten verwendet wird, die von allen Spokes gemeinsam genutzt werden können (z.B. DNS oder AD DS).</span><span class="sxs-lookup"><span data-stu-id="08558-138">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="08558-139">**Spoke-VNETs**:</span><span class="sxs-lookup"><span data-stu-id="08558-139">**Spoke VNets**.</span></span> <span data-ttu-id="08558-140">Ein oder mehrere Azure-VNETs, die als Spokes in der Hub-Spoke-Topologie verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="08558-140">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="08558-141">Spokes können verwendet werden, um Workloads in ihren eigenen VNETs, die getrennt von anderen Spokes verwaltet werden, zu isolieren.</span><span class="sxs-lookup"><span data-stu-id="08558-141">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="08558-142">Jede Workload kann mehrere Schichten umfassen, wobei mehrere Subnetze über Azure Load Balancer verbunden sind.</span><span class="sxs-lookup"><span data-stu-id="08558-142">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="08558-143">Weitere Informationen zur Anwendungsinfrastruktur finden Sie unter [Ausführen von Windows-VM-Workloads][windows-vm-ra] und [Ausführen von Linux-VM-Workloads][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="08558-143">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="08558-144">**VNET-Peering**:</span><span class="sxs-lookup"><span data-stu-id="08558-144">**VNet peering**.</span></span> <span data-ttu-id="08558-145">Über eine [Peeringverbindung][vnet-peering] können zwei VNETs in derselben Azure-Region miteinander verbunden werden.</span><span class="sxs-lookup"><span data-stu-id="08558-145">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="08558-146">Peeringverbindungen sind nicht-transitive Verbindungen zwischen VNETs mit niedrigen Latenzen.</span><span class="sxs-lookup"><span data-stu-id="08558-146">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="08558-147">Sobald eine Peeringverbindung hergestellt wurde, tauschen die VNETs ohne Einsatz eines Routers Datenverkehr über den Azure-Backbone aus.</span><span class="sxs-lookup"><span data-stu-id="08558-147">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="08558-148">In einer Hub-Spoke-Netzwerktopologie wird durch VNET-Peering eine Verbindung zwischen dem Hub und den einzelnen Spokes hergestellt.</span><span class="sxs-lookup"><span data-stu-id="08558-148">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="08558-149">In diesem Artikel werden ausschließlich [Resource Manager](/azure/azure-resource-manager/resource-group-overview)-Bereitstellungen behandelt, Sie können jedoch auch eine Verbindung zwischen einem klassischen VNET und einem Resource Manager-VNET im selben Abonnement herstellen.</span><span class="sxs-lookup"><span data-stu-id="08558-149">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="08558-150">Auf diese Weise können Ihre Spokes klassische Bereitstellungen hosten und trotzdem von gemeinsamen Diensten im Hub profitieren.</span><span class="sxs-lookup"><span data-stu-id="08558-150">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>


## <a name="recommendations"></a><span data-ttu-id="08558-151">Recommendations</span><span class="sxs-lookup"><span data-stu-id="08558-151">Recommendations</span></span>

<span data-ttu-id="08558-152">Die folgenden Empfehlungen gelten für die meisten Szenarios.</span><span class="sxs-lookup"><span data-stu-id="08558-152">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="08558-153">Sofern Sie keine besonderen Anforderungen haben, die Vorrang haben, sollten Sie diese Empfehlungen befolgen.</span><span class="sxs-lookup"><span data-stu-id="08558-153">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="08558-154">Ressourcengruppen</span><span class="sxs-lookup"><span data-stu-id="08558-154">Resource groups</span></span>

<span data-ttu-id="08558-155">Das Hub-VNET und die einzelnen Spoke-VNETs können in verschiedenen Ressourcengruppen und sogar in verschiedenen Abonnements implementiert werden, sofern sie demselben Azure Active Directory-Mandanten (Azure AD) in der gleichen Azure-Region angehören.</span><span class="sxs-lookup"><span data-stu-id="08558-155">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="08558-156">Dies ermöglicht nicht nur eine dezentralisierte Verwaltung der einzelnen Workloads, sondern auch die Verwaltung gemeinsamer Dienste im Hub-VNET.</span><span class="sxs-lookup"><span data-stu-id="08558-156">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="08558-157">VNET und GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="08558-157">VNet and GatewaySubnet</span></span>

<span data-ttu-id="08558-158">Erstellen Sie ein Subnetz mit dem Namen *GatewaySubnet* mit dem Adressbereich /27.</span><span class="sxs-lookup"><span data-stu-id="08558-158">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="08558-159">Dieses Subnetz ist für das Gateway für virtuelle Netzwerke erforderlich.</span><span class="sxs-lookup"><span data-stu-id="08558-159">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="08558-160">Durch Zuweisung von 32 Adressen zu diesem Subnetz wird eine künftige Überschreitung von Beschränkungen der Gatewaygröße verhindert.</span><span class="sxs-lookup"><span data-stu-id="08558-160">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="08558-161">Weitere Informationen zum Einrichten des Gateways finden Sie abhängig von Ihrem Verbindungstyp in den folgenden Referenzarchitekturen:</span><span class="sxs-lookup"><span data-stu-id="08558-161">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="08558-162">[Hybrides Netzwerk mit ExpressRoute][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="08558-162">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="08558-163">[Hybrides Netzwerk mit einem VPN-Gateway][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="08558-163">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="08558-164">Um eine höhere Verfügbarkeit zu erzielen, können Sie ExpressRoute mit einem VPN für Failoverzwecke kombinieren.</span><span class="sxs-lookup"><span data-stu-id="08558-164">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="08558-165">Weitere Informationen finden Sie unter [Verbinden eines lokalen Netzwerks mit Azure unter Verwendung von ExpressRoute mit VPN-Failover][hybrid-ha].</span><span class="sxs-lookup"><span data-stu-id="08558-165">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="08558-166">Eine Hub-Spoke-Topologie kann auch ohne Gateway verwendet werden, wenn keine Konnektivität mit Ihrem lokalen Netzwerk erforderlich ist.</span><span class="sxs-lookup"><span data-stu-id="08558-166">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="08558-167">VNet-Peering</span><span class="sxs-lookup"><span data-stu-id="08558-167">VNet peering</span></span>

<span data-ttu-id="08558-168">Beim VNET-Peering wird eine nicht-transitive Beziehung zwischen zwei VNETs hergestellt.</span><span class="sxs-lookup"><span data-stu-id="08558-168">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="08558-169">Wenn Sie eine Verbindung zwischen Spokes herstellen möchten, sollten Sie eventuell eine separate Peeringverbindung zwischen diesen Spokes herstellen.</span><span class="sxs-lookup"><span data-stu-id="08558-169">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="08558-170">Wenn jedoch zwischen mehreren Spokes eine Verbindung hergestellt werden muss, werden die möglichen Peeringverbindungen sehr schnell zur Neige gehen, da [die Anzahl der VNET-Peerings pro VNET begrenzt ist][vnet-peering-limit].</span><span class="sxs-lookup"><span data-stu-id="08558-170">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="08558-171">In diesem Szenario sollten Sie die Verwendung von benutzerdefinierten Routen (User Defined Routes, UDRs) in Erwägung ziehen, um zu erzwingen, dass der für einen Spoke vorgesehene Datenverkehr an eine NVA, die als Router im Hub-VNET fungiert, gesendet wird.</span><span class="sxs-lookup"><span data-stu-id="08558-171">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="08558-172">Hierdurch können die Spokes miteinander verbunden werden.</span><span class="sxs-lookup"><span data-stu-id="08558-172">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="08558-173">Sie können Spokes auch für die Kommunikation mit Remotenetzwerken über das Gateway für das Hub-VNET konfigurieren.</span><span class="sxs-lookup"><span data-stu-id="08558-173">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="08558-174">Damit der Gatewaydatenverkehr zwischen den Spokes und dem Hub weitergeleitet und eine Verbindung mit Remotenetzwerken hergestellt werden kann, müssen Sie folgende Schritte durchführen:</span><span class="sxs-lookup"><span data-stu-id="08558-174">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="08558-175">Konfigurieren Sie die VNET-Peeringverbindung im Hub dahingehend, dass **Gatewaytransit zugelassen** wird.</span><span class="sxs-lookup"><span data-stu-id="08558-175">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="08558-176">Konfigurieren Sie die VNET-Peeringverbindung in den einzelnen Spokes dahingehend, dass **Remotegateways verwendet werden**.</span><span class="sxs-lookup"><span data-stu-id="08558-176">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="08558-177">Konfigurieren Sie alle VNET-Peeringverbindungen dahingehend, dass **weitergeleiteter Datenverkehr zugelassen** wird.</span><span class="sxs-lookup"><span data-stu-id="08558-177">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="08558-178">Überlegungen</span><span class="sxs-lookup"><span data-stu-id="08558-178">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="08558-179">Konnektivität zwischen Spokes</span><span class="sxs-lookup"><span data-stu-id="08558-179">Spoke connectivity</span></span>

<span data-ttu-id="08558-180">Wenn Konnektivität zwischen Spokes hergestellt werden muss, sollten Sie eine NVA für das Routing im Hub implementieren und UDRs im Spoke zur Weiterleitung des Datenverkehrs an den Hub verwenden.</span><span class="sxs-lookup"><span data-stu-id="08558-180">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="08558-181">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="08558-181">![[2]][2]</span></span>

<span data-ttu-id="08558-182">In diesem Szenario müssen Sie die Peeringverbindungen dahingehend konfigurieren, dass **weitergeleiteter Verkehr zugelassen** wird.</span><span class="sxs-lookup"><span data-stu-id="08558-182">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="08558-183">Umgehung von VNET-Peeringbeschränkungen</span><span class="sxs-lookup"><span data-stu-id="08558-183">Overcoming VNet peering limits</span></span>

<span data-ttu-id="08558-184">Stellen Sie sicher, dass Sie die [Begrenzung der Anzahl von VNET-Peerings pro VNET][vnet-peering-limit] in Azure einhalten.</span><span class="sxs-lookup"><span data-stu-id="08558-184">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="08558-185">Falls Sie mehr Spokes als die maximal zulässige Anzahl benötigen, sollten Sie eventuell eine Hub-Spoke-Hub-Spoke-Topologie erstellen, bei der die erste Ebene von Spokes auch als Hub fungiert.</span><span class="sxs-lookup"><span data-stu-id="08558-185">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="08558-186">Im folgenden Diagramm wird diese Vorgehensweise veranschaulicht.</span><span class="sxs-lookup"><span data-stu-id="08558-186">The following diagram shows this approach.</span></span>

<span data-ttu-id="08558-187">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="08558-187">![[3]][3]</span></span>

<span data-ttu-id="08558-188">Berücksichtigen Sie auch die Dienste, die im Hub gemeinsam genutzt werden, um sicherzustellen, dass sich der Hub für eine größere Anzahl von Spokes skalieren lässt.</span><span class="sxs-lookup"><span data-stu-id="08558-188">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="08558-189">Wenn Ihr Hub beispielsweise Firewalldienste bereitstellt, sollten Sie beim Hinzufügen mehrerer Spokes die Bandbreitenbeschränkungen Ihrer Firewalllösung berücksichtigen.</span><span class="sxs-lookup"><span data-stu-id="08558-189">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="08558-190">Es wird empfohlen, einige dieser gemeinsamen Dienste auf eine zweite Hubebene zu verlagern.</span><span class="sxs-lookup"><span data-stu-id="08558-190">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="08558-191">Bereitstellen der Lösung</span><span class="sxs-lookup"><span data-stu-id="08558-191">Deploy the solution</span></span>

<span data-ttu-id="08558-192">Eine Bereitstellung für diese Architektur ist auf [GitHub][ref-arch-repo] verfügbar.</span><span class="sxs-lookup"><span data-stu-id="08558-192">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="08558-193">Bei dieser werden zum Testen der Konnektivität Ubuntu-VMs in jedem VNET verwendet.</span><span class="sxs-lookup"><span data-stu-id="08558-193">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="08558-194">Es gibt keine tatsächlichen Dienste, die im Subnetz für **gemeinsame Dienste** im **Hub-VNET** gehostet werden.</span><span class="sxs-lookup"><span data-stu-id="08558-194">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="08558-195">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="08558-195">Prerequisites</span></span>

<span data-ttu-id="08558-196">Bevor Sie die Referenzarchitektur in Ihrem eigenen Abonnement bereitstellen können, müssen Sie die folgenden Schritte durchführen.</span><span class="sxs-lookup"><span data-stu-id="08558-196">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="08558-197">Klonen oder forken Sie das GitHub-Repository [AzureCAT-Referenzarchitekturen][ref-arch-repo], oder laden Sie die zugehörige ZIP-Datei herunter.</span><span class="sxs-lookup"><span data-stu-id="08558-197">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="08558-198">Wenn Sie stattdessen die Azure CLI verwenden möchten, vergewissern Sie sich, dass Azure CLI 2.0 auf Ihrem Computer installiert ist.</span><span class="sxs-lookup"><span data-stu-id="08558-198">If you prefer to use the Azure CLI, make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="08558-199">Um die CLI zu installieren, folgen Sie den Anweisungen unter [Installieren von Azure CLI 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="08558-199">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="08558-200">Wenn Sie PowerShell bevorzugen, stellen Sie sicher, dass auf Ihrem Computer das neueste PowerShell-Modul für Azure installiert ist.</span><span class="sxs-lookup"><span data-stu-id="08558-200">If you prefer to use PowerShell, make sure you have the latest PowerShell module for Azure installed on you computer.</span></span> <span data-ttu-id="08558-201">Um das neueste Azure PowerShell-Modul zu installieren, folgen Sie den Anweisungen unter [Installieren von PowerShell für Azure][azure-powershell].</span><span class="sxs-lookup"><span data-stu-id="08558-201">To install the latest Azure PowerShell module, follow the instructions in [Install PowerShell for Azure][azure-powershell].</span></span>

4. <span data-ttu-id="08558-202">Melden Sie sich über eine Eingabeaufforderung, eine bash-Eingabeaufforderung oder die PowerShell-Eingabeaufforderung bei Ihrem Azure-Konto an. Verwenden Sie dazu die unten aufgeführten Befehle, und befolgen Sie die Anweisungen.</span><span class="sxs-lookup"><span data-stu-id="08558-202">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="08558-203">Bereitstellen des simulierten lokalen Rechenzentrums</span><span class="sxs-lookup"><span data-stu-id="08558-203">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="08558-204">Führen Sie die folgenden Schritte durch, um das simulierte lokale Rechenzentrum als Azure-VNET bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="08558-204">To deploy the simulated on-premises datacenter as an Azure VNet, perform the following steps.</span></span>

1. <span data-ttu-id="08558-205">Navigieren Sie zum Ordner `hybrid-networking\hub-spoke\onprem` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.</span><span class="sxs-lookup"><span data-stu-id="08558-205">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="08558-206">Öffnen Sie die Datei `onprem.vm.parameters.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 11 und 12 an, und speichern Sie die Datei.</span><span class="sxs-lookup"><span data-stu-id="08558-206">Open the `onprem.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="08558-207">Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um die simulierte lokale Umgebung als VNET in Azure bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="08558-207">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="08558-208">Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.</span><span class="sxs-lookup"><span data-stu-id="08558-208">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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
  > <span data-ttu-id="08558-209">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `ra-onprem-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="08558-209">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="08558-210">Warten Sie, bis die Bereitstellung abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="08558-210">Wait for the deployment to finish.</span></span> <span data-ttu-id="08558-211">Diese Bereitstellung erstellt ein virtuelles Netzwerk, einen virtuellen Ubuntu-Computer und ein VPN-Gateway.</span><span class="sxs-lookup"><span data-stu-id="08558-211">This deployment creates a virtual network, a virtual machine running Ubuntu, and a VPN gateway.</span></span> <span data-ttu-id="08558-212">Die Erstellung des VPN-Gateways kann mehr als 40 Minuten in Anspruch nehmen.</span><span class="sxs-lookup"><span data-stu-id="08558-212">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="08558-213">Azure-Hub-VNET</span><span class="sxs-lookup"><span data-stu-id="08558-213">Azure hub VNet</span></span>

<span data-ttu-id="08558-214">Führen Sie die folgenden Schritte durch, um das Hub-VNET bereitzustellen und eine Verbindung mit dem oben erstellten simulierten lokalen VNET herzustellen.</span><span class="sxs-lookup"><span data-stu-id="08558-214">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="08558-215">Navigieren Sie zum Ordner `hybrid-networking\hub-spoke\hub` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.</span><span class="sxs-lookup"><span data-stu-id="08558-215">Navigate to the `hybrid-networking\hub-spoke\hub` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="08558-216">Öffnen Sie die Datei `hub.vm.parameters.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 11 und 12 an, und speichern Sie die Datei.</span><span class="sxs-lookup"><span data-stu-id="08558-216">Open the `hub.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="08558-217">Öffnen Sie die Datei `hub.gateway.parameters.json`, geben Sie einen gemeinsam verwendeten Schlüssel wie unten dargestellt zwischen den Anführungszeichen in Zeile 23 an, und speichern Sie die Datei.</span><span class="sxs-lookup"><span data-stu-id="08558-217">Open the `hub.gateway.parameters.json` file and enter a shared key between the quotes in line 23, as shown below, then save the file.</span></span> <span data-ttu-id="08558-218">Notieren Sie sich diesen Wert, da Sie ihn später für die Bereitstellung benötigen.</span><span class="sxs-lookup"><span data-stu-id="08558-218">Keep a note of this value, you will need to use it later in the deployment.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="08558-219">Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um die simulierte lokale Umgebung als VNET in Azure bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="08558-219">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="08558-220">Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.</span><span class="sxs-lookup"><span data-stu-id="08558-220">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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
  > <span data-ttu-id="08558-221">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `ra-hub-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="08558-221">If you decide to use a different resource group name (other than `ra-hub-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="08558-222">Warten Sie, bis die Bereitstellung abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="08558-222">Wait for the deployment to finish.</span></span> <span data-ttu-id="08558-223">Diese Bereitstellung erstellt ein virtuelles Netzwerk, einen virtuellen Ubuntu-Computer, ein VPN-Gateway und eine Verbindung mit dem im vorherigen Abschnitt erstellten Gateway.</span><span class="sxs-lookup"><span data-stu-id="08558-223">This deployment creates a virtual network, a virtual machine running Ubuntu, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="08558-224">Die Erstellung des VPN-Gateways kann mehr als 40 Minuten in Anspruch nehmen.</span><span class="sxs-lookup"><span data-stu-id="08558-224">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="connection-from-on-premises-to-the-hub"></a><span data-ttu-id="08558-225">Verbindung zwischen lokalen Rechenzentren und dem Hub</span><span class="sxs-lookup"><span data-stu-id="08558-225">Connection from on-premises to the hub</span></span>

<span data-ttu-id="08558-226">Führen Sie die folgenden Schritte durch, um eine Verbindung zwischen dem simulierten lokalen Rechenzentrum und dem Hub-VNET herzustellen.</span><span class="sxs-lookup"><span data-stu-id="08558-226">To connect from the simulated on-premises datacenter to the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="08558-227">Navigieren Sie zum Ordner `hybrid-networking\hub-spoke\onprem` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.</span><span class="sxs-lookup"><span data-stu-id="08558-227">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="08558-228">Öffnen Sie die Datei `onprem.connection.parameters.json`, geben Sie einen gemeinsam verwendeten Schlüssel wie unten dargestellt zwischen den Anführungszeichen in Zeile 9 an, und speichern Sie die Datei.</span><span class="sxs-lookup"><span data-stu-id="08558-228">Open the `onprem.connection.parameters.json` file and enter a shared key between the quotes in line 9, as shown below, then save the file.</span></span> <span data-ttu-id="08558-229">Der Wert dieses gemeinsam verwendeten Schlüssels muss mit dem des für das lokale, zuvor bereitgestellte Gateway verwendeten Schlüssels identisch sein.</span><span class="sxs-lookup"><span data-stu-id="08558-229">This shared key value must be the same used in the on-premises gateway you deployed previously.</span></span>

  ```bash
  "sharedKey": "",
  ```

3. <span data-ttu-id="08558-230">Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um die simulierte lokale Umgebung als VNET in Azure bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="08558-230">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="08558-231">Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.</span><span class="sxs-lookup"><span data-stu-id="08558-231">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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
  > <span data-ttu-id="08558-232">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `ra-onprem-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="08558-232">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="08558-233">Warten Sie, bis die Bereitstellung abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="08558-233">Wait for the deployment to finish.</span></span> <span data-ttu-id="08558-234">Diese Bereitstellung stellt eine Verbindung zwischen dem VNET für die Simulation eines lokalen Rechenzentrums und dem Hub-VNET her.</span><span class="sxs-lookup"><span data-stu-id="08558-234">This deployment creates a connection between the VNet used to simulate an on-premises datacenter, and the hub VNet.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="08558-235">Azure-Spoke-VNETs</span><span class="sxs-lookup"><span data-stu-id="08558-235">Azure spoke VNets</span></span>

<span data-ttu-id="08558-236">Führen Sie die folgenden Schritte durch, um die Spoke-VNETs bereitzustellen und eine Verbindung mit dem oben erstellten Hub-VNET herzustellen.</span><span class="sxs-lookup"><span data-stu-id="08558-236">To deploy the spoke VNets, and connect to the hub VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="08558-237">Wechseln Sie in den Ordner `hybrid-networking\hub-spoke\spokes` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.</span><span class="sxs-lookup"><span data-stu-id="08558-237">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="08558-238">Öffnen Sie die Datei `spoke1.web.parameters.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 53 und 54 an, und speichern Sie die Datei.</span><span class="sxs-lookup"><span data-stu-id="08558-238">Open the `spoke1.web.parameters.json` file and enter a username and password between the quotes in line 53 and 54, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="08558-239">Wiederholen Sie den vorherigen Schritt für die Datei `spoke2.web.parameters.json`.</span><span class="sxs-lookup"><span data-stu-id="08558-239">Repeat the previous step for file `spoke2.web.parameters.json`.</span></span>

4. <span data-ttu-id="08558-240">Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um den ersten Spoke bereitzustellen und ihn mit dem Hub zu verbinden.</span><span class="sxs-lookup"><span data-stu-id="08558-240">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="08558-241">Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.</span><span class="sxs-lookup"><span data-stu-id="08558-241">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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
  > <span data-ttu-id="08558-242">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `ra-spoke1-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="08558-242">If you decide to use a different resource group name (other than `ra-spoke1-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="08558-243">Warten Sie, bis die Bereitstellung abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="08558-243">Wait for the deployment to finish.</span></span> <span data-ttu-id="08558-244">Diese Bereitstellung erstellt ein virtuelles Netzwerk, ein Lastenausgleichsmodul mit drei virtuellen Computern, auf denen Ubuntu und Apache ausgeführt werden, sowie eine VNET-Peeringverbindung mit dem im vorherigen Abschnitt erstellten Hub-VNET.</span><span class="sxs-lookup"><span data-stu-id="08558-244">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="08558-245">Diese Bereitstellung kann mehr als 20 Minuten in Anspruch nehmen.</span><span class="sxs-lookup"><span data-stu-id="08558-245">This deployment may take over 20 minutes.</span></span>

6. <span data-ttu-id="08558-246">Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um den ersten Spoke bereitzustellen und ihn mit dem Hub zu verbinden.</span><span class="sxs-lookup"><span data-stu-id="08558-246">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="08558-247">Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.</span><span class="sxs-lookup"><span data-stu-id="08558-247">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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
  > <span data-ttu-id="08558-248">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `ra-spoke2-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="08558-248">If you decide to use a different resource group name (other than `ra-spoke2-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="08558-249">Warten Sie, bis die Bereitstellung abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="08558-249">Wait for the deployment to finish.</span></span> <span data-ttu-id="08558-250">Diese Bereitstellung erstellt ein virtuelles Netzwerk, ein Lastenausgleichsmodul mit drei virtuellen Computern, auf denen Ubuntu und Apache ausgeführt werden, sowie eine VNET-Peeringverbindung mit dem im vorherigen Abschnitt erstellten Hub-VNET.</span><span class="sxs-lookup"><span data-stu-id="08558-250">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="08558-251">Diese Bereitstellung kann mehr als 20 Minuten in Anspruch nehmen.</span><span class="sxs-lookup"><span data-stu-id="08558-251">This deployment may take over 20 minutes.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="08558-252">Herstellen einer Peeringverbindung zwischen Azure-Hub-VNETs und Spoke-VNETs</span><span class="sxs-lookup"><span data-stu-id="08558-252">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="08558-253">Führen Sie die folgenden Schritte durch, um VNET-Peeringverbindungen für das Hub-VNET herzustellen.</span><span class="sxs-lookup"><span data-stu-id="08558-253">To deploy the VNet peering connections for the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="08558-254">Wechseln Sie in den Ordner `hybrid-networking\hub-spoke\hub` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.</span><span class="sxs-lookup"><span data-stu-id="08558-254">Switch to the `hybrid-networking\hub-spoke\hub` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="08558-255">Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um die Peeringverbindung für den ersten Spoke herzustellen.</span><span class="sxs-lookup"><span data-stu-id="08558-255">Run the bash or PowerShell command below to deploy the peering connection to the first spoke.</span></span> <span data-ttu-id="08558-256">Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.</span><span class="sxs-lookup"><span data-stu-id="08558-256">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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

2. <span data-ttu-id="08558-257">Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um die Peeringverbindung für den zweiten Spoke herzustellen.</span><span class="sxs-lookup"><span data-stu-id="08558-257">Run the bash or PowerShell command below to deploy the peering connection to the second spoke.</span></span> <span data-ttu-id="08558-258">Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.</span><span class="sxs-lookup"><span data-stu-id="08558-258">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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

### <a name="test-connectivity"></a><span data-ttu-id="08558-259">Testen der Konnektivität</span><span class="sxs-lookup"><span data-stu-id="08558-259">Test connectivity</span></span>

<span data-ttu-id="08558-260">Führen Sie die folgenden Schritte durch, um zu überprüfen, ob die mit einer lokalen Rechenzentrumsbereitstellung verbundene Hub-Spoke-Topologie funktioniert.</span><span class="sxs-lookup"><span data-stu-id="08558-260">To verify that the hub-spoke topology connected to an on-premises datacenter deployment worked, follow these steps.</span></span>

1. <span data-ttu-id="08558-261">Stellen Sie über das [Azure-Portal][Portal] eine Verbindung mit Ihrem Abonnement her, und navigieren Sie in der Ressourcengruppe `ra-onprem-rg` zum virtuellen Computer `ra-onprem-vm1`.</span><span class="sxs-lookup"><span data-stu-id="08558-261">From the [Azure portal][portal], connect to your subscription, and navigate to the `ra-onprem-vm1` virtual machine in the `ra-onprem-rg` resource group.</span></span>

2. <span data-ttu-id="08558-262">Beachten Sie auf dem Blatt `Overview` `Public IP address` für die VM.</span><span class="sxs-lookup"><span data-stu-id="08558-262">In the `Overview` blade, note the `Public IP address` for the VM.</span></span>

3. <span data-ttu-id="08558-263">Stellen Sie über einen SSH-Client eine Verbindung mit der oben genannten IP-Adresse her, indem Sie den Benutzernamen und das Kennwort verwenden, den bzw. das Sie bei der Bereitstellung angegeben haben.</span><span class="sxs-lookup"><span data-stu-id="08558-263">Use an SSH client to connect to the IP address you noted above using the user name and password you specified during deployment.</span></span>

4. <span data-ttu-id="08558-264">Führen Sie in der Eingabeaufforderung in der VM, mit der Sie eine Verbindung hergestellt haben, den folgenden Befehl aus, um die Konnektivität zwischen dem lokalen VNET und dem VNET von Spoke 1 zu testen.</span><span class="sxs-lookup"><span data-stu-id="08558-264">From the command prompt on the VM you connected to, run the command below to test connectivity from the on-premises VNet to the Spoke1 VNet.</span></span>

  ```bash
  ping 10.1.1.37
  ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="08558-265">Herstellen von Konnektivität zwischen Spokes</span><span class="sxs-lookup"><span data-stu-id="08558-265">Add connectivity between spokes</span></span>

<span data-ttu-id="08558-266">Wenn zwischen Spokes eine Verbindung hergestellt werden soll, müssen Sie für jeden Spoke UDRs bereitstellen, der den für andere Spokes vorgesehenen Datenverkehr an das Gateway im Hub-VNET weiterleitet.</span><span class="sxs-lookup"><span data-stu-id="08558-266">If you want to allow spokes to connect to each other, you must deploy UDRs to each spoke that forward traffic destined to other spokes to the gateway in the hub VNet.</span></span> <span data-ttu-id="08558-267">Führen Sie die folgenden Schritte durch, um zu prüfen, dass Sie derzeit keine Verbindung zwischen Spokes herstellen können. Geben Sie anschließend die UDRs an, und testen Sie erneut die Konnektivität.</span><span class="sxs-lookup"><span data-stu-id="08558-267">Perform the following steps to verify that currently you are not able to connect from a spoke to another, then deploy the UDRs and test connectivity again.</span></span>

1. <span data-ttu-id="08558-268">Wiederholen Sie die Schritte 1 bis 4 oben, wenn Sie nicht mehr mit der Jumpbox-VM verbunden sind.</span><span class="sxs-lookup"><span data-stu-id="08558-268">Repeat steps 1 to 4 above, if you are not connected to the jumpbox VM any longer.</span></span>

2. <span data-ttu-id="08558-269">Stellen Sie eine Verbindung mit einem der Webserver in Spoke 1 her.</span><span class="sxs-lookup"><span data-stu-id="08558-269">Connect to one of the web servers in spoke 1.</span></span>

  ```bash
  ssh 10.1.1.37
  ```

3. <span data-ttu-id="08558-270">Testen Sie die Konnektivität zwischen Spoke 1 und Spoke 2.</span><span class="sxs-lookup"><span data-stu-id="08558-270">Test the connectivity between spoke 1 and spoke 2.</span></span> <span data-ttu-id="08558-271">Bei dem Test sollte ein Fehler auftreten.</span><span class="sxs-lookup"><span data-stu-id="08558-271">It should fail.</span></span>

  ```bash
  ping 10.1.2.37
  ```

4. <span data-ttu-id="08558-272">Kehren Sie zur Eingabeaufforderung Ihres Computers zurück.</span><span class="sxs-lookup"><span data-stu-id="08558-272">Switch back to your computer's command prompt.</span></span>

5. <span data-ttu-id="08558-273">Wechseln Sie in den Ordner `hybrid-networking\hub-spoke\spokes` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.</span><span class="sxs-lookup"><span data-stu-id="08558-273">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you downloaded in the pre-requisites step above.</span></span>

6. <span data-ttu-id="08558-274">Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um eine UDR für den ersten Spoke bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="08558-274">Run the bash or PowerShell command below to deploy an UDR to the first spoke.</span></span> <span data-ttu-id="08558-275">Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.</span><span class="sxs-lookup"><span data-stu-id="08558-275">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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

7. <span data-ttu-id="08558-276">Führen Sie den nachfolgenden bash- oder PowerShell-Befehl aus, um eine UDR für den zweiten Spoke bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="08558-276">Run the bash or PowerShell command below to deploy an UDR to the second spoke.</span></span> <span data-ttu-id="08558-277">Ersetzen Sie die Werte durch Ihr Abonnement, den jeweiligen Namen Ihrer Ressourcengruppe sowie Ihre Azure-Region.</span><span class="sxs-lookup"><span data-stu-id="08558-277">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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

8. <span data-ttu-id="08558-278">Kehren Sie zurück zum SSH-Terminal zurück.</span><span class="sxs-lookup"><span data-stu-id="08558-278">Switch back to the ssh terminal.</span></span>

9. <span data-ttu-id="08558-279">Testen Sie die Konnektivität zwischen Spoke 1 und Spoke 2.</span><span class="sxs-lookup"><span data-stu-id="08558-279">Test the connectivity between spoke 1 and spoke 2.</span></span> <span data-ttu-id="08558-280">Der Test sollte erfolgreich abgeschlossen werden.</span><span class="sxs-lookup"><span data-stu-id="08558-280">It should succeed.</span></span>

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
