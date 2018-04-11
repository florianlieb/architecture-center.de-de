---
title: Entwurfsprinzipien für Azure-Anwendungen
description: Entwurfsprinzipien für Azure-Anwendungen
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 57b04839e14804ad97fc9c86e1f9c4fe6e0da472
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="design-principles-for-azure-applications"></a>Entwurfsprinzipien für Azure-Anwendungen

Befolgen Sie die folgenden Entwurfsprinzipien, um die Skalierbarkeit, Resilienz und Verwaltbarkeit Ihrer Anwendung zu optimieren. 

**[Entwurf mit Blick auf Selbstreparatur](self-healing.md)**: In einem verteilten System kann es zu Ausfällen kommen. Entwerfen Sie Ihre Anwendung so, dass sie bei Ausfällen eine Selbstreparatur durchführt.

**[Herstellen von Redundanz für alle Anwendungskomponenten](redundancy.md)**: Schaffen Sie Redundanz in Ihrer Anwendung, um Ausfälle einzelner Komponenten zu verhindern.
 
**[Minimieren der Koordinierung](minimize-coordination.md)**: Minimieren Sie die Koordinierung zwischen Anwendungsdiensten, um Skalierbarkeit zu erzielen.
 
**[Ausrichtung des Entwurfs auf horizontale Skalierung](scale-out.md)**: Entwerfen Sie Ihre Anwendung so, dass sie durch Hinzufügen oder Entfernen neuer Instanzen je nach Bedarf horizontal skaliert werden kann.

**[Umgehung von Beschränkungen durch Partitionierung](partition.md)**: Setzen Sie Partitionierung ein, um Datenbank-, Netzwerk- und Computebeschränkungen zu umgehen.

**[Entwurf mit Blick auf den Betrieb](design-for-operations.md)**: Setzen Sie es sich beim Entwurf Ihre Anwendung als Ziel, dem Betriebsteam die benötigten Verwaltungstools zur Verfügung zu stellen.

**[Verwendung verwalteter Dienste](managed-services.md)**: Verwenden Sie nach Möglichkeit Platform as a Service (PaaS) statt Infrastructure-as-a-Service (IaaS).

**[Verwendung des idealen Datenspeichers](use-the-best-data-store.md)**: Wählen Sie die Speichertechnologie, die am besten für Ihre Daten und den vorgesehenen Einsatzzweck geeignet ist. 
 
**[Entwurf mit Blick auf die Entwicklung](design-for-evolution.md)**: Erfolgreiche Anwendungen ändern sich im Laufe der Zeit. Ein evolutionärer Entwurf ist für einen kontinuierlichen Innovationsstrom von entscheidender Bedeutung.

**[Ausrichtung des Entwurfs auf die Unternehmensanforderungen](build-for-business.md)**: Jede Entwurfsentscheidung muss durch eine geschäftliche Anforderung gerechtfertigt sein.

