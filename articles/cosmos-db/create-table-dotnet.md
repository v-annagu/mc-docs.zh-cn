---
title: 快速入门 - 将表 API 与 .NET 配合使用 - Azure Cosmos DB
description: 本快速入门介绍如何在 Azure 门户和 .NET 中使用 Azure Cosmos DB 表 API 创建应用程序
author: rockboyfor
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: dotnet
ms.topic: quickstart
origin.date: 05/28/2020
ms.date: 06/22/2020
ms.author: v-yeche
ms.openlocfilehash: 03b06c2dda388531a38210a73c6a060aaddef34b
ms.sourcegitcommit: 48b5ae0164f278f2fff626ee60db86802837b0b4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2020
ms.locfileid: "85098658"
---
<!--Verify sucessfully-->
# <a name="quickstart-build-a-table-api-app-with-net-sdk-and-azure-cosmos-db"></a>快速入门：使用 .NET SDK 和 Azure Cosmos DB 生成表 API 应用 

> [!div class="op_single_selector"]
> * [.NET](create-table-dotnet.md)
> * [Java](create-table-java.md)
> * [Node.js](create-table-nodejs.md)
> * [Python](create-table-python.md)
>  

本快速入门介绍如何使用 .NET 和 Azure Cosmos DB [表 API](table-introduction.md)，通过克隆 GitHub 中的示例来生成应用。 此外，本快速入门还介绍了如何创建 Azure Cosmos DB 帐户，以及如何在基于 Web 的 Azure 门户中使用数据资源管理器创建表和实体。

## <a name="prerequisites"></a>先决条件

如果尚未安装 Visual Studio 2019，可以下载并使用**免费**的 [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/)。 在安装 Visual Studio 的过程中，请确保启用“Azure 开发”。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-a-database-account"></a>创建数据库帐户

[!INCLUDE [cosmos-db-create-dbaccount-table](../../includes/cosmos-db-create-dbaccount-table.md)]

## <a name="add-a-table"></a>添加表

[!INCLUDE [cosmos-db-create-table](../../includes/cosmos-db-create-table.md)]

## <a name="add-sample-data"></a>添加示例数据

[!INCLUDE [cosmos-db-create-table-add-sample-data](../../includes/cosmos-db-create-table-add-sample-data.md)]

## <a name="clone-the-sample-application"></a>克隆示例应用程序

现在让我们从 GitHub 克隆表应用，设置连接字符串，然后运行该应用。 会看到以编程方式处理数据是多么容易。 

1. 打开命令提示符，新建一个名为“git-samples”的文件夹，然后关闭命令提示符。

    ```bash
    md "C:\git-samples"
    ```

2. 打开诸如 git bash 之类的 git 终端窗口，并使用 `cd` 命令更改为要安装示例应用的新文件夹。

    ```bash
    cd "C:\git-samples"
    ```

