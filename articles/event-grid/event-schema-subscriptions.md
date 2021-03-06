---
title: 充当事件网格源的 Azure 订阅
description: 介绍为 Azure 事件网格的订阅事件提供的属性
services: event-grid
author: Johnnytechn
ms.service: event-grid
ms.topic: reference
ms.date: 05/06/2020
ms.author: v-johya
ms.openlocfilehash: de8107d8c51c0daa14d4cda1cbcf5ac152cd5202
ms.sourcegitcommit: 81241aa44adbcac0764e2b5eb865b96ae56da6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2020
ms.locfileid: "83001959"
---
# <a name="azure-subscription-as-an-event-grid-source"></a>充当事件网格源的 Azure 订阅

本文提供 Azure 订阅事件的属性和架构。 有关事件架构的简介，请参阅 [Azure 事件网格事件架构](event-schema.md)。

Azure 订阅和资源组发出相同的事件类型。 这些事件类型与资源更改或操作相关。 主要区别是资源组针对资源组中的资源发出事件，Azure 订阅针对跨订阅的资源发出事件。

已为发送到 `management.chinacloudapi.cn` 的 PUT、PATCH、POST 和 DELETE 操作创建资源事件。 GET 操作不创建事件。 发送到数据平面的操作（如 `myaccount.blob.core.chinacloudapi.cn`）不会创建事件。 操作事件为操作（例如列出资源的键）提供事件数据。

当订阅 Azure 订阅的事件时，终结点接收该订阅的所有事件。 事件可能包括要查看的事件（例如更新虚拟机），以及可能不重要的事件（例如在部署历史记录中编写新条目）。 可以在终结点接收所有事件，并编写代码用于处理需要处理的事件。 或可以在创建事件订阅时设置一个筛选器。

若要以编程方式处理事件，可通过查看 `operationName` 值对事件进行排序。 例如，事件终结点可能只处理值等于 `Microsoft.Compute/virtualMachines/write` 或 `Microsoft.Storage/storageAccounts/write` 的操作的事件。

事件主题是作为操作目标的资源的资源 ID。 若要筛选资源的事件，请在创建事件订阅时提供该资源 ID。 若要按资源类型筛选，请使用以下格式的值：`/subscriptions/<subscription-id>/resourcegroups/<resource-group>/providers/Microsoft.Compute/virtualMachines`


## <a name="event-grid-event-schema"></a>事件网格事件架构

### <a name="available-event-types"></a>可用事件类型

Azure 订阅从 Azure 资源管理器发出管理事件，例如，在创建 VM 或删除存储帐户时。

| 事件类型 | 说明 |
| ---------- | ----------- |
| Microsoft.Resources.ResourceActionCancel | 在资源操作被取消时引发。 |
| Microsoft.Resources.ResourceActionFailure | 在资源操作失败时引发。 |
| Microsoft.Resources.ResourceActionSuccess | 在资源操作成功时引发。 |
| Microsoft.Resources.ResourceDeleteCancel | 在删除操作被取消时引发。 取消模板部署时会发生此事件。 |
| Microsoft.Resources.ResourceDeleteFailure | 在删除操作失败时引发。 |
| Microsoft.Resources.ResourceDeleteSuccess | 在删除操作成功时引发。 |
| Microsoft.Resources.ResourceWriteCancel | 在创建或更新操作被取消时引发。 |
| Microsoft.Resources.ResourceWriteFailure | 在创建或更新操作失败时引发。 |
| Microsoft.Resources.ResourceWriteSuccess | 在创建或更新操作成功时引发。 |

### <a name="example-event"></a>示例事件

以下示例展示了 ResourceWriteSuccess 事件的架构  。 具有不同 `eventType` 值的 ResourceWriteFailure 和 ResourceWriteCancel 事件会使用相同的模式   。

