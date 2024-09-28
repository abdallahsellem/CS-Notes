


### Page Table vs Page Directory :
**the page directory** is a mapping from page id to page locations in database files (physically)
**the page table** is a mapping from page ids to a copy of page in buffer pool frames (to know where the page located in memory)


### Pin Counter vs Latches :

**Pin Counter** are counters that used to indicate number of process is access a specific page so we can't remove it from buffer pool as it is still be accessed 

**Latches** : are (locks) that protect the whole data structure like the buffer pool so no concurrency problems can happen



### Buffer Pool Optimizations :
#### Multiple Buffer pool :
because latches can be a bottleneck in a concurrency systems , we need to solve this problem so we introduced a multiple-buffer pool which isolate quires of fetching some records fetching from others 
![[Pasted image 20240625144813.png]]

for example if i want a record number 7 lets then hash it and do mod to get which buffer pool contain it , this will lead if another process wants record number 8 it will be fetched from another buffer pool so now less latches (locks ) needed since process are isolated 



#### Pre-fetching 

##### Sequential 
it's a mechanism by which we are fetching pages that next to the current page i am currently reading as i'm expecting it will be read after the current page 
![[Pasted image 20240625151408.png]]


##### Index scan 
it's the same mechanism except we aren't fetching pages that are sequentially and physically located next to each other , like this image we are fetching pages that located next to each other on the leaf node of a p+tree as with high probability we need them in a range scan query like the image 
![[Pasted image 20240625151719.png]]
![[Pasted image 20240625151658.png]]




### Scan Sharing 
when there are two or more quires does the same thing like 
1- Select sum(*) from A 
2- Select Avg() from A 

we can make the first and the second queries have the same exactly pointer and after the first query finish there scan we can return back to the first page and scan pages that the second query didn't scan which will lead to faster and optimized scan as we didn't repeat the operations 
![[Pasted image 20240625153206.png]]

## Dirty Pages 







![[Pasted image 20240625160805.png]]

## OS Page Cache (direct I/O)
![[Pasted image 20240625161756.png]]

![[Pasted image 20240625162058.png]]