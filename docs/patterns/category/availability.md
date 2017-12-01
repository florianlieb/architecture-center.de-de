---
title: "Verfügbarkeitsmuster"
description: "Als Verfügbarkeit ist der Zeitanteil definiert, in dem ein System funktioniert und erreichbar ist. Die Verfügbarkeit wird durch Systemfehler, Infrastrukturprobleme, böswillige Angriffe und die Systemauslastung beeinflusst. Sie wird üblicherweise als Prozentsatz der Betriebszeit gemessen. Bei Cloudanwendungen verfügen Benutzer normalerweise über eine Vereinbarung zum Servicelevel (SLA). Dies bedeutet, dass Anwendungen so entworfen und implementiert werden müssen, dass die Verfügbarkeit maximiert wird."
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: f7eb6b0df388b2f1dab83e64ab540cc22f368e19
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="availability-patterns"></a><span data-ttu-id="1983f-107">Verfügbarkeitsmuster</span><span class="sxs-lookup"><span data-stu-id="1983f-107">Availability patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="1983f-108">Als Verfügbarkeit ist der Zeitanteil definiert, in dem ein System funktioniert und erreichbar ist.</span><span class="sxs-lookup"><span data-stu-id="1983f-108">Availability defines the proportion of time that the system is functional and working.</span></span> <span data-ttu-id="1983f-109">Die Verfügbarkeit wird durch Systemfehler, Infrastrukturprobleme, böswillige Angriffe und die Systemauslastung beeinflusst.</span><span class="sxs-lookup"><span data-stu-id="1983f-109">It will be affected by system errors, infrastructure problems, malicious attacks, and system load.</span></span> <span data-ttu-id="1983f-110">Sie wird üblicherweise als Prozentsatz der Betriebszeit gemessen.</span><span class="sxs-lookup"><span data-stu-id="1983f-110">It is usually measured as a percentage of uptime.</span></span> <span data-ttu-id="1983f-111">Bei Cloudanwendungen verfügen Benutzer normalerweise über eine Vereinbarung zum Servicelevel (SLA). Dies bedeutet, dass Anwendungen so entworfen und implementiert werden müssen, dass die Verfügbarkeit maximiert wird.</span><span class="sxs-lookup"><span data-stu-id="1983f-111">Cloud applications typically provide users with a service level agreement (SLA), which means that applications must be designed and implemented in a way that maximizes availability.</span></span>

| <span data-ttu-id="1983f-112">Muster</span><span class="sxs-lookup"><span data-stu-id="1983f-112">Pattern</span></span> | <span data-ttu-id="1983f-113">Zusammenfassung</span><span class="sxs-lookup"><span data-stu-id="1983f-113">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="1983f-114">Überwachung für den Integritätsendpunkt</span><span class="sxs-lookup"><span data-stu-id="1983f-114">Health Endpoint Monitoring</span></span>](../health-endpoint-monitoring.md) | <span data-ttu-id="1983f-115">Funktionale Prüfungen innerhalb einer Anwendung implementieren, auf die externe Tools in regelmäßigen Abständen über verfügbar gemachte Endpunkte zugreifen können</span><span class="sxs-lookup"><span data-stu-id="1983f-115">Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.</span></span> |
| [<span data-ttu-id="1983f-116">Warteschlangenbasierter Lastenausgleich</span><span class="sxs-lookup"><span data-stu-id="1983f-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="1983f-117">Eine Warteschlange verwenden, die als Puffer zwischen einer Aufgabe und einem von ihr aufgerufenen Dienst fungiert, um unregelmäßig auftretende hohe Lasten aufzufangen</span><span class="sxs-lookup"><span data-stu-id="1983f-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="1983f-118">Drosselung</span><span class="sxs-lookup"><span data-stu-id="1983f-118">Throttling</span></span>](../throttling.md) | <span data-ttu-id="1983f-119">Den Verbrauch der von einer Anwendungsinstanz, einem einzelnen Mandanten oder einem gesamten Dienst verwendeten Ressourcen steuern</span><span class="sxs-lookup"><span data-stu-id="1983f-119">Control the consumption of resources used by an instance of an application, an individual tenant, or an entire service.</span></span> |