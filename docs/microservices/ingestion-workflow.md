---
title: Erfassung und Workflow in Microservices
description: Erfassung und Workflow in Microservices
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: 6477c3f2b0cc6d37dcd4637dc0dde4f7a6e3cc74
ms.sourcegitcommit: 94c769abc3d37d4922135ec348b5da1f4bbcaa0a
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/13/2017
---
# <a name="designing-microservices-ingestion-and-workflow"></a>Entwerfen von Microservices: Erfassung und Workflow

Microservices verfügen häufig über einen Workflow, der für eine einzelne Transaktion über mehrere Dienste reicht. Der Workflow muss zuverlässig sein, und es darf nicht zu einem Verlust oder zu einem Teilabschluss von Transaktionen kommen. Außerdem ist es sehr wichtig, die Erfassungsrate für eingehende Anforderungen zu steuern. Wenn viele kleinere Dienste miteinander kommunizieren, kann eine Welle von eingehenden Anforderungen zu einer Überlastung der Kommunikation zwischen den Diensten führen. 

![](./images/ingestion-workflow.png)

## <a name="the-drone-delivery-workflow"></a>Workflow für Drohnenlieferung

In der Anwendung für die Drohnenlieferung (Drone Delivery) müssen die folgenden Vorgänge durchgeführt werden, um eine Lieferung zu planen:

1. Überprüfen des Status des Kundenkontos (Kontodienst (Account))
2. Erstellen einer neuen Paketidentität (Paketdienst (Package))
3. Überprüfen anhand der Abhol- und Lieferorte (Drittanbieter-Transportdienst (Third-party Transportation)), ob für die Lieferung ein Transport durch Dritte erforderlich ist
4. Planen einer Drohne für die Abholung (Drohnendienst (Drone))
5. Erstellen einer neuen Lieferentität (Lieferdienst (Delivery))

Dies ist der Kern der gesamten Anwendung, und der End-to-End-Prozess muss eine hohe Leistung und Zuverlässigkeit aufweisen. Es müssen einige besondere Anforderungen erfüllt werden:

- **Belastungsausgleich**: Zu viele Clientanforderungen können dazu führen, dass das System mit dem Netzwerkdatenverkehr zwischen den Diensten überlastet wird. Außerdem kann es auch für abhängige Back-End-Komponenten, z.B. Speicher- oder Remotedienste, zu einer Überlastung kommen. Als Reaktion darauf werden ggf. die aufrufenden Dienste gedrosselt, sodass im System Rückstaus entstehen. Daher ist es wichtig, für die eingehenden Anforderungen des Systems einen Belastungsausgleich durchzuführen, indem diese zur Verarbeitung in einem Puffer oder einer Warteschlange angeordnet werden. 

- **Garantierte Übermittlung**: Um zu verhindern, dass Clientanforderungen verloren gehen, muss für die Erfassungskomponente eine einmalige Zustellung garantiert sein. 

- **Fehlerbehandlung**: Wenn für einen Dienst ein Fehlercode oder ein nicht vorübergehender Fehler auftritt, kann die Zustellung nicht geplant werden. Ein Fehlercode kann auf einen erwarteten Fehlerzustand (z.B. die Aussetzung des Kundenkontos) oder einen unerwarteten Serverfehler (HTTP 5xx) hinweisen. Ein Dienst kann auch ggf. nicht verfügbar sein, sodass es für den Netzwerkaufruf zu einer Zeitüberschreitung kommt. 

Zuerst sehen wir uns die Erfassungsseite der Gleichung an, also wie das System eingehende Benutzeranforderungen bei hohem Durchsatz erfassen kann. Anschließend beschäftigen wir uns damit, wie mit der Anwendung für die Drohnenlieferung ein zuverlässiger Workflow implementiert werden kann. Wir kommen zu der Erkenntnis, dass sich der Entwurf des Subsystems für die Erfassung auf das Workflow-Back-End auswirkt. 

## <a name="ingestion"></a>Erfassung

Basierend auf den geschäftlichen Anforderungen hat das Entwicklungsteam die folgenden nicht funktionsbezogenen Anforderungen für die Erfassung identifiziert:

