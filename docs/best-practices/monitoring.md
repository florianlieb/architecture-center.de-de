---
title: Anleitung zur Überwachung und Diagnose
description: Bewährte Methoden für die Überwachung verteilter Anwendungen in der Cloud.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 8dd3979233b03db800bd9514263d9c6fedefa074
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="monitoring-and-diagnostics"></a>Überwachung und Diagnose
[!INCLUDE [header](../_includes/header.md)]

Verteilte Anwendungen und Dienste in der Cloud sind naturgemäß komplexe Softwarekomponenten, die viele bewegliche Teile umfassen. In einer Produktionsumgebung sollten Sie unbedingt nachvollziehen können, auf welche Weise Benutzer Ihr System verwenden. Ferner müssen Sie in der Lage sein, die Ressourcennutzung nachzuverfolgen und allgemein die Integrität und Leistung des Systems zu überwachen. Diese Informationen dienen als Diagnosehilfe, um Probleme zu erkennen und zu korrigieren sowie potenzielle Probleme frühzeitig zu erkennen und zu verhindern.

## <a name="monitoring-and-diagnostics-scenarios"></a>Überwachung- und Diagnoseszenarien
Durch die Überwachung erhalten einen Einblick in die Funktionsweise eines Systems. Die Überwachung ist ein wichtiger Aspekt beim Einhalten von Zielen zur Servicequalität. Häufige Szenarien für das Sammeln von Überwachungsdaten sind:

* Sicherstellen, dass das System fehlerfrei bleibt.
* Nachverfolgen der Verfügbarkeit des Systems und seiner Komponenten.
* Verwalten von Leistung, um sicherzustellen, dass sich der Durchsatz des Systems nicht unerwartet verschlechtert, während die Menge an Arbeit sich erhöht.
* Sicherstellen, dass das System mit Kunden vereinbarte SLAs (Service Level Agreements) erfüllt.
* Schutz der Privatsphäre und der Sicherheit des Systems, der Benutzer und ihrer Daten.
* Verfolgen der Vorgänge, die zu Überwachungs- oder rechtlichen Zwecken ausgeführt werden.
* Überwachen der täglichen Nutzung des Systems und Ermitteln von Trends, die zu Problemen führen könnten, wenn sie nicht behandelt werden.
* Nachverfolgen von auftretenden Problemen, vom ersten Bericht bis zur Analyse der möglichen Ursachen, Berichtigung, folgende Softwareupdates und Bereitstellung.
* Ablaufverfolgung von Vorgängen und das Debuggen von Software-Versionen.

> [!NOTE]
> Diese Liste erhebt keinen Anspruch auf Vollständigkeit. In diesem Dokument liegt der Schwerpunkt auf diesen Szenarien, da sie die gängigsten Überwachungssituationen darstellen. Es gibt möglicherweise weitere Szenarien, die weniger häufig vorkommen oder für Ihre Umgebung spezifisch sind.
> 
> 

In den folgenden Abschnitten werden diese Szenarien ausführlich beschrieben. Die Informationen für jedes Szenario werden im folgenden Format behandelt:

1. Eine kurze Übersicht über das Szenario
2. Die typischen Anforderungen dieses Szenarios
3. Die unformatierten Instrumentierungsdaten, die zur Unterstützung des Szenarios erforderlich sind, und mögliche Quellen für diese Informationen
4. Wie Rohdaten analysiert und kombiniert werden können, um sinnvolle Diagnoseinformationen zu generieren

## <a name="health-monitoring"></a>Systemüberwachung
Ein System ist fehlerfrei, wenn es ausgeführt wird und Anforderungen verarbeiten kann. Die Systemüberwachung dient zum Generieren einer Momentaufnahme der aktuellen Systemintegrität, damit Sie überprüfen können, ob alle Komponenten des Systems wie erwartet funktionieren.

### <a name="requirements-for-health-monitoring"></a>Anforderungen für die Systemüberwachung
Ein Operator sollte schnell (innerhalb weniger Sekunden) benachrichtigt werden, wenn ein Teil des Systems als fehlerhaft eingestuft wird. Der Operator sollte ermitteln können, welche Teile des Systems ordnungsgemäß ausgeführt werden und in welchen Teile Probleme auftreten. Die Systemintegrität kann über ein Ampelsystem angezeigt werden:

* Rot für fehlerhaft (das System wurde beendet)
* Gelb für teilweise fehlerfrei (das System wird mit eingeschränkter Funktionalität ausgeführt)
* Grün für völlig fehlerfrei

Eine umfassende Systemüberwachung ermöglicht es einem Operator, der einen Drilldown durch das System zum Anzeigen des Integritätsstatus von Subsystemen und Komponenten ausführen. Wenn z. B. das gesamte System als teilweise fehlerfrei dargestellt ist, sollte der Operator in der Lage sein, es zu vergrößern und zu bestimmen, welche Funktionalität zurzeit nicht verfügbar ist.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Datenquellen, Instrumentierung und Anforderungen für die Datensammlung
Die Rohdaten, die zur Unterstützung der Systemüberwachung erforderlich sind, können als Ergebnis folgender Aktionen generiert werden:

* Nachverfolgen der Ausführung von Benutzeranforderungen. Diese Informationen können verwendet werden, um zu bestimmen, welche Anforderungen erfolgreich ausgeführt wurden, welche fehlgeschlagen sind und wie lange jede Anforderung dauert.
* Synthetische Benutzerüberwachung. Dieser Vorgang simuliert die Schritte, die von einem Benutzer durchgeführt werden, und folgt einer vordefinierten Reihe von Schritten. Die Ergebnisse der einzelnen Schritte sollten erfasst werden.
* Protokollieren von Ausnahmen, Fehlern und Warnungen. Diese Informationen können als Ergebnis von Überwachungsanweisungen erfasst werden, die im Anwendungscode eingebettet sind, sowie durch Abrufen von Informationen aus den Ereignisprotokollen der Dienste, auf die das System verweist.
* Überwachen des Zustands von Drittanbieterdiensten, die vom System verwendet werden. Für diese Überwachung müssen möglicherweise Integritätsdaten abgerufen und analysiert werden, die von diesen Dienste bereitgestellt werden. Diese Informationen können in einer Vielzahl von Formaten vorliegen.
* Endpunktüberwachung. Dieser Mechanismus wird im Abschnitt „Verfügbarkeitsüberwachung“ ausführlicher beschrieben.
* Sammeln von umgebenden Leistungsinformationen, z. B. CPU-Auslastung im Hintergrund oder E/A-Aktivität (einschließlich Netzwerk).

### <a name="analyzing-health-data"></a>Analysieren von Zustandsdaten
Die Systemüberwachung soll hauptsächlich rasch angeben können, ob das System ausgeführt wird. Durch die sofortige Analyse heißer Daten kann eine Warnung ausgelöst werden, wenn eine kritische Komponente als fehlerhaft erkannt wird. (Beispielsweise reagiert sie nicht auf eine aufeinanderfolgende Reihe von Pings.) Der Operator kann dann die entsprechenden Korrekturmaßnahmen einleiten.

Ein fortgeschritteneres System kann ein Prognoseelement umfassen, das eine kalte Analyse über die letzten und die aktuellen Workloads ausführt. Durch eine kalte Analyse können Sie Trends ermitteln und feststellen, ob das System wahrscheinlich fehlerfrei bleibt oder ob das System zusätzliche Ressourcen benötigt. Dieses Prognoseelement sollte auf wichtigen Leistungsmetriken wie den folgenden basieren:

* Die Rate der Anforderungen, die an die einzelnen Dienste oder Subsysteme gerichtet werden.
* Die Antwortzeiten dieser Anforderungen.
* Das Datenvolumen, das bei den einzelnen Diensten eingeht und ausgegeben wird.

Wenn der Wert einer Metrik einen festgelegten Schwellenwert überschreitet, kann das System eine Warnung auslösen, damit von einem Operator oder durch eine automatische Skalierung (falls verfügbar) vorbeugende Maßnahmen für das Aufrechterhalten der Systemintegrität ergriffen werden können. Diese Aktionen beinhalten möglicherweise das Hinzufügen von Ressourcen, den Neustart eines oder mehrerer fehlschlagender Dienste oder die Drosselung auf Anforderungen mit niedrigerer Priorität.

## <a name="availability-monitoring"></a>Verfügbarkeitsüberwachung
Ein wirklich fehlerfreies System erfordert, dass die Komponenten und Subsysteme, aus denen das System besteht, verfügbar sind. Die Verfügbarkeitsüberwachung ist mit der Integritätsüberwachung eng verknüpft. Während die Integritätsüberwachung jedoch eine sofortige Ansicht des aktuellen Systemzustands bereitstellt, ist die Verfügbarkeitsüberwachung für das Nachverfolgen der Verfügbarkeit des Systems und seiner Komponenten zuständig und erstellt Statistiken über die Betriebszeit des Systems.

In vielen Systemen sind einige Komponenten (z. B. eine Datenbank) mit integrierten Redundanz konfiguriert, um schnelles Failover im Falle eines schwerwiegenden Fehlers oder eines Verlusts der Verbindung zuzulassen. Im Idealfall sollten Benutzer gar nicht bemerken, dass ein Fehler aufgetreten ist. Aus der Perspektive der Verfügbarkeitsüberwachung ist es jedoch notwendig, so viele Informationen wie möglich über solche Fehler zu sammeln, um die Ursache zu ermitteln und Korrekturmaßnahmen zu ergreifen, damit sie sich nicht wiederholen.

Die Daten, die zum Nachverfolgen der Verfügbarkeit erforderlich sind, können von einer Reihe von Faktoren auf niedrigerer Ebene abhängen. Viele dieser Faktoren beziehen sich möglicherweise speziell auf Anwendung, System und Umgebung. Ein effektives Überwachungssystem erfasst die Verfügbarkeitsdaten, die diesen Faktoren auf niedriger Ebene entsprechen, und aggregiert diese anschließend, um einen allgemeinen Überblick über das System zu erhalten. Beispielsweise können in einem E-Commerce-System die Geschäftsfunktionen, mit denen der Kunde Bestellungen aufgeben kann, vom Repository abhängen, in dem Bestelldetails gespeichert sind, und vom Zahlungssystem, das die finanziellen Transaktionen für die Zahlung dieser Aufträge verarbeitet. Die Verfügbarkeit des Systemteils, der für die Platzierung von Bestellungen zuständig ist, ist daher eine Funktion der Verfügbarkeit des Repositorys und des Zahlungssubsystems.

### <a name="requirements-for-availability-monitoring"></a>Anforderungen für die Verfügbarkeitsüberwachung
Ein Operator sollte auch die bisherige Verfügbarkeit der einzelnen Systeme und Subsysteme anzeigen können und anhand dieser Informationen Trends erkennen, die in einem oder mehreren Subsystemen in regelmäßigen Abständen Fehler verursachen. (Fallen Dienste zu einer bestimmten Tageszeit aus, die den Spitzenverarbeitungszeiten entspricht?)

Eine Überwachungslösung sollte eine unmittelbare und historische Ansicht der Verfügbarkeit bzw. Nichtverfügbarkeit der einzelnen Subsysteme bereitstellen. Sie sollte außerdem einen Operator schnell warnen können, wenn mindestens ein Dienst ausfällt oder wenn Benutzer keine Verbindung mit Diensten herstellen können. Hierbei geht es nicht nur darum, die einzelnen Dienste zu überwachen, sondern auch darum, die von den einzelnen Benutzern durchgeführten Aktionen zu untersuchen, wenn beim Kommunikationsversuch mit einem Dienst ein Fehler auftritt. In gewissem Umfang sind Verbindungsfehler normal und können durch vorübergehende Fehler verursacht werden. Es kann jedoch hilfreich sein, dem System das Ausgeben einer Warnung für die Anzahl von Verbindungsfehlern mit einem bestimmten Subsystem zu ermöglichen, die während eines bestimmten Zeitraums auftreten.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Datenquellen, Instrumentierung und Anforderungen für die Datensammlung
Wie bei der Integritätsüberwachung können die Rohdaten zur Unterstützung der Verfügbarkeitsüberwachung als Ergebnis der synthetischen Benutzerüberwachung und der Protokollierung aller Ausnahmen, Fehler und Warnungen generiert werden. Darüber hinaus können Verfügbarkeitsdaten durch eine Endpunktüberwachung abgerufen werden. Die Anwendung kann einen oder mehrere Integritätsendpunkte offenlegen, von dem jeder den Zugriff auf den Funktionsbereich innerhalb des Systems prüft. Das Überwachungssystem kann jeden Endpunkt nach einem definierten Zeitplan pingen und die Ergebnisse erfassen (erfolgreich oder Fehler).

Alle Timeouts, Netzwerkkonnektivitätsfehler und wiederholte Verbindungsversuche müssen aufgezeichnet werden. Alle Daten sollten mit einem Zeitstempel versehen sein.

<a name="analyzing-availability-data"></a>

### <a name="analyzing-availability-data"></a>Analysieren von Verfügbarkeitsdaten
Die Instrumentierungsdaten müssen zusammengefasst und korreliert werden, um die folgenden Arten der Analyse zu unterstützen:

* Die sofortige Verfügbarkeit des Systems und der Subsysteme.
* Die Fehlerraten für die Verfügbarkeit des Systems und der Subsysteme. Im Idealfall sollte ein Operator Fehler mit bestimmten Aktivitäten in Beziehung setzen können: Was ist gerade passiert, als der Systemfehler auftrat?
* Eine Verlaufsansicht mit den Fehlerraten des Systems oder der Subsysteme in einem angegeben Zeitraum und die Auslastung des Systems (z. B. die Anzahl von Benutzeranforderungen), als ein Fehler aufgetreten ist.
* Die Gründe für die Nichtverfügbarkeit des Systems oder eines Subsystems. Mögliche Gründe sind zum Beispiel, dass ein Dienst nicht ausgeführt wird, dass die Verbindung verloren gegangen ist oder dass die Verbindung zwar besteht, aber ein Timeout auftritt oder Fehler zurückgegeben werden.

Sie können die prozentuale Verfügbarkeit eines Diensts über einen Zeitraum mit folgender Formel berechnen:

```
%Availability =  ((Total Time – Total Downtime) / Total Time ) * 100
```

