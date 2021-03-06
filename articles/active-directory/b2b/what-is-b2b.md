---
title: 什么是 Azure Active Directory 中的 B2B 协作？
description: Azure Active Directory B2B 协作支持来宾用户访问权限，以便安全地与外部合作伙伴共享资源和协作。
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: overview
ms.date: 07/01/2020
ms.author: v-junlch
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7140d7d57d98b9c63bc7240492464962733a8252
ms.sourcegitcommit: 1008ad28745709e8d666f07a90e02a79dbbe2be5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/03/2020
ms.locfileid: "85945073"
---
# <a name="what-is-guest-user-access-in-azure-active-directory-b2b"></a>什么是 Azure Active Directory B2B 中的来宾用户访问权限？

使用 Azure Active Directory (Azure AD) 企业到企业 (B2B) 协作可以安全地将公司的应用程序和服务与来自任何其他组织的来宾用户共享，同时保持对自己公司数据的控制。 与外部合作伙伴安全放心地合作，不论其规模是大是小，甚至就算他们没有 Azure AD 或 IT 部门也无妨。 合作伙伴通过一个简单的邀请和兑换过程即可使用自己的凭据来访问公司资源。 开发人员可以使用 Azure AD 企业到企业 API 自定义邀请过程或编写应用程序。

请观看视频，了解如何邀请来宾用户使用他们自己的标识登录公司的应用和服务以安全地与之协作。

## <a name="collaborate-with-any-partner-using-their-identities"></a>与使用自己标识的任何合作伙伴协作

借助 Azure AD B2B，合作伙伴可使用自己的标识管理解决方案，因此组织省去了外部管理开销。

- 合作伙伴使用自己的标识和凭据；
- 不需要管理外部帐户或密码。
- 不需要同步帐户或管理帐户生命周期。  

![显示“添加成员”页的屏幕截图](./media/what-is-b2b/add-member.png)

## <a name="invite-guest-users-with-a-simple-invitation-and-redemption-process"></a>通过一个简单的邀请和兑换过程邀请来宾用户

来宾用户可使用自己的工作或学校标识登录应用和服务。 如果来宾用户没有 Azure AD 帐户，当他们兑换邀请时，系统会为他们创建一个帐户。 

- 邀请使用自选电子邮件标识的来宾用户。
- 发送应用的直接链接，或发送邀请至来宾用户自己的访问面板。
- 来宾用户遵循一些简单的兑换步骤登录。

## <a name="use-policies-to-securely-share-your-apps-and-services"></a>使用策略安全地共享你的应用和服务

可以使用授权策略保护企业内容。 可在以下级别强制执行多重身份验证等条件访问策略：

- 租户级别。
- 应用程序级别。
- 针对特定来宾用户，保护企业应用和数据。

![显示“条件访问”选项的屏幕截图](./media/what-is-b2b/tutorial-mfa-policy-2.png)


## <a name="easily-add-guest-users-in-the-azure-ad-portal"></a>在 Azure AD 门户中轻松添加来宾用户

管理员可以在 Azure 门户中轻松地向组织添加来宾用户。

- 在 Azure AD 中创建新的来宾用户，方法类似于添加新用户。
- 来宾用户会立即收到允许他们登录访问面板的可自定义邀请。
- 目录中的来宾用户会被分配到应用或组。  

![显示“新建来宾用户邀请”入口页的屏幕截图](./media/what-is-b2b/add-a-b2b-user-to-azure-portal.png)

## <a name="customize-the-onboarding-experience-for-b2b-guest-users"></a>自定义 B2B 来宾用户的载入体验

使用按组织需求自定义的方法引入外部合作伙伴。

- 使用 [B2B 协作邀请 API](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation) 自定义载入体验。

## <a name="next-steps"></a>后续步骤

- [Azure AD B2B 协作的许可指南](licensing-guidance.md)
- [在门户中添加 B2B 协作来宾用户](add-users-administrator.md)
- [了解邀请兑换过程](redemption-experience.md)
- 此外，还可与往常一样通过 [Microsoft Tech Community](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-B2B/bd-p/AzureAD_B2b)（Microsoft 技术社区）与产品团队联系，获取任何反馈、讨论和建议。

