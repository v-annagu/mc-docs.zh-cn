---
author: lugoldbemicrosoft
ms.service: data-explorer
ms.topic: include
origin.date: 02/03/2020
ms.date: 05/15/2020
ms.author: v-tawe
ms.openlocfilehash: 060c74f3e2cd186a252c9e35cc763500852fd2a1
ms.sourcegitcommit: bfbd6694da33f703481386f2a3f16850c4e94bfa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/15/2020
ms.locfileid: "83417635"
---
## <a name="azure-resource-manager-template"></a>Azure 资源管理器模板

在本文中，你将使用 Azure 资源管理器模板来创建资源组、存储帐户和容器、事件中心以及 Azure 数据资源管理器群集和数据库。 将以下内容保存在名为 `template.json` 的文件中。 使用此文件来运行代码示例。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "eventHubNamespaceName": {
            "type": "string",
            "metadata": {
                "description": "Specifies a the event hub Namespace name."
            }
        },
        "eventHubName": {
            "type": "string",
            "metadata": {
                "description": "Specifies a event hub name."
            }
        },
        "storageAccountType": {
            "type": "string",
            "defaultValue": "Standard_LRS",
            "allowedValues": ["Standard_LRS", "Standard_GRS", "Standard_ZRS", "Premium_LRS"],
            "metadata": {
                "description": "Storage Account type"
            }
        },
        "storageAccountName": {
            "type": "string",
            "defaultValue": "[concat('storage', uniqueString(resourceGroup().id))]",
            "metadata": {
                "description": "Name of the storage account to create"
            }
        },
        "containerName": {
            "type": "string",
            "defaultValue": "[concat('storagecontainer', uniqueString(resourceGroup().id))]",
            "metadata": {
                "description": "Name of the container in storage account to create"
            }
        },
        "eventHubSku": {
            "type": "string",
            "allowedValues": ["Basic", "Standard"],
            "defaultValue": "Standard",
            "metadata": {
                "description": "Specifies the messaging tier for service Bus namespace."
            }
        },
        "kustoClusterName": {
            "type": "string",
            "defaultValue": "[concat('kusto', uniqueString(resourceGroup().id))]",
            "metadata": {
                "description": "Name of the cluster to create"
            }
        },
        "kustoDatabaseName": {
            "type": "string",
            "defaultValue": "kustodb",
            "metadata": {
                "description": "Name of the database to create"
            }
        },
        "clusterPrincipalAssignmentName": {
            "type": "string",
            "defaultValue": "clusterPrincipalAssignment1",
            "metadata": {
                "description": "Specifies the name of the principal assignment"
            }
        },
        "principalIdForCluster": {
            "type": "string",
            "metadata": {
                "description": "Specifies the principal id. It can be user email, application (client) ID, security group name"
            }
        },
        "roleForClusterPrincipal": {
            "type": "string",
            "defaultValue": "AllDatabasesViewer",
            "metadata": {
                "description": "Specifies the cluster principal role. It can be 'AllDatabasesAdmin', 'AllDatabasesViewer'"
            }
        },
        "tenantIdForClusterPrincipal": {
            "type": "string",
            "metadata": {
                "description": "Specifies the tenantId of the cluster principal"
            }
        },
        "principalTypeForCluster": {
            "type": "string",
            "defaultValue": "App",
            "metadata": {
                "description": "Specifies the principal type. It can be 'User', 'App', 'Group'"
            }
        },
        "databasePrincipalAssignmentName": {
            "type": "string",
            "defaultValue": "databasePrincipalAssignment1",
            "metadata": {
                "description": "Specifies the name of the principal assignment"
            }
        },
        "principalIdForDatabase": {
            "type": "string",
            "metadata": {
                "description": "Specifies the principal id. It can be user email, application (client) ID, security group name"
            }
        },
        "roleForDatabasePrincipal": {
            "type": "string",
            "defaultValue": "Admin",
            "metadata": {
                "description": "Specifies the database principal role. It can be 'Admin', 'Ingestor', 'Monitor', 'User', 'UnrestrictedViewers', 'Viewer'"
            }
        },
        "tenantIdForDatabasePrincipal": {
            "type": "string",
            "metadata": {
                "description": "Specifies the tenantId of the database principal"
            }
        },
        "principalTypeForDatabase": {
            "type": "string",
            "defaultValue": "App",
            "metadata": {
                "description": "Specifies the principal type. It can be 'User', 'App', 'Group'"
            }
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for all resources."
            }
        }
    },
    "variables": {
    },
    "resources": [{
            "apiVersion": "2017-04-01",
            "type": "Microsoft.EventHub/namespaces",
            "name": "[parameters('eventHubNamespaceName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "[parameters('eventHubSku')]",
                "tier": "[parameters('eventHubSku')]",
                "capacity": 1
            },
            "properties": {
                "isAutoInflateEnabled": false,
                "maximumThroughputUnits": 0
            }
        }, {
            "apiVersion": "2017-04-01",
            "type": "Microsoft.EventHub/namespaces/eventhubs",
            "name": "[concat(parameters('eventHubNamespaceName'), '/', parameters('eventHubName'))]",
            "location": "[parameters('location')]",
            "dependsOn": ["[resourceId('Microsoft.EventHub/namespaces', parameters('eventHubNamespaceName'))]"],
            "properties": {
                "messageRetentionInDays": 7,
                "partitionCount": 1
            }
        }, {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[parameters('storageAccountName')]",
            "location": "[parameters('location')]",
            "apiVersion": "2018-07-01",
            "sku": {
                "name": "[parameters('storageAccountType')]"
            },
            "kind": "StorageV2",
            "resources": [
                {
                    "name": "[concat('default/', parameters('containerName'))]",
                    "type": "blobServices/containers",
                    "apiVersion": "2018-07-01",
                    "dependsOn": [
                        "[parameters('storageAccountName')]"
                    ],
                    "properties": {
                        "publicAccess": "None"
                    }
                }
            ],
            "properties": {}
        }, {
            "name": "[parameters('kustoClusterName')]",
            "type": "Microsoft.Kusto/clusters",
            "sku": {
                "name": "Standard_D13_v2",
                "tier": "Standard",
                "capacity": 2
            },
            "apiVersion": "2019-09-07",
            "location": "[parameters('location')]",
            "tags": {
                "Created By": "GitHub quickstart template"
            }
        }, {
            "name": "[concat(parameters('kustoClusterName'), '/', parameters('kustoDatabaseName'))]",
            "type": "Microsoft.Kusto/clusters/databases",
            "apiVersion": "2019-09-07",
            "location": "[parameters('location')]",
            "dependsOn": ["[resourceId('Microsoft.Kusto/clusters', parameters('kustoClusterName'))]"],
            "properties": {
                "softDeletePeriodInDays": 365,
                "hotCachePeriodInDays": 31
            }
        }, {
            "type": "Microsoft.Kusto/Clusters/principalAssignments",
            "apiVersion": "2019-11-09",
            "name": "[concat(parameters('kustoClusterName'), '/', parameters('clusterPrincipalAssignmentName'))]",
            "dependsOn": ["[resourceId('Microsoft.Kusto/clusters', parameters('kustoClusterName'))]"],
            "properties": {
                "principalId": "[parameters('principalIdForCluster')]",
                "role": "[parameters('roleForClusterPrincipal')]",
                "tenantId": "[parameters('tenantIdForClusterPrincipal')]",
                "principalType": "[parameters('principalTypeForCluster')]"
            }
        }, {
            "type": "Microsoft.Kusto/Clusters/Databases/principalAssignments",
            "apiVersion": "2019-11-09",
            "name": "[concat(parameters('kustoClusterName'), '/', parameters('kustoDatabaseName'), '/', parameters('databasePrincipalAssignmentName'))]",
            "dependsOn": ["[resourceId('Microsoft.Kusto/clusters/databases', parameters('kustoClusterName'), parameters('kustoDatabaseName'))]"],
            "properties": {
                "principalId": "[parameters('principalIdForDatabase')]",
                "role": "[parameters('roleForDatabasePrincipal')]",
                "tenantId": "[parameters('tenantIdForDatabasePrincipal')]",
                "principalType": "[parameters('principalTypeForDatabase')]"
            }
        }
    ]
}

```
