---
title: 'Einführung von Azure: Grundlegende Phase'
description: Hier wird der grundlegende Wissensstand beschrieben, den ein Unternehmen für die Einführung von Azure benötigt.
author: petertay
ms.openlocfilehash: e9421b610e4eb07a3ed37bca56e513b0689484ef
ms.sourcegitcommit: 9ba82cf84cee06ccba398ec04c51dab0e1ca8974
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/13/2018
---
# <a name="adopting-azure-foundational"></a>Einführung von Azure: Grundlegende Phase

Die Einführung von Azure ist die erste Phase des Organisationsreifegrads eines Unternehmens. Am Ende dieser Phase können Personen in Ihrer Organisation einfache Workloads in Azure bereitstellen.

Die nachfolgende Liste enthält die Aufgaben zum Abschließen der grundlegenden Einführungsphase. Die Liste ist fortlaufend aufgebaut, die Aufgaben müssen also in der angegebenen Reihenfolge ausgeführt werden. Wenn Sie eine Aufgabe abgeschlossen haben, fahren Sie mit der nächsten Aufgabe in der Liste fort. 

1. Ausführliche Informationen zu Azure:
    - **Erläuterungen:** [Wie funktioniert Azure?](azure-explainer.md)
2. Grundlegendes zu digitalen Unternehmensidentitäten in Azure
    - **Erläuterungen:** [Was ist ein Azure Active Directory-Mandant?](tenant-explainer.md)
    - **Vorgehensweise:** [Einrichten eines Azure Active Directory-Mandanten](/azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Anleitung:** [Leitfaden zum Entwerfen eines Azure AD-Mandanten](tenant.md)
    - **Vorgehensweise:** [Hinzufügen neuer Benutzer zu Azure Active Directory](/azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json)    
3. Ausführliche Informationen zu Abonnements in Azure:
    - **Erläuterungen:** [Was ist ein Azure-Abonnement?](subscription-explainer.md)
    - **Anleitung:** [Entwerfen eines Azure-Abonnements](subscription.md)
4. Ausführliche Informationen zur Ressourcenverwaltung in Azure: 
    - **Erläuterungen:** [Was ist Azure Resource Manager?](resource-manager-explainer.md)
    - **Erläuterungen:** [Was ist eine Azure-Ressourcengruppe?](resource-group-explainer.md)
    - **Erläuterungen:** [Grundlegendes zum Zugriff auf Ressourcen in Azure](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Vorgehensweise:** [Erstellen einer Azure-Ressourcengruppe mit dem Azure-Portal](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Anleitung:** [Leitfaden zum Entwerfen einer Azure-Ressourcengruppe](resource-group.md)
    - **Anleitungen:** [Namenskonventionen für Azure-Ressourcen](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json)
5. Bereitstellen einer einfachen Azure-Architektur:
    - In der [Übersicht über Azure-Compute-Optionen](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json) finden Sie Informationen zu verschiedenen Azure-Computeoptionen wie Infrastructure-as-a-Service (IaaS) und Platform-as-a-Service (PaaS).
    - Sie haben grundlegende Informationen zu den verschiedenen Arten von Azure-Computeoptionen erhalten. Wählen Sie nun eine PaaS-Webanwendung oder einen virtuellen IaaS-Computer als erste Ressource in Azure:
    - PaaS: Einführung in Platform-as-a-Service:
        - **Vorgehensweise:** [Bereitstellen einer einfachen Webanwendung in Azure](/azure/app-service/app-service-web-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Anleitung:** Bewährte Methoden zum Bereitstellen einer [einfachen Webanwendung](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) in Azure
    - IaaS: Einführung in virtuelle Netzwerke:
        - **Erläuterungen:** [Virtuelles Azure-Netzwerk](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Vorgehensweise:** [Bereitstellen eines virtuellen Netzwerks in Azure mit dem Portal](/azure/virtual-network/virtual-networks-create-vnet-arm-pportal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - IasS: Bereitstellen einer einzelnen VM-Workload (Windows und Linux):
        - **Vorgehensweise:** [Bereitstellen eines virtuellen Windows-Computers in Azure mit dem Portal](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Anleitungen:** [Bewährte Methoden für das Ausführen eines virtuellen Windows-Computers in Azure](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Vorgehensweise:** [Bereitstellen eines virtuellen Linux-Computers in Azure mit dem Portal](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Anleitungen:** [Bewährte Methoden für das Ausführen eines virtuellen Linux-Computers in Azure](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
