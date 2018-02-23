---
title: "Schützen von Datenlösungen"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 57897c31a8abdcd801874bf92d60360f7a80d1fa
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/14/2018
---
# <a name="securing-data-solutions"></a>Schützen von Datenlösungen

Viele, die Daten in der Cloud bereitstellen, machen sich Gedanken über die erhöhte Zugänglichkeit der Daten sowie über neue Möglichkeiten zum Schutz dieser Daten – insbesondere, wenn bislang ausschließlich lokale Datenspeicher verwendet wurden.

## <a name="challenges"></a>Herausforderungen

* Zentralisierung der Überwachung und Analyse gespeicherter Sicherheitsereignisse aus zahlreichen Protokollen
* Implementierung von Verschlüsselung und Autorisierungsverwaltung für Ihre Anwendungen und Dienste
* Gewährleistung einer funktionierenden Identitätsverwaltung für alle Ihre Lösungskomponenten (sowohl lokal als auch in der Cloud)

## <a name="data-protection"></a>Datenschutz

Der erste Schritt beim Schutz von Informationen besteht darin, die zu schützenden Informationen zu identifizieren. Entwickeln Sie klare, einfache und deutlich kommunizierte Richtlinien, um die wichtigsten Datenressourcen zu identifizieren, zu schützen und zu überwachen – ganz gleich, wo sich diese befinden. Richten Sie für Ressourcen, die für die Mission oder Rentabilität des Unternehmens besonders wichtig sind, den bestmöglichen Schutz ein. Diese Ressourcen werden als HVAs (High Value Assets) bezeichnet. Führen Sie strikte Analysen für den HVA-Lebenszyklus und für Sicherheitsabhängigkeiten durch, und richten Sie geeignete Sicherheitskontrollen und -bedingungen ein. Identifizieren und klassifizieren Sie analog dazu sensible Ressourcen, und definieren Sie die Technologien und Prozesse zur automatischen Anwendung von Sicherheitskontrollen.

Überlegen Sie sich nach der Identifizierung der zu schützenden Daten, wie Sie sie *in Ruhe* und *während der Übertragung* schützen möchten.

* **Ruhende Daten:** Daten, die sich statisch auf physischen Medien (magnetischer oder optischer Datenträger) befinden – lokal oder in der Cloud.
* **Übertragene Daten:** Daten, die zwischen Komponenten, Speicherorten oder Programmen übertragen werden – beispielsweise über das Netzwerk, über einen Service Bus (aus der lokalen Umgebung in die Cloud oder umgekehrt) oder während eines Eingabe-/Ausgabeprozesses.

Weitere Informationen zum Schutz ruhender Daten sowie zum Schutz von Daten während der Übertragung finden Sie unter [Empfohlene Vorgehensweisen für Datensicherheit und Verschlüsselung in Azure](/azure/security/azure-security-data-encryption-best-practices).

## <a name="access-control"></a>Zugriffssteuerung

Um Ihre Daten in der Cloud zu schützen, benötigen Sie eine Kombination aus Identitätsverwaltung und Zugriffssteuerung. Angesichts der Vielfalt und Art von Clouddiensten sowie der zunehmenden Beliebtheit von [Hybrid Cloud](../scenarios/hybrid-on-premises-and-cloud.md) sollten Sie bei der Identitäts- und Zugriffssteuerung einige wichtige Punkte berücksichtigen:

* Zentralisieren Sie Ihre Identitätsverwaltung.
* Ermöglichen Sie einmaliges Anmelden (Single Sign-On, SSO).
* Stellen Sie eine Kennwortverwaltung bereit.
* Erzwingen Sie für Benutzer die mehrstufige Authentifizierung (Multi-Factor Authentication, MFA).
* Verwenden Sie die rollenbasierte Zugriffssteuerung (Role-Based Access Control, RBAC).
* Konfigurieren Sie Richtlinien für bedingten Zugriff. Diese erweitern das klassische Konzept der Benutzeridentität um zusätzliche Eigenschaften wie den Standort des Benutzers, den Gerätetyp und die Patchebene.
* Steuern Sie Standorte, an denen Ressourcen erstellt werden, mit Resource Manager.
* Überwachen Sie die Umgebung aktiv auf verdächtige Aktivitäten.

