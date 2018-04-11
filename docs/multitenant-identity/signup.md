---
title: Registrierung und Onboarding von Mandanten in einer mehrinstanzenfähigen Anwendung
description: Informationen zum Onboarding in einer mehrmandantenfähigen Anwendung
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: claims
pnp.series.next: app-roles
ms.openlocfilehash: dde577d5bab63fb436d52fb4548399d5bd8bb38f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="tenant-sign-up-and-onboarding"></a><span data-ttu-id="0d1a1-103">Registrierung und Onboarding von Mandanten</span><span class="sxs-lookup"><span data-stu-id="0d1a1-103">Tenant sign-up and onboarding</span></span>

<span data-ttu-id="0d1a1-104">[![GitHub](../_images/github.png)-Beispielcode][sample application]</span><span class="sxs-lookup"><span data-stu-id="0d1a1-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="0d1a1-105">In diesem Artikel wird beschrieben, wie ein Prozess für die *Registrierung* bei einer mehrinstanzenfähigen Anwendung implementiert wird, der es einem Kunden ermöglicht, seine Organisation bei Ihrer Anwendung zu registrieren.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-105">This article describes how to implement a *sign-up* process in a multi-tenant application, which allows a customer to sign up their organization for your application.</span></span>
<span data-ttu-id="0d1a1-106">Es gibt mehrere Gründe für das Implementieren eines Registrierungsprozesses:</span><span class="sxs-lookup"><span data-stu-id="0d1a1-106">There are several reasons to implement a sign-up process:</span></span>

* <span data-ttu-id="0d1a1-107">Ermöglichen, dass ein AD-Administrator seine Zustimmung gibt, dass die gesamte Organisation des Kunden die Anwendung nutzt.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-107">Allow an AD admin to consent for the customer's entire organization to use the application.</span></span>
* <span data-ttu-id="0d1a1-108">Erfassen von Kreditkartenzahlungs- oder anderen Kundeninformationen.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-108">Collect credit card payment or other customer information.</span></span>
* <span data-ttu-id="0d1a1-109">Durchführen der pro Mandant einmal erforderlichen Einrichtung für Ihre Anwendung.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-109">Perform any one-time per-tenant setup needed by your application.</span></span>

## <a name="admin-consent-and-azure-ad-permissions"></a><span data-ttu-id="0d1a1-110">Zustimmung des Administrators und Azure AD-Berechtigungen</span><span class="sxs-lookup"><span data-stu-id="0d1a1-110">Admin consent and Azure AD permissions</span></span>
<span data-ttu-id="0d1a1-111">Für die Authentifizierung bei Azure AD benötigt eine Anwendung Zugriff auf das Verzeichnis des Benutzers.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-111">In order to authenticate with Azure AD, an application needs access to the user's directory.</span></span> <span data-ttu-id="0d1a1-112">Die Anwendung benötigt mindestens die Leseberechtigung für das Profil des Benutzers.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-112">At a minimum, the application needs permission to read the user's profile.</span></span> <span data-ttu-id="0d1a1-113">Bei der ersten Anmeldung zeigt Azure AD eine Zustimmungsseite mit einer Auflistung der angeforderten Berechtigungen.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-113">The first time that a user signs in, Azure AD shows a consent page that lists the permissions being requested.</span></span> <span data-ttu-id="0d1a1-114">Durch Klicken auf **Annehmen**erteilt der Benutzer die Zugriffsberechtigung für die Anwendung.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-114">By clicking **Accept**, the user grants permission to the application.</span></span>

<span data-ttu-id="0d1a1-115">Die Zustimmung wird standardmäßig benutzerbezogen gegeben.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-115">By default, consent is granted on a per-user basis.</span></span> <span data-ttu-id="0d1a1-116">Jedem Benutzer, der sich anmeldet, wird die Zustimmungsseite angezeigt.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-116">Every user who signs in sees the consent page.</span></span> <span data-ttu-id="0d1a1-117">Azure AD unterstützt jedoch auch die *Zustimmung des Administrators*, wodurch ein AD-Administrator für eine gesamte Organisation zustimmen kann.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-117">However, Azure AD also supports  *admin consent*, which allows an AD administrator to consent for an entire organization.</span></span>

