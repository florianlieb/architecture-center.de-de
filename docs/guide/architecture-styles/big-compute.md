---
title: Big Compute-Architekturstil
description: "Es werden die Vorteile, Herausforderungen und bewährten Methoden für Big Compute-Architekturen in Azure beschrieben."
author: MikeWasson
ms.openlocfilehash: b16be4133143d7d73062eeb280b44779c390f387
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="big-compute-architecture-style"></a><span data-ttu-id="b51fa-103">Big Compute-Architekturstil</span><span class="sxs-lookup"><span data-stu-id="b51fa-103">Big compute architecture style</span></span>

<span data-ttu-id="b51fa-104">Mit dem Begriff *Big Compute* werden umfangreiche Workloads beschrieben, für die eine große Anzahl von Kernen erforderlich ist (häufig Hunderte oder Tausende).</span><span class="sxs-lookup"><span data-stu-id="b51fa-104">The term *big compute* describes large-scale workloads that require a large number of cores, often numbering in the hundreds or thousands.</span></span> <span data-ttu-id="b51fa-105">Zu den Szenarien gehören unter anderem Bildrendering, Fluiddynamik, Modellierung von finanziellen Risiken, Erdölsuche, Arzneimittelentwicklung und Belastungsanalysen im Maschinenbau.</span><span class="sxs-lookup"><span data-stu-id="b51fa-105">Scenarios include image rendering, fluid dynamics, financial risk modeling, oil exploration, drug design, and engineering stress analysis, among others.</span></span>

![](./images/big-compute-logical.png)

<span data-ttu-id="b51fa-106">Hier sind einige typische Merkmale von Big Compute-Anwendungen aufgeführt:</span><span class="sxs-lookup"><span data-stu-id="b51fa-106">Here are some typical characteristics of big compute applications:</span></span>

- <span data-ttu-id="b51fa-107">Die Arbeit kann in einzelne Aufgaben unterteilt werden, die dann gleichzeitig auf vielen Kernen ausgeführt werden können.</span><span class="sxs-lookup"><span data-stu-id="b51fa-107">The work can be split into discrete tasks, which can be run across many cores simultaneously.</span></span>
- <span data-ttu-id="b51fa-108">Jede Aufgabe ist endlich.</span><span class="sxs-lookup"><span data-stu-id="b51fa-108">Each task is finite.</span></span> <span data-ttu-id="b51fa-109">Es ist eine Eingabe erforderlich, anschließend erfolgt die Verarbeitung, und zuletzt wird die Ausgabe erstellt.</span><span class="sxs-lookup"><span data-stu-id="b51fa-109">It takes some input, does some processing, and produces output.</span></span> <span data-ttu-id="b51fa-110">Die gesamte Anwendung wird über einen endlichen Zeitraum ausgeführt (Minuten bis Tage).</span><span class="sxs-lookup"><span data-stu-id="b51fa-110">The entire application runs for a finite amount of time (minutes to days).</span></span> <span data-ttu-id="b51fa-111">Ein häufiges Muster ist die Bereitstellung einer großen Anzahl von Kernen in einem Burstvorgang und das anschließende Herunterfahren auf null, nachdem die Anwendung beendet wurde.</span><span class="sxs-lookup"><span data-stu-id="b51fa-111">A common pattern is to provision a large number of cores in a burst, and then spin down to zero once the application completes.</span></span> 
- <span data-ttu-id="b51fa-112">Die Anwendung muss nicht rund um die Uhr aktiv sein.</span><span class="sxs-lookup"><span data-stu-id="b51fa-112">The application does not need to stay up 24/7.</span></span> <span data-ttu-id="b51fa-113">Das System muss aber Knotenfehler oder Anwendungsabstürze verarbeiten können.</span><span class="sxs-lookup"><span data-stu-id="b51fa-113">However, the system must handle node failures or application crashes.</span></span>
- <span data-ttu-id="b51fa-114">Bei einigen Anwendungen sind Aufgaben unabhängig voneinander und können parallel ausgeführt werden.</span><span class="sxs-lookup"><span data-stu-id="b51fa-114">For some applications, tasks are independent and can run in parallel.</span></span> <span data-ttu-id="b51fa-115">In anderen Fällen sind Aufgaben eng verknüpft und müssen ggf. interagieren oder Zwischenergebnisse austauschen.</span><span class="sxs-lookup"><span data-stu-id="b51fa-115">In other cases, tasks are tightly coupled, meaning they must interact or exchange intermediate results.</span></span> <span data-ttu-id="b51fa-116">Hierfür kann die Nutzung von Hochgeschwindigkeits-Netzwerktechnologie ratsam sein, z.B. InfiniBand und Remotezugriff auf den direkten Speicher (Remote Direct Memory Access, RDMA).</span><span class="sxs-lookup"><span data-stu-id="b51fa-116">In that case, consider using high-speed networking technologies such as InfiniBand and remote direct memory access (RDMA).</span></span> 
- <span data-ttu-id="b51fa-117">Je nach Workload kann auch der Einsatz von rechenintensiven VM-Größen (H16r, H16mr und A9) ratsam sein.</span><span class="sxs-lookup"><span data-stu-id="b51fa-117">Depending on your workload, you might use compute-intensive VM sizes (H16r, H16mr, and A9).</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="b51fa-118">Einsatzmöglichkeiten für diese Architektur</span><span class="sxs-lookup"><span data-stu-id="b51fa-118">When to use this architecture</span></span>