Weitere Informationen finden Sie unter [Azure-Identitätsverwaltung und Sicherheit der Zugriffssteuerung – Bewährte Methoden](/azure/security/azure-security-identity-management-best-practices).

## <a name="auditing"></a>Überwachung

Zusätzlich zur bereits erwähnten Identitäts- und Zugriffsüberwachung sollten die Dienste und Anwendungen, die Sie in der Cloud verwenden, sicherheitsrelevante Ereignisse generieren, die Sie überwachen können. Die größte Herausforderung bei der Überwachung dieser Ereignisse ist die Bewältigung der Menge an Protokollen, um potenzielle Probleme zu vermeiden und bereits aufgetretene Probleme zu behandeln. Cloudbasierte Anwendungen umfassen häufig zahlreiche dynamische Komponenten, von denen die meisten ein gewisses Maß an Protokollierungs- und Telemetriedaten erzeugen. Verwenden Sie eine zentrale Überwachung und Analyse, um diese Informationsflut zu bewältigen und zu interpretieren.

Weitere Informationen finden Sie unter [Azure-Protokollierung und -Überwachung](/azure/security/azure-log-audit).



## <a name="securing-data-solutions-in-azure"></a>Schützen von Datenlösungen in Azure

### <a name="encryption"></a>Verschlüsselung

**Virtuelle Computer:** Verwenden Sie [Azure Disk Encryption](/azure/security/azure-security-disk-encryption), um die angefügten Datenträger von virtuellen Windows- oder Linux-Computern zu verschlüsseln. Die Lösung kann in [Azure Key Vault](/azure/key-vault/) integriert werden, um die Steuerung und Verwaltung der Datenträger-Verschlüsselungsschlüssel und -geheimnisse zu erleichtern. 

**Azure Storage:** Verwenden Sie [Azure Storage Service Encryption](/azure/storage/common/storage-service-encryption), um ruhende Daten in Azure Storage automatisch zu verschlüsseln. Verschlüsselung, Entschlüsselung und Schlüsselverwaltung sind für Benutzer vollständig transparent. Durch eine clientseitige Verschlüsselung mit Azure Key Vault können Daten auch während der Übertragung geschützt werden. Weitere Informationen finden Sie unter [Clientseitige Verschlüsselung und Azure Key Vault für Microsoft Azure Storage](/azure/storage/common/storage-client-side-encryption).

**SQL Database** und **Azure SQL Data Warehouse:** Verwenden Sie [Transparent Data Encryption](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) (TDE), um Ihre Datenbanken, die dazugehörigen Sicherungen und die Transaktionsprotokolldateien in Echtzeit zu verschlüsseln und zu entschlüsseln, ohne Änderungen an Ihren Anwendung vorzunehmen. SQL-Datenbank kann auch [Always Encrypted](/azure/sql-database/sql-database-always-encrypted-azure-key-vault) verwenden, um sensible Daten auf dem Server (ruhende Daten), auf dem Weg zwischen Client und Server sowie während der Verwendung zu schützen. Die Always Encrypted-Verschlüsselungsschlüssel können mit Azure Key Vault gespeichert werden. 

### <a name="rights-management"></a>Rights Management

[Azure Rights Management](/information-protection/understand-explore/what-is-azure-rms) ist ein cloudbasierter Dienst, der Verschlüsselungs-, Identitäts- und Autorisierungsrichtlinien zum Schutz von Dateien und E-Mails verwendet. Er kann geräteübergreifend auf Smartphones, Tablets und PCs verwendet werden. Da der Schutz auch dann an die Daten gebunden bleibt, wenn diese Ihre Organisation verlassen, können Informationen sowohl innerhalb als auch außerhalb Ihrer Organisation geschützt werden.

### <a name="access-control"></a>Zugriffssteuerung

Verwenden Sie die [rollenbasierte Zugriffssteuerung](/azure/active-directory/role-based-access-control-what-is) (Role-Based Access Control, RBAC), um den Zugriff auf Azure-Ressourcen auf der Grundlage von Benutzerrollen zu steuern. Wenn Sie Active Directory lokal verwenden, können Sie Benutzern per [Synchronisierung mit Azure AD](/azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements) eine auf ihrer lokalen Identität basierende Cloudidentität zur Verfügung stellen.

