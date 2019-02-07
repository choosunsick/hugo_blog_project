---
title: "KBO 경기에서 선수들의 기록 데이터 수집하고 분석하기"
date: 2019-02-07T16:30:58+09:00
draft: False
---

안녕하세요. LOPES 팀의 추선식 입니다. 이번에 저희 팀에서 KBO 사이트의 경기 리뷰와 관련된 자료를 파이썬(PYTHON)을 이용해 수집하는 코드를 작성했고 이를 공유하고자 합니다. 저희 팀에서 [이전에 수집한 데이터](https://github.com/choosunsick/KBO_data)는 경기 결과를 수집했습니다. 그러나 이 자료만으로는 할 수 있는 분석에 한계가 있었습니다. 또한, 미국의 MLB 자료처럼 한국의 KBO 자료도 쉽게 찾아보고 다루기 위해 KBO 경기의 선수들 기록을 수집하는 [프로젝트](https://github.com/LOPES-HUFS/KBO_Data_Wrangling)를 시작했습니다.
 위의 프로젝트에서 파이썬 코드로 데이터를 수집할 출처는 KBO 홈페이지의 경기 [리뷰 페이지](https://www.koreabaseball.com/Schedule/GameCenter/Main.aspx)입니다. 데이터를 수집하는 파이썬 코드를 작동시키기에 앞서 준비작업이 필요합니다. 먼저 위 프로젝트의 링크로 들어가서 깃(git)이나 [깃허브 데스크톱](https://desktop.github.com/)을 사용하여 이 프로젝트를 클론하거나 zip 파일로 다운로드해줍니다. 프로젝트의 파일들을 다운로드 한 이후 코드를 실행하는데 필요한 것들을 설치해야 합니다.

## 준비작업: pipenv와 selenium-chrome driver 설치

윈도우에서 파이썬 가상환경, 셀레니움 크롬드라이버의 설치는 다음과 같습니다. 먼저 [아나콘다](https://www.anaconda.com/download/#windows)로 파이썬을 설치합니다. 아나콘다를 통해 파이썬이 설치된 이후 pipenv와 셀레니움-크롬드라이버의 설치는 자세한 내용은 다음 [링크](https://github.com/LOPES-HUFS/KBO_Data_Wrangling/wiki/윈도우에-셀리리움-설치)를 참고해 주시면 감사하겠습니다.
 우분투와 맥의 경우 가상환경의 설치는 [가상환경설치](https://pipenv.readthedocs.io/en/latest/)를 참조하시면 됩니다. 이어서 우분투와 맥에서 셀레니움과 크롬드라이버의 설치입니다. 먼저 우분투에서 셀레니움과 크롬드라이버의 설치는 다음의 코드로 쉽게할 수 있습니다.

```
sudo apt-get install chromium-chromedriver
sudo apt-get install python3-selenium
```

맥에서 크롬드라이버 설치는 [셀레니움 크롬드라이버 설치](http://www.epistemology.pe.kr/2018/09/25/1153)를 참조하시면 됩니다. 셀레니움과 크롬드라이버가 설치되어 가상환경에서 제대로 작동된다면 코드를 작동시키기 위한 준비는 끝났습니다. 준비작업에 대한 자세한 내용은 프로젝트의 [README 파일](https://github.com/LOPES-HUFS/KBO_Data_Wrangling/blob/master/README.md)을 참고해주세요.

## 한 경기의 데이터 수집하고 확인해보기

위에서 준비작업을 마쳤으면 가상환경에서 프로젝트를 실행시킬 수 있습니다. 먼저 이 프로젝트를 받은 폴더 위치(pipfile.lock 파일이 있는 위치)로 이동합니다. 그 다음 아래의 코드를 통해 가상환경을 실행하여 프로젝트에서 사용하는 패키지들을 받아줍니다.

```
pipenv shell
pipenv install
```

가상환경이 실행되고 필요한 패키지들이 받아졌으면 이제 파이썬을 실행할 차례입니다. 맥이나 우분투에서는 아래의 코드로 가상환경에서 파이썬 3을 실행하실 수 있습니다. 윈도우에서는 conda를 기준으로 `python3`가 아닌 `python`을 쳐야 실행됩니다. 참고로 쥬피터 노트북으로 진행할 경우 아래의 코드에서 `python3` 대신 `jupyter notebook`을 치면 됩니다.

```
#윈도우에서는 python
python3
```

이제 직접 경기를 수집해 보겠습니다. 예를 들어 2018년 10월 10일 KT와 롯데 간 첫 번째 경기를 수집해보겠습니다. 이 경기의 리뷰 내용을 가져와 데이터를 하나씩 확인하는 코드는 주피터 노트북 파일에 있기에 여기서는 단순히 확인만 하고 넘어가겠습니다.

```
import main
import pandas as pd
temp_data = main.get_data("20181010","KTLT1")
```

위의 코드를 실행시키면 2018년 10월 10일 KT와 롯데 간의 1차전 경기 자료가 temp_data에 들어오게 됩니다. 아래는 해당 경기의 리뷰페이지에서 들어온 자료를 확인하는 코드입니다.

```
pd.DataFrame(temp_data['20181010_KTLT1']['scoreboard'])
temp_data['20181010_KTLT1']['ETC_info']
pd.DataFrame(temp_data['20181010_KTLT1']['away_batter']).head()
pd.DataFrame(temp_data['20181010_KTLT1']['away_pitcher']).head()
pd.DataFrame(temp_data['20181010_KTLT1']['home_batter']).head()
pd.DataFrame(temp_data['20181010_KTLT1']['home_pitcher']).head()
```

위에서 수집한 자료를 다루기 편하게 만들기 위해 일부 정보를 추가하고 변경해 보겠습니다. 아래의 코드를 작동시키면 원정팀과 홈팀의 타자 데이터와 투수 데이터가 어떻게 변했는지 확인할 수 있습니다.

```
temp = main.modify_data(temp_data)

pd.DataFrame(temp['20181010_KTLT1']['away_batter']).head()
pd.DataFrame(temp['20181010_KTLT1']['away_pitcher']).head()
pd.DataFrame(temp['20181010_KTLT1']['home_batter']).head()
pd.DataFrame(temp['20181010_KTLT1']['home_pitcher']).head()
```

위에서 변경된 데이터를 확인하면 타자의 타격기록이 숫자로 범주화된 것을 확인할 수 있습니다. 한 타자가 한 이닝에 두 타석 들어올 경우가 있을 수 있기에 타격기록을 범주화하여 다루기 편하게 만들었습니다. 투수 데이터는 계산의 편의를 위해 이닝 열을 분리했습니다. 기존에 1이닝 이상인 경우 이닝의 정수 부분만 이닝 열로 두고 나머지 부분을 잔여이닝 열로 만들어 분리했습니다.

##2018년 정규시즌 한화의 경기 자료로 분석해보기

앞서 한 경기를 시범 삼아 수집해 보았으니 실제로 자료를 분석해보기 위해 2018년 한화의 정규시즌 경기를 모두 수집해 보겠습니다. 이미 2018년 한화의 경기를 수집한 결과가 sample 폴더에 Hanhwa_normalseason_2018.json 파일로 있습니다. 만약 직접 코드를 통해 자료를 만드실려면 쥬피터 노트북의 코드를 작동시키면 2018 정규시즌 한화의 경기 자료가 dict 형식으로 만들어집니다.
이제 준비된 파일을 열어 약간의 처리 과정을 거치면 2018년 한화와 상대 팀의 타자, 투수 데이터가 만들 수 있습니다. 자세한 코드는 쥬피터 노트북에 있기에 일부만 소개하고 넘어가겠습니다.

```
#파일을 열어줍니다.
temp_file_name = "./sample/hanhwa_normalseason_2018.json"

with open(temp_file_name) as outfile:  
    hanhwa_data = json.load(outfile)

away_team = pd.DataFrame()
home_team = pd.DataFrame()

#away_batter와 home_batter 대신 away_pitcher와 home_pitcher를 넣으면 투수 데이터를 만들 수 있습니다.

for i in hanhwa_data.keys():
    away_team = away_team.append(pd.DataFrame(hanhwa_data[i]['away_batter']),sort=True)
    home_team = home_team.append(pd.DataFrame(hanhwa_data[i]['home_batter']),sort=True)

hanhwa_2018_all = pd.concat([away_team,home_team],ignore_index=True)
hanhwa_2018_all = hanhwa_2018_all.fillna(0)

hanhwa_batter = pd.DataFrame(hanhwa_2018_all,
  columns=['경기날짜','선수명','포지션','팀',"더블헤더여부",'홈팀','원정팀','1','2','3','4','5','6','7','8','9','10','11','12','안타','타수','타율','타점','득점'])

#투수 데이터일 경우
hanhwa_pitcher = pd.DataFrame(hanhwa_2018_all,
  columns=['경기날짜','선수명','포지션','팀','더블헤더여부','홈팀','원정팀','등판', 'inning', 'restinning', '승리','패배','무승부','세이브','홀드','삼진','4사구','실점','자책', '타수', '타자'])
```

위 코드를 작동시키고 나면 2018년 한화의 타자, 투수 데이터가 만들어집니다. 그럼 이 데이터로 어떤 분석을 할 수 있는지 알아보겠습니다. 간단하게 타자 개인의 타율부터 시작해서 안타, 득점, 타점의 개수와 1, 2, 3 루타와 홈런의 개수, 피삼진, 볼넷, 병살타의 개수, 출루율, 장타율 등을 구해볼 수 있습니다. 투수 쪽 기록 역시 방어율과 투구 수, 상대 타자수, 총 이닝, 피홈런 수, 피안타 수, 홀드, 세이브, 삼진과 볼넷의 개수, 이닝당 투구 수, 승률 등을 구할 수 있습니다. 개인뿐만 아니라 팀 전체의 기록도 분석할 수 있습니다. 그럼 간단하게 팀 타율과 팀 방어율을 구해 보겠습니다.

```
# 한화의 팀타율을 구해봅니다.
# 한화의 상대팀 또한 자료에 들어있기 때문에 오직 한화 소속 선수들의 자료를 얻기위해 조건을 걸어줍니다.

h = sum(hanhwa_batter[hanhwa_batter['팀'] == "한화"]['안타'])
ab = sum(hanhwa_batter[hanhwa_batter['팀'] == "한화"]['타수'])
# 팀타율
h/ab

# 팀 자책점을 구합니다.
er = sum(hanhwa_pitcher[hanhwa_pitcher['팀'] == "한화"]['자책'])*9

# 팀 총 이닝을 구합니다.
# 잔여 이닝 열을 사용할 때는 /3을 해주어야 원래의 분수 값이 나옵니다.
ip = (sum(hanhwa_pitcher[hanhwa_pitcher['팀'] == "한화"]['inning'].astype(int))+
    sum(hanhwa_pitcher[hanhwa_pitcher['팀'] == "한화"]['restinning'].astype(int))/3)

# 팀방어율
er/ip
```
지금까지 KBO에서 자료를 수집하고 이것을 통해 간단한 분석을 해봤습니다. 참고로 2018년 한화의 데이터 뿐만 아니라 2010년부터 2018년까지의 정규시즌 자료를 정리한 파일이 data 폴더에 있습니다. 덧붙여 [저희 홈페이지](http://lopes.hufs.ac.kr)에서 진행하고 있는 다른 프로젝트들도 확인해 보실 수 있습니다. 긴 글 읽어주셔서 감사합니다.
