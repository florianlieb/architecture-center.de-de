---
title: "Auswählen eines OLTP-Datenspeichers"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 1c27d7d5f3b78f40822de6b77664dbf49b1367f6
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-an-oltp-data-store-in-azure"></a>Auswählen eines OLTP-Datenspeichers in Azure

Onlinetransaktionsverarbeitung (Online Transaction Processing, OLTP) ist die Verwaltung von Transaktionsdaten und Verarbeitung von Transaktionen. In diesem Thema werden Optionen für OLTP-Lösungen in Azure verglichen.

> [!NOTE]
> Weitere Informationen dazu, wann Sie einen OLTP-Datenspeicher verwenden sollten, finden Sie unter [Online transaction processing](../scenarios/online-analytical-processing.md) (Onlinetransaktionsverarbeitung [OLTP]).

## <a name="what-are-your-options-when-choosing-an-oltp-data-store"></a>Welche Optionen stehen Ihnen bei der Auswahl eines OLTP-Datenspeichers zur Verfügung?

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
| | Azure SQL-Datenbank | SQL Server auf einer Azure-VM | Azure Database for MySQL | Azure Database for PostgreSQL |
| --- | --- | --- | --- | --- | --- |
| Verwalteter Dienst | Ja | Nein | Ja | Ja |
| Unterstützte Plattformen | N/V | Windows, Linux, Docker | N/V | N/V |
| Programmierbarkeit<sup>1</sup> | T-SQL, .NET, R | T-SQL, .NET, R, Python | T-SQL, .NET, R, Python | SQL | SQL |

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
| | Azure SQL-Datenbank | SQL Server auf einer Azure-VM| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Sicherheit auf Zeilenebene | Ja | Ja | Ja | Ja |
| Datenmaskierung | Ja | Ja | Nein  | Nein  |
| Transparent Data Encryption | Ja | Ja | Ja | Ja |
| Beschränken des Zugriffs auf bestimmte IP-Adressen | Ja | Ja | Ja | Ja |
| Beschränken des Zugriffs, um nur VNET-Zugriff zuzulassen | Ja | Ja | Nein  | Nein  |
| Authentifizierung über Azure Active Directory | Ja | Ja | Nein  | Nein  |
| Authentifizierung über Active Directory | Nein  | Ja | Nein  | Nein  |
| Multi-Factor Authentication | Ja | Ja | Nein  | Nein  |
| Unterstützung von [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) | Ja | Ja | Ja | Nein  | Nein  |
| Private IP-Adresse | Nein  | Ja | Ja | Nein  | Nein  |

