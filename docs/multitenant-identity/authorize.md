---
title: Autorisierung in mehrmandantenfähigen Anwendungen
description: Informationen zum Durchführen der Autorisierung in einer mehrmandantenfähigen Anwendung
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 03c4d5fa10c75437a7b066534619ba9a123c350c
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="role-based-and-resource-based-authorization"></a><span data-ttu-id="1a179-103">Rollenbasierte und ressourcenbasierte Autorisierung</span><span class="sxs-lookup"><span data-stu-id="1a179-103">Role-based and resource-based authorization</span></span>

<span data-ttu-id="1a179-104">[![GitHub](../_images/github.png)-Beispielcode][sample application]</span><span class="sxs-lookup"><span data-stu-id="1a179-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="1a179-105">Unsere [Referenzimplementierung] ist eine ASP.NET Core-Anwendung.</span><span class="sxs-lookup"><span data-stu-id="1a179-105">Our [reference implementation] is an ASP.NET Core application.</span></span> <span data-ttu-id="1a179-106">In diesem Artikel betrachten wir zwei allgemeine Herangehensweisen an die Autorisierung und verwenden dabei die in ASP.NET Core bereitgestellten Autorisierungs-APIs.</span><span class="sxs-lookup"><span data-stu-id="1a179-106">In this article we'll look at two general approaches to authorization, using the authorization APIs provided in ASP.NET Core.</span></span>

* <span data-ttu-id="1a179-107">**Role-based authorization**.</span><span class="sxs-lookup"><span data-stu-id="1a179-107">**Role-based authorization**.</span></span> <span data-ttu-id="1a179-108">Autorisieren einer Aktion basierend auf den Rollen, die einem Benutzer zugewiesen sind.</span><span class="sxs-lookup"><span data-stu-id="1a179-108">Authorizing an action based on the roles assigned to a user.</span></span> <span data-ttu-id="1a179-109">Einige Aktionen erfordern z.B. eine Administratorrolle.</span><span class="sxs-lookup"><span data-stu-id="1a179-109">For example, some actions require an administrator role.</span></span>
* <span data-ttu-id="1a179-110">**Ressourcenbasierte Autorisierung**.</span><span class="sxs-lookup"><span data-stu-id="1a179-110">**Resource-based authorization**.</span></span> <span data-ttu-id="1a179-111">Autorisieren eine Aktion basierend auf einer bestimmten Ressource.</span><span class="sxs-lookup"><span data-stu-id="1a179-111">Authorizing an action based on a particular resource.</span></span> <span data-ttu-id="1a179-112">Jede Ressource hat beispielsweise einen Besitzer.</span><span class="sxs-lookup"><span data-stu-id="1a179-112">For example, every resource has an owner.</span></span> <span data-ttu-id="1a179-113">Der Besitzer kann die Ressource löschen, andere Benutzer können das nicht.</span><span class="sxs-lookup"><span data-stu-id="1a179-113">The owner can delete the resource; other users cannot.</span></span>

<span data-ttu-id="1a179-114">In einer Standard-App wird eine Kombination beider Varianten verwendet.</span><span class="sxs-lookup"><span data-stu-id="1a179-114">A typical app will employ a mix of both.</span></span> <span data-ttu-id="1a179-115">Um beispielsweise eine Ressource zu löschen, muss der Benutzer Besitzer der Ressource *oder* Administrator sein.</span><span class="sxs-lookup"><span data-stu-id="1a179-115">For example, to delete a resource, the user must be the resource owner *or* an admin.</span></span>

## <a name="role-based-authorization"></a><span data-ttu-id="1a179-116">Rollenbasierte Autorisierung</span><span class="sxs-lookup"><span data-stu-id="1a179-116">Role-Based Authorization</span></span>
<span data-ttu-id="1a179-117">Die [Tailspin Surveys][Tailspin]-Anwendung definiert die folgenden Rollen:</span><span class="sxs-lookup"><span data-stu-id="1a179-117">The [Tailspin Surveys][Tailspin] application defines the following roles:</span></span>

