---
title: Auswählen einer Lösung für die Integration einer lokalen Active Directory-Instanz in Azure
description: Vergleich der Referenzarchitekturen für die Integration einer lokalen Active Directory-Instanz in Azure
ms.date: 04/06/2017
ms.openlocfilehash: 413a5463d90547197c4b6834d353b4ecf61483ee
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a><span data-ttu-id="b31ce-103">Auswählen einer Lösung für die Integration einer lokalen Active Directory-Instanz in Azure</span><span class="sxs-lookup"><span data-stu-id="b31ce-103">Choose a solution for integrating on-premises Active Directory with Azure</span></span>

<span data-ttu-id="b31ce-104">In diesem Artikel werden Optionen für die Integration Ihrer lokalen Active Directory-Umgebung (AD) in ein Azure-Netzwerk verglichen.</span><span class="sxs-lookup"><span data-stu-id="b31ce-104">This article compares options for integrating your on-premises Active Directory (AD) environment with an Azure network.</span></span> <span data-ttu-id="b31ce-105">Für jede einzelne Option stellen wir eine Referenzarchitektur und eine zur Bereitstellung geeignete Lösung vor.</span><span class="sxs-lookup"><span data-stu-id="b31ce-105">We provide a reference architecture and a deployable solution for each option.</span></span>

<span data-ttu-id="b31ce-106">Viele Organisationen nutzen [Active Directory Domain Services (AD DS)][active-directory-domain-services], um mit Benutzern, Computern, Anwendungen oder anderen Ressourcen verknüpfte Identitäten zu authentifizieren, die in einer Sicherheitsbegrenzung enthalten sind.</span><span class="sxs-lookup"><span data-stu-id="b31ce-106">Many organizations use [Active Directory Domain Services (AD DS)][active-directory-domain-services] to authenticate identities associated with users, computers, applications, or other resources that are included in a security boundary.</span></span> <span data-ttu-id="b31ce-107">Verzeichnis- und Identitätsdienste werden in der Regel lokal gehostet, aber wenn Ihre Anwendung zum Teil lokal und zum Teil in Azure gehostet wird, können beim Senden von Authentifizierungsanforderungen von Azure zum lokalen System Latenzen auftreten.</span><span class="sxs-lookup"><span data-stu-id="b31ce-107">Directory and identity services are typically hosted on-premises, but if your application is hosted partly on-premises and partly in Azure, there may be latency sending authentication requests from Azure back to on-premises.</span></span> <span data-ttu-id="b31ce-108">Durch die Implementierung von Verzeichnis- und Identitätsdiensten in Azure können diese Latenzen verringert werden.</span><span class="sxs-lookup"><span data-stu-id="b31ce-108">Implementing directory and identity services in Azure can reduce this latency.</span></span>

<span data-ttu-id="b31ce-109">Azure stellt zwei Lösungen für die Implementierung von Verzeichnis- und Identitätsdiensten in Azure bereit:</span><span class="sxs-lookup"><span data-stu-id="b31ce-109">Azure provides two solutions for implementing directory and identity services in Azure:</span></span> 

* <span data-ttu-id="b31ce-110">Verwenden Sie [Azure AD][azure-active-directory], um eine Active Directory-Domäne in der Cloud zu erstellen und mit Ihrer lokalen Active Directory-Domäne zu verbinden.</span><span class="sxs-lookup"><span data-stu-id="b31ce-110">Use [Azure AD][azure-active-directory] to create an Active Directory domain in the cloud and connect it to your on-premises Active Directory domain.</span></span> <span data-ttu-id="b31ce-111">Mit [Azure AD Connect][azure-ad-connect] können Sie Ihre lokalen Verzeichnisse in Azure AD integrieren.</span><span class="sxs-lookup"><span data-stu-id="b31ce-111">[Azure AD Connect][azure-ad-connect] integrates your on-premises directories with Azure AD.</span></span>

