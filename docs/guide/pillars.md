---
title: "Säulen der Softwarequalität"
description: "Beschreibt die fünf Säulen der Softwarequalität: Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltung und Sicherheit."
author: MikeWasson
ms.openlocfilehash: 78e613368a07718f5923d619ace335d399b0cc80
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="pillars-of-software-quality"></a>Säulen der Softwarequalität 

Eine gelungene Cloudanwendung basiert auf fünf Säulen der Softwarequalität, nämlich Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltung und Sicherheit.

| Säule | Beschreibung |
|--------|-------------|
| Skalierbarkeit | Die Fähigkeit eines Systems, eine höhere Last zu verarbeiten. |
| Verfügbarkeit | Der Zeitanteil, in dem ein System funktioniert und erreichbar ist. |
| Resilienz | Die Fähigkeit eines Systems, nach Ausfällen eine Wiederherstellung durchzuführen und die Betriebsbereitschaft sicherzustellen. |
| Verwaltung | Operative Prozesse, die für die Ausführung eines Systems in der Produktion sorgen. |
| Sicherheit | Schützen von Anwendungen und Daten vor Bedrohungen. |

## <a name="scalability"></a>Skalierbarkeit

Skalierbarkeit ist die Fähigkeit eines Systems, eine höhere Last zu verarbeiten. Es gibt im Wesentlichen zwei Möglichkeiten, eine Anwendung zu skalieren. Beim *vertikalen Hochskalieren* wird die Kapazität einer Ressource erhöht, z.B. durch Verwenden eines größeren virtuellen Computers. Beim *horizontalen Hochskalieren* werden neue Instanzen einer Ressource hinzugefügt, z.B. virtuelle Computer oder Datenbankreplikate. 

Die horizontale Skalierung bietet gegenüber der vertikalen Skalierung einige wichtige Vorteile:

- Echter Cloudmaßstab. Anwendung können für die Ausführung auf Hunderten oder sogar Tausenden von Knoten entworfen werden und damit einen Maßstab erreichen, der auf einem Einzelknoten nicht möglich ist.
- Die horizontale Skalierung ist elastisch. Sie können Instanzen hinzufügen, wenn die Last wächst, und diese in ruhigeren Zeiträumen wieder entfernen.
- Die horizontale Hochskalierung kann automatisch ausgelöst werden, entweder gemäß einem Zeitplan oder als Reaktion auf Änderungen der Auslastung. 
- Das horizontale Hochskalieren kann kostengünstiger sein als das vertikale Hochskalieren. Die Ausführung verschiedener kleiner virtueller Computer kann weniger kosten als ein einziger großer virtueller Computer. 
- Die horizontale Skalierung kann durch Hinzufügen von Redundanz auch die Resilienz verbessern. Wenn eine Instanz ausfällt, wird die Anwendung trotzdem weiter ausgeführt.

Ein Vorteil der vertikalen Skalierung besteht darin, dass Sie hierfür keine Änderungen der Anwendung erforderlich sind. An einem bestimmten Punkt ist jedoch eine Obergrenze erreicht, ab der eine weitere vertikale Hochskalierung nicht mehr möglich ist. An diesem Punkt muss jede weitere Skalierung horizontal erfolgen. 

Die horizontale Skalierung muss im System konzipiert werden. Sie können z.B. virtuelle Computer horizontal hochskalieren, indem Sie sie hinter einem Lastenausgleichsmodul platzieren. Allerdings muss jeder virtuelle Computer im Pool in der Lage sein, jede Clientanforderung zu verarbeiten, daher muss der virtuelle Computer zustandslos sein oder den Zustand extern speichern (beispielsweise in einem verteilten Cache). In verwalteten PaaS-Diensten ist die horizontale und automatische Skalierung häufig bereits integriert. Die einfache Skalierbarkeit ist ein großer Vorteil von PaaS-Diensten.

