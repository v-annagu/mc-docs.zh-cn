---
title: 自定义用户界面
titleSuffix: Azure AD B2C
description: 了解如何自定义使用 Azure Active Directory B2C 的应用程序的用户界面。
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 06/28/2020
ms.author: v-junlch
ms.subservice: B2C
ms.openlocfilehash: c2d23b136138f1565bbc6872743fcb9b038543cd
ms.sourcegitcommit: 3a8a7d65d0791cdb6695fe6c2222a1971a19f745
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2020
ms.locfileid: "85516510"
---
# <a name="customize-the-user-interface-in-azure-active-directory-b2c"></a>自定义 Azure Active Directory B2C 中的用户界面

对 Azure Active Directory B2C (Azure AD B2C) 向客户显示的用户界面进行品牌设计和自定义有助于在应用程序中提供无缝的用户体验。 这些体验包括注册、登录、个人资料编辑和密码重置。 本文介绍用户流和自定义策略的用户界面 (UI) 自定义方法。

## <a name="ui-customization-in-different-scenarios"></a>不同方案中的 UI 自定义

可通过多种方式自定义用户在应用程序中体验到的 UI，每种方法适用于不同的方案。

### <a name="user-flows"></a>用户流

如果使用[用户流](user-flow-overview.md)，可以使用内置的页面布局模板或自己的 HTML 和 CSS 来更改用户流页面的外观。 本文稍后会介绍这两种方法。

使用 [Azure 门户](tutorial-customize-ui.md)配置用户流的 UI 自定义。

