---
title: Hochverfügbarkeit für Azure-Anwendungen
description: Technische Übersichten und ausführliche Informationen zum Entwerfen und Erstellen von Anwendungen für Hochverfügbarkeit in Microsoft Azure.
author: adamglick
ms.date: 05/31/2017
ms.openlocfilehash: f116b9e64f1722b5141ae90239d5c8a8b4a89487
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="high-availability-for-applications-built-on-microsoft-azure"></a>Hochverfügbarkeit für in Microsoft Azure erstellte Anwendungen
Eine hochverfügbare Anwendung gleicht Schwankungen bei Verfügbarkeit und Last sowie vorübergehende Ausfälle in abhängigen Diensten und Hardwarekomponenten aus. Die Anwendung funktioniert weiterhin auf einem akzeptablen Niveau, wie dies durch Geschäftsanforderungen oder Vereinbarungen zum Servicelevel (SLAs) für die Anwendung definiert ist.

## <a name="azure-high-availability-features"></a>Funktionen von Azure für hohe Verfügbarkeit
Azure weist viele integrierte Plattformfunktionen auf, die hoch verfügbare Anwendungen unterstützen. In diesem Abschnitt werden einige dieser wichtigen Funktionen beschrieben.

### <a name="fabric-controller"></a>Fabric Controller
Mit dem Azure Fabric Controller wird der Zustand von Azure-Serverinstanzen bereitgestellt und überwacht. Der Fabric Controller überwacht den Status der Hardware und Software für die Host- und Gastcomputerinstanzen. Wenn ein Fehler erkannt wird, werden SLAs durch automatisches Verschieben der VM-Instanzen aufrechterhalten. Das Konzept von Fehler- und Upgradedomänen unterstützt Compute-SLAs zusätzlich.

Wenn mehrere Clouddienst-Rolleninstanzen bereitgestellt werden, stellt Azure diese in verschiedenen Fehlerdomänen bereit. Eine Fehlerdomänengrenze ist im Wesentlichen ein anderes Hardwarerack in der gleichen Region. Fehlerdomänen reduzieren die Wahrscheinlichkeit, dass ein ermittelter Hardwarefehler die Dienstbereitstellung einer Anwendung unterbricht. Sie können die Anzahl von Fehlerdomänen für Ihre Worker- oder Webrollen nicht verwalten. Der Fabric Controller verwendet dedizierte Ressourcen, die von den in Azure gehosteten Anwendungen getrennt sind. Er weist eine Betriebszeit von 100 % auf, da er das Herzstück des Azure-Systems darstellt. Er überwacht und verwaltet Rolleninstanzen über Fehlerdomänen hinweg.

Im folgenden Diagramm sind gemeinsam genutzte Azure-Ressourcen dargestellt, die vom Fabric Controller über verschiedene Fehlerdomänen hinweg bereitgestellt und verwaltet werden.

![Vereinfachte Darstellung der Fehlerdomänenisolation](./images/high-availability-azure-applications/fault-domain-isolation.png)

Während Fehlerdomänen physische Trennungen zur Vermeidung von Ausfällen darstellen, handelt es sich bei Upgradedomänen um logische Einheiten der Instanzentrennung, die bestimmen, für welche Instanzen eines Diensts zu einem bestimmten Zeitpunkt ein Upgrade durchgeführt wird. Standardmäßig sind fünf Upgradedomänen für Ihre Bereitstellung des gehosteten Diensts definiert. Sie können diesen Wert jedoch in der Dienstdefinitionsdatei ändern. Wenn Sie beispielsweise über acht Instanzen Ihrer Webrolle verfügen, befinden sich zwei Instanzen in drei Upgradedomänen und zwei Instanzen in einer Upgradedomäne. Azure definiert die Updatesequenz basierend auf der Anzahl von Upgradedomänen. Weitere Informationen finden Sie unter [Aktualisieren eines Clouddiensts](/azure/cloud-services/cloud-services-update-azure-service/).

### <a name="features-in-other-services"></a>Funktionen in anderen Diensten
Zusätzlich zu den Plattformfeatures, die Hochverfügbarkeit von Computeressourcen unterstützen, bettet Azure Hochverfügbarkeitsfeatures in die anderen Dienste ein. Beispielsweise behält Azure Storage mindestens drei Replikate aller Daten in Ihrem Azure Storage-Konto bei. Darüber hinaus wird die Georeplikation zum Speichern von Datenkopien in einer sekundären Region ermöglicht. Das Azure Content Delivery Network ermöglicht die Zwischenspeicherung von Blobs auf der ganzen Welt, um Redundanz, Skalierbarkeit und niedrigere Latenzen sicherzustellen. In Azure SQL-Datenbank werden auch mehrere Replikate gespeichert.

