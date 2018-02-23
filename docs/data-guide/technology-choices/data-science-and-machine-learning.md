---
title: "Auswählen einer Machine Learning-Technologie"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 995349c795066ec3067b20ad2615e40b0fb152db
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-machine-learning-technology-in-azure"></a><span data-ttu-id="568fb-102">Auswählen einer Machine Learning-Technologie in Azure</span><span class="sxs-lookup"><span data-stu-id="568fb-102">Choosing a machine learning technology in Azure</span></span>

<span data-ttu-id="568fb-103">Bei Data Science und Machine Learning handelt es sich um eine Workload, die in der Regel von Datenspezialisten verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="568fb-103">Data science and machine learning is a workload that is usually undertaken by data scientists.</span></span> <span data-ttu-id="568fb-104">Sie erfordert besondere Tools, die häufig speziell für die Art von interaktiver Datenuntersuchung und die Modellierungsaufgaben konzipiert sind, die ein Datenspezialist benötigt.</span><span class="sxs-lookup"><span data-stu-id="568fb-104">It requires specialist tools, many of which are designed specifically for the type of interactive data exploration and modeling tasks that a data scientist must perform.</span></span>

<span data-ttu-id="568fb-105">Machine Learning-Lösungen sind iterativ aufgebaut und umfassen zwei Phasen:</span><span class="sxs-lookup"><span data-stu-id="568fb-105">Machine learning solutions are built iteratively, and have two distinct phases:</span></span>
* <span data-ttu-id="568fb-106">Datenvorbereitung und -modellierung</span><span class="sxs-lookup"><span data-stu-id="568fb-106">Data preparation and modeling.</span></span>
* <span data-ttu-id="568fb-107">Bereitstellung und Nutzung von Prognosediensten</span><span class="sxs-lookup"><span data-stu-id="568fb-107">Deployment and consumption of predictive services.</span></span>

## <a name="tools-and-services-for-data-preparation-and-modeling"></a><span data-ttu-id="568fb-108">Tools und Dienste zur Datenvorbereitung und -modellierung</span><span class="sxs-lookup"><span data-stu-id="568fb-108">Tools and services for data preparation and modeling</span></span>
<span data-ttu-id="568fb-109">Für die Arbeit mit Daten bevorzugen Datenspezialisten in der Regel benutzerdefinierten, in Python oder R geschriebenen Code. Dieser wird im Allgemeinen interaktiv ausgeführt und von den Datenspezialisten zum Abfragen und Untersuchen der Daten verwendet. Dabei werden Visualisierungen und Statistiken generiert, um Beziehungen zu ermitteln.</span><span class="sxs-lookup"><span data-stu-id="568fb-109">Data scientists typically prefer to work with data using custom code written in Python or R. This code is generally run interactively, with the data scientists using it to query and explore the data, generating visualizations and statistics to help determine the relationships with it.</span></span> <span data-ttu-id="568fb-110">Den Datenspezialisten stehen zahlreiche interaktive Umgebungen für R und Python zur Verfügung.</span><span class="sxs-lookup"><span data-stu-id="568fb-110">There are many interactive environments for R and Python that data scientists can use.</span></span> <span data-ttu-id="568fb-111">Besonders beliebt ist **Jupyter Notebooks**: Diese Umgebung bietet eine browserbasierte Shell, mit der Datenspezialisten *Notebook-Dateien* erstellen können, die R- oder Python-Code sowie Markdowntext enthalten.</span><span class="sxs-lookup"><span data-stu-id="568fb-111">A particular favorite is **Jupyter Notebooks** that provides a browser-based shell that enables data scientists to create *notebook* files that contain R or Python code and markdown text.</span></span> <span data-ttu-id="568fb-112">Dies ermöglicht eine effektive Zusammenarbeit, da Code und Ergebnisse in einem einzelnen Dokument weitergegeben und dokumentiert werden können.</span><span class="sxs-lookup"><span data-stu-id="568fb-112">This is an effective way to collaborate by sharing and documenting code and results in a single document.</span></span>

