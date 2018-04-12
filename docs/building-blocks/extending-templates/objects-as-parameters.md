---
title: Verwenden eines Objekts als Parameter in einer Azure Resource Manager-Vorlage
description: Beschreibt das Erweitern der Funktionalität der Azure Resource Manager-Vorlagen für die Verwendung von Objekten als Parameter.
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 76f8b9d459f4ab3147b52762b7c26552ec92c7a3
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a><span data-ttu-id="fa79c-103">Verwenden eines Objekts als Parameter in einer Azure Resource Manager-Vorlage</span><span class="sxs-lookup"><span data-stu-id="fa79c-103">Use an object as a parameter in an Azure Resource Manager template</span></span>

<span data-ttu-id="fa79c-104">Beim [Erstellen von Azure Resource Manager-Vorlagen][azure-resource-manager-create-template] können Sie entweder Ressourceneigenschaftswerte direkt in der Vorlage angeben oder einen Parameter definieren und die Werte während der Bereitstellung angeben.</span><span class="sxs-lookup"><span data-stu-id="fa79c-104">When you [author Azure Resource Manager templates][azure-resource-manager-create-template], you can either specify resource property values directly in the template or define a parameter and provide values during deployment.</span></span> <span data-ttu-id="fa79c-105">Es kann ein Parameter für jeden Eigenschaftswert verwendet werden, aber es gilt ein Grenzwert von 255 Parametern pro Bereitstellung.</span><span class="sxs-lookup"><span data-stu-id="fa79c-105">It's fine to use a parameter for each property value for small deployments, but there is a limit of 255 parameters per deployment.</span></span> <span data-ttu-id="fa79c-106">Wenn Sie größere und komplexere Bereitstellungen verwenden, gehen Ihnen ggf. die Parameter aus.</span><span class="sxs-lookup"><span data-stu-id="fa79c-106">Once you get to larger and more complex deployments you may run out of parameters.</span></span>

<span data-ttu-id="fa79c-107">Eine Möglichkeit, dieses Problem zu beheben, ist die Verwendung eines Objekts als Parameter anstatt als Wert.</span><span class="sxs-lookup"><span data-stu-id="fa79c-107">One way to solve this problem is to use an object as a parameter instead of a value.</span></span> <span data-ttu-id="fa79c-108">Definieren Sie hierfür den Parameter in Ihrer Vorlage, und geben Sie ein JSON-Objekt an, anstatt einen einzelnen Wert während der Bereitstellung.</span><span class="sxs-lookup"><span data-stu-id="fa79c-108">To do this, define the parameter in your template and specify a JSON object instead of a single value during deployment.</span></span> <span data-ttu-id="fa79c-109">Verweisen Sie anschließend auf die Untereigenschaften des Parameters, indem Sie die [`parameter()`-Funktion][azure-resource-manager-functions] und den Punktoperator in Ihrer Vorlage verwenden.</span><span class="sxs-lookup"><span data-stu-id="fa79c-109">Then, reference the subproperties of the parameter using the [`parameter()` function][azure-resource-manager-functions] and dot operator in your template.</span></span>

<span data-ttu-id="fa79c-110">Wir sehen uns ein Beispiel an, in dem eine virtuelle Netzwerkressource bereitgestellt wird.</span><span class="sxs-lookup"><span data-stu-id="fa79c-110">Let's take a look at an example that deploys a virtual network resource.</span></span> <span data-ttu-id="fa79c-111">Zuerst geben wir in der Vorlage den Parameter `VNetSettings` an und legen `type` auf `object` fest:</span><span class="sxs-lookup"><span data-stu-id="fa79c-111">First, let's specify a `VNetSettings` parameter in our template and set the `type` to `object`:</span></span>

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
<span data-ttu-id="fa79c-112">Als Nächstes geben wir Werte für das Objekt `VNetSettings` an:</span><span class="sxs-lookup"><span data-stu-id="fa79c-112">Next, let's provide values for the `VNetSettings` object:</span></span>

> [!NOTE]
> <span data-ttu-id="fa79c-113">Informationen zum Angeben von Parameterwerten während der Bereitstellung finden Sie im Abschnitt **parameters** unter [Verstehen der Struktur und Syntax von Azure Resource Manager-Vorlagen][azure-resource-manager-authoring-templates].</span><span class="sxs-lookup"><span data-stu-id="fa79c-113">To learn how to provide parameter values during deploment, see the **parameters** section of [understand the structure and syntax of Azure Resource Manager templates][azure-resource-manager-authoring-templates].</span></span> 

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

