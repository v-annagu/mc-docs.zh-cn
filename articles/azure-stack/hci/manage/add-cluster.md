---
title: 添加或删除 Azure Stack HCI 群集的服务器
description: 了解如何在 Azure Stack HCI 的群集中添加或删除服务器节点
ms.topic: article
author: WenJason
ms.author: v-jay
ms.reviewer: jgerend
origin.date: 05/20/2020
ms.date: 06/22/2020
ms.openlocfilehash: eabfc6c2fef195022d0c794089702018879e50c7
ms.sourcegitcommit: d86e169edf5affd28a1c1a4476d72b01a7fb421d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2020
ms.locfileid: "85096876"
---
# <a name="add-or-remove-servers-for-an-azure-stack-hci-cluster"></a>添加或删除 Azure Stack HCI 群集的服务器

> 适用于：Windows Server 2019

可以在 Azure Stack HCI 的群集中轻松地添加或删除服务器。 请记住，在 CPU 类型、内存、驱动器数量以及驱动器的类型和大小方面，每个新的物理服务器必须与群集中的其他服务器严格匹配。

无论何时添加或删除服务器，都还必须在之后执行群集验证，以确保群集正常运行。

## <a name="obtain-server-hardware-from-your-oem"></a>从 OEM 获取服务器硬件

第一步是从原始 OEM 获取新的 HCI 硬件。 添加要在群集中使用的新服务器硬件时，请始终参考 OEM 提供的文档。

1. 将新的物理服务器置于机架中并进行正确的布线。
1. 启用物理交换机端口，并根据情况调整访问控制列表 (ACL) 和 VLAN ID。
1. 按照 OEM 说明在基板管理控制器 (BMC) 中配置正确的 IP 地址，并应用所有 BIOS 设置。
1. 使用 OEM 提供的工具，将当前的固件基线应用于所有组件。
1. 运行 OEM 验证测试，以确保与现有群集服务器的同质性。

## <a name="add-the-server-to-the-cluster"></a>将服务器添加到群集

服务器正确启动后，请使用 Windows 管理中心将服务器加入群集。

:::image type="content" source="media/manage-cluster/add-server.png" alt-text="“添加服务器”屏幕":::

1. 在“Windows 管理中心”中，从顶部下拉箭头中选择“群集管理器”。
1. 在“群集连接”下，选择该群集。
1. 在“工具”下，选择“服务器”。
1. 在“服务器”下，选择“库存”选项卡。
1. 在“库存”选项卡上，选择“添加”。
1. 在“服务器名称”中，输入要添加的服务器的完全限定域名，单击“添加”，然后再次单击底部的“添加”。
1. 验证服务器是否已成功添加到群集。

## <a name="remove-a-server-from-the-cluster"></a>从群集中删除服务器

从群集中删除服务器的步骤类似于向群集添加服务器的步骤。

请记住，删除服务器时，也会删除与该服务器关联的所有虚拟机、驱动器和工作负荷。

:::image type="content" source="media/manage-cluster/remove-server.png" alt-text="“删除服务器”对话框":::

1. 在“Windows 管理中心”中，从顶部下拉箭头中选择“群集管理器”。
1. 在“群集连接”下，选择该群集。
1. 在“工具”下，选择“服务器”。
1. 在“服务器”下，选择“库存”选项卡。
1. 在“库存”选项卡上，选择要删除的服务器，然后选择“删除”。
1. 若要同时从存储池中删除任何服务器驱动器，请启用该复选框。
1. 验证服务器是否已成功从群集中删除。

## <a name="validate-the-cluster"></a>验证群集

每当在群集中添加或删除服务器时，都必须验证群集。 若要获得 Microsoft 的支持，群集验证必须无错误通过。

:::image type="content" source="media//manage-cluster/validate-cluster.png" alt-text="“验证群集”对话框":::

1. 在“Windows 管理中心”中，从顶部下拉箭头中选择“群集管理器”。
1. 在“工具”下，选择“服务器”。
1. 在“服务器”下，依次选择“库存”选项卡、“更多”、“验证群集”。
1. 验证是否已成功通过所有验证步骤。

## <a name="next-steps"></a>后续步骤

- 若要详细了解群集验证，请参阅 [Validate Hardware for a Failover Cluster](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj134244(v=ws.11))（验证故障转移群集的硬件）。