---
title: "R에서 서로 다른 크기의 행렬 연산"
date: 2018-10-09T15:52:32+09:00
draft: FALSE
---

안녕하세요. Lopes팀의 추선식 입니다. 이번에 저희팀에서 밑바닥부터 시작하는 딥러닝(Deep Learning from Scratch)이란 책을 R로 구현해보면서 알게 된 R의 행렬 연산 함수를 소개해볼까 합니다. R에서는 보통 행렬의 연산은 수학에 기반해 있습니다. 이에 따라 기본적인 행렬 연산은 수학적 정의에 따라 가능하며, 서로 다른 크기의 행렬 연산은 불가능합니다. 만약에 이러한 연산을 시도할 경우 우리가 보게 될 것은 "배열의 크기가 맞지 않습니다."라는 에러 메세지입니다. 그러나 파이썬의 경우 numpy를 통해 서로 다른 크기의 행렬의 연산이 가능합니다.

예를 들면 두 행렬 `np.array([[1,2,3],[4,5,6]])와 np.array([7,8,9])` 간 덧셈이나 뺄셈 나눗셈이 가능합니다. 이 두 행렬 간 덧셈의 결과는 `np.array([[8,10,12],[11,13,15]])`가 됩니다. 즉 두 행렬의 열 숫자가 같은 것을 기반으로 연산이 된 것입니다. 뺄셈이나 나눗셈, 곱셈 역시 마찬가지로 두 행렬의 열을 기반으로 연산이 이루어집니다. 추가적으로 파이썬에서의 행렬 곱셈은 `np.dot()`함수를 가지고도 가능합니다. 이 경우 행렬의 곱셈은 열 기반이 아닌 수학에서의 행렬 곱셈을 하게 됩니다.

그렇다면 파이썬에서 한 연산과 그 결과가 R에서 필요할 때는 어떤 방식을 사용할 수 있을까요. 이때 R에서 사용할 수 있는 방법으로는 apply 계열의 함수나 for문을 사용하여 하나씩 계산을 하는 방법이 있습니다.

```
temp <- matrix(c(1,2,3,4,5,6),2,3,byrow = T)
temp2 <- matrix(c(7,8,9),1,3,byrow = T)

```
예를 들어 이 두 행렬을 가지고 위의 결과 행렬을 얻으려면 `do.call(rbind,lapply(1:2,function(x){temp[x,]+temp2}))` 와 같이 `lapply()`를 써서 얻을 수 있습니다. 물론 이외에도 다양한 방법이 있습니다. 하지만 이러한 방법들은 코드의 완전성과 작동 시간의 측면에서 문제가 발생할 수 있습니다. 따라서 가장 안전한 방법은 R에서 기본적으로 정의된 함수를 사용하는 것입니다.

이 경우 사용할 수 있는 것이 `sweep()` 이란 함수입니다. 이 함수는 data, 연산을 적용할 대상(행(1)이나 열(2)), 기존의 데이터와 연산할 다른 데이터, "연산자"를 인자로 사용합니다. 이제 앞의 예를 그대로 적용해보면 `sweep(temp,2,temp2,"+")` 와 같은 방식으로 사용할 수 있습니다. 이 코드의 결과로는 `matrix(c(8,10,12,11,13,15),2,3,byrow= T)`와같은 행렬을 얻을 수 있습니다. sweep 함수는 이렇게 간단하게 사용할 수 있지만, 주의할 점이 있습니다. 그것은 연산할 데이터의 크기와 연산을 적용할 행렬의 행 또는 열의 크기가 서로 맞아야 한다는 것입니다.

위의 경우에서는 기존 행렬(2*3)에서의 열의 개수와 연산할 데이터의 개수가 3개로 서로 맞아서 각 열에 덧셈해줄 수 있게 됩니다. 즉 `length(temp2)==dim(temp)[2]`가 TRUE 라는 것입니다. 새로운 데이터의 개수가 기존 행렬의 행 또는 열 (연산을 적용할 대상에 따라 달라짐)과 다르게 될 경우 계산이 되지만, 에러 메세지와함께 원하는 결과를 얻지 못하게 됩니다.

이 주의점만 지킨다면 sweep 함수를 사용할 경우 그것의 완전성을 보장할 수 있습니다. 반면 스스로 코딩을 하게 될 경우 계산 결과가 원하는 것과 다르게 나올 가능성을 배제할 수 없습니다. 결과의 완전성 보장 이외에도 R에서 기본적으로 정의된 함수를 사용할 경우 속도 또한 빠르다는 장점이 있습니다. 보통은 스스로 코딩을 한 함수보다 기본 함수의 속도가 더 빠르기 때문입니다. 실제로도 `sweep()`을 사용한 경우와 `lapply()`를 사용하여 계산하는 것 간의 작동 시간을 비교하면 기초 함수인 sweep의 경우가 더 빠른 것을 확인할 수 있습니다.

```
system.time(do.call(rbind,lapply(1:2,function(x){temp[x,]+temp2})))
system.time(sweep(temp,2,temp2,"+"))
```
lapply를 사용하면 2초가 걸리는 반면 sweep을 사용하면 0초가 나옵니다. 이러한 시간 차이는 행렬의 크기가 커질수록 더욱 두드러지게 됩니다. 위에서는 덧셈을 가지고 예시를 들었지만 크기가 다른 행렬 간의 뺄셈, 나눗셈, 곱셈 등에서는 시간 차이가 더 명확하게 드러납니다.