---
title: Implementieren einer DMZ zwischen Azure und dem Internet
description: Erfahren Sie, wie Sie eine sichere Hybrid-Netzwerkarchitektur mit Internetzugriff in Azure implementieren.
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: c88545b1fcae49b413e7e2b6ac5bd92d3fd3456d
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/30/2018
---
# <a name="dmz-between-azure-and-the-internet"></a>DMZ zwischen Azure und dem Internet

Diese Referenzarchitektur zeigt ein sicheres Hybridnetzwerk, das ein lokales Netzwerk in Azure erweitert und auch Internetdatenverkehr zulässt. 

[![0]][0] 

*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*

Diese Referenzarchitektur erweitert die Architektur, die in [Implementieren einer DMZ zwischen Azure und Ihrem lokalen Rechenzentrum][implementing-a-secure-hybrid-network-architecture] beschrieben wird. Dabei wird der privaten DMZ, die den Datenverkehr aus dem lokalen Netzwerk verwaltet, eine öffentliche DMZ für den Internetdatenverkehr hinzugefügt. 

Typische Einsatzmöglichkeiten für diese Architektur sind:

* Hybridanwendungen, in denen Workloads teilweise lokal und teilweise in Azure ausgeführt werden.
* Azure-Infrastruktur, die eingehenden Datenverkehr von lokalen Quellen und dem Internet weiterleitet

## <a name="architecture"></a>Architecture

Die Architektur umfasst die folgenden Komponenten.

* **Öffentliche IP-Adresse:** Die IP-Adresse des öffentlichen Endpunkts. Externe Benutzer, die mit dem Internet verbunden sind, können über diese Adresse auf das System zugreifen.
* **Virtuelles Netzwerkgerät:** Diese Architektur umfasst einen separaten Pool von virtuellen Netzwerkgeräten (Network Virtual Appliance, NVA) für Datenverkehr aus dem Internet.
* **Azure Load Balancer:** Alle aus dem Internet eingehende Anforderungen werden über den Lastenausgleich auf die NVAs in der öffentlichen DMZ verteilt.
* **Öffentliche DMZ – eingehendes Subnetz:** Dieses Subnetz akzeptiert Anforderungen vom Azure Load Balancer. Eingehende Anforderungen werden an eine der NVAs in der öffentlichen DMZ weitergeleitet.
* **Öffentliche DMZ – ausgehendes Subnetz:** Anforderungen, die von der NVA genehmigt wurden, werden über dieses Subnetz an den internen Lastenausgleich für die Webebene weitergeleitet.

## <a name="recommendations"></a>Empfehlungen

Die folgenden Empfehlungen gelten für die meisten Szenarios. Sofern Sie keine besonderen Anforderungen haben, die Vorrang haben, sollten Sie diese Empfehlungen befolgen. 

### <a name="nva-recommendations"></a>Empfehlungen für virtuelle Netzwerkgeräte

Verwenden Sie einen Satz von NVAs für Datenverkehr aus dem Internet und einen anderen für Datenverkehr mit lokalem Ursprung. Wenn Sie für beides nur einen Satz von NVAs verwenden, stellt dies ein Sicherheitsrisiko dar, da kein wirksamer Sicherheitsbereich zwischen den beiden Arten von Netzwerkdatenverkehr aufgebaut wird. Separate NVAs erleichtern das Überprüfen von Sicherheitsregeln und verdeutlichen, welche Regeln der eingehenden Netzwerkanforderung entsprechen. Einen Satz von NVAs implementiert nur Regeln für den Internetdatenverkehr, während ein anderer Satz von NVAs nur Regeln für den lokalen Datenverkehr implementiert.

Schließen Sie eine Ebene-7-NVA ein, um Anwendungsverbindungen auf NVA-Ebene zu beenden und Kompatibilität zwischen den Back-End-Ebenen sicherzustellen. Dies garantiert symmetrischen Konnektivität, bei der Antwortdatenverkehr von den Back-End-Ebenen über die NVA zurückgegeben wird.  

### <a name="public-load-balancer-recommendations"></a>Empfehlungen für den öffentlichen Lastenausgleich

