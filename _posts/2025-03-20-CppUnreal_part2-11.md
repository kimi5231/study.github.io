---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] Component"
date: 2025-03-20
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - Component
---



{% capture notice-1 %}
* Component 패턴(유니티): 모든 것을 부품으로 관리하고 부품을 조립해서 하나의 완성된 결과물을 만드는 형태
* 상속 기반(언리얼)
{% endcapture %}

{% capture notice-2 %}
* Component를 소유하고 관리하는 클래스
{% endcapture %}

{% capture notice-3 %}
* Component의 가장 상위 클래스
{% endcapture %}

{% capture notice-4 %}
* Material을 업데이트하고 Mesh를 출력하는 클래스
{% endcapture %}

{% capture notice-5 %}
* Transform 역할을 하는 Component 클래스
{% endcapture %}

{% capture notice-6 %}
* 스크립트를 따로 빼서 관리하기 위한 클래스
{% endcapture %}

<div class="notice">
  {{ notice-1 | markdownify }}
</div>

![GameObject](https://github.com/user-attachments/assets/369733a1-9af5-490c-a3a2-c414e38e4363)


#### Engine 프로젝트에 GameObject 필터 추가

#### GameObject 필터에 GameObject, Component, MeshRenderer, Transform, MonoBehaviour 클래스 추가

#### GameObject.h
```cpp
#include "Component.h"

class Transform;
class MeshRenderer;
class MonoBehaviour;

class GameObject : public enable_shared_from_this<GameObject>
{
public:
	GameObject();
	virtual ~GameObject();

	void Init();

	void Awake();
	void Start();
	void Update();
	void LateUpdate();

	shared_ptr<Transform> GetTransform();

	void AddComponent(shared_ptr<Component> component);

private:
	array<shared_ptr<Component>, FIXED_COMPONENT_COUNT> _components;
	vector<shared_ptr<MonoBehaviour>> _scripts;
};
```

#### GameObject.cpp
```cpp
#include "pch.h"
#include "GameObject.h"
#include "Transform.h"
#include "MeshRenderer.h"
#include "MonoBehaviour.h"

GameObject::GameObject()
{

}

GameObject::~GameObject()
{

}

void GameObject::Init()
{
	AddComponent(make_shared<Transform>());
}

void GameObject::Awake()
{
	for (shared_ptr<Component>& component : _components)
	{
		if (component)
			component->Awake();
	}

	for (shared_ptr<MonoBehaviour>& script : _scripts)
	{
		script->Awake();
	}
}

void GameObject::Start()
{
	for (shared_ptr<Component>& component : _components)
	{
		if (component)
			component->Start();
	}

	for (shared_ptr<MonoBehaviour>& script : _scripts)
	{
		script->Start();
	}
}

void GameObject::Update()
{
	for (shared_ptr<Component>& component : _components)
	{
		if (component)
			component->Update();
	}

	for (shared_ptr<MonoBehaviour>& script : _scripts)
	{
		script->Update();
	}
}

void GameObject::LateUpdate()
{
	for (shared_ptr<Component>& component : _components)
	{
		if (component)
			component->LateUpdate();
	}

	for (shared_ptr<MonoBehaviour>& script : _scripts)
	{
		script->LateUpdate();
	}
}

shared_ptr<Transform> GameObject::GetTransform()
{
	uint8 index = static_cast<uint8>(COMPONENT_TYPE::TRANSFORM);
	return static_pointer_cast<Transform>(_components[index]);
}

void GameObject::AddComponent(shared_ptr<Component> component)
{
	component->SetGameObject(shared_from_this());

	uint8 index = static_cast<uint8>(component->GetType());
	if (index < FIXED_COMPONENT_COUNT)
	{
		_components[index] = component;
	}
	else
	{
		_scripts.push_back(dynamic_pointer_cast<MonoBehaviour>(component));
	}
}
```

<div class="notice">
  {{ notice-2 | markdownify }}
</div>

#### Component.h
```cpp
enum class COMPONENT_TYPE : uint8
{
	TRANSFORM,
	MESH_RENDERER,

	MONO_BEHAVIOUR,
	END,
};

enum
{
	FIXED_COMPONENT_COUNT = static_cast<uint8>(COMPONENT_TYPE::END) - 1
};

class GameObject;
class Transform;

class Component
{
public:
	Component(COMPONENT_TYPE type);
	virtual ~Component();

public:
	virtual void Awake() { }
	virtual void Start() { }
	virtual void Update() { }
	virtual void LateUpdate() { }

public:
	COMPONENT_TYPE GetType() { return _type; }
	bool IsValid() { return _gameObject.expired() == false; }

	shared_ptr<GameObject> GetGameObject();
	shared_ptr<Transform> GetTransform();

private:
	friend class GameObject;
	void SetGameObject(shared_ptr<GameObject> gameObject) { _gameObject = gameObject; }

protected:
	COMPONENT_TYPE _type;
	weak_ptr<GameObject> _gameObject;
};
```

#### Component.cpp
```cpp
#include "pch.h"
#include "Component.h"
#include "GameObject.h"

Component::Component(COMPONENT_TYPE type) : _type(type)
{

}

Component::~Component()
{
}

shared_ptr<GameObject> Component::GetGameObject()
{
	return _gameObject.lock();
}

shared_ptr<Transform> Component::GetTransform()
{
	return _gameObject.lock()->GetTransform();
}
```

<div class="notice">
  {{ notice-3 | markdownify }}
</div>

#### MeshRenderer.h
```cpp
#include "Component.h"

class Mesh;
class Material;

class MeshRenderer : public Component
{
public:
	MeshRenderer();
	virtual ~MeshRenderer();

	void SetMesh(shared_ptr<Mesh> mesh) { _mesh = mesh; }
	void SetMaterial(shared_ptr<Material> material) { _material = material; }

	virtual void Update() override { Render(); }

	void Render();

private:
	shared_ptr<Mesh> _mesh;
	shared_ptr<Material> _material;
};
```

#### MeshRenderer.cpp
```cpp
#include "pch.h"
#include "MeshRenderer.h"
#include "Mesh.h"
#include "Material.h"

MeshRenderer::MeshRenderer() : Component(COMPONENT_TYPE::MESH_RENDERER)
{

}

MeshRenderer::~MeshRenderer()
{

}

void MeshRenderer::Render()
{
	_material->Update();
	_mesh->Render();
}
```

<div class="notice">
  {{ notice-4 | markdownify }}
</div>

#### Transform.h
```cpp
#include "Component.h"

class Transform : public Component
{
public:
	Transform();
	virtual ~Transform();

private:

};
```

#### Transform.cpp
```cpp
#include "pch.h"
#include "Transform.h"

Transform::Transform() : Component(COMPONENT_TYPE::TRANSFORM)
{

}

Transform::~Transform()
{

}
```

<div class="notice">
  {{ notice-5 | markdownify }}
</div>

#### MonoBehaviour.h
```cpp
#include "Component.h"

class MonoBehaviour : public Component
{
public:
	MonoBehaviour();
	virtual ~MonoBehaviour();

public:

};
```

#### MonoBehaviour.cpp
```cpp
#include "pch.h"
#include "MonoBehaviour.h"

MonoBehaviour::MonoBehaviour() : Component(COMPONENT_TYPE::MONO_BEHAVIOUR)
{

}

MonoBehaviour::~MonoBehaviour()
{

}
```

<div class="notice">
  {{ notice-6 | markdownify }}
</div>

#### EnginePch 헤더에 있는 Transform 구조체 이름을 TransformMatrix로 변경 후 Transform 헤더로 위치 변경

#### Transform.h
```cpp
struct TransformMatrix
{
	Vec4 offset;
};
```

#### Mesh 클래스에서 Transform과 Material 관련된 것들을 삭제

#### Mesh.h
```cpp
class Material;

class Mesh
{
public:
	void Init(const vector<Vertex>& vertexBuffer, const vector<uint32>& indexBuffer);
	void Render();

private:
	void CreateVertexBuffer(const vector<Vertex>& buffer);
	void CreateIndexBuffer(const vector<uint32>& buffer);

private:
	ComPtr<ID3D12Resource>		_vertexBuffer;
	D3D12_VERTEX_BUFFER_VIEW	_vertexBufferView;
	uint32 _vertexCount = 0;

	ComPtr<ID3D12Resource>		_indexBuffer;
	D3D12_INDEX_BUFFER_VIEW		_indexBufferView;
	uint32 _indexCount = 0;
};
```

#### Mesh.cpp
```cpp
void Mesh::Render()
{
	CMD_LIST->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST);	// 정점 연결 방식
	CMD_LIST->IASetVertexBuffers(0, 1, &_vertexBufferView); // Slot: (0~15)
	CMD_LIST->IASetIndexBuffer(&_indexBufferView);

	GEngine->GetTableDescHeap()->CommitTable();

	// 그리기 예약
	
	// Vertex로 그리기
	//CMD_LIST->DrawInstanced(_vertexCount, 1, 0, 0);

	// Index로 그리기
	CMD_LIST->DrawIndexedInstanced(_indexCount, 1, 0, 0, 0);
}
```

#### Engine Init 함수에 Transform을 TransformMatrix로 이름 변경, LateUpdate 함수 추가

#### Engine.h
```cpp
public:
	void LateUpdate();
```

#### Engine.cpp
```cpp
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

	_input->Init(window.hwnd);
	_timer->Init();

	CreateConstantBuffer(CBV_REGISTER::b0, sizeof(TransformMatrix), 256);
	CreateConstantBuffer(CBV_REGISTER::b1, sizeof(MaterialParams), 256);

	ResizeWindow(window.width, window.height);
}

void Engine::LateUpdate()
{
}
```

#### Game 클래스 Mesh 전역변수 삭제 후, GameObject 전역변수 추가, Init 함수에 GameObject와 MeshRenderer 추가 후 그에 맞게 구조 변경

#### Game.cpp
```cpp
#include "pch.h"
#include "Game.h"
#include "Engine.h"
#include "Material.h"
#include "GameObject.h"
#include "MeshRenderer.h"

shared_ptr<GameObject> gameObject = make_shared<GameObject>();

void Game::Init(const WindowInfo& info)
{
	GEngine->Init(info);

	// 정점 정보
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

	// 인덱스 정보
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

	GEngine->GetCmdQueue()->WaitSync();
}

void Game::Update()
{
	GEngine->Update();

	GEngine->RenderBegin();

	gameObject->Update();

	GEngine->RenderEnd();
}
```

출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
