---
title: SQL Graph Database Sample | Microsoft Docs
description: A quick sample that will help you get started with the new syntax introduced in SQL graph database. 
ms.date: "04/19/2017"
ms.prod: sql
ms.prod_service: "database-engine, sql-database"
ms.reviewer: ""
ms.custom: ""
ms.technology: 
ms.topic: "language-reference"
helpviewer_keywords: 
  - "SQL graph"
  - "SQL graph, tsql reference"
ms.assetid: 
author: shkale-msft
ms.author: shkale
monikerRange: "=azuresqldb-current||>=sql-server-2017||=sqlallproducts-allversions||>=sql-server-linux-2017||=azuresqldb-mi-current"
---

# Create a graph database and run some pattern matching queries using T-SQL

[!INCLUDE[sqlserver2017-asdb](../../includes/applies-to-version/sqlserver2017-asdb.md)]

This sample provides a [!INCLUDE[tsql-md](../../includes/tsql-md.md)] script to create a graph database with nodes and edges and then use the new MATCH clause to match some patterns and traverse through the graph. This sample script will work on both Azure SQL Database and [!INCLUDE[sssqlv14](../../includes/sssqlv14-md.md)]  

## Sample Schema

This sample creates a graph schema, as showed in Figure 1, for a hypothetical social network that has People, Restaurant and City nodes. These nodes are connected to each other using Friends, Likes, LivesIn and LocatedIn edges.

![person-cities-restaurants-tables](../../relational-databases/graphs/media/person-cities-restaurants-tables.png "Sql graph database sample")  
Figure 1: Sample schema with restaurant, city, person nodes and LivesIn, LocatedIn, Likes edges.

## Sample Script

