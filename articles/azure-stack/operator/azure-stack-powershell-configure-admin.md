---
title: 使用 PowerShell 连接到 Azure Stack Hub
description: 了解如何使用 PowerShell 连接到 Azure Stack Hub。
author: WenJason
ms.topic: article
origin.date: 5/27/2020
ms.date: 06/22/2020
ms.author: v-jay
ms.reviewer: thoroet
ms.lastreviewed: 09/19/2019
ms.openlocfilehash: 238d3d9a5b3bc62242f1e7bb1bf65c9c7811b7c4
ms.sourcegitcommit: d86e169edf5affd28a1c1a4476d72b01a7fb421d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2020
ms.locfileid: "85096284"
---
# <a name="connect-to-azure-stack-hub-with-powershell"></a>使用 PowerShell 连接到 Azure Stack Hub

可以将 Azure Stack Hub 配置为使用 PowerShell 来管理资源，例如创建套餐、计划、配额以及警报。 本主题有助于配置操作员环境

## <a name="prerequisites"></a>先决条件

如果[已通过 VPN 连接到 ASDK](../asdk/asdk-connect.md#connect-with-vpn)，请通过 [Azure Stack 开发工具包 (ASDK)](../asdk/asdk-connect.md#connect-with-rdp) 或基于 Windows 的外部客户端运行以下先决条件操作。

- 安装 [Azure Stack Hub 兼容的 Azure PowerShell 模块](azure-stack-powershell-install.md)。  
- 下载[使用 Azure Stack Hub 所需的工具](azure-stack-powershell-download.md)。  

## <a name="connect-with-azure-ad"></a>与 Azure AD 连接

若要使用 PowerShell 配置 Azure Stack Hub 操作员环境，请运行以下脚本之一。 将 Azure Active Directory (Azure AD) tenantName 和 Azure 资源管理器终结点值替换为你自己的环境配置。

[!include[Remove Account](../includes/remove-account.md)]

```powershell  
    # Register an Azure Resource Manager environment that targets your Azure Stack Hub instance. Get your Azure Resource Manager endpoint value from your service provider.
    Add-AzureRMEnvironment -Name "AzureStackAdmin" -ArmEndpoint "https://adminmanagement.local.azurestack.external" `
      -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
      -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external

    # Set your tenant name.
    $AuthEndpoint = (Get-AzureRmEnvironment -Name "AzureStackAdmin").ActiveDirectoryAuthority.TrimEnd('/')
    $AADTenantName = "<myDirectoryTenantName>.partner.onmschina.cn"
    $TenantId = (invoke-restmethod "$($AuthEndpoint)/$($AADTenantName)/.well-known/openid-configuration").issuer.TrimEnd('/').Split('/')[-1]

    # After signing in to your environment, Azure Stack Hub cmdlets
    # can be easily targeted at your Azure Stack Hub instance.
    Add-AzureRmAccount -EnvironmentName "AzureStackAdmin" -TenantId $TenantId
```

## <a name="connect-with-ad-fs"></a>与 AD FS 连接

使用 PowerShell 通过 Active Directory 联合身份验证服务 (Azure AD FS) 连接到 Azure Stack Hub 操作员环境。 对于 ASDK，此 Azure 资源管理器终结点设置为 `https://adminmanagement.local.azurestack.external`。 若要获取 Azure Stack Hub 集成系统的 Azure 资源管理器终结点，请与服务提供商联系。

  ```powershell  
  # Register an Azure Resource Manager environment that targets your Azure Stack Hub instance. Get your Azure Resource Manager endpoint value from your service provider.
    Add-AzureRMEnvironment -Name "AzureStackAdmin" -ArmEndpoint "https://adminmanagement.local.azurestack.external" `
      -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
      -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external

  # Sign in to your environment.
  Login-AzureRmAccount -EnvironmentName "AzureStackAdmin"
  ```

> [!Note]  
> AD FS 仅支持通过用户标识进行交互式身份验证。 如果需要凭据对象，则必须使用服务主体 (SPN)。 若要详细了解如何在设置服务主体时将 Azure Stack Hub 和 AD FS 作为标识管理服务，请参阅[管理 AD FS 应用标识](azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity)。

## <a name="test-the-connectivity"></a>测试连接

完成所有设置后，请使用 PowerShell 在 Azure Stack Hub 中创建资源。 例如，可以为应用创建资源组并添加虚拟机。 使用以下命令创建名为“MyResourceGroup”的资源组。

```powershell  
New-AzureRmResourceGroup -Name "MyResourceGroup" -Location "Local"
```

## <a name="next-steps"></a>后续步骤

- [为 Azure Stack Hub 开发模板](../user/azure-stack-develop-templates.md)。
- [使用 PowerShell 部署模板](../user/azure-stack-deploy-template-powershell.md)。
  - [Azure Stack Hub 模块参考](https://docs.microsoft.com/powershell/azure/azure-stack/overview)。
