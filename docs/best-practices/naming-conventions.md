---
title: "Namenskonventionen für Azure-Ressourcen"
description: "Enthält die Namenskonventionen für Azure-Ressourcen. Benennen von virtuellen Computern, Speicherkonten, Netzwerken, virtuellen Netzwerken, Subnetzen und anderen Azure-Entitäten"
author: telmosampaio
ms.date: 05/18/2017
pnp.series.title: Best Practices
ms.openlocfilehash: 5084fc2ba5a18707de1213276111c53203b6cdd7
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="naming-conventions"></a>Namenskonventionen

[!INCLUDE [header](../_includes/header.md)]

Dieser Artikel enthält eine Zusammenfassung der Benennungsregeln und -einschränkungen für Azure-Ressourcen und eine Reihe von grundsätzlichen Empfehlungen für Namenskonventionen.  Sie können diese Empfehlungen als Ausgangspunkt für Ihre eigenen, Ihren Bedürfnissen angepassten Konventionen verwenden.

Die Auswahl eines Namens für eine Ressource in Microsoft Azure ist auf folgenden Gründen wichtig:

* Es ist schwierig, einen Namen zu einem späteren Zeitpunkt zu ändern.
* Namen müssen die Anforderungen ihres bestimmten Ressourcentyps erfüllen.

Durch konsistente Namenskonventionen lassen sich Ressourcen einfacher finden. Außerdem kann damit die Rolle einer Ressource in einer Lösung angegeben werden.

Der Schlüssel zur erfolgreichen Verwendung von Namenskonventionen ist deren Einrichtung und Befolgung für Ihre gesamten Anwendungen und Organisationen.

## <a name="naming-subscriptions"></a>Benennen von Abonnements
Bei der Benennung von Azure-Abonnements verdeutlichen ausführliche Namen den Kontext und den Zweck jedes Abonnements.  Wenn Sie in einer Umgebung mit vielen Abonnements arbeiten, kann die Einhaltung einer gemeinsamen Namenskonvention die Eindeutigkeit verbessern.

Ein empfohlenes Muster für das Benennen von Abonnements:

`<Company> <Department (optional)> <Product Line (optional)> <Environment>`

* Das Unternehmen ist für jedes Abonnement normalerweise gleich. Einige Unternehmen verfügen in ihrer Organisationsstruktur aber ggf. über untergeordnete Unternehmen. Diese Unternehmen werden unter Umständen von einer zentralen IT-Gruppe verwaltet. In diesem Fall ist eine Unterscheidung möglich, indem sowohl der Name des übergeordneten Unternehmens (*Contoso*) als auch der Name des untergeordneten Unternehmens (*Northwind*) verwendet wird.
* Die Abteilung ist ein Name für einen Bereich in der Organisation, in der eine Gruppe von Personen arbeitet. Dieses Element ist innerhalb des Namespace optional.
* Die Produktlinie bezeichnet den spezifischen Namen eines Produkts oder einer Funktion, die von der Abteilung aus ausgeführt wird. Dies ist für interne Dienste und Anwendungen normalerweise optional. Der Einsatz wird aber dringend für öffentliche Dienste empfohlen, für die eine einfache Trennung und Identifikation (z.B. zur eindeutigen Trennung für Abrechnungsdatensätze) erforderlich ist.
* Die Umgebung ist der Name für den Bereitstellungszyklus der Anwendungen oder Dienste wie Entwicklung, QA oder Bereitstellung.

| Unternehmen | Abteilung | Produktlinie oder Dienst | Environment | Vollständiger Name |
| --- | --- | --- | --- | --- |
| Contoso |SocialGaming |AwesomeService |Bereitstellung |Contoso SocialGaming AwesomeService Bereitstellung |
| Contoso |SocialGaming |AwesomeService |Entwicklung |Contoso SocialGaming AwesomeService Entwicklung |
| Contoso |IT |InternalApps |Bereitstellung |Contoso IT InternalApps Bereitstellung |
| Contoso |IT |InternalApps |Entwicklung |Contoso IT InternalApps Entwicklung |

Weitere Informationen zur Organisation von Abonnements für größere Unternehmen finden Sie unter [Azure-Unternehmensgerüst – präskriptive Abonnementgovernance][scaffold].

