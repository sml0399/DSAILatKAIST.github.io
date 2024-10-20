---
title: "[ICLR 2021] Generative Scene Graph Networks"
permalink: Generative_Scene_Graph_Networks.html
tags: [reviews]
use_math: true
usemathjax: true
---

# **Generative Scene Graph Networks**

  

## **0. Background**

본 논문은 scene graph에 관한 논문인데 자세한 내용을 다루기 전에 먼저 scene graph가 무엇인지 알아보자.

Scene graph는 여러 연구분야에서 사용되는 개념이라 상황에 따라 구체적인 모습은 달라질 수 있지만 맥락은 동일하다. 하나의 scene이 주어졌을때 그로부터 graph를 얻는 것인데 여기서의 scene은 image라고 생각하면 편하다.

예를 들어 가족 사진을 하나의 scene이라고 보면 여기서의 scene graph는 "아버지", "어머니", "나", "형"으로 구성된 4개의 노드가 있고 이들을 이어주는 edge를 만들어 graph를 만들 수 있을 것이다.

<p  align="center"><img  src="https://camo.githubusercontent.com/83e4e20b07b16a4e371a781b8bea11b049542fae068dceb0c9883ff5ce7382fa/68747470733a2f2f6966682e63632f672f67773438336e2e6a7067"></p>

위 그림도 scene graph의 한 예시가 될 수 있다.

하지만 무조건 위 graph와 같이 image에 등장하는 object들이 무조건 node일 필요는 없다. 상황에 맞게 scene에서 노드들을 정의하고 그들 관계를 나타내는 graph는 넓은 범위에서 모두 scene graph이다.

본 논문에서 정의하는 scene graph는 위에서 설명한 예시의 scene graph와는 조금 다른 개념으로 하나의 object를 node로 설정하고 이어지는 node는 다른 object가 아니라 그 object의 구성요소가 아래로 이어지는 형태의 graph이다. tree 구조의 graph라고 이해하는게 더 직관적으로 이해할 수 있을 것이다. 

<p  align="center"><img  src="https://camo.githubusercontent.com/5e699114525bd7c00861a2ba66e53660a7f65ba682dbb868a657ed92a1714414/68747470733a2f2f6966682e63632f672f5367445259562e706e67"></p>

위는 본 논문에서 설명하는 scene graph와 유사한 scene graph로 전체 scene이 World이고 그 안에 하나의 groun-1이 존재하고 각 부분들이 하위 노드로 이어지는 형태를 보여준다.

결국 scene graph란 scene(image)에서 graph를 얻는 것이고 그 graph를 어떻게 구성할지는 상황에 따라 매우 다양하다. 



## **1. Motivation**  

  

Human cognition의 핵심은 오직 관측으로부터 object를 찾는 것이고, 최근 관련하여 다양한 unsupervised object-centric representation이 연구되어 왔다. 이러한 연구들을 통해 얻은 object의 symbolic representation은 relational reasoning이나 causal inference 등 다양한 분야에 사용될 수 있다.

  

  

특히나 본 논문에서는 일상에서의 scene이 compositional objects(하나의 큰 object가 여러 작은 object의 조합으로 이루어짐, 예를 들어 얼굴은 눈+코+입으로 구성됨)로 이루어져있다는 특징을 motivation으로 진행된다. 사람은 primitive parts(가장 작은 object 구성요소)와 그것으로 부터 나아가는 part-whole relationship(부분-전체 관계)을 인식할 수 있기 때문에 큰 object를 인지한다는 느낌이다.

  

이렇게 여러 작은 primitive part의 조합으로 object를 추론하기 때문에 복잡한 object도 보다 효율적으로 추론할 수 있게 되고 primitive part의 새로운 조합으로 새로운 object를 만들 수 있다.

  

  

