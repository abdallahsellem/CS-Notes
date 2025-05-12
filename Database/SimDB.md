## A Lightweight Database Management System

### Overview

SimpleDB is a lightweight, file-based database management system (DBMS) implemented in C++. It is designed to provide a basic understanding of database internals, including file storage, indexing using B-Trees, and query execution.

### Features

- **B-Tree Indexing**: Efficient indexing for fast lookups.
    
- **File-based Storage**: Data is stored and retrieved from binary files.
    
- **Simple Query Execution**: Supports basic CRUD (Create, Read, Update, Delete) operations.
    
- **Transaction Support** (Basic): Implements simple write-ahead logging (WAL) for durability.
    
- **Command-Line Interface**: Provides an interactive shell to execute queries.
    

### Prerequisites

- C++ compiler (GCC/Clang)
    
- CMake (for build automation)
    

#### Build Instructions

```
mkdir build && cd build
cmake ..
make
```


### Running the Database

```
./simpledb
```

### Example Commands

```
INSERT INTO users (id, name, age) VALUES (1, 'Alice', 25);
SELECT * FROM users;
DELETE FROM users WHERE id = 1;
```

## Project Structure

```
SimpleDB/
│── src/
│   ├── main.cpp          # Entry point
│   ├── storage.cpp       # Handles file-based storage
│   ├── btree.cpp         # B-Tree indexing implementation
│   ├── parser.cpp        # SQL query parsing
│   ├── executor.cpp      # Executes parsed queries
│── include/
│   ├── storage.h
│   ├── btree.h
│   ├── parser.h
│   ├── executor.h
│── data/                 # Stores database files
│── tests/                # Unit tests
│── CMakeLists.txt        # Build configuration
│── README.md             # Project documentation
```

### **Timeline and Tasks**

#### **Week 1: Project Setup and Storage System**

- Set up the project structure with CMake.
    
- Implement a basic file storage system to store and retrieve data.
    
- Create binary file format for storing tables.
    
- Implement functions to write and read records from files.
    
- Test file storage module.
    

#### **Week 2: B-Tree Indexing Implementation**

- Implement a simple B-Tree structure for indexing.
    
- Integrate B-Tree with the file storage system.
    
- Implement insert, search, and delete operations in the B-Tree.
    
- Test indexing performance with sample datasets.
    

#### **Week 3: Query Parsing and Execution**

- Implement a simple SQL query parser.
    
- Support basic SQL commands: `INSERT`, `SELECT`, `DELETE`.
    
- Implement an execution engine that interacts with storage and indexing.
    
- Write unit tests for query execution.
    

#### **Week 4: Optimization and Finalization**

- Implement simple transaction logging (Write-Ahead Logging - WAL).
    
- Optimize query execution for performance.
    
- Improve error handling and validation.
    
- Final testing and debugging.
    
- Write documentation and prepare for project presentation.
### Project structure in details 

#### 1. file format :

+-------------------+-------------------+----------------+
| Header (Fixed)   | Data Records (Variable Length)     |
+-------------------+-------------------+----------------+

- **header example :** 
	-  **Magic Number (4 bytes)** – Identifier to validate the file (e.g., `"SDB1"`).
	- **Number of Records (4 bytes)** – Keeps track of stored records.
	- **Schema Info (Variable Size or Fixed)** – Defines column structure.
	- **Free Space Offset (4 bytes)** – Points to the next available location in the file.