---
title: Schützen einer Back-End-Web-API in einer mehrinstanzenfähigen Anwendung
description: Schützen einer Back-End-Web-API
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authorize
pnp.series.next: token-cache
ms.openlocfilehash: 65529280c5849e36ed7ff23de08a0b485034d0d8
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="secure-a-backend-web-api"></a><span data-ttu-id="5b666-103">Schützen einer Back-End-Web-API</span><span class="sxs-lookup"><span data-stu-id="5b666-103">Secure a backend web API</span></span>

<span data-ttu-id="5b666-104">[![GitHub](../_images/github.png)-Beispielcode][sample application]</span><span class="sxs-lookup"><span data-stu-id="5b666-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="5b666-105">Die Anwendung [Tailspin Surveys] verwendet eine Back-End-Web-API zum Verwalten von CRUD-Vorgängen für Umfragen.</span><span class="sxs-lookup"><span data-stu-id="5b666-105">The [Tailspin Surveys] application uses a backend web API to manage CRUD operations on surveys.</span></span> <span data-ttu-id="5b666-106">Klickt ein Benutzer beispielsweise auf „My Surveys“, sendet die Webanwendung eine HTTP-Anforderung an die Web-API:</span><span class="sxs-lookup"><span data-stu-id="5b666-106">For example, when a user clicks "My Surveys", the web application sends an HTTP request to the web API:</span></span>

```
GET /users/{userId}/surveys
```

<span data-ttu-id="5b666-107">Die Web-API gibt ein JSON-Objekt zurück:</span><span class="sxs-lookup"><span data-stu-id="5b666-107">The web API returns a JSON object:</span></span>

```
{
  "Published":[],
  "Own":[
    {"Id":1,"Title":"Survey 1"},
    {"Id":3,"Title":"Survey 3"},
    ],
  "Contribute": [{"Id":8,"Title":"My survey"}]
}
```

<span data-ttu-id="5b666-108">Die Web-API lässt keine anonymen Anforderungen zu, sodass sich die Web-App mithilfe von OAuth2-Bearertoken selbst authentifizieren muss.</span><span class="sxs-lookup"><span data-stu-id="5b666-108">The web API does not allow anonymous requests, so the web app must authenticate itself using OAuth 2 bearer tokens.</span></span>

> [!NOTE]
> <span data-ttu-id="5b666-109">Dies ist ein Szenario zwischen Servern.</span><span class="sxs-lookup"><span data-stu-id="5b666-109">This is a server-to-server scenario.</span></span> <span data-ttu-id="5b666-110">Die Anwendung richtet aus dem Browserclient keine AJAX-Aufrufe an die API.</span><span class="sxs-lookup"><span data-stu-id="5b666-110">The application does not make any AJAX calls to the API from the browser client.</span></span>
> 
> 

<span data-ttu-id="5b666-111">Sie können zwischen zwei Ansätzen wählen:</span><span class="sxs-lookup"><span data-stu-id="5b666-111">There are two main approaches you can take:</span></span>

* <span data-ttu-id="5b666-112">Delegierte Benutzeridentität.</span><span class="sxs-lookup"><span data-stu-id="5b666-112">Delegated user identity.</span></span> <span data-ttu-id="5b666-113">Die Webanwendung authentifiziert sich mit der Identität des Benutzers.</span><span class="sxs-lookup"><span data-stu-id="5b666-113">The web application authenticates with the user's identity.</span></span>
* <span data-ttu-id="5b666-114">Anwendungsidentität.</span><span class="sxs-lookup"><span data-stu-id="5b666-114">Application identity.</span></span> <span data-ttu-id="5b666-115">Die Webanwendung authentifiziert sich mit Ihrer Client-ID bei Befolgen des OAuth2-Ablaufs für Clientanmeldeinformationen.</span><span class="sxs-lookup"><span data-stu-id="5b666-115">The web application authenticates with its client ID, using OAuth2 client credential flow.</span></span>

<span data-ttu-id="5b666-116">Die Tailspin-Anwendung implementiert die delegierte Benutzeridentität.</span><span class="sxs-lookup"><span data-stu-id="5b666-116">The Tailspin application implements delegated user identity.</span></span> <span data-ttu-id="5b666-117">Im Folgenden werden die Hauptunterschiede erläutert:</span><span class="sxs-lookup"><span data-stu-id="5b666-117">Here are the main differences:</span></span>

