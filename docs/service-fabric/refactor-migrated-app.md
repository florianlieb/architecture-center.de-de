---
title: Umgestalten einer Azure Service Fabric-Anwendung, die von Azure Cloud Services migriert wurde
description: Umgestalten einer vorhandenen Azure Service Fabric-Anwendung, die von Azure Cloud Services migriert wurde
author: petertay
ms.date: 01/30/2018
ms.openlocfilehash: 08ef3af68b8eaba36a5b871449f0aba764fe5a04
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/08/2018
---
# <a name="refactor-an-azure-service-fabric-application-migrated-from-azure-cloud-services"></a>Umgestalten einer Azure Service Fabric-Anwendung, die von Azure Cloud Services migriert wurde

[![GitHub](../_images/github.png)-Beispielcode][sample-code]

In diesem Artikel erfahren Sie, wie Sie eine vorhandene Azure Service Fabric-Anwendung umgestalten, um ihr zu einer detaillierteren Architektur zu verhelfen. Der Artikel konzentriert sich auf Gestaltungs-, Paket-, Leistungs- und Bereitstellungsaspekte der umgestalteten Service Fabric-Anwendung.

## <a name="scenario"></a>Szenario

Wie im Artikel [Migrieren einer Azure Cloud Services-Anwendung zu Azure Service Fabric][migrate-from-cloud-services] bereits erwähnt, hat das Team für Muster und Vorgehensweisen im Jahr 2012 ein Buch über die Entwicklung und Implementierung einer Cloud Services-Anwendung in Azure geschrieben. Das Buch beschreibt ein fiktives Unternehmen namens Tailspin, das eine Cloud Services-Anwendung namens **Surveys** entwerfen und implementieren möchte. Mit der Anwendung „Surveys“ können Benutzer öffentliche Umfragen erstellen und veröffentlichen. Das folgende Diagramm zeigt die Architektur dieser Version der Anwendung:

![](./images/tailspin01.png)

Die Webrolle **Tailspin.Web** hostet eine ASP.NET-MVC-Website, die Tailspin-Kunden für Folgendes verwenden:
* Registrieren bei der Anwendung „Surveys“
* Erstellen oder Löschen einer einzelnen Umfrage
* Anzeigen der Ergebnisse für eine einzelne Umfrage
* Anfordern des Exports der Umfrageergebnisse in SQL
* Anzeigen aggregierter Umfrageergebnisse und Analysen

Die Webrolle **Tailspin.Web.Survey.Public** hostet zudem eine ASP.NET-MVC-Website, auf der Besucher die Umfragen beantworten. Die Antworten werden zur Speicherung einer Warteschlange hinzugefügt.

Die Workerrolle **Tailspin.Workers.Survey** führt die Hintergrundverarbeitung durch und greift dazu Anforderungen aus verschiedenen Warteschlangen auf.

Als Nächstes hat das Team für Muster und Vorgehensweisen ein neues Projekt erstellt, um diese Anwendung in Azure Service Fabric zu überführen. Ziel dieses Projekts war es, nur Codeänderungen vorzunehmen, die für die Ausführung der Anwendung in einem Azure Service Fabric-Cluster erforderlich sind. Die ursprünglichen Web- und Workerrollen wurden daher nicht in eine detailliertere Architektur aufgegliedert. Die resultierende Architektur ähnelt sehr stark der Clouddienstversion der Anwendung:

![](./images/tailspin02.png)

Der Dienst **Tailspin.Web** wird aus der ursprünglichen Webrolle *Tailspin.Web* portiert.

Der Dienst **Tailspin.Web.Survey.Public** wird aus der ursprünglichen Webrolle *Tailspin.Web.Survey.Public* portiert.

Der Dienst **Tailspin.AnswerAnalysisService** wird aus der ursprünglichen Webrolle *Tailspin.Workers.Survey* portiert.

