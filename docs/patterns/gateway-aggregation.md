---
title: Muster „Gatewayaggregation“
description: Aggregieren Sie mithilfe eines Gateways mehrere einzelne Anforderungen in einer einzelnen Anforderung.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: f59c8b8b02c6db28024d13621b782997e63a4e9e
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="gateway-aggregation-pattern"></a><span data-ttu-id="c6aa8-103">Muster „Gatewayaggregation“</span><span class="sxs-lookup"><span data-stu-id="c6aa8-103">Gateway Aggregation pattern</span></span>

<span data-ttu-id="c6aa8-104">Aggregieren Sie mithilfe eines Gateways mehrere einzelne Anforderungen in einer einzelnen Anforderung.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-104">Use a gateway to aggregate multiple individual requests into a single request.</span></span> <span data-ttu-id="c6aa8-105">Dieses Muster ist hilfreich, wenn ein Client zur Durchführung eines Vorgangs mehrere Aufrufe an verschiedene Back-End-Systeme tätigen muss.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-105">This pattern is useful when a client must make multiple calls to different backend systems to perform an operation.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="c6aa8-106">Kontext und Problem</span><span class="sxs-lookup"><span data-stu-id="c6aa8-106">Context and problem</span></span>

<span data-ttu-id="c6aa8-107">Um eine einzelne Aufgabe auszuführen, muss ein Client möglicherweise mehrere Aufrufe von verschiedenen Back-End-Diensten durchführen.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-107">To perform a single task, a client may have to make multiple calls to various backend services.</span></span> <span data-ttu-id="c6aa8-108">Eine Anwendung, die zur Ausführung einer Aufgabe auf viele Dienste angewiesen ist, muss für jede Anforderung Ressourcen aufwenden.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-108">An application that relies on many services to perform a task must expend resources on each request.</span></span> <span data-ttu-id="c6aa8-109">Wenn der Anwendung neue Features oder Dienste hinzugefügt werden, sind zusätzliche Anforderungen erforderlich, durch die sich der Ressourcenbedarf und die Anzahl der Netzwerkaufrufe weiter erhöhen.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-109">When any new feature or service is added to the application, additional requests are needed, further increasing resource requirements and network calls.</span></span> <span data-ttu-id="c6aa8-110">Eine derart umfangreiche Interaktion (Chattiness) zwischen einer Client- und einer Back-End-Anwendung können Leistung und Skalierung der Anwendung beeinträchtigen.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-110">This chattiness between a client and a backend can adversely impact the performance and scale of the application.</span></span>  <span data-ttu-id="c6aa8-111">Durch Microservicearchitekturen ist dieses Problem immer verbreiteter geworden, da Anwendungen, die um viele kleinere Dienste herum erstellt werden, in der Regel eine größere Anzahl von dienstübergreifenden Aufrufen aufweisen.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-111">Microservice architectures have made this problem more common, as applications built around many smaller services naturally have a higher amount of cross-service calls.</span></span> 

<span data-ttu-id="c6aa8-112">Im folgenden Diagramm sendet der Client Anforderungen an jeden Dienst (1, 2, 3).</span><span class="sxs-lookup"><span data-stu-id="c6aa8-112">In the following diagram, the client sends requests to each service (1,2,3).</span></span> <span data-ttu-id="c6aa8-113">Jeder Dienst verarbeitet die Anforderung und sendet die Antwort an die Anwendung zurück (4, 5, 6).</span><span class="sxs-lookup"><span data-stu-id="c6aa8-113">Each service processes the request and sends the response back to the application (4,5,6).</span></span> <span data-ttu-id="c6aa8-114">Bei einem Mobilfunknetz mit üblicherweise hohen Latenzen ist ein derartiger Einsatz einzelner Anforderungen ineffizient und kann Konnektivitätsabbrüche oder unvollständige Anforderungen verursachen.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-114">Over a cellular network with typically high latency, using individual requests in this manner is inefficient and could result in broken connectivity or incomplete requests.</span></span> <span data-ttu-id="c6aa8-115">Die einzelnen Anforderungen können zwar parallel ausgeführt werden, allerdings muss die Anwendung für jede Anforderung Daten senden und verarbeiten bzw. auf diese warten. Da all diese Vorgänge über separate Verbindungen erfolgen, besteht hierbei eine höhere Wahrscheinlichkeit eines Ausfalls.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-115">While each request may be done in parallel, the application must send, wait, and process data for each request, all on separate connections, increasing the chance of failure.</span></span>

