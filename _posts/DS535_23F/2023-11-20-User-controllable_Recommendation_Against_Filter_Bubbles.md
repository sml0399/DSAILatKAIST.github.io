﻿---
title:  "[SIGIR 2022] User-controllable Recommendation Against Filter Bubbles"
permalink: 2023-11-20-User-controllable_Recommendation_Against_Filter_Bubbles.html
tags: [reviews]
use_math: true
usemathjax: true
---

# [SIGIR-22]User-controllable_Recommendation_Against_Filter_Bubbles



## **1. Problem Definition**[](https://dsailatkaist.github.io/template.html#1-problem-definition)

과거 몇 년 동안 추천시스템은 '개인화된 정보 필터링'이라는 명목으로 정보의 과잉을 감당하기 어려운 사용자들에게 불필요한 정보를 필터링 해주어 빠른 발전을 거두었지만, '필터버블(Filter Bubble)'에 대한 논의는 항상 이어져 왔었다. 

필터버블이란, 추천시스템이 사용자-아이템 간 상호작용을 기반으로 기존 사용자의 선호도에 일치하는 아이템만 계속해서 노출시키는 현상을 가리킨다.  이런 현상이 반복되면, 유사한 아이템만 노출되는 확률이 계속해서 커지게 되고, 사용자가 다양한 카테고리의 콘텐츠를 접할 수 있는 기회가 점점 줄어지게 된다.

즉, 장기적인 관점에서 필터버블은 아이템 혹은 콘텐츠의 다양성과 오리지널리티를 추천시스템에서 배제시키게 되고, 이는 필연적으로 정보의 편식으로 인해 사용자로 하여금 왜곡 효과를 낳게 된다.

따라서, 필터버블을 완화하는 것 또한 추천시스템에서 중요한 과제로 자리매김하게 되었다.

## **2. Motivation**[](https://dsailatkaist.github.io/template.html#2-motivation)

필터버블을 완화하는 방안으로 기존 연구에서는 '다양성(Diversity)'과 공정성(Fairness)'을 높이는 방법을 제안했었다.

- **다양성**:
다양성은 추천 목록에서 유사성이 다른 아이템을 생성하도록 장려하는 방안이다. 이는 후처리 및 엔드-투-엔드 방법으로 나눌 수 있다. 전자는 몇몇 모델에 의해 생성된 추천 목록을 다시 순위 지정을 통해 다양화시키며, 후자는 모델 훈련 및 예측 과정에서 정확도와 다양성의 균형을 맞추는 방향으로 진행된다. 그러나 이러한 방식들 역시 단순히 사용자에게 다양한 아이템을 추천한 후 사용자의 피드백을 통해 새로운 아이템 카테고리를 발굴해가는 것으로 많은 시간이 필요하다. 심지어 다양한 아이템을 추천하는 단계에서 사용자 선호도와 관련 없는 아이템을 많이 가져올 수 있다는 단점이 있다. 

- **공정성**:
추천시스템에서 공정성을 달성하기 위해서는 다양한 사용자 그룹 또는 아이템 카테고리 간에 균형을 맞추어야 한다. 이것은 특정 그룹에게 더 많은 추천을 하도록 조절하거나 특정 카테고리의 아이템을 다른 것보다 자주 포함시키는 것을 의미하여 필터버블을 어느 정도 완화할 수 있다. 그러나 이렇게 균형을 맞추는 과정에서, 일부 사용자 또는 아이템 그룹에 대한 추천 정확성을 희생시킬 수 있다. 예를 들어, 만약 특정 사용자 그룹에게 더 많은 공정성을 부여하려면 그 그룹에 대한 추천 목록에서 다른 사용자 그룹의 선호를 무시하고 그 그룹에 맞춰야 할 수 있기 때문에 그 그룹에 대한 정확한 추천이 희생되고 사용자 경험이 저하될 수 있다.

이렇듯 기존 접근 방식은 다양성, 공정성을 고려하여 필터버블을 완화하지만 정확성과 사용자 경험을 희생해야 한다는 단점이 있다. 

또한, 사용자의 피드백을 통해 추천시스템이 모델 훈련-예측의 무한 루프를 도는 과정에서, 사용자는 추천 결과를 수동적으로만 받아들이게 때문에 진정한 '개별맞춤' 결과를 생성하기까진 많은 시간과 비용을 필요로 한다. 즉, 비록 추천시스템이 사용자의 선호도를 기반으로 결과를 생성하긴 하지만, 다양성과 공정성 향상 과정에서 다시 불필요한 정보까지 포함시킬 수 있기 때문에 사용자가 자신에게 필요한 정보만을 얻기 위해선 생성된 추천 결과에 대해 'like' 혹은 'dislike' 등 지속적인 피드백을 제공하고 학습을 시켜야 한다.