<span data-ttu-id="568fb-113">Folgende Tools werden ebenfalls gerne verwendet:</span><span class="sxs-lookup"><span data-stu-id="568fb-113">Other commonly used tools include:</span></span>
* <span data-ttu-id="568fb-114">**Spyder:** Die interaktive Entwicklungsumgebung (Interactive Development Environment, IDE), die mit der Python-Distribution Anaconda bereitgestellt wird.</span><span class="sxs-lookup"><span data-stu-id="568fb-114">**Spyder**: The interactive development environment (IDE) for Python provided with the Anaconda Python distribution.</span></span>
* <span data-ttu-id="568fb-115">**R Studio:** Eine IDE für die Programmiersprache R.</span><span class="sxs-lookup"><span data-stu-id="568fb-115">**R Studio**: An IDE for the R programming language.</span></span>
* <span data-ttu-id="568fb-116">**Visual Studio Code:** Eine einfache, plattformübergreifende Programmierumgebung, die sowohl Python als auch gängige Frameworks für Machine Learning und KI-Entwicklung unterstützt.</span><span class="sxs-lookup"><span data-stu-id="568fb-116">**Visual Studio Code**: A lightweight, cross-platform coding environment that supports Python as well as commonly used frameworks for machine learning and AI development.</span></span>

<span data-ttu-id="568fb-117">Neben diesen Tools können Datenspezialisten auch Azure-Dienste nutzen, um die Code- und Modellverwaltung zu vereinfachen.</span><span class="sxs-lookup"><span data-stu-id="568fb-117">In addition to these tools, data scientists can leverage Azure services to simplify code and model management.</span></span>

### <a name="azure-notebooks"></a><span data-ttu-id="568fb-118">Azure Notebooks</span><span class="sxs-lookup"><span data-stu-id="568fb-118">Azure Notebooks</span></span>
<span data-ttu-id="568fb-119">Azure Notebooks ist ein Jupyter Notebooks-Onlinedienst, mit dem Datenspezialisten Jupyter Notebooks in cloudbasierten Bibliotheken erstellen, ausführen und weitergeben können.</span><span class="sxs-lookup"><span data-stu-id="568fb-119">Azure Notebooks is an online Jupyter Notebooks service that enables data scientists to create, run, and share Jupyter Notebooks in cloud-based libraries.</span></span>

<span data-ttu-id="568fb-120">Hauptvorteile:</span><span class="sxs-lookup"><span data-stu-id="568fb-120">Key benefits:</span></span>

* <span data-ttu-id="568fb-121">Kostenloser Dienst: Sie benötigen kein Azure-Abonnement.</span><span class="sxs-lookup"><span data-stu-id="568fb-121">Free service&mdash;no Azure subscription required.</span></span>
* <span data-ttu-id="568fb-122">Keine lokale Installation von Jupyter und den unterstützenden R- oder Python-Distributionen erforderlich: Ein Browser genügt.</span><span class="sxs-lookup"><span data-stu-id="568fb-122">No need to install Jupyter and the supporting R or Python distributions locally&mdash;just use a browser.</span></span>
* <span data-ttu-id="568fb-123">Verwalten Sie Ihre eigenen Onlinebibliotheken, und greifen Sie von einem beliebigen Gerät aus darauf zu.</span><span class="sxs-lookup"><span data-stu-id="568fb-123">Manage your own online libraries and access them from any device.</span></span>
* <span data-ttu-id="568fb-124">Teilen Sie Ihre Notebooks mit Projektmitarbeitern.</span><span class="sxs-lookup"><span data-stu-id="568fb-124">Share your notebooks with collaborators.</span></span>

<span data-ttu-id="568fb-125">Überlegungen:</span><span class="sxs-lookup"><span data-stu-id="568fb-125">Considerations:</span></span>

* <span data-ttu-id="568fb-126">Die Notebooks stehen offline nicht zur Verfügung.</span><span class="sxs-lookup"><span data-stu-id="568fb-126">You will be unable to access your notebooks when offline.</span></span>
* <span data-ttu-id="568fb-127">Die eingeschränkten Verarbeitungsfunktionen des kostenlosen Notebookdiensts reichen für umfangreiche oder komplexe Modelle unter Umständen nicht aus.</span><span class="sxs-lookup"><span data-stu-id="568fb-127">Limited processing capabilities of the free notebook service may not be enough to train large or complex models.</span></span>

