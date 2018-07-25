---
title: "2018년 KBO 가을야구 진출팀은 어떤팀일 것인가 ?"
date: 2018-07-24T16:53:31+09:00
draft: FALSE
mathjax: true
mathjaxEnableAutoNumber: true
mathjaxEnableSingleDollar: true
---

안녕하세요. 스터디 그룹 LOPES의 추선식입니다. 메이저리그 야구 통계학이라는 책을 공부하는데, 이 책에서 나온 기대 승률(피타고리안 승률)이란 통계적 지표를 KBO 자료에 적용해보았습니다. 이 책에서는 기대 승률을 다음과 같이 설명합니다.

> <머니볼>을 보면 단장역을 맡고 있는 브래드 피트가 분석보좌 역과 함께 복잡하게 숫자가 널려있는 화이트 보드 앞에서 대화를 나누는 장면이 나온다 그 장면에서 사무실 벽에 걸려있는 화이트보드에는 2001년도 아메리칸리그 서부지구에서 오클랜드가 포스트시즌에 진출할 때 보유하고 있던 실제 수치와 그 공식이 첫 번째 행에 나온다.

$$ 오클랜드(2001): \frac{884^2}{(884^2 + 645^2)} = 0.6525  (2001년 실제 승률은 0.6296) $$

> 884는 2001년도 오클랜드의 실제 팀득점을, 645는 실제 팀실점을 가리키며, 다음 공식에 대입해서 이론 승률을 구했던 것이다. 오클랜드의 2001년 실제승률은 62.96%였으며 공식에 득점과 실점을 투입해서 나오는 0.6525 예측치와 제법 가까운 값이 나왔다. 결국 기대 승률을 예측하는 데 득점과 실점이 중요한 역할을 하기에 역으로 2002년 포스트시즌에 진출할 수 있는 득점과 실점을 계산할 수 있다는 주장이다.

$$ \dfrac{득점^2} {(득점^2 + 실점^2)} = 기대 승률 $$

> 공식의 왼쪽 항에 있는 분수식을 논리적으로 이해할 필요는 없다. 경험적으로 득점과 실점을 이렇게 구성했을 때 실제 승률과 가장 비슷한 기대 승률이 나왔다는 경험적 발견이기 때문이다.

> \[메이저리그 야구 통계학, 김재민, 에이콘, pp.119 ~ 120\]

2018 KBO 프로야구 시즌은 올스타전이 끝나고 후반기에 돌입하여 가을 야구까지 약 2~3개월이 남은 상태입니다. 많은 사람이 어느 팀이 포스트시즌에 진출해 가을야구를 할 것인지 궁금해할 것입니다. 기대 승률이라는 지표와 실제 승률과 전년도 포스트시즌 진출팀의 평균 승률과 같은 통계적 지표들 간의 비교를 통해 어느 팀이 포스트시즌에 진출할 가능성이 높은지를 살펴보겠습니다.

## 기대 승률 및 실제 승률를 비교해 봅시다.

### 2017년 KBO 자료에서 포스트시즌 참가팀 추리기

우선 2017년 KBO 자료를 R로 읽어오겠습니다. 혹시 더 오래된 KBO 자료가 필요하시면 <https://github.com/choosunsick/KBO_data>로 가시면 됩니다.

```
KBO.2017.URL<-"https://lopes.hufs.ac.kr/KBO/KBO_2017_season.csv"
KBO.2017<-read.table(file = KBO.2017.URL, header= TRUE, sep=",",stringsAsFactors = FALSE)
```

읽어온 자료에서 정규시즌과 시범경기 올스타전을 제외한 경기만 뽑아줍니다. 이어서 그 자료에서 경기를 한 팀들을 모으고 고유한 팀만 남겨줍니다. 그러면 2017년 포스트시즌에 경기를 했던 팀만 남게됩니다. 아래의 결과를 보시면 "NC", "롯데", "두산","기아", "SK" 5개 팀을 확인할 수 있습니다.

```
KBO.2017.postseason<-KBO.2017[KBO.2017$비고!="시범경기"&KBO.2017$비고!="정규시즌"&KBO.2017$비고!="올스타전",]
postseasonteam.2017<-unique(c(KBO.2017.postseason$홈팀,KBO.2017.postseason$원정팀))
postseasonteam.2017
```

    ## [1] "NC"   "롯데" "두산" "기아" "SK"

