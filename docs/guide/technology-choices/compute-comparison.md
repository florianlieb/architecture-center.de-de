---
title: Kriterien für die Auswahl einer Azure-Compute-Option
description: Vergleichen von Azure-Computediensten unter verschiedenen Aspekten
author: MikeWasson
layout: LandingPage
ms.date: 04/21/2018
ms.openlocfilehash: ff90ec41c56ae0ecb81bc82128f02fd06d02cb32
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/23/2018
---
# <a name="criteria-for-choosing-an-azure-compute-service"></a>Kriterien für die Auswahl einer Azure-Compute-Option

Der Begriff *Compute* bezieht sich auf das Hostingmodell für die Computeressourcen, auf denen Ihre Anwendungen ausgeführt werden. Die folgenden Tabellen vergleichen Azure-Computedienste unter verschiedenen Aspekten miteinander. Ziehen Sie diese Tabellen zu Rate, wenn Sie eine Compute-Option für Ihre Anwendung auswählen.

## <a name="hosting-model"></a>Hostingmodell

| Kriterien | Virtual Machines | App Service | Service Fabric | Azure-Funktionen | Azure Container Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Anwendungskomposition | Agnostisch | ANWENDUNGEN | Dienste, ausführbare Gastdateien, Container | Funktionen | Container | Container | Geplante Aufträge  |
| Dichte | Agnostisch | Mehrere Apps pro Instanz über App-Pläne | Mehrere Dienste pro VM | Keine dedizierten Instanzen<a href="#note1"><sup>1</sup></a> | Mehrere Container pro VM |Keine dedizierten Instanzen | Mehrere Apps pro VM |
| Mindestanzahl von Knoten | 1<a href="#note2"><sup>2</sup></a>  | 1 | 5<a href="#note3"><sup>3</sup></a> | Keine dedizierten Knoten<a href="#note1"><sup>1</sup></a> | 3 | Keine dedizierten Knoten | 1<a href="#note4"><sup>4</sup></a> |
| Zustandsverwaltung | Zustandslos oder zustandsbehaftet | Zustandslos | Zustandslos oder zustandsbehaftet | Zustandslos | Zustandslos oder zustandsbehaftet | Zustandslos | Zustandslos |
| Webhosting | Agnostisch | Integriert | Agnostisch | Nicht zutreffend | Agnostisch | Agnostisch | Nein  |
| Betriebssystem | Windows, Linux | Windows, Linux  | Windows, Linux | Nicht zutreffend | Windows (Vorschau), Linux | Windows, Linux | Windows, Linux |
| Bereitstellung in dediziertem VNET möglich? | Unterstützt | Unterstützt<a href="#note5"><sup>5</sup></a> | Unterstützt | Nicht unterstützt | Unterstützt | Nicht unterstützt | Unterstützt |
| Hybridkonnektivität | Unterstützt | Unterstützt<a href="#note1"><sup>6</sup></a>  | Unterstützt | Nicht unterstützt | Unterstützt | Nicht unterstützt | Unterstützt |

Notizen

1. <span id="note1">Bei Verwendung eines Verbrauchsplans. Bei Verwendung eines App Service-Plans werden Funktionen auf den VMs ausgeführt, die Ihrem App Service-Plan zugeordnet sind. Weitere Informationen finden Sie unter [Vergleich von Hostingplänen für Azure Functions][function-plans].</a>
2. <span id="note2">Höhere SLA mit mindestens zwei Instanzen.</a>
3. <span id="note3">Für Produktionsumgebungen.</a>
4. <span id="note4">Kann nach Abschluss eines Auftrags zentral auf 0 herunterskaliert werden.</a>
5. <span id="note5">Erfordert App Service-Umgebung (App Service Environment, ASE).</a>
6. <span id="note7">Erfordert ASE oder BizTalk-Hybridverbindungen.</a>

## <a name="devops"></a>DevOps

