---
title: "Kriterien für die Auswahl eines Datenspeichers"
description: "Übersicht über Azure-Computeoptionen"
author: MikeWasson
ms.openlocfilehash: 7fb75cd334438c5b985fa04ad8afe3236f2391f8
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="criteria-for-choosing-a-data-store"></a>Kriterien für die Auswahl eines Datenspeichers

Azure unterstützt zahlreiche Typen von Datenspeicherlösungen mit jeweils unterschiedlichen Features und Funktionen. In diesem Artikel werden die Vergleichskriterien beschrieben, die Sie bei der Bewertung eines Datenspeichers anwenden sollten. Sie sollen Ihnen dabei helfen, zu bestimmen, welche Datenspeichertypen die Anforderungen Ihrer Lösung erfüllen können.

## <a name="general-considerations"></a>Allgemeine Überlegungen

Sammeln Sie für den Vergleich zunächst möglichst viele der folgenden Informationen zu den Anforderungen für Ihre Daten. Anhand dieser Informationen können Sie bestimmen, welche Datenspeichertypen Ihren Anforderungen gerecht werden.

### <a name="functional-requirements"></a>Funktionsanforderungen

- **Datenformat:** Welche Arten von Daten möchten Sie speichern? Zu den allgemeinen Typen gehören Transaktionsdaten, JSON-Objekte, Telemetriedaten, Suchindizes oder Flatfiles.
- **Datengröße:** Wie umfangreich sind die Entitäten, die Sie speichern möchten? Müssen diese Entitäten als einzelnes Dokument beibehalten werden, oder können sie auf mehrere Dokumente, Tabellen, Sammlungen usw. aufgeteilt werden?
- **Skalierung und Struktur:** Welche Gesamtmenge an Speicherkapazität benötigen Sie? Gehen Sie davon aus, dass Sie die Daten partitionieren? 
- **Datenbeziehungen:** Müssen Ihre Daten 1:n- oder m:n-Beziehungen unterstützen? Sind die Beziehungen selbst ein wichtiger Teil der Daten? Müssen Sie Daten innerhalb des gleichen DataSets oder aus externen DataSets verknüpfen oder auf andere Weise kombinieren? 
- **Konsistenzmodell:** Wie wichtig ist es, dass in einem Knoten durchgeführte Aktualisierungen in anderen Knoten angezeigt werden, damit weitere Änderungen vorgenommen werden können? Können Sie letztlich Konsistenz akzeptieren? Benötigen Sie ACID-Garantien für Transaktionen?
- **Schemaflexibilität:** Welche Art von Schemas wenden Sie auf Ihre Daten an? Verwenden Sie ein festes Schema, ein Schema bei Schreibvorgängen oder ein Schema bei Lesevorgängen?
- **Parallelität:** Welche Art von Parallelitätsmechanismus möchten Sie beim Aktualisieren und Synchronisieren von Daten verwenden? Werden in der Anwendung viele Aktualisierungen durchgeführt, die potenziell zu Konflikten führen? Wenn dies der Fall ist, benötigen Sie möglicherweise die Datensatzsperrung und Steuerung für pessimistische Parallelität. Können Sie alternativ die Steuerung für optimistische Parallelität unterstützen? Wenn ja, ist eine einfache zeitstempelbasierte Parallelitätssteuerung ausreichend. Oder benötigen Sie die erweiterte Funktion der Parallelitätssteuerung mit mehreren Versionen?
- **Datenverschiebung:** Müssen in Ihrer Lösung ETL-Aufgaben durchgeführt werden, um Daten in andere Speicher oder Data Warehouses zu verschieben?
- **Datenlebenszyklus:** Werden die Daten einmal geschrieben und häufig gelesen? Können sie in Cool oder Cold Storage verschoben werden?
- **Andere unterstützte Features:** Benötigen Sie andere spezifische Features, z.B. Schemaüberprüfung, Aggregation, Indizierung, Volltextsuche, MapReduce oder andere Abfragefunktionen?

