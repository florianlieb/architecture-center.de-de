---
title: Checkliste für DevOps
description: Checkliste mit Anleitungen zu DevOps.
author: dragon119
ms.date: 01/10/2018
ms.custom: checklist
ms.openlocfilehash: 2e338d2f2e61b404223001a61f44e06e89e7f563
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="devops-checklist"></a>Checkliste für DevOps

DevOps ist die Verschmelzung von Entwicklung, Qualitätssicherung und IT-Vorgängen in eine einheitliche Kultur und einen einheitlichen Prozesssatz für die Bereitstellung von Software. Verwenden Sie diese Checkliste als Ausgangspunkt, um Ihre DevOps-Kultur und -Prozesse zu bewerten.

## <a name="culture"></a>Kultur

**Sorgen Sie für eine einheitliche geschäftliche Ausrichtung aller Organisationen und Teams.** Konflikte in Bezug auf Ressourcen, Zweck, Ziele und Prioritäten in einer Organisation können ein Risiko für einen erfolgreichen Betrieb darstellen. Stellen Sie sicher, dass die Geschäfts-, Entwicklungs- und Betriebsteams die gleichen Ziele verfolgen.

**Stellen Sie sicher, dass das gesamte Team den Softwarelebenszyklus versteht.** Ihr Team muss den Gesamtlebenszyklus der Anwendung verstehen und wissen, in welcher Lebenszyklusphase die Anwendung sich derzeit befindet. So wissen alle Teammitglieder, welche Aufgaben gerade zu erledigen sind und wie sie die zukünftige Entwicklung planen und vorbereiten müssen.

**Verkürzen Sie die Zykluszeiten.** Versuchen Sie, die Zeitspanne zwischen der ersten Idee und einer nutzbaren entwickelten Software zu verkürzen. Begrenzen Sie Größe und Umfang einzelner Releases, um den Testaufwand so niedrig wie möglich zu halten. Automatisieren Sie die Build-, Test-, Konfigurations- und Bereitstellungsprozesse, wo immer dies möglich ist. Beseitigen Sie alle möglicherweise vorhandenen Kommunikationshindernisse innerhalb des Entwicklerteams und zwischen Entwickler- und Betriebsteam. 

**Prüfen und verbessern Sie Prozesse.** Ihre Prozesse und Verfahren – sowohl automatisiert als auch manuell – sollten nie als final betrachtet werden. Setzen Sie regelmäßige Überprüfungen aktueller Workflows, Verfahren und Dokumentationsmaterialien an. Das Ziel ist eine kontinuierliche Verbesserung.

**Sorgen Sie für eine proaktive Planung.** Planen Sie proaktiv für mögliche Fehler. Richten Sie Prozesse ein, um auftretende Probleme schnell zu identifizieren, zur Behebung an die richtigen Teammitglieder zu eskalieren und die Problemlösung zu bestätigen.

**Lernen Sie aus Fehlern.** Fehler lassen sich nicht vermeiden, und es ist wichtig, daraus zu lernen, damit sie sich nicht wiederholen. Wenn ein Fehler im Betrieb auftritt, kategorisieren Sie ihn, dokumentieren Ursache und Lösung, und geben Sie die aus dem Fehler gezogenen Schlussfolgerungen weiter. Aktualisieren Sie Ihre Buildprozesse nach Möglichkeit, damit diese Art Fehler in Zukunft automatisch erkannt wird.

**Optimieren Sie Ihr System im Hinblick auf Geschwindigkeit und Datensammlung.** Jede geplante Verbesserung ist eine Hypothese. Arbeiten Sie in möglichst kleinen Schritten. Betrachten Sie neue Ideen als Experimente. Instrumentieren Sie die Experimente, sodass Sie Produktionsdaten sammeln können, um deren Effektivität zu bewerten. Seien Sie darauf vorbereitet, Fail-Fast-fähig zu sein, wenn die Hypothese sich als falsch herausstellt.