<span data-ttu-id="0d1a1-118">Bei Wahl der Zustimmung des Administrators besagt die Zustimmungsseite, dass der AD-Administrator die Berechtigung für den gesamten Mandanten erteilt:</span><span class="sxs-lookup"><span data-stu-id="0d1a1-118">When the admin consent flow is used, the consent page states that the AD admin is granting permission on behalf of the entire tenant:</span></span>

![Aufforderung zur Administratorzustimmung](./images/admin-consent.png)

<span data-ttu-id="0d1a1-120">Nachdem der Administrator auf **Annehmen** geklickt hat, können sich andere Benutzern in demselben Mandanten anmelden, und der Zustimmungsbildschirm wird in Azure AD nicht mehr angezeigt.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-120">After the admin clicks **Accept**, other users within the same tenant can sign in, and Azure AD will skip the consent screen.</span></span>

<span data-ttu-id="0d1a1-121">Nur ein AD-Administrator kann die Zustimmung des Administrators erteilen, da die Berechtigung für die gesamte Organisation gewährt wird.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-121">Only an AD administrator can give admin consent, because it grants permission on behalf of the entire organization.</span></span> <span data-ttu-id="0d1a1-122">Wenn ein Nichtadministrator die Authentifizierung mittels Zustimmung des Administrators versucht, zeigt Azure AD eine Fehlermeldung:</span><span class="sxs-lookup"><span data-stu-id="0d1a1-122">If a non-administrator tries to authenticate with the admin consent flow, Azure AD displays an error:</span></span>

![Zustimmungsfehler](./images/consent-error.png)

<span data-ttu-id="0d1a1-124">Wenn die Anwendung zu einem späteren Zeitpunkt zusätzliche Berechtigungen erfordert, muss sich der Kunde erneut registrieren und den aktualisierten Berechtigungen zustimmen.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-124">If the application requires additional permissions at a later point, the customer will need to sign up again and consent to the updated permissions.</span></span>  

## <a name="implementing-tenant-sign-up"></a><span data-ttu-id="0d1a1-125">Implementieren der Registrierung von Mandanten</span><span class="sxs-lookup"><span data-stu-id="0d1a1-125">Implementing tenant sign-up</span></span>
<span data-ttu-id="0d1a1-126">Für die Anwendung [Tailspin Surveys][Tailspin] wurden mehrere Anforderungen an den Registrierungsvorgang definiert:</span><span class="sxs-lookup"><span data-stu-id="0d1a1-126">For the [Tailspin Surveys][Tailspin] application,  we defined several requirements for the sign-up process:</span></span>

* <span data-ttu-id="0d1a1-127">Ein Mandant muss sich registrieren, bevor sich Benutzer anmelden können.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-127">A tenant must sign up before users can sign in.</span></span>
* <span data-ttu-id="0d1a1-128">Für die Registrierung ist die Zustimmung des Administrators erforderlich.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-128">Sign-up uses the admin consent flow.</span></span>
* <span data-ttu-id="0d1a1-129">Bei der Registrierung wird der Mandant des Benutzers der Anwendungsdatenbank hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-129">Sign-up adds the user's tenant to the application database.</span></span>
* <span data-ttu-id="0d1a1-130">Nachdem sich ein Mandant registriert ist, zeigt die Anwendung eine Onboardingseite.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-130">After a tenant signs up, the application shows an onboarding page.</span></span>

<span data-ttu-id="0d1a1-131">In diesem Abschnitt durchlaufen wir unsere Implementierung des Registrierungsprozesses.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-131">In this section, we'll walk through our implementation of the sign-up process.</span></span>
<span data-ttu-id="0d1a1-132">Wichtig ist es, beim Anwendungskonzept den Unterschied zwischen „registrieren“ und „anmelden“ zu verstehen.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-132">It's important to understand that "sign up" versus "sign in" is an application concept.</span></span> <span data-ttu-id="0d1a1-133">Während der Authentifizierung weiß Azure AD eigentlich nicht, ob der Benutzer dabei ist, sich zu registrieren.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-133">During the authentication flow, Azure AD does not inherently know whether the user is in process of signing up.</span></span> <span data-ttu-id="0d1a1-134">Das Nachverfolgen des Kontexts obliegt der Anwendung.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-134">It's up to the application to keep track of the context.</span></span>

