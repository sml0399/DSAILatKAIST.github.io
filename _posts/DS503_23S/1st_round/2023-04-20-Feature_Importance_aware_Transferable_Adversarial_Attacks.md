---
title:  "[ICCV 2021] Feature Importance-aware Transferable Adversarial Attacks"
permalink: Feature_Importance_aware_Transferable_Adversarial_Attacks.html
tags: [reviews]
use_math: true
usemathjax: true
---

# **Feature Importance-aware Transferable Adversarial Attacks** 

## **0. Backgrounds**

Adversarial attack(적대적 공격)은 딥러닝 모델을 속이기 위해 설계된 공격 기술로, 사용자가 특정 입력 이미지에 대해 모델이 잘못된 결과를 출력하도록 유도하는 기술이다.
아래 이미지는 적대적 공격의 예시를 보여준다:

<p align="center">
    <img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*qdxy0rNUncghp-7GAog36w.png" alt>
</p>
<p align="center">
    <em>적대적 공격의 예시 (출처: Medium)</em>
</p>

위의 그림에서, 사람이 판다를 나타내는 사진에 noise를 주입함으로써, 사람의 눈으로는 변화를 알아챌 수 없지만 딥러닝 모델은 gibbon (긴팔원숭이)로 오인식하게 하는 적대적 공격의 예시를 확인할 수 있다.

이러한 적대적 공격의 가장 기본적인 형태는 공격하고자 하는 모델 (target model)을 두고, 해당 모델에 대해 적대적 이미지 (adversarial image)의 분류 손실함수 (classification loss)를 높이는 방향으로 이루어져 있다. 아래 식에서 그 형태를 확인할 수 있다:

$$
x^{adv}_0 = x, x^{adv}_n = Clip_{x, \epsilon}{x^{adv}_{n-1} + \nabla_x(J(x^{adv}_{n-1}, y))}
$$.

여기서 $x$, $x^{adv}$는 각각 공격받지 않은 기존 이미지와 적대적 이미지를 나타내고, $J(\cdot, \cdot)$은 cross-entropy loss를 나타낸다. 
즉, 매번 iteration마다 손실함수 값을 높이는 방향으로 적대적 이미지 $x^{adv}$가 업데이트 되는 것이다.

## **1. Problem Definition**  

본 논문은 이러한 적대적 공격 중에서도 전이 기반 공격 (transfer-based attack) 문제를 해결하고자 한다.
이 방법은 특정 모델에 대한 공격이 성공했을 때, 이를 다른 모델에 전송하여 공격을 유지하는 방법이다.
실세계에선 수많은 분류 모델이 존재하고, 공격하고자 하는 타겟 분류 모델 (*i.e.*, target model)의 정보 (*e.g.*, 모델의 구조, 파라미터, 등)가 알려져 있지 않은 상황도 존재한다.
그러한 상황에서 이미 알려져 있는 대리 모델 (surrogate model)을 속이는 적대적 이미지를 생성하여, 타겟 모델도 속이는 공격을 전이 기반 공격이라 한다.
이를 수식으로 나타내면 다음과 같다:

$\arg \max_ {x^{adv}} \mathcal{L}_ \phi(x^{adv}, t), \\
\text{s.t.} \parallel x - x^{adv} \parallel_p \le \epsilon, \\
\text{and} f_ \theta(x^{adv}) \neq t.\tag{1}$

즉, 대리 모델 $h_ \phi$와 정답 레이블 $t$ 대해서 임의의 손실함수 (*e.g.*, classification loss) $\mathcal{L}_ {\phi}$를 최대화하는 적대적 이미지 $x^{adv}$를 생성한다. 
이 때 $x^{adv}$는 기존 이미지 $x$와 육안으로는 다르지 않아야 한다라는 제한을 만족시키기 위해 적대적 섭동 (adversarial perturbation)의 크기를 $\ell_ p$-norm에서 $\epsilon$으로 제한한다.
이렇게 생성된 $x^{adv}$가 타겟 모델 $f_ \theta$의 예측을 망가뜨리는 것이 전이 기반 공격의 목적이다.

## **2. Motivation**  

대리 모델을 속이도록 생성된 적대적 이미지는 대리 모델에 과적합 되어 공격 전이성이 떨어지는 경우가 많다.
기존의 전이 기반 공격은 이러한 과적합을 방지하여 전이성을 높이려는 방향으로 연구되어 왔다.
본 논문 또한 이러한 추세를 따라 대리 모델에서의 과적합을 방지하려고 한다.

