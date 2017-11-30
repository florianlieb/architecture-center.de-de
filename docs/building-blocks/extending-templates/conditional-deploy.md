---
title: Bedingtes Bereitstellen einer Ressource in einer Azure Resource Manager-Vorlage
description: "Erfahren Sie, wie Sie die Funktionalität von Azure Resource Manager-Vorlagen erweitern, um eine bedingte Bereitstellung einer Ressource in Abhängigkeit vom Wert eines Parameters zu erzielen."
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: e911e7dc41b4f71ebfaf13a00f8cdbb5b4e2578b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a>Bedingtes Bereitstellen einer Ressource in einer Azure Resource Manager-Vorlage

Es gibt Situationen, in denen Sie Ihre Vorlage für die Bereitstellung einer Ressource auf Grundlage einer Bedingung, z.B. des Vorhandenseins eines Parameterwerts, entwerfen müssen. Ihre Vorlage kann beispielsweise ein virtuelles Netzwerk bereitstellen und Parameter enthalten, die andere virtuelle Netzwerke für das Peering angeben. Wenn Sie keine Parameterwerte für das Peering angegeben haben, möchten Sie nicht, dass Resource Manager die Peeringressource bereitstellt.

Um dies zu erreichen, verwenden Sie das [`condition`-Element][azure-resource-manager-condition] in der Ressource, um die Länge des Parameterarrays zu testen. Wenn die Länge 0 (null) ist, geben Sie `false` zurück, um die Bereitstellung zu verhindern. Bei sämtlichen Werten größer 0 geben Sie hingegen `true` zurück, um die Bereitstellung zu ermöglichen.

## <a name="example-template"></a>Beispielvorlage

Sehen wir uns eine Beispielvorlage an, die dies veranschaulicht. Unsere Vorlage verwendet das [`condition`-Element][azure-resource-manager-condition] zum Steuern der Bereitstellung der Ressource `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`. Diese Ressource erstellt ein Peering zwischen zwei virtuellen Azure-Netzwerken in derselben Region.

Werfen wir einen Blick auf die einzelnen Abschnitte der Vorlage.

Das `parameters`-Element definiert einen einzelnen Parameter mit dem Namen `virtualNetworkPeerings`: 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "virtualNetworkPeerings": {
      "type": "array",
      "defaultValue": []
    }
  },
```
Unser `virtualNetworkPeerings`-Parameter ist ein `array` mit dem folgenden Schema:

```json
"virtualNetworkPeerings": [
    {
        "remoteVirtualNetwork": {
            "name": "my-other-virtual-network"
        },
        "allowForwardedTraffic": true,
        "allowGatewayTransit": true,
        "useRemoteGateways": false
    }
]
```

Die Eigenschaften in unserem Parameter geben die [Einstellungen im Zusammenhang mit dem Peering virtueller Netzwerke][vnet-peering-resource-schema] an. Wir geben Werte für diese Eigenschaften an, wenn wir die Ressource `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` im Abschnitt `resources` angeben:

```json
"resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "[concat('vnp-', copyIndex())]",
      "condition": "[greater(length(parameters('virtualNetworkPeerings')), 0)]",
      "dependsOn": [
        "virtualNetworks"
      ],
      "copy": {
          "name": "iterator",
          "count": "[length(variables('peerings'))]",
          "mode": "serial"
      },
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
          },
          "variables": {
          },
          "resources": [
            {
              "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
              "apiVersion": "2016-06-01",
              "location": "[resourceGroup().location]",
              "name": "[variables('peerings')[copyIndex()].name]",
              "properties": "[variables('peerings')[copyIndex()].properties]"
            }
          ],
          "outputs": {
          }
        }
      }
    }
]
```
Es gibt mehrere Dinge, die in diesem Teil der Vorlage passieren. Zunächst einmal ist die eigentliche Ressource, die bereitgestellt wird, eine Inlinevorlage vom Typ `Microsoft.Resources/deployments`, die eine eigene Vorlage enthält, die `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` eigentlich bereitstellt.

Unser `name` für die Inlinevorlage wird durch Anhängen der aktuellen Iteration von `copyIndex()` mit dem Präfix `vnp-` eindeutig gemacht. 

Das `condition`-Element gibt an, dass unsere Ressource verarbeitet werden sollte, wenn die `greater()`-Funktion `true` ergibt. Hier testen wir, ob für das `virtualNetworkPeerings`-Parameterarray gilt: `greater()` als 0 (null). Wenn dies der Fall ist, lautet das Ergebnis `true`, und die `condition` ist erfüllt. Andernfalls ist das Ergebnis `false`.

Als Nächstes geben wir unsere `copy`-Schleife an. Es ist eine `serial`-Schleife, das bedeutet, dass die Schleife nacheinander ausgeführt wird, wobei jede Ressource wartet, bis die letzte Ressource bereitgestellt wurde. Die `count`-Eigenschaft gibt an, wie oft die Schleife durchlaufen wird. Hier legen wir normalerweise die Länge des `virtualNetworkPeerings`-Arrays fest, da es die Parameterobjekte enthält, die die Ressource, die wir bereitstellen möchten, angeben. Dabei tritt jedoch ein Fehler bei der Überprüfung auf, wenn das Array leer ist, da Resource Manager bemerkt, dass wir versuchen, auf Eigenschaften zuzugreifen, die nicht vorhanden sind. Wir können dies jedoch umgehen. Werfen wir einen Blick auf die Variablen, die wir benötigen:

```json
  "variables": {
    "workaround": {
       "true": "[parameters('virtualNetworkPeerings')]",
       "false": [{
           "name": "workaround",
           "properties": {}
       }]
     },
     "peerings": "[variables('workaround')[string(greater(length(parameters('virtualNetworkPeerings')), 0))]]"
  },
```

Unsere `workaround`-Variable enthält die beiden Eigenschaften `true` und `false`. Die `true`-Eigenschaft wird anhand des Werts des `virtualNetworkPeerings`-Parameterarrays ausgewertet. Die `false`-Eigenschaft wird in ein leeres Objekt ausgewertet. Dies schließt auch die benannten Eigenschaften ein, die Resource Manager erwartet – beachten Sie, dass `false` tatsächlich ein Array ist (ebenso wie unser `virtualNetworkPeerings`-Parameter), sodass die Überprüfung erfüllt wird. 

Unsere `peerings`-Variable verwendet unsere `workaround`-Variable, indem sie erneut testet, ob die Länge des `virtualNetworkPeerings`-Parameterarrays größer als 0 (null) ist. Wenn dies der Fall, wird `string` als `true` und die `workaround`-Variable als `virtualNetworkPeerings`-Parameterarray ausgewertet. Andernfalls lautet das Ergebnis `false`, und die `workaround`-Variable wird zu einem leeren Objekt ausgewertet, das das erste Element des Arrays ist.

Nachdem wir das Überprüfungsproblem umgangen haben, können wir einfach die Bereitstellung der `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`-Ressource in der geschachtelten Vorlage angeben, indem wir `name` und `properties` aus unserem `virtualNetworkPeerings`-Parameterarray übergeben. Sie können dies auch im `template`-Element erkennen, das im `properties`-Element unserer Ressource geschachtelt ist.

## <a name="next-steps"></a>Nächste Schritte

* Dieses Verfahren ist im [Vorlagenbaustein-Projekt](https://github.com/mspnp/template-building-blocks) und in den [Azure-Referenzarchitekturen](/azure/architecture/reference-architectures/) implementiert. Sie können es verwenden, um Ihre eigene Architektur zu erstellen oder eine unserer Referenzarchitekturen bereitzustellen.

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-group-authoring-templates#resources
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings