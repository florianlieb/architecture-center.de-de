---
title: Onlinetransaktionsverarbeitung (OLTP)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 8650b919fc1a59240343015493a1fe41c8729a72
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="online-transaction-processing-oltp"></a>Onlinetransaktionsverarbeitung (OLTP)

Die Verwaltung von Transaktionsdaten mithilfe von Computersystemen wird als Onlinetransaktionsverarbeitung (Online Transaction Processing, OLTP) bezeichnet. OLTP-Systeme zeichnen Geschäftsinteraktionen auf, die im täglichen Betrieb der Organisation bzw. des Unternehmens stattfinden, und unterstützen das Abfragen dieser Daten, um Rückschlüsse zu ziehen.

## <a name="transactional-data"></a>Transaktionsdaten

Transaktionsdaten sind Informationen, mit denen Interaktionen in Zusammenhang mit den Aktivitäten einer Organisation nachverfolgt werden. Bei diesen Interaktionen handelt es sich in der Regel um Geschäftstransaktionen, etwa von Kunden erhaltene Zahlungen, Zahlungen an Zulieferer, Produkte im Bestand, entgegengenommene Aufträge oder bereitgestellte Dienste. Transaktionsereignisse, die die Transaktionen selbst darstellen, enthalten in der Regel eine Zeitdimension, einige numerische Werte und Verweise auf andere Daten. 

