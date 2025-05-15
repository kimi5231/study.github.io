---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] Lighting #2"
date: 2025-05-15
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - Camera & Lighting
---



{% capture notice-1 %}
* Light는 한 프레임 당 한 번만 셋팅해주면 됨 => 전역
* 전역은 메모리가 크지 않기 때문에 Descriptor를 사용하고, 오브젝트마다 교체가 필요한 것은 메모리가 크기 때문에 DescriptorHeap을 사용함
{% endcapture %}

{% capture notice-2 %}
* default: 쉐이더와 관련된 코드 모음
* params: 파라미터 모음
* utils: 쉐이더 파일에서 사용할 함수 코드 모음
{% endcapture %}

{% capture notice-3 %}
#### padding 룰

* 쉐이더 파일에서는 16byte로 끊어 읽기 때문에 그에 맞춰 정렬하여 구조체를 만드는 것이 좋음
* 16byte 바운더리를 넘을 수 없기 때문에 순서가 중요함
{% endcapture %}



#### Component 필터에 Light 클래스 추가

#### Light.h
```cpp
#pragma once
#include "Component.h"

enum class LIGHT_TYPE : uint8
{
	DIRECTIONAL_LIGHT,
	POINT_LIGHT,
	SPOT_LIGHT,
};

struct LightColor
{
	Vec4	diffuse;
	Vec4	ambient;
	Vec4	specular;
};

struct LightInfo
{
	LightColor	color;
	Vec4		position;
	Vec4		direction;
	int32		lightType;
	float		range;
	float		angle;
	int32		padding;
};

struct LightParams
{
	uint32		lightCount;
	Vec3		padding;
	LightInfo	lights[50];
};

class Light : public Component
{
public:
	Light();
	virtual ~Light();

	virtual void FinalUpdate() override;

public:
	const LightInfo& GetLightInfo() { return _lightInfo; }

	void SetLightDirection(const Vec3& direction) { _lightInfo.direction = direction; }

	void SetDiffuse(const Vec3& diffuse) { _lightInfo.color.diffuse = diffuse; }
	void SetAmbient(const Vec3& ambient) { _lightInfo.color.ambient = ambient; }
	void SetSpecular(const Vec3& specular) { _lightInfo.color.specular = specular; }

	void SetLightType(LIGHT_TYPE type) { _lightInfo.lightType = static_cast<int32>(type); }
	void SetLightRange(float range) { _lightInfo.range = range; }
	void SetLightAngle(float angle) { _lightInfo.angle = angle; }

private:
	LightInfo _lightInfo = {};
};
```

#### Light.cpp
```cpp
#include "pch.h"
#include "Light.h"
#include "Transform.h"

Light::Light() : Component(COMPONENT_TYPE::LIGHT)
{
}

Light::~Light()
{
}

void Light::FinalUpdate()
{
	_lightInfo.position = GetTransform()->GetWorldPosition();
}
```

#### SimpleMath 헤더에 Vector3를 Vector4로 변환하는 오버로딩 함수 추가

#### SimpleMath.h
```cpp
Vector4& operator=(const Vector3& V) noexcept { x = V.x; y = V.y; z = V.z; w = 0.f; return *this; }
```

#### COMPONENT_TYPE enum class에 LIGHT 추가

#### Component.h
```cpp
enum class COMPONENT_TYPE : uint8
{
	TRANSFORM,
	MESH_RENDERER,
	CAMERA,
	LIGHT,

	MONO_BEHAVIOUR,
	END,
};
```

#### GameObject 클래스에 GetLight 함수 추가

#### GameObject.h
```cpp
class Light;

class GameObject : public Object, public enable_shared_from_this<GameObject>
{
  public:
    shared_ptr<Light> GetLight();
}
```

#### GameObject.cpp
```cpp
shared_ptr<Light> GameObject::GetLight()
{
	shared_ptr<Component> component = GetFixedComponent(COMPONENT_TYPE::LIGHT);
	return static_pointer_cast<Light>(component);
}
```

<div class="notice">
  {{ notice-1 | markdownify }}
</div>

#### b0를 전역으로 사용하기 위해 RootSignature CreateRootSignature 함수 코드 수정

#### RootSignature.cpp
```cpp
void RootSignature::CreateRootSignature()
{
	CD3DX12_DESCRIPTOR_RANGE ranges[] =
	{
		CD3DX12_DESCRIPTOR_RANGE(D3D12_DESCRIPTOR_RANGE_TYPE_CBV, CBV_REGISTER_COUNT - 1, 1), // b1~b4
		CD3DX12_DESCRIPTOR_RANGE(D3D12_DESCRIPTOR_RANGE_TYPE_SRV, SRV_REGISTER_COUNT, 0), // t0~t4
	};

	CD3DX12_ROOT_PARAMETER param[2];
	param[0].InitAsConstantBufferView(static_cast<uint32>(CBV_REGISTER::b0)); // b0
	param[0].InitAsDescriptorTable(_countof(ranges), ranges);

	D3D12_ROOT_SIGNATURE_DESC sigDesc = CD3DX12_ROOT_SIGNATURE_DESC(_countof(param), param, 1, &_samplerDesc);
	sigDesc.Flags = D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT; // 입력 조립기 단계

	ComPtr<ID3DBlob> blobSignature;
	ComPtr<ID3DBlob> blobError;
	::D3D12SerializeRootSignature(&sigDesc, D3D_ROOT_SIGNATURE_VERSION_1, &blobSignature, &blobError);
	DEVICE->CreateRootSignature(0, blobSignature->GetBufferPointer(), blobSignature->GetBufferSize(), IID_PPV_ARGS(&_signature));
}
```

