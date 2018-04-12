---
title: Ausführen virtueller Windows-Computer in mehreren Azure-Regionen für hohe Verfügbarkeit
description: Informationen zur Bereitstellung virtueller Computer in mehreren Regionen in Azure für hohe Verfügbarkeit und Resilienz.
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Windows VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: 9772d57e6a11711d77032b049168565d52d919b8
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="run-windows-vms-in-multiple-regions-for-high-availability"></a>Ausführen virtueller Windows-Computer in mehreren Regionen für hohe Verfügbarkeit

Diese Referenzarchitektur zeigt eine Reihe bewährter Methoden zum Ausführen einer n-schichtigen Anwendung in mehreren Azure-Regionen, um Verfügbarkeit und eine stabile Infrastruktur für die Notfallwiederherstellung zu erzielen. 

[![0]][0] 

*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*

## <a name="architecture"></a>Architecture 

Diese Architektur baut auf der Architektur auf, die unter [Ausführen virtueller Windows-Computer für eine Anwendung mit n-Schichten](n-tier.md) dargestellt ist. 

* **Primäre und sekundäre Regionen**. Verwenden Sie zwei Regionen, um eine höhere Verfügbarkeit zu erreichen. Eine ist die primäre Region. Die andere Region ist für das Failover.
* **Azure DNS:** [Azure DNS][azure-dns] ist ein Hostingdienst für DNS-Domänen, der die Namensauflösung unter Verwendung der Microsoft Azure-Infrastruktur durchführt. Durch das Hosten Ihrer Domänen in Azure können Sie Ihre DNS-Einträge mithilfe der gleichen Anmeldeinformationen, APIs, Tools und Abrechnung wie für die anderen Azure-Dienste verwalten.
* **Azure Traffic Manager**. [Traffic Manager][traffic-manager] leitet eingehende Anforderungen an eine der Regionen weiter. Während des normalen Betriebs werden Anforderungen an die primäre Region weitergeleitet. Wenn diese Region nicht mehr verfügbar ist, führt Traffic Manager ein Failover zur sekundären Region aus. Weitere Informationen finden Sie im Abschnitt [Traffic Manager-Konfiguration](#traffic-manager-configuration).
* **Ressourcengruppen:** Erstellen Sie separate [Ressourcengruppen][resource groups] für die primäre Region, die sekundäre Region und für Traffic Manager. Dies bietet Ihnen die Flexibilität, jede Region als eine einzelne Ressourcensammlung zu verwalten. Sie können beispielsweise eine Region erneut bereitstellen, ohne die andere außer Betrieb zu nehmen. [Verknüpfen Sie die Ressourcengruppen][resource-group-links], damit Sie eine Abfrage zum Auflisten aller Ressourcen für die Anwendung ausführen können.
* **VNETs:** Erstellen Sie für jede Region ein separates VNET. Stellen Sie sicher, dass sich die Adressräume nicht überschneiden. 
* **SQL Server Always On-Verfügbarkeitsgruppe**. Bei Verwendung von SQL Server werden [SQL Always On-Verfügbarkeitsgruppen][sql-always-on] empfohlen, um Hochverfügbarkeit zu erzielen. Erstellen Sie eine einzelne Verfügbarkeitsgruppe, die SQL Server-Instanzen in beiden Regionen enthält. 

    > [!NOTE]
    > Ziehen Sie auch eine [Azure SQL-Datenbank][azure-sql-db] in Betracht, die eine relationale Datenbank als Clouddienst bereitstellt. Mit einer SQL-Datenbank müssen Sie weder eine Verfügbarkeitsgruppe konfigurieren noch das Failover verwalten.  
    > 

* **VPN-Gateways**. Erstellen Sie ein [VPN-Gateway][vpn-gateway] in jedem VNet, und konfigurieren Sie eine [VNet-zu-VNet-Verbindung][vnet-to-vnet], um den Netzwerkdatenverkehr zwischen den beiden VNets zu ermöglichen. Dies ist für die SQL Always On-Verfügbarkeitsgruppe erforderlich.

## <a name="recommendations"></a>Empfehlungen

Eine Architektur mit mehreren Regionen kann eine höhere Verfügbarkeit als eine Bereitstellung in einer einzelnen Region bieten. Wenn ein regionaler Ausfall die primäre Region beeinträchtigt, können Sie mit [Traffic Manager][traffic-manager] ein Failover zur sekundären Region ausführen. Diese Architektur kann auch hilfreich sein, wenn bei einem einzelnen Subsystem der Anwendung ein Fehler auftritt.

Es gibt mehrere allgemeine Vorgehensweisen für das Erreichen von Hochverfügbarkeit mit mehreren Regionen: 

* Aktiv/passiv mit Hot Standby. Der Datenverkehr wird an eine Region weitergeleitet, während die andere im Hot Standby wartet. Hot Standby (unmittelbar betriebsbereit) bedeutet, dass die virtuellen Computer in der sekundären Region jederzeit zugeordnet sind und ausgeführt werden.
* Aktiv/passiv mit Cold Standby. Der Datenverkehr wird an eine Region weitergeleitet, während die andere im Cold Standby wartet. Cold Standby (verzögert betriebsbereit) bedeutet, dass die virtuellen Computer in der sekundären Region erst zugewiesen werden, wenn sie für das Failover benötigt werden. Dieser Ansatz erfordert weniger Ausführungszeit, es dauert aber im Allgemeinen länger, bis bei einem Ausfall alle Komponenten online geschaltet sind.
* Aktiv/aktiv. Beide Regionen sind aktiv, und Anforderungen werden per Lastenausgleich zwischen ihnen verteilt. Wenn eine Region nicht verfügbar ist, wird sie aus der Rotation entfernt. 

Bei dieser Referenzarchitektur liegt der Schwerpunkt auf Aktiv/Passiv mit Hot Standby, wobei Traffic Manager für das Failover verwendet wird. Beachten Sie, dass Sie eine kleine Anzahl virtueller Computer für Hot Standby bereitstellen und dann nach Bedarf horizontal skalieren können.

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

### <a name="configure-sql-server-always-on-availability-groups"></a>Konfigurieren von SQL Server Always On-Verfügbarkeitsgruppen

Bei früheren Versionen als Windows Server 2016 erfordern SQL Server Always On-Verfügbarkeitsgruppen einen Domänencontroller, und alle Knoten in der Verfügbarkeitsgruppe müssen sich in der gleichen Active Directory (AD)-Domäne befinden. 

So konfigurieren Sie die Verfügbarkeitsgruppe:

* Platzieren Sie mindestens zwei Domänencontroller in jeder Region.
* Weisen Sie jedem Domänencontroller eine statische IP-Adresse zu.
* Erstellen Sie eine VNet-zu-VNet-Verbindung, um die Kommunikation zwischen den VNets zu ermöglichen.
* Fügen Sie für jedes VNet die IP-Adressen der Domänencontroller (aus beiden Regionen) zur DNS-Serverliste hinzu. Sie können den folgenden CLI-Befehl verwenden. Weitere Informationen finden Sie unter [Verwalten von DNS-Servern, die von einem virtuellen Netzwerk (VNet) verwendet werden][vnet-dns].

    ```bat
    azure network vnet set --resource-group dc01-rg --name dc01-vnet --dns-servers "10.0.0.4,10.0.0.6,172.16.0.4,172.16.0.6"
    ```

* Erstellen Sie einen [Windows Server-Failovercluster][wsfc] (WSFC), der die SQL Server-Instanzen in beiden Regionen enthält. 
* Erstellen Sie eine SQL Server Always On-Verfügbarkeitsgruppe, die SQL Server-Instanzen sowohl in der primären als auch der sekundären Region enthält. Die Schritte finden Sie unter [Erweitern der Always On-Verfügbarkeitsgruppe auf ein Azure-Remoterechenzentrum (PowerShell)](https://blogs.msdn.microsoft.com/sqlcat/2014/09/22/extending-alwayson-availability-group-to-remote-azure-datacenter-powershell/).

  * Legen Sie das primäre Replikat in der primären Region ab.
  * Legen Sie ein oder mehrere sekundäre Replikate in der primären Region ab. Konfigurieren Sie diese für die Verwendung synchroner Commits mit automatischem Failover.
  * Legen Sie ein oder mehrere sekundäre Replikate in der sekundären Region ab. Konfigurieren Sie diese aus Leistungsgründen für die Verwendung *asynchroner* Commits. (Andernfalls müssen alle T-SQL-Transaktionen auf einem Roundtrip über das Netzwerk zur sekundären Region warten.)

    > [!NOTE]
    > Replikate mit asynchronem Commit unterstützen kein automatisches Failover.
    >
    >

Weitere Informationen finden Sie unter [Ausführen virtueller Windows-Computer in einer Azure-Architektur mit n Ebenen](n-tier.md).

## <a name="availability-considerations"></a>Überlegungen zur Verfügbarkeit

Bei einer komplexen n-schichtigen Anwendung müssen Sie möglicherweise nicht die gesamte Anwendung in der sekundären Region replizieren. Stattdessen replizieren Sie nur ein kritisches Subsystem, das zur Unterstützung der Geschäftskontinuität erforderlich ist.

Traffic Manager ist eine mögliche Schwachstelle im System. Wenn beim Traffic Manager-Dienst ein Fehler auftritt, können Clients während der Ausfallzeit nicht auf Ihre Anwendung zugreifen. In der [SLA für Traffic Manager][tm-sla] erfahren Sie, ob Ihre geschäftlichen Anforderungen für Hochverfügbarkeit mit Traffic Manager allein erfüllt werden. Wenn dies nicht der Fall ist, erwägen Sie als Failback eine andere Verwaltungslösung für den Datenverkehr. Wenn der Azure Traffic Manager-Dienst fehlerhaft ist, ändern Sie die CNAME-Einträge im DNS, sodass diese auf die andere Verwaltungslösung für den Datenverkehr verweisen. (Dieser Schritt muss manuell durchgeführt werden. Bis die DNS-Änderungen weitergegeben wurden, ist die Anwendung nicht verfügbar.)

Für den SQL Server-Cluster sind zwei Failoverszenarien zu berücksichtigen:

- Bei allen SQL Server-Datenbankreplikaten in der primären Region treten Fehler auf. Dies kann z. B. während eines regionalen Ausfalls vorkommen. In diesem Fall müssen Sie für die Verfügbarkeitsgruppe ein manuelles Failover ausführen, obwohl Traffic Manager automatisch ein Failover auf dem Front-End ausführt. Führen Sie die Schritte unter [Ausführen eines erzwungenen manuellen Failovers einer SQL Server-Verfügbarkeitsgruppe](https://msdn.microsoft.com/library/ff877957.aspx) aus, in denen beschrieben ist, wie ein erzwungenes Failover mithilfe von SQL Server Management Studio, Transact-SQL oder PowerShell in SQL Server 2016 ausgeführt wird.

   > [!WARNING]
   > Bei einem erzwungenem Failover besteht das Risiko eines Datenverlusts. Sobald die primäre Region wieder online ist, erstellen Sie eine Momentaufnahme der Datenbank, und verwenden Sie [tablediff], um die Unterschiede zu ermitteln.
   >
   >
- Traffic Manager führt ein Failover zur sekundären Region aus, doch ist das primäre SQL Server-Datenbankreplikat weiterhin verfügbar. So kann beispielsweise die Front-End-Ebene fehlgeschlagen, ohne dass dies Auswirkungen auf die virtuellen SQL Server-Computer hat. In diesem Fall wird der Internetdatenverkehr an die sekundäre Region weitergeleitet, und diese Region kann immer noch eine Verbindung mit dem primären Replikat herstellen. Es kommt jedoch zu erhöhter Latenz, da die SQL Server-Verbindungen regionsübergreifend verlaufen. In dieser Situation sollten Sie auf folgende Weise ein manuelles Failover ausführen:

   1. Wechseln Sie bei einem SQL Server-Datenbankreplikat in der sekundären Region vorübergehend zu *synchronen* Commits. Dadurch wird sichergestellt, dass während des Failovers kein Datenverlust auftritt.
   2. Führen Sie ein Failover zu diesem Replikat aus.
   3. Wenn Sie ein Failback zur primären Region ausführen, ändern Sie die Einstellung wieder in asynchrone Commits.

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
[azure-sla]: https://azure.microsoft.com/support/legal/sla/
[azure-sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[install-azure-cli]: /azure/xplat-cli-install
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[services-by-region]: https://azure.microsoft.com/regions/#services
[sql-always-on]: https://msdn.microsoft.com/library/hh510230.aspx
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vnet-dns]: /azure/virtual-network/virtual-networks-manage-dns-in-vnet
[vnet-to-vnet]: /azure/vpn-gateway/vpn-gateway-vnet-vnet-rm-ps
[vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx

[0]: ./images/multi-region-application-diagram.png "Architektur eines hochverfügbaren Netzwerks für Azure-Anwendungen mit n-Schichten"
