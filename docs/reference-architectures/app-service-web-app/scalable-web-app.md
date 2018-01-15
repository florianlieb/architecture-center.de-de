---
title: Skalierbare Webanwendung
description: "Verbessern der Skalierbarkeit in einer Webanwendung, die in Microsoft Azure ausgeführt wird."
author: MikeWasson
pnp.series.title: Azure App Service
pnp.series.prev: basic-web-app
pnp.series.next: multi-region-web-app
ms.date: 11/23/2016
cardTitle: Improve scalability
ms.openlocfilehash: 1fdaf6e3695cb814fa4c275a4a273f9fa9a7b71b
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/08/2018
---
# <a name="improve-scalability-in-a-web-application"></a>Verbessern der Skalierbarkeit in einer Webanwendung

Diese Referenzarchitektur zeigt bewährte Methoden zum Verbessern der Skalierbarkeit und Leistung in einer Azure App Service-Webanwendung.

![[0]][0]

*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*

## <a name="architecture"></a>Architecture  

Diese Architektur basiert auf der unter [Einfache Webanwendung][basic-web-app] gezeigten Architektur. Sie enthält die folgenden Komponenten:

* **Ressourcengruppe**. Eine [Ressourcengruppe][resource-group] ist ein logischer Container für Azure-Ressourcen.
* **[Web-App][app-service-web-app]** und **[API-App][app-service-api-app]**. Eine typische moderne Anwendung kann sowohl eine Website als auch ein oder mehrere RESTful-Web-APIs enthalten. Eine Web-API kann von Browserclients über AJAX, von systemeigenen Clientanwendungen oder von serverseitigen Anwendungen genutzt werden. Hinweise zum Entwerfen von Web-APIs finden Sie unter [Leitfaden zum API-Design][api-guidance].    
* **WebJob**. Verwenden Sie [Azure WebJobs][webjobs] zum Ausführen lang andauernder Aufgaben im Hintergrund. WebJobs können nach einem Zeitplan, fortlaufend oder als Reaktion auf einen Trigger, z. B. das Ablegen einer Nachricht in einer Warteschlange, ausgeführt werden. Ein WebJob wird als Hintergrundprozess im Kontext einer App Service-App ausgeführt.
* **Warteschlange**. Bei der hier gezeigten Architektur setzt die Anwendung Hintergrundaufgaben in eine Warteschlange, indem eine Nachricht in einer [Azure Queue Storage][queue-storage]-Warteschlange abgelegt wird. Die Nachricht löst eine Funktion im WebJob aus. Alternativ können Sie Service Bus-Warteschlangen verwenden. Einen Vergleich finden Sie unter [Azure- und Service Bus-Warteschlangen – Vergleich und Gegenüberstellung][queues-compared].
* **Cache**. Speichern Sie semistatische Daten in [Azure Redis Cache][azure-redis].  
* **CDN**. Verwenden Sie [Azure Content Delivery Network][azure-cdn] (CDN) zum Zwischenspeichern öffentlich verfügbarer Inhalte für geringere Latenz und schnellere Bereitstellung der Inhalte.
* **Datenspeicher**. Verwenden Sie eine [Azure SQL-Datenbank][sql-db] für relationale Daten. Für nicht relationale Daten sollten Sie einen NoSQL-Speicher, z. B. [Cosmos DB][documentdb], in Betracht ziehen.
* **Azure Search**. Verwenden Sie [Azure Search][azure-search] zum Hinzufügen von Suchfunktionalität wie z. B. Suchvorschläge, Fuzzysuche und sprachspezifische Suchen. Azure Search wird normalerweise in Verbindung mit einem anderen Datenspeicher verwendet, insbesondere dann, wenn der primäre Datenspeicher strikte Konsistenz erfordert. Bei dieser Vorgehensweise speichern Sie autorisierende Daten in dem anderen Datenspeicher und den Suchindex in Azure Search. Azure Search kann auch zum Konsolidieren eines einzelnen Suchindex aus mehreren Datenspeichern verwendet werden.  
* **E-Mail/SMS**. Verwenden Sie einen Drittanbieter-Dienst, z. B. SendGrid oder Twilio, zum Senden von E-Mail- oder SMS-Nachrichten, statt diese Funktionalität direkt in die Anwendung zu integrieren.
* **Azure DNS:** [Azure DNS][azure-dns] ist ein Hostingdienst für DNS-Domänen, der die Namensauflösung unter Verwendung der Microsoft Azure-Infrastruktur durchführt. Durch das Hosten Ihrer Domänen in Azure können Sie Ihre DNS-Einträge mithilfe der gleichen Anmeldeinformationen, APIs, Tools und Abrechnung wie für die anderen Azure-Dienste verwalten.

