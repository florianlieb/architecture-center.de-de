---
title: Azure-Referenzarchitekturen
description: "Referenzarchitekturen, Blaupausen und reglementierende Implementierungsanweisungen für gängige Workloads in Azure"
layout: LandingPage
ms.openlocfilehash: 0cc03f87dae39517e1a72a65d4767dcc21879d8f
ms.sourcegitcommit: 9998334bebccb86be0f715ac7dffc0c3175aea68
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/26/2018
---
# <a name="azure-reference-architectures"></a>Azure-Referenzarchitekturen

Unsere Referenzarchitekturen sind nach Szenarien angeordnet, wobei verwandte Architekturen gruppiert sind. Jede Architektur ist mit empfohlenen Methoden sowie Überlegungen in Bezug auf die Skalierbarkeit, Verfügbarkeit, Verwaltbarkeit und Sicherheit versehen. Bei einem Großteil wird außerdem eine bereitstellbare Lösung angegeben.

<section class="series">
    <ul class="panelContent">
    <!--Windows VM -->
    <li>
        <a href="./virtual-machines-windows/index.md">
            <div class="cardSize">
                <div class="cardPadding">
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
    <!-- Linux VM -->
    <li>
        <a href="./virtual-machines-linux/index.md">
            <div class="cardSize">
                <div class="cardPadding">
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
    <li>
        <a href="./hybrid-networking/index.md">
            <div class="cardSize">
                <div class="cardPadding">
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
    <!-- DMZ -->
    <li>
        <a href="./dmz/index.md">
            <div class="cardSize">
                <div class="cardPadding">
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
    <!-- Identity -->
    <li>
        <a href="./identity/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./identity/images/adds-extend-domain.svg" height="140px" >
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
    <!-- Managed web app -->
    <li>
        <a href="./app-service-web-app/index.md">
            <div class="cardSize">
                <div class="cardPadding">
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

<ul class="panelContent cardsI">
<li>
    <a href="./jenkins/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./jenkins/images/logo.svg" alt="Jenkins" height="100%" />
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

<li>
    <a href="./sharepoint/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./sharepoint/images/sharepoint.svg" alt="SharePoint Server 2016" height="100%" />
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

<li>
    <a href="./sap/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./sap/images/sap.svg" alt="SAP NetWeaver and SAP HANA" width="100%" />
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


</section>

