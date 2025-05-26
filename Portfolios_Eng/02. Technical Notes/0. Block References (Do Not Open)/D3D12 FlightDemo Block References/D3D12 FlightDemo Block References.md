# D3D12 FlightDemo Block References
---
## GIF
![[D3D12 FlightDemo.gif|500]] ^8ce308

## Words
```
Constant Buffer View (CBV):
	A GPU-accessible view that allows shaders to read constant (uniform) data such as transformation matrices, lighting parameters, or camera position.
```

^1ee3b0

```
Shader Resource View (SRV):
	A descriptor that allows shaders to access various GPU resources like textures or structured buffers during rendering.
```

^c55cc0

```
Unordered Access View (UAV):
	A descriptor that allows shaders to read from and write to a resource (such as a texture or buffer) in arbitrary order, without the restrictions of constant or shader resource views.

```

^2dd457

```
Sampler: 
	A descriptor that defines how a texture is sampled, including filtering methods and address modes (e.g., wrap, clamp).
```

^801570

```
HLSL (High-Level Shader Language):
	A shader programming language developed by Microsoft, used to write GPU programs for the Direct3D rendering pipeline.
```

^e6dd51

```
Vertex Shader (VS):
	A programmable stage in the graphics pipeline that transforms vertex data (position, normals, etc.) from model space to clip space.
```

^c15357

```
Pixel Shader (PS):
	A programmable stage that determines the final color of each pixel rendered to the screen, often using texture data and lighting calculations.
```

^b15a68

```
Constant Buffer:
	Holds dynamic data like matrices or lighting per frame (accessed via CBVs).
```

^f5f00b

```
Command Allocator:
	Manages the memory for recording GPU commands during one frame.
```

^c80301

## Code
```cpp
bool Game::Initialize()
{
    // 1. Device Initialization
    if (!D3DApp::Initialize()) 
	    return false;   
        
	// 2. Graphics Pipeline Configuration
	BuildRootSignature();
	BuildShadersAndInputLayout();
	BuildPSOs();

	// 3. Resource Asset Setup
	LoadTextures();
	BuildMaterials();
	BuildDescriptorHeaps();

	// 4. Geometry and Vertex Buffer Setup
	BuildShapeGeometry();

	// 5. Frame Resources Setup
	BuildFrameResources();
    
    // Other code omitted for clarity
    return true;
}
```

^db2b1a

### Stage 1
```cpp
bool D3DApp::Initialize()
{
	// Create and initialize the main window
	if(!InitMainWindow())
		return false;

	// Initialize Direct3D device and related resources
	if(!InitDirect3D())
		return false;

    // Setup swap chain, viewport, and depth buffer based on window size
    OnResize();

	return true;
}
```

^cdc89e

```cpp
bool D3DApp::InitDirect3D()
{
	// Other initialization omitted

	// Create command queue, allocator, and command list for GPU commands
	CreateCommandObjects();

	// Create the swap chain for presenting rendered frames to the screen
	CreateSwapChain();

	// Create descriptor heaps (memory areas) for screen images (RTV) and depth info (DSV)
	CreateRtvAndDsvDescriptorHeaps();

	return true;
}
```

^771272

### Stage 2
```cpp
void Game::BuildRootSignature()
{
    CD3DX12_DESCRIPTOR_RANGE texTable;
    texTable.Init(D3D12_DESCRIPTOR_RANGE_TYPE_SRV, 1, 0);

    CD3DX12_ROOT_PARAMETER slotRootParameter[4];
    slotRootParameter[0].InitAsDescriptorTable(1, &texTable, D3D12_SHADER_VISIBILITY_PIXEL);
    slotRootParameter[1].InitAsConstantBufferView(0);
    slotRootParameter[2].InitAsConstantBufferView(1);
    slotRootParameter[3].InitAsConstantBufferView(2);

    auto staticSamplers = GetStaticSamplers();

    CD3DX12_ROOT_SIGNATURE_DESC rootSigDesc(
        4, slotRootParameter,
        (UINT)staticSamplers.size(), staticSamplers.data(),
        D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT
    );

    ComPtr<ID3DBlob> serializedRootSig = nullptr;
    ComPtr<ID3DBlob> errorBlob = nullptr;
    ThrowIfFailed(D3D12SerializeRootSignature(&rootSigDesc, D3D_ROOT_SIGNATURE_VERSION_1,
        serializedRootSig.GetAddressOf(), errorBlob.GetAddressOf()));

    if (errorBlob)
	    OutputDebugStringA((char*)errorBlob->GetBufferPointer());

    ThrowIfFailed(md3dDevice->CreateRootSignature(
        0,
        serializedRootSig->GetBufferPointer(),
        serializedRootSig->GetBufferSize(),
        IID_PPV_ARGS(mRootSignature.GetAddressOf())));
}
```

