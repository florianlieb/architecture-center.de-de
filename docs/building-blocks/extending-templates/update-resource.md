---
title: Aktualisieren einer Ressource in einer Azure Resource Manager-Vorlage
description: Beschreibt das Erweitern der Funktionalität der Azure Resource Manager-Vorlagen für das Aktualisieren einer Ressource.
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: fc2565819c66ee7695224ef5793e7276e6e552e0
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/14/2017
---
# <a name="update-a-resource-in-an-azure-resource-manager-template"></a>Aktualisieren einer Ressource in einer Azure Resource Manager-Vorlage

Es gibt einige Szenarien, in denen Sie eine Ressource während der Bereitstellung aktualisieren müssen. Dieses Szenario kann eintreten, wenn Sie nicht alle Eigenschaften für eine Ressource angeben können, solange nicht andere, abhängige Ressourcen erstellt werden. Wenn Sie z.B einen Back-End-Adresspool für einen Load Balancer erstellen, könnten Sie vielleicht die Netzwerkschnittstellen (NICs) auf Ihren virtuellen Computern (VMs) aktualisieren, um sie in den Back-End-Pool einzubeziehen. Obwohl Resource Manager das Aktualisieren von Ressourcen während der Bereitstellung unterstützt, müssen Sie die Vorlage ordnungsgemäß entwerfen, um Fehler zu vermeiden und sicherzustellen, dass die Bereitstellung als Update behandelt wird.

Zunächst müssen Sie einmal in der Vorlage auf die Ressource verweisen, um sie zu erstellen, und dann müssen Sie mit dem gleichen Namen auf die Ressource verweisen, um sie zu einem späteren Zeitpunkt zu aktualisieren. Wenn jedoch zwei Ressourcen in einer Vorlage den gleichen Namen besitzen, löst der Resource Manager eine Ausnahme aus. Um diesen Fehler zu vermeiden, geben Sie die aktualisierte Ressource mit dem Ressourcentyp `Microsoft.Resources/deployments` in einer zweiten Vorlage an, die entweder verknüpft oder als Untervorlage enthalten ist.

Als Zweites müssen Sie in der geschachtelten Vorlage den Namen der vorhandenen zu ändernden Eigenschaft oder einen neuen Namen für eine hinzuzufügende Eigenschaft angeben. Sie müssen auch die ursprünglichen Eigenschaften und ihre ursprünglichen Werte angeben. Wenn Sie die ursprünglichen Eigenschaften und Werte nicht angeben, setzt Resource Manager voraus, dass Sie eine neue Ressource erstellen möchten, und löscht die ursprüngliche Ressource.

## <a name="example-template"></a>Beispielvorlage

