---
title: Entwurf mit Blick auf Selbstreparatur
description: Resiliente Anwendungen können nach einem Ausfall ohne manuellen Eingriff wiederhergestellt werden.
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 0782b65b77615f7c006724264ab0ca2d2c7c04e2
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="design-for-self-healing"></a>Entwurf mit Blick auf Selbstreparatur

## <a name="design-your-application-to-be-self-healing-when-failures-occur"></a>Entwerfen Sie Ihre Anwendung so, dass sie bei Ausfällen eine Selbstreparatur durchführt.

In einem verteilten System kann es zu Ausfällen kommen. Hardware kann ausfallen. Im Netzwerk können vorübergehende Fehler auftreten. In seltenen Fällen kann ein gesamter Dienst oder eine Region ausfallen, aber auch hierfür muss ein Plan vorhanden sein.

Entwerfen Sie deshalb Ihre Anwendung so, dass sie bei Ausfällen eine Selbstreparatur durchführt. Dies erfordert einen dreifachen Ansatz:

- Erkennen von Ausfällen.
- Ordnungsgemäßes Reagieren auf Ausfälle.
- Protokollieren und Überwachen von Ausfällen, um Einblick in die Abläufe zu gewinnen.

Ihre Reaktion auf einen bestimmten Typ von Ausfällen kann von den Anforderungen an die Verfügbarkeit der Anwendung abhängen. Wenn z.B. eine hohe Verfügbarkeit erforderlich ist, kann während eines regionalen Ausfalls ein automatisches Failover zu einer sekundären Region erfolgen. Allerdings entstehen dadurch höhere Kosten als für die Bereitstellung in einer einzelnen Region. 

Berücksichtigen Sie zudem nicht nur umfangreiche Ereignisse, z.B. regionale Ausfälle, die in der Regel selten auftreten. Sie sollten sich mindestens in gleichem Ausmaß mit der Behandlung lokaler, kurzfristiger Ausfälle, z.B. dem Ausfall von Netzwerk- oder Datenbankverbindungen, befassen.

## <a name="recommendations"></a>Recommendations

**Wiederholen Sie fehlgeschlagene Vorgänge**. Vorübergehende Fehler können durch eine kurze Trennung der Netzwerkverbindung, einen Verlust der Verbindung mit der Datenbank oder eine Zeitüberschreitung bei zu hoher Auslastung eines Diensts verursacht werden. Integrieren Sie zur Behandlung vorübergehender Fehler Wiederholungslogik in die Anwendung. Für viele Azure-Dienste werden automatische Wiederholungen durch das Client-SDK implementiert. Weitere Informationen finden Sie unter [Behandeln vorübergehender Fehler][transient-fault-handling] und [Wiederholungsmuster][retry].

**Schützen Sie ausgefallene Remotedienste (Trennschalter)**. Es empfiehlt sich, nach einem vorübergehenden Fehler den Vorgang zu wiederholen. Wenn jedoch der Fehler weiterhin auftritt, treffen möglicherweise zu viele Aufrufe auf den ausgefallenen Dienst. Dies kann zu kaskadierenden Fehlern führen, wenn sich Anforderungen aufstauen. Verwenden Sie das [Trennschalter-Muster] [ circuit-breaker], um schnell einen Fehler auszulösen (ohne den Remoteaufruf auszuführen), wenn eine hohe Wahrscheinlichkeit besteht, dass ein Vorgang fehlschlägt.  

**Isolieren Sie kritische Ressourcen (Bulkhead)**. Fehler in einem Subsystem können manchmal kaskadiert werden. Dies kann passieren, wenn ein Fehler dazu führt, dass einige Ressourcen, z.B. Threads oder Sockets, nicht rechtzeitig freigegeben werden und die Ressourcen erschöpft sind. Um dies zu vermeiden, sollten Sie ein System in isolierte Gruppen partitionieren, damit ein Ausfall in einer Partition nicht zum Ausfall des gesamten Systems führt.  

**Führen Sie einen Lastenausgleich aus**. Für Anwendungen kann es zu plötzlichen Datenverkehrsspitzen kommen, die von Diensten auf dem Back-End nicht mehr bewältigt werden können. Um dies zu vermeiden, verwenden Sie das [warteschlangenbasierte Lastenausgleichsmuster][load-level], um Arbeitselemente für die asynchrone Ausführung in die Warteschlange zu stellen. Die Warteschlange fungiert als Puffer, um Lastspitzen zu glätten. 

