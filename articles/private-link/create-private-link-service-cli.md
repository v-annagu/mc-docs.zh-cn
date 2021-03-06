---
title: 使用 Azure CLI 创建 Azure 专用链接服务
description: 了解如何使用 Azure CLI 创建 Azure 专用链接服务
services: private-link
author: rockboyfor
ms.service: private-link
ms.topic: article
origin.date: 09/16/2019
ms.date: 06/15/2020
ms.author: v-yeche
ms.openlocfilehash: 3c121d2ed4d27b2d98cf8705efca9ce1d6fbf81d
ms.sourcegitcommit: 3de7d92ac955272fd140ec47b3a0a7b1e287ca14
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/12/2020
ms.locfileid: "84723705"
---
# <a name="create-a-private-link-service-using-azure-cli"></a>使用 Azure CLI 创建专用链接服务
本文介绍了如何使用 Azure CLI 在 Azure 中创建专用链接服务。

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

如果你决定在本地安装并使用 Azure CLI，本快速入门要求使用最新 Azure CLI 版本。 若要查找已安装的版本，请运行 `az --version`。 有关安装或升级信息，请参阅[安装 Azure CLI](https://docs.azure.cn/cli/install-azure-cli?view=azure-cli-latest)。
## <a name="create-a-private-link-service"></a>创建专用链接服务
### <a name="create-a-resource-group"></a>创建资源组

在创建虚拟网络之前，必须创建一个资源组用于托管该虚拟网络。 使用 [az group create](https://docs.azure.cn/cli/group?view=azure-cli-latest#az-group-create) 创建资源组。 此示例在 *chinaeast2* 位置创建一个名为 *myResourceGroup* 的资源组：

```azurecli
az group create --name myResourceGroup --location chinaeast2
```
### <a name="create-a-virtual-network"></a>创建虚拟网络
使用 [az network vnet create](https://docs.azure.cn/cli/network/vnet?view=azure-cli-latest#az-network-vnet-create) 创建虚拟网络。 此示例创建名为 *myVirtualNetwork* 的默认虚拟网络，它具有一个名为 *mySubnet* 的子网：

```azurecli
az network vnet create --resource-group myResourceGroup --name myVirtualNetwork --address-prefix 10.0.0.0/16  
```
### <a name="create-a-subnet"></a>创建子网
使用 [az network vnet subnet create](https://docs.azure.cn/cli/network/vnet/subnet?view=azure-cli-latest#az-network-vnet-subnet-create) 为虚拟网络创建子网。 此示例在 *myVirtualNetwork* 虚拟网络中创建名为 *mySubnet* 的子网：

```azurecli
az network vnet subnet create --resource-group myResourceGroup --vnet-name myVirtualNetwork --name mySubnet --address-prefixes 10.0.0.0/24    
```
### <a name="create-a-internal-load-balancer"></a>创建内部负载均衡器 
使用 [az network lb create](https://docs.azure.cn/cli/network/lb?view=azure-cli-latest#az-network-lb-create) 创建内部负载均衡器。 此示例在名为 *myResourceGroup* 的资源组中创建名为 *myILB* 的内部负载均衡器。 

```azurecli
az network lb create --resource-group myResourceGroup --name myILB --sku standard --vnet-name MyVirtualNetwork --subnet mySubnet --frontend-ip-name myFrontEnd --backend-pool-name myBackEndPool
```

### <a name="create-a-load-balancer-health-probe"></a>创建负载均衡器运行状况探测

运行状况探测会检查所有虚拟机实例，以确保它们可以接收网络流量。 探测器检查失败的虚拟机实例将从负载均衡器中删除，直到它恢复联机状态并且探测器检查确定它运行正常。 使用 [az network lb probe create](https://docs.azure.cn/cli/network/lb/probe?view=azure-cli-latest#az-network-lb-probe-create) 创建运行状况探测，以监视虚拟机的运行状况。 

```azurecli
  az network lb probe create \
    --resource-group myResourceGroup \
    --lb-name myILB \
    --name myHealthProbe \
    --protocol tcp \
    --port 80   
```

### <a name="create-a-load-balancer-rule"></a>创建负载均衡器规则

负载均衡器规则定义传入流量的前端 IP 配置和用于接收流量的后端 IP 池，以及所需源和目标端口。 使用 [az network lb rule create](https://docs.azure.cn/cli/network/lb/rule?view=azure-cli-latest#az-network-lb-rule-create) 创建负载均衡器规则 *myHTTPRule*，以便侦听前端池 *myFrontEnd* 中的端口 80，并且将经过负载均衡的网络流量发送到也使用端口 80 的后端地址池 *myBackEndPool*。 

```azurecli
  az network lb rule create \
    --resource-group myResourceGroup \
    --lb-name myILB \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe  
```
### <a name="create-backend-servers"></a>创建后端服务器

在此示例中，我们未包括虚拟机创建。 可以按照[使用 Azure CLI 创建内部负载均衡器以对 VM 进行负载均衡](../load-balancer/load-balancer-get-started-ilb-arm-cli.md#create-servers-for-the-backend-address-pool)中的步骤创建两个虚拟机，以用作负载均衡器的后端服务器。 

### <a name="disable-private-link-service-network-policies-on-subnet"></a>在子网上禁用专用链接服务网络策略 
专用链接服务需要虚拟网络中你选择的任何子网中的一个 IP。 目前，这些 IP 不支持网络策略。  因此，我们必须禁用子网上的网络策略。 请使用 [az network vnet subnet update](https://docs.azure.cn/cli/network/vnet/subnet?view=azure-cli-latest#az-network-vnet-subnet-update) 更新子网以禁用专用链接服务网络策略。

```azurecli
az network vnet subnet update --resource-group myResourceGroup --vnet-name myVirtualNetwork --name mySubnet --disable-private-link-service-network-policies true 
```

## <a name="create-a-private-link-service"></a>创建专用链接服务  

通过 [az network private-link-service create](https://docs.azure.cn/cli/network/private-link-service?view=azure-cli-latest#az-network-private-link-service-create) 使用标准负载均衡器前端 IP 配置创建专用链接服务。 此示例使用 *myResourceGroup* 资源组中的标准负载均衡器 *myLoadBalancer* 创建名为 *myPLS* 的专用链接服务。 

```azurecli
az network private-link-service create \
--resource-group myResourceGroup \
--name myPLS \
--vnet-name myVirtualNetwork \
--subnet mySubnet \
--lb-name myILB \
--lb-frontend-ip-configs myFrontEnd \
--location chinaeast2
```
在创建后，记下专用链接服务 ID。 稍后，你将需要请求连接到此服务。  

在此阶段，专用链接服务已成功创建，并已准备好接收流量。 请注意，上面的示例仅演示了如何使用 Azure CLI 创建专用链接服务。  我们尚未配置负载均衡器后端池，也未在后端池上配置任何应用程序来侦听流量。 如果你想查看端到端流量流，则强烈建议你在标准负载均衡器后面配置应用程序。  

接下来，我们将演示如何使用 Azure CLI 将此服务映射到不同虚拟网络中的专用终结点。 同样，示例仅限于使用 Azure CLI 创建专用终结点并连接到上面创建的专用链接服务。 另外，你还可以在虚拟网络中创建虚拟机，来为专用终结点发送/接收流量。        

## <a name="private-endpoints"></a>专用终结点

### <a name="create-the-virtual-network"></a>创建虚拟网络 
使用  [az network vnet create](https://docs.azure.cn/cli/network/vnet?view=azure-cli-latest#az-network-vnet-create) 创建虚拟网络。 此示例在名为 *myResourcegroup* 的资源组中创建名为  *myPEVNet*  的虚拟网络： 
```azurecli
az network vnet create \
--resource-group myResourceGroup \
--name myPEVnet \
--address-prefix 10.0.0.0/16  
```
### <a name="create-the-subnet"></a>创建子网 
使用  [az network vnet subnet create](https://docs.azure.cn/cli/network/vnet/subnet?view=azure-cli-latest#az-network-vnet-subnet-create) 在虚拟网络中创建子网。 此示例在资源组 *myResourcegroup* 中的虚拟网络 *myPEVnet* 中创建名为  *mySubnet*  的子网： 

```azurecli 
az network vnet subnet create \
--resource-group myResourceGroup \
--vnet-name myPEVnet \
--name myPESubnet \
--address-prefixes 10.0.0.0/24 
```   
## <a name="disable-private-endpoint-network-policies-on-subnet"></a>在子网上禁用专用终结点网络策略 
可以在虚拟网络中你选择的任何子网中创建专用终结点。 当前，专用终结点不支持网络策略。  因此，我们必须禁用子网上的网络策略。 请使用 [az network vnet subnet update](https://docs.azure.cn/cli/network/vnet/subnet?view=azure-cli-latest#az-network-vnet-subnet-update) 更新子网以禁用专用终结点网络策略。 

```azurecli
az network vnet subnet update \
--resource-group myResourceGroup \
--vnet-name myPEVnet \
--name myPESubnet \
--disable-private-endpoint-network-policies true 
```
## <a name="create-private-endpoint-and-connect-to-private-link-service"></a>创建专用终结点并连接到专用链接服务 
创建一个专用终结点，用于使用上面在虚拟网络中创建的专用链接服务：

```azurecli
az network private-endpoint create \
--resource-group myResourceGroup \
--name myPE \
--vnet-name myPEVnet \
--subnet myPESubnet \
--private-connection-resource-id {PLS_resourceURI} \
--connection-name myPEConnectingPLS \
--location chinaeast2 
```
可以在专用链接服务上使用 `az network private-link-service show` 获取 private-connection-resource-id。 ID 将如下所示：   
/subscriptions/subID/resourceGroups/*resourcegroupname*/providers/Microsoft.Network/privateLinkServices/**privatelinkservicename** 

## <a name="show-private-link-service-connections"></a>显示专用链接服务连接 

使用 [az network private-link-service show](https://docs.azure.cn/cli/network/private-link-service?view=azure-cli-latest#az-network-private-link-service-show) 查看专用链接服务上的连接请求。    
```azurecli 
az network private-link-service show --resource-group myResourceGroup --name myPLS 
```
## <a name="next-steps"></a>后续步骤
- 详细了解 [Azure 专用链接服务](private-link-service-overview.md)

<!-- Update_Description: new article about create private link service cli -->
<!--NEW.date: 01/06/2020-->