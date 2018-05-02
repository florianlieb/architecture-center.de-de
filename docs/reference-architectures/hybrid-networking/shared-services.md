---
title: Implementieren einer Hub-Spoke-Netzwerktopologie mit gemeinsamen Diensten in Azure
description: Es wird beschrieben, wie Sie eine Hub-Spoke-Netzwerktopologie mit gemeinsamen Diensten in Azure implementieren.
author: telmosampaio
ms.date: 02/25/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: 83367a3be2f7a1e33c2ef7018d42f70aae99104d
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/16/2018
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="a3922-103">Implementieren einer Hub-Spoke-Netzwerktopologie mit gemeinsamen Diensten in Azure</span><span class="sxs-lookup"><span data-stu-id="a3922-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="a3922-104">Diese Referenzarchitektur baut auf der [Hub-Spoke][guidance-hub-spoke]-Referenzarchitektur auf, um gemeinsame Dienste in den Hub einzubinden, die von allen Spokes genutzt werden können.</span><span class="sxs-lookup"><span data-stu-id="a3922-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="a3922-105">Als ersten Schritt zur Migration eines Rechenzentrums zur Cloud und Erstellung eines [virtuellen Rechenzentrums] müssen Sie zunächst die Dienste für Identität und Sicherheit freigeben.</span><span class="sxs-lookup"><span data-stu-id="a3922-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="a3922-106">Anhand dieser Referenzarchitektur wird veranschaulicht, wie Sie Ihre Active Directory-Dienste aus Ihrem lokalen Rechenzentrum auf Azure ausdehnen und ein virtuelles Netzwerkgerät (Network Virtual Appliance, NVA) hinzufügen, das in einer Hub-Spoke-Topologie als Firewall fungieren kann.</span><span class="sxs-lookup"><span data-stu-id="a3922-106">This reference architecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="a3922-107">[**Stellen Sie diese Lösung bereit**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="a3922-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="a3922-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="a3922-108">![[0]][0]</span></span>

<span data-ttu-id="a3922-109">*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*</span><span class="sxs-lookup"><span data-stu-id="a3922-109">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="a3922-110">Diese Topologie bietet unter anderem folgende Vorteile:</span><span class="sxs-lookup"><span data-stu-id="a3922-110">The benefits of this topology include:</span></span>

* <span data-ttu-id="a3922-111">**Kosteneinsparungen** durch die Zentralisierung von Diensten, die von mehreren Workloads (z.B. Network Virtual Appliances, kurz NVAs, und DNS-Servern) an einem einzigen Standort gemeinsam genutzt werden können</span><span class="sxs-lookup"><span data-stu-id="a3922-111">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="a3922-112">**Umgehung von Abonnementbeschränkungen** durch die Herstellung von Peeringverbindungen zwischen VNETs verschiedener Abonnements und dem zentralen Hub</span><span class="sxs-lookup"><span data-stu-id="a3922-112">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="a3922-113">**Trennung der Belange** zwischen zentralen IT-Vorgängen (SecOps, InfraOps) und Workloads (DevOps)</span><span class="sxs-lookup"><span data-stu-id="a3922-113">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="a3922-114">Typische Einsatzmöglichkeiten für diese Architektur sind Folgende:</span><span class="sxs-lookup"><span data-stu-id="a3922-114">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="a3922-115">Workloads, die in verschiedenen Umgebungen wie Entwicklungs-, Test- und Produktionsumgebungen eingesetzt werden und gemeinsame Dienste wie DNS, IDS, NTP oder AD DS erfordern</span><span class="sxs-lookup"><span data-stu-id="a3922-115">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="a3922-116">Gemeinsame Dienste werden im Hub-VNET platziert, während die einzelnen Umgebungen in einem Spoke bereitgestellt werden, um die Isolation beizubehalten.</span><span class="sxs-lookup"><span data-stu-id="a3922-116">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="a3922-117">Workloads, bei denen keine Konnektivität untereinander bestehen muss, die jedoch Zugriff auf gemeinsame Dienste erfordern</span><span class="sxs-lookup"><span data-stu-id="a3922-117">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="a3922-118">Unternehmen, die auf eine zentrale Steuerung von Sicherheitsmechanismen (z.B. eine Firewall im Hub als DMZ) und eine getrennte Verwaltung von Workloads in den einzelnen Spokes angewiesen sind</span><span class="sxs-lookup"><span data-stu-id="a3922-118">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="a3922-119">Architecture</span><span class="sxs-lookup"><span data-stu-id="a3922-119">Architecture</span></span>

