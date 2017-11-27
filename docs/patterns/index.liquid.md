---
title: Cloudentwurfsmuster
description: "Cloudentwurfsmuster für Microsoft Azure"
keywords: Azure
ms.openlocfilehash: 264b8296a428f9c1b87314b782efcabc89cf010f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="cloud-design-patterns"></a><span data-ttu-id="2ad8d-104">Cloudentwurfsmuster</span><span class="sxs-lookup"><span data-stu-id="2ad8d-104">Cloud Design Patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="2ad8d-105">Diese Entwurfsmuster können Ihnen dabei helfen, zuverlässige, skalierbare und sichere Anwendungen in der Cloud zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="2ad8d-105">These design patterns are useful for building reliable, scalable, secure applications in the cloud.</span></span>

<span data-ttu-id="2ad8d-106">Für jedes Muster werden das durch das Muster gelöste Problem, Überlegungen zum Anwenden des Musters und ein Beispiel auf der Grundlage von Microsoft Azure beschrieben.</span><span class="sxs-lookup"><span data-stu-id="2ad8d-106">Each pattern describes the problem that the pattern addresses, considerations for applying the pattern, and an example based on Microsoft Azure.</span></span> <span data-ttu-id="2ad8d-107">Die meisten der Muster enthalten Codebeispiele oder -ausschnitte, die die Implementierung des Musters in Azure veranschaulichen.</span><span class="sxs-lookup"><span data-stu-id="2ad8d-107">Most of the patterns include code samples or snippets that show how to implement the pattern on Azure.</span></span> <span data-ttu-id="2ad8d-108">Der Großteil der Muster ist jedoch für alle verteilten Systeme relevant, unabhängig davon, ob sie in Azure oder auf anderen Cloudplattformen gehostet werden.</span><span class="sxs-lookup"><span data-stu-id="2ad8d-108">However, most of the patterns are relevant to any distributed system, whether hosted on Azure or on other cloud platforms.</span></span>

## <a name="problem-areas-in-the-cloud"></a><span data-ttu-id="2ad8d-109">Problembereiche in der Cloud</span><span class="sxs-lookup"><span data-stu-id="2ad8d-109">Problem areas in the cloud</span></span>

<ul id="categories" class="panel">
<span data-ttu-id="2ad8d-110">{%- for category in categories %}</span><span class="sxs-lookup"><span data-stu-id="2ad8d-110">{%- for category in categories %}</span></span>
    <li>
    <span data-ttu-id="2ad8d-111">{% include 'pattern-category-card' %}</span><span class="sxs-lookup"><span data-stu-id="2ad8d-111">{% include 'pattern-category-card' %}</span></span>
    </li>
<span data-ttu-id="2ad8d-112">{%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="2ad8d-112">{%- endfor %}</span></span>
</ul>

## <a name="catalog-of-patterns"></a><span data-ttu-id="2ad8d-113">Musterkatalog</span><span class="sxs-lookup"><span data-stu-id="2ad8d-113">Catalog of patterns</span></span>

| <span data-ttu-id="2ad8d-114">Muster</span><span class="sxs-lookup"><span data-stu-id="2ad8d-114">Pattern</span></span> | <span data-ttu-id="2ad8d-115">Zusammenfassung</span><span class="sxs-lookup"><span data-stu-id="2ad8d-115">Summary</span></span> |
| ------- | ------- |
<span data-ttu-id="2ad8d-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="2ad8d-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span></span>