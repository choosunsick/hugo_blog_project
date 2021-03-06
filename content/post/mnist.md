---
title: "Mnist 손글씨 데이터 읽어오는 패키지 소개"
date: 2020-02-08T17:29:04+09:00
draft: False
categories: ["R"]
---

# dslabs 패키지 소개

스터디 그룹에서 프로젝트를 진행하면서 데이터셋과 관련된 좋은 패키지를 발견하여 소개해드릴까 합니다. R CLAN에 등록된 dslabs이라는 패키지는 딥러닝의 대표 예제인 손글씨 데이터 MNIST 데이터 셋이 있고 트럼프의 트윗내용이나 미국 총기 살해범에 대한 FBI 리포트등 약 26가지의 데이터 셋이 있으며 이런 데이터들을 쉽게 함수를 통해 읽어올 수 있습니다.

```R
install.packages("dslabs")
library(dslabs)
```

패키지 설치가 완료되고나면, 이제 26가지의 데이터 셋을 읽어올 수 있습니다. 대표적으로 MNIST 데이터 셋부터 읽어와 보겠습니다.

```R
mnist <- read_mnist()
str(mnist)
```

패키지 내부의 `read_mnist()` 함수를 사용하면, 손글씨 데이터인 MNIST 데이터를 읽어올 수 있습니다. 이렇게 읽어온 데이터에는 트레이닝 셋과 테스트 셋으로 크게 구분할 수 있고 데이터 행렬과 라벨 벡터로 분류되어 있는 것을 확인할 수 있습니다.

```R
x_train <- mnist$train$images
t_train <- mnist$train$labels
x_test <- mnist$test$images
t_test <- mnist$test$labels

dim(x_train)
length(t_train)
dim(x_test)
length(t_test)
```

훈련 이미지로는 60000장의 이미지가 60000,784 행렬 데이터로 들어있는 것을 확인할 수 있습니다. 그리고, 데이터의 실제 숫자가 무엇인지를 알려주는 숫자 라벨 데이터가 벡터로 들어있습니다. 테스트 셋의 이미지 데이터는 10000,784 행렬 데이터로 들어있는 것을 확인할 수 있습니다. 마찬가지로 테스트 셋 이미지에 대한 라벨 데이터가 있는 것을 확인할 수 있습니다.

```R
image(1:28, 1:28, matrix(x_test[5,], nrow=28)[ , 28:1], col = gray(seq(0, 1, 0.05)), xlab = "", ylab="")
t_test[5]
```

다음과 같이 R의 기본함수인 `image()` 함수를 사용하면 이미지 데이터가 어떻게 숫자로 표시되는지 플롯을 그려볼 수 있습니다. 그려진 이미지가 실제로 같은지를 라벨 데이터를 통해 확인할 수 있습니다. 참고로 MNIST 데이터 셋을 제외한 데이터의 경우 `data()` 함수에 데이터 셋의 이름을 입력하면 데이터를 읽어올 수 있습니다.
