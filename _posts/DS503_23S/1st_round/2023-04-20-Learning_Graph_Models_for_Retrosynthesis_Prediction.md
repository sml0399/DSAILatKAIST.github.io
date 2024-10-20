---
title:  "[NIPS 2021] Learning Graph Models for Retrosynthesis Prediction"
permalink: Learning_Graph_Models_for_Retrosynthesis_Prediction.html
tags: [reviews]
use_math: true
usemathjax: true
---

# <span style="font-weight:bold">Learning Graph Models for Retrosynthesis Analysis</span>

이 논문은 Graph Neural Network를 이용하여 분자 구조의 Retrosynthesis analysis를 수행하는 논문입니다.

해당 논문에 대해 더 궁금하신 분들은 다음 링크를 참조하세요: 
[논문](https://proceedings.neurips.cc/paper/2021/hash/4e2a6330465c8ffcaa696a5a16639176-Abstract.html) / 
[Github 코드](https://github.com/vsomnath/graphretro)

## <span style="font-weight:bold">1. Background</span>
이번 섹션에서는 본 논문을 읽는데 필요한 기본적인 배경 지식을 설명드리겠습니다.

### <span style="text-decoration:underline;font-weight:bold">Machine Learning for Molecular Sciences</span>
최근 Molecular sciences 분야에서 Machine Learning (ML)의 적용이 급격하게 늘어나고 있습니다.
기존에 분자 구조의 특성을 파악을 하기 위해서는 실험실에서 직접 실험을 하거나 DFT나 Molecular simulation 등의 Computational algorithm 을 기반으로 계산을 했어야 했습니다.
이런 두가지 방식 모두 값비싼 전문 인력과 긴 시간이 소요되어, 최근에는 ML 을 기반으로 기존의 지식을 근사하려는 시도들이 늘어나게 된 것입니다.

ML 모델을 통해 분자의 특성을 학습을 하기 위해서는 우선 분자 구조를 기계가 학습할 수 있는 형태로 변환을 시켜주는 것이 중요합니다.
기존에는 크게 두가지의 분자 구조 표현 방식이 있었는데요, SMILES string과 Molecular Fingerprint가 그것들 입니다.

<p align="center">
  <img width="60%" src="https://user-images.githubusercontent.com/40378824/231461299-9edab8c7-d45f-4bd6-b6b6-efbc1e6a4759.png">
</p>
<p align = "center">
Fig1. 분자구조의 두가지 표현 방식
</p>

Figure 1 (a)에서 보시는 바와 같이 SMILES string은 분자의 복잡한 표현 구조를 문자열 형태로 표현하게 됩니다.
반면, Figure 1 (b) 에서 보시는 바와 같이 Molecular Fingerprint는 분자 구조의 특성을 Multi-hot 형태로 나타내어, 분자에 어떤 형태의 구조들이 포함되어 있는지를 나타내게 됩니다.

하지만 이렇게 분자를 표현하는 방식은 전문가의 지식이 필요할 뿐만 아니라 표현하는 과정 중에서 분자 구조의 구조적인 특성을 모두 반영하지 못하여 정보의 손실이 잃어나는 문제점을 가지고 있습니다.
그렇다면 정보의 손실은 최소화 하면서 모두가 손쉽게 분자 구조를 표현할 수 있을까요?


### <span style="text-decoration:underline;font-weight:bold">Graph Neural Networks for Molecules</span>
최근에는 분자 구조를 그래프 형태로 표현하여 분자의 특성을 파악하는 연구들이 많이 수행되고 있습니다. 
Figure 2 에서 보여지듯이, 분자 구조는 아주 자연스럽게 그래프 구조로 표현이 가능합니다.
즉, 분자 구조를 이루는 원자와 결합을 각각 그래프의 노드와 엣지로 표현하는 것입니다.
최근에는 2D 형태의 분자 표현은 물론이고, 3D 형태의 분자 표현 역시 그래프 구조로 표현을 하여 모델을 학습을 해서,
분자 구조의 구조적인 특성을 놓치지 않고 잘 반영하려는 시도들이 이어지고 있습니다.

<p align="center">
  <img width="50%" src="https://user-images.githubusercontent.com/40378824/231464394-c41e274c-d3e9-47d4-950f-34ac8a581a44.png">
</p>
<p align = "center">
Fig2. 분자구조의 그래프 표현 방식
</p>

이렇게 그래프 구조로 표현이 된 분자 구조는 Graph Neural Network (GNN) 모델을 기반으로 학습을 하게 됩니다.
GNN은 기본적으로 엣지로 연결된 노드들이 서로 정보를 주고 받으면서 학습을 하게 되는 구조입니다.
따라서 분자 구조에서 서로 결합된 원자들이 서로 정보를 주고 받으면서 모델이 분자의 구조를 파악하게 되는 것입니다.

최근에는 많은 연구자들이 GNN을 기반으로 분자 구조의 성질을 예측하거나, 두 분자들이 반응하는 화학 반응을 예측하는 등의 Molecular sciences 분야의 문제를 해결하고 있습니다.
이 글에서는 GNN을 기반으로 Retrosynthesis analysis 문제를 해결하는 논문을 다루게 됩니다.

### <span style="text-decoration:underline;font-weight:bold">What is Retrosynthesis Analysis?</span>

Retrosynthesis analysis란 target 분자를 유기 합성 반응에 근거를 둔 추론 방법을 통하여 작용기를 변화시키거나 단계적으로 작은 분자로 분해하여 합성법을 설계하는 수단입니다.
Retrosynthesis analysis 는 Target 분자가 분해하는 방식에 관한 분석이지만, 역으로 target 분자를 합성하는 다양한 synthetic route를 설계하는데 사용할 수 있습니다.
따라서 Retrosynthesis analysis 를 통해 새로운 성질의 분자 구조를 합성하거나 동일한 성질을 갖는 분자 구조를 값싼 방식으로 합성하는 방식을 추론할 수도 있게 됩니다.

<p align="center">
  <img width="40%" src="https://user-images.githubusercontent.com/40378824/231476187-efe528e0-abf4-4587-a5d5-b6bab3dda915.png">
</p>
<p align = "center">
Fig3. Retrosynthesis 방식 예시
</p>

Figure 3 를 통해 간단한 Retrosynthesis analysis 과정을 예시로 들면,
우선 Target 분자 (e.g., Phenylacetic acid)가 주어졌을 때, 해당 분자에서 어떤 결합이 끊어질지 먼저 예측합니다.
결합이 끊어지면 두개 혹은 그 이상의 Synthon (e.g., Nucleophilic “-COOH”, Electrophilic “PhCH2+”)이 발생하게 되는데, 이러한 synthon들은 화학적으로 불안정해서 그대로 존재할 수 없을 것입니다.
따라서 Synthon들과 Synthetic equivalent 관계에 있는 분자들을 Reactant로 정의하게 됩니다.
즉, Synthon과 Synthetic equivalent관계에 있는 Reactant들을 합성하여 Target 분자가 생성될 수 있게 되는 것이죠.


## <span style="font-weight:bold">2. Previous Works</span>
최근 들어 이러한 Retrosynthesis analysis 를 ML 관점에서 풀어내는 연구들이 많이 등장했습니다. 이러한 논문들은 크게 1) Template-based approach, 2) Template Free-based approach, 3)  Semi-template-based approach 로 나누어 볼 수 있습니다.