**Führen Sie ein Failover aus**. Wenn eine Instanz nicht erreicht werden kann, führen Sie ein Failover zu einer anderen Instanz aus. Platzieren Sie für zustandslose Elemente, z.B. einen Webserver, mehrere Instanzen hinter einem Lastenausgleich oder Traffic Manager. Verwenden Sie für Elemente, die Zustände speichern, z.B. eine Datenbank, Replikate und Failover. Je nach Datenspeicher und der Art seiner Replikation muss in der Anwendung möglicherweise letztliche Konsistenz verwendet werden. 

**Gleichen Sie fehlgeschlagene Transaktionen aus**. Vermeiden Sie generell verteilte Transaktionen, da sie die Koordination zwischen Diensten und Ressourcen erfordern. Stellen Sie stattdessen einen Vorgang aus kleineren einzelnen Transaktionen zusammen. Wenn der Vorgang während der Ausführung fehlschlägt, machen Sie die bereits abgeschlossenen Schritte mithilfe [ausgleichender Transaktionen][compensating-transactions] rückgängig. 

**Verwenden Sie für lang andauernde Transaktionen Prüfpunkte**. Prüfpunkte können Resilienz ermöglichen, wenn ein lang andauernder Vorgang fehlschlägt. Wenn der Vorgang neu gestartet wird (wenn er z.B. von einer anderen VM ausgewählt wird), kann er ab dem letzten Prüfpunkt fortgesetzt werden.

**Stufen Sie Funktionalität korrekt herab**. Wenn eine Problemumgehung nicht möglich ist, können Sie manchmal reduzierte Funktionalität bereitstellen, die immer noch hilfreich ist. Stellen Sie sich eine Anwendung vor, in der ein Katalog mit Büchern angezeigt wird. Wenn die Anwendung das Miniaturbild für den Buchdeckel nicht abrufen kann, wird ggf. ein Platzhalterbild angezeigt. Möglicherweise sind gesamte Subsysteme für die Anwendung nicht kritisch. Beispielsweise ist auf einer E-Commerce-Website das Anzeigen von Produktempfehlungen wahrscheinlich weniger kritisch als das Verarbeiten von Bestellungen.

**Drosseln Sie Clients**. Manchmal erzeugt eine kleine Anzahl von Benutzern eine übermäßige Last, wodurch die Verfügbarkeit der Anwendung für andere Benutzer reduziert wird. Drosseln Sie in einem solchen Fall den Client für einen bestimmten Zeitraum. Siehe [Drosselungsmuster][throttle].

**Blockieren Sie schädliche Akteure**. Wenn Sie einen Client drosseln, bedeutet dies nicht zwangsläufig, dass der Client böswillig agiert hat. Es bedeutet lediglich, dass der Client das Dienstkontingent überschritten hat. Wenn ein Client jedoch das Kontingent beständig überschreitet oder sich auf sonstige Weise schädlich verhält, können Sie ihn blockieren. Definieren Sie einen Out-of-band-Prozess, damit Benutzer das Aufheben der Blockierung anfordern können.

**Wählen Sie eine übergeordnete Instanz aus**. Wenn Sie eine Aufgabe koordinieren müssen, wählen Sie einen Koordinator aus, indem Sie eine [übergeordnete Instanz auswählen][leader-election]. Auf diese Weise ist der Koordinator kein Single Point of Failure. Wenn der Koordinator ausfällt, wird ein neuer ausgewählt. Statt einen vollkommen neu entwickelten Algorithmus für die Auswahl einer übergeordneten Instanz zu implementieren, sollten Sie eine bereits fertige Lösung verwenden, z.B. Zoekeeper.  

**Testen Sie mit Fault Injection**. Allzu häufig wird der Erfolgspfad gründlich getestet, jedoch nicht der Fehlerpfad. Ein System kann bereits lange Zeit in der Produktion ausgeführt werden, bevor ein Fehlerpfad getestet wird. Testen Sie die Resilienz des Systems bei Fehlern mithilfe von Fault Injection, indem Sie entweder echte Fehler auslösen oder diese simulieren. 

**Nutzen Sie Chaos Engineering**. Chaos Engineering ist eine Erweiterung des Konzepts von Fault Injection. Dabei werden Fehler oder anormale Bedingungen in Produktionsinstanzen eingefügt. 

Informationen zu einer strukturieren Vorgehensweise für die Selbstreparatur von Anwendungen finden Sie unter [Entwerfen robuster Anwendungen für Azure][resiliency-overview].  

[circuit-breaker]: ../../patterns/circuit-breaker.md
[compensating-transactions]: ../../patterns/compensating-transaction.md
[leader-election]: ../../patterns/leader-election.md
[load-level]: ../../patterns/queue-based-load-leveling.md
[resiliency-overview]: ../../resiliency/index.md
[retry]: ../../patterns/retry.md
[throttle]: ../../patterns/throttling.md
[transient-fault-handling]: ../../best-practices/transient-faults.md

