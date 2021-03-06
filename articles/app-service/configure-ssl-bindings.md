---
title: 使用 TLS/SSL 绑定保护自定义 DNS
description: 通过证书创建 TLS/SSL 绑定，以便保护对自定义域进行的 HTTPS 访问。 通过强制实施 HTTPS 或 TLS 1.2 提高网站的安全性。
tags: buy-ssl-certificates
ms.topic: tutorial
origin.date: 04/30/2020
ms.date: 05/22/2020
ms.author: v-tawe
ms.reviewer: yutlin
ms.custom: seodec18
ms.openlocfilehash: baf45ce63c1fa4e05db50ffcc975b97c33906100
ms.sourcegitcommit: 981a75a78f8cf74ab5a76f9e6b0dc5978387be4b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/22/2020
ms.locfileid: "83801126"
---
# <a name="secure-a-custom-dns-name-with-a-tlsssl-binding-in-azure-app-service"></a>在 Azure 应用服务中使用 TLS/SSL 绑定保护自定义 DNS 名称

本文介绍如何通过创建证书绑定来确保[应用服务应用](https://docs.azure.cn/app-service/)或[函数应用](https://docs.azure.cn/azure-functions/)中[自定义域](app-service-web-tutorial-custom-domain.md)的安全。 完成后，可访问自定义 DNS 名称（例如，`https://www.contoso.com`）的 `https://` 终结点处的应用服务应用。 

![带有自定义 TLS/SSL 证书的 Web 应用](./media/configure-ssl-bindings/app-with-custom-ssl.png)

使用证书来确保[自定义域](app-service-web-tutorial-custom-domain.md)的安全涉及两个步骤：

- [将专用证书添加到应用服务](configure-ssl-certificate.md)，以满足所有[专用证书要求](configure-ssl-certificate.md#private-certificate-requirements)。
-  创建相应自定义域的 TLS 绑定。 本文介绍第二步。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 升级应用的定价层
> * 使用证书确保自定义域的安全
> * 实施 HTTPS
> * 强制实施 TLS 1.1/1.2
> * 使用脚本自动完成 TLS 管理

## <a name="prerequisites"></a>先决条件

按照本操作方法指南操作：

- [创建应用服务应用](/app-service/)
- [将域名映射到应用](app-service-web-tutorial-custom-domain.md)
- [将专用证书添加到应用](configure-ssl-certificate.md)

<!-- > [!NOTE] -->
<!-- > The easiest way to add a private certificate is to [create a free App Service Managed Certificate](configure-ssl-certificate.md#create-a-free-certificate-preview) (Preview). -->

[!INCLUDE [Prepare your web app](../../includes/app-service-ssl-prepare-app.md)]

<a name="upload"></a>

## <a name="secure-a-custom-domain"></a>确保自定义域的安全

执行以下步骤：

在 <a href="https://portal.azure.cn" target="_blank">Azure 门户</a>的左侧菜单中，选择“应用程序服务” > “\<app-name>” 。

在应用的左侧导航窗格中，通过以下方式启动“TLS/SSL 绑定”对话框：

- 选择“自定义域” > “添加绑定” 
- 选择“TLS/SSL 设置” > “添加 TLS/SSL 绑定” 

![为域添加绑定](./media/configure-ssl-bindings/secure-domain-launch.png)

在“自定义域”中，选择要添加绑定的自定义域。

如果应用已具有所选自定义域的证书，请直接转到[创建绑定](#create-binding)。 反之，请继续操作。

### <a name="add-a-certificate-for-custom-domain"></a>为自定义域添加证书

如果应用不具有所选自定义域的证书，则有以下两种选择：

- **上传 PFX 证书** - 遵循[上传专用证书](configure-ssl-certificate.md#upload-a-private-certificate)中的工作流，然后在此处选择此选项。

<!-- - **Import App Service Certificate** - Follow the workflow at [Import an App Service certificate](configure-ssl-certificate.md#import-an-app-service-certificate), then select this option here. -->

<!-- > [!NOTE] -->
<!-- > You can also [Create a free certificate](configure-ssl-certificate.md#create-a-free-certificate-preview) (Preview) or [Import a Key Vault certificate](configure-ssl-certificate.md#import-a-certificate-from-key-vault), but you must do it separately and then return to the **TLS/SSL Binding** dialog. -->

### <a name="create-binding"></a>创建绑定

根据下表的要求在“TLS/SSL 绑定”对话框中配置 TLS 绑定，然后单击“添加绑定” 。

| 设置 | 说明 |
|-|-|
| 自定义域 | 要为其添加 TLS/SSL 绑定的域名。 |
| 私有证书指纹 | 要绑定的证书。 |
| TLS/SSL 类型 | <ul><li>SNI SSL - 可添加多个 SNI SSL 绑定。 选择此选项可以使用多个 SSL 证书来保护同一 IP 地址上的多个域。 大多数新式浏览器（包括 Internet Explorer、Chrome、Firefox 和 Opera）都支持 SNI。</li><li>**IP SSL** - 只能添加一个 IP SSL 绑定。 选择此选项只能使用一个 SSL 证书来保护专用公共 IP 地址。 配置绑定后，请按照[重新映射 IP SSL 的 A 记录](#remap-a-record-for-ip-ssl)中的步骤进行操作。<br/>仅生产或隔离层中支持 IP SSL。 </li></ul> |

操作完成之后，自定义域的 TLS/SSL 状态会更改为“安全”。

![TLS/SSL 绑定成功](./media/configure-ssl-bindings/secure-domain-finished.png)

> [!NOTE]
> “自定义域”中的状态为“安全”意味着已使用证书保护该域，但应用服务并未检查该证书是自签名证书还是已过期证书，这可能也会导致浏览器异常，例如显示错误或警告。 

## <a name="remap-records-for-ip-ssl"></a>重新映射 IP SSL 的记录

如果不在应用中使用 IP SSL，请跳到[针对自定义域测试 HTTPS](#test-https)。

可能需要进行两项更改：

- 默认情况下，应用使用共享的公共 IP 地址。 将证书与 IP SSL 绑定时，应用服务会为应用创建新的专用 IP 地址。 如果已将 A 记录映射到应用，请使用这个新的专用 IP 地址更新域注册表。

    将使用新的专用 IP 地址更新应用的“自定义域”页。 [复制此 IP 地址](app-service-web-tutorial-custom-domain.md#info)，然后[将 A 记录重新映射](app-service-web-tutorial-custom-domain.md#map-an-a-record)到此新 IP 地址。

- 如果已有到 `<app-name>.chinacloudsites.cn` 的 SNI SSL 绑定，请[重新映射任何 CNAME 映射](app-service-web-tutorial-custom-domain.md#map-a-cname-record)，让其改为指向 `sni.<app-name>.chinacloudsites.cn`（添加 `sni` 前缀）。

## <a name="test-https"></a>测试 HTTPS

在不同的浏览器中，导航到 `https://<your.custom.domain>` 以核实其是否适合应用。

![在门户中导航到 Azure 应用](./media/configure-ssl-bindings/app-with-custom-ssl.png)

应用程序代码可以通过“x-appservice-proto”标头检查协议。 该标头的值将为 `http` 或 `https`。 

> [!NOTE]
> 如果应用显示证书验证错误，可能是因为使用自签名证书。
>
> 如果不是这样，可能是在将证书导出为 PFX 文件时遗漏了中间证书。

## <a name="prevent-ip-changes"></a>防止 IP 更改

在删除某个绑定时，即使该绑定是 IP SSL，入站 IP 地址也可能会更改。 在续订已进行 IP SSL 绑定的证书时，了解这一点尤为重要。 若要避免应用的 IP 地址更改，请按顺序执行以下步骤：

1. 上传新证书。
2. 将新证书绑定到所需的自定义域，不要删除旧证书。 此操作替换而不是删除旧的绑定。
3. 删除旧证书。 

## <a name="enforce-https"></a>实施 HTTPS

默认情况下，任何人都仍可使用 HTTP 访问应用。 可以将所有 HTTP 请求都重定向到 HTTPS 端口。

在应用页的左侧导航窗格中，选择“SSL 设置”。 然后，在“仅 HTTPS”中，选择“启用”。

![实施 HTTPS](./media/configure-ssl-bindings/enforce-https.png)

该操作完成后，将导航到指向应用的任一 HTTP URL。 例如：

- `http://<app_name>.chinacloudsites.cn`
- `http://contoso.com`
- `http://www.contoso.com`

## <a name="enforce-tls-versions"></a>强制实施 TLS 版本

应用默认情况下允许 TLS 1.2，这是行业标准（例如 PCI DSS）建议的 TLS 级别。 若要强制实施不同的 TLS 版本，请按照下列步骤操作：

在应用页的左侧导航窗格中，选择“SSL 设置”。 然后，在“TLS 版本”中，选择所需的最低 TLS 版本。 此设置仅控制入站调用。 

![强制实施 TLS 1.1 或 1.2](./media/configure-ssl-bindings/enforce-tls1-2.png)

该操作完成后，你的应用将拒绝使用更低 TLS 版本的所有连接。

## <a name="handle-tls-termination"></a>处理 TLS 终止

在应用服务中，TLS 终止在网络负载均衡器上发生，因此，所有 HTTPS 请求将以未加密的 HTTP 请求形式访问你的应用。 如果应用逻辑需要检查用户请求是否已加密，可以检查 `X-Forwarded-Proto` 标头。

## <a name="automate-with-scripts"></a>使用脚本自动执行

### <a name="azure-cli"></a>Azure CLI

```azurecli
#!/bin/bash

fqdn=<replace-with-www.{yourdomain}>
pfxPath=<replace-with-path-to-your-.PFX-file>
pfxPassword=<replace-with-your=.PFX-password>
resourceGroup=myResourceGroup
webappname=mywebapp$RANDOM

# Create a resource group.
az group create --location chinaeast --name $resourceGroup

# Create an App Service plan in Basic tier (minimum required by custom domains).
az appservice plan create --name $webappname --resource-group $resourceGroup --sku B1

# Create a web app.
az webapp create --name $webappname --resource-group $resourceGroup \
--plan $webappname

echo "Configure a CNAME record that maps $fqdn to $webappname.chinacloudsites.cn"
read -p "Press [Enter] key when ready ..."

# Before continuing, go to your DNS configuration UI for your custom domain and follow the 
# instructions at https://aka.ms/appservicecustomdns to configure a CNAME record for the 
# hostname "www" and point it your web app's default domain name.

# Map your prepared custom domain name to the web app.
az webapp config hostname add --webapp-name $webappname --resource-group $resourceGroup \
--hostname $fqdn

# Upload the SSL certificate and get the thumbprint.
thumbprint=$(az webapp config ssl upload --certificate-file $pfxPath \
--certificate-password $pfxPassword --name $webappname --resource-group $resourceGroup \
--query thumbprint --output tsv)

# Binds the uploaded SSL certificate to the web app.
az webapp config ssl bind --certificate-thumbprint $thumbprint --ssl-type SNI \
--name $webappname --resource-group $resourceGroup

echo "You can now browse to https://$fqdn"
```

### <a name="powershell"></a>PowerShell

```powershell
$fqdn="<Replace with your custom domain name>"
$pfxPath="<Replace with path to your .PFX file>"
$pfxPassword="<Replace with your .PFX password>"
$webappname="mywebapp$(Get-Random)"
$location="China East"

# Create a resource group.
New-AzResourceGroup -Name $webappname -Location $location

# Create an App Service plan in Free tier.
New-AzAppServicePlan -Name $webappname -Location $location `
-ResourceGroupName $webappname -Tier Free

# Create a web app.
New-AzWebApp -Name $webappname -Location $location -AppServicePlan $webappname `
-ResourceGroupName $webappname

Write-Host "Configure a CNAME record that maps $fqdn to $webappname.chinacloudsites.cn"
Read-Host "Press [Enter] key when ready ..."

# Before continuing, go to your DNS configuration UI for your custom domain and follow the 
# instructions at https://aka.ms/appservicecustomdns to configure a CNAME record for the 
# hostname "www" and point it your web app's default domain name.

# Upgrade App Service plan to Basic tier (minimum required by custom SSL certificates)
Set-AzAppServicePlan -Name $webappname -ResourceGroupName $webappname `
-Tier Basic

# Add a custom domain name to the web app. 
Set-AzWebApp -Name $webappname -ResourceGroupName $webappname `
-HostNames @($fqdn,"$webappname.chinacloudsites.cn")

# Upload and bind the SSL certificate to the web app.
New-AzWebAppSSLBinding -WebAppName $webappname -ResourceGroupName $webappname -Name $fqdn `
-CertificateFilePath $pfxPath -CertificatePassword $pfxPassword -SslState SniEnabled
```

## <a name="more-resources"></a>更多资源

* [在应用程序代码中使用 SSL 证书](configure-ssl-certificate-in-code.md)
* [常见问题解答：应用服务证书](https://docs.azure.cn/app-service/faq-configuration-and-management/)
