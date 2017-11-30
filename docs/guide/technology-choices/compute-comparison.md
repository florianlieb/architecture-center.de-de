---
title: "Kriterien für die Auswahl einer Azure-Compute-Option"
description: Dieser Artikel vergleicht Azure-Computedienste unter verschiedenen Aspekten miteinander.
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 640793b56c1713f63456bab75ab4b9289d22a53c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="criteria-for-choosing-an-azure-compute-option"></a>Kriterien für die Auswahl einer Azure-Compute-Option

Der Begriff *Compute* bezieht sich auf das Hostingmodell für die Computeressourcen, auf denen Ihre Anwendungen ausgeführt werden. Die folgenden Tabellen vergleichen Azure-Computedienste unter verschiedenen Aspekten miteinander. Ziehen Sie diese Tabellen zu Rate, wenn Sie eine Compute-Option für Ihre Anwendung auswählen.

## <a name="hosting-model"></a>Hostingmodell

| Kriterien | Virtual Machines | App Service | Service Fabric | Azure-Funktionen | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Anwendungskomposition | Agnostisch | Anwendungen | Dienste, ausführbare Gastdateien | Functions | Container | Rollen | Geplante Aufträge  |
| Dichte | Agnostisch | Mehrere Apps pro Instanz über App-Pläne | Mehrere Dienste pro VM | Keine dedizierten Instanzen<a href="#note1"><sup>1</sup></a> | Mehrere Container pro VM | Eine Rolleninstanz pro VM | Mehrere Apps pro VM |
| Mindestanzahl von Knoten | 1<a href="#note2"><sup>2</sup></a>  | 1 | 5<a href="#note3"><sup>3</sup></a> | Keine dedizierten Knoten<a href="#note1"><sup>1</sup></a> | 3 | 2 | 1<a href="#note4"><sup>4</sup></a> |
| Zustandsverwaltung | Zustandslos oder zustandsbehaftet | Zustandslos | Zustandslos oder zustandsbehaftet | Zustandslos | Zustandslos oder zustandsbehaftet | Zustandslos | Zustandslos |
| Webhosting | Agnostisch | Integriert | Selbstgehostet, IIS in Containern | Nicht zutreffend | Agnostisch | Integriert (IIS) | Nein |
| Betriebssystem | Windows, Linux | Windows, Linux (Vorschau)  | Windows, Linux (Vorschau) | Nicht zutreffend | Windows, Linux | Windows | Windows, Linux |
| Bereitstellung in dediziertem VNET möglich? | Unterstützt | Unterstützt<a href="#note5"><sup>5</sup></a> | Unterstützt | Nicht unterstützt | Unterstützt | Unterstützt<a href="#note6"><sup>6</sup></a> | Unterstützt |
| Hybridkonnektivität | Unterstützt | Unterstützt<a href="#note1"><sup>7</sup></a>  | Unterstützt | Nicht unterstützt | Unterstützt | Unterstützt<a href="#note8"><sup>8</sup></a> | Unterstützt |

Hinweise

1. <span id="note1">Bei Verwendung eines Verbrauchsplans. Bei Verwendung eines App Service-Plans werden Funktionen auf den VMs ausgeführt, die Ihrem App Service-Plan zugeordnet sind. Weitere Informationen finden Sie unter [Vergleich von Hostingplänen für Azure Functions][function-plans].</a>
2. <span id="note2">Höhere SLA mit mindestens zwei Instanzen.</a>
3. <span id="note3">Für Produktionsumgebungen.</a>
4. <span id="note4">Kann nach Abschluss eines Auftrags zentral auf 0 herunterskaliert werden.</a>
5. <span id="note5">Erfordert App Service-Umgebung (App Service Environment, ASE).</a>
6. <span id="note6">nur klassisches VNET.</a>
7. <span id="note7">Erfordert ASE oder BizTalk-Hybridverbindungen.</a>
8. <span id="note8">Klassisches VNET oder Resource Manager-VNET über VNET-Peering.</a>

## <a name="devops"></a>DevOps

