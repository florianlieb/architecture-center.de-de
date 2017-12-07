---
title: "Überwachung der Integrität von Endpunkten"
description: "Implementieren Sie Funktionsprüfungen in einer Anwendung, auf die externe Tools in regelmäßigen Abständen über verfügbar gemachte Endpunkte zugreifen können."
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- availability
- management-monitoring
- resiliency
ms.openlocfilehash: 36171d568b9b5bfbbd48ee762b16adea695cf0e9
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="health-endpoint-monitoring-pattern"></a>Muster für Überwachung der Integrität von Endpunkten

[!INCLUDE [header](../_includes/header.md)]

Implementieren Sie Funktionsprüfungen in einer Anwendung, auf die externe Tools in regelmäßigen Abständen über verfügbar gemachte Endpunkte zugreifen können. Dies kann helfen, die ordnungsgemäße Ausführung von Anwendungen und Diensten zu überprüfen.

## <a name="context-and-problem"></a>Kontext und Problem

Es ist eine bewährte Praxis und oft eine Geschäftsanforderung, Webanwendungen und Back-End-Dienste zu überwachen, um sicherzustellen, dass sie verfügbar sind und einwandfrei funktionieren. Allerdings lassen sich Dienste, die in der Cloud ausgeführt werden, schwieriger als lokale Dienste überwachen. Beispielsweise haben Sie nicht die volle Kontrolle über die Hostingumgebung, und die Dienste hängen in der Regel von anderen Diensten ab, die von Plattform- und anderen Anbietern bereitgestellt werden.

Es gibt viele Faktoren, die sich auf cloudbasierte Anwendungen auswirken, wie z.B. die Netzwerklatenz, Leistung und Verfügbarkeit der zugrunde liegenden Compute- und Speichersysteme und die Netzwerkbandbreite zwischen ihnen. Aufgrund dieser Faktoren kann der Dienst ganz oder teilweise ausfallen. Daher müssen Sie in regelmäßigen Abständen überprüfen, ob der Dienst ordnungsgemäß funktioniert, um den erforderlichen Verfügbarkeitsgrad sicherzustellen, der Teil Ihrer Vereinbarung zum Servicelevel (SLA) sein kann.

## <a name="solution"></a>Lösung

Implementieren Sie die Integritätsüberwachung, indem Sie Anforderungen an einen Endpunkt der Anwendung senden. Die Anwendung muss die notwendigen Prüfungen durchführen und eine Statusangabe zurückgeben.

Eine Überwachungsprüfung der Integrität kombiniert typischerweise zwei Faktoren:

- Die Prüfungen (falls vorhanden), die von der Anwendung oder dem Dienst als Antwort auf die Anforderung zur Überprüfung der Integrität des Endpunkts ausgeführt werden.
- Die Analyse der Ergebnisse durch das Tool oder Framework, das die Integritätsprüfung durchführt.

Der Antwortcode gibt den Status der Anwendung und optional der von ihr verwendeten Komponenten oder Dienste an. Die Wartezeit- oder Antwortzeitprüfung wird vom Überwachungstool oder -framework durchgeführt. Die Abbildung zeigt eine Übersicht über das Muster.

![Übersicht über das Muster](./_images/health-endpoint-monitoring-pattern.png)

Andere Prüfungen, die vom Code für die Integritätsüberwachung in der Anwendung durchgeführt werden können, sind z.B:
- Überprüfung der Verfügbarkeit und Antwortzeit eines Cloudspeichers oder einer Datenbank
- Überprüfung anderer Ressourcen oder Dienste, die sich in der Anwendung befinden oder an anderer Stelle, aber von der Anwendung verwendet werden

Es stehen Dienste und Tools zur Verfügung, die Webanwendungen überwachen, indem sie eine Anforderung an einen konfigurierbaren Satz von Endpunkten senden und die Ergebnisse im Abgleich mit einer Reihe konfigurierbarer Regeln auswerten. Es ist relativ einfach, einen Dienstendpunkt zu erstellen, dessen einziger Zweck es ist, einige Funktionstests auf das System anzuwenden.

Typische Prüfungen, die von den Überwachungstools durchgeführt werden können, sind z.B:

