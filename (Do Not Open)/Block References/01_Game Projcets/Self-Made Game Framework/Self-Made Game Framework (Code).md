# Code Block References
---
## Eng & Kor

### CAssetManager

```cpp
template <typename T>
T* PlacementNew(void*& memoryBlock)
{
    T* manager  = new (memoryBlock) T;
    memoryBlock = (char*)memoryBlock + sizeof(T);
    
    return manager;
}
```

^aa2822

```cpp
CAssetManager::CAssetManager(void* memoryBlock)
{
    mTextureManager   = PlacementNew<CTextureManager>(memoryBlock);
    mSpriteManager    = PlacementNew<CSpriteManager>(memoryBlock);
    mAnimationManager = PlacementNew<CAnimationManager>(memoryBlock);
    mUIManager        = PlacementNew<CUIManager>(memoryBlock);
    mFontManager      = PlacementNew<CFontManager>(memoryBlock);
    mSoundManager     = PlacementNew<CSoundManager>(memoryBlock);
}
```

^413967

```cpp
CAssetManager* CAssetManager::GetInst()
{
    if (!mInst)
    {
        const size_t totalSize = sizeof(CAssetManager)
            + sizeof(CTextureManager)   + sizeof(CSpriteManager)
            + sizeof(CAnimationManager) + sizeof(CUIManager)
            + sizeof(CFontManager)      + sizeof(CSoundManager);

        void* memoryBlock = malloc(totalSize);
        mInst = new (memoryBlock) CAssetManager((char*)memoryBlock + sizeof(CAssetManager));
    }
    return mInst;
}
```

^a4b1a3

---

### CSceneManager
```cpp
void CSceneManager::ChangeRequest(ETransition transition, ESceneState state)
{
	mPending.transition   = transition;
	mPending.pendingState = state;
}
```

^933d3b

```cpp
void CSceneManager::Update(float deltaTime)
{
	if (mPending.transition != ETransition::NONE)
		ChangeApply();

	mScenes.back()->Update(deltaTime);
}
```

^017baa

#### Kor

```cpp
void CSceneManager::ChangeApply()
{
	switch (mPending.transition)
	{
	case ETransition::PUSH:
		PushScene();
		break;
	case ETransition::POP:
		PopScene();
		break;
	case ETransition::SWAP:
		SwapScene();
		break;
	case ETransition::CLEAR:
		ClearScenes();
		break;
	case ETransition::CLEAR_THEN_PUSH:
		ClearThenPushScene();
		break;
	default:
		break;
	}

	// 새로운 씬 플래그 초기화
	mPending.transition   = ETransition::NONE;
	mPending.pendingState = ESceneState::NONE;
}
```

^daf2a4

```cpp
void CSceneManager::PushScene()
{
	// 새로운 씬 생성 및 리소스들 로드
	CScene* newScene = GetSceneFromState(mPending.pendingState);
	newScene->LoadResources();

	// 새로운 씬 추가
	mScenes.push_back(newScene);
	mScenes.back()->Enter();
}
```

^19f52b

```cpp
void CSceneManager::PopScene()
{
	assert(!mScenes.empty());

	// 이전 씬 제거 및 리소스들 언로드
	CScene* oldScene = mScenes.back();
	if (oldScene->Exit())
	{
		oldScene->UnloadResources();
		SAFE_DELETE(oldScene);
		mScenes.pop_back();
	}
}
```

^d37f71

```cpp
void CSceneManager::SwapScene()
{
	// 새로운 씬 생성 및 리소스들 로드
	CScene* newScene = GetSceneFromState(mPending.pendingState);
	newScene->LoadResources();

	// 이전 씬 정리
	PopScene();

	// 새로운 씬 추가
	mScenes.push_back(newScene);
	mScenes.back()->Enter();
}
```

^459179

```cpp
void CSceneManager::ClearScenes()
{
	// 이전 모든 씬 정리
	while (!mScenes.empty())
		PopScene();
}
```

^c06fde

