---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] Scene"
date: 2025-03-21
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - Component
---



{% capture notice-1 %}
* Scene을 관리하는 클래스
* 싱글톤
{% endcapture %}

{% capture notice-2 %}
* 게임에서 장면을 표현하는 클래스
* 역할에 따라 Scene을 분할하고, 보통 각 Scene별로 사용하는 리소스가 많이 다를 경우 분할함
{% endcapture %}

![Manager](https://github.com/user-attachments/assets/b9f5e0a2-020e-460f-aca6-e7bc54b22115)


#### Engine 프로젝트에 Manager 필터 추가

#### EnginePch 헤더에 싱글톤 생성을 위한 DECLARE_SINGLE, 이를 사용하기 위한 GET_SINGLE 매크로 추가

#### EnginePch.h
```cpp
#define DECLARE_SINGLE(type)		\
private:							\
	type() {}						\
	~type() {}						\
public:								\
	static type* GetInstance()		\
	{								\
		static type instance;		\
		return &instance;			\
	}								\

#define GET_SINGLE(type)	type::GetInstance()
```

#### Manager 필터에 SceneManager, Scene 클래스 추가

#### SceneManager.h
```cpp
class Scene;

class SceneManager
{
	DECLARE_SINGLE(SceneManager);

public:
	void Update();
	void LoadScene(wstring sceneName);

public:
	shared_ptr<Scene> GetActiveScene() { return _activeScene; }

private:
	shared_ptr<Scene> _activeScene;
};
```

#### SceneManager.cpp
```cpp
#include "pch.h"
#include "SceneManager.h"
#include "Scene.h"
#include "Engine.h"
#include "GameObject.h"
#include "MeshRenderer.h"
#include "Material.h"

void SceneManager::Update()
{
	if (_activeScene == nullptr)
		return;

	_activeScene->Update();
	_activeScene->LateUpdate();
}

void SceneManager::LoadScene(wstring sceneName)
{
	_activeScene = LoadTestScene();

	_activeScene->Awake();
	_activeScene->Start();
}
```

<div class="notice">
  {{ notice-1 | markdownify }}
</div>

#### Scene.h
```cpp
class GameObject;

class Scene
{
public:
	void Awake();
	void Start();
	void Update();
	void LateUpdate();

	void AddGameObject(shared_ptr<GameObject> gameObject);
	void RemoveGameObject(shared_ptr<GameObject> gameObject);

private:
	vector<shared_ptr<GameObject>> _gameObjects;
};
```

#### Scene.cpp
```cpp
#include "pch.h"
#include "Scene.h"
#include "GameObject.h"

void Scene::Awake()
{
	for (const shared_ptr<GameObject>& gameObject : _gameObjects)
	{
		gameObject->Awake();
	}
}

void Scene::Start()
{
	for (const shared_ptr<GameObject>& gameObject : _gameObjects)
	{
		gameObject->Start();
	}
}

void Scene::Update()
{
	for (const shared_ptr<GameObject>& gameObject : _gameObjects)
	{
		gameObject->Update();
	}
}

void Scene::LateUpdate()
{
	for (const shared_ptr<GameObject>& gameObject : _gameObjects)
	{
		gameObject->LateUpdate();
	}
}

void Scene::AddGameObject(shared_ptr<GameObject> gameObject)
{
	_gameObjects.push_back(gameObject);
}

void Scene::RemoveGameObject(shared_ptr<GameObject> gameObject)
{
	auto findIt = std::find(_gameObjects.begin(), _gameObjects.end(), gameObject);
	if (findIt != _gameObjects.end())
	{
		_gameObjects.erase(findIt);
	}
}
```

<div class="notice">
  {{ notice-2 | markdownify }}
</div>

#### SceneManager 클래스에 LoadTestScene 함수 추가 후 Game Init 함수에 있던 GameObject 내용 옮기기 

#### SceneManager.h
```cpp
private:
	shared_ptr<Scene> LoadTestScene();
```

#### SceneManager.cpp
```cpp
shared_ptr<Scene> SceneManager::LoadTestScene()
{
	shared_ptr<Scene> scene = make_shared<Scene>();

	// TestObject
	shared_ptr<GameObject> gameObject = make_shared<GameObject>();

	vector<Vertex> vec(4);
	vec[0].pos = Vec3(-0.5f, 0.5f, 0.5f);
	vec[0].color = Vec4(1.f, 0.f, 0.f, 1.f);
	vec[0].uv = Vec2(0.f, 0.f);
	vec[1].pos = Vec3(0.5f, 0.5f, 0.5f);
	vec[1].color = Vec4(0.f, 1.f, 0.f, 1.f);
	vec[1].uv = Vec2(1.f, 0.f);
	vec[2].pos = Vec3(0.5f, -0.5f, 0.5f);
	vec[2].color = Vec4(0.f, 0.f, 1.f, 1.f);
	vec[2].uv = Vec2(1.f, 1.f);
	vec[3].pos = Vec3(-0.5f, -0.5f, 0.5f);
	vec[3].color = Vec4(0.f, 1.f, 0.f, 1.f);
	vec[3].uv = Vec2(0.f, 1.f);

	vector<uint32> indexVec;
	{
		indexVec.push_back(0);
		indexVec.push_back(1);
		indexVec.push_back(2);
	}
	{
		indexVec.push_back(0);
		indexVec.push_back(2);
		indexVec.push_back(3);
	}

	gameObject->Init();

	shared_ptr<MeshRenderer> meshRenderer = make_shared<MeshRenderer>();

	{
		shared_ptr<Mesh> mesh = make_shared<Mesh>();
		mesh->Init(vec, indexVec);
		meshRenderer->SetMesh(mesh);
	}

	{
		shared_ptr<Shader> shader = make_shared<Shader>();
		shared_ptr<Texture> texture = make_shared<Texture>();
		shader->Init(L"..\\Resources\\Shader\\default.hlsli");
		texture->Init(L"..\\Resources\\Texture\\veigar.jpg");
		shared_ptr<Material> material = make_shared<Material>();
		material->SetShader(shader);
		material->SetFloat(0, 0.3f);
		material->SetFloat(1, 0.4f);
		material->SetFloat(2, 0.3f);
		material->SetTexture(0, texture);
		meshRenderer->SetMaterial(material);
	}

	gameObject->AddComponent(meshRenderer);

	scene->AddGameObject(gameObject);

	return scene;
}
```

#### Input, Timer 클래스 Manager 필터로 이동 후, 싱글톤으로 변경

#### Input.h
```cpp
class Input
{
	DECLARE_SINGLE(Input);

public:
	void Init(HWND hwnd);
	void Update();

	// 누르고 있을 때
	bool GetButton(KEY_TYPE key) { return GetState(key) == KEY_STATE::PRESS; }
	// 맨 처음 눌렀을 때
	bool GetButtonDown(KEY_TYPE key) { return GetState(key) == KEY_STATE::DOWN; }
	// 맨 처음 눌렀다 뗐을 때
	bool GetButtonUp(KEY_TYPE key) { return GetState(key) == KEY_STATE::UP; }

private:
	inline KEY_STATE GetState(KEY_TYPE key) { return _states[static_cast<uint8>(key)]; }

private:
	HWND _hwnd;
	vector<KEY_STATE> _states;
};
```

#### Timer.h
```cpp
class Timer
{
	DECLARE_SINGLE(Timer);

public:
	void Init();
	void Update();

	uint32 GetFps() { return _fps; }
	float GetDeltaTime() { return _deltaTime; }

private:
	uint64	_frequency = 0;
	uint64	_prevCount = 0;
	float	_deltaTime = 0.f;

private:
	uint32	_frameCount = 0;
	float	_frameTime = 0.f;
	uint32	_fps = 0;
};
```

#### EnginePch 헤더에 있는 INPUT, DELTA_TIME 매크로 싱글톤에 맞게 수정

#### EnginePch.h
```cpp
#define INPUT				GET_SINGLE(Input)
#define DELTA_TIME			GET_SINGLE(Timer)->GetDeltaTime()
```

#### Engine 클래스에서 Input, Timer에 관한 내용 삭제 후 Init, ShowFps 함수에 싱글톤에 맞게 코드 추가, Render 함수에 SceneManager 싱글톤 업데이트 코드 추가, Update 함수에 Render 함수 호출 코드 추가, LateUpdate 함수 삭제

#### Engine.h
```cpp
#pragma once
#include "Device.h"
#include "CommandQueue.h"
#include "SwapChain.h"
#include "RootSignature.h"
#include "Mesh.h"
#include "Shader.h"
#include "ConstantBuffer.h"
#include "TableDescriptorHeap.h"
#include "Texture.h"
#include "DepthStencilBuffer.h"

class Engine
{
public:
	void Init(const WindowInfo& window);
	void Update();

public:
	shared_ptr<Device> GetDevice() { return _device; }
	shared_ptr<CommandQueue> GetCmdQueue() { return _cmdQueue; }
	shared_ptr<SwapChain> GetSwapChain() { return _swapChain; }
	shared_ptr<RootSignature> GetRootSignature() { return _rootSignature; }
	shared_ptr<TableDescriptorHeap> GetTableDescHeap() { return _tableDescHeap; }
	shared_ptr<DepthStencilBuffer> GetDepthStencilBuffer() { return _depthStencilBuffer; }

	shared_ptr<ConstantBuffer> GetConstantBuffer(CONSTANT_BUFFER_TYPE type) { return _constantBuffers[static_cast<uint8>(type)]; }

public:
	void Render();
	void RenderBegin();
	void RenderEnd();

	void ResizeWindow(int32 width, int32 height);

private:
	void ShowFps();
	void CreateConstantBuffer(CBV_REGISTER reg, uint32 bufferSize, uint32 count);

private:
	// 그려질 화면 관련
	WindowInfo _window;
	D3D12_VIEWPORT _viewport{};
	D3D12_RECT _scissorRect{};

	shared_ptr<Device> _device = make_shared<Device>();
	shared_ptr<CommandQueue> _cmdQueue = make_shared<CommandQueue>();
	shared_ptr<SwapChain> _swapChain = make_shared<SwapChain>();
	shared_ptr<RootSignature> _rootSignature = make_shared<RootSignature>();
	shared_ptr<TableDescriptorHeap> _tableDescHeap = make_shared<TableDescriptorHeap>();
	shared_ptr<DepthStencilBuffer> _depthStencilBuffer = make_shared<DepthStencilBuffer>();

	vector<shared_ptr<ConstantBuffer>> _constantBuffers;
};
```

#### Engine.cpp
```cpp
#include "Input.h"
#include "Timer.h"
#include "SceneManager.h"

void Engine::Init(const WindowInfo& window)
{
	_window = window;

	// 그려질 화면 크기 지정
	_viewport = { 0, 0, static_cast<FLOAT>(window.width), static_cast<FLOAT>(window.height), 0.0f, 1.0f };
	_scissorRect = CD3DX12_RECT(0, 0, window.width, window.height);

	_device->Init();
	_cmdQueue->Init(_device->GetDevice(), _swapChain);
	_swapChain->Init(window, _device->GetDevice(), _device->GetDXGI(), _cmdQueue->GetCmdQueue());
	_rootSignature->Init();
	_tableDescHeap->Init(256);
	_depthStencilBuffer->Init(window);

	CreateConstantBuffer(CBV_REGISTER::b0, sizeof(TransformMatrix), 256);
	CreateConstantBuffer(CBV_REGISTER::b1, sizeof(MaterialParams), 256);

	ResizeWindow(window.width, window.height);

	GET_SINGLE(Input)->Init(window.hwnd);
	GET_SINGLE(Timer)->Init();
}

void Engine::Update()
{
	GET_SINGLE(Input)->Update();
	GET_SINGLE(Timer)->Update();

	Render();

	ShowFps();
}

void Engine::Render()
{
	RenderBegin();

	GET_SINGLE(SceneManager)->Update();

	RenderEnd();
}

void Engine::ShowFps()
{
	uint32 fps = GET_SINGLE(Timer)->GetFps();

	WCHAR text[100] = L"";
	::wsprintf(text, L"FPS : %d", fps);

	::SetWindowText(_window.hwnd, text);
}

```

#### Game Init, Update 함수 바뀐 구조에 따라 코드 수정

#### Game.cpp
```cpp
#include "pch.h"
#include "Game.h"
#include "Engine.h"
#include "SceneManager.h"

void Game::Init(const WindowInfo& info)
{
	GEngine->Init(info);

	GET_SINGLE(SceneManager)->LoadScene(L"TestScene");
}

void Game::Update()
{
	GEngine->Update();
}
```

출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
