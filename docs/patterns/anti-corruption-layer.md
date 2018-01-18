---
title: "Muster „Antibeschädigungsebene“"
description: "Es wird beschrieben, wie Sie eine „Fassade“ oder Adapterebene zwischen einer modernen Anwendung und einem älteren System implementieren."
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: e41f080abbef772596ee7f8b10ad72bb03a3b829
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/05/2018
---
# <a name="anti-corruption-layer-pattern"></a>Muster „Antibeschädigungsebene“

Implementieren Sie eine „Fassade“ oder Adapterebene zwischen einer modernen Anwendung und einem älteren System, von dem die Anwendung abhängig ist. Auf dieser Ebene werden Anforderungen zwischen der modernen Anwendung und dem älteren System übersetzt. Verwenden Sie dieses Muster, um sicherzustellen, dass der Entwurf einer Anwendung nicht durch Abhängigkeiten von älteren Systemen eingeschränkt wird. Dieses Muster wurde erstmals von Eric Evans in *Domain-Driven Design* beschrieben.

## <a name="context-and-problem"></a>Kontext und Problem

Die meisten Anwendungen basieren in Bezug auf einige Daten oder Funktionen auf anderen Systemen. Wenn eine ältere Anwendung beispielsweise zu einem modernen System migriert wird, werden dafür ggf. weiterhin vorhandene ältere Ressourcen benötigt. Für neue Features muss das Aufrufen des älteren Systems möglich sein. Dies gilt besonders für Migrationen, die schrittweise erfolgen und bei denen verschiedene Features einer größeren Anwendung im Laufe der Zeit auf ein modernes System verschoben werden.

Für diese älteren Systeme treten häufig Qualitätsprobleme auf, z.B. aufgrund von verworrenen Datenschemas oder veralteten APIs. Die Features und Technologien, die in älteren Systemen eingesetzt werden, können sich von den Komponenten moderner Systeme stark unterscheiden. Zur Sicherstellung der Interoperabilität mit dem älteren System muss die neue Anwendung ggf. veraltete Infrastruktur, Protokolle, Datenmodelle, APIs oder andere Features unterstützen, die Sie andernfalls nicht in eine moderne Anwendung integrieren würden.

Das Ermöglichen des Zugriffs zwischen neuen und alten Systemen kann für das neue System bedeuten, dass zumindest die Anforderungen einiger APIs oder anderer Semantiken des älteren Systems erfüllt werden müssen. Wenn für diese älteren Features Qualitätsprobleme bestehen, kann deren Unterstützung zu einer Beschädigung der modernen Anwendung führen, die ansonsten sauber entworfen wurde. 

## <a name="solution"></a>Lösung

Isolieren Sie ältere und moderne Systeme voneinander, indem Sie dazwischen eine Antibeschädigungsebene anordnen. Auf dieser Ebene wird die Kommunikation zwischen den beiden Systemen übersetzt, damit das ältere System unverändert bleiben kann und der entwurfsbezogene und technologische Ansatz der modernen Anwendung nicht kompromittiert wird.

![](./_images/anti-corruption-layer.png) 

Für die Kommunikation zwischen der modernen Anwendung und der Antibeschädigungsebene werden immer das Datenmodell und die Architektur der Anwendung genutzt. Aufrufe des älteren Systems von der Antibeschädigungsebene aus entsprechen den Datenmodellen bzw. Methoden dieses Systems. Die Antibeschädigungsebene enthält die gesamte Logik, die für die Übersetzung zwischen den beiden Systemen erforderlich ist. Die Ebene kann als Komponente zwischen der Anwendung oder als unabhängiger Dienst implementiert werden.

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

- Die Antibeschädigungsebene kann die Wartezeit für Aufrufe erhöhen, die zwischen den beiden Systemen erfolgen.
- Mit der Antibeschädigungsebene wird ein zusätzlicher Dienst hinzugefügt, der verwaltet und gewartet werden muss.
- Überlegen Sie sich, wie Ihre Antibeschädigungsebene skaliert wird.
- Ermitteln Sie auch, ob Sie mehr als eine Antibeschädigungsebene benötigen. Es kann ratsam sein, die Funktionalität mit verschiedenen Technologien oder Sprachen in mehrere Dienste zu unterteilen, oder es kann andere Gründe für die Partitionierung der Antibeschädigungsebene geben.
- Berücksichtigen Sie, wie die Antibeschädigungsebene in Verbindung mit anderen Anwendungen und Diensten verwaltet werden soll. Wie wird die Ebene in Ihre Überwachungs-, Release- und Konfigurationsprozesse integriert?
- Stellen Sie sicher, dass die Transaktions- und Datenkonsistenz gewahrt ist und überwacht werden kann.
- Überlegen Sie, ob über die Antibeschädigungsebene die gesamte Kommunikation zwischen älteren und modernen Systemen oder nur eine Teilmenge der Features verarbeitet werden muss. 
- Überlegen Sie, ob die Antibeschädigungsebene dauerhaft vorhanden sein muss oder ob sie irgendwann beseitigt werden kann, nachdem alle älteren Funktionen migriert wurden.

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

Verwenden Sie dieses Muster in folgenden Fällen:

- Für eine Migration ist die Durchführung in mehreren Phasen geplant, aber die Integration zwischen neuen und älteren Systemen muss aufrechterhalten werden.
- Das neue und das ältere System verfügen über eine unterschiedliche Semantik, müssen aber trotzdem kommunizieren.

Dieses Muster ist ggf. ungeeignet, wenn zwischen neuem und altem System keine semantischen Unterschiede bestehen. 

## <a name="related-guidance"></a>Verwandte Leitfäden

- [Strangler-Muster][strangler]

[strangler]: ./strangler.md
