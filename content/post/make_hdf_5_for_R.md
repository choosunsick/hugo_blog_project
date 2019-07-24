---
title: "R을 사용해 HDF5 파일로 만들고 데이터 압축하기"
date: 2019-07-17T17:21:17+09:00
draft: False
categories: ["R", "Data Analysis"]
---

# R로 HDF5 파일 만들기

안녕하세요. 스터디 그룹 LOPES의 추선식 입니다. 오늘은 저번에 소개한 [프로젝트](https://github.com/LOPES-HUFS/Korea_Stocks_for_HDF5)에 이어 R을 사용해 데이터를 HDF5 파일로 저장하는 방법을 소개해볼까 합니다. HDF 형식이 낯선 분들을 위해 HDF 형식이 무엇인지 잠깐 살펴보겠습니다. HDF 형식은 대용량 데이터를 저장 및 구성하기 위해 고안된 데이터 형식으로 대용량 데이터를 파일 시스템과 유사한 방식으로 쉽게 저장하고 관리할 수 있으며, C와 C++를 비롯한 R, Python 등의 여러 프로그래밍 언어에 대해 API를 지원하고 있는 점 등의 장점이 있습니다. 나아가 NASA에서 표준 데이터 정보 시스템으로 채택되어 사용되는 만큼 데이터 형식의 안전성 또한 보장 받을 수 있습니다. 참고로 HDF5는 HDF 형식의 최신 버전을 의미합니다.

## 필요한 패키지 설치하기

이제 R을 사용해 데이터를 HDF5 파일로 저장하는 방법에 대해 알아봅시다. R에서 HDF5 파일을 만들고 다루기 위해서는 외부 패키지가 필요합니다. R에는 HDF5를 다루는 다양한 패키지가 있지만, 제가 사용할 것은 "rhdf5" 란 패키지입니다. 다음의 코드로 패키지의 설치가 가능합니다.  

```
install.packages("BiocManager")
BiocManager::install("rhdf5")
library(rhdf5)
```

패키지의 설치가 끝나면 R에서 새로운 HDF5 파일을 만들고 이후 데이터를 저장할 수 있습니다. HDF5가 데이터를 저장하는 방식에는 두 가지 구조가 있습니다. 첫 번째로 같은 유형의 다차원 배열을 저장하는 Dataset 구조가 있고, 두 번째로는 데이터 셋(Dataset)을 저장할 수 있는 상위 구조인 Group 구조입니다. 데이터 셋 구조는 HDF5 형식에서 가장 기본적인 데이터 저장 방식으로 입력하는 데이터의 형식과 크기에 밀접한 연관을 가집니다. 따라서 데이터 셋 구조를 설정하는 과정 역시 중요해집니다. 이제 주식 자료들을 저장하는 것으로 직접 데이터 셋 설정을 알아보겠습니다.

## 문자열 데이터 HDF5로 저장하기

먼저 삼성전자의 종목 데이터를 열어서 데이터의 열(column) 정보를 저장해봅시다. 데이터의 열(column) 정보는 문자열(string)로 나타납니다. 따라서 HDF5에 문자열 데이터를 저장하는 방법에 대해 알아보겠습니다. 입력할 데이터의 형식이 문자열 타입이면 데이터 셋을 만들 때 입력할 문자열 데이터의 최대 길이 +1한 값을 입력해야 합니다. 왜냐하면, HDF5 파일에서는 [null-terminated string](https://en.wikipedia.org/wiki/Null-terminated_string)을 자동으로 추가하기 때문입니다. 그렇기 때문에 +1을 더해주지 않으면 입력한 데이터가 본래 데이터와 다르게 저장되게 됩니다. 아래의 코드로 문자열 데이터를 HDF5 파일에 저장하는 방법을 확인하실 수 있습니다.

```
stock_005930 <- read.csv(file =  "https://raw.githubusercontent.com/LOPES-HUFS/Korea_Stocks_for_HDF5/master/stocks/005930.csv", header = TRUE, stringsAsFactors=FALSE)

stock_005930$Date <- as.integer(gsub("-","",stock_005930$Date))

h5createFile("colname.h5")

h5createDataset(file = "colname.h5", dataset = "colnames",dims = 7 ,storage.mode="character",size= max(nchar(colnames(stock_005930)))+1)

h5write(colnames(stock_005930),"colname.h5","colnames")
```

## 데이터 압축해서 저장하기 

이제 삼성전자의 종목 데이터를 HDF5 파일에 저장하는 것으로 HDF5의 꽃인 압축에 대해서 알아보겠습니다. 압축은 입력되는 데이터의 변화 없이 파일 크기를 줄이는 작업입니다. 이 압축 기능은 데이터 셋을 만들 때 level 인자를 통해 설정할 수 있습니다. 이제 압축 기능이 얼마나 뛰어난지 알아보기 위해 압축하지 않은 경우(0), 중간 수준(6)으로 압축한 경우, 최고 수준(9)으로 압축한 경우 간 파일의 크기를 비교해보겠습니다. 아래의 코드는 압축하지 않은 경우일 때 코드입니다. 위의 코드에서 test.h5를 만든 것과 같은 방식으로 파일 이름을 바꾸어서 열 정보를 저장하고 아래의 과정에서 level 인자를 변경해 반복해줍니다.

```
h5createFile("level_0.h5")

h5createDataset(file = "level_0.h5", dataset = "kor_005930_level0",dims = dim(stock_005930) ,storage.mode="integer",level=0)

h5write(as.matrix(stock_005930),"level_0.h5","kor_005930_level0")

h5createFile("level_6.h5")

h5createDataset(file = "level_6.h5", dataset = "kor_005930_level6",dims = dim(stock_005930) ,storage.mode="integer",level=6)

h5write(as.matrix(stock_005930),"level_6.h5","kor_005930_level6")

h5createFile("level_9.h5")

h5createDataset(file = "level_9.h5", dataset = "kor_005930_level9",dims = dim(stock_005930) ,storage.mode="integer",level=9)

h5write(as.matrix(stock_005930),"level_9.h5","kor_005930_level9")

```

## 압축한 파일들 비교하기

반복을 통해 압축 수준이 다른 3가지 파일이 만들어졌으면 이제 파일 크기를 비교해 봅시다. 최고 수준과 중간 수준으로 압축했을 때는 크기가 별로 차이 나지 않는 것을 확인할 수 있습니다. 그러나, 압축하지 않은 경우에 파일 크기가 거의 2배가량 차이 나는 것도 확인할 수 있습니다. 실제로 모든 주식 종목 데이터를 HDF5 파일로 만드는 경우 최고 수준으로 압축을 진행하면 87MB이며 중간 수준은 88MB지만, 압축을 하지 않은 경우는 190MB로 압축하는 경우와 2배 이상의 크기 차이를 보입니다. 또한, 전체 주식 종목의 CSV 파일들의 크기는 341.6MB 이므로 압축을 하지 않은 경우에도 HDF5로 저장하게 되면 크기가 줄은 것을 확인할 수 있습니다. 물론 반드시 높은 압축 수준이 좋은 것은 아닙니다. 압축 수준이 높을수록 파일의 크기는 작아지지만, 데이터를 입력하는데 더 오랜 시간이 걸리기 때문입니다.

```
file.size("level_0.h5")

file.size("level_6.h5")

file.size("level_9.h5")

```   

이외에 R로 HDF5 파일을 만들고 다루는데 더 자세한 내용을 알고 싶다면, rhdf5 패키지의 [설명서](https://www.bioconductor.org/packages/devel/bioc/vignettes/rhdf5/inst/doc/rhdf5.html)를 참조하실 수 있습니다. 이상으로 글을 마칩니다.