- <span data-ttu-id="b51fa-119">Rechenintensive Vorgänge, z.B. Simulation und Zahlenverarbeitung.</span><span class="sxs-lookup"><span data-stu-id="b51fa-119">Computationally intensive operations such as simulation and number crunching.</span></span>
- <span data-ttu-id="b51fa-120">Simulationen, die rechenintensiv sind und auf CPUs mehrerer Computer (10 bis mehrere Tausend) aufgeteilt werden müssen.</span><span class="sxs-lookup"><span data-stu-id="b51fa-120">Simulations that are computationally intensive and must be split across CPUs in multiple computers (10-1000s).</span></span>
- <span data-ttu-id="b51fa-121">Simulationen, für die zu viel Arbeitsspeicher für einen Computer erforderlich ist und die auf mehrere Computer aufgeteilt werden müssen.</span><span class="sxs-lookup"><span data-stu-id="b51fa-121">Simulations that require too much memory for one computer, and must be split across multiple computers.</span></span>
- <span data-ttu-id="b51fa-122">Rechenvorgänge mit langer Ausführungsdauer, die auf einem einzelnen Computer zu lange dauern würden.</span><span class="sxs-lookup"><span data-stu-id="b51fa-122">Long-running computations that would take too long to complete on a single computer.</span></span>
- <span data-ttu-id="b51fa-123">Kleinere Rechenvorgänge, die mehrere hundert oder tausend Mal ausgeführt werden müssen, z.B. Monte Carlo-Simulationen.</span><span class="sxs-lookup"><span data-stu-id="b51fa-123">Smaller computations that must be run 100s or 1000s of times, such as Monte Carlo simulations.</span></span>

## <a name="benefits"></a><span data-ttu-id="b51fa-124">Vorteile</span><span class="sxs-lookup"><span data-stu-id="b51fa-124">Benefits</span></span>

- <span data-ttu-id="b51fa-125">Hohe Leistung mit „[hochgradig paralleler][embarrassingly-parallel]“ Verarbeitung.</span><span class="sxs-lookup"><span data-stu-id="b51fa-125">High performance with "[embarrassingly parallel][embarrassingly-parallel]" processing.</span></span>
- <span data-ttu-id="b51fa-126">Es können Hunderte oder Tausende von Computerprozessorkernen genutzt werden, um komplexe Probleme schneller zu lösen.</span><span class="sxs-lookup"><span data-stu-id="b51fa-126">Can harness hundreds or thousands of computer cores to solve large problems faster.</span></span>
- <span data-ttu-id="b51fa-127">Zugriff auf spezialisierte Hochleistungs-Hardware mit dedizierten InfiniBand-Hochgeschwindigkeitsnetzwerken.</span><span class="sxs-lookup"><span data-stu-id="b51fa-127">Access to specialized high-performance hardware, with dedicated high-speed InfiniBand networks.</span></span>
- <span data-ttu-id="b51fa-128">Sie können VMs je nach Bedarf für Arbeitsschritte bereitstellen und dann wieder aussondern.</span><span class="sxs-lookup"><span data-stu-id="b51fa-128">You can provision VMs as needed to do work, and then tear them down.</span></span> 