**Planen Sie genügend Zeit für Lernprozesse ein.** Sowohl aus Fehlern als auch aus Erfolgen lässt sich Neues lernen. Bevor Sie neue Projekte in Angriff nehmen, planen Sie genügend Zeit ein, um die wichtigsten Erkenntnisse zusammenzustellen, und stellen Sie sicher, dass Ihr Team diese Erkenntnisse entsprechend umsetzt. Lassen Sie dem Team auch genügend Zeit, um sich neue Kenntnisse und Fähigkeiten anzueignen, zu experimentieren und neue Tools und Techniken kennenzulernen. 

**Dokumentieren Sie Vorgänge.** Dokumentieren Sie alle Tools, Prozesse und automatisierten Aufgaben in der gleichen Qualität wie Ihren Produktcode. Dokumentieren Sie den aktuellen Entwurf und die aktuelle Architektur aller Systeme, die Sie unterstützen. Vergessen Sie auch die Wiederherstellungsprozesse und andere Wartungsverfahren nicht. Konzentrieren Sie sich auf die Schritte, die tatsächlich ausgeführt werden, nicht auf die Prozesse, die theoretisch optimal wären. Überprüfen und aktualisieren Sie die Dokumentation regelmäßig. Wenn es um den Code geht, stellen Sie sicher, dass er aussagekräftige Kommentare enthält – insbesondere in öffentlichen APIs –, und verwenden Sie nach Möglichkeit Tools zum automatischen Generieren der Codedokumentation. 

**Geben Sie Wissen weiter.** Dokumentationsmaterialien sind nur dann nützlich, wenn Benutzer wissen, dass sie vorhanden und wo sie zu finden sind. Stellen Sie sicher, dass die Dokumentation gut organisiert und problemlos aufzufinden ist. Seien Sie kreativ: Geben Sie Wissen beispielsweise in informellen Präsentationen, Videos oder Newslettern weiter.

## <a name="development"></a>Entwicklung

**Stellen Sie für Entwickler Umgebungen bereit, die die gleichen Merkmale aufweisen wie Produktionsumgebungen.** Wenn die Entwicklungs- und Testumgebungen nicht der Produktionsumgebung entsprechen, lassen sich Probleme nur sehr schwer testen und diagnostizieren. Richten Sie daher die Entwicklungs- und Testumgebungen so ein, dass sie der Produktionsumgebung so ähnlich wie möglich sind. Stellen Sie sicher, dass die Testdaten konsistent mit den Daten sind, die in der Produktion verwendet werden – auch wenn es sich (aus Gründen des Datenschutzes oder aufgrund rechtlicher Vorschriften) nicht um echte Produktionsdaten handelt, sondern um Testdaten. Planen Sie die Generierung und Anonymisierung von Beispieldaten für Tests.

**Stellen Sie sicher, dass alle autorisierten Teammitglieder die Infrastruktur einrichten und die Anwendung bereitstellen können.** Um produktionsähnliche Ressourcen einzurichten und die Anwendung bereitzustellen, sollten keine komplizierten manuellen Aufgaben oder detaillierten technischen Systemkenntnisse erforderlich sein. Jeder Benutzer mit der richtigen Berechtigung sollte in der Lage sein, produktionsähnliche Ressourcen zu erstellen oder bereitzustellen, ohne das Betriebsteam involvieren zu müssen. 

> Diese Empfehlung impliziert nicht, dass jeder Benutzer Liveupdates in die Produktionsbereitstellung übertragen kann. Es geht darum, es den Entwicklungs- und QA-Teams so einfach wie möglich zu machen, produktionsähnliche Umgebungen zu erstellen.

**Instrumentieren Sie die Anwendung, um Einblicke zu erhalten.** Um die Integrität Ihrer Anwendung bewerten zu können, müssen Sie wissen, wie leistungsfähig sie ist und ob Fehler oder Probleme auftreten. Schließen Sie die Instrumentierung immer in die Designanforderungen ein, und integrieren Sie sie von Anfang an in die Anwendung. Die Instrumentierung muss eine Ereignisprotokollierung für die Ursachenermittlung umfassen. Sie muss ebenfalls die Sammlung von Telemetrie- und Metrikdaten beinhalten, um die Integrität und Nutzung der Anwendung insgesamt zu überwachen.

