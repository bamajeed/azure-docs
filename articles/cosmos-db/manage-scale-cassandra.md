---
title: Elastically scale with Cassandra API in Azure Cosmos DB
description: Learn about the options available to scale an Azure Cosmos DB Cassandra API account and their advantages/disadvantages
author: TheovanKraay
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 01/13/2020
ms.author: thvankra
---

# Elastically scale an Azure Cosmos DB Cassandra API account

There are a variety of options to explore the elastic nature of the Azure Cosmos DB's API for Cassandra. To understand how to scale effectively in Azure Cosmos DB, it is important to understand how to provision the right amount of request units (RU/s) to account for the performance demands in your system. To learn more about request units, see the [request units](request-units.md) article. 

![Database operations consume Request Units](./media/request-units/request-units.png)

## Handling rate limiting (429 errors)

Azure Cosmos DB will return rate-limited (429) errors if clients consume more resources (RU/s) than the amount that you have provisioned. The Cassandra API in Azure Cosmos DB translates these exceptions to overloaded errors on the Cassandra native protocol. 

If your system is not sensitive to latency, it may be sufficient to handle the throughput rate-limiting by using retries. See the [Java code sample](https://github.com/Azure-Samples/azure-cosmos-cassandra-java-retry-sample) for how to handle rate limiting transparently by using the [Azure Cosmos DB extension](https://github.com/Azure/azure-cosmos-cassandra-extensions) for [Cassandra retry policy](https://docs.datastax.com/en/developer/java-driver/4.4/manual/core/retries/) in Java. You can also use the [Spark extension](https://mvnrepository.com/artifact/com.microsoft.azure.cosmosdb/azure-cosmos-cassandra-spark-helper) to handle rate-limiting.

## Manage scaling

If you need to minimize latency, there is a spectrum of options for managing scale and provisioning throughput (RUs) in the Cassandra API:

* [Manually by using the Azure portal](#use-azure-portal)
* [Programmatically by using the control plane features](#use-control-plane)
* [Programmatically by using CQL commands with a specific SDK](#use-cql-queries)
* [Dynamically by using Autopilot](#use-autopilot)

The following sections explain the advantages and disadvantages of each approach. You can then decide on the best strategy to balance the scaling needs of your system, the overall cost, and efficiency needs for your solution.

## <a id="use-azure-portal"></a>Use the Azure portal

You can scale the resources in Azure Cosmos DB Cassandra API account by using Azure portal. To learn more, see the article on [Provision throughput on containers and databases](set-throughput.md). This article explains the relative benefits of setting throughput at either [database](set-throughput.md#set-throughput-on-a-database) or [container](set-throughput.md#set-throughput-on-a-container) level in the Azure portal. The terms "database" and "container" mentioned in these articles map to "keyspace" and "table" respectively for the Cassandra API.

The advantage of this method is that it is a straightforward turnkey way to manage throughput capacity on the database. However, the disadvantage is that in many cases, your approach to scaling may require certain levels of automation to be both cost-effective and high performing. The next sections explain the relevant scenarios and methods.

## <a id="use-control-plane"></a>Use the control plane

The Azure Cosmos DB's API for Cassandra provides the capability to adjust throughput programmatically by using our various control-plane features. See the [Azure Resource Manager](manage-cassandra-with-resource-manager.md), [Powershell](powershell-samples-cassandra.md), and [Azure CLI](cli-samples-cassandra.md) articles for guidance and samples.

The advantage of this method is that you can automate the scaling up or down of resources based on a timer to account for peak activity, or periods of low activity. Take a look at our sample [here](https://github.com/Azure-Samples/azure-cosmos-throughput-scheduler) for how to accomplish this using Azure Functions and Powershell.

A disadvantage with this approach may be that you cannot respond to unpredictable changing scale needs in real-time. Instead, you may need to leverage the application context in your system, at the client/SDK level, or using [Autopilot](provision-throughput-autopilot.md).

## <a id="use-cql-queries"></a>Use CQL queries with a specific SDK

You can scale the system dynamically with code by executing the [CQL ALTER commands](cassandra-support.md#keyspace-and-table-options) for the given database or container.

The advantage of this approach is that it allows you to respond to scale needs dynamically and in a custom way that suits your application. With this approach, you can still leverage the standard RU/s charges and rates. If your system's scale needs are mostly predictable (around 70% or more), using SDK with CQL may be a more cost-effective method of auto-scaling than using Autopilot. The disadvantage of this approach is that it can be quite complex to implement retries while rate limiting may increase latency.

## <a id="use-autopilot"></a>Use Autopilot

In addition to manual or programmatic way of provisioning throughput, you can also configure Azure cosmos containers in Autopilot mode. Autopilot mode will automatically and instantly scale to your consumption needs within specified RU ranges without compromising SLAs. To learn more, see the [Create Azure Cosmos containers and databases in autopilot mode](provision-throughput-autopilot.md) article.

The advantage of this approach is that it is the easiest way to manage the scaling needs in your system. It guarantees not to apply rate-limiting **within the configured RU ranges**. The disadvantage is that, if the scaling needs in your system are predictable, Autopilot may be a less cost-effective way of handling your scaling needs than using the bespoke control plane or SDK level approaches mentioned above.

## Next steps

- Get started with [creating a Cassandra API account, database, and a table](create-cassandra-api-account-java.md) by using a Java application
