---
title: Pipes und Filter
description: Unterteilen einer Aufgabe, die komplexe Verarbeitungsvorgänge ausführt, in eine Reihe wiederverwendbarer separater Elemente
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- messaging
ms.openlocfilehash: 2c17504f594843c10fcfe221f0087f1087a73fb8
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="pipes-and-filters-pattern"></a><span data-ttu-id="63576-104">Muster „Pipes und Filter“</span><span class="sxs-lookup"><span data-stu-id="63576-104">Pipes and Filters pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="63576-105">Unterteilen Sie eine Aufgabe, die komplexe Verarbeitungsvorgänge ausführt, in eine Reihe von wiederverwendbaren separaten Elementen.</span><span class="sxs-lookup"><span data-stu-id="63576-105">Decompose a task that performs complex processing into a series of separate elements that can be reused.</span></span> <span data-ttu-id="63576-106">Dies kann die Leistung, Skalierbarkeit und Wiederverwendbarkeit verbessern, indem Taskelemente, die Verarbeitungsvorgänge ausführen, unabhängig voneinander bereitgestellt und skaliert werden können.</span><span class="sxs-lookup"><span data-stu-id="63576-106">This can improve performance, scalability, and reusability by allowing task elements that perform the processing to be deployed and scaled independently.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="63576-107">Kontext und Problem</span><span class="sxs-lookup"><span data-stu-id="63576-107">Context and problem</span></span>

<span data-ttu-id="63576-108">Eine Anwendung muss verschiedene Tasks unterschiedlicher Komplexität mit den von ihr verarbeiteten Informationen ausführen.</span><span class="sxs-lookup"><span data-stu-id="63576-108">An application is required to perform a variety of tasks of varying complexity on the information that it processes.</span></span> <span data-ttu-id="63576-109">Ein einfacher, aber unflexibler Ansatz zur Implementierung einer Anwendung besteht darin, diese Verarbeitung in einem monolithischen Modul durchzuführen.</span><span class="sxs-lookup"><span data-stu-id="63576-109">A straightforward but inflexible approach to implementing an application is to perform this processing as a monolithic module.</span></span> <span data-ttu-id="63576-110">Dieser Ansatz würde jedoch die Möglichkeiten zur Umgestaltung des Codes, zu seiner Optimierung und Wiederverwendung einschränken, wenn an anderer Stelle in der Anwendung Teile des gleichen Verarbeitungsmechanismus erforderlich sind.</span><span class="sxs-lookup"><span data-stu-id="63576-110">However, this approach is likely to reduce the opportunities for refactoring the code, optimizing it, or reusing it if parts of the same processing are required elsewhere within the application.</span></span>

<span data-ttu-id="63576-111">Die Abbildung veranschaulicht die Problematik bei der Datenverarbeitung mit dem monolithischen Ansatz.</span><span class="sxs-lookup"><span data-stu-id="63576-111">The figure illustrates the issues with processing data using the monolithic approach.</span></span> <span data-ttu-id="63576-112">Eine Anwendung empfängt und verarbeitet Daten aus zwei Quellen.</span><span class="sxs-lookup"><span data-stu-id="63576-112">An application receives and processes data from two sources.</span></span> <span data-ttu-id="63576-113">Die Daten aus beiden Quellen werden von einem separaten Modul verarbeitet, das eine Reihe von Tasks zur Transformation dieser Daten ausführt, und das Ergebnis wird an die Geschäftslogik der Anwendung übergeben.</span><span class="sxs-lookup"><span data-stu-id="63576-113">The data from each source is processed by a separate module that performs a series of tasks to transform this data, before passing the result to the business logic of the application.</span></span>

![Abbildung 1: Eine mit monolithischen Modulen implementierte Lösung](./_images/pipes-and-filters-modules.png)

<span data-ttu-id="63576-115">Einige Aufgaben, die die monolithischen Module durchführen, sind funktionell sehr ähnlich, doch die Module wurden separat entworfen.</span><span class="sxs-lookup"><span data-stu-id="63576-115">Some of the tasks that the monolithic modules perform are functionally very similar, but the modules have been designed separately.</span></span> <span data-ttu-id="63576-116">Der Code, der die Tasks implementiert, ist eng in einem Modul gekoppelt. Aspekte in Bezug auf die Wiederverwendung oder Skalierbarkeit wurden bei der Entwicklung kaum oder nicht berücksichtigt.</span><span class="sxs-lookup"><span data-stu-id="63576-116">The code that implements the tasks is closely coupled in a module, and has been developed with little or no thought given to reuse or scalability.</span></span>

<span data-ttu-id="63576-117">Die von den einzelnen Modulen ausgeführten Verarbeitungstasks bzw. die Bereitstellungsanforderungen für die einzelnen Tasks können sich jedoch ändern, wenn die Geschäftsanforderungen aktualisiert werden.</span><span class="sxs-lookup"><span data-stu-id="63576-117">However, the processing tasks performed by each module, or the deployment requirements for each task, could change as business requirements are updated.</span></span> <span data-ttu-id="63576-118">Einige Tasks können rechenintensiv sein und sollten auf leistungsstarker Hardware ausgeführt werden, während andere Tasks weniger kostspielige Ressourcen erfordern.</span><span class="sxs-lookup"><span data-stu-id="63576-118">Some tasks might be compute intensive and could benefit from running on powerful hardware, while others might not require such expensive resources.</span></span> <span data-ttu-id="63576-119">Zudem können künftig zusätzliche Verarbeitungsvorgänge erforderlich sein, oder die Reihenfolge, in der die Tasks bei der Verarbeitung ausgeführt werden, kann sich ändern.</span><span class="sxs-lookup"><span data-stu-id="63576-119">Also, additional processing might be required in the future, or the order in which the tasks performed by the processing could change.</span></span> <span data-ttu-id="63576-120">Es ist eine Lösung erforderlich, in der diese Probleme behandelt werden und die bessere Möglichkeiten für die Wiederverwendung von Codes bietet.</span><span class="sxs-lookup"><span data-stu-id="63576-120">A solution is required that addresses these issues, and increases the possibilities for code reuse.</span></span>

