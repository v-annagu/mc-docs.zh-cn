---
title: 使用 .NET 创建存储访问策略
titleSuffix: Azure Storage
description: 了解如何使用 .NET 客户端库创建存储访问策略。
services: storage
author: WenJason
ms.service: storage
ms.topic: how-to
origin.date: 06/16/2019
ms.date: 07/20/2020
ms.author: v-jay
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: 2da4f946be84bbf2f59a53d37b22c11bfb3e4e55
ms.sourcegitcommit: 31da682a32dbb41c2da3afb80d39c69b9f9c1bc6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/16/2020
ms.locfileid: "86414727"
---
# <a name="create-a-stored-access-policy-with-net"></a>使用 .NET 创建存储访问策略

存储访问策略对服务器端的服务级别共享访问签名 (SAS) 提供另一级别的控制。 定义存储访问策略可以将共享访问签名分组到一起，并为通过策略绑定的共享访问签名提供其他限制。 可以使用存储访问策略更改 SAS 的开始时间、到期时间或权限，或者在颁发 SAS 后将其吊销。
  
以下 Azure 存储资源支持存储访问策略：  
  
- Blob 容器  
- 文件共享  
- 队列  
- 表  
  
> [!NOTE]
> 容器上的存储访问策略可以与共享访问签名相关联，后者授予对容器本身的权限，或对它包含的 Blob 的权限。 类似地，文件共享上的存储访问策略可以与共享访问签名相关联，后者授予对共享本身的权限，或对它包含的文件的权限。  
>
> 仅服务 SAS 支持存储访问策略。 帐户 SAS 不支持存储访问策略。  

## <a name="create-a-stored-access-policy"></a>创建存储访问策略

创建存储访问策略的基础 REST 操作是[设置容器 ACL](https://docs.microsoft.com/rest/api/storageservices/set-container-acl)。 你必须通过使用连接字符串中的帐户访问密钥，授权该操作通过共享密钥创建存储访问策略。 不支持使用 Azure AD 凭据授权“设置容器 ACL”操作。 有关详细信息，请参阅[调用 blob 和队列数据操作的权限](https://docs.microsoft.com/rest/api/storageservices/authorize-with-azure-active-directory#permissions-for-calling-blob-and-queue-data-operations)。

以下代码示例会在容器上创建存储访问策略。 可以使用访问策略指定对容器或其 Blob 上的服务 SAS 的约束。

# <a name="net-v12-sdk"></a>[.NET v12 SDK](#tab/dotnet)

若要使用适用于 Azure 存储的 .NET 客户端库版本 12 在容器上创建存储访问策略，请调用以下方法之一：

- [BlobContainerClient.SetAccessPolicy](https://docs.microsoft.com/dotnet/api/azure.storage.blobs.blobcontainerclient.setaccesspolicy)
- [BlobContainerClient.SetAccessPolicyAsync](https://docs.microsoft.com/dotnet/api/azure.storage.blobs.blobcontainerclient.setaccesspolicyasync)

以下示例创建一个有效期为一天并授予读/写权限的存储访问策略：

```csharp
async static Task CreateStoredAccessPolicyAsync(string containerName)
{
    string connectionString = "";

    // Use the connection string to authorize the operation to create the access policy.
    // Azure AD does not support the Set Container ACL operation that creates the policy.
    BlobContainerClient containerClient = new BlobContainerClient(connectionString, containerName);

    try
    {
        await containerClient.CreateIfNotExistsAsync();

        // Create one or more stored access policies.
        List<BlobSignedIdentifier> signedIdentifiers = new List<BlobSignedIdentifier>
        {
            new BlobSignedIdentifier
            {
                Id = "mysignedidentifier",
                AccessPolicy = new BlobAccessPolicy
                {
                    StartsOn = DateTimeOffset.UtcNow.AddHours(-1),
                    ExpiresOn = DateTimeOffset.UtcNow.AddDays(1),
                    Permissions = "rw"
                }
            }
        };
        // Set the container's access policy.
        await containerClient.SetAccessPolicyAsync(permissions: signedIdentifiers);
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine(e.ErrorCode);
        Console.WriteLine(e.Message);
    }
    finally
    {
        await containerClient.DeleteAsync();
    }
}
```

# <a name="net-v11-sdk"></a>[.NET v11 SDK](#tab/dotnet11)

若要使用适用于 Azure 存储的 .NET 客户端库版本 12 在容器上创建存储访问策略，请调用以下方法之一：

- [CloudBlobContainer.SetPermissions](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobcontainer.setpermissions)
- [CloudBlobContainer.SetPermissionsAsync](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobcontainer.setpermissionsasync)

以下示例创建一个有效期为一天并授予读取、写入和列表权限的存储访问策略：

```csharp
private static async Task CreateStoredAccessPolicyAsync(CloudBlobContainer container, string policyName)
{
    // Create a new stored access policy and define its constraints.
    // The access policy provides create, write, read, list, and delete permissions.
    SharedAccessBlobPolicy sharedPolicy = new SharedAccessBlobPolicy()
    {
        // When the start time for the SAS is omitted, the start time is assumed to be the time when Azure Storage receives the request.
        SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
        Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.List |
            SharedAccessBlobPermissions.Write
    };

    // Get the container's existing permissions.
    BlobContainerPermissions permissions = await container.GetPermissionsAsync();

    // Add the new policy to the container's permissions, and set the container's permissions.
    permissions.SharedAccessPolicies.Add(policyName, sharedPolicy);
    await container.SetPermissionsAsync(permissions);
}
```

---

## <a name="see-also"></a>另请参阅

- [使用共享访问签名 (SAS) 授予对 Azure 存储资源的有限访问权限](storage-sas-overview.md)
- [定义存储访问策略](https://docs.microsoft.com/rest/api/storageservices/define-stored-access-policy)
- [Configure Azure Storage connection strings](storage-configure-connection-string.md)（配置 Azure 存储连接字符串）
