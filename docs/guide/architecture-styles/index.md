---
title: Architekturstile
description: Allgemeine Architekturstile für Cloudanwendungen
layout: LandingPage
ms.openlocfilehash: e647d1a0f3305e7754859e5ab8a9a3b46c3d4fb6
ms.sourcegitcommit: d08f6ee27e1e8a623aeee32d298e616bc9bb87ff
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/07/2018
---
# <a name="architecture-styles"></a>Architekturstile

Ein *Architekturstil* ist eine Familie von Architekturen, die bestimmte gemeinsame Eigenschaften aufweisen. Ein gängiger Architekturstil ist beispielsweise eine [Architektur mit n-Schichten][n-tier]. In jüngerer Zeit gewinnen [Microservicearchitekturen][microservices] zunehmend an Popularität. Architekturstile sind nicht auf spezielle Technologien beschränkt, doch einige Technologien eignen sich für bestimmte Architekturen besonders gut. So sind Container beispielsweise ideal für Microservices geeignet.  

Wir haben einen Blick auf eine Reihe von Architekturstilen geworfen, die häufig bei Cloudanwendungen zum Einsatz kommen. Der Artikel zu den einzelnen Stilen enthält Folgendes:

- Eine Beschreibung und ein logisches Diagramm des Stils
- Empfehlungen für die Auswahl des jeweiligen Stils
- Vorteile, Herausforderungen und bewährte Methoden
- Angabe einer empfohlenen Bereitstellung mit relevanten Azure-Diensten


## <a name="a-quick-tour-of-the-styles"></a>Überblick über die Stile   

Dieser Abschnitt enthält einen Überblick über die von uns erläuterten Architekturstile sowie einige allgemeine Überlegungen zu ihrer Verwendung. Weitere Informationen finden Sie in den verknüpften Themen.

### <a name="n-tier"></a>Architektur mit n-Schichten

<img src="./images/n-tier-sketch.svg" style="float:left; margin-top:6px;"/>

Eine **[Architektur mit n-Schichten][n-tier]** ist eine konventionelle Architektur für Unternehmensanwendungen. Abhängigkeiten werden durch Unterteilung der Anwendung in *Schichten* verwaltet, die logische Funktionen ausführen (z.B. Darstellung, Geschäftslogik, Datenzugriff). Eine Schicht kann nur Schichten aufrufen, die ihr untergeordnet sind. Allerdings kann sich diese horizontale Anordnung als Hürde erweisen. Es kann schwierig sein, Änderungen an einem Teil der Anwendung vorzunehmen, ohne den Rest der Anwendung zu ändern. Deshalb können sich häufige Aktualisierungen als umständlich erweisen, was die Möglichkeit zum schnellen Hinzufügen neuer Features einschränken kann.

Die Architektur mit n-Schichten ist hervorragend für die Migration bestehender Anwendungen geeignet, die bereits auf eine Architektur mit Schichten basieren. Aus diesem Grund kommt die Architektur mit n-Schichten bei Infrastructure-as-a-Service-Lösungen (IaaS) oder Anwendungen, die eine Kombination aus IaaS- und verwalteten Diensten umfassen, am häufigsten vor. 

### <a name="web-queue-worker"></a>Web-Warteschlange-Worker

<img src="./images/web-queue-worker-sketch.svg" style="float:left; margin-top:6px;"/>

Für eine reine PaaS-Lösung sollten Sie eine **[Web-Warteschlange-Worker](./web-queue-worker.md)**-Architektur in Betracht ziehen. Bei diesem Stil verfügt die Anwendung über ein Web-Front-End, das HTTP-Anfragen bearbeitet, und einen Back-End-Worker, der CPU-intensive Aufgaben oder zeitintensive Vorgänge ausführt. Das Front-End kommuniziert über eine Warteschlange für asynchrone Nachrichten mit dem Worker. 

Der Web-Warteschlange-Worker-Architektur eignet sich für relativ einfache Domänen mit einigen ressourcenintensiven Aufgaben. Ähnlich wie die Architektur mit n-Schichten ist diese Architektur leicht verständlich. Durch den Einsatz von verwalteten Diensten werden Bereitstellung und Betrieb vereinfacht. Bei komplexen Domänen kann die Verwaltung von Abhängigkeiten jedoch schwierig sein. Das Front-End und der Worker können schnell zu großen, monolithischen Komponenten werden, die schwer zu warten und zu aktualisieren sind. Wie bei der Architektur mit n-Schichten kann dies die Häufigkeit der Aktualisierungen verringern und die Innovationsmöglichkeiten einschränken.

### <a name="microservices"></a>Microservices

<img src="./images/microservices-sketch.svg" style="float:left; margin-top:6px;"/>