Für mehr Skalierbarkeit und Verfügbarkeit stellen Sie die NVAs der öffentlichen DMZ in einer [Verfügbarkeitsgruppe][availability-set] bereit und verwenden einen [Lastenausgleich mit Internetzugriff][load-balancer], um Internetanforderungen auf die NVAs in der Verfügbarkeitsgruppe zu verteilen.  

Konfigurieren Sie den Lastenausgleich so, dass dieser nur Anforderungen über die Ports für den Internetdatenverkehr akzeptiert. Beschränken Sie z.B. eingehende HTTP-Anforderungen auf Port 80 und eingehende HTTPS-Anforderungen auf Port 443.

## <a name="scalability-considerations"></a>Überlegungen zur Skalierbarkeit

Auch wenn Ihre Architektur anfänglich eine einzelne NVA in der öffentlichen DMZ erfordert, wird empfohlen, von Anfang an einen Lastenausgleich vor der öffentlichen DMZ zu platzieren. Dies vereinfacht die Skalierung auf mehrere NVAs, wenn dies in der Zukunft erforderlich wird.

## <a name="availability-considerations"></a>Überlegungen zur Verfügbarkeit

Für den Lastenausgleich mit Internetzugriff muss jede NVA im eingehenden Subnetz der öffentlichen DMZ einen [Integritätstest][lb-probe] implementieren. Wenn ein Integritätstest an diesem Endpunkt nicht antwortet, gilt er als nicht verfügbar. Der Lastenausgleich leitet Anforderungen dann an andere NVAs in derselben Verfügbarkeitsgruppe um. Bedenken Sie, dass ein Fehler bei der Anwendung auftritt, wenn keine der NVAs antwortet. Daher ist es wichtig, eine Überwachung zu konfigurieren, mit der die DevOps gewarnt werden, wenn die Anzahl der fehlerfreien NVAs einen festgelegten Schwellenwert unterschreitet.

## <a name="manageability-considerations"></a>Überlegungen zur Verwaltbarkeit

Sämtliche Vorgänge zur Überwachung und Verwaltung der NVAs in der öffentlichen DMZ sollten über die Jumpbox im Verwaltungssubnetz ausgeführt werden. Wie bereits unter [Implementieren einer DMZ zwischen Azure und Ihrem lokalen Rechenzentrum][implementing-a-secure-hybrid-network-architecture] erläutert, definieren Sie eine einzelne Netzwerkroute aus dem lokalen Netzwerk über das Gateway bis zur Jumpbox, um den Zugriff einzuschränken.

Wenn keine Gatewaykonnektivität zwischen Ihrem lokalen Netzwerk und Azure besteht, können Sie weiterhin auf die Jumpbox zugreifen, indem Sie eine öffentliche IP-Adresse bereitstellen, diese der Jumpbox hinzufügen und sich dann über das Internet anmelden.

## <a name="security-considerations"></a>Sicherheitshinweise

Diese Referenzarchitektur implementiert mehrere Sicherheitsebenen:

* Der Lastenausgleich mit Internetzugriff leitet Anforderungen an die NVAs im eingehenden Subnetz der öffentlichen DMZ weiter – und zwar nur an Ports, die für die Anwendung erforderlich sind.
* Die NSG-Regeln für die ein- und ausgehenden Subnetze der öffentlichen DMZ verhindern, dass die NVAs gefährdet werden, indem sie Anforderungen, auf die die NSG-Regeln nicht zutreffen, blockieren.
* Die NAT-Routingkonfiguration für die NVAs leitet eingehende Anforderungen an Port 80 und Port 443 an den Lastenausgleich für die Webebene weiter, ignoriert jedoch Anforderungen an allen anderen Ports.

Sie sollten alle eingehenden Anforderungen an sämtlichen Ports protokollieren. Überprüfen Sie die Protokolle regelmäßig, und achten Sie dabei auf Anforderungen, die außerhalb der erwarteten Parameter liegen, da sie auf Eindringversuche hindeuten.

## <a name="solution-deployment"></a>Bereitstellung von Lösungen

