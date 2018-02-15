---
title: Azure-Referenzarchitekturen
description: "Referenzarchitekturen, Blaupausen und reglementierende Implementierungsanweisungen für gängige Workloads in Azure"
layout: LandingPage
NOTE: edit the template in ./build/reference-architectures !!!
ms.openlocfilehash: e513e2545fb95b486b11a067479e879f4abd4a8f
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="azure-reference-architectures"></a>Azure-Referenzarchitekturen

Unsere Referenzarchitekturen sind nach Szenarien angeordnet, wobei verwandte Architekturen gruppiert sind. Jede Architektur ist mit empfohlenen Methoden sowie Überlegungen in Bezug auf die Skalierbarkeit, Verfügbarkeit, Verwaltbarkeit und Sicherheit versehen. Bei einem Großteil wird außerdem eine bereitstellbare Lösung angegeben.

<section class="series">
    <ul class="panelContent">
    <!-- Windows VM workloads -->
<li style="display: flex; flex-direction: column;">
    <a href="./virtual-machines-windows/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./virtual-machines-windows/images/n-tier.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Windows-VM-Workloads</h3>
                        <p>Diese Serie beginnt mit bewährten Methoden für die Ausführung einer einzelnen Windows-VM, anschließend mehrerer VMs mit Lastenausgleich und schließlich einer Anwendung mit n-Schichten in mehreren Regionen.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Linux VM workloads -->
<li style="display: flex; flex-direction: column;">
    <a href="./virtual-machines-linux/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./virtual-machines-linux/images/n-tier.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Linux-VM-Workloads</h3>
                        <p>Diese Serie beginnt mit bewährten Methoden für die Ausführung einer einzelnen Linux-VM, anschließend mehrerer VMs mit Lastenausgleich und schließlich einer Anwendung mit n-Schichten in mehreren Regionen.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Hybrid network -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./hybrid-networking/images/vpn.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Hybrides Netzwerk</h3>
                        <p>Diese Serie zeigt Optionen zum Erstellen einer Netzwerkverbindung zwischen einem lokalen Netzwerk und Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Network DMZ -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./dmz/images/secure-vnet-dmz.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Netzwerk-DMZ</h3>
                        <p>Diese Serie zeigt, wie man eine Netzwerk-DMZ erstellt, um die Grenze zwischen einem virtuellen Azure-Netzwerk und einem lokalen Netzwerk oder dem Internet zu schützen.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Identity management -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./identity/images/adds-extend-domain.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Identitätsverwaltung</h3>
                        <p>Diese Serie zeigt Optionen für die Integration Ihrer lokalen Active Directory-Umgebung (AD) in ein Azure-Netzwerk.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- App Service web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./app-service-web-app/images/scalable-web-app.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>App Service-Webanwendung</h3>
                        <p>Diese Serie zeigt bewährte Methoden für Webanwendungen, die Azure App Service nutzen.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    </ul>
</section>

<ul class="panelContent cardsI">
    <!-- Jenkins build server -->
<li style="display: flex; flex-direction: column;">
    <a href="./jenkins/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./jenkins/images/logo.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Jenkins-Buildserver</h3>
                        <p>Bereitstellen und Betreiben eines skalierbaren Jenkins-Servers in Azure auf Unternehmensniveau</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- SharePoint Server 2016 farm -->
<li style="display: flex; flex-direction: column;">
    <a href="./sharepoint/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./sharepoint/images/sharepoint.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SharePoint Server 2016-Farm</h3>
                        <p>Sie stellen eine SharePoint Server 2016-Hochverfügbarkeitsfarm mithilfe von SQL Server AlwaysOn-Verfügbarkeitsgruppen in Azure bereit und führen diese aus.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- SAP NetWeaver and SAP HANA -->
<li style="display: flex; flex-direction: column;">
    <a href="./sap/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./sap/images/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP NetWeaver und SAP HANA</h3>
                        <p>Sie stellen SAP NetWeaver und SAP HANA in einer Hochverfügbarkeitsumgebung in Azure bereit und führen diese aus.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>