---
title: "Muster „Gatewayabladung“"
description: Es wird beschrieben, wie Sie freigegebene oder spezielle Dienstfunktionen an einen Gatewayproxy auslagern.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 6b3e4541aae77349ca91c18c788ddb508912361d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="gateway-offloading-pattern"></a>Muster „Gatewayabladung“

Es wird beschrieben, wie Sie freigegebene oder spezielle Dienstfunktionen an einen Gatewayproxy auslagern. Mit diesem Muster können Sie die Anwendungsentwicklung vereinfachen, indem Sie gemeinsam genutzte Dienstfunktionen, z.B. die Verwendung von SSL-Zertifikaten, aus anderen Teilen der Anwendung auf das Gateway verschieben.

## <a name="context-and-problem"></a>Kontext und Problem

Einige Features werden häufig über mehrere Dienste hinweg verwendet, sodass dafür die entsprechende Konfiguration, Verwaltung und Wartung erforderlich ist. Ein freigegebener oder spezialisierter Dienst, der mit jeder Anwendungsbereitstellung verteilt wird, erhöht den Verwaltungsaufwand und die Wahrscheinlichkeit für Bereitstellungsfehler. Alle Updates eines freigegebenen Features müssen für alle Dienste bereitgestellt werden, von denen dieses Feature gemeinsam genutzt wird.

Der richtige Umgang mit Sicherheitsproblemen (Tokenüberprüfung, Verschlüsselung, SSL-Zertifikatverwaltung) und andere komplexe Aufgaben können bedingen, dass Teammitglieder über sehr spezielle Fähigkeiten verfügen. Beispielsweise muss ein Zertifikat, das von einer Anwendung benötigt wird, auf allen Anwendungsinstanzen konfiguriert und bereitgestellt werden. Für jede neue Bereitstellung muss das Zertifikat verwaltet werden, um sicherzustellen, dass es nicht abläuft. Alle allgemeinen Zertifikate, deren Ablauf bevorsteht, müssen bei jeder Anwendungsbereitstellung aktualisiert, getestet und überprüft werden.

Andere gemeinsam genutzte Dienste, z.B. Authentifizierung, Autorisierung, Protokollierung, Überwachung oder [Drosselung](./throttling.md), können für eine große Zahl von Bereitstellungen schwierig zu implementieren und zu verwalten sein. Es kann besser sein, diese Art von Funktionen zu konsolidieren, um den Aufwand und die Fehlerwahrscheinlichkeit zu reduzieren.

## <a name="solution"></a>Lösung

Lagern Sie einige Features auf ein API-Gateway aus. Dies gilt vor allem für sich überschneidende Bereiche wie die Zertifikatverwaltung, Authentifizierung, SSL-Beendigung, Überwachung, Protokollübersetzung oder Drosselung. 

Im folgenden Diagramm ist ein API-Gateway dargestellt, mit dem eingehende SSL-Verbindungen beendet werden. Hiermit werden Daten im Namen des ursprünglichen Anforderers von einem beliebigen HTTP-Server abgerufen, der dem API-Gateway vorgeschaltet ist.

 ![](./_images/gateway-offload.png)
 
Vorteile dieses Musters:

- Die Entwicklung von Diensten wird vereinfacht, indem das Verteilen und Warten von unterstützenden Ressourcen entfällt, z.B. Webserverzertifikate und die Konfiguration für sichere Websites. Eine einfachere Konfiguration führt zu einer Vereinfachung der Verwaltung und Skalierbarkeit und erleichtert die Durchführung von Dienstupgrades.

- Dedizierte Teams können Features implementieren, für die besondere Kenntnisse erforderlich sind, z.B. im Bereich der Sicherheit. So kann sich Ihr Kernteam auf die Anwendungsfunktionen konzentrieren und diese speziellen Probleme, die trotzdem mehrere Bereiche betreffen, den jeweiligen Experten überlassen.

- Es kann Einheitlichkeit in Bezug auf die Protokollierung und Überwachung von Anforderungen und Antworten erzielt werden. Auch wenn ein Dienst nicht richtig instrumentiert wurde, kann das Gateway so konfiguriert werden, dass ein Mindestmaß an Überwachung und Protokollierung sichergestellt ist.

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

- Stellen Sie sicher, dass das API-Gateway hoch verfügbar und robust gegenüber Fehlern ist. Vermeiden Sie Single Points of Failure, indem Sie mehrere Instanzen Ihres API-Gateways ausführen. 
- Stellen Sie sicher, dass das Gateway für die Kapazitäts- und Skalierungsanforderungen Ihrer Anwendung und Endpunkte ausgelegt ist. Sorgen Sie dafür, dass das Gateway nicht zu einem Engpass für die Anwendung wird und in ausreichendem Maße skaliert werden kann.
- Lagern Sie nur Features aus, die von der gesamten Anwendung genutzt werden, z.B. Sicherheit oder Datenübertragung.
- Geschäftslogik sollte niemals auf das API-Gateway ausgelagert werden. 
- Wenn Sie Transaktionen nachverfolgen müssen, sollten Sie erwägen, Korrelations-IDs für Protokollierungszwecke zu generieren.

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

Verwenden Sie dieses Muster in folgenden Fällen:

- Für eine Anwendungsbereitstellung besteht eine übergreifende Zuständigkeit, z.B. in Bezug auf SSL-Zertifikate oder die Verschlüsselung.
- Ein Feature, dass über Anwendungsbereitstellungen (ggf. mit unterschiedlichen Ressourcenanforderungen) hinweg gemeinsam genutzt wird, z.B. Arbeitsspeicherressourcen, Speicherkapazität oder Netzwerkverbindungen.
- Sie möchten die Zuständigkeit für Probleme, z.B. Netzwerksicherheit, Drosselung oder andere Fragen zu Netzwerkgrenzen, an ein Team mit besserer Spezialisierung vergeben.

Dieses Muster ist unter Umständen nicht geeignet, wenn es dabei zu einer Kopplung zwischen Diensten kommt.

## <a name="example"></a>Beispiel

Indem Nginx als Appliance für die SSL-Auslagerung verwendet wird, wird mit der folgenden Konfiguration eine eingehende SSL-Verbindung beendet und die Verbindung auf einen von drei vorgeschalteten HTTP-Servern verteilt.

```
upstream iis {
        server  10.3.0.10    max_fails=3    fail_timeout=15s;
        server  10.3.0.20    max_fails=3    fail_timeout=15s;
        server  10.3.0.30    max_fails=3    fail_timeout=15s;
}

server {
        listen 443;
        ssl on;
        ssl_certificate /etc/nginx/ssl/domain.cer;
        ssl_certificate_key /etc/nginx/ssl/domain.key;

        location / {
                set $targ iis;
                proxy_pass http://$targ;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto https;
proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Host $host;
        }
}
```

## <a name="related-guidance"></a>Verwandte Anweisungen

- [Muster „Back-Ends für Front-Ends“](./backends-for-frontends.md)
- [Muster „Gatewayaggregation“](./gateway-aggregation.md)
- [Muster „Gatewayrouting“](./gateway-routing.md)

