---
title: "Azure für AWS-Spezialisten"
description: Sie lernen die Grundlagen der Konten, Plattform und Dienste von Microsoft Azure kennen. Zudem lernen Sie die wichtigsten Gemeinsamkeiten und Unterschiede zwischen den Plattformen AWS und Azure kennen. Sie wenden Ihre Erfahrungen zu AWS in Azure an.
keywords: AWS experts, Azure comparison, AWS comparison, difference between azure and aws, azure and aws
author: lbrader
ms.date: 03/24/2017
pnp.series.title: Azure for AWS Professionals
ms.openlocfilehash: b576b11bc152ef721f56e79609cb7a03f2d31dd3
ms.sourcegitcommit: 1c0465cea4ceb9ba9bb5e8f1a8a04d3ba2fa5acd
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/02/2018
---
# <a name="azure-for-aws-professionals"></a>Azure für AWS-Spezialisten

Dieser Artikel ist für AWS-Experten (Amazon Web Services) bestimmt und soll ihnen die Grundlagen zu Konten, Plattform und Diensten von Microsoft Azure vermitteln. Zudem werden die wichtigsten Gemeinsamkeiten und Unterschiede zwischen der AWS- und Azure-Plattform erläutert.

Sie lernen Folgendes:

* Organisation der Konten und Ressourcen in Azure
* Struktur der verfügbaren Lösungen in Azure
* Unterschiede zwischen den wichtigsten Azure- und AWS-Diensten

Die Funktionalität von Azure und AWS hat sich im Laufe der Zeit unabhängig voneinander weiterentwickelt, sodass bedeutende Unterschiede zwischen diesen hinsichtlich Implementierung und Entwurf bestehen.

## <a name="overview"></a>Übersicht

Wie AWS basiert auch Microsoft Azure auf einer zentralen Gruppe von Compute-, Speicher-, Datenbank- und Netzwerkdiensten. In der Vielzahl der Fälle herrscht bei beiden Plattformen eine grundsätzliche Äquivalenz zwischen den angebotenen Produkten und Diensten. Sowohl mit AWS als auch Azure können hochverfügbare Lösungen basierend auf Windows- oder Linux-Hosts erstellt werden. Wenn Sie also mit der Entwicklung mit Linux- und OSS-Technologien vertraut sind, so sind beide Plattformen geeignet.

Die Funktionalität beider Plattformen weist zwar Ähnlichkeiten auf, allerdings sind die Ressourcen, die diese Funktionalität bereitstellen, häufig unterschiedlich organisiert. Zwischen den Diensten, die für die Erstellung einer Lösung erforderlich sind, herrscht nicht immer eine 100%ige Übereinstimmung. Zudem kann es vorkommen, dass ein bestimmter Dienst nur auf einer Plattform angeboten wird, jedoch nicht auf der anderen. Weitere Informationen finden Sie in den [Tabellen zum Vergleich zwischen Azure- und AWS-Diensten](services.md).

## <a name="accounts-and-subscriptions"></a>Konten und Abonnements

