---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 07/17/2020
ms.author: v-junlch
ms.openlocfilehash: 91f759bc84f55f07e5cccd1fcecd6b301594c8c9
ms.sourcegitcommit: 403db9004b6e9390f7fd1afddd9e164e5d9cce6a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/17/2020
ms.locfileid: "86441256"
---
## <a name="configure-your-local-environment"></a>配置本地环境

在开始之前，必须满足以下条件：

+ 具有活动订阅的 Azure 帐户。 [创建帐户](https://www.azure.cn/pricing/1rmb-trial/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell"  
+ [Azure Functions Core Tools](../articles/azure-functions/functions-run-local.md#v2) 2.7.1846 或更高的 2.x 版本。
::: zone-end  

+ [Azure CLI](/cli/install-azure-cli) 2.4 或更高版本。 
::: zone pivot="programming-language-javascript,programming-language-typescript"
+ [Node.js](https://nodejs.org/)，活动 LTS 和维护 LTS 版本（建议使用 8.11.1 和 10.14.1）。
::: zone-end

::: zone pivot="programming-language-powershell"
+ [PowerShell Core](https://docs.microsoft.com/powershell/scripting/install/installing-powershell-core-on-windows)

+ [.NET Core SDK 2.2+](https://www.microsoft.com/net/download)
::: zone-end
::: zone pivot="programming-language-java"  
+ [Java 开发人员工具包](https://aka.ms/azure-jdks)版本 8。

+ [Apache Maven](https://maven.apache.org) 3.0 或更高版本。

> [!IMPORTANT]
> JAVA_HOME 环境变量必须设置为 JDK 的安装位置，以完成本快速入门。
::: zone-end

