---
title: 快速入门：使用模板在 HDInsight 中创建 Spark 群集
description: 本快速入门演示如何使用资源管理器模板在 Azure HDInsight 中创建 Apache Spark 群集，并运行简单的 Spark SQL 查询。
services: azure-hdinsight
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: quickstart
origin.date: 06/12/2019
ms.date: 03/02/2020
ms.author: v-yiso
ms.openlocfilehash: 7200794b021eadc211169a366c5a86dd6bad94d4
ms.sourcegitcommit: 3a8a7d65d0791cdb6695fe6c2222a1971a19f745
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2020
ms.locfileid: "85516596"
---
# <a name="quickstart-create-apache-spark-cluster-in-azure-hdinsight-using-resource-manager-template"></a>快速入门：使用资源管理器模板在 Azure HDInsight 中创建 Apache Spark 群集

在本快速入门中，我们使用 Azure 资源管理器模板在 Azure HDInsight 中创建一个 [Apache Spark](./apache-spark-overview.md) 群集。 然后，我们创建一个 Jupyter 笔记本，并使用它针对 Apache Hive 表运行 Spark SQL 查询。 Azure HDInsight 是适用于企业的分析服务，具有托管、全面且开源的特点。 用于 HDInsight 的 Apache Spark 框架使用内存中处理功能实现快速数据分析和群集计算。 使用 Jupyter 笔记本，可以与数据进行交互、将代码和 Markdown 文本结合使用，以及进行简单的可视化。

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