<p align="center">
  <img width="70%" src="https://user-images.githubusercontent.com/40378824/231400620-7e57a13a-75bc-42cc-911c-584c2a0a3626.png">
</p>
<p align = "center">
Fig1. Retrosynthesis analysis 방법론의 3가지 분류
</p>


### <span style="text-decoration:underline;font-weight:bold">1) Template-based Approach</span>

Template 기반의 방법은 미리 정의된 화학 반응의 Template을 활용하는 방식입니다. 
화학 반응의 Template은 전문가들에 의해 직접 만들어지거나 큰 데이터 베이스에서 알고리즘을 기반으로 추출하게 됩니다.
이렇게 추출된 화학 반응의 Template과 우리가 알고 싶은 화학 반응을 일일이 비교하여 어떤 Template과 가장 잘 매칭이 되는지를 찾는 문제가 됩니다.

이러한 Template 기반의 방법은 정확도가 높고 해석 가능하며 기존의 화학 지식과 부합하는 결과를 도출할 수 있지만, 
기존의 화학 지식에서 벗어나지 못하기 때문에 새로운 화학 반응을 발견하는 등의 과학적인 발견은 하지 못하는 문제점을 가지고 있습니다.


### <span style="text-decoration:underline;font-weight:bold">2) Template-Free Approach</span>

