---
title: Zwischenspeichern von Zugangstoken in einer mehrinstanzenfähigen Anwendung
description: Informationen zum Zwischenspeichern von Zugriffstoken zum Aufrufen einer Back-End-Web-API.
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: web-api
pnp.series.next: adfs
ms.openlocfilehash: cffc15686ef9d77fafb40982efdbcd4a79f5aaf2
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/08/2017
---
# <a name="cache-access-tokens"></a><span data-ttu-id="89e48-103">Zwischenspeichern von Zugriffstoken</span><span class="sxs-lookup"><span data-stu-id="89e48-103">Cache access tokens</span></span>

<span data-ttu-id="89e48-104">[![GitHub](../_images/github.png)-Beispielcode][sample application]</span><span class="sxs-lookup"><span data-stu-id="89e48-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="89e48-105">Es ist relativ aufwendig, ein OAuth-Zugriffstokens abzurufen, da dafür eine HTTP-Anforderung an den Tokenendpunkt erforderlich ist.</span><span class="sxs-lookup"><span data-stu-id="89e48-105">It's relatively expensive to get an OAuth access token, because it requires an HTTP request to the token endpoint.</span></span> <span data-ttu-id="89e48-106">Daher ist es ratsam, Token nach Möglichkeit in einem Cache zwischenzuspeichern.</span><span class="sxs-lookup"><span data-stu-id="89e48-106">Therefore, it's good to cache tokens whenever possible.</span></span> <span data-ttu-id="89e48-107">In der [Azure AD Authentication Library][ADAL] (ADAL) werden aus Azure AD abgerufene Token, einschließlich Aktualisierungstoken, automatisch zwischengespeichert.</span><span class="sxs-lookup"><span data-stu-id="89e48-107">The [Azure AD Authentication Library][ADAL] (ADAL)  automatically caches tokens obtained from Azure AD, including refresh tokens.</span></span>

<span data-ttu-id="89e48-108">Die ADAL bietet eine Standardimplementierung eines Tokencaches.</span><span class="sxs-lookup"><span data-stu-id="89e48-108">ADAL provides a default token cache implementation.</span></span> <span data-ttu-id="89e48-109">Doch dieser Tokencache ist für native Client-Apps vorgesehen und für Web-Apps **nicht** geeignet:</span><span class="sxs-lookup"><span data-stu-id="89e48-109">However, this token cache is intended for native client apps, and is **not** suitable for web apps:</span></span>

* <span data-ttu-id="89e48-110">Es handelt sich um eine statische Instanz, die nicht threadsicher ist.</span><span class="sxs-lookup"><span data-stu-id="89e48-110">It is a static instance, and not thread safe.</span></span>
* <span data-ttu-id="89e48-111">Eine Skalierung auf eine große Anzahl von Benutzern ist nicht möglich, da Token aller Benutzer in dasselbe Verzeichnis übertragen werden.</span><span class="sxs-lookup"><span data-stu-id="89e48-111">It doesn't scale to large numbers of users, because tokens from all users go into the same dictionary.</span></span>
* <span data-ttu-id="89e48-112">Eine gemeinsame Nutzung auf den Webservern einer Farm ist ebenfalls nicht möglich.</span><span class="sxs-lookup"><span data-stu-id="89e48-112">It can't be shared across web servers in a farm.</span></span>

<span data-ttu-id="89e48-113">Stattdessen müssen Sie einen benutzerdefinierten Tokencache implementieren, der von der ADAL-Klasse `TokenCache` abgeleitet ist, sich aber für eine Serverumgebung eignet und den gewünschten Grad an Isolation zwischen Token verschiedener Benutzer bietet.</span><span class="sxs-lookup"><span data-stu-id="89e48-113">Instead, you should implement a custom token cache that derives from the ADAL `TokenCache` class but is suitable for a server environment and provides the desirable level of isolation between tokens for different users.</span></span>

