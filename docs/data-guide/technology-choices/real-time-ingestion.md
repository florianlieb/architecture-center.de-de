---
title: "Auswählen einer Technologie für die Echtzeiterfassung von Nachrichten"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 2e6578b779950b5ef11bda7b8ba1fb2e45e09f4e
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/23/2018
---
# <a name="choosing-a-real-time-message-ingestion-technology-in-azure"></a>Auswählen einer Technologie für die Echtzeiterfassung von Nachrichten in Azure

Bei der Echtzeitverarbeitung werden Datenströme in Echtzeit erfasst und mit minimaler Wartezeit verarbeitet. Viele Lösungen für die Echtzeitverarbeitung erfordern jedoch einen Speicher für die Erfassung von Nachrichten, der als Puffer für Nachrichten fungiert. Der Speicher muss zudem die Verarbeitung mit horizontaler Skalierung, eine zuverlässige Übermittlung sowie weitere Semantik für das Nachrichtenqueuing unterstützen. 

## <a name="what-are-your-options-for-real-time-message-ingestion"></a>Welche Optionen stehen Ihnen bei der Auswahl einer Technologie für die Echtzeiterfassung von Nachrichten zur Verfügung?

- [Azure Event Hubs](/azure/event-hubs/)
- [Azure IoT Hub](/azure/iot-hub/)
- [Kafka in HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started)

## <a name="azure-event-hubs"></a>Azure Event Hubs

[Azure Event Hubs](/azure/event-hubs/) ist eine hochgradig skalierbare Datenstreamingplattform und ein Dienst zur Erfassung von Ereignissen, der pro Sekunde Millionen von Ereignissen empfangen und verarbeiten kann. Event Hubs kann Ereignisse, Daten oder Telemetriedaten, die von verteilter Software und verteilten Geräten erzeugt wurden, verarbeiten und speichern. An einen Event Hub gesendete Daten können transformiert und mit einem beliebigen Echtzeitanalyse-Anbieter oder Batchverarbeitungs-/Speicheradapter gespeichert werden. Event Hubs bietet Funktionen für das Veröffentlichen/Abonnieren in großem Umfang mit geringer Wartezeit und eignet sich daher für Big Data-Szenarien.

## <a name="azure-iot-hub"></a>Azure IoT Hub

[Azure IoT Hub](/azure/iot-hub/) ist ein verwalteter Dienst, der eine zuverlässige und sichere bidirektionale Kommunikation zwischen Millionen von IoT-Geräten und einem cloudbasierten Back-End ermöglicht.

IoT Hub bietet folgende Features:

* Mehrere Optionen für die Kommunikation zwischen Geräten und Cloud (Device-to-Cloud, D2C) sowie zwischen Cloud und Geräten (Cloud-to-Device, C2D). Dazu zählen unter anderem unidirektionales Messaging, Dateiübertragungen und Anforderung-Antwort-Methoden.
* Nachrichtenrouting zu anderen Azure-Diensten.
* Abfragbarer Speicher für Gerätemetadaten und synchronisierte Zustandsinformationen.
* Sichere Kommunikation und Zugriffssteuerung mithilfe von geräteabhängigen Sicherheitsschlüsseln oder X.509-Zertifikaten.
* Überwachung der Ereignisse zur Verwaltung der Gerätekonnektivität und -identität.

Im Hinblick auf die Erfassung von Nachrichten ist IoT Hub von der Funktion her mit Event Hubs vergleichbar. IoT Hub wurde jedoch speziell für die Verwaltung der IoT-Gerätekonnektivität konzipiert und nicht nur für die Erfassung von Nachrichten. Weitere Informationen finden Sie unter [Vergleich zwischen Azure IoT Hub und Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubs). 

## <a name="kafka-on-hdinsight"></a>Kafka in HDInsight

[Apache Kafka](https://kafka.apache.org/) ist eine verteilte Open Source-Streamingplattform, die zum Erstellen von Echtzeit-Datenpipelines und Streaminganwendungen verwendet werden kann. Kafka verfügt auch über Nachrichtenbrokerfunktionen, die einer Nachrichtenwarteschlange ähneln, über die Sie benannte Datenströme veröffentlichen und diese abonnieren können. Apache Kafka ist horizontal skalierbar, fehlertolerant und extrem schnell. [Kafka in HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started) stellt Kafka als verwalteten, hoch skalierbaren und hoch verfügbaren Dienst in Azure bereit. 

Gängige Anwendungsfälle für Kafka sind beispielsweise:

* **Messaging**: Da das Veröffentlichen/Abonnieren-Nachrichtenmuster unterstützt wird, wird Kafka häufig als Nachrichtenbroker genutzt.
* **Aktivitätsüberwachung**. Da Kafka die geordnete Protokollierung von Datensätzen unterstützt, kann die Anwendung zum Nachverfolgen und Neuerstellen von Aktivitäten verwendet werden (beispielsweise Benutzeraktionen auf einer Website).
* **Aggregation**. Mit der Datenstromverarbeitung können Sie Informationen aus unterschiedlichen Datenströmen aggregieren, um die Informationen zu operativen Daten zu kombinieren und zu zentralisieren.
* **Transformation**. Mit der Datenstromverarbeitung können Sie Daten aus mehreren Eingabethemen zu einem oder mehreren Ausgabethemen kombinieren und erweitern.

## <a name="key-selection-criteria"></a>Wichtige Auswahlkriterien

Beantworten Sie die folgenden Fragen, um die Auswahl einzuschränken:

- Benötigen Sie eine bidirektionale Kommunikation zwischen Ihren IoT-Geräten und Azure? Wenn dies der Fall ist, ist IoT Hub für Sie die richtige Wahl.

- Müssen Sie den Zugriff für einzelne Geräte verwalten und in der Lage sein, den Zugriff auf ein bestimmtes Gerät zu widerrufen? Wenn dies der Fall ist, ist IoT Hub für Sie die richtige Wahl.

## <a name="capability-matrix"></a>Funktionsmatrix

In den folgenden Tabellen sind die Hauptunterschiede der Funktionen zusammengefasst: 

| | IoT Hub | Event Hubs | Kafka in HDInsight |
| --- | --- | --- | --- |
| Cloud-zu-Gerät-Kommunikation | Ja | Nein  | Nein  |
| Vom Gerät initiierter Dateiupload | Ja | Nein  | Nein  |
| Gerätestatusinformationen | [Gerätezwillinge](/azure/iot-hub/iot-hub-devguide-device-twins) | Nein  | Nein  |
| Protokollunterstützung | MQTT, AMQP, HTTPS<sup>1</sup> | AMQP, HTTPS | [Kafka-Protokoll](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol) |
| Sicherheit | Geräteabhängige Identität, widerrufbare Zugriffssteuerung | Freigegebene Zugriffsrichtlinien, begrenzte Sperrung über Herausgeberrichtlinien | Authentifizierung mit SASL, austauschbare Autorisierung, Unterstützung der Integration in externe Authentifizierungsdienste |

[1] Sie können auch das [Azure IoT-Protokollgateway](/azure/iot-hub/iot-hub-protocol-gateway) als benutzerdefiniertes Gateway verwenden, um die Protokollanpassung für IoT Hub zu ermöglichen.

Weitere Informationen finden Sie unter [Vergleich zwischen Azure IoT Hub und Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubs).