```cpp
void CSceneManager::ClearThenPushScene()
{
	// 새로운 씬 생성 및 리소스들 로드
	CScene* newScene = GetSceneFromState(mPending.pendingState);
	newScene->LoadResources();

	// 이전 모든 씬 정리
	ClearScenes();

	// 새로운 씬 추가
	mScenes.push_back(newScene);
	mScenes.back()->Enter();
}
```

^e38631

```cpp
void CSceneManager::SwapScene()
{
	// 새로운 씬 생성 및 리소스들 로드
	CScene* newScene = GetSceneFromState(mPending.pendingState);
	newScene->LoadResources();
	
	// 이전 씬 제거 및 리소스들 언로드
	CScene* oldScene = mScenes.back();
	if (oldScene->Exit())
	{
		oldScene->UnloadResources();
		SAFE_DELETE(oldScene);
		mScenes.pop_back();
	}

	// 새로운 씬 추가
	mScenes.push_back(newScene);
	mScenes.back()->Enter();
}
```

^71791d

#### Eng & Kor
```cpp
class CScene abstract
{
	friend class CSceneManager;
	
protected:
	CScene();
	virtual ~CScene();

protected:
	std::vector<std::shared_ptr<class CTexture>> mTextures;
    std::vector<std::shared_ptr<class CFont>> mFonts;
    std::vector<std::shared_ptr<class CSFX>>  mSFXs;
    std::vector<std::shared_ptr<class CBGM>>  mBGMs;
    
protected:
	virtual void LoadResources() = 0;
	
    void UnloadResources()
    {
	    mTextures.clear();
	    mFonts.clear();
	    mSFXs.clear();
	    mBGMs.clear();
    }
}
```

^185ec2

```cpp
class CTextureManager
{
private:
	std::unordered_map<std::string, std::weak_ptr<CTexture>> mTextures;
	
public:
	std::shared_ptr<CTexture> LoadTexture(...);
}
//---------------------------------------------------------------//
class CFontManager
{
private:
	std::unordered_map<std::string, std::weak_ptr<CFont>> mFonts;
	
public:
	std::shared_ptr<CFont> LoadFont(...);
}
//---------------------------------------------------------------//
class CSoundManager
{
private:
	std::unordered_map<std::string, std::weak_ptr<CSFX>> mSFXs;
	std::unordered_map<std::string, std::weak_ptr<CBGM>> mBGMs;
	
public:
	std::shared_ptr<CSFX> LoadSFX(...);
	std::shared_ptr<CBGM> LoadBGM(...);
}
```

^a53e5d

---
### CScene
```cpp
class CScene abstract
{
	friend class CSceneManager;

protected:
	CScene();
	virtual ~CScene();

protected:
    std::vector<CLayer*> mLayers;

protected:
	virtual bool Enter() = 0;
	virtual bool Exit()  = 0;
	    
    virtual void Update(float deltaTime);
    virtual void LateUpdate(float deltaTime);
    virtual void Render(SDL_Renderer* renderer);
	
public:
    template <typename T, int initialCapacity = 50>
    T* InstantiateObject(const std::string& name, ELayer::Type type);
};
```

^36e7b4

```cpp
CScene::CScene()
{
    mLayers.resize(ELayer::Type::MAX);
    for (int i = 0; i < ELayer::Type::MAX; i++)
    {
        mLayers[i] = CMemoryPoolManager::GetInst()->Allocate<CLayer>();
    }
}
```

^31e555

```cpp
CScene::~CScene()
{
    for (CLayer* layer : mLayers)
    {
        CMemoryPoolManager::GetInst()->DeallocateButKeepPool<CLayer>(layer);
    }
}
```

^1b2665

```cpp
void CScene::Update(float deltaTime)
{
    for (CLayer* layer : mLayers)
        layer->Update(deltaTime);

    if (mSceneCollision)
        mSceneCollision->Update(deltaTime);

    if (mCamera)
        mCamera->Update(deltaTime);

    if (mSceneUI)
        mSceneUI->Update(deltaTime);
}
```

^0c3634

```cpp
void CScene::LateUpdate(float deltaTime)
{
    for (CLayer* layer : mLayers)
        layer->LateUpdate(deltaTime);

    if (mSceneCollision)
	    mSceneCollision->LateUpdate(deltaTime);
	    
    if (mSceneUI)
        mSceneUI->LateUpdate(deltaTime);
}
```

