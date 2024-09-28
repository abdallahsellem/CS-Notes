
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

### Hash Index :
A hash index as the name suggests is built on top of a hash table where all columns of the index are hashed and the hash value is used to lookup those keys. In MySQL, explicit hash indexes are only available for the memory storage engine and they are the default type for memory tables. The memory tables also support non-unique hash indexes as it stores values with the same hash (in case of collisions) in the same key as a linked list that it will have to examine each element in order to find a certain value.

To understand this better, let's assume we have the following table:

![3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618660478332/sWaoOywkFN.png?auto=compress,format&format=webp)

And let's say we define a hashed index on the `fname` column using some hash function

that produces the following values:

![4.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618660540113/qOE9PoDHO.png?auto=compress,format&format=webp)

Then the hash index structure will look as follows:

![5.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618660562989/OV86pUqwG.png?auto=compress,format&format=webp)

So when we execute a query with `WHERE fname = 'Peter'`, it will calculate the hash of `Peter` using the same hash function, so

and then it will look up the index and find a pointer to row 3, then it will fetch the row.
#### hash index limitations :
- Because the index only has the hash values and pointers to rows, MySQL will always have to hit the disk to find the rows as it can't find the needed values in the index as opposed to queries on values that are covered by a `B-tree` index (more on this when we discuss covering indexes).
- As we can see, the hash values are random and thus they are not sorted, so a hashed index can't be used to sort results in an `ORDER BY` clause.
- Hash indexes don't support partial key matching because the hash is calculated on all columns of the index, so for example, if the index has columns `A` and `B`, the hash value is
- so we can't find rows using only `A` or only `B`, it has to use both `A` and `B`. So in case queries that use only part of the index are important, then a new index has to be created for those columns separately.
- Hash indexes can't support partial matches one column, so for example, it it's not helpful for `LIKE` clauses because the hash is calculated on the full length of the value (or some part of it, but it's a fixed part anyway).
- Hash indexes only support equality conditions, so only `=`, `IN`, `<=>` but it can't do `<` or `>` and so on, because the values are not sorted as we mentioned before.
- Accessing data in the hash index can be very quick, but in case of having an index on a column of low selectivity (very few distinct values) many collisions can occur which can cause values to be chained in a linked list. In this case when the hash key matches a linked list, all elements of the linked will have to be examined until a match is found in the worst case.
- Some index maintenance operations can also be very slow in case of having too many collisions.
### General Notes :
1) **query optimizer** chooses that index which eliminates the greatest number of rows or scans as few rows as possible for better efficiency.
- Why Not defining a secondary index on each column we are interesed about instead of composite index 
1) _MySQL uses only one index per table per query except for UNION._ (In a UNION, each logical query is run separately, and the results are merged.) So defining multiple indices on multiple columns does not guarantee those indices will be used even if they are part of the query.
1) The storage engine can't optimize access to any columns to the right of the first range condition, so for example:
	```
	WHERE last_name="Smith" AND first_name LIKE 'J%' AND date_of_birth='1976-12-23'
	```
	
	the index will not be able to optimize access for the `date_of_birth` part of the condition because of the range query on the `first_name`.
1) some storage engines such InnoDB construct hash indexes automatically when it detects that some parts of a `B-tree` index are being accessed very frequently which is called an **Adaptive Hash Index**. It's also easy to construct your own hash index even if you are not using memory tables as MySQL supports a variety of hash functions and `TRIGGERS` that can be used to perform index maintenance operations