* <span data-ttu-id="1a179-118">Administrator.</span><span class="sxs-lookup"><span data-stu-id="1a179-118">Administrator.</span></span> <span data-ttu-id="1a179-119">Kann alle Erstellungs-, Lese-, Aktualisierungs- und Löschaktionen auf alle Umfragen anwenden, die zu diesem Mandanten gehören.</span><span class="sxs-lookup"><span data-stu-id="1a179-119">Can perform all CRUD operations on any survey that belongs to that tenant.</span></span>
* <span data-ttu-id="1a179-120">Creator (Ersteller).</span><span class="sxs-lookup"><span data-stu-id="1a179-120">Creator.</span></span> <span data-ttu-id="1a179-121">Kann neue Umfragen erstellen.</span><span class="sxs-lookup"><span data-stu-id="1a179-121">Can create new surveys</span></span>
* <span data-ttu-id="1a179-122">Reader (Leser).</span><span class="sxs-lookup"><span data-stu-id="1a179-122">Reader.</span></span> <span data-ttu-id="1a179-123">Kann alle Umfragen lesen, die zu diesem Mandanten gehören.</span><span class="sxs-lookup"><span data-stu-id="1a179-123">Can read any surveys that belong to that tenant</span></span>

<span data-ttu-id="1a179-124">Rollen gelten für *Benutzer* der Anwendung.</span><span class="sxs-lookup"><span data-stu-id="1a179-124">Roles apply to *users* of the application.</span></span> <span data-ttu-id="1a179-125">In der Anwendung „Surveys“ ist ein Benutzer entweder Administrator, Ersteller oder Leser.</span><span class="sxs-lookup"><span data-stu-id="1a179-125">In the Surveys application, a user is either an administrator, creator, or reader.</span></span>

<span data-ttu-id="1a179-126">Eine Erläuterung der Definition und Verwaltung von Rollen finden Sie unter [Anwendungsrollen].</span><span class="sxs-lookup"><span data-stu-id="1a179-126">For a discussion of how to define and manage roles, see [Application roles].</span></span>

<span data-ttu-id="1a179-127">Unabhängig davon, wie Sie die Rollen verwalten, ist Ihr Autorisierungscode ähnlich.</span><span class="sxs-lookup"><span data-stu-id="1a179-127">Regardless of how you manage the roles, your authorization code will look similar.</span></span> <span data-ttu-id="1a179-128">ASP.NET Core verfügt über eine Abstraktion namens [Autorisierungsrichtlinien][policies].</span><span class="sxs-lookup"><span data-stu-id="1a179-128">ASP.NET Core has an abstraction called [authorization policies][policies].</span></span> <span data-ttu-id="1a179-129">Mithilfe dieses Features definieren Sie Autorisierungsrichtlinien im Code und wenden anschließend diese Richtlinien auf Controlleraktionen an.</span><span class="sxs-lookup"><span data-stu-id="1a179-129">With this feature, you define authorization policies in code, and then apply those policies to controller actions.</span></span> <span data-ttu-id="1a179-130">Die Richtlinie ist vom Controller entkoppelt.</span><span class="sxs-lookup"><span data-stu-id="1a179-130">The policy is decoupled from the controller.</span></span>

### <a name="create-policies"></a><span data-ttu-id="1a179-131">Erstellen von Richtlinien</span><span class="sxs-lookup"><span data-stu-id="1a179-131">Create policies</span></span>
<span data-ttu-id="1a179-132">Um eine Richtlinie zu definieren, erstellen Sie zuerst eine Klasse, die `IAuthorizationRequirement`implementiert.</span><span class="sxs-lookup"><span data-stu-id="1a179-132">To define a policy, first create a class that implements `IAuthorizationRequirement`.</span></span> <span data-ttu-id="1a179-133">Am einfachsten erfolgt diese über eine Ableitung von `AuthorizationHandler`.</span><span class="sxs-lookup"><span data-stu-id="1a179-133">It's easiest to derive from `AuthorizationHandler`.</span></span> <span data-ttu-id="1a179-134">Untersuchen Sie in der `Handle` -Methode die relevanten Ansprüche.</span><span class="sxs-lookup"><span data-stu-id="1a179-134">In the `Handle` method, examine the relevant claim(s).</span></span>