- Dauerhafter Durchsatz von 10.000 Anforderungen/Sekunde
- Fähigkeit zur Verarbeitung von Spitzen im Bereich von bis zu 50.000 Anforderungen/Sekunde ohne Verlust von Clientanforderungen oder Zeitüberschreitung
- Wartezeit von weniger als 500 ms im 99. Perzentil

Die Anforderung zur Verarbeitung von gelegentlichen Datenverkehrsspitzen führt zu einer besonderen Entwurfsanforderung. Theoretisch kann das System horizontal hochskaliert werden, um den maximal zu erwartenden Datenverkehr verarbeiten zu können. Bei dieser Art der Bereitstellung wären aber viele Ressourcen sehr ineffizient. Da die Anwendung in den meisten Fällen nicht so viel Kapazität benötigt, befinden sich Kerne im Leerlauf. Dies kostet Geld, ohne dass sich ein Nutzen daraus ergibt.

Ein besserer Ansatz besteht darin, die eingehenden Anforderungen in einem Puffer anzuordnen und den Puffer als Komponente für den Belastungsausgleich einzusetzen. Bei diesem Entwurf muss der Erfassungsdienst die maximale Erfassungsrate über kurze Zeiträume hinweg verarbeiten können, während die Back-End-Dienste nur die maximale dauerhafte Belastung bewältigen müssen. Durch die Pufferung am Front-End ist es normalerweise nicht erforderlich, dass die Back-End-Dienste hohe Datenverkehrsspitzen verarbeiten. Auf der Skalierungsebene, die für die Anwendung für die Drohnenlieferung benötigt wird, ist [Azure Event Hubs](/azure/event-hubs/) eine gute Wahl für den Belastungsausgleich. Event Hubs ermöglicht geringe Wartezeiten und einen hohen Durchsatz sowie eine kostengünstige Lösung bei hohen Erfassungsvolumen. 

Für unsere Tests haben wir einen Event Hub der Standard-Ebene mit 32 Partitionen und 100 Durchsatzeinheiten verwendet. Hierbei konnten wir eine Erfassung von ca. 32.000 Ereignissen pro Sekunde mit einer Latenz von ca. 90 ms beobachten. Das Standardlimit beträgt derzeit 20 Durchsatzeinheiten, aber Azure-Kunden können weitere Durchsatzeinheiten anfordern, indem Sie eine Supportanfrage erstellen. Weitere Informationen finden Sie unter [Event Hubs-Kontingente](/azure/event-hubs/event-hubs-quotas). Wie bei allen Leistungsmetriken können sich viele Faktoren auf die Leistung auswirken, z.B. die Größe der Nachrichtennutzlast. Diese Zahlen sollten also nicht als Benchmark-Werte interpretiert werden. Wenn ein höherer Durchsatz benötigt wird, kann der Erfassungsdienst per Sharding über mehr als einen Event Hub verteilt werden. Falls noch höhere Durchsatzraten erforderlich sind, sind mit [Event Hubs Dedicated](/azure/event-hubs/event-hubs-dedicated-overview) Bereitstellungen mit Einzelmandant möglich, bei denen über 2 Millionen Ereignisse pro Sekunde eingehen können.

Es ist wichtig zu verstehen, wie für Event Hubs ein solch hoher Durchsatz erzielt werden kann, da sich dies darauf auswirkt, wie ein Client Nachrichten von Event Hubs nutzen sollte. Bei Event Hubs wird keine *Warteschlange* implementiert. Stattdessen wird ein *Ereignisdatenstrom* implementiert. 

Mit einer Warteschlange kann ein einzelner Consumer eine Nachricht aus der Warteschlange entfernen, sodass der nächste Consumer diese Nachricht nicht sieht. Bei Warteschlangen können Sie daher ein [Muster „Konkurrierende Consumer“](../patterns/competing-consumers.md) verwenden, um Nachrichten parallel zu verarbeiten und die Skalierbarkeit zu verbessern. Zur Erhöhung der Resilienz sperrt der Consumer die Nachricht und hebt die Sperre auf, wenn er die Verarbeitung der Nachricht abgeschlossen hat. Falls für den Consumer ein Fehler auftritt – z.B. der zugrunde liegende Knoten abstürzt –, tritt für die Sperre eine Zeitüberschreitung auf, und die Nachricht wird wieder in die Warteschlange eingereiht. 

![](./images/queue-semantics.png)