![](./_images/gateway-aggregation-problem.png) 

## <a name="solution"></a><span data-ttu-id="c6aa8-116">Lösung</span><span class="sxs-lookup"><span data-stu-id="c6aa8-116">Solution</span></span>

<span data-ttu-id="c6aa8-117">Verwenden Sie ein Gateway, um die umfangreiche Interaktion zwischen dem Client und den Diensten (Chattiness) zu reduzieren.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-117">Use a gateway to reduce chattiness between the client and the services.</span></span> <span data-ttu-id="c6aa8-118">Das Gateway empfängt Clientanforderungen, versendet Anforderungen an die verschiedenen Back-End-Systeme, aggregiert die Ergebnisse und sendet sie wieder an den anfordernden Client zurück.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-118">The gateway receives client requests, dispatches requests to the various backend systems, and then aggregates the results and sends them back to the requesting client.</span></span>

<span data-ttu-id="c6aa8-119">Dieses Muster kann die Anzahl der Anforderungen, die die Anwendung an Back-End-Dienste sendet, reduzieren und die Anwendungsleistung für Netzwerke mit hohen Latenzen verbessern.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-119">This pattern can reduce the number of requests that the application makes to backend services, and improve application performance over high-latency networks.</span></span>

<span data-ttu-id="c6aa8-120">Im folgenden Diagramm sendet die Anwendung eine Anforderung an das Gateway (1).</span><span class="sxs-lookup"><span data-stu-id="c6aa8-120">In the following diagram, the application sends a request to the gateway (1).</span></span> <span data-ttu-id="c6aa8-121">Die Anforderung enthält ein Paket mit zusätzlichen Anforderungen.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-121">The request contains a package of additional requests.</span></span> <span data-ttu-id="c6aa8-122">Das Gateway zerlegt diese und verarbeitet jede Anforderung, indem es diese an den entsprechenden Dienst (2) sendet.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-122">The gateway decomposes these and processes each request by sending it to the relevant service (2).</span></span> <span data-ttu-id="c6aa8-123">Jeder Dienst gibt eine Antwort an das Gateway zurück (3).</span><span class="sxs-lookup"><span data-stu-id="c6aa8-123">Each service returns a response to the gateway (3).</span></span> <span data-ttu-id="c6aa8-124">Das Gateway fasst die Antworten der einzelnen Dienste zusammen und sendet die Antwort an die Anwendung (4).</span><span class="sxs-lookup"><span data-stu-id="c6aa8-124">The gateway combines the responses from each service and sends the response to the application (4).</span></span> <span data-ttu-id="c6aa8-125">Die Anwendung erstellt eine einzige Anforderung und erhält nur eine einzige Antwort vom Gateway.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-125">The application makes a single request and receives only a single response from the gateway.</span></span>

![](./_images/gateway-aggregation.png)

## <a name="issues-and-considerations"></a><span data-ttu-id="c6aa8-126">Probleme und Überlegungen</span><span class="sxs-lookup"><span data-stu-id="c6aa8-126">Issues and considerations</span></span>

