---
title: "다양한 역전파 계층 구현하기"
date: 2020-03-30T22:07:35+09:00
draft: FALSE
mathjax: TRUE
tags: ["R로 딥러닝하기", "신경망", "역전파"]
categories: ["R"]
---

## 각 계층 구현하기

이번에는 신경망 모델에서 역전파를 하기 위해 필요한 함수들 예를 들어 `softmax()`와 `loss()`, `sigmoid()`, `ReLU()` 기존 순전파 계산 `model.forward()`의 역전파 등을 구현해 보겠습니다.

### 활성화 함수의 역전파

활성화 함수로는 앞서 `ReLU()`와 `sigmoid()`가 있습니다. 역전파 신경망 모델에서 좀 더 자주 사용하는 `ReLU()` 부터 어떤 방식으로 역전파가 진행되는지 살펴보겠습니다. `ReLU()`에 대해 다시 떠올려보자면, 이 함수는 0을 기준으로 그보다 크면 그 값을 그대로 사용하고 0보다 작거나 같으면 0을 출력하는 함수였습니다. 즉 $x > 0$ 이면 x를 $x< = 0$ 이면 0을 출력합니다.

계산그래프에서 역전파는 미분을 출력하여 전달하는 것인데, 이는 `ReLU()`의 역전파도 마찬가지입니다. 미분을 한다는 것은 변화량을 구한다는 것과 같습니다.`ReLU()`의 경우 기존에 x의 범위에 따라 다른 출력을 한 것과 마찬가지로 미분에 역시 x의 범위에 따라 달라집니다.

먼저 x가 0보다 클 때 순전파에서는 x를 그대로 출력했습니다. 이는 역전파에서도 마찬가지인데 x가 얼마만큼 변하든 x를 그대로 출력하기에 변화량이 없다는 의미에서 입력받은 역전파의 입력신호에 미분값 1을 곱하여 입력신호를 그대로 출력하게 됩니다.

반면 x가 0보다 작은 경우 순전파에서는 0을 출력했습니다. 이는 역전파에서도 마찬가지입니다. 수식으로보면 상수 0은 미분해도 0 값이 나오기 때문이지만, 이는 사실 변화할 수 없다는 것을 의미합니다. 즉 0을 출력한다는 것은 변화할 수 없는 것이기에 신호를 전달하지 않는다는 의미입니다.

`ReLU()`의 역전파를 R로 구현하면 다음과 같습니다. 참고로 구현에는 역전파 계산시 순전파의 계산 값이 필요하기에 순전파 구현도 포함됩니다.

```{r}

x <- matrix(c(1,-2,-0.5,3),2,2)
x
mask <- x< 0
mask

Relu.forward <- function(x){
  mask <- x < 0
  out <- x
  out[mask] <- 0
  return(list(out = out, mask = mask))
}

Relu.backward <- function(forward, dout){
    dout[forward$mask] <- 0
    return(list(dx = dout))
}
```

먼저 순전파 구현은 이전 글의 구현과 살짝 달라졌습니다. 역전파 계산에서 순전파에 들어온 값들에 대한 0과의 비교가 어렵기 때문에 미리 0과 비교해 어느 원소가 0보다 더 큰지 작은지에 대한 위치정보가 필요합니다. 이런 위치 정보를 마스크라 부르는데 그 이유는 이후 역전파에서 마스크를 쓰듯이 이 위치정보를 그대로 적용하기 때문입니다. 마스크는 0과 비교하여 작으면 TRUE, 크면 FALSE 값으로 표시한 객체로 출력됩니다.

`ReLU()`의 출력은 두 가지입니다. 마스크를 적용한 행렬이 첫 번째 출력이고, 0보다 크거나 작은지에 대해 판단하는 마스크가 두 번째 출력입니다. 이 출력값은 리스트로 묶여서 출력됩니다.

`ReLU()`의 역전파는 순전파에서의 출력값과 이전의 노드에서 들어오는 입력신호를 입력받습니다. `ReLU()`의 역전파에서는 들어온 입력신호의 값이 0 보다 큰지 작은지를 판단하는 것이 아닙니다. 순전파 때 구한 마스크를 적용하여 기존 마스크 상에서 0보다 작은 값의 위치에만 0을 입력하고, 그외 부분은 입력된 값을 그대로 출력합니다.

