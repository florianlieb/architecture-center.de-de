---
title: "Identitätsverwaltung für mehrmandantenfähige Anwendungen"
description: "Bewährte Methoden für die Authentifizierung, Autorisierung und Identitätsverwaltung mehrmandantenfähiger Apps."
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.next: tailspin
ms.openlocfilehash: c363ac01e798b522fa95f39586e28fe3af5fae4a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="manage-identity-in-multitenant-applications"></a><span data-ttu-id="97c04-103">Verwalten der Identität in mehrmandantenfähigen Anwendungen</span><span class="sxs-lookup"><span data-stu-id="97c04-103">Manage Identity in Multitenant Applications</span></span>

<span data-ttu-id="97c04-104">In dieser Artikelreihe werden bewährte Methoden für Mehrmandantenfähigkeit bei Verwendung von Azure AD für die Authentifizierung und Identitätsverwaltung beschrieben.</span><span class="sxs-lookup"><span data-stu-id="97c04-104">This series of articles describes best practices for multitenancy, when using Azure AD for authentication and identity management.</span></span>

<span data-ttu-id="97c04-105">[![GitHub](../_images/github.png)-Beispielcode][sample application]</span><span class="sxs-lookup"><span data-stu-id="97c04-105">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="97c04-106">Beim Erstellen einer mehrmandantenfähigen Anwendung besteht eine der ersten Herausforderungen in der Verwaltung der Benutzeridentitäten, da jeder Benutzer nun zu einem Mandanten gehört.</span><span class="sxs-lookup"><span data-stu-id="97c04-106">When you're building a multitenant application, one of the first challenges is managing user identities, because now every user belongs to a tenant.</span></span> <span data-ttu-id="97c04-107">Beispiel:</span><span class="sxs-lookup"><span data-stu-id="97c04-107">For example:</span></span>

* <span data-ttu-id="97c04-108">Benutzer melden sich bei der App mit den Anmeldeinformationen ihrer Organisation an.</span><span class="sxs-lookup"><span data-stu-id="97c04-108">Users sign in with their organizational credentials.</span></span>
* <span data-ttu-id="97c04-109">Benutzer sollten Zugriff auf Daten der eigenen Organisation haben, dürfen jedoch nicht auf Daten anderer Mandanten zugreifen.</span><span class="sxs-lookup"><span data-stu-id="97c04-109">Users should have access to their organization's data, but not data that belongs to other tenants.</span></span>
* <span data-ttu-id="97c04-110">Eine Organisation kann sich für eine Anwendung anmelden und Organisationsmitgliedern dann Anwendungsrollen zuweisen.</span><span class="sxs-lookup"><span data-stu-id="97c04-110">An organization can sign up for the application, and then assign application roles to its members.</span></span>

<span data-ttu-id="97c04-111">Azure Active Directory (Azure AD) verfügt über einige hervorragende Funktionen, die alle diese Szenarios unterstützen.</span><span class="sxs-lookup"><span data-stu-id="97c04-111">Azure Active Directory (Azure AD) has some great features that support all of these scenarios.</span></span>

<span data-ttu-id="97c04-112">Wir haben begleitend für diese Artikelreihe eine vollständige [End-to-End-Implementierung][sample application] einer mehrmandantenfähigen Anwendung erstellt.</span><span class="sxs-lookup"><span data-stu-id="97c04-112">To accompany this series of articles, we created a complete [end-to-end implementation][sample application] of a multitenant application.</span></span> <span data-ttu-id="97c04-113">Die Artikel fassen zusammen, was wir im Verlauf der Erstellung dieser Anwendung gelernt haben.</span><span class="sxs-lookup"><span data-stu-id="97c04-113">The articles reflect what we learned in the process of building the application.</span></span> <span data-ttu-id="97c04-114">Informationen zu den ersten Schritten mit der Anwendung finden Sie unter [Run the Surveys application][running-the-app] (Ausführen der Surveys-Anwendung).</span><span class="sxs-lookup"><span data-stu-id="97c04-114">To get started with the application, see [Run the Surveys application][running-the-app].</span></span>

