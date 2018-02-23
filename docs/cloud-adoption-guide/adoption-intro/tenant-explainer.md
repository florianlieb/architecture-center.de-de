---
title: "Erläuterungen: Was ist ein Azure Active Directory-Mandant?"
description: "Hier wird die interne Funktionsweise von Azure Active Directory für die Bereitstellung von Identity-as-a-Service (IDaaS) in Azure erläutert."
author: petertay
ms.openlocfilehash: ce5a33b92047e1f360eee8fcbc7a726bcf8cd19f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-an-azure-active-directory-tenant"></a>Erläuterungen: Was ist ein Azure Active Directory-Mandant?

In dem erläuternden Artikel [Wie funktioniert Azure?](azure-explainer.md) haben Sie gelernt, dass es sich bei Azure um eine Sammlung von Servern und Netzwerkhardwarekomponenten handelt, auf denen im Auftrag von Benutzern virtualisierte Hardware und Software ausgeführt werden. Darüber hinaus haben Sie erfahren, dass auf einigen dieser Server eine verteilte Orchestrierungsanwendung ausgeführt wird, um das Erstellen, Lesen, Aktualisieren und Löschen von Azure-Ressourcen zu verwalten.

Wie zu erwarten, lässt Azure jedoch nicht für jeden Benutzer die Ausführung dieser Vorgänge für eine Ressource zu. Azure schränkt den Zugriff für diese Vorgänge mithilfe eines vertrauenswürdigen Diensts für digitale Identitäten namens **Azure Active Directory** (Azure AD) ein. Azure AD speichert Benutzernamen, Kennwörter, Profildaten und andere Informationen. Azure AD-Benutzer sind in **Mandanten** segmentiert. Ein Mandant ist ein logisches Konstrukt, das eine sichere, dedizierte Instanz von Azure AD darstellt, die in der Regel einer Organisation zugeordnet ist.

Für die Erstellung eines Mandanten erfordert Azure ein **privilegiertes Konto**. Dieses privilegierte Konto ist entweder mit einem Azure-Konto oder mit Enterprise Agreement verknüpft. Diese Konten sind Abrechnungskonstrukte, die nicht in Azure AD, sondern in einer äußert sicheren Abrechnungsdatenbank gespeichert werden. 

Nach der Erstellung des Mandanten wird eine **Mandanten-ID** für den Mandanten erstellt und in einer äußerst sicheren internen Azure AD-Datenbank gespeichert. Der Besitzer eines privilegierten Kontos kann sich dann bei einem Azure-Portal anmelden und Benutzer zum neu erstellten Azure AD-Mandanten hinzufügen. 

Die meisten Unternehmen besitzen bereits mindestens einen Identitätsverwaltungsdienst (in der Regel Active Directory Domain Services (AD DS)). Azure AD kann Benutzeridentitäten aus AD DS synchronisieren oder einen Verbund mit ihnen erstellen, sodass Unternehmen die Identität nicht separat in den beiden Umgebungen verwalten müssen. Dies wird in den Artikeln zur mittleren und fortgeschrittenen Einführungsphase für digitale Identität ausführlicher behandelt.

## <a name="next-steps"></a>Nächste Schritte

* Sie haben hier Informationen zu Azure AD-Mandanten erhalten. In der grundlegenden Einführungsphase sollten Sie sich nun als Erstes mit dem [Einrichten eines Azure Active Directory-Mandanten][how-to-get-aad-tenant] vertraut machen. Sehen Sie sich anschließend den [Leitfaden zum Entwerfen von Azure AD-Mandanten](tenant.md) an.

<!-- Links -->
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json