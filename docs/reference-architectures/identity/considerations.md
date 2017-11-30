---
title: "Auswählen einer Lösung für die Integration einer lokalen Active Directory-Instanz in Azure"
description: "Vergleich der Referenzarchitekturen für die Integration einer lokalen Active Directory-Instanz in Azure"
ms.date: 04/06/2017
ms.openlocfilehash: 413a5463d90547197c4b6834d353b4ecf61483ee
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a>Auswählen einer Lösung für die Integration einer lokalen Active Directory-Instanz in Azure

In diesem Artikel werden Optionen für die Integration Ihrer lokalen Active Directory-Umgebung (AD) in ein Azure-Netzwerk verglichen. Für jede einzelne Option stellen wir eine Referenzarchitektur und eine zur Bereitstellung geeignete Lösung vor.

Viele Organisationen nutzen [Active Directory Domain Services (AD DS)][active-directory-domain-services], um mit Benutzern, Computern, Anwendungen oder anderen Ressourcen verknüpfte Identitäten zu authentifizieren, die in einer Sicherheitsbegrenzung enthalten sind. Verzeichnis- und Identitätsdienste werden in der Regel lokal gehostet, aber wenn Ihre Anwendung zum Teil lokal und zum Teil in Azure gehostet wird, können beim Senden von Authentifizierungsanforderungen von Azure zum lokalen System Latenzen auftreten. Durch die Implementierung von Verzeichnis- und Identitätsdiensten in Azure können diese Latenzen verringert werden.

Azure stellt zwei Lösungen für die Implementierung von Verzeichnis- und Identitätsdiensten in Azure bereit: 

* Verwenden Sie [Azure AD][azure-active-directory], um eine Active Directory-Domäne in der Cloud zu erstellen und mit Ihrer lokalen Active Directory-Domäne zu verbinden. Mit [Azure AD Connect][azure-ad-connect] können Sie Ihre lokalen Verzeichnisse in Azure AD integrieren.

* Erweitern Sie Ihre vorhandene lokale Active Directory-Infrastruktur auf Azure, indem Sie eine VM in Azure bereitstellen, die AD DS als Domänencontroller ausführt. Diese Architektur wird häufiger verwendet, wenn das lokale Netzwerk und das virtuelle Azure-Netzwerk (VNET) über eine VPN- oder ExpressRoute-Verbindung miteinander verbunden sind. Es sind mehrere Varianten dieser Architektur möglich: 

    - Erstellen einer Domäne in Azure, die dann zu Ihrer lokalen AD-Gesamtstruktur hinzugefügt wird
    - Erstellen einer separaten Gesamtstruktur in Azure, die für Domänen in Ihrer lokalen Gesamtstruktur vertrauenswürdig sind
    - Replizieren einer AD FS-Bereitstellung (Active Directory-Verbunddienste) nach Azure 

In den folgenden Abschnitten werden diese Optionen ausführlicher beschrieben.

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a>Integrieren von lokalen Domänen in Azure AD

Verwenden Sie Azure Active Directory (Azure AD), um eine Domäne in Azure zu erstellen und mit einer lokalen AD-Domäne zu verknüpfen. 

Das Azure AD-Verzeichnis stellt keine Erweiterung eines lokalen Verzeichnisses dar. Es handelt sich vielmehr um eine Kopie, die die gleichen Objekte und Identitäten enthält. Die lokal an diesen Elementen vorgenommenen Änderungen werden in Azure AD kopiert, wohingegen Änderungen in Azure AD nicht wieder zur lokalen Domäne repliziert werden.

Sie können Azure AD auch ohne ein lokales Verzeichnis verwenden. In diesem Fall enthält Azure AD keine Daten, die von einem lokalen Verzeichnis repliziert wurden, sondern verhält sich als primäre Quelle für alle Identitätsinformationen.


**Vorteile**

* Sie müssen keine AD-Infrastruktur in der Cloud verwalten. Azure AD wird vollständig von Microsoft verwaltet und gewartet.
* Azure AD stellt die gleichen Identitätsinformationen bereit, die lokal verfügbar sind.
* Die Authentifizierung kann in Azure durchgeführt werden, wodurch externe Anwendungen und Benutzer nicht so häufig eine Verbindung mit der lokalen Domäne herstellen müssen.

**Herausforderungen**

* Identitätsdienste sind auf Benutzer und Gruppen beschränkt. Es gibt keine Möglichkeit zur Authentifizierung von Dienst- und Computerkonten.
* Sie müssen die Konnektivität mit Ihrer lokalen Domäne konfigurieren, damit Azure AD Directory weiterhin synchronisiert wird. 
* Anwendungen müssen möglicherweise neu geschrieben werden, um eine Authentifizierung über Azure AD zu ermöglichen.

