---
title: Konkurrierende Consumer
description: "Mehreren gleichzeitigen Consumern die Verarbeitung von Nachrichten ermöglichen, die auf dem gleichen Messagingkanal empfangen werden"
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: messaging
ms.openlocfilehash: d72a09ef7613bebe3701634e4eac0716400e471d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="competing-consumers-pattern"></a>Muster „Konkurrierende Consumer“

[!INCLUDE [header](../_includes/header.md)]

Mehreren gleichzeitigen Consumern die Verarbeitung von Nachrichten ermöglichen, die auf dem gleichen Messagingkanal empfangen werden Dadurch wird der Durchsatz optimiert, Skalierbarkeit und Verfügbarkeit werden erhöht, und die Workload wird ausgeglichen.

## <a name="context-and-problem"></a>Kontext und Problem

Eine in der Cloud ausgeführte Anwendung soll eine große Anzahl von Anforderungen verarbeiten. Anstatt jede Anforderung synchron zu verarbeiten, ist es ein gängiges Verfahren, die Anforderungen über ein Messagingsystem an einen anderen Dienst (einen Consumerdienst) zu übergeben, der sie asynchron verarbeitet. Diese Strategie trägt dazu bei, dass die Geschäftslogik in der Anwendung nicht blockiert wird, während Anforderungen verarbeitet werden.

Die Anzahl der Anforderungen kann im Lauf der Zeit aus verschiedenen Gründen stark abweichen. Ein plötzlicher Anstieg der Benutzeraktivität oder der aggregierten Anfragen von mehreren Mandanten kann zu einer nicht vorhersagbaren Workload führen. Zu Spitzenzeiten muss ein System möglicherweise viele Hunderte von Anforderungen pro Sekunde verarbeiten, während zu anderen Zeiten die Anzahl sehr klein werden kann. Darüber hinaus kann sich die Art der durchgeführten Arbeit für die Verarbeitung dieser Anforderungen stark unterscheiden. Die Verwendung einer einzigen Instanz des Consumerdiensts kann dazu führen, dass diese Instanz eine übermäßig hohe Anzahl von Anforderungen verarbeiten muss, oder das Nachrichtensystem kann möglicherweise durch einen Zustrom von Nachrichten von der Anwendung überlastet werden. Zur Verarbeitung dieser schwankenden Workload kann das System mehrere Instanzen des Consumerdiensts ausführen. Diese Consumer müssen jedoch koordiniert werden, um sicherzustellen, dass jede Nachricht nur an einen einzelnen Consumer übermittelt wird. Außerdem muss für die Workload ein Lastenausgleich über alle Consumer stattfinden, um zu verhindern, dass eine Instanz zu einem Engpass wird.

## <a name="solution"></a>Lösung

Implementieren Sie den Kommunikationskanal zwischen der Anwendung und den Instanzen des Consumerdiensts mithilfe einer Warteschlange. Die Anwendung sendet Anforderungen in Form von Nachrichten an die Warteschlange, und die Consumerdienstinstanzen empfangen Nachrichten aus der Warteschlange und verarbeiten diese. Durch diesen Ansatz kann derselbe Pool von Consumerdienstinstanzen Nachrichten von jeder Instanz der Anwendung verarbeiten. Die Abbildung veranschaulicht die Verwendung einer Nachrichtenwarteschlange zur Verteilung von Arbeit an Instanzen eines Diensts.

![Verwenden einer Nachrichtenwarteschlange zur Verteilung von Arbeit an Instanzen eines Diensts](./_images/competing-consumers-diagram.png)

Diese Lösung hat folgende Vorteile:

- Sie bietet ein für unterschiedliche Lasten abgeglichenes System, das starke Schwankungen in der Menge der von Anwendungsinstanzen gesendeten Anfragen verarbeiten kann. Die Warteschlange fungiert als Puffer zwischen den Anwendungsinstanzen und den Consumerdienstinstanzen. Dies kann dazu beitragen, die Auswirkungen auf die Verfügbarkeit und Reaktionsfähigkeit der Anwendung und der Dienstinstanzen zu minimieren, wie unter [Warteschlangenbasiertes Lastenausgleichsmuster](queue-based-load-leveling.md) beschrieben. Eine lang andauernde Verarbeitung einer Nachricht behindert nicht die gleichzeitige Verarbeitung anderer Nachrichten durch andere Instanzen des Consumerdiensts.