### <a name="data-science-virtual-machine"></a><span data-ttu-id="568fb-128">Virtueller Data Science-Computer</span><span class="sxs-lookup"><span data-stu-id="568fb-128">Data science virtual machine</span></span>
<span data-ttu-id="568fb-129">Der virtuelle Data Science-Computer ist ein Image eines virtuellen Azure-Computers mit den Tools und Frameworks, die üblicherweise von Datenspezialisten verwendet werden – einschließlich R, Python, Jupyter Notebooks, Visual Studio Code und Bibliotheken für die Machine Learning-Modellierung (etwa das Microsoft Cognitive Toolkit).</span><span class="sxs-lookup"><span data-stu-id="568fb-129">The data science virtual machine is an Azure virtual machine image that includes the tools and frameworks commonly used by data scientists, including R, Python, Jupyter Notebooks, Visual Studio Code, and libraries for machine learning modeling such as the Microsoft Cognitive Toolkit.</span></span> <span data-ttu-id="568fb-130">Diese Tools können komplex und zeitaufwendig zu installieren sein. Darüber hinaus können sie zahlreiche Abhängigkeiten enthalten, die immer wieder zu Problemen mit der Versionsverwaltung führen.</span><span class="sxs-lookup"><span data-stu-id="568fb-130">These tools can be complex and time consuming to install, and contain many interdependencies that often lead to version management issues.</span></span> <span data-ttu-id="568fb-131">Mit einem vorinstallierten Image verbringen Datenspezialisten weniger Zeit mit der Behandlung von Umgebungsproblemen und können sich stattdessen auf die nötigen Datenuntersuchungen und Modellierungsaufgaben konzentrieren.</span><span class="sxs-lookup"><span data-stu-id="568fb-131">Having a preinstalled image can reduce the time data scientists spend troubleshooting environment issues, allowing them to focus on the data exploration and modeling tasks they need to perform.</span></span>

<span data-ttu-id="568fb-132">Hauptvorteile:</span><span class="sxs-lookup"><span data-stu-id="568fb-132">Key benefits:</span></span>
* <span data-ttu-id="568fb-133">Geringerer Installations-, Verwaltungs- und Problembehandlungsaufwand für Data Science-Tools und -Frameworks.</span><span class="sxs-lookup"><span data-stu-id="568fb-133">Reduced time to install, manage, and troubleshoot data science tools and frameworks.</span></span>
* <span data-ttu-id="568fb-134">Verfügbarkeit der neuesten Versionen aller gängigen Tools und Frameworks</span><span class="sxs-lookup"><span data-stu-id="568fb-134">The latest versions of all commonly used tools and frameworks are included.</span></span>
* <span data-ttu-id="568fb-135">VM-Optionen mit hochgradig skalierbaren Images und GPU-Funktionen für intensive Datenmodellierung</span><span class="sxs-lookup"><span data-stu-id="568fb-135">Virtual machine options include highly scalable images with GPU capabilities for intensive data modeling.</span></span>

<span data-ttu-id="568fb-136">Überlegungen:</span><span class="sxs-lookup"><span data-stu-id="568fb-136">Considerations:</span></span>
* <span data-ttu-id="568fb-137">Der virtuelle Computer steht offline nicht zur Verfügung.</span><span class="sxs-lookup"><span data-stu-id="568fb-137">The virtual machine cannot be accessed when offline.</span></span>
* <span data-ttu-id="568fb-138">Bei der Ausführung eines virtuellen Computers fallen Azure-Gebühren an. Achten Sie daher darauf, dass er nur bei Bedarf ausgeführt wird.</span><span class="sxs-lookup"><span data-stu-id="568fb-138">Running a virtual machine incurs Azure charges, so you must be careful to have it running only when required.</span></span>

### <a name="azure-machine-learning"></a><span data-ttu-id="568fb-139">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="568fb-139">Azure Machine Learning</span></span>

