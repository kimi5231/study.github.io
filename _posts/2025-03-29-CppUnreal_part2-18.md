---
title: "[C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2] World, View 변환 행렬"
date: 2025-03-29
categories:
  - C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2
tags:
  - Vector & Matrix
---



{% capture notice-1 %}
#### 좌표계

## 모델 좌표계 -> 월드 좌표계 -> 카메라 좌표계 -> 투영 좌표계 -> 화면 좌표계

* 모델 좌표계: 모델마다 가지는 로컬 좌표계
* 월드 좌표계: 모델들이 한 곳에 모여있는 공간의 좌표계
* 카메라 좌표계: 카메라가 기준인 좌표계
* 투영 좌표계: 절두체 안에 있는 객체들만 보여주는 좌표계
* 화면 좌표계: 뷰포트 안에 있는 객체들만 보여주는 좌표계
{% endcapture %}

{% capture notice-2 %}
#### 변환

## 월드 변환 -> 카메라 변환 -> 투영 변환 -> 뷰포트 변환
{% endcapture %}

{% capture notice-3 %}
#### 월드 변환

* 변환하는 내용에 맞추어 SRT 변환 행렬을 사용하여 모델의 좌표를 월드 좌표계로 바꾸는 변환
* 모델마다 각각의 월드 좌표를 가지고 있기 때문에 월드 변환 행렬은 모델마다 다르게 가지고 있음
{% endcapture %}

{% capture notice-3 %}
#### 카메라 변환

* 카메라의 월드 변환 행렬의 역행렬을 사용하여 모델의 좌표를 카메라 좌표계로 바꾸는 변환 (카메라 좌표계 == 카메라의 로컬 좌표계)
* 카메라의 월드 변환 행렬은 카메라의 크기는 변화하지 않기 때문에 RT 변환 행렬로 이루어지며, 역행렬을 이용할 때에는 RT의 순서를 반대로 해야 함
* 이때, R 변환 행렬은 직교 행렬의 성질을 띄고 있기 때문에 R 변환 행렬의 역행렬은 R 변환 행렬의 전치 행렬과 같음
{% endcapture %}



<div class="notice">
  {{ notice-1 | markdownify }}
</div>

<div class="notice">
  {{ notice-2 | markdownify }}
</div>

<div class="notice">
  {{ notice-3 | markdownify }}
</div>

#### 월드 변환 행렬

![월드 변환 행렬](https://github.com/user-attachments/assets/7c1f159c-70de-49ab-bfd6-1161f916f2c9)


<div class="notice">
  {{ notice-4 | markdownify }}
</div>

#### 카메라 변환 행렬

![카메라 변환 행렬](https://github.com/user-attachments/assets/b19ce685-32a6-45aa-a953-99ad9ae3d875)


출처: [인프런: C++과 언리얼로 만드는 MMOPRG 게임 개발 시리즈 Part2][source]

[source]: https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard
