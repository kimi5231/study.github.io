---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] Material"
date: 2025-03-20
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - Component
---



{% capture notice-1 %}
* KEY_TYPE_COUNT의 수를 하나 적게 한 것 수정
* 프레임이 잘 나오지 않아 Update 함수에 프레임이 잘 나올 수 있도록 코드 추가
{% endcapture %}

{% capture notice-2 %}
* Shader, Texture 등 Mesh를 그리기 위한 것들을 한 번에 관리하는 클래스
{% endcapture %}

#### Input 클래스 수정

#### Input.h
```cpp
enum
{
	KEY_TYPE_COUNT = static_cast<int32>(UINT8_MAX + 1),
	KEY_STATE_COUNT = static_cast<int32>(KEY_STATE::END),
};
```

#### Input.cpp
```cpp
void Input::Update()
{
	HWND hwnd = ::GetActiveWindow();
	if (_hwnd != hwnd)
	{
		for (uint32 key = 0; key < KEY_TYPE_COUNT; key++)
			_states[key] = KEY_STATE::NONE;

		return;
	}

	BYTE asciiKeys[KEY_TYPE_COUNT] = {};
	if (::GetKeyboardState(asciiKeys) == false)
		return;

	for (uint32 key = 0; key < KEY_TYPE_COUNT; key++)
	{
		// 키가 눌려 있으면 true
		if (asciiKeys[key] & 0x80)
		{
			KEY_STATE& state = _states[key];

			// 이전 프레임에 키를 누른 상태라면 PRESS
			if (state == KEY_STATE::PRESS || state == KEY_STATE::DOWN)
				state = KEY_STATE::PRESS;
			else
				state = KEY_STATE::DOWN;
		}
		else
		{
			KEY_STATE& state = _states[key];

			// 이전 프레임에 키를 누른 상태라면 UP
			if (state == KEY_STATE::PRESS || state == KEY_STATE::DOWN)
				state = KEY_STATE::UP;
			else
				state = KEY_STATE::NONE;
		}
	}
}
```

<div class="notice">
  {{ notice-1 | markdownify }}
</div>

#### ConstantBuffer 헤더에 enum, enum class 추가

#### ConstantBuffer.h
```cpp
enum class CONSTANT_BUFFER_TYPE : uint8
{
	TRANSFORM,
	MATERIAL,
	END
};

enum
{
	CONSTANT_BUFFER_COUNT = static_cast<uint8>(CONSTANT_BUFFER_TYPE::END)
};
```

#### ConstantBuffer 클래스에 CBV_REGISTER 멤버 변수 추가 및 Init, PushData 함수 수정

#### ConstantBuffer.h
```cpp
public:
	void Init(CBV_REGISTER  reg, uint32 size, uint32 count);

	void PushData(void* buffer, uint32 size);
private:
	CBV_REGISTER _reg{};
```

#### ConstantBuffer.cpp
```cpp
void ConstantBuffer::Init(CBV_REGISTER  reg, uint32 size, uint32 count)
{
	_reg = reg;

	// 상수 버퍼는 256byte의 배수로 만들어야 함
	_elementSize = (size + 255) & ~255;
	_elementCount = count;

	CreateBuffer();
	CreateView();
}

void ConstantBuffer::PushData(void* buffer, uint32 size)
{
	assert(_currentIndex < _elementCount);
	assert(_elementSize == ((size + 255) & ~255));

	::memcpy(&_mappedBuffer[_currentIndex * _elementSize], buffer, size);

	D3D12_CPU_DESCRIPTOR_HANDLE cpuHandle = GetCpuHandle(_currentIndex);
	GEngine->GetTableDescHeap()->SetCBV(cpuHandle, _reg);

	_currentIndex++;
}
```

#### Engine 클래스에 기존 ConstantBuffer 관련 내용 삭제 후 vector로 관리하는 ConstantBuffer 멤버 변수 추가 및 CreateConstantBuffer, GetConstantBuffer 함수 추가, Init 함수의 ConstantBuffer 관련 내용도 수정

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
#include "Input.h"
#include "Timer.h"

class Engine
{
public:
	void Init(const WindowInfo& window);
	void Render();

public:
	void Update();

public:
	shared_ptr<Device> GetDevice() { return _device; }
	shared_ptr<CommandQueue> GetCmdQueue() { return _cmdQueue; }
	shared_ptr<SwapChain> GetSwapChain() { return _swapChain; }
	shared_ptr<RootSignature> GetRootSignature() { return _rootSignature; }
	shared_ptr<TableDescriptorHeap> GetTableDescHeap() { return _tableDescHeap; }
	shared_ptr<DepthStencilBuffer> GetDepthStencilBuffer() { return _depthStencilBuffer; }

	shared_ptr<Input> GetInput() { return _input; }
	shared_ptr<Timer> GetTimer() { return _timer; }

	shared_ptr<ConstantBuffer> GetConstantBuffer(CONSTANT_BUFFER_TYPE type) { return _constantBuffers[static_cast<uint8>(type)]; }

public:
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

	shared_ptr<Input> _input = make_shared<Input>();
	shared_ptr<Timer> _timer = make_shared<Timer>();

	vector<shared_ptr<ConstantBuffer>> _constantBuffers;
};
```

#### Engine.cpp
```cpp
#include "Material.h"

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

	CreateConstantBuffer(CBV_REGISTER::b0, sizeof(Transform), 256);
	CreateConstantBuffer(CBV_REGISTER::b1, sizeof(MaterialParams), 256);

	ResizeWindow(window.width, window.height);
}

void Engine::CreateConstantBuffer(CBV_REGISTER reg, uint32 bufferSize, uint32 count)
{
	uint8 typeInt = static_cast<uint8>(reg);
	assert(_constantBuffers.size() == typeInt);

	shared_ptr<ConstantBuffer> buffer = make_shared<ConstantBuffer>();
	buffer->Init(reg, bufferSize, count);
	_constantBuffers.push_back(buffer);
}
```

#### Reasource 필터에 Material 클래스 추가

#### Material.h
```cpp
#pragma once

class Shader;
class Texture;

enum
{
	MATERIAL_INT_COUNT = 5,
	MATERIAL_FLOAT_COUNT = 5,
	MATERIAL_TEXTURE_COUNT = 5,
};

struct MaterialParams
{
	void SetInt(uint8 index, int32 value) { intParams[index] = value; }
	void SetFloat(uint8 index, float value) { floatParams[index] = value; }

	array<int32, MATERIAL_INT_COUNT> intParams;
	array<float, MATERIAL_FLOAT_COUNT> floatParams;
};

class Material
{
public:
	shared_ptr<Shader> GetShader() { return _shader; }

	void SetShader(shared_ptr<Shader> shader) { _shader = shader; }
	void SetInt(uint8 index, int32 value) { _params.SetInt(index, value); }
	void SetFloat(uint8 index, float value) { _params.SetFloat(index, value); }
	void SetTexture(uint8 index, shared_ptr<Texture> texture) { _textures[index] = texture; }

	void Update();

private:
	shared_ptr<Shader>	_shader;
	MaterialParams		_params;
	array<shared_ptr<Texture>, MATERIAL_TEXTURE_COUNT> _textures;
};
```

#### Material.cpp
```cpp
#include "pch.h"
#include "Material.h"
#include "Engine.h"

void Material::Update()
{
	// CBV 업로드
	CONST_BUFFER(CONSTANT_BUFFER_TYPE::MATERIAL)->PushData(&_params, sizeof(_params));

	// SRV 업로드
	for (size_t i = 0; i < _textures.size(); i++)
	{
		if (_textures[i] == nullptr)
			continue;

		SRV_REGISTER reg = SRV_REGISTER(static_cast<int8>(SRV_REGISTER::t0) + i);
		GEngine->GetTableDescHeap()->SetSRV(_textures[i]->GetCpuHandle(), reg);
	}

	// 파이프라인 세팅
	_shader->Update();
}
```

<div class="notice">
  {{ notice-2 | markdownify }}
</div>

#### EnginPch 헤더에 CONST_BUFFER 매크로 추가

#### EnginePch.h
```cpp
#define CONST_BUFFER(type)	GEngine->GetConstantBuffer(type)
```

#### 쉐이더 파일에 TEST_B1 ConstantBuffer MATERIAL_PARAMS로 변경 및 텍스처 개수 추가

#### default.hlsli
```hlsli
cbuffer MATERIAL_PARAMS : register(b1)
{
    int int_0;
    int int_1;
    int int_2;
    int int_3;
    int int_4;
    float float_0;
    float float_1;
    float float_2;
    float float_3;
    float float_4;
};

Texture2D tex_0 : register(t0);
Texture2D tex_1 : register(t1);
Texture2D tex_2 : register(t2);
Texture2D tex_3 : register(t3);
Texture2D tex_4 : register(t4);
```

#### CommandQueue RenderBegin 함수에서 ConstantBuffer 초기화하는 코드 수정

#### CommandQueue.cpp
```cpp
void CommandQueue::RenderBegin(const D3D12_VIEWPORT* vp, const D3D12_RECT* rect)
{
	_cmdAlloc->Reset();
	_cmdList->Reset(_cmdAlloc.Get(), nullptr);

	D3D12_RESOURCE_BARRIER barrier = CD3DX12_RESOURCE_BARRIER::Transition(
		_swapChain->GetBackRTVBuffer().Get(),
		D3D12_RESOURCE_STATE_PRESENT,
		D3D12_RESOURCE_STATE_RENDER_TARGET);

	// cmdList에 RootSignature 추가
	_cmdList->SetGraphicsRootSignature(ROOT_SIGNATURE.Get());
	// Constant Buffer 초기화
	GEngine->GetConstantBuffer(CONSTANT_BUFFER_TYPE::TRANSFORM)->Clear();
	GEngine->GetConstantBuffer(CONSTANT_BUFFER_TYPE::MATERIAL)->Clear();
	// TableDescriptorHeap 초기화
	GEngine->GetTableDescHeap()->Clear();

	// 어떤 DescriptorHeap을 사용할 것인지 지정
	ID3D12DescriptorHeap* descHeap = GEngine->GetTableDescHeap()->GetDescriptorHeap().Get();
	_cmdList->SetDescriptorHeaps(1, &descHeap);

	_cmdList->ResourceBarrier(1, &barrier);

	_cmdList->RSSetViewports(1, vp);
	_cmdList->RSSetScissorRects(1, rect);

	D3D12_CPU_DESCRIPTOR_HANDLE backBufferView = _swapChain->GetBackRTV();
	_cmdList->ClearRenderTargetView(backBufferView, Colors::LightSteelBlue, 0, nullptr);

	D3D12_CPU_DESCRIPTOR_HANDLE depthStencilView = GEngine->GetDepthStencilBuffer()->GetDSVCpuHandle();
	_cmdList->OMSetRenderTargets(1, &backBufferView, FALSE, &depthStencilView);

	_cmdList->ClearDepthStencilView(depthStencilView, D3D12_CLEAR_FLAG_DEPTH, 1.0f, 0, 0, nullptr);
}
```

#### Mesh 클래스 Texture 관련 내용 삭제 후 Material 관련 내용 추가 및 Render 함수 수정

#### Mesh.h
```cpp
class Material;

class Mesh
{
public:
	void Init(const vector<Vertex>& vertexBuffer, const vector<uint32>& indexBuffer);
	void Render();

	void SetTransform(const Transform& t) { _transform = t; }
	void SetMaterial(shared_ptr<Material> mat) { _mat = mat; }

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

	Transform _transform{};
	shared_ptr<Material> _mat = {};
};
```

#### Mesh.cpp
```cpp
void Mesh::Render()
{
	CMD_LIST->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST);	// 정점 연결 방식
	CMD_LIST->IASetVertexBuffers(0, 1, &_vertexBufferView); // Slot: (0~15)
	CMD_LIST->IASetIndexBuffer(&_indexBufferView);

	CONST_BUFFER(CONSTANT_BUFFER_TYPE::TRANSFORM)->PushData(&_transform, sizeof(_transform));

	_mat->Update();

	GEngine->GetTableDescHeap()->CommitTable();

	// 그리기 예약
	
	// Vertex로 그리기
	//CMD_LIST->DrawInstanced(_vertexCount, 1, 0, 0);

	// Index로 그리기
	CMD_LIST->DrawIndexedInstanced(_indexCount, 1, 0, 0, 0);
}
```

#### 쉐이더 파일에 float 사용

#### default.hlsli
```hlsli
VS_OUT VS_Main(VS_IN input)
{
    VS_OUT output = (VS_OUT) 0;

    output.pos = float4(input.pos, 1.f);
    //output.pos += offset0;
    output.pos.x += float_0;
    output.pos.y += float_1;
    output.pos.z += float_2;
    
    output.color = input.color;
    output.uv = input.uv;

    return output;
}
```

#### Game 클래스 Init, Update 함수 수정 및 Mesh, Shader 전역변수 삭제

#### Game.cpp
```cpp
#include "pch.h"
#include "Game.h"
#include "Engine.h"
#include "Material.h"

shared_ptr<Mesh> mesh = make_shared<Mesh>();

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

	mesh->Init(vec, indexVec);

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
	mesh->SetMaterial(material);

	GEngine->GetCmdQueue()->WaitSync();
}

void Game::Update()
{
	GEngine->Update();

	GEngine->RenderBegin();

	{
		static Transform t = {};

		if (INPUT->GetButton(KEY_TYPE::W))
			t.offset.y += 1.f * DELTA_TIME;
		if (INPUT->GetButton(KEY_TYPE::S))
			t.offset.y -= 1.f * DELTA_TIME;
		if (INPUT->GetButton(KEY_TYPE::A))
			t.offset.x -= 1.f * DELTA_TIME;
		if (INPUT->GetButton(KEY_TYPE::D))
			t.offset.x += 1.f * DELTA_TIME;

		mesh->SetTransform(t);

		mesh->Render();
	}

	/*{
		Transform t;
		t.offset = Vec4(0.25f, 0.25f, 0.1f, 0.f);
		mesh->SetTransform(t);
		mesh->SetTexture(texture);

		mesh->Render();
	}*/

	GEngine->RenderEnd();
}
```

출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
