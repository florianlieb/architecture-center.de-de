---
title: "Autorisierung in mehrmandantenfähigen Anwendungen"
description: "Informationen zum Durchführen der Autorisierung in einer mehrmandantenfähigen Anwendung"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 86c308d21f19bb3ac2a4a2240a9a03a504de5cf4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="role-based-and-resource-based-authorization"></a>Rollenbasierte und ressourcenbasierte Autorisierung

[![GitHub](../_images/github.png)-Beispielcode][sample application]

Unsere [Referenzimplementierung] ist eine ASP.NET Core-Anwendung. In diesem Artikel betrachten wir zwei allgemeine Herangehensweisen an die Autorisierung und verwenden dabei die in ASP.NET Core bereitgestellten Autorisierungs-APIs.

* **Rollenbasierte Autorisierung**. Autorisieren einer Aktion basierend auf den Rollen, die einem Benutzer zugewiesen sind. Einige Aktionen erfordern z.B. eine Administratorrolle.
* **Ressourcenbasierte Autorisierung**. Autorisieren eine Aktion basierend auf einer bestimmten Ressource. Jede Ressource hat beispielsweise einen Besitzer. Der Besitzer kann die Ressource löschen, andere Benutzer können das nicht.

In einer Standard-App wird eine Kombination beider Varianten verwendet. Um beispielsweise eine Ressource zu löschen, muss der Benutzer Besitzer der Ressource *oder* Administrator sein.

## <a name="role-based-authorization"></a>Rollenbasierte Autorisierung
Die [Tailspin Surveys][Tailspin]-Anwendung definiert die folgenden Rollen:

* Administrator. Kann alle Erstellungs-, Lese-, Aktualisierungs- und Löschaktionen auf alle Umfragen anwenden, die zu diesem Mandanten gehören.
* Creator (Ersteller). Kann neue Umfragen erstellen.
* Reader (Leser). Kann alle Umfragen lesen, die zu diesem Mandanten gehören.

Rollen gelten für *Benutzer* der Anwendung. In der Anwendung „Surveys“ ist ein Benutzer entweder Administrator, Ersteller oder Leser.

Eine Erläuterung der Definition und Verwaltung von Rollen finden Sie unter [Anwendungsrollen].

Unabhängig davon, wie Sie die Rollen verwalten, ist Ihr Autorisierungscode ähnlich. ASP.NET Core verfügt über eine Abstraktion namens [Autorisierungsrichtlinien][policies]. Mithilfe dieses Features definieren Sie Autorisierungsrichtlinien im Code und wenden anschließend diese Richtlinien auf Controlleraktionen an. Die Richtlinie ist vom Controller entkoppelt.

### <a name="create-policies"></a>Erstellen von Richtlinien
Um eine Richtlinie zu definieren, erstellen Sie zuerst eine Klasse, die `IAuthorizationRequirement`implementiert. Am einfachsten erfolgt diese über eine Ableitung von `AuthorizationHandler`. Untersuchen Sie in der `Handle` -Methode die relevanten Ansprüche.

Es folgt ein Beispiel aus der Tailspin-Anwendung „Surveys“:

```csharp
public class SurveyCreatorRequirement : AuthorizationHandler<SurveyCreatorRequirement>, IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, SurveyCreatorRequirement requirement)
    {
        if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin) || 
            context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

Diese Klasse definiert die Anforderung für einen Benutzer, eine neue Umfrage zu erstellen. Der Benutzer muss die Rolle „SurveyAdmin“ oder „SurveyCreator“ haben.

Definieren Sie in der „startup“-Klasse eine benannte Richtlinie, die eine oder mehrere Anforderungen enthält. Wenn mehrere Anforderungen vorhanden sind, muss der Benutzer *alle* Anforderungen erfüllen, um autorisiert zu werden. Der folgende Code definiert zwei Richtlinien:

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy(PolicyNames.RequireSurveyCreator,
        policy =>
        {
            policy.AddRequirements(new SurveyCreatorRequirement());
            policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement 
            // By adding the CookieAuthenticationDefaults.AuthenticationScheme, if an authenticated
            // user is not in the appropriate role, they will be redirected to a "forbidden" page.
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });

    options.AddPolicy(PolicyNames.RequireSurveyAdmin,
        policy =>
        {
            policy.AddRequirements(new SurveyAdminRequirement());
            policy.RequireAuthenticatedUser();  
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });
});
```

