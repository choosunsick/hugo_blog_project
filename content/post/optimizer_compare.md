---
title: "Optimizer_compare"
date: 2020-08-12T18:26:14+09:00
draft: FALSE
tags: ["R로 딥러닝하기", "신경망", "옵티마이저"]
categories: ["R"]
---

## 옵티마이저(optimizer)란 ?

이번 글에서는 가중치와 편향을 갱신하는 방법을 소개합니다. 기존의 방식과 다른 방식으로 가중치와 편향을 갱신할 수 있으며 이 방법들은 더 나은 성능의 신경망 모델을 만드는데 도움이 됩니다. 이에 따라 momentum, adagrad, Rmsprop, Adam 이라는 4가지의 새로운 방식의 가중치와 편향의 갱신 방법을 소개합니다.

옵티마이저란 가중치와 편향을 갱신하는 방법을 말합니다. 신경망 모델의 목표는 손실 함수의 최소화 입니다. 손실함수를 최소화시키는 방향으로 가중치와 편향을 갱신해 나가는 것이 신경망 모델이 학습하는 과정에서 발생됩니다. 다시 말해 옵티마이저는 신경망 모델에서 가중치와 편향 변수를 최적화 시키는 과정이라 생각할 수 있습니다.

이 글에서 소개할 옵티마이저의 종류는 5개로 SGD, Momentum, Adagrad, Rmsprop, Adam 입니다.