<span data-ttu-id="fa79c-114">Sie sehen, dass der einzelne Parameter drei Untereigenschaften angibt: `name`, `addressPrefixes` und `subnets`.</span><span class="sxs-lookup"><span data-stu-id="fa79c-114">As you can see, our single parameter actually specifies three subproperties: `name`, `addressPrefixes`, and `subnets`.</span></span> <span data-ttu-id="fa79c-115">Mit jeder dieser Untereigenschaften werden entweder ein Wert oder andere Untereigenschaften angegeben.</span><span class="sxs-lookup"><span data-stu-id="fa79c-115">Each of these subproperties either specifies a value or other subproperties.</span></span> <span data-ttu-id="fa79c-116">Das Ergebnis ist, dass mit dem einzelnen Parameter alle Werte angegeben werden, die zum Bereitstellen des virtuellen Netzwerks benötigt werden.</span><span class="sxs-lookup"><span data-stu-id="fa79c-116">The result is that our single parameter specifies all the values necessary to deploy our virtual network.</span></span>

<span data-ttu-id="fa79c-117">Nun sehen wir uns den Rest der Vorlage an, um zu ermitteln, wie das Objekt `VNetSettings` verwendet wird:</span><span class="sxs-lookup"><span data-stu-id="fa79c-117">Now let's have a look at the rest of our template to see how the `VNetSettings` object is used:</span></span>

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
<span data-ttu-id="fa79c-118">Die Werte des Objekts `VNetSettings` werden auf die Eigenschaften angewendet, die für unsere virtuelle Netzwerkressource erforderlich sind, indem die Funktion `parameters()` mit dem `[]`-Arrayindexer und dem Punktoperator verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="fa79c-118">The values of our `VNetSettings` object are applied to the properties required by our virtual network resource using the `parameters()` function with both the `[]` array indexer and the dot operator.</span></span> <span data-ttu-id="fa79c-119">Dieser Ansatz funktioniert, wenn Sie nur die Werte des Parameterobjekts statisch auf die Ressource anwenden möchten.</span><span class="sxs-lookup"><span data-stu-id="fa79c-119">This approach works if you just want to statically apply the values of the parameter object to the resource.</span></span> <span data-ttu-id="fa79c-120">Falls Sie aber ein Array mit Eigenschaftswerten während der Bereitstellung dynamisch zuweisen möchten, können Sie eine [Kopierschleife][azure-resource-manager-create-multiple-instances] verwenden.</span><span class="sxs-lookup"><span data-stu-id="fa79c-120">However, if you want to dynamically assign an array of property values during deployment you can use a [copy loop][azure-resource-manager-create-multiple-instances].</span></span> <span data-ttu-id="fa79c-121">Für die Nutzung einer Kopierschleife geben Sie ein JSON-Array mit Ressourceneigenschaftswerten an, und die Kopierschleife wendet die Werte dynamisch auf die Eigenschaften der Ressource an.</span><span class="sxs-lookup"><span data-stu-id="fa79c-121">To use a copy loop, you provide a JSON array of resource property values and the copy loop dynamically applies the values to the resource's properties.</span></span> 

<span data-ttu-id="fa79c-122">Bei Verwendung des dynamischen Ansatzes sollte ein mögliches Problem beachtet werden.</span><span class="sxs-lookup"><span data-stu-id="fa79c-122">There is one issue to be aware of if you use the dynamic approach.</span></span> <span data-ttu-id="fa79c-123">Um das Problem zu demonstrieren, sehen wir uns ein typisches Array mit Eigenschaftswerten an.</span><span class="sxs-lookup"><span data-stu-id="fa79c-123">To demonstrate the issue, let's take a look at a typical array of property values.</span></span> <span data-ttu-id="fa79c-124">In diesem Beispiel werden die Werte für unsere Eigenschaften in einer Variablen gespeichert.</span><span class="sxs-lookup"><span data-stu-id="fa79c-124">In this example the values for our properties are stored in a variable.</span></span> <span data-ttu-id="fa79c-125">Beachten Sie, dass wir hier zwei Arrays verwenden: ein Array mit dem Namen `firstProperty` und ein Array mit dem Namen `secondProperty`.</span><span class="sxs-lookup"><span data-stu-id="fa79c-125">Notice we have two arrays here&mdash;one named `firstProperty` and one named `secondProperty`.</span></span> 

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

<span data-ttu-id="fa79c-126">Als Nächstes sehen wir uns an, wie wir auf die Eigenschaften in der Variablen mit einer Kopierschleife zugreifen.</span><span class="sxs-lookup"><span data-stu-id="fa79c-126">Now let's take a look at the way we access the properties in the variable using a copy loop.</span></span>

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

