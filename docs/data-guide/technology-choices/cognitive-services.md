---
title: Auswählen einer Cognitive Services-Technologie
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 055769188fbd6742b94094ee18766293812849fa
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="choosing-a-microsoft-cognitive-services-technology"></a>Auswählen einer Cognitive Services-Technologie von Microsoft

Microsoft Cognitive Services sind cloudbasierte APIs, die Sie in KI-Anwendungen und -Datenflüssen verwenden können. Sie umfassen vortrainierte Modelle, die für die Nutzung in Ihrer Anwendung bereit sind und für die Sie keine Daten und kein Modelltraining bereitstellen müssen. Cognitive Services werden vom KI- und Forschungsteam von Microsoft entwickelt und umfassen die neuesten Deep Learning-Algorithmen. Sie werden über HTTP-REST-Schnittstellen genutzt. Außerdem sind SDKs für viele gängige Frameworks für die Anwendungsentwicklung verfügbar.

Cognitive Services umfassen Folgendes:

* Textanalyse
* Maschinelles Sehen
* Videoanalyse
* Spracherkennung und -generierung
* Verstehen natürlicher Sprache
* Intelligente Suche

Hauptvorteile:

* Minimaler Entwicklungsaufwand für modernste KI-Dienste
* Einfache Integration in Apps über HTTP-REST-Schnittstellen
* Integrierte Unterstützung der Nutzung von Cognitive Services in Azure Data Lake Analytics

Überlegungen:

* Nur über das Web verfügbar. Im Allgemeinen ist eine Internetverbindung erforderlich. Eine Ausnahme ist der Custom Vision Service, dessen trainiertes Modell Sie exportieren können, um Vorhersagen auf Geräten und im IoT-Edgebereich zu treffen.
* Eine umfassende Anpassung wird zwar unterstützt, aber die verfügbaren Dienste sind unter Umständen nicht für alle Predictive Analytics-Anforderungen geeignet.

