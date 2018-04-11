---
title: 'Erläuterungen: Was ist Azure Resource Manager?'
description: Hier wird die interne Funktionsweise von Azure Resource Manager erläutert.
author: petertay
ms.openlocfilehash: 60f09901bdc4b292abd73335b78c7d56a76f27a6
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-azure-resource-manager"></a>Erläuterungen: Was ist Azure Resource Manager?

In dem erläuternden Artikel [Wie funktioniert Azure?](azure-explainer.md) haben Sie die interne Architektur von Azure kennengelernt. Diese Architektur umfasst ein Front-End zum Hosten der verteilten Anwendungen, die interne Azure-Dienste verwalten.

Das Azure-Front-End beinhaltet den Dienst Azure Resource Manager. Azure Resource Manager ist für den gesamten Lebenszyklus von in Azure gehosteten Ressourcen verantwortlich – von ihrer Erstellung bis hin zur Löschung. Für die Interaktion mit Azure Resource Manager stehen verschiedene Tools zur Verfügung: Powershell, die Azure-Befehlszeilenschnittstelle, SDKs usw. Alle diese Tools sind jedoch lediglich Wrapper für eine von Azure Resource Manager gehostete RESTful-API.

Die von Azure Resource Manager bereitgestellte RESTful-API ist eine konsistente Schnittstelle, die von einer Reihe von **Ressourcenanbietern** zur Verfügung gestellt wird. Ressourcenanbieter sind einfach Azure-Dienste, die Ressourcen in Azure erstellen, lesen, aktualisieren und löschen. Die RESTful-API enthält Methoden für diese einzelnen Funktionen. 

Die RESTful-API benötigt ein Zugriffstoken für den Benutzer, eine **Abonnement-ID** und etwas Neues: eine **Ressourcengruppen-ID**. Ressourcengruppen werden im erläuternden Artikel zu [Ressourcengruppen](resource-group-explainer.md) beschrieben. Azure Resource Manager benötigt darüber hinaus die **Mandanten-ID**, die als Teil des Zugriffstoken codiert ist. 

Wenn ein gültiger API-Aufruf zum Erstellen einer Ressource empfangen wird, sucht Azure Resource Manager nach Kapazität in der angegebenen Region und kopiert alle erforderlichen Dateien an einen Stagingspeicherort. Die Anforderung wird dann an den Fabric Controller im Rack gesendet, und der Fabric Controller ordnet die Ressourcen zu. Der Fabric Controller antwortet auf die Anforderung mit einer Erfolgs- oder Fehlermeldung sowie einer **Ressourcen-ID** für die neu erstellte Ressource. Diese vier IDs werden intern in Azure gespeichert und dienen zusammen als eindeutiger Bezeichner für eine bereitgestellte Ressource.

## <a name="next-steps"></a>Nächste Schritte

* Nachdem Sie nun eine Vorstellung von der internen Funktionsweise von Azure Resource Manager haben, lesen Sie den Artikel zu [Ressourcengruppen](resource-group-explainer.md), der Sie beim Erstellen Ihrer ersten Ressourcengruppe unterstützt.
