---
title: API-Implementierungsleitfaden
description: Anleitung zur Implementierung einer API
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: b4d197719380bf55033942b3ebcad384170d950d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="api-implementation"></a>API-Implementierung
[!INCLUDE [header](../_includes/header.md)]

Mit einer sorgfältig entworfenen RESTful-Web-API werden die Ressourcen, Beziehungen und Navigationsschemas definiert, auf die mit Clientanwendungen zugegriffen werden kann. Beim Implementieren und Bereitstellen einer Web-API sollten Sie die physischen Anforderungen der Umgebung  berücksichtigen, in der die Web-API gehostet wird. Außerdem sollten Sie eher darauf achten, wie die Web-API erstellt wurde, als auf die logische Struktur der Daten. In diesem Leitfaden geht es hauptsächlich um die bewährten Methoden zur Implementierung einer Web-API und deren Veröffentlichung, um sie für Clientanwendungen verfügbar zu machen. Ausführliche Informationen zum Web-API-Design finden Sie unter [API-Design](/azure/architecture/best-practices/api-design).

## <a name="considerations-for-processing-requests"></a>Aspekte der Verarbeitung von Anforderungen

Beachten Sie die folgenden Punkte, wenn Sie den Code zum Behandeln von Anforderungen implementieren.

### <a name="get-put-delete-head-and-patch-actions-should-be-idempotent"></a>GET-, PUT-, DELETE-, HEAD- und PATCH-Aktionen sollten idempotent sein

Der Code, mit dem diese Anforderungen implementiert werden, sollte nicht mit Nebeneffekten verbunden sein. Eine Anforderung, die für eine Ressource wiederholt ausgeführt wird, sollte zum gleichen Ergebnis führen. Das Senden von mehreren DELETE-Anforderungen an denselben URI sollte die gleiche Auswirkung haben, aber der HTTP-Statuscode in den Antwortnachrichten kann sich unterscheiden. Für die erste DELETE-Anforderung wird ggf. der Statuscode 204 (Kein Inhalt) zurückgegeben, während für eine nachfolgende DELETE-Anforderung unter Umständen der Statuscode 404 (Nicht gefunden) zurückgegeben wird.

> [!NOTE]
> Der Artikel [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) (Muster der Idempotenz) im Blog von Jonathan Oliver enthält eine Übersicht über die Idempotenz und ihre Verbindung mit Datenverwaltungsvorgängen.
>

### <a name="post-actions-that-create-new-resources-should-not-have-unrelated-side-effects"></a>Bei POST-Aktionen, mit denen neue Ressourcen erstellt werden, sollten keine nicht relevanten Nebeneffekte auftreten

Wenn mit einer POST-Anforderung eine neue Ressource erstellt werden soll, sollten die Auswirkungen der Anforderung auf die neue Ressource beschränkt sein (sowie unter Umständen auf alle direkt verwandten Ressourcen, falls eine Verknüpfung besteht). In einem E-Commerce-System werden mit einer POST-Anforderung, mit der eine neue Bestellung für einen Kunden erstellt wird, ggf. auch Lagerbestände verändert und Rechnungsinformationen erzeugt. Es sollten aber keine Informationen geändert werden, die sich nicht direkt auf die Bestellung beziehen, und es sollten sich auch keine anderen Nebeneffekte für den Gesamtzustand des Systems ergeben.

### <a name="avoid-implementing-chatty-post-put-and-delete-operations"></a>Vermeiden der Implementierung von umfangreichen POST-, PUT- und DELETE-Vorgängen

Bauen Sie die Unterstützung von POST-, PUT- und DELETE-Anforderungen über Ressourcenauflistungen ein. Eine POST-Anforderung kann die Details für mehrere neue Ressourcen enthalten und diese derselben Auflistung hinzufügen. Mit einer PUT-Anforderung kann der gesamte Ressourcensatz in einer Auflistung ersetzt werden, und mit einer DELETE-Anforderung kann eine gesamte Auflistung entfernt werden.

Die in der ASP.NET-Web-API 2 enthaltene OData-Unterstützung ermöglicht das Zusammenfassen von Anforderungen in Batches. Eine Clientanwendung kann mehrere Web-API-Anforderungen verpacken und in einer einzelnen HTTP-Anforderung an den Server senden. Sie erhält dann eine einzelne HTTP-Antwort mit den Antworten für die einzelnen Anforderungen. Weitere Informationen finden Sie unter [Introducing Batch Support in Web API and Web API OData](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx) (Einführung der Batchunterstützung für die Web-API und Web-API-OData).

### <a name="follow-the-http-specification-when-sending-a-response"></a>Befolgen der HTTP-Spezifikation beim Senden einer Antwort 

Eine Web-API muss Nachrichten mit dem richtigen HTTP-Statuscode zurückgeben, damit der Client bestimmen kann, wie er das Ergebnis behandeln soll. Außerdem die richtigen HTTP-Header, damit der Client die Art des Ergebnisses versteht, sowie einen richtig formatierten Text, damit der Client das Ergebnis analysieren kann. 

Ein POST-Vorgang sollte in diesem Fall den Statuscode 201 (Erstellt) zurückgeben, und die Antwortnachricht sollte den URI der neu erstellten Ressource im Location-Header enthalten.

### <a name="support-content-negotiation"></a>Unterstützung der Inhaltsaushandlung

Der Text einer Antwortnachricht kann Daten in unterschiedlichen Formaten enthalten. Bei einer HTTP GET-Anforderung können Daten beispielsweise im JSON- oder XML-Format zurückgegeben werden. Wenn der Client eine Anforderung sendet, kann diese einen Accept-Header zum Angeben der Datenformate enthalten, die behandelt werden können. Diese Formate werden als Medientypen angegeben. Ein Client, der eine GET-Anforderung zum Abrufen eines Bilds ausgibt, kann beispielsweise einen Accept-Header angeben, mit dem die für den Client verarbeitbaren Medientypen aufgeführt werden, z. B. „image/jpeg, image/gif, image/png“.  Wenn die Web-API das Ergebnis zurückgibt, sollte sie die Daten mit einem dieser Medientypen formatieren und das Format im Content-Type-Header der Antwort angeben.

Falls der Client keinen Accept-Header angibt, sollten Sie für den Text der Antwort ein geeignetes Standardformat verwenden. Beispiel: Das ASP.NET-Web-API-Framework nutzt für textbasierte Daten standardmäßig JSON.

### <a name="provide-links-to-support-hateoas-style-navigation-and-discovery-of-resources"></a>Bereitstellen von Links zum Unterstützen der Navigation im HATEOAS-Stil und der Ermittlung von Ressourcen

Mit dem HATEOAS-Ansatz kann ein Client von einem Startpunkt aus zu Ressourcen navigieren und diese ermitteln. Dies wird erreicht, indem Links mit URIs verwendet werden. Wenn ein Client eine HTTP GET-Anforderung zum Abrufen einer Ressource ausgibt, sollte die Antwort URIs enthalten, mit denen eine Clientanwendung alle direkt verwandten Ressourcen schnell finden kann. Es kann beispielsweise sein, dass ein Kunde in einer Web-API, die eine E-Commerce-Lösung unterstützt, viele Bestellungen aufgegeben hat. Wenn eine Clientanwendung die Details für einen Kunden abruft, sollte die Antwort Links enthalten, mit denen die Clientanwendung HTTP GET-Anforderungen zum Abrufen dieser Bestellungen senden kann. Darüber hinaus sollten mit Links im HATEOAS-Stil die anderen Vorgänge (POST, PUT, DELETE usw.) beschrieben werden, die von jeder verknüpften Ressource zusammen mit dem entsprechenden URI zum Durchführen der einzelnen Anforderungen unterstützt werden. Dieser Ansatz wird unter [API-Design][api-design] ausführlicher beschrieben.

Derzeit gibt es keine Standards, mit denen die Implementierung von HATEOAS geregelt wird, aber im folgenden Beispiel wird ein möglicher Ansatz veranschaulicht. In diesem Beispiel wird mit einer HTTP GET-Anforderung zum Suchen nach den Details für einen Kunden eine Antwort zurückgegeben, die HATEOAS-Links enthält, mit denen auf die Bestellungen des Kunden verwiesen wird:

```HTTP
GET http://adventure-works.com/customers/2 HTTP/1.1
Accept: text/json
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"CustomerID":2,"CustomerName":"Bert","Links":[
    {"rel":"self",
    "href":"http://adventure-works.com/customers/2",
    "action":"GET",
    "types":["text/xml","application/json"]},
    {"rel":"self",
    "href":"http://adventure-works.com/customers/2",
    "action":"PUT",
    "types":["application/x-www-form-urlencoded"]},
    {"rel":"self",
    "href":"http://adventure-works.com/customers/2",
    "action":"DELETE",
    "types":[]},
    {"rel":"orders",
    "href":"http://adventure-works.com/customers/2/orders",
    "action":"GET",
    "types":["text/xml","application/json"]},
    {"rel":"orders",
    "href":"http://adventure-works.com/customers/2/orders",
    "action":"POST",
    "types":["application/x-www-form-urlencoded"]}
]}
```

Die Kundendaten werden hierbei mit der `Customer` -Klasse dargestellt, wie im folgenden Codeausschnitt zu sehen ist. Die HATEOAS-Links sind in der `Links` -Auflistungseigenschaft enthalten:

```csharp
public class Customer
{
    public int CustomerID { get; set; }
    public string CustomerName { get; set; }
    public List<Link> Links { get; set; }
    ...
}

public class Link
{
    public string Rel { get; set; }
    public string Href { get; set; }
    public string Action { get; set; }
    public string [] Types { get; set; }
}
```

Der HTTP GET-Vorgang ruft die Kundendaten aus dem Speicher ab und erstellt ein `Customer`-Objekt und füllt anschließend die `Links`-Auflistung auf. Das Ergebnis ist als JSON-Antwortnachricht formatiert. Jeder Link umfasst die folgenden Felder:

* Die Beziehung zwischen dem zurückgegebenen Objekt und dem Objekt, das vom Link beschrieben wird. Hierbei wird mit „self“ angegeben, dass der Link ein Verweis zurück auf das Objekt selbst ist (ähnlich wie bei einem `this` -Zeiger in vielen objektorientierten Sprachen), und „orders“ ist der Name einer Auflistung, in der die dazugehörigen Bestellinformationen enthalten sind.
* Der Hyperlink (`Href`) für das Objekt, das mit dem Link beschrieben wird, in Form eines URI.
* Der Typ der HTTP-Anforderung (`Action`), die an diesen URI gesendet werden kann.
* Das Format der Daten (`Types`), die in der HTTP-Anforderung bereitgestellt werden sollen oder die je nach Art der Anforderung in der Antwort zurückgegeben werden können.

