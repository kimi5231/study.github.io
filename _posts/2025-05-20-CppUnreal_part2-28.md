---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] Quaternion #1"
date: 2025-05-20
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - Quaternion
---



{% capture notice-1 %}
* Quaternion(사원수): 4개의 실수 값을 3대의 복소수로 표현한 형태
* 게임에서 회전을 할 때, 기존의 회전 방식은 오일러 각을 사용하는데, 이 방식은 여러 축의 회전이 결합될 때 Gimbal Lock 현상이 발생함
* Gimbal Lock: 세 축 중 하나의 축이 먹통이 되는 현상
* 2D 회전은 복소수로 표현할 수 있으며, 3D 회전은 사원수로 표현이 가능함
* Quaternion으로 표현한 회전은 기존의 오일러 각의 문제점인 Gimbal Lock 현상을 해결할 수 있음
{% endcapture %}



<div class="notice">
  {{ notice-1 | markdownify }}
</div>

출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
