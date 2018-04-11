---
title: CD/CI für Microservices
description: Continuous Integration und Continuous Delivery für Microservices
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: 7d8a81b7bc236e50d722a68a0115b9220d4e094f
ms.sourcegitcommit: 786bafefc731245414c3c1510fc21027afe303dc
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/12/2017
---
# <a name="designing-microservices-continuous-integration"></a>Entwerfen von Microservices: Continuous Integration

Continuous Integration und Continuous Delivery (CI/CD) sind eine wichtige Voraussetzung für die erfolgreiche Verwendung von Microservices. Die Agilität, die von Microservices erwartet wird, lässt sich nur mit einem zuverlässigen CI/CD-Prozess erreichen. Einige der CI/CD-bedingten Herausforderungen für Microservices sind auf die Verwendung mehrerer Codebasen und heterogener Buildumgebungen für die verschiedenen Dienste zurückzuführen. In diesem Kapitel werden die Herausforderungen und einige Lösungsansätze für das Problem beschrieben.

![](./images/ci-cd.png)

Einer der Hauptgründe für die Implementierung einer Microservices-Architektur sind kürzere Versionszyklen. 

In einer rein monolithischen Anwendung gibt es eine einzelne Buildpipeline, die die ausführbare Datei der Anwendung ausgibt. Sämtliche Entwicklungsarbeiten werden dieser Pipeline zugeführt. Bei einem Fehler mit hoher Priorität muss eine Korrektur integriert, getestet und veröffentlicht werden, was die Veröffentlichung neuer Features verzögern kann. Diese Probleme lassen sich durch eine sorgfältige Modulgestaltung sowie durch die Verwendung von Featurebranches behandeln, die die Auswirkungen von Codeänderungen minimieren. Mit zunehmender Komplexität und wachsendem Funktionsumfang einer monolithischen Anwendung nimmt jedoch häufig auch die Fehleranfälligkeit des Releaseprozesses zu. 

Die Microservices-Philosophie sieht kein langwieriges Releaseverfahren vor, in das sich die einzelnen Teams einreihen müssen. Das Team, das den Dienst „A“ erstellt, kann jederzeit ein Update veröffentlichen und muss nicht warten, bis Änderungen für den Dienst „B“ zusammengeführt, getestet und bereitgestellt wurden. Hierfür ist der CI/CD-Prozess unerlässlich. Ihre Releasepipeline muss automatisiert und äußerst zuverlässig sein, um die Risiken von Updatebereitstellungen zu minimieren. Bei täglich oder mehrmals täglich durchgeführten Veröffentlichungen in der Produktionsumgebung darf es nur ganz selten zu Regressionen oder Dienstausfällen kommen. Sollte doch einmal ein fehlerhaftes Update bereitgestellt werden, müssen Sie schnell und zuverlässig einen Rollback oder Rollforward auf eine frühere Dienstversion ausführen können.

![](./images/cicd-monolith.png)

CI/CD umfasst eigentlich mehrere verwandte Prozesse: Continuous Integration, Continuous Delivery und Continuous Deployment.

- Continuous Integration bedeutet, dass Änderungen am Code häufig in der Hauptverzweigung zusammengeführt werden. Durch automatisierte Build- und Testprozesse wird sichergestellt, dass der Code in der Hauptverzweigung immer die nötige Qualität für die Produktionsumgebung hat.

- Continuous Delivery bedeutet, dass Codeänderungen, die den CI-Prozess durchlaufen haben, automatisch in einer produktionsähnlichen Umgebung veröffentlicht werden. Für die Bereitstellung in der aktiven Produktionsumgebung ist unter Umständen eine manuelle Freigabe erforderlich, ansonsten ist der Prozess jedoch automatisiert. Dadurch soll erreicht werden, dass Ihr Code stets für die Bereitstellung in der Produktionsumgebung *bereit* ist.

- Continuous Deployment bedeutet, dass Codeänderungen, die den CI/CD-Prozess durchlaufen haben, automatisch in der Produktionsumgebung bereitgestellt werden.