본 논문에서는 unlabed image로부터 scene graph를 추론할 것이고 이를 위해 variational autoencoder를 사용하였다. 이때의 고려해야할 점은 obeject(ex : 얼굴)를 구성하는 part(ex : 눈, 코, 입)를 찾아야하는데 이 part가 image 상에서 occlusion(ex : 사진에서 눈, 코, 입이 곂쳐 구분하기 힘든 상황)이 나타나면 적절하게 분리하기 어렵하는 것이다. 그래서 object로부터 적절한 primitive part를 찾고 이들을 tree형태로 조합하여 최종 hierarchical scene graph(ex : root에 얼굴이 있고 이어지는 leaf node가 눈, 코, 입으로 구성된 graph 형태)를 얻어내는게 본 논문의 목표라고 이해할 수 있다.

  

  

이를 위한 key point는 scene graph가 recursive structure를 가진다는 것이다. 전체적인 structure of tree를 추정하는 것은 그 tree의 structure of subtree를 추정하는 과정과 비슷하다는 느낌으로 top-down 방식으로 하나의 image로부터 scene graph를 얻기 위해 image에서 여러 object를 찾고 그 object들을 구성하는 part를 찾는 방향으로 이해할 수 있다. 다시 말하면 전체 scene에서 object들을 찾고 각 object에서 그 object를 구성하는 더 작은 기본 요소(primitive part)를 찾는 과정으로 recursive하다고 볼 수 있다.

  

  

이러한 관점은 기존의 scene decomposition method인 SPACE와 유사한데 다만 SPACE는 object에서 part로 분리할때 occlusion에 대해 어려움이 있었고, 이는 SPACE의 bottom-up 방법을 이용하기 때문이었다. 본 논문에서는 이와 다르게 top-down 방식으로 하였고 composition에 대한 prior를 도입하여 occlusion(곂치는 현상)에 대해 더 개선할 수 있도록 하였다.

  

  

## **2. Related Work**

  

  

본 논문과 관련된 여러 이전 연구들과 관련 개념들에 대한 간략한 설명이다.

  

### Object-centric representation

  

본 논문의 방법은 unsupervised object-centric representation learning에 속하는 연구로 이 분야의 method들은 주로 supervision이 없는 상황에서 scene을 object로 분해하고 이러한 object의 representation을 학습하는 end-to-end model을 제안한다. 이러한 모델은 크게 2가지로 나뉠 수 있는데 그 것이 1) scene-mixture models, 2) spatial-attention models 이다. 이러한 모델들와 본 논문의 차이점은 object를 part까지 나눴다는 점이고 또한 inference시에 scene-mixture model과는 다르게 spatial-attention model을 사용하여 object position에 대한 정보를 주었다. 또한 occlusion을 다루기 위해 prior를 도입했다는 차이가 있다.

  

  

### Hierarchical scene representations

  

Scene에서 part-whole relationship을 모델링하는 것은 image classification, parsing, segmentation 분야에서 널리 활용되어 왔다. 하지만 이러한 모델은 object 하나에 대해서만 적용되었고 여러 object를 다루는 scene generation에는 적용할 수 없었다. 또한 기존의 연구들은 part-whole relationship의 학습을 위해 사전에 정의된 part에 대한 정보가 필요했다. 대신 본 논문에서는 multi-object에 대해 part-whole relationship을 학습하고 이때 individual part에 대한 knowledge가 필요하지 않다는게 차이점이다.

  

또한 shape generation에서 part hierarchies가 연구되었었는데 이때의 hierarchy는 input으로 들어왔었다. 이번 연구에서는 hierarchy를 input으로 사용하는게 아니라 static scene을 input으로 하고 ouput으로 hierarchy를 내놓는다는 차이가 있다.

  

  

### Hierarchical latent variable models

  

본 논문의 model은 hierarchical latent variable model과 관련이 있고 이 개념을 이용하여 object와 part간의 relationship을 잘 반영하는 계층적 구조를 학습하는 데에 있다.

  

  

## **3. GENERATIVE SCENE GRAPH NETWORKS**

  

본 논문의 주요 방법론인 Generative Scene Graph Networks에 대한 설명이다.

  

  

