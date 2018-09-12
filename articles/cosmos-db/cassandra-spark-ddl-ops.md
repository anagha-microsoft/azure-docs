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

Validate in cqlsh:<br>
<code>DESCRIBE keyspaces;</code><br>
You should see the keyspace created.

### 3.2. Alter keyspace:<br>
Currently not supported.

### 3.3. Drop keyspace:<br>
sc, below, refers to spark context<br>
<code>val cdbConnector = CassandraConnector(sc)</code><br>
<code>cdbConnector.withSessionDo(session => session.execute("DROP KEYSPACE books_ks"))</code><br>
  
Validate in cqlsh:<br>
<code>DESCRIBE keyspaces;</code>

## 4.  Table
CosmosDB specific 'must know's:<br>
  -Throughput can be assigned at a table level as part of the create table statement.<br>
  -One partition key can store 10 GB of data<br>
  -One record can be max of 2 MB in size<br>
  -One partition key range can store multiple partition keys<br>

### 4.1. Create table:<br>
<code>val cdbConnector = CassandraConnector(sc)</code>

<code>cdbConnector.withSessionDo(session => session.execute("CREATE TABLE IF NOT EXISTS books_ks.books(book_id TEXT PRIMARY KEY,book_author TEXT, book_name TEXT,book_pub_year INT,book_price FLOAT) WITH cosmosdb_provisioned_throughput=4000 , WITH default_time_to_live=630720000;"))</code>

Validate in cqlsh:<br>
<code>USE books_ks;</code>
<code>DESCRIBE books;</code>

Note: Provisioned throughput and default TTL is not visible in the output above.  You can view the throughput in the portal.

### 4.1. Alter table:<br>
**Note:**<br>
(1) Alter table - add/change columns - on the roadmap<br>
(2) Alter provisioned throughput - supported<br>
(3) Alter table TTL - supported<br>
(4) Alter table throughput - supported<br><br>

<code>val cdbConnector = CassandraConnector(sc)</code><br>
<code>cdbConnector.withSessionDo(session => session.execute("ALTER TABLE books_ks.books WITH cosmosdb_provisioned_throughput=8000, WITH default_time_to_live=0;"))</code>