<span data-ttu-id="1a179-135">Es folgt ein Beispiel aus der Tailspin-Anwendung „Surveys“:</span><span class="sxs-lookup"><span data-stu-id="1a179-135">Here is an example from the Tailspin Surveys application:</span></span>

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

<span data-ttu-id="1a179-136">Diese Klasse definiert die Anforderung für einen Benutzer, eine neue Umfrage zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="1a179-136">This class defines the requirement for a user to create a new survey.</span></span> <span data-ttu-id="1a179-137">Der Benutzer muss die Rolle „SurveyAdmin“ oder „SurveyCreator“ haben.</span><span class="sxs-lookup"><span data-stu-id="1a179-137">The user must be in the SurveyAdmin or SurveyCreator role.</span></span>

<span data-ttu-id="1a179-138">Definieren Sie in der „startup“-Klasse eine benannte Richtlinie, die eine oder mehrere Anforderungen enthält.</span><span class="sxs-lookup"><span data-stu-id="1a179-138">In your startup class, define a named policy that includes one or more requirements.</span></span> <span data-ttu-id="1a179-139">Wenn mehrere Anforderungen vorhanden sind, muss der Benutzer *alle* Anforderungen erfüllen, um autorisiert zu werden.</span><span class="sxs-lookup"><span data-stu-id="1a179-139">If there are multiple requirements, the user must meet *every* requirement to be authorized.</span></span> <span data-ttu-id="1a179-140">Der folgende Code definiert zwei Richtlinien:</span><span class="sxs-lookup"><span data-stu-id="1a179-140">The following code defines two policies:</span></span>

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

<span data-ttu-id="1a179-141">Dieser Code legt auch das Authentifizierungsschema fest, das ASP.NET darüber informiert, welche Authentifizierungsmiddleware ausgeführt werden soll, wenn die Authentifizierung nicht erfolgreich ist.</span><span class="sxs-lookup"><span data-stu-id="1a179-141">This code also sets the authentication scheme, which tells ASP.NET which authentication middleware should run if authorization fails.</span></span> <span data-ttu-id="1a179-142">In diesem Fall geben wir die Cookieauthentifizierungs-Middleware an, weil diese den Benutzer an eine Seite mit der Meldung „Unzulässig“ umleiten kann.</span><span class="sxs-lookup"><span data-stu-id="1a179-142">In this case, we specify the cookie authentication middleware, because the cookie authentication middleware can redirect the user to a "Forbidden" page.</span></span> <span data-ttu-id="1a179-143">Der Speicherort der Seite mit der Meldung „Unzulässig“ ist in der `AccessDeniedPath`-Option für die Cookiemiddleware festgelegt. Informationen dazu finden Sie unter [Konfigurieren der Authentifizierungsmiddleware].</span><span class="sxs-lookup"><span data-stu-id="1a179-143">The location of the Forbidden page is set in the `AccessDeniedPath` option for the cookie middleware; see [Configuring the authentication middleware].</span></span>

### <a name="authorize-controller-actions"></a><span data-ttu-id="1a179-144">Autorisieren von Controlleraktionen</span><span class="sxs-lookup"><span data-stu-id="1a179-144">Authorize controller actions</span></span>
<span data-ttu-id="1a179-145">Um eine Aktion in einem MVC-Controller zu autorisieren, legen Sie abschließend die Richtlinie im `Authorize` -Attribut fest:</span><span class="sxs-lookup"><span data-stu-id="1a179-145">Finally, to authorize an action in an MVC controller, set the policy in the `Authorize` attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