* <span data-ttu-id="b31ce-112">Erweitern Sie Ihre vorhandene lokale Active Directory-Infrastruktur auf Azure, indem Sie eine VM in Azure bereitstellen, die AD DS als Domänencontroller ausführt.</span><span class="sxs-lookup"><span data-stu-id="b31ce-112">Extend your existing on-premises Active Directory infrastructure to Azure, by deploying a VM in Azure that runs AD DS as a domain controller.</span></span> <span data-ttu-id="b31ce-113">Diese Architektur wird häufiger verwendet, wenn das lokale Netzwerk und das virtuelle Azure-Netzwerk (VNET) über eine VPN- oder ExpressRoute-Verbindung miteinander verbunden sind.</span><span class="sxs-lookup"><span data-stu-id="b31ce-113">This architecture is more common when the on-premises network and the Azure virtual network (VNet) are connected by a VPN or ExpressRoute connection.</span></span> <span data-ttu-id="b31ce-114">Es sind mehrere Varianten dieser Architektur möglich:</span><span class="sxs-lookup"><span data-stu-id="b31ce-114">Several variations of this architecture are possible:</span></span> 

    - <span data-ttu-id="b31ce-115">Erstellen einer Domäne in Azure, die dann zu Ihrer lokalen AD-Gesamtstruktur hinzugefügt wird</span><span class="sxs-lookup"><span data-stu-id="b31ce-115">Create a domain in Azure and join it to your on-premises AD forest.</span></span>
    - <span data-ttu-id="b31ce-116">Erstellen einer separaten Gesamtstruktur in Azure, die für Domänen in Ihrer lokalen Gesamtstruktur vertrauenswürdig sind</span><span class="sxs-lookup"><span data-stu-id="b31ce-116">Create a separate forest in Azure that is trusted by domains in your on-premises forest.</span></span>
    - <span data-ttu-id="b31ce-117">Replizieren einer AD FS-Bereitstellung (Active Directory-Verbunddienste) nach Azure</span><span class="sxs-lookup"><span data-stu-id="b31ce-117">Replicate an Active Directory Federation Services (AD FS) deployment to Azure.</span></span> 

<span data-ttu-id="b31ce-118">In den folgenden Abschnitten werden diese Optionen ausführlicher beschrieben.</span><span class="sxs-lookup"><span data-stu-id="b31ce-118">The next sections describe each of these options in more detail.</span></span>

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a><span data-ttu-id="b31ce-119">Integrieren von lokalen Domänen in Azure AD</span><span class="sxs-lookup"><span data-stu-id="b31ce-119">Integrate your on-premises domains with Azure AD</span></span>

<span data-ttu-id="b31ce-120">Verwenden Sie Azure Active Directory (Azure AD), um eine Domäne in Azure zu erstellen und mit einer lokalen AD-Domäne zu verknüpfen.</span><span class="sxs-lookup"><span data-stu-id="b31ce-120">Use Azure Active Directory (Azure AD) to create a domain in Azure and link it to an on-premises AD domain.</span></span> 

<span data-ttu-id="b31ce-121">Das Azure AD-Verzeichnis stellt keine Erweiterung eines lokalen Verzeichnisses dar.</span><span class="sxs-lookup"><span data-stu-id="b31ce-121">The Azure AD directory is not an extension of an on-premises directory.</span></span> <span data-ttu-id="b31ce-122">Es handelt sich vielmehr um eine Kopie, die die gleichen Objekte und Identitäten enthält.</span><span class="sxs-lookup"><span data-stu-id="b31ce-122">Rather, it's a copy that contains the same objects and identities.</span></span> <span data-ttu-id="b31ce-123">Die lokal an diesen Elementen vorgenommenen Änderungen werden in Azure AD kopiert, wohingegen Änderungen in Azure AD nicht wieder zur lokalen Domäne repliziert werden.</span><span class="sxs-lookup"><span data-stu-id="b31ce-123">Changes made to these items on-premises are copied to Azure AD, but changes made in Azure AD are not replicated back to the on-premises domain.</span></span>

<span data-ttu-id="b31ce-124">Sie können Azure AD auch ohne ein lokales Verzeichnis verwenden.</span><span class="sxs-lookup"><span data-stu-id="b31ce-124">You can also use Azure AD without using an on-premises directory.</span></span> <span data-ttu-id="b31ce-125">In diesem Fall enthält Azure AD keine Daten, die von einem lokalen Verzeichnis repliziert wurden, sondern verhält sich als primäre Quelle für alle Identitätsinformationen.</span><span class="sxs-lookup"><span data-stu-id="b31ce-125">In this case, Azure AD acts as the primary source of all identity information, rather than containing data replicated from an on-premises directory.</span></span>


<span data-ttu-id="b31ce-126">**Vorteile**</span><span class="sxs-lookup"><span data-stu-id="b31ce-126">**Benefits**</span></span>

