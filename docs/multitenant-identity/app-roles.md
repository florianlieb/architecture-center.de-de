---
title: Anwendungsrollen
description: Informationen zur Autorisierung mithilfe von Anwendungsrollen
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: signup
pnp.series.next: authorize
ms.openlocfilehash: a39c64f003c26f860086701dd988a8bb21fab5bf
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="application-roles"></a><span data-ttu-id="c74d8-103">Anwendungsrollen</span><span class="sxs-lookup"><span data-stu-id="c74d8-103">Application roles</span></span>

<span data-ttu-id="c74d8-104">[![GitHub](../_images/github.png)-Beispielcode][sample application]</span><span class="sxs-lookup"><span data-stu-id="c74d8-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="c74d8-105">Anwendungsrollen werden verwendet, um Benutzern Berechtigungen zuzuweisen.</span><span class="sxs-lookup"><span data-stu-id="c74d8-105">Application roles are used to assign permissions to users.</span></span> <span data-ttu-id="c74d8-106">Die [Tailspin Surveys][Tailspin]-Anwendung definiert beispielsweise die folgenden Rollen:</span><span class="sxs-lookup"><span data-stu-id="c74d8-106">For example, the [Tailspin Surveys][Tailspin] application defines the following roles:</span></span>

* <span data-ttu-id="c74d8-107">Administrator.</span><span class="sxs-lookup"><span data-stu-id="c74d8-107">Administrator.</span></span> <span data-ttu-id="c74d8-108">Kann alle Erstellungs-, Lese-, Aktualisierungs- und Löschaktionen auf alle Umfragen anwenden, die zu diesem Mandanten gehören.</span><span class="sxs-lookup"><span data-stu-id="c74d8-108">Can perform all CRUD operations on any survey that belongs to that tenant.</span></span>
* <span data-ttu-id="c74d8-109">Creator (Ersteller).</span><span class="sxs-lookup"><span data-stu-id="c74d8-109">Creator.</span></span> <span data-ttu-id="c74d8-110">Kann neue Umfragen erstellen.</span><span class="sxs-lookup"><span data-stu-id="c74d8-110">Can create new surveys.</span></span>
* <span data-ttu-id="c74d8-111">Reader (Leser).</span><span class="sxs-lookup"><span data-stu-id="c74d8-111">Reader.</span></span> <span data-ttu-id="c74d8-112">Kann alle Umfragen lesen, die zu diesem Mandanten gehören.</span><span class="sxs-lookup"><span data-stu-id="c74d8-112">Can read any surveys that belong to that tenant.</span></span>

<span data-ttu-id="c74d8-113">Sie sehen, dass Rollen während der [Autorisierung]in Berechtigungen übersetzt werden.</span><span class="sxs-lookup"><span data-stu-id="c74d8-113">You can see that roles ultimately get translated into permissions, during [authorization].</span></span> <span data-ttu-id="c74d8-114">Doch zuerst stellt sich die Frage, wie Rollen zugewiesen und verwaltet werden.</span><span class="sxs-lookup"><span data-stu-id="c74d8-114">But the first question is how to assign and manage roles.</span></span> <span data-ttu-id="c74d8-115">Es gibt im Wesentlichen drei Möglichkeiten:</span><span class="sxs-lookup"><span data-stu-id="c74d8-115">We identified three main options:</span></span>