Eine Bereitstellung für eine Referenzarchitektur, die diese Empfehlungen implementiert, steht auf [GitHub][github-folder] zur Verfügung. Die Referenzarchitektur kann mit virtuellen Windows- oder Linux-Computern bereitgestellt werden. Gehen Sie dabei gemäß den folgenden Anweisungen vor:

1. Klicken Sie auf diese Schaltfläche:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2FvirtualNetwork.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Nachdem der Link im Azure-Portal geöffnet wurde, müssen Sie Werte für einige Einstellungen eingeben:
   * Der Name der **Ressourcengruppe** ist bereits in der Parameterdatei definiert. Wählen Sie also **Neu erstellen**, und geben Sie im Textfeld `ra-public-dmz-network-rg` ein.
   * Wählen Sie im Dropdownfeld **Standort** die Region aus.
   * Lassen Sie die Textfelder für den **Vorlagenstamm-URI** bzw. **Parameterstamm-URI** unverändert.
   * Wählen Sie im Dropdownfeld den **Betriebssystemtyp** aus (**Windows** oder **Linux**).
   * Überprüfen Sie die allgemeinen Geschäftsbedingungen, und aktivieren Sie dann das Kontrollkästchen **Ich stimme den oben genannten Geschäftsbedingungen zu**.
   * Klicken Sie auf die Schaltfläche **Kaufen**.
3. Warten Sie, bis die Bereitstellung abgeschlossen ist.
4. Klicken Sie auf diese Schaltfläche:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fworkload.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Nachdem der Link im Azure-Portal geöffnet wurde, müssen Sie Werte für einige Einstellungen eingeben:
   * Der Name der **Ressourcengruppe** ist bereits in der Parameterdatei definiert. Wählen Sie also **Neu erstellen**, und geben Sie im Textfeld `ra-public-dmz-wl-rg` ein.
   * Wählen Sie im Dropdownfeld **Standort** die Region aus.
   * Lassen Sie die Textfelder für den **Vorlagenstamm-URI** bzw. **Parameterstamm-URI** unverändert.
   * Überprüfen Sie die allgemeinen Geschäftsbedingungen, und aktivieren Sie dann das Kontrollkästchen **Ich stimme den oben genannten Geschäftsbedingungen zu**.
   * Klicken Sie auf die Schaltfläche **Kaufen**.
6. Warten Sie, bis die Bereitstellung abgeschlossen ist.
7. Klicken Sie auf diese Schaltfläche:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fsecurity.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
8. Nachdem der Link im Azure-Portal geöffnet wurde, müssen Sie Werte für einige Einstellungen eingeben:
   * Der Name der **Ressourcengruppe** ist bereits in der Parameterdatei definiert. Wählen Sie also **Vorhandene verwenden** aus, und geben Sie im Textfeld `ra-public-dmz-network-rg` ein.
   * Wählen Sie im Dropdownfeld **Standort** die Region aus.
   * Lassen Sie die Textfelder für den **Vorlagenstamm-URI** bzw. **Parameterstamm-URI** unverändert.
   * Überprüfen Sie die allgemeinen Geschäftsbedingungen, und aktivieren Sie dann das Kontrollkästchen **Ich stimme den oben genannten Geschäftsbedingungen zu**.
   * Klicken Sie auf die Schaltfläche **Kaufen**.
9. Warten Sie, bis die Bereitstellung abgeschlossen ist.
10. Die Parameterdateien enthalten einen hartcodierten Administratorbenutzernamen und das zugehörige Kennwort für alle VMs. Es wird dringend empfohlen, beides sofort zu ändern. Wählen Sie jeden virtuellen Computer in der Bereitstellung im Azure-Portal aus, und klicken Sie dann auf dem Blatt **Support und Problembehandlung** auf **Kennwort zurücksetzen**. Wählen Sie im Dropdownfeld **Modus** die Option **Kennwort zurücksetzen** und dann einen neuen **Benutzernamen** und ein **Kennwort** aus. Klicken Sie auf die Schaltfläche **Aktualisieren**, um Ihre Änderungen zu speichern.


[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "Sichere Hybrid-Netzwerkarchitektur"
