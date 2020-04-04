---
title: "Chapter2 신경망 연습"
date: 2020-03-16T19:35:04+09:00
draft: FALSE
tags: ["R로 딥러닝하기", "신경망", "소프트맥스", "3층 신경망 구현"]
categories: ["R"]
---

## 손글씨 데이터 인식하고 분류하기

모델을 만들었지만 이 모델을 사용하기에는 아직 제한이 있습니다.  왜냐하면, 이 모델은 훈련을 거치지 않았기 때문에 성능이 좋지 못합니다. 그리고, 아직 입력에 필요한 손글씨 데이터도 없습니다. 따라서 모델의 입력 데이터인 손글씨 데이터가 필요합니다. 또한, 모델의 성능을 위해 저자의 초기값 데이터를 읽어와 모델에 적용시켜줍니다. 해당 초기값을 사용하여 모델을 구성하면 약 93% 성능을 가진 모델을 사용할 수 있습니다.

먼저 손글씨 데이터를 읽어와 보겠습니다. 손글씨 데이터는 0부터 9까지의 숫자를 손으로 쓴 데이터를 말합니다. 이 데이터는 해당 신경망 분류모델에서 자주 사용되는 예제입니다. R의 라이브러리 중 dslabs라는 패키지를 사용하면 데이터를 쉽게 불러올 수 있습니다.

```{r}
#install.packages("dslabs")
library(dslabs)

mnist <- read_mnist()
str(mnist)
```

데이터를 읽어오면 리스트 내에 훈련 데이터와 테스트 데이터를 확인할 수 있습니다. 훈련 데이터는 60000개의 이미지가 테스트 데이터에는 10000개의 이미지가 행렬로 담겨 있습니다. 이미지가 어떤 숫자인지를 알려주는 정답은 라벨이란 이름으로 있습니다.

```{r}
x_train <- mnist$train$images
t_train <- mnist$train$labels
x_test <- mnist$test$images
t_test <- mnist$test$labels

dim(x_train)
dim(x_test)
```

훈련데이터와 테스트 데이터를 리스트에서 분리해줍니다. 훈련데이터와 테스트 데이터의 크기를 확인해볼 수 있는데, 이미지의 장수가 행이 되고 이미지의 크기인 28x28=784가 열이 됩니다. 이제 이미지 데이터의 훈련데이터와 테스트 데이터에 대한 정규화를 해줍니다. 정답 라벨의 경우 원핫 인코딩의 형태로 변경해 줍니다. 원핫 인코딩이란 분류할 대상이 맞으면 1 아니면 0으로 체크하는 방식입니다. 손글씨의 경우 0부터 9까지의 10개의 숫자들 중 정답에 해당하는 숫자가 1 아닌 숫자를 0으로 표시하게 됩니다.

```{r}
x_train_normalize <- x_train/255
x_test_normalize <- x_test/255

making_one_hot_label <-function(t_label,nrow,ncol){
    data <- matrix(FALSE,nrow = nrow,ncol = ncol)
    t_index <- t_label+1
    for(i in 1:NROW(data)){
        data[i, t_index[i]] <- TRUE
    }
    return(data)
}

t_train_onehotlabel <- making_one_hot_label(t_train,60000,10)
t_test_onehotlabel <- making_one_hot_label(t_test,10000,10)

```

원핫 인코딩으로 바꾸는 방법은 함수를 통해 가능합니다. 라벨의 정답은 숫자가 0부터 있기에 0의 인덱스는 1이 됩니다. 마찬가지로 정답이 1이면 인덱스는 1이 더해진 2가 됩니다. 따라서 정답 라벨에 +1을 해주고, 그것을 원핫인코딩 행렬에서 위치값으로 사용합니다.

이제 모델을 준비할 차례입니다. 참고로 손글씨 데이터의 각층의 노드 개수는 입력층은 이미지의 크기에 따라 784개이고, 첫 번째 은닉층의 경우 50개, 두 번째 은닉층의 경우 100개, 출력층은 10개로 설정됩니다. 두 은닉층의 노드 개수 설정은 임의로 정한 값입니다. 모델은 손글씨 인식과 분류에 사용할 모델의 초기값 설정과 항등함수 대신 소프트맥스 함수를 사용하는 것  외에 기존의 3층 신경망과 똑같습니다.  

```{r}
source("./DeepLearningFromForR/functions.R")
source("./DeepLearningFromForR/chapter_2_3.R")
```