### 3.1 Generative Process

  

x라는 image가 set of foreground variable인 Z<sub>fg</sub>와 background variable Z<sub>bg</sub> 로 다음과 같이 구성된다고 하자.

  

주어진 forground 정보와 그로부터 얻은 backgound 정보, 그 둘로부터 얻은 x의 곱의 형태를 띄고 있다.

  

  

<p  align="center"><img  src="https://user-images.githubusercontent.com/48014450/233981211-8d1d8d99-687e-47c7-b98e-8e0f95e9e267.png"></p>

  

  

여기서 그럼 Z<sub>fg</sub> 가 무엇이나면 아래 그림으로 이해할 수 있다. 여기서 leaf node는 scene에서의 primitive entity로 더이상 분해되지 않는 최소 단위이고 internal node는 child node의 조합으로 나타난 abstract node로 이해 할 수 있다. 이때의 조합에 관한 정보는 둘 사이의 edge에 담겨있으며 조합은 affine transformation으로 rotation, scaling, translation을 포함한다. Z<sub>v</sub><sup>pose</sup> 나 Z<sub>v</sub><sup>appr</sup> 의 의미는 각각 node v와 그 parent 사이의 상대적인 위치 정보와 node v과 그의 child의 정보를 종합한 표현을 나타낸다고 이해할 수 있다.

  

<p  align="center"><img  src="https://user-images.githubusercontent.com/48014450/233982909-d3e8dcf8-d817-496a-ba09-8dddc08f6be3.png"></p>

  

위의 그림을 바탕으로 최종 Z<sub>fg</sub>를 구하면 다음 식과 같다. 모든 노드 집합에 대해 root node인 r에 대해 정보와 root node를 제외한 node들에 대해 위치과 표현 정보를 하나하나 더해주어 전체 foreground 정보를 얻는다 과정이다.

  

<p  align="center"><img  src="https://user-images.githubusercontent.com/48014450/233981218-2ae299bd-2efe-4f72-b821-e3cef2b25a4d.png"  height="50px"  width="600px"></p>

  

  

#### - Representing tree structures

  

근데 위의 계산식은 일단 tree 구조가 있어야 계산할 수 있다. 그럼 위의 tree 구조자체는 어떻게 얻을까? 위와 같은 tree 구조를 다루기 위해선 latent representation에 대한 개념이 필요하다. 일단 scene에 대한 가능한 tree 구조에 대해 어느 정도 제한을 두기 위해 각 node에 maximum out-degree를 설정하고 구체적인 구조를 결정하기 위해 각 node와 연결될 수 있는 가능한 edge들을 고려할 것이다.

  

이를 위해 node v와 그의 parent node 사이의 edge에 대해 Bernoulli Variable P<sub>v</sub><sup>pres</sup> 를 설정하였다. P<sub>v</sub><sup>pres</sup> = 0이란 뜻은 edge가 없다는 뜻이다. $\bar{z}$<sub>r</sub><sup>pres</sup> 가 root r에서 node v까지 이어지는 variable의 곱이라고 하면 다음과 같이 나타낼 수 있다. 즉, node v까지의 edge의 존재 유무는 node v부터 parent까지의 edge와 root r부터 그의 parant까지의 edge의 곱으로 나타낼 수 있다는 뜻이다.

  
  

<p  align="center"><img  src="https://user-images.githubusercontent.com/48014450/233981219-f4fc7b9d-03c6-4f1f-98fd-ba6da03e097a.png"  height="40px"  width="500px"></p>

  
  

이 사실을 바탕으로 Z<sub>fg</sub> 식을 재구성하면 다음과 같다. edge까지 고려해준 식으로 바뀌었다.

  

<p  align="center"><img  src="https://user-images.githubusercontent.com/48014450/233981222-5a905a16-85ac-4ae2-88dc-d756ec97b9b8.png"></p>

  

  

#### - Differentiable decoder

  

