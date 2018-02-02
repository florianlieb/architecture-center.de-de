---
title: Anleitungen zum Content Delivery Network
description: "Anleitungen zum Content Delivery Network (CDN) für die Bereitstellung von in Azure gehosteten Inhalten mit hoher Bandbreite."
author: dragon119
ms.date: 09/30/2016
pnp.series.title: Best Practices
ms.openlocfilehash: fffe0b0523c0a9c817f4346744ff3b5e3f11dede
ms.sourcegitcommit: cf207fd10110f301f1e05f91eeb9f8dfca129164
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/29/2018
---
# <a name="content-delivery-network"></a>Content Delivery Network
[!INCLUDE [header](../_includes/header.md)]

Das Microsoft Azure [Content Delivery Network (CDN)](/azure/cdn/cdn-overview) bietet Entwicklern eine globale Lösung für die Bereitstellung von Inhalten mit hoher Bandbreite, die in Azure oder an einem beliebigen anderen Ort gehostet werden. Mit einem CDN können Sie öffentlich verfügbare Objekte zwischenspeichern, die aus dem Azure-Blobspeicher, einer Webanwendung, einem virtuellen Computer, einem Anwendungsordner oder einem anderen HTTP-/HTTPS-Speicherort geladen werden. Der zum Zwischenspeichern verwendete CDN-Cache kann an strategischen Standorten angesiedelt werden, um beim Übermitteln des Inhalts an die Benutzer eine maximale Bandbreite zu gewährleisten. Ein CDN wird normalerweise zum Bereitstellen statischer Inhalte wie Bilder, Stylesheets, Dokumente, Dateien, clientseitige Skripts und HTML-Seiten verwendet.

Sie können das CDN auch als Cache zum Ausliefern von dynamischem Inhalt verwenden, z. B. für einen PDF-Bericht oder ein Diagramm, das von spezifischen Eingaben abhängig ist. Wenn mehrere Benutzer die gleichen Eingabewerte eingeben, sollte das Ergebnis identisch sein.

Die Hauptvorteile beim Verwenden des CDNs sind niedrigere Latenz und schnellere Bereitstellung des Inhalts für die Benutzer, und zwar unabhängig von der Entfernung des geografischen Standorts der Benutzer zu dem Datencenter, in dem die Anwendung gehostet wird.  

![CDN-Diagramm](./images/cdn/CDN.png)

Mithilfe des CDN können Sie u.U. auch die Auslastung der Anwendung verringern, weil der Verarbeitungsaufwand für den Zugriff auf den Inhalt und die Inhaltsbereitstellung wegfällt. Durch diese Reduzierung der Auslastung können Sie die Leistung und Skalierbarkeit der Anwendung erhöhen sowie die Hostingkosten senken, denn Sie benötigen weniger Verarbeitungsressourcen, um ein bestimmtes Leistungs- und Verfügbarkeitsniveau zu erreichen.

## <a name="how-and-why-a-cdn-is-used"></a>Wie und warum ein CDN verwendet wird
Ein CDN wird häufig für Folgendes verwendet:  

