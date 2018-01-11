---
title: Entwerfen, Erstellen und Betreiben von Microservices in Azure mit Kubernetes
description: Entwerfen, Erstellen und Betreiben von Microservices in Azure
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: 857e91a8eeefec18b459f2e66fde9a4f8bbe7b21
ms.sourcegitcommit: 744ad1381e01bbda6a1a7eff4b25e1a337385553
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/08/2018
---
# <a name="designing-building-and-operating-microservices-on-azure"></a>Entwerfen, Erstellen und Betreiben von Microservices in Azure

![](./images/drone.svg)

Microservices haben sich zu einem beliebten Architekturstil für die Erstellung robuster, hochgradig skalierbarer, unabhängig bereitstellbarer Cloudanwendungen entwickelt, die sich schnell weiterentwickeln lassen. Damit Microservices nicht nur ein Schlagwort bleiben, erfordern sie allerdings eine andere Herangehensweise an die Gestaltung und Erstellung von Anwendungen. 

Diese Artikelreihe beschäftigt sich mit der Erstellung und dem Betrieb einer Microservices-Architektur in Azure. Dabei werden folgende Themen behandelt:

- Entwerfen einer Microservices-Architektur unter Verwenden von DDD (Domain-Driven Design) 
- Auswählen der richtigen Azure-Technologien für Compute, Speicherung, Messaging und andere Designelemente
- Nachvollziehen der Entwurfsmuster für Microservices
- Erstellen eines Entwurfs zur Gewährleistung von Resilienz, Skalierbarkeit und Leistung
- Erstellen einer CI/CD-Pipeline


In der gesamten Reihe konzentrieren wir uns auf ein bestimmtes End-to-End-Szenario: einen Drohnenlieferdienst, mit dem Kunden Termine für Pakete festlegen können, die dann per Drohne abgeholt und zugestellt werden. Den Code unserer Referenzimplementierung finden Sie auf GitHub.

[![GitHub](../_images/github.png) Referenzimplementierung][drone-ri]

Zuerst sollten wir uns jedoch mit den Grundlagen beschäftigen. Was sind Microservices, und welche Vorteile bietet die Implementierung einer Microservices-Architektur?

## <a name="why-build-microservices"></a>Was spricht für die Erstellung von Microservices?

In einer Microservices-Architektur setzt sich die Anwendung aus kompakten, unabhängigen Diensten zusammen. Im Anschluss finden Sie einige definierende Merkmale von Microservices:

- Jeder Microservice implementiert eine einzelne Geschäftsfunktion.
- Ein Microservice ist so kompakt, dass er von einem kleinen Entwicklerteam geschrieben und verwaltet werden kann.
- Microservices werden in getrennten Prozessen ausgeführt und kommunizieren über klar definierte APIs oder Nachrichtenmuster. 
- Microservices teilen sich keine Datenspeicher oder Datenschemas. Jeder Microservice ist für die Verwaltung seiner eigenen Daten zuständig. 
- Microservices verfügen über separate Codebasen und teilen sich keinen Quellcode. Sie können jedoch gemeinsame Hilfsprogrammbibliotheken verwenden.
- Jeder Microservice kann unabhängig von anderen Diensten bereitgestellt und aktualisiert werden. 

Ordnungsgemäß implementierte Microservices bieten eine ganze Reihe von Vorteilen:

- **Flexibilität:** Die unabhängige Bereitstellung von Microservices vereinfacht die Verwaltung von Fehlerkorrekturen und Featureveröffentlichungen. Sie können einen Dienst aktualisieren, ohne die gesamte Anwendung erneut bereitstellen zu müssen, und im Problemfall einen Rollback für ein Update ausführen. In vielen herkömmlichen Anwendung kann die Entdeckung eines Fehlers in einem Teil der Anwendung den gesamte Releaseprozess blockieren. Dies kann dazu führen, dass sich die Einführung neuer Features verzögert, bis eine Fehlerkorrektur integriert, getestet und veröffentlicht wurde.  

