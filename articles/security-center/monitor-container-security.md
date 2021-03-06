---
title: 在 Azure 安全中心中监视容器的安全性
description: 了解如何从 Azure 安全中心检查容器的安全状况
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 05/13/2020
ms.author: v-tawe
origin.date: 02/12/2020
ms.openlocfilehash: b508be57915ebcbfc0b19330249f68bfffff386d
ms.sourcegitcommit: cbaa1aef101f67bd094f6ad0b4be274bbc2d2537
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/28/2020
ms.locfileid: "84126569"
---
# <a name="monitoring-the-security-of-your-containers"></a>监视容器的安全性

本页说明如何使用概念部分的[容器安全文章](container-security.md)中介绍的容器安全功能。

Azure 安全中心涵盖容器安全的以下三个方面：

- **漏洞管理** - 如果你使用的是安全中心的标准定价层（请参阅[定价](/security-center/security-center-pricing)），则可以在每次推送新映像时扫描基于 ARM 的 Azure 容器注册表。 扫描程序（由 Qualys 提供支持）会以安全中心建议的方式提供发现结果。
    有关详细说明，请参阅下文中的[扫描容器注册表中的漏洞](#scanning-your-arm-based-container-registries-for-vulnerabilities)。

- **强化容器的 Docker 主机** - 安全中心发现托管在 IaaS Linux VM 或其他运行 Docker 的 Linux 计算机上的非托管容器，并持续将容器的配置与 Internet 安全中心 (CIS) Docker 基准进行比较。 如果容器不满足任何控件的要求，安全中心会发出警报。 持续监控因配置不当造成的安全风险是任何安全计划的关键组成部分。 
    有关详细说明，请参阅下文中的[强化容器的 Docker 主机](#hardening-your-containers-docker-hosts)。

- **强化 Azure Kubernetes 服务群集** - 安全中心会在发现 Azure Kubernetes 服务群集配置中的漏洞时提供建议。 有关可能提供的具体建议的详细信息，请参阅 [Kubernetes 服务建议](recommendations-reference.md#recs-containers)。

- **运行时保护** - 如果你使用的是安全中心的标准定价层，则会获得针对容器化环境的实时威胁保护。 安全中心在主机和 AKS 群集级别生成可疑活动的警报。 有关可能出现的相关安全警报的详细信息，请参阅 [Azure Kubernetes 服务群集的警报](alerts-reference.md#alerts-akscluster)和警报参考表的[容器的警报 - 主机级别](alerts-reference.md#alerts-containerhost)部分。

## <a name="scanning-your-arm-based-container-registries-for-vulnerabilities"></a>扫描基于 ARM 的容器注册表中的漏洞 

1. 启用 Azure 容器注册表映像的漏洞扫描：

    1. 请确保你使用的是 Azure 安全中心的标准定价层。

    1. 在“定价和设置”页面，为你的订阅启用可选的容器注册表包：![启用容器注册表包](media/monitor-container-security/enabling-container-registries-bundle.png)

        安全中心现在可以扫描推送到注册表的映像。 

        >[!NOTE]
        >此功能按映像收费。


1. 若要触发映像的扫描，请将其推送到注册表。 

    扫描完成后（通常在大约 10 分钟后），安全中心建议中会提供发现结果。
    

1. 若要查看发现结果，请访问“建议”页面。 如果发现问题，你会看到以下建议：

    ![解决问题的建议 ](media/monitor-container-security/acr-finding.png)


1. 选择建议。 
    “建议详细信息”页随即打开，并且其中包含其他信息。 这些信息包括具有易受攻击映像（“受影响的资源”）的注册表列表以及补救步骤。 

1. 选择特定的注册表以查看其中具有易受攻击存储库的存储库。

    ![选择一个注册表](media/monitor-container-security/acr-finding-select-registry.png)

    “注册表详细信息”页随即打开，并列出受影响的存储库。

1. 选择特定的存储库以查看其中具有易受攻击映像的存储库。

    ![选择存储库](media/monitor-container-security/acr-finding-select-repository.png)

    “存储库详细信息”页随即打开。 它列出了易受攻击的映像以及对发现结果严重性的评估。

1. 选择特定映像以查看漏洞。

    ![选择映像](media/monitor-container-security/acr-finding-select-image.png)

    所选映像的发现结果列表随即打开。

    ![发现结果列表](media/monitor-container-security/acr-findings.png)

1. 若要详细了解发现结果，请选择“发现结果”。 

    发现结果详细信息窗格随即打开。

    [![发现结果详细信息窗格](media/monitor-container-security/acr-finding-details-pane.png)](media/monitor-container-security/acr-finding-details-pane.png#lightbox)

    此窗格包括问题的详细说明，以及有助于缓解威胁的外部资源的链接。

1. 按照此窗格的“修正”部分中的步骤进行操作。

1. 执行修复安全问题所需的步骤后，请替换注册表中的映像：

    1. 推送已更新的映像。 这会触发扫描。 
    
    1. 请查看建议页面，了解“应修复 Azure 容器注册表映像中的漏洞”的建议。 
    
        如果建议仍然显示，并且你处理的映像仍显示在易受攻击映像列表中，请再次检查修正步骤。

    1. 确定更新的映像已推送、已扫描，且不再显示在建议中时，请从注册表中删除“旧”版本易受攻击的映像。


## <a name="hardening-your-containers-docker-hosts"></a>强化容器的 Docker 主机

安全中心不断监视 Docker 主机的配置，并会生成反映行业标准的安全建议。

查看适用于容器的 Docker 主机的 Azure 安全中心安全建议：

1. 从安全中心导航栏中，打开“计算和应用”，然后选择“容器”选项卡 。

1. （可选）将容器资源列表筛选为容器主机。

    ![容器资源筛选器](media/monitor-container-security/container-resources-filter.png)

1. 从容器主机计算机列表中，选择一台进行进一步调查。

    ![容器主机建议](media/monitor-container-security/container-resources-filtered-to-hosts.png)

    “容器主机信息页”随即打开，其中包含主机的详细信息和建议列表。

1. 从“建议”列表中，选择要进一步调查的建议。

    ![容器主机建议列表](media/monitor-container-security/container-host-rec.png)

1. （可选）阅读说明、信息、威胁和修正步骤。 

1. 选择页面底部的“执行操作”。

    [![“执行操作”按钮](media/monitor-container-security/host-security-take-action-button.png)](media/monitor-container-security/host-security-take-action.png#lightbox)

    Log Analytics 随即打开，其中包含可运行的自定义操作。 默认自定义查询包括评估的所有失败规则的列表，以及有助于你解决问题的指南。

    [![Log Analytics 操作](media/monitor-container-security/log-analytics-for-action-small.png)](media/monitor-container-security/log-analytics-for-action.png#lightbox)

1. 如有必要，请调整查询参数。

1. 当确定命令适合主机使用时，请选择“运行”。



## <a name="next-steps"></a>后续步骤

本文介绍如何使用安全中心的容器安全功能。 

有关其他相关材料，请参阅以下页面： 

- [容器的安全中心建议](recommendations-reference.md#recs-containers)
- [AKS 群集级别的警报](alerts-reference.md#alerts-akscluster)
- [容器主机级别的警报](alerts-reference.md#alerts-containerhost)
