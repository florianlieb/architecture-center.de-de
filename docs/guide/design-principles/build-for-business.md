---
title: Ausrichtung des Entwurfs auf die Unternehmensanforderungen
description: Jede Entwurfsentscheidung muss durch eine geschäftliche Anforderung gerechtfertigt sein
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 110a441ae74334d212a717da2cb038d60b24bb1f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="build-for-the-needs-of-the-business"></a>Ausrichtung des Entwurfs auf die Unternehmensanforderungen

## <a name="every-design-decision-must-be-justified-by-a-business-requirement"></a>Jede Entwurfsentscheidung muss durch eine geschäftliche Anforderung gerechtfertigt sein

Dieses Entwurfsprinzip ist offensichtlich, aber es ist von entscheidender Bedeutung, sich beim Entwerfen einer Lösung daran zu halten. Rechnen Sie mit Millionen von Benutzern oder mit einigen Tausend? Ist ein einstündiger Ausfall der Anwendung akzeptabel? Erwarten Sie beim Datenverkehr große Bedarfsspitzen, oder kann die Arbeitsauslastung gut vorhergesagt werden? Letztendlich muss jede Entwurfsentscheidung durch eine geschäftliche Anforderung gerechtfertigt sein. 

## <a name="recommendations"></a>Empfehlungen

**Definieren Sie Geschäftsziele**, z.B. Recovery Time Objective (RTO), Recovery Point Objective (RPO) und die maximal tolerierbare Ausfalldauer (Maximum Tolerable Outage, MTO). Diese Werte sollten als Grundlage für Entscheidungen dienen, die in Bezug auf die Architektur getroffen werden. Zur Erreichung eines niedrigen RTO-Werts können Sie beispielsweise ein automatisiertes Failover in eine sekundäre Region implementieren. Wenn Ihre Lösung aber einen höheren RTO-Wert tolerieren kann, ist dieser Redundanzgrad ggf. nicht erforderlich.

**Dokumentieren Sie Vereinbarungen zum Servicelevel (SLAs) und Servicelevelziele (SLOs)**, z.B. Verfügbarkeits- und Leistungsmetriken. Sie erstellen ggf. eine Lösung mit einer Verfügbarkeit von 99,95%. Ist dies ausreichend? Die Antwort ist eine geschäftliche Entscheidung. 

**Modellieren Sie die Anwendung basierend auf dem Geschäftsbereich**. Analysieren Sie zuerst die Geschäftsanforderungen. Nutzen Sie diese Anforderungen, um die Anwendung zu modellieren. Erwägen Sie die Nutzung eines Entwurfsansatzes, der am Geschäftsbereich ausgerichtet ist (Domain-Driven Design, DDD), um [Domänenmodelle][domain-model] zu erstellen, die die Geschäftsprozesse und Anwendungsfälle widerspiegeln. 

**Erfassen Sie sowohl funktionsbezogene als auch nicht funktionsbezogene Anforderungen**. Bei funktionsbezogenen Anforderungen können Sie erkennen, ob sich die Anwendung richtig verhält. Bei nicht funktionsbezogenen Anforderungen können Sie erkennen, ob die Anwendung die entsprechenden Schritte *gut* ausführt. Stellen Sie vor allem sicher, dass Sie Ihre Anforderungen in Bezug auf die Skalierbarkeit, Verfügbarkeit und Wartezeit verstehen. Diese Anforderungen beeinflussen Entwurfsentscheidungen und die Wahl der Technologie.

**Teilen Sie die Workload auf**. Der Begriff „Workload“ steht in diesem Kontext für eine einzelne Funktion oder Computingaufgabe, die von anderen Aufgaben logisch getrennt werden kann. Für unterschiedliche Workloads gelten unter Umständen unterschiedliche Anforderungen in Bezug auf die Verfügbarkeit, Skalierbarkeit, Datenkonsistenz und Notfallwiederherstellung. 

**Planen Sie im Hinblick auf Wachstum**. Eine Lösung erfüllt ggf. Ihre derzeitigen Anforderungen in Bezug auf Benutzeranzahl, Transaktionsmenge, Datenspeicherung usw. Eine robuste Anwendung ist aber in der Lage, Wachstum ohne größere Änderungen an der Architektur zu bewältigen. Informationen hierzu finden Sie unter [Ausrichtung des Entwurfs auf horizontale Skalierung](scale-out.md) und [Umgehung von Beschränkungen durch Partitionierung](partition.md). Berücksichtigen Sie auch, dass sich Ihr Geschäftsmodell und die geschäftlichen Anforderungen im Laufe der Zeit voraussichtlich ändern. Wenn das Dienstmodell und die Datenmodelle einer Anwendung zu starr gestaltet sind, ist es schwierig, die Anwendung für neue Anwendungsfälle und Szenarien weiterzuentwickeln. Informationen hierzu finden Sie unter [Design for evolution](design-for-evolution.md) (Ausrichtung des Entwurfs auf die weitere Entwicklung).

**Verwalten Sie die Kosten**. Bei einer herkömmlichen Anwendung zahlen Sie für Hardware im Voraus (Investitionsaufwand (CAPEX)). Bei einer Cloudanwendung zahlen Sie für die Ressourcen, die Sie nutzen. Es ist wichtig, dass Sie sich mit dem Preismodell für die genutzten Dienste vertraut machen. Zu den Gesamtkosten gehören Netzwerkbandbreiten-Nutzung, Speicherung, IP-Adressen, Dienstnutzung und andere Faktoren. Weitere Informationen finden Sie in der [Azure-Preisübersicht][pricing]. Berücksichtigen Sie auch Ihre Betriebskosten. In der Cloud müssen Sie die Hardware oder andere Infrastruktur nicht verwalten, aber die Verwaltung Ihrer Anwendungen ist weiterhin erforderlich, z.B. DevOps, Reaktion auf Vorfälle, Notfallwiederherstellung usw. 

[domain-model]: https://martinfowler.com/eaaCatalog/domainModel.html
[pricing]: https://azure.microsoft.com/pricing/