- Dadurch wird die Zuverlässigkeit verbessert. Wenn ein Produzent, anstatt dieses Muster zu verwenden, direkt mit einem Consumer kommuniziert, diesen jedoch nicht überwacht, besteht eine hohe Wahrscheinlichkeit, dass Nachrichten verloren gehen oder nicht verarbeitet werden, wenn beim Consumer ein Fehler auftritt. Bei diesem Muster werden Nachrichten nicht an eine bestimmte Dienstinstanz gesendet. Wenn bei einer Dienstinstanz ein Fehler auftritt, wird kein Produzent blockiert, und Nachrichten können von jeder funktionsfähigen Dienstinstanz verarbeitet werden.

- Es ist keine komplexe Koordination zwischen den Consumern oder zwischen dem Produzenten und den Consumerinstanzen erforderlich. Die Nachrichtenwarteschlange stellt sicher, dass jede Nachricht mindestens einmal übermittelt wird.

- Sie ist skalierbar. Das System kann die Anzahl der Instanzen des Consumerdiensts dynamisch erhöhen oder verringern, wenn die Anzahl der Nachrichten schwankt.

- Dies kann die Resilienz verbessern, wenn die Nachrichtenwarteschlange transaktionale Lesevorgänge bereitstellt. Wenn eine Consumerdienstinstanz die Nachricht als Teil eines Transaktionsvorgang liest und verarbeitet und bei der Consumerdienstinstanz ein Fehler auftritt, kann dieses Muster sicherstellen, dass die Nachricht an die Warteschlange zurückgegeben wird, wo sie von einer anderen Instanz des Consumerdiensts abgerufen und verarbeitet werden kann.

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:

