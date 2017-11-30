---
title: Cachefremd
description: Daten bei Bedarf aus einem Datenspeicher in einen Cache laden
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: e0a6a91fda6ea43236f6eea552f7b8f8d31160ad
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="cache-aside-pattern"></a>Cachefremdes Muster

[!INCLUDE [header](../_includes/header.md)]

Laden Sie Daten bei Bedarf aus einem Datenspeicher in einen Cache. Dies kann die Leistung verbessern und trägt zudem dazu bei, die Konsistenz zwischen den Daten im Cache und den Daten im zugrunde liegenden Datenspeicher aufrechtzuerhalten.

## <a name="context-and-problem"></a>Kontext und Problem

Anwendungen verwenden einen Cache, um wiederholte Zugriffe auf Informationen in einem Datenspeicher zu verbessern. Allerdings kann nicht erwartet werden, dass zwischengespeicherte Daten immer vollständig konsistent mit den Daten im Datenspeicher sind. Anwendungen sollten eine Strategie implementieren, mit der sichergestellt wird, dass die Daten im Cache so aktuell wie möglich sind. Sie sollten jedoch auch Situationen erkennen und handhaben können, die auftreten, wenn die Daten im Cache veraltet sind.

## <a name="solution"></a>Lösung

Viele kommerzielle Cachesysteme bieten Read-Through- und Write-Through-/Write-Behind-Vorgänge. In diesen Systemen ruft eine Anwendung Daten mithilfe von Verweisen auf den Cache ab. Wenn die Daten nicht im Cache enthalten sind, werden sie aus dem Datenspeicher abgerufen und dem Cache hinzugefügt. Änderungen an den im Cache gespeicherten Daten werden automatisch auch in den Datenspeicher zurückgeschrieben.

Bei Caches, die diese Funktionalität nicht bereitstellen, ist es die Aufgabe der Anwendungen, die den Cache verwenden, die Daten zu verwalten.

Eine Anwendung kann die Funktionalität eines Read-Through-Cache durch Implementieren einer Strategie mit dem cachefremden Muster emulieren. Mit dieser Strategie werden Daten bei Bedarf in den Cache geladen. Die Abbildung veranschaulicht die Verwendung des cachefremden Musters zum Speichern von Daten im Cache.

![Verwendung des cachefremden Musters zum Speichern von Daten im Cache](./_images/cache-aside-diagram.png)


Aktualisiert eine Anwendung Informationen, kann sie die Write-Through-Strategie einsetzen, indem sie die Änderung am Datenspeicher vornimmt und das entsprechende Element im Cache ungültig macht.

Wenn das Element das nächste Mal benötigt wird, führt die Strategie mit dem cachefremden Muster dazu, dass die aktualisierten Daten aus dem Datenspeicher abgerufen und wieder zum Cache hinzugefügt werden.

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll: 

**Lebensdauer der zwischengespeicherten Daten**. Viele Caches implementieren eine Ablaufrichtlinie, mit der die Daten ungültig gemacht und aus dem Cache entfernt werden, wenn für einen angegebenen Zeitraum nicht darauf zugegriffen wurde. Damit das cachefremde Muster wirksam ist, stellen Sie sicher, dass die Ablaufrichtlinie zum Zugriffsmuster für Anwendungen passt, die die Daten verwenden. Legen Sie keinen zu kurzen Ablaufzeitraum fest. Dies könnte dazu führen, dass Anwendungen Daten kontinuierlich aus dem Datenspeicher abrufen und dem Cache hinzuzufügen. Legen Sie auch keinen zu langen Ablaufzeitraum fest, damit Sie keine veralteten Daten im Cache haben. Denken Sie daran, dass das Zwischenspeichern für relativ statische Daten oder für Daten, die häufig gelesen werden, am effektivsten ist.

**Entfernen von Daten**. Die meisten Caches haben im Vergleich mit dem Datenspeicher, aus dem die Daten stammen, eine beschränkte Größe, und sie müssen Daten ggf. entfernen. In den meisten Caches werden dann die am längsten nicht mehr verwendeten Elemente entfernt, aber dies kann möglicherweise angepasst werden. Konfigurieren Sie die globale Ablaufeigenschaft und andere Eigenschaften des Cache sowie das Ablaufdatum der einzelnen Element im Cache, um sicherzustellen, dass der Cache kostengünstig ist. Es ist nicht immer angebracht, eine globale Entfernungsrichtlinie auf jedes Element im Cache anzuwenden. Wenn es z.B. mit viel Aufwand verbunden ist, ein Element im Cache aus dem Datenspeicher abzurufen, kann es von Vorteil sein, dieses Element im Cache zu belassen und stattdessen häufiger verwendete, jedoch weniger aufwändige Elemente zu löschen.

