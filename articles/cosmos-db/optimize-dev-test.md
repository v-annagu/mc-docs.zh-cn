---
title: Azure Cosmos DB 中的开发和测试优化
description: 本文介绍 Azure Cosmos DB 为服务的免费开发和测试提供多个选项的情况。
author: rockboyfor
ms.service: cosmos-db
ms.topic: conceptual
origin.date: 04/27/2020
ms.date: 07/06/2020
ms.author: v-yeche
ms.openlocfilehash: d4706796ae8311f5bcdf0224a09b4acc5231a65d
ms.sourcegitcommit: f5484e21fa7c95305af535d5a9722b5ab416683f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/24/2020
ms.locfileid: "85323265"
---
# <a name="optimize-development-and-testing-cost-in-azure-cosmos-db"></a>在 Azure Cosmos DB 中优化开发和测试成本

本文介绍免费使用 Azure Cosmos DB 进行开发和测试的各种选项，以及在开发或测试帐户中进行成本优化的各种技巧。

## <a name="azure-cosmos-db-emulator-locally-downloadable-version"></a>Azure Cosmos DB 模拟器（可以本地下载的版本）

[Azure Cosmos DB 模拟器](local-emulator.md)是一个本地的可下载版本，模拟 Azure Cosmos DB 云服务。 即使没有网络连接，也可以编写并测试使用 Azure Cosmos DB API 的代码，不需支付任何费用。 Azure Cosmos DB 模拟器提供了一个用于开发的具有云服务高保真特性的本地环境。 可以在本地开发和测试应用程序，无需创建 Azure 订阅。 做好将应用程序部署到云的准备以后，即可更新连接到云中的 Azure Cosmos DB 终结点所需的连接字符串，不需进行其他的修改。 也可在 Azure DevOps 中[通过 Azure Cosmos DB 模拟器生成任务设置 CI/CD 管道](tutorial-setup-ci-cd.md)，以便运行测试。 若要开始操作，可以参阅 [Azure Cosmos DB 模拟器](local-emulator.md)一文。

## <a name="azure-cosmos-db-free-tier"></a>Azure Cosmos DB 免费层

使用 Azure Cosmos DB 免费层，可以轻松上手、开发和测试应用程序，甚至免费运行小型生产工作负载。 在帐户上启用免费层后，一开始你将在该帐户中免费获得 400 RU/秒的吞吐量和 5 GB 的存储空间。 还可以创建包含 25 个容器的共享吞吐量数据库，这些容器在数据库级别的吞吐量均为 400 RU/秒，所有这些操作都可以通过免费层实现（免费层帐户的限制为 5 个共享吞吐量数据库）。 免费层在帐户的生命周期内无限期地持续有效，并且附带常规 Azure Cosmos DB 帐户的所有[优点和功能](introduction.md#key-benefits)，包括无限制的存储和吞吐量（RU/秒）、SLA、高可用性以及在所有 Azure 中国区域进行统包式多区域分布等。 每个 Azure 订阅最多可以有一个免费层帐户，并且必须在创建帐户时选择加入使用。 首先，[在 Azure 门户中创建一个启用了免费层的新帐户](create-cosmosdb-resources-portal.md) 或使用 [ARM 模板](manage-sql-with-resource-manager.md#free-tier)。 如需更多详细信息，请参阅[定价页](https://www.azure.cn/pricing/details/cosmos-db/)。

<!--Not Available on ## Try Azure Cosmos DB for free-->
<!--Not Available on [Try Azure Cosmos DB for free](https://www.azure.cn/try/cosmosdb/)-->

## <a name="azure-trial-account"></a>Azure 试用帐户

Azure Cosmos DB 包含在 [Azure 试用帐户](https://www.azure.cn/pricing/1rmb-trial)中，该帐户提供 Azure 信用额度和资源，可以免费使用特定的一段时间。 特别对于 Azure Cosmos DB，该试用帐户提供 5 GB 的存储和 400 RU 的全年预配吞吐量。 任何开发人员都可以通过此体验轻松地测试 Azure Cosmos DB 的功能，或者将其与其他 Azure 服务集成，没有任何费用。 Azure 试用帐户提供 $200 的信用额度，可以在头 30 天使用。 在选择升级之前，即使开始使用服务也不会收费。 若要开始操作，请访问 [Azure 试用帐户](https://www.azure.cn/pricing/1rmb-trial)页。

## <a name="use-shared-throughput-databases"></a>使用共享吞吐量数据库

在[共享吞吐量数据库](set-throughput.md#set-throughput-on-a-database)中，数据库中的所有容器共享数据库的已预配吞吐量（RU/秒）。 例如，如果为数据库预配了 400 RU/秒并且有四个容器，则所有四个容器共享此 400 RU/秒的吞吐量。 在开发或测试环境中，对每个容器的访问可能不是很频繁，因此，需求低于最小值 400 RU/秒，将容器置于共享吞吐量数据库中有助于优化成本。

例如，假设你的开发或测试帐户有四个容器。 如果创建具有专用吞吐量（最小为 400 RU/秒）的四个容器，则总吞吐量将为 1600 RU/秒。 相反，如果创建一个共享吞吐量数据库（至少 400 RU/秒）并将容器置于其中，则总吞吐量将只有 400 RU/秒。 通常情况下，共享吞吐量数据库非常适合不需要在任何单个容器上保证吞吐量的方案。  详细了解[共享吞吐量数据库](set-throughput.md#set-throughput-on-a-database)。

## <a name="next-steps"></a>后续步骤

可以按照以下文章的说明，开始使用模拟器或免费的 Azure Cosmos DB 帐户：

* 详细了解[开发和测试优化](optimize-dev-test.md)
* 详细了解[了解 Azure Cosmos DB 帐单](understand-your-bill.md)
* 详细了解如何[优化吞吐量成本](optimize-cost-throughput.md)
* 详细了解如何[优化存储成本](optimize-cost-storage.md)
* 详细了解如何[优化读取和写入成本](optimize-cost-reads-writes.md)
* 详细了解如何[优化查询成本](optimize-cost-queries.md)
* 详细了解[优化多区域 Azure Cosmos 帐户的成本](optimize-cost-regions.md)

<!-- Update_Description: update meta properties, wording update, update link -->