<span data-ttu-id="568fb-140">Azure Machine Learning ist ein cloudbasierter Dienst zur Verwaltung von Machine Learning-Experimenten und -Modellen.</span><span class="sxs-lookup"><span data-stu-id="568fb-140">Azure Machine Learning is a cloud-based service for managing machine learning experiments and models.</span></span> <span data-ttu-id="568fb-141">Er enthält einen Experimentierdienst zur Nachverfolgung von Trainingsskripts für die Datenvorbereitung und -modellierung und speichert einen Verlauf aller Ausführungen, um die Modellleistung iterationsübergreifend vergleichen zu können.</span><span class="sxs-lookup"><span data-stu-id="568fb-141">It includes an experimentation service that tracks data preparation and modeling training scripts, maintaining a history of all executions so you can compare model performance across iterations.</span></span> <span data-ttu-id="568fb-142">Ein plattformübergreifendes Clienttool namens Azure Machine Learning Workbench bietet eine zentrale Schnittstelle für die Skriptverwaltung und den Verlauf und ermöglicht Datenspezialisten gleichzeitig die Erstellung von Skripts im Tool ihrer Wahl (beispielsweise Jupyter Notebooks oder Visual Studio Code).</span><span class="sxs-lookup"><span data-stu-id="568fb-142">A cross-platform client tool named Azure Machine Learning Workbench provides a central interface for script management and history, while still enabling data scientists to create scripts in their tool of choice, such as Jupyter Notebooks or Visual Studio Code.</span></span>

<span data-ttu-id="568fb-143">In Azure Machine Learning Workbench können Sie mithilfe der interaktiven Datenvorbereitungstools allgemeine Datentransformationsaufgaben vereinfachen und die Skriptausführungsumgebung für die Ausführung von Modelltrainingsskripts konfigurieren (lokal, in einem skalierbaren Docker-Container oder in Spark).</span><span class="sxs-lookup"><span data-stu-id="568fb-143">From Azure Machine Learning Workbench, you can use the interactive data preparation tools to simplify common data transformation tasks, and you can configure the script execution environment to run model training scripts locally, in a scalable Docker container, or in Spark.</span></span>

<span data-ttu-id="568fb-144">Das bereitstellungsbereite Modell können Sie dann mithilfe der Workbench-Umgebung verpacken und als Webdienst in einem Docker-Container, in Spark für Azure HDinsight, in Microsoft Machine Learning Server oder in SQL Server bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="568fb-144">When you are ready to deploy your model, use the Workbench environment to package the model and deploy it as a web service to a Docker container, Spark on Azure HDinsight, Microsoft Machine Learning Server, or SQL Server.</span></span> <span data-ttu-id="568fb-145">Mit dem Azure Machine Learning-Modellverwaltungsdienst können Sie Modellbereitstellungen in der Cloud, auf Edgegeräten oder innerhalb des gesamten Unternehmens nachverfolgen und verwalten.</span><span class="sxs-lookup"><span data-stu-id="568fb-145">The Azure Machine Learning Model Management service then enables you to track and manage model deployments in the cloud, on edge devices, or across the enterprise.</span></span>

<span data-ttu-id="568fb-146">Hauptvorteile:</span><span class="sxs-lookup"><span data-stu-id="568fb-146">Key benefits:</span></span>

* <span data-ttu-id="568fb-147">Zentrale Verwaltung von Skripts und Ausführungsverlauf zur Vereinfachung des Vergleichs von Modellversionen</span><span class="sxs-lookup"><span data-stu-id="568fb-147">Central management of scripts and run history, making it easy to compare model versions.</span></span>
* <span data-ttu-id="568fb-148">Interaktive Datentransformation über einen visuellen Editor</span><span class="sxs-lookup"><span data-stu-id="568fb-148">Interactive data transformation through a visual editor.</span></span>
* <span data-ttu-id="568fb-149">Komfortable Bereitstellung und Verwaltung von Modellen in der Cloud oder auf Edgegeräten</span><span class="sxs-lookup"><span data-stu-id="568fb-149">Easy deployment and management of models to the cloud or edge devices.</span></span>

<span data-ttu-id="568fb-150">Überlegungen:</span><span class="sxs-lookup"><span data-stu-id="568fb-150">Considerations:</span></span>
* <span data-ttu-id="568fb-151">Erfordert eine gewisse Erfahrung mit dem Modellverwaltungsmodell und der Umgebung des Workbench-Tools.</span><span class="sxs-lookup"><span data-stu-id="568fb-151">Requires some familiarity with the model management model and Workbench tool environment.</span></span>

