---
title: Zeitreihendaten
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: ceb8f34d4fd950e5270edfea05945a824c4492f0
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="time-series-solutions"></a>Zeitreihenlösungen

Bei Zeitreihendaten handelt es sich um eine nach Zeit strukturierte Gruppe von Werten. Beispiele für Zeitreihendaten sind Sensordaten, Aktienkurse, Clickstreamdaten und Anwendungstelemetriedaten. Durch die Analyse von Zeitreihendaten können historische Trends ermittelt, Warnungen in Echtzeit generiert oder Vorhersagemodelle erstellt werden.

![Time Series Insights](./images/time-series-insights.png) 

Zeitreihendaten stellen dar, wie eine Ressource oder ein Prozess sich im Zeitablauf ändert. Die Daten besitzen einen Zeitstempel, und Zeit ist die aussagekräftigste Achse für die Betrachtung oder Analyse der Daten. Zeitreihendaten gehen normalerweise in zeitlich korrekter Reihenfolge ein und werden in der Regel als Einfügung und nicht als Aktualisierung der Datenbank behandelt. Veränderungen werden also im Zeitverlauf gemessen, wodurch Sie sich die Daten aus der Vergangenheit ansehen und zukünftige Veränderungen vorhersagen können. Zeitreihendaten lassen sich somit am besten mit Punkt- oder Liniendiagrammen visualisieren.

![In einem Liniendiagramm visualisierte Zeitreihendaten](./images/time-series-chart.png)

Im Anschluss folgen einige Beispiele für Zeitreihendaten:

- Im Zeitverlauf erfasste Aktienkurse zur Erkennung von Trends
- Serverleistung wie CPU-Auslastung, E/A-Last, Arbeitsspeicherauslastung und Auslastung der Netzwerkbandbreite
- Telemetriedaten von Sensoren an Industriemaschinen zur Erkennung eines bevorstehenden Ausfalls und Auslösung von Warnbenachrichtigungen
- Echtzeittelemetriedaten eines Fahrzeugs wie Geschwindigkeit, Bremsen und Beschleunigen über einen bestimmen Zeitraum, um eine aggregierte Risikobewertung für den Fahrer zu erzeugen

Wie Sie sehen, ist in jedem dieser Fälle Zeit die sinnvollste Achse. Die Darstellung der Ereignisse in der Reihenfolge ihres Eintreffens ist ein zentrales Merkmal von Zeitreihendaten, da sich dadurch eine natürliche zeitliche Sortierung ergibt. Dies unterscheidet sich von Daten, die für OLTP-Standarddatenpipelines erfasst werden. Dort können die Daten in beliebiger Reihenfolge eingefügt und jederzeit aktualisiert werden.

## <a name="when-to-use-this-solution"></a>Verwendung dieser Lösung

Entscheiden Sie sich für eine Zeitreihenlösung, wenn Sie Daten erfassen müssen, deren strategischer Nutzen auf Veränderungen im Zeitverlauf zurückgeht, und Sie hauptsächlich neue Daten einfügen und nur ganz selten Aktualisierungen vornehmen (wenn überhaupt). Auf der Grundlage dieser Informationen können Sie unter anderem Anomalien erkennen, Trends visualisieren und aktuelle Daten mit historischen Daten vergleichen. Diese Art von Architektur eignet sich zudem am besten für Vorhersagemodelle und Ergebnisprognosen, da Sie über einen Änderungsverlauf verfügen, der in verschiedensten Vorhersagemodellen genutzt werden kann. 

Die Verwendung von Zeitreihen bietet folgende Vorteile:

* Eindeutige Darstellung der Veränderungen einer Ressource oder eines Prozesses im Zeitverlauf
* Schnelle Erkennung von Veränderungen für eine Reihe verwandter Quellen, wodurch Anomalien und sich entwickelnde Trends deutlich sichtbar werden
* Beste Option für Vorhersagemodelle und Prognosen

### <a name="internet-of-things-iot"></a>Internet der Dinge (IoT, Internet of Things)

Von IoT-Geräten gesammelte Daten eignen sich bestens für die Speicherung und Analyse von Zeitreihendaten. Die eingehenden Daten werden eingefügt und selten aktualisiert (wenn überhaupt). Die Daten werden mit einem Zeitstempel versehen und in der Reihenfolge eingefügt, in der sie empfangen wurden. Darüber hinaus werden diese Daten üblicherweise in chronologischer Reihenfolge angezeigt, sodass Benutzer Trends erkennen, Anomalien entdecken und die Informationen für Vorhersageanalysen verwenden können.

