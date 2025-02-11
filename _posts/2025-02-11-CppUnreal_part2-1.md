---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] 프로젝트 설정"
date: 2025-02-11
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - DirectX12 초기화
---



{% capture notice-1 %}
* 자주 사용할 헤더들을 모아두는 클래스
{% endcapture %}

{% capture notice-2 %}
* 정적 라이브러리: 프로그램에 포함되어 있는 라이브러리
* 동적 라이브러리: 프로그램을 실행하면 가져와서 사용하는 라이브러리
{% endcapture %}

{% capture notice-3 %}
* 자주 사용할 헤더, 라이브러리, 타입들을 모아두는 클래스
{% endcapture %}

#### 프로젝트 설정

![프로젝트 생성](https://github.com/user-attachments/assets/0f256df2-a6b2-4c2b-8f93-772860b4f89c)


1) Windows 데스크톱 애플리케이션 프로젝트를 생성한다.

2) 게임 개발을 위해 윈도우API의 기본 메시지 루프를 수정한다. 

#### client.cpp
```cpp
 while (true)
 {
     // 메세지가 있는지 없는지 계속 확인
     if (PeekMessage(&msg, nullptr, 0, 0, PM_REMOVE))
     {
         // 프로그램 종료
         if (msg.message == WM_QUIT)
             break;
     
         // 메세지 처리
         if (!TranslateAccelerator(msg.hwnd, hAccelTable, &msg))
         {
             TranslateMessage(&msg);
             DispatchMessage(&msg);
         }
     }
 }
```

![미리 컴파일된 헤더 설정](https://github.com/user-attachments/assets/65ec808c-2d5b-4b15-9ece-06ee439a317d)


3) client 프로젝트의 미리 컴파일된 헤더를 사용으로 변경하고 파일 이름을 pch.h로 변경한다. 이때, 구성과 플랫폼은 모든 구성과 모든 플랫폼으로 바꾼다.

![폴더 정리](https://github.com/user-attachments/assets/26bcdb78-84d7-4b8e-9214-3b7e6b502b3a)


4) 소스 파일 폴더에 Game, Utils 폴더를 추가한 뒤, 헤더 파일에 있던 파일들을 전부 Utils 폴더에 옮긴 뒤 헤더 파일 폴더를 삭제한다.

5) Utils 폴더에 pch 클래스를 추가한다.

#### pch.h
```cpp
#pragma once

#include <vector>
#include <memory>

using namespace std;
```

<div class="notice">
  {{ notice-1 | markdownify }}
</div>

![pch cpp 파일 설정](https://github.com/user-attachments/assets/a6d00b10-e3d4-42cf-b8dc-f71d56ecf578)


6) pch.cpp의 미리 컴파일된 헤더를 만들기로 변경한다.

7) Game 폴더에 Game 클래스를 추가한다.

#### Game.h
```cpp
#pragma once
class Game
{
public:
	void Init();
	void Update();
};
```

#### Game.cpp
```cpp
#include "pch.h"
#include "Game.h"

void Game::Init()
{
}

void Game::Update()
{
}
```

8) Client에 Game 객체 생성 후 메인 루프에 포함한다.

#### client.cpp
```cpp
 unique_ptr<Game> game = make_unique<Game>();
 game->Init();

 // 기본 메시지 루프입니다:
 while (true)
 {
     // 메세지가 있는지 없는지 계속 확인
     if (PeekMessage(&msg, nullptr, 0, 0, PM_REMOVE))
     {
         // 프로그램 종료
         if (msg.message == WM_QUIT)
             break;
     
         // 메세지 처리
         if (!TranslateAccelerator(msg.hwnd, hAccelTable, &msg))
         {
             TranslateMessage(&msg);
             DispatchMessage(&msg);
         }
     }

     game->Update();
 }
```

9) 프로젝트 실행 시 나오는 윈도우 창 메뉴를 없앤다.