<span data-ttu-id="5b666-118">**Delegierte Benutzeridentität**</span><span class="sxs-lookup"><span data-stu-id="5b666-118">**Delegated user identity**</span></span>

* <span data-ttu-id="5b666-119">Das an die Web-API gesendete Bearertoken enthält die Identität des Benutzers.</span><span class="sxs-lookup"><span data-stu-id="5b666-119">The bearer token sent to the web API contains the user identity.</span></span>
* <span data-ttu-id="5b666-120">Die Web-API trifft Autorisierungsentscheidungen basierend auf der Identität des Benutzers.</span><span class="sxs-lookup"><span data-stu-id="5b666-120">The web API makes authorization decisions based on the user identity.</span></span>
* <span data-ttu-id="5b666-121">Die Webanwendung muss von der Web-API eingehende Fehler des Typs „403 (Verboten)“ behandeln, wenn der Benutzer für das Ausführen einer Aktion nicht autorisiert ist.</span><span class="sxs-lookup"><span data-stu-id="5b666-121">The web application needs to handle 403 (Forbidden) errors from the web API, if the user is not authorized to perform an action.</span></span>
* <span data-ttu-id="5b666-122">Zumeist trifft die Webanwendung weiterhin einige sich auf die Benutzeroberfläche beziehende Autorisierungsentscheidungen (wie das Ein- und Ausblenden von Benutzeroberflächenelementen).</span><span class="sxs-lookup"><span data-stu-id="5b666-122">Typically, the web application still makes some authorization decisions that affect UI, such as showing or hiding UI elements).</span></span>
* <span data-ttu-id="5b666-123">Die Web-API kann möglicherweise von nicht vertrauenswürdigen Clients verwendet werden, z. B. einer JavaScript-Anwendung oder einer systemeigenen Clientanwendung.</span><span class="sxs-lookup"><span data-stu-id="5b666-123">The web API can potentially be used by untrusted clients, such as a JavaScript application or a native client application.</span></span>

<span data-ttu-id="5b666-124">**Anwendungsidentität**</span><span class="sxs-lookup"><span data-stu-id="5b666-124">**Application identity**</span></span>

* <span data-ttu-id="5b666-125">Die Web-API ruft keine Informationen zum Benutzer ab.</span><span class="sxs-lookup"><span data-stu-id="5b666-125">The web API does not get information about the user.</span></span>
* <span data-ttu-id="5b666-126">Die Web-API kann keine Autorisierung anhand der Identität des Benutzers vornehmen.</span><span class="sxs-lookup"><span data-stu-id="5b666-126">The web API cannot perform any authorization based on the user identity.</span></span> <span data-ttu-id="5b666-127">Alle Autorisierungsentscheidungen werden von der Webanwendung getroffen.</span><span class="sxs-lookup"><span data-stu-id="5b666-127">All authorization decisions are made by the web application.</span></span>  
* <span data-ttu-id="5b666-128">Die Web-API kann nicht von einem nicht vertrauenswürdigen Client (JavaScript oder native Clientanwendung) verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="5b666-128">The web API cannot be used by an untrusted client (JavaScript or native client application).</span></span>
* <span data-ttu-id="5b666-129">Dieser Ansatz ist möglicherweise etwas einfacher zu implementieren, da keine Autorisierungslogik in der Web-API vorhanden ist.</span><span class="sxs-lookup"><span data-stu-id="5b666-129">This approach may be somewhat simpler to implement, because there is no authorization logic in the Web API.</span></span>

<span data-ttu-id="5b666-130">Bei beiden Ansätzen muss die Webanwendung ein Zugriffstoken als Anmeldeinformation erhalten, die für das Aufrufen der Web-API erforderlich ist.</span><span class="sxs-lookup"><span data-stu-id="5b666-130">In either approach, the web application must get an access token, which is the credential needed to call the web API.</span></span>

