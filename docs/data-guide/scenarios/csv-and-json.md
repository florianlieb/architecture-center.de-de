---
title: Verarbeiten von CSV- und JSON-Dateien
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 02e684d562cfe555f9e3596ad0a2f1a00d05c7a7
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/03/2018
---
# <a name="working-with-csv-and-json-files-for-data-solutions"></a>Verwenden von CSV- und JSON-Dateien für Datenlösungen

CSV und JSON sind wahrscheinlich die am häufigsten verwendeten Formate für die Erfassung, den Austausch und die Speicherung unstrukturierter oder teilweise strukturierter Daten. 

## <a name="about-csv-format"></a>Informationen zum CSV-Format

Dateien vom Typ CSV (Comma-Separated Values, durch Trennzeichen getrennte Werte) werden häufig verwendet, um tabellarische Daten zwischen Systemen im Nur-Text-Format auszutauschen. Sie enthalten üblicherweise eine Kopfzeile mit den Spaltennamen für die Daten, werden ansonsten aber als teilweise strukturierte Dateien betrachtet. Das liegt daran, dass CSV-Dateien hierarchische oder relationale Daten nicht auf natürliche Weise darstellen können. Für Datenbeziehungen werden üblicherweise mehrere CSV-Dateien verwendet. Dabei werden Fremdschlüssel in Spalten einzelner oder mehrerer Dateien gespeichert, die Beziehungen zwischen diesen Dateien werden jedoch nicht durch das Format selbst ausgedrückt. Dateien im CSV-Format können neben Kommas auch andere Trennzeichen wie etwa Tabulatoren oder Leerzeichen verwenden.

Ungeachtet dieser Einschränkungen werden CSV-Dateien gern für den Datenaustausch verwendet, da sie von einer Vielzahl geschäftlicher, verbraucherorientierter und wissenschaftlicher Anwendungen unterstützt werden. So können CSV-Dateien beispielsweise von Datenbank- und Tabellenkalkulationsprogrammen importiert und exportiert werden. Auch die meisten Batch- und Datenstrom-Verarbeitungsmodule wie Spark und Hadoop unterstützen die Serialisierung und Deserialisierung von Dateien im CSV-Format nativ und ermöglichen die Anwendung eines Schemas beim Lesen. Die Verwendung der Daten wird durch die zur Verfügung stehenden Datenabfrageoptionen und die Möglichkeit, die Informationen zur schnelleren Verarbeitung in einem effizienteren Datenformat zu speichern, erleichtert.

## <a name="about-json-format"></a>Informationen zum JSON-Format

JSON-Daten (JavaScript Object Notation) werden als Schlüssel-Wert-Paare in einem teilweise strukturierten Format dargestellt. JSON wird häufig mit XML verglichen, da beide Daten in einem hierarchischen Format speichern können, in dem untergeordnete Daten zusammen mit übergeordneten Daten dargestellt werden. Beide sind selbstbeschreibend und für Menschen lesbar, JSON-Dokumente sind jedoch in der Regel deutlich kleiner und werden daher besonders gerne für den Onlinedatenaustausch verwendet – insbesondere in Verbindung mit REST-basierten Webdiensten. 

Dateien im JSON-Format haben gegenüber CSV-Dateien mehrere Vorteile:

* Bei JSON bleiben hierarchische Strukturen erhalten. Dies vereinfacht die Speicherung verwandter Daten in einem einzelnen Dokument sowie die Darstellung komplexer Beziehungen.
* Die meisten Programmiersprachen unterstützen die Deserialisierung von JSON in Objekte nativ oder stellen einfache JSON-Serialisierungsbibliotheken bereit.
* Dank der Unterstützung von Objektlisten lassen sich mit JSON chaotische Umwandlungen von Listen in ein relationales Datenmodell vermeiden.
* Das JSON-Dateiformat wird häufig für NoSQL-Datenbanken wie MongoDB, Couchbase und Azure Cosmos DB verwendet.

Da ein Großteil der übertragenen Daten bereits im JSON-Format vorliegt, unterstützen die meisten webbasierten Programmiersprachen die Verwendung von JSON nativ oder über externe Bibliotheken für die Serialisierung und Deserialisierung von JSON-Daten. Diese universelle Unterstützung des Formats hat zur Verwendung von JSON in logischen Formaten geführt – In Form von Datenstrukturdarstellung, Austauschformaten für heiße Daten und Datenspeicherung für kalte Daten.

