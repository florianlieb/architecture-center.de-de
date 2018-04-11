---
title: Checkliste für Resilienz
description: Checkliste, die Hinweise zu Überlegungen hinsichtlich der Resilienz während des Entwurfs bereitstellt.
author: petertaylor9999
ms.date: 01/10/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: ca4bf77c9348f6c656348d9cd61d3a1241d69ba8
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/08/2018
---
# <a name="resiliency-checklist"></a>Checkliste für Resilienz

Resilienz ist die Fähigkeit des Systems, nach Ausfällen für ein System eine Wiederherstellung durchzuführen und die Betriebsbereitschaft sicherzustellen. Sie gehört zu den [Säulen der Softwarequalität](../guide/pillars.md). Damit Sie Ihre Anwendung für Resilienz entwerfen können, müssen Sie eine Vielzahl von Fehlermodi, die auftreten können, einplanen und entschärfen. Verwenden Sie diese Prüfliste, um Ihre Anwendungsarchitektur vom Standpunkt der Resilienz aus zu überprüfen. Sehen Sie sich auch die [Checkliste für Resilienz für bestimmte Azure-Dienste](./resiliency-per-service.md) an.

## <a name="requirements"></a>Anforderungen

**Definieren Sie die Verfügbarkeitsanforderungen Ihrer Kunden.** Ihre Kunden haben Anforderungen an die Verfügbarkeit der Komponenten in der Anwendung, die Entwurf der Anwendung beeinflussen. Vereinbaren Sie mit den Kunden Verfügbarkeitsziele für die einzelnen Bestandteile der Anwendung, andernfalls erfüllt der Entwurf die Kundenerwartungen möglicherweise nicht. Weitere Informationen finden Sie unter [Definieren Ihrer Anforderungen an die Resilienz](../resiliency/index.md#defining-your-resiliency-requirements).

## <a name="application-design"></a>Anwendungsentwurf

**Führen Sie eine Analyse im Fehlermodus (FMA) für die Anwendung durch.** FMA ist ein Verfahren, um zu einem frühen Zeitpunkt in der Entwurfsphase Resilienz in einer Anwendung zu implementieren. Weitere Informationen finden Sie unter [Analyse im Fehlermodus][fma]. Ziele von FMA:  

* Identifizieren, welche Arten von Fehlern in einer Anwendung auftreten können.
* Erfassen möglicher Einflüsse und Auswirkungen der einzelnen Arten von Fehlern auf die Anwendung.
* Identifizieren von Strategien für die Wiederherstellung.
  

**Stellen Sie mehrere Instanzen von Diensten bereit.** Wenn Ihre Anwendung von einer einzelnen Instanz eines Diensts abhängig ist, entsteht dadurch ein Single Point of Failure. Durch die Bereitstellung von mehreren Instanzen werden Resilienz und Skalierbarkeit verbessert. Wählen Sie für [Azure App Service](/azure/app-service/app-service-value-prop-what-is/) einen [App Service-Plan](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/) aus, der mehrere Instanzen bietet. Konfigurieren Sie für Azure Cloud Services alle Rollen so, dass sie [mehrere Instanzen](/azure/cloud-services/cloud-services-choose-me/#scaling-and-management) verwenden. Stellen Sie für [Azure Virtual Machines (VMs)](/azure/virtual-machines/virtual-machines-windows-about/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) sicher, dass die VM-Architektur mehr als eine VM enthält und dass jede VM zu einer [Verfügbarkeitsgruppe][availability-sets] gehört.   

**Verwenden Sie die automatische Skalierung, um auf den Anstieg der Last zu reagieren.** Wenn Ihre Anwendung nicht so konfiguriert ist, dass sie beim Anstieg der Last automatisch horizontal hochskaliert wird, ist es möglich, dass Dienste der Anwendung fehlschlagen, wenn zu viele Benutzeranforderungen auftreten. Weitere Informationen finden Sie hier:

* Allgemein: [Checkliste für die Skalierbarkeit](./scalability.md)
* Azure App Service: [Manuelles oder automatisches Skalieren der Instanzenzahl][app-service-autoscale]
* Cloud Services: [Automatisches Skalieren eines Clouddiensts][cloud-service-autoscale]
* Virtual Machines: [Automatische Skalierung und Skalierungsgruppen für virtuelle Computer][vmss-autoscale]

**Verwenden Sie den Lastenausgleich, um Anforderungen zu verteilen.** Mit dem Lastenausgleich werden die Anforderungen Ihrer Anwendung auf fehlerfreie Dienstinstanzen verteilt, indem fehlerhafte Instanzen aus der Rotation entfernt werden. Wenn der Dienst Azure App Service oder Azure Cloud Services verwendet, erfolgt bereits ein Lastenausgleich. Wenn Ihre Anwendung allerdings Azure-VMs verwendet, müssen Sie einen Lastenausgleich bereitstellen. Weitere Informationen finden Sie in der Übersicht über [Azure Load Balancer](/azure/load-balancer/load-balancer-overview/).

**Konfigurieren Sie Azure-Anwendungsgateways für die Verwendung mehrerer Instanzen.** Abhängig von den Anforderungen Ihrer Anwendung kann [Azure Application Gateway](/azure/application-gateway/application-gateway-introduction/) für die Verteilung von Anforderungen an die Dienste der Anwendung besser geeignet sein. Allerdings sind einzelne Instanzen des Application Gateway-Diensts nicht durch einen SLA garantiert. Daher ist es möglich, dass bei einem Ausfall der Application Gateway-Instanz die Anwendung ausfällt. Stellen Sie mehr als eine mittlere oder größere Application Gateway-Instanz bereit, um die Verfügbarkeit des Diensts gemäß den Bedingungen der [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/) zu gewährleisten.

**Verwenden Sie Verfügbarkeitsgruppen für jede Logikschicht.** Das Platzieren Ihrer Instanzen in einer [Verfügbarkeitsgruppe][availability-sets] bietet eine höhere [SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/). 

**Erwägen Sie die Bereitstellung Ihrer Anwendung in mehreren Regionen.** Wenn Ihre Anwendung in einer einzelnen Region bereitgestellt wird, ist Ihre Anwendung nicht mehr verfügbar, falls doch einmal die gesamte Region ausfällt. Dies kann unter den Bestimmungen der SLA für Ihre Anwendung nicht akzeptabel sein. Wenn dies der Fall ist, erwägen Sie die Bereitstellung Ihrer Anwendung und der zugehörigen Dienste in mehreren Regionen. Für eine Bereitstellung in mehreren Regionen kann ein Aktiv/Aktiv-Muster (Verteilung von Anforderungen über mehrere aktive Instanzen) oder ein Aktiv/Passiv-Muster (eine "warme" Instanz wird für den Fall, dass die primäre Instanz ausfällt, als Reserve vorgehalten) verwendet werden. Es wird empfohlen, dass Sie mehrere Instanzen der Dienste Ihrer Anwendung in Regionspaaren bereitstellen. Weitere Informationen finden Sie unter [Geschäftskontinuität und Notfallwiederherstellung: Azure-Regionspaare](/azure/best-practices-availability-paired-regions).

**Verwenden Sie Azure Traffic Manager zum Weiterleiten des Datenverkehrs für Ihre Anwendung an verschiedene Regionen.**  [Azure Traffic Manager][traffic-manager] führt den Lastenausgleich auf DNS-Ebene durch und leitet den Datenverkehr an verschiedene Regionen weiter. Die Weiterleitung basiert auf der angegebenen Methode für das [Datenverkehrrouting][traffic-manager-routing] und der Integrität der Endpunkte Ihrer Anwendung. Ohne Traffic Manager ist Ihre Bereitstellung auf eine Region beschränkt. Dadurch wird die Skalierung eingeschränkt und die Latenz für einige Benutzer erhöht. Zudem entstehen Ausfallzeiten der Anwendung, falls in der gesamten Region der Dienst unterbrochen wird.

**Konfigurieren und testen Sie Integritätstest für Ihre Lastenausgleichsmodule und Traffic Manager.** Stellen Sie sicher, dass Ihre Integritätslogik die kritischen Teile des Systems überprüft und in geeigneter Weise auf Integritätstests reagiert.

* Die Integritätstest für [Azure Traffic Manager][traffic-manager] und [Azure Load Balancer][load-balancer] dienen einem bestimmten Zweck. Bei Traffic Manager bestimmt der Integritätstest, ob ein Failover auf eine andere Region erforderlich ist. Beim Lastenausgleich bestimmt er, ob eine VM aus der Rotation entfernt werden muss.      
* Bei einem Traffic Manager-Test sollte Ihr Integritätsendpunkt alle kritischen Abhängigkeiten überprüfen, die innerhalb der gleichen Region bereitgestellt wurden und deren Ausfall ein Failover auf eine andere Region auslösen sollte.  
* Bei einem Lastenausgleich sollte der Integritätsendpunkt die Integrität der VM melden. Nehmen Sie keine weiteren Schichten oder externen Dienste auf. Andernfalls verursacht ein Fehler, der außerhalb der VM auftritt, dass der Lastenausgleich die VM aus der Rotation entfernt.
* Anweisungen zum Implementieren der Integritätsüberwachung in Ihrer Anwendung finden Sie unter [Überwachungsmuster für den Integritätsendpunkt](https://msdn.microsoft.com/library/dn589789.aspx).

**Überwachen Sie Dienste von Drittanbietern.** Wenn Ihre Anwendung Abhängigkeiten von den Diensten von Drittanbietern aufweist, identifizieren Sie, wo und wie Fehler in diesen Diensten von Drittanbietern auftreten können und welche Auswirkungen diese Fehler auf die Anwendung haben. Der Dienst eines Drittanbieters weist möglicherweise keine Überwachungs- und Diagnosefunktionen auf. Daher ist es wichtig, dass Ihre Aufrufe dieser Dienste protokolliert werden und über einen eindeutigen Bezeichner mit der Integritäts- und Diagnoseprotokollierung Ihrer Anwendung korreliert werden. Weitere Informationen zu bewährten Verfahren für Überwachung und Diagnose finden Sie unter [Anleitung zur Überwachung und Diagnose][monitoring-and-diagnostics-guidance].

**Stellen Sie sicher, dass alle von Ihnen genutzten Dienste von Drittanbietern eine SLA bereitstellen.** Wenn Ihre Anwendung von einem Dienst eines Drittanbieters abhängt, der Drittanbieter aber keine Garantie für die Verfügbarkeit in Form einer SLA bietet, kann die Verfügbarkeit Ihrer Anwendung auch nicht garantiert werden. Ihre SLA ist nur so gut wie die Komponente mit der geringsten Verfügbarkeit in Ihrer Anwendung.

**Implementieren Sie ggf. Resilienzmuster für Remotevorgänge.** Wenn Ihre Anwendung von der Kommunikation zwischen Remotediensten abhängig ist, folgen Sie den Entwurfsmustern für den Umgang mit vorübergehenden Fehlern wie z.B. [Wiederholungsmuster][retry-pattern] und [Trennschalter-Muster][circuit-breaker]. Weitere Informationen finden Sie unter [Resilienzstrategien](../resiliency/index.md#resiliency-strategies).

**Implementieren Sie nach Möglichkeit asynchrone Vorgänge.** Synchrone Vorgänge können zu einer Monopolisierung von Ressourcen führen und andere Vorgänge blockieren, während der Aufrufer auf den Abschluss des Vorgangs wartet. Entwerfen Sie die einzelnen Teile der Anwendung so, dass nach Möglichkeit asynchrone Vorgänge genutzt werden. Weitere Informationen zum Implementieren der asynchronen Programmierung in C# finden Sie unter [Asynchrone Programmierung mit Async und Await][asynchronous-c-sharp].

## <a name="data-management"></a>Datenverwaltung

**Untersuchen Sie die Replikationsmethoden für die Datenquellen Ihrer Anwendung.** Ihre Anwendungsdaten werden in unterschiedlichen Datenquellen gespeichert und weisen unterschiedliche Anforderungen an die Verfügbarkeit auf. Bewerten Sie die Replikationsmethoden für jeden Typ von Datenspeicher in Azure, beispielsweise die [Azure-Speicherreplikation](/azure/storage/storage-redundancy/) und die [aktive Georeplikation in Azure SQL-Datenbank](/azure/sql-database/sql-database-geo-replication-overview/), um sicherzustellen, dass die Datenanforderungen Ihrer Anwendung erfüllt werden.

**Stellen Sie sicher, dass kein einzelnes Benutzerkonto Zugriff auf Produktions- und Sicherungsdaten hat.** Die Datensicherungen sind gefährdet, wenn ein einzelnes Benutzerkonto über die Berechtigung zum Schreiben in Produktions- und Sicherungsquellen verfügt. Ein böswilliger Benutzer könnte absichtlich alle Daten löschen, einem normalen Benutzer könnte dies versehentlich passieren. Entwerfen Sie Ihre Anwendung mit Einschränkungen der Berechtigungen für jedes Benutzerkonto, sodass nur die Benutzer, für die Schreibzugriff erforderlich ist, über Schreibzugriff verfügen, und auch nur auf Produktions- oder Sicherungsquellen, aber nicht auf beide.

**Dokumentieren Sie den Failover- und Failbackprozess der Datenquellen, und testen Sie ihn.** Bei einem schwerwiegenden Ausfall der Datenquelle muss ein menschlicher Operator einer Reihe von dokumentierten Anweisungen folgen, um ein Failover auf eine neue Datenquelle auszuführen. Wenn die dokumentierten Schritte Fehler aufweisen, kann ein Operator sie nicht erfolgreich befolgen und kein Failover für die Ressource ausführen. Testen Sie die Schritte der Anweisung regelmäßig, um sicherzustellen, dass ein Operator, der sie befolgt, erfolgreich ein Failover und ein Failback der Datenquelle ausführen kann.

**Überprüfen Sie die Datensicherungen.** Überprüfen Sie regelmäßig, ob Ihre Sicherungsdaten den Erwartungen entsprechen, indem Sie mit einem Skript Datenintegrität, Schema und Abfragen überprüfen. Eine Sicherung ist nur sinnvoll, wenn sie verwendet werden kann, um Ihre Datenquellen wiederherzustellen. Protokollieren und melden Sie Inkonsistenzen, damit der Sicherungsdienst repariert werden kann.

**Erwägen Sie die Verwendung eines Speicherkontotyps, der georedundant ist.** Daten, die in einem Azure-Speicherkonto gespeichert sind, werden immer lokal repliziert. Bei der Bereitstellung eines Speicherkontos können Sie jedoch aus mehreren Replikationsstrategien auswählen. Wählen Sie einen [georedundanten Speicher mit Lesezugriff (RA-GRS) in Azure](/azure/storage/storage-redundancy/#read-access-geo-redundant-storage) aus, um Ihre Anwendungsdaten für den seltenen Fall zu schützen, dass eine vollständige Region nicht mehr verfügbar ist.

> [!NOTE]
> Verlassen Sie sich für VMs nicht auf die RA-GRS-Replikation, um die VM-Datenträger (VHD-Dateien) wiederherzustellen. Verwenden Sie stattdessen [Azure Backup][azure-backup].   
>
>

## <a name="security"></a>Sicherheit

**Implementieren Sie Schutz auf Anwendungsebene vor verteilten Denial-of-Service-Angriffen (DDoS).** Azure-Dienste sind auf Netzwerkebene vor DDos-Angriffen geschützt. Allerdings kann Azure nicht vor Angriffen auf Anwendungsebene schützten, da es schwierig ist, zwischen echten und böswilligen Benutzeranforderungen zu unterscheiden. Weitere Informationen zum Schutz vor DDoS-Angriffen auf Anwendungsebene finden Sie im Abschnitt über den Schutz vor DDoS im Dokument [Microsoft Azure-Netzwerksicherheit](http://download.microsoft.com/download/C/A/3/CA3FC5C0-ECE0-4F87-BF4B-D74064A00846/AzureNetworkSecurity_v3_Feb2015.pdf) (PDF-Download).

**Implementieren Sie das Prinzip der geringsten Rechte für den Zugriff auf die Ressourcen der Anwendung.** Die Standardeinstellung für den Zugriff auf die Ressourcen der Anwendung sollte möglichst restriktiv sein. Erteilen Sie höhere Berechtigungen über spezielle Genehmigungen. Das standardmäßige Gewähren von übermäßigem Zugriff auf Ressourcen der Anwendung kann dazu führen, dass eine Person absichtlich oder versehentlich Ressourcen löscht. Azure bietet [rollenbasierte Zugriffssteuerung](/azure/active-directory/role-based-access-built-in-roles/) zum Verwalten von Benutzerberechtigungen. Aber es ist wichtig sicherzustellen, dass die geringsten Berechtigungen für solche Ressourcen gewährt werden, die eigene Berechtigungssysteme aufweisen, wie z.B. SQL Server.

## <a name="testing"></a>Testen

**Führen Sie Failover- und Failbacktests für die Anwendung aus.** Wenn Sie Failover und Failback nicht vollständig getestet haben, können Sie nicht sicher sein, dass die abhängigen Dienste in der Anwendung bei einer Notfallwiederherstellung in synchronisierter Weise wiederhergestellt werden. Stellen Sie sicher, dass für die abhängigen Dienste Ihrer Anwendung Failover und Failback in der richtigen Reihenfolge ausgeführt werden.

**Führen Sie Fault Injection-Tests für Ihre Anwendung aus.** Die Anwendung kann aus vielen verschiedenen Ursachen ausfallen, z.B. Ablauf des Zertifikats, Auslastung der Systemressourcen in einer VM oder Speicherfehler. Testen Sie Ihre Anwendung in einer Umgebung, die der Produktion möglichst ähnlich ist, indem Sie echte Fehler simulieren oder auslösen. Löschen Sie z.B Zertifikate, beanspruchen Sie die Systemressourcen künstlich, oder löschen Sie eine Speicherquelle. Überprüfen Sie die Fähigkeit Ihrer Anwendung zur Wiederherstellung nach allen Typen von Fehlern, allein und in Kombination. Stellen Sie sicher, dass Fehler nicht im System weitergegeben oder kaskadiert werden.

**Führen Sie Tests in der Produktion mithilfe von synthetischen und echten Benutzerdaten aus.** Test und Produktion sind selten identisch, daher ist es wichtig, eine Blau/Grün- oder eine Canary-Bereitstellung zu verwenden und Ihre Anwendung in der Produktion zu testen. Dadurch können Sie Ihre Anwendung in der Produktion unter echter Last testen und sicherstellen, dass sie wie erwartet funktioniert, wenn sie vollständig bereitgestellt wird.

## <a name="deployment"></a>Bereitstellung

**Dokumentieren Sie den Freigabeprozess für Ihre Anwendung.** Ohne detaillierte Dokumentation des Freigabeprozesses kann ein Operator ein fehlerhaftes Update bereitstellen oder Einstellungen für Ihre Anwendung nicht ordnungsgemäß konfigurieren. Definieren und dokumentieren Sie den Freigabeprozess eindeutig, und stellen Sie sicher, dass er für das gesamte Betriebsteam verfügbar ist. 

**Automatisieren Sie den Bereitstellungsprozess für Ihre Anwendung.** Wenn Ihr Betriebspersonal zum manuellen Bereitstellen der Anwendung erforderlich ist, können Benutzerfehler dazu führen, dass die Bereitstellung fehlschlägt. 

**Entwerfen Sie den Freigabeprozess so, dass die Verfügbarkeit der Anwendung maximiert wird.** Wenn es für Ihren Freigabeprozess erforderlich ist, dass Dienste während der Bereitstellung offline geschaltet werden, ist die Anwendung erst wieder verfügbar, wenn sie wieder online geschaltet werden. Verwenden Sie die Bereitstellungsverfahren [Blau/Grün](http://martinfowler.com/bliki/BlueGreenDeployment.html) oder [Canary-Release](http://martinfowler.com/bliki/CanaryRelease.html) zum Bereitstellen der Anwendung in der Produktion. Beide Verfahren umfassen das Bereitstellen des Freigabecodes zusammen mit dem Produktionscode, damit Benutzer des Freigabecodes im Fall eines Fehlers an den Produktionscode umgeleitet werden können.

**Protokollieren und überwachen Sie die Bereitstellungen Ihrer Anwendung.** Bei Verwendung der Verfahren für Stagingbereitstellungen wie Blau/Grün oder Canary-Releases gibt es mehr als eine Version der Anwendung, die in der Produktion ausgeführt wird. Wenn ein Problem auftreten sollte, ist es entscheidend zu ermitteln, welche Version der Anwendung ein Problem verursacht. Implementieren Sie eine stabile Protokollierungsstrategie, um so viele versionsspezifische Informationen wie möglich zu erfassen.

**Bereiten Sie einen Zurücksetzungsplan für die Bereitstellung vor.** Es ist möglich, dass Ihre Anwendungsbereitstellung fehlschlägt und dazu führt, dass Ihre Anwendung nicht mehr verfügbar ist. Entwerfen Sie einen Zurücksetzungsprozess, um zur letzten als funktionierend bekannten Version zurückzukehren und Ausfallzeiten zu minimieren. 

## <a name="operations"></a>Vorgänge

**Implementieren Sie bewährte Methoden für die Überwachung und Warnungen in Ihrer Anwendung.** Ohne ordnungsgemäße Überwachung, Diagnose und Warnungen gibt es keine Möglichkeit, Fehler in der Anwendung zu erkennen und einen Operator aufzufordern, sie zu beheben. Weitere Informationen finden Sie unter [Anleitung zur Überwachung und Diagnose][monitoring-and-diagnostics-guidance].

**Messen Sie Statistiken zu Remoteaufrufen, und stellen Sie die Informationen dem Anwendungsteam zur Verfügung.**  Wenn Sie Statistiken zu Remoteaufrufen nicht in Echtzeit nachverfolgen und melden und keine einfache Möglichkeit bieten, diese Informationen zu überprüfen, verfügt das Betriebsteam nicht über unmittelbare Einblicke in die Integrität Ihrer Anwendung. Und wenn Sie nur Durchschnittszeiten von Remoteaufrufen messen, haben Sie nicht genügend Informationen, um Probleme in den Diensten aufzudecken. Fassen Sie Metriken zu Remoteaufrufen wie Latenz, Durchsatz und Fehler in den 99 und 95 Quantilen zusammen. Führen Sie statistische Analysen von Metriken durch, um Fehler zu erkennen, die in den einzelnen Quantilen auftreten.

**Verfolgen Sie die Anzahl der vorübergehenden Ausnahmen und Wiederholungsversuche in einem geeigneten Zeitrahmen.** Wenn Sie vorübergehende Ausnahmen und Wiederholungsversuche im Zeitverlauf nicht nachverfolgen und überwachen, ist es möglich, dass ein Problem oder Fehler durch die Wiederholungslogik Ihrer Anwendung verborgen bleibt. Anders ausgedrückt: Wenn Ihre Überwachung und Protokollierung nur den Erfolg oder das Fehlschlagen eines Vorgangs aufzeigt, bleibt die Tatsache verborgen, dass der Vorgang aufgrund von Ausnahmen mehrere Male wiederholt werden musste. Ein Trend zunehmender Ausnahmen im Zeitverlauf weist darauf hin, dass der Dienst möglicherweise ein Problem hat und ausfallen kann. Weitere Informationen finden Sie unter [Anleitung zu dienstspezifischen Wiederholungsmechanismen][retry-service-guidance].

**Implementieren Sie ein Frühwarnsystem, das einen Operator benachrichtigt.** Identifizieren Sie Key Performance Indicators für die Integrität Ihrer Anwendung, z.B. vorübergehende Ausnahmen und Latenz von Remoteaufrufen, und legen Sie jeweils entsprechende Schwellenwerte fest. Senden Sie eine Warnung an Operatoren, wenn der Schwellenwert erreicht wird. Legen Sie diese Schwellenwerte auf Ebenen fest, auf denen Probleme identifiziert werden, bevor sie kritisch werden und eine Wiederherstellung erfordern.

**Stellen Sie sicher, dass mehrere Personen im Team geschult sind, die Anwendung zu überwachen und manuelle Wiederherstellungsschritte auszuführen.** Wenn nur ein einziger Operator im Team die Anwendung überwachen und Wiederherstellungsschritte starten kann, wird diese Person zu einem Single Point of Failure. Arbeiten Sie mehrere Personen in die Erkennung und Wiederherstellung ein, und stellen Sie sicher, dass immer mindestens eine dieser Personen verfügbar ist.

**Stellen Sie sicher, dass mit Ihrer Anwendung keine [Grenzwerte des Azure-Abonnements](/azure/azure-subscription-service-limits/) überschritten werden.** Bei Azure-Abonnements gelten Grenzwerte für bestimmte Ressourcentypen, z.B. die Anzahl von Ressourcengruppen, die Anzahl von Kernen und die Anzahl von Speicherkonten.  Wenn mit den Anforderungen Ihrer Anwendung Grenzwerte des Azure-Abonnements überschritten werden, erstellen Sie ein weiteres Azure-Abonnement für zusätzliche Ressourcen.

**Stellen Sie sicher, dass mit Ihrer Anwendung keine [Grenzwerte von Diensten](/azure/azure-subscription-service-limits/) überschritten werden.** Für einzelne Azure-Dienste gelten Nutzungsgrenzwerte, beispielsweise Grenzwerte für Speicher, Durchsatz, die Anzahl der Verbindungen, Anforderungen pro Sekunde und weitere Metriken. Ihre Anwendung schlägt fehl, wenn sie versucht, jenseits dieser Grenzwerte Ressourcen zu verwenden. Dies führt zu Diensteinschränkungen und möglicherweise zu Ausfallzeiten für die betroffenen Benutzer. Abhängig vom spezifischen Dienst und den Anforderungen Ihrer Anwendung können Sie diese Grenzwerte häufig durch zentrales Hochskalieren (z.B. die Auswahl eines anderen Tarifs) oder horizontales Skalieren (Hinzufügen neuer Instanzen) vermeiden.  

**Entwerfen Sie die Speicheranforderungen Ihrer Anwendung so, dass sie innerhalb der Skalierbarkeits- und Leistungsziele für Azure-Speicher liegen.** Azure-Speicher ist so konzipiert, dass er in vordefinierten Skalierbarkeits- und Leistungszielen verwendet werden kann. Entwerfen Sie Ihre Anwendung also für die Speichernutzung innerhalb dieser Ziele. Wenn Sie diese Ziele überschreiten, wird für Ihre Anwendung der Speicher eingeschränkt. Um dieses Problem zu beheben, stellen Sie zusätzliche Speicherkonten bereit. Wenn Sie den Grenzwert für Speicherkonten erreichen, stellen Sie zusätzliche Azure-Abonnements für zusätzliche Speicherkonten bereit. Weitere Informationen finden Sie unter [Skalierbarkeits- und Leistungszielen für Azure Storage](/azure/storage/storage-scalability-targets/).

**Wählen Sie die richtige VM-Größe für Ihre Anwendung aus.** Messen Sie die tatsächlichen Werte für CPU, Arbeitsspeicher, Datenträger und E/A Ihrer VMs in der Produktion, und stellen Sie sicher, dass die ausgewählte VM-Größe ausreichend ist. Falls nicht, können in Ihrer Anwendung Kapazitätsprobleme auftreten, wenn die VMs ihre Grenzwerte erreichen. VM-Größen werden ausführlich unter [Größen für virtuelle Computer in Azure](/azure/virtual-machines/virtual-machines-windows-sizes/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) beschrieben.

**Bestimmen Sie, ob die Workload der Anwendung im Zeitverlauf stabil ist oder schwankt.** Wenn Ihre Workload im Zeitverlauf schwankt, verwenden Sie Azure-VM-Skalierungsgruppen, um die Anzahl der VM-Instanzen automatisch zu skalieren. Andernfalls müssen Sie die VM-Anzahl manuell erhöhen oder verringern. Weitere Informationen finden Sie unter [Übersicht über VM-Skalierungsgruppen](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/).

**Wählen Sie die richtige Dienstebene für Azure SQL-Datenbank aus.** Wenn Ihre Anwendung Azure SQL-Datenbank verwendet, stellen Sie sicher, dass Sie die geeignete Dienstebene ausgewählt haben. Wenn Sie eine Ebene auswählen, die die Anforderungen Ihrer Anwendung an die Datenbanktransaktionseinheiten (DTU) nicht verarbeiten kann, wird die Datenverwendung eingeschränkt. Weitere Informationen zur Auswahl des richtigen Dienstplans finden Sie unter [SQL-Datenbankoptionen und -leistung: Grundlegendes zum Angebot in den einzelnen Tarifen](/azure/sql-database/sql-database-service-tiers/).

**Erstellen Sie einen Prozess für die Interaktion mit dem Azure-Support.** Wenn kein Prozess für die Kontaktaufnahme mit dem [Azure-Support](https://azure.microsoft.com/support/plans/) festgelegt ist, bevor der Bedarf entsteht, den Support zu kontaktieren, verlängert sich die Ausfallzeit, da der Supportprozess zum ersten Mal ausgeführt wird. Machen Sie den Prozess für die Kontaktaufnahme mit dem Support und die Eskalation von Problemen von Anfang an zu einem Teil der Resilienz Ihrer Anwendung.

**Stellen Sie sicher, dass Ihre Anwendung nicht mehr als die maximale Anzahl von Speicherkonten pro Abonnement verwendet.** Azure lässt maximal 200 Speicherkonten pro Abonnement zu. Wenn Ihre Anwendung mehr Speicherkonten erfordert, als derzeit im Abonnement verfügbar sind, müssen Sie ein neues Abonnement für zusätzliche Speicherkonten erstellen. Weitere Informationen finden Sie unter [Grenzwerte für Azure-Abonnements, -Dienste und -Kontingente sowie allgemeine Beschränkungen](/azure/azure-subscription-service-limits/#storage-limits).

**Stellen Sie sicher, dass Ihre Anwendung die Skalierbarkeitsziele für VM-Datenträger nicht überschreitet.** Eine Azure-IaaS-VM unterstützt das Anfügen von mehreren Datenträgern. Dies ist abhängig von verschiedenen Faktoren wie der VM-Größe und des Typs von Speicherkonto. Wenn Ihre Anwendung die Skalierbarkeitsziele für VM-Datenträger überschreitet, stellen Sie zusätzliche Speicherkonten für die VM-Datenträger bereit. Weitere Informationen finden Sie unter [Skalierbarkeits- und Leistungsziele für Azure Storage](/azure/storage/storage-scalability-targets/#scalability-targets-for-virtual-machine-disks).

## <a name="telemetry"></a>Telemetrie

**Protokollieren Sie Telemetriedaten, während die Anwendung in der Produktionsumgebung ausgeführt wird.** Erfassen Sie zuverlässige Telemetrieinformationen, während die Anwendung in der Produktionsumgebung ausgeführt wird. Andernfalls verfügen Sie nicht über ausreichende Informationen, um die Ursache von Problemen zu diagnostizieren, während Benutzer die Anwendung aktiv nutzen. Weitere Informationen finden Sie unter [Überwachung und Diagnose][monitoring-and-diagnostics-guidance].

**Implementieren Sie die Protokollierung mithilfe eines asynchronen Musters.** Wenn Protokollierungsvorgänge synchron sind, können sie Anwendungscode blockieren. Stellen Sie sicher, dass die Protokollierungsvorgänge als asynchrone Vorgänge implementiert werden.

**Korrelieren Sie Protokolldaten über Dienstgrenzen hinweg.** In einer typischen Anwendung mit n-Schichten kann eine Benutzeranforderung mehrere Dienstgrenzen überqueren. Beispiel: Eine Benutzeranforderung entsteht in der Regel in der Webebene, wird an die Geschäftsebene übergeben und schließlich in der Datenebene beibehalten. In komplexeren Szenarien kann eine Benutzeranforderung an viele verschiedene Dienste und Datenspeicher verteilt werden. Stellen Sie sicher, dass Ihr Protokollierungssystem Aufrufe über Dienstgrenzen hinweg korreliert, damit Sie die Anforderung in der gesamten Anwendung verfolgen können.

## <a name="azure-resources"></a>Azure-Ressourcen

**Verwenden Sie Azure Resource Manager-Vorlagen für die von Bereitstellung von Ressourcen.** Resource Manager-Vorlagen erleichtern das Automatisieren von Bereitstellungen über PowerShell oder die Azure-CLI, wodurch ein zuverlässigerer Bereitstellungsprozess erreicht wird. Weitere Informationen finden Sie unter [Übersicht über den Azure Resource Manager][resource-manager].

**Versehen Sie Ressourcen mit aussagekräftigen Namen.** Aussagekräftige Namen vereinfachen das Auffinden einer bestimmten Ressource und das Verstehen ihrer Rolle. Weitere Informationen finden Sie unter [Namenskonventionen für Azure-Ressourcen](../best-practices/naming-conventions.md).

**Verwenden Sie die rollenbasierte Zugriffssteuerung (Role-Based Access Control, RBAC).** Kontrollieren Sie den Zugriff auf die von Ihnen bereitgestellten Azure-Ressourcen mit RBAC. Mit RBAC können Sie Mitgliedern Ihres DevOps-Teams Autorisierungsrollen zuweisen, um das versehentliche Löschen oder Ändern von bereitgestellten Ressourcen zu verhindern. Weitere Informationen finden Sie unter [Erste Schritte mit der Zugriffsverwaltung im Azure-Portal](/azure/active-directory/role-based-access-control-what-is/).

**Verwenden Sie Ressourcensperren für kritische Ressourcen wie VMs.** Ressourcensperren verhindern, dass ein Operator versehentlich eine Ressource löscht. Weitere Informationen finden Sie unter [Sperren von Ressourcen mit Azure Resource Manager](/azure/azure-resource-manager/resource-group-lock-resources/).

**Wählen Sie Regionspaare aus.** Wählen Sie bei der Bereitstellung in zwei Regionen die Regionen aus dem gleichen Regionspaar aus. Im Falle eines umfassenden Ausfalls hat bei jedem Regionspaar die Wiederherstellung einer der Regionen Priorität. Einige Dienste, beispielsweise georedundanter Speicher, bieten eine automatische Replikation im Regionspaar. Weitere Informationen finden Sie unter [Geschäftskontinuität und Notfallwiederherstellung: Azure-Regionspaare](/azure/best-practices-availability-paired-regions).

**Organisieren Sie Ressourcengruppen nach Lebenszyklus und Funktion.**  Im Allgemeinen sollte eine Ressourcengruppe Ressourcen enthalten, die den gleichen Lebenszyklus aufweisen. Dies erleichtert das Verwalten von Bereitstellungen, das Löschen von Testbereitstellungen und das Zuweisen von Zugriffsrechten. Dadurch wird die Wahrscheinlichkeit verringert, dass eine Produktionsbereitstellung versehentlich gelöscht oder geändert wird. Erstellen Sie separate Ressourcengruppen für Produktions-, Entwicklungs- und Testumgebungen. Nehmen Sie in einer Bereitstellung für mehrere Regionen Ressourcen für jede Region in separate Ressourcengruppen auf. Dies erleichtert die erneute Bereitstellung einer Region ohne Auswirkungen auf die anderen Regionen.

## <a name="next-steps"></a>Nächste Schritte

- [Checkliste für Resilienz für bestimmte Azure-Dienste](./resiliency-per-service.md)
- [Fehlermodusanalyse](../resiliency/failure-mode-analysis.md)


<!-- links -->
[app-service-autoscale]: /azure/monitoring-and-diagnostics/insights-how-to-scale/
[asynchronous-c-sharp]: /dotnet/articles/csharp/async
[availability-sets]:/azure/virtual-machines/virtual-machines-windows-manage-availability/
[azure-backup]: https://azure.microsoft.com/documentation/services/backup/
[circuit-breaker]: ../patterns/circuit-breaker.md
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale/
[fma]: ../resiliency/failure-mode-analysis.md
[load-balancer]: /azure/load-balancer/load-balancer-overview/
[monitoring-and-diagnostics-guidance]: ../best-practices/monitoring.md
[resource-manager]: /azure/azure-resource-manager/resource-group-overview/
[retry-pattern]: ../patterns/retry.md
[retry-service-guidance]: ../best-practices/retry-service-specific.md
[traffic-manager]: /azure/traffic-manager/traffic-manager-overview/
[traffic-manager-routing]: /azure/traffic-manager/traffic-manager-routing-methods/
[vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview/