- **Nachrichtensortierung:** Die Reihenfolge, in der Consumerdienstinstanzen Nachrichten empfangen, ist nicht garantiert und entspricht nicht unbedingt der Reihenfolge, in der die Nachrichten erstellt wurden. Entwerfen Sie das System so, dass die Verarbeitung von Nachrichten mit Sicherheit idempotent ist, da auf diese Weise jede Abhängigkeit von der Reihenfolge der Nachrichtenverarbeitung vermieden werden kann. Weitere Informationen finden Sie unter [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) (Idempotenzmuster) im Blog von Jonathan Oliver.

    > Microsoft Azure Service Bus-Warteschlangen können eine garantierte FIFO-Sortierung (First In, First Out) von Nachrichten mithilfe von Nachrichtensitzungen implementieren. Weitere Informationen finden Sie unter [Messagingmuster mithilfe von Sitzungen](https://msdn.microsoft.com/magazine/jj863132.aspx).

- **Entwerfen von Diensten für Resilienz:** Wenn das System für das Erkennen und Neustarten von fehlerhaften Dienstinstanzen ausgelegt ist, kann es erforderlich sein, die von den Dienstinstanzen durchgeführte Verarbeitung als idempotente Vorgänge zu implementieren, damit es minimale Auswirkungen hat, wenn eine einzelne Nachricht mehrmals abgerufen und verarbeitet wird.

- **Erkennen von nicht verarbeitbaren Nachrichten:** Eine falsch formatierte Nachricht oder eine Aufgabe, die Zugriff auf nicht verfügbare Ressourcen benötigt, kann zu Fehlern in einer Dienstinstanz führen. Das System sollte verhindern, dass solche Nachrichten an die Warteschlange zurückgegeben werden, und stattdessen die Details dieser Nachrichten an anderer Stelle erfassen und speichern, damit sie bei Bedarf analysiert werden können.

- **Verarbeiten von Ergebnissen:** Die Dienstinstanz, die eine Nachricht verarbeitet, ist vollständig von der Anwendungslogik, die die Nachricht generiert, entkoppelt und kann möglicherweise nicht direkt mit dieser kommunizieren. Wenn die Dienstinstanz Ergebnisse generiert, die an die Anwendungslogik zurückgegeben werden müssen, müssen diese Informationen an einem Ort gespeichert werden, der für beide zugänglich ist. Um zu verhindern, dass die Anwendungslogik unvollständige Daten abruft, muss das System angeben, wann die Verarbeitung abgeschlossen ist.

     > Wenn Sie Azure verwenden, kann ein Workerprozess mithilfe einer dedizierten Nachrichtenantwort-Warteschlange Ergebnisse zurück an die Anwendungslogik übergeben. Die Anwendungslogik muss in der Lage sein, diese Ergebnisse mit der ursprünglichen Nachricht zu korrelieren. Dieses Szenario wird unter [Einführung in asynchrone Nachrichten](https://msdn.microsoft.com/library/dn589781.aspx) ausführlicher beschrieben.

- **Skalieren des Messagingsystems:** In einer groß angelegten Lösung könnte eine einzelne Warteschlange durch die Anzahl von Nachrichten überlastet werden und einen Engpass im System darstellen. Ziehen Sie in diesem Fall in Betracht, das Nachrichtensystem zu partitionieren, sodass Nachrichten von bestimmten Produzenten an eine bestimmte Warteschlange gesendet werden, oder verwenden Sie einen Lastenausgleich, um Nachrichten auf mehrere Warteschlangen zu verteilen.

- **Sicherstellen der Zuverlässigkeit des Messagingsystems:** Ein zuverlässiges Messagingsystem muss gewährleisten, dass eine Nachricht nach dem Einstellen in eine Warteschlange durch eine Anwendung nicht verloren geht. Dies ist besonders wichtig, um sicherzustellen, dass alle Nachrichten mindestens einmal übermittelt werden.

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

Verwenden Sie dieses Muster in folgenden Fällen:

- Die Workload für eine Anwendung wird in Aufgaben unterteilt, die asynchron ausgeführt werden können.
- Aufgaben sind unabhängig voneinander und können parallel ausgeführt werden.
- Die Menge der Arbeit kann stark schwanken, sodass eine skalierbare Lösung erforderlich ist.
- Die Lösung muss Hochverfügbarkeit bereitstellen und stabil bleiben, wenn bei der Verarbeitung einer Aufgabe ein Fehler auftritt.

Dieses Muster ist in folgenden Fällen möglicherweise nicht geeignet:

- Es ist nicht einfach, die Anwendungsworkload in einzelne Aufgaben zu trennen, oder es besteht eine starke Abhängigkeit zwischen Aufgaben.
- Aufgaben müssen synchron ausgeführt werden, und die Anwendungslogik muss vor dem Fortfahren warten, bis eine Aufgabe abgeschlossen ist.
- Aufgaben müssen in einer bestimmten Reihenfolge ausgeführt werden.

> Einige Messagingsysteme unterstützen Sitzungen, mit denen ein Produzent Nachrichten gruppieren und sicherstellen kann, dass sie alle vom selben Consumer verarbeitet werden. Dieser Mechanismus kann mit priorisierte Nachrichten verwendet werden (sofern diese unterstützt werden), um eine Form der Nachrichtensortierung zu implementieren, bei der Nachrichten von einem Produzenten an einen einzelnen Consumer nacheinander übermittelt werden.

## <a name="example"></a>Beispiel

Azure bietet Speicherwarteschlangen und Service Bus-Warteschlangen, die als Mechanismus für die Implementierung dieses Musters verwendet werden können. Die Anwendungslogik kann Nachrichten an eine Warteschlange senden, und Consumer, die als Aufgaben in Rollen implementiert sind, können Nachrichten aus dieser Warteschlange abrufen und verarbeiten. Aus Gründen der Resilienz kann ein Consumer in einer Service Bus-Warteschlange den `PeekLock`-Modus verwenden, wenn er eine Nachricht aus der Warteschlange abruft. In diesem Modus wird die Nachricht nicht entfernt, sondern einfach vor anderen Consumern verborgen. Der ursprüngliche Consumer kann die Nachricht löschen, wenn er die Verarbeitung abgeschlossen hat. Wenn beim Consumer ein Fehler auftritt, kommt es beim Peek-Lock zu einem Timeout, und die Nachricht wird wieder sichtbar, sodass sie von einem anderen Consumer abgerufen werden kann.

> Ausführliche Informationen zur Verwendung von Azure Service Bus-Warteschlangen finden Sie unter [Service Bus-Warteschlangen, -Themen und -Abonnements](https://msdn.microsoft.com/library/windowsazure/hh367516.aspx).
Informationen zu Azure-Speicherwarteschlangen finden Sie unter [Erste Schritte mit Azure Queue Storage mit .NET](https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-queues/).

Der folgende Code aus der `QueueManager`-Klasse in der bei [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) verfügbaren Lösung CompetingConsumers zeigt, wie Sie mithilfe einer `QueueClient`-Instanz im Ereignishandler `Start` in einer Web- oder Workerrolle eine Warteschlange erstellen können.

```csharp
private string queueName = ...;
private string connectionString = ...;
...

public async Task Start()
{
  // Check if the queue already exists.
  var manager = NamespaceManager.CreateFromConnectionString(this.connectionString);
  if (!manager.QueueExists(this.queueName))
  {
    var queueDescription = new QueueDescription(this.queueName);

    // Set the maximum delivery count for messages in the queue. A message
    // is automatically dead-lettered after this number of deliveries. The
    // default value for dead letter count is 10.
    queueDescription.MaxDeliveryCount = 3;

    await manager.CreateQueueAsync(queueDescription);
  }
  ...

  // Create the queue client. By default the PeekLock method is used.
  this.client = QueueClient.CreateFromConnectionString(
    this.connectionString, this.queueName);
}
```

Der nächste Codeausschnitt veranschaulicht, wie eine Anwendung einen Batch von Nachrichten erstellen und an die Warteschlange senden kann.

```csharp
public async Task SendMessagesAsync()
{
  // Simulate sending a batch of messages to the queue.
  var messages = new List<BrokeredMessage>();

  for (int i = 0; i < 10; i++)
  {
    var message = new BrokeredMessage() { MessageId = Guid.NewGuid().ToString() };
    messages.Add(message);
  }
  await this.client.SendBatchAsync(messages);
}
```

Der folgende Code zeigt, wie eine Consumerdienstinstanz über einen ereignisgesteuerten Ansatz Nachrichten aus der Warteschlange abrufen kann. Der `processMessageTask`-Parameter für die `ReceiveMessages`-Methode ist ein Delegat, der auf den Code verweist, der beim Empfang einer Nachricht ausgeführt werden soll. Dieser Code wird asynchron ausgeführt.

```csharp
private ManualResetEvent pauseProcessingEvent;
...

public void ReceiveMessages(Func<BrokeredMessage, Task> processMessageTask)
{
  // Set up the options for the message pump.
  var options = new OnMessageOptions();

  // When AutoComplete is disabled it's necessary to manually
  // complete or abandon the messages and handle any errors.
  options.AutoComplete = false;
  options.MaxConcurrentCalls = 10;
  options.ExceptionReceived += this.OptionsOnExceptionReceived;

  // Use of the Service Bus OnMessage message pump.
  // The OnMessage method must be called once, otherwise an exception will occur.
  this.client.OnMessageAsync(
    async (msg) =>
    {
      // Will block the current thread if Stop is called.
      this.pauseProcessingEvent.WaitOne();

      // Execute processing task here.
      await processMessageTask(msg);
    },
    options);
}
...

private void OptionsOnExceptionReceived(object sender,
  ExceptionReceivedEventArgs exceptionReceivedEventArgs)
{
  ...
}
```

Beachten Sie, dass Features für automatische Skalierung, z.B. die in Azure verfügbaren, zum Starten und Beenden von Rolleninstanzen verwendet werden können, wenn die Länge der Warteschlange variiert. Weitere Informationen finden Sie unter [Anleitungen für die automatische Skalierung](https://msdn.microsoft.com/library/dn589774.aspx). Es ist außerdem nicht notwendig, eine genaue Entsprechung zwischen Rolleninstanzen und Arbeitsprozessen zu verwalten: Eine einzelne Rolleninstanz kann mehrere Arbeitsprozesse implementieren. Weitere Informationen finden Sie unter [Computeressourcen-Konsolidierungsmuster](compute-resource-consolidation.md).

## <a name="related-patterns-and-guidance"></a>Zugehörige Muster und Anleitungen

Die folgenden Muster und Anweisungen könnten für die Implementierung dieses Musters relevant sein:

- [Einführung in asynchrone Nachrichten:](https://msdn.microsoft.com/library/dn589781.aspx) Warteschlangen sind ein Mechanismus für asynchrone Kommunikation. Wenn ein Consumerdienst eine Antwort an eine Anwendung senden muss, kann es erforderlich sein, ein Antwortmessaging zu implementieren. Die Einführung in asynchrone Nachrichten enthält Informationen zum Implementieren des Anforderung-Antwort-Messaging mithilfe von Nachrichtenwarteschlangen.

- [Richtlinien für die automatische Skalierung:](https://msdn.microsoft.com/library/dn589774.aspx) Es kann möglich sein, Instanzen eines Consumerdiensts zu starten und zu beenden, da die Länge der Warteschlange, in die Anwendungen Nachrichten senden, variiert. Automatische Skalierung kann dabei helfen, den Durchsatz während Zeiten hoher Verarbeitung beizubehalten.

- [Computeressourcen-Konsolidierungsmuster:](compute-resource-consolidation.md) Eventuell ist es möglich, mehrere Instanzen eines Consumerdiensts in einem einzelnen Prozess zusammenzuführen, um Kosten und Verwaltungsaufwand zu reduzieren. Das Computeressourcen-Konsolidierungsmuster beschreibt die Vor- und Nachteile dieses Ansatzes.

- [Warteschlangenbasiertes Lastenausgleichsmuster:](queue-based-load-leveling.md) Durch die Einführung einer Nachrichtenwarteschlange kann eine höhere Resilienz des Systems erreicht werden, sodass Dienstinstanzen stark abweichende Mengen von Anforderungen von Anwendungsinstanzen verarbeiten können. Die Nachrichtenwarteschlange fungiert als Puffer, in dem die Last ausgeglichen wird. Das warteschlangenbasierte Lastenausgleichsmuster beschreibt dieses Szenario ausführlicher.

- Diesem Muster ist eine [Beispielanwendung](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) zugeordnet.
