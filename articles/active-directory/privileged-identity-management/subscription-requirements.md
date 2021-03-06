---
title: 使用 Privileged Identity Management 所要满足的许可证要求 - Azure Active Directory | Microsoft Docs
description: 介绍使用 Azure AD Privileged Identity Management (PIM) 所要满足的许可要求。
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: markwahl-msft
ms.assetid: 34367721-8b42-4fab-a443-a2e55cdbf33d
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: pim
ms.date: 02/07/2020
ms.author: v-junlch
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2d8b1aa3a11be249fe5f414b91d6c79a0200da02
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "77067723"
---
# <a name="license-requirements-to-use-privileged-identity-management"></a>使用 Privileged Identity Management 所要满足的许可证要求

若要使用 Azure Active Directory (Azure AD) Privileged Identity Management (PIM)，目录必须具有有效的许可证。 此外，必须将许可证分配给管理员和相关用户。 本文介绍使用 Privileged Identity Management 所要满足的许可证要求。

## <a name="valid-licenses"></a>有效的许可证

[!INCLUDE [Azure AD Premium P2 license](../../../includes/active-directory-p2-license.md)]

## <a name="how-many-licenses-must-you-have"></a>必须拥有多少个许可证？

确保你的目录具有的 Azure AD Premium P2 许可证至少与将执行以下任务的员工一样多：

- 已分配到使用 PIM 管理的 Azure AD 角色的合格用户
- 能够在 PIM 中批准或拒绝请求的用户
- 已通过实时或直接（基于时间）分配方法分配到 Azure 资源角色的用户  

对于以下任务，不需要  使用 Azure AD Premium P2 许可证：

- 设置 PIM、配置策略和接收警报的、具有全局管理员或特权角色管理员角色的用户不需要许可证。

有关许可证的详细信息，请参阅[使用 Azure Active Directory 门户分配或删除许可证](../fundamentals/license-users-groups.md)。

## <a name="example-license-scenarios"></a>示例许可证方案

下面是一些示例许可证方案，可帮助你确定必须拥有的许可证数量。

| 场景 | 计算 | 许可证数量 |
| --- | --- | --- |
| Woodgrove Bank 有用于不同部门的 10 个管理员和用于配置和管理 PIM 的 2 个全局管理员。 他们使五个管理员符合条件。 | 五个许可证用于符合条件的管理员 | 5 |
| Graphic Design Institute 有 25 个管理员，其中 14 个是通过 PIM 管理的。 角色激活需要批准，并且组织中有三个不同的用户可以批准激活。 | 14 个许可证用于符合条件的角色 + 三个审批者 | 17 |
| Contoso 有 50 个管理员，其中 42 个是通过 PIM 管理的。 角色激活需要批准，并且组织中有五个不同的用户可以批准激活。 Contoso 还每月对分配给管理员角色的用户进行一次审阅，审阅者是用户的经理，其中有六个不在 PIM 管理的管理员角色中。 | 42 个许可证用于符合条件的角色 + 五个审批者 + 六个审阅者 | 53 |

## <a name="what-happens-when-a-license-expires"></a>许可证过期时会发生什么情况？

如果 Azure AD Premium P2、EMS E5 或试用许可证过期，则不再可以在目录中使用 Privileged Identity Management 功能：

- 对 Azure AD 角色的永久角色分配会受到影响。
- 用户不再可以使用 Azure 门户中的 Privileged Identity Management 服务以及 Privileged Identity Management 的图形 API cmdlet 和 PowerShell 接口来激活特权角色、管理特权访问权限。
- 将删除 Azure AD 角色的符合条件的角色分配，因为用户不再能够激活特权角色。
- 角色分配更改时，Privileged Identity Management 将不再发送电子邮件。

## <a name="next-steps"></a>后续步骤

- [部署 Privileged Identity Management](pim-deployment-plan.md)
- [开始使用 Privileged Identity Management](pim-getting-started.md)
- [无法在 Privileged Identity Management 中管理的角色](pim-roles.md)

<!-- Update_Description: wording update -->