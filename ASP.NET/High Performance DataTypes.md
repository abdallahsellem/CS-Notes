# High-Performance Data Structures in C\#

This document provides an overview of high-performance data structures in C# designed for memory efficiency and performance in scenarios like web servers, game development, or data processing. Each section details the core concept and where the structure itself and its referenced data are stored (stack, heap, or unmanaged memory).

## ArrayPool

- **Core Concept**: A class in System.Buffers that manages a pool of reusable arrays to minimize memory allocations and reduce garbage collection (GC) pressure.
    
- **Storage**:
    
    - **Structure Itself**: Stored on the **heap** (as a class). The internal pool metadata (e.g., bucket arrays) is also heap-allocated.
        
    - **Referenced Data**: Arrays rented via Rent are stored on the **heap**, reused across calls to reduce allocations.
        
- **Details**:
    
    - Provides Rent(int minimumLength) to acquire an array and Return(T[] array) to return it to the pool.
        
    - ArrayPool\<T>.Shared is a thread-safe, default pool for most use cases
	Rented arrays may contain stale data, requiring manual clearing if needed.
    
- **Use Cases**: Temporary buffers for I/O operations (e.g., reading streams or sockets).
    
- **Caveats**: Must manually return arrays to avoid memory leaks. Use clearArray = true in Return for sensitive data.
    

## Span

- **Core Concept**: A ref struct that represents a contiguous region of memory (e.g., array, stack, or unmanaged memory) for zero-copy, high-performance data access. As a ref struct, Span\<T> can only live on the stack (not the heap) and cannot be used in async methods, lambda expressions, or stored in fields.
    
- **Storage**:
    
    - **Structure Itself**: Stored on the **stack** (as a ref struct). It consists of a pointer (or reference) and a length.
        
    - **Referenced Data**:
        
        - **Heap**: When wrapping a portion of an array (e.g., array.AsSpan()).
            
        - **Stack**: When wrapping stackalloc-allocated memory (e.g., Span\<int> s = stackalloc int[10]).
            
        - **Unmanaged Memory**: When pointing to native memory (e.g., via pointers).
            
- **Details**:
    
    - Enables zero-copy operations, reducing memory and CPU overhead.
        
    - No additional heap allocations when slicing or manipulating.
        
- **Use Cases**: Slicing arrays or stack-allocated buffers for parsing without copying.
    
- **Caveats**: Stack-only nature limits use in async or long-lived scenarios.
    

## ReadOnlySpan

- **Core Concept**: A ref struct that provides read-only access to a contiguous region of memory, ensuring immutability for safe, high-performance data access. As a ref struct, ReadOnlySpan\<T> can only live on the stack (not the heap) and cannot be used in async methods, lambda expressions, or stored in fields.
    
- **Storage**:
    
    - **Structure Itself**: Stored on the **stack** (as a ref struct). Contains a pointer (or reference) and a length.
        
    - **Referenced Data**:
        
        - **Heap**: When wrapping arrays or strings (e.g., text.AsSpan()).
            
        - **Stack**: When wrapping stackalloc memory.
            
        - **Unmanaged Memory**: When pointing to native memory.
            
- **Details**:
    
    - Similar to Span\<T> but enforces read-only access.
        
    - Commonly used with strings, which are heap-allocated immutable objects.
        
- **Use Cases**: Parsing strings or binary data without modification.
    
- **Caveats**: Cannot modify data; use Span\<T> for mutation.
    

## Memory

- **Core Concept**: A struct (not a ref struct) that represents a managed, contiguous region of memory (e.g., an array or portion thereof) and can be stored on the heap or stack, used in async methods, or passed freely.
    
- **Storage**:
    
    - **Structure Itself**: Stored on the **stack** (as a local variable) or **heap** (if a field in a class or boxed). Holds a reference to the underlying memory and metadata (e.g., offset, length).
        
    - **Referenced Data**: Typically on the **heap** (e.g., when wrapping an array via array.AsMemory()).
        
- **Details**:
    
    - Suitable for async or long-lived scenarios, unlike Span\<T>.
        
    - Can be converted to Span\<T> via .Span for stack-based operations.
        
- **Use Cases**: Passing array segments to async methods.
    
- **Caveats**: Less performant than Span\<T> due to potential heap allocations.
    

## MemoryPool

- **Core Concept**: A class in System.Buffers that provides a pool of memory blocks (not limited to arrays) for high-performance memory management, with ArrayPool\<T> as a specific implementation.
    
