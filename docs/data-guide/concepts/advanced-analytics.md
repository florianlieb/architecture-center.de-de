---
title: Erweiterte Analyse
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 31ba357fe37b1de35a6eea324d2d1d6766e172e5
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/27/2018
---
# <a name="advanced-analytics"></a>Erweiterte Analyse

Die erweiterte Analyse geht über historische Berichte und die Datenaggregation herkömmlicher Business Intelligence (BI) hinaus und verwendet mathematische, wahrscheinlichkeitsbasierte und statistische Modelle, um eine vorhersageorientierte Verarbeitung und eine automatisierte Entscheidungsfindung zu ermöglichen.

Erweiterte Analyselösungen umfassen in der Regel folgende Workloads:

* Interaktive Untersuchung und Visualisierung von Daten
* Training von Machine Learning-Modellen
* Vorhersageorientierte Verarbeitung (Echtzeit oder Batch)

Die meisten Architekturen für die erweiterte Analyse enthalten einige oder alle der folgenden Komponenten:

* **Datenspeicher**. Erweiterte Analyselösungen benötigen Daten zum Trainieren von Machine Learning-Modellen. Datenspezialisten müssen die Daten in der Regel untersuchen, um die für die Vorhersage geeigneten Merkmale sowie die statistischen Beziehungen zwischen ihnen und den Werten zu ermitteln, die sie vorhersagen (eine so genannte Bezeichnung). Bei der vorhergesagten Bezeichnung kann es sich um einen quantitativen Wert handeln – beispielsweise um einen zukünftigen finanziellen Wert oder um die Dauer einer Flugverzögerung in Minuten. Es kann sich dabei jedoch auch um eine kategorische Klasse wie „true“ oder „false“, „Flugverzögerung“ oder „keine Flugverzögerung“ oder um Kategorien wie „geringes Risiko“, „mittleres Risiko“ und „hohes Risiko“ handeln.

* **Batchverarbeitung:** Zum Trainieren eines Machine Learning-Modells müssen in der Regel große Datenmengen verarbeitet werden. Das Trainieren des Modells kann einige Zeit dauern (von mehreren Minuten bis hin zu mehreren Stunden). Für das Training können Skripts in Sprachen wie Python oder R verwendet werden. Außerdem kann das Training mithilfe von Plattformen für die verteilte Verarbeitung horizontal hochskaliert werden, um den Vorgang zu beschleunigen. Ein Beispiel für eine solche Plattform wäre etwa Apache Spark – gehostet in HDInsight oder in einem Docker-Container.

* **Echtzeiterfassung von Nachrichten:** In der Produktionsumgebung führen viele erweiterte Analysen Echtzeitdatenströme einem als Webdienst veröffentlichten Vorhersagemodell zu. Der eingehende Datenstrom wird in der Regel in einer Art von Warteschlange erfasst, und ein Streamverarbeitungsmodul ruft die Daten aus dieser Warteschlange ab und wendet die Vorhersage nahezu in Echtzeit auf die Eingabedaten an.  

* **Streamverarbeitung:** Mit einem trainierten Modell geht die Vorhersage (oder Bewertung) für eine bestimmte Gruppe von Merkmalen üblicherweise sehr schnell (im Millisekundenbereich). Nach der Erfassung von Echtzeitnachrichten können die relevanten Merkmalswerte an den Vorhersagedienst übergeben werden, um eine vorhergesagte Bezeichnung zu generieren.

* **Analysedatenspeicher:** In bestimmten Fällen werden die vorhergesagten Bezeichnungswerte zur Berichterstellung und späteren Analyse in den Analysedatenspeicher geschrieben.

* **Analysen und Berichte:** Wie der Name schon sagt, erzeugen erweiterte Analyselösungen in der Regel eine Art von Bericht oder Analysefeed mit den vorhergesagten Datenwerten. Vorhergesagte Bezeichnungswerte werden häufig auf Echtzeitdashboards verwendet.

