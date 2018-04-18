---
title: Ausführen von Windows-VMs für eine n-schichtige Architektur
description: Vorgehensweise zur Implementierung einer mehrschichtigen Architektur in Azure mit Fokus auf eine zuverlässige Verfügbarkeit, Sicherheit, Skalierbarkeit und Verwaltbarkeit
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Windows VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 5ed94eb9ab8203d35d9597336e367d54e03944d7
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="run-windows-vms-for-an-n-tier-application"></a>Ausführen von Windows-VMs für eine n-schichtige Anwendung

Anhand dieser Referenzarchitektur werden einige bewährte Methoden für die Ausführung virtueller Windows-Computer (VMs) für eine n-schichtige Anwendung veranschaulicht. [**So stellen Sie diese Lösung bereit**.](#deploy-the-solution) 

![[0]][0]

*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*

## <a name="architecture"></a>Architecture 

Es gibt viele Möglichkeiten für die Implementierung einer n-schichtigen Architektur. Das Diagramm zeigt eine typische 3-schichtige Webanwendung. Diese Architektur baut auf den Informationen unter [Run load-balanced VMs for scalability and availability][multi-vm] (Ausführen von VMs mit Lastenausgleich zur Erzielung von Skalierbarkeit und Verfügbarkeit) auf. In der Internet- und Unternehmensschicht werden VMs mit Lastenausgleich verwendet.

* **Verfügbarkeitsgruppen:** Erstellen Sie eine [Verfügbarkeitsgruppe][azure-availability-sets] für jede Schicht, und stellen Sie mindestens zwei VMs in jeder Schicht bereit. Dies berechtigt die VMs zu einer höheren [Vereinbarung zum Servicelevel (SLA)][vm-sla] für VMs. Sie können eine einzelne VM in einer Verfügbarkeitsgruppe bereitstellen, die einzelne VM ist jedoch nicht für eine SLA-Garantie qualifiziert, es sei denn, die VM verwendet Azure Premium Storage für alle Betriebssystem- und Datenfestplatten.  
* **Subnetze:** Erstellen Sie für jede Schicht ein separates Subnetz. Geben Sie mit der [CIDR]-Notation den Adressbereich und die Subnetzmaske an. 
* **Lastenausgleichsmodule:** Verwenden Sie ein [Lastenausgleichsmodul mit Internetzugriff][load-balancer-external], um eingehenden Internetdatenverkehr auf die Internetschicht zu verteilen, und ein [internes Lastenausgleichsmodul][load-balancer-internal], um den Netzwerkdatenverkehr von der Internetschicht auf die Unternehmensschicht zu verteilen.
* **Jumpbox:** Wird auch als [geschützter Host] bezeichnet. Dies ist eine geschützte VM im Netzwerk, die von Administratoren zum Herstellen der Verbindung mit anderen VMs verwendet wird. Die Jumpbox verfügt über eine NSG, die Remotedatenverkehr nur von öffentlichen IP-Adressen zulässt, die in einer Liste mit sicheren Absendern aufgeführt sind. Die NSG sollte Remotedesktop-Datenverkehr (RDP) zulassen.
* **Überwachung:** Mit Überwachungssoftware wie [Nagios], [Zabbix] oder [Icinga] können Informationen über die Antwortzeit, die VM-Betriebszeit und die allgemeine Integrität Ihres Systems bereitstellen. Installieren Sie die Überwachungssoftware auf einer VM, die sich in einem separaten Verwaltungssubnetz befindet.
* <strong>NSGs:</strong> Verwenden Sie [Netzwerksicherheitsgruppen][nsg] (NSGs), um den Netzwerkdatenverkehr im VNET zu beschränken. In der hier gezeigten 3-schichtigen Architektur akzeptiert die Datenbankschicht beispielsweise keinen Datenverkehr vom Web-Front-End, sondern nur von der Unternehmensschicht und dem Verwaltungssubnetz.
* **SQL Server Always On-Verfügbarkeitsgruppe:** Stellt Hochverfügbarkeit in der Datenschicht durch Replikation und Failover bereit.
* **AD DS-Server (Active Directory Domain Services):** Bei älteren Versionen als Windows Server 2016 müssen SQL Server Always On-Verfügbarkeitsgruppen Mitglied einer Domäne sein. Dies ist auf die Abhängigkeit der Verfügbarkeitsgruppen von der Windows Server Failover Cluster-Technologie (WSFC) zurückzuführen. Windows Server 2016 bietet die Möglichkeit, einen Failovercluster ohne Active Directory zu erstellen. In diesem Fall sind die AD DS-Server für diese Architektur nicht erforderlich. Weitere Informationen finden Sie unter [Neuerungen beim Failoverclustering unter Windows Server 2016][wsfc-whats-new].
* **Azure DNS:** [Azure DNS][azure-dns] ist ein Hostingdienst für DNS-Domänen, der die Namensauflösung unter Verwendung der Microsoft Azure-Infrastruktur durchführt. Durch das Hosten Ihrer Domänen in Azure können Sie Ihre DNS-Einträge mithilfe der gleichen Anmeldeinformationen, APIs, Tools und Abrechnung wie für die anderen Azure-Dienste verwalten.

## <a name="recommendations"></a>Empfehlungen

Ihre Anforderungen können von der hier beschriebenen Architektur abweichen. Verwenden Sie diese Empfehlungen als Startpunkt. 

### <a name="vnet--subnets"></a>VNET/Subnetze

Legen Sie bei der Erstellung des VNET fest, wie viele IP-Adressen Ihre Ressourcen in jedem Subnetz benötigen. Geben Sie mithilfe der [CIDR] eine Subnetzmaske und einen VNET-Adressbereich an, der für die erforderlichen IP-Adressen groß genug ist. Verwenden Sie einen Adressraum, der in die standardmäßigen [privaten IP-Adressblöcke][private-ip-space] 10.0.0.0/8, 172.16.0.0/12 und 192.168.0.0/16 fällt.

Wählen Sie einen Adressbereich, der sich nicht mit Ihrem lokalen Netzwerk überschneidet, für den Fall, dass Sie später ein Gateway zwischen dem VNET und dem lokalen Netzwerk einrichten müssen. Sobald Sie das VNET erstellt haben, können Sie den Adressbereich nicht mehr ändern.

Entwerfen Sie Subnetze unter Berücksichtigung der Funktionalität und Sicherheitsanforderungen. Alle VMs innerhalb derselben Schicht oder Rolle sollten im selben Subnetz platziert werden, was eine Sicherheitsbegrenzung darstellen kann. Weitere Informationen zum Entwerfen von VNETs und Subnetzen finden Sie unter [Planen und Entwerfen von Azure Virtual Networks][plan-network].

Geben Sie für jedes Subnetz den Adressraum für das Subnetz in CIDR-Notation an. Mit „10.0.0.0/24“ wird beispielsweise ein Bereich von 256 IP-Adressen erstellt. Von diesen sind 251 für VMs nutzbar; die restlichen fünf sind reserviert. Achten Sie darauf, dass sich die Adressbereiche nicht mit anderen Subnetzen überlappen. Weitere Informationen finden Sie unter [Azure Virtual Network – häufig gestellte Fragen][vnet faq].

### <a name="network-security-groups"></a>Netzwerksicherheitsgruppen

Verwenden Sie NSG-Regeln, um den Datenverkehr zwischen den Schichten zu beschränken. In der oben gezeigten 3-schichtigen Architektur kommuniziert die Internetschicht beispielsweise nicht direkt mit der Datenbankschicht. Um dies zu erzwingen, sollte die Datenbankschicht eingehenden Datenverkehr aus dem Subnetz der Internetschicht blockieren.  

1. Erstellen Sie eine NSG, und ordnen Sie diese dem Subnetz der Datenbankschicht zu.
2. Fügen Sie eine Regel hinzu, die den gesamten eingehenden Datenverkehr vom VNET ablehnt. (Verwenden Sie den `VIRTUAL_NETWORK`-Tag in der Regel.) 
3. Fügen Sie eine Regel mit einer höheren Priorität hinzu, die eingehenden Datenverkehr aus dem Subnetz der Unternehmensschicht zulässt. Diese Regel überschreibt die vorherige Regel und ermöglicht die Kommunikation zwischen der Unternehmens- und Datenbankschicht.
4. Fügen Sie eine Regel hinzu, die eingehenden Datenverkehr innerhalb des Subnetzes der Datenbankschicht selbst zulässt. Diese Regel ermöglicht die Kommunikation zwischen VMs in der Datenbankschicht, die für die Replikation und das Failover der Datenbank erforderlich ist.
5. Fügen Sie eine Regel hinzu, die RDP-Datenverkehr vom Subnetz der Jumpbox zulässt. Diese Regel erlaubt Administratoren, über die Jumpbox eine Verbindung mit der Datenbankschicht herzustellen.
   
   > [!NOTE]
   > Eine NSG ist mit Standardregeln versehen, die sämtlichen eingehenden Datenverkehr im VNET zulassen. Diese Regeln können nicht gelöscht, aber durch die Erstellung von Regeln mit höherer Priorität überschrieben werden.
   > 
   > 

### <a name="load-balancers"></a>Load Balancer

Das externe Lastenausgleichsmodul verteilt den Internetdatenverkehr auf die Internetschicht. Erstellen Sie eine öffentliche IP-Adresse für dieses Lastenausgleichsmodul. Weitere Informationen finden Sie unter [Erstellen eines Load Balancers mit Internetzugriff im Azure-Portal][lb-external-create].

Das interne Lastenausgleichsmodul verteilt den Netzwerkdatenverkehr von der Internetschicht auf die Unternehmensschicht. Um diesem Lastenausgleichsmodul eine private IP-Adresse zuzuweisen, erstellen Sie eine Front-End-IP-Konfiguration, und ordnen Sie diese dem Subnetz für die Unternehmensschicht zu. Weitere Informationen finden Sie unter [Erstellen eines internen Lastenausgleichs über das Azure-Portal][lb-internal-create].

### <a name="sql-server-always-on-availability-groups"></a>SQL Server Always On-Verfügbarkeitsgruppen

Um Hochverfügbarkeit von SQL Server zu erzielen, werden [AlwaysOn-Verfügbarkeitsgruppen][sql-alwayson] empfohlen. Bei früheren Versionen als Windows Server 2016 erfordern AlwaysOn-Verfügbarkeitsgruppen einen Domänencontroller, und alle Knoten in der Verfügbarkeitsgruppe müssen sich in der gleichen AD-Domäne befinden.

Andere Schichten stellen über einen [Verfügbarkeitsgruppenlistener][sql-alwayson-listeners] eine Verbindung mit der Datenbank her. Mit dem Listener kann ein SQL-Client eine Verbindung herstellen, ohne den Namen der physischen SQL Server-Instanz zu kennen. VMs, die auf die Datenbank zugreifen, müssen Mitglied der Domäne sein. Der Client (in diesem Fall eine andere Schicht) verwendet DNS, um den Namen des virtuellen Netzwerks des Listeners in IP-Adressen aufzulösen.

Konfigurieren Sie die SQL Server-Always On-Verfügbarkeitsgruppe wie folgt:

1. Erstellen Sie einen WSFC-Cluster (Windows Server Failover Clustering), eine SQL Server Always On-Verfügbarkeitsgruppe und ein primäres Replikat. Weitere Informationen finden Sie unter [Erste Schritte mit Always On-Verfügbarkeitsgruppen (SQL Server)][sql-alwayson-getting-started]. 
2. Erstellen Sie ein internes Lastenausgleichsmodul mit einer statischen privaten IP-Adresse.
3. Erstellen Sie einen Verfügbarkeitsgruppenlistener, und ordnen Sie den DNS-Namen des Listeners der IP-Adresse eines internen Lastenausgleichsmoduls zu. 
4. Erstellen Sie eine Lastenausgleichsregel für den SQL Server-Überwachungsport (standardmäßig TCP-Port 1433). Die Lastenausgleichsregel muss *Floating IP*, auch als „Direct Server Return“ bezeichnet, aktivieren. Dies bewirkt, dass die VM direkt auf den Client antwortet, was eine direkte Verbindung mit dem primären Replikat ermöglicht.
  
   > [!NOTE]
   > Wenn die Floating IP aktiviert ist, muss die Portnummer des Front-Ends mit der des Back-Ends in der Lastenausgleichsregel übereinstimmen.
   > 
   > 

Wenn ein SQL-Client versucht, eine Verbindung herzustellen, leitet das Lastenausgleichsmodul die Verbindungsanforderung an das primäre Replikat weiter. Bei einem Failover zu einem anderen Replikat leitet das Lastenausgleichsmodul nachfolgende Anforderungen automatisch an ein neues primäres Replikat weiter. Weitere Informationen finden Sie unter [Konfigurieren eines ILB-Listeners für AlwaysOn-Verfügbarkeitsgruppen in Azure][sql-alwayson-ilb].

Bei einem Failover werden bestehende Clientverbindungen geschlossen. Nachdem das Failover abgeschlossen ist, werden neue Verbindungen an das neue primäre Replikat weitergeleitet.

Wenn Ihre Anwendung deutlich mehr Lesezugriffe als Schreibzugriffe tätigt, können Sie einige der schreibgeschützten Abfragen in ein sekundäres Replikat auslagern. Weitere Informationen finden Sie unter [Verwenden eines Listeners zum Herstellen einer Verbindung mit einem schreibgeschützten sekundären Replikat (schreibgeschütztes Routing)][sql-alwayson-read-only-routing].

Testen Sie Ihre Bereitstellung, indem Sie [ein manuelles Failover der Verfügbarkeitsgruppe erzwingen][sql-alwayson-force-failover].

### <a name="jumpbox"></a>Jumpbox

Die Jumpbox setzt minimale Leistungsanforderungen voraus. Wählen Sie daher eine kleine VM-Größe für die Jumpbox wie „Standard A1“. 

Erstellen Sie eine [öffentliche IP-Adresse] für die Jumpbox. Platzieren Sie die Jumpbox in dasselbe VNET wie die anderen VMs, jedoch in ein separates Verwaltungssubnetz.

Verweigern Sie den RDP-Zugriff über das öffentliche Internet auf die VMs, auf denen die Anwendungsworkload ausgeführt wird. Der gesamte RDP-Zugriff auf diese VMs muss stattdessen über die Jumpbox erfolgen. Ein Administrator meldet sich bei der Jumpbox und von der Jumpbox aus dann bei der anderen VM an. Die Jumpbox lässt RDP-Datenverkehr aus dem Internet zu, jedoch nur von bekannten, sicheren IP-Adressen.

Erstellen Sie zum Schutz der Jumpbox eine NSG, und wenden Sie diese auf das Subnetz der Jumpbox an. Fügen Sie eine NSG-Regel hinzu, die ausschließlich RDP-Verbindungen von einer sicheren Gruppe öffentlicher IP-Adressen zulässt. Die NSG kann entweder dem Subnetz oder der Jumpbox-NIC hinzugefügt werden. In diesem Fall empfehlen wir, diese der NIC hinzuzufügen, sodass RDP-Datenverkehr nur an die Jumpbox erlaubt ist, auch wenn Sie andere VMs zum selben Subnetz hinzufügen.

Konfigurieren Sie die NSGs für die anderen Subnetze, um RDP-Datenverkehr aus dem Verwaltungssubnetz zuzulassen.

## <a name="availability-considerations"></a>Überlegungen zur Verfügbarkeit

Durch das Vorhandensein mehrerer VMs in der Datenbankschicht wird nicht automatisch eine hochverfügbare Datenbank erstellt. Bei einer relationalen Datenbank müssen Sie in der Regel Replikation und Failover einsetzen, um Hochverfügbarkeit zu erzielen. Für SQL Server wird die Verwendung von [AlwaysOn-Verfügbarkeitsgruppen][sql-alwayson] empfohlen. 

Wenn Sie eine höhere Hochverfügbarkeit benötigen, als die [Azure-SLA für VMs][vm-sla] bietet, replizieren Sie die Anwendung über zwei Regionen, und verwenden Sie für das Failover den Azure Traffic Manager. Weitere Informationen finden Sie unter [Ausführen von Windows-VMs in mehreren Regionen für Hochverfügbarkeit][multi-dc].   

## <a name="security-considerations"></a>Sicherheitshinweise

Verschlüsseln Sie sensible ruhende Daten, und verwalten Sie die Datenbankverschlüsselungsschlüssel mithilfe von [Azure Key Vault][azure-key-vault]. Key Vault kann Verschlüsselungsschlüssel in Hardwaresicherheitsmodulen (HSMs) speichern. Weitere Informationen finden Sie unter [Konfigurieren der Azure Key Vault-Integration für SQL Server auf virtuellen Azure-Computern (Resource Manager)][sql-keyvault]. Es wird außerdem empfohlen, Anwendungsgeheimnisse wie Datenbankverbindungszeichenfolgen in Key Vault zu speichern.

Für die Erstellung einer DMZ zwischen dem Internet und dem virtuellen Azure-Netzwerk sollten Sie eventuell eine virtuelle Netzwerkappliance (Network Virtual Appliance, NVA) hinzufügen. NVA ist ein Oberbegriff für eine virtuelle Appliance, die netzwerkbezogene Aufgaben wie Erstellung von Firewalls, Paketüberprüfung, Überwachung und benutzerdefiniertes Routing ausführen kann. Weitere Informationen finden Sie unter [Implementieren einer DMZ zwischen Azure und dem Internet][dmz].

## <a name="scalability-considerations"></a>Überlegungen zur Skalierbarkeit

Die Lastenausgleichsmodule verteilen den Netzwerkdatenverkehr auf die Internet- und Unternehmensschicht. Führen Sie eine horizontale Skalierung durch, indem Sie neue VM-Instanzen hinzufügen. Beachten Sie, dass die Internet- und Unternehmensschicht je nach Auslastung unabhängig voneinander skaliert werden können. Um das Risiko von Komplikationen zu reduzieren, die durch die Notwendigkeit zur Aufrechterhaltung der Clientaffinität verursacht werden, sollten die VMs in der Internetschicht zustandslos sein. Die VMs, die die Geschäftslogik hosten, sollten ebenfalls zustandslos sein.

## <a name="manageability-considerations"></a>Überlegungen zur Verwaltbarkeit

Vereinfachen Sie die Verwaltung des gesamten Systems durch den Einsatz von zentralisierten Verwaltungstools wie [Azure Automation][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef] oder [Puppet][puppet]. Diese Tools können Diagnose- und Integritätsinformationen konsolidieren, die von mehreren VMs erfasst wurden, um einen allgemeinen Überblick über das System bereitzustellen.

## <a name="deploy-the-solution"></a>Bereitstellen der Lösung

Eine Bereitstellung für diese Referenzarchitektur ist auf [GitHub][github-folder] verfügbar. 

### <a name="prerequisites"></a>Voraussetzungen

Bevor Sie die Referenzarchitektur in Ihrem eigenen Abonnement bereitstellen können, müssen Sie die folgenden Schritte ausführen.

1. Klonen oder Forken Sie das GitHub-Repository [Referenzarchitekturen][ref-arch-repo], oder laden Sie die entsprechende ZIP-Datei herunter.

2. Vergewissern Sie sich, dass Azure CLI 2.0 auf Ihrem Computer installiert ist. Um die Befehlszeilenschnittstelle zu installieren, befolgen Sie die Anweisungen unter [Installieren von Azure CLI 2.0][azure-cli-2].

3. Installieren Sie das npm-Paket mit den [Azure Bausteinen][azbb].

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

4. Melden Sie sich über eine Eingabeaufforderung, eine bash-Eingabeaufforderung oder die PowerShell-Eingabeaufforderung bei Ihrem Azure-Konto an. Verwenden Sie dazu die unten aufgeführten Befehle, und befolgen Sie die Anweisungen.

   ```bash
   az login
   ```

### <a name="deploy-the-solution-using-azbb"></a>Bereitstellen der Lösung mit azbb

Um die Windows-VMs für eine n-schichtige Anwendungsreferenzarchitektur bereitzustellen, führen Sie die folgenden Schritte aus:

1. Navigieren Sie zum `virtual-machines\n-tier-windows`-Ordner für das Repository, das Sie oben in Schritt 1 der Voraussetzungen als Klon erstellt haben.

2. Die Parameterdatei gibt einen Standard-Administratorbenutzernamen und ein Standardkennwort für jede VM in der Bereitstellung an. Sie müssen diese vor der Bereitstellung der Referenzarchitektur ändern. Öffnen Sie die `n-tier-windows.json`-Datei, und ersetzen Sie jedes Feld **adminUsername** und **adminPassword** durch neue Einstellungen.
  
   > [!NOTE]
   > Während dieser Bereitstellung werden sowohl in den **VirtualMachineExtension**-Objekten als auch in den **extensions**-Einstellungen für einige der **VirtualMachine**-Objekte mehrere Skripts ausgeführt. Für einige dieser Skripts sind der Administratorbenutzername und das Kennwort erforderlich, die Sie soeben geändert haben. Wir empfehlen Ihnen, diese Skripts zu überprüfen, um sicherzustellen, dass Sie die richtigen Anmeldeinformationen angegeben haben. Wenn Sie nicht die richtigen Anmeldeinformationen angegeben haben, tritt bei der Bereitstellung möglicherweise ein Fehler auf.
   > 
   > 

Speichern Sie die Datei .

3. Stellen Sie die Referenzarchitektur mithilfe des Befehlszeilentools **azbb** bereit, wie unten dargestellt.

   ```bash
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-windows.json --deploy
   ```

Weitere Informationen zum Bereitstellen dieser Beispielreferenzarchitektur mithilfe von Azure-Bausteinen finden Sie im [GitHub-Repository][git].


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-application.md
[multi-vm]: multi-vm.md
[n-tier]: n-tier.md
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[geschützter Host]: https://en.wikipedia.org/wiki/Bastion_host
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[öffentliche IP-Adresse]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[sql-alwayson]: https://msdn.microsoft.com/library/hh510230.aspx
[sql-alwayson-force-failover]: https://msdn.microsoft.com/library/ff877957.aspx
[sql-alwayson-getting-started]: https://msdn.microsoft.com/library/gg509118.aspx
[sql-alwayson-ilb]: /azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener
[sql-alwayson-listeners]: https://msdn.microsoft.com/library/hh213417.aspx
[sql-alwayson-read-only-routing]: https://technet.microsoft.com/library/hh213417.aspx#ConnectToSecondary
[sql-keyvault]: /azure/virtual-machines/virtual-machines-windows-ps-sql-keyvault
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[wsfc-whats-new]: https://technet.microsoft.com/windows-server-docs/failover-clustering/whats-new-in-failover-clustering
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-diagram.png "n-schichtige Architektur mit Microsoft Azure"