## <a name="use-affixes-to-avoid-ambiguity"></a>Verwenden von Affixen zur Vermeidung von Mehrdeutigkeit

Beim Benennen von Ressourcen in Azure wird empfohlen, gängige Präfixe oder Suffixe zu verwenden, um den Typ und Kontext der Ressource zu identifizieren.  Während alle Informationen zum Typ, zu Metadaten und zum Kontext programmgesteuert verfügbar sind, vereinfacht das Anwenden gemeinsamer Affixe die visuelle Identifizierung.  Wenn Sie Affixe in Ihre Namenskonvention integrieren, ist es wichtig, eindeutig festzulegen, ob das Affix am Anfang (Präfix) oder am Ende (Suffix) des Namens steht.

Hier sind zwei mögliche Namen für einen Dienst angegeben, der ein Berechnungsmodul hostet:

* SvcCalculationEngine (Präfix)
* CalculationEngineSvc (Suffix)

Affixe können auf verschiedene Aspekte verweisen, die die entsprechenden Ressourcen beschreiben. Die folgende Tabelle zeigt einige üblicherweise verwendete Beispiele.

| Aspekt | Beispiel | Hinweise |
| --- | --- | --- |
| Environment |dev, prod, QA |identifiziert die Umgebung für die Ressource |
| Standort |uw (USA, Westen), ue (USA, Osten) |identifiziert die Region, in welcher die Ressource bereitgestellt wird |
| Instanz |01, 02 |für Ressourcen, die mehr als eine benannte Instanz besitzen (Webdienste usw.) |
| Produkt oder Dienst |service |identifiziert das Element (Produkt, Anwendung oder Dienst), das von der Ressource unterstützt wird |
| Rolle |sql, web, messaging |identifiziert die Rolle der zugeordneten Ressource |

Bei der Entwicklung einer bestimmten Namenskonvention für Ihr Unternehmen oder Ihre Projekte ist es wichtig, allgemeine Affixe und deren Position (Suffix oder Präfix) auszuwählen.

## <a name="naming-rules-and-restrictions"></a>Benennungsregeln und -einschränkungen

Mit jedem Ressourcen- oder Diensttyp in Azure wird eine Reihe von Benennungseinschränkungen und der entsprechende Umfang durchgesetzt. Alle Namenskonventionen bzw. -muster müssen die erforderlichen Benennungsregeln bzw. den Umfang erfüllen.  Beispiel: Während der Name einer VM einem DNS-Namen zugeordnet ist (und deshalb in der gesamten Azure-Umgebung eindeutig sein muss), ist der Name eines VNET auf die Ressourcengruppe beschränkt, in der es erstellt wird.

Vermeiden Sie es, Sonderzeichen (`-` oder `_`) als erstes oder letztes Zeichen eines Namens zu verwenden. Diese Zeichen führen bei den meisten Validierungsregeln zu einem Fehler.

