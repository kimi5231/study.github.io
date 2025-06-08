---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] Tessellation"
date: 2025-06-06
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - Terrain
---



{% capture notice-1 %}
#### Tessellation

* 메쉬의 삼각형을 더 잘게 잘라서 새로운 정점을 추가하여 삼각형을 추가하는 개념
* Level Of Detail(LOD): 모델을 레벨에 따라 Low에서 High 폴리곤으로 설정하고, 거리에 따라 렌더링 하기 편하게 레벨을 바꿔가며 렌더링하는 것
* Control Point(CP): 제어점, CP를 기준으로 정점들이 생김
* Patch: CP Group
{% endcapture %}

{% capture notice-2 %}
* Vertex Shader: Tessellation을 하게 되면면 CP를 기준으로 정점을 추가할 것이기 때문에 Vertex Shader는 아무것도 하지 않고 정보를 넘기게 됨
* Hull Shader: 삼각형을 설정한 만큼 여러 개의 삼각형으로 나눔, 나누면서 생긴 정점 추가
* Domain Shader: 기존에 Vertex Shader에서 했던 것 처럼 변환, 정점의 위치를 CP를 기준으로 비례로 나타냄
{% endcapture %}



<div class="notice">
  {{ notice-1 | markdownify }}
</div>

<div class="notice">
  {{ notice-2 | markdownify }}
</div>

#### tessellation shader 파일 추가

#### tessellation.px
```px
#ifndef _TESSELLATION_FX_
#define _TESSELLATION_FX_

#include "params.fx"

// Vertex Shader

struct VS_IN
{
    float3 pos : POSITION;
    float2 uv : TEXCOORD;
};

struct VS_OUT
{
    float3 pos : POSITION;
    float2 uv : TEXCOORD;
};

VS_OUT VS_Main(VS_IN input)
{
    VS_OUT output = input;

    return output;
}

// Hull Shader

struct PatchTess
{
    float edgeTess[3] : SV_TessFactor;
    float insideTess : SV_InsideTessFactor;
};

struct HS_OUT
{
    float3 pos : POSITION;
    float2 uv : TEXCOORD;
};

// Constant HS
PatchTess ConstantHS(InputPatch<VS_OUT, 3> input, int patchID : SV_PrimitiveID)
{
    PatchTess output = (PatchTess) 0.f;

    output.edgeTess[0] = 1;
    output.edgeTess[1] = 2;
    output.edgeTess[2] = 3;
    output.insideTess = 1;

    return output;
}

// Control Point HS
[domain("tri")] // 패치의 종류 (tri, quad, isoline)
[partitioning("integer")] // subdivision mode (integer 소수점 무시, fractional_even, fractional_odd)
[outputtopology("triangle_cw")] // (triangle_cw, triangle_ccw, line)
[outputcontrolpoints(3)] // 하나의 입력 패치에 대해, HS가 출력할 제어점 개수
[patchconstantfunc("ConstantHS")] // ConstantHS 함수 이름
HS_OUT HS_Main(InputPatch<VS_OUT, 3> input, int vertexIdx : SV_OutputControlPointID, int patchID : SV_PrimitiveID)
{
    HS_OUT output = (HS_OUT) 0.f;

    output.pos = input[vertexIdx].pos;
    output.uv = input[vertexIdx].uv;

    return output;
}

// Domain Shader

struct DS_OUT
{
    float4 pos : SV_Position;
    float2 uv : TEXCOORD;
};

[domain("tri")]
DS_OUT DS_Main(const OutputPatch<HS_OUT, 3> input, float3 location : SV_DomainLocation, PatchTess patch)
{
    DS_OUT output = (DS_OUT) 0.f;

    float3 localPos = input[0].pos * location[0] + input[1].pos * location[1] + input[2].pos * location[2];
    float2 uv = input[0].uv * location[0] + input[1].uv * location[1] + input[2].uv * location[2];

    output.pos = mul(float4(localPos, 1.f), g_matWVP);
    output.uv = uv;

    return output;
}

// Pixel Shader

float4 PS_Main(DS_OUT input) : SV_Target
{
    return float4(1.f, 0.f, 0.f, 1.f);
}

#endif
```