## <a name="recommendations"></a>Empfehlungen

Ihre Anforderungen können von der hier beschriebenen Architektur abweichen. Verwenden Sie die Empfehlungen in diesem Abschnitt als Ausgangspunkt.

### <a name="app-service-apps"></a>App Service-Apps
Es wird empfohlen, die Webanwendung und die Web-API als separate App Service-Apps zu erstellen. Dieser Entwurf ermöglicht Ihnen die Ausführung in separaten App Service-Plänen, sodass sie unabhängig voneinander skaliert werden können. Wenn Sie diesen Grad an Skalierbarkeit anfänglich nicht benötigen, können Sie die Apps in demselben Plan bereitstellen und sie später bei Bedarf in separate Pläne verschieben.

> [!NOTE]
> Beim Basic-, Standard- und Premium-Plan werden Ihnen die VM-Instanzen im Plan und nicht pro App in Rechnung gestellt. Informationen finden Sie unter [App Service – Preise][app-service-pricing].
> 
> 

Wenn Sie beabsichtigen, die Features *Einfache Tabellen* oder *Einfache APIs* der Mobile App Service-Apps zu verwenden, erstellen Sie für diesen Zweck eine separate App Service-App.  Diese Features benötigen ein bestimmtes Anwendungsframework, um sie zu aktivieren.

### <a name="webjobs"></a>WebJobs
Erwägen Sie die Bereitstellung ressourcenintensiver WebJobs in einer leeren App Service-App im Rahmen eines separaten App Service-Plans. Dadurch werden dedizierte Instanzen für den WebJob bereitgestellt. Informationen finden Sie unter [Anleitungen für Hintergrundaufträge][webjobs-guidance].  

### <a name="cache"></a>Cache
Sie können die Leistung und Skalierbarkeit verbessern, indem Sie [Azure Redis Cache][azure-redis] zum Zwischenspeichern einiger Daten verwenden. Ziehen Sie die Verwendung von Redis Cache für Folgendes in Betracht:

* Semistatische Transaktionsdaten
* Sitzungszustand
* HTML-Ausgabe Dies kann in Anwendungen nützlich sein, die eine komplexe HTML-Ausgabe rendern.

Ausführlichere Anweisungen zum Entwerfen einer Cachingstrategie finden Sie unter [Anleitungen zum Caching][caching-guidance].

### <a name="cdn"></a>CDN
Verwenden Sie [Azure CDN][azure-cdn] zum Zwischenspeichern statischer Inhalte. Der wichtigste Vorteil eines CDN ist die verringerte Latenz für Benutzer, da Inhalte auf einem Edgeserver zwischengespeichert werden, der sich in geografischer Nähe zum Benutzer befindet. CDN kann auch die Auslastung der Anwendung verringern, da dieser Datenverkehr nicht von der Anwendung gehandhabt wird.

Wenn Ihre App größtenteils aus statischen Seiten besteht, ziehen Sie die Verwendung von [CDN zum Zwischenspeichern der gesamten App][cdn-app-service] in Betracht. Andernfalls speichern Sie statische Inhalte wie Bilder, CSS und HTML-Dateien in [Azure Storage, und verwenden Sie CDN zum Zwischenspeichern dieser Dateien][cdn-storage-account].