**Verfolgen Sie Ihre technischen Schulden nach.** In vielen Projekten können Releasezeitpläne bis zu einem gewissen Grad Vorrang vor der Codequalität erhalten. Wenn dies der Fall ist, stellen Sie eine sorgfältige Nachverfolgung sicher. Dokumentieren Sie alle Abkürzungen und andere nicht optimale Implementierungen, und planen Sie für zukünftige Releases Zeit ein, um diese Probleme erneut anzugehen.

**Erwägen Sie, Updates direkt in die Produktionsumgebung zu übermitteln.** Um den Releasezyklus insgesamt zu beschleunigen, ziehen Sie in Betracht, ordnungsgemäß getestete Codecommits direkt in die Produktionsumgebung zu übermitteln. Verwenden Sie [Feature Toggles][feature-toggles], um zu steuern, welche Features aktiviert sind. So können Sie schnell von der Entwicklung zum Release gelangen, indem Sie die Toggles verwenden, um Features zu aktivieren oder zu deaktivieren. Toggles sind auch hilfreich bei Tests wie [Canary Releases][canary-release], bei denen ein bestimmtes Feature nur für einen Teil der Produktionsumgebung bereitgestellt wird.

## <a name="testing"></a>Testen

**Automatisieren Sie das Testing.** Das manuelle Testen von Software ist mühsam und fehleranfällig. Automatisieren Sie häufige Testaufgaben, und integrieren Sie die Tests in Ihren Buildprozess. Mit automatisierten Tests können Sie sicherstellen, dass Tests alle notwendigen Funktionen abdecken und sich wiederholen lassen. Tests der integrierten Benutzeroberfläche sollten ebenfalls durch ein automatisiertes Tool durchgeführt werden. Azure bietet Entwicklungs- und Testressourcen, die Sie bei der Konfiguration und Ausführung von Tests unterstützen. Weitere Informationen finden Sie unter [Entwicklung und Test][dev-test].

**Testen Sie auf Fehler.** Wie reagiert ein System, wenn es keine Verbindung zu einem Dienst herstellen kann? Kann es sich selbst wiederherstellen, sobald der Dienst wieder verfügbar ist? Integrieren Sie Fault Injection-Tests als standardmäßigen Bestandteil der Überprüfung in Test- und Stagingumgebungen. Wenn Ihre Testprozesse und -verfahren ausgereift sind, ziehen Sie in Betracht, diese Tests in der Produktionsumgebung auszuführen. 

**Testen Sie in der Produktion.** Der Releaseprozess endet nicht mit der Bereitstellung in der Produktion. Richten Sie Tests ein, um sicherzustellen, dass der bereitgestellte Code erwartungsgemäß funktioniert. Bei Bereitstellungen, die nur selten aktualisiert werden, planen Sie Produktionstests als regulären Bestandteil der Wartung.

**Automatisieren Sie Leistungstests, um Leistungsprobleme frühzeitig zu erkennen.** Die Auswirkungen eines ernsthaften Leistungsproblems können genauso schwerwiegend sein wie ein Fehler im Code. Automatisierte Funktionstests können zwar Anwendungsfehler verhindern, erkennen aber möglicherweise keine Leistungsprobleme. Definieren Sie akzeptable Leistungsziele für Metriken wie Latenz, Ladezeiten und Ressourcennutzung. Integrieren Sie automatisierte Leistungstests in Ihre Pipeline, um sicherzustellen, dass die Anwendung diese Ziele erfüllt.

**Führen Sie Kapazitätstests durch.** Auch wenn eine Anwendung unter Testbedingungen gut funktioniert, können in der Produktion aufgrund von Skalierungs- oder Ressourceneinschränkungen Probleme auftreten. Definieren Sie immer die maximal zu erwartenden Grenzwerte hinsichtlich Kapazität und Nutzung. Führen Sie Tests aus, um sicherzustellen, dass die Anwendung diese Einschränkungen bewältigen kann. Testen Sie jedoch auch, was passiert, wenn diese Grenzwerte überschritten werden. Kapazitätstests sollten regelmäßig durchgeführt werden.

