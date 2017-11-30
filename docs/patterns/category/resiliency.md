---
title: Resilienzmuster
description: "Die Resilienz ist die Fähigkeit eines Systems, Fehler ordnungsgemäß zu behandeln und nach Fehlern ordnungsgemäß wiederhergestellt werden zu können. Aufgrund des Funktionsprinzips von Cloudhosting, bei dem Anwendungen oft mehrinstanzenfähig sind, gemeinsame Plattformdienste nutzen, um Ressourcen und Bandbreite konkurrieren, über das Internet kommunizieren und auf Standardhardware ausgeführt werden, ist die Wahrscheinlichkeit höher, dass sowohl vorübergehende als auch mehr permanente Fehler auftreten. Um die Resilienz sicherzustellen, sind eine schnelle und effiziente Fehlererkennung sowie Wiederherstellung erforderlich."
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: a3b9d72989e0de57c689bcec51e20653d0441d31
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="resiliency-patterns"></a>Resilienzmuster

Die Resilienz ist die Fähigkeit eines Systems, Fehler ordnungsgemäß zu behandeln und nach Fehlern ordnungsgemäß wiederhergestellt werden zu können. Aufgrund des Funktionsprinzips von Cloudhosting, bei dem Anwendungen oft mehrinstanzenfähig sind, gemeinsame Plattformdienste nutzen, um Ressourcen und Bandbreite konkurrieren, über das Internet kommunizieren und auf Standardhardware ausgeführt werden, ist die Wahrscheinlichkeit höher, dass sowohl vorübergehende als auch mehr permanente Fehler auftreten. Um die Resilienz sicherzustellen, sind eine schnelle und effiziente Fehlererkennung sowie Wiederherstellung erforderlich.

| Muster | Zusammenfassung |
| ------- | ------- |
| [Bulkhead](../bulkhead.md) | Isolieren Sie Elemente einer Anwendung in Pools, sodass bei Ausfall eines Elements die anderen Elemente weiterhin ausgeführt werden. |
| [Sicherung](../circuit-breaker.md) | Behandeln Sie Fehler, deren Behebung beim Herstellen einer Verbindung mit einem Remotedienst oder einer Remoteressource unterschiedlich lange dauern kann. |
| [Ausgleichende Transaktion](../compensating-transaction.md) | Machen Sie durch eine Reihe von Schritten ausgeführte Arbeit rückgängig, die zusammen einen letztlich konsistenten Vorgang definieren. |
| [Überwachung des Integritätsendpunkts](../health-endpoint-monitoring.md) | Implementieren Sie funktionale Prüfungen in einer Anwendung, auf die externe Tools in regelmäßigen Abständen über verfügbar gemachte Endpunkte zugreifen können. |
| [Auswahl einer übergeordneten Instanz](../leader-election.md) | Koordinieren Sie die Aktionen, die von einer Sammlung zusammenarbeitender Taskinstanzen ausgeführt werden, in einer verteilten Anwendung, indem eine Instanz als übergeordnete Instanz ausgewählt wird, die die Verantwortung für die Verwaltung der anderen Instanzen übernimmt. |
| [Warteschlangenbasierter Lastenausgleich](../queue-based-load-leveling.md) | Verwenden Sie eine Warteschlange, die als Puffer zwischen einem Task und einem von diesem aufgerufenen Dienst fungiert, um unregelmäßig auftretende hohe Lasten aufzufangen. |
| [Wiederholung](../retry.md) | Ermöglichen Sie einer Anwendung beim Herstellen einer Verbindung mit einem Dienst oder einer Netzwerkressource die Behandlung antizipierter, temporärer Fehler, indem ein zuvor nicht erfolgreich durchgeführter Vorgang transparent wiederholt wird. |
| [Scheduler-Agent-Supervisor](../scheduler-agent-supervisor.md) | Koordinieren Sie eine Reihe von Aktionen in einer verteilten Gruppe von Diensten und anderen Remoteressourcen. |