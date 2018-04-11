---
title: Entwurf mit Blick auf den Betrieb
description: Setzen Sie es sich beim Entwurf Ihrer Anwendung als Ziel, dem Betriebsteam die benötigten Verwaltungstools zur Verfügung zu stellen.
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 76338cc27daf82ccb99df4e4c51c7a5ac6f26065
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="design-for-operations"></a>Entwurf mit Blick auf den Betrieb

## <a name="design-an-application-so-that-the-operations-team-has-the-tools-they-need"></a>Setzen Sie es sich beim Entwurf Ihrer Anwendung als Ziel, dem Betriebsteam die benötigten Verwaltungstools zur Verfügung zu stellen.

Durch die Cloud hat sich die Rolle des Betriebsteams erheblich geändert. Es ist nicht mehr für die Verwaltung der Hardware und Infrastruktur zum Hosten der Anwendung zuständig.  Der allgemeine Betrieb ist allerdings weiterhin ein ausschlaggebender Teil der Ausführung einer erfolgreichen Cloudanwendung. Zu den wichtigsten Funktionen des Betriebsteams gehören:

- Bereitstellung
- Überwachung
- Eskalation
- Reaktion auf Vorfälle
- Sicherheitsüberwachung

Stabile Protokollierung und Nachverfolgung sind in Cloudanwendungen besonders wichtig. Beziehen Sie das Betriebsteam in den Entwurf und die Planung ein, um sicherzustellen, dass es von der Anwendung alle für eine erfolgreiche Arbeit erforderlichen Daten und Einblicke erhält.  <!-- to do: Link to DevOps checklist -->

## <a name="recommendations"></a>Recommendations

**Stellen Sie die Überwachungsfähigkeit aller Komponenten sicher**. Sobald eine Lösung bereitgestellt ist und ausgeführt wird, stellen Protokolle und Ablaufverfolgungen Ihren primären Einblick in das System dar. Die *Ablaufverfolgung* zeichnet einen Pfad durch das System auf und ist nützlich zum Identifizieren von Engpässen, Leistungsproblemen und Schwachstellen. Die *Protokollierung* erfasst einzelne Ereignisse, z.B. Änderungen am Zustand der Anwendung, Fehler und Ausnahmen. Führen Sie die Protokollierung in Produktionssystemen durch, andernfalls verlieren Sie Einblicke genau zu den Zeiten, in denen Sie sie am meisten benötigen.

**Ermöglichen Sie die Überwachung**. Überwachung bietet Einblicke in die (gute oder schlechte) Leistung einer Anwendung im Hinblick auf Verfügbarkeit, Leistung und Systemintegrität. Beispielsweise erhalten Sie durch Überwachung Aufschluss darüber, ob Sie die SLA erfüllen. Die Überwachung erfolgt während des normalen Systembetriebs. Sie sollte möglichst nah an Echtzeit liegen, damit die Betriebsmitarbeiter schnell auf Probleme reagieren können. Im Idealfall hilft Überwachung dabei, Probleme zu vermeiden, bevor sie zu einem kritischen Fehler führen. Weitere Informationen finden Sie unter [Überwachung und Diagnose][monitoring].

**Bereiten Sie eine Grundursachenanalyse vor**. Als Grundursachenanalyse wird der Vorgang bezeichnet, die zugrunde liegende Ursache von Ausführungsfehlern zu suchen. Dies geschieht, nachdem ein Fehler bereits aufgetreten ist. 

**Verwenden Sie die verteilte Ablaufverfolgung**. Verwenden Sie ein verteiltes Ablaufverfolgungssystem, das für Parallelität, asynchrone Abläufe und Cloudvolumen entwickelt wurde. Ablaufverfolgungen sollten eine Korrelations-ID einschließen, die über Dienstgrenzen hinweg gilt. In einem einzelnen Vorgang können mehrere Anwendungsdienste aufgerufen werden. Wenn bei einem Vorgang ein Fehler auftritt, hilft die Korrelations-ID dabei, die Ursache des Fehlers zu ermitteln. 

**Standardisieren Sie Protokollen und Metriken**. Das Betriebsteam muss Protokolle von den verschiedenen Diensten in der Lösung sammeln. Wenn jeder Dienst ein eigenes Protokollierungsformat verwendet, ist es schwierig oder unmöglich, hilfreiche Informationen zu gewinnen. Definieren Sie ein gemeinsames Schema, das Felder wie Korrelations-ID, Ereignisname, IP-Adresse des Senders usw. enthält. Die einzelnen Dienste können benutzerdefinierte Schemas ableiten, die das Basisschema erben und zusätzliche Felder enthalten.

**Automatisieren Sie Verwaltungsaufgaben**, einschließlich der Bereitstellung und Überwachung. Durch das Automatisieren einer Aufgabe wird sie wiederholbar und weniger anfällig für menschliche Fehler. 

**Behandeln Sie Konfiguration als Code**. Pflegen Sie Konfigurationsdateien in ein Versionskontrollsystem ein, sodass Sie Änderungen nachverfolgen und mit einer Version versehen und bei Bedarf ein Rollback durchführen können. 


<!-- links -->

[monitoring]: ../../best-practices/monitoring.md


