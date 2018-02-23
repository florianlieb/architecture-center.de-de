---
title: Anleitungen zum Content Delivery Network
description: "Anleitungen zum Content Delivery Network (CDN) für die Bereitstellung von in Azure gehosteten Inhalten mit hoher Bandbreite."
author: dragon119
ms.date: 02/02/2018
pnp.series.title: Best Practices
ms.openlocfilehash: 9ee9099c85818af9486408f6ece41d3f6fcd9b44
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/23/2018
---
# <a name="best-practices-for-using-content-delivery-networks-cdns"></a>Bewährte Methoden für die Verwendung von Content Delivery Networks (CDNs)

Ein Content Delivery Network (CDN) ist ein verteiltes Netzwerk mit Servern, über die Webinhalte auf effiziente Weise für Benutzer bereitgestellt werden können. In CDNs werden zwischengespeicherte Inhalte auf Edgeservern gespeichert, die sich in der Nähe der Endbenutzer befinden, um die Wartezeit zu verringern. 

CDNs werden normalerweise zum Übermitteln statischer Inhalte verwendet, z.B. Bilder, Stylesheets, Dokumente, clientseitige Skripts und HTML-Seiten. Die Hauptvorteile beim Verwenden eines CDN sind kürzere Wartezeit und schnellere Bereitstellung des Inhalts für die Benutzer, und zwar unabhängig von der Entfernung des geografischen Standorts der Benutzer zu dem Datencenter, in dem die Anwendung gehostet wird. CDNs können auch zur Reduzierung der Last einer Webanwendung beitragen, da die Anwendung keine Anforderungen für den Inhalt verarbeiten muss, der im CDN gehostet wird.
 
![CDN-Diagramm](./images/cdn/CDN.png)

In Azure ist das [Azure Content Delivery Network](/azure/cdn/cdn-overview) eine globale CDN-Lösung für die Bereitstellung von Inhalten mit hoher Bandbreite, die in Azure oder an einem beliebigen anderen Ort gehostet werden. Mit dem Azure CDN können Sie öffentlich verfügbare Objekte zwischenspeichern, die über Azure Blob Storage, eine Webanwendung, einen virtuellen Computer oder einen öffentlich zugänglichen Webserver geladen wurden. 

In diesem Thema werden einige allgemeine bewährte Methoden und Aspekte zur Verwendung eines CDN beschrieben. Weitere Informationen zur Nutzung von Azure CDN finden Sie in der [CDN-Dokumentation](/azure/cdn/).

## <a name="how-and-why-a-cdn-is-used"></a>Wie und warum ein CDN verwendet wird

Ein CDN wird häufig für Folgendes verwendet:  

* Bereitstellen statischer Ressourcen für Clientanwendungen, häufig von einer Website. Bei diesen Ressourcen kann es sich um Bilder, Stylesheets, Dokumente, Dateien, clientseitige Skripts, HTML-Seiten, HTML-Fragmente oder andere Inhalte handeln, die der Server nicht für jede Anforderung ändern muss. Die Anwendung kann Elemente zur Laufzeit erstellen und dem CDN verfügbar machen (z. B. durch Erstellen einer Liste mit aktuellen Schlagzeilen), führt diese Aktion aber nicht für jede Anforderung aus.
* Bereitstellen von öffentlichen statischen und freigegebenen Inhalten auf Geräten wie Mobiltelefonen und Tablet-PCs. Die Anwendung selbst ist ein Webdienst, der eine API für Clients bietet, die auf verschiedenen Geräten ausgeführt werden. Darüber hinaus kann das CDN statische Datasets (über den Webdienst) zur Verwendung durch die Clients bereitstellen, z. B. zum Generieren der Client-UI. Beispielsweise könnte das CDN zum Verteilen von JSON- oder XML-Dokumenten genutzt werden.
* Verwaltung vollständiger Websites, die nur aus öffentlichen statischen Inhalten für Clients bestehen, ohne dass dedizierte Compute-Ressourcen erforderlich sind.
* Streaming von Videodateien an den Client bei Bedarf. Videos profitieren von der niedrigen Latenz und der zuverlässigen Konnektivität aus den Rechenzentren auf der ganzen Welt, die CDN-Verbindungen anbieten. Microsoft Azure Media Services (AMS) ist in Azure CDN integriert, damit Inhalte zur weiteren Verteilung direkt für das CDN bereitgestellt werden können. Weitere Informationen finden Sie unter [Übersicht über Streamingendpunkte](/azure/media-services/media-services-streaming-endpoints-overview).
* Allgemeines Verbessern der Benutzerfreundlichkeit insbesondere für Benutzer mit großer Entfernung zu dem Datencenter, auf dem die Anwendung gehostet ist. Diese Benutzer würden andernfalls u. U. unter einer hohen Latenz leiden. Ein Großteil der Gesamtgröße des Inhalts in einer Webanwendung ist häufig statisch, und das Verwenden des CDN kann dazu beitragen, die Leistung und die allgemeine Benutzerfreundlichkeit aufrechtzuerhalten, während es gleichzeitig nicht erforderlich ist, die Anwendung in mehreren Rechenzentren bereitzustellen. Eine Liste mit den Azure CDN-Knotenstandorten finden Sie unter [POP-Standorte von Azure Content Delivery Network (CDN)](/azure/cdn/cdn-pop-locations/).
* Unterstützung von IoT-Lösungen (Internet of Things, Internet der Dinge). Die große Anzahl von Geräten und Appliances, die an einer IoT-Lösung beteiligt sind, kann für eine Anwendung schnell zu einer Belastung werden, wenn Firmwareupdates direkt auf jedes Gerät verteilt werden müssen.
* Bewältigung von Spitzen und zunehmender Nachfrage, ohne die Anwendung zu skalieren und ohne die aus der Skalierung folgenden höheren Betriebskosten hinnehmen zu müssen. Wenn zum Beispiel ein Update für ein Betriebssystem veröffentlicht wird, etwa für ein Hardwaregerät wie ein bestimmtes Routermodell, oder für ein Consumergerät, z. B. ein Smart-TV, steigt der Bedarf stark an, da dieses Update innerhalb kurzer Zeit von Millionen von Benutzern und Geräten heruntergeladen wird.