| Kriterien | Virtual Machines | App Service | Service Fabric | Azure-Funktionen | Azure Container Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Lokales Debugging | Agnostisch | IIS Express, weitere<a href="#note1b"><sup>1</sup></a> | Lokaler Knotencluster | Azure Functions-Befehlszeilenschnittstelle | Lokale Containerruntime | Lokale Containerruntime | Nicht unterstützt |
| Programmiermodell | Agnostisch | Webanwendung, WebJobs für Hintergrundtasks | Ausführbare Gastdatei, Dienstmodell, Akteurmodell, Container | Funktionen mit Auslösern | Agnostisch | Agnostisch | Befehlszeilenanwendung |
| Anwendungsupdate | Keine integrierte Unterstützung | Bereitstellungsslots | Rollierendes Upgrade (pro Dienst) | Keine integrierte Unterstützung | Je nach Orchestrator, die meisten unterstützen rollierende Updates | Aktualisieren des Containerimages | Nicht zutreffend |

Notizen

1. <span id="note1b">Optionen umfassen IIS Express für ASP.NET oder node.js (iisnode); PHP-Webserver; Azure Toolkit für IntelliJ, Azure Toolkit für Eclipse. App Service unterstützt auch das Remotedebuggen einer bereitgestellten Web-App.</a>
2. <span id="note2b">Informationen dazu finden Sie unter [Ressourcenanbieter und -typen][resource-manager-supported-services]. 


## <a name="scalability"></a>Skalierbarkeit

| Kriterien | Virtual Machines | App Service | Service Fabric | Azure-Funktionen | Azure Container Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Automatische Skalierung | VM-Skalierungsgruppen | Integrierter Dienst | VM Scale Sets | Integrierter Dienst | Nicht unterstützt | Nicht unterstützt | N/V |
| Load Balancer | Azure Load Balancer | Integriert | Azure Load Balancer | Integriert | Azure Load Balancer |  Keine integrierte Unterstützung | Azure Load Balancer |
| Skalierungslimit | Plattformimage: 1.000 Knoten pro VMSS, benutzerdefiniertes Image: 100 Knoten pro VMSS | 20 Instanzen, 50 mit App Service-Umgebung | 100 Knoten pro VMSS | Unendlich<a href="#note1c"><sup>1</sup></a> | 100 |20 Containergruppen pro Abonnement <a href="#note2c"><sup>2</sup></a> | Standardmäßig maximal 20 Kerne; wenden Sie sich für eine Erhöhung an den Kundendienst |

Notizen

1. <span id="note1c">Bei Verwendung eines Verbrauchsplans. Bei Verwendung eines App Service-Plans gelten die Skalierungslimits von App Service. Weitere Informationen finden Sie unter [Vergleich von Hostingplänen für Azure Functions][function-plans].</a>
2. <span id="note2c">Siehe [Kontingente und Regionsverfügbarkeit für Azure Container Instances](/azure/container-instances/container-instances-quotas).</a>


## <a name="availability"></a>Verfügbarkeit

| Kriterien | Virtual Machines | App Service | Service Fabric | Azure-Funktionen | Azure Container Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SLA | [SLA für virtuelle Computer][sla-vm] | [SLA für App Service][sla-app-service] | [SLA für Service Fabric][sla-sf] | [SLA für Functions][sla-functions] | [SLA für Azure Container Service][sla-acs] | [SLA für Container Instances](https://azure.microsoft.com/support/legal/sla/container-instances/) | [SLA für Azure Batch][sla-batch] |
| Failover über mehrere Regionen | Traffic Manager | Traffic Manager | Traffic Manager, regionsübergreifender Cluster | Nicht unterstützt  | Traffic Manager | Nicht unterstützt | Nicht unterstützt |

## <a name="other"></a>Andere

| Kriterien | Virtual Machines | App Service | Service Fabric | Azure-Funktionen | Azure Container Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | In VM konfiguriert | Unterstützt | Unterstützt  | Unterstützt | In VM konfiguriert | Nicht unterstützt | Unterstützt |
| Kosten | [Windows][cost-windows-vm], [Linux][cost-linux-vm] | [App Service – Preise][cost-app-service] | [Service Fabric – Preise][cost-service-fabric] | [Azure Functions – Preise][cost-functions] | [Azure Container Service – Preise][cost-acs] | [Container Instances – Preise](https://azure.microsoft.com/pricing/details/container-instances/) | [Azure Batch – Preise][cost-batch]
| Geeignete Architekturstile | N-schichtig, Big Compute (HPC) | Web-Warteschlange-Worker | Microservices, ereignisgesteuerte Architektur (Event-Driven Architecture, EDA) | Microservices, EDA | Microservices, EDA | Microservices, Automatisierung von Aufgaben, Batchaufträge  | Big Compute |

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/container-service/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/container-service/
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services