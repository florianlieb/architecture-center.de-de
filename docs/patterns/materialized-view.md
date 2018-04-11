---
title: Materialisierte Sicht
description: Generieren Sie vorausgefüllte Sichten für die Daten in einem oder mehreren Datenspeichern, wenn die Daten für erforderliche Abfrageoperationen nicht ideal formatiert sind.
keywords: Entwurfsmuster
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: 992abcb57204c65a7ca9e9e2525d3ea7339c4a2c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="materialized-view-pattern"></a>Muster für materialisierte Sichten

[!INCLUDE [header](../_includes/header.md)]

Generieren Sie vorausgefüllte Sichten für die Daten in einem oder mehreren Datenspeichern, wenn die Daten für erforderliche Abfrageoperationen nicht ideal formatiert sind. Dies kann für die Unterstützung effizienter Abfragen und Datenextraktionen hilfreich sein und die Leistung verbessern.

## <a name="context-and-problem"></a>Kontext und Problem

Beim Speichern von Daten sind Entwickler und Datenadministratoren häufig primär damit beschäftigt, wie die Daten gespeichert werden, und nicht damit, wie sie gelesen werden. Das ausgewählte Speicherformat hängt in der Regel eng mit dem Format der Daten, den Anforderungen für den Umgang mit der Größe und der Integrität der Daten und der Art des verwendeten Speichers zusammen. Beispielsweise werden bei der Verwendung des NoSQL-Dokumentspeichers die Daten häufig als eine Reihe von Aggregaten dargestellt, die jeweils alle Informationen für die Entität enthalten.

Allerdings kann dies negative Auswirkungen auf Abfragen haben. Wenn für eine Abfrage nur eine Teilmenge der Daten einiger Entitäten benötigt wird, z.B. eine Zusammenfassung der Bestellungen von mehreren Kunden ohne sämtliche Bestelldetails, müssen alle Daten für die relevanten Entitäten extrahiert werden, um die erforderlichen Informationen abzurufen.

## <a name="solution"></a>Lösung

Eine gängige Lösung für die Unterstützung effizienter Abfragen ist, im Voraus eine Sicht zu generieren, mit der die Daten in einem Format materialisiert werden, das für das erforderliche Resultset geeignet ist. Mit dem Muster für materialisierte Sichten wird das Generieren vorausgefüllter Sichten von Daten in Umgebungen beschrieben, in denen die Quelldaten nicht in einem geeigneten Format für Abfragen vorliegen, in denen das Generieren einer Abfrage schwierig ist oder in denen die Abfrageleistung aufgrund der Art der Daten oder des Datenspeichers schlecht ist.

Mit diesen materialisierten Sichten, die nur die für eine Abfrage erforderlichen Daten enthalten, können Anwendungen schnell die benötigten Informationen abrufen. Zusätzlich zum Verknüpfen von Tabellen oder Kombinieren von Datenentitäten können materialisierte Sichten die aktuellen Werte von berechneten Spalten oder Datenelementen, die Ergebnisse der Kombination von Werten oder der Ausführung von Transformationen mit den Datenelementen und Werte, die als Teil der Abfrage angegeben werden, enthalten. Eine materialisierte Sicht kann sogar für nur eine einzelne Abfrage optimiert werden.

Ein wichtiger Punkt ist, dass eine materialisierte Sicht und die darin enthaltenen Daten vollständig verworfen werden können, da sie aus den Quelldatenspeichern komplett neu erstellt werden können. Eine materialisierte Sicht wird nicht direkt von einer Anwendung aktualisiert, und daher handelt es sich um einen speziellen Cache.

Wenn sich die Quelldaten für die Sicht ändern, muss die Sicht aktualisiert werden, sodass die neuen Informationen enthalten sind. Sie können einrichten, dass dies automatisch oder immer dann, wenn das System eine Änderung an den ursprünglichen Daten erkennt, durchgeführt wird. In einigen Fällen kann es notwendig sein, die Sicht manuell erneut zu generieren. Die Abbildung zeigt ein Beispiel für eine mögliche Verwendung des Musters für materialisierte Sichten.

![Abbildung 1 zeigt ein Beispiel für eine mögliche Verwendung des Musters für materialisierte Sichten](./_images/materialized-view-pattern-diagram.png)


## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:

