---
title: 有关 Azure Cosmos DB Autopilot 模式下的吞吐量的常见问题
description: 有关 Azure Cosmos DB 数据库和容器的 Autopilot 模式的常见问题
author: rockboyfor
ms.author: v-yeche
ms.service: cosmos-db
ms.topic: conceptual
origin.date: 12/16/2019
ms.date: 01/20/2020
ms.openlocfilehash: 00211753520d246300ddc101251215c100134fe5
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "76270022"
---
# <a name="frequently-asked-questions-about-provisioned-throughput-in-azure-cosmos-db-autopilot-mode-preview"></a>有关 Azure Cosmos DB Autopilot 模式下的预配吞吐量的常见问题（预览版）
在 Autopilot 模式下，Azure Cosmos DB 将根据使用情况自动管理和缩放容器或数据库的 RU/s。 本文解答了有关 Autopilot 模式的常见问题。 

## <a name="frequently-asked-questions"></a>常见问题

### <a name="is-autopilot-mode-supported-for-all-apis"></a>是否所有 API 都支持 Autopilot 模式？
是的，所有 API 都支持 Autopilot 模式：Core (SQL)、Gremlin、Table、Cassandra 和 API for MongoDB。

### <a name="is-autopilot-mode-supported-for-multi-master-accounts"></a>多主帐户是否支持 Autopilot 模式？
是的，多主帐户支持 Autopilot 模式。 添加到 Cosmos 帐户的每个区域都提供最大 RU/s。 

