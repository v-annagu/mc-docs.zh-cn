---
title: 针对完全相同的输入设计 Azure Functions
description: 将 Azure Functions 构建为幂等
author: craigshoemaker
ms.author: v-junlch
origin.date: 09/12/2019
ms.date: 09/24/2019
ms.topic: article
ms.service: azure-functions
manager: gwallace
ms.openlocfilehash: 827094befd6b1e2db65932002c93a388f3396d03
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "71673605"
---
# <a name="designing-azure-functions-for-identical-input"></a>针对完全相同的输入设计 Azure Functions

事件驱动型和基于消息的体系结构这一现实要求在确保数据完整性和系统稳定性的同时接受完全相同的请求。

让我们以电梯调用按钮来说明这一点。 按按钮时，按钮会亮起，电梯会送到你所在的楼层。 过了一会儿，另外一个人来到你所在的大厅。 该人对你微笑了一下，然后将亮起的按钮又按了一次。 你也对该人报之以微笑，忽然想到，调用电梯的命令是幂等的。

按电梯调用按钮两次、三次或四次对最终结果没有影响。 当你按按钮时，不管按多少次，电梯都会送到你的楼层。 对于电梯这样的幂等系统，不管发出相同的命令多少次，结果都一样。

在构建应用程序时，请考虑以下场景：

- 如果库存控制应用程序尝试删除同一产品多次，会发生什么情况？
- 如果有多个要求为同一人创建员工记录的请求，人力资源应用程序会如何表现？
- 如果银行应用收到 100 个要求进行相同金额提款的请求，则资金会转到哪里？

在许多情况下，对同一函数的请求可能会收到相同的命令。 一些情况包括：

- 将同一请求发送许多次的重试策略
- 缓存对应用程序重播的命令
- 发送多个相同请求时出现应用程序错误

为了保护数据完整性和系统运行状况，幂等应用程序包含的逻辑可能会包含以下行为：

- 在尝试执行删除操作之前，验证数据是否存在
- 在尝试执行创建操作之前，查看数据是否已存在
- 协调在数据中形成最终一致性的逻辑
- 并发控制
- 重复内容检测
- 数据新鲜度验证
- 用于验证输入数据的保护逻辑

最终通过确保某些给定的操作可以执行且只执行一次来实现幂等性。