```
-- Create a graph demo database
CREATE DATABASE graphdemo;
go

USE  graphdemo;
go

-- Create NODE tables
CREATE TABLE Person (
  ID INTEGER PRIMARY KEY,
  name VARCHAR(100)
) AS NODE;

CREATE TABLE Restaurant (
  ID INTEGER NOT NULL,
  name VARCHAR(100),
  city VARCHAR(100)
) AS NODE;

CREATE TABLE City (
  ID INTEGER PRIMARY KEY,
  name VARCHAR(100),
  stateName VARCHAR(100)
) AS NODE;

-- Create EDGE tables. 
CREATE TABLE likes (rating INTEGER) AS EDGE;
CREATE TABLE friendOf AS EDGE;
CREATE TABLE livesIn AS EDGE;
CREATE TABLE locatedIn AS EDGE;

-- Insert data into node tables. Inserting into a node table is same as inserting into a regular table
INSERT INTO Person (Id, Person)
	VALUES (1, 'John')
		 , (2, 'Mary')
		 , (3, 'Alice')
		 , (4, 'Jacob')
		 , (5, 'Julie');

INSERT INTO Restaurant (Id, name, city)
	VALUES (1, 'Taco Dell','Bellevue')
		 , (2, 'Ginger and Spice','Seattle')
		 , (3, 'Noodle Land', 'Redmond');

INSERT INTO City (Id, name, stateName)
	VALUES (1,'Bellevue','wa')
		 , (2,'Seattle','wa')
		 , (3,'Redmond','wa');

-- Insert into edge table. While inserting into an edge table,
-- you need to provide the $node_id from $from_id and $to_id columns.

/* Insert which restaurants each person likes */
INSERT INTO likes 
	VALUES ((SELECT $node_id FROM Person WHERE ID = 1), (SELECT $node_id FROM Restaurant WHERE ID = 1), 9)
		 , ((SELECT $node_id FROM Person WHERE ID = 2), (SELECT $node_id FROM Restaurant WHERE ID = 2), 9)
		 , ((SELECT $node_id FROM Person WHERE ID = 3), (SELECT $node_id FROM Restaurant WHERE ID = 3), 9)
		 , ((SELECT $node_id FROM Person WHERE ID = 4), (SELECT $node_id FROM Restaurant WHERE ID = 3), 9)
		 , ((SELECT $node_id FROM Person WHERE ID = 5), (SELECT $node_id FROM Restaurant WHERE ID = 3), 9);

/* Associate in which city live each person*/
INSERT INTO livesIn 
	VALUES ((SELECT $node_id FROM Person WHERE ID = 1), (SELECT $node_id FROM City WHERE ID = 1))
		 , ((SELECT $node_id FROM Person WHERE ID = 2), (SELECT $node_id FROM City WHERE ID = 2))
		 , ((SELECT $node_id FROM Person WHERE ID = 3), (SELECT $node_id FROM City WHERE ID = 3))
		 , ((SELECT $node_id FROM Person WHERE ID = 4), (SELECT $node_id FROM City WHERE ID = 3))
		 , ((SELECT $node_id FROM Person WHERE ID = 5), (SELECT $node_id FROM City WHERE ID = 1));

/* Insert data where the restaurants are located */
INSERT INTO locatedIn 
	VALUES ((SELECT $node_id FROM Restaurant WHERE ID = 1), (SELECT $node_id FROM City WHERE ID =1))
		 , ((SELECT $node_id FROM Restaurant WHERE ID = 2), (SELECT $node_id FROM City WHERE ID =2))
		 , ((SELECT $node_id FROM Restaurant WHERE ID = 3), (SELECT $node_id FROM City WHERE ID =3));

/* Insert data into the friendOf edge */
INSERT INTO friendOf 
	VALUES ((SELECT $NODE_ID FROM Person WHERE ID = 1), (SELECT $NODE_ID FROM Person WHERE ID = 2))
		 , ((SELECT $NODE_ID FROM Person WHERE ID = 2), (SELECT $NODE_ID FROM Person WHERE ID = 3))
		 , ((SELECT $NODE_ID FROM Person WHERE ID = 3), (SELECT $NODE_ID FROM Person WHERE ID = 1))
		 , ((SELECT $NODE_ID FROM Person WHERE ID = 4), (SELECT $NODE_ID FROM Person WHERE ID = 2))
		 , ((SELECT $NODE_ID FROM Person WHERE ID = 5), (SELECT $NODE_ID FROM Person WHERE ID = 4));


-- Find Restaurants that John likes
SELECT Restaurant.name
FROM Person, likes, Restaurant
WHERE MATCH (Person-(likes)->Restaurant)
AND Person.name = 'John';

-- Find Restaurants that John's friends like
SELECT Restaurant.name 
FROM Person person1, Person person2, likes, friendOf, Restaurant
WHERE MATCH(person1-(friendOf)->person2-(likes)->Restaurant)
AND person1.name='John';

-- Find people who like a restaurant in the same city they live in
SELECT Person.name
FROM Person, likes, Restaurant, livesIn, City, locatedIn
WHERE MATCH (Person-(likes)->Restaurant-(locatedIn)->City AND Person-(livesIn)->City);
```

## Clean Up  
Clean up the schema and database created for the sample.

```
USE graphdemo;
go

DROP TABLE IF EXISTS likes;
DROP TABLE IF EXISTS Person;
DROP TABLE IF EXISTS Restaurant;
DROP TABLE IF EXISTS City;
DROP TABLE IF EXISTS friendOf;
DROP TABLE IF EXISTS livesIn;
DROP TABLE IF EXISTS locatedIn;

USE master;
go
DROP DATABASE graphdemo;
go
```

## Script explanation  
This script uses the new T-SQL syntax to create node and edge tables. Shows how to insert data into node and edge tables using `INSERT` statement and also shows how to use `MATCH` clause for pattern matching and navigation.

|Command	|Notes
|---  |---  |
|[CREATE TABLE &#40;Transact-SQL&#41;](../../t-sql/statements/create-table-sql-graph.md)  |Create graph node or edge table  |
|[INSERT &#40;Transact-SQL&#41;](../../t-sql/statements/insert-sql-graph.md)  |Insert into a node or edge table  |
|[MATCH &#40;Transact-SQL&#41;](../../t-sql/queries/match-sql-graph.md)  |Use MATCH to match a pattern or traverse through the graph  |
