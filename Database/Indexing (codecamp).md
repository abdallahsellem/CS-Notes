
### Cluster-Index :

- You can consider that a clustered index is a `B-Tree` index whose leaf nodes are the actual data blocks on disk, since the index & data reside together. This kind of index physically organizes the data on disk as per the logical order of the index key.


### Secondary Index 
- Secondary index is also maintained in the B+ Tree and it’s sorted as per the key on which the index was created.
- Secondary indices does not impact physical storage locations unlike primary indices.
- if the frequency of query a specific column is high so its possible we need index on that column to speed the retrieval 

#### How to create a secondary index in MySQL?
````sql
CREATE INDEX secondary_idx_1 ON index_demo (name);
````
![[Pasted image 20240515073407.png]]

#### structure of secondary index  :
- the secondary index has reference to the primary key’s address, although it’s not the case. Retrieving data through the secondary index means you have to traverse two B+ trees — one is the secondary index B+ tree itself, and the other is the primary index B+ tree.
- ![[Pasted image 20240515073510.png]]
- if a primary key is very large like a `URL`, since secondary indexes contain a copy of the primary key column value, it can be inefficient in terms of storage. More secondary keys means a greater number of duplicate copies of the primary key column value, so more storage in case of a large primary key. Also the primary key itself stores the keys, so the combined effect on storage will be very high
#### Consideration before you delete a Primary Index:

- In MySQL, you can delete a primary index by dropping the primary key. We have already seen that a secondary index depends on a primary index. So if you delete a primary index, all secondary indices have to be updated to contain a copy of the new primary index key which MySQL auto adjusts.
	

### Composite Index: 
#### how to create it :

```sql
CREATE INDEX composite_index_1 ON index_demo (phone_no, name, age);
```


#### how it works :
- The columns used in composite indices are concatenated together, and those concatenated keys are stored in sorted order using a B+ Tree. When you perform a search, concatenation of your search keys is matched against those of the composite index.


### Partial Index

We already know that Indices speed up our queries at the cost of space. The more indices you have, the more the storage requirement. We have already created an index called `secondary_idx_1` on the column `name`. The column `name` can contain large values of any length. Also in the index, the row locators’ or row pointers’ metadata have their own size. So overall, an index can have a high storage & memory load.

In MySQL, it’s possible to create an index on the first few bytes of data as well

```sql
CREATE INDEX secondary_index_1 ON index_demo (name(4));
```

#### How to identify if you need a composite index:

- Analyze your queries first according to your use cases. If you see certain fields are appearing together in many queries, you may consider creating a composite index.
- If you are creating an index in `col1` & a composite index in (`col1`, `col2`), then only the composite index should be fine. `col1` alone can be served by the composite index itself since it’s a left side prefix of the index.
- Consider cardinality. If columns used in the composite index end up having high cardinality together, they are good candidate for the composite index.
### General Notes :
1) **query optimizer** chooses that index which eliminates the greatest number of rows or scans as few rows as possible for better efficiency.
- Why Not defining a secondary index on each column we are interesed about instead of composite index 
1) _MySQL uses only one index per table per query except for UNION._ (In a UNION, each logical query is run separately, and the results are merged.) So defining multiple indices on multiple columns does not guarantee those indices will be used even if they are part of the query.