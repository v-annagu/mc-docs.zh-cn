---
ms.openlocfilehash: a8c084b3a7d479c3a471c19d05a4bcc81a055227
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "63827127"
---
## <a name="what-are-service-bus-queues"></a>什么是 Service Bus 队列？

服务总线队列支持中转消息传送  通信模型。 在使用队列时，分布式应用程序的组件不会直接相互通信，而是通过充当中介（代理）的队列交换消息。 消息创建方（发送方）将消息传送到队列，然后继续对其进行处理。 消息使用方（接收方）以异步方式从队列中提取消息并处理它。 创建方不必等待使用方的答复即可继续处理并发送更多消息。 队列为一个或多个竞争使用方提供**先入先出 (FIFO)** 消息传递方式。 也就是说，接收方通常会按照消息添加到队列中的顺序来接收并处理消息，并且每条消息仅由一个消息使用方接收并处理。

![QueueConcepts](./media/howto-service-bus-queues/sb-queues-08.png)

Service Bus 队列是一种可用于各种应用场景的通用技术：

-   多层 Azure 应用程序中 Web 角色和辅助角色之间的通信。
-   混合解决方案中本地应用程序和 Azure 托管应用程序之间的通信。
-   在不同组织或组织的各部门中本地运行的分布式应用程序组件之间的通信。

利用队列，可以更轻松地缩放应用程序，并增强体系结构的弹性。

## <a name="create-a-service-namespace"></a>创建服务命名空间

若要开始在 Azure 中使用服务总线队列，必须先创建一个服务命名空间。 命名空间提供了用于对应用程序中的 Service Bus 资源进行寻址的范围容器。

创建服务命名空间：

1.  登录到 [Azure 经典门户][]。

2.  在门户的左侧导航窗格中，单击“服务总线”  。

3.  在门户的下方窗格中，单击“创建”  。
    ![](./media/howto-service-bus-queues/sb-queues-03.png)

4.  在“添加新命名空间”对话框中，输入命名空间名称  。 系统会立即检查该名称是否可用。   
    ![](./media/howto-service-bus-queues/sb-queues-04.png)

5.  在确保命名空间名称可用后，选择应承载命名空间的国家或地区（确保使用在其中部署计算资源的同一国家/地区）。

     > [!IMPORTANT]
     > 选取要选择用于部署应用程序的**相同区域**。 这会提供最佳性能。

6.  将对话框中的其他字段保留其默认值（“消息传送”和“标准层”），然后单击“确定”复选标记   。 系统现已创建命名空间并已将其启用。 可能需要等待几分钟，因为系统将为帐户配置资源。

    ![](./media/howto-service-bus-queues/getting-started-multi-tier-27.png)

创建的命名空间将花费一段时间来激活，然后显示在门户中。 请等到命名空间状态变为“活动”后再继续操作  。

## <a name="obtain-the-default-management-credentials-for-the-namespace"></a>获取命名空间的默认管理凭据

若要在新命名空间上执行管理操作（如创建队列），则必须获取该命名空间的管理凭据。 可以从 [Azure 经典门户][]中获取这些凭据。

### <a name="to-obtain-management-credentials-from-the-portal"></a>从门户中获取管理凭据

1.  在左侧导航窗格中，单击“服务总线”节点以显示可用命名空间的列表  ：   
    ![](./media/howto-service-bus-queues/sb-queues-13.png)

2.  从显示的列表中选择刚刚创建的命名空间：   
    ![](./media/howto-service-bus-queues/sb-queues-09.png)

3.  单击“连接信息”  。   
    ![](./media/howto-service-bus-queues/sb-queues-06.png)

4.  在“访问连接信息”窗格中，找到包含 SAS 密钥和密钥名称的连接字符串  。   

    ![](./media/howto-service-bus-queues/multi-web-45.png)

5.  记下该密钥或将其复制到剪贴板。

  [Azure 经典门户]: http://manage.windowsazure.cn