Azure-Dienste können abhängig von der Größe und den Anforderungen Ihres Unternehmens mit verschiedenen Preisoptionen erworben werden. Weitere Informationen finden Sie auf der Seite [Azure-Preise](https://azure.microsoft.com/pricing/).

[Azure-Abonnements](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-infrastructure-subscription-accounts-guidelines/) stellen eine Gruppierung von Ressourcen mit einem zugewiesenen Besitzer dar, der für die Verwaltung der Abrechnung und Berechtigungen zuständig ist. Im Gegensatz zu AWS, bei dem alle unter dem jeweiligen AWS-Konto erstellten Ressourcen mit diesem Konto verknüpft sind, sind Abonnements nicht von den jeweiligen Besitzerkonten abhängig und können bei Bedarf neuen Besitzern zugewiesen werden.

![Vergleich der Struktur und des Besitzes von AWS-Konten und Azure-Abonnements](./images/azure-aws-account-compare.png "Vergleich der Struktur und des Besitzes von AWS-Konten und Azure-Abonnements")
<br/>*Vergleich der Struktur und des Besitzes von AWS-Konten und Azure-Abonnements*
<br/><br/>

Abonnements sind drei Arten von Administratorkonten zugewiesen:

-   **Kontoadministrator** – Der Abonnementbesitzer und das Konto, für das die im Abonnement verwendeten Ressourcen in Rechnung gestellt werden. Eine Änderung des Kontoadministrators ist nur durch Übertragung des Abonnementbesitzes möglich.

-   **Dienstadministrator** – Dieses Konto ist mit Rechten zum Erstellen und Verwalten von Ressourcen im Abonnement ausgestattet, jedoch nicht zum Verwalten der Abrechnung. Standardmäßig sind der Konto- und Dienstadministrator demselben Konto zugeordnet. Der Kontoadministrator kann dem Dienstadministratorkonto einen separaten Benutzer für die Verwaltung technischer und betrieblicher Aufgaben eines Abonnements zuweisen. Pro Abonnement gibt es nur einen Dienstadministrator.

-   **Co-Administrator** – Einem Abonnement können mehrere Co-Administratorkonten zugewiesen sein. Co-Administratoren besitzen die vollständige Kontrolle über Abonnementsressourcen und Benutzer, können jedoch nicht den Dienstadministrator ändern.

Unterhalb der Abonnementebene können bestimmten Ressourcen auch Benutzerrollen und individuelle Berechtigungen zugewiesen werden, ähnlich wie bei der Vergabe von Berechtigungen an IAM-Benutzer und -Gruppen in AWS. In Azure sind alle Benutzerkonten entweder mit einem Microsoft-Konto oder einem Geschäfts-, Schul- oder Uni-Konto (einem mit Azure Active Directory verwalteten Konto) verknüpft.

Wie bei AWS-Konten gelten für Abonnements standardmäßige Dienstkontingente und -einschränkungen. Eine vollständige Liste zu diesen Einschränkungen finden Sie unter [Einschränkungen für Azure-Abonnements und Dienste, Kontingente und Einschränkungen](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/).
Diese Einschränkungen können bis zum Maximalwert erhöht werden, wenn Sie eine entsprechende [Supportanfrage im Verwaltungsportal stellen](https://blogs.msdn.microsoft.com/girishp/2015/09/20/increasing-core-quota-limits-in-azure/).

### <a name="see-also"></a>Weitere Informationen

-   [Hinzufügen oder Ändern von Azure-Administratorrollen](https://azure.microsoft.com/documentation/articles/billing-add-change-azure-subscription-administrator/)

-   [Herunterladen von Azure-Rechnungen und täglichen Nutzungsdaten](https://azure.microsoft.com/documentation/articles/billing-download-azure-invoice-daily-usage-date/)

## <a name="resource-management"></a>Ressourcenverwaltung

Der Begriff „Ressource“ wird in Azure und AWS deckungsgleich verwendet und bezeichnet Serverinstanzen, Speicherobjekte, Netzwerkgeräte oder sonstige Entitäten, die Sie innerhalb der Plattform erstellen oder konfigurieren können.

Azure-Ressourcen werden über eins von zwei Modellen bereitgestellt und verwaltet: [Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview) oder das ältere [klassische Azure-Bereitstellungsmodell](/azure/azure-resource-manager/resource-manager-deployment-model).
Neue Ressourcen werden mithilfe des Resource Manager-Modells erstellt.

### <a name="resource-groups"></a>Ressourcengruppen

Sowohl Azure als auch AWS verfügen über Entitäten zur Organisation von Ressourcen, die als „Ressourcengruppen“ bezeichnet werden (z.B. VMs, Speicher, virtuelle Netzwerkgeräte). [Azure-Ressourcengruppen](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/) lassen sich jedoch nicht direkt mit AWS-Ressourcengruppen vergleichen.

Während eine Ressource bei AWS mehreren Ressourcengruppen zugeteilt werden kann, ist eine Azure-Ressource immer nur mit einer Ressourcengruppe verknüpft. Eine Ressource, die in einer Ressourcengruppe erstellt wird, kann in eine andere Gruppe verschoben werden, sich jedoch nur in jeweils einer Ressourcengruppe befinden. Ressourcengruppen zählen zu den grundlegenden Gruppierungsmethoden in Azure Resource Manager.

Ressourcen können auch mithilfe von [Tags](https://azure.microsoft.com/documentation/articles/resource-group-using-tags/) organisiert werden.
Bei Tags handelt es sich um Schlüssel-Wert-Paare, mit denen Sie Ressourcen unabhängig von ihrer Zugehörigkeit zu einer Ressourcengruppe in Ihrem Abonnement gruppieren können.

### <a name="management-interfaces"></a>Verwaltungsoberflächen

Azure bietet verschiedene Möglichkeiten zur Verwaltung Ihrer Ressourcen:

-   [Weboberfläche](https://azure.microsoft.com/documentation/articles/resource-group-portal/):
    Wie das AWS-Dashboard bietet auch das Azure-Portal eine vollständige webbasierte Verwaltungsoberfläche für Azure-Ressourcen.

-   [REST-API](https://azure.microsoft.com/documentation/articles/resource-manager-rest-api/):
    Die Azure Resource Manager-REST-API ermöglicht einen programmgesteuerten Zugriff auf die meisten im Azure-Portal verfügbaren Funktionen.

-   [Befehlszeile](https://azure.microsoft.com/documentation/articles/xplat-cli-azure-resource-manager/):
    Das Tool Azure CLI 2.0 verfügt über eine Befehlszeilenschnittstelle, mit der Azure-Ressourcen erstellt und verwaltet werden können. Die Azure CLI ist für [Windows, Linux und Mac OS](https://aka.ms/azurecli2) verfügbar.

-   [PowerShell](https://azure.microsoft.com/documentation/articles/powershell-azure-resource-manager/):
    Mit den Azure-Modulen für PowerShell können Sie anhand eines Skripts automatisierte Verwaltungsaufgaben ausführen. PowerShell ist für [Windows, Linux und Mac OS](https://github.com/PowerShell/PowerShell) verfügbar.

-   [Vorlagen](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/):
    Azure Resource Manager-Vorlagen bieten ähnliche auf JSON-Vorlagen basierende Ressourcenverwaltungsfunktionen wie der AWS CloudFormation-Dienst.

Bei beiden Schnittstellen spielen Ressourcengruppen eine zentrale Rolle für die Art und Weise, wie Azure-Ressourcen erstellt, bereitgestellt oder geändert werden. Dies lässt sich mit der Rolle eines „Stapels“ vergleichen, die dieser bei der Gruppierung von AWS-Ressourcen bei CloudFormation-Bereitstellungen spielt.

Die Syntax und Struktur dieser Schnittstellen unterscheiden sich von ihren Pendants in AWS, bieten jedoch eine vergleichbare Funktionalität. Darüber hinaus sind viele Verwaltungstools von Drittanbietern, die in AWS verwendet werden (z.B. [Terraform von Hashicorp](https://www.terraform.io/docs/providers/azurerm/) und [Netflix Spinnaker](http://www.spinnaker.io/)), auch in Azure verfügbar.

### <a name="see-also"></a>Weitere Informationen

-   [Richtlinien für Azure-Ressourcengruppen](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)

## <a name="regions-and-zones-high-availability"></a>Regionen und Zonen (Hochverfügbarkeit)

Bei AWS wird Verfügbarkeit hauptsächlich über Verfügbarkeitszonen bereitgestellt. Bei Azure kommen bei der Erstellung von hochverfügbaren Lösungen Fehlerdomänen und Verfügbarkeitsgruppen zum Einsatz. Regionspaare bieten zusätzliche Notfallwiederherstellungsfunktionen.

### <a name="availability-zones-azure-fault-domains-and-availability-sets"></a>Verfügbarkeitszonen, Azure-Fehlerdomänen und Verfügbarkeitsgruppen

Bei AWS ist eine Region in mindestens zwei Verfügbarkeitszonen unterteilt. Eine Verfügbarkeitszone entspricht einem physisch isolierten Rechenzentrum in einer geografischen Region.
Wenn Sie Ihre Anwendungsserver in separaten Verfügbarkeitszonen bereitstellen, hat ein Hardware- oder Konnektivitätsausfall in einer Zone keine Auswirkungen auf Server, die in anderen Zonen gehostet werden.

In Azure definiert eine [Fehlerdomäne](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/) eine Gruppe von VMs, die gemeinsam eine physische Stromversorgungsquelle und einen Netzwerkswitch verwenden.
Durch [Verfügbarkeitsgruppen](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-manage-availability/) können Sie VMs auf mehrere Fehlerdomänen aufteilen. Wenn Instanzen derselben Verfügbarkeitsgruppe zugewiesen sind, teilt Azure diese gleichmäßig auf mehrere Fehlerdomänen auf. Wenn in einer Fehlerdomäne ein Strom- oder ein Netzwerkausfall auftritt, ist zumindest ein Teil der VMs in einer Gruppe nicht vom Ausfall betroffen, da sich dieser in einer anderen Fehlerdomäne befindet.

![Vergleich von AWS-Verfügbarkeitszonen mit Azure-Fehlerdomänen und -Verfügbarkeitsgruppen](./images/zone-fault-domains.png "Vergleich von AWS-Verfügbarkeitszonen mit Azure-Fehlerdomänen und -Verfügbarkeitsgruppen")
<br/>*Vergleich von AWS-Verfügbarkeitszonen mit Azure-Fehlerdomänen und -Verfügbarkeitsgruppen*
<br/><br/>

Verfügbarkeitsgruppen sollten nach der Rolle der Instanz in Ihrer Anwendung organisiert werden, um sicherzustellen, dass eine Instanz in jeder Rolle betriebsbereit ist. In einer Standardwebanwendung mit drei Ebenen empfiehlt es sich beispielsweise, eine separate Verfügbarkeitsgruppe für Front-End-, Anwendungs- und Dateninstanzen zu erstellen.

![Azure-Verfügbarkeitsgruppen für die einzelnen Anwendungsrollen](./images/three-tier-example.png "Verfügbarkeitsgruppen für die einzelnen Anwendungsrollen")
<br/>*Azure-Verfügbarkeitsgruppen für die einzelnen Anwendungsrollen*
<br/><br/>

Wenn VM-Instanzen zu Verfügbarkeitsgruppen hinzugefügt werden, wird ihnen auch eine [Updatedomäne](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/) zugewiesen.
Eine Updatedomäne ist eine Gruppe von VMs, die gleichzeitig für geplante Wartungsereignisse festgelegt sind. Durch die Verteilung von VMs auf mehrere Updatedomänen wird sichergestellt, dass geplante Update- und Patchingereignisse jeweils nur eine Teilmenge dieser VMs betreffen.

### <a name="paired-regions"></a>Regionspaare

In Azure wird durch den Einsatz von [Regionspaaren](https://azure.microsoft.com/documentation/articles/best-practices-availability-paired-regions/) Unterstützung für Redundanz zwischen zwei vordefinierten geographischen Regionen geboten. So wird sichergestellt, dass Ihre Lösung auch dann verfügbar ist, wenn ein Ausfall eine ganze Azure-Region betrifft.

Im Gegensatz zu AWS-Verfügbarkeitszonen, bei denen es sich um physisch getrennte Rechenzentren handelt, die sich jedoch in relativ nahe gelegenen geografischen Gebieten befinden können, liegen Regionspaare in der Regel mindestens 480 km voneinander entfernt. Denn damit soll sichergestellt werden, dass sich größere Katastrophen nur auf eine der Regionen eines Paars auswirken. Bei benachbarten Paaren können Datenbank- und Speicherdienstdaten synchronisiert werden, und sie sind so konfiguriert, dass Plattformupdates jeweils nur in einer Region eines Paares ausgeführt werden.

[Georedundanter Azure-Speicher](https://azure.microsoft.com/documentation/articles/storage-redundancy/#geo-redundant-storage) wird automatisch im entsprechenden Regionspaar gesichert. Bei allen anderen Ressourcen heißt die Erstellung einer vollständig redundanten Lösung mit Regionspaaren, dass in beiden Regionen eine vollständige Kopie Ihrer Lösung erstellt wird.

### <a name="see-also"></a>Weitere Informationen

-   [Regionen und Verfügbarkeit für virtuelle Computer in Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-regions-and-availability/)

-   [Hochverfügbarkeit für Azure-Anwendungen](../resiliency/high-availability-azure-applications.md)

-   [Notfallwiederherstellung für Azure-Anwendungen](../resiliency/disaster-recovery-azure-applications.md)

-   [Geplante Wartung für virtuelle Linux-Computer in Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-planned-maintenance/)

## <a name="services"></a>Dienste

Eine vollständige Auflistung bezüglich der Zuordnung sämtlicher Dienste zwischen den Plattformen finden sie in der [umfassenden Vergleichsmatrix zu AWS- und Azure-Diensten](https://aka.ms/azure4aws-services).

Es sind nicht alle Azure-Produkte und -Dienste in allen Regionen verfügbar. Weitere Informationen finden Sie auf der Seite [Produkte nach Region](https://azure.microsoft.com/regions/services/). Auf der Seite [Vereinbarungen zum Servicelevel (SLAs)](https://azure.microsoft.com/support/legal/sla/) finden Sie die Richtlinien zu Garantien für Betriebszeiten und Downtimegutschriften für die einzelnen Azure-Produkte oder -Dienste.

Die folgenden Abschnitte enthalten einen kurzen Überblick über die Unterschiede häufig verwendeter Funktionen und Dienste zwischen der AWS- und Azure-Plattform.

### <a name="compute-services"></a>Compute Services

#### <a name="ec2-instances-and-azure-virtual-machines"></a>EC2-Instanzen und virtuelle Azure-Computer

Obwohl die Größen von AWS-Instanztypen und virtuellen Azure-Computern eine ähnliche Aufteilung aufweisen, bestehen Unterschiede in den RAM-, CPU- und Speicherkapazitäten.

-   [Amazon EC2-Instanztypen](https://aws.amazon.com/ec2/instance-types/)

-   [Größen für virtuelle Computer in Azure (Windows)](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/)

-   [Größen für virtuelle Computer in Azure (Linux)](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-sizes/)

Im Gegensatz zur sekundengenauen Abrechnung in AWS werden On-Demand-VMs von Azure minutengenau abgerechnet.

In Azure sind keine Pendants zu EC2 Spot Instances oder Dedicated Hosts vorhanden.

#### <a name="ebs-and-azure-storage-for-vm-disks"></a>EBS und Azure Storage für VM-Datenträger

Durch [Datenträger](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-about-disks-vhds/) in Blob Storage wird eine dauerhafte Datenspeicherung für Azure-VMs sichergestellt. Dies ist vergleichbar mit der Art und Weise, wie Speicherdatenträgervolumes von EC2-Instanzen in Elastic Block Store (EBS) gespeichert werden. Der [temporäre Azure-Speicher](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/) stellt außerdem denselben temporären Lese-/Schreibspeicher mit niedriger Latenz für VMs bereit wie EC2-Instanzspeicher (auch als kurzlebige Speicher bezeichnet).

Mit [Azure Premium Storage](https://docs.microsoft.com/azure/storage/storage-premium-storage) werden Datenträger-E/As für höhere Leistung unterstützt.
Vergleichen lässt sich dies mit den Speicheroptionen mit bereitgestellten IOPS von AWS.

#### <a name="lambda-azure-functions-azure-web-jobs-and-azure-logic-apps"></a>Lambda, Azure Functions, Azure WebJobs und Azure Logic Apps

[Azure Functions](https://azure.microsoft.com/services/functions/) ist das primäre Pendant zu AWS Lambda hinsichtlich der Bereitstellung von serverlosem On-Demand-Code.
Die Lambda-Funktionalität weist jedoch auch Überschneidungen mit anderen Azure-Diensten auf:

-   [WebJobs](https://azure.microsoft.com/documentation/articles/web-sites-create-web-jobs/): Ermöglicht die Erstellung von geplanten oder fortlaufend ausgeführten Hintergrundaufgaben

-   [Logic Apps](https://azure.microsoft.com/services/logic-apps/): Stellt Verwaltungsdienste für die Kommunikation, Integration und für Unternehmensregeln bereit

#### <a name="autoscaling-azure-vm-scaling-and-azure-app-service-autoscale"></a>Automatische Skalierung mit Azure-VM-Skalierungsgruppen und Azure App Service

Die automatische Skalierung in Azure wird von zwei Diensten durchgeführt:

-   [VM-Skalierungsgruppen](https://azure.microsoft.com/documentation/articles/virtual-machine-scale-sets-overview/): Ermöglichen die Bereitstellung und Verwaltung einer identischen VM-Gruppe. Die Anzahl der Instanzen kann je nach Leistungsanforderungen automatisch skaliert werden.

-   [Automatische Skalierung mit App Service](https://azure.microsoft.com/documentation/articles/web-sites-scale/): Bietet die Möglichkeit zur automatischen Skalierung von Lösungen mit Azure App Service


#### <a name="container-service"></a>Container Service
[Azure Container Service](https://docs.microsoft.com/azure/container-service/container-service-intro) unterstützt Docker-Container, die über Docker Swarm, Kubernetes oder DC/OS verwaltet werden.

#### <a name="other-compute-services"></a>Sonstige Computedienste 


Azure bietet mehrere Computedienste, für die es keine direkte Übereinstimmung in AWS gibt:

-   [Azure Batch](https://azure.microsoft.com/documentation/articles/batch-technical-overview/): Ermöglicht die Verwaltung von rechenintensiven Aufgaben für eine skalierbare Sammlung von virtuellen Computern

-   [Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-overview/): Plattform für die Entwicklung und das Hosting skalierbarer [Microservicelösungen](https://azure.microsoft.com/documentation/articles/service-fabric-overview-microservices/)

#### <a name="see-also"></a>Weitere Informationen

-   [Erstellen eines virtuellen Linux-Computers in Azure mithilfe des Portals](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-quick-create-portal/)

-   [Azure-Referenzarchitektur: Ausführen einer Linux-VM in Azure](https://azure.microsoft.com/documentation/articles/guidance-compute-single-vm-linux/)

-   [Erste Schritte mit Node.js-Web-Apps in Azure App Service](https://azure.microsoft.com/documentation/articles/app-service-web-nodejs-get-started/)

-   [Azure-Referenzarchitektur: Grundlegende Webanwendung](https://azure.microsoft.com/documentation/articles/guidance-web-apps-basic/)

-   [Erstellen Sie Ihre erste Funktion in Azure Functions](https://azure.microsoft.com/documentation/articles/functions-create-first-azure-function/)

### <a name="storage"></a>Speicher

#### <a name="s3ebsefs-and-azure-storage"></a>S3/EBS/EFS und Azure Storage

In der AWS-Plattform ist der Cloudspeicher primär in drei Dienste unterteilt:

-   **Simple Storage Service (S3)**: Grundlegender Objektspeicher. Stellt Daten über eine per Internet zugängliche API zur Verfügung.

-   **Elastic Block Storage (EBS)**: Speicher auf Blockebene für den Zugriff durch eine einzelne VM

-   **Elastic File System (EFS)**: Dateispeicher, der für den Einsatz als freigegebener Speicher für Tausende von EC2-Instanzen vorgesehen ist.

In Azure Storage können mit an Abonnements gebundene [Speicherkonten](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) die folgenden Speicherdienste erstellt und verwaltet werden:

-   [Blob Storage](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/): Speichert alle Arten von Text- oder Binärdaten, z.B. ein Dokument, eine Mediendatei oder ein Installer einer Anwendung. Sie können Blob Storage für den privaten Zugriff einrichten oder Inhalte öffentlich im Internet zugänglich machen. Blob Storage dient demselben Zweck wie AWS S3 und EBS.
-   [Table Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/): Speichert strukturierte Datensätze. Table Storage ist ein Datenspeicher für NoSQL-Schlüsselattribute, der eine schnelle Entwicklung und einen schnellen Zugriff auf große Datenmengen ermöglicht. Dies lässt sich mit dem SimpleDB- and DynamoDB-Dienst von AWS vergleichen.

-   [Queue Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/): Bietet Messaging für die Workflowverarbeitung und die Kommunikation zwischen Komponenten von Clouddiensten.

-   [File Storage](https://azure.microsoft.com/documentation/articles/storage-java-how-to-use-file-storage/): Bietet einen gemeinsam genutzten Speicher für Legacyanwendungen mit dem SMB-Standardprotokoll (Server Message Block). File Storage wird ähnlich wie EFS in der AWS-Plattform verwendet.
 
#### <a name="glacier-and-azure-storage"></a>Glacier und Azure Storage 

[Azure Archive Blob Storage](/azure/storage/blobs/storage-blob-storage-tiers#archive-access-tier) ist vergleichbar mit dem Glacier-Speicherdienst von AWS. Es ist für selten verwendete Daten konzipiert, die für mindestens 180 Tage gespeichert werden und bei denen eine Abrufwartezeit von mehreren Stunden akzeptabel ist. 

Für Daten, die zwar selten verwendet werden, beim Zugriff aber umgehend verfügbar sein müssen, stellt [Azure Cool Blob Storage](/azure/storage/blobs/storage-blob-storage-tiers#cool-access-tier) im Vergleich zum Standard-Blobspeicher eine kostengünstigere Speicheroption dar. Diese Speicherebene ist vergleichbar mit AWS S3 (Speicherdienst für selten verwendete Daten).

#### <a name="see-also"></a>Weitere Informationen

-   [Checkliste zu Leistung und Skalierbarkeit von Microsoft Azure Storage](https://azure.microsoft.com/documentation/articles/storage-performance-checklist/)

-   [Azure Storage-Sicherheitsleitfaden](https://azure.microsoft.com/documentation/articles/storage-security-guide/)

-   [Muster und Übungen: Anleitungen zum Content Delivery Network (CDN)](https://azure.microsoft.com/documentation/articles/best-practices-cdn/)

### <a name="networking"></a>Netzwerk

#### <a name="elastic-load-balancing-azure-load-balancer-and-azure-application-gateway"></a>Elastischer Lastenausgleich, Azure Load Balancer und Azure Application Gateway

Die Pendants zu den beiden Diensten für den elastischen Lastenausgleich in Azure zählen Folgende:

-   [Load Balancer](https://azure.microsoft.com/documentation/articles/load-balancer-overview/): Bietet dieselbe Funktionalität wie AWS Classic Load Balancer, mit dem Sie den Datenverkehr für mehrere VMs auf Netzwerkebene verteilen können. Zudem werden Failoverfunktionen bereitgestellt.

-   [Application Gateway](https://azure.microsoft.com/documentation/articles/application-gateway-introduction/): Bietet regelbasiertes Routing auf Anwendungsebene, vergleichbar mit AWS Application Load Balancer.

#### <a name="route-53-azure-dns-and-azure-traffic-manager"></a>Route 53, Azure DNS und Azure Traffic Manager

In AWS bietet Route 53 Funktionen zur Verwaltung von DNS-Namen sowie Datenverkehrrouting und Failoverdienste auf DNS-Ebene. In Azure wird dies über zwei Dienste abgewickelt:

-   [Azure DNS](https://azure.microsoft.com/documentation/services/dns/): Bietet Domänen- und DNS-Verwaltung

-   [Traffic Manager](https://azure.microsoft.com/documentation/articles/traffic-manager-overview/): Bietet Datenverkehrrouting, Lastenausgleich und Failoverfunktionen auf DNS-Ebene

#### <a name="direct-connect-and-azure-expressroute"></a>Direct Connect und Azure ExpressRoute

Azure bietet über seinen [ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/)-Dienst ähnliche dedizierte Site-to-Site-Verbindungen an. Mit ExpressRoute kann über eine dedizierte VPN-Verbindung eine direkte Verbindung zwischen Ihrem lokalen Netzwerk und Azure-Ressourcen hergestellt werden. Azure bietet auch konventionellere [Site-to-Site-VPN-Verbindungen](https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/) zu geringeren Kosten.

#### <a name="see-also"></a>Weitere Informationen

-   [Erstellen eines virtuellen Netzwerks im Azure-Portal](https://azure.microsoft.com/documentation/articles/virtual-networks-create-vnet-arm-pportal/)

-   [Planen und Entwerfen von Azure Virtual Networks](https://azure.microsoft.com/documentation/articles/virtual-network-vnet-plan-design-arm/)

-   [Bewährte Methoden für die Azure-Netzwerksicherheit](https://azure.microsoft.com/documentation/articles/azure-security-network-security-best-practices/)

### <a name="database-services"></a>Datenbankdienste

#### <a name="rds-and-azure-relational-database-services"></a>RDS und Azure-Dienste für relationale Datenbanken

In Azure stehen verschiedene Dienste für relationale Datenbanken zur Verfügung, die dem Dienst für relationale Datenbanken (Relational Database Service, RDS) von AWS entsprechen.

-   [SQL-Datenbank](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)
-   [Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/overview)
-   [Azure-Datenbank für PostgreSQL](https://docs.microsoft.com/azure/postgresql/overview)

Andere Datenbankmodule wie [SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server/), [Oracle](https://azure.microsoft.com/campaigns/oracle/) und [MySQL](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-classic-mysql-2008r2/) können über Azure-VM-Instanzen bereitgestellt werden.

Die Kosten für AWS RDS werden durch die Menge der Hardwareressourcen bestimmt, die Ihre Instanz verbraucht, z.B. CPU-, RAM- und Speicherkapazitäten sowie Netzwerkbandbreite. Bei den Azure-Datenbankdiensten hängen die Kosten von der Datenbankgröße, der Anzahl der gleichzeitigen Verbindungen und den Durchsatzstufen ab.

#### <a name="see-also"></a>Weitere Informationen

-   [Tutorials zu Azure-SQL-Datenbanken](https://azure.microsoft.com/documentation/articles/sql-database-explore-tutorials/)

-   [Konfigurieren der Georeplikation für Azure SQL-Datenbank mit dem Azure-Portal](https://azure.microsoft.com/documentation/articles/sql-database-geo-replication-portal/)

-   [Einführung in Azure Cosmos DB: DocumentDB-API](https://azure.microsoft.com/documentation/articles/documentdb-introduction/)

-   [Verwenden des Azure-Tabellenspeichers mit Node.js](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/)

### <a name="security-and-identity"></a>Sicherheit und Identität

#### <a name="directory-service-and-azure-active-directory"></a>Verzeichnisdienst und Azure Active Directory

Azure unterteilt Verzeichnisdienste in folgende Angebote:

-   [Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-whatis/): Cloudbasierter Verzeichnis- und Identitätsverwaltungsdienst

-   [Azure Active Directory B2B](https://azure.microsoft.com/documentation/articles/active-directory-b2b-collaboration-overview/): Ermöglicht Zugriff auf Unternehmensanwendungen mit von Partnern verwalteten Identitäten

-   [Azure Active Directory B2C](https://azure.microsoft.com/documentation/articles/active-directory-b2c-overview/): Dienst, der Unterstützung für einmaliges Anmelden und Benutzerverwaltung für kundenorientierte Anwendungen bietet

-   [Azure Active Directory Domain Services](https://azure.microsoft.com/documentation/articles/active-directory-ds-overview/): Gehosteter Domänencontrollerdienst, der mit Active Directory kompatible Domänenbeitritts- und Benutzerverwaltungsfunktionen ermöglicht

#### <a name="web-application-firewall"></a>Web Application Firewall

Neben der [Application Gateway-WAF](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/) (Web Application Firewall) können Sie auch [Web Application Firewalls](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/) von Drittanbietern wie [Barracuda Networks](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/) verwenden.

#### <a name="see-also"></a>Weitere Informationen

-   [Erste Schritte mit Microsoft Azure-Sicherheit](https://azure.microsoft.com/documentation/articles/azure-security-getting-started/)

-   [Azure-Identitätsverwaltung und Sicherheit der Zugriffssteuerung – Bewährte Methoden](https://azure.microsoft.com/documentation/articles/azure-security-identity-management-best-practices/)

### <a name="application-and-messaging-services"></a>Anwendungs- und Messagingdienste

#### <a name="simple-email-service"></a>Simple Email Service

AWS stellt mit Simple Email Service (SES) Funktionen zum Versand von Benachrichtigungs-, Transaktions- oder Marketing-E-Mails bereit. In Azure bieten Lösungen von Drittanbietern wie [SendGrid](https://sendgrid.com/partners/azure/) E-Mail-Dienste an.

#### <a name="simple-queueing-service"></a>Simple Queueing Service

AWS Simple Queueing Service (SQS) stellt ein Messagingsystem bereit, das innerhalb der AWS-Plattform eine Verbindung mit Anwendungen, Diensten und Geräten herstellt. Azure bietet zwei Dienste mit ähnlicher Funktionalität an:

-   [Queue Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/): Ein Cloudmessagingdienst, der die Kommunikation zwischen Anwendungskomponenten innerhalb der Azure-Plattform ermöglicht

-   [Service Bus](https://azure.microsoft.com/en-us/services/service-bus/): Ein zuverlässigeres Messagingsystem, um für Anwendungen, Diensten und Geräte eine Verbindung herzustellen. Mit dem entsprechenden [Service Bus Relay](https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-what-is-it) kann Service Bus auch eine Verbindung mit remote gehosteten Anwendungen und Diensten herstellen.

#### <a name="device-farm"></a>Device Farm

AWS Device Farm bietet geräteübergreifende Testdienste. In Azure bietet [Xamarin Test Cloud](https://www.xamarin.com/test-cloud) ähnliche geräteübergreifende Front-End-Tests für Mobilgeräte.

Zusätzlich zu den Front-End-Tests bietet [Azure DevTest Labs](https://azure.microsoft.com/services/devtest-lab/) Back-End-Testressourcen für Linux- und Windows-Umgebungen.

#### <a name="see-also"></a>Weitere Informationen

-   [Gewusst wie: Verwenden von Queue Storage mit Node.js](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/)

-   [Verwenden von Service Bus-Warteschlangen](https://azure.microsoft.com/documentation/articles/service-bus-nodejs-how-to-use-queues/)

### <a name="analytics-and-big-data"></a>Analysen und Big Data

Die [Cortana Intelligence Suite](https://azure.microsoft.com/suites/cortana-intelligence-suite/) ist ein Paket aus Azure-Produkten und -Diensten, das für die Erfassung, Organisation, Analyse und Visualisierung großer Datenmengen bestimmt ist. Die Cortana Suite besteht aus folgenden Diensten:

-   [HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/): Verwaltete Apache-Verteilung, die Hadoop, Spark, Storm oder HBase umfasst

-   [Data Factory](https://azure.microsoft.com/documentation/services/data-factory/): Bietet Datenorchestrierungs- und Datenpipelinefunktionen

-   [SQL Data Warehouse](https://azure.microsoft.com/documentation/services/sql-data-warehouse/): Umfangreicher relationaler Datenspeicher

-   [Data Lake Store](https://azure.microsoft.com/documentation/services/data-lake-store/): Umfangreicher Speicher, der für Big Data-Analyseworkloads optimiert ist

-   [Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/): Wird verwendet, um Predictive Analytics zu erstellen und auf Daten anzuwenden

-   [Stream Analytics](https://azure.microsoft.com/documentation/services/stream-analytics/): Datenanalyse in Echtzeit

-   [Data Lake Analytics](https://azure.microsoft.com/documentation/articles/data-lake-analytics-overview/): Umfassender Analysedienst, der für die Verwendung mit Data Lake Store optimiert ist

-   [Power BI](https://powerbi.microsoft.com/): Wird zur Bereitstellung von Datenvisualisierung verwendet

#### <a name="see-also"></a>Weitere Informationen

-   [Cortana Intelligence Gallery](https://gallery.cortanaintelligence.com/)

-   [Überblick über Big Data-Lösungen von Microsoft](https://msdn.microsoft.com/library/dn749804.aspx)

-   [Blog zu Azure Data Lake und Azure HDInsight](https://blogs.msdn.microsoft.com/azuredatalake/)

### <a name="internet-of-things"></a>Internet der Dinge

#### <a name="see-also"></a>Weitere Informationen

-   [Erste Schritte mit Azure IoT Hub](https://azure.microsoft.com/documentation/articles/iot-hub-csharp-csharp-getstarted/)

-   [Vergleich von IoT Hub und Event Hubs](https://azure.microsoft.com/documentation/articles/iot-hub-compare-event-hubs/)

### <a name="mobile-services"></a>Mobile Services

#### <a name="notifications"></a>Benachrichtigungen

Notification Hubs unterstützt nicht das Senden von SMS oder E-Mail-Nachrichten. Daher sind für diese Zustellarten Dienste von Drittanbietern erforderlich.

#### <a name="see-also"></a>Weitere Informationen

-   [Erstellen einer Android-App](https://azure.microsoft.com/documentation/articles/app-service-mobile-android-get-started/)

-   [Authentifizierung und Autorisierung in Azure Mobile Apps](https://azure.microsoft.com/documentation/articles/app-service-mobile-auth/)

-   [Senden von Pushbenachrichtigungen mit Azure Notification Hubs](https://azure.microsoft.com/documentation/articles/notification-hubs-android-push-notification-google-fcm-get-started/)

### <a name="management-and-monitoring"></a>Verwaltung und Überwachung

#### <a name="see-also"></a>Weitere Informationen
-   [Anleitungen zu Überwachung und Diagnose](https://azure.microsoft.com/documentation/articles/best-practices-monitoring/)

-   [Bewährte Methoden für das Erstellen von Azure Resource Manager-Vorlagen](https://azure.microsoft.com/documentation/articles/resource-manager-template-best-practices/)

-   [Azure Resource Manager-Schnellstartvorlagen](https://azure.microsoft.com/documentation/templates/)


## <a name="next-steps"></a>Nächste Schritte

-   [Umfassende Vergleichsmatrix zu AWS- und Azure-Diensten](https://aka.ms/azure4aws-services)

-   [Interaktives Azure Platform Big Picture](http://azureplatform.azurewebsites.net/)

-   [Erste Schritte mit Azure](https://azure.microsoft.com/get-started/)

-   [Azure-Lösungsarchitekturen](https://azure.microsoft.com/solutions/architecture/)

-   [Azure-Referenzarchitekturen](https://azure.microsoft.com/documentation/articles/guidance-architecture/)

-   [Muster und Übungen: Anweisungen zu Azure](https://azure.microsoft.com/documentation/articles/guidance/)

-   [Kostenloser Onlinekurs: Microsoft Azure for AWS Experts](http://aka.ms/azureforaws)
