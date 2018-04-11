---
title: 'Erläuterungen: Was ist ein Azure-Abonnement?'
description: Hier erhalten Sie Informationen zu Abonnements, Konten und Angeboten von Azure.
author: alexbuckgit
ms.openlocfilehash: 1650d90d6f78b46b7fe4128d2dab6a80bd6cca78
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/23/2018
---
# <a name="explainer-what-is-an-azure-subscription"></a><span data-ttu-id="623d7-103">Erläuterungen: Was ist ein Azure-Abonnement?</span><span class="sxs-lookup"><span data-stu-id="623d7-103">Explainer: What is an Azure subscription?</span></span>

<span data-ttu-id="623d7-104">In dem erläuternden Artikel [Was ist ein Azure Active Directory-Mandant?](tenant-explainer.md) haben Sie erfahren, dass die digitale Identität für Ihre Organisation in einem Azure Active Directory-Mandanten gespeichert wird.</span><span class="sxs-lookup"><span data-stu-id="623d7-104">In the [what is an Azure Active Directory tenant?](tenant-explainer.md) explainer article, you learned that digital identity for your organization is stored in an Azure Active Directory tenant.</span></span> <span data-ttu-id="623d7-105">Darüber hinaus haben Sie gelernt, dass Azure beim Authentifizieren von Benutzeranforderungen zum Erstellen, Lesen, Aktualisieren oder Löschen einer Ressource Azure Active Directory vertraut.</span><span class="sxs-lookup"><span data-stu-id="623d7-105">You also learned that Azure trusts Azure Active Directory to authenticate user requests to create, read, update, or delete a resource.</span></span> 

<span data-ttu-id="623d7-106">Es ist grundsätzlich klar, warum die Ausführung dieser Vorgänge für eine Ressource beschränkt werden muss: Dadurch wird verhindert, dass nicht authentifizierte und nicht autorisierte Benutzer auf unsere Ressourcen zugreifen.</span><span class="sxs-lookup"><span data-stu-id="623d7-106">We fundamentally understand why it's necessary to restrict access to these operations on a resource - to prevent unauthenticated and unauthorized users from accessing our resources.</span></span> <span data-ttu-id="623d7-107">Diese Ressourcenvorgänge besitzen jedoch weitere Eigenschaften, deren Steuerung eine Organisation übernehmen möchte, etwa die Anzahl von Ressourcen, die ein Benutzer oder eine Benutzergruppe erstellen kann, sowie die Kosten für die Ausführung dieser Ressourcen.</span><span class="sxs-lookup"><span data-stu-id="623d7-107">However, these resource operations have other properties that an organization would like to control, such as the number of resources a user or group of users is allowed to create, and, the cost to run those resources.</span></span> 

<span data-ttu-id="623d7-108">Dieser **Abonnement** genannte Steuerungsmechanismus wird von Azure implementiert.</span><span class="sxs-lookup"><span data-stu-id="623d7-108">Azure implements this control, and it is named a **subscription**.</span></span> <span data-ttu-id="623d7-109">Ein Abonnement gruppiert Benutzer und die von diesen Benutzern erstellten Ressourcen.</span><span class="sxs-lookup"><span data-stu-id="623d7-109">A subscription groups together users and the resources that have been created by those users.</span></span> <span data-ttu-id="623d7-110">Alle einzelnen Ressourcen tragen zu einem [Gesamtgrenzwert][subscription-service-limits] für diese bestimmte Ressourcen bei.</span><span class="sxs-lookup"><span data-stu-id="623d7-110">Each of those resources contributes to an [overall limit][subscription-service-limits] on that particular resource.</span></span>

<span data-ttu-id="623d7-111">Organisationen können Abonnements verwenden, um auf der Grundlage von Benutzern, Teams, Projekten oder mithilfe zahlreicher anderer Strategien Kosten zu verwalten und die Erstellung von Ressourcen zu beschränken.</span><span class="sxs-lookup"><span data-stu-id="623d7-111">Organizations can use subscriptions to manage costs and creation of resource by users, teams, projects, or using many other strategies.</span></span> <span data-ttu-id="623d7-112">Diese Strategien werden in den Artikeln zur mittleren und fortgeschrittenen Einführungsphase behandelt.</span><span class="sxs-lookup"><span data-stu-id="623d7-112">These strategies will be discussed in the intermediate and advanced adoption stage articles.</span></span> 

## <a name="next-steps"></a><span data-ttu-id="623d7-113">Nächste Schritte</span><span class="sxs-lookup"><span data-stu-id="623d7-113">Next steps</span></span>

* <span data-ttu-id="623d7-114">Sie haben hier Informationen zu Azure-Abonnements erhalten. Erfahren Sie nun mehr über das [Erstellen eines Abonnements](subscription.md), bevor Sie Ihre ersten Azure-Ressourcen erstellen.</span><span class="sxs-lookup"><span data-stu-id="623d7-114">Now that you have learned about Azure subscriptions, learn more about [creating a subscription](subscription.md) before you create your first Azure resources..</span></span>

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/get-started/
[azure-offers]: https://azure.microsoft.com/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/azure/active-directory/sign-up-organization