#### client.cpp
```cpp
ATOM MyRegisterClass(HINSTANCE hInstance)
{
    WNDCLASSEXW wcex;

    wcex.cbSize = sizeof(WNDCLASSEX);

    wcex.style          = CS_HREDRAW | CS_VREDRAW;
    wcex.lpfnWndProc    = WndProc;
    wcex.cbClsExtra     = 0;
    wcex.cbWndExtra     = 0;
    wcex.hInstance      = hInstance;
    wcex.hIcon          = LoadIcon(hInstance, MAKEINTRESOURCE(IDI_CLIENT));
    wcex.hCursor        = LoadCursor(nullptr, IDC_ARROW);
    wcex.hbrBackground  = (HBRUSH)(COLOR_WINDOW+1);
	// 메뉴 없애기
    wcex.lpszMenuName = nullptr;
    wcex.lpszClassName  = szWindowClass;
    wcex.hIconSm        = LoadIcon(wcex.hInstance, MAKEINTRESOURCE(IDI_SMALL));

    return RegisterClassExW(&wcex);
}
```

![프로젝트 추가](https://github.com/user-attachments/assets/9ba046db-9ae6-4d4d-a757-2eb794c05e11)


10) DLL(동적 연결 라이브러리) 프로젝트를 추가한다. 이 프로젝트는 라이브러리 용이기 때문에 별도로 실행할 수 없다.

<div class="notice">
  {{ notice-2 | markdownify }}
</div>

![엔진 폴더 정리](https://github.com/user-attachments/assets/b17402e1-ff0a-49ca-afb2-45c13de0bfb1)


11) Engine 프로젝트에 Engine, Resource, Utils 폴더를 추가한 뒤, Utils 폴더에 pch 클래스만 옮긴 뒤 나머지 파일은 삭제한다. 그리고 소스 파일과 헤더 파일 폴더를 삭제한다.

12) Engine 프로젝트 Utils 폴더에 EnginePch 클래스를 추가한다.

#### EnginePch.h
```cpp
#pragma once
// include
#include <windows.h>
#include <tchar.h>
#include <memory>
#include <string>
#include <vector>
#include <array>
#include <list>
#include <map>

using namespace std;

// DirectX
#include "d3dx12.h"
#include <d3d12.h>
#include <wrl.h>
#include <d3dcompiler.h>
#include <dxgi.h>
#include <DirectXMath.h>
#include <DirectXPackedVector.h>
#include <DirectXColors.h>

using namespace DirectX;
using namespace DirectX::PackedVector;
using namespace Microsoft::WRL;

// lib
#pragma comment(lib, "d3d12")
#pragma comment(lib, "dxgi")
#pragma comment(lib, "dxguid")
#pragma comment(lib, "d3dcompiler")

// typedef
using int8 = __int8;
using int16 = __int16;
using int32 = __int32;
using int64 = __int64;
using uint8 = unsigned __int8;
using uint16 = unsigned __int16;
using uint32 = unsigned __int32;
using uint64 = unsigned __int64;
using Vec2 = XMFLOAT2;
using Vec3 = XMFLOAT3;
using Vec4 = XMFLOAT4;
using Matrix = XMMATRIX;

void HelloEngine();
```

#### EnginePch.cpp
```cpp
#include "pch.h"
#include "EnginePch.h"

void HelloEngine()
{

}
```

<div class="notice">
  {{ notice-3 | markdownify }}
</div>

13) Engine 프로젝트 Utils 폴더에 d3dx12 헤더를 추가한다.

![Output 폴더](https://github.com/user-attachments/assets/6247b9d8-2738-4dc2-ba27-1045e85b8648)
![엔진 경로 설정](https://github.com/user-attachments/assets/fe744e98-81e8-46ad-83da-14694fdfd265)


14) Output 폴더를 추가한 후 Engine 프로젝트의 출력 디렉터리 경로를 수정한다.

![리소스 폴더](https://github.com/user-attachments/assets/f023b490-cc33-48e6-9ee7-939424151074)


15) Resources 폴더를 추가한다.

![Client 포함 디렉터리](https://github.com/user-attachments/assets/be59d37b-c0f5-44df-8cfc-d23e4bed0d52)
![Client 라이브러리 디렉터리](https://github.com/user-attachments/assets/2d2ab2e9-e5b8-4d7c-b999-e3f6a88e7b18)
![클라이언트 추가 종속성](https://github.com/user-attachments/assets/021da8dc-5f3d-4667-b685-40d1b7b08d0e)


16) Client 프로젝트에서 Engine 라이브러리를 사용하기 위해 포함 디렉터리, 라이브러리 디렉터리, 추가 종속성 경로를 수정한다.

출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
