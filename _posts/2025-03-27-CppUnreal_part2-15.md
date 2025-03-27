---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] 행렬"
date: 2025-03-27
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - Vector & Matrix
---



{% capture notice-1 %}
#### Matrix

* 스칼라 값으로 이루어진 2차원 배열
* 벡터도 행의 크기가 1인 행렬로 볼 수 있으며, 따라서 행렬과 연산이 가능
* 벡터를 변환시킬 때 사용되며, 게임에서는 주로 4차원 행렬을 사용
{% endcapture %}

{% capture notice-2 %}
* 행렬의 덧셈과 뺄셈은 같은 크기의 행렬끼리만 가능
{% endcapture %}

{% capture notice-3 %}
* 행렬의 곱셈은 앞에 오는 행렬의 열과 뒤에 오는 행렬의 행의 크기가 같아야 가능함
* 결과물은 앞에 오는 행렬의 행과 뒤에 오는 행렬의 열의 크기로 만들어짐
* 교환 법칙은 성립하지 않으나, 결합 법칙은 성립함
{% endcapture %}

{% capture notice-4 %}
* 대각 요소가 전부 1이고, 나머지 요소들이 전부 0인 정방 행렬
* 대각 요소: 행렬의 대각선을 이루고 있는 요소
* 정방 행렬: 행과 열의 크기가 같은 행렬
* 어떤 행렬과 단위 행렬을 곱하면 무조건 어던 행렬과 같은 행렬이 나옴
{% endcapture %}

{% capture notice-5 %}
* 어떤 행렬과 그 행렬의 역행렬을 곱하면 단위 행렬이 됨
* 역행렬은 항상 존재하지 않으며, 역행렬이 존재하는 행렬을 가역행렬이라고 함
* 어떤 행렬과 그 행렬의 역행렬의 곱은 교환 법칙이 성립함
{% endcapture %}

{% capture notice-6 %}
* 행렬의 행과 열을 뒤집은 행렬
* 대각선을 기준으로 뒤집은 것과 같음
* 
{% endcapture %}

{% capture notice-7 %}
* 모든 행 벡터 혹은 열 벡터가 전부 직교하며 크기가 1인 행렬
* 직교 행렬과 그 행렬의 전치 행렬의 곱은 단위 행렬임
{% endcapture %}



<div class="notice">
  {{ notice-1 | markdownify }}
</div>

#### 행렬과 스칼라 값의 곱셈



#### 행렬의 덧셈, 뺄셈



<div class="notice">
  {{ notice-2 | markdownify }}
</div>

#### 행렬의 곱셈



<div class="notice">
  {{ notice-3 | markdownify }}
</div>

#### 단위 행렬



<div class="notice">
  {{ notice-4 | markdownify }}
</div>

#### 역행렬



<div class="notice">
  {{ notice-5 | markdownify }}
</div>

#### 전치 행렬



<div class="notice">
  {{ notice-6 | markdownify }}
</div>

#### 직교 행렬



<div class="notice">
  {{ notice-7 | markdownify }}
</div>

출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
