# **Memory Pool Library**
---
## **Overview**
This library implements a **fixed-type**, **dynamic-size** memory pool system in C++. It enhances performance by optimizing memory allocation and deallocation through memory reuse, enabling efficient memory management. All memory pool's `creation`, `allocation`, and `deallocation` are handled exclusively through the `CMemoryPoolManager`.

For `allocation`, the project provides two approaches, **manual memory management** and **automatic memory management** using a custom [[Portfolios_Eng/03_Technical Notes/SharedPtr Library|CSharedPtr]].
> ⚠️ Note: The [[Portfolios_Eng/03_Technical Notes/SharedPtr Library|CSharedPtr]] used in the memory pool system is a slightly modified version of the standard one. 
- **Manual Memory Management**:
	- Allows explicit control over a specific memory pool and immediate deallocation, making it possible to optimize performance and manage memory flexibly.  
- **Automatic Memory Management**:
	- Uses [[Portfolios_Eng/03_Technical Notes/SharedPtr Library|CSharedPtr]] to safely share memory across different parts of the program, automatically freeing memory when it is no longer in use.

These two methods are designed to be used selectively based on the situation, offering flexibility in memory control as needed.

[**View Repository**](https://github.com/Woo95/MemoryPool)

---
## **Class Descriptions**
### `IMemoryPool Class`
- **Role**:
	- A base interface and **abstract class** for all memory pools. Since `CMemoryPoolManager` cannot manage `CStaticMemoryPool<T>` via `void*`, it relies on `IMemoryPool` to safely manage memory pools of `T type`.
- **Key Method**:
	- [[MemoryPool Library (Code)#^da8d6e|virtual ~IMemoryPool()]]:
		- Declared virtual to ensure proper destruction of derived classes like `CStaticMemoryPool<T>`. This allows destructors to be called correctly through a base class pointer.
### `CStaticMemoryPool<T> Class`
- **Role**:
	- A class that manages a **fixed-type** memory pool for objects of `T type`. A separate memory pool is created for each type.
- **Key Methods**:
	- [[MemoryPool Library (Code)#^c2ed89|void ExpandPool()]]:
		- The size of the memory pool is **dynamically resized**. The initial size can be set through the constructor of `CStaticMemoryPool<T>` via `CMemoryPoolManager`. The pool expands as needed, allocating new memory blocks of the initial size, and pushing the indices of free blocks onto the `mFreeIdx` stack.
	- [[MemoryPool Library (Code)#^350665|T* Allocate()]]:
		- The `mFreeIdx` stack, the indices of free blocks are managed, so the most recently freed or front-most block is prioritized for allocation. The pointer target is then constructed in-place using `placement new`, ensuring proper initialization and virtual function support. If no blocks are available, [[MemoryPool Library (Code)#^c2ed89|ExpandPool()]] is called.
	- [[MemoryPool Library (Code)#^cc3746|void Deallocate(T* deallocPtr)]]:
		- Calls the destructor on the pointer target, then frees the memory block by adding it back to the `mFreeIdx` stack for reuse, optimizing memory reuse and performance.
### `CMemoryPoolManager Class`
- **Role**:
	-  A `singleton` class that manages the `creation`, `allocation`, and `deallocation` of all memory pools. This class is designed to create and manage memory pools of various types.
- **Key Methods**:
	- [[MemoryPool Library (Code)#^1ecaa6|bool CreatePool\<T\>(int initCapacity)]]:
		- Creates a memory pool for `T type`. The initial capacity can be set via `initCapacity`. If a pool for this type already exists, it will not be created.
	- [[MemoryPool Library (Code)#^34624f|bool DeletePool\<T\>()]]:
		- Deletes the memory pool of `T type` and frees the associated resources.
	- [[MemoryPool Library (Code)#^8be1c4|T* Allocate\<T\>()]]:
		- Returns an available `T type` pointer from the memory pool of the same type. If no free block exists, the pool will expand.
	- [[MemoryPool Library (Code)#^820f8e|void Deallocate\<T\>(T* deallocPtr)]]:
		- The function deallocates the memory and returns the pointer to the pool for reuse. If the pool becomes unused after deallocation, it is deleted.
	- [[MemoryPool Library (Code)#^afa457|void DeallocateButKeepPool\<T\>(T* deallocPtr)]]:
		- The function deallocates the memory and returns the pointer to the pool for reuse. Even if the pool becomes unused after deallocation, it remains.

---
## **Code Breakdown**
### `Usage Example #1 - (Manual memory management)`

![[MemoryPool Library (Code)#^69ef85]]
Example demonstrating **manual memory management** using the CMemoryPoolManager for the Player-type memory pool.

---
### `Usage Example #2 - (Automatic memory management)` 

![[MemoryPool Library (Code)#^d2dde6]]
Example demonstrating **automatic memory management** using [[Portfolios_Eng/03_Technical Notes/SharedPtr Library|CSharedPtr]] with reference counting for the Player-type memory pool.