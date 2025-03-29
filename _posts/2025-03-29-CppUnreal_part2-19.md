---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] Projection, Screen 변환 행렬"
date: 2025-03-29
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - Vector & Matrix
---



{% capture notice-1 %}
* Eye(카메라가 찍는 위치), Focus(카메라가 주시하는 위치), Up 벡터를 이용해 Up, Right, Look 벡터를 구할 수 있으며, 이를 통해 카메라 변환 행렬을 구할 수 있음
* 각 변환의 단계들은 변환 행렬의 역행렬을 통해 이전 단계로 되돌아갈 수 있음
{% endcapture %}

{% capture notice-2 %}
* 투영 변환에는 여러 가지 종류가 있으며, 그 중 원근 투영 변환은 깊이값을 이용해 원근감을 표현할 수 있음
{% endcapture %}

{% capture notice-3 %}
* 뷰포트: 렌더링을 할 사각 영역
* 뷰포트에 깊이값도 표현할 수 있음
{% endcapture %}



<div class="notice">
  {{ notice-1 | markdownify }}
</div>

<div class="notice">
  {{ notice-2 | markdownify }}
</div>

#### 원근 투영 변환 행렬



<div class="notice">
  {{ notice-3 | markdownify }}
</div>

#### 뷰포트 변환 행렬



출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