Nach dem ersten Release sollten Sie immer dann Leistungs- und Kapazitätstests durchführen, wenn der Produktionscode aktualisiert wird. Verwenden Sie Verlaufsdaten, um Tests zu optimieren und zu bestimmen, welche Arten von Tests durchgeführt werden müssen.

**Führen Sie automatisierte Penetrationstests durch.** Die Bereitstellung von Anwendungssicherheit ist genauso wichtig wie das Testen jeder anderen Funktionalität. Richten Sie automatisierte Penetrationstests als standardmäßigen Bestandteil des Build- und Bereitstellungsprozesses ein. Planen Sie regelmäßige Sicherheitstests und Überprüfungen auf Sicherheitsrisiken für bereitgestellte Anwendungen. Überwachen Sie das System auf offene Ports, Endpunkte und Angriffe. Automatisierte Tests entbinden Sie jedoch nicht von der Verpflichtung, in regelmäßigen Abständen detaillierte Sicherheitsüberprüfungen durchzuführen.

**Führen Sie automatisierte Tests bezüglich der Geschäftskontinuität durch.** Entwickeln Sie Tests für die Geschäftskontinuität in großem Umfang, einschließlich Wiederherstellung aus Sicherungen und Failovermechanismen. Richten Sie automatisierte Prozesse ein, um diese Tests regelmäßig durchzuführen.

## <a name="release"></a>Release

**Automatisieren Sie Bereitstellungen.** Automatisieren Sie die Bereitstellung der Anwendung in Test-, Staging- und Produktionsumgebungen. Die Automatisierung ermöglicht schnellere und zuverlässigere Bereitstellungen und stellt konsistente Bereitstellungen in jeder unterstützten Umgebung sicher. Zudem eliminiert sie das Risiko menschlicher Fehler, die bei manuellen Bereitstellungen passieren können. Sie vereinfacht auch die Bereitstellung zu möglichst passenden Uhrzeiten, um mögliche Auswirkungen von Ausfallzeiten zu minimieren.

**Nutzen Sie Continuous Integration.** Continuous Integration (CI) ist ein Verfahren, mit dem regelmäßig der gesamte Entwicklercode in einer zentralen Codebasis zusammengeführt wird (Merging). Darauf basierend werden automatisch die standardmäßigen Build- und Testprozesse ausgeführt. Continuous Integration stellt sicher, dass das gesamte Team gleichzeitig ohne Konflikte mit der gleichen Codebasis arbeiten kann. CI stellt auch sicher, dass Fehler im Code so frühzeitig wie möglich gefunden werden. Der CI-Prozess sollte optimalerweise jedes Mal ausgeführt werden, wenn Code committet oder eingecheckt wird. Dieser Prozess sollte zumindest einmal am Tag ausgeführt werden.

> Erwägen Sie den Einsatz eines [trunkbasierten Entwicklungsmodells][trunk-based]. In diesem Modell committen Entwickler ihren Code in einen einzelnen Branch (Trunk). Eine Anforderung lautet, dass Commits den Build nicht unterbrechen dürfen. Dieses Modell vereinfacht die Continuous Integration, da alle Featureaufgaben im Trunk ausgeführt und mögliche Mergekonflikte aufgelöst werden, wenn der Commit erfolgt.

**Ziehen Sie Continuous Delivery in Betracht.** Continuous Delivery (CD) ist ein Verfahren, um sicherzustellen, dass Code jederzeit bereitgestellt werden kann. Dieses Verfahren führt automatische Build-, Test- und Bereitstellungsprozesse für den Code in produktionsähnlichen Umgebungen aus. Richten Sie eine vollständige CI/CD-Pipeline ein, um Codefehler so schnell wie möglich zu erkennen und sicherzustellen, dass ordnungsgemäß getestete Updates innerhalb kürzester Zeit freigegeben werden können.

> Continuous *Deployment* ist ein weiterer Prozess, der automatisch alle Updates, die durch die CI/CD-Pipeline weitergegeben wurden, in der Produktion bereitstellt. Continuous Deployment erfordert stabile automatische Testverfahren sowie eine erweiterte Prozessplanung und eignet sich möglicherweise nicht für jedes Team.

