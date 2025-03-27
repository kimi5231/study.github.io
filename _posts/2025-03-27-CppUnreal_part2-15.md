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

<img width="219" alt="행렬의 스칼라 곱" src="https://github.com/user-attachments/assets/5c632fb0-da0c-4084-b639-e19e067291c9" />


#### 행렬의 덧셈, 뺄셈

![행렬의 덧셈 뺄셈](https://github.com/user-attachments/assets/6ddaae08-aa2b-4004-b442-f822615054bc)


<div class="notice">
  {{ notice-2 | markdownify }}
</div>

#### 행렬의 곱셈

<img width="714" alt="행렬의 곱셈" src="https://github.com/user-attachments/assets/9a0449b6-9c9f-40cb-aae6-02a9ef41cb2c" />


<div class="notice">
  {{ notice-3 | markdownify }}
</div>

#### 단위 행렬

<img width="118" alt="단위 행렬" src="https://github.com/user-attachments/assets/3332907f-c8ba-4331-8cd7-848340b672ad" />
![단위 행렬 공식](https://github.com/user-attachments/assets/4244f402-fb0c-4a85-9a6e-a1e4763d88ce)


<div class="notice">
  {{ notice-4 | markdownify }}
</div>

#### 역행렬

<img width="345" alt="역행렬 공식" src="https://github.com/user-attachments/assets/ed52a2de-bcdc-46da-85d5-2ca729af29c9" />
![역행렬 존재](https://github.com/user-attachments/assets/37140ddc-9c64-4ce6-b460-a36298144ec8)
![역행렬 교환 법칙](https://github.com/user-attachments/assets/26ce9d12-fb54-478a-b93b-154cadbbe93d)


<div class="notice">
  {{ notice-5 | markdownify }}
</div>

#### 전치 행렬

![전치 행렬](https://github.com/user-attachments/assets/14240652-c5c5-4228-96e7-7701e101a8a6)
![전치 행렬 연산](https://github.com/user-attachments/assets/e5b18834-a472-40fe-9753-a4af4ebdd1ac)


<div class="notice">
  {{ notice-6 | markdownify }}
</div>

#### 직교 행렬

![직교 행렬 연산](https://github.com/user-attachments/assets/84da7828-9ff9-4137-9805-40fd463d8e4d)


<div class="notice">
  {{ notice-7 | markdownify }}
</div>

출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
