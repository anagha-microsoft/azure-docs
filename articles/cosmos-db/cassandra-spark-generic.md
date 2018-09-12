---
title: 'Quickstart: Working with Azure CosmosDB Cassandra API from Spark | Microsoft Docs'
description: This document details fundamentals of connecting to CosmosDB Cassandra API from any Spark environment, and covers basic DDL and DML operations.
services: cosmos-db
author: anagha-microsoft
manager: kfile

ms.service: cosmos-db
ms.component: cosmosdb-cassandra
ms.custom: basics, DDL, DML
ms.devlang: spark-scala
ms.topic: quickstart
ms.date: 09/12/2018
ms.author: ankhanol

---

# Working with CosmosDB Cassandra API from Spark

Azure Cosmos DB is Microsoft's multi-model nosql database PaaS. With CosmosDB, you can provision a massively scalable no-sql instance with, choice of a supported model - document (SQL API, MongoDB API), key-value (Table API), graph (Gremlin API), and column-oriented (Cassandra API) with global replication in a matter of a minute, and get started. 

This document details fundamentals of working with the CosmosDB Cassandra API from any Spark environment - covers connectivity, basic DDL and DML operations in Spark-Scala.  Refer to service specific documentation for details of working with CosmosDB Cassandra API from Azure Databricks and HDInsight.

1.  Working with CosmosDB Cassandra API from Azure Databricks
2.  Working with CosmosDB Cassandra API from HDInsight-Spark

## 1. Prerequisites

1.  Provision a CosmosDB Cassandra API account
2.  Provision your choice of Spark environment

## 2. Dependencies for connectivity

**2.1.  Datastax Spark connector for Cassandra:**<BR>
Connectivity to CosmosDB Cassandra API is enabled via the Datastax Cassandra connector for Spark.  Identify and use the version of the connector at Maven central, that is compatible with the Spark and Scala versions of your Spark environment - https://mvnrepository.com/artifact/com.datastax.spark/spark-cassandra-connector

**2.2.  CosmosDB dependencies for the connector:**<BR>
From a CosmosDB perspective, we need two other classes from Microsoft in addition to the Datastax connector - a connection factory and custom retry policy. 
Ref: SPARKC-437.  We are in the process of publishing a jar on maven, in the meanwhile, the classes can be found at the links below.

  - CosmosDbConnectionFactory.scala - add link to Azure samples<br>
  - CosmosDbMultipleRetryPolicy.scala - add link to Azure samples<br>
    
   The retry policy for CosmosDB is configured to handle http status code 429 - "Request Rate Large" exceptions. The CosmosDB Cassandra    API, translates these exceptions to overloaded errors on the Cassandra native protocol, which we want to retry with back-offs. The reason for doing so is because CosmosDB follows a provisioned throughput model, and having this retry policy protects your spark jobs against spikes of data ingress/egress that would momentarily exceed the allocated throughput for your collection, resulting in the request rate limiting exceptions.

   Note - that this retry policy is meant to only protect your spark jobs against momentary spikes. If you have not configured enough RUs on your collection for the intended throughput of your workload such that the retries don't catch up, then the retry policy will result in rethrows.
    
**2.3.  CosmosDB instance details:**<BR>
 You will need the following-
        - CosmosDB Cassandra API account name
        - CosmosDB Cassandra API account endpoint
        - CosmosDB Cassandra API account key 
    
## 3. Connector specific throughput configuration