Für Event Hubs wird dagegen eine Streamingsemantik genutzt. Consumer lesen den Datenstrom unabhängig mit ihrer eigenen Geschwindigkeit. Jeder Consumer ist dafür verantwortlich, seine aktuelle Position im Datenstrom nachzuverfolgen. Ein Consumer sollte seine aktuelle Position in vordefinierten Intervallen in einen beständigen Speicher schreiben. Falls für den Consumer ein Fehler auftritt (z.B. ein Absturz des Consumers oder Ausfall des Hosts), kann eine neue Instanz das Lesen des Datenstroms dann ab der zuletzt aufgezeichneten Position fortsetzen. Dieser Prozess wird als *Setzen von Prüfpunkten* bezeichnet. 

Aus Leistungsgründen führt ein Consumer das Setzen von Prüfpunkten nicht nach jeder Nachricht durch. Stattdessen erfolgt dies in festen Intervallen, z.B. nach der Verarbeitung von *n* Nachrichten oder alle *n* Sekunden. Aus diesem Grund werden einige Ereignisse bei einem Ausfall eines Consumers ggf. zweimal verarbeitet, da eine neue Instanz den Vorgang immer ab dem letzten Prüfpunkt fortsetzt. Dies ist mit einem Nachteil verbunden: Häufige Prüfpunkte können die Leistung beeinträchtigen, und eine geringe Zahl von Prüfpunkten bedeutet, dass Sie nach einem Fehler mehr Ereignisse wiederholen müssen.  

![](./images/stream-semantics.png)
 
Event Hubs ist nicht für konkurrierende Consumer ausgelegt. Es können zwar mehrere Consumer einen Datenstrom lesen, aber jeder durchläuft den Datenstrom unabhängig von den anderen. Stattdessen wird für Event Hubs das Muster „Partitionierter Consumer“ verwendet. Ein Event Hub verfügt über bis zu 32 Partitionen. Die horizontale Skalierung wird erzielt, indem jeder Partition ein separater Consumer zugewiesen wird.

Was bedeutet dies für den Workflow für die Drohnenlieferung? Damit sich der volle Nutzen von Event Hubs entfaltet, kann der Scheduler für die Lieferung nicht warten, bis jede Nachricht verarbeitet wurde, bevor mit der nächsten Nachricht fortgefahren wird. Falls diese Vorgehensweise gewählt wird, verbringt der Scheduler einen Großteil seiner Zeit mit dem Warten auf den Abschluss von Netzwerkaufrufen. Stattdessen müssen Batches mit Nachrichten parallel verarbeitet werden, indem asynchrone Aufrufe der Back-End-Dienste durchgeführt werden. Wie wir sehen werden, ist auch die Auswahl der richtigen Strategie für das Setzen von Prüfpunkten wichtig.  

## <a name="workflow"></a>Workflow

Wir haben uns drei Optionen zum Lesen und Verarbeiten der Nachrichten angesehen: Ereignisprozessorhost, Service Bus-Warteschlangen und die IoTHub React-Bibliothek. Wir haben IoTHub React gewählt, beginnen aber mit dem Ereignisprozessorhost, um diese Wahl zu verdeutlichen. 

### <a name="event-processor-host"></a>Ereignisprozessorhost

Der Ereignisprozessorhost ist für die Batchverarbeitung von Nachrichten ausgelegt. Die Anwendung implementiert die `IEventProcessor`-Schnittstelle, und der Prozessorhost erstellt eine Ereignisprozessorinstanz für jede Partition im Event Hub. Der Ereignisprozessorhost ruft dann jede `ProcessEventsAsync`-Methode des Ereignisprozessors mit Batches von Ereignisnachrichten auf. Die Anwendung steuert, wann in der `ProcessEventsAsync`-Methode Prüfpunkte gesetzt werden, und der Ereignisprozessorhost schreibt die Prüfpunkte in den Azure-Speicher. 

Innerhalb einer Partition wartet der Ereignisprozessorhost darauf, dass `ProcessEventsAsync` zurückgegeben wird, bevor der Aufruf mit dem nächsten Batch erneut durchgeführt wird. Mit diesem Ansatz wird das Programmiermodell vereinfacht, weil Ihr Ereignisverarbeitungscode nicht eintrittsinvariant sein muss. Dies bedeutet aber auch, dass der Ereignisprozessor jeweils nur einen Batch verarbeitet, und dies wirkt sich negativ auf die Geschwindigkeit aus, mit der der Prozessorhost Nachrichten senden kann.