* <span data-ttu-id="5b666-131">Für die delegierte Benutzeridentität muss das Token vom Identitätsanbieter stammen, der ein Token für den Benutzer ausstellen kann.</span><span class="sxs-lookup"><span data-stu-id="5b666-131">For delegated user identity, the token has to come from the IDP, which can issue a token on behalf of the user.</span></span>
* <span data-ttu-id="5b666-132">Für Clientanmeldeinformationen kann eine Anwendung das Token vom Identitätsanbieter abrufen oder einen eigenen Tokenserver hosten.</span><span class="sxs-lookup"><span data-stu-id="5b666-132">For client credentials, an application might get the token from the IDP or host its own token server.</span></span> <span data-ttu-id="5b666-133">(Schreiben Sie aber keinen Tokenserver von Grund auf neu, sondern verwenden Sie ein sorgfältig getestetes Framework, z. B. [IdentityServer3].) Beim Authentifizieren mit Azure AD wird dringend empfohlen, das Zugriffstoken auch beim Vorgang mit Clientanmeldeinformationen aus Azure AD abzurufen.</span><span class="sxs-lookup"><span data-stu-id="5b666-133">(But don't write a token server from scratch; use a well-tested framework like [IdentityServer3].) If you authenticate with Azure AD, it's strongly recommended to get the access token from Azure AD, even with client credential flow.</span></span>

<span data-ttu-id="5b666-134">Im Rest dieses Artikels wird davon ausgegangen, dass die Anwendung mithilfe von Azure AD authentifiziert wird.</span><span class="sxs-lookup"><span data-stu-id="5b666-134">The rest of this article assumes the application is authenticating with Azure AD.</span></span>

![Abrufen des Zugriffstokens](./images/access-token.png)

## <a name="register-the-web-api-in-azure-ad"></a><span data-ttu-id="5b666-136">Registrieren der Web-API in Azure AD</span><span class="sxs-lookup"><span data-stu-id="5b666-136">Register the web API in Azure AD</span></span>
<span data-ttu-id="5b666-137">Damit Azure AD ein Bearertoken für die Web-API ausstellen kann, müssen in Azure AD verschiedene Einstellungen konfiguriert werden.</span><span class="sxs-lookup"><span data-stu-id="5b666-137">In order for Azure AD to issue a bearer token for the web API, you need to configure some things in Azure AD.</span></span>

1. <span data-ttu-id="5b666-138">Registrieren Sie die Web-API in Azure AD.</span><span class="sxs-lookup"><span data-stu-id="5b666-138">Register the web API in Azure AD.</span></span>

2. <span data-ttu-id="5b666-139">Fügen Sie dem Anwendungsmanifest der Web-API in der `knownClientApplications` -Eigenschaft die Client-ID der Web-App hinzu.</span><span class="sxs-lookup"><span data-stu-id="5b666-139">Add the client ID of the web app to the web API application manifest, in the `knownClientApplications` property.</span></span> <span data-ttu-id="5b666-140">Siehe [Aktualisieren des Anwendungsmanifests].</span><span class="sxs-lookup"><span data-stu-id="5b666-140">See [Update the application manifests].</span></span>

3. <span data-ttu-id="5b666-141">Erteilen Sie der Webanwendung Berechtigungen zum Aufrufen der Web-API.</span><span class="sxs-lookup"><span data-stu-id="5b666-141">Give the web application permission to call the web API.</span></span> <span data-ttu-id="5b666-142">Im Azure-Verwaltungsportal können Sie zwei Arten von Berechtigungen festlegen: „Anwendungsberechtigungen“ für die Anwendungsidentität (Vorgang mit Clientanmeldeinformationen) oder „Delegierte Berechtigungen“ für die delegierte Benutzeridentität.</span><span class="sxs-lookup"><span data-stu-id="5b666-142">In the Azure Management Portal, you can set two types of permissions: "Application Permissions" for application identity (client credential flow), or "Delegated Permissions" for delegated user identity.</span></span>
   
   ![Delegierte Berechtigungen](./images/delegated-permissions.png)

## <a name="getting-an-access-token"></a><span data-ttu-id="5b666-144">Abrufen eines Zugriffstokens</span><span class="sxs-lookup"><span data-stu-id="5b666-144">Getting an access token</span></span>
<span data-ttu-id="5b666-145">Vor dem Aufruf der Web-API ruft die Webanwendung ein Zugriffstoken aus Azure AD ab.</span><span class="sxs-lookup"><span data-stu-id="5b666-145">Before calling the web API, the web application gets an access token from Azure AD.</span></span> <span data-ttu-id="5b666-146">Verwenden Sie in einer .NET-Anwendung die [Azure AD-Authentifizierungsbibliothek (ADAL) für .NET][ADAL].</span><span class="sxs-lookup"><span data-stu-id="5b666-146">In a .NET application, use the [Azure AD Authentication Library (ADAL) for .NET][ADAL].</span></span>

<span data-ttu-id="5b666-147">Beim Vorgang mit OAuth 2-Autorisierungscode tauscht die Anwendung einen Autorisierungscode gegen ein Zugriffstoken.</span><span class="sxs-lookup"><span data-stu-id="5b666-147">In the OAuth 2 authorization code flow, the application exchanges an authorization code for an access token.</span></span> <span data-ttu-id="5b666-148">Der folgende Code verwendet die ADAL, um das Zugriffstoken abzurufen.</span><span class="sxs-lookup"><span data-stu-id="5b666-148">The following code uses ADAL to get the access token.</span></span> <span data-ttu-id="5b666-149">Dieser Code wird während des `AuthorizationCodeReceived` -Ereignisses abgerufen.</span><span class="sxs-lookup"><span data-stu-id="5b666-149">This code is called during the `AuthorizationCodeReceived` event.</span></span>

```csharp
// The OpenID Connect middleware sends this event when it gets the authorization code.   
public override async Task AuthorizationCodeReceived(AuthorizationCodeReceivedContext context)
{
    string authorizationCode = context.ProtocolMessage.Code;
    string authority = "https://login.microsoftonline.com/" + tenantID
    string resourceID = "https://tailspin.onmicrosoft.com/surveys.webapi" // App ID URI
    ClientCredential credential = new ClientCredential(clientId, clientSecret);

    AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
    AuthenticationResult authResult = await authContext.AcquireTokenByAuthorizationCodeAsync(
        authorizationCode, new Uri(redirectUri), credential, resourceID);

    // If successful, the token is in authResult.AccessToken
}
```

<span data-ttu-id="5b666-150">Hier die verschiedenen erforderlichen Parameter:</span><span class="sxs-lookup"><span data-stu-id="5b666-150">Here are the various parameters that are needed:</span></span>

* <span data-ttu-id="5b666-151">`authority`.</span><span class="sxs-lookup"><span data-stu-id="5b666-151">`authority`.</span></span> <span data-ttu-id="5b666-152">Von der Mandanten-ID des angemeldeten Benutzers abgeleitet.</span><span class="sxs-lookup"><span data-stu-id="5b666-152">Derived from the tenant ID of the signed in user.</span></span> <span data-ttu-id="5b666-153">(Nicht die Mandanten-ID des SaaS-Anbieters)</span><span class="sxs-lookup"><span data-stu-id="5b666-153">(Not the tenant ID of the SaaS provider)</span></span>  
* <span data-ttu-id="5b666-154">`authorizationCode`.</span><span class="sxs-lookup"><span data-stu-id="5b666-154">`authorizationCode`.</span></span> <span data-ttu-id="5b666-155">Der Autorisierungscode, den Sie vom Identitätsanbieter erhalten haben.</span><span class="sxs-lookup"><span data-stu-id="5b666-155">the auth code that you got back from the IDP.</span></span>
* <span data-ttu-id="5b666-156">`clientId`.</span><span class="sxs-lookup"><span data-stu-id="5b666-156">`clientId`.</span></span> <span data-ttu-id="5b666-157">Die Client-ID der Webanwendung.</span><span class="sxs-lookup"><span data-stu-id="5b666-157">The web application's client ID.</span></span>
* <span data-ttu-id="5b666-158">`clientSecret`.</span><span class="sxs-lookup"><span data-stu-id="5b666-158">`clientSecret`.</span></span> <span data-ttu-id="5b666-159">Der geheime Clientschlüssel der Webanwendung.</span><span class="sxs-lookup"><span data-stu-id="5b666-159">The web application's client secret.</span></span>
* <span data-ttu-id="5b666-160">`redirectUri`.</span><span class="sxs-lookup"><span data-stu-id="5b666-160">`redirectUri`.</span></span> <span data-ttu-id="5b666-161">Der Umleitung-URI, den Sie für OpenID Connect festgelegt haben.</span><span class="sxs-lookup"><span data-stu-id="5b666-161">The redirect URI that you set for OpenID connect.</span></span> <span data-ttu-id="5b666-162">Hier folgt der Rückruf des Identitätsanbieters mit dem Token.</span><span class="sxs-lookup"><span data-stu-id="5b666-162">This is where the IDP calls back with the token.</span></span>
* <span data-ttu-id="5b666-163">`resourceID`.</span><span class="sxs-lookup"><span data-stu-id="5b666-163">`resourceID`.</span></span> <span data-ttu-id="5b666-164">Die App-ID-URI der Web-API, die Sie bei der Registrierung der Web-API in Azure AD erstellt haben.</span><span class="sxs-lookup"><span data-stu-id="5b666-164">The App ID URI of the web API, which you created when you registered the web API in Azure AD</span></span>
* <span data-ttu-id="5b666-165">`tokenCache`.</span><span class="sxs-lookup"><span data-stu-id="5b666-165">`tokenCache`.</span></span> <span data-ttu-id="5b666-166">Ein Objekt, das die Zugriffstoken zwischengespeichert.</span><span class="sxs-lookup"><span data-stu-id="5b666-166">An object that caches the access tokens.</span></span> <span data-ttu-id="5b666-167">Siehe [Tokencaching].</span><span class="sxs-lookup"><span data-stu-id="5b666-167">See [Token caching].</span></span>

<span data-ttu-id="5b666-168">Wenn `AcquireTokenByAuthorizationCodeAsync` erfolgreich ist, speichert die ADAL das Token zwischen.</span><span class="sxs-lookup"><span data-stu-id="5b666-168">If `AcquireTokenByAuthorizationCodeAsync` succeeds, ADAL caches the token.</span></span> <span data-ttu-id="5b666-169">Später können Sie das Token durch Aufrufen von „AcquireTokenSilentAsync“ aus dem Cache abrufen:</span><span class="sxs-lookup"><span data-stu-id="5b666-169">Later, you can get the token from the cache by calling AcquireTokenSilentAsync:</span></span>

```csharp
AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
var result = await authContext.AcquireTokenSilentAsync(resourceID, credential, new UserIdentifier(userId, UserIdentifierType.UniqueId));
```

<span data-ttu-id="5b666-170">Hierbei ist `userId` die Objekt-ID des Benutzers, die sich im Anspruch `http://schemas.microsoft.com/identity/claims/objectidentifier` befindet.</span><span class="sxs-lookup"><span data-stu-id="5b666-170">where `userId` is the user's object ID, which is found in the `http://schemas.microsoft.com/identity/claims/objectidentifier` claim.</span></span>

## <a name="using-the-access-token-to-call-the-web-api"></a><span data-ttu-id="5b666-171">Aufrufen der Web-API mithilfe des Zugriffstokens</span><span class="sxs-lookup"><span data-stu-id="5b666-171">Using the access token to call the web API</span></span>
<span data-ttu-id="5b666-172">Sobald Sie das Token haben, senden Sie es im „Authorization“-Header der HTTP-Anforderungen an die Web-API.</span><span class="sxs-lookup"><span data-stu-id="5b666-172">Once you have the token, send it in the Authorization header of the HTTP requests to the web API.</span></span>

```
Authorization: Bearer xxxxxxxxxx
```

<span data-ttu-id="5b666-173">Die folgende Erweiterungsmethode aus der Anwendung „Surveys“ legt den „Authorization“-Header unter Verwendung der **HttpClient** -Klasse auf eine HTTP-Anforderung fest.</span><span class="sxs-lookup"><span data-stu-id="5b666-173">The following extension method from the Surveys application sets the Authorization header on an HTTP request, using the **HttpClient** class.</span></span>

```csharp
public static async Task<HttpResponseMessage> SendRequestWithBearerTokenAsync(this HttpClient httpClient, HttpMethod method, string path, object requestBody, string accessToken, CancellationToken ct)
{
    var request = new HttpRequestMessage(method, path);
    if (requestBody != null)
    {
        var json = JsonConvert.SerializeObject(requestBody, Formatting.None);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        request.Content = content;
    }

    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
    request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

    var response = await httpClient.SendAsync(request, ct);
    return response;
}
```

## <a name="authenticating-in-the-web-api"></a><span data-ttu-id="5b666-174">Authentifizierung in der Web-API</span><span class="sxs-lookup"><span data-stu-id="5b666-174">Authenticating in the web API</span></span>
<span data-ttu-id="5b666-175">Die Web-API muss das Bearertoken authentifizieren.</span><span class="sxs-lookup"><span data-stu-id="5b666-175">The web API has to authenticate the bearer token.</span></span> <span data-ttu-id="5b666-176">In ASP.NET Core, können Sie das Paket [Microsoft.AspNet.Authentication.JwtBearer][JwtBearer] verwenden.</span><span class="sxs-lookup"><span data-stu-id="5b666-176">In ASP.NET Core, you can use the [Microsoft.AspNet.Authentication.JwtBearer][JwtBearer] package.</span></span> <span data-ttu-id="5b666-177">Dieses Paket enthält Middleware, die der Anwendung das Empfangen von OpenID Connect-Bearertoken ermöglicht.</span><span class="sxs-lookup"><span data-stu-id="5b666-177">This package provides middleware that enables the application to receive OpenID Connect bearer tokens.</span></span>

<span data-ttu-id="5b666-178">Registrieren Sie die Middleware in Ihrer Web-API-Klasse `Startup` .</span><span class="sxs-lookup"><span data-stu-id="5b666-178">Register the middleware in your web API `Startup` class.</span></span>

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ApplicationDbContext dbContext, ILoggerFactory loggerFactory)
{
    // ...

    app.UseJwtBearerAuthentication(new JwtBearerOptions {
        Audience = configOptions.AzureAd.WebApiResourceId,
        Authority = Constants.AuthEndpointPrefix,
        TokenValidationParameters = new TokenValidationParameters {
            ValidateIssuer = false
        },
        Events= new SurveysJwtBearerEvents(loggerFactory.CreateLogger<SurveysJwtBearerEvents>())
    });
    
    // ...
}
```

* <span data-ttu-id="5b666-179">**Audience**.</span><span class="sxs-lookup"><span data-stu-id="5b666-179">**Audience**.</span></span> <span data-ttu-id="5b666-180">Legen Sie diese Einstellung auf die App-ID-URI der Web-API fest, die Sie bei der Registrierung der Web-API in Azure AD erstellt haben.</span><span class="sxs-lookup"><span data-stu-id="5b666-180">Set this to the App ID URL for the web API, which you created when you registered the web API with Azure AD.</span></span>
* <span data-ttu-id="5b666-181">**Authority**.</span><span class="sxs-lookup"><span data-stu-id="5b666-181">**Authority**.</span></span> <span data-ttu-id="5b666-182">Legen Sie diese Einstellung für eine mehrinstanzenfähige Anwendung auf `https://login.microsoftonline.com/common/` fest.</span><span class="sxs-lookup"><span data-stu-id="5b666-182">For a multitenant application, set this to `https://login.microsoftonline.com/common/`.</span></span>
* <span data-ttu-id="5b666-183">**TokenValidationParameters**.</span><span class="sxs-lookup"><span data-stu-id="5b666-183">**TokenValidationParameters**.</span></span> <span data-ttu-id="5b666-184">Legen Sie für eine mehrinstanzenfähige Anwendung **ValidateIssuer** auf „false“ fest.</span><span class="sxs-lookup"><span data-stu-id="5b666-184">For a multitenant application, set **ValidateIssuer** to false.</span></span> <span data-ttu-id="5b666-185">Das bedeutet, dass die Anwendung den Aussteller überprüft.</span><span class="sxs-lookup"><span data-stu-id="5b666-185">That means the application will validate the issuer.</span></span>
* <span data-ttu-id="5b666-186">**Events** ist eine von **JwtBearerEvents** abgeleitete Klasse.</span><span class="sxs-lookup"><span data-stu-id="5b666-186">**Events** is a class that derives from **JwtBearerEvents**.</span></span>