> [!NOTE] 
> Der Code der einzelnen Web- und Workerrollen wurde jeweils nur geringfügig geändert, **Tailspin.Web** und **Tailspin.Web.Survey.Public** wurden allerdings für das Selbsthosting eines Webservers vom Typ [Kestrel] angepasst. Bei der früheren Version der Anwendung „Surveys“ handelt es sich um eine ASP.NET-Anwendung, die mit Internet Information Services (IIS) gehostet wurde. IIS kann jedoch nicht als Dienst in Service Fabric ausgeführt werden. Daher muss jeder Webserver selbsthostingfähig sein (wie etwa [Kestrel]). In bestimmten Situationen kann IIS in einem Container in Service Fabric ausgeführt werden. Weitere Informationen finden Sie unter [Szenarien für die Verwendung von Containern][container-scenarios].  

Nun gestaltet Tailspin die Anwendung „Surveys“ zu einer detaillierten Architektur um. Ziel dieser Umgestaltung ist es, die Entwicklung, Erstellung und Bereitstellung der Anwendung zu vereinfachen. Durch die Aufgliederung der vorhandenen Web- und Workerrollen in eine detailliertere Architektur möchte Tailspin die enge Kopplung der Kommunikations- und Datenabhängigkeiten zwischen diesen Rollen beseitigen.

Außerdem erwartet sich Tailspin folgende Vorteile von der Migration der Anwendung „Surveys“ zu einer detaillierten Architektur:
* Jeder Dienst kann in unabhängigen Projekten mit einem Umfang verpackt werden, der problemlos von einem kleinen Team verwaltet werden kann.
* Jeder Dienst kann unabhängig versioniert und bereitgestellt werden.
* Jeder Dienst kann mit der optimalen Technologie für den jeweiligen Dienst implementiert werden. Ein Service Fabric-Cluster kann beispielsweise Dienste enthalten, die mit verschiedenen .NET Framework-Versionen, mit Java oder mit anderen Sprachen wie C# oder C++ erstellt wurden.
* Jeder Dienst kann unabhängig skaliert werden, um auf eine Zu- oder Abnahme der Last zu reagieren.

> [!NOTE] 
> Mehrinstanzenfähigkeit wird im Rahmen der Umgestaltung dieser Anwendung nicht behandelt. Tailspin hat mehrere Möglichkeiten zur Unterstützung von Mehrinstanzenfähigkeit und kann diese Designentscheidungen später ohne Auswirkungen auf das ursprüngliche Design treffen. So kann Tailspin beispielsweise für jeden Mandanten in einem Cluster separate Instanzen der Dienste oder für jeden Mandanten einen separaten Cluster erstellen.

## <a name="design-considerations"></a>Überlegungen zum Entwurf
 
Das folgende Diagramm zeigt die Architektur der Anwendung „Surveys“ nach der Umgestaltung zu einer detaillierteren Architektur:

![](./images/surveys_03.png)

**Tailspin.Web** ist ein zustandsloser Dienst für eine selbstgehostete ASP.NET-MVC-Anwendung, mit der Tailspin-Kunden Umfragen erstellen und Umfrageergebnisse anzeigen. Dieser Dienst hat größtenteils den gleichen Code wie der Dienst *Tailspin.Web* aus der portierten Service Fabric-Anwendung. Wie weiter oben erwähnt, verwendet dieser Dienst ASP.NET Core und implementiert einen Weblistener, anstatt Kestrel als Web-Front-End zu verwenden.

**Tailspin.Web.Survey.Public** ist ein zustandsloser Dienst und hostet ebenfalls selbst eine ASP.NET-MVC-Website. Benutzer besuchen diese Website, um Umfragen aus einer Liste auszuwählen und zu beantworten. Dieser Dienst hat größtenteils den gleichen Code wie der Dienst *Tailspin.Web.Survey.Public* aus der portierten Service Fabric-Anwendung. Dieser Dienst verwendet ebenfalls ASP.NET Core und implementiert auch einen Weblistener, anstatt Kestrel als Web-Front-End zu verwenden.

**Tailspin.SurveyResponseService** ist ein zustandsbehafteter Dienst, der Umfrageantworten in Azure Blob Storage speichert. Darüber hinaus führt er Antworten in den Umfrageanalysedaten zusammen. Der Dienst wird als zustandsbehafteter Dienst implementiert, da er [ReliableConcurrentQueue][reliable-concurrent-queue] verwendet, um Umfrageantworten in Batches zu verarbeiten. Diese Funktion wurde ursprünglich im Dienst *Tailspin.AnswerAnalysisService* in der portierten Service Fabric-Anwendung implementiert.

