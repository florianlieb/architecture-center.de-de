---
title: Auswahl einer übergeordneten Instanz
description: Koordinieren Sie die Aktionen, die von einer Sammlung zusammenarbeitender Taskinstanzen ausgeführt werden, in einer verteilten Anwendung, indem eine Instanz als übergeordnete Instanz ausgewählt wird, die die Verantwortung für die Verwaltung der anderen Instanzen übernimmt.
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- resiliency
ms.openlocfilehash: 3e7d47f70f660f2507f0619e1c41bf9a32a25be4
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="leader-election-pattern"></a>Muster für die Auswahl einer übergeordneten Instanz

[!INCLUDE [header](../_includes/header.md)]

Koordinieren Sie die von einer Sammlung zusammenarbeitender Instanzen ausgeführten Aktionen in einer verteilten Anwendung, indem Sie eine Instanz als übergeordnete Instanz auswählen, die die Verantwortung für die Verwaltung der anderen übernimmt. Dies kann dazu beitragen, sicherzustellen, dass Instanzen nicht in Konflikt miteinander, um freigegebene Ressourcen oder versehentlich auch mit der Arbeit anderer Instanzen stehen.

## <a name="context-and-problem"></a>Kontext und Problem

Eine typische Cloudanwendung weist viele Aufgaben auf, die koordiniert ausgeführt werden. Bei diesen Aufgaben kann es sich um Instanzen handeln, die alle denselben Code ausführen und auf dieselben Ressourcen zugreifen möchten, oder sie können parallel gemeinsam an Bestandteilen einer komplexen Berechnung arbeiten.

Zu einem großen Teil werden die Taskinstanzen möglicherweise separat ausgeführt, es kann jedoch auch notwendig sein, die Aktionen der einzelnen Instanzen zu koordinieren, um sicherzustellen, dass sie nicht in Konflikt miteinander, um freigegebene Ressourcen oder versehentlich auch mit der Arbeit anderer Instanzen stehen.

Beispiel: 

- In einem cloudbasierten System, in dem eine horizontale Skalierung implementiert ist, können mehrere Instanzen desselben Tasks gleichzeitig ausgeführt werden, wobei jede Instanz einem anderen Benutzer zugeordnet ist. Wenn diese Instanzen in eine freigegebene Ressource schreiben, müssen ihre Aktionen koordiniert werden, um zu verhindern, dass jede Instanz die von der anderen vorgenommenen Änderungen überschreibt.
- Wenn die Aufgaben einzelne Elemente einer komplexen Berechnung parallel ausführen, müssen die Ergebnisse aggregiert werden, wenn alle abgeschlossen sind.

Alle Taskinstanzen sind gleichrangig, daher gibt es keine natürliche übergeordnete Instanz, die als Koordinator oder Aggregator fungieren kann.

## <a name="solution"></a>Lösung

Eine einzelne Aufgabeninstanz sollte zur übergeordneten Instanz gewählt werden. Diese Instanz koordiniert die Aktionen der anderen untergeordneten Taskinstanzen. Wenn alle Taskinstanzen denselben Code ausführen, kann jede als übergeordnete Instanz fungieren. Aus diesem Grund muss der Wahlprozess sorgfältig verwaltet werden, um zu verhindern, dass mehrere Instanzen die übergeordnete Rolle gleichzeitig annehmen.

Das System muss einen stabilen Mechanismus für die Auswahl der übergeordneten Instanz bereitstellen. Diese Methode muss mit Ereignissen wie Netzwerkausfällen oder Prozessfehlern umgehen können. In vielen Lösungen überwachen die untergeordneten Aufgabeninstanzen die übergeordnete durch eine Taktmethode oder durch Polls. Wenn die festgelegte übergeordnete Instanz unerwartet beendet wird oder ein Netzwerkausfall dafür sorgt, dass sie nicht verfügbar ist, müssen die anderen Instanzen eine neue übergeordnete Instanz wählen.