Das simple Hinzufügen weiterer Instanzen bedeutet jedoch nicht, dass eine Anwendung skaliert wird. Dadurch wird der Engpass einfach nur an eine andere Stelle verlagert. Wenn Sie z.B. ein Web-Front-End für die Verarbeitung einer größeren Anzahl von Clientanforderungen skalieren, kann dies Sperrkonflikte in der Datenbank auslösen. In diesem Fall müssten Sie weitere Maßnahmen ergreifen, z.B. optimistische Parallelität oder Datenpartitionierung einrichten, um einen höheren Durchsatz für die Datenbank zu ermöglichen.

Führen Sie immer Leistungs- und Auslastungstests durch, um diese potenziellen Engpässe zu finden. Die zustandsbehafteten Teile eines Systems, wie etwa die Datenbanken, sind die häufigste Ursache für Engpässe und erfordern einen sorgfältigen Entwurf, um sich horizontal skalieren zu lassen. Durch das Beheben eines Engpasses können sich weitere Engpässe an anderen Stellen zeigen.

Verwenden Sie die [Checkliste für die Skalierbarkeit][scalability-checklist], um Ihren Entwurf vom Standpunkt der Skalierbarkeit aus zu überprüfen.

### <a name="scalability-guidance"></a>Leitfaden für die Skalierbarkeit

- [Entwurfsmuster für Skalierbarkeit und Leistung][scalability-patterns]
- Bewährte Methoden: [Automatische Skalierung][autoscale], [Hintergrundaufträge][background-jobs], [Caching][caching], [CDN][cdn], [Datenpartitionierung][data-partitioning]

## <a name="availability"></a>Verfügbarkeit

Verfügbarkeit ist der Zeitanteil, in dem ein System funktioniert und erreichbar ist. Sie wird üblicherweise als Prozentsatz der Betriebszeit gemessen. Anwendungsfehler, Infrastrukturprobleme und Systemauslastung können die Verfügbarkeit reduzieren. 

Eine Cloudanwendung sollte ein Servicelevelziel (Service Level Objective, SLO) aufweisen, das die erwartete Verfügbarkeit klar definiert und angibt, wie die Verfügbarkeit gemessen wird. Betrachten Sie beim Definieren der Verfügbarkeit den kritischen Pfad. Möglicherweise kann das Web-Front-End Clientanforderungen verarbeiten, aber wenn keine Transaktion ausgeführt werden kann, weil eine Verbindung mit der Datenbank nicht möglich ist, ist die Anwendung für Benutzer nicht verfügbar. 

Verfügbarkeit wird oft in Form von Neunen angegeben – „vier Neunen“ bedeutet beispielsweise eine Verfügbarkeit von 99,99 %. Die folgende Tabelle zeigt die potenziellen kumulativen Ausfallzeiten für verschiedene Verfügbarkeitsebenen.

| Betriebszeit in % | Ausfallzeit pro Woche | Ausfallzeit pro Monat | Ausfallzeit pro Jahr |
|----------|-------------------|--------------------|-------------------|
| 99 % | 1,68 Stunden | 7,2 Stunden | 3,65 Tage |
| 99,9 % | 10 Minuten | 43,2 Minuten | 8,76 Stunden |
| 99,95 % | 5 Minuten | 21,6 Minuten | 4,38 Stunden |
| 99,99 % | 1 Minute | 4,32 Minuten | 52,56 Minuten |
| 99.999% | 6 Sekunden | 26 Sekunden | 5,26 Minuten |

Beachten Sie, dass eine Betriebszeit von 99 % eine Ausfallzeit von fast zwei Stunden pro Woche bedeuten kann. Für viele Anwendungen – insbesondere für Endverbraucheranwendungen – ist das kein akzeptables SLO. Am anderen Ende des Spektrums bedeuten fünf Neunen (99,999 %) eine Ausfallzeit von nur fünf Minuten in einem ganzen *Jahr*. Es ist schwer genug, einen Ausfall so schnell zu entdecken, geschweige denn das Problem zu lösen. Um eine sehr hohe Verfügbarkeit zu erreichen (99,99 % oder höher), dürfen Sie sich nicht auf manuelle Eingriffe verlassen, um Ihre Systeme nach einem Ausfall wiederherzustellen. Die Anwendung muss eine Selbstdiagnose und Selbstreparatur ausführen können – an diese Stelle ist die Resilienz von entscheidender Bedeutung.