- <span data-ttu-id="c6aa8-127">Das Gateway sollte keine Kopplung zwischen Diensten für die Back-End-Dienste vornehmen.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-127">The gateway should not introduce service coupling across the backend services.</span></span>
- <span data-ttu-id="c6aa8-128">Das Gateway sollte sich in der Nähe der Back-End-Dienste befinden, um die Latenz so gering wie möglich zu halten.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-128">The gateway should be located near the backend services to reduce latency as much as possible.</span></span>
- <span data-ttu-id="c6aa8-129">Der Gatewaydienst kann ein Single Point of Failure darstellen.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-129">The gateway service may introduce a single point of failure.</span></span> <span data-ttu-id="c6aa8-130">Stellen Sie sicher, dass das Gateway die Verfügbarkeitsanforderungen Ihrer Anwendung erfüllt.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-130">Ensure the gateway is properly designed to meet your application's availability requirements.</span></span>
- <span data-ttu-id="c6aa8-131">Das Gateway kann einen Engpass darstellen.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-131">The gateway may introduce a bottleneck.</span></span> <span data-ttu-id="c6aa8-132">Stellen Sie sicher, dass das Gateway über die entsprechende Leistung zur Verarbeitung der Last verfügt und skaliert werden kann, um dem erwarteten Anstieg nachzukommen.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-132">Ensure the gateway has adequate performance to handle load and can be scaled to meet your anticipated growth.</span></span>
- <span data-ttu-id="c6aa8-133">Führen Sie einen Auslastungstest für das Gateway durch, um sicherzustellen, dass keine kaskadierenden Ausfälle bei den Diensten auftreten.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-133">Perform load testing against the gateway to ensure you don't introduce cascading failures for services.</span></span>
- <span data-ttu-id="c6aa8-134">Implementieren Sie einen stabilen Entwurf unter Verwendung von Mustern wie [Bulkheads][bulkhead], [Sicherung][circuit-breaker], [Wiederholung][retry] und Timeouts.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-134">Implement a resilient design, using techniques such as [bulkheads][bulkhead], [circuit breaking][circuit-breaker], [retry][retry], and timeouts.</span></span>
- <span data-ttu-id="c6aa8-135">Wenn Dienstaufrufe zu lange dauern, kann es helfen, ein Timeout durchzuführen und einen Teil des Datasets zurückzugeben.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-135">If one or more service calls takes too long, it may be acceptable to timeout and return a partial set of data.</span></span> <span data-ttu-id="c6aa8-136">Berücksichtigen Sie, wie sich Ihre Anwendung in diesem Szenario verhält.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-136">Consider how your application will handle this scenario.</span></span>
- <span data-ttu-id="c6aa8-137">Verwenden Sie asynchrone E/A-Vorgänge, um sicherzustellen, dass eine Verzögerung am Back-End keine Leistungsprobleme in der Anwendung verursacht.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-137">Use asynchronous I/O to ensure that a delay at the backend doesn't cause performance issues in the application.</span></span>
- <span data-ttu-id="c6aa8-138">Implementieren Sie mithilfe von Korrelations-IDs verteilte Ablaufverfolgungen, um jeden einzelnen Aufruf nachzuverfolgen.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-138">Implement distributed tracing using correlation IDs to track each individual call.</span></span>
- <span data-ttu-id="c6aa8-139">Überwachen Sie Anforderungsmetriken und Antwortgrößen.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-139">Monitor request metrics and response sizes.</span></span>
- <span data-ttu-id="c6aa8-140">Betrachten Sie die Rückgabe von zwischengespeicherten Daten als Failoverstrategie, um Ausfälle zu behandeln.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-140">Consider returning cached data as a failover strategy to handle failures.</span></span>
- <span data-ttu-id="c6aa8-141">Statt eine Aggregation in das Gateway zu integrieren, sollten Sie eventuell einen Aggregationsdienst hinter dem Gateway platzieren.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-141">Instead of building aggregation into the gateway, consider placing an aggregation service behind the gateway.</span></span> <span data-ttu-id="c6aa8-142">Für die Aggregation von Anforderungen gelten wahrscheinlich andere Ressourcenanforderungen als für andere Dienste im Gateway, was sich auf die Routing- und Abladungsfunktionalität des Gateways auswirken kann.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-142">Request aggregation will likely have different resource requirements than other services in the gateway and may impact the gateway's routing and offloading functionality.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="c6aa8-143">Verwendung dieses Musters</span><span class="sxs-lookup"><span data-stu-id="c6aa8-143">When to use this pattern</span></span>

<span data-ttu-id="c6aa8-144">Verwenden Sie dieses Muster in folgenden Fällen:</span><span class="sxs-lookup"><span data-stu-id="c6aa8-144">Use this pattern when:</span></span>

