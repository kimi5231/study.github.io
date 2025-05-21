---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] 직교 투영"
date: 2025-05-21
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - Rendering
---



{% capture notice-1 %}
* 원근 투영: 원근감을 표현하는 투영 방식, 주로 객체를 그릴 때 사용
* 직교 투영: 원근감을 표현하지 않는 투영 방식, 주로 UI를 그릴 때 사용
{% endcapture %}



#### 직교 투영 행렬



<div class="notice">
  {{ notice-1 | markdownify }}
</div>

#### 쉐이더 파일들의 확장자를 전부 fx로 수정 후 default 쉐이더 이름을 forward로 변경



#### 속성 페이지에서 셰이더 모델 5.0으로 지정 후 셰이더 형식을 효과로 변경



#### SceneManager 헤더에 Layer 수를 나타내는 enum 추가

#### SceneManager.h
```cpp
enum
{
	MAX_LAYER = 32
};
```

#### SceneManager 클래스에 Layer 관리를 위한 멤버 변수, 멤버 함수 추가

#### SceneManager.h
```cpp
class SceneManager
{
public:
  void SetLayerName(uint8 index, const wstring& name);
  const wstring& IndexToLayerName(uint8 index) { return _layerNames[index]; }
  uint8 LayerNameToIndex(const wstring& name);

private:
  array<wstring, MAX_LAYER> _layerNames;
  map<wstring, uint8> _layerIndex;
}
```

#### SceneManager.cpp
```cpp
void SceneManager::SetLayerName(uint8 index, const wstring& name)
{
	// 기존 데이터 삭제
	const wstring& prevName = _layerNames[index];
	_layerIndex.erase(prevName);

	_layerNames[index] = name;
	_layerIndex[name] = index;
}

uint8 SceneManager::LayerNameToIndex(const wstring& name)
{
	auto findIt = _layerIndex.find(name);
	if (findIt == _layerIndex.end())
		return 0;

	return findIt->second;
}
```

#### GameObject 클래스에 layerIndex 멤버 변수 추가 후 Set, Get 함수 추가

#### GameObject.h
```cpp
class GameObject : public Object, public enable_shared_from_this<GameObject>
{
public:
  void SetLayerIndex(uint8 layer) { _layerIndex = layer; }
  uint8 GetLayerIndex() { return _layerIndex; }

private:
  uint8 _layerIndex = 0;
}
```

#### Camera 클래스에 cullingMask 멤버 변수 추가 후 관련 함수 추가

#### Camera.h
```cpp
class Camera : public Component
{
public:
  void SetProjectionType(PROJECTION_TYPE type) { _type = type; }
  PROJECTION_TYPE GetProjectionType() { return _type; }

  void SetCullingMaskLayerOnOff(uint8 layer, bool on)
  {
    if (on)
      _cullingMask |= (1 << layer);
    else
      _cullingMask &= ~(1 << layer);
  }

  void SetCullingMaskAll() { SetCullingMask(UINT32_MAX); }
  void SetCullingMask(uint32 mask) { _cullingMask = mask; }
  bool IsCulled(uint8 layer) { return (_cullingMask & (1 << layer)) != 0; }

private:
  uint32 _cullingMask = 0;
}
```

#### Camera Render 함수에 Layer를 이용한 간단한 컬링 코드 추가 후 각 카메라마다 각자의 변환 행렬이 적용되도록 FinalUpdate 함수에 있는 코드 추가 

#### Camera.cpp
```cpp
void Camera::Render()
{
	S_MatView = _matView;
	S_MatProjection = _matProjection;

	shared_ptr<Scene> scene = GET_SINGLE(SceneManager)->GetActiveScene();

	const vector<shared_ptr<GameObject>>& gameObjects = scene->GetGameObjects();

	for (auto& gameObject : gameObjects)
	{
		if (gameObject->GetMeshRenderer() == nullptr)
			continue;

		if (IsCulled(gameObject->GetLayerIndex()))
			continue;

		if (gameObject->GetCheckFrustum())
		{
			if (_frustum.ContainsSphere(
				gameObject->GetTransform()->GetWorldPosition(),
				gameObject->GetTransform()->GetBoundingSphereRadius()) == false)
			{
				continue;
			}
		}

		gameObject->GetMeshRenderer()->Render();
	}
}
```

#### Resources 클래스에 Init, CreateDefaultShader 함수 추가

#### Resources.h
```cpp
class Resources
{
public:
	void Init();

private:
	void CreateDefaultShader();
}
```