다음은 `sigmoid` 함수의 역전파입니다. 시그모이드 함수는 x와 + 외에 지수함수와 나누기 과정이 있었습니다. 그렇기에 이 두 가지의 역전파 방식을 알아야 시그모이드 함수의 역전파를 구성할 수 있습니다. 먼저 시그모이드 함수는 `1/1+exp(-x)`과  같았습니다. 이 식을 분해해보면, -1과 x를 서로 곱하고 그것에 exp인 지수함수를 적용하고, 그 값에 1을 더해주고 마지막으로 1에 대해 해당 값으로 나누어줍니다.     

역전파일때 노드 순서는 나누기, 더하기, 지수함수, 곱하기 노드 순서가 됩니다. 나누기 함수의 역전파를 예를 들면 y = 1/x에 대한 미분을 생각해보면 됩니다. 1/x의 미분은 -1/x^2이 됩니다. 이 식은 -y^2과 같으며, 역전파 계산은 순전파에서 흘러온 값에 -y^2을 곱하면 됩니다.

지수 함수의 역전파는 간단합니다. 그냥 순전파 때의 값을 곱하고 출력하면 됩니다. 순전파의 출력을 그대로 곱하는 이유는 지수 함수의 미분이 지수함수형태로 출력되기 때문에 순전파의 값을 그대로 곱해줍니다. 시그모이드 함수에서는 순전파 때 값이 exp(-x)이기에 이 값을 곱해주게 됩니다.

+와 x 노드의 경우 역전파가 어떻게 작동하는지 설명했기에 생략하고, 최종적으로 시그모이드 함수의 역전파가 어떻게 출력되는지 살펴보면, y^2exp(-x)가 역전파의 최종 출력이며, 이 값을 y로 정리하면, y(1-y)로 정리할 수 있습니다. 따라서 R로 구현해보면 다음과 같습니다.

```{r}
sigmoid.forward <- function(x){
  out <- 1 / (1+exp(-x))
  return(list(out = out))
}

sigmoid.backward <- function(forward,dout){
  dx <- dout*(1 - forward$out)*forward$out
  return(list(dx = dx))
}
```

마찬가지로 시그모이드 함수의 역전파 역시 순전파 계산을 포함하기에 순전파 계산의 값을 인자로 받고 역전파의 입력신호를 인자로 받습니다.  

### Affine 계층의 역전파

신경망 순전파 모델에서는 가중치와 편향을 갱신하기 위해 가중치 신호의 총합과 편향의 합을 계산했었습니다. 이 계산에서 가중치와 편향은 모두 행렬로 나타나며, 행렬의 곱과 행렬 간 브로드캐스트 연산이 필요했습니다. 앞서 활성화 함수를 비롯한 곱셈과 덧셈노드의 경우 모두 단일한 값을 바탕하여 역전파 계산이 진행되었습니다. 그러나 가중치 총합과 편향의 합을 계산하는 어파인 변환의 경우 행렬을 바탕으로 역전파 계산이 이루어집니다.

어파인 변환 계층은 행렬의 덧셈과 곱셈으로 이루어져 있습니다. 덧셈노드의 역전파의 경우 행렬을 입력값으로 받아도 입력값을 그대로 내보내는 역할을 합니다. 그럼 행렬의 곱셈노드의 역전파는 어떻게 이루어질까요? 행렬의 곱셈 또한 기존 곱셈노드와 같은 방식으로 역전파가 진행됩니다. 단 주의할 점은 행렬이기 때문에 순전파에서 흘러온 두 행렬의 전치행렬로 서로 바꾸어 곱한다는 점입니다.

전치행렬을 사용하는 이유는 가중치 행렬에 대해 X 행렬의 모든 원소로 편미분을 진행하면 그 결과가 W 행렬의 전치행렬로 도출되기 때문입니다. X 행렬 역시 마찬가지로 모든 원소에 대해 편미분을 진행하면 X 행렬의 전치행렬이 도출됩니다. 이에 따라 계산그래프의 곱셈노드는 전치행렬을 곱하게 되는 것입니다.  

전치행렬에 대해 예를 들어 보면 다음과 같습니다. 1x2 크기의 X행렬과 2x3 크기의 W행렬의 곱이 순전파 계산에서 진행되었다면 역전파 계산에서는 1x2 행렬이 흘러온 신호로 이전 노드에서의 출력된 미분 값과 3x2 행렬(2x3 행렬의 전치행렬)이 반대로 2x3행렬이 흘러온 신호로는 2x1 행렬(1x2행렬의 전치행렬)이 곱해지게 됩니다.  