- **Kompakter Code, kleine Teams:** Ein Microservice muss so kompakt sein, dass er von einem einzelnen Featureteam erstellt, getestet und bereitgestellt werden kann. Eine kompakte Codebasis ist leichter nachvollziehbar. In einer umfangreichen monolithischen Anwendung werden Codeabhängigkeiten im Laufe der Zeit häufig unübersichtlich, sodass zum Hinzufügen eines neuen Features Code an zahlreichen Stellen bearbeitet werden muss. Da sich Microservices keinen Code und keine Datenspeicher teilen, enthält eine Microservices-Architektur weniger Abhängigkeiten und vereinfacht dadurch das Hinzufügen neuer Features. Kleinere Teams sind außerdem agiler. Eine Regel besagt, dass ein Team zu groß ist, wenn zwei Pizzen nicht ausreichen, um das gesamte Team zu versorgen. Das ist natürlich keine exakte Metrik und hängt vom Appetit der Teammitglieder ab. Entscheidend ist jedoch, dass große Gruppen in der Regel weniger produktiv sind, da sich die Kommunikation verlangsamt, der Verwaltungsaufwand steigt und die Agilität abnimmt.  

- **Kombination verschiedener Technologien:** Teams können sich für die Technologie entscheiden, die am besten zu ihrem Dienst passt, und eine Kombination aus geeigneten Technologiestapeln verwenden. 

- **Resilienz:** Der Ausfall eines einzelnen Microservice hat nicht den Ausfall der gesamten Anwendung zur Folge – vorausgesetzt, die Upstream-Microservices sind für eine korrekte Behandlung von Ausfällen konzipiert (beispielsweise durch Implementierung des Trennschalter-Musters).

- **Skalierbarkeit**. In einer Microservices-Architektur kann jeder Microservice unabhängig skaliert werden. Dadurch lassen sich Subsysteme, die mehr Ressourcen benötigen, horizontal hochskalieren, ohne die gesamte Anwendung hochzuskalieren. Wenn Sie Dienste in Containern bereitstellen, können Sie Microservices zudem dichter auf einem einzelnen Host platzieren und Ressourcen effizienter nutzen.

- **Datenisolation:** Schemaaktualisierungen sind wesentlich einfacher, da nur ein einzelner Microservice betroffen ist. Bei einer monolithischen Anwendung können sich Schemaaktualisierungen als echte Herausforderung erweisen, da verschiedene Teile der Anwendung unter Umständen die gleichen Daten nutzen, was jegliche Anpassung des Schemas zu einer riskanten Angelegenheit macht.
 
## <a name="no-free-lunch"></a>Nichts ist umsonst

Diese Vorteile haben ihren Preis. In dieser Artikelreihe werden einige der Herausforderungen im Zusammenhang mit der Erstellung robuster, skalierbarer und verwaltbarer Microservices behandelt.

- **Dienstgrenzen:** Bei der Erstellung von Microservices müssen Sie sich genau überlegen, wo die Grenzen zwischen den Diensten verlaufen sollen. Ist ein Dienst erst einmal erstellt und in der Produktionsumgebung bereitgestellt, kann es schwierig sein, diese Grenzen zu verändern. Die Wahl geeigneter Dienstgrenzen ist eine der größten Herausforderungen beim Entwerfen einer Microservices-Architektur. Wie groß sollten die einzelnen Dienste sein? Wann sollte sich eine Funktion über mehrere Dienste erstrecken und wann nur einen einzelnen Dienst umfassen? In diesem Leitfaden beschreiben wir die Ermittlung von Dienstgrenzen mit einem DDD-Ansatz (Domain-Driven Design). Zunächst wird eine [Domänenanalyse](./domain-analysis.md) durchgeführt, um die Kontextgrenzen zu finden. Danach wird auf der Grundlage funktionsbezogener und nicht funktionsbezogener Anforderungen eine Reihe [taktischer DDD-Muster](./microservice-boundaries.md) angewendet. 

- **Datenkonsistenz und -integrität:** Ein Grundprinzip von Microservices ist die Verwaltung der jeweils eigenen Daten. Dieses Prinzip sorgt zwar für eine Entkopplung der Dienste, kann aber zu Herausforderungen bei der Datenintegrität oder Redundanz führen. Einige der Aspekte werden in den [Überlegungen zu Daten](./data-considerations.md) behandelt.

- **Netzwerkkonflikte und -latenz**. Die Verwendung zahlreicher kompakter, differenzierter Dienste kann zu einem erhöhten Kommunikationsaufkommen zwischen Diensten sowie zu einer längeren End-to-End-Wartezeit führen. Das Kapitel [Kommunikation zwischen Diensten](./interservice-communication.md) enthält Informationen zum Messaging zwischen Diensten. In Microservices-Architekturen kann sowohl eine synchrone als auch eine asynchrone Kommunikation verwendet werden. Ein gutes [API-Design](./api-design.md) sorgt dafür, dass Dienste lose gekoppelt bleiben und unabhängig bereitgestellt und aktualisiert werden können.
 
