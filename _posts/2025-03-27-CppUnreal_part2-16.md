---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] Scale, Rotation, Translation 변환 행렬"
date: 2025-03-27
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - Vector & Matrix
---



{% capture notice-1 %}
* SRT: Scale(크기 변환) + Rotation(회전) + Translation(이동)
* 벡터의 변환은 Translation이 3차원 좌표계로 표현할 수 없기 때문에 4차원 좌표계인 동차 좌표계를 사용 (x, y, z, w)
* Scale은 원점이 기준일 때와 원점이 기준이 아닐 때에 따라 크기 변환의 결과가 달라짐
* Rotation은 원점이 기준일 때는 자전하고, 원점이 기준이 아닐 때는 공전함
* Scale과 Rotation은 원점인지, 원점이 아닌지의 영향을 받기 때문의 벡터의 변환 순서는 SRT를 지켜야 함
{% endcapture %}



#### 벡터의 변환

![동차 좌표계](https://github.com/user-attachments/assets/b3bbe55a-925b-4319-87c7-24357b6886b4)


<div class="notice">
  {{ notice-1 | markdownify }}
</div>

출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