따라서 행렬의 곱셈노드가 전치행렬을 곱한다는 점을 제외하면 어파인 변환의 역전파는 덧셈노드와 곱셈노드로 이루어진 간단한 역전파입니다. 이제 어파인 변환 계층의 역전파를 구현해 봅시다.

```{r}
Affine.forward <- function(W, b, x){
    out <- sweep((x %*% W),2, b,'+')
    return(list(out  =  out, W  =  W, x  =  x))
}

Affine.backward <- function(forward, dout){
  dx <- dout %*% t(forward$W)
  dW <- t(forward$x) %*% dout
  db <- matrix(colSums(dout), nrow = 1)
  return(list(dx  =  dx, dW  =  dW, db  =  db))
}
```

역전파 계산시 순전파 값이 필요하기에 순전파부터 구현합니다. 기존의 함수와 차이점은 함수의 출력이 리스트로 된다는 점입니다. 어파인 변환의 역전파에서 핵심인 전치행렬은 `t()` 함수를 통해 찾을 수 있습니다. db 객체는 덧셈노드의 역전파 출력 신호 중 하나로 역전파 입력신호인 dout을 입력 받아 그대로 출력합니다. dx,dw 객체는 역전파의 입력 신호를 입력 받아 서로 바꾸어 전치행렬을 곱해준 값을 출력합니다.  

### softmax with loss 계층의 역전파

기존에는 softmax 함수와 loss 함수를 따로 분리해서 사용하였습니다. 그러나 역전파에서는 이 두 함수를 하나로 묶어서 사용하는데 그 이유는 두 계층을 합칠 경우 역전파 출력이 단순해지기 때문입니다. 두 함수를 분리할 경우 손실함수의 변화량 계산이 복잡하고 오래걸리게 됩니다. 이에 따라 두 계층을 하나로 합치게 된 것입니다.

`softmax with loss()` 계층의 역전파는 `cross_entropy_error` 함수의 역전파와 `softmax`함수의 역전파로 구성됩니다. 먼저`cross_entropy_error` 함수는 로그, 곱셈노드, 덧셈노드로 이루어져 있습니다. 참고로 여러개의 데이터를 사용하는 배치의 경우 역전파의 최종 값에 데이터 개수로 나누어 주면 됩니다.

```{r}
cross_entropy_error <- function(y, t) {
    delta <- 1e-7
    batch_size <- dim(y)[1]
    return(-sum(t*log(y+delta))/batch_size)
}
```

log 노드의 역전파는 1/x로 나타낼 수 있습니다. `cross_entropy_error` 함수의 역전파는 분류하는 개수에 따라 출력되는 개수가 달라지는데 분류 개수가 10개면 10개를 출력합니다. `cross_entropy_error` 함수의 순전파의 계산 노드 순서는 로그노드, 곱셈노드, 덧셈노드, 곱셈노드로 이어집니다. 역전파의 경우 반대로 이어집니다. 참고로 마지막에 배치계산을 위한 나누기는 역전파 계산에서 제외됩니다.

순전파의 마지막 곱셈노드(코드에서는 맨앞의 -를 의미)에서 입력된 신호는 정답 라벨과 softmax 함수가 도출한 특정 분류 값에 대한 확률 값에 로그를 씌운 값의 곱 신호 하나와 -1 신호가 흘러옵니다. 따라서 첫번째 곱셈노드의 역전파는 순전파에서 두번째 신호인 -1을 1에 곱한 값이 다음 노드로 흘러갑니다.

다음은 덧셈노드(`sum()`함수를 의미)이므로 -1이 그대로 흘러가는데 이 때 분류하는 개수의 수 만큼 곱셈노드가 존재합니다. 여러 노드가 있다해도 과정은 다 같은 방식으로 진행되니 걱정할 필요가 없습니다. 이 곱셈노드는 순전파에서 첫번째 분류 기준에 대한 정답(원핫 인코딩) 라벨이 흘러오고 다음으로 softmax 함수가 도출한 특정 분류 값에 대한 확률 값에 로그를 씌운 값이 흘러옵니다. 따라서 역전파는 정답 라벨에 -1을 곱한 값이 로그 노드의 입력신호로 흘러갑니다.