## <a name="challenges"></a>Herausforderungen

Es gibt verschiedene Herausforderungen, die beim Planen der CDN-Verwendung berücksichtigt werden müssen.  

* **Bereitstellung**. Entscheiden Sie, wo der Ursprung liegen soll, aus dem das CDN den Inhalt abruft, und ob Sie den Inhalt in mehreren Speichersystemen bereitstellen müssen. Berücksichtigen Sie den Prozess zum Bereitstellen von statischen Inhalten und Ressourcen. Möglicherweise müssen Sie z. B. einen separaten Schritt einführen, um den Inhalt in Azure Blob Storage zu laden.
* **Versionierung und Cachesteuerung**. Berücksichtigen Sie, wie Sie statische Inhalte aktualisieren und neue Versionen bereitstellen möchten. Machen Sie sich damit vertraut, wie das CDN die Zwischenspeicherung durchführt und die Gültigkeitsdauer (TTL) behandelt. Für Azure CDN finden Sie Informationen hierzu unter [Funktionsweise der Zwischenspeicherung](/azure/cdn/cdn-how-caching-works).
* **Testen**. Es kann schwierig sein, lokale Tests Ihrer CDN-Einstellungen durchzuführen, wenn Sie eine Anwendung lokal oder in einer Stagingumgebung entwickeln und testen.
* **Suchmaschinenoptimierung (SEO)**. Wenn Sie das CDN verwenden, wird der Inhalt, wie z. B. Bilder und Dokumente, von einer anderen Domäne aus ausgeliefert. Dies kann Auswirkungen auf die Suchmaschinenoptimierung (SEO, Search Engine Optimization) für diesen Inhalt haben.
* **Sicherheit für den Inhalt**. Nicht für alle CDNs sind alle Formen der Zugriffssteuerung für den Inhalt verfügbar. Einige CDN-Dienste, z.B. Azure CDN, unterstützen die tokenbasierte Authentifizierung zum Schützen des CDN-Inhalts. Weitere Informationen finden Sie unter [Schützen von Azure Content Delivery Network-Assets mit Tokenauthentifizierung](/azure/cdn/cdn-token-auth).
* **Sicherheit für Clients**. Clients stellen möglicherweise eine Verbindung aus einer Umgebung heraus her, die keinen Zugriff auf Ressourcen im CDN zulässt. Dies könnte eine Umgebung mit eingeschränkter Sicherheit sein, in der der Zugriff auf eine Gruppe bekannter Quellen beschränkt ist oder in der das Laden von Ressourcen nur vom Seitenursprung möglich ist. Für solche Fälle ist eine Fallbackimplementierung erforderlich.
* **Resilienz**. Das CDN ist eine potenzielle einzelne Fehlerquelle („Single Point of Failure“) für eine Anwendung. 