- Validieren des Antwortcodes. Die HTTP-Antwort 200 (OK) gibt beispielsweise an, dass die Anwendung fehlerfrei antwortet. Das Überwachungssystem kann auch auf andere Antwortcodes prüfen, um umfassendere Ergebnisse zu liefern.
- Überprüfung des Inhalts der Antwort, um Fehler zu erkennen, auch wenn der Statuscode 200 (OK) zurückgegeben wird. Hierdurch können Fehler erkannt werden, die nur einen Teil der zurückgegebenen Antwort der Webseite oder des Diensts betreffen. Zum Beispiel die Überprüfung des Titels einer Seite oder die Suche nach einer bestimmten Wortgruppe, die angibt, dass die richtige Seite zurückgegeben wurde.
- Messung der Antwortzeit, die eine Kombination aus der Netzwerklatenz und der Zeit angibt, die die Anwendung für die Ausführung der Anforderung benötigt hat. Ein steigender Wert kann auf ein auftauchendes Problem mit der Anwendung oder dem Netzwerk hinweisen.
- Überprüfung von Ressourcen oder Diensten außerhalb der Anwendung wie z.B. ein Content Delivery Network, das von der Anwendung verwendet wird, um Inhalte aus globalen Caches bereitzustellen.
- Überprüfung des Ablaufs von SSL-Zertifikaten.
- Messung der Antwortzeit eines DNS-Lookups nach der URL der Anwendung zum Messen von DNS-Latenz und DNS-Fehlern.
- Validierung der URL, die vom DNS-Lookup zurückgegeben wird, um richtige Einträge sicherzustellen. Dies kann helfen, böswillige Umleitungen von Anforderungen bei einem erfolgreichen Angriff auf den DNS-Server zu vermeiden.

Es ist auch nützlich, diese Prüfungen nach Möglichkeit von verschiedenen lokalen oder gehosteten Standorten aus durchzuführen, um die Antwortzeiten zu messen und zu vergleichen. Idealerweise sollten Sie Anwendungen von Standorten aus überwachen, die sich in der Nähe von Kunden befinden, um einen genauen Überblick über die Leistung jedes Standorts zu erhalten. Zusätzlich zur Bereitstellung eines zuverlässigeren Prüfmechanismus können die Ergebnisse Ihnen helfen, den Bereitstellungsort für die Anwendung zu bestimmen und zu entscheiden, ob sie in mehr als einem Rechenzentrum bereitgestellt werden soll.

Außerdem sollten Tests auf alle von Kunden verwendeten Dienstinstanzen angewendet werden, um sicherzustellen, dass die Anwendung für alle Kunden einwandfrei funktioniert. Wenn beispielsweise der Kundenspeicher auf mehr als ein Speicherkonto verteilt ist, sollte der Überwachungsprozess alle diese Konten überprüfen.

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:

Validierung der Antwort. Ist zum Beispiel bloß der Statuscode 200 (OK) ausreichend, um zu bestätigen, dass die Anwendung ordnungsgemäß funktioniert? Obwohl dies die grundlegendste Kennzahl für die Anwendungsverfügbarkeit und die Minimalimplementierung dieses Musters ist, liefert es wenig Informationen über die Vorgänge, Trends und mögliche anstehende Probleme in der Anwendung.

   >  Stellen Sie sicher, dass die Anwendung 200 (OK) richtigerweise nur dann zurückgibt, wenn die Zielressource gefunden und verarbeitet wird. In einigen Szenarien, wie z.B. bei Verwendung einer Masterseite als Host der Zielwebseite, sendet der Server den Statuscode 200 (OK) statt 404 (Nicht gefunden) zurück, auch wenn die Zielinhaltsseite nicht gefunden wurde.

Die Anzahl der Endpunkte, die für eine Anwendung verfügbar gemacht werden. Ein Ansatz sieht vor, dass mindestens ein Endpunkt für die Kerndienste, die die Anwendung verwendet, und ein weiterer Endpunkt für Dienste mit geringerer Priorität verfügbar gemacht wird, sodass jedem Überwachungsergebnis unterschiedliche Wichtigkeitsstufen zugewiesen werden können. Erwägen Sie außerdem, weitere Endpunkte, wie z.B. einen für jeden Kerndienst, für mehr Granularität bei der Überwachung verfügbar zu machen. Beispielsweise kann eine Integritätsprüfung die Datenbank, den Speicher und einen externen Geocodierungsdienst einer Anwendung überprüfen, wobei jede Komponente einen anderen Grad an Verfügbarkeit und Antwortzeit erfordert. Die Anwendung kann immer noch funktionieren, auch wenn der Geocodierungsdienst oder eine andere Hintergrundaufgabe für einige Minuten nicht verfügbar ist.