### <a name="non-functional-requirements"></a>Nicht funktionsbezogene Anforderungen

- **Leistung und Skalierbarkeit:** Wie lauten Ihre Anforderungen an die Datenleistung? Haben Sie spezielle Anforderungen an Datenerfassungs- und Datenverarbeitungsraten? Welche zulässigen Antwortzeiten für die Abfrage und Aggregation der Daten nach der Erfassung werden benötigt? Auf welche Größe muss der Datenspeicher zentral hochskaliert werden können? Ist Ihre Workload eher leseintensiv oder schreibintensiv?
- **Zuverlässigkeit:** Welche Gesamt-SLA muss unterstützt werden? Welche Fehlertoleranzebene müssen Sie für Datenconsumer bereitstellen? Welche Art von Sicherungs-und Wiederherstellungsfunktionen benötigen Sie? 
- **Replikation:** Müssen Ihre Daten auf mehrere Replikate oder Regionen verteilt werden? Welche Art von Datenreplikationsfunktionen benötigen Sie? 
- **Einschränkungen:** Entsprechen die Einschränkungen eines bestimmten Datenspeichers Ihren Anforderungen an die Skalierung, die Anzahl der Verbindungen und den Durchsatz? 

### <a name="management-and-cost"></a>Verwaltung und Kosten

- **Verwalteter Dienst:** Verwenden Sie nach Möglichkeit einen verwalteten Datendienst, es sei denn, Sie benötigen bestimmte Funktionen, die nur in einem IaaS-gehosteten Datenspeicher enthalten sind.
- **Regionale Verfügbarkeit:** Ist der Dienst im Fall von verwalteten Diensten in allen Azure-Regionen verfügbar? Muss Ihre Lösung in bestimmten Azure-Regionen gehostet werden?
- **Übertragbarkeit:** Müssen Ihre Daten zu lokalen oder externen Rechenzentren oder zu anderen Cloudhostingumgebungen migriert werden?
- **Lizenzierung:** Bevorzugen Sie einen proprietären gegenüber einem OSS-Lizenztyp? Gibt es andere externe Einschränkungen im Hinblick auf den zu verwendenden Lizenztyp?
- **Gesamtkosten:** Wie hoch sind die Gesamtkosten für die Verwendung des Diensts in Ihrer Lösung? Wie viele Instanzen müssen ausgeführt werden, um Ihren Anforderungen an die Betriebszeit und den Durchsatz gerecht zu werden? Berücksichtigen Sie bei dieser Berechnung auch die Betriebskosten. Ein Grund für verwaltete Dienste sind die reduzierten Betriebskosten.
- **Kosteneffizienz:** Können Sie Ihre Daten partitionieren, um sie kosteneffektiver zu speichern? Können beispielsweise umfangreiche Objekte aus einer kostenintensiven relationalen Datenbank in einen Objektspeicher verschoben werden?

### <a name="security"></a>Sicherheit

- **Sicherheit**. Welche Art von Verschlüsselung benötigen Sie? Benötigen Sie die Verschlüsselung ruhender Daten? Welche Authentifizierungsmethode möchten Sie zur Herstellung einer Verbindung mit den Daten verwenden?
- **Überwachung:** Welche Art von Überwachungsprotokoll müssen Sie generieren?
- **Netzwerkanforderungen:** Muss der Zugriff auf Ihre Daten über andere Netzwerkressourcen beschränkt oder auf andere Weise verwaltet werden? Darf nur innerhalb der Azure-Umgebung auf die Daten zugegriffen werden? Müssen die Daten über bestimmte IP-Adressen oder Subnetze zugänglich sein? Müssen sie über Anwendungen oder Dienste zugänglich sein, die lokal oder in anderen externen Rechenzentren gehostet werden?

### <a name="devops"></a>DevOps