**[Weitere Informationen][aad]**

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a>Verknüpfung von AD DS in Azure mit einer lokalen Gesamtstruktur

Stellen Sie AD DS-Server (AD Domain Services) für Azure bereit. Erstellen Sie eine Domäne in Azure, und fügen Sie sie zu Ihrer lokalen AD-Gesamtstruktur hinzu. 

Ziehen Sie diese Option in Erwägung, wenn Sie AD DS-Features verwenden, die zurzeit nicht von Azure AD implementiert sind. 

**Vorteile**

* AD DS bietet Zugriff auf die gleichen Identitätsinformationen, die lokal verfügbar sind.
* Sie können Benutzer-, Dienst- und Computerkonten lokal und in Azure authentifizieren.
* Sie müssen keine separate AD-Gesamtstruktur verwalten. Die Domäne in Azure kann der lokalen Gesamtstruktur angehören.
* Sie können die von lokalen Gruppenrichtlinienobjekten definierte Gruppenrichtlinie auf die Domäne in Azure anwenden.

**Herausforderungen**

* Sie müssen Ihre eigenen AD DS-Server und -Domänen in der Cloud bereitstellen und verwalten.
* Es gibt möglicherweise gewisse Latenzen bei der Synchronisierung zwischen den Domänenservern in der Cloud und den lokal ausgeführten Servern.

**[Weitere Informationen][ad-ds]**

## <a name="ad-ds-in-azure-with-a-separate-forest"></a>AD DS in Azure mit einer separaten Gesamtstruktur

Stellen Sie AD DS-Server (AD Domain Services) für Azure bereit, erstellen Sie jedoch eine separate Active Directory-[Gesamtstruktur][ad-forest-defn], die von der lokalen Gesamtstruktur getrennt ist. Domänen in Ihrer lokalen Gesamtstruktur sind für diese Gesamtstruktur vertrauenswürdig.

Zu den typischen Einsatzmöglichkeiten dieser Architektur zählen die Verwaltung einer Sicherheitstrennung für Objekte und Identitäten, die in der Cloud gespeichert werden, sowie die Migration einzelner Domänen aus einem lokalen System in die Cloud.

**Vorteile**

* Sie können lokale Identitäten implementieren und von reinen Azure-Identitäten trennen.
* Sie müssen keine Replikation von der lokalen AD-Gesamtstruktur nach Azure durchführen.

**Herausforderungen**

* Eine Authentifizierung in Azure für lokale Identitäten erfordert zusätzliche Netzwerkhops zu lokalen AD-Servern.
* Sie müssen Ihre eigenen AD DS-Server und die Gesamtstruktur in der Cloud bereitstellen und die entsprechenden Vertrauensstellungen zwischen Gesamtstrukturen einrichten.

**[Weitere Informationen][ad-ds-forest]**

## <a name="extend-ad-fs-to-azure"></a>Erweiterung von AD FS auf Azure

Replizieren Sie eine AD FS-Bereitstellung (Active Directory-Verbunddienste) nach Azure, um eine Verbundauthentifizierung und -autorisierung für in Azure ausgeführte Komponenten auszuführen. 

Typische Einsatzmöglichkeiten für diese Architektur sind Folgende:

* Authentifizieren und Autorisieren von Benutzern von Partnerorganisationen
* Ermöglichung einer Benutzerauthentifizierung über Webbrowser, die außerhalb der Firewall der Organisation ausgeführt werden
* Ermöglichung eines Verbindungsaufbaus durch Benutzer über autorisierte externe Geräte wie Mobilgeräte 

**Vorteile**

* Sie können Ansprüche unterstützende Anwendungen nutzen.
* AD FS bietet die Möglichkeit, zur Authentifizierung externen Partnern zu vertrauen.
* Es besteht Kompatibilität mit einem großen Spektrum an Authentifizierungsprotokollen.

**Herausforderungen**

* Sie müssen Ihre eigenen AD DS-, AD FS- und AD FS-Webanwendungsproxy-Server in Azure bereitstellen.
* Die Konfiguration diese Architektur kann komplex sein.

**[Weitere Informationen][adfs]**

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: https://msdn.microsoft.com/library/ms676906.aspx
[adfs]: ./adfs.md

[active-directory-domain-services]: https://technet.microsoft.com/library/dd448614.aspx
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
