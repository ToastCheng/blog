
mongo index:
_id: 
default not a cluster index

single field: 
mongodb will just use one single field index to q

compound field:
when single field is not enough.

text:
like single field index, but with 
1. collation
2. case insensitive

geoindexes:
2d indexes: flat
2d sphere

multikey:
used is the value is an array.
for each key in the array there is an entry with its value.

e.g. N is the size of the array field, mongo will generate N different index keys pointing to the same document.

hash:
used in shard to generate random key and increase write performance.

sparse:
only index keys in documents that do have the field.

filter:
only index keys in documents that match the argument. e.g. { age: {$gte: 10} } 

unique:
unique! cannot use in sharded environment.


TTL (time to live):
mongo will run TTL round every minute to remove documents that has exceeds TTL.


partial:



### Shard keys:
[Doc](https://docs.mongodb.com/manual/core/ranged-sharding/)

`sh.shardCollection(<namespace>, <key>)`
e.g.
range-based sharding:
`sh.shardCollection(database.collection, { field: 1 })`

hashed sharding:
`sh.shardCollection(database.collection, { field: "hashed" })`

### Range-based sharding
e.g.
shard 1: min < field < 10
shard 2: 10  < field < 20
shard 3: 20  < field < max

Use it if the field is:
1. Large Shard Key Cardinality (number of distinct key value)
2. Low Shard Key Frequency     (no frequent key)
3. Non-Monotonically Changing Shard Keys (like timestamp)

### Hashed sharding
- More even data distribution than range-based sharding.
- More likely to broadcase operation for ranged query.
- Work with monotonically key.


"close shard key"

### Compound index, Prefix
index: { "item": 1, "location": 1, "stock": 1 }
has
prefix: { "item": 1 }, { "item": 1, "location": 1 }

### mongos
an interface between client and sharded cluster.

driver --> mongos --> shard A, shard B, shard C...

#### Targeted operation
Query with shard key, mongos can know which shard to ask.
- shard key or
- the prefix of a compound shard key

#### Broadcast operation
Query without shard key, mongos needs to ask for every shard.
Suffer from tail latency.
mongos merges the data for client to use.

All updateOne(), replaceOne() and deleteOne() operations must include the shard key or _id in the query document. MongoDB returns an error if these methods are used without the shard key or _id.



### Optimization
- createIndex()
- c