- **Fähigkeiten:** Ist Ihr Team besonders vertraut mit der Verwendung bestimmter Programmiersprachen, Betriebssysteme oder anderen Technologien? Stellen andere dagegen potenziell eine Schwierigkeit für das Team dar?
- **Clients:** Besteht eine gute Clientunterstützung für Ihre Entwicklungssprachen?

In den folgenden Abschnitten werden verschiedene Datenspeichermodelle in Bezug auf das Workloadprofil, die Datentypen und Beispiele für Anwendungsfälle verglichen.

## <a name="relational-database-management-systems-rdbms"></a>Managementsysteme für relationale Datenbanken (RDBMS)

<table>
<tr><td>**Workload**</td>
    <td>
        <ul>
            <li>Die Erstellung neuer Datensätze sowie die Aktualisierung vorhandener Daten werden regelmäßig durchgeführt.</li>
            <li>Mehrere Vorgänge müssen in einer einzelnen Transaktion abgeschlossen werden.</li>
            <li>Aggregationsfunktionen sind für Kreuztabellen erforderlich.</li>
            <li>Starke Integration in Berichtstools ist erforderlich.</li>
            <li>Beziehungen werden mithilfe von Datenbankeinschränkungen erzwungen.</li>
            <li>Die Abfrageleistung wird mithilfe von Indizes optimiert.</li>
            <li>Ermöglicht den Zugriff auf bestimmte Teilmengen von Daten.</li>
        </ul>
    </td>
</tr>
<tr><td>**Datentyp**</td>
    <td>
        <ul>
            <li>Daten sind stark normalisiert.</li>
            <li>Datenbankschemas sind erforderlich und werden erzwungen.</li>
            <li>m:n-Beziehungen zwischen Datenentitäten in der Datenbank.</li>
            <li>Einschränkungen werden im Schema definiert und gelten für alle Daten in der Datenbank.</li>
            <li>Daten erfordern eine hohe Integrität. Indizes und Beziehungen müssen genau verwaltet werden.</li>
            <li>Daten erfordern eine starke Konsistenz. Transaktionen werden so durchgeführt, dass sichergestellt wird, dass alle Daten für alle Benutzer und Prozesse 100 % konsistent sind.</li>
            <li>Die Größe der einzelnen Dateneinträge soll klein bis mittelgroß sein.</li>
        </ul>
    </td>
</tr>
<tr><td>**Beispiele**</td>
    <td>
        <ul>
            <li>Branchenspezifisch (Personalverwaltung, Customer Relationship Management, Enterprise Resource Planning)</li>
            <li>Bestandsverwaltung</li>
            <li>Berichtsdatenbank</li>
            <li>Buchhaltung</li>
            <li>Asset-Management</li>
            <li>Fondsmanagement</li>
            <li>Bestellungsverwaltung</li>
        </ul>
    </td>
</tr>
</table>

## <a name="document-databases"></a>Dokumentdatenbanken

<table>
<tr><td>**Workload**</td>
    <td>
        <ul>
            <li>Allgemeiner Zweck.</li>
            <li>Einfüge- und Aktualisierungsvorgänge werden häufig durchgeführt. Die Erstellung neuer Datensätze sowie die Aktualisierung vorhandener Daten werden regelmäßig durchgeführt.</li>
            <li>Keine objektrelationalen Impedanzabweichungen. Dokumente können besser mit den im Anwendungscode verwendeten Objektstrukturen abgeglichen werden.</li>
            <li>Optimistische Parallelität wird häufiger verwendet.</li>
            <li>Daten müssen durch die verarbeitende Anwendung geändert und verarbeitet werden.</li>
            <li>Für die Daten ist ein Index für mehrere Felder erforderlich.</li>
            <li>Einzelne Dokumente werden abgerufen und als einzelner Block geschrieben.</li>
    </td>