#### TableDescriptorHeap Init, GetCPUHandle 함수도 바뀐 RootSignature에 맞춰 코드 수정

#### TableDescriptorHeap.cpp
```cpp
void TableDescriptorHeap::Init(uint32 count)
{
	_groupCount = count;

	D3D12_DESCRIPTOR_HEAP_DESC desc = {};
	desc.NumDescriptors = count * (REGISTER_COUNT - 1); // b0는 전역
	desc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_SHADER_VISIBLE;
	desc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV;

	DEVICE->CreateDescriptorHeap(&desc, IID_PPV_ARGS(&_descHeap));

	_handleSize = DEVICE->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV);
	_groupSize = _handleSize * (REGISTER_COUNT - 1); // b0는 전역
}

D3D12_CPU_DESCRIPTOR_HANDLE TableDescriptorHeap::GetCPUHandle(uint8 reg)
{
	assert(reg > 0);
	D3D12_CPU_DESCRIPTOR_HANDLE handle = _descHeap->GetCPUDescriptorHandleForHeapStart();
	handle.ptr += _currentGroupIndex * _groupSize;
	handle.ptr += (reg - 1) * _handleSize;
	return handle;
}
```

#### Engine Init 함수도 추가된 조명에 맞춰 코드 수정

#### Engine.cpp
```cpp
#include "Light.h"

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
}
```

#### Constantbuffer 클래스에 전역을 위한 SetGlobalData 함수 추가

#### Constantbuffer.h
```cpp
class ConstantBuffer
{
public:
  void SetGlobalData(void* buffer, uint32 size);
}
```

#### Constantbuffer.cpp
```cpp
void ConstantBuffer::SetGlobalData(void* buffer, uint32 size)
{
	assert(_elementSize == ((size + 255) & ~255));
	::memcpy(&_mappedBuffer[0], buffer, size);
	CMD_LIST->SetGraphicsRootConstantBufferView(0, GetGpuVirtualAddress(0));
}
```

#### CONSTANT_BUFFER_TYPE enum class에 GLOBAL 추가

#### Constantbuffer.h
```cpp
enum class CONSTANT_BUFFER_TYPE : uint8
{
	GLOBAL,
	TRANSFORM,
	MATERIAL,
	END
};
```

#### SceneManager Render에 있는 내용을 Scene 클래스에 Render 함수 추가 후 옮기기

#### SceneManager.cpp
```cpp
void SceneManager::Render()
{
	if (_activeScene)
		_activeScene->Render();
}
```

#### Scene.h
```cpp
class Scene
{
public:
  void Render();
}
```

#### Scene.cpp
```cpp
#include "Camera.h"
#include "Engine.h"
#include "ConstantBuffer.h"
#include "Light.h"

void Scene::Render()
{
	for (auto& gameObject : _gameObjects)
	{
		if (gameObject->GetCamera() == nullptr)
			continue;

		gameObject->GetCamera()->Render();
	}
}
```

#### Scene 클래스에 PushLightData 함수 추가 후 Render 함수에 호출 코드 추가

#### Scene.h
```cpp
class Scene
{
private:
	void PushLightData();
}
```

#### Scene.cpp
```cpp
void Scene::Render()
{
	PushLightData();

	for (auto& gameObject : _gameObjects)
	{
		if (gameObject->GetCamera() == nullptr)
			continue;

		gameObject->GetCamera()->Render();
	}
}

void Scene::PushLightData()
{
	LightParams lightParams = {};

	for (auto& gameObject : _gameObjects)
	{
		if (gameObject->GetLight() == nullptr)
			continue;

		const LightInfo& lightInfo = gameObject->GetLight()->GetLightInfo();

		lightParams.lights[lightParams.lightCount] = lightInfo;
		lightParams.lightCount++;
	}

	CONST_BUFFER(CONSTANT_BUFFER_TYPE::GLOBAL)->SetGlobalData(&lightParams, sizeof(lightParams));
}
```

#### TransformParams 구조체에 변환 행렬 추가

#### EnginePch.h
```cpp
struct TransformParams
{
	Matrix matWorld;
	Matrix matView;
	Matrix matProjection;
	Matrix matWV;
	Matrix matWVP;
};
```

