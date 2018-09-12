---
title: 'Create/insert operations against Azure CosmosDB Cassandra API from Spark | Microsoft Docs'
description: This document how to create/insert into tables in CosmosDB Cassandra API
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

# Create/insert operations against Azure CosmosDB Cassandra API from Spark

This document how to create/insert into tables in CosmosDB Cassandra API.

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

## 3. Dataframe API
Covers per record TTL, consistency setting, create ifNotExists while creating<br>

```scala<code>
// Generate a simple dataset containing five values
val booksDF = Seq(
   ("b00001", "Arthur Conan Doyle", "A study in scarlet", 1887),
   ("b00023", "Arthur Conan Doyle", "A sign of four", 1890),
   ("b01001", "Arthur Conan Doyle", "The adventures of Sherlock Holmes", 1892),
   ("b00501", "Arthur Conan Doyle", "The memoirs of Sherlock Holmes", 1893),
   ("b00300", "Arthur Conan Doyle", "The hounds of Baskerville", 1901)
).toDF("book_id", "book_author", "book_name", "book_pub_year")

booksDF.printSchema
booksDF.show
<br>

-><br>
booksDF:org.apache.spark.sql.DataFrame = [book_id: string, book_author: string ... 2 more fields]
root
 |-- book_id: string (nullable = true)
 |-- book_author: string (nullable = true)
 |-- book_name: string (nullable = true)
 |-- book_pub_year: integer (nullable = false)

+-------+------------------+--------------------+-------------+
|book_id|       book_author|           book_name|book_pub_year|
+-------+------------------+--------------------+-------------+
| b00001|Arthur Conan Doyle|  A study in scarlet|         1887|
| b00023|Arthur Conan Doyle|      A sign of four|         1890|
| b01001|Arthur Conan Doyle|The adventures of...|         1892|
| b00501|Arthur Conan Doyle|The memoirs of Sh...|         1893|
| b00300|Arthur Conan Doyle|The hounds of Bas...|         1901|
+-------+------------------+--------------------+-------------+

booksDF: org.apache.spark.sql.DataFrame = [book_id: string, book_author: string ... 2 more fields]
</code>