| Category (Kategorie) | Dienst oder Entität | Scope | Länge | Schreibweise | Gültige Zeichen | Vorgeschlagenes Muster | Beispiel |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Ressourcengruppe |Ressourcengruppe |Global |1-64 |Groß-/Kleinschreibung nicht beachten |Alphanumerisch, Unterstrich, Klammern, Bindestrich, Punkt (außer am Ende) |`<service short name>-<environment>-rg` |`profx-prod-rg` |
| Ressourcengruppe |Verfügbarkeitsgruppe |Ressourcengruppe |1-80 |Groß-/Kleinschreibung nicht beachten |Alphanumerisch, Unterstrich und Bindestrich |`<service-short-name>-<context>-as` |`profx-sql-as` |
| Allgemein |Tag |Zugeordnete Entität |512 (Name), 256 (Wert) |Groß-/Kleinschreibung nicht beachten |Alphanumerisch |`"key" : "value"` |`"department" : "Central IT"` |
| Compute |Virtual Machine |Ressourcengruppe |1 - 15 (Windows), 1 - 64 (Linux) |Groß-/Kleinschreibung nicht beachten |Alphanumerisch, Unterstrich und Bindestrich |`<name>-<role>-vm<number>` |`profx-sql-vm1` |
| Compute |Funktionen-App | Global |1 - 60 |Groß-/Kleinschreibung nicht beachten |Alphanumerisch und Bindestrich |`<name>-func` |`calcprofit-func` |
| Speicher |Speicherkontoname (Daten) |Global |3-24 |Kleinbuchstaben |Alphanumerisch |`<globally unique name><number>` (Funktion verwenden, um eine eindeutige GUID für die Benennung von Speicherkonten zu berechnen) |`profxdata001` |
| Speicher |Speicherkontoname (Datenträger) |Global |3-24 |Kleinbuchstaben |Alphanumerisch |`<vm name without dashes>st<number>` |`profxsql001st0` |
| Speicher | Containername |Speicherkonto |3-63 |Kleinbuchstaben |Alphanumerisch und Bindestrich |`<context>` |`logs` |
| Speicher |Blobname | Container |1-1024 |Groß-/Kleinschreibung beachten |Jedes URL-Zeichen |`<variable based on blob usage>` |`<variable based on blob usage>` |
| Speicher |Warteschlangenname |Speicherkonto |3-63 |Kleinbuchstaben |Alphanumerisch und Bindestrich |`<service short name>-<context>-<num>` |`awesomeservice-messages-001` |
| Speicher |Tabellenname | Speicherkonto |3-63 |Groß-/Kleinschreibung nicht beachten |Alphanumerisch |`<service short name><context>` |`awesomeservicelogs` |
| Speicher |Dateiname | Speicherkonto |3-63 |Kleinbuchstaben | Alphanumerisch |`<variable based on blob usage>` |`<variable based on blob usage>` |
| Speicher |Data Lake-Speicher | Global |3-24 |Kleinbuchstaben | Alphanumerisch |`<name>-dtl` |`telemetry-dtl` |
| Netzwerk |Virtual Network (VNet) |Ressourcengruppe |2 - 64 |Groß-/Kleinschreibung muss nicht beachtet werden. |Alphanumerisch, Bindestrich, Unterstrich und Punkt |`<service short name>-vnet` |`profx-vnet` |
| Netzwerk |Subnetz |Übergeordnetes VNet |2-80 |Groß-/Kleinschreibung nicht beachten |Alphanumerisch, Unterstrich, Bindestrich und Punkt |`<descriptive context>` |`web` |
| Netzwerk |Netzwerkschnittstelle |Ressourcengruppe |1-80 |Groß-/Kleinschreibung muss nicht beachtet werden. |Alphanumerisch, Bindestrich, Unterstrich und Punkt |`<vmname>-nic<num>` |`profx-sql1-nic1` |
| Netzwerk |Netzwerksicherheitsgruppen (NSG) |Ressourcengruppe |1-80 |Groß-/Kleinschreibung muss nicht beachtet werden. |Alphanumerisch, Bindestrich, Unterstrich und Punkt |`<service short name>-<context>-nsg` |`profx-app-nsg` |
| Netzwerk |Netzwerksicherheitsgruppen-Regel |Ressourcengruppe |1-80 |Groß-/Kleinschreibung muss nicht beachtet werden. |Alphanumerisch, Bindestrich, Unterstrich und Punkt |`<descriptive context>` |`sql-allow` |
| Netzwerk |Öffentliche IP-Adresse |Ressourcengruppe |1-80 |Groß-/Kleinschreibung muss nicht beachtet werden. |Alphanumerisch, Bindestrich, Unterstrich und Punkt |`<vm or service name>-pip` |`profx-sql1-pip` |
| Netzwerk |Lastenausgleichsmodul |Ressourcengruppe |1-80 |Groß-/Kleinschreibung muss nicht beachtet werden. |Alphanumerisch, Bindestrich, Unterstrich und Punkt |`<service or role>-lb` |`profx-lb` |
| Netzwerk |Konfiguration der Regeln für die Lastenverteilung |Lastenausgleichsmodul |1-80 |Groß-/Kleinschreibung muss nicht beachtet werden. |Alphanumerisch, Bindestrich, Unterstrich und Punkt |`<descriptive context>` |`http` |
| Netzwerk |Azure Application Gateway |Ressourcengruppe |1-80 |Groß-/Kleinschreibung muss nicht beachtet werden. |Alphanumerisch, Bindestrich, Unterstrich und Punkt |`<service or role>-agw` |`profx-agw` |
| Netzwerk |Traffic Manager-Profil |Ressourcengruppe |1 - 63 |Groß-/Kleinschreibung muss nicht beachtet werden. |Alphanumerisch, Bindestrich und Punkt |`<descriptive context>` |`app1` |