Sehen wir uns eine Beispielvorlage an, die dies veranschaulicht. Unsere Vorlage stellt das virtuelle Netzwerk `firstVNet` bereit, das ein Subnetz mit dem Namen `firstSubnet` enthält. Anschließend stellt sie die virtuelle Netzwerkschnittstelle (NIC) `nic1` bereit und ordnet sie unserem Subnetz zu. Damit enthält die Bereitstellungsressource `updateVNet` eine geschachtelte Vorlage, die unsere `firstVNet`-Ressource durch Hinzufügen eines zweiten Subnetzes namens `secondSubnet` aktualisiert. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": [
      {
      "apiVersion": "2016-03-30",
      "name": "firstVNet",
      "location":"[resourceGroup().location]",
      "type": "Microsoft.Network/virtualNetworks",
      "properties": {
          "addressSpace":{"addressPrefixes": [
              "10.0.0.0/22"
          ]},
          "subnets":[              
              {
                  "name":"firstSubnet",
                  "properties":{
                    "addressPrefix":"10.0.0.0/24"
                  }
              }
            ]
      }
    },
    {
        "apiVersion": "2015-06-15",
        "type":"Microsoft.Network/networkInterfaces",
        "name":"nic1",
        "location":"[resourceGroup().location]",
        "dependsOn": [
            "firstVNet"
        ],
        "properties": {
            "ipConfigurations":[
                {
                    "name":"ipconfig1",
                    "properties": {
                        "privateIPAllocationMethod":"Dynamic",
                        "subnet": {
                            "id": "[concat(resourceId('Microsoft.Network/virtualNetworks','firstVNet'),'/subnets/firstSubnet')]"
                        }
                    }
                }
            ]
        }
    },
    {
      "apiVersion": "2015-01-01",
      "type": "Microsoft.Resources/deployments",
      "name": "updateVNet",
      "dependsOn": [
          "nic1"
      ],
      "properties": {
        "mode": "Incremental",
        "parameters": {},
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "variables": {},
          "resources": [
              {
                  "apiVersion": "2016-03-30",
                  "name": "firstVNet",
                  "location":"[resourceGroup().location]",
                  "type": "Microsoft.Network/virtualNetworks",
                  "properties": {
                      "addressSpace": "[reference('firstVNet').addressSpace]",
                      "subnets":[
                          {
                              "name":"[reference('firstVNet').subnets[0].name]",
                              "properties":{
                                  "addressPrefix":"[reference('firstVNet').subnets[0].properties.addressPrefix]"
                                  }
                          },
                          {
                              "name":"secondSubnet",
                              "properties":{
                                  "addressPrefix":"10.0.1.0/24"
                                  }
                          }
                     ]
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

Werfen wir zunächst einen Blick auf das Ressourcenobjekt für unsere `firstVNet`-Ressource. Beachten Sie, dass wir neue Einstellungen für `firstVNet` in einer geschachtelten Vorlage angeben, da Resource Manager keine gleichnamige Bereitstellung innerhalb einer Vorlage erlaubt und geschachtelte Vorlagen als eine andere Vorlage gelten. Durch die neuen Angaben für die Werte unserer `firstSubnet`-Ressource weisen wir Resource Manager an, die vorhandene Ressource zu aktualisieren, anstatt sie zu löschen und erneut bereitzustellen. Abschließend werden unsere neuen Einstellungen für `secondSubnet` während des Updates übernommen.

## <a name="try-the-template"></a>Testen der Vorlage

Befolgen Sie die nachfolgenden Schritte, wenn Sie diese Vorlage testen möchten:

1.  Wechseln Sie zum Azure-Portal, wählen Sie das Pluszeichen (**+**) aus, suchen Sie nach dem Ressourcentyp **Vorlagenbereitstellung**, und wählen Sie ihn aus.
2.  Navigieren Sie zur Seite **Vorlagenbereitstellung**, und wählen Sie die Schaltfläche **Erstellen** aus. Mit dieser Schaltfläche öffnen Sie das Blatt **Benutzerdefinierte Bereitstellung**.
3.  Wählen Sie das Symbol **Bearbeiten** aus.
4.  Löschen Sie die leere Vorlage.
5.  Kopieren Sie die Beispielvorlage, und fügen Sie sie im rechten Bereich ein.
6.  Wählen Sie die Schaltfläche **Speichern** aus.
7.  Sie kehren zum Bereich **Benutzerdefinierte Bereitstellung** zurück, aber dieses Mal sind einige Dropdownfelder verfügbar. Wählen Sie Ihr Abonnement aus, erstellen Sie eine neue oder vorhandene Ressourcengruppe, und wählen Sie dann einen Speicherort aus. Lesen Sie die Nutzungsbedingungen, und wählen Sie die Schaltfläche **Ich stimme zu**.
8.  Wählen Sie die Schaltfläche **Kaufen** aus.

Nachdem die Bereitstellung abgeschlossen ist, öffnen Sie die Ressourcengruppe, die Sie im Verwaltungsportal angegeben haben. Sie sehen ein virtuelles Netzwerk mit dem Namen `firstVNet` und eine NIC mit dem Namen `nic1`. Klicken Sie auf `firstVNet` und dann auf `subnets`. Sie sehen das ursprünglich erstellte `firstSubnet` und das `secondSubnet`, das in der `updateVNet`-Ressource hinzugefügt wurde. 

![Ursprüngliches Subnetz und aktualisiertes Subnetz](../_images/firstVNet-subnets.png)

Gehen Sie dann zurück zur Ressourcengruppe, und klicken Sie auf `nic1` und dann auf `IP configurations`. Im `IP configurations`-Abschnitt ist `subnet` auf `firstSubnet (10.0.0.0/24)` festgelegt. 

![IP-Konfigurationseinstellungen für nic1](../_images/nic1-ipconfigurations.png)

Das ursprüngliche `firstVNet` wurde nicht neu erstellt, sondern aktualisiert. Wäre `firstVNet` neu erstellt worden, wäre `nic1` nicht `firstVNet` zugeordnet.

## <a name="next-steps"></a>Nächste Schritte

* Dieses Verfahren ist im [Vorlagenbaustein-Projekt](https://github.com/mspnp/template-building-blocks) und in den [Azure-Referenzarchitekturen](/azure/architecture/reference-architectures/) implementiert. Sie können es verwenden, um Ihre eigene Architektur zu erstellen oder eine unserer Referenzarchitekturen bereitzustellen.