Dieser Code legt auch das Authentifizierungsschema fest, das ASP.NET darüber informiert, welche Authentifizierungsmiddleware ausgeführt werden soll, wenn die Authentifizierung nicht erfolgreich ist. In diesem Fall geben wir die Cookieauthentifizierungs-Middleware an, weil diese den Benutzer an eine Seite mit der Meldung „Unzulässig“ umleiten kann. Der Speicherort der Seite mit der Meldung „Unzulässig“ ist in der `AccessDeniedPath`-Option für die Cookiemiddleware festgelegt. Informationen dazu finden Sie unter [Konfigurieren der Authentifizierungsmiddleware].

### <a name="authorize-controller-actions"></a>Autorisieren von Controlleraktionen
Um eine Aktion in einem MVC-Controller zu autorisieren, legen Sie abschließend die Richtlinie im `Authorize` -Attribut fest:

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

In früheren Versionen von ASP.NET wird die **Roles** -Eigenschaft für dieses Attribut festgelegt:

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]

```

Dies wird in ASP.NET Core noch immer unterstützt, doch gibt es verglichen mit Autorisierungsrichtlinien einige Nachteile:

* Es wird von einem bestimmten Anspruchstyp ausgegangen. Mit Richtlinien können alle Anspruchstypen überprüft werden. Rollen sind lediglich ein Anspruchstyp.
* Der Rollenname ist im Attribut hartcodiert. Bei Richtlinien befindet sich die Autorisierungslogik zentral an einem Ort, wodurch das Aktualisieren oder Laden von Daten aus Konfigurationseinstellungen erleichtert wird.
* Richtlinien erlauben komplexere Autorisierungsentscheidungen (z.B. Alter >= 21), die durch einfache Rollenmitgliedschaft nicht ausgedrückt werden können.

## <a name="resource-based-authorization"></a>Ressourcenbasierte Autorisierung
Eine *ressourcenbasierte Autorisierung* findet dann statt, wenn die Autorisierung von einer bestimmten Ressource abhängig ist, die von einem Vorgang betroffen ist. In der Tailspin-Anwendung „Surveys“ hat jede Umfrage einen Besitzer und 0 bis n Teilnehmer.

* Der Besitzer kann die Umfrage lesen, aktualisieren, löschen, veröffentlichen und ihre Veröffentlichung aufheben.
* Der Besitzer kann der Umfrage Teilnehmer zuweisen.
* Teilnehmer können die Umfrage lesen und aktualisieren.

Beachten Sie, dass es sich bei „Besitzer“ und „Teilnehmer“ nicht um Anwendungsrollen handelt, die entsprechenden Werte werden für jede Umfrage in der Anwendungsdatenbank gespeichert. Um beispielsweise zu prüfen, ob ein Benutzer eine Umfrage löschen darf, überprüft die Anwendung, ob der Benutzer der Besitzer dieser Umfrage ist.

In ASP.NET Core implementieren Sie die ressourcenbasierte Autorisierung, indem Sie aus **AuthorizationHandler** ableiten und die **Handle**-Methode überschreiben.

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

Beachten Sie, dass diese Klasse für Umfrageobjekte stark typisiert ist.  Registrieren Sie die Klasse für DI beim Start:

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

Verwenden Sie zur Durchführung von Autorisierungsüberprüfungen die **IAuthorizationService** -Schnittstelle, die Sie in Ihre Controller einfügen können. Der folgende Code überprüft, ob ein Benutzer eine Umfrage lesen kann:

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

Da wir ein `Survey`-Objekt übergeben, wird jetzt der `SurveyAuthorizationHandler` aufgerufen.

Es empfiehlt sich, in Ihrem Autorisierungscode alle rollen- und ressourcenbasierten Berechtigungen des Benutzers zusammenzuführen und anschließend diesen Berechtigungssatz mit dem gewünschten Vorgang abzugleichen.
Es folgt ein Beispiel aus der App „Surveys“. Die Anwendung definiert mehrere Typen von Berechtigungen:

* Admin
* Contributor (Teilnehmer)
* Creator (Ersteller)
* Owner (Besitzer)
* Reader (Leser)

Die Anwendung definiert außerdem eine Reihe möglicher Aktionen für Umfragen:

* Erstellen
* Lesen
* Aktualisieren
* Löschen
* Veröffentlichen
* Veröffentlichung aufheben

Der folgende Code erstellt eine Liste der Berechtigungen für einen bestimmten Benutzer und eine Umfrage. Beachten Sie, dass dieser Code sowohl die App-Rollen des Benutzers als auch die Felder für Besitzer/Teilnehmer in der Umfrage untersucht.

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement requirement, Survey resource)
    {
        var permissions = new List<UserPermissionType>();
        int surveyTenantId = context.User.GetSurveyTenantIdValue();
        int userId = context.User.GetSurveyUserIdValue();
        string user = context.User.GetUserName();

        if (resource.TenantId == surveyTenantId)
        {
            // Admin can do anything, as long as the resource belongs to the admin's tenant.
            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin))
            {
                context.Succeed(requirement);
                return Task.FromResult(0);
            }

            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
            {
                permissions.Add(UserPermissionType.Creator);
            }
            else
            {
                permissions.Add(UserPermissionType.Reader);
            }

            if (resource.OwnerId == userId)
            {
                permissions.Add(UserPermissionType.Owner);
            }
        }
        if (resource.Contributors != null && resource.Contributors.Any(x => x.UserId == userId))
        {
            permissions.Add(UserPermissionType.Contributor);
        }

        if (ValidateUserPermissions[requirement](permissions))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

In einer mehrinstanzenfähigen Anwendung müssen Sie sicherstellen, dass Berechtigungen nicht an einen anderen Mandanten „durchsickern“. In der Surveys-App gilt die Berechtigung für Teilnehmer mandantenübergreifend – Sie können einen Benutzer aus einem anderen Mandanten als Teilnehmer zuweisen. Die anderen Berechtigungstypen sind auf die Ressourcen beschränkt, die zum Mandanten des Benutzers gehören. Um diese Anforderung zu erzwingen, prüft der Code die Mandanten-ID, bevor die Berechtigung gewährt wird. (Das `TenantId`-Feld wird zugewiesen, wenn die Umfrage erstellt wird.)

Im nächste Schritt wird der Vorgang (Lesen, Aktualisieren, Löschen usw.) mit den Berechtigungen verglichen. Die App „Surveys“ implementiert diesen Schritt mithilfe einer Nachschlagetabelle für Funktionen:

```csharp
static readonly Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>> ValidateUserPermissions
    = new Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>>

    {
        { Operations.Create, x => x.Contains(UserPermissionType.Creator) },

        { Operations.Read, x => x.Contains(UserPermissionType.Creator) ||
                                x.Contains(UserPermissionType.Reader) ||
                                x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Update, x => x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Delete, x => x.Contains(UserPermissionType.Owner) },

        { Operations.Publish, x => x.Contains(UserPermissionType.Owner) },

        { Operations.UnPublish, x => x.Contains(UserPermissionType.Owner) }
    };
```

[**Weiter**][web-api]

<!-- Links -->
[Tailspin]: tailspin.md

[Anwendungsrollen]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[Referenzimplementierung]: tailspin.md
[Konfigurieren der Authentifizierungsmiddleware]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