Transaktionen müssen normalweise *unteilbar* und *konsistent* sein. Unteilbarkeit bedeutet, dass eine gesamte Transaktion immer als eine Arbeitseinheit erfolgreich ist oder fehlschlägt und nie in einem halb abgeschlossenen Zustand belassen wird. Wenn eine Transaktion nicht abgeschlossen werden kann, muss das Datenbanksystem einen Rollback für alle Schritte ausführen, die im Rahmen der Transaktion bereits durchgeführt wurden. In einem herkömmlichen RDBMS erfolgt dieser Rollback automatisch, wenn eine Transaktion nicht abgeschlossen werden kann. Konsistenz bedeutet, dass Transaktionen die Daten immer in einem gültigen Zustand hinterlassen. (Hierbei handelt es sich um sehr informelle Beschreibungen von Unteilbarkeit und Konsistenz. Es gibt offiziellere Definitionen für diese Eigenschaften, etwa [ACID](https://en.wikipedia.org/wiki/ACID).)

Transaktionsdatenbanken können mit verschiedenen Sperrstrategien (etwa pessimistisches Sperren) hohe Konsistenz für Transaktionen unterstützen, um eine hohe Konsistenz aller Daten im Unternehmenskontext für alle Benutzer und Prozesse zu gewährleisten. 

Die Datenspeicherebene in einer dreischichtigen Architektur ist die am häufigsten verwendete Bereitstellungsarchitektur, die Transaktionsdaten verwendet. Eine dreischichtige Architektur setzt sich üblicherweise aus einer Präsentationsebene, einer Geschäftslogikebene und einer Datenspeicherebene zusammen. Eine ähnliche Bereitstellungsarchitektur ist die [n-schichtige](/azure/architecture/guide/architecture-styles/n-tier) Architektur, die mehrere mittlere Ebenen für die Verarbeitung von Geschäftslogik enthalten kann.

## <a name="typical-traits-of-transactional-data"></a>Typische Merkmale von Transaktionsdaten

Transaktionsdaten weisen tendenziell die folgenden Merkmale auf:

| Anforderung | BESCHREIBUNG |
| --- | --- |
| Normalisierung | Stark normalisiert |
| Schema | Schema beim Schreiben, strikte Erzwingung|
| Konsistenz | Hohe Konsistenz, ACID-Garantien |
| Integrität | Hohe Integrität |
| Verwendung von Transaktionen | Ja |
| Sperrstrategie | Pessimistisch oder optimistisch|
| Aktualisierbar | Ja |
| Erweiterbar | Ja |
| Workload | Intensive Schreibvorgänge, moderate Lesevorgänge |
| Indizierung | Primäre und sekundäre Indizes |
| Bezugsgröße | Kleine bis mittlere Größe |
| Modell | Relational |
| Datenform | Tabellarisch |
| Abfrageflexibilität | Sehr flexibel |
| Skalieren | Klein (MBs) bis groß (einige TBs) | 

## <a name="when-to-use-this-solution"></a>Verwendung dieser Lösung

OLTP ist die geeignete Wahl, wenn Sie Geschäftstransaktionen effizient verarbeiten und speichern und sofort auf konsistente Weise für Clientanwendungen verfügbar machen müssen. Verwenden Sie diese Architektur, wenn sich spürbare Verzögerungen bei der Verarbeitung negativ auf den täglichen Betrieb des Unternehmens auswirken würden.

OLTP-Systeme sind zur effizienten Verarbeitung und Speicherung von Transaktionen sowie zum effizienten Abfragen von Transaktionsdaten konzipiert. Die effiziente Verarbeitung und Speicherung einzelner Transaktionen durch ein OLTP-System wird zum Teil durch Datennormalisierung erzielt, d. h. die Aufteilung der Daten in kleinere weniger redundante Blöcke. Dies steigert die Effizienz, da das OLTP-System so eine große Anzahl von Transaktionen unabhängig voneinander verarbeiten kann und die zusätzliche Verarbeitung entfällt, die zum Aufrechterhalten der Datenintegrität erforderlich ist, wenn redundante Daten vorhanden sind.

## <a name="challenges"></a>Herausforderungen
Die Implementierung und Verwendung eines OLTP-Systems kann einige Herausforderungen mit sich bringen:

- OLTP-Systeme eignen sich nicht immer gut zum Verarbeiten von Aggregaten für große Datenmengen, es gibt jedoch Ausnahmen, beispielsweise eine sorgfältig geplante SQL Server-basierte Lösung. Analysen der Daten, die auf Aggregatberechnungen für Millionen einzelner Transaktionen beruhen, sind für ein OLTP-System sehr ressourcenintensiv. Ihre Ausführung kann viel Zeit in Anspruch nehmen und andere Transaktionen in der Datenbank verlangsamen oder blockieren.
- Bei der Ausführung von Analysen und Erstellung von Berichten für stark normalisierte Daten sind die Abfragen in der Regel komplex, da die meisten Abfragen die Daten mithilfe von Verknüpfungen denormalisieren müssen. Zudem erfordern die Namenskonventionen für Datenbankobjekte in OLTP-Systemen oft sehr kurze und prägnante Namen. Die stärkere Normalisierung in Verbindung mit diesen Namenskonventionen macht es für Geschäftsbenutzer schwierig, OLTP-Systeme ohne die Hilfe eines Datenbankadministrators oder Datenentwicklers abzufragen.
- Das unbegrenzte Speichern des Verlaufs von Transaktionen und das Speichern von zu vielen Daten in einer Tabelle können je nach Anzahl gespeicherter Transaktionen die Abfrageleistung beeinträchtigen. Die allgemeine Lösung besteht darin, ein relevantes Zeitfenster (z. B. das aktuelle Geschäftsjahr) im OLTP-System zu verwalten und historische Daten in andere Systeme auszulagern, beispielsweise in einen Data Mart oder ein [Data Warehouse](./data-warehousing.md).

## <a name="oltp-in-azure"></a>OLTP in Azure

Anwendungen wie in [App Service-Web-Apps](/azure/app-service/app-service-web-overview) gehostete Websites, in App Service ausgeführte REST-APIs, mobile Anwendungen oder Desktopanwendungen kommunizieren in der Regel über eine REST-API-Zwischenstufe mit dem OLTP-System.

In der Praxis sind die meisten Workloads keine reinen OLTP-Workloads. Meist gibt es auch eine analytische Komponente. Darüber hinaus besteht ein zunehmender Bedarf an Funktionen zur Echtzeit-Berichterstellung, beispielsweise zur Ausführung von Berichten für das Betriebssystem. Dies wird auch als HTAP (Hybrid Transactional and Analytical Processing, hybride Verarbeitung von Transaktionen und Analysen) bezeichnet. Weitere Informationen finden Sie unter [Online analytical processing (OLAP)](./online-analytical-processing.md) (Analytische Onlineverarbeitung (OLAP)).

In Azure erfüllen alle folgenden Datenspeicher die grundlegenden Anforderungen für OLTP und die Verwaltung von Transaktionsdaten:

- [Azure SQL-Datenbank](/azure/sql-database/)
- [SQL Server auf einer Azure-VM](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [Azure Database for MySQL](/azure/mysql/)
- [Azure-Datenbank für PostgreSQL](/azure/postgresql/)

## <a name="key-selection-criteria"></a>Wichtige Auswahlkriterien

Beantworten Sie die folgenden Fragen, um die Auswahl einzuschränken:

- Möchten Sie einen verwalteten Dienst verwenden, anstatt Ihre eigenen Server zu verwalten?

- Gelten für Ihre Lösung bestimmte Abhängigkeiten bezüglich der Kompatibilität mit Microsoft SQL Server, MySQL oder PostgreSQL? Die Treiber, die Ihre Anwendung zur Kommunikation mit dem Datenspeicher unterstützt, oder die Annahmen, die sie bezüglich der verwendeten Datenbank trifft, können die zur Auswahl verfügbaren Datenspeicher begrenzen.

- Sind Ihre Anforderungen im Hinblick auf den Durchsatz von Schreibvorgängen besonders hoch? Wenn dies der Fall ist, sollten Sie eine Option auswählen, die In-Memory-Tabellen bereitstellt. 

- Ist Ihre Lösung mehrinstanzenfähig? Wenn dies der Fall ist, sollten Sie Optionen mit Unterstützung für Kapazitätspools erwägen, bei denen mehrere Datenbankinstanzen anstelle von festen Ressourcen pro Datenbank einen Pool für elastische Ressourcen nutzen. Dadurch können Sie die Kapazität besser auf alle Datenbankinstanzen verteilen und die Kosteneffektivität Ihrer Lösung verbessern.

- Müssen Ihre Daten mit geringer Wartezeit in mehreren Regionen lesbar sein? Wenn dies der Fall ist, sollten Sie eine Option auswählen, die lesbare sekundäre Replikate unterstützt.

- Muss Ihre Datenbank über geografische Regionen hinweg hoch verfügbar sein? Wenn dies der Fall ist, sollten Sie eine Option auswählen, die die geografische Replikation unterstützt. Erwägen Sie auch die Optionen, die ein automatisches Failover vom primären Replikat zu einem sekundären Replikat unterstützen.

- Gelten für Ihre Datenbank bestimmte Sicherheitsanforderungen? Wenn dies der Fall ist, sollten Sie die Optionen erwägen, die Funktionen wie Sicherheit auf Zeilenebene, Datenmaskierung und transparente Datenverschlüsselung bieten.

## <a name="capability-matrix"></a>Funktionsmatrix

In den folgenden Tabellen sind die Hauptunterschiede in Bezug auf die Funktionen zusammengefasst.

### <a name="general-capabilities"></a>Allgemeine Funktionen 

|                              | Azure SQL-Datenbank | SQL Server auf einer Azure-VM | Azure Database for MySQL | Azure Database for PostgreSQL |
|------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|      Verwalteter Dienst      |        Ja         |                   Nein                   |           Ja            |              Ja              |
|       Unterstützte Plattformen       |        N/V         |         Windows, Linux, Docker         |           N/V            |              N/V              |
| Programmierbarkeit<sup>1</sup> |   T-SQL, .NET, R   |         T-SQL, .NET, R, Python         |  T-SQL, .NET, R, Python  |              SQL              |

[1] Ohne Clienttreiberunterstützung, wodurch viele Programmiersprachen eine Verbindung mit dem OLTP-Datenspeicher herstellen und diesen verwenden können

### <a name="scalability-capabilities"></a>Skalierbarkeitsfunktionen

| | Azure SQL-Datenbank | SQL Server auf einer Azure-VM| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- |
| Maximale Größe der Datenbankinstanz | [4 TB](/azure/sql-database/sql-database-resource-limits) | 256 TB | [1 TB](/azure/mysql/concepts-limits) | [1 TB](/azure/postgresql/concepts-limits) |
| Unterstützung von Kapazitätspools  | Ja | Ja | Nein  | Nein  |
| Unterstützung des horizontalen Hochskalierens von Clustern  | Nein  | Ja | Nein  | Nein  |
| Dynamische Skalierbarkeit (zentrales Hochskalieren)  | Ja | Nein | Ja | Ja |

### <a name="analytic-workload-capabilities"></a>Funktionen für Analyseworkloads

| | Azure SQL-Datenbank | SQL Server auf einer Azure-VM| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Temporäre Tabellen | Ja | Ja | Nein  | Nein  |
| In-Memory-Tabellen (speicheroptimiert) | Ja | Ja | Nein  | Nein  |
| Columnstore-Unterstützung | Ja | Ja | Nein  | Nein  |
| Adaptive Abfrageverarbeitung | Ja | Ja | Nein  | Nein  |

### <a name="availability-capabilities"></a>Verfügbarkeitsfunktionen

| | Azure SQL-Datenbank | SQL Server auf einer Azure-VM| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Lesbare sekundäre Replikate | Ja | Ja | Nein  | Nein  | 
| Geografische Replikation | Ja | Ja | Nein  | Nein  | 
| Automatisches Failover zum sekundären Replikat | Ja | Nein  | Nein  | Nein |
| Point-in-Time-Wiederherstellung | Ja | Ja | Ja | Ja |

### <a name="security-capabilities"></a>Sicherheitsfunktionen

|                                                                                                             | Azure SQL-Datenbank | SQL Server auf einer Azure-VM | Azure Database for MySQL | Azure Database for PostgreSQL |
|-------------------------------------------------------------------------------------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|                                             Sicherheit auf Zeilenebene                                              |        Ja         |                  Ja                   |           Ja            |              Ja              |
|                                                Datenmaskierung                                                 |        Ja         |                  Ja                   |            Nein             |              Nein                |
|                                         Transparent Data Encryption                                         |        Ja         |                  Ja                   |           Ja            |              Ja              |
|                                  Beschränken des Zugriffs auf bestimmte IP-Adressen                                   |        Ja         |                  Ja                   |           Ja            |              Ja              |
|                                  Beschränken des Zugriffs, um nur VNET-Zugriff zuzulassen                                  |        Ja         |                  Ja                   |            Nein             |              Nein                |
|                                    Authentifizierung über Azure Active Directory                                    |        Ja         |                  Ja                   |            Nein             |              Nein                |
|                                       Authentifizierung über Active Directory                                       |         Nein          |                  Ja                   |            Nein             |              Nein                |
|                                         Multi-Factor Authentication                                         |        Ja         |                  Ja                   |            Nein             |              Nein                |
| Unterstützung von [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) |        Ja         |                  Ja                   |           Ja            |              Nein                |
|                                                 Private IP-Adresse                                                  |         Nein          |                  Ja                   |           Ja            |              Nein                |

