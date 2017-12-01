---
title: Sicherheitsmuster
description: "Die Sicherheit ist die Fähigkeit eines Systems, böswillige oder unbeabsichtigte Aktionen, die nicht dem Verwendungszweck des Systems entsprechen, sowie die Offenlegung oder den Verlust von Informationen zu verhindern. Cloudanwendungen werden im Internet außerhalb vertrauenswürdiger lokaler Grenzen verfügbar gemacht, sind oft öffentlich zugänglich und können von nicht vertrauenswürdigen Benutzern verwendet werden. Durch den Entwurf und die Bereitstellung von Anwendungen muss sichergestellt werden, dass sie vor böswilligen Angriffen geschützt sind, der Zugriff auf genehmigte Benutzer beschränkt ist und vertrauliche Daten sicher sind."
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 266b5c4283d82a107783fc7a746f065be9027b51
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="security-patterns"></a><span data-ttu-id="98272-106">Sicherheitsmuster</span><span class="sxs-lookup"><span data-stu-id="98272-106">Security patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="98272-107">Die Sicherheit ist die Fähigkeit eines Systems, böswillige oder unbeabsichtigte Aktionen, die nicht dem Verwendungszweck des Systems entsprechen, sowie die Offenlegung oder den Verlust von Informationen zu verhindern.</span><span class="sxs-lookup"><span data-stu-id="98272-107">Security is the capability of a system to prevent malicious or accidental actions outside of the designed usage, and to prevent disclosure or loss of information.</span></span> <span data-ttu-id="98272-108">Cloudanwendungen werden im Internet außerhalb vertrauenswürdiger lokaler Grenzen verfügbar gemacht, sind oft öffentlich zugänglich und können von nicht vertrauenswürdigen Benutzern verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="98272-108">Cloud applications are exposed on the Internet outside trusted on-premises boundaries, are often open to the public, and may serve untrusted users.</span></span> <span data-ttu-id="98272-109">Durch den Entwurf und die Bereitstellung von Anwendungen muss sichergestellt werden, dass sie vor böswilligen Angriffen geschützt sind, der Zugriff auf genehmigte Benutzer beschränkt ist und vertrauliche Daten sicher sind.</span><span class="sxs-lookup"><span data-stu-id="98272-109">Applications must be designed and deployed in a way that protects them from malicious attacks, restricts access to only approved users, and protects sensitive data.</span></span>

| <span data-ttu-id="98272-110">Muster</span><span class="sxs-lookup"><span data-stu-id="98272-110">Pattern</span></span> | <span data-ttu-id="98272-111">Zusammenfassung</span><span class="sxs-lookup"><span data-stu-id="98272-111">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="98272-112">Verbundidentität</span><span class="sxs-lookup"><span data-stu-id="98272-112">Federated Identity</span></span>](../federated-identity.md) | <span data-ttu-id="98272-113">Authentifizierung an einen externen Identitätsanbieter delegieren</span><span class="sxs-lookup"><span data-stu-id="98272-113">Delegate authentication to an external identity provider.</span></span> |
| [<span data-ttu-id="98272-114">Gatekeeper</span><span class="sxs-lookup"><span data-stu-id="98272-114">Gatekeeper</span></span>](../gatekeeper.md) | <span data-ttu-id="98272-115">Anwendungen und Dienste durch Verwendung einer dedizierten Hostinstanz schützen, die als Broker zwischen Clients und der Anwendung oder dem Dienst fungiert, Anforderungen überprüft und bereinigt sowie Anforderungen und Daten zwischen ihnen weiterleitet</span><span class="sxs-lookup"><span data-stu-id="98272-115">Protect applications and services by using a dedicated host instance that acts as a broker between clients and the application or service, validates and sanitizes requests, and passes requests and data between them.</span></span> |
| [<span data-ttu-id="98272-116">Valet-Schlüssel</span><span class="sxs-lookup"><span data-stu-id="98272-116">Valet Key</span></span>](../valet-key.md) | <span data-ttu-id="98272-117">Ein Token oder einen Schlüssel verwenden, das bzw. der Clients eingeschränkten direkten Zugriff auf eine bestimmte Ressource oder einen bestimmten Dienst bietet</span><span class="sxs-lookup"><span data-stu-id="98272-117">Use a token or key that provides clients with restricted direct access to a specific resource or service.</span></span> |