---
title: 使用 ODBC 连接到 Azure 数据资源管理器
description: 本文介绍如何设置与 Azure 数据资源管理器的开放式数据库连接 (ODBC) 连接。
author: orspod
ms.author: v-tawe
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: conceptual
origin.date: 06/30/2019
ms.date: 11/18/2019
ms.openlocfilehash: d6ae42e1d14d4c4e4e4e8058deaedfbafa35aba3
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "74657959"
---
# <a name="connect-to-azure-data-explorer-with-odbc"></a>使用 ODBC 连接到 Azure 数据资源管理器

开放式数据库连接 ([ODBC](https://docs.microsoft.com/sql/odbc/reference/odbc-overview)) 是一种广泛接受的应用程序编程接口 (API)，适用于数据库访问。 使用 ODBC 可从没有专用连接器的应用程序连接到 Azure 数据资源管理器。

在后台，应用程序会在 ODBC 接口中调用函数，这些函数在特定于数据库的模块（称为“驱动程序”）中实现。  Azure 数据资源管理器支持部分 SQL Server 通信协议 ([MS-TDS](https://docs.microsoft.com/azure/kusto/api/tds/))；因此，它可以使用适用于 SQL Server 的 ODBC 驱动程序。

<!-- Using the following video, you can learn to create an ODBC connection.  -->

<!-- > [!VIDEO https://www.youtube.com/embed/qA5wxhrOwog] -->

或者，可以[配置 ODBC 数据源](#configure-the-odbc-data-source)，如下所述。 

本文介绍如何使用 SQL Server ODBC 驱动程序，使你能够从任何支持 ODBC 的应用程序连接到 Azure 数据资源管理器。 

## <a name="prerequisites"></a>必备条件

需要满足以下条件：

* 适用于操作系统的 [Microsoft ODBC Driver for SQL Server 17.2.0.1 或更高版本](https://docs.microsoft.com/sql/connect/odbc/download-odbc-driver-for-sql-server)。

## <a name="configure-the-odbc-data-source"></a>配置 ODBC 数据源

按以下步骤操作，使用 ODBC Driver for SQL Server 配置 ODBC 数据源。

1. 在 Windows 中搜索“ODBC 数据源”，  打开 ODBC 数据源桌面应用。

1. 选择 **添加** 。

    ![添加数据源](media/connect-odbc/add-data-source.png)

1. 选择“ODBC Driver 17 for SQL Server”，然后选择“完成”。  

    ![选择驱动程序](media/connect-odbc/select-driver.png)

1. 输入连接的名称和说明以及要连接到的群集，然后选择“下一步”  。 群集 URL 应采用 \<ClusterName\>.\<Region\>.kusto.chinacloudapi.cn 格式。

    ![选择服务器](media/connect-odbc/select-server.png)

1. 选择“Active Directory 集成”，然后选择“下一步”。  

    ![Active Directory 已集成](media/connect-odbc/active-directory-integrated.png)

1. 选择包含示例数据的数据库，然后选择“下一步”。 

    ![更改默认数据库](media/connect-odbc/change-default-database.png)

1. 在下一屏幕中将所有选项保留为默认设置，然后选择“完成”。 

1. 选择“测试数据源”  。

    ![测试数据源](media/connect-odbc/test-data-source.png)

1. 验证测试是否成功，然后选择“确定”。  如果测试不成功，请检查在前面的步骤中指定的值，并确保有足够的权限连接到群集。

    ![测试成功](media/connect-odbc/test-succeeded.png)

## <a name="next-steps"></a>后续步骤

* [从 Tableau 连接到 Azure 数据资源管理器](tableau.md)