> [!NOTE] 
> Der Prozessorhost muss hierbei nicht tatsächlich (im Sinne einer Thread-Blockierung) *warten*. Da die `ProcessEventsAsync`-Methode asynchron ist, können vom Prozessorhost andere Arbeitsschritte ausgeführt werden, während die Verarbeitung der Methode abgeschlossen wird. Für diese Partition wird aber kein weiterer Batch mit Nachrichten bereitgestellt, bis die Methode zurückgegeben wurde. 

In der Drohnenanwendung kann ein Batch mit Nachrichten parallel verarbeitet werden. Das Warten auf den Abschluss der Verarbeitung des gesamten Batchs kann aber trotzdem zu einem Engpass führen. Die Verarbeitung kann nur so schnell wie die langsamste Nachricht im Batch erfolgen. Jegliche Abweichung bei den Antwortzeiten kann zu einem „Long Tail“-Zustand führen, bei dem einige langsame Antworten das gesamte System beeinträchtigen. Unsere Leistungstests haben ergeben, dass wir mit diesem Ansatz unseren Zieldurchsatz nicht erreichen konnten. Dies bedeutet *nicht*, dass Sie die Verwendung des Ereignisprozessorhosts vermeiden sollten. Vermeiden Sie in der `ProcesssEventsAsync`-Methode aber Aufgaben mit langer Ausführungsdauer, um einen hohen Durchsatz zu erzielen. Sorgen Sie dafür, dass jeder Batch schnell verarbeitet wird.

### <a name="iothub-react"></a>IotHub React 