> [!NOTE]
> Virtuelle Computer in Azure haben zwei getrennte Namen: VM-Name und Hostname. Wenn Sie im Portal eine VM erstellen, wird der gleiche Name sowohl für den Hostnamen als auch für den Namen der VM-Ressource verwendet. Die obigen Einschränkungen gelten für den Hostnamen. Der eigentliche Ressourcenname kann bis zu 64 Zeichen lang sein.

Microsoft fügt Azure häufig neue Dienste hinzu. Die obige Tabelle enthält die am häufigsten verwendeten Dienste für die Bereiche Netzwerk, Compute und Speicher. Erwägen Sie für andere Dienste die Verwendung eines Suffixes mit drei Buchstaben.

## <a name="organizing-resources-with-tags"></a>Organisieren von Ressourcen mit Tags

Azure Resource Manager unterstützt das Kennzeichnen von Entitäten mit beliebigen Textzeichenfolgen, um den Kontext zu identifizieren und die Automatisierung zu optimieren.  Mit dem Tag `"sqlVersion: "sql2014ee"` können beispielsweise VMs in einer Bereitstellung mit SQL Server 2014 Enterprise Edition für die Ausführung eines automatisierten Skripts identifiziert werden.  Tags sollten zusätzlich zu den gewählten Namenskonventionen zum Verbessern und Erweitern des Kontexts verwendet werden.

> [!TIP]
> Ein weiterer Vorteil von Tags ist, dass sie sich über Ressourcengruppen erstrecken, wodurch Sie die Möglichkeit haben, Entitäten über verschiedene Bereitstellungen hinweg zu verknüpfen und zu korrelieren.

Jede Ressource oder Ressourcengruppe kann maximal **15** Tags haben. Der Tagname ist auf 512 Zeichen beschränkt und der Tagwert auf 256 Zeichen.

Weitere Informationen zur Verwendung von Tags für Ressourcen finden Sie unter [Verwenden von Tags zum Organisieren von Azure-Ressourcen](/azure/azure-resource-manager/resource-group-using-tags/).

Einige der üblichen Anwendungsfälle für die Verwendung von Tags sind:

* **Abrechnung**– Gruppieren von Ressourcen und Zuordnen dieser zu Codes für Abrechnung oder verbrauchsbasierte Kostenzuteilung
* **ServicekontextIdentifikation**– Identifizieren von Gruppen von Ressourcen über Ressourcengruppen hinweg für allgemeine Operationen und Gruppierung
* **Zugriffssteuerung und Sicherheitskontext** – Identifizieren der Administratorrolle basierend auf Portfolio, System, Service, App, Instanz usw.

> [!TIP]
> Markieren Sie frühzeitig – markieren Sie oft.  Es ist besser, ein Basisschema zum Verwenden von Tags zu haben, das Sie mit der Zeit anpassen, als nachträglich eines erstellen zu müssen.

Ein Beispiel für einige übliche Ansätze zum Verwenden von Tags:

| Tag-Name | Schlüssel | Beispiel | Kommentar |
| --- | --- | --- | --- |
| Rechnung an / ID für interne verbrauchsbasierte Kostenzuteilung |billTo |`IT-Chargeback-1234` |Ein interner E/A- oder Abrechnungscode |
| zuständige Person (DRI, Directly Responsible Individual) |managedBy |`joe@contoso.com` |Alias oder E-Mail-Adresse |
| Projektname |project-name |`myproject` |Name des Projekts oder der Produktlinie |
| Projektversion |project-version |`3.4` |Version des Projekts oder der Produktlinie |
| Environment |Environment |`<Production, Staging, QA >` |Umgebungsbezeichner |
| Ebene |Ebene |`Front End, Back End, Data` |Ebenen- oder Rollen-/Kontextidentifikation |
| Datenprofil |dataProfile |`Public, Confidential, Restricted, Internal` |Vertraulichkeit der in der Ressource gespeicherten Daten |

