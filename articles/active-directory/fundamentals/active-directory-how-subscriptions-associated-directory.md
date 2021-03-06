---
title: 将现有 Azure 订阅添加到租户 - Azure AD
description: 有关将现有 Azure 订阅添加到 Azure Active Directory 租户的说明。
services: active-directory
author: msaburnley
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 07/02/2020
ms.author: v-junlch
ms.reviewer: jeffsta
ms.custom: it-pro, seodec18, contperfq4
ms.collection: M365-identity-device-management
ms.openlocfilehash: 06c3aa782729154f55e641d29d19f21d46751a01
ms.sourcegitcommit: 1008ad28745709e8d666f07a90e02a79dbbe2be5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/03/2020
ms.locfileid: "85945000"
---
# <a name="associate-or-add-an-azure-subscription-to-your-azure-active-directory-tenant"></a>将 Azure 订阅关联或添加到 Azure Active Directory 租户

Azure 订阅与 Azure Active Directory (Azure AD) 之间存在信任关系。 订阅信任 Azure AD 对用户、服务和设备执行身份验证。

多个订阅可以信任同一个 Azure AD 目录。 每个订阅只能信任一个目录。

如果订阅过期，则将失去与该订阅关联的所有其他资源的访问权限。 但是，Azure AD 目录保留在 Azure 中。 可以使用不同的 Azure 订阅来关联和管理该目录。

所有用户都有一个用于身份验证的“主”目录。 用户还可以充当其他目录中的来宾。 可在 Azure AD 中查看每位用户的主目录和来宾目录。

> [!Important]
> 当你将订阅与其他目录关联时，具有使用[基于角色的访问控制 (RBAC)](../../role-based-access-control/role-assignments-portal.md) 分配的角色的用户将失去其访问权限。 经典订阅管理员（包括服务管理员和共同管理员）也将失去其访问权限。
>
> 当订阅与不同的目录关联时，还会从该订阅中删除策略分配。
>
> 如果将 Azure Kubernetes 服务 (AKS) 群集移到其他订阅，或者将拥有该群集的订阅移到新租户，该群集将会由于失去角色分配和服务主体权限而丢失功能。 有关 AKS 的详细信息，请参阅 [Azure Kubernetes 服务 (AKS)](/aks/)。


## <a name="before-you-begin"></a>准备阶段

在关联或添加订阅之前，请执行以下任务：

- 查看下述在关联或添加订阅后会发生的更改的列表，以及你可能受到的具体影响：

  - 已使用 RBAC 为其分配了角色的用户将失去其访问权限
  - 服务管理员和共同管理员将失去其访问权限
  - 如果你有任何密钥保管库，这些密钥保管库将无法访问，而且你必须在关联后对其进行修复
  - 如果对资源（例如虚拟机或逻辑应用）使用任何托管标识，则必须在关联后重新启用或重新创建这些标识
  - 如果拥有已注册的 Azure Stack，则将必须在关联后重新注册它

- 使用符合以下条件的帐户登录：

  - 具有该订阅的[所有者](../../role-based-access-control/built-in-roles.md#owner)角色分配。 有关如何分配“所有者”角色的详细信息，请参阅[使用 RBAC 和 Azure 门户管理对 Azure 资源的访问权限](../../role-based-access-control/role-assignments-portal.md)。
  - 在当前目录和新目录中存在。 当前目录已与订阅相关联。 要将新目录与订阅相关联。 若要详细了解如何获取其他目录的访问权限，请参阅[在 Azure 门户中添加 Azure Active Directory B2B 协作用户](../b2b/add-users-administrator.md)。

- 请确保未使用 Azure 云服务提供商 (CSP) 订阅（MS-AZR-0145P、MS-AZR-0146P、MS-AZR-159P）、Microsoft 内部订阅 (MS-AZR-0015P) 或 Microsoft Imagine 订阅 (MS-AZR-0144P)。

## <a name="associate-a-subscription-to-a-directory"></a>将订阅关联到目录<a name="to-associate-an-existing-subscription-to-your-azure-ad-directory"></a>

将现有订阅关联到 Azure AD 目录

1. 登录，然后在[门户](https://portal.azure.cn/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview)中，选择“Azure Active Directory”。

2. 选择“切换目录”。

    ![订阅页面，其中突出显示了“更改目录”选项](./media/active-directory-how-subscriptions-associated-directory/change-directory-button.png)

3. 选择“所有目录”，然后单击要切换的目录。

    ![“更改目录”页，显示要更改到的目录](./media/active-directory-how-subscriptions-associated-directory/edit-directory-ui.png)
  
更改订阅目录是服务级操作，不会影响订阅的账单所有权。 帐户管理员仍可从[帐户中心](https://account.windowsazure.cn/subscriptions)更改服务管理员。 若要删除原始目录，必须将订阅的账单所有权转让给新的帐户管理员。若要详细了解如何转让账单所有权，请参阅[将 Azure 订阅所有权转让给其他帐户](/billing/billing-subscription-transfer)。

## <a name="post-association-steps"></a>关联后的步骤

将订阅关联到不同的目录后，可能需要执行以下任务来恢复操作：

- 如果有任何密钥保管库，则必须更改该密钥保管库租户 ID。 有关详细信息，请参阅[在订阅移动后更改密钥保管库租户 ID](../../key-vault/general/subscription-move-fix.md)。

- 如果对资源使用了系统分配的托管标识，则必须重新启用这些标识。 如果使用了用户分配的托管标识，则必须重新创建这些标识。 重新启用或重新创建托管标识后，必须重新建立分配给这些标识的权限。 有关详细信息，请参阅[什么是 Azure 资源的托管标识？](../managed-identities-azure-resources/overview.md)。

- 如果已使用此订阅注册了 Azure Stack，则必须重新注册。 有关详细信息，请参阅[将 Azure Stack 注册到 Azure](/azure-stack/operator/azure-stack-registration)。

## <a name="next-steps"></a>后续步骤

- 若要创建新的 Azure AD 租户，请参阅[快速入门：在 Azure Active Directory 中创建新租户](active-directory-access-create-new-tenant.md)。

- 若要详细了解 Azure 如何控制资源访问，请参阅[经典订阅管理员角色、Azure RBAC 角色和 Azure AD 管理员角色](../../role-based-access-control/rbac-and-directory-admin-roles.md)。

- 若要详细了解如何在 Azure AD 中分配角色，请参阅[为使用 Azure Active Directory 的用户分配管理员和非管理员角色](active-directory-users-assign-role-azure-portal.md)。