## <a name="introduction"></a><span data-ttu-id="97c04-115">Einführung</span><span class="sxs-lookup"><span data-stu-id="97c04-115">Introduction</span></span>

<span data-ttu-id="97c04-116">Angenommen, Sie schreiben eine SaaS-Unternehmensanwendung, die in der Cloud gehostet werden soll.</span><span class="sxs-lookup"><span data-stu-id="97c04-116">Let's say you're writing an enterprise SaaS application to be hosted in the cloud.</span></span> <span data-ttu-id="97c04-117">Selbstredend hat die Anwendung Benutzer:</span><span class="sxs-lookup"><span data-stu-id="97c04-117">Of course, the application will have users:</span></span>

![Benutzer](./images/users.png)

<span data-ttu-id="97c04-119">Doch diese Benutzer gehören zu Organisationen:</span><span class="sxs-lookup"><span data-stu-id="97c04-119">But those users belong to organizations:</span></span>

![Organisationsbenutzer](./images/org-users.png)

<span data-ttu-id="97c04-121">Beispiel: Tailspin verkauft Abonnements für seine SaaS-Anwendung.</span><span class="sxs-lookup"><span data-stu-id="97c04-121">Example: Tailspin sells subscriptions to its SaaS application.</span></span> <span data-ttu-id="97c04-122">Contoso und Fabrikam registrieren Sie sich für die App.</span><span class="sxs-lookup"><span data-stu-id="97c04-122">Contoso and Fabrikam sign up for the app.</span></span> <span data-ttu-id="97c04-123">Wenn Alice (`alice@contoso`) sich anmeldet, sollte die Anwendung wissen, dass Alice zu Contoso gehört.</span><span class="sxs-lookup"><span data-stu-id="97c04-123">When Alice (`alice@contoso`) signs in, the application should know that Alice is part of Contoso.</span></span>

* <span data-ttu-id="97c04-124">Alice *muss* Zugriff auf Contoso-Daten haben.</span><span class="sxs-lookup"><span data-stu-id="97c04-124">Alice *should* have access to Contoso data.</span></span>
* <span data-ttu-id="97c04-125">Alice *darf keinen* Zugriff auf Fabrikam-Daten haben.</span><span class="sxs-lookup"><span data-stu-id="97c04-125">Alice *should not* have access to Fabrikam data.</span></span>

<span data-ttu-id="97c04-126">In dieser Anleitung wird gezeigt, wie Sie Benutzeridentitäten in einer mehrmandantenfähigen Anwendung verwalten und dabei [Azure Active Directory][AzureAD] (Azure AD) für die Anmeldung und Authentifizierung verwenden.</span><span class="sxs-lookup"><span data-stu-id="97c04-126">This guidance will show you how to manage user identities in a multitenant application, using [Azure Active Directory][AzureAD] (Azure AD) to handle sign-in and authentication.</span></span>

## <a name="what-is-multitenancy"></a><span data-ttu-id="97c04-127">Was bedeutet Mehrmandantenfähigkeit?</span><span class="sxs-lookup"><span data-stu-id="97c04-127">What is multitenancy?</span></span>
<span data-ttu-id="97c04-128">Ein *Mandant* ist eine Gruppe von Benutzern.</span><span class="sxs-lookup"><span data-stu-id="97c04-128">A *tenant* is a group of users.</span></span> <span data-ttu-id="97c04-129">Bei einer SaaS-Anwendung ist der Mandant ein Abonnent oder Kunde der Anwendung.</span><span class="sxs-lookup"><span data-stu-id="97c04-129">In a SaaS application, the tenant is a subscriber or customer of the application.</span></span> <span data-ttu-id="97c04-130">*Mehrmandantenfähigkeit* ist eine Architektur, in der mehrere Mandanten gemeinsam die gleiche physische Instanz der App nutzen.</span><span class="sxs-lookup"><span data-stu-id="97c04-130">*Multitenancy* is an architecture where multiple tenants share the same physical instance of the app.</span></span> <span data-ttu-id="97c04-131">Obwohl Mandanten physische Ressourcen (z. B. VMs oder Speicher) gemeinsam nutzen, erhält jeder Mandant eine eigene logische Instanz der App.</span><span class="sxs-lookup"><span data-stu-id="97c04-131">Although tenants share physical resources (such as VMs or storage), each tenant gets its own logical instance of the app.</span></span>

