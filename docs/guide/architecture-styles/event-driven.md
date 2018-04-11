---
title: Ereignisgesteuerter Architekturstil
description: Dieser Artikel beschreibt die Vorteile, Herausforderungen und bewährten Methoden für ereignisgesteuerte und IoT-Architekturen in Azure.
author: MikeWasson
ms.openlocfilehash: 3289bf784b02d62e3d0c1a29b4839c9be3501134
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/23/2018
---
# <a name="event-driven-architecture-style"></a>Ereignisgesteuerter Architekturstil

Eine ereignisgesteuerte Architektur besteht aus **Ereignisproducern**, die einen Ereignisstrom generieren, und **Ereignisconsumern**, die auf die Ereignisse lauschen. 

![](./images/event-driven.svg)

Ereignisse werden nahezu in Echtzeit übermittelt, sodass Consumer unmittelbar nach dem Auftreten des Ereignisses reagieren können. Producer sind von Consumern entkoppelt – ein Producer weiß nicht, welche Consumer lauschen. Consumer sind auch voneinander entkoppelt, und jeder Consumer sieht alle Ereignisse. Dies ist ein Unterschied zum Muster der [konkurrierenden Consumer][competing-consumers], in dem Consumer Nachrichten aus einer Warteschlange abrufen und eine Nachricht nur einmal verarbeitet wird (vorausgesetzt, dass sie fehlerfrei ist). In einigen Systemen, z.B. dem Internet der Dinge (Internet of Things, IoT), müssen Ereignisse in sehr großen Mengen erfasst werden.

Eine ereignisgesteuerte Architektur kann ein Veröffentlichungs-/Abonnementmodell oder eine Ereignisstrommodell verwenden. 

- **Veröffentlichung/Abonnement**: Die Messaginginfrastruktur verfolgt die Abonnements nach. Wenn ein Ereignis veröffentlicht wird, sendet die Infrastruktur das Ereignis an jeden Abonnenten. Nachdem ein Ereignis empfangen wurde, kann es nicht wiedergegeben werden, und für neue Abonnenten wird es nicht angezeigt. 

- **Ereignisströme**: Ereignisse werden in ein Protokoll geschrieben. Ereignisse unterliegen einer strikten Reihenfolge (innerhalb einer Partition) und sind dauerhaft. Clients abonnieren den Ereignisstrom nicht, stattdessen kann ein Client in jedem Teil des Ereignisstroms Lesevorgänge ausführen. Der Client ist dafür zuständig, im Ereignisstrom vorzurücken. Das bedeutet, dass ein Client jederzeit beitreten und Ereignisse wiedergeben kann.

Auf Consumerseite gibt es einige gängige Variationen:

- **Einfache Ereignisverarbeitung**. Ein Ereignis löst unmittelbar eine Aktion im Consumer aus. Sie können z.B. Azure Functions mit einem Service Bus-Trigger verwenden, sodass eine Funktion ausgeführt wird, sobald eine Nachricht in einem Service Bus-Thema veröffentlicht wurde.

- **Komplexe Ereignisverarbeitung**. Ein Consumer verarbeitet eine Reihe von Ereignissen und sucht dabei mithilfe von Technologien wie Azure Stream Analytics oder Apache Storm nach Mustern in den Ereignisdaten. Sie können z.B. Messergebnisse eines eingebetteten Geräts für ein bestimmtes Zeitfenster aggregieren und eine Nachricht generieren, wenn der gleitende Durchschnitt einen bestimmten Schwellenwert überschreitet. 

- **Ereignisstromverarbeitung**. Verwenden Sie eine Plattform zur Verarbeitung von Datenströmen wie z.B. Azure IoT Hub oder Apache Kafka als Pipeline zum Erfassen von Ereignissen und Weiterleiten der Ereignisse an Datenstromprozessoren. Die Datenstromprozessoren verarbeiten oder transformieren den Datenstrom. Es können mehrere Datenstromprozessoren für verschiedene Subsysteme der Anwendung vorhanden sein. Dieser Ansatz eignet sich gut für IoT-Workloads.

Die Quelle der Ereignisse befindet sich möglicherweise außerhalb des Systems – z.B. physische Geräte in einer IoT-Lösung. In diesem Fall muss das System in der Lage sein, die Daten mit der Kapazität hinsichtlich Umfang und Durchsatz zu erfassen, die für die Datenquelle erforderlich ist.

In dem logischen Diagramm oben wird jeder Consumertyp als einzelnes Feld angezeigt. In der Praxis werden meist mehrere Instanzen eines Consumers bereitgestellt, damit der Consumer nicht zum Single Point of Failure im System wird. Mehrere Instanzen sind möglicherweise auch erforderlich, um die Menge und Häufigkeit von Ereignissen zu bewältigen. Auch verarbeitet ein einzelner Consumer Ereignisse möglicherweise in mehreren Threads. Das kann eine Herausforderung darstellen, wenn Ereignisse in einer bestimmten Reihenfolge verarbeitet werden müssen oder eine „Exactly-Once“-Semantik erfordern. Weitere Informationen finden Sie unter [Minimieren der Koordination][minimize-coordination]. 