<span data-ttu-id="89e48-114">Die `TokenCache`-Klasse speichert ein Tokenwörterbuch, das nach Aussteller, Ressource, Client-ID und Benutzer indiziert ist.</span><span class="sxs-lookup"><span data-stu-id="89e48-114">The `TokenCache` class stores a dictionary of tokens, indexed by issuer, resource, client ID, and user.</span></span> <span data-ttu-id="89e48-115">Ein benutzerdefinierter Tokencache sollte dieses Wörterbuch in einen Sicherungsspeicher, z.B. in Redis Cache, schreiben.</span><span class="sxs-lookup"><span data-stu-id="89e48-115">A custom token cache should write this dictionary to a backing store, such as a Redis cache.</span></span>

<span data-ttu-id="89e48-116">In der Tailspin-Anwendung „Surveys“ implementiert die `DistributedTokenCache` -Klasse den Tokencache.</span><span class="sxs-lookup"><span data-stu-id="89e48-116">In the Tailspin Surveys application, the `DistributedTokenCache` class implements the token cache.</span></span> <span data-ttu-id="89e48-117">In dieser Implementierung wird die [IDistributedCache][distributed-cache]-Abstraktion aus ASP.NET Core verwendet.</span><span class="sxs-lookup"><span data-stu-id="89e48-117">This implementation uses the [IDistributedCache][distributed-cache] abstraction from ASP.NET Core.</span></span> <span data-ttu-id="89e48-118">Deshalb können alle `IDistributedCache` -Implementierungen als Sicherungsspeicher verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="89e48-118">That way, any `IDistributedCache` implementation can be used as a backing store.</span></span>

* <span data-ttu-id="89e48-119">Standardmäßig verwendet die App „Surveys“ einen Redis-Cache.</span><span class="sxs-lookup"><span data-stu-id="89e48-119">By default, the Surveys app uses a Redis cache.</span></span>
* <span data-ttu-id="89e48-120">Bei einem Einzelinstanz-Webserver können Sie den [In-Memory-Cache][in-memory-cache] von ASP.NET Core verwenden.</span><span class="sxs-lookup"><span data-stu-id="89e48-120">For a single-instance web server, you could use the ASP.NET Core [in-memory cache][in-memory-cache].</span></span> <span data-ttu-id="89e48-121">(Dies ist auch eine gute Wahl für die lokale Ausführung der App während der Entwicklung.)</span><span class="sxs-lookup"><span data-stu-id="89e48-121">(This is also a good option for running the app locally during development.)</span></span>

<span data-ttu-id="89e48-122">`DistributedTokenCache` speichert die Cachedaten als Schlüssel-Wert-Paare im Sicherungsspeicher.</span><span class="sxs-lookup"><span data-stu-id="89e48-122">`DistributedTokenCache` stores the cache data as key/value pairs in the backing store.</span></span> <span data-ttu-id="89e48-123">Der Schlüssel besteht aus Benutzer-ID und Client-ID, damit der Sicherungsspeicher separate Cachedaten für jede eindeutige Benutzer-Client-Kombination enthält.</span><span class="sxs-lookup"><span data-stu-id="89e48-123">The key is the user ID plus client ID, so the backing store holds separate cache data for each unique combination of user/client.</span></span>

![Tokencache](./images/token-cache.png)

<span data-ttu-id="89e48-125">Der Sicherungsspeicher ist nach Benutzer partitioniert.</span><span class="sxs-lookup"><span data-stu-id="89e48-125">The backing store is partitioned by user.</span></span> <span data-ttu-id="89e48-126">Für jede HTTP-Anforderung werden die Token für den Benutzer aus dem Sicherungsspeicher gelesen und in das `TokenCache`-Wörterbuch geladen.</span><span class="sxs-lookup"><span data-stu-id="89e48-126">For each HTTP request, the tokens for that user are read from the backing store and loaded into the `TokenCache` dictionary.</span></span> <span data-ttu-id="89e48-127">Wenn Redis als Sicherungsspeicher verwendet wird, liest/schreibt jede Serverinstanz in einer Serverfarm in den gleichen Cache bzw. aus dem gleichen Cache, und dieser Ansatz wird auf viele Benutzer skaliert.</span><span class="sxs-lookup"><span data-stu-id="89e48-127">If Redis is used as the backing store, every server instance in a server farm reads/writes to the same cache, and this approach scales to many users.</span></span>