본 논문은 다수의 모델이 다른 구조와 파라미터 값을 가지고 있다고 해도, 이미지를 표현하는데 중요하게 작용하는 특징점 (feature)는 모두 비슷할 것이라는 점에서 착안한다.
따라서, 모델의 output logit (Eq. 1)을 망가뜨리는 것이 아니라, 모델이 학습한 특징점을 망가뜨린다.
즉, 대리 모델에서 특징점을 망가뜨린 적대적 이미지는, 다수의 모델이 비슷한 특징점을 학습하기 때문에 타겟 모델에서도 특징점을 망가뜨리는 것이다.

또한, 그러한 특징점 중에서도 이미지의 주요 물체에 특정한 특징점을 망가뜨리는 기법을 제안한다. 
주요 물체에 특정한 특징점이 다수의 모델이 비슷하게 학습하는 특징점일 것이라는 점에서 착안한 기법이다.
즉, 특징점의 중요도 (importance)를 측정해서, 중요한 특징점을 중심으로 적대적 공격을 수행한다.
그렇게 해서 본 논문의 제목인 Feature Importance-aware한 적대적 공격이 만들어지는 것이다.

## **3. Method**  

<p align="center">
    <img src="https://user-images.githubusercontent.com/48055164/232194584-60ae1390-9f6b-40f4-b96c-60ae0df3ddc8.png" alt>
</p>
<p align="center">
    <em>본 논문에서 제안한 Feature Importance-Aware Attack (FIA)의 framework</em>
</p>

위의 그림은 본 논문에서 제안한 Feature Importance-Aware Attack (FIA)의 전체적인 framework를 보여준다.
대리 모델에서 이미지 feature를 추출하고, 각 특징점이 얼마나 중요한지를 측정하여 (Sec. 3.1) 각 feature에 중요도를 가한다. 
그 다음, 중요한 (feature importance가 높은) positive feature를 억제하고, 중요하지 않은 (feature importance가 낮은) negative feature를 촉진한다 (Sec 3.2).
모델이 적대적 이미지로부터 중요한 특징점을 추출하지 못하고, 따라서 옳은 예측을 하지 못하게 하는 것이다.

### **3.1. Feature Importance by Aggregate Gradient** 
본 논문에서는 feature importance를 측정하기 위해 aggregate gradient라는 개념을 제안한다.
True label $t$에 대해 logit $l(x, t)$을 구하고, 그에 대해 k번째 층까지의 gradient를 구함으로써 해당 층에서 추출된 feature $f_k(x)$의 중요도를 측정할 수 있다:
$$
\Delta^x_k = \frac{\partial l(x, t)}{\partial f_k(x)} \tag{2}.
$$

하지만 이런 gradient에도 물체에 관련된 정보 외에 noisy한 정보가 있을 수 있다. 
따라서, 이미지에 random pixel dropping을 가하여 다수의 이미지를 만들고, 각 이미지에 대한 gradient를 합산하여 aggregate gradient를 계산한다. 
Pixel dropping은 이미지 픽셀에 랜덤하게 마스크를 가하는 것으로 구현하였다.
Aggregate gradient를 계산하는 과정은 아래 그림에서도 볼 수 있다.

<p align="center">
    <img src="https://user-images.githubusercontent.com/48055164/232198730-350d3038-2023-4afb-947b-ebcdf33be422.png" alt>
</p>

또한, 하나의 이미지에서 생성된 gradient와 비교하여 aggregate gradient의 우수성은 아래 그림에서 확인할 수 있다.

<p align="center">
    <img src="https://user-images.githubusercontent.com/48055164/232198907-5a9e5c8a-3108-46dd-8f59-147c4b908e87.png" alt>
</p>
Raw gradient는 주요 물체 외의 영역도 나타내는 반면, aggregate gradient는 주요 물체 부분만 나타내는 것을 확인할 수 있다.


### **3.2. Attack Algorithm** 
이렇게 추출한 aggregate gradient를 이용하여 적대적 이미지를 생성하게 된다.
아래의 손실함수를 이용하여 Eq. 1의 문제를 풀게 된다:
$$
\mathcal{L}(x^{adv}) = \sum(\Delta \odot f_k(x^{adv})), \tag{3} \\
\arg \min_{x^{adv}} \mathcal{L}(x^{adv}), \\
\text{s.t.} \parallel x - x^{adv} \parallel_p \le \epsilon.
$$
Eq. 2에서 추출한 aggregate gradient를 각 feature에 element-wise 곱을 통하여 중요한 feature의 activation 값을 높이고, $\mathcal{L}(x^{adv})$를 통하여 중요한 feature의 activation 값을 억제함으로써 모델이 중요한 물체에 대해 적절한 특징점을 추출하지 못하도록 한다.
이러한 적대적 공격의 알고리즘은 아래의 그림에 더 명확하게 명시되어 있다.