* Bereitstellen statischer Ressourcen für Clientanwendungen, häufig von einer Website. Bei diesen Ressourcen kann es sich um Bilder, Stylesheets, Dokumente, Dateien, clientseitige Skripts, HTML-Seiten, HTML-Fragmente oder andere Inhalte handeln, die der Server nicht für jede Anforderung ändern muss. Die Anwendung kann Elemente zur Laufzeit erstellen und dem CDN verfügbar machen (z. B. durch Erstellen einer Liste mit aktuellen Schlagzeilen), führt diese Aktion aber nicht für jede Anforderung aus.
* Bereitstellen von öffentlichen statischen und freigegebenen Inhalten auf Geräten wie Mobiltelefonen und Tablet-PCs. Die Anwendung selbst ist ein Webdienst, der eine API für Clients bietet, die auf verschiedenen Geräten ausgeführt werden. Darüber hinaus kann das CDN statische Datasets (über den Webdienst) zur Verwendung durch die Clients bereitstellen, z. B. zum Generieren der Client-UI. Beispielsweise könnte das CDN zum Verteilen von JSON- oder XML-Dokumenten genutzt werden.
* Verwaltung vollständiger Websites, die nur aus öffentlichen statischen Inhalten für Clients bestehen, ohne dass dedizierte Compute-Ressourcen erforderlich sind.
* Streaming von Videodateien an den Client bei Bedarf. Videos profitieren von der niedrigen Latenz und der zuverlässigen Konnektivität aus den Rechenzentren auf der ganzen Welt, die CDN-Verbindungen anbieten. Microsoft Azure Media Services (AMS) ist in Azure CDN integriert, damit Inhalte zur weiteren Verteilung direkt für das CDN bereitgestellt werden können. Weitere Informationen finden Sie unter [Übersicht über Streamingendpunkte](/azure/media-services/media-services-streaming-endpoints-overview).
* Allgemeines Verbessern der Benutzerfreundlichkeit insbesondere für Benutzer mit großer Entfernung zu dem Datencenter, auf dem die Anwendung gehostet ist. Diese Benutzer würden andernfalls u. U. unter einer hohen Latenz leiden. Ein Großteil der Gesamtgröße des Inhalts in einer Webanwendung ist häufig statisch, und das Verwenden des CDN kann dazu beitragen, die Leistung und die allgemeine Benutzerfreundlichkeit aufrechtzuerhalten, während es gleichzeitig nicht erforderlich ist, die Anwendung in mehreren Rechenzentren bereitzustellen.
* Verarbeiten der wachsenden Last bei Anwendungen, die Lösungen für das Internet der Dinge (IoT, Internet of Things) unterstützen. Die große Anzahl solcher Geräte und Appliances kann eine Anwendung schnell überlasten, wenn sie  Broadcastmeldungen verarbeiten und die Firmwareupdateverteilung für jedes Gerät direkt verwalten müsste.
* Bewältigung von Spitzen und zunehmender Nachfrage, ohne die Anwendung zu skalieren und ohne die aus der Skalierung folgenden höheren Betriebskosten hinnehmen zu müssen. Wenn zum Beispiel ein Update für ein Betriebssystem veröffentlicht wird, etwa für ein Hardwaregerät wie ein bestimmtes Routermodell, oder für ein Consumergerät, z. B. ein Smart-TV, steigt der Bedarf stark an, da dieses Update innerhalb kurzer Zeit von Millionen von Benutzern und Geräten heruntergeladen wird.

Die folgende Liste enthält Beispiele für die mittlere Zeit, die an verschiedenen geografischen Standorten bis zum ersten Byte benötigt wird. Die Zielwebrolle wird in der Azure-Region „USA, Westen“ bereitgestellt. Es gibt eine starke Wechselbeziehung zwischen dem größeren Boost durch das CDN und der Entfernung zu einem CDN-Knoten. Eine vollständige Liste aller Azure CDN-Knotenstandorte finden Sie unter [POP-Standorte von Azure Content Delivery Network (CDN)](/azure/cdn/cdn-pop-locations/).

|  | Zeit (ms) bis zum ersten Byte (Ursprung) | Zeit (ms) bis zum ersten Byte (CDN) | % Beschleunigung durch CDN |
| --- | --- | --- | --- |
| \*San Jose, Kalifornien, USA |47,5 |46,5 |2% |
| \*\*Dulles, VA, USA |109 |40,5 |169 % |
| Buenos Aires, AR |210 |151 |39 % |
| \*London, Großbritannien |195 |44 |343 % |
| Shanghai, CN |242 |206 |17 % |
| \*Singapur |214 |74 |189% |
| \*Tokio, JP |163 |48 |204% |
| Seoul, KR |190 |190 |0 % |

\* Hat einen Azure CDN-Knoten in derselben Stadt.  
\*\* Hat einen Azure CDN-Knoten in einer Nachbarstadt.  

## <a name="challenges"></a>Herausforderungen
Es gibt verschiedene Herausforderungen, die beim Planen der CDN-Verwendung zu berücksichtigen sind:  

* **Bereitstellung**. Entscheiden Sie, wo der Ursprung liegen soll, aus dem das CDN den Inhalt abruft, und ob Sie den Inhalt in mehreren Speichersystemen bereitstellen müssen (z.B. im CDN und an einem alternativen Speicherort).

  Ihr Anwendungsbereitstellungsmechanismus muss den Prozess zum Bereitstellen statischer Inhalte und Ressourcen sowie das Bereitstellen der Anwendungsdateien, z.B. ASPX-Seiten, berücksichtigen. Möglicherweise müssen Sie z. B. einen separaten Schritt einführen, um den Inhalt in Azure Blob Storage zu laden.
