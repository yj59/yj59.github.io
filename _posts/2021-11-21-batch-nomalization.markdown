---
layout: post
title:  "Batch Nomalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift   "
date:   2021-11-21
categories: deep_learning
use_math: true

permalink: /deep_learning/batch_norm/
---

<br/>

파라미터 vs 하이퍼 파라미터

파라미터 : 모델 내부에서 결정되는 변수(가중치 W, 편향치 b)

하이퍼 파라미터 : 모델링 시 사용자가 설정해주는 값(lr, epoch, iteration(한 번 학습시키는데 필요한 batch의 수))

<br/>

## Abstract   

![1](https://user-images.githubusercontent.com/93882395/142740335-fe7a0829-f19d-424a-9033-5b0ee1cb3984.png){: width="550"}

 &nbsp; 신경망 학습 과정에서 이전 레이어로 인해 파라미터 값이 변하면서 각 레이어의 input 분포도가 달라진다. 이 경우 모델의 학습 속도가 낮아지는데,  lr의 값을 줄이거나 가중치 초기화를 신중히 설정해야 할 뿐만 아니라 비선형 모델의 학습이 어려워지기 때문이다(Internal Covariate Shift). 논문에서는 mini-batch를 통해 레이어의 input에 정규화를 수행하여 ICS 문제를 해결하고자 한다. 

<br/>

![2](https://user-images.githubusercontent.com/93882395/142740333-0bfb9fc5-350d-403c-89d7-f61149ddab64.png){: width="550"}

 &nbsp; 배치 정규화 사용 결과 더 큰 learning rate 사용이 가능하며, 가중치 초기화에 신경을 덜 쓰게 된다(weight regularization term 등 제외). 또, batch norm이 정규화 기능을 수행하면서 경우에 따라 dropout을 사용하지 않아도 유사한 성능을 구현한다.

+ 실험 결과 : 이미지 분류 모델에 배치 정규화를 적용한 후, 14times 적은 훈련 step에서 original 모델과 동일한 accuracy 달성 (성능 향상)

<br/>

## 1. introdution

![3](https://user-images.githubusercontent.com/93882395/142739361-e50af61f-d8e9-4b3f-b12c-768008fed4c7.png){: width="550"}

 &nbsp; 확률적 경사하강법(SGD)은 loss 함수를 최소화시킬 수 있는 파라미터 Θ를 찾는다. 

 &nbsp; 실제 학습에서는 mini-batch를 고려하여 한 번에 batch 사이즈 만큼 forward와 backpropagation을 진행하기 위해 m만큼 데이터를 나눌 수 있도록 수식을 변형한다. mini-batch를 이용해 전체 데이터의 gradient을 근사한다(m 사이즈를 늘릴수록 전체 데이터 근사도 높음).

<br/>

>Using mini-batches of examples, as opposed to one exam- ple at a time, is helpful in several ways. First, the gradient of the loss over a mini-batch is an estimate of the gradient over the training set, whose quality improves as the batch size increases. Second, computation over a batch can be much more efﬁcient than m computations for individual examples, due to the parallelism afforded by the modern computing platforms.
>While stochastic gradient is simple and effective, it requires careful tuning of the model hyper-parameters, speciﬁcally the learning rate used in optimization, as well as the initial values for the model parameters. The train- ing is complicated by the fact that the inputs to each layer are affected by the parameters of all preceding layers – so that small changes to the network parameters amplify as the network becomes deeper.



mini-batch를 사용한 SGD의 장점

+ mini-batch의 loss 함수 gradient는 전체 학습 데이터의 gradient 근사한다(mini-batch 크기가 커질 수록 품질 향상).

+ 컴퓨터 병렬 처리로 개별 batch보다 m개의 mini-batch를 계산하는 것이 효율적이다.

SGD의 단점

+ 최적화에 사용되는 lr값과 초기 가중치 설정을 주의해야 한다.
  + 이전 레이어로 인해 파라미터가 변하면서 각 레이어에 대한 input 값의 분포도가 변할 수 있다. 특히 레이어의 개수가 많아질수록 파라미터의 변화율이 커진다.

<br/>

 &nbsp; 한 입력 레이어를 기준으로 생각했을 때, 해당 레이어의 input값 분포도는 매 학습마다 변화한다(covariate shift : 레이어로 인한 파라미터의 변화. 학습 속도 저하 요인).  초기 입력 레이어의 분포도 변화는 domain adaptation 방법을 적용하여 완화시킬 수 있었으나 DNN에서 문제가 발생할 경우, 히든 레이어에도 적용할 수 있는 해결안이 필요하다.

> domain adaptation : 두 도메인에 같은 task를 적용하기 위해 사용됨(transfer learning)
>
> source dataset을 통해 학습된 네트워크가 target dataset를 분류고자 할 경우, 두 domain의 간격을 줄여주는 방법(source domain과 target domain의 격차 해소)  

<br/>

![4](https://user-images.githubusercontent.com/93882395/142743372-04de2514-c5c5-47d6-a66b-a07256975215.png){: width="550"}



 &nbsp; 어떤 네트워크에서 두 개의 레이어가 연속적으로 존재하는 상황을 가정했을 때(수식 1 : input값 u가 레이어 F1과 F2를 거치고 난 후의 l값) F2 레이어만 놓고 수식을 세우면 다음과 같은 식이 정의된다. F2의 파라미터를 학습할 때 입력값으로 들어오는 것은 앞 레이어 F1의 output에 영향을 받는다.

sub-network의 input 분포도가 일정하게 유지되면 파라미터 학습 과정이 효과적일 것이다.

<br/>

![5](https://user-images.githubusercontent.com/93882395/142743635-16ab70d5-0056-4ce5-b2bb-256054381f80.png){: width="550"}



 &nbsp; input 분포도를 고정시키면 sub-network에 좋은 결과를 나타낸다. sigmoid 함수의 경우, 분포도가 일정하지 않다면 기울기 소실으로 인해 모델의 학습 속도가 느려진다. (큰 값이 input으로 주어질 경우, 기울기가 0에 가까워지도록 saturate되어 기울기 소실 )

<br/>

![6](https://user-images.githubusercontent.com/93882395/142744087-b4b09e34-9c7f-4c76-a5e7-22c50a5e80f0.png){: width="550"}



 &nbsp; input값 x는 앞 레이어의 파라미터에 영향을 받으므로 학습 도중 고차원 공간 데이터 x가 saturated regime으로 이동해 수렴이 늦춰질 수 있다. 네트워크가 깊어질수록 saturated가 가속화된다. (기존 해결 방안으로는 ReLu 함수, initialization, lr의 크기 감소 등 존재)

입력값 분포도를 더욱 고정적으로 만들어 Optimizer가 saturated regime에 매핑되지 않고 학습이 안정적으로 이루어지면서 속도가 개선되도록 한다.

<br/>

## 2. Towards Reducing Internal Covariate Shift

whitening

 입력 데이터의 전처리를 하기 위한 다양한 기술의 일종으로, 평균이 0일 뿐만 아니라 공분산이 단위행렬인 정규분포 형태의 데이터로 변환하는 기법 

=> 서로 다른 피처끼리 연관성이 없는 형태로 데이터 전처리 : 각각의 피처에 대해서 decorrelated될 수 있다.

입력 데이터의 각각의 특징들이 decorrelated되고 평균값은 0이며 1만큼의 varience를 가지도록 함.

단, nn 안에서 화이트닝을 진행하도록 하는 것은 간단한 작업이 아님(네트워크 자체를 직접적으로 수정하거나 파라미터를 바꿔주는 등의 작업 필요로 함  => 많은 연산 필요, 효과적이지 않을 뿐더러 어려움)

<br/>

## 3. Normalization via Mini-Batch Statistics

<br/>

![31](https://user-images.githubusercontent.com/93882395/142744810-04ccec09-8a4d-48a5-b13f-9255d95b8ae5.png){: width="550"}

모든 레이어의 input 값을 모두 whitening하기에는 너무 많은 연산 비용이 들고, 모든 곳에서 미분을 할 수도 없다.

논문에서는 두 가지 가설을 세운다.

1. 한 레이어의 input과 output의 feature들이 직접 연관되게 whitening하지 않고,  독립적으로 존재해야 한다. => 각각의 freature들에 대해 개별적으로 정규화를 수행해 평균이 0, 분산이 1을 가지도록 설정한다.
2. 단순히 정규화만 진행하게 되면 시그모이드 함수 자체에서 선형적인 구간에만 대부분의 데이터가 쏠리게 된다. => 입력 데이터들이 선형적인 모양을 띠게 되면서 전체 네트워크의 비선형성을 잃어버릴 수 있다.

<br/>

![32](https://user-images.githubusercontent.com/93882395/142745996-0801cd10-6791-4f4a-97d8-13737732867a.png){: width="550"}



정규화된 데이터에 gamma와 beta를 계산해 비선형성을 유지하는 파라미터를 output으로 반환 => gamma와 beta 두 가지 파라미터르 학습시키는 방향으로 레이어를 구성하여 문제를 해결한다. 

gamma와 beta값을 적절히 설정해서 다시 원래의 데이터로 간단하게 되돌릴 수도 있다.



<br/>![33](https://user-images.githubusercontent.com/93882395/142746448-08f72c4b-7791-4e8d-a907-9f0bec2951b2.png){: width="550"}



정규화는 전체 데이터를 처리하지만, 실제로 모든 데이터를 전부 고려해서 정규화를 수행하는 것은 불가능하므로 두 번째 가정을 세운다. => SGD에서 mini-batch를 사용한 것과 동일한 방법으로, mini-batch에서의 평균값과 분산값을 계산한다.

mini-batch data로 layer의 input 값을 정규화할 경우,  모두 backpropagation 처리가 가능하다. => normalize를 mini-batch 단위로 처리하게 되면 한번에 처리하는 연산량이 감소한다

일반적으로 만약 k번째 레이어에 대해 배치 정규화를 수행한다면 k번째 가중치값을 이용해서 행렬곱을 수행한 후 정규화를 수행한다. 이후,  activation을 거쳐 다음 레이어의 인풋값으로 이동한다. 여기서 배치 정규화의 경우 일종의 화이트닝 transfomation 역할을 수행하게 된다.

<br/>

![34](https://user-images.githubusercontent.com/93882395/142746482-67816727-40ef-4a44-9aa3-8f4ef262070e.png){: width="550"}



+ 평균과 분산을 구하는 수식은 입력값에 대한 수식과 매우 유사하나 입력 데이터들은 각각의 히든 레이어로 들어가는 입력 값들에 해당함(이전 레이어에서 나온 activation값)

+ m= 입력 데이터의 개수(batch-size), 만약 하나의 모델에 이미지 128개가 들어올 경우, m=128

+ 평균값과 분산값은 모두 x값을 통해 진행하기 때문에 실제로 학습을 진행하는 파라미터는 \gamma와 \beta 뿐

+ 이미지 64개, 3차원인 레이어의 경우 이미지 각각마다의 \gamma와 \beta가 차원마다 개별 존재(수식은 한 개의 차원에 있는 x값에 대해서만 설정한 것)

+  => 레이어의 입력 차원이 k일 때, 학습할 두 개의 파라미터 \gamma와 \beta 또한 k 차원을 가진다(= 총 2k개의 파라미터를 가짐)

+ 결론 : 히든 레이어는 앞쪽에서 건너온 x값이 있다고 가정했을때, 현재 batch에 포함되어 있는 x값들의 평균과 분산을 구하고 정규화된 \hat{x}값을 구한다.

+  \epsilon같은 경우는 연산을 수행할 때 안전성을 더하기 위해서 넣어주는 작은 크기의 상수. 만약 mini-batch의 원소가 동일한 분포도를 가질 경우, epsilon이 무시되고 평균 0, 분산 1인 연산 그대로 진행

+ 알아두어야할 점 : 입력 데이터에 대한 정규화는 앞선 식에서 모두 정규화를 마칠 수 있지만,  배치 정규화의 경우에는 네트워크의 성능이 향상되는 방향으로 알아서 추가적인 파라미터인 \gamma와 \beta를 학습하도록 아키텍쳐가 구성되어 있다. => 정규화가 자동으로 파라미터를 학습시키면서 경사하강법 과정이 오작동하는 경우 방지(정규화 과정이 항상 동일할 경우 b 발산 가능)

  \gamma : 비선형 구조를 만족시키기 위해 일종의 스케일링 수행

  \beta : 값의 위치를 조금 바꾸어주는 translation 효과

  \gamma와 \beta를 수식에 합산하면서 결과적인 output인 y_{i}를 출력할 수 있음 => 다음 레이어의 입력으로 들어감

  

> (추가 : 왜 활성화함수로 ReLu 대신 Sigmoid를 사용하는가? =>예를 들어 x가 자연수이고 이값을 0,1로 분류할때 선형함수라고 가정하고 대충 평균에 분류경계가 형성되는데 학습데이터 중에 아주 큰값이 들어오면 평균에 있던 경계가 이동하면서 분류율이 떨어지죠. 시그모이드 함수를 잘보면 극단치값이 들어오면 무한대로 발산시키기에 분류경계가 변하지 않아요. 좀더 자세한건 김성님의 모두를 위한 동영상 강좌를 보시면 이해하실겁니다.)

<br/>

![35](https://user-images.githubusercontent.com/93882395/142746887-18d9dd3a-010a-4fbc-9b78-14972b3522fa.png){: width="550"}



![36](https://user-images.githubusercontent.com/93882395/142747027-c89381a9-6ee5-4368-9c38-461fae01687a.png){: width="550"}

체인룰에 의해 loss값을 y로 미분한 값이 y 파라미터로 들어오게 되면, 다른 파라미터에도 체인룰을 적용해 각각파라미터의 gradient를 계산할 수 있다.

<br/>

![37](https://user-images.githubusercontent.com/93882395/142747204-2796b357-f11c-47d9-9c48-9270a95211b9.png){: width="550"}

배치 정규화는 네트워크에 정규화된 activation을 제공하는 미분 가능한 레이어이다. 이 배치 정규화 레이어로 인해 우리는 항상 일정한 분포도를 가지는 입력값으로 모델 학습이 가능하다. (ICS 감소, 학습 고속화)

배치 정규화의 gamma, beta 파라미터는 필요에 따라 input 값의 분포도를 동일하게 유지할 수 있는 능력을 배치 정규화에 제공한다.

<br/>

## 3.1 Training  and  Inference  with  Batch- Normalized Networks

<br/>

![38](https://user-images.githubusercontent.com/93882395/142747537-b9ebf694-1328-44dc-8120-5816862a8741.png){: width="550"}

![39](https://user-images.githubusercontent.com/93882395/142747538-c9428cf3-1a13-4f24-a21f-764b061e1e35.png){: width="550"}



네트워크에 배치 정규화를 적용하기 위해 먼저 활성화할 부분을 정의하고, 그 사이에 배치 정규화 레이어를 끼워 넣는다. => x를 input으로 받는 정규화는 BN레이어로부터 BN(x)를 input으로 받게 된다.

학습 과정에서는 mini-batch의 평균과 분산을 이용하지만, inference 단계에서는 minibatch를 이용하지 않는다. (배치 정규화가 온전하게 이루어지지 않음) 논문에서는 output이 오로지 input에 의존하기를 원한다. 



>inference 시에 입력되는 값을 통해서 정규화를 하게 되면 모델이 학습을 통해서 입력 데이터의 분포를 추정하는 의미 자체가 없어지게 된다. **즉, inference 에서는 결과를 Deterministic 하게 하기 위하여 고정된 평균과, 분산을 이용하여 정규화를 수행하게 된다.**

<br/>

![40](https://user-images.githubusercontent.com/93882395/142747781-bf12522a-eeb8-44e0-ad95-079b5d1d0871.png){: width="550"}



실제로 평가를 진행할 때는 학습 과정에서 사용했던 데이터에 대한 mini-batch의 이동 평균(moving average)를 이용한다. (평균값과 분산값 고정)추론 이전에 미리 mini-batch 별로 시행됐던 평균과 분산을 이용해 이동 평균을 계산한다.  => mini-batch에서 각각의 평균을 다시 평균값으로 계산하면 모평균 추청 값이 됨

==> 하나의 mini-batch를 통한 학습이 끝날 때마다 추론 테스트 수행(해당 mini-batch까지에 한해서만 이동평균 적용)

<br/>

## 3.2 Batch  Normalization  enables  higher learning rates

<br/>

![41](https://user-images.githubusercontent.com/93882395/142748438-e8f9ff12-44f3-42e3-b152-3ec5998e9f66.png){: width="550"}



![42](https://user-images.githubusercontent.com/93882395/142748515-0f03412e-0abb-4d54-9673-5dbb4e848e1d.png){: width="550"}

cnn에서 배치정규화를 사용하는 방법 설명



W와 b는 학습할 파라미터고, g는 sigmoid나 ReLU와 같은 activation 함수이다. DNN과 마찬가지로 x=Wu+b를 input으로 받아 output을 activation 함수에 전달한다.

입력값 u 자체를 정규화할 수 도 있겠으나, 네트워크 자체를 정규화하는 것이 더 효과가 좋다.(u는 비선형함수의 출력값인 경우가 많으므로, 학습 도중 분포가 지속적으로 변함 => u 자체의 평균과 분산값을 제어한다고 해서 CS가 크게 감소하지도 않음)

정규화 과정에서 파라미터 b는 제거된다. $g(BN(Wu))$ => bias의 기능을 BN의 beta가 수행함

 => gamma와 beta는 모든 activation에서 사용하는 것이 아니라 각각의 채널 (피처맵) 단위로 사용한다(채널의 개수만큼 gamma와 beta를 가진다).

=> 모든 활성화 함수마다 BN을 적용하지 않으므로 cnn에 이러한 배치정규화를 사용한다고 하더라도 파라미터의 수가 크게 증가하지 않음

<br/>

## 3.3 Batch  Normalization  enables  higher learning rates

![43](https://user-images.githubusercontent.com/93882395/142748897-e6153167-473d-4611-a465-4dcf265ff09a.png){: width="550"}

![44](https://user-images.githubusercontent.com/93882395/142748899-52e6d6ec-5f61-4011-af4f-612f807fb84e.png){: width="550"}



일반적인 DNN은 학습 속도가 높으면 기울기가 소실되거나 폭주할 가능성이 있다. 배치 정규화가 네트워크 전체의 activation을 정규화함으로써 각 파라미터들의 변경이 기울기에 영향을 미치는 것을 방지하게 된다.

또, lr을 크게 설정할 경우,  역전파 과정에서 기울기를 증폭시켜 explosion 현상이 발생한다 (파라미터의 분포도가 균일x)

=> 배치 정규화를 사용하면 역전파가 매개변수의 scale에 영향을 받지 않는다(분포도가 균일하게 생성됨)

레이어 야코비 행렬(다변수 함수에서의 미분값)과 역전파에 영향을 미치지 않으며 가중치를 크게 설정할수록 배치 정규화가 파라미터 값을 고정시킨다.

<br/>

## 3.4 Batch  Normalization  regularizes  the model

<br/>

![50](https://user-images.githubusercontent.com/93882395/142748147-573ba6e9-917b-4f86-a175-f70ecdc58fa5.png){: width="550"}

배치 정규화를 사용할 경우 과적합을 제거하거나 줄여 dropout과 같은 일반화 효과를 얻을 수 있다.