Mit den HATEOAS-Links in der HTTP-Beispielantwort wird angegeben, dass eine Clientanwendung die folgenden Vorgänge durchführen kann:

* Eine HTTP GET-Anforderung an den URI `http://adventure-works.com/customers/2`, um die Details des Kunden abzurufen (erneut). Die Daten können im XML- oder JSON-Format zurückgegeben werden.
* Eine HTTP PUT-Anforderung an den URI `http://adventure-works.com/customers/2`, um die Details des Kunden zu ändern. Die neuen Daten müssen in der Anforderungsnachricht im Format „x-www-form-urlencoded“ bereitgestellt werden.
* Eine HTTP DELETE-Anforderung an den URI `http://adventure-works.com/customers/2`, um den Kunden zu löschen. Die Anforderung erwartet keine zusätzlichen Informationen oder Rückgabedaten im Text der Antwortnachricht.
* Eine HTTP GET-Anforderung an den URI `http://adventure-works.com/customers/2/orders`, um alle Bestellungen des Kunden per Suche zu ermitteln. Die Daten können im XML- oder JSON-Format zurückgegeben werden.
* Eine HTTP PUT-Anforderung an den URI `http://adventure-works.com/customers/2/orders`, um eine neue Bestellung für den Kunden zu erstellen. Die Daten müssen in der Anforderungsnachricht im Format „x-www-form-urlencoded“ bereitgestellt werden.

## <a name="considerations-for-handling-exceptions"></a>Aspekte der Behandlung von Ausnahmen

Berücksichtigen Sie die folgenden Punkte, wenn ein Vorgang eine nicht abgefangene Ausnahme auslöst.

### <a name="capture-exceptions-and-return-a-meaningful-response-to-clients"></a>Erfassen von Ausnahmen und Zurückgeben einer aussagekräftigen Antwort an Clients

Mit dem Code zum Implementieren eines HTTP-Vorgangs sollte eine umfassende Ausnahmebehandlung bereitgestellt werden, anstatt zuzulassen, dass unerwartete Ausnahmen ins Framework gelangen. Wenn eine Ausnahme den erfolgreichen Abschluss des Vorgangs verhindert, kann die Ausnahme in der Antwortnachricht zurück übergeben werden. Sie sollte aber eine aussagekräftige Beschreibung des Fehlers enthalten, der die Ausnahme verursacht hat. Außerdem sollte die Ausnahme den passenden HTTP-Statuscode enthalten, anstatt einfach für jede Situation den Statuscode 500 zurückzugeben. Wenn eine Benutzeranforderung beispielsweise eine Datenbankaktualisierung bewirkt, die eine Verletzung einer Einschränkung darstellt (z. B. das versuchte Löschen eines Kunden, für den noch Bestellungen ausstehen), sollten Sie den Statuscode 409 (Conflict) und einen Nachrichtentext zurückgeben, um die Ursache des Konflikts anzugeben. Falls die Anforderung aufgrund einer anderen Bedingung nicht durchführbar ist, können Sie den Statuscode 400 (Bad Request) zurückgeben. Eine vollständige Liste mit HTTP-Statuscodes finden Sie auf der W3C-Website im Thema zu den [Statuscodedefinitionen](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).

Im Codebeispiel werden unterschiedliche Bedingungen abgefangen, und es wird eine entsprechende Antwort zurückgegeben.

```csharp
[HttpDelete]
[Route("customers/{id:int}")]
public IHttpActionResult DeleteCustomer(int id)
{
    try
    {
        // Find the customer to be deleted in the repository
        var customerToDelete = repository.GetCustomer(id);

        // If there is no such customer, return an error response
        // with status code 404 (Not Found)
        if (customerToDelete == null)
        {
                return NotFound();
        }

        // Remove the customer from the repository
        // The DeleteCustomer method returns true if the customer
        // was successfully deleted
        if (repository.DeleteCustomer(id))
        {
            // Return a response message with status code 204 (No Content)
            // To indicate that the operation was successful
            return StatusCode(HttpStatusCode.NoContent);
        }
        else
        {
            // Otherwise return a 400 (Bad Request) error response
            return BadRequest(Strings.CustomerNotDeleted);
        }
    }
    catch
    {
        // If an uncaught exception occurs, return an error response
        // with status code 500 (Internal Server Error)
        return InternalServerError();
    }
}
```

> [!TIP]
> Binden Sie keine Informationen ein, die für einen Angreifer nützlich wären, der einen Eindringversuch in Ihre API startet.
  
Viele Webserver fangen Fehlerbedingungen selbst ab, bevor sie die Web-API erreichen. Wenn Sie beispielsweise die Authentifizierung für eine Website konfigurieren und der Benutzer keine richtigen Authentifizierungsinformationen angibt, sollte der Webserver mit dem Statuscode 401 (Unauthorized) antworten. Nachdem ein Client authentifiziert wurde, kann Ihr Code eigene Überprüfungen durchführen, um zu bestätigen, dass der Client auf die angeforderte Ressource zugreifen kann. Wenn diese Autorisierung nicht erfolgreich ist, sollten Sie den Statuscode 403 (Forbidden) zurückgeben.
 
### <a name="handle-exceptions-consistently-and-log-information-about-errors"></a>Einheitliches Behandeln von Ausnahmen und Protokollieren von Informationen zu Fehlern

Erwägen Sie zum einheitlichen Behandeln von Ausnahmen die Implementierung einer globalen Strategie zur Fehlerbehandlung in der gesamten Web-API. Außerdem sollten Sie für eine Fehlerprotokollierung sorgen, bei der für jede Ausnahme alle Details erfasst werden. Dieses Fehlerprotokoll kann ausführliche Informationen enthalten, sofern es nicht für Clients über das Web zugänglich ist. 

### <a name="distinguish-between-client-side-errors-and-server-side-errors"></a>Unterscheiden zwischen clientseitigen und serverseitigen Fehlern

Im HTTP-Protokoll wird zwischen Fehlern unterschieden, die aufgrund der Clientanwendung auftreten (HTTP 4xx-Statuscodes), und Fehlern, die aufgrund eines Problems auf dem Server auftreten (HTTP 5xx-Statuscodes). Achten Sie darauf, dass Sie diese Konvention in allen Fehlerantwortnachrichten befolgen.

## <a name="considerations-for-optimizing-client-side-data-access"></a>Aspekte der Optimierung des clientseitigen Datenzugriffs
In einer verteilten Umgebung, z. B. mit einem Webserver und Clientanwendungen, ist das Netzwerk eines der Elemente, die am stärksten beachtet werden müssen. Es können sich erhebliche Engpässe ergeben, und zwar vor allem, wenn eine Clientanwendung häufig Anforderungen sendet oder Daten empfängt. Daher sollten Sie versuchen, die Menge des im Netzwerk übertragenen Datenverkehrs möglichst zu verringern. Beachten Sie beim Implementieren des Codes zum Abrufen und Verwalten von Daten die folgenden Punkte:

### <a name="support-client-side-caching"></a>Unterstützen der clientseitigen Zwischenspeicherung

Das HTTP 1.1-Protokoll unterstützt die Zwischenspeicherung auf Clients und Zwischenservern, über die eine Anforderung mit Cache-Control-Header weitergeleitet wird. Wenn eine Clientanwendung eine HTTP GET-Anforderung an die Web-API sendet, kann die Antwort einen Cache-Control-Header enthalten. Hiermit wird angegeben, ob die Daten im Text der Antwort vom Client oder einem Zwischenserver, über den die Anforderung weitergeleitet wurde, sicher zwischengespeichert werden können. Außerdem wird angegeben, nach welchem Zeitraum die Daten ablaufen und als veraltet angesehen werden. Das folgende Beispiel enthält eine HTTP GET-Anforderung und die dazugehörige Antwort mit einem Cache-Control-Header:

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
```

```HTTP
HTTP/1.1 200 OK
...
Cache-Control: max-age=600, private
Content-Type: text/json; charset=utf-8
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

In diesem Beispiel wird mit dem Cache-Control-Header angegeben, dass die zurückgegebenen Daten nach 600 Sekunden ablaufen sollen. Außerdem sind sie nur für einen einzelnen Client geeignet und dürfen nicht in einem freigegebenen Cache gespeichert werden, der von anderen Clients verwendet wird (sie sind als *private* festgelegt). Im Cache-Control-Header kann anstelle von *private* auch *public* angegeben werden, damit die Daten in einem freigegebenen Cache gespeichert werden können. Oder es kann *no-store* angegeben werden, wenn die Daten vom Client **nicht** zwischengespeichert werden dürfen. Im folgenden Codebeispiel wird veranschaulicht, wie Sie einen Cache-Control-Header in einer Antwortnachricht erstellen:

```csharp
public class OrdersController : ApiController
{
    ...
    [Route("api/orders/{id:int:min(0)}")]
    [HttpGet]
    public IHttpActionResult FindOrderByID(int id)
    {
        // Find the matching order
        Order order = ...;
        ...
        // Create a Cache-Control header for the response
        var cacheControlHeader = new CacheControlHeaderValue();
        cacheControlHeader.Private = true;
        cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);
        ...

        // Return a response message containing the order and the cache control header
        OkResultWithCaching<Order> response = new OkResultWithCaching<Order>(order, this)
        {
            CacheControlHeader = cacheControlHeader
        };
        return response;
    }
    ...
}
```

In diesem Code wird eine benutzerdefinierte `IHttpActionResult`-Klasse mit dem Namen `OkResultWithCaching` verwendet. Diese Klasse ermöglicht dem Controller das Festlegen des Cacheheaderinhalts:

```csharp
public class OkResultWithCaching<T> : OkNegotiatedContentResult<T>
{
    public OkResultWithCaching(T content, ApiController controller)
        : base(content, controller) { }

    public OkResultWithCaching(T content, IContentNegotiator contentNegotiator, HttpRequestMessage request, IEnumerable<MediaTypeFormatter> formatters)
        : base(content, contentNegotiator, request, formatters) { }

    public CacheControlHeaderValue CacheControlHeader { get; set; }
    public EntityTagHeaderValue ETag { get; set; }

    public override async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        HttpResponseMessage response;
        try
        {
            response = await base.ExecuteAsync(cancellationToken);
            response.Headers.CacheControl = this.CacheControlHeader;
            response.Headers.ETag = ETag;
        }
        catch (OperationCanceledException)
        {
            response = new HttpResponseMessage(HttpStatusCode.Conflict) {ReasonPhrase = "Operation was cancelled"};
        }
        return response;
    }
}
```

