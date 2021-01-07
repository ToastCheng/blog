
```sql
SELECT user
FROM user_table
WHERE YEAR(create_at) = 2013;
```

see detail
```sql
EXPLAIN SELECT user
FROM user_table
WHERE YEAR(create_at) = 2013;
```

### type:
- const / eq_ref: find a single row that match filter (binary search)
  - primary key
  - unique key
- ref / RANGE: perform index range scan, find a starting point of a range in B+ tree. Scan via the doulby link list.
- INDEX (full index scan): scan through whole B+ tree's leaf nodes.
- ALL (full table scan): badly, load every columns into memory.


index inside a function (`YEAR(creat_at)`) cannot be treated as index.
Because database just cannot gerantee the output of that function

### Execution plan


index only scan
mysql put all index in memory, no disk IO

### Pitfull - multicolumn index order
multicolumn index
used from left to right

### Pitfull - inequality operator
database stop using index on inequality operator