이제 초기값을 불러와 보겠습니다. 초기값을 불러오는 방식은 hdf5 형식을 이용하거나 json 파일 형식을 이용하는 방식 등 여러가지가 있습니다. 이 글에서는 최근 자주 사용되는 hdf5 형식으로 저장된 데이터를 R로 읽어와 가중치와 편향 행렬을 객체로 저장해보겠습니다. 참고로 hdf5 형식은 데이터를 계층적으로 저장하는 형식입니다. 이 형식에 대해 더 자세히 알고 싶은 분들은 링크(https://www.hdfgroup.org/solutions/hdf5/)를 참조해주시기 바랍니다.

```{r}
#install.packages("BiocManager")
#BiocManager::install("rhdf5")
library(rhdf5)
init <- H5Fopen("./deep_learn_R/Data/sample_weights.h5")
model <- list(matrix(init$"W1",784,50), matrix(init$"b1",1,50), matrix(init$"W2",50,100), matrix(init$"b2",1,100), matrix(init$"W3",100,10), matrix(init$"b3",1,10))
names(model) <- c("W1", "b1", "W2", "b2", "W3", "b3")
str(model)
```

이제 남은 작업은 모델에 데이터를 입력하고 그 성능을 확인하는 일만 남았습니다. 물론 모델의 성능을 알기 위해서는 모델을 검사할 성능 검사기가 필요합니다. 따라서 모델의 분류 결과에 대한 정답과 비교해 정확도를 판단하는 함수가 필요합니다.

```{r}
model.evaluate.single <- function(model,x,t){
  y <- do.call(rbind,lapply(1:NROW(x),function(i)max.col(model.forward(model,x[i,]))))
  t <- max.col(t)
  accuracy <- (sum(ifelse(y==t,1,0))) / dim(x)[1]
  return(accuracy)
}

model.evaluate.single(model,x_test_normalize,t_test_onehotlabel)
```

모델을 검사하는 함수는 `model.evaluate.single` 이란 이름으로 정의하였는데, 함수의 이름에 single이 붙은 이유는 이미지 데이터가 1장씩 계산되어 쌓여나가기 때문입니다. 1장씩 총 10000 장의 이미지가 모델에 입력되어 1장씩 어떤 숫자인지 예측하고 그것을 쌓아 정답 라벨 값과 비교하여 모델의 정확도를 표시합니다. 그러나 이렇게 이미지를 1장씩 입력받아 쌓아나가는게 최선일까요?

## 배치처리하기

그렇지 않습니다. 이미지 데이터를 한번에 여러장으로 묶어서 입력하고, 데이터에 대한 모델의 예측을 한번에 처리하는 방법도 있습니다. 이 방법을 배치처리라 말합니다. 책에서는 이미지 데이터를 100개씩 묶어 배치처리를 진행했지만, R에서는 더 많은 양의 이미지 데이터를 한번에 받아 진행해도 무리가 없기에 전체 이미지 데이터를 입력받아 전체 이미지 데이터에 대한 숫자 예측을 진행합니다.

배치차이에 따라 모델의 성능평가는 차이가 없지만, 함수의 작동시간에서 2초대와 0.4초대로 아주 큰 차이가 납니다. 따라서 전체 데이터를 입력받아 한번에 계산을 진행하고 결과를 얻는 배치 방식이 더 유용함을 알 수 있습니다.

```{r}
model.evaluate <- function(model,x,t){
  y <- max.col(model.forward(model,x))
  t <- max.col(t)
  accuracy <- (sum(ifelse(y==t,1,0))) / dim(x)[1]
  return(accuracy)
}

model.evaluate(model,x_test_normalize,t_test_onehotlabel)

system.time(model.evaluate.single(model,x_test_normalize,t_test_onehotlabel))
system.time(model.evaluate(model,x_test_normalize,t_test_onehotlabel))
```

이번 글에서는 신경망 분류모델에 필요한 함수들에 대해 알아보았으며, 직접 3층 신경망을 구현하고, 저자가 준비한 학습된 가중치와 편향을 이용하여 실제로 손글씨 이미지 데이터를 인식하고 분류하여 약 93% 성능을 가진 모델을 만들어 보았습니다. 다음 글에서는 저자가 어떤 방식으로 모델의 성능을 93% 끌어올리는 가중치와 편향을 학습시켰는지 모델의 학습 방법에 대해 알아보겠습니다.