### <a name="azure-batch-ai"></a><span data-ttu-id="568fb-152">Azure Batch AI</span><span class="sxs-lookup"><span data-stu-id="568fb-152">Azure Batch AI</span></span>

<span data-ttu-id="568fb-153">Mit Azure Batch AI können Sie Ihre Machine Learning-Experimente parallel ausführen und Modelle in einem Cluster aus virtuellen Computern mit GPUs bedarfsorientiert trainieren.</span><span class="sxs-lookup"><span data-stu-id="568fb-153">Azure Batch AI enables you to run your machine learning experiments in parallel, and perform model training at scale across a cluster of virtual machines with GPUs.</span></span> <span data-ttu-id="568fb-154">Batch AI-Training ermöglicht horizontales Hochskalieren von Deep Learning-Aufträgen in GPU-Clustern unter Verwendung von Frameworks wie Cognitive Toolkit, Caffe, Chainer und TensorFlow.</span><span class="sxs-lookup"><span data-stu-id="568fb-154">Batch AI training enables you to scale out deep learning jobs across clustered GPUs, using frameworks such as Cognitive Toolkit, Caffe, Chainer, and TensorFlow.</span></span> 

<span data-ttu-id="568fb-155">Modelle aus dem Batch AI-Training können mithilfe der Azure Machine Learning-Modellverwaltung bereitgestellt, verwaltet und überwacht werden.</span><span class="sxs-lookup"><span data-stu-id="568fb-155">Azure Machine Learning Model Management can be used to take models from Batch AI training to deploy, manage, and monitor them.</span></span> 

### <a name="azure-machine-learning-studio"></a><span data-ttu-id="568fb-156">Azure Machine Learning Studio</span><span class="sxs-lookup"><span data-stu-id="568fb-156">Azure Machine Learning Studio</span></span>

<span data-ttu-id="568fb-157">Azure Machine Learning Studio ist eine cloudbasierte, visuelle Entwicklungsumgebung, mit der Sie Datenexperimente erstellen, Machine Learning-Modelle trainieren und diese als Webdienste in Azure veröffentlichen können.</span><span class="sxs-lookup"><span data-stu-id="568fb-157">Azure Machine Learning Studio is a cloud-based, visual development environment for creating data experiments, training machine learning models, and publishing them as web services in Azure.</span></span> <span data-ttu-id="568fb-158">Die visuelle Drag & Drop-Oberfläche ermöglicht Datenspezialisten und Powerusern die schnelle Erstellung von Machine Learning-Lösungen und unterstützt benutzerdefinierte R- und Python-Logik sowie ein breites Spektrum an etablierten Statistikalgorithmen und Techniken für Machine Learning-Modellierungsaufgaben. Darüber hinaus verfügt sie über integrierte Jupyter Notebooks-Unterstützung.</span><span class="sxs-lookup"><span data-stu-id="568fb-158">Its visual drag-and-drop interface lets data scientists and power users create machine learning solutions quickly, while supporting custom R and Python logic, a wide range of established statistical algorithms and techniques for machine learning modeling tasks, and built-in support for Jupyter Notebooks.</span></span>

<span data-ttu-id="568fb-159">Hauptvorteile:</span><span class="sxs-lookup"><span data-stu-id="568fb-159">Key benefits:</span></span>

* <span data-ttu-id="568fb-160">Interaktive visuelle Oberfläche für Machine Learning-Modelle mit minimalem Programmieraufwand</span><span class="sxs-lookup"><span data-stu-id="568fb-160">Interactive visual interface enables machine learning modeling with minimal code.</span></span>
* <span data-ttu-id="568fb-161">Integrierte Jupyter Notebooks für Datenuntersuchungen</span><span class="sxs-lookup"><span data-stu-id="568fb-161">Built-in Jupyter Notebooks for data exploration.</span></span>
* <span data-ttu-id="568fb-162">Direkte Bereitstellung trainierter Modelle als Azure-Webdienste</span><span class="sxs-lookup"><span data-stu-id="568fb-162">Direct deployment of trained models as Azure web services.</span></span>

<span data-ttu-id="568fb-163">Überlegungen:</span><span class="sxs-lookup"><span data-stu-id="568fb-163">Considerations:</span></span>