### <a name="issuer-validation"></a><span data-ttu-id="5b666-187">Überprüfung des Ausstellers</span><span class="sxs-lookup"><span data-stu-id="5b666-187">Issuer validation</span></span>
<span data-ttu-id="5b666-188">Überprüfen Sie den Aussteller des Tokens im **JwtBearerEvents.TokenValidated**-Ereignis.</span><span class="sxs-lookup"><span data-stu-id="5b666-188">Validate the token issuer in the **JwtBearerEvents.TokenValidated** event.</span></span> <span data-ttu-id="5b666-189">Der Aussteller wird im Anspruch „iss“ gesendet.</span><span class="sxs-lookup"><span data-stu-id="5b666-189">The issuer is sent in the "iss" claim.</span></span>

<span data-ttu-id="5b666-190">In der Anwendung „Surveys“ wird die [Mandantenanmeldung]nicht von der Web-API verarbeitet.</span><span class="sxs-lookup"><span data-stu-id="5b666-190">In the Surveys application, the web API doesn't handle [tenant sign-up].</span></span> <span data-ttu-id="5b666-191">Daher wird nur überprüft, ob der Aussteller bereits in der Anwendungsdatenbank vorhanden ist.</span><span class="sxs-lookup"><span data-stu-id="5b666-191">Therefore, it just checks if the issuer is already in the application database.</span></span> <span data-ttu-id="5b666-192">Falls nicht, wird eine Ausnahme ausgelöst, die einen Authentifizierungsfehler verursacht.</span><span class="sxs-lookup"><span data-stu-id="5b666-192">If not, it throws an exception, which causes authentication to fail.</span></span>

