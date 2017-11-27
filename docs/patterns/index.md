---
title: Cloudentwurfsmuster
description: "Cloudentwurfsmuster für Microsoft Azure"
keywords: Azure
ms.openlocfilehash: bf9fb2555f5c80cab9e4616ba52155bf1284d26f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="cloud-design-patterns"></a>Cloudentwurfsmuster

Diese Entwurfsmuster können Ihnen dabei helfen, zuverlässige, skalierbare und sichere Anwendungen in der Cloud zu erstellen.

Für jedes Muster werden das durch das Muster gelöste Problem, Überlegungen zum Anwenden des Musters und ein Beispiel auf der Grundlage von Microsoft Azure beschrieben. Die meisten der Muster enthalten Codebeispiele oder -ausschnitte, die die Implementierung des Musters auf Azure veranschaulichen. Der Großteil der Muster ist jedoch für alle verteilten Systeme relevant, unabhängig davon, ob sie auf Azure oder anderen Cloudplattformen gehostet werden.

## <a name="challenges-in-cloud-development"></a>Herausforderungen bei der Cloudentwicklung

<table>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/availability.md"><img src="_images/category/availability.svg" alt="Availability" /></a></td>
    <td>
        <h3><a href="./category/availability.md">Availability</a></h3>
        <p>Die Verfügbarkeit ist der Anteil der Zeit, während dessen das System funktionsfähig und in Betrieb ist. Sie wird in der Regel als Prozentsatz der Betriebszeit gemessen. Die Verfügbarkeit kann durch Systemfehler, Infrastrukturprobleme, böswillige Angriffe und die Systemauslastung beeinflusst werden.  Cloudanwendungen bieten Benutzern in der Regel eine Vereinbarung zum Servicelevel (SLA). Anwendungen müssen daher zur Maximierung der Verfügbarkeit konzipiert werden.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/data-management.md"><img src="_images/category/data-management.svg" alt="Data Management" /></a></td>
    <td>
        <h3><a href="./category/data-management.md">Datenverwaltung</a></h3>
        <p>Die Datenverwaltung ist das wichtigste Element von Cloudanwendungen und wirkt sich auf die meisten Qualitätsattribute aus. Meist werden Daten aus Gründen der Leistung, Skalierbarkeit oder Verfügbarkeit an verschiedenen Speicherorten und auf mehreren Servern gehostet, und dies kann eine ganze Reihe von Herausforderungen mit sich bringen. Beispielsweise muss die Konsistenz der Daten aufrechterhalten werden, und normalerweise müssen Daten zwischen unterschiedlichen Speicherorten synchronisiert werden.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/design-implementation.md"><img src="_images/category/design-implementation.svg" alt="Design and Implementation" /></a></td>
    <td>
        <h3><a href="./category/design-implementation.md">Entwurf und Implementierung</a></h3>
        <p>Ein guter Entwurf berücksichtigt Faktoren wie Konsistenz und Kohärenz im Komponentenentwurf und in der Bereitstellung, Wartbarkeit zur Vereinfachung der Verwaltung und Entwicklung sowie Wiederverwendbarkeit, um die Verwendung von Komponenten und Subsystemen in anderen Anwendungen und Szenarien zu ermöglichen. Während der Entwurfs- und Implementierungsphase getroffene Entscheidungen haben großen Einfluss auf die Qualität und die Gesamtkosten von in der Cloud gehosteten Anwendungen und Diensten.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/messaging.md"><img src="_images/category/messaging.svg" alt="Messaging" /></a></td>
    <td>
        <h3><a href="./category/messaging.md">Nachrichten</a></h3>
        <p>Die verteilte Struktur von Cloudanwendungen erfordert eine Messaginginfrastruktur, die die Komponenten und Dienste – idealerweise in einer lose gekoppelten Weise – miteinander verbindet, um die Skalierbarkeit zu maximieren. Asynchrones Messaging ist weit verbreitet und bietet viele Vorteile, beinhaltet aber auch Herausforderungen, beispielsweise die Reihenfolge von Nachrichten, die Verwaltung von nicht verarbeitbaren Nachrichten, Idempotenz und vieles mehr.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/management-monitoring.md"><img src="_images/category/management-monitoring.svg" alt="Management and Monitoring" /></a></td>
    <td>
        <h3><a href="./category/management-monitoring.md">Verwaltung und Überwachung</a></h3>
        <p>Cloudanwendungen werden in einem Remotedatencenter ausgeführt, in dem Sie keine vollständige Kontrolle über die Infrastruktur oder – in manchen Fällen – das Betriebssystem haben. Die Verwaltung und Überwachung kann daher schwieriger sein als bei einer lokalen Bereitstellung. Anwendungen müssen Laufzeitinformationen verfügbar machen, die Administratoren und Betreiber zum Verwalten und Überwachen des Systems verwenden können. Zudem müssen sie sich ändernde Geschäftsanforderungen und Anpassungen ohne Beenden oder erneutes Bereitstellen der Anwendung unterstützen.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/performance-scalability.md"><img src="_images/category/performance-scalability.svg" alt="Performance and Scalability" /></a></td>
    <td>
        <h3><a href="./category/performance-scalability.md">Leistung und Skalierbarkeit</a></h3>
        <p>Die Leistung gibt Aufschluss über die Reaktionsfähigkeit eines Systems, d. h. seine Fähigkeit, alle Aktionen innerhalb eines bestimmten Zeitintervalls auszuführen. Die Skalierbarkeit ist dagegen die Fähigkeit eines Systems, einen Anstieg der Last ohne Auswirkungen auf die Leistung zu bewältigen oder die Anzahl verfügbarer Ressourcen schnell und problemlos zu erhöhen. Bei Cloudanwendungen treten in der Regel variable Arbeitsauslastungen und Aktivitätsspitzen auf. Diese vorherzusagen, ist insbesondere in einem Szenario mit mehreren Mandanten nahezu unmöglich. Stattdessen sollten sich Anwendungen innerhalb der jeweiligen Grenzen je nach Bedarf horizontal hoch- und herunterskalieren lassen. Die Skalierbarkeit betrifft nicht nur Compute-Instanzen, sondern auch andere Elemente wie den Datenspeicher, die Messaginginfrastruktur usw.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/resiliency.md"><img src="_images/category/resiliency.svg" alt="Resiliency" /></a></td>
    <td>
        <h3><a href="./category/resiliency.md">Resilienz</a></h3>
        <p>Die Resilienz ist die Fähigkeit eines Systems, Fehler ordnungsgemäß zu behandeln und nach Fehlern ordnungsgemäß wiederhergestellt werden zu können. Aufgrund des Funktionsprinzips von Cloudhosting, bei dem Anwendungen oft mehrmandantenfähig sind, freigegebene Plattformdienste nutzen, um Ressourcen und Bandbreite konkurrieren, über das Internet kommunizieren und auf Standardhardware ausgeführt werden, ist die Wahrscheinlichkeit höher, dass sowohl vorübergehende als auch mehr permanente Fehler auftreten. Zum Gewährleisten der Resilienz sind eine schnelle und effiziente Fehlererkennung sowie Wiederherstellung erforderlich.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/security.md"><img src="_images/category/security.svg" alt="Security" /></a></td>
    <td>
        <h3><a href="./category/security.md">Sicherheit</a></h3>
        <p>Die Sicherheit ist die Fähigkeit eines Systems, böswillige oder unbeabsichtigte Aktionen, die nicht dem Verwendungszweck des Systems entsprechen, sowie die Offenlegung oder den Verlust von Informationen zu verhindern. Cloudanwendungen werden im Internet außerhalb vertrauenswürdiger lokaler Grenzen verfügbar gemacht, sind oft öffentlich zugänglich und können von nicht vertrauenswürdigen Benutzern verwendet werden. Durch den Entwurf und die Bereitstellung von Anwendungen muss sichergestellt werden, dass sie vor böswilligen Angriffen geschützt sind, der Zugriff auf genehmigte Benutzer beschränkt ist und vertrauliche Daten sicher sind.</p>
    </td>