* <span data-ttu-id="b31ce-127">Sie müssen keine AD-Infrastruktur in der Cloud verwalten.</span><span class="sxs-lookup"><span data-stu-id="b31ce-127">You don't need to maintain an AD infrastructure in the cloud.</span></span> <span data-ttu-id="b31ce-128">Azure AD wird vollständig von Microsoft verwaltet und gewartet.</span><span class="sxs-lookup"><span data-stu-id="b31ce-128">Azure AD is entirely managed and maintained by Microsoft.</span></span>
* <span data-ttu-id="b31ce-129">Azure AD stellt die gleichen Identitätsinformationen bereit, die lokal verfügbar sind.</span><span class="sxs-lookup"><span data-stu-id="b31ce-129">Azure AD provides the same identity information that is available on-premises.</span></span>
* <span data-ttu-id="b31ce-130">Die Authentifizierung kann in Azure durchgeführt werden, wodurch externe Anwendungen und Benutzer nicht so häufig eine Verbindung mit der lokalen Domäne herstellen müssen.</span><span class="sxs-lookup"><span data-stu-id="b31ce-130">Authentication can happen in Azure, reducing the need for external applications and users to contact the on-premises domain.</span></span>

<span data-ttu-id="b31ce-131">**Herausforderungen**</span><span class="sxs-lookup"><span data-stu-id="b31ce-131">**Challenges**</span></span>

* <span data-ttu-id="b31ce-132">Identitätsdienste sind auf Benutzer und Gruppen beschränkt.</span><span class="sxs-lookup"><span data-stu-id="b31ce-132">Identity services are limited to users and groups.</span></span> <span data-ttu-id="b31ce-133">Es gibt keine Möglichkeit zur Authentifizierung von Dienst- und Computerkonten.</span><span class="sxs-lookup"><span data-stu-id="b31ce-133">There is no ability to authenticate service and computer accounts.</span></span>
* <span data-ttu-id="b31ce-134">Sie müssen die Konnektivität mit Ihrer lokalen Domäne konfigurieren, damit Azure AD Directory weiterhin synchronisiert wird.</span><span class="sxs-lookup"><span data-stu-id="b31ce-134">You must configure connectivity with your on-premises domain to keep the Azure AD directory synchronized.</span></span> 
* <span data-ttu-id="b31ce-135">Anwendungen müssen möglicherweise neu geschrieben werden, um eine Authentifizierung über Azure AD zu ermöglichen.</span><span class="sxs-lookup"><span data-stu-id="b31ce-135">Applications may need to be rewritten to enable authentication through Azure AD.</span></span>

<span data-ttu-id="b31ce-136">**[Weitere Informationen][aad]**</span><span class="sxs-lookup"><span data-stu-id="b31ce-136">**[Read more...][aad]**</span></span>

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a><span data-ttu-id="b31ce-137">Verknüpfung von AD DS in Azure mit einer lokalen Gesamtstruktur</span><span class="sxs-lookup"><span data-stu-id="b31ce-137">AD DS in Azure joined to an on-premises forest</span></span>

<span data-ttu-id="b31ce-138">Stellen Sie AD DS-Server (AD Domain Services) für Azure bereit.</span><span class="sxs-lookup"><span data-stu-id="b31ce-138">Deploy AD Domain Services (AD DS) servers to Azure.</span></span> <span data-ttu-id="b31ce-139">Erstellen Sie eine Domäne in Azure, und fügen Sie sie zu Ihrer lokalen AD-Gesamtstruktur hinzu.</span><span class="sxs-lookup"><span data-stu-id="b31ce-139">Create a domain in Azure and join it to your on-premises AD forest.</span></span> 

<span data-ttu-id="b31ce-140">Ziehen Sie diese Option in Erwägung, wenn Sie AD DS-Features verwenden, die zurzeit nicht von Azure AD implementiert sind.</span><span class="sxs-lookup"><span data-stu-id="b31ce-140">Consider this option if you need to use AD DS features that are not currently implemented by Azure AD.</span></span> 

<span data-ttu-id="b31ce-141">**Vorteile**</span><span class="sxs-lookup"><span data-stu-id="b31ce-141">**Benefits**</span></span>

* <span data-ttu-id="b31ce-142">AD DS bietet Zugriff auf die gleichen Identitätsinformationen, die lokal verfügbar sind.</span><span class="sxs-lookup"><span data-stu-id="b31ce-142">Provides access to the same identity information that is available on-premises.</span></span>
* <span data-ttu-id="b31ce-143">Sie können Benutzer-, Dienst- und Computerkonten lokal und in Azure authentifizieren.</span><span class="sxs-lookup"><span data-stu-id="b31ce-143">You can authenticate user, service, and computer accounts on-premises and in Azure.</span></span>
* <span data-ttu-id="b31ce-144">Sie müssen keine separate AD-Gesamtstruktur verwalten.</span><span class="sxs-lookup"><span data-stu-id="b31ce-144">You don't need to manage a separate AD forest.</span></span> <span data-ttu-id="b31ce-145">Die Domäne in Azure kann der lokalen Gesamtstruktur angehören.</span><span class="sxs-lookup"><span data-stu-id="b31ce-145">The domain in Azure can belong to the on-premises forest.</span></span>
* <span data-ttu-id="b31ce-146">Sie können die von lokalen Gruppenrichtlinienobjekten definierte Gruppenrichtlinie auf die Domäne in Azure anwenden.</span><span class="sxs-lookup"><span data-stu-id="b31ce-146">You can apply group policy defined by on-premises Group Policy Objects to the domain in Azure.</span></span>

