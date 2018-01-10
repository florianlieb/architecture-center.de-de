---
title: API-Gateways
description: API-Gateways in Microservices
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: 6483d416363e24f4084d6b856847a740bf4054d9
ms.sourcegitcommit: a8453c4bc7c870fa1a12bb3c02e3b310db87530c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/29/2017
---
# <a name="designing-microservices-api-gateways"></a>Entwerfen von Microservices: API-Gateways

In einer Microservices-Architektur interagiert ein Client ggf. mit mehreren Front-End-Diensten. Woher weiß ein Client aber, welche Endpunkte er aufrufen soll? Was passiert, wenn neue Dienste eingeführt oder vorhandene Dienste umgestaltet werden? Wie behandeln Dienste SSL-Beendigung, Authentifizierung und andere Anliegen? Hier kann ein *API-Gateway* hilfreich sein. 

![](./images/gateway.png)

## <a name="what-is-an-api-gateway"></a>Was ist ein API-Gateway?

Ein API-Gateway befindet sich zwischen Clients und Diensten. Es fungiert als Reverseproxy und leitet Anforderungen von Clients an Dienste weiter. Darüber hinaus kann es verschiedene übergreifende Aufgaben wie Authentifizierung, SSL-Beendigung und Ratenbegrenzung übernehmen. Wenn Sie kein Gateway bereitstellen, müssen Clients Anforderungen direkt an Front-End-Dienste senden. Der direkte Kontakt zwischen Diensten und Clients kann jedoch problematisch sein:

- Er kann zu komplexem Clientcode führen. Der Client muss den Überblick über mehrere Endpunkte behalten und über eine robuste Fehlerbehandlung verfügen. 
- Er führt zu einer Kopplung zwischen Client und Back-End. Der Client benötigt Informationen zur Aufspaltung der einzelnen Dienste. Dies erschwert die Verwaltung des Clients sowie die Umgestaltung von Diensten.
- Für einen einzelnen Vorgang müssen möglicherweise mehrere Dienste aufgerufen werden. Dies kann mehrere Netzwerkroundtrips zwischen Client und Server erforderlich machen und die Wartezeit erheblich erhöhen. 
- Jeder öffentliche Dienst muss Aspekte wie Authentifizierung, SSL und Clientratenbegrenzung behandeln. 
- Dienste müssen ein clientfreundliches Protokoll wie HTTP oder WebSocket verfügbar machen. Dies schränkt die in Frage kommenden [Kommunikationsprotokolle](./interservice-communication.md) ein. 
- Dienste mit öffentlichen Endpunkten stellen ein potenzielles Angriffsziel dar und müssen gehärtet werden.

Ein Gateway trägt zur Bewältigung dieser Herausforderungen bei, indem es Clients von Diensten entkoppelt. Gateways können eine ganze Reihe unterschiedlicher Funktionen übernehmen, die Sie möglicherweise gar nicht alle benötigen. Die Funktionen lassen sich in folgende Entwurfsmuster unterteilen:

[Gatewayrouting:](../patterns/gateway-routing.md) Verwenden Sie das Gateway als Reverseproxy, um Anforderungen mittels Layer-7-Routing an Back-End-Dienste weiterzuleiten. Das Gateway stellt einen einzelnen Endpunkt für Clients bereit und trägt zur Entkopplung von Clients und Diensten bei. 

[Gatewayaggregation:](../patterns/gateway-aggregation.md) Aggregieren Sie mehrere Anforderungen mithilfe des Gateways in einer einzelnen Anforderung. Dieses Muster kommt zur Anwendung, wenn für einen einzelnen Vorgang mehrere Back-End-Dienste aufgerufen werden müssen. Der Client sendet eine einzelne Anforderung an das Gateway. Das Gateway sendet Anforderungen an die verschiedenen Back-End-Systeme, aggregiert die Ergebnisse und gibt sie an den Client zurück. Dadurch verringert sich die Kommunikation zwischen Client Back-End. 

[Gatewayabladung:](../patterns/gateway-offloading.md) Nutzen Sie das Gateway, um Funktionen einzelner Dienste (insbesondere übergreifende Aspekte) an das Gateway auszulagern. Es kann hilfreich sein, diese Funktionen an einem zentralen Ort zusammenzufassen, anstatt ihre Implementierung den einzelnen Diensten zu überlassen. Das gilt insbesondere für Features wie Authentifizierung und Autorisierung, deren korrekte Implementierung spezielles Fachwissen voraussetzt. 

Folgende Funktionen können beispielsweise an ein Gateway ausgelagert werden:

