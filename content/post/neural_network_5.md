---
title: "Chapter3 2층 신경망 구현"
date: 2020-03-14T22:14:14+09:00
draft: False
tags: ["R로 딥러닝하기", "신경망", "순전파", "기울기", "모델 구현"]
categories: ["R"]
---

## 신경망 모델에서 기울기

신경망 모델에서 기울기는 무엇을 의미할까요? 신경망 모델에서 기울기는 가중치 행렬의 원소들이 각각 조금 씩 변할 때 손실 함수의 변화정도를 의미하는 것입니다. 앞서 모든 변수에 대해 미분을 진행하는 것은 곧 가중치의 모든 원소에 대해 미분을 진행하는 것과 같은 것임을 밝혔습니다.

그러므로 가중치에 대한 손실 함수의 기울기는 가중치 행렬 W의 모든 원소에 대한 미분으로 이루어집니다. 예를 들어 W의 1행 1열의 원소를 조금 변화할 때 손실 함수의 값은 얼마나 변하는가를 측정하는 것이 신경망 모델에서 기울기의 역할입니다. 이제 간단한 신경망을 구현해 실제로 기울기를 구해보겠습니다.

```{r}
source("./DeepLearningFromForR/functions.R")
simplemodel  <- function(){
   W <<- matrix(c(0.47355232, 0.85557411, 0.9977393, 0.03563661,0.84668094,0.69422093), nrow = 2)
}
simplemodel()
W

simplemodel.forward <- function(x) {
  return(x %*% W)
}

numerical_gradient_simplemodel <- function(f,x,t){
    h <- 1e-4
    vec <- vector()

    for(i in 1:length(W)){
        W[i] <<- (W[i] + h)
        fxh1 <- f(x,t)
        W[i] <<- (W[i] - (2*h))
        fxh2 <- f(x,t)
        vec <- c(vec, (fxh1 - fxh2) / (2*h))
        W[i] <<- W[i]+h
    }
    return(matrix(vec, nrow = nrow(W) ,ncol = ncol(W)))
}

x <- matrix(c(0.6,0.9),nrow = 1,ncol = 2)
x

p <- simplemodel.forward(x)
p   

t <- matrix(c(0,0,1),nrow = 1,ncol = 3)
t

loss <- function(x,t){cross_entropy_error(softmax(simplemodel.forward (x)),t)}
loss(x,t)

dw <- numerical_gradient_simplemodel(loss,x,t)
dw

```

먼저 `source()` 함수를 사용해 이전에 정의했던 `softmax()`와 `cross_entropy_error()`를 불러옵니다. 그리고, 초기값을 `simplemodel` 함수로 설정합니다. 간단한 신경망이므로 x와 W값의 곱 계산에 `softmax` 함수를 적용한 코드로 `simplemodel.forward` 함수로 정의합니다. 그 다음 가중치 행렬 W1의 기울기 값을 구하기 위해 기존 `numerical_gradient`를 `simplemodel`에 맞게 바꾸어 가중치 W1을 미분합니다. `source()` 함수로 불러온 `cross_entropy_error()`를 이용해 손실 함수의 값을 계산하는 loss(손실 함수)도 만듭니다.

이제 간단한 신경망 모델의 기울기를 구해보면 기존 가중치 행렬과 같은 크기의 기울기 정보를 얻을 수 있습니다. 결과를 살펴보면, 가중치 행렬 1행1열의 원소는 약 0.21로 양수 값을 가지고 있습니다. 이 값이 의미하는 바는 가중치 행렬 W의 첫번째 원소를 1e-4 만큼 변화하면, 손실 함수의 값이 약 0.21 만큼 증가한다는 뜻입니다.