### <a name="what-is-the-pricing-for-autopilot"></a>Autopilot 采用何种定价方式？
有关详细信息，请参阅 Azure Cosmos DB [定价页](https://www.azure.cn/pricing/details/cosmos-db/)。 

### <a name="how-do-i-enable-autopilot-for-my-containers-or-databases"></a>如何为容器或数据库启用 Autopilot？
可以在使用 Azure 门户创建的新容器和数据库上启用 Autopilot 模式。 

### <a name="is-there-cli-or-sdk-support-to-create-containers-or-databases-with-autopilot-mode"></a>是否支持通过 CLI 或 SDK 创建使用 Autopilot 模式的容器或数据库？
目前，在预览版本中，只能通过 Azure 门户创建使用 Autopilot 模式的资源。 尚不支持 CLI 和 SDK。

### <a name="can-i-enable-autopilot-on-an-existing-container-or-a-database"></a>能否在现有容器或数据库上启用 Autopilot？
目前，可以在创建新容器和数据库时对它们启用 Autopilot。 尚不支持在现有容器和数据库上启用 Autopilot 模式。 可以使用 [Azure 数据工厂](../data-factory/connector-azure-cosmos-db.md)或[更改源](change-feed.md)将现有容器迁移到新容器。 

### <a name="can-i-turn-off-autopilot-mode-on-a-container-or-database"></a>能否在容器或数据库上禁用 Autopilot 模式？
是的，可以通过为预配吞吐量切换到“手动”选项来禁用 Autopilot。 在预览版本中，从 Autopilot 模式切换到手动模式后，无法再次为同一资源启用 Autopilot。 

### <a name="is-autopilot-mode-supported-for-shared-throughput-databases"></a>共享吞吐量数据库是否支持 Autopilot 模式？
是的，共享吞吐量数据库支持 Autopilot 模式。 若要启用此功能，请在创建数据库时选择 Autopilot 模式和“预配吞吐量”选项  。 

### <a name="what-is-the-number-of-allowed-collections-per-shared-throughput-database-when-autopilot-is-enabled"></a>启用 Autopilot 后，每个共享吞吐量数据库允许的集合数是多少？
对于启用了 Autopilot 模式的共享吞吐量数据库，允许的集合数为：MIN(25, 数据库的最大 RU/s / 1000)。 例如，如果数据库的最大吞吐量为 20,000 RU/s，则数据库可以有 MIN(25, 20,000 RU/s / 1000) = 20 个集合。 

### <a name="what-is-the-storage-limit-associated-with-each-max-rus-option"></a>与每个最大 RU/s 选项关联的存储限制是多少？  
每个最大 RU/s 的存储限制（以 GB 为单位）为：数据库或容器的最大 RU/s / 100。 例如，如果最大 RU/s 为 20,000 RU/s，则资源可支持 200 GB 的存储。 请参阅 [Autopilot 限制](provision-throughput-autopilot.md#autopilot-limits)一文，了解可用的最大 RU/s 和存储选项。 

### <a name="what-happens-if-i-exceed-the-storage-limit-associated-with-my-max-throughput"></a>如果超出与最大吞吐量关联的存储限制，会发生什么情况？
如果超出了与数据库或容器的最大吞吐量关联的存储限制，Azure Cosmos DB 会自动将最大吞吐量提高到可以支持该级别存储的下一个最高层级。 例如，假设为数据库或容器预配了 4000 RU/s 的最大吞吐量选项，该选项的存储限制为 50 GB。 如果资源存储增加到 100 GB，则数据库或容器的最大 RU/s 将自动增加到 20,000 RU/s（最多可支持 200 GB）。 

### <a name="how-quickly-will-autopilot-scale-up-and-down-based-on-spikes-in-traffic"></a>Autopilot 根据流量高峰进行增减的速度有多快？
Autopilot 将根据传入流量在最小和最大 RU/s 范围内立即增加或减少 RU/s。 计费时间间隔为 1 小时，即，按某一小时内的最高 RU/s 收费。 

### <a name="can-i-specify-a-custom-max-throughput-rus-value-for-autopilot-mode"></a>能否为 Autopilot 模式指定自定义最大吞吐量 (RU/s) 值？
目前，预览版本中有[四个最大吞吐量 (RU/s) 选项](provision-throughput-autopilot.md#autopilot-limits)可供选择。

### <a name="can-i-increase-the-max-rus-move-to-a-higher-tier-on-the-database-or-container"></a>能否增加数据库或容器的最大 RU/s（移至较高层级）？ 
是的。 可以从容器的“缩放和设置”选项或数据库的“缩放”选项中，为 Autopilot 选择更高的最大 RU/s   。 这是一个异步增加操作，随着服务预配更多资源来支持更高的规模，该操作可能需要一段时间才能完成（通常为 4-6 小时，具体取决于所选的 RU/s）。 

### <a name="can-i-reduce-the-max-rus-move-to-a-lower-tier-on-the-database-or-container"></a>能否降低数据库或容器的最大 RU/s（移至较低层级）？
是的。 只要数据库或容器的当前存储低于要缩减到的最大 RU/s 层级的相关[存储限制](#what-is-the-storage-limit-associated-with-each-max-rus-option)，就可以将最大 RU/s 降低到该层级。 例如，如果选择 20,000 RU/s 作为最大 RU/s，但存储小于 50 GB（与 4000 RU/s 关联的存储限制），则可以将最大 RU/s 缩减为 4000 RU/s。

### <a name="what-is-the-mapping-between-the-max-rus-and-physical-partitions"></a>最大 RU/s 与物理分区之间的映射是什么？
首次选择最大 RU/s 时，Azure Cosmos DB 将进行以下预配：最大 RU/s / 10,000 RU/s = 物理分区数。 每个[物理分区](partition-data.md#physical-partitions)最多可支持 10,000 RU/s 和 50 GB 的存储。 随着存储大小的增长，Azure Cosmos DB 将自动拆分分区，以添加更多物理分区来应对存储的增长；如果存储[超过相关限制](#what-is-the-storage-limit-associated-with-each-max-rus-option)，Azure Cosmos DB 将增加最大 RU/s 层级。 

数据库或容器的最大 RU/s 在所有物理分区之间平均分配。 因此，任何单个物理分区可以扩展到的总吞吐量为：数据库或容器的最大 RU/s / 物理分区数。 

### <a name="what-happens-if-incoming-requests-exceed-the-max-rus-of-the-database-or-container"></a>如果传入请求超过数据库或容器的最大 RU/s，会发生什么情况？
如果 RU/s 总消耗量超过容器或数据库的最大 RU/s，则超过最大 RU/s 的请求将受到限制并返回 429 状态代码。 导致标准化利用率超过 100% 的请求也将受到限制。 标准化利用率定义为所有物理分区中 RU/s 利用率的最大值。 例如，假设最大吞吐量为 20,000 RU/s，并且有两个物理分区 P_1 和 P_2，每个分区都可以扩展到 10,000 RU/s。 在给定的秒数内，如果 P_1 已使用 6000 RU，而 P_2 已使用 8000 RU，则标准化利用率为 MAX(6000 RU / 10,000 RU, 8000 RU / 10,000 RU) = 0.8。

> [!NOTE]
> Azure Cosmos DB 客户端 SDK 和数据导入工具（Azure 数据工厂、批量执行程序库）会在出现 429 错误时自动重试，因此，偶尔出现 429 错误是正常的。 持续大量的 429 错误可能表明你需要提高最大 RU/s 或查看[热分区](#is-it-still-possible-to-see-429s-throttlingrate-limiting-when-autopilot-is-enabled)的分区策略。

### <a name="is-it-still-possible-to-see-429s-throttlingrate-limiting-when-autopilot-is-enabled"></a>启用 Autopilot 后，是否仍有可能看到 429 错误（限制/速率限制）？ 
是的。 在两种情况下有可能看到 429 错误。 第一种，当 RU/s 总消耗量超过容器或数据库的最大 RU/s 时，服务将相应地限制请求。 

第二种，如果存在热分区，即与其他分区键值相比，请求数量高到不成比例的逻辑分区键值，则基础物理分区可能会超出其 RU/s 预算。 为避免出现热分区，最佳做法是[选择一个合适的分区键](partitioning-overview.md#choose-partitionkey)，以实现存储和吞吐量的均匀分配。 

例如，如果选择 20,000 RU/s 的最大吞吐量选项，并具有 200 GB 的存储和四个物理分区，则每个物理分区最多可以自动扩展至 5000 RU/s。 如果特定逻辑分区键上有一个热分区，则当它所在的基础物理分区超过 5000 RU/s（即超过 100% 的标准化利用率）时，将显示 429 错误。

## <a name="next-steps"></a>后续步骤

* 了解如何[在 Azure Cosmos 容器或数据库上启用 Autopilot](provision-throughput-autopilot.md#create-a-database-or-a-container-with-autopilot-mode)。
* 了解 [Autopilot 的优点](provision-throughput-autopilot.md#benefits-of-autopilot-mode)。
* 详细了解[逻辑分区和物理分区](partition-data.md)。

<!-- Update_Description: new article about autopilot faq -->
<!--NEW.date: 01/13/2020-->