Template-Free 기반의 방법은 주로 기계 번역 분야의 방법론에서 영감을 받아 고안이 되었습니다.
즉, SMILES로 나타내어진 Product 분자를 기계 번역 모델의 input으로 넣고, SMILES로 나타내어지는 Reactant 분자를 기계 번역 모델의 output으로 도출하게 됩니다.

이러한 Template-Free 기반의 방법은 Scratch로부터 생성되기 때문에 기존의 화학 지식을 넘어선 분자 구조를 생성할 수 있습니다. 하지만 한편으로는 화학적으로 불가능한 SMILES 형태가 생성되며 모델의 생성에 대한 설명이 불가능하다는 문제점 역시 가지고 있습니다. 


### <span style="text-decoration:underline;font-weight:bold;color:crimson">3) Semi-Template-based Approach</span>

Semi-Template 기반의 방식은 기존의 두 방식 (Template-based / Template-Free 기반)의 문제점을 해결하기 위해 제안되었습니다. 이 방식은 two-step 으로 진행이 되는데, 

1) <span style="text-decoration:underline">Edit Prediction</span>: Product 분자에서 어느 부분을 분해할지를 예측하고, 
2) <span style="text-decoration:underline">Synthon Completion (Reactant Generation)</span>: 그렇게 분해된 synthon을 기반으로 다시 분자를 생성
   
하여 Product 분자를 n개의 Reactant 분자로 나누게 됩니다.

<p align="center">
  <img width="60%" src="https://user-images.githubusercontent.com/40378824/231430217-62799abf-203b-4977-9bed-8b03da964d36.png">
</p>
<p align = "center">
Fig2. Semi-Template 기반의 two-step 방식. (a) Edit Prediction: Product 분자에서 어느 부분이 분해될지 예측함. (b) Synthon completion: 분해된 n개의 synthon 을 기반으로 분자를 생성함.
</p>

이러한 Semi-Template 기반의 방식은 기존의 화학분야에서 전문가들이 retrosynthesis를 하는 방식과 매우 유사하고, Template 기반의 방식에 비해서는 새로운 화학 지식을 얻을 수 있다는 장점이 있고, Template-Free 기반의 방식에 비해서는 화학적으로 가능한 형태의 분자를 생성할 수 있다는 장점이 있습니다.

최근에 이 글에서 리뷰하는 논문을 포함하여 Semi-Template 기반의 방법론 3가지가 소개되었습니다.
3가지 방법론은 모두 Edit Prediction 부분 보다는 Synthon Completion (Reactant Generation)에 초점이 맞추어져 있습니다. 그럼, 우선 두 논문에서의 Synthon completion 방식을 먼저 간단히 소개드리겠습니다.

<span style="font-weight:bold"> [2020 ICML] A Graph to Graphs Framework for Retrosynthesis Prediction </span>

해당 논문에서는 Variational graph translation 방식으로 reactant를 generation 하는 G2Gs framework를 제안합니다. Product 분자를 분해하여 n개의 Synthon이 잘려나왔을 때, autoregressive 한 방식으로 graph transformation action을 generation 하여 synthon에 적용하게 됩니다.

<p align="center">
  <img width="60%" src="https://user-images.githubusercontent.com/40378824/231434349-acbc9349-dda6-44db-9927-5a97f2b46329.png">
</p>
<p align = "center">
Fig3. G2Gs의 Variational Graph Translation 기반의 Reactant Generation.
</p>

G2Gs는 Decoder 는 Encoder 로 얻어진 Synthon S 와 Reactant G 가 합쳐진 representation z가 주어졌을 때, 다음 수식에 따라 학습이 됩니다.

$P(t\vert z, S)=P(a_ {1:T}\vert z, S)=\prod_ {i=1} ^{T} {P(a_ {i}\vert z, S ^{i-1})}$


Generator는 Autoregressive 하게 action 을 Generation 하면서 학습됩니다. 이때 action으로 3가지 종류가 있는데, 1. Termination Prediction, 2. Nodes Selection, 3. Edge Labeling 이 그것들 입니다.

$P(G\vert z, S)=\sum_ {t \in \mathcal{T}} {P(t\vert z, S)}$

