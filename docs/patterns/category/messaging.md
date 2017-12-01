---
title: Nachrichtenmuster
description: "Die verteilte Struktur von Cloudanwendungen erfordert eine Messaginginfrastruktur, die die Komponenten und Dienste – idealerweise lose gekoppelt – miteinander verbindet, um die Skalierbarkeit zu maximieren. Asynchrones Messaging ist weit verbreitet und bietet viele Vorteile, beinhaltet aber auch Herausforderungen, beispielsweise die Reihenfolge von Nachrichten, die Verwaltung von nicht verarbeitbaren Nachrichten, Idempotenz und vieles mehr."
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 6151f7f76fc7b3a953988122db75bdc25b49811f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="messaging-patterns"></a>Nachrichtenmuster

[!INCLUDE [header](../../_includes/header.md)]

Die verteilte Struktur von Cloudanwendungen erfordert eine Messaginginfrastruktur, die die Komponenten und Dienste – idealerweise lose gekoppelt – miteinander verbindet, um die Skalierbarkeit zu maximieren. Asynchrones Messaging ist weit verbreitet und bietet viele Vorteile, beinhaltet aber auch Herausforderungen, beispielsweise die Reihenfolge von Nachrichten, die Verwaltung von nicht verarbeitbaren Nachrichten, Idempotenz und vieles mehr.

| Muster | Zusammenfassung |
| ------- | ------- |
| [Konkurrierende Consumer](../competing-consumers.md) | Mehreren gleichzeitigen Consumern die Verarbeitung von Nachrichten ermöglichen, die auf dem gleichen Messagingkanal empfangen werden |
| [Pipes und Filter](../pipes-and-filters.md) | Eine Aufgabe, die komplexe Verarbeitungsvorgänge ausführt, in eine Reihe wiederverwendbarer separater Elemente unterteilen |
| [Prioritätswarteschlange](../priority-queue.md) | An Dienste gesendete Anforderungen priorisieren, sodass Anforderungen mit einer höheren Priorität schneller empfangen und verarbeitet werden als Anforderungen mit einer niedrigeren Priorität |
| [Warteschlangenbasierter Lastenausgleich](../queue-based-load-leveling.md) | Eine Warteschlange verwenden, die als Puffer zwischen einer Aufgabe und einem von ihr aufgerufenen Dienst fungiert, um unregelmäßig auftretende hohe Lasten aufzufangen |
| [Scheduler-Agent-Supervisor](../scheduler-agent-supervisor.md) | Eine Reihe von Aktionen in einer verteilten Gruppe von Diensten und anderen Remoteressourcen koordinieren |