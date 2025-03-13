---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] Input과 Timer"
date: 2025-03-13
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - Component
---



{% capture notice-1 %}
* 키보드, 마우스 등의 입력을 감지하는 클래스
{% endcapture %}

{% capture notice-2 %}
* 시간을 측정하는 클래스
{% endcapture %}

{% capture notice-2 %}
* 전방 선언의 경우 헤더에서 make_shared가 불가능하지만 헤더를 포함한 경우에서는 가능
{% endcapture %}

#### Engine 프로젝트 Engine 필터에 Input, Timer 클래스 추가

#### Input.h
```cpp
enum class KEY_TYPE
{
	UP = VK_UP,
	DOWN = VK_DOWN,
	LEFT = VK_LEFT,
	RIGHT = VK_RIGHT,

	W = 'W',
	A = 'A',
	S = 'S',
	D = 'D',
};

enum class KEY_STATE
{
	NONE,
	PRESS,
	DOWN,
	UP,
	END
};

enum
{
	KEY_TYPE_COUNT = static_cast<int32>(UINT8_MAX),
	KEY_STATE_COUNT = static_cast<int32>(KEY_STATE::END),
};


class Input
{
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

#### Input.cpp
```cpp
#include "pch.h"
#include "Input.h"

void Input::Init(HWND hwnd)
{
	_hwnd = hwnd;
	_states.resize(KEY_TYPE_COUNT, KEY_STATE::NONE);
}

void Input::Update()
{
	HWND hwnd = ::GetActiveWindow();
	if (_hwnd != hwnd)
	{
		for (uint32 key = 0; key < KEY_TYPE_COUNT; key++)
			_states[key] = KEY_STATE::NONE;

		return;
	}

	for (uint32 key = 0; key < KEY_TYPE_COUNT; key++)
	{
		// 키가 눌려 있으면 true
		if (::GetAsyncKeyState(key) & 0x8000)
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

#### Timer.h
```cpp
class Timer
{
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

#### Timer.cpp
```cpp
#include "pch.h"
#include "Timer.h"

void Timer::Init()
{
	::QueryPerformanceFrequency(reinterpret_cast<LARGE_INTEGER*>(&_frequency));
	::QueryPerformanceCounter(reinterpret_cast<LARGE_INTEGER*>(&_prevCount)); // CPU 클럭
}

void Timer::Update()
{
	uint64 currentCount;
	::QueryPerformanceCounter(reinterpret_cast<LARGE_INTEGER*>(&currentCount));

	_deltaTime = (currentCount - _prevCount) / static_cast<float>(_frequency);
	_prevCount = currentCount;

	_frameCount++;
	_frameTime += _deltaTime;

	if (_frameTime > 1.f)
	{
		_fps = static_cast<uint32>(_frameCount / _frameTime);

		_frameTime = 0.f;
		_frameCount = 0;
	}
}
```

<div class="notice">
  {{ notice-2 | markdownify }}
</div>

#### Engine 클래스에 Input, Timer 헤더 추가, Input, Timer 멤버 변수 및 Get 함수 추가 

#### Engine.h
```cpp
#include "Input.h"
#include "Timer.h"

class Engine
{
public:
	shared_ptr<Input> GetInput() { return _input; }
	shared_ptr<Timer> GetTimer() { return _timer; }
private:
	shared_ptr<Input> _input = make_shared<Input>();
	shared_ptr<Timer> _timer = make_shared<Timer>();
}
```

#### Engine Init 함수 make_shared 전부 헤더로 옮긴 뒤 Input, Timer 초기화 코드 추가

#### Engine.h
```cpp
private:
	// 그려질 화면 관련
	WindowInfo _window;
	D3D12_VIEWPORT _viewport{};
	D3D12_RECT _scissorRect{};

	shared_ptr<Device> _device = make_shared<Device>();
	shared_ptr<CommandQueue> _cmdQueue = make_shared<CommandQueue>();
	shared_ptr<SwapChain> _swapChain = make_shared<SwapChain>();
	shared_ptr<RootSignature> _rootSignature = make_shared<RootSignature>();
	shared_ptr<ConstantBuffer> _cb = make_shared<ConstantBuffer>();
	shared_ptr<TableDescriptorHeap> _tableDescHeap = make_shared<TableDescriptorHeap>();
	shared_ptr<DepthStencilBuffer> _depthStencilBuffer = make_shared<DepthStencilBuffer>();
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
	_cb->Init(sizeof(Transform), 256);
	_tableDescHeap->Init(256);
	_depthStencilBuffer->Init(window);

	_input->Init(window.hwnd);
	_timer->Init();

	ResizeWindow(window.width, window.height);
}
```

<div class="notice">
  {{ notice-3 | markdownify }}
</div>

#### Engine 클래스에 Update 함수 추가

#### Engine.h
```cpp
public:
	void Update();
```

#### Engine.cpp
```cpp
void Engine::Update()
{
	_input->Update();
	_timer->Update();
}
```

#### EnginePch 클래스에 INPUT, DELTA_TIME 매크로 추가

#### EnginePch.h
```cpp
#define INPUT				GEngine->GetInput()
#define DELTA_TIME			GEngine->GetTimer()->GetDeltaTime()
```

#### Game 클래스 Update 함수에 GEngine Update 코드 추가 및 Input 테스트 코드 추가

#### Game.cpp
```cpp
void Game::Update()
{
	GEngine->Update();

	GEngine->RenderBegin();

	shader->Update();

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

		mesh->SetTexture(texture);

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

#### Engine 클래스에 ShowFps 함수 추가 및 Update 함수에서 ShowFps 함수 호출

#### Engine.h
```cpp
private:
	void ShowFps();
```

#### Engine.cpp
```cpp
void Engine::Update()
{
	_input->Update();
	_timer->Update();

	ShowFps();
}

void Engine::ShowFps()
{
	uint32 fps = _timer->GetFps();

	WCHAR text[100] = L"";
	::wsprintf(text, L"FPS : %d", fps);

	::SetWindowText(_window.hwnd, text);
}
```

출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