3. 运行下列命令，克隆示例存储库。 此命令在计算机上创建示例应用程序的副本。

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-table-dotnet-core-getting-started.git
    ```

> [!TIP]
> 有关类似代码的更详细演练，请参阅 [Cosmos DB 表 API 示例](table-storage-how-to-use-dotnet.md)一文。

## <a name="open-the-sample-application-in-visual-studio"></a>在 Visual Studio 中打开示例应用程序

1. 在 Visual Studio 的“文件”菜单中选择“打开”，然后选择“项目/解决方案”。   

    ![打开解决方案](media/create-table-dotnet/azure-cosmosdb-open-solution.png) 

2. 导航到示例应用程序所克隆到的文件夹，然后打开 TableStorage.sln 文件。

## <a name="review-the-code"></a>查看代码

此步骤是可选的。 如果有意了解如何使用代码创建数据库资源，可以查看以下代码片段。 否则，可以直接跳转到本文档的[更新连接字符串](#update-your-connection-string)部分。

* 下面的代码演示如何在 Azure 存储中创建表：

    ```csharp
    public static async Task<CloudTable> CreateTableAsync(string tableName)
    {
      string storageConnectionString = AppSettings.LoadAppSettings().StorageConnectionString;

      // Retrieve storage account information from connection string.
      CloudStorageAccount storageAccount = CreateStorageAccountFromConnectionString(storageConnectionString);

      // Create a table client for interacting with the table service
      CloudTableClient tableClient = storageAccount.CreateCloudTableClient(new TableClientConfiguration());

      Console.WriteLine("Create a Table for the demo");

      // Create a table client for interacting with the table service 
      CloudTable table = tableClient.GetTableReference(tableName);
      if (await table.CreateIfNotExistsAsync())
      {
          Console.WriteLine("Created Table named: {0}", tableName);
      }
      else
      {
          Console.WriteLine("Table {0} already exists", tableName);
      }

      Console.WriteLine();
      return table;
    }

    ```

* 以下代码演示了如何在表中插入数据：

    ```csharp
    public static async Task<CustomerEntity> InsertOrMergeEntityAsync(CloudTable table, CustomerEntity entity)
    {
      if (entity == null)
      {
          throw new ArgumentNullException("entity");
      }

      try
      {
          // Create the InsertOrReplace table operation
          TableOperation insertOrMergeOperation = TableOperation.InsertOrMerge(entity);

          // Execute the operation.
          TableResult result = await table.ExecuteAsync(insertOrMergeOperation);
          CustomerEntity insertedCustomer = result.Result as CustomerEntity;

          if (result.RequestCharge.HasValue)
          {
              Console.WriteLine("Request Charge of InsertOrMerge Operation: " + result.RequestCharge);
          }

          return insertedCustomer;
      }
      catch (StorageException e)
      {
          Console.WriteLine(e.Message);
          Console.ReadLine();
          throw;
      }
    }

    ```
* 以下代码演示了如何查询表中的数据：

    ```csharp
    public static async Task<CustomerEntity> RetrieveEntityUsingPointQueryAsync(CloudTable table, string partitionKey, string rowKey)
    {
      try
      {
          TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>(partitionKey, rowKey);
          TableResult result = await table.ExecuteAsync(retrieveOperation);
          CustomerEntity customer = result.Result as CustomerEntity;
          if (customer != null)
          {
              Console.WriteLine("\t{0}\t{1}\t{2}\t{3}", customer.PartitionKey, customer.RowKey, customer.Email, customer.PhoneNumber);
          }

          if (result.RequestCharge.HasValue)
          {
              Console.WriteLine("Request Charge of Retrieve Operation: " + result.RequestCharge);
          }

          return customer;
      }
      catch (StorageException e)
      {
          Console.WriteLine(e.Message);
          Console.ReadLine();
          throw;
      }
    }

    ```

* 以下代码演示了如何删除表中的数据：

    ```csharp
    public static async Task DeleteEntityAsync(CloudTable table, CustomerEntity deleteEntity)
    {
      try
      {
          if (deleteEntity == null)
          {
              throw new ArgumentNullException("deleteEntity");
          }

          TableOperation deleteOperation = TableOperation.Delete(deleteEntity);
          TableResult result = await table.ExecuteAsync(deleteOperation);

          if (result.RequestCharge.HasValue)
          {
              Console.WriteLine("Request Charge of Delete Operation: " + result.RequestCharge);
          }

      }
      catch (StorageException e)
      {
          Console.WriteLine(e.Message);
          Console.ReadLine();
          throw;
      }
    }

    ```

## <a name="update-your-connection-string"></a>更新连接字符串

现在返回到 Azure 门户，获取连接字符串信息，并将其复制到应用。 这样，应用程序就可以与托管的数据库进行通信。 

1. 在 [Azure 门户](https://portal.azure.cn/)中，单击“连接字符串”。 使用窗口右侧的复制按钮复制“主连接字符串”。

    ![在“连接字符串”窗格中查看并复制“主连接字符串”](./media/create-table-dotnet/connection-string.png)

2. 在 Visual Studio 中打开 **Settings.json** 文件。 

3. 将门户中的“主连接字符串”粘贴到 StorageConnectionString 值中。 粘贴引号内的字符串。

    ```csharp
    {
      "StorageConnectionString": "<Primary connection string from Azure portal>"
    }
    ```

4. 按 CTRL+S 保存 **Settings.json** 文件。

现已使用与 Azure Cosmos DB 进行通信所需的所有信息更新应用。 

## <a name="build-and-deploy-the-app"></a>生成并部署应用

1. 在 Visual Studio 中，右键单击“解决方案资源管理器”中的“CosmosTableSamples”项目，然后单击“管理 NuGet 包”。   

    ![管理 NuGet 包](media/create-table-dotnet/azure-cosmosdb-manage-nuget.png)

2. 在 NuGet 的“浏览”框中，键入 Microsoft.Azure.Cosmos.Table。 这样会查找 Cosmos DB 表 API 客户端库。 请注意，此库目前仅适用于 .NET Framework 和 .NET Standard。 

    ![NuGet 的“浏览”选项卡](media/create-table-dotnet/azure-cosmosdb-nuget-browse.png)

3. 单击“安装”以安装 **Microsoft.Azure.Cosmos.Table** 库。 这会安装 Azure Cosmos DB 表 API 包和所有依赖项。

4. 运行整个应用时，示例数据将插入到表实体中，运行结束时会删除这些数据，因此，如果运行整个示例，你将看不到插入的任何数据。 但是，可以插入一些断点来查看数据。 打开 BasicSamples.cs 文件并右键单击第 52 行，选择“断点”，然后选择“插入断点”。  在第 55 行中插入另一个断点。

    ![添加断点](media/create-table-dotnet/azure-cosmosdb-breakpoint.png) 

5. 按 F5 运行应用程序。 控制台窗口会显示 Azure Cosmos DB 中新的表数据库的名称（在本例中为 demoa13b1）。 

    ![控制台输出](media/create-table-dotnet/azure-cosmosdb-console.png)

    点击第一个断点后，返回到 Azure 门户中的数据资源管理器。 单击“刷新”按钮，展开 demo* 表，然后单击“实体”。 右侧的“实体”选项卡将显示为 Walter Harp 添加的新实体。 请注意，新实体的电话号码为 425-555-0101。

    ![新建实体](media/create-table-dotnet/azure-cosmosdb-entity.png)

    如果收到说明在运行项目时无法找到 Settings.json 文件的错误，可以通过将以下 XML 条目添加到项目设置来解决该问题。 右键单击 CosmosTableSamples，选择“编辑 CosmosTableSamples.csproj”并添加以下 itemGroup： 

    ```csharp
     <ItemGroup>
       <None Update="Settings.json">
         <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
       </None>
     </ItemGroup>
    ```

6. 关闭数据资源管理器中的“实体”选项卡。

7. 按 F5，运行应用到下一个断点。 

    点击该断点后，切换回 Azure 门户，再次单击“实体”以打开“实体”选项卡，请注意该电话号码已更新为 425-555-0105。 

8. 按 F5 运行该应用。 

    该应用添加的实体可在表 API 目前不支持的高级示例应用中使用。 该应用然后会删除示例应用创建的表。

9. 在控制台窗口中，按 Enter 结束应用的执行。 

## <a name="review-slas-in-the-azure-portal"></a>在 Azure 门户中查看 SLA

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>后续步骤

在本快速入门教程中，已了解如何创建 Azure Cosmos DB 帐户、使用数据资源管理器创建表和运行应用。  现在可以使用表 API 进行数据查询了。  

> [!div class="nextstepaction"]
> [将表数据导入表 API](table-import.md)

<!-- Update_Description: update meta properties, wording update, update link -->