Im Kontext von Kubernetes und Microservices werden in der CI-Phase Containerimages erstellt, getestet und mithilfe von Push in eine Containerregistrierung übertragen. In der Bereitstellungsphase werden Pod-Spezifikationen aktualisiert, um das neueste Produktionsimage zu übernehmen.

## <a name="challenges"></a>Herausforderungen

- **Zahlreiche kompakte und unabhängige Codebasen:** Jedes Team ist für die Erstellung seines eigenen Diensts verantwortlich und verfügt über eine eigene Buildpipeline. In einigen Organisationen verwenden die Teams unter Umständen separate Coderepositorys. Dies kann dazu führen, dass das Know-how für die Erstellung des Systems auf mehrere Teams verteilt ist und niemand in der Organisation weiß, wie die gesamte Anwendung bereitgestellt wird. Was passiert beispielsweise in einem Notfallwiederherstellungsszenario, wenn Sie schnell eine Bereitstellung in einem neuen Cluster durchführen müssen?   

- **Mehrere Sprachen und Frameworks:** Wenn jedes Team seinen eigenen Technologie-Mix verwendet, kann es schwierig sein, einen einzelnen Buildprozess zu erstellen, der für die gesamte Organisation geeignet ist. Der Buildprozess muss so flexibel sein, dass jedes Team ihn für die gewählte Sprache oder das gewählte Framework anpassen kann. 

- **Integration und Auslastungstests:** Wenn Teams Updates nach ihrem eigenen Zeitplan veröffentlichen, kann die Entwicklung zuverlässiger End-to-End-Tests zur Herausforderung werden – insbesondere, wenn Dienste von anderen Diensten abhängen. Darüber hinaus kann der Betrieb eines vollwertigen Produktionsclusters teuer sein, weshalb es unwahrscheinlich ist, dass jedes Team allein zu Testzwecken einen eigenen vollwertigen Cluster auf Produktionsniveau betreiben kann. 

- **Releaseverwaltung:** Jedes Team sollte die Möglichkeit haben, ein Update in der Produktionsumgebung bereitzustellen. Das bedeutet nicht, dass jedes einzelne Teammitglied über entsprechende Berechtigungen verfügt. Durch eine zentrale Release-Manager-Rolle werden Bereitstellungen jedoch unter Umständen ausgebremst. Je zuverlässiger und stärker automatisiert Ihr CI/CD-Prozess ist, desto weniger Bedarf besteht für eine zentrale Instanz. Für die Veröffentlichung wichtiger Featureupdates und kleinerer Fehlerbehebung können jeweils unterschiedliche Richtlinien gelten. Die Verwendung eines dezentralen Konzepts darf jedoch nicht als Verzicht auf jegliche Art von Governance missverstanden werden.

- **Versionsverwaltung für Containerimages:** Im Rahmen des Entwicklungs- und Testzyklus erstellt der CI/CD-Prozess zahlreiche Containerimages. Nicht alle dieser Images kommen für die Veröffentlichung in Frage, und nur einige der so genannten Release Candidates werden mithilfe von Push in die Produktionsumgebung übertragen. Entwickeln Sie eine klare Versionsverwaltungsstrategie, damit Sie wissen, welche Images derzeit in der Produktionsumgebung bereitgestellt sind, und bei Bedarf einen Rollback auf eine vorherige Version ausführen können. 

- **Dienstupdates:** Wenn Sie einen Dienst auf eine neue Version aktualisieren, dürfen dadurch keine anderen Dienste beeinträchtigt werden, die von diesem Dienst abhängen. Bei einem parallelen Update werden vorübergehend verschiedene Versionen ausgeführt. 
 
Diese Herausforderungen spiegeln ein grundlegendes Dilemma wider: Einerseits müssen Teams so unabhängig wie möglich arbeiten können. Andererseits ist eine gewisse Koordinierung erforderlich, damit eine einzelne Person beispielsweise einen Integrationstest durchführen, die gesamte Lösung in einem neuen Cluster erneut bereitstellen oder im Falle eines fehlerhaften Updates einen Rollback ausführen kann. 
 
## <a name="cicd-approaches-for-microservices"></a>CI/CD-Ansätze für Microservices