```csharp
public override async Task TokenValidated(TokenValidatedContext context)
{
    var principal = context.Ticket.Principal;
    var tenantManager = context.HttpContext.RequestServices.GetService<TenantManager>();
    var userManager = context.HttpContext.RequestServices.GetService<UserManager>();
    var issuerValue = principal.GetIssuerValue();
    var tenant = await tenantManager.FindByIssuerValueAsync(issuerValue);

    if (tenant == null)
    {
        // The caller was not from a trusted issuer. Throw to block the authentication flow.
        throw new SecurityTokenValidationException();
    }

    var identity = principal.Identities.First();

    // Add new claim for survey_userid
    var registeredUser = await userManager.FindByObjectIdentifier(principal.GetObjectIdentifierValue());
    identity.AddClaim(new Claim(SurveyClaimTypes.SurveyUserIdClaimType, registeredUser.Id.ToString()));
    identity.AddClaim(new Claim(SurveyClaimTypes.SurveyTenantIdClaimType, registeredUser.TenantId.ToString()));

    // Add new claim for Email
    var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
    if (!string.IsNullOrWhiteSpace(email))
    {
        identity.AddClaim(new Claim(ClaimTypes.Email, email));
    }
}
```

<span data-ttu-id="5b666-193">Wie dieses Beispiel zeigt, können Sie auch das **TokenValidated**-Ereignis zum Ändern der Ansprüche verwenden.</span><span class="sxs-lookup"><span data-stu-id="5b666-193">As this example shows, you can also use the **TokenValidated** event to modify the claims.</span></span> <span data-ttu-id="5b666-194">Denken Sie daran, dass die Ansprüche direkt von Azure AD stammen.</span><span class="sxs-lookup"><span data-stu-id="5b666-194">Remember that the claims come directly from Azure AD.</span></span> <span data-ttu-id="5b666-195">Wenn die Webanwendung die abgerufenen Ansprüche ändert, werden diese Änderungen nicht in dem von der Web-API empfangenen Bearertoken angezeigt.</span><span class="sxs-lookup"><span data-stu-id="5b666-195">If the web application modifies the claims that it gets, those changes won't show up in the bearer token that the web API receives.</span></span> <span data-ttu-id="5b666-196">Weitere Informationen finden Sie unter [Transformationen von Ansprüchen][claims-transformation].</span><span class="sxs-lookup"><span data-stu-id="5b666-196">For more information, see [Claims transformations][claims-transformation].</span></span>