Ob für die Überwachung derselbe Endpunkt verwendet werden soll, der für den allgemeinen Zugriff verwendet wird, wobei aber auf dem Endpunkt für den allgemeinen Zugriff ein bestimmter Pfad gewählt wird, der für die Überprüfung der Integrität bestimmt ist, z.B. „/HealthCheck/{GUID}/“. Auf diese Weise können einige Funktionstests in der Anwendung von den Überwachungstools ausgeführt werden, wie z.B. das Hinzufügen einer neuen Benutzerregistrierung, das Anmelden und die Erteilung eines Testauftrags, während gleichzeitig überprüft wird, ob der allgemeine Zugriffsendpunkt verfügbar ist.

Die Art der Informationen, die im Dienst als Antwort auf Überwachungsanforderungen gesammelt werden sollen, und wie diese Informationen zurückgegeben werden können. Die meisten vorhandenen Tools und Frameworks untersuchen nur den HTTP-Statuscode, den der Endpunkt zurückgibt. Um zusätzliche Informationen zurückzugeben und zu überprüfen, müssen Sie möglicherweise ein Überwachungshilfsprogramm oder einen Dienst selbst erstellen.

Umfang der zu sammelnden Informationen. Eine übermäßige Verarbeitungslast während der Überprüfung kann die Anwendung ausbremsen und somit Auswirkungen auf andere Benutzer haben. Die dafür benötigte Zeit kann das Zeitlimit des Überwachungssystems überschreiten, sodass die Anwendung als nicht verfügbar markiert wird. Die meisten Anwendungen bieten eine Instrumentierung wie Fehlerhandler und Leistungsindikatoren, die die Leistung und detaillierte Fehlerinformationen protokollieren. Dies kann ausreichend sein, anstatt zusätzliche Informationen nach einer Integritätsprüfung zurückzugeben.

Zwischenspeichern des Endpunktstatus. Ein zu häufiges Durchführen der Integritätsprüfung kann mit hoher Systemlast verbunden sein. Wenn der Integritätsstatus beispielsweise über ein Dashboard gemeldet wird, möchten Sie nicht, dass jede Anforderung aus dem Dashboard eine Integritätsprüfung auslöst. Lassen Sie stattdessen den Systemstatus regelmäßig überprüfen und den Status zwischenspeichern. Machen Sie einen Endpunkt verfügbar, der den Status aus dem Cache zurückgibt.

Konfigurieren der Sicherheit der Überwachungsendpunkte zum Schutz vor öffentlichem Zugriff. Durch diesen kann die Anwendung böswilligen Angriffen ausgesetzt sein, aufgrund der Offenlegung sensibler Informationen gefährdet werden oder Opfer von Denial-of-Service-Angriffen (DoS) werden. Typischerweise sollte dies in der Anwendungskonfiguration erfolgen, damit problemlos eine Aktualisierung ohne Neustart der Anwendung durchgeführt werden kann. Erwägen Sie eine oder mehrere der folgenden Vorgehensweisen:

