---
title: Verwenden eines Objekts als Parameter in einer Azure Resource Manager-Vorlage
description: "Beschreibt das Erweitern der Funktionalität der Azure Resource Manager-Vorlagen für die Verwendung von Objekten als Parameter."
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 08ee1cf2924f78ce366c58e20e84a512785f85cc
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a>Verwenden eines Objekts als Parameter in einer Azure Resource Manager-Vorlage

Beim [Erstellen von Azure Resource Manager-Vorlagen][azure-resource-manager-create-template] können Sie entweder Ressourceneigenschaftswerte direkt in der Vorlage angeben oder einen Parameter definieren und die Werte während der Bereitstellung angeben. Es kann ein Parameter für jeden Eigenschaftswert verwendet werden, aber es gilt ein Grenzwert von 255 Parametern pro Bereitstellung. Wenn Sie größere und komplexere Bereitstellungen verwenden, gehen Ihnen ggf. die Parameter aus.

Eine Möglichkeit, dieses Problem zu beheben, ist die Verwendung eines Objekts als Parameter anstatt als Wert. Definieren Sie hierfür den Parameter in Ihrer Vorlage, und geben Sie ein JSON-Objekt an, anstatt einen einzelnen Wert während der Bereitstellung. Verweisen Sie anschließend auf die Untereigenschaften des Parameters, indem Sie die [`parameter()`-Funktion][azure-resource-manager-functions] und den Punktoperator in Ihrer Vorlage verwenden.

Wir sehen uns ein Beispiel an, in dem eine virtuelle Netzwerkressource bereitgestellt wird. Zuerst geben wir in der Vorlage den Parameter `VNetSettings` an und legen `type` auf `object` fest:

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
Als Nächstes geben wir Werte für das Objekt `VNetSettings` an:

> [!NOTE]
> Informationen zum Angeben von Parameterwerten während der Bereitstellung finden Sie im Abschnitt **parameters** unter [Verstehen der Struktur und Syntax von Azure Resource Manager-Vorlagen][azure-resource-manager-authoring-templates]. 

```json
"parameters":{
    "VNetSettings":{
        "value":{
            "name":"VNet1",
            "addressPrefixes": [
                {
                    "name": "firstPrefix",
                    "addressPrefix": "10.0.0.0/22"
                }
            ],
            "subnets":[
                {
                    "name": "firstSubnet",
                    "addressPrefix": "10.0.0.0/24"
                },
                {
                    "name":"secondSubnet",
                    "addressPrefix":"10.0.1.0/24"
                }
            ]
        }
    }
}
```

Sie sehen, dass der einzelne Parameter drei Untereigenschaften angibt: `name`, `addressPrefixes` und `subnets`. Mit jeder dieser Untereigenschaften werden entweder ein Wert oder andere Untereigenschaften angegeben. Das Ergebnis ist, dass mit dem einzelnen Parameter alle Werte angegeben werden, die zum Bereitstellen des virtuellen Netzwerks benötigt werden.

Nun sehen wir uns den Rest der Vorlage an, um zu ermitteln, wie das Objekt `VNetSettings` verwendet wird:

```json
...
"resources": [
    {
        "apiVersion": "2015-06-15",
        "type": "Microsoft.Network/virtualNetworks",
        "name": "[parameters('VNetSettings').name]",
        "location":"[resourceGroup().location]",
        "properties": {
          "addressSpace":{
              "addressPrefixes": [
                "[parameters('VNetSettings').addressPrefixes[0].addressPrefix]"
              ]
          },
          "subnets":[
              {
                  "name":"[parameters('VNetSettings').subnets[0].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[0].addressPrefix]"
                  }
              },
              {
                  "name":"[parameters('VNetSettings').subnets[1].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[1].addressPrefix]"
                  }
              }
          ]
        }
    }
  ]
```
Die Werte des Objekts `VNetSettings` werden auf die Eigenschaften angewendet, die für unsere virtuelle Netzwerkressource erforderlich sind, indem die Funktion `parameters()` mit dem `[]`-Arrayindexer und dem Punktoperator verwendet wird. Dieser Ansatz funktioniert, wenn Sie nur die Werte des Parameterobjekts statisch auf die Ressource anwenden möchten. Falls Sie aber ein Array mit Eigenschaftswerten während der Bereitstellung dynamisch zuweisen möchten, können Sie eine [Kopierschleife][azure-resource-manager-create-multiple-instances] verwenden. Für die Nutzung einer Kopierschleife geben Sie ein JSON-Array mit Ressourceneigenschaftswerten an, und die Kopierschleife wendet die Werte dynamisch auf die Eigenschaften der Ressource an. 