**Tailspin.SurveyManagementService** ist ein zustandsloser Dienst zum Speichern und Abrufen von Umfragen und der Fragen von Umfragen. Der Dienst verwendet Azure Blob Storage. Diese Funktion wurde ebenfalls ursprünglich in den Datenzugriffskomponenten der Dienste *Tailspin.Web* und *Tailspin.Web.Survey.Public* in der portierten Service Fabric-Anwendung implementiert. Tailspin hat die ursprüngliche Funktion in diesem Dienst umgestaltet, um eine unabhängige Skalierung zu ermöglichen.

**Tailspin.SurveyAnswerService** ist ein zustandsloser Dienst zum Abrufen von Umfrageantworten und -analysen. Der Dienst verwendet ebenfalls Azure Blob Storage. Diese Funktion wurde ebenfalls ursprünglich in den Datenzugriffskomponenten des Diensts *Tailspin.Web* in der portierten Service Fabric-Anwendung implementiert. Tailspin hat die ursprüngliche Funktion in diesem Dienst umgestaltet, da das Unternehmen mit weniger Last rechnet und durch die Verwendung von weniger Instanzen Ressourcen sparen möchte.

**Tailspin.SurveyAnalysisService** ist ein zustandsloser Dienst, der Zusammenfassungsdaten für Umfrageantworten in einem Redis Cache speichert, um sie schnell abrufen zu können. Dieser Dienst wird von *Tailspin.SurveyResponseService* aufgerufen, wenn eine Umfrage beantwortet wird, und die neuen Umfrageantwortdaten werden in den Zusammenfassungsdaten zusammengeführt. Dieser Dienst enthält die Funktion, die ursprünglich im Dienst *Tailspin.AnswerAnalysisService* aus der portierten Service Fabric-Anwendung implementiert wurde.

## <a name="stateless-versus-stateful-services"></a>Gegenüberstellung von Zustandslosen und zustandsbehafteten Diensten

Azure Service Fabric unterstützt folgende Programmiermodelle:
* Beim Modell der ausführbaren Gastanwendungsdatei kann jede beliebige ausführbare Datei als Dienst verpackt und in einem Service Fabric-Cluster bereitgestellt werden. Service Fabric orchestriert und verwaltet die Ausführung der ausführbaren Gastanwendungsdatei.
* Das Containermodell ermöglicht die Bereitstellung von Diensten in Containerimages. Service Fabric unterstützt die Erstellung und Verwaltung von Containern oberhalb von Linux-Kernelcontainern und Windows Server-Containern. 
* Das Reliable Services-Programmiermodell ermöglicht die Erstellung zustandsloser oder zustandsbehafteter Dienste mit Integration in alle Service Fabric-Plattformfeatures. Zustandsbehaftete Dienste ermöglichen die Speicherung des replizierten Zustands im Service Fabric-Cluster. Bei zustandslosen Diensten ist dies nicht möglich.
* Das Reliable Actors-Programmiermodell ermöglicht die Erstellung von Diensten, die das Muster „Virtueller Akteur“ implementieren.

Mit Ausnahme des Diensts *Tailspin.SurveyResponseService* sind alle Dienste in der Anwendung „Surveys“ zustandslose Reliable Services. Dieser Dienst implementiert [ReliableConcurrentQueue][reliable-concurrent-queue], um Umfrageantworten zu verarbeiten, wenn sie empfangen werden. Antworten in „ReliableConcurrentQueue“ werden in Azure Blob Storage gespeichert und zur Analyse an *Tailspin.SurveyAnalysisService* übergeben. Tailspin entscheidet sich für einen ReliableConcurrentQueue-basierten Ansatz, da für Antworten keine strenge FIFO-Sortierung (First In – First Out) erforderlich ist, wie sie beispielsweise von einer Warteschlange wie Azure Service Bus geboten wird. „ReliableConcurrentQueue“ bietet zudem einen hohen Durchsatz und eine geringe Wartezeit beim Hinzufügen zur Warteschlange und beim Entfernen aus der Warteschlange.