### 2017년 포스트시즌 진출팀의 승률 평균 값 구하기

이제 위에서 확인한 5개 팀의 승률을 함수를 통해 구할 수 있습니다. 5개 팀의 승률 평균 값을 구하는 이유는 이후 기대 승률 값과 비교를 하기위해서 입니다. 승률을 구하는 과정에서 필요한 함수는 두 가지로 데이터를 전처리하는 함수와 승률을 계산하는 함수입니다. 데이터를 전처리 함수는 데이터, 연도, 팀이름의 3가지 인자를 받는 함수로 연도와 팀이름을 이용해 데이터에서 경기결과를 새롭게 만들어 주는 역할을 합니다.

이 함수의 작동원리는 다음과 같습니다. 먼저 데이터에서 인자로 입력된 팀이 홈팀일 때의 정규시즌 데이터를 뽑아냅니다. 이후 경기연도를 열로 만들어 입력된 년도의 데이터만 뽑아냅니다. 경기결과 열은 홈팀점수와 원정팀점수 간의 비교를 통해 홈팀점수가 더 높으면 승, 낮으면 패, 같으면 무로 나누어 구성해줍니다. 이런 과정을 인자로 받은 팀이 원정팀인 경우에도 똑같이 진행하여 경기결과 열을 만듭니다. 이렇게 만들어진 두 데이터를 다시 합쳐서 반환합니다.

```
data.cleaning <- function(x,y,z){#x=데이터,y=년도,z=팀이름
  teamdata.home<-x[x$홈팀==z & x$비고 =="정규시즌",] #
  teamdata.home$경기연도<-substr(teamdata.home$Date,1,4)
  teamdata.home<-teamdata.home[teamdata.home$경기연도==y,]
  teamdata.home$경기결과 <- ifelse(teamdata.home$홈팀점수>teamdata.home$원정팀점수,"승",
                               ifelse(teamdata.home$홈팀점수<teamdata.home$원정팀점수,"패","무"))
  teamdata.away<-x[x$원정팀==z & x$비고 =="정규시즌",]
  teamdata.away$경기연도<-substr(teamdata.away$Date,1,4)
  teamdata.away<-teamdata.away[teamdata.away$경기연도==y,]
  teamdata.away$경기결과 <- ifelse(teamdata.away$홈팀점수>teamdata.away$원정팀점수,"패",
                               ifelse(teamdata.away$홈팀점수<teamdata.away$원정팀점수,"승","무"))
  teamdata<-rbind(teamdata.home,teamdata.away)
  teamdata<-teamdata[order(teamdata$Date,decreasing = FALSE),]
  return(teamdata)
}
```

`head(data.cleaning(KBO.2017,"2017","두산"))` 로 전처리 함수의 결과를 확인할 수 있습니다.

### 전처리 함수를 적용한 데이터를 가지고 승률 함수에 적용하기

전처리 함수로 처리한 데이터를 승률을 구하는 함수에 넣어서 해당 팀의 정규시즌 승, 패, 무, 승률을 계산합니다. 승률 함수는 전처리 함수에서 만든 경기결과 열을 통해 해당 팀의 승리, 패배, 무승부(있는 경우에만)를 가지고 승률 공식에 대입하여 계산하는 역할을 합니다.

$$ 승률 공식: \dfrac{총 승리한 경기수}{전체 경기에서 무승부를 제외한 경기 수} $$

이 함수의 인자는 하나로 데이터만 필요로 합니다. 참고로 이 함수에서 데이터가 2010년의 데이터인지 아닌지를 검사하는데, 이는 2010년에는 승률을 구하는 공식이 다르기 때문입니다.

이제 데이터가 2010년이 아닌 경우면 위의 공식으로 계산을 하게됩니다. 그후에 해당 팀의 기록에 무승부가 있는지 없는지를 검사합니다. 이것은 승률 계산시 무승부를 제외하기에 필요한 검사입니다. 이 검사 이후에 무승부가 있는 경우와 없는 경우를 나누어 해당 팀의 승률을 계산해줍니다. 이렇게 계산된 승률은 해당 팀의 승리와 패배, 무승부 기록과 함께 데이터 프레임(data.frame)으로 반환됩니다.

