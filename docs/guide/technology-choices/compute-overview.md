---
title: "Übersicht über Azure-Compute-Optionen"
description: "Übersicht über Azure-Compute-Optionen"
author: MikeWasson
ms.openlocfilehash: a23dd49f24bc52db6f357540e3ebccb19e0497ee
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="overview-of-azure-compute-options"></a>Übersicht über Azure-Compute-Optionen

Der Begriff *Compute* bezieht sich auf das Hostingmodell für die Computeressourcen, auf denen Ihre Anwendung ausgeführt wird. 

An einem Ende des Spektrums befindet sich IaaS (**Infrastructure-as-a-Service**). Bei IaaS stellen Sie die benötigten VMs zusammen mit den zugeordneten Netzwerk- und Speicherkomponenten bereit. Anschließend stellen Sie die Software und Anwendungen bereit, die für diese VMs jeweils vorgesehen sind. Dieses Modell weist die größte Ähnlichkeit mit einer herkömmlichen lokalen Umgebung auf – mit der Ausnahme, dass die Infrastruktur von Microsoft verwaltet wird. Sie verwalten weiterhin die einzelnen VMs.  

Bei PaaS (**Platform-as-a-Service**) ist eine verwaltete Hostingumgebung vorhanden, in der Sie Ihre Anwendung bereitstellen können, ohne VMs oder Netzwerkressourcen verwalten zu müssen. Anstatt beispielsweise einzelne VMs zu erstellen, geben Sie eine Instanzanzahl an, und der Dienst übernimmt das Bereitstellen, Konfigurieren und Verwalten der erforderlichen Ressourcen. Azure App Service ist ein Beispiel für einen PaaS-Dienst.

Es wird ein Spektrum von IaaS bis zu PaaS in Reinform abgedeckt. Für Azure-VMs kann beispielsweise die automatische Skalierung durchgeführt werden, indem VM-Skalierungsgruppen verwendet werden. Diese Funktion für die automatische Skalierung entspricht nicht genau PaaS, aber es handelt sich um die Art von Verwaltungsfeature, das Teil eines PaaS-Diensts sein kann.

Bei FaaS (**Functions-as-a-Service**) geht dies noch weiter, da es auch nicht mehr erforderlich ist, sich um die Hostingumgebung zu kümmern. Anstatt Computeinstanzen zu erstellen und Code auf diesen Instanzen bereitzustellen, stellen Sie einfach Ihren Code bereit, der vom Dienst dann automatisch ausgeführt wird. Es ist nicht erforderlich, dass Sie die Computeressourcen verwalten. Für diese Dienste wird die serverlose Architektur genutzt, und das nahtlose zentrale Hoch- und Herunterskalieren auf die jeweilige Ebene, die zum Verarbeiten des Datenverkehrs erforderlich ist, kann durchgeführt werden. Bei Azure Functions handelt es sich um einen Dienst vom Typ „FaaS“.

IaaS bietet das höchste Maß an Steuerung, Flexibilität und Portabilität. FaaS bietet Einfachheit, elastische Skalierung und potenzielle Kosteneinsparungen, da Sie nur für die Zeiten zahlen, in denen Ihr Code ausgeführt wird. PaaS liegt zwischen IaaS und FaaS. Im Allgemeinen gilt Folgendes: Je mehr Flexibilität ein Dienst ermöglicht, desto größer ist Ihre Verantwortung für die Konfiguration und Verwaltung der Ressourcen. Mit FaaS-Diensten werden automatisch fast alle Aspekte der Anwendungsausführung verwaltet, während es bei IaaS-Lösungen erforderlich ist, dass Sie die von Ihnen erstellten VMs und Netzwerkkomponenten bereitstellen, konfigurieren und verwalten.

Hier sind die wichtigsten Compute-Optionen aufgeführt, die in Azure derzeit verfügbar sind:

- [Virtual Machines](/azure/virtual-machines/) ist ein IaaS-Dienst, mit dem Sie VMs in einem virtuellen Netzwerk (VNet) bereitstellen und verwalten können.
- [App Service](/azure/app-service/app-service-value-prop-what-is) ist ein verwalteter Dienst zum Hosten von Web-Apps, mobilen App-Back-Ends, RESTful-APIs oder automatisierten Geschäftsprozessen.
- [Service Fabric](/azure/service-fabric/service-fabric-overview) ist eine Plattform für verteilte Systeme, die in vielen Umgebungen ausgeführt werden kann, z.B. Azure oder lokal. Service Fabric ist ein Orchestrator von Microservices in einem Cluster mit Computern. 
- Mit [Azure Container Service](/azure/container-service/container-service-intro) können Sie einen Cluster mit VMs, die für die Ausführung von Anwendungen in Containern vorkonfiguriert sind, erstellen, konfigurieren und verwalten.
- [Azure Functions](/azure/azure-functions/functions-overview) ist ein verwalteter FaaS-Dienst.
- [Azure Batch](/azure/batch/batch-technical-overview) ist ein verwalteter Dienst zum Ausführen von umfassenden parallelen HPC-Anwendungen (High-Performance Computing).
- Bei [Cloud Services](/azure/cloud-services/cloud-services-choose-me) handelt es sich um einen verwalteten Dienst zum Ausführen von Cloudanwendungen. Hierfür wird ein PaaS-Hostingmodell verwendet. 

Berücksichtigen Sie bei der Auswahl einer Compute-Option die folgenden Faktoren:

- Hostingmodell: Wie wird der Dienst gehostet? Welche Anforderungen und Einschränkungen gelten für diese Hostingumgebung? 
- DevOps: Ist eine integrierte Unterstützung für Anwendungsupgrades vorhanden? Welches Bereitstellungsmodell wird verwendet?
- Skalierbarkeit: Wie wird für den Dienst das Hinzufügen oder Entfernen von Instanzen durchgeführt? Kann die automatische Skalierung basierend auf der Last und anderen Metriken erfolgen? 
- Verfügbarkeit: Was umfasst die Vereinbarung zum Servicelevel (SLA) des Diensts? 
- Kosten: Zusätzlich zu den Kosten des eigentlichen Diensts sollten Sie die Betriebskosten für die Verwaltung einer Lösung berücksichtigen, die für den Dienst erstellt wird. Für IaaS-Lösungen können die Betriebskosten unter Umständen höher sein.
- Welche allgemeinen Beschränkungen gelten für die einzelnen Dienste? 
- Welche Art von Anwendungsarchitekturen sind für diesen Dienst geeignet? 

Ein ausführlicherer Vergleich von Compute-Optionen in Azure finden Sie unter [Criteria for choosing an Azure compute option](./compute-comparison.md) (Kriterien für die Wahl einer Azure-Compute-Option).