---
title: Fehlermodusanalyse
description: Anleitungen zur Durchführung der Fehlermodusanalyse für Cloudlösungen, die auf Azure basieren.
author: MikeWasson
ms.date: 03/24/2017
ms.custom: resiliency
pnp.series.title: Design for Resiliency
ms.openlocfilehash: 95068bf8b1f5b559255e27819aaddb454d3427bc
ms.sourcegitcommit: d08f6ee27e1e8a623aeee32d298e616bc9bb87ff
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/07/2018
---
# <a name="failure-mode-analysis"></a>Fehlermodusanalyse
[!INCLUDE [header](../_includes/header.md)]

Die Fehlermodusanalyse (Failure Mode Analysis, FMA) ist ein Prozess zum Erzielen von Resilienz in einem System durch das Identifizieren möglicher Schwachstellen im System. Die Fehlermodusanalyse sollte in die Planungs- und Designphasen eingebunden werden, damit Sie die Wiederherstellung nach einem Fehler von Anfang an in das System integrieren können.

Hier folgt der allgemeine Prozess zum Durchführen einer Fehlermodusanalyse:

1. Identifizieren Sie alle Komponenten im System. Beziehen Sie externe Abhängigkeiten ein, z. B. Identitätsanbieter, Drittanbieterdienste usw.   
2. Identifizieren Sie für jede Komponente potenzielle Fehler, die auftreten können. Eine einzelne Komponente kann mehrere Fehlermodi aufweisen. Beispielsweise sollten Sie Lese- und Schreibfehler getrennt betrachten, da deren Auswirkung und mögliche Entschärfung unterschiedlich ausfallen werden.
3. Bewerten Sie die einzelnen Fehlermodi nach ihrem Gesamtrisiko. Beachten Sie folgende Faktoren:  

   * Wie hoch ist die Fehlerwahrscheinlichkeit? Tritt der Fehler relativ häufig auf? Tritt der Fehler sehr selten auf? Sie brauchen keine exakten Zahlen. Der Zweck besteht darin, der Priorität einen Rang zuzuweisen.
   * Welche Auswirkungen hat dies auf die Anwendung in Bezug auf Verfügbarkeit, Datenverlust, Kosten und Geschäftsunterbrechung?
4. Legen Sie für jeden Fehlermodus fest, wie die Anwendung reagieren und wiederhergestellt werden soll. Berücksichtigen Sie Kompromisse in Bezug auf Kosten und Anwendungskomplexität.   

Dieser Artikel enthält als Ausgangspunkt für den FMA-Prozess einen Katalog mit möglichen Fehlerzuständen und den entsprechenden Gegenmaßnahmen. Der Katalog ist nach Technologie oder Azure-Dienst sowie einer zusätzlichen allgemeinen Kategorie für das Design auf Anwendungsebene gegliedert. Der Katalog ist nicht komplett, deckt aber viele der wichtigsten Azure-Dienste ab.

## <a name="app-service"></a>App Service
### <a name="app-service-app-shuts-down"></a>Die App Service-App wird heruntergefahren.
**Erkennung**: Mögliche Ursachen:

* Erwartetes Herunterfahren

  * Ein Operator fährt die Anwendung herunter, z. B. über das Azure-Portal.
  * Die App wurde aufgrund von Inaktivität entladen. (Nur, wenn die Einstellung `Always On` deaktiviert ist.)
* Unerwartetes Herunterfahren

  * Die App stürzt ab.
  * Eine App Service-VM-Instanz steht nicht mehr zur Verfügung.

Die „Application_End“-Protokollierung fängt das Herunterfahren der Anwendungsdomäne ab (sanfter Prozessabsturz). Dies ist die einzige Möglichkeit, das Herunterfahren der Anwendungsdomäne abzufangen.

**Wiederherstellung**:

* Wenn das Herunterfahren erwartet wurde, verwenden Sie das Herunterfahrereignis der Anwendung, um das Programm ordnungsgemäß herunterzufahren. Verwenden Sie z. B. in ASP.NET die Methode `Application_End`.
* Wenn die Anwendung bei Inaktivität entladen wurde, wird sie bei der nächsten Anforderung automatisch neu gestartet. Allerdings fallen für Sie die Kosten für den „Kaltstart“ an.
* Um zu verhindern, dass die Anwendung bei Inaktivität entladen wird, aktivieren Sie die Einstellung `Always On` in der Web-App. Weitere Informationen finden Sie unter [Konfigurieren von Web-Apps in Azure App Service][app-service-configure].
* Um zu verhindern, dass ein Operator die Anwendung herunterfährt, legen Sie eine Ressourcensperre mit `ReadOnly`-Ebene fest. Weitere Informationen finden Sie unter [Sperren von Ressourcen mit Azure Resource Manager][rm-locks].
* Wenn die App abstürzt oder eine App Service-VM ist nicht mehr verfügbar ist, startet App Service die App automatisch neu.

**Diagnose**: Anwendungs- und Webserverprotokolle. Weitere Informationen finden Sie unter [Aktivieren der Diagnoseprotokollierung für Web-Apps in Azure App Service][app-service-logging].

### <a name="a-particular-user-repeatedly-makes-bad-requests-or-overloads-the-system"></a>Ein bestimmter Benutzer stellt wiederholt unzulässige Anforderungen oder überlastet das System.
**Erkennung**: Authentifizieren Sie Benutzer, und beziehen Sie die Benutzer-ID in die Anwendungsprotokolle ein.

**Wiederherstellung**:

* Verwenden Sie [Azure API Management][api-management], um Anforderungen von dem Benutzer zu beschränken. Weitere Informationen finden Sie unter [Erweiterte Anforderungsbegrenzung mit Azure API Management][api-management-throttling].
* Blockieren Sie den Benutzer.

**Diagnose**: Protokollieren Sie alle Authentifizierungsanforderungen.

### <a name="a-bad-update-was-deployed"></a>Es wurde eine unzulässige Aktualisierung bereitgestellt.
**Erkennung**: Überwachen Sie die Anwendungsintegrität über das Azure-Portal (weitere Informationen finden Sie unter [Überwachen der Azure Web App-Leistung][app-insights-web-apps]), oder implementieren Sie das [Überwachungsmuster für den Integritätsendpunkt][health-endpoint-monitoring-pattern].