Für die Wahl einer übergeordneten Instanz zwischen mehreren in einer verteilten Umgebung gibt es mehrere Strategien, einschließlich:
- Auswählen der Aufgabeninstanz mit der niedrigsten Instanz- oder Prozess-ID.
- Wettbewerb um den Abruf eines verteilten gegenseitigen Ausschlusses. Die erste Aufgabeninstanz, die den gegenseitigen Ausschluss abruft, ist die übergeordnete Instanz. Das System muss jedoch sicherstellen, dass beim Beenden oder Trennen der übergeordneten Instanz vom restlichen System der gegenseitige Ausschluss aufgehoben wird, sodass eine andere Aufgabeninstanz die übergeordnete Instanz werden kann.
- Implementieren eines der verbreiteten Algorithmen für die Wahl der übergeordneten Instanz, z.B. [Bullyalgorithmus](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html) oder [Ringalgorithmus](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html). Bei diesen Algorithmen wird davon ausgegangen, dass jeder Kandidat im Wahlverfahren eine eindeutige ID aufweist und zuverlässig mit den anderen Kandidaten kommunizieren kann.

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:
- Die Wahl einer übergeordneten Instanz sollte gegenüber vorübergehenden und permanenten Fehlern robust sein.
- Es muss möglich sein, zu erkennen, wenn die übergeordnete Instanz einen Fehler aufweist oder anderweitig nicht mehr verfügbar ist, z.B. aufgrund eines Kommunikationsfehlers. Wie schnell eine Erkennung benötigt wird, ist systemabhängig. Einige Systeme funktionieren möglicherweise für kurze Zeit ohne übergeordnete Instanz. In dieser Zeit kann ein vorübergehender Fehler behoben werden. In anderen Fällen kann es erforderlich sein, den Ausfall der übergeordneten Instanz sofort zu erkennen und eine neue Wahl auszulösen.
- In einem System, in dem eine automatische horizontale Skalierung implementiert ist, kann die übergeordnete Instanz möglicherweise beendet werden, wenn das System zurückskaliert und einige der Verarbeitungsressourcen herunterfährt.
- Mit der Verwendung eines freigegebenen, nicht verteilten gegenseitigen Ausschlusses wird eine Abhängigkeit von dem externen Dienst, der den gegenseitigen Ausschluss bereitstellt, eingeführt. Der Dienst stellt einen Single Point of Failure dar. Wenn er aus irgendeinem Grund nicht verfügbar ist, kann das System keine übergeordnete Instanz wählen.
- Die Verwendung eines einzelnen dedizierten Prozesses als übergeordnete Instanz ist ein einfaches Verfahren. Wenn bei dem Verfahren jedoch ein Fehler auftritt, kann es während des Neustarts zu einer erheblichen Verzögerung kommen. Die resultierende Latenz kann die Leistung und die Antwortzeiten anderer Prozesse beeinträchtigen, wenn sie zum Koordinieren eines Vorgangs auf die übergeordnete Instanz warten.
- Die größte Flexibilität für die Feinabstimmung und Optimierung des Codes ist bei der manuellen Implementierung eines der Wahlalgorithmen für die übergeordnete Instanz gegeben.

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

Verwenden Sie dieses Muster, wenn die Aufgaben in einer verteilten Anwendung wie einer in der Cloud gehosteten Lösung eine sorgfältige Koordination erfordern und keine natürliche übergeordnete Instanz besteht.

>  Vermeiden Sie es, die übergeordnete Instanz zu einem Engpass im System zu machen. Die übergeordnete Instanz dient zum Koordinieren der Arbeit der untergeordneten Aufgaben und muss nicht unbedingt selbst an dieser Arbeit teilnehmen – obwohl dies möglich sein sollte, wenn die Aufgabe nicht zur übergeordneten Instanz gewählt wurde.

Dieses Muster ist in folgenden Fällen möglicherweise nicht geeignet:
- Es gibt eine natürliche übergeordnete Instanz oder einen dedizierten Prozess, der immer die Rolle der übergeordneten Instanz übernehmen kann. Beispielsweise kann es möglich sein, einen Singleton-Prozess zu implementieren, der die Aufgabeninstanzen koordiniert. Wenn bei diesem Prozess ein Fehler auftritt oder seine Integrität beeinträchtigt wird, kann das System ihn herunterfahren und neu starten.
- Die Koordination zwischen Aufgaben kann mithilfe einer weniger aufwendigen Methode erreicht werden. Wenn beispielsweise mehrere Aufgabeninstanzen einfach einen koordinierten Zugriff auf eine freigegebene Ressource benötigen, ist eine optimistische oder pessimistische Sperrung zum Steuern des Zugriffs eine bessere Lösung.
- Eine Lösung eines Drittanbieters ist besser geeignet. Beispielsweise verwendet der Microsoft Azure HDInsight-Dienst (basierend auf Apache Hadoop) die von Apache ZooKeeper bereitgestellten Dienste zum Koordinieren der Zuordnung und Verringerung von Aufgaben, die Daten erfassen und zusammenfassen.