Jedes Dienstteam sollte seine Buildumgebung in einem eigenen Container platzieren. Dieser Container muss über sämtliche Buildtools verfügen, die zum Erstellen der Codeartefakte für den jeweiligen Dienst erforderlich sind. Häufig steht ein offizielles Docker-Image für Ihre Kombination aus Sprache und Framework zur Verfügung. Anschließend können Sie den Build mithilfe von `docker run` oder Docker Compose ausführen. 

Mit diesem Ansatz ist die Einrichtung einer neuen Buildumgebung ein Kinderspiel. Ein Entwickler, der Ihren Code erstellen möchte, muss nicht erst eine Reihe von Buildtools installieren, sondern einfach nur das Containerimage ausführen. Vielleicht sogar noch wichtiger: Auch Ihr Buildserver kann für diese Aufgabe konfiguriert werden. Dadurch müssen Sie die Tools nicht auf dem Buildserver installieren oder sich mit Toolversionskonflikten beschäftigen. 

Führen Sie den Dienst bei lokalen Entwicklungs- und Testaufgaben mithilfe von Docker innerhalb eines Containers aus. Im Zuge dieses Prozesses müssen Sie unter Umständen weitere Container mit Pseudodiensten oder Testdatenbanken ausführen, die für lokale Tests benötigt werden. Sie können diese Container mithilfe von Docker Compose koordinieren oder Kubernetes mithilfe von Minikube lokal ausführen. 

Wenn der Code bereit ist, öffnen Sie eine Pull-Anforderung, und führen Sie ihn zu einem Master zusammen. Dadurch wird auf dem Buildserver ein Auftrag gestartet, der Folgendes umfasst:

1. Erstellen der Codeobjekte 
2. Ausführen von Komponententests für den Code
3. Erstellen des Containerimages
4. Testen des Containerimages durch Ausführen von Funktionstests in einem aktiven Container. Dieser Schritt dient zur Erkennung von Fehlern in der Docker-Datei (beispielsweise ein fehlerhafter Einstiegspunkt).
5. Übertragen des Images an eine Containerregistrierung mithilfe von Push
6. Aktualisieren des Testclusters mit dem neuen Image zur Ausführung von Integrationstests

Wenn das Image für die Produktionsumgebung bereit ist, aktualisieren Sie die Bereitstellungsdateien nach Bedarf, um das neueste Image anzugeben (einschließlich möglicher Kubernetes-Konfigurationsdateien). Wenden Sie dann das Update auf den Produktionscluster an.

Im Anschluss folgen einige Empfehlungen zur Verbesserung der Zuverlässigkeit von Bereitstellungen:
 
- Definieren Sie organisationsweite Konventionen für Containertags und für die Versionsverwaltung sowie Namenskonventionen für im Cluster bereitgestellte Ressourcen (Pods, Dienste und Ähnliches). Dies kann die Diagnose von Bereitstellungsproblemen vereinfachen. 

- Erstellen Sie zwei separate Containerregistrierungen: eine für Entwicklungs-/Testaufgaben und eine für die Produktion. Übertragen Sie ein Image erst dann mithilfe von Push in die Produktionsregistrierung, wenn Sie bereit sind, es in der Produktionsumgebung bereitzustellen. Wenn Sie diese Vorgehensweise mit der semantischen Versionsverwaltung von Containerimages kombinieren, verringert sich die Wahrscheinlichkeit, dass Sie versehentlich eine Version bereitstellen, die nicht für die Veröffentlichung freigegeben wurde.

## <a name="updating-services"></a>Aktualisieren von Diensten

Für die Aktualisierung eines Diensts, der sich bereits in Produktion befindet, gibt es verschiedene Strategien. Hier werden drei gängige Optionen erläutert: paralleles Update, Blaugrün-Bereitstellung und Canary-Release.

### <a name="rolling-update"></a>Paralleles Update 

Bei einem parallelen Update stellen Sie neue Instanzen eines Diensts bereit, und diese neuen Instanzen nehmen sofort Anforderungen entgegen. Wenn die neuen Instanzen online geschaltet werden, werden die vorherigen Instanzen entfernt.

