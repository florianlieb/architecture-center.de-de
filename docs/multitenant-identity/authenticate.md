---
title: "Authentifizierung in mehrinstanzenfähigen Anwendungen"
description: "Wie eine mehrinstanzenfähige Anwendung Benutzer von Azure AD authentifizieren kann"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: tailspin
pnp.series.next: claims
ms.openlocfilehash: 74f4e85e282799b7eee92caf2da083fb264f8733
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="authenticate-using-azure-ad-and-openid-connect"></a>Authentifizieren mithilfe von Azure AD und OpenID Connect

[![GitHub](../_images/github.png)-Beispielcode][sample application]

Die Surveys-Anwendung verwendet das OIDC-Protokoll (OpenID Connect), um Benutzer bei Azure Active Directory (Azure AD) zu authentifizieren. Die Surveys-Anwendung verwendet ASP.NET Core, das über integrierte Middleware für OIDC verfügt. Im folgenden Diagramm wird dargestellt, was geschieht, wenn sich der Benutzer auf übergeordneter Ebene anmeldet.

![Authentifizierungsfluss](./images/auth-flow.png)

1. Der Benutzer klickt in der App auf die Schaltfläche „Anmelden“. Diese Aktion wird von einem MVC-Controller bearbeitet.
2. Der MVC-Controller gibt eine **ChallengeResult** -Aktion zurück.
3. Die Middleware fängt die **ChallengeResult** -Aktion ab, und erstellt eine 302-Antwort, die den Benutzer auf die Azure AD-Anmeldeseite umleitet.
4. Der Benutzer authentifiziert sich mit Azure AD.
5. Azure AD sendet ein ID-Token an die Anwendung.
6. Die Middleware überprüft das ID-Token. Der Benutzer ist jetzt in der Anwendung authentifiziert.
7. Die Middleware leitet den Benutzer zur Anwendung zurück.

## <a name="register-the-app-with-azure-ad"></a>Registrieren der App bei Azure AD
Um OpenID Connect zu aktivieren, registriert der SaaS-Anbieter die Anwendung in seinen eigenen Azure AD-Mandanten.