^3dc4c3

^fcd7e1
```cpp
void Game::BuildShadersAndInputLayout()
{
    mShaders["standardVS"] = d3dUtil::CompileShader(L"Shaders\\Default.hlsl", nullptr, "VS", "vs_5_1");
    mShaders["opaquePS"]   = d3dUtil::CompileShader(L"Shaders\\Default.hlsl", nullptr, "PS", "ps_5_1");

    mInputLayout =
    {
        { "POSITION",  0, DXGI_FORMAT_R32G32B32_FLOAT, 0,  0, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
        { "NORMAL",    0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 12, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
        { "TEXCOORD",  0, DXGI_FORMAT_R32G32_FLOAT,    0, 24, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
    };
}
```

^9e39a9

```cpp
void Game::BuildPSOs()
{
    D3D12_GRAPHICS_PIPELINE_STATE_DESC opaquePsoDesc = {};
    opaquePsoDesc.InputLayout = { mInputLayout.data(), (UINT)mInputLayout.size() };
    opaquePsoDesc.pRootSignature = mRootSignature.Get();
    opaquePsoDesc.VS = 
    { 
        reinterpret_cast<BYTE*>(mShaders["standardVS"]->GetBufferPointer()),
        mShaders["standardVS"]->GetBufferSize() 
    };
    opaquePsoDesc.PS =
    { 
        reinterpret_cast<BYTE*>(mShaders["opaquePS"]->GetBufferPointer()),
        mShaders["opaquePS"]->GetBufferSize() 
    };
    opaquePsoDesc.RasterizerState        = CD3DX12_RASTERIZER_DESC(D3D12_DEFAULT);
    opaquePsoDesc.BlendState             = CD3DX12_BLEND_DESC(D3D12_DEFAULT);
    opaquePsoDesc.DepthStencilState      = CD3DX12_DEPTH_STENCIL_DESC(D3D12_DEFAULT);
    opaquePsoDesc.SampleMask             = UINT_MAX;
    opaquePsoDesc.PrimitiveTopologyType  = D3D12_PRIMITIVE_TOPOLOGY_TYPE_TRIANGLE;
    opaquePsoDesc.NumRenderTargets       = 1;
    opaquePsoDesc.RTVFormats[0]          = mBackBufferFormat;
    opaquePsoDesc.SampleDesc.Count       = m4xMsaaState ? 4 : 1;
    opaquePsoDesc.SampleDesc.Quality     = m4xMsaaState ? (m4xMsaaQuality - 1) : 0;
    opaquePsoDesc.DSVFormat              = mDepthStencilFormat;

    ThrowIfFailed(md3dDevice->CreateGraphicsPipelineState(&opaquePsoDesc, IID_PPV_ARGS(&mOpaquePSO)));
}
```

^95ff5b

### Stage 3
```cpp
void Game::LoadTextures()
{
    std::vector<std::pair<std::string, std::wstring>> textureVec = 
    {
        { "EagleTex",     L"Textures/Eagle.dds" },
        { "RaptorTex",    L"Textures/Raptor.dds" },
        { "DesertTex",    L"Textures/Desert.dds" },
        { "JetColorTex",  L"Textures/Jet.dds" },
        { "Jet2ColorTex", L"Textures/Jet2.dds" },
        { "MissileTex",   L"Textures/Missile.dds" },
        { "TitleTex",     L"Textures/TitleScreen.dds" },
        { "MainMenuTex",  L"Textures/MainMenuScreen.dds" },
        { "PauseTex",     L"Textures/PauseScreen.dds" },
        { "CursorTex",    L"Textures/Cursor.dds" }
    };

    for (size_t i = 0; i < textureVec.size(); ++i)
    {
        const std::string& name = textureVec[i].first;
        const std::wstring& filename = textureVec[i].second;

        auto tex = std::make_unique<Texture>();
        tex->Name = name;
        tex->Filename = filename;

        ThrowIfFailed(DirectX::CreateDDSTextureFromFile12(
            md3dDevice.Get(), mCommandList.Get(),
            tex->Filename.c_str(), tex->Resource, tex->UploadHeap));

        mTextures[name] = std::move(tex);
    }
}

```

^d68072

