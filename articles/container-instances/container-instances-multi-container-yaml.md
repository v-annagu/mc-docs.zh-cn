---
title: 教程 - 部署多容器组 - YAML
description: 本教程介绍如何将 YAML 文件和 Azure CLI 配合使用，以便在 Azure 容器实例中部署包含多个容器的容器组。
ms.topic: article
origin.date: 04/03/2019
ms.date: 01/15/2020
ms.author: v-yeche
ms.openlocfilehash: d1afed8a8d9be422513c23557526cc866c6c2334
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "77428824"
---
<!--Verified successfully-->
# <a name="tutorial-deploy-a-multi-container-group-using-a-yaml-file"></a>教程：使用 YAML 文件部署多容器组

> [!div class="op_single_selector"]
> * [YAML](container-instances-multi-container-yaml.md)
> * [Resource Manager](container-instances-multi-container-group.md)
>

Azure 容器实例支持使用[容器组](container-instances-container-groups.md)将多个容器部署到单台主机上。 当生成应用程序 sidecar 以用于日志记录、监视或用于某些其他配置（其中服务需要第二个附加进程）时，容器组很有用。

在本教程中，我们将按步骤运行简单的双容器挎斗配置，只需使用 Azure CLI 部署 [YAML 文件](container-instances-reference-yaml.md)即可。 YAML 文件提供了一种简单的格式来指定实例设置。 你将学习如何执行以下操作：

> [!div class="checklist"]
> * 配置 YAML 文件
> * 部署容器组
> * 查看容器的日志

> [!NOTE]
> 多容器组当前仅限于 Linux 容器。

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="configure-a-yaml-file"></a>配置 YAML 文件

若要在 Azure CLI 中使用 [az container create][az-container-create] 命令部署多容器组，必须在 YAML 文件中指定容器组配置。 然后，将 YAML 文件作为参数传递给命令。

首先将以下 YAML 复制到一个名为 **deploy-aci.yaml** 的新文件中。

<!--Not Available on  In Azure Cloud Shell, you can use Visual Studio Code to create the file in your working directory-->

```
code deploy-aci.yaml
```

此 YAML 文件定义了一个名为“myContainerGroup”的容器组，其中包含两个容器、一个公共 IP 地址和两个公开端口。 这些容器从公共 Azure 映像部署。 该组中的第一个容器运行面向 Internet 的 Web 应用程序。 第二个容器 sidecar 定期通过容器组的本地网络向在第一个容器中运行的 Web 应用程序发出 HTTP 请求。

```YAML
apiVersion: 2018-10-01
location: chinaeast2
name: myContainerGroup
properties:
  containers:
  - name: aci-tutorial-app
    properties:
      image: mcr.microsoft.com/azuredocs/aci-helloworld:latest
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
      ports:
      - port: 80
      - port: 8080
  - name: aci-tutorial-sidecar
    properties:
      image: mcr.microsoft.com/azuredocs/aci-tutorial-sidecar
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - protocol: tcp
      port: '80'
    - protocol: tcp
      port: '8080'
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

若要使用专用容器映像注册表，请将 `imageRegistryCredentials` 属性添加到容器组，其中包含针对环境修改的值：

```YAML
  imageRegistryCredentials:
  - server: imageRegistryLoginServer
    username: imageRegistryUsername
    password: imageRegistryPassword