- 具有活动订阅的 Azure 帐户。 [创建试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="create-an-apache-spark-cluster"></a>创建 Apache Spark 群集

### <a name="review-the-template"></a>查看模板

本快速入门中使用的模板来自 [Azure 快速入门模板](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-spark-linux)。

```Json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "clusterName": {
      "type": "string",
      "metadata": {
        "description": "The name of the HDInsight cluster to create."
      }
    },
    "clusterLoginUserName": {
      "type": "string",
      "defaultValue": "admin",
      "metadata": {
        "description": "These credentials can be used to submit jobs to the cluster and to log into cluster dashboards."
      }
    },
    "clusterLoginPassword": {
      "type": "securestring",
      "metadata": {
        "description": "The password must be at least 10 characters in length and must contain at least one digit, one non-alphanumeric character, and one upper or lower case letter."
      }
    },
    "sshUserName": {
      "type": "string",
      "defaultValue": "sshuser",
      "metadata": {
        "description": "These credentials can be used to remotely access the cluster."
      }
    },
    "sshPassword": {
      "type": "securestring",
      "metadata": {
        "description": "The password must be at least 10 characters in length and must contain at least one digit, one non-alphanumeric character, and one upper or lower case letter."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    }
  },
  "variables": {
    "defaultStorageAccount": {
      "name": "[uniqueString(resourceGroup().id)]",
      "type": "Standard_LRS"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('defaultStorageAccount').name]",
      "location": "[parameters('location')]",
      "apiVersion": "2016-01-01",
      "sku": {
        "name": "[variables('defaultStorageAccount').type]"
      },
      "kind": "Storage",
      "properties": {}
    },
    {
      "type": "Microsoft.HDInsight/clusters",
      "name": "[parameters('clusterName')]",
      "location": "[parameters('location')]",
      "apiVersion": "2018-06-01-preview",
      "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/',variables('defaultStorageAccount').name)]"
      ],
      "tags": {},
      "properties": {
        "clusterVersion": "3.6",
        "osType": "Linux",
        "tier": "Standard",
        "clusterDefinition": {
          "kind": "spark",
          "configurations": {
            "gateway": {
              "restAuthCredential.isEnabled": true,
              "restAuthCredential.username": "[parameters('clusterLoginUserName')]",
              "restAuthCredential.password": "[parameters('clusterLoginPassword')]"
            }
          }
        },
        "storageProfile": {
          "storageaccounts": [
            {
              "name": "[replace(replace(reference(resourceId('Microsoft.Storage/storageAccounts', variables('defaultStorageAccount').name), '2016-01-01').primaryEndpoints.blob,'https://',''),'/','')]",
              "isDefault": true,
              "container": "[parameters('clusterName')]",
              "key": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('defaultStorageAccount').name), '2016-01-01').keys[0].value]"
            }
          ]
        },
        "computeProfile": {
          "roles": [
            {
              "name": "headnode",
              "targetInstanceCount": 2,
              "hardwareProfile": {
                "vmSize": "Standard_D12_v2"
              },
              "osProfile": {
                "linuxOperatingSystemProfile": {
                  "username": "[parameters('sshUserName')]",
                  "password": "[parameters('sshPassword')]"
                }
              },
              "virtualNetworkProfile": null,
              "scriptActions": []
            },
            {
              "name": "workernode",
              "targetInstanceCount": 2,
              "hardwareProfile": {
                "vmSize": "Standard_D13_v2"
              },
              "osProfile": {
                "linuxOperatingSystemProfile": {
                  "username": "[parameters('sshUserName')]",
                  "password": "[parameters('sshPassword')]"
                }
              },
              "virtualNetworkProfile": null,
              "scriptActions": []
            }
          ]
        }
      }
    }
  ],
  "outputs": {
    "storage": {
      "type": "object",
      "value": "[reference(resourceId('Microsoft.Storage/storageAccounts', variables('defaultStorageAccount').name))]"
    },
    "cluster": {
      "type": "object",
      "value": "[reference(resourceId('Microsoft.HDInsight/clusters',parameters('clusterName')))]"
    }
  }
}
```

该模板中定义了两个 Azure 资源：

* [Microsoft.Storage/storageAccounts](https://docs.microsoft.com/azure/templates/microsoft.storage/storageaccounts)：创建 Azure 存储帐户。
* [Microsoft.HDInsight/cluster](https://docs.microsoft.com/azure/templates/microsoft.hdinsight/clusters)：创建 HDInsight 群集。

### <a name="deploy-the-template"></a>部署模板

1. 选择下面的“部署到 Azure”按钮以登录到 Azure，并打开资源管理器模板。

    [![“部署到 Azure”](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-spark-linux%2Fazuredeploy.json)

1. 输入或选择下列值：

    |属性 |说明 |
    |---|---|
    |订阅|从下拉列表中选择用于此群集的 Azure 订阅。|
    |资源组|从下拉列表中选择现有资源组，或选择“新建”。|
    |位置|将使用用于资源组的位置自动填充此值。|
    |群集名称|输入任何全局唯一的名称。 对于此模板，请只使用小写字母和数字。|
    |群集登录用户名|提供用户名，默认值为 **admin**。|
    |群集登录密码|提供密码。 密码长度不得少于 10 个字符，且至少必须包含一个数字、一个大写字母和一个小写字母、一个非字母数字字符（' " ` 字符除外）。 |
    |SSH 用户名|提供用户名，默认值为 **sshuser**|
    |SSH 密码|提供密码。|

    ![使用 Azure 资源管理器模板在 HDInsight 中创建 Spark 群集](./media/apache-spark-jupyter-spark-sql/resource-manager-template-spark.png "使用 Azure 资源管理器模板在 HDInsight 中创建 Spark 群集")

1. 查看“条款和条件”。 接着选择“我同意上述条款和条件”，然后选择“购买” 。 你会收到一则通知，说明正在进行部署。 创建群集大约需要 20 分钟时间。

如果在创建 HDInsight 群集时遇到问题，可能是因为你没有这样做的适当权限。 有关详细信息，请参阅[访问控制要求](../hdinsight-hadoop-customize-cluster-linux.md#access-control)。

## <a name="review-deployed-resources"></a>查看已部署的资源

创建群集后，你将收到“部署成功”通知，其中包含“转到资源”链接。 “资源组”页将列出新的 HDInsight 群集以及与此群集关联的默认存储。 每个群集都有一个 [Azure 存储](../hdinsight-hadoop-use-blob-storage.md)帐户依赖项。 该帐户称为默认存储帐户。 HDInsight 群集及其默认存储帐户必须共存于同一个 Azure 区域中。 删除群集不会删除存储帐户。

## <a name="create-a-jupyter-notebook"></a>创建 Jupyter 笔记本

[Jupyter Notebook](https://jupyter.org/) 是支持各种编程语言的交互式笔记本环境。 通过此笔记本可以与数据进行交互、结合代码和 markdown 文本以及执行简单的可视化效果。

1. 打开 [Azure 门户](https://portal.azure.cn)。
2. 选择“HDInsight 群集”，然后选择所创建的群集。

    ![在 Azure 门户中打开 HDInsight 群集](./media/apache-spark-jupyter-spark-sql/azure-portal-open-hdinsight-cluster.png)

3. 在门户的“群集仪表板”部分中，选择“Jupyter Notebook”。  出现提示时，请输入群集的群集登录凭据。

   ![打开 Jupyter Notebook 以运行交互式 Spark SQL 查询](./media/apache-spark-jupyter-spark-sql/hdinsight-spark-open-jupyter-interactive-spark-sql-query.png "打开 Jupyter Notebook 以运行交互式 Spark SQL 查询")

4. 选择“新建” > “PySpark”，创建笔记本 。 

   ![创建 Jupyter Notebook 以运行交互式 Spark SQL 查询](./media/apache-spark-jupyter-spark-sql/hdinsight-spark-create-jupyter-interactive-spark-sql-query.png "创建 Jupyter Notebook 以运行交互式 Spark SQL 查询")

   新 Notebook 随即会创建，并以 Untitled(Untitled.pynb) 名称打开。

## <a name="run-apache-spark-sql-statements"></a>运行 Apache Spark SQL 语句

SQL（结构化查询语言）是用于查询和转换数据的最常见、最广泛使用的语言。 Spark SQL 作为 Apache Spark 的扩展使用，可使用熟悉的 SQL 语法处理结构化数据。

1. 验证 kernel 已就绪。 如果在 Notebook 中的内核名称旁边看到空心圆，则内核已准备就绪。 实心圆表示内核正忙。

    ![HDInsight Spark 中的 Hive 查询](./media/apache-spark-jupyter-spark-sql/jupyter-spark-kernel-status.png "HDInsight Spark 中的 Hive 查询")

    首次启动 Notebook 时，内核在后台执行一些任务。 等待内核准备就绪。 
2. 将以下代码粘贴到一个空单元格中，然后按 **SHIFT + ENTER** 来运行这些代码。 此命令列出群集上的 Hive 表：

    ```sql
    %%sql
    SHOW TABLES
    ```

    将 Jupyter Notebook 与 HDInsight 群集配合使用时，会获得一个预设 `spark` 会话，可以使用它通过 Spark SQL 来运行 Hive 查询。 `%%sql` 指示 Jupyter Notebook 使用预设 `spark` 会话运行 Hive 查询。 该查询从默认情况下所有 HDInsight 群集都带有的 Hive 表 (hivesampletable) 检索前 10 行。 第一次提交查询时，Jupyter 将为笔记本创建 Spark 应用程序。 该操作需要大约 30 秒才能完成。 Spark 应用程序准备就绪后，查询将在大约一秒钟内执行并生成结果。 输出如下所示：

    ![HDInsight Spark 中的 Hive 查询](./media/apache-spark-jupyter-spark-sql/hdinsight-spark-get-started-hive-query.png "HDInsight Spark 中的 Hive 查询")

    每次在 Jupyter 中运行查询时，Web 浏览器窗口标题中都会显示“(繁忙)”状态和 Notebook 标题。 右上角“PySpark”文本的旁边还会出现一个实心圆。
    
2. 运行另一个查询，请查看 `hivesampletable` 中的数据。

    ```sql
    %%sql
    SELECT * FROM hivesampletable LIMIT 10
    ```
    
    屏幕在刷新后会显示查询输出。

    ![HDInsight 中的 Hive 查询输出](./media/apache-spark-jupyter-spark-sql/hdinsight-spark-get-started-hive-query-output.png "HDInsight 中的 Hive 查询输出")

1. 请在 Notebook 的“文件”菜单中选择“关闭并停止” 。 关闭笔记本会释放群集资源，包括 Spark 应用程序。

## <a name="clean-up-resources"></a>清理资源

完成本快速入门后，可以删除群集。 有了 HDInsight，便可以将数据存储在 Azure 存储中，因此可以在群集不用时安全地删除群集。 此外，还需要为 HDInsight 群集付费，即使不用也是如此。 由于群集费用数倍于存储空间费用，因此在群集不用时删除群集可以节省费用。

从 Azure 门户导航到群集，然后选择“删除”。

![在 Azure 门户中删除 HDInsight 群集](./media/apache-spark-jupyter-spark-sql/hdinsight-azure-portal-delete-cluster.png "删除 HDInsight 群集")

还可以选择资源组名称来打开“资源组”页，然后选择“删除资源组”。 通过删除资源组，可以删除 HDInsight 群集和默认存储帐户。

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已了解如何在 HDInsight 中创建 Apache Spark 群集并运行基本的 Spark SQL 查询。 转到下一教程，了解如何使用 HDInsight 群集针对示例数据运行交互式查询。

> [!div class="nextstepaction"]
>[在 Apache Spark 上运行交互式查询](./apache-spark-load-data-run-query.md)


