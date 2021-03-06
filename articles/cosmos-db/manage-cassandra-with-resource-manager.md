---
title: 用于 Azure Cosmos DB Cassandra API 的资源管理器模板
description: 使用 Azure 资源管理器模板创建和配置 Azure Cosmos DB Cassandra API。
author: rockboyfor
ms.service: cosmos-db
ms.topic: how-to
origin.date: 05/19/2020
ms.date: 06/22/2020
ms.author: v-yeche
ms.openlocfilehash: 99da22d927c0a5bbc122fbdaf0690a8b8faac54a
ms.sourcegitcommit: 48b5ae0164f278f2fff626ee60db86802837b0b4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2020
ms.locfileid: "85098555"
---
<!--Verify successfully-->
# <a name="manage-azure-cosmos-db-cassandra-api-resources-using-azure-resource-manager-templates"></a>使用 Azure 资源管理器模板管理 Azure Cosmos DB Cassandra API 资源

在本文中，你将了解如何使用 Azure 资源管理器模板来帮助部署和管理 Azure Cosmos DB 帐户、密钥空间和表。

本文仅提供 Cassandra API 帐户的示例，若要查找其他 API 类型帐户的示例，请参阅以下文章：将 Azure 资源管理器模板与适用于 [SQL](manage-sql-with-resource-manager.md)、[Gremlin](manage-gremlin-with-resource-manager.md)、[MongoDB](manage-mongodb-with-resource-manager.md)、[表](manage-table-with-resource-manager.md)的 Azure Cosmos DB API 配合使用。

> [!IMPORTANT]
>
> * 帐户名称限制为 44 个字符，全部小写。
> * 若要更改吞吐量值，请用更新的 RU/s 重新部署模板。
> * 当你在 Azure Cosmos 帐户中添加或删除位置时，无法同时修改其他属性。 必须单独执行这些操作。

若要创建以下任何 Azure Cosmos DB 资源，请将下列示例模板复制到新的 json 文件中。 在部署具有不同名称和值的同一资源的多个实例时，可以选择创建要使用的参数 json 文件。 可以通过多种方式部署 Azure 资源管理器模板，包括 [Azure 门户](../azure-resource-manager/templates/deploy-portal.md)、[Azure CLI](../azure-resource-manager/templates/deploy-cli.md)、[Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md) 和 [GitHub](../azure-resource-manager/templates/deploy-to-azure-button.md)。

<a name="create-autoscale"></a>

## <a name="azure-cosmos-account-for-cassandra-with-autoscale-provisioned-throughput"></a>预配了自动缩放的吞吐量的 Cassandra 的 Azure Cosmos 帐户

此模板在两个区域创建一个 Azure Cosmos 帐户，其中包含用于一致性和故障转移的选项，以及为自动缩放吞吐量配置的密钥空间和表。 此模板还支持从 Azure 快速入门模板库进行一键部署。