* **Versionierung und Cachesteuerung**. Berücksichtigen Sie, wie Sie statische Inhalte aktualisieren und neue Versionen bereitstellen möchten. Der Inhalt des CDN wird beim Verwenden des Azure-Portals unter Umständen [bereinigt](/azure/cdn/cdn-purge-endpoint/), wenn neue Versionen Ihrer Inhalte verfügbar sind. Dies ist eine ähnliche Herausforderung wie das Verwalten der clientseitigen Zwischenspeicherung, wie sie z.B. in einem Webbrowser auftritt.
* **Testen**. Es kann schwierig sein, lokale Tests Ihrer CDN-Einstellungen durchzuführen, wenn Sie eine Anwendung lokal oder in einer Stagingumgebung entwickeln und testen.
* **Suchmaschinenoptimierung (SEO)**. Wenn Sie das CDN verwenden, wird der Inhalt, wie z. B. Bilder und Dokumente, von einer anderen Domäne aus ausgeliefert. Dies kann Auswirkungen auf die Suchmaschinenoptimierung (SEO, Search Engine Optimization) für diesen Inhalt haben.
* **Sicherheit für den Inhalt**. Viele CDN-Dienste, wie z. B. Azure CDN, bieten derzeit keinerlei Zugriffssteuerung für den Inhalt an.
* **Sicherheit für Clients**. Clients stellen möglicherweise eine Verbindung aus einer Umgebung heraus her, die keinen Zugriff auf Ressourcen im CDN zulässt. Dies könnte eine Umgebung mit eingeschränkter Sicherheit sein, in der der Zugriff auf eine Gruppe bekannter Quellen beschränkt ist oder in der das Laden von Ressourcen nur vom Seitenursprung möglich ist. Für solche Fälle ist eine Fallbackimplementierung erforderlich.
* **Resilienz**. Das CDN ist eine potenzielle einzelne Fehlerquelle („Single Point of Failure“) für eine Anwendung. Es hat eine niedrigere Verfügbarkeits-SLA als Blob Storage (was zum direkten Bereitstellen von Inhalten verwendet werden kann), sodass Sie möglicherweise einen Fallbackmechanismus für kritische Inhalte implementieren müssen.

  Im Azure-Portal können Sie die Verfügbarkeit des CDN-Inhalts, die Bandbreite, die übertragenen Daten, die Treffer, die Cachetrefferquote und die Cachemetriken in [Echtzeit](/azure/cdn/cdn-real-time-stats/) überwachen und [Berichte zusammenfassen](/azure/cdn/cdn-analyze-usage-patterns/).

In folgenden Szenarien kann CDN weniger nützlich sein:  

* Wenn der Inhalt eine niedrige Trefferrate hat, wird unter Umständen innerhalb des Gültigkeitszeitraums nur wenige Male darauf zugegriffen (bestimmt durch die Einstellung für die Gültigkeitsdauer). Beim ersten Herunterladen eines Elements fallen zwei Transaktionsgebühren an (vom Ursprung in das CDN und dann aus dem CDN an den Kunden).
* Wenn die Daten privat sind, z. B. bei großen Unternehmen oder Lieferketten-Ökosystemen.

## <a name="general-guidelines-and-good-practices"></a>Allgemeine Richtlinien und bewährte Verfahren
Das Verwenden des CDN ist eine gute Möglichkeit, um die Belastung Ihrer Anwendung zu minimieren und Verfügbarkeit sowie Leistung zu maximieren. Erwägen Sie den Einsatz dieser Strategie für alle von Ihrer Anwendung genutzten Inhalte und Ressourcen, sofern sie dafür geeignet sind. Berücksichtigen Sie beim Entwerfen Ihrer Strategie zur CDN-Nutzung folgende Punkte:  

### <a name="origin"></a>Origin
Zum Bereitstellen von Inhalten über das CDN müssen Sie einfach einen HTTP- und/oder HTTPS-[Endpunkt](/azure/cdn/cdn-create-new-endpoint) angeben, den der CDN-Dienst für den Zugriff auf den Inhalt und für die Zwischenspeicherung verwendet.

Der Endpunkt kann einen Azure Blob Storage-Container angeben, der den statischen Inhalt enthält, den Sie über das CDN bereitstellen möchten. Der Container muss als öffentlich markiert sein. Nur Blobs in einem öffentlichen Container, die öffentlichen Lesezugriff haben, sind über das CDN verfügbar.

Der Endpunkt kann einen Ordner namens **cdn** im Stammverzeichnis einer Compute-Ebene der Anwendung (z.B. eine Webrolle oder ein virtueller Computer) angeben. Die Ergebnisse aus Anforderungen für Ressourcen, einschließlich dynamischer Ressourcen wie z. B. ASPX-Seiten, werden im CDN zwischengespeichert. Der minimale Zeitraum zum Zwischenspeichern beträgt 300 Sekunden. Kürzere Zeiträume verhindern, dass der Inhalt im CDN bereitgestellt wird. (Weitere Informationen finden Sie unten im Abschnitt *Cachesteuerung*.)

Bei der Verwendung von Azure Web-Apps wird der Endpunkt durch Auswählen des Standorts beim Erstellen der CDN-Instanz auf den Stammordner der Website festgelegt. Der gesamte Inhalt der Website ist über das CDN verfügbar.

In den meisten Fällen wird die Flexibilität und Kontrolle erhöht, wenn Ihr CDN-Endpunkt auf einen Ordner innerhalb einer Compute-Ebene der Anwendung verweist. Auf diese Weise wird zum Beispiel das Verwalten von aktuellen und zukünftigen Routinganforderungen und das dynamische Generieren statischer Inhalte, wie etwa Miniaturansichten, vereinfacht.

