---
title: "Informationen zur Tailspin-Anwendung „Surveys“"
description: "Übersicht über die Tailspin-Anwendung „Surveys“"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: index
pnp.series.next: authenticate
ms.openlocfilehash: 028f7940d2e3cd7e8e629554f8af290ec5fdd184
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="the-tailspin-scenario"></a><span data-ttu-id="71a5f-103">Das Tailspin-Szenario</span><span class="sxs-lookup"><span data-stu-id="71a5f-103">The Tailspin scenario</span></span>

<span data-ttu-id="71a5f-104">[![GitHub](../_images/github.png)-Beispielcode][sample application]</span><span class="sxs-lookup"><span data-stu-id="71a5f-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="71a5f-105">Tailspin ist ein fiktives Unternehmen, das eine SaaS-Anwendung mit dem Namen „Surveys“ entwickelt.</span><span class="sxs-lookup"><span data-stu-id="71a5f-105">Tailspin is a fictitious company that is developing a SaaS application named Surveys.</span></span> <span data-ttu-id="71a5f-106">Diese Anwendung ermöglicht Organisationen das Erstellen und Veröffentlichen von Online-Umfragen.</span><span class="sxs-lookup"><span data-stu-id="71a5f-106">This application enables organizations to create and publish online surveys.</span></span>

* <span data-ttu-id="71a5f-107">Eine Organisation kann sich für die Anwendung registrieren.</span><span class="sxs-lookup"><span data-stu-id="71a5f-107">An organization can sign up for the application.</span></span>
* <span data-ttu-id="71a5f-108">Nach der Registrierung der Organisation können sich Benutzer bei der Anwendung mit ihren Organisationsanmeldeinformationen anmelden.</span><span class="sxs-lookup"><span data-stu-id="71a5f-108">After the organization is signed up, users can sign into the application with their organizational credentials.</span></span>
* <span data-ttu-id="71a5f-109">Benutzer können Umfragen erstellen, bearbeiten und veröffentlichen.</span><span class="sxs-lookup"><span data-stu-id="71a5f-109">Users can create, edit, and publish surveys.</span></span>

> [!NOTE]
> <span data-ttu-id="71a5f-110">Informationen zu den ersten Schritten mit der Anwendung finden Sie unter [Ausführen der Surveys-Anwendung].</span><span class="sxs-lookup"><span data-stu-id="71a5f-110">To get started with the application, see [Run the Surveys application].</span></span>
> 
> 

## <a name="users-can-create-edit-and-view-surveys"></a><span data-ttu-id="71a5f-111">Benutzer können Umfragen erstellen, bearbeiten und anzeigen</span><span class="sxs-lookup"><span data-stu-id="71a5f-111">Users can create, edit, and view surveys</span></span>
<span data-ttu-id="71a5f-112">Ein authentifizierter Benutzer kann alle Umfragen anzeigen, die er erstellt hat oder für die er über Rechte vom Typ „Mitwirkender“ besitzt, und neue Umfragen erstellen.</span><span class="sxs-lookup"><span data-stu-id="71a5f-112">An authenticated user can view all the surveys that he or she has created or has contributor rights to, and create new surveys.</span></span> <span data-ttu-id="71a5f-113">Beachten Sie, dass der Benutzer mit seiner Organisationsidentität angemeldet ist: `bob@contoso.com`.</span><span class="sxs-lookup"><span data-stu-id="71a5f-113">Notice that the user is signed in with his organizational identity, `bob@contoso.com`.</span></span>

![Surveys-App](./images/surveys-screenshot.png)

<span data-ttu-id="71a5f-115">Dieser Screenshot zeigt die Seite „Edit Survey“:</span><span class="sxs-lookup"><span data-stu-id="71a5f-115">This screenshot shows the Edit Survey page:</span></span>

![Umfrage bearbeiten](./images/edit-survey.png)

<span data-ttu-id="71a5f-117">Benutzer können innerhalb desselben Mandanten auch alle von anderen Benutzern erstellten Umfragen anzeigen.</span><span class="sxs-lookup"><span data-stu-id="71a5f-117">Users can also view any surveys created by other users within the same tenant.</span></span>

![Mandantenumfragen](./images/tenant-surveys.png)

## <a name="survey-owners-can-invite-contributors"></a><span data-ttu-id="71a5f-119">Besitzer von Umfragen können Mitwirkende einladen</span><span class="sxs-lookup"><span data-stu-id="71a5f-119">Survey owners can invite contributors</span></span>
<span data-ttu-id="71a5f-120">Wenn ein Benutzer eine Umfrage erstellt, kann er andere Personen als Teilnehmer zur Umfrage einladen.</span><span class="sxs-lookup"><span data-stu-id="71a5f-120">When a user creates a survey, he or she can invite other people to be contributors on the survey.</span></span> <span data-ttu-id="71a5f-121">Teilnehmer können die Umfrage bearbeiten, sie aber nicht löschen oder bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="71a5f-121">Contributors can edit the survey, but cannot delete or publish it.</span></span>  

