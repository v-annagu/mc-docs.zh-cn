---
title: include 文件
description: include 文件
services: vpn-gateway
author: WenJason
ms.service: vpn-gateway
ms.topic: include
origin.date: 03/03/2020
ms.date: 04/06/2020
ms.author: v-jay
ms.custom: include file
ms.openlocfilehash: 6b0bd03686cc6abc7d5884f64c4a1fc076cefa24
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "80634601"
---
可以通过以下步骤使用资源管理器部署模型和 Azure 门户创建一个 VNet。 有关虚拟网络的详细信息，请参阅[虚拟网络概述](../articles/virtual-network/virtual-networks-overview.md)。

>[!NOTE]
>使用虚拟网络作为跨界体系结构的一部分时，请务必与本地网络管理员进行协调，以划分一个 IP 地址范围专供此虚拟网络使用。 如果 VPN 连接的两端存在重复的地址范围，则会以意外方式路由流量。 此外，若要将此虚拟网络连接到另一个虚拟网络，地址空间不能与另一虚拟网络重叠。 相应地规划网络配置。
>
>

1. 登录到 [Azure 门户](https://portal.azure.cn)。
1. 在“搜索资源、服务和文档(G+/)”中，键入“虚拟网络”。  

   ![查找“虚拟网络”资源页](./media/vpn-gateway-basic-vnet-rm-portal-include/marketplace.png "查找“虚拟网络”资源页")
1. 从“市场”结果中选择“虚拟网络”。  

   ![选择虚拟网络](./media/vpn-gateway-basic-vnet-rm-portal-include/marketplace-results.png "查找“虚拟网络”资源页")
1. 在“虚拟网络”页上选择“创建”。  

   ![虚拟网络页](./media/vpn-gateway-basic-vnet-rm-portal-include/vnet-click-create.png "选择“创建”")
1. 选择“创建”后，会打开“创建虚拟网络”页。  
1. 在“基本信息”选项卡上，配置“项目详细信息”和“实例详细信息”VNet 设置。   

   ![“基本信息”选项卡](./media/vpn-gateway-basic-vnet-rm-portal-include/basics.png "“基本信息”选项卡")在填写字段时，如果在字段中输入的字符通过了验证，则会出现绿色的对钩标记。 某些值是自动填写的，你可以将其替换为自己的值：

   - **订阅**：确认列出的订阅是正确的。 可以使用下拉列表更改订阅。
   - **资源组**：选择现有资源组，或单击“新建”以创建新资源组  。 有关资源组的详细信息，请参阅 [Azure 资源管理器概述](../articles/azure-resource-manager/management/overview.md#resource-groups)。
   - **名称**：输入虚拟网络的名称。
   - **区域**：选择 VNet 的位置。 该位置确定要部署到此 VNet 的资源将位于哪里。

1. 在“IP 地址”选项卡上配置值。  以下示例中显示的值用于演示目的。 根据所需的设置调整这些值。

   ![“IP 地址”选项卡](./media/vpn-gateway-basic-vnet-rm-portal-include/addresses.png "“IP 地址”选项卡")  
   - **IPv4 地址空间**：默认情况下，系统会自动创建一个地址空间。 可以单击该地址空间，将其调整为反映你自己的值。 还可以添加更多的地址空间。
   - **IPv6**：如果配置需要 IPv6 地址空间，请选中“添加 IPv6 地址空间”框以输入该信息。 
   - **子网**：如果你使用默认地址空间，则系统会自动创建一个默认子网。 如果更改了地址空间，则需要添加子网。 选择“+ 添加子网”打开“添加子网”窗口。   配置以下设置，然后选择“添加”以添加值： 
      - **子网名称**：在此示例中，我们已将子网命名为“FrontEnd”。
      - **子网地址范围**：此子网的地址范围。

1. 在“安全性”选项卡上，此时请保留默认值： 

   - **DDos 防护**：基本
   - **防火墙**：已禁用
1. 选择“查看 + 创建”以验证虚拟网络设置。 
1. 验证设置后，选择“创建”。 
