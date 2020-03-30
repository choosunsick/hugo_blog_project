---
title: "역전파와 간단한 계산문제"
date: 2020-03-30T22:07:33+09:00
draft: FALSE
tags: ["R로 딥러닝하기", "신경망", "역전파"]
categories: ["R"]
---

## 역전파

계산그래프의 역전파는 부분적인 계산을 계속 출력하여 최종 계산을 이끌어내듯이 연쇄법칙과 같은 원리를 사용합니다. 즉 복잡한 미분을 작은 계산들의 곱으로 표현하여 쉽게 계산해낼 수 있는 것이 계산그래프를 이용한 역전파의 장점입니다. 참고로 역전파의 목표 역시 가중치와 편향의 입력값 변화에 따른 손실함수의 변화량을 구하는 것이 목표이기에 여전히 미분을 사용한다는 점을 알아두어야합니다.

계산그래프의 역전파가 어떤 구조로 구성되는지 알아봅시다. 먼저 자주 사용되는 덧셈과 곱셈노드의 역전파 구조와 계산방법을 파악해 봅시다. 첫 번째로 덧셈노드의 역전파입니다. 덧셈노드의 역전파는 흘러온 신호를 그대로 출력하는 역할을 합니다.

예를 들어보면, $$z = x+y$$일 때 이것의 역전파는 x에 대한 z의 미분과 y에 대한 z의 미분 값이 x와y 두 가지로 전파 됩니다. 먼저 역전파에서 최초의 입력값 1이 출력되어 덧셈노드로 출력됩니다. 이후 각각의 미분 값은 1로 이 값을 곱하여 각 방향으로 그대로 출력하게 됩니다. 따라서 덧셈노드의 역전파는 항등함수와 같이 입력신호를 그대로 출력하여 다음 노드로 전달합니다.

두 번째로 곱셈노드의 역전파는 입력신호에 순전파 때 값을 서로 바꿔서 곱해주는 방식입니다. 예를 들면 $$z = xy$$라 하고 $$x=10$$ $$y=5$$이면 x에 대한 z의 미분은 5가됩니다. y에 대한 z의 미분 값은 10이됩니다. 만약 이전의 노드에서 1.3이라는 값이 출력되었다면, x값이 전파된 쪽에는 $$1.3*5=6.5$$가 되며, y값이 전파된 쪽에는 $$1.3*10=13$$이 됩니다.

두 역전파 구조에서 최초 입력값이 모두 1이라는 점부터 살펴봅시다. 역전파의 최초 입력 신호는 왜 1로 주어지는 것일까요? 그 이유는 값의 변화가 있다고 가정하기 때문입니다. 단순히 수식으로 살펴보면 z에 대한 z의 미분 값으로 1이 나옵니다. 그러나 그 의미는 구하고자하는 변화량이 있다고 가정하는 것으로 나타낼 수 있습니다.

이제 사과와 오렌지를 사는 예제에서 덧셈노드와 곱셈노드가 섞인 간단한 역전파 계층을 직접 코드로 구현해봅시다.

### 간단한 계층 구현하기

개당 100원하는 사과를 2개, 개당 150원하는 오렌지를 3개 산다고 할 때 최종 금액에 소비세 10%가 붙으면 전체 가격을 구하고, 사과와 오렌지 가격 및 개수 변동에 대한 전체 금액의 변화 값을 알아봅시다. 이 계산을 위해서는 곱셈노드와 덧셈노드의 순전파와 역전파가 필요합니다. 먼저 곱셈노드의 순전파와 역전파부터 구현해보도록 하겠습니다.

```{r}
MulLayer.forward <- function(x, y){
  return(list(x  =  x, y = y, out = (x * y)))
}

MulLayer.backward <- function(forward, dout){
  dx <- dout * forward$y
  dy <- dout * forward$x
  return(list(dx = dx, dy = dy))
}
```

곱셈노드의 순전파는 일반적인 수학계산과 같습니다. 대신 순전파 때 값을 역전파에서 사용해야하기 때문에 입력 받은 x와 y를 함수에서 리스트로 저장해 출력해줍니다. 곱셈노드의 역전파 계산은 설명한 바와 같이 순전파의 입력신호들을 순서를 바꿔서 곱한 값을 계산하여 출력합니다.

다음은 덧셈노드의 구현입니다.

```{r}
AddLayer.forward <- function(x, y){
  return(list(out  = (x + y)))
}

AddLayer.backward <- function(dout){
  dx <- dout * 1
  dy <- dout * 1
  return(list(dx  =  dx,dy  =  dy))
}

```

덧셈노드의 순전파 역시 수학식과 똑같습니다. 역전파 또한, 간단합니다. 역전파의 이전 노드에서 받은 출력신호를 입력신호로 받고 그 값에 각 변수 x,y의 미분 값 1을 곱한 값들을 출력합니다. 이제 구현한 곱셈노드와 덧셈노드를 가지고 문제를 풀어 봅시다.

```{r}
apple <- 100
apple_num <- 2
orange <- 150
orange_num <- 3
tax <- 1.1
```

기본적인 문제 세팅인 사과와 오렌지의 가격과 개수 세금 등부터 변수로 저장해줍니다. 다음은 사과와 오렌지의 총 가격을 계산할 차례입니다.