## <a name="when-to-use-this-architecture"></a>Einsatzmöglichkeiten für diese Architektur

- Mehrere Subsysteme, die die gleichen Ereignisse verarbeiten müssen 
- Echtzeitverarbeitung mit minimaler zeitlicher Verzögerung
- Komplexe Ereignisverarbeitung, z.B. Musterabgleich oder Aggregation über bestimmte Zeitfenster hinweg
- Große Mengen und hohe Datengeschwindigkeit, z.B. in IoT-Systemen

## <a name="benefits"></a>Vorteile

- Producer und Consumer sind entkoppelt.
- Keine Punkt-zu-Punkt-Integrationen. Neue Consumer lassen sich problemlos zum System hinzufügen.
- Consumer können sofort auf Ereignisse reagieren, sobald diese eingehen. 
- In hohem Maß skalierbar und verteilt. 
- Subsysteme weisen unabhängige Perspektiven des Ereignisstroms auf.

## <a name="challenges"></a>Herausforderungen

- Garantierte Übermittlung. In manchen Systemen – insbesondere in IoT-Szenarien – ist es von entscheidender Bedeutung, sicherstellen zu können, dass Ereignisse übermittelt werden.
- Verarbeitung der Ereignisse in einer bestimmten Reihenfolge oder genau einmal. Jeder Consumertyp wird aus Gründen der Resilienz und Skalierbarkeit in der Regel in mehreren Instanzen ausgeführt. Dies kann zu einer Herausforderung werden, wenn die Ereignisse (innerhalb eines Consumertyps) in einer bestimmten Reihenfolge verarbeitet werden müssen oder die Verarbeitungslogik nicht idempotent ist.

## <a name="iot-architecture"></a>IoT-Architektur

Ereignisgesteuerte Architekturen sind von zentraler Bedeutung für IoT-Lösungen. Das folgende Diagramm zeigt eine mögliche logische Architektur für IoT. Das Diagramm veranschaulicht die Komponenten der Architektur, die Ereignisströme verarbeiten.

![](./images/iot.png)

Das **Cloudgateway** erfasst Geräteereignisse an der Cloudgrenze mithilfe eines zuverlässigen Messagingsystems mit niedriger Latenz.

Geräte können Ereignisse direkt an das Cloudgateway oder über ein **Bereichsgateway**. Ein Bereichsgateway ist ein spezialisiertes Gerät oder Softwareprogramm, das sich üblicherweise am gleichen Ort befindet wie die Geräte. Es empfängt Ereignisse und leitet sie an das Cloudgateway weiter. Das Bereichsgateway kann auch Vorverarbeitungsfunktionen für die Geräteereignisse ausführen, z.B. Filtern, Aggregation oder Protokolltransformation.

Nach dem Erfassen durchlaufen Ereignisse einen oder mehrere **Datenstromprozessoren**, die die Daten weiterleiten (z.B. zum Speicher) oder Analyse- oder andere Verarbeitungsfunktionen ausführen.

Nachfolgend finden Sie einige häufige Verarbeitungsarten. (Diese Liste ist sicherlich nicht vollständig.)

- Schreiben von Ereignisdaten in einen Cold Storage zur Archivierung oder Batchanalyse.

- Analyse des langsamsten Pfads – der Ereignisstrom wird (nahezu) in Echtzeit analysiert, um Anomalien zu erkennen, Muster in rollierenden Zeitfenstern zu ermitteln oder Warnungen auszulösen, wenn eine bestimmte Bedingung im Ereignisstrom auftritt. 

- Verarbeitung bestimmter Arten von nicht telemetriebezogenen Nachrichten von Geräten, z.B. Benachrichtigungen und Warnungen. 

- Machine Learning.

Die grauen Felder stehen für Komponenten eines IoT-Systems, die nicht in direktem Zusammenhang mit der Ereignisstromverarbeitung stehen, sondern der Vollständigkeit halber hier mit aufgeführt werden.

- Die **Geräteregistrierung** ist eine Datenbank der bereitgestellten Geräte und enthält die Geräte-IDs sowie üblicherweise einige Metadaten zu den Geräten, beispielsweise den Standort.

- Die **Bereitstellungs-API** ist eine allgemeine externe Schnittstelle für die Bereitstellung und Registrierung neuer Geräte.

- In einigen IoT-Lösungen können **Nachrichten mit Befehlen und Steuerungsinformationen** an die Geräte gesendet werden.

> Dieser Abschnitt stellt eine sehr allgemeine Übersicht über IoT dar, und es gibt eine Vielzahl von Feinheiten und Herausforderungen, die es in diesem Bereich zu berücksichtigen gilt. Weitere Informationen und Details finden Sie in der als PDF-Download verfügbaren [Microsoft Azure IoT Reference Architecture][iot-ref-arch] (Referenzarchitektur zu Microsoft Azure IoT).

 <!-- links -->

[competing-consumers]: ../../patterns/competing-consumers.md
[iot-ref-arch]: https://azure.microsoft.com/updates/microsoft-azure-iot-reference-architecture-available/
[minimize-coordination]: ../design-principles/minimize-coordination.md


