---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] 삼각함수"
date: 2025-03-24
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - Vector & Matrix
---



{% capture notice-1 %}
* 두 점 사이의 거리는 피타고라스의 정리를 이용하여 구할 수 있음
* 빗변을 두 점 사이의 거리라고 했을 때 두 점의 좌표를 피타고라스의 정리에 대입하여 구할 수 있음
{% endcapture %}

{% capture notice-2 %}
* 반지름의 길이가 1인 단위원으로도 정의할 수 있음
* 1 radian(약 57°): 호의 길이가 1이 되는 각도
* cos은 우함수(짝함수)이고, sin은 기함수(홀함수)
* 삼각함수는 2π를 주기로 그래프가 겹침, 2π 생략 가능
{% endcapture %}

{% capture notice-3 %}
* 비율에 따른 각도를 구하는 함수
{% endcapture %}

![피타고라스의 정리](https://github.com/user-attachments/assets/c7874b76-61df-461a-9d22-df399fd5d5fa)


<div class="notice">
  {{ notice-1 | markdownify }}
</div>

#### 삼각함수

![삼각함수](https://github.com/user-attachments/assets/f215cf5b-78a6-43dd-856b-cad8dbc453ef)


<div class="notice">
  {{ notice-2 | markdownify }}
</div>

#### 역삼각함수

![역삼각함수](https://github.com/user-attachments/assets/949d7177-78ec-47e0-b8a1-f9d64c18ed93)


<div class="notice">
  {{ notice-3 | markdownify }}
</div>

#### 코사인 법칙

![코사인 법칙](https://github.com/user-attachments/assets/d5bd237d-afe9-446d-9c5b-4de09640c9ca)


#### 삼각함수의 덧셈 정리

![덧셈 정리](https://github.com/user-attachments/assets/8563fc3f-2229-4ad5-8f48-284715ca9872)


출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