## <a name="solution"></a><span data-ttu-id="63576-121">Lösung</span><span class="sxs-lookup"><span data-stu-id="63576-121">Solution</span></span>

<span data-ttu-id="63576-122">Unterteilen Sie die für jeden Datenstrom erforderliche Verarbeitung in eine Reihe von separaten Komponenten (oder Filtern), die jeweils einen einzelnen Task ausführen.</span><span class="sxs-lookup"><span data-stu-id="63576-122">Break down the processing required for each stream into a set of separate components (or filters), each performing a single task.</span></span> <span data-ttu-id="63576-123">Durch die Standardisierung des Formats der Daten, die jede Komponente empfängt und sendet, können diese Filter zu einer Pipeline zusammengefasst werden.</span><span class="sxs-lookup"><span data-stu-id="63576-123">By standardizing the format of the data that each component receives and sends, these filters can be combined together into a pipeline.</span></span> <span data-ttu-id="63576-124">Hierdurch werden doppelte Codes vermieden und ein einfaches Entfernen, Ersetzen oder Integrieren von zusätzlichen Komponenten ermöglicht, wenn sich die Verarbeitungsanforderungen ändern.</span><span class="sxs-lookup"><span data-stu-id="63576-124">This helps to avoid duplicating code, and makes it easy to remove, replace, or integrate additional components if the processing requirements change.</span></span> <span data-ttu-id="63576-125">Die folgende Abbildung zeigt eine Lösung, bei der Pipes und Filter implementiert wurden.</span><span class="sxs-lookup"><span data-stu-id="63576-125">The next figure shows a solution implemented using pipes and filters.</span></span>

![Abbildung 2: Lösung mit implementierten Pipes und Filtern](./_images/pipes-and-filters-solution.png)


<span data-ttu-id="63576-127">Die Verarbeitungszeit für eine einzelne Anforderung hängt von der Geschwindigkeit des langsamsten Filters in der Pipeline ab.</span><span class="sxs-lookup"><span data-stu-id="63576-127">The time it takes to process a single request depends on the speed of the slowest filter in the pipeline.</span></span> <span data-ttu-id="63576-128">Ein oder mehrere Filter könnten einen Engpass darstellen, insbesondere bei einer großen Anzahl von Anforderungen in einem Datenstrom aus einer bestimmten Datenquelle.</span><span class="sxs-lookup"><span data-stu-id="63576-128">One or more filters could be a bottleneck, especially if a large number of requests appear in a stream from a particular data source.</span></span> <span data-ttu-id="63576-129">Ein wesentlicher Vorteil der Pipelinestruktur besteht darin, dass sie die Ausführung paralleler Instanzen von langsamen Filtern ermöglicht, wodurch das System die Last verteilen und den Durchsatz verbessern kann.</span><span class="sxs-lookup"><span data-stu-id="63576-129">A key advantage of the pipeline structure is that it provides opportunities for running parallel instances of slow filters, enabling the system to spread the load and improve throughput.</span></span>

<span data-ttu-id="63576-130">Die Filter, aus denen sich eine Pipeline zusammensetzt, können auf verschiedenen Computern ausgeführt werden, sodass sie unabhängig voneinander skaliert werden und die Elastizität vieler Cloudumgebungen nutzen können.</span><span class="sxs-lookup"><span data-stu-id="63576-130">The filters that make up a pipeline can run on different machines, enabling them to be scaled independently and take advantage of the elasticity that many cloud environments provide.</span></span> <span data-ttu-id="63576-131">Ein rechenintensiver Filter kann auf leistungsstarker Hardware ausgeführt werden, während andere weniger anspruchsvolle Filter auf preiswerterer Standardhardware gehostet werden können.</span><span class="sxs-lookup"><span data-stu-id="63576-131">A filter that is computationally intensive can run on high performance hardware, while other less demanding filters can be hosted on less expensive commodity hardware.</span></span> <span data-ttu-id="63576-132">Die Filter müssen sich nicht einmal im selben Rechenzentrum oder am selben geografischen Standort befinden, weshalb jedes Element in einer Pipeline in einer Umgebung nahe bei den benötigten Ressourcen ausgeführt werden kann.</span><span class="sxs-lookup"><span data-stu-id="63576-132">The filters don't even have to be in the same data center or geographical location, which allows each element in a pipeline to run in an environment that is close to the resources it requires.</span></span>  <span data-ttu-id="63576-133">Die folgende Abbildung zeigt ein Beispiel, das für die Pipeline für die Daten aus Quelle 1 gilt.</span><span class="sxs-lookup"><span data-stu-id="63576-133">The next figure shows an example applied to the pipeline for the data from Source 1.</span></span>

![Abbildung 3: Beispiel für die Pipeline für die Daten aus Quelle 1](./_images/pipes-and-filters-load-balancing.png)

<span data-ttu-id="63576-135">Wenn die Ein- und Ausgabe eines Filters als Datenstrom strukturiert sind, ist es möglich, die Verarbeitung für jeden Filter parallel durchzuführen.</span><span class="sxs-lookup"><span data-stu-id="63576-135">If the input and output of a filter are structured as a stream, it's possible to perform the processing for each filter in parallel.</span></span> <span data-ttu-id="63576-136">Der erste Filter in der Pipeline kann mit der Durchführung der zugehörigen Tasks beginnen und die entsprechenden Ergebnisse ausgeben, die direkt an den nächsten Filter in der Sequenz übergeben werden, bevor der erste Filter seine Tasks abgeschlossen hat.</span><span class="sxs-lookup"><span data-stu-id="63576-136">The first filter in the pipeline can start its work and output its results, which are passed directly on to the next filter in the sequence before the first filter has completed its work.</span></span>

