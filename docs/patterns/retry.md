---
title: Wiederholen
description: Ermöglichen Sie einer Anwendung beim Herstellen einer Verbindung mit einem Dienst oder einer Netzwerkressource die Behandlung antizipierter, temporärer Fehler, indem ein zuvor nicht erfolgreich durchgeführter Vorgang transparent wiederholt wird.
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- resiliency
ms.openlocfilehash: 73fdcbcc2bd75593a4c8e33dc2259c90593e14db
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/23/2018
---
# <a name="retry-pattern"></a>Wiederholungsmuster

[!INCLUDE [header](../_includes/header.md)]

Ermöglichen Sie einer Anwendung beim Herstellen einer Verbindung mit einem Dienst oder einer Netzwerkressource die Behandlung vorübergehender Fehler, indem ein fehlgeschlagener Vorgang transparent wiederholt wird. Dies kann die Stabilität der Anwendung verbessern.

## <a name="context-and-problem"></a>Kontext und Problem

Eine Anwendung, die mit in der Cloud ausgeführten Elementen kommuniziert, muss gegenüber vorübergehenden Fehlern empfindlich sein, die in dieser Umgebung auftreten können. Fehler umfassen den vorübergehenden Verlust der Netzwerkkonnektivität mit Komponenten und Diensten, die vorübergehende Nichtverfügbarkeit eines Diensts oder Timeouts, die auftreten, wenn ein Dienst ausgelastet ist.

Diese Fehler werden in der Regel automatisch behoben, und wenn die Aktion, die einen Fehler ausgelöst hat, nach einer angemessenen Verzögerung wiederholt wird, wird sie wahrscheinlich erfolgreich ausgeführt. Beispielsweise kann ein Datenbankdienst, der eine große Anzahl gleichzeitiger Anforderungen verarbeitet, eine Drosselungsstrategie implementieren, durch die weitere Anforderungen vorübergehend abgelehnt werden, bis die Arbeitsauslastung abgenommen hat. Eine Anwendung, die auf die Datenbank zuzugreifen versucht, kann möglicherweise keine Verbindung herstellen, doch ist bei einem wiederholten Versuch nach einer Verzögerung der Zugriff möglich.

## <a name="solution"></a>Lösung

In der Cloud sind vorübergehende Fehler nicht ungewöhnlich, und eine Anwendung sollte so entworfen sein, dass diese elegant und transparent behandelt werden. Dadurch verringern sich die Auswirkungen, die Fehler auf die Geschäftsaufgaben haben können, die von der Anwendung ausgeführt werden.

Wenn eine Anwendung beim Senden einer Anforderung an einen Remotedienst einen Fehler erkennt, kann dieser mithilfe der folgenden Strategien behandelt werden:

- **Abbrechen**. Wenn der Fehler anzeigt, dass er nicht vorübergehend ist oder durch eine Wiederholung wahrscheinlich nicht erfolgreich behoben werden kann, sollte die Anwendung den Vorgang abbrechen und eine Ausnahme melden. Beispielsweise kann ein Authentifizierungsfehler, der durch Bereitstellung ungültiger Anmeldeinformationen entsteht, wahrscheinlich nicht erfolgreich behoben werden, unabhängig davon, wie oft dies versucht wird.

- **Wiederholen**. Ist der gemeldete Fehler ungewöhnlich oder selten, wurde er möglicherweise durch ungewöhnliche Umstände verursacht, wie z. B. ein Netzwerkpaket, das während der Übertragung beschädigt wurde. In diesem Fall kann die Anwendung die fehlgeschlagene Anforderung sofort wiederholen, da der gleiche Fehler wahrscheinlich nicht erneut auftreten wird und die Anforderung dann erfolgreich ausgeführt werden kann.

- **Wiederholen nach Verzögerung.** Wenn der Fehler durch einen der eher häufig auftretenden Konnektivitäts- oder Auslastungsfehler verursacht wird, benötigt das Netzwerk oder der Dienst möglicherweise eine kurze Zeit, um die Verbindungsprobleme zu beheben oder die Arbeitsrückstände aufzuarbeiten. Die Anwendung sollte eine entsprechende Zeit lang warten, bevor die Anforderung wiederholt wird.

Bei häufiger auftretenden vorübergehenden Fehlern sollte der Zeitraum zwischen den Wiederholungen so gewählt werden, dass Anforderungen von mehreren Instanzen der Anwendung so gleichmäßig wie möglich verteilt werden. Dadurch sinkt die Wahrscheinlichkeit, dass ein ausgelasteter Dienst weiterhin überlastet wird. Wenn viele Instanzen einer Anwendung einen Dienst ständig mit Wiederholungsanforderungen überlasten, benötigt der Dienst mehr Zeit zur Wiederherstellung.