```
Yearly.Win.Per <- function(x){#x=data
  if(all(x$경기연도=="2010")==TRUE){
    if(NROW(table(x$경기결과))>2){
      win.percentage <- table(x$경기결과)[2]/sum(table(x$경기결과))
      result<-data.frame("승"=table(x$경기결과)[2],"패"=table(x$경기결과)[3],"무"=table(x$경기결과)[1],"승률"=win.percentage,row.names = NULL)
    }
    else{
      win.percentage <- table(x$경기결과)[1]/sum(table(x$경기결과))
      result<-data.frame("승"=table(x$경기결과)[1],"패"=table(x$경기결과)[2],"무"="무승부 없음","승률"=win.percentage,row.names = NULL,stringsAsFactors = FALSE)
    }
  }
  else{
    if(NROW(table(x$경기결과))>2){
      win.percentage <- table(x$경기결과)[2]/(sum(table(x$경기결과))-table(x$경기결과)[1])
      result<-data.frame("승"=table(x$경기결과)[2],"패"=table(x$경기결과)[3],"무"=table(x$경기결과)[1],"승률"=win.percentage,row.names = NULL)
    }
    else{
      win.percentage <- table(x$경기결과)[1]/sum(table(x$경기결과))
      result<-data.frame("승"=table(x$경기결과)[1],"패"=table(x$경기결과)[2],"무"="무승부 없음","승률"=win.percentage,row.names = NULL,stringsAsFactors = FALSE)
    }
  }
  return(result)
}
```

위의 함수를 2017년 두산팀 데이터에 적용해보겠습니다.

```
Doosan.win.percentage<-Yearly.Win.Per(data.cleaning(KBO.2017,"2017","두산"))
Doosan.win.percentage
```

    ##   승 패 무      승률
    ## 1 84 57  3 0.5957447

이제 같은 방식으로 앞서 구한 전년도 포스트시즌 진출한 ("NC","롯데","두산","기아","SK") 5개 팀 모두의 승률을 구해줍니다. 5개 팀의 승률을 구하면 그것의 평균을 구해줍니다. 평균 승률을 구하는 이유는 기대 승률과 비교와 시즌 종료시 포스트시즌 진출에 필요한 득점의 수를 추정하는데 필요하기 때문입니다.

```
fiveteam.win.percentage<-do.call(rbind,lapply(1:5,FUN = function(i){Yearly.Win.Per(data.cleaning(KBO.2017,"2017",postseasonteam.2017[i]))}))
win.percentage.mean<-mean(fiveteam.win.percentage$승률)
win.percentage.mean
```

    ## [1] 0.5704552

## 2018년 전반기 상위팀의 승률과 기대 승률 구하기


이제 작년 포스트시즌 진출팀의 평균 승률을 구했으니 비교할 대상인 현재 시즌에 상위팀 ("두산","한화","SK","LG","넥센")의 전반기 승률과 기대 승률을 구해볼 차례입니다. 먼저 2018년 전반기 데이터를 R로 가져옵니다. 2018년 두산팀의 데이터를 예시로 먼저 승률을 구해보았습니다.

```
KBO.2018.half.URL <- "https://lopes.hufs.ac.kr/KBO/KBO_2018_season_half.csv"
KBO.2018.half<-read.table(file = KBO.2018.half.URL, header= TRUE, sep=",",stringsAsFactors = FALSE)
Doosan.2018<-data.cleaning(KBO.2018.half,"2018","두산")
Yearly.Win.Per(Doosan.2018)
```

    ##   승 패          무      승률
    ## 1 58 29 무승부 없음 0.6666667

("두산","한화","SK","LG","넥센") 5개 팀의 승률을 구하면 다음과 같습니다.

```
Topfive.team.2018<-c("두산","한화","SK","LG","넥센")
Topfive.team.2018.win.percentage<-do.call(rbind,lapply(1:5,FUN = function(i){Yearly.Win.Per(data.cleaning(KBO.2018.half,"2018",Topfive.team.2018[i]))}))
row.names(Topfive.team.2018.win.percentage) <- Topfive.team.2018
Topfive.team.2018.win.percentage
```

    ##      승 패          무      승률
    ## 두산 58 29 무승부 없음 0.6666667
    ## 한화 52 37 무승부 없음 0.5842697
    ## SK   48 37           1 0.5647059
    ## LG   48 41           1 0.5393258
    ## 넥센 46 46 무승부 없음 0.5000000