<span data-ttu-id="fa79c-127">Die Funktion `copyIndex()` gibt die aktuelle Iteration der Kopierschleife zurück, und wir verwenden diese Rückgabe gleichzeitig als Index für beide Arrays.</span><span class="sxs-lookup"><span data-stu-id="fa79c-127">The `copyIndex()` function returns the current iteration of the copy loop, and we use that as an index into each of the two arrays simultaneously.</span></span>

<span data-ttu-id="fa79c-128">Dies funktioniert gut, wenn die beiden Arrays die gleiche Länge haben.</span><span class="sxs-lookup"><span data-stu-id="fa79c-128">This works fine when the two arrays are the same length.</span></span> <span data-ttu-id="fa79c-129">Das Problem tritt auf, wenn Sie einen Fehler gemacht haben und die beiden Arrays eine unterschiedliche Länge aufweisen. In diesem Fall besteht Ihre Vorlage die Überprüfung während der Bereitstellung nicht.</span><span class="sxs-lookup"><span data-stu-id="fa79c-129">The issue arises if you've made a mistake and the two arrays are different lengths&mdash;in this case your template will fail validation during deployment.</span></span> <span data-ttu-id="fa79c-130">Sie können dieses Problem vermeiden, indem Sie Ihre gesamten Eigenschaften in ein einzelnes Objekt einbinden, weil Sie dann viel einfacher erkennen können, wenn ein Wert fehlt.</span><span class="sxs-lookup"><span data-stu-id="fa79c-130">You can avoid this issue by including all your properties in a single object, because it is much easier to see when a value is missing.</span></span> <span data-ttu-id="fa79c-131">Unten ist ein weiteres Parameterobjekt angegeben, bei dem jedes Element des Arrays `propertyObject` eine Vereinigung der vorherigen Arrays `firstProperty` und `secondProperty` ist.</span><span class="sxs-lookup"><span data-stu-id="fa79c-131">For example, let's take a look another parameter object in which each element of the `propertyObject` array is the union of the `firstProperty` and `secondProperty` arrays from earlier.</span></span>

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

<span data-ttu-id="fa79c-132">Fällt Ihnen das dritte Element im Array auf?</span><span class="sxs-lookup"><span data-stu-id="fa79c-132">Notice the third element in the array?</span></span> <span data-ttu-id="fa79c-133">Dafür fehlt die `number`-Eigenschaft, aber das Fehlen ist viel leichter zu erkennen, wenn Sie die Parameterwerte auf diese Weise erstellen.</span><span class="sxs-lookup"><span data-stu-id="fa79c-133">It's missing the `number` property, but it's much easier to notice that you've missed it when you're authoring the parameter values this way.</span></span>

## <a name="using-a-property-object-in-a-copy-loop"></a><span data-ttu-id="fa79c-134">Verwenden eines Eigenschaftsobjekts in einer Kopierschleife</span><span class="sxs-lookup"><span data-stu-id="fa79c-134">Using a property object in a copy loop</span></span>

<span data-ttu-id="fa79c-135">Dieser Ansatz ist noch nützlicher, wenn er mit der [seriellen Kopierschleife][azure-resource-manager-create-multiple] kombiniert wird, besonders beim Bereitstellen von untergeordneten Ressourcen.</span><span class="sxs-lookup"><span data-stu-id="fa79c-135">This approach becomes even more useful when combined with the [serial copy loop][azure-resource-manager-create-multiple], particularly for deploying child resources.</span></span> 

<span data-ttu-id="fa79c-136">Zur Verdeutlichung sehen wir uns eine Vorlage an, mit der eine [Netzwerksicherheitsgruppe (NSG)][nsg] mit zwei Sicherheitsregeln bereitgestellt wird.</span><span class="sxs-lookup"><span data-stu-id="fa79c-136">To demonstrate this, let's look at a template that deploys a [network security group (NSG)][nsg] with two security rules.</span></span> 

<span data-ttu-id="fa79c-137">Zuerst sehen wir uns die Parameter an.</span><span class="sxs-lookup"><span data-stu-id="fa79c-137">First, let's take a look at our parameters.</span></span> <span data-ttu-id="fa79c-138">In der Vorlage ist zu erkennen, dass wir einen Parameter mit dem Namen `networkSecurityGroupsSettings` definiert haben, der ein Array mit dem Namen `securityRules` enthält.</span><span class="sxs-lookup"><span data-stu-id="fa79c-138">When we look at our template we'll see that we've defined one parameter named `networkSecurityGroupsSettings` that includes an array named `securityRules`.</span></span> <span data-ttu-id="fa79c-139">Dieses Array enthält zwei JSON-Objekte, die einige Einstellungen für eine Sicherheitsregel angeben.</span><span class="sxs-lookup"><span data-stu-id="fa79c-139">This array contains two JSON objects that specify a number of settings for a security rule.</span></span>

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

