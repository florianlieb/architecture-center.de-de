---
title: Muster „Antibeschädigungsebene“
description: Es wird beschrieben, wie Sie eine „Fassade“ oder Adapterebene zwischen einer modernen Anwendung und einem älteren System implementieren.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: ac898519c9aa0a0aa2301da9f48756db0eb2af7c
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/16/2018
---
# <a name="anti-corruption-layer-pattern"></a>Muster „Antibeschädigungsebene“

Implementieren Sie eine Fassade oder Adapterebene zwischen unterschiedlichen Subsystemen ohne gemeinsame Semantik. Diese Ebene dient zum Übersetzen der Anforderungen, die ein Subsystem an ein anderes Subsystem richtet. Verwenden Sie dieses Muster, um sicherzustellen, dass der Entwurf einer Anwendung nicht durch Abhängigkeiten von externen Systemen eingeschränkt wird. Dieses Muster wurde erstmals von Eric Evans in *Domain-Driven Design* beschrieben.

## <a name="context-and-problem"></a>Kontext und Problem

Die meisten Anwendungen basieren in Bezug auf einige Daten oder Funktionen auf anderen Systemen. Wenn eine ältere Anwendung beispielsweise zu einem modernen System migriert wird, werden dafür ggf. weiterhin vorhandene ältere Ressourcen benötigt. Für neue Features muss das Aufrufen des älteren Systems möglich sein. Dies gilt besonders für Migrationen, die schrittweise erfolgen und bei denen verschiedene Features einer größeren Anwendung im Laufe der Zeit auf ein modernes System verschoben werden.

Für diese älteren Systeme treten häufig Qualitätsprobleme auf, z.B. aufgrund von verworrenen Datenschemas oder veralteten APIs. Die Features und Technologien, die in älteren Systemen eingesetzt werden, können sich von den Komponenten moderner Systeme stark unterscheiden. Zur Sicherstellung der Interoperabilität mit dem älteren System muss die neue Anwendung ggf. veraltete Infrastruktur, Protokolle, Datenmodelle, APIs oder andere Features unterstützen, die Sie andernfalls nicht in eine moderne Anwendung integrieren würden.

Das Ermöglichen des Zugriffs zwischen neuen und alten Systemen kann für das neue System bedeuten, dass zumindest die Anforderungen einiger APIs oder anderer Semantiken des älteren Systems erfüllt werden müssen. Wenn für diese älteren Features Qualitätsprobleme bestehen, kann deren Unterstützung zu einer Beschädigung der modernen Anwendung führen, die ansonsten sauber entworfen wurde. 

Ähnliche Probleme können durch jedes externe System verursacht werden, das nicht von Ihrem Entwicklerteam kontrolliert wird (nicht nur durch Legacysysteme). 

## <a name="solution"></a>Lösung

Platzieren Sie zur Isolierung eine Antibeschädigungsebene zwischen unterschiedlichen Systemen. Diese Ebene übersetzt die Kommunikation zwischen den beiden Systemen, sodass eines der Systeme unverändert bleiben kann und das Design sowie das technologische Konzept des anderen Systems nicht kompromittiert wird.

![](./_images/anti-corruption-layer.png) 

Das obige Diagramm zeigt eine Anwendung mit zwei Subsystemen. Aufrufe, die von Subsystem A an Subsystem B gerichtet werden, durchlaufen eine Antibeschädigungsebene. Bei der Kommunikation zwischen Subsystem A und der Antibeschädigungsebene werden immer das Datenmodell und die Architektur des Subsystems A verwendet. Aufrufe, die von der Antibeschädigungsebene an Subsystem B gerichtet werden, entsprechen dagegen dem Datenmodell bzw. den Methoden dieses Subsystems. Die Antibeschädigungsebene enthält die gesamte Logik, die für die Übersetzung zwischen den beiden Systemen erforderlich ist. Die Ebene kann als Komponente zwischen der Anwendung oder als unabhängiger Dienst implementiert werden.

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

- Die Antibeschädigungsebene kann die Wartezeit für Aufrufe erhöhen, die zwischen den beiden Systemen erfolgen.
- Mit der Antibeschädigungsebene wird ein zusätzlicher Dienst hinzugefügt, der verwaltet und gewartet werden muss.
- Überlegen Sie sich, wie Ihre Antibeschädigungsebene skaliert wird.
- Ermitteln Sie auch, ob Sie mehr als eine Antibeschädigungsebene benötigen. Es kann ratsam sein, die Funktionalität mit verschiedenen Technologien oder Sprachen in mehrere Dienste zu unterteilen, oder es kann andere Gründe für die Partitionierung der Antibeschädigungsebene geben.
- Berücksichtigen Sie, wie die Antibeschädigungsebene in Verbindung mit anderen Anwendungen und Diensten verwaltet werden soll. Wie wird die Ebene in Ihre Überwachungs-, Release- und Konfigurationsprozesse integriert?
- Stellen Sie sicher, dass die Transaktions- und Datenkonsistenz gewahrt ist und überwacht werden kann.
- Überlegen Sie, ob die Antibeschädigungsebene die gesamte Kommunikation zwischen unterschiedlichen Systemen oder nur die Kommunikation bestimmter Features verarbeiten muss. 
- Wenn die Antibeschädigungsebene Teil einer Anwendungsmigrationsstrategie ist, überlegen Sie sich, ob es sich dabei um eine dauerhafte Lösung handelt oder ob sie nach der Migration sämtlicher Legacyfunktionen nicht mehr benötigt wird.

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

Verwenden Sie dieses Muster in folgenden Fällen:

- Für eine Migration ist die Durchführung in mehreren Phasen geplant, aber die Integration zwischen neuen und älteren Systemen muss aufrechterhalten werden.
- Zwei oder mehr Subsysteme müssen trotz unterschiedlicher Semantik miteinander kommunizieren. 

Dieses Muster ist ggf. ungeeignet, wenn zwischen neuem und altem System keine semantischen Unterschiede bestehen. 

## <a name="related-guidance"></a>Verwandte Leitfäden

- [Einschnürungsmuster](./strangler.md)
