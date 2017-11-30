---
title: "Muster für Entwurf und Implementierung"
description: "Ein guter Entwurf berücksichtigt Faktoren wie Konsistenz und Kohärenz im Komponentenentwurf und in der Bereitstellung, Wartbarkeit zur Vereinfachung der Verwaltung und Entwicklung sowie Wiederverwendbarkeit, um die Verwendung von Komponenten und Subsystemen in anderen Anwendungen und Szenarien zu ermöglichen. Während der Entwurfs- und Implementierungsphase getroffene Entscheidungen haben großen Einfluss auf die Qualität und die Gesamtkosten von in der Cloud gehosteten Anwendungen und Diensten."
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 3d6e528b8c88c6fcc265d6425dcdae6fae5166fb
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="design-and-implementation-patterns"></a>Muster für Entwurf und Implementierung

Ein guter Entwurf berücksichtigt Faktoren wie Konsistenz und Kohärenz im Komponentenentwurf und in der Bereitstellung, Wartbarkeit zur Vereinfachung der Verwaltung und Entwicklung sowie Wiederverwendbarkeit, um die Verwendung von Komponenten und Subsystemen in anderen Anwendungen und Szenarien zu ermöglichen. Während der Entwurfs- und Implementierungsphase getroffene Entscheidungen haben großen Einfluss auf die Qualität und die Gesamtkosten von in der Cloud gehosteten Anwendungen und Diensten.

| Muster | Zusammenfassung |
| ------- | ------- |
| [Ambassador](../ambassador.md) | Hilfsdienste erstellen, die Anforderungen im Auftrag von Verbraucherdiensten oder -anwendungen über das Netzwerk senden |
| [Antibeschädigungsebene](../anti-corruption-layer.md) | Eine „Fassade“ oder Adapterebene zwischen einer modernen Anwendung und einem älteren System implementieren |
| [Back-Ends für Front-Ends](../backends-for-frontends.md) | Separate Back-End-Dienste zur Nutzung durch bestimmte Front-End-Anwendungen oder -Schnittstellen erstellen |
| [CQRS](../cqrs.md) | Mithilfe separater Schnittstellen Vorgänge trennen, die Daten von Vorgängen zur Aktualisierung von Daten lesen |
| [Computeressourcenkonsolidierung](../compute-resource-consolidation.md) | Mehrere Aufgaben oder Vorgänge in einer einzelnen Berechnungseinheit konsolidieren |
| [Externer Konfigurationsspeicher](../external-configuration-store.md) | Konfigurationsinformationen aus dem Anwendungsbereitstellungspaket an einen zentralen Speicherort verschieben |
| [Gatewayaggregation](../gateway-aggregation.md) | Mithilfe eines Gateways mehrere einzelne Anforderungen in einer einzelnen Anforderung aggregieren |
| [Gatewayabladung](../gateway-offloading.md) | Freigegebene oder spezielle Dienstfunktionen an einen Gatewayproxy auslagern |
| [Gatewayrouting](../gateway-routing.md) | Anforderungen an mehrere Dienste mit einem einzelnen Endpunkt weiterleiten |
| [Auswahl einer übergeordneten Instanz](../leader-election.md) | Die von einer Sammlung zusammenarbeitender Aufgabeninstanzen ausgeführten Aktionen in einer verteilten Anwendung koordinieren, indem eine Instanz als übergeordnete Instanz ausgewählt wird, die die Verantwortung für die Verwaltung der anderen Instanzen übernimmt |
| [Pipes und Filter](../pipes-and-filters.md) | Eine Aufgabe, die komplexe Verarbeitungsvorgänge ausführt, in eine Reihe wiederverwendbarer separater Elemente unterteilen |
| [Sidecar](../sidecar.md) | Komponenten einer Anwendung zwecks Isolation und Kapselung in einem separaten Prozess oder Container bereitstellen |
| [Hosten von statischen Inhalten](../static-content-hosting.md) | Statische Inhalte in einem cloudbasierten Speicherdienst bereitstellen, der die Inhalte direkt an den Client übermitteln kann |
| [Einschnürung](../strangler.md) | Ein älteres System inkrementell migrieren, indem bestimmte Teile der Funktionalität nach und nach durch neue Anwendungen und Dienste ersetzt werden |