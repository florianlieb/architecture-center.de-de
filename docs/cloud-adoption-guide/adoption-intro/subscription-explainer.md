---
title: "Erläuterungen: Was ist ein Azure-Abonnement?"
description: Hier erhalten Sie Informationen zu Abonnements, Konten und Angeboten von Azure.
author: abuck
ms.openlocfilehash: 7b88e9489b40b100eecb76602b45901566b3f37f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-an-azure-subscription"></a>Erläuterungen: Was ist ein Azure-Abonnement?

In dem erläuternden Artikel [Was ist ein Azure Active Directory-Mandant?](tenant-explainer.md) haben Sie erfahren, dass die digitale Identität für Ihre Organisation in einem Azure Active Directory-Mandanten gespeichert wird. Darüber hinaus haben Sie gelernt, dass Azure beim Authentifizieren von Benutzeranforderungen zum Erstellen, Lesen, Aktualisieren oder Löschen einer Ressource Azure Active Directory vertraut. 

Es ist grundsätzlich klar, warum die Ausführung dieser Vorgänge für eine Ressource beschränkt werden muss: Dadurch wird verhindert, dass nicht authentifizierte und nicht autorisierte Benutzer auf unsere Ressourcen zugreifen. Diese Ressourcenvorgänge besitzen jedoch weitere Eigenschaften, deren Steuerung eine Organisation übernehmen möchte, etwa die Anzahl von Ressourcen, die ein Benutzer oder eine Benutzergruppe erstellen kann, sowie die Kosten für die Ausführung dieser Ressourcen. 

Dieser **Abonnement** genannte Steuerungsmechanismus wird von Azure implementiert. Ein Abonnement gruppiert Benutzer und die von diesen Benutzern erstellten Ressourcen. Alle einzelnen Ressourcen tragen zu einem [Gesamtgrenzwert][subscription-service-limits] für diese bestimmte Ressourcen bei.

Organisationen können Abonnements verwenden, um auf der Grundlage von Benutzern, Teams, Projekten oder mithilfe zahlreicher anderer Strategien Kosten zu verwalten und die Erstellung von Ressourcen zu beschränken. Diese Strategien werden in den Artikeln zur mittleren und fortgeschrittenen Einführungsphase behandelt. 

## <a name="next-steps"></a>Nächste Schritte

* Sie haben hier Informationen zu Azure-Abonnements erhalten. Erfahren Sie nun mehr über das [Erstellen eines Abonnements](subscription.md), bevor Sie Ihre ersten Azure-Ressourcen erstellen.

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/en-us/get-started/
[azure-offers]: https://azure.microsoft.com/en-us/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/en-us/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/en-us/azure/active-directory/sign-up-organization
