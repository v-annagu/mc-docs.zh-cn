---
ms.openlocfilehash: 638efeea4ffe5bacb23ed6ecc0b101ed8f32cb3b
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "63827089"
---
请确保你已创建服务总线命名空间，如[此处][namespace-how-to]所示。

1. 登录到 [Azure 门户][azure-portal]。
2. 在门户左侧的导航窗格中，单击“服务总线”（如果未看到“服务总线”，请单击“所有服务”）。   
3. 单击要在其中创建队列的命名空间。 在此示例中，它是“sbnstest1”  。
   
    ![创建队列][createqueue1]
4. 在命名空间窗口中单击“队列”，然后在“队列”窗口中单击“+ 队列”。   
   
    ![选择“队列”][createqueue2]
5. 输入队列**名称**，其他值则保留默认值。
   
    ![选择“新建”][createqueue3]
6. 在窗口底部，单击“创建”。 

[createqueue1]: ./media/service-bus-create-queue-portal/create-queue1.png
[createqueue2]: ./media/service-bus-create-queue-portal/create-queue2.png
[createqueue3]: ./media/service-bus-create-queue-portal/create-queue3.png

[namespace-how-to]: ../articles/service-bus-messaging/service-bus-create-namespace-portal.md
[azure-portal]: https://portal.azure.cn