Parallele Updates sind das Standardverhalten in Kubernetes, wenn Sie die Pod-Spezifikation für eine Bereitstellung aktualisieren. Der Bereitstellungscontroller erstellt eine neue Replikatgruppe (ReplicaSet) für die aktualisierten Pods. Anschließend wird die neue Replikatgruppe zentral hoch- und die alte Gruppe zentral herunterskaliert, sodass die gewünschte Replikatanzahl erhalten bleibt. Alte Pods werden erst gelöscht, wenn die neuen Pods bereit sind. Kubernetes protokolliert den Verlauf des Updates, damit Sie bei Bedarf mithilfe von kubectl einen Rollback für das Update ausführen können. 

Falls Ihr Dienst eine langwierige Startaufgabe ausführt, können Sie einen Bereitschaftstest definieren. Der Bereitschaftstest meldet, wenn der Container für den Empfang von Datenverkehr bereit ist. Kubernetes sendet erst dann Datenverkehr an den Pod, wenn der Test eine Erfolgsmeldung zurückgibt. 

Eine der Herausforderungen bei parallelen Updates besteht darin, dass während der Aktualisierung eine Mischung aus alten und neuen Versionen ausgeführt wird und Datenverkehr empfängt. Während dieser Zeit können Anforderungen jeweils an eine der beiden Versionen weitergeleitet werden. Dies kann abhängig vom Umfang der Änderungen zwischen den beiden Versionen zu Problemen führen. 

### <a name="blue-green-deployment"></a>Blaugrün-Bereitstellung

Bei einer Blaugrün-Bereitstellung stellen Sie die neue Version zusammen mit der vorherigen Version bereit. Nach der Überprüfung der neuen Version leiten Sie sämtlichen Datenverkehr von der vorherigen Version zur neuen Version um. Anschließend überwachen Sie die Anwendung auf mögliche Probleme. Im Problemfall können Sie zur alten Version zurückkehren. Liegen keine Probleme, können Sie die alte Version löschen.

Bei einer eher traditionellen monolithischen oder n-schichtigen Anwendung war eine Blaugrün-Bereitstellung in der Regel mit der Bereitstellung zweier identischer Umgebungen verbunden. Dabei musste die neue Version in einer Stagingumgebung bereitgestellt und der Clientdatenverkehr anschließend an die Stagingumgebung umgeleitet werden (beispielsweise durch Austauschen der VIP-Adressen).

In Kubernetes müssen Sie für eine Blaugrün-Bereitstellung keinen separaten Cluster bereitstellen. Stattdessen können Sie Selektoren nutzen. Erstellen Sie eine neue Bereitstellungsressource mit einer neuen Pod-Spezifikation und einem anderen Satz von Bezeichnungen. Erstellen Sie diese Bereitstellung, ohne die vorherige Bereitstellung zu löschen oder den Dienst zu ändern, der darauf verweist. Wenn die neuen Pods aktiv sind, können Sie den Selektor des Diensts auf die neue Bereitstellung aktualisieren. 

Ein Vorteil von Blaugrün Bereitstellungen ist, dass alle Pods gleichzeitig gewechselt werden. Nach der Aktualisierung des Diensts werden alle neuen Anforderungen an die neue Version weitergeleitet. Ein Nachteil ist, dass während der Aktualisierung doppelt so viele Pods für den Dienst (aktuelle und neue Version) aktiv sind. Wenn die Pods einen hohen Bedarf an CPU- oder Arbeitsspeicherressourcen haben, müssen Sie den Cluster zur Bewältigung des Ressourcenbedarfs möglicherweise vorübergehend horizontal hochskalieren. 

### <a name="canary-release"></a>Canary-Release

Bei einem Canary-Release führen Sie eine aktualisierte Version für eine geringe Anzahl von Clients ein. Anschließend überwachen Sie das Verhalten des neuen Diensts, bevor Sie ihn für alle Clients einführen. Dies ermöglicht eine langsame, kontrollierte Einführung, die Beobachtung echter Daten und die Erkennung von Problemen, bevor alle Kunden betroffen sind.

