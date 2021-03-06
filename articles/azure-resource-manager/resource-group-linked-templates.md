---
title: "链接 Azure 部署的模板 | Microsoft 文档"
description: "介绍如何使用 Azure 资源管理器模板中的链接模板创建一个模块化的模板的解决方案。 演示如何传递参数值、指定参数文件和动态创建的 URL。"
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: 27d8c4b2-1e24-45fe-88fd-8cf98a6bb2d2
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/28/2017
ms.author: tomfitz
ms.openlocfilehash: a86d4d8705c7093e3900a9738ddbd364db8bd3b8
ms.sourcegitcommit: 29bac59f1d62f38740b60274cb4912816ee775ea
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/29/2017
---
# <a name="using-linked-templates-when-deploying-azure-resources"></a>部署 Azure 资源时使用链接的模版

若要部署解决方案，可以使用单个模板，或者使用包含多个链接模板的主模板。 对于中小型解决方案，单个模板更易于理解和维护。 可以查看单个文件中的所有资源和值。 对于高级方案，使用链接模板可将解决方案分解为目标组件，并重复使用模板。

使用链接模板时，需创建一个用于在部署期间接收参数值的主模板。 主模板包含所有链接模板，并根据需要将值传递给这些模板。

![链接模板](./media/resource-group-linked-templates/nestedTemplateDesign.png)

## <a name="link-to-a-template"></a>链接到模板

若要链接到另一个模板，请将一个**部署**资源添加到主模板。

```json
"resources": [
  {
      "apiVersion": "2017-05-10",
      "name": "linkedTemplate",
      "type": "Microsoft.Resources/deployments",
      "properties": {
          "mode": "Incremental",
          <inline-template-or-external-template>
      }
  }
]
```

为部署资源提供的属性根据是要链接到外部模板，还是要在主模板中嵌入内联模板而异。

### <a name="inline-template"></a>内联模板

若要嵌入链接模板，请使用 **template** 属性并包含该模板。

```json
"resources": [
  {
    "apiVersion": "2017-05-10",
    "name": "nestedTemplate",
    "type": "Microsoft.Resources/deployments",
    "properties": {
      "mode": "Incremental",
      "template": {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {},
        "variables": {},
        "resources": [
          {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[variables('storageName')]",
            "apiVersion": "2015-06-15",
            "location": "West US",
            "properties": {
              "accountType": "Standard_LRS"
            }
          }
        ]
      },
      "parameters": {}
    }
  }
]
```

### <a name="external-template-and-external-parameters"></a>外部模板和外部参数

若要链接到外部模板和参数文件，请使用 **templateLink** 和 **parametersLink**。 链接到某个模板时，资源管理器服务必须能够访问该模板。 不能指定本地文件，或者只能在本地网络中使用的文件。 只能提供包含 **http** 或 **https** 的 URI 值。 一种做法是将链接模板放入存储帐户，并对该项使用 URI。

```json
"resources": [
  {
     "apiVersion": "2017-05-10",
     "name": "linkedTemplate",
     "type": "Microsoft.Resources/deployments",
     "properties": {
       "mode": "incremental",
       "templateLink": {
          "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.json",
          "contentVersion":"1.0.0.0"
       },
       "parametersLink": {
          "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.parameters.json",
          "contentVersion":"1.0.0.0"
       }
     }
  }
]
```

### <a name="external-template-and-inline-parameters"></a>外部模板和内联参数

或者，可以提供内联参数。 若要将值从主模板传递给链接模板，请使用**参数**。

```json
"resources": [
  {
     "apiVersion": "2017-05-10",
     "name": "linkedTemplate",
     "type": "Microsoft.Resources/deployments",
     "properties": {
       "mode": "incremental",
       "templateLink": {
          "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.json",
          "contentVersion":"1.0.0.0"
       },
       "parameters": {
          "StorageAccountName":{"value": "[parameters('StorageAccountName')]"}
        }
     }
  }
]
```

## <a name="using-variables-to-link-templates"></a>使用变量来链接模板