Bei Verwendung des dynamischen Ansatzes sollte ein mögliches Problem beachtet werden. Um das Problem zu demonstrieren, sehen wir uns ein typisches Array mit Eigenschaftswerten an. In diesem Beispiel werden die Werte für unsere Eigenschaften in einer Variablen gespeichert. Beachten Sie, dass wir hier zwei Arrays verwenden: ein Array mit dem Namen `firstProperty` und ein Array mit dem Namen `secondProperty`. 

```json
"variables": {
    "firstProperty": [
        {
            "name": "A",
            "type": "typeA"
        },
        {
            "name": "B",
            "type": "typeB"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ],
    "secondProperty": [
        "one","two", "three"
    ]
}
```

Als Nächstes sehen wir uns an, wie wir auf die Eigenschaften in der Variablen mit einer Kopierschleife zugreifen.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    ...
    "copy": {
        "name": "copyLoop1",
        "count": "[length(variables('firstProperty'))]"
    },
    ...
    "properties": {
        "name": { "value": "[variables('firstProperty')[copyIndex()].name]" },
        "type": { "value": "[variables('firstProperty')[copyIndex()].type]" },
        "number": { "value": "[variables('secondProperty')[copyIndex()]]" }
    }
}
```

Die Funktion `copyIndex()` gibt die aktuelle Iteration der Kopierschleife zurück, und wir verwenden diese Rückgabe gleichzeitig als Index für beide Arrays.

Dies funktioniert gut, wenn die beiden Arrays die gleiche Länge haben. Das Problem tritt auf, wenn Sie einen Fehler gemacht haben und die beiden Arrays eine unterschiedliche Länge aufweisen. In diesem Fall besteht Ihre Vorlage die Überprüfung während der Bereitstellung nicht. Sie können dieses Problem vermeiden, indem Sie Ihre gesamten Eigenschaften in ein einzelnes Objekt einbinden, weil Sie dann viel einfacher erkennen können, wenn ein Wert fehlt. Unten ist ein weiteres Parameterobjekt angegeben, bei dem jedes Element des Arrays `propertyObject` eine Vereinigung der vorherigen Arrays `firstProperty` und `secondProperty` ist.

```json
"variables": {
    "propertyObject": [
        {
            "name": "A",
            "type": "typeA",
            "number": "one"
        },
        {
            "name": "B",
            "type": "typeB",
            "number": "two"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ]
}
```

Fällt Ihnen das dritte Element im Array auf? Dafür fehlt die `number`-Eigenschaft, aber das Fehlen ist viel leichter zu erkennen, wenn Sie die Parameterwerte auf diese Weise erstellen.

## <a name="using-a-property-object-in-a-copy-loop"></a>Verwenden eines Eigenschaftsobjekts in einer Kopierschleife

Dieser Ansatz ist noch nützlicher, wenn er mit der [seriellen Kopierschleife][azure-resource-manager-create-multiple] kombiniert wird, besonders beim Bereitstellen von untergeordneten Ressourcen. 

Zur Verdeutlichung sehen wir uns eine Vorlage an, mit der eine [Netzwerksicherheitsgruppe (NSG)][nsg] mit zwei Sicherheitsregeln bereitgestellt wird. 

Zuerst sehen wir uns die Parameter an. In der Vorlage ist zu erkennen, dass wir einen Parameter mit dem Namen `networkSecurityGroupsSettings` definiert haben, der ein Array mit dem Namen `securityRules` enthält. Dieses Array enthält zwei JSON-Objekte, die einige Einstellungen für eine Sicherheitsregel angeben.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters":{ 
      "networkSecurityGroupsSettings": {
      "value": {
          "securityRules": [
            {
              "name": "RDPAllow",
              "description": "allow RDP connections",
              "direction": "Inbound",
              "priority": 100,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.0.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "3389",
              "access": "Allow",
              "protocol": "Tcp"
            },
            {
              "name": "HTTPAllow",
              "description": "allow HTTP connections",
              "direction": "Inbound",
              "priority": 200,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.1.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "80",
              "access": "Allow",
              "protocol": "Tcp"
            }
          ]
        }
      }
    }
  }
```

Als Nächstes sehen wir uns unsere Vorlage an. Mit der ersten Ressource mit dem Namen `NSG1` wird die NSG bereitgestellt. Die zweite Ressource mit dem Namen `loop-0` erfüllt zwei Funktionen: Erstens ist sie von der NSG abhängig (`dependsOn`), sodass ihre Bereitstellung erst beginnt, wenn `NSG1` abgeschlossen ist, und zweitens ist sie die erste Iteration der sequenziellen Schleife. Die dritte Ressource ist eine geschachtelte Vorlage, die unsere Sicherheitsregeln wie im letzten Beispiel unter Verwendung eines Objekts für die Parameterwerte bereitstellt.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "networkSecurityGroupsSettings": {"type":"object"}
  },
  "variables": {},
  "resources": [
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "NSG1",
      "location":"[resourceGroup().location]",
      "properties": {
          "securityRules":[]
      }
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "loop-0",
        "dependsOn": [
            "NSG1"
        ],
        "properties": {
            "mode":"Incremental",
            "parameters":{},
            "template": {
                "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                "contentVersion": "1.0.0.0",
                "parameters": {},
                "variables": {},
                "resources": [],
                "outputs": {}
            }
        }       
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "[concat('loop-', copyIndex(1))]",
        "dependsOn": [
          "[concat('loop-', copyIndex())]"
        ],
        "copy": {
          "name": "iterator",
          "count": "[length(parameters('networkSecurityGroupsSettings').securityRules)]"
        },
        "properties": {
          "mode": "Incremental",
          "template": {
            "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
           "parameters": {},
            "variables": {},
            "resources": [
                {
                    "name": "[concat('NSG1/' , parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].name)]",
                    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
                    "apiVersion": "2016-09-01",
                    "location":"[resourceGroup().location]",
                    "properties":{
                        "description": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].description]",
                        "priority":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].priority]",
                        "protocol":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].protocol]",
                        "sourcePortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourcePortRange]",
                        "destinationPortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationPortRange]",
                        "sourceAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourceAddressPrefix]",
                        "destinationAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationAddressPrefix]",
                        "access":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].access]",
                        "direction":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].direction]"
                        }
                  }
            ],
            "outputs": {}
          }
        }
    }
  ],          
  "outputs": {}
}