## <a name="authorization"></a><span data-ttu-id="5b666-197">Autorisierung</span><span class="sxs-lookup"><span data-stu-id="5b666-197">Authorization</span></span>
<span data-ttu-id="5b666-198">Eine allgemeine Erörterung der Autorisierung finden Sie unter [Rollenbasierte und ressourcenbasierte Autorisierung][Authorization].</span><span class="sxs-lookup"><span data-stu-id="5b666-198">For a general discussion of authorization, see [Role-based and resource-based authorization][Authorization].</span></span> 

<span data-ttu-id="5b666-199">Die JwtBearer-Middleware verarbeitet die Autorisierungsantworten.</span><span class="sxs-lookup"><span data-stu-id="5b666-199">The JwtBearer middleware handles the authorization responses.</span></span> <span data-ttu-id="5b666-200">Wenn Sie z. B. eine Controlleraktion auf authentifizierte Benutzer beschränken möchten, verwenden Sie das Attribut **[Authorize]**, und geben Sie **JwtBearerDefaults.AuthenticationScheme** als Authentifizierungsschema an:</span><span class="sxs-lookup"><span data-stu-id="5b666-200">For example, to restrict a controller action to authenticated users, use the **[Authorize]** atrribute and specify **JwtBearerDefaults.AuthenticationScheme** as the authentication scheme:</span></span>