## <a name="encrypting-cached-tokens"></a><span data-ttu-id="89e48-128">Verschlüsseln zwischengespeicherter Token</span><span class="sxs-lookup"><span data-stu-id="89e48-128">Encrypting cached tokens</span></span>
<span data-ttu-id="89e48-129">Token sind vertrauliche Daten, da sie den Zugriff auf Ressourcen eines Benutzers gewähren.</span><span class="sxs-lookup"><span data-stu-id="89e48-129">Tokens are sensitive data, because they grant access to a user's resources.</span></span> <span data-ttu-id="89e48-130">(Sie können zudem, im Gegensatz zum Kennwort eines Benutzers, nicht einfach den Hash eines Tokens speichern.) Deshalb müssen Token unbedingt vor Gefährdung geschützt werden.</span><span class="sxs-lookup"><span data-stu-id="89e48-130">(Moreover, unlike a user's password, you can't just store a hash of the token.) Therefore, it's critical to protect tokens from being compromised.</span></span> <span data-ttu-id="89e48-131">Doch wenn sich jemand das Kennwort verschafft, können alle zwischengespeicherten Zugriffstoken abgerufen werden.</span><span class="sxs-lookup"><span data-stu-id="89e48-131">The Redis-backed cache is protected by a password, but if someone obtains the password, they could get all of the cached access tokens.</span></span> <span data-ttu-id="89e48-132">Aus diesem Grund verschlüsselt der `DistributedTokenCache` alle Elemente, die in den Sicherungsspeicher geschrieben werden.</span><span class="sxs-lookup"><span data-stu-id="89e48-132">For that reason, the `DistributedTokenCache` encrypts everything that it writes to the backing store.</span></span> <span data-ttu-id="89e48-133">Die Verschlüsselung erfolgt mithilfe der [Datenschutz][data-protection]-APIs von ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="89e48-133">Encryption is done using the ASP.NET Core [data protection][data-protection] APIs.</span></span>

> [!NOTE]
> <span data-ttu-id="89e48-134">Wenn die Bereitstellung auf Azure-Websites erfolgt, werden die Verschlüsselungsschlüssel im Netzwerkspeicher gesichert und auf allen Computern synchronisiert (siehe [Schlüsselverwaltung und Lebensdauer][key-management]).</span><span class="sxs-lookup"><span data-stu-id="89e48-134">If you deploy to Azure Web Sites, the encryption keys are backed up to network storage and synchronized across all machines (see [Key management and lifetime][key-management]).</span></span> <span data-ttu-id="89e48-135">Bei der Ausführung auf Azure-Websites werden Schlüssel standardmäßig nicht verschlüsselt. Sie können jedoch [die Verschlüsselung mit einem X.509-Zertifikat aktivieren][x509-cert-encryption].</span><span class="sxs-lookup"><span data-stu-id="89e48-135">By default, keys are not encrypted when running in Azure Web Sites, but you can [enable encryption using an X.509 certificate][x509-cert-encryption].</span></span>
> 
> 

## <a name="distributedtokencache-implementation"></a><span data-ttu-id="89e48-136">„DistributedTokenCache“-Implementierung</span><span class="sxs-lookup"><span data-stu-id="89e48-136">DistributedTokenCache implementation</span></span>
<span data-ttu-id="89e48-137">Die `DistributedTokenCache`-Klasse wird von der ADAL-[TokenCache][tokencache-class]-Klasse abgeleitet.</span><span class="sxs-lookup"><span data-stu-id="89e48-137">The `DistributedTokenCache` class derives from the ADAL [TokenCache][tokencache-class] class.</span></span>

<span data-ttu-id="89e48-138">Im Konstruktor erstellt die `DistributedTokenCache` -Klasse einen Schlüssel für den aktuellen Benutzer und lädt den Cache aus dem Sicherungsspeicher:</span><span class="sxs-lookup"><span data-stu-id="89e48-138">In the constructor, the `DistributedTokenCache` class creates a key for the current user and loads the cache from the backing store:</span></span>