Die Verwaltung eines Canary-Release ist im Vergleich zu einer Blaugrün-Bereitstellung oder einem parallelen Update komplexer, da Anforderungen dynamisch an verschiedene Versionen des Diensts weitergeleitet werden müssen. In Kubernetes können Sie einen Dienst so konfigurieren, dass er sich über zwei Replikatgruppen (einen für jede Version) erstreckt, und die Replikatanzahl manuell anpassen. Aufgrund des von Kubernetes verwendeten Lastenausgleichs zwischen Pods ist dieser Ansatz jedoch nicht sehr präzise. Ein Beispiel: Wenn Sie insgesamt über zehn Replikate verfügen, können Sie Datenverkehr nur in Zehn-Prozent-Schritten verlagern. Bei Verwendung eines Dienstnetzes können Sie mithilfe der entsprechenden Routingregeln eine komplexere Canary-Release-Strategie implementieren. Im Anschluss finden Sie einige hilfreiche Ressourcen:

- Kubernetes ohne Dienstnetz: [Canary-Bereitstellungen](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#canary-deployments)
- Linkerd: [Dynamic request routing](https://linkerd.io/features/routing/) (Dynamisches Anforderungsrouting)
- Istio: [Canary Deployments using Istio](https://istio.io/blog/canary-deployments-using-istio.html) (Canary-Bereitstellungen mit Istio)

## <a name="conclusion"></a>Zusammenfassung

In den letzten Jahren hat in der Branche ein grundlegender Wandel stattgefunden – weg vom *System of Record* hin zum *System of Engagement*.

Bei einem System of Record handelt es sich um herkömmliche Backoffice-Datenverwaltungsanwendungen. Den Kern dieser Systeme bildet meist ein RDBMS, das die alleingültige Quelle darstellt. Der Begriff „System of Engagement“ wird Geoffrey Moore zugeschrieben, der ihn 2011 in seiner Arbeit *Systems of Engagement and the Future of Enterprise IT* verwendet hat. Bei einem System of Engagement handelt es sich um Anwendungen, deren Schwerpunkt auf Kommunikation und Zusammenarbeit liegt. Sie verbinden Benutzer in Echtzeit. Sie müssen rund um die Uhr verfügbar sein. Neue Features werden regelmäßig eingeführt, ohne die Anwendung offline zu schalten. Benutzer haben höhere Ansprüche und weniger Verständnis für unerwartete Verzögerungen oder Ausfälle.

Im Verbraucherbereich kann eine höhere Benutzerfreundlichkeit einen messbaren geschäftlichen Nutzen haben. Die Zeit, die ein Benutzer mit einer Anwendung verbringt, führt ggf. direkt zu Umsätzen. Und im Bereich geschäftlicher Systeme haben sich die Erwartungen der Benutzer verändert. Wenn diese Systeme die Kommunikation und Zusammenarbeit fördern sollen, müssen sie sich ein Beispiel an kundenorientierten Anwendungen nehmen.

Microservices sind eine Reaktion auf diese Veränderungen. Durch die Aufspaltung einer monolithischen Anwendung in eine Reihe lose gekoppelter Dienste können wir den Releasezyklus der einzelnen Dienste steuern und häufige Updates ohne Ausfallzeiten oder beeinträchtigende Änderungen ermöglichen. Microservices tragen auch zur besseren Skalierbarkeit, Fehlerisolation und Resilienz bei. Inzwischen vereinfachen Cloudplattformen die Erstellung und Ausführung von Microservices mit automatisierter Bereitstellung von Computeressourcen, Containerorchestratoren als Dienst und ereignisgesteuerten serverlosen Umgebungen.

Andererseits sind Microservices-Architekturen auch mit zahlreichen Herausforderungen verbunden. Entscheidend ist ein solides Design. Gehen Sie bei der Analyse der Domäne, bei der Wahl der Technologien, bei der Modellierung der Daten, bei der Gestaltung der APIs sowie beim Aufbau einer ausgereiften DevOps-Kultur mit der nötigen Sorgfalt vor. Wir hoffen, wir konnten Ihnen mit diesem Leitfaden und der dazugehörigen [Referenzimplementierung](https://github.com/mspnp/microservices-reference-implementation) weiterhelfen. 

