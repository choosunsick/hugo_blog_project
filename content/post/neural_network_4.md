---
title: "Chapter3 미분과 확률적 경사하강법"
date: 2020-03-14T22:14:08+09:00
draft: False
tags: ["R로 딥러닝하기", "신경망", "순전파", "미분", "확률적 경사하강법"]
categories: ["R"]
---

이 시리즈는 R로 딥러닝을 구현하고 설명하는 것에 목표를 둔 글입니다. 신경망 챕터에 이어서 3번째 순서입니다. [이전 글](https://choosunsick.github.io/post/neural_network_3/)과 내용이 이어집니다. 이전 글들은 [이곳](https://choosunsick.github.io/post/contents_list/)에서 찾아 보실 수 있습니다.

## 수치미분과 기울기

미분을 사용하는 이유는 작은 변화에 따른 손실 함수의 변화 방향을 알기위해서 입니다. 신경망 모델에서 미분을 활용하는 방식을 경사하강법이라 말하는데 이에 대해 알아보기에 앞서 미분과 기울기에 대해 먼저 살펴보겠습니다.

### 미분

미분은 특정 순간의 변화량을 의미합니다. 예를 들어 마라톤 선수가 10분에서 2Km를 달렸다고 가정해봅시다. 이 때 속도는 분당 0.2Km 입니다. 즉 1분당 0.2Km 만큼 변화하는 것이라 말할 수 있습니다. 특정 순간의 변화량이기에 기존 10분에서 1분, 1초, 0.1초에 달린 거리 등으로 간격을 줄여서 한순간의 변화량을 얻는 것입니다. 수식으로 보면, x에 대한 f(x)의 변화량을 f(x)의 x에 대한 미분을 의미합니다. 즉 x의 작은 값의 변화가 f(x)의 값을 얼마나 변화시키는지를 확인하는 것입니다.

미분을 R로 구현하자면 다음과 같습니다.

```{r}
numerical_diff_t <- function(f,x){
  h <- 10e-50
  return((f(x+h)-f(x))/h)
}
```

이 함수는 미분할 함수와 그 함수에 필요한 값을 인자로 받습니다. 이런 미분 구현에는 한가지 문제점이 있습니다. 바로 두 점 사이의 함수의 차이 값에서 문제가 발생합니다. 위 코드 구현을 보면 함수 f(x+h)와 f(x) 사이의 차분 값을 계산하는데 이것은 미분의 근사치를 계산하는 것입니다. 따라서 실제로 필요한 값인 x에서의 기울기 값과 오차가 발생합니다.

이런 오차가 발생하는 이유는 h값이 아주 작은 수이기는 하지만 0으로 무한하게 좁혀가는 것이 불가능하기에 발생한 오차입니다. 오차 값을 최소화하기 위해서 f(x+h)와 f(x-h)의 차분 값을 계산하는 방법을 사용합니다. 이 방식을 중앙 차분이라하며 R에서 구현은 다음과 같습니다.

```{r}
numerical_diff <- function(f,x){
  h <- 1e-4
  return((f(x+h)-f(x-h))/(2*h))
}

function_1 <- function(x){
  return(0.01*x^2+0.1*x)
}
numerical_diff_t(function_1,5)
numerical_diff(function_1,5)
numerical_diff(function_1,10)
```

앞에서 작성한 수치 미분 함수가 잘 작동하는지 확인하기 위해서 미분 공식을 이용해서 작성한 `function_1`을 이용해서 비교해봅시다! 이 두 함수를 이용해서 5와 10에서의 미분 값을 계산해보면 두 함수 0.2와 0.3이 나오는 것을 알 수 있습니다.

이제 변수가 여러 개일 때 미분에 대해 알아봅시다. 변수가 한 개가 아닐 때는 미분은 편미분이라 합니다. 예를 들어 함수가 f(x) = x1^2+x2^2(`function_2 <- function(x){return(sum(colSums(x^2)))}`) 인 경우 변수가 x1와 x2 두 가지이기에 어느 변수에 대한 미분인지를 구분해주어야 합니다. x1가 3이고 x2이 4인 경우 두 변수에 대한 미분을 구현해보면 다음과 같습니다.

```{r}
function_tmp1<- function(x1){
  return(x1*x1 + 4^2)
}

function_tmp2<- function(x2){
  return(3^2 + x2*x2)
}

numerical_diff(function_tmp1, 3)
numerical_diff(function_tmp2, 4)
```

### 기울기

편미분은 변수가 하나인 미분과 마찬가지로 다른 변수를 고정시키고 특정한 순간의 변화량을 구합니다. 그렇다면 두 변수에 대해 동시에 편미분을 계산하고 싶으면 어떻게 해야할까요 ? 모든 변수에 대한 편미분을 진행해 벡터로 묶은 것을 기울기라 합니다. 기울기를 구하는 방식의 작동은 수치미분의 방식과 유사합니다.

```{r}
numerical_gradient <- function(f,x){
    h <- 1e-4
    vec <- vector()
    temp <- rep(0,length(x))

    for(i in 1:length(x)){
        temp[i]<-(temp[i] + h)
        fxh1 <- f(x+temp)
        temp[i]<-(temp[i] - (2*h))
        fxh2<- f(x+temp)
        vec <- c(vec, (fxh1 - fxh2) / (2*h))
        temp[i]  <- 0
    }
    return(matrix(vec, nrow = nrow(x) ,ncol = ncol(x)))
}

function_2 <- function(x){return(sum(colSums(x^2)))}

```

R로 기울기를 구하는 방식을 구현했을 때 방식은 x의 모든 원소에 대하여 수치미분을 진행하는 것입니다. 그렇기에 함수 안에 반복문이 들어가며 반복하는 횟수는 x의 개수가 됩니다. 모든 원소에 대해 수치미분을 진행한 것을 벡터로 쌓아 행렬로 만들어 반환해줍니다.  

```{r}
numerical_gradient(function_2,matrix(c(3,4),nrow = 1,ncol = 2))
numerical_gradient(function_2,matrix(c(0,2),nrow = 1,ncol = 2))
numerical_gradient(function_2,matrix(c(3,0),nrow = 1,ncol = 2))
```

그렇다면 기울기가 의미하는 바는 무엇일까요? 기울기 그림을 그려보겠습니다. 기울기의 그림은 방향을 가진 벡터로 그려지는데, 그림을 보시면, 0.0을 향해 화살표의 방향이 그려진 것을 확인할 수 있으며, 가장 낮은 지점(원점)에서 멀어질수록 화살표의 크기가 커짐을 알 수 있습니다. 기울기가 가장 낮은 지점을 가르키지만 실제로 그 지점이 반드시 가장 낮은 지점이라고 말할 수 없습니다. 정확히 말해 기울기가 가리키는 지점은 각 지점에서 함수의 출력값을 최소화하는 방향입니다.

```{r}
#install.packages("ggplot2")
library(ggplot2)
#install.packages("ggquiver")
library(ggquiver)

x0 <-  seq(-2, 2.25, 0.25)
x1 <-  seq(-2, 2.25, 0.25)
X <- matrix(rep(x0,NROW(x0)))

Y <- c()
for(i in 1:NROW(x1)){
  data <- matrix(rep(x1[i],NROW(x1)))
  Y<- rbind(Y,data)
}
ma<-matrix(0,nrow = 2,ncol = NROW(X))
for(i in 1:NROW(X)){
  ma[1,i] <- X[i]
  ma[2,i] <- Y[i]
}

grad <- numerical_gradient(function_2,ma)

X <- matrix(X,nrow = 1,ncol = NROW(X))
Y <- matrix(Y,nrow = 1,ncol = NROW(Y))

data <- data.frame(x=t(X),y=t(Y),u=-grad[1,],v=-grad[2,])
p<-ggplot(data = data,aes(x = data$x,y = data$y,u=data$u,v=data$v))+geom_quiver()+xlab("x0")+ylab("x1")
p
```

## 확률적 경사하강법(SGD,Stochastic Gradient Descent)

가중치와 편향의 크기가 광범위한 경우, 단순한 수치 미분 방식으로는 손실 함수 값이 최소화되는 곳이 어느 지점인지 알기 어렵다는 단점이 있습니다. 즉 수치 미분을 통한 기울기를 이용하더라도 해당 방향에 최소값이 있는지 반드시 보장할 수는 없습니다. 왜냐하면 극소값이나 안장점과 같이 최소값이 아닌 지점에서 기울기가 0이 되는 경우도 있기 때문입니다. 극소값은 한정된 범위에서 최소값을 의미합니다. 안장점이란 방향에 따라 극소값 혹은 극대값이 되는 지점을 의미합니다. 그렇기 때문에 기울기 정보를 이용해 해당 방향으로 나아가는 것은 최소값으로 나아가는 것이 아닌 값을 줄어드는 지점으로 나아가는 것입니다.

이런 문제를 위해 확률적 경사하강법을 사용합니다. 경사하강법은 현재 위치에서 기울기의 방향 정보를 통해 기울어진 방향으로 일정 거리만큼 이동합니다. 그리고, 이동한 지점에서도 기울기를 구하고 기울기의 방향에 따라 이동하여 나아가기를 반복합니다. 이렇게 반복하여 손실 함수 값을 점차 줄여나가는 것이 경사하강법입니다. 경사하강법은 최대값을 찾는 경우 경사상승법이라는 이름으로 불립니다. 이런 방법들은 신경망 모델의 학습에서 많이 사용되는 방법들입니다.

확률적 경사하강법의 경우 한개의 데이터가 아니라 여러개의 데이터를 미니배치의 방식으로 묶어서 기울기 정보를 얻고 그 정보를 통해 손실 함수 값을 최적화 시키는 방식입니다. 확률적으로 무작위로 데이터를 여러개 골라냈다는 점에서 확률적이라 말하며, 하나의 데이터를 이용하는 것보다 효율적으로 손실 함수의 최적 값을 찾을 수 있습니다.

경사하강법에서 기울기의 정보에 따라 해당 방향으로 나아갈 때 나아갈 정도를 학습률이라 말합니다. 학습률은 가중치와 편향의 변화량을 얼마나 반영 하는지를 정하는 값입니다. 학습률은 보통 0.01이나 0.001와 같은 값으로 정해야합니다. 왜냐하면, 학습률이 너무 크거나 너무 작을 경우 최소값을 제대로 찾아갈 수 없기 때문입니다. 이를 R로 구현하면 다음과 같습니다.

```{r}
gradient_descent<- function(f,init_x,lr=0.01,step_num=100){
  x <- init_x

  for(i in 1:step_num){
    grad <- numerical_gradient(f,x)
    x <- x-(lr*grad)
  }
  return(x)
}

function_2 <- function(x){return(sum(colSums(x^2)))}

init_x <- matrix(c(-3,4),nrow = 1,ncol = 2)
gradient_descent(function_2,init_x = init_x,lr=0.1,step_num = 100)
```

위의 코드에서, `f`는 최적화시키려는 함수이며(최소값을 찾기 위한) `lr`은 학습률을 의미하고, `step_num`은 경사하강법을 반복할 횟수를 의미합니다. 즉 갱신을 반복하는 횟수가 `step_num`입니다. 작성한 함수가 잘 작동하는지를 알아보기 위해 x1^2+x2^2의 최소값을 찾아봅시다. 이 함수의 최소값은 두 변수가 모두 0인 경우가 최소값임을 미리 알 수 있습니다.

초기값을 -3, 4에서 시작하여 최소값을 찾아나간다고 할 때 함수가 미분 값인 0, 0을 찾을 수 있는지 확인해보겠습니다. 학습률은 0.1로 100번 갱신을 반복한다고 할 때 함수가 찾아낸 값은 -6.111108e-10, 8.148144e-10 로 거의 원점에 가까운 지점임을 알 수 있습니다.

학습률이 너무 클 경우 가중치와 편향의 갱신이 너무 큰 값으로 커지게 됩니다. 반면, 학습률이 너무 작을 경우 기존의 가중치와 편향 값이 갱신되지 않고 종료됩니다. 만약 학습률이 0.1 보다 아주 크거나 아주 작으면 어떻게 될까요? 다음 코드의 결과를 확인해 봅시다.

```{r}
gradient_descent(function_2,init_x = init_x,lr=10,step_num = 100)
gradient_descent(function_2,init_x = init_x,lr=1e-10,step_num = 100)
```

당연히 두 경우 모두 최소값을 제대로 찾지 못하는 것을 확인할 수 있습니다. 따라서 학습률을 적절하게 설정하는 일은 아주 중요합니다.

위의 코드에서 100번 갱신하는 과정을 그림으로 그려보면 다음과 같습니다.

```{r}
x_history <- c()
gradient_descent<- function(f,init_x,lr=0.01,step_num=100){
  x <- init_x

  for(i in 1:step_num){
    grad <- numerical_gradient(f,x)
    x_history <- rbind(x_history,x)
    x <- x-(lr*grad)
  }
  x_history <- data.frame(x_history)
  colnames(x_history)<-c("x","y")
  return(x_history)
}

init_x <- matrix(c(-3,4),nrow = 1,ncol = 2)
lr <- 0.1
step_num <- 100
x_history <- gradient_descent(function_2, init_x, lr=lr, step_num=step_num)

circleFun <- function(center = c(0,0),diameter = 1, npoints = 100){
    r = diameter / 2
    tt <- seq(0,2*pi,length.out = npoints)
    xx <- center[1] + r * cos(tt)
    yy <- center[2] + r * sin(tt)
    return(data.frame(x = xx, y = yy))
}

temp<-cbind(circleFun(diameter = 1),circleFun(diameter = 3),circleFun(diameter = 6),circleFun(diameter = 9))
colnames(temp) <- c("dia1_x","dia1_y","dia3_x","dia3_y","dia6_x","dia6_y","dia9_x","dia9_y")

p_4_10<-ggplot(x_history,aes(x,y))+geom_point()+expand_limits(x=c(-3.5, 3.5),y=c(-4.5, 4.5))+scale_y_continuous(breaks=seq(-4.5,4.5, 1.5))+scale_x_continuous(breaks=seq(-4,4, 1))
p_4_10+geom_path(data = temp,aes(dia1_x,dia1_y),linetype="dotted")+geom_path(data = temp,aes(dia3_x,dia3_y),linetype="dotted")+geom_path(data = temp,aes(dia6_x,dia6_y),linetype="dotted")+geom_path(data = temp,aes(dia9_x,dia9_y),linetype="dotted")+xlab("x0")+ylab("x1")
```