```

Wir werfen einen genaueren Blick darauf, wie wir die Eigenschaftswerte in der untergeordneten Ressource `securityRules` angeben. Auf unsere gesamten Eigenschaften wird mit der Funktion `parameter()` verwiesen, und anschließend verwenden wir den Punktoperator, um auf das Array `securityRules` zu verweisen, das mit dem aktuellen Wert der Iteration indiziert wird. Abschließend nutzen wir einen weiteren Punktoperator, um auf den Namen des Objekts zu verweisen. 

## <a name="try-the-template"></a>Testen der Vorlage

Befolgen Sie die nachfolgenden Schritte, wenn Sie diese Vorlage testen möchten: 

1.  Wechseln Sie zum Azure-Portal, wählen Sie das Pluszeichen (**+**) aus, suchen Sie nach dem Ressourcentyp **Vorlagenbereitstellung**, und wählen Sie ihn aus.
2.  Navigieren Sie zur Seite **Vorlagenbereitstellung**, und wählen Sie die Schaltfläche **Erstellen** aus. Mit dieser Schaltfläche öffnen Sie das Blatt **Benutzerdefinierte Bereitstellung**.
3.  Wählen Sie die Schaltfläche **Vorlage bearbeiten**.
4.  Löschen Sie die leere Vorlage. 
5.  Kopieren Sie die Beispielvorlage, und fügen Sie sie im rechten Bereich ein.
6.  Wählen Sie die Schaltfläche **Speichern** aus.
7.  Wenn Sie in den Bereich **Benutzerdefinierte Bereitstellung** zurückgekehrt sind, wählen Sie die Schaltfläche **Parameter bearbeiten** aus.
8.  Löschen Sie auf dem Blatt **Parameter bearbeiten** die vorhandene Vorlage.
9.  Kopieren Sie die Beispielparametervorlage, und fügen Sie sie ein.
10. Wählen Sie die Schaltfläche **Speichern**, sodass Sie wieder zum Blatt **Benutzerdefinierte Bereitstellung** gelangen.
11. Wählen Sie auf dem Blatt **Benutzerdefinierte Bereitstellung** Ihr Abonnement aus, erstellen Sie entweder eine neue oder verwenden Sie eine vorhandene Ressourcengruppe, und wählen Sie einen Speicherort aus. Lesen Sie die Nutzungsbedingungen, und aktivieren Sie das Kontrollkästchen **Ich stimme zu**.
12. Wählen Sie die Schaltfläche **Kaufen** aus.

## <a name="next-steps"></a>Nächste Schritte

* Sie können diese Verfahren ausweiten, um einen [Transformer und Collector für Eigenschaftsobjekte](./collector.md) zu implementieren. Die Transformer- und Collector-Verfahren sind allgemeingültiger und können über Ihre Vorlagen verknüpft werden.
* Dieses Muster ist auch im [Vorlagenbaustein-Projekt](https://github.com/mspnp/template-building-blocks) und in den [Azure-Referenzarchitekturen](/azure/architecture/reference-architectures/) implementiert. Sie können sich unsere Vorlagen ansehen, um zu ermitteln, wie wir dieses Verfahren implementiert haben.

<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg