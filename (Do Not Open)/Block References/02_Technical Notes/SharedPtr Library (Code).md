# SharedPtr Library Block References
---
## Eng & Kor

### From CRefCounter Class
```cpp
void IncrementRef()
{
	mRefCount++;
}
```

^15063f

```cpp
void DecrementRef()
{
	mRefCount--;

	if (mRefCount == 0)
		delete this;
}
```

^e68875

### From CSharedPtr\<T> Class
```cpp
CSharedPtr(T* ptr) :
	mPtr(ptr)
{
	mPtr->IncrementRef();
}
```

^af2f1a

```cpp
CSharedPtr(const CSharedPtr<T>& other) :
	mPtr(other.mPtr)
{
	mPtr->IncrementRef();
}
```

^7ed3c8

```cpp
~CSharedPtr()
{
	if (mPtr)
		mPtr->DecrementRef();
}
```

^19ba77

```cpp
void operator = (T* ptr)
{
	if (mPtr)
		mPtr->DecrementRef();

	mPtr = ptr;

	if (mPtr)
		mPtr->IncrementRef();
}
```

^88b6a9

```cpp
void operator = (const CSharedPtr<T>& other)
{
	if (mPtr)
		mPtr->DecrementRef();

	mPtr = other.mPtr;

	if (mPtr)
		mPtr->IncrementRef();
}
```

^9a7d66

---
## Eng

### Example

```cpp
int main()
{
	// create CObject
	CObject* obj = new CObject;

	CSharedPtr<CObject> sharedPtr1 = obj; // refCount: 1
	CSharedPtr<CObject> sharedPtr2 = obj; // refCount: 2

	sharedPtr1 = nullptr; // refCount: 1
	sharedPtr2 = nullptr; // refCount: 0, delete CObject

	return 0;
}
```

^787350

---
## Kor

### Example

```cpp
int main()
{
	// CObject 생성
	CObject* obj = new CObject;

	CSharedPtr<CObject> sharedPtr1 = obj; // 참조 카운트: 1
	CSharedPtr<CObject> sharedPtr2 = obj; // 참조 카운트: 2

	sharedPtr1 = nullptr; // 참조 카운트: 1
	sharedPtr2 = nullptr; // 참조 카운트: 0, CObject 삭제

	return 0;
}
```

^1d1de5