즉 양수 값이 나온 원소는 손실 함수의 값을 증가시키기 때문에 음의 방향으로 원소값을 줄여야한다는 의미입니다. 예를 들면 W의 1행 1열의 원소 0.4735523 값이 감소해야 손실 함수의 값이 줄어들게 됩니다. 마찬가지로 다른 양수 결과 값들 역시 그 값이 감소해야합니다. 이제 값이 음수인 경우를 살펴봅시다. 예를 들어 마지막 6번째 원소인 2행 3열의 원소의 경우 그 값이 약 -0.54인데, 이는 이 원소를 1e-4 만큼 변화시키면 손실 함수의 값이 약 0.54만큼 줄어든다는 의미입니다. 즉 음수 값이 나온 원소는 손실 함수 값을 내려주기 때문에 양의 방향으로 원소값을 늘려주어야 한다는 의미입니다.

이제 경사하강법이 무엇인지 알았으니 전체 학습과정을 구현하고 그 과정에서 기울기 정보에 따라 가중치를 직접 갱신해보는 일만 남았습니다.

## 학습 알고리즘 구현하기

R로 전체 학습 과정을 구현하기에 앞서 전체 학습이 어떻게 진행되는지 다시금 짚어보고 넘어가겠습니다. 먼저 학습은 신경망 모델에서 가중치와 편향 값을 기존의 랜덤한 초기값에서 훈련데이터를 입력받아 그것에 맞게 점차 갱신시켜 나가는 과정을 말합니다. 이제 해야할 것은 훈련데이터를 가져오는 것입니다. 훈련데이터를 가져오면 훈련시킬 데이터를 미니배치하여 일부분 가져옵니다.

데이터가 준비되면 각 가중치와 편향 별로 기울기를 구합니다. 기울기가 구해지면 기울기 정보를 토대로 확률적 경사하강법을 사용해 가중치와 편향을 갱신하며 이 과정을 반복합니다. 남은 것은 직접 2층 신경망 분류모델을 R로 구현하고 학습과 훈련을 해보는 일입니다. 먼저 초기 값을 설정하는 함수를 작성합니다.  

```{r}
TwoLayerNet  <- function(input_size, hidden_size, output_size, weight_init_std = 0.01) {
  W1 <<- weight_init_std*matrix(rnorm(n = input_size*hidden_size), nrow = input_size, ncol = hidden_size)
  b1 <<- matrix(rep(0,hidden_size),nrow=1,ncol=hidden_size)
  W2 <<- weight_init_std*matrix(rnorm(n = hidden_size*output_size), nrow = hidden_size, ncol = output_size)
  b2 <<- matrix(rep(0,output_size),nrow=1,ncol=output_size)
  return(list(input_size, hidden_size, output_size,weight_init_std))
}

init <- TwoLayerNet(input_size=784, hidden_size=50, output_size=10)
```

초기값을 설정하는 함수는 입력데이터의 크기와 은닉층의 크기, 분류 기준의 개수를 입력받아 설정합니다. 이 값들은 모두 가중치와 편향 행렬의 크기를 정하는데 사용됩니다. 첫 번째 가중치 행렬은 입력데이터의 크기를 행으로하고, 은닉층의 크기를 열로 지정하여 만듭니다. 입력데이터의 크기가 행이 되는 이유는 행렬 곱셈을 위해 맞추기 위함입니다. 예를 들어 손글씨 데이터 셋의 입력데이터 행렬의 크기는 입력데이터의 개수가 행이고 열은 784(이미지의 크기 784==28^2)로 나타납니다.

마찬가지로 다음층의 행렬 계산을 위해 은닉층의 크기가 첫 번째 가중치 행렬의 열이 됩니다. 두 번째 가중치 행렬도 마찬가지로 만들어지는데 여기서는 2층 신경망이므로 은닉층의 크기와 출력의 개수로 구성됩니다. 이렇게 구성하는 이유 역시 행렬 곱셈을 위해 크기를 맞추어 주는 것입니다. 참고로 은닉층이 2개라면 첫 번째 은닉층의 크기와 두 번째 은닉층의 크기로 구성됩니다.