前面的示例演示了用于模板链接的硬编码 URL 值。 这种方法可能适用于简单的模板，但如果使用一组大型模块化模板时，将无法正常工作。 相反，可以创建一个存储主模板的基 URL 的静态变量，并从基 URL 动态创建用于链接模板的 URL。 这种方法的好处是可以轻松地移动或派生模板，因为只需在主模板中更改静态变量。 主模板会在整个分解后的模板中传递正确的 URI。

以下示例演示如何使用基 URL 来创建两个用于链接模板的 URL（**sharedTemplateUrl** 和 **vmTemplate**）。

```json
"variables": {
    "templateBaseUrl": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/postgresql-on-ubuntu/",
    "sharedTemplateUrl": "[concat(variables('templateBaseUrl'), 'shared-resources.json')]",
    "vmTemplateUrl": "[concat(variables('templateBaseUrl'), 'database-2disk-resources.json')]"
}
```

还可以使用 [deployment()](resource-group-template-functions-deployment.md#deployment) 获取当前模板的基 URL，并使用该 URL 来获取同一位置其他模板的 URL。 如果模板位置发生变化（原因可能是改版）或者想要避免对模板文件中的 URL 进行硬编码，则此方法非常有用。

```json
"variables": {
    "sharedTemplateUrl": "[uri(deployment().properties.templateLink.uri, 'shared-resources.json')]"
}
```

## <a name="get-values-from-linked-template"></a>从链接模板中获取值

若要从链接模板中获取输出值，请使用如下所示的语法检索属性值：`"[reference('<name-of-deployment>').outputs.<property-name>.value]"`。

以下示例演示如何引用链接模板和检索输出值。 链接模板返回一条简单的消息。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [],
    "outputs": {
        "greetingMessage": {
            "value": "Hello World",
            "type" : "string"
        }
    }
}
```

父模板部署链接模板并获取返回值。 请注意，该模板按名称引用部署资源，并使用链接模板返回的属性的名称。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [
        {
            "apiVersion": "2017-05-10",
            "name": "linkedTemplate",
            "type": "Microsoft.Resources/deployments",
            "properties": {
                "mode": "incremental",
                "templateLink": {
                    "uri": "[uri(deployment().properties.templateLink.uri, 'helloworld.json')]",
                    "contentVersion": "1.0.0.0"
                }
            }
        }
    ],
    "outputs": {
        "messageFromLinkedTemplate": {
            "type": "string",
            "value": "[reference('linkedTemplate').outputs.greetingMessage.value]"
        }
    }
}
```

与其他资源类型一样，可在链接的模板和其他资源间设置依赖关系。 因此，其他资源需要从链接的模板获取输出值时，可确保在其之前部署链接的模板。 或者，链接的模板依赖于其他资源时，可确保在其之前部署其他资源。

以下示例演示一个部署公共 IP 地址并返回资源 ID 的模板：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "publicIPAddresses_name": {
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "name": "[parameters('publicIPAddresses_name')]",
            "apiVersion": "2017-06-01",
            "location": "eastus",
            "properties": {
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Dynamic",
                "idleTimeoutInMinutes": 4
            },
            "dependsOn": []
        }
    ],
    "outputs": {
        "resourceID": {
            "type": "string",
            "value": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIPAddresses_name'))]"
        }
    }
}
```

在部署负载均衡器时，若要使用前面所述模板中的公共 IP 地址，请链接到该模板，并添加与部署资源之间的依赖关系。 负载均衡器上的公共 IP 地址设置为链接模板的输出值。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "loadBalancers_name": {
            "defaultValue": "mylb",
            "type": "string"
        },
        "publicIPAddresses_name": {
            "defaultValue": "myip",
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/loadBalancers",
            "name": "[parameters('loadBalancers_name')]",
            "apiVersion": "2017-06-01",
            "location": "eastus",
            "properties": {
                "frontendIPConfigurations": [
                    {
                        "name": "LoadBalancerFrontEnd",
                        "properties": {
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIPAddress": {
                                "id": "[reference('linkedTemplate').outputs.resourceID.value]"
                            }
                        }
                    }
                ],
                "backendAddressPools": [],
                "loadBalancingRules": [],
                "probes": [],
                "inboundNatRules": [],
                "outboundNatRules": [],
                "inboundNatPools": []
            },
            "dependsOn": [
                "linkedTemplate"
            ]
        },
        {
            "apiVersion": "2017-05-10",
            "name": "linkedTemplate",
            "type": "Microsoft.Resources/deployments",
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                    "uri": "[uri(deployment().properties.templateLink.uri, 'publicip.json')]",
                    "contentVersion": "1.0.0.0"
                },
                "parameters":{
                    "publicIPAddresses_name":{"value": "[parameters('publicIPAddresses_name')]"}
                }
            }
        }
    ]
}
```

