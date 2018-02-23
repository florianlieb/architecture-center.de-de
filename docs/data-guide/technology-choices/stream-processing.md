---
title: "Auswählen einer Technologie für die Datenstromverarbeitung"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: e06f46e2951159219bd8cc430102e2ec0c5d6d4d
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-stream-processing-technology-in-azure"></a>Auswählen einer Technologie für die Datenstromverarbeitung in Azure

In diesem Artikel werden die Technologieoptionen für die Echtzeit-Datenstromverarbeitung in Azure verglichen.

Die Echtzeit-Datenstromverarbeitung liest Nachrichten aus warteschlangen- oder dateibasiertem Speicher, verarbeitet die Nachrichten und leitet das Ergebnis an eine andere Nachrichtenwarteschlange, einen anderen Dateispeicher oder eine andere Datenbank weiter. Die Verarbeitung beinhaltet unter Umständen das Abfragen, Filtern und Aggregieren von Nachrichten. Datenstromverarbeitungs-Engines müssen in der Lage sein, unbegrenzte Datenströme zu nutzen und mit minimaler Wartezeit Ergebnisse zu erzeugen. Weitere Informationen finden Sie unter [Real time processing](../scenarios/real-time-processing.md) (Verarbeitung in Echtzeit).

## <a name="what-are-your-options-when-choosing-a-technology-for-real-time-processing"></a>Welche Technologien für die Echtzeitverarbeitung stehen zur Verfügung?
In Azure erfüllen alle folgenden Datenspeicher die grundlegenden Anforderungen für die Echtzeitverarbeitung:
- [Azure Stream Analytics](/azure/stream-analytics/)
- [HDInsight mit Spark Streaming](/azure/hdinsight/spark/apache-spark-streaming-overview)
- [HDInsight mit Storm](/azure/hdinsight/storm/apache-storm-overview)
- [Azure-Funktionen](/azure/azure-functions/functions-overview)
- [WebJobs in Azure App Service](/azure/app-service/web-sites-create-web-jobs)

## <a name="key-selection-criteria"></a>Wichtige Auswahlkriterien

Beginnen Sie bei Szenarien für die Echtzeitverarbeitung mit der Auswahl des geeigneten Diensts für Ihre Anforderungen, indem Sie die folgenden Fragen beantworten:

- Bevorzugen Sie beim Erstellen der Logik für die Datenstromverarbeitung einen deklarativen oder einen imperativen Ansatz?

- Benötigen Sie integrierten Unterstützung für temporale Verarbeitung oder Windowing?

- Werden Daten in anderen Formaten als Avro, JSON oder CSV empfangen? Falls ja, ziehen Sie Optionen in Erwägung, die mithilfe von benutzerdefiniertem Code alle Formate unterstützen.

- Müssen Sie die Verarbeitung auf über 1 GB/s skalieren? Falls ja, ziehen Sie die Optionen in Erwägung, die mit der Clustergröße skaliert werden. 

## <a name="capability-matrix"></a>Funktionsmatrix

In der folgenden Tabellen sind die Hauptunterschiede der Funktionen zusammengefasst: 

### <a name="general-capabilities"></a>Allgemeine Funktionen
| | Azure Stream Analytics | HDInsight mit Spark Streaming | HDInsight mit Storm | Azure-Funktionen | WebJobs in Azure App Service |
| --- | --- | --- | --- | --- | --- | 
| Programmierbarkeit | Stream Analytics-Abfragesprache, JavaScript | Scala, Python, Java | Java, C# | C#, F#, Node.js | C#, Node.js, PHP, Java, Python |
| Programmierparadigma | Deklarativ | Mischung aus deklarativ und imperativ | Imperativ | Imperativ | Imperativ |    
| Preismodell | Nach Streamingeinheiten | Nach Clusterstunde | Nach Clusterstunde | Nach Funktionsausführung und Ressourcenverbrauch | Nach App Service-Plan-Stunde |  

### <a name="integration-capabilities"></a>Integrationsfunktionen
| | Azure Stream Analytics | HDInsight mit Spark Streaming | HDInsight mit Storm | Azure-Funktionen | WebJobs in Azure App Service |
| --- | --- | --- | --- | --- | --- | 
| Eingaben | [Stream Analytics-Eingaben](/azure/stream-analytics/stream-analytics-define-inputs)  | Event Hubs, IoT Hub, Kafka, HDFS  | Event Hubs, IoT Hub, Speicherblobs, Azure Data Lake Store  | [Unterstützte Bindungen](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Service Bus, Speicherwarteschlangen, Speicherblobs, Event Hubs, WebHooks, Cosmos DB, Dateien |
| Senken |  [Stream Analytics-Ausgaben](/azure/stream-analytics/stream-analytics-define-outputs) | HDFS | Event Hubs, Service Bus, Kafka | [Unterstützte Bindungen](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Service Bus, Speicherwarteschlangen, Speicherblobs, Event Hubs, WebHooks, Cosmos DB, Dateien | 

### <a name="processing-capabilities"></a>Verarbeitungsfunktionen
| | Azure Stream Analytics | HDInsight mit Spark Streaming | HDInsight mit Storm | Azure-Funktionen | WebJobs in Azure App Service |
| --- | --- | --- | --- | --- | --- | 
| Unterstützung für integrierte temporal Verarbeitung/Windowing | Ja | Ja | Ja | Nein  | Nein  |
| Datenformate für die Eingabe | Avro, JSON oder CSV, UTF-8-codiert | Jedes Format mit benutzerdefiniertem Code | Jedes Format mit benutzerdefiniertem Code | Jedes Format mit benutzerdefiniertem Code | Jedes Format mit benutzerdefiniertem Code |
| Skalierbarkeit | [Abfragepartitionen](/azure/stream-analytics/stream-analytics-parallelization) | Durch die Clustergröße begrenzt | Durch die Clustergröße begrenzt | Parallele Verarbeitung von bis zu 200 Funktions-App-Instanzen | Durch die Kapazität des App Service-Plans begrenzt | 
| Unterstützung für verspäteten Eingang und Behandlung von Ereignissen mit falscher Reihenfolge | Ja | Ja | Ja | Nein  | Nein  |

Weitere Informationen:

- [Choosing a real-time message ingestion technology](./real-time-ingestion.md) (Auswählen einer Technologie für die Echtzeiterfassung von Nachrichten)
- [Auswählen einer Stream Analytics-Plattform: Vergleich von Apache Storm und Azure Stream Analytics](/azure/stream-analytics/stream-analytics-comparison-storm)
- [Verarbeitung in Echtzeit](../scenarios/real-time-processing.md)