이제 다른 비교 지표인 기대 승률을 구해보겠습니다. 기대 승률은 함수를 통해 구할 수 있습니다. 이 함수는 데이터와 팀이름을 인자로 받아, 팀이름을 홈팀과 원정팀일 때로 구분하고 각각 득점과 실점의 합을 구하고 그것을 기대 승률 공식에 대입하여 기대 승률, 총득점, 총실점을 반환합니다.

```
pythagorean.expectation<-function(x,y){#x=data,y=teamname
  team.data.home<-x[x$홈팀==y,]
  team.data.away<-x[x$원정팀==y,]
  Runs.score.home <- team.data.home$홈팀점수
  Runs.score.away <- team.data.away$원정팀점수
  Runs.home <- team.data.home$원정팀점수
  Runs.away <- team.data.away$홈팀점수
  Total.Runs.score<-sum(Runs.score.home,Runs.score.away)
  Total.Runs<-sum(Runs.home,Runs.away)
  expectancy.percentage <- Total.Runs.score^2/(Total.Runs^2+Total.Runs.score^2)
  expectancy.table<-data.frame(총득점=Total.Runs.score,총실점=Total.Runs,기대승률=expectancy.percentage)
  return(expectancy.table)
}
```

## 현재 상위 5개 팀의 2017년도 총득점, 총실점 구해보기

이를 구하는 방법은 작년의 데이터를 기대 승률 함수에 적용합니다. 책에서와 같이 여기서 구한 총실점을 이번년도 총실점에도 그러할 것이라 가정합니다.

```
fiveteam.expectation.2017<-do.call(rbind,lapply(1:5,FUN = function(i){pythagorean.expectation(data.cleaning(KBO.2017,"2017",Topfive.team.2018[i]),Topfive.team.2018[i])}))
row.names(fiveteam.expectation.2017) <-Topfive.team.2018
fiveteam.expectation.2017
```

    ##      총득점 총실점  기대승률
    ## 두산    849    678 0.6105973
    ## 한화    737    820 0.4468434
    ## SK      761    767 0.4960734
    ## LG      699    677 0.5159843
    ## 넥센    789    764 0.5160937

이제 위에서 적용한 함수를 2018년도 데이터에도 적용해 상위 5개팀의 기대 승률을 구합니다.

```
Topfive.team.2018.expectation<-do.call(rbind,lapply(1:5,FUN = function(i){pythagorean.expectation(data.cleaning(KBO.2018.half,"2018",Topfive.team.2018[i]),Topfive.team.2018[i])}))
row.names(Topfive.team.2018.expectation)<-Topfive.team.2018
Topfive.team.2018.expectation
```

    ##      총득점 총실점  기대승률
    ## 두산    555    435 0.6194570
    ## 한화    438    441 0.4965871
    ## SK      480    405 0.5841415
    ## LG      503    458 0.5467238
    ## 넥센    497    472 0.5257826

총실점을 전년도 것으로 놓고 전년도 포스트시즌 진출팀의 평균 승률인 57%를 맞추려면 시즌 종료까지 총 몇 득점을 더 해야 하는지 추정해보겠습니다. 추정 방법은 다음과 같이 기대 승률을 구하는 공식을 역추하는 함수를 통해서 득점을 추정하는 것입니다. 즉 전년도 총 실점과 전년도 포스트시즌 진출팀의 평균 승률을 인자로 받으면서 미지수인 총득점의 제곱 값을 `sqrt()` 함수로 풀어주는 것을 통해 목표득점을 추정해 볼 수 있습니다. 최종적으로는 위에서 구한 현재까지의 총득점과 목표득점 간의 차이를 통해 시즌 끝까지 가을 야구 진출하기위한 평균 승률을 달성하기 위한 요구 득점을 알아낼 수 있습니다.

```
Runs.score.estimate<-function(x,y){#x=전년도총실점,y=전년도 포스트시즌진출팀 평균승률
a<-sqrt(y/(1-y)*x^2)
a<-round(a)
return(a)
}
Runs.score.estimate.2018<-Runs.score.estimate(fiveteam.expectation.2017$총실점,win.percentage.mean)
Runs.score.estimate.result<-data.frame(현재득점=Topfive.team.2018.expectation$총득점,목표득점=Runs.score.estimate.2018,요구득점=Runs.score.estimate.2018-Topfive.team.2018.expectation$총득점,row.names = Topfive.team.2018)
Runs.score.estimate.result
```

    ##      현재득점 목표득점 요구득점
    ## 두산      555      781      226
    ## 한화      438      945      507
    ## SK        480      884      404
    ## LG        503      780      277
    ## 넥센      497      880      383