Reactant $G$ 에 대한 확률은 최종적으로 Reactant G까지 갈 수 있는 모든 action 경로들의 합으로 표현되는데, 최종적으로 모델은 VAE와 동일하게 ELBO term으로 학습됩니다.

$\mathcal{L}_ {ELBO} = \mathbb{E}_ {z\sim q}[log P(G\vert z, S)] - KL[q(z\vert G,S)\vert\vert p(z\vert S)]$

이때 $q(z\vert G,S)$ 는 $\mu$와 $\sigma^ {2}$로 parametrized 된 확률 분포이고, $\mu$와 $\sigma^ {2}$ 는 reactant $G$ 와 synthon $S$로부터 학습되어 얻어집니다.

한 문장으로 요약을 하자면, 이 논문은 VAE 방식으로 graph transformation action을 생성하여 주어진 Synthon을 변환시켜서 Reactant를 만들어내는 것입니다.

<span style="font-weight:bold"> [2020 NIPS] RetroXpert: Decompose Retrosynthesis Prediction Like A Chemist </span>

해당 논문에서는 기계 번역 기반으로 Synthon으로부터 Reactant를 생성하는 RetroXpert framework를 제안합니다.
Product 분자를 분해하여 n 개의 Synthon이 잘려나왔을 때, 그 Synthon을 다시 SMILES 형태로 변환하고, 그 SMILES를 기반으로 새로운 SMILES를 생성하는 방식입니다.

<p align="center">
  <img width="40%" src="https://user-images.githubusercontent.com/40378824/231442288-2e72cec5-ad28-406e-98c2-4dced0b4363d.png">
</p>
<p align = "center">
Fig4. RetroXpert의 SMILES 기반의 Reactant Generation.
</p>

하지만 이 두 방식은 모두 생성 모델을 기반으로 구성이 되어 있기 때문에, Reactant의 생성이 복잡할 뿐만 아니라 화학적으로 불가능한 구조가 생성될 수 밖에 없습니다.


## <span style="font-weight:bold">3. Learning Graph Models for Retrosynthesis Prediction</span>

이제 본격적으로 Learning Graph Models for Retrosynthesis Prediction 논문에 대해서 설명드리겠습니다.
이 논문에서는 이전 두 개의 논문이 생성 모델을 기반으로 Reactant를 생성하기 때문에 문제점이 발생을 하는 것을 지적을 하면서, 
분류 모델을 기반으로 Reactant 를 Completion 하는 문제로 변환을 하여 GraphRetro라는 프레임워크를 제안합니다.
즉, Reactant의 후보군을 미리 정해놓고, Synthon을 기반으로 어떤 Reactant가 될 지에 대해 Prediction을 하는 방식으로 문제를 풀게 되는 것입니다.
그러면 Semi-Template-based 기반의 two-step 중 Edit prediction 방식을 먼저 자세하게 살펴보도록 하겠습니다.

### <span style="text-decoration:underline;font-weight:bold">1) Edit Prediction</span>

아까도 말씀드린 대로, Edit prediction은 Target 분자의 어느 부분을 분해할 것인지를 예측하는 것입니다.
이 논문에서는 분자 구조의 결합 구조 (bond)가 끊어질 확률을 scoring 하는 방식으로 학습을 하게 됩니다.

원자 $u$와 원자 $v$ 사이의 결합 $(u, v)$는 결합 구조 종류 $k$ 가 바뀌었는지 여부에 따라 $y_ {uvk} \in \{0, 1\}$ 로 레이블링이 되어 있습니다.
그리고 각각의 원자 $u$ 는 hydrogen 개수의 변동 여부에 따라 $y_ {u} \in \{0, 1\}$ 로 레이블링 되어 있습니다.
우리는 Edit prediction에서 이러한 $y$ 값들을 예측해야 합니다.

Edit Prediction 모델을 학습하기 위해서 우선 원자의 임베딩을 학습해야 합니다.
원자의 임베딩을 학습하기 위해서 GNN의 일종인 Message Passing Network (MPN)를 사용합니다.
원자의 임베딩은 아래와 같이 계산됩니다.

$\{ \mathbf{c}_ {u} \}=\text{MPN}(\mathcal{G}, \{ \mathbf{x}_ {u} \}, \{ \mathbf{x}_ {uv} \}_ {v \in \mathcal{N}(u)})$

