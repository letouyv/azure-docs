---
title: 'Migrate your data to a Cassandra API account in Azure Cosmos DB- Tutorial'
description: In this tutorial, learn how to copy data from Apache Cassandra to a Cassandra API account in Azure Cosmos DB.
author: TheovanKraay
ms.author: thvankra
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: tutorial
ms.date: 12/03/2018
ms.custom: seodec18
#Customer intent: As a developer, I want to migrate my existing Cassandra workloads to Azure Cosmos DB so that the overhead to manage resources, clusters, and garbage collection is automatically handled by Azure Cosmos DB.
---

# Tutorial: Migrate your data to a Cassandra API account
[!INCLUDE[appliesto-cassandra-api](../includes/appliesto-cassandra-api.md)]

As a developer, you might have existing Cassandra workloads that are running on-premises or in the cloud, and you might want to migrate them to Azure. You can migrate such workloads to a Cassandra API account in Azure Cosmos DB. This tutorial provides instructions on different options available to migrate Apache Cassandra data into the Cassandra API account in Azure Cosmos DB.

This tutorial covers the following tasks:

> [!div class="checklist"]
> * Plan for migration
> * Prerequisites for migration
> * Migrate data by using the `cqlsh` `COPY` command
> * Migrate data by using Spark

If you don’t have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

## Prerequisites for migration

* **Estimate your throughput needs:** Before migrating data to the Cassandra API account in Azure Cosmos DB, you should estimate the throughput needs of your workload. In general, start with the average throughput required by the CRUD operations, and then include the additional throughput required for the Extract Transform Load or spiky operations. You need the following details to plan for migration: 

  * **Existing data size or estimated data size:** Defines the minimum database size and throughput requirement. If you are estimating data size for a new application, you can assume that the data is uniformly distributed across the rows, and estimate the value by multiplying with the data size. 

  * **Required throughput:** Approximate throughput rate of read (query/get) and write (update/delete/insert) operations. This value is required to compute the required request units, along with steady-state data size.  

  * **The schema:** Connect to your existing Cassandra cluster through `cqlsh`, and export the schema from Cassandra: 

    ```bash
    cqlsh [IP] "-e DESC SCHEMA" > orig_schema.cql
    ```

    After you identify the requirements of your existing workload, create an Azure Cosmos DB account, database, and containers, according to the gathered throughput requirements.  

  * **Determine the RU charge for an operation:** You can determine the RUs by using any of the SDKs supported by the Cassandra API. This example shows the .NET version of getting RU charges.

    ```csharp
    var tableInsertStatement = table.Insert(sampleEntity);
    var insertResult = await tableInsertStatement.ExecuteAsync();

    foreach (string key in insertResult.Info.IncomingPayload)
      {
         byte[] valueInBytes = customPayload[key];
         double value = Encoding.UTF8.GetString(valueInBytes);
         Console.WriteLine($"CustomPayload:  {key}: {value}");
      }
    ```

* **Allocate the required throughput:** Azure Cosmos DB can automatically scale storage and throughput as your requirements grow. You can estimate your throughput needs by using the [Azure Cosmos DB request unit calculator](https://www.documentdb.com/capacityplanner). 

* **Create tables in the Cassandra API account:** Before you start migrating data, pre-create all your tables from the Azure portal or from `cqlsh`. If you're migrating to an Azure Cosmos DB account that has database-level throughput, make sure to provide a partition key when you create the containers.

* **Increase throughput:** The duration of your data migration depends on the amount of throughput you provisioned for the tables in Azure Cosmos DB. Increase the throughput for the duration of migration. With the higher throughput, you can avoid rate limiting and migrate in less time. After you've completed the migration, decrease the throughput to save costs. We also recommend that you have the Azure Cosmos DB account in the same region as your source database. 

* **Enable TLS:** Azure Cosmos DB has strict security requirements and standards. Be sure to enable TLS when you interact with your account. When you use CQL with SSH, you have an option to provide TLS information.

## Options to migrate data

You can move data from existing Cassandra workloads to Azure Cosmos DB by using the `cqlsh` `COPY` command, or by using Spark. 

### Migrate data by using the cqlsh COPY command

Use the [CQL COPY command](https://cassandra.apache.org/doc/latest/tools/cqlsh.html#cqlsh) to copy local data to the Cassandra API account in Azure Cosmos DB.

1. Get your Cassandra API account’s connection string information:

   * Sign in to the [Azure portal](https://portal.azure.com), and go to your Azure Cosmos DB account.

   * Open the **Connection String** pane. Here you see all the information that you need to connect to your Cassandra API account from `cqlsh`.

1. Sign in to `cqlsh` by using the connection information from the portal.

1. Use the `CQL` `COPY` command to copy local data to the Cassandra API account.

   ```bash
   COPY exampleks.tablename FROM filefolderx/*.csv 
   ```

### Migrate data by using Spark 

Use the following steps to migrate data to the Cassandra API account with Spark:

1. Provision an [Azure Databricks cluster](spark-databricks.md) or an [Azure HDInsight cluster](spark-hdinsight.md). 

1. Move data to the destination Cassandra API endpoint. Refer to this [how-to guide](migrate-data-databricks.md) for migration with Azure Databricks.

Migrating data by using Spark jobs is a recommended option if you have data residing in an existing cluster in Azure virtual machines or any other cloud. To do this, you must set up Spark as an intermediary for one-time or regular ingestion. You can accelerate this migration by using Azure ExpressRoute connectivity between your on-premises environment and Azure. 

## Clean up resources

When they're no longer needed, you can delete the resource group, the Azure Cosmos DB account, and all the related resources. To do so, select the resource group for the virtual machine, select **Delete**, and then confirm the name of the resource group to delete.

## Next steps

In this tutorial, you've learned how to migrate your data to a Cassandra API account in Azure Cosmos DB. You can now learn about other concepts in Azure Cosmos DB:

> [!div class="nextstepaction"]
> [Tunable data consistency levels in Azure Cosmos DB](../consistency-levels.md)