- SSL-Beendigung
- Authentifizierung
- IP-Whitelisting
- Clientratenbegrenzung (Drosselung)
- Protokollierung und Überwachung
- Zwischenspeicherung von Antworten
- Web Application Firewall
- GZIP-Komprimierung
- Wartung statischer Inhalte

## <a name="choosing-a-gateway-technology"></a>Wählen einer Gatewaytechnologie

In diesem Abschnitt finden Sie einige Optionen für die Implementierung eines API-Gateways in Ihrer Anwendung.

- **Reverseproxyserver:** Nginx und HAProxy sind gängige Reverseproxyserver, die Features wie Lastenausgleich, SSL und Layer-7-Routing unterstützen. Beides sind kostenlose Open-Source-Produkte, die auch als kostenpflichtige Editionen mit zusätzlichen Features und Unterstützungsoptionen erhältlich sind. Nginx und HAProxy sind ausgereifte, leistungsfähige Produkte mit großem Funktionsumfang. Sie können mit Drittanbietermodulen oder benutzerdefinierten Lua-Skripts erweitert werden. Nginx unterstützt auch ein JavaScript-basiertes Skripterstellungsmodul namens NginScript.

- **Dienstnetz-Eingangscontroller:** Ziehen Sie bei Verwendung eines Dienstnetzes wie linkerd oder Istio die Features in Erwägung, die vom Eingangscontroller für das Dienstnetz bereitgestellt werden. Der Istio-Eingangscontroller unterstützt beispielsweise Layer-7-Routing, HTTP-Umleitungen, Wiederholungen und andere Features. 

- [Azure Application Gateway:](/azure/application-gateway/) Application Gateway ist ein verwalteter Lastenausgleichsdienst, der für Layer-7-Routing und SSL-Beendigung geeignet ist. Darüber hinaus verfügt er über eine Web Application Firewall (WAF).

- [Azure API Management:](/azure/api-management/) API Management ist eine sofort einsatzbereite Lösung zum Veröffentlichen von APIs für externe und interne Kunden. Sie bietet hilfreiche Features für die Verwaltung einer öffentlichen API wie Ratenbeschränkung, IP-Whitelisting und Authentifizierung mit Azure Active Directory oder anderen Identitätsanbietern. Da API Management über keinen Lastenausgleich verfügt, sollte die Lösung mit einem Lastenausgleich wie Application Gateway oder einem Reverseproxy kombiniert werden.

Berücksichtigen Sie bei der Wahl einer Gatewaytechnologie Folgendes:

**Features:** Die oben aufgeführten Optionen unterstützen alle Layer-7-Routing. Die Unterstützung anderer Features variiert. Je nach benötigten Features können Sie mehrere Gateways bereitstellen. 

**Bereitstellung:** Azure Application Gateway und API Management sind verwaltete Dienste. Nginx und HAProxy werden in der Regel in Containern innerhalb des Clusters ausgeführt, können aber auch auf dedizierten virtuellen Computern außerhalb des Clusters bereitgestellt werden. Dadurch wird das Gateway vom Rest der Workload isoliert, verursacht aber auch einen höheren Verwaltungsaufwand.

**Verwaltung:** Wenn Dienste aktualisiert oder neue Dienste hinzugefügt werden, müssen ggf. die Gatewayroutingregeln aktualisiert werden. Überlegen Sie sich, wie dieser Prozess gehandhabt werden soll. Ähnliche Überlegungen müssen für den Umgang mit SSL-Zertifikaten, IP-Whitelists und anderen Konfigurationsaspekten angestellt werden.

## <a name="deployment-considerations"></a>Überlegungen zur Bereitstellung

### <a name="deploying-nginx-or-haproxy-to-kubernetes"></a>Bereitstellen von Nginx oder HAProxy für Kubernetes

Sie können Nginx oder HAProxy für Kubernetes als [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) oder [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) bereitstellen, das das Nginx- oder HAProxy-Containerimage angibt. Speichern Sie die Konfigurationsdatei für den Proxy mithilfe eines ConfigMap-Elements, und binden Sie dieses als Volume ein. Erstellen Sie einen LoadBalancer-Dienst, um das Gateway über einen Azure Load Balancer verfügbar zu machen. 

<!-- - Configure a readiness probe that serves a static file from the gateway (rather than routing to another service). -->