Verwenden Sie den [bedingten Zugriff in Azure Active Directory](/azure/active-directory/active-directory-conditional-access-azure-portal), um den Anwendungszugriff in Ihrer Umgebung auf der Grundlage bestimmter Bedingungen zu steuern. Ein Beispiel für eine solche Richtlinienanweisung wäre etwa: _Wenn Auftragnehmer versuchen, über nicht vertrauenswürdige Netzwerke auf unsere Cloud-Apps zuzugreifen, soll der Zugriff blockiert werden._ 

[Azure AD Privileged Identity Management](/azure/active-directory/active-directory-privileged-identity-management-configure) kann Sie bei der Verwaltung, Steuerung und Überwachung Ihrer Benutzer sowie der Aufgaben unterstützen, die sie mit ihren Administratorrechten ausführen. Dies ist ein wichtiger Schritt, um einzuschränken, wer in Ihrer Organisation privilegierte Vorgänge in Azure AD, Azure, Office 365 oder SaaS-Apps ausführen sowie deren Aktivitäten überwachen kann.

### <a name="network"></a>Netzwerk

Verwenden Sie zum Schutz von Daten während der Übertragung immer SSL/TLS, wenn Daten zwischen verschiedenen Standorten ausgetauscht werden. Manchmal muss der gesamte Kommunikationskanal zwischen der lokalen Infrastruktur und der Cloudinfrastruktur mithilfe eines virtuellen privaten Netzwerks (VPN) oder mithilfe von [ExpressRoute](/azure/expressroute/) isoliert werden. Weitere Informationen finden Sie unter [Erweitern lokaler Datenlösungen auf die Cloud](../scenarios/hybrid-on-premises-and-cloud.md).

Verwenden Sie [Netzwerksicherheitsgruppen](/azure/virtual-network/virtual-networks-nsg) (NSGs), um die Anzahl potenzieller Angriffsvektoren zu verringern. Eine Netzwerksicherheitsgruppe enthält eine Liste mit Sicherheitsregeln, die ein- oder ausgehenden Netzwerkdatenverkehr basierend auf IP-Adresse, Port und Protokoll (für die Quelle bzw. das Ziel) zulassen oder ablehnen. 

Verwenden Sie [Dienstendpunkte im virtuellen Netzwerk](/azure/virtual-network/virtual-network-service-endpoints-overview), um Azure SQL- oder Azure Storage-Ressourcen zu schützen, sodass nur Datenverkehr aus Ihrem virtuellen Netzwerk auf diese Ressourcen zugreifen kann.

Virtuelle Computer in einem virtuellen Azure-Netzwerk (VNet) können mittels [Peering in virtuellen Netzwerken](/azure/virtual-network/virtual-network-peering-overview) sicher mit anderen VNets kommunizieren. Netzwerkdatenverkehr zwischen virtuellen Netzwerken, die mittels Peering verknüpft sind, ist privat. Datenverkehr zwischen den virtuellen Netzwerken bleibt innerhalb des Microsoft-Backbone-Netzwerks.

Weitere Informationen finden Sie unter [Azure-Netzwerksicherheit](/azure/security/azure-network-security).

### <a name="monitoring"></a>Überwachung

[Azure Security Center](/azure/security-center/security-center-intro) erfasst, analysiert und integriert automatisch Protokolldaten aus Ihren Azure-Ressourcen, aus dem Netzwerk und aus verbundenen Partnerlösungen (beispielsweise Firewalllösungen), um echte Bedrohungen zu erkennen und weniger falsch positive Ergebnisse zu generieren. 

[Log Analytics](/azure/log-analytics/log-analytics-overview) bietet zentralen Zugriff auf Ihre Protokolle und unterstützt Sie bei der Datenanalyse sowie bei der Erstellung benutzerdefinierter Warnungen.

Die [Bedrohungserkennung von Azure SQL-Datenbank](/azure/sql-database/sql-database-threat-detection) erkennt anomale Aktivitäten, die auf ungewöhnliche und potenziell schädliche Versuche hindeuten, auf Datenbanken zuzugreifen oder diese zu missbrauchen. Sicherheitsbeauftragte oder andere zugewiesene Administratoren können eine sofortige Benachrichtigung über verdächtige Datenbankaktivitäten erhalten, sobald diese auftreten. Jede Benachrichtigung enthält Details zur verdächtigen Aktivität und empfiehlt die Vorgehensweise zur weiteren Untersuchung und Abwendung der Bedrohung.


