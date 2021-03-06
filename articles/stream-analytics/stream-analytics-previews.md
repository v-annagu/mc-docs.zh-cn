---
title: Azure 流分析预览功能
description: 本文列出了当前以预览版提供的 Azure 流分析功能。
author: Johnnytechn
ms.author: v-johya
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 07/06/2020
ms.openlocfilehash: 4d7d7b2a518e008a4c30bb20f95ff0817fac352b
ms.sourcegitcommit: 9bc3e55f01e0999f05e7b4ebaea95f3ac91d32eb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/10/2020
ms.locfileid: "86226165"
---
# <a name="azure-stream-analytics-preview-features"></a>Azure 流分析预览功能

本文汇总了当前以预览版提供的 Azure 流分析的所有功能。 建议不要在生产环境中使用预览功能。

## <a name="public-previews"></a>公共预览版

以下功能以公共预览版提供。 现在可以使用这些功能，但请勿在生产环境中使用它们。

### <a name="online-scaling"></a>联机缩放

通过联机缩放，如果需要更改 SU 分配，无需停止作业。 可以增加或减少正在运行的作业的 SU 容量，而无需将其停止。 这基于客户对流分析目前提供的长时间运行的任务关键管道的承诺。 有关详细信息，请参阅[配置 Azure 流分析流式处理单元](stream-analytics-streaming-unit-consumption.md#configure-stream-analytics-streaming-units-sus)。

### <a name="debug-query-steps-in-visual-studio"></a>在 Visual Studio 中调试查询步骤

在适用于 Visual Studio 的 Azure 流分析工具中执行本地测试时，可以轻松预览数据关系图上的中间行集。 

### <a name="local-testing-with-live-data-in-visual-studio-code"></a>在 Visual Studio Code 中使用实时数据进行本地测试

将作业提交到 Azure 之前，可以针对本地计算机上的实时数据测试查询。 每个测试迭代平均花费不到 2 到 3 秒，这会使开发过程非常高效。

### <a name="visual-studio-code-for-azure-stream-analytics"></a>适用于 Azure 流分析的 Visual Studio Code

可以在 Visual Studio Code 中创建 Azure 流分析作业。 请参阅我们的 [VS Code 入门教程](/stream-analytics/quick-create-vs-code)。


### <a name="real-time-high-performance-scoring-with-custom-ml-models-managed-by-azure-machine-learning"></a>通过 Azure 机器学习管理的自定义 ML 模型进行实时高性能评分

Azure 流分析通过利用自定义预先训练的机器学习模型（由 Azure 机器学习管理，并在 Azure Kubernetes 服务 (AKS) 或 Azure 容器实例 (ACI) 中托管），并使用不需要编写代码的工作流，来支持高性能实时评分。 [注册](https://aka.ms/asapreview1)即可获取预览版


### <a name="live-data-testing-in-visual-studio"></a>Visual Studio 中的实时数据测试

适用于 Azure 流分析的 Visual Studio 工具增强了本地测试功能，通过该功能可针对来自云源（如事件中心或 IoT 中心）的实时事件流测试查询。 了解如何[使用适用于 Visual Studio 的 Azure 流分析工具在本地测试实时数据](stream-analytics-live-data-local-testing.md)。


<!--.NET user-defined-function is not available in China.-->
## <a name="other-previews"></a>其他预览

如若请求，还可以使用以下预览功能。

### <a name="support-for-azure-stack"></a>支持 Azure Stack
此功能在 Azure IoT Edge 运行时启用，利用自定义 Azure Stack 功能，比如以原生方式支持在 Azure Stack 上运行的本地输入和输出（例如事件中心、IoT 中心、Blob 存储）。 利用这一新的集成，你可以构建混合体系结构，该体系结构可以在数据生成的位置附近分析数据，从而降低延迟并最大程度增加见解。
可以通过此功能构建混合体系结构，该体系结构可以在数据生成的位置附近分析数据，从而降低延迟并最大程度增加见解。 必须[注册](https://aka.ms/asapreview1)才能获取此预览版。
