---
title: Verbinden eines lokalen Netzwerks mit Azure
description: Empfohlene Architekturen für sichere, stabile Netzwerkverbindungen zwischen lokalen Netzwerken und Azure
layout: LandingPage
ms.openlocfilehash: 372efb8ecf69245a5895c51e3da156a348bd665e
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/08/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="connect-an-on-premises-network-to-azure"></a>Verbinden eines lokalen Netzwerks mit Azure

Diese Referenzarchitekturen zeigen bewährte Methoden für die Erstellung einer stabilen Netzwerkverbindung zwischen einem lokalen Netzwerk und Azure. [Welche Option sollte ich auswählen?](./considerations.md)

<section class="series">
    <ul class="panelContent">
    <!-- VPN -->
<li style="display: flex; flex-direction: column;">
    <a href="./vpn.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/vpn.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>VPN</h3>
                        <p>Erweitern Sie ein lokales Netzwerk auf Azure, indem Sie ein Site-to-Site-VPN (virtuelles privates Netzwerk) verwenden.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- ExpressRoute -->
<li style="display: flex; flex-direction: column;">
    <a href="./expressroute.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/expressroute.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ExpressRoute</h3>
                        <p>Erweitern Sie ein lokales Netzwerk mit Azure ExpressRoute auf Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- ExpressRoute with VPN failover -->
<li style="display: flex; flex-direction: column;">
    <a href="./expressroute-vpn-failover.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/expressroute-vpn-failover.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ExpressRoute mit VPN-Failover</h3>
                        <p>Erweitern Sie ein lokales Netzwerk mit Azure ExpressRoute auf Azure, indem Sie ein VPN als Failoververbindung verwenden.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Hub-spoke topology -->
<li style="display: flex; flex-direction: column;">
    <a href="./hub-spoke.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/hub-spoke.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Hub-Spoke-Topologie</h3>
                        <p>Der Hub ist ein zentraler Konnektivitätspunkt für Ihr lokales Netzwerk. Bei Speichen handelt es sich um VNETs, die eine Peerverbindung mit dem Hub herstellen und zur Isolierung von Workloads verwendet werden können.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Hub-spoke topology with shared services -->
<li style="display: flex; flex-direction: column;">
    <a href="./shared-services.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/shared-services.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Hub-Spoke-Topologie mit gemeinsamen Diensten</h3>
                        <p>Stellen Sie eine Hub-Spoke-Topologie mit gemeinsamen Diensten bereit, z.B. Active Directory-Dienste und ein virtuelles Netzwerkgerät (Network Virtual Appliance, NVA). Gemeinsame Dienste können von allen Spokes genutzt werden.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    </ul>
</section>

<ul class="panelContent cardsI">
</ul>