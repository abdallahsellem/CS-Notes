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


### Clustering Index :
#### definition: 
a **Clustering Index** is a file used if the records are physically ordered on a non-key field means that it does not have a distinct value for each record
- example: 

| 1 | sda |
| ---- | ---- |
| 1 | das |
| 2 | xxz |
| 2 | fsd |
| 3 | asda |
we can make a Clustering Index on id Field 
- A clustering index is another example of a nondense index because it has an entry
for every distinct value of the indexing field, which is a nonkey by definition and
hence has duplicate values rather than a unique value for every record in the file



#### use cases :

Example 2. Suppose that we consider the same ordered file with r = 300,000
records stored on a disk with block size B = 4,096 bytes. Imagine that it is ordered by
the attribute Zipcode and there are 1,000 zip codes in the file (with an average 300
records per zip code, assuming even distribution across zip codes.) The index in this
case has 1,000 index entries of 11 bytes each (5-byte Zipcode and 6-byte block
pointer) with a blocking factor bfri = ⎣(B/Ri)⎦ = ⎣(4,096/11)⎦ = 372 index entries per
block. The number of index blocks is hence bi = ⎡(ri/bfri)⎤ = ⎡(1,000/372)⎤ = 3 blocks.
To perform a binary search on the index file would need ⎡(log2 bi)⎤ = ⎡(log2 3)⎤ = 2
block accesses. Again, this index would typically be loaded in main memory (occu-
pies 11,000 or 11 Kbytes) and takes negligible time to search in memory. One block
access to the data file would lead to the first record with a given zip code.

- there is two methods to make Clustering Index :
	- ![[Pasted image 20240217155047.png]]
	- ![[Pasted image 20240217155111.png]]
-
### Secondary Indexes :
#### definition: 
- Secondary index is a file used if the records of a specific field is unordered and not unique 
- Many secondary indexes (and hence, indexing fields) can be created for the same file each represents an additional means of accessing that file based on some specific field.
- there is one index entry for each record in the data file, which contains the value of the field for the record and a pointer either to the block in which the record is stored or to the record itself. Hence, such an index is dense
- Because the records of the data file are not physically ordered by values of the secondary key field, we cannot use block anchors. That is why an index entry is created for each record in the data file, rather than for each block, as in the case of a primary index.
##### why Secondary index : 
A secondary index usually needs more storage space and longer search time than
does a primary index, because of its larger number of entries. However, the improve-
ment in search time for an arbitrary record is much greater for a secondary index
than for a primary index, since we would have to do a linear search on the data file
if the secondary index did not exist

- Example :
	- ![[Pasted image 20240217155705.png]]
	- **A binary search on this secondary index needs ⎡(log2 bi)⎤ = ⎡(log21,099)⎤ = 11 block**
	- ![[Pasted image 20240217155727.png]]
	- in secondary index there are numerous records have the same value so there are 3 possible solutions to solve this :
		- include duplicate index entries with the same K(i) value—one for each record. This would be a **dense index**.
			- example of <ki,pi> :
				- <1,4>
				- <1,5>
				- <1,7>
		- have variable-length records for the index entries, with a repeating field for the pointer. We keep a list of pointers <P(i, 1), … , P(i, k)> in the index entry for K(i)—one pointer to each block that contains a record whose indexing field value equals K(i). In either option 1 or option 2, the binary search algorithm on the index must be modified appropriately to account for a variable number of index entries per index key value
			- example of <ki,pi> :
				- <1,[3,4,5,2]> 
				- <4,[6,9,10,12]>
				- every index entry can have multiple pointers to each repeated value in the field 
		- create an extra level of indirection to handle the multiple pointers. In this nondense scheme, the pointer P(i) in index entry <K(i), P(i)> points to a disk block, which contains a set of record pointers; each record pointer in that disk block points to one of the data file records with value K(i) for the indexing field. If some value K(i) occurs in too many records, so that their record pointers cannot fit in a single disk block, a cluster or linked list of blocks is used. This technique is illustrated in Below Figure . Retrieval via the index requires one or more additional block accesses because of the extra level.
			- ![[Pasted image 20240217164344.png]]



### summary :
- **dense** mean that each index entry pointer to each record entry 
- **Anchor** mean that we have pointer to the first record of the block only 
![[Pasted image 20240217164629.png]]