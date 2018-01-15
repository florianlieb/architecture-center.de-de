---
title: "Ausführen virtueller Linux-Computer in mehreren Azure-Regionen für Hochverfügbarkeit"
description: "Erfahren Sie, wie Sie virtuelle Computer für Hochverfügbarkeit und Resilienz in mehreren Regionen in Azure bereitstellen."
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Linux VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: 7d720a004d21edbffc0ddeba54e291aa817550e0
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/08/2018
---
# <a name="run-linux-vms-in-multiple-regions-for-high-availability"></a>Ausführen virtueller Linux-Computer in mehreren Regionen für Hochverfügbarkeit

Diese Referenzarchitektur zeigt eine Reihe bewährter Methoden zum Ausführen einer n-schichtigen Anwendung in mehreren Azure-Regionen, um Verfügbarkeit und eine stabile Infrastruktur für die Notfallwiederherstellung zu erzielen. 

![[0]][0]

*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*

## <a name="architecture"></a>Architecture 

Diese Architektur basiert auf der in [Ausführen virtueller Linux-Computer in einer n-schichtigen Anwendung](n-tier.md) gezeigten Architektur. 

* **Primäre und sekundäre Regionen**. Verwenden Sie zwei Regionen, um eine höhere Verfügbarkeit zu erreichen. Eine ist die primäre Region. Die andere Region ist für Failover vorgesehen.
* **Azure DNS:** [Azure DNS][azure-dns] ist ein Hostingdienst für DNS-Domänen, der die Namensauflösung unter Verwendung der Microsoft Azure-Infrastruktur durchführt. Durch das Hosten Ihrer Domänen in Azure können Sie Ihre DNS-Einträge mithilfe der gleichen Anmeldeinformationen, APIs, Tools und Abrechnung wie für die anderen Azure-Dienste verwalten.
* **Azure Traffic Manager**. [Traffic Manager][traffic-manager] leitet eingehende Anforderungen an eine der Regionen weiter. Während des normalen Betriebs werden Anforderungen an die primäre Region weitergeleitet. Wenn diese Region nicht mehr verfügbar ist, führt Traffic Manager ein Failover zur sekundären Region aus. Weitere Informationen finden Sie im Abschnitt [Traffic Manager-Konfiguration](#traffic-manager-configuration).
* **Ressourcengruppen:** Erstellen Sie separate [Ressourcengruppen][resource groups] für die primäre Region, die sekundäre Region und für Traffic Manager. Dies bietet Ihnen die Flexibilität, jede Region als eine einzelne Ressourcensammlung zu verwalten. Sie können beispielsweise eine Region erneut bereitstellen, ohne die andere außer Betrieb zu nehmen. [Verknüpfen Sie die Ressourcengruppen][resource-group-links], damit Sie eine Abfrage zum Auflisten aller Ressourcen für die Anwendung ausführen können.
* **VNETs:** Erstellen Sie für jede Region ein separates VNET. Stellen Sie sicher, dass sich die Adressräume nicht überschneiden.
* **Apache Cassandra:** Stellen Sie Cassandra für Hochverfügbarkeit in Rechenzentren mehrerer Azure-Regionen bereit. In jeder Region werden die Knoten in unterschiedlichen Racks und in Fehler- und Upgradedomänen konfiguriert, um Resilienz innerhalb der Region zu erreichen.

## <a name="recommendations"></a>Empfehlungen

Eine Architektur mit mehreren Regionen kann eine höhere Verfügbarkeit als eine Bereitstellung in einer einzelnen Region bieten. Wenn ein regionaler Ausfall die primäre Region beeinträchtigt, können Sie mit [Traffic Manager][traffic-manager] ein Failover zur sekundären Region ausführen. Diese Architektur kann auch hilfreich sein, wenn bei einem einzelnen Subsystem der Anwendung ein Fehler auftritt.

Es gibt mehrere allgemeine Vorgehensweisen für das Erreichen von Hochverfügbarkeit mit mehreren Regionen:   

* Aktiv/passiv mit Hot Standby. Der Datenverkehr wird an eine Region weitergeleitet, während die andere im Hot Standby wartet. Hot Standby (unmittelbar betriebsbereit) bedeutet, dass die virtuellen Computer in der sekundären Region jederzeit zugeordnet sind und ausgeführt werden.
* Aktiv/passiv mit Cold Standby. Der Datenverkehr wird an eine Region weitergeleitet, während die andere im Cold Standby wartet. Cold Standby (verzögert betriebsbereit) bedeutet, dass die virtuellen Computer in der sekundären Region erst zugewiesen werden, wenn sie für das Failover benötigt werden. Dieser Ansatz erfordert weniger Ausführungszeit, es dauert aber im Allgemeinen länger, bis bei einem Ausfall alle Komponenten online geschaltet sind.
* Aktiv/aktiv. Beide Regionen sind aktiv, und Anforderungen werden per Lastenausgleich zwischen ihnen verteilt. Wenn eine Region nicht verfügbar ist, wird sie aus der Rotation entfernt. 

Bei dieser Architektur liegt der Schwerpunkt auf aktiv/passiv mit Hot Standby, wobei Traffic Manager für das Failover verwendet wird. Beachten Sie, dass Sie eine kleine Anzahl virtueller Computer für Hot Standby bereitstellen und dann nach Bedarf horizontal hochskalieren können.


### <a name="regional-pairing"></a>Regionspaare

Jede Azure-Region ist mit einer anderen Region innerhalb desselben Gebiets gepaart. Sie wählen im Allgemeinen Regionen aus dem gleichen Regionspaar aus (z.B. „USA, Osten 2“ und „USA, Mitte“). Das bietet die folgenden Vorteile:

* Bei einem umfassenden Ausfall wird die Wiederherstellung mindestens einer Region aus jedem Paar priorisiert.
* Geplante Azure-Systemupdates werden in Regionspaaren nacheinander ausgeführt, um mögliche Ausfallzeiten zu minimieren.
* Regionspaare befinden sich innerhalb des gleichen geografischen Gebiets, um Anforderungen in Bezug auf den Datenspeicherort zu erfüllen.

Sie sollten allerdings sicherstellen, dass beide Regionen alle Azure-Dienste, die für Ihre Anwendung erforderlich sind, unterstützen (siehe [Dienste nach Region][services-by-region]). Weitere Informationen zu Regionspaaren finden Sie unter [Geschäftskontinuität und Notfallwiederherstellung: Azure-Regionspaare][regional-pairs].

### <a name="traffic-manager-configuration"></a>Traffic Manager-Konfiguration

Beachten Sie beim Konfigurieren von Traffic Manager die folgenden Punkte:

* **Routing:** Traffic Manager unterstützt mehrere [Routingalgorithmen][tm-routing]. Verwenden Sie für das in diesem Artikel beschriebenen Szenario Routing nach *Priorität* (ehemals Routingmethode *Failover*). Bei dieser Einstellung sendet Traffic Manager alle Anforderungen an die primäre Region, bis die primäre Region nicht mehr erreichbar ist. Zu diesem Zeitpunkt wird automatisch ein Failover zur sekundären Region ausgeführt. Weitere Informationen finden Sie unter [Konfigurieren der Routingmethode „Failover“][tm-configure-failover].
* **Integritätstest:** Traffic Manager verwendet einen HTTP- oder HTTPS-[Test][tm-monitoring], um die Verfügbarkeit jeder Region zu überwachen. Der Test prüft auf eine HTTP 200-Antwort für einen angegebenen URL-Pfad. Es hat sich bewährt, einen Endpunkt zu erstellen, der die Gesamtintegrität der Anwendung meldet, und diesen Endpunkt für den Integritätstest zu verwenden. Andernfalls meldet der Test eventuell einen fehlerfreien Endpunkt, obwohl wichtige Teile der Anwendung fehlerhaft sind. Weitere Informationen finden Sie unter [Überwachungsmuster für den Integritätsendpunkt][health-endpoint-monitoring-pattern].

Wenn Traffic Manager ein Failover ausführt, können die Clients die Anwendung für eine bestimmte Zeit nicht erreichen. Die Dauer wird durch folgende Faktoren beeinflusst:

* Der Integritätstest muss erkennen, dass die primäre Region nicht erreichbar ist.
* Die DNS-Server müssen die zwischengespeicherten DNS-Einträge für die IP-Adresse aktualisieren, die von der DNS-Gültigkeitsdauer (TTL) abhängig ist. Die Standardgültigkeitsdauer beträgt 300 Sekunden (5 Minuten), Sie können diesen Wert aber bei der Erstellung des Traffic Manager-Profils anpassen.

Weitere Informationen finden Sie unter [Traffic Manager-Überwachung][tm-monitoring].

Bei Failovern durch Traffic Manager sollten Sie ein manuelles Failback ausführen, anstatt ein automatisches Failback zu implementieren. Andernfalls könnte eine Situation eintreten, bei der die Anwendung zwischen den Regionen hin und her wechselt. Überprüfen Sie vor einem Failback, ob alle Subsysteme der Anwendung fehlerfrei sind.

Beachten Sie, dass Traffic Manager in der Standardeinstellung automatisch Failbacks ausführt. Um dies zu verhindern, verringern Sie die Priorität der primären Region nach einem Failover manuell. Angenommen, die primäre Region hat die Priorität 1 und die sekundäre Datenbank die Priorität 2. Nach einem Failover legen Sie dann die Priorität der primären Region auf 3 fest, um ein automatisches Failback zu verhindern. Wenn Sie wieder zurück wechseln möchten, ändern Sie die Priorität wieder in 1.

Mit dem folgenden Befehl für die [Azure-Befehlszeilenschnittstelle][install-azure-cli] wird die Priorität aktualisiert:

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --priority 3
```    

Ein anderer Ansatz besteht darin, den Endpunkt vorübergehend zu deaktivieren, bis Sie zum Ausführen eines Failbacks bereit sind:

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --status Disabled
```    

Je nach Ursache eines Failovers müssen Sie die Ressourcen innerhalb einer Region möglicherweise erneut bereitstellen. Testen Sie vor dem Failback die Betriebsbereitschaft. Beim Test sollten z.B. folgende Punkte geprüft werden:

* Virtuelle Computer sind richtig konfiguriert. (Alle erforderliche Software ist installiert, IIS wird ausgeführt usw.)
* Subsysteme der Anwendung sind fehlerfrei.
* Funktionstests. (Beispielsweise, dass die Datenbankebene von der Webebene aus erreichbar ist.)

### <a name="cassandra-deployment-across-multiple-regions"></a>Cassandra-Bereitstellung über mehrere Regionen

Cassandra-Rechenzentren sind eine Gruppe zusammengehöriger Datenknoten, die gemeinsam in einem Cluster konfiguriert sind, um Replikation und Workload zu trennen.

Wir empfehlen [DataStax Enterprise][datastax] für die Verwendung in der Produktion. Weitere Informationen zur Ausführung von DataStax in Azure finden Sie im [DataStax Enterprise-Bereitstellungshandbuch für Azure][cassandra-in-azure]. Die folgenden allgemeinen Empfehlungen gelten für alle Cassandra-Editionen: 

* Weisen Sie jedem Knoten eine öffentliche IP-Adresse zu. Dadurch können die Cluster über Regionen hinweg über die Azure-Backbone-Infrastruktur kommunizieren, die einen hohen Durchsatz zu niedrigen Preisen bietet.
* Schützen Sie die Knoten durch geeignete Firewall- und NSG-Konfigurationen, die Datenverkehr nur von und zu bekannten Hosts (einschließlich Clients und anderer Clusterknoten) zulassen. Beachten Sie, dass Cassandra verschiedene Ports für Kommunikation, OpsCenter, Spark usw. verwendet. Informationen zur Portverwendung in Cassandra finden Sie unter [Konfigurieren des Portzugriffs in der Firewall][cassandra-ports].
* Verwenden Sie SSL-Verschlüsselung für alle Kommunikationen der Arten [Client-zu-Knoten][ssl-client-node] und [Knoten-zu-Knoten][ssl-node-node].
* Befolgen Sie innerhalb einer Region die Richtlinien in den [Empfehlungen für Cassandra](n-tier.md#cassandra).

## <a name="availability-considerations"></a>Überlegungen zur Verfügbarkeit

Bei einer komplexen n-schichtigen Anwendung müssen Sie möglicherweise nicht die gesamte Anwendung in der sekundären Region replizieren. Stattdessen replizieren Sie nur ein kritisches Subsystem, das zur Unterstützung der Geschäftskontinuität erforderlich ist.

Traffic Manager ist eine mögliche Schwachstelle im System. Wenn beim Traffic Manager-Dienst ein Fehler auftritt, können Clients während der Ausfallzeit nicht auf Ihre Anwendung zugreifen. In der [SLA für Traffic Manager][tm-sla] erfahren Sie, ob Ihre geschäftlichen Anforderungen für Hochverfügbarkeit mit Traffic Manager allein erfüllt werden. Wenn dies nicht der Fall ist, erwägen Sie als Failback eine andere Verwaltungslösung für den Datenverkehr. Wenn der Azure Traffic Manager-Dienst fehlerhaft ist, ändern Sie die CNAME-Einträge im DNS, sodass diese auf die andere Verwaltungslösung für den Datenverkehr verweisen. (Dieser Schritt muss manuell durchgeführt werden. Bis die DNS-Änderungen weitergegeben wurden, ist die Anwendung nicht verfügbar.)

Für den Cassandra-Cluster hängen die relevanten Failoverszenarien von den Konsistenzebenen, die von der Anwendung verwendet werden, und der Anzahl der verwendeten Replikate ab. Weitere Informationen zu Konsistenzebenen und der Verwendung in Cassandra finden Sie unter [Konfigurieren der Datenkonsistenz][cassandra-consistency] und [Cassandra: Wie viele Knoten kommunizieren mit Quorum?][cassandra-consistency-usage]. Die Verfügbarkeit der Daten wird in Cassandra durch die Konsistenzebene, die von der Anwendung verwendet wird, und vom Replikationsverfahren bestimmt. Weitere Informationen zur Replikation in Cassandra finden Sie unter [Datenreplikation in NoSQL-Datenbanken im Überblick][cassandra-replication].

## <a name="manageability-considerations"></a>Überlegungen zur Verwaltbarkeit

Beim Aktualisieren der Bereitstellung aktualisieren Sie immer jeweils eine Region, um die Möglichkeit eines globalen Fehlers aufgrund einer falschen Konfiguration oder eines Fehlers in der Anwendung zu reduzieren.

Testen Sie die Resilienz des Systems gegenüber Fehlern. Hier sind einige häufige Fehlerszenarien aufgeführt, die getestet werden können:

* Herunterfahren von VM-Instanzen
* Auslasten von Ressourcen, z.B. CPU und Speicher
* Trennen/Verzögern des Netzwerks
* Absturz von Prozessen
* Ablauf von Zertifikaten
* Simulieren von Hardwarefehlern
* Herunterfahren des DNS-Diensts auf den Domänencontrollern

Messen Sie die Wiederherstellungszeiten, und stellen Sie sicher, dass diese Ihren geschäftlichen Anforderungen entsprechen. Testen Sie auch Kombinationen von Fehlermodi.


<!-- Links -->
[hybrid-vpn]: ../hybrid-networking/vpn.md
[azure-dns]: /azure/dns/dns-overview
[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: http://www.planetcassandra.org/data-replication-in-nosql-databases-explained/
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2
[cassandra-ports]: https://docs.datastax.com/en/datastax_enterprise/5.0/datastax_enterprise/sec/configFirewallPorts.html
[datastax]: https://www.datastax.com/products/datastax-enterprise
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[install-azure-cli]: /azure/xplat-cli-install
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssl-client-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLClientToNode_t.html
[ssl-node-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLNodeToNode_t.html
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx
[0]: ./images/multi-region-application-diagram.png "Architektur eines hochverfügbaren Netzwerks für n-schichtige Azure-Anwendungen"
