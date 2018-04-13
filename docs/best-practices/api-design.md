---
title: Leitfaden zum API-Design
description: Leitfaden zum Erstellen einer gut konzipierten Web-API
author: dragon119
ms.date: 01/12/2018
pnp.series.title: Best Practices
ms.openlocfilehash: a8c4a81835ebd3ebdba2fd2cec624a9a9d5646f5
ms.sourcegitcommit: ea7108f71dab09175ff69322874d1bcba800a37a
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/17/2018
---
# <a name="api-design"></a>API-Design

Die meisten modernen Webanwendungen machen APIs verfügbar, die Clients für die Interaktion mit der Anwendung nutzen können. Eine gut entworfene Web-API sollte Folgendes unterstützen:

* **Unabhängigkeit von der Plattform**. Jeder Client muss die API unabhängig von ihrer internen Implementierung aufrufen können. Dies erfordert Standardprotokolle und einen Mechanismus, der es dem Client und dem Webdienst ermöglicht, sich bezüglich des Formats der auszutauschenden Daten zu verständigen.

* **Serviceentwicklung**. Die Web-API sollte weiterentwickelt werden können, und Funktionen sollten sich unabhängig von Clientanwendungen hinzufügen lassen. Während der Weiterentwicklung der API müssen vorhandene Clientanwendungen weiterhin ohne Änderungen funktionieren. Alle Funktionen müssen auffindbar sein, damit Clientanwendungen sie vollständig nutzen können.

Dieser Leitfaden enthält Informationen zu den Aspekten, die beim Entwerfen einer Web-API zu berücksichtigen sind.

## <a name="introduction-to-rest"></a>Einführung in REST

Im Jahr 2000 schlug Roy Fielding REST (Representational State Transfer) als Architekturansatz zum Entwerfen von Webdiensten vor. REST ist ein Architekturstil zum Erstellen verteilter Systeme, die auf Hypermedia basieren. REST ist unabhängig von zugrundeliegenden Protokollen und nicht unbedingt an HTTP gebunden. Die meisten gängigen REST-Implementierungen verwenden jedoch HTTP als Anwendungsprotokoll, und dieses Handbuch behandelt insbesondere den Entwurf von REST-APIs für HTTP.

Ein wichtiger Vorteil von REST gegenüber HTTP besteht darin, dass es offene Standards nutzt und die Implementierung der API oder der Clientanwendungen nicht an eine bestimmte Implementierung bindet. Beispielsweise könnte ein REST-Webdienst in ASP.NET geschrieben werden, und Clientanwendungen können alle Sprachen und Toolsets verwenden, die HTTP-Anforderungen generieren und HTTP-Antworten analysieren können.

Hier finden Sie einige der wichtigsten Entwurfsprinzipien von RESTful-APIs mit HTTP:

- REST-APIs wurden mit Fokus auf *Ressourcen* entwickelt, bei denen es sich um eine beliebige Art von Objekten, Daten oder Diensten handelt, auf die der Client zugreifen kann. 

- Eine Ressource weist einen *Bezeichner* auf. Dies ist ein URI, der die Ressource eindeutig identifiziert. Der URI für eine bestimmte Kundenbestellung kann z.B. wie folgt aussehen: 
 
    ```http
    http://adventure-works.com/orders/1
    ```
 
- Clients interagieren mit einem Dienst durch den Austausch von *Darstellungen* von Ressourcen. Viele Web-APIs verwenden JSON als Austauschformat. Beispielsweise könnte eine GET-Anforderung an den oben aufgeführten URI diesen Antworttext zurückgeben:

    ```json
    {"orderId":1,"orderValue":99.90,"productId":1,"quantity":1}
    ```

- REST-APIs verwenden eine einheitliche Schnittstelle, was zur Entkopplung der Client- und der Dienstimplementierungen beiträgt. Für REST-APIs, die auf HTTP basieren, umfasst die einheitliche Schnittstelle die Verwendung von HTTP-Standardverben zum Ausführen von Vorgängen für Ressourcen. Die häufigsten Vorgänge sind GET, POST, PUT, PATCH und DELETE. 

- REST-APIs verwenden ein zustandsloses Anforderungsmodell. HTTP-Anforderungen müssen unabhängig sein und können in beliebiger Reihenfolge erfolgen, sodass das Beibehalten von Übergangsstatusinformationen zwischen Anforderungen nicht möglich ist. Informationen werden ausschließlich in den Ressourcen selbst gespeichert, und jede Anforderung sollte eine atomare Operation sein. Diese Einschränkung ermöglicht eine hohe Skalierbarkeit von Webdiensten, da keine Affinität zwischen Clients und bestimmten Servern beibehalten werden muss. Jeder Server kann jede Anforderung von jedem beliebigen Client verarbeiten. Allerdings können andere Faktoren die Skalierbarkeit einschränken. Beispielsweise schreiben viele Webdienste in einen Back-End-Datenspeicher, dessen horizontale Hochskalierung schwierig sein kann. (Im Artikel [Datenpartitionierung](./data-partitioning.md) werden Strategien zur horizontalen Hochskalierung eines Datenspeichers beschrieben.)

- REST-APIs werden von Hypermedia Links gesteuert, die in der Darstellung enthalten sind. Das folgende Beispiel zeigt die JSON-Darstellung einer Bestellung. Es enthält Links zum Abrufen oder Aktualisieren des Kunden, der der Bestellung zugeordnet ist. 
 
    ```json
    {
        "orderID":3,
        "productID":2,
        "quantity":4,
        "orderValue":16.60,
        "links": [
            {"rel":"product","href":"http://adventure-works.com/customers/3", "action":"GET" },
            {"rel":"product","href":"http://adventure-works.com/customers/3", "action":"PUT" } 
        ]
    } 
    ```