<span data-ttu-id="63576-137">Ein weiterer Vorteil ist die Resilienz, die dieses Modell bieten kann.</span><span class="sxs-lookup"><span data-stu-id="63576-137">Another benefit is the resiliency that this model can provide.</span></span> <span data-ttu-id="63576-138">Wenn ein Filter Fehler verursacht oder der Computer, auf dem dieser ausgeführt wird, nicht mehr verfügbar ist, kann die Pipeline die von dem Filter ausgeführten Tasks erneut planen und diese Tasks an eine andere Instanz der Komponente weiterleiten.</span><span class="sxs-lookup"><span data-stu-id="63576-138">If a filter fails or the machine it's running on is no longer available, the pipeline can reschedule the work that the filter was performing and direct this work to another instance of the component.</span></span> <span data-ttu-id="63576-139">Der Ausfall eines einzelnen Filters führt nicht zwangsläufig zum Ausfall der gesamten Pipeline.</span><span class="sxs-lookup"><span data-stu-id="63576-139">Failure of a single filter doesn't necessarily result in failure of the entire pipeline.</span></span>

<span data-ttu-id="63576-140">Die Verwendung des Musters „Pipes und Filter“ in Verbindung mit dem [Muster „Kompensierende Transaktion“](compensating-transaction.md) ist eine alternative Vorgehensweise zur Implementierung verteilter Transaktionen.</span><span class="sxs-lookup"><span data-stu-id="63576-140">Using the Pipes and Filters pattern in conjunction with the [Compensating Transaction pattern](compensating-transaction.md) is an alternative approach to implementing distributed transactions.</span></span> <span data-ttu-id="63576-141">Eine verteilte Transaktion kann in einzelne, kompensierbare Tasks zerlegt werden, die mit einem für das Muster „Kompensierende Transaktion“ implementierten Filter implementiert werden können.</span><span class="sxs-lookup"><span data-stu-id="63576-141">A distributed transaction can be broken down into separate, compensable tasks, each of which can be implemented by using a filter that also implements the Compensating Transaction pattern.</span></span> <span data-ttu-id="63576-142">Die Filter in einer Pipeline können als separate gehostete Tasks implementiert werden, die in der Nähe der Daten, die sie verwalten, ausgeführt werden.</span><span class="sxs-lookup"><span data-stu-id="63576-142">The filters in a pipeline can be implemented as separate hosted tasks running close to the data that they maintain.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="63576-143">Probleme und Überlegungen</span><span class="sxs-lookup"><span data-stu-id="63576-143">Issues and considerations</span></span>

<span data-ttu-id="63576-144">Bei der Entscheidung, wie dieses Muster implementiert werden soll, sind die folgenden Punkte zu beachten:</span><span class="sxs-lookup"><span data-stu-id="63576-144">You should consider the following points when deciding how to implement this pattern:</span></span>
- <span data-ttu-id="63576-145">**Komplexität**.</span><span class="sxs-lookup"><span data-stu-id="63576-145">**Complexity**.</span></span> <span data-ttu-id="63576-146">Durch die zusätzliche Flexibilität, die dieses Muster bietet, erhöht sich möglicherweise auch die Komplexität, insbesondere wenn die Filter in einer Pipeline auf verschiedenen Server verteilt sind.</span><span class="sxs-lookup"><span data-stu-id="63576-146">The increased flexibility that this pattern provides can also introduce complexity, especially if the filters in a pipeline are distributed across different servers.</span></span>

- <span data-ttu-id="63576-147">**Zuverlässigkeit**:</span><span class="sxs-lookup"><span data-stu-id="63576-147">**Reliability**.</span></span> <span data-ttu-id="63576-148">Verwenden Sie eine Infrastruktur, die sicherstellt, dass die zwischen Filtern in einer Pipeline weitergeleiteten Daten nicht verloren gehen.</span><span class="sxs-lookup"><span data-stu-id="63576-148">Use an infrastructure that ensures that data flowing between filters in a pipeline won't be lost.</span></span>

- <span data-ttu-id="63576-149">**Idempotenz**:</span><span class="sxs-lookup"><span data-stu-id="63576-149">**Idempotency**.</span></span> <span data-ttu-id="63576-150">Wenn ein Filter in einer Pipeline nach dem Empfang einer Nachricht Fehler verursacht und der Task auf einer anderen Instanz des Filters neu geplant wird, kann es sein, dass ein Teil des Tasks bereits abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="63576-150">If a filter in a pipeline fails after receiving a message and the work is rescheduled to another instance of the filter, part of the work might have already been completed.</span></span> <span data-ttu-id="63576-151">Wenn dieser Task einige Punkte hinsichtlich des globalen Status aktualisiert (z.B. die in einer Datenbank gespeicherten Informationen), kann das Update wiederholt werden.</span><span class="sxs-lookup"><span data-stu-id="63576-151">If this work updates some aspect of the global state (such as information stored in a database), the same update could be repeated.</span></span> <span data-ttu-id="63576-152">Ein ähnliches Problem kann auftreten, wenn ein Filter Fehler verursacht, nachdem er die zugehörigen Ergebnisse für den nächsten Filter in der Pipeline bereitstellt, jedoch bevor er darauf hinweist, dass der Task erfolgreich abgeschlossen wurde.</span><span class="sxs-lookup"><span data-stu-id="63576-152">A similar issue might occur if a filter fails after posting its results to the next filter in the pipeline, but before indicating that it's completed its work successfully.</span></span> <span data-ttu-id="63576-153">In diesen Fällen könnte derselbe Task von einer anderen Instanz des Filters wiederholt werden, sodass die gleichen Ergebnisse zweimal bereitgestellt werden.</span><span class="sxs-lookup"><span data-stu-id="63576-153">In these cases, the same work could be repeated by another instance of the filter, causing the same results to be posted twice.</span></span> <span data-ttu-id="63576-154">Dies könnte dazu führen, dass nachfolgende Filter in der Pipeline dieselben Daten zweimal verarbeiten.</span><span class="sxs-lookup"><span data-stu-id="63576-154">This could result in subsequent filters in the pipeline processing the same data twice.</span></span> <span data-ttu-id="63576-155">Deshalb sollten Filter in einer Pipeline so entworfen sein, dass sie idempotent sind.</span><span class="sxs-lookup"><span data-stu-id="63576-155">Therefore filters in a pipeline should be designed to be idempotent.</span></span> <span data-ttu-id="63576-156">Weitere Informationen finden Sie unter [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) (Idempotenzmuster) im Blog von Jonathan Oliver.</span><span class="sxs-lookup"><span data-stu-id="63576-156">For more information see [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog.</span></span>

