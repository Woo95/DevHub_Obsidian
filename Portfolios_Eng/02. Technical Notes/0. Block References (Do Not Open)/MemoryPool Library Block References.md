# MemoryPool Library Block References
---
## From IMemoryPool Class
```cpp
virtual ~IMemoryPool() = default;
```

^da8d6e
## From CStaticMemoryPool\<T> Class
```cpp
// returns the address of the next unused memory block in the pool
T* Allocate()
{
	if (mFreeIdx.empty())
	{
		ExpandPool();
	}

	size_t idx = mFreeIdx.top();
	mFreeIdx.pop();

	T* currPool = mMemoryPool[idx / mBlockSize];
	// uses placement new to construct an object in an already allocated memory block at the given address in the pool
	T* obj = new (&currPool[idx % mBlockSize]) T;

	return obj;
}
```

^350665

```cpp
void Deallocate(T* deallocPtr)
{
	// call the destructor for the ptr allocated with placement new, cleaning up the object while leaving the memory block unchanged
	deallocPtr->~T();

	size_t poolAmount = mMemoryPool.size();
	for (size_t i = 0; i < poolAmount; i++)
	{
		// check which MemoryPool block contains the deallocPtr
		if (IsWithinRange(deallocPtr, &mMemoryPool[i][0], &mMemoryPool[i][mBlockSize - 1]))
		{
			size_t currPoolIdx = deallocPtr - &mMemoryPool[i][0];
			size_t totalPoolIdx = (mBlockSize * i) + currPoolIdx;
			mFreeIdx.push(totalPoolIdx);

			break;
		}
	}
}
```

^cc3746

```cpp
void ExpandPool()
{
	// allocate or reallocate memory
	T* newPool = (T*)malloc(sizeof(T) * mBlockSize);
	mMemoryPool.emplace_back(newPool);

	// update mFreeIdx
	size_t freeIdx = (mBlockSize * mMemoryPool.size()) - 1;
	for (int i = 0; i < mBlockSize; i++)
	{
		mFreeIdx.push(freeIdx - i);
	}
}
```

^c2ed89

## From CMemoryPoolManager Class
```cpp
template <typename T>
bool CreatePool(int initCapacity)
{
	if (HasPool<T>() || initCapacity <= 0)
		return false;

	std::type_index key = typeid(T);
	mPools[key] = new CStaticMemoryPool<T>(initCapacity);

	return true;
}
```

^1ecaa6

```cpp
template <typename T>
bool DeletePool()
{
	if (!HasPool<T>())
		return false;

	std::type_index key = typeid(T);
	SAFE_DELETE(mPools[key]);
	mPools.erase(key);

	return true;
}
```

^34624f

```cpp
template <typename T>
T* Allocate()
{
	if (HasPool<T>())
	{
		CStaticMemoryPool<T>* pool = GetPool<T>();
		return pool->Allocate();
	}
	return nullptr;
}
```

^8be1c4

```cpp
template <typename T>
void Deallocate(T* deallocPtr)
{
	if (HasPool<T>())
	{
		CStaticMemoryPool<T>* pool = GetPool<T>();
		pool->Deallocate(deallocPtr);

		if (pool->IsPoolUnused())
		{
			DeletePool<T>();
		}
	}
}
```

^820f8e

```cpp
template <typename T>
void DeallocateButKeepPool(T* deallocPtr)
{
	if (HasPool<T>())
	{
		CStaticMemoryPool<T>* pool = GetPool<T>();
		pool->Deallocate(deallocPtr);
	}
}
```

^afa457

## Examples
```cpp
int main()
{
	// create a memory pool with a size of 100 for the Player-type
	CMemoryPoolManager::GetInst()->CreatePool<Player>(100);

	// from the Player-type memory pool, return a ptr from an available memory block.
	Player* ptr = CMemoryPoolManager::GetInst()->Allocate<Player>();

	// return the ptr used from the memory block back to the memory pool for reuse; the pool may be `deleted` if it becomes unused after this deallocation
	CMemoryPoolManager::GetInst()->Deallocate(ptr);

	// Alternatively, use this to `prevent` the pool from being deleted even if it becomes unused after deallocation
	CMemoryPoolManager::GetInst()->DeallocateButKeepPool(ptr);

	// delete the Player-type memory pool
	CMemoryPoolManager::GetInst()->DeletePool<Player>();

	// delete the memory pool manager instance
	CMemoryPoolManager::DestroyInst();

	return 0;
}
```

^69ef85

```cpp
// classes that intend to use CSharedPtr must inherit from CRefCounter
int main()
{
	// create a memory pool with a size of 100 for the Player-type
	CMemoryPoolManager::GetInst()->CreatePool<Player>(100);

	// from the Player-type memory pool, return a ptr from an available memory block.
	Player* playerPtr = CMemoryPoolManager::GetInst()->Allocate<Player>();

	CSharedPtr<Player> ptr1 = playerPtr; // refCount: 1
	CSharedPtr<Player> ptr2 = playerPtr; // refCount: 2

	ptr1 = nullptr; // refCount: 1
	// automatically manage memory and return the memory to the pool
	ptr2 = nullptr; // refCount: 0

	// delete the memory pool manager instance
	CMemoryPoolManager::DestroyInst();

	return 0;
}
```

^d2dde6