---
title: "Implementieren einer hoch verfügbaren Hybrid-Netzwerkarchitektur"
description: "Erläutert, wie Sie eine sichere Site-to-Site-Netzwerkarchitektur implementieren, die ein virtuelles Azure-Netzwerk und ein lokales Netzwerk umfasst, die unter Verwendung von ExpressRoute mit VPB-Gatewayfailover verbunden werden."
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.prev: expressroute
cardTitle: Improving availability
ms.openlocfilehash: 4c101f17e5e91085b61178f9efb2bc5acb61189c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a><span data-ttu-id="af0c8-103">Verbinden eines lokalen Netzwerks mit Azure unter Verwendung von ExpressRoute mit VPN-Failover</span><span class="sxs-lookup"><span data-stu-id="af0c8-103">Connect an on-premises network to Azure using ExpressRoute with VPN failover</span></span>

<span data-ttu-id="af0c8-104">Diese Referenzarchitektur zeigt, wie ein lokales Netzwerk unter Verwendung von ExpressRoute mit einem virtuellen Azure-Netzwerk (VNet) verbunden wird, wobei ein Site-to-Site-VPN (Virtual Private Network) als Failoververbindung dient.</span><span class="sxs-lookup"><span data-stu-id="af0c8-104">This reference architecture shows how to connect an on-premises network to an Azure virtual network (VNet) using ExpressRoute, with a site-to-site virtual private network (VPN) as a failover connection.</span></span> <span data-ttu-id="af0c8-105">Der Datenverkehr zwischen dem lokalen Netzwerk und dem Azure-VNet wird über eine ExpressRoute-Verbindung übertragen.</span><span class="sxs-lookup"><span data-stu-id="af0c8-105">Traffic flows between the on-premises network and the Azure VNet through an ExpressRoute connection.</span></span> <span data-ttu-id="af0c8-106">Wenn die ExpressRoute-Verbindung ausfällt, wird der Datenverkehr über einen IPSec-VPN-Tunnel weitergeleitet.</span><span class="sxs-lookup"><span data-stu-id="af0c8-106">If there is a loss of connectivity in the ExpressRoute circuit, traffic is routed through an IPSec VPN tunnel.</span></span> [<span data-ttu-id="af0c8-107">**Stellen Sie diese Lösung bereit**.</span><span class="sxs-lookup"><span data-stu-id="af0c8-107">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="af0c8-108">Beachten Sie, dass die VPN-Route nur Private Peering-Verbindungen behandelt, wenn die ExpressRoute-Verbindung nicht verfügbar ist.</span><span class="sxs-lookup"><span data-stu-id="af0c8-108">Note that if the ExpressRoute circuit is unavailable, the VPN route will only handle private peering connections.</span></span> <span data-ttu-id="af0c8-109">Public Peering- und Microsoft-Peering-Verbindungen werden über das Internet geleitet.</span><span class="sxs-lookup"><span data-stu-id="af0c8-109">Public peering and Microsoft peering connections will pass over the Internet.</span></span> 

<span data-ttu-id="af0c8-110">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="af0c8-110">![[0]][0]</span></span>

<span data-ttu-id="af0c8-111">*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*</span><span class="sxs-lookup"><span data-stu-id="af0c8-111">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="af0c8-112">Architektur</span><span class="sxs-lookup"><span data-stu-id="af0c8-112">Architecture</span></span> 

<span data-ttu-id="af0c8-113">Diese Architektur besteht aus den folgenden Komponenten.</span><span class="sxs-lookup"><span data-stu-id="af0c8-113">The architecture consists of the following components.</span></span>

* <span data-ttu-id="af0c8-114">**Lokales Netzwerk**.</span><span class="sxs-lookup"><span data-stu-id="af0c8-114">**On-premises network**.</span></span> <span data-ttu-id="af0c8-115">Ein privates lokales Netzwerk innerhalb einer Organisation.</span><span class="sxs-lookup"><span data-stu-id="af0c8-115">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="af0c8-116">**VPN-Gerät**.</span><span class="sxs-lookup"><span data-stu-id="af0c8-116">**VPN appliance**.</span></span> <span data-ttu-id="af0c8-117">Ein Gerät oder ein Dienst, der externe Konnektivität mit dem lokalen Netzwerk bereitstellt.</span><span class="sxs-lookup"><span data-stu-id="af0c8-117">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="af0c8-118">Bei dem VPN-Gerät kann es sich um ein Hardwaregerät oder eine Softwarelösung, z.B. den Routing- und RAS-Dienst unter Windows Server 2012, handeln.</span><span class="sxs-lookup"><span data-stu-id="af0c8-118">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="af0c8-119">Eine Liste der unterstützten VPN-Geräte und Informationen zur Konfiguration ausgewählter VPN-Geräte für die Verbindung mit Azure finden Sie unter [Informationen zu VPN-Geräten für VPN-Gatewayverbindungen zwischen Standorten][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="af0c8-119">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="af0c8-120">**ExpressRoute-Verbindung**.</span><span class="sxs-lookup"><span data-stu-id="af0c8-120">**ExpressRoute circuit**.</span></span> <span data-ttu-id="af0c8-121">Eine vom Konnektivitätsanbieter bereitgestellte Layer 2- oder Layer 3-Verbindung, die das lokale Netzwerk über die Edgerouter mit Azure verbindet.</span><span class="sxs-lookup"><span data-stu-id="af0c8-121">A layer 2 or layer 3 circuit supplied by the connectivity provider that joins the on-premises network with Azure through the edge routers.</span></span> <span data-ttu-id="af0c8-122">Für die Verbindung wird die vom Konnektivitätsanbieter verwaltete Hardwareinfrastruktur verwendet.</span><span class="sxs-lookup"><span data-stu-id="af0c8-122">The circuit uses the hardware infrastructure managed by the connectivity provider.</span></span>

* <span data-ttu-id="af0c8-123">**ExpressRoute-Gateway für virtuelle Netzwerke**.</span><span class="sxs-lookup"><span data-stu-id="af0c8-123">**ExpressRoute virtual network gateway**.</span></span> <span data-ttu-id="af0c8-124">Das ExpressRoute-Gateway für virtuelle Netzwerke ermöglicht es dem VNet, mit der ExpressRoute-Leitung eine Verbindung herzustellen, die für die Konnektivität mit Ihrem lokalen Netzwerk verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="af0c8-124">The ExpressRoute virtual network gateway enables the VNet to connect to the ExpressRoute circuit used for connectivity with your on-premises network.</span></span>

* <span data-ttu-id="af0c8-125">**VPN-Gateway für virtuelle Netzwerke**.</span><span class="sxs-lookup"><span data-stu-id="af0c8-125">**VPN virtual network gateway**.</span></span> <span data-ttu-id="af0c8-126">Die VPN-Gateway für virtuelle Netzwerke ermöglicht es dem VNet, eine Verbindung mit dem VPN-Gerät im lokalen Netzwerk herzustellen.</span><span class="sxs-lookup"><span data-stu-id="af0c8-126">The VPN virtual network gateway enables the VNet to connect to the VPN appliance in the on-premises network.</span></span> <span data-ttu-id="af0c8-127">Die VPN-Gateway für virtuelle Netzwerke ist so konfiguriert, dass es Anforderungen aus dem lokalen Netzwerk nur über das VPN-Gerät akzeptiert.</span><span class="sxs-lookup"><span data-stu-id="af0c8-127">The VPN virtual network gateway is configured to accept requests from the on-premises network only through the VPN appliance.</span></span> <span data-ttu-id="af0c8-128">Weitere Informationen finden Sie unter [Verbinden eines lokalen Netzwerks mit einem Microsoft Azure Virtual Network][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="af0c8-128">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

* <span data-ttu-id="af0c8-129">**VPN-Verbindung**.</span><span class="sxs-lookup"><span data-stu-id="af0c8-129">**VPN connection**.</span></span> <span data-ttu-id="af0c8-130">Die Verbindung verfügt über Eigenschaften, die den Verbindungstyp (IPSec) und den Schlüssel angeben, der für das lokale VPN-Gerät freigegeben wird, um Datenverkehr zu verschlüsseln.</span><span class="sxs-lookup"><span data-stu-id="af0c8-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>

* <span data-ttu-id="af0c8-131">**Azure Virtual Network (VNet)**.</span><span class="sxs-lookup"><span data-stu-id="af0c8-131">**Azure Virtual Network (VNet)**.</span></span> <span data-ttu-id="af0c8-132">Jedes VNet befindet sich in einer einzelnen Azure-Region und kann mehrere Logikschichten hosten.</span><span class="sxs-lookup"><span data-stu-id="af0c8-132">Each VNet resides in a single Azure region, and can host multiple application tiers.</span></span> <span data-ttu-id="af0c8-133">Logikschichten können mithilfe von Subnetzen in jedem VNet segmentiert werden.</span><span class="sxs-lookup"><span data-stu-id="af0c8-133">Application tiers can be segmented using subnets in each VNet.</span></span>

* <span data-ttu-id="af0c8-134">**Gatewaysubnetz**.</span><span class="sxs-lookup"><span data-stu-id="af0c8-134">**Gateway subnet**.</span></span> <span data-ttu-id="af0c8-135">Die Gateways für virtuelle Netzwerke befinden sich in demselben Subnetz.</span><span class="sxs-lookup"><span data-stu-id="af0c8-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="af0c8-136">**Cloudanwendung**.</span><span class="sxs-lookup"><span data-stu-id="af0c8-136">**Cloud application**.</span></span> <span data-ttu-id="af0c8-137">Die in Azure gehostete Anwendung.</span><span class="sxs-lookup"><span data-stu-id="af0c8-137">The application hosted in Azure.</span></span> <span data-ttu-id="af0c8-138">Sie kann mehrere Schichten umfassen, wobei mehrere Subnetze über Azure Load Balancer verbunden sind.</span><span class="sxs-lookup"><span data-stu-id="af0c8-138">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="af0c8-139">Weitere Informationen zur Anwendungsinfrastruktur finden Sie unter [Ausführen von Windows-VM-Workloads][windows-vm-ra] und [Ausführen von Linux-VM-Workloads][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="af0c8-139">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

## <a name="recommendations"></a><span data-ttu-id="af0c8-140">Recommendations</span><span class="sxs-lookup"><span data-stu-id="af0c8-140">Recommendations</span></span>

<span data-ttu-id="af0c8-141">Die folgenden Empfehlungen gelten für die meisten Szenarios.</span><span class="sxs-lookup"><span data-stu-id="af0c8-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="af0c8-142">Sofern Sie keine besonderen Anforderungen haben, die Vorrang haben, sollten Sie diese Empfehlungen befolgen.</span><span class="sxs-lookup"><span data-stu-id="af0c8-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="af0c8-143">VNet und GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="af0c8-143">VNet and GatewaySubnet</span></span>

<span data-ttu-id="af0c8-144">Erstellen Sie das ExpressRoute-Gateway für virtuelle Netzwerke und das VPN-Gateway für virtuelle Netzwerke in demselben VNet.</span><span class="sxs-lookup"><span data-stu-id="af0c8-144">Create the ExpressRoute virtual network gateway and the VPN virtual network gateway in the same VNet.</span></span> <span data-ttu-id="af0c8-145">Dies bedeutet, dass sie das Subnetz mit dem Namen *GatewaySubnet* gemeinsam nutzen sollten.</span><span class="sxs-lookup"><span data-stu-id="af0c8-145">This means that they should share the same subnet named *GatewaySubnet*.</span></span>

<span data-ttu-id="af0c8-146">Wenn das VNet bereits ein Subnetz mit dem Namen *GatewaySubnet* enthält, stellen Sie sicher, dass es einen /27-Adressraum oder einen größeren Adressraum aufweist.</span><span class="sxs-lookup"><span data-stu-id="af0c8-146">If the VNet already includes a subnet named *GatewaySubnet*, ensure that it has a /27 or larger address space.</span></span> <span data-ttu-id="af0c8-147">Wenn das vorhandene Subnetz zu klein ist, entfernen Sie es mit dem folgenden PowerShell-Befehl:</span><span class="sxs-lookup"><span data-stu-id="af0c8-147">If the existing subnet is too small, use the following PowerShell command to remove the subnet:</span></span> 

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

<span data-ttu-id="af0c8-148">Wenn das VNet kein Subnetz mit dem Namen **GatewaySubnet** enthält, erstellen Sie mit dem folgenden Powershell-Befehl ein neues Subnetz:</span><span class="sxs-lookup"><span data-stu-id="af0c8-148">If the VNet does not contain a subnet named **GatewaySubnet**, create a new one using the following Powershell command:</span></span>

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a><span data-ttu-id="af0c8-149">VPN- und ExpressRoute-Gateways</span><span class="sxs-lookup"><span data-stu-id="af0c8-149">VPN and ExpressRoute gateways</span></span>

<span data-ttu-id="af0c8-150">Stellen Sie sicher, dass Ihre Organisation die [ExpressRoute-Voraussetzungen][expressroute-prereq] zum Herstellen der Verbindung mit Azure erfüllt.</span><span class="sxs-lookup"><span data-stu-id="af0c8-150">Verify that your organization meets the [ExpressRoute prerequisite requirements][expressroute-prereq] for connecting to Azure.</span></span>

<span data-ttu-id="af0c8-151">Wenn Sie bereits im Azure-VNet über ein VPN-Gateway für virtuelle Netzwerke verfügen, entfernen Sie es mit dem folgenden Powershell-Befehl:</span><span class="sxs-lookup"><span data-stu-id="af0c8-151">If you already have a VPN virtual network gateway in your Azure VNet, use the following  Powershell command to remove it:</span></span>

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

<span data-ttu-id="af0c8-152">Befolgen Sie die Anweisungen in [Implementieren einer sicheren Hybrid-Netzwerkarchitektur mit Azure ExpressRoute][implementing-expressroute], um eine sichere ExpressRoute-Verbindung herzustellen.</span><span class="sxs-lookup"><span data-stu-id="af0c8-152">Follow the instructions in [Implementing a hybrid network architecture with Azure ExpressRoute][implementing-expressroute] to establish your ExpressRoute connection.</span></span>

<span data-ttu-id="af0c8-153">Befolgen Sie die Anweisungen in [Implementieren einer sicheren Hybrid-Netzwerkarchitektur mit Azure und lokalem VPN][implementing-vpn], um eine Verbindung mit dem VPN-Gateway für virtuelle Netzwerke herzustellen.</span><span class="sxs-lookup"><span data-stu-id="af0c8-153">Follow the instructions in [Implementing a hybrid network architecture with Azure and On-premises VPN][implementing-vpn] to establish your VPN virtual network gateway connection.</span></span>

<span data-ttu-id="af0c8-154">Nachdem Sie die Verbindung mit dem Gateway für virtuelle Netzwerke hergestellt haben, testen Sie die Umgebung wie folgt:</span><span class="sxs-lookup"><span data-stu-id="af0c8-154">After you have established the virtual network gateway connections, test the environment as follows:</span></span>

1. <span data-ttu-id="af0c8-155">Stellen Sie sicher, dass Sie vom lokalen Netzwerk eine Verbindung mit dem Azure-VNet herstellen können.</span><span class="sxs-lookup"><span data-stu-id="af0c8-155">Make sure you can connect from your on-premises network to your Azure VNet.</span></span>
2. <span data-ttu-id="af0c8-156">Wenden Sie sich an Ihren Anbieter, um die ExpressRoute-Konnektivität zu Testzwecken zu beenden.</span><span class="sxs-lookup"><span data-stu-id="af0c8-156">Contact your provider to stop ExpressRoute connectivity for testing.</span></span>
3. <span data-ttu-id="af0c8-157">Stellen Sie sicher, dass Sie weiterhin über das VPN-Gateway für virtuelle Verbindungen eine Verbindung vom lokalen Netzwerk mit dem Azure-VNet herstellen können.</span><span class="sxs-lookup"><span data-stu-id="af0c8-157">Verify that you can still connect from your on-premises network to your Azure VNet using the VPN virtual network gateway connection.</span></span>
4. <span data-ttu-id="af0c8-158">Wenden Sie sich an Ihren Anbieter, um die ExpressRoute-Konnektivität wiederherzustellen.</span><span class="sxs-lookup"><span data-stu-id="af0c8-158">Contact your provider to reestablish ExpressRoute connectivity.</span></span>

## <a name="considerations"></a><span data-ttu-id="af0c8-159">Überlegungen</span><span class="sxs-lookup"><span data-stu-id="af0c8-159">Considerations</span></span>

<span data-ttu-id="af0c8-160">Überlegungen zu ExpressRoute finden Sie im Leitfaden [Implementieren einer sicheren Hybrid-Netzwerkarchitektur mit Azure ExpressRoute][guidance-expressroute].</span><span class="sxs-lookup"><span data-stu-id="af0c8-160">For ExpressRoute considerations, see the [Implementing a Hybrid Network Architecture with Azure ExpressRoute][guidance-expressroute] guidance.</span></span>

<span data-ttu-id="af0c8-161">Überlegungen zu Site-to-Site-VPNs finden Sie im Leitfaden [Implementieren einer sicheren Hybrid-Netzwerkarchitektur mit Azure und lokalem VPN][guidance-vpn].</span><span class="sxs-lookup"><span data-stu-id="af0c8-161">For site-to-site VPN considerations, see the [Implementing a Hybrid Network Architecture with Azure and On-premises VPN][guidance-vpn] guidance.</span></span>

<span data-ttu-id="af0c8-162">Informationen zu allgemeinen Sicherheitsaspekten für Azure finden Sie unter [Microsoft-Clouddienste und Netzwerksicherheit][best-practices-security].</span><span class="sxs-lookup"><span data-stu-id="af0c8-162">For general Azure security considerations, see [Microsoft cloud services and network security][best-practices-security].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="af0c8-163">Bereitstellen der Lösung</span><span class="sxs-lookup"><span data-stu-id="af0c8-163">Deploy the solution</span></span>

<span data-ttu-id="af0c8-164">**Voraussetzungen**</span><span class="sxs-lookup"><span data-stu-id="af0c8-164">**Prequisites.**</span></span> <span data-ttu-id="af0c8-165">Sie müssen über eine lokale Infrastruktur verfügen, die bereits mit einem geeigneten Netzwerkgerät konfiguriert ist.</span><span class="sxs-lookup"><span data-stu-id="af0c8-165">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="af0c8-166">Führen Sie die folgenden Schritte aus, um die Lösung bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="af0c8-166">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="af0c8-167">Klicken Sie auf die Schaltfläche unten:</span><span class="sxs-lookup"><span data-stu-id="af0c8-167">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="af0c8-168">Warten Sie, bis der Link im Azure-Portal geöffnet wird, und gehen Sie dann folgendermaßen vor:</span><span class="sxs-lookup"><span data-stu-id="af0c8-168">Wait for the link to open in the Azure portal, then follow these steps:</span></span>   
   * <span data-ttu-id="af0c8-169">Der Name der **Ressourcengruppe** ist bereits in der Parameterdatei definiert. Wählen Sie also **Neu erstellen**, und geben Sie im Textfeld `ra-hybrid-vpn-er-rg` ein.</span><span class="sxs-lookup"><span data-stu-id="af0c8-169">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="af0c8-170">Wählen Sie im Dropdownfeld **Standort** die Region aus.</span><span class="sxs-lookup"><span data-stu-id="af0c8-170">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="af0c8-171">Lassen Sie die Textfelder für den **Vorlagenstamm-URI** bzw. **Parameterstamm-URI** unverändert.</span><span class="sxs-lookup"><span data-stu-id="af0c8-171">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="af0c8-172">Überprüfen Sie die allgemeinen Geschäftsbedingungen, und aktivieren Sie dann das Kontrollkästchen **Ich stimme den oben genannten Geschäftsbedingungen zu**.</span><span class="sxs-lookup"><span data-stu-id="af0c8-172">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="af0c8-173">Klicken Sie auf die Schaltfläche **Kaufen**.</span><span class="sxs-lookup"><span data-stu-id="af0c8-173">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="af0c8-174">Warten Sie, bis die Bereitstellung abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="af0c8-174">Wait for the deployment to complete.</span></span>
4. <span data-ttu-id="af0c8-175">Klicken Sie auf die Schaltfläche unten:</span><span class="sxs-lookup"><span data-stu-id="af0c8-175">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. <span data-ttu-id="af0c8-176">Warten Sie, bis der Link im Azure-Portal geöffnet wird, und gehen Sie dann folgendermaßen vor:</span><span class="sxs-lookup"><span data-stu-id="af0c8-176">Wait for the link to open in the Azure portal, then enter then follow these steps:</span></span>
   * <span data-ttu-id="af0c8-177">Wählen Sie **Vorhandene verwenden** im Abschnitt **Ressourcengruppe** aus, und geben Sie im Textfeld `ra-hybrid-vpn-er-rg` ein.</span><span class="sxs-lookup"><span data-stu-id="af0c8-177">Select **Use existing** in the **Resource group** section and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="af0c8-178">Wählen Sie im Dropdownfeld **Standort** die Region aus.</span><span class="sxs-lookup"><span data-stu-id="af0c8-178">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="af0c8-179">Lassen Sie die Textfelder für den **Vorlagenstamm-URI** bzw. **Parameterstamm-URI** unverändert.</span><span class="sxs-lookup"><span data-stu-id="af0c8-179">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="af0c8-180">Überprüfen Sie die allgemeinen Geschäftsbedingungen, und aktivieren Sie dann das Kontrollkästchen **Ich stimme den oben genannten Geschäftsbedingungen zu**.</span><span class="sxs-lookup"><span data-stu-id="af0c8-180">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="af0c8-181">Klicken Sie auf die Schaltfläche **Kaufen**.</span><span class="sxs-lookup"><span data-stu-id="af0c8-181">Click the **Purchase** button.</span></span>

<!-- links -->

[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md


[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[expressroute-prereq]: /azure/expressroute/expressroute-prerequisites
[implementing-expressroute]: ./expressroute.md
[implementing-vpn]: ./vpn.md
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[best-practices-security]: /azure/best-practices-network-security
[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-architectures.vsdx
[0]: ./images/expressroute-vpn-failover.png "Hoch verfügbare Hybrid-Netzwerkarchitektur mit ExpressRoute- und VPN-Gateway"