<span data-ttu-id="b31ce-147">**Herausforderungen**</span><span class="sxs-lookup"><span data-stu-id="b31ce-147">**Challenges**</span></span>

* <span data-ttu-id="b31ce-148">Sie müssen Ihre eigenen AD DS-Server und -Domänen in der Cloud bereitstellen und verwalten.</span><span class="sxs-lookup"><span data-stu-id="b31ce-148">You must deploy and manage your own AD DS servers and domain in the cloud.</span></span>
* <span data-ttu-id="b31ce-149">Es gibt möglicherweise gewisse Latenzen bei der Synchronisierung zwischen den Domänenservern in der Cloud und den lokal ausgeführten Servern.</span><span class="sxs-lookup"><span data-stu-id="b31ce-149">There may be some synchronization latency between the domain servers in the cloud and the servers running on-premises.</span></span>

<span data-ttu-id="b31ce-150">**[Weitere Informationen][ad-ds]**</span><span class="sxs-lookup"><span data-stu-id="b31ce-150">**[Read more...][ad-ds]**</span></span>

## <a name="ad-ds-in-azure-with-a-separate-forest"></a><span data-ttu-id="b31ce-151">AD DS in Azure mit einer separaten Gesamtstruktur</span><span class="sxs-lookup"><span data-stu-id="b31ce-151">AD DS in Azure with a separate forest</span></span>

<span data-ttu-id="b31ce-152">Stellen Sie AD DS-Server (AD Domain Services) für Azure bereit, erstellen Sie jedoch eine separate Active Directory-[Gesamtstruktur][ad-forest-defn], die von der lokalen Gesamtstruktur getrennt ist.</span><span class="sxs-lookup"><span data-stu-id="b31ce-152">Deploy AD Domain Services (AD DS) servers to Azure, but create a separate Active Directory [forest][ad-forest-defn] that is separate from the on-premises forest.</span></span> <span data-ttu-id="b31ce-153">Domänen in Ihrer lokalen Gesamtstruktur sind für diese Gesamtstruktur vertrauenswürdig.</span><span class="sxs-lookup"><span data-stu-id="b31ce-153">This forest is trusted by domains in your on-premises forest.</span></span>

<span data-ttu-id="b31ce-154">Zu den typischen Einsatzmöglichkeiten dieser Architektur zählen die Verwaltung einer Sicherheitstrennung für Objekte und Identitäten, die in der Cloud gespeichert werden, sowie die Migration einzelner Domänen aus einem lokalen System in die Cloud.</span><span class="sxs-lookup"><span data-stu-id="b31ce-154">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span>

<span data-ttu-id="b31ce-155">**Vorteile**</span><span class="sxs-lookup"><span data-stu-id="b31ce-155">**Benefits**</span></span>

* <span data-ttu-id="b31ce-156">Sie können lokale Identitäten implementieren und von reinen Azure-Identitäten trennen.</span><span class="sxs-lookup"><span data-stu-id="b31ce-156">You can implement on-premises identities and separate Azure-only identities.</span></span>
* <span data-ttu-id="b31ce-157">Sie müssen keine Replikation von der lokalen AD-Gesamtstruktur nach Azure durchführen.</span><span class="sxs-lookup"><span data-stu-id="b31ce-157">You don't need to replicate from the on-premises AD forest to Azure.</span></span>

<span data-ttu-id="b31ce-158">**Herausforderungen**</span><span class="sxs-lookup"><span data-stu-id="b31ce-158">**Challenges**</span></span>