> [!NOTE]
> Außerdem wird im HTTP-Protokoll auch die *no-cache*-Direktive für den Cache-Control-Header definiert. Es ist etwas verwirrend, dass diese Direktive nicht etwa „nicht zwischenspeichern“, sondern „zwischengespeicherte Informationen vor dem Zurückgeben per Server neu bewerten“ bedeutet. Die Daten können zwar zwischengespeichert werden, aber sie werden bei jeder Verwendung überprüft, um sicherzustellen, dass sie noch aktuell sind.
>
>

Für die Cacheverwaltung ist die Clientanwendung oder der Zwischenserver verantwortlich, aber bei einer richtigen Implementierung kann Bandbreite gespart und die Leistung verbessert werden. Zu diesem Zweck wird verhindert, dass keine Daten mehr abgerufen werden müssen, die bereits vorher abgerufen wurden.

Der *max-age*-Wert im Cache-Control-Header ist nur ein Anhaltspunkt und keine Garantie, dass sich die entsprechenden Daten während des angegebenen Zeitraums nicht ändern. Die Web-API sollte den max-age-Wert je nach der erwarteten Volatilität der Daten auf einen geeigneten Wert festlegen. Wenn dieser Zeitraum abgelaufen ist, sollte der Client das Objekt aus dem Cache entfernen.

> [!NOTE]
> Die meisten modernen Webbrowser unterstützen die clientseitige Zwischenspeicherung, indem Anforderungen die passenden Cache-Control-Header hinzugefügt werden und die Header der Ergebnisse (wie beschrieben) untersucht werden. Einige ältere Browser speichern aber keine Werte zwischen, die über eine URL mit einer Abfragezeichenfolge zurückgegeben werden. Normalerweise ist dies kein Problem für benutzerdefinierte Clientanwendungen, bei denen basierend auf dem hier beschriebenen Protokoll eine eigene Strategie zur Cacheverwaltung implementiert wird.
>
> Einige ältere Proxys weisen das gleiche Verhalten auf und speichern unter Umständen keine Anforderungen zwischen, die auf URLs mit Abfragezeichenfolgen basieren. Dies kann für benutzerdefinierte Clientanwendungen ein Problem darstellen, die über einen Proxy dieser Art eine Verbindung mit einem Webserver herstellen.
>

### <a name="provide-etags-to-optimize-query-processing"></a>Bereitstellen von ETags zum Optimieren der Abfrageverarbeitung

Wenn eine Clientanwendung ein Objekt abruft, kann die Antwortnachricht auch ein *ETag* (Entitätstag) enthalten. Ein ETag ist eine opake Zeichenfolge, mit der die Version einer Ressource angegeben wird. Wenn eine Ressource geändert wird, wird auch das ETag geändert. Dieses ETag sollte von der Clientanwendung als Teil der Daten zwischengespeichert werden. Im folgenden Codebeispiel wird veranschaulicht, wie Sie einer HTTP GET-Anforderung ein ETag als Teil der Antwort hinzufügen. In diesem Code wird die `GetHashCode`-Methode eines Objekts zum Generieren eines numerischen Werts verwendet, mit dem das Objekt identifiziert wird (Sie können diese Methode bei Bedarf außer Kraft setzen und einen eigenen Hashwert generieren, indem Sie einen Algorithmus verwenden, z. B. MD5):

```csharp
public class OrdersController : ApiController
{
    ...
    public IHttpActionResult FindOrderByID(int id)
    {
        // Find the matching order
        Order order = ...;
        ...

        var hashedOrder = order.GetHashCode();
        string hashedOrderEtag = $"\"{hashedOrder}\"";
        var eTag = new EntityTagHeaderValue(hashedOrderEtag);

        // Return a response message containing the order and the cache control header
        OkResultWithCaching<Order> response = new OkResultWithCaching<Order>(order, this)
        {
            ...,
            ETag = eTag
        };
        return response;
    }
    ...
}
```

Die von der Web-API bereitgestellte Antwortnachricht sieht wie folgt aus:

```HTTP
HTTP/1.1 200 OK
...
Cache-Control: max-age=600, private
Content-Type: text/json; charset=utf-8
ETag: "2147483648"
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

> [!TIP]
> Aus Sicherheitsgründen sollten Sie nicht zulassen, dass sensible Daten oder Daten, die über eine authentifizierte Verbindung (HTTPS) zurückgegeben werden, zwischengespeichert werden.
>
>

Eine Clientanwendung kann eine nachfolgende GET-Anforderung ausgeben, um jederzeit dieselbe Ressource abzurufen. Wenn sich die Ressource geändert hat (also ein anderes ETag aufweist), sollte die zwischengespeicherte Version verworfen und die neue Version dem Cache hinzugefügt werden. Falls eine Ressource umfangreich ist und für die Übertragung zurück auf den Client eine erhebliche Menge an Bandbreite erfordert, kann die Verwendung wiederholter Anforderungen zum Abrufen derselben Daten ineffizient werden. Als Lösung definiert das HTTP-Protokoll den folgenden Prozess zum Optimieren von GET-Anforderungen, die Sie in einer Web-API unterstützen sollten:

* Der Client erstellt eine GET-Anforderung mit dem ETag für die derzeit zwischengespeicherte Version der Ressource, auf die in einem If-None-Match-HTTP-Header verwiesen wird:

    ```HTTP
    GET http://adventure-works.com/orders/2 HTTP/1.1
    If-None-Match: "2147483648"
    ```
* Der GET-Vorgang in der Web-API ruft das aktuelle ETag für die angeforderten Daten ab („Order 2“ im obigen Beispiel) und vergleicht es mit dem Wert im If-None-Match-Header.
* Wenn das aktuelle ETag für die angeforderten Daten mit dem von der Anforderung bereitgestellten ETag übereinstimmt, hat sich die Ressource nicht geändert. Die Web-API sollte also eine HTTP-Antwort mit leerem Nachrichtentext und dem Statuscode 304 (Not Modified) zurückgeben.
* Wenn das aktuelle ETag für die angeforderten Daten nicht mit dem von der Anforderung bereitgestellten ETag übereinstimmt, haben sich die Daten geändert. Die Web-API sollte also eine HTTP-Antwort mit den neuen Daten im Nachrichtentext und dem Statuscode 200 (OK) zurückgeben.
* Falls die angeforderten Daten nicht mehr vorhanden sind, sollte die Web-API eine HTTP-Antwort mit dem Statuscode 404 (Nicht gefunden) zurückgeben.
* Der Client verwendet den Statuscode zum Verwalten des Cache. Wenn sich die Daten nicht geändert haben (Statuscode 304), kann das Objekt zwischengespeichert bleiben, und die Clientanwendung sollte weiterhin diese Version des Objekts nutzen. Wenn sich die Daten geändert haben (Statuscode 200), sollte das zwischengespeicherte Objekt verworfen und das neue Objekt eingefügt werden. Falls die Daten nicht mehr verfügbar sind (Statuscode 404), sollte das Objekt aus dem Cache entfernt werden.

> [!NOTE]
> Wenn der Antwortheader den Cache-Control-Header „no-store“ enthält, sollte das Objekt unabhängig vom HTTP-Statuscode aus dem Cache entfernt werden.
>

Im Code unten wird die erweiterte `FindOrderByID` -Methode zum Unterstützen des If-None-Match-Headers veranschaulicht. Beachten Sie Folgendes: Wenn der If-None-Match-Header weggelassen wird, wird immer die angegebene Bestellung abgerufen:

```csharp
public class OrdersController : ApiController
{
    [Route("api/orders/{id:int:min(0)}")]
    [HttpGet]
    public IHttpActionResult FindOrderByID(int id)
    {
        try
        {
            // Find the matching order
            Order order = ...;

            // If there is no such order then return NotFound
            if (order == null)
            {
                return NotFound();
            }

            // Generate the ETag for the order
            var hashedOrder = order.GetHashCode();
            string hashedOrderEtag = $"\"{hashedOrder}\"";

            // Create the Cache-Control and ETag headers for the response
            IHttpActionResult response;
            var cacheControlHeader = new CacheControlHeaderValue();
            cacheControlHeader.Public = true;
            cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);
            var eTag = new EntityTagHeaderValue(hashedOrderEtag);

            // Retrieve the If-None-Match header from the request (if it exists)
            var nonMatchEtags = Request.Headers.IfNoneMatch;

            // If there is an ETag in the If-None-Match header and
            // this ETag matches that of the order just retrieved,
            // then create a Not Modified response message
            if (nonMatchEtags.Count > 0 &&
                String.CompareOrdinal(nonMatchEtags.First().Tag, hashedOrderEtag) == 0)
            {
                response = new EmptyResultWithCaching()
                {
                    StatusCode = HttpStatusCode.NotModified,
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag
                };
            }
            // Otherwise create a response message that contains the order details
            else
            {
                response = new OkResultWithCaching<Order>(order, this)
                {
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag
                };
            }

            return response;
        }
        catch
        {
            return InternalServerError();
        }
    }
...
}
```

Dieses Beispiel enthält eine zusätzliche benutzerdefinierte `IHttpActionResult`-Klasse mit dem Namen `EmptyResultWithCaching`. Diese Klasse dient lediglich als Wrapper für ein `HttpResponseMessage` -Objekt ohne Antworttext:

```csharp
public class EmptyResultWithCaching : IHttpActionResult
{
    public CacheControlHeaderValue CacheControlHeader { get; set; }
    public EntityTagHeaderValue ETag { get; set; }
    public HttpStatusCode StatusCode { get; set; }
    public Uri Location { get; set; }