```json
[{
  "subject": "/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
  "eventType": "Microsoft.Resources.ResourceWriteSuccess",
  "eventTime": "2018-07-19T18:38:04.6117357Z",
  "id": "4db48cba-50a2-455a-93b4-de41a3b5b7f6",
  "data": {
    "authorization": {
      "scope": "/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
      "action": "Microsoft.Storage/storageAccounts/write",
      "evidence": {
        "role": "Subscription Admin"
      }
    },
    "claims": {
      "aud": "{audience-claim}",
      "iss": "{issuer-claim}",
      "iat": "{issued-at-claim}",
      "nbf": "{not-before-claim}",
      "exp": "{expiration-claim}",
      "_claim_names": "{\"groups\":\"src1\"}",
      "_claim_sources": "{\"src1\":{\"endpoint\":\"{URI}\"}}",
      "http://schemas.microsoft.com/claims/authnclassreference": "1",
      "aio": "{token}",
      "http://schemas.microsoft.com/claims/authnmethodsreferences": "rsa,mfa",
      "appid": "{ID}",
      "appidacr": "2",
      "http://schemas.microsoft.com/2012/01/devicecontext/claims/identifier": "{ID}",
      "e_exp": "{expiration}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname": "{last-name}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname": "{first-name}",
      "ipaddr": "{IP-address}",
      "name": "{full-name}",
      "http://schemas.microsoft.com/identity/claims/objectidentifier": "{ID}",
      "onprem_sid": "{ID}",
      "puid": "{ID}",
      "http://schemas.microsoft.com/identity/claims/scope": "user_impersonation",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "{ID}",
      "http://schemas.microsoft.com/identity/claims/tenantid": "{ID}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": "{user-name}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "{user-name}",
      "uti": "{ID}",
      "ver": "1.0"
    },
    "correlationId": "{ID}",
    "resourceProvider": "Microsoft.Storage",
    "resourceUri": "/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
    "operationName": "Microsoft.Storage/storageAccounts/write",
    "status": "Succeeded",
    "subscriptionId": "{subscription-id}",
    "tenantId": "{tenant-id}"
  },
  "dataVersion": "2",
  "metadataVersion": "1",
  "topic": "/subscriptions/{subscription-id}"
}]
```

以下示例展示了 ResourceDeleteSuccess 事件的架构  。 具有不同 `eventType` 值的 ResourceDeleteFailure 和 ResourceDeleteCancel 事件会使用相同的模式   。

```json
[{
  "subject": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
  "eventType": "Microsoft.Resources.ResourceDeleteSuccess",
  "eventTime": "2018-07-19T19:24:12.763881Z",
  "id": "19a69642-1aad-4a96-a5ab-8d05494513ce",
  "data": {
    "authorization": {
      "scope": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
      "action": "Microsoft.Storage/storageAccounts/delete",
      "evidence": {
        "role": "Subscription Admin"
      }
    },
    "claims": {
      "aud": "{audience-claim}",
      "iss": "{issuer-claim}",
      "iat": "{issued-at-claim}",
      "nbf": "{not-before-claim}",
      "exp": "{expiration-claim}",
      "_claim_names": "{\"groups\":\"src1\"}",
      "_claim_sources": "{\"src1\":{\"endpoint\":\"{URI}\"}}",
      "http://schemas.microsoft.com/claims/authnclassreference": "1",
      "aio": "{token}",
      "http://schemas.microsoft.com/claims/authnmethodsreferences": "rsa,mfa",
      "appid": "{ID}",
      "appidacr": "2",
      "http://schemas.microsoft.com/2012/01/devicecontext/claims/identifier": "{ID}",
      "e_exp": "262800",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname": "{last-name}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname": "{first-name}",
      "ipaddr": "{IP-address}",
      "name": "{full-name}",
      "http://schemas.microsoft.com/identity/claims/objectidentifier": "{ID}",
      "onprem_sid": "{ID}",
      "puid": "{ID}",
      "http://schemas.microsoft.com/identity/claims/scope": "user_impersonation",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "{ID}",
      "http://schemas.microsoft.com/identity/claims/tenantid": "{ID}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": "{user-name}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "{user-name}",
      "uti": "{ID}",
      "ver": "1.0"
    },
    "correlationId": "{ID}",
    "httpRequest": {
      "clientRequestId": "{ID}",
      "clientIpAddress": "{IP-address}",
      "method": "DELETE",
      "url": "https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}?api-version=2018-02-01"
    },
    "resourceProvider": "Microsoft.Storage",
    "resourceUri": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
    "operationName": "Microsoft.Storage/storageAccounts/delete",
    "status": "Succeeded",
    "subscriptionId": "{subscription-id}",
    "tenantId": "{tenant-id}"
  },
  "dataVersion": "2",
  "metadataVersion": "1",
  "topic": "/subscriptions/{subscription-id}"
}]
```

以下示例展示了 ResourceActionSuccess 事件的架构  。 具有不同 `eventType` 值的 ResourceActionFailure 和 ResourceActionCancel 事件会使用相同的模式   。