**Nehmen Sie kleine inkrementelle Änderungen vor.** Bei umfangreichen Codeänderungen ist das Risiko größer, dass sich Fehler einschleichen. Nehmen Sie Änderungen nach Möglichkeit nur in geringem Umfang vor. So können Sie die möglichen Auswirkungen jeder einzelnen Änderung eingrenzen und Fehler einfacher verstehen und debuggen.

**Steuern Sie die Verbreitung von Änderungen.** Stellen Sie sicher, dass Sie die vollständige Kontrolle darüber haben, welche Updates für Endbenutzer sichtbar sein sollen. Verwenden Sie ggf. Feature Toggles, um zu steuern, wann Features für Endbenutzer aktiviert werden.

**Implementieren Sie Strategien für die Releaseverwaltung, um das Bereitstellungsrisiko zu senken.** Die Bereitstellung eines Anwendungsupdates in der Produktion birgt immer ein gewisses Risiko. Um dieses Risiko zu minimieren, nutzen Sie Strategien wie z.B. [Canary Releases][canary-release] oder [Blue-Green Deployments][blue-green], um Updates nur für einen Teil der Benutzer bereitzustellen. Überprüfen Sie, ob das Update erwartungsgemäß funktioniert, und führen Sie es dann im restlichen System ein.

**Dokumentieren Sie alle Änderungen.** Kleine Updates und Konfigurationsänderungen können zu Irritationen und Versionskonflikten führen. Führen Sie eine sorgfältige Liste sämtlicher Änderungen – ganz egal, wie klein eine Änderung auch sein mag. Protokollieren Sie jede Art von Änderung, z.B. aufgespielte Patches oder Richtlinien- und Konfigurationsänderungen. (Speichern Sie keine vertraulichen Daten in diesen Protokollen. Protokollieren Sie z.B., dass Anmeldeinformationen aktualisiert wurden und welcher Benutzer die Änderung vorgenommen hat. Führen Sie jedoch nicht die aktualisierten Anmeldeinformationen auf.) Die Liste der Änderungen sollte für das gesamte Team sichtbar sein. 

**Automatisieren Sie Bereitstellungen.** Automatisieren Sie alle Bereitstellungen, und richten Sie Systeme ein, die Probleme während des Rollouts erkennen. Richten Sie einen Prozess für die Risikominderung ein, um den in der Produktion vorhandenen Code und die vorhandenen Daten zu sichern, bevor das Update Code und Daten in allen Produktionsinstanzen ersetzt. Richten Sie ein automatisiertes Verfahren ein, um bei Fehlerbehebungen einen Rollforward sowie bei Änderungen einen Rollback auszuführen.

**Ziehen Sie in Betracht, die Infrastruktur unveränderlich zu machen.** Eine unveränderliche Infrastruktur setzt das Prinzip um, dass die Infrastruktur nach dem Bereitstellen in der Produktion nicht mehr geändert wird. Andernfalls kann es zu einem Zustand kommen, in dem Ad-hoc-Änderungen angewendet wurden, sodass es schwierig ist, die genauen Änderungen zu identifizieren. Bei einer unveränderlichen Infrastruktur werden im Rahmen jeder neuen Bereitstellung ganze Server ausgetauscht. So können Code und Hostingumgebung als Block getestet und bereitgestellt werden. Nach der Bereitstellung werden Infrastrukturkomponenten bis zum nächsten Build- und Bereitstellungszyklus nicht geändert. 

## <a name="monitoring"></a>Überwachung

**Sorgen Sie dafür, dass Systeme überwacht werden können.** Das Betriebsteam sollte jederzeit klaren Einblick in die Integrität und den Status eines Systems oder Diensts haben. Richten Sie externe Integritätsendpunkte für die Statusüberwachung ein, und stellen Sie sicher, dass Anwendungen so programmiert sind, dass die Betriebsmetriken instrumentiert werden. Verwenden Sie ein allgemeines und konsistentes Schema, mit dem Sie Ereignisse systemübergreifend korrelieren können. [Azure-Diagnose][azure-diagnostics] und [Application Insights][app-insights] sind die Standardmethoden zum Nachverfolgen der Integrität und des Status von Azure-Ressourcen. Microsoft [Operation Management Suite][oms] bietet ebenfalls zentrale Überwachungs- und Verwaltungsfunktionen für Cloud- oder Hybridlösungen.