#### Transform PushData 함수에서 변환 행렬 셋팅

#### Transform.cpp
```cpp
void Transform::PushData()
{
	TransformParams transformParams = {};
	transformParams.matWorld = _matWorld;
	transformParams.matView = Camera::S_MatView;
	transformParams.matProjection = Camera::S_MatProjection;
	transformParams.matWV = _matWorld * Camera::S_MatView;
	transformParams.matWVP = _matWorld * Camera::S_MatView * Camera::S_MatProjection;

	CONST_BUFFER(CONSTANT_BUFFER_TYPE::TRANSFORM)->PushData(&transformParams, sizeof(transformParams));
}
```

#### 쉐이더 파일에 있던 기존 파라미터들의 이름 수정 후 LightColor, LightInfo, GLOBAL_PARAMS 추가

#### default.hlsli
```cpp
struct LightColor
{
    float4      diffuse;
    float4      ambient;
    float4      specular;
};

struct LightInfo
{
    LightColor  color;
    float4	    position;
    float4	    direction; 
    int		    lightType;
    float	    range;
    float	    angle;
    int  	    padding;
};

cbuffer GLOBAL_PARAMS : register(b0)
{
    int         g_lightCount;
    float3      g_lightPadding;
    LightInfo   g_light[50];
}

cbuffer TRANSFORM_PARAMS : register(b1)
{
    row_major matrix g_matWorld;
    row_major matrix g_matView;
    row_major matrix g_matProjection;
    row_major matrix g_matWV;
    row_major matrix g_matWVP;
};

cbuffer MATERIAL_PARAMS : register(b2)
{
    int     g_int_0;
    int     g_int_1;
    int     g_int_2;
    int     g_int_3;
    int     g_int_4;
    float   g_float_0;
    float   g_float_1;
    float   g_float_2;
    float   g_float_3;
    float   g_float_4;
};

Texture2D g_tex_0 : register(t0);
Texture2D g_tex_1 : register(t1);
Texture2D g_tex_2 : register(t2);
Texture2D g_tex_3 : register(t3);
Texture2D g_tex_4 : register(t4);

SamplerState g_sam_0 : register(s0);
```

#### Shader 필터에 params, utils 쉐이더 파일 추가 후 용도에 따라 파일 분할

#### default.hlsli
```cpp
#ifdef _DEFAULT_HLSL_
#define _DEFAULT_HLSL_

#include "params.hlsli"

struct VS_IN
{
    float3 pos : POSITION;
    float2 uv : TEXCOORD;
    float3 normal : NORMAL;
};

struct VS_OUT
{
    float4 pos : SV_Position;
    float2 uv : TEXCOORD;
    float3 viewPos : POSITION;
    float3 viewNormal : NORMAL;
};

VS_OUT VS_Main(VS_IN input)
{
    VS_OUT output = (VS_OUT) 0;

    output.pos = mul(float4(input.pos, 1.f), g_matWVP);
    output.uv = input.uv;

    output.viewPos = mul(float4(input.pos, 1.f), g_matWV).xyz;
    output.viewNormal = normalize(mul(float4(input.normal, 0.f), g_matWV).xyz);

    return output;
}

float4 PS_Main(VS_OUT input) : SV_Target
{
    float4 color = tex_0.Sample(g_sam_0, input.uv);
    return color;
}

#endif
```

#### params.hlsli
```cpp
#ifdef _PARAMS_HLSL_
#define _PARAMS_HLSL_

struct LightColor
{
    float4      diffuse;
    float4      ambient;
    float4      specular;
};

struct LightInfo
{
    LightColor  color;
    float4	    position;
    float4	    direction; 
    int		    lightType;
    float	    range;
    float	    angle;
    int  	    padding;
};

cbuffer GLOBAL_PARAMS : register(b0)
{
    int         g_lightCount;
    float3      g_lightPadding;
    LightInfo   g_light[50];
}

cbuffer TRANSFORM_PARAMS : register(b1)
{
    row_major matrix g_matWorld;
    row_major matrix g_matView;
    row_major matrix g_matProjection;
    row_major matrix g_matWV;
    row_major matrix g_matWVP;
};

cbuffer MATERIAL_PARAMS : register(b2)
{
    int     g_int_0;
    int     g_int_1;
    int     g_int_2;
    int     g_int_3;
    int     g_int_4;
    float   g_float_0;
    float   g_float_1;
    float   g_float_2;
    float   g_float_3;
    float   g_float_4;
};

Texture2D g_tex_0 : register(t0);
Texture2D g_tex_1 : register(t1);
Texture2D g_tex_2 : register(t2);
Texture2D g_tex_3 : register(t3);
Texture2D g_tex_4 : register(t4);

SamplerState g_sam_0 : register(s0);

#endif
```

<div class="notice">
  {{ notice-2 | markdownify }}
</div>

<div class="notice">
  {{ notice-3 | markdownify }}
</div>

출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
