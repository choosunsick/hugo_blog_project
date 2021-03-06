---
title: "Chapter2 3층 신경망 구현"
date: 2020-03-16T19:34:26+09:00
draft: FALSE
tags: ["R로 딥러닝하기", "신경망", "소프트맥스", "3층 신경망 구현"]
categories: ["R"]
---

이 시리즈는 R로 딥러닝을 구현하고 설명하는 것에 목표를 둔 글입니다. 벡터와 행렬연산에 이어 2번째 순서입니다. [이전 글](https://choosunsick.github.io/post/activation_fuctions/)과 내용이 이어집니다. 이전 글들은 [이곳](https://choosunsick.github.io/post/contents_list/)에서 찾아 보실 수 있습니다.

## 소프트맥스 함수  

출력층에서 사용하는 함수에는 항등함수와 소프트맥스 함수가 있습니다. 항등함수의 경우 이름 그대로 입력이 곧 출력이 되는 함수로 신경망 회귀 모델을 만들 때 사용합니다.

```{r}
identify_fun <- function(x){
  return(x)
}
x <- matrix(c(0.3, 2.9, 4), 1, 3)
> x
     [,1] [,2] [,3]
[1,]  0.3  2.9    4
> identify_fun(x)
     [,1] [,2] [,3]
[1,]  0.3  2.9    4
```

항등함수는 보시는 바와 같이 입력 값을 그대로 출력합니다.

반면 신경망 분류모델의 경우 소프트맥스 함수를 사용합니다. 소프트맥스 함수는 분류모델의 결과에 대해 확률로 변환을 이루게 할 뿐 그외 어떠한 개입도 하지 않습니다. 계산된 결과 값 중 가장 큰 값의 인덱스를 따르는 점은 변하지 않습니다.  그렇기 때문에, 실제로 모델 학습이 완료되고 나서는 추론을 할 경우에는 소프트맥스 함수를 생략할 수 있습니다. 소프트맥스 함수를 구현해보겠습니다.

```{r}
softmax <- function(a){
  return(exp(a) / sum(exp(a)))
}
x <- matrix(c(0.3, 2.9, 4), 1, 3)
> x
     [,1] [,2] [,3]
[1,]  0.3  2.9    4

> softmax(x)
           [,1]      [,2]      [,3]
[1,] 0.01821127 0.2451918 0.7365969

> sum(softmax(x))
[1] 1

> exp(100)
[1] 2.688117e+43

> softmax(1000)
[1] NaN
```

소프트맥스 함수는 지수함수의 분수로 구성됩니다. 분수 형태를 취하기 때문에 그 출력 값은 0과 1 사이의 실수값이 나옵니다. 이 실수값이 곧 신경망 분류모델의 결과 값에 대한 확률로 나타납니다. 출력된 확률 값의 개수는 분류하려는 대상의 개수에 따라 결정됩니다. 또한, 확률 값이기 때문에 출력된 값들의 총합은 1이 되는 특징이 있습니다.

구현된 소프트맥스 함수의 결과와 그 결과에 `sum()` 함수를 통해 총합이 1이되는 것을 확인해 볼 수 있습니다. 그러나, 이 구현에는 한가지 문제점이 있습니다. 바로 지수함수의 오버플로 문제입니다. 오버플로 문제란 컴퓨터가 표현할 수 있는 수의 범위가 한정되어 너무 큰 값을 표현할 수 없을 때 발생하는 문제입니다.

지수함수의 경우 100이상만 되는 값이 입력되면 0이 40개가 넘는 아주 큰 수가 되며 이것은 계산의 불안정을 불러일으킵니다. 예를 들어 소프트맥스 함수에 1000 값을 입력하면, 값이 너무 커져 표시할 수 없기에 숫자가 아니라는 의미의 NaN 값이 나오게 됩니다. 따라서 이 문제를 방지할 수 있는 함수로 수정이 필요합니다.

```{r}
softmax <- function(a){
  exp_a <- exp(a - max(a))
  sum_exp_a <- sum(exp_a)
  return(exp_a / sum_exp_a)
}
> x
     [,1] [,2] [,3]
[1,]  0.3  2.9    4
> softmax(x)
           [,1]      [,2]      [,3]
[1,] 0.01821127 0.2451918 0.7365969
> sum(softmax(x))
[1] 1
> softmax(1000)
[1] 1
```

이렇게 중간에 `max()` 함수를 통해 수식을 조정해주면, 100과 같이 큰 값이 들어가도 오버플로 문제가 발생하지 않습니다.

## 3층 신경망 구현하기

신경망 구현에 필요한 함수들을 다 알아보았습니다. 이제부터 직접 3층 신경망을 구현하여 알아보겠습니다. 3층 신경망의 구조부터 떠올려보겠습니다. 3층 신경망은 입력층과 2개의 은닉층 그리고 출력층 있는 네트워크 구조입니다. 입력층의 노드 개수는 입력하는 데이터에 따라 달라지며, 출력층의 노드 개수는 분류하려는 대상의 개수로 설정하는 것이 일반적입니다. 은닉층의 노드 개수는 임의로 설정해도 되지만 입력층의 노드 개수 보다 작거나 같은 수준으로 설정합니다.  

3층 신경망의 구조를 떠올리고 나면 신호의 전달이 다음과 같이 진행됨을 알 수 있습니다. 입력층-은닉층(W1,b1), 은닉층-은닉층(W2,b2), 은닉층-출력층(W3,b3)으로 3개의 신호 전달이 있습니다. 3개의 신호 전달됨에 따라 초기값 역시 3개의 가중치와 편향이 필요합니다. 또한, 신호 전달 과정을 구현함에 있어 활성화 함수로서 시그모이드 함수와 출력층에서 결과를 출력하기 위한 항등함수가 사용됩니다. 이제 신호 전달과정을 나타낸 함수와 초기값을 만드는 함수 2가지를 구현해보겠습니다.

```{r}
source("./functions.R")
source("./numerical_gradient.R")

init <- function(){
  W1 <- matrix(seq(0.1, 0.6, 0.1), nrow = 2, ncol = 3)
  b1 <- matrix(seq(0.1, 0.3, 0.1), nrow = 1, ncol = 3)
  W2 <- matrix(seq(0.1, 0.6, 0.1), nrow = 3, ncol = 2)
  b2 <- matrix(c(0.1, 0.2), nrow = 1, ncol = 2)
  W3 <- matrix(seq(0.1, 0.4, 0.1), nrow = 2, ncol = 2)
  b3 <- matrix(c(0.1, 0.2), nrow = 1,ncol = 2)
  model <- list(W1, b1, W2, b2, W3, b3)
  names(model) <- c("W1", "b1", "W2", "b2", "W3", "b3")
  return(model)
}

model.forward <- function(model, x){
  a1 <- sweep(x %*% model$W1, 2, model$b1, "+")
  z1 <- sigmoid(a1)
  a2 <- sweep(z1 %*% model$W2, 2, model$b2, "+")
  z2 <- sigmoid(a2)
  a3 <- sweep(z2 %*% model$W3, 2, model$b3, "+")
  return(identify_fun(a3))
}
```

초기값을 만들기 위한 함수는 `init()`란 이름으로 작성합니다. 이 함수를 실행시키면, 초기값들을 사용할 수 있습니다. 모델의 초기값은 다를 수 있으며 가중치와 편향 값을 튜닝할수록 더 나은 성능의 모델을 만들 수 있습니다. 이것은 저자가 준비한 초기값으로 손글씨 데이터를 인식하고 분류할 때 다시 확인할 수 있습니다.

`init()` 함수의 초기값들이 모두 행렬의 형태인 것에 의문이 있을 수 있습니다. 가중치를 행렬로 표시하는 이유는 층별 노드의 개수와 연관됩니다. 즉 가중치 행렬의 행은 이전층의 노드 개수이고, 열은 다음층의 노드 개수입니다. 예를 들면 W1(1층의 가중치)의 행은 입력층의 노드 개수를 의미하고, 열은 첫 번째 은닉층의 노드 개수를 의미합니다.

편향의 경우 벡터로도 나타낼 수 있습니다만, 가중치 행렬과 같은 의미에서 1행 n열의 행렬로 이해하는게 좋습니다. 즉 편향 행렬의 행은 모든 층에서 하나의 노드만 존재하기에 편향 노드 개수는 항상 1이며, n열은 다음층의 노드 개수를 의미합니다. 예를 들면 b1(1층의 편향)은 1행 3열의 행렬로 1행은 입력층의 편향의 개수이며, 3열은 첫 번째 은닉층의 노드 개수입니다. 초기값을 층별로 설정하고 나면 이제 신호 전달 과정을 하나의 계산 과정으로 묶어주어야 합니다.   

3개의 신호 전달 과정과 출력층에서 확률로 변환하는 계산 과정을 `model.forward()` 함수로 묶어줍니다. 먼저 입력층에서 첫 번째 은닉층까지의 신호 전달을 계산해봅시다.

입력된 x에 입력층에서 은닉층으로 전달하는 과정의 가중치 행렬인 W1을 곱해주고 거기에 편향 b1을 더해줍니다. 편향을 더할 때 `sweep()`함수를 사용하는 이유는 행렬의 크기가 서로 다르기 때문입니다. 만약 `sweep()` 함수를 사용하지 않는다면, 벡터로 바꾸어 재활용 규칙을 활용하거나, 행렬의 크기를 변경하거나 rray 패키지로 브로드캐스트 연산을 하는 방법도 있습니다. 하지만, sweep 함수를 사용하는 것이 가장 빠른 방법입니다.

입력층에서 은닉층으로의 신호 전달 과정에서 활성화 값이 계산되면 이제 활성화 함수인 시그모이드 함수를 통해 출력 신호로 만들어 줍니다. 이 출력 신호는 다음층의 입력 신호로 역할하며 이 과정이 마지막 출력층에서 출력 신호가 확률로 변경되기 전까지 반복됩니다. 마지막 출력층에서는 활성화 함수 대신 항등 함수 또는 소프트맥스 함수를 사용하여 출력된 분류 결과를 확인할 수 있습니다.     

```{r}
model <- init()

model
$W1
     [,1] [,2] [,3]
[1,]  0.1  0.3  0.5
[2,]  0.2  0.4  0.6

$b1
     [,1] [,2] [,3]
[1,]  0.1  0.2  0.3

$W2
     [,1] [,2]
[1,]  0.1  0.4
[2,]  0.2  0.5
[3,]  0.3  0.6

$b2
     [,1] [,2]
[1,]  0.1  0.2

$W3
     [,1] [,2]
[1,]  0.1  0.3
[2,]  0.2  0.4

$b3
     [,1] [,2]
[1,]  0.1  0.2


x <- c(1, 0.5)
> x
[1] 1.0 0.5
y <- model.forward(model, x)
> y
          [,1]      [,2]
[1,] 0.3168271 0.6962791
```

`init()`함수를 실행하여 초기값 행렬을 확인할 수 있고, 간단한 인풋값으로 model.forward 함수의 결과를 확인할 수 있습니다. 이렇게 구현한 3층 신경망 모델을 토대로 손글씨 데이터를 통해 신경망 분류 모델이 어떻게 분류를 할 수 있는지 알아볼 것입니다.
