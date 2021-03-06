如果要在部署期间使用参数文件传递参数值，需要使用类似于以下示例的格式创建一个 JSON 文件：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "webSiteName": {
            "value": "ExampleSite"
        },
        "webSiteHostingPlanName": {
            "value": "DefaultPlan"
        },
        "webSiteLocation": {
            "value": "West US"
        },
        "adminPassword": {
            "reference": {
               "keyVault": {
                  "id": "/subscriptions/{guid}/resourceGroups/{group-name}/providers/Microsoft.KeyVault/vaults/{vault-name}"
               }, 
               "secretName": "sqlAdminPassword" 
            }   
        }
   }
}
```

参数文件的大小不能超过 64 KB。

如果需要为参数（如密码）提供敏感值，请将该值添加到密钥保管库。 在部署过程中检索密钥保管库，如前面的示例所示。 有关详细信息，请参阅[在部署期间传递安全值](../articles/azure-resource-manager/resource-manager-keyvault-parameter.md)。 

