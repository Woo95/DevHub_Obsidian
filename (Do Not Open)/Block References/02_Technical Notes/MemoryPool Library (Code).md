# MemoryPool Library Block References
---
## Eng & Kor

### From IMemoryPool Class
```cpp
virtual ~IMemoryPool() = default;
```

^da8d6e

### From CMemoryPoolManager Class
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

---
## Eng

### From CStaticMemoryPool\<T\> Class
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

### Examples
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

---
## Kor

### From CStaticMemoryPool\<T\> Class
```cpp
// 풀에서 사용되지 않은 다음 메모리 블록의 주소를 반환
T* Allocate()
{
	if (mFreeIdx.empty())
	{
		ExpandPool();
	}

	size_t idx = mFreeIdx.top();
	mFreeIdx.pop();

	T* currPool = mMemoryPool[idx / mBlockSize];
	// 이미 할당된 메모리 블록의 지정된 주소에 placement new를 사용해 객체를 생성
	T* obj = new (&currPool[idx % mBlockSize]) T;

	return obj;
}
```

^aade06

```cpp
void Deallocate(T* deallocPtr)
{
	// placement new로 할당된 포인터의 소멸자를 호출하여 객체를 정리하되, 메모리 블록 자체는 그대로 유지
	deallocPtr->~T();

	size_t poolAmount = mMemoryPool.size();
	for (size_t i = 0; i < poolAmount; i++)
	{
		// deallocPtr가 어느 MemoryPool 블록에 포함되는지 확인
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

^467684

```cpp
void ExpandPool()
{
	// 메모리 할당 또는 재할당
	T* newPool = (T*)malloc(sizeof(T) * mBlockSize);
	mMemoryPool.emplace_back(newPool);

	// mFreeIdx 갱신
	size_t freeIdx = (mBlockSize * mMemoryPool.size()) - 1;
	for (int i = 0; i < mBlockSize; i++)
	{
		mFreeIdx.push(freeIdx - i);
	}
}
```

^a512da

### Examples
```cpp
int main()
{
	// Player 타입을 위한 크기 100의 메모리 풀 생성
	CMemoryPoolManager::GetInst()->CreatePool<Player>(100);

	// Player 타입의 메모리 풀에서 사용 가능한 메모리 블록의 포인터를 반환
	Player* ptr = CMemoryPoolManager::GetInst()->Allocate<Player>();

	// 사용한 포인터를 메모리 풀로 반환하여 재사용
	// 해제 후, 풀이 더 이상 사용되지 않으면 삭제
	CMemoryPoolManager::GetInst()->Deallocate(ptr);

	// 또는, 해제 후에 더 이상 사용되지 않더라도 풀이 삭제되지 않도록 할 때 사용
	CMemoryPoolManager::GetInst()->DeallocateButKeepPool(ptr);

	// Player 타입의 메모리 풀 삭제
	CMemoryPoolManager::GetInst()->DeletePool<Player>();

	// 메모리 풀 매니저 객체 삭제
	CMemoryPoolManager::DestroyInst();

	return 0;
}
```

^2ce538

```cpp
// CSharedPtr를 사용하려는 클래스 객체는 반드시 CRefCounter를 상속 필수
int main()
{
	// Player 타입을 위한 크기 100의 메모리 풀 생성
	CMemoryPoolManager::GetInst()->CreatePool<Player>(100);

	// Player 타입의 메모리 풀에서 사용 가능한 메모리 블록의 포인터를 반환
	Player* playerPtr = CMemoryPoolManager::GetInst()->Allocate<Player>();

	CSharedPtr<Player> ptr1 = playerPtr; // 참조 카운트: 1
	CSharedPtr<Player> ptr2 = playerPtr; // 참조 카운트: 2

	ptr1 = nullptr; // refCount: 1
	// 자동으로 메모리 관리 및 메모리 풀에 메모리 반환
	ptr2 = nullptr; // refCount: 0

	// 메모리 풀 매니저 객체 삭제
	CMemoryPoolManager::DestroyInst();

	return 0;
}
```

^03d993