```

## <a name="deploy-the-container-group"></a>部署容器组

使用 [az group create][az-group-create] 命令创建资源组：

```azurecli
az group create --name myResourceGroup --location chinaeast2
```

使用 [az container create][az-container-create] 命令部署容器组并传递 YAML 文件作为参数：

```azurecli
az container create --resource-group myResourceGroup --file deploy-aci.yaml
```

将在几秒钟内收到来自 Azure 的初始响应。

## <a name="view-deployment-state"></a>查看部署状态

若要查看部署状态，请运行下面的 [az container show][az-container-show] 命令：

```azurecli
az container show --resource-group myResourceGroup --name myContainerGroup --output table
```

若要查看正在运行的应用程序，请在浏览器中转到它的 IP 地址。 例如，在此示例输出中，IP 为 `52.168.26.124`：

```bash
Name              ResourceGroup    Status    Image                                                                                               IP:ports              Network    CPU/Memory       OsType    Location
----------------  ---------------  --------  --------------------------------------------------------------------------------------------------  --------------------  ---------  ---------------  --------  ----------
myContainerGroup  danlep0318r      Running   mcr.microsoft.com/azuredocs/aci-tutorial-sidecar,mcr.microsoft.com/azuredocs/aci-helloworld:latest  20.42.26.114:80,8080  Public     1.0 core/1.5 gb  Linux     chinaeast2
```

## <a name="view-container-logs"></a>查看容器日志

使用 [az container logs][az-container-logs] 命令查看容器的日志输出。 `--container-name` 参数指定从中拉取日志的容器。 在此示例中，指定了 `aci-tutorial-app` 容器。

```azurecli
az container logs --resource-group myResourceGroup --name myContainerGroup --container-name aci-tutorial-app
```

输出：

```console
listening on port 80
::1 - - [21/Mar/2019:23:17:48 +0000] "HEAD / HTTP/1.1" 200 1663 "-" "curl/7.54.0"
::1 - - [21/Mar/2019:23:17:51 +0000] "HEAD / HTTP/1.1" 200 1663 "-" "curl/7.54.0"
::1 - - [21/Mar/2019:23:17:54 +0000] "HEAD / HTTP/1.1" 200 1663 "-" "curl/7.54.0"
```

若要查看 sidecar 容器的日志，请运行一个类似的命令，指定 `aci-tutorial-sidecar` 容器。

```azurecli
az container logs --resource-group myResourceGroup --name myContainerGroup --container-name aci-tutorial-sidecar
```

输出：

```console
Every 3s: curl -I http://localhost                          2019-03-21 20:36:41

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0  1663    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
HTTP/1.1 200 OK
X-Powered-By: Express
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Wed, 29 Nov 2017 06:40:40 GMT
ETag: W/"67f-16006818640"
Content-Type: text/html; charset=UTF-8
Content-Length: 1663
Date: Thu, 21 Mar 2019 20:36:41 GMT
Connection: keep-alive
```

如你所见，sidecar 通过该组的本地网络定期向主 Web 应用程序发出 HTTP 请求，确保其正在运行。 如果此 sidecar 示例收到的 HTTP 响应代码不是 `200 OK`，则可将其扩展以触发警报。

## <a name="next-steps"></a>后续步骤

在本教程中，你使用了 YAML 文件在 Azure 容器实例中部署多容器组。 你已了解如何：

> [!div class="checklist"]
> * 为多容器组配置 YAML 文件
> * 部署容器组
> * 查看容器的日志

也可使用[资源管理器模板](container-instances-multi-container-group.md)指定多容器组。 需要使用容器组部署其他 Azure 服务资源时，可以轻松地针对方案改编资源管理器模板。

<!-- LINKS - External -->

<!-- LINKS - Internal -->

[aci-tutorial]: ./container-instances-tutorial-prepare-app.md
[az-container-create]: https://docs.microsoft.com/cli/azure/container?view=azure-cli-latest#az-container-create
[az-container-logs]: https://docs.microsoft.com/cli/azure/container?view=azure-cli-latest#az-container-logs
[az-container-show]: https://docs.microsoft.com/cli/azure/container?view=azure-cli-latest#az-container-show
[az-group-create]: https://docs.azure.cn/cli/group?view=azure-cli-latest#az-group-create
[az-group-deployment-create]: https://docs.azure.cn/cli/group/deployment?view=azure-cli-latest#az-group-deployment-create

<!--Not Available on [template-reference]: /templates/microsoft.containerinstance/containergroups-->

<!-- Update_Description: new article about container instances multi container yaml -->
<!--NEW.date: 01/15/2020-->