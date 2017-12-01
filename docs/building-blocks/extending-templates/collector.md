---
title: "Implementieren eines Transformers und Collectors für Eigenschaften in eine Azure Resource Manager-Vorlage"
description: "Es wird beschrieben, wie Sie einen Transformer und Collector für Eigenschaften in eine Azure Resource Manager-Vorlage implementieren."
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 893779e652b845b3d936d11936dc767ef632fa43
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="implement-a-property-transformer-and-collector-in-an-azure-resource-manager-template"></a>Implementieren eines Transformers und Collectors für Eigenschaften in eine Azure Resource Manager-Vorlage

Unter [Verwenden eines Objekts als Parameter in einer Azure Resource Manager-Vorlage][objects-as-parameters] wurde beschrieben, wie Sie Werte von Ressourceneigenschaften in einem Objekt speichern und während der Bereitstellung auf eine Ressource anwenden. Dies ist eine sehr gute Möglichkeit, um Ihre Parameter zu verwalten. Bei jeder Verwendung in Ihrer Vorlage müssen Sie aber trotzdem weiterhin die Eigenschaften des Objekts den Ressourceneigenschaften zuordnen.

Zur Umgehung dieses Problems können Sie eine Vorlage für die Transformation und Sammlung von Eigenschaften (Transformer und Collector) implementieren, mit der Ihr Objektarray durchlaufen und in das JSON-Schema transformiert wird, das von der Ressource erwartet wird.

> [!IMPORTANT]
> Für diesen Ansatz ist es erforderlich, dass Sie eingehend mit Resource Manager-Vorlagen und -Funktionen vertraut sind.

Wir sehen uns nun an, wie wir einen Collector und Transformer für Eigenschaften implementieren können. Hierzu verwenden wir ein Beispiel, in dem eine [Netzwerksicherheitsgruppe (NSG)][nsg] bereitgestellt wird. Im Diagramm unten ist die Beziehung zwischen unseren Vorlagen und Ressourcen in diesen Vorlagen dargestellt:

![Architektur für Collectors und Transformers für Eigenschaften](../_images/collector-transformer.png)

Unsere **aufrufende Vorlage** enthält zwei Ressourcen:
* Einen Vorlagenlink, über den die **Collector-Vorlage** aufgerufen wird.
* Die bereitzustellende NSG-Ressource.

Unsere **Collector-Vorlage** enthält zwei Ressourcen:
* Eine **anchor**-Ressource.
* Einen Vorlagenlink, über den die Transformer-Vorlage in einer Kopierschleife aufgerufen wird.

Unsere **Transformer-Vorlage** enthält eine einzelne Ressource: eine leere Vorlage mit einer Variablen, die den `source`-JSON-Code in das JSON-Schema transformiert, der von der NSG-Ressource in der **Hauptvorlage** erwartet wird.

## <a name="parameter-object"></a>Parameterobjekt

Wir verwenden das Parameterobjekt `securityRules` aus [Objekte als Parameter][objects-as-parameters]. Mit der **Transformer-Vorlage** wird jedes Objekt im Array `securityRules` in das JSON-Schema transformiert, das von der NSG-Ressource in unserer **aufrufenden Vorlage** erwartet wird.

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

Wir sehen uns zuerst die **Transformer-Vorlage** an.

## <a name="transform-template"></a>Transformer-Vorlage

Die **Transformer-Vorlage** enthält zwei Parameter, die von der **Collector-Vorlage** übergeben werden: 
* `source` ist ein Objekt, das eines der Eigenschaftswertobjekte aus dem Eigenschaftenarray empfängt. In unserem Beispiel wird jedes Objekt aus dem Array `"securityRules"` einzeln übergeben.
* `state` ist ein Array, das die verketteten Ergebnisse aller vorherigen Transformationen empfängt. Dies ist die Sammlung mit dem transformierten JSON-Code.

Unsere Parameter sehen wie folgt aus:

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "source": { "type": "object" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
  },
