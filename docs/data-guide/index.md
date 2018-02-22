---
title: Azure-Datenarchitekturleitfaden
description: 
author: zoinerTejada
ms:date: 02/12/2018
layout: LandingPage
ms.openlocfilehash: 848601f27faf56ea069852d8983e4d10fbad9d77
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="azure-data-architecture-guide"></a>Azure-Datenarchitekturleitfaden

In diesem Leitfaden wird ein strukturierter Ansatz für den Entwurf von datenorientierten Lösungen in Microsoft Azure vorgestellt. Er basiert auf bewährten Methoden, die aus Kundeninteraktionen abgeleitet wurden.

## <a name="introduction"></a>Einführung

Die Cloud verändert die Art und Weise, wie Anwendungen entwickelt werden, und auch die Verarbeitung und Speicherung von Daten. Anstelle einer einzelnen allgemeinen Datenbank, die alle Daten einer Lösung enthält, nutzen _mehrsprachige Persistenzlösungen_ mehrere spezielle Datenspeicher, die jeweils zur Bereitstellung bestimmter Funktionen optimiert sind. Dadurch ändert sich die Perspektive auf die Daten in der Lösung. Mehrere Ebenen von Geschäftslogik, die aus und in eine(r) einzelne(n) Datenschicht lesen und schreiben, gehören der Vergangenheit an. Stattdessen werden Lösungen rund um eine *Datenpipeline* konzipiert, die beschreibt, wie Daten durch eine Lösung fließen, wo sie verarbeitet, wo sie gespeichert und wie sie von der nächsten Komponente in der Pipeline genutzt werden. 

## <a name="how-this-guide-is-structured"></a>Aufbau dieses Leitfadens

Dieser Leitfaden basiert auf einem Schlüsselkonzept: der Unterscheidung zwischen *relationalen* Daten und *nicht relationalen* Daten. 

![](./images/guide-steps.svg)

Relationale Daten werden in der Regel in einem herkömmlichen RDBMS oder einem Data Warehouse gespeichert. Sie verfügen über ein vordefiniertes Schema (Schema-on-Write, SOW) mit einer Reihe von Einschränkungen zur Aufrechterhaltung der referentiellen Integrität. Die meisten relationalen Datenbanken verwenden zum Abfragen die strukturierte Abfragesprache (Structured Query Language, SQL). Zu den Lösungen, die relationale Datenbanken nutzen, zählen die Onlinetransaktionsverarbeitung (Online Transaction Processing, OLTP) und die analytische Onlineverarbeitung (Online Analytical Processing, OLAP).

Nicht relationale Daten sind alle Daten, die nicht das in herkömmlichen RDBMS-Systemen übliche [relationale Modell](https://en.wikipedia.org/wiki/Relational_model) verwenden. Dazu können Schlüssel-Wert-Daten, JSON-Daten, Diagrammdaten, Zeitreihendaten und andere Datentypen gehören. Der Begriff *NoSQL* bezieht sich auf Datenbanken, die für die Speicherung verschiedener Typen nicht relationaler Daten ausgelegt sind. Der Begriff ist jedoch nicht ganz zutreffend, da viele nicht relationale Datenspeicher SQL-kompatible Abfragen unterstützen. Nicht relationale Daten und NoSQL-Datenbanken sind Themen, die bei Diskussionen über *Big Data*-Lösungen häufig zur Sprache kommen. Eine Big Data-Architektur ist für die Erfassung, Verarbeitung und Analyse von Daten konzipiert, die für herkömmliche Datenbanksysteme zu groß oder zu komplex sind. 

Der Architekturleitfaden enthält die folgenden Abschnitte zu diesen zwei Hauptkategorien:

- **Konzepte.** Einführende Artikel, in denen die wichtigsten Konzepte vorgestellt werden, mit denen Sie bei der Arbeit mit dieser Art von Daten vertraut sein müssen.
- **Szenarien.** Ein repräsentativer Satz von Datenszenarien, einschließlich einer Erläuterung der relevanten Azure-Dienste und der geeigneten Architektur für das Szenario.
- **Auswahl der Technologie.** Ein ausführlicher Vergleich verschiedener Datentechnologien, die in Azure verfügbar sind, einschließlich Open Source-Optionen. In jeder Kategorie finden Sie eine Beschreibung der wichtigsten Auswahlkriterien und eine Funktionsmatrix, die Ihnen die Auswahl der passenden Technologie für Ihr Szenario erleichtert.

In diesem Leitfaden geht es nicht um Data Science oder Datenbanktheorie. Zu diesen Themen wurden bereits ganze Bücher verfasst. Stattdessen möchten wir Ihnen dabei helfen, die passende Datenarchitektur oder Datenpipeline für Ihr Szenario zu finden und anschließend die Azure-Dienste und -Technologien auszuwählen, die am besten für Ihre Anforderungen geeignet sind. Wenn Sie bereits eine bestimmte Architektur geplant haben, können Sie direkt mit dem Abschnitt zur Auswahl der Technologie fortfahren.

## <a name="traditional-rdbms"></a>Herkömmliches RDBMS

### <a name="concepts"></a>Konzepte

- [Relationale Daten](./concepts/relational-data.md) 
- [Transaktionsdaten](./concepts/transactional-data.md) 
- [Semantische Modellierung](./concepts/semantic-modeling.md) 

### <a name="scenarios"></a>Szenarien

- [Analytische Onlineverarbeitung (OLAP)](./scenarios/online-analytical-processing.md)
- [Onlinetransaktionsverarbeitung (OLTP)](./scenarios/online-transaction-processing.md) 
- [Data Warehousing und Data Marts](./scenarios/data-warehousing.md)
- [ETL](./scenarios/etl.md) 

## <a name="big-data-and-nosql"></a>Big Data und NoSQL

### <a name="concepts"></a>Konzepte

- [Nicht relationale Datenspeicher](./concepts/non-relational-data.md)
- [Arbeiten mit CSV- und JSON-Dateien](./concepts/csv-and-json.md)
- [Big Data-Architekturen](./concepts/big-data.md)
- [Erweiterte Analyse](./concepts/advanced-analytics.md) 
- [Bedarfsorientiertes Machine Learning](./concepts/machine-learning-at-scale.md)

### <a name="scenarios"></a>Szenarien

- [Batchverarbeitung](./scenarios/batch-processing.md)
- [Verarbeitung in Echtzeit](./scenarios/real-time-processing.md)
- [Freitextsuche](./scenarios/search.md)
- [Interaktive Datenuntersuchung](./scenarios/interactive-data-exploration.md)
- [Verarbeitung natürlicher Sprache](./scenarios/natural-language-processing.md)
- [Zeitreihenlösungen](./scenarios/time-series.md)

## <a name="cross-cutting-concerns"></a>Querschnittsthemen

- [Datenübertragung](./scenarios/data-transfer.md) 
- [Erweitern lokaler Datenlösungen auf die Cloud](./scenarios/hybrid-on-premises-and-cloud.md) 
- [Schützen von Datenlösungen](./scenarios/securing-data-solutions.md) 