```cpp
void Game::BuildMaterials()
{
    struct MaterialInfo 
    {
        std::string Name;
        int MatCBIndex;
        int DiffuseSrvHeapIndex;
    };

    std::vector<MaterialInfo> materialVec = 
    { { "Eagle",0,0 }, { "Raptor",1,1 }, { "Desert",2,2 }, { "Missile",3,3 }, { "Title",4,4 }, { "MainMenu",5,5 }, { "Pause",6,6 }, { "Cursor",7,7 } };

    for (auto& m : materialVec)
    {
        auto mat = std::make_unique<Material>();
        mat->Name = m.Name;
        mat->MatCBIndex = m.MatCBIndex;
        mat->DiffuseSrvHeapIndex = m.DiffuseSrvHeapIndex;
        mat->DiffuseAlbedo = XMFLOAT4(1.0f, 1.0f, 1.0f, 1.0f);
        mat->FresnelR0 = XMFLOAT3(0.05f, 0.05f, 0.05f);
        mat->Roughness = 0.2f;
        mMaterials[m.Name] = std::move(mat);
    }
}

```

^1e6763

```cpp
void Game::BuildDescriptorHeaps()
{
    D3D12_DESCRIPTOR_HEAP_DESC heapDesc = {};
    heapDesc.NumDescriptors = 10;
    heapDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV;
    heapDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_SHADER_VISIBLE;
    ThrowIfFailed(md3dDevice->CreateDescriptorHeap(&heapDesc, IID_PPV_ARGS(&mSrvDescriptorHeap)));

    CD3DX12_CPU_DESCRIPTOR_HANDLE handle(mSrvDescriptorHeap->GetCPUDescriptorHandleForHeapStart());

    D3D12_SHADER_RESOURCE_VIEW_DESC srvDesc = {};
    srvDesc.Shader4ComponentMapping = D3D12_DEFAULT_SHADER_4_COMPONENT_MAPPING;
    srvDesc.ViewDimension = D3D12_SRV_DIMENSION_TEXTURE2D;
    srvDesc.Texture2D.MostDetailedMip = 0;
    srvDesc.Texture2D.ResourceMinLODClamp = 0.0f;

    for (auto key : { "JetColorTex", "Jet2ColorTex", "DesertTex", "MissileTex",
                      "TitleTex", "MainMenuTex", "PauseTex", "CursorTex" })
    {
        auto tex = mTextures[key]->Resource;
        srvDesc.Format = tex->GetDesc().Format;
        srvDesc.Texture2D.MipLevels = tex->GetDesc().MipLevels;
        md3dDevice->CreateShaderResourceView(tex.Get(), &srvDesc, handle);
        handle.Offset(1, mCbvSrvDescriptorSize);
    }
}

```

^a35546

