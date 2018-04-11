---
title: Linux-VM-Workloads
description: Beschreibung einiger gängiger Architekturen für die Bereitstellung von VMs, die in Azure Anwendungen auf Unternehmensebene hosten
layout: LandingPage
ms.openlocfilehash: ef06fb93355f4676f44954930bcfeb2c2d012d43
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/23/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="linux-vm-workloads"></a>Linux-VM-Workloads

Diese Referenzarchitekturen zeigen bewährte Methoden für die Ausführung von Linux-VMs in Azure.

<section class="series">
    <ul class="panelContent">
    <!-- Single VM -->
<li style="display: flex; flex-direction: column;">
    <a href="./single-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/single-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Einzelne VM</h3>
                        <p>Grundlegende Empfehlungen für die Ausführung einer beliebigen Linux-VM in Azure</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Load balanced VMs -->
<li style="display: flex; flex-direction: column;">
    <a href="./multi-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/multi-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>VMs mit Lastenausgleich</h3>
                        <p>Platzieren mehrerer VMs hinter einem Lastenausgleichsmodul zur Erzielung von Skalierbarkeit und Verfügbarkeit</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- N-tier application -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/n-tier.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Anwendung mit n-Schichten</h3>
                        <p>VMs, die mit Apache Cassandra für eine Anwendung mit n-Schichten konfiguriert sind</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Multi-region application -->
<li style="display: flex; flex-direction: column;">
    <a href="./multi-region-application.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/multi-region-application.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Anwendung in mehreren Regionen</h3>
                        <p>Zur Erzielung von Hochverfügbarkeit in zwei Regionen bereitgestellte Anwendung mit n-Schichten</p>
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