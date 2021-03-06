---
title: Azure AD Connect：用户登录 | Microsoft Docs
description: Azure AD Connect 用户登录的自定义设置。
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.assetid: 547b118e-7282-4c7f-be87-c035561001df
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/23/2020
ms.subservice: hybrid
ms.author: v-junlch
ms.collection: M365-identity-device-management
ms.openlocfilehash: 23e9545bd3f5f45b64207b42b92f934cfbd9c2ab
ms.sourcegitcommit: a4a2521da9b29714aa6b511fc6ba48279b5777c8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/24/2020
ms.locfileid: "82126553"
---
# <a name="azure-ad-connect-user-sign-in-options"></a>Azure AD Connect 用户登录选项
Azure Active Directory (Azure AD) Connect 可让用户使用同一组密码登录云和本地资源。 本文介绍每个标识模型的重要概念，以帮助你选择登录到 Azure AD 时想要使用的标识。

如果已熟悉了 Azure AD 标识模型，并且想详细了解某个特定的方法，则请参阅相应的链接：

* [密码哈希同步](#password-hash-synchronization)
* [使用 PingFederate 进行联合身份验证](#federation-with-pingfederate)

> [!NOTE] 
> 请务必记住，通过为 Azure AD 配置联合，可以在 Azure AD 租户与联合域之间建立信任。 有了此可信的联合域，用户将能够在租户中访问 Azure AD 云资源。  
>

## <a name="choosing-the-user-sign-in-method-for-your-organization"></a>为组织选择用户登录方法
实施 Azure AD Connect 的第一项决策是选择用户登录时要使用的身份验证方法。 必须确保选择符合组织安全要求和高级要求的适当方法。 身份验证至关重要，因为它用于验证访问云中应用和数据的用户的身份。 若要选择适当的身份验证方法，需要考虑时间、现有基础结构、复杂性和实现所选内容的成本。 这些因素对每个组织都不同，并可能随时间变化。

Azure AD 支持以下身份验证方法： 

* **云身份验证** - 如果选择此身份验证方法，Azure AD 将处理用户登录时的身份验证过程。 使用云身份验证时，可以选择： 
   * **密码哈希同步 (PHS)** - 通过密码哈希同步，用户可以使用与其在本地使用的相同用户名和密码，而无需部署除 Azure AD Connect 以外的其他任何基础结构。 

* **联合身份验证** - 如果选择此身份验证方法，Azure AD 会将身份验证过程移交给单独的受信任身份验证系统（例如 AD FS 或第三方联合身份验证服务）来验证用户的登录。 

由于大多数组织只想让用户登录 Office 365 和其他基于 Azure AD 的资源，因此，我们建议使用默认的密码哈希同步选项。


### <a name="password-hash-synchronization"></a>密码哈希同步
凭借密码哈希同步，可将用户密码的哈希从本地 Active Directory 同步到 Azure AD。 当在本地更改或重置密码时，新密码哈希将立即同步到 Azure AD，以便用户始终可用相同密码访问云资源与本地资源。 密码绝不会被发送到 Azure AD，也不会以明文的形式存储在 Azure AD 中。
![密码哈希同步](./media/plan-connect-user-signin/passwordhash.png)

有关详细信息，请参阅[密码哈希同步](how-to-connect-password-hash-synchronization.md)一文。

### <a name="federation-that-uses-a-new-or-existing-farm-with-ad-fs-in-windows-server-2012-r2"></a>在 Windows Server 2012 R2 中使用新的或现有 AD FS 场进行联合身份验证
凭借联合登录，用户可以使用其本地密码登录到 Azure 基于 AD 的服务。 当用户处于企业网络上时，他们甚至无需输入其密码。 使用 AD FS 的联合身份验证选项，可在 Windows Server 2012 R2 中部署新的或现有的 AD FS 场。 如果选择指定现有场，Azure AD Connect 将在场与 Azure AD 之间配置信任，使你的用户能够登录。

![在 Windows Server 2012 R2 中使用 AD FS 进行联合身份验证](./media/plan-connect-user-signin/federatedsignin.png)

#### <a name="deploy-federation-with-ad-fs-in-windows-server-2012-r2"></a>在 Windows Server 2012 R2 中部署使用 AD FS 的联合身份验证

如果要部署新场，则需要：

* 用于联合服务器的 Windows Server 2012 R2 服务器。
* 用于 Web 应用程序代理的 Windows Server 2012 R2 服务器。
* 一个 .pfx 文件，其中包含一个所需联合服务名称的 TLS/SSL 证书。 例如：fs.contoso.com。

如果要部署新场或使用现有场，则需要：

* 联合服务器上的本地管理员凭据。
* 要将 Web 应用程序代理角色部署在上面的任何工作组服务器（未加入域）上的本地管理员凭据。
* 运行向导的计算机能够通过 Windows 远程管理连接到要安装 AD FS 或 Web 应用程序代理的任何其他计算机。


### <a name="federation-with-pingfederate"></a>使用 PingFederate 进行联合身份验证
凭借联合登录，用户可以使用其本地密码登录到 Azure 基于 AD 的服务。 当用户处于企业网络上时，他们甚至无需输入其密码。

有关配置 PingFederate 以与 Azure Active Directory 一起使用的详细信息，请参阅 [PingFederate 与 Azure Active Directory 和 Office 365 的集成](https://www.pingidentity.com/AzureADConnect)

有关使用 PingFederate 设置 Azure AD Connect 的信息，请参阅 [Azure AD Connect 自定义安装](how-to-connect-install-custom.md#configuring-federation-with-pingfederate)

#### <a name="sign-in-by-using-an-earlier-version-of-ad-fs-or-a-third-party-solution"></a>使用早期版本的 AD FS 或第三方解决方案登录
如果已使用早期版本的 AD FS（例如 AD FS 2.0）或第三方联合身份验证提供程序配置了云登录，则可以通过 Azure AD Connect 选择跳过用户登录配置。 这样，便可以获取最新的同步和 Azure AD Connect 的其他功能，同时仍可使用现有的解决方案进行登录。

有关详细信息，请参阅 [Azure AD 第三方联合身份验证兼容性列表](how-to-connect-fed-compatibility.md)。


## <a name="user-sign-in-and-user-principal-name"></a>用户登录名和用户主体名
### <a name="understanding-user-principal-name"></a>了解用户主体名
在 Active Directory 中，默认的用户主体名 (UPN) 后缀是在其中创建用户帐户的域的 DNS 名称。 在大多数情况下，这是在 Internet 上注册为企业域的域名。 但是，可以使用 Active Directory 域和信任来添加更多的 UPN 后缀。

用户的 UPN 的格式为 username@domain。 例如，对于名为“contoso.com”的 Active Directory 域，名为 John 的用户的 UPN 可能是“john@contoso.com”。 用户的 UPN 基于 RFC 822。 尽管 UPN 和电子邮件共享相同的格式，但用户的 UPN 值与用户的电子邮件地址可能相同，也可能不相同。

### <a name="user-principal-name-in-azure-ad"></a>Azure AD 中的用户主体名
Azure AD Connect 向导使用 userPrincipalName 属性，或让你从本地指定要用作 Azure AD 中的用户主体名的属性（在自定义安装中）。 这是用于登录 Azure AD 的值。 如果 userPrincipalName 属性的值不对应于 Azure AD 中已验证的域，则 Azure AD 会将该值替换为默认的 .partner.onmschina.cn 值。

Azure Active Directory 中的每个目录随附内置域名，格式为 contoso.partner.onmschina.cn，可让你开始使用 Azure 或其他 Microsoft 服务。 可以使用自定义域来改善和简化登录体验。 有关 Azure AD 中的自定义域名以及如何验证域的信息，请阅读[将自定义域名添加到 Azure Active Directory](../fundamentals/add-custom-domain.md)。

## <a name="azure-ad-sign-in-configuration"></a>Azure AD 登录配置
### <a name="azure-ad-sign-in-configuration-with-azure-ad-connect"></a>使用 Azure AD Connect 配置 Azure AD 登录
Azure AD 登录体验取决于 Azure AD是否能够匹配要同步到某个自定义域（在 Azure AD 目录中已验证）的用户的用户主体名后缀。 在配置 Azure AD 登录设置时 Azure AD Connect 将提供帮助，使用户在云中能获得类似于本地登录的登录体验。

Azure AD Connect 列出了为域定义的 UPN 后缀，并尝试在 Azure AD 中将其与自定义域进行匹配。 然后它会帮助你执行需要执行的相应操作。
Azure AD 登录页列出了为本地 Active directory 定义的 UPN 后缀，并根据每个后缀显示相应的状态。 状态值可以是下列其中一项：

| 状态 | 说明 | 所需操作 |
|:--- |:--- |:--- |
| 已验证 |Azure AD Connect 在 Azure AD 中找到匹配的已验证域。 此域的所有用户均可使用其本地凭据登录。 |无需采取任何措施。 |
| 未验证 |Azure AD Connect 在 Azure AD 中找到了匹配的但未验证的自定义域。 如果域未验证，则在同步后此域的用户的 UPN 后缀将更改为默认的 .partner.onmschina.cn 后缀。 | [在 Azure AD 中验证自定义域。](../fundamentals/add-custom-domain.md#verify-your-custom-domain-name) |
| 未添加 |Azure AD Connect 未找到对应于 UPN 后缀的自定义域。 如果未在 Azure 中添加域且域未进行验证，则此域的用户的 UPN 后缀将更改为默认的 .partner.onmschina.cn 后缀。 | [添加和验证与 UPN 后缀相对应的自定义域。](../fundamentals/add-custom-domain.md) |

Azure AD 登录页列出了针对本地 Active Directory 定义的 UPN 后缀，以及 Azure AD 中对应的自定义域与当前验证状态。 在自定义安装中，现在可以在“Azure AD 登录”页上选择用户主体名的属性。 

![Azure AD 登录页](./media/plan-connect-user-signin/custom_azure_sign_in.png)

可以单击“刷新”按钮，从 Azure AD 中重新提取自定义域最新的状态。

### <a name="selecting-the-attribute-for-the-user-principal-name-in-azure-ad"></a>选择 Azure AD 中的用户主体名的属性
属性 userPrincipalName 是用户登录 Azure AD 和 Office 365 时使用的属性。 应在同步处理用户之前对在 Azure AD 中使用的域（也称为 UPN 后缀）进行验证。

强烈建议保留默认属性 userPrincipalName。 如果此属性不可路由且无法验证，则可以选择另一个属性（例如 email）作为保存登录 ID 的属性。 这就是所谓的备用 ID。 “备用 ID”属性值必须遵循 RFC 822 标准。

> [!NOTE]
> 所有 Office 365 工作负荷都不允许使用替代 ID。 有关详细信息，请参阅[配置备用登录 ID](https://technet.microsoft.com/library/dn659436.aspx)。
>
>

#### <a name="different-custom-domain-states-and-their-effect-on-the-azure-sign-in-experience"></a>不同的自定义域状态及其对 Azure 登录体验的影响
请务必要了解 Azure AD 目录中的自定义域状态与本地定义的 UPN 后缀之间的关系。 让我们逐步了解当使用 Azure AD Connnect 设置同步时可能遇到的不同 Azure 登录体验。

对于下面的信息，假设我们所关注的是 UPN 后缀 contoso.com，它在本地目录中用作 UPN 的一部分，例如 user@contoso.com。

###### <a name="express-settingspassword-hash-synchronization"></a>快速设置/密码哈希同步

| 状态 | 对 Azure 用户登录体验的影响 |
|:---:|:--- |
| 未添加 |在这种情况下，并未在 Azure AD 目录中针对 contoso.com 添加任何自定义域。 在本地具有后缀 @contoso.com 的 UPN 的用户将无法使用其本地 UPN 来登录 Azure。 他们需要为默认的 Azure AD 目录添加后缀，以改用 Azure AD 向他们提供的新 UPN。 例如，如果要将用户同步到 Azure AD 目录 azurecontoso.partner.onmschina.cn，则为本地用户 user@contoso.com 指定 UPN user@azurecontoso.partner.onmschina.cn。 |
| 未验证 |在这种情况下，我们拥有已添加在 Azure AD 目录中的自定义域 contoso.com。 但是，该域尚未验证。 如果在没有验证域的情况下继续同步用户，则 Azure AD 将为用户分配新 UPN，如同“未添加”方案中所做的一样。 |
| 已验证 |在这种情况下，我们拥有已在 Azure AD 中为 UPN 后缀添加并验证了的自定义域 contoso.com。 在用户被同步到 Azure AD 后，用户可以使用其本地用户主体名（例如 user@contoso.com）登录到 Azure。 |

###### <a name="ad-fs-federation"></a>AD FS 联合
无法使用 Azure AD 中的默认 .partner.onmschina.cn 域或 Azure AD 中未验证的自定义域创建联合。 在运行 Azure AD Connect 向导时，如果选择使用未验证的域创建联合，则 Azure AD Connect 将发出提示，并指出要为域创建的将托管 DNS 的必需记录。 有关详细信息，请参阅[验证选择用于联合的 Azure AD 域](how-to-connect-install-custom.md#verify-the-azure-ad-domain-selected-for-federation)。

如果选择的用户登录选项为“与 AD FS 联合”，则必须有一个自定义域才能继续在 Azure AD 中创建联合。  针对我们的讨论，这意味着我们应在 Azure AD 目录中添加自定义域 contoso.com。

| 状态 | 对 Azure 用户登录体验的影响 |
|:---:|:--- |
| 未添加 |在这种情况下，Azure AD Connect 没有在 Azure AD 目录中找到 UPN 后缀 contoso.com 的匹配自定义域。 如果需要让用户在 AD FS 中使用其本地 UPN（例如 user@contoso.com）登录，则需要添加自定义域 contoso.com。 |
| 未验证 |在这种情况下，Azure AD Connect 将发出提示，并提供有关如何在后面的阶段验证域的相应详细信息。 |
| 已验证 |在这种情况下，可以继续进行配置，而不需要采取任何进一步的操作。 |

## <a name="changing-the-user-sign-in-method"></a>更改用户登录方法 <a name="changing-user-sign-in-method"></a>
在使用向导完成 Azure AD Connect 的初始配置后，可以使用 Azure AD Connect 中的可用任务将用户的登录方法在“联合”、“密码哈希同步之间切换。 再次运行 Azure AD Connect 向导，随后将看到可执行的任务列表。 在任务列表中选择“更改用户登录”。 

![更改用户登录](./media/plan-connect-user-signin/changeusersignin.png)

在下一页上，系统将要求你提供 Azure AD 的凭据。

![连接到 Azure AD](./media/plan-connect-user-signin/changeusersignin2.png)

在“用户登录”  页上，选择所需的用户登录选项。

![连接到 Azure AD](./media/plan-connect-user-signin/changeusersignin2a.png)

> [!NOTE]
> 如果只是要暂时切换到密码哈希同步，请选中“请勿切换用户帐户”  复选框。 不选中该选项会将每个用户转换为联合用户，并且该操作可能需要花费几小时。
>
>

## <a name="next-steps"></a>后续步骤
- 了解有关[将本地标识与 Azure Active Directory 集成](whatis-hybrid-identity.md)的详细信息。
- 了解有关 [Azure AD Connect 设计概念](plan-connect-design-concepts.md)的详细信息。