</tr>
<tr><td>**Datentyp**</td>
    <td>
        <ul>
            <li>Daten können auf denormalisierte Weise verwaltet werden.</li>
            <li>Die Größe der einzelnen Dokumentdaten ist relativ gering.</li>
            <li>Für jeden Dokumenttyp kann ein eigenes Schema verwendet werden.</li>
            <li>Dokumente können optionale Felder enthalten.</li>
            <li>Dokumentdaten sind teilweise strukturiert, d.h., die Datentypen der einzelnen Felder sind nicht streng definiert.</li>
            <li>Die Datenaggregation wird unterstützt.</li>
        </ul>
    </td>
</tr>
<tr><td>**Beispiele**</td>
    <td>
        <ul>
            <li>Produktkatalog</li>
            <li>Benutzerkonten</li>
            <li>Stückliste</li>
            <li>Personalisierung</li>
            <li>Content Management</li>
            <li>Betriebsdaten</li>
            <li>Bestandsverwaltung</li>
            <li>Transaktionsverlaufsdaten</li>
            <li>Materialisierte Sicht anderer NoSQL-Datenspeicher. Ersetzt Datei-/Blob-Indizierung.</li>
        </ul>
    </td>
</tr>
</table>

## <a name="keyvalue-stores"></a>Schlüssel-Wert-Speicher

<table>
<tr><td>**Workload**</td>
    <td>
        <ul>
            <li>Die Identifizierung und der Zugriff auf Daten erfolgen mithilfe eines einzigen ID-Schlüssels, z.B. über ein Wörterbuch.</li>
            <li>Extrem skalierbar.</li>
            <li>Keine Verknüpfungen, Sperren oder Unions sind erforderlich.</li>
            <li>Keine Aggregationsmechanismen werden verwendet.</li>
            <li>Sekundäre Indizes werden generell nicht verwendet.</li>
        </ul>
    </td>
</tr>
<tr><td>**Datentyp**</td>
    <td>
        <ul>
            <li>Die Datengröße ist tendenziell umfangreich.</li>
            <li>Jeder Schlüssel ist einem einzelnen Wert zugeordnet, d.h. einem nicht verwalteten Datenblob.</li>
            <li>Keine Schemas werden erzwungen.</li>
            <li>Keine Beziehungen zwischen Entitäten.</li>
        </ul>
    </td>
</tr>
<tr><td>**Beispiele**</td>
    <td>
        <ul>
            <li>Datenzwischenspeicherung</li>
            <li>Sitzungsverwaltung</li>
            <li>Benutzereinstellungs- und Benutzerprofilverwaltung</li>
            <li>Produktempfehlungen und Ad-Serving</li>
            <li>Wörterbücher</li>
        </ul>
    </td>
</tr>
</table>

## <a name="graph-databases"></a>Diagrammdatenbanken

<table>
<tr><td>**Workload**</td>
    <td>
        <ul>
            <li>Die Beziehungen zwischen Datenelementen sind sehr komplex und erfordern viele Hops zwischen zugehörigen Datenelementen.</li>
            <li>Die Beziehungen zwischen Datenelementen sind dynamisch und ändern sich mit der Zeit.</li>
            <li>Beziehungen zwischen Objekten sind bevorzugte Beziehungen, bei denen keine Fremdschlüssel und Verknüpfungen durchlaufen werden müssen.</li>
        </ul>
    </td>
</tr>
<tr><td>**Datentyp**</td>
    <td>
        <ul>
            <li>Daten bestehen aus Knoten und Beziehungen.</li>
            <li>Knoten ähneln Tabellenzeilen oder JSON-Dokumenten.</li>
            <li>Beziehungen sind genau so wichtig wie Knoten und werden direkt in der Abfragesprache verfügbar gemacht.</li>
            <li>Zusammengesetzte Objekte, z.B. eine Person mit mehreren Telefonnummern, werden meist in separate kleinere Knoten unterteilt und mit durchlaufbaren Beziehungen kombiniert. </li>
        </ul>
    </td>
</tr>
<tr><td>**Beispiele**</td>
    <td>
        <ul>
            <li>Organigramme</li>
            <li>Social Graphs</li>
            <li>Betrugserkennung</li>
            <li>Analyse</li>
            <li>Empfehlungsmodule</li>
        </ul>
    </td>