<span data-ttu-id="a3922-120">Die Architektur umfasst die folgenden Komponenten.</span><span class="sxs-lookup"><span data-stu-id="a3922-120">The architecture consists of the following components.</span></span>

* <span data-ttu-id="a3922-121">**Lokales Netzwerk**.</span><span class="sxs-lookup"><span data-stu-id="a3922-121">**On-premises network**.</span></span> <span data-ttu-id="a3922-122">Ein in einer Organisation betriebenes privates lokales Netzwerk.</span><span class="sxs-lookup"><span data-stu-id="a3922-122">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="a3922-123">**VPN-Gerät**:</span><span class="sxs-lookup"><span data-stu-id="a3922-123">**VPN device**.</span></span> <span data-ttu-id="a3922-124">Ein Gerät oder ein Dienst, der externe Konnektivität mit dem lokalen Netzwerk bereitstellt.</span><span class="sxs-lookup"><span data-stu-id="a3922-124">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="a3922-125">Bei dem VPN-Gerät kann es sich um ein Hardwaregerät oder eine Softwarelösung wie den Routing- und RAS-Dienst (RRAS) unter Windows Server 2012 handeln.</span><span class="sxs-lookup"><span data-stu-id="a3922-125">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="a3922-126">Eine Liste der unterstützten VPN-Appliances und Informationen zur Konfiguration ausgewählter VPN-Geräte für die Verbindung mit Azure finden Sie unter [Informationen zu VPN-Geräten und IPsec-/IKE-Parametern für VPN-Gatewayverbindungen zwischen Standorten][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="a3922-126">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="a3922-127">**VPN-Gateway für ein virtuelles Netzwerk oder ExpressRoute-Gateway**:</span><span class="sxs-lookup"><span data-stu-id="a3922-127">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="a3922-128">Über das Gateway für virtuelle Netzwerke kann das VNET zur Konnektivität mit Ihrem lokalen Netzwerk eine Verbindung mit dem VPN-Gerät oder eine ExpressRoute-Verbindung herstellen.</span><span class="sxs-lookup"><span data-stu-id="a3922-128">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="a3922-129">Weitere Informationen finden Sie unter [Verbinden eines lokalen Netzwerks mit einem virtuellen Microsoft Azure-Netzwerk][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="a3922-129">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="a3922-130">In den Bereitstellungsskripts für diese Referenzarchitektur werden ein VPN-Gateway für die Konnektivität und ein VNET in Azure verwendet, um Ihr lokales Netzwerk zu simulieren.</span><span class="sxs-lookup"><span data-stu-id="a3922-130">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="a3922-131">**Hub-VNET**:</span><span class="sxs-lookup"><span data-stu-id="a3922-131">**Hub VNet**.</span></span> <span data-ttu-id="a3922-132">Das Azure-VNET, das als Hub in der Hub-Spoke-Topologie verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="a3922-132">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="a3922-133">Der Hub ist der zentrale Konnektivitätspunkt für Ihr lokales Netzwerk, mit dem Sie Dienste hosten, die von den verschiedenen in den Spoke-VNETs gehosteten Workloads genutzt werden können.</span><span class="sxs-lookup"><span data-stu-id="a3922-133">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="a3922-134">**Gatewaysubnetz**:</span><span class="sxs-lookup"><span data-stu-id="a3922-134">**Gateway subnet**.</span></span> <span data-ttu-id="a3922-135">Die Gateways für die virtuellen Netzwerke befinden sich im selben Subnetz.</span><span class="sxs-lookup"><span data-stu-id="a3922-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="a3922-136">**Subnetz für gemeinsame Dienste**:</span><span class="sxs-lookup"><span data-stu-id="a3922-136">**Shared services subnet**.</span></span> <span data-ttu-id="a3922-137">Ein Subnetz im Hub-VNET, das für das Hosten von Diensten verwendet wird, die von allen Spokes gemeinsam genutzt werden können (z.B. DNS oder AD DS).</span><span class="sxs-lookup"><span data-stu-id="a3922-137">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="a3922-138">**DMZ-Subnetz**:</span><span class="sxs-lookup"><span data-stu-id="a3922-138">**DMZ subnet**.</span></span> <span data-ttu-id="a3922-139">Ein Subnetz im Hub-VNET zum Hosten von NVAs, die als Sicherheitseinrichtungen, z.B. Firewalls, dienen können.</span><span class="sxs-lookup"><span data-stu-id="a3922-139">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

* <span data-ttu-id="a3922-140">**Spoke-VNETs**:</span><span class="sxs-lookup"><span data-stu-id="a3922-140">**Spoke VNets**.</span></span> <span data-ttu-id="a3922-141">Ein oder mehrere Azure-VNETs, die als Spokes in der Hub-Spoke-Topologie verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="a3922-141">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="a3922-142">Spokes können verwendet werden, um Workloads in ihren eigenen VNETs, die getrennt von anderen Spokes verwaltet werden, zu isolieren.</span><span class="sxs-lookup"><span data-stu-id="a3922-142">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="a3922-143">Jede Workload kann mehrere Schichten umfassen, wobei mehrere Subnetze über Azure Load Balancer verbunden sind.</span><span class="sxs-lookup"><span data-stu-id="a3922-143">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="a3922-144">Weitere Informationen zur Anwendungsinfrastruktur finden Sie unter [Ausführen von Windows-VM-Workloads][windows-vm-ra] und [Ausführen von Linux-VM-Workloads][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="a3922-144">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="a3922-145">**VNET-Peering**:</span><span class="sxs-lookup"><span data-stu-id="a3922-145">**VNet peering**.</span></span> <span data-ttu-id="a3922-146">Über eine [Peeringverbindung][vnet-peering] können zwei VNETs in derselben Azure-Region miteinander verbunden werden.</span><span class="sxs-lookup"><span data-stu-id="a3922-146">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="a3922-147">Peeringverbindungen sind nicht-transitive Verbindungen zwischen VNETs mit niedrigen Latenzen.</span><span class="sxs-lookup"><span data-stu-id="a3922-147">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="a3922-148">Sobald eine Peeringverbindung hergestellt wurde, tauschen die VNETs ohne Einsatz eines Routers Datenverkehr über den Azure-Backbone aus.</span><span class="sxs-lookup"><span data-stu-id="a3922-148">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="a3922-149">In einer Hub-Spoke-Netzwerktopologie wird durch VNET-Peering eine Verbindung zwischen dem Hub und den einzelnen Spokes hergestellt.</span><span class="sxs-lookup"><span data-stu-id="a3922-149">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="a3922-150">In diesem Artikel werden ausschließlich [Resource Manager](/azure/azure-resource-manager/resource-group-overview)-Bereitstellungen behandelt, Sie können jedoch auch eine Verbindung zwischen einem klassischen VNET und einem Resource Manager-VNET im selben Abonnement herstellen.</span><span class="sxs-lookup"><span data-stu-id="a3922-150">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="a3922-151">Auf diese Weise können Ihre Spokes klassische Bereitstellungen hosten und trotzdem von gemeinsamen Diensten im Hub profitieren.</span><span class="sxs-lookup"><span data-stu-id="a3922-151">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="a3922-152">Empfehlungen</span><span class="sxs-lookup"><span data-stu-id="a3922-152">Recommendations</span></span>

<span data-ttu-id="a3922-153">Alle Empfehlungen für die [Hub-Spoke][guidance-hub-spoke]-Referenzarchitektur gelten auch für die Referenzarchitektur für gemeinsame Dienste.</span><span class="sxs-lookup"><span data-stu-id="a3922-153">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span> 

<span data-ttu-id="a3922-154">Außerdem gelten die folgenden Empfehlungen für die meisten Szenarien mit gemeinsamen Diensten.</span><span class="sxs-lookup"><span data-stu-id="a3922-154">ALso, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="a3922-155">Sofern Sie keine besonderen Anforderungen haben, die Vorrang haben, sollten Sie diese Empfehlungen befolgen.</span><span class="sxs-lookup"><span data-stu-id="a3922-155">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="a3922-156">Identity</span><span class="sxs-lookup"><span data-stu-id="a3922-156">Identity</span></span>

<span data-ttu-id="a3922-157">Die meisten Organisationen großer Unternehmen verfügen im lokalen Rechenzentrum über eine AD DS-Umgebung (Active Directory Domain Services).</span><span class="sxs-lookup"><span data-stu-id="a3922-157">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="a3922-158">Um die Verwaltung von Assets zu ermöglichen, die aus Ihrem lokalen Netzwerk nach Azure verschoben werden und von AD DS abhängig sind, ist es ratsam, AD DS-Domänencontroller in Azure zu hosten.</span><span class="sxs-lookup"><span data-stu-id="a3922-158">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="a3922-159">Falls Sie Gruppenrichtlinienobjekte nutzen, die Sie für Azure und Ihre lokale Umgebung separat steuern möchten, sollten Sie für jede Azure-Region einen anderen AD-Standort verwenden.</span><span class="sxs-lookup"><span data-stu-id="a3922-159">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="a3922-160">Ordnen Sie Ihre Domänencontroller in einem zentralen VNET (Hub) an, auf das abhängige Workloads zugreifen können.</span><span class="sxs-lookup"><span data-stu-id="a3922-160">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="a3922-161">Sicherheit</span><span class="sxs-lookup"><span data-stu-id="a3922-161">Security</span></span>

<span data-ttu-id="a3922-162">Wenn Sie Workloads aus Ihrer lokalen Umgebung nach Azure verschieben, müssen einige dieser Workloads auf VMs gehostet werden.</span><span class="sxs-lookup"><span data-stu-id="a3922-162">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="a3922-163">Aus Gründen der Konformität müssen Sie für Datenverkehr, der diese Workloads durchläuft, unter Umständen Einschränkungen erzwingen.</span><span class="sxs-lookup"><span data-stu-id="a3922-163">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span> 

<span data-ttu-id="a3922-164">Sie können virtuelle Netzwerkgeräte in Azure verwenden, um unterschiedliche Arten von Sicherheits- und Leistungsdiensten zu hosten.</span><span class="sxs-lookup"><span data-stu-id="a3922-164">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="a3922-165">Wenn Sie derzeit mit einem bestimmten Satz von lokalen Geräten vertraut sind, ist es ratsam, jeweils die gleichen virtualisierten Geräte in Azure zu verwenden (falls zutreffend).</span><span class="sxs-lookup"><span data-stu-id="a3922-165">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="a3922-166">Für die Bereitstellungsskripts dieser Referenzarchitektur wird eine Ubuntu-VM mit aktivierter IP-Weiterleitung verwendet, um ein virtuelles Netzwerkgerät zu imitieren.</span><span class="sxs-lookup"><span data-stu-id="a3922-166">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enabled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="a3922-167">Überlegungen</span><span class="sxs-lookup"><span data-stu-id="a3922-167">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="a3922-168">Umgehung von VNET-Peeringbeschränkungen</span><span class="sxs-lookup"><span data-stu-id="a3922-168">Overcoming VNet peering limits</span></span>

<span data-ttu-id="a3922-169">Stellen Sie sicher, dass Sie die [Begrenzung der Anzahl von VNET-Peerings pro VNET][vnet-peering-limit] in Azure einhalten.</span><span class="sxs-lookup"><span data-stu-id="a3922-169">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="a3922-170">Falls Sie mehr Spokes als die maximal zulässige Anzahl benötigen, sollten Sie eventuell eine Hub-Spoke-Hub-Spoke-Topologie erstellen, bei der die erste Ebene von Spokes auch als Hub fungiert.</span><span class="sxs-lookup"><span data-stu-id="a3922-170">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="a3922-171">Im folgenden Diagramm wird diese Vorgehensweise veranschaulicht.</span><span class="sxs-lookup"><span data-stu-id="a3922-171">The following diagram shows this approach.</span></span>

<span data-ttu-id="a3922-172">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="a3922-172">![[3]][3]</span></span>

<span data-ttu-id="a3922-173">Berücksichtigen Sie auch die Dienste, die im Hub gemeinsam genutzt werden, um sicherzustellen, dass sich der Hub für eine größere Anzahl von Spokes skalieren lässt.</span><span class="sxs-lookup"><span data-stu-id="a3922-173">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="a3922-174">Wenn Ihr Hub beispielsweise Firewalldienste bereitstellt, sollten Sie beim Hinzufügen mehrerer Spokes die Bandbreitenbeschränkungen Ihrer Firewalllösung berücksichtigen.</span><span class="sxs-lookup"><span data-stu-id="a3922-174">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="a3922-175">Es wird empfohlen, einige dieser gemeinsamen Dienste auf eine zweite Hubebene zu verlagern.</span><span class="sxs-lookup"><span data-stu-id="a3922-175">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="a3922-176">Bereitstellen der Lösung</span><span class="sxs-lookup"><span data-stu-id="a3922-176">Deploy the solution</span></span>

<span data-ttu-id="a3922-177">Eine Bereitstellung für diese Architektur ist auf [GitHub][ref-arch-repo] verfügbar.</span><span class="sxs-lookup"><span data-stu-id="a3922-177">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="a3922-178">Bei dieser werden zum Testen der Konnektivität Ubuntu-VMs in jedem VNET verwendet.</span><span class="sxs-lookup"><span data-stu-id="a3922-178">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="a3922-179">Es gibt keine tatsächlichen Dienste, die im Subnetz für **gemeinsame Dienste** im **Hub-VNET** gehostet werden.</span><span class="sxs-lookup"><span data-stu-id="a3922-179">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="a3922-180">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="a3922-180">Prerequisites</span></span>

<span data-ttu-id="a3922-181">Bevor Sie die Referenzarchitektur in Ihrem eigenen Abonnement bereitstellen können, müssen Sie die folgenden Schritte ausführen.</span><span class="sxs-lookup"><span data-stu-id="a3922-181">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="a3922-182">Klonen oder Forken Sie das GitHub-Repository [Referenzarchitekturen][ref-arch-repo], oder laden Sie die entsprechende ZIP-Datei herunter.</span><span class="sxs-lookup"><span data-stu-id="a3922-182">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="a3922-183">Vergewissern Sie sich, dass Azure CLI 2.0 auf Ihrem Computer installiert ist.</span><span class="sxs-lookup"><span data-stu-id="a3922-183">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="a3922-184">Anweisungen zur CLI-Installation finden Sie unter [Installieren von Azure-CLI 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="a3922-184">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="a3922-185">Installieren Sie das npm-Paket mit den [Azure Bausteinen][azbb].</span><span class="sxs-lookup"><span data-stu-id="a3922-185">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="a3922-186">Melden Sie sich über eine Eingabeaufforderung, eine Bash-Eingabeaufforderung oder die PowerShell-Eingabeaufforderung an Ihrem Azure-Konto an. Verwenden Sie hierzu den unten aufgeführten Befehl, und befolgen Sie die Anweisungen.</span><span class="sxs-lookup"><span data-stu-id="a3922-186">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="a3922-187">Bereitstellen des simulierten lokalen Rechenzentrums mit azbb</span><span class="sxs-lookup"><span data-stu-id="a3922-187">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="a3922-188">Führen Sie die folgenden Schritte aus, um das simulierte lokale Rechenzentrum als Azure-VNET bereitzustellen:</span><span class="sxs-lookup"><span data-stu-id="a3922-188">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="a3922-189">Navigieren Sie zum Ordner `hybrid-networking\shared-services-stack\` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.</span><span class="sxs-lookup"><span data-stu-id="a3922-189">Navigate to the `hybrid-networking\shared-services-stack\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="a3922-190">Öffnen Sie die Datei `onprem.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 45 und 46 ein, und speichern Sie die Datei.</span><span class="sxs-lookup"><span data-stu-id="a3922-190">Open the `onprem.json` file and enter a username and password between the quotes in line 45 and 46, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

3. <span data-ttu-id="a3922-191">Führen Sie `azbb` aus, um die simulierte lokale Umgebung wie unten gezeigt bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="a3922-191">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="a3922-192">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `onprem-vnet-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="a3922-192">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="a3922-193">Warten Sie, bis die Bereitstellung abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="a3922-193">Wait for the deployment to finish.</span></span> <span data-ttu-id="a3922-194">Bei dieser Bereitstellung werden ein virtuelles Netzwerk, ein virtueller Windows-Computer und ein VPN-Gateway erstellt.</span><span class="sxs-lookup"><span data-stu-id="a3922-194">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="a3922-195">Die Erstellung des VPN-Gateways kann mehr als 40 Minuten in Anspruch nehmen.</span><span class="sxs-lookup"><span data-stu-id="a3922-195">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="a3922-196">Azure-Hub-VNET</span><span class="sxs-lookup"><span data-stu-id="a3922-196">Azure hub VNet</span></span>

<span data-ttu-id="a3922-197">Führen Sie die folgenden Schritte durch, um das Hub-VNET bereitzustellen und eine Verbindung mit dem oben erstellten simulierten lokalen VNET herzustellen.</span><span class="sxs-lookup"><span data-stu-id="a3922-197">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="a3922-198">Öffnen Sie die Datei `hub-vnet.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 50 und 51 ein, und speichern Sie die Datei.</span><span class="sxs-lookup"><span data-stu-id="a3922-198">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 50 and 51, as shown below.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="a3922-199">Geben Sie in Zeile 52 unter `osType` entweder `Windows` oder `Linux` ein, um Windows Server 2016 Datacenter bzw. Ubuntu 16.04 als Betriebssystem für die Jumpbox zu installieren.</span><span class="sxs-lookup"><span data-stu-id="a3922-199">On line 52, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="a3922-200">Geben Sie einen gemeinsam verwendeten Schlüssel wie unten dargestellt zwischen den Anführungszeichen in Zeile 83 ein, und speichern Sie die Datei.</span><span class="sxs-lookup"><span data-stu-id="a3922-200">Enter a shared key between the quotes in line 83, as shown below, then save the file.</span></span>

   ```bash
   "sharedKey": "",
   ```

4. <span data-ttu-id="a3922-201">Führen Sie `azbb` aus, um die simulierte lokale Umgebung wie unten gezeigt bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="a3922-201">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="a3922-202">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `hub-vnet-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="a3922-202">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="a3922-203">Warten Sie, bis die Bereitstellung abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="a3922-203">Wait for the deployment to finish.</span></span> <span data-ttu-id="a3922-204">Bei dieser Bereitstellung werden ein virtuelles Netzwerk, ein virtueller Computer, ein VPN-Gateway und eine Verbindung mit dem im vorherigen Abschnitt erstellten Gateway erstellt.</span><span class="sxs-lookup"><span data-stu-id="a3922-204">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="a3922-205">Die Erstellung des VPN-Gateways kann mehr als 40 Minuten in Anspruch nehmen.</span><span class="sxs-lookup"><span data-stu-id="a3922-205">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="adds-in-azure"></a><span data-ttu-id="a3922-206">AD DS in Azure</span><span class="sxs-lookup"><span data-stu-id="a3922-206">ADDS in Azure</span></span>

<span data-ttu-id="a3922-207">Führen Sie die folgenden Schritte aus, um die AD DS-Domänencontroller in Azure bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="a3922-207">To deploy the ADDS domain controllers in Azure, perform the following steps.</span></span>

1. <span data-ttu-id="a3922-208">Öffnen Sie die Datei `hub-adds.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 14 und 15 ein, und speichern Sie die Datei.</span><span class="sxs-lookup"><span data-stu-id="a3922-208">Open the `hub-adds.json` file and enter a username and password between the quotes in lines 14 and 15, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="a3922-209">Führen Sie `azbb` aus, um die AD DS-Domänencontroller wie unten gezeigt bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="a3922-209">Run `azbb` to deploy the ADDS domain controllers as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg - l <location> -p hub-adds.json --deploy
   ```
  
   > [!NOTE]
   > <span data-ttu-id="a3922-210">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `hub-adds-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="a3922-210">If you decide to use a different resource group name (other than `hub-adds-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

   > [!NOTE]
   > <span data-ttu-id="a3922-211">Dieser Teil der Bereitstellung kann mehrere Minuten dauern, da die beiden VMs in die Domäne eingebunden werden müssen, die im simulierten lokalen Rechenzentrum gehostet wird, und anschließend AD DS darauf installiert werden muss.</span><span class="sxs-lookup"><span data-stu-id="a3922-211">This part of the deployment may take several minutes, since it requires joining the two VMs to the domain hosted int he simulated on-premises datacenter, then installing AD DS on them.</span></span>

### <a name="nva"></a><span data-ttu-id="a3922-212">NVA</span><span class="sxs-lookup"><span data-stu-id="a3922-212">NVA</span></span>

<span data-ttu-id="a3922-213">Führen Sie die folgenden Schritte aus, um eine NVA im Subnetz `dmz` bereitzustellen:</span><span class="sxs-lookup"><span data-stu-id="a3922-213">To deploy an NVA in the `dmz` subnet, perform the following steps:</span></span>

1. <span data-ttu-id="a3922-214">Öffnen Sie die Datei `hub-nva.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 13 und 14 ein, und speichern Sie die Datei.</span><span class="sxs-lookup"><span data-stu-id="a3922-214">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```
2. <span data-ttu-id="a3922-215">Führen Sie `azbb` aus, um die NVA-VM und die benutzerdefinierten Routen bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="a3922-215">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="a3922-216">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `hub-nva-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="a3922-216">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="a3922-217">Azure-Spoke-VNETs</span><span class="sxs-lookup"><span data-stu-id="a3922-217">Azure spoke VNets</span></span>

<span data-ttu-id="a3922-218">Führen Sie die folgenden Schritte aus, um die Spoke-VNETs bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="a3922-218">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="a3922-219">Öffnen Sie die Datei `spoke1.json`, geben Sie einen Benutzernamen und das Kennwort wie unten dargestellt zwischen den Anführungszeichen in Zeile 52 und 53 ein, und speichern Sie die Datei.</span><span class="sxs-lookup"><span data-stu-id="a3922-219">Open the `spoke1.json` file and enter a username and password between the quotes in lines 52 and 53, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="a3922-220">Geben Sie in Zeile 54 unter `osType` entweder `Windows` oder `Linux` ein, um Windows Server 2016 Datacenter bzw. Ubuntu 16.04 als Betriebssystem für die Jumpbox zu installieren.</span><span class="sxs-lookup"><span data-stu-id="a3922-220">On line 54, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="a3922-221">Führen Sie `azbb` aus, um die erste Spoke-VNET-Umgebung wie unten gezeigt bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="a3922-221">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
   ```
  
   > [!NOTE]
   > <span data-ttu-id="a3922-222">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `spoke1-vnet-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="a3922-222">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="a3922-223">Wiederholen Sie den obigen Schritt 1 für die Datei `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="a3922-223">Repeat step 1 above for file `spoke2.json`.</span></span>

5. <span data-ttu-id="a3922-224">Führen Sie `azbb` aus, um die zweite Spoke-VNET-Umgebung wie unten gezeigt bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="a3922-224">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="a3922-225">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `spoke2-vnet-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="a3922-225">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="a3922-226">Herstellen einer Peeringverbindung zwischen Azure-Hub-VNETs und Spoke-VNETs</span><span class="sxs-lookup"><span data-stu-id="a3922-226">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="a3922-227">Führen Sie die folgenden Schritte aus, um eine Peeringverbindung vom Hub-VNET mit den Spoke-VNETs zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="a3922-227">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="a3922-228">Öffnen Sie die Datei `hub-vnet-peering.json`, und stellen Sie sicher, dass der Ressourcengruppenname und der Name des virtuellen Netzwerks für die einzelnen VNET-Peerings ab Zeile 29 korrekt sind.</span><span class="sxs-lookup"><span data-stu-id="a3922-228">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="a3922-229">Führen Sie `azbb` aus, um die erste Spoke-VNET-Umgebung wie unten gezeigt bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="a3922-229">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
   ```

   > [!NOTE]
   > <span data-ttu-id="a3922-230">Wenn Sie sich für einen anderen Ressourcengruppennamen (außer `hub-vnet-rg`) entscheiden, sollten Sie alle Parameterdateien mit diesem Namen suchen und durch Ihren eigenen Ressourcengruppennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="a3922-230">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

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
[virtual datacenter]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "Topologie mit gemeinsamen Diensten in Azure"
[3]: ./images/hub-spokehub-spoke.svg "Hub-Spoke-Hub-Spoke-Topologie in Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