```csharp
public DistributedTokenCache(
    ClaimsPrincipal claimsPrincipal,
    IDistributedCache distributedCache,
    ILoggerFactory loggerFactory,
    IDataProtectionProvider dataProtectionProvider)
    : base()
{
    _claimsPrincipal = claimsPrincipal;
    _cacheKey = BuildCacheKey(_claimsPrincipal);
    _distributedCache = distributedCache;
    _logger = loggerFactory.CreateLogger<DistributedTokenCache>();
    _protector = dataProtectionProvider.CreateProtector(typeof(DistributedTokenCache).FullName);
    AfterAccess = AfterAccessNotification;
    LoadFromCache();
}
```

<span data-ttu-id="89e48-139">Der Schlüssel wird durch Verkettung von Benutzer-ID und Client-ID erstellt.</span><span class="sxs-lookup"><span data-stu-id="89e48-139">The key is created by concatenating the user ID and client ID.</span></span> <span data-ttu-id="89e48-140">Beide Angaben stammen aus den Ansprüchen im `ClaimsPrincipal`des Benutzers:</span><span class="sxs-lookup"><span data-stu-id="89e48-140">Both of these are taken from claims found in the user's `ClaimsPrincipal`:</span></span>

```csharp
private static string BuildCacheKey(ClaimsPrincipal claimsPrincipal)
{
    string clientId = claimsPrincipal.FindFirstValue("aud", true);
    return string.Format(
        "UserId:{0}::ClientId:{1}",
        claimsPrincipal.GetObjectIdentifierValue(),
        clientId);
}
```

<span data-ttu-id="89e48-141">Um zwischengespeicherte Daten zu laden, lesen Sie das serialisierte Blob aus dem Sicherungsspeicher und rufen `TokenCache.Deserialize` zum Konvertieren des Blobs in Cachedaten auf.</span><span class="sxs-lookup"><span data-stu-id="89e48-141">To load the cache data, read the serialized blob from the backing store, and call `TokenCache.Deserialize` to convert the blob into cache data.</span></span>

```csharp
private void LoadFromCache()
{
    byte[] cacheData = _distributedCache.Get(_cacheKey);
    if (cacheData != null)
    {
        this.Deserialize(_protector.Unprotect(cacheData));
    }
}
```

<span data-ttu-id="89e48-142">Wenn die ADAL auf den Cache zugreift, wird ein `AfterAccess` -Ereignis ausgelöst.</span><span class="sxs-lookup"><span data-stu-id="89e48-142">Whenever ADAL access the cache, it fires an `AfterAccess` event.</span></span> <span data-ttu-id="89e48-143">Wenn sich die Daten im Cache geändert haben, ist die `HasStateChanged` -Eigenschaft „true“.</span><span class="sxs-lookup"><span data-stu-id="89e48-143">If the cache data has changed, the `HasStateChanged` property is true.</span></span> <span data-ttu-id="89e48-144">Aktualisieren Sie in diesem Fall den Sicherungsspeicher, um die Änderung zu übernehmen, und legen Sie dann `HasStateChanged` auf „false“ fest.</span><span class="sxs-lookup"><span data-stu-id="89e48-144">In that case, update the backing store to reflect the change, and then set `HasStateChanged` to false.</span></span>

```csharp
public void AfterAccessNotification(TokenCacheNotificationArgs args)
{
    if (this.HasStateChanged)
    {
        try
        {
            if (this.Count > 0)
            {
                _distributedCache.Set(_cacheKey, _protector.Protect(this.Serialize()));
            }
            else
            {
                // There are no tokens for this user/client, so remove the item from the cache.
                _distributedCache.Remove(_cacheKey);
            }
            this.HasStateChanged = false;
        }
        catch (Exception exp)
        {
            _logger.WriteToCacheFailed(exp);
            throw;
        }
    }
}
```