```{r}

mul_apple_layer.forward <- MulLayer.forward(apple, apple_num)
mul_apple_layer.forward

apple_price <- mul_apple_layer.forward$out
apple_price

mul_orange_layer.forward <- MulLayer.forward(orange, orange_num)
mul_orange_layer.forward

orange_price <- mul_orange_layer.forward$out
orange_price

```

전체 사과의 가격은 사과의 구매 개수에 사과의 개당 가격을 곱한 것입니다. 오렌지도 마찬가지 방식으로 계산해줍니다. 사과는 개당 100원에 2개를 샀으니 $$100*2=200원$$이 되고, 오렌지는 개당 150원에 3개를 샀으니 $$150*3=450$$원이 됩니다. 

```{r}
add_apple_orange_layer.forward <- AddLayer.forward(apple_price, orange_price)
add_apple_orange_layer.forward

all_price <- add_apple_orange_layer.forward$out
all_price

mul_tax_layer.forward <- MulLayer.forward(all_price, tax)
mul_tax_layer.forward

price <- mul_tax_layer.forward$out
price
```

전체 가격은 $$200+450=650원$$으로 그 가격에 세금 10%를 곱하면 구할 수 있습니다. 이제 650원에 소비세 10%를 곱해주면 $$650+650*0.1=715원$$이 됩니다.  

이제 사과나 오렌지의 개수나 개당 가격 혹은 소비세가 변화할 때 전체 가격이 어떻게 변화할지를 알아봅시다. 먼저 소비세가 변화할 때 전체 가격의 변화를 살펴봅시다.

```{r}
dprice <- 1

tax.backward<- MulLayer.backward(mul_tax_layer.forward, dprice)
tax.backward

dall_price <- tax.backward$dx
dall_price

dtax <- tax.backward$dy
dtax
```

`dall_price`, `dtax`두 개의 출력값을 찾을 수 있는데 이것 중 소비세의 변화에 따른 전체 가격의 변화는 `dtax` 값을 의미합니다. 즉 소비세 1(100%) 오르면 전체 가격은 650원 올라갑니다. 실제로 1이 오르면 $$650*2.1=1365$$로 기존 715원에서 650원 오른 1365원인 것을 확인할 수 있습니다. 참고로 소비세의 경우 %이기 때문에 1값이 오르면, 100%가 상승한 것으로 봐야합니다. 이제 `dall_price`의 행방을 살펴보면 이 값은 1.1로 다음 덧셈노드의 입력신호가 됩니다.

```{r}
add_apple_orange_layer.backward <- AddLayer.backward(dall_price)
add_apple_orange_layer.backward

dapple_price <- add_apple_orange_layer.backward$dx
dapple_price

dorange_price <- add_apple_orange_layer.backward$dy
dorange_price
```

덧셈노드의 역전파는 입력값을 그대로 출력합니다. 따라서 사과와 오렌지의 가격을 계산한 곱셈노드에 1.1을 입력신호로 받게 됩니다. 이제 사과의 가격이나 개수가 변화할 때 전체 가격의 변화를 살펴봅시다.

```{r}
apple.backward<- MulLayer.backward(mul_apple_layer.forward, dapple_price)
apple.backward
dapple <- apple.backward$dx
dapple
dapple_num <- apple.backward$dy
dapple_num
```

사과의 전체 가격은 사과의 개수와 개당 가격으로 이루어지는데, 사과의 개수가 변화할 때 전체 가격의 변화를 살펴보겠습니다. 이 문제에서 곱셈노드의 역전파는 순전파에서 사과의 가격인 100에 1.1을 곱한 값인 110이 출력됩니다. 즉 사과의 개수가 1개 늘어나면 전체 가격은 110원 만큼 증가한다는 의미입니다.

같은 방식으로 사과의 개당 가격이 변화할 때 전체 가격의 변화량을 살펴본다면, 사과의 개수인 2를 1.1과 곱하여 2.2를 출력하게 됩니다. 즉 사과의 개당 가격이 1원 올라갈 때 전체 가격은 2.2원 올라간다는 의미입니다. 1원이라는 단위는 의미있는 돈의 단위가 아니기에 100원으로 본다면 개당 가격이 100원 올라가면 전체 가격은 220원 증가하는 것을 의미합니다.     

```{r}

orange.backward<- MulLayer.backward(mul_orange_layer.forward, dorange_price)

dorange <- orange.backward$dx
dorange_num <- orange.backward$dy

c(dapple_num, dapple, dorange_num, dorange, dtax)
```

마찬가지로 오렌지에 대해서 살펴봅시다. 역전파 입력신호로는 1.1이 입력됩니다. 순전파에서 입력된 오렌지의 개당 가격과 개수를 서로 바꿔서 곱해주면 개당 가격이 입력되는 쪽에 개수 3개를 곱한 3.3이 출력됩니다. 반대로 개수가 입력되는 쪽에는 $$150*1.1= 165$$가 출력됩니다.

최종적으로 각 변화량에 대해 정리하자면 다음과 같습니다. 먼저 사과의 경우 사과를 1개 더 살 때마다 전체 가격은 110원 올라가며, 사과 가격이 1원 오르면, 전체 가격은 2.2원 올라간다는 의미입니다. 오렌지의 경우는 오렌지를 1개 더 살 때마다 전체 가격은 165원 올라가며, 오렌지 가격이 1원 오르면 전체 가격은 3.3원이 올라간다는 의미입니다.