Eine detailliertere Erläuterung der Verfügbarkeitsfeatures der Azure-Plattform finden Sie in der [technischen Dokumentation zur Resilienz](index.md). Weitere Informationen finden Sie auch unter [Bewährte Methoden für das Entwerfen umfassender Dienste mit Microsoft Azure](https://azure.microsoft.com/blog/best-practices-for-designing-large-scale-services-on-windows-azure/).

Obwohl Azure mehrere Funktionen zur Unterstützung von Hochverfügbarkeit bereitstellt, ist es wichtig, die Grenzen dieser Funktionen zu kennen:

* Azure garantiert in Bezug auf die Computeleistung, dass Ihre Rollen verfügbar sind und ausgeführt werden, kann jedoch nicht erkennen, ob Ihre Anwendung ausgeführt wird oder überlastet ist.
* Für Azure SQL-Datenbank werden Daten synchron innerhalb der Region repliziert. Sie können die aktive Georeplikation auswählen, bei der bis zu vier zusätzliche Datenbankkopien in derselben Region (oder verschiedenen Regionen) zulässig sind. Bei diesen Datenbankreplikaten handelt es sich zwar nicht um Point-in-Time-Sicherungen, diese Funktionen werden jedoch von der SQL-Datenbank bereitgestellt. Weitere Informationen finden Sie unter [Wiederherstellen einer Azure SQL-Datenbank mit automatisierten Datenbanksicherungen: Point-in-Time-Wiederherstellung](/azure/sql-database/sql-database-recovery-using-backups#point-in-time-restore).
* Für Azure Storage werden Tabellen- und Blobdaten standardmäßig in eine andere Region repliziert. Sie können auf die Replikate jedoch erst zugreifen, nachdem Microsoft ein Failover zum alternativen Standort durchgeführt hat. Ein Regionsfailover findet nur bei einer längeren regionsweiten Dienstunterbrechung statt, und es gibt keine SLA für die Dauer eines geografischen Failovers. Beachten Sie auch, dass sich jegliche Datenbeschädigungen schnell auf die Replikate ausbreiten. Aus diesen Gründen müssen Plattformverfügbarkeitsfeatures durch anwendungsspezifische Verfügbarkeitsfeatures ergänzt werden, z.B. durch das Blobmomentaufnahmefeature für die Erstellung von Point-in-Time-Sicherungen von Blobdaten.

### <a name="availability-sets-for-azure-virtual-machines"></a>Verfügbarkeitsgruppen für virtuelle Azure-Computer
In diesem Dokument werden hauptsächlich Clouddienste behandelt, die auf einem PaaS-Modell (Platform as a Service) basieren. Es gibt auch bestimmte Verfügbarkeitsfeatures für virtuelle Azure-Computer, die ein IaaS-Modell (Infrastructure-as-a-Service) verwenden. Um Hochverfügbarkeit bei virtuellen Computern zu erzielen, müssen Sie Verfügbarkeitsgruppen verwenden, die eine ähnliche Funktion für Fehler- und Upgradedomänen erfüllen. Innerhalb einer Verfügbarkeitsgruppe positioniert Azure die virtuellen Computer so, dass ermittelte Hardwarefehler und Wartungsaktivitäten nicht dazu führen, dass alle Computer in dieser Gruppe ausfallen. Verfügbarkeitsgruppen sind erforderlich, um die Azure-SLA für die Verfügbarkeit von virtuellen Computern zu erreichen.

Das folgende Diagramm zeigt zwei Verfügbarkeitsgruppen für virtuelle Webcomputer bzw. virtuelle SQL Server-Computer.

![Verfügbarkeitsgruppen für virtuelle Azure-Computer](./images/high-availability-azure-applications/availability-set-for-azure-virtual-machines.png)

> [!NOTE]
> Im vorherigen Diagramm ist SQL Server installiert und wird auf virtuellen Computern ausgeführt. Dies unterscheidet sich von der Azure SQL-Datenbank, bei der eine Datenbank als verwalteter Dienst bereitgestellt wird.
> 
> 

## <a name="application-strategies-for-high-availability"></a>Anwendungsstrategien für Hochverfügbarkeit
Die meisten Anwendungsstrategien für Hochverfügbarkeit umfassen entweder Redundanz oder die Entfernung von festen Abhängigkeiten zwischen Anwendungskomponenten. Der Anwendungsentwurf sollte Fehlertoleranz während zeitweiliger Ausfälle von Azure oder Drittanbieterdiensten unterstützen. In den folgenden Abschnitten werden Anwendungsmuster zur Erhöhung der Verfügbarkeit Ihrer Clouddienste beschrieben.

### <a name="asynchronous-communication-and-durable-queues"></a>Asynchrone Kommunikation und permanente Warteschlangen
Um die Verfügbarkeit in Azure-Anwendungen zu erhöhen, betrachten wir die asynchrone Kommunikation zwischen lose gekoppelten Diensten. In diesem Muster werden Nachrichten zur späteren Verarbeitung entweder an Speicherwarteschlangen oder an Azure Service Bus-Warteschlangen geschrieben. Wenn eine Nachricht an die Warteschlange geschrieben wird, erfolgt sofort die Steuerelementrückgabe an den Absender. Die Nachricht wird von einem anderen Dienst der Anwendung, die üblicherweise als Workerrolle implementiert ist, verarbeitet. Wenn der Verarbeitungsdienst angehalten wird, werden die Nachrichten in der Warteschlange gesammelt, bis der Verarbeitungsdienst wiederhergestellt wird. Es gibt keine direkte Abhängigkeit zwischen dem Absender im Front-End und dem Nachrichtenprozessor. Hierdurch werden synchrone Dienstaufrufe verhindert, die Engpässe in verteilten Anwendungen verursachen können.

Eine Variante dieses Musters speichert Informationen über fehlerhafte Datenbankaufrufe in Azure Storage (Blobs, Tabellen, Warteschlangen) oder Service Bus-Warteschlangen. Ein Beispiel: Ein synchroner Aufruf an einen anderen Dienst innerhalb einer Anwendung (z.B. Azure SQL-Datenbank) führt wiederholt zu einem Fehler. Sie können diese Anforderung möglicherweise in einem permanenten Speicher serialisieren. Zu einem späteren Zeitpunkt, wenn der Dienst oder die Datenbank wieder online ist, kann die Anwendung die Anforderung aus dem Speicher erneut übermitteln. Der Unterschied in diesem Modell besteht darin, dass der temporäre Speicherort nur bei Ausfällen verwendet wird und kein fester Bestandteil des Anwendungsworkflows ist.

In beiden Szenarien verhindern die asynchrone Kommunikation und der temporäre Speicher, dass ein ausgefallener Back-End-Dienst zu einem Ausfall der gesamten Anwendung führt. Warteschlangen dienen als logisches Zwischenelement. Weitere Informationen zur Auswahl des Warteschlangendiensts finden Sie unter [Storage-Warteschlangen und Service Bus-Warteschlangen – Vergleich und Gegenüberstellung](/azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted/).

### <a name="fault-detection-and-retry-logic"></a>Fehlererkennung und Wiederholungslogik
Ein wichtiger Aspekt beim Entwerfen von hochverfügbaren Anwendungen ist die Verwendung einer Wiederholungslogik im Code, um einen vorübergehend nicht verfügbaren Dienst richtig behandeln zu können. Die aktuellen SDK-Versionen für Azure Storage und den Azure Service Bus bieten native Unterstützung für Wiederholungen. Weitere Informationen zum Bereitstellen einer benutzerdefinierten Wiederholungslogik für Ihre Anwendung finden Sie unter [Muster „Wiederholung“](../patterns/retry.md).

### <a name="reference-data-pattern-for-high-availability"></a>Verweisdatenmuster für Hochverfügbarkeit
Verweisdaten sind die schreibgeschützten Daten einer Anwendung. Diese Daten stellen den Geschäftskontext bereit, innerhalb dessen die Anwendung während eines Geschäftsvorgangs Transaktionsdaten generiert. Um die Integrität der Transaktionsdaten sicherzustellen, ist eine Momentaufnahme der Verweisdaten zu dem Zeitpunkt, an dem die Transaktion abgeschlossen wurde, erforderlich.

Für die ordnungsgemäße Ausführung der Anwendung sind Verweisdaten erforderlich. Verschiedene Anwendungen erstellen und verwalten Verweisdaten – Systeme zur Masterdatenverwaltung (MDM) führen diese Funktion häufig aus. Diese Systeme sind für den Lebenszyklus der Verweisdaten zuständig. Beispiele für Verweisdaten: Produktkataloge, Mitarbeitermaster, Teilemaster und Gerätemaster. Verweisdaten können auch von außerhalb der Organisation stammen, z.B. Postleitzahlen oder Steuersätze. Strategien zur Erhöhung der Verfügbarkeit von Verweisdaten sind üblicherweise weniger kompliziert als Strategien für Transaktionsdaten. Verweisdaten haben den Vorteil, größtenteils unveränderlich zu sein.

Web- und Workerrollen in Azure, die Verweisdaten verwenden, können autonom zur Runtime erstellt werden, indem die Verweisdaten zusammen mit der Anwendung bereitgestellt werden. Diese Vorgehensweise ist ideal, wenn die Größe des lokalen Speichers eine solche Bereitstellung zulässt. Eingebettete SQL-Datenbanken, NoSQL-Datenbanken oder lokal bereitgestellte XML-Dateien unterstützen die Autonomie von Azure-Computeskalierungseinheiten. Sie sollten jedoch einen Mechanismus einrichten, um die Daten in jeder Rolle ohne erneute Bereitstellung zu aktualisieren. Platzieren Sie zu diesem Zweck alle Updates der Verweisdaten in einem Cloudspeicherendpunkt (z.B. im Azure Blob Storage oder in der SQL-Datenbank). Fügen Sie jeder Rolle Code hinzu, der die Datenupdates beim Starten der Rolle in die Computeknoten herunterlädt. Alternativ dazu können Sie auch Code hinzufügen, der einem Administrator erlaubt, einen erzwungenen Download in die Rolleninstanzen durchzuführen.

Um die Verfügbarkeit zu erhöhen, sollten Sie Rollen auch einen Satz Verweisdaten enthalten, falls der Speicher ausfällt. Auf diese Weise können die Rollen mit einem Basissatz an Verweisdaten starten, bis die Speicherressource für die Updates verfügbar wird.

![Hochverfügbarkeit von Anwendungen durch autonome Computeknoten](./images/high-availability-azure-applications/application-high-availability-through-autonomous-compute-nodes.png)

Bei diesem Muster dauert das Starten neuer Bereitstellungen oder Rolleninstanzen möglicherweise länger, wenn Sie große Mengen von Verweisdaten bereitstellen oder herunterladen. Dieser Nachteil kann jedoch akzeptabel sein, wenn Sie dafür die Autonomie erhalten, dass die Verweisdaten für jede Rolle sofort verfügbar sind und Sie sich nicht auf externe Speicherdienste verlassen müssen.

### <a name="transactional-data-pattern-for-high-availability"></a>Transaktionsdatenmuster für Hochverfügbarkeit
Transaktionsdaten sind die Daten, die von einer Anwendung in einem Geschäftskontext generiert werden. Transaktionsdaten sind eine Kombination aus den Geschäftsprozessen, die von der Anwendung implementiert werden, und den Verweisdaten, die diese Prozesse unterstützen. Beispiele für Transaktionsdaten sind Bestellungen, erweiterte Versandmitteilungen, Rechnungen und CRM-Verkaufschancen. Transaktionsdaten werden zu Aufbewahrungszwecken oder zur weiteren Verarbeitung an externe Systeme übermittelt.

Verweisdaten können sich innerhalb der für diese Daten zuständigen Systeme verändern. Daher müssen Transaktionsdaten den Point-in-Time-Verweisdatenkontext speichern, um externe Abhängigkeiten für die semantische Konsistenz auf ein Minimum zu reduzieren. Beispiel: Ein Produkt wird einige Monate nach Ausführung einer Bestellung aus dem Katalog entfernt. Es empfiehlt sich, einen möglichst umfangreichen Verweisdatenkontext mit der Transaktion zu speichern. Durch diese Vorgehensweise wird die mit der Transaktion verknüpfte Semantik beibehalten, auch wenn sich die Verweisdaten nach der Erfassung der Transaktion ändern.

Wie bereits erwähnt, können Architekturen, die eine lose Kopplung und asynchrone Kommunikation verwenden, eine höhere Verfügbarkeit bieten. Dies gilt auch für Transaktionsdaten, die Implementierung ist jedoch komplexer. Bei konventionellen Transaktionsmustern garantiert üblicherweise die Datenbank die Transaktion. Wenn Sie Zwischenschichten einführen, muss der Anwendungscode die Daten in verschiedenen Schichten ordnungsgemäß verarbeiten, um Konsistenz und Dauerhaftigkeit sicherzustellen.

Die folgende Sequenz beschreibt einen Workflow, der die Erfassung der Transaktionsdaten von ihrer Verarbeitung trennt:

1. Webcomputeknoten: Vorlegen der Verweisdaten.
2. Externer Speicher: Speichern der Transaktionszwischendaten.
3. Webcomputeknoten: Abschließen der Endbenutzertransaktion.
4. Webcomputeknoten: Senden der abgeschlossenen Transaktionsdaten mit dem jeweiligen Verweisdatenkontext an einen temporären permanenten Speicher, für den eine vorhersagbare Reaktion garantiert wird.
5. Webcomputeknoten: Signalisieren des Abschlusses der Transaktion an den Benutzer.
6. Hintergrundcomputeknoten: Extrahieren der Transaktionsdaten, Weiterverarbeiten der Daten bei Bedarf und Senden der Daten an den endgültigen Speicherort im aktuellen System.

Das folgende Diagramm zeigt eine mögliche Implementierung dieses Entwurfs in einem in Azure gehosteten Clouddienst.

![Hochverfügbarkeit durch lose Kopplung](./images/disaster-recovery-high-availability-azure-applications/application-high-availability-through-loose-coupling.png)

Die gestrichelten Pfeile im vorherigen Diagramm weisen auf eine asynchrone Verarbeitung hin. Die Front-End-Webrolle hat keine Kenntnis über diese asynchrone Verarbeitung. Dadurch wird die Transaktion an ihrem endgültigen Ziel mit Verweis auf das aktuelle System gespeichert. Aufgrund der durch dieses asynchrone Modell eingeführten Latenz stehen die Transaktionsdaten nicht sofort für Abfragen zur Verfügung. Daher muss jede Transaktionsdateneinheit in einem Cache oder einer Benutzersitzung gespeichert werden, um die sofortigen Anforderungen der Benutzeroberfläche zu erfüllen.

Die Webrolle ist vom Rest der Infrastruktur unabhängig. Ihr Verfügbarkeitsprofil ist eine Kombination der Webrolle und der Azure-Warteschlange, nicht der gesamten Infrastruktur. Dieser Ansatz sorgt nicht nur für Hochverfügbarkeit, sondern ermöglicht auch unabhängig vom Back-End-Speicher eine horizontale Skalierung der Webrolle. Dieses Modell für hohe Verfügbarkeit kann die Wirtschaftlichkeit von Vorgängen beeinflussen. Zusätzliche Komponenten wie Azure-Warteschlangen und -Workerrollen können sich auf die monatlichen Nutzungskosten auswirken.

Das vorherige Diagramm zeigt eine Implementierung dieses entkoppelten Ansatzes für Transaktionsdaten. Es gibt viele weitere mögliche Implementierungen. Die folgende Liste enthält einige Alternativen:

* Eine Workerrolle kann zwischen Webrolle und Speicherwarteschlange platziert werden.
* Eine Service Bus-Warteschlange kann anstelle einer Azure Storage-Warteschlange verwendet werden.
* Beim endgültigen Ziel kann es sich um Azure Storage oder einen anderen Datenbankanbieter handeln.
* Azure Cache kann auf der Webschicht verwendet werden, um die unmittelbaren Anforderungen an eine Zwischenspeicherung nach der Transaktion zu erfüllen.

### <a name="scalability-patterns"></a>Skalierbarkeitsmuster
Es ist wichtig zu beachten, dass sich die Skalierbarkeit eines Clouddiensts direkt auf die Verfügbarkeit auswirkt. Wenn Ihr Dienst aufgrund einer erhöhten Last nicht mehr reagiert, nehmen die Benutzer dies als Ausfall der Anwendung wahr. Befolgen Sie die bewährten Methoden für die Skalierbarkeit basierend auf der erwarteten Anwendungslast und Ihren zukünftigen Erwartungen. Die Maximierung der Skalierung erfordert eine Vielzahl von Überlegungen, z.B. die Verwendung einzelner oder mehrerer Speicherkonten, die gemeinsame Nutzung über mehrere Datenbanken hinweg sowie Strategien für die Zwischenspeicherung. Ausführliche Informationen zu diesen Mustern finden Sie unter [Bewährte Methoden für das Entwerfen umfassender Dienste mit Microsoft Azure](https://azure.microsoft.com/blog/best-practices-for-designing-large-scale-services-on-windows-azure/).

## <a name="next-steps"></a>Nächste Schritte
In dieser Dokumentreihe werden die Themen Notfallwiederherstellung und Hochverfügbarkeit für in Microsoft Azure erstellte Anwendungen behandelt. Der nächste Artikel dieser Reihe ist [Notfallwiederherstellung für in Microsoft Azure erstellte Anwendungen](disaster-recovery-azure-applications.md).