There are two areas of focus when tuning Spark integration with the CosmosDB Cassandra API.  The listing below details CosmosDB Cassandra API specific throughput configuration.  General information regarding this configuration can be found on the [Configuration Reference](https://github.com/datastax/spark-cassandra-connector/blob/master/doc/reference.md) page of the DataStax Spark Cassandra Connector github repository.
<table class="table">
<tr><th>Property Name</th><th>Description</th></tr>
<tr>
  <td><code>spark.cassandra.output.batch.size.rows</code></td>
  <td>Leave this to <code>1</code>. This is prefered for Cosmos DB's provisioning model in order to achieve higher throughput for heavy workloads.</td>
</tr>
<tr>
  <td><code>spark.cassandra.connection.connections_per_executor_max</code></td>
  <td><code>10*n</code><br/><br/>Which would be equivalent to 10 connections per node in an n-node Cassandra cluster. Hence if you require 5 connections per node per executor for a 5 node Cassandra cluster, then you would need to set this configuration to 25.<br/>(Modify based on the degree of parallelism/number of executors that your spark job are configured for)</td>
</tr>
<tr>
  <td><code>spark.cassandra.output.concurrent.writes</code></td>
  <td><code>100</code><br/><br/>Defines the number of parallel writes that can occur per executor. As batch.size.rows is <code>1</code>, make sure to scale up this value accordingly. (Modify this based on the degree of parallelism/throughput that you want to achieve for your workload)</td>
</tr>
<tr>
  <td><code>spark.cassandra.concurrent.reads</code></td>
  <td><code>512</code><br /><br />Defines the number of parallel reads that can occur per executor. (Modify this based on the degree of parallelism/throughput that you want to achieve for your workload)</td>
</tr>
<tr>
  <td><code>spark.cassandra.output.throughput_mb_per_sec</code></td>
  <td>Defines the total write throughput per executor. This can be used as an upper cap for your spark job throughput, and base it on the provisioned throughput of your Cosmos DB Collection.</td>
</tr>
<tr>
  <td><code>spark.cassandra.input.reads_per_sec</code></td>
  <td>Defines the total read throughput per executor. This can be used as an upper cap for your spark job throughput, and base it on the provisioned throughput of your Cosmos DB Collection.</td>
</tr>
<tr>
  <td><code>spark.cassandra.output.batch.grouping.buffer.size</code></td>
  <td>1000</td>
</tr>
<tr>
  <td><code>spark.cassandra.connection.keep_alive_ms</code></td>
  <td>60000</td>
</tr>
</table>

Regarding throughput and degree of parallelism, it is important to tune the relevant parameters based on the amount of load you expect your upstream/downstream flows to be, the executors provisioned for your spark jobs, and the throughput you have provisioned for your Cosmos DB account.
 
 ## 4. Connecting to the CosmosDB Cassandra API<BR>
 
 ### 4.1. Spark project/program-level:
 1.  Add the maven coordinates to the Datastax Cassandra connector for Spark
 2.  Add the two Scala classes above to your solution.
    
 ### 4.2. Individual class-level:
 #### (i) Imports:
 <code>//datastax Spark connector</code>
<code>import com.datastax.spark.connector._</code>
<code>import com.datastax.spark.connector.cql.CassandraConnector</code>

<code>//CosmosDB library for multiple retry</code>
<code>import com.microsoft.azure.cosmosdb.cassandra</code>
 
 #### (ii) Spark session configuration:
 The following are the various Spark session configuration you can set for connectivity and throughput configuration:
 <code>//Connection-related</code>
 <code>spark.conf.set("spark.cassandra.connection.host","YOUR_ACCOUNT_NAME.cassandra.cosmosdb.azure.com")</code>
 <code>spark.conf.set("spark.cassandra.connection.port","10350")</code>
 <code>spark.conf.set("spark.cassandra.connection.ssl.enabled","true")</code>
 <code>spark.conf.set("spark.cassandra.auth.username","YOUR_ACCOUNT_NAME")</code>
 <code>spark.conf.set("spark.cassandra.auth.password","YOUR_ACCOUNT_KEY")</code><br>
 <code>spark.conf.set("spark.cassandra.connection.factory", "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")</code>
 <code>//Throughput related</code><br>
 <code>spark.conf.set("spark.cassandra.output.batch.size.rows", "1")</code>
 <code>spark.conf.set("spark.cassandra.connection.connections_per_executor_max", "10")</code>
 <code>spark.conf.set("spark.cassandra.output.concurrent.writes", "1000")</code>
 <code>spark.conf.set("spark.cassandra.concurrent.reads", "512")</code>
 <code>spark.conf.set("spark.cassandra.output.batch.grouping.buffer.size", "1000")</code>
 <code>spark.conf.set("spark.cassandra.connection.keep_alive_ms", "600000000")</code>

 
 