Wenn Ihre Anwendung eine komplexere Domäne umfasst, sollten Sie einen Wechsel zu einer **[Microservicearchitektur][microservices]** in Erwägung ziehen. Eine Microserviceanwendung besteht aus vielen kleinen, unabhängigen Diensten. Jeder Dienst implementiert wiederum eine einzelne Geschäftsfunktion. Die Dienste sind lose gekoppelt und kommunizieren dabei über API-Verträge.

Jeder Dienst kann von einem kleinen, dedizierten Entwicklungsteam erstellt werden. Einzelne Dienste können ohne großen Koordinationsaufwand zwischen den Teams bereitgestellt werden, was häufige Aktualisierungen begünstigt. Die Erstellung und Verwaltung einer Mikroservicearchitektur ist komplexer als bei einer Architektur mit n-Schichten oder einer Web-Warteschlange-Worker-Architektur. Hierfür ist eine ausgereifte Entwicklungs- und DevOps-Kultur vonnöten. Bei richtiger Umsetzung kann dieser Stil jedoch zu einer höheren Releasegeschwindigkeit, einer schnelleren Innovationsrate und einer stabileren Architektur führen. 

### <a name="cqrs"></a>CQRS-Architektur

<img src="./images/cqrs-sketch.svg" style="float:left; margin-top:6px;"/>

Der **[CQRS](./cqrs.md)**-Stil (Command and Query Responsibility Segregation) trennt Lese- und Schreibvorgänge in separate Modelle. Dadurch werden die Teile des Systems, die Daten aktualisieren, von den Teilen, die die Daten lesen, isoliert. Außerdem können Lesevorgänge in einer materialisierten Sicht ausgeführt werden, die physisch von der Datenbank mit Schreibzugriff getrennt ist. Dadurch können Sie die Workloads für Lese- und Schreibvorgänge unabhängig voneinander skalieren und die materialisierte Sicht in Bezug auf Abfragen optimieren.

Der CQRS-Stil ist am sinnvollsten, wenn er auf ein Subsystem einer größeren Architektur angewendet wird. Im Allgemeinen sollten Sie diesen Stil nicht für die gesamte Anwendung erzwingen, da hierdurch unnötigerweise Komplexität entsteht. Ziehen Sie ihn für kollaborative Domänen in Erwägung, bei denen viele Benutzer auf die gleichen Daten zugreifen.

### <a name="event-driven-architecture"></a>Ereignisgesteuerte Architektur 

<img src="./images/event-driven-sketch.svg" style="float:left; margin-top:6px;"/>

**[Ereignisgesteuerte Architekturen](./event-driven.md)** beruhen auf einem Veröffentlichen/Abonnieren-Modell (Publish-Subscribe, Pub-Sub), bei dem Produzenten Ereignisse veröffentlichen, die Konsumenten abonnieren. Die Produzenten sind unabhängig von den Konsumenten, die wiederum unabhängig voneinander sind. 

Ziehen Sie eine ereignisgesteuerte Architektur bei Anwendungen in Betracht, die große Datenmengen mit sehr geringer Latenz erfassen und verarbeiten, wie etwa IoT-Lösungen. Dieser Stil ist auch nützlich, wenn verschiedene Subsysteme unterschiedliche Verarbeitungstypen für die gleichen Ereignisdaten durchführen müssen.

<br />

### <a name="big-data-big-compute"></a>Big Data, Big Compute

**[Big Data](./big-data.md)** und **[Big Compute](./big-compute.md)** sind besondere Architekturstile für Workloads, die bestimmten speziellen Profilen entsprechen. Bei Big Data wird ein sehr großes Dataset in Blöcke unterteilt, die ein gesamtes Dataset für die Analyse und Berichterstellung parallel verarbeiten. Big Compute, auch als „High Performance Computing“ (HPC) bezeichnet, führt parallele Berechnungen über eine große Anzahl (Tausenden) von Kernen durch. Zu Domänen zählen Simulationen, Modellierung und 3D-Rendering.

## <a name="architecture-styles-as-constraints"></a>Architekturstile als Einschränkungen

Ein Architekturstil erstellt Einschränkungen für den Entwurf, wie etwa für die Gruppe der Elemente, die angezeigt werden können, und die erlaubten Beziehungen zwischen diesen Elementen. Einschränkungen bestimmen die „Form“ einer Architektur, indem sie die Auswahlmöglichkeiten einschränken. Wenn eine Architektur den Einschränkungen eines bestimmten Stils entspricht, entstehen bestimmte wünschenswerte Eigenschaften. 

Zu den Einschränkungen bei Microservices zählen beispielsweise Folgende: 

- Ein Dienst stellt eine einzelne Zuständigkeit dar. 
- Dienste sind voneinander unabhängig. 
- Die Daten gelten ausschließlich für den Dienst, dem sie angehören. Dienste geben keine Daten frei.