- Schützen Sie den Endpunkt, indem eine Authentifizierung angefordert wird. Sie können dies tun, indem Sie einen Authentifizierungssicherheitsschlüssel im Anforderungsheader verwenden oder Anmeldeinformationen mit der Anforderung übergeben, sofern der Überwachungsdienst oder das Tool die Authentifizierung unterstützt.

 - Verwenden Sie einen verborgenen oder ausgeblendeten Endpunkt. Machen Sie z.B. den Endpunkt an einer anderen IP-Adresse verfügbar als an der, die von der standardmäßigen Anwendungs-URL verwendet wird. Konfigurieren Sie den Endpunkt an einem nicht standardmäßigen HTTP-Port, und/oder verwenden Sie einen komplexen Pfad zur Testseite. In der Regel können Sie in der Anwendungskonfiguration zusätzliche Endpunktadressen und -ports angeben und bei Bedarf Einträge für diese Endpunkte dem DNS-Server hinzufügen, um die direkte Angabe der IP-Adresse zu vermeiden.

 - Machen Sie eine Methode auf einem Endpunkt verfügbar, die einen Parameter wie einen Schlüsselwert oder einen Wert für die Betriebsart akzeptiert. Je nach Wert, der für diesen Parameter angegeben wird, kann der Code beim Empfangen einer Anforderung einen oder mehrere bestimmte Tests durchführen oder den Fehler 404 (Nicht gefunden) zurückgeben, wenn der Parameterwert nicht erkannt wird. Die erkannten Parameterwerte können in der Anwendungskonfiguration festgelegt werden.

     >  DoS-Angriffe haben wahrscheinlich weniger Auswirkungen auf einen separaten Endpunkt, auf dem grundlegende Funktionstests durchgeführt werden, ohne den Betrieb der Anwendung zu beeinträchtigen. Vermeiden Sie im Idealfall Tests, die vertrauliche Informationen preisgeben. Wenn Sie Informationen zurückgeben müssen, die für einen Angreifer nützlich sein könnten, überlegen Sie, wie Sie den Endpunkt und die Daten vor unbefugtem Zugriff schützen können. In diesem Fall reicht es nicht aus, sich nur auf Verschleierung zu verlassen. Außerdem sollten Sie in Erwägung ziehen, eine HTTPS-Verbindung zu verwenden und sensible Daten zu verschlüsseln, obwohl dies die Verarbeitungslast des Servers erhöht.

