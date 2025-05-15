---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] Lighting #1"
date: 2025-05-12
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - Camera & Lighting
---



{% capture notice-1 %}
#### 유니티에서의 조명

* Directional Light: 태양과 같은 존재, 일정한 방향으로 동일한 빛을 보내는 조명
* Point Light: 한 지점을 기준으로 일정 거리만큼 밝히는 조명, 지점에서 멀어질수록 빛이 약해짐
* Spot Light: 일정 범위만 밝히는 조명
{% endcapture %}

{% capture notice-2 %}
#### 빛에 포함되는 속성

* Diffuse(난반사): 빛이 표면에 닿을 때 표면의 노멀 벡터와 빛이 이루는 cos값만큼 표면이 빛을 받음
* Ambient(환경): 기본적으로 있는 빛
* Specular(정반사): 반사된 빛이 카메라의 위치와 동일할 때 반짝임이 가장 크고, 동일하지 않을 때는 반사된 빛과 카메라의 위치가 이루는 각도에 따라 반짝임의 크기가 달라짐
{% endcapture %}



<div class="notice">
  {{ notice-1 | markdownify }}
</div>

<div class="notice">
  {{ notice-2 | markdownify }}
</div>

출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