```csharp
[Authorize(ActiveAuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
```

<span data-ttu-id="5b666-201">Dies gibt den Statuscode 401 zurück, wenn der Benutzer nicht authentifiziert ist.</span><span class="sxs-lookup"><span data-stu-id="5b666-201">This returns a 401 status code if the user is not authenticated.</span></span>

<span data-ttu-id="5b666-202">Soll eine Controlleraktion mithilfe einer Autorisierungsrichtlinie beschränkt werden, geben Sie den Richtliniennamen im Attribut **[Authorize]** an:</span><span class="sxs-lookup"><span data-stu-id="5b666-202">To restrict a controller action by authorizaton policy, specify the policy name in the **[Authorize]** attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
```

<span data-ttu-id="5b666-203">Dies gibt den Statuscode 401 zurück, wenn der Benutzer nicht authentifiziert ist, und 403, wenn der Benutzer authentifiziert, aber nicht autorisiert ist.</span><span class="sxs-lookup"><span data-stu-id="5b666-203">This returns a 401 status code if the user is not authenticated, and 403 if the user is authenticated but not authorized.</span></span> <span data-ttu-id="5b666-204">Registrieren Sie die Richtlinie beim Start:</span><span class="sxs-lookup"><span data-stu-id="5b666-204">Register the policy on startup:</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthorization(options =>
    {
        options.AddPolicy(PolicyNames.RequireSurveyCreator,
            policy =>
            {
                policy.AddRequirements(new SurveyCreatorRequirement());
                policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement 
                policy.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme);
            });
        options.AddPolicy(PolicyNames.RequireSurveyAdmin,
            policy =>
            {
                policy.AddRequirements(new SurveyAdminRequirement());
                policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement 
                policy.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme);
            });
    });
    
    // ...
}
```

<span data-ttu-id="5b666-205">[**Weiter**][token cache]</span><span class="sxs-lookup"><span data-stu-id="5b666-205">[**Next**][token cache]</span></span>

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[JwtBearer]: https://www.nuget.org/packages/Microsoft.AspNet.Authentication.JwtBearer

[Tailspin Surveys]: tailspin.md
[IdentityServer3]: https://github.com/IdentityServer/IdentityServer3
[Aktualisieren des Anwendungsmanifests]: ./run-the-app.md#update-the-application-manifests
[Update the application manifests]: ./run-the-app.md#update-the-application-manifests
[Tokencaching]: token-cache.md
[Token caching]: token-cache.md
[Mandantenanmeldung]: signup.md
[tenant sign-up]: signup.md
[claims-transformation]: claims.md#claims-transformations
[Authorization]: authorize.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[token cache]: token-cache.md
