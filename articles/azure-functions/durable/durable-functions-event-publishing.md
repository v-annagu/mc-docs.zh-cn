---
title: 从 Durable Functions 发布到 Azure 事件网格（预览）
description: 了解如何配置 Durable Functions 的自动 Azure 事件网格发布。
ms.topic: conceptual
ms.date: 06/09/2020
ms.openlocfilehash: 909afdd758709c70bfc844320de81d1a77b3fb53
ms.sourcegitcommit: f1a76ee3242698123a3d77f44c860db040b48f70
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/09/2020
ms.locfileid: "84563732"
---
# <a name="durable-functions-publishing-to-azure-event-grid-preview"></a>从 Durable Functions 发布到 Azure 事件网格（预览）

本文介绍了如何设置 Durable Functions，以便将业务流程生命周期事件（例如“已创建”、“已完成”和“失败”）发布到自定义的 [Azure 事件网格主题](/event-grid/overview)。

此功能在以下场景中非常有用：

* **蓝/绿部署等 DevOps 场景**：在实施[并列部署策略](durable-functions-versioning.md#side-by-side-deployments)之前，你可能想要了解是否有任何任务正在运行。

* **高级监视和诊断支持**：可以在已针对查询优化的外部存储（例如 Azure SQL 数据库或 Azure Cosmos DB）中跟踪业务流程状态信息。

* **长时间运行的后台活动**：如果对长时间运行的后台活动使用 Durable Functions，此功能有助于了解当前状态。

## <a name="prerequisites"></a>先决条件

* 在 Durable Functions 项目中安装 [Microsoft.Azure.WebJobs.Extensions.DurableTask](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.DurableTask)。
* 安装 [Azure 存储模拟器](../../storage/common/storage-use-emulator.md)（仅限 Windows）或使用现有的 Azure 存储帐户。
* 安装 [Azure CLI](/cli/?view=azure-cli-latest) 

## <a name="create-a-custom-event-grid-topic"></a>创建自定义事件网格主题

创建事件网格主题，以便从 Durable Functions 发送事件。 以下说明介绍如何使用 Azure CLI 创建主题。 也可以[使用 PowerShell](../../event-grid/custom-event-quickstart-powershell.md) 或[使用 Azure 门户](../../event-grid/custom-event-quickstart-portal.md)来创建主题。

### <a name="create-a-resource-group"></a>创建资源组

使用 `az group create` 命令创建资源组。 目前，Azure 事件网格不支持所有区域。 有关支持哪些区域的信息，请参阅 [Azure 事件网格概述](../../event-grid/overview.md)。

```azurecli
az group create --name eventResourceGroup --location chinanorth2
```

### <a name="create-a-custom-topic"></a>创建自定义主题

事件网格主题提供用户定义的终结点，可向该终结点发布事件。 用主题的唯一名称替换 `<topic_name>`。 主题名称必须唯一，因为它将用作 DNS 条目。

```azurecli
az eventgrid topic create --name <topic_name> -l chinanorth2 -g eventResourceGroup
```

## <a name="get-the-endpoint-and-key"></a>获取终结点和密钥

获取主题的终结点。 将 `<topic_name>` 替换为所选的名称。

```azurecli
az eventgrid topic show --name <topic_name> -g eventResourceGroup --query "endpoint" --output tsv
```

获取主题密钥。 将 `<topic_name>` 替换为所选的名称。

```azurecli
az eventgrid topic key list --name <topic_name> -g eventResourceGroup --query "key1" --output tsv
```

现在，可将事件发送到主题。

## <a name="configure-event-grid-publishing"></a>配置事件网格发布

在 Durable Functions 项目中，找到 `host.json` 文件。

### <a name="durable-functions-1x"></a>Durable Functions 1.x

在 `durableTask` 属性中添加 `eventGridTopicEndpoint` 和 `eventGridKeySettingName`。

```json
{
  "durableTask": {
    "eventGridTopicEndpoint": "https://<topic_name>.chinanorth2-1.eventgrid.chinacloudapi.cn/api/events",
    "eventGridKeySettingName": "EventGridKey"
  }
}
```

### <a name="durable-functions-2x"></a>Durable Functions 2.x

在文件的 `durableTask` 属性中添加 `notifications` 部分，并将 `<topic_name>` 替换为所选的名称。 如果 `durableTask` 或 `extensions` 属性不存在，请像下面的示例一样创建它们：

```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "notifications": {
        "eventGrid": {
          "topicEndpoint": "https://<topic_name>.chinanorth2-1.eventgrid.chinacloudapi.cn/api/events",
          "keySettingName": "EventGridKey"
        }
      }
    }
  }
}
```

可能的 Azure 事件网格配置属性可以在 [host.json 文档](../functions-host-json.md#durabletask)中找到。 配置 `host.json` 文件后，函数应用会将生命周期事件发送到事件网格主题。 同时在本地和 Azure 中运行函数应用时，将启动此操作。

在函数应用和 `local.settings.json` 中设置主题密钥的应用设置。 以下 JSON 是用于本地调试的 `local.settings.json` 示例。 将 `<topic_key>` 替换为主题密钥。  

```json
{
    "IsEncrypted": false,
    "Values": {
        "AzureWebJobsStorage": "UseDevelopmentStorage=true",
        "AzureWebJobsDashboard": "UseDevelopmentStorage=true",
        "EventGridKey": "<topic_key>"
    }
}
```

如果使用的是[存储模拟器](../../storage/common/storage-use-emulator.md)（仅限 Windows），请确保它正常工作。 最好是在执行之前运行 `AzureStorageEmulator.exe clear all` 命令。

如果使用的是现有 Azure 存储帐户，请将 `local.settings.json` 中的 `UseDevelopmentStorage=true` 替换为其连接字符串。

## <a name="create-functions-that-listen-for-events"></a>创建用于侦听事件的函数

使用 Azure 门户，创建另一个函数应用来侦听 Durable Functions 应用发布的事件。 最好是将该应用放置在事件网格主题所在的同一区域。

### <a name="create-an-event-grid-trigger-function"></a>创建事件网格触发器函数

1. 在函数应用中，选择“函数”****，然后选择“+ 添加”**** 

   :::image type="content" source="./media/durable-functions-event-publishing/function-add-function.png" alt-text="在 Azure 门户中添加函数。" border="true":::

1. 搜索“事件网格”****，然后选择“Azure 事件网格触发器”**** 模板。 

    :::image type="content" source="./media/durable-functions-event-publishing/function-select-event-grid-trigger.png" alt-text="在 Azure 门户中选择事件网格触发器模板。" border="true":::

1. 为新触发器命名，然后选择“创建函数”****。

    :::image type="content" source="./media/durable-functions-event-publishing/function-name-event-grid-trigger.png" alt-text="在 Azure 门户中为事件网格触发器命名。" border="true":::


    创建包含以下代码的函数：

    # <a name="c-script"></a>[C# 脚本](#tab/csharp-script)

    ```csharp
    #r "Newtonsoft.Json"
    using Newtonsoft.Json;
    using Newtonsoft.Json.Linq;
    using Microsoft.Extensions.Logging;

    public static void Run(JObject eventGridEvent, ILogger log)
    {
        log.LogInformation(eventGridEvent.ToString(Formatting.Indented));
    }
    ```

   # <a name="javascript"></a>[JavaScript](#tab/javascript)

   ```javascript
   module.exports = async function(context, eventGridEvent) {
       context.log(typeof eventGridEvent);
       context.log(eventGridEvent);
   }
   ```

---

### <a name="add-an-event-grid-subscription"></a>添加事件网格订阅

现在可以为创建的事件网格主题添加事件网格订阅。 有关详细信息，请参阅 [Azure 事件网格中的概念](/event-grid/concepts)。

1. 在新函数中，选择“集成”****，然后选择“事件网格触发器(eventGridEvent)”****。 

    :::image type="content" source="./media/durable-functions-event-publishing/eventgrid-trigger-link.png" alt-text="选择“事件网格触发器”链接。" border="true":::

1. 选择“创建事件网格描述”****。

    :::image type="content" source="./media/durable-functions-event-publishing/create-event-grid-subscription.png" alt-text="创建事件网格订阅。" border="true":::

1. 为事件订阅命名，并选择“事件网格主题”**** 主题类型。 

1. 选择订阅。 然后，选择为事件网格主题创建的资源组和资源。 

1. 选择“创建” ****。

    :::image type="content" source="./media/durable-functions-event-publishing/event-grid-subscription-details.png" alt-text="创建事件网格订阅。" border="true":::

现已准备好接收生命周期事件。

## <a name="run-durable-functions-app-to-send-the-events"></a>运行 Durable Functions 应用以发送事件

在前面配置的 Durable Functions 项目中，开始在本地计算机上调试并启动业务流程。 该应用将 Durable Functions 生命周期事件发布到事件网格。 通过检查 Azure 门户中的事件网格日志，验证事件网格是否触发了所创建的侦听器函数。

```
2019-04-20T09:28:21.041 [Info] Function started (Id=3301c3ef-625f-40ce-ad4c-9ba2916b162d)
2019-04-20T09:28:21.104 [Info] {
    "id": "054fe385-c017-4ce3-b38a-052ac970c39d",
    "subject": "durable/orchestrator/Running",
    "data": {
        "hubName": "DurableFunctionsHub",
        "functionName": "Sample",
        "instanceId": "055d045b1c8a415b94f7671d8df693a6",
        "reason": "",
        "runtimeStatus": "Running"
    },
    "eventType": "orchestratorEvent",
    "eventTime": "2019-04-20T09:28:19.6492068Z",
    "dataVersion": "1.0",
    "metadataVersion": "1",
    "topic": "/subscriptions/<your_subscription_id>/resourceGroups/eventResourceGroup/providers/Microsoft.EventGrid/topics/durableTopic"
}

2019-04-20T09:28:21.104 [Info] Function completed (Success, Id=3301c3ef-625f-40ce-ad4c-9ba2916b162d, Duration=65ms)
2019-04-20T09:28:37.098 [Info] Function started (Id=36fadea5-198b-4345-bb8e-2837febb89a2)
2019-04-20T09:28:37.098 [Info] {
    "id": "8cf17246-fa9c-4dad-b32a-5a868104f17b",
    "subject": "durable/orchestrator/Completed",
    "data": {
        "hubName": "DurableFunctionsHub",
        "functionName": "Sample",
        "instanceId": "055d045b1c8a415b94f7671d8df693a6",
        "reason": "",
        "runtimeStatus": "Completed"
    },
    "eventType": "orchestratorEvent",
    "eventTime": "2019-04-20T09:28:36.5061317Z",
    "dataVersion": "1.0",
    "metadataVersion": "1",
    "topic": "/subscriptions/<your_subscription_id>/resourceGroups/eventResourceGroup/providers/Microsoft.EventGrid/topics/durableTopic"
}
2019-04-20T09:28:37.098 [Info] Function completed (Success, Id=36fadea5-198b-4345-bb8e-2837febb89a2, Duration=0ms)
```

## <a name="event-schema"></a>事件架构

以下列表解释了生命周期事件架构：

* **`id`** ：事件网格事件的唯一标识符。
* **`subject`** ：事件主题的路径。 `durable/orchestrator/{orchestrationRuntimeStatus}`。 `{orchestrationRuntimeStatus}` 为 `Running`、`Completed`、`Failed` 和 `Terminated`。  
* **`data`** ：Durable Functions 特定的参数。
  * **`hubName`** ：[任务中心](durable-functions-task-hubs.md)名称。
  * **`functionName`** ：业务流程协调程序函数名称。
  * **`instanceId`** ：Durable Functions instanceId。
  * **`reason`** ：与跟踪事件关联的其他数据。 
  * **`runtimeStatus`** ：业务流程运行时状态。 值为 Running、Completed、Failed 和 Canceled。
* **`eventType`** ：“orchestratorEvent”
* **`eventTime`** ：事件时间 (UTC)。
* **`dataVersion`** ：生命周期事件架构的版本。
* **`metadataVersion`** ：元数据的版本。
* **`topic`** ：事件网格主题资源。

## <a name="how-to-test-locally"></a>如何在本地测试

若要进行本地测试，请阅读 [Azure Functions 事件网格触发器本地调试](../functions-debug-event-grid-trigger-local.md)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [了解 Durable Functions 中的实例管理](durable-functions-instance-management.md)

> [!div class="nextstepaction"]
> [了解 Durable Functions 中的版本控制](durable-functions-versioning.md)

