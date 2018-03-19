---
layout: LandingPage
ms.openlocfilehash: 9bd86f1b3527f1116d4f5169baf76f8a5b9a385b
ms.sourcegitcommit: ea7108f71dab09175ff69322874d1bcba800a37a
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/17/2018
---
# <a name="azure-application-architecture-guide"></a>Azure-Anwendungsarchitekturleitfaden

Dieser Leitfaden stellt eine strukturierte Vorgehensweise zum Entwerfen von Anwendungen in Azure vor, die skalierbar, resilient und hochverfügbar sind. Er basiert auf bewährten Methoden, die wir im Rahmen von Kundeninteraktionen erarbeitet haben.

<img src="./images/guide-steps.svg" style="max-width:800px;"/>

## <a name="introduction"></a>Einführung

Die Cloud verändert die Art und Weise, wie Anwendungen entworfen werden. Anwendungen werden nicht mehr als Monolithen entwickelt, sondern in kleinere, dezentralisierte Dienste zerlegt. Diese Dienste kommunizieren über APIs oder durch asynchrone Nachrichten bzw. Ereignisse. Anwendungen lassen sich durch bedarfsgesteuertes Hinzufügen neuer Instanzen horizontal skalieren. 

Diese Trends bringen neue Herausforderung mit sich. Der Anwendungszustand ist verteilt. Vorgänge werden parallel und asynchron ausgeführt. Bei Ausfällen muss das System als Ganzes resilient sein. Bereitstellungen müssen automatisiert und vorhersagbar sein. Überwachung und Telemetriedaten spielen eine entscheidend Rolle, um Einblick in das System zu erhalten. Der Azure-Anwendungsarchitekturleitfaden soll Sie auf folgende Änderungen vorbereiten. 

<table>
<thead>
    <tr><th>Konventionelle lokale Systeme</th><th>Moderne Cloud</th></tr>
</thead>
<tbody>
<tr><td>Monolithisch, zentralisiert<br/>
Entwurf mit Blick auf vorhersagbare Skalierbarkeit<br/>
Relationale Datenbank<br/>
Hohe Konsistenz<br/>
Serielle und synchronisierte Verarbeitung<br/>
Entwurf mit Blick auf Vermeidung von Ausfällen (MTBF)<br/>
Gelegentlich umfangreiche Updates<br/>
Manuelle Verwaltung<br/>
Snowflake-Server</td>
<td>
Zerlegt, dezentralisiert<br/>
Entwurf mit Blick auf elastische Skalierung<br/>
Polyglot Persistence (Kombination aus Speichertechnologien)<br/>
Letztliche Konsistenz<br/>
Parallele und asynchrone Verarbeitung<br/>
Entwurf mit Blick auf Ausfälle (MTTR)<br/>
Häufige geringfügige Updates<br/>
Automatisierte Selbstverwaltung<br/>
Unveränderliche Infrastruktur<br/>
</td>
</tbody>
</table>

Dieser Leitfaden richtet sich an Anwendungsarchitekten, Entwickler und Betriebsteams. Er dient nicht als Anleitung für die Verwendung der einzelnen Azure-Dienste. Nachdem Sie diesen Leitfaden gelesen haben, werden Sie die Architekturmuster und bewährten Methoden, die beim Erstellen der Azure-Cloudplattform anzuwenden sind, kennen. Sie können auch eine [E-Book-Version des Handbuchs][ebook] herunterladen.

## <a name="how-this-guide-is-structured"></a>Aufbau dieses Leitfadens

Der Azure-Anwendungsarchitekturleitfaden ist als Abfolge von Schritten aufgebaut, die sich von der Architektur über den Entwurf bis hin zur Implementierung erstrecken. Zu jedem Schritt werden unterstützende Anweisungen bereitgestellt, die Sie beim Entwurf der Anwendungsarchitektur unterstützen.

**[Architekturstile][arch-styles]**: Der erste Entscheidungspunkt ist gleichzeitig der wichtigste. Welche Art von Architektur erstellen Sie? Ist eine Microservicearchitektur, eine konventionellere Anwendung mit n-Schichten oder eine Big Data-Lösung das Ziel? Wir haben sieben unterschiedliche Architekturstile ermittelt. Jeder Stil weist Vor- und Nachteile auf.

> &#10148; [Azure-Referenzarchitekturen][ref-archs] geben empfohlene Bereitstellungen in Azure und Überlegungen in Bezug auf die Skalierbarkeit, Verfügbarkeit, Verwaltbarkeit und Sicherheit an. Bei einem Großteil werden außerdem bereitstellbare Ressourcen-Manager-Vorlagen zur Verfügung gestellt.

**[Auswahl der Technologie][technology-choices]**: Direkt am Anfang sollte eine Entscheidung für eine von zwei Technologieoptionen getroffen werden, da dies Auswirkungen auf die gesamte Architektur hat. Wählen Sie zwischen der Compute- und der Speichertechnologie. Die Benennung *Compute* bezieht sich auf das Hostingmodell für die Computeressourcen, mit der Ihre Anwendungen ausgeführt werden. Zu Speichern zählen Datenbanken, aber auch Speicher für Nachrichtenwarteschlangen, Caches, IoT-Daten, unstrukturierte Protokolldaten und sonstige Daten, die eine Anwendung in einem Speicher speichern kann. 

> &#10148; [Computeoptionen][compute-options] und [Speicheroptionen][storage-options] bieten ausführliche Vergleichskriterien für die Auswahl von Compute- und Speicherdiensten.

**[Entwurfsprinzipien][design-principles]**: Beim Entwurfsprozess sollten Sie die folgenden zehn allgemeinen Entwurfsprinzipien berücksichtigen. 

> &#10148; Die Artikel zu den [bewährten Methoden][best-practices] enthalten spezielle Anweisungen zu Themen wie etwa automatische Skalierung, Caching, Datenpartitionierung und API-Entwurf.   

**[Säulen][pillars]**: Eine gelungene Cloudanwendung basiert auf fünf Säulen der Softwarequalität, nämlich Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltung und Sicherheit. 

> &#10148; Verwenden Sie unsere [Checklisten zur Entwurfsüberprüfung][checklists], um Ihren Entwurf im Hinblick auf diese Qualitätssäulen zu überprüfen. 

**[Cloudentwurfsmuster][patterns]**: Diese Entwurfsmuster können Ihnen dabei helfen, zuverlässige, skalierbare und sichere Anwendungen in Azure zu erstellen. Jedes Muster beschreibt ein Problem, einen Ansatz zu dessen Lösung und ein auf Azure basierendes Beispiel.

> &#10148; Sehen Sie sich den vollständigen [Katalog von Cloudentwurfsmustern](../patterns/index.md) an.


[arch-styles]: ./architecture-styles/index.md
[best-practices]: ../best-practices/index.md
[checklists]: ../checklist/index.md
[compute-options]: ./technology-choices/compute-comparison.md
[design-principles]: ./design-principles/index.md
[ebook]: https://azure.microsoft.com/campaigns/cloud-application-architecture-guide/
[patterns]: ../patterns/index.md?toc=/azure/architecture/guide/toc.json
[pillars]: ./pillars.md
[ref-archs]: ../reference-architectures/index.md
[storage-options]: ./technology-choices/data-store-comparison.md
[technology-choices]: ./technology-choices/index.md