Alternativ können Sie auch einen Eingangscontroller erstellen. Ein Eingangscontroller ist eine Kubernetes-Ressource, die einen Lastenausgleich oder Reverseproxyserver bereitstellt. Hierfür stehen mehrere Implementierungen (unter anderem Nginx und HAProxy) zur Verfügung. Eine separate Ressource namens „Ingress“ (Eingang) definiert die Einstellungen für den Eingangscontroller (beispielsweise Routingregeln und TLS-Zertifikate). Dadurch müssen Sie keine komplexen spezifischen Konfigurationsdateien für eine bestimmte Proxyservertechnologie verwalten. Eingangscontroller von Kubernetes befinden sich zum Zeitpunkt der Artikelerstellung noch in der Betaphase. Ihre Entwicklung ist also noch nicht abgeschlossen.

Das Gateway ist ein potenzieller Engpass oder Single Point of Failure des Systems. Stellen Sie daher immer mindestens zwei Replikate bereit, um eine hohe Verfügbarkeit zu gewährleisten. Abhängig von der Last müssen die Replikate ggf. weiter horizontal hochskaliert werden. 

Unter Umständen empfiehlt es sich auch, das Gateway auf einer dedizierten Gruppe von Knoten im Cluster auszuführen. Dieser Ansatz bietet folgende Vorteile:

- Isolation: Sämtlicher eingehender Datenverkehr geht an eine feste Gruppe von Knoten, die von Back-End-Diensten isoliert werden können.

- Stabile Konfiguration: Im Falle einer falschen Gatewaykonfiguration ist möglicherweise die gesamte Anwendung nicht verfügbar. 

- Leistung: Aus Leistungsgründen empfiehlt sich unter Umständen die Verwendung einer bestimmten VM-Konfiguration für das Gateway.

<!-- - Load balancing. You can configure the external load balancer so that requests always go to a gateway node. That can save a network hop, which would otherwise happen whenever a request lands on a node that isn't running a gateway pod. This consideration applies mainly to large clusters, where the gateway runs on a relatively small fraction of the total nodes. In Azure Container Service (ACS), this approach currently requires [ACS Engine](https://github.com/Azure/acs-engine)) which allows you to create multiple agent pools. Then you can deploy the gateway as a DaemonSet to the front-end pool. -->

### <a name="azure-application-gateway"></a>Azure Application Gateway

So verbinden Sie Application Gateway mit einem Kubernetes-Cluster in Azure:

1. Erstellen Sie im Cluster-VNet ein leeres Subnetz.
2. Stellen Sie Application Gateway bereit.
3. Erstellen Sie einen Kubernetes-Dienst vom Typ [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport). Dadurch wird der Dienst auf jedem Knoten verfügbar gemacht, sodass er von außerhalb des Clusters erreichbar ist. Ein Lastenausgleich wird nicht erstellt.
5. Rufen Sie die zugewiesene Portnummer für den Dienst ab.
6. Fügen Sie eine Application Gateway-Regel hinzu, die folgende Kriterien erfüllt:
    - Der Back-End-Adresspool enthält die virtuellen Agent-Computer.
    - Die HTTP-Einstellung gibt die Dienstportnummer an.
    - Der Gatewaylistener lauscht an den Ports 80/443.
    
Legen Sie die Instanzanzahl mindestens auf „2“ fest, um für hohe Verfügbarkeit zu sorgen.

### <a name="azure-api-management"></a>Azure API Management 

So verbinden Sie API Management mit einem Kubernetes-Cluster in Azure:

1. Erstellen Sie im Cluster-VNet ein leeres Subnetz.
2. Stellen Sie in diesem Subnetz API Management bereit.
3. Erstellen Sie einen Kubernetes-Dienst vom Typ „LoadBalancer“. Verwenden Sie die [Anmerkung für einen internen Lastenausgleich](https://kubernetes.io/docs/concepts/services-networking/service/#internal-load-balancer), um anstelle des standardmäßigen Lastenausgleichs mit Internetzugriff einen internen Lastenausgleich zu erstellen.
4. Ermitteln Sie mithilfe von kubectl oder der Azure-Befehlszeilenschnittstelle die private IP-Adresse des internen Lastenausgleichs.
5. Erstellen Sie mithilfe von API Management eine API für die Weiterleitung an die private IP-Adresse des Lastenausgleichs.

Kombinieren Sie API Management ggf. mit einem Reverseproxy (Nginx, HAProxy oder Azure Application Gateway). Informationen zur Verwendung von API Management mit Application Gateway finden Sie unter [Integrieren von API Management in ein internes VNET mit Application Gateway](/azure/api-management/api-management-howto-integrate-internal-vnet-appgateway).

> [!div class="nextstepaction"]
> [Protokollierung und Überwachung](./logging-monitoring.md)