decoder는 recursive 조합 과정을 따르는 역할로 encoder의 반대 역할이다. 구체적으로는 먼저 각 leaf node로부터 g()라는 neural network를 이용하여 small patch $\hat{x}$<sub>v</sub> 와 대응하는 binary mask $\hat{m}$<sub>v</sub> 를 얻는다. 이러한 primitive patch를 가지고 composition을 통해 object와 더 나아가 scene을 얻는데 그 과정은 다음과 같다.

  

  

<p  align="center"><img  src="https://user-images.githubusercontent.com/48014450/233981224-8f0ce0c1-8ce8-4b00-9927-5feaa9116b6e.png"  height="100px"  width="500px"></p>

  

위 식은 아래 그림으로 이해하면 더 편하다. 유의해야할 점은 단순히 patch와 mask를 합친게 아니라 node v의 pose까지 고려하여 occlusion을 다루었다는 점이다.

  

  

<p  align="center"><img  src="https://user-images.githubusercontent.com/48014450/233999904-8b2379cd-33a8-4ced-a222-3591dc57e28b.png"  height="400px"  width="600px"></p>

  

  

위 과정을 거치면 최종적으로 root r에 대한 $\hat{x}$<sub>r</sub> 와 $\hat{m}$<sub>r</sub> 을 얻게 된다. 그 이후에 다른 spatial broadcast decoder를 이용하여 Z<sub>bg</sub> 를 $\hat{x}$<sub>bg</sub> 로 decode해준다. 이렇게 얻은 $\hat{x}$<sub>r</sub>와 $\hat{x}$<sub>bg</sub>를 가지고 최종 full scene은 다음과 같이 얻을 수 있다.

  

  

<p  align="center"><img  src="https://user-images.githubusercontent.com/48014450/234002632-0aabef5f-2995-4bd0-b403-45ab5b8eac07.png"  height="40px"  width="600px"></p>

  

  

### 3.2 Inference and Learning

  

  

가장 첫번째 식을 계산하는 과정이 intractable integral이기 때문에 variational inference로 근사 할 것이다.

  

<p  align="center"><img  src="https://user-images.githubusercontent.com/48014450/234003966-3434e7d0-a2ce-4252-bf61-9b8233978638.png"  height="40px"  width="600px"></p>

  

foreground variable을 유추하기 위해 probabilistic scene graph가 recursive하다는 사실을 이용할 것이고 이는 root node의 child가 가장 먼저 추론되고 그 이후 child node의 subtree로 내려가면서 추론하는 개념이다.
  

이러한 top-down factorization은 아래와 같은 식으로 나타난다. 기본적으로 parent의 정보를 가지고 node v를 유추하는 과정이다.

  

<p  align="center"><img  src="https://user-images.githubusercontent.com/48014450/234003971-4dabc9e8-96db-4935-86de-c37e070bafa4.png"  height="80px"  width="600px"></p>

  

위 식은 이전식과 달라진게 x에 대한 condition이 더 주어졌다는 것이다. 이는 top-down information에 bottom-up image feature를 더해준 것으로 이는 entity v에 대해 더 relevent한 information을 제공하는 역할을 한다.

  

  

## **4. Experiment**

  

  

### **Experiment setup**

  

* Dataset

  

GSGN의 효과를 제대로 측정하기 위해 본 논문에서는 dataset을 직접 만들었다. unsupervised object-centric representation learning에 흔히 쓰이는 Multi-dSprites와 CLEVR을 합쳐 하나의 데이터 셋을 만들었고 이를 각각 2D Shapes과 Compositional CLEVR datasets이라고 부르기로 하였다.

  

각 데이터 셋은 3가지 type의 primitive part와 이러한 part들의 조합으로 구성되는 총 10가지 type의 object가 있다. object type 중 3가지는 하나의 part로 구성되고 다른 3가지는 2개의 part로 구성되고 마지막 4가지 object는 3개의 part 조합으로 구성된다. scene을 만들기 위해 object를 random(size, type, position and orientation)하게 변화를 주어 구성하였다.

  

  

* baseline

  