가중치 행렬의 원소 값은 랜덤한 정규분포 난수로 설정합니다. 초기값의 설정은 이후 모델의 성능을 좌우할만큼 중요하지만 여기서는 난수를 초기값으로 설정한다는 것만 짚고 넘어가겠습니다. 모델 성능에 영향을 줄 수 있는 초기값 설정 방법들은 추후에 설명하겠습니다.   

첫 번째 편향 행렬 크기를 알아보기 전에, 편향은 모든 층에서 항상 1개의 노드만 가진다는 것을 기억해 봅시다. 즉 모든 편향 행렬의 행은 1이라는 말로 바꿀 수 있습니다. 편향 행렬은 가중치와 입력 신호가 곱해진 행렬에 더해지는데 계산을 위해 행렬의 행은 정해져 있으니 편향 행렬의 열을 맞추어 주어야 합니다.

두 번째 편향 행렬도 마찬가지로 출력의 개수를 열로 하여 행렬 덧셈을 위한 크기를 맞춰줍니다. 참고로 2층 신경망이 아닌 다층 신경망의 경우 편향 행렬의 열 크기는 각 층의 은닉층의 크기가 됩니다. 편향 행렬의 원소는 모든 층에서 초기값 0으로 시작됩니다. 그 이유는 말 그대로 어떠한 편향도 없는 상태에서 시작하기 위함입니다. 이렇게 편향 행렬까지의 설정을 함수화하고 실행하면, 가중치와 편향 행렬이 준비됩니다.

다음 단계는 훈련데이터와 테스트데이터 셋을 가져옵니다. 이전 글에서 만든 필요한 함수들은 functions, utils 라는 스크립트에 있습니다. 만든 함수를 가져오는 것은 `source()` 함수를 통해 불러올 수 있습니다.

```{r}
source("./DeepLearningFromForR/utils.R")
library(dslabs)

mnist<-read_mnist()
x_train<-mnist$train$images
t_train<-mnist$train$labels
x_test<-mnist$test$images
t_test<-mnist$test$labels

t_train_onehotlabel <- making_one_hot_label(t_train,60000,10)
t_test_onehotlabel <- making_one_hot_label(t_test,10000,10)

x_train_normalize <- x_train/255
x_test_normalize <- x_test/255

```

불러온 함수들을 확인합니다.

```{r}
sigmoid
softmax
cross_entropy_error
```

이제 신경망 모델의 계산을 담당하는 함수들을 구현해 봅시다.

```{r}
source("./DeepLearningFromForR/functions.R")
source("./DeepLearningFromForR/numerical_gradient.R")

model.forward <- function(x){
    z1 <- sigmoid(sweep((x %*% W1),2, b1,'+'))
    return(softmax(sweep((z1 %*% W2),2, b2,'+')))
}

loss <-function(x,t){
    return(cross_entropy_error(model.forward(x),t))
}

numerical_gradient_W1 <- function(f,x,t){
    h <- 1e-4
    vec <- matrix(0, nrow = nrow(W1) ,ncol = ncol(W1))
    for(i in 1:length(W1)){
        origin <- W1[i]
        W1[i] <<- (W1[i] + h)
        fxh1 <- f(x, t)
        W1[i] <<- (W1[i] - (2*h))
        fxh2 <- f(x, t)
        vec[i] <- (fxh1 - fxh2) / (2*h)
        W1[i] <<- origin
    }
    return(vec)
}

numerical_gradient <- function(f,x,t) {
    grads  <- list(W1 = numerical_gradient_W1(f,x,t),
                   b1 = numerical_gradient_b1(f,x,t),
                   W2 = numerical_gradient_W2(f,x,t),
                   b2 = numerical_gradient_b2(f,x,t))
    return(grads)
}

```

순전파를 구현한 `model.forward` 함수에는 `sigmoid` 함수와 `softmax` 함수가 포함되어 있기에 따로 호출할 필요는 없습니다. `model.forward` 함수는 복잡해 보이지만, 이 함수는 데이터를 입력받아 모델의 분류 기준에 대한 추정 값을 출력하는 역할을 합니다.