Weitere Informationen finden Sie im Artikel zum [Internet der Dinge](../concepts/big-data.md#internet-of-things-iot).

### <a name="real-time-analytics"></a>Echtzeitanalysen

Zeitreihendaten sind oftmals zeitkritisch. Das bedeutet, sie müssen zeitnah verarbeitet werden, um Trends in Echtzeit erkennen zu können oder Warnungen zu generieren. In diesen Szenarien kann jede verspätete Erkenntnis zu Ausfällen führen und negative Auswirkungen auf das Unternehmen haben. Darüber hinaus müssen häufig Daten aus verschiedensten Quellen (beispielsweise von Sensoren) korreliert werden.

Im Idealfall verfügen Sie über eine Streamverarbeitungsebene, die die eingehenden Daten in Echtzeit behandelt und sämtliche Daten mit hoher Präzision und Granularität verarbeitet. Je nach Streamingarchitektur und den Komponenten Ihrer Ebenen für Streampufferung und Streamverarbeitung ist dies jedoch nicht immer möglich. Gegebenenfalls müssen die Zeitreihendaten reduziert und gewisse Abstriche bei der Präzision gemacht werden. Zu diesem Zweck werden gleitende Zeitfenster (beispielsweise mehrere Sekunden) verarbeitet, damit die Verarbeitungsebene die Berechnungen zeitnah ausführen kann. Bei der Darstellung größerer Zeiträume ist unter Umständen auch ein Downsampling oder eine Aggregation der Daten erforderlich – etwa beim Zoomen der Ansicht, um die erfassten Daten mehrerer Monate anzuzeigen.

## <a name="challenges"></a>Herausforderungen

* Zeitreihendaten sind häufig äußerst umfangreich. Das gilt besonders in IoT-Szenarien. Die Speicherung, Indizierung, Abfrage, Analyse und Visualisierung von Zeitreihendaten kann sich als Herausforderung erweisen. 
* Es ist nicht immer einfach, die richtige Kombination aus Hochgeschwindigkeitsspeicher und leistungsstarken Computevorgängen für die Arbeit mit Echtzeitanalysen zu finden und gleichzeitig die Markteinführungszeit und die Gesamtkosten für Investitionen zu minimieren.

## <a name="architecture"></a>Architektur

In vielen Szenarien mit Zeitreihendaten (beispielsweise IoT) werden die Daten in Echtzeit erfasst. Daher ist eine Architektur mit [Echtzeitverarbeitung](./real-time-processing.md) angebracht. 

Daten aus einzelnen oder mehreren Datenquellen werden durch [IoT Hub](/azure/iot-hub/), [Event Hubs](/azure/event-hubs/) oder [Kafka in HDInsight](/azure/hdinsight/kafka/apache-kafka-introduction) auf der Streampufferungsebene erfasst. Als Nächstes werden die Daten auf der Streamverarbeitungsebene verarbeitet und optional für Vorhersageanalysen an einen Machine Learning-Dienst weitergegeben. Die verarbeiteten Daten werden in einem Analysedatenspeicher wie [HBase](/azure/hdinsight/hbase/apache-hbase-overview), [Azure Cosmos DB](/azure/cosmos-db/), Azure Data Lake oder Blob Storage gespeichert. Zur Analyse können die Zeitreihendaten mit einer Analyse- und Berichtsanwendung oder einem entsprechenden Dienst angezeigt werden. Beispiele wären etwa Power BI und OpenTSDB (sofern in HBase gespeichert).

Eine weitere Option ist die Verwendung von [Azure Time Series Insights](/azure/time-series-insights/). Time Series Insights ist ein vollständig verwalteter Dienst für Zeitreihendaten. In dieser Architektur wird Time Series Insights für die Streamverarbeitung, als Datenspeicher und für Analysen und Berichte verwendet. Der Dienst akzeptiert Streamingdaten von IoT Hub oder Event Hubs, speichert, verarbeitet und analysiert sie und zeigt sie nahezu in Echtzeit an. Er aggregiert die Daten nicht vorab, sondern speichert die Rohereignisse.

Time Series Insights kann an ein Schema angepasst werden, sodass Sie Insights ganz ohne vorherige Datenvorbereitung gewinnen können. Dadurch lassen sich nahtlos verschiedenste Datenquellen erkunden, vergleichen und korrelieren. Darüber hinaus bietet der Dienst SQL-ähnliche Filter und Aggregate, die Möglichkeit zum Erstellen, Visualisieren, Vergleichen und Überlagern verschiedener Zeitreihenmuster und Wärmebilder sowie die Möglichkeit zum Speichern und Weitergeben von Abfragen. 

## <a name="technology-choices"></a>Auswahl der Technologie

- [Datenspeicher](../technology-choices/data-storage.md)
- [Analysen, Visualisierungen und Berichte](../technology-choices/analysis-visualizations-reporting.md)
- [Analysedatenspeicher](../technology-choices/analytical-data-stores.md)
- [Streamverarbeitung](../technology-choices/stream-processing.md)