| Kriterien | Virtual Machines | App Service | Service Fabric | Azure-Funktionen | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Lokales Debugging | Agnostisch | IIS Express, weitere<a href="#note1b"><sup>1</sup></a> | Lokaler Knotencluster | Azure Functions-Befehlszeilenschnittstelle | Lokale Containerruntime | Lokaler Emulator | Nicht unterstützt |
| Programmiermodell | Agnostisch | Webanwendung, WebJobs für Hintergrundtasks | Ausführbare Gastdatei, Dienstmodell, Akteurmodell, Container | Funktionen mit Auslösern | Agnostisch | Webrolle, Workerrolle | Befehlszeilenanwendung |
| Ressourcen-Manager | Unterstützt | Unterstützt | Unterstützt | Unterstützt | Unterstützt | Eingeschränkt<a href="#note2b"><sup>2</sup></a> | Unterstützt |  
| Anwendungsupdate | Keine integrierte Unterstützung | Bereitstellungsslots | Rollierendes Upgrade (pro Dienst) | Keine integrierte Unterstützung | Je nach Orchestrator, die meisten unterstützen rollierende Updates | VIP-Austausch oder rollierendes Update | Nicht zutreffend |

Hinweise

1. <span id="note1b">Optionen umfassen IIS Express für ASP.NET oder node.js (iisnode); PHP-Webserver; Azure Toolkit für IntelliJ, Azure Toolkit für Eclipse. App Service unterstützt auch das Remotedebuggen einer bereitgestellten Web-App.</a>
2. <span id="note2b">Informationen dazu finden Sie unter [Ressourcenanbieter und -typen][resource-manager-supported-services]. 


## <a name="scalability"></a>Skalierbarkeit

| Kriterien | Virtual Machines | App Service | Service Fabric | Azure-Funktionen | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Automatische Skalierung | VM-Skalierungsgruppen | Integrierter Dienst | VM-Skalierungsgruppen | Integrierter Dienst | Nicht unterstützt | Integrierter Dienst | N/V |
| Load Balancer | Azure Load Balancer | Integriert | Azure Load Balancer | Integriert | Azure Load Balancer | Integriert | Azure Load Balancer |
| Skalierungslimit | Plattformimage: 1.000 Knoten pro VMSS, benutzerdefiniertes Image: 100 Knoten pro VMSS | 20 Instanzen, 50 mit App Service-Umgebung | 100 Knoten pro VMSS | Unendlich<a href="#note1c"><sup>1</sup></a> | 100 | Kein definiertes Limit, maximal 200 empfohlen | Standardmäßig maximal 20 Kerne; wenden Sie sich für eine Erhöhung an den Kundendienst |

Hinweise

1. <span id="note1c">Bei Verwendung eines Verbrauchsplans. Bei Verwendung eines App Service-Plans gelten die Skalierungslimits von App Service. Weitere Informationen finden Sie unter [Vergleich von Hostingplänen für Azure Functions][function-plans].</a>

## <a name="availability"></a>Availability

| Kriterien | Virtual Machines | App Service | Service Fabric | Azure-Funktionen | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SLA | [SLA für virtuelle Computer][sla-vm] | [SLA für App Service][sla-app-service] | [SLA für Service Fabric][sla-sf] | [SLA für Functions][sla-functions] | [SLA für Azure Container Service][sla-acs] | [SLA für Cloud Services][sla-cloud-service] | [SLA für Azure Batch][sla-batch] |
| Failover über mehrere Regionen | Traffic Manager | Traffic Manager | Traffic Manager, regionsübergreifender Cluster | Nicht unterstützt  | Traffic Manager | Traffic Manager | Nicht unterstützt |

## <a name="security"></a>Sicherheit

| Kriterien | Virtual Machines | App Service | Service Fabric | Azure-Funktionen | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | In VM konfiguriert | Unterstützt | Unterstützt  | Unterstützt | In VM konfiguriert | Unterstützt | Unterstützt |
| RBAC | Unterstützt | Unterstützt | Unterstützt | Unterstützt | Unterstützt | Nicht unterstützt | Unterstützt |

## <a name="other"></a>Sonstige

| Kriterien | Virtual Machines | App Service | Service Fabric | Azure-Funktionen | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Kosten | [Windows][cost-windows-vm], [Linux][cost-linux-vm] | [App Service – Preise][cost-app-service] | [Service Fabric – Preise][cost-service-fabric] | [Azure Functions – Preise][cost-functions] | [Azure Container Service – Preise][cost-acs] | [Cloud Services – Preise][cost-cloud-services] | [Azure Batch – Preise][cost-batch]
| Geeignete Architekturstile | N-schichtig, Big Compute (HPC) | Web-Warteschlange-Worker | Microservices, ereignisgesteuerte Architektur (Event-Driven Architecture, EDA) | Microservices, EDA | Microservices, EDA | Web-Warteschlange-Worker | Big Compute |

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/container-service/
[cost-cloud-services]: https://azure.microsoft.com/pricing/details/cloud-services/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/container-service/
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-cloud-service]: https://azure.microsoft.com/support/legal/sla/cloud-services/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services