<span data-ttu-id="fa79c-140">Als Nächstes sehen wir uns unsere Vorlage an.</span><span class="sxs-lookup"><span data-stu-id="fa79c-140">Now let's take a look at our template.</span></span> <span data-ttu-id="fa79c-141">Mit der ersten Ressource mit dem Namen `NSG1` wird die NSG bereitgestellt.</span><span class="sxs-lookup"><span data-stu-id="fa79c-141">Our first resource named `NSG1` deploys the NSG.</span></span> <span data-ttu-id="fa79c-142">Die zweite Ressource mit dem Namen `loop-0` erfüllt zwei Funktionen: Erstens ist sie von der NSG abhängig (`dependsOn`), sodass ihre Bereitstellung erst beginnt, wenn `NSG1` abgeschlossen ist, und zweitens ist sie die erste Iteration der sequenziellen Schleife.</span><span class="sxs-lookup"><span data-stu-id="fa79c-142">Our second resource named `loop-0` performs two functions: first, it `dependsOn` the NSG so its deployment doesn't begin until `NSG1` is completed, and it is the first iteration of the sequential loop.</span></span> <span data-ttu-id="fa79c-143">Die dritte Ressource ist eine geschachtelte Vorlage, die unsere Sicherheitsregeln wie im letzten Beispiel unter Verwendung eines Objekts für die Parameterwerte bereitstellt.</span><span class="sxs-lookup"><span data-stu-id="fa79c-143">Our third resource is a nested template that deploys our security rules using an object for its parameter values as in the last example.</span></span>

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

<span data-ttu-id="fa79c-144">Wir werfen einen genaueren Blick darauf, wie wir die Eigenschaftswerte in der untergeordneten Ressource `securityRules` angeben.</span><span class="sxs-lookup"><span data-stu-id="fa79c-144">Let's take a closer look at how we specify our property values in the `securityRules` child resource.</span></span> <span data-ttu-id="fa79c-145">Auf unsere gesamten Eigenschaften wird mit der Funktion `parameter()` verwiesen, und anschließend verwenden wir den Punktoperator, um auf das Array `securityRules` zu verweisen, das mit dem aktuellen Wert der Iteration indiziert wird.</span><span class="sxs-lookup"><span data-stu-id="fa79c-145">All of our properties are referenced using the `parameter()` function, and then we use the dot operator to reference our `securityRules` array, indexed by the current value of the iteration.</span></span> <span data-ttu-id="fa79c-146">Abschließend nutzen wir einen weiteren Punktoperator, um auf den Namen des Objekts zu verweisen.</span><span class="sxs-lookup"><span data-stu-id="fa79c-146">Finally, we use another dot operator to reference the name of the object.</span></span> 

## <a name="try-the-template"></a><span data-ttu-id="fa79c-147">Testen der Vorlage</span><span class="sxs-lookup"><span data-stu-id="fa79c-147">Try the template</span></span>

<span data-ttu-id="fa79c-148">Befolgen Sie die nachfolgenden Schritte, wenn Sie diese Vorlage testen möchten:</span><span class="sxs-lookup"><span data-stu-id="fa79c-148">If you would like to experiment with this template, follow these steps:</span></span> 

