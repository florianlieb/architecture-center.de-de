---
title: Semantische Modellierung
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: e989a7a5a58e7d05e261931005069bb12bd79186
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="semantic-modeling"></a>Semantische Modellierung

Ein semantisches Datenmodell ist ein konzeptionelles Modell, mit dem die Bedeutung der darin enthaltenen Datenelemente beschrieben wird. In Organisationen werden häufig eigene Begriffe verwendet, bei denen es sich um Synonyme handeln kann oder die für denselben Begriff unterschiedliche Bedeutungen haben. Beispiel: In einer Bestandsdatenbank wird ein Ausrüstungsteil anhand einer Asset-ID und einer Seriennummer nachverfolgt, während in einer Vertriebsdatenbank die Seriennummer als Asset-ID bezeichnet wird. Es gibt keine einfache Möglichkeit, diese Werte miteinander in Beziehung zu setzen, ohne dass ein Modell verwendet wird, mit dem die Beziehung beschrieben wird. 

Die semantische Modellierung ermöglicht ein bestimmtes Maß an Abstraktion für das Datenbankschema, damit Benutzer nicht über die zugrunde liegenden Datenstrukturen informiert sein müssen. Dies erleichtert Endbenutzern das Abfragen von Daten ohne Durchführung von Aggregierungs- und Verknüpfungsvorgängen für das zugrunde liegende Schema. Außerdem erhalten Spalten normalerweise benutzerfreundlichere Namen, damit der Kontext und die Bedeutung der Daten besser erkennbar ist.

Die semantische Modellierung wird hauptsächlich für Szenarien mit hohem Leseaufwand verwendet, z.B. Analytics und Business Intelligence (OLAP), und nicht für die transaktionale Datenverarbeitung (OLTP) mit hohem Schreibaufwand. Dies liegt vor allem an der Art einer typischen semantischen Ebene:

- Das Aggregierungsverhalten ist so festgelegt, dass es von Tools für die Berichterstellung richtig angezeigt werden.
- Die Geschäftslogik und die Berechnungen sind definiert.
- Zeitabhängige Berechnungen sind enthalten.
- Daten werden häufig aus mehreren Quellen integriert. 

Aus diesen Gründen wird die semantische Ebene meist über einem Data Warehouse angeordnet.

![Beispieldiagramm mit einer semantischen Ebene zwischen einem Data Warehouse und einem Berichterstellungstool](./images/semantic-modeling.png)

Es gibt zwei Hauptarten von Semantikmodellen:

* **Tabellarisch**: Hierfür werden Konstrukte der relationalen Modellierung verwendet (Modell, Tabellen, Spalten). Intern werden Metadaten von Konstrukten der OLAP-Modellierung geerbt (Cubes, Dimensionen, Measures). Für den Code und Skripts werden OLAP-Metadaten genutzt.
* **Mehrdimensional**: Hierfür werden herkömmliche Konstrukte der OLAP-Modellierung verwendet (Cubes, Dimensionen, Measures).

In Frage kommender Azure-Dienst:
- [Azure Analysis Services](https://azure.microsoft.com/services/analysis-services/)

## <a name="example-use-case"></a>Beispiel eines Anwendungsfalls

In einer Organisation werden Daten in einer großen Datenbank gespeichert. Diese Daten sollen für geschäftliche Benutzer und Kunden verfügbar gemacht werden, damit diese eigene Berichte erstellen und Analysen durchführen können. Eine Option besteht darin, diesen Benutzern direkten Zugriff auf die Datenbank zu gewähren. Dieser Ansatz ist aber mit mehreren Nachteilen verbunden, z.B. dem Aufwand für die Verwaltung der Sicherheit und Steuerung des Zugriffs. Außerdem kann das Design der Datenbank, z.B. Namen von Tabellen und Spalten, für Benutzer schwer verständlich sein. Benutzer müssen wissen, welche Tabellen abgefragt werden sollen, wie diese Tabellen verknüpft werden müssen und wie es sich mit der weiteren Geschäftslogik verhält, die angewendet werden muss, um die richtigen Ergebnisse zu erhalten. Benutzer müssen sich zudem mit einer Abfragesprache, z.B. SQL, auskennen, um beginnen zu können. Normalerweise führt dies dazu, dass mehrere Benutzer die gleichen Metriken melden, aber mit unterschiedlichen Ergebnissen.

Eine andere Option besteht darin, alle Informationen, die Benutzer benötigen, in einem Semantikmodell zusammenzufassen. Das Semantikmodell kann von Benutzern mit einem Berichterstellungstool ihrer Wahl einfacher abgefragt werden. Die vom Semantikmodell bereitgestellten Daten werden per Pullvorgang aus einem Data Warehouse abgerufen, um sicherzustellen, dass alle Benutzer die einzige und richtige Version angezeigt bekommen. Darüber hinaus werden über das Semantikmodell auch Anzeigenamen für Tabellen und Spalten, Beziehungen zwischen Tabellen, Beschreibungen, Berechnungen und die Sicherheit auf Zeilenebene bereitgestellt.

## <a name="typical-traits-of-semantic-modeling"></a>Typische Merkmale der semantischen Modellierung

Für die semantische Modellierung und analytische Verarbeitung gelten in der Regel die folgenden Merkmale:

| Anforderung | Beschreibung |
| --- | --- |
| Normalisierung | Stark normalisiert |
| Schema | Schema für Schreibvorgänge, strikte Erzwingung|
| Nutzung von Transaktionen | Nein |
| Sperrstrategie | Keine |
| Aktualisierbar | Nein (normalerweise Neuberechnung des Cubes erforderlich) |
| Erweiterbar | Nein (normalerweise Neuberechnung des Cubes erforderlich) |
| Workload | Hoher Leseaufwand, schreibgeschützt |
| Indizierung | Mehrdimensionale Indizierung |
| Bezugsgröße | Kleine bis mittlere Größe |
| Modell | Mehrdimensional |
| Datenform:| Cube oder Stern/Schneeflockenschema |
| Abfrageflexibilität | Sehr flexibel |
| Skalierung: | Groß (Dutzende bis mehrere Hundert GB) |

## <a name="see-also"></a>Weitere Informationen

- [Data Warehousing](../scenarios/data-warehousing.md)
- [Analytische Onlineverarbeitung (Online Analytical Processing, OLAP)](../scenarios/online-analytical-processing.md)