$\mathcal{N}(u)$ 는 원자 $u$와 결합구조를 이루고 있는 원자들의 집합입니다.
이렇게 얻어진 원자의 임베딩을 기반으로 아래와 같이 원자와 결합의 edit score를 계산합니다.

$s_ {u}=\mathbf{u_ {a}}^ {T} \tau(\mathbf{W}_ {\mathbf{a}}\mathbf{c}_ {u} + b)$

$s_ {uvk}=\mathbf{u_ {k}}^ {T} \tau(\mathbf{W}_ {\mathbf{k}}\mathbf{c}_ {uv} + b_ {k})$

이때 $\tau$는 ReLU 활성 함수를 의미하고, 결합 $(u, v)$의 임베딩 $\mathbf{c}_ {uv}  = (\text{ABS}(\mathbf{c}_ {u}, \mathbf{c}_ {v}) \vert \vert \mathbf{c}_ {u}+ \mathbf{c}_ {v})$ 로 임베딩이 permutation에 invariant하도록 설계됩니다.

하지만 일반적으로 레이블들이 독립적인 classification 문제와는 달리 edit prediction 문제의 레이블들은 서로 독립적이지 않다는 특징을 가지고 있습니다.
예를 들어, 분자 구조에서 aromatic ring 구조는 안정되어 물질의 합성 과정에서도 크게 변하지 않을 가능성이 큽니다.
즉, Aromatic ring에 있는 bond들은 모두 함께 레이블이 0이여야 한다는 것입니다.
따라서 이러한 종속적인 관계를 score 에 투영시킬 필요가 있을 겁니다.

저자는 이를 위해서 결합 $(u, v)$를 노드로 하고 결합끼리 공유하는 원자를 엣지로 하는 그래프를 새로 만드는 것을 제안합니다.
새로 만들어진 그래프를 통해서, 결합들의 종속적인 관계를 아래 수식을 통해 LSTM 업데이트 방식과 유사한 방식으로 표현을 하게 됩니다.

$f_ {uvk} = \sigma(\mathbf{W_ {kx}^{f} x_ {uv}} + \mathbf{W_ {km}^{f} m_ {uv}})$

$i_ {uvk} = \sigma(\mathbf{W_ {kx}^{i} x_ {uv}} + \mathbf{W_ {km}^{i} m_ {uv}})$

$\tilde{m}_ {uvk} = \mathbf{u_ {m}}\tau(\mathbf{W_ {kx}^{m} x_ {uv}} + \mathbf{W_ {km}^{m} m_ {uv}})$

$\tilde{s}_ {uvk} = f_ {uvk} \cdot s_ {uvk}+ i_ {uvk} \cdot \tilde{m}_ {uvk}$

최종적으로 모델의 Edit prediction 모듈은 아래의 Loss로 학습됩니다

$\mathcal{L}_ {e} = -\sum_ {(\mathcal{G}_ {p}, E)}(\sum_{((u, v), k) \in E}y_ {uvk} \log{\tilde{s}_ {uvk}} +  \sum_{u \in E} y_ {u} \log{s_ {u}})$

이렇게 학습이 된 모델은 Test 단계에서 어느 부분이 잘려나갈지 예측을 하게 되고,
그 예측의 결과로 $C$개의 Synthon이 발생하게 됩니다.
그러면 이제 발생한 Synthon을 토대로 Reactant 분자를 만드는 부분을 소개드리겠습니다.

### <span style="text-decoration:underline;font-weight:bold">2) Synthon Completion</span>

발생한 Synthon들은 Leaving group이라고 불리는 원자 집합들과 결합하여 Reactant 분자가 될 수 있습니다.
반대로 이야기 하면 Reactant 분자들을 합성해서 Product 분자를 만들었을 때, Reactant 분자들에서 떨어져나간 부분을 Leaving Group이라고 부르게 되는 것 입니다.
저자들은 이러한 Leaving group의 집합을 미리 정의하고, Synthon이 주어졌을 때 가장 가능성이 높은 Leaving group을 선택하는 방식으로 Synthon Completion을 수행하게 됩니다.
이전의 생성 모델 기반의 방법론들은 새로운 분자 구조를 만들어내는 구조라면, 이 모델은 기존에 데이터셋에 존재할 가능성이 높은 분자 구조를 예측하는 방식이라고 볼 수 있을 것 같습니다.