Beachten Sie, dass Vorgänge zum Speichern von Elementen, die aus „ReliableConcurrentQueue“ entfernt wurden, im Idealfall idempotent sein sollten. Wenn bei der Verarbeitung eines Elements aus der Warteschlange eine Ausnahme ausgelöst wird, kann das gleiche Element mehrmals verarbeitet werden. In der Anwendung „Surveys“ ist der Vorgang zum Zusammenführen von Umfrageantworten in *Tailspin.SurveyAnalysisService* nicht idempotent, da Tailspin beschlossen hat, dass es sich bei den Umfrageanalysedaten lediglich um eine aktuelle Momentaufnahme der Analysedaten handelt und diese nicht konsistent sein müssen. Da die in Azure Blob Storage gespeicherten Umfrageantworten letztlich konsistent sind, kann die Abschlussanalyse der Umfrage jederzeit auf der Grundlage dieser Daten neu berechnet werden.

## <a name="communication-framework"></a>Kommunikationsframework

Jeder Dienst in der Anwendung „Surveys“ kommuniziert über eine RESTful-Web-API. RESTful-APIs bieten folgende Vorteile:
* Benutzerfreundlichkeit: Jeder Dienst basiert auf ASP.NET Core-MVC und unterstützt somit nativ die Erstellung von Web-APIs.
* Sicherheit: Zwar ist es nicht erforderlich, für jeden Dienst SSL zu verwenden, Tailspin hat jedoch die Möglichkeit, dies so einzurichten. 
* Versionsverwaltung: Clients können für eine bestimmte Version einer Web-API geschrieben und getestet werden.

Dienste in der Anwendung „Surveys“ verwenden den von Service Fabric implementierten [Reverseproxy][reverse-proxy]. Der Reverseproxy ist ein Dienst, der auf den einzelnen Knoten im Service Fabric-Cluster ausgeführt wird. Er bietet Endpunktauflösung und automatische Wiederholung und behandelt andere Arten von Verbindungsfehlern. Zur Verwendung des Reverseproxys erfolgt jeder RESTful-API-Aufruf für einen bestimmten Dienst über einen vordefinierten Reverseproxyport.  Wenn der Reverseproxyport also beispielsweise auf **19081** festgelegt wurde, kann ein Aufruf für *Tailspin.SurveyAnswerService* wie folgt durchgeführt werden:

```csharp
static SurveyAnswerService()
{
    httpClient = new HttpClient
    {
        BaseAddress = new Uri("http://localhost:19081/Tailspin/SurveyAnswerService/")
    };
}
```
Geben Sie zum Aktivieren des Reverseproxys bei der Erstellung des Service Fabric-Clusters einen Reverseproxyport an. Weitere Informationen finden Sie unter [Reverseproxy in Azure Service Fabric][reverse-proxy].

## <a name="performance-considerations"></a>Überlegungen zur Leistung

Tailspin hat die ASP.NET Core-Dienste für *Tailspin.Web* und *Tailspin.Web.Surveys.Public* mithilfe von Visual Studio-Vorlagen erstellt. Diese Vorlagen beinhalten standardmäßig eine Protokollierung in der Konsole. Die Protokollierung in die Konsole kann während der Entwicklung und beim Debuggen genutzt werden, sollte aber entfernt werden, wenn die Anwendung in der Produktionsumgebung bereitgestellt wird.

> [!NOTE]
> Weitere Informationen zum Einrichten der Überwachung und Diagnose für Service Fabric-Anwendungen, die in der Produktionsumgebung ausgeführt werden, finden Sie unter [Überwachung und Diagnose für Azure Service Fabric][monitoring-diagnostics].

So sollten beispielsweise die folgenden Zeilen in *startup.cs* für jeden der Web-Front-End-Dienste auskommentiert werden:

```csharp
// This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    //loggerFactory.AddConsole(Configuration.GetSection("Logging"));
    //loggerFactory.AddDebug();

    app.UseMvc();
}
```

> [!NOTE]
> Diese Zeilen können bedingt ausgeschlossen werden, wenn Visual Studio bei der Veröffentlichung auf „Release“ festgelegt ist.

