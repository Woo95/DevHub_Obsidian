# **SharedPtr Library**
---
## **Overview**
This library implements a custom **shared pointer** as a way to deeply understand the internal structure and behavior of C++ smart pointers. It mimics the **reference counting** mechanism of `std::shared_ptr`, with added focus on clarity and minimal overhead. The implementation is **header-only** and aims to provide safe memory management by automatically deleting resources when no longer needed.

[**View Repository**](https://github.com/Woo95/SharedPtr)

---
## **Class Descriptions**
### `CRefCounter Class` 
- **Role**: 
	- A base class for objects that need reference counting.
- **Key Methods**:
	- [[SharedPtr Library (Code)#^15063f|void IncrementRef()]]:
		- Increments the reference count.
	- [[SharedPtr Library (Code)#^e68875|void DecrementRef()]]:
		- Decrements the reference count.
		- Deletes the resource when the count reaches 0.
### `CSharedPtr<T> Class`
- **Role**: 
	- A smart pointer that manages the lifetime of a dynamically allocated resource using reference counting.
- **Key Methods**:
	- These functions are the only ones directly involved in reference count management and pointer assignment.
		- [[SharedPtr Library (Code)#^af2f1a|CSharedPtr(T* ptr)]]:
			- *Constructor* that takes a raw pointer and increases its reference count.
		- [[SharedPtr Library (Code)#^7ed3c8|CSharedPtr(const CSharedPtr\<T\>& other)]]:
			- *Shallow Copy Constructor* that shares the pointer and increases the count.
		- [[SharedPtr Library (Code)#^19ba77|~CSharedPtr()]]:
			- *Destructor* that decreases the count and deletes the object if it reaches 0.
		- [[SharedPtr Library (Code)#^88b6a9|void operator = (T* ptr)]]:
			- Assigns a raw pointer, properly updating the reference count.
		- [[SharedPtr Library (Code)#^9a7d66|void operator = (const CSharedPtr\<T\>& other)]]:
			- Assigns another smart pointer, sharing the resource and adjusting the count.

---
## **Code Breakdown**
### `Usage Example`
![[SharedPtr Library (Code)#^787350]]
Example demonstrating the use of `CSharedPtr` with **reference counting** to **manage the memory** of a `CObject`.