```json
[{   
  "subject": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.EventHub/namespaces/{namespace}/AuthorizationRules/RootManageSharedAccessKey",
  "eventType": "Microsoft.Resources.ResourceActionSuccess",
  "eventTime": "2018-10-08T22:46:22.6022559Z",
  "id": "{ID}",
  "data": {
    "authorization": {
      "scope": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.EventHub/namespaces/{namespace}/AuthorizationRules/RootManageSharedAccessKey",
      "action": "Microsoft.EventHub/namespaces/AuthorizationRules/listKeys/action",
      "evidence": {
        "role": "Contributor",
        "roleAssignmentScope": "/subscriptions/{subscription-id}",
        "roleAssignmentId": "{ID}",
        "roleDefinitionId": "{ID}",
        "principalId": "{ID}",
        "principalType": "ServicePrincipal"
      }     
    },
    "claims": {
      "aud": "{audience-claim}",
      "iss": "{issuer-claim}",
      "iat": "{issued-at-claim}",
      "nbf": "{not-before-claim}",
      "exp": "{expiration-claim}",
      "aio": "{token}",
      "appid": "{ID}",
      "appidacr": "2",
      "http://schemas.microsoft.com/identity/claims/identityprovider": "{URL}",
      "http://schemas.microsoft.com/identity/claims/objectidentifier": "{ID}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "{ID}",       "http://schemas.microsoft.com/identity/claims/tenantid": "{ID}",
      "uti": "{ID}",
      "ver": "1.0"
    },
    "correlationId": "{ID}",
    "httpRequest": {
      "clientRequestId": "{ID}",
      "clientIpAddress": "{IP-address}",
      "method": "POST",
      "url": "https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.EventHub/namespaces/{namespace}/AuthorizationRules/RootManageSharedAccessKey/listKeys?api-version=2017-04-01"
    },
    "resourceProvider": "Microsoft.EventHub",
    "resourceUri": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.EventHub/namespaces/{namespace}/AuthorizationRules/RootManageSharedAccessKey",
    "operationName": "Microsoft.EventHub/namespaces/AuthorizationRules/listKeys/action",
    "status": "Succeeded",
    "subscriptionId": "{subscription-id}",
    "tenantId": "{tenant-id}"
  },
  "dataVersion": "2",
  "metadataVersion": "1",
  "topic": "/subscriptions/{subscription-id}" 
}]
```

### <a name="event-properties"></a>事件属性

事件具有以下顶级数据：

| 属性 | 类型 | 说明 |
| -------- | ---- | ----------- |
| 主题 | string | 事件源的完整资源路径。 此字段不可写入。 事件网格提供此值。 |
| subject | string | 事件主题的发布者定义路径。 |
| eventType | string | 此事件源的一个注册事件类型。 |
| EventTime | string | 基于提供程序 UTC 时间的事件生成时间。 |
| id | string | 事件的唯一标识符。 |
| 数据 | object | 订阅事件数据。 |
| dataVersion | string | 数据对象的架构版本。 发布者定义架构版本。 |
| metadataVersion | string | 事件元数据的架构版本。 事件网格定义顶级属性的架构。 事件网格提供此值。 |

数据对象具有以下属性：

| 属性 | 类型 | 说明 |
| -------- | ---- | ----------- |
| authorization | object | 操作请求的授权。 |
| 声明 | object | 声明的属性。 有关详细信息，请参阅 [JWT 规范](https://self-issued.info/docs/draft-ietf-oauth-json-web-token.html)。 |
| correlationId | string | 用于故障排除的操作 ID。 |
| httpRequest | object | 操作的详细信息。 仅在更新现有资源或删除资源时才包含此对象。 |
| resourceProvider | string | 操作的资源提供程序。 |
| resourceUri | string | 操作中资源的 URI。 |
| operationName | string | 执行的操作。 |
| 状态 | string | 操作状态。 |
| subscriptionId | string | 资源的订阅 ID。 |
| tenantId | string | 资源的租户 ID。 |

## <a name="tutorials-and-how-tos"></a>教程和操作指南
|标题 |说明  |
|---------|---------|
| [如何：通过门户订阅事件](subscribe-through-portal.md) | 使用门户订阅 Azure 订阅的事件。 |
| [Azure CLI：订阅 Azure 订阅的事件](./scripts/event-grid-cli-azure-subscription.md) |用于在 Azure 订阅中创建事件网格订阅并将事件发送到 WebHook 的示例脚本。 |
| [PowerShell：订阅 Azure 订阅的事件](./scripts/event-grid-powershell-azure-subscription.md)| 用于在 Azure 订阅中创建事件网格订阅并将事件发送到 WebHook 的示例脚本。 |

## <a name="next-steps"></a>后续步骤

* 有关 Azure 事件网格的简介，请参阅[什么是事件网格？](overview.md)。
* 有关创建 Azure 事件网格订阅的详细信息，请参阅[事件网格订阅架构](subscription-creation-schema.md)。