물론 모델의 실질적인 계산은 loss 함수에서 이루어지는데, 이 함수에서 `model.forward` 함수와 `cross_entropy_error` 함수가 적용됩니다.

numerical_gradient 함수는 W1~b2까지 4가지가 있습니다. 이 함수들은 모두 구조는 같고, 기울기를 구하는 객체만 다른 함수들입니다. 함수는 대표적으로 W1만 새로 구현했습니다만, 전체 함수는 `source()` 함수를 통해 불러오기 때문에 작동에는 이상이 없습니다. 기울기를 구하는 방법은 다음과 같이 진행됩니다.

```{r}
W1[i] <<- (W1[i] + h)
fxh1 <- f(x, t)
W1[i] <<- (W1[i] - (2*h))
fxh2 <- f(x, t)
vec[i] <- (fxh1 - fxh2) / (2*h)
```

그 다음 다시 원래 값으로 복원(` W1[i] <<- origin`)해주고 반복합니다. 이 과정을 각 가중치와 편향 행렬의 모든 원소에 적용해 줍니다. 이렇게 각 가중치와 편향 행렬들의 기울기 값들을 묶어서 하나의 함수로 만든 것이 `numerical_gradient` 함수입니다.

이제 함수들이 준비됬으니 직접 이미지 100개씩 배치 학습을 진행해 봅시다.

```{r}
learning_rate <- 0.1
iters_num <- 100
train_loss_list <- data.frame(lossvalue=rep(0,iters_num))
train_size <- dim(x_train_normalize)[1]
batch_size <- 100

for(i in 1:iters_num){
  batch_mask <- sample(train_size,batch_size)
  x <- x_train_normalize[batch_mask,]
  t <- t_train_onehotlabel[batch_mask,]
  grads <- numerical_gradient(loss,x,t)
  W1 <- W1 - (grads$W1 * learning_rate)
  W2 <- W2 - (grads$W2 * learning_rate)
  b1 <- b1 - (grads$b1 * learning_rate)
  b2 <- b2 - (grads$b2 * learning_rate)
  loss_value <- loss(x, t)
  train_loss_list[i,1] <- loss_value
  print(i)
}
```

for문 안에 각 가중치와 편향을 갱신하는 과정을 확인할 수 있습니다. `W1 <- W1 - (grads$W1 * learning_rate)` 이렇게 기울기와 학습률을 곱한 것을 기존의 가중치에서 빼줌으로서 가중치와 편향 행렬을 새롭게 갱신합니다. 손실 함수의 값 또한 지속적으로 저장하여 이후 반복문이 종료된 후 손실 함수 값이 어느정도 떨어졌는지 확인할 수 있습니다.

이 코드를 실행시키기 전에 주의할 점이 있습니다. 이 코드는 실행시간이 약 8시간 정도 걸립니다. 따라서 직접 학습을 진행할 경우 엄청난 시간이 소모됩니다. 참고로 600번 즉 1에폭의 학습을 한 가중치와 편향의 결과를 아래와 같이 읽어와 테스트 할 수 있습니다.

```{r}
library(rhdf5)
init <- H5Fopen("./weights_600.h5")
model <- list(matrix(init$"W1",784,50), matrix(init$"b1",1,50), matrix(init$"W2",50,10), matrix(init$"b2",1,10))
names(model) <- c("W1", "b1", "W2", "b2")
W1 <- model$W1
W2 <- model$W2
b1 <- model$b1
b2 <- model$b2
init$losslist
```

1 에폭을 돌렸을 때 가중치와 편향 값을 불러오면, 정확도 성능이 약 78% 정도인 것을 확인할 수 있습니다. 또한 손실 함수 값이 점차적으로 하향하는 그래프를 다음과 같이 그려볼 수 있습니다.

```{r}
model.evaluate(x_test_normalize,t_test_onehotlabel)
```

다음은 역전파 알고리즘을 이용해 학습 과정을 약 1000배 정도 시간을 줄이고 성능도 더 많이 좋아지는 모델을 만들어 볼 것입니다.  
