

## How b+ tree optimize Searching better than binary Search
### The Problem with Binary Search

1. **Binary Search Basics**:
    
    - Binary search works on sorted data and repeatedly divides the search space in half to find a target value.
    - In a database context, each "page" can be thought of as a fixed-size chunk of data, like a block of records.
2. **Example**:
    
    - Suppose you have a small 2MB database.
    - This database is divided into pages, each 4KB in size.
    - Thus, you have 2 MB4 KB=512\frac{2 \text{ MB}}{4 \text{ KB}} = 5124 KB2 MB​=512 pages.
3. **Logarithmic Search Complexity**:
    
    - Binary search on 512 pages has a worst-case scenario of log⁡2(512)=9\log_2(512) = 9log2​(512)=9 pages.
    - This means you might need to read up to 9 pages to find a record.
4. **Problem**:
    
    - Page fetches (reading a page from storage into memory) are slow.
    - Reading up to 9 pages just to find a single record can be inefficient.

### How SQLite Uses a B+ Tree

1. **B+ Tree Structure**:
    
    - A B+ tree is a type of balanced tree data structure where:
        - **Interior pages** (non-leaf nodes) contain pointers to child nodes and key ranges.
        - **Leaf pages** (leaf nodes) contain actual data records and are linked sequentially.
2. **Advantages**:
    
    - B+ trees reduce the number of page reads required to find a record.
    - They are optimized for database systems where the cost of reading from disk is high.
3. **Example**:
    
    - Imagine each leaf page in SQLite holds sandwich records.
    - Each record is 40 bytes, and with a primary key of 4 bytes, you can fit 100 records into one 4KB page.
4. **Adding Records**:
    
    - When you add the 101st record, it doesn't fit in the existing page, so SQLite splits the page into two leaf pages.
    - An interior page is then created to keep track of these leaf pages, acting as a root that points to them.
5. **Efficiency**:
    
    - Each entry in an interior page is around 8 bytes (4 bytes for the key and 4 bytes for the page number).
    - Therefore, a 4KB interior page can store pointers to 500 child pages.
6. **Scalability**:
    
    - For a 2MB database with 512 leaf pages, one interior page can keep track of all of them.
    - Instead of searching through multiple pages, you now search the interior page to find the relevant leaf page.
    - Worst-case search now involves reading only 2 pages: one interior and one leaf.

### Key Points Illustrated

1. **Logarithmic Complexity vs. B+ Tree**:
    
    - Binary search complexity: log⁡2(n)\log_2(n)log2​(n).
    - B+ tree complexity: Typically log⁡m(n)\log_{m}(n)logm​(n) where mmm is the branching factor (number of child nodes each interior node can point to).
2. **Example Scenario**:
    
    - Let's say you have a database with 10,000 records.
    - If records are 40 bytes each, you need 10,000×40 bytes4 KB≈100\frac{10,000 \times 40 \text{ bytes}}{4 \text{ KB}} \approx 1004 KB10,000×40 bytes​≈100 pages.
    - Binary search would require log⁡2(100)≈7\log_2(100) \approx 7log2​(100)≈7 page reads.
    - With a B+ tree where interior pages point to 500 children, you'd likely need just one interior page and one leaf page read, totaling 2 reads.

### Simplified Example

- **Binary Search**:
    
    - Suppose searching for a record among 512 pages: read 9 pages.
- **B+ Tree**:
    
    - Single interior page keeps track of 500 leaf pages.
    - Find the key range in the interior page, then go directly to the relevant leaf page.
    - Only 2 page reads.

### Tree Grow :

Since our new root can hold about 500 references to second-level interior pages and those second-level pages hold about 500 references to leaf pages, we can store about 250,000 pages, or about 1GB of data. If we continue adding rows, we’ll need to split the root page again and increase our tree depth to 4.

At a tree depth of 4, we can hold about 500³ leaf pages, or about a 476GB database. That means we only need to read 4 pages to find a given record—even in a huge database!
### Summary

SQLite uses a B+ tree structure to minimize the number of pages read from disk. Instead of searching through multiple pages in a binary search manner, a B+ tree allows for quick lookups by leveraging interior pages that store key ranges. This makes data retrieval much more efficient, especially as the database size grows.