**Wiederherstellen**. Verwenden Sie mehrere [Bereitstellungsslots][app-service-slots], und führen Sie ein Rollback auf die letzte bekannte geeignete Bereitstellung aus. Weitere Informationen finden Sie unter [Einfache Webanwendung][ra-web-apps-basic].

## <a name="azure-active-directory"></a>Azure Active Directory
### <a name="openid-connect-oidc-authentication-fails"></a>Fehler bei der OpenID Connect-Authentifizierung.
**Erkennung**: Mögliche Fehlermodi:

1. Azure AD ist nicht verfügbar oder kann aufgrund eines Netzwerkproblems nicht erreicht werden. Bei der Umleitung zum Authentifizierungsendpunkt ist ein Fehler aufgetreten und die OpenID Connect-Middleware (OICD) löst eine Ausnahme aus.
2. Der Azure AD-Mandant ist nicht vorhanden. Bei der Umleitung zum Authentifizierungsendpunkt wird ein HTTP-Fehlercode zurückgegeben und die OpenID Connect-Middleware (OICD) löst eine Ausnahme aus.
3. Der Benutzer kann nicht authentifiziert werden. Es ist keine Erkennungsstrategie erforderlich. Azure AD behandelt Anmeldefehler.

**Wiederherstellung**:

1. Fangen Sie nicht behandelte Ausnahmen von der Middleware ab.
2. Behandeln Sie `AuthenticationFailed`-Ereignisse.
3. Leiten Sie den Benutzer zu einer Fehlerseite um.
4. Der Benutzer versucht den Vorgang erneut.

## <a name="azure-search"></a>Azure Search
### <a name="writing-data-to-azure-search-fails"></a>Beim Schreiben von Daten in Azure Search tritt ein Fehler auf.
**Erkennung**: Fangen Sie `Microsoft.Rest.Azure.CloudException`-Fehler ab.

**Wiederherstellung**:

Das [Search .NET SDK][search-sdk] versucht den Vorgang nach vorübergehenden Fehlern automatisch erneut. Alle Ausnahmen, die vom Client-SDK ausgelöst werden, sollten als nicht vorübergehende Fehler behandelt werden.

Die standardmäßige Wiederholungsrichtlinie verwendet einen exponentiellen Backoff. Rufen Sie `SetRetryPolicy` für die Klasse `SearchIndexClient` oder `SearchServiceClient` auf, um eine andere Wiederholungsrichtlinie zu verwenden. Weitere Informationen finden Sie unter [Automatische Wiederholungsversuche][auto-rest-client-retry].

**Diagnose**: Verwenden Sie [Durchsuchen der Datenverkehrsanalyse][search-analytics].

### <a name="reading-data-from-azure-search-fails"></a>Beim Lesen von Daten aus Azure Search tritt ein Fehler auf.
**Erkennung**: Fangen Sie `Microsoft.Rest.Azure.CloudException`-Fehler ab.

**Wiederherstellung**:

Das [Search .NET SDK][search-sdk] versucht den Vorgang nach vorübergehenden Fehlern automatisch erneut. Alle Ausnahmen, die vom Client-SDK ausgelöst werden, sollten als nicht vorübergehende Fehler behandelt werden.

Die standardmäßige Wiederholungsrichtlinie verwendet einen exponentiellen Backoff. Rufen Sie `SetRetryPolicy` für die Klasse `SearchIndexClient` oder `SearchServiceClient` auf, um eine andere Wiederholungsrichtlinie zu verwenden. Weitere Informationen finden Sie unter [Automatische Wiederholungsversuche][auto-rest-client-retry].

**Diagnose**: Verwenden Sie [Durchsuchen der Datenverkehrsanalyse][search-analytics].

