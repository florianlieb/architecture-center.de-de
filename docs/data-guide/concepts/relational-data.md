---
title: Relationale Daten
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 55c7354b8cec13318bbf3fda1c648cde17288854
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/31/2018
---
# <a name="relational-data"></a>Relationale Daten

Bei relationalen Daten handelt es sich um Daten, die mit dem relationalen Modell modelliert werden. Bei diesem Modell werden Daten als Tupel ausgedrückt. Ein *Tupel* ist ein Satz mit Attribut-Wert-Paaren. Beispiel für ein Tupel: (itemid = 5, orderid = 1, item = "Chair", amount = 200.00). Eine Tupelmenge, bei der die Tupel die Attribute gemeinsam nutzen, wird als *Relation* bezeichnet. 

Relationen werden natürlich als Tabellen dargestellt, wobei jedes Tupel in der Tabelle als Zeile verfügbar gemacht wird. Für Zeilen gilt – im Gegensatz zu Tupeln – aber eine explizite Reihenfolge. Im Datenbankschema sind die Spalten (Überschriften) jeder Tabelle definiert. Jede Spalte wird übergreifend für alle Zeilen der Tabelle mit einem Namen und einem Datentyp für alle Werte definiert, die in dieser Spalte gespeichert sind.

![Beispiel mit Daten bei Nutzung einer relationalen Datenbank](./images/example-relational.png)

Ein Datenspeicher, in dem Daten nach dem relationalen Modell organisiert sind, wird als relationale Datenbank bezeichnet. Die Zeilen einer Tabelle werden mit Primärschlüsseln eindeutig identifiziert. Fremdschlüsselfelder werden in einer Tabelle verwendet, um auf eine Zeile in einer anderen Tabelle zu verweisen, indem auf den Primärschlüssel der anderen Tabelle verwiesen wird. Fremdschlüssel werden verwendet, um die referentielle Integrität zu wahren und sicherzustellen, dass die referenzierten Zeilen nicht geändert oder gelöscht werden, während die verweisende Zeile davon abhängig ist. 

![Beispiel mit Daten bei Nutzung einer relationalen Datenbank](./images/example-relational2.png)

Relationale Datenbanken unterstützen verschiedene Arten von Einschränkungen, die zur Sicherstellung der Datenintegrität beitragen:

- Mit eindeutigen Einschränkungen wird erreicht, dass alle Werte einer Spalte eindeutig sind. 

- Mit Fremdschlüsseleinschränkungen wird eine Verknüpfung zwischen den Daten in zwei Tabellen erzwungen. Ein Fremdschlüssel verweist auf den Primärschlüssel oder einen anderen eindeutigen Schlüssel aus der anderen Tabelle. Eine Fremdschlüsseleinschränkung erzwingt die referentielle Integrität, und Änderungen, die zu ungültigen Fremdschlüsselwerten führen, werden nicht zugelassen.

- CHECK-Einschränkungen, die auch als Integritätseinschränkungen bezeichnet werden, sorgen für eine Begrenzung der Werte, die in einer Spalte oder bezogen auf Werte in anderen Spalten derselben Zeile gespeichert werden können. 

Für die meisten relationalen Datenbanken wird die Structured Query Language (SQL) verwendet, die einen deklarativen Ansatz für das Durchführen von Abfragen ermöglicht. In der Abfrage wird das gewünschte Ergebnis beschrieben, aber nicht die Schritte zum Ausführen der Abfrage. Das Modul wählt dann den besten Weg zum Durchführen der Abfrage. Dies unterscheidet sich von einem prozeduralen Ansatz, bei dem die Verarbeitungsschritte über das Abfrageprogramm explizit angegeben werden. In relationalen Datenbanken können Routinen mit ausführbarem Code aber in Form von gespeicherten Prozeduren und Funktionen gespeichert werden, sodass eine Mischung von deklarativen und prozeduralen Ansätzen möglich ist.

Zur Verbesserung der Abfrageleistung werden in relationalen Datenbanken *Indizes* verwendet. Mit primären Indizes, die vom Primärschlüssel genutzt werden, wird die Reihenfolge der Daten auf dem Datenträger definiert. Bei sekundären Indizes ist eine andere Kombination von Feldern möglich, sodass die gewünschten Zeilen effizient abgefragt werden können, ohne dass die gesamten Daten auf dem Datenträger neu sortiert werden müssen.

Da mit relationalen Datenbanken die referentielle Integrität erzwungen wird, kann das Skalieren einer relationalen Datenbank schwierig sein. Dies liegt daran, dass von jedem Abfrage- oder Einfügevorgang eine große Zahl von Tabellen betroffen sein kann. Sie können eine relationale Datenbank horizontal hochskalieren, indem Sie für die Daten das *Sharding* durchführen. Hierfür ist aber ein sorgfältiger Schemaentwurf erforderlich. Weitere Informationen finden Sie unter [Sharding-Muster](../../patterns/sharding.md).

Wenn Daten nicht relational sind oder über Anforderungen verfügen, die für eine relationale Datenbank nicht geeignet sind, können Sie einen [nicht relationalen oder NoSQL](./non-relational-data.md)-Datenspeicher verwenden.
