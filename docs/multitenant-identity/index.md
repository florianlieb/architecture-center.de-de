---
title: Identitätsverwaltung für mehrmandantenfähige Anwendungen
description: Bewährte Methoden für die Authentifizierung, Autorisierung und Identitätsverwaltung mehrmandantenfähiger Apps.
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
# <a name="manage-identity-in-multitenant-applications"></a>Verwalten der Identität in mehrmandantenfähigen Anwendungen

In dieser Artikelreihe werden bewährte Methoden für Mehrmandantenfähigkeit bei Verwendung von Azure AD für die Authentifizierung und Identitätsverwaltung beschrieben.

[![GitHub](../_images/github.png)-Beispielcode][sample application]

Beim Erstellen einer mehrmandantenfähigen Anwendung besteht eine der ersten Herausforderungen in der Verwaltung der Benutzeridentitäten, da jeder Benutzer nun zu einem Mandanten gehört. Beispiel:

* Benutzer melden sich bei der App mit den Anmeldeinformationen ihrer Organisation an.
* Benutzer sollten Zugriff auf Daten der eigenen Organisation haben, dürfen jedoch nicht auf Daten anderer Mandanten zugreifen.
* Eine Organisation kann sich für eine Anwendung anmelden und Organisationsmitgliedern dann Anwendungsrollen zuweisen.

Azure Active Directory (Azure AD) verfügt über einige hervorragende Funktionen, die alle diese Szenarios unterstützen.

Wir haben begleitend für diese Artikelreihe eine vollständige [End-to-End-Implementierung][sample application] einer mehrmandantenfähigen Anwendung erstellt. Die Artikel fassen zusammen, was wir im Verlauf der Erstellung dieser Anwendung gelernt haben. Informationen zu den ersten Schritten mit der Anwendung finden Sie unter [Run the Surveys application][running-the-app] (Ausführen der Surveys-Anwendung).

## <a name="introduction"></a>Einführung

Angenommen, Sie schreiben eine SaaS-Unternehmensanwendung, die in der Cloud gehostet werden soll. Selbstredend hat die Anwendung Benutzer:

![Benutzer](./images/users.png)

Doch diese Benutzer gehören zu Organisationen:

![Organisationsbenutzer](./images/org-users.png)

Beispiel: Tailspin verkauft Abonnements für seine SaaS-Anwendung. Contoso und Fabrikam registrieren Sie sich für die App. Wenn Alice (`alice@contoso`) sich anmeldet, sollte die Anwendung wissen, dass Alice zu Contoso gehört.

* Alice *muss* Zugriff auf Contoso-Daten haben.
* Alice *darf keinen* Zugriff auf Fabrikam-Daten haben.

In dieser Anleitung wird gezeigt, wie Sie Benutzeridentitäten in einer mehrmandantenfähigen Anwendung verwalten und dabei [Azure Active Directory][AzureAD] (Azure AD) für die Anmeldung und Authentifizierung verwenden.

## <a name="what-is-multitenancy"></a>Was bedeutet Mehrmandantenfähigkeit?
Ein *Mandant* ist eine Gruppe von Benutzern. Bei einer SaaS-Anwendung ist der Mandant ein Abonnent oder Kunde der Anwendung. *Mehrmandantenfähigkeit* ist eine Architektur, in der mehrere Mandanten gemeinsam die gleiche physische Instanz der App nutzen. Obwohl Mandanten physische Ressourcen (z. B. VMs oder Speicher) gemeinsam nutzen, erhält jeder Mandant eine eigene logische Instanz der App.

In der Regel werde Anwendungsdaten von den Benutzern innerhalb eines Mandanten, jedoch nicht mit anderen Mandanten gemeinsam genutzt.

![Mehrere Mandanten](./images/multitenant.png)

Vergleichen Sie diese Architektur mit einer Architektur mit nur einem Mandanten, bei der jeder Mandant über eine dedizierte physische Instanz verfügt. Bei einer Architektur mit nur einem Mandanten fügen Sie Mandanten hinzu, indem Sie neue Instanzen der App in Betrieb nehmen.

