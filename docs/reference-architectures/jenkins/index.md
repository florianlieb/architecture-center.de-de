---
title: Ausführen eines Jenkins-Servers in Azure
description: Diese Referenzarchitektur veranschaulicht, wie Sie einen für Unternehmen konzipierten skalierbaren Jenkins-Server in Azure bereitstellen und betreiben, der durch einmaliges Anmelden (Single Sign-On, SSO) geschützt ist.
author: njray
ms.date: 01/21/18
ms.openlocfilehash: 5f9c54e71a8750e88de1ae633ccc1316f8375d3a
ms.sourcegitcommit: 0de300b6570e9990e5c25efc060946cb9d079954
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/03/2018
---
# <a name="run-a-jenkins-server-on-azure"></a>Ausführen eines Jenkins-Servers in Azure

Diese Referenzarchitektur veranschaulicht, wie Sie einen für Unternehmen konzipierten skalierbaren Jenkins-Server in Azure bereitstellen und betreiben, der durch einmaliges Anmelden (Single Sign-On, SSO) geschützt ist. Die Architektur verwendet zudem Azure Monitor zum Überwachen des Zustands des Jenkins-Servers. [**So stellen Sie diese Lösung bereit**.](#deploy-the-solution)

![In Azure ausgeführter Jenkins-Server][0]

*Laden Sie eine [Visio-Datei](https://archcenter.blob.core.windows.net/cdn/Jenkins-architecture.vsdx) herunter, die dieses Architekturdiagramm enthält.*

Diese Architektur unterstützt die Notfallwiederherstellung mit Azure-Diensten, deckt jedoch keine komplexeren Szenarien mit horizontaler Skalierung und mehreren Mastern oder Hochverfügbarkeit (High Availability, HA) ohne Downtime ab. Allgemeine Informationen zu den verschiedenen Azure-Komponenten, etwa ein ausführliches Tutorial zum Erstellen einer CI/CD-Pipeline in Azure, finden Sie unter [Jenkins in Azure][jenkins-on-azure].

Der Schwerpunkt dieses Dokuments liegt auf den zentralen Azure-Vorgängen, die zur Unterstützung von Jenkins erforderlich sind. Dazu zählen die Verwendung von Azure Storage zum Speichern von Buildartefakten, die für SSO erforderlichen Sicherheitselemente, andere integrierbare Dienste und Skalierbarkeit für die Pipeline. Die Architektur ist so konzipiert, dass sie mit einem vorhandenen Quellcodeverwaltungs-Repository verwendet werden kann. Ein allgemeines Szenario ist beispielsweise das Starten von Jenkins-Aufträgen auf der Grundlage von GitHub-Commits.

## <a name="architecture"></a>Architecture

Die Architektur umfasst die folgenden Komponenten:

- **Ressourcengruppe:** Eine [Ressourcengruppe][rg] dient zum Gruppieren von Azure-Ressourcen, damit sie nach Lebensdauer, Besitzer und anderen Kriterien verwaltet werden können. Verwenden Sie Ressourcengruppen, um Azure-Ressourcen als Gruppe bereitzustellen und zu überwachen und Abrechnungskosten nach Ressourcengruppe nachzuverfolgen. Sie können auch Ressourcen als Gruppe löschen, was für Testbereitstellungen sehr nützlich ist.

- **Jenkins-Server:** Ein virtueller Computer, der als Jenkins-Master fungiert, wird bereitgestellt, um [Jenkins][azure-market] als Automatisierungsserver auszuführen. Diese Referenzarchitektur verwendet die [Lösungsvorlage für Jenkins in Azure][solution] und ist auf einem virtuellen Linux-Computer (Ubuntu 16.04 LTS) in Azure installiert. Andere Jenkins-Angebote sind im Azure Marketplace erhältlich.

  > [!NOTE]
  > Nginx ist auf dem virtuellen Computer installiert und fungiert als Reverseproxy für Jenkins. Sie können Nginx zum Aktivieren von SSL für den Jenkins-Server konfigurieren.
  > 
  > 

- **Virtuelles Netzwerk**. Ein [virtuelles Netzwerk][vnet] verknüpft Azure-Ressourcen und bietet eine logische Isolierung. In dieser Architektur wird der Jenkins-Server in einem virtuellen Netzwerk ausgeführt.

- **Subnetze:** Der Jenkins-Server ist in einem [Subnetz][subnet] isoliert, um die Verwaltung und Aufteilung des Netzwerkdatenverkehrs ohne Auswirkungen auf die Leistung zu vereinfachen.

- <strong>NSGs</strong>. Verwenden Sie [Netzwerksicherheitsgruppen][nsg] (NSGs), um Netzwerkdatenverkehr vom Internet auf ein Subnetz eines virtuellen Netzwerks zu beschränken.

- **Verwaltete Datenträger:** Ein [verwalteter Datenträger][managed-disk] ist eine persistente virtuelle Festplatte (Virtual Hard Disk, VHD), die als Anwendungsspeicher sowie zum Speichern des Zustands des Jenkins-Servers und Bereitstellen der Notfallwiederherstellung verwendet wird. Datenträger werden in Azure Storage gespeichert. Für eine hohe Leistung wird [Storage Premium][premium] empfohlen.

- **Azure Blob Storage:** Das [Windows Azure Storage-Plug-In][configure-storage] nutzt Azure Blob Storage zum Speichern der Buildartefakte, die erstellt und für andere Jenkins-Builds freigegeben werden.

- <strong>Azure Active Directory (Azure AD):</strong> [Azure AD][azure-ad] unterstützt die Benutzerauthentifizierung, sodass Sie SSO einrichten können. Azure AD-[Dienstprinzipale][service-principal] definieren die Richtlinie und Berechtigungen für jede Rollenauthentifizierung im Workflow unter Verwendung der [rollenbasierten Zugriffssteuerung][rbac] (Role-Based Access Control, RBAC). Jeder Dienstprinzipal ist einem Jenkins-Auftrag zugeordnet.

- **Azure Key Vault:** Diese Architektur verwendet [Key Vault][key-vault] zum Verwalten von Geheimnissen und kryptografischen Schlüssel, die zum Bereitstellen von Azure-Ressourcen verwendet werden, wenn Geheimnisse erforderlich sind. Zusätzliche Hilfe beim Speichern von Geheimnissen, die der Anwendung in der Pipeline zugeordnet sind, finden Sie auch auf der Seite für das [Azure-Anmeldeinformationen-Plug-In][configure-credential] für Jenkins.

- **Azure-Überwachungsdienste:** Dieser Dienst [überwacht][monitor] den virtuellen Azure-Computer, auf dem Jenkins gehostet wird. Bei dieser Bereitstellung werden der Status des virtuellen Computers und die CPU-Auslastung überwacht und Warnungen gesendet.

## <a name="recommendations"></a>Empfehlungen

Die folgenden Empfehlungen gelten für die meisten Szenarios. Sofern Sie keine besonderen Anforderungen haben, die Vorrang haben, sollten Sie diese Empfehlungen befolgen.

### <a name="azure-ad"></a>Azure AD

Mit dem [Azure AD][azure-ad]-Mandanten für Ihr Azure-Abonnement wird SSO für Jenkins-Benutzer aktiviert. Darüber hinaus wird er zum Einrichten von [Dienstprinzipalen][service-principal] verwendet, die Jenkins-Aufträgen den Zugriff auf Azure-Ressourcen ermöglichen.

SSO-Authentifizierung und -Autorisierung werden von dem auf dem Jenkins-Server installierten Azure AD-Plug-In implementiert. SSO ermöglicht bei der Anmeldung beim Jenkins-Server die Authentifizierung mit Ihren Organisationsanmeldeinformationen aus Azure AD. Beim Konfigurieren des Azure AD-Plug-Ins können Sie die Ebene des autorisierten Zugriffs eines Benutzers auf den Jenkins-Server angeben.

Um Jenkins-Aufträgen den Zugriff auf Azure-Ressourcen zu gewähren, erstellt ein Azure AD-Administrator Dienstprinzipale. Diese gewähren Anwendungen – in diesem Fall den Jenkins-Aufträgen – [authentifizierten, autorisierten Zugriff][ad-sp] auf Azure-Ressourcen.

Mit der [rollenbasierten Zugriffssteuerung][rbac] wird der Zugriff auf Azure-Ressourcen für Benutzer oder Dienstprinzipale über die zugewiesene Rolle genauer definiert und gesteuert. Sowohl integrierte als auch benutzerdefinierte Rollen werden unterstützt. Rollen tragen darüber hinaus zum Schützen der Pipeline bei und stellen sicher, dass die Zuständigkeiten eines Benutzers oder Agents richtig zugewiesen und autorisiert werden. Darüber hinaus kann die RBAC eingerichtet werden, um den Zugriff auf Azure-Ressourcen zu beschränken. Beispielsweise kann für einen Benutzer die Nutzung auf die Ressourcen in einer bestimmten Ressourcengruppe beschränkt werden.

### <a name="storage"></a>Speicher

Erstellen Sie mit dem über den Azure Marketplace installierten [Windows Azure Storage-Plug-In][storage-plugin] von Jenkins Artefakte, die für andere Builds und Tests freigegeben werden können. Damit dieses Plug-In von den Jenkins-Aufträgen verwendet werden kann, muss ein Azure Storage-Konto konfiguriert werden.

### <a name="jenkins-azure-plugins"></a>Azure-Plug-Ins für Jenkins

Mit der Lösungsvorlage für Jenkins in Azure werden mehrere Azure-Plug-Ins installiert. Das Azure DevOps-Team erstellt und wartet die Lösungsvorlage und die folgenden Plug-Ins. Diese können mit anderen Jenkins-Angeboten im Azure Marketplace sowie auf jedem lokal eingerichteten Jenkins-Master verwendet werden:

-   Das [Azure AD-Plug-In][configure-azure-ad] ermöglicht dem Jenkins-Server auf der Grundlage von Azure AD die Unterstützung von SSO für Benutzer.

-   Das Plug-In für [Azure-VM-Agents][configure-agent] nutzt eine Azure Resource Manager-Vorlage zum Erstellen von Jenkins-Agents auf virtuellen Azure-Computern.

-   Das [Azure-Anmeldeinformationen-Plug-In][configure-credential] ermöglicht Ihnen das Speichern von Anmeldeinformationen des Azure-Dienstprinzipals in Jenkins.

-   Das [Microsoft Azure Storage-Plug-In][configure-storage] lädt Buildartefakte in [Azure Blob Storage][blob] hoch bzw. lädt Buildabhängigkeiten daraus herunter.

Es wird darüber hinaus empfohlen, die stetig wachsende Liste der verfügbaren Azure-Plug-Ins zu überprüfen, die mit Azure-Ressourcen verwendet werden können. Wenn Sie die aktuelle Liste anzeigen möchten, navigieren Sie zu [Jenkins Plugin Index][index] (Index der Jenkins-Plug-Ins), und suchen Sie nach Azure. Die folgenden Plug-Ins sind beispielsweise für die Bereitstellung verfügbar:

-   Das Plug-In für [Azure-Container-Agents][container-agents] ermöglicht die Ausführung eines Containers als Agent in Jenkins.

-   Mit dem Plug-In für die [kontinuierliche Kubernetes-Bereitstellung](https://aka.ms/azjenkinsk8s) werden Ressourcenkonfigurationen in einem Kubernetes-Cluster bereitgestellt.

-   Das [Azure Container Service][acs]-Plug-In stellt Konfigurationen in Azure Container Service mit Kubernetes, DC/OS mit Marathon oder Docker Swarm bereit.

-   [Azure Functions][functions] stellt Ihr Projekt in Azure Functions bereit.

-   [Azure App Service][app-service] dient zur Bereitstellung in Azure App Service.

## <a name="scalability-considerations"></a>Überlegungen zur Skalierbarkeit

Jenkins kann zur Unterstützung sehr großer Workloads skaliert werden. Führen Sie bei elastischen Builds keine Builds auf dem Jenkins-Masterserver aus. Lagern Sie stattdessen Buildtasks auf Jenkins-Agents aus, da diese elastisch horizontal herunter- und hochskaliert werden können. Erwägen Sie zwei Optionen für die Skalierung von Agents:

- Erstellen Sie mithilfe des Plug-Ins für [Azure-VM-Agents][vm-agent] Jenkins-Agents, die auf virtuellen Azure-Computern ausgeführt werden. Dieses Plug-In ermöglicht die elastische horizontale Skalierung für Agents und kann verschiedene Typen virtueller Computer verwenden. Sie können ein anderes Basisimage im Azure Marketplace oder ein benutzerdefiniertes Image auswählen. Ausführliche Informationen zum Skalieren von Jenkins-Agents finden Sie in der Jenkins-Dokumentation unter [Architecting for Scale][scale] (Entwerfen einer Architektur für die Skalierung).

- Verwenden Sie das Plug-In für [Azure-Container-Agents][container-agents], um einen Container als Agent in [Azure Container Service mit Kubernetes](/azure/container-service/kubernetes/) oder [Azure Container Instances](/azure/container-instances/) auszuführen.

Das Skalieren virtueller Computer ist im Allgemeinen teurer als das Skalieren von Containern. Damit Sie Container für die Skalierung verwenden können, muss Ihr Buildprozess jedoch mit Containern ausgeführt werden.

Verwenden Sie darüber hinaus Azure Storage für die Freigabe von Buildartefakten, die unter Umständen in der nächsten Phase der Pipeline von anderen Build-Agents verwendet werden.

### <a name="scaling-the-jenkins-server"></a>Skalieren des Jenkins-Servers 

Sie können den virtuellen Computer des Jenkins-Servers zentral hoch- oder herunterskalieren, indem Sie die VM-Größe ändern. In der [Lösungsvorlage für Jenkins in Azure][azure-market] ist standardmäßig die Größe „DS2 v2“ (mit zwei CPUs, 7 GB) angegeben. Diese Größe verarbeitet eine kleine bis mittlere Teamworkload. Ändern Sie die Größe des virtuellen Computers, indem Sie beim Erstellen des Servers eine andere Option auswählen. 

Die Auswahl der richtigen Servergröße hängt von der Größe der erwarteten Workload ab. Die Jenkins-Community stellt einen [Leitfaden für die Auswahl][selection-guide] bereit, anhand dessen Sie die Konfiguration ermitteln können, die Ihre Anforderungen am besten erfüllt. Azure bietet zahlreiche [Größen für virtuelle Linux-Computer][sizes-linux], um verschiedenste Anforderungen zu erfüllen. Weitere Informationen zum Skalieren des Jenkins-Masters finden Sie in den [Best Practices][best-practices] der Jenkins-Community.


## <a name="availability-considerations"></a>Überlegungen zur Verfügbarkeit

Verfügbarkeit im Kontext eines Jenkins-Servers bedeutet Folgendes: Sie können jegliche Zustandsinformationen im Zusammenhang mit Ihrem Workflow (etwa Testergebnisse, von Ihnen erstellte Bibliotheken oder andere Artefakte) wiederherstellen. Entscheidende Workflowzustände oder Artefakte müssen zur Wiederherstellung des Workflows beibehalten werden, falls der Jenkins-Server ausfällt. Berücksichtigen Sie beim Bewerten der Verfügbarkeitsanforderungen zwei allgemeine Metriken:

-   RTO (Recovery Time Objective) gibt an, wie lange Sie Vorgänge ohne Jenkins ausführen können.

-   RPO (Recovery Point Objective) gibt an, wie viele Daten verloren gehen dürfen, wenn sich eine Dienstunterbrechung auf Jenkins auswirkt.

In der Praxis geben RTO und RPO Redundanz und Sicherung an. Bei der Verfügbarkeit geht es nicht um Hardwarewiederherstellung (diese ist Teil von Azure), sondern darum sicherzustellen, dass der Zustand des Jenkins-Servers gespeichert wird. Microsoft bietet eine [Vereinbarung zum Servicelevel][sla] (Service Level Agreement, SLA) für einzelne VM-Instanzen an. Falls diese SLA Ihre Verfügbarkeitsanforderungen nicht erfüllt, stellen Sie sicher, dass Sie einen Tarif für die Notfallwiederherstellung besitzen, oder ziehen Sie die Bereitstellung eines [Jenkins-Servers mit mehreren Mastern ][multi-master] (nicht in diesem Dokument behandelt) in Betracht.

Erwägen Sie die Verwendung der [Skripts][disaster] für die Notfallwiederherstellung in Schritt 7 der Bereitstellung, um ein Azure Storage-Konto mit verwalteten Datenträgern zum Speichern des Jenkins-Serverzustands zu erstellen. Wenn Jenkins ausfällt, kann der Systemzustand wiederhergestellt werden, der in diesem separaten Speicherkonto gespeichert ist.

## <a name="security-considerations"></a>Sicherheitshinweise

Verwenden Sie zum Schutz eines grundlegenden Jenkins-Servers die folgenden Vorgehensweisen, da ein Jenkins-Server in seinem Grundzustand nicht sicher ist.

-   Richten Sie eine sichere Methode für die Anmeldung beim Jenkins-Server ein. Diese Architektur verwendet HTTP und besitzt eine öffentliche IP, HTTP ist jedoch nicht standardmäßig sicher. Ziehen Sie die Einrichtung von [HTTPS auf dem Nginx-Server][nginx] in Betracht, der für die sichere Anmeldung verwendet wird.

    > [!NOTE]
    > Wenn Sie Ihrem Server SSL hinzufügen, erstellen Sie eine NSG-Regel für das Jenkins-Subnetz, damit Port 443 geöffnet wird. Weitere Informationen finden Sie unter [Öffnen von Ports zu einem virtuellen Computer mit dem Azure-Portal][port443].
    > 

-   Stellen Sie sicher, dass die Jenkins-Konfiguration die websiteübergreifende Anforderungsfälschung verhindert (Manage Jenkins \> Configure Global Security (Jenkins verwalten > Globale Sicherheit konfigurieren)). Dies ist die Standardeinstellung für einen Microsoft Jenkins-Server.

-   Konfigurieren Sie den schreibgeschützten Zugriff auf das Jenkins-Dashboard mithilfe des [Plug-Ins für die Matrixautorisierungsstrategie][matrix].

-   Installieren Sie das [Azure-Anmeldeinformationen-Plug-In][configure-credential], um mithilfe von Key Vault Geheimnisse für die Azure-Ressourcen, die Agents in der Pipeline und Drittanbieterkomponenten zu verarbeiten.

-   Verwenden Sie RBAC, um den Zugriff des Dienstprinzipals auf die Mindestanforderungen für die Auftragsausführung zu beschränken. Dies dient zur Begrenzung des Schadens, der durch einen nicht autorisierten Auftrag entstehen kann.

Jenkins-Aufträge benötigen häufig Geheimnisse für den Zugriff auf Azure-Dienste, für die eine Autorisierung erforderlich ist, etwa Azure Container Service. Verwenden Sie [Key Vault][key-vault] zusammen mit dem [Azure-Anmeldeinformationen-Plug-In][configure-credential] für die sichere Verwaltung dieser Geheimnisse. Speichern Sie mithilfe von Key Vault Dienstprinzipal-Anmeldeinformationen, Kennwörter, Token und andere Geheimnisse.

Eine zentrale Übersicht über den Sicherheitszustand Ihrer Azure-Ressourcen können Sie sich mit [Azure Security Center][security-center] verschaffen. Mit Security Center werden potenzielle Sicherheitsprobleme überwacht, und Sie erhalten eine umfassende Darstellung des Sicherheitszustands Ihrer Bereitstellung. Security Center wird für jedes Azure-Abonnement individuell konfiguriert. Aktivieren Sie die Sammlung von Sicherheitsdaten wie im [Schnellstarthandbuch zu Azure Security Center][quick-start] beschrieben. Nachdem die Datensammlung aktiviert wurde, durchsucht Security Center virtuelle Computer automatisch, die unter diesem Abonnement erstellt werden.

Der Jenkins-Server verfügt über ein eigenes Benutzerverwaltungssystem, und die Jenkins-Community stellt Best Practices zum [Sichern einer Jenkins-Instanz in Azure][secure-jenkins] zur Verfügung. Die Lösungsvorlage für Jenkins in Azure implementiert diese Best Practices.

## <a name="manageability-considerations"></a>Überlegungen zur Verwaltbarkeit

Verwenden Sie Ressourcengruppen, um die bereitgestellten Azure-Ressourcen zu organisieren. Stellen Sie Produktionsumgebungen und Entwicklungs-/Testumgebungen in separaten Ressourcengruppen bereit, sodass Sie die Ressourcen der einzelnen Umgebungen überwachen und Abrechnungskosten nach Ressourcengruppe zusammenfassen können. Sie können auch Ressourcen als Gruppe löschen, was für Testbereitstellungen sehr nützlich ist.

In Azure sind mehrere Features für die [Überwachung und Diagnose][monitoring-diag] der Gesamtstruktur enthalten. Zur Überwachung der CPU-Auslastung wird in dieser Architektur Azure Monitor bereitgestellt. Sie können beispielsweise mithilfe von Azure Monitor die CPU-Auslastung überwachen und eine Benachrichtigung senden, wenn die CPU-Auslastung 80 Prozent überschreitet. (Eine hohe CPU-Auslastung weist darauf hin, dass Sie unter Umständen den virtuellen Computer mit dem Jenkins-Server zentral hochskalieren sollten.) Sie können auch einen angegebenen Benutzer benachrichtigen, wenn der virtuelle Computer ausfällt oder nicht mehr verfügbar ist.

## <a name="communities"></a>Communitys

Communitys können Fragen beantworten und Sie beim Einrichten einer erfolgreichen Bereitstellung unterstützen. Beachten Sie Folgendes:

-   [Blog der Jenkins-Community](https://jenkins.io/node/)
-   [Azure-Forum](https://azure.microsoft.com/support/forums/)
-   [Stack Overflow: Jenkins](https://stackoverflow.com/tags/jenkins/info)

Weitere Best Practices der Jenkins-Community finden Sie unter [Jenkins best practices][jenkins-best] (Best Practices für Jenkins).

## <a name="deploy-the-solution"></a>Bereitstellen der Lösung

Führen Sie zum Bereitstellen dieser Architektur die folgenden Schritte aus, um die [Lösungsvorlage für Jenkins in Azure][azure-market] zu installieren, und installieren Sie anschließend die Skripts, mit denen die Überwachung und Notfallwiederherstellung eingerichtet wird.

### <a name="prerequisites"></a>Voraussetzungen

- Für diese Referenzarchitektur ist ein Azure-Abonnement erforderlich. 
- Zum Erstellen eines Azure-Dienstprinzipals sind Administratorrechte für den Azure AD-Mandanten erforderlich, der dem bereitgestellten Jenkins-Server zugeordnet ist.
- Bei diesen Anweisungen wird davon ausgegangen, dass der Jenkins-Administrator gleichzeitig ein Azure-Benutzer ist, der mindestens über die Berechtigung „Mitwirkender“ verfügt.

### <a name="step-1-deploy-the-jenkins-server"></a>Schritt 1: Bereitstellen des Jenkins-Servers

1.  Öffnen Sie in Ihrem Webbrowser das [Azure Marketplace-Image für Jenkins][azure-market], und wählen links auf der Seite **JETZT ANFORDERN** aus.

2.  Sehen Sie sich die Preisdetails an, und wählen Sie **Weiter** aus. Wählen Sie anschließend **Erstellen** aus, um den Jenkins-Server über das Azure-Portal zu konfigurieren.

Ausführliche Informationen finden Sie unter [Erstellen eines Jenkins-Servers auf einem virtuellen Azure-Linux-Computer über das Azure-Portal][create-jenkins]. Für diese Referenzarchitektur reicht es, den Server mit der Administratoranmeldung betriebsbereit zu machen. Anschließend können Sie ihn für die Verwendung verschiedener anderer Dienste bereitstellen.

### <a name="step-2-set-up-sso"></a>Schritt 2: Einrichten von SSO

Dieser Schritt wird vom Jenkins-Administrator ausgeführt, der außerdem ein Benutzerkonto im Azure AD-Verzeichnis des Abonnements besitzen und der Rolle „Mitwirkender“ zugewiesen sein muss.

Verwenden Sie das [Azure AD-Plug-In][configure-azure-ad] aus dem Jenkins Update Center auf dem Jenkins-Server, und befolgen Sie die Anweisungen zum Einrichten von SSO.

### <a name="step-3-provision-jenkins-server-with-azure-vm-agent-plugin"></a>Schritt 3: Bereitstellen des Jenkins-Servers mit dem Azure-VM-Agent-Plug-In

Dieser Schritt wird vom Jenkins-Administrator zum Einrichten des bereits installierten Azure-VM-Agent-Plug-Ins ausgeführt.

Führen Sie zum Konfigurieren des Plug-Ins [diese Schritte][configure-agent] aus. Ein Tutorial zum Einrichten von Dienstprinzipalen für das Plug-In finden Sie unter [Skalieren Ihrer Jenkins-Bereitstellungen für verschiedene Anforderungen mit Azure-VM-Agents][scale-agent].

### <a name="step-4-provision-jenkins-server-with-azure-storage"></a>Schritt 4: Bereitstellen des Jenkins-Servers mit Azure Storage

Dieser Schritt wird vom Jenkins-Administrator ausgeführt, der das bereits installierte Windows Azure Storage-Plug-In einrichtet.

Führen Sie zum Konfigurieren des Plug-Ins [diese Schritte][configure-storage] aus.

### <a name="step-5-provision-jenkins-server-with-azure-credential-plugin"></a>Schritt 5: Bereitstellen des Jenkins-Servers mit dem Azure Anmeldeinformationen-Plug-In

Dieser Schritt wird vom Jenkins-Administrator zum Einrichten des bereits installierten Azure Anmeldeinformationen-Plug-Ins ausgeführt.

Führen Sie zum Konfigurieren des Plug-Ins [diese Schritte][configure-credential] aus.

### <a name="step-6-provision-jenkins-server-for-monitoring-by-the-azure-monitor-service"></a>Schritt 6: Bereitstellen des Jenkins-Servers für die Überwachung durch den Azure Monitor-Dienst

Befolgen Sie zum Einrichten der Überwachung Ihres Jenkins-Servers die Anweisungen unter [Erstellen von Metrikwarnungen in Azure Monitor für Azure-Dienste – Azure-Portal][create-metric].

### <a name="step-7-provision-jenkins-server-with-managed-disks-for-disaster-recovery"></a>Schritt 7: Bereitstellen des Jenkins-Servers mit Managed Disks für die Notfallwiederherstellung

Die Microsoft Jenkins-Produktgruppe hat Notfallwiederherstellungsskripts entwickelt, mit denen ein verwalteter Datenträger zum Speichern des Jenkins-Zustands erstellt wird. Wenn der Server ausfällt, kann sein letzter Zustand wiederhergestellt werden.

Laden Sie die Notfallwiederherstellungsskripts von [GitHub][disaster] herunter, und installieren Sie sie.

[acs]: https://aka.ms/azjenkinsacs
[ad-sp]: /azure/active-directory/develop/active-directory-integrating-applications
[app-service]: https://plugins.jenkins.io/azure-app-service
[azure-ad]: /azure/active-directory/
[azure-market]: https://azuremarketplace.microsoft.com/marketplace/apps/azure-oss.jenkins?tab=Overview
[best-practices]: https://jenkins.io/doc/book/architecting-for-scale/
[blob]: /azure/storage/common/storage-java-jenkins-continuous-integration-solution
[configure-azure-ad]: https://plugins.jenkins.io/azure-ad
[configure-agent]: https://plugins.jenkins.io/azure-vm-agents
[configure-credential]: https://plugins.jenkins.io/azure-credentials
[configure-storage]: https://plugins.jenkins.io/windows-azure-storage
[container-agents]: https://aka.ms/azcontaineragent
[create-jenkins]: /azure/jenkins/install-jenkins-solution-template
[create-metric]: /azure/monitoring-and-diagnostics/insights-alerts-portal
[disaster]: https://github.com/Azure/jenkins/tree/master/disaster_recovery
[functions]: https://aka.ms/azjenkinsfunctions
[index]: https://plugins.jenkins.io
[jenkins-best]: https://wiki.jenkins.io/display/JENKINS/Jenkins+Best+Practices
[jenkins-on-azure]: /azure/jenkins/
[key-vault]: /azure/key-vault/
[managed-disk]: /azure/virtual-machines/linux/managed-disks-overview
[matrix]: https://plugins.jenkins.io/matrix-auth
[monitor]: /azure/monitoring-and-diagnostics/
[monitoring-diag]: /azure/architecture/best-practices/monitoring
[multi-master]: https://jenkins.io/doc/book/architecting-for-scale/
[nginx]: https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04
[nsg]: /azure/virtual-network/virtual-networks-nsg
[quick-start]: /azure/security-center/security-center-get-started
[port443]: /azure/virtual-machines/windows/nsg-quickstart-portal
[premium]: /azure/virtual-machines/linux/premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rg]: /azure/azure-resource-manager/resource-group-overview
[scale]: https://jenkins.io/doc/book/architecting-for-scale/
[scale-agent]: /azure/jenkins/jenkins-azure-vm-agents
[selection-guide]: https://jenkins.io/doc/book/hardware-recommendations/
[service-principal]: /azure/active-directory/develop/active-directory-application-objects
[secure-jenkins]: https://jenkins.io/blog/2017/04/20/secure-jenkins-on-azure/
[security-center]: /azure/security-center/security-center-intro
[sizes-linux]: /azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json
[solution]: https://azure.microsoft.com/blog/announcing-the-solution-template-for-jenkins-on-azure/
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/
[storage-plugin]: https://wiki.jenkins.io/display/JENKINS/Windows+Azure+Storage+Plugin
[subnet]: /azure/virtual-network/virtual-network-manage-subnet
[vm-agent]: https://wiki.jenkins.io/display/JENKINS/Azure+VM+Agents+plugin
[vnet]: /azure/virtual-network/virtual-networks-overview
[0]: ./images/jenkins-server.png 