#### Shader 헤더에 ShaderArg 구조체 추가 후 Shader CreateGraphicsShader 함수 매개변수 수정 후 코드 수정

#### Shader.h
```cpp
struct ShaderArg
{
	const string vs = "VS_Main";
	const string hs;
	const string ds;
	const string gs;
	const string ps = "PS_Main";
};

class Shader : public Object
{
public:
	void CreateGraphicsShader(const wstring& path, ShaderInfo info = ShaderInfo(), ShaderArg arg = ShaderArg());
}
```

#### Shader.cpp
```cpp
void Shader::CreateGraphicsShader(const wstring& path, ShaderInfo info, ShaderArg arg)
{
	_info = info;

	CreateVertexShader(path, arg.vs, "vs_5_0");
	CreatePixelShader(path, arg.ps, "ps_5_0");

	if (arg.hs.empty() == false)
		CreateHullShader(path, arg.hs, "hs_5_0");

	if (arg.ds.empty() == false)
		CreateDomainShader(path, arg.ds, "ds_5_0");

	if (arg.gs.empty() == false)
		CreateGeometryShader(path, arg.gs, "gs_5_0");

	D3D12_INPUT_ELEMENT_DESC desc[] =
	{
		// 1
		{ "POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
		{ "TEXCOORD", 0, DXGI_FORMAT_R32G32_FLOAT, 0, 12, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
		{ "NORMAL", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 20, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
		{ "TANGENT", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 32, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
	
		// 2
		{ "W", 0, DXGI_FORMAT_R32G32B32A32_FLOAT, 1, 0,  D3D12_INPUT_CLASSIFICATION_PER_INSTANCE_DATA, 1},
		{ "W", 1, DXGI_FORMAT_R32G32B32A32_FLOAT, 1, 16, D3D12_INPUT_CLASSIFICATION_PER_INSTANCE_DATA, 1},
		{ "W", 2, DXGI_FORMAT_R32G32B32A32_FLOAT, 1, 32, D3D12_INPUT_CLASSIFICATION_PER_INSTANCE_DATA, 1},
		{ "W", 3, DXGI_FORMAT_R32G32B32A32_FLOAT, 1, 48, D3D12_INPUT_CLASSIFICATION_PER_INSTANCE_DATA, 1},
		{ "WV", 0, DXGI_FORMAT_R32G32B32A32_FLOAT, 1, 64, D3D12_INPUT_CLASSIFICATION_PER_INSTANCE_DATA, 1},
		{ "WV", 1, DXGI_FORMAT_R32G32B32A32_FLOAT, 1, 80, D3D12_INPUT_CLASSIFICATION_PER_INSTANCE_DATA, 1},
		{ "WV", 2, DXGI_FORMAT_R32G32B32A32_FLOAT, 1, 96, D3D12_INPUT_CLASSIFICATION_PER_INSTANCE_DATA, 1},
		{ "WV", 3, DXGI_FORMAT_R32G32B32A32_FLOAT, 1, 112, D3D12_INPUT_CLASSIFICATION_PER_INSTANCE_DATA, 1},
		{ "WVP", 0, DXGI_FORMAT_R32G32B32A32_FLOAT, 1, 128, D3D12_INPUT_CLASSIFICATION_PER_INSTANCE_DATA, 1},
		{ "WVP", 1, DXGI_FORMAT_R32G32B32A32_FLOAT, 1, 144, D3D12_INPUT_CLASSIFICATION_PER_INSTANCE_DATA, 1},
		{ "WVP", 2, DXGI_FORMAT_R32G32B32A32_FLOAT, 1, 160, D3D12_INPUT_CLASSIFICATION_PER_INSTANCE_DATA, 1},
		{ "WVP", 3, DXGI_FORMAT_R32G32B32A32_FLOAT, 1, 176, D3D12_INPUT_CLASSIFICATION_PER_INSTANCE_DATA, 1},
	};

	_graphicsPipelineDesc.InputLayout = { desc, _countof(desc) };
	_graphicsPipelineDesc.pRootSignature = GRAPHICS_ROOT_SIGNATURE.Get();

	_graphicsPipelineDesc.RasterizerState = CD3DX12_RASTERIZER_DESC(D3D12_DEFAULT);
	_graphicsPipelineDesc.BlendState = CD3DX12_BLEND_DESC(D3D12_DEFAULT);
	_graphicsPipelineDesc.DepthStencilState = CD3DX12_DEPTH_STENCIL_DESC(D3D12_DEFAULT);
	_graphicsPipelineDesc.SampleMask = UINT_MAX;
	_graphicsPipelineDesc.PrimitiveTopologyType = GetTopologyType(info.topology);
	_graphicsPipelineDesc.NumRenderTargets = 1;
	_graphicsPipelineDesc.RTVFormats[0] = DXGI_FORMAT_R8G8B8A8_UNORM;
	_graphicsPipelineDesc.SampleDesc.Count = 1;
	_graphicsPipelineDesc.DSVFormat = DXGI_FORMAT_D32_FLOAT;

	switch (info.shaderType)
	{
	case SHADER_TYPE::DEFERRED:
		_graphicsPipelineDesc.NumRenderTargets = RENDER_TARGET_G_BUFFER_GROUP_MEMBER_COUNT;
		_graphicsPipelineDesc.RTVFormats[0] = DXGI_FORMAT_R32G32B32A32_FLOAT; // POSITION
		_graphicsPipelineDesc.RTVFormats[1] = DXGI_FORMAT_R32G32B32A32_FLOAT; // NORMAL
		_graphicsPipelineDesc.RTVFormats[2] = DXGI_FORMAT_R8G8B8A8_UNORM; // COLOR
		break;
	case SHADER_TYPE::FORWARD:
		_graphicsPipelineDesc.NumRenderTargets = 1;
		_graphicsPipelineDesc.RTVFormats[0] = DXGI_FORMAT_R8G8B8A8_UNORM;
		break;
	case SHADER_TYPE::LIGHTING:
		_graphicsPipelineDesc.NumRenderTargets = 2;
		_graphicsPipelineDesc.RTVFormats[0] = DXGI_FORMAT_R8G8B8A8_UNORM;
		_graphicsPipelineDesc.RTVFormats[1] = DXGI_FORMAT_R8G8B8A8_UNORM;
		break;
	case SHADER_TYPE::PARTICLE:
		_graphicsPipelineDesc.NumRenderTargets = 1;
		_graphicsPipelineDesc.RTVFormats[0] = DXGI_FORMAT_R8G8B8A8_UNORM;
		break;
	case SHADER_TYPE::SHADOW:
		_graphicsPipelineDesc.NumRenderTargets = 1;
		_graphicsPipelineDesc.RTVFormats[0] = DXGI_FORMAT_R32_FLOAT;
		break;
	}

	switch (info.rasterizerType)
	{
	case RASTERIZER_TYPE::CULL_BACK:
		_graphicsPipelineDesc.RasterizerState.FillMode = D3D12_FILL_MODE_SOLID;
		_graphicsPipelineDesc.RasterizerState.CullMode = D3D12_CULL_MODE_BACK;
		break;
	case RASTERIZER_TYPE::CULL_FRONT:
		_graphicsPipelineDesc.RasterizerState.FillMode = D3D12_FILL_MODE_SOLID;
		_graphicsPipelineDesc.RasterizerState.CullMode = D3D12_CULL_MODE_FRONT;
		break;
	case RASTERIZER_TYPE::CULL_NONE:
		_graphicsPipelineDesc.RasterizerState.FillMode = D3D12_FILL_MODE_SOLID;
		_graphicsPipelineDesc.RasterizerState.CullMode = D3D12_CULL_MODE_NONE;
		break;
	case RASTERIZER_TYPE::WIREFRAME:
		_graphicsPipelineDesc.RasterizerState.FillMode = D3D12_FILL_MODE_WIREFRAME;
		_graphicsPipelineDesc.RasterizerState.CullMode = D3D12_CULL_MODE_NONE;
		break;
	}

	switch (info.depthStencilType)
	{
	case DEPTH_STENCIL_TYPE::LESS:
		_graphicsPipelineDesc.DepthStencilState.DepthEnable = TRUE;
		_graphicsPipelineDesc.DepthStencilState.DepthFunc = D3D12_COMPARISON_FUNC_LESS;
		break;
	case DEPTH_STENCIL_TYPE::LESS_EQUAL:
		_graphicsPipelineDesc.DepthStencilState.DepthEnable = TRUE;
		_graphicsPipelineDesc.DepthStencilState.DepthFunc = D3D12_COMPARISON_FUNC_LESS_EQUAL;
		break;
	case DEPTH_STENCIL_TYPE::GREATER:
		_graphicsPipelineDesc.DepthStencilState.DepthEnable = TRUE;
		_graphicsPipelineDesc.DepthStencilState.DepthFunc = D3D12_COMPARISON_FUNC_GREATER;
		break;
	case DEPTH_STENCIL_TYPE::GREATER_EQUAL:
		_graphicsPipelineDesc.DepthStencilState.DepthEnable = TRUE;
		_graphicsPipelineDesc.DepthStencilState.DepthFunc = D3D12_COMPARISON_FUNC_GREATER_EQUAL;
		break;
	case DEPTH_STENCIL_TYPE::NO_DEPTH_TEST:
		_graphicsPipelineDesc.DepthStencilState.DepthEnable = FALSE;
		_graphicsPipelineDesc.DepthStencilState.DepthFunc = D3D12_COMPARISON_FUNC_LESS;
		break;
	case DEPTH_STENCIL_TYPE::NO_DEPTH_TEST_NO_WRITE:
		_graphicsPipelineDesc.DepthStencilState.DepthEnable = FALSE;
		_graphicsPipelineDesc.DepthStencilState.DepthWriteMask = D3D12_DEPTH_WRITE_MASK_ZERO;
		break;
	case DEPTH_STENCIL_TYPE::LESS_NO_WRITE:
		_graphicsPipelineDesc.DepthStencilState.DepthEnable = TRUE;
		_graphicsPipelineDesc.DepthStencilState.DepthFunc = D3D12_COMPARISON_FUNC_LESS;
		_graphicsPipelineDesc.DepthStencilState.DepthWriteMask = D3D12_DEPTH_WRITE_MASK_ZERO;
		break;
	}

	D3D12_RENDER_TARGET_BLEND_DESC& rt = _graphicsPipelineDesc.BlendState.RenderTarget[0];

	switch (info.blendType)
	{
	case BLEND_TYPE::DEFAULT:
		rt.BlendEnable = FALSE;
		rt.LogicOpEnable = FALSE;
		rt.SrcBlend = D3D12_BLEND_ONE;
		rt.DestBlend = D3D12_BLEND_ZERO;
		break;
	case BLEND_TYPE::ALPHA_BLEND:
		rt.BlendEnable = TRUE;
		rt.LogicOpEnable = FALSE;
		rt.SrcBlend = D3D12_BLEND_SRC_ALPHA;
		rt.DestBlend = D3D12_BLEND_INV_SRC_ALPHA;
		break;
	case BLEND_TYPE::ONE_TO_ONE_BLEND:
		rt.BlendEnable = TRUE;
		rt.LogicOpEnable = FALSE;
		rt.SrcBlend = D3D12_BLEND_ONE;
		rt.DestBlend = D3D12_BLEND_ONE;
		break;
	}

	DEVICE->CreateGraphicsPipelineState(&_graphicsPipelineDesc, IID_PPV_ARGS(&_pipelineState));
}
```