- <span data-ttu-id="63576-157">**Wiederholte Nachrichten**:</span><span class="sxs-lookup"><span data-stu-id="63576-157">**Repeated messages**.</span></span> <span data-ttu-id="63576-158">Wenn ein Filter in einer Pipeline Fehler verursacht, nachdem eine Nachricht für die nächste Phase der Pipeline bereitgestellt wurde, wird möglicherweise eine weitere Instanz des Filters ausgeführt und eine Kopie derselben Nachricht für die Pipeline bereitgestellt.</span><span class="sxs-lookup"><span data-stu-id="63576-158">If a filter in a pipeline fails after posting a message to the next stage of the pipeline, another instance of the filter might be run, and it'll post a copy of the same message to the pipeline.</span></span> <span data-ttu-id="63576-159">Dies könnte dazu führen, dass zwei Instanzen derselben Nachricht an den nächsten Filter übergeben werden.</span><span class="sxs-lookup"><span data-stu-id="63576-159">This could cause two instances of the same message to be passed to the next filter.</span></span> <span data-ttu-id="63576-160">Um dies zu vermeiden, sollten die Pipeline doppelte Nachrichten erkennen und entfernen.</span><span class="sxs-lookup"><span data-stu-id="63576-160">To avoid this, the pipeline should detect and eliminate duplicate messages.</span></span>

    >  <span data-ttu-id="63576-161">Wenn Sie die Pipeline mithilfe von Nachrichtenwarteschlangen (z.B. Microsoft Azure Service Bus-Warteschlangen) implementieren, bietet die Message Queuing-Infrastruktur möglicherweise eine Funktion zur automatischen Erkennung und Entfernung doppelter Nachrichten.</span><span class="sxs-lookup"><span data-stu-id="63576-161">If you're implementing the pipeline by using message queues (such as Microsoft Azure Service Bus queues), the message queuing infrastructure might provide automatic duplicate message detection and removal.</span></span>

- <span data-ttu-id="63576-162">**Kontext und Status**:</span><span class="sxs-lookup"><span data-stu-id="63576-162">**Context and state**.</span></span> <span data-ttu-id="63576-163">In einer Pipeline werden die einzelnen Filter im Wesentlichen separat ausgeführt und sollten nicht auf Annahmen zur Art, wie diese aufgerufen wurden, basieren.</span><span class="sxs-lookup"><span data-stu-id="63576-163">In a pipeline, each filter essentially runs in isolation and shouldn't make any assumptions about how it was invoked.</span></span> <span data-ttu-id="63576-164">Das bedeutet, dass jeder Filter für die Durchführung des jeweiligen Tasks mit ausreichendem Kontext versehen werden sollte.</span><span class="sxs-lookup"><span data-stu-id="63576-164">This means that each filter should be provided with sufficient context to perform its work.</span></span> <span data-ttu-id="63576-165">Dieser Kontext kann eine große Menge an Statusinformationen beinhalten.</span><span class="sxs-lookup"><span data-stu-id="63576-165">This context could include a large amount of state information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="63576-166">Verwendung dieses Musters</span><span class="sxs-lookup"><span data-stu-id="63576-166">When to use this pattern</span></span>

<span data-ttu-id="63576-167">Verwenden Sie dieses Muster in folgenden Fällen:</span><span class="sxs-lookup"><span data-stu-id="63576-167">Use this pattern when:</span></span>
- <span data-ttu-id="63576-168">Die von einer Anwendung benötigten Verarbeitungsschritte können mühelos in eine Reihe von unabhängigen Schritten zerlegt werden.</span><span class="sxs-lookup"><span data-stu-id="63576-168">The processing required by an application can easily be broken down into a set of independent steps.</span></span>

- <span data-ttu-id="63576-169">Die von einer Anwendung ausgeführten Verarbeitungsschritte stellen unterschiedliche Anforderungen an die Skalierbarkeit.</span><span class="sxs-lookup"><span data-stu-id="63576-169">The processing steps performed by an application have different scalability requirements.</span></span>

    >  <span data-ttu-id="63576-170">Es ist möglich, Filter zu gruppieren, die im selben Prozess skaliert werden sollen.</span><span class="sxs-lookup"><span data-stu-id="63576-170">It's possible to group filters that should scale together in the same process.</span></span> <span data-ttu-id="63576-171">Weitere Informationen finden Sie unter [Muster „Computeressourcenkonsolidierung“](compute-resource-consolidation.md).</span><span class="sxs-lookup"><span data-stu-id="63576-171">For more information, see the [Compute Resource Consolidation pattern](compute-resource-consolidation.md).</span></span>

- <span data-ttu-id="63576-172">Es wird Flexibilität benötigt, um eine Neuanordnung der von einer Anwendung ausgeführten Verarbeitungsschritte oder das Hinzufügen und Entfernen von Schritten zu ermöglichen.</span><span class="sxs-lookup"><span data-stu-id="63576-172">Flexibility is required to allow reordering of the processing steps performed by an application, or the capability to add and remove steps.</span></span>

