---
layout: post
title:  "Convolutional Neural Network with Theano"
date:   2016-04-28 0:20:00
categories: Deep Learning
description: Theano를 기반으로 구현한 Supervised Learning
tags:
- DeepLearning
- Supervised learning
- Neural Networks
- Theano
- Convolutional Network
---

# CNN의 특징

## Sparse Connective 

Convolutional Neural Networks(CNN)은 뉴런간 지역적인 상관관계를 이용한 것으로, 인접한 레이어와 뉴런간의 연결 패턴을 적용한 것이다. 즉, hidden layer **m**의 입력들은 **m-1** layer의 출력 중 일부이며, unit들은 공간적으로 연속적으로 입력 데이터를 받아 들인다고 할 수 있다.

<img class="col one center" src="/images/201604/sparse_1D_nn.png"/>

위 그림에서 unit간 연결 관계를 살펴보면, 입력을 제외한 모든 Layer의 unit들은 세개의 입력만을 하위 layer에서 받아들인다.  하지만 layer **m+1**의 unit의 입력은 layer **m**을 거치면서 5개가 된다. 따라서 적은 입력을 받는 unit으로 구성한다 하더라도 layer와 unit의 연결 수의 상관관계에 따라서 unit은 더 많은 입력 데이터에 대한 학습을 할 수 있다.

## Shared Weights

CNN에서는 filter의 값이 입력 unit에 대하여 모두 동일하게 적용된다.

<img class="col one center" src="/images/201604/conv_1D_nn.png"/>

즉, 위 그림에서 hidden unit의 각 연결 패턴은 unit 마다 동일하다. weight 또한 동일하다. 뿐만 아니라 Shared Weight에 대하여 Gradient Descendent 역시 약간의 수정만 한다면 적용할 수 있다.

Layer의 unit들을 이처럼 반복하는 동작은 보이는 영역에 대하여 unit의 위치와는 독립적인 feature들을 학습하도록 할 수 있다. 또한 weight sharing은 학습의 효율을 극대화 할 수 있는데, 이는 학습할 파라미터의 수를 줄일 수 있기 때문이다. Weight sharing으로 대표되는 이러한 제약조건은 CNN 모델로 하여금 vision 문제에 대하여 보다 일반화된 문제 풀이 방법을 제시한다.

# Details and Notation

## Hypothesis function

Weight Sharing을 하면서 모든 unit에 대해 동일한 연결 패턴을 적용하는 문제는 입력 데이터에 대한 linear filter를 convolution 연산을 수행하고, bias를 더한 뒤 non-linear 함수에 적용하는 것으로 구현 할 수 있다. 만일 k번째 feature map을 $h^k$로 정의한다면, filter는 $W^k$로 bias는 $b^k$라 할 수 있으며, 이들의 상관관계는 다음과 같다.

$$ h^k _{ij} = s((W^k * x)_{ij} + b_k) $$

representation을 보다 다양하게 해주기 위해서 hidden layer는 여러 개의 feature map을 갖는다. 그리고 Hidden Layer의 Weight matrix $W$는 destination feature map, source feature map, source vertical position, source horizontal position 등으로 구별지을 수 있다. bias $b$는 하나의 element를 갖는 vector로서, 그 element는 각각의 feature map에 적용된다.

## Convolution Operator

Theano는 Conolution의 기능을 `theano.tensor.signal.conv2d` 함수를 통해 제공한다. 입력 parameter는 다음과 같다.

| paramter | 설명 |
|---|---|
|input| 입력 이미지. image shape은 input_shape과 동일하게 reshape 되어 있어야 한다 |
|input_shape| batch size, source features (eg. r, g, b), input image width, input image height 등 input image에 대한 4d tensor |
|filter| weight Matrix |
|filter_shape| filter의 '# of output feature, # of input feature, filter size (width), filter size (height)'를 나타내는 4d tensor |

## MaxPooling

Pooling은 Neural Network를 구성할때 사용하는 Sub-sampling을 의미하며, over-fitting을 피하기 위한 generalization의 방법으로 사용된다. 또한 연산량을 줄여준다.

Theano에서 Maxpooling은 `theano.tensor.signal.downsample.max_pool_2d`를 통해 할 수 있다.

{% highlight python %}
from theano.tensor.signal import downsample

input = T.dtensor4('input')
maxpool_shape = (2, 2)
pool_out = downsample.max_pool_2d(input, maxpool_shape, ignore_border=True)
f = theano.function([input],pool_out)
{% endhighlight %}

**ignore_border**의 역할은 pooling을 할 때 경계면에 놓였다면, pooling 자체를 안하고 아무 데이터도 내놓지 않거나(ignore_border=True), 그래도 남아있는 데이터로 pooling을 하는 경우(ignore_border=False)로 나누인다.

# LeNet-5

LeNet-5는 다음과 같은 절차에 따라 실행한다.

<img class="col three center" src="/images/201605/LeNet5.png"/>
<div class="col three caption">
Architecture of LeNet-5. Source: Yann LeCun et al. Gradient-Based Learning Applied to Document Recognition
</div>

1. Convolution
1. Maxpooling
1. Convolution
1. Maxpolling
1. Fully Connected
1. Fully Connected
1. Softmax

이것을 theano tutorial에서는 다음과 같이 구현했다.

1. **LeNetConvPoolLayer** 구성
  * Convolution, MaxPooling을 함께 수행하는 Layer 구현
1. **FullyConnected HiddenLayer** 구성
  * 이전의 Multilayer Perceptrion을 구현한 Layer 재사용

첫번째 LeNetConvPoolLayer에서 사용할 Representaion은 20개, 두번째 LeNetConvPoolLayer는 50개의 feature를 사용하였다. Filter로 사용된 Matrix는 (5,5)였으며 단계에 따라서 입력 데이터의 크기는 다음과 같이 변하였다.

| 단계 | 동작 | 입력 | 출력
| 1 |Convolution | (28, 28) | 20 * (24, 24)| 
|2|Maxpooling | 20 * (24, 24) | 20 * (12, 12) |
|3|Convolution| 20 * (12, 12) | 50 * (8, 8) |
|4|Maxpolling| 50 * (8, 8) | 50 * (4, 4) |
|5|Fully Connected(ANN) | 50 * (4, 4) | 500 |
|6|Fully Connected(Logistic Regression) | 500 | 10 |
|7|Negative Log Liklihood|

실행해보니 내 맥북에서는 점심시간에 돌린것이 6시간 가까이 돌고나니 끝났다.