> [!NOTE]
> Azure CDN kann keine Inhalte bereitstellen, die eine Authentifizierung erfordern.
> 
> 

Ausführlichere Anleitungen finden Sie unter [Anleitungen zum Content Delivery Network (CDN)][cdn-guidance].

### <a name="storage"></a>Speicher
Moderne Anwendungen verarbeiten häufig große Datenmengen. Für eine Skalierung für die Cloud ist es wichtig, den richtigen Speichertyp auszuwählen. Hier sind einige grundlegende Empfehlungen dafür. 

| Zu speichernde Objekte | Beispiel | Empfohlener Speicher |
| --- | --- | --- |
| Dateien |Bilder, Dokumente, PDF-Dateien |Azure Blob Storage |
| Schlüssel/Wert-Paare |Nach Benutzer-ID gesuchte Benutzerprofildaten |Azure Table Storage |
| Kurze Nachrichten zum Auslösen der Weiterverarbeitung |Bestellanforderungen |Azure Queue-Speicher, Service Bus-Warteschlange oder Service Bus-Thema |
| Nicht relationale Daten mit einem flexiblen Schema, die grundlegende Abfragen erfordern |Produktkatalog |Dokumentdatenbank, z. B. Azure Cosmos DB, MongoDB oder Apache CouchDB |
| Relationale Daten, die eine umfassendere Abfrageunterstützung, ein striktes Schema und/oder starke Konsistenz erfordern |Produktbestand |Azure SQL-Datenbank |

## <a name="scalability-considerations"></a>Überlegungen zur Skalierbarkeit

Ein großer Vorteil von Azure App Service ist die Möglichkeit, Ihre Anwendung abhängig von der Last zu skalieren. Hier sind einige Punkte aufgeführt, die beim Planen der Skalierung für Ihre Anwendung zu bedenken sind.

### <a name="app-service-app"></a>App Service-App
Wenn Ihre Lösung mehrere App Service-Apps enthält, sollten Sie deren Bereitstellung in separaten App Service-Plänen in Betracht ziehen. Dieser Ansatz ermöglicht es Ihnen, diese unabhängig voneinander zu skalieren, da sie auf separaten Instanzen ausgeführt werden. 

Ziehen Sie ebenso in Erwägung, einen WebJob in einem eigenen Plan abzulegen, damit Hintergrundaufgaben nicht auf denselben Instanzen ausgeführt werden, die auch HTTP-Anforderungen verarbeiten.  

### <a name="sql-database"></a>SQL-Datenbank
Erhöhen Sie die Skalierbarkeit einer SQL-Datenbank durch *Sharding* der Datenbank. Sharding bezeichnet ein horizontales Partitionieren der Datenbank. Durch Sharding können Sie die Datenbank mithilfe von [Tools für elastische Datenbanken][sql-elastic] horizontal skalieren. Sharding kann unter anderem folgende Vorteile bieten:

- Besserer Transaktionsdurchsatz
- Abfragen können schneller für eine Teilmenge der Daten ausgeführt werden

### <a name="azure-search"></a>Azure Search
Azure Search erspart den Aufwand komplexer Datensuchen aus dem primären Datenspeicher und ermöglicht eine Skalierung zur Handhabung von Lasten. Weitere Informationen finden Sie unter [Skalieren von Ressourcenebenen für Abfrage und Indizierung von Arbeitslasten in Azure Search][azure-search-scaling].

## <a name="security-considerations"></a>Sicherheitshinweise
Dieser Abschnitt enthält Sicherheitshinweise, die für die in diesem Artikel beschriebenen Azure-Dienste spezifisch sind. Es handelt sich nicht um eine vollständige Liste der bewährten Sicherheitsmethoden. Weitere Sicherheitshinweise finden Sie unter [Schützen einer App in Azure App Service][app-service-security].

