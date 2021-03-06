---
title: Azure 事件网格 - 订阅验证疑难解答
description: 本文介绍如何解决订阅验证问题。
services: event-grid
author: Johnnytechn
ms.service: event-grid
ms.topic: conceptual
ms.date: 06/12/2020
ms.author: v-johya
ms.openlocfilehash: 237b9888812d1602107d524bb0cb928945d3e68b
ms.sourcegitcommit: 3de7d92ac955272fd140ec47b3a0a7b1e287ca14
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/12/2020
ms.locfileid: "84723853"
---
# <a name="troubleshoot-azure-event-grid-errors"></a>排查 Azure 事件网格错误
此故障排除指南介绍有关如何解决订阅验证问题的信息。 


## <a name="troubleshoot-event-subscription-validation"></a>排查事件订阅验证问题

在创建事件订阅的过程中，如果看到诸如 `The attempt to validate the provided endpoint https://your-endpoint-here failed. For more details, visit https://aka.ms/esvalidation` 之类的错误消息，则表明验证握手失败。 若要解决此错误，请验证以下各方面：

- 使用 Postman 或 curl 或类似工具，通过一个[示例 SubscriptionValidationEvent](webhook-event-delivery.md#validation-details) 请求正文向 Webhook URL 发出 HTTP POST 请求。
- 如果 Webhook 实现了同步验证握手机制，请验证 ValidationCode 是否作为响应的一部分返回。
- 如果 Webhook 实现了异步验证握手机制，请验证 HTTP POST 是否返回了“200 正常”。
- 如果 Webhook 在响应中返回了“403 (禁止访问)”，请检查 Webhook 是否位于 Azure 应用程序网关或 Web 应用程序防火墙后面。 如果是，则需要禁用这些防火墙规则，然后重新执行 HTTP POST：

  920300（请求缺少 Accept 标头，我们可以解决此问题）

  942430（受限 SQL 字符异常情况检测 (args)：已超出特殊字符数 (12)）

  920230（检测到多个 URL 编码）

  942130（SQL 注入攻击：检测到 SQL 同义反复。）

  931130（可能的远程文件包含 (RFI) 攻击 = 域外引用/链接）

> [!IMPORTANT]
> 有关 webhook 终结点验证的详细信息，请参阅 [Webhook 事件传送](webhook-event-delivery.md)。

## <a name="validate-event-grid-event-subscription-using-postman"></a>使用 Postman 验证事件网格事件订阅
下面是一个示例，演示如何使用 Postman 来验证事件网格事件的 webhook 订阅： 

![通过使用 Postman 实现的事件网格事件订阅验证](./media/troubleshoot-subscription-validation/event-subscription-validation-postman.png)

下面是 SubscriptionValidationEvent JSON 的示例：

```json
[
  {
    "id": "2d1781af-3a4c-4d7c-bd0c-e34b19da4e66",
    "topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "subject": "",
    "data": {
      "validationCode": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6",
    },
    "eventType": "Microsoft.EventGrid.SubscriptionValidationEvent",
    "eventTime": "2018-01-25T22:12:19.4556811Z",
    "metadataVersion": "1",
    "dataVersion": "1"
  }
]
```

下面是成功响应的示例：

```json
{
  "validationResponse": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6"
}
```

若要详细了解 webhook 事件网格事件验证，请参阅[事件网格事件的终结点验证](webhook-event-delivery.md#endpoint-validation-with-event-grid-events)。

## <a name="validate-cloud-event-subscription-using-postman"></a>使用 Postman 验证云事件订阅
下面是一个示例，演示如何使用 Postman 来验证云事件的 webhook 订阅： 

![通过使用 Postman 实现的云事件订阅验证](./media/troubleshoot-subscription-validation/cloud-event-subscription-validation-postman.png)

使用 HTTP OPTIONS 方法执行云事件验证****。 若要详细了解 webhook 云事件验证，请参阅[云事件的终结点验证](webhook-event-delivery.md#endpoint-validation-with-event-grid-events)。

## <a name="event-grid-event-subscription-validation-using-curl"></a>通过使用 Curl 实现的事件网格事件订阅验证 
下面是用于验证事件网格事件的 webhook 订阅的 Curl 命令的示例： 

```bash
curl -X POST -d '[{"id": "2d1781af-3a4c-4d7c-bd0c-e34b19da4e66","topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx","subject": "","data": {"validationCode": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6"},"eventType": "Microsoft.EventGrid.SubscriptionValidationEvent","eventTime": "2018-01-25T22:12:19.4556811Z", "metadataVersion": "1","dataVersion": "1"}]' -H 'Content-Type: application/json' https://{your-webhook-url.com}
```

## <a name="next-steps"></a>后续步骤
如需更多帮助，请在 [Stack Overflow 论坛](https://stackoverflow.com/questions/tagged/azure-eventgrid)中发布问题，或开具[支持票证](https://www.azure.cn/support/contact/)。 