Durch die Einhaltung dieser Einschränkungen entsteht ein System, in dem Dienste unabhängig voneinander bereitgestellt werden können, Fehler isoliert werden, häufige Aktualisierungen möglich sind und mühelos neue Technologien in der Anwendung implementiert werden können.

Bevor Sie sich für einen Architekturstil entscheiden, vergewissern Sie sich, dass Sie sich über die zugrunde liegenden Prinzipien und Einschränkungen des jeweiligen Stils im Klaren sind. Andernfalls kann es sein, dass Sie einen Entwurf erhalten, der dem Stil zwar oberflächlich entspricht, aber nicht das volle Potenzial dieses Stils ausschöpft. Des Weiteren ist es auch wichtig, pragmatisch zu sein. Manchmal ist es besser, eine Einschränkung zu lockern, als auf architektonische Reinheit zu bestehen.


In der folgenden Tabelle wird zusammengefasst, wie die einzelnen Stile Abhängigkeiten verwalten und welche Domänentypen am besten für die jeweiligen Stile geeignet sind.

| Architekturstil |  Verwaltung von Abhängigkeiten | Domänentyp |
|--------------------|------------------------|-------------|
| n-Schichten | Horizontale in Subnetze unterteilte Schichten | Konventioneller Geschäftsbereich, niedrige Aktualisierungshäufigkeit |
| Web-Warteschlange-Worker | Front- und Back-End-Aufträge, die von asynchronen Nachrichten entkoppelt sind | Relativ einfache Domäne mit einigen ressourcenintensiven Aufgaben |
| Microservices | Vertikal (funktional) zerlegte Dienste, die sich über APIs gegenseitig aufrufen | Komplexe Domäne, häufige Aktualisierungen |
| CQRS-Architektur | Trennung von Lese-/Schreibvorgängen, Schema und Skalierung werden getrennt voneinander optimiert | Kollaborative Domäne, bei denen viele Benutzer auf die gleichen Daten zugreifen |
| Ereignisgesteuert | Produzent/Konsument, unabhängige Ansicht pro Subsystem | IoT- und Echtzeitsysteme |
| Große Datenmengen | Unterteilung riesiger Datasets in kleine Blöcke, parallele Verarbeitung lokaler Datasets | Batch- und Echtzeit-Datenanalysen, Predictive Analytics durch ML |
| Big Compute| Datenzuordnung zu Tausenden von Kernen | Rechenintensive Bereiche wie Simulationen |


## <a name="consider-challenges-and-benefits"></a>Berücksichtigung der Herausforderungen und Vorteile

Einschränkungen bergen auch Herausforderungen. Daher ist es wichtig, sich über die Abwägungen bei der Anwendung dieser Stile bewusst zu sein. Achten Sie darauf, ob die Vorteile eines Architekturstils die Herausforderungen *für die jeweilige Unterdomäne und den damit verbundenen Kontext* überwiegen. 

Im Folgenden werden einige der Herausforderungen aufgeführt, die bei der Auswahl eines Architekturstils zu berücksichtigen sind:

- **Komplexität**: Ist die Komplexität der Architektur für Ihre Domäne gerechtfertigt? Fragen Sie sich umgekehrt auch, ob der Stil für Ihre Domäne eventuell zu simpel ist. In diesem Fall riskieren Sie, am Ende einen sogenannten [„Ball of Mud“][ball-of-mud] zu erhalten, da die Architektur nicht im Hinblick auf eine korrekte Verwaltung von Abhängigkeiten unterstützt.

- **Asynchrone Nachrichten und letztliche Konsistenz**: Mit asynchronen Nachrichten können Dienste entkoppelt und die Zuverlässigkeit (aufgrund der Möglichkeit zur Wiederholung von Nachrichten) sowie die Skalierbarkeit erhöht werden. Dies führt allerdings auch zu Herausforderungen, wie etwa AlwaysOn-Semantiken und letztliche Konsistenz.

- **Kommunikation zwischen Diensten**: Wenn Sie eine Anwendung in separate Dienste zerlegen, besteht die Gefahr, dass die Kommunikation zwischen den Diensten zu inakzeptablen Latenzen oder zu einer Netzwerküberlastung führt (z.B. in einer Microservicearchitektur). 

- **Verwaltbarkeit**: Wie schwierig ist es, die Anwendung zu verwalten und zu überwachen, Aktualisierungen bereitzustellen und sonstige damit zusammenhängende Aufgaben auszuführen?


[ball-of-mud]: https://en.wikipedia.org/wiki/Big_ball_of_mud
[microservices]: ./microservices.md
[n-tier]: ./n-tier.md