## <a name="what-are-your-options-when-choosing-amongst-the-cognitive-services"></a>Welche Optionen haben Sie bei der Auswahl der Cognitive Services?
In Azure sind Dutzende von Cognitive Services verfügbar. Die aktuelle Liste ist jeweils in einem Verzeichnis enthalten, das nach dem unterstützten Funktionsbereich kategorisiert ist:
- [Bildanalyse](https://azure.microsoft.com/services/cognitive-services/directory/vision/)
- [Spracheingabe](https://azure.microsoft.com/services/cognitive-services/directory/speech/)
- [Einblicke und Wissen](https://azure.microsoft.com/services/cognitive-services/directory/know/)
- [Suchen,](https://azure.microsoft.com/services/cognitive-services/directory/search/)
- [Sprache](https://azure.microsoft.com/services/cognitive-services/directory/lang/)

## <a name="key-selection-criteria"></a>Wichtige Auswahlkriterien

Beantworten Sie zunächst die folgenden Fragen, um die Auswahlmöglichkeiten einzuschränken:

- Welche Art von Daten verarbeiten Sie? Grenzen Sie Ihre Optionen basierend auf dem Typ der Eingabedaten ein, die Sie verarbeiten. Wenn es bei Ihrer Eingabe beispielsweise um Text geht, sollten Sie einen Dienst mit dem Eingabetyp „Text“ wählen. 

- Verfügen Sie über die Daten zum Trainieren eines Modells? Wenn ja, können Sie die Verwendung der benutzerdefinierten Dienste erwägen, die Ihnen das Trainieren der zugrunde liegenden Modelle mit von Ihnen bereitgestellten Daten ermöglichen, um die Genauigkeit und Leistung zu verbessern. 

## <a name="capability-matrix"></a>Funktionsmatrix

In den folgenden Tabellen sind die Hauptunterschiede in Bezug auf die Funktionen zusammengefasst: 

### <a name="uses-prebuilt-models"></a>Verwendung von vordefinierten Modellen

|                                                   |             Eingabetyp              |                                                                                Hauptvorteil                                                                                |
|---------------------------------------------------|-------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                Textanalyse-API                 |                Text                 |                                                       Werten Sie Stimmungen und Themen aus, um zu verstehen, was sich Ihre Benutzer wünschen.                                                        |
|                API für Entitätenverknüpfung                 |                Text                 |                                               Erweitern Sie die Datenlinks Ihrer App um die Erkennung und Unterscheidung benannter Entitäten.                                               |
| Language Understanding Intelligent Service (LUIS) |                Text                 |                                                          Bringen Sie Ihren Apps bei, Befehle Ihrer Benutzer zu verstehen.                                                          |
|                 QnA Maker Service                 |                Text                 |                                             Verwandeln Sie Informationen im FAQ-Format in einfach zu findende Antworten.                                              |
|              API für linguistische Analyse              |                Text                 |                                                            Vereinfachen Sie komplexe Sprachkonzepte, und analysieren Sie Texte.                                                             |
|           Knowledge Exploration Service           |                Text                 |                                          Ermöglichen Sie die interaktive Suche in strukturierten Daten per Eingabe in natürlicher Sprache.                                          |
|              Websprachmodell-API               |                Text                 |                                                         Nutzen Sie Vorhersagesprachmodelle, die mit Daten aus dem gesamten Web trainiert wurden.                                                         |
|              Academic Knowledge-API               |                Text                 |                                        Greifen Sie auf die umfangreichen akademischen Inhalte von Microsoft Academic Graph mit Auffüllung per Bing zu.                                         |
|               Bing-Vorschlagssuche-API                |                Text                 |                                                        Erweitern Sie Ihre App um intelligente Optionen für Vorschläge bei Suchvorgängen.                                                        |
|               Bing-Rechtschreibprüfungs-API                |                Text                 |                                                             Ermitteln und korrigieren Sie Rechtschreibfehler in Ihrer App.                                                             |
|                Textübersetzungs-API                |                Text                 |                                                                           Maschinelle Übersetzung                                                                            |
|                Empfehlungs-API                |                Text                 |                                                             Sagen Sie vorher, für welche Artikel sich Ihre Kunden interessieren, und empfehlen Sie diese.                                                              |
|              Bing-Entitätssuche-API               |       Text (Websuchabfrage)       |                                                           Identifizieren und erweitern Sie Entitätsinformationen aus dem Web.                                                           |
|               Bing-Bildersuche-API               |       Text (Websuchabfrage)       |                                                                            Suchen Sie nach Bildern.                                                                             |
|               Bing-News-Suche-API                |       Text (Websuchabfrage)       |                                                                             Suchen Sie nach News.                                                                              |
|               Bing-Videosuche-API               |       Text (Websuchabfrage)       |                                                                            Suchen Sie nach Videos.                                                                             |
|                Bing-Websuche-API                |       Text (Websuchabfrage)       |                                                        Erhalten Sie umfassende Suchergebnisse aus Milliarden von Webdokumenten.                                                        |
|                  Bing-Spracheingabe-API                  |           Text oder Sprache            |                                                                  Konvertieren Sie Sprache in Text und wieder zurück.                                                                   |
|              Sprechererkennungs-API              |               Spracheingabe                |                                                       Identifizieren und authentifizieren Sie Sprecher anhand ihrer Stimme.                                                        |
|               Sprachübersetzungs-API               |               Spracheingabe                |                                                                   Führen Sie Sprachübersetzungen in Echtzeit durch.                                                                   |
|                Maschinelles Sehen-API                |    Bilder (oder Frames aus Videos)    | Filtern Sie verwertbare Informationen aus Bildern heraus, nutzen Sie die automatische Erstellung von Fotobeschreibungen, die Ableitung von Tags und die Erkennung von berühmten Personen und die Textextraktion, und erstellen Sie präzise Miniaturansichten. |
|                 Content Moderator                 |        Text, Bilder oder Video        |                                                               Automatisierte Bild-, Text- und Videomoderation                                                                |
|                    Emotionen-API                    | Bilder (Fotos mit Menschen) |                                                              Identifizieren Sie den Emotionsbereich von Menschen.                                                               |
|                     Gesichtserkennungs-API                      | Bilder (Fotos mit Menschen) |                                                       Nutzen Sie die Erkennung, Analyse, Organisation und Markierung von Gesichtern auf Fotos.                                                       |
|                   Video-Indexer                   |                Video                |                        Führen Sie Videoauswertungen in Bezug auf Stimmung, Sprachtranskription, Sprachübersetzung, Gesichter- und Emotionserkennung und Extraktion von Schlüsselwörtern durch.                         |

### <a name="trained-with-custom-data-you-provide"></a>Trainiert mit von Ihnen bereitgestellten benutzerdefinierten Daten

| | Eingabetyp | Hauptvorteil |
| --- | --- | --- |
| Custom Vision Service | Bilder (oder Frames aus Videos) | Passen Sie Ihre eigenen Modelle für maschinelles Sehen an. |
| Benutzerdefinierter Spracherkennungsdienst | Spracheingabe | Überwinden Sie die Grenzen der Spracherkennung, z.B. Sprachstil, Hintergrundgeräusche und Vokabular. | 
| Custom Decision Service | Webinhalt (z.B. RSS-Feed) | Verwenden Sie Machine Learning, um automatisch die richtigen Inhalte für Ihre Homepage auswählen zu lassen. |
| API für die benutzerdefinierte Bing-Suche | Text (Websuchabfrage) | Kommerziell einsetzbares Suchtool |