Schlägt die Anforderung immer noch fehl, kann die Anwendung warten und einen erneuten Versuch unternehmen. Bei Bedarf kann dieser Vorgang mit zunehmender Verzögerung zwischen den Wiederholungsversuchen ausgeführt werden, bis eine maximale Anzahl von Anforderungsversuchen unternommen wurde. Die Verzögerung kann inkrementell oder exponentiell erhöht werden, je nach Art des Fehlers und der Wahrscheinlichkeit, dass er während dieser Zeit behoben wird.

Das folgende Diagramm veranschaulicht das Aufrufen eines Vorgangs in einem gehosteten Dienst mithilfe dieses Musters. Wenn die Anforderung nach einer vordefinierten Anzahl von Versuchen nicht erfolgreich ausgeführt werden kann, sollte die Anwendung den Fehler als Ausnahme behandeln und entsprechend vorgehen.

![Abbildung 1: Aufrufen eines Vorgangs in einem gehosteten Dienst mithilfe des Wiederholungsmusters](./_images/retry-pattern.png)

Die Anwendung sollte alle Zugriffsversuche auf einen Remotedienst in Code einschließen, der eine Wiederholungsrichtlinie implementiert, die einer der oben aufgeführten Strategien entspricht. Anforderungen, die an verschiedene Dienste gesendet werden, können unterschiedliche Richtlinien unterliegen. Einige Anbieter stellen Bibliotheken bereit, die Wiederholungsrichtlinien implementieren, in denen die Anwendung die maximale Anzahl von Wiederholungen, die Zeit zwischen Wiederholungsversuchen und andere Parameter angeben kann.

Eine Anwendung sollte die Details der Fehler und fehlgeschlagenen Vorgänge protokollieren. Diese Informationen sind für Betreiber nützlich. Wenn ein Dienst häufig nicht verfügbar oder ausgelastet ist, liegt dies oft daran, dass die Ressourcen des Diensts ausgeschöpft sind. Sie können die Häufigkeit dieser Fehler reduzieren, indem Sie den Dienst horizontal skalieren. Wenn beispielsweise ein Datenbankdienst ständig überlastet ist, kann es nützlich sein, die Datenbank zu partitionieren und die Last auf mehrere Server zu verteilen.