<span data-ttu-id="0d1a1-135">Wenn ein anonymer Benutzer die Tailspin-Anwendung „Surveys“ besucht, werden dem Benutzer zwei Schaltflächen angezeigt: eine zum Anmelden und eine zum Registrieren des Unternehmens.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-135">When an anonymous user visits the Surveys application, the user is shown two buttons, one to sign in, and one to "enroll your company" (sign up).</span></span>

![Anmeldeseite der Anwendung](./images/sign-up-page.png)

<span data-ttu-id="0d1a1-137">Diese Schaltflächen lösen Aktionen in der `AccountController`-Klasse aus.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-137">These buttons invoke actions in the `AccountController` class.</span></span>

<span data-ttu-id="0d1a1-138">Die `SignIn` -Aktion gibt ein **ChallegeResult**zurück, was bewirkt, dass die OpenID Connect-Middleware zum Authentifizierungsendpunkt umgeleitet wird.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-138">The `SignIn` action returns a **ChallegeResult**, which causes the OpenID Connect middleware to redirect to the authentication endpoint.</span></span> <span data-ttu-id="0d1a1-139">Dies ist die Standardmethode zum Auslösen der Authentifizierung in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-139">This is the default way to trigger authentication in ASP.NET Core.</span></span>  

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

<span data-ttu-id="0d1a1-140">Vergleichen Sie dies nun mit der `SignUp` -Aktion:</span><span class="sxs-lookup"><span data-stu-id="0d1a1-140">Now compare the `SignUp` action:</span></span>

```csharp
[AllowAnonymous]
public IActionResult SignUp()
{
    var state = new Dictionary<string, string> { { "signup", "true" }};
    return new ChallengeResult(
        OpenIdConnectDefaults.AuthenticationScheme,
        new AuthenticationProperties(state)
        {
            RedirectUri = Url.Action(nameof(SignUpCallback), "Account")
        });
}
```

<span data-ttu-id="0d1a1-141">Die `SignUp`-Aktion gibt wie die `SignIn`-Aktion ein `ChallengeResult` zurück.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-141">Like `SignIn`, the `SignUp` action also returns a `ChallengeResult`.</span></span> <span data-ttu-id="0d1a1-142">Aber dieses Mal fügen wir den `AuthenticationProperties` in the `ChallengeResult`einige Zustandsinformationen hinzu:</span><span class="sxs-lookup"><span data-stu-id="0d1a1-142">But this time, we add a piece of state information to the `AuthenticationProperties` in the `ChallengeResult`:</span></span>

* <span data-ttu-id="0d1a1-143">signup: Ein boolesches Flag, das angibt, dass der Benutzer den Registrierungsprozess gestartet hat.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-143">signup: A Boolean flag, indicating that the user has started the sign-up process.</span></span>

<span data-ttu-id="0d1a1-144">Die Zustandsinformationen in `AuthenticationProperties` werden dem OpenID Connect-Parameter [Status] hinzugefügt, der per Roundtrips den Authentifizierungsvorgang durchläuft.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-144">The state information in `AuthenticationProperties` gets added to the OpenID Connect [state] parameter, which round trips during the authentication flow.</span></span>

![„State“-Parameter](./images/state-parameter.png)

<span data-ttu-id="0d1a1-146">Nachdem der Benutzer in Azure AD authentifiziert und zur Anwendung umgeleitet wurde, enthält das Authentifizierungsticket den Zustand.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-146">After the user authenticates in Azure AD and gets redirected back to the application, the authentication ticket contains the state.</span></span> <span data-ttu-id="0d1a1-147">Wir nutzen diese Tatsache, um sicherzustellen, dass der Wert von „signup“ sich im gesamten Authentifizierungsablauf nicht ändert.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-147">We are using this fact to make sure the "signup" value persists across the entire authentication flow.</span></span>

## <a name="adding-the-admin-consent-prompt"></a><span data-ttu-id="0d1a1-148">Hinzufügen der Aufforderung zur Administratorzustimmung</span><span class="sxs-lookup"><span data-stu-id="0d1a1-148">Adding the admin consent prompt</span></span>
<span data-ttu-id="0d1a1-149">In Azure AD wird der Vorgang zur Administratorzustimmung ausgelöst, indem in der Authentifizierungsanforderung der Abfragezeichenfolge ein „prompt“-Parameter hinzugefügt wird:</span><span class="sxs-lookup"><span data-stu-id="0d1a1-149">In Azure AD, the admin consent flow is triggered by adding a "prompt" parameter to the query string in the authentication request:</span></span>