Dies ist zu SLA-Zwecken hilfreich. (Die [SLA-Überwachung](#SLA-monitoring) wird weiter unten in diesem Leitfaden ausführlicher beschrieben.) Die Definition von *Downtime* hängt vom Dienst ab. Z. B. definiert der Visual Studio Team Services-Builddienst Ausfallzeiten als den Zeitraum (Gesamtzahl der Minuten), in dem der Builddienst nicht verfügbar ist. Eine Minute wird als nicht verfügbar betrachtet, wenn alle fortlaufend an den Builddienst gesendeten HTTP-Anforderungen zum Ausführen kundeninitiierter Vorgänge während dieser Minute zu einem Fehlercode führen oder keine Antwort zurückgeben.

## <a name="performance-monitoring"></a>Leistungsüberwachung
Während das System (durch eine wachsende Anzahl von Benutzern) immer mehr belastet wird, wächst die Größe der Datasets, auf die diese Benutzer zugreifen, und die Fehlerwahrscheinlichkeit für die einzelnen Komponenten nimmt zu. Häufig ist eine Abnahme der Leistung Komponentenfehlern vorangestellt. Wenn Sie in der Lage sind, eine solche Leistungsabnahme zu erkennen, können Sie proaktive Maßnahmen zur Abhilfe ergreifen.

Die Systemleistung hängt von einer Reihe von Faktoren ab. Jeder dieser Faktoren wird in der Regel mithilfe von Leistungskennzahlen (Key Performance Indicators, KPIs) gemessen, z. B. die Anzahl von Datenbanktransaktionen pro Sekunde oder die Menge der Netzwerkanforderungen, die in einem bestimmten Zeitrahmen erfolgreich bearbeitet werden. KPIs können als spezifische Leistungskennzahlen zur Verfügung stehen oder aus einer Kombination von Metriken abgeleitet werden.

> [!NOTE]
> Das Bestimmen einer schlechten oder guten Leistung erfordert, dass Sie das Leistungsniveau verstehen, mit der das System betrieben werden können sollte. Dies erfordert die Beobachtung des Systems, während es bei einer durchschnittlichen Belastung arbeitet, sowie das Erfassen der Daten für die einzelnen KPIs über einen Zeitraum. Dabei wird z. B. die Auslastung des laufenden Systems in einer Testumgebung simuliert und die entsprechenden Daten gesammelt, und zwar vor der Bereitstellung des Systems in einer Produktionsumgebung.
> 
> Sie sollten auch sicherstellen, dass die Überwachung zu Leistungszwecken keine Belastung für das System darstellt. Sie können möglicherweise die Detailebene für die in der Leistungsüberwachung gesammelten Daten dynamisch anpassen.
> 
> 

### <a name="requirements-for-performance-monitoring"></a>Anforderungen für die Leistungsüberwachung
Um die Systemleistung zu untersuchen, muss ein Operator in der Regel Informationen wie diese erhalten:

* Die Antwortrate bei Benutzeranforderungen.
* Die Anzahl gleichzeitiger Benutzeranfragen.
* Das Volumen des Netzwerkverkehrs.
* Die Sätze, zu denen geschäftliche Transaktionen vorgenommen werden.
* Die durchschnittliche Verarbeitungszeit für Anforderungen.

Es kann auch hilfreich sein, Tools zur Verfügung zu stellen, die einem Operator das Erkennen von Korrelationen ermöglichen, z. B.:

* Die Anzahl gleichzeitiger Benutzer im Vergleich zu den Anforderungslatenzzeiten (wie lange es bis zum Starten der Anforderungsverarbeitung dauert, nachdem der Benutzer sie gesendet hat).
* Die Anzahl gleichzeitiger Benutzer im Vergleich zur durchschnittlichen Antwortzeit (wie lange es bis zum Abschluss einer Anforderung dauert, nachdem die Verarbeitung begonnen hat).
* Das Anforderungsvolumen im Vergleich zur Anzahl von Fehlern bei der Verarbeitung.

Neben diesen allgemeinen funktionalen Informationen sollte ein Operator auch eine detaillierte Ansicht der Leistung für die einzelnen Komponenten im System erhalten. Diese Daten werden in der Regel mithilfe von Low-Level-Leistungsindikatoren bereitgestellt, die u. a. folgende Informationen erfassen:

* Speicherauslastung.
* Anzahl der Threads.
* CPU-Verarbeitungszeit.
* Länge der Anforderungswarteschlange.
* Datenträger- oder Netzwerk-E/A-Raten und -Fehler.
* Anzahl gelesener oder geschriebener Bytes.
* Middleware-Indikatoren, z. B. die Warteschlangenlänge.

Bei allen Visualisierungen sollte ein Operator die Möglichkeit erhalten, einen Zeitraum festzulegen. Die angezeigten Daten können eine Momentaufnahme der aktuellen Situation und/oder eine Verlaufsansicht der Leistung sein.

Ein Operator sollte basierend auf den Leistungsmeasures eine Warnung für jeden angegebenen Wert während eines angegebenen Zeitintervalls ausgeben können.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Datenquellen, Instrumentierung und Anforderungen für die Datensammlung
Durch Überwachen der Fortschritte von eingehenden und das System passierenden Benutzeranforderungen können Sie allgemeine Leistungsdaten sammeln (Durchsatz, Anzahl gleichzeitiger Benutzer, Anzahl von Geschäftstransaktionen, Fehlerraten und so weiter). Dies beinhaltet das Einbinden von Ablaufverfolgungsanweisungen an wichtigen Punkten im Anwendungscode zusammen mit Informationen zur zeitlichen Steuerung. Alle Fehler, Ausnahmen und Warnungen sollten mit einer ausreichenden Datenmenge erfasst werden, damit ein Bezug zu den Anforderungen hergestellt werden kann, die sie verursacht haben. Das IIS-Protokoll (Internet Information Services) ist eine weitere nützliche Datenquelle.

Wenn möglich, sollten Sie Leistungsdaten für alle externen Systeme erfassen, die die Anwendung verwendet. Diese externen Systeme bieten möglicherweise eigene Leistungsindikatoren oder andere Features für die Anforderung von Leistungsdaten. Wenn dies nicht möglich ist, erfassen Sie Informationen wie z. B. die Start- und Endzeit der einzelnen Anforderungen an ein externes System zusammen mit dem Status des Vorgangs (Erfolgreich, Fehler oder Warnung). Beispielsweise können Sie einen Stoppuhransatz verfolgen, um Anforderungen zeitlich zu erfassen: Starten Sie einen Timer, wenn die Anforderung gestartet wird, und stoppen Sie den Timer, wenn die Anforderung abgeschlossen ist.

Detaillierte Low-Level-Leistungsdaten für einzelne Komponenten in einem System stehen möglicherweise über Features und Dienste wie Windows-Leistungsindikatoren und Azure-Diagnose zur Verfügung.

### <a name="analyzing-performance-data"></a>Analysieren von Leistungsdaten
Ein Großteil der Analysearbeit besteht aus dem Aggregieren von Leistungsdaten nach Benutzeranforderungstyp und/oder nach Subsystem oder Dienst, an das bzw. den die einzelnen Anforderungen gesendet werden. Ein Beispiel für eine Benutzeranforderung ist das Hinzufügen eines Elements zu einem Einkaufswagen oder der Auftragsabschluss in einem E-Commerce-System.

Eine weitere häufige Anforderung ist die Zusammenfassung von Leistungsdaten in ausgewählten Prozentsätzen. Beispielsweise kann ein Operator die Antwortzeiten für 99 Prozent der Anforderungen, 95 Prozent der Anforderungen und 70 Prozent der Anforderungen ermitteln. Möglicherweise sind für jeden Prozentwert SLA-Ziele oder andere Ziele festgelegt. Die fortlaufenden Ergebnisse sollten nahezu in Echtzeit gemeldet werden, um Probleme sofort zu erkennen. Außerdem sollten die Ergebnisse zu statistischen Zwecken über längere Zeit aggregiert werden.

Im Fall von leistungsbeeinträchtigenden Latenzproblemen sollte ein Operator in der Lage sein, die Ursache für den Engpass schnell zu identifizieren, indem er die Latenzzeit der einzelnen Schritte jeder Anforderung untersucht. Die Leistungsdaten müssen daher ein Mittel zum Korrelieren von Leistungsindikatoren für jeden Schritt bereitstellen, um sie mit einer bestimmten Anforderung zu verknüpfen.

Je nach Visualisierungsanforderungen kann es hilfreich sein, einen Datencube zu generieren und zu speichern, der Ansichten der Rohdaten enthält. Dieser Datencube kann komplexe Ad-hoc-Abfragen und die Analyse der Leistungsdaten ermöglichen.

## <a name="security-monitoring"></a>Sicherheitsüberwachung
Alle kommerziellen Systeme, die sensible Daten enthalten, müssen eine Sicherheitsstruktur implementieren. Die Komplexität des Sicherheitsmechanismus ist in der Regel eine Funktion der Sensibilität der Daten. In einem System, das die Authentifizierung von Benutzern verlangt, sollten Sie Folgendes aufzeichnen:

* Alle Anmeldeversuche, sowohl fehlerhafte als auch erfolgreiche.
* Alle von einem authentifizierten Benutzer ausgeführten Vorgänge sowie die Details aller Ressourcen, auf die der Benutzer zugreift.
* Zeitpunkt, zu dem ein Benutzer eine Sitzung beendet und sich abmeldet.

Die Überwachung kann dabei helfen, Angriffe auf das System zu erkennen. Zum Beispiel kann eine große Anzahl misslungener Anmeldeversuche auf einen Brute-Force-Angriff hinweisen. Ein unerwarteter Anstieg von Anforderungen kann das Ergebnis eines verteilten Denial-of-Service-Angriffs (DDoS) sein. Sie müssen vorbereitet sein, um alle Anforderungen an alle Ressourcen unabhängig von der Quelle dieser Anforderungen zu überwachen. Ein System mit einer Sicherheitslücke bei der Anmeldung stellt möglicherweise unbeabsichtigt Ressourcen zum externen Gebrauch zur Verfügung, ohne dass ein Benutzer sich dafür anmelden muss.

### <a name="requirements-for-security-monitoring"></a>Anforderungen für die Sicherheitsüberwachung
Die wichtigsten Aspekte der Sicherheitsüberwachung sollten einem Operator schnell Folgendes ermöglichen:

* Erkennen von Angriffsversuchen von einer nicht authentifizierten Entität.
* Identifizieren von Versuchen von Entitäten, Vorgänge für Daten auszuführen, für die ihnen kein Zugriff erteilt wurde.
* Ermitteln, ob das System oder ein Teil des Systems einem Angriff von außerhalb oder von innerhalb ausgesetzt ist. (Beispielsweise kann ein böswilliger authentifizierter Benutzer versuchen, einen Systemausfall zu verursachen.)

Um diese Anforderungen zu unterstützen, sollte ein Operator über Folgendes benachrichtigt werden:

* Wenn ein Konto innerhalb eines angegebenen Zeitraums wiederholt fehlerhafte Anmeldeversuche unternimmt.
* Wenn ein authentifiziertes Konto während eines angegebenen Zeitraums wiederholt versucht, auf eine verbotene Ressource zuzugreifen.
* Wenn während eines angegebenen Zeitraums eine große Anzahl nicht authentifizierter oder nicht autorisierter Anforderungen auftritt.

Die einem Operator bereitgestellten Informationen sollten für jede Anforderung die Hostadresse der Quelle beinhalten. Wenn Sicherheitsverletzungen regelmäßig aus einem bestimmten Adressbereich auftreten, könnten diese Hosts blockiert werden.

Eine wichtige Rolle bei der Wahrung der Sicherheit eines Systems ist das schnelle Erkennen von Aktionen, die vom üblichen Muster abweichen. Informationen wie z. B. die Anzahl fehlerhafter und/oder erfolgreicher Anmeldeanforderungen kann visuell dargestellt werden, um zu erkennen, ob zu einer ungewöhnlichen Zeit eine Spitze in der Aktivität vorliegt. (Ein Beispiel für diese Aktivität ist, wenn Benutzer sich um 3:00 Uhr morgens anmelden und eine große Anzahl von Vorgängen ausführen, wenn der Arbeitstag um 9:00 Uhr beginnt). Anhand dieser Informationen kann auch die zeitbasierte automatische Skalierung konfiguriert werden. Wenn z. B. ein Operator beobachtet, dass eine große Anzahl von Benutzern sich regelmäßig zu einer bestimmten Tageszeit anmeldet, kann er den Start zusätzlicher Authentifizierungsdienste veranlassen, um das Arbeitsvolumen verarbeiten zu können. Er kann diese zusätzlichen Dienste wieder beenden, wenn die Spitze vorbei ist.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Datenquellen, Instrumentierung und Anforderungen für die Datensammlung
Sicherheit ist bei den meisten verteilten Systemen ein allumfassender Aspekt. Die relevanten Daten werden wahrscheinlich an mehreren Punkten in einem System generiert. Sie sollten die Implementierung eines SIEM-Ansatzes (Security Information and Event Management) erwägen, um die Sicherheitsinformationen aus Ereignissen zu sammeln, die von der Anwendung, Netzwerkgeräten, Servern, Firewalls, Antivirensoftware und anderen Elementen des Eindringschutzes ausgelöst wurden.

Die Sicherheitsüberwachung kann Daten aus Tools integrieren, die nicht Teil der Anwendung sind. Dazu gehören beispielsweise Hilfsprogramme, die Portüberwachungsaktivitäten durch externe Einrichtungen ermitteln, oder Netzwerkfilter, die nicht authentifizierte Zugriffsversuche auf Anwendung und Daten erkennen.

In allen Fällen muss ein Administrator anhand der erfassten Daten die Art des Angriffs ermitteln und die entsprechenden Gegenmaßnahmen einleiten können.

### <a name="analyzing-security-data"></a>Analysieren von Sicherheitsdaten
Ein Feature der Sicherheitsüberwachung ist die Vielzahl von Quellen, aus denen die Daten entstehen. Die verschiedenen Formate und Detailebenen erfordern häufig eine komplexe Analyse der erfassten Daten, um sie zu einem Thread zusammenhängender Informationen zusammenzufassen. Abgesehen von den einfachsten Fällen (z. B. dem Erkennen einer großen Anzahl fehlerhafter Anmeldungen oder wiederholter nicht autorisierter Zugriffsversuche auf wichtige Ressourcen) ist es gelegentlich nicht möglich, eine komplexe automatisierte Verarbeitung von Sicherheitsdaten durchzuführen. Stattdessen kann es sich anbieten, diese Daten mit einem Zeitstempel, ansonsten aber in ihrer ursprünglichen Form, in ein sicheres Repository zu schreiben, um sie einer manuellen Expertenanalyse zu unterziehen.

<a name="SLA-monitoring"></a>

## <a name="sla-monitoring"></a>SLA-Überwachung
Viele kommerzielle Systeme, die zahlende Kunden unterstützen, bieten Garantien in Bezug auf die Leistung des Systems in Form von SLAs. Im Wesentlichen wird durch SLAs festgelegt, dass das System ein definiertes Arbeitsvolumen innerhalb eines vereinbarten Zeitrahmens und ohne Verlust wichtiger Informationen verarbeiten kann. Die SLA-Überwachung soll sicherstellen, dass das System messbare SLAs erfüllen kann.

> [!NOTE]
> Die SLA-Überwachung ist mit der Leistungsüberwachung eng verknüpft. Während die Leistungsüberwachung jedoch sicherstellen soll, dass das System *optimal* funktioniert, unterliegt die SLA-Überwachung einer vertraglichen Verpflichtung, die definiert, was *optimal* eigentlich bedeutet.
> 
> 

SLAs werden häufig wie folgt definiert:

* Allgemeine Systemverfügbarkeit. Beispielsweise kann eine Organisation garantieren, dass das System zu 99,9 Prozent der Zeit zur Verfügung steht. Dies entspricht einer Ausfallzeit von nicht mehr als 9 Stunden pro Jahr oder ungefähr 10 Minuten pro Woche.
* Operativer Durchsatz. Dieser Aspekt wird oft als eine oder mehrere „Hochwassermarken“ ausgedrückt, z. B. die Garantie, dass das System bis zu 100.000 Benutzeranfragen gleichzeitig unterstützt oder 10.000 Geschäftstransaktionen gleichzeitig verarbeiten kann.
* Operative Reaktionszeit. Das System kann auch die Geschwindigkeit garantieren, mit der Anforderungen verarbeitet werden. Ein Beispiel dafür ist, dass 99 Prozent aller Geschäftstransaktionen innerhalb von 2 Sekunden abgeschlossen werden und dass keine einzelne Transaktion länger als 10 Sekunden dauert.

> [!NOTE]
> Einige Verträge für kommerzielle Systeme können auch SLAs für den Kundensupport umfassen. Beispielsweise könnten alle Helpdeskanfragen innerhalb von fünf Minuten beantwortet und 99 Prozent aller Probleme innerhalb eines Arbeitstags vollständig behoben werden. Effektive [Problemverfolgung](#issue-tracking) (weiter unten in diesem Abschnitt beschrieben) ist der Schlüssel für die Einhaltung von SLAs wie diesen.
> 
> 

### <a name="requirements-for-sla-monitoring"></a>Anforderungen für die SLA-Überwachung
Auf der höchsten Ebene sollte ein Operator auf einen Blick feststellen können, ob das System die vereinbarten SLAs erfüllt oder nicht. Falls nicht, sollte der Operator einen Drilldown durchführen und die zugrunde liegenden Faktoren untersuchen können, um die Gründe für die nicht dem Standard entsprechende Leistung zu ermitteln.

Typische Indikatoren höherer Ebene, die visuell dargestellt werden können, umfassen:

* Der Prozentsatz der Betriebszeit des Dienstes.
* Der Anwendungsdurchsatz (gemessen in Bezug auf erfolgreiche Transaktionen und/oder Vorgänge pro Sekunde).
* Die Anzahl der Anforderungen für die erfolgreiche/fehlgeschlagene Anwendung.
* Die Anzahl der Anwendungs- und Systemfehler, Ausnahmen und Warnungen.

All diese Indikatoren sollten nach einem bestimmten Zeitraum gefiltert werden können.

Eine Cloud-Anwendung umfasst wahrscheinlich eine Anzahl von Subsystemen und Komponenten. Ein Operator sollte auf höherer Ebene einen Indikator auswählen und sehen können, wie sich dieser aus der Integrität der zugrunde liegenden Elementen zusammensetzt. Wenn z. B. die Verfügbarkeit des gesamten Systems einen zulässigen Wert unterschreitet, sollte ein Operator in der Lage sein, Details anzuzeigen und zu ermitteln, welche Elemente zu diesem Fehler beitragen.

> [!NOTE]
> Die Verfügbarkeit der Systeme muss sorgfältig definiert werden. In einem System, das Redundanz verwendet, um eine maximale Verfügbarkeit zu gewährleisten, können einzelne Instanzen der Elemente fehlschlagen, aber das System bleibt dennoch funktionsfähig. Die durch die Systemüberwachung dargestellte Systemverfügbarkeit sollte die aggregierte Betriebszeit von jedem Element angeben und nicht unbedingt, ob das System tatsächlich angehalten wurde. Darüber hinaus können Fehler isoliert werden. Also auch wenn ein bestimmtes System nicht verfügbar ist, ist möglicherweise der Rest des Systems verfügbar, wenn auch mit verringerter Funktionalität. (In einem E-Commerce-System kann ein Fehler im System verhindern, dass ein Kunde Bestellungen aufgibt, aber der Kunde kann möglicherweise weiterhin den Produktkatalog durchsuchen.)
> 
> 

Zu Warnzwecken sollte das System möglicherweise ein Ereignis auslösen, wenn einer der allgemeinen Indikatoren den angegebenen Schwellenwert überschreitet. Die Low-Level-Details der verschiedenen Faktoren, aus denen sich der High-Level-Indikator zusammensetzt, sollten dem Warnsystem als Kontextdaten zur Verfügung stehen.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Datenquellen, Instrumentierung und Anforderungen für die Datensammlung
Die Rohdaten, die zur Unterstützung der SLA-Überwachung erforderlich sind, ähneln den Rohdaten, die für die Leistungsüberwachung benötigt werden, und umfassen zudem einige Aspekte der Integritäts- und Verfügbarkeitsüberwachung. (Weitere Informationen hierzu finden Sie in den betreffenden Abschnitten.) Sie können diese Daten erfassen durch:

* Durchführen von Endpunkt-Überwachung
* Protokollieren von Ausnahmen, Fehlern und Warnungen.
* Nachverfolgen der Ausführung von Benutzeranforderungen.
* Überwachen der Verfügbarkeit von Drittanbieterdiensten, die vom System verwendet werden.
* Verwenden von Leistungsmetriken und -indikatoren.

Alle Daten müssen zeitlich festgelegt und mit einem Zeitstempel versehen werden.

### <a name="analyzing-sla-data"></a>Analysieren von SLA-Daten
Die Instrumentierungsdaten müssen aggregiert werden, um einen Überblick über die allgemeine Leistung des Systems zu generieren. Aggregierte Daten müssen auch einen Drilldown unterstützen, um eine Untersuchung der Leistung der zugrunde liegenden Subsysteme zu ermöglichen. Sie sollten z. B. in der Lage sein:

* Die Gesamtzahl der Benutzeranforderungen in einem bestimmten Zeitraum zu berechnen und die Erfolgs- und Fehlerereignisrate dieser Anforderungen zu ermitteln.
* Die Antwortzeiten von Benutzeranforderungen zu kombinieren, um einen allgemeinen Überblick über die Reaktionszeiten zu generieren.
* Den Status der Benutzeranforderungen zu analysieren, um die gesamte Antwortzeit einer Anforderung in die Antwortzeiten der einzelnen Arbeitselemente in dieser Anforderung zu unterteilen.  
* Die Gesamtverfügbarkeit des Systems als Betriebszeit in Prozent für einen bestimmten Zeitraum zu ermitteln.
* Die zeitliche Verfügbarkeit in Prozent für die einzelnen Komponenten und Dienste im System zu analysieren. Hierzu gehört z. B. die Analyse von Protokollen, die von Drittanbieterdiensten erstellt wurden.

Viele kommerzielle Systeme sind erforderlich, um reale Leistungsangaben gegen vereinbarte SLAs für einen bestimmten Zeitraum zu melden, in der Regel einen Monat. Diese Informationen können verwendet werden, um Guthaben oder andere Formen der Rückzahlung für Kunden zu berechnen, wenn während dieses Zeitraums die SLAs nicht erfüllt werden. Sie können die Verfügbarkeit für einen Dienst berechnen, indem Sie die im Abschnitt [Analysieren von Verfügbarkeitsdaten](#analyzing-availability-data)beschriebene Technik verwenden.

Für interne Zwecke kann eine Organisation auch die Anzahl und Art der Vorfälle nachverfolgen, die die Ursache für das Fehlschlagen von Diensten waren. Erfahren Sie, wie Sie diese Probleme schnell beheben oder vollständig beseitigen. Dies trägt zur Reduzierung von Ausfallzeiten bei und dient der Erfüllung der SLAs.

## <a name="auditing"></a>Überwachung
Je nach Art der Anwendung gibt es möglicherweise gesetzliche Bestimmungen, die Anforderungen für die Überwachung von Benutzervorgängen und für das Aufzeichnen des gesamten Datenzugriffs festlegen. Die Überwachung kann nachweisen, dass Kunden mit bestimmten Anforderungen verknüpft sind. Nichtabstreitbarkeit ist ein wichtiger Faktor bei vielen E-Business-Systemen zur Erhaltung der Vertrauensstellung zwischen einem Kunden und der Organisation, die für die Anwendung oder den Dienst zuständig ist.

### <a name="requirements-for-auditing"></a>Anforderungen für das Überwachen
Analysten müssen in der Lage sein, die Abfolge der von den Benutzern durchgeführten Geschäftsvorgänge zu verfolgen, um die Aktionen des Benutzers nachvollziehen zu können. Dies kann einfach zu Aufzeichnungszwecken oder im Rahmen einer forensischen Untersuchung erforderlich sein.

Überwachungsinformationen sind sehr vertraulich. Wahrscheinlich umfassen sie Daten, durch die die Benutzer des Systems sowie die von ihnen ausgeführten Aufgaben identifiziert werden. Aus diesem Grund werden Überwachungsinformationen wahrscheinlich in Form von Berichten bereitgestellt, die nur vertrauenswürdigen Analysten zur Verfügung stehen, und nicht als interaktives System, das grafische Drilldownvorgänge unterstützt. Ein Analyst sollte eine Reihe von Berichten generieren können. Beispielsweise können Berichte alle Benutzeraktivitäten während eines angegebenen Zeitraums auflisten, die Abfolge der Aktivitäten für einen einzelnen Benutzer beschreiben oder die aufeinanderfolgenden Vorgänge für eine oder mehrere Ressourcen umfassen.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Datenquellen, Instrumentierung und Anforderungen für die Datensammlung
Die primären Informationsquellen für die Überwachung können umfassen:

* Das Sicherheitssystem, das die Benutzerauthentifizierung verwaltet.
* Ablaufverfolgungsprotokolle, die die Benutzeraktivität aufzeichnen.
* Sicherheitsprotokolle zur Nachverfolgung aller identifizierbaren und nicht identifizierbaren Netzwerkanforderungen.

Das Format der Überwachungsdaten und die Form, in der sie gespeichert sind, unterliegen möglicherweise gesetzlichen Anforderungen. Beispielsweise können die Daten möglicherweise in keiner Form bereinigt werden. (Sie müssen im ursprünglichen Format aufgezeichnet werden.) Der Zugriff auf das Repository, in dem sie gespeichert sind, muss geschützt sein, um Manipulationen zu verhindern.

### <a name="analyzing-audit-data"></a>Analysieren von Überwachungsdaten
Analysten müssen auf die Rohdaten in ihrer Gesamtheit und ursprünglichen Form zugreifen können. Abgesehen von der Anforderung zum Generieren allgemeiner Überwachungsberichte, sind die Tools zum Analysieren der Daten wahrscheinlich spezialisiert und befinden sich außerhalb des Systems.

## <a name="usage-monitoring"></a>Nutzungsüberwachung
Die Nutzungsüberwachung verfolgt, wie die Features und Komponenten einer Anwendung verwendet werden. Ein Operator kann die erfassten Daten zu folgenden Zwecken verwenden:

* Ermitteln sehr häufig verwendeter Features und potenzieller Hotspots im System. Elemente mit hohem Datenverkehr könnten von funktioneller Partitionierung oder sogar der Replikation profitieren, um die Last gleichmäßig zu verteilen. Ein Operator kann anhand dieser Informationen auch ermitteln, welche Funktionen selten verwendet werden und mögliche Kandidaten für die Stilllegung sind oder in einer zukünftigen Version des Systems ersetzt werden können.
* Erhalten von Informationen über die operationellen Ereignisse im System unter normalen Gebrauch. Bei einer E-Commerce-Website könnten Sie z. B. die statistischen Informationen über die Anzahl der Transaktionen und die Menge an Kunden aufzeichnen, die dafür verantwortlich sind. Diese Informationen könnten zur Kapazitätsplanung bei steigender Kundenzahl verwendet werden.
* Ermitteln der Benutzerzufriedenheit mit der Leistung oder der Funktionalität des Systems (möglicherweise indirekt). Wenn eine große Anzahl von Kunden in einem E-Commerce-System regelmäßig ihre Einkaufswagen verwirft, könnte die Ursache beispielsweise ein Problem mit der Checkout-Funktionalität sein.
* Generieren von Abrechnungsinformationen. Eine kommerzielle Anwendung oder ein Dienst mit mehreren Mandanten kann Kunden die verwendeten Ressourcen in Rechnung stellen.
* Erzwingen von Kontingenten. Wenn ein Benutzer in einem System mit mehreren Mandanten das kostenpflichtige Kontingent an Verarbeitungszeit oder Ressourcenverbrauch innerhalb eines bestimmten Zeitraums überschreitet, kann der Zugriff begrenzt oder die Verarbeitung gedrosselt werden.

### <a name="requirements-for-usage-monitoring"></a>Anforderungen für die Nutzungsüberwachung
Um die Systemnutzung zu untersuchen, muss ein Operator in der Regel Informationen wie diese erhalten:

* Die Anzahl der Anforderungen, die von den einzelnen Subsystemen verarbeitet und an die einzelnen Ressourcen geleitet werden.
* Die Arbeit, die die einzelnen Benutzer ausführen.
* Die Größe des Datenspeichers, der von den einzelnen Benutzern belegt wird.
* Die Ressourcen, auf die die einzelnen Benutzer zugreifen.

Ein Operator sollte auch Diagramme generieren können. Ein Diagramm kann z. B. die Benutzer anzeigen, die die meisten Ressourcen verschlingen, oder die am häufigsten verwendeten Ressourcen oder Systemfunktionen.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Datenquellen, Instrumentierung und Anforderungen für die Datensammlung
Die Nutzungsverfolgung kann auf einer relativ allgemeinen Ebene ausgeführt werden. Dabei können die Anfangs- und die Endzeiten der einzelnen Anforderungen sowie die Art der jeweiligen Anforderung erfasst werden (Lesen, Schreiben usw., abhängig von der betreffenden Ressource). Sie erhalten diese Informationen durch:

* Ablaufverfolgung der Benutzeraktivität.
* Erfassen von Leistungsindikatoren, die die Auslastung für die einzelnen Ressourcen messen.
* Überwachen des Ressourcenverbrauchs durch jeden Benutzer.

Zu Zwecken der Messung müssen Sie auch erkennen können, welcher Benutzer für das Durchführen welcher Vorgänge verantwortlich ist und welche Ressourcen von diesen Vorgänge verwendet werden. Die gesammelten Informationen sollten detailliert genug sein, um eine genaue Abrechnung zu ermöglichen.

<a name="issue-tracking"></a>

## <a name="issue-tracking"></a>Problemverfolgung
Kunden und andere Benutzer könnten bei unerwarteten Ereignissen oder unerwartetem Verhalten im System möglicherweise Probleme melden. Die Problemverfolgung betrifft das Verwalten dieser Probleme, das Zuordnen zu Lösungen für alle zugrunde liegenden Probleme im System und dem Informieren der Kunden über mögliche Lösungen.

### <a name="requirements-for-issue-tracking"></a>Anforderungen für die Problemverfolgung
Operatoren führen die Problemverfolgung häufig anhand eines separaten Systems durch, mit dem sie die Details der von den Benutzern gemeldeten Probleme aufzeichnen und in Berichten erfassen können. Zu diesen Details zählen die Aufgaben, die der Benutzer durchführen wollte, die Symptome des Problems, die Abfolge der Ereignisse und alle ausgegebenen Fehler- oder Warnmeldungen.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Datenquellen, Instrumentierung und Anforderungen für die Datensammlung
Die ursprüngliche Datenquelle für die Problemverfolgung von Daten ist der Benutzer, der das Problem überhaupt gemeldet hat. Der Benutzer kann möglicherweise zusätzliche Daten bereitstellen, beispielsweise:

* Ein Absturzabbild (wenn die Anwendung eine Komponente umfasst, die auf dem Desktop des Benutzers ausgeführt wird).
* Einen Screenshot.
* Datum und Uhrzeit, wann der Fehler aufgetreten ist, zusammen mit anderen Umgebungsinformationen wie z. B. dem Standort des Benutzers.

Diese Informationen können beim Debuggen und beim Erstellen eines Backlogs für zukünftige Versionen der Software helfen.

### <a name="analyzing-issue-tracking-data"></a>Analysieren der Problemverfolgungsdaten
Verschiedene Benutzer melden möglicherweise dasselbe Problem. Das Problemverfolgungssystem sollte gemeinsame Berichte zuordnen.

Der Verlauf der Debugmaßnahmen sollte für jeden Problembericht aufgezeichnet werden. Wenn das Problem gelöst ist, kann der Kunde über die Lösung informiert werden.

Wenn ein Benutzer ein Problem meldet, zu dem es eine bekannte Lösung im Problemverfolgungssystem gibt, sollte der Operator den Benutzer umgehend über die Lösung informieren können.

## <a name="tracing-operations-and-debugging-software-releases"></a>Ablaufverfolgung von Vorgängen und das Debuggen von Software-Versionen.
Wenn ein Benutzer ein Problem meldet, sind ihm häufig nur die unmittelbaren Auswirkungen auf seine Vorgänge bekannt. Der Benutzer kann nur die Ergebnisse der eigenen Erfahrungen an einen Operator melden, der für die Wartung des Systems zuständig ist. Diese Erfahrungen sind in der Regel nur sichtbare Symptome für ein grundlegendes Problem oder mehrere. In vielen Fällen müssen Analysten die Abfolge der zugrunde liegenden Vorgänge prüfen, um die eigentliche Ursache des Problems zu rekonstruieren. Dieser Prozess wird als *Fehlerursachenanalyse*bezeichnet.

> [!NOTE]
> Die Fehlerursachenanalyse kann Ineffizienzen im Design einer Anwendung aufdecken. In diesen Fällen kann es möglich sein, die betroffenen Elemente zu überarbeiten und im Rahmen einer späteren Version bereitzustellen. Dieser Vorgang erfordert eine sorgfältige Steuerung und die aktualisierten Komponenten sollten genau überwacht werden.
> 
> 

### <a name="requirements-for-tracing-and-debugging"></a>Anforderungen für Ablaufverfolgung und Debuggen
Für die Ablaufverfolgung unerwarteter Ereignisse und anderer Probleme ist es wichtig, dass die Überwachungsdaten aussagekräftig sind, damit ein Analyst die Ursprünge dieser Probleme nachverfolgen und die Abfolge der aufgetretenen Ereignisse rekonstruieren kann. Diese Informationen müssen ausreichen, um einem Analysten die Diagnose der Problemursache zu ermöglichen. Ein Entwickler kann dann die erforderlichen Änderungen vornehmen, um ein erneutes Auftreten zu verhindern.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Datenquellen, Instrumentierung und Anforderungen für die Datensammlung
Die Problembehandlung kann die Ablaufverfolgung aller Methoden (und ihrer Parameter) beinhalten, die im Rahmen eines Vorgangs aufgerufen werden. Daraus wird eine Struktur erstellt, die den logischen Fluss des Systems bei Eingang einer bestimmten Kundenanforderung darstellt. Vom System als Ergebnis dieses Flusses generierte Ausnahmen und Warnungen müssen erfasst und protokolliert werden.

Um das Debuggen zu unterstützen, kann das System Hooks bereitstellen, mit denen ein Operator Zustandsinformationen an wichtigen Stellen im System erfassen kann. Alternativ dazu kann das System detaillierte Schritt-für-Schritt-Informationen liefern, während die ausgewählten Vorgänge fortschreiten. Die Erfassung von Daten auf dieser Detailebene kann das System zusätzlich belasten und sollte nur vorübergehend durchgeführt werden. Ein Operator verwendet diesen Prozess in erster Linie, wenn eine besonders ungewöhnliche Serie von Ereignissen auftritt und schwer zu replizieren ist oder wenn eine neue Version eines oder mehrerer Elemente in einem System sorgfältig überwacht werden muss, um sicherzustellen, dass die Elemente wie erwartet funktionieren.

## <a name="the-monitoring-and-diagnostics-pipeline"></a>Überwachungs- und Diagnosepipeline
Die Überwachung eines umfangreichen verteilten Systems stellt eine erhebliche Herausforderung dar. Jedes der im vorherigen Abschnitt erläuterten Szenarios sollte nicht unbedingt isoliert betrachtet werden. Es gibt wahrscheinlich erhebliche Überschneidungen bei der Überwachung und Diagnose von Daten, die für die einzelnen Situationen erforderlich sind, auch wenn diese Daten auf verschiedene Weise verarbeitet und dargestellt werden. Aus diesen Gründen sollten Sie die Überwachung und Diagnose ganzheitlich betrachten.

Sie können sich die gesamte Überwachung und den Diagnoseprozess als eine Pipeline vorstellen, die die in Abbildung 1 dargestellten Phasen umfasst.

![Phasen in der Überwachungs- und Diagnosepipeline](./images/monitoring/Pipeline.png)

*Abbildung 1: Die Phasen in der Überwachungs- und Diagnosepipeline*

Abbildung 1 zeigt, wie die Daten für die Überwachung und Diagnose aus einer Vielzahl von Datenquellen stammen können. In der Instrumentierungs- und der Sammlungsphase werden die Quellen ermittelt, von denen die Daten erfasst werden müssen. Dabei wird festgelegt, welche Daten auf welche Weise erfasst werden und wie diese Daten so formatiert werden, dass sie leicht überprüft werden können. In der Analyse-/Diagnosephase werden anhand der Rohdaten aussagekräftige Informationen generiert, mit denen ein Operator den Systemzustands feststellen kann. Der Operator kann anhand dieser Informationen Entscheidungen über mögliche Maßnahmen treffen und die Ergebnisse in die Instrumentierungs- und die Sammlungsphase zurückführen. Die Visualisierungs-/Warnungsphase bietet eine Nutzungsansicht des Systemzustands. Darin können Informationen anhand einer Reihe von Dashboards nahezu in Echtzeit angezeigt werden. Zudem kann sie Berichte, Graphen und Diagramme generieren, um eine historische Ansicht der Daten zum Ermitteln langfristiger Trends bereitzustellen. Wenn Informationen darauf hinweisen, dass eine KPI wahrscheinlich die zulässigen Begrenzungen überschreitet, kann in dieser Phase außerdem eine Warnung an einen Operator ausgelöst werden. In einigen Fällen kann über eine Warnung auch ein automatisierter Prozess ausgelöst werden, der versucht, Korrekturmaßnahmen wie z. B. die automatische Skalierung durchzuführen.

Beachten Sie, dass diese Schritte den fortlaufenden Prozessablauf bilden, in dem die Phasen parallel erfolgen. Im Idealfall sollten alle Phasen dynamisch konfigurierbar sein. Insbesondere in manchen Fällen, wenn ein System neu bereitgestellt wurde oder Probleme aufgetreten sind, kann das häufigere Sammeln erweiterter Daten erforderlich sein. In anderen Fällen sollte es möglich sein, zur Aufzeichnung grundlegender Informationen zurückzukehren, um zu überprüfen, ob das System ordnungsgemäß funktioniert.

Darüber hinaus sollte der gesamte Überwachungsprozess als dynamische, fortlaufende Lösung angesehen werden, die der Feinabstimmung und Verbesserungen durch Feedback unterliegt. Sie könnten z. B. mit der Messung zahlreicher Faktoren beginnen, um die Systemintegrität zu ermitteln. Eine langfristige Analyse führt dann zu einer feineren Abstimmung, in deren Verlauf irrelevante Messungen verworfen werden, sodass Sie sich genauer auf die wirklich benötigten Daten konzentrieren können, während unnötige Daten auf ein Mindestmaß reduziert werden.

## <a name="sources-of-monitoring-and-diagnostic-data"></a>Datenquellen der Überwachung und Diagnose
Die vom Überwachungsprozess verwendeten Informationen können aus verschiedenen Quellen stammen, wie in Abbildung 1 dargestellt. Auf der Anwendungsebene stammen Informationen aus Protokollen der Ablaufverfolgung, die im Code des Systems integriert sind. Entwickler sollten bei der Nachverfolgung des Steuerungsflusses durch ihren Code einen standardisierten Ansatz befolgen. Beispielsweise kann beim Einstieg in eine Methode eine Ablaufverfolgungsmeldung ausgegeben werden, die den Namen der Methode, die aktuelle Uhrzeit, den Wert der einzelnen Parameter und andere relevante Informationen angibt. Das Aufzeichnen der Einstiegs- und Ausgangszeiten könnten sich ebenso als nützlich erweisen.

Sie sollten alle Ausnahmen und Warnungen protokollieren und sicherstellen, dass Sie eine vollständige Ablaufverfolgung von geschachtelten Ausnahmen und Warnungen beibehalten. Im Idealfall sollten Sie auch Informationen zum Identifizieren des Benutzers erfassen, der den Code ausführt, sowie Informationen zur Aktivitätskorrelation (zum Nachverfolgen von Anforderungen, während sie das System passieren). Ferner sollten Sie alle Versuche zum Zugreifen auf Ressourcen wie Nachrichtenwarteschlangen, Datenbanken, Dateien und andere abhängige Dienste protokollieren. Diese Informationen können zu Mess- und Überwachungszwecken verwendet werden.

Viele Anwendungen verwenden Bibliotheken und Frameworks für allgemeine Aufgaben wie den Zugriff auf einen Datenspeicher oder die Kommunikation über ein Netzwerk. Diese Frameworks können möglicherweise so konfiguriert werden, dass sie eigene Meldungen zur Ablaufverfolgung und unformatierte Diagnoseinformationen ausgeben, z. B. Transaktionsraten oder Datenübertragungserfolge und -fehler.

> [!NOTE]
> Viele moderne Frameworks veröffentlichen automatisch Leistung und Ablaufverfolgungsereignisse. Zum Erfassen dieser Informationen muss einfach eine Möglichkeit bereitgestellt werden, sie abzurufen und an einer Stelle zu speichern, an der sie verarbeitet und analysiert werden können.
> 
> 

Das Betriebssystem, auf dem die Anwendung ausgeführt wird, kann als Quelle detaillierter, systemweiter Informationen dienen, z. B. Leistungsindikatoren, die E/A-Raten, Speicherauslastung und CPU-Auslastung angeben. Betriebssystemfehler (z. B. Fehler beim Öffnen einer Datei) können ebenfalls gemeldet werden.

Sie sollten auch die zugrunde liegende Infrastruktur und Komponenten berücksichtigen, mit denen das System ausgeführt wird. Virtuelle Computer, virtuelle Netzwerke und Speicherdienste können alle Quellen für wichtige Leistungsindikatoren auf Infrastrukturebene und andere Diagnosedaten sein.

Wenn die Anwendung andere externe Dienste verwendet, wie z. B. einen Webserver oder ein Datenverwaltungssystem, veröffentlichen diese Dienste möglicherweise ihre eigenen Ablaufverfolgungsinformationen, Protokolle und Leistungsindikatoren. Beispiele sind SQL Server Dynamic Management Views zum Nachverfolgen von Vorgängen in einer SQL Server-Datenbank sowie IIS-Ablaufverfolgungsprotokolle zum Aufzeichnen von Anforderungen an einen Webserver.

Wenn die Komponenten eines Systems geändert und neue Versionen bereitgestellt werden, ist es wichtig, Probleme, Ereignisse und Metriken den einzelnen Versionen zuordnen zu können. Diese Informationen sollten an die Versionspipeline gebunden sein, damit Probleme mit einer bestimmten Version einer Komponente schnell nachverfolgt und behoben werden können.

Sicherheitsprobleme können an jedem Punkt im System auftreten. Beispielsweise kann ein Benutzer versuchen, sich mit einer ungültigen Benutzer-ID oder einem ungültigen Kennwort anzumelden. Ein authentifizierter Benutzer kann versuchen, nicht autorisierten Zugriff auf eine Ressource zu erhalten, oder ein Benutzer stellt möglicherweise einen ungültigen oder veralteten Schlüssel für den Zugriff auf verschlüsselte Informationen bereit. Sicherheitsinformationen für erfolgreiche und fehlgeschlagene Anforderungen sollten immer protokolliert werden.

Der Abschnitt [Instrumentieren einer Anwendung](#instrumenting-an-application) enthält weitere Anleitungen zu den Informationen, die Sie erfassen sollten. Hierzu können Sie jedoch eine Vielzahl von Strategien einsetzen:

* **Anwendungs-/Systemüberwachung**. Diese Strategie nutzt interne Quellen innerhalb der Anwendung, der Anwendungsframeworks, des Betriebssystems und der Infrastruktur. Der Anwendungscode kann an wesentlichen Punkten im Lebenszyklus einer Clientanforderung eigene Überwachungsdaten generieren. Die Anwendung kann Anweisungen für die Ablaufverfolgung enthalten, die je nach Umständen selektiv aktiviert oder deaktiviert werden können. Es kann auch möglich sein, die Diagnose mithilfe eines Diagnoseframeworks dynamisch einzufügen. Diese Frameworks stellen in der Regel Plug-Ins bereit, die sich an verschiedenen Punkten der Instrumentierung in Ihrem Code anfügen und Ablaufverfolgungsdaten an diesen Punkten erfassen können.
  
    Darüber hinaus kann Ihr Code und/oder die zugrunde liegende Infrastruktur an kritischen Punkten Ereignisse auslösen. Überwachungsagenten, die für die Überwachung dieser Ereignisse konfiguriert sind, können die Ereignisinformationen erfassen.
* **Überwachung realer Benutzer**. Dieser Ansatz zeichnet die Interaktionen zwischen einem Benutzer und der Anwendung auf und beobachtet den Fluss der einzelnen Anforderungen und Antworten. Diese Informationen können zwei Zwecken dienen: Sie messen die Nutzung durch die einzelnen Benutzer, und sie zeigen, ob Benutzer die angemessene Servicequalität erhalten (z. B. schnelle Antwortzeiten, niedrige Latenz und eine minimale Fehlerquote). Anhand der erfassten Daten können Sie Problembereiche identifizieren, in denen Fehler am häufigsten auftreten. Sie können mithilfe der Daten auch Elemente ermitteln, die das System verlangsamen, möglicherweise aufgrund von Hotspots in der Anwendung oder einer anderen Form von Engpass. Wenn dieser Ansatz sorgfältig implementiert wird, ist es eventuell möglich, Benutzerabläufe in der Anwendung zu Debug- und Testzwecken zu rekonstruieren.
  
  > [!IMPORTANT]
  > Sie sollten die Daten, die durch die Überwachung realer Benutzer erfasst werden, als hochsensibel behandeln, da sie möglicherweise vertrauliches Material umfassen. Wenn Sie die erfassten Daten speichern, sorgen Sie für eine sichere Aufbewahrung. Wenn Sie die Daten zur Leistungsüberwachung und zum Debuggen verwenden möchten, löschen Sie zuerst alle personenbezogenen Informationen.
  > 
  > 
* **Überwachung synthetischer Benutzer**. Bei dieser Vorgehensweise schreiben Sie Ihren eigenen Testclient, der einen Benutzer simuliert und eine konfigurierbare, aber typische Reihe von Vorgängen ausführt. Sie können die Leistung des Testclients verfolgen, um den Zustand des Systems zu ermitteln. Sie können auch mehrere Instanzen des Testclients als Teil eines Belastungstests einrichten, um festzustellen, wie das System unter Stress reagiert und welche Art von Überwachungsausgabe unter diesen Bedingungen generiert wird.
  
  > [!NOTE]
  > Sie können die tatsächliche und synthetische Benutzerüberwachung durch Einschließen von Code implementieren, der die Ausführung von Methodenaufrufen und anderen kritischen Teilen einer Anwendung verfolgt und zeitlich einordnet.
  > 
  > 
* **Profilerstellung**. Dieser Ansatz zielt in erster Linie auf das Überwachen und Verbessern der Anwendungsleistung ab. Statt auf der Funktionsebene der Überwachung realer und synthetischer Benutzer anzusetzen, erfasst er Informationen auf niedrigerer Ebene, während die Anwendung ausgeführt wird. Sie können die Profilerstellung anhand regelmäßiger Stichproben zum Ausführungszustand einer Anwendung implementieren (und dabei ermitteln, welcher Teil des Codes von der Anwendung zu einem bestimmten Zeitpunkt ausgeführt wird). Sie können auch eine Instrumentierung verwenden, die Tests an wichtigen Stellen in den Code einfügt (z. B. den Start und das Ende eines Methodenaufrufs) und aufzeichnet, welche Methoden zu welchem Zeitpunkt aufgerufen wurden und wie lange die einzelnen Aufrufe gedauert haben. Diese Daten können Sie dann analysieren, um zu bestimmen, welche Teile der Anwendung zu Leistungsproblemen führen können.
* **Endpunktüberwachung**. Diese Technik verwendet einen oder mehrere diagnostische Endpunkte, die von der Anwendung speziell für die Überwachung verfügbar gemacht werden. Ein Endpunkt stellt einen Pfad in den Anwendungscode bereit und kann Informationen über den Zustand des Systems zurückgeben. Verschiedene Endpunkte können sich auf verschiedene Aspekte der Funktionalität konzentrieren. Sie können Ihren eigenen Diagnose-Client schreiben, der periodisch Anforderungen an diese Endpunkte sendet, und die Antworten anpassen. Weitere Informationen finden Sie unter [Überwachungsmuster für den Integritätsendpunkt](../patterns/health-endpoint-monitoring.md).

Zur maximalen Absicherung sollten Sie eine Kombination dieser Techniken verwenden.

<a name="instrumenting-an-application"></a>

## <a name="instrumenting-an-application"></a>Instrumentieren einer Anwendung
Die Instrumentierung ist ein wichtiger Bestandteil des Überwachungsprozesses. Sie können nur dann fundierte Entscheidungen für die Leistung und Integrität des Systems treffen, wenn Sie zuerst die Daten erfassen, die diese Entscheidungen ermöglichen. Die Informationen, die Sie mithilfe der Instrumentierung sammeln, sollten zum Bewerten der Leistung, zum Diagnostizieren von Problemen und zum Treffen von Entscheidungen ausreichen, ohne dass Sie sich bei einem Remoteproduktionsserver anmelden und die Ablaufverfolgung (und das Debuggen) manuell ausführen müssen. Instrumentierungsdaten umfassen normalerweise Metriken und Informationen, die in Ablaufverfolgungsprotokolle geschrieben werden:

Der Inhalt eines Ablaufverfolgungsprotokolls können Textdaten sein, die von der Anwendung geschrieben werden, oder binäre Daten, die als Ergebnis eines Ablaufverfolgungsereignisses erstellt werden (wenn die Anwendung die Ereignisablaufverfolgung für Windows, ETW, verwendet). Der Inhalt kann auch aus Systemprotokollen generiert werden, die Ereignisse aufzeichnen, die durch Teile der Infrastruktur, z. B. einen Webserver, ausgelöst werden. Protokollmeldungen sind oft auf Lesbarkeit für Personen ausgelegt, sollten aber auch in einem Format geschrieben sein, das eine einfache Analyse durch ein automatisiertes System ermöglicht.

Protokolle sollten zudem kategorisiert werden. Schreiben Sie nicht alle Ablaufverfolgungsdaten in ein einzelnes Protokoll, sondern verwenden Sie separate Protokolle zum Aufzeichnen der Ablaufverfolgungsausgabe von verschiedenen betrieblichen Aspekten des Systems. Dadurch können Sie Protokollmeldungen schnell filtern, indem Sie das entsprechende Protokoll lesen, statt eine einzelne lange Datei verarbeiten zu müssen. Schreiben Sie nie Informationen, die unterschiedlichen Sicherheitsanforderungen unterliegen, in dasselbe Protokoll (z. B. Überwachungsinformationen und Debugging-Daten).

> [!NOTE]
> Ein Protokoll kann als Datei im Dateisystem implementiert oder in einem anderen Format wie einem Blob im Blobspeicher aufbewahrt werden. Protokollinformationen können auch in strukturierteren Speichern gehalten werden, z. B. Zeilen in einer Tabelle.
> 
> 

Metriken sind generell ein Maß oder eine Anzahl eines Aspekts oder einer Ressource im System zu einem bestimmten Zeitpunkt, mit einem oder mehren zugeordneten Tags oder Dimensionen (manchmal auch als *Stichprobe* bezeichnet). Eine einzelne Instanz einer Metrik ist für sich betrachtet in der Regel nicht nützlich. Stattdessen müssen Metriken über einen Zeitraum erfasst werden. Die wichtigste zu berücksichtigende Frage ist, welche Metriken erfasst werden sollten und wie häufig. Die zu häufige Datengenerierung für Metriken kann oft eine erhebliche zusätzliche Last für das System bedeuten, während zu seltenes Erfassen der Metriken dazu führen kann, dass Sie die Umstände verpassen, die zu einem wichtigen Ereignis führen. Die Überlegungen variieren von Metrik zu Metrik. Z. B. kann die CPU-Auslastung auf einem Server von Sekunde zu Sekunde erheblich variieren, aber eine hohe Auslastung wird nur dann zu einem Problem, wenn sie über mehrere Minuten andauert.

<a name="information-for-correlating-data"></a>

### <a name="information-for-correlating-data"></a>Informationen zum Korrelieren von Daten
Sie können einfach einzelne Leistungsindikatoren auf Systemebene überwachen, Metriken für Ressourcen erfassen und Ablaufverfolgungsinformationen für die Anwendung aus verschiedenen Protokolldateien erhalten. Für einige Formen der Überwachung ist jedoch die Analyse- und Diagnosephase in der Überwachungspipeline erforderlich, um zwischen den aus verschiedenen Quellen abgerufenen Daten einen Zusammenhang herzustellen. Diese Daten können in den Rohdaten verschiedene Formen annehmen, und dem Analyseprozess müssen ausreichende Instrumentierungsdaten bereitgestellt werden, um die verschiedenen Formen abzubilden. Beispielsweise kann auf der Ebene des Anwendungsframeworks eine Aufgabe durch eine Thread-ID identifiziert werden. Innerhalb einer Anwendung kann die gleiche Arbeit der Benutzer-ID des Benutzer zugeordnet sein, der diese Aufgabe ausführt.

Darüber hinaus gibt es wahrscheinlich keine 1:1-Zuordnung zwischen Threads und Benutzeranforderungen, da asynchrone Vorgänge dieselben Threads wiederverwenden, um Vorgänge für mehrere Benutzer durchzuführen. Um die Angelegenheit noch komplizierter zu machen, kann eine einzelne Anforderung von mehreren Threads verarbeitet werden, während die Ausführung das System durchläuft. Ordnen Sie nach Möglichkeit jeder Anforderung eine eindeutige Aktivitäts-ID zu, die als Teil des Anforderungskontexts durch das System weitergegeben wird. (Das Verfahren zum Generieren und Einschließen von Aktivitäts-IDs in Ablaufverfolgungsinformationen hängt von der Technologie ab, die zum Erfassen der Ablaufverfolgungsdaten verwendet wird.)

Alle Überwachungsdaten sollten auf gleiche Weise mit einem Zeitstempel versehen werden. Notieren Sie aus Gründen der Konsistenz alle Datums- und Uhrzeitangaben mithilfe der Coordinated Universal Time. So können Sie Sequenzen von Ereignissen leichter nachverfolgen.

> [!NOTE]
> Computer in verschiedenen Zeitzonen und Netzwerken werden vielleicht nicht synchronisiert. Verlassen Sie sich nicht nur auf Zeitstempel, wenn Sie Instrumentierungsdaten korrelieren, die mehrere Computer umfassen.
> 
> 

### <a name="information-to-include-in-the-instrumentation-data"></a>In die Instrumentierungsdaten einzubeziehende Informationen
Beachten Sie die folgenden Punkte, wenn Sie entscheiden, welche Instrumentierungsdaten gesammelt werden sollen:

* Stellen Sie sicher, dass Informationen, die von Ablaufverfolgungsereignissen erfasst werden, für Computer und Personen lesbar sind. Implementieren Sie umfassend definierte Schemas für diese Informationen, um die automatisierte Verarbeitung von Protokolldaten über Systeme hinweg zu vereinfachen und Konsistenz für Vorgänge und technische Mitarbeiter sicherzustellen, die die Protokolle lesen. Schließen Sie die Umgebungsinformationen mit ein, wie z. B. die Bereitstellungsumgebung, den Computer, auf dem der Prozess ausgeführt wird, die Details des Prozesses und die Aufrufliste.  
* Aktivieren Sie die Profilerstellung nur bei Bedarf, da sie eine erhebliche Belastung für das System darstellen kann. Bei der Profilerstellung mithilfe der Instrumentierung wird ein Ereignis (z. B. ein Methodenaufruf) jedes Mal aufgezeichnet, wenn es auftritt. Bei Stichproben werden hingegen nur ausgewählte Ereignisse aufgezeichnet. Die Auswahl kann nach Zeit (alle *n* Sekunden) oder nach Häufigkeit (alle *n* Anforderungen) erfolgen. Wenn Ereignisse sehr häufig auftreten, kann die Profilerstellung mit der Instrumentierungsmethode eine zu große Belastung darstellen und die Gesamtleistung beeinträchtigen. In diesem Fall ist der Stichprobenansatz vorzuziehen. Wenn Ereignisse allerdings selten auftreten, werden sie bei der Stichprobenentnahme möglicherweise verpasst. In diesem Fall ist die Instrumentierung möglicherweise der bessere Ansatz.
* Geben Sie genügend Kontext an, damit ein Entwickler oder Administrator die Quelle jeder einzelnen Anforderung ermitteln kann. Dazu gehört zum Beispiel eine Form von Aktivitäts-ID, die eine bestimmte Instanz einer Anforderung identifiziert. Dazu gehören beispielsweise auch Informationen, mit deren Hilfe diese Aktivität mit der durchgeführten Computerarbeit und den verwendeten Ressourcen in Bezug gesetzt werden kann. Beachten Sie, dass diese Arbeit Prozess- und Computergrenzen überschreiten kann. Für die Messung sollte der Kontext (entweder direkt oder indirekt über andere korrelierte Informationen) auch einen Verweis auf den Kunden beinhalten, der die Anforderung veranlasst hat. Dieser Kontext stellt wertvolle Informationen über den Anwendungszustand zum Zeitpunkt der Aufzeichnung der Überwachungsdaten bereit.
* Notieren Sie alle Anforderungen und die Standorte oder Regionen, aus denen solche Anforderungen gesendet werden. Anhand dieser Informationen lässt sich feststellen, ob standortspezifische Hotspots vorliegen. Außerdem kann mithilfe dieser Informationen ermittelt werden, ob eine Anwendung oder die von ihr verwendeten Daten neu partitioniert werden sollten.
* Die Details der Ausnahmen sollten Sie sorgfältig aufzeichnen und erfassen. Oftmals gehen sehr wichtige Debuginformationen aufgrund einer mangelhaften Ausnahmebehandlung verloren. Erfassen Sie alle Details zu Ausnahmen, die von der Anwendung ausgelöst werden, einschließlich aller inneren Ausnahmen und anderer Kontextinformationen. Beziehen Sie nach Möglichkeit die Aufrufliste ein.
* Sorgen Sie für Konsistenz bei den Daten, die von den verschiedenen Elementen Ihrer Anwendung erfasst werden, da dies bei der Analyse von Ereignissen und bei deren Korrelation zu Benutzeranfragen hilfreich sein kann. Erwägen Sie die Nutzung eines umfassenden und konfigurierbaren Protokollierungspakets, um Informationen zu sammeln, anstatt sich darauf zu verlassen, dass Entwickler die verschiedenen Teile des Systems nach dem gleichen Ansatz implementieren. Das Sammeln von Daten zu wesentlichen Leistungsindikatoren, wie z. B. ausgeführtes E/A-Volumen, Netzwerkauslastung, Anzahl der Anforderungen, Speichernutzung und CPU-Auslastung. Einige Infrastrukturdienste stellen ihre eigenen Leistungsindikatoren bereit, wie z. B. die Anzahl der Verbindungen mit einer Datenbank, die Rate, mit der Transaktionen ausgeführt werden, und die Anzahl der erfolgreichen oder fehlerhaften Transaktionen. Anwendungen können auch ihre eigenen Leistungsindikatoren definieren.
* Protokollieren Sie alle Aufrufe an externe Dienste wie Datenbanksysteme, Webdienste oder andere Dienste auf Systemebene, die Teil der Infrastruktur sind. Zeichnen Sie Informationen über die Verarbeitungszeit für jeden Aufruf auf, und ob der Aufruf erfolgreich war oder fehlgeschlagen ist. Wenn möglich, erfassen Sie Informationen über alle Wiederholungsversuche und Fehlversuche für vorübergehende Fehler, die auftreten.

### <a name="ensuring-compatibility-with-telemetry-systems"></a>Sicherstellen der Kompatibilität mit Telemetrie-Systemen
In vielen Fällen werden mit der Instrumentierung produzierte Informationen als eine Reihe von Ereignissen generiert und zur Verarbeitung und Analyse an ein separates Telemetriesystem übergeben. Ein Telemetriesystem ist in der Regel unabhängig von einer bestimmten Anwendung oder Technologie, erwartet Informationen jedoch in einem bestimmten Format, das in der Regel durch ein Schema definiert ist. Das Schema gibt gewissermaßen einen Vertrag an, in dem die Datenfelder und Typen definiert sind, die das Telemetriesystem erfassen kann. Das Schema sollte generalisiert sein, damit Daten aus einer Reihe von Plattformen und Geräten stammen können.

Ein allgemeines Schema sollte Felder enthalten, die auf alle Instrumentierungsereignisse zutreffen, wie z. B. den Namen und die Uhrzeit des Ereignisses, die IP-Adresse des Absenders und die für die Korrelation mit anderen Ereignissen erforderlichen Details (z. B. eine Benutzer-ID, eine Geräte-ID und eine Anwendungs-ID). Denken Sie daran, dass Ereignisse von einer beliebigen Anzahl von Geräten ausgelöst werden können, also sollte das Schema nicht vom Gerätetyp abhängig sein. Darüber hinaus können Ereignisse für dieselbe Anwendung durch verschiedene Geräte ausgelöst werden. Die Anwendung unterstützt womöglich Roaming oder eine andere Form der geräteübergreifenden Verteilung.

Das Schema kann auch Domänenfelder enthalten, die für ein bestimmtes Szenario relevant sind, das für verschiedene Anwendungen gilt. Dabei kann es sich um Informationen zu Ausnahmen, zu Ereignissen bei Anwendungsstart und -ende sowie zu erfolgreichen und/oder fehlerhaften Web Services-API-Aufrufen handeln. Alle Anwendungen, die den gleichen Satz von Feldern für die Domäne verwenden, sollten den gleichen Satz von Ereignissen ausgeben, was einen Satz gemeinsamer Berichte und Analysen ermöglicht, die zu erstellen sind.

Zudem kann ein Schema benutzerdefinierte Felder zum Erfassen der Details von anwendungsspezifischen Ereignissen enthalten.

### <a name="best-practices-for-instrumenting-applications"></a>Empfehlungen zum Instrumentieren von Anwendungen
Die folgende Liste enthält Empfehlungen zum Instrumentieren einer verteilten Anwendung, die in der Cloud ausgeführt wird.

* Erstellen Sie Protokolle so, dass sie einfach zu lesen und leicht zu analysieren sind. Verwenden Sie die strukturierte Protokollierung, wo möglich. Seien Sie in Protokollmeldungen prägnant und anschaulich.
* Identifizieren Sie in allen Protokollen die Quelle und stellen Sie Kontext- und Zeitinformationen bereit, während jeder Protokolldatensatz geschrieben wird.
* Verwenden Sie die gleiche Zeitzone und das gleiche Format für alle Zeitstempel. Dadurch können Sie Ereignisse für Vorgänge korrelieren, die Hardware und Dienste in unterschiedlichen geografischen Regionen umfassen.
* Kategorisieren Sie Protokolle und schreiben Sie Meldungen in die entsprechende Protokolldatei.
* Legen Sie keine vertraulichen Informationen zum System oder persönliche Informationen über Benutzer offen. Bereinigen Sie diese Informationen, bevor sie protokolliert werden, aber stellen Sie sicher, dass alle relevanten Details erhalten bleiben. Entfernen Sie z. B. die ID und das Kennwort von allen Datenbank-Verbindungszeichenfolgen, aber schreiben Sie die restlichen Daten in das Protokoll, sodass ein Analyst bestimmen kann, ob das System auf die richtige Datenbank zugreift. Protokollieren Sie alle kritische Ausnahmen, aber ermöglichen Sie es dem Administrator, Ausnahmen und Warnungen der unteren Ebene zu aktivieren und deaktivieren. Erfassen und protokollieren Sie darüber hinaus alle logischen Wiederholungsinformationen. Diese Daten können bei der Überwachung des vorübergehenden Systemzustands nützlich sein.
* Verfolgen Sie Aufrufe außerhalb des Prozesses, z. B. Anforderungen an externe Webdienste oder Datenbanken.
* Vermischen Sie keine Protokollmeldungen mit verschiedenen Sicherheitsanforderungen in derselben Protokolldatei. Schreiben Sie z. B. keine Informationen zum Debuggen und zum Überwachen in das gleiche Protokoll.
* Mit Ausnahme von Überwachungsereignissen müssen alle Aufrufe der Protokollierung „Fire-and-Forget“-Vorgänge sein, die den Geschäftsbetrieb nicht blockieren dürfen. Überwachungsereignisse sind außergewöhnlich, da sie für das Unternehmen wichtig sind und als grundlegender Bestandteil der Geschäftsvorgänge klassifiziert werden können.
* Stellen Sie sicher, dass die Protokollierung erweiterbar und nicht direkt von einem konkreten Ziel abhängig ist. Anstatt z. B. Daten mithilfe von *System.Diagnostics.Trace* zu schreiben, definieren Sie eine abstrakte Schnittstelle (z. B. *ILogger*), die Protokollierungsmethoden verfügbar macht und mithilfe angemessener Maßnahmen implementiert werden kann.
* Stellen Sie sicher, dass die gesamte Protokollierung ausfallsicher ist und niemals kaskadierende Fehler auslöst. Protokollierungen dürfen keine Ausnahmen auslösen.
* Behandeln Sie die Instrumentierung als fortlaufenden, iterativen Prozess und überprüfen Sie Protokolle regelmäßig, nicht nur, wenn ein Problem vorliegt.

## <a name="collecting-and-storing-data"></a>Erfassen und Speichern von Daten
In der Sammlungsphase des Überwachungsprozesses werden die durch die Instrumentierung generierten Informationen abgerufen, so formatiert, dass sie in der Analyse-/Diagnosephase leichter zu nutzen sind, und als transformierte Daten in einem zuverlässigen Speicher abgelegt. Die Instrumentierungsdaten, die Sie aus verschiedenen Teilen eines verteilten Systems erfassen, können in einer Vielzahl von Speicherorten und in unterschiedlichen Formaten gehalten werden. Ihr Anwendungscode kann z. B. Ablaufverfolgungsprotokolle und Ereignisprotokolldaten für die Anwendung generieren, während die Leistungsindikatoren, die die wichtigsten Aspekte der von der Anwendung genutzten Infrastruktur überwachen, mithilfe anderer Technologien erfasst werden können. Alle von Ihrer Anwendung verwendeten Drittanbieterkomponenten und -dienste können Instrumentierungsinformationen in unterschiedlichen Formaten bereitstellen, indem sie separate Ablaufverfolgungsdateien, Blobspeicher oder sogar einen benutzerdefinierten Datenspeicher verwenden.

Die Datensammlung erfolgt häufig über einen Sammlungsdienst, der unabhängig von der Anwendung ausgeführt werden kann, die die Instrumentierungsdaten generiert. Abbildung 2 zeigt ein Beispiel für diese Architektur, in dem das Subsystem der Instrumentierungsdatensammlung hervorgehoben ist.

![Beispiel für das Erfassen von Instrumentierungsdaten](./images/monitoring/TelemetryService.png)

*Abbildung 2: Erfassen von Instrumentierungsdaten*

Bitte beachten Sie, dass dies eine vereinfachte Ansicht ist. Der Sammlungsdienst ist nicht unbedingt ein einzelner Prozess und umfasst möglicherweise viele Bestandteile, die auf verschiedenen Computern ausgeführt werden, wie in den folgenden Abschnitten beschrieben. Wenn die Analyse einiger Telemetriedaten darüber hinaus schnell ausgeführt werden muss (heiße Analyse, wie im Abschnitt [Unterstützung von heißer, warmer und kalter Analyse](#supporting-hot-warm-and-cold-analysis) weiter unten in diesem Dokument beschrieben), können lokale Komponenten außerhalb des Erfassungsdienstes die Analyseaufgaben sofort durchführen. Abbildung 2 zeigt diese Situation für ausgewählte Ereignisse. Nach der analytischen Verarbeitung können die Ergebnisse direkt an das Visualisierungs- und Warnsubsystem gesendet werden. Daten zur warmen oder kalten Analyse werden im Speicher gehalten, während sie auf die Verarbeitung warten.

Für Azure-Anwendungen und -Dienste bietet die Azure-Diagnose eine mögliche Lösung für das Erfassen von Daten. Die Azure-Diagnose sammelt Daten aus den folgenden Quellen für jeden Computeknoten, verdichtet sie und lädt sie dann in Azure Storage hoch:

* IIS-Protokolle
* Protokolle zu IIS-Anforderungsfehlern
* Windows-Ereignisprotokolle
* Leistungsindikatoren
* Absturzabbilder
* Infrastrukturprotokolle der Azure-Diagnose  
* Benutzerdefinierte Fehlerprotokolle
* .NET-EventSource
* Manifestbasierte ETW

Weitere Informationen finden Sie im Artikel [Azure: Telemetry Basics and Troubleshooting](http://social.technet.microsoft.com/wiki/contents/articles/18146.windows-azure-telemetry-basics-and-troubleshooting.aspx)(Azure: Telemetrie-Grundlagen und Problembehandlung).

### <a name="strategies-for-collecting-instrumentation-data"></a>Strategien zum Erfassen von Instrumentierungsdaten
Angesichts der Elastizität der Cloud und um zu vermeiden, dass Telemetriedaten manuell von jedem Knoten im System abgerufen werden müssen, sollten Sie dafür sorgen, dass Daten an einen zentralen Standort übertragen und dort konsolidiert werden können. In einem System, das mehrere Rechenzentren umfasst, kann es hilfreich sein, Daten zunächst zu sammeln, zu konsolidieren und je nach Region zu speichern und dann die regionalen Daten in einem einzigen zentralen System zu aggregieren.

Um die Verwendung der Bandbreite zu optimieren, können Sie festlegen, dass weniger dringende Daten in Segmenten als Batches übertragen werden. Jedoch dürfen die Daten nicht unbegrenzt verzögert werden, insbesondere, wenn sie zeitgebundene Informationen beinhalten.

#### <a name="pulling-and-pushing-instrumentation-data"></a>*Pullen und Pushen von Instrumentierungsdaten*
Das Subsystem für die Instrumentierungsdatensammlung kann Instrumentierungsdaten aktiv aus den verschiedenen Protokollen und anderen Quellen für die einzelnen Instanzen der Anwendung abrufen (das *Pullmodell*). Alternativ dazu kann es als passiver Empfänger fungieren, der wartet, bis die Daten von den Komponenten gesendet werden, aus denen sich die einzelnen Instanzen der Anwendung zusammensetzen (das *Pushmodell*).

Ein Ansatz zum Implementieren des Pullmodells ist die Verwendung von Überwachungs-Agents, die lokal auf den einzelnen Instanzen der Anwendung ausgeführt werden. Ein Überwachungs-Agent ist ein separater Prozess, der per Pull in regelmäßigen Abständen Telemetriedaten abruft, die vom lokalen Knoten gesammelt werden, und diese Informationen direkt in den zentralen Speicher schreibt, der von allen Instanzen der Anwendung gemeinsam verwendet wird. Dies ist der Mechanismus, der in der Azure-Diagnose implementiert wird. Jede Instanz einer Web- oder Workerrolle in Azure kann so konfiguriert werden, dass Diagnose- und andere lokal gespeicherte Ablaufverfolgungsinformationen erfasst werden. Der Überwachungs-Agent, der neben den einzelnen Instanzen ausgeführt wird, kopiert die angegebenen Daten in Azure Storage. Im Artikel [Aktivieren der Diagnose in Azure Cloud Services und Virtual Machines](/azure/cloud-services/cloud-services-dotnet-diagnostics) finden Sie nähere Einzelheiten. Einige Elemente, wie z. B. IIS-Protokolle, Absturzabbilder und benutzerdefinierte Fehlerprotokolle, werden in Blobspeicher geschrieben. Daten aus dem Windows-Ereignisprotokoll, ETW-Ereignisse und Leistungsindikatoren werden im Tabellenspeicher aufgezeichnet. Dieser Mechanismus ist in Abbildung 3 dargestellt.

![Abbildung zur Verwendung eines Überwachungs-Agents zum Abrufen von Informationen und zum Schreiben in freigegebenen Speicher](./images/monitoring/PullModel.png)

*Abbildung 3: Verwenden eines Überwachungs-Agents zum Abrufen von Informationen und Schreiben in einen freigegebenen Speicher*

> [!NOTE]
> Die Verwendung eines Überwachungs-Agents eignet sich hervorragend zum Erfassen von Instrumentierungsdaten, die naturgemäß per Pull aus einer Datenquelle übertragen werden. Ein Beispiel sind Informationen von SQL Server Dynamic Management Views oder die Länge einer Azure Service Bus-Warteschlange.
> 
> 

Es ist möglich, den soeben beschriebenen Ansatz zum Speichern von Telemetriedaten für eine kleine Anwendung einzusetzen, die auf einer begrenzten Anzahl von Knoten an einem einzigen Standort ausgeführt wird. Eine komplexe, hochgradig skalierbare, globale Cloudanwendung kann jedoch riesige Mengen von Daten aus Hunderten von Web- und Workerrollen, Datenbankshards und anderen Diensten generieren. Diese Datenflut kann leicht die an einem einzelnen zentralen Standort verfügbare E/A-Bandbreite überlasten. Daher muss Ihre Telemetrielösung skalierbar sein, damit sie beim Erweitern des Systems nicht zum Engpass wird. Ihre Lösung sollte im Idealfall ein gewisses Maß an Redundanz bieten, um das Risiko zu reduzieren, dass wichtige Überwachungsinformationen (z. B. Audit- oder Rechnungsdaten) bei einem teilweisen Systemausfall verloren gehen.

Um diese Aspekte abzudecken, können Sie Warteschlangen implementieren, wie in Abbildung 4 dargestellt. In dieser Architektur stellt der lokale Überwachungs-Agent (sofern er entsprechend konfiguriert werden kann) oder (andernfalls) der benutzerdefinierte Datensammlungsdienst Daten in einer Warteschlange bereit. Ein separater, asynchron ausgeführter Prozess (der Speicherschreibdienst in Abbildung 4) verwendet die Daten aus dieser Warteschlange und schreibt sie in freigegebenen Speicher. Eine Nachrichtenwarteschlange ist für dieses Szenario geeignet, da sie eine „Mindestens-einmal“-Semantik umfasst, um sicherzustellen, dass in die Warteschlange eingereihte Daten nach dem Bereitstellen nicht verloren gehen. Sie können den Speicherschreibdienst anhand einer separaten Workerrolle implementieren.

![Abbildung zur Verwendung einer Warteschlange zum Puffern von Instrumentierungsdaten](./images/monitoring/BufferedQueue.png)

*Abbildung 4: Mithilfe einer Warteschlange Instrumentierungsdaten puffern*

Der lokale Datensammlungsdienst kann Daten unmittelbar nach deren Empfang einer Warteschlange hinzufügen. Die Warteschlange fungiert als Puffer, und der Speicherschreibdienst kann die Daten in seinem eigenen Tempo abrufen und schreiben. Standardmäßig arbeitet eine Warteschlange nach dem Prinzip „First In, First Out“. Sie können Nachrichten jedoch priorisieren, um ihren Weg durch die Warteschlange zu beschleunigen, wenn sie Daten enthalten, die schneller verarbeitet werden müssen. Weitere Informationen finden Sie unter [Prioritätswarteschlange](https://msdn.microsoft.com/library/dn589794.aspx). Alternativ dazu können Sie, abhängig von der Form der erforderlichen analytischen Verarbeitung, verschiedene Kanäle (z. B. Service Bus-Topics) verwenden, um Daten an verschiedene Ziele zu leiten.

Aus Gründen der Skalierbarkeit könnten Sie mehrere Instanzen des Speicherschreibdienstes ausführen. Gibt es eine große Anzahl von Ereignissen, können Sie einen Event Hub verwenden, um die Daten an verschiedene Computeressourcen zur Verarbeitung und Speicherung zu senden.

<a name="consolidating-instrumentation-data"></a>

#### <a name="consolidating-instrumentation-data"></a>*Konsolidierung von Instrumentierungsdaten*
Die vom Datensammlungsdienst aus einer einzelnen Instanz einer Anwendung abgerufenen Instrumentierungsdaten bieten eine lokalisierte Ansicht der Integrität und Leistung dieser Instanz. Um die allgemeine Integrität des Systems zu bewerten, ist es erforderlich, einige Aspekte der Daten in den lokalen Ansichten zu konsolidieren. Diesen Schritt können Sie nach der Speicherung der Daten durchführen. In einigen Fällen kann die Konsolidierung auch während der Sammlung erfolgen. Statt direkt in den freigegebenen Speicher geschrieben zu werden, können die Instrumentierungsdaten einen separaten Datenkonsolidierungsdienst durchlaufen, der Daten kombiniert und die Rolle eines Filter- und Bereinigungsprozesses übernimmt. Beispielsweise können Instrumentierungsdaten mit den gleichen Korrelationsinformationen, z. B. einer Aktivitäts-ID, zusammengelegt werden. (Es ist möglich, dass ein Benutzer einen Geschäftsvorgang auf einem Knoten beginnt und dann bei einem Knotenausfall oder durch die Konfiguration des Lastenausgleichs auf einen anderen Knoten verwiesen wird.) Dieser Prozess kann auch alle doppelt vorhandenen Daten erkennen und entfernen (immer eine Möglichkeit, wenn der Telemetriedienst Nachrichtenwarteschlangen verwendet, um Instrumentierungsdaten aus dem Speicher zu pushen). Abbildung 5 zeigt ein Beispiel für diese Struktur.

![Beispiel für das Verwenden eines Diensts zum Konsolidieren von Instrumentierungsdaten](./images/monitoring/Consolidation.png)

*Abbildung 5: Verwenden eines separaten Dienstes zum Konsolidieren und Bereinigen von Instrumentierungsdaten*

### <a name="storing-instrumentation-data"></a>Speichern von Instrumentierungsdaten
Die vorherigen Diskussionen haben auf eher simple Weise dargestellt, wie Instrumentierungsdaten gespeichert werden. In der Realität kann es sinnvoll sein, die verschiedenen Arten von Informationen mithilfe von Technologien zu speichern, die am besten für die Art der Verwendung geeignet sind.

Beispielsweise weisen Azure-Blob- und -Tabellenspeicher Gemeinsamkeiten im Hinblick auf den Zugriff auf. Es gibt jedoch Einschränkungen in den Vorgängen, die Sie damit ausführen können, und die Granularität der darin enthaltenen Daten ist sehr unterschiedlich. Wenn Sie weitere analytische Vorgänge durchführen müssen oder Volltext-Suchfunktionen für die Daten erforderlich sind, ist möglicherweise die Nutzung einer Datenspeicherung besser geeignet, die für bestimmte Typen von Abfragen und Datenzugriff optimierte Funktionen bietet. Beispiel:

* Leistungsindikatordaten können in einer SQL-Datenbank gespeichert werden, um eine Ad-hoc-Analyse zu ermöglichen.
* Ablaufverfolgungsprotokolle sollten eher in Azure Cosmos DB gespeichert werden.
* Sicherheitsinformationen können in HDFS geschrieben werden.
* Informationen, die eine Volltextsuche erfordern, können über Elasticsearch gespeichert werden. (So können Suchvorgänge auch mithilfe der umfassenden Indexierung beschleunigt werden.)

Sie können einen zusätzlichen Dienst implementieren, der die Daten in regelmäßigen Abständen aus dem freigegebenen Speicher abruft, sie partitioniert und gemäß ihrem Zweck filtert, und sie anschließend in einen entsprechenden Satz von Datenspeichern schreibt, wie in Abbildung 6 dargestellt. Ein alternativer Ansatz besteht darin, diese Funktionalität in den Konsolidierungs- und Bereinigungsvorgang einzubeziehen und die Daten direkt beim Abrufen in diese Speicher zu schreiben, anstatt sie in einem freigegebenen Zwischenspeicherbereich zu speichern. Jeder Ansatz hat Vor- und Nachteile. Das Implementieren eines separaten Partitionierungsdiensts verringert die Belastung des Konsolidierungs- und Bereinigungsdiensts und ermöglicht, dass wenigstens einige der partitionierten Daten bei Bedarf neu generiert werden (abhängig davon, wie viele Daten im freigegebenen Speicher beibehalten werden). Allerdings werden dadurch zusätzliche Ressourcen verbraucht. Zudem gibt es möglicherweise eine Verzögerung zwischen den Empfang von Instrumentierungsdaten von den einzelnen Anwendungsinstanzen und der Konvertierung dieser Daten in verwertbare Informationen.

![Partitionierung und Speicherung von Daten](./images/monitoring/DataStorage.png)

*Abbildung 6: Partitionierung von Daten anhand analytischer Anforderungen und Speicheranforderungen*

Die gleichen Instrumentierungsdaten können für mehrere Zwecke erforderlich sein. Leistungsindikatoren können z. B. verwendet werden, um eine historische Ansicht der Systemleistung über einen Zeitraum bereitzustellen. Diese Informationen können mit anderen Nutzungsdaten kombiniert werden, um Abrechnungsinformationen für Kunden zu generieren. In diesen Fällen können dieselben Daten an mehrere Ziele gesendet werden, z. B. eine Dokumentendatenbank, die als langfristiger Speicher für Abrechnungsinformationen dient, und einen mehrdimensionalen Speicher für die Verarbeitung komplexer Leistungsanalysen.

Sie sollten auch berücksichtigen, wie dringend die Daten benötigt werden. Daten, die Informationen zu Warnzwecken bereitstellen, müssen schnell zugänglich sein, daher sollten sie in einem schnellen Datenspeicher gehalten und so indiziert oder strukturiert werden, dass die vom Warnsystem durchgeführten Abfragen optimiert durchgeführt werden. In einigen Fällen kann es für den Telemetriedienst, der die Daten auf den einzelnen Knoten sammelt, notwendig sein, diese zu formatieren und lokal zu speichern, sodass eine lokale Instanz des Warnsystems Sie schnell auf Probleme hinweisen kann. Die gleichen Daten können an den in den vorherigen Diagrammen gezeigten Speicherschreibdienst verteilt und zentral gespeichert werden, wenn sie für weitere Zwecke erforderlich sind.

Informationen, die für eine eingehendere Analyse, für die Berichterstellung und für das Erkennen historischer Trends verwendet werden, sind weniger dringend und können so gespeichert werden, dass Data Mining und Ad-hoc-Abfragen unterstützt werden. Weitere Informationen finden Sie im Abschnitt [Unterstützung von heißer, warmer und kalter Analyse](#supporting-hot-warm-and-cold-analysis) weiter unten in diesem Dokument.

#### <a name="log-rotation-and-data-retention"></a>*Protokollrotation und Aufbewahrung von Daten*
Die Instrumentierung kann erhebliche Datenmengen generieren. Diese Daten können an mehreren Stellen gehalten werden, von den rohen Protokolldateien, den Ablaufverfolgungsdateien und anderen Informationen, die in jedem Knoten erfasst werden, zur konsolidierten, bereinigten und partitionierten Ansicht der Daten im freigegebenen Speicher. In einigen Fällen können die ursprünglichen, rohen Quelldaten aus den einzelnen Knoten entfernt werden, sobald die Daten verarbeitet und übertragen wurden. In anderen Fällen kann es notwendig oder einfach nützlich sein, die unformatierten Informationen zu speichern. Z. B. kann es möglicherweise besser sein, zu Debugzwecken generierte Daten in unformatierter Form zu belassen. Diese Daten können jedoch schnell verworfen werden, sobald alle Fehler behoben wurden.

Leistungsdaten haben häufig eine längere Lebensdauer, damit sie zum Erkennen von Leistungstrends und für die Kapazitätsplanung verwendet werden können. Die konsolidierte Ansicht dieser Daten bleibt in der Regel für einen begrenzten Zeitraum online, um einen schnellen Zugriff zu ermöglichen. Danach können diese Daten archiviert oder verworfen werden. Für die Messung und die Kundenabrechnung gesammelte Daten müssen möglicherweise dauerhaft gespeichert werden. Darüber hinaus kann es gesetzlich vorgeschrieben sein, dass für Überwachungs- und Sicherheitszwecke gesammelte Daten ebenfalls archiviert und gespeichert werden müssen. Diese Daten sind zudem vertraulich und müssen möglicherweise verschlüsselt oder auf andere Art geschützt werden, um Manipulationen zu verhindern. Sie sollten niemals Benutzerkennwörter oder andere Informationen aufzeichnen, die für Identitätsdiebstähle verwendet werden können. Diese Angaben sollten aus den Daten bereinigt werden, bevor sie gespeichert werden.

#### <a name="down-sampling"></a>*Downsampling*
Dies ist zum Speichern von Verlaufsdaten sinnvoll, um langfristige Trends zu ermitteln zu können. Statt alte Daten in ihrer Gesamtheit zu speichern, kann ein Downsampling der Daten möglich sein, um die Auflösung zu reduzieren und Speicherkosten zu sparen. Anstatt beispielsweise Leistungsindikatoren minütlich zu speichern, können Sie Daten, die über einen Monat alt sind, so konsolidieren, dass sie einen stündlichen Überblick geben.

### <a name="best-practices-for-collecting-and-storing-logging-information"></a>Bewährte Methoden für das Sammeln und Speichern von Protokollierungsinformationen
Die folgende Liste enthält bewährte Methoden zum Erfassen und Speichern von Protokollierungsinformationen:

* Der Überwachungs-Agent oder der Datensammlungsdienst sollte als prozessunabhängiger Dienst ausgeführt werden und einfach bereitzustellen sein.
* Die gesamte Ausgabe vom Überwachungs-Agent bzw. Datensammlungsdienst sollte ein agnostisches Format aufweisen, das unabhängig vom Computer, Betriebssystem oder Netzwerkprotokoll ist. Geben Sie z. B. Informationen in einem selbstbeschreibenden Format wie JSON, MessagePack oder Protobuf anstelle von ETL/ETW aus. Mithilfe standardisierter Formate kann das System Verarbeitungspipelines erstellen: Komponenten, die Daten im vereinbarten Format lesen, transformieren und ausgeben, können einfach integriert werden.
* Der Prozess der Überwachung und Datensammlung muss ausfallsicher sein und darf keine kaskadierenden Fehlerbedingungen auslösen.
* Bei einem vorübergehenden Fehler beim Senden der Informationen an eine Datensenke sollten der Überwachungs-Agent oder der Datensammlungsdienst darauf vorbereitet sein, Telemetriedaten neu anzuordnen, damit die neuesten Informationen zuerst gesendet werden. (Der Überwachungs-Agent/Datensammlungsdienst entscheidet sich möglicherweise nach eigenem Ermessen dazu, die älteren Daten zu löschen, oder dazu, sie lokal zu speichern und später zu senden.)

## <a name="analyzing-data-and-diagnosing-issues"></a>Analysieren von Daten und Diagnose von Problemen
Ein wichtiger Teil des Überwachungs- und Diagnoseprozesses ist die Analyse der gesammelten Daten, um einen Überblick über den allgemeinen Zustand des Systems zu erhalten. Sie sollten eigene KPIs und Leistungsmetriken definiert haben und wissen, wie Sie die gesammelten Daten so strukturieren können, dass sie Ihren Analyseanforderungen entsprechen. Sie sollten auch die in verschiedenen Metriken und Protokolldateien erfassten Daten in einen Zusammenhang setzen können, da diese Informationen für die Überwachung einer Sequenz von Ereignissen und bei der Diagnose auftretender Probleme entscheidend sein können.

Wie im Abschnitt [Konsolidierung von Instrumentierungsdaten](#consolidating-instrumentation-data)beschrieben, werden die Daten für jeden Teil des Systems in der Regel lokal erfasst, müssen aber im Allgemeinen mit Daten kombiniert werden, die an anderen am System beteiligten Standorten generiert werden. Diese Informationen erfordern eine sorgfältige Korrelation, um sicherzustellen, dass die Daten genau kombiniert werden. Die Verwendungsdaten für einen Vorgang können zum Beispiel einen Knoten, der eine Website für Benutzerverbindungen hostet, einen Knoten mit einem separaten Dienst, auf den im Rahmen dieses Vorgangs zugegriffen wird, und Datenspeicher auf einem weiteren Knoten umfassen. Diese Informationen müssen miteinander verknüpft werden, um einen allgemeinen Überblick über die Verwendung von Ressourcen und der Verarbeitung für den Vorgang zu bieten. Eine Vorverarbeitung und Filterung von Daten kann auf dem Knoten erfolgen, auf dem die Daten erfasst werden, während Aggregation und Formatierung eher auf einem zentralen Knoten ausgeführt werden.

<a name="supporting-hot-warm-and-cold-analysis"></a>

### <a name="supporting-hot-warm-and-cold-analysis"></a>Unterstützung von heißer, warmer und kalter Analyse
Das Analysieren und Umformatieren von Daten zu Visualisierungs-, Berichterstellungs- und Alarmierungszwecken kann ein komplexer Prozess sein, der einen eigenen Satz von Ressourcen nutzt. Einige Formen der Überwachung sind zeitkritisch und erfordern eine sofortige Analyse der Daten, um effektiv zu sein. Dies bezeichnet man als *heiße Analyse*. Beispiele hierfür sind die für Warnungen und einige Aspekte der Sicherheitsüberwachung erforderlichen Analysen (z. B. Erkennen eines Angriffs auf das System). Daten, die für diese Zwecke benötigt werden, müssen schnell verfügbar und für eine effiziente Verarbeitung strukturiert sein. In einigen Fällen kann es erforderlich sein, die Analyseverarbeitung auf die einzelnen Knoten zu verschieben, auf denen die Daten gespeichert sind.

Andere Arten der Analyse sind weniger zeitkritisch und erfordern möglicherweise Berechnung und Aggregation, nachdem die Rohdaten empfangen wurden. Dies wird als *warme Analyse* bezeichnet. Die Leistungsanalyse wird oft zu dieser Kategorie gezählt. In diesem Fall ist ein isoliertes, einzelnes Leistungsereignis statistisch wahrscheinlich nicht signifikant. (Es kann auf eine plötzliche Spitze oder Störung zurückzuführen sein.) Die Daten aus einer Reihe von Ereignissen sollten ein zuverlässigeres Bild der Systemleistung ergeben.

Die warme Analyse kann auch zur Diagnose von Integritätsproblemen verwendet werden. Ein Integritätsereignis wird in der Regel in einer heißen Analyse verarbeitet und kann sofort eine Warnung auslösen. Ein Operator sollte die Ursachen für das Integritätsereignis näher untersuchen können, indem er die Daten aus dem warmen Pfad analysiert. Diese Daten sollten Informationen über die Ereignisse enthalten, die zu dem Problem geführt haben, durch das das Integritätsereignis verursacht wurde.

Einige Arten der Überwachung generieren langfristigere Daten. Diese Analyse kann zu einem späteren Zeitpunkt, möglicherweise nach einem vordefinierten Zeitplan, ausgeführt werden. In einigen Fällen müssen bei der Analyse komplexe Filterungen großer Datenmengen durchgeführt werden, die über einen Zeitraum erfasst wurden. Dies wird als *kalte Analyse* bezeichnet. Die wichtigste Anforderung ist, dass die Daten nach ihrer Erfassung sicher gespeichert werden. Z. B. erfordert die Nutzungsüberwachung ein genaues Bild des Systemzustands zu regelmäßigen Zeiten, allerdings müssen diese Zustandsinformationen nicht umgehend für die Verarbeitung bereitstehen, nachdem sie gesammelt wurden.

Ein Operator kann die kalte Analyse auch zum Bereitstellen von Daten für die vorausschauende Integritätsanalyse verwenden. Der Operator kann historische Informationen über einen angegebenen Zeitraum sammeln und in Verbindung mit den aktuellen Integritätsdaten verwenden (abgerufen vom heißen Pfad), um Trends zu ermitteln, die bald zu Integritätsproblemen führen könnten. In diesen Fällen kann es erforderlich sein, eine Warnung auszulösen, damit Korrekturmaßnahmen eingeleitet werden können.

### <a name="correlating-data"></a>Korrelieren von Daten
Die von der Instrumentierung erfassten Daten können eine Momentaufnahme des Systemzustands bereitstellen, allerdings besteht der Zweck der Analyse darin, die Daten als Aktionsgrundlage verwenden zu können. Zum Beispiel:

* Wodurch wurde z. B. zu einem bestimmten Zeitpunkt eine ressourcenintensive E/A-Last auf Systemebene verursacht?
* Ist sie das Ergebnis einer großen Anzahl von Datenbankvorgängen?
* Schlägt sich dies in den Antwortzeiten der Datenbank nieder, der Anzahl der Transaktionen pro Sekunde und den Antwortzeiten der Anwendung am gleichen Verbindungspunkt?

In diesem Fall könnte eine korrigierende Maßnahme zum Reduzieren der Last darin bestehen, die Daten horizontal über mehrere Servern zu partitionieren. Darüber hinaus können auf jeder Ebene des Systems Ausnahmen aufgrund eines Fehlers auftreten. Eine Ausnahme auf einer Ebene löst häufig eine Ausnahme auf der darüber liegenden Ebene aus.

Aus diesen Gründen müssen Sie in der Lage sein, die verschiedenen Typen von Überwachungsdaten auf jeder Ebene zu korrelieren, um eine allgemeine Übersicht über den Zustand des Systems und der darauf ausgeführten Anwendungen zu erstellen. Sie können dann anhand dieser Informationen Entscheidungen darüber fällen, ob das System akzeptabel funktioniert oder nicht, und bestimmen, was zur Verbesserung der Systemqualität getan werden muss.

Wie im Abschnitt [Informationen zum Korrelieren von Daten](#information-for-correlating-data)beschrieben, müssen Sie sicherstellen, dass die unformatierten Instrumentierungsdaten genügend Informationen zu Kontext und Aktivitäts-ID enthalten, um die erforderlichen Aggregationen für korrelierende Ereignisse zu unterstützen. Darüber hinaus werden diese Daten möglicherweise in verschiedenen Formaten gespeichert, und es kann erforderlich sein, die Informationen zur Analyse in ein standardisiertes Format umzuwandeln.

### <a name="troubleshooting-and-diagnosing-issues"></a>Problembehandlung und Diagnose von Problemen
Für die Diagnose ist es erforderlich, die Ursache für Fehler oder unerwartetes Verhalten ermitteln zu können. Dazu gehört auch die Durchführung der Fehlerursachenanalyse. In der Regel sind folgende Informationen erforderlich:

* Ausführliche Informationen aus Ereignisprotokollen und Ablaufverfolgungen, entweder für das gesamte System oder für ein bestimmtes Subsystem in einem angegebenen Zeitfenster.
* Vollständige Stapelüberwachungen, die aus Ausnahmen und Fehlern einer angegebenen Stufe resultieren, welche während einer angegebenen Zeitspanne innerhalb des Systems oder auf einem angegebenen Subsystem auftreten.
* Absturzabbilder für fehlerhafte Prozesse an einer beliebigen Stelle im System oder für ein angegebenes Subsystem innerhalb eines angegebenen Zeitraums.
* Aktivitätsprotokolle mit den Aufzeichnungen der Vorgänge, die während einer angegebenen Zeitspanne von allen oder von ausgewählten Benutzern ausgeführt werden.

Das Analysieren von Daten zum Zweck der Problembehandlung erfordert häufig tiefgehende technische Kenntnisse der Systemarchitektur und der verschiedenen Komponenten der Lösung. Demzufolge ist häufig ein hohes Maß manuellen Eingreifens erforderlich, um die Daten zu interpretieren, die Ursache der Probleme festzustellen und eine geeignete Strategie zu deren Korrektur vorzuschlagen. Möglicherweise ist es zweckdienlich, eine Kopie dieser Informationen im ursprünglichen Format zu speichern und sie für die kalte Analyse durch einen Experten verfügbar zu machen.

## <a name="visualizing-data-and-raising-alerts"></a>Visualisieren von Daten und Auslösen von Warnungen
Ein wichtiger Aspekt eines jeden Überwachungssystems ist die Fähigkeit, die Daten so darzustellen, dass ein Operator Trends oder Probleme schnell erkennen kann. Zudem muss ein Operator schnell informiert werden können, wenn ein wichtiges Ereignis eingetreten ist, das möglicherweise seine Aufmerksamkeit erfordert.

Die Darstellung von Daten kann unterschiedliche Formen annehmen, einschließlich der Visualisierung mithilfe von Dashboards, Warnungen und Berichtserstellung.

### <a name="visualization-by-using-dashboards"></a>Visualisierung mithilfe von Dashboards
Die gängigste Methode zum Visualisieren von Daten ist die Verwendung von Dashboards, die Informationen als eine Reihe von Diagrammen, Graphen oder eine andere Art von Abbildung darstellen können. Diese Elemente könnten parametrisiert werden, und ein Analyst sollte die wichtigen Parameter (z. B. den Zeitraum) für jede beliebige Situation auswählen können.

Dashboards können hierarchisch organisiert werden. Dashboards der höchsten Ebene bieten einen allgemeinen Überblick über die einzelnen Aspekte des Systems, erlauben dem Operator jedoch auch, die näheren Details zu untersuchen. Ein Dashboard, das z. B. die gesamte E/A für das System darstellt, sollte es einem Analysten ermöglichen, die E/A-Raten für jeden einzelnen Datenträger zu ermitteln, um festzustellen, ob ein bestimmtes Gerät oder mehrere für eine unverhältnismäßig große Menge an Datenverkehr verantwortlich sind. Im Idealfall sollte das Dashboard auch verknüpfte Informationen anzeigen, wie z. B. die Quelle der einzelnen Anforderungen (Benutzer oder Aktivität), die diese E/A verursachen. Anhand dieser Informationen kann dann ermittelt werden, ob (und wie) die Last gleichmäßiger über die Geräte verteilt werden sollte und ob das System eine bessere Leistung zeigen würde, wenn weitere Geräte hinzugefügt würden.

Ein Dashboard könnte auch eine Farbcodierung oder andere visuelle Hinweise verwenden, um Werte anzuzeigen, die anomal scheinen oder außerhalb eines erwarteten Bereichs liegen. In Bezug auf das vorherige Beispiel:

* Ein Datenträger mit einer E/A-Rate, die sich über einen längeren Zeitraum der maximalen Kapazität nähert (heißer Datenträger) kann rot hervorgehoben werden.
* Ein Datenträger mit einer E/A-Rate, die sich regelmäßig über kurze Zeiträume am oberen Limit bewegt (warmer Datenträger) kann gelb hervorgehoben werden.
* Ein Datenträger mit normaler Nutzung kann grün angezeigt werden.

Beachten Sie, dass einem Dashboard die Rohdaten zur Verfügung stehen müssen, damit es effektiv funktioniert. Wenn Sie Ihr eigenes Dashboardsystem erstellen oder ein von einer anderen Organisation entwickeltes Dashboard nutzen, müssen Sie wissen, welche Instrumentierungsdaten Sie in welcher Granularität sammeln müssen und wie diese Daten für die Nutzung durch das Dashboard formatiert sein müssen.

Ein gutes Dashboard zeigt nicht nur Informationen an, es bietet einem Analysten auch die Möglichkeit, Ad-hoc-Fragen zu den Informationen zu stellen. Einige Systeme bieten Verwaltungstools, mit denen ein Operator diese Aufgaben ausführen und die zugrunde liegenden Daten untersuchen kann. Alternativ kann es je nach dem Repository, das zum Speichern der Informationen verwendet wird, möglich sein, diese Daten direkt abzufragen oder sie zur weiteren Analyse und Berichterstellung in Tools wie Microsoft Excel zu importieren.

> [!NOTE]
> Sie sollten den Zugriff auf Dashboards auf autorisierte Mitarbeiter beschränken, da die Informationen kommerziell vertraulich sein könnten. Sie sollten auch die den Dashboards zugrunde liegenden Daten schützen, um Benutzer an deren Änderung zu hindern.
> 
> 

### <a name="raising-alerts"></a>Warnungen auslösen
Unter Warnungen versteht man den Prozess der Analyse der Überwachungs- und Instrumentierungsdaten und dem Generieren einer Benachrichtigung, wenn ein wichtiges Ereignis erkannt wird.

Mit Warnungen kann man sicherstellen, dass das System fehlerfrei, reaktionsfähig und sicher bleibt. Sie sind ein wichtiger Teil jedes Systems, das den Benutzern Leistungs-, Verfügbarkeits- und Datenschutzgarantien zusichert und in dem möglicherweise sofort auf die Daten reagiert werden muss. Ein Operator muss unter Umständen über das Ereignis informiert werden, das die Warnung ausgelöst hat. Warnungen können auch zum Aufrufen von Systemfunktionen wie für die automatische Skalierung verwendet werden.

Warnungen hängen in der Regel von den folgenden Instrumentierungsdaten ab:

* Sicherheitsereignisse. Ergeben die Ereignisprotokolle, dass wiederholte Authentifizierungs- und/oder Autorisierungsfehler auftreten, wird das System möglicherweise gerade angegriffen, und ein Operator sollte informiert werden.
* Leistungsmetriken. Das System muss schnell reagieren, wenn eine bestimmte Leistungsmetrik einen angegebenen Schwellenwert überschreitet.
* Verfügbarkeitsinformationen. Wenn ein Fehler gefunden wird, kann es erforderlich sein, ein oder mehrere Subsysteme schnell neu zu starten oder ein Failover auf eine Backupressource durchzuführen. Wiederholte Fehler in einem Subsystem können auf schwerwiegendere Probleme hinweisen.

Operatoren können Warnungsinformationen über viele Übermittlungskanäle erhalten, wie z. B. E-Mail, Pager oder SMS. Eine Warnung kann auch einen Hinweis darauf enthalten, wie kritisch eine Situation ist. Viele Warnsysteme unterstützen Abonnentengruppen, und alle Operatoren, die Mitglieder derselben Gruppe sind, können dieselben Warnungen erhalten.

Ein Warnsystem sollten anpassbar sein, und die entsprechenden Werte aus den zugrunde liegenden Instrumentierungsdaten können als Parameter angegeben werden. Dieser Ansatz ermöglicht es einem Operator, Daten zu filtern und sich auf die Schwellenwerte oder Kombinationen von Werten zu konzentrieren, die von Interesse sind. Beachten Sie, dass dem Warnsystem in einigen Fällen die unformatierten Instrumentierungsdaten bereitgestellt werden können. In anderen Situationen ist es möglicherweise angebracht, aggregierte Daten bereitzustellen. (Beispielsweise kann eine Warnung ausgelöst werden, wenn die CPU-Auslastung für einen Knoten während der letzten 10 Minuten 90 Prozent überschritten hat.) Die dem Warnsystem bereitgestellten Angaben sollten auch alle erforderlichen Zusammenfassungs- und Kontextinformationen umfassen. Diese Daten können die Wahrscheinlichkeit verringern, dass falsch-positive Ereignisse eine Warnung auslösen.

### <a name="reporting"></a>Berichterstellung
Die Berichterstellung wird verwendet, um eine allgemeine Übersicht über das System zu generieren. Sie kann neben den aktuellen Informationen auch Verlaufsdaten umfassen. Anforderungen an die Berichterstellung selbst fallen in zwei breite Kategorien: die operative Berichterstellung für die Sicherheitsberichterstellung.

Die operative Berichterstellung umfasst in der Regel die folgenden Aspekte:

* Das Aggregieren von Statistiken, mit denen die Ressourcennutzung des gesamten Systems oder angegebener Subsysteme innerhalb eines angegebenen Zeitraums nachvollzogen werden kann.
* Das Identifizieren von Trends bei der Ressourcenverwendung für das gesamte System oder für die angegebenen Subsysteme während eines bestimmten Zeitraums.
* Das Überwachen von Ausnahmen, die im gesamten System oder in bestimmten Subsystemen während eines angegebenen Zeitraums aufgetreten sind.
* Das Ermitteln der Anwendungseffizienz im Hinblick auf die bereitgestellten Ressourcen und das Feststellen, ob die Menge der Ressourcen (und die damit verbundenen Kosten) reduziert werden können, ohne die Leistung unnötig zu beeinträchtigen.

Anhand der Sicherheitsberichterstellung wird die Nutzung des Systems durch den Kunden verfolgt. Dazu gehört:

* Überwachen von Benutzervorgängen. Dies erfordert das Aufzeichnen der einzelnen Anforderungen mit Datumsangaben und Uhrzeiten, die von jedem Benutzer ausgeführt wurden. Die Daten sollten strukturiert werden, damit ein Administrator die Reihenfolge der durch einen bestimmten Benutzer durchgeführten Vorgänge über einen bestimmten Zeitraum schnell rekonstruieren kann.
* Nachverfolgen der Ressourcennutzung nach Benutzer. Hierfür muss aufgezeichnet werden, wie und wie lange jede Anforderung für einen Benutzer auf die verschiedenen Ressourcen zugreift, aus denen das System besteht. Ein Administrator muss diese Daten nutzen können, um einen Nutzungsbericht pro Benutzer über einen angegebenen Zeitraum zu generieren, möglicherweise zu Abrechnungszwecken.

In vielen Fällen können Batchprozesse Berichte nach einem definierten Zeitplan generieren. (Latenz ist normalerweise kein Problem.) Sie sollten aber bei Bedarf auch auf Ad-hoc-Basis generiert werden können. Wenn Sie beispielsweise Daten in einer relationalen Datenbank speichern, wie z. B. der Azure SQL-Datenbank, können Sie ein Tool wie SQL Server Reporting Services zum Extrahieren und Formatieren der Daten verwenden und diese als eine Reihe von Berichten darstellen.

## <a name="related-patterns-and-guidance"></a>Zugehörige Muster und Anleitungen
* [Leitfaden zur automatischen Skalierung](../best-practices/auto-scaling.md) wird beschrieben, wie der Verwaltungsaufwand dadurch gesenkt wird, dass der Operator nicht mehr ständig die Leistung des Systems überwachen und Entscheidungen über das Hinzufügen und Entfernen von Ressourcen treffen muss.
* [Überwachungsmuster für den Integritätsendpunkt](https://msdn.microsoft.com/library/dn589789.aspx) (Muster zur Integritätsüberwachung für Endpunkte) beschreibt, wie Sie funktionale Prüfungen innerhalb einer Anwendung implementieren, auf die externe Tools in regelmäßigen Abständen über verfügbar gemachte Endpunkte zugreifen können.
* [Prioritätswarteschlangen-Muster](https://msdn.microsoft.com/library/dn589794.aspx) zeigt, wie Nachrichten in der Warteschlange so priorisiert werden, dass dringende Anforderungen vor weniger dringenden Nachrichten empfangen und verarbeitet werden.

## <a name="more-information"></a>Weitere Informationen
* [Microsoft Azure-Speicher: Überwachung, Diagnose und Problembehandlung](/azure/storage/storage-monitoring-diagnosing-troubleshooting)
* [Azure: Telemetry Basics and Troubleshooting (Telemetriegrundlagen und Problembehandlung)](http://social.technet.microsoft.com/wiki/contents/articles/18146.windows-azure-telemetry-basics-and-troubleshooting.aspx)
* [Aktivieren der Diagnose in Azure Cloud Services und Virtual Machines](/azure/cloud-services/cloud-services-dotnet-diagnostics)
* [Azure Redis Cache](https://azure.microsoft.com/services/cache/), [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) und [HDInsight](https://azure.microsoft.com/services/hdinsight/)
* [Verwenden von Service Bus-Warteschlangen](/azure/service-bus-messaging/service-bus-dotnet-get-started-with-queues)
* [SQL Server-Business Intelligence in Azure Virtual Machines](/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-bi)
* [Empfangen von Warnbenachrichtigungen](/azure/monitoring-and-diagnostics/insights-receive-alert-notifications) und [Nachverfolgen der Dienstintegrität](/azure/monitoring-and-diagnostics/insights-service-health)
* [Application Insights](/azure/application-insights/app-insights-overview)

