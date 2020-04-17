---
title: "Chapter4 오차역전파법을 적용하기"
date: 2020-03-30T22:07:37+09:00
draft: FALSE
mathjax: TRUE
tags: ["R로 딥러닝하기", "신경망", "역전파"]
categories: ["R"]
---

이 시리즈는 R로 딥러닝을 구현하고 설명하는 것에 목표를 둔 글입니다. 신경망 챕터에 이어서 3번째 순서입니다. [이전 글](https://choosunsick.github.io/post/neural_network_backward_3/)과 내용이 이어집니다. 이전 글들은 [이곳](https://choosunsick.github.io/post/contents_list/)에서 찾아 보실 수 있습니다.

## 오차역전파법을 적용한 2층 신경망 구현하기

이제 오차역전파법이 적용된 2층 신경망을 구현해보겠습니다. 역전파가 적용된 신경망 모델 역시 기존 순전파에서 진행한 학습 과정은 같습니다. 그렇지만 다시한번 전체 학습 과정을 살펴보면 다음과 같습니다. 학습 목표는 순전파와 동일하게 손실 함수의 값을 줄이고 최적의 가중치와 편향 값을 찾는 것입니다.

먼저 모델에 적용할 훈련데이터를 준비합니다. 훈련데이터 일부를 무작위로 가져와 학습에 사용하는 방법을 미니배치라고 하며, 이 방법을 사용할 것입니다. 다음으로 모델에서 무작위로 만들어진 가중치와 편향에 대한 기울기를 구합니다. 단 역전파에서는 수치미분이 아닌 오차역전파 방식으로 기울기를 구하게 됩니다. 다음으로는 가중치와 편향을 손실 함수 값이 작아지는 방향으로 갱신합니다. 이제 손실함수 값이 최소값이 되도록 이 과정을 반복해줍니다.

오차역전파 법을 적용한 2층 신경망을 직접 구현해봅시다. 첫번째 구현은 가중치와 편향을 랜덤으로 만들어주는 함수입니다.

```{r}

TwoLayerNet <- function(input_size, hidden_size, output_size, weight_init_std = 0.01) {
    W1 <- weight_init_std*matrix(rnorm(n = input_size*hidden_size), nrow = input_size, ncol = hidden_size)
    b1 <- matrix(rep(0,hidden_size),nrow = 1,ncol = hidden_size)
    W2 <- weight_init_std*matrix(rnorm(n = hidden_size*output_size), nrow = hidden_size, ncol = output_size)
    b2 <- matrix(rep(0,output_size),nrow = 1,ncol = output_size)
    params <<- list(W1 = W1, b1 = b1, W2 = W2, b2 = b2)
    return(list(input_size, hidden_size, output_size,weight_init_std))
}

> TwoLayerNet(input_size = 784, hidden_size = 50, output_size = 10)

[[1]]
[1] 784

[[2]]
[1] 50

[[3]]
[1] 10

[[4]]
[1] 0.01

```

이 함수는 구현하는 신경망의 층수에 따라 구현이 달라지는데 이 코드는 2층 신경망을 기준으로 구현했습니다. 2층 신경망은 2개의 가중치와 편향이 필요합니다. 기존 순전파의 초기값 설정과 다를바가 없습니다. 차이점은 각 층의 가중치와 편향 행렬을 리스트로 묶어 저장한다는 점 외에는 없습니다.

다음으로는 순전파 계산 과정에 필요한 함수들의 구현을 알아보겠습니다. 역전파는 먼저 순전파 계산이 진행된 후에 계산이 진행됩니다.

순전파와 역전파 계산에 필요한 함수는 활성화 함수인 `ReLU()`와 순전파에서 가중치 행렬와 편향 행렬의 계산을 하는 `affine()` 계층,  `softmax`와 `loss` 함수를 같이 묶은 `softmaxwithloss()`입니다.

```{r}
source("./DeepLearningFromForR/functions.R")
```

역전파 계산에 앞서 순전파 계산에 필요한 것으로는 손실 함수 계산에 필요한 model.backward 함수가 있습니다. 이 함수에 대한 구현은 다음과 같습니다. 이 함수는 함수의 이름만 변경 되었을 뿐 사실상 계산 구조는 순전파에서와 같습니다.

입력 신호와 첫 번째 가중치 및 편향을 계산하고, 이후 활성화 함수인 Relu 함수를 사용하여 다음 층의 입력 신호로 전달합니다. 다음층에서 마찬가지로 입력 신호와 두 번째 가중치와 편향 행렬을 계산합니다. 본래 순전파에서는 이 값에 softmax 함수도 사용했었지만 여기서는 softmax 함수의 적용은 softmaxwithloss 함수로 빠져있습니다.   

```{r}
model.backward <- function(x){
    Affine_1_layer <- Affine.forward(params$W1, params$b1, x)
    Relu_1_layer <- Relu.forward(Affine_1_layer$out)
    Affine_2_layer <- Affine.forward(params$W2, params$b2, Relu_1_layer$out)
    return(list(x = Affine_2_layer$out, Affine_1.forward = Affine_1_layer, Affine_2.forward = Affine_2_layer, Relu_1.forward = Relu_1_layer))
}
```

손실 함수 계산 역시 순전파 때랑 크게 달라진게 없습니다. 단지 backward_loss 함수에서 softmax와 cross_entropy_error 함수를 적용하는 코드가 생겼을 뿐입니다.

```{r}
backward_loss <- function(x, t){
    temp <- model.backward(x)
    y <- temp$x
    last_layer.forward <- SoftmaxWithLoss.forward(y, t)
    return(list(loss = last_layer.forward$loss, softmax = last_layer.forward, predict =  temp))
}
```

다음으로는 순전파 계산과 가장 큰 차이가 있는 gradient 함수입니다. 기존의 순전파 계산은 backward_loss 함수를 사용하는 반면 역전파 계산은 마지막층에서부터 첫 번째 층으로 역순으로 올라갑니다.

```{r}
gradient <- function(x, t) {
    # 순전파
    temp_loss <- backward_loss(x, t)
    #역전파
    dout <- 1
    last_layer.backward <- SoftmaxWithLoss.backward(temp_loss$softmax, dout)
    Affine_2_layer.backward <- Affine.backward(temp_loss$predict$Affine_2.forward, dout = last_layer.backward$dx)
    Relu_1_layer.backward <- Relu.backward(temp_loss$predict$Relu_1.forward, dout = Affine_2_layer.backward$dx)
    Affine_1_layer.backward <- Affine.backward(temp_loss$predict$Affine_1.forward, dout = Relu_1_layer.backward$dx)

    grads  <- list(W1 = Affine_1_layer.backward$dW, b1 = Affine_1_layer.backward$db, W2 = Affine_2_layer.backward$dW, b2 = Affine_2_layer.backward$db)
    return(grads)
}
```

이제 구현이 끝났으니 코드의 작동과 기능을 확인할 차례입니다.

### 오차역전파법으로 구한 기울기 검증하기

오차역전파법으로 기울기를 구하는 방식을 구현했습니다. 이제 오차역전파법으로 구한 기울기가 정확한 값을 보이는지 기존의 수치미분 방식으로 구한 기울기 값과 비교해볼 차례입니다. 두 방식의 결과를 서로 비교하여 오차역전파법이 제대로 구현되었는지 확인하는 과정을 기울기 검증이라고 합니다. 기울기 검증 방법은 다음과 같습니다.

먼저 데이터를 불러옵니다.

```{r}
library(dslabs)
source("./DeepLearningFromForR/utils.R")
source("./DeepLearningFromForR/numerical_gradient.R")

mnist_data <- get_data()

making_one_hot_label <- function(t_label,nrow,ncol){
  data <- matrix(FALSE,nrow = nrow,ncol = ncol)
  t_index <- t_label+1
  for(i in 1:NROW(data)){
    data[i, t_index[i]] <- TRUE
  }
  return(data)
}
t_train_onehotlabel <- making_one_hot_label(mnist_data$t_train,60000,10)
t_test_onehotlabel <- making_one_hot_label(mnist_data$t_test,10000,10)

x_batch <- mnist_data$x_train[1:3,]

t_batch <- t_train_onehotlabel[1:3,]
```

기울기를 비교할 데이터는 훈련데이터를 3개씩 뽑아 비교에 사용합니다.

```{r}
W1 <- params$W1
b1 <- params$b1
W2 <- params$W2
b2 <- params$b2

model.backward.test <- function(x){
  Affine_1_layer <- Affine.forward(W1, b1, x)
  Relu_1_layer <- Relu.forward(Affine_1_layer$out)
  Affine_2_layer <- Affine.forward(W2, b2, Relu_1_layer$out)
  return(list(x  =  Affine_2_layer$out, Affine_1.forward  =  Affine_1_layer, Affine_2.forward  =  Affine_2_layer, Relu_1.forward  =  Relu_1_layer))
}

loss.test <- function(x,t){
  temp  <- model.backward.test(x)
  y <- temp$x
  last_layer.forward <- SoftmaxWithLoss.forward(y, t)
  return(last_layer.forward$loss)
}
```

순전파의 수치미분 값을 비교할 함수를 test란 이름으로 만들어줍니다. 이 함수들은  순전파에 사용하는 함수에 params 리스트 대신 객체 W1~b2로 변경한 것입니다. 이제 두 가지를 과정을 서로 비교해보겠습니다.

```{r}
grad_numerical <- numerical_gradient(loss.test,x_batch, t_batch)
grad_backprop <- gradient(x_batch, t_batch)

W1_diff <- mean(abs(grad_backprop$W1-grad_numerical$W1))
b1_diff <- mean(abs(grad_backprop$b1-grad_numerical$b1))
W2_diff <- mean(abs(grad_backprop$W2-grad_numerical$W2))
b2_diff <- mean(abs(grad_backprop$b2-grad_numerical$b2))

temp <- list(W1 = W1_diff, b1 = b1_diff, W2 = W2_diff, b2 = b2_diff)
> temp

$W1
[1] 4.020843e-10

$b1
[1] 2.226819e-09

$W2
[1] 5.757658e-09

$b2
[1] 1.400857e-07

```

먼저 수치미분 방식으로 기울기를 구해줍니다. 단 위에서 사용한 수치미분 방식에는 `sigmoid` 함수를 활성화 함수로 사용하지 않고 `Relu` 함수를 활성화 함수로 사용합니다. 또한, `softmax` 함수를 `cross_entropy_error` 함수와 함께 적용하는 순서의 차이 정도만 있습니다.

위의 코드를 통해 구한 기울기를 비교하면 0에 가까운 값이 도출됩니다. 곧 아주 미미한 차이가 나타남을 확인할 수 있습니다. 만약 기존의 시그모이드 함수를 사용한 순전파 과정으로 기울기를 구한다면 이러한 미미한 차이가 아닌 상당한 차이를 확인할 수 있기 때문에 주의해야합니다. 또한, 이 차이는 가중치의 랜덤값에 영향을 받기 때문에 비교 수치가 약간의 차이가 있을 수 있습니다.  

수치 미분 방식과 오차역전파법을 통해 구한 기울기의 차이가 0이되는 일은 드물게 발생합니다. 왜냐하면 컴퓨터의 계산의 정밀도에 한계가 있기 때문입니다. 두 방식의 기울기의 차가 크면 오차역전파법을 잘못 구현했다고 볼 수 있습니다. 이제 오차역전파법과 수치미분 방식의 기울기가 서로 큰 차이가 없음을 확인했기 때문에 오차역전파법 기능 구현에 문제가 없음을 확인했다고 할 수 있습니다.

### 오차역전파법 사용한 학습 구현하기

이제 오차역전파법을 통한 학습 과정을 알아보겠습니다. 학습과정은 순전파와 크게 다른 바가 없습니다. 차이점은 단지 역전파를 통해 기울기를 찾고 그 값으로 갱신한다는 점입니다. 먼저 필요한 객체들과, 훈련데이터 및 테스트셋을 준비합니다.

```{r}
source("./DeepLearningFromForR/utils.R")
library(dslabs)

mnist_data <- get_data()

model.evaluate.backward <- function(x,t){
    y <- max.col(model.backward(x)$x)
    t <- max.col(t)
    accuracy <- (sum(ifelse(y == t,1,0))) / dim(x)[1]
    return(accuracy)
}

x_train_normalize <- mnist_data$x_train
x_test_normalize <- mnist_data$x_test

t_train_onehotlabel <- making_one_hot_label(mnist_data$t_train,60000,10)
t_test_onehotlabel <- making_one_hot_label(mnist_data$t_test,10000,10)


iters_num <- 10000
train_size <- dim(x_train_normalize)[1]
batch_size <- 100
learning_rate <- 0.1

train_loss_list <- data.frame(lossvalue  =  rep(0,iters_num))
train_acc_list <- data.frame(train_acc  =  0)
test_acc_list <- data.frame(test_acc  =  0)

iter_per_epoch <- max(train_size / batch_size)
```

필요한 데이터가 준비되면 이제 모델의 학습을 시작할 수 있습니다. 모델 학습은 순전파 때와 다르게 시간이 적게걸립니다.

```{r}

for(i in 1:iters_num){
  batch_mask <- sample(train_size ,batch_size)
  x_batch <- x_train_normalize[batch_mask,]
  t_batch <- t_train_onehotlabel[batch_mask,]

  grad <- gradient(x_batch, t_batch)

  params$W1 <- params$W1 - (grad$W1 * learning_rate)
  params$W2 <- params$W2 - (grad$W2 * learning_rate)
  params$b1 <- params$b1 - (grad$b1 * learning_rate)
  params$b2 <- params$b2 - (grad$b2 * learning_rate)

  loss_value <- backward_loss(x_batch, t_batch)$loss
  train_loss_list <- rbind(train_loss_list,loss_value)

  if(i %% iter_per_epoch == 0){
    train_acc <- model.evaluate.backward(x_train_normalize, t_train_onehotlabel)
    test_acc <- model.evaluate.backward(x_test_normalize, t_test_onehotlabel)
    train_acc_list <- rbind(train_acc_list,train_acc)
    test_acc_list <- rbind(test_acc_list,test_acc)
    print(c(train_acc, test_acc))
  }
}
[1] 0.90545 0.91020
[1] 0.9216667 0.9218000
[1] 0.9349667 0.9357000
[1] 0.9445167 0.9444000
[1] 0.9505333 0.9468000
[1] 0.9557333 0.9523000
[1] 0.9605 0.9570
[1] 0.9639167 0.9603000
[1] 0.9664167 0.9630000
[1] 0.9696667 0.9635000
[1] 0.9710667 0.9655000
[1] 0.9725667 0.9671000
[1] 0.9736 0.9670
[1] 0.9752 0.9689
[1] 0.9767833 0.9689000
[1] 0.97675 0.96910

plot(x = 1:17,y = train_acc_list$train_acc,"l")
lines(x = 1:17,y = test_acc_list$test_acc,col = "red")
```
![정확도 곡선](https://user-images.githubusercontent.com/19144813/78006588-2b6c2980-7378-11ea-9496-e1dc7a1501c7.png)

이 값들은 가중치의 랜덤 값에 따라 달라지기에 실제로 실행시 약간의 차이가 있을 수 있습니다. 그러나. 퍼센트를 확인해보면 약 97%정도 정확도를 확인할 수 있습니다. 그래프를 통해서도 정확도가 0에서 97%까지 올라간 것을 확인할 수 있습니다. 이상으로 딥러닝 역전파 모델에 대한 구현을 마칩니다.