#### Shader 클래스에 CreateHullShader, CreateDomainShader 함수 추가 및 hsBlob, dsBlob 멤버 변수 추가

#### Shader.h
```cpp
class Shader : public Object
{
private:
	void CreateHullShader(const wstring& path, const string& name, const string& version);
	void CreateDomainShader(const wstring& path, const string& name, const string& version);

private:
	ComPtr<ID3DBlob>					_hsBlob;
	ComPtr<ID3DBlob>					_dsBlob;
}
```

#### Shader.cpp
```cpp
void Shader::CreateHullShader(const wstring& path, const string& name, const string& version)
{
	CreateShader(path, name, version, _hsBlob, _graphicsPipelineDesc.HS);
}

void Shader::CreateDomainShader(const wstring& path, const string& name, const string& version)
{
	CreateShader(path, name, version, _dsBlob, _graphicsPipelineDesc.DS);
}
```

#### Resources CreateDefaultShader 함수에서 ARG 사용하는 shader들에 코드 추가 및 Tessellation 추가, CreateDefaultMaterial 함수에도 Tessellation 추가

#### Resources.cpp
```cpp
void Resources::CreateDefaultShader()
{
	// Skybox
	{
		ShaderInfo info =
		{
			SHADER_TYPE::FORWARD,
			RASTERIZER_TYPE::CULL_NONE,
			DEPTH_STENCIL_TYPE::LESS_EQUAL
		};

		shared_ptr<Shader> shader = make_shared<Shader>();
		shader->CreateGraphicsShader(L"..\\Resources\\Shader\\skybox.fx", info);
		Add<Shader>(L"Skybox", shader);
	}

	// Deferred
	{
		ShaderInfo info =
		{
			SHADER_TYPE::DEFERRED
		};

		shared_ptr<Shader> shader = make_shared<Shader>();
		shader->CreateGraphicsShader(L"..\\Resources\\Shader\\deferred.fx", info);
		Add<Shader>(L"Deferred", shader);
	}

	// Forward
	{
		ShaderInfo info =
		{
			SHADER_TYPE::FORWARD,
		};

		shared_ptr<Shader> shader = make_shared<Shader>();
		shader->CreateGraphicsShader(L"..\\Resources\\Shader\\forward.fx", info);
		Add<Shader>(L"Forward", shader);
	}

	// Texture
	{
		ShaderInfo info =
		{
			SHADER_TYPE::FORWARD,
			RASTERIZER_TYPE::CULL_NONE,
			DEPTH_STENCIL_TYPE::NO_DEPTH_TEST_NO_WRITE
		};

		ShaderArg arg =
		{
			"VS_Tex",
			"",
			"",
			"",
			"PS_Tex"
		};

		shared_ptr<Shader> shader = make_shared<Shader>();
		shader->CreateGraphicsShader(L"..\\Resources\\Shader\\forward.fx", info, arg);
		Add<Shader>(L"Texture", shader);
	}

	// DirLight
	{
		ShaderInfo info =
		{
			SHADER_TYPE::LIGHTING,
			RASTERIZER_TYPE::CULL_NONE,
			DEPTH_STENCIL_TYPE::NO_DEPTH_TEST_NO_WRITE,
			BLEND_TYPE::ONE_TO_ONE_BLEND
		};

		ShaderArg arg =
		{
			"VS_DirLight",
			"",
			"",
			"",
			"PS_DirLight"
		};

		shared_ptr<Shader> shader = make_shared<Shader>();
		shader->CreateGraphicsShader(L"..\\Resources\\Shader\\lighting.fx", info, arg);
		Add<Shader>(L"DirLight", shader);
	}

	// PointLight
	{
		ShaderInfo info =
		{
			SHADER_TYPE::LIGHTING,
			RASTERIZER_TYPE::CULL_NONE,
			DEPTH_STENCIL_TYPE::NO_DEPTH_TEST_NO_WRITE,
			BLEND_TYPE::ONE_TO_ONE_BLEND
		};

		ShaderArg arg =
		{
			"VS_PointLight",
			"",
			"",
			"",
			"PS_PointLight"
		};

		shared_ptr<Shader> shader = make_shared<Shader>();
		shader->CreateGraphicsShader(L"..\\Resources\\Shader\\lighting.fx", info, arg);
		Add<Shader>(L"PointLight", shader);
	}

	// Final
	{
		ShaderInfo info =
		{
			SHADER_TYPE::LIGHTING,
			RASTERIZER_TYPE::CULL_BACK,
			DEPTH_STENCIL_TYPE::NO_DEPTH_TEST_NO_WRITE,
		};

		ShaderArg arg =
		{
			"VS_Final",
			"",
			"",
			"",
			"PS_Final"
		};

		shared_ptr<Shader> shader = make_shared<Shader>();
		shader->CreateGraphicsShader(L"..\\Resources\\Shader\\lighting.fx", info, arg);
		Add<Shader>(L"Final", shader);
	}

	// Compute
	{
		shared_ptr<Shader> shader = make_shared<Shader>();
		shader->CreateComputeShader(L"..\\Resources\\Shader\\compute.fx", "CS_Main", "cs_5_0");
		Add<Shader>(L"ComputeShader", shader);
	}

	// Particle
	{
		ShaderInfo info =
		{
			SHADER_TYPE::PARTICLE,
			RASTERIZER_TYPE::CULL_BACK,
			DEPTH_STENCIL_TYPE::LESS_NO_WRITE,
			BLEND_TYPE::ALPHA_BLEND,
			D3D_PRIMITIVE_TOPOLOGY_POINTLIST
		};

		ShaderArg arg =
		{
			"VS_Main",
			"",
			"",
			"GS_Main",
			"PS_Main"
		};

		shared_ptr<Shader> shader = make_shared<Shader>();
		shader->CreateGraphicsShader(L"..\\Resources\\Shader\\particle.fx", info, arg);
		Add<Shader>(L"Particle", shader);
	}

	// ComputeParticle
	{
		shared_ptr<Shader> shader = make_shared<Shader>();
		shader->CreateComputeShader(L"..\\Resources\\Shader\\particle.fx", "CS_Main", "cs_5_0");
		Add<Shader>(L"ComputeParticle", shader);
	}

	// Shadow
	{
		ShaderInfo info =
		{
			SHADER_TYPE::SHADOW,
			RASTERIZER_TYPE::CULL_BACK,
			DEPTH_STENCIL_TYPE::LESS,
		};

		shared_ptr<Shader> shader = make_shared<Shader>();
		shader->CreateGraphicsShader(L"..\\Resources\\Shader\\shadow.fx", info);
		Add<Shader>(L"Shadow", shader);
	}

	// Tessellation
	{
		ShaderInfo info =
		{
			SHADER_TYPE::FORWARD,
			RASTERIZER_TYPE::WIREFRAME,
			DEPTH_STENCIL_TYPE::LESS,
			BLEND_TYPE::DEFAULT,
			D3D_PRIMITIVE_TOPOLOGY_3_CONTROL_POINT_PATCHLIST
		};

		ShaderArg arg =
		{
			"VS_Main",
			"HS_Main",
			"DS_Main",
			"",
			"PS_Main",
		};

		shared_ptr<Shader> shader = make_shared<Shader>();
		shader->CreateGraphicsShader(L"..\\Resources\\Shader\\tessellation.fx", info, arg);
		Add<Shader>(L"Tessellation", shader);
	}
}

void Resources::CreateDefaultMaterial()
{
	// Skybox
	{
		shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"Skybox");
		shared_ptr<Material> material = make_shared<Material>();
		material->SetShader(shader);
		Add<Material>(L"Skybox", material);
	}

	// DirLight
	{
		shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"DirLight");
		shared_ptr<Material> material = make_shared<Material>();
		material->SetShader(shader);
		material->SetTexture(0, GET_SINGLE(Resources)->Get<Texture>(L"PositionTarget"));
		material->SetTexture(1, GET_SINGLE(Resources)->Get<Texture>(L"NormalTarget"));
		Add<Material>(L"DirLight", material);
	}

	// PointLight
	{
		const WindowInfo& window = GEngine->GetWindow();
		Vec2 resolution = { static_cast<float>(window.width), static_cast<float>(window.height) };

		shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"PointLight");
		shared_ptr<Material> material = make_shared<Material>();
		material->SetShader(shader);
		material->SetTexture(0, GET_SINGLE(Resources)->Get<Texture>(L"PositionTarget"));
		material->SetTexture(1, GET_SINGLE(Resources)->Get<Texture>(L"NormalTarget"));
		material->SetVec2(0, resolution);
		Add<Material>(L"PointLight", material);
	}

	// Final
	{
		shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"Final");
		shared_ptr<Material> material = make_shared<Material>();
		material->SetShader(shader);
		material->SetTexture(0, GET_SINGLE(Resources)->Get<Texture>(L"DiffuseTarget"));
		material->SetTexture(1, GET_SINGLE(Resources)->Get<Texture>(L"DiffuseLightTarget"));
		material->SetTexture(2, GET_SINGLE(Resources)->Get<Texture>(L"SpecularLightTarget"));
		Add<Material>(L"Final", material);
	}

	// Compute
	{
		shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"ComputeShader");
		shared_ptr<Material> material = make_shared<Material>();
		material->SetShader(shader);
		Add<Material>(L"ComputeShader", material);
	}

	// Particle
	{
		shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"Particle");
		shared_ptr<Material> material = make_shared<Material>();
		material->SetShader(shader);
		Add<Material>(L"Particle", material);
	}

	// ComputeParticle
	{
		shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"ComputeParticle");
		shared_ptr<Material> material = make_shared<Material>();
		material->SetShader(shader);

		Add<Material>(L"ComputeParticle", material);
	}

	// Instancing
	{
		shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"Deferred");
		shared_ptr<Texture> texture = GET_SINGLE(Resources)->Load<Texture>(L"Leather", L"..\\Resources\\Texture\\Leather.jpg");
		shared_ptr<Texture> texture2 = GET_SINGLE(Resources)->Load<Texture>(L"Leather_Normal", L"..\\Resources\\Texture\\Leather_Normal.jpg");
		shared_ptr<Material> material = make_shared<Material>();
		material->SetShader(shader);
		material->SetTexture(0, texture);
		material->SetTexture(1, texture2);
		Add<Material>(L"GameObject", material);
	}

	// Shadow
	{
		shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"Shadow");
		shared_ptr<Material> material = make_shared<Material>();
		material->SetShader(shader);
		Add<Material>(L"Shadow", material);
	}

	// Tessellation
	{
		shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"Tessellation");
		shared_ptr<Material> material = make_shared<Material>();
		material->SetShader(shader);
		Add<Material>(L"Tessellation", material);
	}
}
```