![Teilnehmer hinzufügen](./images/add-contributor.png)

<span data-ttu-id="71a5f-123">Ein Benutzer kann Teilnehmer aus anderen Mandanten hinzufügen, was die mandantenübergreifende Freigabe von Ressourcen ermöglicht.</span><span class="sxs-lookup"><span data-stu-id="71a5f-123">A user can add contributors from other tenants, which enables cross-tenant sharing of resources.</span></span> <span data-ttu-id="71a5f-124">In diesem Screenshot fügt Bob (`bob@contoso.com`) die Benutzerin Alice (`alice@fabrikam.com`) einer von Bob erstellten Umfrage als Mitwirkende hinzu.</span><span class="sxs-lookup"><span data-stu-id="71a5f-124">In this screenshot, Bob (`bob@contoso.com`) is adding Alice (`alice@fabrikam.com`) as a contributor to a survey that Bob created.</span></span>

<span data-ttu-id="71a5f-125">Wenn sich Alice anmeldet, sieht sie die Umfrage unter „Surveys I can contribute to“.</span><span class="sxs-lookup"><span data-stu-id="71a5f-125">When Alice logs in, she sees the survey listed under "Surveys I can contribute to".</span></span>

![Teilnehmer an Umfrage](./images/contributor.png)

<span data-ttu-id="71a5f-127">Beachten Sie, dass sich Alice an Ihrem eigenen Mandanten und nicht als Gast des Contoso-Mandanten anmeldet.</span><span class="sxs-lookup"><span data-stu-id="71a5f-127">Note that Alice signs into her own tenant, not as a guest of the Contoso tenant.</span></span> <span data-ttu-id="71a5f-128">Alice verfügt nur für diese Umfrage über eine Berechtigung vom Typ „Mitwirkender“ und kann keine anderen Umfragen des Contoso-Mandanten anzeigen.</span><span class="sxs-lookup"><span data-stu-id="71a5f-128">Alice has contributor permissions only for that survey &mdash; she cannot view other surveys from the Contoso tenant.</span></span>

## <a name="architecture"></a><span data-ttu-id="71a5f-129">Architektur</span><span class="sxs-lookup"><span data-stu-id="71a5f-129">Architecture</span></span>
<span data-ttu-id="71a5f-130">Die Anwendung „Surveys“ besteht aus einem Web-Front-End und einem Web-API-Back-End.</span><span class="sxs-lookup"><span data-stu-id="71a5f-130">The Surveys application consists of a web front end and a web API backend.</span></span> <span data-ttu-id="71a5f-131">Beide werden mit [ASP.NET Core] implementiert.</span><span class="sxs-lookup"><span data-stu-id="71a5f-131">Both are implemented using [ASP.NET Core].</span></span>

<span data-ttu-id="71a5f-132">Die Webanwendung nutzt Azure Active Directory (Azure AD) zum Authentifizieren von Benutzern.</span><span class="sxs-lookup"><span data-stu-id="71a5f-132">The web application uses Azure Active Directory (Azure AD) to authenticate users.</span></span> <span data-ttu-id="71a5f-133">Die Webanwendung ruft auch Azure AD auf, um OAuth 2-Zugriffstoken für die Web-API zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="71a5f-133">The web application also calls Azure AD to get OAuth 2 access tokens for the Web API.</span></span> <span data-ttu-id="71a5f-134">Zugriffstoken werden im Azure Redis Cache zwischengespeichert.</span><span class="sxs-lookup"><span data-stu-id="71a5f-134">Access tokens are cached in Azure Redis Cache.</span></span> <span data-ttu-id="71a5f-135">Mit diesem Cache können mehrere Instanzen denselben Tokencache gemeinsam nutzen (z.B. in einer Serverfarm).</span><span class="sxs-lookup"><span data-stu-id="71a5f-135">The cache enables multiple instances to share the same token cache (e.g., in a server farm).</span></span>

![Architektur](./images/architecture.png)

<span data-ttu-id="71a5f-137">[**Weiter**][authentication]</span><span class="sxs-lookup"><span data-stu-id="71a5f-137">[**Next**][authentication]</span></span>

<!-- Links -->

[authentication]: authenticate.md

[Ausführen der Surveys-Anwendung]: ./run-the-app.md
[ASP.NET Core]: /aspnet/core
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