In Azure wird in der Vereinbarung zum Servicelevel (SLA) die garantierte Betriebszeit und Konnektivität beschrieben, die Microsoft zusichert. Wenn die Vereinbarung zum Servicelevel für einen bestimmten Dienst 99,95 % beträgt, können Sie erwarten, dass der Dienst 99,95 % der Zeit verfügbar ist.

Anwendungen benötigen häufig mehrere Dienste. Im Allgemeinen ist die Wahrscheinlichkeit, dass einer der Dienste ausfällt, von den anderen Diensten unabhängig. Angenommen, Ihre Anwendung verwendet zwei Dienste, und jeder weist eine SLA von 99,9 % auf. Die kombinierte SLA für beide Dienste beträgt 99,9% &times; 99,9% &asymp; 99,8% oder geringfügig weniger als jeder Dienst für sich genommen. 

Verwenden Sie die [Checkliste für die Verfügbarkeit][availability-checklist], um Ihren Entwurf vom Standpunkt der Verfügbarkeit aus zu überprüfen.

### <a name="availability-guidance"></a>Leitfaden für die Verfügbarkeit

- [Entwurfsmuster für Verfügbarkeit][availability-patterns]
- Bewährte Methoden: [automatische Skalierung][autoscale], [Hintergrundaufträge][background-jobs]

## <a name="resiliency"></a>Resilienz

Resilienz ist die Fähigkeit des Systems, nach Ausfällen für ein System eine Wiederherstellung durchzuführen und die Betriebsbereitschaft sicherzustellen. Das Ziel der Resilienz besteht darin, nach einem Ausfall wieder die volle Funktionsfähigkeit einer Anwendung herzustellen. Resilienz steht in engem Zusammenhang mit der Verfügbarkeit.

In der herkömmlichen Anwendungsentwicklung lag der Fokus darauf, die MTBF (Mean Time Between Failures) zu verkürzen. Es wurde viel Mühe darauf verwandt, das System vor Ausfällen zu schützen. Beim Cloud Computing ist aufgrund verschiedener Faktoren eine andere Herangehensweise erforderlich:

- Verteilte Systeme sind komplex, und ein Ausfall an einem Punkt kann durch das gesamte System kaskadieren.
- Die Kosten für Cloudumgebungen werden niedrig gehalten, indem Standardhardware eingesetzt wird. Mit gelegentlichen Hardwarefehlern muss also gerechnet werden. 
- Anwendungen nutzen oft externe Dienste, die zeitweilig nicht verfügbar sein können oder Benutzer mit hohem Volumen drosseln. 
- Benutzer erwarten heute, dass eine Anwendung rund um die Uhr verfügbar ist und niemals offline geschaltet wird.

All diese Faktoren bedeuten, dass Cloudanwendungen so entworfen werden müssen, dass gelegentliche Ausfälle erwartet werden und die Anwendung danach wiederhergestellt werden kann. In der Azure-Plattform sind viele Resilienzfeatures bereits integriert. Beispiel: 

- Azure Storage, SQL-Datenbank und Cosmos DB bieten integrierte Datenreplikation, sowohl innerhalb einer Region als auch regionsübergreifend.
- Azure Managed Disks werden automatisch in verschiedenen Skalierungseinheiten platziert, um die Auswirkungen von Hardwarefehlern zu begrenzen.
- Virtuelle Computer in einer Verfügbarkeitsgruppe werden auf mehrere Fehlerdomänen verteilt. Eine Fehlerdomäne ist eine Gruppe virtueller Computer, die eine Stromquelle und einen Netzwerkswitch gemeinsam nutzen. Durch Verteilen der virtuellen Computer auf verschiedene Fehlerdomänen werden die Auswirkungen von physischen Hardwareausfällen, Netzwerkausfällen oder Stromunterbrechungen begrenzt.

