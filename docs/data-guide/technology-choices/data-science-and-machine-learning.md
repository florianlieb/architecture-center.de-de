---
title: Auswählen einer Machine Learning-Technologie
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 995349c795066ec3067b20ad2615e40b0fb152db
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-machine-learning-technology-in-azure"></a>Auswählen einer Machine Learning-Technologie in Azure

Bei Data Science und Machine Learning handelt es sich um eine Workload, die in der Regel von Datenspezialisten verwendet wird. Sie erfordert besondere Tools, die häufig speziell für die Art von interaktiver Datenuntersuchung und die Modellierungsaufgaben konzipiert sind, die ein Datenspezialist benötigt.

Machine Learning-Lösungen sind iterativ aufgebaut und umfassen zwei Phasen:
* Datenvorbereitung und -modellierung
* Bereitstellung und Nutzung von Prognosediensten

## <a name="tools-and-services-for-data-preparation-and-modeling"></a>Tools und Dienste zur Datenvorbereitung und -modellierung
Für die Arbeit mit Daten bevorzugen Datenspezialisten in der Regel benutzerdefinierten, in Python oder R geschriebenen Code. Dieser wird im Allgemeinen interaktiv ausgeführt und von den Datenspezialisten zum Abfragen und Untersuchen der Daten verwendet. Dabei werden Visualisierungen und Statistiken generiert, um Beziehungen zu ermitteln. Den Datenspezialisten stehen zahlreiche interaktive Umgebungen für R und Python zur Verfügung. Besonders beliebt ist **Jupyter Notebooks**: Diese Umgebung bietet eine browserbasierte Shell, mit der Datenspezialisten *Notebook-Dateien* erstellen können, die R- oder Python-Code sowie Markdowntext enthalten. Dies ermöglicht eine effektive Zusammenarbeit, da Code und Ergebnisse in einem einzelnen Dokument weitergegeben und dokumentiert werden können.

Folgende Tools werden ebenfalls gerne verwendet:
* **Spyder:** Die interaktive Entwicklungsumgebung (Interactive Development Environment, IDE), die mit der Python-Distribution Anaconda bereitgestellt wird.
* **R Studio:** Eine IDE für die Programmiersprache R.
* **Visual Studio Code:** Eine einfache, plattformübergreifende Programmierumgebung, die sowohl Python als auch gängige Frameworks für Machine Learning und KI-Entwicklung unterstützt.

Neben diesen Tools können Datenspezialisten auch Azure-Dienste nutzen, um die Code- und Modellverwaltung zu vereinfachen.

### <a name="azure-notebooks"></a>Azure Notebooks
Azure Notebooks ist ein Jupyter Notebooks-Onlinedienst, mit dem Datenspezialisten Jupyter Notebooks in cloudbasierten Bibliotheken erstellen, ausführen und weitergeben können.

Hauptvorteile:

* Kostenloser Dienst: Sie benötigen kein Azure-Abonnement.
* Keine lokale Installation von Jupyter und den unterstützenden R- oder Python-Distributionen erforderlich: Ein Browser genügt.
* Verwalten Sie Ihre eigenen Onlinebibliotheken, und greifen Sie von einem beliebigen Gerät aus darauf zu.
* Teilen Sie Ihre Notebooks mit Projektmitarbeitern.

Überlegungen:

* Die Notebooks stehen offline nicht zur Verfügung.
* Die eingeschränkten Verarbeitungsfunktionen des kostenlosen Notebookdiensts reichen für umfangreiche oder komplexe Modelle unter Umständen nicht aus.

### <a name="data-science-virtual-machine"></a>Virtueller Data Science-Computer
Der virtuelle Data Science-Computer ist ein Image eines virtuellen Azure-Computers mit den Tools und Frameworks, die üblicherweise von Datenspezialisten verwendet werden – einschließlich R, Python, Jupyter Notebooks, Visual Studio Code und Bibliotheken für die Machine Learning-Modellierung (etwa das Microsoft Cognitive Toolkit). Diese Tools können komplex und zeitaufwendig zu installieren sein. Darüber hinaus können sie zahlreiche Abhängigkeiten enthalten, die immer wieder zu Problemen mit der Versionsverwaltung führen. Mit einem vorinstallierten Image verbringen Datenspezialisten weniger Zeit mit der Behandlung von Umgebungsproblemen und können sich stattdessen auf die nötigen Datenuntersuchungen und Modellierungsaufgaben konzentrieren.