* <span data-ttu-id="568fb-164">Begrenzte Skalierbarkeit:</span><span class="sxs-lookup"><span data-stu-id="568fb-164">Limited scalability.</span></span> <span data-ttu-id="568fb-165">Die maximale Größe eines Trainingsdatasets beträgt 10 GB.</span><span class="sxs-lookup"><span data-stu-id="568fb-165">The maximum size of a training dataset is 10 GB.</span></span>
* <span data-ttu-id="568fb-166">Nur online verfügbar:</span><span class="sxs-lookup"><span data-stu-id="568fb-166">Online only.</span></span> <span data-ttu-id="568fb-167">Es steht keine Offlineentwicklungsumgebung zur Verfügung.</span><span class="sxs-lookup"><span data-stu-id="568fb-167">No offline development environment.</span></span>

## <a name="tools-and-services-for-deploying-machine-learning-models"></a><span data-ttu-id="568fb-168">Tools und Dienste für die Bereitstellung von Machine Learning-Modellen</span><span class="sxs-lookup"><span data-stu-id="568fb-168">Tools and services for deploying machine learning models</span></span>

<span data-ttu-id="568fb-169">Nachdem ein Datenspezialist ein Machine Learning-Modell erstellt hat, müssen Sie es üblicherweise bereitstellen und über Anwendungen oder in anderen Datenflüssen nutzen.</span><span class="sxs-lookup"><span data-stu-id="568fb-169">After a data scientist has created a machine learning model, you will typically need to deploy it and consume it from applications or in other data flows.</span></span> <span data-ttu-id="568fb-170">Für Machine Learning-Modelle gibt es verschiedene potenzielle Bereitstellungsziele.</span><span class="sxs-lookup"><span data-stu-id="568fb-170">There are a number of potential deployment targets for machine learning models.</span></span>

### <a name="spark-on-azure-hdinsight"></a><span data-ttu-id="568fb-171">Spark in Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="568fb-171">Spark on Azure HDInsight</span></span>

<span data-ttu-id="568fb-172">Apache Spark enthält Spark MLlib – ein Framework und eine Bibliothek für Machine Learning-Modelle.</span><span class="sxs-lookup"><span data-stu-id="568fb-172">Apache Spark includes Spark MLlib, a framework and library for machine learning models.</span></span> <span data-ttu-id="568fb-173">Die Microsoft Machine Learning-Bibliothek für Spark (MMLSpark) unterstützt auch Deep Learning-Algorithmen für Prognosemodelle in Spark.</span><span class="sxs-lookup"><span data-stu-id="568fb-173">The Microsoft Machine Learning library for Spark (MMLSpark) also provides deep learning algorithm support for predictive models in Spark.</span></span>

<span data-ttu-id="568fb-174">Hauptvorteile:</span><span class="sxs-lookup"><span data-stu-id="568fb-174">Key benefits:</span></span>

* <span data-ttu-id="568fb-175">Spark ist eine verteilte Plattform, die hohe Skalierbarkeit für umfangreiche Machine Learning-Prozesse bietet.</span><span class="sxs-lookup"><span data-stu-id="568fb-175">Spark is a distributed platform that offers high scalability for high-volume machine learning processes.</span></span>
* <span data-ttu-id="568fb-176">Sie können Modelle direkt über Azure Machine Learning Workbench für Spark in HDinsight bereitstellen und sie mithilfe des Azure Machine Learning-Modellverwaltungsdiensts verwalten.</span><span class="sxs-lookup"><span data-stu-id="568fb-176">You can deploy models directly to Spark in HDinsight from Azure Machine Learning Workbench, and manage them using the Azure Machine Learning Model Management service.</span></span>

<span data-ttu-id="568fb-177">Überlegungen:</span><span class="sxs-lookup"><span data-stu-id="568fb-177">Considerations:</span></span>

* <span data-ttu-id="568fb-178">Spark wird in einem HDinsght-Cluster ausgeführt, sodass während der gesamten Ausführung Gebühren anfallen.</span><span class="sxs-lookup"><span data-stu-id="568fb-178">Spark runs in an HDinsght cluster that incurs charges the whole time it is running.</span></span> <span data-ttu-id="568fb-179">Dadurch können unnötige Kosten entstehen, wenn der Machine Learning-Dienst nur gelegentlich genutzt wird.</span><span class="sxs-lookup"><span data-stu-id="568fb-179">If the machine learning service will only be used occasionally, this may result in unnecessary costs.</span></span>

