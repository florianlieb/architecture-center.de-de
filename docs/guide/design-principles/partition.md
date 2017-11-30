---
title: Umgehung von Grenzwerten durch Partitionierung
description: Einsetzen der Partitionierung, um Datenbank-, Netzwerk- und Computegrenzwerte zu umgehen
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 4371490385b24230551bf17db0075052f320b574
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="partition-around-limits"></a>Umgehung von Grenzwerten durch Partitionierung

## <a name="use-partitioning-to-work-around-database-network-and-compute-limits"></a>Einsetzen der Partitionierung, um Datenbank-, Netzwerk- und Computegrenzwerte zu umgehen

In der Cloud ist für alle Dienste die Fähigkeit zum zentralen Hochskalieren beschränkt. Grenzwerte für Azure-Dienste sind in [Einschränkungen für Azure-Abonnements und Dienste, Kontingente und Einschränkungen][azure-limits] dokumentiert. Zu den Grenzwerten gehört die Anzahl der Kerne, die Datenbankgröße, der Abfragedurchsatz und der Netzwerkdurchsatz. Wenn Ihr System ausreichend groß wird, können Sie einen oder mehrere dieser Grenzwerte erreichen. Nutzen Sie die Partitionierung zum Umgehen dieser Grenzwerte.

Es gibt viele Möglichkeiten, ein System zu partitionieren, Beispiele:

- Partitionieren Sie eine Datenbank, um Grenzwerte für die Datenbankgröße, Daten-E/A oder die Anzahl gleichzeitiger Sitzungen zu vermeiden.

- Partitionieren Sie eine Warteschlange oder einen Nachrichtenbus, um Grenzwerte für die Anzahl von Anforderungen oder die Anzahl von gleichzeitigen Verbindungen zu vermeiden.

- Partitionieren Sie eine App Service-Web-App, um Grenzwerte für die Anzahl der Instanzen pro App Service-Plan zu vermeiden. 

Eine Datenbank kann *horizontal*, *vertikal* oder *funktional* partitioniert werden.

- Bei der horizontalen Partitionierung, die auch als Sharding bezeichnet wird, enthält jede Partition Daten für eine Teilmenge des gesamten Datasets. Die Partitionen nutzen das gleiche Datenschema. Beispiel: Kunden, deren Namen mit A&ndash;M beginnen, kommen in eine Partition, und die mit N&ndash;Z in eine andere Partition.

- Bei der vertikalen Partitionierung enthält jede Partition eine Teilmenge der Felder für die Elemente im Datenspeicher. Beispiel: Nehmen Sie häufig verwendete Felder in eine Partition und weniger häufig verwendete Felder in eine andere Partition auf.

- Bei der funktionalen Partitionierung werden Daten nach der Art und Weise partitioniert, in der sie im System von jedem begrenzten Kontext verwendet werden. Beispiel: Speichern Sie Rechnungsdaten in einer Partition und Daten zum Produktbestand in einer anderen. Die Schemas sind voneinander unabhängig.

Ausführlichere Anleitungen finden Sie unter [Datenpartitionierung][data-partitioning-guidance].

## <a name="recommendations"></a>Recommendations

**Partitionieren Sie verschiedene Teile der Anwendung**. Datenbanken sind ein offensichtlicher Kandidat für die Partitionierung, aber Sie könnten auch Speicher, Cache, Warteschlangen und Serverinstanzen partitionieren.

**Entwerfen Sie den Partitionsschlüssel, um Hotspots zu vermeiden**. Wenn Sie eine Datenbank partitionieren, aber ein Shard noch immer den Großteil der Anforderungen erhält, ist das Problem nicht gelöst. Im Idealfall wird die Last gleichmäßig auf alle Partitionen verteilt. Verwenden Sie z.B. einen Hash der Kunden-ID und nicht den ersten Buchstaben des Kundennamens, da einige Buchstaben häufiger vorkommen. Das gleiche Prinzip gilt beim Partitionieren einer Nachrichtenwarteschlange. Wählen Sie einen Partitionsschlüssel aus, der zu einer gleichmäßigen Verteilung von Nachrichten über den Satz von Warteschlangen führt. Weitere Informationen finden Sie unter [Sharding][sharding].

**Umgehen Sie die Grenzwerte für Azure-Abonnements und -Dienste durch Partitionierung**. Für einzelne Komponenten und Dienste gelten Grenzwerte, aber es gibt auch Grenzwerte für Abonnements und Ressourcengruppen. Bei sehr große Anwendungen müssen Sie möglicherweise diese Grenzwerte durch Partitionierung umgehen.  

**Partitionieren Sie auf unterschiedlichen Ebenen**. Nehmen Sie einen Datenbankserver, der auf einer VM bereitgestellt wurde. Die VM weist eine VHD auf, die von Azure Storage gesichert wird. Das Speicherkonto gehört zu einem Azure-Abonnement. Für jeden Schritt in der Hierarchie gelten Grenzwerte. Der Datenbankserver weist möglicherweise einen Grenzwert für Verbindungspools auf. Für VMs gelten CPU- und Netzwerkgrenzwerte. Für den Speicher gelten IOPS-Grenzwerte. Für das Abonnement ist die Anzahl der VM-Kerne begrenzt. Im Allgemeinen ist es einfacher, die Partitionierung eher unten in der Hierarchie durchzuführen. Nur bei großen Anwendungen sollte eine Partitionierung auf Abonnementebene erforderlich sein. 

<!-- links -->

[azure-limits]: /azure/azure-subscription-service-limits
[data-partitioning-guidance]: ../../best-practices/data-partitioning.md
[sharding]: ../../patterns/sharding.md

 