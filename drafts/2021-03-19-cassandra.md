- ORDER BY

WITH CLUSTERING ORDER BY (empno ASC);

p2p column based
all nodes hold data and can answer rw query

ap system (a>c)

### hinted handoff
if a machine is down, missing data is replayed via "hinted handoff"

### consistency levels 
For any read or write request
how many replicas do I need to hear when I do a read or a write for that read or write is considered successful:
- ALL: all replica
- QUORUM: majority of replicas (e.g. 2 out of 3)
- ONE: one replica suffice

coordinator (node that recieves write)

Single column slice restrictions are allowed only on the last clustering column being restricted.

---
# Limitations in Cassandra query

A primary key in Cassandra is composed of partition key and clustering key. Both can be multiple key:

e.g.
```
PRIMARY KEY (partition_key, clustering_key)
PRIMARY KEY ((partition_key1, partition_key2), clustering_key)
PRIMARY KEY ((partition_key1, partition_key2), clustering_key1, clustering_key2)
```

## Basic Query
### 1. Always provide partition key
The partition key should be provided so that Cassandra knows where to find the necessary data. Otherwise, you should add `ALLOW FILTERING` to enable the query.

## Range Query
### 2. Range query only works on "last specified" clustering key
Cassandra accepts range query on clustering keys `k` only if
1. Partition key is specified.
2. All clustering keys in front of `k` are specified.

e.g.
```
CREATE TABLE tbl (
  partition_key int,
  clustering_key1 int,
  clustering_key2 int,
  PRIMARY KEY (partition_key, clustering_key1, clustering_key2)
)

SELECT * FROM tbl WHERE partition_key = 1 AND clustering_key1 > 2;
-- work!

SELECT * FROM tbl WHERE partition_key = 1 AND clustering_key1 = 2 AND clustering_key2 > 3;
-- work!

SELECT * FROM tbl WHERE partition_key = 1 AND clustering_key2 > 3;
-- rejected!

SELECT * FROM tbl WHERE partition_key = 1 AND clustering_key1 > 2 AND clustering_key2 = 3;
-- rejected!
```

## Sort Query
### 3. Sorting the data is only enable for clustering keys

By default, Cassandra keey the data sorted ascendingly by clustering key **on disk**. Or you can specify the order by `WITH CLUSTERING ORDER BY`.

e.g.
```
CREATE TABLE ... (
    ...
) WITH CLUSTERING ORDER BY (clustering_key1 DESC, clustering_key2 ASC, clustering_key3 ASC)
```

### 4. Sorting is only guaranteed within the same partition
If you query data from multiple partitions, the query result is not sorted.

## Secondary Index
Under the hood secondary index in Cassandra stores the corresponding primary key of each indexed value into a internal table.
So when a query only specify the indexed field, Cassandra can know which partition to find the data we need. 

### 5. Secondary index only support =,




## Reference
- [Partition Key vs Composite Key vs Clustering Columns in Cassandra](https://www.bmc.com/blogs/cassandra-clustering-columns-partition-composite-key/)
- [Where and Order By Clauses in Cassandra CQL](https://stackoverflow.com/questions/35708118/where-and-order-by-clauses-in-cassandra-cql)
- [A deep look at the CQL WHERE clause](https://www.datastax.com/blog/deep-look-cql-where-clause)
- [Cassandra Native Secondary Index Deep Dive](https://www.datastax.com/blog/cassandra-native-secondary-index-deep-dive)
- [7 mistakes when using Apache Cassandra](https://blog.softwaremill.com/7-mistakes-when-using-apache-cassandra-51d2cf6df519)




---

Backlog
Planning progress- Revenue