## <a name="linked-templates-in-deployment-history"></a>部署历史记录中的链接模板

资源管理器将每个链接模板作为部署历史记录中的单独部署进行处理。 因此，包含三个链接模板的父模板在部署历史记录中显示为：

![部署历史记录](./media/resource-group-linked-templates/deployment-history.png)

部署后，可以使用历史记录中这些不同的条目来检索输出值。 以下模板创建一个公共 IP 地址并输出该 IP 地址：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "publicIPAddresses_name": {
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "name": "[parameters('publicIPAddresses_name')]",
            "apiVersion": "2017-06-01",
            "location": "southcentralus",
            "properties": {
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Static",
                "idleTimeoutInMinutes": 4,
                "dnsSettings": {
                    "domainNameLabel": "[concat(parameters('publicIPAddresses_name'), uniqueString(resourceGroup().id))]"
                }
            },
            "dependsOn": []
        }
    ],
    "outputs": {
        "returnedIPAddress": {
            "type": "string",
            "value": "[reference(parameters('publicIPAddresses_name')).ipAddress]"
        }
    }
}
```

以下模板链接到前面所述的模板。 它创建三个公共 IP 地址。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
    },
    "variables": {},
    "resources": [
        {
            "apiVersion": "2017-05-10",
            "name": "[concat('linkedTemplate', copyIndex())]",
            "type": "Microsoft.Resources/deployments",
            "properties": {
              "mode": "Incremental",
              "templateLink": {
                "uri": "[uri(deployment().properties.templateLink.uri, 'static-public-ip.json')]",
                "contentVersion": "1.0.0.0"
              },
              "parameters":{
                  "publicIPAddresses_name":{"value": "[concat('myip-', copyIndex())]"}
              }
            },
            "copy": {
                "count": 3,
                "name": "ip-loop"
            }
        }
    ]
}
```

部署后，可使用以下 PowerShell 脚本检索输出值：

```powershell
$loopCount = 3
for ($i = 0; $i -lt $loopCount; $i++)
{
    $name = 'linkedTemplate' + $i;
    $deployment = Get-AzureRmResourceGroupDeployment -ResourceGroupName examplegroup -Name $name
    Write-Output "deployment $($deployment.DeploymentName) returned $($deployment.Outputs.returnedIPAddress.value)"
}
```

或使用 Azure CLI 脚本：

```azurecli
for i in 0 1 2;
do
    name="linkedTemplate$i";
    deployment=$(az group deployment show -g examplegroup -n $name);
    ip=$(echo $deployment | jq .properties.outputs.returnedIPAddress.value);
    echo "deployment $name returned $ip";
done
```

## <a name="securing-an-external-template"></a>保护外部模板

尽管链接模板必须可从外部使用，但它无需向公众正式发布。 可以将模板添加到只有存储帐户所有者可以访问的专用存储帐户。 然后，在部署期间创建共享访问签名 (SAS) 令牌，用于启用访问。 将该 SAS 令牌添加到链接模板的 URI。 即使令牌作为安全字符串传入，链接模板的 URI（包括 SAS 令牌）也将记录在部署操作中。 若要限制公开，请设置令牌的到期时间。

也可将参数文件限制为通过 SAS 令牌进行访问。

