---
title: Azure 流量分析架构更新 - 2020 年 3 月 | Azure Docs
description: 在流量分析架构中包含新字段的示例查询。
services: network-watcher
documentationcenter: na
author: vinigam
manager: agummadi
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/30/2020
ms.author: v-tawe
origin.date: 03/06/2020
ms.openlocfilehash: f9e29e8036c70e1a769bb5e4b5f7e1005bf51e1a
ms.sourcegitcommit: b81ea2ab9eafa986986fa3eb1e784cfe9bbf9ec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/13/2020
ms.locfileid: "83367820"
---
# <a name="sample-queries-with-new-fields-in-traffic-analytics-schema-august-2019-schema-update"></a>流量分析架构中包含新字段的示例查询（2019 年 8 月版架构更新）

[流量分析日志架构](https://docs.azure.cn/network-watcher/traffic-analytics-schema)已更新，包含以下新字段：SrcPublicIPs_s、DestPublicIPs_s、NSGRule_s  。 在接下来的几个月内，以下较旧的字段将弃用：VMIP_s、Subscription_g、Region_s、NSGRules_s、Subnet_s、VM_s、NIC_s、PublicIPs_s、FlowCount_d        。
新字段提供有关源和目标 IP 的信息并简化查询。

以下三个示例演示如何将旧字段替换为新字段。

## <a name="example-1---vmip_s-subscription_g-region_s-subnet_s-vm_s-nic_s-publicips_s"></a>示例 1 - VMIP_s、Subscription_g、Region_s、Subnet_s、VM_s、NIC_s、PublicIPs_s

我们不必专门从 AzurePublic 和 ExternalPublic 流的 FlowDirection_s 字段中推断 Azure 和外部公共流的源和目标情况。 对于 NVA（网络虚拟设备），FlowDirection_s 字段也可能不适合使用。

```Old Kusto query
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FASchemaVersion_s == "1"
| extend isAzureOrExternalPublicFlows = FlowType_s in ("AzurePublic", "ExternalPublic")
| extend SourceAzureVM = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', VM_s, "N/A"), VM1_s),
SourceAzureVMIP = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', VM_s, "N/A"), SrcIP_s),
SourceAzureVMSubscription = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', Subscription_g, "N/A"), Subscription1_g),
SourceAzureRegion = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', Region_s, "N/A"), Region1_s),
SourceAzureSubnet = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', Subnet_s, "N/A"), Subnet1_s),
SourceAzureNIC = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', NIC_s, "N/A"), NIC1_s),
DestAzureVM = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', VM_s, "N/A"), VM2_s),
DestAzureVMIP = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', VM_s, "N/A"), DestIP_s),
DestAzureVMSubscription = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', Subscription_g, "N/A"), Subscription2_g),
DestAzureRegion = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', Region_s, "N/A"), Region2_s),
DestAzureSubnet = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', Subnet_s, "N/A"), Subnet2_s),
DestAzureNIC = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', NIC_s, "N/A"), NIC2_s),
SourcePublicIPsAggregated = iif(isAzureOrExternalPublicFlows and FlowDirection_s == 'I', PublicIPs_s, "N/A"),
DestPublicIPsAggregated = iif(isAzureOrExternalPublicFlows and FlowDirection_s == 'O', PublicIPs_s, "N/A")
```


```New Kusto query
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FASchemaVersion_s == "2"
| extend SourceAzureVM = iif(isnotempty(VM1_s), VM1_s, "N/A"),
SourceAzureVMIP = iif(isnotempty(SrcIP_s), SrcIP_s, "N/A"),
SourceAzureVMSubscription = iif(isnotempty(Subscription1_g), Subscription1_g, "N/A"),
SourceAzureRegion = iif(isnotempty(Region1_s), Region1_s, "N/A"),
SourceAzureSubnet = iif(isnotempty(Subnet1_s), Subnet1_s, "N/A"),
SourceAzureNIC = iif(isnotempty(NIC1_s), NIC1_s, "N/A"),
DestAzureVM = iif(isnotempty(VM2_s), VM2_s, "N/A"),
DestAzureVMIP = iif(isnotempty(DestIP_s), DestIP_s, "N/A"),
DestAzureVMSubscription = iif(isnotempty(Subscription2_g), Subscription2_g, "N/A"),
DestAzureRegion = iif(isnotempty(Region2_s), Region2_s, "N/A"),
DestAzureSubnet = iif(isnotempty(Subnet2_s), Subnet2_s, "N/A"),
DestAzureNIC = iif(isnotempty(NIC2_s), NIC2_s, "N/A"),
SourcePublicIPsAggregated = iif(isnotempty(SrcPublicIPs_s), SrcPublicIPs_s, "N/A"),
DestPublicIPsAggregated = iif(isnotempty(DestPublicIPs_s), DestPublicIPs_s, "N/A")
```


## <a name="example-2---nsgrules_s"></a>示例 2 - NSGRules_s

早期字段的格式为：<Index value 0)>|<NSG_RULENAME>|<Flow Direction>|<Flow Status>|<FlowCount ProcessedByRule>

我们以前在 NSG 和 NSGRules 之间聚合数据。 现在我们不进行聚合。 因此，NSGList_s 仅包含一个 NSG，NSGRules_s 之前也仅包含一个规则。 我们在此处删除了复杂的格式，同样的情况也存在于下方所述的其他字段中：

```Old Kusto query
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FASchemaVersion_s == "1"
| extend NSGRuleComponents = split(NSGRules_s, "|")
| extend NSGName = NSGList_s // remains same
| extend NSGRuleName = NSGRuleComponents[1],
         FlowDirection = NSGRuleComponents[2],
         FlowStatus = NSGRuleComponents[3],
         FlowCountProcessedByRule = NSGRuleComponents[4]
| project NSGName, NSGRuleName, FlowDirection, FlowStatus, FlowCountProcessedByRule
```

```New Kusto query
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FASchemaVersion_s == "2"
| extend NSGRuleComponents = split(NSGRules_s, "|")
| project NSGName = NSGList_s,
NSGRuleName = NSGRule_s ,
FlowDirection = FlowDirection_s,
FlowStatus = FlowStatus_s,
FlowCountProcessedByRule = AllowedInFlows_d + DeniedInFlows_d + AllowedOutFlows_d + DeniedOutFlows_d
```

## <a name="example-3---flowcount_d"></a>示例 3 - FlowCount_d

由于我们不会跨 NSG 合并数据，因此 FlowCount_d 即为 AllowedInFlows_d + DeniedInFlows_d + AllowedOutFlows_d + DeniedOutFlows_d。
上面 4 个中只有 1 个不为零，剩余的三个为 0。 该字段指示捕获流的 NIC 中的状态和计数。

如果允许该流，则填充一个前缀为“Allowed”的字段。 否则，填充一个前缀为“Denied”的字段。
如果流是入站流，则填充一个后缀为“\_d”（如“InFlows_d”）的字段。 否则，填充“OutFlows_d”。

根据上述 2 个条件，我们知道将填充 4 个中的哪一个。


## <a name="next-steps"></a>后续步骤
若要获取常见问题的解答，请参阅[流量分析常见问题解答](traffic-analytics-faq.md)。若要查看有关功能的详细信息，请参阅[流量分析文档](traffic-analytics.md)