    public async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        HttpResponseMessage response = new HttpResponseMessage(StatusCode);
        response.Headers.CacheControl = this.CacheControlHeader;
        response.Headers.ETag = this.ETag;
        response.Headers.Location = this.Location;
        return response;
    }
}
```

> [!TIP]
> In diesem Beispiel wird das ETag für die Daten generiert, indem die Daten, die aus der zugrunde liegenden Datenquelle abgerufen werden, mit einem Hashwert versehen werden. Wenn das ETag auf andere Art berechnet werden kann, kann der Prozess weiter optimiert werden, und die Daten müssen nur dann aus der Datenquelle abgerufen werden, wenn sie sich geändert haben.  Dieser Ansatz ist besonders nützlich, wenn die Daten sehr umfangreich sind oder das Zugreifen auf die Datenquelle zu einer längeren Wartezeit führen kann (beispielsweise bei einer Remotedatenbank als Datenquelle).
>

### <a name="use-etags-to-support-optimistic-concurrency"></a>Verwenden von ETags zum Unterstützen der optimistischen Parallelität

Zum Ermöglichen von Updates für zuvor zwischengespeicherte Daten unterstützt das HTTP-Protokoll die Strategie der optimistischen Parallelität. Wenn die Clientanwendung nach dem Abrufen und Zwischenspeichern einer Ressource nachfolgend eine PUT- oder DELETE-Anforderung sendet, um die Ressource zu ändern oder zu entfernen, sollte diese einen If-Match-Header mit Verweis auf das ETag enthalten. Mit diesen Informationen kann die Web-API dann bestimmen, ob die Ressource bereits von einem anderen Benutzer geändert wurde, seitdem sie abgerufen wurde, und wie folgt eine geeignete Antwort zurück an die Clientanwendung senden:

* Der Client erstellt eine PUT-Anforderung mit den neuen Details für die Ressource und dem ETag für die derzeit zwischengespeicherte Version der Ressource, auf die in einem If-Match-HTTP-Header verwiesen wird. Das folgende Beispiel enthält eine PUT-Anforderung, mit der eine Bestellung aktualisiert wird:

    ```HTTP
    PUT http://adventure-works.com/orders/1 HTTP/1.1
    If-Match: "2282343857"
    Content-Type: application/x-www-form-urlencoded
    Content-Length: ...
    productID=3&quantity=5&orderValue=250
    ```
* Der PUT-Vorgang in der Web-API ruft das aktuelle ETag für die angeforderten Daten ab („Order 1“ im obigen Beispiel) und vergleicht es mit dem Wert im If-Match-Header.
* Wenn das aktuelle ETag für die angeforderten Daten mit dem von der Anforderung bereitgestellten ETag übereinstimmt, hat sich die Ressource nicht geändert. Die Web-API sollte also das Update durchführen und eine Nachricht mit dem HTTP-Statuscode 204 (No Content) zurückgeben, wenn der Vorgang erfolgreich ist. Die Antwort kann Cache-Control- und ETag-Header für die aktualisierte Version der Ressource enthalten. Die Antwort sollte immer den Location-Header enthalten, mit dem auf den URI der gerade aktualisierten Ressource verwiesen wird.
* Wenn das aktuelle ETag für die angeforderten Daten nicht mit dem von der Anforderung bereitgestellten ETag übereinstimmt, wurden die Daten von einem anderen Benutzer geändert, seitdem sie abgerufen wurden. Die Web-API sollte eine HTTP-Antwort mit einem leeren Nachrichtentext und dem Statuscode 412 (Precondition Failed) zurückgeben.
* Falls die zu aktualisierende Ressource nicht mehr vorhanden ist, sollte die Web-API eine HTTP-Antwort mit dem Statuscode 404 (Nicht gefunden) zurückgeben.
* Der Client nutzt den Statuscode und die Antwortheader zum Verwalten des Cache. Wenn die Daten aktualisiert wurden (Statuscode 204), kann das Objekt zwischengespeichert bleiben (sofern im Cache-Control-Header nicht „no-store“ angegeben ist), aber das ETag muss aktualisiert werden. Falls die Daten von einem anderen Benutzer geändert (Statuscode 412) oder Nicht gefunden wurden (Statuscode 404), sollte das zwischengespeicherte Objekt verworfen werden.

Im nächsten Codebeispiel wird eine Implementierung des PUT-Vorgangs für den Orders-Controller veranschaulicht:

```csharp
public class OrdersController : ApiController
{
    [HttpPut]
    [Route("api/orders/{id:int}")]
    public IHttpActionResult UpdateExistingOrder(int id, DTOOrder order)
    {
        try
        {
            var baseUri = Constants.GetUriFromConfig();
            var orderToUpdate = this.ordersRepository.GetOrder(id);
            if (orderToUpdate == null)
            {
                return NotFound();
            }

            var hashedOrder = orderToUpdate.GetHashCode();
            string hashedOrderEtag = $"\"{hashedOrder}\"";

            // Retrieve the If-Match header from the request (if it exists)
            var matchEtags = Request.Headers.IfMatch;

            // If there is an Etag in the If-Match header and
            // this etag matches that of the order just retrieved,
            // or if there is no etag, then update the Order
            if (((matchEtags.Count > 0 &&
                String.CompareOrdinal(matchEtags.First().Tag, hashedOrderEtag) == 0)) ||
                matchEtags.Count == 0)
            {
                // Modify the order
                orderToUpdate.OrderValue = order.OrderValue;
                orderToUpdate.ProductID = order.ProductID;
                orderToUpdate.Quantity = order.Quantity;

                // Save the order back to the data store
                // ...

                // Create the No Content response with Cache-Control, ETag, and Location headers
                var cacheControlHeader = new CacheControlHeaderValue();
                cacheControlHeader.Private = true;
                cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);

                hashedOrder = order.GetHashCode();
                hashedOrderEtag = $"\"{hashedOrder}\"";
                var eTag = new EntityTagHeaderValue(hashedOrderEtag);

                var location = new Uri($"{baseUri}/{Constants.ORDERS}/{id}");
                var response = new EmptyResultWithCaching()
                {
                    StatusCode = HttpStatusCode.NoContent,
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag,
                    Location = location
                };

                return response;
            }

            // Otherwise return a Precondition Failed response
            return StatusCode(HttpStatusCode.PreconditionFailed);
        }
        catch
        {
            return InternalServerError();
        }
    }
    ...
}
```

> [!TIP]
> Die Verwendung des If-Match-Headers ist absolut optional. Wenn er weggelassen wird, versucht die Web-API stets, die angegebene Bestellung zu aktualisieren. Hierbei kann es unter Umständen vorkommen, dass ein Update eines anderen Benutzers versehentlich überschrieben wird. Geben Sie zur Vermeidung von Problemen aufgrund von verloren gegangenen Updates immer einen If-Match-Header an.
>
>

## <a name="considerations-for-handling-large-requests-and-responses"></a>Aspekte zur Behandlung umfangreicher Anforderungen und Antworten
Wenn eine Clientanwendung Anforderungen ausgibt, bei denen Daten gesendet oder empfangen werden, kann es vorkommen, dass diese mehrere Megabyte groß (oder noch größer) sind. Das Warten auf den Abschluss der Übertragung dieser Datenmenge kann dazu führen, dass die Clientanwendung nicht mehr reagiert. Beachten Sie die folgenden Punkte, wenn Sie Anforderungen behandeln müssen, die größere Datenmengen enthalten:

### <a name="optimize-requests-and-responses-that-involve-large-objects"></a>Optimieren von Anforderungen und Antworten, die große Objekte enthalten

Einige Ressourcen können große Objekte sein oder große Felder enthalten, z. B. Grafiken oder andere Arten von Binärdaten. Eine Web-API sollte das Streamen unterstützen, um das Hoch- und Herunterladen dieser Ressourcen zu optimieren.

Das HTTP-Protokoll stellt das Verfahren für die segmentierte Transfercodierung bereit, mit dem große Datenobjekte zurück auf einen Client gestreamt werden können. Wenn der Client eine HTTP GET-Anforderung für ein großes Objekt sendet, kann die Web-API die Antwort in einzelnen *Segmenten* („Chunks“) über eine HTTP-Verbindung zurücksenden. Die Länge der Daten in der Antwort ist am Anfang unter Umständen noch nicht bekannt (ggf. werden sie erst generiert). Der Server, auf dem die Web-API gehostet wird, sollte also mit jedem Segment eine Antwortnachricht senden, für die der Header „Transfer-Encoding: Chunked“ angegeben ist, und kein Content-Length-Header. Die Clientanwendung kann die einzelnen Segmente empfangen, um die vollständige Antwort zusammenzusetzen. Die Datenübertragung ist abgeschlossen, wenn der Server das letzte Segment mit einer Größe von null zurücksendet. 

Es ist vorstellbar, dass eine einzelne Anforderung zu einem riesigen Objekt führt, für das Ressourcen in erheblichem Umfang verbraucht werden. Wenn die Web-API während des Streamingvorgangs ermittelt, dass die Menge der Daten in einer Anforderung nicht mehr akzeptabel ist, kann sie den Vorgang abbrechen und eine Antwortnachricht mit dem Statuscode 413 (Request Entity Too Large) zurückgeben.

Sie können die Größe von großen Objekten, die über das Netzwerk übertragen werden, per HTTP-Komprimierung verringern. Dieser Ansatz ist hilfreich, um die Menge des Datenverkehrs im Netzwerk und die damit verbundene Netzwerklatenz zu reduzieren. Der Nachteil ist, dass auf dem Client und dem Server, auf dem die Web-API gehostet wird, zusätzlicher Verarbeitungsaufwand anfällt. Eine Clientanwendung, die den Eingang von komprimierten Daten erwartet, kann beispielsweise den Anforderungsheader „Accept-Encoding: gzip“ einfügen (auch andere Algorithmen zur Datenkomprimierung können angegeben werden). Wenn der Server die Komprimierung unterstützt, sollte die Antwort den Inhalt im gzip-Format im Nachrichtentext sowie den Antwortheader „Content-Encoding: gzip“ enthalten.

Sie können die codierte Komprimierung mit dem Streaming kombinieren. Komprimieren Sie die Daten vor dem Streamen, und geben Sie die gzip-Inhaltscodierung und die segmentierte Transfercodierung in den Nachrichtenheadern an. Beachten Sie außerdem, dass einige Webserver (z. B. Internet Information Server) so konfiguriert werden können, dass HTTP-Antworten automatisch komprimiert werden. Dies gilt unabhängig davon, ob die Web-API die Daten komprimiert oder nicht.

### <a name="implement-partial-responses-for-clients-that-do-not-support-asynchronous-operations"></a>Implementieren von Teilantworten für Clients, die keine asynchronen Vorgänge unterstützen

Als Alternative zum asynchronen Streaming kann eine Clientanwendung Daten für große Objekte explizit in Segmenten anfordern. Diese werden als Teilantworten bezeichnet. Die Clientanwendung sendet eine HTTP HEAD-Anforderung, um Informationen zum Objekt abzurufen. Wenn die Web-API Teilantworten unterstützt, sollte sie auf die HEAD-Anforderung mit einer Antwortnachricht reagieren, die einen Accept-Ranges-Header und einen Content-Length-Header zum Angeben der Gesamtgröße des Objekts enthält. Der Text der Nachricht sollte aber leer sein. Die Clientanwendung kann diese Informationen verwenden, um eine Reihe von GET-Anforderungen zu erstellen, mit denen ein Bereich für den zu empfangenden Bytebereich angegeben wird. Die Web-API sollte eine Antwortnachricht mit dem HTTP-Status 206 (Partial Content), einen Content-Length-Header zum Angeben des tatsächlichen Betrags an Daten im Text der Antwortnachricht und einen Content-Range-Header zurückgeben, mit dem angegeben wird, welchen Teil des Objekts (z. B. Bytes 4.000 bis 8.000) diese Daten darstellen.

HTTP HEAD-Anforderungen und Teilantworten werden unter [API-Design][api-design] ausführlicher beschrieben.

### <a name="avoid-sending-unnecessary-100-continue-status-messages-in-client-applications"></a>Vermeiden des Sendens von unnötigen 100-Continue-Statusnachrichten in Clientanwendungen

Eine Clientanwendung, die eine große Datenmenge an einen Server senden möchte, kann auch zuerst ermitteln, ob auf dem Server die Bereitschaft zum Annehmen der Anforderung besteht. Vor dem Senden der Daten kann die Clientanwendung eine HTTP-Anforderung mit einem Header „Expect: 100-Continue“ und einem Content-Length-Header zum Angeben der Größe für die Daten übermitteln. Der Nachrichtentext ist hierbei leer. Wenn der Server zum Behandeln der Anforderung bereit ist, sollte er mit einer Nachricht antworten, in der der HTTP-Status 100 (Continue) angegeben ist. Die Clientanwendung kann den Vorgang dann fortsetzen und die gesamte Anforderung senden, einschließlich der Daten im Nachrichtentext.

Wenn Sie einen Dienst per IIS hosten, werden Expect: 100-Continue-Header vom Treiber „HTTP.sys“ automatisch erkannt und behandelt, bevor Anforderungen an Ihre Webanwendung übergeben werden. Dies bedeutet, dass Sie diese Header wahrscheinlich nicht in Ihrem Anwendungscode sehen, und Sie können davon ausgehen, dass per IIS bereits alle Nachrichten herausgefiltert wurden, die als ungeeignet oder zu groß angesehen werden.

Wenn Sie Clientanwendungen mit dem .NET Framework erstellen, werden für alle POST- und PUT-Nachrichten standardmäßig zuerst Nachrichten mit Expect: 100-Continue-Headern gesendet. Wie auf der Serverseite auch, wird der Prozess vom .NET Framework transparent behandelt. Dieser Prozess führt aber dazu, dass jede POST- und PUT-Anforderung zwei Roundtrips zum Server verursacht. Dies gilt auch für kleine Anforderungen. Wenn von Ihrer Anwendung keine Anforderungen mit großen Datenmengen gesendet werden, können Sie dieses Feature deaktivieren. Verwenden Sie hierzu die `ServicePointManager`-Klasse, um `ServicePoint`-Objekte in der Clientanwendung zu erstellen. Ein `ServicePoint`-Objekt behandelt die Verbindungen, die der Client basierend auf dem Schema und den Hostfragmenten von URIs, mit denen Ressourcen auf dem Server identifiziert werden, mit einem Server herstellt. Sie können die `Expect100Continue`-Eigenschaft des `ServicePoint`-Objekts dann auf „false“ festlegen. Alle nachfolgenden POST- und PUT-Anforderungen, die vom Client über einen URI gesendet werden, der mit dem Schema und den Hostfragmenten des `ServicePoint`-Objekts übereinstimmt, werden ohne Expect: 100-Continue-Header gesendet. Im folgenden Code wird veranschaulicht, wie Sie ein `ServicePoint`-Objekt konfigurieren, mit dem alle an URIs gesendeten Anforderungen mit dem Schema `http` und dem Host `www.contoso.com` konfiguriert werden.

```csharp
Uri uri = new Uri("http://www.contoso.com/");
ServicePoint sp = ServicePointManager.FindServicePoint(uri);
sp.Expect100Continue = false;
```

Sie können die statische `Expect100Continue`-Eigenschaft der `ServicePointManager`-Klasse auch so festlegen, dass der Standardwert dieser Eigenschaft für alle nachfolgend erstellten `ServicePoint`-Objekte angegeben wird. Weitere Informationen finden Sie unter [ServicePoint-Klasse](https://msdn.microsoft.com/library/system.net.servicepoint.aspx).

### <a name="support-pagination-for-requests-that-may-return-large-numbers-of-objects"></a>Unterstützen der Paginierung für Anforderungen, bei denen ggf. eine große Zahl von Objekten zurückgegeben wird

Wenn eine Auflistung eine große Zahl von Ressourcen enthält, kann das Ausgeben einer GET-Anforderung an den entsprechenden URI zu erheblichem Verarbeitungsaufwand auf dem Hostserver der Web-API führen. Dies kann die Leistung beeinträchtigen und einen signifikanten Anstieg des Netzwerkdatenverkehrs und somit längere Wartezeiten nach sich ziehen.

Zum Behandeln dieser Fälle sollte die Web-API Abfragezeichenfolgen unterstützen, mit denen die Clientanwendung Anforderungen verfeinern oder Daten in besser verwaltbaren, diskreten Blöcken (bzw. Seiten) abrufen kann. Im folgenden Code wird die `GetAllOrders`-Methode im `Orders`-Controller veranschaulicht. Mit dieser Methode werden die Details der Bestellungen abgerufen. Wenn für diese Methode keine Einschränkungen vorliegen, kann darüber ggf. eine große Datenmenge zurückgegeben werden. Mit den Parametern `limit` und `offset` soll der Umfang der Daten auf eine kleinere Teilmenge reduziert werden. In diesem Fall sind dies standardmäßig nur die ersten zehn Bestellungen:

```csharp
public class OrdersController : ApiController
{
    ...
    [Route("api/orders")]
    [HttpGet]
    public IEnumerable<Order> GetAllOrders(int limit=10, int offset=0)
    {
        // Find the number of orders specified by the limit parameter
        // starting with the order specified by the offset parameter
        var orders = ...
        return orders;
    }
    ...
}
```

Eine Clientanwendung kann eine Anforderung senden, um 30 Bestellungen ab Offset 50 abzurufen, indem der URI `http://www.adventure-works.com/api/orders?limit=30&offset=50` verwendet wird.

> [!TIP]
> Sie sollten verhindern, dass Clientanwendungen Abfragezeichenfolgen angeben können, die zu einem URI mit einer Länge von mehr als 2.000 Zeichen führen. Viele Webclients und -server können keine URIs dieser Länge verarbeiten.
>
>

## <a name="considerations-for-maintaining-responsiveness-scalability-and-availability"></a>Aspekte zur Aufrechterhaltung der Reaktionsfähigkeit, Skalierbarkeit und Verfügbarkeit
Eine Web-API kann von vielen Clientanwendungen, die weltweit an den unterschiedlichsten Orten ausgeführt werden, gemeinsam verwendet werden. Es ist wichtig sicherzustellen, dass für die Implementierung der Web-API Folgendes gilt: Auch bei einer hohen Auslastung bleibt die Reaktionsfähigkeit erhalten, sie kann zur Unterstützung stark variierender Workloads skaliert werden, und für Clients, auf denen unternehmenskritische Vorgänge ausgeführt werden, wird die Verfügbarkeit garantiert. Beachten Sie die folgenden Punkte, wenn Sie ermitteln möchten, wie diese Anforderungen erfüllt werden können:

### <a name="provide-asynchronous-support-for-long-running-requests"></a>Bereitstellen von asynchroner Unterstützung für Anforderungen mit langer Ausführungsdauer

Eine Anforderung, deren Verarbeitung unter Umständen sehr lange dauern kann, sollte durchgeführt werden, ohne den übermittelnden Client zu blockieren. Die Web-API kann einige erste Überprüfungen durchführen, um die Anforderung zu validieren, eine separate Aufgabe zum Ausführen der Arbeit initiieren und dann eine Antwortnachricht mit dem HTTP-Code 202 (Accepted) zurückgeben. Die Aufgabe kann asynchron im Rahmen der Web-API-Verarbeitung ausgeführt oder auf eine Hintergrundaufgabe verlagert werden.

Außerdem sollte die Web-API über ein Verfahren zum Zurückgeben der Ergebnisse einer Verarbeitung an die Clientanwendung verfügen. Hierzu können Sie einen Abrufmechanismus für Clientanwendungen bereitstellen, um regelmäßig abzufragen, ob die Verarbeitung abgeschlossen ist, und das Ergebnis abzurufen. Sie können die Web-API auch so einrichten, dass nach Abschluss des Vorgangs eine Benachrichtigung gesendet wird.

Sie können einen einfachen Abrufmechanismus implementieren, indem Sie einen *polling*-URI angeben, der als virtuelle Ressource fungiert. Gehen Sie hierbei wie folgt vor:

1. Die Clientanwendung sendet die erste Anfrage an die Web-API.
2. Die Web-API speichert Informationen zur Anforderung in einer Tabelle, die im Tabellenspeicher oder im Microsoft Azure Cache vorgehalten wird, und generiert einen eindeutigen Schlüssel für diesen Eintrag, z. B. in Form einer GUID.
3. Von der Web-API wird die Verarbeitung als separate Aufgabe initiiert. Die Web-API zeichnet den Status der Aufgabe in der Tabelle als *Running* auf.
4. Die Web-API gibt eine Antwortnachricht mit dem HTTP-Statuscode 202 (Accepted) und der GUID des Tabelleneintrags im Text der Nachricht zurück.
5. Wenn die Aufgabe abgeschlossen ist, speichert die Web-API die Ergebnisse in der Tabelle und legt den Status der Aufgabe auf *Complete* fest. Beachten Sie Folgendes: Wenn die Durchführung der Aufgabe nicht erfolgreich ist, kann die Web-API auch Informationen zum Fehler speichern und den Status auf *Failed* festlegen.
6. Während der Ausführung der Aufgabe kann der Client mit seiner eigenen Verarbeitung fortfahren. Er kann regelmäßig eine Anforderung an den URI */polling/{guid}* senden. Hierbei steht *{guid}* für die GUID, die von der Web-API in der 202-Antwortnachricht zurückgegeben wird.
7. Die Web-API am URI */polling/{guid}* fragt den Status der entsprechenden Aufgabe in der Tabelle ab und gibt eine Antwortnachricht mit dem HTTP-Statuscode 200 (OK) und dem Status zurück (*Running*, *Complete* oder *Failed*). Wenn die Aufgabe abgeschlossen ist oder nicht erfolgreich war, kann die Antwortnachricht auch die Ergebnisse der Verarbeitung oder Informationen enthalten, die zur Ursache des Fehlers verfügbar sind.

