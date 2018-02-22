---
title: "Auswählen einer Technologie zur Verarbeitung von natürlicher Sprache"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: dacf7bf9cf3e9efed212f34da93c1470954965cf
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a>Auswählen einer Technologie zur Verarbeitung von natürlicher Sprache in Azure

Die Freiformtextverarbeitung wird für Dokumente durchgeführt, die Absätze mit Text enthalten. Normalerweise dient dies der Suchunterstützung, wird aber auch für andere Aufgaben zur Verarbeitung von natürlicher Sprache (Natural Language Processing, NLP) verwendet, z.B. Stimmungsanalyse, Themenerkennung, Spracherkennung, Schlüsselwortextraktion und Dokumentkategorisierung. In diesem Artikel geht es um die Auswahlmöglichkeiten in Bezug auf Technologie, die der Unterstützung von NLP-Aufgaben dient.

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a>Welche Optionen stehen Ihnen bei der Auswahl eines NLP-Diensts zur Verfügung?

In Azure verfügen die folgenden Dienste über Funktionen zur Verarbeitung natürlicher Sprache:

- [Azure HDInsight mit Spark und Spark MLlib](/azure/hdinsight/spark/apache-spark-overview)
- [Microsoft Cognitive Services](/azure/#pivot=products&panel=cognitive)

## <a name="key-selection-criteria"></a>Wichtige Auswahlkriterien

Beantworten Sie zunächst die folgenden Fragen, um die Auswahlmöglichkeiten einzuschränken:

- Möchten Sie vorgefertigte Modelle verwenden? Wenn ja, können Sie erwägen, die APIs von Microsoft Cognitive Services zu verwenden.

- Müssen Sie benutzerdefinierte Modelle anhand eines großen Korpus mit Textdaten trainieren? Wenn ja, können Sie die Verwendung von Azure HDInsight mit Spark MLlib und Spark NLP erwägen.

- Benötigen Sie spezielle NLP-Funktionen, z.B. Tokenisierung, Wortstammerkennung, Lemmatisierung und Vorkommenshäufigkeit/Inverse Dokumenthäufigkeit (Term Frequency/Inverse Document Frequency, TF/IDF)? Wenn ja, können Sie die Verwendung von Azure HDInsight mit Spark MLlib und Spark NLP erwägen.

- Benötigen Sie einfache, allgemeine NLP-Funktionen wie Entitäts- und Absichtsidentifizierung, Themenerkennung, Rechtschreibprüfung oder Standpunktanalyse? Wenn ja, können Sie erwägen, die APIs von Microsoft Cognitive Services zu verwenden.

## <a name="capability-matrix"></a>Funktionsmatrix

In den folgenden Tabellen sind die Hauptunterschiede in Bezug auf die Funktionen zusammengefasst:  

### <a name="general-capabilities"></a>Allgemeine Funktionen

| | Azure HDInsight | Microsoft Cognitive Services |
| --- | --- | --- |
| Bereitstellung von vortrainierten Modellen als Dienst | Nein  | Ja |
| REST-API | Ja | Ja |
| Programmierbarkeit | Python, Scala, Java | C#, Java, Node.js, Python, PHP, Ruby |
| Unterstützung der Verarbeitung von umfangreichen Datasets und großen Dokumenten | Ja | Nein  |

### <a name="low-level-natural-language-processing-capabilities"></a>Funktionen zur speziellen Verarbeitung von natürlicher Sprache

| | Azure HDInsight | Microsoft Cognitive Services |  
| --- | --- | --- | 
| Tokenizer | Ja (Spark NLP) | Ja (API für linguistische Analyse) |
| Wortstammerkennung | Ja (Spark NLP) | Nein  |
| Lemmatisierung | Ja (Spark NLP) | Nein  |
| Satzteilmarkierung | Ja (Spark NLP) | Ja (API für linguistische Analyse) |
| Vorkommenshäufigkeit/Inverse Dokumenthäufigkeit (Term Frequency/Inverse Document Frequency, TF/IDF) | Ja (Spark MLlib) | Nein  |
| Zeichenfolgenähnlichkeit – Berechnung der Edit-Distanz | Ja (Spark MLlib) | Nein  |
| N-Gramm-Berechnung | Ja (Spark MLlib) | Nein  |
| Stoppwortentfernung | Ja (Spark MLlib) | Nein  |

### <a name="high-level-natural-language-processing-capabilities"></a>Funktionen zur allgemeinen Verarbeitung von natürlicher Sprache

| | Azure HDInsight | Microsoft Cognitive Services |
| --- | --- | --- | 
| Entitäts-/Absichtsidentifizierung und -extraktion | Nein  | Ja (Language Understanding Intelligent Service-API, LUIS-API) |    
| Themenerkennung | Ja (Spark NLP) | Ja (Textanalyse-API) |
| Rechtschreibprüfung | Ja (Spark NLP) | Ja (Bing-Rechtschreibprüfungs-API) |
| Stimmungsanalyse | Ja (Spark NLP) | Ja (Textanalyse-API) |
| Spracherkennung | Nein  | Ja (Textanalyse-API) |
| Unterstützt neben Englisch noch mehrere andere Sprachen | Nein  | Ja (variiert je nach API) |

## <a name="see-also"></a>Weitere Informationen

[Verarbeitung natürlicher Sprache](../scenarios/natural-language-processing.md)