Nichtsdestoweniger müssen Sie weiterhin Resilienz in Ihre Anwendung integrieren. Resilienzstrategien können auf allen Ebenen der Architektur angewendet werden. Einige Risikominderungen sind eher taktischer Natur – beispielsweise das Wiederholen eines Remoteaufrufs nach einem vorübergehenden Netzwerkausfall. Andere Risikominderungen sind eher strategisch, z.B. das Failover der gesamten Anwendung in eine sekundäre Region. Taktische Risikominderungen können einen großen Unterschied ausmachen. Es passiert eher selten, dass eine gesamte Region ausfällt. Vorübergehende Probleme wie eine Netzwerküberlastung sind wesentlich häufiger – gehen Sie diese also zuerst an. Die richtigen Überwachungs- und Diagnosemaßnahmen sind ebenfalls von großer Bedeutung, sowohl zum Erkennen von Problemen, wenn diese auftreten, als auch zum Ermitteln der Grundursachen.

Wenn Sie eine resiliente Anwendung entwerfen, müssen Sie Ihre Verfügbarkeitsanforderungen genau kennen. Wie viel Ausfallzeit ist akzeptabel? Dies ist teilweise eine Kostenfunktion. Welche Kosten fallen bei potenziellen Ausfallzeiten für Ihr Unternehmen an? Wie viel sollten Sie investieren, um die Hochverfügbarkeit der Anwendung sicherzustellen?

Verwenden Sie die [Checkliste für die Resilienz][resiliency-checklist], um Ihren Entwurf vom Standpunkt der Resilienz aus zu überprüfen.

### <a name="resiliency-guidance"></a>Leitfaden zu Resilienz

- [Entwickeln robuster Anwendungen für Azure][resiliency]
- [Entwurfsmuster für Resilienz][resiliency-patterns]
- Bewährte Methoden: [Behandeln vorübergehender Fehler][transient-fault-handling], [Wiederholungsanleitung für bestimmte Dienste][retry-service-specific]

## <a name="management-and-devops"></a>Verwaltung und DevOps

Diese Säule deckt die Betriebsprozesse ab, die für die Ausführung einer Anwendung in der Produktion sorgen.

Bereitstellungen müssen zuverlässig und vorhersagbar sein. Sie sollten automatisiert erfolgen, um das Risiko menschlicher Fehler zu senken. Es sollte ein schneller und routinierter Prozess sein, damit die Freigabe neuer Features oder Fehlerbehebungen nicht verlangsamt wird. Gleichermaßen wichtig ist auch, dass Sie einen schnellen Rollback oder Rollforward ausführen können, wenn es ein Problem bei einem Update gibt.

Die Überwachung und die Diagnose sind von entscheidender Bedeutung. Cloudanwendungen werden in einem Remoterechenzentrum ausgeführt, in dem Sie keine vollständige Kontrolle über die Infrastruktur oder – in manchen Fällen – das Betriebssystem haben. Bei einer großen Anwendung ist es nicht praktikabel, sich bei virtuellen Computern anzumelden, um ein Problem zu beheben oder Protokolldateien zu durchsuchen. Bei PaaS-Diensten gibt es möglicherweise nicht einmal einen dedizierten virtuellen Computer, bei dem Sie sich anmelden könnten. Überwachung und Diagnose bieten Einblick in das System, sodass Sie ermitteln können, wann und wo Fehler auftreten. Alle Systeme müssen beobachtet werden können. Verwenden Sie ein allgemeines und konsistentes Protokollschema, mit dem Sie Ereignisse systemübergreifend korrelieren können.

Der Überwachungs- und Diagnoseprozess besteht aus mehreren unterschiedlichen Phasen:

