---
title: Datenverwaltungsmuster
description: "Die Datenverwaltung ist das wichtigste Element von Cloudanwendungen und wirkt sich auf die meisten Qualitätsattribute aus. Meist werden Daten aus Gründen der Leistung, Skalierbarkeit oder Verfügbarkeit an verschiedenen Speicherorten und auf mehreren Servern gehostet, und dies kann eine ganze Reihe von Herausforderungen mit sich bringen. Beispielsweise muss die Konsistenz der Daten aufrechterhalten werden, und normalerweise müssen Daten zwischen unterschiedlichen Speicherorten synchronisiert werden."
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: a009a06268f114ab7be4544dd81710612dabd8f4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="data-management-patterns"></a>Datenverwaltungsmuster

[!INCLUDE [header](../../_includes/header.md)]

Die Datenverwaltung ist das wichtigste Element von Cloudanwendungen und wirkt sich auf die meisten Qualitätsattribute aus. Meist werden Daten aus Gründen der Leistung, Skalierbarkeit oder Verfügbarkeit an verschiedenen Speicherorten und auf mehreren Servern gehostet, und dies kann eine ganze Reihe von Herausforderungen mit sich bringen. Beispielsweise muss die Konsistenz der Daten aufrechterhalten werden, und normalerweise müssen Daten zwischen unterschiedlichen Speicherorten synchronisiert werden.

| Muster | Zusammenfassung |
| ------- | ------- |
| [Cache-Aside](../cache-aside.md) | Daten bei Bedarf in einen Cache aus einem Datenspeicher laden |
| [Befehlsabfrage-Zuständigkeitstrennung (Command Query Responsibility Segregation, CQRS)](../cqrs.md) | Mithilfe separater Schnittstellen Vorgänge trennen, die Daten von Vorgängen zur Aktualisierung von Daten lesen |
| [Ereignisherkunftsermittlung](../event-sourcing.md) | Einen nur zum Anfügen vorgesehenen Speicher verwenden, um die vollständige Serie von Ereignissen aufzuzeichnen, die an Daten in einer Domäne ausgeführte Aktionen beschreiben |
| [Indextabelle](../index-table.md) | Indizes für die Felder im Datenspeicher erstellen, auf die häufig von Abfragen verwiesen wird |
| [Materialisierte Sicht](../materialized-view.md) | Voraufgefüllte Sichten für die Daten in einem oder mehreren Datenspeichern generieren, wenn die Daten für erforderliche Abfragevorgänge nicht ideal formatiert sind |
| [Sharding](../sharding.md) | Einen Datenspeicher in einen Satz horizontaler Partitionen oder Shards unterteilen |
| [Hosten von statischen Inhalten](../static-content-hosting.md) | Statische Inhalte in einem cloudbasierten Speicherdienst bereitstellen, der die Inhalte direkt an den Client übermitteln kann |
| [Valet-Schlüssel](../valet-key.md) | Ein Token oder einen Schlüssel verwenden, das bzw. der Clients eingeschränkten direkten Zugriff auf eine bestimmte Ressource oder einen bestimmten Dienst bietet |