앞서 가중치와 편향을 갱신할 때 기존에는 단순한 확률적 경사하강법(SGD)을 이용했습니다. 확률적 경사하강법은 앞서 순전파의 작동에서 설명한 적이 있습니다. 그에 대한 설명은 [다음 링크](https://choosunsick.github.io/post/neural_network_4/)에서 찾아 보실 수 있습니다. 기존의 갱신 과정을 함수화하면 다음과 같습니다.

```R
sgd.update <- function(network, grads, lr = 0.01){
  for(i in names(network)){network[[i]] <- network[[i]] - (grads[[i]]*lr)}
  return(network)
}
```

SGD를 사용했을 때 단점은 무엇일까요? SGD는 손실함수의 기울기가 무작정 낮아지는 방향으로 진행되기 때문에 손실함수의 기울기가 최저값이 되는 값을 찾는데 비효율적인 탐색경로를 지니게 됩니다. 즉 손실함수의 기울기 값을 찾아가는 과정에서 손실함수의 값이 커졋다 작아졌다를 반복하는 지그재그 모양의 탐색경로를 지니게 됩니다. 이에 따라 탐색 경로가 비효율적이라는 것입니다.  

![SGD 탐색 경로](https://user-images.githubusercontent.com/19144813/81788907-3355e680-953e-11ea-8f06-365f4e9e7785.png)

### momentum 방식 소개

이제 SGD 보다 효율적인 갱신 방식들을 소개합니다. 먼저 momentum입니다. 모멘텀은 운동량을 뜻하는 단어로 물리와 관련됩니다. 식으로 보면 $$v = av - 학습률*손실함수의 기울기  $$ 와 $$W = W+v $$입니다. 단순히 말하면 속도라는 변수를 추가해 가중치와 편향에 더해 주는 것입니다. 위의 식은 기울기 방향으로 물체가 가속한다는 의미를 지닙니다. 위 식에서 W는 가중치와 편향 변수를 지칭하며, a의 값은 0.9 정도로 설정되며, av는 가속도에 대한 저항으로 생각할 수 있습니다. 기울기 방향으로 가속하기에 기존의 SGD보다 효율적인 탐색경로를 보이게 됩니다. 즉 탐색경로가 SGD보다 덜 지그재그모양으로 나타나게 됩니다.  

momentum의 R 구현은 다음과 같습니다.

```
optimizer <- list(Momentum=NULL, AdaGrad=NULL, Rmsprop=NULL, Adam=list("iter"=0,"m"=NULL,"v"=NULL))

momentum.update <- function(network, grad,v, lr = 0.01, momentum=0.9){
  if (is.null(v) == TRUE) {
    v <- rep(list(NA),NROW(network))
    names(v) <- names(network)
    for(i in 1:NROW(network)){
      v[[i]] <- matrix(0,dim(network[[i]])[1],dim(network[[i]])[2])
    }
  }
  for(i in 1:NROW(network)){
    optimizer$Momentum[[i]] <<- v[[i]]*momentum - (lr*grad[[i]])

    network[[i]] <- network[[i]]+optimizer$Momentum[[i]]}
  names(optimizer$Momentum) <- names(network)
  return(network)
}
```
먼저 v 값을 계속 갱신해야하기에 저장할 저장소 역할의 리스트를 만들어 줍니다. 이 리스트는 차후 다른 갱신 방법들에서도 사용됩니다. 여기서 조심해야할 점은 v는 값이 아니라는 점입니다. v는 리스트로 가중치와 편향의 변수의 수 만큼 만들어 집니다. 예를 들면 2층 신경망일 경우 2개의 가중치와 2개의 편향이 있으므로 v는 총 4개의 행렬로 이루어진 리스트가 됩니다. v는 가중치와 편향의 개수가 더 많은 심층 신경망의 경우 가중치와 편향의 개수와 같은 개수를 지닌 리스트가 됩니다. v의 초기값은 0으로 이루어진 행렬 주어집니다. 물론 그 행렬의 크기는 가중치나 편향 행렬의 크기와 같은 행렬이 됩니다. 이후 v값을 순서대로 갱신하고 나면 가중치와 편향 값의 갱신이 공식에서와 같이 이루어집니다.

### Adagrad 방식

신경망 학습에서 학습률은 중요합니다. 학습률이 너무 높거나 너무 낮으면 학습이 원활하게 진행되지 않는데 이를 앞서 확인할 수 있었습니다. 학습률과 관련된 내용은 [다음 링크](https://choosunsick.github.io/post/neural_network_4/) 에서 다시 확인하실 수 있습니다.
Adagrad 방식은 학습률 감소 기법과 관련됩니다. 학습률 감소란 학습을 진행하면서 학습률을 점차 줄여나가는 기법입니다. 학습 처음에는 크게 학습하다가 나중에는 적게 학습한다는 것으로 이 기법은 신경망 학습에 자주 사용되는 기법입니다.

학습률을 서서히 낮추는 방법 중 가장 간단한 방법은 가중치와 편향 변수 전체의 학습률 값을 일괄적으로 낮추는 것입니다. adagrad는 일괄적으로 낮추는 대신 좀 더 발전시켜서 각각의 가중치와 편향 변수에 맞추어 학습률 값을 만들어줍니다. adagrad의 갱신 방법을 수식으로 살펴보면 다음과 같습니다.

$$h = h + 손실함수의 기울기^2 $$ 와 $$ W = W - (학습률*손실함수의 기울기 / √h+1e-7$$ 입니다. 여기서 분모에 1e-7을 더하는 이유는 0으로 나누는 사태를 방지하기 위해서 입니다. 위 식에서 √h로 나누는 것은 가중치와 편향 변수의 원소 중에 값이 크게 갱신된 경우 학습률이 낮아지게 만들기 위해서입니다. 즉 가중치와 편향의 원소 별로 학습률 감소의 원리가 적용된다는 것입니다. adagrad의 R 구현은 다음과 같습니다.

```
AdaGrad.update <- function(network,grad,h,lr=0.01){
  if (is.null(h) == TRUE) {
    h <- rep(list(NA),NROW(network))
    names(h) <- names(network)
    for(i in 1:NROW(network)){
      h[[i]] <- matrix(0,dim(network[[i]])[1],dim(network[[i]])[2])
    }
  }

  for(i in 1:NROW(network)){
    optimizer$AdaGrad[[i]]  <<- h[[i]] + grad[[i]]^2
    network[[i]] <- network[[i]] - (lr*grad[[i]] / (sqrt(optimizer$AdaGrad[[i]])+1e-7))
  }
  names(optimizer$AdaGrad) <- names(network)
  return(network)
}
```

위 코드를 보면 `momentum.update()`의 v의 초기값을 만드는 과정이 h 초기값을 만드는 과정이 똑같다는 것을 알 수 있습니다. 같은 방식으로 h 값을 설정해주고 나면 이제 위 식을 코드로 구현한 과정이 있습니다. 단 Adagrad의 경우 주의해야할 점이 있는데 손실함수의 기울기 값을 계속 제곱하기에 무한히 계속 학습을 반복할 경우 갱신강도가 약해져 갱신량이 0에 수렴하게 됩니다. 이런 문제점을 해결한 것이 Rmsprop 방식입니다.

Rmsprop은 기존의 adagrad 공식에서 베타란 값을 추가하고 지수이동평균 방법을 사용하여 adagrad 방식의 한계점을 극복합니다. 참고로 Rmsprop 방식의 R 코드 구현은 다음과 같습니다.

```
Rmsprop.update <- function(network, grad, h, lr=0.01, beta=0.9){
  if (is.null(h) == TRUE) {
    h <- rep(list(NA),NROW(network))
    names(h) <- names(network)
    for(i in 1:NROW(network)){
      h[[i]] <- matrix(0,dim(network[[i]])[1],dim(network[[i]])[2])
    }
  }
  for(i in 1:NROW(network)){
    optimizer$Rmsprop[[i]]  <<- (beta * h[[i]]) + (1.0 - beta)*(grads[[i]] * grads[[i]])
    network[[i]] <- network[[i]] - (lr * grads[[i]]) / (sqrt(optimizer$Rmsprop[[i]])+ 1e-7)
  }
  names(optimizer$Rmsprop) <- names(network)
  return(network)
}
```

### Adam

Adam 기법은 momentum 기법과 Rmsprop 기법을 혼합한 기법입니다. 이에 따라 Adam에서는 속도 파라미터 m (momentum에서는 v로 표시한 것)과 속도 감소 파라미터 v(adagrad에서는 h로 표시)를 필요로 합니다. 다른 기법들과 다르게 두 가지의 인자가 필요해서 코드와 수식이 조금 더 복잡해집니다. Adam과 관련된 내용은 관련된 원논문을 [다음 링크](https://arxiv.org/pdf/1412.6980.pdf)에서 확인하실 수 있습니다. Adam을 R로 구현하면 다음과 같습니다.

```
Adam.update <- function(network,grads,iter,m,v,lr=0.001,beta1=0.9,beta2=0.999){
  if(is.null(m) == TRUE){
    m <- rep(list(NA),NROW(network))
    v <- rep(list(NA),NROW(network))
    names(m) <- names(network)
    names(v) <- names(network)
    for(i in 1:NROW(network)){
      m[[i]] <- matrix(0,dim(network[[i]])[1],dim(network[[i]])[2])
      v[[i]] <- matrix(0,dim(network[[i]])[1],dim(network[[i]])[2])
    }
  }
  optimizer$Adam$iter <<- iter+1
  lr_t <- (lr*sqrt(1.0 - beta2^optimizer$Adam$iter))/ (1.0 - beta1^optimizer$Adam$iter)
  temp_m_list <- rep(list(NA),NROW(network))
  temp_v_list <- rep(list(NA),NROW(network))
  for(i in 1:NROW(network)){
    temp_m_list[[i]] <- m[[i]] + (1 - beta1) * (grads[[i]] - m[[i]])
    temp_v_list[[i]] <- v[[i]] + (1 - beta2) * (grads[[i]]^2 - v[[i]])
    network[[i]] <- network[[i]] - (lr_t * temp_m_list[[i]]/ (sqrt(temp_v_list[[i]]) + 1e-7))
  }
  optimizer$Adam$m <<- temp_m_list
  optimizer$Adam$v <<- temp_v_list
  return(network)
}
```

## 옵티마이저(optimizer) 비교

이제 각 옵티마이저 간 학습을 통해 성능을 비교해봅시다.

먼저 역전파를 작동시키는 데 필요한 함수를 불러옵니다.  

```
# install.packages("dslabs") 이미 설치한 적이 있으면 생략합니다.
library(dslabs)
source("./layers.R")
source("./utils.R")
source("./optimizer.R")
```

MNIST 자료를 가져오는 방법에 대한 내용은 [Mnist 손글씨 데이터 읽어오는 패키지 소개](https://choosunsick.github.io/post/mnist/)을 참고한다. 자료를 가져오는 코드는 아래와 같다. 아래 코드에 대한 소개는 다음을 참고한다.

```

mnist_data <- get_data()

x_train_normalize <- mnist_data$x_train
x_test_normalize <- mnist_data$x_test

t_train_onehotlabel <- making_one_hot_label(mnist_data$t_train,60000, 10)
t_test_onehotlabel <- making_one_hot_label(mnist_data$t_test,10000, 10)
```

학습할 네트워크를 만들어줍니다. 사용하는 활성화 함수에 따라 적절한 초기값을 선택합니다. 기존에는 초기값을 임의의 정규분포 난수 값에 0.01을 곱한 것을 사용했습니다. 이것을 사용했던 이유는 가능한 초기값을 작은 값에서 시작하기 위해서 였습니다. 그러나 가장 작은 값으로 하기위해 초기값을 0으로 설정한다면 문제가 발생합니다. 가중치가 0 혹은 균일한 값이 된다면, 모든 가중치가 똑같이 갱신되는 일이 발생합니다. 이렇게 될 경우 층을 여러개로 하는 의미가 사라집니다. 이에 따라 가중치 초기값은 무작위 값이 필요하며 활성화 함수에 따라 적절한 초기값을 사용하는게 좋습니다.

적절한 초기값에는 S자 활성화 함수의 경우 xavier 초기값을 ReLU 함수의 경우 he 초기값을 선택합니다. xavier 초기값이란 사비에르 글로로트와 요슈아 벤지오가 논문에서 권장하는 가중치 초깃값으로 가중치 초기값을 고르게 분포하게 만드는 방법입니다. 데이터가 치우치지 않게 하기 때문에 학습이 효율적으로 이루어지게 됩니다. he 초기값이란 카이밍 히라는 사람이 찾아낸 ReLU 함수에 특화된 초기값입니다. 마찬가지로 ReLU 활성화 함수를 사용할 때 초기값을 넓게 분포 시키기 위한 방법입니다.

각 방법에 대한 R 구현은 `make_weight_init()` 함수가 담당합니다.

```
TwoLayerNet <- function(input_size, hidden_size, output_size, weight_init_std  =  0.01) {
  W1 <- weight_init_std * matrix(rnorm(n  =  input_size*hidden_size), nrow  =  input_size, ncol  =  hidden_size)
  b1 <- matrix(rep(0,hidden_size), nrow = 1, ncol = hidden_size)
  W2 <- weight_init_std * matrix(rnorm(n  =  hidden_size*output_size), nrow  =  hidden_size, ncol  =  output_size)
  b2 <- matrix(rep(0,output_size),nrow = 1, ncol = output_size)
  network <- list(W1 = W1, b1 = b1, W2 = W2, b2 = b2)

  return(network)
}

make_weight_init  <- function(input_size, hidden_size, weight_init_std=0.01){
  if (weight_init_std == "xavier"){
    return(sqrt(1.0 / input_size) * matrix(rnorm(n = input_size * hidden_size), nrow = input_size, ncol =  hidden_size))
  }
  else if (weight_init_std == "he") {
    return(sqrt(2.0 / input_size) * matrix(rnorm(n = input_size * hidden_size), nrow = input_size, ncol  =  hidden_size))
  }
  else {
    return(weight_init_std * matrix(rnorm(n  =  input_size*hidden_size), nrow  =  input_size, ncol  =  hidden_size))
  }
}

network <- TwoLayerNet(input_size = 784, hidden_size = 50, output_size = 10)
network$W1 <- make_weight_init(784,50,"he")
network$W2 <- make_weight_init(50,10,"he")
```

`make_weight_init()` 함수를 사용하여 ReLU 활성화 함수에 맞는 he 초기값을 가진 가중치 변수를 새로 만들어 줍니다.

이제 옵티마이저를 변경해가면서 학습할 차례입니다. 학습에 필요한 변수들을 만들어줍니다.
학습을 위해 필요한 함수를 새롭게 작성합니다. 기존에 사용하던 함수와 차이점은 인자에 forward 함수를 넣어서 사용한다는 점 외에는 없습니다.

```
forward <- function(x){
  Affine_1 <- Affine.forward(network$W1, network$b1, x)
  Relu_1 <- Relu.forward(Affine_1$out)
  Affine_2 <- Affine.forward(network$W2, network$b2, Relu_1$out)
  return(list(x = Affine_2$out, Affine_1.forward = Affine_1, Affine_2.forward = Affine_2, Relu_1.forward = Relu_1))
}

loss <- function(model.forward, x, t){
  temp <- model.forward(x)
  y <- temp$x
  last_layer.forward <- SoftmaxWithLoss.forward(y, t)
  return(list(loss = last_layer.forward$loss, softmax = last_layer.forward, predict =  temp))
}


gradient <- function(model.forward, x, t) {
  # 순전파
  temp <- loss(model.forward, x, t)
  # 역전파
  dout <- 1
  last.backward <- SoftmaxWithLoss.backward(temp$softmax, dout)
  Affine_2.backward <- Affine.backward(temp$predict$Affine_2.forward, dout  =  last.backward$dx)
  Relu_1.backward <- Relu.backward(temp$predict$Relu_1.forward, dout  =  Affine_2.backward$dx)
  Affine_1.backward <- Affine.backward(temp$predict$Affine_1.forward, dout  =  Relu_1.backward$dx)
  grads  <- list(W1  =  Affine_1.backward$dW, b1  =  Affine_1.backward$db, W2  =  Affine_2.backward$dW, b2  =  Affine_2.backward$db)
  return(grads)
}

model.evaluate <- function(model,x,t){
    temp <- model(x)
    y <- max.col(temp$x)
    t <- max.col(t)
    accuracy <- (sum(ifelse(y == t,1,0))) / dim(x)[1]
    return(accuracy)
}
```
이제 옵티마이저를 선택하는 함수를 만들어 줍니다.

```
get_optimizer <- function(network,grad,name){
  if(name=="SGD"){
    return(sgd.update(network,grad))
  }
  else if(name=="momentum"){
    return(momentum.update(network,grad,optimizer$Momentum))
  }
  else if(name=="adagrad"){
    return(AdaGrad.update(network,grad,optimizer$AdaGrad))
  }
  else if(name=="Rmsprop"){
    return(Rmsprop.update(network,grad,optimizer$Rmsprop))
  }
  else{
    return(Adam.update(network, grad, optimizer$Adam$iter,m = optimizer$Adam$m, v = optimizer$Adam$v))
  }
}
```
이 함수는 입력된 옵티마이저 이름에 따라 적절한 옵티마이저를 선택해 갱신을 진행합니다.
이제 직접 옵티마이저 별로 학습을 진행해 봅시다.

기존의 훈련 방법을 함수화 해줍니다. 기존에 for문과 외부 변수를 통해 학습을 진행하던 과정을 함수로 묶어 만들어줍니다. 단 이 과정에서 모델의 정확성을 판단하는 것을 에폭이 아닌 반복문에 횟수에 따라 정확성을 평가하고 저장하기에 약 하나의 옵티마이저당 30분 정도의 시간이 좀 걸리게 됩니다.  

```
train_model <- function(batch_size, iters_num, optimizer_name){
  train_size <- dim(x_train_normalize)[1]
  train_loss_list <- data.frame(lossvalue  =  rep(0,iters_num))
  train_acc_list <- data.frame(train_acc  =  rep(0,iters_num))
  test_acc_list <- data.frame(test_acc  =  rep(0,iters_num))
  for(i in 1:iters_num){
    batch_mask <- sample(train_size ,batch_size)
    x_batch <- x_train_normalize[batch_mask,]
    t_batch <- t_train_onehotlabel[batch_mask,]

    grad <- gradient(model.forward=forward, x_batch, t_batch)

    network <<- get_optimizer(network,grad,optimizer_name)

    train_loss_list[i,1] <- loss(forward ,x_batch, t_batch)$loss
    test_acc_list[i,1]  <- model.evaluate(forward, x_test_normalize, t_test_onehotlabel)
  }
  return(data.frame(train_loss_list=train_loss_list,test_acc_list=test_acc_list))
}
```

이제 5가지 옵티마이저에 따른 2000번의 학습을 진행합니다. 참고로 매번 학습을 진행할 때 초기값을 재설정해주어야 학습이 이어서 진행되지 않습니다.

```
sgd_result <- train_model(100, 2000, "SGD")
momentum_result <- train_model(100, 2000, "momentum")
adagrad_result <- train_model(100, 2000, "adagrad")
Rmsprop_result <- train_model(100, 2000, "Rmsprop")
adam_result <- train_model(100, 2000, "adam")
```

그래프를 통해 결과를 비교해 봅시다. 아래의 코드로 그림을 그려 볼 수 있습니다.

```
ggplot(data = sgd_result)+geom_point(aes(x=1:2000,y=lossvalue,color="sgd"))+geom_point(data = momentum_result,aes(x=1:2000,y=lossvalue,color="momentum"))+geom_point(data = adagrad_result,aes(x=1:2000,y=lossvalue,color="adagrad"))+geom_point(data = Rmsprop_result,aes(x=1:2000,y=lossvalue,color="Rmsprop"))+geom_point(data = adam_result,aes(x=1:2000,y=lossvalue,color="adam"))
```

그림을 보시면 SGD의 성능이 가장 좋지 않은 것을 알 수 있습니다. 또한 Rmsprop 과 Adam 의 경우가 손실 값이 가장 많이 떨어지는 것을 확인할 수 있습니다.

![손실값 비교]((https://user-images.githubusercontent.com/19144813/90000460-28405d00-dccb-11ea-942f-33f36553d1e6.png))

테스트 셋에 대한 정확도 역시 그래프로 비교합니다. 아래의 코드로 그림을 그려볼 수 있습니다.

```
ggplot(data = sgd_result)+geom_point(aes(x=1:2000,y=test_acc,color="sgd"))+geom_point(data = momentum_result,aes(x=1:2000,y=test_acc,color="momentum"))+geom_point(data = adagrad_result,aes(x=1:2000,y=test_acc,color="adagrad"))+geom_point(data = Rmsprop_result,aes(x=1:2000,y=test_acc,color="Rmsprop"))+geom_point(data = adam_result,aes(x=1:2000,y=test_acc,color="adam"))
```

마찬가지로 그림을 보시면 SGD의 정확도가 가장 낮은 것을 알 수 있습니다. 반대로 Rmsprop 과 Adam 의 경우에 높은 정확도를 보이는 것을 확인할 수 있습니다.

![정확도 비교](https://user-images.githubusercontent.com/19144813/90000743-80775f00-dccb-11ea-9e1c-fa511bb97f3f.png)

결론적으로 같은 조건일 때 모델의 성능을 가장 좋게 만드는 옵티마이저는 Rmsprop 과 Adam 입니다. 
