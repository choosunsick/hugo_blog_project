---
title: "R을 사용해 모든 주식 종목 데이터를 하나의 파일로 저장하기"
date: 2019-07-10T17:16:03+09:00
draft: False
categories: ["R"], ["Data Analysis"] 
---

안녕하세요. 스터디 그룹 LOPES의 추선식 입니다. 이전의 [프로젝트](https://github.com/LOPES-HUFS/Korea_Stocks_for_HDF5) 소개글에서 언급한 것과 같이 이번에는 HDF5 형식에 대한 간략한 소개와 함께 R을 사용해 HDF5 파일을 만드는 튜토리얼을 소개하겠습니다. 간단한 튜토리얼에 이어서 전체 주식 종목을 하나의 HDF5 파일로 만들면서 HDF5 파일이 가지는 가장 큰 장점인 압축하는 기능에 대해서 알아볼 것입니다. 그럼 본격적으로 들어가기에 앞서서 HDF5 파일 형식이 무엇인지부터 알아보겠습니다.

## HDF(Hierarchical Data Format) 파일 형식이란 ?

HDF 파일은 데이터를 저장하는 형식 중 하나입니다. 이 HDF 파일은 계층적 데이터 형식(Hierarchical Data Format)이란 이름으로 데이터를 그룹(group)과 데이터 셋(dataset)으로 만들어 계층적으로 저장합니다. 이 형식의 최근 버전인 HDF5는 현재 [HDF그룹](https://www.hdfgroup.org/solutions/hdf5/)에서 관리하고 있습니다. HDF5 형식의 데이터 관리 방식은 마치 폴더와 파일의 구조와 유사합니다. HDF5 형식이 주식 종목 데이터를 저장하는 과정을 폴더와 파일에 비유하자면 다음과 같습니다.

HDF5 형식의 그룹(group) 구조는 KOSPI와 KOSDAQ 을 분리하는 분류용 폴더입니다. 이후 주식 종목의 데이터가 그룹 안에 데이터 셋(dataset)으로 입력되는데 이는 종목들이 KOSPI와 KOSDAQ 에 따라 분류되어 파일로 저장되는 과정입니다. 즉 데이터 셋 구조는 폴더에 저장되는 파일들이라 생각할 수 있습니다. HDF5 파일은 KOSPI와 KOSDAQ 을 모두 포함하는 전체 주식이란 폴더로 생각할 수 있습니다. 이런 구조에 따라 데이터들이 저장되기 때문에 HDF5 형식은 대용량 자료를 단일한 파일로 다룰 수 있다는 장점이 있습니다. 현재 HDF5형식은 R과 파이썬(Python) 등에서도 호환되어 쉽게 읽고 데이터를 다룰 수 있습니다. 이 형식에 대해 좀 더 자세히 알고 싶다면 [링크](https://support.hdfgroup.org/HDF5/whatishdf5.html)를 참조해주시기 바랍니다.

## 튜토리얼

이제 본격적으로 R을 사용해 HDF5 형식의 파일을 만드는 방법에 대해 알아보겠습니다. R에서 HDF5 형식을 다루기 위해서는 'rhdf5' 라는 패키지가 필요합니다.

```
install.packages("BiocManager")
BiocManager::install("rhdf5")
library(rhdf5)
```

위와 같은 코드로 rhdf5 패키지를 설치할 수 있습니다. 패키지 설치가 완료되면 이제 R에서 HDF5 파일을 다룰 기본적인 준비가 된 것입니다.

## R에서 HDF5 파일 형식 만들기

이제 주식 종목 데이터들을 하나의 HDF5 파일로 만들 차례입니다. 전체 주식 종목을 하나로 만들기에 앞서서 먼저 삼성전자(005930)와 LG전자(066570)의 주식 종목 데이터를 가지고 HDF5 파일로 저장하는 과정을 살펴보겠습니다. 참고로 전체 주식 종목을 가지고 HDF5 파일로 만들 경우 약 90MB 정도의 크기가 나오게 됩니다.

```
stock_005930 <- read.csv(file =  "https://raw.githubusercontent.com/LOPES-HUFS/Korea_Stocks_for_HDF5/master/stocks/005930.csv", header = TRUE, stringsAsFactors=FALSE)

str(stock_005930)

stock_005930$Date <- as.integer(paste(substr(stock_005930$Date,1,4),substr(stock_005930$Date,6,7),substr(stock_005930$Date,9,10),sep = ""))
```

우선 삼성전자와 LG전자의 종목 데이터를 깃허브에서 `read.csv()` 함수로 읽어와 줍니다. `str()` 함수를 통해 데이터의 행과 열을 확인할 수 있는데, 이때 데이터의 행과 열이 너무 많거나 적지는 않은지 확인해야 합니다. 나중에 파일의 압축과 관련하거나 HDF5 파일을 쓰거나 읽을 때 데이터의 크기가 문제가 될 수 있기 때문에 데이터의 행과 열을 확인해야 합니다. 각 열의 데이터 타입 역시 확인할 수 있는데 날짜가 string 형식이므로 나중에 파일의 크기를 줄이기 위해 "-"를 지우고 int 타입으로 만들어 모든 열의 데이터 타입을 통일 시켜 줍니다.

```
h5createFile("sample.h5")
```

이제 주식 종목을 읽어왔으니 저장할 HDF5 파일을 만들 차례입니다. 새로운 HDF5 파일을 만드는 방식은 `h5createFile("파일이름")` 함수를 사용합니다. 이 함수가 정상적으로 작동했다면 결괏값으로 TRUE가 나오는데, 만약 FALSE가 결과로 나온다면 같은 이름의 HDF5 파일이 존재하는 경우로 에러가 발생한 경우입니다.

## HDF5 파일의 데이터 구조 - 그룹과 데이터 셋

```
# 그룹 구조 만들기
h5createGroup("sample.h5","KOSPI")

# 그룹 안의 데이터 셋으로 저장하는 방식
h5write(as.matrix(stock_005930), file="sample.h5",name="KOSPI/kor_005930")

# 데이터 셋으로 바로 저장하는 방식
h5write(c("Date","Open","High","Low","Close","Volume","Adj_Close"), file="sample.h5",name="colnames")

```

HDF5 파일에는 그룹과 데이터 셋이란 두 가지 구조가 있습니다. 그룹은 데이터 셋들을 저장하는 상위 구조의 역할을 합니다. 그룹은 `h5createGroup("만든 HDF5 파일", "그룹의 이름")` 코드를 통해 만들 수 있습니다. 그룹을 만들었으니 이제 그룹 안에 데이터 셋으로 종목 데이터를 집어넣을 차례입니다. `h5write(데이터, file="HDF5 파일이름",name="그룹이름/데이터셋이름")`으로 데이터를 그룹 안에 넣을 수 있습니다. 물론 그룹을 만들지 않고, 데이터를 여러 데이터 셋 집합으로 저장할 수도 있습니다. 세부 분류가 필요 없으면 데이터 셋의 집합으로 데이터를 저장하는 방식이 파일의 크기를 줄이는 데 도움이 됩니다. 이 방식으로 주식 종목 데이터의 열 이름을 저장해 볼 것입니다. 기존에 그룹 안에 데이터를 넣을 때와 차이점은 이름에 그룹 경로를 포함하느냐 차이만 나고 같은 방식으로 `h5write()` 함수로 데이터 셋을 만들 수 있습니다. 같은 방식으로 LG전자 데이터를 입력할 수 있습니다.

## 만들어진 HDF5 파일에서 데이터 읽어오기

이제 위에서 만든 h5 파일인 "sample.h5"를 열어서 입력한 데이터를 다루어 보겠습니다.

```
sample <- H5Fopen("sample.h5")

# 입력된 데이터 셋 확인
h5ls(sample)
```

`H5Fopen("HDF5파일이름")` 함수를 사용해서 만든 sample 파일을 읽어와 줍니다. 읽어온 파일을 변수로 저장해주고 `h5ls()` 함수로 저장된 데이터 셋들을 확인해줍니다.

```
samsung_sample <- data.frame(sample$KOSPI$kor_005930)
colnames(samsung_sample) <- sample$colnames
samsung_sample$Date <- paste(substr(samsung_sample$Date,1,4),"-",substr(samsung_sample$Date,5,6),"-",substr(samsung_sample$Date,7,8),sep = "")

# HDF5 파일을 닫기
h5closeAll()
```

읽어온 삼성전자의 데이터를 새로운 데이터 프레임으로 만들어줍니다. 그룹 안에 있는 데이터 셋은 $, $ 두 번 쓰거나, "그룹 이름/데이터 셋 이름" 같은 데이터 셋의 경로로 확인할 수 있습니다. 이어서 해당 데이터의 열 정보인 colnames로 열 이름을 다시 입력해 줍니다. 기존의 종목 데이터로 만들기 위해 숫자로 된 날짜 정보를 "-"를 사용해 복원해줍니다. 그러면 기존의 종목 데이터와 같은 모습을 되찾는 것을 확인할 수 있습니다. 이렇게 데이터를 원래대로 만들었으니 만들기위해 열었던 HDF5 파일을 닫아줍니다.

## 전체 주식 종목 하나의 파일로 만들기 - 압축하기

이제 튜토리얼이 끝났으니 전체 주식 종목을 하나의 파일로 저장해보겠습니다. 튜토리얼에서처럼 2가지의 주식 종목만으로는 HDF5 파일의 형식이 가진 장점인 대용량 자료를 하나의 파일에 저장한다는 장점을 잘 보여 주지 못합니다. 전체 주식 종목 데이터를 깃허브에서 하나하나 읽어올 경우 시간이 오래 걸리기 때문에 폴더에서 파일을 읽어오도록 변경합니다. 따라서 이 프로젝트를 클론하거나 zip으로 받아서 이용해야 합니다. 또한, 다운로드한 주식 파일들을 R의 디렉터리 환경을 다운로드한 파일로 변경해주어야 합니다. `setwd("자료를 다운로드한 위치/Korea_Stocks_for_HDF5")` 이 코드를 통해 R의 기본 디렉터리를 변경해주면 이후 다운로드한 주식 종목 데이터를 읽어오는 과정에서 문제가 생기지 않습니다.

```
h5createFile("all_stock.h5")

create_h5_file <- function(stocknumber){
    file_path <- paste("./stocks",stocknumber,sep="")
    stock_temp <- read.csv(file = file_path, header = TRUE, stringsAsFactors=FALSE)
    stock_temp$Date <- as.integer(paste(substr(stock_temp$Date,1,4),substr(stock_temp$Date,6,7),substr(stock_temp$Date,9,10),sep = ""))
    data_name <- paste("kor_",stocknumber,sep = "")
    h5createDataset("all_stock.h5", data_name, c(NROW(stock_temp),7),storage.mode = "integer",level=9)
    h5write(as.matrix(stock_temp), file="all_stock.h5",name=data_name)
  }

stock_list <- dir("./stocks")
lapply(stock_list,function(x){create_h5_file(x)})

```

전체 종목 데이터를 하나의 파일로 만들 때는 튜토리얼에서 진행한 HDF5 파일을 만드는 과정과 차이가 있습니다. 그 차이점은 데이터 입력에서 스스로 데이터 셋의 구조를 정한다는 점입니다. 데이터 셋의 구조를 정하는 이유는 가장 최적의 파일 크기로 만들기 위해서 입니다. 전체 주식 종목 데이터 파일들의 크기를 다 더하면 약 340MB가 됩니다. 이 파일들을 하나의 파일로 병합하더라도 최소 200MB가 넘어갈 것입니다. 그렇다면 파일의 크기를 최적화하려면 어떻게 해야할 까요?   

파일 크기 최적화에 대한 해답은 압축 기능을 사용하는 것입니다. 데이터를 저장할 때 `h5createDataset()` 함수를 사용해 데이터 셋의 구조와 압축과 관련된 요소들을 직접 설정할 수 있습니다. 일반적으로 파일의 크기에 관련되는 요소는 입력 데이터의 크기와 데이터 저장 타입과 압축률 등이 있습니다. 여기서 데이터의 크기는 정해져 있는 경우가 많기에 저장 타입과 압축률에 대하여 스스로 설정할 수 있습니다. 예를 들면, 문자열보다는 integer 형식의 데이터 타입이 더 크기가 작기 때문에 입력 데이터의 형식을 integer로 통일합니다. 데이터 형식을 통일 시켜주면, 이제 저장 타입을 integer로 설정할 수 있습니다. 그리고, 최적의 파일 크기를 만들기 위해 압축률을 최고 수준인 9로 설정합니다.

이렇게 압축을 통해 파일의 크기를 최적화하는 것은 HDF5 파일이 가진 가장 큰 장점 중의 하나입니다. 만약 압축하지 않는다면 파일의 크기는 약 200MB 정도의 크기를 가지게 됩니다. 그러나 모든 주식 종목의 데이터를 최적의 데이터 셋 구조로 설정하여 파일로 만들 경우 약 90MB 정도 크기가 나오게 됩니다. 이렇게 만들어진 결과물은 "all_stock.h5" 파일에 담겨 있습니다.

지금까지 R을 사용해 HDF5 파일을 만들고 사용하는 방법과 HDF5 파일의 꽃인 압축 기능에 대해 알아보았습니다. 다음에는 HDF5 파일을 만드는 과정에서 알아두면 좋은 팁들을 소개해보겠습니다. 긴 글 읽어주셔서 감사합니다.  