Sie können [Abfragezeichenfolgen](/azure/cdn/cdn-query-string/) verwenden, um Objekte im Cache zu unterscheiden, wenn Inhalte von dynamischen Quellen, z.B. ASPX-Seiten, bereitgestellt werden. Allerdings kann dieses Verhalten durch eine Einstellung im Azure-Portal bei der Angabe des CDN-Endpunkts deaktiviert werden. Bei der Bereitstellung von Inhalten aus Blob Storage werden Abfragezeichenfolgen als Zeichenfolgenliterale behandelt, sodass zwei Elemente mit dem gleichen Namen und unterschiedlichen Abfragezeichenfolgen als separate Elemente im CDN gespeichert werden.

Sie können die URL-Umschreibung für Ressourcen wie Skripts und für andere Inhalte nutzen, um zu vermeiden, dass die Dateien in den CDN-Ursprungsordner verschoben werden.

Wenn Sie Azure Storage Blobs zum Speichern von Inhalten für das CDN verwenden, wird bei der URL der Ressourcen in Blobs für den Container- und den Blob-Namen die Groß- und Kleinschreibung beachtet.

Wenn Sie benutzerdefinierte Ursprünge oder Azure Web-Apps verwenden, geben Sie den Pfad zur CDN-Instanz in den Links zu Ressourcen an. Das Folgenden gibt beispielsweise eine Bilddatei im Ordner **images** der Website an, die über das CDN bereitgestellt wird:

```XML
<img src="http://[your-cdn-endpoint].azureedge.net/Images/image.jpg" />
```

### <a name="deployment"></a>Bereitstellung
Statische Inhalte müssen möglicherweise unabhängig von der Anwendung bereitgestellt werden, wenn Sie sie nicht in das Anwendungsbereitstellungspaket oder den Anwendungsbereitstellungsprozess einschließen. Berücksichtigen Sie, wie sich dies auf den Versionierungsansatz auswirkt, den Sie zum Verwalten der Anwendungskomponenten und des statischen Ressourceninhalts verwenden.