#### SceneManager LoadTestScene 함수에 Tessellation Test 추가 및 Object 주석 처리

#### SceneManager.cpp
```cpp
shared_ptr<Scene> SceneManager::LoadTestScene()
{
#pragma region LayerMask
	SetLayerName(0, L"Default");
	SetLayerName(1, L"UI");
#pragma endregion

#pragma region ComputeShader
	{
		shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"ComputeShader");

		// UAV 용 Texture 생성
		shared_ptr<Texture> texture = GET_SINGLE(Resources)->CreateTexture(L"UAVTexture",
			DXGI_FORMAT_R8G8B8A8_UNORM, 1024, 1024,
			CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_DEFAULT), D3D12_HEAP_FLAG_NONE,
			D3D12_RESOURCE_FLAG_ALLOW_UNORDERED_ACCESS);

		shared_ptr<Material> material = GET_SINGLE(Resources)->Get<Material>(L"ComputeShader");
		material->SetShader(shader);
		material->SetInt(0, 1);
		GEngine->GetComputeDescHeap()->SetUAV(texture->GetUAVHandle(), UAV_REGISTER::u0);

		// 쓰레드 그룹 (1 * 1024 * 1)
		material->Dispatch(1, 1024, 1);
	}
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
	camera->GetCamera()->SetCullingMaskLayerOnOff(layerIndex, true); // UI는 안 찍음
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

#pragma region Object
	/*{
		shared_ptr<GameObject> obj = make_shared<GameObject>();
		obj->AddComponent(make_shared<Transform>());
		obj->GetTransform()->SetLocalScale(Vec3(100.f, 100.f, 100.f));
		obj->GetTransform()->SetLocalPosition(Vec3(0, 0.f, 500.f));
		obj->SetStatic(false);
		shared_ptr<MeshRenderer> meshRenderer = make_shared<MeshRenderer>();
		{
			shared_ptr<Mesh> sphereMesh = GET_SINGLE(Resources)->LoadSphereMesh();
			meshRenderer->SetMesh(sphereMesh);
		}
		{
			shared_ptr<Material> material = GET_SINGLE(Resources)->Get<Material>(L"GameObject");
			meshRenderer->SetMaterial(material->Clone());
		}
		obj->AddComponent(meshRenderer);
		scene->AddGameObject(obj);
	}*/
#pragma endregion

#pragma region Plane
	{
		shared_ptr<GameObject> obj = make_shared<GameObject>();
		obj->AddComponent(make_shared<Transform>());
		obj->GetTransform()->SetLocalScale(Vec3(1000.f, 1.f, 1000.f));
		obj->GetTransform()->SetLocalPosition(Vec3(0.f, -100.f, 500.f));
		obj->SetStatic(true);
		shared_ptr<MeshRenderer> meshRenderer = make_shared<MeshRenderer>();
		{
			shared_ptr<Mesh> mesh = GET_SINGLE(Resources)->LoadCubeMesh();
			meshRenderer->SetMesh(mesh);
		}
		{
			shared_ptr<Material> material = GET_SINGLE(Resources)->Get<Material>(L"GameObject")->Clone();
			material->SetInt(0, 0);
			meshRenderer->SetMaterial(material);
		}
		obj->AddComponent(meshRenderer);
		scene->AddGameObject(obj);
	}
#pragma endregion

#pragma region UI_Test
	for (int32 i = 0; i < 6; i++)
	{
		shared_ptr<GameObject> sphere = make_shared<GameObject>();
		sphere->SetLayerIndex(GET_SINGLE(SceneManager)->LayerNameToIndex(L"UI")); // UI
		sphere->AddComponent(make_shared<Transform>());
		sphere->GetTransform()->SetLocalScale(Vec3(100.f, 100.f, 100.f));
		sphere->GetTransform()->SetLocalPosition(Vec3(-350.f + (i * 120), 250.f, 500.f));
		shared_ptr<MeshRenderer> meshRenderer = make_shared<MeshRenderer>();
		{
			shared_ptr<Mesh> mesh = GET_SINGLE(Resources)->LoadRectangleMesh();
			meshRenderer->SetMesh(mesh);
		}
		{
			shared_ptr<Shader> shader = GET_SINGLE(Resources)->Get<Shader>(L"Texture");

			shared_ptr<Texture> texture;
			if (i < 3)
				texture = GEngine->GetRTGroup(RENDER_TARGET_GROUP_TYPE::G_BUFFER)->GetRTTexture(i);
			else if(i < 5)
				texture = GEngine->GetRTGroup(RENDER_TARGET_GROUP_TYPE::LIGHTING)->GetRTTexture(i - 3);
			else
				texture = GEngine->GetRTGroup(RENDER_TARGET_GROUP_TYPE::SHADOW)->GetRTTexture(0);

			shared_ptr<Material> material = make_shared<Material>();
			material->SetShader(shader);
			material->SetTexture(0, texture);
			meshRenderer->SetMaterial(material);
		}
		sphere->AddComponent(meshRenderer);
		scene->AddGameObject(sphere);
	}
#pragma endregion

#pragma region Directional Light
	{
		shared_ptr<GameObject> light = make_shared<GameObject>();
		light->AddComponent(make_shared<Transform>());
		light->GetTransform()->SetLocalPosition(Vec3(0, 1000, 500));
		light->AddComponent(make_shared<Light>());
		light->GetLight()->SetLightDirection(Vec3(0, -1, 0.f));
		light->GetLight()->SetLightType(LIGHT_TYPE::DIRECTIONAL_LIGHT);
		light->GetLight()->SetDiffuse(Vec3(1.f, 1.f, 1.f));
		light->GetLight()->SetAmbient(Vec3(0.1f, 0.1f, 0.1f));
		light->GetLight()->SetSpecular(Vec3(0.1f, 0.1f, 0.1f));

		scene->AddGameObject(light);
	}
#pragma endregion

#pragma region Tessellation Test
	{
		shared_ptr<GameObject> gameObject = make_shared<GameObject>();
		gameObject->AddComponent(make_shared<Transform>());
		gameObject->GetTransform()->SetLocalPosition(Vec3(0, 0, 300));
		gameObject->GetTransform()->SetLocalScale(Vec3(100, 100, 100));
		gameObject->GetTransform()->SetLocalRotation(Vec3(0, 0, 0));

		shared_ptr<MeshRenderer> meshRenderer = make_shared<MeshRenderer>();
		{
			shared_ptr<Mesh> mesh = GET_SINGLE(Resources)->LoadRectangleMesh();
			meshRenderer->SetMesh(mesh);
			meshRenderer->SetMaterial(GET_SINGLE(Resources)->Get<Material>(L"Tessellation"));
		}
		gameObject->AddComponent(meshRenderer);

		scene->AddGameObject(gameObject);
	}
#pragma endregion

	return scene;
}
```

출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
