---
title: "Ausführen eines virtuellen Linux-Computers in Azure"
description: "Hier erfahren Sie, wie Sie eine Linux-VM unter Azure ausführen und die Skalierbarkeit, Resilienz, Verwaltbarkeit und Sicherheit im Blick behalten."
author: telmosampaio
ms.date: 12/12/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: a51e0d7ed4e35c5331241cf78d1715e63f9b4d86
ms.sourcegitcommit: 1c0465cea4ceb9ba9bb5e8f1a8a04d3ba2fa5acd
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/02/2018
---
# <a name="run-a-linux-vm-on-azure"></a>Ausführen eines virtuellen Linux-Computers in Azure

Anhand dieser Referenzarchitektur werden einige bewährte Methoden für die Ausführung virtueller Linux-Computer in Azure veranschaulicht. Sie enthält Empfehlungen für die Bereitstellung des virtuellen Computers sowie der Netzwerk-und Speicherkomponenten. Diese Architektur kann verwendet werden, um eine einzelne VM-Instanz auszuführen. Sie dient als Grundlage für komplexere Architekturen wie N-schichtige Anwendungen. [**Stellen Sie diese Lösung bereit.**](#deploy-the-solution)

![[0]][0]

*Laden Sie eine [Visio-Datei][visio-download] herunter, die dieses Architekturdiagramm enthält.*

## <a name="architecture"></a>Architecture

Für die Bereitstellung eines virtuellen Azure-Computers sind zusätzliche Komponenten wie Compute-, Netzwerk- und Speicherressourcen erforderlich.

* **Ressourcengruppe:** Eine [Ressourcengruppe][resource-manager-overview] ist ein Container, der verwandte Ressourcen enthält. Im Allgemeinen sollten Sie Ressourcen in einer Lösung gruppieren, die auf deren Lebensdauer und der Person basiert, von der diese Ressourcen verwaltet werden. Bei einer einzelnen VM-Workload können Sie eine einzige Ressourcengruppe für alle Ressourcen erstellen.
* **VM**. Sie können eine VM über eine Liste mit veröffentlichten Images oder über ein benutzerdefiniertes verwaltetes Image oder eine virtuelle Festplattendatei (VHD) bereitstellen, die in Azure Blob Storage hochgeladen wurde. Azure unterstützt die Ausführung verschiedener beliebter Linux-Distributionen, z.B. CentOS, Debian, Red Hat Enterprise, Ubuntu und FreeBSD. Weitere Informationen finden Sie unter [Azure und Linux][azure-linux].
* **Betriebssystem-Datenträger:** Der Datenträger für das Betriebssystem ist eine in [Azure Storage][azure-storage] gespeicherte virtuelle Festplatte, sodass diese weiterhin bestehen bleibt, auch wenn der Hostcomputer ausgefallen ist. Für Linux-VMs ist `/dev/sda1` der Datenträger für das Betriebssystem.
* **Temporärer Datenträger** Die VM wird mit einem temporären Datenträger erstellt. Dieser Datenträger wird auf dem Hostcomputer auf einem physischen Laufwerk gespeichert. Er wird **nicht** in Azure Storage gespeichert und kann bei Neustarts und anderen Ereignissen während des VM-Lebenszyklus gelöscht werden. Verwenden Sie diesen Datenträger nur für temporäre Daten, z. B. Auslagerungsdateien. Für Linux-VMs ist der temporäre Datenträger `/dev/sdb1` und wird in `/mnt/resource` oder `/mnt` bereitgestellt.
* **Datenträger** Bei einem [Datenträger für Daten][data-disk] handelt es sich um eine persistente VHD, die für Anwendungsdaten verwendet wird. Datenträger werden in Azure Storage gespeichert, z.B. auf dem Betriebssystem-Datenträger.
* **Virtuelles Netzwerk (VNet) und Subnetz:** Jeder virtuelle Azure-Computer wird in einem virtuellen Netzwerk (VNet) bereitgestellt, das in mehrere Subnetze segmentiert werden kann.
* **Öffentliche IP-Adresse** Eine öffentliche IP-Adresse wird für die Kommunikation mit der VM benötigt, z.B. über SSH.
* **Azure DNS:** [Azure DNS][azure-dns] ist ein Hostingdienst für DNS-Domänen, der die Namensauflösung unter Verwendung der Microsoft Azure-Infrastruktur durchführt. Durch das Hosten Ihrer Domänen in Azure können Sie Ihre DNS-Einträge mithilfe der gleichen Anmeldeinformationen, APIs, Tools und Abrechnung wie für die anderen Azure-Dienste verwalten.  
* **Netzwerkschnittstelle (NIC)** Eine zugewiesene NIC ermöglicht der VM die Kommunikation mit dem virtuellen Netzwerk.
* **Netzwerksicherheitsgruppen (NSG)** [Netzwerksicherheitsgruppen][nsg] dienen zum Zulassen oder Verweigern von Netzwerkdatenverkehr für eine Netzwerkressource. Sie können eine NSG einer einzelnen NIC oder einem Subnetz zuordnen. Wenn Sie sie einem Subnetz zuordnen, gelten die NSG-Regeln für alle VMs in diesem Subnetz.
* **Diagnose:** Diagnoseprotokolle sind für die Verwaltung und Problembehandlung des virtuellen Computers von entscheidender Bedeutung.

## <a name="recommendations"></a>Empfehlungen

Diese Architektur veranschaulicht grundlegende Empfehlungen für die Ausführung einer Linux-VM in Azure. Wir raten jedoch davon ab, für unternehmenskritische Workloads nur eine VM zu verwenden, da so eine einzelne Fehlerquelle („Single Point of Failure“) entsteht. Stellen Sie mehrere VMs in einer [Verfügbarkeitsgruppe][availability-set] bereit, um eine höhere Verfügbarkeit zu erzielen. Weitere Informationen finden Sie unter [Ausführen mehrerer VMs in Azure][multi-vm]. 

### <a name="vm-recommendations"></a>Empfehlungen für virtuelle Computer

Azure bietet viele verschiedene Größen von virtuellen Computern. [Storage Premium][premium-storage] wird aufgrund seiner hohen Leistung und niedrigen Wartezeit empfohlen und wird von [bestimmten VM-Größen][premium-storage-supported] unterstützt. Wählen Sie eine dieser Größen aus, falls bei Ihnen nicht eine spezielle Workload erforderlich ist, z. B. High Performance Computing. Weitere Informationen finden Sie unter [Größen virtueller Computer][virtual-machine-sizes].

Starten Sie beim Verschieben einer vorhandenen Workload in Azure mit der VM-Größe, die Ihren lokalen Servern am ehesten entspricht. Messen Sie dann die Leistung Ihrer tatsächlichen Workload hinsichtlich CPU, Arbeitsspeicher und Datenträger-IOPS (E/A-Vorgänge pro Sekunde), und passen Sie die Größe nach Bedarf an. Wenn Sie mehrere Netzwerkschnittstellenkarten für Ihre VM benötigen, sollten Sie sich darüber im Klaren sein, dass für jede [VM-Größe][vm-size-tables] eine maximale Anzahl von Netzwerkschnittstellenkarten definiert ist.

Wenn Sie Azure-Ressourcen bereitstellen, müssen Sie eine Region angeben. Es ist im Allgemeinen ratsam, eine Region zu wählen, der sich in der Nähe Ihrer internen Benutzer oder Ihrer Kunden befindet. Es sind jedoch nicht alle VM-Größen in allen Regionen verfügbar. Weitere Informationen finden Sie unter [Dienste nach Region][services-by-region]. Führen Sie den folgenden Befehl über die Azure-Befehlszeilenschnittstelle (CLI) aus, um eine Liste mit den in einer bestimmten Region verfügbaren VM-Größen anzuzeigen:

```
az vm list-sizes --location <location>
```

Informationen zum Auswählen eines veröffentlichten VM-Image finden Sie unter [Suchen nach Linux-VM-Images][select-vm-image].

Aktivieren Sie die Überwachung und Diagnose, einschließlich grundlegender Integritätsmetriken, Infrastrukturprotokolle zur Diagnose sowie [Startdiagnose][boot-diagnostics]. Startdiagnosen dienen dazu, einen Fehler beim Startvorgang zu untersuchen, wenn Ihre VM einen nicht startfähigen Zustand hat. Weitere Informationen finden Sie unter [Aktivieren von Überwachung und Diagnose][enable-monitoring].  

### <a name="disk-and-storage-recommendations"></a>Empfehlungen für Datenträger und Speicher

Zum Erzielen einer optimalen E/A-Leistung empfehlen wir [Storage Premium][premium-storage] zum Speichern von Daten auf SSDs (Solid State Drives). Die Kosten basieren auf der Kapazität des bereitgestellten Datenträgers. IOPS und Durchsatz (also die Datenübertragungsrate) richten sich ebenfalls nach der Datenträgergröße. Berücksichtigen Sie beim Bereitstellen eines Datenträgers also alle drei Faktoren (Kapazität, IOPS und Durchsatz). 

Wir empfehlen auch die Verwendung von [verwalteten Datenträgern](/azure/storage/storage-managed-disks-overview). Bei verwalteten Datenträgern ist kein Speicherkonto erforderlich. Sie geben einfach die Größe und den Typ des Datenträgers an, und dieser wird als Ressource mit Hochverfügbarkeit bereitgestellt.

Wenn Sie nicht verwaltete Datenträger verwenden, erstellen Sie separate Azure Storage-Konten für jeden virtuellen Computer, um die virtuellen Festplatten (VHDs) zu speichern und das Überschreiten der [IOPS-Grenzwerte][vm-disk-limits] für Speicherkonten zu vermeiden.

Fügen Sie einen oder mehrere Datenträger hinzu. Wenn Sie eine virtuelle Festplatte (VHD) erstellen, ist sie nicht formatiert. Melden Sie sich an der VM an, um den Datenträger zu formatieren. Wenn Sie keine verwalteten Datenträger verwenden und über eine große Zahl von Datenträgern verfügen, sollten Sie sich der E/A-Grenzwerte des Speicherkontos bewusst sein. Weitere Informationen finden Sie unter [Grenzwerte für Datenträger virtueller Computer][vm-disk-limits].

In der Linux-Shell werden Datenträger für Daten als `/dev/sdc`, `/dev/sdd` usw. angezeigt. Sie können `lsblk` ausführen, um die Blockgeräte einschließlich der Datenträger aufzulisten. Um einen Datenträger für Daten zu verwenden, erstellen Sie eine Partition und ein Dateisystem und binden den Datenträger ein. Beispiel: 

```bat
# Create a partition.
sudo fdisk /dev/sdc     # Enter 'n' to partition, 'w' to write the change.

# Create a file system.
sudo mkfs -t ext3 /dev/sdc1

# Mount the drive.
sudo mkdir /data1
sudo mount /dev/sdc1 /data1
```

Wenn Sie einen Datenträger für Daten hinzufügen, wird dem Datenträger eine logische Einheitennummer (Logical Unit Number, LUN) zugewiesen. Optional können Sie die LUN-ID angeben, z.B. wenn Sie einen Datenträger austauschen und dieselbe LUN-ID beibehalten möchten oder über eine Anwendung verfügen, die nach einer bestimmten LUN-ID sucht. Beachten Sie jedoch, dass die LUN-IDs für jeden Datenträger eindeutig sein muss.

Es kann ratsam sein, den E/A-Scheduler zu ändern, um die Leistung auf SSDs zu optimieren. Bei den Datenträgern für VMs mit Storage Premium-Konten handelt es sich nämlich um SSDs. Eine übliche Empfehlung ist die Verwendung des NOOP-Schedulers für SSDs. Sie sollten aber ein Tool wie [iostat] zur Überwachung der E/A-Datenträgerleistung für Ihre Workload einsetzen.

Erstellen Sie ein separates Speicherkonto zum Speichern der Diagnoseprotokolle, um die Leistung zu maximieren. Ein standardmäßiger lokal redundanter Speicher (LRS) reicht für Diagnoseprotokolle aus.

### <a name="network-recommendations"></a>Netzwerkempfehlungen

Die öffentliche IP-Adresse kann dynamisch oder statisch sein. Die Standardeinstellung ist „Dynamisch“.

* Reservieren Sie eine [statische IP-Adresse][static-ip], falls Sie eine statische IP-Adresse benötigen, die sich nicht ändert – z.B. wenn Sie einen A-Eintrag in DNS erstellen oder die IP-Adresse auf eine Liste mit sicheren Adressen setzen müssen.
* Sie können auch einen vollständig qualifizierten Domänennamen (FQDN) für die IP-Adresse erstellen. Sie können anschließend einen [CNAME-Eintrag][cname-record] in DNS registrieren, der auf den FQDN verweist. Weitere Informationen finden Sie unter [Erstellen eines vollqualifizierten Domänennamens im Azure-Portal][fqdn]. Sie können [Azure DNS][azure-dns] oder einen anderen DNS-Dienst verwenden.

Alle Netzwerksicherheitsgruppen enthalten eine Reihe von [Standardregeln][nsg-default-rules], einschließlich einer Regel, die den gesamten eingehenden Internetverkehr blockiert. Die Standardregeln können nicht gelöscht, aber von anderen Regeln überschrieben werden. Um Internetdatenverkehr zu ermöglichen, erstellen Sie Regeln, die eingehenden Datenverkehr an bestimmten Ports zulassen, z.B. Port 80 für HTTP.

Fügen Sie zum Aktivieren von SSH eine NSG-Regel hinzu, die eingehenden Datenverkehr am TCP-Port 22 zulässt.

## <a name="scalability-considerations"></a>Überlegungen zur Skalierbarkeit

Sie können eine VM zentral hoch- oder herunterskalieren, indem Sie die [VM-Größe ändern][vm-resize]. Um horizontal hochzuskalieren, platzieren Sie zwei oder mehr VMs hinter einem Lastenausgleich. Weitere Informationen finden Sie unter [Ausführen von VMs mit Lastenausgleich zur Steigerung von Skalierbarkeit und Verfügbarkeit][multi-vm].

## <a name="availability-considerations"></a>Überlegungen zur Verfügbarkeit

Stellen Sie mehrere VMs in einer Verfügbarkeitsgruppe bereit, um eine höhere Verfügbarkeit zu erzielen. Dies führt auch zu einer höheren [Vereinbarung zum Servicelevel (Service Level Agreement, SLA)][vm-sla].

Ihr virtueller Computer kann von einer [geplanten Wartung][planned-maintenance] oder [ungeplanten Wartung][manage-vm-availability] betroffen sein. Sie können [VM-Neustartprotokolle][reboot-logs] verwenden, um zu ermitteln, ob ein VM-Neustart durch einen geplanten Wartungsvorgang verursacht wurde.

Virtuelle Festplatten (VHDs) werden im [Azure-Speicher][azure-storage] gespeichert. Der Azure-Speicher wird repliziert, um Dauerhaftigkeit und Verfügbarkeit sicherzustellen.

Als Schutz vor versehentlichen Datenverlusten während des normalen Betriebs (z.B. aufgrund eines Benutzerfehlers) sollten Sie auch Point-in-Time-Sicherungen implementieren, indem Sie [Blobmomentaufnahmen][blob-snapshot] oder ein anderes Tool verwenden.

## <a name="manageability-considerations"></a>Überlegungen zur Verwaltbarkeit

**Ressourcengruppen.** Legen Sie eng miteinander verknüpfte Ressourcen mit demselben Lebenszyklus in derselben [Ressourcengruppe][resource-manager-overview] ab. Mithilfe von Ressourcengruppen können Sie Ressourcen als Gruppe bereitstellen und überwachen und Abrechnungskosten nach Ressourcengruppe verfolgen. Sie können auch Ressourcen als Gruppe löschen, was für Testbereitstellungen sehr nützlich ist. Vergeben Sie aussagekräftige Ressourcennamen, um das Auffinden einer bestimmten Ressource und das Erfassen ihrer Rolle zu vereinfachen. Weitere Informationen finden Sie unter [Empfohlene Namenskonventionen für Azure-Ressourcen][naming-conventions].

**SSH**. Bevor Sie eine Linux-VM erstellen, wird ein 2048-Bit-RSA-Paar aus privatem und öffentlichem Schlüssel generiert. Verwenden Sie die öffentliche Schlüsseldatei bei der Erstellung der VM. Weitere Informationen finden Sie unter [Verwenden von SSH mit Linux und Mac in Azure][ssh-linux].

**Beenden einer VM.** Unter Azure wird zwischen den Zuständen „Stopped“ (Beendet) und „Deallocated“ (Zuordnung aufgehoben) unterschieden. Ihnen werden nur Gebühren berechnet, wenn der VM-Status angehalten wird, aber nicht, wenn die Zuordnung für den virtuellen Computer aufgehoben wurde.

Sie können die Zuordnung des virtuellen Computers auch mit der Schaltfläche **Beenden** im Azure-Portal aufheben. Wenn das Herunterfahren über das Betriebssystem erfolgt, während Sie angemeldet sind, wird der virtuelle Computer zwar beendet, aber die Zuordnung wird **nicht** aufgehoben. Es fallen also weiter Kosten an.

**Löschen einer VM.** Wenn Sie eine VM löschen, werden die VHDs nicht gelöscht. Dies bedeutet, dass Sie die VM problemlos löschen können, ohne dass Daten verloren gehen. Allerdings wird Ihnen der Speicherplatz weiter in Rechnung gestellt. Um die VHD zu löschen, löschen Sie die Datei aus dem [Blobspeicher][blob-storage].

Zur Verhinderung des versehentlichen Löschens verwenden Sie eine [Ressourcensperre][resource-lock], um die gesamte Ressourcengruppe oder einzelne Ressourcen, z. B. einen virtuellen Computer, zu sperren.

## <a name="security-considerations"></a>Sicherheitshinweise

Verwenden Sie [Azure Security Center][security-center], um sich eine zentrale Übersicht über den Sicherheitszustand Ihrer Azure-Ressourcen zu verschaffen. Mit Security Center werden potenzielle Sicherheitsprobleme überwacht, und Sie erhalten eine umfassende Darstellung des Sicherheitszustands Ihrer Bereitstellung. Security Center wird für jedes Azure-Abonnement individuell konfiguriert. Aktivieren Sie die Sammlung von Sicherheitsdaten wie im [Schnellstarthandbuch zu Azure Security Center][security-center-get-started] beschrieben. Nachdem die Datensammlung aktiviert wurde, durchsucht Security Center VMs automatisch, die unter diesem Abonnement erstellt werden.

**Patchverwaltung.** Falls aktiviert, prüft Security Center, ob kritische und Sicherheitsupdates fehlen. 

**Antimalware.** Falls aktiviert, prüft Security Center, ob Antischadsoftware installiert ist. Sie können auch das Security Center nutzen, um Antischadsoftware über das Azure-Portal zu installieren.

**Vorgänge.** Arbeiten Sie mit der [rollenbasierten Zugriffssteuerung][rbac] (Role-Based Access Control, RBAC), um den Zugriff auf die von Ihnen bereitgestellten Azure-Ressourcen zu steuern. Mithilfe der RBAC können Sie Mitglieder Ihres DevOps-Teams Autorisierungsrollen zuweisen. Die Rolle „Leser“ kann beispielsweise Azure-Ressourcen anzeigen, diese aber nicht erstellen, verwalten oder löschen. Einige Rollen sind für bestimmte Azure-Ressourcentypen spezifisch. Die VM-Rolle „Mitwirkender“ kann z. B. eine VM neu starten oder Ihre Zuordnung aufheben, das Administratorkennwort zurücksetzen, eine neue VM erstellen usw. Andere [integrierte RBAC-Rollen][rbac-roles], die für diese Architektur nützlich sein können, sind u. a. [DevTest Labs-Benutzer][rbac-devtest] und [Netzwerkmitwirkender][rbac-network]. Ein Benutzer kann mehreren Rollen zugewiesen werden. Außerdem können Sie für noch präzisere Berechtigungen benutzerdefinierte Rollen erstellen.

> [!NOTE]
> Die RBAC schränkt nicht die Aktionen eines Benutzers ein, der bei einer VM angemeldet ist. Diese Berechtigungen werden vom Kontotyp im Gastbetriebssystem bestimmt.   

Verwenden Sie [Überwachungsprotokolle][audit-logs], um Bereitstellungsaktionen und andere VM-Ereignisse anzuzeigen.

**Datenverschlüsselung.** Ziehen Sie [Azure Disk Encryption][disk-encryption] in Betracht, wenn Sie die Datenträger für Betriebssystem und Daten verschlüsseln müssen. 

## <a name="deploy-the-solution"></a>Bereitstellen der Lösung

Eine Bereitstellung für diese Architektur ist auf [GitHub][github-folder] verfügbar. Folgendes wird bereitgestellt:

  * Ein virtuelles Netzwerk mit einzelnem Subnetz namens **web**, auf dem die VM gehostet wird
  * Eine NSG mit zwei Regeln für den eingehenden Datenverkehr, die SSH- und HTTP-Datenverkehr an die VM zulassen
  * Ein virtueller Computer mit der neuesten Version von Ubuntu (16.04.3 LTS)
  * Eine benutzerdefinierte Beispielskripterweiterung, die die beiden Datenträger formatiert und Apache HTTP Server auf der Ubuntu-VM bereitstellt.

### <a name="prerequisites"></a>Voraussetzungen

Bevor Sie die Referenzarchitektur in Ihrem eigenen Abonnement bereitstellen können, müssen Sie die folgenden Schritte ausführen.

1. Klonen oder Forken Sie das GitHub-Repository [AzureCAT-Referenzarchitekturen][ref-arch-repo], oder laden Sie die zugehörige ZIP-Datei herunter.

2. Vergewissern Sie sich, dass Azure CLI 2.0 auf Ihrem Computer installiert ist. Anweisungen zur CLI-Installation finden Sie unter [Installieren von Azure-CLI 2.0][azure-cli-2].

3. Installieren Sie das npm-Paket mit den [Azure Bausteinen][azbb].

4. Melden Sie sich über eine Eingabeaufforderung, eine bash-Eingabeaufforderung oder die PowerShell-Eingabeaufforderung bei Ihrem Azure-Konto an. Verwenden Sie dazu die unten aufgeführten Befehle, und befolgen Sie die Anweisungen.

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>Bereitstellen der Lösung mit azbb

Gehen Sie zum Bereitstellen der einzelnen VM-Beispielworkloads folgendermaßen vor:

1. Navigieren Sie zum Ordner `virtual-machines\single-vm\parameters\linux` für das Repository, das Sie im vorherigen Schritt „Voraussetzungen“ heruntergeladen haben.

2. Öffnen Sie die Datei `single-vm-v2.json`, geben Sie einen Benutzernamen und den öffentlichen SSH-Schlüssel wie unten dargestellt zwischen den Anführungszeichen an, und speichern Sie die Datei.

  ```bash
  "adminUsername": "",
  "sshPublicKey": "",
  ```

3. Führen Sie `azbb` aus, um die Beispiel-VM wie unten dargestellt bereitzustellen.

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

Weitere Informationen zum Bereitstellen dieser Beispielreferenzarchitektur finden Sie in unserem [GitHub-Repository][git].

## <a name="next-steps"></a>Nächste Schritte

- Informieren Sie sich über unsere [Azure-Bausteine][azbbv2].
- Stellen Sie [mehrere VMs][multi-vm] in Azure bereit.

<!-- links -->
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-linux]: /azure/virtual-machines/virtual-machines-linux-azure-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-linux-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[azure-dns]: /azure/dns/dns-overview
[fqdn]: /azure/virtual-machines/virtual-machines-linux-portal-create-fqdn
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[iostat]: https://en.wikipedia.org/wiki/Iostat
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[multi-vm]: multi-vm.md
[naming-conventions]: /azure/architecture/best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-linux-planned-maintenance
[premium-storage]: /azure/virtual-machines/linux/premium-storage
[premium-storage-supported]: /azure/virtual-machines/linux/premium-storage#supported-vms
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/security-center-intro
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-linux-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssh-linux]: /azure/virtual-machines/virtual-machines-linux-mac-create-ssh-keys
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-linux-sizes
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-linux-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[0]: ./images/single-vm-diagram.png "Architektur einer einzelnen Linux-VM in Azure"
