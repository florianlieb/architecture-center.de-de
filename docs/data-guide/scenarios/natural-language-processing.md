---
title: Verarbeitung natürlicher Sprache
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 0afd8ac9a8a2e56f79ade0b2e10328630866c03c
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/03/2018
---
# <a name="natural-language-processing"></a>Verarbeitung natürlicher Sprache

Die Verarbeitung natürlicher Sprache (Natural Language Processing, NLP) wird für Aufgaben wie die Standpunktanalyse, Textgegenstandserkennung, Sprachenerkennung, Schlüsselbegriffserkennung und Dokumentkategorisierung verwendet.

![](./images/nlp-pipeline.png)

## <a name="when-to-use-this-solution"></a>Verwendung dieser Lösung

NLP kann zum Klassifizieren von Dokumenten verwendet werden, beispielsweise zur Kennzeichnung von Dokumenten als „vertraulich“ oder „Spam“. Die Ausgabe der NLP kann zur anschließenden Verarbeitung oder Suche verwendet werden. Eine weitere Verwendungsmöglichkeit der NLP ist die Zusammenfassung von Text durch Identifizieren der im Dokument vorhandenen Entitäten. Diese Entitäten können auch verwendet werden, um Dokumente mit Schlüsselwörtern zu markieren, sodass sie anhand des Inhalts gesucht und abgerufen werden können. Entitäten können zu Themen kombiniert und mit Zusammenfassungen versehen werden, in denen die wichtigen Themen in jedem Dokument beschrieben werden. Anhand der erkannten Themen können die Dokumente zur Navigation kategorisiert oder verwandte Dokumente für ein ausgewähltes Thema aufgelistet werden. Ein weiterer Verwendungszweck für die NLP ist die Standpunktbewertung von Texten, um die positive oder negative Stimmung eines Dokuments zu bewerten. Bei diesen Ansätzen werden viele Techniken der Verarbeitung natürlicher Sprache verwendet, beispielsweise: 

- **Tokenizer**. Aufteilung des Textes in Wörter oder Ausdrücke.
- **Wortstammerkennung und Lemmatisierung**. Normalisierung von Wörtern, sodass unterschiedliche Formen dem kanonischen Wort mit der gleichen Bedeutung entsprechen. Beispielweise werden „Ausführung“ und „ausgeführt“ dem Wort „ausführen“ zugeordnet. 
- **Entitätsextraktion**. Identifizieren von Themen im Text.
- **Wortarterkennung**. Erkennung von Text als Verb, Nomen, Partizip, Verbalphrase usw.
- **Erkennung von Satzgrenzen**. Erkennung von vollständigen Sätzen innerhalb von Textabschnitten.

Bei der Verwendung der NLP zum Extrahieren von Informationen und Einblicken aus Freitext sind meist die in Objektspeicher wie Azure Storage oder Azure Data Lake Store gespeicherten unformatierten Dokumente der Ausgangspunkt. 

## <a name="challenges"></a>Herausforderungen

- Die Verarbeitung einer Sammlung von Freitextdokumenten ist in der Regel rechen-, ressourcen- und zeitintensiv.
- Ohne ein standardisiertes Dokumentformat kann es sehr schwierig sein, durchweg genaue Ergebnisse zu erzielen, wenn die Freitextverarbeitung zum Extrahieren bestimmter Fakten aus einem Dokument verwendet wird. Wenn Sie beispielsweise eine Textdarstellung einer Rechnung haben, kann es schwierig sein, einen Prozess zu erstellen, der die Rechnungsnummer und das Rechnungsdatum für Rechnungen von einer beliebigen Anzahl von Lieferanten korrekt extrahiert.

## <a name="architecture"></a>Architecture

In einer NLP-Lösung wird die Freitextverarbeitung für Dokumente ausgeführt, die Textabschnitte enthalten. Die allgemeine Architektur kann auf [Batchverarbeitung](../big-data/batch-processing.md) oder [Echtzeit-Datenstromverarbeitung](../big-data/real-time-processing.md) beruhen.

Die tatsächliche Verarbeitung variiert je nach gewünschtem Ergebnis. Im Hinblick auf die Pipeline kann die NLP jedoch in einem Batch oder in Echtzeit angewendet werden kann. Beispielsweise kann die Standpunktanalyse für Textblöcke verwendet werden, um eine Stimmungspunktzahl zu erhalten. Die Verarbeitung kann in diesem Fall durch Ausführen eines Batchprozesses für Daten im Speicher oder in Echtzeit anhand von kleineren, durch einen Messagingdienst übermittelten Datenblöcken erfolgen.

## <a name="technology-choices"></a>Auswahl der Technologie

- [Verarbeitung natürlicher Sprache](../technology-choices/natural-language-processing.md)