* **Orchestrierung:** Die anfängliche Untersuchung und Modellierung wird interaktiv von Datenspezialisten durchgeführt, und viele erweiterte Analyselösungen trainieren Modelle regelmäßig mit neuen Daten, um die Präzision der Modelle weiter zu verbessern. Dieses erneute Trainieren kann mithilfe eines orchestrierten Workflows automatisiert werden.

## <a name="machine-learning"></a>Machine Learning
Machine Learning ist eine mathematische Modellierungstechnik zum Trainieren eines Vorhersagemodells. Im Allgemeinen wird ein statistischer Algorithmus auf ein großes Dataset mit historischen Daten angewendet, um Beziehungen zwischen den enthaltenen Feldern zu ermitteln.

Die Machine Learning-Modellierung wird in der Regel von Datenspezialisten durchgeführt. Diese müssen die Daten vor dem Trainieren des Modells sorgfältig untersuchen und vorbereiten. Die Untersuchung und Vorbereitung ist für gewöhnlich mit einer umfangreichen interaktiven Datenanalyse und -visualisierung verbunden. Dabei werden in der Regel Sprachen wie Python und R in interaktiven Tools und Umgebungen verwendet, die speziell für diese Aufgabe konzipiert sind.

Manchmal können auch [vortrainierte Modelle](/machine-learning-server/install/microsoftml-install-pretrained-models) verwendet werden, die auf von Microsoft gesammelten und entwickelten Trainingsdaten basieren. Ein vortrainiertes Modell hat den Vorteil, dass Sie neue Inhalte sofort bewerten und klassifizieren können, auch wenn Sie nicht über die nötigen Trainingsdaten oder über die Ressourcen für die Verwaltung umfangreicher Datasets oder das Training komplexer Modelle verfügen.

Machine Learning lässt sich in zwei Hauptkategorien unterteilen:

* **Beaufsichtigtes Lernen:** Beaufsichtigtes Lernen ist der gängigste Machine Learning-Ansatz. Bei diesem Lernmodell setzen sich die Quelldaten aus einer Reihe von *Merkmalsdatenfeldern* zusammen, die in einer mathematischen Beziehung zu mindestens einem *Bezeichnungsdatenfeld* stehen. In der Trainingsphase des Machine Learning-Prozesses enthält das Dataset sowohl Merkmale als auch bekannte Bezeichnungen, und es wird ein Algorithmus angewendet, um eine passende Funktion zu finden, die auf der Grundlage der Merkmale die entsprechenden Bezeichnungsvorhersagen berechnet. Üblicherweise wird eine Teilmenge des Trainingsdatasets zurückgehalten und zur Überprüfung der Leistung des trainierten Modells verwendet. Das trainierte Modell kann dann in der Produktionsumgebung bereitgestellt und zur Vorhersage unbekannter Werte verwendet werden. 

* **Unbeaufsichtigtes Lernen:** Bei einem Modell mit unbeaufsichtigtem Lernen enthalten die Trainingsdaten keine bekannten Bezeichnungswerte. Die Vorhersagen des Algorithmus basieren stattdessen auf dem ersten Kontakt mit den Daten. Die gängigste Form des unbeaufsichtigten Lernens ist *Clustering*. Dabei bestimmt der Algorithmus auf der Grundlage statistischer Ähnlichkeiten der Merkmale, wie sich die Daten am besten auf eine bestimmte Anzahl von Clustern aufteilen lassen. Beim Clustering ist das vorhergesagte Ergebnis die Clusternummer, zu der die eingegebenen Merkmale gehören. Unbeaufsichtigtes Lernen kann zwar gelegentlich zur direkten Generierung hilfreicher Vorhersagen genutzt werden (beispielsweise bei der Identifizierung von Benutzergruppen in einer Kundendatenbank mittels Clustering). Häufiger wird es jedoch zur Ermittlung der Daten verwendet, die sich beim Trainieren eines Modells am besten für einen Algorithmus für beaufsichtigtes Lernen eignen.

In Frage kommender Azure-Dienst:

