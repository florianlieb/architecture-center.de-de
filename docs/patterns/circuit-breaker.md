---
title: Trennschalter
description: Behandeln Sie Fehler, deren Behebung beim Herstellen einer Verbindung mit einem Remotedienst oder einer Remoteressource unterschiedlich lange dauern kann.
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: resiliency
ms.openlocfilehash: ce110d0bbda600575d328895f2feca5aa253479d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="circuit-breaker-pattern"></a>Trennschalter-Muster

Behandeln Sie Fehler, deren Behebung beim Herstellen einer Verbindung mit einem Remotedienst oder einer Remoteressource unterschiedlich lange dauern kann. Dies kann die Stabilität und Resilienz einer Anwendung verbessern.

## <a name="context-and-problem"></a>Kontext und Problem

In einer verteilten Umgebung können bei Aufrufen von Remoteressourcen und -diensten vorübergehende Störungen auftreten, z. B. langsame Netzwerkverbindungen, Timeouts oder überlastete bzw. vorübergehend nicht verfügbare Ressourcen, die zu Fehlern führen können. Diese Störungen korrigieren sich typischerweise nach kurzer Zeit selbst, und eine robuste Cloudanwendung sollte bereit sein, sie mit einer Strategie wie dem [Wiederholungsmuster][retry-pattern] zu behandeln.

Es kann aber auch Situationen geben, in denen Störungen bzw. Fehler auf unvorhergesehene Ereignisse zurückzuführen sind, deren Behebung viel länger dauern kann. Der Schweregrad dieser Störungen kann von einem teilweisen Verlust der Konnektivität bis hin zum vollständigen Ausfall eines Diensts reichen. In solchen Situationen kann es für eine Anwendung zwecklos sein, einen Vorgang, der unwahrscheinlich ist, ständig zu wiederholen. Stattdessen sollte die Anwendung schnell akzeptieren, dass der Vorgang fehlerhaft ist, und diesen Fehler entsprechend behandeln.

Wenn ein Dienst zudem sehr ausgelastet ist, kann ein Ausfall in einem Teil des Systems zu überlappenden Fehlern führen. Ein Vorgang, der einen Dienst aufruft, könnte z. B. so konfiguriert werden, dass er ein Timeout implementiert und mit einer Fehlermeldung antwortet, wenn der Dienst innerhalb dieses Zeitraums nicht antwortet. Diese Strategie könnte jedoch dazu führen, dass viele gleichzeitige Anforderungen an denselben Vorgang bis zum Ablauf der Timeoutperiode blockiert werden. Diese blockierten Anforderungen können kritische Systemressourcen belegen, z. B. Arbeitsspeicher, Threads, Datenbankverbindungen usw. Infolgedessen könnten diese Ressourcen ausgelastet werden. Dies könnte zum Ausfall anderer, möglicherweise nicht zusammengehöriger Teile des Systems führen, die dieselben Ressourcen benötigen. In solchen Situationen wäre es vorzuziehen, wenn der Vorgang sofort einen Fehler aufweist und nur dann versucht wird, den Dienst aufzurufen, wenn er voraussichtlich erfolgreich sein wird. Beachten Sie, dass das Festlegen eines kürzeren Timeouts zur Lösung dieses Problems beitragen kann, aber der Timeout sollte nicht so kurz sein, dass beim Vorgang die meiste Zeit Fehler auftreten, selbst wenn die Anforderung an den Dienst letztendlich erfolgreich sein sollte.

## <a name="solution"></a>Lösung

