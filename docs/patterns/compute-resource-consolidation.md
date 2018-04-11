---
title: Computeressourcenkonsolidierung
description: Konsolidieren mehrerer Tasks oder Vorgänge in einer einzelnen Compute-Einheit
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
ms.openlocfilehash: 85191fc630549559f8a1395e5a8622a7a6140a2d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="compute-resource-consolidation-pattern"></a>Muster „Computeressourcenkonsolidierung“

[!INCLUDE [header](../_includes/header.md)]

Konsolidieren Sie mehrere Tasks oder Vorgänge in einer einzelnen Compute-Einheit. Dies kann die Auslastung der Computerressourcen erhöhen und die Kosten sowie den Verwaltungsaufwand bei der Computeverarbeitung in Anwendungen senken, die in der Cloud gehostet werden.

## <a name="context-and-problem"></a>Kontext und Problem

Eine Cloudanwendung implementiert häufig eine Vielzahl von Vorgängen. Bei einigen Lösungen ist es sinnvoll, zunächst dem Entwurfsprinzip der Trennung von Zuständigkeiten zu folgen und diese Vorgänge in separate Compute-Einheiten aufzuteilen, die einzeln gehostet und bereitgestellt werden (z.B. als separate App Service-Web-Apps, als separate VMs oder als separate Clouddienstrollen). Doch obwohl diese Strategie dazu beitragen kann, das logische Design der Lösung zu vereinfachen, kann die Bereitstellung einer großen Anzahl von Compute-Einheiten im Rahmen derselben Anwendung die Kosten für das Laufzeithosting erhöhen und zu einer komplexeren Systemverwaltung führen.

Die Abbildung zeigt beispielhaft die vereinfachte Struktur einer cloudbasierten Lösung, die mit mehr als einer Compute-Einheit implementiert ist. Jede Compute-Einheit wird in einer eigenen virtuellen Umgebung ausgeführt. Jede Funktion wurde als separater Task implementiert (Task A bis Task E), der in einer eigenen Compute-Einheit ausgeführt wird.

![Ausführen von Tasks in einer Cloudumgebung mit einer Gruppe dedizierter Compute-Einheiten](./_images/compute-resource-consolidation-diagram.png)


Jede Compute-Einheit verbraucht kostenpflichtige Ressourcen, auch wenn sie sich im Leerlauf befindet oder nur selten genutzt wird. Daher ist dies nicht immer die kosteneffizienteste Lösung.

In Azure betrifft dies Rollen in Cloud Services, App Services und Virtual Machines. Diese Elemente werden in einer eigenen virtuellen Umgebung ausgeführt. Das Ausführen einer Sammlung separater Rollen, Websites oder virtueller Computer, die zur Ausführung einer Reihe klar definierter Vorgänge konzipiert sind, aber als Teil einer einzigen Lösung kommunizieren und zusammenarbeiten müssen, kann dies zu einer ineffizienten Ressourcennutzung führen.

## <a name="solution"></a>Lösung

Um Kosten zu senken, die Auslastung zu erhöhen, die Kommunikationsgeschwindigkeit zu verbessern und den Verwaltungsaufwand zu senken, können mehrere Tasks oder Vorgänge in einer einzigen Compute-Einheit konsolidiert werden.