</tr>
</table>

## <a name="column-family-databases"></a>Column-Family-Datenbanken

<table>
<tr><td>**Workload**</td>
    <td>
        <ul>
            <li>In den meisten Column-Family-Datenbanken werden Schreibvorgänge extrem schnell durchgeführt.</li>
            <li>Aktualisierungs- und Löschvorgänge sind selten.</li>
            <li>Bietet Zugriff mit hohem Durchsatz und niedriger Latenz.</li>
            <li>Unterstützt den einfachen Abfragezugriff auf eine bestimmte Gruppe von Feldern in einem wesentlich größeren Datensatz.</li>
            <li>Extrem skalierbar.</li>
        </ul>
    </td>
</tr>
<tr><td>**Datentyp**</td>
    <td>
        <ul>
            <li>Daten werden in Tabellen gespeichert, die aus einer Schlüsselspalte und einer oder mehreren Spaltenfamilien bestehen.</li>
            <li>Bestimmte Spalten können nach einzelnen Zeilen variieren.</li>
            <li>Der Zugriff auf einzelne Zellen erfolgt über get- und put-Befehle.</li>
            <li>Mehrere Zeilen werden unter Verwendung eines scan-Befehls zurückgegeben.</li>
        </ul>
    </td>
</tr>
<tr><td>**Beispiele**</td>
    <td>
        <ul>
            <li>Recommendations</li>
            <li>Personalisierung</li>
            <li>Sensordaten</li>
            <li>Telemetrie</li>
            <li>Nachrichten</li>
            <li>Analysen sozialer Medien</li>
            <li>Webanalysen</li>
            <li>Aktivitätsüberwachung</li>
            <li>Wetter- und andere Zeitreihendaten</li>
        </ul>
    </td>
</tr>
</table>

## <a name="search-engine-databases"></a>Suchmaschinen-Datenbanken

<table>
<tr><td>**Workload**</td>
    <td>
        <ul>
            <li>Indizierung von Daten aus mehreren Quellen und Diensten.</li>
            <li>Abfragen werden ad hoc durchgeführt und können komplex sein.</li>
            <li>Aggregation ist erforderlich.</li>
            <li>Volltextsuche ist erforderlich.</li>
            <li>Ad-hoc-Self-Service-Abfragen sind erforderlich.</li>
            <li>Datenanalyse mit Index für alle Felder ist erforderlich.</li>
        </ul>
    </td>
</tr>
<tr><td>**Datentyp**</td>
    <td>
        <ul>
            <li>Teilweise strukturiert oder unstrukturiert</li>
            <li>Text</li>
            <li>Text mit Verweis auf strukturierte Daten</li>
        </ul>
    </td>
</tr>
<tr><td>**Beispiele**</td>
    <td>
        <ul>
            <li>Produktkataloge</li>
            <li>Websitesuche</li>
            <li>Protokollierung</li>
            <li>Analyse</li>
            <li>Shopping-Websites</li>
        </ul>
    </td>
</tr>
</table>

## <a name="data-warehouse"></a>Data Warehouse

<table>
<tr><td>**Workload**</td>
    <td>
        <ul>
            <li>Datenanalysen</li>
            <li>Enterprise BI   </li>
        </ul>
    </td>
</tr>
<tr><td>**Datentyp**</td>
    <td>
        <ul>
            <li>Verlaufsdaten aus mehreren Quellen.</li>
            <li>In der Regel denormalisiert in einem „Stern“- oder „Schneeflocken“-Schema, das aus Fakten- und Dimensionstabellen besteht.</li>
            <li>Wird normalerweise mit neuen Daten auf Basis eines Zeitplans geladen.</li>
            <li>Dimensionstabellen enthalten häufig mehrere Verlaufsversionen einer Entität, die als *langsam veränderliche Dimension* bezeichnet wird.</li>
        </ul>
    </td>
