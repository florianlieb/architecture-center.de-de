---
title: Integrieren von lokalen AD-Domänen in Azure Active Directory
description: Erfahren Sie, wie Sie eine sichere hybride Netzwerkarchitektur mit Azure Active Directory implementieren.
author: telmosampaio
pnp.series.title: Identity management
ms.date: 11/28/2016
pnp.series.next: adds-extend-domain
pnp.series.prev: ./index
cardTitle: Integrate on-premises AD with Azure AD
ms.openlocfilehash: 431de4b2e08c79f70cc9830fda8315e07bf22c64
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/30/2018
---
# <a name="integrate-on-premises-active-directory-domains-with-azure-active-directory"></a>Integrieren von lokalen Active Directory-Domänen in Azure Active Directory

Azure Active Directory (Azure AD) ist ein cloudbasierter mehrinstanzenfähiger Verzeichnis- und Identitätsdienst. Die folgende Referenzarchitektur zeigt bewährte Methoden zum Integrieren von lokalen Active Directory-Domänen in Azure AD, um cloudbasierte Identitätsauthentifizierung bereitzustellen. [**So stellen Sie diese Lösung bereit**.](#deploy-the-solution)

[![0]][0] 

*Laden Sie eine [Visio-Datei][visio-download] mit dieser Architektur herunter.*

> [!NOTE]
> Der Einfachheit halber sind in diesem Diagramm nur die Verbindungen dargestellt, die sich direkt auf Azure AD beziehen. Protokollbezogener Datenverkehr, der im Rahmen des Authentifizierungs- und Identitätsverbunds auftreten kann, ist dagegen nicht dargestellt. So kann es beispielsweise sein, dass eine Webanwendung den Webbrowser umleitet, damit die Anforderung über Azure AD authentifiziert wird. Sobald die Anforderung authentifiziert ist, kann sie mit den entsprechenden Identitätsinformationen an die Webanwendung zurückgegeben werden.
> 

Typische Einsatzmöglichkeiten für diese Referenzarchitektur sind:

* Webanwendungen, die in Azure bereitgestellt werden und Zugriff für Remotebenutzer bereitstellen, die zu Ihrer Organisation gehören.
* Implementieren von Self-Service-Funktionen für Endbenutzer, z.B. Zurücksetzen ihrer Kennwörter und Delegieren von Gruppenverwaltung. Beachten Sie, dass hierfür eine Azure AD Premium-Edition erforderlich ist.
* Architekturen, in denen das lokale Netzwerk und das Azure VNet der Anwendung nicht über eine VPN-Tunnel- oder ExpressRoute-Verbindung verbunden sind.

> [!NOTE]
> Azure AD unterstützt derzeit nur Benutzerauthentifizierung. Einige Anwendungen und Dienste, z. B. SQL Server, erfordern möglicherweise Computerauthentifizierung. In diesem Fall ist diese Lösung nicht geeignet.
> 

Weitere Überlegungen finden Sie unter [Auswählen einer Lösung für die Integration einer lokalen Active Directory-Instanz in Azure][considerations]. 

## <a name="architecture"></a>Architecture

Diese Architektur besteht aus den folgenden Komponenten.

* **Azure AD-Mandant**. Eine Instanz von [Azure AD][azure-active-directory], die von Ihrer Organisation erstellt wurde. Sie fungiert als ein Verzeichnisdienst für Cloudanwendungen, indem sie Objekte speichert, die aus dem lokalen Active Directory kopiert wurden, und stellt Identitätsdienste bereit.
* **Webschicht-Subnetz**. Dieses Subnetz enthält virtuelle Computer, die eine Webanwendung ausführen. Azure AD kann als Identitätsbroker für diese Anwendung fungieren.
* **Lokaler AD DS-Server**. Eine lokaler Verzeichnis- und Identitätsdienst. Das AD DS-Verzeichnis kann mit Azure AD synchronisiert werden, sodass Azure AD lokale Benutzer authentifizieren kann.
* **Azure AD Connect-Synchronisierungsserver**. Ein lokaler Computer, auf dem der [Azure AD Connect][azure-ad-connect]-Synchronisierungsdienst ausgeführt wird. Dieser Dienst synchronisiert Informationen, die sich im lokalen Active Directory befinden, mit Azure AD. Wenn Sie beispielsweise die Bereitstellung oder das Aufheben der Bereitstellung von Gruppen und Benutzern lokal ausführen, werden diese Änderungen an Azure AD weitergegeben. 
  
  > [!NOTE]
  > Aus Sicherheitsgründen speichert Azure AD die Kennwörter eines Benutzers als Hash. Ist für einen Benutzer eine Kennwortzurücksetzung erforderlich, muss diese lokal ausgeführt und der neue Hash an Azure AD gesendet werden. Azure AD Premium-Editionen enthalten Funktionen, mit denen diese Aufgabe automatisiert werden kann, sodass Benutzern in die Lage versetzt werden, ihre eigenen Kennwörter zurückzusetzen.
  > 

* **Virtuelle Computer (VMs) für eine n-schichtige Anwendung**. Die Bereitstellung beinhaltet eine Infrastruktur für eine n-schichtige Anwendung. Weitere Informationen zu diesen Ressourcen finden Sie unter [Ausführen von Windows-VMs für eine n-schichtige Anwendung][implementing-a-multi-tier-architecture-on-Azure].

## <a name="recommendations"></a>Empfehlungen

Die folgenden Empfehlungen gelten für die meisten Szenarios. Sofern Sie keine besonderen Anforderungen haben, die Vorrang haben, sollten Sie diese Empfehlungen befolgen. 

### <a name="azure-ad-connect-sync-service"></a>Azure AD Connect-Synchronisierungsdienst

Der Azure AD Connect-Synchronisierungsdienst stellt sicher, dass Identitätsinformationen, die in der Cloud gespeichert sind, mit denjenigen konsistent sind, die lokal gespeichert sind. Sie installieren diesen Dienst mit der Azure AD Connect-Software. 

Bevor Sie vor Azure AD Connect-Synchronisierung implementieren, müssen Sie die Synchronisierungsanforderungen Ihrer Organisation ermitteln. Dies umfasst beispielsweise, was, von welchen Domänen und wie oft synchronisiert werden muss. Weitere Informationen hierzu finden Sie unter [Ermitteln der Anforderungen an die Verzeichnissynchronisierung][aad-sync-requirements].

Sie können den Azure AD Connect-Synchronisierungsdienst auf einem virtuellen Computer oder auf einem lokal gehosteten Computer ausführen. Je nach Schwankung der Informationen in Ihrem Active Directory-Verzeichnis ist es unwahrscheinlich, dass die Last für den Azure AD Connect-Synchronisierungsdienst nach der Erstsynchronisierung mit Azure AD hoch ist. Durch Ausführen des Diensts auf einem virtuellen Computer wird es einfacher, den Server bei Bedarf zu skalieren. Überwachen Sie die Aktivitäten auf dem virtuellen Computer, wie dies im Abschnitt „Überwachung“ beschrieben ist, um zu ermitteln, ob Skalierung erforderlich ist.

Wenn Sie mehrere lokale Domänen in einer Gesamtstruktur haben, empfiehlt es sich, die Informationen für die vollständige Gesamtstruktur im einem einzigen Azure AD-Mandanten zu speichern und zu synchronisieren. Filtern Sie nach Informationen für Identitäten, die in mehreren Domänen vorkommen, sodass jede Identität nur einmal in Azure AD vorhanden ist, statt dupliziert zu werden. Duplizierung kann zu Inkonsistenzen führen, wenn Daten synchronisiert werden. Weitere Informationen finden Sie im Abschnitt „Topologieempfehlungen“ weiter unten. 

Verwenden Sie Filterung, damit nur erforderliche Daten in Azure AD gespeichert werden. Beispielsweise könnte es sein, dass Ihre Organisation keine Informationen über inaktive Konten in Azure AD speichern möchte. Filterung kann nach Gruppen, Domänen, Organisationseinheiten oder Attributen erfolgen. Sie können Filter kombinieren, um komplexere Regeln zu erzeugen. Beispielsweise könnten Sie Objekte synchronisieren, die in einer Domäne gespeichert sind und einen bestimmten Wert in einem ausgewählten Attribut haben. Ausführliche Informationen hierzu finden Sie unter [Azure AD Connect-Synchronisierung: Konfigurieren der Filterung][aad-filtering].

Um hohe Verfügbarkeit für den AD Connect-Synchronisierungsdienst sicherzustellen, führen Sie einen sekundären Stagingserver aus. Weitere Informationen finden Sie in Abschnitt „Topologieempfehlungen“.

### <a name="security-recommendations"></a>Sicherheitsempfehlungen

**Verwaltung von Benutzerkennwörtern.** Die Azure AD Premium-Editionen unterstützen das Rückschreiben von Kennwörtern, sodass Ihre lokalen Benutzer die Möglichkeit haben, Self-Service-Kennwortzurücksetzungen über das Azure-Portal auszuführen. Dieses Feature sollte erst aktiviert werden, nachdem die Kennwortsicherheitsrichtlinien Ihrer Organisation überprüft wurden. Beispielsweise können Sie einschränken, welche Benutzer ihre Kennwörter ändern können, und Sie können die Oberfläche für die Kennwortverwaltung anpassen. Weitere Informationen hierzu finden Sie unter [Anpassen der Kennwortverwaltung an die Anforderungen Ihrer Organisation][aad-password-management]. 

**Schützen von lokalen Anwendungen, auf die extern zugegriffen werden kann.** Verwenden Sie den Azure AD-Anwendungsproxy, um für externe Benutzer über Azure AD kontrollierten Zugriff auf lokale Webanwendungen bereitzustellen. Nur Benutzer, die gültige Anmeldeinformationen in Ihrem Azure-Verzeichnis haben, sind berechtigt, die Anwendung zu verwenden. Weitere Informationen hierzu finden Sie im Artikel [Aktivieren des Anwendungsproxys über das Azure-Portal][aad-application-proxy].

**Aktive Überwachung von Azure AD auf Anzeichen für verdächtige Aktivitäten.**    Sie sollten erwägen, die Azure AD Premium P2-Edition zu verwenden, die Azure AD Identity Protection umfasst. Identity Protection nutzt adaptive Machine Learning-Algorithmen und heuristische Verfahren, um Anomalien und Risikoereignisse zu erkennen, die auf die Kompromittierung einer Identität hindeuten können. Beispielsweise kann Identity Protection potenziell ungewöhnliche Aktivitäten erkennen, etwa auffällige Anmeldeaktivitäten, Anmeldungen von unbekannten Quellen oder über IP-Adressen mit verdächtigen Aktivitäten oder Anmeldungen von Geräten, die möglicherweise infiziert sind. Mit diesen Daten generiert Identity Protection Berichte und Warnungen, damit Sie diese Risikoereignisse untersuchen und geeignete Aktionen durchführen können. Weitere Informationen finden Sie unter [Azure Active Directory Identity Protection][aad-identity-protection].
  
Sie können die Berichtsfunktion von Azure AD im Azure-Portal verwenden, um sicherheitsbezogene Aktivitäten zu überwachen, die in Ihrem System auftreten. Weitere Informationen zum Verwenden dieser Berichte finden Sie unter [Anleitung für Azure Active Directory-Berichte][aad-reporting-guide].

### <a name="topology-recommendations"></a>Topologieempfehlungen

Konfigurieren Sie Azure AD Connect so, dass eine Topologie implementiert wird, die den Anforderungen Ihrer Organisation am ehesten entspricht. Folgende Topologien gehören zu den Topologien, die von Azure AD Connect unterstützt werden:

* **Einzelne Gesamtstruktur, einzelnes Azure AD-Verzeichnis**. In dieser Topologie synchronisiert Azure AD Connect Objekte und Identitätsinformationen aus einer oder mehreren Domänen in einer einzelnen lokalen Gesamtstruktur in einem einzelnen Azure AD-Mandanten. Dies ist die Standardtopologie, die durch die Expressinstallation von Azure AD Connect implementiert wird.
  
  > [!NOTE]
  > Sofern Sie keinen Server im Stagingmodus ausführen (weiter unten beschrieben), sollten Sie nicht mehrere Azure AD Connect-Synchronisierungsserver verwenden, um verschiedene Domänen in derselben lokalen Gesamtstruktur mit demselben Azure AD-Mandanten zu verbinden.
  > 
  > 

* **Mehrere Gesamtstrukturen, einzelnes Azure AD-Verzeichnis**. In dieser Topologie synchronisiert Azure AD Connect Objekte und Identitätsinformationen aus mehrere Gesamtstrukturen in einem einzelnen Azure AD-Mandanten. Verwenden Sie diese Topologie, wenn Ihre Organisation mehrere lokale Gesamtstrukturen hat. Sie können Identitätsinformationen konsolidieren, sodass jeder eindeutige Benutzer im Azure AD-Verzeichnis einmal dargestellt ist, selbst wenn er in mehreren Gesamtstrukturen vorhanden ist. Für alle Gesamtstrukturen wird derselbe Azure AD Connect-Synchronisierungsserver verwendet. Der Azure AD Connect-Synchronisierungsserver muss nicht zu einer der Domänen gehören, muss aber aus allen Gesamtstrukturen erreichbar sein.
  
  > [!NOTE]
  > In dieser Topologie dürfen Sie keine separaten Azure AD Connect-Synchronisierungsserver verwenden, um jede lokale Gesamtstruktur mit einem einzelnen Azure AD-Mandanten zu verbinden. Dies kann zu doppelten Identitätsinformationen in Azure AD führen, wenn Benutzer in mehreren Gesamtstrukturen vorhanden sind.
  > 
  > 

* **Mehrere Gesamtstrukturen, separate Topologien**. In dieser Topologie werden Identitätsinformationen aus separaten Gesamtstrukturen in einem einzelnen Azure AD-Mandanten zusammengeführt, wobei alle Gesamtstrukturen als separate Entitäten behandelt werden. Diese Topologie ist nützlich, wenn Sie Gesamtstrukturen aus verschiedenen Organisationen kombinieren und die Identitätsinformationen für jeden Benutzer nur in einer Gesamtstruktur gespeichert werden.
  
  > [!NOTE]
  > Wenn die globalen Adresslisten (GAL) in jeder Gesamtstruktur synchronisiert werden, kann ein Benutzer aus einer Gesamtstruktur in einer anderen als Kontakt vorhanden sein. Dies kann passieren, wenn Ihre Organisation GALSync mit Forefront Identity Manager 2010 oder Microsoft Identity Manager 2016 implementiert hat. In diesem Szenario können Sie angeben, dass Benutzer über ihr *Mail*-Attribut identifiziert werden sollen. Außerdem können Sie Identitäten mithilfe der Attribute *ObjectSID* und *MsExchMasterAccountSID* abgleichen. Dies ist nützlich, wenn Sie mindestens eine Ressourcengesamtstruktur mit deaktivierten Konten haben.
  > 
  > 

* **Stagingserver**. In dieser Konfiguration wird eine zweite Instanz des Azure AD Connect-Synchronisierungsservers parallel zur ersten ausgeführt. Diese Struktur unterstützt Szenarien wie die folgenden:
  
  * Hochverfügbarkeit.
  * Testen und Bereitstellen einer neuen Konfigurations des Azure AD Connect-Synchronisierungsservers.
  * Einführen eines neuen Servers und Außerbetriebnahme einer alten Konfiguration. 
    
    In diesen Szenarien wird die zweite Instanz im *Stagingmodus* ausgeführt. Der Server speichert importierte Objekte und Synchronisierungsdaten in seiner Datenbank, übergibt die Daten aber nicht an Azure AD. Wenn Sie den Stagingmodus deaktivieren, beginnt der Server damit, Daten in Azure AD zu schreiben sowie ggf. Kennwörter in die lokalen Verzeichnisse zurückzuschreiben. Weitere Informationen hierzu finden Sie unter [Azure AD Connect Sync: Operative Aufgaben und Überlegungen][aad-connect-sync-operational-tasks].

* **Mehrere Azure AD-Verzeichnisse**. Es wird empfohlen, dass Sie ein einzelnes Azure AD-Verzeichnis für eine Organisation erstellen, es kann aber Situationen geben, in denen Sie Informationen über getrennte Azure AD-Verzeichnisse verteilen müssen. In diesem Fall können Sie Synchronisierungs- und Kennwortrückschreibungsprobleme vermeiden, indem Sie sicherstellen, dass jedes Objekt aus der lokalen Gesamtstruktur nur einmal im Azure AD-Verzeichnis vorhanden ist. Um dieses Szenario zu implementieren, konfigurieren Sie separate Azure AD Connect-Synchronisierungsserver für jedes Azure AD-Verzeichnis, und verwenden Sie Filterung, sodass jeder Azure AD Connect-Synchronisierungsserver einen wechselseitig exklusiven Satz von Objekten verwaltet. 

Weitere Informationen zu diesen Topologien finden Sie unter [Topologien für Azure AD Connect][aad-topologies].

### <a name="user-authentication"></a>Benutzerauthentifizierung

Standardmäßig konfiguriert der Azure AD Connect-Synchronisierungsserver Kennwortsynchronisierung zwischen der lokalen Domäne und Azure AD, und für den Azure AD-Dienst wird davon ausgegangen, dass Benutzer sich authentifizieren, indem sie das Kennwort bereitstellen, dass sie lokal verwenden. Für viele Organisationen ist dies geeignet, Sie sollten aber die vorhandenen Richtlinien und die vorhandene Infrastruktur Ihres Unternehmens berücksichtigen. Beispiel: 

* Die Sicherheitsrichtlinie Ihrer Organisation könnte verhindern, dass Kennworthashes in der Cloud synchronisiert werden.
* Sie könnten fordern, dass für Benutzer nahtloses einmaligen Anmelden verwendet wird, wenn sie im Unternehmensnetzwerk von Computern, die zur Domäne gehören, auf Cloudressourcen zugreifen.
* Ihre Organisation hat möglicherweise bereits Active Directory-Verbunddienste (AD FS) oder einen Drittanbieter-Verbundanbieter bereitgestellt. Sie können Azure AD so konfigurieren, dass statt der in der Cloud gespeicherten Kennwortinformationen diese Infrastruktur zum Implementieren von Authentifizierung und SSO verwendet wird.

Weitere Informationen hierzu finden Sie unter [Azure AD Connect-Optionen für die Benutzeranmeldung][aad-user-sign-in].

### <a name="azure-ad-application-proxy"></a>Azure AD-Anwendungsproxy 

Verwenden Sie Azure AD, um Zugriff auf lokale Anwendungen bereitzustellen.

Machen Sie Ihre lokalen Webanwendungen über Anwendungsproxy-Connectors verfügbar, die von der Azure AD-Komponente für Anwendungsproxys verwaltet werden. Der Anwendungsproxy-Connector öffnet eine ausgehende Netzwerkverbindung zum Azure AD-Anwendungsproxy, und Anforderungen von Remotebenutzern werden von Azure AD über diese Verbindung zurück an die Web-Apps weitergeleitet. Dadurch entfällt die Notwendigkeit, eingehende Ports in der lokalen Firewall zu öffnen, wodurch die Angriffsfläche verkleinert wird, die sich für Ihre Organisation ergibt.

Weitere Informationen hierzu finden Sie unter [Veröffentlichen von Anwendungen mit Azure AD-Anwendungsproxy][aad-application-proxy].

### <a name="object-synchronization"></a>Objektsynchronisierung 

In der Standardkonfiguration synchronisiert Azure AD Connect Objekte aus Ihrem lokalen Active Directory-Verzeichnis anhand der Regeln, die im Artikel [Azure AD Connect-Synchronisierung: Grundlegendes zur Standardkonfiguration][aad-connect-sync-default-rules] angegeben sind. Objekte, die diese Regeln erfüllen, werden synchronisiert, während alle anderen Objekte ignoriert werden. Einige Beispielregeln:

* Benutzerobjekte benötigen ein eindeutiges *sourceAnchor*-Attribut, und das *accountEnabled*-Attribut muss einen Wert haben.
* Jedes Benutzerobjekt muss ein *sAMAccountName*-Attribut haben und darf weder mit dem Text *Azure AD_* noch mit dem Text *MSOL_* beginnen.

Azure AD Connect wendet verschiedene Regeln auf User-, Contact-, Group-, ForeignSecurityPrincipal- und Computer-Objekte an. Verwenden Sie den Synchronisierungsregel-Editor, der mit Azure AD Connect installiert wurde, wenn Sie den Standardsatz der Regeln ändern müssen. Weitere Informationen hierzu finden Sie unter [Azure AD Connect-Synchronisierung: Grundlegendes zur Standardkonfiguration][aad-connect-sync-default-rules].

Sie können auch Ihre eigenen Filter definieren, um die zu synchronisierenden Objekte nach Domäne oder Organisationseinheit zu begrenzen. Alternativ Sie können komplexere benutzerdefinierte Filter implementieren, z.B. einen Filter wie den, der unter [Azure AD Connect-Synchronisierung: Konfigurieren der Filterung][aad-filtering] beschrieben ist.

### <a name="monitoring"></a>Überwachung 

Systemüberwachung wird von den folgenden lokal installierten Agents ausgeführt:

* Azure AD Connect installiert einen Agent, der Informationen zu Synchronisierungsvorgängen erfasst. Verwenden Sie das Blatt „Azure AD Connect Health“ im Azure-Portal, um dessen Integrität und Leistung überwachen. Weitere Informationen hierzu finden Sie unter [Verwenden von Azure AD Connect Health für die Synchronisierung][aad-health].
* Wenn Sie die Integrität der AD DS-Domänen und -Verzeichnisse von Azure überwachen möchten, installieren Sie den Azure AD Connect Health für AD DS-Agent auf einem Computer in der lokalen Domäne. Verwenden Sie das Blatt „Azure Active Directory Connect Health“ im Azure-Portal für die Systemüberwachung. Weitere Informationen hierzu finden Sie unter [Verwenden von Azure AD Connect Health mit AD DS][aad-health-adds]. 
* Installieren Sie den Azure AD Connect Health für AD FS-Agent, um die Integrität von Diensten zu überwachen, die lokal ausgeführt werden, und verwenden Sie das Blatt „Azure Active Directory Connect Health“ im Azure-Portal, um AD FS zu überwachen. Weitere Informationen hierzu finden Sie unter [Verwenden von Azure AD Connect Health mit AD FS][aad-health-adfs].

Weitere Informationen zum Installieren von AD Connect Health-Agents und zu deren Anforderungen finden Sie unter [Installieren des Azure AD Connect Health-Agents][aad-agent-installation].

## <a name="scalability-considerations"></a>Überlegungen zur Skalierbarkeit

Azure AD-Dienst unterstützt Skalierbarkeit auf Basis von Replikaten, mit einem einzelnen primären Replikat, das Schreibvorgänge verwaltet, und mehreren schreibgeschützten sekundären Replikaten. Azure AD leitet versuchte Schreibvorgänge, die für sekundäre Replikate erfolgt sind, transparent an das primäre Replikat weiter und bietet so Konsistenz. Alle Änderungen, die am primären Replikat vorgenommen wurden, werden an die sekundären Replikate weitergegeben. Diese Architektur bietet eine gute Skalierung, denn die meisten Vorgänge für Azure AD sind keine Schreib-, sondern Lesevorgänge. Weitere Informationen hierzu finden Sie unter [Azure AD: Under the hood of our geo-redundant, highly available, distributed cloud directory][aad-scalability].

Ermitteln Sie für den Azure AD Connect-Synchronisierungsserver, wie viele Objekte aus Ihrem lokalen Verzeichnis wahrscheinlich zu synchronisieren sind. Sind dies weniger als 100.000 Objekte, können Sie die standardmäßige SQL Server Express LocalDB-Software verwenden, die mit Azure AD Connect bereitgestellt wird. Haben Sie eine größere Anzahl von Objekten, sollten Sie eine Produktionsversion von SQL Server installieren und eine benutzerdefinierte Installation von Azure AD Connect ausführen, wobei Sie angeben, dass eine vorhandene Instanz von SQL Server verwendet werden soll.

## <a name="availability-considerations"></a>Überlegungen zur Verfügbarkeit

Der Azure AD-Dienst ist geografisch verteilt und wird in mehreren Datencentern ausgeführt, die auf der ganzen Welt verteilt sind und automatisches Failover bieten. Fällt ein Datencenter aus, stellt Azure AD sicher, dass Ihre Verzeichnisdaten in mindestens zwei weiteren regional verteilten Datencentern für Zugriff durch die Instanz verfügbar sind.

> [!NOTE]
> Die Vereinbarung zum Servicelevel (SLA) für Azure AD Basic- und Premium-Dienste garantiert eine Verfügbarkeit von mindestens 99,9 %. Für den Free-Tarif von Azure AD gibt es keine SLA. Weitere Informationen hierzu finden Sie unter [SLA für Azure Active Directory][sla-aad].
> 
> 

Sie sollten sich überlegen, eine zweite Instanz des Azure AD Connect-Synchronisierungsservers im Stagingmodus bereitzustellen, um die Verfügbarkeit zu erhöhen, wie dies im Abschnitt „Topologieempfehlungen“ erläutert ist. 

Wenn Sie nicht die SQL Server Express LocalDB-Instanz verwenden, die zu Azure AD Connect gehört, sollten Sie SQL-Clustering verwenden, um hohe Verfügbarkeit zu erreichen. Lösungen wie Datenbankspiegelung und Always On werden von Azure AD Connect nicht unterstützt.

Weitere Informationen dazu, wie hohe Verfügbarkeit des Azure AD Connect-Synchronisierungsservers erreicht und wie ein Wiederherstellen nach einem Ausfall vorgenommen wird, finden Sie unter [Azure AD Connect Sync: Operative Aufgaben und Überlegungen – Notfallwiederherstellungl][aad-sync-disaster-recovery].

## <a name="manageability-considerations"></a>Überlegungen zur Verwaltbarkeit

Es gibt zwei Aspekte für das Verwalten von Azure AD:

* Verwalten von Azure AD in der Cloud
* Verwalten der Azure AD Connect-Synchronisierungsserver

Azure AD bietet die folgenden Optionen zum Verwalten von Domänen und Verzeichnissen in der Cloud: 

* **Azure Active Directory-PowerShell-Modul**. Verwenden Sie dieses [Modul][aad-powershell], wenn Sie allgemeine Azure AD-Verwaltungsaufgaben wie Benutzerverwaltung, Domänenverwaltung und Konfigurieren von einmaligem Anmelden mit Skripts ausführen müssen.
* **Blatt für Azure AD-Verwaltung im Azure-Portal**. Dieses Blatt enthält eine interaktive Verwaltungsansicht des Verzeichnisses und ermöglicht es Ihnen, die meisten Aspekte von Azure AD zu steuern und zu konfigurieren. 

Mit Azure AD Connect werden die folgenden Tools installiert, um Azure AD Connect-Synchronisierungsdienste über Ihre lokalen Computer verwalten zu können:
  
* **Microsoft Azure Active Directory Connect-Konsole** Mit diesem Tool können Sie die Konfiguration des Azure AD Sync-Servers ändern, anpassen, wie die Synchronisierung erfolgt, den Stagingmodus aktivieren oder deaktivieren und den Benutzeranmeldemodus wechseln. Beachten Sie, dass Sie Active Directory FS-Anmeldung mit Ihrer lokalen Infrastruktur aktivieren können.
* **Synchronization Service Manager**. Verwenden Sie die Registerkarte *Vorgänge* in diesem Tool, um den Synchronisierungsprozess zu verwalten und zu erkennen, ob irgendwelche Teile des Prozesses fehlgeschlagen sind. Mit diesem Tool können Sie Synchronisierungen manuell auslösen. Auf der Registerkarte *Connectors* können Sie die Verbindungen für die Domänen steuern, denen das Synchronisierungsmodul zugewiesen ist.
* **Synchronisierungsregel-Editor**. Mit diesem Tool können Sie die Art und Weise anpassen, wie Objekte transformiert werden, wenn sie zwischen einem lokalen Verzeichnis und Azure AD kopiert werden. Das Tool ermöglicht es Ihnen, weitere Attribute und Objekte für die Synchronisierung anzugeben und dann Filter auszuführen, mit denen bestimmt wird, welche Objekte synchronisiert oder nicht synchronisiert werden sollen. Weitere Informationen hierzu finden Sie im Dokument [Azure AD Connect-Synchronisierung: Grundlegendes zur Standardkonfiguration][aad-connect-sync-default-rules] im Abschnitt „Synchronisierungsregel-Editor“.

Weitere Informationen und Tipps zum Verwalten von Azure AD Connect finden Sie unter [Azure AD Connect-Synchronisierung: Bewährte Methoden zum Ändern der Standardkonfiguration][aad-sync-best-practices].

## <a name="security-considerations"></a>Sicherheitshinweise

Verwenden Sie bedingte Zugriffssteuerung, um Authentifizierungsanforderungen von unerwarteten Quellen abzulehnen:

- Lösen Sie [Azure Multi-Factor Authentication (MFA)][azure-multifactor-authentication] aus, wenn ein Benutzer versucht, eine Verbindung von einem nicht vertrauenswürdigen Standort herzustellen, etwa über das Internet anstelle eines vertrauenswürdigen Netzwerks.

- Verwenden Sie den Geräteplattformtyp des Benutzers (iOS, Android, Windows Mobile, Windows), um die Richtlinie für Zugriff auf Anwendungen und Funktionen zu bestimmen.

- Speichern Sie für die Geräte der Benutzer den Status „Aktiviert/Deaktiviert“, und integrieren Sie diese Informationen in die Zugriffsrichtlinienüberprüfungen. Hat ein Benutzer z.B. sein Smartphone verloren, oder wurde es ihm gestohlen, sollte es als „Deaktiviert“ gespeichert werden, um zu verhindern, dass mit ihm Zugriff erlangt werden kann.

- Steuern Sie Benutzerzugriff auf Ressourcen anhand von Gruppenmitgliedschaften. Verwenden Sie [Azure AD-Regeln für dynamische Mitgliedschaft][aad-dynamic-membership-rules], um eine Gruppenverwaltung zu vereinfachen. Eine kurze Übersicht, wie dies funktioniert, finden Sie unter [Introduction to Dynamic Memberships for Groups][aad-dynamic-memberships].

- Verwenden Sie Risikorichtlinien zum bedingten Zugriff mit Azure AD Identity Protection,um erweiterten Schutz basierend auf ungewöhnlichen Anmeldeaktivitäten oder anderen Ereignissen zu bieten.

Weitere Informationen hierzu finden Sie unter [Bedingter Zugriff mit Azure Active Directory][aad-conditional-access].

## <a name="deploy-the-solution"></a>Bereitstellen der Lösung

Eine Bereitstellung für eine Referenzarchitektur, die diese Empfehlungen und Überlegungen implementiert, steht auf GitHub zur Verfügung. In dieser Referenzarchitektur wird ein simuliertes lokales Netzwerk in Azure bereitgestellt, das Sie zum Testen und Experimentieren verwenden können. Die Referenzarchitektur kann sowohl mit Windows- als auch mit Linux-VMs bereitgestellt werden, indem entsprechend den folgenden Anweisungen vorgegangen wird: 

1. Klicken Sie auf diese Schaltfläche:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fidentity%2Fazure-ad%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Nachdem der Link im Azure-Portal geöffnet wurde, müssen Sie Werte für einige Einstellungen eingeben: 
   * Der Name der **Ressourcengruppe** ist bereits in der Parameterdatei definiert. Wählen Sie also **Neu erstellen**, und geben Sie im Textfeld `ra-aad-onpremise-rg` ein.
   * Wählen Sie im Dropdownfeld **Standort** die Region aus.
   * Lassen Sie die Textfelder für den **Vorlagenstamm-URI** bzw. **Parameterstamm-URI** unverändert.
   * Wählen Sie im Dropdownfeld **Betriebssystemtyp** die Option **Windows** oder **Linux** aus.
   * Überprüfen Sie die allgemeinen Geschäftsbedingungen, und aktivieren Sie dann das Kontrollkästchen **Ich stimme den oben genannten Geschäftsbedingungen zu**.
   * Klicken Sie auf die Schaltfläche **Kaufen**.
3. Warten Sie, bis die Bereitstellung abgeschlossen ist.
4. Die Parameterdateien enthalten einen hartcodierten Administratorbenutzernamen und das dazugehörige Kennwort, und es wird dringend empfohlen, beides sofort auf allen VMs zu ändern. Klicken Sie im Azure-Portal auf die einzelnen VMs und dann auf dem Blatt **Support + Problembehandlung** auf **Kennwort zurücksetzen**. Wählen Sie im Dropdownfeld **Modus** die Option **Kennwort zurücksetzen** und dann einen neuen **Benutzernamen** und ein **Kennwort** aus. Klicken Sie auf die Schaltfläche **Aktualisieren**, um den neuen Benutzernamen und das Kennwort dauerhaft zu übernehmen.

<!-- links -->

[implementing-a-multi-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[aad-agent-installation]: /azure/active-directory/active-directory-aadconnect-health-agent-install
[aad-application-proxy]: /azure/active-directory/active-directory-application-proxy-enable
[aad-conditional-access]: /azure/active-directory//active-directory-conditional-access
[aad-connect-sync-default-rules]: /azure/active-directory/active-directory-aadconnectsync-understanding-default-configuration
[aad-connect-sync-operational-tasks]: /azure/active-directory/active-directory-aadconnectsync-operations#staging-mode
[aad-dynamic-memberships]: https://youtu.be/Tdiz2JqCl9Q
[aad-dynamic-membership-rules]: /azure/active-directory/active-directory-accessmanagement-groups-with-advanced-rules
[aad-editions]: /azure/active-directory/active-directory-editions
[aad-filtering]: /azure/active-directory/active-directory-aadconnectsync-configure-filtering
[aad-health]: /azure/active-directory/active-directory-aadconnect-health-sync
[aad-health-adds]: /azure/active-directory/active-directory-aadconnect-health-adds
[aad-health-adfs]: /azure/active-directory/active-directory-aadconnect-health-adfs
[aad-identity-protection]: /azure/active-directory/active-directory-identityprotection
[aad-password-management]: /azure/active-directory/active-directory-passwords-customize
[aad-powershell]: https://msdn.microsoft.com/library/azure/mt757189.aspx
[aad-reporting-guide]: /azure/active-directory/active-directory-reporting-guide
[aad-scalability]: https://blogs.technet.microsoft.com/enterprisemobility/2014/09/02/azure-ad-under-the-hood-of-our-geo-redundant-highly-available-distributed-cloud-directory/
[aad-sync-best-practices]: /azure/active-directory/active-directory-aadconnectsync-best-practices-changing-default-configuration
[aad-sync-disaster-recovery]: /azure/active-directory/active-directory-aadconnectsync-operations#disaster-recovery
[aad-sync-requirements]: /azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements
[aad-topologies]: /azure/active-directory/active-directory-aadconnect-topologies
[aad-user-sign-in]: /azure/active-directory/active-directory-aadconnect-user-signin
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
[azure-multifactor-authentication]: /azure/multi-factor-authentication/multi-factor-authentication
[considerations]: ./considerations.md
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[sla-aad]: https://azure.microsoft.com/support/legal/sla/active-directory/v1_0/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx


[0]: ./images/azure-ad.png "Cloud-Identitätsarchitektur mit Azure Active Directory"