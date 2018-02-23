---
title: "Erläuterungen: Was ist eine Azure-Ressourcengruppe?"
description: "Hier wird die interne Azure-Funktion einer Ressourcengruppe erläutert."
author: petertay
ms.openlocfilehash: e7c7334bd88c28f57498486bd2bed3c349565222
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/09/2018
---
# <a name="what-is-an-azure-resource-group"></a>Was ist eine Azure-Ressourcengruppe?

In dem erläuternden Artikel [Was ist Azure Resource Manager?](resource-manager-explainer.md) haben Sie erfahren, dass Azure Resource Manager einen **Ressourcengruppenbezeichner** benötigt, wenn ein Aufruf zum Erstellen, Lesen, Aktualisieren oder Löschen einer Ressource ausgeführt wird. Diese Ressourcengruppen-ID verweist auf eine **Ressourcengruppe**. Bei einer Ressourcengruppe handelt es sich einfach um einen Bezeichner, den Azure Resource Manager auf Ressourcen anwendet, um sie zu gruppieren. Diese Ressourcengruppen-ID ermöglicht Azure Resource Manager die Ausführung von Vorgängen für eine Gruppe von Ressourcen mit dieser ID.

Beispiel: Ein Benutzer kann einen Aufruf zum **Löschen** an eine Azure Resource Manager-RESTful-API senden und dabei die Ressourcengruppen-ID ohne eine bestimmte Ressourcen-ID angeben. Azure Resource Manager fragt eine interne Azure-Datenbank auf alle Ressourcen mit der angegebenen Ressourcengruppen-ID ab und ruft die RESTful-API auf, um die einzelnen Ressourcen zu löschen.

Eine Ressourcengruppe darf keine Ressourcen aus verschiedenen Abonnements enthalten. Der Grund dafür ist, dass zwischen der Mandanten-ID und der Abonnement-ID eine 1:n-Beziehung besteht – mehrere Abonnements können dem gleichen Mandanten für die Bereitstellung von Authentifizierung und Autorisierung vertrauen, jedes Abonnement kann jedoch nur einem Mandanten vertrauen. Zwischen Abonnement-ID und Ressourcengruppen-ID besteht ebenfalls eine 1:n-Beziehung – mehrere Ressourcengruppen können zum gleichen Abonnement gehören, aber jede Ressourcengruppe kann jeweils nur zu einem Abonnement gehören. Weiterhin besteht zwischen Ressourcengruppen-ID und Ressourcen-ID eine 1:n-Beziehung – eine einzelne Ressourcengruppe kann mehrere Ressourcen enthalten, aber jede Ressource kann nur zu einer einzelnen Ressourcengruppe gehören.

## <a name="next-steps"></a>Nächste Schritte

* Hier haben Sie mehr über Azure-Ressourcengruppen erfahren. Lesen Sie nun grundlegende Informationen zum [Beschränken des Zugriffs auf Ressourcen](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json). Dieser Schritt gehört zwar nicht zur grundlegenden Einführungsphase, ist aber für die mittlere Einführungsphase relevant. Anschließend können Sie [Ihre erste Ressourcengruppe erstellen](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json) und sich den [Entwurfsleitfaden für Azure-Ressourcengruppen](resource-group.md) ansehen.