로그노드 역전파는 순전파에서 첫번째 구분 기준 값에 대한 소프트맥스 함수의 적용 값이 흘러옵니다. 이 값을 y1으로 표기하면, 로그노드의 역전파는 최종적으로 첫번째 정답라벨의 원핫 인코딩 벡터 중 첫 번째 값인 -t1/y1이 됩니다. 이런 값들이 분류 기준 개수에 따라 -tn/yn(n은 분류 기준의 개수를 의미)까지 존재하게 되며 이후 소프트맥스 함수의 역전파의 입력신호가 됩니다.  

```{r}
softmax_single <- function(a){
    c <- max(a)
    sum_exp_a <- sum(exp(a - c))
    return(exp(a - c) / sum_exp_a)
}

```

다음으로 소프트맥스 함수의 역전파입니다. 소프트맥스 함수는 순전파에서 지수함수와 덧셈 나눗셈 곱셈노드로 이어집니다. 코드에서는 `exp()`함수가 지수노드이며 `sum()` 함수가 덧셈노드이고, `1/sum_exp_a`이 나눗셈 노드이고 `exp(a - c)*1/sum_exp_a` 가 곱셈노드로 나타납니다. 역전파에서는 이와 반대로 이어지는데 먼저 곱셈노드에서는  `cross_entropy_error` 역전파 값인 -tn/yn를 입력신호로 받습니다.

순전파의 마지막 곱셈노드에서 `1/sum_exp_a`와 `exp(a - c)`가 순서대로 흘러옵니다. 이때 역전파의 첫번째 노드인 곱셈노드의 역전파는 `1/sum_exp_a`가 흘러온 곳으로 `exp(a - c)`와 -tn/yn가 곱해지게 됩니다. 이때 식을 `y = exp(a - c) / sum_exp_a`라 정리하면, `-tn*sum_exp_a`가 됩니다. 해당 값이 n개가 나누기 노드로 입력됩니다.

나눗셈노드의 역전파는 순전파 때 값인 `1/sum_exp_a`의 값에 제곱을 하고 나눈 후 -1을 곱하는 것이였습니다. 식으로 보자면, `-1/sum_exp_a^2`을 `-tn*sum_exp_a`에 곱하는 값을 출력합니다. 이것을 정리하면, t1에서 tn까지의 합에 `1/sum_exp_a`를 곱하는 식이 나옵니다. 단 여기서 t1~tn까지의 합은 원핫 인코딩의 합으로 1이 된다는 점을 기억할 수 있습니다. 따라서 출력은 `1/sum_exp_a`가 됩니다. 이 출력이 덧셈노드를 통과해 지수함수의 역전파에 대한 입력신호가 됩니다.

지수함수의 역전파에 다른 입력신호로는 이전 곱셈노드의 출력인 `tn/exp(a-c)`이 됩니다. 이 값과 `1/sum_exp_a` 이 값에 지수함수의 역전파인 `exp(a-c)`를 곱해줍니다. 식을 정리하면 `exp(a-c)/sum_exp_a-tn`이 되고 이 식을 y로 정리하면 yn-tn이 됩니다.

이제 역전파 과정을 구현해보겠습니다. 다른 것과 마찬가지로 순전파 계산이 필요하기 때문에 순전파부터 구현합니다.  

```{r}
source("./DeepLearningFromForR/functions.R")

SoftmaxWithLoss.forward <- function(x, t){
    y <- softmax(x)
    loss <- cross_entropy_error(y, t)
    return(list(loss  =  loss , y  =  y, t  =  t))
}

SoftmaxWithLoss.backward <- function(forward, dout = 1){
    dx <- (forward$y - forward$t) / dim(forward$t)[1]
    return(list(dx  =  dx))
}
```

순전파 구현에는 손실함수 값과 소프트맥스를 적용한 값 그리고 정답 라벨을 리턴합니다. 역전파 구현은 순전파 때 값 y와 t를 인자로 받고 역전파의 최초 입력신호인 1을 인자로 받아 계산합니다. 최종 출력이 yn-tn의 형태이기 때문에 `(forward$y - forward$t) `와 같이 구현해주면 끝이지만 데이터를 배치로 다루기 때문에 `dim(forward$t)[1]`로 나누어 주면 역전파의 출력이 끝나게 됩니다.