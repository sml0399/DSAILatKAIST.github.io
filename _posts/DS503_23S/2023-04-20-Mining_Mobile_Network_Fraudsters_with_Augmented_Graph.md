﻿---
title:  "[Entropy 2023] Mining Mobile Network Fraudsters with Augmented Graph"
permalink: Mining_Mobile_Network_Fraudsters_with_Augmented_Graph.html
tags: [reviews]
use_math: true
usemathjax: true
---


## 1. Introduction
세계적으로 휴대폰을 이용한 보이스피싱 범죄 등 사기 범죄가 증가하고 있으며, 5G 네트워크의 상용화로 인해 이러한 사기 범죄자들과 피해자의 수는 더욱 증가하고 있다. 이에 따라 통화내역 분석을 통해  사기 범죄자들을 찾아내는 것은 중요한 연구 주제이다.  이 논문에서는 5G 휴대전화 통신망에서 그래프 마이닝 기술을 사용하여 사기 범죄자들을 탐지하는 방법을 제시한다. 이를 위해서는 데이터 저장, 정제, 분석 및 의사 결정 단계가 필요하며, 이 단계를 "4D"로 아래 그림과 같이 요약할 수 있다.
![Figure 1](https://user-images.githubusercontent.com/109724570/232186915-21048ca1-4646-42a7-9d5a-cb74af9e6db1.png)

최근 그래프 기반 머신러닝(GNN) 기술이 부상하면서 그래프 방식으로 사기 탐지 문제를 다루는 것이 주류가 되었. 그러나 현재 그래프 머신러닝을 이용한 통신 사기 탐지에는 두 가지 주요한 문제가 있다.

첫째, 대부분의 연구에서 그래프 불균형 문제를 고려하지 않았다는 점이다. 좀 더 구체적으로 사기범죄자들은 전체 사용자 중에서도 소수를 차지하며, 이로 인해 그래프 신경망 모델이 메시지 집계 후 정상 사용자로 사기범을 판별하기가 쉽다. 

둘째, 레이어 수가 증가하면서 GNN의 성능이 감소하는 과도한 스무딩 문제이다. 이 문제는 GNN의 성능에 영향을 주는 중요한 요인이기도 하다.

이 논문에서는 이러한 문제를 해결하여 사기범지자들을 더 정확하게 판별할 수 있는 방법들을 제시하고자 한다.

## 2. Problem Definition
### 2.1. 그래프 불균형 문제(Graph imbalance problem)
![equation 1](https://user-images.githubusercontent.com/109724570/232190139-639fab97-d3cd-4569-ad05-0391a313666b.png)

이 논문에서는 그래프 불균형 문제를 위와 같은 식으로 수치화 한다.  따라서 IR 수치는 0~1 사이에 존재하며, IR이 1보다 작을 때는 데이터가 불균형하고, IR이 1일 때는 균형하다고 간주한다.

### 2.2. 그래프 기반 통신 사기 탐지(Graph-based telecom fraud detection)
모바일 네트워크에서 구독자(개인)를 노드(Node)로, 구독자의 행동을 노드(Node) 특성으로, 구독자 간의 통신 행동을 엣지(edge)로 설정하여 그래프를 구축한 뒤, GNN을 사용하여 사기 구독자를 탐지한다. 구체적으로, 모바일 네트워크의 사용자 행동 그래프 G=(V, X, A, E, Y)가 정의된다. 

V는 모바일 네트워크의 사용자를 나타내는 노드(Node) 집합이다.
X는 노드(Node)들의 행동 특성 집합이다. 벡터를 행렬로 쌓으면 그래프 G의 특성 행렬 X가 생성되는 것이다.
A는 그래프 G의 인접 행렬을 나타내는데, ai,j = 1은 i번째 노드와 j번째 노드(Node) 사이에 연결이 있다는 의미이고, ai,j = 0은 두 노드(Node)간에 연결이 없다는 것을 의미한다.
E는 엣지(Edge)의 집합을 나타낸다. 
Y는 집합 V의 모든 노드(Node)에 대응하는 레이블 집합이다.

위 값들로 그래프 신경망은 노드 임베딩(노드 -> 벡터) 표현 학습을 할 수 있는 것이다.


## 3. Proposed Method
![Figure 2](https://user-images.githubusercontent.com/109724570/232192547-69319891-b5d6-4310-ab41-76fd02d30cde.png)

이 논문에서 제안하는 방법의 파이프라인은 그림 2와 같다. 먼저, 노드의 원래 특성을 완전 연결 계층(Fully connected layer)에 입력하여 변환한다. 그런 다음 변환된 그래프와 노드 임베딩을 잘 설계된 강화 학습 기반 이웃 샘플러에 입력하여 노드 유사성을 기반으로 노드 이웃을 샘플링한다. 그 후, GNN을 사용하여 필터링된 노드 이웃에 메시지 집계를 수행하고 노드 임베딩을 학습한다. 이 과정을 약한 분류기로 간주하고, 앙상블 학습 방법인 AdaBoost를 사용하여 여러 약한 분류기를 통합하여 각 노드의 최종 임베딩 표현을 얻는다. 마지막으로, 균형 잡힌 포컬 손실 함수(a balanced focal loss function)를 사용하여 위의 훈련 과정을 감독한다. 아래에서는 각 과정에 대해 상세히 소개한다.

### 3.1.  강화학습 기반 이웃 추출(Reinforcement Learning-Based Neighbor Sampler)
그래프 불균형 문제를 해결하기 위해, 강화 학습 기술을 사용하여 각 노드의 이웃을 샘플링하여 유효한 이웃만 유지한다.. 전체 과정은 노드 특성 변환과 이웃 샘플링의 두 단계로 나뉜다.

#### 3.1.1.   Node Feature Transformation
노드를 임베딩하기 위해 여기서는 제안된 모델의 각 계층에서 FCN(Fully Conneted Network)을 노드 레이블 예측기로 사용하고 두 노드의 예측 사이의 유클리드 거리를 노드 유사도 측정치로 사용한다. 이를 통해 변환된 임베딩에 대한 식(equation)은 아래와 같이 표현된다.
![equation 2](https://user-images.githubusercontent.com/109724570/232194241-410e5aa7-3533-48ce-b656-3990b4ade967.png)

여기서 σ는 활성화 함수를 의미하며, 이 논문에서는 ReLU를 활성화 함수로 사용하였다. W는 학습된 가중치(weight)이다.
이후 GNN _l_ 계층에서 노드 v와 이웃 노드 v' 사이의 거리를 계산할 때, 유클리드 거리를 이용하여 거리를 계산하고 그 식은 아래와 같다.

![equation 4](https://user-images.githubusercontent.com/109724570/232194467-8b1279fe-aba5-43f4-b0c0-4b68ab4a6879.png)

여기서 목표는 노드들을 임베딩할 때 같은 클래스의 것은 가깝게, 다른 클래스의 노드들은 멀리 있게 하는 것으로서 레이블의 신호를 사용하여 완전 연결 계층을 훈련시키기 위해 교차 엔트로피 손실 함수의 최소화를 최적화 목표로 사용한다.
![equation 5](https://user-images.githubusercontent.com/109724570/232194802-54ad7b60-af1b-4c31-9919-2722c6524018.png)

#### 3.1.2. Neighbor Sampling
그래프에서 각 노드는 많은 수의 이웃 노드를 가지지만, 이들이 중심 노드의 클래스를 결정하는데 동일하게 기여하지는 않는다. 이러한 불균형 문제는 그래프 모델에 치명적일 수 있으므로 이 연구에서는 불리한 영향을 균형 있게 만드는 샘플링 메커니즘을 설계한다. 최적의 이웃 샘플링을 위해 샘플링 기준을 설계하고, Bernoulli multi-armed bandit (BMAB)기반의 강화 학습(RL) 알고리즘을 사용하여 데이터에서 가장 적절한 샘플링 확률 p를 자동으로 학습한다. 이후 중심 노드와의 유사도를 기준으로 이웃 노드를 순위화하고, 각 노드 v에 대해 p * Degree(각 노드와 연결된 노드의 수) 이웃을 샘플링한다. 
4번식에 따라 계산된 유클리디안 거리에 따라서 인접한 노드 v, v'간의 유사도는 아래식과 같이 계산할 수 있다.
![equation 6](https://user-images.githubusercontent.com/109724570/232195740-e6751a93-c2e1-4828-a222-aefdffd7fb11.png)

훈련 중인 이웃 샘플러의 샘플링 효과를 측정하기 위해 노드들의 평균 유사도를 정의한다. 구체적으로, e번째 epoch의 _l_-번째 계층에서 노드들의 평균 유사도가 아래 식과 같이 정의된다.
![equation 7](https://user-images.githubusercontent.com/109724570/232196170-dfb7faf5-7546-4299-8cba-6444aeda69a7.png)

BMAB 모델은 B(**S, A, f, T**)로 표현할 수 있으며, 여기서 S는 상태 공간, A는 행동 공간, f는 보상과 패널티 함수, T는 종료 조건이다.. 주어진 초기 샘플링 확률 p(_l_)에 대해, 에이전트는 현재 상태를 기반으로 행동을 선택하고, 보상은 연속적인 두 에포크 사이의 평균 유사도 차이에 따라 달라진다. 다음으로, 각 BMAB 구성 요소의 세부 사항을 소개한다.

**State** 위 방정식에서 분자는 훈련 데이터 세트의 각 노드와 이웃 사이의 유사도 누적 합을 나타내고, 분모는 훈련 데이터 세트의 엣지 수를 나타낸다. 더 큰 평균 유사도를 구하는 것이 우리의 목표이므로 인접한 두 에포크의 노드 평균 유사도에서의 양수 또는 음수 차이를 시스템 상태로 정의한다. 즉, 상태 집합 S = {s1, s2}으로 나타낸다. 여기서 s1은 현재  에포크의 평균 노드 유사도가 이전 에포크보다 클 때이고, s2는 현재  에포크의 평균 노드 유사도가 이전 에포크보다 작거나 같음을 나타낸다.

**Action** 설계된 강화학습 모듈의 목표는 최적의 필터링 임계값 p를 학습하는 것이다. 모델 실행 중에는 BMAB가 실시간 상태 s에 따라 p의 크기를 조정해야한다. 여기서 행동 공간 A = a1, a2를 제공하며, 행동 a1은 샘플링 확률 p에 고정 길이 τ를 더하고, 행동 a2는 샘플링 확률 p에서 고정 길이 τ를 빼는 것이다. 상태 s ∈ S에 대해, 상태와 행동 사이의 매핑 관계 π는 아래 식과 같이 정의된다.  여기서 τ는 이웃 샘플링 확률 p의 변화 단계 크기로, 모델의 초매개변수로 수동으로 설정된다.
![equation 8](https://user-images.githubusercontent.com/109724570/232197272-d11547cb-0582-4f9e-b0b1-e523f9c605d9.png)

**Reward** 적절한 샘플링 확률 p는 중심 노드와 유사한 이웃 노드를 적절한 수만큼 샘플링하면서 큰 차이를 갖는 이웃 노드는 버리도록 해야 한다. 따라서, 에이전트가 p(_l_) 하에서 훈련 세트의 노드들의 평균 유사도를 높이면 긍정적인 보상을 받고, 그렇지 않으면 부정적인 보상, 즉 패널티를 받는 보상 메커니즘이 아래 식과 같이 설계된다.
![equation 9](https://user-images.githubusercontent.com/109724570/232197487-821fbf37-513d-41b7-b17f-85a2de216e2c.png)

새로 선택된 이웃의 에포크 의 평균 거리가 이전 에포크보다 작을 때 +1을 보상하고, 그 반대의 경우에는 -1을 보상한다.

**Termination** 
적절한 종료 조건은 BMAB에서 최적의 임계값 p를 얻고 컴퓨팅 리소스를 절약하는 데 중요하다. 만약 강화학습이 마지막 15 에포크에서 수렴한다면, 최적의 임계값 p(l)이 발견되었으으므로 샘플러를 종료해야 한다. 여기서 공식적인 종료 조건을 다음식과 같이 제시한다
![equation 10](https://user-images.githubusercontent.com/109724570/232200314-0b15e1b4-c6e4-47f7-83ae-a75f6b49be01.png)


이것은 강화학습 기반 학습과정에서 15개 연속 에포크의 보상 합계가 2 이하임을 의미한다. 샘플러가 최소한 15 라운드 동안 학습되도록 하기 위해 e ≥ 15로 지정한다. 위 부등식이 만족되면 샘플러가 수렴하고 이 시점에서 최적의 이웃 샘플링 확률 p를 얻는다.

### 3.2. GNN-Based Weak Classifier
이웃 샘플링 후에는 노드 이웃 정보를 집계해야 한다.  이 논문에서는 일반적인 GNN(예: GCN, GAT)을 약한 분류기로 취급한 다음 앙상블 학습(섹션 4.3 참조)을 사용하여 GNN의 전체 성능을 개선하려는 것이다. 여기서는 Message Passing Neural Network(MPNN) [49]을 사용하여 다양한 유형의 GNN을 일관되게 설명한다. MPNN의 순방향 전달에는 메시지 전달 단계와 읽기 단계의 두 단계가 있다. 메시지 전달 단계는 _t_ time steps 동안 실행되며, 메시지 함수 Mt 및 노드 업데이트 함수 Ut에 따라 결정된다. 메시지 전달 단계 동안 각 노드의  hidden state(t) 는 메시지 m_(t+1)에 따라 조정된다.
![equation 11-13](https://user-images.githubusercontent.com/109724570/232201927-912b91a9-8ef2-4e75-b7ec-1baa2fffac74.png)

yˆv는 노드 v의 라벨 예측 결과이다. 메시지 함수 Mt, 벡터 업데이트 함수 Ut 및 readout 함수 R은 모두 학습 가능한 미분 가능 함수이다. 모델 구현에서는 GraphSAGE의 평균 집계기를 사용하여 메시지 집계를 수행한다. 이 방법은 attention 및 기타 방법에 비해 모델 복잡성을 크게 줄인다. 식은 다음과 같다.
![equation 14](https://user-images.githubusercontent.com/109724570/232202429-9fbb50f9-8925-4b97-a91d-0a2a7c89ddad.png)

여기서 σ는 활성화 함수, W는 학습할 가중치, N(v)는 노드 v의 이웃이다. MEAN()은 집합 내 노드 임베딩의 평균을 계산하는 연산을 나타낸다.

### 3.3. Ensemble GNN with SAMME.R
원래의 GAT(Graph Attention Network) 모델은 대부분의 GNN 모델처럼 오버스무딩 문제로 인해 GAT 레이어 수가 증가함에 따라 성능이 급격히 떨어진다.  우리는 이 문제를 해결하기 위해 이전 섹션의 각 GNN 약한 분류기로부터 얻은 노드 임베딩을 SAMME.R 알고리즘 [50]과 결합하여 앙상블 학습을 고려한다. 구체적으로, 각 GNN 약한 분류기에 다른 가중치를 할당하고, 여러 GNN에 의해 학습된 노드 임베딩을 통합하여 최종 임베딩을 얻는다.

여기서 제안하는 방법은 l-번째 레이어(즉, l-번째 GNN)를 l-번째 기본 분류기로 간주한다. l-번째 GNN에서 노드 v의 최종 임베딩 z(l)v를 얻은 후, 소프트맥스 함수로 z(l)v를 정규화하여 노드 v가 l-번째 약한 분류기에서 각 클래스에 속할 확률을 얻는다. 따라서 노드 v가 k개의 다른 카테고리로 분류될 확률 벡터를 아래 식과 같이 표현할 수 있다.
![equation 15](https://user-images.githubusercontent.com/109724570/232202636-d98c6108-898c-47e6-854f-fae6d1e12a3f.png)

첫 번째 (l-1) 번째 기본 분류기가 f(l-1)일 때, 새로운 기본 분류기 h(l)(v)를 추가하여 전체 모델의 성능을 개선하고자 한다면 l-번째 GNN의 출력에 따라 SAMME.R 알고리즘과 결합하여, l-번째 레이어의 기본 분류기 h(l)(x)를 아래 식과 같이 계산할 수 있다.
![equation 16](https://user-images.githubusercontent.com/109724570/232202811-0a4bf324-d99b-42db-8626-7a24d45cb576.png)

잘못 분류된 노드가 (l + 1) 번째 기본 분류기에서 올바르게 분류될 확률을 높이기 위해, 노드의 가중치를 아래 식과 같이 업데이트한다.
![equation 17](https://user-images.githubusercontent.com/109724570/232202870-2abe43f4-d89f-4c28-aae2-5602e516fb27.png)

여기서 w(l+1)v는 (l + 1) 번째 기본 분류기에서 노드 v의 가중치를 나타내고, p(v)는 (15)번 식으로 계산된다. 이후 각 레이어의 기본 분류기 h(v)를 결합하여 앙상블 분류 결과를 아래와 같이 얻는다.
![equation 18](https://user-images.githubusercontent.com/109724570/232202993-c70b7a52-ce2d-4464-898e-d67695be34e7.png)

### 3.4. Proposed Algorithm
손실 함수는 불균형 학습에 매우 중요하다. 그러나 널리 사용되는 교차 엔트로피 손실 함수는 극단적으로 불균형한 문제에서 성능이 좋지 않는 문제가 있다. 여기에서는 그래프 불균형 문제에 대해 아래 식과 같이 컴퓨터 비전 분야의 클래스 균형 집중 손실(class-balanced focal loss)을 사용하여 제안된 모델의 학습 과정을 제한한다.
![equation 19](https://user-images.githubusercontent.com/109724570/232203148-a00ce9ab-705f-4afd-9def-cccaf47bec5c.png)

여기서 초매개변수 β ∈ [0, 1]는 표본 수 n이 증가함에 따라 유효한 수가 얼마나 빠르게 증가하는 지를 제어하며, n ∈ Z은 표본 수이고, γ ≥ 0은 조절 가능한 집중 매개변수이다.

지금까지 설명한 이 논문에서 제시하는 모델의 훈련과정을 종합하면 아래 알고리즘1과 같다.
![algorithm](https://user-images.githubusercontent.com/109724570/232203319-fb3fdfdf-434f-4a6c-a448-3fb873003d4d.png)

## 4. Experiments
이 섹션에서는 두 개의 실제 통신사기 탐지 데이터셋에서 제안된 방법의 성능을 제시한다. 주요 포함 내용은 다음과 같다.
 • 그래프 기반 이상 탐지에서 최신 기술과 제안된 방법의 비교
 • 제안된 방법의 그래프 불균형 문제를 해결하는데 효과적인지 
 • 제안된 방법이 GNN over-smoothing 문제를 극복하는데 효과적인지

### 4.1. Experimental Setup
#### 4.1.1. Dataset
이 논문은 중국의 두 개의 실제 불균형한 통신사기 탐지 데이터세트인, Sichuan과 BUPT를 사용한다. Sichuan 데이터셋은 중국 Sichuan 지역의 23개 도시에서 6106명의 사용자의 CDR(call detail records) 데이터를 포함하고 있으며, 불균형 비율은 0.4735(IR = 1962/4144 = 0.4735)이다. BUPT 데이터셋은 중국 도시의 일주일간 사용자의 CDR 데이터를 포함하고, 불균형 비율은 0.0809(IR = 8074/99861 = 0.0809이다. 데이터는 사기 범죄자와 정상 사용자 등으로 구분된다. 자세한 내용은 아래 표1과 같다.

![table 1](https://user-images.githubusercontent.com/109724570/232206987-0eb8fe1f-2a8f-4b4a-a560-64a0e980a48a.png)

#### 4.1.2. Baseline Method
이 논문에서 제안하는 방법의 효과를 검증하기 위해, 반지도 학습 환경에서의 다양한 GNN 베이스라인과 비교를 시행한다. 구체적으로 GCN, GAT, 그리고 GraphSAGE를 일반적인 GNN 모델로 선택했으며,  FdGars, GraphConsis, GEM, SemiGNN, 그리고 BTG를 최신 GNN 기반 사기 탐지기로 선택하였다.

• GCN: 스펙트럼 그래프 합성곱을 사용한 이웃 정보 집계 GNN 
• GAT: 이웃 노드 정보를 집계하는 어텐션 메커니즘을 사용한 GNN 
• GraphSAGE: 고정된 수의 샘플링된 이웃을 갖는 귀납적 GNN 
• FdGars: GCN 기반 사회적 의견 사기 탐지 시스템 
• GraphConsis: 그래프 불일치를 위한 이종 그래프 신경망 
• GEM: 어텐션 메커니즘 기반 이종 그래프 사기 탐지 GNN •
 SemiGNN: 다중 뷰를 포함한 계층적 어텐션 집계를 위한 GNN 
 • BTG: 희소 그래프를 위한 GNN 기반 통신 사기 탐지기

#### 4.1.3. Evaluation Metrics and Experiment Settings
불균형 문제의 평가를 위해 **Macro AUC, Macro recall, Accuracy,** 그리고 **Marco F1**과 같은 네 가지 널리 사용되는 지표를 사용하여 모든 비교 방법의 성능을 측정하였다.

**AUC** ROC 곡선 아래의 면적으로 정의되며, 식은 아래와 같다.
![equation 20](https://user-images.githubusercontent.com/109724570/232207384-894c1ae3-930a-4a94-a771-a6565b48cd84.png)

U+와 U-는 각각 테스트 세트의 소수 및 다수 클래스 집합을 나타낸다. 또한, ranku는 예측 점수를 기준으로 노드 u의 순위를 나타낸다.

**Recall** 불균형 문제에 대해 매우 중요한 지표로서, 감지된 소수 중요한 카테고리의 비율을 정확하게 측정할 수 있다. 이는 다음과 같이 정의된다.  

![equation 21](https://user-images.githubusercontent.com/109724570/232207586-b610eed2-cc35-4c0a-a8be-eb12b82e518d.png)

TP와 TN은 각각 혼동 행렬에서 true positive 및 true negative 샘플의 수를 나타낸다. 여기서는 Macro recall이 사용되었는데 이는 다중 클래스의 산술 평균으로, 다른 클래스의 중요성에 관계없이 모든 클래스를 동등하게 취급한다.

**Accuracy** 올바르게 분류된 샘플 수와 전체 샘플 수의 비율로 정의된다.
![equation 22](https://user-images.githubusercontent.com/109724570/232207744-6a3f1a36-fb1f-434e-86c4-822c51753f51.png)

**F1**  F1 스코어는 불균형 문제를 평가하기 위한 또 다른 종합적인 지표로, 다음과 같이 정의된다.
![equation 23](https://user-images.githubusercontent.com/109724570/232207826-be03ce44-0316-4fb8-89da-96f864e5eb3b.png)

여기서 사용된 매크로 F1은 각 클래스에 대한 F1 점수의 산술 평균을 계산하는 것이다.

위의 지표에서 점수가 높을수록 모델의 성능이 더 좋다는 것을 의미한다. 실험에서는 훈련 샘플을 무작위로 선택하고 훈련 세트의 양성 및 음성 샘플 비율을 전체 데이터 세트와 동일하게 유지하였다. 제안된 방법에서는 실험에서 Adam optimizer를 사용하여 매개변수 최적화를 수행하며, 구체적인 설정은 다음과 같다. 두 데이터 세트 모두 hidden 임베딩 크기(64), 학습률(0.01), 모델 레이어(2), 훈련 크기(0.2), 테스트 크기(0.6), 최대 epoch(30) 및 γ=2, α=1로 설정하였다.. 제안된 방법은 Pytorch로 구현되며, 모든 모델은 Python3.7.10, 1 GeForce RTX 3090 GPU, 64GB RAM, 16 코어 Intel(R) Xeon(R) Gold 5218 CPU @2.30GHz 리눅스 서버에서 실행되었다.

### 4.2. Overall Evaluation
실험에서 훈련 크기:검증 크기:테스트 크기를 0.2:0.2:0.6으로 설정하였다.아래  표 2에 나와있는 4가지 평가 지표에서 기준 방법과 제안된 모델의 점수를 확인한 결과 우리가 제안하는 방법이 두 실제 통신 사기 탐지 데이터 세트의 모든 지표에서 다른 방법들보다 우수한 것을 알 수 있었다. 또한 실험 결과에서 일반 GNN(예: GCN, GAT, GraphSAGE)이  FdGars, GraphConsis, GEM, SemiGNN과 같은 GNN 기반 사기 탐지기보다 성능이 좋다는 것을 알 수 있었다. 

![table 2](https://user-images.githubusercontent.com/109724570/232208249-2865903a-b548-43e3-8ce9-442560c07772.png)

### 4.3. Alleviation of Graph Imbalance Problem
그래프 불균형 문제에 대한 효과를 검증하기 위해 다양한 IR(0.1에서 1까지 범위)에 따라 두 통신 사기 데이터 세트에서 무작위로 샘플을 추출하여 IR에 맞는 새로운 데이터 세트를 생성하였다. 가장 성능이 좋은 기준 방법(GCN, GAT, BTG) 3가지를 선택하여 이 논문에서 제안하는 방법과 비교한 결과가 아래 그림 3와 같다.
![Figure 3](https://user-images.githubusercontent.com/109724570/232208483-8a92de74-cabc-426f-b063-bd6f2bf244b5.png)

그림에서 볼 수 있듯이 이 논문에서 제안된 방법은 다양한 불균형 비율에서 기준선보다 성능이 좋은 것을 확인할 수 있다. IR이 감소할 때 모든 모델의 점수가 감소하지만, 이 논문에서 제안된 방법은 더 느리게 감소한다. 이는 제안된 방법의 불균형 문제에 대한 특별한 설계가 중요한 역할을 하는 것을 입증한다.특히, 강화학습 기반의 이웃 샘플링 전략은 데이터 분포를 균형 있게 유지하고 클래스별로 균형 잡힌 포컬 손실이 모델 훈련 과정에서 감독 역할을 한다.

### 4.4. Avoidance of Over-Smoothing Effects
GNN 오버스무딩 문제에 대한 제안된 방법의 효과를 확인하기 위해, AUC와 리콜에서 GCN, GAT, BTG와 제안된 방법의 성능을 비교한다. 이번에는 공간문제상 BUPT 데이터셋에서만 실험을 진행하였다.. 신경망 계층 수(1-21)를 변경하고 나머지 하이퍼파라미터는 동일하게 설정한 결과 아래 그림과 같이 GCN, GAT 및 BTG는 오버스무딩 효과에 분명히 영향을 받는다. 즉 네트워크 계층 수가 증가함에 따라 AUC와 리콜의 성능이 크게 저하된다. 반면, 이 논문에서 제안된 방법의 점수는 천천히 상승한 다음 안정적으로 유지되는 것이 확인된다. 이는 제안된 방법이 GNN의 오버스무딩 효과를 효과적으로 피할 수 있음을 보여준다. 이는 SAMME.R 알고리즘을 사용하여 여러 GNN 모델을 앙상블하기 때문에 오버스무딩 효과를 피할 수 있기 때문이다.

![Figure 4](https://user-images.githubusercontent.com/109724570/232208861-521506e9-0ae1-46ca-a54e-545b3719ff0f.png)

## 5. Conclusions
본 논문에서는 그래프 불균형과 과적합 문제를 해결하기 위해 강화학습과 앙상블 학습을 결합한 새로운 GNN 모델을 제안한다. 실험 결과, 이 모델이 효과적임을 보여주며, 불균형 문제에 대한 강화학습 기반 이웃 샘플링 메커니즘과 과적합 문제에 대한 AdaBoost 기반 앙상블 아키텍처가 유용하다는 것이 발견되었다. 이러한 모델은 통신 사기 탐지뿐만 아니라 사회적 네트워크 사기 탐지, 금융 사기 탐지, 사이버 보안 등 그래프 데이터 불균형 문제가 발생하는 다른 분야에도 적용될 수 있다.

## 6. Author Information 
### 6.1. Author affiliation
**Xinxin Hu,  Hongchang Chen, Xing Li, Junjie Zhang, and Shuxin Liu ** National Digital Switching System Engineering and Technological Research Center, Zhengzhou 450002, China
**Haotian Chen** The Edward S. Rogers Sr. Department of Electrical & Computer Engineering, University of Toronto, Toronto, ON M5S 3G4, Canada
### 6.1. Author r Contributions
X.H.: Conceptualization, Methodology, Software, Writing—Original draft; H.C. (Haotian Chen): Methodology, Software, Validation; H.C. (Hongchang Chen): Formal analysis, Project administration, Supervision; X.L.: Supervision,Visualization; J.Z.: Formal analysis, Writing— Review & Editing; S.L.: Funding acquisition, Supervision. All authors reviewed the manuscript. All authors have read and agreed to the published version of the manuscript.

## 7. Additional materials
Data Availability Statement: The datasets that support this study are available from https://aistudio.baidu.com/aistudio/datasetdetail/40690 (accessed on 1 September 2022) and https://github.com/ khznxn/TF-Dataset (accessed on 1 September 2022).