## <a name="example"></a>Beispiel

Das Projekt DistributedMutex in der Projektmappe LeaderElection (ein Beispiel für dieses Muster ist bei [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election) zu finden) zeigt, wie mit einer Lease für ein Azure Storage-Blob ein Mechanismus zum Implementieren eines freigegebenen, verteilten gegenseitigen Ausschlusses bereitgestellt werden kann. Mit diesem gegenseitigen Ausschluss kann eine übergeordnete Instanz aus einer Gruppe von Rolleninstanzen in einem Azure-Clouddienst gewählt werden. Die erste Rolleninstanz, die die Lease erhält, wird als übergeordnete Instanz bestimmt und bleibt diese, bis sie die Lease freigibt oder diese nicht mehr erneuern kann. Andere Rolleninstanzen können die Bloblease für den Fall, dass die übergeordnete Instanz nicht mehr verfügbar ist, weiterhin überwachen.

>  Eine Bloblease ist eine exklusive Schreibsperre auf ein Blob. Ein einzelnes Blob kann jeweils nur einer Lease unterliegen. Eine Rolleninstanz kann eine Lease für ein angegebenes Blob anfordern und erhält diese, wenn keine andere Rolleninstanz über eine Lease für dasselbe Blob verfügt. Andernfalls löst die Anforderung eine Ausnahme aus.
> 
> Um eine fehlerhafte Rolleninstanz zu vermeiden, die die Lease unbegrenzt beibehält, geben Sie eine Gültigkeitsdauer für die Lease an. Wenn diese abläuft, ist die Lease wieder verfügbar. Während eine Rolleninstanz über die Lease verfügt, kann sie jedoch eine Erneuerung der Lease anfordern und erhält dann die Lease für einen weiteren Zeitraum. Die Rolleninstanz kann diesen Vorgang fortlaufend wiederholen, wenn sie die Lease beibehalten möchte.
> Weitere Informationen zum Leasen eines Blobs finden Sie unter [Leasen eines Blobs (REST-API)](https://msdn.microsoft.com/library/azure/ee691972.aspx).

Die `BlobDistributedMutex`-Klasse im folgenden C#-Beispiel enthält die `RunTaskWhenMutexAquired`-Methode, die einer Rolleninstanz ermöglicht, eine Lease für ein angegebenes Blob abzurufen. Die Details des Blobs (Name, Container und Speicherkonto) werden in den Konstruktor in einem `BlobSettings`-Objekt übergeben, wenn das `BlobDistributedMutex`-Objekt erstellt wird (eine einfache Struktur, die im Beispielcode enthalten ist). Der Konstruktor akzeptiert auch eine `Task`, die auf den Code verweist, den die Rolleninstanz ausführen sollte, wenn sie erfolgreich die Lease für das Blob abruft und zur übergeordneten Instanz gewählt wird. Beachten Sie, dass der Code für die Verarbeitung der Details auf niedriger Ebene zum Abrufen der Lease in der separaten Hilfsklasse `BlobLeaseManager` implementiert wird.

```csharp
public class BlobDistributedMutex
{
  ...
  private readonly BlobSettings blobSettings;
  private readonly Func<CancellationToken, Task> taskToRunWhenLeaseAcquired;
  ...

  public BlobDistributedMutex(BlobSettings blobSettings,
           Func<CancellationToken, Task> taskToRunWhenLeaseAquired)
  {
    this.blobSettings = blobSettings;
    this.taskToRunWhenLeaseAquired = taskToRunWhenLeaseAquired;
  }

  public async Task RunTaskWhenMutexAcquired(CancellationToken token)
  {
    var leaseManager = new BlobLeaseManager(blobSettings);
    await this.RunTaskWhenBlobLeaseAcquired(leaseManager, token);
  }
  ...
```

Die `RunTaskWhenMutexAquired`-Methode im obigen Codebeispiel ruft die im folgenden Codebeispiel gezeigte `RunTaskWhenBlobLeaseAcquired`-Methode auf, um die Lease abzurufen. Die `RunTaskWhenBlobLeaseAcquired`-Methode wird asynchron ausgeführt. Nach dem Abruf der Lease wurde die Rolleninstanz zur übergeordneten Instanz gewählt. Der Zweck des Delegaten `taskToRunWhenLeaseAcquired` besteht darin, die Aufgaben für die Koordinierung der anderen Rolleninstanzen auszuführen. Wenn die Lease nicht abgerufen wird, wurde eine andere Rolleninstanz zur übergeordneten Instanz gewählt, und die aktuelle Rolleninstanz bleibt untergeordnet. Beachten Sie, dass die `TryAcquireLeaseOrWait`-Methode eine Hilfsmethode ist, die zum Abrufen der Lease das `BlobLeaseManager`-Objekt verwendet.

```csharp
  private async Task RunTaskWhenBlobLeaseAcquired(
    BlobLeaseManager leaseManager, CancellationToken token)
  {
    while (!token.IsCancellationRequested)
    {
      // Try to acquire the blob lease.
      // Otherwise wait for a short time before trying again.
      string leaseId = await this.TryAquireLeaseOrWait(leaseManager, token);

      if (!string.IsNullOrEmpty(leaseId))
      {
        // Create a new linked cancellation token source so that if either the
        // original token is canceled or the lease can't be renewed, the
        // leader task can be canceled.
        using (var leaseCts =
          CancellationTokenSource.CreateLinkedTokenSource(new[] { token }))
        {
          // Run the leader task.
          var leaderTask = this.taskToRunWhenLeaseAquired.Invoke(leaseCts.Token);
          ...
        }
      }
    }
    ...
  }
```

Die von der übergeordneten Instanz gestartete Aufgabe wird ebenfalls asynchron ausgeführt. Während der Ausführung dieser Aufgabe versucht die im folgenden Codebeispiel gezeigte `RunTaskWhenBlobLeaseAquired`-Methode in regelmäßigen Abständen, die Lease zu verlängern. Dadurch wird sichergestellt, dass die Rolleninstanz die übergeordnete Instanz bleibt. In der Beispiellösung ist die Verzögerung zwischen Erneuerungsanforderungen kürzer als die als Dauer der Lease angegebene Zeit. Dies verhindert, dass eine andere Rolleninstanz zur übergeordneten Instanz gewählt wird. Wenn bei der Erneuerung aus irgendeinem Grund ein Fehler auftritt, wird die Aufgabe abgebrochen.

Wenn die Lease nicht erneuert werden kann oder die Aufgabe abgebrochen wird (möglicherweise aufgrund des Herunterfahrens der Rolleninstanz), wird die Lease freigegeben. An diesem Punkt kann diese oder eine andere Rolleninstanz zur übergeordneten Instanz gewählt werden. Der Codeausschnitt unten zeigt diesen Teil des Vorgangs.

```csharp
  private async Task RunTaskWhenBlobLeaseAcquired(
    BlobLeaseManager leaseManager, CancellationToken token)
  {
    while (...)
    {
      ...
      if (...)
      {
        ...
        using (var leaseCts = ...)
        {
          ...
          // Keep renewing the lease in regular intervals.
          // If the lease can't be renewed, then the task completes.
          var renewLeaseTask =
            this.KeepRenewingLease(leaseManager, leaseId, leaseCts.Token);

          // When any task completes (either the leader task itself or when it
          // couldn't renew the lease) then cancel the other task.
          await CancelAllWhenAnyCompletes(leaderTask, renewLeaseTask, leaseCts);
        }
      }
    }
  }
  ...
}
```

Die `KeepRenewingLease`-Methode ist eine weitere Hilfsmethode, die das `BlobLeaseManager`-Objekt zum Erneuern der Lease verwendet. Die `CancelAllWhenAnyCompletes`-Methode bricht die als die ersten beiden Parameter angegebenen Aufgaben ab. Das folgende Diagramm veranschaulicht, wie mit der `BlobDistributedMutex`-Klasse eine übergeordnete Instanz gewählt und eine Aufgabe ausgeführt wird, die Vorgänge koordiniert.

![Abbildung 1 zeigt die Funktionen der BlobDistributedMutex-Klasse](./_images/leader-election-diagram.png)


Im folgenden Codebeispiel wird veranschaulicht, wie die `BlobDistributedMutex`-Klasse in einer Workerrolle verwendet wird. Dieser Code ruft eine Lease für das Blob `MyLeaderCoordinatorTask` im Container der Lease im Entwicklungsspeicher ab und gibt an, dass der in der `MyLeaderCoordinatorTask`-Methode definierte Code immer dann ausgeführt werden soll, wenn die Rolleninstanz als übergeordnete Instanz gewählt wurde.

```csharp
var settings = new BlobSettings(CloudStorageAccount.DevelopmentStorageAccount,
  "leases", "MyLeaderCoordinatorTask");
var cts = new CancellationTokenSource();
var mutex = new BlobDistributedMutex(settings, MyLeaderCoordinatorTask);
mutex.RunTaskWhenMutexAcquired(this.cts.Token);
...

// Method that runs if the role instance is elected the leader
private static async Task MyLeaderCoordinatorTask(CancellationToken token)
{
  ...
}
```

Beachten Sie die folgenden Punkte bezüglich der Beispiellösung:
- Das Blob ist potenziell ein Single Point of Failure. Wenn der Blob-Dienst nicht mehr verfügbar ist oder nicht auf ihn zugegriffen werden kann, kann die übergeordnete Instanz die Lease nicht erneuern, und keine andere Rolleninstanz ist in der Lage, die Lease abzurufen. In diesem Fall kann keine Rolleninstanz als übergeordnete Instanz fungieren. Der Blob-Dienst ist jedoch auf Stabilität ausgelegt, daher wird ein vollständiger Ausfall des Blob-Diensts als sehr unwahrscheinlich betrachtet.
- Wenn der von der übergeordneten Instanz ausgeführte Task angehalten wird, kann die übergeordnete Instanz die Lease weiterhin erneuern. Dadurch werden alle anderen Rolleninstanzen daran gehindert, die Lease abzurufen und die Rolle der übergeordneten Instanz zu übernehmen, um Aufgaben zu koordinieren. In der Praxis sollte die Integrität der übergeordneten Instanz häufig überprüft werden.
- Der Wahlvorgang ist nicht deterministisch. Es sind keine Prognosen darüber möglich, welche Rolleninstanz die Bloblease abruft und zur übergeordneten Instanz wird.
- Das als Ziel für die Bloblease verwendete Blob sollte nicht für andere Zwecke verwendet werden. Wenn eine Rolleninstanz versucht, Daten in diesem Blob zu speichern, kann auf diese Daten nicht zugegriffen werden, es sei denn, die Rolleninstanz ist die übergeordnete Instanz und verfügt über die Bloblease.

## <a name="related-patterns-and-guidance"></a>Zugehörige Muster und Anleitungen

Die folgenden Richtlinien sind unter Umständen beim Implementieren dieses Musters ebenfalls relevant:
- Dieses Muster verfügt über eine herunterladbare [Beispielanwendung](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election).
- [Richtlinien für die automatische Skalierung:](https://msdn.microsoft.com/library/dn589774.aspx) Instanzen der Aufgabenhosts können gestartet und beendet werden, wenn sich die Auslastung Anwendung ändert. Automatische Skalierung kann dabei helfen, den Durchsatz und die Leistung während Zeiten maximaler Verarbeitung beizubehalten.
- [Richtlinien zur Computepartitionierung:](https://msdn.microsoft.com/library/dn589773.aspx) In diesen Richtlinien wird beschrieben, wie Aufgaben in einem Clouddienst so Hosts zugewiesen werden, dass die Betriebskosten minimiert und gleichzeitig die Skalierbarkeit, Leistung, Verfügbarkeit und Sicherheit des Diensts beibehalten werden.
- Das [aufgabenbasierte asynchrone Muster](https://msdn.microsoft.com/library/hh873175.aspx)
- Ein Beispiel zur Veranschaulichung des [Bullyalgorithmus](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html)
- Ein Beispiel zur Veranschaulichung des [Ringalgorithmus](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html)
- Der Artikel [Apache ZooKeeper in Microsoft Azure](https://msopentech.com/opentech-projects/apache-zookeeper-on-windows-azure-2/) auf der Microsoft Open Technologies-Website
- [Apache Curator](http://curator.apache.org/), eine Clientbibliothek für Apache ZooKeeper
- Der Artikel [Leasen eines Blobs (REST-API)](https://msdn.microsoft.com/library/azure/ee691972.aspx) bei MSDN