Um die Anwendung zu registrieren, führen Sie die Schritte im Abschnitt [Hinzufügen einer Anwendung](/azure/active-directory/active-directory-integrating-applications/#adding-an-application) des Artikels [Integrieren von Anwendungen in Azure Active Directory](/azure/active-directory/active-directory-integrating-applications/) aus.

Die spezifischen Schritte für die Surveys-Anwendung finden Sie unter [Ausführen der Surveys-Anwendung](./run-the-app.md). Beachten Sie Folgendes:

- In einer mehrinstanzenfähigen Anwendung müssen Sie die Mehrinstanzenfähigkeit explizit konfigurieren. Dies ermöglicht anderen Organisationen den Zugriff auf die Anwendung.

- Bei der Antwort-URL handelt es sich um die URL, an die Azure AD OAuth 2.0-Antworten sendet. Bei Verwendung von ASP.NET Core muss dieser Wert dem Pfad entsprechen, den Sie in der Authentifizierungsmiddleware konfigurieren (siehe nächster Abschnitt), 

## <a name="configure-the-auth-middleware"></a>Konfigurieren der Authentifizierungsmiddleware
Dieser Abschnitt beschreibt, wie die Authentifizierungsmiddleware in ASP.NET Core für die Authentifizierung mehrerer Mandanten mit OpenID Connect konfiguriert wird.

Fügen Sie in Ihrer [Startklasse](/aspnet/core/fundamentals/startup) die OpenID Connect-Middleware hinzu:

```csharp
app.UseOpenIdConnectAuthentication(new OpenIdConnectOptions {
    ClientId = configOptions.AzureAd.ClientId,
    ClientSecret = configOptions.AzureAd.ClientSecret, // for code flow
    Authority = Constants.AuthEndpointPrefix,
    ResponseType = OpenIdConnectResponseType.CodeIdToken,
    PostLogoutRedirectUri = configOptions.AzureAd.PostLogoutRedirectUri,
    SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme,
    TokenValidationParameters = new TokenValidationParameters { ValidateIssuer = false },
    Events = new SurveyAuthenticationEvents(configOptions.AzureAd, loggerFactory),
});
```

Beachten Sie, dass einige der Einstellungen aus Optionen der Laufzeitkonfiguration entnommen werden. Die Middlewareoptionen bedeuten Folgendes:

* **ClientId**. Dies ist die Client-ID der Anwendung, die Sie erhalten haben, als Sie die Anwendung in Azure AD registriert haben.
* **Authority**. Legen Sie diese Einstellung für eine mehrinstanzenfähige Anwendung auf `https://login.microsoftonline.com/common/` fest. Dies ist die URL für den gemeinsamen Azure AD-Endpunkt, über den sich Benutzer aller Azure AD-Mandanten anmelden können. Weitere Informationen über den gemeinsamen Endpunkt finden Sie in diesem [Blogbeitrag](http://www.cloudidentity.com/blog/2014/08/26/the-common-endpoint-walks-like-a-tenant-talks-like-a-tenant-but-is-not-a-tenant/).
* Legen Sie unter **TokenValidationParameters** den Parameter **ValidateIssuer** auf „false“ fest. Dies bedeutet, dass die App für die Überprüfung des Ausstellerwerts im ID-Token zuständig ist. (Die Middleware überprüft das Token weiterhin selbst.) Weitere Informationen zum Überprüfen des Ausstellers finden Sie unter [Überprüfung des Ausstellers](claims.md#issuer-validation).
* **PostLogoutRedirectUri**. Geben Sie eine URL an, an die Benutzer nach dem Abmelden weitergeleitet werden. Hierbei sollte es sich um eine Seite handeln, die anonyme Anforderungen zulässt – typischerweise die Startseite.
* **SignInScheme**. Legen Sie diesen Eintrag auf `CookieAuthenticationDefaults.AuthenticationScheme`fest. Diese Einstellung bedeutet, dass die Benutzeransprüche nach der Authentifizierung des Benutzers lokal in einem Cookie gespeichert werden. Dieses Cookie legt fest, wie der Benutzer während der Browsersitzung angemeldet bleibt.
* **Ereignisse.** Ereignisrückrufe: siehe [Authentifizierungsereignisse](#authentication-events).

Fügen Sie außerdem die Middleware für die Cookie-Authentifizierung zur Pipeline hinzu. Diese Middleware ist dafür verantwortlich, die Benutzeransprüche in ein Cookie zu schreiben, und es im Anschluss aus einem Cookie auszulesen, während Seiten geladen werden.

```csharp
app.UseCookieAuthentication(new CookieAuthenticationOptions {
    AutomaticAuthenticate = true,
    AutomaticChallenge = true,
    AccessDeniedPath = "/Home/Forbidden",
    CookieSecure = CookieSecurePolicy.Always,

    // The default setting for cookie expiration is 14 days. SlidingExpiration is set to true by default
    ExpireTimeSpan = TimeSpan.FromHours(1),
    SlidingExpiration = true
});
```

## <a name="initiate-the-authentication-flow"></a>Initiieren des Authentifizierungsflusses
Zum Starten des Authentifizierungsflusses in ASP.NET MVC geben Sie ein **ChallengeResult** aus dem Controller zurück:

```csharp
[AllowAnonymous]
public IActionResult SignIn()
{
    return new ChallengeResult(
        OpenIdConnectDefaults.AuthenticationScheme,
        new AuthenticationProperties
        {
            IsPersistent = true,
            RedirectUri = Url.Action("SignInCallback", "Account")
        });
}
```

Dies bewirkt, dass die Middleware eine 302 (Found)-Antwort zurückgibt, mit der zum Authentifizierungsendpunkt weitergeleitet wird.

## <a name="user-login-sessions"></a>Benutzeranmeldesitzungen
Wie bereits erwähnt, schreibt die Middleware für die Cookie-Authentifizierung die Benutzeransprüche in ein Cookie, wenn sich der Benutzer zum ersten Mal anmeldet. Danach werden HTTP-Anforderungen authentifiziert, indem das Cookie gelesen wird.

Standardmäßig schreibt die Cookiemiddleware ein [Sitzungscookie][session-cookie], das gelöscht wird, sobald der Benutzer den Browser schließt. Wenn der Benutzer die Seite das nächste Mal besucht, muss er sich erneut anmelden. Wenn Sie jedoch den Parameter **IsPersistent** in **ChallengeResult** auf „true“ festlegen, schreibt die Middleware ein dauerhaftes Cookie, sodass der Benutzer auch nach Schließen des Browsers angemeldet bleibt. Sie können den Ablauf der Cookies konfigurieren. Informationen dazu finden Sie unter [Steuern von Cookieoptionen][cookie-options]. Dauerhafte Cookies sind praktischer für die Benutzer, aber für einige Anwendungen ungeeignet, bei denen sich die Benutzer jedes Mal erneut anmelden sollen (beispielsweise eine Bankinganwendung).

## <a name="about-the-openid-connect-middleware"></a>Informationen über die OpenID Connect-Middleware
Die OpenID Connect-Middleware in ASP.NET verbirgt die meisten Protokolldetails. Dieser Abschnitt enthält einige Hinweise zur Implementierung, die für das Verständnis des Protokollflusses nützlich sein können.

Zunächst sehen wir uns den Authentifizierungsablauf in Bezug auf ASP.NET an (dabei ignorieren wir die Details des Protokollflusses von OIDC zwischen der App und Azure AD). Im folgenden Diagramm wird der Prozess veranschaulicht.

![Anmeldefluss](./images/sign-in-flow.png)

In diesem Diagramm gibt es zwei MVC-Controller. Der Kontocontroller verarbeitet Anmeldeanforderungen, und der Home-Controller verarbeitet die Homepage.

So verläuft der Authentifizierungsprozess:

1. Der Benutzer klickt auf die Schaltfläche „Anmelden“, und der Browser sendet eine GET-Anforderung. Beispiel: `GET /Account/SignIn/`.
2. Der Kontocontroller gibt ein `ChallengeResult`zurück
3. Die OIDC-Middleware gibt eine HTTP 302-Antwort zurück, die zu Azure AD umleitet.
4. Der Browser sendet die Authentifizierungsanforderung an Azure AD.
5. Der Benutzer meldet sich bei Azure AD an, und Azure AD sendet eine Authentifizierungsantwort zurück.
6. Die OIDC-Middleware erstellt einen Anforderungsprinzipal und übergibt diesen der Middleware zur Cookie-Authentifizierung.
7. Die Cookie-Middleware serialisiert den Anforderungsprinzipal und setzt ein Cookie.
8. Die OIDC-Middleware leitet an die Rückruf-URL der Anwendung zurück.
9. Der Browser folgt der Umleitung, und sendet das Cookie in der Anforderung.
10. Die Cookie-Middleware deserialisiert das Cookie in einen Anforderungsprinzipal und setzt `HttpContext.User` mit dem Anforderungsprinzipal gleich. Die Anforderung wird an einen MVC-Controller weitergeleitet.

### <a name="authentication-ticket"></a>Authentifizierungsticket
Wenn die Authentifizierung erfolgreich ist, erstellt die OIDC-Middleware ein Authentifizierungsticket mit einem Anspruchsprinzipal, der die Ansprüche des Benutzers enthält. Sie können im **AuthenticationValidated**- oder **TicketReceived**-Ereignis auf das Ticket zugreifen.

> [!NOTE]
> Bis der Authentifizierungsfluss vollständig abgeschlossen ist, enthält `HttpContext.User` weiterhin einen anonymen Prinzipal, *nicht* den authentifizierten Benutzer. Der anonyme Prinzipal hat eine leere Auflistung von Ansprüchen. Wenn die Authentifizierung abgeschlossen ist, und die App weiterleitet, deserialisiert die Cookie-Middleware das Authentifizierungscookie und legt `HttpContext.User` auf einen Anspruchsprinzipal fest, der den authentifizierten Benutzer darstellt.
> 
> 

### <a name="authentication-events"></a>Authentifizierungsereignisse
Während des Authentifizierungsvorgangs löst die OpenID Connect-Middleware eine Reihe von Ereignissen aus:

* **RedirectToIdentityProvider**. Wird aufgerufen, unmittelbar bevor die Middleware an den Authentifizierungsendpunkt umleitet. Sie können dieses Ereignis verwenden, um die Umleitungs-URL zu ändern und beispielsweise Anforderungsparameter hinzuzufügen. Ein Beispiel finden Sie unter [Hinzufügen der Aufforderung zur Administratorzustimmung](signup.md#adding-the-admin-consent-prompt).
* **AuthorizationCodeReceived**. Wird mit dem Autorisierungscode aufgerufen.
* **TokenResponseReceived**. Wird aufgerufen, nachdem die Middleware ein Zugriffstoken vom Identitätsanbieter erhält, aber bevor es überprüft wird. Gilt nur für den Autorisierungscodefluss.
* **TokenValidated**. Wird aufgerufen, nachdem die Middleware das ID-Token überprüft hat. An diesem Punkt besitzt die Anwendung bereits einen Satz überprüfter Ansprüche zum Benutzer. Sie können dieses Ereignis verwenden, um zusätzliche Überprüfungen der Ansprüche durchzuführen oder diese zu transformieren. Weitere Informationen finden Sie unter [Arbeiten mit Ansprüchen](claims.md).
* **UserInformationReceived**. Wird aufgerufen, wenn die Middleware ein Benutzerprofil vom Userinfo-Endpoint erhält. Gilt nur für die Autorisierung eines Codeflusses, und auch nur dann, wenn die Middleware-Optionen `GetClaimsFromUserInfoEndpoint = true` entsprechen.
* **TicketReceived**. Wird aufgerufen, wenn die Authentifizierung abgeschlossen ist. Dies ist das letzte Ereignis, vorausgesetzt, die Authentifizierung wurde erfolgreich ausgeführt. Nachdem dieses Ereignis ausgeführt wurde, wird der Benutzer in der App angemeldet.
* **AuthenticationFailed**. Wird aufgerufen, wenn die Authentifizierung nicht erfolgreich ist. Verwenden Sie dieses Ereignis zum Verarbeiten von Authentifizierungsfehlern – z.B. durch Weiterleiten an eine Fehlerseite.

Um Rückrufe für diese Ereignisse zu erhalten, richten Sie die Option **Ereignisse** auf der Middleware ein. Es gibt zwei Möglichkeiten, die Ereignishandler zu bestimmen: Inline mit Lambda-Ausdrücken oder in einer Klasse, die von **OpenIdConnectEvents**abgeleitet ist. Der zweite Ansatz empfiehlt sich, wenn Ihre Ereignisrückrufe wesentliche Logik enthalten, und diese damit Ihre Startklasse nicht überlasten. Unsere Referenzimplementierung verwendet diesen Ansatz.

### <a name="openid-connect-endpoints"></a>OpenID Connect-Endpunkte
Azure AD unterstützt [OpenID Connect Discovery](https://openid.net/specs/openid-connect-discovery-1_0.html) – hierbei gibt der Identitätsanbieter (IDP) ein JSON-Metadatendokument von einem [bekannten Endpunkt](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig) („.well-known“) zurück. Das Metadatendokument enthält Informationen wie die folgenden:

* Die URL des Autorisierungsendpunkts. An diese URL leitet die App weiter, um den Benutzer zu authentifizieren.
* Die URL des Endpunkts „Sitzung beenden“, an dem die App den Benutzers abmeldet.
* Die URL zum Abrufen der Signaturschlüssel, die der Client zum Überprüfen der vom IDP empfangenen OIDC-Token verwendet.

Die OIDC-Middleware weiß automatisch, wie diese Metadaten abzurufen sind. Richten Sie die Option **Authority** in der Middleware ein, und die Middleware erstellt die URL für die Metadaten. (Sie können die Metadaten-URL überschreiben, indem Sie die Option **MetadataAddress** einrichten.)

### <a name="openid-connect-flows"></a>OpenID Connect-Abläufe
Standardmäßig verwendet die OIDC-Middleware einen hybriden Flow mit Formularbereitstellungs-Antwortmodus.

* *Hybrider Flow* bedeutet, dass der Client ein ID-Token und einen Autorisierungscode im gleichen Roundtrip an den Autorisierungsserver übermitteln kann.
* *Formularbereitstellungs-Antwortmodus* bedeutet, dass der Autorisierungsserver eine HTTP POST-Anforderung verwendet, um das ID-Token und den Autorisierungscode an die App zu senden. Die Werte sind Formular-URL-codiert (content type = „application/X-www-form-urlencoded“).

Wenn die OIDC-Middleware an den Autorisierungsendpunkt umleitet, enthält die Umleitungs-URL alle Abfragezeichenfolgen-Parameter, die OIDC benötigt. Für den Hybriddatenfluss:

* client_id. Dieser Wert wird in der Option **ClientId** festgelegt.
* scope = „openid profile“. Das bedeutet, dass es sich um eine OIDC-Anforderung handelt und das Benutzerprofil verwendet werden soll.
* response_type = „code id_token“. Dies gibt den Hybridflow an.
* response_mode = „form_post“. Dies gibt die Formularbereitstellungsantwort an.

Um einen anderen Datenfluss anzugeben, bestimmen Sie die Eigenschaft **ResponseType** in den Optionen. Beispiel:

```csharp
app.UseOpenIdConnectAuthentication(options =>
{
    options.ResponseType = "code"; // Authorization code flow

    // Other options
}
```

[**Weiter**][claims]

[claims]: claims.md
[cookie-options]: /aspnet/core/security/authentication/cookie#controlling-cookie-options
[session-cookie]: https://en.wikipedia.org/wiki/HTTP_cookie#Session_cookie
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