### <a name="web-service-in-a-container"></a><span data-ttu-id="568fb-180">Webdienst in einem Container</span><span class="sxs-lookup"><span data-stu-id="568fb-180">Web service in a container</span></span>

<span data-ttu-id="568fb-181">Sie können ein Machine Learning-Modell als Python-Webdienst in einem Docker-Container bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="568fb-181">You can deploy a machine learning model as a Python web service in a Docker container.</span></span> <span data-ttu-id="568fb-182">Sie können das Modell in Azure oder auf einem Edgegerät bereitstellen, wo es lokal mit den Daten verwendet werden kann, auf denen es basiert.</span><span class="sxs-lookup"><span data-stu-id="568fb-182">You can deploy the model to Azure or to an edge device, where it can be used locally with the data on which it operates.</span></span>

<span data-ttu-id="568fb-183">Hauptvorteile:</span><span class="sxs-lookup"><span data-stu-id="568fb-183">Key Benefits:</span></span>

* <span data-ttu-id="568fb-184">Mit Containern lassen sich Dienste einfach und üblicherweise kostengünstig verpacken und bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="568fb-184">Containers are a lightweight and generally cost effective way to package and deploy services.</span></span>
* <span data-ttu-id="568fb-185">Durch die Möglichkeit zur Bereitstellung auf einem Edgegerät können Sie Ihre Prognoselogik näher bei den Daten platzieren.</span><span class="sxs-lookup"><span data-stu-id="568fb-185">The ability to deploy to an edge device enables you to move your predictive logic closer to the data.</span></span>
* <span data-ttu-id="568fb-186">Die Bereitstellung in einem Container kann direkt über Azure Machine Learning Workbench erfolgen.</span><span class="sxs-lookup"><span data-stu-id="568fb-186">You can deploy to a container directly from Azure Machine Learning Workbench.</span></span>

<span data-ttu-id="568fb-187">Überlegungen:</span><span class="sxs-lookup"><span data-stu-id="568fb-187">Considerations:</span></span>

* <span data-ttu-id="568fb-188">Da dieses Bereitstellungsmodell auf Docker-Containern basiert, sollten Sie mit dieser Technologie vertraut sein, bevor Sie sie zur Bereitstellung eines Webdiensts nutzen.</span><span class="sxs-lookup"><span data-stu-id="568fb-188">This deployment model is based on Docker containers, so you should be familiar with this technology before deploying a web service this way.</span></span>

### <a name="microsoft-machine-learning-server"></a><span data-ttu-id="568fb-189">Microsoft Machine Learning Server</span><span class="sxs-lookup"><span data-stu-id="568fb-189">Microsoft Machine Learning Server</span></span>

<span data-ttu-id="568fb-190">Machine Learning Server (ehemals Microsoft R Server) ist eine skalierbare, speziell für Machine Learning-Szenarien konzipierte Plattform für R- und Python-Code.</span><span class="sxs-lookup"><span data-stu-id="568fb-190">Machine Learning Server (formerly Microsoft R Server) is a scalable platform for R and Python code, specifically designed for machine learning scenarios.</span></span>

<span data-ttu-id="568fb-191">Hauptvorteile:</span><span class="sxs-lookup"><span data-stu-id="568fb-191">Key benefits:</span></span>

* <span data-ttu-id="568fb-192">Hohe Skalierbarkeit</span><span class="sxs-lookup"><span data-stu-id="568fb-192">High scalability.</span></span>
* <span data-ttu-id="568fb-193">Direkte Bereitstellung über Azure Machine Learning Workbench</span><span class="sxs-lookup"><span data-stu-id="568fb-193">Direct deployment from Azure Machine Learning Workbench.</span></span>

<span data-ttu-id="568fb-194">Überlegungen:</span><span class="sxs-lookup"><span data-stu-id="568fb-194">Considerations:</span></span>

* <span data-ttu-id="568fb-195">Machine Learning Server muss in Ihrem Unternehmen bereitgestellt und verwaltet werden.</span><span class="sxs-lookup"><span data-stu-id="568fb-195">You need to deploy and manage Machine Learning Server in your enterprise.</span></span>

