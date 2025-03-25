---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] 벡터"
date: 2025-03-24
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - Vector & Matrix
---



{% capture notice-1 %}
#### Vector

* 크기와 방향을 가지고 있음
* 목적지 좌표에서 출발지 좌표를 빼면 벡터의 성분을 구할 수 있음
{% endcapture %}

{% capture notice-2 %}
* 수식에서 앞에 위치한 벡터의 머리에 뒤에 위치한 벡터의 꼬리를 붙여서 계산
* 벡터의 각 성분끼리 덧셈, 뺄셈을 하여 계산
{% endcapture %}

{% capture notice-3 %}
* 벡터의 각 성분에 스칼라 값을 곱하거나 나누면 됨
* 교환법칙, 결합법칙 성립
{% endcapture %}

{% capture notice-4 %}
#### 벡터의 크기

* 벡터의 모든 좌표가 주어졌다고 가정했을 때 피타고라스의 정리를 활용하여 벡터의 크기를 구할 수 있음
* 3차원 벡터는 피타고라스의 정리를 두 번 사용하면 크기를 구할 수 있음
{% endcapture %}

{% capture notice-5 %}
#### 단위 벡터

* 크기가 1인 벡터
* 벡터이므로 방향 정보는 그대로 있고 크기가 1임
* 벡터의 각 성분을 벡터의 크기로 나누면 구할 수 있음
* 객체를 원하는 방향으로 이동시킬 때 사용할 수 있음
{% endcapture %}

{% capture notice-6 %}
* 두 벡터의 내적은 결과값로 스칼라 값이 나옴
* 벡터의 내적을 통해 각도의 정보를 알 수 있음
=> 결과값이 양수: 예각
=> 결과값이 음수: 둔각
=> 결과값이 0: 직각
* 교환 법칙이 성립
{% endcapture %}

{% capture notice-7 %}
* 두 벡터의 외적은 결과값으로 두 벡터 모두에게 수직인 벡터가 나옴
* 교환 법칙이 성립하지 않음, 순서가 중요함
* 법선 벡터(두 벡터 모두에게 수직인 벡터)를 구할 수 있음
* 곱하는 순서를 통해 결과값으로 나오는 벡터의 방향을 바꿀 수 있음
* 어떤 영역 안에 있는지 확인할 때와 쿨타임을 나타낼 때 주로 사용함
{% endcapture %}



<div class="notice">
  {{ notice-1 | markdownify }}
</div>

#### 벡터의 덧셈, 뺄셈



<div class="notice">
  {{ notice-2 | markdownify }}
</div>

#### 벡터와 스칼라 곱셈, 나눗셈



<div class="notice">
  {{ notice-3 | markdownify }}
</div>

<div class="notice">
  {{ notice-4 | markdownify }}
</div>

<div class="notice">
  {{ notice-5 | markdownify }}
</div>

#### 벡터의 내적



<div class="notice">
  {{ notice-6 | markdownify }}
</div>

#### 벡터의 외적



<div class="notice">
  {{ notice-7 | markdownify }}
</div>

출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