**Vorbereiten des Cache**. Viele Lösungen füllen den Cache vorab mit den Daten auf, die eine Anwendung wahrscheinlich als Teil der Verarbeitung beim Starten benötigt. Das cachefremde Muster kann dennoch nützlich sein, wenn einige dieser Daten abgelaufen sind oder entfernt werden.

**Konsistenz**. Durch Implementieren des cachefremden Musters ist die Konsistenz zwischen dem Datenspeicher und dem Cache nicht garantiert. Ein Element im Datenspeicher kann jedoch jederzeit von einem externen Prozess geändert werden, und diese Änderung wird möglicherweise erst im Cache wiedergegeben, wenn das Element das nächste Mal geladen wird. In einem System, das Daten über Datenspeicher repliziert, kann dies ein ernsthaftes Problem werden, wenn die Synchronisierung häufig auftritt.

**Lokales (speicherinternes) Zwischenspeichern**. Ein Cache kann für eine Anwendungsinstanz lokal und im Speicher gespeichert sein. Das cachefremde Muster kann in dieser Umgebung nützlich sein, wenn eine Anwendung wiederholt auf die gleichen Daten zugreift. Allerdings ist ein lokaler Cache privat. Daher können verschiedene Anwendungsinstanzen jeweils über eine Kopie der gleichen zwischengespeicherten Daten verfügen. Diese Daten können schnell zwischen Caches inkonsistent werden, sodass es möglicherweise erforderlich ist, dass Daten in einem privaten Cache ablaufen und häufiger aktualisiert werden. In diesen Szenarien sollten Sie die Verwendung eines freigegebenen oder verteilten Mechanismus zum Zwischenspeichern prüfen.

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

Verwenden Sie dieses Muster in folgenden Fällen:

- Ein Cache stellt keine nativen Read-Through- und Write-Through-Vorgänge bereit.
- Der Ressourcenbedarf ist nicht vorhersehbar. Mit diesem Muster können Anwendungen Daten bei Bedarf laden. Es werden im Voraus keine Annahmen darüber getroffen, welche Daten eine Anwendung benötigt.

Dieses Muster ist in folgenden Fällen möglicherweise nicht geeignet:

- Wenn das zwischengespeicherte Dataset statisch ist. Wenn die Daten in den verfügbaren Cachespeicher passen, bereiten Sie den Cache beim Start mit den Daten vor, und wenden Sie eine Richtlinie an, die verhindert, dass die Daten ablaufen.
- Zum Zwischenspeichern von Sitzungszustandsinformationen in einer Webanwendung, die in einer Webfarm gehostet wird. In dieser Umgebung sollten Sie vermeiden, dass Abhängigkeiten basierend auf der Client/Server-Affinität entstehen.

## <a name="example"></a>Beispiel

In Microsoft Azure können Sie Azure Redis Cache verwenden, um einen verteilten Cache zu erstellen, der von mehreren Instanzen einer Anwendung gemeinsam genutzt werden kann. 

Rufen Sie die statische `Connect`-Methode auf, und übergeben Sie die Verbindungszeichenfolge, um eine Verbindung mit einer Azure Redis Cache-Instanz herzustellen. Die Methode gibt ein `ConnectionMultiplexer`-Element zurück, das die Verbindung darstellt. Ein Ansatz zur Freigabe einer `ConnectionMultiplexer` -Instanz in Ihrer Anwendung ist das Verwenden einer statischen Eigenschaft, die wie im folgenden Beispiel eine verbundene Instanz zurückgibt. Dieser Ansatz ist eine threadsichere Möglichkeit, um nur eine einzelne verbundene Instanz zu initialisieren.

```csharp
private static ConnectionMultiplexer Connection;

// Redis Connection string info
private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
{
    string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
    return ConnectionMultiplexer.Connect(cacheConnection);
});

public static ConnectionMultiplexer Connection => lazyConnection.Value;
```

Die `GetMyEntityAsync`-Methode im folgenden Codebeispiel zeigt eine Implementierung des cachefremden Musters basierend auf Azure Redis Cache. Diese Methode ruft mit dem Read-Through-Ansatz ein Objekt aus dem Cache ab.

Ein Objekt wird mit einer ganzzahligen ID als Schlüssel identifiziert. Die `GetMyEntityAsync`-Methode versucht, ein Element mit diesem Schlüssel aus dem Cache abzurufen. Wenn ein übereinstimmendes Element gefunden wird, wird es zurückgegeben. Wenn im Cache keine Übereinstimmung vorhanden ist, ruft die `GetMyEntityAsync`-Methode das Objekt aus einem Datenspeicher ab, fügt es dem Cache hinzu und gibt es zurück. Der Code, der die Daten tatsächlich aus dem Datenspeicher liest, ist hier nicht dargestellt, da er vom Datenspeicher abhängt. Beachten Sie, dass für das zwischengespeicherte Element konfiguriert ist, dass es abläuft. Dadurch wird verhindert, dass es veraltet ist, wenn es an anderer Stelle aktualisiert wird.