- **Storage**:
    
    - **Structure Itself**: Stored on the **heap** (as a class). Internal metadata is heap-allocated.
        
    - **Referenced Data**: Typically on the **heap** (e.g., arrays in the default ArrayPool\<T>-based implementation). Custom pools could manage other memory types.
        
    - **IMemoryOwner**: The rented memory is wrapped in a heap-allocated IMemoryOwner\<T>, providing a Memory\<T> view.
        
- **Details**:
    
    - MemoryPool\<T>.Shared uses ArrayPool\<T> internally.
        
    - Dispose IMemoryOwner\<T> to return memory to the pool.
        
- **Use Cases**: Managing large buffers for async I/O.
    
- **Caveats**: More complex than ArrayPool\<T>; requires disposal.
    

## ValueListBuilder

- **Core Concept**: A ref struct in System.Buffers that provides a stack-based, growable list-like structure for building collections without heap allocations. As a ref struct, ValueListBuilder\<T> can only live on the stack (not the heap) and cannot be used in async methods, lambda expressions, or stored in fields.
    
- **Storage**:
    
    - **Structure Itself**: Stored on the **stack** (as a ref struct). Contains metadata and a reference to the backing storage.
        
    - **Referenced Data**:
        
        - **Stack**: When initialized with a stackalloc-allocated Span\<T>.
            
        - **Heap**: When growing beyond initial capacity, it rents a buffer from ArrayPool\<T>.
            
- **Details**:
    
    - Starts with stack-based storage but may use heap-allocated arrays for growth.
        
    - Must be disposed to return rented heap memory.
        
- **Use Cases**: Building temporary lists during parsing without heap allocations.
    
- **Caveats**: Stack-only restrictions; requires disposal for heap-allocated buffers.
    

## Stack and stackalloc

- **Core Concept**: Stack\<T> is a managed, heap-based, last-in-first-out (LIFO) collection in System.Collections.Generic, while stackalloc is a C# keyword for allocating memory directly on the stack for high-performance scenarios.
    
- **Storage**:
    
    - **Stack**:
        
        - **Structure Itself**: Stored on the **heap** (as a class). Internal state (e.g., capacity) is heap-allocated.
            
        - **Referenced Data**: Elements are stored in a heap-allocated internal array.
            
    - **stackalloc**:
        
        - **Structure Itself**: N/A (keyword, not a structure). Allocates memory directly on the **stack**.
            
        - **Referenced Data**: Stored on the **stack**, limited by stack size (typically 1 MB).
            
- **Details**:
    
    - Stack\<T>: General-purpose but less performant due to heap allocations.
        
    - stackalloc: Fast for small, short-lived buffers; can be wrapped in Span\<T>.
        
- **Use Cases**:
    
    - Stack\<T>: Algorithms like depth-first search.
        
    - stackalloc: Small, temporary buffers for performance-critical operations.
        
- **Caveats**:
    
    - Stack\<T>: Heap allocations increase GC pressure.
        
    - stackalloc: Risks stack overflow for large allocations.
        

## Summary Table

|   |   |   |
|---|---|---|
|Data Structure|Structure Storage|Referenced Data Storage|
|**ArrayPool**|Heap (class)|Heap (arrays)|
|**Span**|Stack (ref struct)|Heap (arrays), Stack (stackalloc), or Unmanaged|
|**ReadOnlySpan**|Stack (ref struct)|Heap (arrays/strings), Stack (stackalloc), or Unmanaged|
|**Memory**|Stack or Heap (struct)|Heap (arrays)|
|**MemoryPool**|Heap (class)|Heap (arrays, typically)|
|**ValueListBuilder**|Stack (ref struct)|Stack (stackalloc) or Heap (ArrayPool)|
|**Stack**|Heap (class)|Heap (internal array)|
|**stackalloc**|N/A (keyword)|Stack (allocated memory)|

## Key Notes

- **Stack-Based Structures**: Span\<T>, ReadOnlySpan\<T>, and ValueListBuilder\<T> are stack-allocated (ref struct), offering low overhead but restricted to synchronous, short-lived use.
    
- **Heap-Based Structures**: ArrayPool\<T>, MemoryPool\<T>, and Stack\<T> are heap-allocated, suitable for long-lived or async scenarios but with potential allocation costs.
    
- **Flexible Storage**: Memory\<T> can be stack- or heap-allocated, bridging synchronous and async use cases.
    
- **Referenced Data**: Most structures reference heap-allocated arrays (e.g., from ArrayPool\<T>), but Span\<T>, ReadOnlySpan\<T>, and ValueListBuilder\<T> can use stack-allocated (stackalloc) or unmanaged memory for specific scenarios.
    
- **Performance**: Stack-based storage minimizes allocations and GC pressure, while heap-based storage offers flexibility at the cost of potential overhead.