Optionen zur Implementierung von Benachrichtigungen:

- Verwenden Sie ein Azure Notification Hub, um asynchrone Antworten per Pushvorgang an Clientanwendungen zu übertragen. Weitere Informationen finden Sie unter [Azure Notification Hubs – Benachrichtigen von Benutzern über .NET-Back-End](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/).
- Verwenden Sie das Comet-Modell, um eine dauerhafte Netzwerkverbindung zwischen dem Client und dem Hostserver der Web-API aufrechtzuerhalten. Nutzen Sie diese Verbindung, um Nachrichten vom Server zurück an den Client zu übertragen. Eine Beispiellösung ist im Artikel [Erstellen einer einfachen Comet-Anwendung in Microsoft .NET Framework](https://msdn.microsoft.com/magazine/jj891053.aspx) des MSDN-Magazins enthalten.
- Verwenden Sie SignalR, um Daten über eine dauerhafte Netzwerkverbindung in Echtzeit vom Webserver auf den Client zu übertragen. SignalR ist für ASP.NET-Webanwendungen als NuGet-Paket verfügbar. Weitere Informationen finden Sie auf der Website [ASP.NET SignalR](http://signalr.net/).

### <a name="ensure-that-each-request-is-stateless"></a>Sicherstellen, dass jede Anforderung zustandslos ist

Jede Anforderung sollte als atomarisch angesehen werden. Es sollten keine Abhängigkeiten zwischen der Anforderung einer Clientanwendung und allen nachfolgenden Anforderungen bestehen, die von demselben Client übermittelt werden. Dieser Ansatz ist in Bezug auf die Skalierbarkeit hilfreich. Instanzen des Webdiensts können auf einer Reihe von Servern bereitgestellt werden. Clientanforderungen können an all diese Instanzen geleitet werden, und die Ergebnisse sollten immer gleich sein. Die Verfügbarkeit wird aus einem ähnlichen Grund verbessert. Wenn ein Webserver ausfällt, können Anforderungen an eine andere Instanz weitergeleitet werden (per Azure Traffic Manager), während der Server neu gestartet wird. Dies hat keine negativen Auswirkungen auf Clientanwendungen.

### <a name="track-clients-and-implement-throttling-to-reduce-the-chances-of-dos-attacks"></a>Nachverfolgen von Clients und Implementieren der Drosselung, um die Gefahr von DoS-Angriffen zu verringern

Wenn ein bestimmter Client innerhalb eines bestimmten Zeitraums eine große Zahl von Anforderungen sendet, kann dies zu einer Monopolisierung des Diensts führen und die Leistung anderer Clients beeinträchtigen. Zur Beseitigung dieses Problems kann eine Web-API die Aufrufe von Clientanwendungen überwachen, indem entweder die IP-Adresse aller eingehenden Anforderungen nachverfolgt wird oder indem jeder authentifizierte Zugriff protokolliert wird. Sie können diese Informationen nutzen, um den Zugriff auf Ressourcen zu beschränken. Wenn ein Client einen festgelegten Grenzwert überschreitet, kann die Web-API eine Antwortnachricht mit dem Status 503 (Service Unavailable) zurückgeben und einen Retry-After-Header einfügen. Mit diesem Header wird angegeben, wann der Client die nächste Anforderung senden kann, ohne dass diese abgelehnt wird. Diese Strategie kann die Gefahr eines DoS-Angriffs (Denial Of Service) über Clients in Verbindung mit einem Zusammenbruch des Systems verringern.

### <a name="manage-persistent-http-connections-carefully"></a>Sorgfältiges Verwalten von dauerhaften HTTP-Verbindungen

Das HTTP-Protokoll unterstützt dauerhafte HTTP-Verbindungen, sofern sie verfügbar sind. Mit der HTTP 1.0-Spezifikation wurde der Connection:Keep-Alive-Header hinzugefügt. Hiermit kann eine Clientanwendung für den Server angeben, dass dieselbe Verbindung zum Senden nachfolgender Anforderungen genutzt werden kann, anstatt neue Verbindungen zu öffnen. Die Verbindung wird automatisch geschlossen, wenn der Client die Verbindung nicht innerhalb eines vom Host definierten Zeitraums wiederverwendet. Dies ist das Standardverhalten in HTTP 1.1, das von Azure-Diensten verwendet wird. Es ist also nicht erforderlich, Keep-Alive-Header in Nachrichten einzufügen.

Das Offenhalten einer Verbindung kann die Reaktionsfähigkeit verbessern, indem die Latenz und Netzwerküberlastung reduziert wird. Aber es kann sich negativ auf die Skalierbarkeit auswirken, wenn nicht benötigte Verbindungen länger als erforderlich geöffnet bleiben, da das gleichzeitige Herstellen von Verbindungen für andere Clients eingeschränkt wird. Außerdem kann es sich auf die Akkulaufzeit auswirken, wenn die Clientanwendung auf einem Mobilgerät ausgeführt wird. Falls die Anwendung nur gelegentlich Anforderungen an den Server sendet, kann das Aufrechterhalten einer offenen Verbindung dazu führen, dass der Akku schneller leer ist. Um sicherzustellen, dass eine Verbindung unter HTTP 1.1 nicht dauerhaft eingerichtet wird, kann der Client einen Connection:Close-Header in Nachrichten einfügen, um das Standardverhalten außer Kraft zu setzen. Wenn ein Server eine sehr große Anzahl von Clients behandelt, kann er einen Connection:Close-Header in Antwortnachrichten einfügen, mit dem die Verbindung geschlossen wird und Serverressourcen gespart werden.

> [!NOTE]
> Dauerhafte HTTP-Verbindungen sind ein rein optionales Feature zum Reduzieren des Netzwerkaufwands, der mit dem wiederholten Einrichten eines Kommunikationskanals verbunden ist. Weder die Web-API noch die Clientanwendung sollten davon abhängig sein, dass eine dauerhafte HTTP-Verbindung verfügbar ist. Nutzen Sie keine dauerhaften HTTP-Verbindungen, um Benachrichtigungssysteme im Comet-Stil zu implementieren. Stattdessen sollten Sie Sockets (oder WebSockets, falls verfügbar) auf TCP-Ebene verwenden. Beachten Sie außerdem Folgendes: Der Nutzen von Keep-Alive-Headern ist eingeschränkt, wenn eine Clientanwendung mit einem Server über einen Proxy kommuniziert. Nur die Verbindung mit dem Client und dem Proxy ist dauerhafter Art.
>
>

## <a name="considerations-for-publishing-and-managing-a-web-api"></a>Aspekte zur Veröffentlichung und Verwaltung einer Web-API
Die Web-API muss in einer Hostumgebung bereitgestellt werden, um sie für Clientanwendungen verfügbar zu machen. Bei dieser Umgebung handelt es sich normalerweise um einen Webserver, aber es kann auch eine andere Art von Hostprozess sein. Berücksichtigen Sie beim Veröffentlichen einer Web-API die folgenden Punkte:

* Alle Anforderungen müssen authentifiziert und autorisiert werden, und die Zugriffssteuerung muss im angemessenen Umfang durchgesetzt werden.
* Eine kommerzielle Web-API kann in Bezug auf die Reaktionszeiten verschiedenen Qualitätsgarantien unterliegen. Es ist wichtig sicherzustellen, dass die Hostumgebung skalierbar ist, wenn die Auslastung im Laufe der Zeit erheblich variieren kann.
* Es kann erforderlich sein, Anforderungen aus Gründen der Monetarisierung zu messen.
* Es kann erforderlich sein, den Fluss des Datenverkehrs an die Web-API zu regulieren und für bestimmte Clients, die ihr Kontingent ausgeschöpft haben, eine Drosselung zu implementieren.
* Unter Umständen ist es gesetzlich vorgeschrieben, alle Anforderungen und Antworten zu protokollieren und zu überwachen.
* Zum Sicherstellen der Verfügbarkeit kann es erforderlich sein, die Integrität des Servers zu überwachen, auf dem die Web-API gehostet wird, und ihn bei Bedarf neu zu starten.

Es ist hilfreich, diese Probleme von den technischen Problemen in Bezug auf die Implementierung der Web-API abkoppeln zu können. Erwägen Sie deshalb das Erstellen einer [Fassade](http://en.wikipedia.org/wiki/Facade_pattern), die als separater Prozess ausgeführt wird und Anforderungen an die Web-API umleitet. Mit dieser Fassade können die Verwaltungsvorgänge bereitgestellt und geprüfte Anforderungen an die Web-API weitergeleitet werden. Die Nutzung einer Fassade ist ferner mit vielen funktionellen Vorteilen verbunden, z. B.:

* Sie dient als Integrationspunkt für mehrere Web-APIs.
* Sie ermöglicht das Transformieren von Nachrichten und das Übersetzen von Kommunikationsprotokollen für Clients, die mit unterschiedlicher Technologie erstellt wurden.
* Sie ermöglicht das Zwischenspeichern von Anforderungen und Antworten, um die Auslastung des Servers zu reduzieren, auf dem die Web-API gehostet wird.

## <a name="considerations-for-testing-a-web-api"></a>Aspekte zum Testen einer Web-API
Eine Web-API sollte so gründlich wie jede andere Software getestet werden. Erwägen Sie die Erstellung von Komponententests zur Überprüfung der Funktionalität einer Web-API, die über zusätzliche weitere Anforderungen verfügt, um die richtige Funktionsweise sicherzustellen. Achten Sie besonders auf die folgenden Aspekte:

* Testen Sie alle Routen, um zu überprüfen, ob dabei die richtigen Vorgänge aufgerufen werden. Achten Sie besonders darauf, ob der HTTP-Statuscode 405 (Method Not Allowed) unerwartet zurückgegeben wird. Dies kann auf eine fehlende Übereinstimmung zwischen einer Route und den HTTP-Methoden (GET, POST, PUT, DELETE) hindeuten, die auf dieser Route genutzt werden können.

    Senden Sie HTTP-Anforderungen an Routen, die dafür keine Unterstützung aufweisen, z. B. per Übermittlung einer POST-Anforderung an eine bestimmte Ressource (POST-Anforderungen sollten nur an Ressourcenauflistungen gesendet werden). In diesen Fällen *muss* die einzig gültige Antwort der Statuscode 405 (Not Allowed) sein.
* Vergewissern Sie sich, dass alle Routen richtig geschützt sind und geeignete Authentifizierungs- und Autorisierungsüberprüfungen aufweisen.

  > [!NOTE]
  > Für einige Aspekte der Sicherheit, z. B. die Benutzerauthentifizierung, ist meist nicht die Web-API verantwortlich, sondern die Hostumgebung. Es ist trotzdem erforderlich, Sicherheitstests in den Bereitstellungsprozess einzubinden.
  >
  >
* Testen Sie die Ausnahmebehandlung, die von den einzelnen Vorgängen durchgeführt wird, und stellen Sie sicher, dass eine passende und aussagekräftige HTTP-Antwort zurück an die Clientanwendung übergeben wird.
* Achten Sie darauf, dass Anforderungs- und Antwortnachrichten richtig formatiert sind. Wenn eine HTTP POST-Anforderung beispielsweise die Daten für eine neue Ressource im Format „x-www-form-urlencoded“ enthält, müssen Sie bestätigen, dass der entsprechende Vorgang die Daten richtig analysiert, die Ressourcen erstellt und eine Antwort mit den Details der neuen Ressource zurückgibt, einschließlich des richtigen Location-Headers.
* Überprüfen Sie alle Links und URIs in Antwortnachrichten. Beispielsweise sollte eine HTTP POST-Nachricht den URI der neu erstellten Ressource zurückgeben. Alle HATEOAS-Links müssen gültig sein.

* Stellen Sie sicher, dass jeder Vorgang für unterschiedliche Eingabekombinationen die richtigen Statuscodes zurückgibt. Beispiel:

  * Wenn eine Abfrage erfolgreich ist, sollte der Vorgang den Statuscode 200 (OK) zurückgeben.
  * Wenn eine Ressource nicht gefunden wird, sollte der Vorgang den HTTP-Statuscode 404 (Nicht gefunden) zurückgeben.
  * Wenn der Client eine Anforderung sendet, mit der eine Ressource erfolgreich gelöscht wird, sollte der Statuscode 204 (No Content) lauten.
  * Wenn der Client eine Anforderung sendet, mit der eine neue Ressource erstellt wird, sollte der Statuscode 201 (Created) lauten.

Achten Sie auf unerwartete Antwortstatuscodes im Bereich 5xx. Diese Nachrichten werden normalerweise vom Server gemeldet, um anzugeben, dass eine gültige Anforderung nicht erfüllt werden konnte.

* Testen Sie die unterschiedlichen Anforderungsheaderkombinationen, die von einer Clientanwendung angegeben werden können, und stellen Sie sicher, dass die Web-API in Antwortnachrichten die erwarteten Informationen zurückgibt.
* Testen Sie Abfragezeichenfolgen. Wenn ein Vorgang optionale Parameter verwenden kann (z. B. Paginierungsanforderungen), sollten Sie die unterschiedlichen Kombinationen und die Reihenfolge von Parametern testen.
* Vergewissern Sie sich, dass asynchrone Vorgänge erfolgreich abgeschlossen werden. Wenn die Web-API das Streamen für Anforderungen unterstützt, die große binäre Objekte zurückgeben (z. B. Video oder Audio), sollten Sie sicherstellen, dass Clientanforderungen beim Streamen der Daten nicht blockiert werden. Wenn die Web-API das Abrufen für Datenänderungsvorgänge mit langer Ausführungsdauer implementiert, sollten Sie sicherstellen, dass die Vorgänge ihren Status während des Ablaufs richtig melden.

Außerdem sollten Sie Leistungstests erstellen und ausführen, um zu überprüfen, ob die Web-API auch in Notfällen zufriedenstellend arbeitet. Sie können mit Visual Studio Ultimate ein Projekt zum Testen der Webleistung und Auslastung erstellen. Weitere Informationen finden Sie unter [Run performance tests on an application before a release](https://msdn.microsoft.com/library/dn250793.aspx) (Ausführen von Leistungstests für eine Anwendung vor der Veröffentlichung).

## <a name="publish-and-manage-a-web-api-using-the-azure-api-management-service"></a>Veröffentlichen und Verwalten einer Web-API mit dem Azure API Management-Dienst
Azure stellt den [API Management-Dienst](https://azure.microsoft.com/documentation/services/api-management/) bereit, den Sie zum Veröffentlichen und Verwalten einer Web-API verwenden können. Hiermit können Sie einen Dienst generieren, der für eine oder mehrere Web-APIs als „Fassade“ (Façade) dient. Bei diesem Dienst handelt es sich selbst um einen skalierbaren Webdienst, den Sie mit dem Azure-Verwaltungsportal erstellen und konfigurieren können. Sie können diesen Dienst verwenden, um eine Web-API wie folgt zu veröffentlichen und zu verwalten:

1. Stellen Sie die Web-API auf einer Website, in einem Azure-Clouddienst oder auf einem virtuellen Azure-Computer bereit.
2. Verbinden Sie den API Management-Dienst mit der Web-API. Anforderungen, die an die URL der Verwaltungs-API gesendet werden, werden den URIs in der Web-API zugeordnet. Ein und derselbe API Management-Dienst kann Anforderungen an mehr als eine Web-API weiterleiten. So können Sie mehrere Web-APIs zu einem zentralen Management-Dienst zusammenfassen. Außerdem kann von mehr als einem API Management-Dienst auf dieselbe Web-API verwiesen werden, wenn Sie die Funktionalität, die für unterschiedliche Anwendungen verfügbar ist, einschränken oder partitionieren müssen.

   > [!NOTE]
   > Die URIs in HATEOAS-Links, die als Teil der Antwort für HTTP GET-Anforderungen generiert werden, sollten auf die URL des API Management-Diensts verweisen, und nicht auf den Webserver, auf dem die Web-API gehostet wird.
   >
   >
3. Geben Sie für jede Web-API die HTTP-Vorgänge an, die von der Web-API zusammen mit den optionalen Parametern, die ein Vorgang als Eingabe verwenden kann, verfügbar gemacht werden. Sie können auch konfigurieren, ob der API Management-Dienst die von der Web-API empfangene Antwort zwischenspeichern soll, um wiederholte Anforderungen für dieselben Daten zu optimieren. Zeichnen Sie die Details der HTTP-Antworten auf, die von den einzelnen Vorgängen generiert werden können. Diese Informationen werden zum Generieren der Dokumentation für Entwickler verwendet. Es ist also wichtig, dass sie richtig und vollständig sind.

   Sie können Vorgänge entweder manuell definieren, indem Sie die Assistenten im Azure-Verwaltungsportal verwenden, oder Sie können sie aus einer Datei importieren, in der die Definitionen im WADL- oder Swagger-Format enthalten sind.
4. Konfigurieren Sie die Sicherheitseinstellungen zwischen dem API Management-Dienst und dem Webserver, auf dem die Web-API gehostet wird. Der API Management-Dienst unterstützt derzeit die Standardauthentifizierung und die wechselseitige Authentifizierung mit Zertifikaten sowie die OAuth 2.0-Benutzerautorisierung.
5. Erstellen Sie ein Produkt. Ein Produkt ist die Einheit der Veröffentlichung. Sie fügen die Web-APIs, die Sie zuvor mit dem Management-Dienst verbunden haben, dem Produkt hinzu. Wenn das Produkt veröffentlicht wird, werden die Web-APIs für Entwickler verfügbar gemacht.

   > [!NOTE]
   > Vor dem Veröffentlichen eines Produkts können Sie auch Benutzergruppen definieren, die Zugriff auf das Produkt haben, und diesen Gruppen Benutzer hinzufügen. So haben Sie die Kontrolle darüber, welche Entwickler und Anwendungen die Web-API verwenden können. Wenn eine Web-API genehmigt werden muss, müssen Entwickler eine Anforderung an den Produktadministrator senden, bevor sie Zugriff erhalten können. Der Administrator kann dem Entwickler den Zugriff gewähren oder verweigern. Außerdem können vorhandene Entwickler blockiert werden, wenn sich die Umstände ändern.
   >
   >
6. Konfigurieren Sie Richtlinien für jede Web-API. Mit Richtlinien werden beispielsweise folgende Aspekte geregelt: ob domänenübergreifende Aufrufe zulässig sind, wie Clients authentifiziert werden, ob zwischen den Datenformaten XML und JSON transparent konvertiert werden soll, ob Aufrufe für einen bestimmten IP-Bereich beschränkt werden sollen, Verwendungskontingente und ob die Aufrufrate begrenzt werden soll. Richtlinien können global über das gesamte Produkt hinweg, für eine einzelne Web-API in einem Produkt oder für einzelne Vorgänge in einer Web-API angewendet werden.

Weitere Informationen finden Sie in der [Dokumentation zu API Management](/azure/api-management/). 

> [!TIP]
> Azure stellt den Azure Traffic Manager bereit, mit dem Sie das Failover und den Lastenausgleich implementieren und die Latenz über mehrere Instanzen einer Website hinweg, die an unterschiedlichen geografischen Orten gehostet wird, reduzieren können. Sie können den Azure Traffic Manager zusammen mit dem API Management-Dienst verwenden. Der API Management-Dienst kann Anforderungen über den Azure Traffic Manager an die Instanzen einer Website weiterleiten.  Weitere Informationen finden Sie unter [Traffic Manager-Methoden für das Datenverkehrsrouting](/azure/traffic-manager/traffic-manager-routing-methods/).
>
> Wenn Sie benutzerdefinierte DNS-Namen für Ihre Websites verwenden, sollten Sie in dieser Struktur den richtigen CNAME-Eintrag für jede Website konfigurieren, damit jeweils auf den DNS-Namen der Azure Traffic Manager-Website verwiesen wird.
>

## <a name="support-developers-building-client-applications"></a>Unterstützen von Entwicklern beim Erstellen von Clientanwendungen
Entwickler, die Clientanwendungen erstellen, benötigen normalerweise Informationen dazu, wie sie auf die Web-API zugreifen können. Außerdem benötigen sie Dokumentation zu den Parametern, Datentypen, Rückgabetypen und Rückgabecodes, mit denen die verschiedenen Anforderungen und Antworten zwischen dem Webdienst und der Clientanwendung beschrieben werden.

### <a name="document-the-rest-operations-for-a-web-api"></a>Dokumentieren der REST-Vorgänge für eine Web-API
Der Azure API Management-Dienst enthält ein Entwicklerportal, mit dem die von einer Web-API verfügbar gemachten REST-Vorgänge beschrieben werden. Wenn ein Produkt veröffentlicht wird, wird es in diesem Portal angezeigt. Entwickler können sich über dieses Portal für den Zugriff registrieren. Der Administrator kann die Anforderung dann genehmigen oder ablehnen. Wenn Entwickler die Genehmigung erhalten, wird ihnen ein Abonnementschlüssel zugewiesen. Dieser wird zum Authentifizieren der Aufrufe von Clientanwendungen verwendet, die von den Entwicklern entwickelt werden. Dieser Schlüssel muss bei jedem Web-API-Aufruf bereitgestellt werden. Andernfalls wird der Aufruf abgelehnt.

Außerdem bietet das Portal Folgendes:

* Dokumentation zum Produkt mit den verfügbaren Vorgängen, den erforderlichen Parametern und den unterschiedlichen Antworten, die zurückgegeben werden können. Beachten Sie, dass diese Informationen anhand der Details generiert werden, die im Abschnitt „Veröffentlichen einer Web-API mit dem Microsoft Azure API Management-Dienst“ in Schritt 3 der Liste enthalten sind.
* Codeausschnitte, die das Aufrufen von Vorgängen für mehrere Sprachen veranschaulichen, z. B. JavaScript, C#, Java, Ruby, Python und PHP.
* Eine Entwicklerkonsole, über die Entwickler eine HTTP-Anforderung zum Testen der Vorgänge im Produkt und zum Anzeigen der Ergebnisse senden können.
* Eine Seite, auf der Entwickler Fehler oder aufgetretene Probleme melden können.

Über das Azure-Verwaltungsportal können Sie das Entwicklerportal anpassen und die Formatierung und das Layout ändern, um es an das Branding Ihres Unternehmens anzugleichen.

### <a name="implement-a-client-sdk"></a>Implementieren eines Client-SDK
Das Erstellen einer Clientanwendung, mit der REST-Anforderungen zum Zugreifen auf eine Web-API aufgerufen werden, erfordert das Schreiben einer beträchtlichen Menge an Code. Jede Anforderung muss richtig erstellt und formatiert werden, die Anforderung muss an den Hostserver des Webdiensts gesendet werden, und die Antwort muss analysiert werden, um zu ermitteln, ob die Anforderung erfolgreich war. Außerdem müssen alle zurückgegebenen Daten extrahiert werden. Um die Clientanwendung hiervon abzukoppeln, können Sie ein SDK bereitstellen, das die REST-Schnittstelle umschließt und diese Low-Level-Details in einem funktionelleren Methodensatz abstrahiert. Eine Clientanwendung verwendet diese Methoden, mit denen Aufrufe transparent in REST-Anforderungen konvertiert werden. Anschließend werden die Antworten zurück in Methodenrückgabewerte konvertiert. Dies ist ein gängiges Verfahren, das von vielen Diensten implementiert wird, einschließlich des Azure-SDK.

Die Erstellung eines clientseitigen SDK ist ein aufwendiges Unterfangen, da es konsistent implementiert und sorgfältig getestet werden muss. Ein Großteil dieses Prozesses kann aber „mechanisiert“ werden, und viele Anbieter stellen Tools bereit, mit denen viele Aufgaben automatisiert werden können.

## <a name="monitoring-a-web-api"></a>Überwachen einer Web-API
Je nach Art der Veröffentlichung und Bereitstellung Ihrer Web-API können Sie die Web-API direkt überwachen, oder Sie können Informationen zur Verwendung und Integrität sammeln, indem Sie den Datenverkehr analysieren, der den API Management-Dienst durchläuft.

### <a name="monitoring-a-web-api-directly"></a>Direktes Überwachen einer Web-API
Wenn Sie Ihre Web-API mit der ASP.NET-Web-API-Vorlage (entweder als Web-API-Projekt oder Webrolle in einem Azure-Clouddienst) und Visual Studio 2013 implementiert haben, können Sie per ASP.NET Application Insights Daten zur Verfügbarkeit, Leistung und Verwendung sammeln. Application Insights ist ein Paket, mit dem Informationen zu Anforderungen und Antworten transparent nachverfolgt und aufgezeichnet werden, wenn die Web-API in der Cloud bereitgestellt wird. Nachdem das Paket installiert und konfiguriert wurde, müssen Sie in Ihrer Web-API zur Nutzung keinen Code ändern. Wenn Sie die Web-API auf einer Azure-Website bereitstellen, wird der gesamte Datenverkehr untersucht, und die folgenden statistischen Daten werden gesammelt:

* Reaktionszeit des Servers
* Anzahl von Serveranforderungen und die Details jeder Anforderung
* Langsamste Anforderungen in Bezug auf die durchschnittliche Reaktionszeit
* Details nicht erfolgreicher Anforderungen
* Anzahl von Sitzungen, die von unterschiedlichen Browsern und Benutzer-Agents initiiert werden
* Am häufigsten angezeigte Seiten (vor allem nützlich für Webanwendungen und nicht für Web-APIs)
* Unterschiedliche Benutzerrollen, mit denen auf die Web-API zugegriffen wird

Sie können diese Daten im Azure-Verwaltungsportal in Echtzeit anzeigen. Außerdem können Sie WebTests erstellen, mit denen die Integrität der Web-API überwacht wird. Ein WebTest sendet eine regelmäßige Anforderung an einen angegebenen URI in der Web-API und erfasst die Antwort. Sie können die Definition einer erfolgreichen Antwort angeben (z. B. HTTP-Statuscode 200). Und wenn die Anforderung diese Antwort nicht zurückgibt, können Sie vorgeben, dass eine Warnung an einen Administrator gesendet wird. Bei Bedarf kann der Administrator den Server neu starten, auf dem die Web-API gehostet wird, wenn er ausgefallen ist.

Weitere Informationen finden Sie unter [Einrichten von Application Insights für Ihre ASP.NET-Website](/azure/application-insights/app-insights-asp-net/).

### <a name="monitoring-a-web-api-through-the-api-management-service"></a>Überwachen einer Web-API per API Management-Dienst
Wenn Sie Ihre Web-API mit dem API Management-Dienst veröffentlicht haben, enthält die Seite „API Management“ im Azure-Verwaltungsportal ein Dashboard, auf dem Sie die allgemeine Leistung des Diensts anzeigen können. Auf der Seite „Analytics“ können Sie auf die Details zur Verwendung des Produkts zugreifen. Diese Seite enthält die folgenden Registerkarten:

* **Verwendung**: Diese Registerkarte enthält Informationen zur Anzahl der durchgeführten API-Aufrufe und zur Bandbreite, die zum Behandeln dieser Aufrufe im Laufe der Zeit verwendet wurde. Sie können die Verwendungsdetails nach Produkt, API und Vorgang filtern.
* **Integrität**: Auf dieser Registerkarte können Sie das Ergebnis von API-Anforderungen (zurückgegebenen HTTP-Statuscodes), die Effektivität der Richtlinie für die Zwischenspeicherung, die API-Reaktionszeit und die Reaktionszeit des Diensts anzeigen. Sie können auch die Integritätsdaten nach Produkt, API und Vorgang filtern.
* **Aktivität**: Diese Registerkarte enthält eine Textzusammenfassung der Anzahl von erfolgreichen Aufrufen, fehlgeschlagenen Aufrufen, blockierten Aufrufen, der durchschnittlichen Reaktionszeit und Reaktionszeiten für Produkt, Web-API und Vorgang. Auf dieser Seite ist auch die Anzahl von Aufrufen angegeben, die von den einzelnen Entwicklern durchgeführt wurden.
* **Auf einen Blick**: Auf dieser Registerkarte wird eine Zusammenfassung der Leistungsdaten angezeigt, z. B. die Entwickler mit den meisten API-Aufrufen und die Produkte, Web-APIs und Vorgänge, von denen diese Aufrufe empfangen wurden.

Anhand dieser Informationen können Sie bestimmen, ob eine bestimmte Web-API oder ein Vorgang einen Engpass verursacht, und bei Bedarf die Hostumgebung skalieren und weitere Server hinzufügen. Sie können auch prüfen, ob eine oder mehrere Anwendungen eine unangemessen hohe Menge an Ressourcen verbrauchen, und entsprechende Richtlinien anwenden, um Kontingente festzulegen und die Aufrufraten zu beschränken.

> [!NOTE]
> Sie können die Details für ein veröffentlichtes Produkt ändern. Die Änderungen werden dann sofort angewendet. Beispielsweise können Sie einen Vorgang einer Web-API hinzufügen oder daraus entfernen, ohne dass Sie hierfür das Produkt neu veröffentlichen müssen, in dem die Web-API enthalten ist.
>
>

## <a name="more-information"></a>Weitere Informationen
* [ASP.NET-Web-API-OData](http://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api) enthält Beispiele und weitere Informationen zur Implementierung einer OData-Web-API per ASP.NET.
* Unter [Einführung der Batchunterstützung für die Web-API und Web-API-OData](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx) wird beschrieben, wie Sie Batchvorgänge in einer Web-API mit OData implementieren.
* [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) (Muster der Idempotenz) im Blog von Jonathan Oliver enthält eine Übersicht über die Idempotenz und ihre Verbindung mit Datenverwaltungsvorgängen.
* [Statuscodedefinitionen](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) auf der W3C-Website enthält eine vollständige Liste mit HTTP-Statuscodes und den dazugehörigen Beschreibungen.
* Unter [Ausführen von Hintergrundaufgaben mit WebJobs in Azure App Service](/azure/app-service-web/web-sites-create-web-jobs/) finden Sie Informationen und Beispiele zur Verwendung von WebJobs für die Durchführung von Hintergrundvorgängen.
* Unter [Azure Notification Hubs – Benachrichtigen von Benutzern](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/) wird beschrieben, wie Sie einen Azure Notification Hub zum Übertragen von asynchronen Antworten per Pushvorgang auf Clientanwendungen verwenden.
* Unter [API Management](https://azure.microsoft.com/services/api-management/) wird beschrieben, wie Sie ein Produkt veröffentlichen, das kontrollierten und sicheren Zugriff auf eine Web-API ermöglicht.
* Unter [Azure API-Verwaltung für REST-API-Referenz](https://msdn.microsoft.com/library/azure/dn776326.aspx) wird beschrieben, wie Sie die API Management-REST-API zum Erstellen von benutzerdefinierten Verwaltungsanwendungen verwenden.
* Unter [Traffic Manager-Routingmethoden](/azure/traffic-manager/traffic-manager-routing-methods/) wird zusammengefasst, wie Azure Traffic Manager verwendet werden kann, um für mehrere Instanzen einer Website, auf der eine Web-API gehostet wird, den Lastenausgleich für Anforderungen durchzuführen.
* Unter [Einrichten von Application Insights für ASP.NET](/azure/application-insights/app-insights-asp-net/) werden ausführliche Informationen zum Installieren und Konfigurieren von Application Insights in einem ASP.NET-Web-API-Projekt bereitgestellt.


<!-- links -->

[api-design]: ./api-design.md