```csharp
// Set five minute expiration as a default
private const double DefaultExpirationTimeInMinutes = 5.0;

public async Task<MyEntity> GetMyEntityAsync(int id)
{
  // Define a unique key for this method and its parameters.
  var key = $"MyEntity:{id}";
  var cache = Connection.GetDatabase();
  
  // Try to get the entity from the cache.
  var json = await cache.StringGetAsync(key).ConfigureAwait(false);
  var value = string.IsNullOrWhiteSpace(json) 
                ? default(MyEntity) 
                : JsonConvert.DeserializeObject<MyEntity>(json);
  
  if (value == null) // Cache miss
  {
    // If there's a cache miss, get the entity from the original store and cache it.
    // Code has been omitted because it's data store dependent.  
    value = ...;

    // Avoid caching a null value.
    if (value != null)
    {
      // Put the item in the cache with a custom expiration time that 
      // depends on how critical it is to have stale data.
      await cache.StringSetAsync(key, JsonConvert.SerializeObject(value)).ConfigureAwait(false);
      await cache.KeyExpireAsync(key, TimeSpan.FromMinutes(DefaultExpirationTimeInMinutes)).ConfigureAwait(false);
    }
  }

  return value;
}
```

>  Die Beispiele verwenden die Azure Redis Cache-API, um auf den Speicher zuzugreifen und Informationen aus dem Cache abzurufen. Weitere Informationen finden Sie unter [Verwenden von Azure Redis Cache](https://docs.microsoft.com/en-us/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) und [Gewusst wie: Erstellen einer Web-App mit Redis Cache](https://docs.microsoft.com/en-us/azure/redis-cache/cache-web-app-howto)

Die unten gezeigte `UpdateEntityAsync`-Methode veranschaulicht, wie ein Objekt im Cache für ungültig erklärt wird, wenn der Wert von der Anwendung geändert wird. Der Code aktualisiert den ursprünglichen Datenspeicher und entfernt dann das zwischengespeicherte Element aus dem Cache.

```csharp
public async Task UpdateEntityAsync(MyEntity entity)
{
    // Update the object in the original data store.
    await this.store.UpdateEntityAsync(entity).ConfigureAwait(false); 

    // Invalidate the current cache object.
    var cache = Connection.GetDatabase();
    var id = entity.Id;
    var key = $"MyEntity:{id}"; // The key for the cached object.
    await cache.KeyDeleteAsync(key).ConfigureAwait(false); // Delete this key from the cache.
}
```

> [!NOTE]
> Die Reihenfolge der Schritte ist wichtig. Aktualisieren Sie den Datenspeicher, *bevor* Sie das Element aus dem Cache entfernen. Wenn Sie zuerst das zwischengespeicherte Element entfernen, entsteht ein kleines Zeitfenster, in dem ein Client das Element abrufen kann, bevor der Datenspeicher aktualisiert wird. Dies führt zu einem Cachefehler (da das Element aus dem Cache entfernt wurde). Dadurch wird die frühere Version des Elements aus dem Datenspeicher abgerufen und wieder im Cache hinzugefügt. Das Ergebnis sind veraltete Cachedaten.


## <a name="related-guidance"></a>Verwandte Leitfäden 

Die folgenden Informationen sind unter Umständen auch relevant, wenn dieses Muster implementiert wird:

- [Caching Guidance (Leitfaden zum Caching)](https://docs.microsoft.com/en-us/azure/architecture/best-practices/caching). Enthält weitere Informationen zum Zwischenspeichern von Daten in einer Cloudlösung und die Probleme, die Sie bedenken sollten, wenn Sie einen Cache implementieren.

- [Data Consistency Primer (Grundlagen der Datenkonsistenz)](https://msdn.microsoft.com/library/dn589800.aspx). Cloudanwendungen verwenden in der Regel Daten, die auf Datenspeicher verteilt sind. Das Verwalten und Erhalten der Datenkonsistenz in dieser Umgebung ist ein wichtiger Aspekt des Systems, insbesondere die Probleme mit Parallelität und Dienstverfügbarkeit, die auftreten können. Dieser Artikel erläutert Probleme im Zusammenhang mit der Konsistenz verteilter Daten und fasst zusammen, wie eine Anwendung letztlich Konsistenz implementieren kann, um die Verfügbarkeit von Daten beizubehalten.