## <a name="challenges"></a><span data-ttu-id="b51fa-129">Herausforderungen</span><span class="sxs-lookup"><span data-stu-id="b51fa-129">Challenges</span></span>

- <span data-ttu-id="b51fa-130">Die Verwaltung der VM-Infrastruktur.</span><span class="sxs-lookup"><span data-stu-id="b51fa-130">Managing the VM infrastructure.</span></span>
- <span data-ttu-id="b51fa-131">Die Verwaltung des Umfangs der Zahlenverarbeitung.</span><span class="sxs-lookup"><span data-stu-id="b51fa-131">Managing the volume of number crunching.</span></span> 
- <span data-ttu-id="b51fa-132">Die rechtzeitige Bereitstellung von Tausenden von Kernen.</span><span class="sxs-lookup"><span data-stu-id="b51fa-132">Provisioning thousands of cores in a timely manner.</span></span>
- <span data-ttu-id="b51fa-133">Für eng verknüpfte Aufgaben kann das Hinzufügen von mehr Kernen auch negative Auswirkungen haben.</span><span class="sxs-lookup"><span data-stu-id="b51fa-133">For tightly coupled tasks, adding more cores can have diminishing returns.</span></span> <span data-ttu-id="b51fa-134">Unter Umständen müssen Sie etwas experimentieren, um die optimale Anzahl von Kernen zu ermitteln.</span><span class="sxs-lookup"><span data-stu-id="b51fa-134">You may need to experiment to find the optimum number of cores.</span></span>

## <a name="big-compute-using-azure-batch"></a><span data-ttu-id="b51fa-135">Big Compute per Azure Batch</span><span class="sxs-lookup"><span data-stu-id="b51fa-135">Big compute using Azure Batch</span></span>

<span data-ttu-id="b51fa-136">[Azure Batch][batch] ist ein verwalteter Dienst zum Ausführen von umfassenden HPC-Anwendungen (High-Performance Computing).</span><span class="sxs-lookup"><span data-stu-id="b51fa-136">[Azure Batch][batch] is a managed service for running large-scale high-performance computing (HPC) applications.</span></span>

<span data-ttu-id="b51fa-137">Mit Azure Batch konfigurieren Sie einen VM-Pool und laden die Anwendungen und Datendateien hoch.</span><span class="sxs-lookup"><span data-stu-id="b51fa-137">Using Azure Batch, you configure a VM pool, and upload the applications and data files.</span></span> <span data-ttu-id="b51fa-138">Anschließend stellt der Batch-Dienst die VMs bereit, weist den VMs Aufgaben zu, führt die Aufgaben aus und überwacht den Fortschritt.</span><span class="sxs-lookup"><span data-stu-id="b51fa-138">Then the Batch service provisions the VMs, assign tasks to the VMs, runs the tasks, and monitors the progress.</span></span> <span data-ttu-id="b51fa-139">Batch kann die VMs als Reaktion auf die Workload automatisch horizontal hochskalieren.</span><span class="sxs-lookup"><span data-stu-id="b51fa-139">Batch can automatically scale out the VMs in response to the workload.</span></span> <span data-ttu-id="b51fa-140">Batch ermöglicht auch eine Auftragsplanung.</span><span class="sxs-lookup"><span data-stu-id="b51fa-140">Batch also provides job scheduling.</span></span>

![](./images/big-compute-batch.png) 

## <a name="big-compute-running-on-virtual-machines"></a><span data-ttu-id="b51fa-141">Big Compute auf virtuellen Computern</span><span class="sxs-lookup"><span data-stu-id="b51fa-141">Big compute running on Virtual Machines</span></span>

