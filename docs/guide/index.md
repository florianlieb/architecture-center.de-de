---
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 80cb7fde0694257a5c413b702505e27f18aed8d3
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/23/2018
---
# <a name="azure-application-architecture-guide"></a>Azure-Anwendungsarchitekturleitfaden

Dieser Leitfaden stellt eine strukturierte Vorgehensweise zum Entwerfen von Anwendungen in Azure vor, die skalierbar, resilient und hochverfügbar sind. Er basiert auf bewährten Methoden, die wir im Rahmen von Kundeninteraktionen erarbeitet haben.

<br/>

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

### <a name="architecture-styles"></a>Architekturstile

Der erste Entscheidungspunkt ist gleichzeitig der wichtigste. Welche Art von Architektur erstellen Sie? Ist eine Microservicearchitektur, eine konventionellere Anwendung mit n-Schichten oder eine Big Data-Lösung das Ziel? Wir haben unterschiedliche Architekturstile ermittelt. Jeder Stil weist Vor- und Nachteile auf.

Weitere Informationen:

- [Architekturstile][arch-styles]
- [Azure-Referenzarchitekturen][ref-archs]

### <a name="technology-choices"></a>Auswahl der Technologie

Direkt am Anfang sollte eine Entscheidung für eine von zwei Technologieoptionen getroffen werden, da dies Auswirkungen auf die gesamte Architektur hat. Wählen Sie zwischen Computediensten und Datenspeichern. *Compute* bezieht sich auf das Hostingmodell für die Computeressourcen, mit dem Ihre Anwendungen ausgeführt werden. Zu *Datenspeichern* zählen Datenbanken, aber auch Speicher für Nachrichtenwarteschlangen, Caches, Protokolle und sonstige Daten, die eine Anwendung in einem Speicher speichern kann. 

Weitere Informationen:

- [Wählen eines Computediensts](./technology-choices/compute-overview.md)
- [Auswählen eines Datenspeichers](./technology-choices/data-store-overview.md)

### <a name="design-principles"></a>Entwurfsprinzipien

Wir haben zehn allgemeine Entwurfsprinzipien identifiziert, die die Skalierbarkeit, Resilienz und Verwaltbarkeit Ihrer Anwendung optimieren. Diese Entwurfsprinzipien gelten für alle Architekturstile. Beim Entwurfsprozess sollten Sie die folgenden zehn allgemeinen Entwurfsprinzipien berücksichtigen. Berücksichtigen Sie dann die bewährten Methoden für bestimmte Aspekte der Architektur, etwa automatische Skalierung, Caching, Datenpartitionierung, API-Design usw.

Weitere Informationen:

- [Entwurfsprinzipien für Azure-Anwendungen][design-principles]
- [Bewährte Methoden für das Erstellen für die Cloud][best-practices]

### <a name="quality-pillars"></a>Qualitätssäulen

Eine gelungene Cloudanwendung basiert auf fünf Säulen der Softwarequalität, nämlich Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltung und Sicherheit. Verwenden Sie unsere Checklisten zur Entwurfsüberprüfung, um Ihre Architektur im Hinblick auf diese Qualitätssäulen zu überprüfen.

Weitere Informationen:

- [Säulen der Softwarequalität][pillars]
- [Checklisten für die Entwurfsüberprüfung][checklists] 

### <a name="cloud-design-patterns"></a>Cloudentwurfsmuster

Entwurfsmuster bieten allgemeine Lösungen für häufige Probleme beim Softwareentwurf. Wir haben verschiedene Entwurfsmuster identifiziert, die insbesondere beim Entwerfen verteilter Anwendungen für die Cloud praktisch sind.

Weitere Informationen:

- [Katalog mit Cloudentwurfsmustern](../patterns/index.md)


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