- [Azure Machine Learning](/azure/machine-learning/)
- [Machine Learning Server (R Server) in HDInsight](/azure/hdinsight/r-server/r-server-overview)

## <a name="deep-learning"></a>Deep Learning

Machine Learning-Modelle, die auf mathematischen Techniken wie linearer oder logistischer Regression basieren, stehen bereits seit einiger Zeit zur Verfügung. Seit einiger Zeit werden vermehrt *Deep Learning-Techniken* genutzt, die auf neuronalen Netzwerken basieren. Dies ist unter anderem auf die Verfügbarkeit hochgradig skalierbarer Verarbeitungssysteme zurückzuführen, mit denen sich komplexe Modelle schneller trainieren lassen. Die zunehmende Verbreitung von Big Data erleichtert darüber hinaus das Trainieren von Deep Learning-Modellen in verschiedenen Domänen.

Berücksichtigen Sie bei der Gestaltung einer Cloudarchitektur für die erweiterte Analyse den Bedarf für eine umfangreiche Verarbeitung von Deep Learning-Modellen. Diese können über Plattformen für die verteilte Verarbeitung (beispielsweise Apache Spark) sowie über die neueste Generation virtueller Computer mit GPU-Hardware bereitgestellt werden.

In Frage kommender Azure-Dienst:

- [Deep Learning Virtual Machine](/azure/machine-learning/data-science-virtual-machine/deep-learning-dsvm-overview)
- [Apache Spark in HDInsight](/azure/hdinsight/spark/apache-spark-overview)

## <a name="artificial-intelligence"></a>Künstliche Intelligenz

Künstliche Intelligenz (KI) bezieht sich auf Szenarien, in denen ein Computer kognitive Fähigkeiten des Menschen wie Lernen und Problemlösung imitiert. Aufgrund der Verwendung von Machine Learning-Algorithmen wird KI als Oberbegriff betrachtet. Die meisten KI-Lösungen basieren auf einer Kombination aus Vorhersagediensten (die häufig als Webdienste implementiert werden) und Schnittstellen für natürliche Sprache (wie etwa Chatbots, die per Text oder Sprache interagieren) und werden von KI-Apps auf mobilen Geräten oder anderen Clients angeboten. Manchmal ist das Machine Learning-Modell auch in die KI-App eingebettet. 

## <a name="model-deployment"></a>Modellbereitstellung

Die Vorhersagedienste, die KI-Anwendungen unterstützen, können benutzerdefinierte Machine Learning-Modelle oder vorgefertigte kognitive Dienste nutzen, die den Zugriff auf vortrainierte Modelle ermöglichen. Die Bereitstellung benutzerdefinierter Modelle in der Produktionsumgebung wird als Operationalisierung bezeichnet. Dabei werden die KI-Modelle, die in der Verarbeitungsumgebung trainiert und getestet wurden, serialisiert und zur Durchführung von Batch- oder Self-Service-Vorhersagen für externe Anwendungen und Dienste bereitgestellt. Zur Verwendung der Vorhersagefunktion des Modells wird es deserialisiert und unter Verwendung der gleichen Machine Learning-Bibliothek geladen, die den Algorithmus enthält, mit dem das Modell ursprünglich trainiert wurde. Diese Bibliothek bietet Vorhersagefunktionen (häufig Bewertung oder Vorhersage genannt), die auf der Grundlage des Modells und der Merkmale eine Vorhersage zurückgeben. Diese Logik wird dann in eine Funktion eingeschlossen, die direkt von einer Anwendung aufgerufen oder als Webdienst verfügbar gemacht werden kann. 

In Frage kommender Azure-Dienst:

- [Azure Machine Learning](/azure/machine-learning/)
- [Machine Learning Server (R Server) in HDInsight](/azure/hdinsight/r-server/r-server-overview)


## <a name="see-also"></a>Weitere Informationen

- [Auswählen einer Cognitive Services-Technologie](../technology-choices/cognitive-services.md)
- [Auswählen einer Machine Learning-Technologie](../technology-choices/data-science-and-machine-learning.md)