<span data-ttu-id="b51fa-142">Sie können [Microsoft HPC Pack][hpc-pack] verwenden, um einen Cluster mit VMs zu verwalten und HPC-Aufträge zu planen und zu überwachen.</span><span class="sxs-lookup"><span data-stu-id="b51fa-142">You can use [Microsoft HPC Pack][hpc-pack] to administer a cluster of VMs, and schedule and monitor HPC jobs.</span></span> <span data-ttu-id="b51fa-143">Bei diesem Ansatz müssen Sie die VMs und die Netzwerkinfrastruktur bereitstellen und verwalten.</span><span class="sxs-lookup"><span data-stu-id="b51fa-143">With this approach, you must provision and manage the VMs and network infrastructure.</span></span> <span data-ttu-id="b51fa-144">Erwägen Sie diesen Ansatz, wenn Sie über vorhandene HPC-Workloads verfügen und einige oder alle nach Azure verschieben möchten.</span><span class="sxs-lookup"><span data-stu-id="b51fa-144">Consider this approach if you have existing HPC workloads and want to move some or all it to Azure.</span></span> <span data-ttu-id="b51fa-145">Sie können den gesamten HPC-Cluster nach Azure verschieben oder Ihren HPC-Cluster weiter lokal nutzen und Azure für Burstkapazität-Zwecke verwenden.</span><span class="sxs-lookup"><span data-stu-id="b51fa-145">You can move the entire HPC cluster to Azure, or keep your HPC cluster on-premises but use Azure for burst capacity.</span></span> <span data-ttu-id="b51fa-146">Weitere Informationen finden Sie unter [HPC-, Batch- und Big Compute-Lösungen, mit Azure-VMs][batch-hpc-solutions].</span><span class="sxs-lookup"><span data-stu-id="b51fa-146">For more information, see [Batch and HPC solutions for large-scale computing workloads][batch-hpc-solutions].</span></span>

### <a name="hpc-pack-deployed-to-azure"></a><span data-ttu-id="b51fa-147">Bereitstellung von HPC Pack in Azure</span><span class="sxs-lookup"><span data-stu-id="b51fa-147">HPC Pack deployed to Azure</span></span>

<span data-ttu-id="b51fa-148">Bei diesem Szenario wird der HPC-Cluster vollständig in Azure erstellt.</span><span class="sxs-lookup"><span data-stu-id="b51fa-148">In this scenario, the HPC cluster is created entirely within Azure.</span></span>

![](./images/big-compute-iaas.png) 
 
<span data-ttu-id="b51fa-149">Der Hauptknoten stellt Verwaltungs- und Auftragsplanungsdienste für den Cluster bereit.</span><span class="sxs-lookup"><span data-stu-id="b51fa-149">The head node provides management and job scheduling services to the cluster.</span></span> <span data-ttu-id="b51fa-150">Verwenden Sie für eng verknüpfte Aufgaben ein RDMA-Netzwerk, das zwischen VMs eine Kommunikation mit sehr hoher Bandbreite und geringer Wartezeit ermöglicht.</span><span class="sxs-lookup"><span data-stu-id="b51fa-150">For tightly coupled tasks, use an RDMA network that provides very high bandwidth, low latency communication between VMs.</span></span> <span data-ttu-id="b51fa-151">Weitere Informationen finden Sie unter [Bereitstellen eines HPC Pack 2016-Clusters in Azure][deploy-hpc-azure].</span><span class="sxs-lookup"><span data-stu-id="b51fa-151">For more information see [Deploy an HPC Pack 2016 cluster in Azure][deploy-hpc-azure].</span></span>

### <a name="burst-an-hpc-cluster-to-azure"></a><span data-ttu-id="b51fa-152">Burst eines HPC-Clusters in Azure</span><span class="sxs-lookup"><span data-stu-id="b51fa-152">Burst an HPC cluster to Azure</span></span>

<span data-ttu-id="b51fa-153">Bei diesem Szenario wird HPC Pack in einer Organisation lokal ausgeführt, und Azure-VMs werden für Burstkapazitäts-Zwecke genutzt.</span><span class="sxs-lookup"><span data-stu-id="b51fa-153">In this scenario, an organization is running HPC Pack on-premises, and uses Azure VMs for burst capacity.</span></span> <span data-ttu-id="b51fa-154">Der Hauptknoten des Clusters ist lokal angeordnet.</span><span class="sxs-lookup"><span data-stu-id="b51fa-154">The cluster head node is on-premises.</span></span> <span data-ttu-id="b51fa-155">Per ExpressRoute oder VPN Gateway wird für das lokale Netzwerk eine Verbindung mit dem Azure-VNet hergestellt.</span><span class="sxs-lookup"><span data-stu-id="b51fa-155">ExpressRoute or VPN Gateway connects the on-premises network to the Azure VNet.</span></span>

![](./images/big-compute-hybrid.png) 


[batch]: /azure/batch/
[batch-hpc-solutions]: /azure/batch/batch-hpc-solutions
[deploy-hpc-azure]: /azure/virtual-machines/windows/hpcpack-2016-cluster
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hpc-pack]: https://technet.microsoft.com/library/cc514029

 
