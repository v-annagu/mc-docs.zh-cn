---
title: include 文件
description: include 文件
services: cosmos-db
author: rockboyfor
ms.service: cosmos-db
ms.topic: include
origin.date: 04/05/2019
ms.date: 04/27/2020
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: 7923690d7583a486680fd7f0c8793b0d246a73d2
ms.sourcegitcommit: f9c242ce5df12e1cd85471adae52530c4de4c7d7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/24/2020
ms.locfileid: "82134498"
---
可以在数据资源管理器中使用查询来检索和筛选数据。

1. 在数据资源管理器的“项”选项卡顶部，  查看默认查询 `SELECT * FROM c`。 此查询检索并显示容器中按 ID 排序的所有文档。 

    ![数据资源管理器中的默认查询是“SELECT * FROM c”](./media/cosmos-db-create-sql-api-query-data/azure-cosmosdb-data-explorer-query.png)

1. 若要更改查询，请选择“编辑筛选器”，  将默认查询替换为 `ORDER BY c._ts DESC`，然后选择“应用筛选器”。 

    ![添加“ORDER BY c._ts DESC”并单击“应用筛选器”，更改默认查询](./media/cosmos-db-create-sql-api-query-data/azure-cosmosdb-data-explorer-edit-query.png)

   此修改后的查询根据文档的时间戳按降序显示文档，所以现在最先列出的是第二个文档。 

    ![将查询更改为 ORDER BY c._ts DESC，然后单击“应用筛选器”](./media/cosmos-db-create-sql-api-query-data/azure-cosmosdb-data-explorer-edited-query.png)

如果熟悉 SQL 语法，可以在查询谓词框中输入任何受支持的 [SQL 查询](../articles/cosmos-db/sql-api-sql-query.md)。 还可以使用数据资源管理器创建存储过程、UDF 和触发器以执行服务器端业务逻辑。 

数据资源管理器可以通过 Azure 门户轻松访问 API 中提供的所有内置编程数据访问功能。 也可通过门户缩放吞吐量、获取密钥和连接字符串，以及查看 Azure Cosmos DB 帐户的指标和 SLA。

<!-- Update_Description: update meta properties, wording update, update link -->