* [<span data-ttu-id="c74d8-116">Azure AD-App-Rollen</span><span class="sxs-lookup"><span data-stu-id="c74d8-116">Azure AD App Roles</span></span>](#roles-using-azure-ad-app-roles)
* [<span data-ttu-id="c74d8-117">Azure AD-Sicherheitsgruppen</span><span class="sxs-lookup"><span data-stu-id="c74d8-117">Azure AD security groups</span></span>](#roles-using-azure-ad-security-groups)
* <span data-ttu-id="c74d8-118">[Anwendungsrollen-Manager](#roles-using-an-application-role-manager)</span><span class="sxs-lookup"><span data-stu-id="c74d8-118">[Application role manager](#roles-using-an-application-role-manager).</span></span>

## <a name="roles-using-azure-ad-app-roles"></a><span data-ttu-id="c74d8-119">Rollen auf Grundlage von Azure AD-App-Rollen</span><span class="sxs-lookup"><span data-stu-id="c74d8-119">Roles using Azure AD App Roles</span></span>
<span data-ttu-id="c74d8-120">Dies ist der Ansatz, den wir für die Tailspin-App „Surveys“ gewählt haben.</span><span class="sxs-lookup"><span data-stu-id="c74d8-120">This is the approach that we used in the Tailspin Surveys app.</span></span>

<span data-ttu-id="c74d8-121">Bei diesem Ansatz legt der SaaS-Anbieter Anwendungsrollen fest, indem diese dem Anwendungsmanifest hinzugefügt werden.</span><span class="sxs-lookup"><span data-stu-id="c74d8-121">In this approach, The SaaS provider defines the application roles by adding them to the application manifest.</span></span> <span data-ttu-id="c74d8-122">Nachdem sich ein Kunde registriert hat, werden Benutzern von einem Administrator des AD-Verzeichnisses Rollen zugewiesen.</span><span class="sxs-lookup"><span data-stu-id="c74d8-122">After a customer signs up, an admin for the customer's AD directory assigns users to the roles.</span></span> <span data-ttu-id="c74d8-123">Wenn sich ein Benutzer anmeldet, werden die dem Benutzer zugewiesenen Rollen als Ansprüche gesendet.</span><span class="sxs-lookup"><span data-stu-id="c74d8-123">When a user signs in, the user's assigned roles are sent as claims.</span></span>

> [!NOTE]
> <span data-ttu-id="c74d8-124">Wenn der Kunde über Azure AD Premium verfügt, kann der Administrator eine Sicherheitsgruppe zu einer Rolle zuweisen, und die Mitglieder der Gruppe erben die App-Rolle.</span><span class="sxs-lookup"><span data-stu-id="c74d8-124">If the customer has Azure AD Premium, the admin can assign a security group to a role, and members of the group will inherit the app role.</span></span> <span data-ttu-id="c74d8-125">Dies ist eine praktische Möglichkeit, Rollen zu verwalten, da der Gruppenbesitzer kein AD-Administrator sein muss.</span><span class="sxs-lookup"><span data-stu-id="c74d8-125">This is a convenient way to manage roles, because the group owner doesn't need to be an AD admin.</span></span>
> 
> 

<span data-ttu-id="c74d8-126">Vorteile dieses Ansatzes:</span><span class="sxs-lookup"><span data-stu-id="c74d8-126">Advantages of this approach:</span></span>

* <span data-ttu-id="c74d8-127">Einfaches Programmiermodell.</span><span class="sxs-lookup"><span data-stu-id="c74d8-127">Simple programming model.</span></span>
* <span data-ttu-id="c74d8-128">Rollen sind anwendungsspezifisch.</span><span class="sxs-lookup"><span data-stu-id="c74d8-128">Roles are specific to the application.</span></span> <span data-ttu-id="c74d8-129">Die Rollenansprüche für eine Anwendung werden nicht an eine andere Anwendung gesendet.</span><span class="sxs-lookup"><span data-stu-id="c74d8-129">The role claims for one application are not sent to another application.</span></span>
* <span data-ttu-id="c74d8-130">Wenn der Kunde die Anwendung aus seinem AD-Mandanten entfernt, werden die Rollen entfernt.</span><span class="sxs-lookup"><span data-stu-id="c74d8-130">If the customer removes the application from their AD tenant, the roles go away.</span></span>
* <span data-ttu-id="c74d8-131">Die Anwendung benötigt keinen zusätzlichen Active Directory-Berechtigungen außer der Leseberechtigung für das Profil des Benutzers.</span><span class="sxs-lookup"><span data-stu-id="c74d8-131">The application doesn't need any extra Active Directory permissions, other than reading the user's profile.</span></span>

<span data-ttu-id="c74d8-132">Nachteile:</span><span class="sxs-lookup"><span data-stu-id="c74d8-132">Drawbacks:</span></span>

* <span data-ttu-id="c74d8-133">Kunden ohne Azure AD Premium können Rollen keine Sicherheitsgruppen zuweisen.</span><span class="sxs-lookup"><span data-stu-id="c74d8-133">Customers without Azure AD Premium cannot assign security groups to roles.</span></span> <span data-ttu-id="c74d8-134">Für diese Kunden müssen alle Benutzerzuweisungen durch einen AD-Administrator erfolgen.</span><span class="sxs-lookup"><span data-stu-id="c74d8-134">For these customers, all user assignments must be done by an AD administrator.</span></span>
* <span data-ttu-id="c74d8-135">Wenn Sie eine Back-End-Web-API haben, die von der Web-App getrennt ist, gelten die Rollenzuweisungen für die Web-App nicht für die Web-API.</span><span class="sxs-lookup"><span data-stu-id="c74d8-135">If you have a backend web API, which is separate from the web app, then role assignments for the web app don't apply to the web API.</span></span> <span data-ttu-id="c74d8-136">Informationen zu diesem Punkt finden Sie unter [Schützen einer Back-End-Web-API].</span><span class="sxs-lookup"><span data-stu-id="c74d8-136">For more discussion of this point, see [Securing a backend web API].</span></span>

### <a name="implementation"></a><span data-ttu-id="c74d8-137">Implementierung</span><span class="sxs-lookup"><span data-stu-id="c74d8-137">Implementation</span></span>
<span data-ttu-id="c74d8-138">**Definieren der Rollen.**</span><span class="sxs-lookup"><span data-stu-id="c74d8-138">**Define the roles.**</span></span> <span data-ttu-id="c74d8-139">Der SaaS-Anbieter deklariert die App-Rollen im [Anwendungsmanifest].</span><span class="sxs-lookup"><span data-stu-id="c74d8-139">The SaaS provider declares the app roles in the [application manifest].</span></span> <span data-ttu-id="c74d8-140">Hier ein Beispiel des Manifesteintrags für die App „Surveys“.</span><span class="sxs-lookup"><span data-stu-id="c74d8-140">For example, here is the manifest entry for the Surveys app:</span></span>

```
"appRoles": [
  {
    "allowedMemberTypes": [
      "User"
    ],
    "description": "Creators can create Surveys",
    "displayName": "SurveyCreator",
    "id": "1b4f816e-5eaf-48b9-8613-7923830595ad",
    "isEnabled": true,
    "value": "SurveyCreator"
  },
  {
    "allowedMemberTypes": [
      "User"
    ],
    "description": "Administrators can manage the Surveys in their tenant",
    "displayName": "SurveyAdmin",
    "id": "c20e145e-5459-4a6c-a074-b942bbd4cfe1",
    "isEnabled": true,
    "value": "SurveyAdmin"
  }
],
```

<span data-ttu-id="c74d8-141">Die `value`-Eigenschaft wird im Rollenanspruch angezeigt.</span><span class="sxs-lookup"><span data-stu-id="c74d8-141">The `value`  property appears in the role claim.</span></span> <span data-ttu-id="c74d8-142">Die `id` -Eigenschaft ist der eindeutige Bezeichner der definierten Rolle.</span><span class="sxs-lookup"><span data-stu-id="c74d8-142">The `id` property is the unique identifier for the defined role.</span></span> <span data-ttu-id="c74d8-143">Generieren Sie für `id`immer einen neuen GUID-Wert.</span><span class="sxs-lookup"><span data-stu-id="c74d8-143">Always generate a new GUID value for `id`.</span></span>

<span data-ttu-id="c74d8-144">**Benutzer zuweisen**.</span><span class="sxs-lookup"><span data-stu-id="c74d8-144">**Assign users**.</span></span> <span data-ttu-id="c74d8-145">Wenn sich ein neuer Kunde registriert, wird die Anwendung im AD-Mandanten des Kunden registriert.</span><span class="sxs-lookup"><span data-stu-id="c74d8-145">When a new customer signs up, the application is registered in the customer's AD tenant.</span></span> <span data-ttu-id="c74d8-146">An diesem Punkt kann ein AD-Administrator für diesen Mandanten Benutzer Rollen zuweisen.</span><span class="sxs-lookup"><span data-stu-id="c74d8-146">At this point, an AD admin for that tenant can assign users to roles.</span></span>

> [!NOTE]
> <span data-ttu-id="c74d8-147">Wie zuvor erwähnt, können Kunden mit Azure AD Premium Sicherheitsgruppen zu Rollen zuweisen.</span><span class="sxs-lookup"><span data-stu-id="c74d8-147">As noted earlier, customers with Azure AD Premium can also assign security groups to roles.</span></span>
> 
> 

<span data-ttu-id="c74d8-148">Der folgende Screenshot aus dem Azure-Portal zeigt Benutzer und Gruppen für die Survey-Anwendung.</span><span class="sxs-lookup"><span data-stu-id="c74d8-148">The following screenshot from the Azure portal shows users and groups for the Survey application.</span></span> <span data-ttu-id="c74d8-149">„Admin“ und „Creator“ sind Gruppen, die den Rollen SurveyAdmin bzw. SurveyCreator zugewiesen wurden.</span><span class="sxs-lookup"><span data-stu-id="c74d8-149">Admin and Creator are groups, assigned to SurveyAdmin and SurveyCreator roles respectively.</span></span> <span data-ttu-id="c74d8-150">Alice ist eine Benutzerin, die direkt der SurveyAdmin-Rolle zugewiesen wurde.</span><span class="sxs-lookup"><span data-stu-id="c74d8-150">Alice is a user who was assigned directly to the SurveyAdmin role.</span></span> <span data-ttu-id="c74d8-151">Bob und Charles sind Benutzer, die noch nicht direkt einer Rolle zugewiesen wurden.</span><span class="sxs-lookup"><span data-stu-id="c74d8-151">Bob and Charles are users that have not been directly assigned to a role.</span></span>

![Benutzer und Gruppen](./images/running-the-app/users-and-groups.png)

<span data-ttu-id="c74d8-153">Wie im folgenden Screenshot gezeigt, gehört Charles zur Gruppe „Admin“ und erbt daher die SurveyAdmin-Rolle.</span><span class="sxs-lookup"><span data-stu-id="c74d8-153">As shown in the following screenshot, Charles is part of the Admin group, so he inherits the SurveyAdmin role.</span></span> <span data-ttu-id="c74d8-154">Bob wurde noch keiner Rolle zugewiesen.</span><span class="sxs-lookup"><span data-stu-id="c74d8-154">In the case of Bob, he has not been assigned a role yet.</span></span>

![Gruppenmitglieder von „Admin“](./images/running-the-app/admin-members.png)


> [!NOTE]
> <span data-ttu-id="c74d8-156">Ein alternativer Ansatz besteht darin, dass die Anwendung Rollen programmgesteuert über die Azure AD-Graph-API zuweist.</span><span class="sxs-lookup"><span data-stu-id="c74d8-156">An alternative approach is for the application to assign roles programmatically, using the Azure AD Graph API.</span></span> <span data-ttu-id="c74d8-157">Dazu muss die Anwendung jedoch Schreibberechtigungen für das AD-Verzeichnis des Kunden erhalten.</span><span class="sxs-lookup"><span data-stu-id="c74d8-157">However, this requires the application to obtain write permissions for the customer's AD directory.</span></span> <span data-ttu-id="c74d8-158">Eine Anwendung mit solchen Berechtigungen könnte eine Menge Unheil anrichten – der Kunde muss darauf vertrauen, dass die App keinen Schaden im Verzeichnis verursacht.</span><span class="sxs-lookup"><span data-stu-id="c74d8-158">An application with those permissions could do a lot of mischief &mdash; the customer is trusting the app not to mess up their directory.</span></span> <span data-ttu-id="c74d8-159">Viele Kunden werden einen Zugriff auf dieser Ebene nicht gewähren.</span><span class="sxs-lookup"><span data-stu-id="c74d8-159">Many customers might be unwilling to grant this level of access.</span></span>
> 

<span data-ttu-id="c74d8-160">**Rollenansprüche abrufen**.</span><span class="sxs-lookup"><span data-stu-id="c74d8-160">**Get role claims**.</span></span> <span data-ttu-id="c74d8-161">Wenn sich ein Benutzer anmeldet, empfängt die Anwendung die dem Benutzer zugewiesenen Rollen in einem Anspruch des Typs `http://schemas.microsoft.com/ws/2008/06/identity/claims/role`.</span><span class="sxs-lookup"><span data-stu-id="c74d8-161">When a user signs in, the application receives the user's assigned role(s) in a claim with type `http://schemas.microsoft.com/ws/2008/06/identity/claims/role`.</span></span>  

<span data-ttu-id="c74d8-162">Ein Benutzer kann mehrere Rollen oder keine Rolle haben.</span><span class="sxs-lookup"><span data-stu-id="c74d8-162">A user can have multiple roles, or no role.</span></span> <span data-ttu-id="c74d8-163">Setzen Sie in Ihrem Autorisierungscode nicht voraus, dass der Benutzer genau einen Rollenanspruch aufweist.</span><span class="sxs-lookup"><span data-stu-id="c74d8-163">In your authorization code, don't assume the user has exactly one role claim.</span></span> <span data-ttu-id="c74d8-164">Schreiben Sie stattdessen Code, der prüft, ob ein bestimmter Anspruchswert vorhanden ist:</span><span class="sxs-lookup"><span data-stu-id="c74d8-164">Instead, write code that checks whether a particular claim value is present:</span></span>

```csharp
if (context.User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
```

## <a name="roles-using-azure-ad-security-groups"></a><span data-ttu-id="c74d8-165">Rollen auf Grundlage von Azure AD-Sicherheitsgruppen</span><span class="sxs-lookup"><span data-stu-id="c74d8-165">Roles using Azure AD security groups</span></span>
<span data-ttu-id="c74d8-166">Bei diesem Ansatz werden Rollen als AD-Sicherheitsgruppen dargestellt.</span><span class="sxs-lookup"><span data-stu-id="c74d8-166">In this approach, roles are represented as AD security groups.</span></span> <span data-ttu-id="c74d8-167">Die Anwendung weist Berechtigungen für Benutzer anhand ihrer Mitgliedschaft in Sicherheitsgruppen zu.</span><span class="sxs-lookup"><span data-stu-id="c74d8-167">The application assigns permissions to users based on their security group memberships.</span></span>

<span data-ttu-id="c74d8-168">Vorteile:</span><span class="sxs-lookup"><span data-stu-id="c74d8-168">Advantages:</span></span>

* <span data-ttu-id="c74d8-169">Kunden ohne Azure AD Premium ermöglicht dieser Ansatz das Verwenden von Sicherheitsgruppen zum Verwalten von Rollenzuweisungen.</span><span class="sxs-lookup"><span data-stu-id="c74d8-169">For customers who do not have Azure AD Premium, this approach enables the customer to use security groups to manage role assignments.</span></span>

<span data-ttu-id="c74d8-170">Nachteile:</span><span class="sxs-lookup"><span data-stu-id="c74d8-170">Disadvantages:</span></span>

* <span data-ttu-id="c74d8-171">Komplexität.</span><span class="sxs-lookup"><span data-stu-id="c74d8-171">Complexity.</span></span> <span data-ttu-id="c74d8-172">Da jeder Mandant verschiedene Gruppenansprüche sendet, muss die App für jeden Mandanten nachverfolgen, welche Sicherheitsgruppen zu welchen Anwendungsrollen gehören.</span><span class="sxs-lookup"><span data-stu-id="c74d8-172">Because every tenant sends different group claims, the app must keep track of which security groups correspond to which application roles, for each tenant.</span></span>
* <span data-ttu-id="c74d8-173">Wenn der Kunde die Anwendung aus seinem AD-Mandanten entfernt, verbleiben die Sicherheitsgruppen in seinem AD-Verzeichnis.</span><span class="sxs-lookup"><span data-stu-id="c74d8-173">If the customer removes the application from their AD tenant, the security groups are left in their AD directory.</span></span>

### <a name="implementation"></a><span data-ttu-id="c74d8-174">Implementierung</span><span class="sxs-lookup"><span data-stu-id="c74d8-174">Implementation</span></span>
<span data-ttu-id="c74d8-175">Legen Sie im Anwendungsmanifest die `groupMembershipClaims` -Eigenschaft auf „SecurityGroup“ fest.</span><span class="sxs-lookup"><span data-stu-id="c74d8-175">In the application manifest, set the `groupMembershipClaims` property to "SecurityGroup".</span></span> <span data-ttu-id="c74d8-176">Dies ist erforderlich, um Ansprüche von Gruppenmitgliedschaften aus AAD abzurufen.</span><span class="sxs-lookup"><span data-stu-id="c74d8-176">This is needed to get group membership claims from AAD.</span></span>

```
{
   // ...
   "groupMembershipClaims": "SecurityGroup",
}
```

<span data-ttu-id="c74d8-177">Wenn sich ein neuer Kunde registriert, weist die Anwendung den Kunden an, Sicherheitsgruppen für die Rollen zu erstellen, die von der Anwendung benötigt werden.</span><span class="sxs-lookup"><span data-stu-id="c74d8-177">When a new customer signs up, the application instructs the customer to create security groups for the roles needed by the application.</span></span> <span data-ttu-id="c74d8-178">Der Kunde muss anschließend die Gruppenobjekt-IDs in die Anwendung eingeben.</span><span class="sxs-lookup"><span data-stu-id="c74d8-178">The customer then needs to enter the group object IDs into the application.</span></span> <span data-ttu-id="c74d8-179">Die Anwendung speichert diese in einer Tabelle, die Gruppen-IDs Anwendungsrollen mandantenbezogen zuordnet.</span><span class="sxs-lookup"><span data-stu-id="c74d8-179">The application stores these in a table that maps group IDs to application roles, per tenant.</span></span>

> [!NOTE]
> <span data-ttu-id="c74d8-180">Alternativ dazu könnte die Anwendung die Gruppen über die Azure AD-Graph-API programmgesteuert erstellen.</span><span class="sxs-lookup"><span data-stu-id="c74d8-180">Alternatively, the application could create the groups programmatically, using the Azure AD Graph API.</span></span>  <span data-ttu-id="c74d8-181">Dieses Verfahren wäre weniger fehleranfällig.</span><span class="sxs-lookup"><span data-stu-id="c74d8-181">This would be less error prone.</span></span> <span data-ttu-id="c74d8-182">Dazu muss die Anwendung jedoch Lese- und Schreibberechtigungen für alle Gruppen für das AD-Verzeichnis des Kunden erhalten.</span><span class="sxs-lookup"><span data-stu-id="c74d8-182">However, it requires the application to obtain "read and write all groups" permissions for the customer's AD directory.</span></span> <span data-ttu-id="c74d8-183">Viele Kunden werden einen Zugriff auf dieser Ebene nicht gewähren.</span><span class="sxs-lookup"><span data-stu-id="c74d8-183">Many customers might be unwilling to grant this level of access.</span></span>
> 
> 

<span data-ttu-id="c74d8-184">Wenn sich ein Benutzer anmeldet:</span><span class="sxs-lookup"><span data-stu-id="c74d8-184">When a user signs in:</span></span>

1. <span data-ttu-id="c74d8-185">Die Anwendung empfängt die Gruppen des Benutzers als Ansprüche.</span><span class="sxs-lookup"><span data-stu-id="c74d8-185">The application receives the user's groups as claims.</span></span> <span data-ttu-id="c74d8-186">Der Wert jedes Anspruchs ist die Objekt-ID einer Gruppe.</span><span class="sxs-lookup"><span data-stu-id="c74d8-186">The value of each claim is the object ID of a group.</span></span>
2. <span data-ttu-id="c74d8-187">Azure AD schränkt die Anzahl der Gruppen ein, die im Token gesendet werden.</span><span class="sxs-lookup"><span data-stu-id="c74d8-187">Azure AD limits the number of groups sent in the token.</span></span> <span data-ttu-id="c74d8-188">Wenn die Anzahl der Gruppen diesen Grenzwert überschreitet, sendet Azure AD einen speziellen „Überschreitungsanspruch“.</span><span class="sxs-lookup"><span data-stu-id="c74d8-188">If the number of groups exceeds this limit, Azure AD sends a special "overage" claim.</span></span> <span data-ttu-id="c74d8-189">Wenn diesen Anspruch vorhanden ist, muss die Anwendung die Azure AD Graph-API abfragen, um alle Gruppen abzurufen, denen der Benutzer angehört.</span><span class="sxs-lookup"><span data-stu-id="c74d8-189">If that claim is present, the application must query the Azure AD Graph API to get all of the groups to which that user belongs.</span></span> <span data-ttu-id="c74d8-190">Weitere Informationen finden Sie unter [Autorisierung in Cloudanwendungen mit AD-Gruppen] im Abschnitt „Gruppenanspruchsüberschreitung“.</span><span class="sxs-lookup"><span data-stu-id="c74d8-190">For details, see [Authorization in Cloud Applications using AD Groups], under the section titled "Groups claim overage".</span></span>
3. <span data-ttu-id="c74d8-191">Die Anwendung schlägt die Objekt-IDs in ihrer eigenen Datenbank nach, um die entsprechenden Anwendungsrollen zu suchen, die dem Benutzer zuzuweisen sind.</span><span class="sxs-lookup"><span data-stu-id="c74d8-191">The application looks up the object IDs in its own database, to find the corresponding application roles to assign to the user.</span></span>
4. <span data-ttu-id="c74d8-192">Die Anwendung fügt dem Benutzerprinzipal einen benutzerdefinierten Anspruchswert hinzu, der die Anwendungsrolle ausdrückt.</span><span class="sxs-lookup"><span data-stu-id="c74d8-192">The application adds a custom claim value to the user principal that expresses the application role.</span></span> <span data-ttu-id="c74d8-193">Beispiel: `survey_role` = „SurveyAdmin“.</span><span class="sxs-lookup"><span data-stu-id="c74d8-193">For example: `survey_role` = "SurveyAdmin".</span></span>

<span data-ttu-id="c74d8-194">Autorisierungsrichtlinien müssen den benutzerdefinierten Rollenanspruch und nicht den Gruppenanspruch verwenden.</span><span class="sxs-lookup"><span data-stu-id="c74d8-194">Authorization policies should use the custom role claim, not the group claim.</span></span>

## <a name="roles-using-an-application-role-manager"></a><span data-ttu-id="c74d8-195">Rollen auf Grundlage eines Anwendungsrollen-Managers</span><span class="sxs-lookup"><span data-stu-id="c74d8-195">Roles using an application role manager</span></span>
<span data-ttu-id="c74d8-196">Bei diesem Ansatz werden Anwendungsrollen überhaupt nicht in Azure AD gespeichert.</span><span class="sxs-lookup"><span data-stu-id="c74d8-196">With this approach, application roles are not stored in Azure AD at all.</span></span> <span data-ttu-id="c74d8-197">Stattdessen speichert die Anwendung die Rollenzuweisungen für jeden Benutzer in der eigenen Datenbank, z.B. über die **RoleManager**-Klasse in ASP.NET Identity.</span><span class="sxs-lookup"><span data-stu-id="c74d8-197">Instead, the application stores the role assignments for each user in its own DB &mdash; for example, using the **RoleManager** class in ASP.NET Identity.</span></span>

<span data-ttu-id="c74d8-198">Vorteile:</span><span class="sxs-lookup"><span data-stu-id="c74d8-198">Advantages:</span></span>

* <span data-ttu-id="c74d8-199">Die App hat die vollständige Kontrolle über die Rollen und Benutzerzuweisungen.</span><span class="sxs-lookup"><span data-stu-id="c74d8-199">The app has full control over the roles and user assignments.</span></span>

<span data-ttu-id="c74d8-200">Nachteile:</span><span class="sxs-lookup"><span data-stu-id="c74d8-200">Drawbacks:</span></span>

* <span data-ttu-id="c74d8-201">Komplexer, schwieriger zu verwalten.</span><span class="sxs-lookup"><span data-stu-id="c74d8-201">More complex, harder to maintain.</span></span>
* <span data-ttu-id="c74d8-202">AD-Sicherheitsgruppen können nicht zum Verwalten von Rollenzuweisungen verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="c74d8-202">Cannot use AD security groups to manage role assignments.</span></span>
* <span data-ttu-id="c74d8-203">Benutzerinformationen werden in der Anwendungsdatenbank gespeichert und können die Synchronität mit dem AD-Verzeichnis des Mandanten verlieren, wenn Benutzer hinzugefügt oder entfernt werden.</span><span class="sxs-lookup"><span data-stu-id="c74d8-203">Stores user information in the application database, where it can get out of sync with the tenant's AD directory, as users are added or removed.</span></span>   


<span data-ttu-id="c74d8-204">[**Weiter**][Autorisierung]</span><span class="sxs-lookup"><span data-stu-id="c74d8-204">[**Next**][authorization]</span></span>

<!-- Links -->
[Tailspin]: tailspin.md

[Autorisierung]: authorize.md
[authorization]: authorize.md
[Schützen einer Back-End-Web-API]: web-api.md
[Securing a backend web API]: web-api.md
[Anwendungsmanifest]: /azure/active-directory/active-directory-application-manifest/
[application manifest]: /azure/active-directory/active-directory-application-manifest/
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