Wie und wann wird die Sicht aktualisiert. Im Idealfall wird sie als Reaktion auf ein Ereignis neu generiert, das auf eine Änderung an den Quelldaten hinweist. Dies kann jedoch zu übermäßigem Mehraufwand führen, wenn sich die Quelldaten schnell ändern. Erwägen Sie alternativ, eine geplante Aufgabe, einen externer Trigger oder eine manuelle Aktion zu verwenden, um die Sicht neu zu generieren.

In einigen Systemen, etwa wenn das Muster für die Ereignisherkunftsermittlung verwendet wird, um nur für Ereignisse, mit denen die Daten geändert wurden, einen Speicher zu verwalten, sind materialisierte Sichten erforderlich. Das Vorausfüllen von Sichten, indem alle Ereignisse überprüft werden, um den aktuellen Status zu bestimmen, ist möglicherweise die einzige Möglichkeit zum Abrufen von Informationen aus dem Ereignisspeicher. Wenn Sie die Ereignisherkunftsermittlung nicht verwenden, müssen Sie überlegen, ob eine materialisierte Sicht hilfreich sind kann. Materialisierte Sichten sind meist speziell auf eine Abfrage oder eine kleine Anzahl von Abfragen zugeschnitten. Wenn viele Abfragen verwendet werden, können materialisierte Sichten zu nicht akzeptablen Kapazitätsanforderungen und Kosten für den Speicher führen.

Berücksichtigen Sie die Auswirkungen auf die Datenkonsistenz beim Generieren und beim Aktualisieren der Sicht, wenn dies nach einem Zeitplan erfolgt. Wenn die Quelldaten genau dann geändert werden, wenn die Sicht generiert wird, ist die Kopie der Daten in der Sicht mit den ursprünglichen Daten nicht vollständig konsistent.

Überlegen Sie, wo Sie die Sicht speichern werden. Die Sicht muss sich nicht im gleichen Speicher oder in der gleichen Partition wie die ursprünglichen Daten befinden. Sie können eine Teilmenge von verschiedenen Partitionen kombinieren.

Eine Sicht kann neu erstellt werden, wenn sie verloren geht. Daher kann eine Sicht in einem Cache oder an einem weniger zuverlässigen Speicherort gespeichert werden, wenn sie nicht beständig ist und nur verwendet wird, um die Abfrageleistung zu verbessern, indem sie den aktuellen Status der Daten widerspiegelt, oder um die Skalierbarkeit zu verbessern.

Beim Definieren einer materialisierten Sicht können Sie deren Nutzen ggf. maximieren, indem Sie Datenelemente oder Spalten hinzufügen, die auf der Berechnung oder der Transformation vorhandener Datenelemente, auf in der Abfrage übergebenen Werten oder auf Kombinationen dieser Werte basieren.

Wenn der Speichermechanismus dies unterstützt, sollten Sie die materialisierte Sicht indizieren, um die Leistung zu steigern. Die meisten relationalen Datenbanken unterstützen eine Indizierung für Sichten. Dies gilt auch für Big Data-Lösungen, die auf Apache Hadoop basieren.

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

Dieses Muster ist für Folgendes hilfreich:
- Zum Erstellen von materialisierten Sichten für Daten, die nicht einfach direkt abgefragt werden können oder bei denen Abfragen sehr komplex sein müssen, um Daten zu extrahieren, die auf normalisierte, teilweise strukturierte oder unstrukturierte Weise gespeichert sind.
- Zum Erstellen temporärer Sichten, die die Abfrageleistung deutlich verbessern können oder direkt als Quellensichten oder Datenübertragungsobjekte für die Benutzeroberfläche, die Berichterstellung oder die Anzeige dienen können.
- Zum Unterstützen von Szenarien mit gelegentlichen oder ohne Verbindungen, wenn die Verbindung mit dem Datenspeicher nicht immer verfügbar ist. In diesem Fall kann die Sicht lokal zwischengespeichert werden.
- Zum Vereinfachen von Abfragen und zum Verfügbarmachen von Daten für Experimente in einer Weise, die keine Kenntnisse in Bezug auf das Quelldatenformat erfordert. Beispielsweise durch Verknüpfen von verschiedenen Tabellen in einer oder mehreren Datenbanken oder in einer oder mehreren Domänen in NoSQL-Speichern und anschließendes Formatieren der Daten entsprechend der tatsächlichen Verwendung.
- Zum Zugreifen auf bestimmte Teilmengen der Quelldaten, die aus Sicherheits- oder Datenschutzgründen nicht allgemein zugänglich oder offen für Änderungen sein sollen bzw. die nicht vollständig für Benutzer verfügbar gemacht werden sollen.
- Zum Überbrücken unterschiedlicher Datenspeicher, um deren jeweilige Funktionen zu nutzen. Beispiel: Verwenden eines Cloudspeichers, der beim Schreiben als Referenzdatenspeicher effizient ist, und einer relationalen Datenbank, die gute Abfrage- und Leseleistungen bietet, um die materialisierten Sichten zu speichern.