#### Resources.cpp
```cpp
void Resources::Init()
{
	CreateDefaultShader();
}

void Resources::CreateDefaultShader()
{
	// Skybox
	{
		ShaderInfo info =
		{
			RASTERIZER_TYPE::CULL_NONE,
			DEPTH_STENCIL_TYPE::LESS_EQUAL
		};

		shared_ptr<Shader> shader = make_shared<Shader>();
		shader->Init(L"..\\Resources\\Shader\\skybox.fx", info);
		Add<Shader>(L"Skybox", shader);
	}

	// Forward
	{
		ShaderInfo info =
		{
		};

		shared_ptr<Shader> shader = make_shared<Shader>();
		shader->Init(L"..\\Resources\\Shader\\forward.fx", info);
		Add<Shader>(L"Forward", shader);
	}
}
```

#### Engine Init 함수에 Resources Init 코드 추가

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

	CreateConstantBuffer(CBV_REGISTER::b0, sizeof(LightParams), 1);
	CreateConstantBuffer(CBV_REGISTER::b1, sizeof(TransformParams), 256);
	CreateConstantBuffer(CBV_REGISTER::b2, sizeof(MaterialParams), 256);

	ResizeWindow(window.width, window.height);

	GET_SINGLE(Input)->Init(window.hwnd);
	GET_SINGLE(Timer)->Init();
	GET_SINGLE(Resources)->Init();
}
```

#### Texture Init 함수 이름을 Load로 변경

#### Texture.h
```cpp
class Texture : public Object
{
public:
	virtual void Load(const wstring& path);
}
```

#### Texture.cpp
```cpp
void Texture::Load(const wstring& path)
{
	CreateTexture(path);
	CreateView();
}
```

#### SceneManager LoadTestScene 함수에 LayerMask, UI_Camera, UI_Test 추가 후 변경된 내용에 따라 SkyBox, Cube 코드 수정

#### SceneManager.cpp
```cpp
shared_ptr<Scene> SceneManager::LoadTestScene()
{
#pragma region LayerMask
	SetLayerName(0, L"Default");
	SetLayerName(1, L"UI");
#pragma endregion

	shared_ptr<Scene> scene = make_shared<Scene>();

#pragma region Camera
	shared_ptr<GameObject> camera = make_shared<GameObject>();
	camera->SetName(L"Main_Camera");
	camera->AddComponent(make_shared<Transform>());
	camera->AddComponent(make_shared<Camera>()); // Near=1, Far=1000, FOV=45도
	camera->AddComponent(make_shared<TestCameraScript>());
	camera->GetTransform()->SetLocalPosition(Vec3(0.f, 0.f, 0.f));
	uint8 layerIndex = GET_SINGLE(SceneManager)->LayerNameToIndex(L"UI");
	scene->AddGameObject(camera);
#pragma endregion

#pragma region UI_Camera
	{
		shared_ptr<GameObject> camera = make_shared<GameObject>();
		camera->SetName(L"Orthographic_Camera");
		camera->AddComponent(make_shared<Transform>());
		camera->AddComponent(make_shared<Camera>()); // Near=1, Far=1000, 800*600
		camera->GetTransform()->SetLocalPosition(Vec3(0.f, 0.f, 0.f));
		camera->GetCamera()->SetProjectionType(PROJECTION_TYPE::ORTHOGRAPHIC);
		uint8 layerIndex = GET_SINGLE(SceneManager)->LayerNameToIndex(L"UI");
		camera->GetCamera()->SetCullingMaskAll(); // 다 끄기
		camera->GetCamera()->SetCullingMaskLayerOnOff(layerIndex, false); // UI만 찍음
		scene->AddGameObject(camera);
	}
#pragma endregion

#pragma region SkyBox
	{
		shared_ptr<GameObject> skybox = make_shared<GameObject>();
		skybox->AddComponent(make_shared<Transform>());
		skybox->SetCheckFrustum(false);
		shared_ptr<MeshRenderer> meshRenderer = make_shared<MeshRenderer>();
		{
			shared_ptr<Mesh> sphereMesh = GET_SINGLE(Resources)->LoadSphereMesh();
			meshRenderer->SetMesh(sphereMesh);
		}
		{
			shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"Skybox");
			shared_ptr<Texture> texture = GET_SINGLE(Resources)->Load<Texture>(L"Sky01", L"..\\Resources\\Texture\\Sky01.jpg");
			shared_ptr<Material> material = make_shared<Material>();
			material->SetShader(shader);
			material->SetTexture(0, texture);
			meshRenderer->SetMaterial(material);
		}
		skybox->AddComponent(meshRenderer);
		scene->AddGameObject(skybox);
	}
#pragma endregion