```

In unserer Vorlage wird auch eine Variable mit dem Namen `instance` definiert. Hiermit wird die eigentliche Transformation unseres `source`-Objekts in das erforderliche JSON-Schema durchgeführt:

```json
  "variables": {
    "instance": [
      {
        "name": "[parameters('source').name]",
        "properties":{
            "description": "[parameters('source').description]",
            "protocol": "[parameters('source').protocol]",
            "sourcePortRange": "[parameters('source').sourcePortRange]",
            "destinationPortRange": "[parameters('source').destinationPortRange]",
            "sourceAddressPrefix": "[parameters('source').sourceAddressPrefix]",
            "destinationAddressPrefix": "[parameters('source').destinationAddressPrefix]",
            "access": "[parameters('source').access]",
            "priority": "[parameters('source').priority]",
            "direction": "[parameters('source').direction]"            
        }
      }
    ]

  },
```

Abschließend werden in der Ausgabe (`output`) unserer Vorlage die gesammelten Transformationen des Parameters `state` mit der aktuellen Transformation verkettet, die von der Variablen `instance` durchgeführt wird:

```json
  "outputs": {
    "collection": {
      "type": "array",
      "value": "[concat(parameters('state'), variables('instance'))]"
    }
```

Als Nächstes sehen wir uns unsere **Collector-Vorlage** an, um zu verfolgen, wie die Parameterwerte übergeben werden.

## <a name="collector-template"></a>Collector-Vorlage

Die **Collector-Vorlage** enthält drei Parameter:
* `source` ist unser vollständiges Parameterobjektarray. Es wird von der **aufrufenden Vorlage** übergeben. Dieses Array hat den gleichen Namen wie der Parameter `source` in der **Transformer-Vorlage**, aber es gibt einen wichtigen Unterschied, den Sie vielleicht schon bemerkt haben: Dies ist das vollständige Array, aber es wird jeweils nur ein Element dieses Arrays an die **Transformer-Vorlage** übergeben.
* `transformTemplateUri` ist der URI der **Transformer-Vorlage**. Wir definieren ihn hier als Parameter, um die Vorlage wiederverwenden zu können.
* `state` ist ein ursprünglich leeres Array, das wir an die **Transformer-Vorlage** übergeben. Darin wird die Sammlung mit den transformierten Parameterobjekten gespeichert, wenn die Kopierschleife abgeschlossen ist.

Unsere Parameter sehen wie folgt aus:

```json
  "parameters": {
    "source": { "type": "array" },
    "transformTemplateUri": { "type": "string" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
``` 

Als Nächstes definieren wir eine Variable mit dem Namen `count`. Ihr Wert ist die Länge des Parameterobjektarrays `source`:

```json
  "variables": {
    "count": "[length(parameters('source'))]"
  },
```

Wie Sie vielleicht schon erwartet haben, verwenden wir sie für die Anzahl von Durchläufen (Iterationen) unserer Kopierschleife.

Wir sehen uns nun die Ressourcen an. Es werden zwei Ressourcen definiert:
* `loop-0` ist die nullbasierte Ressource für unsere Kopierschleife.
* `loop-` wird mit dem Ergebnis der Funktion `copyIndex(1)` verkettet, um einen eindeutigen iterationsbasierten Namen für die Ressource zu generieren, der mit `1` beginnt.

Unsere Ressourcen sehen wie folgt aus:

```json
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "loop-0",
      "properties": {
        "mode": "Incremental",
        "parameters": { },
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": { },
          "variables": { },
          "resources": [ ],
          "outputs": {
            "collection": {
              "type": "array",
              "value": "[parameters('state')]"
            }
          }
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "[concat('loop-', copyindex(1))]",
      "copy": {
        "name": "iterator",
        "count": "[variables('count')]",
        "mode": "serial"
      },
      "dependsOn": [
        "loop-0"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": { "uri": "[parameters('transformTemplateUri')]" },
        "parameters": {
          "source": { "value": "[parameters('source')[copyindex()]]" },
          "state": { "value": "[reference(concat('loop-', copyindex())).outputs.collection.value]" }
        }
      }
    }
  ],
```

Wir werfen einen genaueren Blick auf die Parameter, die wir an die **Transformer-Vorlage** in der geschachtelten Vorlage übergeben. Sie erinnern sich, dass mit dem Parameter `source` das aktuelle Objekt im Parameterobjektarray `source` übergeben wird. Die Sammlung wird im Parameter `state` durchgeführt. Es wird die Ausgabe des vorherigen Durchlaufs durch die Kopierschleife verwendet (die Funktion `reference()` verwendet die Funktion `copyIndex()` ohne Parameter, um auf das `name`-Element des vorherigen verknüpften Vorlagenobjekts zu verweisen) und an den aktuellen Durchlauf übergeben.

Abschließend gibt die Ausgabe (`output`) unserer Vorlage die Ausgabe (`output`) des letzten Durchlaufs unserer **Transformer-Vorlage** zurück:

```json
  "outputs": {
    "result": {
      "type": "array",
      "value": "[reference(concat('loop-', variables('count'))).outputs.collection.value]"
    }
  }
```
Es kann ungewöhnlich erscheinen, dass die Ausgabe (`output`) des letzten Durchlaufs der **Transformer-Vorlage** an die **aufrufende Vorlage** übergeben wird, da sie anscheinend im Parameter `source` gespeichert wurde. Bedenken Sie aber, dass dies der letzte Durchlauf der **Transformer-Vorlage** ist, die das gesamte Array mit den Eigenschaftsobjekten enthält, und dass dies die Daten sind, die wir zurückgeben möchten.

Zum Schluss sehen wir uns noch an, wie die **Collector-Vorlage** aus der **aufrufenden Vorlage** aufgerufen wird.

## <a name="calling-template"></a>Aufrufende Vorlage

Mit der **aufrufenden Vorlage** wird ein einzelner Parameter mit dem Namen `networkSecurityGroupsSettings` definiert:

```json
...
"parameters": {
    "networkSecurityGroupsSettings": {
        "type": "object"
    }
```

Als Nächstes definiert unsere Vorlage eine einzelne Variable mit dem Namen `collectorTemplateUri`:

```json
"variables": {
    "collectorTemplateUri": "[uri(deployment().properties.templateLink.uri, 'collector.template.json')]"
  }
```

Wie zu erwarten ist, ist dies der URI für die **Collector-Vorlage**, die von der verknüpften Vorlagenressource verwendet wird:

```json
{
    "apiVersion": "2015-01-01",
    "name": "collector",
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('linkedTemplateUri')]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "source" : {"value": "[parameters('networkSecurityGroupsSettings').securityRules]"},
            "transformTemplateUri": { "value": "[uri(deployment().properties.templateLink.uri, 'transform.json')]"}
        }
    }
}
```

Wir übergeben zwei Parameter an die **Collector-Vorlage**:
* `source` ist unser Eigenschaftsobjektarray. In unserem Beispiel ist dies der Parameter `networkSecurityGroupsSettings`.
* `transformTemplateUri` ist die Variable, die wir gerade mit dem URI der **Collector-Vorlage** definiert haben.

Zuletzt weist die Ressource `Microsoft.Network/networkSecurityGroups` die Ausgabe (`output`) der verknüpften Vorlagenressource `collector` direkt der dazugehörigen `securityRules`-Eigenschaft zu:

```json
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "networkSecurityGroup1",
      "location": "[resourceGroup().location]",
      "properties": {
        "securityRules": "[reference('firstResource').outputs.result.value]"
      }
    }
  ],
  "outputs": {
      "instance":{
          "type": "array",
          "value": "[reference('firstResource').outputs.result.value]"
      }

  }
```

## <a name="next-steps"></a>Nächste Schritte

* Dieses Verfahren ist im [Vorlagenbaustein-Projekt](https://github.com/mspnp/template-building-blocks) und in den [Azure-Referenzarchitekturen](/azure/architecture/reference-architectures/) implementiert. Sie können hiermit Ihre eigene Architektur erstellen, oder Sie können eine unserer Referenzarchitekturen bereitstellen.

<!-- links -->
[objects-as-parameters]: ./objects-as-parameters.md
[resource-manager-linked-template]: /azure/azure-resource-manager/resource-group-linked-templates
[resource-manager-variables]: /azure/azure-resource-manager/resource-group-template-functions-deployment
[nsg]: /azure/virtual-network/virtual-networks-nsg
