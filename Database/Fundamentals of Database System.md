# Chapter 7 (indexing):

## What's Indexing : 
- **index** is just a file  which has pointers to blocks of Storage (provide access to the data without affect the real order or physical order of data on the disk )
- Types of Indexes is 
	- single level indexes : 
		- primary, secondary, and clustering
	- multilevel indexes 
- B+ trees have become a commonly accepted default structure for generating indexes on demand in most relational DBMSs
## types of Single ordered indexes :

- Primary index :
	- A primary index is specified on the ordering key field of an ordered file of records
		- ordering key field : is the field which data are ordered on disk based on it , it automatically assigned by the dbms 
- clustering index :
	- If the ordering field is not a key field—that is, if numerous records in the file can have the same value for the ordering field— another type of index, called a clustering index
	- Notice that a file can have at most one physical ordering field, so it can have at most one primary index or one clustering index, but not both 
- Secondary Index : 
	-  a secondary index, can be specified on any nonordering field of a file. A data file can have several secondary indexes in addition to its primary access method
### Primary Index :
- A **primary index** is an ordered file whose records are of fixed length with two fields, and it acts like an access structure to efficiently search for and access the data records in a data file. 
- The first field is of the same data type as the ordering key field—called the primary key—of the data file, and the second field is a pointer to a disk block (a block address)
- the two field values of index entry i as `<K(i), P(i)>` 
- there are different types of index entries < K (i), X > as follows:
	- X may be the physical address of a block (or page) in the file, as in the case of P(i) above.
	- X may be the record address made up of a block address and a record id (or offset) within the block.
- Example of Primary Index : 
	- The first three index entries are as follows:
		- <K(1) = (Aaron, Ed), P(1) = address of block 1> 
		- <K(2) = (Adams, John), P(2) = address of block 2> 
		- <K(3) = (Alexander, Ed), P(3) = address of block 3>
	- ![[Pasted image 20240215144205.png]]
- The index file for a primary index occupies a much smaller space than does the data file, for two reasons. 
	- First, there are fewer index entries than there are records in the data file. 
	- Second, each index entry is typically smaller in size than a data record because it has only two fields, both of which tend to be short in size; consequently, more index entries than data records can fit in one block. 
- Therefore, a binary search on the index file requires fewer block accesses than a binary search on the data file
- Example on a power of Indexing :
	- ![[Pasted image 20240215144449.png]]
- **A major problem with a primary index**—as with any ordered file :
	- is insertion and deletion of records. With a primary index, the problem is compounded because if we attempt to insert a record in its correct position in the data file, we must not only move records to make space for the new record but also change some index entries, since moving records will change the anchor records of some blocks. Using an unordered overflow file