> [Microsoft Entity Framework](https://docs.microsoft.com/ef/) bietet Funktionen für das Wiederholen von Datenbankvorgängen. Auch die meisten Azure-Dienste und Client-SDKs enthalten einen Wiederholungsmechanismus. Weitere Informationen finden Sie unter [Wiederholungsanleitung für bestimmte Dienste](https://docs.microsoft.com/azure/architecture/best-practices/retry-service-specific).

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Bei der Entscheidung, wie dieses Muster implementiert werden soll, sind die folgenden Punkte zu beachten.

Die Wiederholungsrichtlinie sollte entsprechend den geschäftlichen Anforderungen der Anwendung und der Art des Fehlers angepasst werden. Bei einigen nicht kritischen Vorgängen ist es besser, Fail-Fast-fähig zu sein, statt sie mehrere Male zu wiederholen und dadurch den Durchsatz der Anwendung zu verringern. Bei einer interaktiven Webanwendung, die auf einen Remotedienst zugreift, ist es beispielsweise besser, nach einer kleineren Anzahl von Wiederholungen mit nur einer kurzen Verzögerung zwischen den Wiederholungsversuchen einen Fehler auszugeben und dem Benutzer eine entsprechende Meldung anzuzeigen (z. B. „Versuchen Sie es später noch einmal“). Bei einer Batchanwendung ist es möglicherweise besser, die Anzahl der Wiederholungsversuche mit einer exponentiell zunehmenden Verzögerung zwischen den einzelnen Versuchen zu erhöhen.

Durch eine aggressive Wiederholungsrichtlinie mit minimaler Verzögerung zwischen den Versuchen und einer großen Anzahl von Wiederholungen kann ein ausgelasteter Dienst, der nahe oder an seiner Kapazitätsgrenze ausgeführt wird, zusätzlich beeinträchtigt werden. Diese Wiederholungsrichtlinie könnte sich auch auf die Reaktionsfähigkeit der Anwendung auswirken, wenn ständig versucht wird, einen fehlgeschlagenen Vorgang auszuführen.

Wenn eine Anforderung nach einer erheblichen Anzahl von Wiederholungen immer noch fehlschlägt, ist es besser, wenn die Anwendung weitere Anforderungen an die gleiche Ressource verhindert und einfach sofort einen Fehler meldet. Nach Ablauf des Zeitraums kann die Anwendung eine oder mehrere Anforderungen vorläufig zulassen, um festzustellen, ob diese erfolgreich ausgeführt werden. Weitere Informationen zu dieser Strategie finden Sie unter [Trennschalter-Muster](circuit-breaker.md).

Achten Sie darauf, ob der Vorgang idempotent ist. Wenn das der Fall ist, ist eine Wiederholung grundsätzlich sicher. Andernfalls könnte der Vorgang bei Wiederholungen mehr als einmal ausgeführt werden, was unbeabsichtigte Nebeneffekte haben kann. Beispielsweise kann ein Dienst die Anforderung empfangen, sie erfolgreich verarbeiten, jedoch das Senden einer Antwort fehlschlagen. An diesem Punkt kann die Wiederholungslogik die Anforderung erneut senden, da davon ausgegangen wird, dass die erste Anforderung nicht empfangen wurde.

Eine Anforderung an einen Dienst kann aus einer Vielzahl von Gründen fehlschlagen, die je nach Art des Fehlers verschiedene Ausnahmen auslösen. Einige Ausnahmen geben einen Fehler an, der schnell behoben werden kann, während andere angeben, dass der Fehler länger andauert. Es ist nützlich, wenn bei der Wiederholungsrichtlinie der Zeitraum zwischen den Wiederholungsversuchen entsprechend dem Typ der Ausnahme angepasst wird.

Beachten Sie, wie sich das Wiederholen eines Vorgangs, der Teil einer Transaktion ist, auf die Transaktionskonsistenz insgesamt auswirkt. Nehmen Sie eine Feinanpassung der Wiederholungsrichtlinie für Transaktionsvorgänge vor, um die Wahrscheinlichkeit einer erfolgreichen Ausführung zu erhöhen und die Notwendigkeit zu verringern, alle Transaktionsschritte rückgängig zu machen.

Stellen Sie sicher, dass der gesamte Wiederholungscode vollständig für eine Vielzahl von Fehlerbedingungen getestet ist. Vergewissern Sie sich, dass er keine schwerwiegenden Auswirkungen auf die Leistungsfähigkeit oder Zuverlässigkeit der Anwendung hat, keine übermäßige Auslastung von Diensten und Ressourcen bewirkt und auch keine Racebedingungen oder Engpässe erzeugt.

Implementieren Sie eine Wiederholungslogik nur unter Berücksichtigung des gesamten Kontexts eines fehlgeschlagenen Vorgangs. Wenn beispielsweise eine Aufgabe, die eine Wiederholungsrichtlinie enthält, eine andere Aufgabe aufruft, die ebenfalls eine Wiederholungsrichtlinie enthält, kann diese zusätzliche Ebene von Wiederholungen zu lange Verzögerungen bei der Verarbeitung führen. Möglicherweise ist es besser, die Aufgabe auf niedrigerer Ebene so zu konfigurieren, dass sie Fail-Fast-fähig ist und die Ursache des Fehlers an die Aufgabe zurückmeldet, von der sie aufgerufen wurde. Diese Aufgabe auf höherer Ebene kann dann den Fehler basierend auf ihrer eigenen Richtlinie behandeln.

Es ist wichtig, alle Konnektivitätsfehler zu protokollieren, die zu einer Wiederholung führen, damit zugrunde liegende Probleme mit der Anwendung, Diensten oder Ressourcen ermittelt werden können.

Untersuchen Sie die Fehler, die am wahrscheinlichsten für einen Dienst oder eine Ressource auftreten können, um festzustellen, ob diese wahrscheinlich lang andauernd oder terminal sind. Wenn das der Fall ist, sollte der Fehler besser als Ausnahme behandelt werden. Die Anwendung kann die Ausnahme melden oder protokollieren und dann fortfahren, indem entweder ein alternativer Dienst (sofern verfügbar) aufgerufen oder eingeschränkte Funktionalität geboten wird. Weitere Informationen zum Erkennen und Behandeln lang andauernder Fehler finden Sie unter der [Trennschalter-Muster](circuit-breaker.md).

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

Verwenden Sie dieses Muster, wenn es bei einer Anwendung zu vorübergehenden Fehlern bei der Interaktion mit einem Remotedienst oder dem Zugriff auf eine Remoteressource kommen kann. Diese Fehler sind erwartungsgemäß von kurzer Dauer und das Wiederholen einer zuvor fehlgeschlagenen Anforderung kann bei einem weiteren Versuch erfolgreich sein.

Dieses Muster ist in folgenden Fällen möglicherweise nicht geeignet:

- Wenn ein Fehler wahrscheinlich länger andauert, da sich dies auf die Reaktionsfähigkeit einer Anwendung auswirken kann. Die Anwendung verschwendet möglicherweise Zeit und Ressourcen bei dem Versuch, eine Anforderung zu wiederholen, die wahrscheinlich fehlschlägt.
- Bei der Behandlung von Fehlern, die nicht auf vorübergehenden Fehlern basieren, z. B. interne Ausnahmen, die auf Fehler in der Geschäftslogik einer Anwendung zurückzuführen sind.
- Als Alternative zur Behebung von Skalierbarkeitsproblemen in einem System. Wenn bei einer Anwendung häufig Auslastungsfehler auftreten, ist dies oft ein Zeichen dafür, dass der Dienst oder die Ressource, auf den bzw. die zugegriffen wird, hochskaliert werden sollte.

## <a name="example"></a>Beispiel

Dieses Beispiel in C# zeigt eine Implementierung des Wiederholungsmusters. Die unten gezeigte Methode `OperationWithBasicRetryAsync` ruft einen externen Dienst asynchron über die Methode `TransientOperationAsync` auf. Die Details der Methode `TransientOperationAsync` entsprechen dem jeweiligen Dienst und sind im Beispielcode ausgelassen.

```csharp
private int retryCount = 3;
private readonly TimeSpan delay = TimeSpan.FromSeconds(5);

public async Task OperationWithBasicRetryAsync()
{
  int currentRetry = 0;

  for (;;)
  {
    try
    {
      // Call external service.
      await TransientOperationAsync();

      // Return or break.
      break;
    }
    catch (Exception ex)
    {
      Trace.TraceError("Operation Exception");

      currentRetry++;

      // Check if the exception thrown was a transient exception
      // based on the logic in the error detection strategy.
      // Determine whether to retry the operation, as well as how
      // long to wait, based on the retry strategy.
      if (currentRetry > this.retryCount || !IsTransient(ex))
      {
        // If this isn't a transient error or we shouldn't retry, 
        // rethrow the exception.
        throw;
      }
    }

    // Wait to retry the operation.
    // Consider calculating an exponential delay here and
    // using a strategy best suited for the operation and fault.
    await Task.Delay(delay);
  }
}

// Async method that wraps a call to a remote service (details not shown).
private async Task TransientOperationAsync()
{
  ...
}
```

Die Anweisung, mit der diese Methode aufgerufen wird, befindet sich in einem try/catch-Block, der in eine for-Schleife eingeschlossen ist. Die for-Schleife wird beendet, wenn die `TransientOperationAsync`-Methode erfolgreich aufgerufen und keine Ausnahme ausgelöst wird. Wenn die `TransientOperationAsync`-Methode fehlschlägt, wird mit dem catch-Block die Ursache des Fehlers überprüft. Wird davon ausgegangen, dass es sich um einen vorübergehenden Fehler handelt, wartet der Code für eine kurzen Verzögerungszeit, bevor der Vorgang wiederholt wird.

Die for-Schleife verfolgt auch die Anzahl, wie oft eine Ausführung des Vorgangs versucht wurde, und wenn der Code dreimal fehlschlägt, wird davon ausgegangen, dass die Ausnahme länger andauert. Wenn die Ausnahme nicht vorübergehend ist oder lang andauert, löst der catch-Handler eine Ausnahme aus. Diese Ausnahme beendet die for-Schleife und sollte von dem Code abgefangen werden, der die `OperationWithBasicRetryAsync`-Methode aufruft.

Die unten gezeigte `IsTransient`-Methode prüft auf einen bestimmten Satz von Ausnahmen, die für die Umgebung relevant sind, in der der Code ausgeführt wird. Die Definition einer vorübergehenden Ausnahme unterscheidet sich je nach Ressourcen, auf die zugegriffen wird, und der Umgebung, in der der Vorgang ausgeführt wird.

```csharp
private bool IsTransient(Exception ex)
{
  // Determine if the exception is transient.
  // In some cases this is as simple as checking the exception type, in other
  // cases it might be necessary to inspect other properties of the exception.
  if (ex is OperationTransientException)
    return true;

  var webException = ex as WebException;
  if (webException != null)
  {
    // If the web exception contains one of the following status values
    // it might be transient.
    return new[] {WebExceptionStatus.ConnectionClosed,
                  WebExceptionStatus.Timeout,
                  WebExceptionStatus.RequestCanceled }.
            Contains(webException.Status);
  }

  // Additional exception checking logic goes here.
  return false;
}
```

## <a name="related-patterns-and-guidance"></a>Zugehörige Muster und Anleitungen

- [Muster „Trennschalter“](circuit-breaker.md). Das Wiederholungsmuster eignet sich zum Behandeln vorübergehender Fehler. Wenn bei einem Fehler davon ausgegangen wird, dass er länger andauert, ist es möglicherweise besser, das Trennschalter-Muster zu implementieren. Das Wiederholungsmuster kann auch in Verbindung mit einem Trennschalter verwendet werden, um einen umfassenden Ansatz zur Behandlung von Fehlern bereitzustellen.
- [Wiederholungsanleitung für bestimmte Dienste](https://docs.microsoft.com/azure/architecture/best-practices/retry-service-specific)
- [Verbindungsstabilität](https://docs.microsoft.com/ef/core/miscellaneous/connection-resiliency)
