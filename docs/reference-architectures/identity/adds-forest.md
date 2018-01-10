---
title: Erstellen einer AD DS-Ressourcengesamtstruktur in Azure
description: "Informationen zum Erstellen einer vertrauenswürdigen Active Directory-Domäne in Azure.\nAnleitungen, VPN-Gateway, ExpressRoute, Lastenausgleich, virtuelles Netzwerk, Active Directory"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-extend-domain
pnp.series.next: adfs
cardTitle: Create an AD DS forest in Azure
ms.openlocfilehash: b946afa91e8bd303c51f97e18be170c4105cc8c5
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/08/2017
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a>Erstellen einer Active Directory Domain Services (AD DS)-Ressourcengesamtstruktur in Azure

Diese Referenzarchitektur zeigt, wie eine separate Active Directory-Domäne in Azure erstellt wird, der Domänen in der lokalen Active Directory-Gesamtstruktur vertrauen. [**So stellen Sie diese Lösung bereit**.](#deploy-the-solution)

[![0]][0] 

*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*

In Active Directory Domain Services (AD DS) werden Identitätsinformationen in einer hierarchischen Struktur gespeichert. Der oberste Knoten in der hierarchischen Struktur wird als Gesamtstruktur bezeichnet. Eine Gesamtstruktur enthält Domänen, und Domänen enthalten andere Objekttypen. In dieser Referenzarchitektur wird eine AD DS-Gesamtstruktur in Azure mit einer unidirektionalen ausgehenden Vertrauensstellung mit einer lokalen Domäne erstellt. Die Gesamtstruktur in Azure enthält eine Domäne, die lokal nicht vorhanden ist. Aufgrund der Vertrauensstellung werden Anmeldungen bei lokalen Domänen als vertrauenswürdig für den Zugriff auf Ressourcen in der separaten Azure-Domäne angesehen. 

Zu den typischen Verwendungsmöglichkeiten dieser Architektur zählen die Verwaltung einer Sicherheitstrennung für in der Cloud gespeicherte Objekte und Identitäten sowie die Migration einzelner Domänen aus einem lokalen System in die Cloud. 

Weitere Überlegungen finden Sie unter [Auswählen einer Lösung für die Integration einer lokalen Active Directory-Instanz in Azure][considerations]. 

## <a name="architecture"></a>Architecture

Diese Architektur besteht aus den folgenden Komponenten.

* **Lokales Netzwerk**. Das lokale Netzwerk enthält eine eigene Active Directory-Gesamtstruktur und eigene Domänen.
* **Active Directory-Server**. Hierbei handelt es sich um Domänencontroller, die Domänendienste implementieren und als virtuelle Computer in der Cloud ausgeführt werden. Diese Server hosten eine Gesamtstruktur mit einer oder mehrere Domänen, die von den lokal gehosteten getrennt sind.
* **Unidirektionale Vertrauensstellung**. Das Beispiel im Diagramm zeigt eine unidirektionale Vertrauensstellung von der Domäne in Azure zur lokalen Domäne. Diese Beziehung ermöglicht lokalen Benutzern den Zugriff auf Ressourcen in der Domäne in Azure, jedoch nicht umgekehrt. Es kann eine bidirektionale Vertrauensstellung erstellt werden, wenn Cloudbenutzer auch Zugriff auf lokale Ressourcen benötigen.
* **Active Directory-Subnetz**. Die AD DS-Server werden in einem separaten Subnetz gehostet. NSG-Regeln (Netzwerksicherheitsgruppe) schützen die AD DS-Server und stellen eine Firewall für Datenverkehr von unerwarteten Quellen dar.
* **Azure-Gateway**. Das Azure-Gateway stellt eine Verbindung zwischen dem lokalen Netzwerk und dem Azure-VNet her. Dabei kann es sich um eine [VPN-Verbindung][azure-vpn-gateway] oder um [Azure ExpressRoute][azure-expressroute] handeln. Weitere Informationen finden Sie unter [Implementieren einer sicheren Hybrid-Netzwerkarchitektur in Azure][implementing-a-secure-hybrid-network-architecture].

## <a name="recommendations"></a>Empfehlungen

Spezifische Empfehlungen zur Implementierung von Active Directory in Azure finden Sie in den folgenden Artikeln:

- [Erweitern von Active Directory Domain Services (AD DS) auf Azure][adds-extend-domain] 
- [Richtlinien für die Bereitstellung von Windows Server Active Directory auf virtuellen Azure-Computern][ad-azure-guidelines]

### <a name="trust"></a>Vertrauen

Die lokalen Domänen sind in einer anderen Gesamtstruktur als die Domänen in der Cloud enthalten. Um die Authentifizierung von lokalen Benutzern in der Cloud zu ermöglichen, müssen die Domänen in Azure der Anmeldedomäne in der lokalen Gesamtstruktur vertrauen. Ebenso kann es bei Bereitstellung einer Anmeldedomäne für externe Benutzer in der Cloud erforderlich sein, dass die lokale Gesamtstruktur der Clouddomäne vertraut.

Sie können Vertrauensstellungen auf Gesamtstrukturebene durch [Erstellen von Gesamtstrukturvertrauensstellungen][creating-forest-trusts] oder auf Domänenebene durch [Erstellen externer Vertrauensstellungen][creating-external-trusts] einrichten. Eine Vertrauensstellung auf Gesamtstrukturebene erstellt eine Beziehung zwischen allen Domänen in zwei Gesamtstrukturen. Eine Vertrauensstellung auf Ebene externer Domänen erstellt nur eine Beziehung zwischen zwei angegebenen Domänen. Sie sollten Vertrauensstellungen auf Ebene externer Domänen nur zwischen Domänen in verschiedenen Gesamtstrukturen erstellen.

Vertrauensstellungen können unidirektional oder bidirektional sein:

* Eine unidirektionale Vertrauensstellung ermöglicht Benutzern in einer Domäne oder Gesamtstruktur (als *eingehende* Domäne oder Gesamtstruktur bezeichnet) Zugriff auf die Ressourcen in einer anderen Domäne oder Gesamtstruktur (als *ausgehende* Domäne oder Gesamtstruktur bezeichnet).
* Eine bidirektionale Vertrauensstellung ermöglicht Benutzern in beiden Domänen oder Gesamtstrukturen den Zugriff auf Ressourcen in der jeweils anderen Domäne oder Gesamtstruktur.

Die folgende Tabelle enthält eine Übersicht über Vertrauensstellungskonfigurationen für einige einfache Szenarien:

| Szenario | Lokale Vertrauensstellung | Cloudvertrauensstellung |
| --- | --- | --- |
| Lokale Benutzer benötigen Zugriff auf Ressourcen in der Cloud, aber nicht umgekehrt |Unidirektional, eingehend |Unidirektional, ausgehend |
| Benutzer in der Cloud benötigen Zugriff auf lokale Ressourcen, aber nicht umgekehrt |Unidirektional, ausgehend |Unidirektional, eingehend |
| Sowohl Benutzer in der Cloud als auch lokale Benutzer benötigen Zugriff auf Ressourcen in der Cloud und auf lokale Ressourcen |Bidirektional, eingehend und ausgehend |Bidirektional, eingehend und ausgehend |

## <a name="scalability-considerations"></a>Überlegungen zur Skalierbarkeit

Active Directory ist für Domänencontroller, die Teil derselben Domäne sind, automatisch skalierbar. Anforderungen werden auf alle Controller innerhalb einer Domäne verteilt. Sie können einen weiteren Domänencontroller hinzufügen, und er wird automatisch mit der Domäne synchronisiert. Konfigurieren Sie keinen separaten Lastenausgleich, um Datenverkehr an Controller innerhalb der Domäne weiterzuleiten. Stellen Sie sicher, dass alle Domänencontroller über genügend Arbeitsspeicher und Speicherressourcen für den Umgang mit der Domänendatenbank verfügen. Erstellen Sie alle Domänencontroller als virtuelle Computer derselben Größe.

## <a name="availability-considerations"></a>Überlegungen zur Verfügbarkeit

Stellen Sie mindestens zwei Domänencontroller für jede Domäne bereit. Dies ermöglicht eine automatische Replikation zwischen Servern. Erstellen Sie eine Verfügbarkeitsgruppe für die virtuellen Computer, die als Active Directory-Server für die Handhabung der einzelnen Domänen dienen. Es müssen mindestens zwei Server in dieser Verfügbarkeitsgruppe enthalten sein.

Überlegen Sie auch, ob Sie einen oder mehrere Server in jeder Domäne als [Standby-Betriebsmaster][standby-operations-masters] für den Fall festlegen, dass die Verbindung mit einem Server mit FSMO-Rolle (Flexible Single Master Operation) fehlschlägt.

## <a name="manageability-considerations"></a>Überlegungen zur Verwaltbarkeit

Informationen zu Überlegungen in Bezug auf Verwaltung und Überwachung finden Sie unter [Erweitern von Active Directory auf Azure][adds-extend-domain]. 
 
Weitere Informationen finden Sie unter [Überwachen von Active Directory][monitoring_ad]. Sie können Tools wie z.B. [Microsoft Systems Center][microsoft_systems_center] auf einem Überwachungsserver im Verwaltungssubnetz installieren, um diese Aufgaben auszuführen.

## <a name="security-considerations"></a>Sicherheitshinweise

Vertrauensstellungen auf Gesamtstrukturebene sind transitiv. Wenn Sie eine Vertrauensstellung auf Gesamtstrukturebene zwischen einer lokalen Gesamtstruktur und einer Gesamtstruktur in der Cloud erstellen, wird diese Vertrauensstellung auf weitere neue Domänen ausgeweitet, die in beiden Gesamtstrukturen erstellt werden. Wenn Sie Domänen verwenden, um eine Trennung aus Sicherheitsgründen bereitzustellen, sollten Sie Vertrauensstellungen nur auf Domänenebene erstellen. Vertrauensstellungen auf Domäneneben sind nicht transitiv.

Spezifische Überlegungen zur Sicherheit für Active Directory finden Sie im Abschnitt „Sicherheitshinweise“ unter [Erweitern von Active Directory auf Azure][adds-extend-domain].

## <a name="deploy-the-solution"></a>Bereitstellen der Lösung

Eine Lösung zur Bereitstellung für diese Referenzarchitektur ist auf [GitHub][github] verfügbar. Sie benötigen die neueste Version der Azure-Befehlszeilenschnittstelle, um das PowerShell-Skript auszuführen, mit dem die Lösung bereitgestellt wird. Um die Referenzarchitektur bereitzustellen, gehen Sie folgendermaßen vor:

1. Laden Sie den Projektmappenordner von [GitHub][github] auf Ihren lokalen Computer herunter, oder klonen Sie ihn.

2. Öffnen Sie die Azure-Befehlszeilenschnittstelle, und navigieren Sie zum lokalen Projektmappenordner.

3. Führen Sie den folgenden Befehl aus:
   
    ```Powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    Ersetzen Sie `<subscription id>` durch Ihre Azure-Abonnement-ID.
   
    Geben Sie für `<location>` eine Azure-Region an, z.B. `eastus` oder `westus`.
   
    Der `<mode>`-Parameter steuert die Granularität der Bereitstellung. Er kann einen der folgenden Werte annehmen:
   
   * `Onpremise`: stellt die simulierte lokale Umgebung bereit
   * `Infrastructure`: stellt die VNET-Infrastruktur und die Jumpbox in Azure bereit
   * `CreateVpn`: stellt das Gateway des virtuellen Azure-Netzwerks bereit und verbindet es mit dem simulierten lokalen Netzwerk.
   * `AzureADDS`: stellt die virtuellen Computer, die als Active Directory DS-Server fungieren, auf diesen virtuellen Computern Active Directory und dann die Domäne in Azure bereit.
   * `WebTier`: stellt die virtuellen Computer der Webebene und den Lastenausgleich bereit.
   * `Prepare`: stellt alle vorherigen Bereitstellungen bereit. **Dies ist die empfohlene Option, wenn Sie nicht über ein lokales Netzwerk verfügen, aber die vollständige oben beschriebene Referenzarchitektur für Tests oder Prüfverfahren bereitstellen möchten.** 
   * `Workload`: stellt die virtuellen Computer der Geschäfts- und Datenebene sowie Lastenausgleichsmodule bereit. Beachten Sie, dass diese virtuellen Computer nicht in der Bereitstellung `Prepare` enthalten sind.

4. Warten Sie, bis die Bereitstellung abgeschlossen ist. Das Bereitstellen der Bereitstellung `Prepare` dauert mehrere Stunden.
     
5. Wenn Sie die simulierte lokale Konfiguration verwenden, konfigurieren Sie die eingehende Vertrauensstellung:
   
   1. Stellen Sie eine Verbindung mit der Jumpbox her (*ra-adtrust-mgmt-vm1* in der Ressourcengruppe *ra-adtrust-security-rg*). Melden Sie sich als *testuser* mit dem Kennwort *AweS0me@PW* an.
   2. Öffnen Sie in der Jumpbox eine RDP-Sitzung auf dem ersten virtuellen Computer in der Domäne *contoso.com* (die lokale Domäne). Dieser virtuelle Computer hat die IP-Adresse 192.168.0.4. Der Benutzername ist *contoso\testuser* mit dem Kennwort *AweS0me@PW*.
   3. Laden Sie das Skript [incoming-trust.ps1][incoming-trust] herunter, und führen Sie es aus, um die eingehende Vertrauensstellung von der Domäne *treyresearch.com* zu erstellen.

6. Wenn Sie Ihre eigene lokale Infrastruktur verwenden, gehen Sie folgendermaßen vor:
   
   1. Laden Sie das Skript [incoming-trust.ps1][incoming-trust] herunter.
   2. Bearbeiten Sie das Skript, und ersetzen Sie den Wert der Variablen `$TrustedDomainName` durch den Namen Ihrer eigenen Domäne.
   3. Führen Sie das Skript aus.

7. Stellen Sie von der Jumpbox eine Verbindung mit dem ersten virtuellen Computer in der Domäne *treyresearch.com* (die Domäne in der Cloud) her. Dieser virtuelle Computer hat die IP-Adresse 10.0.4.4. Der Benutzername ist *treyresearch\testuser* mit dem Kennwort *AweS0me@PW*.

8. Laden Sie das Skript [outgoing-trust.ps1][outgoing-trust] herunter, und führen Sie es aus, um die eingehende Vertrauensstellung von der Domäne *treyresearch.com* zu erstellen. Wenn Sie Ihre eigenen lokalen Computer verwenden, bearbeiten Sie zuerst das Skript. Legen Sie die Variable `$TrustedDomainName` auf den Namen Ihrer lokalen Domäne fest, und geben Sie die IP-Adressen der Active Directory DS-Server für diese Domäne in der Variablen `$TrustedDomainDnsIpAddresses` an.

9. Warten Sie einige Minuten, bis die vorherigen Schritte abgeschlossen sind, und stellen Sie dann eine Verbindung mit einem lokalen virtuellen Computer her. Führen Sie die im Artikel [Überprüfen einer Vertrauensstellung][verify-a-trust] beschriebenen Schritte aus, um festzustellen, ob die Vertrauensstellung zwischen den Domänen *contoso.com* und *treyresearch.com* richtig konfiguriert ist.

## <a name="next-steps"></a>Nächste Schritte

* Informieren Sie sich über bewährte Methoden für das [Erweitern Ihrer lokalen AD DS-Domäne auf Azure][adds-extend-domain].
* Informieren Sie sich über bewährte Methoden für das [Erstellen einer AD FS-Infrastruktur][adfs] in Azure.

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[considerations]: ./considerations.md
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx
[0]: ./images/adds-forest.png "Schützen einer Hybrid-Netzwerkarchitektur mit separaten Active Directory-Domänen"