이렇게 추정한 정보와 더불어 현재 기대 승률과 실제 승률 그리고 전년도 포스트시즌 진출팀의 평균 승률 간의 비교를 통해 상위 5개 팀의 가을 야구 가능성을 점쳐 볼 수 있습니다. 기대 승률과 평균 승률 간 비교하는 열은 전반기까지의 성적만으로도 포스트시즌에 가기에 충분한지 즉 기대 승률이 평균 승률과 같거나 높으면 전반기와 같은 득점과 실점 추세를 유지한다면 충분히 올라갈 수 있다는 의미합니다. 이어서 기대 승률과 실제 승률 간 비교 열은 전반기에 해당팀이 얼마나 전략적으로 잘해왔는지 혹은 운이 좋았는지를 말해줍니다.

```
table.result<-cbind(Topfive.team.2018.win.percentage,Topfive.team.2018.expectation,Runs.score.estimate.result[,2:3],평균승률=win.percentage.mean,기대vs평균=Topfive.team.2018.expectation$기대승률>win.percentage.mean,기대vs실제=Topfive.team.2018.expectation$기대승률>Topfive.team.2018.win.percentage$승률)

table.result
```
결과표

| 이름 | 승 | 패 |  무  |    승률   | 총득점 | 총실점 |  기대승률 |목표득점 |  요구득점  |     평균승률     |기대vs평균|기대vs실제|
|:----:|:--:|:--:|:----:|:---------:|:------:|:------:|:---------:|:------:|:--------:|:-------------:|:-----:|:-------:|
| 두산 | 58 | 29 | 없음 | 0.6666667 | 555    | 435    | 0.6194570 |  781   |   226    |   0.5704552   |  TRUE |  FALSE  |
| 한화 | 52 | 37 | 없음 | 0.5842697 | 438    | 441    | 0.4965871 |  945   |   507    |   0.5704552   | FALSE |  FALSE  |
| SK   | 48 | 37 | 1    | 0.5647059 | 480    | 405    | 0.5841415 |  884   |   404    |   0.5704552   |  TRUE |   TRUE  |
| LG   | 48 | 41 | 1    | 0.5393258 | 503    | 458    | 0.5467238 |  780   |   277    |   0.5704552   | FALSE |   TRUE  |
| 넥센 | 46 | 46 | 없음 | 0.5000000 | 497    | 472    | 0.5257826 |  880   |   383    |   0.5704552   | FALSE |   TRUE  |  





| 이름  | 목표득점 |  요구득점  |     평균승률     |기대vs평균|기대vs실제|
|:---- |:------:|:--------:|:-------------:|:-----:|:-------:|
| 두산  |  781   |   226    |   0.5704552   |  TRUE |  FALSE  |
| 한화  |  945   |   507    |   0.5704552   | FALSE |  FALSE  |
| SK   |  884   |   404    |   0.5704552   |  TRUE |   TRUE  |
| LG   |  780   |   277    |   0.5704552   | FALSE |   TRUE  |
| 넥센  |  880   |   383    |   0.5704552   | FALSE |   TRUE  |


위의 표에서 보면 두산과 SK의 현재 기대 승률만이 작년 포스트시즌 진출팀의 평균 승률을 넘고 나머지 팀은 넘지 못하는 것을 알 수 있습니다. 두산은 지금과 같은 추세를 유지한다면 문제없이 가을야구를 하게 될 것입니다. 현재 SK는 기대 승률을 57%로 유지하고 있지만, 아슬아슬하게 유지하고 있기에 득점을 좀 더 늘려야 안정적으로 포스트시즌에 진출할 수 있을 것입니다.

넥센과 LG의 경우에도 득점을 늘리고 실점을 줄여야 안정적으로 57%의 이상의 기대 승률을 달성할 수 있습니다. 물론 LG의 경우 필요로 하는 득점의 수가 그렇게 많지 않은 점을 감안하면 넥센보다는 수월하게 진출할 것입니다.