### Stage4
```cpp
void Game::BuildShapeGeometry()
{
	GeometryGenerator geoGen;
	GeometryGenerator::MeshData box = geoGen.CreateBox(1, 0, 1, 1);
	GeometryGenerator::MeshData Eagle = geoGen.CreateMesh("Models/Eagle.txt");
	GeometryGenerator::MeshData Raptor = geoGen.CreateMesh("Models/Raptor.txt");
	GeometryGenerator::MeshData grid = geoGen.CreateGrid(460.0f, 460.0f, 50, 50);

	UINT BoxVertexOffset = 0;
	UINT EagleVertexOffset = (UINT)box.Vertices.size();
	UINT RaptorVertexOffset = EagleVertexOffset + (UINT)Eagle.Vertices.size();
	UINT GridVertexOffset = RaptorVertexOffset + (UINT)Raptor.Vertices.size();

	UINT BoxIndexOffset = 0;
	UINT EagleIndexOffset = (UINT)box.Indices32.size();
	UINT RaptorIndexOffset = EagleIndexOffset + (UINT)Eagle.Indices32.size();
	UINT GridIndexOffset = RaptorIndexOffset + (UINT)Raptor.Indices32.size();

	// Update submeshes
	SubmeshGeometry boxSubmesh;
	boxSubmesh.IndexCount = (UINT)box.Indices32.size();
	boxSubmesh.StartIndexLocation = BoxIndexOffset;
	boxSubmesh.BaseVertexLocation = BoxVertexOffset;

	SubmeshGeometry EagleSubmesh;
	EagleSubmesh.IndexCount = (UINT)Eagle.Indices32.size();
	EagleSubmesh.StartIndexLocation = EagleIndexOffset;
	EagleSubmesh.BaseVertexLocation = EagleVertexOffset;

	SubmeshGeometry RaptorSubmesh;
	RaptorSubmesh.IndexCount = (UINT)Raptor.Indices32.size();
	RaptorSubmesh.StartIndexLocation = RaptorIndexOffset;
	RaptorSubmesh.BaseVertexLocation = RaptorVertexOffset;

	SubmeshGeometry gridSubmesh;
	gridSubmesh.IndexCount = (UINT)grid.Indices32.size();
	gridSubmesh.StartIndexLocation = GridIndexOffset;
	gridSubmesh.BaseVertexLocation = GridVertexOffset;

	// Total vertex count
	UINT TotalVertexCount = (UINT)(box.Vertices.size() + Eagle.Vertices.size() + Raptor.Vertices.size() + grid.Vertices.size());

	// Update vertices
	std::vector<Vertex> vertices(TotalVertexCount);

	size_t BoxVertexCount = box.Vertices.size();
	size_t EagleVertexCount = Eagle.Vertices.size();
	size_t RaptorVertexCount = Raptor.Vertices.size();
	size_t GridVertexCount = grid.Vertices.size();

	// Copy vertices
	for (size_t i = 0; i < BoxVertexCount; ++i)
	{
		vertices[i].Pos = box.Vertices[i].Position;
		vertices[i].Normal = box.Vertices[i].Normal;
		vertices[i].TexC = box.Vertices[i].TexC;
	}

	for (size_t i = 0; i < EagleVertexCount; ++i)
	{
		vertices[BoxVertexCount + i].Pos = Eagle.Vertices[i].Position;
		vertices[BoxVertexCount + i].Normal = Eagle.Vertices[i].Normal;
		vertices[BoxVertexCount + i].TexC = Eagle.Vertices[i].TexC;
	}

	for (size_t i = 0; i < RaptorVertexCount; ++i)
	{
		vertices[BoxVertexCount + EagleVertexCount + i].Pos = Raptor.Vertices[i].Position;
		vertices[BoxVertexCount + EagleVertexCount + i].Normal = Raptor.Vertices[i].Normal;
		vertices[BoxVertexCount + EagleVertexCount + i].TexC = Raptor.Vertices[i].TexC;
	}

	for (size_t i = 0; i < GridVertexCount; ++i)
	{
		auto& p = grid.Vertices[i].Position;
		vertices[BoxVertexCount + EagleVertexCount + RaptorVertexCount + i].Pos = p;
		vertices[BoxVertexCount + EagleVertexCount + RaptorVertexCount + i].Pos.y = GetHillsHeight(p.x, p.z);
		vertices[BoxVertexCount + EagleVertexCount + RaptorVertexCount + i].Normal = GetHillsNormal(p.x, p.z);
		vertices[BoxVertexCount + EagleVertexCount + RaptorVertexCount + i].TexC = grid.Vertices[i].TexC;
	}

	// Update indices
	std::vector<std::uint16_t> indices;

	indices.insert(indices.end(), std::begin(box.GetIndices16()), std::end(box.GetIndices16()));
	indices.insert(indices.end(), std::begin(Eagle.GetIndices16()), std::end(Eagle.GetIndices16()));
	indices.insert(indices.end(), std::begin(Raptor.GetIndices16()), std::end(Raptor.GetIndices16()));
	indices.insert(indices.end(), std::begin(grid.GetIndices16()), std::end(grid.GetIndices16()));

	const UINT vbByteSize = (UINT)vertices.size() * sizeof(Vertex);
	const UINT ibByteSize = (UINT)indices.size() * sizeof(std::uint16_t);

	auto geo = std::make_unique<MeshGeometry>();
	geo->Name = "ShapeGeo";

	// Vertex and index buffers creation
	ThrowIfFailed(D3DCreateBlob(vbByteSize, &geo->VertexBufferCPU));
	CopyMemory(geo->VertexBufferCPU->GetBufferPointer(), vertices.data(), vbByteSize);

	ThrowIfFailed(D3DCreateBlob(ibByteSize, &geo->IndexBufferCPU));
	CopyMemory(geo->IndexBufferCPU->GetBufferPointer(), indices.data(), ibByteSize);

	geo->VertexBufferGPU = d3dUtil::CreateDefaultBuffer(md3dDevice.Get(), mCommandList.Get(), vertices.data(), vbByteSize, geo->VertexBufferUploader);
	geo->IndexBufferGPU  = d3dUtil::CreateDefaultBuffer(md3dDevice.Get(), mCommandList.Get(), indices.data(), ibByteSize, geo->IndexBufferUploader);

	// Geometry properties
	geo->VertexByteStride = sizeof(Vertex);
	geo->VertexBufferByteSize = vbByteSize;
	geo->IndexFormat = DXGI_FORMAT_R16_UINT;
	geo->IndexBufferByteSize = ibByteSize;

	// Draw arguments
	geo->DrawArgs["box"]    = boxSubmesh;
	geo->DrawArgs["Eagle"]  = EagleSubmesh;
	geo->DrawArgs["Raptor"] = RaptorSubmesh;
	geo->DrawArgs["grid"]   = gridSubmesh;

	mGeometries[geo->Name] = std::move(geo);
}
```

^e40791

### Stage 5
```cpp
void Game::BuildFrameResources()
{
	for (int i = 0; i < gNumFrameResources; ++i)
		mFrameResources.push_back(std::make_unique<FrameResource>(md3dDevice.Get(), 1, 200, (UINT)mMaterials.size()));
}
```

^925a50