Die JSON-Serialisierung und -Deserialisierung wird von vielen Batch- und Datenstrom-Verarbeitungsmodulen nativ unterstützt. Die Daten in JSON-Dokumenten werden zwar möglicherweise letztlich in leistungsoptimierteren Formaten wie Parquet oder Avro gespeichert, sie fungieren aber als alleingültige Rohdatenquelle, was für die erneute Verarbeitung der Daten nach Bedarf entscheidend ist.

## <a name="when-to-use-csv-or-json-formats"></a>Verwendung des CSV- oder des JSON-Formats

CSV-Dateien werden eher zum Exportieren und Importieren von Daten oder zur Verarbeitung für Analysen und Machine Learning verwendet. Dateien im JSON-Format bieten die gleichen Vorteile, werden aber eher in Lösungen für den Austausch heißer Daten verwendet. JSON-Dokumente werden häufig von webbasierten und mobilen Geräten, die Onlinetransaktionen ausführen, von IoT-Geräten (Internet of Things, Internet der Dinge) für die uni- oder bidirektionale Kommunikation oder von Clientanwendungen gesendet, die mit SaaS- und PaaS-Diensten oder serverlosen Architekturen kommunizieren. 

Sowohl das CSV- als auch das JSON-Dateiformat vereinfacht den Austausch von Daten zwischen unterschiedlichen Systemen oder Geräten. Ihre teilweise strukturierten Formate ermöglichen die flexible Übertragung nahezu jeglicher Art von Daten, und die universelle Unterstützung der Formate erleichtert ihre Verwendung. Wenn die verarbeiteten Daten zur effizienteren Abfrage in binären Formaten gespeichert werden, können beide als unformatierte alleingültige Quelle verwendet werden. 

## <a name="working-with-csv-and-json-data-in-azure"></a>Verwenden von CSV- und JSON-Daten in Azure

Azure bietet mehrere Lösungen für die Verwendung von CSV- und JSON-Dateien, um verschiedenste Anforderungen zu erfüllen. Dateien dieser Art befinden sich in erster Linie in Azure Storage oder in Azure Data Lake Store. Die meisten Azure-Dienste, die diese und andere textbasierte Dateien verwenden, sind in beide Objektspeicherdienste integriert. In bestimmten Fällen sollen die Daten jedoch möglicherweise direkt in Azure SQL oder in einen anderen Datenspeicher importiert werden. SQL Server unterstützt die Speicherung und Verwendung von JSON-Dokumenten nativ, was das [Importieren und Verarbeiten dieser Dateitypen](/sql/relational-databases/json/import-json-documents-into-sql-server) vereinfacht. Mit einem Hilfsprogramm wie SQL Bulk Import können Sie ganz einfach [CSV-Dateien importieren](/sql/relational-databases/json/import-json-documents-into-sql-server).

Abhängig vom jeweiligen Szenario können Sie für die Daten eine [Batchverarbeitung](../big-data/batch-processing.md) oder eine [Echtzeitverarbeitung](../big-data/real-time-processing.md) durchführen.

## <a name="challenges"></a>Herausforderungen

Bei der Verwendung dieser Formate müssen ein paar Punkte berücksichtigt werden:

* Wenn das Datenmodell ganz ohne Einschränkungen verwendet wird, sind CSV- und JSON-Dateien anfällig für Datenbeschädigungen, sprich: Schlechte Ausgangsdaten führen zu schlechten Ergebnissen. So verfügt etwa keine der Dateien über ein Konzept für ein Datums-/Uhrzeitobjekt, wodurch in einem Datumsfeld beispielsweise „ABC123“ angegeben werden kann.

* Die Verwendung von CSV- und JSON-Dateien als Cold Storage-Lösung führt in Verbindung mit Big Data zu keinen guten Skalierungsergebnissen. In den meisten Fällen können sie nicht in Partitionen für die parallele Verarbeitung aufgeteilt werden, und sie lassen sich auch nicht so gut komprimieren wie Binärformate. Dies führt häufig dazu, dass die Daten in leseoptimierten Formaten wie Parquet und ORC (Optimized Row Columnar) verarbeitet und gespeichert werden, die auch Indizes und Inlinestatistiken zu den enthaltenen Daten bereitstellen.

* Auf die teilweise strukturierten Daten muss unter Umständen ein Schema angewendet werden, um sie leichter abfragen und analysieren zu können. Hierzu müssen die Daten üblicherweise in einer anderen Form gespeichert werden, die die Datenspeicherungsanforderungen Ihrer Umgebung erfüllt (beispielsweise in einer Datenbank).