以下示例演示在链接到模板时如何传递 SAS 令牌：

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "containerSasToken": { "type": "string" }
  },
  "resources": [
    {
      "apiVersion": "2017-05-10",
      "name": "linkedTemplate",
      "type": "Microsoft.Resources/deployments",
      "properties": {
        "mode": "incremental",
        "templateLink": {
          "uri": "[concat(uri(deployment().properties.templateLink.uri, 'helloworld.json'), parameters('containerSasToken'))]",
          "contentVersion": "1.0.0.0"
        }
      }
    }
  ],
  "outputs": {
  }
}
```

在 PowerShell 中，使用以下命令获取容器的令牌并部署模板：

```powershell
Set-AzureRmCurrentStorageAccount -ResourceGroupName ManageGroup -Name storagecontosotemplates
$token = New-AzureStorageContainerSASToken -Name templates -Permission r -ExpiryTime (Get-Date).AddMinutes(30.0)
$url = (Get-AzureStorageBlob -Container templates -Blob parent.json).ICloudBlob.uri.AbsoluteUri
New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateUri ($url + $token) -containerSasToken $token
```

在 Azure CLI 中，使用以下代码获取容器的令牌并部署模板：

```azurecli
expiretime=$(date -u -d '30 minutes' +%Y-%m-%dT%H:%MZ)
connection=$(az storage account show-connection-string \
    --resource-group ManageGroup \
    --name storagecontosotemplates \
    --query connectionString)
token=$(az storage container generate-sas \
    --name templates \
    --expiry $expiretime \
    --permissions r \
    --output tsv \
    --connection-string $connection)
url=$(az storage blob url \
    --container-name templates \
    --name parent.json \
    --output tsv \
    --connection-string $connection)
parameter='{"containerSasToken":{"value":"?'$token'"}}'
az group deployment create --resource-group ExampleGroup --template-uri $url?$token --parameters $parameter
```

## <a name="example-templates"></a>示例模板

### <a name="hello-world-from-linked-template"></a>来自链接模板的 Hello World

若要部署[父模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/helloworldparent.json)和[链接模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/helloworld.json)，请使用 PowerShell：

```powershell
New-AzureRmResourceGroupDeployment `
  -ResourceGroupName examplegroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/helloworldparent.json
```

也可以使用 Azure CLI：

```azurecli-interactive
az group deployment create \
  -g examplegroup \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/helloworldparent.json
```

### <a name="load-balancer-with-public-ip-address-in-linked-template"></a>链接模板中使用公共 IP 地址的负载均衡器

若要部署[父模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip-parentloadbalancer.json)和[链接模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip.json)，请使用 PowerShell：

```powershell
New-AzureRmResourceGroupDeployment `
  -ResourceGroupName examplegroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/linkedtemplates/public-ip-parentloadbalancer.json
```

也可以使用 Azure CLI：

```azurecli-interactive
az group deployment create \
  -g examplegroup \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/linkedtemplates/public-ip-parentloadbalancer.json
```

### <a name="multiple-public-ip-addresses-in-linked-template"></a>链接模板中的多个公共 IP 地址

若要部署[父模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/static-public-ip-parent.json)和[链接模板](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/static-public-ip.json)，请使用 PowerShell：

```powershell
New-AzureRmResourceGroupDeployment `
  -ResourceGroupName examplegroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/linkedtemplates/static-public-ip-parent.json
```

也可以使用 Azure CLI：

```azurecli-interactive
az group deployment create \
  -g examplegroup \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/linkedtemplates/static-public-ip-parent.json
```

## <a name="next-steps"></a>后续步骤

* 若要了解如何为资源定义部署顺序，请参阅[在 Azure 资源管理器模板中定义依赖关系](resource-group-define-dependencies.md)。
* 若要了解如何定义一个资源而创建多个实例，请参阅[在 Azure 资源管理器中创建多个资源实例](resource-group-create-multiple.md)。
* 有关在存储帐户中设置模板和生成 SAS 令牌的步骤，请参阅[使用 Resource Manager 模板和 Azure PowerShell 部署资源](resource-group-template-deploy.md)或[使用 Resource Manager 模板和 Azure CLI 部署资源](resource-group-template-deploy-cli.md)。