<span data-ttu-id="89e48-145">„TokenCache“ sendet zwei weitere Ereignisse:</span><span class="sxs-lookup"><span data-stu-id="89e48-145">TokenCache sends two other events:</span></span>

* <span data-ttu-id="89e48-146">`BeforeWrite`.</span><span class="sxs-lookup"><span data-stu-id="89e48-146">`BeforeWrite`.</span></span> <span data-ttu-id="89e48-147">Wird aufgerufen, unmittelbar bevor die ADAL Daten in den Cache schreibt.</span><span class="sxs-lookup"><span data-stu-id="89e48-147">Called immediately before ADAL writes to the cache.</span></span> <span data-ttu-id="89e48-148">Es dient zum Implementieren einer Strategie für Parallelität.</span><span class="sxs-lookup"><span data-stu-id="89e48-148">You can use this to implement a concurrency strategy</span></span>
* <span data-ttu-id="89e48-149">`BeforeAccess`.</span><span class="sxs-lookup"><span data-stu-id="89e48-149">`BeforeAccess`.</span></span> <span data-ttu-id="89e48-150">Wird aufgerufen, unmittelbar bevor ADAL Daten aus dem Cache liest.</span><span class="sxs-lookup"><span data-stu-id="89e48-150">Called immediately before ADAL reads from the cache.</span></span> <span data-ttu-id="89e48-151">Hier können Sie den Cache erneut laden, um die neueste Version abzurufen.</span><span class="sxs-lookup"><span data-stu-id="89e48-151">Here you can reload the cache to get the latest version.</span></span>

<span data-ttu-id="89e48-152">In unserem Fall haben wir entschieden, diese beiden Ereignisse nicht zu behandeln.</span><span class="sxs-lookup"><span data-stu-id="89e48-152">In our case, we decided not to handle these two events.</span></span>

* <span data-ttu-id="89e48-153">Hinsichtlich der Parallelität hat der letzte Schreibzugriff Priorität.</span><span class="sxs-lookup"><span data-stu-id="89e48-153">For concurrency, last write wins.</span></span> <span data-ttu-id="89e48-154">Das ist in Ordnung, da Token für jeden Benutzer und Client eigenständig gespeichert werden und somit nur ein Konflikt auftritt, wenn für einen Benutzer zwei gleichzeitige Anmeldesitzungen vorhanden sind.</span><span class="sxs-lookup"><span data-stu-id="89e48-154">That's OK, because tokens are stored independently for each user + client, so a conflict would only happen if the same user had two concurrent login sessions.</span></span>
* <span data-ttu-id="89e48-155">Für Lesevorgänge wird der Cache bei jeder Anforderung geladen.</span><span class="sxs-lookup"><span data-stu-id="89e48-155">For reading, we load the cache on every request.</span></span> <span data-ttu-id="89e48-156">Anforderungen sind kurzlebig.</span><span class="sxs-lookup"><span data-stu-id="89e48-156">Requests are short lived.</span></span> <span data-ttu-id="89e48-157">Wenn der Cache in diesem Zeitraum geändert wird, wählt die nächste Anforderung den neuen Wert.</span><span class="sxs-lookup"><span data-stu-id="89e48-157">If the cache gets modified in that time, the next request will pick up the new value.</span></span>

<span data-ttu-id="89e48-158">[**Weiter**][client-assertion]</span><span class="sxs-lookup"><span data-stu-id="89e48-158">[**Next**][client-assertion]</span></span>

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[client-assertion]: ./client-assertion.md
[data-protection]: /aspnet/core/security/data-protection/
[distributed-cache]: /aspnet/core/performance/caching/distributed
[key-management]: /aspnet/core/security/data-protection/configuration/default-settings
[in-memory-cache]: /aspnet/core/performance/caching/memory
[tokencache-class]: https://msdn.microsoft.com/library/azure/microsoft.identitymodel.clients.activedirectory.tokencache.aspx
[x509-cert-encryption]: /aspnet/core/security/data-protection/implementation/key-encryption-at-rest#x509-certificate
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