Wenn Tailspin die Anwendung „Surveys“ dann in der Produktionsumgebung bereitstellt, wird Visual Studio in den Modus **Release** versetzt.

## <a name="deployment-considerations"></a>Überlegungen zur Bereitstellung

Die umgestaltete Anwendung „Surveys“ besteht aus fünf zustandslosen Diensten und einem zustandsbehafteten Dienst. Die Clusterplanung ist also auf die Bestimmung der richtigen VM-Größe und der Anzahl von Knoten beschränkt. In der Datei *applicationmanifest.xml*, die den Cluster beschreibt, legt Tailspin das Attribut *InstanceCount* des Tags *StatelessService* für jeden der Dienste auf „-1“ fest. Durch den Wert „-1“ wird Service Fabric angewiesen, auf jedem Knoten im Cluster eine Instanz des Diensts zu erstellen.

> [!NOTE]
> Bei zustandsbehafteten Diensten muss zusätzlich die richtige Anzahl von Partitionen und Replikaten für die Daten geplant werden.

Tailspin stellt den Cluster über das Azure-Portal bereit. Der Ressourcentyp „Service Fabric-Cluster“ stellt die gesamte erforderliche Infrastruktur bereit – einschließlich VM-Skalierungsgruppen und Lastenausgleich. Die empfohlenen VM-Größen werden während des Bereitstellungsprozesses für den Service Fabric-Cluster im Azure-Portal angezeigt. Da die virtuellen Computer in einer VM-Skalierungsgruppe bereitgestellt werden, können sie sowohl zentral als auch horizontal hochskaliert werden, wenn die Benutzerauslastung zunimmt.

> [!NOTE]
> Wie bereits erwähnt waren die beiden Web-Front-Ends in der migrierten Version der Anwendung „Surveys“ selbstgehostet (unter Verwendung von ASP.NET Core und einem Kestrel-Webserver). Die migrierte Version der Anwendung „Surveys“ verwendet zwar keinen Reverseproxy, es wird jedoch dringend empfohlen, einen Reverseproxy wie IIS, Nginx oder Apache zu verwenden. Weitere Informationen finden Sie in der [Einführung in die Kestrel-Webserverimplementierung in ASP.NET Core][kestrel-intro].
> In der umgestalteten Anwendung „Surveys“ werden die beiden Web-Front-Ends unter Verwendung von ASP.NET Core mit [WebListener][weblistener] als Webserver selbstgehostet, sodass kein Reverseproxy erforderlich ist.

## <a name="next-steps"></a>Nächste Schritte

Der Code der Anwendung „Surveys“ steht auf [GitHub][sample-code] zur Verfügung.

Wenn Sie noch nicht mit [Azure Service Fabric][service-fabric] gearbeitet haben, richten Sie zunächst Ihre Entwicklungsumgebung ein, und laden Sie anschließend das neueste [Azure SDK][azure-sdk] und das [Azure Service Fabric SDK][service-fabric-sdk] herunter. Das SDK enthält den OneBox-Cluster-Manager, sodass Sie die Anwendung „Surveys“ lokal mit uneingeschränktem F5-Debugging bereitstellen und testen können.

<!-- links -->
[azure-sdk]: https://azure.microsoft.com/downloads/archive-net-downloads/
[container-scenarios]: /azure/service-fabric/service-fabric-containers-overview
[kestrel]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/kestrel?tabs=aspnetcore2x
[kestrel-intro]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/kestrel?tabs=aspnetcore1x
[migrate-from-cloud-services]: migrate-from-cloud-services.md
[monitoring-diagnostics]: /azure/service-fabric/service-fabric-diagnostics-overview
[reliable-concurrent-queue]: /azure/service-fabric/service-fabric-reliable-services-reliable-concurrent-queue
[reverse-proxy]: /azure/service-fabric/service-fabric-reverseproxy
[sample-code]: https://github.com/mspnp/cloud-services-to-service-fabric/tree/master/servicefabric-phase-2
[service-fabric]: /azure/service-fabric/service-fabric-get-started
[service-fabric-sdk]: /azure/service-fabric/service-fabric-get-started
[weblistener]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/weblistener