- <span data-ttu-id="63576-173">Die Systemleistung kann durch die Verteilung der Verarbeitungsschritte auf verschiedene Server verbessert werden.</span><span class="sxs-lookup"><span data-stu-id="63576-173">The system can benefit from distributing the processing for steps across different servers.</span></span>

- <span data-ttu-id="63576-174">Es ist eine zuverlässige Lösung erforderlich, die die Auswirkungen von Fehlern in einem Schritt während der Datenverarbeitung minimiert.</span><span class="sxs-lookup"><span data-stu-id="63576-174">A reliable solution is required that minimizes the effects of failure in a step while data is being processed.</span></span>

<span data-ttu-id="63576-175">Dieses Muster ist in folgenden Fällen möglicherweise nicht geeignet:</span><span class="sxs-lookup"><span data-stu-id="63576-175">This pattern might not be useful when:</span></span>
- <span data-ttu-id="63576-176">Die von einer Anwendung ausgeführten Verarbeitungsschritte sind nicht unabhängig voneinander oder müssen gemeinsam im Rahmen derselben Transaktion ausgeführt werden.</span><span class="sxs-lookup"><span data-stu-id="63576-176">The processing steps performed by an application aren't independent, or they have to be performed together as part of the same transaction.</span></span>

- <span data-ttu-id="63576-177">Die Menge an Kontext- oder Statusinformationen, die für einen Schritt erforderlich sind, macht diese Vorgehensweise ineffizient.</span><span class="sxs-lookup"><span data-stu-id="63576-177">The amount of context or state information required by a step makes this approach inefficient.</span></span> <span data-ttu-id="63576-178">Statusinformationen können stattdessen möglicherweise in einer Datenbank gespeichert werden. Verwenden Sie diese Strategie jedoch nicht, wenn die zusätzliche Auslastung der Datenbank zu übermäßigen Konflikten führt.</span><span class="sxs-lookup"><span data-stu-id="63576-178">It might be possible to persist state information to a database instead, but don't use this strategy if the additional load on the database causes excessive contention.</span></span>

## <a name="example"></a><span data-ttu-id="63576-179">Beispiel</span><span class="sxs-lookup"><span data-stu-id="63576-179">Example</span></span>

<span data-ttu-id="63576-180">Sie können eine Reihe von Nachrichtenwarteschlangen verwenden, um die für die Implementierung einer Pipeline erforderliche Infrastruktur bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="63576-180">You can use a sequence of message queues to provide the infrastructure required to implement a pipeline.</span></span> <span data-ttu-id="63576-181">Eine anfängliche Nachrichtenwarteschlange empfängt nicht verarbeitete Nachrichten.</span><span class="sxs-lookup"><span data-stu-id="63576-181">An initial message queue receives unprocessed messages.</span></span> <span data-ttu-id="63576-182">Eine als Filtertask implementierte Komponente lauscht auf eine Nachricht in dieser Warteschlange, führt die zugehörigen Tasks aus und stellt die transformierte Nachricht dann für die nächste Warteschlange in der Sequenz bereit.</span><span class="sxs-lookup"><span data-stu-id="63576-182">A component implemented as a filter task listens for a message on this queue, performs its work, and then posts the transformed message to the next queue in the sequence.</span></span> <span data-ttu-id="63576-183">Ein anderer Filtertask kann u.a. auf Nachrichten in dieser Warteschlange lauschen, diese verarbeiten und die Ergebnisse für eine andere Warteschlange bereitstellen, bis die vollständig transformierten Daten in der letzten Nachricht in der Warteschlange angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="63576-183">Another filter task can listen for messages on this queue, process them, post the results to another queue, and so on until the fully transformed data appears in the final message in the queue.</span></span> <span data-ttu-id="63576-184">Die folgende Abbildung zeigt die Implementierung einer Pipeline mithilfe von Nachrichtenwarteschlangen.</span><span class="sxs-lookup"><span data-stu-id="63576-184">The next figure illustrates implementing a pipeline using message queues.</span></span>

![Abbildung 4: Implementierung einer Pipeline mithilfe von Nachrichtenwarteschlangen](./_images/pipes-and-filters-message-queues.png)


<span data-ttu-id="63576-186">Wenn Sie eine Lösung in Azure erstellen, können Sie durch Service Bus-Warteschlangen einen zuverlässigen und skalierbaren Warteschlangenmechanismus bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="63576-186">If you're building a solution on Azure you can use Service Bus queues to provide a reliable and scalable queuing mechanism.</span></span> <span data-ttu-id="63576-187">Die unten in C# gezeigte Klasse `ServiceBusPipeFilter` zeigt, wie Sie einen Filter implementieren können, der Eingabenachrichten von einer Warteschlange empfängt, diese Nachrichten verarbeitet und die Ergebnisse für eine andere Warteschlange bereitstellt.</span><span class="sxs-lookup"><span data-stu-id="63576-187">The `ServiceBusPipeFilter` class shown below in C# demonstrates how you can implement a filter that receives input messages from a queue, processes these messages, and posts the results to another queue.</span></span>

>  <span data-ttu-id="63576-188">Die Klasse `ServiceBusPipeFilter` wird im Projekt „PipesAndFilters.Shared“ definiert, das auf [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters) zur Verfügung gestellt wird.</span><span class="sxs-lookup"><span data-stu-id="63576-188">The `ServiceBusPipeFilter` class is defined in the PipesAndFilters.Shared project available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span></span>

