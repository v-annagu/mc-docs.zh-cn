---
title: 使用 PowerShell 创建和管理 Azure Cosmos DB
description: 使用 Azure Powershell 管理 Azure Cosmos 帐户、数据库、容器和吞吐量。
author: rockboyfor
ms.service: cosmos-db
ms.topic: sample
origin.date: 05/13/2020
ms.date: 06/22/2020
ms.author: v-yeche
ms.custom: seodec18
ms.openlocfilehash: 1e2e371634df1348450294fd58a7bbeac8e4bcb8
ms.sourcegitcommit: 873e5c5e4156efed505a78d4f5a6e50c494e76d4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/07/2020
ms.locfileid: "86036748"
---
# <a name="manage-azure-cosmos-db-sql-api-resources-using-powershell"></a>使用 PowerShell 管理 Azure Cosmos DB SQL API 资源

以下指南介绍了如何使用 Powershell 通过脚本来自动管理 Azure Cosmos DB 资源，其中包括帐户、数据库、容器和吞吐量。

> [!NOTE]
> 本文中的示例使用 [Az.CosmosDB](https://docs.microsoft.com/powershell/module/az.cosmosdb) 管理 cmdlet。 有关最新更改，请参阅 [Az.CosmosDB](https://docs.microsoft.com/powershell/module/az.cosmosdb) API 参考页。

若要对 Azure Cosmos DB 进行跨平台管理，可以将 `Az` 和 `Az.CosmosDB` cmdlet 与[跨平台 Powershell](https://docs.microsoft.com/powershell/scripting/install/installing-powershell) 以及 [Azure CLI](manage-with-cli.md)、[REST API][rp-rest-api] 或 [Azure 门户](create-sql-api-dotnet.md#create-account)配合使用。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="getting-started"></a>入门

请按照[如何安装和配置 Azure PowerShell][powershell-install-configure] 中的说明安装 PowerShell 并在其中登录 Azure 帐户。

## <a name="azure-cosmos-accounts"></a>Azure Cosmos 帐户

以下部分演示如何管理 Azure Cosmos 帐户，包括：

* [创建 Azure Cosmos 帐户](#create-account)
* [更新 Azure Cosmos 帐户](#update-account)
* [列出订阅中的所有 Azure Cosmos 帐户](#list-accounts)
* [获取 Azure Cosmos 帐户](#get-account)
* [删除 Azure Cosmos 帐户](#delete-account)
* [更新 Azure Cosmos 帐户标记](#update-tags)
* [列出 Azure Cosmos 帐户的密钥](#list-keys)
* [重新生成 Azure Cosmos 帐户的密钥](#regenerate-keys)
* [列出 Azure Cosmos 帐户的连接字符串](#list-connection-strings)
* [修改 Azure Cosmos 帐户的故障转移优先级](#modify-failover-priority)
* [触发 Azure Cosmos 帐户的手动故障转移](#trigger-manual-failover)

<a name="create-account"></a>
### <a name="create-an-azure-cosmos-account"></a>创建 Azure Cosmos 帐户

此命令创建一个 Azure Cosmos DB 数据库帐户，该帐户使用[多区域][distribute-data-multiple-regionally]、[自动故障转移](how-to-manage-database-account.md#automatic-failover)和有限过期[一致性策略](consistency-levels.md)。

```powershell
$resourceGroupName = "myResourceGroup"
$locations = @("China North 2", "China East 2")
$accountName = "mycosmosaccount"
$apiKind = "Sql"
$consistencyLevel = "BoundedStaleness"
$maxStalenessInterval = 300
$maxStalenessPrefix = 100000

New-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Location $locations `
    -Name $accountName `
    -ApiKind $apiKind `
    -EnableAutomaticFailover:$true `
    -DefaultConsistencyLevel $consistencyLevel `
    -MaxStalenessIntervalInSeconds $maxStalenessInterval `
    -MaxStalenessPrefix $maxStalenessPrefix
```

* `$resourceGroupName`：要在其中部署 Cosmos 帐户的 Azure 资源组。 它必须已存在。
* `$locations`：数据库帐户的区域，从写入区域开始，按故障转移优先级排序。
* `$accountName`：Azure Cosmos 帐户的名称。 必须独一无二且必须为小写，仅包含字母数字和“-”字符，长度为 3 到 31 个字符。
* `$apiKind`：要创建的 Cosmos 帐户的类型。 有关详细信息，请参阅 [Cosmos DB 中的 API](introduction.md#develop-applications-on-cosmos-db-using-popular-open-source-software-oss-apis)。
* `$consistencyPolicy`、`$maxStalenessInterval` 和 `$maxStalenessPrefix`：Azure Cosmos 帐户的默认一致性级别和设置。 有关详细信息，请参阅 [Azure Cosmos DB 中的一致性级别](consistency-levels.md)。

可以为 Azure Cosmos 帐户配置 IP 防火墙和虚拟网络服务终结点。 有关如何为 Azure Cosmos DB 配置 IP 防火墙的信息，请参阅[配置 IP 防火墙](how-to-configure-firewall.md)。 若要了解如何为 Azure Cosmos DB 启用服务终结点，请参阅[配置从虚拟网络进行访问的权限](how-to-configure-vnet-service-endpoint.md)。

<!--Not Available on , and private endpoints-->
<!--Not Available on [Configure access from private endpoints](how-to-configure-private-endpoints.md)-->

<a name="list-accounts"></a>
### <a name="list-all-azure-cosmos-accounts-in-a-resource-group"></a>列出资源组中的所有 Azure Cosmos 帐户

该命令列出资源组中的所有 Azure Cosmos 帐户。

```powershell
$resourceGroupName = "myResourceGroup"

Get-AzCosmosDBAccount -ResourceGroupName $resourceGroupName
```

<a name="get-account"></a>
### <a name="get-the-properties-of-an-azure-cosmos-account"></a>获取 Azure Cosmos 帐户的属性

使用此命令可以获取现有 Azure Cosmos 帐户的属性。

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBAccount -ResourceGroupName $resourceGroupName -Name $accountName
```

<a name="update-account"></a>
### <a name="update-an-azure-cosmos-account"></a>更新 Azure Cosmos 帐户

此命令可更新 Azure Cosmos DB 数据库帐户属性。 可更新的属性包括：

* 添加或删除区域
* 更改默认的一致性策略
* 更改 IP 范围筛选器
* 更改虚拟网络配置
* 启用多主数据库

> [!NOTE]
> 不能同时添加或删除区域 (`locations`) 并更改 Azure Cosmos 帐户的其他属性。 修改区域的操作必须作为单独的操作与任何其他对帐户的更改操作分开执行。
> [!NOTE]
> 此命令可添加和删除区域，但不可修改故障转移优先级或触发手动故障转移。 请参阅[修改故障转移优先级](#modify-failover-priority)和[触发手动故障转移](#trigger-manual-failover)。

```powershell
# Create account with two regions
$resourceGroupName = "myResourceGroup"
$locations = @("China North 2", "China East 2")
$accountName = "mycosmosaccount"
$apiKind = "Sql"
$consistencyLevel = "Session"
$enableAutomaticFailover = $true

# Create the Cosmos DB account
New-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Location $locations `
    -Name $accountName `
    -ApiKind $apiKind `
    -EnableAutomaticFailover:$enableAutomaticFailover `
    -DefaultConsistencyLevel $consistencyLevel

# Add a region to the account
$locations2 = @("China North 2", "China East 2", "China East")
$locationObjects2 = @()
$i = 0
ForEach ($location in $locations2) {
    $locationObjects2 += @{ locationName = "$location"; failoverPriority = $i++ }
}

Update-AzCosmosDBAccountRegion `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -LocationObject $locationObjects2

Write-Host "Update-AzCosmosDBAccountRegion returns before the region update is complete."
Write-Host "Check account in Azure portal or using Get-AzCosmosDBAccount for region status."
Write-Host "When region was added, press any key to continue."
$HOST.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown") | OUT-NULL
$HOST.UI.RawUI.Flushinputbuffer()

# Remove a region from the account
$locations3 = @("China North 2", "China East")
$locationObjects3 = @()
$i = 0
ForEach ($location in $locations3) {
    $locationObjects3 += @{ locationName = "$location"; failoverPriority = $i++ }
}

Update-AzCosmosDBAccountRegion `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -LocationObject $locationObjects3

Write-Host "Update-AzCosmosDBAccountRegion returns before the region update is complete."
Write-Host "Check account in Azure portal or using Get-AzCosmosDBAccount for region status."
```
<a name="multi-master"></a>
### <a name="enable-multiple-write-regions-for-an-azure-cosmos-account"></a>为 Azure Cosmos 帐户启用多个写入区域

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$enableAutomaticFailover = $false
$enableMultiMaster = $true

# First disable automatic failover - cannot have both automatic
# failover and multi-master on an account
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableAutomaticFailover:$enableAutomaticFailover

# Now enable multi-master
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableMultipleWriteLocations:$enableMultiMaster
```

<a name="delete-account"></a>
### <a name="delete-an-azure-cosmos-account"></a>删除 Azure Cosmos 帐户

此命令删除现有的 Azure Cosmos 帐户。

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Remove-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -PassThru:$true
```

<a name="update-tags"></a>
### <a name="update-tags-of-an-azure-cosmos-account"></a>更新 Azure Cosmos 帐户的标记

此命令设置 Azure Cosmos 帐户的 [Azure 资源标记][azure-resource-tags]。 标记可以在创建帐户时使用 `New-AzCosmosDBAccount` 来设置，也可以在更新帐户时使用 `Update-AzCosmosDBAccount` 来设置。
> [!NOTE]

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$tags = @{dept = "Finance"; environment = "Production";}

Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -Tag $tags
```

<a name="list-keys"></a>
### <a name="list-account-keys"></a>列出帐户密钥

创建 Azure Cosmos 帐户时，服务生成两个主访问密钥，这两个密钥可用于访问 Azure Cosmos 帐户时进行的身份验证。 还会生成只读密钥，用于对只读操作进行身份验证。
提供两个访问密钥后，Azure Cosmos DB 支持在不中断 Azure Cosmos 帐户的情况下重新生成密钥，并以轮换的方式一次使用一个密钥。
Cosmos DB 帐户有两个读写密钥（主密钥和辅助密钥）和两个只读密钥（主密钥和辅助密钥）。

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -Type "Keys"
```

<a name="list-connection-strings"></a>
### <a name="list-connection-strings"></a>列出连接字符串

以下命令检索用于将应用程序连接到 Cosmos DB 帐户的连接字符串。

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -Type "ConnectionStrings"
```

<a name="regenerate-keys"></a>
### <a name="regenerate-account-keys"></a>重新生成帐户密钥

应定期重新生成 Azure Cosmos 帐户访问密钥，确保连接安全。 系统会向帐户分配主要和辅助访问密钥。 这样，在某个时刻重新生成某个密钥时，客户端仍可进行访问。
Azure Cosmos 帐户有四种类型的密钥（Primary、Secondary、PrimaryReadonly 和 SecondaryReadonly）

```powershell
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "mycosmosaccount" # Must be all lower case
$keyKind = "primary" # Other key kinds: secondary, primaryReadOnly, secondaryReadOnly

New-AzCosmosDBAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -KeyKind $keyKind
```

<a name="enable-automatic-failover"></a>
### <a name="enable-automatic-failover"></a>启用自动故障转移

以下命令设置一个 Cosmos DB 帐户，使之能够在主要区域变得不可用时自动故障转移到次要区域。

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$enableAutomaticFailover = $true
$enableMultiMaster = $false

# First disable multi-master - cannot have both automatic
# failover and multi-master on an account
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableMultipleWriteLocations:$enableMultiMaster

# Now enable automatic failover
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableAutomaticFailover:$enableAutomaticFailover
```

<a name="modify-failover-priority"></a>
### <a name="modify-failover-priority"></a>修改故障转移优先级

对于配置了自动故障转移的帐户，可以更改在主要副本不可用时 Cosmos 将次要副本提升为主要副本的顺序。

对于以下示例，假定当前的故障转移优先级为 `China North 2 = 0`、`China East 2 = 1`、`China East = 2`。 此命令会将其更改为 `China North 2 = 0`、`China East = 1`、`China East 2 = 2`。

> [!CAUTION]
> 在 `failoverPriority=0` 的情况下更改位置会触发 Azure Cosmos 帐户的手动故障转移。 任何其他的优先级更改不会触发故障转移。

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$locations = @("China North 2", "China East", "China East 2") # Regions ordered by UPDATED failover priority

Update-AzCosmosDBAccountFailoverPriority `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -FailoverPolicy $locations
```

<a name="trigger-manual-failover"></a>
### <a name="trigger-manual-failover"></a>触发手动故障转移

对于配置了手动故障转移的帐户，可以通过将设置修改为 `failoverPriority=0` 来进行故障转移并将所有次要副本提升为主要副本。 此操作可用于启动灾难恢复演练以测试灾难恢复规划。

在下面的示例中，假设帐户的当前故障转移优先级为 `China North 2 = 0` 和 `China East 2 = 1`，然后将区域互换。

> [!CAUTION]
> 在 `failoverPriority=0` 的情况下更改 `locationName` 会触发 Azure Cosmos 帐户的手动故障转移。 任何其他优先级更改都不会触发故障转移。

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$locations = @("China East 2", "China North 2") # Regions ordered by UPDATED failover priority

Update-AzCosmosDBAccountFailoverPriority `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -FailoverPolicy $locations
```

## <a name="azure-cosmos-db-database"></a>Azure Cosmos DB 数据库

以下部分演示了如何管理 Azure Cosmos DB 数据库，具体包括：

* [创建 Azure Cosmos DB 数据库](#create-db)
* [创建共享吞吐量的 Azure Cosmos DB 数据库](#create-db-ru)
* [获取 Azure Cosmos DB 数据库的吞吐量](#get-db-ru)
* [列出帐户中的所有 Azure Cosmos DB 数据库](#list-db)
* [获取单个 Azure Cosmos DB 数据库](#get-db)
* [删除 Azure Cosmos DB 数据库](#delete-db)

<a name="create-db"></a>
### <a name="create-an-azure-cosmos-db-database"></a>创建 Azure Cosmos DB 数据库

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

New-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

<a name="create-db-ru"></a>
### <a name="create-an-azure-cosmos-db-database-with-shared-throughput"></a>创建共享吞吐量的 Azure Cosmos DB 数据库

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$databaseRUs = 400

New-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName `
    -Throughput $databaseRUs
```

<a name="get-db-ru"></a>
### <a name="get-the-throughput-of-an-azure-cosmos-db-database"></a>获取 Azure Cosmos DB 数据库的吞吐量

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Get-AzCosmosDBSqlDatabaseThroughput `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

<a name="list-db"></a>
### <a name="get-all-azure-cosmos-db-databases-in-an-account"></a>获取帐户中的所有 Azure Cosmos DB 数据库

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName
```

<a name="get-db"></a>
### <a name="get-a-single-azure-cosmos-db-database"></a>获取单个 Azure Cosmos DB 数据库

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Get-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

<a name="delete-db"></a>
### <a name="delete-an-azure-cosmos-db-database"></a>删除 Azure Cosmos DB 数据库

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Remove-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

## <a name="azure-cosmos-db-container"></a>Azure Cosmos DB 容器

以下部分演示了如何管理 Azure Cosmos DB 容器，具体包括：

* [创建 Azure Cosmos DB 容器](#create-container)
* [使用大分区键创建 Azure Cosmos DB 容器](#create-container-big-pk)
* [获取 Azure Cosmos DB 容器的吞吐量](#get-container-ru)
* [使用自定义索引创建 Azure Cosmos DB 容器](#create-container-custom-index)
* [在索引关闭的情况下创建 Azure Cosmos DB 容器](#create-container-no-index)
* [创建键和 TTL 都独一无二的 Azure Cosmos DB 容器](#create-container-unique-key-ttl)
* [创建带冲突解决方案的 Azure Cosmos DB 容器](#create-container-lww)
* [列出数据库中的所有 Azure Cosmos DB 容器](#list-containers)
* [获取数据库中的单个 Azure Cosmos DB 容器](#get-container)
* [删除 Azure Cosmos DB 容器](#delete-container)

<a name="create-container"></a>
### <a name="create-an-azure-cosmos-db-container"></a>创建 Azure Cosmos DB 容器

```powershell
# Create an Azure Cosmos DB container with default indexes and throughput at 400 RU
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath
```

<a name="create-container-big-pk"></a>
### <a name="create-an-azure-cosmos-db-container-with-a-large-partition-key-size"></a>使用大分区键大小创建 Azure Cosmos DB 容器

```powershell
# Create an Azure Cosmos DB container with a large partition key value (version = 2)
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -PartitionKeyVersion 2
```

<a name="get-container-ru"></a>
### <a name="get-the-throughput-of-an-azure-cosmos-db-container"></a>获取 Azure Cosmos DB 容器的吞吐量

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"

Get-AzCosmosDBSqlContainerThroughput `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName
```

<a name="create-container-custom-index"></a>
### <a name="create-an-azure-cosmos-db-container-with-custom-index-policy"></a>使用自定义索引策略创建 Azure Cosmos DB 容器

```powershell
# Create a container with a custom indexing policy
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$indexPathIncluded = "/*"
$indexPathExcluded = "/myExcludedPath/*"

$includedPathIndex = New-AzCosmosDBSqlIncludedPathIndex -DataType String -Kind Range
$includedPath = New-AzCosmosDBSqlIncludedPath -Path $indexPathIncluded -Index $includedPathIndex

$indexingPolicy = New-AzCosmosDBSqlIndexingPolicy `
    -IncludedPath $includedPath `
    -ExcludedPath $indexPathExcluded `
    -IndexingMode Consistent `
    -Automatic $true

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -IndexingPolicy $indexingPolicy
```

<a name="create-container-no-index"></a>
### <a name="create-an-azure-cosmos-db-container-with-indexing-turned-off"></a>在索引关闭的情况下创建 Azure Cosmos DB 容器

```powershell
# Create an Azure Cosmos DB container with no indexing
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"

$indexingPolicy = New-AzCosmosDBSqlIndexingPolicy `
    -IndexingMode None

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -IndexingPolicy $indexingPolicy
```

<a name="create-container-unique-key-ttl"></a>
### <a name="create-an-azure-cosmos-db-container-with-unique-key-policy-and-ttl"></a>创建键策略和 TTL 都独一无二的 Azure Cosmos DB 容器

```powershell
# Create a container with a unique key policy and TTL of one day
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$uniqueKeyPath = "/myUniqueKeyPath"
$ttlInSeconds = 86400 # Set this to -1 (or don't use it at all) to never expire

$uniqueKey = New-AzCosmosDBSqlUniqueKey `
    -Path $uniqueKeyPath

$uniqueKeyPolicy = New-AzCosmosDBSqlUniqueKeyPolicy `
    -UniqueKey $uniqueKey

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -UniqueKeyPolicy $uniqueKeyPolicy `
    -TtlInSeconds $ttlInSeconds
```

<a name="create-container-lww"></a>
### <a name="create-an-azure-cosmos-db-container-with-conflict-resolution"></a>创建带冲突解决方案的 Azure Cosmos DB 容器

若要将所有冲突都写入 ConflictsFeed，然后分别进行处理，请传递 `-Type "Custom" -Path ""`。

```powershell
# Create container with last-writer-wins conflict resolution policy
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$conflictResolutionPath = "/myResolutionPath"

$conflictResolutionPolicy = New-AzCosmosDBSqlConflictResolutionPolicy `
    -Type LastWriterWins `
    -Path $conflictResolutionPath

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -ConflictResolutionPolicy $conflictResolutionPolicy
```

若要创建冲突解决策略以使用存储过程，请调用 `New-AzCosmosDBSqlConflictResolutionPolicy` 并传递参数 `-Type` 和 `-ConflictResolutionProcedure`。

```powershell
# Create container with custom conflict resolution policy using a stored procedure
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$conflictResolutionSprocName = "mysproc"

$conflictResolutionSproc = "/dbs/$databaseName/colls/$containerName/sprocs/$conflictResolutionSprocName"

$conflictResolutionPolicy = New-AzCosmosDBSqlConflictResolutionPolicy `
    -Type Custom `
    -ConflictResolutionProcedure $conflictResolutionSproc

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -ConflictResolutionPolicy $conflictResolutionPolicy
```

<a name="list-containers"></a>
### <a name="list-all-azure-cosmos-db-containers-in-a-database"></a>列出数据库中的所有 Azure Cosmos DB 容器

```powershell
# List all Azure Cosmos DB containers in a database
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Get-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName
```

<a name="get-container"></a>
### <a name="get-a-single-azure-cosmos-db-container-in-a-database"></a>获取数据库中的单个 Azure Cosmos DB 容器

```powershell
# Get a single Azure Cosmos DB container in a database
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"

Get-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName
```

<a name="delete-container"></a>
### <a name="delete-an-azure-cosmos-db-container"></a>删除 Azure Cosmos DB 容器

```powershell
# Delete an Azure Cosmos DB container
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"

Remove-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName
```

## <a name="next-steps"></a>后续步骤

* [所有 PowerShell 示例](powershell-samples.md)
* [如何管理 Azure Cosmos 帐户](how-to-manage-database-account.md)
* [创建 Azure Cosmos DB 容器](how-to-create-container.md)
* [在 Azure Cosmos DB 中配置生存时间](how-to-time-to-live.md)

<!--Reference style links - using these makes the source content way more readable than using inline links-->

[powershell-install-configure]: /powershell-install-configure
[scaling-multiple-regionally]: distribute-data-globally.md#EnableGlobalDistribution
[distribute-data-multiple-regionally]: distribute-data-globally.md
[azure-resource-groups]: /azure-resource-manager/resource-group-overview#resource-groups
[azure-resource-tags]: /azure-resource-manager/resource-group-using-tags
[rp-rest-api]: https://docs.microsoft.com/rest/api/cosmos-db-resource-provider/

<!-- Update_Description: update meta properties, wording update, update link -->