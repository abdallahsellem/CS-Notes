### normal index scan failure 
- sometimes database can do a full scan for a query despite i am querying on the primary key 
	- for Example :
		`Select Name from students where id > 100`
		if the table has 50 million row so the query optimzer will know if he do an index scan it will do random access to like go and find all indexes greater than 100 and for each index get the page it contain so he will hit the index and then hit the disk for each row and this will be very slow , so he somehow smart and just doing full table scan to get the pages faster 
### Bitmap index scan :
 if i have an index on column like **grades** and i want to get all student that have grade >10 , and my table have 100 million row , if we do normal index scan like we explained in [[#normal index scan failure]] it will take much much time , so we need to do alternative way to optimize the query and this situation **bitmap index scan** become useful 

bitmap is an array of bits where each bit reference a page number , so the database will do index scan but won't hit the disk to get the page each time , it will change the value of the bitmap if it found an row that belong that page a bit represent 
for example  :
	it found row 43 its grade greater than 10 and this row in page number 5 , so he will change bit number 5 in the bitmap to 1 
after getting the rows that greater than  10  and sets their page bits in the bitmap , it will go and fetch all pages that are setted in the bitmap once , so most of us will think some of these pages will contain grades will be less than 10 so the database re-check it again to discard all rows that less than 10  and return the correct results 

##### beautiful example of bitmap operations :
suppose we have an index on **ID** and another index on **grades** 
given this 
` select name from student where id >100 and grades < 50 `

database will do two index scan 
1) id index scan 
2) grade index scan 
and a bitmap generated for each index 
so the final result will be the **AND** bit operation on the two bitmap ("Genius idea") so we are discarding bits that aren't exist in the two bitmaps 
![[Pasted image 20240703212059.png]]


### Key vs Non-key Column indexing 

Before diving into indexing, let's clarify the terms:

- **Key columns:** These are the columns used to build the index structure. They determine the order of rows in the index.
- **Non-key columns:** These are additional columns included in a non-clustered index using the `INCLUDE` clause. They are not part of the index structure but are stored with the index for faster retrieval.

#### Key Column Indexing

- **Clustered index:** This is a special type of index where the physical order of rows on disk matches the order of the index key. There can only be one clustered index per table.
- **Non-clustered index:** This type of index creates a separate structure pointing to the actual data rows. Multiple non-clustered indexes can exist on a table.

**Key column indexing is crucial for:**

- **Ordering data:** Clustered indexes define the physical order of data.
- **Searching for specific values:** Both clustered and non-clustered indexes excel at finding rows based on key column values.
- **Range queries:** Finding data within a specific range of key column values.

#### Non-Key Column Indexing

- **Included columns:** These are added to a non-clustered index using the `INCLUDE` clause.
- **Purpose:** To reduce the number of data page lookups needed to satisfy a query.

**Non-key column indexing is beneficial when:**

- **Frequently accessed columns:** Including frequently used columns in the index can improve query performance.
- **Covering indexes:** If all columns required by a query are included in the index, the query can be resolved without accessing the underlying table (covering index).
- **Reducing I/O:** By minimizing data page lookups, non-key columns can significantly enhance query performance.

#### Tips and Tricks

- **Identify frequently accessed columns:** Analyze query performance to determine which columns are bottlenecks.
- **Create indexes on columns used in WHERE, JOIN, ORDER BY, and GROUP BY clauses.**
- **Consider index cardinality:** High cardinality (many distinct values) is generally good for indexing.
- **Avoid over-indexing:** Too many indexes can degrade performance.
- **Monitor index usage:** Use tools like query plans to assess index effectiveness.
- **Update statistics regularly:** Accurate statistics help the query optimizer choose the best execution plan.
- **Use filtered indexes:** Create indexes on a subset of rows to improve performance for specific queries.
- **Leverage included columns:** Optimize non-clustered indexes by including necessary columns.
- **Test and refine:** Create and test different index configurations to find the optimal solution.

#### Example

Consider a table `Orders` with columns `OrderID`, `CustomerID`, `OrderDate`, `TotalAmount`, `ShipAddress`.

- **Clustered index:** Create a clustered index on `OrderID` to efficiently access orders by their ID.
- **Non-clustered index:** Create a non-clustered index on `CustomerID` for efficient customer-based queries.
- **Non-clustered index with included columns:** Create a non-clustered index on `OrderDate` with `TotalAmount` included to quickly calculate total sales by date without accessing the base table.

By carefully applying these tips and understanding the concepts of key and non-key column indexing, you can significantly enhance the performance of your database queries.

### Cluster vs Non-Cluster Index
**Imagine a Library**

Think of a library. The books are the data, and you want to find a specific book quickly.

**Clustered Index**: This is like the library itself. The books are arranged on the shelves in a specific order (based on the Dewey Decimal System or similar). This order is the clustered index. If you know the exact location (shelf and book number), you can find the book directly. There can only be one way to arrange all the books on the shelves (one clustered index per library).

**Non-Clustered Index:** This is like the library catalog. It contains information about the books (title, author, subject) and a reference to the book's location. You can use the catalog to quickly find the book's location and then go to the shelves. You can have multiple catalogs (non-clustered indexes) for different ways to find books (author, title, subject).
Key Differences

**Order**: Clustered indexes define the physical order of the data, while non-clustered indexes create a separate structure that points to the data.
**Uniqueness**: A table can have only one clustered index, but multiple non-clustered indexes.
**Performance**: Clustered indexes are generally faster for range queries (finding data within a specific range), while non-clustered indexes are better for equality searches (finding specific values).

In Summary

**Clustered Index:** The data itself is organized based on the index.
**Non-Clustered Index**: An extra lookup is needed to find the data after using the index.

### Bloom Filters :
#### The Concept

A Bloom filter is a probabilistic data structure used to determine whether an element is a member of a set. It consists of a bit array of m bits, all initially set to 0, and k different hash functions.

#### Insertion

To insert an element x into the Bloom filter:

1. Apply k hash functions to x, producing k hash values.
2. For each hash value h, set the bit at index h mod m to 1.

#### Membership Test

To test if an element x is possibly in the Bloom filter:

1. Apply the same k hash functions to x.
2. If all corresponding bits in the bit array are 1, x _might_ be in the set (false positive).
3. If at least one bit is 0, x is definitely not in the set.

#### Example

Let's say we want to create a Bloom filter with a bit array of size 8 (m = 8) and use two hash functions (k = 2):

- Hash function 1: h1(x) = x % 8
- Hash function 2: h2(x) = (x * 3 + 1) % 8

To insert the elements 3 and 5:

- For 3:
    - h1(3) = 3 % 8 = 3
    - h2(3) = (3 * 3 + 1) % 8 = 2
    - Set bits at indices 3 and 2 to 1.
- For 5:
    - h1(5) = 5 % 8 = 5
    - h2(5) = (5 * 3 + 1) % 8 = 4
    - Set bits at indices 5 and 4 to 1.

The bit array now looks like this:

```
01101100
```

To check if 6 is in the set:

- h1(6) = 6 % 8 = 6
- h2(6) = (6 * 3 + 1) % 8 = 7
- The bits at indices 6 and 7 are both 0, so 6 is definitely not in the set.

**Note:** A larger bit array and more hash functions will reduce the false positive rate.

### Index scan Vs Index only scan 
index only scan when we are query on an data that are already exist in the index , so there is no need to scan the physical disk to get the data from the table , we can return it directly from the index.
this can be done by using **include** in creating the index:
#### example on using include : 

**Query:**

```SQL
SELECT customer_id, order_date 
FROM orders 
WHERE customer_id = 123;
```

this query required an index scan because `order_date` wasn't in the index.

To enable an index-only scan, we can modify the index definition:

```SQL
CREATE INDEX idx_orders_customer_id_with_order_date
  ON orders (customer_id, total_amount)
  INCLUDE (order_date);
```
Now the index includes `customer_id`, `total_amount`, and `order_date`. The query can be executed using an index-only scan, significantly improving performance.
#### Example of Index Only Scan
**Table:** `orders`

- Columns: `order_id`, `customer_id`, `order_date`, `total_amount`
- Index: `idx_orders_customer_id` on `customer_id` and `total_amount`

**Query:**

```SQL

SELECT customer_id, total_amount 
FROM orders 
WHERE customer_id = 123;
```
#### Example of Index Scan

**Table:** `orders` (same as above)

- Index: `idx_orders_customer_id` on `customer_id` and `total_amount`

**Query:**


```SQL
SELECT customer_id, order_date 
FROM orders 
WHERE customer_id = 123;
```

### Postgres vs MySQL (B+Tree)

1. **PostgreSQL**:
    
    - **Secondary Indexes**: In PostgreSQL, a secondary index can point directly to the data row (tuple). This means that the size of the index entry is relatively small because it contains a direct pointer, typically a 32-bit or 64-bit address.
2. **MySQL**:
    
    - **Secondary Indexes**: In MySQL, especially InnoDB (the default storage engine), secondary indexes point to the primary key. Therefore, the size of the secondary index entries depends on the size of the primary key.
    - If the primary key is an integer, this isn't a significant issue because integers are relatively small.
    - However, if the primary key is a UUID (Universally Unique Identifier), the secondary index entries become much larger because UUIDs are 128-bit values.

#### Implications for Storage and Performance

1. **Bloat**: Larger primary keys, like UUIDs, cause secondary indexes to be larger. This can lead to:
    
    - Increased storage requirements.
    - More memory usage to store index entries.
    - Potential issues fitting all relevant data into memory, which is critical for performance.
2. **Write Amplification**:
    
    - **Write amplification** refers to the phenomenon where writing data causes multiple write operations due to the need to update multiple indexes.
    - This issue is exacerbated when secondary indexes are large because more data has to be written and updated with each insert or update operation.
### Degree of B+tree and Number of Key-pointer in a single page

the degree of a B+ tree (often called the order) does indeed relate to the number of key-pointer pairs that can be stored in a page. However, there are several mechanisms and strategies in place to ensure efficient use of space and optimize memory, even within this constraint.

#### Degree of B+ Tree and Page Usage

1. **Order of the B+ Tree**:
    
    - The order mmm of a B+ tree is the maximum number of children (pointers) an internal node can have.
    - For an internal node, this means it can store up to m−1m-1m−1 keys and mmm pointers.
    - For a leaf node, it can store up to m−1m-1m−1 key-pointer pairs.
2. **Page Size**:
    
    - The page size is typically fixed, such as 4 KB or 8 KB.
    - The number of key-pointer pairs that can fit in a page depends on the size of each key and pointer.

#### Free Space Management and Optimization

1. **Fill Factor**:
    
    - A B+ tree typically does not aim to fill every page completely. Instead, it uses a "fill factor" which is a target percentage of how full each node should be.
    - This allows for some free space within each page, making future insertions less likely to cause splits.
#### Example Calculation

Assume:

- Page size: 4 KB (4096 bytes)
- Key size: 8 bytes
- Pointer size: 8 bytes

For an internal node:

- Each entry (key-pointer pair) is 16 bytes.
- A page can hold up to 409616=256\frac{4096}{16} = 256164096​=256 entries.

For a leaf node:

- If the leaf node stores data pointers instead of direct data:
    - Each entry (key-data pointer pair) is 16 bytes.
    - A page can hold up to 409616=256\frac{4096}{16} = 256164096​=256 entries.

But considering a fill factor (say 70%):

- The target number of entries per page would be 256×0.7≈179256 \times 0.7 \approx 179256×0.7≈179.
### das