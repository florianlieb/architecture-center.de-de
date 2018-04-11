---
title: Netzwerk-DMZ
description: Erläuterung und Vergleich der verschiedenen verfügbaren Methoden zum Schutz von Anwendungen und Komponenten, die in Azure als Teil eines Hybridsystems ausgeführt werden, vor unbefugtem Eindringen
layout: LandingPage
ms.openlocfilehash: 98df0a25767c7a7282e67381c6465fe3263ce1fa
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/23/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="network-dmz"></a>Netzwerk-DMZ

Diese Referenzarchitekturen zeigen bewährte Methoden für die Erstellung einer Netzwerk-DMZ, die die Grenze zwischen einem virtuellen Azure-Netzwerk und einem lokalen Netzwerk oder dem Internet schützt.

<section class="series">
    <ul class="panelContent">
    <!-- DMZ between Azure and on-premises -->
<li style="display: flex; flex-direction: column;">
    <a href="./secure-vnet-hybrid.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/secure-vnet-hybrid.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>DMZ zwischen Azure und einem lokalen Netzwerk</h3>
                        <p>Implementiert ein sicheres hybrides Netzwerk, das ein lokales Netzwerk auf Azure erweitert.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- DMZ between Azure and the Internet -->
<li style="display: flex; flex-direction: column;">
    <a href="./secure-vnet-dmz.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/secure-vnet-dmz.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>DMZ zwischen Azure und dem Internet</h3>
                        <p>Implementiert ein sicheres Netzwerk, das Internetdatenverkehr nach Azure akzeptiert.</p>
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