**Aggregieren und korrelieren Sie Protokolle und Metriken**. Ein ordnungsgemäß instrumentiertes Telemetriesystem liefert eine große Menge an Rohdaten zur Leistung und Ereignisprotokollen. Stellen Sie sicher, dass die Telemetrie- und Protokolldaten innerhalb kürzester Zeit verarbeitet und korreliert werden, sodass dem Betriebsteam jederzeit der aktuelle Stand der Systemintegrität angezeigt wird. Daten sollten so organisiert und angezeigt werden, dass sie eine zusammenhängende Übersicht über Probleme bieten, sodass nach Möglichkeit ersichtlich ist, wenn Ereignisse miteinander in Verbindung stehen.

> Suchen Sie in der Aufbewahrungsrichtlinie Ihres Unternehmens nach den Anforderungen in Bezug darauf, wie Daten verarbeitet werden und wie lange sie gespeichert werden müssen. 

**Implementieren Sie automatische Warnungen und Benachrichtigungen.** Richten Sie Überwachungstools wie [Azure Monitor][azure-monitor] ein, um Muster oder Bedingungen zu erkennen, die auf potenzielle oder aktuelle Probleme hinweisen. Senden Sie Warnungen an die Teammitglieder, die diese Probleme beheben können. Optimieren Sie die Warnungen, um falsch positive Ergebnisse zu vermeiden.

**Überwachen Sie den Ablauf von Assets und Ressourcen.** Einige Ressourcen und Assets, wie z.B. Zertifikate, laufen nach einem bestimmten Zeitraum ab. Stellen Sie sicher, dass Sie genau nachverfolgen, welche Assets ablaufen, wann der Ablauf erfolgt und von welchen Diensten oder Features diese Assets benötigt werden. Verwenden Sie automatisierte Prozesse, um diese Assets zu überwachen. Benachrichtigen Sie das Betriebsteam, bevor ein Asset abläuft, und eskalieren Sie das Problem, falls durch den Ablauf eine Störung der Anwendung droht.

## <a name="management"></a>Verwaltung

**Automatisieren Sie Betriebsaufgaben.** Die manuelle Ausführung der immer gleichen Betriebsprozesse ist fehleranfällig. Automatisieren Sie diese Aufgaben nach Möglichkeit, um eine konsistente Ausführung und Qualität sicherzustellen. Code, der die Automatisierung implementiert, muss in der Quellcodeverwaltung versioniert werden. Und wie bei jedem anderen Code auch müssen Automatisierungstools getestet werden.

**Implementieren Sie die Bereitstellung als „Infrastructure-as-Code“.** Minimieren Sie den Umfang manueller Konfigurationsaufgaben, die für die Bereitstellung von Ressourcen ausgeführt werden müssen. Verwenden Sie stattdessen Skripts und [Azure Resource Manager][resource-manager]-Vorlagen. Verwalten Sie die Skripts und Vorlagen in der Quellcodeverwaltung, wie jeden anderen von Ihnen verwalteten Code auch. 

**Ziehen Sie die Verwendung von Containern in Betracht.** Container bieten eine standardmäßige, paketbasierte Schnittstelle für die Bereitstellung von Anwendungen. Bei diesem Ansatz werden Anwendungen mithilfe von eigenständigen Paketen bereitgestellt, die alle Softwarefunktionen, Abhängigkeiten und Dateien enthalten, die zur Ausführung der Anwendung erforderlich sind. So lässt sich der Bereitstellungsprozess ganz erheblich vereinfachen. 

Container erstellen auch eine Abstraktionsschicht zwischen der Anwendung und dem zugrunde liegenden Betriebssystem, die für umgebungsübergreifende Konsistenz sorgt. Diese Abstraktion kann einen Container auch von anderen Prozessen oder Anwendungen isolieren, die auf einem Host ausgeführt werden. 

