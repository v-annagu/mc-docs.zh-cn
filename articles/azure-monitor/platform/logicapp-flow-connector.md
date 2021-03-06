---
title: 将 Azure Monitor 日志与 Azure 逻辑应用和 Power Automate 配合使用
description: 了解如何使用 Azure 逻辑应用和 Power Automate 通过 Azure Monitor 连接器快速自动执行可重复的过程。
ms.service: azure-monitor
ms.subservice: logs
ms.topic: conceptual
author: Johnnytechn
ms.author: v-johya
ms.date: 05/20/2020
ms.openlocfilehash: 52d9087de92078275bc457fc5aa11db00493374c
ms.sourcegitcommit: a04b0b1009b0c62f2deb7c7acee75a1304d98f87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/22/2020
ms.locfileid: "83796951"
---
# <a name="azure-monitor-logs-connector-for-logic-apps-and-flow"></a>适用于逻辑应用和流的 Azure Monitor 日志连接器
借助 [Azure 逻辑应用](/logic-apps/) 和 [Power Automate](https://ms.flow.microsoft.com)，可以使用数百个操作为多种服务创建自动化工作流。 使用 Azure Monitor 日志连接器，可以在 Azure Monitor 中生成可从 Log Analytics 工作区或 Application Insights 应用程序检索数据的工作流。 本文介绍该连接器附带的操作，并演练如何使用这些数据生成工作流。

例如，可以创建一个逻辑应用，以在 Office 365 的电子邮件通知中使用 Azure Monitor 日志数据，在 Azure DevOps 中创建 Bug，或者发布 Slack 消息。  可通过简单计划或从连接的服务中的某些操作（例如收到电子邮件或推文时）触发工作流。 

## <a name="actions"></a>操作
下表描述了 Azure Monitor 日志连接器附带的操作。 可通过这两个操作对 Log Analytics 工作区或 Application Insights 应用程序运行日志查询。 两者的差异在于返回数据的方式。

> [!NOTE]
> Azure Monitor 日志连接器取代了 [Azure Log Analytics 连接器](https://docs.microsoft.com/connectors/azureloganalytics/)和 [Azure Application Insights 连接器](https://docs.microsoft.com/connectors/applicationinsights/)。 此连接器的功能与被取代的连接器相同，是针对 Log Analytics 工作区或 Application Insights 应用程序运行查询的首选方法。


| 操作 | 说明 |
|:---|:---|
| [运行查询并列出结果](https://docs.microsoft.com/connectors/azuremonitorlogs/#run-query-and-list-results) | 以自身对象的形式返回每行。 若要在工作流的剩余阶段单独处理每行，请使用此操作。 该操作通常后接 [For each 活动](../../logic-apps/logic-apps-control-flow-loops.md#foreach-loop)。 |
| [运行查询并将结果可视化](https://docs.microsoft.com/connectors/azuremonitorlogs/#run-query-and-visualize-results) | 以单个带有格式的对象形式返回结果集中的所有行。 若要在工作流的剩余阶段使用整个结果集（例如，在邮件中发送结果），请使用此操作。  |

## <a name="walkthroughs"></a>演练
以下教程演示了 Azure 逻辑应用中 Azure Monitor 连接器的用法。 可以在 Power Automate 中执行相同的示例，唯一的差别是创建初始工作流并在完成创建后运行工作流的方式。 两者的工作流和操作配置相同。 若要开始，请参阅[在 Power Automate 中从模板创建流](https://docs.microsoft.com/power-automate/get-started-logic-template)。


### <a name="create-a-logic-app"></a>创建逻辑应用

在 Azure 门户中转到“逻辑应用”，然后单击“添加”。  选择用于存储新逻辑应用的“订阅”、“资源组”和“区域”，并为逻辑应用指定唯一的名称。  

![创建逻辑应用](./media/logicapp-flow-connector/create-logic-app.png)


单击“查看 + 创建”，然后选择“创建” 。 部署完成后，单击“转到资源”打开“逻辑应用设计器”。 

### <a name="create-a-trigger-for-the-logic-app"></a>为逻辑应用创建触发器
在“从通用触发器开始”下，选择“重复”。  这会创建一个按固定间隔自动运行的逻辑应用。 在操作的“频率”框中选择“天”，然后在“间隔”框中输入 **1** 以每日运行工作流一次。  

![重复操作](./media/logicapp-flow-connector/recurrence-action.png)

## <a name="walkthrough-mail-visualized-results"></a>演练：邮件可视化结果
以下教程介绍了如何创建一个通过电子邮件发送 Azure Monitor 日志查询结果的逻辑应用。 

### <a name="add-azure-monitor-logs-action"></a>添加 Azure Monitor 日志操作
单击“+ 新建步骤”，添加在完成重复操作后运行的操作。 在“选择操作”中键入 **azure monitor**，然后选择“Azure Monitor 日志”。 

![Azure Monitor 日志操作](./media/logicapp-flow-connector/select-azure-monitor-connector.png)

单击“Azure Log Analytics - 运行查询并将结果可视化”。

![运行查询并将结果可视化的操作](./media/logicapp-flow-connector/select-query-action-visualize.png)


### <a name="add-azure-monitor-logs-action"></a>添加 Azure Monitor 日志操作

选择 Log Analytics 工作区的“订阅”和“资源组”。  为“资源类型”选择“Log Analytics 工作区”，然后在“资源名称”下选择工作区的名称。 

将以下日志查询添加到“查询”窗口中。  

```Kusto
Event
| where EventLevelName == "Error" 
| where TimeGenerated > ago(1day)
| summarize TotalErrors=count() by Computer
| sort by Computer asc   
```

为“时间范围”选择“在查询中设置”，为“图表类型”选择“HTML 表”。  
   
![运行查询并将结果可视化的操作](./media/logicapp-flow-connector/run-query-visualize-action.png)

与当前连接关联的帐户将发送邮件。 可以单击“更改连接”来指定另一帐户。

### <a name="add-email-action"></a>添加电子邮件操作

依次单击“+ 新建步骤”、“+ 添加操作” 。 在“选择操作”中键入 **outlook**，然后选择“Office 365 Outlook”。 

![选择 Outlook 连接器](./media/logicapp-flow-connector/select-outlook-connector.png)

选择“发送电子邮件(V2)”。

![Office 365 Outlook 选择窗口](./media/logicapp-flow-connector/select-mail-action.png)

单击“正文”框中的任意位置打开“动态内容”窗口，其中包含逻辑应用中以前操作的值。  选择“查看更多”，然后选择“正文”，其中显示了 Log Analytics 操作中的查询结果。 

![选择正文](./media/logicapp-flow-connector/select-body.png)

在“收件人”窗口中指定收件人电子邮件地址，在“主题”中指定主题 。 

![邮件操作](./media/logicapp-flow-connector/mail-action.png)


### <a name="save-and-test-your-logic-app"></a>保存并测试逻辑应用
单击“保存”，然后单击“运行”以执行逻辑应用的测试运行。 

![“保存”和“运行”](./media/logicapp-flow-connector/save-run.png)


逻辑应用完成时，请检查指定的收件人的邮件。  应已收到正文类似以下内容的邮件：

![示例电子邮件](./media/logicapp-flow-connector/sample-mail.png)



## <a name="next-steps"></a>后续步骤

- 详细了解 [Azure Monitor 中的日志查询](../log-query/log-query-overview.md)。
- 详细了解[逻辑应用](/logic-apps/)
- 详细了解 [Microsoft Flow](https://ms.flow.microsoft.com)。


