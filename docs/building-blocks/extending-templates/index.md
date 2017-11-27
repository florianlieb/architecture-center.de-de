---
title: Erweitern der Azure Resource Manager-Vorlagenfunktionen
description: "Beschreibt Tipps und Tricks für die Erweiterung der Azure Resource Manager-Vorlagenfunktionen"
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: d9cad6cfbfe8d42bcb21ef9e77504b80aef2d694
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="extend-azure-resource-manager-template-functionality"></a>Erweitern der Azure Resource Manager-Vorlagenfunktionen

2016 erstellte das AzureCAT Patterns & Practices-Team eine Reihe von [Vorlagenbausteinen](https://github.com/mspnp/template-building-blocks/wiki) für Azure Resource Manager mit dem Ziel, die Ressourcenbereitstellung zu vereinfachen. Jeder Baustein enthält eine Reihe vorgefertigter Vorlagen, mit denen Ressourcen bereitgestellt werden können, die durch separate Parameterdateien angegeben werden.

Die Bausteinvorlagen wurden so konzipiert, dass sie miteinander kombiniert werden können, um größere und komplexere Bereitstellungen zu ermöglichen. Die Bereitstellung eines virtuellen Computers in Azure beispielsweise erfordert ein virtuelles Netzwerk, Speicherkonten und weitere Ressourcen. Die [Bausteinvorlage für virtuelle Netzwerke](https://github.com/mspnp/template-building-blocks/wiki/VNet-(v1)) stellt ein virtuelles Netzwerk und die zugehörigen Subnetze bereit. Die [Bausteinvorlage für virtuelle Computer](https://github.com/mspnp/template-building-blocks/wiki/Windows-and-Linux-VMs-(v1)) stellt Speicherkonten, Netzwerkschnittstellen und die eigentlichen virtuellen Computer bereit. Sie können dann ein Skript oder eine Vorlage erstellen, um beide Bausteinvorlagen mit den entsprechenden Parameterdateien aufzurufen und so in einem einzigen Vorgang eine vollständige Architektur bereitzustellen.

Während der Entwicklung der Bausteinvorlagen entwarf das Patterns & Practices-Team verschiedene Konzepte, um die Azure Resource Manager-Vorlagenfunktionen zu erweitern. In dieser Serie beschreiben wir verschiedene dieser Konzepte, sodass Sie diese in Ihren eigenen Vorlagen verwenden können.

> [!NOTE]
> Für diese Artikel wird vorausgesetzt, dass Sie über erweiterte Kenntnisse zu Azure Resource Manager-Vorlagen verfügen.