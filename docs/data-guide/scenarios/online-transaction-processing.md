---
title: Onlinetransaktionsverarbeitung (OLTP)
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 07e7f680c8ee5e8589ff7cd2236ff95f6ee84f4c
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="online-transaction-processing-oltp"></a>Onlinetransaktionsverarbeitung (OLTP)

Die Verwaltung von [Transaktionsdaten](../concepts/transactional-data.md) mithilfe von Computersystemen wird als Onlinetransaktionsverarbeitung (Online Transaction Processing, OLTP) bezeichnet. OLTP-Systeme zeichnen Geschäftsinteraktionen auf, die im täglichen Betrieb der Organisation bzw. des Unternehmens stattfinden, und unterstützen das Abfragen dieser Daten, um Rückschlüsse zu ziehen.

![OLTP in Azure](./images/oltp-data-pipeline.png)

## <a name="when-to-use-this-solution"></a>Verwendung dieser Lösung

OLTP ist die geeignete Wahl, wenn Sie Geschäftstransaktionen effizient verarbeiten und speichern und sofort auf konsistente Weise für Clientanwendungen verfügbar machen müssen. Verwenden Sie diese Architektur, wenn sich spürbare Verzögerungen bei der Verarbeitung negativ auf den täglichen Betrieb des Unternehmens auswirken würden.

OLTP-Systeme sind zur effizienten Verarbeitung und Speicherung von Transaktionen sowie zum effizienten Abfragen von Transaktionsdaten konzipiert. Die effiziente Verarbeitung und Speicherung einzelner Transaktionen durch ein OLTP-System wird zum Teil durch Datennormalisierung erzielt, d. h. die Aufteilung der Daten in kleinere weniger redundante Blöcke. Dies steigert die Effizienz, da das OLTP-System so eine große Anzahl von Transaktionen unabhängig voneinander verarbeiten kann und die zusätzliche Verarbeitung entfällt, die zum Aufrechterhalten der Datenintegrität erforderlich ist, wenn redundante Daten vorhanden sind.

## <a name="challenges"></a>Herausforderungen
Die Implementierung und Verwendung eines OLTP-Systems kann einige Herausforderungen mit sich bringen:

- OLTP-Systeme eignen sich nicht immer gut zum Verarbeiten von Aggregaten für große Datenmengen, es gibt jedoch Ausnahmen, beispielsweise eine sorgfältig geplante SQL Server-basierte Lösung. Analysen der Daten, die auf Aggregatberechnungen für Millionen einzelner Transaktionen beruhen, sind für ein OLTP-System sehr ressourcenintensiv. Ihre Ausführung kann viel Zeit in Anspruch nehmen und andere Transaktionen in der Datenbank verlangsamen oder blockieren.
- Bei der Ausführung von Analysen und Erstellung von Berichten für stark normalisierte Daten sind die Abfragen in der Regel komplex, da die meisten Abfragen die Daten mithilfe von Verknüpfungen denormalisieren müssen. Zudem erfordern die Namenskonventionen für Datenbankobjekte in OLTP-Systemen oft sehr kurze und prägnante Namen. Die stärkere Normalisierung in Verbindung mit diesen Namenskonventionen macht es für Geschäftsbenutzer schwierig, OLTP-Systeme ohne die Hilfe eines Datenbankadministrators oder Datenentwicklers abzufragen.
- Das unbegrenzte Speichern des Verlaufs von Transaktionen und das Speichern von zu vielen Daten in einer Tabelle können je nach Anzahl gespeicherter Transaktionen die Abfrageleistung beeinträchtigen. Die allgemeine Lösung besteht darin, ein relevantes Zeitfenster (z. B. das aktuelle Geschäftsjahr) im OLTP-System zu verwalten und historische Daten in andere Systeme auszulagern, beispielsweise in einen Data Mart oder ein [Data Warehouse](../technology-choices/data-warehouses.md).

## <a name="oltp-in-azure"></a>OLTP in Azure

Anwendungen wie in [App Service-Web-Apps](/azure/app-service/app-service-web-overview) gehostete Websites, in App Service ausgeführte REST-APIs, mobile Anwendungen oder Desktopanwendungen kommunizieren in der Regel über eine REST-API-Zwischenstufe mit dem OLTP-System.

In der Praxis sind die meisten Workloads keine reinen OLTP-Workloads. Meist gibt es auch eine [analytische Komponente](../scenarios/online-analytical-processing.md). Darüber hinaus besteht ein zunehmender Bedarf an Funktionen zur Echtzeit-Berichterstellung, beispielsweise zur Ausführung von Berichten für das Betriebssystem. Dies wird auch als HTAP (Hybrid Transactional and Analytical Processing, hybride Verarbeitung von Transaktionen und Analysen) bezeichnet. Weitere Informationen finden Sie unter [Auswählen eines OLAP-Datenspeichers in Azure](../technology-choices/olap-data-stores.md).

## <a name="technology-choices"></a>Auswahl der Technologie

Datenspeicher:

- [Azure SQL-Datenbank](/azure/sql-database/)
- [SQL Server auf einer Azure-VM](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [Azure Database for MySQL](/azure/mysql/)
- [Azure-Datenbank für PostgreSQL](/azure/postgresql/)

Weitere Informationen finden Sie unter [Auswählen eines OLTP-Datenspeichers in Azure](../technology-choices/oltp-data-stores.md).

Datenquellen:

- [App Service](/azure/app-service/)
- [Mobile Apps](/azure/app-service-mobile/)