```csharp
public class ServiceBusPipeFilter
{
  ...
  private readonly string inQueuePath;
  private readonly string outQueuePath;
  ...
  private QueueClient inQueue;
  private QueueClient outQueue;
  ...

  public ServiceBusPipeFilter(..., string inQueuePath, string outQueuePath = null)
  {
     ...
     this.inQueuePath = inQueuePath;
     this.outQueuePath = outQueuePath;
  }

  public void Start()
  {
    ...
    // Create the outbound filter queue if it doesn't exist.
    ...
    this.outQueue = QueueClient.CreateFromConnectionString(...);

    ...
    // Create the inbound and outbound queue clients.
    this.inQueue = QueueClient.CreateFromConnectionString(...);
  }

  public void OnPipeFilterMessageAsync(
    Func<BrokeredMessage, Task<BrokeredMessage>> asyncFilterTask, ...)
  {
    ...

    this.inQueue.OnMessageAsync(
      async (msg) =>
    {
      ...
      // Process the filter and send the output to the
      // next queue in the pipeline.
      var outMessage = await asyncFilterTask(msg);

      // Send the message from the filter processor
      // to the next queue in the pipeline.
      if (outQueue != null)
      {
        await outQueue.SendAsync(outMessage);
      }

      // Note: There's a chance that the same message could be sent twice
      // or that a message gets processed by an upstream or downstream
      // filter at the same time.
      // This would happen in a situation where processing of a message was
      // completed, it was sent to the next pipe/queue, and then failed
      // to complete when using the PeekLock method.
      // Idempotent message processing and concurrency should be considered
      // in a real-world implementation.
    },
    options);
  }

  public async Task Close(TimeSpan timespan)
  {
    // Pause the processing threads.
    this.pauseProcessingEvent.Reset();

    // There's no clean approach for waiting for the threads to complete
    // the processing. This example simply stops any new processing, waits
    // for the existing thread to complete, then closes the message pump
    // and finally returns.
    Thread.Sleep(timespan);

    this.inQueue.Close();
    ...
  }

  ...
}
```

<span data-ttu-id="63576-189">Die `Start`-Methode in der Klasse `ServiceBusPipeFilter` stellt eine Verbindung mit einem Paar aus Eingabe- und Ausgabewarteschlangen her, wohingegen die `Close`-Methode die Verbindung mit der Eingabewarteschlange trennt.</span><span class="sxs-lookup"><span data-stu-id="63576-189">The `Start` method in the `ServiceBusPipeFilter` class connects to a pair of input and output queues, and the `Close` method disconnects from the input queue.</span></span> <span data-ttu-id="63576-190">Die `OnPipeFilterMessageAsync`-Methode führt die tatsächliche Verarbeitung der Nachrichten durch, wobei der Parameter `asyncFilterTask` die für diese Methode auszuführenden Verarbeitungsschritte angibt.</span><span class="sxs-lookup"><span data-stu-id="63576-190">The `OnPipeFilterMessageAsync` method performs the actual processing of messages, the `asyncFilterTask` parameter to this method specifies the processing to be performed.</span></span> <span data-ttu-id="63576-191">Die Methode `OnPipeFilterMessageAsync` wartet auf eingehende Nachrichten in der Eingabewarteschlange, führt den vom Parameter `asyncFilterTask` angegebenen Code bei jeder eingehenden Nachricht aus und stellt die Ergebnisse für die Ausgabewarteschlange bereit.</span><span class="sxs-lookup"><span data-stu-id="63576-191">The `OnPipeFilterMessageAsync` method waits for incoming messages on the input queue, runs the code specified by the `asyncFilterTask` parameter over each message as it arrives, and posts the results to the output queue.</span></span> <span data-ttu-id="63576-192">Die Warteschlangen selbst werden vom Konstruktor angegeben.</span><span class="sxs-lookup"><span data-stu-id="63576-192">The queues themselves are specified by the constructor.</span></span>

<span data-ttu-id="63576-193">Die Beispiellösung implementiert Filter in einer Gruppe von Workerrollen.</span><span class="sxs-lookup"><span data-stu-id="63576-193">The sample solution implements filters in a set of worker roles.</span></span> <span data-ttu-id="63576-194">Die einzelnen Workerrollen können unabhängig voneinander skaliert werden, je nachdem, wie komplex die von ihr ausgeführte Verarbeitung von Geschäftsinformationen ist oder welche Ressourcen für die Verarbeitung benötigt werden.</span><span class="sxs-lookup"><span data-stu-id="63576-194">Each worker role can be scaled independently, depending on the complexity of the business processing that it performs or the resources required for processing.</span></span> <span data-ttu-id="63576-195">Darüber hinaus können zur Verbesserung des Durchsatzes mehrere Instanzen von jeder Workerrolle parallel ausgeführt werden.</span><span class="sxs-lookup"><span data-stu-id="63576-195">Additionally, multiple instances of each worker role can be run in parallel to improve throughput.</span></span>

<span data-ttu-id="63576-196">Der folgende Code zeigt eine Azure-Workerrolle namens `PipeFilterARoleEntry`, die im Projekt „PipeFilterA“ in der Beispiellösung definiert wurde.</span><span class="sxs-lookup"><span data-stu-id="63576-196">The following code shows an Azure worker role named `PipeFilterARoleEntry`, defined in the PipeFilterA project in the sample solution.</span></span>

```csharp
public class PipeFilterARoleEntry : RoleEntryPoint
{
  ...
  private ServiceBusPipeFilter pipeFilterA;

  public override bool OnStart()
  {
    ...
    this.pipeFilterA = new ServiceBusPipeFilter(
      ...,
      Constants.QueueAPath,
      Constants.QueueBPath);

    this.pipeFilterA.Start();
    ...
  }

  public override void Run()
  {
    this.pipeFilterA.OnPipeFilterMessageAsync(async (msg) =>
    {
      // Clone the message and update it.
      // Properties set by the broker (Deliver count, enqueue time, ...)
      // aren't cloned and must be copied over if required.
      var newMsg = msg.Clone();

      await Task.Delay(500); // DOING WORK

      Trace.TraceInformation("Filter A processed message:{0} at {1}",
        msg.MessageId, DateTime.UtcNow);

      newMsg.Properties.Add(Constants.FilterAMessageKey, "Complete");

      return newMsg;
    });

    ...
  }

  ...
}
```

