---
title: Einfache Webanwendung
description: "Empfohlene Architektur für eine einfache Webanwendung, die in Microsoft Azure ausgeführt wird."
author: MikeWasson
ms.date: 11/23/2016
cardTitle: Basic web application
ms.openlocfilehash: b7475c4087a184bb7608d0c45ffecee912c920d7
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="basic-web-application"></a>Einfache Webanwendung
[!INCLUDE [header](../../_includes/header.md)]

Diese Referenzarchitektur demonstriert eine Reihe bewährter Verfahren für eine Webanwendung, die [Azure App Service][app-service] und [Azure SQL-Datenbank][sql-db] verwendet. [**Stellen Sie diese Lösung bereit**.](#deploy-the-solution)

![[0]][0]

*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*

## <a name="architecture"></a>Architektur 

> [!NOTE]
> Der Schwerpunkt liegt bei dieser Architektur nicht auf der Anwendungsentwicklung, und es wird nicht von einem bestimmten Anwendungsframework ausgegangen. Das Ziel besteht darin, zu verstehen, wie verschiedene Azure-Dienste ineinandergreifen.
>
>

Die Architektur besteht aus den folgenden Komponenten:

* **Ressourcengruppe**. Eine [Ressourcengruppe](/azure/azure-resource-manager/resource-group-overview) ist ein logischer Container für Azure-Ressourcen.
* **App Service-App**. [Azure App Service][app-service] ist eine umfassend verwaltete Plattform zum Erstellen und Bereitstellen von Cloudanwendungen.     
* **App Service-Plan**. Ein [App Service-Plan][app-service-plans] dient zur Bereitstellung der verwalteten virtuellen Computer (VMs), auf denen Ihre App gehostet wird. Alle einem Plan zugeordneten Apps werden auf den gleichen VM-Instanzen ausgeführt.

* **Bereitstellungsslots**.  Ein [Bereitstellungsslot][deployment-slots] ermöglicht Ihnen das Staging einer Bereitstellung und ihren Austausch gegen die Produktionsbereitstellung. Auf diese Weise vermeiden Sie die direkte Bereitstellung in der Produktionsumgebung. Im Abschnitt [Verwaltbarkeit](#manageability-considerations) finden Sie spezifische Empfehlungen.

* **IP-Adresse**. Die App Service-App verfügt über eine öffentliche IP-Adresse und einen Domänennamen. Der Domänenname ist eine Unterdomäne von `azurewebsites.net`, z.B. `contoso.azurewebsites.net`. Um einen benutzerdefinierten Domänennamen zu verwenden, z.B. `contoso.com`, erstellen Sie DNS-Einträge (Domain Name Service), die der IP-Adresse den benutzerdefinierten Domänennamen zuordnen. Weitere Informationen finden Sie unter [Konfigurieren eines benutzerdefinierten Domänennamens in Azure App Service][custom-domain-name].
* **Azure SQL-Datenbank**. [SQL-Datenbank][sql-db] ist eine relationale Datenbank-as-a-Service in der Cloud.
* **Logischer Server**. In Azure SQL-Datenbank werden Ihre Datenbanken auf einem logischen Server gehostet. Sie können pro logischem Server mehrere Datenbanken erstellen.
* **Azure Storage**. Erstellen Sie ein Azure-Speicherkonto mit einem Blob-Container zum Speichern von Diagnoseprotokollen.
* **Azure Active Directory** (Azure AD). Verwenden Sie Azure AD oder einen anderen Identitätsanbieter für die Authentifizierung.

## <a name="recommendations"></a>Empfehlungen

Ihre Anforderungen können von der hier beschriebenen Architektur abweichen. Verwenden Sie die Empfehlungen in diesem Abschnitt als Ausgangspunkt.

### <a name="app-service-plan"></a>App Service-Plan
Verwenden Sie die Standard- oder Premium-Tarife, da sie horizontales Skalieren, automatische Skalierung und SSL (Secure Sockets Layer) unterstützen. Alle Tarife unterstützen verschiedene *Instanzgrößen*, die sich in der Anzahl der Kerne und im Arbeitsspeicher unterscheiden. Sie können den Tarif oder die Instanzgröße nach der Erstellung eines Plans ändern. Weitere Informationen zu App Service-Plänen finden Sie unter [App Service – Preise][app-service-plans-tiers].

Die Instanzen im App Service-Plan werden Ihnen in Rechnung gestellt, auch wenn die App nicht ausgeführt wird. Achten Sie darauf, nicht verwendete Pläne (z.B. Testbereitstellungen) zu löschen.

### <a name="sql-database"></a>SQL-Datenbank
Verwenden Sie die [V12-Version][sql-db-v12] von SQL-Datenbank. SQL-Datenbank unterstützt die [Diensttarife][sql-db-service-tiers] Basic, Standard und Premium mit mehreren Leistungsebenen innerhalb der einzelnen Tarife, die in [Datenbanktransaktionseinheiten (Database Transaction Units, DTUs)][sql-dtu] gemessen werden. Führen Sie Kapazitätsplanung durch, und wählen Sie einen Tarif und eine Leistungsstufe aus, die Ihren Anforderungen entsprechen.

### <a name="region"></a>Region
Stellen Sie den App Service-Plan und die SQL-Datenbank in der gleichen Region bereit, um die Netzwerklatenz zu minimieren. Wählen Sie grundsätzlich die Ihren Benutzern am nächsten gelegene Region aus.

Die Ressourcengruppe weist ebenfalls eine Region auf, die angibt, wo die Metadaten der Bereitstellung gespeichert werden. Implementieren Sie die Ressourcengruppe und ihre Ressourcen in der gleichen Region. Dies kann die Verfügbarkeit während der Bereitstellung verbessern. 

## <a name="scalability-considerations"></a>Überlegungen zur Skalierbarkeit

Ein großer Vorteil von Azure App Service ist die Möglichkeit, Ihre Anwendung abhängig von der Last zu skalieren. Hier sind einige Punkte aufgeführt, die beim Planen der Skalierung für Ihre Anwendung zu bedenken sind.

### <a name="scaling-the-app-service-app"></a>Skalieren der App Service-App

Es gibt zwei Möglichkeiten zum Skalieren einer App Service-App:

* *Zentrales Hochskalieren*, was die Änderung der Instanzgröße bedeutet. Die Instanzgröße bestimmt den Arbeitsspeicher, die Anzahl der Kerne und den Massenspeicher für die einzelnen VM-Instanzen. Das zentrale Hochskalieren kann manuell erfolgen, indem Sie die Instanzgröße oder den Plantarif ändern.  

* *Horizontales Skalieren*, was das Hinzufügen von Instanzen zum Verarbeiten höherer Auslastungen bedeutet. Für jede Tarifebene gibt es eine Maximalanzahl Instanzen. 

  Sie können manuell horizontal skalieren, indem Sie die Instanzanzahl ändern, oder [automatische Skalierung][web-app-autoscale] verwenden, wenn Azure automatisch auf der Grundlage eines Zeitplans und/oder von Leistungsmetriken Instanzen hinzufügen oder entfernen soll. Die Skalierungsvorgänge erfolgen schnell – normalerweise innerhalb von Sekunden. 

  Um die automatische Skalierung zu aktivieren, erstellen Sie ein *Profil* für die automatische Skalierung, das die Minimal- und Maximalanzahl der Instanzen definiert. Profile können geplant werden. Beispielsweise können Sie separate Profile für Wochentage und Wochenenden erstellen. Optional enthält ein Profil Regeln dafür, wann Instanzen hinzugefügt oder entfernt werden sollen. (Beispiel: Füge zwei Instanzen hinzu, wenn die CPU-Auslastung 5 Minuten lang über 70 % liegt.)
  
Empfehlungen zum Skalieren einer Web App:

* Vermeiden Sie nach Möglichkeit zentrales Hoch- und Herunterskalieren, da dies zu einem Neustart der Anwendung führen kann. Wählen Sie stattdessen einen Tarif und eine Größe aus, der bzw. die Ihre Leistungsanforderungen unter typischer Last erfüllt, und skalieren Sie die Instanzen horizontal hoch, um Änderungen beim Datenverkehrsvolumen zu bewältigen.    
* Aktivieren Sie die automatische Skalierung. Wenn Ihre Anwendung eine vorhersehbare normale Arbeitsauslastung aufweist, erstellen Sie Profile, um die Instanzanzahl im Voraus zu planen. Wenn die Arbeitsauslastung nicht vorhersehbar ist, verwenden Sie regelbasierte automatische Skalierung, um auf Änderungen der Arbeitsauslastung beim Eintreten zu reagieren. Beide Ansätze können kombiniert werden.
* Die CPU-Auslastung ist im Allgemeinen eine gute Metrik für Regeln zur automatischen Skalierung. Jedoch sollten Sie einen Auslastungstest Ihrer Anwendung durchführen, potenzielle Engpässe identifizieren und Regeln für die automatische Skalierung auf diesen Daten aufbauen.  
* Regeln für die automatische Skalierung beinhalten eine *Abkühlphase*; damit wird die Warteperiode nach dem Abschluss einer Skalierungsaktion bezeichnet, bevor eine neue Skalierungsaktion gestartet wird. In der Abkühlphase stabilisiert sich das System, bevor eine erneute Skalierung erfolgt. Legen Sie eine kürzere Abkühlphase für das Hinzufügen von Instanzen und eine längere Abkühlphase für das Entfernen von Instanzen fest. Legen Sie beispielsweise 5 Minuten für das Hinzufügen einer Instanz aber 60 Minuten für das Entfernen einer Instanz fest. Es ist besser, bei hoher Auslastung schnell neue Instanzen hinzuzufügen, um den zusätzlichen Datenverkehr zu bewältigen, und das System dann nach und nach wieder herunterzuskalieren.

### <a name="scaling-sql-database"></a>Skalieren von SQL-­Datenbank
Wenn Sie eine höhere Dienstebene oder Leistungsstufe für SQL-Datenbank benötigen, können Sie einzelne Datenbanken ohne Ausfallzeit der Anwendung zentral hochskalieren. Weitere Informationen finden Sie unter [SQL-Datenbankoptionen und -leistung: Grundlegendes zum Angebot in den einzelnen Tarifen][sql-db-scale].

## <a name="availability-considerations"></a>Überlegungen zur Verfügbarkeit
Zum Entstehungszeitpunkt dieses Artikels ist die SLA (Vereinbarung zum Servicelevel) für App Service 99,95 % und die SLA für SQL-Datenbank 99,99 % für die Tarife Basic, Standard und Premium. 

> [!NOTE]
> Die App Service-SLA gilt sowohl für einzelne als auch für mehrere Instanzen.  
>
>

### <a name="backups"></a>Backups
Im Fall eines Datenverlusts stellt SQL-Datenbank Point-in-Time-Wiederherstellung und Geowiederherstellung bereit. Diese Funktionen sind in allen Tarifen verfügbar und automatisch aktiviert. Sie brauchen keine Sicherungen zu planen oder zu verwalten. 

- Verwenden Sie die Point-in-Time-Wiederherstellung, für die [Wiederherstellung nach Bedienungsfehlern][sql-human-error], um einen früheren Zeitpunkt der Datenbankausführung wiederherzustellen. 
- Verwenden Sie Geowiederherstellung zur [Wiederherstellung nach einem Dienstausfall][sql-outage-recovery], indem Sie eine Datenbank aus einer georedundanten Sicherung wiederherstellen. 

Weitere Informationen finden Sie unter [Geschäftskontinuität für die Cloud und Notfallwiederherstellung für Datenbanken mit SQL-Datenbank][sql-backup].

App Service bietet eine Funktion zum [Sichern und Wiederherstellen][web-app-backup] für Ihre Anwendungsdateien. Beachten Sie dabei jedoch, dass die gesicherten Dateien Anwendungseinstellungen in unverschlüsseltem Text enthalten, die Geheimnisse enthalten können, wie z.B. Verbindungszeichenfolgen. Vermeiden Sie die Verwendung der App Service-Sicherungsfunktion für die Sicherung Ihrer SQL-Datenbanken, da sie die Datenbank in eine SQL BACPAC-Datei exportiert und dabei [DTUs][sql-dtu] verbraucht. Verwenden Sie stattdessen die oben beschriebene SQL-Datenbank Point-in-Time-Wiederherstellung.

## <a name="manageability-considerations"></a>Überlegungen zur Verwaltbarkeit
Erstellen Sie separate Ressourcengruppen für Produktions-, Entwicklungs- und Testumgebungen. Dies erleichtert das Verwalten von Bereitstellungen, das Löschen von Testbereitstellungen und das Zuweisen von Zugriffsrechten.

Berücksichtigen Sie beim Zuweisen von Ressourcengruppen die folgenden Punkte:

* Lebenszyklus. Platzieren Sie Ressourcen mit gleichem Lebenszyklus im Allgemeinen in der gleichen Ressourcengruppe.
* Zugriff. Sie können [rollenbasierte Zugriffssteuerung][rbac] (Role-Based Access Control) verwenden, um Zugriffsrichtlinien auf die in einer Gruppe enthaltenen Ressourcen anzuwenden.
* Abrechnung. Sie können die angefallenen Kosten für die Ressourcengruppe anzeigen.  

Weitere Informationen finden Sie unter [Übersicht über den Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview).

### <a name="deployment"></a>Bereitstellung
Die Bereitstellung umfasst zwei Schritte:

1. Bereitstellen der Azure-Ressourcen. Wir empfehlen für diesen Schritt die Verwendung von [Azure-Ressource-Manager-Vorlagen][arm-template]. Mithilfe von Vorlagen können Bereitstellungen über PowerShell oder die Azure-Befehlszeilenschnittstelle (CLI) leichter automatisiert werden.
2. Bereitstellen der Anwendung (Code, Binarys und Inhaltsdateien). Es gibt eine Reihe verschiedener Optionen, einschließlich der Bereitstellung aus einem lokalen Git-Repository, der Verwendung von Visual Studio oder Continuous Deployment aus einer cloudbasierten Quellcodeverwaltung. Weitere Informationen finden Sie unter [Bereitstellen der App in Azure App Service][deploy].  

Eine App Service-App weist immer einen Bereitstellungsslot mit dem Namen `production` auf, der die Produktions-Livewebsite darstellt. Wir empfehlen, zum Bereitstellen von Updates einen Stagingslot zu erstellen. Zu den Vorteilen beim Verwenden eines Stagingslots gehören:

* Sie können den Erfolg der Bereitstellung überprüfen, bevor Sie sie in die Produktionsumgebung einbringen.
* Das Bereitstellen in einem Stagingslot stellt sicher, dass alle Instanzen warmgelaufen sind, bevor sie in die Produktionsumgebung eingebracht werden. Viele Anwendungen weisen eine erhebliche Aufwärmdauer und Kaltstartzeit auf.

Ferner empfehlen wir die Erstellung eines dritten Slots zur Aufnahme der letzten als funktionierend bekannten Bereitstellung. Verschieben Sie nach dem Vertauschen von Staging- und Produktionsumgebung die bisherige Produktionsumgebung (die sich jetzt im Stagingslot befindet) in den Slot für die letzte als funktionierend bekannte Bereitstellung. Wenn Sie zu einem späteren Zeitpunkt ein Problem feststellen, können Sie auf diese Weise schnell zur letzten als funktionierend bekannten Version zurückkehren.

![[1]][1]

Wenn Sie eine frühere Version wiederherstellen, achten Sie darauf, dass alle Änderungen am Datenbankschema abwärtskompatibel sind.

Verwenden Sie keine Slots in Ihrer Produktionsumgebung für Tests, da alle Apps in einem App Service-Plan die gleichen VM-Instanzen nutzen. Auslastungstests können z.B. den aktiven Produktionsstandort beeinträchtigen. Erstellen Sie stattdessen separate App Service-Pläne für die Produktion und für Tests. Indem Sie Testbereitstellungen in einen separaten Plan aufnehmen, isolieren Sie sie von der Produktionsversion.

### <a name="configuration"></a>Konfiguration
Speichern Sie die Konfigurationseinstellungen als [App-Einstellungen][app-settings]. Definieren Sie die App-Einstellungen in Ihren Ressourcen-Manager-Vorlagen oder mithilfe von PowerShell. Zur Laufzeit stehen App-Einstellungen der Anwendung als Umgebungsvariablen zur Verfügung.

Checken Sie niemals Kennwörter, Zugriffsschlüssel oder Verbindungszeichenfolgen in die Quellcodeverwaltung ein. Übergeben Sie diese stattdessen als Parameter an ein Bereitstellungsskript, das diese Werte als Anwendungseinstellungen speichert.

Wenn Sie einen Bereitstellungsslot tauschen, werden die App-Einstellungen standardmäßig ebenfalls getauscht. Wenn Sie für die Produktions- und die Stagingumgebung unterschiedliche Einstellungen benötigen, können Sie Anwendungseinstellungen erstellen, die fest bei einem Slot verbleiben und nicht getauscht werden.

### <a name="diagnostics-and-monitoring"></a>Diagnose und Überwachung
Aktivieren Sie die [Diagnoseprotokollierung][diagnostic-logs], einschließlich Anwendungsprotokollierung und Webserverprotokollierung. Konfigurieren Sie die Verwendung von Blob-Speicher für die Protokollierung. Erstellen Sie aus Leistungsgründen ein separates Speicherkonto für Diagnoseprotokolle. Verwenden Sie nicht das gleiche Speicherkonto für Protokolle und Anwendungsdaten. Detailliertere Anweisungen zur Protokollierung finden Sie unter [Leitfaden zu Überwachung und Diagnose][monitoring-guidance].

Verwenden Sie einen Dienst wie [New Relic][new-relic] oder [Application Insights][app-insights], um Leistung und Verhalten der Anwendung unter Last zu überwachen. Beachten Sie die [Grenzwerte der Datenrate][app-insights-data-rate] für Application Insights.

Führen Sie Leistungstests mithilfe eines Tools wie [Visual Studio Team Services][vsts] aus. Eine allgemeine Übersicht zur Leistungsanalyse in Cloud-Anwendungen finden Sie im [Performance Analysis Primer][perf-analysis] (Einsteig in die Leistungsanalyse).

Tipps zur Problembehandlung bei der Anwendung:

* Verwenden Sie das [Blatt „Problembehandlung“][troubleshoot-blade] im Azure-Portal, um Lösungen für häufige Probleme zu finden.
* Aktivieren Sie [Protokollstreaming][web-app-log-stream], um Protokollierungsinformationen nahezu in Echtzeit anzuzeigen.
* Das [Kudu-Dashboard][kudu] weist eine Reihe von Tools zum Überwachen und Debuggen Ihrer Anwendung auf. Weitere Informationen finden Sie unter [Online-Tools für Azure-Websites, die Sie kennen sollten][kudu] (Blogbeitrag). Sie können das Kudu-Dashboard vom Azure-Portal aus erreichen. Öffnen Sie das Blatt für Ihre App, klicken Sie auf **Extras**, und klicken Sie dann auf **Kudu**.
* Wenn Sie Visual Studio verwenden, finden Sie Tipps zu Debuggen und Problembehandlung im Artikel [Problembehandlung von Web-Apps in Azure App Service in Visual Studio][troubleshoot-web-app].

## <a name="security-considerations"></a>Sicherheitshinweise
Dieser Abschnitt enthält Sicherheitshinweise, die für die in diesem Artikel beschriebenen Azure-Dienste spezifisch sind. Es handelt sich nicht um eine vollständige Liste der bewährten Sicherheitsmethoden. Weitere Sicherheitshinweise finden Sie unter [Schützen einer App in Azure App Service][app-service-security].

### <a name="sql-database-auditing"></a>SQL-Datenbanküberwachung
Die Überwachung kann Ihnen dabei helfen, die gesetzlichen Bestimmungen einzuhalten und Einblicke in Abweichungen und Unregelmäßigkeiten zu erhalten, die auf Geschäftsvorgänge oder mutmaßliche Sicherheitsverstöße hinweisen können. Weitere Informationen finden Sie unter [Erste Schritte mit der Überwachung von SQL-Datenbanken][sql-audit].

### <a name="deployment-slots"></a>Bereitstellungsslots
Jeder Bereitstellungsslot weist eine öffentliche IP-Adresse auf. Schützen Sie die nicht produktiven Slots mithilfe einer [Azure Active Directory-Anmeldung][aad-auth], sodass nur Mitglieder Ihres Entwicklungs- und DevOps-Teams auf diese Endpunkte zugreifen können.

### <a name="logging"></a>Protokollierung
Protokolle sollten niemals Benutzerkennwörter oder andere Informationen aufzeichnen, die für Identitätsdiebstähle verwendet werden können. Entfernen Sie diese Details aus den Daten, bevor Sie sie speichern.   

### <a name="ssl"></a>SSL
Eine App Service-App umfasst ohne Zusatzkosten einen SSL-Endpunkt in einer Unterdomäne `azurewebsites.net`. Der SSL-Endpunkt schließt ein Platzhalterzertifikat für die `*.azurewebsites.net`-Domäne ein. Wenn Sie einen benutzerdefinierten Domänennamen verwenden, müssen Sie ein Zertifikat bereitstellen, das der benutzerdefinierten Domäne entspricht. Die einfachste Möglichkeit besteht darin, ein Zertifikat direkt über das Azure-Portal zu erwerben. Außerdem können Sie Zertifikate von anderen Zertifizierungsstellen importieren. Weitere Informationen finden Sie unter [Kaufen und Konfigurieren eines SSL-Zertifikats für Ihren Azure App Service][ssl-cert].

Als bewährte Sicherheitsmethode sollte Ihre Anwendung HTTPS durch Umleitung von HTTP-Anforderungen durchsetzen. Sie können dies innerhalb Ihrer Anwendung implementieren oder eine Regel zum Umschreiben von URL verwenden, wie unter [Aktivieren von HTTPS für eine App in Azure App Service][ssl-redirect].

### <a name="authentication"></a>Authentifizierung
Wir empfehlen die Authentifizierung über einen Identitätsanbieter (IDP), wie etwa Azure AD, Facebook, Google oder Twitter. Verwenden Sie OAuth 2 oder OpenID Connect (OIDC) für den Authentifizierungsablauf. Azure AD bietet Funktionen zum Verwalten von Benutzern und Gruppen, Erstellen von Anwendungsrollen, Integrieren Ihrer lokalen Identitäten und Nutzen von Back-End-Diensten, wie etwa Office 365 und Skype for Business.

Vermeiden Sie die direkte Verwaltung von Benutzeranmeldungen und Anmeldeinformationen durch die Anwendung, da dadurch eine mögliche Angriffsfläche entsteht.  Den Mindeststandard sollten E-Mail-Bestätigung, Kennwortwiederherstellung und mehrstufige Authentifizierung bilden, außerdem eine Überprüfung der Kennwortsicherheit und die sichere Speicherung von Kennworthashes. Die großen Identitätsanbieter übernehmen alle diese Dinge für Sie und überwachen und verbessern ihre Sicherheitspraktiken ständig.

Erwägen Sie den Einsatz von [App Service-Authentifizierung][app-service-auth] zum Implementieren des OAuth/OIDC-Authentifizierungsablaufs. App Service-Authentifizierung bietet folgende Vorteile:

* Einfach zu konfigurieren.
* Für einfache Authentifizierungsszenarien ist kein Code erforderlich.
* Unterstützt delegierte Autorisierung mithilfe von OAuth-Zugriffstoken, um Ressourcen im Benutzerauftrag zu nutzen.
* Bietet einen integrierten Tokencache.

Einige Einschränkungen der App Service-Authentifizierung:  

* Eingeschränkte Anpassungsoptionen.
* Die delegierte Autorisierung ist auf eine Back-End-Ressource pro Anmeldesitzung beschränkt.
* Wenn Sie mehr als einen IDP verwenden, gibt es keinen integrierten Mechanismus für die Startbereichsermittlung.
* In Szenarien mit mehreren Mandanten muss die Anwendung die Logik für die Überprüfung des Tokenbenutzers implementieren.

## <a name="deploy-the-solution"></a>Bereitstellen der Lösung
Ein Beispiel einer Ressourcen-Manager-Vorlage für diese Architektur ist [auf GitHub verfügbar][paas-basic-arm-template].

Wenn Sie die Vorlage mit PowerShell bereitstellen möchten, führen Sie die folgenden Befehle aus:

```
New-AzureRmResourceGroup -Name <resource-group-name> -Location "West US"

$parameters = @{"appName"="<app-name>";"environment"="dev";"locationShort"="uw";"databaseName"="app-db";"administratorLogin"="<admin>";"administratorLoginPassword"="<password>"}

New-AzureRmResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile .\PaaS-Basic.json -TemplateParameterObject  $parameters
```

Weitere Informationen finden Sie unter [Bereitstellen von Ressourcen mit Azure Resource Manager-Vorlagen][deploy-arm-template].

<!-- links -->

[aad-auth]: /azure/app-service-mobile/app-service-mobile-how-to-configure-active-directory-authentication
[app-insights]: /azure/application-insights/app-insights-overview
[app-insights-data-rate]: /azure/application-insights/app-insights-pricing
[app-service]: https://azure.microsoft.com/documentation/services/app-service/
[app-service-auth]: /azure/app-service-api/app-service-api-authentication
[app-service-plans]: /azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview
[app-service-plans-tiers]: https://azure.microsoft.com/pricing/details/app-service/
[app-service-security]: /azure/app-service-web/web-sites-security
[app-settings]: /azure/app-service-web/web-sites-configure
[arm-template]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[custom-domain-name]: /azure/app-service-web/web-sites-custom-domain-name
[deploy]: /azure/app-service-web/web-sites-deploy
[deploy-arm-template]: /azure/resource-group-template-deploy
[deployment-slots]: /azure/app-service-web/web-sites-staged-publishing
[diagnostic-logs]: /azure/app-service-web/web-sites-enable-diagnostic-log
[kudu]: https://azure.microsoft.com/blog/windows-azure-websites-online-tools-you-should-know-about/
[monitoring-guidance]: ../../best-practices/monitoring.md
[new-relic]: http://newrelic.com/
[paas-basic-arm-template]: https://github.com/mspnp/reference-architectures/tree/master/app-service-web-app/basic-web-app/Paas-Basic/Templates
[perf-analysis]: https://github.com/mspnp/performance-optimization/blob/master/Performance-Analysis-Primer.md
[rbac]: /azure/active-directory/role-based-access-control-what-is
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[sla]: https://azure.microsoft.com/support/legal/sla/
[sql-audit]: /azure/sql-database/sql-database-auditing-get-started
[sql-backup]: /azure/sql-database/sql-database-business-continuity
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-db-overview]: /azure/sql-database/sql-database-technical-overview
[sql-db-scale]: /azure/sql-database/sql-database-service-tiers#scaling-up-or-scaling-down-a-single-database
[sql-db-service-tiers]: /azure/sql-database/sql-database-service-tiers
[sql-db-v12]: /azure/sql-database/sql-database-features
[sql-dtu]: /azure/sql-database/sql-database-service-tiers
[sql-human-error]: /azure/sql-database/sql-database-business-continuity#recover-a-database-after-a-user-or-application-error
[sql-outage-recovery]: /azure/sql-database/sql-database-business-continuity#recover-a-database-to-another-region-from-an-azure-regional-data-center-outage
[ssl-redirect]: /azure/app-service-web/web-sites-configure-ssl-certificate#bkmk_enforce
[sql-resource-limits]: /azure/sql-database/sql-database-resource-limits
[ssl-cert]: /azure/app-service-web/web-sites-purchase-ssl-web-site
[troubleshoot-blade]: https://azure.microsoft.com/updates/self-service-troubleshooting-for-app-service-web-apps-customers/
[troubleshoot-web-app]: /azure/app-service-web/web-sites-dotnet-troubleshoot-visual-studio
[visio-download]: https://archcenter.azureedge.net/cdn/app-service-reference-architectures.vsdx
[vsts]: https://www.visualstudio.com/features/vso-cloud-load-testing-vs.aspx
[web-app-autoscale]: /azure/app-service-web/web-sites-scale
[web-app-backup]: /azure/app-service-web/web-sites-backup
[web-app-log-stream]: /azure/app-service-web/web-sites-enable-diagnostic-log#streamlogs
[0]: ./images/basic-web-app.png "Architektur einer einfachen Azure Web-Anwendung"
[1]: ./images/paas-basic-web-app-staging-slots.png "Austauschen der Slots für Produktions- und Stagingbereitstellungen"
