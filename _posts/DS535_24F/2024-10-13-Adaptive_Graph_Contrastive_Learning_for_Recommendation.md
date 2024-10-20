﻿---
title:  "[KDD-23] Adaptive Graph Contrastive Learning for Recommendation"
permalink: 2024-10-13-Adaptive_Graph_Contrastive_Learning_for_Recommendation.html
tags: [reviews]
use_ math: true
usemathjax: true
---

# **AdaGCL**
Yangqin Jiang / Adaptive Graph Contrastive Learning for Recommendation / KDD '23

  

## **1. Problem Definition**


추천 시스템에서 협업 필터링(collaborative filtering, CF)은 널리 사용되는 기법으로, 사용자의 과거 행동 및 평가 데이터를 바탕으로 유사한 사용자나 아이템을 추출하여 새로운 아이템을 추천하는 방식입니다. 최근에는 사용자-아이템 간의 상호작용을 이분 그래프로 표현하여 그래프 신경망(graph neural network, GNN) 기반의 CF 모델로 정보를 전달하는 방식이 급부상하고 있습니다. GNN 기반 모델은 사용자와 아이템의 표현을 더 정교하게 만들 수 있으며, 개인화된 추천에서도 좋은 성능을 보이기 때문입니다. 그러나 그래프 기반의 CF 모델에도 여전히 해결해야 할 문제가 남아있습니다.

**첫째, 실제 데이터는 다양한 요인으로 인해 노이즈가 발생할 수 있습니다**. 사용자가 관심 없는 제품을 클릭하거나, 인기 아이템을 과도하게 추천받아 무심코 조회할 때가 있습니다. 이러한 잘못된 상호작용 데이터를 모두 활용할 경우, 사용자의 진정한 관심사를 정확히 반영하지 못합니다.

**둘째, 추천 시스템의 데이터는 매우 희소하고 편향된 분포를 가집니다**. 대부분의 사용자는 극소수의 인기 있는 제품들과만 상호작용을 하고, 대부분의 아이템과는 거의 상호작용을 하지 않는 롱테일 분포를 따릅니다. 이러한 상황에서 양질의 학습 신호가 제한되면 모델은 사용자의 선호도를 충분히 반영할 수 없게 됩니다. 

요약하자면, 이 논문은 그래프 기반 협업 필터링 모델에서 발생하는 다음의 문제들을 해결하고자 합니다. 
- **문제 1. 데이터에 포함된 노이즈를 어떻게 제거할 수 있을까?**
- **문제 2. 데이터에 대한 양질의 학습 신호를 어떻게 생성할 수 있을까?**



## **2. Motivation**

앞서 언급한 데이터 희소성과 노이즈 문제를 해결하기 위해, 최근의 추천 시스템 연구에서는 **자기 지도 학습**(self-supervised learning, SSL)을 도입하고 있습니다. 
> SSL은 데이터에 대한 추가적인 라벨 없이도 학습 신호를 생성하여 모델을 훈련하는 방법입니다. 주로 원본 데이터를 변형하거나 여러 뷰를 생성해, 모델이 변형된 데이터로부터 스스로 유용한 특성을 학습하도록 유도합니다. 이는 특히 라벨이 부족하거나 없는 상황에서 효과적입니다.

최근 연구들은 SSL의 한 방식인 **대조 학습**(contrastive learning, CL)을 통하여 그래프 데이터로부터 양질의 학습 신호를 생성하려고 합니다.

> CL은 두 개의 서로 다른 데이터 뷰 사이의 유사성을 극대화하고, 상이한 뷰들 간의 차이를 극대화하는 방식으로 학습을 진행합니다. 본질적으로, 같은 노드에서 생성된 두 뷰는 유사해야 하며, 다른 노드에서 생성된 뷰는 서로 차이가 나야 한다는 원칙을 따릅니다. 이를 통해 모델이 데이터의 중요한 특징을 더 잘 학습하게 되고, 노이즈를 제거하고 희소한 데이터를 보완하는 데 효과적입니다.

이러한 방식은 주어진 데이터의 구조를 활용하여 의미 있는 표현을 학습하고, 노이즈와 희소성 문제를 해결하는 데 특히 유리합니다. 하지만 지금까지의 방식들은 학습 데이터을 충분한 고려하지 않고 CL을 적용하여 본질적인 문제를 해결하지 못했습니다.

예를 들어, SGL(Self-supervised Graph Learning)[^1] 모델은 사용자-아이템 상호작용 그래프에서 랜덤 노드 드롭아웃, 엣지 드롭아웃, 랜덤 워크 등의 기법을 사용하여 두 개의 뷰를 생성하고, 이를 통해 대조 학습을 수행합니다.

