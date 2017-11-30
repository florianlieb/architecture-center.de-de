---
title: Drosselung
description: Steuern Sie den Verbrauch der von einer Anwendungsinstanz, einem einzelnen Mandanten oder einem gesamten Dienst verwendeten Ressourcen.
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- availability
- performance-scalability
ms.openlocfilehash: 29156fc72f40a952dd53adcb20ffa7c3d0af79b4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="throttling-pattern"></a>Muster „Drosselung“

[!INCLUDE [header](../_includes/header.md)]

Steuern Sie den Verbrauch der von einer Anwendungsinstanz, einem einzelnen Mandanten oder einem gesamten Dienst verwendeten Ressourcen. So kann das System auch dann weiter funktionieren und Vereinbarungen zum Servicelevel (Service Level Agreement, SLAs) erfüllen, wenn ein steigender Bedarf zu einer extremen Ressourcenauslastung führt.

## <a name="context-and-problem"></a>Kontext und Problem

Die Last einer Cloudanwendung schwankt normalerweise im Laufe der Zeit, je nach Anzahl der aktiven Benutzer oder den Arten der von ihnen ausgeführten Aktivitäten. Dies ist beispielsweise während den Geschäftszeiten der Fall, in denen aller Wahrscheinlichkeit nach mehr Benutzer aktiv sind, oder am Ende eines jeden Monats, an dem das System rechenintensive Analysen durchführen muss. Zudem kann es zu plötzlichen und unvorhergesehenen Aktivitätsspitzen kommen. Wenn die Verarbeitungsanforderungen des Systems die Kapazität der verfügbaren Ressourcen überschreiten, kommt es zu einer Leistungsverschlechterung und eventuell sogar zu Ausfällen. Wenn das System einen vereinbarten Servicelevel erfüllen muss, kann ein solcher Ausfall untragbar sein.

Für den Umgang mit einer schwankenden Last in der Cloud gibt es zahlreiche Strategien, was von den Geschäftszielen der Anwendung abhängt. Eine Strategie besteht darin, die bereitgestellten Ressourcen zu einem beliebigen Zeitpunkt durch automatische Skalierung an die Benutzeranforderungen anzupassen. Hierdurch können nicht nur Benutzeranforderungen konsistent erfüllt werden, sondern es werden gleichzeitig laufende Kosten optimiert. Die automatische Skalierung kann zwar die Bereitstellung zusätzlicher Ressourcen auslösen, eine solche Bereitstellung erfolgt jedoch nicht unmittelbar. Steigt die Nachfrage schnell, kann es innerhalb eines bestimmten Zeitfensters zu einem Ressourcendefizit kommen.

## <a name="solution"></a>Lösung

