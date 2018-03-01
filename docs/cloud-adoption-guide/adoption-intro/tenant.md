---
title: 'Anleitungen: Entwerfen eines Azure AD-Mandanten'
description: "Leitfaden für den Entwurf von Azure-Mandanten im Rahmen einer Strategie für die grundlegende Cloudeinführung"
author: telmosampaio
ms.openlocfilehash: 9ac52e9fd44bd8b9c777625002d5960f4f269be2
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/27/2018
---
# <a name="guidance-azure-ad-tenant-design"></a>Anleitungen: Entwerfen eines Azure AD-Mandanten

Ein Azure AD-Mandant stellt Dienste für digitale Identitäten und Namespaces für [Azure-Abonnements](subscription-explainer.md) bereit. Wenn Sie sich an die Struktur der grundlegenden Einführung halten, haben Sie bereits gelernt, wie Sie einen [Azure AD-Mandanten erhalten][how-to-get-aad-tenant]. 

## <a name="design-considerations"></a>Überlegungen zum Entwurf

- In der grundlegenden Einführungsphase können Sie mit einem einzelnen Azure AD-Mandanten beginnen. Falls Ihre Organisation über ein Office 365-Abonnement oder Azure-Abonnement verfügt, besitzen Sie bereits einen Azure AD-Mandanten, den Sie verwenden können. Verfügen Sie über keines dieser Abonnements, lesen Sie den Artikel zum [Einrichten eines Azure Active Directory-Mandanten][how-to-get-aad-tenant]. 
- In der mittleren und fortgeschrittenen Einführungsphase erfahren Sie, wie Sie lokale Verzeichnisse mit Azure AD synchronisieren oder für lokale Verzeichnisse einen Verbund mit Azure AD einrichten. Dies ermöglicht die Verwendung lokaler digitaler Identitäten in Azure AD. In der grundlegenden Phase fügen Sie jedoch neue Benutzer hinzu, die nur eine Identität in einem einzelnen Azure AD-Mandanten besitzen. Sie sind für die Verwaltung dieser Identitäten verantwortlich. Sie müssen beispielsweise neue Azure AD-Benutzer integrieren, Azure AD-Benutzer entfernen, die nicht mehr auf Azure-Ressourcen zugreifen dürfen, und andere Änderungen an Benutzerberechtigungen vornehmen.

## <a name="next-steps"></a>Nächste Schritte

* Sie besitzen nun einen Azure AD-Mandanten. Erfahren Sie jetzt, wie Sie [einen Benutzer hinzufügen][azure-ad-add-user]. Wenn Sie neue Benutzer zu Ihrem Azure AD-Mandanten hinzugefügt haben, lesen Sie als Nächstes mehr zu [Azure-Abonnements](subscription-explainer.md).

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