이전의 hierarchical scene representations은 single-object scenes을 가정하였고 특히나 prior knowledge가 필요하였다. 그래서 우리의 dataset에 직접적으로 적용할 수가 없다. 그 대신 SOTA non-hierarchical scene decomposition model인 SPACE를 우리의 상황에 바꾸어서 baseline으로 사용하였다.

  

  

### **Result**

  

  

#### - Scene graph inference

  

scene graph inference의 과정은 다음 그림으로 이해할 수 있다. input image에 대한 bounding box는 pose variable을 나타내고 reconstruction은 inferred appearence variable을 decoder로 넣은 후의 결과이다. 그림을 보면 알 수 있다시피 object와 part를 잘 분리하는 것을 알 수 있다.

  

<p  align="center"><img  src="https://user-images.githubusercontent.com/48014450/234011866-15bb7af4-e0fc-4496-9b1d-573bf40724ce.png"  height="400px"  width="800px"></p>

  

  

다음은 수치적인 result를 나타낸다.

  

<p  align="center"><img  src="https://user-images.githubusercontent.com/48014450/234014047-9ca7528a-63bd-4c97-a259-e3678f0e5745.png"  height="600px"  width="600px"></p>

  

counting accuracy는 inferred scene graph에서 노드 수의 정확성을 측정하고 F1 점수는 inferred node가 실제 entities를 잘 capture하는 지를 나타낸다. 대부분 좋은 결과를 내지만 GSGN-No_Aux는 scene graph structure 형성에 실패하는데 이 이유는 모든 presence variable이 1로 가는데 이는 redundant nodes때문이라고 말한다.

  

  

Compositional CLEVR dataset에 대해서도 GSGN이 SPACE보다 더 좋은 성능을 보였는데 이는 SPACE가 occlusion이 발생한 부분에 대해 분해를 어려워하기 때문이라고 말한다. 이와 관련된 성능은 Table 3에서 더 자세히 볼 수 있는데 Table 3은 occlusion level에 따라 실험한 결과로 <100인 경우 SPACE-P의 성능이 매우 낮은데 이때 occluded part에 대해 miss하는 경향이 있기 때문이다.

  

  

#### - Scene graph manipulation.

  

GSGN으로 얻은 scene graph는 interpretable한 tree structure와 pose variable을 가지기 때문에 infered scene graph에서 얻은 latent variable들을 조합하여 새로운 object와 scene을 만들 수 있다.

  

<p  align="center"><img  src="https://user-images.githubusercontent.com/48014450/234019972-9cba7300-9f6a-460e-9c90-e39aff9a5bfc.png"  height="400px"  width="600px"></p>

  

위 그림은 그 결과로 scene graph inference에서 얻은 scene graph에다가 스케일과 좌표의 변화를 주어 얻은 새로운 object와 scene이다. 새로운 object와 scene도 어색함이 없이 잘 생성되는 것을 확인할 수 있다.

  

  

## **5. Conclusion**

  

본 논문에서는 multi-object scenes에 대해 개별 parts의 지식없이 deep generative model을 적용한 unsupervised scene graph을 수행한 GSGN을 제안하였다. GSGN은 top-down prior와 bottom-up image features를 활용하였고 이는 여러 occlusion이 발생할때 적절히 대처하게 해주었다. 또한 GSGN은 scene graph manipulation을 통해 new object와 scene도 generate할 수 있었다.

  

이러한 방식을 바탕으로 좀 더 realistic한 환경에 deeper hierarchies와 more complex appearance에도 잘 적용하게 하는 것이 future work이다. 개인적으로 본 논문의 방법론이 실제 상황에 scene에 적용되기 위해서는 많은 연구가 필요할 것 같다. 다만 recursive하게 scene을 분해하는 과정은 흥미로웠다.

  

  

## **6. Reference & Additional materials**


  

* Github Implementation

  

* Reference

  

- [[ICLR-21] Generative Scene Graph Networks](https://openreview.net/forum?id=RmcPm9m3tnk)
