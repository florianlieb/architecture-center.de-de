---
title: Bereitstellen von SAP NetWeaver und SAP HANA in Azure
description: "Enthält die bewährten Methoden zum Ausführen von SAP HANA in einer Hochverfügbarkeitsumgebung in Azure."
author: njray
ms.date: 06/29/2017
ms.openlocfilehash: 27a97103c0c6f305cb8e830d670c8d0ba7e22aa5
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="deploy-sap-netweaver-and-sap-hana-on-azure"></a>Bereitstellen von SAP NetWeaver und SAP HANA in Azure

Anhand dieser Referenzarchitektur werden einige bewährte Methoden für die Ausführung von SAP HANA in einer Hochverfügbarkeitsumgebung in Azure veranschaulicht. [**Stellen Sie diese Lösung bereit**.](#deploy-the-solution)

![0][0]

*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*

> [!NOTE]
> Zum Bereitstellen dieser Referenzarchitektur ist eine geeignete Lizenzierung von SAP-Produkten und anderen nicht von Microsoft stammenden Technologiekomponenten erforderlich. Informationen zur Partnerschaft zwischen Microsoft und SAP finden Sie unter [SAP HANA in Azure][sap-hana-on-azure].

## <a name="architecture"></a>Architektur

Diese Architektur besteht aus den folgenden Komponenten:

- **Virtuelles Netzwerk (VNET)**. Ein VNet ist eine Darstellung eines logisch isolierten Netzwerks in Azure. Alle VMs in dieser Referenzarchitektur werden in demselben VNet bereitgestellt. Das VNet ist in Subnetze unterteilt. Erstellen Sie ein getrenntes Subnetz für jede Ebene, z.B. Anwendung (SAP NetWeaver), Datenbank (SAP HANA), Verwaltung (Jumpbox) und Active Directory.

- **Virtuelle Computer (Virtual Machines, VMs)**: Die VMs für diese Architektur werden auf mehreren unterschiedlichen Ebenen gruppiert.

    - **SAP NetWeaver**: Enthält SAP ASCS, SAP Web Dispatcher und die SAP-Anwendungsserver. 
    
    - **SAP HANA**: In dieser Referenzarchitektur wird SAP HANA für die Datenbankebene verwendet und auf einer einzelnen [GS5][vm-sizes-mem]-Instanz ausgeführt. SAP HANA ist für OLAP-Produktionsworkloads auf GS5- oder [SAP HANA auf großen Azure-Instanzen][azure-large-instances] zertifiziert. Diese Referenzarchitektur gilt für virtuelle Azure-Computer der G- und M-Serie. Informationen zu SAP HANA auf großen Azure-Instanzen finden Sie unter [Übersicht und Architektur von SAP HANA in Azure (große Instanzen)][azure-large-instances].
   
    - **Jumpbox**: Wird auch als „geschützter Host“ bezeichnet. Dies ist eine geschützte VM im Netzwerk, die von Administratoren zum Herstellen der Verbindung mit anderen VMs verwendet wird. 
     
    - **Windows Server Active Directory-Domänencontroller**: Die Domänencontroller werden verwendet, um den Windows Server-Failovercluster zu konfigurieren (siehe unten).
 
- **Verfügbarkeitsgruppen**: Ordnen Sie die VMs für den SAP Web Dispatcher, SAP-Anwendungsserver und SAP ACSC-Rollen in separaten Verfügbarkeitsgruppen an, und stellen Sie mindestens zwei VMs für jede Rolle bereit. Auf diese Weise können die VMs für eine höhere Vereinbarung zum Servicelevel (SLA) genutzt werden.
    
- **NICs**: Für die VMs, auf denen SAP NetWeaver und SAP HANA ausgeführt werden, sind zwei Netzwerkschnittstellen (NICs) erforderlich. Jede NIC wird einem anderen Subnetz zugewiesen, um unterschiedliche Arten von Datenverkehr aufteilen zu können. Weitere Informationen finden Sie unter [Empfehlungen](#recommendations).

- **Windows Server-Failovercluster**: Die VMs, auf denen SAP ACSC ausgeführt wird, sind als Failovercluster für Hochverfügbarkeit konfiguriert. Zur Unterstützung des Failoverclusters führt SIOS DataKeeper Cluster Edition den CSV-Vorgang (Cluster Shared Volume, Freigegebenes Clustervolume) durch, indem unabhängige Datenträger repliziert werden, die sich im Besitz der Clusterknoten befinden. Ausführliche Informationen finden Sie unter [Running SAP applications on the Microsoft platform][running-sap] (Ausführen von SAP-Anwendungen auf der Microsoft-Plattform).
    
- **Lastenausgleichsmodule**: Es werden zwei [Azure Load Balancer][azure-lb]-Instanzen verwendet. Mit der ersten Instanz, die im Diagramm links dargestellt ist, wird Datenverkehr auf die SAP Web Dispatcher-VMs verteilt. Mit dieser Konfiguration wird die Option für den parallelen Web Dispatcher implementiert, die unter [Hochverfügbarkeit des SAP Web Dispatchers][sap-dispatcher-ha] beschrieben ist. Das zweite Lastenausgleichsmodul, das im Diagramm auf der rechten Seite dargestellt ist, ermöglicht das Failover im Windows Server-Failovercluster, indem eingehende Verbindungen an den aktiven bzw. fehlerfreien Knoten weitergeleitet werden.

- **VPN Gateway**: Mit dem VPN Gateway wird Ihr lokales Netzwerk auf das Azure VNet erweitert. Sie können auch eine ExpressRoute-Verbindung verwenden, bei der eine dedizierte private Verbindung genutzt wird, die nicht über das öffentliche Internet verläuft. In der Beispiellösung wird das Gateway nicht bereitgestellt. Weitere Informationen finden Sie unter [Connect an on-premises network to Azure][hybrid-networking] (Verbinden eines lokalen Netzwerks mit Azure).

## <a name="recommendations"></a>Recommendations

Ihre Anforderungen können von der hier beschriebenen Architektur abweichen. Verwenden Sie diese Empfehlungen als Ausgangspunkt.

### <a name="load-balancers"></a>Load Balancer

[SAP Web Dispatcher][sap-dispatcher] verarbeitet den Lastenausgleich für HTTP(S)-Datenverkehr von Dualstapel-Servern (ABAP und Java). SAP legt den Schwerpunkt schon seit Jahren auf Einzelstapel-Anwendungsserver, sodass sehr wenige Anwendungen heutzutage unter einem Dualstapel-Bereitstellungsmodell ausgeführt werden. Mit dem im Architekturdiagramm implementierten Azure Load Balancer wird der Hochverfügbarkeitscluster für den SAP Web Dispatcher implementiert.

Der Lastenausgleich des Datenverkehrs zu den Anwendungsservern wird intern bei SAP durchgeführt. Für Datenverkehr von SAPGUI-Clients, die eine Verbindung mit einem SAP-Server per DIAG und über Remotefunktionsaufrufe herstellen, nimmt der SCS-Nachrichtenserver den Lastenausgleich vor, indem SAP App Server-[Anmeldegruppen][logon-groups] erstellt werden. 

SMLG ist eine SAP ABAP-Transaktion zum Verwalten der Lastenausgleichsfunktion von SAP Central Services für die Anmeldung. Der Back-End-Pool der Anmeldegruppe enthält mehr als einen ABAP-Anwendungsserver. Clients, die auf ASCS-Clusterdienste zugreifen, stellen über eine Front-End-IP-Adresse eine Verbindung mit dem Azure Load Balancer her. Der Name des virtuellen Netzwerks für den ASCS-Cluster verfügt auch über eine IP-Adresse. Optional kann diese Adresse einer zusätzlichen IP-Adresse im Azure Load Balancer zugeordnet werden, damit für den Cluster die Remoteverwaltung ermöglicht wird.  

### <a name="nics"></a>NICs

Für Verwaltungsfunktionen der SAP-Landschaft ist die Aufteilung des Serverdatenverkehrs auf verschiedene NICs erforderlich. Beispielsweise sollten Geschäftsdaten von Verwaltungs- und Sicherungsdatenverkehr getrennt werden. Die Zuweisung mehrerer NICs zu unterschiedlichen Subnetzen ermöglicht diese Datentrennung. Weitere Informationen finden Sie im Abschnitt „Network“ (Netzwerk) der PDF-Datei [Building High Availability for SAP NetWeaver and SAP HANA][sap-ha] (Ermöglichen von Hochverfügbarkeit für SAP NetWeaver und SAP HANA).

Weisen Sie die NIC für die Verwaltung dem Verwaltungssubnetz und die NIC für die Datenkommunikation einem separaten Subnetz zu. Ausführliche Informationen zur Konfiguration finden Sie unter [Erstellen und Verwalten eines virtuellen Windows-Computers mit mehreren Netzwerkkarten][multiple-vm-nics].

### <a name="azure-storage"></a>Azure Storage

Wir empfehlen für alle Datenbankserver-VMs die Verwendung von Azure Storage Premium, um eine einheitliche Wartezeit für Lese- und Schreibvorgänge zu erzielen. Für SAP-Anwendungsserver, z.B. die virtuellen (A)SCS-Computer, können Sie Azure-Standardspeicher verwenden, da die Anwendung im Arbeitsspeicher ausgeführt wird und Datenträger nur für die Protokollierung verwendet werden.

Wir empfehlen die Verwendung von [Azure Managed Disks][managed-disks], um die beste Zuverlässigkeit zu erzielen. Mit Managed Disks (verwalteten Datenträgern) stellen Sie sicher, dass die Datenträger für VMs in einer Verfügbarkeitsgruppe isoliert sind, um Single Points of Failure zu vermeiden.

> [!NOTE]
> Derzeit werden für die Resource Manager-Vorlage für diese Referenzarchitektur keine verwalteten Datenträger verwendet. Es ist geplant, die Vorlage zu aktualisieren, um die Verwendung von verwalteten Datenträgern zu ermöglichen.

Zur Erreichung eines hohen Durchsatzes in Bezug auf IOPS und die Datenträger-Bandbreite gelten für das Azure-Speicherlayout die gängigen Vorgehensweisen zur Leistungsoptimierung für Speichervolumes. Wenn beispielsweise mehrere Datenträger per Striping verknüpft werden, um ein größeres Datenträgervolume zu erstellen, verbessert sich die E/A-Leistung. Eine Aktivierung des Lesecaches für Speicherinhalte, die sich in unregelmäßigen Abständen ändern, bewirkt eine höhere Geschwindigkeit beim Datenabruf. Ausführliche Informationen zu den Leistungsanforderungen finden Sie unter [SAP Note 1943937 – Hardware Configuration Check Tool][sap-1943937] (SAP-Hinweis 1943937 – Tool für die Überprüfung der Hardwarekonfiguration).

Für den Sicherungsdatenspeicher empfehlen wir die Verwendung der [kalten Speicherebene][cool-blob-storage] des Azure-Blobspeichers. Die kalte Speicherebene ist eine kostengünstige Möglichkeit zum Speichern von Daten, auf die weniger häufig zugegriffen wird und die eine lange Lebensdauer haben.

## <a name="scalability-considerations"></a>Überlegungen zur Skalierbarkeit

Auf der SAP-Anwendungsschicht sind in Azure viele verschiedene VM-Größen für das zentrale Hochskalieren verfügbar. Eine umfassende Liste finden Sie unter [SAP Note 1928533 – SAP Applications on Azure: Supported Products and Azure VM Types][sap-1928533] (SAP-Hinweis 1928533 – SAP-Anwendungen in Azure: Unterstützte Produkte und Azure-VM-Typen). Führen Sie das horizontale Hochskalieren durch, indem Sie der Verfügbarkeitsgruppe weitere VMs hinzufügen.

Für SAP HANA auf virtuellen Azure-Computern mit OLTP- und OLAP-SAP-Anwendungen ist die von SAP zertifizierte VM-Größe „GS5“ mit einer einzelnen VM-Instanz. Für größere Workloads bietet Microsoft auch [große Azure-Instanzen][azure-large-instances] für SAP HANA auf physischen Servern an, die zusammen in einem zertifizierten Microsoft Azure-Datencenter angeordnet sind. Derzeit ermöglichen diese Datencenter eine Arbeitsspeicherkapazität von bis zu 4 TB für eine einzelne Instanz. Die Konfiguration von mehreren Knoten ist auch mit einer Gesamtkapazität des Arbeitsspeichers von bis zu 32 TB möglich.

## <a name="availability-considerations"></a>Überlegungen zur Verfügbarkeit

Bei dieser verteilten Installation der SAP-Anwendung in einer zentralisierten Datenbank wird die Basisinstallation repliziert, um Hochverfügbarkeit zu erzielen. Der Entwurf für die Hochverfügbarkeit variiert für jede Ebene der Architektur:

- **Web Dispatcher**: Hochverfügbarkeit wird über redundante SAP Web Dispatcher-Instanzen mit SAP-Anwendungsdatenverkehr erzielt. Informationen hierzu finden Sie in der SAP-Dokumentation unter [SAP Web Dispatcher][swd].

- **ASCS**: Zur Sicherstellung der Hochverfügbarkeit von ASCS auf virtuellen Azure-Computern unter Windows wird das Windows Sever-Failoverclustering mit SIOS DataKeeper verwendet, um das freigegebene Clustervolume zu implementieren. Implementierungsdetails finden Sie unter [Clustering SAP ASCS on Azure][clustering] (Clustering für SAP ASCS in Azure).

- **Anwendungsserver**: Hochverfügbarkeit wird erzielt, indem für Datenverkehr in einem Pool mit Anwendungsservern ein Lastenausgleich durchgeführt wird.

- **Datenbankebene**: In dieser Referenzarchitektur wird eine einzelne SAP HANA-Datenbankinstanz bereitgestellt. Stellen Sie zur Erreichung der Hochverfügbarkeit mehr als eine Instanz bereit, und verwenden Sie die HANA-Systemreplikation (HSR), um das manuelle Failover zu implementieren. Zur Aktivierung des automatischen Failovers ist eine Hochverfügbarkeitserweiterung für die jeweilige Linux-Distribution erforderlich.

### <a name="disaster-recovery-considerations"></a>Überlegungen zur Notfallwiederherstellung

Für jede Ebene wird eine andere Strategie genutzt, um per Notfallwiederherstellung für den erforderlichen Schutz zu sorgen.

- **Anwendungsserver**: SAP-Anwendungsserver enthalten keine Geschäftsdaten. In Azure besteht eine einfache Strategie für die Notfallwiederherstellung darin, SAP-Anwendungsserver in einer anderen Region zu erstellen. Bei Konfigurationsänderungen oder Kernel-Updates auf dem primären Anwendungsserver müssen die gleichen Änderungen auf die VMs in der Region für die Notfallwiederherstellung kopiert werden. Beispielsweise werden die ausführbaren Kerneldateien auf die VMs für die Notfallwiederherstellung kopiert.

- **SAP Central Services**: Auch bei dieser Komponente des SAP-Anwendungsstapels werden keine Geschäftsdaten beibehalten. Sie können eine VM in der Region für die Notfallwiederherstellung erstellen, die zum Ausführen der SCS-Rolle bestimmt ist. Der einzige Inhalt, der vom primären SCS-Knoten synchronisiert wird, ist der Inhalt der Freigabe **/sapmnt**. Wenn Konfigurationsänderungen oder Kernel-Updates auf den primären SCS-Servern stattfinden, müssen sie für die SCS-Einheiten der Notfallwiederherstellung wiederholt werden. Verwenden Sie zum Synchronisieren der beiden Server einfach einen regulär geplanten Kopierauftrag, um **/sapmnt** an den Ort für die Notfallwiederherstellung zu kopieren. Ausführliche Informationen zum Erstellungs-, Kopier- und Testfailoverprozess erhalten Sie, indem Sie den Artikel [SAP NetWeaver: Building a Hyper-V and Microsoft Azure-based Disaster Recovery Solution][sap-netweaver-dr] (SAP NetWeaver: Erstellen einer Lösung für die Notfallwiederherstellung auf Basis von Hyper-V und Microsoft Azure) herunterladen und unter „4.3. SAP SPOF layer (ASCS)“ (4.3. SAP-SPOF-Schicht (ASCS)) nachsehen.

- **Datenbankebene**: Verwenden Sie für HANA unterstützte Replikationslösungen, z.B. HSR oder Speicherreplikation. 

## <a name="manageability-considerations"></a>Überlegungen zur Verwaltbarkeit

SAP HANA verfügt über eine Sicherungsfunktion, für die die zugrunde liegende Azure-Infrastruktur genutzt wird. Zum Sichern der SAP HANA-Datenbank, die auf virtuellen Azure-Computern ausgeführt wird, werden sowohl die SAP HANA-Momentaufnahme als auch die Azure-Speichermomentaufnahme verwendet, um für die Konsistenz von Sicherungsdateien zu sorgen. Ausführliche Informationen finden Sie unter [Sicherungsanleitung für SAP HANA in Azure Virtual Machines][hana-backup] und [Fragen zum Azure Backup-Dienst][backup-faq].

In Azure sind mehrere Funktionen für die [Überwachung und Diagnose][monitoring] der Gesamtstruktur enthalten. Außerdem wird die erweiterte Überwachung von virtuellen Azure-Computern (Linux oder Windows) von der Azure Operations Management Suite (OMS) durchgeführt.

Verwenden Sie die Azure-Erweiterung für die „Erweiterte Überwachung“ von SAP, um die SAP-basierte Überwachung von Ressourcen und Leistung von Diensten für die SAP-Infrastruktur durchzuführen. Mit dieser Erweiterung werden statistische Daten zur Azure-Überwachung in die SAP-Anwendung eingespeist, damit die Betriebssystemüberwachung und DBA Cockpit-Funktionen ermöglicht werden. 

## <a name="security-considerations"></a>Sicherheitshinweise

SAP verfügt über ein eigenes Modul für die Benutzerverwaltung (Users Management Engine, UME), um den rollenbasierten Zugriff und die Autorisierung in der SAP-Anwendung zu steuern. Ausführliche Informationen finden Sie unter [SAP HANA Security – An Overview][sap-security] (SAP HANA-Sicherheit – Eine Übersicht). (Für den Zugriff ist ein Konto für den SAP Service Marketplace erforderlich.)

Um die Sicherheit der Infrastruktur zu gewährleisten, sind die Daten während der Übertragung und im Ruhezustand geschützt. Im Abschnitt „Sicherheitshinweise“ des Artikels [Azure Virtual Machines – Planung und Implementierung für SAP NetWeaver][netweaver-on-azure] geht es um die Netzwerksicherheit. In diesem Artikel sind auch die Netzwerkports angegeben, die Sie in den Firewalls öffnen müssen, um die Anwendungskommunikation zu ermöglichen. 

Zum Verschlüsseln von IaaS-VM-Datenträgern (Windows und Linux) können Sie [Azure Disk Encryption][disk-encryption] verwenden. Azure Disk Encryption verwendet das BitLocker-Feature von Windows und das DM-Crypt-Feature von Linux, um die Volumeverschlüsselung für das Betriebssystem und die Datenträger bereitzustellen. Die Lösung funktioniert auch für Azure Key Vault, damit Sie die Verschlüsselungsschlüssel und Geheimnisse für die Datenträgerverschlüsselung in Ihrem Key Vault-Abonnement steuern und verwalten können. Daten auf den VM-Datenträgern werden in Ihrem Azure-Speicher im ruhenden Zustand verschlüsselt.

Für die SAP HANA-Verschlüsselung von ruhenden Daten empfehlen wir die Verwendung der nativen SAP HANA-Verschlüsselungstechnologie.

> [!NOTE]
> Vermeiden Sie es, die HANA-Verschlüsselung von ruhenden Daten mit Azure Disk Encryption auf demselben Server zu verwenden.

Erwägen Sie die Verwendung von [Netzwerksicherheitsgruppen][nsg] (NSGs), um den Datenverkehr zwischen den verschiedenen Subnetzen im VNet einzuschränken.

## <a name="deploy-the-solution"></a>Bereitstellen der Lösung 

Die Bereitstellungsskripts für diese Referenzarchitektur sind auf [GitHub][github] verfügbar.


### <a name="prerequisites"></a>Voraussetzungen

- Sie müssen Zugriff auf das SAP Software Download Center haben, um die Installation durchführen zu können.
 
- Installieren Sie die neueste Version von [Azure PowerShell][azure-ps]. 

- Für diese Bereitstellung sind 51 Kerne erforderlich. Stellen Sie vor dem Bereitstellen sicher, dass Ihr Abonnement über ein ausreichendes Kontingent an VM-Kernen verfügt. Verwenden Sie das Azure-Portal, um per Supportanfrage ein höheres Kontingent anzufordern, falls diese Voraussetzung nicht erfüllt ist.
 
- Für diese Bereitstellung wird eine VM der GS-Serie verwendet. Überprüfen Sie [hier][region-availability] die Verfügbarkeit der GS-Serie nach Region.

- Sie können eine Schätzung für die Kosten dieser Bereitstellung erstellen, indem Sie den [Azure-Preisrechner][azure-pricing] nutzen. 
 
Für diese Referenzarchitektur werden die folgenden VMs bereitgestellt:

| Ressourcenname | Größe des virtuellen Computers | Zweck  |
|---------------|---------|----------|
| `ra-sapApps-scs-vm1` ... `ra-sapApps-scs-vmN` | DS11v2 | SAP Central Services |
| `ra-sapApps-vm1` ... `ra-sapApps-vmN` | DS11v2 | SAP NetWeaver-Anwendung |
| `ra-sap-wdp-vm1` ... `ra-sap-wdp-vmN` | DS11v2 | SAP Web Dispatcher |
| `ra-sap-data-vm1` | GS5 | SAP HANA-Datenbankinstanz |
| `ra-sap-jumpbox-vm1` | DS1V2 | Jumpbox |

Es wird eine einzelne SAP HANA-Instanz bereitgestellt. Für die Anwendungs-VMs wird die Anzahl von bereitzustellenden Instanzen in den Vorlagenparametern angegeben.

### <a name="deploy-sap-infrastructure"></a>Bereitstellen der SAP-Infrastruktur

Sie können diese Architektur inkrementell oder auf einmal bereitstellen. Für die erste Bereitstellung empfehlen wir eine inkrementelle Vorgehensweise, damit Sie verfolgen können, was bei den einzelnen Bereitstellungsschritten passiert. Geben Sie das Inkrement an, indem Sie einen der folgenden *mode*-Parameter verwenden.

| Mode           | Funktionsbeschreibung                                                                                                            |
|----------------|-----------------------------------------------------|
| infrastructure | Stellt die Netzwerkinfrastruktur in Azure bereit.        |
| workload       | Stellt die SAP-Server im Netzwerk bereit.             |
| all            | Stellt alle vorherigen Bereitstellungen bereit.              |

Führen Sie die folgenden Schritte aus, um die Lösung bereitzustellen:

1. Laden Sie das [GitHub-Repository][github] auf Ihren lokalen Computer herunter, oder klonen Sie es.

2. Öffnen Sie ein PowerShell-Fenster, und navigieren Sie zum Ordner `/sap/sap-hana/`.

3. Führen Sie das folgende PowerShell-Cmdlet aus. Geben Sie für `<subscription id>` Ihre Azure-Abonnement-ID ein. Geben Sie für `<location>` eine Azure-Region an, z.B. `eastus` oder `westus`. Geben Sie für `<mode>` einen der oben aufgeführten Modi an.

    ```powershell
     .\Deploy-ReferenceArchitecture -SubscriptionId <subscription id> -Location <location> -ResourceGroupName <resource group> <mode>
    ```

4.  Melden Sie sich an Ihrem Azure-Konto an, wenn die Aufforderung angezeigt wird. 

Je nach ausgewähltem Modus kann es mehrere Stunden dauern, bis die Bereitstellungsskripts abgeschlossen sind.

> [!WARNING]
> Die Parameterdateien enthalten an verschiedenen Stellen ein hartcodiertes Kennwort (`AweS0me@PW`). Ändern Sie diese Werte vor der Bereitstellung.
 
### <a name="configure-sap-applications-and-database"></a>Konfigurieren der SAP-Anwendungen und der Datenbank

Führen Sie nach dem Bereitstellen der SAP-Infrastruktur wie unten angegeben auf den virtuellen Computern die Installation und Konfiguration Ihrer SAP-Anwendungen und der HANA-Datenbank durch.

> [!NOTE]
> Für die SAP-Installationsanleitung müssen Sie über einen SAP Support Portal-Benutzernamen und ein Kennwort zum Herunterladen der [SAP-Installationsleitfäden][sap-guide] verfügen.

1. Melden Sie sich an der Jumpbox (`ra-sap-jumpbox-vm1`) an. Sie verwenden die Jumpbox zum Anmelden an anderen VMs. 

2.  Melden Sie sich für jede VM mit dem Namen `ra-sap-wdp-vm1` ... `ra-sap-wdp-vmN` an der VM an, und installieren und konfigurieren Sie die SAP Web Dispatcher-Instanz gemäß den Schritten im Wiki zur [Web Dispatcher-Installation][sap-dispatcher-install].

3.  Melden Sie sich mit dem Namen `ra-sap-data-vm1` an der VM an. Installieren und konfigurieren Sie die SAP HANA-Datenbankinstanz gemäß [SAP HANA Server Installation and Update Guide][hana-guide] (SAP HANA-Server – Installations- und Updateleitfaden).

4. Melden Sie sich für jede VM mit dem Namen `ra-sapApps-scs-vm1` ... `ra-sapApps-scs-vmN` an der VM an, und installieren und konfigurieren Sie SAP Central Services (SCS) gemäß den [SAP-Installationsleitfäden][sap-guide].

5.  Melden Sie sich für jede VM mit dem Namen `ra-sapApps-vm1` ... `ra-sapApps-vmN` an der VM an, und installieren und konfigurieren Sie die SAP NetWeaver-Anwendung gemäß den [SAP-Installationsleitfäden][sap-guide].

**_Mitwirkende dieser Referenzarchitektur_** &mdash; Rick Rainey, Ross Sponholtz, Ben Trinh

[azure-large-instances]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[azure-lb]: /azure/load-balancer/load-balancer-overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[azure-ps]: /powershell/azure/overview
[backup-faq]: /azure/backup/backup-azure-backup-faq
[clustering]: https://blogs.msdn.microsoft.com/saponsqlserver/2015/05/20/clustering-sap-ascs-instance-using-windows-server-failover-cluster-on-microsoft-azure-with-sios-datakeeper-and-azure-internal-load-balancer/
[cool-blob-storage]: /azure/storage/storage-blob-storage-tiers
[disk-encryption]: /azure/security/azure-security-disk-encryption
[github]: https://github.com/mspnp/reference-architectures/tree/master/sap/sap-hana
[hana-backup]: /azure/virtual-machines/workloads/sap/sap-hana-backup-guide
[hana-guide]: https://help.sap.com/viewer/2c1988d620e04368aa4103bf26f17727/2.0.01/en-US/7eb0167eb35e4e2885415205b8383584.html
[hybrid-networking]: ../hybrid-networking/index.md
[logon-groups]: https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing
[managed-disks]: /azure/storage/storage-managed-disks-overview
[monitoring]: /azure/architecture/best-practices/monitoring
[multiple-vm-nics]: /azure/virtual-machines/windows/multiple-nics
[netweaver-on-azure]: /azure/virtual-machines/workloads/sap/planning-guide
[nsg]: /azure/virtual-network/virtual-networks-nsg
[region-availability]: https://azure.microsoft.com/regions/services/
[running-SAP]: https://blogs.msdn.microsoft.com/saponsqlserver/2016/06/07/sap-on-sql-general-update-for-customers-partners-june-2016/
[sap-1943937]: https://launchpad.support.sap.com/#/notes/1943937
[sap-1928533]: https://launchpad.support.sap.com/#/notes/1928533
[sap-dispatcher]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm
[sap-dispatcher-ha]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/9a9a6b48c673e8e10000000a42189b/frameset.htm
[sap-dispatcher-install]: https://wiki.scn.sap.com/wiki/display/SI/Web+Dispatcher+Installation
[sap-guide]: https://service.sap.com/instguides
[sap-ha]: https://support.sap.com/content/dam/SAAP/SAP_Activate/AGS_70.pdf
[sap-hana-on-azure]: https://azure.microsoft.com/services/virtual-machines/sap-hana/
[sap-netweaver-dr]: http://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx
[sap-security]: https://archive.sap.com/documents/docs/DOC-62943
[visio-download]: https://archcenter.azureedge.net/cdn/SAP-HANA-architecture.vsdx
[vm-sizes-mem]: /azure/virtual-machines/windows/sizes-memory
[swd]: https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm
[0]: ./images/sap-hana.png "SAP HANA-Architektur mit Microsoft Azure"