Das Trennschalter-Muster, das von Michael Nygard in seinem Buch [Release It!](https://pragprog.com/book/mnee/release-it) gemeinverständlich dargestellt wurde, kann verhindern, dass eine Anwendung wiederholt versucht einen Vorgang auszuführen, der voraussichtlich nicht erfolgreich sein wird. Dabei wird die Fortsetzung des Vorgangs gestattet, ohne auf eine Behebung der Störung zu warten oder CPU-Zyklen zu verschwenden, während ermittelt wird, dass es sich um einen länger anhaltenden Fehler handelt. Mithilfe des Trennschalter-Musters kann eine Anwendung auch ermitteln, ob der Fehler behoben wurde. Wenn das Problem scheinbar behoben ist, kann die Anwendung versuchen, den Vorgang aufzurufen.

> Der Zweck des Trennschalter-Musters unterscheidet sich von dem des Wiederholungsmusters. Mithilfe des Wiederholungsmusters kann eine Anwendung einen Vorgang in der Annahme wiederholen, dass er erfolgreich ausgeführt wird. Das Trennschalter-Muster verhindert, dass von einer Anwendung ein Vorgang durchgeführt wird, der voraussichtlich nicht erfolgreich sein wird. Eine Anwendung kann diese beiden Muster kombinieren, indem sie das Wiederholungsmuster verwendet, um einen Vorgang über einen Trennschalter aufzurufen. Die Wiederholungslogik sollte jedoch auf Ausnahmen reagieren, die vom Trennschalter zurückgegeben werden, und Wiederholungsversuche abbrechen, wenn der Trennschalter anzeigt, dass ein Fehler nicht vorübergehend ist.

Ein Trennschalter fungiert als Proxy für Vorgänge, bei denen möglicherweise Fehler auftreten. Der Proxy sollte die Anzahl der kürzlich aufgetretenen Fehler überwachen und anhand dieser Informationen entscheiden, ob der Vorgang fortgesetzt oder sofort eine Ausnahme zurückgegeben werden soll.

Der Proxy kann als Zustandsautomat mit den folgenden Zuständen implementiert werden, die die Funktionalität eines elektrischen Trennschalters simulieren:

- **Geschlossen**: Die Anforderung der Anwendung wird an den Vorgang weitergeleitet. Der Proxy führt eine Zählung der Anzahl der letzten Fehler durch, und wenn der Aufruf des Vorgangs erfolglos ist, inkrementiert der Proxy diesen Zähler. Überschreitet die Anzahl der letzten Fehler innerhalb eines bestimmten Zeitraums einen bestimmten Schwellenwert, wird der Proxy in den Zustand **Geöffnet** versetzt. An diesem Punkt startet der Proxy einen Timeout-Timer, und wenn dieser Timer abläuft, wird der Proxy in den Zustand **Halb geöffnet** versetzt.

    > Der Zweck des Timeout-Timers ist es, dem System Zeit zur Behebung des Problems zu geben, das die Störung verursacht hat, bevor es der Anwendung erlaubt wird, den Vorgang erneut auszuführen.

- **Geöffnet**: Die Anforderung der Anwendung schlägt sofort fehl und eine Ausnahme wird an die Anwendung zurückgegeben.

- **Halb geöffnet**: Eine begrenzte Anzahl von Anforderungen aus der Anwendung dürfen den Vorgang durchlaufen und aufrufen. Wenn diese Anforderungen erfolgreich sind, wird davon ausgegangen, dass die Störung, die zuvor den Fehler verursacht hat, behoben wurde und der Trennschalter in den Zustand **Geschlossen** wechselt (der Fehlerzähler wird zurückgesetzt). Wenn bei einer Anforderung ein Fehler auftritt, geht der Trennschalter davon aus, dass die Störung noch vorhanden ist, sodass er in den Zustand **Geöffnet** zurückkehrt und den Timeout-Timer neu startet, um dem System eine weitere Frist für die Wiederherstellung nach dem Fehler zu geben.

    > Mit dem Zustand **Halb geöffnet** kann verhindert werden, dass ein in der Wiederherstellung befindlicher Dienst plötzlich mit Anforderungen überschwemmt wird. Wenn ein Dienst wiederhergestellt wird, kann er möglicherweise eine begrenzte Anzahl von Anforderungen unterstützen, bis die Wiederherstellung abgeschlossen ist. Aber während der Wiederherstellung kann eine Vielzahl an Aufgaben dazu führen, dass der Dienst erneut den Timeout überschreitet oder ausfällt.

![Trennschalterzustände](./_images/circuit-breaker-diagram.png)

In der Abbildung ist der vom Zustand **Geschlossen** verwendete Fehlerzähler zeitbasiert. Er wird in regelmäßigen Abständen automatisch zurückgesetzt. Dadurch wird verhindert, dass der Trennschalter bei gelegentlichen Ausfällen in den Zustand **Geöffnet** wechselt. Der Fehlerschwellenwert, der den Trennschalter in den Zustand **Geöffnet** versetzt, wird nur erreicht, wenn innerhalb eines bestimmten Intervalls eine bestimmte Anzahl von Fehlern aufgetreten ist. Der Zähler, der vom Zustand **Halb geöffnet** verwendet wird, zeichnet die Anzahl der erfolgreichen Versuche beim Aufrufen des Vorgangs auf. Der Trennschalter kehrt in den Zustand **Geschlossen** zurück, nachdem eine bestimmte Anzahl aufeinanderfolgender Vorgangsaufrufe erfolgreich durchgeführt wurde. Wenn bei einem Aufruf ein Fehler auftritt, wechselt der Trennschalter sofort in den Zustand **Geöffnet** und der Erfolgszähler wird zurückgesetzt, wenn er das nächste Mal in den Zustand **Halb geöffnet** wechselt.

> Die Wiederherstellung des Systems erfolgt extern, möglicherweise durch Wiederherstellen oder Neustarten einer fehlerhaften Komponente oder durch Reparieren einer Netzwerkverbindung.

Das Trennschalter-Muster sorgt für Stabilität, während sich das System nach einem Fehler erholt und die Auswirkungen auf die Leistung minimiert. Es kann dabei helfen, die Antwortzeit des Systems zu wahren, indem es eine Anforderung für einen Vorgang, der voraussichtlich nicht erfolgreich sein wird, schnell ablehnt, anstatt darauf zu warten, dass der Vorgang abgebrochen wird oder niemals zurückkehrt. Wenn der Trennschalter bei jeder Zustandsänderung ein Ereignis auslöst, kann diese Information dazu verwendet werden, den Zustand des durch den Trennschalter geschützten Teils des Systems zu überwachen oder einen Administrator zu alarmieren, wenn ein Trennschalter in den Zustand **Geöffnet** wechselt.

Das Muster ist anpassbar und kann je nach Art des möglichen Fehlers geändert werden. So können Sie z. B. einem Trennschalter einen ansteigenden Timeout-Timer zuweisen. Sie könnten den Trennschalter zunächst für ein paar Sekunden in den Zustand **Geöffnet** versetzen. Wenn der Fehler nicht behoben wurde, können Sie den Timeout auf ein paar Minuten erhöhen usw. In manchen Fällen kann es sinnvoll sein, anstelle des Zustandes **Geöffnet**, der einen Fehler zurückgibt und eine Ausnahme auslöst, einen Standardwert zurückzugeben, der für die Anwendung relevant ist.

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Bei der Entscheidung, wie dieses Muster implementiert werden soll, sind die folgenden Punkte zu beachten:

**Fehlerbehandlung**: Eine Anwendung, die einen Vorgang über einen Trennschalter aufruft, muss darauf vorbereitet sein, die aufgetretenen Ausnahmen zu behandeln, wenn der Vorgang nicht verfügbar ist. Die Art und Weise, wie Ausnahmen behandelt werden, ist anwendungsspezifisch. Eine Anwendung könnte z. B. vorübergehend ihre Funktionalität herabsetzen, einen alternativen Vorgang aufrufen, um zu versuchen, dieselbe Aufgabe auszuführen oder dieselben Daten zu erhalten, oder die Ausnahme dem Benutzer melden und ihn auffordern, es später erneut zu versuchen.

**Arten von Ausnahmen**: Eine Anforderung kann aus vielen Gründen fehlerhaft sein, von denen einige auf eine schwerwiegendere Art von Problemen hinweisen können als andere. Eine Anforderung kann z. B. fehlerhaft sein, weil ein Remotedienst abgestürzt ist und die Wiederherstellung mehrere Minuten dauert, oder weil der Dienst vorübergehend überlastet ist und einen Timeout verursacht hat. Ein Trennschalter könnte in der Lage sein, die Art der aufgetretenen Ausnahmen zu untersuchen und seine Strategie entsprechend der Art dieser Ausnahmen anzupassen. Es könnte z. B. eine größere Anzahl von Timeoutausnahmen erforderlich sein, um den Trennschalter in den Zustand **Geöffnet** zu versetzen, verglichen mit der Anzahl der Störungen, die durch nicht verfügbare Dienste hervorgerufen werden.

**Protokollierung**: Ein Trennschalter sollte alle fehlerhaften Anforderungen (und möglicherweise erfolgreiche Anforderungen) protokollieren, damit ein Administrator die Integrität des Vorgangs überwachen kann.

**Wiederherstellbarkeit**: Sie sollten den Trennschalter so konfigurieren, dass er dem wahrscheinlichen Wiederherstellungsmuster des zu schützenden Vorgangs entspricht. Wenn sich der Trennschalter z. B. über einen längeren Zeitraum im Zustand **Geöffnet** befindet, kann er Ausnahmen auslösen, auch wenn die Fehlerursache behoben ist. Ebenso könnte ein Trennschalter schwanken und die Reaktionszeiten von Anwendungen verkürzen, wenn er zu schnell vom Zustand **Geöffnet** in den Zustand **Halb geöffnet** wechselt.

**Testen fehlerhafter Vorgänge**: Im Zustand **Geöffnet** kann ein Trennschalter für den Remotedienst oder die Ressource regelmäßig den Ping-Befehl ausführen, um zu ermitteln, ob der Dienst oder die Ressource wieder verfügbar ist, anstatt einen Timer zur Ermittlung zu verwenden, wann in den Zustand **Halb geöffnet** gewechselt werden muss. Dieser Ping-Befehl könnte die Form eines Aufrufversuchs für einen Vorgang annehmen, der zuvor fehlerhaft war, oder er könnte einen speziellen Vorgang verwenden, der vom Remotedienst speziell für die Prüfung der Integrität des Dienstes bereitgestellt wird, wie durch das [Muster zur Überwachung des Integritätsendpunkts](health-endpoint-monitoring.md) beschrieben.

**Manuelle Außerkraftsetzung**: In einem System, in sich dem die Wiederherstellungszeit für einen fehlerhaften Vorgang als extrem variabel erweist, ist es vorteilhaft, eine manuelle Option zum Zurücksetzen bereitzustellen, mit der ein Administrator einen Trennschalter schließen (und den Fehlerzähler zurücksetzen) kann. Ebenso könnte ein Administrator einen Trennschalter in den Zustand **Geöffnet** zwingen (und den Timeout-Timer neu starten), wenn der durch den Trennschalter geschützte Vorgang vorübergehend nicht verfügbar ist.

**Parallelität**: Auf denselben Trennschalter kann von einer großen Anzahl gleichzeitiger Instanzen einer Anwendung zugegriffen werden. Die Implementierung sollte keine gleichzeitigen Anforderungen blockieren oder übermäßigen Aufwand für die einzelnen Aufrufe eines Vorgangs verursachen.

**Ressourcendifferenzierung**: Seien Sie bei der Verwendung eines einzelnen Trennschalters für einen Ressourcentyp vorsichtig, wenn mehrere unabhängige Anbieter zugrunde liegen. In einem Datenspeicher, der mehrere Shards enthält, kann z. B. auf einen Shard uneingeschränkt zugegriffen werden, während bei einem anderen Shard ein temporäres Problem auftritt. Wenn die Fehlerreaktionen in diesen Szenarien zusammengeführt werden, könnte eine Anwendung versuchen, auf einige Shards zuzugreifen, selbst wenn ein Fehler sehr wahrscheinlich ist, während der Zugriff auf andere Shards blockiert wird, obwohl er voraussichtlich erfolgreich sein wird.

**Beschleunigter Abschaltvorgang**: Manchmal kann eine Fehlerreaktion ausreichend Informationen enthalten, damit der Trennschalter sofort und für eine minimale Zeitspanne ausgelöst wird. Die Fehlerreaktion einer freigegebenen Ressource, die überlastet ist, könnte z. B. darauf hinweisen, dass eine sofortige Wiederholung nicht empfohlen wird. Stattdessen sollte es die Anwendung in einigen Minuten erneut versuchen.

> [!NOTE]
> Ein Dienst kann HTTP 429 (Zu viele Anforderungen) zurückgeben, wenn er den Client beschränkt, oder HTTP 503 (Dienst nicht verfügbar), wenn der Dienst gerade nicht verfügbar ist. Die Antwort kann zusätzliche Informationen enthalten, z. B. die erwartete Dauer der Verzögerung.

**Wiedergeben fehlerhafter Anforderungen**: Im Zustand **Geöffnet** könnte ein Trennschalter auch die Details jeder Anforderung in einem Journal aufzeichnen und dafür sorgen, dass diese Anforderungen wiedergegeben werden, wenn die Remoteressource oder der Remotedienst verfügbar wird, anstatt einfach nur schnell auszufallen.

**Ungeeignete Timeouts für externe Dienste**: Ein Trennschalter kann Anwendungen möglicherweise nicht vollständig vor fehlerhaften Vorgängen in externen Diensten schützen, die mit einer langen Timeoutperiode konfiguriert sind. Wenn der Timeout zu lang ist, kann es vorkommen, dass ein Thread, der einen Trennschalter ausführt, für längere Zeit blockiert wird, bevor der Trennschalter anzeigt, dass der Vorgang fehlerhaft ist. In dieser Zeit könnten viele andere Anwendungsinstanzen ebenfalls versuchen, den Dienst über den Trennschalter aufzurufen und eine beträchtliche Anzahl von Threads zu binden, bevor sie alle ausfallen.

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

Verwenden Sie dieses Muster in folgenden Fällen:

- Damit verhindert wird, dass eine Anwendung versucht, einen Remotedienst aufzurufen oder auf eine freigegebene Ressource zuzugreifen, wenn sich dieser Vorgang höchstwahrscheinlich als fehlerhaft erweist.

Verwenden Sie dieses Muster in folgenden Fällen nicht:

- Für die Verwaltung des Zugriffs auf lokale private Ressourcen in einer Anwendung, z. B. die In-Memory-Datenstruktur. In dieser Umgebung würde die Verwendung eines Trennschalters zu einem zusätzlichen Aufwand für Ihr System führen.
- Als Ersatz für die Behandlung von Ausnahmen in der Geschäftslogik Ihrer Anwendungen.

## <a name="example"></a>Beispiel

In einer Webanwendung werden mehrere Seiten mit Daten gefüllt, die von einem externen Dienst abgerufen werden. Wenn das System minimales Caching implementiert, führen die meisten Zugriffe auf diese Seiten zu einem Roundtrip zum Dienst. Verbindungen von der Webanwendung zum Dienst können mit einer Timeoutperiode konfiguriert werden (normalerweise 60 Sekunden). Wenn der Dienst in dieser Zeit nicht antwortet, wird die Logik der einzelnen Webseiten davon ausgehen, dass der Dienst nicht verfügbar ist und eine Ausnahme auslösen.

Wenn der Dienst jedoch fehlerhaft und das System sehr ausgelastet ist, könnten Benutzer gezwungen sein, bis zu 60 Sekunden zu warten, bevor eine Ausnahme auftritt. Schließlich könnten Ressourcen wie Arbeitsspeicher, Verbindungen und Threads ausgelastet sein. Dadurch könnten andere Benutzer daran gehindert werden, sich mit dem System zu verbinden, selbst wenn sie nicht auf Seiten zugreifen, die Daten aus dem Dienst abrufen.

Die Skalierung des Systems durch Hinzufügen weiterer Webserver und die Implementierung eines Lastausgleichs könnte sich verzögern, wenn die Ressourcen ausgelastet sind. Es wird das Problem jedoch nicht lösen, da die Benutzeranforderungen immer noch nicht reagieren und allen Webservern eventuell immer noch keine Ressourcen zur Verfügung stehen.

Ein Umschließen der Logik, die sich mit dem Dienst verbindet und die Daten in einem Trennschalter abruft, könnte helfen, dieses Problem zu lösen und den Dienstfehler eleganter zu handhaben. Benutzeranforderungen werden weiterhin fehlerhaft sein, aber sie werden schneller als fehlerhaft erkannt und Ressourcen werden daher nicht blockiert.

Die Klasse `CircuitBreaker` verwaltet Zustandsinformationen zu einem Trennschalter in einem Objekt, das die im folgenden Code gezeigte `ICircuitBreakerStateStore`-Schnittstelle implementiert.

```csharp
interface ICircuitBreakerStateStore
{
  CircuitBreakerStateEnum State { get; }

  Exception LastException { get; }

  DateTime LastStateChangedDateUtc { get; }

  void Trip(Exception ex);

  void Reset();

  void HalfOpen();

  bool IsClosed { get; }
}
```

Die `State`-Eigenschaft gibt den aktuellen Zustand des Trennschalters an, der gemäß der `CircuitBreakerStateEnum`-Enumeration entweder **Geöffnet**, **Halb geöffnet** oder **Geschlossen** ist. Die `IsClosed`-Eigenschaft sollte „true“ sein, wenn der Trennschalter geschlossen ist, aber „false“, wenn er geöffnet oder halb geöffnet ist. Die Methode `Trip` schaltet den Zustand des Trennschalters in den Zustand „Geöffnet“ und zeichnet die Ausnahme auf, die zur Zustandsänderung geführt hat, sowie das Datum und die Uhrzeit, zu der die Ausnahme aufgetreten ist. Die Eigenschaften `LastException` und `LastStateChangedDateUtc` geben diese Informationen zurück. Die Methode `Reset` schließt den Trennschalter und die Methode `HalfOpen` legt den Trennschalter auf „Halb geöffnet“ fest.

Die Klasse `InMemoryCircuitBreakerStateStore` enthält in diesem Beispiel eine Implementierung der Schnittstelle `ICircuitBreakerStateStore`. Die Klasse `CircuitBreaker` erstellt eine Instanz dieser Klasse, um den Zustand des Trennschalters aufzunehmen.

Die `ExecuteAction`-Methode in der `CircuitBreaker`-Klasse dient als Wrapper für einen Vorgang, der als `Action`-Delegat angegeben ist. Wenn der Trennschalter geschlossen ist, ruft `ExecuteAction` den Delegat `Action` auf. Wenn der Vorgang fehlerhaft ist, ruft ein Ausnahmehandler `TrackException` auf, wodurch der Trennschalter in den Zustand „Geöffnet“ versetzt wird. Im folgende Codebeispiel wird dieser Ablauf hervorgehoben.

```csharp
public class CircuitBreaker
{
  private readonly ICircuitBreakerStateStore stateStore =
    CircuitBreakerStateStoreFactory.GetCircuitBreakerStateStore();

  private readonly object halfOpenSyncObject = new object ();
  ...
  public bool IsClosed { get { return stateStore.IsClosed; } }

  public bool IsOpen { get { return !IsClosed; } }

  public void ExecuteAction(Action action)
  {
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open.
      ... (see code sample below for details)
    }

    // The circuit breaker is Closed, execute the action.
    try
    {
      action();
    }
    catch (Exception ex)
    {
      // If an exception still occurs here, simply
      // retrip the breaker immediately.
      this.TrackException(ex);

      // Throw the exception so that the caller can tell
      // the type of exception that was thrown.
      throw;
    }
  }

  private void TrackException(Exception ex)
  {
    // For simplicity in this example, open the circuit breaker on the first exception.
    // In reality this would be more complex. A certain type of exception, such as one
    // that indicates a service is offline, might trip the circuit breaker immediately.
    // Alternatively it might count exceptions locally or across multiple instances and
    // use this value over time, or the exception/success ratio based on the exception
    // types, to open the circuit breaker.
    this.stateStore.Trip(ex);
  }
}
```

Das folgende Beispiel zeigt den Code (im vorherigen Beispiel ausgelassen), der ausgeführt wird, wenn der Trennschalter nicht geschlossen ist. Zuerst wird geprüft, ob der Trennschalter für einen längeren Zeitraum als die durch das lokale Feld `OpenToHalfOpenWaitTime` in der Klasse `CircuitBreaker` vorgegebene Zeit geöffnet war. In diesem Fall legt die `ExecuteAction`-Methode den Trennschalter auf „Halb geöffnet“ fest und versucht dann, den vom Delegaten `Action` angegebenen Vorgang auszuführen.

Wenn der Vorgang erfolgreich ist, wird der Trennschalter in den Zustand „Geschlossen“ zurückgesetzt. Wenn der Vorgang fehlerhaft ist, wird er in den Zustand „Geöffnet“ zurückversetzt und der Zeitpunkt, zu dem die Ausnahme aufgetreten ist, wird aktualisiert, sodass der Trennschalter eine weitere Zeitspanne wartet, bevor er erneut versucht, den Vorgang auszuführen.

Wenn der Trennschalter nur kurzzeitig geöffnet war (kürzer als durch den Wert `OpenToHalfOpenWaitTime` angegeben), löst die Methode `ExecuteAction` einfach eine `CircuitBreakerOpenException`-Ausnahme aus und gibt den Fehler zurück, der den Übergang des Trennschalters in den Zustand „Geöffnet“ verursacht hat.

Darüber hinaus verwendet er eine Sperre, um zu verhindern, dass der Trennschalter versucht, gleichzeitige Aufrufe des Vorgangs durchzuführen, während er halb geöffnet ist. Ein gleichzeitiger Versuch zum Aufrufen des Vorgangs wird so behandelt, als ob der Trennschalter geöffnet wäre, und er wird mit einer Ausnahme fehlerhaft beendet, wie später beschrieben.

```csharp
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open. Check if the Open timeout has expired.
      // If it has, set the state to HalfOpen. Another approach might be to
      // check for the HalfOpen state that had be set by some other operation.
      if (stateStore.LastStateChangedDateUtc + OpenToHalfOpenWaitTime < DateTime.UtcNow)
      {
        // The Open timeout has expired. Allow one operation to execute. Note that, in
        // this example, the circuit breaker is set to HalfOpen after being
        // in the Open state for some period of time. An alternative would be to set
        // this using some other approach such as a timer, test method, manually, and
        // so on, and check the state here to determine how to handle execution
        // of the action.
        // Limit the number of threads to be executed when the breaker is HalfOpen.
        // An alternative would be to use a more complex approach to determine which
        // threads or how many are allowed to execute, or to execute a simple test
        // method instead.
        bool lockTaken = false;
        try
        {
          Monitor.TryEnter(halfOpenSyncObject, ref lockTaken)
          if (lockTaken)
          {
            // Set the circuit breaker state to HalfOpen.
            stateStore.HalfOpen();

            // Attempt the operation.
            action();

            // If this action succeeds, reset the state and allow other operations.
            // In reality, instead of immediately returning to the Closed state, a counter
            // here would record the number of successful operations and return the
            // circuit breaker to the Closed state only after a specified number succeed.
            this.stateStore.Reset();
            return;
          }
          catch (Exception ex)
          {
            // If there's still an exception, trip the breaker again immediately.
            this.stateStore.Trip(ex);

            // Throw the exception so that the caller knows which exception occurred.
            throw;
          }
          finally
          {
            if (lockTaken)
            {
              Monitor.Exit(halfOpenSyncObject);
            }
          }
        }
      }
      // The Open timeout hasn't yet expired. Throw a CircuitBreakerOpen exception to
      // inform the caller that the call was not actually attempted,
      // and return the most recent exception received.
      throw new CircuitBreakerOpenException(stateStore.LastException);
    }
    ...
```

Wenn Sie ein `CircuitBreaker`-Objekt zum Schutz eines Vorgangs verwenden möchten, erstellt eine Anwendung eine Instanz der Klasse `CircuitBreaker` und ruft die Methode `ExecuteAction` auf, wobei der auszuführende Vorgang als Parameter angegeben wird. Die Anwendung sollte darauf vorbereitet sein, die Ausnahme `CircuitBreakerOpenException` abzufangen, wenn der Vorgang fehlerhaft beendet wird, da der Trennschalter geöffnet ist. Der folgende Code zeigt ein Beispiel:

```csharp
var breaker = new CircuitBreaker();

try
{
  breaker.ExecuteAction(() =>
  {
    // Operation protected by the circuit breaker.
    ...
  });
}
catch (CircuitBreakerOpenException ex)
{
  // Perform some different action when the breaker is open.
  // Last exception details are in the inner exception.
  ...
}
catch (Exception ex)
{
  ...
}
```

## <a name="related-patterns-and-guidance"></a>Zugehörige Muster und Anleitungen

Die folgenden Muster können für die Implementierung dieses Musters relevant sein:

- [Wiederholungsmuster][retry-pattern]: Beschreibt, wie eine Anwendung beim Herstellen einer Verbindung mit einem Dienst oder einer Netzwerkressource antizipierte, temporäre Fehler behandeln kann, indem ein zuvor nicht erfolgreich durchgeführter Vorgang transparent wiederholt wird.

- [Muster für Überwachung des Integritätsendpunkts](health-endpoint-monitoring.md): Ein Trennschalter könnte in der Lage sein, den Zustand eines Diensts zu testen, indem er eine Anforderung an einen Endpunkt sendet, der durch den Dienst exponiert wird. Der Dienst sollte Informationen über seinen Zustand zurückgeben.


[retry-pattern]: ./retry.md