<span data-ttu-id="97c04-132">In der Regel werde Anwendungsdaten von den Benutzern innerhalb eines Mandanten, jedoch nicht mit anderen Mandanten gemeinsam genutzt.</span><span class="sxs-lookup"><span data-stu-id="97c04-132">Typically, application data is shared among the users within a tenant, but not with other tenants.</span></span>

![Mehrere Mandanten](./images/multitenant.png)

<span data-ttu-id="97c04-134">Vergleichen Sie diese Architektur mit einer Architektur mit nur einem Mandanten, bei der jeder Mandant über eine dedizierte physische Instanz verfügt.</span><span class="sxs-lookup"><span data-stu-id="97c04-134">Compare this architecture with a single-tenant architecture, where each tenant has a dedicated physical instance.</span></span> <span data-ttu-id="97c04-135">Bei einer Architektur mit nur einem Mandanten fügen Sie Mandanten hinzu, indem Sie neue Instanzen der App in Betrieb nehmen.</span><span class="sxs-lookup"><span data-stu-id="97c04-135">In a single-tenant architecture, you add tenants by spinning up new instances of the app.</span></span>

![Einzelner Mandant](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a><span data-ttu-id="97c04-137">Mehrmandantenfähigkeit und horizontale Skalierung</span><span class="sxs-lookup"><span data-stu-id="97c04-137">Multitenancy and horizontal scaling</span></span>
<span data-ttu-id="97c04-138">Zur Skalierung in der Cloud werden im Allgemeinen mehrere physische Instanzen hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="97c04-138">To achieve scale in the cloud, it’s common to add more physical instances.</span></span> <span data-ttu-id="97c04-139">Dies wird als *horizontale Skalierung* oder *horizontales Hochskalieren* bezeichnet. Nehmen Sie als Beispiel eine Web-App.</span><span class="sxs-lookup"><span data-stu-id="97c04-139">This is known as *horizontal scaling* or *scaling out*. Consider a web app.</span></span> <span data-ttu-id="97c04-140">Um mehr Datenverkehr zu bewältigen, können Sie weitere Server-VMs hinzufügen und für diese einen Lastenausgleich vornehmen.</span><span class="sxs-lookup"><span data-stu-id="97c04-140">To handle more traffic, you can add more server VMs and put them behind a load balancer.</span></span> <span data-ttu-id="97c04-141">Jede VM führt eine separate physische Instanz der Web-App aus.</span><span class="sxs-lookup"><span data-stu-id="97c04-141">Each VM runs a separate physical instance of the web app.</span></span>

![Lastenausgleich für eine Website](./images/load-balancing.png)

<span data-ttu-id="97c04-143">Jede Anforderung kann an eine beliebige Instanz weitergeleitet werden.</span><span class="sxs-lookup"><span data-stu-id="97c04-143">Any request can be routed to any instance.</span></span> <span data-ttu-id="97c04-144">Zusammen arbeitet das System als eine einzelne logische Instanz.</span><span class="sxs-lookup"><span data-stu-id="97c04-144">Together, the system functions as a single logical instance.</span></span> <span data-ttu-id="97c04-145">Sie können eine VM entfernen oder eine neue VM hinzufügen, ohne dass sich dies auf Benutzer auswirkt.</span><span class="sxs-lookup"><span data-stu-id="97c04-145">You can tear down a VM or spin up a new VM, without affecting users.</span></span> <span data-ttu-id="97c04-146">In dieser Architektur ist jede physische Instanz mehrmandantenfähig, und zum Skalieren fügen Sie weitere Instanzen hinzu.</span><span class="sxs-lookup"><span data-stu-id="97c04-146">In this architecture, each physical instance is multi-tenant, and you scale by adding more instances.</span></span> <span data-ttu-id="97c04-147">Falls eine Instanz ausfällt, sollte dies keine Auswirkungen auf irgendeinen Mandanten haben.</span><span class="sxs-lookup"><span data-stu-id="97c04-147">If one instance goes down, it should not affect any tenant.</span></span>

## <a name="identity-in-a-multitenant-app"></a><span data-ttu-id="97c04-148">Identität in einer mehrmandantenfähigen App</span><span class="sxs-lookup"><span data-stu-id="97c04-148">Identity in a multitenant app</span></span>
<span data-ttu-id="97c04-149">In einer mehrmandantenfähigen App müssen Sie Benutzer im Kontext von Mandanten berücksichtigen.</span><span class="sxs-lookup"><span data-stu-id="97c04-149">In a multitenant app, you must consider users in the context of tenants.</span></span>

<span data-ttu-id="97c04-150">**Authentifizierung**</span><span class="sxs-lookup"><span data-stu-id="97c04-150">**Authentication**</span></span>

* <span data-ttu-id="97c04-151">Benutzer melden Sie sich bei der App mit den Anmeldeinformationen für Ihre Organisation an.</span><span class="sxs-lookup"><span data-stu-id="97c04-151">Users sign into the app with their organization credentials.</span></span> <span data-ttu-id="97c04-152">Sie müssen keine neuen Benutzerprofile für die App erstellen.</span><span class="sxs-lookup"><span data-stu-id="97c04-152">They don't have to create new user profiles for the app.</span></span>
* <span data-ttu-id="97c04-153">Benutzer innerhalb derselben Organisation gehören zum selben Mandanten.</span><span class="sxs-lookup"><span data-stu-id="97c04-153">Users within the same organization are part of the same tenant.</span></span>
* <span data-ttu-id="97c04-154">Wenn sich ein Benutzer anmeldet, weiß die Anwendung, zu welchem Mandanten der Benutzer gehört.</span><span class="sxs-lookup"><span data-stu-id="97c04-154">When a user signs in, the application knows which tenant the user belongs to.</span></span>

<span data-ttu-id="97c04-155">**Autorisierung**</span><span class="sxs-lookup"><span data-stu-id="97c04-155">**Authorization**</span></span>

* <span data-ttu-id="97c04-156">Bei der Autorisierung der Aktionen eines Benutzers (z. B. Anzeigen einer Ressource) muss die App den Mandanten des Benutzers berücksichtigen.</span><span class="sxs-lookup"><span data-stu-id="97c04-156">When authorizing a user's actions (say, viewing a resource), the app must take into account the user's tenant.</span></span>
* <span data-ttu-id="97c04-157">Benutzer können innerhalb der Anwendung Rollen wie „Admin“ oder „Standardbenutzer“ zugewiesen werden.</span><span class="sxs-lookup"><span data-stu-id="97c04-157">Users might be assigned roles within the application, such as "Admin" or "Standard User".</span></span> <span data-ttu-id="97c04-158">Rollenzuweisungen müssen vom Kunden und nicht vom SaaS-Anbieter verwaltet werden.</span><span class="sxs-lookup"><span data-stu-id="97c04-158">Role assignments should be managed by the customer, not by the SaaS provider.</span></span>

<span data-ttu-id="97c04-159">**Beispiel:**</span><span class="sxs-lookup"><span data-stu-id="97c04-159">**Example.**</span></span> <span data-ttu-id="97c04-160">Alice, eine Mitarbeiterin von Contoso, navigiert in ihrem Browser zur Anwendung und klickt auf die Schaltfläche „Anmelden“.</span><span class="sxs-lookup"><span data-stu-id="97c04-160">Alice, an employee at Contoso, navigates to the application in her browser and clicks the “Log in” button.</span></span> <span data-ttu-id="97c04-161">Sie wird zu einer Anmeldeseite umgeleitet, auf der sie ihre Unternehmensanmeldeinformationen (Benutzername und Kennwort) eingibt.</span><span class="sxs-lookup"><span data-stu-id="97c04-161">She is redirected to a login screen where she enters her corporate credentials (username and password).</span></span> <span data-ttu-id="97c04-162">An dieser Stelle wird sie in der Anwendung als `alice@contoso.com`angemeldet.</span><span class="sxs-lookup"><span data-stu-id="97c04-162">At this point, she is logged into the app as `alice@contoso.com`.</span></span> <span data-ttu-id="97c04-163">Die Anwendung erkennt auch, dass Alice eine Administratorbenutzerin dieser Anwendung ist.</span><span class="sxs-lookup"><span data-stu-id="97c04-163">The application also knows that Alice is an admin user for this application.</span></span> <span data-ttu-id="97c04-164">Da sie eine Administratorin ist, wird ihr eine Liste aller Ressourcen gezeigt, die zu Contoso gehören.</span><span class="sxs-lookup"><span data-stu-id="97c04-164">Because she is an admin, she can see a list of all the resources that belong to Contoso.</span></span> <span data-ttu-id="97c04-165">Sie kann jedoch nicht Ressourcen von Fabrikam anzeigen, da sie nur in ihrem Mandanten Administratorin ist.</span><span class="sxs-lookup"><span data-stu-id="97c04-165">However, she cannot view Fabrikam's resources, because she is an admin only within her tenant.</span></span>

<span data-ttu-id="97c04-166">In dieser Anleitung untersuchen wir insbesondere den Einsatz von Azure AD für die Identitätsverwaltung.</span><span class="sxs-lookup"><span data-stu-id="97c04-166">In this guidance, we'll look specifically at using Azure AD for identity management.</span></span>

* <span data-ttu-id="97c04-167">Wir setzen voraus, dass der Kunde Benutzerprofile (einschließlich Office 365- und Dynamics CRM-Mandanten) in Azure AD speichert.</span><span class="sxs-lookup"><span data-stu-id="97c04-167">We assume the customer stores their user profiles in Azure AD (including Office365 and Dynamics CRM tenants)</span></span>
* <span data-ttu-id="97c04-168">Kunden mit einem lokalen Active Directory-Verzeichnis (AD) können [Azure AD Connect][ADConnect] verwenden, um ihr lokales AD-Verzeichnis mit Azure AD zu synchronisieren.</span><span class="sxs-lookup"><span data-stu-id="97c04-168">Customers with on-premise Active Directory (AD) can use [Azure AD Connect][ADConnect] to sync their on-premise AD with Azure AD.</span></span>

<span data-ttu-id="97c04-169">Wenn AD Connect von einem Kunden mit lokalem AD Azure (aufgrund von IT-Unternehmensrichtlinien oder aus anderen Gründen) nicht verwendet werden darf, kann der SaaS-Anbieter über Active Directory-Verbunddienste (AD FS) einen Verbund mit dem AD des Kunden einrichten.</span><span class="sxs-lookup"><span data-stu-id="97c04-169">If a customer with on-premise AD cannot use Azure AD Connect (due to corporate IT policy or other reasons), the SaaS provider can federate with the customer's AD through Active Directory Federation Services (AD FS).</span></span> <span data-ttu-id="97c04-170">Diese Option wird unter [Federating with a customer's AD FS](Herstellen eines Verbunds mit den AD FS eines Kunden) beschrieben.</span><span class="sxs-lookup"><span data-stu-id="97c04-170">This option is described in [Federating with a customer's AD FS].</span></span>

<span data-ttu-id="97c04-171">Andere Aspekte der Mehrmandantenfähigkeit (z. B. Datenpartitionierung, Einzelmandantenkonfiguration usw.) werden in dieser Anleitung nicht behandelt.</span><span class="sxs-lookup"><span data-stu-id="97c04-171">This guidance does not consider other aspects of multitenancy such as data partitioning, per-tenant configuration, and so forth.</span></span>

<span data-ttu-id="97c04-172">[**Weiter**][tailpin]</span><span class="sxs-lookup"><span data-stu-id="97c04-172">[**Next**][tailpin]</span></span>



<!-- Links -->
[ADConnect]: /azure/active-directory/active-directory-aadconnect
[AzureAD]: /azure/active-directory

[Federating with a customer's AD FS]: adfs.md (Herstellen eines Verbunds mit den AD FS eines Kunden)
[tailpin]: tailspin.md

[running-the-app]: ./run-the-app.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