<span data-ttu-id="63576-197">Diese Rolle enthält ein `ServiceBusPipeFilter`-Objekt.</span><span class="sxs-lookup"><span data-stu-id="63576-197">This role contains a `ServiceBusPipeFilter` object.</span></span> <span data-ttu-id="63576-198">Die `OnStart`-Methode in der Rolle stellt eine Verbindung mit den Warteschlangen für den Empfang von Eingabenachrichten und die Bereitstellung von Ausgabenachrichten bereit (die Namen der Warteschlangen sind in der Klasse `Constants` definiert).</span><span class="sxs-lookup"><span data-stu-id="63576-198">The `OnStart` method in the role connects to the queues for receiving input messages and posting output messages (the names of the queues are defined in the `Constants` class).</span></span> <span data-ttu-id="63576-199">Die `Run`-Methode ruft die `OnPipeFilterMessagesAsync`-Methode auf, um eine Verarbeitung für jede empfangene Nachricht durchzuführen (in diesem Beispiel wird die Verarbeitung simuliert, indem eine kurze Zeit lang gewartet wird).</span><span class="sxs-lookup"><span data-stu-id="63576-199">The `Run` method invokes the `OnPipeFilterMessagesAsync` method to perform some processing on each message that's received (in this example, the processing is simulated by waiting for a short period of time).</span></span> <span data-ttu-id="63576-200">Nach Abschluss der Verarbeitung wird eine neue Nachricht erstellt, die die Ergebnisse enthält (in diesem Fall wurde der Eingabenachricht eine benutzerdefinierte Eigenschaft hinzugefügt), und diese Nachricht wird für die Ausgabewarteschlange bereitgestellt.</span><span class="sxs-lookup"><span data-stu-id="63576-200">When processing is complete, a new message is constructed containing the results (in this case, the input message has a custom property added), and this message is posted to the output queue.</span></span>

<span data-ttu-id="63576-201">Der Beispielcode enthält eine weitere Workerrolle namens `PipeFilterBRoleEntry` im Projekt „PipeFilterB“.</span><span class="sxs-lookup"><span data-stu-id="63576-201">The sample code contains another worker role named `PipeFilterBRoleEntry` in the PipeFilterB project.</span></span> <span data-ttu-id="63576-202">Diese Rolle weist Ähnlichkeiten mit `PipeFilterARoleEntry` auf, nur dass sie andere Verarbeitungsschritte in der `Run`-Methode durchführt.</span><span class="sxs-lookup"><span data-stu-id="63576-202">This role is similar to `PipeFilterARoleEntry` except that it performs different processing in the `Run` method.</span></span> <span data-ttu-id="63576-203">In der Beispiellösung werden diese beiden Rollen zu einer Pipeline zusammengefasst. Die Ausgabewarteschlange für die Rolle `PipeFilterARoleEntry` ist die Eingabewarteschlange für die Rolle `PipeFilterBRoleEntry`.</span><span class="sxs-lookup"><span data-stu-id="63576-203">In the example solution, these two roles are combined to construct a pipeline, the output queue for the `PipeFilterARoleEntry` role is the input queue for the `PipeFilterBRoleEntry` role.</span></span>

<span data-ttu-id="63576-204">Die Beispiellösung bietet außerdem zwei zusätzliche Rollen mit den Namen `InitialSenderRoleEntry` (im Projekt „InitialSender“) und `FinalReceiverRoleEntry` (im Projekt „FinalReceiver“).</span><span class="sxs-lookup"><span data-stu-id="63576-204">The sample solution also provides two additional roles named `InitialSenderRoleEntry` (in the InitialSender project) and `FinalReceiverRoleEntry` (in the FinalReceiver project).</span></span> <span data-ttu-id="63576-205">Die Rolle `InitialSenderRoleEntry` stellt die erste Nachricht in der Pipeline bereit.</span><span class="sxs-lookup"><span data-stu-id="63576-205">The `InitialSenderRoleEntry` role provides the initial message in the pipeline.</span></span> <span data-ttu-id="63576-206">Die `OnStart`-Methode stellt eine Verbindung mit einer einzelnen Warteschlange her, während die `Run`-Methode eine Methode für diese Warteschlange bereitstellt.</span><span class="sxs-lookup"><span data-stu-id="63576-206">The `OnStart` method connects to a single queue and the `Run` method posts a method to this queue.</span></span> <span data-ttu-id="63576-207">Diese Warteschlange ist die Eingabewarteschlange, die von der Rolle `PipeFilterARoleEntry` verwendet wird. Wenn Sie also eine Nachricht für diese bereitstellen, wird die Nachricht von der Rolle `PipeFilterARoleEntry` empfangen und verarbeitet.</span><span class="sxs-lookup"><span data-stu-id="63576-207">This queue is the input queue used by the `PipeFilterARoleEntry` role, so posting a message to it causes the message to be received and processed by the `PipeFilterARoleEntry` role.</span></span> <span data-ttu-id="63576-208">Die verarbeitete Nachricht übergibt dann die Rolle `PipeFilterBRoleEntry`.</span><span class="sxs-lookup"><span data-stu-id="63576-208">The processed message then passes through the `PipeFilterBRoleEntry` role.</span></span>