**Implementieren Sie Resilienz und Selbstreparatur.** Resilienz bezeichnet die Fähigkeit einer Anwendung, nach Ausfällen eine Wiederherstellung durchzuführen. Resilienzstrategien umfassen Wiederholungsversuche nach vorübergehenden Fehlern sowie die Durchführung eines Failovers in eine sekundäre Instanz oder sogar in eine andere Region. Weitere Informationen finden Sie unter [Entwerfen robuster Anwendungen für Azure][resiliency]. Instrumentieren Sie Ihre Anwendungen, sodass Probleme sofort gemeldet werden und Sie Ausfälle oder andere Systemfehler beheben können.

**Erstellen Sie ein Betriebshandbuch.** Ein Betriebshandbuch oder *Runbook* dokumentiert die Verfahren und Verwaltungsinformationen, die vom Betriebspersonal für die Ausführung und Wartung eines Systems benötigt werden. Dokumentieren Sie des Weiteren alle Betriebsszenarien und Pläne für die Schadenminimierung, die nach einem Ausfall oder einer anderen Unterbrechung des Diensts ins Spiel kommen können. Erstellen Sie diese Dokumentation während des Entwicklungsprozesses, und halten Sie sie danach immer auf dem neuesten Stand. Diese Dokumentation sollte regelmäßig überprüft, getestet und verbessert werden. 

Es ist von entscheidender Bedeutung, diese Dokumentation für alle Benutzer freizugeben, die sie benötigen. Fordern Sie die Teammitglieder auf, zur Dokumentation beizutragen und ihr Wissen zu teilen. Das gesamte Team sollte Zugriff auf die Dokumente haben. Machen Sie es den Mitgliedern möglichst einfach, die Dokumente auf dem neuesten Stand zu halten.

**Dokumentieren Sie Verfahren für Bereitschaftsdienste.** Stellen Sie sicher, dass alle Verpflichtungen, Zeitpläne und Verfahren für Bereitschaftsdienste dokumentiert sind und allen Teammitgliedern zur Verfügung gestellt wurden. Diese Informationen müssen immer aktuell sein.

**Dokumentieren Sie Eskalationsverfahren im Fall von Abhängigkeiten von Drittanbietern.** Wenn Ihre Anwendung von externen Drittanbieterdiensten abhängig ist, die nicht Ihrer direkten Kontrolle unterliegen, müssen Sie einen Plan entwickeln, wie bei Ausfällen verfahren werden soll. Erstellen Sie eine Dokumentation Ihrer geplanten Prozesse für die Schadenminimierung. Fügen Sie Informationen zu Supportkontakten und Eskalationspfaden ein.

**Sorgen Sie für Konfigurationsverwaltung.** Konfigurationsänderungen sollten geplant werden, für Vorgänge sichtbar sein und aufgezeichnet werden. Hierfür können Sie eine Datenbank für die Konfigurationsverwaltung verwenden oder einen Configuration-as-Code-Ansatz wählen. Die Konfiguration sollte regelmäßig überwacht werden, um sicherzustellen, dass die erwarteten Funktionen und Merkmale auch tatsächlich vorhanden sind.