Hauptvorteile:
* Geringerer Installations-, Verwaltungs- und Problembehandlungsaufwand für Data Science-Tools und -Frameworks.
* Verfügbarkeit der neuesten Versionen aller gängigen Tools und Frameworks
* VM-Optionen mit hochgradig skalierbaren Images und GPU-Funktionen für intensive Datenmodellierung

Überlegungen:
* Der virtuelle Computer steht offline nicht zur Verfügung.
* Bei der Ausführung eines virtuellen Computers fallen Azure-Gebühren an. Achten Sie daher darauf, dass er nur bei Bedarf ausgeführt wird.

### <a name="azure-machine-learning"></a>Azure Machine Learning

Azure Machine Learning ist ein cloudbasierter Dienst zur Verwaltung von Machine Learning-Experimenten und -Modellen. Er enthält einen Experimentierdienst zur Nachverfolgung von Trainingsskripts für die Datenvorbereitung und -modellierung und speichert einen Verlauf aller Ausführungen, um die Modellleistung iterationsübergreifend vergleichen zu können. Ein plattformübergreifendes Clienttool namens Azure Machine Learning Workbench bietet eine zentrale Schnittstelle für die Skriptverwaltung und den Verlauf und ermöglicht Datenspezialisten gleichzeitig die Erstellung von Skripts im Tool ihrer Wahl (beispielsweise Jupyter Notebooks oder Visual Studio Code).

In Azure Machine Learning Workbench können Sie mithilfe der interaktiven Datenvorbereitungstools allgemeine Datentransformationsaufgaben vereinfachen und die Skriptausführungsumgebung für die Ausführung von Modelltrainingsskripts konfigurieren (lokal, in einem skalierbaren Docker-Container oder in Spark).

Das bereitstellungsbereite Modell können Sie dann mithilfe der Workbench-Umgebung verpacken und als Webdienst in einem Docker-Container, in Spark für Azure HDinsight, in Microsoft Machine Learning Server oder in SQL Server bereitstellen. Mit dem Azure Machine Learning-Modellverwaltungsdienst können Sie Modellbereitstellungen in der Cloud, auf Edgegeräten oder innerhalb des gesamten Unternehmens nachverfolgen und verwalten.

Hauptvorteile:

* Zentrale Verwaltung von Skripts und Ausführungsverlauf zur Vereinfachung des Vergleichs von Modellversionen
* Interaktive Datentransformation über einen visuellen Editor
* Komfortable Bereitstellung und Verwaltung von Modellen in der Cloud oder auf Edgegeräten

Überlegungen:
* Erfordert eine gewisse Erfahrung mit dem Modellverwaltungsmodell und der Umgebung des Workbench-Tools.

### <a name="azure-batch-ai"></a>Azure Batch AI

Mit Azure Batch AI können Sie Ihre Machine Learning-Experimente parallel ausführen und Modelle in einem Cluster aus virtuellen Computern mit GPUs bedarfsorientiert trainieren. Batch AI-Training ermöglicht horizontales Hochskalieren von Deep Learning-Aufträgen in GPU-Clustern unter Verwendung von Frameworks wie Cognitive Toolkit, Caffe, Chainer und TensorFlow. 

Modelle aus dem Batch AI-Training können mithilfe der Azure Machine Learning-Modellverwaltung bereitgestellt, verwaltet und überwacht werden. 

### <a name="azure-machine-learning-studio"></a>Azure Machine Learning Studio

Azure Machine Learning Studio ist eine cloudbasierte, visuelle Entwicklungsumgebung, mit der Sie Datenexperimente erstellen, Machine Learning-Modelle trainieren und diese als Webdienste in Azure veröffentlichen können. Die visuelle Drag & Drop-Oberfläche ermöglicht Datenspezialisten und Powerusern die schnelle Erstellung von Machine Learning-Lösungen und unterstützt benutzerdefinierte R- und Python-Logik sowie ein breites Spektrum an etablierten Statistikalgorithmen und Techniken für Machine Learning-Modellierungsaufgaben. Darüber hinaus verfügt sie über integrierte Jupyter Notebooks-Unterstützung.

Hauptvorteile:

* Interaktive visuelle Oberfläche für Machine Learning-Modelle mit minimalem Programmieraufwand
* Integrierte Jupyter Notebooks für Datenuntersuchungen
* Direkte Bereitstellung trainierter Modelle als Azure-Webdienste

Überlegungen:

* Begrenzte Skalierbarkeit: Die maximale Größe eines Trainingsdatasets beträgt 10 GB.
* Nur online verfügbar: Es steht keine Offlineentwicklungsumgebung zur Verfügung.

