---
ms.openlocfilehash: 95bd333d0823c18cd31d94a62f22483718b851c7
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "78266139"
---
::: zone pivot="programming-language-csharp"  
## <a name="register-binding-extensions"></a>注册绑定扩展

除了 HTTP 和计时器触发器，绑定将实现为扩展包。 在终端窗口中运行以下 [dotnet add package](https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package) 命令，将存储扩展包添加到项目中。

```bash
dotnet add package Microsoft.Azure.WebJobs.Extensions.Storage --version 3.0.4
```

现在，你可以将存储输出绑定添加到项目。  
::: zone-end  