**Erwerben Sie einen Azure-Supportplan, und informieren Sie sich über den Prozess.** Azure bietet eine Reihe von [Supportplänen an][azure-support-plans]. Ermitteln Sie den Plan, der sich für Ihre Anforderungen am besten eignet, und stellen Sie sicher, dass das gesamte Team den Plan kennt und weiß, wie vorzugehen ist. Alle Teammitglieder müssen die Einzelheiten des Plans kennen und wissen, wie der Supportprozess funktioniert und wie sie in Azure ein Supportticket eröffnen. Wenn Sie ein Ereignis erwarten, das sich in hohem Maß auf Ihre Anwendung auswirken wird, kann der Azure-Support Ihnen beim Erhöhen der Dienstlimits helfen. Weitere Informationen finden Sie unter [Häufig gestellte Fragen zum Azure-Support](https://azure.microsoft.com/support/faq/).

**Befolgen Sie beim Gewähren des Zugriffs auf Ressourcen das Prinzip der geringsten Rechte.** Gehen Sie beim Verwalten des Zugriffs auf Ressourcen mit großer Umsicht vor. Standardmäßig sollte der Zugriff verweigert werden, sofern einem Benutzer nicht explizit Zugriff auf eine Ressource gewährt wurde. Gewähren Sie Benutzern nur Zugriff auf diejenigen Ressourcen, die sie zum Durchführen ihrer Aufgaben benötigen. Verfolgen Sie Benutzerberechtigungen nach, und führen Sie regelmäßige Sicherheitsüberwachungen durch.

<strong>Verwenden Sie die rollenbasierte Zugriffssteuerung.</strong> Das Zuweisen von Benutzerkonten und des Zugriffs auf Ressourcen sollte kein manueller Prozess sein. Verwenden Sie die [rollenbasierte Zugriffssteuerung][rbac] (Role-Based Access Control, RBAC), um Zugriff basierend auf [Azure Active Directory][azure-ad]-Identitäten und -Gruppen zu gewähren. 

**Verwenden Sie ein System zur Nachverfolgung von Fehlern.** Ohne eine gute Methode zum Nachverfolgen von Fehlern kann es leicht passieren, dass etwas übersehen wird, Aufgaben doppelt ausgeführt werden oder sich zusätzliche Probleme einschleichen. Verlassen Sie sich nicht auf eine informelle Kommunikation zwischen Benutzern, um den Status von Fehlern nachzuverfolgen. Verwenden Sie ein Tool zum Nachverfolgen von Fehlern, um Details zu Problemen aufzuzeichnen, Ressourcen zur Behebung zuzuweisen und ein Überwachungsprotokoll in Bezug auf Fortschritt und Status bereitzustellen. 

**Verwalten Sie alle Ressourcen in einem Change Management-System.** Alle Aspekte Ihres DevOps-Prozesses sollten in einem Verwaltungs- und Versionierungssystem enthalten sein, sodass Änderungen problemlos nachverfolgt und überwacht werden können. Hierzu gehören Code, Infrastruktur, Konfiguration, Dokumentation und Skripts. Behandeln Sie all diese Arten von Ressourcen über den gesamten Test-, Build- und Reviewprozess hinweg als Code. 

**Verwenden Sie Checklisten.** Legen Sie Betriebschecklisten an, um sicherzustellen, dass die Prozesse befolgt werden. Bei Verwendung umfangreicher Handbücher kann es leicht passieren, dass etwas übersehen wird. Bei der Verwendung einer Checkliste dagegen wird die Aufmerksamkeit auf Details gelenkt, die andernfalls möglicherweise nicht bemerkt worden wären. Verwalten und pflegen Sie die Checklisten, und suchen Sie kontinuierlich nach Möglichkeiten, Aufgaben zu automatisieren und Prozesse zu optimieren.

Weitere Informationen zu DevOps finden Sie auf der Website zu Visual Studio unter [Was ist DevOps?][what-is-devops].

<!-- links -->

[app-insights]: /azure/application-insights/
[azure-ad]: https://azure.microsoft.com/services/active-directory/
[azure-diagnostics]: /azure/monitoring-and-diagnostics/azure-diagnostics
[azure-monitor]: /azure/monitoring-and-diagnostics/monitoring-overview
[azure-support-plans]: https://azure.microsoft.com/support/plans/
[blue-green]: https://martinfowler.com/bliki/BlueGreenDeployment.html
[canary-release]:https://martinfowler.com/bliki/CanaryRelease.html
[dev-test]: https://azure.microsoft.com/solutions/dev-test/
[feature-toggles]: https://www.martinfowler.com/articles/feature-toggles.html
[oms]: https://www.microsoft.com/cloud-platform/operations-management-suite
[rbac]: /azure/active-directory/role-based-access-control-what-is
[resiliency]: ../resiliency/index.md
[resource-manager]: /azure/azure-resource-manager/
[trunk-based]: https://trunkbaseddevelopment.com/
[what-is-devops]: https://www.visualstudio.com/learn/what-is-devops/