특이하게도 한화의 경우 현재 2위지만 기대 승률은 49%로 굉장히 낮습니다. 이런 모습의 원인은 한화가 득점보다 실점이 많기 때문입니다. 물론 실제 승률은 높지만, 그것은 전반기에 운이 좋았다라고 말할 수 있습니다. 왜냐하면 득점은 적고 실점이 많은 상태에서 승리가 많은 경우는 적은 점수차로 상대방 팀을 이긴 것이기 때문입니다. 따라서 만약 이러한 추세가 계속될 경우 한화는 실점이 많아지기에 순위가 내려갈 수 있습니다. 즉 한화는 현재 순위를 유지하기 위해서는 요구하는 득점이 가장 많은점을 보아 앞으로 득점력을 많이 올려야 하며 총실점이 총득점보다 많기에 실점 자체를 줄여야 한다고 생각합니다. 그렇지 않으면 한화는 앞으로 현재 순위를 유지하기 힘들 것입니다.

### 참고: 10개 팀 결과

다음 코드는 10개의 전체팀 결과를 보여줍니다.

```
Allteam<-unique(KBO.2018.half$원정팀)[1:10]

Allteam.win.percentage<-do.call(rbind,lapply(1:10,FUN = function(i){Yearly.Win.Per(data.cleaning(KBO.2018.half,"2018",Allteam[i]))}))
Allteam.expectation<-do.call(rbind,lapply(1:10,FUN = function(i){pythagorean.expectation(data.cleaning(KBO.2018.half,"2018",Allteam[i]),Allteam[i])}))
Allteam.expectation.2017<-do.call(rbind,lapply(1:10,FUN = function(i){pythagorean.expectation(data.cleaning(KBO.2017,"2017",Allteam[i]),Allteam[i])}))

Allteam.Runs.score.estimate.2018<-Runs.score.estimate(Allteam.expectation.2017$총실점,win.percentage.mean)
Allteam.Runs.score.estimate.result<-data.frame(현재득점=Allteam.expectation$총득점,목표득점=Allteam.Runs.score.estimate.2018,요구득점=Allteam.Runs.score.estimate.2018-Allteam.expectation$총득점,row.names = Allteam)

Allteam.table<-cbind(Allteam.win.percentage,Allteam.expectation,Allteam.Runs.score.estimate.result[,2:3],평균승률=win.percentage.mean,기대vs평균=Allteam.expectation$기대승률>win.percentage.mean,기대vs실제=Allteam.expectation$기대승률>Allteam.win.percentage$승률)
row.names(Allteam.table) <- Allteam
Allteam.table<-Allteam.table[order(Allteam.table$승률,decreasing = TRUE),]
Allteam.table
```

    ##     승 패    무        승률      총득점   총실점   기대승률     목표득점

    ## 두산 58 29 무승부 없음 0.6666667    555    435  0.6194570      781      
    ## 한화 52 37 무승부 없음 0.5842697    438    441  0.4965871      945      
    ## SK  48 37    1     0.5647059    480    405  0.5841415      884      
    ## LG  48 41    1     0.5393258    503    458  0.5467238      780      
    ## 넥센 46 46 무승부 없음 0.5000000    497    472  0.5257826      880      
    ## 기아 40 45 무승부 없음 0.4705882    485    465  0.5210433      856      
    ## 삼성 39 49    2     0.4431818    464    522  0.4413793     1050      
    ## 롯데 37 47    2     0.4404762    480    511  0.4687490      808      
    ## KT  35 50    2     0.4117647    441    486  0.4515704     1010      
    ## NC  34 56 무승부 없음 0.3777778    364    512  0.3357389      859     

    ##        요구득점     평균승률      기대vs평균   기대vs실제
    ## 두산     226      0.5704552       TRUE      FALSE
    ## 한화     507      0.5704552      FALSE      FALSE
    ## SK      404      0.5704552       TRUE       TRUE
    ## LG      277      0.5704552      FALSE       TRUE
    ## 넥센     383      0.5704552      FALSE       TRUE
    ## 기아     371      0.5704552      FALSE       TRUE
    ## 삼성     586      0.5704552      FALSE      FALSE
    ## 롯데     328      0.5704552      FALSE       TRUE
    ## KT      569      0.5704552      FALSE       TRUE
    ## NC      495      0.5704552      FALSE      FALSE