```
/authorize?prompt=admin_consent&...
```

<span data-ttu-id="0d1a1-150">Die Tailspin-Anwendung „Surveys“ fügt die Aufforderung während des `RedirectToAuthenticationEndpoint` -Ereignisses hinzu.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-150">The Surveys application adds the prompt during the `RedirectToAuthenticationEndpoint` event.</span></span> <span data-ttu-id="0d1a1-151">Dieses Ereignis wird aufgerufen, unmittelbar bevor die Middleware an den Authentifizierungsendpunkt umgeleitet wird.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-151">This event is called right before the middleware redirects to the authentication endpoint.</span></span>

```csharp
public override Task RedirectToAuthenticationEndpoint(RedirectContext context)
{
    if (context.IsSigningUp())
    {
        context.ProtocolMessage.Prompt = "admin_consent";
    }

    _logger.RedirectToIdentityProvider();
    return Task.FromResult(0);
}
```

<span data-ttu-id="0d1a1-152">Durch Festlegen von` ProtocolMessage.Prompt` wird die Middleware aufgefordert, den „prompt“-Parameter der Authentifizierungsanforderung hinzuzufügen.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-152">Setting` ProtocolMessage.Prompt` tells the middleware to add the "prompt" parameter to the authentication request.</span></span>

<span data-ttu-id="0d1a1-153">Beachten Sie, dass die Aufforderung nur während der Registrierung erforderlich ist.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-153">Note that the prompt is only needed during sign-up.</span></span> <span data-ttu-id="0d1a1-154">In der regulären Anmeldung darf sie nicht enthalten sein.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-154">Regular sign-in should not include it.</span></span> <span data-ttu-id="0d1a1-155">Um sie zu unterscheiden, prüfen wir, ob der `signup` -Wert im Authentifizierungsstatus enthalten ist.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-155">To distinguish between them, we check for the `signup` value in the authentication state.</span></span> <span data-ttu-id="0d1a1-156">Die folgende Erweiterungsmethode überprüft diese Bedingung:</span><span class="sxs-lookup"><span data-stu-id="0d1a1-156">The following extension method checks for this condition:</span></span>

```csharp
internal static bool IsSigningUp(this BaseControlContext context)
{
    Guard.ArgumentNotNull(context, nameof(context));

    string signupValue;
    // Check the HTTP context and convert to string
    if ((context.Ticket == null) ||
        (!context.Ticket.Properties.Items.TryGetValue("signup", out signupValue)))
    {
        return false;
    }

    // We have found the value, so see if it's valid
    bool isSigningUp;
    if (!bool.TryParse(signupValue, out isSigningUp))
    {
        // The value for signup is not a valid boolean, throw                
        throw new InvalidOperationException($"'{signupValue}' is an invalid boolean value");
    }

    return isSigningUp;
}
```

## <a name="registering-a-tenant"></a><span data-ttu-id="0d1a1-157">Registrieren eines Mandanten</span><span class="sxs-lookup"><span data-stu-id="0d1a1-157">Registering a Tenant</span></span>
<span data-ttu-id="0d1a1-158">Die Tailspin-Anwendung „Surveys“ speichert Informationen zu jedem Mandanten und Benutzer in der Datenbank.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-158">The Surveys application stores some information about each tenant and user in the application database.</span></span>

![Tabelle „Tenant“](./images/tenant-table.png)

<span data-ttu-id="0d1a1-160">In der Tabelle „Tenant“ ist „IssuerValue“ der Wert des Ausstelleranspruchs für den Mandanten.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-160">In the Tenant table, IssuerValue is the value of the issuer claim for the tenant.</span></span> <span data-ttu-id="0d1a1-161">Bei Azure AD ist dies `https://sts.windows.net/<tentantID>` , ein für jeden Mandanten eindeutiger Wert.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-161">For Azure AD, this is `https://sts.windows.net/<tentantID>` and gives a unique value per tenant.</span></span>