## <a name="cassandra"></a>Cassandra
### <a name="reading-or-writing-to-a-node-fails"></a>Beim Lesen aus einem Knoten oder beim Schreiben in einen Knoten ist ein Fehler aufgetreten.
**Erkennung**: Fangen Sie die Ausnahme ab. Für .NET-Clients ist dies normalerweise `System.Web.HttpException`. Andere Clients weisen möglicherweise andere Ausnahmetypen auf.  Weitere Informationen finden Sie unter [Cassandra error handling done right](http://www.datastax.com/dev/blog/cassandra-error-handling-done-right) (Richtige Cassandra-Fehlerbehandlung).

**Wiederherstellung**:

* Jeder [Cassandra-Client](https://wiki.apache.org/cassandra/ClientOptions) verfügt über eigene Wiederholungsrichtlinien und -funktionen. Weitere Informationen finden Sie unter [Cassandra error handling done right][cassandra-error-handling] (Richtige Cassandra-Fehlerbehandlung).
* Verwenden Sie eine rackfähige Bereitstellung mit Datenknoten, die über die Fehlerdomänen verteilt sind.
* Nehmen Sie die Bereitstellung in mehreren Regionen mit lokaler Quorumkonsistenz vor. Wenn ein nicht vorübergehender Fehler auftritt, führen Sie einen Failover in eine andere Region aus.

**Diagnose**: Anwendungsprotokolle

## <a name="cloud-service"></a>Clouddienst
### <a name="web-or-worker-roles-are-unexpectedlybeing-shut-down"></a>Web- oder Workerrollen werden unerwartet heruntergefahren.
**Erkennung**: Das Ereignis [RoleEnvironment.Stopping][RoleEnvironment.Stopping] wird ausgelöst.

<strong>Wiederherstellen</strong>. Setzen Sie die Methode [RoleEntryPoint.OnStop][RoleEntryPoint.OnStop] für eine ordnungsgemäße Bereinigung außer Kraft. Weitere Informationen finden Sie unter [The Right Way to Handle Azure OnStop Events][onstop-events] (Der richtige Weg zum Behandeln von Azure OnStop-Ereignissen – Blog).

## <a name="cosmos-db"></a>Cosmos DB 
### <a name="reading-data-fails"></a>Beim Lesen von Daten tritt ein Fehler auf.
**Erkennung**: Fangen Sie `System.Net.Http.HttpRequestException` oder `Microsoft.Azure.Documents.DocumentClientException` ab.

**Wiederherstellung**:

* Das SDK führt bei Fehlversuchen automatisch Wiederholungsversuche durch. Konfigurieren Sie zum Festlegen der Anzahl von Wiederholungsversuchen und der maximalen Wartezeit `ConnectionPolicy.RetryOptions`. Ausnahmen, die vom Client ausgelöst werden, unterliegen entweder nicht der Wiederholungsrichtlinie oder sind keine vorübergehenden Fehler.
* Wenn Cosmos DB den Client einschränkt, wird ein HTTP 429-Fehler zurückgegeben. Überprüfen Sie den Statuscode unter `DocumentClientException`. Wenn Sie ständig den Fehler 429 erhalten, sollten Sie in Erwägung ziehen, den Durchsatzwert der Sammlung zu erhöhen.
    * Wenn Sie die MongoDB-API verwenden, gibt der Dienst bei der Einschränkung den Fehlercode 16500 zurück.
* Replizieren Sie die Cosmos DB-Datenbank über zwei oder mehr Regionen hinweg. Alle Replikate können gelesen werden. Geben Sie den Parameter `PreferredLocations` mithilfe des Client-SDKs an. Dies ist eine sortierte Liste von Azure-Regionen. Alle Lesevorgänge werden an die erste verfügbare Region in der Liste gesendet. Wenn bei der Anforderung ein Fehler auftritt, versucht der Client die anderen Regionen in der Liste in der entsprechenden Reihenfolge. Weitere Informationen finden Sie unter [Einrichten der globalen Verteilung von Azure Cosmos DB mithilfe der SQL-API][cosmosdb-multi-region].

**Diagnose**: Protokollieren Sie alle Fehler auf der Clientseite.

### <a name="writing-data-fails"></a>Beim Schreiben von Daten tritt ein Fehler auf.
**Erkennung**: Fangen Sie `System.Net.Http.HttpRequestException` oder `Microsoft.Azure.Documents.DocumentClientException` ab.

**Wiederherstellung**:

* Das SDK führt bei Fehlversuchen automatisch Wiederholungsversuche durch. Konfigurieren Sie zum Festlegen der Anzahl von Wiederholungsversuchen und der maximalen Wartezeit `ConnectionPolicy.RetryOptions`. Ausnahmen, die vom Client ausgelöst werden, unterliegen entweder nicht der Wiederholungsrichtlinie oder sind keine vorübergehenden Fehler.
* Wenn Cosmos DB den Client einschränkt, wird ein HTTP 429-Fehler zurückgegeben. Überprüfen Sie den Statuscode unter `DocumentClientException`. Wenn Sie ständig den Fehler 429 erhalten, sollten Sie in Erwägung ziehen, den Durchsatzwert der Sammlung zu erhöhen.
* Replizieren Sie die Cosmos DB-Datenbank über zwei oder mehr Regionen hinweg. Tritt bei der primären Region ein Fehler auf, wird eine andere Region für den Schreibvorgang höher gestuft. Sie können einen Failover auch manuell auslösen. Das SDK führt die automatische Erkennung und das automatische Routing durch, sodass der Anwendungscode auch nach einem Failover weiterhin funktioniert. Während der Failoverperiode (normalerweise Minuten) weisen Schreibvorgänge eine höhere Wartezeit auf, da das SDK die neue Schreibregion sucht.
  Weitere Informationen finden Sie unter [Einrichten der globalen Verteilung von Azure Cosmos DB mithilfe der SQL-API][cosmosdb-multi-region].
* Als Fallback wird das Dokument in eine Sicherungswarteschlange gestellt und die Warteschlange später verarbeitet.

**Diagnose**: Protokollieren Sie alle Fehler auf der Clientseite.

## <a name="elasticsearch"></a>Elasticsearch
### <a name="reading-data-from-elasticsearch-fails"></a>Beim Lesen von Daten aus Elasticsearch tritt ein Fehler auf.
**Erkennung**: Fangen Sie die entsprechende Ausnahme für den verwendeten [Elasticsearch-Client][elasticsearch-client] ab.

**Wiederherstellung**:

* Verwenden Sie einen Wiederholungsmechanismus. Jeder Client verfügt über eigene Wiederholungsrichtlinien.
* Stellen Sie mehrere Elasticsearch-Knoten bereit, und verwenden Sie die Replikation für die Hochverfügbarkeit.

Weitere Informationen finden Sie unter [Ausführen von Elasticsearch für Azure][elasticsearch-azure].

**Diagnose**: Sie können Überwachungstools für Elasticsearch verwenden oder alle Fehler auf der Clientseite mit der Nutzlast protokollieren. Weitere Informationen finden Sie im Abschnitt „Überwachung“ in [Ausführen von Elasticsearch für Azure][elasticsearch-azure].

### <a name="writing-data-to-elasticsearch-fails"></a>Beim Schreiben von Daten in Elasticsearch tritt ein Fehler auf.
**Erkennung**: Fangen Sie die entsprechende Ausnahme für den verwendeten [Elasticsearch-Client][elasticsearch-client] ab.  

**Wiederherstellung**:

* Verwenden Sie einen Wiederholungsmechanismus. Jeder Client verfügt über eigene Wiederholungsrichtlinien.
* Wenn die Anwendung eine reduzierte Konsistenzebene tolerieren kann, sollten Sie das Schreiben mit der `write_consistency`-Einstellung `quorum` in Betracht ziehen.

Weitere Informationen finden Sie unter [Ausführen von Elasticsearch für Azure][elasticsearch-azure].

**Diagnose**: Sie können Überwachungstools für Elasticsearch verwenden oder alle Fehler auf der Clientseite mit der Nutzlast protokollieren. Weitere Informationen finden Sie im Abschnitt „Überwachung“ in [Ausführen von Elasticsearch für Azure][elasticsearch-azure].

## <a name="queue-storage"></a>Queue Storage
### <a name="writing-a-message-to-azure-queue-storage-fails-consistently"></a>Beim Schreiben einer Nachricht in Azure Queue Storage tritt fortlaufend ein Fehler auf.
**Erkennung**: Nach *N* Wiederholungsversuchen tritt weiterhin ein Fehler beim Schreibvorgang auf.

**Wiederherstellung**:

* Speichern Sie die Daten in einem lokalen Cache, und leiten Sie die Schreibvorgänge zu einem späteren Zeitpunkt, wenn der Dienst verfügbar ist, an den Speicher weiter.
* Erstellen Sie eine sekundäre Warteschlange, und schreiben Sie in diese Warteschlange, wenn die primäre Warteschlange nicht verfügbar ist.

**Diagnose**: Verwenden Sie die [Speichermetriken][storage-metrics].

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>Die Anwendung kann eine bestimmte Nachricht aus der Warteschlange nicht verarbeiten.
**Erkennung**: Dies ist anwendungsspezifisch. Die Nachricht enthält z. B. ungültige Daten oder bei der Geschäftslogik ist ein Fehler aufgetreten.

**Wiederherstellung**:

Verschieben Sie die Nachricht in eine separate Warteschlange. Führen Sie einen separaten Prozess aus, um die Nachrichten in der Warteschlange zu untersuchen.

Erwägen Sie die Verwendung von Azure Service Bus Messaging-Warteschlangen, die zu diesem Zweck eine [Warteschlange für unzustellbare Nachrichten][sb-dead-letter-queue] bereitstellen.

> [!NOTE]
> Wenn Sie Storage-Warteschlangen mit WebJobs verwenden, stellt das WebJobs SDK eine integrierte Behandlung von nicht verarbeiteten Nachrichten bereit. Weitere Informationen finden Sie unter [Verwenden von Azure Queue Storage mit dem Webaufträge-SDK][sb-poison-message].

**Diagnose**: Verwenden Sie die Anwendungsprotokollierung.

## <a name="redis-cache"></a>Redis Cache
### <a name="reading-from-the-cache-fails"></a>Beim Lesen aus dem Cache tritt ein Fehler aus.
**Erkennung**: Fangen Sie `StackExchange.Redis.RedisConnectionException` ab.

**Wiederherstellung**:

1. Wiederholen Sie den Vorgang bei vorübergehenden Fehlern. Azure Redis Cache unterstützt die integrierte Wiederholung. Weitere Informationen finden Sie unter [Redis Cache-Wiederholungsrichtlinien][redis-retry].
2. Behandeln Sie nicht vorübergehende Fehler als Cachefehler, und führen Sie einen Fallback auf die ursprüngliche Datenquelle aus.

**Diagnose**: Verwenden Sie die [Redis Cache-Diagnose][redis-monitor].

### <a name="writing-to-the-cache-fails"></a>Beim Schreiben in den Cache tritt ein Fehler auf.
**Erkennung**: Fangen Sie `StackExchange.Redis.RedisConnectionException` ab.

**Wiederherstellung**:

1. Wiederholen Sie den Vorgang bei vorübergehenden Fehlern. Azure Redis Cache unterstützt die integrierte Wiederholung. Weitere Informationen finden Sie unter [Redis Cache-Wiederholungsrichtlinien][redis-retry].
2. Wenn der Fehler nicht vorübergehend ist, ignorieren Sie ihn und lassen andere Transaktionen später in den Cache schreiben.

**Diagnose**: Verwenden Sie die [Redis Cache-Diagnose][redis-monitor].

## <a name="sql-database"></a>SQL-Datenbank
### <a name="cannot-connect-to-the-database-in-the-primary-region"></a>Es kann keine Verbindung zur Datenbank in der primären Region hergestellt werden.
**Erkennung**: Beim Herstellen der Verbindung ist ein Fehler aufgetreten.

**Wiederherstellung**:

Voraussetzung: Die Datenbank muss für die aktive Georeplikation konfiguriert sein. Weitere Informationen finden Sie unter [Aktive Georeplikation in Azure SQL-Datenbank][sql-db-replication].

* Bei Abfragen lesen Sie aus einem sekundären Replikat.
* Für Einfüge- und Aktualisierungsvorgänge führen Sie einen manuellen Failover zu einem sekundären Replikat durch. Weitere Informationen finden Sie unter [Initiieren eines geplanten oder ungeplanten Failovers für die Azure SQL-Datenbank][sql-db-failover].

Das Replikat verwendet eine andere Verbindungszeichenfolge, sodass Sie die Verbindungszeichenfolge in Ihrer Anwendung aktualisieren müssen.

### <a name="client-runs-out-of-connections-in-the-connection-pool"></a>Die Verbindungen im Verbindungspool gehen für den Client zur Neige.
**Erkennung**: Fangen Sie `System.InvalidOperationException`-Fehler ab.

**Wiederherstellung**:

* Wiederholen Sie den Vorgang.
* Isolieren Sie als vorbeugende Maßnahme die Verbindungspools für jeden Anwendungsfall, sodass ein Anwendungsfall nicht alle Verbindungen kontrollieren kann.
* Erhöhen Sie die maximale Anzahl von Verbindungspools.

**Diagnose**: Anwendungsprotokolle.

### <a name="database-connection-limit-is-reached"></a>Der Grenzwert für Datenbankverbindungen wurde erreicht.
**Erkennung**: Azure SQL-Datenbank begrenzt die Anzahl der gleichzeitigen Worker, Anmeldungen und Sitzungen. Die Grenzwerte hängen von der Dienstschicht ab. Weitere Informationen finden Sie unter [Ressourceneinschränkungen für Azure SQL-Datenbank][sql-db-limits].

Um diese Fehler zu erkennen, fangen Sie `System.Data.SqlClient.SqlException` ab, und überprüfen Sie den Wert von `SqlException.Number` für den SQL-Fehlercode. Eine Liste der relevanten Fehlercodes finden Sie unter [SQL-Fehlercodes für SQL-Datenbank-Clientanwendungen: Datenbankverbindungsfehler und andere Probleme][sql-db-errors].

**Wiederherstellen**. Diese Fehler werden als vorübergehend betrachtet, sodass ein erneuter Versuch das Problem beheben kann. Wenn Sie diese Fehler immer wieder feststellen, sollten Sie eine Skalierung der Datenbank in Betracht ziehen.

**Diagnose**: Die [sys.event_log][sys.event_log]-Abfrage gibt erfolgreiche Datenbankverbindungen, Verbindungsfehler und Deadlocks zurück.

* Erstellen Sie eine [Warnungsregel][azure-alerts] für fehlerhafte Verbindungen.
* Aktivieren Sie die [SQL-Datenbanküberwachung][sql-db-audit], und prüfen Sie auf fehlerhafte Anmeldungen.

## <a name="service-bus-messaging"></a>Service Bus Messaging
### <a name="reading-a-message-from-a-service-bus-queue-fails"></a>Beim Lesen einer Nachricht aus einer Service Bus-Warteschlange ist ein Fehler aufgetreten.
**Erkennung**: Fangen Sie vom Client-SDK stammende Ausnahmen ab. Die Basisklasse für Service Bus-Ausnahmen ist [MessagingException][sb-messagingexception-class]. Wenn es sich um einen vorübergehenden Fehler handelt, ist die `IsTransient`-Eigenschaft „true“.

Weitere Informationen finden Sie unter [Service Bus-Messagingausnahmen][sb-messaging-exceptions].

**Wiederherstellung**:

1. Wiederholen Sie den Vorgang bei vorübergehenden Fehlern. Weitere Informationen finden Sie unter [Service Bus-Wiederholungsrichtlinien][sb-retry].
2. Nachrichten, die keinem Empfänger übermittelt werden können, befinden sich in einer *Warteschlange für unzustellbare Nachrichten*. Verwenden Sie diese Warteschlange, um zu prüfen, welche Nachrichten nicht empfangen werden konnten. Es erfolgt keine automatische Bereinigung der Warteschlange für unzustellbare Nachrichten. Nachrichten verbleiben dort, bis Sie sie explizit abrufen. Weitere Informationen finden Sie unter [Übersicht über Service Bus-Warteschlangen für unzustellbare Nachrichten][sb-dead-letter-queue].

### <a name="writing-a-message-to-a-service-bus-queue-fails"></a>Beim Schreiben einer Nachricht in eine Service Bus-Warteschlange ist ein Fehler aufgetreten.
**Erkennung**: Fangen Sie vom Client-SDK stammende Ausnahmen ab. Die Basisklasse für Service Bus-Ausnahmen ist [MessagingException][sb-messagingexception-class]. Wenn es sich um einen vorübergehenden Fehler handelt, ist die `IsTransient`-Eigenschaft „true“.

Weitere Informationen finden Sie unter [Service Bus-Messagingausnahmen][sb-messaging-exceptions].

**Wiederherstellung**:

1. Der Service Bus-Client wiederholt den Versuch nach einem vorübergehenden Fehler automatisch. Standardmäßig wird ein exponentieller Backoff verwendet. Nach der maximalen Anzahl der Wiederholungsversuche oder der maximalen Timeoutperiode löst der Client eine Ausnahme aus. Weitere Informationen finden Sie unter [Service Bus-Wiederholungsrichtlinien][sb-retry].
2. Wenn das Warteschlangenkontingent überschritten wird, löst der Client [QuotaExceededException][QuotaExceededException] aus. Weitere Details finden Sie in der Ausnahmemeldung. Entfernen Sie einige Nachrichten aus der Warteschlange, bevor Sie den Versuch wiederholen, und ziehen Sie in Betracht, das Trennschalter-Muster zu verwenden, um weitere Wiederholungsversuche zu vermeiden, während das Kontingent überschritten wurde. Stellen Sie darüber hinaus sicher, dass für die [BrokeredMessage.TimeToLive]-Eigenschaft kein zu hoher Wert festgelegt ist.
3. Innerhalb einer Region kann die Resilienz mithilfe von [partitionierten Warteschlangen oder Themen][sb-partition] verbessert werden. Eine nicht partitionierte Warteschlange bzw. ein Thema ist einem Nachrichtenspeicher zugewiesen. Wenn dieser Nachrichtenspeicher nicht verfügbar ist, treten für alle Vorgänge der Warteschlange oder des Themas Fehler auf. Eine partitionierte Warteschlange bzw. ein Thema ist über mehrere Nachrichtenspeicher hinweg partitioniert.
4. Erstellen Sie zur zusätzlichen Resilienz zwei Service Bus-Namespaces in verschiedenen Regionen, und replizieren Sie die Nachrichten. Sie können entweder die aktive oder die passive Replikation verwenden.

   * Aktive Replikation: Der Client sendet alle Nachrichten an beide Warteschlangen. Der Empfänger lauscht an beiden Warteschlangen. Kennzeichnen Sie Nachrichten mit einem eindeutigen Bezeichner, damit der Client doppelte Nachrichten verwerfen kann.
   * Passive Replikation: Der Client sendet die Nachricht an eine Warteschlange. Wenn ein Fehler aufgetreten ist, führt der Client einen Fallback zur anderen Warteschlange aus. Der Empfänger lauscht an beiden Warteschlangen. Dieser Ansatz reduziert die Anzahl doppelter Nachrichten, die gesendet werden. Der Empfänger muss jedoch weiterhin doppelte Nachrichten verarbeiten.

     Weitere Informationen finden Sie unter [GeoReplication-Beispiel][sb-georeplication-sample] und [Bewährte Methoden zum Schützen von Anwendungen vor Service Bus-Ausfällen und Notfällen](/azure/service-bus-messaging/service-bus-outages-disasters/).

### <a name="duplicate-message"></a>Doppelt vorhandene Nachricht.
**Erkennung**: Überprüfen Sie die Eigenschaften `MessageId` und `DeliveryCount` der Nachricht.

**Wiederherstellung**:

* Gestalten Sie Ihre Verarbeitungsvorgänge für Nachrichten nach Möglichkeit idempotent. Andernfalls speichern Sie Nachrichten-IDs von bereits verarbeiteten Nachrichten, und überprüfen Sie die ID vor der Verarbeitung einer Nachricht.
* Aktivieren Sie die Duplikaterkennung, indem Sie beim Erstellen der Warteschlange `RequiresDuplicateDetection` auf „true“ festlegen. Mit dieser Einstellung löscht Service Bus automatisch alle Nachrichten, die mit derselben `MessageId` wie eine vorherige Nachricht gesendet werden.  Beachten Sie Folgendes:

  * Diese Einstellung verhindert, dass doppelte Nachrichten in die Warteschlange gestellt werden. Sie hindert einen Empfänger nicht daran, dieselbe Nachricht mehrmals zu verarbeiten.
  * Die Duplikaterkennung verfügt über ein Zeitfenster. Wenn dieses Zeitfenster beim Senden eines Duplikats überschritten wird, kann das Duplikat nicht erkannt werden.

**Diagnose**: Protokollieren Sie doppelte Nachrichten.

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>Die Anwendung kann eine bestimmte Nachricht aus der Warteschlange nicht verarbeiten.
**Erkennung**: Dies ist anwendungsspezifisch. Die Nachricht enthält z. B. ungültige Daten oder bei der Geschäftslogik ist ein Fehler aufgetreten.

**Wiederherstellung**:

Zwei Fehlermodi müssen berücksichtigt werden.

* Der Empfänger erkennt den Fehler. In diesem Fall verschieben Sie die Nachricht in die Warteschlange für unzustellbare Nachrichten. Führen Sie später einen separaten Prozess aus, um die Nachrichten in der Warteschlange für unzustellbare Nachrichten zu untersuchen.
* Beim Empfänger tritt während der Verarbeitung der Nachricht ein Fehler auf, z. B. ein Ausnahmefehler. Verwenden Sie den Modus `PeekLock`, um diesen Fall zu behandeln. Wenn in diesem Modus die Sperre abläuft, wird die Nachricht für andere Empfänger verfügbar. Wenn die Nachricht die maximale Anzahl der Zustellungen oder die Gültigkeitsdauer überschreitet, wird die Nachricht automatisch in die Warteschlange für unzustellbare Nachrichten verschoben.

Weitere Informationen finden Sie unter [Übersicht über Service Bus-Warteschlangen für unzustellbare Nachrichten][sb-dead-letter-queue].

**Diagnose**: Sobald die Anwendung eine Nachricht in die Warteschlange für unzustellbare Nachrichten verschiebt, schreiben Sie ein Ereignis in die Anwendungsprotokolle.

## <a name="service-fabric"></a>Service Fabric
### <a name="a-request-to-a-service-fails"></a>Bei der Anforderung eines Diensts ist ein Fehler aufgetreten.
**Erkennung**: Der Dienst gibt einen Fehler zurück.

**Wiederherstellung**:

* Suchen Sie erneut einen Proxy (`ServiceProxy` oder `ActorProxy`), und rufen Sie die Dienst-/Actor-Methode erneut auf.
* **Zustandsbehafteter Dienst**: Umschließen Sie zuverlässige Sammlungen in einer Transaktion. Wenn ein Fehler aufgetreten ist, wird für die Transaktion ein Rollback ausgeführt. Die Anforderung wird erneut verarbeitet, wenn sie per Pullvorgang aus einer Warteschlange abgerufen wird.
* **Zustandsloser Dienst**: Wenn der Dienst Daten in einem externen Speicher speichert, müssen alle Vorgänge idempotent sein.

**Diagnose**: Anwendungsprotokoll

### <a name="service-fabric-node-is-shut-down"></a>Der Service Fabric-Knoten wird heruntergefahren.
**Erkennung**: Es wird ein Abbruchtoken an die `RunAsync`-Methode des Diensts übergeben. Service Fabric bricht die Aufgabe vor dem Herunterfahren des Knotens ab.

**Wiederherstellen**. Verwenden Sie das Abbruchtoken, um das Herunterfahren zu erkennen. Wenn der Abbruch von Service Fabric angefordert wird, beenden Sie sämtliche Arbeit und dann `RunAsync` so schnell wie möglich.

**Diagnose**: Anwendungsprotokolle

## <a name="storage"></a>Speicher
### <a name="writing-data-to-azure-storage-fails"></a>Beim Schreiben von Daten in Azure Storage ist ein Fehler aufgetreten.
**Erkennung**: Der Client erhält beim Schreiben entsprechende Fehlermeldungen.

**Wiederherstellung**:

1. Versuchen Sie den Vorgang erneut, um die Wiederherstellung nach vorübergehenden Fehlern zu erreichen. Die [Wiederholungsrichtlinie][Storage.RetryPolicies] im Client-SDK verarbeitet dies automatisch.
2. Implementieren Sie das Trennschalter-Muster, um eine Überlastung des Speichers zu vermeiden.
3. Wenn bei N Wiederholungsversuchen Fehler auftreten, führen Sie einen ordnungsgemäßen Fallback aus. Beispiel: 

   * Speichern Sie die Daten in einem lokalen Cache, und leiten Sie die Schreibvorgänge zu einem späteren Zeitpunkt, wenn der Dienst verfügbar ist, an den Speicher weiter.
   * Wenn der Schreibvorgang in einem Transaktionsbereich erfolgt ist, kompensieren Sie die Transaktion.

**Diagnose**: Verwenden Sie die [Speichermetriken][storage-metrics].

### <a name="reading-data-from-azure-storage-fails"></a>Beim Lesen von Daten aus Azure Storage tritt ein Fehler auf.
**Erkennung**: Der Client erhält beim Lesen entsprechende Fehlermeldungen.

**Wiederherstellung**:

1. Versuchen Sie den Vorgang erneut, um die Wiederherstellung nach vorübergehenden Fehlern zu erreichen. Die [Wiederholungsrichtlinie][Storage.RetryPolicies] im Client-SDK verarbeitet dies automatisch.
2. Wenn für RA-GRS-Speicher beim Lesen aus dem primären Endpunkt ein Fehler auftritt, versuchen Sie den Lesevorgang mit einem sekundären Endpunkt. Das Client-SDK kann dies automatisch verarbeiten. Weitere Informationen finden Sie unter [Azure Storage-Replikation][storage-replication].
3. Wenn bei *N* Wiederholungsversuchen Fehler auftreten, führen Sie eine Fallbackaktion zum ordnungsgemäßen Herabstufen durch. Wenn z. B. ein Produktbild nicht aus dem Speicher abgerufen werden kann, zeigen Sie ein allgemeines Platzhalterbild an.

**Diagnose**: Verwenden Sie die [Speichermetriken][storage-metrics].

## <a name="virtual-machine"></a>Virtual Machine
### <a name="connection-to-a-backend-vm-fails"></a>Beim Herstellen der Verbindung mit einer Back-End-VM ist ein Fehler aufgetreten.
**Erkennung**: Netzwerkverbindungsfehler.

**Wiederherstellung**:

* Stellen Sie mindestens zwei Back-End-VMs in einer Verfügbarkeitsgruppe hinter einem Lastenausgleich bereit.
* Wenn der Verbindungsfehler vorübergehend ist, wird TCP manchmal erfolgreich versuchen, die Nachricht erneut zu senden.
* Implementieren Sie eine Wiederholungsrichtlinie in der Anwendung.
* Für persistente oder nicht vorübergehende Fehler implementieren Sie das [Trennschalter][circuit-breaker]-Muster.
* Überschreitet die aufrufende VM den Ausgangsgrenzwert für das Netzwerk, wird die ausgehende Warteschlange gefüllt. Wenn die ausgehende Warteschlange beständig voll ist, sollten Sie eine horizontale Skalierung in Betracht ziehen.

**Diagnose**: Protokollieren Sie Ereignisse an den Dienstgrenzen.

### <a name="vm-instance-becomes-unavailable-or-unhealthy"></a>Eine VM-Instanz steht nicht mehr zur Verfügung oder ist fehlerhaft.
**Erkennung**: Konfigurieren Sie einen Load Balancer-[Integritätstest][lb-probe], der angibt, ob die VM-Instanz fehlerfrei ist. Der Test sollte überprüfen, ob wichtige Funktionen ordnungsgemäß reagieren.

**Wiederherstellen**. Stellen Sie für jede Anwendungsschicht mehrere VM-Instanzen in dieselbe Verfügbarkeitsgruppe, und platzieren Sie einen Lastenausgleich vor den virtuellen Computern. Wenn beim Integritätstest ein Fehler auftritt, beendet der Load Balancer das Senden neuer Verbindungen an die fehlerhafte Instanz.

**Diagnose**: Verwenden Sie die Load Balancer-[Protokollanalysen][lb-monitor].

* Konfigurieren Sie Ihr Überwachungssystem so, dass es alle Endpunkte der Integritätsüberwachung überwacht.

### <a name="operator-accidentally-shuts-down-a-vm"></a>Der Operator fährt versehentlich einen virtuellen Computer herunter.
**Erkennung**: N/V

**Wiederherstellen**. Legen Sie eine Ressourcensperre mit der Stufe `ReadOnly` fest. Weitere Informationen finden Sie unter [Sperren von Ressourcen mit Azure Resource Manager][rm-locks].

**Diagnose**: Verwenden Sie [Azure-Aktivitätsprotokolle][azure-activity-logs].

## <a name="webjobs"></a>WebJobs
### <a name="continuous-job-stops-running-when-the-scm-host-is-idle"></a>Wenn der SCM-Host im Leerlauf ist, wird der fortlaufende Auftrag abgebrochen.
**Erkennung**: Übergeben Sie ein Abbruchtoken an die WebJob-Funktion. Weitere Informationen finden Sie unter [Ordnungsgemäßes Herunterfahren][web-jobs-shutdown].

**Wiederherstellen**. Aktivieren Sie die Einstellung `Always On` in der Web-App. Weitere Informationen finden Sie unter [Ausführen von Hintergrundaufgaben mit WebJobs][web-jobs].

## <a name="application-design"></a>Anwendungsentwurf
### <a name="application-cant-handle-a-spike-in-incoming-requests"></a>Die Anwendung kann nicht mit einer Steigerung der eingehenden Anforderungen zurechtkommen.
**Erkennung**: Dies hängt von der Anwendung ab. Typische Symptome:

* Die Website beginnt mit der Rückgabe von „HTTP 5xx“-Fehlercodes.
* Abhängige Dienste wie der Datenbank- oder Speicherdienst beginnen damit, Anforderungen zu beschränken. Suchen Sie in Abhängigkeit vom Dienst nach HTTP-Fehlern wie HTTP 429 (Zu viele Anforderungen).
* Die Länge der HTTP-Warteschlange nimmt zu.

**Wiederherstellung**:

* Führen Sie eine horizontale Skalierung durch, um die erhöhte Workload zu bewältigen.
* Reduzieren Sie Fehler, um zu vermeiden, dass sich überlappende Fehler die gesamte Anwendung stören. Mögliche Lösungsstrategien:

  * Implementieren Sie das [Drosselungsmuster][throttling-pattern], um eine Überlastung der Back-End-Systeme zu vermeiden.
  * Verwenden Sie den [warteschlangenbasierten Lastenausgleich][queue-based-load-leveling], um Anforderungen zu puffern und mit angemessener Geschwindigkeit zu verarbeiten.
  * Priorisieren Sie bestimmte Clients. Wenn die Anwendung z. B. über kostenlose und kostenpflichtige Tarife verfügt, beschränken Sie Kunden mit kostenlosen Tarifen, aber nicht Kunden mit kostenpflichtigen Tarifen. Weitere Informationen finden Sie unter [Muster „Prioritätswarteschlange“][priority-queue-pattern].

**Diagnose**: Verwenden Sie die [App Service-Diagnoseprotokollierung][app-service-logging]. Verwenden Sie einen Dienst wie [Azure Log Analytics][azure-log-analytics], [Application Insights][app-insights] oder [New Relic][new-relic], um die Diagnoseprotokolle besser zu verstehen.

### <a name="one-of-the-operations-in-a-workflow-or-distributed-transaction-fails"></a>Bei einem der Vorgänge in einem Workflow oder in einer verteilten Transaktion ist ein Fehler aufgetreten.
**Erkennung**: Nach *N* Wiederholungsversuchen tritt der Fehler weiterhin auf.

**Wiederherstellung**:

* Implementieren Sie als vorbeugende Maßnahme das Muster [Scheduler-Agent-Supervisor][scheduler-agent-supervisor], um den gesamten Workflow zu verwalten.
* Wiederholen Sie den Versuch nicht bei Timeouts. Die Erfolgsquote für diesen Fehler ist gering.
* Verschieben Sie die Arbeit in eine Warteschlange, um es zu einem späteren erneut zu versuchen.

**Diagnose**: Protokollieren Sie alle Vorgänge (erfolgreiche und fehlerhafte), einschließlich der Ausgleichsmaßnahmen. Verwenden Sie Korrelations-IDs, damit Sie alle Vorgänge innerhalb einer Transaktion verfolgen können.

### <a name="a-call-to-a-remote-service-fails"></a>Beim Aufruf eines Remotediensts ist ein Fehler aufgetreten.
**Erkennung**: HTTP-Fehlercode.

**Wiederherstellung**:

1. Wiederholen Sie den Vorgang bei vorübergehenden Fehlern.
2. Wenn der Aufruf nach *N* Versuchen Fehler aufweist, führen Sie einen Fallback aus. (Dies ist anwendungsspezifisch.)
3. Implementieren Sie das [Trennschalter-Muster][circuit-breaker], um überlappende Fehler zu vermeiden.

**Diagnose**: Protokollieren Sie alle Fehler von Remoteaufrufen.

## <a name="next-steps"></a>Nächste Schritte
Weitere Informationen zur Fehlermodusanalyse finden Sie unter [Resilience by design for cloud services][resilience-by-design-pdf] (Designorientierte Resilienz für Clouddienste, PDF-Download).

<!-- links -->

[api-management]: https://azure.microsoft.com/documentation/services/api-management/
[api-management-throttling]: /azure/api-management/api-management-sample-flexible-throttling/
[app-insights]: /azure/application-insights/app-insights-overview/
[app-insights-web-apps]: /azure/application-insights/app-insights-azure-web-apps/
[app-service-configure]: /azure/app-service-web/web-sites-configure/
[app-service-logging]: /azure/app-service-web/web-sites-enable-diagnostic-log/
[app-service-slots]: /azure/app-service-web/web-sites-staged-publishing/
[auto-rest-client-retry]: https://github.com/Azure/autorest/tree/master/docs
[azure-activity-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs/
[azure-alerts]: /azure/monitoring-and-diagnostics/insights-alerts-portal/
[azure-log-analytics]: /azure/log-analytics/log-analytics-overview/
[BrokeredMessage.TimeToLive]: https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx
[cassandra-error-handling]: http://www.datastax.com/dev/blog/cassandra-error-handling-done-right
[circuit-breaker]: https://msdn.microsoft.com/library/dn589784.aspx
[cosmosdb-multi-region]: /azure/cosmos-db/tutorial-global-distribution-sql-api
[elasticsearch-azure]: ../elasticsearch/index.md
[elasticsearch-client]: https://www.elastic.co/guide/en/elasticsearch/client/index.html
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[onstop-events]: https://azure.microsoft.com/blog/the-right-way-to-handle-azure-onstop-events/
[lb-monitor]: /azure/load-balancer/load-balancer-monitor-log/
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview/#learn-about-the-types-of-probes
[new-relic]: https://newrelic.com/
[priority-queue-pattern]: https://msdn.microsoft.com/library/dn589794.aspx
[queue-based-load-leveling]: https://msdn.microsoft.com/library/dn589783.aspx
[QuotaExceededException]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx
[ra-web-apps-basic]: ../reference-architectures/app-service-web-app/basic-web-app.md
[redis-monitor]: /azure/redis-cache/cache-how-to-monitor/
[redis-retry]: ../best-practices/retry-service-specific.md#azure-redis-cache
[resilience-by-design-pdf]: http://download.microsoft.com/download/D/8/C/D8C599A4-4E8A-49BF-80EE-FE35F49B914D/Resilience_by_Design_for_Cloud_Services_White_Paper.pdf
[RoleEntryPoint.OnStop]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstop.aspx
[RoleEnvironment.Stopping]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.stopping.aspx
[rm-locks]: /azure/azure-resource-manager/resource-group-lock-resources/
[sb-dead-letter-queue]: /azure/service-bus-messaging/service-bus-dead-letter-queues/
[sb-georeplication-sample]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/GeoReplication
[sb-messagingexception-class]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.messagingexception.aspx
[sb-messaging-exceptions]: /azure/service-bus-messaging/service-bus-messaging-exceptions/
[sb-outages]: /azure/service-bus-messaging/service-bus-outages-disasters/#protecting-queues-and-topics-against-datacenter-outages-or-disasters
[sb-partition]: /azure/service-bus-messaging/service-bus-partitioning/
[sb-poison-message]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#poison
[sb-retry]: ../best-practices/retry-service-specific.md#service-bus
[search-sdk]: https://msdn.microsoft.com/library/dn951165.aspx
[scheduler-agent-supervisor]: https://msdn.microsoft.com/library/dn589780.aspx
[search-analytics]: /azure/search/search-traffic-analytics/
[sql-db-audit]: /azure/sql-database/sql-database-auditing-get-started/
[sql-db-errors]: /azure/sql-database/sql-database-develop-error-messages/#resource-governance-errors
[sql-db-failover]: /azure/sql-database/sql-database-geo-replication-failover-portal/
[sql-db-limits]: /azure/sql-database/sql-database-resource-limits/
[sql-db-replication]: /azure/sql-database/sql-database-geo-replication-overview/
[storage-metrics]: https://msdn.microsoft.com/library/dn782843.aspx
[storage-replication]: /azure/storage/storage-redundancy/
[Storage.RetryPolicies]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.aspx
[sys.event_log]: https://msdn.microsoft.com/library/dn270018.aspx
[throttling-pattern]: https://msdn.microsoft.com/library/dn589798.aspx
[web-jobs]: /azure/app-service-web/web-sites-create-web-jobs/
[web-jobs-shutdown]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#graceful