<span data-ttu-id="63576-209">Die Eingabewarteschlange für die Rolle `FinalReceiveRoleEntry` ist die Ausgabewarteschlange für die Rolle `PipeFilterBRoleEntry`.</span><span class="sxs-lookup"><span data-stu-id="63576-209">The input queue for the `FinalReceiveRoleEntry` role is the output queue for the `PipeFilterBRoleEntry` role.</span></span> <span data-ttu-id="63576-210">Die unten gezeigte `Run`-Methode in der Rolle `FinalReceiveRoleEntry` empfängt die Nachricht und führt die abschließende Verarbeitung durch.</span><span class="sxs-lookup"><span data-stu-id="63576-210">The `Run` method in the `FinalReceiveRoleEntry` role, shown below, receives the message and performs some final processing.</span></span> <span data-ttu-id="63576-211">Anschließend schreibt sie die Werte der benutzerdefinierten Eigenschaften, die von den Filtern in der Pipeline hinzugefügt wurden, in die Ablaufverfolgungsausgabe.</span><span class="sxs-lookup"><span data-stu-id="63576-211">Then it writes the values of the custom properties added by the filters in the pipeline to the trace output.</span></span>

```csharp
public class FinalReceiverRoleEntry : RoleEntryPoint
{
  ...
  // Final queue/pipe in the pipeline to process data from.
  private ServiceBusPipeFilter queueFinal;

  public override bool OnStart()
  {
    ...
    // Set up the queue.
    this.queueFinal = new ServiceBusPipeFilter(...,Constants.QueueFinalPath);
    this.queueFinal.Start();
    ...
  }

  public override void Run()
  {
    this.queueFinal.OnPipeFilterMessageAsync(
      async (msg) =>
      {
        await Task.Delay(500); // DOING WORK

        // The pipeline message was received.
        Trace.TraceInformation(
          "Pipeline Message Complete - FilterA:{0} FilterB:{1}",
          msg.Properties[Constants.FilterAMessageKey],
          msg.Properties[Constants.FilterBMessageKey]);

        return null;
      });
    ...
  }

  ...
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="63576-212">Zugehörige Muster und Anleitungen</span><span class="sxs-lookup"><span data-stu-id="63576-212">Related patterns and guidance</span></span>

<span data-ttu-id="63576-213">Die folgenden Muster und Anweisungen können für die Implementierung dieses Musters ebenfalls relevant sein:</span><span class="sxs-lookup"><span data-stu-id="63576-213">The following patterns and guidance might also be relevant when implementing this pattern:</span></span>
- <span data-ttu-id="63576-214">Ein Beispiel für dieses Muster steht auf [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span><span class="sxs-lookup"><span data-stu-id="63576-214">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span></span>
- <span data-ttu-id="63576-215">[Muster „Konkurrierende Consumer“](competing-consumers.md):</span><span class="sxs-lookup"><span data-stu-id="63576-215">[Competing Consumers pattern](competing-consumers.md).</span></span> <span data-ttu-id="63576-216">Eine Pipeline kann mehrere Instanzen eines oder mehrerer Filter enthalten.</span><span class="sxs-lookup"><span data-stu-id="63576-216">A pipeline can contain multiple instances of one or more filters.</span></span> <span data-ttu-id="63576-217">Diese Vorgehensweise ist nützlich, um parallele Instanzen von langsamen Filtern auszuführen, sodass das System die Last verteilen und den Durchsatz verbessern kann.</span><span class="sxs-lookup"><span data-stu-id="63576-217">This approach is useful for running parallel instances of slow filters, enabling the system to spread the load and improve throughput.</span></span> <span data-ttu-id="63576-218">Jede Instanz eines Filters konkurriert mit anderen Instanzen um die Eingabe, wobei zwei Instanzen eines Filters nicht die gleichen Daten verarbeiten können sollten.</span><span class="sxs-lookup"><span data-stu-id="63576-218">Each instance of a filter will compete for input with the other instances, two instances of a filter shouldn't be able to process the same data.</span></span> <span data-ttu-id="63576-219">In diesem Artikel wird diese Vorgehensweise erläutert.</span><span class="sxs-lookup"><span data-stu-id="63576-219">Provides an explanation of this approach.</span></span>
- <span data-ttu-id="63576-220">[Muster „Computeressourcenkonsolidierung“](compute-resource-consolidation.md):</span><span class="sxs-lookup"><span data-stu-id="63576-220">[Compute Resource Consolidation pattern](compute-resource-consolidation.md).</span></span> <span data-ttu-id="63576-221">Filter, die im selben Prozess skaliert werden sollen, können gruppiert werden.</span><span class="sxs-lookup"><span data-stu-id="63576-221">It might be possible to group filters that should scale together into the same process.</span></span> <span data-ttu-id="63576-222">Dieser Artikel enthält weitere Informationen über die Vor- und Nachteile dieser Vorgehensweise.</span><span class="sxs-lookup"><span data-stu-id="63576-222">Provides more information about the benefits and tradeoffs of this strategy.</span></span>
- <span data-ttu-id="63576-223">[Muster „Kompensierende Transaktion“](compensating-transaction.md):</span><span class="sxs-lookup"><span data-stu-id="63576-223">[Compensating Transaction pattern](compensating-transaction.md).</span></span> <span data-ttu-id="63576-224">Ein Filter kann als umkehrbarer Vorgang oder mit einem kompensierenden Vorgang, der im Fehlerfall den Zustand einer früheren Version wiederherstellt, implementiert werden.</span><span class="sxs-lookup"><span data-stu-id="63576-224">A filter can be implemented as an operation that can be reversed, or that has a compensating operation that restores the state to a previous version in the event of a failure.</span></span> <span data-ttu-id="63576-225">In diesem Artikel wird erläutert, wie dieses Muster zur Verwaltung oder Bereitstellung von letztlicher Konsistenz implementiert werden kann.</span><span class="sxs-lookup"><span data-stu-id="63576-225">Explains how this can be implemented to maintain or achieve eventual consistency.</span></span>
- <span data-ttu-id="63576-226">[Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) (Idempotenzmuster) im Blog von Jonathan Oliver</span><span class="sxs-lookup"><span data-stu-id="63576-226">[Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog.</span></span>