Leaving group은 USPTO-50K 데이터셋을 기반으로 미리 정의가 됩니다.
해당 데이터셋은 총 50,000개의 화학 반응과 72,000개의 Synthon으로 이루어져 있습니다.
이 데이터셋을 기반으로 만들어진 Leaving group vocubulary $\mathcal{X}$ 는 총 170 개의 leaving group을 가지게 됩니다 ($\vert \mathcal{X} \vert = 170$).
이렇게 구성된 Leaving group vocabulary 에서 각각의 synthon $c \leq C$ 에 적합한 leaving group이 선택되게 되는 것입니다.

각각의 Synthon $c$ 의 Leaving group을 선택할 때에는 Product 분자의 임베딩 $\mathbf{c}_ {\mathcal{G}_ {p}}$, Synthon의 임베딩 $\mathbf{c}_ {\mathcal{G}_ {s_ {c}}}$, 그리고 이전 Synthon의 leaving group의 임베딩 $\mathbf{e}_ {l_ {c-1}}$ 을 활용하여 아래의 수식과 같이 선택합니다.

$\hat{q}_ {l_ {c}}=\text{Softmax}(\mathbf{U}_ {\tau}(\mathbf{W_ {1}} \mathbf{c}_ {\mathcal{G}_ {p}} + \mathbf{W_ {2}} \mathbf{c}_ {\mathcal{G}_ {s_ {c}}} + \mathbf{W_ {3}} \mathbf{e}_ {l_ {c-1}}))$

최종적으로 모델 학습은 아래의 Cross-entropy Loss function으로 이루어집니다.

$\mathcal{L}_ {s}=\sum_ {c=1}^{C}{\mathcal{L}(\hat{q}_ {l_ {c}}, q_ {l_ {c}})}$


## <span style="font-weight:bold">4. Experiments</span>

<span style="text-decoration:underline;font-weight:bold">Dataset.</span>
Retrosynthesis 연구에서 Benchmark 데이터셋인 USPTO-50K 를 기반으로 실험했습니다.

<span style="text-decoration:underline;font-weight:bold">Evaluation.</span>
모델의 성능은 top-$n$ accuracy ($n=1,3,5,10$) 기준으로 평가했습니다. 
즉, 모델이 제시한 Rank $\leq n$ 안에 정답이 들어있는지에 대한 평가입니다.

<span style="text-decoration:underline;font-weight:bold">Experimental Results.</span>

<p align="center">
  <img width="70%" src="https://user-images.githubusercontent.com/40378824/231692241-e1fe432f-b344-47a5-825e-d414d2fdd81f.png">
</p>

Reaction Class가 알려져 있지 않았을 때 (Reaction class unknown) Semi-Template-based 방식에서 GraphRetro 가 Top-5 를 제외한 모든 결과에서 가장 좋은 성능을 내는 것을 확인할 수 있었습니다.
Reaction Class가 알려져 있을 때 (Reaction class known) Semi-Template-based 방식에서 Top-1 이 가장 좋은 성능을 내는 것 역시 확인할 수 있었습니다.

<p align="center">
  <img width="70%" src="https://user-images.githubusercontent.com/40378824/231694806-38d6f309-2683-47aa-ae49-48cb8e83241b.png">
</p>

논문에서는 또한 각각의 모듈에 대한 성능 평가 역시 보여주었습니다.
Edit Prediction은 Edit 예측에 대한 Top-$n$ 정확도를 보여주었고, Synthon Completion은 ground-truth edit 이후에 주어진 Synthon으로 부터 얼마나 Leaving group을 잘 선택하는지에 대한 정확도를 보여주었습니다.
다만 아쉬운 점은 기존의 Semi-Template-based 모델들과 다른 Edit Prediction / Synthon Completion 모듈을 사용했는데, 기존의 모델들과 비교를 해줬다면 더 좋은 실험이지 않았을까 하는 아쉬움이 있습니다.

## <span style="font-weight:bold">5. Conclusion</span>

이 논문에서는 새로운 방식의 Semi-Template 기반의 Retrosynthesis Analysis 방법을 제안했습니다.
기존의 Semi-Template 기반의 방식이 생성 모델 기반의 방법을 선택하면서 가졌던 문제를 분류 문제로 다시 Formulation 한 점이 인상 깊었습니다.
글 읽어주셔서 감사합니다.