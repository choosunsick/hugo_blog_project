---
title: "Request_html 패키지 소개"
date: 2019-09-30T15:50:02+09:00
draft: False
---

안녕하세요. LOPES 팀의 추선식 입니다. 이번에 기존의 KBO의 자료를 모으는 프로젝트를 개선하면서 자료를 모으는 데 좋은 패키지를 찾아서 한번 소개를 해볼까 합니다. 제가 소개할 패키지는 Request_html이라는 패키지입니다. 패키지의 이름에서 볼 수 있듯이 이 패키지는 Request 패키지와 연관되어 있습니다. Request 패키지는 여러분이 웹상에서 자료를 수집할 때 가장 흔하게 접할 수 있는 패키지입니다. 저 또한, 개인 프로젝트를 진행할 때 Request 패키지를 사용해 자료를 모으려 했지만 어려움이 있었습니다. 그래서 셀레니움(Selenium)과 크롬드라이버(Chrome-driver)를 사용하여 KBO의 자료를 모았지만 Request_html을 이용해서 편하게 자료를 모을 수 있었습니다.

이 패키지는 Request와 같이 웹상의 자료를 수집할 수 있는데 Request_html은 html을 파싱하는데 더 뛰어나고, 자바스크립트를 지원한다는 차이점이 있습니다. 자바스크립트를 지원한다는 것은 자바스크립트로 렌더링 된 웹 페이지를 html로 풀어낼 수 있다는 말입니다. 이런 설명만으로는 어떤 기능인지 쉽게 알기 어렵기에 실제로 Request와 Request_html을 이용해서 동적 페이지인 KBO 리뷰 페이지의 자료를 가져와 보겠습니다.

## Request 패키지를 통해 자료 가져와보기

필요한 패키지들을 미리 다운받아줍니다.

```
pip install requests
pip install requests_html
pip install beautifulsoup4
pip install lxml
```

패키지 설치가 끝나면 파이썬을 실행하여 패키지들을 import하고 리퀘스트 패키지를 이용해 자료를 모아봅시다.

```
>>> import requests
>>> from requests_html import HTMLSession
>>> from bs4 import BeautifulSoup
>>> import lxml

>>> temp_url = 'https://www.koreabaseball.com/Schedule/GameCenter/Main.aspx?gameDate=20100327&gameId=20100327HTOB0&section=REVIEW'

>>> temp = requests.get(temp_url)
>>> soup = BeautifulSoup(temp.text, 'lxml')
>>> soup.find_all('table')
[]
```

위의 코드를 보면 Requests를 사용했을 때는 결과가 아무런 값도 없는 리스트임을 확인할 수 있습니다. 만약에 입력한 주소의 자료를 가져왔다면 해당 리뷰 페이지에 있는 첫 번째 테이블인 스코어보드에 대한 정보가 보여야 합니다. 그러나 보시다시피 Request 패키지로는 자료를 가져오는 데 실패했습니다. 물론 셀레니움과 크롬드라이버를 설치해서 이용하면 자료를 가져올 수 있습니다. 그렇다면 이제 Request_html 패키지를 사용해서 정보를 가져와 봅시다.

## Requset_html 패키지를 통해 자료 가져오기

```
>>> session = HTMLSession()
>>> r = session.get(temp_url)
>>> r.html.render()
>>> soup = BeautifulSoup(r.html.html, "lxml")
>>> soup.find_all('table')[0]
<table class="tbl" id="tblScordboard1">
<colgroup>
<col width="35%"/>
<col width="65%"/>
</colgroup>
<thead>
<tr>
<th> </th>
<th>TEAM</th>
</tr>
</thead>
<tbody><tr><td class="lose">패</td><th><span class="logo"><img alt="" src="//crdfcowjurxm984864.cdn.ntruss.com/resources/images/emblem/regular/2018/initial_HT.png"/></span>0승 1패 0무</th></tr><tr><td class="win">승</td><th><span class="logo"><img alt="" src="//crdfcowjurxm984864.cdn.ntruss.com/resources/images/emblem/regular/2018/initial_OB.png"/></span>1승 0패 0무</th></tr></tbody></table>
```

먼저 `HTMLSession()` 코드로 세션을 열어줍니다. 이 작업은 마치 셀레니움과 크롬드라이버를 사용할 때 크롬드라이버의 위치를 선택해 잡아주는 것이랑 똑같은 것입니다. 이어서 `session.get()`은 셀레니움과 크롬드라이버에서 `driver.get()`의 역할과 같습니다. 가장 중요한 자바스크립트로 구성된 페이지를 푸는 것은 `r.html.render()` 코드로 이루어집니다. 참고로 render() 함수를 처음 사용할 때 작동이 안 되거나 에러가 발생한다면 `pip3 install pyppeteer`로 해결할 수 있습니다. 마지막 코드 결과를 보면 셀레니움과 크롬드라이버가 없이도 리뷰 페이지의 테이블 내용이 가져온 것을 확인할 수 있습니다.

Request_html을 사용하면 자바스크립트로 된 동적 페이지에 대한 자료 수집을 하는 경우에는 셀레니움과 웹 드라이버를 설치하고 웹 창을 새로 띄우는 번거로움을 느낄 필요가 없습니다. 물론 이 패키지가 셀레니움과 웹 드라이버의 모든 기능을 대체할 수 있는 것은 아닙니다. 예를 들면 클릭이 필요한 웹사이트의 자료를 수집할 경우에는 Request_html로 대체할 수 없습니다.

추가로 Request_html 패키지의 기능들에 어떤 것이 있는지 더 자세히 알고 싶은 분들은 [패키지 설명서 링크](https://pypi.org/project/requests-html/)를 참조해주시기 바랍니다.