- **Komplexität**. Eine Microservices-Anwendung weist eine höhere Komplexität auf. Die einzelnen Dienste können zwar einfach gestaltet sein, sie müssen aber als Ganzes zusammenarbeiten. Ein einzelner Benutzervorgang kann mehrere Dienste umfassen. Im Kapitel [Erfassung und Workflow](./ingestion-workflow.md) untersuchen wir einige der Aspekte in Verbindung mit der Erfassung von Anforderungen mit hohem Durchsatz, mit der Koordinierung eines Workflows sowie mit der Behandlung von Fehlern. 

- **Kommunikation zwischen Clients und Anwendung:**  Wie sollten Clients mit den Diensten kommunizieren, wenn Sie eine Anwendung in zahlreiche kompakte Dienste aufspalten? Soll ein Client jeden einzelnen Dienst direkt aufrufen oder Anforderungen über ein [API-Gateway](./gateway.md) weiterleiten?

- **Überwachung:** Die Überwachung einer verteilten Anwendung kann deutlich schwieriger sein als die Überwachung einer monolithischen Anwendung, da hierzu Telemetriedaten aus mehreren Diensten korreliert werden müssen. Hiermit beschäftigen wir uns im Kapitel [Protokollierung und Überwachung](./logging-monitoring.md).

- **Continuous Integration und Continuous Delivery (CI/CD):** Eines der Hauptziele von Microservices ist Agilität. Zur Erreichung dieses Ziels benötigen Sie automatisierte und stabile [CI/CD-Features](./ci-cd.md), um einzelne Dienste schnell und zuverlässig in Test- und Produktionsumgebungen bereitstellen zu können.

## <a name="the-drone-delivery-application"></a>Die Drohnenlieferungsanwendung

Zur Erläuterung dieser Aspekte und zur Veranschaulichung einiger bewährter Methoden für eine Microservices-Architektur haben wir als Referenzimplementierung die Drohnenlieferungsanwendung erstellt. Die Referenzimplementierung finden Sie auf [GitHub][drone-ri].

Fabrikam, Inc. führt einen Drohnenlieferdienst ein. Das Unternehmen verfügt über eine Drohnenflotte. Unternehmen registrieren sich bei dem Dienst, und Benutzer können eine Drohne anfordern, die auszuliefernde Waren abholt. Wenn ein Kunde einen Abholtermin festlegt, weist ein Back-End-System eine Drohne zu und teilt dem Benutzer die voraussichtliche Lieferzeit mit. Während der Durchführung der Lieferung kann der Kunde die Position der Drohne nachverfolgen, und die voraussichtliche Ankunftszeit wird kontinuierlich aktualisiert.

Dieses Szenario beinhaltet eine recht komplizierte Domäne. Zu den Anliegen des Unternehmens zählen unter anderem die Zeitplanung für die Drohnen, die Nachverfolgung von Paketen, die Verwaltung von Benutzerkonten sowie die Speicherung und Analyse von Verlaufsdaten. Darüber hinaus ist Fabrikam an einer schnellen Marktreife interessiert und möchte dann zeitnah neue Features und Funktionen hinzufügen. Die Anwendung muss cloudfähig sein und über ein hohes Servicelevelziel (Service Level Objective, SLO) verfügen. Fabrikam erwartet außerdem erhebliche Unterschiede bei den Datenspeicher- und Abfrageanforderungen der verschiedenen Systemkomponenten. Aufgrund dieser Überlegungen hat sich Fabrikam bei seiner Drohnenlieferungsanwendung für eine Microservices-Architektur entschieden.

> [!NOTE]
> Hilfreiche Informationen zur Wahl einer Microservices-Architektur oder eines anderen Architekturstils finden Sie im [Azure-Anwendungsarchitekturleitfaden](../guide/index.md).

Unsere Referenzimplementierung verwendet Kubernetes mit [Azure Container Service (ACS)](/azure/container-service/kubernetes/). Viele der allgemeinen architekturbezogenen Entscheidungen und Herausforderungen gelten jedoch für alle Containerorchestratoren (einschließlich [Azure Service Fabric](/azure/service-fabric/)). 

> [!div class="nextstepaction"]
> [Domänenanalyse](./domain-analysis.md)


<!-- links -->

[drone-ri]: https://github.com/mspnp/microservices-reference-implementation