#pragma region Cube
	{
		shared_ptr<GameObject> sphere = make_shared<GameObject>();
		sphere->AddComponent(make_shared<Transform>());
		sphere->GetTransform()->SetLocalScale(Vec3(100.f, 100.f, 100.f));
		sphere->GetTransform()->SetLocalPosition(Vec3(0.f, 0.f, 150.f));
		shared_ptr<MeshRenderer> meshRenderer = make_shared<MeshRenderer>();
		{
			shared_ptr<Mesh> sphereMesh = GET_SINGLE(Resources)->LoadCubeMesh();
			meshRenderer->SetMesh(sphereMesh);
		}
		{
			shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"Forward");
			shared_ptr<Texture> texture = GET_SINGLE(Resources)->Load<Texture>(L"Leather", L"..\\Resources\\Texture\\Leather.jpg");
			shared_ptr<Texture> texture2 = GET_SINGLE(Resources)->Load<Texture>(L"Leather_Normal", L"..\\Resources\\Texture\\Leather_Normal.jpg");
			shared_ptr<Material> material = make_shared<Material>();
			material->SetShader(shader);
			material->SetTexture(0, texture);
			material->SetTexture(1, texture2);
			meshRenderer->SetMaterial(material);
		}
		sphere->AddComponent(meshRenderer);
		scene->AddGameObject(sphere);
	}
#pragma endregion

#pragma region UI_Test
	{
		shared_ptr<GameObject> sphere = make_shared<GameObject>();
		sphere->SetLayerIndex(GET_SINGLE(SceneManager)->LayerNameToIndex(L"UI")); // UI
		sphere->AddComponent(make_shared<Transform>());
		sphere->GetTransform()->SetLocalScale(Vec3(100.f, 100.f, 100.f));
		sphere->GetTransform()->SetLocalPosition(Vec3(0, 0, 500.f));
		shared_ptr<MeshRenderer> meshRenderer = make_shared<MeshRenderer>();
		{
			shared_ptr<Mesh> mesh = GET_SINGLE(Resources)->LoadRectangleMesh();
			meshRenderer->SetMesh(mesh);
		}
		{
			shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"Forward");
			shared_ptr<Texture> texture = GET_SINGLE(Resources)->Load<Texture>(L"Leather", L"..\\Resources\\Texture\\Leather.jpg");
			shared_ptr<Material> material = make_shared<Material>();
			material->SetShader(shader);
			material->SetTexture(0, texture);
			meshRenderer->SetMaterial(material);
		}
		sphere->AddComponent(meshRenderer);
		scene->AddGameObject(sphere);
	}
#pragma endregion

#pragma region Green Directional Light
	{
		shared_ptr<GameObject> light = make_shared<GameObject>();
		light->AddComponent(make_shared<Transform>());
		light->AddComponent(make_shared<Light>());
		light->GetLight()->SetLightDirection(Vec3(1.f, 0.f, 1.f));
		light->GetLight()->SetLightType(LIGHT_TYPE::DIRECTIONAL_LIGHT);
		light->GetLight()->SetDiffuse(Vec3(0.5f, 0.5f, 0.5f));
		light->GetLight()->SetAmbient(Vec3(0.1f, 0.1f, 0.1f));
		light->GetLight()->SetSpecular(Vec3(0.3f, 0.3f, 0.3f));

		scene->AddGameObject(light);
	}

#pragma endregion

	return scene;
}
```

#### Resources 클래스에 LoadRectangleMesh 함수 추가

#### Resources.h
```cpp
class Resources
{
public:
	shared_ptr<Mesh> LoadRectangleMesh();
}
```

#### Resources.cpp
```cpp
shared_ptr<Mesh> Resources::LoadRectangleMesh()
{
	shared_ptr<Mesh> findMesh = Get<Mesh>(L"Rectangle");
	if (findMesh)
		return findMesh;

	float w2 = 0.5f;
	float h2 = 0.5f;

	vector<Vertex> vec(4);

	// 앞면
	vec[0] = Vertex(Vec3(-w2, -h2, 0), Vec2(0.0f, 1.0f), Vec3(0.0f, 0.0f, -1.0f), Vec3(1.0f, 0.0f, 0.0f));
	vec[1] = Vertex(Vec3(-w2, +h2, 0), Vec2(0.0f, 0.0f), Vec3(0.0f, 0.0f, -1.0f), Vec3(1.0f, 0.0f, 0.0f));
	vec[2] = Vertex(Vec3(+w2, +h2, 0), Vec2(1.0f, 0.0f), Vec3(0.0f, 0.0f, -1.0f), Vec3(1.0f, 0.0f, 0.0f));
	vec[3] = Vertex(Vec3(+w2, -h2, 0), Vec2(1.0f, 1.0f), Vec3(0.0f, 0.0f, -1.0f), Vec3(1.0f, 0.0f, 0.0f));

	vector<uint32> idx(6);

	// 앞면
	idx[0] = 0; idx[1] = 1; idx[2] = 2;
	idx[3] = 0; idx[4] = 2; idx[5] = 3;

	shared_ptr<Mesh> mesh = make_shared<Mesh>();
	mesh->Init(vec, idx);
	Add(L"Rectangle", mesh);

	return mesh;
}
```

출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