</tr>
</table>

## <a name="catalog-of-patterns"></a>Musterkatalog

| Muster | Zusammenfassung |
| ------- | ------- |
| [Botschafter](./ambassador.md) | Hilfsdienste erstellen, die Anforderungen im Auftrag von Verbraucherdiensten oder -anwendungen über das Netzwerk senden |
| [Antibeschädigungsebene](./anti-corruption-layer.md) | Eine „Fassade“ oder Adapterebene zwischen einer modernen Anwendung und einem älteren System implementieren |
| [Back-Ends für Front-Ends](./backends-for-frontends.md) | Separate Back-End-Dienste zur Nutzung durch bestimmte Front-End-Anwendungen oder -Schnittstellen erstellen |
| [Bulkhead](./bulkhead.md) | Elemente einer Anwendung in Pools isolieren, sodass die anderen Elemente beim Ausfall eines Elements weiterhin ausgeführt werden |
| [Cache-Aside](./cache-aside.md) | Daten bei Bedarf in einen Cache aus einem Datenspeicher laden |
| [Trennschalter](./circuit-breaker.md) | Fehler behandeln, deren Behebung beim Herstellen einer Verbindung mit einem Remotedienst oder einer Remoteressource unterschiedlich lange dauern kann |
| [Befehlsabfrage-Zuständigkeitstrennung (Command Query Responsibility Segregation, CQRS)](./cqrs.md) | Mithilfe separater Schnittstellen Vorgänge trennen, die Daten von Vorgängen zur Aktualisierung von Daten lesen |
| [Ausgleichende Transaktion](./compensating-transaction.md) | Durch eine Reihe von Schritten, die zusammen letztlich einen konsistenten Vorgang definieren, ausgeführte Arbeit rückgängig machen |
| [Konkurrierende Consumer](./competing-consumers.md) | Mehreren gleichzeitigen Consumern die Verarbeitung von Nachrichten ermöglichen, die auf dem gleichen Messagingkanal empfangen werden |
| [Computeressourcenkonsolidierung](./compute-resource-consolidation.md) | Mehrere Aufgaben oder Vorgänge in einer einzelnen Berechnungseinheit konsolidieren |
| [Ereignisherkunftsermittlung](./event-sourcing.md) | Einen nur zum Anfügen vorgesehenen Speicher verwenden, um die vollständige Serie von Ereignissen aufzuzeichnen, die an Daten in einer Domäne ausgeführte Aktionen beschreiben |
| [Externer Konfigurationsspeicher](./external-configuration-store.md) | Konfigurationsinformationen aus dem Anwendungsbereitstellungspaket an einen zentralen Speicherort verschieben |
| [Verbundidentität](./federated-identity.md) | Authentifizierung an einen externen Identitätsanbieter delegieren |
| [Gatekeeper](./gatekeeper.md) | Anwendungen und Dienste durch Verwendung einer dedizierten Hostinstanz schützen, die als Broker zwischen Clients und der Anwendung oder dem Dienst fungiert, Anforderungen überprüft und bereinigt sowie Anforderungen und Daten zwischen ihnen weiterleitet |
| [Gatewayaggregation](./gateway-aggregation.md) | Mithilfe eines Gateways mehrere einzelne Anforderungen in einer einzelnen Anforderung aggregieren |
| [Gatewayabladung](./gateway-offloading.md) | Freigegebene oder spezielle Dienstfunktionen an einen Gatewayproxy auslagern |
| [Gatewayrouting](./gateway-routing.md) | Anforderungen an mehrere Dienste mit einem einzelnen Endpunkt weiterleiten |
| [Überwachung für den Integritätsendpunkt](./health-endpoint-monitoring.md) | Funktionale Prüfungen innerhalb einer Anwendung implementieren, auf die externe Tools in regelmäßigen Abständen über verfügbar gemachte Endpunkte zugreifen können |
| [Indextabelle](./index-table.md) | Indizes für die Felder im Datenspeicher erstellen, auf die häufig von Abfragen verwiesen wird |
| [Auswahl einer übergeordneten Instanz](./leader-election.md) | Die von einer Sammlung zusammenarbeitender Aufgabeninstanzen ausgeführten Aktionen in einer verteilten Anwendung koordinieren, indem eine Instanz als übergeordnete Instanz ausgewählt wird, die die Verantwortung für die Verwaltung der anderen Instanzen übernimmt |
| [Materialisierte Sicht](./materialized-view.md) | Voraufgefüllte Sichten für die Daten in einem oder mehreren Datenspeichern generieren, wenn die Daten für erforderliche Abfragevorgänge nicht ideal formatiert sind |
| [Pipes und Filter](./pipes-and-filters.md) | Eine Aufgabe, die komplexe Verarbeitungsvorgänge ausführt, in eine Reihe wiederverwendbarer separater Elemente unterteilen |
| [Prioritätswarteschlange](./priority-queue.md) | An Dienste gesendete Anforderungen priorisieren, sodass Anforderungen mit einer höheren Priorität schneller empfangen und verarbeitet werden als Anforderungen mit einer niedrigeren Priorität |
| [Warteschlangenbasierter Lastenausgleich](./queue-based-load-leveling.md) | Eine Warteschlange verwenden, die als Puffer zwischen einer Aufgabe und einem von ihr aufgerufenen Dienst fungiert, um unregelmäßig auftretende hohe Lasten aufzufangen |
| [Wiederholung](./retry.md) | Einer Anwendung beim Herstellen einer Verbindung mit einem Dienst oder einer Netzwerkressource die Behandlung antizipierter, temporärer Fehler ermöglichen, indem ein Vorgang, bei dem zuvor ein Fehler aufgetreten ist, transparent wiederholt wird |
| [Scheduler-Agent-Supervisor](./scheduler-agent-supervisor.md) | Eine Reihe von Aktionen in einer verteilten Gruppe von Diensten und anderen Remoteressourcen koordinieren |
| [Sharding](./sharding.md) | Einen Datenspeicher in einen Satz horizontaler Partitionen oder Shards unterteilen |
| [Sidecar](./sidecar.md) | Komponenten einer Anwendung zwecks Isolation und Kapselung in einem separaten Prozess oder Container bereitstellen |
| [Hosten von statischen Inhalten](./static-content-hosting.md) | Statische Inhalte in einem cloudbasierten Speicherdienst bereitstellen, der die Inhalte direkt an den Client übermitteln kann |
| [Einschnürung](./strangler.md) | Ein älteres System inkrementell migrieren, indem bestimmte Teile der Funktionalität nach und nach durch neue Anwendungen und Dienste ersetzt werden |
| [Drosselung](./throttling.md) | Den Verbrauch der von einer Anwendungsinstanz, einem einzelnen Mandanten oder einem gesamten Dienst verwendeten Ressourcen steuern |
| [Valet-Schlüssel](./valet-key.md) | Ein Token oder einen Schlüssel verwenden, das bzw. der Clients eingeschränkten direkten Zugriff auf eine bestimmte Ressource oder einen bestimmten Dienst bietet |