### <a name="cross-origin-resource-sharing-cors"></a>Ressourcenfreigabe zwischen verschiedenen Ursprüngen (CORS)
Wenn Sie eine Website und Web-API als separate Apps erstellen, kann die Website keine clientseitigen AJAX-Aufrufe an die API vornehmen, sofern Sie CORS nicht aktivieren.

> [!NOTE]
> Die Browsersicherheit verhindert, dass eine Webseite AJAX-Anforderungen an eine andere Domäne richtet. Diese Einschränkung wird als Richtlinie des gleichen Ursprungs bezeichnet und verhindert, dass eine schädliche Website sensible Daten von einer anderen Website liest. CORS ist ein W3C-Standard, der einem Server eine weniger strenge Anwendung der Richtlinie des gleichen Ursprungs ermöglicht und einige Anforderungen zwischen verschiedenen Ursprüngen zulässt, während andere abgelehnt werden.
> 
> 

App Services verfügt über integrierte Unterstützung für CORS, ohne dass Anwendungscode geschrieben werden muss. Informationen finden Sie unter [Nutzen einer API-App aus JavaScript mit CORS][cors]. Fügen Sie die Website zur Liste der zulässigen Ursprünge für die API hinzu.

### <a name="sql-database-encryption"></a>Verschlüsselung in der SQL-Datenbank
Verwenden Sie [Transparent Data Encryption][sql-encryption], wenn in der Datenbank ruhende Daten verschlüsselt werden sollen. Dieses Feature führt eine Ver- und Entschlüsselung einer gesamten Datenbank (einschließlich Sicherungen und Transaktionsprotokolldateien) in Echtzeit durch und erfordert keine Änderungen an der Anwendung. Die Verschlüsselung führt zu höherer Latenz, und es empfiehlt sich daher, die Daten zu trennen, die in einer eigenen Datenbank gesichert werden müssen, und die Verschlüsselung nur für diese Datenbank zu aktiveren.  
  

<!-- links -->

[api-guidance]: ../../best-practices/api-design.md
[app-service-security]: /azure/app-service-web/web-sites-security
[app-service-web-app]: /azure/app-service-web/app-service-web-overview
[app-service-api-app]: /azure/app-service-api/app-service-api-apps-why-best-platform
[app-service-pricing]: https://azure.microsoft.com/pricing/details/app-service/
[azure-cdn]: https://azure.microsoft.com/services/cdn/
[azure-dns]: /azure/dns/dns-overview
[azure-redis]: https://azure.microsoft.com/services/cache/
[azure-search]: https://azure.microsoft.com/documentation/services/search/
[azure-search-scaling]: /azure/search/search-capacity-planning
[background-jobs]: ../../best-practices/background-jobs.md
[basic-web-app]: basic-web-app.md
[basic-web-app-scalability]: basic-web-app.md#scalability-considerations
[caching-guidance]: ../../best-practices/caching.md
[cdn-app-service]: /azure/app-service-web/cdn-websites-with-cdn
[cdn-storage-account]: /azure/cdn/cdn-create-a-storage-account-with-cdn
[cdn-guidance]: ../../best-practices/cdn.md
[cors]: /azure/app-service-api/app-service-api-cors-consume-javascript
[documentdb]: https://azure.microsoft.com/documentation/services/documentdb/
[queue-storage]: /azure/storage/storage-dotnet-how-to-use-queues
[queues-compared]: /azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted
[resource-group]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-elastic]: /azure/sql-database/sql-database-elastic-scale-introduction
[sql-encryption]: https://msdn.microsoft.com/library/dn948096.aspx
[tm]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.azureedge.net/cdn/app-service-reference-architectures.vsdx
[web-app-multi-region]: ./multi-region.md
[webjobs-guidance]: ../../best-practices/background-jobs.md
[webjobs]: /azure/app-service/app-service-webjobs-readme
[0]: ./images/scalable-web-app.png "Webanwendung in Azure mit verbesserter Skalierbarkeit"
