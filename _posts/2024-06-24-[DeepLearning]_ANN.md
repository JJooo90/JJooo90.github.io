---
title:  "[DeepLearning] ANN 정리 "
excerpt: "ANN 정리"

categories:
  - ANN, DeepLeanring
tags:
  - [ANN, DeepLeanring]

toc: true
toc_sticky: true
 
date: 2024-06-24
last_modified_at: 2024-06-24
---

## 1. ANN (Artificial Neural Network)
  1. 인공지능(Artificial Intelligence)는 인간의 지능이 갖고 있는 기능을 갖춘 컴퓨터 시스템을 뜻하며, 인간의 지능을 기계 등에 인공적으로 구현한 것
  2. 머신러닝(Machine Learning) 혹은 기계학습은 인공지능의 한 분야로, 컴퓨터가 학습할 수 있도록 하는 알고리즘과 기술을 개발하는 분야
  3. 딥러닝(Deep Learning)은 여러 비선형 변환기법의 조합을 통해 높은 수준의 추상화(다량의 복잡한 자료들에서 핵심적인 내용만 추려내는 작업)을 시도하는 기계학습 알고리즘의 집합

  > 즉, 딥러닝 ⊂ 머신러닝 ⊂ 인공지능 

  > \# 최종 정리 : 사람의 신경망 원리와 구조를 모방하여 만든 기계학습 알고리즘  
    - 신호,자극 : input data  
    - 임계값 : weight(가중치)  
    - 행동 : output  
      -> 신호, 자극 등을 받고 임계값을 넘어서면 결과 신호를 전달하여 행동함  

    # 문제점  
    1. 학습 과정에서 최적의 파라미터 값을 찾기 어려움  
    2. 과적합(Overfittting) 문제  
    3. 느린 학습시간
1. 
### 1. EarlyStopping
### 2. Sequential 모델

## 2. CNN (Convolutional Neural Network)
  1. 합성곱 층(Convolutional Layer)  
    - 필터(filter) : 입력 이미지의 작은 패치와 컨볼루션 연산을 수행하는 매개변수, 작은 크기의 행렬로 이미지 전체를 스캔하며 활성화 맵을 생성한다.  
    - Stride : 필터가 입력 이미지를 이동하는 간격 (클수록 출력 이미지가 작아진다.)  
    - Padding : 입력 이미지 가장자리
  2. 풀링 층(Pooling Layer)  
    - Max Pooling : 주어진 영역 내에서 가장 큰 값을 선택하여 특징을 추출하고, 모델의 복잡도를 낮춘다.  
    - Average Pooling : 평균 값을 계산하여 특징을 추출한다.

  - 완전연결 층(Fully Connected Layer) 마지막 단계

  - 활성화 함수(Activation Function)