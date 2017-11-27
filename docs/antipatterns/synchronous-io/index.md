---
title: "Antimuster „Synchrone E/A-Vorgänge“"
description: "Ein Blockieren des aufrufenden Threads, während ein E/A-Vorgang abgeschlossen wird, kann die Leistung senken und sich auf die vertikale Skalierbarkeit auswirken."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: d5b3635565c6b71ef7716f54ee8cccc76093c3a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="synchronous-io-antipattern"></a>Antimuster „Synchrone E/A-Vorgänge“

Ein Blockieren des aufrufenden Threads, während ein E/A-Vorgang abgeschlossen wird, kann die Leistung senken und sich auf die vertikale Skalierbarkeit auswirken.

## <a name="problem-description"></a>Problembeschreibung

Ein synchroner E/A-Vorgang blockiert den aufrufenden Thread, während der E/A-Vorgang abgeschlossen wird. Der aufrufende Thread wird in einen Wartezustand versetzt und kann in diesem Intervall keine sinnvollen Vorgänge ausführen, wodurch Verarbeitungsressourcen verschwendet werden.

Häufige Beispiele für E/A-Vorgänge sind etwa folgende:

- Abrufen oder dauerhaftes Speichern von Daten in einer Datenbank oder einer anderen Art persistentem Speicher
- Senden einer Anforderung an einen Webdienst
- Übermitteln einer Nachricht oder Abrufen einer Nachricht aus einer Warteschlange
- Schreiben in eine lokale Datei oder Lesen aus einer lokalen Datei

Dieses Antimuster tritt üblicherweise aus folgenden Gründen auf:

- Es ist anscheinend die intuitivste Art, einen Vorgang auszuführen. 
- Die Anwendung erfordert eine Antwort aus einer Anforderung.
- Die Anwendung verwendet eine Bibliothek, die nur synchrone Methoden für E/A-Vorgänge bereitstellt. 
- Eine externe Bibliothek führt intern synchrone E/A-Vorgänge aus. Ein einziger synchroner E/A-Aufruf kann eine vollständige Aufrufkette blockieren.

Der folgende Code lädt eine Datei in einen Azure-Blobspeicher hoch. Es gibt zwei Stellen, an denen der Code blockiert wird, weil er auf die synchrone E/A-Verarbeitung wartet: die `CreateIfNotExists`-Methode und die `UploadFromStream`-Methode.

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

container.CreateIfNotExists();
var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    blockBlob.UploadFromStream(fileStream);
}
```

Hier sehen Sie ein Beispiel für das Warten auf die Antwort eines externen Diensts. Die `GetUserProfile`-Methode ruft einen Remotedienst auf, der ein `UserProfile` zurückgibt.

```csharp
public interface IUserProfileService
{
    UserProfile GetUserProfile();
}

public class SyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public SyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is a synchronous method that calls the synchronous GetUserProfile method.
    public UserProfile GetUserProfile()
    {
        return _userProfileService.GetUserProfile();
    }
}
```

Den vollständigen Code für beide Beispiele finden Sie [hier][sample-app].

## <a name="how-to-fix-the-problem"></a>Beheben des Problems

Ersetzen Sie synchrone E/A-Vorgänge durch asynchrone Vorgänge. Dadurch wird der aktuelle Thread wieder frei und kann weiterhin sinnvolle Tasks ausführen, anstatt zu blockieren. So lässt sich die Auslastung der Computeressourcen verbessern. Die asynchrone Ausführung von E/A-Vorgängen ist insbesondere dann effizient, wenn ein unerwarteter Anstieg der Anforderungen von Clientanwendungen verarbeitet werden muss. 

Viele Bibliotheken bieten sowohl synchrone als auch asynchrone Versionen von Methoden. Verwenden Sie nach Möglichkeit die asynchronen Versionen. Hier ist die asynchrone Version des vorherigen Beispiels, das eine Datei in einen Azure-Blobspeicher hochlädt.

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

await container.CreateIfNotExistsAsync();

var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

Der `await`-Operator gibt die Steuerung an die aufrufende Umgebung zurück, während der asynchrone Vorgang ausgeführt wird. Der Code nach dieser Anweisung fungiert als Fortsetzung, die ausgeführt wird, wenn der asynchrone Vorgang abgeschlossen ist.

Ein sorgfältig entworfener Dienst sollte auch asynchrone Vorgänge bereitstellen. Hier sehen Sie eine asynchrone Version des Webdiensts, der Benutzerprofile zurückgibt. Die `GetUserProfileAsync`-Methode benötigt eine asynchrone Version des Benutzerprofildiensts.

```csharp
public interface IUserProfileService
{
    Task<UserProfile> GetUserProfileAsync();
}

public class AsyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public AsyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is an synchronous method that calls the Task based GetUserProfileAsync method.
    public Task<UserProfile> GetUserProfileAsync()
    {
        return _userProfileService.GetUserProfileAsync();
    }
}
```

Bei Bibliotheken, die keine asynchronen Versionen von Vorgängen bereitstellen, ist es unter Umständen möglich, asynchrone Wrapper für ausgewählte synchrone Methoden zu erstellen. Gehen Sie dabei sehr vorsichtig vor. Mit diesem Ansatz lässt sich zwar möglicherweise die Reaktionsfähigkeit in dem Thread verbessern, der den asynchronen Wrapper aufruft, aber es werden auch mehr Ressourcen verbraucht. Möglicherweise wird ein gesonderter Thread erstellt, und durch die Synchronisierung der von diesem Thread ausgeführten Verarbeitung entsteht Overhead. In diesem Blogbeitrag werden einige Nachteile erläutert: [Should I expose asynchronous wrappers for synchronous methods?][async-wrappers] (Sollen asynchrone Wrapper für synchrone Methoden verfügbar gemacht werden?)

Hier sehen Sie ein Beispiel eines asynchronen Wrappers um eine synchrone Methode.

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

Jetzt kann der aufrufende Code im Wrapper warten:

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a>Überlegungen

- E/A-Vorgänge, die voraussichtlich sehr kurzlebig sind und keine Konflikte verursachen, können als synchrone Vorgänge leistungsfähig sein. Ein Beispiel ist das Lesen kleiner Dateien auf einem SSD-Laufwerk. Der Overhead durch das Umleiten eines Tasks in einen anderen Thread und das Synchronisieren mit diesem Thread nach Abschluss des Tasks kann die Vorteile der asynchronen E/A-Verarbeitung überwiegen. Diese Fälle sind jedoch relativ selten, und die meisten E/A-Vorgänge lassen sich asynchron ausführen.

- Durch Verbesserung der E/A-Leistung werden möglicherweise andere Teile des Systems zu Engpässen. Dadurch, dass die Blockierung von Threads aufgehoben wird, kann eine höhere Anzahl gleichzeitiger Anforderungen an gemeinsam genutzte Ressourcen entstehen, wodurch wiederum möglicherweise nicht genügend Ressourcen verfügbar sind oder Ressourcen gedrosselt werden müssen. Wenn dies zu einem Problem wird, müssen Sie möglicherweise Ihre Webserver horizontal hochskalieren oder Datenspeicher partitionieren, um Konflikte zu reduzieren.

## <a name="how-to-detect-the-problem"></a>Erkennen des Problems

Aus Benutzersicht reagiert die Anwendung nicht. Bei der Anwendung können Timeoutausnahmen auftreten. In diesen Fällen können auch HTTP 500-Fehler (interner Server) zurückgegeben werden. Auf dem Server werden eingehende Clientanforderungen blockiert, bis ein Thread verfügbar wird, sodass die Anforderungswarteschlangen zu lang werden, was sich in HTTP 503-Fehlern (Dienst nicht verfügbar) manifestiert.

Sie können die folgenden Schritte ausführen, um das Problem zu identifizieren:

1. Überwachen Sie das Produktionssystem, und ermitteln Sie, ob blockierte Workerthreads den Durchsatz einschränken.

2. Wenn Anforderungen aufgrund einer zu geringen Anzahl von Threads blockiert werden, überprüfen Sie die Anwendung, um zu ermitteln, welche Vorgänge synchrone E/A-Vorgänge ausführen.

3. Führen Sie kontrollierte Auslastungstests jedes Vorgangs durch, der synchrone E/A-Vorgänge ausführt, um herauszufinden, ob diese Vorgänge die Systemleistung beeinträchtigen.

## <a name="example-diagnosis"></a>Beispieldiagnose

In den folgenden Abschnitten werden diese Schritte auf die zuvor beschriebene Beispielanwendung angewendet.

### <a name="monitor-web-server-performance"></a>Überwachen der Webserverleistung

Bei Azure-Webanwendungen und -Webrollen lohnt es sich, die Leistung des IIS-Webservers zu überwachen. Achten Sie besonders auf die Länge der Anforderungswarteschlange, um herauszufinden, ob Anforderungen in Zeiträumen hoher Aktivität dadurch blockiert werden, dass sie auf verfügbare Threads warten. Sie können diese Informationen erfassen, indem Sie die Azure-Diagnose aktivieren. Weitere Informationen finden Sie unter:

- [Überwachen von Apps in Azure App Service][web-sites-monitor]
- [Erstellen und Verwenden von Leistungsindikatoren in einer Azure-Anwendung][performance-counters]

Instrumentieren Sie die Anwendung, um festzustellen, wie Anforderungen verarbeitet werden, sobald sie akzeptiert wurden. Durch Nachverfolgen eines Anforderungsflows kann ermittelt werden, ob die Anforderung langsame Aufrufe ausführt und den aktuellen Thread blockiert. Mithilfe eines Threadprofils lassen sich Anforderungen markieren, die blockiert werden.

### <a name="load-test-the-application"></a>Auslastungstest der Anwendung

Das folgende Diagramm zeigt die Leistung der oben gezeigten synchronen `GetUserProfile`-Methode unter verschiedenen Auslastungen mit bis zu 4.000 gleichzeitigen Benutzern. Es handelt sich um eine ASP.NET-Anwendung, die in einer Azure-Clouddienst-Webrolle ausgeführt wird.

![Leistungsdiagramm für die Beispielanwendung mit synchronen E/A-Vorgängen][sync-performance]

Für den synchronen Vorgang wurde eine Wartezeit von 2 Sekunden hartcodiert, um eine synchrone E/A-Verarbeitung zu simulieren, daher liegt die Mindestantwortzeit bei knapp über 2 Sekunden. Wenn die Auslastung ca. 2.500 Benutzer erreicht, stabilisiert sich die durchschnittliche Antwortzeit auf einem bestimmten Niveau, obwohl die Menge an Anforderungen pro Sekunde weiter steigt. Beachten Sie, dass die Skalierung für diese beiden Messungen logarithmisch ist. Die Anzahl von Anforderungen pro Sekunde verdoppelt sich zwischen diesem Punkt und dem Ende des Tests.

In einer isolierten Umgebung ergibt sich aus diesem Test nicht eindeutig, ob die synchrone E/A-Verarbeitung ein Problem darstellt. Bei größerer Auslastung erreicht die Anwendung möglicherweise einen kritischen Punkt, an dem der Webserver Anforderungen nicht mehr rechtzeitig verarbeiten kann, sodass Clientanwendungen Timeoutausnahmen empfangen.

Eingehende Anforderungen werden vom IIS-Webserver in die Warteschlange eingereiht und an einen Thread weitergeleitet, der im ASP.NET-Threadpool ausgeführt wird. Da jeder Vorgang die E/A-Verarbeitung synchron ausführt, wird der Thread blockiert, bis der Vorgang abgeschlossen ist. Wenn die Workload steigt, werden schlussendlich alle ASP.NET-Threads im Threadpool zugewiesen und blockiert. An diesem Punkt müssen alle weiteren eingehenden Anforderungen in der Warteschlange auf einen verfügbaren Thread warten. Wenn die Warteschlange immer länger wird, treten bei Anforderungen Timeouts auf.

### <a name="implement-the-solution-and-verify-the-result"></a>Implementieren der Lösung und Überprüfen des Ergebnisses

Das nächste Diagramm zeigt die Ergebnisse des Auslastungstests für die asynchrone Version des Codes.

![Leistungsdiagramm für die Beispielanwendung mit asynchronen E/A-Vorgängen][async-performance]

Der Durchsatz ist wesentlich höher. Im gleichen Zeitraum wie beim vorherigen Test verarbeitet das System erfolgreich nahezu das 10-Fache des Durchsatzes (gemessen in Anforderungen pro Sekunde). Darüber hinaus ist die durchschnittliche Antwortzeit relativ konstant und bleibt etwa 25-mal kürzer als im vorherigen Test.


[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO


[async-wrappers]: http://blogs.msdn.com/b/pfxteam/archive/2012/03/24/10287244.aspx
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg



