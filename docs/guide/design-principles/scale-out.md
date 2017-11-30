---
title: Ausrichtung des Entwurfs auf horizontale Skalierung
description: Cloudanwendungen sollten mit Blick auf die horizontale Skalierung entworfen werden.
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 8f9b3e99a53f5941f708b0de124f37e6ff7e5ab2
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="design-to-scale-out"></a>Ausrichtung des Entwurfs auf horizontale Skalierung

## <a name="design-your-application-so-that-it-can-scale-horizontally"></a>Entwerfen der Anwendung mit Blick auf die horizontale Skalierung

Einer der Hauptvorteile der Cloud ist die elastische Skalierbarkeit: Sie können so viel Kapazität nutzen, wie Sie benötigen. Wenn die Auslastung steigt, können Sie das System horizontal hochskalieren und es wieder herunterskalieren, wenn die zusätzliche Kapazität nicht mehr benötigt wird. Entwerfen Sie Ihre Anwendung so, dass sie durch Hinzufügen oder Entfernen von Instanzen ganz nach Bedarf horizontal skaliert werden kann.

## <a name="recommendations"></a>Recommendations

**Vermeiden Sie Instanzbindung**. Bindung oder *Sitzungsaffinität* tritt auf, wenn Anforderungen desselben Clients immer an den gleichen Server weitergeleitet werden. Dies beeinträchtigt die Fähigkeit einer Anwendung, sich horizontal hochskalieren zu lassen. Beispielsweise wird der Datenverkehr eines Benutzers mit hohem Datenvolumen nicht auf mehrere Instanzen verteilt. Gründe für die Bindung können das Speichern des Sitzungszustands im Arbeitsspeicher und die Verwendung von bestimmten Schlüsseln für die Verschlüsselung sein. Stellen Sie sicher, dass jede Instanz jede Anforderung verarbeiten kann. 

**Identifizieren Sie Engpässe**. Die horizontale Skalierung ist kein Wundermittel für jedes Leistungsproblem. Wenn beispielsweise Ihre Back-End-Datenbank der Engpass ist, hilft das Hinzufügen weiterer Webserver nicht. Identifizieren und beheben Sie zuerst die Engpässe im System, bevor Sie weitere Instanzen einrichten, um das Problem zu lösen. Zustandsbehaftete Teile des Systems sind die wahrscheinlichste Ursache von Engpässen. 

**Teilen Sie Workloads je nach Skalierbarkeitsanforderungen auf.**  Anwendungen bestehen häufig aus mehreren Workloads, die jeweils unterschiedliche Anforderungen an die Skalierung aufweisen. Eine Anwendung verfügt möglicherweise über eine öffentliche Site und eine separate Site für die Verwaltung. Auf der öffentlichen Site treten möglicherweise plötzliche Datenverkehrs-Lastspitzen auf, während die Verwaltungssite eine kleinere und besser vorhersehbare Last aufweist. 

**Lagern Sie ressourcenintensive Tasks aus.** Tasks, die große Mengen an CPU- oder E/A-Ressourcen benötigen, sollten nach Möglichkeit in [Hintergrundaufträge][background-jobs] verschoben werden, um die Last auf dem Front-End, das die Benutzeranforderungen verarbeitet, zu minimieren.

**Verwenden Sie integrierte Features für die automatische Skalierung**. Viele Azure-Computedienste bieten integrierte Unterstützung für die automatische Skalierung. Wenn eine Anwendung eine vorhersehbare, regelmäßige Workload aufweist, sollte das horizontale Hochskalieren gemäß einem Zeitplan erfolgen. Skalieren Sie das System z.B. während der Geschäftszeiten hoch. Wenn die Workload nicht vorhersehbar ist, verwenden Sie Leistungsmetriken wie die CPU-Auslastung oder die Länge der Anforderungswarteschlange, um die automatische Skalierung auszulösen. Informationen zu den bewährten Methoden finden Sie unter [Automatische Skalierung][autoscaling].

**Ziehen Sie für kritische Workloads eine aggressive automatische Skalierung in Betracht**. Bei kritischen Workloads müssen Sie der Nachfrage immer einen Schritt voraus sein. Es ist besser, bei hoher Auslastung schnell neue Instanzen hinzuzufügen, um den zusätzlichen Datenverkehr zu bewältigen, und das System dann nach und nach wieder herunterzuskalieren.

**Konzipieren Sie Ihr System so, dass es horizontal herunterskaliert werden kann**.  Denken Sie daran, dass es bei elastischer Skalierung Zeiträume geben wird, in denen die Anwendung horizontal herunterskaliert wird, indem Instanzen entfernt werden. Die Anwendung muss die Entfernung von Instanzen ordnungsgemäß verarbeiten können. Nachfolgenden finden Sie einige Möglichkeiten, die Anwendung für ein horizontales Herunterskalieren zu konfigurieren:

- Die Anwendung sollte auf „Herunterfahren“-Ereignisse lauschen (wenn verfügbar) und das Herunterfahren ordnungsgemäß ausführen. 
- Clients/Consumer eines Diensts sollten die Behandlung vorübergehender Fehler und Wiederholungsversuche unterstützen. 
- Ziehen Sie bei Tasks mit langer Ausführungsdauer in Betracht, die Arbeit aufzuteilen, indem Sie Prüfpunkte oder das Muster [Pipes und Filter][pipes-filters-pattern] verwenden. 
- Platzieren Sie Arbeitselemente in einer Warteschlange, sodass eine andere Instanz die Verarbeitung aufnehmen kann, wenn eine Instanz mitten in der Verarbeitung entfernt wird. 


<!-- links -->

[autoscaling]: ../../best-practices/auto-scaling.md
[background-jobs]: ../../best-practices/background-jobs.md
[pipes-filters-pattern]: ../../patterns/pipes-and-filters.md
