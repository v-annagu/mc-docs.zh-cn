---
title: Azure 服务总线管理库 | Azure
description: 本文介绍如何使用 Azure 服务总线管理库动态预配服务总线命名空间和实体。
services: service-bus-messaging
documentationcenter: na
author: axisc
manager: timlt
editor: spelluru
ms.assetid: ''
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
origin.date: 01/24/2020
ms.date: 2/6/2020
ms.author: v-lingwu
ms.openlocfilehash: 639ef9f459a6713c55c9b7dd97510e2390a9d425
ms.sourcegitcommit: a04b0b1009b0c62f2deb7c7acee75a1304d98f87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/22/2020
ms.locfileid: "83796804"
---
# <a name="service-bus-management-libraries"></a>服务总线管理库

Azure 服务总线管理库可以动态预配服务总线命名空间和实体。 这样可以实现复杂的部署和消息方案，并能以编程方式确定要预配的实体。 这些库目前可用于 .NET。

## <a name="supported-functionality"></a>受支持的功能

* 创建、更新、删除命名空间
* 创建、更新、删除队列
* 创建、更新、删除主题
* 创建、更新、删除订阅

## <a name="prerequisites"></a>先决条件

若要开始使用服务总线管理库，必须使用 Azure Active Directory (Azure AD) 服务进行身份验证。 Azure AD 要求身份验证为服务主体，并且该主体提供对 Azure 资源的访问权限。 有关创建服务主体的信息，请参阅以下文章之一：  

* [使用 Azure 门户创建可访问资源的 Active Directory 应用程序和服务主体](/azure-resource-manager/resource-group-create-service-principal-portal)
* [使用 Azure PowerShell 创建服务主体来访问资源](/azure-resource-manager/resource-group-authenticate-service-principal)
* [使用 Azure CLI 创建服务主体来访问资源](/azure-resource-manager/resource-group-authenticate-service-principal-cli)

这些教程提供 `AppId`（客户端 ID）、`TenantId` 和 `ClientSecret`（身份验证密钥），这些都将用于管理库进行的身份验证。 对于用于运行的资源组，需要至少具有 [Azure 服务总线数据所有者](/role-based-access-control/built-in-roles.md#azure-service-bus-data-owner)或[参与者](/role-based-access-control/built-in-roles.md#contributor)权限 。

## <a name="programming-pattern"></a>编程模式

所有服务总线资源的操纵模式都遵循常用协议：

1. 使用 **Microsoft.IdentityModel.Clients.ActiveDirectory** 库从 Azure AD 获取令牌：
    ```csharp
    var context = new AuthenticationContext($"https://login.chinacloudapi.cn/{tenantId}");

   var result = await context.AcquireTokenAsync("https://management.core.chinacloudapi.cn/", new ClientCredential(clientId, clientSecret));
   ```
2. 创建 `ServiceBusManagementClient` 对象：

   ```csharp
   var creds = new TokenCredentials(token);
   var sbClient = new ServiceBusManagementClient(creds)
   {
       SubscriptionId = SettingsCache["SubscriptionId"]
   };
   ```
3. 将 `CreateOrUpdate` 参数设置为指定值：

   ```csharp
   var queueParams = new QueueCreateOrUpdateParameters()
   {
       Location = SettingsCache["DataCenterLocation"],
       EnablePartitioning = true
   };
   ```
4. 执行调用：

   ```csharp
   await sbClient.Queues.CreateOrUpdateAsync(resourceGroupName, namespaceName, QueueName, queueParams);
   ```

## <a name="complete-code-to-create-a-queue"></a>完成创建队列的代码
下面是创建服务总线队列的完整代码： 

```csharp
using System;
using System.Threading.Tasks;

using Microsoft.Azure.Management.ServiceBus;
using Microsoft.Azure.Management.ServiceBus.Models;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.Rest;

namespace SBusADApp
{
    class Program
    {
        static void Main(string[] args)
        {
            CreateQueue().GetAwaiter().GetResult();
        }

        private static async Task CreateQueue()
        {
            try
            {
                var subscriptionID = "<SUBSCRIPTION ID>";
                var resourceGroupName = "<RESOURCE GROUP NAME>";
                var namespaceName = "<SERVICE BUS NAMESPACE NAME>";
                var queueName = "<NAME OF QUEUE YOU WANT TO CREATE>";

                var token = await GetToken();

                var creds = new TokenCredentials(token);
                var sbClient = new ServiceBusManagementClient(creds)
                {
                    SubscriptionId = subscriptionID,
                };

                var queueParams = new SBQueue();

                Console.WriteLine("Creating queue...");
                await sbClient.Queues.CreateOrUpdateAsync(resourceGroupName, namespaceName, queueName, queueParams);
                Console.WriteLine("Created queue successfully.");
            }
            catch (Exception e)
            {
                Console.WriteLine("Could not create a queue...");
                Console.WriteLine(e.Message);
                throw e;
            }
        }

        private static async Task<string> GetToken()
        {
            try
            {
                var tenantId = "<AZURE AD TENANT ID>";
                var clientId = "<APPLICATION/CLIENT ID>";
                var clientSecret = "<CLIENT SECRET>";

                var context = new AuthenticationContext($"https://login.chinacloudapi.cn/{tenantId}");

                var result = await context.AcquireTokenAsync(
                    "https://management.chinacloudapi.cn/",
                    new ClientCredential(clientId, clientSecret)
                );

                // If the token isn't a valid string, throw an error.
                if (string.IsNullOrEmpty(result.AccessToken))
                {
                    throw new Exception("Token result is empty!");
                }

                return result.AccessToken;
            }
            catch (Exception e)
            {
                Console.WriteLine("Could not get a token...");
                Console.WriteLine(e.Message);
                throw e;
            }
        }

    }
}
```

> [!IMPORTANT]
> 有关完整示例，请参阅 [GitHub 上的 .NET 管理示例](https://github.com/Azure-Samples/service-bus-dotnet-management/)。 

## <a name="next-steps"></a>后续步骤
[Microsoft.Azure.Management.ServiceBus API 参考](https://docs.azure.cn/dotnet/api/Microsoft.Azure.Management.ServiceBus?view=azure-dotnet)