^c1f92b

```cpp
void CScene::Render(SDL_Renderer* renderer)
{
    for (CLayer* layer : mLayers)
        layer->Render(renderer);

    if (mSceneUI)
        mSceneUI->Render(renderer);
}
```

^8dbf95

```cpp
template <typename T, int initialCapacity = 50>
T* InstantiateObject(const std::string& name, ELayer::Type type)
{
	if (!CMemoryPoolManager::GetInst()->HasPool<T>())
		CMemoryPoolManager::GetInst()->CreatePool<T>(initialCapacity);

	T* obj = CMemoryPoolManager::GetInst()->Allocate<T>();
	obj->SetName(name);
	obj->mScene = this;
	obj->mLayer = mLayers[type];

	if (!obj->Init())
	{
		CMemoryPoolManager::GetInst()->Deallocate<T>(obj);
		return nullptr;
	}
	
	mLayers[type]->AddObject(obj);
	return obj;
}
```

^144420

---

### CLayer
```cpp
class CLayer
{
	friend class CScene;

public:
	CLayer();
	~CLayer();

private:
    std::vector<class CObject*> mObjects;
	ESort::Type mSort = ESort::Y;

protected:
	void Update(float deltaTime);
	void LateUpdate(float deltaTime);
	void Render(SDL_Renderer* renderer);

public:
	void AddObject(CObject* obj) { mObjects.emplace_back(obj); }

private:
	static bool SortY(CObject* objA, CObject* objB);
};
```

^70bdb1

```cpp
CLayer::~CLayer()
{
    for (CObject* obj : mObjects)
    {
        obj->Release();
    }
}
```

^65f9d6

```cpp
void CLayer::Update(float deltaTime)
{
    for (CObject* obj : mObjects)
    {
        if (!obj->GetActive())
        {
            obj->Destroy();

            continue;
        }
        else if (!obj->GetEnable())
        {
            continue;
        }
        obj->Update(deltaTime);
    }
}
```

^d0565b

```cpp
void CLayer::LateUpdate(float deltaTime)
{
    for (size_t i = mObjects.size(); i > 0; i--)
    {
        CObject* obj = mObjects[i - 1];

        if (!obj->GetActive())
        {
            std::swap(mObjects[i - 1], mObjects.back());
            mObjects.pop_back();
            
            obj->Release();
            
            continue;
        }
        else if (!obj->GetEnable())
        {
            continue;
        }
        obj->LateUpdate(deltaTime);
    }
}
```

^e4106e

```cpp
void CLayer::Render(SDL_Renderer* renderer)
{
    if (mSort == ESort::Type::Y)
	    std::sort(mObjects.begin(), mObjects.end(), SortY);

    for (CObject* obj : mObjects)
    {
        if (!obj->GetActive() || !obj->GetEnable())
        {
            continue;
        }
        obj->Render(renderer);
    }
}
```

^549045

---

### CObject
```cpp
class CObject abstract : public CEntityBase
{
	friend class CScene;
	friend class CLayer;

protected:
	CObject();
	virtual ~CObject();
	
protected:
	CScene* mScene;
	CLayer* mLayer;
	
	CComponent* mRootComponent;
	
protected:
	virtual bool Init();
	virtual void Update(float deltaTime);
	virtual void LateUpdate(float deltaTime);
	virtual void Render(SDL_Renderer* renderer);
	
	virtual void Release() = 0;
	
public:
	CTransform* GetTransform() const { return mRootComponent->GetTransform(); }
	CComponent* GetComponent(const std::string& name = "")
	{
		if (name.empty())
			return mRootComponent;
			
		size_t hashID = std::hash<std::string>()(name);
		return mRootComponent->FindComponent(hashID);
	}
	template <typename T>
	T* GetComponent() const { return mRootComponent->FindComponent<T>(); }
	
	template <typename T, int initialCapacity = 10>
	T* AllocateComponent(const std::string& name);
};
```

^24ed16

