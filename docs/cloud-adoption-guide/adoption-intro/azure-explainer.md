---
title: "Erläuterungen: Wie funktioniert Azure?"
description: "Enthält eine Beschreibung der internen Funktionsweise von Azure."
author: petertay
ms.openlocfilehash: 847d24b7057d80f3d34aac7900cfb64fec60a640
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-how-does-azure-work"></a>Erläuterungen: Wie funktioniert Azure?

Azure ist die öffentliche Cloudplattform von Microsoft. Azure umfasst eine große Sammlung von Diensten, z.B. Platform-as-a-Service (PaaS), Infrastructure-as-a-Service (IaaS), Database-as-a-Service (DBaaS) und viele andere. Aber was ist Azure genau, und wie funktioniert es?

Wie andere Cloudplattformen auch, basiert Azure auf einer Technologie, die als **Virtualisierung** bezeichnet wird. Ein Großteil der Computerhardware kann per Software emuliert werden, da es sich bei Computerhardware in den meisten Fällen lediglich um einen Satz mit Anweisungen handelt, die permanent oder semipermanent in Silizium codiert sind. Indem eine Emulationsebene verwendet wird, mit der Softwareanweisungen Hardwareanweisungen zugeordnet werden, kann virtualisierte Hardware so per Software ausgeführt werden, als ob es sich wirklich um Hardware handeln würde.

Im Wesentlichen umfasst die Cloud eine Gruppe von physischen Servern in einem oder mehreren Rechenzentren, auf denen virtualisierte Hardware im Namen der Kunden ausgeführt wird. Wie wird es also für die Cloud erreicht, dass Millionen von Instanzen virtualisierter Hardware für Millionen von Kunden gleichzeitig erstellt, gestartet, beendet und gelöscht werden können?

Wir sehen uns die Architektur der Hardware im Rechenzentrum an.  Jedes Rechenzentrum enthält eine Sammlung mit Servern, die in Serverracks angeordnet sind. Jedes Serverrack enthält viele Server**blades** sowie einen Netzwerkswitch für die Netzwerkkonnektivität und eine Stromversorgungseinheit für die Stromversorgung. Racks werden auch in größeren Einheiten gruppiert, die als **Cluster** bezeichnet werden. 

Innerhalb jedes Racks oder Clusters haben die meisten Server die Aufgabe, diese virtualisierten Hardwareinstanzen im Namen des Benutzers auszuführen. Auf einigen Servern wird aber Software für die Cloudverwaltung ausgeführt, die als Fabric Controller bezeichnet wird. Der **Fabric Controller** ist eine verteilte Anwendung mit vielen Aufgaben. Er dient zum Zuordnen von Diensten, Überwachen der Integrität des Servers und der darauf ausgeführten Dienste und Wiederherstellen der Serverintegrität nach einem Ausfall.

Jede Instanz des Fabric Controllers ist mit einem anderen Satz von Servern verbunden, auf denen Software für die Cloudorchestrierung ausgeführt wird, die normalerweise als **Front-End** bezeichnet wird. Auf dem Front-End werden die Webdienste, RESTful-APIs und internen Azure-Datenbanken gehostet, die für alle Funktionen der Cloud verwendet werden. 

Beispielsweise hostet das Front-End die Dienste, mit denen Kundenanforderungen zur Zuordnung von Azure-Ressourcen verarbeitet werden, z.B. [virtuelle Netzwerke][vnet], [virtuelle Computer][vms] und Dienste wie [CosmosDB]. Zuerst überprüft das Front-End den Benutzer und stellt sicher, dass der Benutzer zur Zuordnung der angeforderten Ressourcen berechtigt ist. Wenn die Berechtigung vorhanden ist, zieht das Front-End eine Datenbank heran, um ein Serverrack mit ausreichender Kapazität zu ermitteln. Anschließend weist es den Fabric Controller im Rack an, die Ressource zuzuordnen.

Einfach ausgedrückt: Azure ist eine große Sammlung von Servern und Netzwerkhardware und verfügt über einen komplexen Satz mit verteilten Anwendungen, die die Konfiguration und den Betrieb der virtualisierten Hardware und Software auf diesen Servern orchestrieren. Diese Orchestrierung macht Azure so leistungsstark. Benutzer müssen sich nicht mehr um das Pflegen und Aktualisieren der Hardware kümmern, sondern dies wird von Azure im Hintergrund durchgeführt. 

## <a name="next-steps"></a>Nächste Schritte

* Nachdem Sie jetzt die interne Funktionsweise von Azure verstanden haben, besteht der erste Schritt zur Einführung von Azure darin, sich über die [digitale Identität in Azure](tenant-explainer.md) zu informieren. Anschließend können Sie [Ihren ersten Benutzer in Azure AD erstellen][docs-add-users-to-aad].

<!-- Links -->

[cosmosdb]: /azure/cosmos-db/introduction
[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[vms]: /azure/virtual-machines/
[vnet]: /azure/virtual-network/virtual-networks-overview