- Instrumentierung: Generieren der Rohdaten aus Anwendungsprotokollen, Webserverprotokollen, in die Azure-Plattform integrierten Diagnosefunktionen und anderen Diensten.
- Sammlung und Speicherung: Konsolidieren der Daten an einer zentralen Stelle.
- Analyse und Diagnose: Beheben von Problemen und Übersicht über die Integrität insgesamt.
- Visualisierung und Warnungen: Verwenden von Telemetriedaten, um Tendenzen zu erkennen oder das Betriebsteam zu benachrichtigen.

Verwenden Sie die [DevOps-Checkliste][devops-checklist], um Ihren Entwurf vom Verwaltungs- und DevOps-Standpunkt aus zu überprüfen.

### <a name="management-and-devops-guidance"></a>Leitfaden für Verwaltung und DevOps

- [Entwurfsmuster für Verwaltung und Überwachung][management-patterns]
- Bewährte Methoden: [Überwachung und Diagnose][monitoring]

## <a name="security"></a>Sicherheit

Sie müssen die Sicherheit für den gesamten Lebenszyklus einer Anwendung berücksichtigen – von Entwurf und Implementierung bis hin zu Bereitstellung und Betrieb. Die Azure-Plattform bietet Schutz vor einer Vielzahl von Bedrohungen, z.B. Eindringen in das Netzwerk und DDoS-Angriffe. Dennoch müssen Sie weiterhin Sicherheitsfunktionen in Ihre Anwendung und Ihre DevOps-Prozesse integrieren.

Im Folgenden finden Sie einige allgemeine Sicherheitsbereiche, die es zu berücksichtigen gilt. 

### <a name="identity-management"></a>Identitätsverwaltung

Ziehen Sie in Betracht, Azure Active Directory (Azure AD) zur Authentifizierung und Autorisierung von Benutzern zu verwenden. Azure AD ist ein vollständig verwalteter Identitäts- und Zugriffsverwaltungsdienst. Sie können damit Domänen erstellen, die nur in Azure existieren, oder Azure AD in Ihre eigenen lokalen Active Directory-Identitäten integrieren. Azure AD lässt sich ebenfalls in Office 365, Dynamics CRM Online und viele SaaS-Anwendungen von Drittanbietern integrieren. Bei Endverbraucheranwendungen ermöglicht Azure Active Directory B2C es Benutzern, sich mit ihren vorhandenen Konten sozialer Netzwerke (z.B. Facebook, Google oder LinkedIn) zu authentifizieren oder ein neues Benutzerkonto zu erstellen, das von Azure AD verwaltet wird.

Wenn Sie eine lokale Active Directory-Umgebung in ein Azure-Netzwerk integrieren möchten, gibt es je nach Anforderungen verschiedene Herangehensweisen. Weitere Informationen finden Sie in unseren Referenzarchitekturen zur [Identitätsverwaltung][identity-ref-arch].

### <a name="protecting-your-infrastructure"></a>Schützen Ihrer Infrastruktur 

Kontrollieren Sie den Zugriff auf die von Ihnen bereitgestellten Azure-Ressourcen. Jedes Azure-Abonnement weist eine [Vertrauensstellung][ad-subscriptions] mit einem Azure AD-Mandanten auf. Verwenden Sie die [rollenbasierte Zugriffssteuerung][rbac] (Role-Based Access Control, RBAC), um den Benutzern in Ihrer Organisation die richtigen Berechtigungen für Azure-Ressourcen zu gewähren. Gewähren Sie den Zugriff, indem Sie Benutzern oder Gruppen für einen bestimmten Bereich RBAC-Rollen zuweisen. Bei dem Bereich kann es sich um ein Abonnement, eine Ressourcengruppe oder eine einzelne Ressource handeln. [Überwachen][resource-manager-auditing] Sie alle Änderungen an der Infrastruktur. 

### <a name="application-security"></a>Anwendungssicherheit

