---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
origin.date: 03/27/2020
ms.date: 04/20/2020
ms.author: v-tawe
ms.openlocfilehash: 239670f0394eaaaaf5f471d3e8ee650369229cfc
ms.sourcegitcommit: 89ca2993f5978cd6dd67195db7c4bdd51a677371
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2020
ms.locfileid: "82595258"
---
:::row:::
    :::column span="3":::
        JavaScript 语音 SDK 以 npm 包的形式提供（请参阅 <a href="https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk" target="_blank">microsoft-cognitiveservices-speech-sdk <span class="docon docon-navigate-external x-hidden-focus"></span></a>），并与 GitHub 存储库 <a href="https://github.com/Microsoft/cognitive-services-speech-sdk-js" target="_blank">cognitive-services-speech-sdk-js <span class="docon docon-navigate-external x-hidden-focus"></span></a> 配套。
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="JavaScript" src="https://docs.microsoft.com/media/logos/logo_js.svg"  width="60px">
        </div>
    :::column-end:::
:::row-end:::

> [!TIP]
> 尽管 JavaScript 语音 SDK 以 npm 包的形式提供，但是客户端 Web 浏览器和 Node.js 都可以使用它 - 请考虑每种环境的各种体系结构隐含内容。 例如，文档对象模型 (DOM) 不可用于服务器端应用程序，就像<a href="https://nodejs.org/api/fs.html" target="_blank">文件系统<span class="docon docon-navigate-external x-hidden-focus"></span></a>不可用于客户端应用程序一样。

### <a name="nodejs-package-manager-npm"></a>Node.js 包管理器 (NPM)

若要安装 JavaScript 语音 SDK，请运行以下 `npm install` 命令。

```nodejs
npm install microsoft-cognitiveservices-speech-sdk
```

### <a name="html-script-tag"></a>HTML 脚本标记

或者，可以直接在 HTML `<head>` 元素中包含一个 `<script>` 标记，该标记依赖于 <a href="https://www.jsdelivr.com/package/npm/microsoft-cognitiveservices-speech-sdk" target="_blank">JSDelivr  NPM syndicate <span class="docon docon-navigate-external x-hidden-focus"></span></a>。

```html
<script src="https://cdn.jsdelivr.net/npm/microsoft-cognitiveservices-speech-sdk@1.10.1/distrib/lib/microsoft.cognitiveservices.speech.sdk.min.js">
</script>
```

有关详细信息，请参阅 <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/browser" target="_blank">Web 浏览器语音 SDK 快速入门<span class="docon docon-navigate-external x-hidden-focus"></span></a>。
