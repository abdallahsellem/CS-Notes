
## Tuple Oriented Pages 
### Record IDS 
each database assigns a unique id for each tuple that represent the location of this tuples  physically in disk 
and it consist of this 
-> file id + page id + slot number 
- SQLLite uses the Record id as a primary key 
- don't use Record id as primary key and this why : 
	- - In enterprise databases, data **moves** (e.g., during table reorganization, vacuuming, page splitting, clustering).
    
	- If you use record IDs as primary keys, you’d need to **update all foreign keys** referencing them when the record physically moves — that’s a nightmare!
#### Cons of Tuple Oriented :
1) Fragmentation 
2) Random Disk I/o : as when updating multiple tuples and each tuple exist in a different page so each page need to be fetched 
3) useless I/o : updating one tuple require fetching the entire page 
### Log Structure Storage

![[Pasted image 20250611205051.png]]

![[Pasted image 20250611205120.png]]

- MemTable Internal data-structure can be Skip-list or Balanced Tree 
- LSM are very fast in Write operations in database 
- LSM is slow for Read operations in database : 
	- **Scattered data** :Because data is written in immutable chunks (**SSTables**), **multiple SSTables might contain the same key** at different versions.

	- A read must **check multiple SSTables** (possibly across different “levels”).


#### LSM Databases :
![[Pasted image 20250611210321.png]]

## How NULL is Stored  :
![[Pasted image 20250611213535.png]]
## Word Alignment :