In folgenden Szenarien kann CDN weniger nützlich sein:  

* Wenn der Inhalt eine niedrige Trefferrate hat, wird unter Umständen innerhalb des Gültigkeitszeitraums nur wenige Male darauf zugegriffen (bestimmt durch die Einstellung für die Gültigkeitsdauer). 
* Wenn die Daten privat sind, z. B. bei großen Unternehmen oder Lieferketten-Ökosystemen.

## <a name="general-guidelines-and-good-practices"></a>Allgemeine Richtlinien und bewährte Verfahren

Die Verwendung eines CDN ist eine gute Möglichkeit, um die Belastung Ihrer Anwendung zu minimieren und die Verfügbarkeit und Leistung zu maximieren. Erwägen Sie den Einsatz dieser Strategie für alle von Ihrer Anwendung genutzten Inhalte und Ressourcen, sofern sie dafür geeignet sind. Berücksichtigen Sie beim Entwerfen Ihrer Strategie zur CDN-Nutzung folgende Punkte.

### <a name="deployment"></a>Bereitstellung
Statische Inhalte müssen möglicherweise unabhängig von der Anwendung bereitgestellt werden, wenn Sie sie nicht in das Anwendungsbereitstellungspaket oder den Anwendungsbereitstellungsprozess einschließen. Berücksichtigen Sie, wie sich dies auf den Versionierungsansatz auswirkt, den Sie zum Verwalten der Anwendungskomponenten und des statischen Ressourceninhalts verwenden.

Erwägen Sie die Nutzung von Bündelungs- und Minimierungsverfahren, um die Ladezeiten für Clients zu reduzieren. Bei der Bündelung werden mehrere Dateien zu einer einzelnen Datei kombiniert. Bei der Minimierung werden unnötige Zeichen aus Skripts und CSS-Dateien entfernt, ohne dass die Funktionalität geändert wird.

Wenn Sie den Inhalt an einem zusätzlichen Speicherort bereitstellen müssen, wird dafür ein zusätzlicher Schritt im Bereitstellungsprozess benötigt. Wenn die Anwendung den Inhalt für das CDN aktualisiert, vielleicht in regelmäßigen Abständen oder als Reaktion auf ein Ereignis, muss sie den aktualisierten Inhalt an allen zusätzlichen Speicherorten sowie auf dem Endpunkt des CDN speichern.

Überlegen Sie sich, wie Sie die lokale Entwicklung und das Testen verarbeiten, wenn erwartet wird, dass einige statische Inhalte von einem CDN bereitgestellt werden. Beispielsweise können Sie die Inhalte im Rahmen Ihres Buildskripts für das CDN bereitstellen. Alternativ hierzu können Sie Kompilierungsdirektiven oder Flags verwenden, um zu steuern, wie die Ressourcen von der Anwendung geladen werden. Im Debugmodus kann die Anwendung beispielsweise statische Ressourcen aus einem lokalen Ordner laden. Im Releasemodus nutzt die Anwendung das CDN.

Erwägen Sie die verschiedenen Optionen für die Dateikomprimierung, z.B. GZip (GNU-Zip). Die Komprimierung kann auf dem Ursprungsserver über die Hosting-Webanwendung oder direkt auf den Edgeservern durch das CDN erfolgen. Weitere Informationen finden Sie unter [Verbessern der Leistung durch Komprimieren von Dateien in Azure CDN](/azure/cdn/cdn-improve-performance).


### <a name="routing-and-versioning"></a>Routing und Versionierung
Möglicherweise müssen Sie zu verschiedenen Zeitpunkten unterschiedliche CDN-Instanzen verwenden. Beispielsweise empfiehlt es sich u. U. beim Bereitstellen der neuen Version einer Anwendung, ein neues CDN zu verwenden und das alte CDN (mit Inhalt in einem älteren Format) für die vorherigen Versionen beizubehalten. Wenn Sie Azure Blob Storage als Inhaltsursprung verwenden, können Sie ein separates Speicherkonto oder einen separaten Container erstellen und mit dem CDN-Endpunkt darauf verweisen. 

Verwenden Sie nicht die Abfragezeichenfolge, um unterschiedliche Versionen der Anwendung in Links zu Ressourcen im CDN zu kennzeichnen, denn die Abfragezeichenfolge ist ein Teil des Ressourcennamens (der Blob-Name), wenn Inhalte aus Azure Blob Storage abgerufen werden. Diese Herangehensweise kann sich auch darauf auswirken, wie der Client Ressourcen zwischenspeichert.

