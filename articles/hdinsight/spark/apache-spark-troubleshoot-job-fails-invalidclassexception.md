---
title: Apache Spark 中的 InvalidClassException 错误 - Azure HDInsight
description: 在 Azure HDInsight 中，Apache Spark 作业因 InvalidClassException（类版本不匹配）而失败
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: v-yiso
origin.date: 07/29/2019
ms.date: 12/09/2019
ms.openlocfilehash: 3354caf58d0d7eeac06166f386372a613605da68
ms.sourcegitcommit: 3a8a7d65d0791cdb6695fe6c2222a1971a19f745
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2020
ms.locfileid: "85516577"
---
# <a name="apache-spark-job-fails-with-invalidclassexception-class-version-mismatch-in-azure-hdinsight"></a>在 Azure HDInsight 中，Apache Spark 作业因 InvalidClassException（类版本不匹配）而失败

本文介绍在 Azure HDInsight 群集中使用 Apache Spark 组件时出现的问题的故障排除步骤和可能的解决方案。

## <a name="issue"></a>问题

尝试在 Spark 2.x 群集中创建 Apache Spark 作业。 它失败，出现类似于以下内容的错误：

```
18/09/18 09:32:26 WARN TaskSetManager: Lost task 0.0 in stage 1.0 (TID 1, wn7-dev-co.2zyfbddadfih0xdq0cdja4g.ax.internal.chinacloudapp.cn, executor 4): java.io.InvalidClassException:
org.apache.commons.lang3.time.FastDateFormat; local class incompatible: stream classdesc serialVersionUID = 2, local class serialVersionUID = 1
        at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:699)
        at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1885)
        at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1751)
        at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2042)
        at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1573)
```

## <a name="cause"></a>原因

此错误可能是由于向 `spark.yarn.jars` 配置中添加其他 JAR 导致的，具体来说是包含不同版本的 `commons-lang3` 包并引入类不匹配的阴影 JAR。 默认情况下，Spark 2.1/2/3 使用版本 3.5 的 `commons-lang3`。

> [!TIP]
> 为库添加阴影就是将其内容放入你自己的 jar 中，并更改其包。 这与打包库不同，后者是将库放入你自己的 jar 中，而不重新打包。

## <a name="resolution"></a>解决方法

删除 jar，或重新编译自定义 jar (AzureLogAppender)，并使用 [maven-shade-plugin](https://maven.apache.org/plugins/maven-shade-plugin/examples/class-relocation.html) 重新定位类。

## <a name="next-steps"></a>后续步骤

如果你的问题未在本文中列出，或者无法解决问题，请访问以下渠道以获取更多支持：


* 如果需要更多帮助，可以从 [Azure 门户](https://portal.azure.cn/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/)提交支持请求。 从菜单栏中选择“支持”  ，或打开“帮助 + 支持”  中心。 有关更多详细信息，请参阅[如何创建 Azure 支持请求](https://docs.microsoft.com/azure/azure-supportability/how-to-create-azure-support-request)。 在 Microsoft Azure 订阅中可以访问订阅管理和计费支持；通过 [Azure 支持计划](https://azure.microsoft.com/support/plans/)之一提供技术支持。
