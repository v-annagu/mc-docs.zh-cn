---
title: 从异地备份还原数据仓库
description: 有关异地还原 SQL 池的指南。
services: synapse-analytics
author: WenJason
manager: digimobile
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
origin.date: 07/12/2019
ms.date: 07/06/2020
ms.author: v-jay
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 9fe6fee799138998ff1a2a6959253a8501dbadab
ms.sourcegitcommit: 7ea2d04481512e185a60fa3b0f7b0761e3ed7b59
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/02/2020
ms.locfileid: "85845892"
---
# <a name="geo-restore-for-sql-pool"></a>SQL 池的异地还原

本文介绍了如何通过 Azure 门户和 PowerShell 从异地备份还原 SQL 池。

## <a name="before-you-begin"></a>准备阶段

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

**验证 DTU 容量。** 每个 SQL 池都由一个具有默认 DTU 配额的 SQL 服务器（例如 myserver.database.chinacloudapi.cn）托管。 验证 SQL Server 的剩余 DTU 配额是否足够进行数据库还原。

## <a name="restore-from-an-azure-geographical-region-through-powershell"></a>通过 PowerShell 从 Azure 地理区域还原

若要从异地备份还原，请使用 [Get-AzSqlDatabaseGeoBackup](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabasegeobackup?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) 和 [Restore-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/restore-azsqldatabase?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) cmdlet。

> [!NOTE]
> 可以执行到第 2 代的异地还原！ 若要执行此操作，请将一个第 2 代 ServiceObjectiveName（例如 DW1000**c**）指定为可选参数。
>

1. 开始之前，请确保[安装 Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。
2. 打开 PowerShell。
3. 连接到 Azure 帐户，并列出与帐户关联的所有订阅。
4. 选择包含要还原的数据仓库的订阅。
5. 获取要恢复的数据仓库。
6. 创建对数据仓库的恢复请求。
7. 验证异地还原的数据仓库的状态。
8. 若要在完成还原后配置数据仓库，请参阅[在恢复后配置数据库](../../sql-database/sql-database-disaster-recovery.md?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#configure-your-database-after-recovery)。

```Powershell
$SubscriptionName="<YourSubscriptionName>"
$ResourceGroupName="<YourResourceGroupName>"
$ServerName="<YourServerNameWithoutURLSuffixSeeNote>"  # Without database.chinacloudapi.cn
$TargetResourceGroupName="<YourTargetResourceGroupName>" # Restore to a different server.
$TargetServerName="<YourtargetServerNameWithoutURLSuffixSeeNote>"  
$DatabaseName="<YourDatabaseName>"
$NewDatabaseName="<YourDatabaseName>"
$TargetServiceObjective="<YourTargetServiceObjective-DWXXXc>"

Connect-AzAccount -Environment AzureChinaCloud
Get-AzSubscription
Select-AzSubscription -SubscriptionName $SubscriptionName
Get-AzureSqlDatabase -ServerName $ServerName

# Get the data warehouse you want to recover
$GeoBackup = Get-AzSqlDatabaseGeoBackup -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName

# Recover data warehouse
$GeoRestoredDatabase = Restore-AzSqlDatabase –FromGeoBackup -ResourceGroupName $TargetResourceGroupName -ServerName $TargetServerName -TargetDatabaseName $NewDatabaseName –ResourceId $GeoBackup.ResourceID -ServiceObjectiveName $TargetServiceObjective

# Verify that the geo-restored data warehouse is online
$GeoRestoredDatabase.status
```

如果源数据库启用了 TDE，则已恢复的数据库会启用 TDE。

## <a name="restore-from-an-azure-geographical-region-through-azure-portal"></a>通过 Azure 门户从 Azure 地理区域还原

按下述步骤从异地备份还原 SQL 池：

1. 登录到 [Azure 门户](https://portal.azure.cn/)帐户。
2. 单击“+ 创建资源”。

   ![新建 DW](./media/sql-data-warehouse-restore-from-geo-backup/georestore-new.png)

3. 依次单击“数据库”、“Azure Synapse Analytics”（以前称为“SQL DW”） 。

   ![新建 DW 2](./media/sql-data-warehouse-restore-from-geo-backup/georestore-new-02.png)

4. 填充在“基本信息”选项卡中请求的信息，然后单击“下一步:其他设置”。

   ![基础知识](./media/sql-data-warehouse-restore-from-geo-backup/georestore-dw-1.png)

5. 对于“使用现有的数据”参数，请选择“备份”，然后从向下滚动选项中选择适当的备份。 单击“查看 + 创建”。

   ![backup](./media/sql-data-warehouse-restore-from-geo-backup/georestore-select.png)

6. 数据仓库还原后，请检查“状态”是否为“联机”。

## <a name="next-steps"></a>后续步骤

- [还原现有的 SQL 池](sql-data-warehouse-restore-active-paused-dw.md)
- [还原已删除的 SQL 池](sql-data-warehouse-restore-deleted-dw.md)