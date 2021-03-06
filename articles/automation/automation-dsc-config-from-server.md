---
title: 从现有服务器创建配置 - Azure 自动化
description: 了解如何从 Azure 自动化的现有服务器创建配置。
keywords: dsc,powershell,配置,设置
services: automation
ms.service: automation
ms.subservice: dsc
author: WenJason
ms.author: v-jay
origin.date: 08/08/2019
ms.date: 05/11/2020
ms.topic: conceptual
manager: digimobile
ms.openlocfilehash: f756a7daf9e570888616a3693a0e5620c16d3238
ms.sourcegitcommit: 7443ff038ea8afe511f7419d9c550d27fb642246
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2020
ms.locfileid: "83001645"
---
# <a name="create-configurations-from-existing-servers"></a>从现有服务器创建配置

> 适用于：Windows PowerShell 5.1

从现有服务器创建配置可能很难。
可能不需要所有设置，只需要你关注的设置。 
即使在这个时候，仍需了解必须以什么顺序应用设置，以便成功应用配置。

> [!NOTE]
> 本文引用了一个由开放源代码社区维护的解决方案。
> 支持仅以 GitHub 协作的形式提供，而不是由 Azure 提供。

## <a name="community-project-reversedsc"></a>社区项目：ReverseDSC

已创建一个名为 [ReverseDSC](https://github.com/microsoft/reversedsc) 的社区维护解决方案，目的是让其在此领域中使用（从 SharePoint 开始）。

该解决方案基于 [SharePointDSC 资源](https://github.com/powershell/sharepointdsc)并对其进行了扩展，目的是协调从现有 SharePoint 服务器[收集信息](https://github.com/Microsoft/sharepointDSC.reverse#how-to-use)的操作。
最新版有多个[提取模式](https://github.com/Microsoft/SharePointDSC.Reverse/wiki/Extraction-Modes)，用于确定要包括何种级别的信息。

使用此解决方案的结果是生成与 SharePointDSC 配置脚本配合使用的[配置数据](https://github.com/Microsoft/sharepointDSC.reverse#configuration-data)。

生成数据文件后，可以在 [DSC 配置脚本](https://docs.microsoft.com/powershell/scripting/dsc/overview/overview)中使用这些文件生成 MOF 文件，并[将 MOF 文件上传到 Azure 自动化](/automation/tutorial-configure-servers-desired-state#create-and-upload-a-configuration-to-azure-automation)。
然后从[本地](/automation/automation-dsc-onboarding#onboarding-physicalvirtual-windows-machines-on-premises-or-in-a-cloud-other-than-azure)或[在 Azure 中](/automation/automation-dsc-onboarding#onboarding-azure-vms)注册服务器以拉取配置。

若要试用 ReverseDSC，请访问 [PowerShell 库](https://www.powershellgallery.com/packages/ReverseDSC/)并下载解决方案，或单击“项目站点”以查看[文档](https://github.com/Microsoft/sharepointDSC.reverse)。

## <a name="next-steps"></a>后续步骤

- [Windows PowerShell Desired State Configuration 概述](https://docs.microsoft.com/powershell/scripting/dsc/overview/overview)
- [DSC 资源](https://docs.microsoft.com/powershell/scripting/dsc/resources/resources)
- [配置本地配置管理器](https://docs.microsoft.com/powershell/scripting/dsc/managing-nodes/metaconfig)
