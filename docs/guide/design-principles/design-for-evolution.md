---
title: Ausrichtung des Entwurfs auf Änderungen
description: Ein evolutionärer Entwurf ist für einen kontinuierlichen Innovationsstrom von entscheidender Bedeutung.
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 35e91228f3fb0a303594ec06f05b6865008e3a4f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="design-for-evolution"></a>Ausrichtung des Entwurfs auf die weitere Entwicklung

## <a name="an-evolutionary-design-is-key-for-continuous-innovation"></a>Ein evolutionärer Entwurf ist für einen kontinuierlichen Innovationsstrom von entscheidender Bedeutung

Alle erfolgreichen Anwendungen verändern sich im Laufe der Zeit, z.B. in Bezug auf das Beheben von Fehlern, Hinzufügen von neuen Features, Einbinden neuer Technologie oder das Erhöhen der Skalierbarkeit und Resilienz von vorhandenen Systemen. Wenn alle Teile einer Anwendung eng miteinander verknüpft sind, ist es sehr schwierig, Änderungen im System vorzunehmen. Eine Änderung in einem Teil der Anwendung kann zu einer Beschädigung eines anderen Teils führen oder bewirken, dass Änderungen für die gesamte Codebase erforderlich sind.

Dieses Problem ist nicht auf monolithische Anwendungen beschränkt. Eine Anwendung kann in Dienste unterteilt werden, aber trotzdem über eine enge Verzahnung verfügen, die bewirkt, dass das System starr und unflexibel ist. Wenn der Entwurfsvorgang von Diensten aber auf die Weiterentwicklung ausgerichtet ist, können Teams innovativ arbeiten und ständig neue Features bereitstellen. 

Microservices werden immer häufiger eingesetzt, um ein evolutionäres Design zu erzielen, da so viele der hier aufgeführten Aspekte abgedeckt werden können.

## <a name="recommendations"></a>Empfehlungen

**Sorgen Sie für eine hohe Kohäsion und eine lose Kopplung**. Ein Dienst verfügt über eine hohe *Kohäsion*, wenn darüber Funktionen bereitgestellt werden, die logisch zusammengehören. Dienste sind *lose gekoppelt*, wenn Sie einen Dienst ändern können, ohne den anderen Dienst ändern zu müssen. Hohe Kohäsion bedeutet normalerweise, dass für Änderungen einer Funktion Änderungen in anderen verwandten Funktionen erforderlich sind. Wenn es sich herausstellt, dass für eine Aktualisierung eines Diensts koordinierte Aktualisierungen anderer Dienste durchgeführt werden müssen, kann dies darauf hindeuten, dass Ihre Dienste keine hohe Kohäsion aufweisen. Ein Ziel eines am Geschäftsbereich ausgerichteten Entwurfs (Domain-Driven Design, DDD) ist die Identifizierung dieser Grenzen.

**Kapseln Sie Domänenwissen**. Wenn ein Client einen Dienst nutzt, sollte nicht der Client dafür verantwortlich sein, die Geschäftsregeln der Domäne durchzusetzen. Stattdessen sollte der Dienst das gesamte Domänenwissen kapseln, das in dessen Zuständigkeitsbereich fällt. Andernfalls muss jeder Client die Geschäftsregeln durchsetzen, und das Domänenwissen ist bei Ihnen auf unterschiedliche Teile der Anwendung verteilt. 

**Verwenden Sie asynchrone Nachrichten**. Asynchrone Nachrichten sind eine Möglichkeit, den Nachrichtenproducer vom Consumer zu entkoppeln. Der Producer ist nicht vom Consumer abhängig, was das Beantworten der Nachricht oder das Durchführen einer bestimmten Aktion betrifft. Bei einer Pub/Sub-Architektur ist der Producer ggf. gar nicht darüber informiert, von welchem Consumer die Nachricht genutzt wird. Neue Dienste können die Nachrichten ohne Änderungen des Producers leicht nutzen.

**Integrieren Sie Domänenwissen nicht in ein Gateway**. Gateways können in einer Microservices-Architektur nützlich sein, z.B. für das Anforderungsrouting, die Protokollübersetzung, den Lastenausgleich oder die Authentifizierung. Das Gateway sollte aber auf diese Art von Infrastrukturfunktionen beschränkt sein. Dafür sollte kein Domänenwissen implementiert werden, um zu verhindern, dass starke Abhängigkeiten bestehen.

**Machen Sie offene Schnittstellen verfügbar**. Vermeiden Sie die Erstellung von benutzerdefinierten Übersetzungsebenen, die zwischen Diensten angeordnet sind. Stattdessen sollte ein Dienst eine API mit einem gut definierten API-Vertrag verfügbar machen. Die API sollte mit einer Versionsangabe versehen sein, damit Sie die API weiterentwickeln können, während die Abwärtskompatibilität aufrechterhalten wird. Auf diese Weise können Sie einen Dienst aktualisieren, ohne Updates für alle Upstream-Dienste zu koordinieren, die davon abhängig sind. Öffentliche Dienste sollten eine RESTful-API per HTTP verfügbar machen. Für Back-End-Dienste wird aus Leistungsgründen ggf. ein Messagingprotokoll im RPC-Stil verwendet. 

**Führen Sie den Entwurf und das Testen anhand von Dienstverträgen durch**. Wenn Dienste gut definierte APIs verfügbar machen, können Sie basierend auf diesen APIs entwickeln und testen. So können Sie einen einzelnen Dienst entwickeln und testen, ohne alle davon abhängigen Dienste bereitzustellen. (Die Integration und Auslastungstests führen Sie natürlich weiterhin für die echten Dienste durch.)

**Abstrahieren Sie die Infrastruktur weg von der Domänenlogik**. Verhindern Sie, dass die Domänenlogik mit den infrastrukturbezogenen Funktionen vermischt wird, z.B. Messaging oder Persistenz. Andernfalls sind für Änderungen der Domänenlogik Updates der Infrastrukturebenen erforderlich (und umgekehrt). 

**Lagern Sie Funktionen in einen separaten Dienst aus, falls Probleme durch Überschneidungen zu erwarten sind**. Wenn mehrere Dienste beispielsweise eine Authentifizierung von Anforderungen durchführen müssen, können Sie diese Funktion in einen eigenen Dienst verschieben. Anschließend können Sie den Authentifizierungsdienst weiterentwickeln – indem Sie beispielsweise einen neuen Authentifizierungsfluss hinzufügen –, ohne Änderungen in anderen Diensten vornehmen zu müssen, die diesen Dienst nutzen.

**Stellen Sie Dienste unabhängig voneinander bereit**. Wenn das DevOps-Team einen einzelnen Dienst unabhängig von anderen Diensten in der Anwendung bereitstellen kann, können Updates schneller und sicherer durchgeführt werden. Für Fehlerbehebungen und neue Features kann der Rollout in regelmäßigeren Abständen erfolgen. Entwerfen Sie sowohl die Anwendung als auch den Releaseprozess, um unabhängige Updates zu unterstützen.