---
title: 部署历史记录删除
description: 介绍 Azure 资源管理器如何从部署历史记录中自动删除部署。 当历史记录即将超过限制（800 条）时，将删除部署。
ms.topic: conceptual
origin.date: 07/06/2020
ms.date: 07/13/2020
ms.testscope: yes
ms.testdate: 07/13/2020
ms.author: v-yeche
ms.openlocfilehash: fc345e21d4f9b70aa863ba4e0686e37434e74756
ms.sourcegitcommit: 2bd0be625b21c1422c65f20658fe9f9277f4fd7c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/17/2020
ms.locfileid: "86441105"
---
<!--Verified successfully on 2020/07/13 by harris-->
# <a name="automatic-deletions-from-deployment-history"></a>从部署历史记录自动删除

每次部署模板时，有关部署的信息都会写入到部署历史记录中。 每个资源组在其部署历史记录中最多只能有 800 个部署。

当你接近限制时，Azure 资源管理器会很快开始自动从历史记录中删除部署。 自动删除是对过去的行为的更改。 以前，必须从部署历史记录中手动删除部署，以避免出现错误。

> [!NOTE]
> 从历史记录中删除部署不会影响已部署的任何资源。
>
> 如果对资源组使用 [CanNotDelete 锁](../management/lock-resources.md)，则无法删除该资源组的部署。 必须删除锁才能利用部署历史记录中的自动删除功能。

## <a name="when-deployments-are-deleted"></a>删除部署时

达到 790 个部署时，将从部署历史记录中删除部署。 Azure 资源管理器会删除一小部分最早的部署，清理空间以记录将来的部署。 大部分历史记录都保持不变。 始终最先删除最早的部署。

:::image type="content" border="false" source="./media/deployment-history-deletions/deployment-history.svg" alt-text="从部署历史记录中删除内容":::

除了部署之外，还可以在运行 [what-if 操作](template-deploy-what-if.md)或验证部署时触发删除操作。

为部署指定与历史记录中的部署相同的名称时，将重置其在历史记录中的位置。 部署会移至历史记录中的最新位置。 在发生错误后[回退到该部署](rollback-on-error.md)时，也会重置部署的位置。

> [!NOTE]
> 如果资源组已经达到 800 条记录的限制，则下一个部署将失败并出现错误。 会立即启动自动删除过程。 等待片刻后，可以再次尝试进行部署。

## <a name="opt-out-of-automatic-deletions"></a>选择退出自动删除

可以选择退出从历史记录中自动删除条目的功能。 仅当要自行管理部署历史记录时才使用此选项。 仍强制实施在历史记录中保留 800 个部署的限制。 如果超过 800 个部署，你将收到错误，并且部署将失败。

要禁用自动删除，请注册 `Microsoft.Resources/DisableDeploymentGrooming` 功能标志。 注册功能标志时，即为整个 Azure 订阅选择退出自动删除功能。 不能仅为特定资源组选择退出。

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

对于 PowerShell，请使用 [Register-AzProviderFeature](https://docs.microsoft.com/powershell/module/az.resources/Register-AzProviderFeature)。

```powershell
Register-AzProviderFeature -ProviderNamespace Microsoft.Resources -FeatureName DisableDeploymentGrooming
```

要查看订阅的当前状态，请使用：

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Resources -FeatureName DisableDeploymentGrooming
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

对于 Azure CLI，请使用 [az feature register](https://docs.azure.cn/cli/feature?view=azure-cli-latest#az-feature-register)。

```azurecli
az feature register --namespace Microsoft.Resources --name DisableDeploymentGrooming
```

要查看订阅的当前状态，请使用：

```azurecli
az feature show --namespace Microsoft.Resources --name DisableDeploymentGrooming
```

# <a name="rest"></a>[REST](#tab/rest)

对于 REST API，请使用[功能 - 注册](https://docs.microsoft.com/rest/api/resources/features/register)。

```rest
POST https://management.chinacloudapi.cn/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Resources/features/DisableDeploymentGrooming/register?api-version=2015-12-01
```

要查看订阅的当前状态，请使用：

```rest
GET https://management.chinacloudapi.cn/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Resources/features/DisableDeploymentGrooming/register?api-version=2015-12-01
```

---

## <a name="next-steps"></a>后续步骤

* 要了解如何查看部署历史记录，请参阅[使用 Azure 资源管理器查看部署历史记录](deployment-history.md)。

<!-- Update_Description: update meta properties, wording update, update link -->