- Zugreifen auf einen Endpunkt, der durch Authentifizierung geschützt ist. Nicht alle Tools und Frameworks können so konfiguriert werden, dass Anmeldeinformationen in die Anforderung der Integritätsprüfung einbezogen werden. Beispielsweise können die in Microsoft Azure integrierten Funktionen zur Integritätsprüfung keine Authentifizierungsinformationen bereitstellen. Einige Alternativen von Drittanbietern sind [Pingdom](https://www.pingdom.com/), [Panopta](http://www.panopta.com/), [NewRelic](https://newrelic.com/) und [Statuscake](https://www.statuscake.com/).

- Sicherstellen, dass der Überwachungs-Agent richtig funktioniert. Ein Ansatz sieht vor, einen Endpunkt verfügbar zu machen, der einfach einen Wert aus der Anwendungskonfiguration oder einen Zufallswert zurückgibt, der zum Testen des Agents verwendet werden kann.

   >  Stellen Sie außerdem sicher, dass das Überwachungssystem sich selbst überprüft, z.B. durch einen Selbsttest und integrierten Test, um falsche positive Ergebnisse zu vermeiden.

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

Dieses Muster ist für die folgenden Aufgaben hilfreich:
- Überwachung von Websites und Webanwendungen auf Verfügbarkeit.
- Überwachung von Websites und Webanwendungen auf ordnungsgemäßen Betrieb.
- Überwachung von Diensten auf mittlerer Ebene oder gemeinsam genutzten Diensten, um einen Fehler zu erkennen und zu isolieren, der andere Anwendungen stören könnte.
- Ergänzen Sie die vorhandene Instrumentierung in der Anwendung, z.B. Leistungsindikatoren und Fehlerhandler. Die Integritätsprüfung macht die Protokollierung und Prüfung in der Anwendung nicht überflüssig. Die Instrumentierung kann nützliche Informationen für ein bestehendes Framework liefern, das Leistungsindikatoren und Fehlerprotokolle überwacht, um Fehlfunktionen oder andere Probleme zu erkennen. Sie kann jedoch keine Informationen liefern, wenn die Anwendung nicht verfügbar ist.

## <a name="example"></a>Beispiel

Die folgenden Codebeispiele, die der `HealthCheckController`-Klasse entnommen wurden (ein Beispiel, das dieses Muster veranschaulicht, ist auf [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring) verfügbar), demonstrieren das Verfügbarmachen eines Endpunkts für die Durchführung einer Reihe von Integritätsprüfungen.

Die `CoreServices`-Methode, die unten in C# gezeigt wird, wendet eine Reihe von Prüfungen auf Dienste an, die in der Anwendung verwendet werden. Wenn alle Tests fehlerfrei verlaufen, gibt die Methode den Statuscode 200 (OK) zurück. Wenn einer der Tests eine Ausnahme auslöst, gibt die Methode den Statuscode 500 (Interner Fehler) zurück. Die Methode kann optional zusätzliche Informationen zurückgeben, sobald ein Fehler auftritt, wenn das Überwachungstool oder -framework diese nutzen kann.

```csharp
public ActionResult CoreServices()
{
  try
  {
    // Run a simple check to ensure the database is available.
    DataStore.Instance.CoreHealthCheck();

    // Run a simple check on our external service.
    MyExternalService.Instance.CoreHealthCheck();
  }
  catch (Exception ex)
  {
    Trace.TraceError("Exception in basic health check: {0}", ex.Message);

    // This can optionally return different status codes based on the exception.
    // Optionally it could return more details about the exception.
    // The additional information could be used by administrators who access the
    // endpoint with a browser, or using a ping utility that can display the
    // additional information.
    return new HttpStatusCodeResult((int)HttpStatusCode.InternalServerError);
  }
  return new HttpStatusCodeResult((int)HttpStatusCode.OK);
}
```
Die `ObscurePath`-Methode zeigt, wie Sie einen Pfad in der Anwendungskonfiguration lesen und als Endpunkt für Tests verwenden können. Dieses Beispiel in C# zeigt auch, wie Sie eine ID als Parameter akzeptieren und verwenden können, um nach gültigen Anforderungen zu suchen.

```csharp
public ActionResult ObscurePath(string id)
{
  // The id could be used as a simple way to obscure or hide the endpoint.
  // The id to match could be retrieved from configuration and, if matched,
  // perform a specific set of tests and return the result. If not matched it
  // could return a 404 (Not Found) status.

  // The obscure path can be set through configuration to hide the endpoint.
  var hiddenPathKey = CloudConfigurationManager.GetSetting("Test.ObscurePath");

  // If the value passed does not match that in configuration, return 404 (Not Found).
  if (!string.Equals(id, hiddenPathKey))
  {
    return new HttpStatusCodeResult((int)HttpStatusCode.NotFound);
  }

  // Else continue and run the tests...
  // Return results from the core services test.
  return this.CoreServices();
}
```

Die `TestResponseFromConfig`-Methode zeigt, wie Sie einen Endpunkt verfügbar machen können, der eine Überprüfung auf einen bestimmten Konfigurationseinstellungswert durchführt.

```csharp
public ActionResult TestResponseFromConfig()
{
  // Health check that returns a response code set in configuration for testing.
  var returnStatusCodeSetting = CloudConfigurationManager.GetSetting(
                                                          "Test.ReturnStatusCode");

  int returnStatusCode;

  if (!int.TryParse(returnStatusCodeSetting, out returnStatusCode))
  {
    returnStatusCode = (int)HttpStatusCode.OK;
  }

  return new HttpStatusCodeResult(returnStatusCode);
}
```
## <a name="monitoring-endpoints-in-azure-hosted-applications"></a>Überwachen von Endpunkten in von Azure gehosteten Anwendungen

Es folgen Optionen für die Überwachung von Endpunkten in Azure-Anwendungen:

- Verwenden Sie die integrierten Überwachungsfunktionen von Azure.

- Verwenden Sie einen Dienst oder ein Framework eines Drittanbieters wie Microsoft System Center Operations Manager.

- Erstellen Sie selbst ein Hilfsprogramm oder einen Dienst, das/der auf Ihrem eigenen oder einem gehosteten Server ausgeführt wird.

   >  Auch wenn Azure eine recht umfassende Reihe von Überwachungsoptionen bietet, können Sie zusätzliche Dienste und Tools nutzen, um zusätzliche Informationen bereitzustellen. Der Azure-Verwaltungsdienst bietet einen integrierten Überwachungsmechanismus für Warnungsregeln. Im Abschnitt „Warnungen“ auf der Seite der Verwaltungsdienste im Azure-Portal können Sie bis zu zehn Warnungsregeln pro Abonnement für Ihre Dienste konfigurieren. Diese Regeln geben eine Bedingung und einen Schwellenwert für einen Dienst an, wie z.B. die CPU-Auslastung oder Anzahl der Anforderungen oder Fehler pro Sekunde. Der Dienst kann automatisch E-Mail-Benachrichtigungen an Adressen senden, die Sie in jeder Regel definieren.

Die Bedingungen, die Sie überwachen können, variieren je nach Hostingmechanismus, den Sie für Ihre Anwendung wählen (wie z. B. Azure Web Sites, Azure Cloud Services, Azure Virtual Machines oder Azure Mobile Services). Doch alle bieten die Möglichkeit, eine Warnungsregel zu erstellen, die einen Webendpunkt verwendet, den Sie in den Einstellungen für Ihren Dienst angeben. Dieser Endpunkt muss rechtzeitig reagieren, damit das Warnsystem erkennen kann, dass die Anwendung ordnungsgemäß funktioniert.

>  Weitere Informationen finden Sie unter [Erstellen von Warnungsbenachrichtigungen][portal-alerts].

Wenn Sie Ihre Anwendung in Azure Cloud Services-Web- und -Workerrollen oder Azure Virtual Machines hosten, können Sie einen der in Azure integrierten Dienste namens Traffic Manager nutzen. Traffic Manager ist ein Routing- und Lastenausgleichsdienst, der Anforderungen an bestimmte Instanzen Ihrer gehosteten Cloud Services-Anwendung auf Grundlage einer Reihe von Regeln und Einstellungen verteilen kann.

Zusätzlich zu den Routinganforderungen pingt Traffic Manager regelmäßig eine URL, einen Port und einen relativen Pfad, die/den Sie angeben, um festzustellen, welche Instanzen der in seinen Regeln definierten Anwendung aktiv sind und auf Anforderungen reagieren. Wenn der Statuscode 200 (OK) erkannt wird, markiert Traffic Manager die Anwendung als verfügbar. Alle anderen Statuscodes bewirken, dass Traffic Manager die Anwendung als offline markiert. Sie können den Status in der Traffic Manager-Konsole anzeigen und die Regel so konfigurieren, dass Anforderungen an andere Instanzen der Anwendung umgeleitet werden, die antworten.

Traffic Manager wartet jedoch nur zehn Sekunden auf eine Antwort von der Überwachungs-URL. Daher müssen Sie sicherstellen, dass Ihr Code zur Überprüfung des Integritätsstatus in dieser Zeit ausgeführt wird, wobei die Netzwerklatenz für den Roundtrip vom Traffic Manager zu Ihrer Anwendung und zurück berücksichtigt werden muss.

>  Erfahren Sie mehr zur Verwendung von [Traffic Manager zum Überwachen Ihrer Anwendungen](https://azure.microsoft.com/documentation/services/traffic-manager/). Traffic Manager wird auch unter [Multiple Datacenter Deployment Guidance](https://msdn.microsoft.com/library/dn589779.aspx) (Bereitstellungsanleitung für mehrere Rechenzentren) erläutert.

## <a name="related-guidance"></a>Verwandte Leitfäden

Der folgende Leitfaden kann für die Implementierung dieses Musters relevant sein:
- [Instrumentierungs- und Telemetrieanleitung](https://msdn.microsoft.com/library/dn589775.aspx). Die Überprüfung des Status von Diensten und Komponenten erfolgt in der Regel mithilfe von Tests. Es ist jedoch auch nützlich, über Informationen zu verfügen, um die Anwendungsleistung zu überwachen und Ereignisse zu erkennen, die zur Laufzeit auftreten. Diese Daten können an Überwachungstools als Zusatzinformationen für die Integritätsüberwachung zurückgegeben werden. In der Instrumentierungs- und Telemetrieanleitung wird das Erfassen von Ferndiagnoseinformationen untersucht, die mithilfe der Instrumentierung in Anwendungen gesammelt werden.
- [Empfangen von Warnungsbenachrichtigungen][portal-alerts].
- Zu diesem Muster gehört eine herunterladbare [Beispielanwendung](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring).

[portal-alerts]: https://azure.microsoft.com/documentation/articles/insights-receive-alert-notifications/