## <a name="tools-and-services-for-deploying-machine-learning-models"></a>Tools und Dienste für die Bereitstellung von Machine Learning-Modellen

Nachdem ein Datenspezialist ein Machine Learning-Modell erstellt hat, müssen Sie es üblicherweise bereitstellen und über Anwendungen oder in anderen Datenflüssen nutzen. Für Machine Learning-Modelle gibt es verschiedene potenzielle Bereitstellungsziele.

### <a name="spark-on-azure-hdinsight"></a>Spark in Azure HDInsight

Apache Spark enthält Spark MLlib – ein Framework und eine Bibliothek für Machine Learning-Modelle. Die Microsoft Machine Learning-Bibliothek für Spark (MMLSpark) unterstützt auch Deep Learning-Algorithmen für Prognosemodelle in Spark.

Hauptvorteile:

* Spark ist eine verteilte Plattform, die hohe Skalierbarkeit für umfangreiche Machine Learning-Prozesse bietet.
* Sie können Modelle direkt über Azure Machine Learning Workbench für Spark in HDinsight bereitstellen und sie mithilfe des Azure Machine Learning-Modellverwaltungsdiensts verwalten.

Überlegungen:

* Spark wird in einem HDinsght-Cluster ausgeführt, sodass während der gesamten Ausführung Gebühren anfallen. Dadurch können unnötige Kosten entstehen, wenn der Machine Learning-Dienst nur gelegentlich genutzt wird.

### <a name="web-service-in-a-container"></a>Webdienst in einem Container

Sie können ein Machine Learning-Modell als Python-Webdienst in einem Docker-Container bereitstellen. Sie können das Modell in Azure oder auf einem Edgegerät bereitstellen, wo es lokal mit den Daten verwendet werden kann, auf denen es basiert.

Hauptvorteile:

* Mit Containern lassen sich Dienste einfach und üblicherweise kostengünstig verpacken und bereitstellen.
* Durch die Möglichkeit zur Bereitstellung auf einem Edgegerät können Sie Ihre Prognoselogik näher bei den Daten platzieren.
* Die Bereitstellung in einem Container kann direkt über Azure Machine Learning Workbench erfolgen.

Überlegungen:

* Da dieses Bereitstellungsmodell auf Docker-Containern basiert, sollten Sie mit dieser Technologie vertraut sein, bevor Sie sie zur Bereitstellung eines Webdiensts nutzen.

### <a name="microsoft-machine-learning-server"></a>Microsoft Machine Learning Server

Machine Learning Server (ehemals Microsoft R Server) ist eine skalierbare, speziell für Machine Learning-Szenarien konzipierte Plattform für R- und Python-Code.

Hauptvorteile:

* Hohe Skalierbarkeit
* Direkte Bereitstellung über Azure Machine Learning Workbench

Überlegungen:

* Machine Learning Server muss in Ihrem Unternehmen bereitgestellt und verwaltet werden.

### <a name="microsoft-sql-server"></a>Microsoft SQL Server

Microsoft SQL Server unterstützt R und Python nativ. Dadurch können Sie auf diesen Sprachen basierende Machine Learning-Modelle als Transact-SQL-Funktionen in einer Datenbank kapseln.

Hauptvorteile:

* Einfache Einbeziehung in datenschichtinterne Logik durch Kapselung von Prognoselogik in einer Datenbankfunktion

Überlegungen:

* Setzt eine SQL Server-Datenbank als Datenschicht für Ihre Anwendung voraus.

### <a name="azure-machine-learning-web-service"></a>Azure Machine Learning-Webdienst

Wenn Sie ein Machine Learning-Modell mithilfe von Azure Machine Learning Studio erstellen, können Sie es als Webdienst bereitstellen. Dieser kann dann über eine REST-Schnittstelle von jeder beliebigen Clientanwendung genutzt werden, die über HTTP kommunizieren kann.

Hauptvorteile:

* Problemlose Entwicklung und Bereitstellung
* Webdienstverwaltungsportal mit grundlegenden Überwachungsmetriken
* Integrierte Unterstützung des Aufrufens von Azure Machine Learning-Webdiensten über Azure Data Lake Analytics, Azure Data Factory und Azure Stream Analytics

Überlegungen:

* Nur verfügbar für Modelle, die mit Azure Machine Learning Studio erstellt wurden.
* Nur webbasierter Zugriff: Trainierte Modelle können nicht lokal oder offline ausgeführt werden.