- <span data-ttu-id="c6aa8-145">Ein Client muss mit mehreren Back-End-Diensten kommunizieren, um einen Vorgang durchführen zu können.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-145">A client needs to communicate with multiple backend services to perform an operation.</span></span>
- <span data-ttu-id="c6aa8-146">Der Client kann Netzwerke mit deutlich hoher Latenz verwenden (z.B. Mobilfunknetze).</span><span class="sxs-lookup"><span data-stu-id="c6aa8-146">The client may use networks with significant latency, such as cellular networks.</span></span>

<span data-ttu-id="c6aa8-147">Dieses Muster ist in folgenden Fällen möglicherweise nicht geeignet:</span><span class="sxs-lookup"><span data-stu-id="c6aa8-147">This pattern may not be suitable when:</span></span>

- <span data-ttu-id="c6aa8-148">Sie möchten die Anzahl der Aufrufe zwischen einem Client und einem einzelnen Dienst für mehrere Vorgänge reduzieren.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-148">You want to reduce the number of calls between a client and a single service across multiple operations.</span></span> <span data-ttu-id="c6aa8-149">In diesem Szenario kann es besser sein, dem Dienst einen Batchvorgang hinzuzufügen.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-149">In that scenario, it may be better to add a batch operation to the service.</span></span>
- <span data-ttu-id="c6aa8-150">Der Client oder die Anwendung befindet sich in der Nähe der Back-End-Dienste, und Latenzen spielen keine wesentliche Rolle.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-150">The client or application is located near the backend services and latency is not a significant factor.</span></span>

## <a name="example"></a><span data-ttu-id="c6aa8-151">Beispiel</span><span class="sxs-lookup"><span data-stu-id="c6aa8-151">Example</span></span>

<span data-ttu-id="c6aa8-152">Das folgende Beispiel zeigt, wie mit Lua ein einfacher NGINX-Dienst für die Gatewayaggregation erstellt wird.</span><span class="sxs-lookup"><span data-stu-id="c6aa8-152">The following example illustrates how to create a simple a gateway aggregation NGINX service using Lua.</span></span>

```lua
worker_processes  4;

events {
  worker_connections 1024;
}

http {
  server {
    listen 80;

    location = /batch {
      content_by_lua '
        ngx.req.read_body()

        -- read json body content
        local cjson = require "cjson"
        local batch = cjson.decode(ngx.req.get_body_data())["batch"]

        -- create capture_multi table
        local requests = {}
        for i, item in ipairs(batch) do
          table.insert(requests, {item.relative_url, { method = ngx.HTTP_GET}})
        end

        -- execute batch requests in parallel
        local results = {}
        local resps = { ngx.location.capture_multi(requests) }
        for i, res in ipairs(resps) do
          table.insert(results, {status = res.status, body = cjson.decode(res.body), header = res.header})
        end

        ngx.say(cjson.encode({results = results}))
      ';
    }

    location = /service1 {
      default_type application/json;
      echo '{"attr1":"val1"}';
    }

    location = /service2 {
      default_type application/json;
      echo '{"attr2":"val2"}';
    }
  }
}
```

## <a name="related-guidance"></a><span data-ttu-id="c6aa8-153">Verwandte Leitfäden</span><span class="sxs-lookup"><span data-stu-id="c6aa8-153">Related guidance</span></span>

- [<span data-ttu-id="c6aa8-154">Muster „Back-Ends für Front-Ends“</span><span class="sxs-lookup"><span data-stu-id="c6aa8-154">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="c6aa8-155">Muster „Gatewayabladung“</span><span class="sxs-lookup"><span data-stu-id="c6aa8-155">Gateway Offloading pattern</span></span>](./gateway-offloading.md)
- [<span data-ttu-id="c6aa8-156">Muster „Gatewayrouting“</span><span class="sxs-lookup"><span data-stu-id="c6aa8-156">Gateway Routing pattern</span></span>](./gateway-routing.md)

[bulkhead]: ./bulkhead.md
[circuit-breaker]: ./circuit-breaker.md
[retry]: ./retry.md