### <a name="microsoft-sql-server"></a><span data-ttu-id="568fb-196">Microsoft SQL Server</span><span class="sxs-lookup"><span data-stu-id="568fb-196">Microsoft SQL Server</span></span>

<span data-ttu-id="568fb-197">Microsoft SQL Server unterstützt R und Python nativ. Dadurch können Sie auf diesen Sprachen basierende Machine Learning-Modelle als Transact-SQL-Funktionen in einer Datenbank kapseln.</span><span class="sxs-lookup"><span data-stu-id="568fb-197">Microsoft SQL Server supports R and Python natively, enabling you to encapsulate machine learning models built in these languages as Transact-SQL functions in a database.</span></span>

<span data-ttu-id="568fb-198">Hauptvorteile:</span><span class="sxs-lookup"><span data-stu-id="568fb-198">Key benefits:</span></span>

* <span data-ttu-id="568fb-199">Einfache Einbeziehung in datenschichtinterne Logik durch Kapselung von Prognoselogik in einer Datenbankfunktion</span><span class="sxs-lookup"><span data-stu-id="568fb-199">Encapsulate predictive logic in a database function, making it easy to include in data-tier logic.</span></span>

<span data-ttu-id="568fb-200">Überlegungen:</span><span class="sxs-lookup"><span data-stu-id="568fb-200">Considerations:</span></span>

* <span data-ttu-id="568fb-201">Setzt eine SQL Server-Datenbank als Datenschicht für Ihre Anwendung voraus.</span><span class="sxs-lookup"><span data-stu-id="568fb-201">Assumes a SQL Server database as the data tier for your application.</span></span>

### <a name="azure-machine-learning-web-service"></a><span data-ttu-id="568fb-202">Azure Machine Learning-Webdienst</span><span class="sxs-lookup"><span data-stu-id="568fb-202">Azure Machine Learning web service</span></span>

<span data-ttu-id="568fb-203">Wenn Sie ein Machine Learning-Modell mithilfe von Azure Machine Learning Studio erstellen, können Sie es als Webdienst bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="568fb-203">When you create a machine learning model using Azure Machine Learning Studio, you can deploy it as a web service.</span></span> <span data-ttu-id="568fb-204">Dieser kann dann über eine REST-Schnittstelle von jeder beliebigen Clientanwendung genutzt werden, die über HTTP kommunizieren kann.</span><span class="sxs-lookup"><span data-stu-id="568fb-204">This can then be consumed through a REST interface from any client applications capable of communicating by HTTP.</span></span>

<span data-ttu-id="568fb-205">Hauptvorteile:</span><span class="sxs-lookup"><span data-stu-id="568fb-205">Key benefits:</span></span>

* <span data-ttu-id="568fb-206">Problemlose Entwicklung und Bereitstellung</span><span class="sxs-lookup"><span data-stu-id="568fb-206">Ease of development and deployment.</span></span>
* <span data-ttu-id="568fb-207">Webdienstverwaltungsportal mit grundlegenden Überwachungsmetriken</span><span class="sxs-lookup"><span data-stu-id="568fb-207">Web service management portal with basic monitoring metrics.</span></span>
* <span data-ttu-id="568fb-208">Integrierte Unterstützung des Aufrufens von Azure Machine Learning-Webdiensten über Azure Data Lake Analytics, Azure Data Factory und Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="568fb-208">Built-in support for calling Azure Machine Learning web services from Azure Data Lake Analytics, Azure Data Factory, and Azure Stream Analytics.</span></span>

<span data-ttu-id="568fb-209">Überlegungen:</span><span class="sxs-lookup"><span data-stu-id="568fb-209">Considerations:</span></span>

* <span data-ttu-id="568fb-210">Nur verfügbar für Modelle, die mit Azure Machine Learning Studio erstellt wurden.</span><span class="sxs-lookup"><span data-stu-id="568fb-210">Only available for models built using Azure Machine Learning Studio.</span></span>
* <span data-ttu-id="568fb-211">Nur webbasierter Zugriff: Trainierte Modelle können nicht lokal oder offline ausgeführt werden.</span><span class="sxs-lookup"><span data-stu-id="568fb-211">Web-based access only, trained models cannot run on-premises or offline.</span></span>