</tr>
<tr><td>**Beispiele**</td>
    <td>Enterprise Data Warehouse, das Daten für analytische Modelle, Berichte und Dashboards umfasst
    </td>
</tr>
</table>


## <a name="time-series-databases"></a>Zeitreihendatenbanken

<table>
<tr><td>**Workload**</td>
    <td>
        <ul>
            <li>Der überwiegende Anteil der Vorgänge sind Schreibvorgänge (95 bis 99 %).</li>
            <li>Datensätze werden generell sequenziell in zeitlicher Reihenfolge angefügt.</li>
            <li>Aktualisierungen sind selten.</li>
            <li>Löschvorgänge werden massenweise und in zusammenhängenden Blöcken oder Datensätzen durchgeführt.</li>
            <li>Leseanforderungen können größer als der verfügbare Arbeitsspeicher sein.</li>
            <li>Üblicherweise erfolgen mehrere Lesevorgänge gleichzeitig.</li>
            <li>Daten werden sequenziell in aufsteigender oder absteigender zeitlicher Reihenfolge gelesen.</li>
        </ul>
    </td>
</tr>
<tr><td>**Datentyp**</td>
    <td>
        <ul>
            <li>Ein Zeitstempel wird als primärer Schlüssel und Sortiermechanismus verwendet.</li>
            <li>Messungen ab dem Eintrag oder Beschreibungen des Eintrags.</li>
            <li>Mit Tags werden zusätzliche Informationen zum Typ oder Ursprung sowie andere Informationen zum Eintrag definiert.</li>
        </ul>
    </td>
</tr>
<tr><td>**Beispiele**</td>
    <td>
        <ul>
            <li>Überwachungs- und Ereignistelemetrie</li>
            <li>Sensor- oder andere IoT-Daten</li>
        </ul>
    </td>
</tr>
</table>

## <a name="object-storage"></a>Objektspeicher

<table>
<tr><td>**Workload**</td>
    <td>
        <ul>
            <li>Identifizierung erfolgt nach Schlüssel.</li>
            <li>Auf Objekte kann öffentlich oder privat zugegriffen werden.</li>
            <li>Inhalte sind normalerweise Ressourcen, z.B. eine Tabelle, ein Bild oder eine Videodatei.</li>
            <li>Inhalte müssen dauerhaft (persistent) sein und sich außerhalb von Anwendungsebenen oder virtuellen Computern befinden.</li>
        </ul>
    </td>
</tr>
<tr><td>**Datentyp**</td>
    <td>
        <ul>
            <li>Die Datengröße ist umfangreich.</li>
            <li>Blobdaten</li>
            <li>Wert ist nicht transparent.</li>
        </ul>
    </td>
</tr>
<tr><td>**Beispiele**</td>
    <td>
        <ul>
            <li>Bilder, Videos, Office-Dokumente, PDF-Dateien</li>
            <li>CSS, Skripts, CSV</li>
            <li>Statisches HTML, JSON</li>
            <li>Protokoll- und Überwachungsdateien</li>
            <li>Datenbanksicherungen</li>
        </ul>
    </td>
</tr>
</table>

## <a name="shared-files"></a>Freigegebene Dateien

<table>
<tr><td>**Workload**</td>
    <td>
        <ul>
            <li>Migration aus vorhandenen Apps, die mit dem Dateisystem interagieren.</li>
            <li>Erfordert eine SMB-Schnittstelle.</li>
        </ul>
    </td>
</tr>
<tr><td>**Datentyp**</td>
    <td>
        <ul>
            <li>Dateien in einer hierarchischen Gruppe von Ordnern.</li>
            <li>Zugänglich über E/A-Standardbibliotheken.</li>
        </ul>
    </td>
</tr>
<tr><td>**Beispiele**</td>
    <td>
        <ul>
            <li>Legacydateien</li>
            <li>Freigegebener Inhalt zugänglich innerhalb verschiedener virtueller Computer oder App-Instanzen</li>
        </ul>
    </td>
</tr>
</table>