따라서, 저자는 사용자가 직접 컨트롤을 통해 자신이 원하는 추천 결과를 생성할 수 있게끔 'User-Controllable Recommendation System(UCRS)' 방안을 제시했다.

![rsloop](https://i.ibb.co/9wXYx34/rsloop.png)

사용자가 제어할 수 있는 추천시스템인 UCRS는 기존 추천시스템 외에 아래 3가지 기능을 추가시켰다.

 - **필터버블의 탐지하여 사용자에게 알려주는 필터버블 경고 기능**
 -  **4가지 수준의 제어 명령 기능**
 -  **사용자 제어에 따라 추천결과를 조정하는 응답 메커니즘**

이로써, 저자는 사용자가 추천시스템의 동작을 제어할 수 있는 방법을 제공하여 사용자의 참여를 촉진하고, 사용자가 더 많은 표현력과 제어권을 가지게 함으로써 사용자 경험을 향상시키는 것을 목적으로 삼았다.

이와 더불어, 저자는 앞서 말한 응답 메커니즘 단계에서 사용자 A가 1년 전에는 액션 영화를 많이 시청했지만 현재는 코미디 영화에 더 관심이 있을 수 있는 것처럼 시간이 지남에 따라 사용자의 선호도가 변할 수 있다는 점까지 인식하였다. 이에 따라, 사용자 표현의 오래된 정보가 추천에 미치는 영향을 완화하는 데 중점을 둔 'User-Controllable Inference(UCI)' 프레임 워크가 제안되었다.

UCI는 사용자가 제어 명령을 제공하면,  반 사실(counterfactual)인 대조적 추론을 사용하여 과거에 나온 사용자 표현의 영향을 줄이는 방안이다. 예를 들어, 여성인 사용자 B는 여성 사용자 그룹이 선호하는 영화 리스트에 질려 남성 사용자 그룹이 선호하는 영화 리스트를 추천시키는 제어 명령을 내렸다고 가정하자. 이 때, 반 사실인 대조적 추론은 '사용자 B가 남성이라면 추천 리스트는 어떻게 변할까?'라는 질문의 대한 예측이라고 생각할 수 있다. 즉, UCI는 오래된 사용자 표현이 폐기되는 반 사실 세계를 가정하고 이런 조건에 맞는 새로운 추천 결과를 생성한다. 이로써 과거 사용자-아이템 간 상호작용 패턴에 극한되지 않고 사용자가 원하는 추천을 얻을 수 있다.

## 3. Preliminary
### 3.1 Experimental settings
저자는 Method를 확립하기에 앞서 추천시스템에서 필터버블이 어떠한 유형으로 발생되는지 알아보는 사전 실험을 수행했다:

 1.  대표적인 추천 모델인 Factorization Machine (FM)을 DIGIX-Video, Amazon-Book 및 ML-1M과 같은 세 개의 공개 데이터셋에 훈련시킴.
 2. 각 사용자에 대해 상위 10개의 추천 아이템을 수집함.
 3. 사용자 그룹을 ID, 성별 및 연령을 고려한 사용자 특성(User Features)과 아이템 카테고리에 대한 관심도 등을 고려한 사용자 상호작용(User Interactions) 두 가지 요인으로 분류함.
 4. FM이 생성한 추천결과와 사용자 그룹에 따른 사용자의 과거 상호작용 패턴을 비교함.

### 3.2 Analysis
분석 결과로는, 사용자 특성과 아이템 특성(Item Features) 2가지 측면에서 필터버블이 존재한다는 사실이 발견되었다.

![fbresult](https://i.ibb.co/0c9Qt8R/fbresult.png)

이미지 2(a)는 DIGIX-Video의 남성 및 여성 사용자별 상위 3개 아이템 카테고리에 대한 과거 분포를 시각화한 결과이다. 여성 사용자 그룹은 로맨스 영화를 더 선호한 반면, 남성 사용자 그룹은 액션 영화를 더 선호했다. 그 결과, 이미지 2(b)와 2(c)에서 알 수 있듯이, 추천 결과에서도 편향된 분포를 유지하게 되었다.

이러하듯이, 사용자는 계속해서 유사한 아이템을 추천 받게 된다. 추천 모델은 이러한 편향을 강화하고 상위 특정 카테고리를 더 노출시키는 경향으로 이어져 결국 남성과 여성 사용자 그룹 간의 심각한 분리를 야기시키게 된다.

또한, 이미지 2(d) 및 2(e)는 Amazon-Book 및 ML-1M 데이터셋에 대해 사용자 상호작용에 따라 나눈 결과이다. 동 결과에서도 과거 사용자 상호작용 패턴에 따른 카테고리 편향 증폭이 발견되었다. 즉, 사용자으로부터 가장 큰 관심을 받은 카테고리가 이후 추천 목록에서도 증가된 것이다. 이는 필터버블의 강화를 초래하고 사용자의 관심을 좁혀 사용자 그룹 분리로 이어지게 된다.

따라서, 저자는 사용자 특성과 아이템 특성 2가지 측면에서 필터버블을 완화하는 UCRS 체계를 제안했다.


## **4. Method**[](https://dsailatkaist.github.io/template.html#3-method)
전반적으로 저자는 커버리지(Coverage), 격리지수(Isolation Index), 최다 카테고리 지배도(Majority Category Domination, MCD) 등 지표를 통해 필터버블의 수준을 실시간으로 감지하고 사용자에게 알람을 보내는 경고 기능을 구현했다.

또한, 제어 명령 기능과 관련해서는 앞서 사전 실험 결과에 따라, 사용자 특성과 아이템 특성 2가지 방면에서 UCRS 제어시스템을 구현했다. 

마지막으로, 사용자 제어에 따라 추천결과를 조정하는 반 사실 추론 응답 메커니즘을 통해 필터버블을 완화하는 동시에 추천 정확성은 유지하고 사용자가 원하는 결과를 얻을 수 있는 추천시스템을 구현했다.

![ucrs](https://i.ibb.co/7Nrkm21/ucrs.png)

### 4.1 필터버블 감지 지표
- **Coverage**:
필터버블은 추천 항목의 다양성을 감소시키는 경우가 많으므로 추천 목록의 카테고리 수를 계산하는 Coverage를 다양성 척도로 사용했다.

- **Isolation Index**: 
다양성 척도 외에도 저자는 서로 다른 사용자 그룹 간의 분리수준을 측정하기 위해 Isolation Index를 제안했다. 두 사용자 그룹 a와 b가 주어졌을 때 권장 격리 지수는 다음과 같이 계산할 수 있다:
![isolation](https://i.ibb.co/vDmB3Dx/isolation.png)
여기서 $I$는 아이템 집합을 의미하며,  $a_ {i}$와 $b_{i}$는 그룹 $a$와 그룹 $b$에서 추천 아이템 $i$를 받은 사용자 수를 나타낸다. $a_ {n} = ∑_ {i ∈ I} a_ {i}$는 그룹 $a$에서 해당 아이템의 총 노출 빈도이며, $b_{n}$도 같은 방법으로 계산된다.  마지막으로, $s$는 그룹 a의 아이템 노출 빈도 가중 평균 값에서 그룹 b의 가중 평균 값을 뺀 값과 같으며, $\frac{a_ i}{a_ i+b_ i}$가 가중치가 된다. 즉, s는 0~1 사이의 값을 가지며,  두 그룹 간의 권장 분리 수준을 나타내고, 값이 클수록 더 심한 분리를 의미하게 된다. 

- **MCD**:
MCD는 아이템 특성과 관련된 필터버블을 감지하는 용도로 사용되어 가장 자주 추천되는 아이템 카테고리의 비율을 확인할 수 있다. 시간이 지남에 따라 MCD가 증가하면 아이템 카테고리와 관련된 필터 버블의 심각성이 커지고 있음을 나타낸다. 

결론적으로 이러한 지표들을 통해 사용자에게 실시간으로 필터버블 경고를 보내고 이를 제어할지 여부를 결정하게 도울 수 있다.

### 4.2 제어 명령 기능
사용자의 과거 상호작용 데이터인 𝐷가 주어졌을 때, 기존의 추천 모델은 추천 𝑅을 예측하기 위해 𝑃(𝑅|𝐷)를 사용한다. 그러나 UCRS는 추가로 사용자 제어인 𝐶를 고려하며 사용자 개입(𝑑𝑜(𝐶))을 통해 𝑃(𝑅|𝐷, 𝑑𝑜(𝐶))를 추정할 것을 제시했다. 이때, 사용자 및 아이템 제어 각각 'Fine-grained controls'와 'Coarse-grained controls' 2가지 세부 수준으로 또 나눠져 총 4가지 유형의 사용자 제어가 제시됐다.

#### 4.2.1 사용자 특성 제어(User-feature Controls)
N가지 사용자 특성과 사용자 𝑢가 있을 때, 사용자 𝑢는 $x_ u = [ x^1_ {u}, \ldots, x^n_ {u}, \ldots,  x^N_ {u} ], \text{ where }  x^n_ {u} \in \{0,1 \}$ 로 표현되며, 여기서 $x^n_ {u} \in \{0,1 \}$는 사용자 𝑢가 사용자 특성 $x^n$을 가지고 있는지 여부를 나타낸다. 예를 들어,  [$x^{1}$과 $x^{2}$]가 남성과 여성을 나타낸다면, $x_ {u}$ = [0, 1]은 사용자 𝑢가 여성임을 뜻한다. 

**Fine-grained controls**
사용자 특성 측면에서 발생되는 필터버블(예: 성별 및 연령)을 완화하기 위해 저자는 UCRS가 다른 사용자 그룹이 좋아하는 항목을 더 많이 추천하도록 하는 세분화된 사용자 특성 컨트롤을 설계했다. 

예를 들어, 30대의 중년 사용자는 10대가 좋아하는 영화에 관심이 있을 수 있다. 이 때, 사용자 $u$에 대한 $𝑃(𝑅 \vert 𝐷, 𝑑𝑜(𝐶))$를 계산할 때, 제어령은 $do(C=c_ u(+ \hat{x},\alpha))$와 같이 쓸 수 있다. 여기서 $c_u(+\hat{x},\alpha)$는 다른 사용자 그룹$\hat{x}$에서 많이 추천되는 아이템을 더 많이 노출시키는 제어령이다. 참고로 $\hat{x}$는 사용자 $u$가 기존에 가지고 있지 않았던 feature이어야 한다. $α ∈ [0,1]$는 사용자 제어의 강도를 조정하는 계수이다.

**Coarse-grained controls**
하지만, 사용자는 단순히 필터버블을 제거하길 원하지 다른 사용자 그룹이 좋아하는 아이템은 추천 받고 싶지 않을 수 있다. 혹은 어떤 사용자 그룹이 더 좋은 추천 결과를 가지는지 모르는 사용자도 있을 것이다. 따라서 저자는 사용자가 자신이 속한 그룹의 필터버블을 벗어날 수 있도록 coarse-grained 수준의 제어령도 설계했다. 

예를 들어, 중년 사용자는 추천리스트가 '연령=30세'로 제한되는 것을 원하지 않는다고 가정하자. $𝑃(𝑅 \vert 𝐷, 𝑑𝑜(𝐶))$에서 제어령은 $do(C = c_ u(-\overline{x},\alpha))$로 표현된다. 이로써 기존에 자신의 사용자 그룹에 제시되었던 feature $\overline{x}$을 줄임으로써 필터버블을 완화할 수 있다. 


#### 4.2.2 아이템 특성 제어(Item-feature Controls)
사용자 기능 제어는 사용자 특성과 관련된 필터버블을 해결하지만 사용자 상호 작용의 영향은 고려하지 않는다. 사용자 특성 제어를 보완하기 위해 아이템 특성 제어를 도입시키면서 추천 목록을 조정할 수 있다. 이러한 제어령을 사용하면 카테고리(예: 액션 영화, 로맨스 영화) 등 아이템 특성을 고려하여 추천시스템을 설정할 수 있게 된다.

M개의 아이템 특성과 아이템$i$는 $h _{i} = [ h^1 _{i}, \ldots, h^m _{i}, \ldots,  h^M _{i} ], \text{ where }  h^m _{i} \in \{0,1 \}$ 로 나타내며, 여기서 $h^m _{i} \in \{0,1 \}$ 는 아이템$i$가 아이템 특성 $h^m$ 을 가지고 있는지를 나타낸다.

**Fine-grained controls**
Fine-grained controls에서는 사용자가 특정 아이템 카테고리의 추천을 늘리도록 허용할 수 있다. 예를 들어 로맨스 영화와 같은 특정 카테고리의 아이템을 더 많이 받을 수 있게끔 구체적인 제어령을 내릴 수 있다. $do(C=c_i(+\hat{h},\beta))$로 표현되며, $\hat{h}$는 타겟 아이템 카테고리이며, $β ∈ [ 0 , 1 ]$는 사용자 제어 강도를 조정하는 계수이다.

**Coarse-grained controls**
 Coarse-grained controls는 사용자가 타겟 카테고리를 지정해야 하는 부담을 줄이기 위해 제안됐다. 이는 사용자의 과거 상호작용에서 가장 큰 비중을 차지했던 아이템 카테고리의 추천을 줄이도록 진행된다. 제어령은 $do(C=c_i(-\overline{h},\beta))$처럼 표현된다.


종합적으로 보면,  UCRS의 Fine-grained controls는 사용자가 세분화된 사용자 혹은 아이템 특성을 활용하여 구체적인 제어 명령을 통해 특정 추천 목표를 달성하는 방안이고,  Coarse-grained controls는 세분화된 특성에 대해서 사용자가 구체적인 제어 명령을 지시할 필요 없이 간단하게 필터버블을 완화하는 방법이다.


### 4.3 반 사실적(Counterfactual) 응답 메커니즘
Fine-grained controls나 Coarse-grained controls에서는 연령 혹은 카테고리 등 변경된 features를 기반으로 추천리스트를 새롭게 생성하게 된다. 여기서 기존 features에 대해 사실과 다른 질문에 답하는 추론 과정이 포함된다. 이게 무슨 뜻인지 아래 내용에서 더 자세히 설명하겠다.

#### 4.3.1 Response to User-feature Controls
앞서 User-feature Controls의 Fine-grained controls에서 $do(C=c_ u(+\hat{x},\alpha))$ 제어령을 사용하는 것을 확인했다. 이에 따라, UCRS는 변경된 $\hat{x}$를 기반으로 추천을 생성해야 한다. 예를 들어, 30대 중년이 10대가 좋아하는 영화 리스트를 보고 싶은 경우, 인과적 관점에서 볼 때, 제어령의 목표는 반 사실적인 질문에 답하는 것이다. 즉, "사용자가 $\hat{x}$에 속한다면 사용자의 추천은 어떻게 될 것인가?"라는 질문에 답하는 것이라고 생각하면 된다. 마찬가지로, Coarse-grained controls는 "사용자가 실제 그룹 $\overline{x}$에 속하지 않는 경우 사용자의 추천은 어떻게 될 것인가?"라는 질문에 답하는 것이다. 이런 반 사실적 질문에 답하기 위해 UCI 프레임워크는 사용자 features과 추천 간의 인과적 연관성을 식별하고 반 사실적 추론을 수행해야 한다.

![causal_graph](https://i.ibb.co/Sx0GdrJ/causal-graph.png)

**Causal view of generating recommendations**
Figure 4는 추천 생성 과정을 나타낸다. 대부분의 모델에서 추천시스템은 사용자 ID, 나이, 성별 등 상호작용을 통해 사용자 표현을 학습한다. 즉, 사용자 $u$와 항목 $i$에 대한 표현은 사용자가 한 아이템을 선호할 확률$Y_{u,i}\in [0,1]$를 예측하는 데 사용된다.  $Y_{u,i}$는 각 features와 상응하는 그룹들의 선호도를 반영한다.

반 사실적 추론에 답하려면 Fine-grained controls에서는 특정 $\hat{x}$를 변경하고, Coarse-grained controls에서는 특정 $\overline{x}$를 제거하는 게 직관적인 판단일 것이다. 그러나 Figure 4에서 볼 수 있듯이 추천시스템 학습 과정에는 features 간 상호작용 또한 이루어진다. 예를 들어, 나이를 30세에서 18세로 변경하거나 아예 제거하더라도, 'User ID'는 여전히 과거 features 간의 상호작용을 인코딩하므로 사용자가 새로 원하는 추천을 방해하게 된다.

이런 방해를 제거하기 위해 기존 연구에서는 confounder balancing, back-door/front-door adjustments이 널리 사용됐다. confounder balancing과 back-door adjustments를 사용하려면 방해 요소가 표현에 미치는 인과적 효과를 추정해야 하는데 이런 추정은 거의 불가능하다. 왜냐하면 (1) 사용자 상호작용은 시간 지남에 따라 새로운 상호작용이 지속적으로 추가되는 동적인 고차원 공간에서 이뤄지며, (2) 사용자 상호작용이 표현에 미치는 영향은 추천시스템의 학습 과정에 따라 결정되고, 학습 접근 방식에 따라 결과도 달라지기 때문이다. 또한 front-door adjustments 조정을 위해서는 모든 back-door 경로를 차단하는 매개체를 밝혀내야 하는데, 이는 Figure 4의 인과 그래프에는 적용할 수 없다. 이러한 문제를 해결하기 위해 저자는 추론 과정에서 'User ID' 표현이 예측 결과 $Y_{u,i}$에 미치는 영향을 직접적으로 줄이는 방법을 제안했으며, 이를 통해 학습 과정을 알지 못해도 features가 가지고 있는 과거 정보의 영향을 효과적으로 줄일 수 있게 됐다.

**Implementation of counterfactual inference**
UCI 프레임워크는 먼저 사실과 반대되는 추론을 사용하여 'User ID' 표현의 효과를 추정하고, 원래 예측 $Y_{u,i}$에서 해당 효과를 뺀 다음 추정을 재개한다. 즉, 사용자 $u$가 ID 표현이 없는 경우를 가정한 $Y_{\hat{u},i}$를 예측하는 것이다. $Y_{u,i}-Y_{\hat{u},i}$를 사용하여 'User ID'의 효과를 측정할 수 있으며 $α$ 계수를 사용하여 강도를 조절할 수 있다.

![uci](https://i.ibb.co/QFJSc4J/uci.png)


#### 4.3.2 Response to Item-feature Control
Item-feature Control에서 Fine-grained controls는 아이템 카테고리$\hat{h}$의 수를 늘리는 것을 목표로 하고, Coarse-grained controls는 최다 카테고리 $\overline{h}$를 줄이는 것을 목표로 한다. 사실 이러한 과정 중에는 이하 2가지 질문이 내포되어 있다: (1) 사용자가 더 많은 특정 카테고리 $\hat{h}$를 원한다면 어떻게 될 것인가? (2) 사용자가 최다 카테고리 $\overline{h}$를 원하지 않으면 어떻게 될 것인가? 이러한 질문에 답하기 위해 UCI 프레임워크는 사용자가 추천 순서(ranking)을 제어할 수 있도록 아래와 같이 설계했다.

![ranking](https://i.ibb.co/0XZhPPg/ranking.png)

여기서 $Y_{u,i}$는 수정된 순서 점수이고 $β ∈ [ 0 , 1 ]$는 제어 강도를 조정하는 계수이다. $r(i)$는 아이템 $i$의 정규화 항을 나타낸다.

![r(i)](https://i.ibb.co/94WHy7y/r.png)

Fine-grained controls에서 $r(i)$는 더 많은 타겟 카테고리 $\hat{h}$를 추천하도록 권장하고, Coarse-grained controls를 적용하는 경우 최다 카테고리 $\overline{h}$를 줄이도록 조정이 된다.

**Target category prediction**
현실에서는 아이템 카테고리의 수가 많기 때문에 Fine-grained controls에서 타겟 카테고리를 선별하는 것은 사용자에게 시간적 부담이 될 수 있다. Coarse-grained controls를 통해 이러한 부담을 일부 해소할 수 있지만, 만약 Fine-grained controls에서 사용자가 선호할 수 있는 타겟 카테고리를 예측할 수만 있다면 사용자 부담도 줄고 더욱 정확한 결과를 생성할 수 있다. 사용자가 추천리스트에서 최대 카테고리를 줄이려는 경우, 해당 사용자가 선호할 가능성이 높은 아이템 카테고리를 예측한 다음, Fine-grained controls를 통해 제어를 개선할 수 있다.

![target](https://i.ibb.co/Vmp4KgW/target.png)

Figure 5에서 볼 수 있듯이, 사용자의 상호작용 아이템을 시간에 따라 분류한 다음, 상호작용 시퀀스를 두 부분으로 나누어 아이템 카테고리의 분포를 별도로 구할 수 있다. 그런 다음 다층 퍼셉트론(MLP)을 사용하여 첫 번째 부분을 기반으로 두 번째 부분의 분포를 예측한다. 훈련 중에 MLP는 모든 사용자의 카테고리 분포를 사용하여 (1) 시간적 관심사 변화(예: 일부 카테고리에 대한 선호도가 점차 증가)와 (2) 아이템 카테고리 간의 관계(예: 액션 영화를 좋아하는 사용자는 범죄 영화도 좋아할 수 있음)를 포착할 수 있다. 추론 단계에서는 분포의 두 번째 부분을 사용하여 상위 K개의 타겟 카테고리를 예측하게 된다. 또한, $do(\overline{h}=0)$를 사용하여 카테고리 $\overline{h}$를 축소시킨다. 이로써, 상위 K개의 아이템 카테고리는 Fine-grained controls의 최종 추천리스트로 간주되게 된다. 궁극적으로 UCI는 타켓 카테고리 예측을 사용하여 Fine-grained controls를 더욱 향상시킬 수 있게 된다.

## **5. Experiment**[](https://dsailatkaist.github.io/template.html#4-experiment)
### **Research question**
- **RQ1. UCI는 4가지 수준의 사용자 제어를 통해 어떻게 필터버블을 제거하고 추천을 조정하는가?**
- **RQ2. 사용자가 추천을 제어할 때 계수(예: α 및 β)는 결과에 어떻게 영향을 미치는가?**
- **RQ3. 제안된 사실과 반대되는 추론이 추천에 어떤 영향을 미치는?**

### **Experiment setup**[](https://dsailatkaist.github.io/template.html#experiment-setup)

 **Dataset**
 - DIGIX-Video
 - ML-1M
 - Amazon-Book
 ![data](https://i.ibb.co/VQ7cnkc/data.png)

- DIGIX-Video2
- 는 2021 DIGIX AI Challenge에서 공개된 비디오 추천 데이터셋이다. 이 데이터셋에는 사용자와 아이템의 다양한 특성이 포함되어 있으며, 연령, 성별, 아이템 카테고리 등이 포함되어 있다. 
- ML-1M은 널리 사용되는 영화 데이터셋으로, 각 영화에는 여러 장르 카테고리가 포함되어 있다.
- Amazon-Book은 책 추천을 위해 만들어진 데이터셋으로, 사용자는 ID feature만 가지고 있으며 각 책 카테고리에 대한 레이블은 계층적인 topology로 이루어져 있다. 따라서 저자는 데이터 분석에서는 최상위 카테고리만 유지함으로써 각 책은 하나의 카테고리에만 할당되게 지정하였다. 

또한 저자는 각 데이터셋에 대해 10-core 설정을 사용하며, 점수가 4 이상인 데이터들만 양성 샘플로 처리했다. 타임스탬프로 상호작용을 정렬한 후 상호작용의 80%, 10%, 10%를 각각 훈련, 검증 및 테스트 세트로 사용했다.


**Baseline**
모든 baseline과 제안된 UCI는 모델에 독립적이며, Factorization Machine(FM)과  Neural Factorization Machine(NFM) 2가지 추천 모델를 기준으로 비교되었다.

##### **Evaluation of user-feature controls**
- woUF는 user feature 없이 모델을 학습하므로 추천 학습 중에 서로 다른 사용자 그룹 간의 분리를 완화함
-  changeUF는 훈련된 추천 모델을 활용하고 user feature만을 변경하여 추론 대상 $\hat{x}$에 적용한다. changeUF는 fine-grained user-feature controls에서 사용됨
-   maskUF는 추론 시 기존 사용자 특성 $\overline{x}$ (예: 나이=30)을 제거하여 coarse-grained user-feature controls에서 사용됨
-   Fairco는 공정한 노출 기회를 구현하는 랭킹 알고리즘
-   Diversity는 추천리스트를 다양화하기 위해 목록 내 유사성을 최소화하는 재랭킹 알고리즘

##### **Evaluation of item-feature controls**
- woIF는 item feature 없이 모델을 학습함
- Fairco(상동)
- Diversity(상동)
- Reranking은 UCI의 한 가지 변형으로, 𝑌′𝑢,𝑖 = 𝑌𝑢,𝑖 + 𝛽 · 𝑟(𝑖)에서 랭킹 알고리즘만 사용하며 반 사실 추론과 타켓 카테고리 예측은 사용되지 않음
- C-UCI: 'Coarse-grained controls' UCI 포로토타입
- F-UCI: 'Fined-grained controls' UCI 포로토타입

**Evaluation Metric**
##### **Evaluation of user-feature controls**
- Recall: 정확도 척도
- NDCG: 정확도 척도
- Coverage: 다양성 척도
- Isolation Index: 격리지수 척도
- DIS-EUC: 사용자와 그룹의 추천리스트 간의 거리 비교 척도
$\overline{x}$와 $\hat{x}$ 은 각각 사용자 $u$의 기존 그룹과 타겟 그룹을 나타냄; $d_{u} \in R^{M}$은 사용자 $𝑢$의 추천 아이템 카테고리에 대한 분포임; $\overline{g_u} ∈ R^{M}$는 원본 그룹 $\overline{x}$(예: 30세 사용자)의 사용자들의 평균 카테고리 분포를 나타내며;$\hat{g_u} ∈ R^{M}$은 대상 그룹 𝑥ˆ의 동일한 분포를 나타냅니다. 이후, 우리는 사용자 𝑢에 대해 $DIS-EUC = dis(𝑑_𝑢, \hat{g_𝑢}) - dis(𝑑_𝑢, \overline{𝑔_u})$를 계산합니다. 여기서 dis(·)는 유클리드 거리를 사용함. DIS-EUC는 사용자가 두 그룹 사이의 거리 차이를 측정하는데, 더 큰 거리는 더 심각한 그룹 분리와 필터버블을 나타냄.

##### **Evaluation of item-feature controls**
- Recall: 정확도 척도
- NDCG: 정확도 척도
- Coverage: 다양성 척도
- MCD: 최다 카테고리 비율 척도
- Weighted-NDCG: 카테고리 선호도 우선순위 척도
- TCD(Target Category Domination): 목표 카테고리 비율 척도




### **Result**[](https://dsailatkaist.github.io/template.html#result)
![rs1](https://i.ibb.co/DGLCcZ2/rs1.png)


![rs2](https://i.ibb.co/kD7nSjt/rs2.png)

![rs3](https://i.ibb.co/WpSs466/rs3.png)

Table 2와 Table 3는각각 Fine-grained controls과 Coarse-grained controls 결과이다. 

- baseline (즉, woUF, changeUF 및 maskUF)은 추천 정확도를 약간 감소시키고 격리 지수 및 DIS-EUC 측면에서 그룹 분리를 완화시켰다. 한편, 대부분의 경우 다양성이 약간 상승했다. 그러나 전반적인 성능은 FM 또는 NFM과 매우 유사했다. 즉, 사용자 특성이 훈련에서 제거되거나 추론을 위해 변경/제거되더라도, 사용자 ID 표현은 여전히 과거 상호 작용을 인코딩하며 이는 사용자의 원래 특성에 영향을 받아 FM 및 NFM과 유사한 추천을 이끌어낸다. 
- 공정성 및 다양성 방법은 필터버블 문제를 효과적으로 완화하고 추천 목록을 다양화할 수 있다. 그러나 이들은 성능을 급격히 감소시켰다. 예를 들어, Table 3의 NDCG에 따른 FM의 Fairco의 정확도는 15.38% 감소했다. 이는 공정성 및 다양성의 목표를 추구함에 따라 필연적으로 관련 없는 항목을 많이 추천하게 되어 긍정적인 항목의 기회를 차지하기 때문이다.
- UCI는 필터 버블에서 그룹 분리를 유의미하게 완화시키면서 뛰어난 정확도를 달성했다. 또한, 다양성도 FM 및 NFM과 비교하여 증가하는데, 이는 정확성과 다양성 사이의 딜레마를 완화했다. 저자는 이러한 개선을 반 사실추론의 효과적인 작용에 기인한다고 보고 있다. 이는 오래된 사용자 ID 표현의 영향을 줄이는 데 효과적이며, 이로 인해 추천 모델이 과거 상호 작용과 유사한 항목을 덜 추천하고 추천을 더 다양하게 만들었다. 동시에, 우수한 정확도는 사용자 제어 𝑐𝑢 (·, 𝛼)의 𝛼가 사용자 ID 및 다른 그룹 특성 (예: 연령 및 성별)의 표현에 대한 영향을 조절하여 개인 선호도와 그룹 선호도 사이의 균형을 더 잘 조정하기 때문입니다. Figure 4에 나와 있는 것과 같다.


## **6. Conclusion**[](https://dsailatkaist.github.io/template.html#5-conclusion)
-   저자는 사용자가 필터버블 완화를 능동적으로 제어할 수 있도록 하는 UCRS (User-Controllable Recommender System) 를 제안하여 사용자 특성 및 과거 상호작용을 기반으로 유사한 항목을 과도하게 추천하는 문제를 해결하는 시도를 보였다.
-   UCRS 프로토타입은 필터버블의 심각도를 감지하고 사용자에게 4가지 제어 명령을 제공하여 사용자로 하여금 추천시스템의 제어권을 능동적으로 실시할 수 있도록 추천시스템 생태계에 방향을 제시했다.
-   세 가지 데이터셋을 대상으로 한 실험을 통해 UCRS 프로토타입과 UCI 프레임워크는 정확성과 다양성 측면에서 유망한 성과를 보였으며 사용자 만족도와 추천 생태계에 대한 참여도를 높일 수 있을 것으로 기대된다.
-  단, 동 연구는 UCRS 프로토타입과 UCI 프레임워크를 평가할 때 온라인 실시간 데이터 대신 오프라인 데이터셋을 사용했고, 일부 사용자가 필터버블을 완화하고 제어 기능을 제공할 의향이 있다고 가정을 했기 때문에 현실 데이터와 비교하면 strong assumption일 것으로 판단된다.

----------

## **Author Information**[](https://dsailatkaist.github.io/template.html#author-information)

-   Minkyung Choi
    -   Affiliation:
 [Human Factors and Ergonomics Lab – Human Factors and Ergonomics Lab (HFEL) (kaist.ac.kr)](http://hfel.kaist.ac.kr/)
 
    -   Research Topic:
    Data Science, Computer Vision, VR

## **Reference & Additional materials**[](https://dsailatkaist.github.io/template.html#6-reference--additional-materials)

-   Github Implementation:
https://github.com/WenjieWWJ/UCR

-   Reference:
Wang, W., Feng, F., Nie, L., & Chua, T. S. (2022, July). User-controllable recommendation against filter bubbles. In _Proceedings of the 45th International ACM SIGIR Conference on Research and Development in Information Retrieval_ (pp. 1251-1261).
