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

Azure Cosmos DB is Microsoft's globally distributed multi-model nosql database PaaS. With CosmosDB, you can provision a massively scalable no-sql instance with, choice of a supported model - document (SQL API, MongoDB API), key-value (Table API), graph (Gremlin API), and column-oriented (Cassandra API) with global replication within a minute and get started. 

This document details fundamentals of connecting to CosmosDB Cassandra API from any Spark environment, and covers basic DDL and DML operations in Spark-Scala.  Refer to service specific documentation to learn specifics of working with CosmosDB Cassandra API from Azure Databricks and HDInsight.

1.  Working with CosmosDB Cassandra API from Azure Databricks
2.  Working with CosmosDB Cassandra API from HDInsight-Spark

## Prerequisites

1.  Provision a CosmosDB Cassandra API account (add link)
2.  Provision your choice of Spark environment

## Dependencies for connectivity

1.  **Connector:**
Connectivity to CosmosDB Cassandra API is enabled via the Datastax Cassandra connector for Spark.  Identify and use the version of the connector at Maven central, that is compatible with the Spark and Scala versions of your Spark environment - https://mvnrepository.com/artifact/com.datastax.spark/spark-cassandra-connector

2.  **CosmosDB:**
From a CosmosDB perspective, we need two other classes from Microsoft in addition to the Datastax connector - a connection factory and custom retry policy. 
Ref: SPARKC-437.  We are in the process of publishing a jar in maven, in the meanwhile, the classes can be found at the links below.

  - CosmosDbConnectionFactory.scala - add link to Azure samples<br>
  - CosmosDbMultipleRetryPolicy.scala - add link to Azure samples<br>
 
 3.  **Instance details:**
 You will need the CosmosDB Cassandra API account name, and the key (add link to get account name & key from portal).  
 
 