Tasks können nach Kriterien gruppiert werden, die auf den von der Umgebung bereitgestellten Features sowie den damit verbundenen Kosten basieren. Ein gängiger Ansatz besteht darin, nach Tasks zu suchen, die ein ähnliches Profil hinsichtlich ihrer Skalierbarkeit, Lebensdauer und Verarbeitungsanforderungen aufweisen. Das Gruppieren dieser Tasks ermöglicht es, sie als eine Einheit zu skalieren. Die Elastizität vieler Cloudumgebungen ermöglicht es, je nach Workload zusätzliche Instanzen einer Compute-Einheit zu starten und zu beenden. Beispielsweise bietet Azure eine automatische Skalierung, die Sie auf Rollen in Cloud Services, App Services und Virtual Machines anwenden können. Weitere Informationen finden Sie im [Leitfaden für die automatische Skalierung](https://msdn.microsoft.com/library/dn589774.aspx).

Betrachten Sie als Gegenbeispiel die folgenden zwei Tasks. Sie veranschaulichen die Verwendung der Skalierbarkeit, um zu bestimmen, welche Vorgänge nicht gruppiert werden sollten:

- Task 1 ruft seltene, zeitunempfindlichen Nachrichten ab, die an eine Warteschlange gesendet werden.
- Task 2 verarbeitet Spitzen im Netzwerkdatenverkehr.

Der zweite Task erfordert Elastizität, die das Starten und Beenden einer großen Anzahl von Instanzen der Compute-Einheit beinhalten kann. Die Anwendung derselben Skalierung auf den ersten Task würde lediglich dazu führen, dass mehr Tasks auf seltene Nachrichten in derselben Warteschlange lauschen. Dies wäre eine Vergeudung von Ressourcen.

In vielen Cloudumgebungen ist es möglich, die einer Compute-Einheit zur Verfügung stehenden Ressourcen in Bezug auf die Anzahl von CPU-Kernen, den Arbeitsspeicher, den Festplattenspeicher usw. anzugeben. Allgemein gilt: Je mehr Ressourcen angegeben werden, desto höher sind die Kosten. Um Kosteneinsparungen zu erzielen, muss eine teure Compute-Einheit maximal ausgelastet werden und darf nicht für längere Zeit inaktiv sein.

Wenn Tasks vorhanden sind, die für kurze Spitzen viel CPU-Leistung anfordern, sollten diese in einer einzigen Compute-Einheit zusammengefasst werden, die die benötigte Leistung bereitstellt. Es ist hierbei jedoch wichtig, die Anforderung zur maximalen Auslastung teurer Ressourcen gegen die Konflikte abzuwägen, die bei einer Überlastung der Ressourcen auftreten können. Rechenintensive Tasks mit langer Ausführungszeit sollten beispielsweise nicht dieselbe Compute-Einheit verwenden.

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Beachten Sie bei der Implementierung dieses Musters die folgenden Punkte:

**Skalierbarkeit und Elastizität**. Viele Cloudlösungen implementieren Skalierbarkeit und Elastizität auf Ebene der Compute-Einheit, indem sie Instanzen von Compute-Einheiten starten und beenden. Vermeiden Sie es, Tasks mit konfliktverursachenden Skalierbarkeitsanforderungen in derselben Compute-Einheit zu gruppieren.

**Lebensdauer**. Die Cloudinfrastruktur recycelt in regelmäßigen Abständen die virtuelle Umgebung, die eine Compute-Einheit hostet. Wenn in einer Compute-Einheit zu viele Tasks mit langer Ausführungszeit enthalten sind, muss die Einheit möglicherweise so konfiguriert werden, dass sie nicht vor dem Abschluss dieser Tasks recycelt wird. Alternativ können Sie die Tasks mit einem Prüfpunktansatz entwerfen. Hierdurch können die Tasks sauber beendet und an dem Punkt fortgesetzt werden, an dem sie beim Neustart der Compute-Einheit unterbrochen wurden.

**Releaserhythmus**. Wenn sich die Implementierung oder Konfiguration eines Tasks häufig ändert, muss die Compute-Einheit, die den aktualisierten Code hostet, möglicherweise beendet, neu konfiguriert, neu bereitgestellt und dann neu gestartet werden. Dies erfordert auch, dass alle anderen Tasks innerhalb derselben Compute-Einheit beendet, neu bereitgestellt und neu gestartet werden.

**Sicherheit**. Tasks in derselben Compute-Einheit können sich einen Sicherheitskontext teilen und auf dieselben Ressourcen zugreifen. Es muss ein hohes Maß an Vertrauen zwischen den Tasks bestehen, und es muss sichergestellt sein, dass die Tasks sich nicht gegenseitig beschädigen oder beeinträchtigen. Zusätzlich wird durch eine Erhöhung der Anzahl von Tasks, die in einer Compute-Einheit ausgeführt werden, die Angriffsfläche der Einheit vergrößert. Jeder Task ist nur so sicher wie derjenige mit dem höchsten Sicherheitsrisiko.

**Fehlertoleranz**. Wenn ein Task in einer Compute-Einheit zu einem Fehler führt oder der Task sich ungewöhnlich verhält, kann dies Auswirkungen auf die anderen Tasks haben, die innerhalb derselben Einheit ausgeführt werden. Wenn z.B. ein Task nicht ordnungsgemäß gestartet wird, kann dies zu einem Fehler für die gesamte Startlogik der Compute-Einheit führen, sodass andere Tasks in derselben Einheit nicht ausgeführt werden können.

**Konflikte**. Vermeiden Sie Konflikte zwischen Tasks, die um Ressourcen in derselben Compute-Einheit konkurrieren. Im Idealfall sollten Tasks, die dieselbe Compute-Einheit verwenden, unterschiedliche Merkmale in Bezug auf die Ressourcennutzung aufweisen. Beispielsweise sollten zwei rechenintensive Tasks nicht in derselben Compute-Einheit platziert werden, und ebenso auch keine zwei Tasks, die viel Speicher verbrauchen. Die Zusammenlegung eines rechenintensiven Tasks mit einem Task, der viel Arbeitsspeicher benötigt, ist hingegen eine geeignete Kombination.

> [!NOTE]
>  Ziehen Sie in Betracht, die Computerressourcen nur für ein System zu konsolidieren, das seit einiger Zeit in Produktion ist, sodass Operatoren und Entwickler das System überwachen und ein _Wärmebild_ erstellen können, das die Nutzung der verschiedenen Ressourcen durch jeden Task zeigt. Anhand dieses Wärmebilds kann festgestellt werden, welche Tasks gute Kandidaten für die gemeinsame Nutzung von Computeressourcen sind.

**Komplexität**. Die Kombination mehrerer Tasks in einer einzigen Compute-Einheit erhöht die Komplexität des Codes in der Einheit und erschwert möglicherweise das Testen, Debuggen sowie die Wartung.

**Stabile logische Architektur**. Entwickeln und implementieren Sie den Code in jedem Task so, dass er selbst dann nicht geändert werden muss, wenn sich die physische Umgebung für die Taskausführung ändert.

**Weitere Strategien**. Die Konsolidierung von Computeressourcen ist nur eine Möglichkeit, die Kosten zu senken, die mit der gleichzeitigen Ausführung mehrerer Tasks verbunden sind. Es ist eine sorgfältige Planung und Überwachung erforderlich, um sicherzustellen, dass die Konsolidierung ein wirksamer Ansatz bleibt. Andere Strategien sind möglicherweise besser geeignet – je nach Art der Arbeit und abhängig davon, wo sich die Benutzer befinden, die diese Aufgaben ausführen. Zum Beispiel kann die funktionale Zerlegung der Workload (wie beschrieben im [Leitfaden zur Computepartitionierung](https://msdn.microsoft.com/library/dn589773.aspx)) eine bessere Option darstellen.

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

Verwenden Sie dieses Muster für Tasks, die nicht kosteneffizient sind, wenn sie in eigenen Compute-Einheiten ausgeführt werden. Wenn ein Task viel Zeit im Leerlauf verbringt, kann es teuer sein, diesen Task in einer dedizierten Einheit auszuführen.

Dieses Muster eignet sich möglicherweise nicht für Tasks, die kritische fehlertolerante Vorgänge ausführen, oder für Tasks, die hochsensible oder private Daten verarbeiten und einen eigenen Sicherheitskontext benötigen. Diese Tasks sollten in einer eigenen, isolierten Umgebung in einer separaten Compute-Einheit ausgeführt werden.

## <a name="example"></a>Beispiel

Beim Erstellen eines Clouddiensts in Azure ist es möglich, die Verarbeitung mehrerer Tasks in einer einzigen Rolle zu konsolidieren. Typischerweise handelt es sich dabei um eine Workerrolle, die Hintergrund- oder asynchrone Verarbeitungstasks ausführt.

> In einigen Fällen ist es möglich, Hintergrund- oder asynchrone Verarbeitungsaufgaben in die Webrolle einzubinden. Diese Technik trägt zur Kostensenkung bei und vereinfacht die Bereitstellung, wenngleich sie sich auf die Skalierbarkeit und Reaktionsfähigkeit der öffentlich zugänglichen Schnittstelle auswirken kann, die von der Webrolle bereitgestellt wird. Der Artikel [Combining Multiple Azure Worker Roles into an Azure Web Role](http://www.31a2ba2a-b718-11dc-8314-0800200c9a66.com/2012/02/combining-multiple-azure-worker-roles.html) (Kombinieren mehrerer Azure-Workerrollen in einer Azure-Webrolle) enthält eine detaillierte Beschreibung der Implementierung von Hintergrund- oder asynchronen Verarbeitungstasks in einer Webrolle.

Die Rolle ist für das Starten und Beenden der Tasks verantwortlich. Wenn der Azure Fabric Controller eine Rolle lädt, löst er das Ereignis `Start` für die Rolle aus. Sie können die Methode `OnStart` der Klasse `WebRole` oder `WorkerRole` überschreiben, um dieses Ereignis zu behandeln – um beispielsweise die Daten und andere Ressourcen zu initialisieren, von denen die Tasks in dieser Methode abhängen.

Wenn die Methode `OnStart ` abgeschlossen wurde, kann die Rolle auf Anforderungen antworten. Weitere Informationen und Anweisungen zur Verwendung der Methoden `OnStart` und `Run` in einer Rolle finden Sie im Abschnitt [Prozesse für den Anwendungsstart](https://msdn.microsoft.com/library/ff803371.aspx#sec16) im Patterns & Practices-Leitfaden zum [Verschieben von Anwendungen in die Cloud](https://msdn.microsoft.com/library/ff728592.aspx).

> Halten Sie den Code in der `OnStart`-Methode so kurz wie möglich. Azure legt keine Zeit bis zum Abschluss dieser Methode fest, aber die Rolle kann erst auf eingehende Netzwerkanforderungen reagieren, wenn diese Methode abgeschlossen wurde.

Wenn die Methode `OnStart` abgeschlossen wurde, führt die Rolle die Methode `Run` aus. Ab diesem Zeitpunkt kann der Fabric Controller mit dem Senden von Anforderungen an die Rolle beginnen.

Platzieren Sie den Code für die eigentliche Erstellung der Tasks in der `Run`-Methode. Beachten Sie, dass die `Run`-Methode die Lebensdauer der Rolleninstanz definiert. Wenn diese Methode abgeschlossen wurde, veranlasst der Fabric Controller das Herunterfahren der Rolle.

Wenn eine Rolle heruntergefahren oder recycelt wird, verhindert der Fabric Controller, dass weitere eingehende Anforderungen vom Load Balancer empfangen werden und löst das Ereignis `Stop` aus. Sie können dieses Ereignis erfassen, indem Sie die `OnStop`-Methode der Rolle außer Kraft setzen und alle erforderlichen Bereinigungen durchführen, bevor die Rolle beendet wird.

> Alle Aktionen, die mit der `OnStop`-Methode ausgeführt werden, müssen innerhalb von fünf Minuten abgeschlossen sein (oder innerhalb von 30 Sekunden, wenn Sie den Azure-Emulator auf einem lokalen Computer verwenden). Andernfalls geht der Azure Fabric Controller davon aus, dass die Rolle angehalten wurde und erzwingt ihre Beendigung.

Die Tasks werden von der Methode `Run` gestartet, die auf den Abschluss der Tasks wartet. Die Tasks implementieren die Geschäftslogik des Clouddiensts und können auf Nachrichten reagieren, die über den Azure-Load Balancer an die Rolle gesendet werden. Die Abbildung zeigt den Lebenszyklus von Tasks und Ressourcen in einer Rolle im Azure-Clouddienst.

![Der Lebenszyklus von Tasks und Ressourcen in einer Rolle im Azure-Clouddienst](./_images/compute-resource-consolidation-lifecycle.png)


Die Datei _WorkerRole.cs_ im Projekt _ComputeResourceConsolidation.Worker_ zeigt ein Beispiel, wie Sie dieses Muster in einem Azure-Clouddienst implementieren können.

> Das Projekt _ComputeResourceConsolidation.Worker_ ist Teil der Lösung _ComputeResourceConsolidation_, die in [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation) für den Download zur Verfügung steht.

Die Methoden `MyWorkerTask1` und `MyWorkerTask2` veranschaulichen, wie verschiedene Tasks innerhalb derselben Workerrolle ausgeführt werden können. Der folgende Code zeigt `MyWorkerTask1`. Es handelt sich um einen einfachen Task, der für 30 Sekunden in den Ruhezustand wechselt und dann eine Ablaufverfolgungsnachricht ausgibt. Dieser Prozess wird wiederholt, bis der Task abgebrochen wird. Der Code in `MyWorkerTask2` ist ähnlich.

```csharp
// A sample worker role task.
private static async Task MyWorkerTask1(CancellationToken ct)
{
  // Fixed interval to wake up and check for work and/or do work.
  var interval = TimeSpan.FromSeconds(30);

  try
  {
    while (!ct.IsCancellationRequested)
    {
      // Wake up and do some background processing if not canceled.
      // TASK PROCESSING CODE HERE
      Trace.TraceInformation("Doing Worker Task 1 Work");

      // Go back to sleep for a period of time unless asked to cancel.
      // Task.Delay will throw an OperationCanceledException when canceled.
      await Task.Delay(interval, ct);
    }
  }
  catch (OperationCanceledException)
  {
    // Expect this exception to be thrown in normal circumstances or check
    // the cancellation token. If the role instances are shutting down, a
    // cancellation request will be signaled.
    Trace.TraceInformation("Stopping service, cancellation requested");

    // Rethrow the exception.
    throw;
  }
}
```

> Der Beispielcode zeigt eine gängige Implementierung eines Hintergrundprozesses. In einer echten Anwendung können Sie dieselbe Struktur verwenden – mit dem Unterschied, dass Sie Ihre eigene Verarbeitungslogik in den Textkörper der Schleife einfügen sollten, die auf die Abbruchanforderung wartet.

Nachdem die Workerrolle die von ihr verwendeten Ressourcen initialisiert hat, startet die `Run`-Methode die beiden Tasks gleichzeitig, wie hier gezeigt.

```csharp
/// <summary>
/// The cancellation token source use to cooperatively cancel running tasks
/// </summary>
private readonly CancellationTokenSource cts = new CancellationTokenSource();

/// <summary>
/// List of running tasks on the role instance
/// </summary>
private readonly List<Task> tasks = new List<Task>();

// RoleEntry Run() is called after OnStart().
// Returning from Run() will cause a role instance to recycle.
public override void Run()
{
  // Start worker tasks and add to the task list
  tasks.Add(MyWorkerTask1(cts.Token));
  tasks.Add(MyWorkerTask2(cts.Token));

  foreach (var worker in this.workerTasks)
  {
      this.tasks.Add(worker);
  }

  Trace.TraceInformation("Worker host tasks started");
  // The assumption is that all tasks should remain running and not return,
  // similar to role entry Run() behavior.
  try
  {
    Task.WaitAll(tasks.ToArray());
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then re-throw the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }

  // If there wasn't a cancellation request, stop all tasks and return from Run()
  // An alternative to canceling and returning when a task exits would be to
  // restart the task.
  if (!cts.IsCancellationRequested)
  {
    Trace.TraceInformation("Task returned without cancellation request");
    Stop(TimeSpan.FromMinutes(5));
  }
}
...
```

In diesem Beispiel wartet die `Run`-Methode auf den Abschluss der Tasks. Wenn ein Task abgebrochen wird, geht die Methode `Run` davon aus, dass die Rolle heruntergefahren wird und wartet auf den Abbruch der verbleibenden Tasks, bevor sie abgeschlossen wird (sie wartet maximal fünf Minuten, bevor sie beendet wird). Wenn ein Task aufgrund einer erwarteten Ausnahme zu einem Fehler führt, bricht die Methode `Run` den Task ab.

> Sie können auch umfassendere Strategien für die Überwachung und Ausnahmebehandlung in der `Run`-Methode implementieren. Beispielsweise können Sie fehlerhafte Tasks neu starten oder Code einfügen, über den die Rolle einzelne Tasks beenden und starten kann.

Die im folgenden Code gezeigte `Stop`-Methode wird aufgerufen, wenn der Fabric Controller die Rolleninstanz herunterfährt (der Aufruf erfolgt über die Methode `OnStop`). Der Code beendet jeden Task ordnungsgemäß, indem er ihn abbricht. Wenn ein Task länger als fünf Minuten dauert, beendet die Abbruchverarbeitung in der Methode `Stop` den Wartevorgang, und die Rolle wird beendet.

```csharp
// Stop running tasks and wait for tasks to complete before returning
// unless the timeout expires.
private void Stop(TimeSpan timeout)
{
  Trace.TraceInformation("Stop called. Canceling tasks.");
  // Cancel running tasks.
  cts.Cancel();

  Trace.TraceInformation("Waiting for canceled tasks to finish and return");

  // Wait for all the tasks to complete before returning. Note that the
  // emulator currently allows 30 seconds and Azure allows five
  // minutes for processing to complete.
  try
  {
    Task.WaitAll(tasks.ToArray(), timeout);
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then rethrow the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }
}
```

## <a name="related-patterns-and-guidance"></a>Zugehörige Muster und Anleitungen

Die folgenden Muster und Anweisungen können für die Implementierung dieses Musters ebenfalls relevant sein:

- [Leitfaden für die automatische Skalierung](https://msdn.microsoft.com/library/dn589774.aspx). Die automatische Skalierung kann verwendet werden, um Dienstinstanzen zu starten und zu beenden, die Computeressourcen hosten – je nach erwartetem Verarbeitungsbedarf.

- [Richtlinien zur Computepartitionierung](https://msdn.microsoft.com/library/dn589773.aspx). Hier wird beschrieben, wie die Dienste und Komponenten in einem Clouddienst so zugewiesen werden, dass die Ausführungskosten minimiert und gleichzeitig die Skalierbarkeit, Leistung, Verfügbarkeit und Sicherheit des Diensts beibehalten werden.

- Zu diesem Muster gehört eine herunterladbare [Beispielanwendung](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).
