---
title: Implementieren einer Hub-Spoke-Netzwerktopologie in Azure
description: Vorgehensweise zum Implementieren einer Hub-Spoke-Netzwerktopologie in Azure
author: telmosampaio
ms.date: 02/23/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 1a2855f0d4a903fc4d7a022aef20ea73fe763e2c
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/08/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="b2a6b-103">Implementieren einer Hub-Spoke-Netzwerktopologie in Azure</span><span class="sxs-lookup"><span data-stu-id="b2a6b-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="b2a6b-104">Diese Referenzarchitektur zeigt, wie eine Hub-Spoke-Topologie in Azure implementiert wird.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="b2a6b-105">Bei einem *Hub* handelt es sich um ein virtuelles Netzwerk (VNET) in Azure, das als zentraler Konnektivitätspunkt für Ihr lokales Netzwerk fungiert.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="b2a6b-106">*Spokes* sind VNETs, die eine Peeringverbindung mit dem Hub herstellen und zur Isolierung von Workloads verwendet werden können.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="b2a6b-107">Der Datenverkehr wird über eine ExpressRoute- oder VPN-Gatewayverbindung zwischen dem lokalen Rechenzentrum und dem Hub weitergeleitet.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="b2a6b-108">[**Stellen Sie diese Lösung bereit**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="b2a6b-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="b2a6b-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="b2a6b-109">![[0]][0]</span></span>

<span data-ttu-id="b2a6b-110">*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*</span><span class="sxs-lookup"><span data-stu-id="b2a6b-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="b2a6b-111">Diese Topologie bietet unter anderem folgende Vorteile:</span><span class="sxs-lookup"><span data-stu-id="b2a6b-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="b2a6b-112">**Kosteneinsparungen** durch die Zentralisierung von Diensten, die von mehreren Workloads (z.B. Network Virtual Appliances, kurz NVAs, und DNS-Servern) an einem einzigen Standort gemeinsam genutzt werden können</span><span class="sxs-lookup"><span data-stu-id="b2a6b-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="b2a6b-113">**Umgehung von Abonnementbeschränkungen** durch die Herstellung von Peeringverbindungen zwischen VNETs verschiedener Abonnements und dem zentralen Hub</span><span class="sxs-lookup"><span data-stu-id="b2a6b-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="b2a6b-114">**Trennung der Belange** zwischen zentralen IT-Vorgängen (SecOps, InfraOps) und Workloads (DevOps)</span><span class="sxs-lookup"><span data-stu-id="b2a6b-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="b2a6b-115">Typische Einsatzmöglichkeiten für diese Architektur sind Folgende:</span><span class="sxs-lookup"><span data-stu-id="b2a6b-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="b2a6b-116">Workloads, die in verschiedenen Umgebungen wie Entwicklungs-, Test- und Produktionsumgebungen eingesetzt werden und gemeinsame Dienste wie DNS, IDS, NTP oder AD DS erfordern</span><span class="sxs-lookup"><span data-stu-id="b2a6b-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="b2a6b-117">Gemeinsame Dienste werden im Hub-VNET platziert, während die einzelnen Umgebungen in einem Spoke bereitgestellt werden, um die Isolation beizubehalten.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="b2a6b-118">Workloads, bei denen keine Konnektivität untereinander bestehen muss, die jedoch Zugriff auf gemeinsame Dienste erfordern</span><span class="sxs-lookup"><span data-stu-id="b2a6b-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="b2a6b-119">Unternehmen, die auf eine zentrale Steuerung von Sicherheitsmechanismen (z.B. eine Firewall im Hub als DMZ) und eine getrennte Verwaltung von Workloads in den einzelnen Spokes angewiesen sind</span><span class="sxs-lookup"><span data-stu-id="b2a6b-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="b2a6b-120">Architecture</span><span class="sxs-lookup"><span data-stu-id="b2a6b-120">Architecture</span></span>