![image_ sample](https://i.postimg.cc/rpbHbTP0/sgl.png)
<p align="center"><em>Figure 1: SGL 모델의 전체 프레임워크</em></p>

이러한 접근은 주어진 그래프의 구조와 무관하게 무작위로 데이터 증강을 수행하기 때문에 다음과 같은 문제가 발생할 수 있습니다:
- **한계점 1. 유익한 신호의 손실**: 증강 과정이 주어진 데이터와 무관하게 이루어지기 때문에, 중요한 학습 신호가 의도치 않게 제거되거나, 반대로 노이즈가 그대로 포함될 수 있습니다.
- **한계점 2. 뷰의 표현력 제한**: 미리 정해진 방식대로만 뷰를 생성하기 때문에, 뷰의 표현력이 제한되어 대조 학습의 잠재력을 충분히 활용하지 못하게 됩니다.

이러한 문제를 극복하기 위해, 본 논문에서는 데이터의 특성에 맞게 적응적으로 뷰를 생성하는 **AdaGCL(Adaptive Graph Contrastive Learning)** 프레임워크를 제안합니다. AdaGCL은 다음과 같은 두 가지 접근을 통해 기존의 문제를 해결합니다:
- **해결 방안 1. 데이터 맞춤형 증강**: 데이터의 특성에 맞춰 증강을 적용해 노이즈를 효과적으로 제거하고, 학습 신호의 질을 향상시키자!
- **해결 방안 2. 학습 가능한 증강**: 뷰 생성 시 학습 가능한 변수를 도입하여 뷰의 표현력을 강화하자!




## **3. Method**

  ![image_ sample](https://i.postimg.cc/tg3zpHrm/adagcl-framework.png)
<p align="center"><em>Figure 2: AdaGCL 모델의 전체 프레임워크</em></p>

AdaGCL은 두 가지의 적응형 대조 뷰 생성기를 사용하여 협업 필터링 모델을 개선합니다. 첫 번째 생성기는 그래프 생성 모델을 사용하여 그래프 분포를 기반으로 뷰를 생성하고, 두 번째 생성기는 그래프 노이즈 제거 모델을 사용하여 사용자-아이템 그래프의 노이즈를 줄입니다. 이 두 개의 적응형 대조 뷰는 모델 학습에 필요한 추가적인 학습 신호를 제공합니다.

  
### 3.0 Notation
- 사용자 집합 $\mathcal{U} = \{u_  i\}_  {i=1}^I$, 아이템 집합 $\mathcal{V} = \{v_ j\}_ {j=1}^J$
-- $I$: 사용자 수, $J$: 아이템 수

- $\mathbf{R}\in\mathbb{R}^{I\times J}$: 상호작용을 나타내는 행렬 
-- implicit interaction setting에서 구매된 제품은 1, 아니면 0으로 표시

- $\mathcal{N}_ x$: $x$ 노드와 상호작용한 노드들의 집합
-- $\mathcal{N}_ x$는 상호작용 행렬 $\mathbf{R}$를 기반으로 정의됩니다 

- $\mathbf{E}^{(l)}\in\mathbb{R}^{(I+J)\times d}$: $l$번째 layer에 대한 사용자와 아이템의 임베딩 행렬
-- $d$: 임베딩의 차원


### 3.1 Local Collaborative Relation Learning

![image_ sample](https://i.postimg.cc/K8Q7TdL9/local-collaborative-relation-learning.png)
<p align="center"><em>Figure 3: Local Collaborative Relation Learning의 작동 방식</em></p>

Main task는 기존 interaction graph에서 GCN(graph convolutional network)을 적용하여 local collaborative relation이 반영된 각 노드의 임베딩을 얻고, 이를 바탕으로 사용자와 아이템 사이의 상호작용을 예측합니다. 이때 GCN에서의 임베딩 전파는 LightGCN[^2]과 유사한 방식으로 진행됩니다. (편의 상 논문의 표기가 아닌 LightGCN의 표기를 사용하였습니다.) 

>GCN은 노드의 이웃 정보를 활용하여 노드 임베딩을 계산하는 모델로, LightGCN은 비선형 활성화 함수와 추가적인 파라미터를 제거하여 더 간단하고 효율적인 방식으로 임베딩을 학습하는 모델입니다. LightGCN의 그래프 컨볼루션 연산은 다음과 같이 정의됩니다:
>
>$\mathbf{e}_ u^{(l+1)} = \sum_ {i\in\mathcal{N}_ u} \frac{1}{\sqrt{\vert\mathcal{N}_ u \vert\mathcal{N}_ i\vert}} \mathbf{e}_ i^{(l)} \\ 
\mathbf{e}_ i^{(l+1)} = \sum_ {u\in\mathcal{N}_ i} \frac{1}{\sqrt{\vert\mathcal{N}_ i\vert \vert\mathcal{N}_ u\vert}} \mathbf{e}_ u^{(l)}$

다만 본 논문에서는 레이어에 따라 self-aggregation을 적용한 방식으로 임베딩 전파를 수행합니다:

$\mathbf{e}_ u^{(l+1)} = \mathbf{e}_ u^{(l)} + \sum_ {i\in\mathcal{N}_ u} \frac{1}{\sqrt{\vert\mathcal{N}_ u\vert \vert\mathcal{N}_ i\vert}} \mathbf{e}_ i^{(l)} \\
\mathbf{e}_ i^{(l+1)} = \mathbf{e}_ i^{(l)} + \sum_ {u\in\mathcal{N}_ i} \frac{1}{\sqrt{\vert\mathcal{N}_ i\vert \vert\mathcal{N}_ u\vert}} \mathbf{e}_ u^{(l)}$

이를 행렬 형태로 나타내면, user-item 그래프의 인접 행렬(adjacency matrix)을

$\mathbf{A} = \begin{bmatrix} \mathbf{0} & \mathbf{R} \\ \mathbf{R}^\top & \mathbf{0} \end{bmatrix} \in \mathbb{R}^{(I+J) \times (I+J)}$

와 같이 나타낼 때, aggregation을 다음과 같이 표현할 수 있습니다:

$\mathbf{E}^{(l+1)} = (\mathbf{I} + \mathbf{D}^{-1/2} \mathbf{A} \mathbf{D}^{-1/2}) \mathbf{E}^{(l)}$

여기서, degree matrix $\mathbf{D}$는 $(I+J) \times (I+J)$ 대각 행렬로, $D_ {ii}$는 $i$번째 노드의 degree를 의미합니다. 마지막으로 노드의 임베딩을 계산할 때는 모든 레이어의 임베딩을 합산하며, 사용자 $i$와 아이템 $j$에 대한 선호도는 내적으로 예측합니다:

$\mathbf{E} = \sum_ {l=0}^L \mathbf{E}^{(l)}, \quad \hat{y}_ {i,j} = \mathbf{e}_ i^\top \mathbf{e}_ j$



### 3.2 Adaptive View Generators for Graph CL

#### 3.2.1 Dual-View GCL Paradigm

![image_ sample](https://i.postimg.cc/zv7DpZNB/dual-view-gcl.png)
<p align="center"><em>Figure 4: Dual-View GCL Paradigm</em></p>

본 논문에서는 두 가지의 데이터 맞춤형 view generator를 제안합니다. 이때 두 가지 view가 동일한 분포에서 생성될 경우, model collapse가 발생하여 CL의 효과가 감소할 수 있습니다. 
> Model collapse는 모델이 학습하는 동안 모든 노드가 매우 유사한 임베딩을 가지게 되어, 학습된 임베딩들이 구분되지 않는 상황을 의미합니다. 이는 모델이 데이터를 효과적으로 학습하지 못하는 결과를 초래할 수 있습니다.

이를 방지하기 위해, 그래프 분포를 기반으로 재구성하는 graph generative model, 그래프의 위상 정보를 활용해 노이즈를 제거하는 graph denoising model로 서로 다른 view generator를 사용합니다. Graph generative model은 기존의 그래프 구조를 바탕으로 새로운 view를 생성하며, 노드 간의 관계를 보존하려는 목적을 가지고 있습니다. 반면, graph denoising model은 그래프에서 발생할 수 있는 노이즈를 식별하여 이를 제거하는 과정을 통해 깨끗한 뷰를 제공합니다.

기존의 자기 지도 협업 필터링 방식과 마찬가지로, 동일한 노드에서 생성된 두 개의 다른 뷰는 positive pair로, 다른 노드에서 생성된 뷰들은 negative pair들로 간주합니다. 

- **Positive pair**: $(\mathbf{e}'_ i, \mathbf{e}''_ i)$ for $u_ i \in \mathcal{U}$
- **Negative pairs**: $\{(\mathbf{e}'_ i, \mathbf{e}''_ {i'}) : u_ i, u_ {i'} \in \mathcal{U}, u_ i \neq u_ {i'}\}$

Positive pair은 임베딩을 가깝게 하여 similarity를 높게 하고, negative pair들은 임베딩을 멀어지게 하여 similarity를 낮게 하면 모델이 각 노드의 고유한 특징을 잘 학습할 수 있게 됩니다. Loss는 CL에 많이 쓰이는 InfoNCE(information noise-contrastive estimation) loss을 사용합니다:

$\mathcal{L}_ {\mathrm{ssl}}^{\mathrm{user}} = \sum_ {u_ i \in \mathcal{U}} -\log \frac{\exp(s(\mathbf{e}_ {i'}, \mathbf{e}_ {i''}) / \tau)}{\sum_ {u_ {i'} \in \mathcal{U}} \exp(s(\mathbf{e}_ {i'}, \mathbf{e}_ {i'}'') / \tau)}$

여기서 $s(\cdot)$는 cosine similarity를 의미하고, $\tau$는 softmax의 *temperature*로 불리는 hyperparameter입니다. 아이템에 대해서도 비슷한 방식으로 $\mathcal{L}_ {\mathrm{ssl}}^{\mathrm{item}}$을 계산하고, 이 두 손실을 합하여 SSL의 목표 함수를 다음과 같이 정의합니다:

$\mathcal{L}_ {\mathrm{ssl}} = \mathcal{L}_ {\mathrm{ssl}}^{\mathrm{user}} + \mathcal{L}_ {\mathrm{ssl}}^{\mathrm{item}}$





#### 3.2.2 Graph Generative Model

![image_ sample](https://i.postimg.cc/mr2w6bBf/graph-generative-model.png)
<p align="center"><em>Figure 5: Graph Generative Model의 작동 방식</em></p>

Graph generative model로는 VGAE[^3](variational graph auto-encoder)를 사용합니다.

> VGAE는 Variational Auto-Encoder(VAE)와 Graph Neural Network(GNN)를 결합한 모델로, 그래프에서 노드 간의 상호작용 정보를 기반으로 노드 임베딩을 학습합니다. VGAE는 그래프의 구조를 반영하여 각 노드의 임베딩을 생성하고, 이를 통해 그래프의 잠재 구조를 추론하거나 새로운 노드 간의 연결을 예측합니다.

1. $L$개의 layer를 가지는 GCN을 사용하여 임베딩을 얻습니다.

2. 두 개의 MLP(multi-layer perceptron)를 통해 각각 평균과 표준편차를 계산합니다. 
3. 또 다른 MLP를 디코더로 사용해서 새로운 그래프를 생성합니다. 
4. VGAE의 Loss는  노드 임베딩과 표준 Gaussian 분포 간의 KL(Kullback-Leibler) divergence, 기존 그래프와 새로 생성된 그래프 간의 차이를 측정하는 cross entropy loss의 합으로 설정합니다:
$\mathcal{L}_ {\mathrm{gen}} = \beta\mathcal{L}_ {\mathrm{kl}} + \mathcal{L}_ {\mathrm{dis}}.$
-- $\beta$는 KL divergence의 중요도를 조절하는 파라미터로, 논문에는 $\beta$가 명시되어 있지 않지만 코드에서는 $\beta=0.1$로 설정하였습니다.

> 수식으로는 아래와 같이 나타낼 수 있습니다.
> 1. Interaction graph $\mathbf{R}$이 주어질 때, GCN을 사용하여 임베딩을 다음과 같이 구합니다:
> $\mathbf{E}=\mathrm{GCN}(\mathbf{R}, \mathbf{E}^{(0)})$
> 
> 2. 임베딩을 기반으로 평균과 표준편차 행렬을 MLP를 사용하여 계산합니다:
> $\boldsymbol{\mu}=\mathrm{MLP}_ {\boldsymbol{\mu}}(\mathbf{E})\in\mathbb{R}^{(I+J)\times d},\quad \log \boldsymbol{\sigma}=\mathrm{MLP}_ {\boldsymbol{\sigma}} (\mathbf{E})\in\mathbb{R}^{(I+J)\times d}$
> 
> 3. Reparametrization trick으로 잠재 벡터 $\mathbf{z}_ i$를 샘플링합니다:
> $\mathbf{z}_ i=\boldsymbol{\mu}_ i+\boldsymbol{\sigma}_ i\odot \boldsymbol{\epsilon}\in\mathbb{R}^d, \quad \boldsymbol{\epsilon} \sim \mathcal{N}(\mathbf{0},\mathbf{I})$
>  이를 바탕으로 노드 간의 연결 여부를 MLP로 예측합니다:
>  $\hat{R}_ {ij}=\mathrm{sigmoid}(\mathrm{MLP}_ {\mathrm{decoder}}(\mathbf{z}_ i \odot \mathbf{z}_ j))$
>  4. Loss는 다음과 같이 계산합니다:
>  $\mathcal{L}_ {\mathrm{kl}}=-\frac{1}{2}\sum_ {i=1}^N\sum_ {k=1}^d (1+\log {\sigma}_ {i,k}^2-\mu_ {i,k}^2-\sigma_ {i,k}^2)$
>  $\mathcal{L}_ {\mathrm{dis}}=-\sum_ {(i, j)\in\mathcal{E}} \left( R_ {ij}\log \hat{R}_ {ij} + (1-R_ {ij})\log(1-\hat{R}_ {ij}) \right)$






#### 3.2.3 Graph Denoising Model
![item_ sample](https://i.postimg.cc/5225QNBh/adagcl-denoising.png)
<p align="center"><em>Figure 6: Graph Denoising Model의 작동 방식</em></p>

Graph denoising model은 그래프의 노이즈를 제거하는 과정으로, 이를 통해 더 정제된 임베딩을 학습할 수 있도록 도와줍니다. 이 모델은 GCN의 레이어마다 입력 그래프에 대한 필터링을 수행합니다. 

1. 이진 행렬 $\mathbf{M}^{(l)}\in\{0,1\}^{\vert\mathcal{V}\vert\times\vert\mathcal{V}\vert}$을 사용해 noisy edge를 마스킹합니다:
-- $\mathbf{R}^{(l)}=\mathbf{R}\odot \mathbf{M}^{(l)}$
-- 모든 edge가 아닌 *noisy* edge를 제거해야 하기 때문에, 제거된 엣지의 개수에 대한 패널티를 부여합니다:
$\mathcal{L}_ {\mathrm{penalty}}=\sum_ {l=1}^L \lVert \mathbf{M}^{(l)}\rVert_ 0 = \sum_ {l=1}^L \sum_ {(i,j)\in\mathcal{E}} \mathbf{I}[m_ {i,j}^{(l)} \neq 0]$
-- 그러나 L0 norm은 non-differentiable하며, 계산적으로 다루기 어렵기 때문에 reparametrization trick을 사용합니다. 

2. 이진 행렬의 원소 $m_ {i,j}^{(l)}$가 베르누이 분포를 따른다고 가정합니다: 
-- $m_ {i,j}^{(l)} \sim \mathrm{Bernoulli}(\pi_ {i,j}^{(l)})$
-- 하지만 이산적인 베르누이 분포는 미분할 수 없기 때문에, 이를 연속적으로 근사하는 Hard Concrete Distribution을 사용하여 학습이 가능하도록 만듭니다.
>  Github 코드를 기반으로 수식을 전개해보면 다음과 같습니다.
> 1. $\pi_ {i,j}^{(l)}$은 $\mathbf{e}_ {i}^{(l)}, \mathbf{e}_ {j}^{(l)}$에 MLP를 통과한 값으로 설정합니다:
> $\pi_ {i,j}^{(l)}=\mathrm{MLP}_ {\mathrm{att}}([\mathrm{MLP}_ {\mathrm{nb}}(\mathbf{e}_ {i}^{(l)})\parallel \mathrm{MLP}_ {\mathrm{self}}(\mathbf{e}_ {j}^{(l)})])$
> -- $\mathrm{MLP}_ {\mathrm{nb}}, \mathrm{MLP}_ {\mathrm{self}}:\mathbb{R}^{d}\rightarrow \mathbb{R}^{d}$는 ReLU activation의 1-layer이며, $\mathrm{MLP}_ {\mathrm{att}}: \mathbb{R}^{2d}\rightarrow \mathbb{R}$는 one linear layer입니다.
> 
> 2. Uniform random noise를 샘플링합니다:
> $\epsilon\sim\mathcal{U}(0,1)$
> 
> 3. 위 noise로부터 Gumbel-Max trick으로 Gumbel noise로 변환합니다:
> $\tilde{\epsilon}=\log \epsilon - \log(1-\epsilon), \quad s_ {i,j}^{(l)}={\tilde{\epsilon}+ \pi_ {i,j}^{(l)}}$
> 
> 4. Sigmoid 함수로 값의 범위를 $[0,1]$로 압축합니다:
> $\tilde{s}_ {i,j}^{(l)}=\sigma(s_ {i,j}^{(l)})=1/(1+e^{-s_ {i,j}^{(l)}})$
>
> 5. 해당 값을 stretch하여 값의 범위를 $[\gamma, \zeta]$로 변환합니다:
> $t_ {i,j}^{(l)}=\tilde{s}_ {i,j}^{(l)}\times(\zeta-\gamma)+\gamma$
> -- $\gamma$와 $\zeta$는 Hard Concrete Distribution에서 값을 stretch할 때 사용되는 하이퍼파라미터입니다. 기본값은 $-0.45, 1.05$입니다.
> 
> 6. $[0,1]$ 범위를 넘지 않도록 clamp합니다.
>  $m_ {i,j}^{(l)}=\min(\max(t_ {i,j}^{(l)}, 0), 1)$ 

3. Reparametrization trick을 이용하여 페널티를 다음과 같이 재정의하여, noisy edge가 적절하게 필터링되도록 합니다:
-- $\mathcal{L}_ c = \sum_ {l=1}^L \sum_ {(u_ i, v_ j) \in \mathcal{E}} (1 - P_ {\sigma(s_ {i,j}^{(l)})}(0 \vert \theta^{(l)}))$
-- 여기서 $s_ {i,j}^{(l)}$는 noise를 고려한 score이며, $P_ {\sigma(s_ {i,j}^{(l)})}$는 $\sigma(s_ {i,j}^{(l)})$에 대한 누적 분포 함수를 나타냅니다.
> -- 위 식에서의 $\sigma$는 sigmoid와 stretching을 포함합니다.
> -- Github 코드를 참고할 때, $\mathcal{L}_ c$는 다음과 같이 계산됩니다:
> 
> $\mathcal{L}_ c=\sum_ {l=1}^{L} \sum_ {(u_ i, v_ j)\in\mathcal{E}} \frac{1}{\lvert \mathcal{E} \rvert} \sigma\left( \pi_ {i,j}^{(l)}- \log \left(\frac{-\gamma}{\zeta}\right) \right)$
>
> -- 여기서의 $\sigma(\cdot)$은 sigmoid 함수입니다.





### 3.3 Learning Task-aware View Generators

두 가지의 생성된 뷰를 주요 CF 태스크에 맞게 조정하기 위해서, 추천 task에서 널리 사용되는 BPR(Bayesian Personalized Ranking) 손실을 도입합니다: $$ \mathcal{L}_ {\mathrm{bpr}} = \sum_ {(u,i,j) \in O} -\log\sigma(\hat{y}_ {ui} - \hat{y}_ {uj}), $$ 여기서 훈련 데이터 $O = \{(u, i, j) \mid (u, i) \in O^+, (u, j) \in O^-\}$는 관찰된 상호작용 $O^+$와 관찰되지 않은 상호작용 $O^- = (U \times I )/ O^+$ 의 쌍을 나타냅니다.  $\hat{y}_ {ui}, \hat{y}_ {uj}$는 사용자 $u$와 관찰된 아이템 $i$, 관찰되지 않은 아이템 $j$ 사이의 예측된 상호작용 점수이며, BPR 손실은 $\hat{y}_ {ui}$와 $\hat{y}_ {uj}$의 차이를 최대화하여 관찰된 상호작용이 더 높은 순위를 갖도록 유도합니다.

Graph generative model은 노드 간의 관계를 복원하는 역할을 하며, 이를 통해 얻은 임베딩에 BPR 손실을 적용하여 예측된 선호도 차이를 학습합니다: 

$\mathcal{L}_ {\mathrm{gen}} = \mathcal{L}_ {\mathrm{kl}} + \mathcal{L}_ {\mathrm{dis}} + \mathcal{L}_ {\mathrm{bpr}}^{\mathrm{gen}} + \lambda_ 2 \\vert\Theta\\vert_ F^2$

 여기서 $\Theta$는 모델 파라미터 집합이며, $\lambda_ 2$는 weight decay로 정규화하는 hyperparameter입니다. 

Graph denoising model은 noisy edge를 제거한 그래프에서 임베딩을 생성한 후, 동일하게 BPR 손실을 적용하여 학습됩니다: 

$\mathcal{L}_ {\mathrm{den}} = \mathcal{L}_ c + \mathcal{L}_ {\mathrm{bpr}}^{\mathrm{den}} + \lambda_ 2 \\vert\Theta\\vert_ F^2$

이처럼 두 가지 view generator는 각각 Graph generative model과 Graph denoising model로 학습된 임베딩에 BPR 손실을 적용하여, 노이즈를 제거하고 더 정교한 추천을 수행할 수 있습니다.

### 3.4 Model Training

모델 학습은 두 단계로 나뉘며, 상위 수준 학습은 전체 추천 시스템의 성능을 개선하는 데 중점을 둡니다. 이 단계에서는 기존 추천 작업에 대한 BPR loss와 자기 지도 학습 작업에 대한 SSL loss를 공동으로 최적화합니다:

$\mathcal{L}_ {\text{upper}} = \mathcal{L}_ {\text{BPR}} + \lambda_ 1 \mathcal{L}_ {\text{ssl}} + \lambda_ 2 \vert\vert\Theta\vert\vert^2_ F$

여기서 $\Theta$는 LightGCN의 사용자 및 아이템 임베딩을 포함한 매개변수 집합을 의미하고, $\lambda_ 1$과 $\lambda_ 2$는 각각 자기 지도 학습과 L2 정규화의 강도를 조절하는 hyperparameter입니다. 

하위 수준 학습에서는 graph generative model, graph denoising model들을 최적화하는 작업이 포함되며, 이는 다음과 같이 표현됩니다:

$\mathcal{L}_ {\text{lower}} = \mathcal{L}_ {\text{gen}} + \mathcal{L}_ {\text{den}}$

이들은 서로 상호 보완적으로 작동합니다. 상위 수준 학습에서는 전체 추천 성능을 개선하는 데 집중하고, 하위 수준 학습에서는 각 view generator의 성능을 개별적으로 최적화하여, 두 학습 과정이 함께 모델의 최종 성능을 극대화합니다.

### 3.5 Time Complexity


제안된 모델의 시간 복잡도는 다음과 같이 분석할 수 있습니다.

1. **Local collaborative relation learning module**: $O(L \times \vert\mathcal{E}\vert \times d)$
-- $L$은 그래프 신경망 레이어의 수, $\vert\mathcal{E}\vert$는 사용자-아이템 상호작용 그래프의 엣지 수, $d$는 임베딩 차원입니다.
-- $L$개의 레이어를 사용하여 각 엣지에 대해 임베딩을 계산하는 과정.

3. **Graph generative model (VGAE)**: $O(\vert\mathcal{E}\vert \times d^2)$
-- 각 엣지에 대해 임베딩을 계산하고 MLP를 거쳐 그래프를 재구성하는 과정.

4.  **Graph denoising model**: $O(L \times \vert\mathcal{E}\vert \times d^2)$
-- 각 레이어마다 MLP로 노이즈를 찾아내고, 엣지마다 연산을 수행하는 과정.

5. **Contrastive learning**:  $O(L \times B \times (I + J) \times d)$
 -- $B$는 단일 배치에 포함된 사용자/아이템의 수, $I$와 $J$는 각각 사용자와 아이템의 수를 나타냅니다.
 -- 배치에 포함된 사용자 및 아이템($B$)에 대해 각 임베딩을 비교하는 연산이 필요하며, 각 노드가 $L$개의 레이어를 거쳐 임베딩을 계산하는 과정.

  

## **4. Experiment**

 
### **4.1 Experiment setup**

![image_ sample](https://i.postimg.cc/XvV01jRg/data-statistics.png)
<p align="center"><em>Figure 7: 실험 데이터셋들에 대한 정보</em></p>

* **데이터셋**: LastFM, Yelp, BeerAdvocate 데이터셋에서 평가되었으며, 자세한 정보는 위의 표에 명시되어 있습니다. 
-- LastFM은 음악 추천 데이터셋으로, 사용자와 음악 간의 상호작용을 포함합니다. 
-- Yelp는 음식점 리뷰 데이터셋으로 사용자와 음식점 간의 상호작용을 포함합니다.
-- BeerAdvocate는 맥주 리뷰 데이터셋으로, 사용자와 다양한 맥주 간의 상호작용을 포함합니다.
* **실험 세팅**: 각 데이터는 7:2:1의 비율로 training, validation, testing set으로 나뉘며, test set 내에서 사용자의 상호작용이 있었던 아이템과 없었던 아이템을 모두 포함한 리스트를 생성한 후, 모델이 각 아이템에 대한 선호도를 예측하고 그에 따라 랭킹을 매깁니다. 이 랭킹을 기반으로 추천 성능을 평가합니다.
* **평가 지표**: Recall@N, NDCG@N, N=20에 대한 성능 지표를 구했습니다. 

> **Recall**은 사용자가 실제로 상호작용한 아이템들 중에서, 모델이 추천한 상위 N개의 아이템에 포함된 비율을 측정하는 지표입니다. Recall@N은 다음과 같이 정의됩니다: 
> 
> $\text{Recall@N} = \frac{\text{Number of relevant items in top N recommendations}}{\text{Total number of relevant items for the user}}$
> 
>  즉, 추천 시스템이 실제로 사용자가 상호작용한 아이템들을 얼마나 잘 포함시켰는지를 평가합니다.
>  
> **NDCG**(Normalized Discounted Cumulative Gain)는 추천된 아이템의 순서를 고려하여 추천 리스트의 품질을 평가하는 지표입니다. 
> * **DCG**(Discounted Cumulative Gain)는 추천 리스트에서 순위에 따라 가중치를 부여한 누적 이득을 계산합니다: 
> 
> $\text{DCG@N} = \sum_ {i=1}^{N} \frac{rel_ i}{\log_ 2(i+1)}$
> 
>  여기서, $rel_ i$는 아이템 $i$의 관련성을 나타냅니다. 상위에 있는 아이템일수록 높은 가중치를 받습니다. 
>  
> * **IDCG**(Ideal DCG)는 아이템들이 이상적으로 순위가 매겨졌을 때의 DCG입니다. 
> 
> * **NDCG**는 DCG를 IDCG로 나누어, 0과 1 사이로 정규화한 값입니다: 
> 
> $\text{NDCG@N} = \frac{\text{DCG@N}}{\text{IDCG@N}}$ 
> 

* **대조군 설정**: 총 15개의 모델과 비교 실험을 진행했으며,  간략한 설명을 아래에 적어놓았습니다.

-- **BiasMF**: 사용자와 아이템에 대한 편향을 반영한 행렬 분해 방법.
-- **NCF**: 신경망으로 행렬 분해를 대체하여 복잡한 상호작용을 학습하는 방법.
-- **AutoR**: auto-encoder를 사용하여 사용자/아이템 표현을 개선한 방법.
-- **GCMC**: GCN을 사용하여 상호작용 행렬을 완성한 방법.
-- **PinSage**: 그래프 합성곱 기반의 무작위 샘플링 기법을 사용한 방법.
-- **NGCF**: 상호작용 그래프를 통해 정보를 전파하고 잠재 표현을 학습한 방법.
-- **STGCN**: 그래프 합성곱 인코더와 그래프 오토인코더를 결합한 방법.
-- **LightGCN**: 비선형 변환 없이 간단한 graph convolution을 활용한 방법.
-- **GCCF**: GCN에서 비선형 활성화를 제거한 방법.
-- **HCCF**: Hypergraph 신경망과 cross-view 대조 학습 아키텍처를 사용하여 지역 및 전역 협업 관계를 학습한 방법.
-- **SHT**: Hypergraph 신경망과 transformer를 통합하여 상호작용의 잡음을 줄인 방법.
-- **SLRec**: 노드 특징 간의 대조 학습을 정규화 항으로 사용한 방법.
-- **SGL**: 랜덤 노드 드롭아웃, 엣지 드롭아웃, 랜덤 워크으로 대조 학습을 수행한 방법.
-- **NCL**: 잠재적인 이웃을 대조 쌍으로 포함시켜 대조 학습을 강화한 방법.
-- **DirectAU**: 초구면 상에서 정렬과 균일성을 최적화한 방법.


### **4.2 Overall Performance Comparison**

![image_ sample](https://i.postimg.cc/pL1WYx7h/performance-comparison.png)
<p align="center"><em>Figure 8: LastFM, Yelp, BeerAdvocate 데이터셋에 대한 성능 비교</em></p>

본 실험에서는 대조군들과 AdaGCL의 주요 성능을 비교하였고, Recall@20과 NDCG@20에서 각각 3개의 데이터셋에 대해 평균적으로 5~10%의 성능 향상을 보였습니다. AdaGCL은 두 개의 *적응형* view generator를 사용해 하나는 그래프 노이즈를 제거하고, 다른 하나는 그래프 구조를 복원하여 다양한 정보를 유지합니다. 이를 통해 model collapse를 방지하고, 서로 다른 관점에서 생성된 임베딩을 결합함으로써 더 정교한 추천을 가능하게 합니다.



### **4.3 Model Ablation Test**
![image_ sample](https://i.postimg.cc/4dpXgc3V/ablation-study.png)
<p align="center"><em>Figure 9: AdaGCL의 구성 요소들에 대한 Ablation study</em></p>

본 실험에서는 제안된 AdaGCL 모델의 구성 요소들이 성능에 얼마나 중요한 역할을 하는지 확인하기 위해 수행되었습니다. 각 요소를 제거하거나 대체했을 때 성능이 어떻게 변하는지 평가하였습니다. Ablation test는 세 가지 주요 실험 설정을 기반으로 진행되었습니다. 첫 번째로, BPR 손실을 제거한 **w/o Task** 실험에서 성능이 크게 저하되었습니다. 두 번째로, denoising view generator를 제거한 **Gen+Gen** 설정에서는 model collapse의 문제로 성능이 감소하였습니다. 마지막으로, 무작위 증강을 사용한 **EdgeD** 실험에서는 대조 뷰의 정보 손실로 인해 성능 저하가 발생했습니다.

- **w/o Task**: $\mathcal{L}_ {\mathrm{gen}} = \mathcal{L}_ {\mathrm{kl}} + \mathcal{L}_ {\mathrm{dis}} + \cancel{\mathcal{L}_ {\mathrm{bpr}}^{\mathrm{gen}}} + \lambda_ 2 \\vert\Theta\\vert_ F^2, \mathcal{L}_ {\mathrm{den}} = \mathcal{L}_ c + \cancel{\mathcal{L}_ {\mathrm{bpr}}^{\mathrm{den}}} + \lambda_ 2 \\vert\Theta\\vert_ F^2$

BPR 손실이 없는 경우, view generator 학습이 적절히 안내되지 않기 때문에 추천 시스템에서 더 관련성 높은 사용자-아이템 상호작용 패턴을 포착하는 데 실패합니다. 이는 auto-encoding loss와 잡음 제거 loss만으로는 충분하지 않음을 의미합니다.

- **Gen+Gen**: denoising view generator $\rightarrow$ generative view generator

두 개의 generative view generator만을 사용할 경우, 동일한 관점에서 대조 학습이 이루어져 model collapse의 위험이 커지고, 대조 뷰의 다양성이 부족해집니다.

- **EdgeD**: generative view generator, denoising view generator $\rightarrow$ random edge drop view generator

-  무작위 edge drop으로 생성한 대조 뷰는 임의로 증강된 데이터로 인해 중요한 패턴이 보존되지 않아, 성능이 저하됩니다. 반면, generative와 denoising view generator의 결합은 더 나은 학습 신호를 제공합니다.

Ablation test를 통해 AdaGCL의 구성 요소들, 특히 BPR 손실과 두 가지 상이한 view generator의 결합이 성능에 필수적임을 확인할 수 있습니다.


### **4.4 Model Robustness Test**

본 실험의 목적은 AdaGCL이 다양한 데이터 잡음과 희소성 조건에서 얼마나 견고하게 작동하는지를 평가하는 것입니다. 이를 위해, 잡음 비율과 데이터 희소성에 따라 모델의 성능 변화를 살펴보았습니다.

#### 4.4.1 Performance w.r.t. Data Noise Degree

![image_ sample](https://i.postimg.cc/kMcdvqRJ/noise-ratio.png)
<p align="center"><em>Figure 10: 잡음 비율에 따른 상대적인 성능 변화</em></p>

- **실험 설정**: 실제 엣지의 일정 비율(5%, 10%, 15%, 20%, 25%)을 무작위로 가짜 엣지로 대체한 그래프를 입력으로 사용하여 모델을 재훈련했습니다.  AdaGCL의 성능을 LightGCN 및 SGL과 비교했으며, 잡음의 비율에 따른 *상대적인* 성능 변화를 비교했습니다.
- **실험 결과**: 실험 결과, AdaGCL은 잡음 비율이 25%로 증가했을 때에도 Recall@20이 평균 2% 이하로만 감소하여, LightGCN의 8%, SGL의 10% 성능 감소와 비교했을 때 더 우수한 견고성을 보였습니다. LightGCN과 SGL은 노이즈가 증가할수록 성능이 급격히 하락했지만, AdaGCL은 적응형 대조 뷰 생성기가 노이즈를 효과적으로 처리함으로써 성능 하락 폭이 훨씬 작았습니다. 이는 AdaGCL의 대조 학습이 노이즈에 더 강한 학습 신호를 제공함을 시사합니다.

#### 4.4.2 Performance w.r.t. Data Sparsity

![image_ sample](https://i.postimg.cc/76kpSWxJ/sparsity-degree.png)
<p align="center"><em>Figure 11: 사용자, 아이템의 희소성 정도에 따른 성능 변화</em></p>

- **실험 설정**: 사용자와 아이템들을 훈련 데이터에서의 상호작용 수를 기준으로 나누었으며, 사용자 측과 아이템 측에서 그룹을 나누는 기준은 위 그림에 명시된 바와 같습니다. 이 실험 역시 LightGCN 및 SGL 모델들과 비교했습니다.

- **실험 결과**: AdaGCL은 다양한 희소성 정도를 가진 데이터셋에서 일관된 우수한 성능을 나타내며, 이는 사용자와 아이템 모두에 대한 희소한 데이터를 처리하는 데 있어 견고성을 나타냅니다. 이는 적응형 대조 뷰 쌍이 제공하는 높은 품질의 SSL 신호가 데이터 희소성의 부정적인 영향을 완화하기 때문이라 볼 수 있습니다.



위 실험들을 통해 AdaGCL이 데이터 잡음과 희소성이 있는 상황에서도 높은 견고성을 유지하며, 다른 대조군에 비해 데이터 품질이 낮은 환경에서도 우수한 성능을 발휘함을 확인할 수 있었습니다.

### **4.5 Hyperparameter Analysis**
본 실험에서는 AdaGCL이 대조 학습에 쓰이는 InfoNCE 손실에 대한 주요 하이퍼파라미터 $\lambda_ 1$에 얼마나 민감한지를 조사합니다. 이 값이 클수록 대조 학습이 더 강조되며, 작을수록 추천 작업에 대한 영향을 덜 받습니다.

![image_ sample](https://i.postimg.cc/j54Bc45g/hyperparameter-analysis.png)
<p align="center"><em>Figure 12: Hyperparameter에 따른 성능 변화</em></p>

- **실험 설정**: $\lambda_ 1$가 {1, 1e-1, 1e-2, 1e-3, 1e-4} 의 값에서 성능이 어떻게 변화하는지 확인합니다.

- **실험 결과**: Yelp 데이터에서는 $\lambda_ 1=1$일 때, LastFM 데이터에서는 $\lambda_ 1=0.1$일 때 최고의 성능을 보이는 것을 알 수 있습니다. $\lambda_ 1$ 값이 너무 클 경우, 대조 학습이 추천 태스크보다 지나치게 강조되면서 사용자의 실제 선호도를 반영하지 못하는 결과를 초래할 수 있습니다. 이는 대조 뷰 간의 유사성 극대화에 집중하여 실제 추천 성능이 떨어지게 됩니다. 또한, Yelp 데이터는 상호작용이 상대적으로 많아 대조 학습의 비중을 더 크게 설정할 수 있었던 반면, LastFM 데이터는 상호작용이 적기 때문에 $\lambda_ 1$ 값을 적당히 낮춰 추천 작업을 더 강조하는 것이 성능을 높이는 데 유리했습니다.

결론적으로, AdaGCL은 $\lambda_ 1$ 값에 대해 일정한 민감성을 가지고 있으며, 적절한 $\lambda_ 1$ 값 설정을 통해 대조 학습과 추천 작업 간의 균형을 맞추는 것이 성능 향상에 중요합니다.

### **4.6 Embedding Visualization Analysis**

본 실험에서는 AdaGCL과 SGL의 임베딩을 시각화하여 모델의 이점을 분석했습니다.

![image_ sample](https://i.postimg.cc/SNv3gsSh/view-embedding-sgl.png)
<p align="center"><em>Figure 13: SGL에 대한 임베딩 시각화</em></p>

![image_ sample](https://i.postimg.cc/m2DK3MGj/view-embedding-adagcl.png)
<p align="center"><em>Figure 14: AdaGCL에 대한 임베딩 시각화</em></p>
  

- **실험 세팅**

 Yelp 데이터셋에서 2,000개의 노드를 무작위로 샘플링하여 세 가지 뷰(Main View, CL View 1, 2)의 임베딩을 t-SNE를 사용해 2차원 공간으로 맵핑했습니다. KMeans 알고리즘을 사용하여 압축된 2차원 임베딩을 기준으로 노드를 클러스터링하고, 각 클러스터에 색상을 부여했습니다. 
-- AdaGCL의 CL View 1, 2는 각각 그래프 생성 모델과 그래프 잡음 제거 모델에 의해 생성된 뷰입니다.
-- Noisy한 데이터(25%의 엣지를 가짜 엣지로 대체)에 대해서도 동일한 실험 세팅을 적용하여 임베딩을 시각화했습니다.

> **t-SNE**(t-distributed Stochastic Neighbor Embedding)는 고차원 데이터를 저차원 공간(주로 2차원이나 3차원)으로 시각화하기 위한 차원 축소 기법입니다. 이 방법은 데이터의 국소적인 구조를 잘 유지하면서 시각적으로 표현할 수 있어, 임베딩의 클러스터링 패턴을 확인하는 데 적합합니다.
> 
> **KMeans**는 데이터를 사전 지정된 K개의 클러스터로 나누는 군집화 알고리즘으로, 각 클러스터의 중심(centroid)을 기준으로 데이터를 분할합니다. 이를 통해 데이터의 중심을 파악하고, 유사한 데이터 포인트들이 어떻게 그룹화되는지를 확인할 수 있습니다.

- **실험 결과1. 일반 데이터에서의 비교**

SGL의 (a), (b), &#40;c&#41; 그림을 보면 클러스터의 임베딩들이 넓게 펴져 있으며, CL View 1, 2가 유사한 분포를 보입니다. SGL은 랜덤 엣지 드롭 방식을 사용하여 대조 뷰를 생성하지만, 이 방식은 데이터의 구조적 특징을 반영하지 않아 대조 뷰 간의 차이가 미미할 수 있습니다. 반면, AdaGCL의 (a), (b), &#40;c&#41; 그림에서는 보다 명확한 클러스터링 효과로 서로 구별되는 임베딩을 생성합니다. 이는 뷰를 적응적으로 조정함으로써 그래프의 복잡한 구조를 더 잘 포착하여, 더 명확하면서도 다양한 뷰를 생성할 수 있음을 알 수 있습니다.

- **실험 결과2. 잡음이 있는 데이터에서의 비교**

SGL의 (d), (e), (f) 그림은 과도한 균일 분포를 보이는 반면, AdaGCL의 (d), (e), (f)는 임베딩의 클러스터링 효과가 잘 드러나는 것을 볼 수 있습니다. 잡음이 있는 데이터에서 AdaGCL은 두 개의 적응형 뷰 생성기를 통해 노이즈 신호를 효과적으로 제거하고, 중요한 패턴을 유지하지만, SGL은 랜덤 증강만으로는 잡음을 걸러내지 못해 임베딩이 더 퍼지고, 균일한 분포를 보여 협업 패턴을 제대로 반영하지 못합니다.

결론적으로, AdaGCL의 적응형 대조 뷰 생성기는 임베딩에서 더 명확한 구조를 유지함으로써 잡음이 있는 데이터에서도 높은 성능을 발휘하며, 이는 추천 성능을 더욱 향상시키는 중요한 요소입니다.


## **5. Conclusion**

본 연구는 적응형 뷰 생성기를 활용하여 대조 학습 기반 추천 시스템의 성능을 개선하는 AdaGCL 모델을 제안했습니다. AdaGCL은 그래프 생성 모델과 그래프 노이즈 제거 모델을 결합하여 대조 뷰를 생성함으로써, 데이터의 노이즈 문제를 효과적으로 해결하고 더 정확한 사용자-아이템 상호작용 모델을 학습할 수 있도록 했습니다. 

다양한 데이터셋에서의 실험을 통해 AdaGCL은 기존의 대조군 모델들에 비해 유의미한 성능 향상을 보이며, 특히 데이터가 희소하거나 잡음이 많은 상황에서도 견고한 성능을 발휘함을 확인했습니다. AdaGCL은 대조 학습의 효과를 극대화하면서 노이즈를 처리하는 능력을 강화하여, 추천 시스템의 성능을 한층 더 끌어올리는 유망한 접근법임을 입증했습니다.

향후 연구에서는 인과 추론 기법을 AdaGCL에 통합하여 대조 학습에서의 자가 지도 신호를 더욱 해석 가능하게 하는 방향으로 확장할 수 있습니다. 이를 통해 사용자 행동의 근본적인 원인을 더 정확하게 파악하고, 개인화된 추천의 질을 더욱 높일 수 있을 것입니다.


## **Author Information**

**이천우**: 
- Contact: dlcjsdn07@kaist.ac.kr
  

## **6. Reference & Additional materials**

  

- Github Implementation: [AdaGCL Repository](https://github.com/HKUDS/AdaGCL)


[^1]:  Wu, Jiancan, et al. "Self-supervised graph learning for recommendation." Proceedings of the 44th international ACM SIGIR conference on research and development in information retrieval. 2021.
[^2]: He, Xiangnan, et al. "Lightgcn: Simplifying and powering graph convolution network for recommendation." Proceedings of the 43rd International ACM SIGIR conference on research and development in Information Retrieval. 2020.
[^3]: Kipf, Thomas N., and Max Welling. "Variational graph auto-encoders." arXiv preprint arXiv:1611.07308 (2016).