<span data-ttu-id="1a179-146">In früheren Versionen von ASP.NET wird die **Roles** -Eigenschaft für dieses Attribut festgelegt:</span><span class="sxs-lookup"><span data-stu-id="1a179-146">In earlier versions of ASP.NET, you would set the **Roles** property on the attribute:</span></span>

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]
```

<span data-ttu-id="1a179-147">Dies wird in ASP.NET Core noch immer unterstützt, doch gibt es verglichen mit Autorisierungsrichtlinien einige Nachteile:</span><span class="sxs-lookup"><span data-stu-id="1a179-147">This is still supported in ASP.NET Core, but it has some drawbacks compared with authorization policies:</span></span>

* <span data-ttu-id="1a179-148">Es wird von einem bestimmten Anspruchstyp ausgegangen.</span><span class="sxs-lookup"><span data-stu-id="1a179-148">It assumes a particular claim type.</span></span> <span data-ttu-id="1a179-149">Mit Richtlinien können alle Anspruchstypen überprüft werden.</span><span class="sxs-lookup"><span data-stu-id="1a179-149">Policies can check for any claim type.</span></span> <span data-ttu-id="1a179-150">Rollen sind lediglich ein Anspruchstyp.</span><span class="sxs-lookup"><span data-stu-id="1a179-150">Roles are just a type of claim.</span></span>
* <span data-ttu-id="1a179-151">Der Rollenname ist im Attribut hartcodiert.</span><span class="sxs-lookup"><span data-stu-id="1a179-151">The role name is hard-coded into the attribute.</span></span> <span data-ttu-id="1a179-152">Bei Richtlinien befindet sich die Autorisierungslogik zentral an einem Ort, wodurch das Aktualisieren oder Laden von Daten aus Konfigurationseinstellungen erleichtert wird.</span><span class="sxs-lookup"><span data-stu-id="1a179-152">With policies, the authorization logic is all in one place, making it easier to update or even load from configuration settings.</span></span>
* <span data-ttu-id="1a179-153">Richtlinien erlauben komplexere Autorisierungsentscheidungen (z.B. Alter >= 21), die durch einfache Rollenmitgliedschaft nicht ausgedrückt werden können.</span><span class="sxs-lookup"><span data-stu-id="1a179-153">Policies enable more complex authorization decisions (e.g., age >= 21) that can't be expressed by simple role membership.</span></span>

## <a name="resource-based-authorization"></a><span data-ttu-id="1a179-154">Ressourcenbasierte Autorisierung</span><span class="sxs-lookup"><span data-stu-id="1a179-154">Resource based authorization</span></span>
<span data-ttu-id="1a179-155">Eine *ressourcenbasierte Autorisierung* findet dann statt, wenn die Autorisierung von einer bestimmten Ressource abhängig ist, die von einem Vorgang betroffen ist.</span><span class="sxs-lookup"><span data-stu-id="1a179-155">*Resource based authorization* occurs whenever the authorization depends on a specific resource that will be affected by an operation.</span></span> <span data-ttu-id="1a179-156">In der Tailspin-Anwendung „Surveys“ hat jede Umfrage einen Besitzer und 0 bis n Teilnehmer.</span><span class="sxs-lookup"><span data-stu-id="1a179-156">In the Tailspin Surveys application, every survey has an owner and zero-to-many contributors.</span></span>

* <span data-ttu-id="1a179-157">Der Besitzer kann die Umfrage lesen, aktualisieren, löschen, veröffentlichen und ihre Veröffentlichung aufheben.</span><span class="sxs-lookup"><span data-stu-id="1a179-157">The owner can read, update, delete, publish, and unpublish the survey.</span></span>
* <span data-ttu-id="1a179-158">Der Besitzer kann der Umfrage Teilnehmer zuweisen.</span><span class="sxs-lookup"><span data-stu-id="1a179-158">The owner can assign contributors to the survey.</span></span>
* <span data-ttu-id="1a179-159">Teilnehmer können die Umfrage lesen und aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="1a179-159">Contributors can read and update the survey.</span></span>

<span data-ttu-id="1a179-160">Beachten Sie, dass es sich bei „Besitzer“ und „Teilnehmer“ nicht um Anwendungsrollen handelt, die entsprechenden Werte werden für jede Umfrage in der Anwendungsdatenbank gespeichert.</span><span class="sxs-lookup"><span data-stu-id="1a179-160">Note that "owner" and "contributor" are not application roles; they are stored per survey, in the application database.</span></span> <span data-ttu-id="1a179-161">Um beispielsweise zu prüfen, ob ein Benutzer eine Umfrage löschen darf, überprüft die Anwendung, ob der Benutzer der Besitzer dieser Umfrage ist.</span><span class="sxs-lookup"><span data-stu-id="1a179-161">To check whether a user can delete a survey, for example, the app checks whether the user is the owner for that survey.</span></span>

<span data-ttu-id="1a179-162">In ASP.NET Core implementieren Sie die ressourcenbasierte Autorisierung, indem Sie aus **AuthorizationHandler** ableiten und die **Handle**-Methode überschreiben.</span><span class="sxs-lookup"><span data-stu-id="1a179-162">In ASP.NET Core, implement resource-based authorization by deriving from **AuthorizationHandler** and overriding the **Handle** method.</span></span>

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

<span data-ttu-id="1a179-163">Beachten Sie, dass diese Klasse für Umfrageobjekte stark typisiert ist.</span><span class="sxs-lookup"><span data-stu-id="1a179-163">Notice that this class is strongly typed for Survey objects.</span></span>  <span data-ttu-id="1a179-164">Registrieren Sie die Klasse für DI beim Start:</span><span class="sxs-lookup"><span data-stu-id="1a179-164">Register the class for DI on startup:</span></span>

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

<span data-ttu-id="1a179-165">Verwenden Sie zur Durchführung von Autorisierungsüberprüfungen die **IAuthorizationService** -Schnittstelle, die Sie in Ihre Controller einfügen können.</span><span class="sxs-lookup"><span data-stu-id="1a179-165">To perform authorization checks, use the **IAuthorizationService** interface, which you can inject into your controllers.</span></span> <span data-ttu-id="1a179-166">Der folgende Code überprüft, ob ein Benutzer eine Umfrage lesen kann:</span><span class="sxs-lookup"><span data-stu-id="1a179-166">The following code checks whether a user can read a survey:</span></span>

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

<span data-ttu-id="1a179-167">Da wir ein `Survey`-Objekt übergeben, wird jetzt der `SurveyAuthorizationHandler` aufgerufen.</span><span class="sxs-lookup"><span data-stu-id="1a179-167">Because we pass in a `Survey` object, this call will invoke the `SurveyAuthorizationHandler`.</span></span>

<span data-ttu-id="1a179-168">Es empfiehlt sich, in Ihrem Autorisierungscode alle rollen- und ressourcenbasierten Berechtigungen des Benutzers zusammenzuführen und anschließend diesen Berechtigungssatz mit dem gewünschten Vorgang abzugleichen.</span><span class="sxs-lookup"><span data-stu-id="1a179-168">In your authorization code, a good approach is to aggregate all of the user's role-based and resource-based permissions, then check the aggregate set against the desired operation.</span></span>
<span data-ttu-id="1a179-169">Es folgt ein Beispiel aus der App „Surveys“.</span><span class="sxs-lookup"><span data-stu-id="1a179-169">Here is an example from the Surveys app.</span></span> <span data-ttu-id="1a179-170">Die Anwendung definiert mehrere Typen von Berechtigungen:</span><span class="sxs-lookup"><span data-stu-id="1a179-170">The application defines several permission types:</span></span>

* <span data-ttu-id="1a179-171">Administrator</span><span class="sxs-lookup"><span data-stu-id="1a179-171">Admin</span></span>
* <span data-ttu-id="1a179-172">Contributor (Teilnehmer)</span><span class="sxs-lookup"><span data-stu-id="1a179-172">Contributor</span></span>
* <span data-ttu-id="1a179-173">Creator (Ersteller)</span><span class="sxs-lookup"><span data-stu-id="1a179-173">Creator</span></span>
* <span data-ttu-id="1a179-174">Owner (Besitzer)</span><span class="sxs-lookup"><span data-stu-id="1a179-174">Owner</span></span>
* <span data-ttu-id="1a179-175">Reader (Leser)</span><span class="sxs-lookup"><span data-stu-id="1a179-175">Reader</span></span>

<span data-ttu-id="1a179-176">Die Anwendung definiert außerdem eine Reihe möglicher Aktionen für Umfragen:</span><span class="sxs-lookup"><span data-stu-id="1a179-176">The application also defines a set of possible operations on surveys:</span></span>

* <span data-ttu-id="1a179-177">Erstellen</span><span class="sxs-lookup"><span data-stu-id="1a179-177">Create</span></span>
* <span data-ttu-id="1a179-178">Lesen</span><span class="sxs-lookup"><span data-stu-id="1a179-178">Read</span></span>
* <span data-ttu-id="1a179-179">Aktualisieren</span><span class="sxs-lookup"><span data-stu-id="1a179-179">Update</span></span>
* <span data-ttu-id="1a179-180">Löschen</span><span class="sxs-lookup"><span data-stu-id="1a179-180">Delete</span></span>
* <span data-ttu-id="1a179-181">Veröffentlichen</span><span class="sxs-lookup"><span data-stu-id="1a179-181">Publish</span></span>
* <span data-ttu-id="1a179-182">Veröffentlichung aufheben</span><span class="sxs-lookup"><span data-stu-id="1a179-182">Unpublsh</span></span>

<span data-ttu-id="1a179-183">Der folgende Code erstellt eine Liste der Berechtigungen für einen bestimmten Benutzer und eine Umfrage.</span><span class="sxs-lookup"><span data-stu-id="1a179-183">The following code creates a list of permissions for a particular user and survey.</span></span> <span data-ttu-id="1a179-184">Beachten Sie, dass dieser Code sowohl die App-Rollen des Benutzers als auch die Felder für Besitzer/Teilnehmer in der Umfrage untersucht.</span><span class="sxs-lookup"><span data-stu-id="1a179-184">Notice that this code looks at both the user's app roles, and the owner/contributor fields in the survey.</span></span>

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

<span data-ttu-id="1a179-185">In einer mehrinstanzenfähigen Anwendung müssen Sie sicherstellen, dass Berechtigungen nicht an einen anderen Mandanten „durchsickern“.</span><span class="sxs-lookup"><span data-stu-id="1a179-185">In a multi-tenant application, you must ensure that permissions don't "leak" to another tenant's data.</span></span> <span data-ttu-id="1a179-186">In der Surveys-App gilt die Berechtigung für Teilnehmer mandantenübergreifend – Sie können einen Benutzer aus einem anderen Mandanten als Teilnehmer zuweisen.</span><span class="sxs-lookup"><span data-stu-id="1a179-186">In the Surveys app, the Contributor permission is allowed across tenants &mdash; you can assign someone from another tenant as a contriubutor.</span></span> <span data-ttu-id="1a179-187">Die anderen Berechtigungstypen sind auf die Ressourcen beschränkt, die zum Mandanten des Benutzers gehören.</span><span class="sxs-lookup"><span data-stu-id="1a179-187">The other permission types are restricted to resources that belong to that user's tenant.</span></span> <span data-ttu-id="1a179-188">Um diese Anforderung zu erzwingen, prüft der Code die Mandanten-ID, bevor die Berechtigung gewährt wird.</span><span class="sxs-lookup"><span data-stu-id="1a179-188">To enforce this requirement, the code checks the tenant ID before granting the permission.</span></span> <span data-ttu-id="1a179-189">(Das `TenantId`-Feld wird zugewiesen, wenn die Umfrage erstellt wird.)</span><span class="sxs-lookup"><span data-stu-id="1a179-189">(The `TenantId` field as assigned when the survey is created.)</span></span>

<span data-ttu-id="1a179-190">Im nächste Schritt wird der Vorgang (Lesen, Aktualisieren, Löschen usw.) mit den Berechtigungen verglichen.</span><span class="sxs-lookup"><span data-stu-id="1a179-190">The next step is to check the operation (read, update, delete, etc) against the permissions.</span></span> <span data-ttu-id="1a179-191">Die App „Surveys“ implementiert diesen Schritt mithilfe einer Nachschlagetabelle für Funktionen:</span><span class="sxs-lookup"><span data-stu-id="1a179-191">The Surveys app implements this step by using a lookup table of functions:</span></span>

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

<span data-ttu-id="1a179-192">[**Weiter**][web-api]</span><span class="sxs-lookup"><span data-stu-id="1a179-192">[**Next**][web-api]</span></span>

<!-- Links -->
[Tailspin]: tailspin.md

[Anwendungsrollen]: app-roles.md
[Application roles]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[Referenzimplementierung]: tailspin.md
[reference implementation]: tailspin.md
[Konfigurieren der Authentifizierungsmiddleware]: authenticate.md#configure-the-auth-middleware
[Configuring the authentication middleware]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