Im Allgemeinen gelten die bewährten Methoden für die Sicherheit in der Anwendungsentwicklung auch in der Cloud weiterhin. Hierzu gehören Aspekte wie standortunabhängiges SSL, Schutz vor CSRF- und XSS-Angriffen, Verhindern von Angriffen durch Einschleusung von SQL-Befehlen usw. 

Cloudanwendungen verwenden häufig verwaltete Dienste, die über Zugriffsschlüssel verfügen. Implementieren Sie diese niemals in die Quellcodeverwaltung. Speichern Sie Anwendungsgeheimnisse in Azure Key Vault.

### <a name="data-sovereignty-and-encryption"></a>Datensouveränität und -verschlüsselung

Stellen Sie sicher, dass Ihre Daten in der richtigen geopolitischen Zone bleiben, wenn Sie die Hochverfügbarkeitsfeatures von Azure verwenden. Der georeplizierte Azure-Speicher nutzt das Konzept der [Regionspaare][paired-region] in der gleichen geopolitischen Region. 

Verwenden Sie Key Vault, um Kryptografieschlüssel und Geheimnisse zu sichern. Mit Key Vault können Sie Schlüssel und Geheimnisse mithilfe von Schlüsseln verschlüsseln, die durch Hardwaresicherheitsmodule (HSMs) geschützt werden. Viele Speicher- und Datenbankdienste in Azure unterstützen die Verschlüsselung ruhender Daten – hierzu gehören [Azure Storage][storage-encryption], [Azure SQL-Datenbank][sql-db-encryption], [Azure SQL Data Warehouse][data-warehouse-encryption] und [Cosmos DB][documentdb-encryption].

### <a name="security-resources"></a>Sicherheitsressourcen

- [Azure Security Center][security-center] bietet eine integrierte Sicherheitsüberwachung und Richtlinienverwaltung für all Ihre Azure-Abonnements. 
- [Dokumentation zur Azure-Sicherheit][security-documentation]
- [Microsoft Trust Center][trust-center]



<!-- links -->

[dr-guidance]: ../resiliency/disaster-recovery-azure-applications.md
[identity-ref-arch]: ../reference-architectures/identity/index.md
[resiliency]: ../resiliency/index.md

[ad-subscriptions]: /azure/active-directory/active-directory-how-subscriptions-associated-directory
[data-warehouse-encryption]: /azure/data-lake-store/data-lake-store-security-overview#data-protection
[documentdb-encryption]: /azure/documentdb/documentdb-nosql-database-security
[rbac]: /azure/active-directory/role-based-access-control-what-is
[paired-region]: /azure/best-practices-availability-paired-regions
[resource-manager-auditing]: /azure/azure-resource-manager/resource-group-audit
[security-blog]: https://azure.microsoft.com/blog/tag/security/
[security-center]: https://azure.microsoft.com/services/security-center/
[security-documentation]: /azure/security/
[sql-db-encryption]: /azure/sql-database/sql-database-always-encrypted-azure-key-vault
[storage-encryption]: /azure/storage/storage-service-encryption
[trust-center]: https://azure.microsoft.com/support/trust-center/
 

<!-- patterns -->
[availability-patterns]: ../patterns/category/availability.md
[management-patterns]: ../patterns/category/management-monitoring.md
[resiliency-patterns]: ../patterns/category/resiliency.md
[scalability-patterns]: ../patterns/category/performance-scalability.md


<!-- practices -->
[autoscale]: ../best-practices/auto-scaling.md
[background-jobs]: ../best-practices/background-jobs.md
[caching]: ../best-practices/caching.md
[cdn]: ../best-practices/cdn.md
[data-partitioning]: ../best-practices/data-partitioning.md
[monitoring]: ../best-practices/monitoring.md
[retry-service-specific]: ../best-practices/retry-service-specific.md
[transient-fault-handling]: ../best-practices/transient-faults.md


<!-- checklist -->
[availability-checklist]: ../checklist/availability.md
[devops-checklist]: ../checklist/dev-ops.md
[resiliency-checklist]: ../checklist/resiliency.md
[scalability-checklist]: ../checklist/scalability.md