<span data-ttu-id="b2a6b-121">Die Architektur umfasst die folgenden Komponenten.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="b2a6b-122">**Lokales Netzwerk**.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-122">**On-premises network**.</span></span> <span data-ttu-id="b2a6b-123">Ein in einer Organisation betriebenes privates lokales Netzwerk.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="b2a6b-124">**VPN-Gerät**:</span><span class="sxs-lookup"><span data-stu-id="b2a6b-124">**VPN device**.</span></span> <span data-ttu-id="b2a6b-125">Ein Gerät oder ein Dienst, der externe Konnektivität mit dem lokalen Netzwerk bereitstellt.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="b2a6b-126">Bei dem VPN-Gerät kann es sich um ein Hardwaregerät oder eine Softwarelösung wie den Routing- und RAS-Dienst (RRAS) unter Windows Server 2012 handeln.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="b2a6b-127">Eine Liste der unterstützten VPN-Appliances und Informationen zur Konfiguration ausgewählter VPN-Geräte für die Verbindung mit Azure finden Sie unter [Informationen zu VPN-Geräten und IPsec-/IKE-Parametern für VPN-Gatewayverbindungen zwischen Standorten][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="b2a6b-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="b2a6b-128">**VPN-Gateway für ein virtuelles Netzwerk oder ExpressRoute-Gateway**:</span><span class="sxs-lookup"><span data-stu-id="b2a6b-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="b2a6b-129">Über das Gateway für virtuelle Netzwerke kann das VNET zur Konnektivität mit Ihrem lokalen Netzwerk eine Verbindung mit dem VPN-Gerät oder eine ExpressRoute-Verbindung herstellen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="b2a6b-130">Weitere Informationen finden Sie unter [Verbinden eines lokalen Netzwerks mit einem virtuellen Microsoft Azure-Netzwerk][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="b2a6b-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="b2a6b-131">In den Bereitstellungsskripts für diese Referenzarchitektur werden ein VPN-Gateway für die Konnektivität und ein VNET in Azure verwendet, um Ihr lokales Netzwerk zu simulieren.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="b2a6b-132">**Hub-VNET**:</span><span class="sxs-lookup"><span data-stu-id="b2a6b-132">**Hub VNet**.</span></span> <span data-ttu-id="b2a6b-133">Das Azure-VNET, das als Hub in der Hub-Spoke-Topologie verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="b2a6b-134">Der Hub ist der zentrale Konnektivitätspunkt für Ihr lokales Netzwerk, mit dem Sie Dienste hosten, die von den verschiedenen in den Spoke-VNETs gehosteten Workloads genutzt werden können.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="b2a6b-135">**Gatewaysubnetz**:</span><span class="sxs-lookup"><span data-stu-id="b2a6b-135">**Gateway subnet**.</span></span> <span data-ttu-id="b2a6b-136">Die Gateways für virtuelle Netzwerke befinden sich in demselben Subnetz.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="b2a6b-137">**Spoke-VNETs**:</span><span class="sxs-lookup"><span data-stu-id="b2a6b-137">**Spoke VNets**.</span></span> <span data-ttu-id="b2a6b-138">Ein oder mehrere Azure-VNETs, die als Spokes in der Hub-Spoke-Topologie verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="b2a6b-139">Spokes können verwendet werden, um Workloads in ihren eigenen VNETs, die getrennt von anderen Spokes verwaltet werden, zu isolieren.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="b2a6b-140">Jede Workload kann mehrere Schichten umfassen, wobei mehrere Subnetze über Azure Load Balancer verbunden sind.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="b2a6b-141">Weitere Informationen zur Anwendungsinfrastruktur finden Sie unter [Ausführen von Windows-VM-Workloads][windows-vm-ra] und [Ausführen von Linux-VM-Workloads][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="b2a6b-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="b2a6b-142">**VNET-Peering**:</span><span class="sxs-lookup"><span data-stu-id="b2a6b-142">**VNet peering**.</span></span> <span data-ttu-id="b2a6b-143">Über eine [Peeringverbindung][vnet-peering] können zwei VNETs in derselben Azure-Region miteinander verbunden werden.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-143">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="b2a6b-144">Peeringverbindungen sind nicht-transitive Verbindungen zwischen VNETs mit niedrigen Latenzen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="b2a6b-145">Sobald eine Peeringverbindung hergestellt wurde, tauschen die VNETs ohne Einsatz eines Routers Datenverkehr über den Azure-Backbone aus.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="b2a6b-146">In einer Hub-Spoke-Netzwerktopologie wird durch VNET-Peering eine Verbindung zwischen dem Hub und den einzelnen Spokes hergestellt.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="b2a6b-147">In diesem Artikel werden ausschließlich [Resource Manager](/azure/azure-resource-manager/resource-group-overview)-Bereitstellungen behandelt, Sie können jedoch auch eine Verbindung zwischen einem klassischen VNET und einem Resource Manager-VNET im selben Abonnement herstellen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-147">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="b2a6b-148">Auf diese Weise können Ihre Spokes klassische Bereitstellungen hosten und trotzdem von gemeinsamen Diensten im Hub profitieren.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-148">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="b2a6b-149">Empfehlungen</span><span class="sxs-lookup"><span data-stu-id="b2a6b-149">Recommendations</span></span>

<span data-ttu-id="b2a6b-150">Die folgenden Empfehlungen gelten für die meisten Szenarios.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="b2a6b-151">Sofern Sie keine besonderen Anforderungen haben, die Vorrang haben, sollten Sie diese Empfehlungen befolgen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="b2a6b-152">Ressourcengruppen</span><span class="sxs-lookup"><span data-stu-id="b2a6b-152">Resource groups</span></span>

<span data-ttu-id="b2a6b-153">Das Hub-VNET und die einzelnen Spoke-VNETs können in verschiedenen Ressourcengruppen und sogar in verschiedenen Abonnements implementiert werden, sofern sie demselben Azure Active Directory-Mandanten (Azure AD) in der gleichen Azure-Region angehören.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-153">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="b2a6b-154">Dies ermöglicht nicht nur eine dezentralisierte Verwaltung der einzelnen Workloads, sondern auch die Verwaltung gemeinsamer Dienste im Hub-VNET.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-154">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="b2a6b-155">VNET und GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="b2a6b-155">VNet and GatewaySubnet</span></span>

<span data-ttu-id="b2a6b-156">Erstellen Sie ein Subnetz mit dem Namen *GatewaySubnet* mit dem Adressbereich /27.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-156">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="b2a6b-157">Dieses Subnetz ist für das Gateway für virtuelle Netzwerke erforderlich.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-157">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="b2a6b-158">Durch Zuweisung von 32 Adressen zu diesem Subnetz wird eine künftige Überschreitung von Beschränkungen der Gatewaygröße verhindert.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-158">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="b2a6b-159">Weitere Informationen zum Einrichten des Gateways finden Sie abhängig von Ihrem Verbindungstyp in den folgenden Referenzarchitekturen:</span><span class="sxs-lookup"><span data-stu-id="b2a6b-159">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="b2a6b-160">[Hybrides Netzwerk mit ExpressRoute][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="b2a6b-160">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="b2a6b-161">[Hybrides Netzwerk mit einem VPN-Gateway][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="b2a6b-161">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="b2a6b-162">Um eine höhere Verfügbarkeit zu erzielen, können Sie ExpressRoute mit einem VPN für Failoverzwecke kombinieren.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-162">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="b2a6b-163">Weitere Informationen finden Sie unter [Verbinden eines lokalen Netzwerks mit Azure unter Verwendung von ExpressRoute mit VPN-Failover][hybrid-ha].</span><span class="sxs-lookup"><span data-stu-id="b2a6b-163">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="b2a6b-164">Eine Hub-Spoke-Topologie kann auch ohne Gateway verwendet werden, wenn keine Konnektivität mit Ihrem lokalen Netzwerk erforderlich ist.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-164">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="b2a6b-165">VNet-Peering</span><span class="sxs-lookup"><span data-stu-id="b2a6b-165">VNet peering</span></span>

<span data-ttu-id="b2a6b-166">Beim VNET-Peering wird eine nicht-transitive Beziehung zwischen zwei VNETs hergestellt.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-166">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="b2a6b-167">Wenn Sie eine Verbindung zwischen Spokes herstellen möchten, sollten Sie eventuell eine separate Peeringverbindung zwischen diesen Spokes herstellen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-167">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="b2a6b-168">Wenn jedoch zwischen mehreren Spokes eine Verbindung hergestellt werden muss, werden die möglichen Peeringverbindungen sehr schnell zur Neige gehen, da [die Anzahl der VNET-Peerings pro VNET begrenzt ist][vnet-peering-limit].</span><span class="sxs-lookup"><span data-stu-id="b2a6b-168">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="b2a6b-169">In diesem Szenario sollten Sie die Verwendung von benutzerdefinierten Routen (User Defined Routes, UDRs) in Erwägung ziehen, um zu erzwingen, dass der für einen Spoke vorgesehene Datenverkehr an eine NVA, die als Router im Hub-VNET fungiert, gesendet wird.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-169">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="b2a6b-170">Hierdurch können die Spokes miteinander verbunden werden.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-170">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="b2a6b-171">Sie können Spokes auch für die Kommunikation mit Remotenetzwerken über das Gateway für das Hub-VNET konfigurieren.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-171">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="b2a6b-172">Damit der Gatewaydatenverkehr zwischen den Spokes und dem Hub weitergeleitet und eine Verbindung mit Remotenetzwerken hergestellt werden kann, müssen Sie folgende Schritte durchführen:</span><span class="sxs-lookup"><span data-stu-id="b2a6b-172">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="b2a6b-173">Konfigurieren Sie die VNET-Peeringverbindung im Hub dahingehend, dass **Gatewaytransit zugelassen** wird.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-173">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="b2a6b-174">Konfigurieren Sie die VNET-Peeringverbindung in den einzelnen Spokes dahingehend, dass **Remotegateways verwendet werden**.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-174">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="b2a6b-175">Konfigurieren Sie alle VNET-Peeringverbindungen dahingehend, dass **weitergeleiteter Datenverkehr zugelassen** wird.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-175">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="b2a6b-176">Überlegungen</span><span class="sxs-lookup"><span data-stu-id="b2a6b-176">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="b2a6b-177">Konnektivität zwischen Spokes</span><span class="sxs-lookup"><span data-stu-id="b2a6b-177">Spoke connectivity</span></span>

<span data-ttu-id="b2a6b-178">Wenn Konnektivität zwischen Spokes hergestellt werden muss, sollten Sie eine NVA für das Routing im Hub implementieren und UDRs im Spoke zur Weiterleitung des Datenverkehrs an den Hub verwenden.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-178">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="b2a6b-179">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="b2a6b-179">![[2]][2]</span></span>

<span data-ttu-id="b2a6b-180">In diesem Szenario müssen Sie die Peeringverbindungen dahingehend konfigurieren, dass **weitergeleiteter Verkehr zugelassen** wird.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-180">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="b2a6b-181">Umgehung von VNET-Peeringbeschränkungen</span><span class="sxs-lookup"><span data-stu-id="b2a6b-181">Overcoming VNet peering limits</span></span>

<span data-ttu-id="b2a6b-182">Stellen Sie sicher, dass Sie die [Begrenzung der Anzahl von VNET-Peerings pro VNET][vnet-peering-limit] in Azure einhalten.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-182">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="b2a6b-183">Falls Sie mehr Spokes als die maximal zulässige Anzahl benötigen, sollten Sie eventuell eine Hub-Spoke-Hub-Spoke-Topologie erstellen, bei der die erste Ebene von Spokes auch als Hub fungiert.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-183">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="b2a6b-184">Im folgenden Diagramm wird diese Vorgehensweise veranschaulicht.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-184">The following diagram shows this approach.</span></span>

<span data-ttu-id="b2a6b-185">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="b2a6b-185">![[3]][3]</span></span>

<span data-ttu-id="b2a6b-186">Berücksichtigen Sie auch die Dienste, die im Hub gemeinsam genutzt werden, um sicherzustellen, dass sich der Hub für eine größere Anzahl von Spokes skalieren lässt.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-186">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="b2a6b-187">Wenn Ihr Hub beispielsweise Firewalldienste bereitstellt, sollten Sie beim Hinzufügen mehrerer Spokes die Bandbreitenbeschränkungen Ihrer Firewalllösung berücksichtigen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-187">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="b2a6b-188">Es wird empfohlen, einige dieser gemeinsamen Dienste auf eine zweite Hubebene zu verlagern.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-188">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="b2a6b-189">Bereitstellen der Lösung</span><span class="sxs-lookup"><span data-stu-id="b2a6b-189">Deploy the solution</span></span>

<span data-ttu-id="b2a6b-190">Eine Bereitstellung für diese Architektur ist auf [GitHub][ref-arch-repo] verfügbar.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-190">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="b2a6b-191">Bei dieser werden zum Testen der Konnektivität Ubuntu-VMs in jedem VNET verwendet.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-191">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="b2a6b-192">Es gibt keine tatsächlichen Dienste, die im Subnetz für **gemeinsame Dienste** im **Hub-VNET** gehostet werden.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-192">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="b2a6b-193">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="b2a6b-193">Prerequisites</span></span>

<span data-ttu-id="b2a6b-194">Bevor Sie die Referenzarchitektur in Ihrem eigenen Abonnement bereitstellen können, müssen Sie die folgenden Schritte ausführen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-194">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="b2a6b-195">Klonen oder Forken Sie das GitHub-Repository [AzureCAT-Referenzarchitekturen][ref-arch-repo], oder laden Sie die zugehörige ZIP-Datei herunter.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-195">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="b2a6b-196">Vergewissern Sie sich, dass Azure CLI 2.0 auf Ihrem Computer installiert ist.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-196">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="b2a6b-197">Anweisungen zur CLI-Installation finden Sie unter [Installieren von Azure-CLI 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="b2a6b-197">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="b2a6b-198">Installieren Sie das npm-Paket mit den [Azure-Bausteinen][azbb].</span><span class="sxs-lookup"><span data-stu-id="b2a6b-198">Install the [Azure buulding blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="b2a6b-199">Melden Sie sich über eine Eingabeaufforderung, eine Bash-Eingabeaufforderung oder die PowerShell-Eingabeaufforderung an Ihrem Azure-Konto an. Verwenden Sie hierzu den unten aufgeführten Befehl, und befolgen Sie die Anweisungen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-199">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="b2a6b-200">Bereitstellen des simulierten lokalen Rechenzentrums mit azbb</span><span class="sxs-lookup"><span data-stu-id="b2a6b-200">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="b2a6b-201">Führen Sie die folgenden Schritte aus, um das simulierte lokale Rechenzentrum als Azure-VNET bereitzustellen:</span><span class="sxs-lookup"><span data-stu-id="b2a6b-201">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="b2a6b-202">Navigieren Sie zum Ordner `hybrid-networking\hub-spoke\` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-202">Navigate to the `hybrid-networking\hub-spoke\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="b2a6b-203">Öffnen Sie die Datei `onprem.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 36 und 37 ein, und speichern Sie die Datei.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-203">Open the `onprem.json` file and enter a username and password between the quotes in line 36 and 37, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="b2a6b-204">Geben Sie in Zeile 38 unter `osType` entweder `Windows` oder `Linux` ein, um Windows Server 2016 Datacenter bzw. Ubuntu 16.04 als Betriebssystem für die Jumpbox zu installieren.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-204">On line 38, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

4. <span data-ttu-id="b2a6b-205">Führen Sie `azbb` aus, um die simulierte lokale Umgebung wie unten gezeigt bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-205">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="b2a6b-206">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `onprem-vnet-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-206">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="b2a6b-207">Warten Sie, bis die Bereitstellung abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-207">Wait for the deployment to finish.</span></span> <span data-ttu-id="b2a6b-208">Bei dieser Bereitstellung werden ein virtuelles Netzwerk, ein virtueller Computer und ein VPN-Gateway erstellt.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-208">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="b2a6b-209">Die Erstellung des VPN-Gateways kann mehr als 40 Minuten in Anspruch nehmen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-209">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="b2a6b-210">Azure-Hub-VNET</span><span class="sxs-lookup"><span data-stu-id="b2a6b-210">Azure hub VNet</span></span>

<span data-ttu-id="b2a6b-211">Führen Sie die folgenden Schritte durch, um das Hub-VNET bereitzustellen und eine Verbindung mit dem oben erstellten simulierten lokalen VNET herzustellen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-211">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="b2a6b-212">Öffnen Sie die Datei `hub-vnet.json`, und geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 39 und 40 ein.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-212">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 39 and 40, as shown below.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="b2a6b-213">Geben Sie in Zeile 41 unter `osType` entweder `Windows` oder `Linux` ein, um Windows Server 2016 Datacenter bzw. Ubuntu 16.04 als Betriebssystem für die Jumpbox zu installieren.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-213">On line 41, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="b2a6b-214">Geben Sie einen gemeinsam verwendeten Schlüssel wie unten dargestellt zwischen den Anführungszeichen in Zeile 72 ein, und speichern Sie die Datei.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-214">Enter a shared key between the quotes in line 72, as shown below, then save the file.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="b2a6b-215">Führen Sie `azbb` aus, um die simulierte lokale Umgebung wie unten gezeigt bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-215">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="b2a6b-216">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `hub-vnet-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-216">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="b2a6b-217">Warten Sie, bis die Bereitstellung abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-217">Wait for the deployment to finish.</span></span> <span data-ttu-id="b2a6b-218">Bei dieser Bereitstellung werden ein virtuelles Netzwerk, ein virtueller Computer, ein VPN-Gateway und eine Verbindung mit dem im vorherigen Abschnitt erstellten Gateway erstellt.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-218">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="b2a6b-219">Die Erstellung des VPN-Gateways kann mehr als 40 Minuten in Anspruch nehmen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-219">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="optional-test-connectivity-from-onprem-to-hub"></a><span data-ttu-id="b2a6b-220">(Optional) Testen der Konnektivität vom lokalen Standort zum Hub</span><span class="sxs-lookup"><span data-stu-id="b2a6b-220">(Optional) Test connectivity from onprem to hub</span></span>

<span data-ttu-id="b2a6b-221">Führen Sie die folgenden Schritte aus, um die Konnektivität der simulierten lokalen Umgebung mit dem Hub-VNET mit Windows-VMs zu testen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-221">To test conectivity from the simulated on-premises environment to the hub VNet using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="b2a6b-222">Navigieren Sie im Azure-Portal zur Ressourcengruppe `onprem-jb-rg`, und klicken Sie dann auf die VM-Ressource `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-222">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2.  <span data-ttu-id="b2a6b-223">Klicken Sie im Portal oben links auf dem Blatt für Ihre VM auf `Connect`, und folgen Sie den Anweisungen zur Verwendung des Remotedesktops zum Herstellen einer Verbindung mit der VM.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-223">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="b2a6b-224">Achten Sie darauf, dass Sie den Benutzernamen und das Kennwort verwenden, wie in den Zeilen 36 und 37 in der Datei `onprem.json` angegeben.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-224">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="b2a6b-225">Öffnen Sie auf der VM eine PowerShell-Konsole, und verwenden Sie das `Test-NetConnection`-Cmdlet, um sicherzustellen, dass Sie wie unten gezeigt eine Verbindung mit der Hub-Jumpbox-VM herstellen können.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-225">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

  ```powershell
  Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
  ```
  > [!NOTE]
  > <span data-ttu-id="b2a6b-226">Standardmäßig lassen Windows Server-VMs in Azure keine ICMP-Antworten zu.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-226">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="b2a6b-227">Wenn Sie `ping` zum Testen der Konnektivität nutzen möchten, müssen Sie für jede VM ICMP-Datenverkehr in der erweiterten Windows-Firewall aktivieren.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-227">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="b2a6b-228">Führen Sie die folgenden Schritte aus, um die Konnektivität der simulierten lokalen Umgebung mit dem Hub-VNET mit Linux-VMs zu testen:</span><span class="sxs-lookup"><span data-stu-id="b2a6b-228">To test conectivity from the simulated on-premises environment to the hub VNet using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="b2a6b-229">Navigieren Sie im Azure-Portal zur Ressourcengruppe `onprem-jb-rg`, und klicken Sie dann auf die VM-Ressource `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-229">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="b2a6b-230">Klicken Sie im Portal oben links auf dem Blatt für Ihre VM auf `Connect`, und kopieren Sie dann den im Portal angezeigten `ssh`-Befehl.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-230">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="b2a6b-231">Führen Sie an einer Linux-Eingabeaufforderung `ssh` aus, um wie unten gezeigt eine Verbindung mit der Jumpbox der simulierten lokalen Umgebung herzustellen, indem Sie die oben in Schritt 2 kopierten Informationen verwenden.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-231">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

  ```bash
  ssh <your_user>@<public_ip_address>
  ```

4. <span data-ttu-id="b2a6b-232">Nutzen Sie das Kennwort, das Sie in Zeile 37 in der Datei `onprem.json` angegeben haben, um eine Verbindung mit der VM herzustellen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-232">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

5. <span data-ttu-id="b2a6b-233">Verwenden Sie den Befehl `ping`, um die Konnektivität mit der Hub-Jumpbox wie unten gezeigt zu testen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-233">Use the `ping` command to test connectivity to the hub jumpbox, as shown below.</span></span>

  ```bash
  ping 10.0.0.68
  ```

### <a name="azure-spoke-vnets"></a><span data-ttu-id="b2a6b-234">Azure-Spoke-VNETs</span><span class="sxs-lookup"><span data-stu-id="b2a6b-234">Azure spoke VNets</span></span>

<span data-ttu-id="b2a6b-235">Führen Sie die folgenden Schritte aus, um die Spoke-VNETs bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-235">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="b2a6b-236">Öffnen Sie die Datei `spoke1.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 47 und 48 ein, und speichern Sie die Datei.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-236">Open the `spoke1.json` file and enter a username and password between the quotes in lines 47 and 48, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="b2a6b-237">Geben Sie in Zeile 49 unter `osType` entweder `Windows` oder `Linux` ein, um Windows Server 2016 Datacenter bzw. Ubuntu 16.04 als Betriebssystem für die Jumpbox zu installieren.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-237">On line 49, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="b2a6b-238">Führen Sie `azbb` aus, um die erste Spoke-VNET-Umgebung wie unten gezeigt bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-238">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
  ```
  
  > [!NOTE]
  > <span data-ttu-id="b2a6b-239">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `spoke1-vnet-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-239">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

3. <span data-ttu-id="b2a6b-240">Wiederholen Sie den obigen Schritt 1 für die Datei `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-240">Repeat step 1 above for file `spoke2.json`.</span></span>

4. <span data-ttu-id="b2a6b-241">Führen Sie `azbb` aus, um die zweite Spoke-VNET-Umgebung wie unten gezeigt bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-241">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="b2a6b-242">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `spoke2-vnet-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-242">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="b2a6b-243">Herstellen einer Peeringverbindung zwischen Azure-Hub-VNETs und Spoke-VNETs</span><span class="sxs-lookup"><span data-stu-id="b2a6b-243">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="b2a6b-244">Führen Sie die folgenden Schritte aus, um eine Peeringverbindung vom Hub-VNET mit den Spoke-VNETs zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-244">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="b2a6b-245">Öffnen Sie die Datei `hub-vnet-peering.json`, und stellen Sie sicher, dass der Ressourcengruppenname und der Name des virtuellen Netzwerks für die einzelnen VNET-Peerings ab Zeile 29 korrekt sind.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-245">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="b2a6b-246">Führen Sie `azbb` aus, um die erste Spoke-VNET-Umgebung wie unten gezeigt bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-246">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
  ```

  > [!NOTE]
  > <span data-ttu-id="b2a6b-247">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `hub-vnet-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-247">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="test-connectivity"></a><span data-ttu-id="b2a6b-248">Testen der Konnektivität</span><span class="sxs-lookup"><span data-stu-id="b2a6b-248">Test connectivity</span></span>

<span data-ttu-id="b2a6b-249">Führen Sie die folgenden Schritte aus, um die Konnektivität der simulierten lokalen Umgebung mit den Spoke-VNETs mit Windows-VMs zu testen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-249">To test conectivity from the simulated on-premises environment to the spoke VNets using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="b2a6b-250">Navigieren Sie im Azure-Portal zur Ressourcengruppe `onprem-jb-rg`, und klicken Sie dann auf die VM-Ressource `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-250">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2.  <span data-ttu-id="b2a6b-251">Klicken Sie im Portal oben links auf dem Blatt für Ihre VM auf `Connect`, und folgen Sie den Anweisungen zur Verwendung des Remotedesktops zum Herstellen einer Verbindung mit der VM.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-251">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="b2a6b-252">Achten Sie darauf, dass Sie den Benutzernamen und das Kennwort verwenden, wie in den Zeilen 36 und 37 in der Datei `onprem.json` angegeben.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-252">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="b2a6b-253">Öffnen Sie auf der VM eine PowerShell-Konsole, und verwenden Sie das `Test-NetConnection`-Cmdlet, um sicherzustellen, dass Sie wie unten gezeigt eine Verbindung mit der Hub-Jumpbox-VM herstellen können.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-253">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

  ```powershell
  Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
  Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
  ```

<span data-ttu-id="b2a6b-254">Führen Sie die folgenden Schritte aus, um die Konnektivität der simulierten lokalen Umgebung mit den Spoke-VNETs mit Linux-VMs zu testen:</span><span class="sxs-lookup"><span data-stu-id="b2a6b-254">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="b2a6b-255">Navigieren Sie im Azure-Portal zur Ressourcengruppe `onprem-jb-rg`, und klicken Sie dann auf die VM-Ressource `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-255">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="b2a6b-256">Klicken Sie im Portal oben links auf dem Blatt für Ihre VM auf `Connect`, und kopieren Sie dann den im Portal angezeigten `ssh`-Befehl.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-256">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="b2a6b-257">Führen Sie an einer Linux-Eingabeaufforderung `ssh` aus, um wie unten gezeigt eine Verbindung mit der Jumpbox der simulierten lokalen Umgebung herzustellen, indem Sie die oben in Schritt 2 kopierten Informationen verwenden.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-257">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

  ```bash
  ssh <your_user>@<public_ip_address>
  ```

5. <span data-ttu-id="b2a6b-258">Nutzen Sie das Kennwort, das Sie in Zeile 37 in der Datei `onprem.json` angegeben haben, um eine Verbindung mit der VM herzustellen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-258">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

6. <span data-ttu-id="b2a6b-259">Verwenden Sie den Befehl `ping`, um die Konnektivität mit den Jumpbox-VMs in den einzelnen Spokes wie unten gezeigt zu testen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-259">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke, as shown below.</span></span>

  ```bash
  ping 10.1.0.68
  ping 10.2.0.68
  ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="b2a6b-260">Herstellen von Konnektivität zwischen Spokes</span><span class="sxs-lookup"><span data-stu-id="b2a6b-260">Add connectivity between spokes</span></span>

<span data-ttu-id="b2a6b-261">Wenn Sie zulassen möchten, dass Spokes Verbindungen miteinander herstellen können, müssen Sie ein virtuelles Netzwerkgerät als Router im virtuellen Hubnetzwerk verwenden. Außerdem müssen Sie erzwingen, dass Datenverkehr von den Spokes über den Router verläuft, wenn versucht wird, eine Verbindung mit anderen Spokes herzustellen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-261">If you want to allow spokes to connect to each other, you need to use a newtwork virtual appliance (NVA) as a router in the hub virtual netowrk, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="b2a6b-262">Führen Sie die folgenden Schritte aus, um ein einfaches Beispiel-Netzwerkgerät als einzelne VM und die erforderlichen benutzerdefinierten Routen bereitzustellen, um die Verbindungsherstellung für zwei Spoke-VNETs zu ermöglichen:</span><span class="sxs-lookup"><span data-stu-id="b2a6b-262">To deploy a basic sample NVA as a single VM, and the necessary uder defined routes to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="b2a6b-263">Öffnen Sie die Datei `hub-nva.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 13 und 14 ein, und speichern Sie die Datei.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-263">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```
2. <span data-ttu-id="b2a6b-264">Führen Sie `azbb` aus, um die NVA-VM und die benutzerdefinierten Routen bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-264">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="b2a6b-265">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `hub-nva-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="b2a6b-265">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

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

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Hub-Spoke-Topologie in Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Hub-Spoke-Topologie in Azure mit transitivem Routing"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Hub-Spoke-Topologie in Azure mit transitivem Routing unter Verwendung eines NVA"
[3]: ./images/hub-spokehub-spoke.svg "Hub-Spoke-Hub-Spoke-Topologie in Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