Das Bereitstellen neuer Versionen von statischen Inhalten beim Aktualisieren einer Anwendung kann eine Herausforderung darstellen, wenn die vorherigen Ressourcen im CDN zwischengespeichert sind. Weitere Informationen finden Sie unten im Abschnitt zur Cachesteuerung.

Erwägen Sie, den Zugriff auf den CDN-Inhalt nach Land zu beschränken. Mit Azure CDN können Sie Anforderungen auf Basis des Ursprungslands filtern und den bereitgestellten Inhalt einschränken. Weitere Informationen finden Sie unter [Einschränken des Zugriffs auf Inhalte nach Ländern](/azure/cdn/cdn-restrict-access-by-country/).

### <a name="cache-control"></a>Cachesteuerung
Bedenken Sie, wie das Zwischenspeichern innerhalb des Systems verwaltet werden soll. Für Azure CDN können Sie beispielsweise globale Cacheregeln festlegen und anschließend die benutzerdefinierte Zwischenspeicherung für bestimmte Ursprungsendpunkte festlegen. Außerdem können Sie steuern, wie die Zwischenspeicherung in einem CDN durchgeführt wird, indem Sie Header mit Cacheanweisungen am Ursprung senden. 

Weitere Informationen finden Sie unter [Funktionsweise der Zwischenspeicherung](/azure/cdn/cdn-how-caching-works).

Um zu verhindern, dass Objekte im CDN verfügbar sind, können Sie sie aus dem Ursprung entfernen oder den CDN-Endpunkt löschen. Bei Verwendung des Blobspeichers können Sie den Container oder das Blob aber auch als privat kennzeichnen. Die Elemente werden jedoch erst nach Ablauf der Gültigkeitsdauer entfernt. Sie können einen CDN-Endpunkt auch manuell endgültig löschen.

### <a name="security"></a>Sicherheit

Das CDN kann Inhalte per HTTPS (SSL) bereitstellen, indem das vom CDN bereitgestellte Zertifikat verwendet wird, sowie auch per Standard-HTTP. Zur Vermeidung von Browserwarnungen zu gemischten Inhalten müssen Sie unter Umständen HTTPS verwenden, um statische Inhalte anzufordern, die auf per HTTPS geladenen Seiten angezeigt werden.

Wenn Sie statische Objekte, z.B. Schriftdateien, mit dem CDN bereitstellen, können ggf. Probleme aufgrund einer Richtlinie desselben Ursprungs auftreten, falls Sie einen *XMLHttpRequest*-Aufruf zum Anfordern dieser Ressourcen aus einer anderen Domäne verwenden. Viele Webbrowser verhindern CORS (Cross-Origin Resource Sharing, ursprungsübergreifende Ressourcenfreigabe), sofern der Webserver nicht so konfiguriert wurde, dass die entsprechenden Antwortheader festgelegt werden. Sie können das CDN konfigurieren, um CORS zu unterstützen, indem Sie eine der folgenden Methoden verwenden:

* Konfigurieren Sie das CDN, um den Antworten CORS-Header hinzuzufügen. Weitere Informationen finden Sie unter [Verwendung von Azure CDN mit CORS](/azure/cdn/cdn-cors). 
* Fügen Sie dem Speicherendpunkt CORS-Regeln hinzu, wenn Azure Blob Storage der Ursprung ist. Weitere Informationen finden Sie unter [Cross-Origin Resource Sharing (CORS)-Support für die Azure Storage-Dienste](http://msdn.microsoft.com/library/azure/dn535601.aspx).
* Konfigurieren Sie die Anwendung, um die CORS-Header festzulegen. Informationen hierzu finden Sie beispielsweise unter [Aktivieren von Cross-Origin-Anforderungen (CORS)](/aspnet/core/security/cors) in der ASP.NET Core-Dokumentation.

### <a name="cdn-fallback"></a>CDN-Fallback
Berücksichtigen Sie, wie Ihre Anwendung einen Fehler oder einen vorübergehenden Ausfall des CDN verarbeitet. Clientanwendungen können möglicherweise Kopien der Ressourcen verwenden, die bei früheren Anforderungen lokal (auf dem Client) zwischengespeichert wurden, oder Sie können Code einbauen, der einen Ausfall erkennt und bei Nichtverfügbarkeit des CDNs die Ressourcen vom Ursprung anfordert (aus dem Anwendungsordner oder Azure-Blob-Container, der die Ressourcen enthält).