```cpp
CObject::CObject() :
	mScene(nullptr),
	mLayer(nullptr)
{
	mRootComponent = new CComponent;
	
	mRootComponent->SetName("RootComponent");
	mRootComponent->mObject = this;
}
```

^b8195f

```cpp
CObject::~CObject()
{
	SAFE_DELETE(mRootComponent);
}
```

^b2dfbe

```cpp
bool CObject::Init()
{
	return mRootComponent->Init();
}
```

^6c4ee4

```cpp

void CObject::Update(float deltaTime)
{
	mRootComponent->Update(deltaTime);
}
```

^321393

```cpp
void CObject::LateUpdate(float deltaTime)
{
	mRootComponent->LateUpdate(deltaTime);
}
```

^2a3d8a

```cpp
void CObject::Render(SDL_Renderer* renderer)
{
	mRootComponent->Render(renderer);
}
```

^fba18c

```cpp
template <typename T, int initialCapacity = 10>
T* AllocateComponent(const std::string& name)
{
	if (!CMemoryPoolManager::GetInst()->HasPool<T>())
		CMemoryPoolManager::GetInst()->CreatePool<T>(initialCapacity);

	T* component = CMemoryPoolManager::GetInst()->Allocate<T>();
	component->SetName(name);

	return component;
}
```

^ab3424

---

### CComponent
```cpp
class CComponent : public CEntityBase
{
	friend class CObject;
	
protected:
	CComponent();
	virtual ~CComponent();
	
protected:
	size_t mTypeID = -1;
	
	CObject*    mObject    = nullptr;
	CTransform* mTransform = nullptr;
	
	CComponent* mParent = nullptr;
	std::vector<CComponent*> mChilds;
	
protected:
	virtual bool Init();
	virtual void Update(float deltaTime);
	virtual void LateUpdate(float deltaTime);
	virtual void Render(SDL_Renderer* renderer);
	
	virtual void Release();
	
public:
	CObject* GetObject() const { return mObject; }
	CTransform* GetTransform() const { return mTransform; }
	
	void AddChild(CComponent* child);
	bool DeleteChild(CComponent* child);
	
private:
	CComponent* FindRootComponent();
	CComponent* FindComponent(size_t id);
	
	template <typename T>
	T* FindComponent();
};
```

^47760c

```cpp
CComponent::CComponent()
{
	mTransform = CMemoryPoolManager::GetInst()->Allocate<CTransform>();
}
```

^750ae3

```cpp
CComponent::~CComponent()
{
	for (CComponent* child : mChilds)
	{
		child->Release();
	}
	CMemoryPoolManager::GetInst()->DeallocateButKeepPool<CTransform>(mTransform);
}
```

^0e00e0

```cpp
bool CComponent::Init()
{
	for (CComponent* child : mChilds)
	{
		if (!child->Init())
			return false;
	}
	return true;
}
```

^3286fc

```cpp
void CComponent::Update(float deltaTime)
{
	for (CComponent* child : mChilds)
	{
		if (!child->GetActive())
		{
			child->Destroy();
			continue;
		}
		else if (!child->GetEnable())
		{
			continue;
		}
		child->Update(deltaTime);
	}
}
```

^34b1f6

```cpp
void CComponent::LateUpdate(float deltaTime)
{
	for (size_t i = mChilds.size(); i > 0; i--)
	{
		CComponent* child = mChilds[i - 1];
		
		if (!child->GetActive())
		{
			std::swap(mChilds[i - 1], mChilds.back());
			mChilds.pop_back();
			
			std::vector<CTransform*> transChilds = mTransform->GetChilds();
			
			std::swap(mTransform->GetChilds()[i - 1], mTransform->GetChilds().back());
			mTransform->GetChilds().pop_back();
			
			child->Release();
			
			continue;
		}
		else if (!child->GetEnable())
		{
			continue;
		}
		child->LateUpdate(deltaTime);
	}
}
```

^d23f4e

```cpp
void CComponent::Render(SDL_Renderer* renderer)
{
	for (CComponent* child : mChilds)
	{
		if (!child->GetActive() || !child->GetEnable())
			continue;
			
		child->Render(renderer);
	}
}
```

^7c581f