<p align="center">
    <img src="https://user-images.githubusercontent.com/48055164/232214609-522eead8-299c-42be-aff6-a277631c7e1f.png" alt>
</p>


## **4. Experiment**  

In this section, please write the overall experiment results.  
At first, write experiment setup that should be composed of contents.  

### **Experiment setup**  
* 본 논문이 제안한 FIA의 성능을 실험하기 위해 NIPS 2017 adversarial competition 데이터셋을 사용하였다. 이는 ImageNet과 비슷한 이미지로 이루어져있고, 클래스도 ImageNet-1k의 클래스를 사용한다.
* 다양한 대리 모델과 타겟 모델에 대해 실험을 진행하였다. Inception-V3, Inception-V4, Inception-ResNet-V2, ResNet-v1-50, ResNet-v1-152, VGG16, VGG19를 사용하였다. 
* FIA와 성능을 비교할 baseline 전이 기반 적대적 공격으로는 MIM, DIM, TIM, PIM을 사용하였고, 각 방법의 혼합인 TIDIM, PIDIM, PITIDIM와도 비교하였다. 추가로 NRDM, FDA 기법과도 결과를 비교하였다.
* Evaluation metric으로는 attack success rate (ASR)을 사용하였다. ASR은 적대적 이미지에 대한 타겟 모델의 오분류율로, top-1 classification accuracy를 1에서 뺀 값으로 계산하였다.  

### **4.1. Comparison of Transferability** 

<p align="center">
    <img src="https://user-images.githubusercontent.com/48055164/232217855-9e6a3b8c-14b6-4739-b2b8-e658357d2000.png" alt>
</p>
<p align="center">
    <em>Baseline 전이 기반 적대적 공격과의 attack success rate 비교.</em>
</p>

위의 표에서 본 논문이 제안한 FIA와 기존 baseline 적대적 공격과의 attack success rate를 비교를 확인할 수 있다.
맨 왼쪽의 열은 대리 모델을 나타내고, 맨 위쪽의 행은 타겟 모델을 나타낸다.
즉, 왼쪽의 모델과 오른쪽의 모델이 같으면 (*로 표시) 타겟 모델의 정보를 아는 상황에서 적대적 공격을 하는 시나리오인 것이다.
테이블에서 볼 수 있듯이, 기존 적대적 공격과 비교하여 FIA가 attack success rate, 즉 공격 전이율을 크게 향상시키는 것을 확인할 수 있다.
또한, FIA는 기존의 공격 framework에 쉽게 적용할 수 있고, 실제로 PIDIM에 FIA를 더한 FIA+PIDIM도 높은 ASR를 보이는 것을 확인할 수 있다.

### **4.2. Effects of Parameters in Aggregate Gradient**
<p align="center">
    <img src="https://user-images.githubusercontent.com/48055164/232282242-86fb72fc-c33c-43bc-9326-2f08d5852429.png" alt>
</p>
<p align="center">
    <em>Hyperparameter $p_d$와 ensemble number의 변화에 따른 ASR 변화 분석.</em>
</p>

위 그림의 그래프는 본 논문의 기법에서 사용된 hyperparameter인 $p_d$와 ensemble number의 변화에 따른 ASR 변화 분석을 보여준다.
$p_d$는 aggregate gradient을 계산하기 전 pixel dropping을 얼마나 할지를 결정하는 확률을 나타내고, ensemble number는 pixel dropping에 사용되는 마스크의 갯수를 나타낸다.
분석 그래프에 의하면, 각각의 parameter는 FIA의 성능에 큰 영향을 끼친다.
$p_d$의 값이 너무 높을 때는 이미지의 유의미한 정보가 손실이 될 수 있기 때문에 이미지의 주요 물체를 고려하는 적대적 이미지를 생성하기 어려운 현상을 보인다.
Ensemble number 또한 값이 높을수록 대체적으로 좋은 성능을 보이지만, 일정 값 이후로는 ASR이 saturate되는 현상을 보인다.

## **5. Conclusion**  

전이 기반 적대적 공격은 2018년부터 꾸준히 연구되어 왔다.
FIA는 feature의 중요도를 고려한 적대적 공격을 제안하여, 다수의 모델이 공통적으로 학습 및 표현하는 feature를 망가뜨리는 방식을 새로 제안하였다.
이러한 방식은 기존의 전이 기반 적대적 공격과 비교하여 공격 전이율을 크게 향상시켰다.

---  
## **Author Information**  

* 김우재
    * Affiliation: SGVR Lab, KAIST
    * Research topic: Adversarial Attack, Adversarial Defense

## **6. Reference & Additional materials**  

* [Github Implementation](https://github.com/hcguoO0/FIA)  
* [Paper](https://arxiv.org/abs/2107.14185) 