---
title: 将 Blob 存储事件发送到 Web 终结点 - 模板
description: 使用 Azure 事件网格和 Azure 资源管理器模板创建 Blob 存储帐户并订阅其事件。 将事件发送到 Webhook。
services: event-grid
keywords: ''
author: Johnnytechn
ms.author: v-johya
ms.date: 05/06/2020
ms.topic: quickstart
ms.service: event-grid
ms.custom: subject-armqs
ms.openlocfilehash: e7d4d3d558d83e95166919e82433e9c4c97affdf
ms.sourcegitcommit: 81241aa44adbcac0764e2b5eb865b96ae56da6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2020
ms.locfileid: "83002000"
---
# <a name="route-blob-storage-events-to-web-endpoint-by-using-azure-resource-manager-template"></a>使用 Azure 资源管理器模板将 Blob 存储事件路由到 Web 终结点

Azure 事件网格是针对云的事件处理服务。 在本文中，你将使用 **Azure 资源管理器模板**创建 Blob 存储帐户、订阅该 Blob 存储的事件，并触发事件来查看结果。 通常，你会将事件发送到处理事件数据并执行操作的终结点。 但是，为了简化本文，你将事件发送到收集并显示消息的 Web 应用。

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial/)。

## <a name="prerequisites"></a>先决条件

### <a name="create-a-message-endpoint"></a>创建消息终结点

在订阅 Blob 存储的事件之前，让我们创建事件消息的终结点。 通常情况下，终结点基于事件数据执行操作。 为了简化此快速入门，将部署用于显示事件消息的[预建的 Web 应用](https://github.com/Azure-Samples/azure-event-grid-viewer)。 所部署的解决方案包括应用服务计划、应用服务 Web 应用和 GitHub 中的源代码。

1. 选择“部署到 Azure”  将解决方案部署到你的订阅。 在 Azure 门户中，为参数提供值。

    [部署到 Azure](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fazure-event-grid-viewer%2Fmaster%2Fazuredeploy.json)
1. 部署可能需要几分钟才能完成。 部署成功后，请查看 Web 应用以确保它正在运行。 在 Web 浏览器中导航到 `https://<your-site-name>.chinacloudsites.cn`

1. 查看站点，但是尚未有事件发布到它。

   ![查看新站点](./media/blob-event-quickstart-portal/view-site.png)

## <a name="create-a-storage-account-with-an-event-grid-subscription"></a>创建包含事件网格订阅的存储帐户

### <a name="review-the-template"></a>查看模板

本快速入门中使用的模板来自 [Azure 快速入门模板](https://github.com/Azure/azure-quickstart-templates/tree/master/101-event-grid-subscription-and-storage)。

```JSON
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageName": {
            "type": "string",
            "defaultValue": "[concat('storage', uniqueString(resourceGroup().id))]",
            "metadata": {
                "description": "Provide a unique name for the Blob Storage account."
            }
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Provide a location for the Blob Storage account that supports Event Grid."
            }
        },
        "eventSubName": {
            "type": "string",
            "defaultValue": "subToStorage",
            "metadata": {
                "description": "Provide a name for the Event Grid subscription."
            }
        },
        "endpoint": {
            "type": "string",
            "metadata": {
                "description": "Provide the URL for the WebHook to receive events. Create your own endpoint for events."
            }
        }
    },
    "resources": [
        {
            "name": "[parameters('storageName')]",
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2017-10-01",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "StorageV2",
            "location": "[parameters('location')]",
            "tags": {},
            "properties": {
                "accessTier": "Hot"
            }
        },
        {
            "type": "Microsoft.Storage/storageAccounts/providers/eventSubscriptions",
            "name": "[concat(parameters('storageName'), '/Microsoft.EventGrid/', parameters('eventSubName'))]",
            "apiVersion": "2018-01-01",
            "dependsOn": [
                "[parameters('storageName')]"
            ],
            "properties": {
                "destination": {
                    "endpointType": "WebHook",
                    "properties": {
                        "endpointUrl": "[parameters('endpoint')]"
                    }
                },
                "filter": {
                    "subjectBeginsWith": "",
                    "subjectEndsWith": "",
                    "isSubjectCaseSensitive": false,
                    "includedEventTypes": [
                        "All"
                    ]
                }
            }
        }
    ]
}
```

该模板中定义了两个 Azure 资源：

* [**Microsoft.Storage/storageAccounts**](https://docs.microsoft.com/azure/templates/Microsoft.Storage/storageAccounts)：创建 Azure 存储帐户。
* [**Microsoft.Storage/storageAccounts/providers/eventSubscriptions**](https://docs.microsoft.com/azure/templates/microsoft.eventgrid/eventsubscriptions)：为存储帐户创建 Azure 事件网格订阅。

### <a name="deploy-the-template"></a>部署模板

1. 选择以下链接登录到 Azure 并打开一个模板。 该模板将创建 Key Vault 和机密。

    [部署到 Azure](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-event-grid-subscription-and-storage%2Fazuredeploy.json)

2. 指定“终结点”  ：提供你的 Web 应用的 URL 并将 `api/updates` 添加到主页 URL。
3. 选择“购买”以部署模板。 

  此处使用 Azure 门户来部署模板。 还可以使用 Azure PowerShell、Azure CLI 和 REST API。 若要了解其他部署方法，请参阅[部署模板](../azure-resource-manager/templates/deploy-powershell.md)。

> [!NOTE]
> 可在[此处](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Eventgrid)找到更多 Azure 事件网格模板示例。

## <a name="validate-the-deployment"></a>验证部署

再次查看 Web 应用，并注意现已向该应用发送了订阅验证事件。 选择眼睛图标以展开事件数据。 事件网格发送验证事件，以便终结点可以验证它是否想要接收事件数据。 Web 应用包含用于验证订阅的代码。

![查看订阅事件](./media/blob-event-quickstart-portal/view-subscription-event.png)

现在，让我们触发一个事件，看事件网格如何将消息分发到终结点。

通过上传一个文件来为 Blob 存储触发一个事件。 该文件不需要有任何特定内容。 本文假定你有一个名为 testfile.txt 的文件，但是可以使用任何文件。

将文件上传到 Azure Blob 存储时，事件网格会向你在订阅时配置的终结点发送一条消息。 消息采用 JSON 格式，它包含一个或多个事件的数组。 在以下示例中，JSON 消息包含一个事件的数组。 查看你的 Web 应用，将会看到已收到了一个 blob 已创建事件。

![查看结果](./media/blob-event-quickstart-portal/view-results.png)

## <a name="clean-up-resources"></a>清理资源

不再需要资源组时，[可将其删除](../azure-resource-manager/management/delete-resource-group.md?tabs=azure-portal#delete-resource-group
)。

## <a name="next-steps"></a>后续步骤

有关 Azure 资源管理器模板的详细信息，请参阅以下文章：

- [Azure 资源管理器文档](/azure-resource-manager)
- [在 Azure 资源管理器模板中定义资源](https://docs.microsoft.com/azure/templates/)
- [Azure 快速入门模板](https://azure.microsoft.com/resources/templates/)
- [Azure 事件网格概述](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Eventgrid)。