1.  <span data-ttu-id="fa79c-149">Wechseln Sie zum Azure-Portal, wählen Sie das Pluszeichen (**+**) aus, suchen Sie nach dem Ressourcentyp **Vorlagenbereitstellung**, und wählen Sie ihn aus.</span><span class="sxs-lookup"><span data-stu-id="fa79c-149">Go to the Azure portal, select the **+** icon, and search for the **template deployment** resource type, and select it.</span></span>
2.  <span data-ttu-id="fa79c-150">Navigieren Sie zur Seite **Vorlagenbereitstellung**, und wählen Sie die Schaltfläche **Erstellen** aus.</span><span class="sxs-lookup"><span data-stu-id="fa79c-150">Navigate to the **template deployment** page, select the **create** button.</span></span> <span data-ttu-id="fa79c-151">Mit dieser Schaltfläche öffnen Sie das Blatt **Benutzerdefinierte Bereitstellung**.</span><span class="sxs-lookup"><span data-stu-id="fa79c-151">This button opens the **custom deployment** blade.</span></span>
3.  <span data-ttu-id="fa79c-152">Wählen Sie die Schaltfläche **Vorlage bearbeiten**.</span><span class="sxs-lookup"><span data-stu-id="fa79c-152">Select the **edit template** button.</span></span>
4.  <span data-ttu-id="fa79c-153">Löschen Sie die leere Vorlage.</span><span class="sxs-lookup"><span data-stu-id="fa79c-153">Delete the empty template.</span></span> 
5.  <span data-ttu-id="fa79c-154">Kopieren Sie die Beispielvorlage, und fügen Sie sie im rechten Bereich ein.</span><span class="sxs-lookup"><span data-stu-id="fa79c-154">Copy and paste the sample template into the right pane.</span></span>
6.  <span data-ttu-id="fa79c-155">Wählen Sie die Schaltfläche **Speichern** aus.</span><span class="sxs-lookup"><span data-stu-id="fa79c-155">Select the **save** button.</span></span>
7.  <span data-ttu-id="fa79c-156">Wenn Sie in den Bereich **Benutzerdefinierte Bereitstellung** zurückgekehrt sind, wählen Sie die Schaltfläche **Parameter bearbeiten** aus.</span><span class="sxs-lookup"><span data-stu-id="fa79c-156">When you are returned to the **custom deployment** pane, select the **edit parameters** button.</span></span>
8.  <span data-ttu-id="fa79c-157">Löschen Sie auf dem Blatt **Parameter bearbeiten** die vorhandene Vorlage.</span><span class="sxs-lookup"><span data-stu-id="fa79c-157">On the **edit parameters** blade, delete the existing template.</span></span>
9.  <span data-ttu-id="fa79c-158">Kopieren Sie die Beispielparametervorlage, und fügen Sie sie ein.</span><span class="sxs-lookup"><span data-stu-id="fa79c-158">Copy and paste the sample parameter template from above.</span></span>
10. <span data-ttu-id="fa79c-159">Wählen Sie die Schaltfläche **Speichern**, sodass Sie wieder zum Blatt **Benutzerdefinierte Bereitstellung** gelangen.</span><span class="sxs-lookup"><span data-stu-id="fa79c-159">Select the **save** button, which returns you to the **custom deployment** blade.</span></span>
11. <span data-ttu-id="fa79c-160">Wählen Sie auf dem Blatt **Benutzerdefinierte Bereitstellung** Ihr Abonnement aus, erstellen Sie entweder eine neue oder verwenden Sie eine vorhandene Ressourcengruppe, und wählen Sie einen Speicherort aus.</span><span class="sxs-lookup"><span data-stu-id="fa79c-160">On the **custom deployment** blade, select your subscription, either create new or use existing resource group, and select a location.</span></span> <span data-ttu-id="fa79c-161">Lesen Sie die Nutzungsbedingungen, und aktivieren Sie das Kontrollkästchen **Ich stimme zu**.</span><span class="sxs-lookup"><span data-stu-id="fa79c-161">Review the terms and conditions, and select the **I agree** checkbox.</span></span>
12. <span data-ttu-id="fa79c-162">Wählen Sie die Schaltfläche **Kaufen** aus.</span><span class="sxs-lookup"><span data-stu-id="fa79c-162">Select the **purchase** button.</span></span>

## <a name="next-steps"></a><span data-ttu-id="fa79c-163">Nächste Schritte</span><span class="sxs-lookup"><span data-stu-id="fa79c-163">Next steps</span></span>

* <span data-ttu-id="fa79c-164">Sie können diese Verfahren ausweiten, um einen [Transformer und Collector für Eigenschaftsobjekte](./collector.md) zu implementieren.</span><span class="sxs-lookup"><span data-stu-id="fa79c-164">You can expand upon these techniques to implement a [property object transformer and collector](./collector.md).</span></span> <span data-ttu-id="fa79c-165">Die Transformer- und Collector-Verfahren sind allgemeingültiger und können über Ihre Vorlagen verknüpft werden.</span><span class="sxs-lookup"><span data-stu-id="fa79c-165">The transformer and collector techniques are more general and can be linked from your templates.</span></span>
* <span data-ttu-id="fa79c-166">Dieses Muster ist auch im [Vorlagenbaustein-Projekt](https://github.com/mspnp/template-building-blocks) und in den [Azure-Referenzarchitekturen](/azure/architecture/reference-architectures/) implementiert.</span><span class="sxs-lookup"><span data-stu-id="fa79c-166">This technique is also implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="fa79c-167">Sie können sich unsere Vorlagen ansehen, um zu ermitteln, wie wir dieses Verfahren implementiert haben.</span><span class="sxs-lookup"><span data-stu-id="fa79c-167">You can review our templates to see how we've implemented this technique.</span></span>

<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg