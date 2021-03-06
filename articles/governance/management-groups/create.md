---
title: 创建管理组以组织资源 - Azure 治理
description: 了解如何使用门户、Azure PowerShell 和 Azure CLI 创建 Azure 管理组以管理多个资源。
author: rthorn17
manager: rithorn
ms.service: azure-resource-manager
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
origin.date: 04/05/2019
ms.date: 12/02/2019
ms.author: v-tawe
ms.topic: conceptual
ms.openlocfilehash: 59035317a05b18bfb529598f8b47f569e84a6525
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "74658110"
---
<!--Verify successfully-->
# <a name="create-management-groups-for-resource-organization-and-management"></a>创建用来组织和管理资源的管理组

管理组是一些容器，可以帮助跨多个订阅管理访问权限、策略和符合性。 可以创建这些容器来构建可以与 [Azure Policy](../policy/overview.md) 和 [Azure 基于角色的访问控制](../../role-based-access-control/overview.md)配合使用的有效且高效的层次结构。 若要详细了解管理组，请参阅[使用 Azure 管理组整理资源](overview.md)。

在目录中创建的第一个管理组可能需要最多 15 分钟才能完成。 一些进程会首次运行以在 Azure 中为目录设置管理组服务。 在进程完成后将显示通知。

## <a name="create-a-management-group"></a>创建管理组

可以使用门户、PowerShell 或 Azure CLI 创建管理组。 目前无法使用资源管理器模板创建管理组。

### <a name="create-in-portal"></a>在门户中创建

1. 登录到 [Azure 门户](http://portal.azure.cn)。

1. 选择“所有服务” > “管理 + 治理”。

1. 选择“成本管理 + 计费” 

1. 在“成本管理 + 计费 - 管理组”页上，选择“管理组” 

1. 选择“+ 添加管理组”。 

   ![管理组操作页](./media/main.png)

1. 填写管理组 ID 字段。

   -  “管理组 ID”是用来在此管理组上提交命令的目录唯一标识符。 此标识符一旦创建便无法再编辑，因为它用来在整个 Azure 系统中标识这个组。 [根管理组](overview.md#root-management-group-for-each-directory)是自动创建的，其 ID 为 Azure Active Directory ID。 对于所有其他管理组，请分配唯一的 ID。
   - 显示名称字段是在 Azure 门户中显示的名称。 创建管理组时，单独的显示名称是一个可选字段，并且可以随时更改。  

   ![用于创建新管理组的“选项”窗格](./media/create_context_menu.png)  

1. 选择“保存”。 

### <a name="create-in-powershell"></a>在 PowerShell 中创建

在 PowerShell 中，使用 [New-AzManagementGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azmanagementgroup) cmdlet 创建新的管理组。

```powershell
New-AzManagementGroup -GroupName 'Contoso'
```

**GroupName** 是要创建的唯一标识符。 此 ID 由其他命令用来引用此组，并且以后无法更改。

如果希望管理组在 Azure 门户中显示一个不同的名称，请添加 **DisplayName** 参数。 例如，若要创建 GroupName 为 Contoso 且显示名称为“Contoso Group”的管理组，请使用以下 cmdlet：

```powershell
New-AzManagementGroup -GroupName 'Contoso' -DisplayName 'Contoso Group'
```

在上述示例中，新的管理组是在根管理组下创建的。 若要指定一个不同的管理组作为父级，请使用 **ParentId** 参数。

```powershell
$parentGroup = Get-AzManagementGroup -GroupName Contoso
New-AzManagementGroup -GroupName 'ContosoSubGroup' -ParentId $parentGroup.id
```

### <a name="create-in-azure-cli"></a>在 Azure CLI 中创建

在 Azure CLI 中，使用 [az account management-group create](/cli/account/management-group?view=azure-cli-latest#az-account-management-group-create) 命令创建新的管理组。

```azurecli
az account management-group create --name Contoso
```

**name** 是要创建的唯一标识符。 此 ID 由其他命令用来引用此组，并且以后无法更改。

如果希望管理组在 Azure 门户中显示一个不同的名称，请添加 **display-name** 参数。 例如，若要创建 GroupName 为 Contoso 且显示名称为“Contoso Group”的管理组，请使用以下命令：

```azurecli
az account management-group create --name Contoso --display-name 'Contoso Group'
```

在上述示例中，新的管理组是在根管理组下创建的。 若要指定一个不同的管理组作为父级，请使用 **parent** 参数并提供父组的名称。

```azurecli
az account management-group create --name ContosoSubGroup --parent Contoso
```

## <a name="next-steps"></a>后续步骤

若要了解有关管理组的详细信息，请参阅：

- [创建管理组来组织 Azure 资源](create.md)
- [如何更改、删除或管理管理组](manage.md)
- [在 Azure PowerShell 资源模块中查看管理组](https://docs.microsoft.com/powershell/module/azurerm.resources/?view=azurermps-6.13.0&viewFallbackFrom=azurermps-6.12.0#resources)
- [在 REST API 中查看管理组](https://docs.microsoft.com/rest/api/resources/managementgroups)
- [在 Azure CLI 中查看管理组](https://docs.azure.cn/cli/account/management-group?view=azure-cli-latest)
