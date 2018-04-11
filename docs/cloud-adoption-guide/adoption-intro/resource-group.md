---
title: 'Anleitung: Entwerfen einer Azure-Ressourcengruppe'
description: Leitfaden für das Entwerfen von Azure-Ressourcengruppen im Rahmen einer Strategie für die grundlegende Cloudeinführung
author: petertay
ms.openlocfilehash: ac6cbb03be8cdba020641d3b9034ad9d20101acf
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/09/2018
---
# <a name="guidance-azure-resource-group-design"></a>Anleitung: Entwerfen einer Azure-Ressourcengruppe

Eine [Ressourcengruppe](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#resource-groups) in Azure ist ein logischer Container, in dem Ressourcen zusammengefasst werden. Jede in Azure bereitgestellte Ressource muss in einer einzelnen Ressourcengruppe bereitgestellt werden.

## <a name="design-considerations"></a>Überlegungen zum Entwurf

- Alle Ressourcen einer Ressourcengruppe sollten über den gleichen Lebenszyklus verfügen. Das heißt für Sie, dass Sie Ressourcen mit dem gleichen Lebenszyklus als Gruppe bereitstellen, aktualisieren und löschen sollten. Beispiel: Die Computeressourcen einer Webanwendung werden üblicherweise als eine Einheit bereitgestellt. Eine Datenbank, die auch von anderen Webanwendungen verwendet wird, wird jedoch wahrscheinlich in einem anderen Lebenszyklus verwaltet und sollte daher in einer eigenen Ressourcengruppe enthalten sein.
- Eine Ressourcengruppe kann Ressourcen enthalten, die sich in unterschiedlichen Regionen befinden.
- Alle Ressourcen in einer Ressourcengruppe müssen einem einzelnen Abonnement zugeordnet sein. 
- Eine Ressource kann zwischen Ressourcengruppen, jedoch nicht in eine Ressourcengruppe verschoben werden, die eine in einem anderen Abonnement bereitgestellte Ressource enthält.
- Die Gruppenzuordnung einer Ressource wirkt sich nicht auf die Verbindung oder die Interaktion mit Ressourcen in anderen Ressourcengruppen aus. Ein virtueller Computer, der einer Ressourcengruppe zugewiesen ist, kann beispielsweise eine Verbindung mit einer Datenbank herstellen, die einer anderen Ressourcengruppe zugewiesen ist, sofern zwischen ihnen eine Netzwerkverbindung besteht.
- Eine Ressourcengruppe kann zum Festlegen der Zugriffssteuerung für administrative Aktionen verwendet werden. Sie können RBAC-Berechtigungen (Role-Based Access Control, rollenbasierte Zugriffssteuerung) auf Abonnementebene oder auf Ressourcengruppenebene anwenden. Alle auf Abonnementebene zugewiesenen Berechtigungen werden auf Ressourcengruppenebene geerbt. In den mittleren und fortgeschrittenen Einführungsphasen erfahren Sie mehr über RBAC und Ressourcenberechtigungen.

## <a name="proven-practices"></a>Bewährte Methoden

- In der grundlegenden Phase verwalten Sie wahrscheinlich nur wenige POC-Projekte (Proof of Concept), die jeweils nur eine geringe Anzahl von Ressourcen enthalten. Da POC-Ressourcen in der Regel den gleichen Lebenszyklus haben, können Sie eine Ressourcengruppe für jedes dieser Projekte erstellen.
- In der mittleren Einführungsphase verwalten Sie mehrere Projekte. Für andere Projektarten sind andere Ressourcengruppenentwürfe unter Umständen besser geeignet. Wenn Sie ursprüngliche POC-Projekte in die Produktion heraufstufen möchten, können Sie Ressourcen in eine andere Ressourcengruppe verschieben, wenn diese zum gleichen Abonnement gehört. Daher sollten Sie in dieser Phase diese Ressourcen im gleichen Abonnement bereitstellen, damit Sie Ressourcen in Zukunft neu organisieren können.

## <a name="next-steps"></a>Nächste Schritte

* Nachdem Sie hier die bewährten Methoden für die grundlegende Einführungsphase kennengelernt haben, können Sie Ressourcengruppen erstellen und ihnen Ressourcen hinzufügen. In dieser Phase verwalten Sie eine geringe Anzahl von Ressourcen. Die Verwaltung wird jedoch komplexer, wenn Sie weitere Ressourcen hinzufügen. Unter [Benennungskonventionen](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json) finden Sie Informationen zum Benennen und Kennzeichnen von Ressourcen zur Vorbereitung auf die mittlere Einführungsphase.
