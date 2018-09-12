---
title: 'DDL operations against Azure CosmosDB Cassandra API from Spark | Microsoft Docs'
description: This document how to create keyspaces and tables in CosmosDB Cassandra API
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

# DDL operations against Azure CosmosDB Cassandra API from Spark

This document details keyspace DDL and table DDL constructs with CosmosDB Cassandra API.

## 1.  Imports
<code>import org.apache.spark.sql.cassandra.\_</code><br>
<code>//datastax Spark connector</code><br>
<code>import com.datastax.spark.connector._</code><br>
<code>import com.datastax.spark.connector.cql.CassandraConnector</code><br>

<code>//CosmosDB library for multiple retry</code><br>
<code>import com.microsoft.azure.cosmosdb.cassandra</code>

## 2. Configuration
<code>//Connection-related</code>
<code>spark.conf.set("spark.cassandra.connection.host","YOUR_ACCOUNT_NAME.cassandra.cosmosdb.azure.com")</code>
<code>spark.conf.set("spark.cassandra.connection.port","10350")</code>
<code>spark.conf.set("spark.cassandra.connection.ssl.enabled","true")</code>
<code>spark.conf.set("spark.cassandra.auth.username","YOUR_ACCOUNT_NAME")</code>
<code>spark.conf.set("spark.cassandra.auth.password","YOUR_ACCOUNT_KEY")</code><br>
<code>spark.conf.set("spark.cassandra.connection.factory", "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")</code><br>
<code>//Throughput-related...adjust as needed</code><br>
<code>spark.conf.set("spark.cassandra.output.batch.size.rows", "1")</code>
<code>spark.conf.set("spark.cassandra.connection.connections_per_executor_max", "10")</code>
<code>spark.conf.set("spark.cassandra.output.concurrent.writes", "1000")</code><br>
<code>spark.conf.set("spark.cassandra.concurrent.reads", "512")</code>
<code>spark.conf.set("spark.cassandra.output.batch.grouping.buffer.size", "1000")</code>
<code>spark.conf.set("spark.cassandra.connection.keep_alive_ms", "600000000")</code>

## 3.  Keyspace
### 3.1. Create keyspace:<br>

<code>//Cassandra connector instance</code><br>
<code>val cdbConnector = CassandraConnector(sc)</code><br><br>
<code>// Create keyspace</code><br>
<code>cdbConnector.withSessionDo(session => session.execute("CREATE KEYSPACE IF NOT EXISTS books_ks WITH REPLICATION = {'class': 'SimpleStrategy', 'replication_factor': 1 } "))</code>