<span data-ttu-id="0d1a1-162">Wenn sich ein neuer Mandant registriert, schreibt die Tailspin-Anwendung „Surveys“ einen Mandantendatensatz in die Datenbank.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-162">When a new tenant signs up, the Surveys application writes a tenant record to the database.</span></span> <span data-ttu-id="0d1a1-163">Dies geschieht im `AuthenticationValidated` -Ereignis.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-163">This happens inside the `AuthenticationValidated` event.</span></span> <span data-ttu-id="0d1a1-164">(Führen Sie diesen Schritt nicht vor diesem Ereignis aus, da das ID-Token noch nicht auf Gültigkeit geprüft wurde und Sie deshalb den Anspruchswerten nicht vertrauen können.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-164">(Don't do it before this event, because the ID token won't be validated yet, so you can't trust the claim values.</span></span> <span data-ttu-id="0d1a1-165">Siehe [Authentifizierung].)</span><span class="sxs-lookup"><span data-stu-id="0d1a1-165">See [Authentication].</span></span>

<span data-ttu-id="0d1a1-166">Hier der entsprechende Code in der Tailspin-Anwendung „Surveys“:</span><span class="sxs-lookup"><span data-stu-id="0d1a1-166">Here is the relevant code from the Surveys application:</span></span>

```csharp
public override async Task TokenValidated(TokenValidatedContext context)
{
    var principal = context.AuthenticationTicket.Principal;
    var userId = principal.GetObjectIdentifierValue();
    var tenantManager = context.HttpContext.RequestServices.GetService<TenantManager>();
    var userManager = context.HttpContext.RequestServices.GetService<UserManager>();
    var issuerValue = principal.GetIssuerValue();
    _logger.AuthenticationValidated(userId, issuerValue);

    // Normalize the claims first.
    NormalizeClaims(principal);
    var tenant = await tenantManager.FindByIssuerValueAsync(issuerValue)
        .ConfigureAwait(false);

    if (context.IsSigningUp())
    {
        if (tenant == null)
        {
            tenant = await SignUpTenantAsync(context, tenantManager)
                .ConfigureAwait(false);
        }

        // In this case, we need to go ahead and set up the user signing us up.
        await CreateOrUpdateUserAsync(context.Ticket, userManager, tenant)
            .ConfigureAwait(false);
    }
    else
    {
        if (tenant == null)
        {
            _logger.UnregisteredUserSignInAttempted(userId, issuerValue);
            throw new SecurityTokenValidationException($"Tenant {issuerValue} is not registered");
        }

        await CreateOrUpdateUserAsync(context.Ticket, userManager, tenant)
            .ConfigureAwait(false);
    }
}
```

<span data-ttu-id="0d1a1-167">Im Code werden folgende Schritte ausgeführt:</span><span class="sxs-lookup"><span data-stu-id="0d1a1-167">This code does the following:</span></span>

1. <span data-ttu-id="0d1a1-168">Überprüfen Sie, ob der „issuer“-Wert des Mandanten bereits in der Datenbank vorhanden ist.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-168">Check if the tenant's issuer value is already in the database.</span></span> <span data-ttu-id="0d1a1-169">Wenn sich der Mandant nicht registriert hat, gibt `FindByIssuerValueAsync` NULL zurück.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-169">If the tenant has not signed up, `FindByIssuerValueAsync` returns null.</span></span>
2. <span data-ttu-id="0d1a1-170">Wenn sich der Benutzer registriert:</span><span class="sxs-lookup"><span data-stu-id="0d1a1-170">If the user is signing up:</span></span>
   1. <span data-ttu-id="0d1a1-171">Fügen Sie den Mandanten der Datenbank hinzu (`SignUpTenantAsync`).</span><span class="sxs-lookup"><span data-stu-id="0d1a1-171">Add the tenant to the database (`SignUpTenantAsync`).</span></span>
   2. <span data-ttu-id="0d1a1-172">Fügen Sie den authentifizierten Benutzer der Datenbank hinzu (`CreateOrUpdateUserAsync`).</span><span class="sxs-lookup"><span data-stu-id="0d1a1-172">Add the authenticated user to the database (`CreateOrUpdateUserAsync`).</span></span>
3. <span data-ttu-id="0d1a1-173">Führen Sie andernfalls den normalen Anmeldungsvorgang aus:</span><span class="sxs-lookup"><span data-stu-id="0d1a1-173">Otherwise complete the normal sign-in flow:</span></span>
   1. <span data-ttu-id="0d1a1-174">Wenn der Aussteller des Mandanten nicht in der Datenbank gefunden wurde, bedeutet dies, dass der Mandant nicht registriert ist und sich der Kunde registrieren muss.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-174">If the tenant's issuer was not found in the database, it means the tenant is not registered, and the customer needs to sign up.</span></span> <span data-ttu-id="0d1a1-175">Lösen Sie in diesem Fall eine Ausnahme aus, damit die Authentifizierung misslingt.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-175">In that case, throw an exception to cause the authentication to fail.</span></span>
   2. <span data-ttu-id="0d1a1-176">Erstellen Sie andernfalls einen Datenbankeintrag für diesen Benutzer, falls noch nicht erfolgt (`CreateOrUpdateUserAsync`).</span><span class="sxs-lookup"><span data-stu-id="0d1a1-176">Otherwise, create a database record for this user, if there isn't one already (`CreateOrUpdateUserAsync`).</span></span>

<span data-ttu-id="0d1a1-177">Hier ist die `SignUpTenantAsync`-Methode, die den Mandanten zur Datenbank hinzufügt.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-177">Here is the `SignUpTenantAsync` method that adds the tenant to the database.</span></span>

```csharp
private async Task<Tenant> SignUpTenantAsync(BaseControlContext context, TenantManager tenantManager)
{
    Guard.ArgumentNotNull(context, nameof(context));
    Guard.ArgumentNotNull(tenantManager, nameof(tenantManager));

    var principal = context.Ticket.Principal;
    var issuerValue = principal.GetIssuerValue();
    var tenant = new Tenant
    {
        IssuerValue = issuerValue,
        Created = DateTimeOffset.UtcNow
    };

    try
    {
        await tenantManager.CreateAsync(tenant)
            .ConfigureAwait(false);
    }
    catch(Exception ex)
    {
        _logger.SignUpTenantFailed(principal.GetObjectIdentifierValue(), issuerValue, ex);
        throw;
    }

    return tenant;
}
```

<span data-ttu-id="0d1a1-178">Es folgt eine Zusammenfassung des gesamten Registrierungsprozesses in der Tailspin-Anwendung „Surveys“:</span><span class="sxs-lookup"><span data-stu-id="0d1a1-178">Here is a summary of the entire sign-up flow in the Surveys application:</span></span>

1. <span data-ttu-id="0d1a1-179">Der Benutzer klickt auf die Schaltfläche **Registrieren** .</span><span class="sxs-lookup"><span data-stu-id="0d1a1-179">The user clicks the **Sign Up** button.</span></span>
2. <span data-ttu-id="0d1a1-180">Die `AccountController.SignUp` -Aktion gibt ein Abfrageergebnis zurück.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-180">The `AccountController.SignUp` action returns a challege result.</span></span>  <span data-ttu-id="0d1a1-181">Der Authentifizierungsstatus enthält den „signup“-Wert.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-181">The authentication state includes "signup" value.</span></span>
3. <span data-ttu-id="0d1a1-182">Fügen Sie im `RedirectToAuthenticationEndpoint`-Ereignis die `admin_consent`-Eingabeaufforderung hinzu.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-182">In the `RedirectToAuthenticationEndpoint` event, add the `admin_consent` prompt.</span></span>
4. <span data-ttu-id="0d1a1-183">Die OpenID Connect-Middleware wird zu Azure AD umgeleitet, der Benutzer wird authentifiziert.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-183">The OpenID Connect middleware redirects to Azure AD and the user authenticates.</span></span>
5. <span data-ttu-id="0d1a1-184">Suchen Sie im `AuthenticationValidated` -Ereignis nach dem Status „signup“.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-184">In the `AuthenticationValidated` event, look for the "signup" state.</span></span>
6. <span data-ttu-id="0d1a1-185">Fügen Sie den Mandanten der Datenbank hinzu.</span><span class="sxs-lookup"><span data-stu-id="0d1a1-185">Add the tenant to the database.</span></span>

<span data-ttu-id="0d1a1-186">[**Weiter**][app roles]</span><span class="sxs-lookup"><span data-stu-id="0d1a1-186">[**Next**][app roles]</span></span>

<!-- Links -->
[app roles]: app-roles.md
[Tailspin]: tailspin.md

[Status]: http://openid.net/specs/openid-connect-core-1_0.html#AuthRequest
[state]: http://openid.net/specs/openid-connect-core-1_0.html#AuthRequest
[Authentifizierung]: authenticate.md
[Authentication]: authenticate.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