Leonard Richardson hat 2008 das folgende [Reifemodell](https://martinfowler.com/articles/richardsonMaturityModel.html) für Web-APIs vorgeschlagen:

- Ebene 0: Definieren Sie einen URI, und alle Vorgänge sind POST-Anforderungen an diesen URI.
- Ebene 1: Erstellen Sie separate URIs für einzelne Ressourcen.
- Ebene 2: Verwenden Sie HTTP-Methoden zum Definieren von Vorgängen für Ressourcen.
- Ebene 3: Verwenden Sie Hypermedia (HATEOAS, unten beschriebenen).

Ebene 3 entspricht einer echten RESTful-API gemäß der Definition von Fielding. In der Praxis fallen viele veröffentlichte Web-APIs in den Bereich von Ebene 2.  

## <a name="organize-the-api-around-resources"></a>Organisieren der API unter Berücksichtigung von Ressourcen

Konzentrieren sich auf die Geschäftseinheiten, die die Web-API verfügbar macht. In einem E-Commerce-System können die primären Entitäten beispielsweise Kunden und Bestellungen sein. Eine Bestellung kann durch Senden einer HTTP POST-Anforderung erstellt werden, welche die Bestellinformationen enthält. Die HTTP-Antwort gibt an, ob die Bestellung erfolgreich aufgegeben wurde oder nicht. Sofern möglich sollten Ressourcen-URIs auf Nomen (Ressource) und nicht auf Verben (die Vorgänge für die Ressource) basieren. 

```HTTP
http://adventure-works.com/orders // Good

http://adventure-works.com/create-order // Avoid
```

Eine Ressource muss nicht auf einem einzelnen physischen Datenelement basieren. Beispielsweise kann eine Bestellressource intern als mehrere Tabellen in einer relationalen Datenbank implementiert werden, auf dem Client jedoch als einzelne Entität angezeigt werden. Vermeiden Sie das Erstellen von APIs, die einfach die interne Struktur einer Datenbank widerspiegeln. Der Zweck von REST ist das Modellieren von Entitäten und der Vorgänge, die eine Anwendung für diese Entitäten ausführen kann. Ein Client sollte nicht für die interne Implementierung verfügbar gemacht werden.

Entitäten sind häufig in Sammlungen (Bestellungen, Kunden) gruppiert. Eine Sammlung ist als Ressource vom Element innerhalb der Sammlung getrennt und muss über einen eigenen URI verfügen. Der folgende URI könnte z.B. die Sammlung von Bestellungen darstellen: 

```HTTP
http://adventure-works.com/orders
```

Durch Senden einer HTTP GET-Anforderung an den URI der Sammlung wird eine Liste von Elementen in der Sammlung abgerufen. Jedes Element in der Sammlung verfügt auch über einen eigenen eindeutigen URI. Eine HTTP GET-Anforderung an den URI eines Elements gibt die Details dieses Elements zurück. 

Führen Sie eine konsistente Benennungskonvention für URIs ein. Im Allgemeinen ist es hilfreich, Nomen in Pluralform für URIs zu verwenden, die auf Sammlungen verweisen. Es wird empfohlen, URIs für Sammlungen und Elemente in einer Hierarchie zu organisieren. Beispiel: `/customers` ist der Pfad zur Sammlung „Kunden“, und `/customers/5` ist der Pfad zum Kunden mit der ID 5. Dieser Ansatz trägt dazu bei, dass die Web-API intuitiv bleibt. Darüber hinaus können viele Web-API-Frameworks Anforderungen auf Grundlage von parametrisierte URI-Pfaden weiterleiten, sodass Sie eine Route für den Pfad `/customers/{id}` definieren könnten.

Berücksichtigen Sie auch die Beziehungen zwischen verschiedenen Ressourcentypen und wie Sie diese Zuordnungen verfügbar machen können. Beispielsweise kann `/customers/5/orders` alle Bestellungen für Kunde 5 darstellen. Sie könnten auch in die andere Richtung gehen und die Zuordnung einer Bestellung an einen Kunden mit einem URI umgekehrt darstellen, z.B. `/orders/99/customer`. Eine zu umfangreiche Erweiterung dieses Modells kann jedoch aufwendig zu implementieren sein. Eine bessere Lösung ist das Bereitstellen navigierbarer Links zu zugeordneten Ressourcen im Text der HTTP-Antwortnachricht. Dieser Mechanismus wird im Abschnitt [Verwenden des HATEOAS-Ansatzes, um die Navigation zu verwandten Ressourcen zu ermöglichen](#using-the-hateoas-approach-to-enable-navigation-to-related-resources) weiter unten ausführlicher beschrieben.

In komplexeren Systemen kann es attraktiv erscheinen, URIs bereitzustellen, die einem Client das Navigieren über mehrere Beziehungsebenen hinweg ermöglichen, z.B. `/customers/1/orders/99/products`. Dieses Maß an Komplexität kann jedoch schwierig zu verwalten sein und ist unflexibel, wenn sich die Beziehungen zwischen Ressourcen in der Zukunft ändern. Versuchen Sie stattdessen, URIs relativ einfach zu halten. Sobald eine Anwendung über einen Verweis auf eine Ressource verfügt, kann dieser Verweis zum Suchen von Elementen im Zusammenhang mit dieser Ressource verwendet werden. Die vorhergehende Abfrage kann durch den URI `/customers/1/orders` ersetzt werden, um alle Bestellungen für Kunde 1 zu finden, und dann durch `/orders/99/products` ersetzt werden, um die Produkte in einer Bestellung zu finden.

> [!TIP]
> Vermeiden Sie, dass komplexere Ressourcen-URIs als *Auflistung/Element/Auflistung* erforderlich sind.

Dazu kommt, dass alle Webanforderungen den Webserver belasten. Je mehr Anforderungen, desto größer die Last. Versuchen Sie daher, „geschwätzige“ Web-APIs vermeiden, die eine große Anzahl von kleinen Ressourcen verfügbar machen. Eine solche API erfordert möglicherweise, dass eine Clientanwendung mehrere Anforderungen sendet, um alle erforderlichen Daten zu finden. Stattdessen sollten Sie die Daten denormalisieren und zugehörige Informationen zu größeren Ressourcen zusammenzufassen, die mit einer einzelnen Anforderung abgerufen werden können. Diesen Ansatz müssen Sie jedoch gegen den Aufwand für das Abrufen von Daten abwägen, die für den Client nicht erforderlich sind. Das Abrufen von großen Objekten kann die Latenz einer Anforderung erhöhen und zusätzliche Bandbreitenkosten verursachen. Weitere Informationen zu diesen leistungsbezogenen Antimustern finden Sie unter [Zu viele E/A-Vorgänge](../antipatterns/chatty-io/index.md) und [Irrelevante Abrufe](../antipatterns/extraneous-fetching/index.md).

Vermeiden Sie das Einführen von Abhängigkeiten zwischen der Web-API und den zugrunde liegenden Datenquellen. Sind Ihre Daten beispielsweise in einer relationalen Datenbank gespeichert, muss die Web-API nicht jede Tabelle als Sammlung von Ressourcen verfügbar machen. Tatsächlich ist dies wahrscheinlich ein schlechter Entwurf. Stellen Sie sich die Web-API stattdessen als eine Abstraktion der Datenbank vor. Führen Sie bei Bedarf eine Zuordnungsebene zwischen der Datenbank und der Web-API ein. Auf diese Weise sind Clientanwendungen von Änderungen am zugrunde liegenden Datenbankschema isoliert.

Schließlich ist es auch möglich, dass nicht jeder von einer Web-API für eine bestimmte Ressource implementierte Vorgang zugeordnet werden kann. Sie können solche Szenarien *ohne Ressourcen* z.B. über HTTP-Anforderungen behandeln, die eine Funktion aufrufen und die Ergebnisse als HTTP-Antwortnachricht zurückgeben. Beispiel: Eine Web-API, die einfache Rechenvorgänge wie Addition und Subtraktion implementiert, könnte URIs bereitstellen, die diese Vorgänge als Pseudoressourcen verfügbar machen und die Abfragezeichenfolge zum Angeben der erforderlichen Parameter verwenden. Eine GET-Anforderung an den URI */add?operand1=99&operand2=1* würde z.B. eine Antwortnachricht mit dem Wert 100 im Text zurückgeben. Verwenden Sie diese URI-Formen jedoch nur sparsam.

## <a name="define-operations-in-terms-of-http-methods"></a>Definieren von Vorgängen im Hinblick auf HTTP-Methoden

Das HTTP-Protokoll definiert eine Reihe von Methoden, die einer Anforderung semantische Bedeutung zuweisen. Diese allgemeinen HTTP-Methoden werden von den meisten RESTful-Web-APIs verwendet:

* **GET** ruft eine Kopie der Ressource am angegebenen URI ab. Der Text der Antwortnachricht enthält die Details der angeforderten Ressource.
* **POST** erstellt eine neue Ressource am angegebenen URI. Der Text der Anforderungsnachricht enthält die Details der neuen Ressource. Beachten Sie, dass POST auch zum Auslösen von Vorgängen verwendet werden kann, die nicht tatsächlich Ressourcen erstellen.
* **PUT** erstellt oder ersetzt die Ressource am angegebenen URI. Der Text der Anforderungsnachricht gibt die zu erstellende oder zu aktualisierende Ressource an.
* **PATCH** führt eine partielle Aktualisierung einer Ressource durch. Der Anforderungstext gibt die Menge der auf die Ressource anzuwendenden Änderungen an.
* **DELETE** entfernt die Ressource am angegebenen URI.

Die Auswirkungen einer bestimmten Anforderung sollten davon abhängen, ob die Ressource, eine Sammlung oder ein einzelnes Element ist. In der folgende Tabelle sind anhand des e-Commerce-Beispiels die allgemeinen Konventionen zusammengefasst, die bei den meisten RESTful-Implementierungen verwendet werden. Beachten Sie, dass u. U. nicht alle diese Anforderungen implementiert werden; dies hängt vom jeweiligen Szenario ab.

| **Ressource** | **POST** | **GET** | **PUT** | **DELETE** |
| --- | --- | --- | --- | --- |
| /Kunden |Neuen Kunden erstellen |Alle Kunden abrufen |Massenaktualisierung aller Kunden |Alle Kunden entfernen |
| /Kunden/1 |Error |Details für Kunden 1 abrufen |Details von Kunde 1 aktualisieren, falls vorhanden |Kunde 1 entfernen |
| /Kunden/1/Bestellungen |Neue Bestellung für Kunden 1 erstellen |Alle Bestellungen für Kunde 1 abrufen |Massenaktualisierung von Bestellungen für Kunde 1 |Alle Bestellungen für Kunde 1 entfernen |

Die Unterschiede zwischen POST, PUT und PATCH können verwirrend sein.

- Eine POST-Anforderung erstellt eine Ressource. Der Server weist einen URI für die neue Ressource zu und gibt diesen URI an den Client zurück. Im REST-Modell wenden Sie häufig POST-Anforderungen auf Sammlungen an. Die neue Ressource wird der Sammlung hinzugefügt. Eine POST-Anforderung kann auch verwendet werden, um Daten zur Verarbeitung an eine vorhandene Ressource zu senden, ohne dass eine neue Ressource erstellt wird.

- Eine PUT-Anforderung erstellt eine Ressource *oder* aktualisiert eine vorhandene Ressource. Der Client gibt den URI für die Ressource an. Der Anforderungstext enthält eine vollständige Darstellung der Ressource. Wenn bereits eine Ressource mit diesem URI vorhanden ist, wird sie ersetzt. Andernfalls wird eine neue Ressource erstellt, wenn der Server diese Vorgehensweise unterstützt. PUT-Anforderungen werden am häufigsten auf Ressourcen angewendet, bei denen es sich um einzelne Elemente handelt, z.B. einen bestimmten Kunden anstatt von Sammlungen. Ein Server kann Aktualisierungen über PUT unterstützen, die Erstellung jedoch möglicherweise nicht. Die Unterstützung der Erstellung über PUT hängt davon ab, ob der Client einen URI auf sinnvolle Weise einer Ressource zuweisen kann, bevor diese vorhanden ist. Ist dies nicht der Fall, verwenden Sie POST zum Erstellen und PUT oder PATCH zum Aktualisieren von Ressourcen.

- Eine PATCH-Anforderung führt eine *partielle Aktualisierung* einer vorhandenen Ressource durch. Der Client gibt den URI für die Ressource an. Der Anforderungstext gibt einen Satz von auf die Ressource anzuwendenden *Änderungen* an. Dies kann effizienter als die Verwendung von PUT sein, da der Client nur die Änderungen sendet, nicht die gesamte Darstellung der Ressource. Technisch gesehen kann PATCH kann auch eine neue Ressource erstellen (durch Angabe einer Reihe von Aktualisierungen für eine NULL-Ressource), wenn der Server dies unterstützt. 

PUT-Anforderungen müssen idempotent sein. Wenn ein Client die gleiche PUT-Anforderung mehrmals sendet, sollten die Ergebnisse immer identisch sein (dieselbe Ressource wird mit den gleichen Werten geändert). Bei POST- und PATCH-Anforderungen besteht keine Garantie für Idempotenz.

## <a name="conform-to-http-semantics"></a>Einhaltung der HTTP-Semantik

Dieser Abschnitt beschreibt einige typische Überlegungen zum Entwerfen einer API, die der HTTP-Spezifikation entspricht. Es wird jedoch nicht jedes mögliche Detail oder Szenario abgedeckt. Lesen Sie im Zweifelsfall die Informationen zu den HTTP-Spezifikationen.

### <a name="media-types"></a>Medientypen

Wie bereits erwähnt tauschen Clients und Server Darstellungen von Ressourcen aus. In einer POST-Anforderung z.B. enthält der Anforderungstext eine Darstellung der Ressource, die zu erstellen ist. In einer GET-Anforderung enthält der Antworttext eine Darstellung der abgerufenen Ressource.

Im HTTP-Protokoll werden Formate durch die Verwendung von *Medientypen* angegeben, die auch als „MIME-Typen“ bezeichnet werden. Für nicht binäre Daten unterstützen die meisten Web-APIs JSON (media type = application/json) und möglicherweise XML (media type = application/xml). 

Der Content-Type-Header in einer Anforderung oder Antwort gibt das Format der Darstellung an. Hier ist ein Beispiel für eine POST-Anforderung, die JSON-Daten enthält:

```HTTP
POST http://adventure-works.com/orders HTTP/1.1
Content-Type: application/json; charset=utf-8
Content-Length: 57

{"Id":1,"Name":"Gizmo","Category":"Widgets","Price":1.99}
```

Wenn der Server den Medientyp nicht unterstützt, muss er den HTTP-Statuscode 415 (Nicht unterstützter Medientyp) zurückgeben.

Eine Clientanforderung kann einen Accept-Header beinhalten, der eine Liste von Medientypen enthält, die der Client in der Antwortnachricht vom Server akzeptiert. Beispiel: 

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
Accept: application/json
```

Wenn der Server keinen der Medientypen auf der Liste anbieten kann, muss er den HTTP-Statuscode 406 (Nicht akzeptabel) zurückgeben. 

### <a name="get-methods"></a>GET-Methoden

Eine erfolgreiche GET-Methode gibt in der Regel den HTTP-Statuscode 200 (OK) zurück. Wenn die Ressource nicht gefunden werden kann, sollte die Methode 404 (Nicht gefunden) zurückgeben.

### <a name="post-methods"></a>POST-Methoden

Wenn eine POST-Methode eine neue Ressource erstellt, wird der HTTP-Statuscode 201 (Erstellt) zurückgegeben. Der URI der neuen Ressource ist im Location-Header der Antwort enthalten. Der Antworttext enthält eine Darstellung der Ressource.

Wenn die Methode etwas verarbeitet, jedoch keine neue Ressource erstellt, kann sie den HTTP-Statuscode 200 zurückgegeben und das Ergebnis des Vorgangs in den Antworttext aufnehmen. Als Alternative kann die Methode auch den HTTP-Statuscode 204 (Kein Inhalt) ohne Antworttext zurückgeben, wenn kein Ergebnis zum Zurückgeben vorhanden ist.

Wenn der Client ungültige Daten in die Anforderung einfügt, muss der Server den HTTP-Statuscode 400 (Ungültige Anforderung) zurückgeben. Der Antworttext kann zusätzliche Informationen über den Fehler oder einen Link zu einem URI enthalten, unter dem weitere Details verfügbar sind.

### <a name="put-methods"></a>PUT-Methoden

Wenn eine PUT-Methode eine neue Ressource erstellt, wird wie bei einer POST-Methode der HTTP-Statuscode 201 (Erstellt) zurückgegeben. Wenn die Methode eine vorhandene Ressource aktualisiert, wird entweder 200 (OK) oder 204 (Kein Inhalt) zurückgegeben. In einigen Fällen ist es u.U. nicht möglich, eine vorhandene Ressource zu aktualisieren. In diesem Fall empfiehlt es sich, den HTTP-Statuscode 409 (Konflikt) zurückzugeben. 

Erwägen Sie das Implementieren von HTTP PUT-Massenvorgängen, die Stapelaktualisierungen für mehrere Ressourcen in einer Auflistung durchführen können. Die PUT-Anforderung sollte den URI der Auflistung angeben, und Anforderungstext sollte die Details der zu ändernden Ressourcen angeben. Dieser Ansatz trägt zum Reduzieren von „Geschwätzigkeit“ und zur Leistungssteigerung bei.

### <a name="patch-methods"></a>PATCH-Methoden

Mit einer PATCH-Anforderung sendet der Client einen Satz von Aktualisierungen in Form eines *Patch-Dokuments* an eine vorhandene Ressource. Der Server verarbeitet das Patch-Dokument, um die Aktualisierung durchzuführen. Das Patch-Dokument beschreibt nicht die gesamte Ressource, nur einen Satz von anzuwendenden Änderungen. In der Spezifikation für die PATCH-Methode ([RFC 5789](https://tools.ietf.org/html/rfc5789)) ist kein bestimmtes Format für Patch-Dokumente festgelegt. Das Format muss vom Medientyp in der Anforderung abgeleitet werden.

JSON ist wahrscheinlich das gängigste Datenformat für Web-APIs. Es gibt zwei JSON-basierte Patch-Hauptformate, die *JSON Patch* und *JSON Merge Patch* heißen.

JSON Merge Patch ist etwas einfacher. Das Patch-Dokument hat die gleiche Struktur wie die ursprüngliche JSON-Ressource, enthält jedoch nur die Teilmenge von Feldern, die geändert oder hinzugefügt werden soll. Außerdem kann ein Feld gelöscht werden, indem im Patch-Dokument `null` als Feldwert angegeben wird. (Merge Patch ist daher nicht geeignet, wenn die ursprüngliche Ressource explizite NULL-Werte enthalten kann.)

Beispiel: Angenommen, die ursprüngliche Ressource weist die folgende JSON-Darstellung auf:

```json
{ 
    "name":"gizmo",
    "category":"widgets",
    "color":"blue",
    "price":10
}
```

Hier ist ein möglicher JSON Merge Patch für diese Ressource:

```json
{ 
    "price":12,
    "color":null,
    "size":"small"
}
```

Dieser weist den Server an, „price“ (Preis) zu aktualisieren, „color“ (Farbe) zu löschen und „size“ (Größe) hinzuzufügen. „Name“ und „category“ (Kategorie) werden nicht geändert. Die genauen Details zu JSON Merge Patch finden Sie unter [RFC 7396](https://tools.ietf.org/html/rfc7396). Der Medientyp für JSON Merge Patch ist „application/merge-patch+json“.

Merge Patch ist nicht geeignet, wenn die ursprüngliche Ressource aufgrund der besonderen Bedeutung von `null` im Patch-Dokument explizite NULL-Werte enthalten kann. Außerdem gibt das Patch-Dokument nicht die Reihenfolge an, in welcher der Server die Aktualisierungen durchführen soll. Dies kann je nach Daten und Domäne von Bedeutung sein oder nicht. JSON Patch, in [RFC 6902](https://tools.ietf.org/html/rfc6902) definiert, ist flexibler. Es gibt die Änderungen als eine Sequenz von anzuwendenden Vorgängen an. Zu Vorgängen gehören das Hinzufügen, Entfernen, Ersetzen, Kopieren und Testen (zum Überprüfen von Werten). Der Medientyp für JSON Patch ist „application/json-patch+json“.

Hier sehen Sie einige typische Fehlerzustände, die beim Verarbeiten einer PATCH-Anforderung auftreten können, zusammen mit den entsprechenden HTTP-Statuscodes.

| Fehlerzustand | HTTP-Statuscode |
|-----------|------------|
| Das Format des Patch-Dokuments wird nicht unterstützt. | 415 (Nicht unterstützter Medientyp) |
| Falsch formatiertes Patch-Dokument. | 400 (Ungültige Anforderung) |
| Das Patch-Dokument ist gültig, aber die Änderungen können auf die Ressource in ihrem aktuellen Zustand nicht angewendet werden. | 409 (Konflikt)

### <a name="delete-methods"></a>DELETE-Methoden

Wenn der Löschvorgang erfolgreich ist, sollte der Webserver mit dem HTTP-Statuscode 204 reagieren, der angibt, dass der Prozess erfolgreich verarbeitet wurde, der Antworttext jedoch keine weiteren Informationen enthält. Wenn die Ressource nicht vorhanden ist, kann der Webserver „HTTP 404“ (Nicht gefunden) zurückgeben.

### <a name="asynchronous-operations"></a>Asynchrone Vorgänge

In einigen Fällen kann ein POST-, PUT-, PATCH- oder DELETE-Vorgang eine Verarbeitung erfordern, die einige Zeit in Anspruch nimmt. Wenn Sie vor dem Senden einer Antwort an den Client den Abschluss abwarten, kann dies zu einer nicht akzeptablen Wartezeit führen. Wenn dies der Fall ist, erwägen Sie die Verwendung eines asynchronen Vorgangs. Geben Sie den HTTP-Statuscode 202 (Akzeptiert) zurück, um anzugeben, dass die Anforderung zur Verarbeitung angenommen wurde, aber noch nicht abgeschlossen ist. 

Sie müssen einen Endpunkt verfügbar machen, der den Status einer asynchronen Anforderung zurückgibt, damit der Client den Status durch Abfragen des Statusendpunkts überwachen kann. Nehmen Sie den URI des Statusendpunkts in den Location-Header der 202-Antwort auf. Beispiel: 

```http
HTTP/1.1 202 Accepted
Location: /api/status/12345
```

Wenn der Client eine GET-Anforderung an diesen Endpunkt sendet, muss die Antwort auf den aktuellen Status der Anforderung enthalten. Optional kann sie auch die geschätzte Zeit bis zum Abschluss oder einen Link zum Abbrechen des Vorgangs enthalten. 

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "status":"In progress",
    "link": { "rel":"cancel", "method":"delete", "href":"/api/status/12345"
}
```

Wenn der asynchrone Vorgang eine neue Ressource erstellt, muss der Statusendpunkt nach Abschluss des Vorgangs den Statuscode 303 (See Other [Siehe anderswo]) zurückgeben. Nehmen Sie in die 303-Antwort einen Location-Header auf, der den URI der neuen Ressource angibt:

```http
HTTP/1.1 303 See Other
Location: /api/orders/12345
```

Weitere Informationen finden Sie unter [Asynchronous operations in REST](https://www.adayinthelifeof.nl/2011/06/02/asynchronous-operations-in-rest/) (Asynchrone Vorgänge in REST).

## <a name="filter-and-paginate-data"></a>Filtern und Paginieren von Daten

Das Verfügbarmachen einer Sammlung von Ressourcen durch einen einzigen URI kann dazu führen, dass Anwendungen große Datenmengen abrufen, auch wenn nur eine Teilmenge der Informationen erforderlich ist. Beispiel: Angenommen, eine Clientanwendung muss alle Bestellungen mit Kosten über einem bestimmten Wert finden. Dazu könnte sie alle Bestellungen vom URI */orders* abrufen und diese Bestellungen auf Clientseite filtern. Dieser Prozess ist eindeutig sehr ineffizient. Er verschwendet Netzwerkbandbreite und Verarbeitungsleistung auf dem Server, der die Web-API hostet.

Stattdessen kann die API zulassen, dass ein in der Abfragezeichenfolge des URIs ein Filter wie */orders?minCost=n* übergeben wird. Dann ist die Web-API für die Analyse und Verarbeitung der `minCost`-Parameter in der Abfragezeichenfolge und das Zurückgeben der gefilterten Ergebnisse auf Serverseite verantwortlich. 

GET-Anforderungen über Sammlungsressourcen geben u.U. eine große Anzahl von Elementen zurück. Entwerfen Sie daher eine Web-API so, dass die von jeder einzelnen Anforderung zurückgegebene Datenmenge begrenzt ist. Erwägen Sie die Unterstützung von Abfragezeichenfolgen, welche die maximale Anzahl von abzurufenden Elementen angeben, und ein Startoffset in der Sammlung. Beispiel: 

```
/orders?limit=25&offset=50
```

Erwägen Sie auch das Festlegen einer Obergrenze für die Anzahl der zurückgegebenen Elemente, um Denial-of-Service-Angriffe zu verhindern. Zur Unterstützung von Clientanwendungen sollten GET-Anforderungen, die paginierte Daten zurückgeben, auch Metadaten enthalten, die die Gesamtanzahl der in der Auflistung verfügbaren Ressourcen angeben. Ziehen Sie auch andere intelligente Pagingstrategien in Betracht. Weitere Informationen hierzu finden Sie unter [API-Designhinweise: Smart Paging](http://bizcoder.com/api-design-notes-smart-paging).

Sie können eine ähnliche Strategie zum Sortieren von Daten während des Abrufvorgangs nutzen. Dazu stellen Sie einen Sortierparameter bereit, der einen Feldnamen als Wert verwendet, z.B. */orders?sort=ProductID*. Dieser Ansatz kann jedoch die Zwischenspeicherung beeinträchtigen, da Abfragezeichenfolgen-Parameter einen Teil des Ressourcenbezeichners darstellen, der von zahlreichen Cacheimplementierungen als Schlüssel für zwischengespeicherte Daten verwendet wird.

Sie können diesen Ansatz erweitern, um die zurückgegebenen Felder für die einzelnen Elemente zu begrenzen, wenn jedes Element eine große Datenmenge enthält. Sie können beispielsweise einen Abfragezeichenfolgen-Parameter verwenden, der eine durch Trennzeichen getrennte Liste der Felder akzeptiert, z.B. */Bestellungen?Felder=ProductID,Menge*. 

Geben Sie für alle optionalen Parameter in Abfragezeichenfolgen aussagekräftige Standardwerte an. Legen Sie z. B. den `limit`-Parameter auf 10 und den `offset`-Parameter auf 0 fest, wenn Sie die Paginierung implementieren; legen Sie den Sortierungsparameter auf den Schlüssel der Ressource fest, wenn Sie Bestellungen implementieren, und legen Sie den `fields`-Parameter auf alle Felder in der Ressource fest, wenn Sie Projektionen unterstützen.

## <a name="support-partial-responses-for-large-binary-resources"></a>Unterstützung von partielle Antworten für große binäre Ressourcen

Eine Ressource kann große binäre Felder wie z.B. Dateien oder Bilder enthalten. Zum Beheben von Problemen, die durch unzuverlässige und unterbrochene Verbindungen verursacht werden, und zur Verbesserung der Antwortzeiten können Sie zulassen, dass solche Ressourcen in Blöcken abgerufen werden. Dazu muss die Web-API den Accept-Ranges-Header für GET-Anforderungen für große Ressourcen unterstützen. Dieser Header gibt an, dass der GET-Vorgang partielle Anforderungen unterstützt. Die Clientanwendung kann GET-Anforderungen senden, die eine Teilmenge einer Ressource zurückgeben, die als Bytebereich angegeben ist. 

Erwägen Sie auch die Implementierung von HTTP HEAD-Anforderungen für diese Ressourcen. Eine HEAD-Anforderung ähnelt einer GET-Anforderung, gibt jedoch nur die HTTP-Header zurück, welche die Ressource beschreiben, sowie einen leeren Nachrichtentext. Eine Clientanwendung kann eine HEAD-Anforderung ausgeben, um zu bestimmen, ob eine Ressource mithilfe von partiellen GET-Anforderungen abgerufen wird. Beispiel: 

```HTTP
HEAD http://adventure-works.com/products/10?fields=productImage HTTP/1.1
```

Hier ist ein Beispiel für eine Antwortnachricht: 

```HTTP
HTTP/1.1 200 OK

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
```

Der Content-Length-Header gibt die Gesamtgröße der Ressource an, und der Accept-Ranges-Header gibt an, dass der entsprechende GET-Vorgang Teilergebnisse unterstützt. Die Clientanwendung kann diese Informationen verwenden, um das Bild in kleineren Blöcken abzurufen. Die erste Anforderung Ruft die ersten 2500 Bytes mithilfe des Range-Headers ab:

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
```

Die Antwortnachricht gibt an, dass dies eine partielle Antwort ist, in dem der Statuscode „HTTP 206“ zurückgegeben wird. Der Content-Length-Header gibt die tatsächliche Anzahl der im Nachrichtentext (nicht die Größe der Ressource) zurückgegebenen Bytes an, und der Content-Range-Header gibt an, um welchen Teil der Ressource es sich handelt (0 – 2499 von 4580 Bytes):

```HTTP
HTTP/1.1 206 Partial Content

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580

[...]
```

Eine nachfolgende Anforderung von der Clientanwendung kann den Rest der Ressource abrufen.

## <a name="use-hateoas-to-enable-navigation-to-related-resources"></a>Verwenden von HATEOAS, um die Navigation zu verwandten Ressourcen zu ermöglichen

Eine Hauptmotivation hinter REST besteht darin, dass der gesamte Ressourcensatz navigiert werden kann, ohne dass zuvor Kenntnisse des URI-Schemas erforderlich sind. Jede HTTP GET-Anforderung sollte über Hyperlinks in der Antwort die erforderlichen Informationen zum Finden der direkt auf das angeforderte Objekt bezogenen Ressourcen enthalten; außerdem sollten Informationen bereitgestellt werden, die die für jede dieser Ressourcen verfügbaren Vorgänge beschreiben. Dieses Prinzip wird als Hypertext als das Modul des Anwendungszustands (HATEOAS, Hypertext as the Engine of Application State) bezeichnet. Beim System handelt es sich im Grunde um einen finiten Statuscomputer, und die Antwort auf jede Anforderung enthält die notwendigen Informationen für den Wechseln von einem Zustand in einen anderen; weitere Informationen sollten nicht erforderlich sein.

> [!NOTE]
> Derzeit sind keine Standards oder Spezifikationen vorhanden, die die Modellierung des HATEOAS-Prinzips definieren. Die Beispiele in diesem Abschnitt veranschaulichen eine mögliche Lösung.
>
>

Beispielsweise kann die Darstellung einer Bestellung zur Verarbeitung der Beziehung zwischen der Bestellung und dem Kunden Links enthalten, welche die verfügbaren Vorgänge für den Kunden der Bestellung identifizieren. Hier ist eine mögliche Darstellung: 

```json
{
  "orderID":3,
  "productID":2,
  "quantity":4,
  "orderValue":16.60,
  "links":[
    {
      "rel":"customer",
      "href":"http://adventure-works.com/customers/3", 
      "action":"GET",
      "types":["text/xml","application/json"] 
    },
    {
      "rel":"customer",
      "href":"http://adventure-works.com/customers/3", 
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"customer",
      "href":"http://adventure-works.com/customers/3",
      "action":"DELETE",
      "types":[]
    },
    {
      "rel":"self",
      "href":"http://adventure-works.com/orders/3", 
      "action":"GET",
      "types":["text/xml","application/json"]
    },
    {
      "rel":"self",
      "href":"http://adventure-works.com/orders/3", 
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"self",
      "href":"http://adventure-works.com/orders/3", 
      "action":"DELETE",
      "types":[]
    }]
}
```

In diesem Beispiel weist das `links`-Array einen Satz von Links auf. Jeder Link stellt einen Vorgang für eine verknüpfte Entität dar. Die Daten für jeden Link beinhalten die Beziehung („Kunde“), den URI (`http://adventure-works.com/customers/3`), die HTTP-Methode und die unterstützten MIME-Typen. Dies sind alle Informationen, die eine Clientanwendung zum Aufrufen des Vorgangs benötigt. 

Das `links`-Array enthält auch auf sich selbst verweisende Informationen über die abgerufene Ressource selbst. Diese weisen die Beziehung *self* (selbst) auf.

Die Gruppe der zurückgegebenen Links kann sich je nach dem Zustand der Ressource ändern. Dies ist damit gemeint, dass Hypertext das Modul des Anwendungszustands (HATEOAS, Hypertext as the Engine of Application State) ist.

## <a name="versioning-a-restful-web-api"></a>Versionsverwaltung einer RESTful-Web-API

Es ist sehr unwahrscheinlich, dass eine Web-API statisch bleibt. Aufgrund von veränderten Unternehmensanforderungen werden u. U. neue Auflistungen von Ressourcen hinzugefügt, die Beziehungen zwischen Ressourcen können sich ändern, und die Struktur der Daten in Ressourcen wird möglicherweise geändert. Das Aktualisieren einer Web-API, um neue oder unterschiedliche Anforderungen zu behandeln, ist ein relativ unkomplizierter Prozess. Sie müssen jedoch die Auswirkungen solcher Änderungen auf Client-Anwendungen berücksichtigen, die die Web-API nutzen. Das Problem ist, dass der Entwickler, der eine Web-API entwirft und implementiert, zwar die vollständige Kontrolle über diese API hat, jedoch nicht das gleiche Maß an Kontrolle über Clientanwendungen hat, die u. U. von remote agierenden Drittanbietern erstellt werden. Es kommt primär darauf an, dass vorhandene Clientanwendungen unverändert weiter ausgeführt werden können, und dass neue Clientanwendungen neue Features und Ressourcen nutzen können.

Die Versionsverwaltung ermöglicht einer Web-API das Angeben der Funktionen und Ressourcen, die sie verfügbar macht, und eine Clientanwendung kann Anforderungen an eine bestimmte Version einer Funktion oder einer Ressource senden. In den folgenden Abschnitten sind verschiedene Ansätze mit ihren jeweiligen Vor- und Nachteilen beschrieben.

### <a name="no-versioning"></a>Keine Versionsverwaltung
Dies ist der einfachste Ansatz und ist möglicherweise für einige interne APIs akzeptabel. Umfassende Änderungen können als neue Ressourcen oder neue Links dargestellt werden.  Das Hinzufügen von Inhalt zu vorhandenen Ressourcen stellt möglicherweise keine fehlerhafte Änderung dar, da Clientanwendungen, die diesen Inhalt nicht erwarten, ihn einfach ignorieren.

Beispielsweise sollte eine Anforderung an den URI *http://adventure-works.com/customers/3* die Details eines einzelnen Kunden mit den von der Clientanwendung erwarteten Feldern `id`, `name` und `address` zurückgeben:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [!NOTE]
> Der Einfachheit halber enthalten die in diesem Abschnitt gezeigten Beispielantworten keine HATEOAS-Links.
>
>

Wenn dem Schema der Kundenressource das `DateCreated` Feld hinzugefügt wird, sieht die Antwort wie folgt aus:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

Vorhandene Clientanwendungen werden möglicherweise weiterhin ordnungsgemäß ausgeführt, wenn sie in der Lage sind, unbekannte Felder zu ignorieren. Neue Clientanwendungen können für die Behandlung dieses neuen Felds konfiguriert werden. Allerdings können radikalere Änderungen am Schema von Ressourcen (z. B. das Entfernen oder Umbenennen von Feldern) oder Änderungen an den Beziehungen zwischen Ressourcen fehlerhafte Änderungen darstellen, die verhindern, dass vorhandene Clientanwendungen nicht ordnungsgemäß funktioniert. In diesen Fällen sollten Sie einen der folgenden Ansätze in Betracht ziehen.

### <a name="uri-versioning"></a>URI-Versionsverwaltung
Bei jeder Änderung der Web-API oder des Ressourcenschemas fügen Sie eine für jede Ressource eine Versionsnummer für den URI hinzu. Die bereits vorhandenen URIs sollten weiterhin Ressourcen zurückgeben, die ihrem ursprünglichen Schema entsprechen.

Erweiterung des vorherigen Beispiels: Wenn das Feld `address` in untergeordnete Felder umstrukturiert wird, die jeweils einzelne Teile der Adresse enthalten (etwa `streetAddress`, `city`, `state` und `zipCode`), kann diese Version der Ressource über einen URI verfügbar gemacht werden, der eine Versionsnummer enthält (Beispiel: http://adventure-works.com/v2/customers/3:).

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Dieser Versionsverwaltungsmechanismus ist sehr einfach, hängt jedoch vom Server ab, der die Anforderung an das entsprechende Endgerät weiterleitet. Dieser Mechanismus kann jedoch schwerfällig werden, wenn die API mehrere Iterationen durchläuft und der Server eine Reihe von verschiedenen Versionen unterstützen muss. Aus der Sicht einer Puristen rufen die Clientanwendungen außerdem in allen Fällen dieselben Daten (Kunde 3) ab; daher sollte sich der URI im Grunde nicht abhängig von der Version unterscheiden. Dieses Schema erschwert zudem die Implementierung von HATEOAS, da alle Links die Versionsnummer in den URIs enthalten müssen.

### <a name="query-string-versioning"></a>Versionsverwaltung der Abfragezeichenfolge
Anstatt mehrere URIs bereitzustellen, können Sie auch die Version der Ressource angeben, indem Sie in der Abfragezeichenfolge einen Parameter an die HTTP-Anforderung anfügen (Beispiel: *http://adventure-works.com/customers/3?version=2*). Der Versionsparameter sollte einen aussagekräftigen Standardwert wie z. B. „1“ aufweisen, wenn er von älteren Clientanwendungen weggelassen wird.

Dieser Ansatz hat den semantischen Vorteil, dass dieselbe Ressource immer vom gleichen URI abgerufen wird; er hängt jedoch davon ab, dass der Code, der die Anforderung behandelt, die Abfragezeichenfolge analysiert und die entsprechende HTTP-Antwort zurücksendet. Dieser Ansatz bringt beim Implementieren von HATEOAS dieselben Schwierigkeiten mit sich, wie der Mechanismus für die URI-Versionsverwaltung.

> [!NOTE]
> Einige ältere Webbrowser und Webproxys speichern keine Anforderungen zwischen, deren URI eine Abfragezeichenfolge enthält. Dies kann sich negativ auf die Leistung von Webanwendungen auswirken, die eine Web-API verwenden und in einem solchen Webbrowser ausgeführt werden.
>
>

### <a name="header-versioning"></a>Header-Versionsverwaltung
Anstatt die Versionsnummer als Abfragezeichenfolgen-Parameter hinzuzufügen, können Sie einen benutzerdefinierten Header implementieren, der die Version der Ressource angibt. Dieser Ansatz erfordert, dass die Clientanwendung allen Anforderungen den entsprechenden Header hinzufügt; der Code, der die Clientanforderung behandelt, kann jedoch einen Standardwert (Version 1) verwenden, wenn der Versionsheader fehlt. In den folgenden Beispielen wird ein benutzerdefinierter Header mit dem Namen *Custom-Header* verwendet. Der Wert dieses Headers gibt die Version der Web-API an.

Version 1:

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=1
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

Version 2:

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=2
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Beachten Sie, dass die HATEOAS-Implementierung wie auch bei den beiden vorhergehenden Ansätzen die Angabe der entsprechenden benutzerdefinierten Header in allen Links erfordert.

### <a name="media-type-versioning"></a>Versionsverwaltung des Medientyps
Wenn eine Clientanwendung eine HTTP GET-Anforderung an einen Webserver sendet, sollte sie wie weiter oben in diesem Handbuch beschrieben anhand eines Accept-Headers das Format des Inhalts vorgeben, den sie verarbeiten kann. Der Zweck des *Accept*-Headers besteht häufig darin, dass die Clientanwendung angeben kann, ob der Text der Antwort im XML-, JSON- oder einem anderen gängigen Format vorliegt, das der Client analysieren kann. Allerdings ist es möglich, benutzerdefinierte Medientypen zu definieren, die Informationen enthalten, die es der Clientanwendung ermöglichen, die erwartete Version einer Ressource anzugeben. Das folgende Beispiel zeigt eine Anforderung die einen *Accept*-Header mit dem Wert *application/vnd.adventure-works.v1+json* angibt. Das Element *vnd.adventure works.v1* weist den Webserver an, Version 1 der Ressource zurückzugeben, während das Element *json* angibt, dass der Antworttext im JSON-Format zurückgegeben werden soll:

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
Accept: application/vnd.adventure-works.v1+json
```

Der Code zum Verarbeiten der Anforderung ist dafür verantwortlich, den *Accept*-Header zu verarbeiten und so weit wie möglich zu berücksichtigen. (Die Clientanwendung kann im *Accept*-Header mehrere Formate angeben. In diesem Fall kann der Webserver das am besten geeignete Format für den Antworttext auswählen.) Der Webserver bestätigt das Format der Daten im Antworttext mithilfe des Content-Type-Headers:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

Wenn im Accept-Header keine bekannten Medientypen angegeben sind, kann der Webserver eine Antwortnachricht vom Typ HTTP 406 (Nicht akzeptabel) generieren oder eine Nachricht mit einem Standardmedientyp zurückgeben.

Dieser Ansatz ist wohl der ursprünglichste Mechanismus zur Versionsverwaltung und eignet sich natürlich für das HATEOAS-Prinzip, bei dem der MIME-Typ der zugehörigen Daten in Ressourcenlinks enthalten sein kann.

> [!NOTE]
> Beim Auswählen einer Strategie für die Versionsverwaltung sollten Sie auch die Auswirkungen auf die Leistung berücksichtigen, insbesondere auf die Zwischenspeicherung auf dem Webserver. Die Schemas der URI- und Abfragezeichenfolge-Versionsverwaltung sind insofern cachefreundlich, dass dieselbe Kombination aus URI und Abfragezeichenfolge jedes Mal auf dieselben Daten verweist.
>
> Die Versionsverwaltungsmechanismen für Header und Medientyp erfordern in der Regel zusätzliche Logik, um die Werte im benutzerdefinierten Header oder im Accept-Header zu untersuchen. In einer umfangreichen Umgebung können zahlreiche Clients, die verschiedene Versionen einer Web-API verwenden, zu einer erheblichen Menge von doppelt vorhandenen Daten in einem serverseitigen Cache führen. Dieses Problem wird akut, wenn eine Clientanwendung mit einem Webserver über einen Proxy kommuniziert, der Zwischenspeicherung implementiert und eine Anforderung nur an den Webserver weiterleitet, wenn sein Cache derzeit keine Kopie der angeforderten Daten enthält.
>
>

## <a name="open-api-initiative"></a>Open API Initiative
Die [Open API Initiative](https://www.openapis.org/) wurde von einem Branchenkonsortium ins Leben gerufen, um REST-API-Beschreibungen anbieterübergreifend zu standardisieren. Im Rahmen dieser Initiative wurde die Swagger 2.0-Spezifikation in OpenAPI Specification (OAS) umbenannt und in die Open API Initiative integriert.

Es kann ratsam sein, OpenAPI für Ihre Web-APIs zu nutzen. Zu berücksichtigende Punkte:

- Die OpenAPI-Spezifikation umfasst eine Reihe von wertenden Richtlinien zum Entwurf einer REST-API. Hierdurch ergeben sich Vorteile bei der Interoperabilität, aber Sie müssen Ihre API mit mehr Sorgfalt entwerfen, damit sie mit der Spezifikation konform ist.
- Für OpenAPI wird ein Ansatz genutzt, bei dem der Vertrag im Vordergrund steht, und nicht die Implementierung. Dies bedeutet, dass Sie zuerst den API-Vertrag (die Schnittstelle) entwerfen und dann Code schreiben, mit dem der Vertrag implementiert wird. 
- Mit Tools wie Swagger können Clientbibliotheken oder Dokumentationen aus API-Verträgen generiert werden. Informationen hierzu finden Sie beispielsweise unter [ASP.NET Core-Web-API-Hilfeseiten mit Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger).

## <a name="more-information"></a>Weitere Informationen
* [REST-API-Richtlinien von Microsoft](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md). Ausführliche Empfehlungen für das Entwerfen von öffentlichen REST-APIs.
* [Das REST-Cookbook](http://restcookbook.com/). Einführung in das Erstellen von RESTful-APIs.
* [Web-API-Checkliste](https://mathieu.fenniak.net/the-api-checklist/). Eine nützliche Liste der zu berücksichtigenden Punkte beim Entwerfen und Implementieren einer Web-API.
* [Open API Initiative](https://www.openapis.org/). Dokumentation und Implementierungsdetails zu Open-API.