[![部署到 Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cosmosdb-cassandra-autoscale%2Fazuredeploy.json)

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {
      "accountName": {
         "type": "string",
         "defaultValue": "[concat('cassandra-', uniqueString(resourceGroup().id))]",
         "metadata": {
            "description": "Cosmos DB account name, max length 44 characters"
         }
      },
      "location": {
         "type": "string",
         "defaultValue": "[resourceGroup().location]",   
         "metadata": {
            "description": "Location for the Cosmos DB account."
         }
      },
      "primaryRegion":{
         "type":"string",
         "metadata": {
            "description": "The primary replica region for the Cosmos DB account."
         }
      },
      "secondaryRegion":{
         "type":"string",
         "metadata": {
           "description": "The secondary replica region for the Cosmos DB account."
        }
      },
      "defaultConsistencyLevel": {
         "type": "string",
         "defaultValue": "Session",
         "allowedValues": [ 
            "Eventual", 
            "ConsistentPrefix", 
            "Session", 
            "BoundedStaleness", 
            "Strong" 
         ],
         "metadata": {
            "description": "The default consistency level of the Cosmos DB account."
         }
      },
      "maxStalenessPrefix": {
         "type": "int",
         "defaultValue": 100000,
         "minValue": 10,
         "maxValue": 1000000,
         "metadata": {
            "description": "Max stale requests. Required for BoundedStaleness. Valid ranges, Single Region: 10 to 1000000. Multi Region: 100000 to 1000000."
         }
      },
      "maxIntervalInSeconds": {
         "type": "int",
         "defaultValue": 300,
         "minValue": 5,
         "maxValue": 86400,
         "metadata": {
            "description": "Max lag time (seconds). Required for BoundedStaleness. Valid ranges, Single Region: 5 to 84600. Multi Region: 300 to 86400."
         }
      },
      "automaticFailover": {
         "type": "bool",
         "defaultValue": true,
         "allowedValues": [ 
            true, 
            false 
         ],
         "metadata": {
            "description": "Enable automatic failover for regions"
         }
      },
      "keyspaceName": {
         "type": "string",
         "metadata": {
            "description": "The name for the Cassandra Keyspace"
         }
      },
      "tableName": {
         "type": "string",
         "metadata": {
            "description": "The name for the Cassandra table"
         }
      },
      "throughputPolicy":{
            "type": "string",
            "defaultValue": "Autoscale",
            "allowedValues": [ 
               "Manual", 
               "Autoscale" 
            ],
            "metadata": {
                "description": "The throughput policy for the Cassandra table"
            }
        },
        "manualProvisionedThroughput": {
            "type": "int",
            "defaultValue": 400,
            "minValue": 400,
            "maxValue": 1000000,
            "metadata": {
                "description": "Throughput value when using Manaual Throughput Policy for the Cassandra table"
            }
        },
        "autoscaleMaxThroughput": {
            "type": "int",
            "defaultValue": 4000,
            "minValue": 4000,
            "maxValue": 1000000,
            "metadata": {
                "description": "Maximum throughput when using Autoscale Throughput Policy for the Cassandra table"
            }
        }
   },
   "variables": {
      "accountName": "[toLower(parameters('accountName'))]",
      "consistencyPolicy": {
         "Eventual": {
            "defaultConsistencyLevel": "Eventual"
         },
         "ConsistentPrefix": {
            "defaultConsistencyLevel": "ConsistentPrefix"
         },
         "Session": {
            "defaultConsistencyLevel": "Session"
         },
         "BoundedStaleness": {
            "defaultConsistencyLevel": "BoundedStaleness",
            "maxStalenessPrefix": "[parameters('maxStalenessPrefix')]",
            "maxIntervalInSeconds": "[parameters('maxIntervalInSeconds')]"
         },
         "Strong": {
            "defaultConsistencyLevel": "Strong"
         }
      },
      "locations": [
        {
            "locationName": "[parameters('primaryRegion')]",
            "failoverPriority": 0,
            "isZoneRedundant": false
        },
        {
            "locationName": "[parameters('secondaryRegion')]",
            "failoverPriority": 1,
            "isZoneRedundant": false
        }
      ],
      "throughputPolicy": {
            "Manual": {
                  "throughput": "[parameters('manualProvisionedThroughput')]"
            },
            "Autoscale": {
                  "autoscaleSettings": { "maxThroughput": "[parameters('autoscaleMaxThroughput')]" }
            }
        },
        "throughputPolicyToUse": "[if(equals(parameters('throughputPolicy'), 'Manual'), variables('throughputPolicy').Manual, variables('throughputPolicy').Autoscale)]"
   },
   "resources": [
      {
        "type": "Microsoft.DocumentDB/databaseAccounts",
        "name": "[variables('accountName')]",
        "apiVersion": "2020-04-01",
        "location": "[parameters('location')]",
        "kind": "GlobalDocumentDB",
        "properties": {
            "capabilities": [{ "name": "EnableCassandra" }],
            "consistencyPolicy": "[variables('consistencyPolicy')[parameters('defaultConsistencyLevel')]]",
            "locations": "[variables('locations')]",
            "databaseAccountOfferType": "Standard",
            "enableAutomaticFailover": "[parameters('automaticFailover')]"
        }
      },
      {
        "type": "Microsoft.DocumentDB/databaseAccounts/cassandraKeyspaces",
        "name": "[concat(variables('accountName'), '/', parameters('keyspaceName'))]",
        "apiVersion": "2020-04-01",
        "dependsOn": [ 
           "[resourceId('Microsoft.DocumentDB/databaseAccounts/', variables('accountName'))]" 
         ],
        "properties":{
            "resource":{
               "id": "[parameters('keyspaceName')]"
            }
        }
      },
      {
        "type": "Microsoft.DocumentDb/databaseAccounts/cassandraKeyspaces/tables",
        "name": "[concat(variables('accountName'), '/', parameters('keyspaceName'), '/', parameters('tableName'))]",
        "apiVersion": "2020-04-01",
        "dependsOn": [ 
           "[resourceId('Microsoft.DocumentDB/databaseAccounts/cassandraKeyspaces', variables('accountName'), parameters('keyspaceName'))]" 
         ],
        "properties": {
            "resource":{
                "id":  "[parameters('tableName')]",
                "schema": {
                  "columns": [
                    { "name": "loadid", "type": "uuid" },
                    { "name": "machine", "type": "uuid" },
                    { "name": "cpu", "type": "int" },
                    { "name": "mtime", "type": "int" },
                    { "name": "load", "type": "float" }
                  ],
                  "partitionKeys": [ 
                    { "name": "machine" }, 
                    { "name": "cpu" }, 
                    { "name": "mtime" } 
                  ],
                  "clusterKeys": [ 
                    { "name": "loadid", "orderBy": "asc" } 
                  ]
                }
            },
            "options": "[variables('throughputPolicyToUse')]"
        }
      }
   ]
}
```

<a name="create-manual"></a>

## <a name="azure-cosmos-account-for-cassandra-with-standard-provisioned-throughput"></a>预配了标准吞吐量的 Cassandra 的 Azure Cosmos 帐户

此模板在两个区域创建一个 Azure Cosmos 帐户，其中包含用于一致性和故障转移的选项，以及为标准吞吐量配置的密钥空间和表。 此模板还支持从 Azure 快速入门模板库进行一键部署。

[![部署到 Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cosmosdb-cassandra%2Fazuredeploy.json)

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "accountName": {
            "type": "string",
            "defaultValue": "[concat('sql-', uniqueString(resourceGroup().id))]",
            "metadata": {
                "description": "Cosmos DB account name, max length 44 characters"
            }
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",   
            "metadata": {
                "description": "Location for the Cosmos DB account."
            }
        },
        "primaryRegion":{
            "type":"string",
            "metadata": {
                "description": "The primary replica region for the Cosmos DB account."
            }
        },
        "secondaryRegion":{
            "type":"string",
            "metadata": {
              "description": "The secondary replica region for the Cosmos DB account."
          }
        },
        "defaultConsistencyLevel": {
            "type": "string",
            "defaultValue": "Session",
            "allowedValues": [ "Eventual", "ConsistentPrefix", "Session", "BoundedStaleness", "Strong" ],
            "metadata": {
                "description": "The default consistency level of the Cosmos DB account."
            }
        },
        "maxStalenessPrefix": {
            "type": "int",
            "defaultValue": 100000,
            "minValue": 10,
            "maxValue": 2147483647,
            "metadata": {
                "description": "Max stale requests. Required for BoundedStaleness. Valid ranges, Single Region: 10 to 1000000. Multi Region: 100000 to 1000000."
            }
        },
        "maxIntervalInSeconds": {
            "type": "int",
            "defaultValue": 300,
            "minValue": 5,
            "maxValue": 86400,
            "metadata": {
                "description": "Max lag time (seconds). Required for BoundedStaleness. Valid ranges, Single Region: 5 to 84600. Multi Region: 300 to 86400."
            }
        },
        "automaticFailover": {
            "type": "bool",
            "defaultValue": true,
            "allowedValues": [ true, false ],
            "metadata": {
                "description": "Enable automatic failover for regions"
            }
        },
        "keyspaceName": {
            "type": "string",
            "metadata": {
                "description": "The name for the Cassandra Keyspace"
            }
        },
        "tableName": {
            "type": "string",
            "metadata": {
                "description": "The name for the first Cassandra table"
            }
        },
        "throughput": {
            "type": "int",
            "defaultValue": 400,
            "minValue": 400,
            "maxValue": 1000000,
            "metadata": {
                "description": "The throughput for both Cassandra tables"
            }       
        }
    },
    "variables": {
        "accountName": "[toLower(parameters('accountName'))]",
        "consistencyPolicy": {
            "Eventual": {
                "defaultConsistencyLevel": "Eventual"
            },
            "ConsistentPrefix": {
                "defaultConsistencyLevel": "ConsistentPrefix"
            },
            "Session": {
                "defaultConsistencyLevel": "Session"
            },
            "BoundedStaleness": {
                "defaultConsistencyLevel": "BoundedStaleness",
                "maxStalenessPrefix": "[parameters('maxStalenessPrefix')]",
                "maxIntervalInSeconds": "[parameters('maxIntervalInSeconds')]"
            },
            "Strong": {
                "defaultConsistencyLevel": "Strong"
            }
        },
        "locations": 
        [ 
            {
                "locationName": "[parameters('primaryRegion')]",
                "failoverPriority": 0,
                "isZoneRedundant": false
            }, 
            {
                "locationName": "[parameters('secondaryRegion')]",
                "failoverPriority": 1,
                "isZoneRedundant": false
            }
        ]
    },
    "resources": 
    [
        {
            "type": "Microsoft.DocumentDB/databaseAccounts",
            "name": "[variables('accountName')]",
            "apiVersion": "2020-03-01",
            "location": "[parameters('location')]",
            "kind": "GlobalDocumentDB",
            "properties": {
                "capabilities": [{ "name": "EnableCassandra" }],
                "consistencyPolicy": "[variables('consistencyPolicy')[parameters('defaultConsistencyLevel')]]",
                "locations": "[variables('locations')]",
                "databaseAccountOfferType": "Standard",
                "enableAutomaticFailover": "[parameters('automaticFailover')]"
            }
        },
        {
            "type": "Microsoft.DocumentDB/databaseAccounts/cassandraKeyspaces",
            "name": "[concat(variables('accountName'), '/', parameters('keyspaceName'))]",
            "apiVersion": "2020-03-01",
            "dependsOn": [ "[resourceId('Microsoft.DocumentDB/databaseAccounts/', variables('accountName'))]" ],
            "properties":{
                "resource":{
                    "id": "[parameters('keyspaceName')]"
                }
            }
        },
        {
            "type": "Microsoft.DocumentDb/databaseAccounts/cassandraKeyspaces/tables",
            "name": "[concat(variables('accountName'), '/', parameters('keyspaceName'), '/', parameters('tableName'))]",
            "apiVersion": "2020-03-01",
            "dependsOn": [ "[resourceId('Microsoft.DocumentDB/databaseAccounts/cassandraKeyspaces', variables('accountName'), parameters('keyspaceName'))]" ],
            "properties":
            {
                "resource":{
                    "id":  "[parameters('tableName')]",
                    "schema": {
                        "columns": [
                            { "name": "loadid", "type": "uuid" },
                            { "name": "machine", "type": "uuid" },
                            { "name": "cpu", "type": "int" },
                            { "name": "mtime", "type": "int" },
                            { "name": "load", "type": "float" }
                        ],
                        "partitionKeys": [ 
                            { "name": "machine" }, 
                            { "name": "cpu" }, 
                            { "name": "mtime" } 
                        ],
                        "clusterKeys": [ 
                            { "name": "loadid", "orderBy": "asc" } 
                        ]
                    },
                    "options": { "throughput": "[parameters('throughput')]" }
                }
            }
        }
    ]
}
```

## <a name="next-steps"></a>后续步骤

下面是一些其他资源：

* [Azure 资源管理器文档](/azure-resource-manager/)

    <!--Not Available on [Azure Cosmos DB resource provider schema](https://docs.microsoft.com/azure/templates/microsoft.documentdb/allversions)-->
    
* [Azure Cosmos DB 快速入门模板](https://github.com/Azure/azure-quickstart-templates/?resourceType=Microsoft.DocumentDB&pageNumber=1&sort=Popular)
* [排查常见 Azure 资源管理器部署错误](../azure-resource-manager/templates/common-deployment-errors.md)

<!-- Update_Description: update meta properties, wording update, update link -->