* <span data-ttu-id="b31ce-159">Eine Authentifizierung in Azure für lokale Identitäten erfordert zusätzliche Netzwerkhops zu lokalen AD-Servern.</span><span class="sxs-lookup"><span data-stu-id="b31ce-159">Authentication within Azure for on-premises identities requires extra network hops to the on-premises AD servers.</span></span>
* <span data-ttu-id="b31ce-160">Sie müssen Ihre eigenen AD DS-Server und die Gesamtstruktur in der Cloud bereitstellen und die entsprechenden Vertrauensstellungen zwischen Gesamtstrukturen einrichten.</span><span class="sxs-lookup"><span data-stu-id="b31ce-160">You must deploy your own AD DS servers and forest in the cloud, and establish the appropriate trust relationships between forests.</span></span>

<span data-ttu-id="b31ce-161">**[Weitere Informationen][ad-ds-forest]**</span><span class="sxs-lookup"><span data-stu-id="b31ce-161">**[Read more...][ad-ds-forest]**</span></span>

## <a name="extend-ad-fs-to-azure"></a><span data-ttu-id="b31ce-162">Erweiterung von AD FS auf Azure</span><span class="sxs-lookup"><span data-stu-id="b31ce-162">Extend AD FS to Azure</span></span>

<span data-ttu-id="b31ce-163">Replizieren Sie eine AD FS-Bereitstellung (Active Directory-Verbunddienste) nach Azure, um eine Verbundauthentifizierung und -autorisierung für in Azure ausgeführte Komponenten auszuführen.</span><span class="sxs-lookup"><span data-stu-id="b31ce-163">Replicate an Active Directory Federation Services (AD FS) deployment to Azure, to perform federated authentication and authorization for components running in Azure.</span></span> 

<span data-ttu-id="b31ce-164">Typische Einsatzmöglichkeiten für diese Architektur sind Folgende:</span><span class="sxs-lookup"><span data-stu-id="b31ce-164">Typical uses for this architecture:</span></span>

* <span data-ttu-id="b31ce-165">Authentifizieren und Autorisieren von Benutzern von Partnerorganisationen</span><span class="sxs-lookup"><span data-stu-id="b31ce-165">Authenticate and authorize users from partner organizations.</span></span>
* <span data-ttu-id="b31ce-166">Ermöglichung einer Benutzerauthentifizierung über Webbrowser, die außerhalb der Firewall der Organisation ausgeführt werden</span><span class="sxs-lookup"><span data-stu-id="b31ce-166">Allow users to authenticate from web browsers running outside of the organizational firewall.</span></span>
* <span data-ttu-id="b31ce-167">Ermöglichung eines Verbindungsaufbaus durch Benutzer über autorisierte externe Geräte wie Mobilgeräte</span><span class="sxs-lookup"><span data-stu-id="b31ce-167">Allow users to connect from authorized external devices such as mobile devices.</span></span> 

<span data-ttu-id="b31ce-168">**Vorteile**</span><span class="sxs-lookup"><span data-stu-id="b31ce-168">**Benefits**</span></span>

* <span data-ttu-id="b31ce-169">Sie können Ansprüche unterstützende Anwendungen nutzen.</span><span class="sxs-lookup"><span data-stu-id="b31ce-169">You can leverage claims-aware applications.</span></span>
* <span data-ttu-id="b31ce-170">AD FS bietet die Möglichkeit, zur Authentifizierung externen Partnern zu vertrauen.</span><span class="sxs-lookup"><span data-stu-id="b31ce-170">Provides the ability to trust external partners for authentication.</span></span>
* <span data-ttu-id="b31ce-171">Es besteht Kompatibilität mit einem großen Spektrum an Authentifizierungsprotokollen.</span><span class="sxs-lookup"><span data-stu-id="b31ce-171">Compatibility with large set of authentication protocols.</span></span>

<span data-ttu-id="b31ce-172">**Herausforderungen**</span><span class="sxs-lookup"><span data-stu-id="b31ce-172">**Challenges**</span></span>

* <span data-ttu-id="b31ce-173">Sie müssen Ihre eigenen AD DS-, AD FS- und AD FS-Webanwendungsproxy-Server in Azure bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="b31ce-173">You must deploy your own AD DS, AD FS, and AD FS Web Application Proxy servers in Azure.</span></span>
* <span data-ttu-id="b31ce-174">Die Konfiguration diese Architektur kann komplex sein.</span><span class="sxs-lookup"><span data-stu-id="b31ce-174">This architecture can be complex to configure.</span></span>

<span data-ttu-id="b31ce-175">**[Weitere Informationen][adfs]**</span><span class="sxs-lookup"><span data-stu-id="b31ce-175">**[Read more...][adfs]**</span></span>

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: https://msdn.microsoft.com/library/ms676906.aspx
[adfs]: ./adfs.md

[active-directory-domain-services]: https://technet.microsoft.com/library/dd448614.aspx
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
