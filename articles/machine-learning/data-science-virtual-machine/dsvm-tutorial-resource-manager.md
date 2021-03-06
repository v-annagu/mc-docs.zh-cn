---
title: 快速入门：创建 Data Science VM - 资源管理器模板
titleSuffix: Azure Data Science Virtual Machine
description: 在本快速入门中，你将使用 Azure 资源管理器模板快速部署 Data Science Virtual Machine
services: machine-learning
author: lobrien
ms.author: laobri
ms.custom: subject-armqs
ms.date: 06/10/2020
ms.service: machine-learning
ms.subservice: data-science-vm
ms.topic: quickstart
ms.openlocfilehash: c3951d2bf287b7c7c55a1d71efcbc9e33c4b80a0
ms.sourcegitcommit: 1c01c98a2a42a7555d756569101a85e3245732fd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2020
ms.locfileid: "85097793"
---
# <a name="quickstart-create-an-ubuntu-data-science-virtual-machine-using-a-resource-manager-template"></a>快速入门：使用资源管理器模板创建 Ubuntu Data Science Virtual Machine
[!INCLUDE [applies-to-skus](../../../includes/aml-applies-to-basic-enterprise-sku.md)]

本快速入门将介绍如何使用 Azure 资源管理器模板创建 Ubuntu 18.04 Data Science Virtual Machine。 Data Science Virtual Machine 是基于云的虚拟机，预加载了一套数据科学和机器学习框架及工具。 当部署在 GPU 驱动的计算资源上时，所有工具和库都配置为使用 GPU。 

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

## <a name="prerequisites"></a>先决条件

* Azure 订阅。 如果没有 Azure 订阅，请在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)

* 若要从本地环境使用本文档中的 CLI 命令，需要使用 [Azure CLI](/cli/install-azure-cli?view=azure-cli-latest)

## <a name="create-a-workspace"></a>创建工作区

### <a name="review-the-template"></a>查看模板

本快速入门中使用的模板来自 [Azure 快速启动模板](https://azure.microsoft.com/resources/templates/101-vm-ubuntu-DSVM-GPU-or-CPU/)。 本文的完整模板太长，无法在此处显示。 若要查看完整模板，请参阅 [azuredeploy.json](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-ubuntu-DSVM-GPU-or-CPU/azuredeploy.json)。 定义 DSVM 细节的部分如下所示：

```json
    {

      "type": "Microsoft.Compute/virtualMachines",

      "apiVersion": "2019-03-01",

      "name": "[concat(variables('virtualMachineName'),'-', parameters('Cpu-Gpu'))]",

      "location": "[parameters('location')]",

      "dependsOn": [

        "[resourceId('Microsoft.Network/networkInterfaces/', variables('networkInterfaceName'))]",

        "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]"

      ],

      "properties": {

        "hardwareProfile": {

          "vmSize": "[variables('vmSize')[parameters('Cpu-Gpu')]]"

        },

        "storageProfile": {

          "osDisk": {

            "createOption": "fromImage",

            "managedDisk": {

              "storageAccountType": "[variables('osDiskType')]"

            }

          },

          "imageReference": {

            "publisher": "microsoft-dsvm",

            "offer": "ubuntu-1804",

            "sku": "1804-gen2",

            "version": "latest"

          }

        },

        "networkProfile": {

          "networkInterfaces": [

            {

              "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('networkInterfaceName'))]"

            }

          ]

        },

        "osProfile": {

          "computerName": "[variables('virtualMachineName')]",

          "adminUsername": "[parameters('adminUsername')]",

          "adminPassword": "[parameters('adminPasswordOrKey')]",

          "linuxConfiguration": "[if(equals(parameters('authenticationType'), 'password'), json('null'), variables('linuxConfiguration'))]"

        }

      }

    }
```

该模板中定义了以下资源：

* [Microsoft.Compute/virtualMachines](https://docs.microsoft.com/azure/templates/microsoft.compute/virtualmachines)：创建基于云的虚拟机。 在此模板中，将虚拟机配置为运行 Ubuntu 18.04 的 Data Science Virtual Machine。

### <a name="deploy-the-template"></a>部署模板 

若要通过 Azure CLI 使用该模板，请登录并选择你的订阅（请参阅[使用 Azure CLI 登录](/cli/authenticate-azure-cli?view=azure-cli-latest)）。 运行：

```azurecli
read -p "Enter the name of the resource group to create:" resourceGroupName &&
read -p "Enter the Azure location (e.g., centralus):" location &&
read -p "Enter the authentication type (must be 'password' or 'sshPublicKey') :" authenticationType &&
read -p "Enter the login name for the administrator account (may not be 'admin'):" adminUsername &&
read -p "Enter administrator account secure string (value of password or ssh public key):" adminPasswordOrKey &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-ubuntu-DSVM-GPU-or-CPU/azuredeploy.json" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri $templateUri --parameters adminUsername=$adminUsername authenticationType=$authenticationType adminPasswordOrKey=$adminPasswordOrKey && 
echo "Press [ENTER] to continue ..." &&
read
```

运行上述命令时，请输入：

1. 要创建的包含 DSVM 和关联资源的资源组的名称。 
1. 要在其中进行部署的 Azure 位置
1. 要使用的身份验证类型（输入字符串 `password` 或 `sshPublicKey`）
1. 管理员帐户的登录名（此值不能为 `admin`）
1. 帐户的密码或 SSH 公钥的值

## <a name="review-deployed-resources"></a>查看已部署的资源

若要查看 Data Science Virtual Machine，请执行以下操作：

1. 转到 https://portal.azure.cn 
1. 登录 
1. 选择刚创建的资源组

你将看到资源组的信息： 

:::image type="content" source="media/dsvm-tutorial-resource-manager/resource-group-home.png" alt-text="包含 DSVM 的基本资源组的屏幕截图":::

单击虚拟机资源转到其信息页面。 可在此处找到 VM 的相关信息，包括连接详细信息。 

## <a name="clean-up-resources"></a>清理资源

如果不想使用此虚拟机，请删除它。 由于 DSVM 与其他资源（例如存储帐户）关联，因此可能需要删除所创建的整个资源组。 可以使用门户删除资源组，方法是单击“删除”按钮并进行确认。 也可以从 CLI 中使用以下命令删除资源组： 

```azurecli
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你通过 Azure 资源管理器模板创建了 Data Science Virtual Machine。 

> [!div class="nextstepaction"]
> [示例程序和 ML 演练](dsvm-samples-and-walkthroughs.md)