Berücksichtigen Sie, wie die Bündelung (Kombinieren mehrerer Dateien in einer Datei) und die Minimierung (Entfernen unnötiger Zeichen wie Leerzeichen, Zeilenumbrüche, Kommentare und andere Zeichen) für Skript- und CSS-Dateien verarbeitet werden. Diese häufig verwendeten Techniken können Ladezeiten für Clients reduzieren und sind mit der Bereitstellung von Inhalten über das CDN kompatibel. Weitere Informationen finden Sie unter [Bündelung und Minimierung](http://www.asp.net/mvc/tutorials/mvc-4/bundling-and-minification).

Wenn Sie den Inhalt an einem zusätzlichen Speicherort bereitstellen müssen, wird dafür ein zusätzlicher Schritt im Bereitstellungsprozess benötigt. Wenn die Anwendung den Inhalt für das CDN aktualisiert, vielleicht in regelmäßigen Abständen oder als Reaktion auf ein Ereignis, muss sie den aktualisierten Inhalt an allen zusätzlichen Speicherorten sowie auf dem Endpunkt des CDN speichern.

Sie können keinen CDN-Endpunkt für eine Anwendung im lokalen Azure-Emulator in Visual Studio einrichten. Diese Einschränkung betrifft Komponententests, Funktionstests und abschließende Tests vor der Bereitstellung. Um Ihre Ziele trotzdem zu erreichen, müssen Sie einen alternativen Mechanismus implementieren. Sie könnten beispielsweise vorab den Inhalt mit einer benutzerdefinierten Anwendung oder einem entsprechenden Programm im CDN bereitstellen und die Tests während des Cachezeitraums durchführen. Alternativ können Sie auch Kompilierungsdirektiven oder globale Konstanten verwenden, um zu steuern, von wo die Anwendung die Ressourcen lädt. Bei der Ausführung im Debugmodus können z. B. Ressourcen wie clientseitige Skriptpakete und andere Inhalte aus einem lokalen Ordner geladen werden, und CDN wird bei der Ausführung im Releasemodus verwendet.

Bedenken Sie, welchen Ansatz für die Komprimierung das CDN unterstützen soll:

* Sie können auf Ihrem Ursprungsserver die [Komprimierung aktivieren](/azure/cdn/cdn-improve-performance/). In diesem Fall unterstützt das CDN die Komprimierung standardmäßig und sendet komprimierten Inhalt in einem Format wie ZIP oder GZIP an die Clients. Wenn Sie einen Anwendungsordner als CDN-Endpunkt verwenden, kann der Server einen Teil des Inhalts automatisch auf die gleiche Weise komprimieren wie beim direkten Bereitstellen für einen Webbrowser oder einen anderen Clienttyp. Das Format hängt vom Wert des **Accept-Encoding**-Headers in der vom Client gesendeten Anforderung ab. Als Standardverfahren wird der Inhalt in Azure automatisch komprimiert, wenn die CPU-Auslastung unter 50 % liegt. Wenn Sie einen Clouddienst zum Hosten der Anwendung verwenden, kann das Ändern der Einstellungen die Verwendung eines Starttasks erfordern, um die Komprimierung der dynamischen Ausgabe in IIS zu aktivieren. Weitere Informationen finden Sie unter [Enabling gzip compression with Microsoft Azure CDN through a Web Role](http://blogs.msdn.com/b/avkashchauhan/archive/2012/03/05/enableing-gzip-compression-with-windows-azure-cdn-through-web-role.aspx) (Aktivieren der GZIP-Komprimierung mit Azure CDN über eine Webrolle).
* Sie können die Komprimierung direkt auf CDN-Edgeservern aktivieren. In diesem Fall komprimiert CDN die Dateien und sendet sie an die Endbenutzer. Weitere Informationen finden Sie unter [Azure CDN-Komprimierung](/azure/cdn/cdn-improve-performance/).

### <a name="routing-and-versioning"></a>Routing und Versionierung
Möglicherweise müssen Sie zu verschiedenen Zeitpunkten unterschiedliche CDN-Instanzen verwenden. Beispielsweise empfiehlt es sich u. U. beim Bereitstellen der neuen Version einer Anwendung, ein neues CDN zu verwenden und das alte CDN (mit Inhalt in einem älteren Format) für die vorherigen Versionen beizubehalten. Wenn Sie Azure Blob Storage als Inhaltsursprung verwenden, können Sie ein separates Speicherkonto oder einen separaten Container erstellen und mit dem CDN-Endpunkt darauf verweisen. Bei Verwendung des cdn-Stammordners innerhalb der Anwendung als CDN-Endpunkt können Sie URL-Umschreibungstechniken für die Umleitung von Anforderungen in einen anderen Ordner verwenden.

Verwenden Sie nicht die Abfragezeichenfolge, um unterschiedliche Versionen der Anwendung in Links zu Ressourcen im CDN zu kennzeichnen, denn die Abfragezeichenfolge ist ein Teil des Ressourcennamens (der Blob-Name), wenn Inhalte aus Azure Blob Storage abgerufen werden. Diese Herangehensweise kann sich auch darauf auswirken, wie der Client Ressourcen zwischenspeichert.

Das Bereitstellen neuer Versionen von statischen Inhalten beim Aktualisieren einer Anwendung kann eine Herausforderung darstellen, wenn die vorherigen Ressourcen im CDN zwischengespeichert sind. Weitere Informationen finden Sie im Abschnitt *Cachesteuerung*.

Erwägen Sie, den Zugriff auf den CDN-Inhalt nach Land zu beschränken. Mit Azure CDN können Sie Anforderungen auf Basis des Ursprungslands filtern und den bereitgestellten Inhalt einschränken. Weitere Informationen finden Sie unter [Einschränken des Zugriffs auf Inhalte nach Ländern](/azure/cdn/cdn-restrict-access-by-country/).

### <a name="cache-control"></a>Cachesteuerung
Bedenken Sie, wie das Zwischenspeichern innerhalb des Systems verwaltet werden soll. Wenn Sie beispielsweise einen Ordner als CDN-Ursprung verwenden, können Sie die Cachefähigkeit von Seiten, die den Inhalt generieren, sowie die Inhaltsablaufzeit für alle Ressourcen in einem bestimmten Ordner angeben. Sie können auch Cacheeigenschaften für das CDN und für den Client mithilfe von standardmäßigen HTTP-Headern angeben. Obwohl Sie das Zwischenspeichern wahrscheinlich bereits auf dem Server und dem Client verwalten, kann das CDN Sie darauf aufmerksam machen, wie Ihre Inhalte wo zwischengespeichert werden.

Um zu verhindern, dass Objekte im CDN verfügbar sind, können Sie sie aus dem Ursprung (Blobcontainer oder *cdn*-Stammordner der Anwendung) entfernen oder den CDN-Endpunkt löschen. Bei Verwendung des Blobspeichers können Sie den Container oder das Blob aber auch als privat kennzeichnen. Allerdings werden Elemente nur aus dem CDN entfernt, wenn ihre Gültigkeitsdauer (Time-To-Live, TTL) abläuft. Wenn keine Cachegültigkeitsdauer angegeben wird (z.B. wenn der Inhalt aus Blob Storage geladen wird), wird er bis zu sieben Tage lang im CDN zwischengespeichert.  Sie können auch manuell [einen CDN-Endpunkt bereinigen](/azure/cdn/cdn-purge-endpoint/).

In einer Webanwendung können Sie die Zwischenspeicherung und die Gültigkeitsdauer für den gesamten Inhalt mit dem *clientCache*-Element im Abschnitt *system.webServer/staticContent* der web.config-Datei festlegen. Beachten Sie: Wenn Sie eine web.config-Datei in einem Ordner ablegen, wirkt sich dies auf alle Dateien in diesem Ordner und in allen Unterordnern aus.

Wenn Sie den Inhalt für das CDN dynamisch erstellen (z.B. in Ihrem Anwendungscode), sollten Sie sicherstellen, dass Sie die *Cache.SetExpires*-Eigenschaft auf jeder Seite angeben. Die Ausgabe von Seiten, die die standardmäßige Cachefähigkeitseinstellung *public* verwenden, wird vom CDN nicht zwischengespeichert.  Legen Sie die Cachegültigkeitsdauer auf einen passenden Wert fest, um sicherzustellen, dass der Inhalt nicht verworfen, sondern in sehr kurzen Intervallen aus der Anwendung neu geladen wird.  

### <a name="security"></a>Sicherheit
Das CDN kann Inhalte per HTTPS (SSL) bereitstellen, indem das vom CDN bereitgestellte Zertifikat verwendet wird, sowie auch per Standard-HTTP. Zur Vermeidung von Browserwarnungen zu gemischten Inhalten müssen Sie unter Umständen HTTPS verwenden, um statische Inhalte anzufordern, die auf per HTTPS geladenen Seiten angezeigt werden.

Wenn Sie statische Objekte, z.B. Schriftdateien, mit dem CDN bereitstellen, können ggf. Probleme aufgrund einer Richtlinie desselben Ursprungs auftreten, falls Sie einen *XMLHttpRequest*-Aufruf zum Anfordern dieser Ressourcen aus einer anderen Domäne verwenden. Viele Webbrowser verhindern CORS (Cross-Origin Resource Sharing, ursprungsübergreifende Ressourcenfreigabe), sofern der Webserver nicht so konfiguriert wurde, dass die entsprechenden Antwortheader festgelegt werden. Sie können das CDN konfigurieren, um CORS zu unterstützen, indem Sie eine der folgenden Methoden verwenden:

* Nutzen Sie die CDN-Regel-Engine, um den Antworten CORS-Header hinzuzufügen. Diese Methode ist normalerweise am besten geeignet, da sowohl Platzhalter als auch mehrere spezifisch zulässige Ursprünge unterstützt werden. Weitere Informationen finden Sie unter [Verwendung von Azure CDN mit CORS](https://docs.microsoft.com/en-us/azure/cdn/cdn-cors). 
* Fügen Sie den Diensteigenschaften eine *CorsRule* hinzu. Sie können diese Methode nutzen, wenn der Ursprung, von dem aus Sie Inhalte bereitstellen, Azure Blob Storage ist. Die Regel kann die zulässigen Ursprünge für CORS-Anforderungen, die zulässigen Methoden wie z. B. GET und das maximale Alter in Sekunden für die Regel angeben (der Zeitraum, in dem der Client die verknüpften Ressourcen nach dem Laden des ursprünglichen Inhalts anfordern muss). Wenn Sie CORS für Storage zur Verwendung mit dem CDN festlegen, wird nur der Platzhalter „*“ für die Liste mit den zulässigen Ursprüngen unterstützt. Weitere Informationen finden Sie unter [Cross-Origin Resource Sharing (CORS)-Support für die Azure Storage-Dienste](http://msdn.microsoft.com/library/azure/dn535601.aspx).
* Konfigurieren Sie Regeln für die ausgehende Richtung in der Anwendungskonfigurationsdatei, um einen *Access-Control-Allow-Origin*-Header für alle Antworten festzulegen. Sie können diese Methode verwenden, wenn auf dem Ursprungsserver IIS ausgeführt wird. Wenn Sie diese Methode mit CDN nutzen, wird nur der Platzhalter „*“ für die Liste mit den zulässigen Ursprüngen unterstützt. Weitere Informationen zum Verwenden der Regeln für das Umschreiben finden Sie im Thema zum [URL-Umschreibemodul](http://www.iis.net/learn/extensions/url-rewrite-module).

### <a name="custom-domains"></a>Benutzerdefinierte Domänen
Das Azure CDN ermöglicht Ihnen das Angeben eines [benutzerdefinierten Domänennamens](/azure/cdn/cdn-map-content-to-custom-domain/) und die Verwendung des Namens zum Zugriff auf Ressourcen über das CDN. Sie können einen benutzerdefinierten Unterdomänennamen auch mit dem *CNAME* -Eintrag in DNS einrichten. Mit diesem Ansatz erhalten Sie eine zusätzliche Abstraktions- und Kontrollebene.

Wenn Sie einen *CNAME*verwenden, können Sie nicht gleichzeitig SSL verwenden, weil das CDN ein eigenes einzelnes SSL-Zertifikat verwendet, und dieses Zertifikat passt nicht zu Ihren benutzerdefinierten Domänen-/Unterdomänennamen.

### <a name="cdn-fallback"></a>CDN-Fallback
Berücksichtigen Sie, wie Ihre Anwendung einen Fehler oder einen vorübergehenden Ausfall des CDN verarbeitet. Clientanwendungen können möglicherweise Kopien der Ressourcen verwenden, die bei früheren Anforderungen lokal (auf dem Client) zwischengespeichert wurden, oder Sie können Code einbauen, der einen Ausfall erkennt und bei Nichtverfügbarkeit des CDNs die Ressourcen vom Ursprung anfordert (aus dem Anwendungsordner oder Azure-Blob-Container, der die Ressourcen enthält).

Im Beispiel unten werden die Fallbackmechanismen mit [Tag-Hilfsprogrammen](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro) in einer Razor-Ansicht veranschaulicht.

```HTML
...
<link rel="stylesheet" href="https://[your-cdn-endpoint].azureedge.net/lib/bootstrap/dist/css/bootstrap.min.css"
      asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
      asp-fallback-test-class="sr-only" asp-fallback-test-property="position" asp-fallback-test-value="absolute"/>
<link rel="stylesheet" href="~/css/site.min.css" asp-append-version="true"/>
...
<script src="https://[your-cdn-endpoint].azureedge.net/lib/jquery/dist/jquery-2.2.0.min.js"
        asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
        asp-fallback-test="window.jQuery">
</script>
<script src="https://[your-cdn-endpoint].azureedge.net/lib/bootstrap/dist/js/bootstrap.min.js"
        asp-fallback-src="~/lib/bootstrap/dist/js/bootstrap.min.js"
        asp-fallback-test="window.jQuery && window.jQuery.fn && window.jQuery.fn.modal">
</script>
...
```

### <a name="search-engine-optimization"></a>Suchmaschinenoptimierung
Wenn die Suchmaschinenoptimierung (SEO) bei Ihrer Anwendung ein wichtiger Aspekt ist, führen Sie die folgenden Aufgaben aus:

* Nehmen Sie in jede Seite oder Ressource einen kanonischen *Rel* -Header auf.
* Verwenden Sie einen *CNAME* -Unterdomäneneintrag, und greifen Sie auf die Ressourcen mit diesem Namen zu.
* Berücksichtigen Sie die Auswirkungen der Tatsache, dass die IP-Adresse des CDN möglicherweise ein Land oder eine Region darstellt, das bzw. die sich von der Anwendung unterscheidet.
* Wenn Sie Azure Blob Storage als Ursprung verwenden, behalten Sie die gleiche Dateistruktur für Ressourcen im CDN wie in den Anwendungsordnern bei.

### <a name="monitoring-and-logging"></a>Überwachung und Protokollierung
Nehmen Sie CDN in Ihre Anwendungsüberwachungsstrategie auf, um Ausfälle oder längere hohe Latenzzeiten zu erkennen und zu messen.  Für die Überwachung können Sie den CDN-Profil-Manager auf der Azure-Portalwebsite verwenden.

Aktivieren Sie die Protokollierung für das CDN, und überwachen Sie dieses Protokoll als Teil Ihrer täglichen Aufgaben.

Erwägen Sie, den CDN-Datenverkehr im Hinblick auf Verwendungsmuster zu analysieren. Das Azure-Portal bietet Tools, mithilfe derer Sie Folgendes überwachen können:

* Bandbreite,
* übertragene Daten,
* Treffer (Statuscodes),
* Cachestatus,
* Cachetrefferquote und
* das Verhältnis von IPV4/IPV6-Anforderungen.

Weitere Informationen finden Sie unter [Analysieren von Azure CDN-Verwendungsmustern](/azure/cdn/cdn-analyze-usage-patterns/).

### <a name="cost-implications"></a>Kostenauswirkungen
Sie bezahlen für ausgehende Datenübertragungen aus dem CDN.  Falls Sie Blobspeicher zum Hosten Ihrer Inhalte verwenden, zahlen Sie darüber hinaus für Speichertransaktionen, wenn das CDN Daten aus Ihrer Anwendung lädt. Legen Sie eine realistische Cachegültigkeitsdauer für Inhalte fest, um deren Aktualität sicherzustellen, aber keine so kurzen Zeiträume, dass Inhalte wiederholt aus der Anwendung oder aus Blob Storage in das CDN geladen werden müssen.

Auch Elemente, die nur selten heruntergeladen werden, erhöhen die Transaktionsgebühren, ohne dass die Serverauslastung deutlich reduziert wird.

### <a name="bundling-and-minification"></a>Bündelung und Minimierung
Verwenden Sie die Bündelung und Minimierung, um die Größe von Ressourcen zu verringern, die im CDN gespeichert werden, wie z. B. JavaScript-Code und HTML-Seiten. Mit dieser Strategie können Sie die Zeit verkürzen, die zum Herunterladen dieser Elemente auf den Client benötigt wird.

Bündelung und Minimierung können von ASP.NET verarbeitet werden. In einem MVC-Projekt definieren Sie Ihre Pakete in *BundleConfig.cs*. Ein Verweis auf das minimierte Skriptbündel wird durch den Aufruf der *Script.Render* -Methode, normalerweise im Code in der Ansichtsklasse, erstellt. Diese Referenz enthält eine Abfragezeichenfolge mit einem Hash, der auf dem Inhalt des Bündels basiert. Wenn sich der Bündelinhalt ändert, wird auch der generierte Hash geändert.  

Standardmäßig ist für Azure CDN-Instanzen die Einstellung für den *Status der Abfragezeichenfolge* deaktiviert. Damit aktualisierte Skriptbündel ordnungsgemäß vom CDN verarbeitet werden, müssen Sie die Einstellung für den *Status der Abfragezeichenfolge* für die CDN-Instanz aktivieren. Beachten Sie, dass es eine Stunde oder länger dauern kann, bevor die Einstellung wirksam wird.

### <a name="features"></a>Features

Azure umfasst verschiedene CDN-Produkte. Sehen Sie sich bei der Wahl eines CDN die Features an, die von den einzelnen Produkten unterstützt werden. Ausführliche Informationen finden Sie unter [Azure CDN-Features][cdn-features]. Verfügbare Premium-Features sind:

- **[Regelmodul](/azure/cdn/cdn-rules-engine)**: Mit der Regel-Engine können Sie anpassen, wie HTTP-Anforderungen behandelt werden, z.B. Sperren der Übermittlung bestimmter Inhaltstypen, Definieren einer Zwischenspeicherungsrichtlinie oder Ändern von HTTP-Headern. 

- **[Echtzeitstatistiken](/azure/cdn/cdn-real-time-stats)**: Sie können Echtzeitdaten überwachen, z.B. Bandbreite, Cachestatus und gleichzeitige Verbindungen mit Ihrem CDN-Profil, und [Echtzeitwarnungen](/azure/cdn/cdn-real-time-alerts) erhalten. 


## <a name="rules-engine-url-rewriting-example"></a>Beispiel für URL-Umschreibung mit der Regel-Engine

Im folgenden Diagramm wird veranschaulicht, wie Sie die [URL-Umschreibung](https://technet.microsoft.com/library/ee215194.aspx) bei Verwendung des CDN durchführen. Anforderungen vom CDN für zwischengespeicherte Inhalte werden basierend auf dem Ressourcentyp (z. B. Skripts und Bilder) an bestimmte Ordner innerhalb des Anwendungsstammverzeichnisses umgeleitet.  

![Regel-Engine-Diagramm](./images/cdn/rules-engine.png)

Durch diese Umschreiberegeln werden folgende Umleitungen ausgeführt:

* Die erste Regel ermöglicht Ihnen das Einbetten einer Version in den Dateinamen einer Ressource, die dann ignoriert wird. *Dateiname_v123.jpg* wird zum Beispiel in *Dateiname.jpg* umgeschrieben.
* Die nächsten vier Regeln zeigen das Umleiten von Anforderungen, wenn Sie die Ressourcen nicht in einem Ordner mit dem Namen *cdn** im Stammverzeichnis der Webrolle speichern möchten. Die Regeln ordnen die URLs *cdn/Images*, *cdn/Content*, *cdn/Scripts* und *cdn/bundles* ihren jeweiligen Stammordnern in der Webrolle zu.

Beachten Sie, dass Sie zum Verwenden der URL-Umschreibung einige Änderungen an der Bündelung von Ressourcen vornehmen müssen.     

## <a name="more-information"></a>Weitere Informationen
* [Azure CDN](https://azure.microsoft.com/services/cdn/)
* [Dokumentation zu Azure Content Delivery Network (CDN)](https://azure.microsoft.com/documentation/services/cdn/)
* [Verwenden von Azure CDN](/azure/cdn/cdn-create-new-endpoint/)
* [Integrieren eines Clouddiensts in Azure CDN](/azure/cdn/cdn-cloud-service-with-cdn/)(https://azure.microsoft.com/blog/2011/03/18/best-practices-for-the-windows-azure-content-delivery-network/)


<!-- links -->

[cdn-features]: /azure/cdn/cdn-overview#azure-cdn-features