> [!TIP]
> 如果只想修改用户流页的横幅徽标、背景图像和背景色，可以试用本文稍后介绍的[公司品牌(预览)](#company-branding-preview) 功能。

### <a name="custom-policies"></a>自定义策略

如果使用[自定义策略](custom-policy-overview.md)在应用程序中提供注册或登录、密码重置或个人资料编辑，则可使用策略文件来自定义 UI。

如果需要根据客户的决策提供动态内容，可以使用可根据查询字符串中发送的参数动态更改页面内容的自定义策略。 例如，可以基于从 Web 或移动应用程序传递的参数，更改 Azure AD B2C 注册或登录页面上的背景图像。

### <a name="javascript"></a>Javascript

可以在[用户流](user-flow-javascript-overview.md)和[自定义策略](page-layout.md)中启用客户端 JavaScript 代码。

## <a name="page-layout-templates"></a>页面布局模板

用户流提供多个可供选择的内置模板，以便为用户体验页面提供专业的外观。 这些布局模板还可充当你自己的自定义操作的起点。

在左侧菜单中的“自定义”下选择“页面布局”，然后选择“模板”。  

![Azure 门户的用户流页中的模板选择下拉列表](./media/customize-ui-overview/template-selection.png)

接下来，从列表中选择一个模板。 下面是每个模板的登录页面示例：

| 海蓝 | 青灰 | 经典 |
|:-:|:-:|:-:|
|![注册登录页上呈现的“海蓝”模板的示例](./media/customize-ui-overview/template-ocean-blue.png)|![注册登录页面上呈现的“青灰”模板示例](./media/customize-ui-overview/template-slate-gray.png)|![注册登录页面上呈现的“经典”模板示例](./media/customize-ui-overview/template-classic.png)|

选择某个模板时，选定的布局将应用于用户流中的所有页面，并且每个页面的 URI 将显示在“自定义页面 URI”字段中。

## <a name="custom-html-and-css"></a>自定义 HTML 和 CSS

如果希望使用自定义的 HTML 和 CSS 设计自己的策略布局，则可以通过针对策略中存在的每个布局名称切换“使用自定义页面内容”开关来实现。 请按照以下有关自定义布局配置的说明进行操作：

Azure AD B2C 使用称作[跨源资源共享 (CORS)](https://www.w3.org/TR/cors/) 的方法在客户浏览器中运行代码。

在运行时，将从用户流或自定义策略中指定的 URL 加载内容。 用户体验中的每个页面从你为该页面指定的 URL 加载其内容。 从 URL 加载内容后，该内容将与 Azure AD B2C 插入的 HTML 片段合并，然后向客户显示页面。

在使用自己的 HTML 和 CSS 文件自定义 UI 之前，请查看以下指导：

- Azure AD B2C 将 HTML 内容**合并**到页面中。 请勿复制并尝试更改 Azure AD B2C 提供的默认内容。 最好从头开始构建 HTML5 内容，将默认内容用作参考。
- 可将 **JavaScript** 包含在[用户流](user-flow-javascript-overview.md)和[自定义策略](javascript-samples.md)的自定义内容中。
- 支持的**浏览器版本**包括：
  - Internet Explorer 11、10 和 Microsoft Edge
  - 对 Internet Explorer 9 和 8 的支持有限
  - Google Chrome 42.0 和更高版本
  - Mozilla Firefox 38.0 和更高版本
  - 适用于 iOS 和 macOS 的 Safari 版本 12 及更高版本
- 不要在 HTML 中包含**窗体标记**。 窗体标记会干扰 Azure AD B2C 插入的 HTML 所生成的 POST 操作。

### <a name="where-do-i-store-ui-content"></a>在何处存储 UI 内容？

使用自己的 HTML 和 CSS 文件自定义 UI 时，可将 UI 内容托管在支持 CORS 的任何公用 HTTPS 终结点上。 例如，[Azure Blob 存储](../storage/blobs/storage-blobs-introduction.md)、Web 服务器、CDN、AWS S3 或文件共享系统。

重要的一点是，你将内容托管在一个[启用了 CORS](https://enable-cors.org/server.html) 且公开可用的 HTTPS 终结点上。 在内容中指定绝对 URL 时，必须使用绝对 URL。

## <a name="get-started-with-custom-html-and-css"></a>自定义 HTML 和 CSS 入门

请遵循以下指导原则在用户体验页面中开始使用自己的 HTML 和 CSS。

- 使用位于 `<body>` 中某处的空 `<div id="api"></div>` 元素创建格式正确的 HTML 内容。 此元素标记插入 Azure AD B2C 内容的位置。 以下示例显示了一个最小化页面：

    ```html
    <!DOCTYPE html>
    <html>
      <head>
        <title>!Add your title here!</title>
        <link rel="stylesheet" href="https://mystore1.blob.core.chinacloudapi.cn/b2c/style.css">
      </head>
      <body>
        <h1>My B2C Application</h1>
        <div id="api"></div>   <!-- Leave this element empty because Azure AD B2C will insert content here. -->
      </body>
    </html>
    ```

- 使用 CSS 设置 Azure AD B2C 插入页面的 UI 元素的样式。 以下示例显示了一个简单的 CSS 文件，其中还包含注册注入 HTML 元素的设置：

    ```css
    h1 {
      color: blue;
      text-align: center;
    }
    .intro h2 {
      text-align: center;
    }
    .entry {
      width: 400px ;
      margin-left: auto ;
      margin-right: auto ;
    }
    .divider h2 {
      text-align: center;
    }
    .create {
      width: 400px ;
      margin-left: auto ;
      margin-right: auto ;
    }
    ```

- 在 HTTPS 终结点上托管内容（允许 CORS）。 配置 CORS 时必须启用 GET 和 OPTIONS 请求方法。
- 创建或编辑用户流或自定义策略，以使用创建的内容。

### <a name="html-fragments-from-azure-ad-b2c"></a>Azure AD B2C 中的 HTML 片段

下表列出了 Azure AD B2C 合并到内容中 `<div id="api"></div>` 元素的 HTML 片段。

| 插入的页 | HTML 说明 |
| ------------- | ------------------- |
| 标识提供者选项 | 包含标识提供者的按钮列表，客户可在注册或登录时选择这些按钮。 这些按钮包括社交标识提供者。 |
| 本地帐户注册 | 包含一个窗体，用于基于电子邮件地址或用户名的本地帐户注册。 该窗体可以包含不同的输入控件，如文本输入框、密码输入框、单选按钮、单选下拉框和多选复选框。 |
| 社交帐户注册 | 使用社交标识提供者的现有帐户进行注册时，可能会显示此页面。 在必须使用注册表单收集客户的其他信息时使用此页面。 |
| 统一注册或登录 | 处理可以使用社交标识提供者的客户的注册和登录。 |
| 多重身份验证 | 用户可以在注册或登录期间（使用文字或语音）验证其电话号码。 |
| 错误 | 向客户提供错误信息。 |

## <a name="localize-content"></a>本地化内容

可通过在 Azure AD B2C 租户中启用[语言自定义](user-flow-language-customization.md)来本地化 HTML 内容。 启用此功能可让 Azure AD B2C 将 OpenID Connect 参数 `ui_locales` 转发到终结点。 内容服务器可使用此参数提供特定语言的 HTML 页。

可以基于所用的区域设置从不同位置拉取内容。 在已启用 CORS 的终结点中，可以设置文件夹结构以托管特定语言的内容。 如果使用通配符值 `{Culture:RFC5646}`，则会调用正确的语言。

例如，自定义的页面 URI 可能类似于以下内容：

```HTTP
https://contoso.blob.core.chinacloudapi.cn/{Culture:RFC5646}/myHTML/unified.html
```

可以通过从以下位置提取内容来加载法语页面：

```HTTP
https://contoso.blob.core.chinacloudapi.cn/fr/myHTML/unified.html
```

## <a name="examples"></a>示例

可以在 GitHub 上的 [B2C-AzureBlobStorage-Client](https://github.com/azureadquickstarts/b2c-azureblobstorage-client) 存储库中找到多个示例模板文件。

模板中的示例 HTML 和 CSS 文件位于 [/sample_templates](https://github.com/AzureADQuickStarts/B2C-AzureBlobStorage-Client/tree/master/sample_templates) 目录中。

## <a name="next-steps"></a>后续步骤

- 如果使用**用户流**，可以根据教程开始自定义 UI：

    [在 Azure Active Directory B2C 中自定义应用程序的用户界面](tutorial-customize-ui.md)。
<!-- Update_Description: wording update -->
