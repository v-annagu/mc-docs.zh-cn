---
title: include 文件
description: include 文件
ms.topic: include
ms.date: 06/19/2020
ms.author: v-tawe
origin.date: 01/23/2020
ms.openlocfilehash: 9dd3f51660bfcedfbcabdfdd4d402d0d92049d3d
ms.sourcegitcommit: 48b5ae0164f278f2fff626ee60db86802837b0b4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2020
ms.locfileid: "85102035"
---
### <a name="create-an-environment-variable"></a>创建环境变量

使用运行时密钥和运行时终结点，创建两个用于身份验证和访问的环境变量：

* `LUIS_RUNTIME_KEY` - 用于验证请求的运行时资源密钥。
* `LUIS_RUNTIME_ENDPOINT` - 与密钥关联的运行时终结点。
* `LUIS_APP_ID` - 公共 LUIS IoT 应用 ID 为 `df67dcdb-c37d-46af-88e1-8b97951ca1c2`。
* `LUIS_APP_SLOT_NAME` - `production` 或 `staging`

若要根据本快速入门访问自己的应用，则需执行其他步骤：
* 创建应用并获取应用 ID
* 将运行时密钥分配给 LUIS 门户中的应用
* 使用浏览器测试 URL，看是否可以访问应用

使用操作系统的说明。

#### <a name="windows"></a>[Windows](#tab/windows)

```console
setx LUIS_RUNTIME_KEY <replace-with-your-resource-key>
setx LUIS_RUNTIME_ENDPOINT <replace-with-your-resource-endpoint>
setx LUIS_APP_ID <replace-with-your-app-id>
setx LUIS_APP_SLOT_NAME <replace-with-production-or-staging>
```

添加环境变量后，请重启控制台窗口。

#### <a name="linux"></a>[Linux](#tab/linux)

```bash
export LUIS_RUNTIME_KEY= <replace-with-your-resource-key>
export LUIS_RUNTIME_ENDPOINT= <replace-with-your-resource-endpoint>
export LUIS_APP_ID= <replace-with-your-app-id>
export LUIS_APP_SLOT_NAME= <replace-with-production-or-staging>
```

添加环境变量后，请从控制台窗口运行 `source ~/.bashrc`，使更改生效。

#### <a name="macos"></a>[macOS](#tab/unix)

编辑 `.bash_profile`，然后添加环境变量：

```bash
export LUIS_RUNTIME_KEY= <replace-with-your-resource-key>
export LUIS_RUNTIME_ENDPOINT= <replace-with-your-resource-endpoint>
export LUIS_APP_ID= <replace-with-your-app-id>
export LUIS_APP_SLOT_NAME= <replace-with-production-or-staging>
```

添加环境变量后，请从控制台窗口运行 `source .bash_profile`，使更改生效。

***