Dieses Muster ist in den folgenden Situationen nicht nützlich:
- Die Quelldaten sind einfach und können leicht abgefragt werden.
- Die Quelldaten ändern sich sehr schnell, oder Zugriff ist ohne eine Sicht möglich. In diesen Fällen sollten Sie den Verarbeitungsaufwand durch das Erstellen von Sichten vermeiden.
- Konsistenz hat einen hohen Stellenwert. Die Sichten sind möglicherweise nicht immer vollständig mit den ursprünglichen Daten konsistent.

## <a name="example"></a>Beispiel

Die folgende Abbildung zeigt ein Beispiel für die Verwendung des Musters für materialisierte Sichten, um eine Zusammenfassung der Umsätze zu generieren. Daten in den Tabellen „Order“, „OrderItem“ und „Customer“ in getrennten Partitionen in einem Azure-Speicherkonto werden kombiniert, um eine Sicht zu generieren, die den Gesamtwert der Verkäufe für einzelnen Produkte in der Kategorie „Electronics“ sowie die Anzahl der Kunden, die die einzelnen Artikel gekauft haben, enthält.

![Abbildung 2: Verwenden des Musters für materialisierte Sichten zum Generieren einer Umsatzzusammenfassung](./_images/materialized-view-summary-diagram.png)


Das Erstellen dieser materialisierten Sicht erfordert komplexe Abfragen. Indem jedoch das Abfrageergebnis als materialisierte Sicht verfügbar gemacht wird, können Benutzer problemlos die Ergebnisse abrufen und sie direkt verwenden oder in eine andere Abfrage integrieren. Die Sicht wird wahrscheinlich in einem Berichtssystem oder in einem Dashboard verwendet und kann nach einem Zeitplan z.B. wöchentlich aktualisiert werden.

>  Obwohl in diesem Beispiel Azure-Tabellenspeicher genutzt wird, bieten viele Managementsysteme für relationale Datenbanken auch native Unterstützung für materialisierte Sichten.

## <a name="related-patterns-and-guidance"></a>Zugehörige Muster und Anleitungen

Die folgenden Muster und Anleitungen können auch relevant sein, wenn dieses Muster implementiert wird:
- [Data Consistency Primer (Grundlagen der Datenkonsistenz)](https://msdn.microsoft.com/library/dn589800.aspx). Die Zusammenfassungsinformationen in einer materialisierten Sicht müssen verwaltet werden, sodass sie die zugrunde liegenden Datenwerte widerspiegeln. Wenn sich die Datenwerte ändern, ist es möglicherweise nicht praktikabel, die Zusammenfassungsdaten in Echtzeit zu aktualisieren. Stattdessen müssen Sie einen letztendlich konsistenten Ansatz einsetzen. Fasst die Probleme bei der Aufrechterhaltung von Konsistenz in verteilten Daten zusammen und beschreibt die Vor- und Nachteile der verschiedenen Konsistenzmodelle.
- [CQRS-Muster (Command and Query Responsibility Segregation)](cqrs.md). Wird verwendet, um die Informationen in einer materialisierten Sicht zu aktualisieren, indem auf Ereignisse reagiert wird, die aufgrund von Änderungen der zugrunde liegenden Datenwerte auftreten.
- [Muster für Ereignisherkunftsermittlung](event-sourcing.md). Wird in Verbindung mit dem CQRS-Muster verwendet, um die Informationen in einer materialisierten Sicht zu verwalten. Wenn die Datenwerte, auf denen eine materialisierte Sicht basiert, geändert werden, kann das System Ereignisse auslösen, die diese Änderungen beschreiben, und in einem Ereignisspeicher speichern.
- [Indextabellenmuster](index-table.md). Die Daten in einer materialisierten Sicht sind in der Regel mit einem Primärschlüssel organisiert. Abfragen müssen aber möglicherweise Informationen aus dieser Sicht abrufen, indem sie Daten in anderen Feldern untersuchen. Verwenden Sie dieses Muster, um für Datenspeicher, die native sekundäre Indizes nicht unterstützen, sekundäre Indizes über Datasets zu erstellen.
