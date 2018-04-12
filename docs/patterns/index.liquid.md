---
title: Cloudentwurfsmuster
description: Cloudentwurfsmuster für Microsoft Azure
keywords: Azure
ms.openlocfilehash: 4747c896fc6fc5866be782d76c5290d6b49ad451
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="cloud-design-patterns"></a><span data-ttu-id="e259a-104">Cloudentwurfsmuster</span><span class="sxs-lookup"><span data-stu-id="e259a-104">Cloud Design Patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="e259a-105">Diese Entwurfsmuster können Ihnen dabei helfen, zuverlässige, skalierbare und sichere Anwendungen in der Cloud zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="e259a-105">These design patterns are useful for building reliable, scalable, secure applications in the cloud.</span></span>

<span data-ttu-id="e259a-106">Für jedes Muster werden das durch das Muster gelöste Problem, Überlegungen zum Anwenden des Musters und ein Beispiel auf der Grundlage von Microsoft Azure beschrieben.</span><span class="sxs-lookup"><span data-stu-id="e259a-106">Each pattern describes the problem that the pattern addresses, considerations for applying the pattern, and an example based on Microsoft Azure.</span></span> <span data-ttu-id="e259a-107">Die meisten der Muster enthalten Codebeispiele oder -ausschnitte, die die Implementierung des Musters in Azure veranschaulichen.</span><span class="sxs-lookup"><span data-stu-id="e259a-107">Most of the patterns include code samples or snippets that show how to implement the pattern on Azure.</span></span> <span data-ttu-id="e259a-108">Der Großteil der Muster ist jedoch für alle verteilten Systeme relevant, unabhängig davon, ob sie in Azure oder auf anderen Cloudplattformen gehostet werden.</span><span class="sxs-lookup"><span data-stu-id="e259a-108">However, most of the patterns are relevant to any distributed system, whether hosted on Azure or on other cloud platforms.</span></span>

## <a name="problem-areas-in-the-cloud"></a><span data-ttu-id="e259a-109">Problembereiche in der Cloud</span><span class="sxs-lookup"><span data-stu-id="e259a-109">Problem areas in the cloud</span></span>

<ul id="categories" class="panel">
<span data-ttu-id="e259a-110">{%- for category in categories %}</span><span class="sxs-lookup"><span data-stu-id="e259a-110">{%- for category in categories %}</span></span>
    <li>
    <span data-ttu-id="e259a-111">{% include 'pattern-category-card' %}</span><span class="sxs-lookup"><span data-stu-id="e259a-111">{% include 'pattern-category-card' %}</span></span>
    </li>
<span data-ttu-id="e259a-112">{%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="e259a-112">{%- endfor %}</span></span>
</ul>

## <a name="catalog-of-patterns"></a><span data-ttu-id="e259a-113">Musterkatalog</span><span class="sxs-lookup"><span data-stu-id="e259a-113">Catalog of patterns</span></span>

| <span data-ttu-id="e259a-114">Muster</span><span class="sxs-lookup"><span data-stu-id="e259a-114">Pattern</span></span> | <span data-ttu-id="e259a-115">Zusammenfassung</span><span class="sxs-lookup"><span data-stu-id="e259a-115">Summary</span></span> |
|---------|---------|
|         |         |

<span data-ttu-id="e259a-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="e259a-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span></span>