Eine alternative Strategie zur automatischen Skalierung besteht darin, die Nutzung von Ressourcen durch Anwendungen bis zu einem bestimmten Grenzwert zuzulassen und diese bei Erreichen dieses Grenzwerts dann zu drosseln. Das System sollte die eigene Ressourcennutzung überwachen, damit Anforderungen von einem oder mehreren Benutzern bei einer Überschreitung des Schwellenwertes gedrosselt werden können. Auf diese Weise kann das System weiterhin funktionieren und alle bestehenden Vereinbarungen zum Servicelevel (SLAs) erfüllen. Weitere Informationen zur Überwachung der Ressourcennutzung finden Sie unter [Instrumentierungs- und Telemetrieleitfaden](https://msdn.microsoft.com/library/dn589775.aspx).

Das System kann mehrere Drosselungsstrategien implementieren, wie etwa Folgende:

- Ablehnung von Anforderungen eines einzelnen Benutzers, der in einem bestimmten Zeitraum bereits mehr als n-mal pro Sekunde auf System-APIs zugegriffen hat. Hierfür muss das System den Ressourcenverbrauch für jeden Mandanten oder Benutzer, der eine Anwendung ausführt, messen. Weitere Informationen finden Sie im [Leitfaden zur Dienstmessung](https://msdn.microsoft.com/library/dn589796.aspx).

- Deaktivieren oder Beschränken der Funktionalität ausgewählter nicht wesentlicher Dienste, sodass wesentliche Dienste ungehindert mit ausreichenden Ressourcen ausgeführt werden können. Wenn die Anwendung beispielsweise Videoausgaben streamt, kann sie zu einer geringeren Auflösung wechseln.

- Abschwächen des Aktivitätsvolumens durch einen Lastenausgleich (diese Vorgehensweise wird unter [Muster „Warteschlangenbasierter Lastenausgleich“](queue-based-load-leveling.md) näher beschrieben). In einer mehrinstanzenfähigen Umgebung wird die Leistung durch diese Vorgehensweise bei jedem Mandanten beeinträchtigt. Wenn das System eine Kombination von Mandanten mit unterschiedlichen SLAs unterstützen muss, können die Aufgaben für Mandanten mit hoher Priorität sofort durchgeführt werden. Anforderungen für andere Mandanten können zurückgehalten und bearbeitet werden, wenn der Rückstand nachgelassen hat. Zur Implementierung dieser Vorgehensweise kann als Unterstützung das [Muster „Prioritätswarteschlange“][] verwendet werden.

- Verschieben von Vorgängen, die für Anwendungen oder Mandanten mit geringerer Priorität durchgeführt werden. Diese Vorgänge können angehalten oder beschränkt werden, wobei eine Ausnahme erzeugt wird, um den Mandanten darüber zu informieren, dass das System ausgelastet ist und der Vorgang später erneut wiederholt werden sollte.

Die Abbildung zeigt ein Flächendiagramm zum Ressourcenverbrauch (eine Kombination aus Speicher, CPU, Bandbreite und anderen Faktoren) im Zeitverlauf für Anwendungen, die auf alle drei Funktionen zurückgreifen. Ein Feature ist ein Funktionsbereich wie etwa eine Komponente, die eine bestimmte Gruppe von Aufgaben ausführt, ein Codeausschnitt, der eine komplexe Berechnung durchführt, oder ein Element, das einen Dienst bereitstellt (z.B. ein In-Memory-Cache). Diese Features sind mit den Buchstaben A, B und C gekennzeichnet.

![Abbildung 1: Diagramm zur Darstellung des Ressourcenverbrauchs im Zeitverlauf für Anwendungen, die für drei Benutzer ausgeführt werden](./_images/throttling-resource-utilization.png)


> Der Bereich unmittelbar unter der Linie eines Features stellt die Ressourcen dar, die von Anwendungen beim Aufrufen dieses Features verwendet werden. Der Bereich unterhalb der Linie von Feature A beispielsweise stellt die Ressourcen dar, die von Anwendungen verwendet werden, die Feature A verwenden, während der Bereich zwischen den Linien von Feature A und Feature B die Ressourcen darstellt, die von Anwendungen verwendet werden, die Feature B aufrufen. Die aggregierten Bereiche der einzelnen Features stellen die gesamte Ressourcennutzung des Systems dar.

Die vorherige Abbildung zeigt die Auswirkungen bei einer Verschiebung von Vorgängen. Kurz vor Zeitpunkt T1 erreichen die gesamten Ressourcen, die sämtlichen diese Features nutzenden Anwendungen zugewiesen sind, einen Schwellenwert (den Grenzwert des Ressourcenverbrauchs). An diesem Punkt besteht die Gefahr, dass Anwendungen die verfügbaren Ressourcen ausschöpfen. In diesem System ist Feature B weniger entscheidend als Feature A oder Feature C. Es wird daher vorübergehend deaktiviert, und die von diesem Feature verwendeten Ressourcen werden freigegeben. Im Zeitraum zwischen Zeitpunkt T1 und T2 werden die Anwendungen, die Feature A und Feature C verwenden, wie gewohnt ausgeführt. Letztendlich sinkt der Ressourcenverbrauch dieser beiden Features so weit, dass bei Zeitpunkt T2 genügend Kapazitäten für die erneute Aktivierung von Feature B vorhanden sind.

Die Vorgehensweisen zur automatischen Skalierung und Drosselung können auch kombiniert werden, um weiterhin die Reaktionsfähigkeit der Anwendungen und die Erfüllung der SLAs sicherzustellen. Wenn davon ausgegangen wird, dass die Nachfrage weiterhin hoch bleibt, stellt die Drosselung eine vorübergehende Lösung während der horizontalen Skalierung des Systems dar. An dieser Stelle kann die komplette Funktionalität des Systems wiederhergestellt werden.

Die folgende Abbildung zeigt ein Flächendiagramm zum gesamten Ressourcenverbrauch sämtlicher Anwendungen, die in einem bestimmten Zeitraum in einem System ausgeführt werden, und wie die Drosselung mit automatischer Skalierung kombiniert werden kann.

![Abbildung 2: Diagramm zu den Auswirkungen einer Kombination von Drosselung und automatischer Skalierung](./_images/throttling-autoscaling.png)


Bei Zeitpunkt T1 ist der Schwellenwert erreicht, der den weichen Grenzwert des Ressourcenverbrauchs angibt. An dieser Stelle kann das System mit der horizontalen Skalierung beginnen. Wenn die neuen Ressourcen jedoch nicht schnell genug zur Verfügung stehen, werden die vorhandenen Ressourcen möglicherweise aufgebraucht, und das System könnte ausfallen. Um dies zu verhindern, wird das System wie zuvor beschrieben vorübergehend gedrosselt. Wenn die automatische Skalierung abgeschlossen ist und die zusätzlichen Ressourcen zur Verfügung stehen, kann die Drosselung wieder gelockert werden.

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Bei der Entscheidung, wie dieses Muster implementiert werden soll, sind die folgenden Punkte zu beachten:

- Die Drosselung einer Anwendung und die anzuwendende Strategie sind Bestandteil der architekturbezogenen Entscheidung, die sich auf den gesamten Entwurf eines Systems auswirkt. Bereits frühzeitig im Anwendungsentwurfsprozess sollte eine Drosselung in Erwägung gezogen werden, da es schwierig ist, nach der Implementierung eines Systems Ressourcen hinzuzufügen.

- Eine Drosselung muss schnell erfolgen. Das System muss einen Aktivitätsanstieg erkennen und entsprechend darauf reagieren können. Das System muss außerdem in der Lage sein, zu seinem ursprünglichen Zustand zurückzukehren, sobald sich die Last verringert hat. Dies setzt voraus, dass die entsprechenden Leistungsdaten kontinuierlich erfasst und überwacht werden.

- Wenn ein Dienst vorübergehend eine Benutzeranforderung ablehnen muss, sollte dieser einen bestimmten Fehlercode zurückgeben, um die Clientanwendung darüber zu informieren, dass die Ablehnung zur Durchführung eines Vorgangs auf eine Drosselung zurückzuführen ist. Die Clientanwendung wartet möglicherweise für einen gewissen Zeitraum, bis sie die Anforderung wiederholt.

- Die Drosselung kann als vorübergehende Maßnahme während der automatischen Skalierung eines Systems genutzt werden. In manchen Fällen empfiehlt es sich, beim plötzlichen Auftreten einer Aktivitätsspitze von wahrscheinlich nicht sehr langer Dauer einfach eine Drosselung anstelle einer automatischen Skalierung vorzunehmen, da eine Skalierung die laufenden Kosten deutlich in die Höhe treiben kann.

- Wenn während der automatischen Skalierung eines Systems eine Drosselung als vorübergehende Maßnahme verwendet wird und der Ressourcenbedarf sehr schnell ansteigt, kann das System möglicherweise nicht mehr funktionieren – selbst wenn es im gedrosselten Modus betrieben wird. Wenn dies nicht hinnehmbar ist, sollten Sie eventuell größere Kapazitätsreserven vorhalten und eine weitreichendere automatische Skalierung konfigurieren.

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

Verwenden Sie dieses Muster in folgenden Fällen:

- Um sicherzustellen, dass ein System weiterhin die Vereinbarungen zum Servicelevel erfüllt

- Um zu verhindern, dass ein einzelner Mandant die von einer Anwendung bereitgestellten Ressourcen monopolisiert

- Um Aktivitätsspitzen zu verarbeiten

- Um die Kosten für ein System durch Beschränkung des maximalen Ressourcenbedarfs zu optimieren, der für die Aufrechterhaltung seiner Funktionsfähigkeit erforderlich ist

## <a name="example"></a>Beispiel

Die letzte Abbildung zeigt, wie eine Drosselung in einem mehrinstanzenfähigen System implementiert werden kann. Benutzer der einzelnen Mandantenorganisationen greifen auf eine in der Cloud gehostete Anwendung zu, in der sie Umfragen ausfüllen und einreichen. Die Anwendung beinhaltet eine Instrumentierung, die die Geschwindigkeit überwacht, mit der diese Benutzer Anforderungen an die Anwendung senden.

Um zu verhindern, dass Benutzer eines Mandanten die Reaktionsfähigkeit und Verfügbarkeit der Anwendung für alle anderen Benutzer beeinträchtigen, wird die Anzahl der Anforderungen, die die Benutzer eines beliebigen Mandanten pro Sekunde senden können, begrenzt. Die Anwendung blockiert Anforderungen, die diesen Grenzwert überschreiten.

![Abbildung 3: Implementieren der Drosselung in einer mehrinstanzenfähigen Anwendung](./_images/throttling-multi-tenant.png)


## <a name="related-patterns-and-guidance"></a>Zugehörige Muster und Anleitungen

Die folgenden Muster und Anleitungen können auch relevant sein, wenn dieses Muster implementiert wird:
- [Instrumentations- und Telemetrieanleitungen](https://msdn.microsoft.com/library/dn589775.aspx). Für die Drosselung müssen Informationen über die Nutzlast eines Diensts gesammelt werden. Dieser Leitfaden beschreibt, wie benutzerdefinierte Überwachungsinformationen generiert und erfasst werden.
- [Leitfaden zur Dienstmessung](https://msdn.microsoft.com/library/dn589796.aspx): Dieser Leitfaden beschreibt, wie die Nutzung von Diensten gemessen wird, um Einblick über deren Verwendung zu erhalten. Diese Informationen können als Entscheidungshilfe zur Auswahl einer Drosselungsmethode für einen Dienst dienen.
- [Leitfaden für die automatische Skalierung](https://msdn.microsoft.com/library/dn589774.aspx): Die Drosselung kann als Übergangsmaßnahme während der automatischen Skalierung eines Systems oder zur Vermeidung der automatischen Skalierung eines Systems verwendet werden. Dieser Leitfaden enthält Informationen über Strategien zur automatischen Skalierung.
- [Muster „Warteschlangenbasierter Lastenausgleich“](queue-based-load-leveling.md): Ein warteschlangenbasierter Lastenausgleich ist eine häufig verwendete Methode zur Implementierung der Drosselung. Eine Warteschlange kann als Puffer herangezogen werden, um die Geschwindigkeit auszugleichen, mit der die von einer Anwendung gesendeten Anforderungen an einen Dienst gesendet werden.
- [Muster „Prioritätswarteschlange“][]: Ein System kann im Rahmen seiner Drosselungsstrategie Prioritätswarteschlangen einsetzen, um die Leistung von kritischen oder höherwertigen Anwendungen aufrechtzuerhalten und gleichzeitig die von weniger wichtigen Anwendungen zu reduzieren.

[Muster „Prioritätswarteschlange“]: priority-queue.md