[IotHub React](https://github.com/Azure/toketi-iothubreact) ist eine Akka Streams-Bibliothek zum Lesen von Ereignissen aus Event Hub. Akka Streams ist ein datenstrombasiertes Programmierframework, mit dem die [Reactive Streams](http://www.reactive-streams.org/)-Spezifikation implementiert wird. Es ermöglicht die Erstellung von effizienten Streamingpipelines, bei denen alle Streamingvorgänge asynchron durchgeführt werden und die Pipeline Rückstaus korrekt verarbeiten kann. Zu Rückstaus kommt es, wenn eine Ereignisquelle Ereignisse schneller produziert, als diese von nachgeschalteten Consumern empfangen werden können. Dies ist genau die Situation, in der für das System für die Drohnenlieferung eine Datenverkehrsspitze auftritt. Wenn sich die Back-End-Dienste verlangsamen, gilt dies auch für IoTHub React. Wenn die Kapazität erhöht wird, sendet IoTHub React mehr Nachrichten über die Pipeline.

Akka Streams ist außerdem ein außerordentlich natürliches Programmiermodell zum Streamen von Ereignissen aus Event Hubs. Anstatt eine Schleife durch einen Batch mit Ereignissen durchzuführen, definieren Sie eine Gruppe von Vorgängen, die auf jedes Ereignis angewendet werden, und überlassen Akka Streams das Streaming. Von Akka Streams wird eine Streamingpipeline basierend auf *Quellen*, *Datenflüssen* und *Senken* definiert. Eine Quelle generiert einen Ausgabestream, ein Datenfluss verarbeitet einen Eingabestream und erstellt einen Ausgabestream, und eine Senke nutzt einen Datenstrom, ohne eine Ausgabe zu produzieren.

Hier ist der Code des Scheduler-Diensts angegeben, mit dem die Akka Streams-Pipeline eingerichtet wird:

```java
IoTHub iotHub = new IoTHub();
Source<MessageFromDevice, NotUsed> messages = iotHub.source(options);

messages.map(msg -> DeliveryRequestEventProcessor.parseDeliveryRequest(msg))
        .filter(ad -> ad.getDelivery() != null).via(deliveryProcessor()).to(iotHub.checkpointSink())
        .run(streamMaterializer);
```

Mit diesem Code wird Event Hubs als Quelle konfiguriert. Mit der `map`-Anweisung wird jede Ereignisnachricht in eine Java-Klasse deserialisiert, die für eine Lieferanforderung steht. Die `filter`-Anweisung entfernt alle `null`-Objekte aus dem Datenstrom. Dies dient als Schutz vor einem Fall, in dem eine Nachricht nicht deserialisiert werden kann. Die `via`-Anweisung verknüpft die Quelle mit einem Datenfluss, bei dem jede Lieferanforderung verarbeitet wird. Die `to`-Methode verknüpft den Datenfluss mit der Prüfpunktsenke, die in IoTHub React integriert ist.

Für IoTHub React wird eine andere Strategie für das Setzen von Prüfpunkten als für den Ereignisprozessorhost verwendet. Prüfpunkte werden von der Prüfpunktsenke geschrieben, also der abschließenden Phase in der Pipeline. Das Design von Akka Streams ermöglicht der Pipeline das Fortsetzen des Datenstromvorgangs, während die Senke den Prüfpunkt schreibt. Dies bedeutet, dass die Upstream-Verarbeitungsphasen nicht warten müssen, bis das Setzen von Prüfpunkten durchgeführt wird. Sie können konfigurieren, dass das Setzen von Prüfpunkten nach einer Zeitüberschreitung oder nach der Verarbeitung einer bestimmten Anzahl von Nachrichten erfolgt.

Mit der `deliveryProcessor`-Methode wird der Akka Streams-Datenfluss erstellt:  

```java
private static Flow<AkkaDelivery, MessageFromDevice, NotUsed> deliveryProcessor() {
    return Flow.of(AkkaDelivery.class).map(delivery -> {
        CompletableFuture<DeliverySchedule> completableSchedule = DeliveryRequestEventProcessor
                .processDeliveryRequestAsync(delivery.getDelivery(), 
                        delivery.getMessageFromDevice().properties());
        
        completableSchedule.whenComplete((deliverySchedule,error) -> {
            if (error!=null){
                Log.info("failed delivery" + error.getStackTrace());
            }
            else{
                Log.info("Completed Delivery",deliverySchedule.toString());
            }
                                
        });
        completableSchedule = null;
        return delivery.getMessageFromDevice();
    });
}
```

Der Datenfluss ruft eine statische `processDeliveryRequestAsync`-Methode auf, mit der die eigentliche Arbeit zur Verarbeitung jeder Nachricht durchgeführt wird.

### <a name="scaling-with-iothub-react"></a>Skalieren mit IoTHub React

Der Scheduler-Dienst ist so konzipiert, dass jede Containerinstanz von einer einzelnen Partition liest. Wenn die Event Hub-Instanz beispielsweise über 32 Partitionen verfügt, wird der Scheduler-Dienst mit 32 Replikaten bereitgestellt. Dies ermöglicht in Bezug auf die horizontale Skalierung eine hohe Flexibilität. 

Je nach Größe des Clusters kann auf einem Knoten im Cluster mehr als ein Scheduler-Dienstpod ausgeführt werden. Falls der Scheduler-Dienst mehr Ressourcen benötigen sollte, kann der Cluster aber horizontal hochskaliert werden, um die Pods auf mehr Knoten zu verteilen. Unsere Leistungstests haben ergeben, dass der Scheduler-Dienst speicher- und threadgebunden ist, sodass die Leistung stark von der VM-Größe und der Anzahl von Pods pro Knoten abhängig war.

Jede Instanz muss über die Informationen dazu verfügen, von welcher Event Hubs-Partition gelesen werden soll. Zum Konfigurieren der Partitionsnummer haben wir den Ressourcentyp [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) in Kubernetes genutzt. Pods in einem StatefulSet verfügen über einen beständigen Bezeichner, der einen numerischen Index enthält. Der genaue Podname lautet `<statefulset name>-<index>`, und dieser Wert ist über die [Downward-API](https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/) von Kubernetes für den Container verfügbar. Zur Laufzeit liest der Scheduler-Dienst den Podnamen und verwendet den Podindex als Partitions-ID.

Falls Sie den Scheduler-Dienst noch stärker horizontal hochskalieren müssen, können Sie mehr als einen Pod pro Event Hub-Partition zuweisen, damit mehrere Pods die einzelnen Partitionen lesen. In diesem Fall liest aber jede Instanz alle Ereignisse in der zugewiesenen Partition. Zur Vermeidung einer doppelten Verarbeitung müssten Sie einen Hashalgorithmus verwenden, damit jede Instanz einen Teil der Nachrichten überspringt. Auf diese Weise können mehrere Leser den Datenstrom nutzen, aber jede Nachricht wird nur von einer Instanz verarbeitet. 
 
![](./images/eventhub-hashing.png)

### <a name="service-bus-queues"></a>Service Bus-Warteschlangen

Die dritte von uns erwogene Option war das Kopieren von Nachrichten von Event Hubs in eine Service Bus-Warteschlange und das anschließende Lesen der Nachrichten aus Service Bus durch den Scheduler. Es kann ungewöhnlich erscheinen, die eingehenden Anforderungen in Event Hubs nur zu schreiben, um sie in Service Bus zu kopieren.  Die Idee bestand aber darin, die unterschiedlichen Stärken der einzelnen Dienste zu nutzen: Verwendung von Event Hubs zum Absorbieren von hohen Datenverkehrsspitzen, während die Warteschlangensemantik in Service Bus verwendet wird, um die Workload nach dem Muster „Konkurrierende Consumer“ zu verarbeiten. Bedenken Sie, dass unser Ziel für den dauerhaften Durchsatz unterhalb unserer erwarteten Spitzenlast liegt. Die Service Bus-Warteschlange muss also nicht so schnell wie die Erfassung von Nachrichten sein.
 
Bei diesem Ansatz wurden für unsere Proof of Concept-Implementierung eine Menge von ca. 4.000 Vorgängen pro Sekunde erzielt. Für diese Tests wurden Back-End-Pseudodienste verwendet, mit denen keine richtigen Arbeitsschritte ausgeführt wurden, sondern nur eine feste Menge von Wartezeit pro Dienst hinzugefügt wurde. Beachten Sie hierbei, dass unsere Leistungszahlen deutlich unterhalb des theoretisch möglichen Höchstwerts für Service Bus lagen. Mögliche Gründe für die Abweichung sind:

- Fehlen von optimalen Werten für verschiedene Clientparameter, z.B. Grenzwert für den Verbindungspool, Parallelitätsgrad, Vorabrufanzahl und Batchgröße

- Netzwerk-E/A-Engpässe

- Verwendung des [PeekLock](/rest/api/servicebus/peek-lock-message-non-destructive-read)-Modus anstelle von [ReceiveAndDelete](/rest/api/servicebus/receive-and-delete-message-destructive-read), der zur Sicherstellung der mindestens einmaligen Zustellung von Nachrichten benötigt wurde

Weitere Leistungstests hätten uns ggf. zur Grundursache geführt und es uns ermöglicht, diese Probleme zu lösen. Da mit IotHub React aber unser Leistungsziel erreicht wurde, haben wir uns für diese Option entschieden. Service Bus ist jedoch ebenfalls eine sinnvolle Option für dieses Szenario.

## <a name="handling-failures"></a>Behandeln von Fehlern 

Es gibt drei allgemeine Fehlerklassen, die berücksichtigt werden müssen.

1. Ein nachgeschalteter Dienst kann unter Umständen einen nicht vorübergehenden Fehler aufweisen. Dies ist jeder Fehler, der normalerweise nicht von selbst wieder behoben werden kann. Zu den nicht vorübergehenden Fehlern gehören auch normale Fehlerbedingungen, z.B. die ungültige Eingabe in eine Methode. Außerdem gehören dazu nicht behandelte Ausnahmen im Anwendungscode oder ein Prozessabsturz. Wenn diese Art von Fehler auftritt, muss die gesamte geschäftliche Transaktion als Fehler gekennzeichnet werden. Es kann erforderlich sein, andere Schritte in derselben Transaktion rückgängig zu machen, die bereits erfolgreich durchgeführt wurden. (Siehe „Kompensierende Transaktionen“ weiter unten.)
 
2. Für einen nachgeschalteten Dienst kann es zu einem vorübergehenden Fehler kommen, z.B. zu einer Netzwerkzeitüberschreitung. Diese Fehler können häufig einfach dadurch behoben werden, indem der Aufruf wiederholt wird. Wenn der Vorgang nach einer bestimmten Anzahl von Versuchen weiterhin nicht erfolgreich ist, wird dies als nicht vorübergehender Fehler eingestuft. 

3. Der Scheduler-Dienst selbst kann einen Fehler aufweisen (z.B. aufgrund eines Knotenabsturzes). In diesem Fall startet Kubernetes eine neue Instanz des Diensts. Alle Transaktionen, die bereits in Bearbeitung waren, müssen jedoch fortgesetzt werden. 

## <a name="compensating-transactions"></a>Kompensierende Transaktionen

Wenn ein nicht vorübergehender Fehler auftritt, kann sich die aktuelle Transaktion im Zustand *Teilweise fehlgeschlagen* befinden, in dem mindestens ein Schritt bereits erfolgreich durchgeführt wurde. Falls vom Drohnendienst beispielsweise bereits eine Drohne eingeplant wurde, muss die Drohne storniert werden. In diesem Fall muss die Anwendung die erfolgreichen Schritte rückgängig machen, indem eine [kompensierende Transaktion](../patterns/compensating-transaction.md) verwendet wird. Es kann vorkommen, dass dies mit einem externen System oder sogar manuell durchgeführt werden muss. 

Wenn die Logik für die kompensierenden Transaktionen komplex ist, können Sie die Erstellung eines separaten Diensts erwägen, der für diesen Prozess zuständig ist. In der Anwendung für die Drohnenlieferung reiht der Scheduler-Dienst fehlgeschlagene Vorgänge in eine dedizierte Warteschlange ein. Ein separater Microservice, der als „Supervisor“ bezeichnet wird, liest aus dieser Warteschlange und ruft eine Stornierungs-API für die Dienste auf, die kompensiert werden müssen. Dies ist eine Variante des [Musters „Scheduler-Agent-Supervisor“][scheduler-agent-supervisor]. Der Supervisor-Dienst kann auch andere Aktionen durchführen, z.B. das Benachrichtigen des Benutzers per SMS oder E-Mail oder das Senden einer Warnung an ein Dashboard für Vorgänge. 

![](./images/supervisor.png)

## <a name="idempotent-vs-non-idempotent-operations"></a>Idempotente und nicht idempotente Vorgänge

Um den Verlust von Anforderungen zu verhindern, muss vom Scheduler-Dienst garantiert werden, dass alle Nachrichten mindestens einmal verarbeitet werden. Event Hubs können eine mindestens einmalige Zustellung garantieren, wenn die Prüfpunkte vom Client richtig gesetzt werden.

Wenn der Scheduler-Dienst abstürzt, kann dies während der Verarbeitung von einer oder mehreren Clientanforderungen passieren. Diese Nachrichten werden von einer anderen Instanz des Schedulers übernommen und erneut verarbeitet. Was passiert, wenn eine Anforderung zweimal verarbeitet wird? Es ist wichtig, die doppelte Durchführung von Arbeitsschritten zu vermeiden. Es soll schließlich verhindert werden, dass vom System zwei Drohnen für dasselbe Paket losgeschickt werden.

Ein Ansatz besteht darin, alle Vorgänge so zu entwerfen, dass sie idempotent sind. Ein Vorgang ist idempotent, wenn er mehrere Male aufgerufen werden kann, ohne dass es nach dem ersten Aufruf zu Nebenwirkungen kommt. Anders ausgedrückt: Das Ergebnis ist unabhängig davon, ob ein Client den Vorgang einmal, zweimal oder viele Male aufruft, immer gleich. Im Wesentlichen sollten doppelte Aufrufe vom Dienst ignoriert werden. Damit eine Methode mit Nebenwirkungen idempotent ist, muss der Dienst doppelte Aufrufe erkennen können. Beispielsweise können Sie festlegen, dass der Aufrufer die ID zuweist, anstatt vom Dienst eine neue ID generieren zu lassen. Der Dienst kann dann eine Überprüfung auf doppelte IDs durchführen.

> [!NOTE]
> In der HTTP-Spezifikation ist angegeben, dass GET-, PUT- und DELETE-Methoden idempotent sein müssen. Für POST-Methoden besteht keine Garantie für Idempotenz. Wenn eine POST-Methode eine neue Ressource erstellt, ist im Allgemeinen nicht garantiert, dass dieser Vorgang idempotent ist. 

Es ist nicht immer einfach, idempotente Methoden zu schreiben. Eine weitere Option ist, dass der Scheduler den Status jeder Transaktion in einem permanenten Speicher nachverfolgt. Bei jeder Verarbeitung einer Nachricht wird hierbei der Status im permanenten Speicher geprüft. Nach jedem Schritt wird das Ergebnis in den Speicher geschrieben. Dieser Ansatz kann mit Leistungsbeeinträchtigungen verbunden sein.

## <a name="example-idempotent-operations"></a>Beispiel: Idempotente Vorgänge

In der HTTP-Spezifikation ist angegeben, dass PUT-Methoden idempotent sein müssen. In der Spezifikation ist Idempotenz wie folgt definiert:

>  Eine Anforderungsmethode wird als „idempotent“ angesehen, wenn die beabsichtigte Auswirkung mehrerer identischer Anforderungen mit dieser Methode auf den Server der Auswirkung für eine einzelne Anforderung dieser Art entspricht. ([RFC 7231](https://tools.ietf.org/html/rfc7231#section-4))

Es ist wichtig, beim Erstellen einer neuen Entität den Unterschied zwischen der PUT- und POST-Semantik zu verstehen. In beiden Fällen sendet der Client im Anforderungstext eine Darstellung einer Entität. Aber die Bedeutung des URI unterscheidet sich.

- Für eine POST-Methode stellt der URI eine übergeordnete Ressource der neuen Entität dar, z.B. eine Sammlung. Zur Erstellung einer neuen Lieferung kann der URI beispielsweise `/api/deliveries` lauten. Der Server erstellt die Entität und weist sie als neuen URI zu, z.B. `/api/deliveries/39660`. Dieser URI wird im Adressheader der Antwort zurückgegeben. Jedes Mal, wenn der Client eine Anforderung sendet, erstellt der Server eine neue Entität mit einem neuen URI.

- Für eine PUT-Methode identifiziert der URI die Entität. Falls bereits eine Entität mit diesem URI vorhanden ist, ersetzt der Server die vorhandene Entität durch die Version in der Anforderung. Wenn keine Entität mit diesem URI vorhanden ist, wird sie vom Server erstellt. Angenommen, der Client sendet eine PUT-Anforderung an `api/deliveries/39660`. Es besteht die Annahme, dass keine Lieferung mit diesem URI vorhanden ist, und der Server erstellt eine neue Lieferung. Wenn der Client dann die gleiche Anforderung erneut sendet, ersetzt der Server die vorhandene Entität.

Hier ist die Implementierung der PUT-Methode durch den Delivery-Dienst angegeben. 

```csharp
[HttpPut("{id}")]
[ProducesResponseType(typeof(Delivery), 201)]
[ProducesResponseType(typeof(void), 204)]
public async Task<IActionResult> Put([FromBody]Delivery delivery, string id)
{
    logger.LogInformation("In Put action with delivery {Id}: {@DeliveryInfo}", id, delivery.ToLogInfo());
    try
    {
        var internalDelivery = delivery.ToInternal();

        // Create the new delivery entity.
        await deliveryRepository.CreateAsync(internalDelivery);

        // Create a delivery status event.
        var deliveryStatusEvent = new DeliveryStatusEvent { DeliveryId = delivery.Id, Stage = DeliveryEventType.Created };
        await deliveryStatusEventRepository.AddAsync(deliveryStatusEvent);

        // Return HTTP 201 (Created)
        return CreatedAtRoute("GetDelivery", new { id= delivery.Id }, delivery);
    }
    catch (DuplicateResourceException)
    {
        // This method is mainly used to create deliveries. If the delivery already exists then update it.
        logger.LogInformation("Updating resource with delivery id: {DeliveryId}", id);

        var internalDelivery = delivery.ToInternal();
        await deliveryRepository.UpdateAsync(id, internalDelivery);

        // Return HTTP 204 (No Content)
        return NoContent();
    }
}
```

Es wird erwartet, dass die meisten Anforderungen eine neue Entität erstellen. Die Methode ruft also optimistischerweise `CreateAsync` für das Repositoryobjekt auf und verarbeitet dann alle Ausnahmen aufgrund doppelter Ressourcen, indem stattdessen die Ressource aktualisiert wird. 

> [!div class="nextstepaction"]
> [API-Gateways](./gateway.md)

<!-- links -->

[scheduler-agent-supervisor]: ../patterns/scheduler-agent-supervisor.md