![Einzelner Mandant](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a>Mehrmandantenfähigkeit und horizontale Skalierung
Zur Skalierung in der Cloud werden im Allgemeinen mehrere physische Instanzen hinzugefügt. Dies wird als *horizontale Skalierung* oder *horizontales Hochskalieren* bezeichnet. Nehmen Sie als Beispiel eine Web-App. Um mehr Datenverkehr zu bewältigen, können Sie weitere Server-VMs hinzufügen und für diese einen Lastenausgleich vornehmen. Jede VM führt eine separate physische Instanz der Web-App aus.

![Lastenausgleich für eine Website](./images/load-balancing.png)

Jede Anforderung kann an eine beliebige Instanz weitergeleitet werden. Zusammen arbeitet das System als eine einzelne logische Instanz. Sie können eine VM entfernen oder eine neue VM hinzufügen, ohne dass sich dies auf Benutzer auswirkt. In dieser Architektur ist jede physische Instanz mehrmandantenfähig, und zum Skalieren fügen Sie weitere Instanzen hinzu. Falls eine Instanz ausfällt, sollte dies keine Auswirkungen auf irgendeinen Mandanten haben.

## <a name="identity-in-a-multitenant-app"></a>Identität in einer mehrmandantenfähigen App
In einer mehrmandantenfähigen App müssen Sie Benutzer im Kontext von Mandanten berücksichtigen.

**Authentifizierung**

* Benutzer melden Sie sich bei der App mit den Anmeldeinformationen für Ihre Organisation an. Sie müssen keine neuen Benutzerprofile für die App erstellen.
* Benutzer innerhalb derselben Organisation gehören zum selben Mandanten.
* Wenn sich ein Benutzer anmeldet, weiß die Anwendung, zu welchem Mandanten der Benutzer gehört.

**Autorisierung**

* Bei der Autorisierung der Aktionen eines Benutzers (z. B. Anzeigen einer Ressource) muss die App den Mandanten des Benutzers berücksichtigen.
* Benutzer können innerhalb der Anwendung Rollen wie „Admin“ oder „Standardbenutzer“ zugewiesen werden. Rollenzuweisungen müssen vom Kunden und nicht vom SaaS-Anbieter verwaltet werden.

**Beispiel:** Alice, eine Mitarbeiterin von Contoso, navigiert in ihrem Browser zur Anwendung und klickt auf die Schaltfläche „Anmelden“. Sie wird zu einer Anmeldeseite umgeleitet, auf der sie ihre Unternehmensanmeldeinformationen (Benutzername und Kennwort) eingibt. An dieser Stelle wird sie in der Anwendung als `alice@contoso.com`angemeldet. Die Anwendung erkennt auch, dass Alice eine Administratorbenutzerin dieser Anwendung ist. Da sie eine Administratorin ist, wird ihr eine Liste aller Ressourcen gezeigt, die zu Contoso gehören. Sie kann jedoch nicht Ressourcen von Fabrikam anzeigen, da sie nur in ihrem Mandanten Administratorin ist.

In dieser Anleitung untersuchen wir insbesondere den Einsatz von Azure AD für die Identitätsverwaltung.

* Wir setzen voraus, dass der Kunde Benutzerprofile (einschließlich Office 365- und Dynamics CRM-Mandanten) in Azure AD speichert.
* Kunden mit einem lokalen Active Directory-Verzeichnis (AD) können [Azure AD Connect][ADConnect] verwenden, um ihr lokales AD-Verzeichnis mit Azure AD zu synchronisieren.

Wenn AD Connect von einem Kunden mit lokalem AD Azure (aufgrund von IT-Unternehmensrichtlinien oder aus anderen Gründen) nicht verwendet werden darf, kann der SaaS-Anbieter über Active Directory-Verbunddienste (AD FS) einen Verbund mit dem AD des Kunden einrichten. Diese Option wird unter [Federating with a customer's AD FS](Herstellen eines Verbunds mit den AD FS eines Kunden) beschrieben.

Andere Aspekte der Mehrmandantenfähigkeit (z. B. Datenpartitionierung, Einzelmandantenkonfiguration usw.) werden in dieser Anleitung nicht behandelt.

[**Weiter**][tailpin]



<!-- Links -->
[ADConnect]: /azure/active-directory/active-directory-aadconnect
[AzureAD]: /azure/active-directory

[Federating with a customer's AD FS]: adfs.md (Herstellen eines Verbunds mit den AD FS eines Kunden)
[tailpin]: tailspin.md

[running-the-app]: ./run-the-app.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