## <a name="tips-and-tricks"></a>Tipps und Tricks

Für einige Arten von Ressourcen ist in Bezug auf die Benennung und Konventionen ggf. besondere Sorgfalt erforderlich.

### <a name="virtual-machines"></a>Virtuelle Computer

Vor allem in größeren Topologien vereinfachen sorgfältig benannte virtuelle Computer die Identifizierung der Rolle und des Zwecks jedes Computers und ermöglichen eine besser vorhersagbare Skripterstellung.

### <a name="storage-accounts-and-storage-entities"></a>Speicherkonten und -entitäten

Es gibt zwei vorrangige Anwendungsfälle für Speicherkonten: Sicherung von Datenträgern für VMs und Speicherung von Daten in Blobs, Warteschlangen und Tabellen.  Bei für VM-Datenträger verwendeten Speicherkonten sollte als Namenskonvention die Zuordnung zum Namen der übergeordneten VM genutzt werden (und außerdem ein Zahlensuffix hinzugefügt werden, da für Highend-VM-SKUs häufig mehrere Speicherkonten benötigt werden).

> [!TIP]
> Für Speicherkonten – ob für Daten oder Datenträger – sollte eine Namenskonvention gewählt werden, die die Nutzung von mehreren Speicherkonten ermöglicht (also immer mit numerischem Suffix).

Es ist möglich, einen benutzerdefinierten Domänennamen zu konfigurieren, mit dem Sie auf Blobdaten in Ihrem Azure Storage-Konto zugreifen können. Der Standardendpunkt für den Blob-Dienst lautet „https://<name>.blob.core.windows.net“.

Aber wenn Sie dem Blobendpunkt für Ihr Speicherkonto eine benutzerdefinierte Domäne zuordnen (z.B. www.contoso.com), können Sie auch unter Verwendung dieser Domäne auf Blobdaten in Ihrem Speicherkonto zugreifen. Mit einem benutzerdefinierten Domänennamen kann auf `http://mystorage.blob.core.windows.net/mycontainer/myblob` beispielsweise mit `http://www.contoso.com/mycontainer/myblob` zugegriffen werden.

Weitere Informationen zum Konfigurieren dieser Funktion finden Sie unter [Konfigurieren eines benutzerdefinierten Domänennamens für Ihren Blob Storage-Endpunkt](/azure/storage/storage-custom-domain-name/).

Weitere Informationen zur Benennung von Blobs, Containern und Tabellen finden Sie in der folgenden Liste:

* [Benennen von Containern, BLOBs und Metadaten und Verweisen auf diese](https://msdn.microsoft.com/library/dd135715.aspx)
* [Benennen von Warteschlangen und Metadaten](https://msdn.microsoft.com/library/dd179349.aspx)
* [Benennen von Tabellen](https://msdn.microsoft.com/library/azure/dd179338.aspx)

Ein Blobname kann jede Kombination von Zeichen enthalten, jedoch müssen reservierte URL-Zeichen ordnungsgemäß mit Escapezeichen versehen werden. Vermeiden Sie Blobnamen, die mit einem Punkt (.), einem Schrägstrich (/) oder einer Sequenz oder einer Kombination aus beidem endet. Gemäß der Konvention ist der Schrägstrich das Trennzeichen für das **virtuelle** Verzeichnis. Verwenden Sie in einem Blobnamen keinen umgekehrten Schrägstrich (\\). Die Client-APIs lassen dies möglicherweise zu, ermitteln dann aber den Hash nicht richtig, und die Signaturen stimmen nicht überein.

Es ist nicht möglich, den Namen eines Speicherkontos oder -containers nach dem Erstellen zu ändern. Wenn Sie einen neuen Namen verwenden möchten, müssen Sie die Komponente löschen und neu erstellen.

> [!TIP]
> Es wird empfohlen, dass Sie vor der Entwicklung eines neuen Diensts oder einer neuen Anwendung eine Namenskonvention für alle Speicherkonten und -typen einrichten.

<!-- links -->

[scaffold]: /azure/azure-resource-manager/resource-manager-subscription-governance
