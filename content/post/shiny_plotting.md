---
title: "R 샤이니: 자신만의 앱에 그래프 올리기"
date: 2020-09-12T16:00:31+09:00
draft: False
tags: ["R", "plot", "샤이니","shiny"]
categories: ["R 샤이니"]
---
# 자신만의 샤이니 앱 만들기

 이전 글에서 샤이니 앱을 도커화(dockerizing) 해보았습니다. 이에 대한 자세한 내용은 [이전 글](https://choosunsick.github.io/post/dockerizing/)을 참고해주시면 감사하겠습니다. 이제 자신만의 샤이니 앱을 만들 때 웹페이지에서의 선택(클릭)에 따라 유동적으로 변하는 그래프를 그리는 방법에 대해 알아보겠습니다. 이에 따라 이 글에 최종 목적은 KBO의 팀별 승패 그래프를 팀 선택에서 따라 다르게 변화하는 그래프를 그려보는 것입니다.

## 자신만의 샤이니 앱을 만들기 위한 준비

이제 자신만의 앱을 만들기 위한 준비가 필요합니다. 앱에서 사용할 외부 데이터를 추가하고 외부 라이브러리도 설치하고 추가해주어야 합니다.

### 샤이니 앱에 외부데이터 추가하기

 자신만의 샤이니 앱을 만들 때 자신이 가지고 있는 데이터를 사용하는 것은 거의 필수적입니다. 외부 데이터를 자신의 샤이니 앱에 추가하는 방법에 대해 알아봅시다. 먼저 데이터를 저장할 data 폴더를 app.r 파일과 Dockerfile이 있는 위치에 만들어 줍니다. 그리고 자신이 사용할 외부 데이터를 폴더에 넣어 줍니다. Dockerfile을 열어 기존에 COPY 부분에 `COPY data /srv/shiny-server/data/` 해당 명령어를 추가해 줍니다. 이 명령어는 데이터 폴더를 복사하고 도커에서 이미지를 만들때 data 폴더의 위치를 `/srv/shiny-server/data/` 로 사용한다는 의미입니다.
 그렇다면 데이터 폴더 안에 넣은 데이터는 app.R에서 어떻게 사용할 수 있을까요 ? 예를 들어 KBO 팀들 간 승, 패 기록을 저장한 데이터를 사용한다고 해봅시다. 이 데이터는 [링크](https://github.com/choosunsick/r_shiny_test)의 데이터 폴더에서 다운받을 수 있습니다. 이 데이터가 data 폴더에 넣어주면 이제 데이터를 사용할 준비가 완료됩니다. app.R에서 데이터를 사용하는 방법은  `scoreboard <- read.csv("./data/scoreboard.csv",stringsAsFactors = F)` 이런식으로 상대주소를 사용해 읽어올 수 있습니다. 이 코드는 app 파일에서 ui와 server 부분 윗부분에 선언하여 글로벌 변수로서 ui나 server에서 모두 사용할 수 있게 해줍니다.

### 외부 패키지 설치하고 추가하기

 자신만의 앱을 만들고 플롯팅을 하려면 `ggplot2`와 같은 라이브러리와 다른 외부 패키지가 필요한 경우가 생기게 됩니다. 물론 R의 기본 함수만 사용할 경우 외부 패키지를 불러올 필요가 없습니다. 그러나 저희는 ggplot2, tidyverse와 같은 패키지를 사용할 예정이기에 외부 패키지를 사용할 수 있도록 app.R 파일에서 `library(ggplot2)`와 같이 사용합니다. 일반적으로 R에서 외부 라이브러리를 사용할 때  install.packages() 함수도 같이 사용하는데 인스톨 부분이 빠진 이유는 Dockerfile에서 인스톨 해주기 때문입니다. 따라서 app.R에서 사용하는 외부 패키지들은 모두 Dockerfile에서 install이 필요합니다.

 기존에 Dockerfile에서 다운 받는 패키지들은 tidyverse, shiny, shinydashboard 3가지 패키지만 인스톨 했습니다. 그리고 패키지의 의존성을 제외하고 다운 받았습니다. 그러나 이럴 경우 패키지 내부에서 사용하는 의존성 패키지의 관련된 함수를 사용하지 못하게 됩니다. 따라서 `RUN R -e "install.packages('ggplot2', repos='http://cran.rstudio.com/',dependencies=TRUE)"` 와 같이 필요한 패키지를 의존성(`dependencies=TRUE`)과 함께 인스톨해줍니다. 이제 Dockerfile에서 새로 인스톨 되는 패키지는 다음과 같습니다.

```
RUN R -e "install.packages('RSQLite', repos='http://cran.rstudio.com/',dependencies=TRUE)"
RUN R -e "install.packages('DBI', repos='http://cran.rstudio.com/',dependencies=TRUE)"
RUN R -e "install.packages('dplyr', repos='http://cran.rstudio.com/',dependencies=TRUE)"
RUN R -e "install.packages('ggplot2', repos='http://cran.rstudio.com/',dependencies=TRUE)"
RUN R -e "install.packages('shinythemes', repos='http://cran.rstudio.com/',dependencies=TRUE)"

```

## 자신만의 UI 구성하고 테마 적용하기

 이제 자신의 샤이니 앱 UI를 꾸며줄 수 있는 테마를 적용해 봅시다. app.R에서 `library(shinythemes)` 샤이니 테마 패키지를 불러와 줍니다. UI를 구성하는 페이지에는 동적페이지(fluidpage), 고정형 페이지(fixedpage), 대쉬보드형 페이지(dashboardpage), 내비게이터 형 페이지(navbarpage) 등이 있습니다. 기본적으로 사용되는 페이지는 fluidpage입니다. 앞선 글에서 앱을 구동할 때도 fluidpage 방식으로 앱의 UI를 구성했습니다. 동적 페이지를 기본적으로 사용하는 이유는 웹과 사용자간의 상호작용 때문입니다.

 ```R
library(shiny)
library(RSQLite)
library(dplyr)
library(tidyverse)
library(ggplot2)
library(shinythemes)
library(shinydashboard)
 ```

 UI를 구성하는 페이지 종류를 살펴보았으니 해당 페이지에 테마를 적용하는 방법에 대해 알아봅시다. 샤이니 테마란 폰트와 배경 색 글자색 등을 변경해주는 것입니다. 샤이니의 다양한 테마에 대한 것은 [샤이니 테마 홈페이지](https://rstudio.github.io/shinythemes/)를 참조하세요. 테마를 적용하는 방법은 간단합니다. 어떤 페이지를 사용하든 `theme = shinytheme("cerulean")` 와 같이 코드 한 줄만 추가해 주면 됩니다. 여기서 "cerulean" 부분은 샤이니 테마의 이름입니다. 원하시는 테마를 샤이니 테마 홈페이지에서 보고 직접 골라서 사용할 수 있습니다.
 이제 다른 UI 구성을 살펴볼 차례 입니다. 먼저 아래의 코드를 살펴봐 주시기 바랍니다. 아래 코드의 첫 번째 줄은 `navbarPage("KBO 그래프",theme = shinytheme("cerulean"),` 샤이니 앱의 제목을 정하고 테마를 적용하는 코드입니다. 참고로 여기서 사용하는 navbarpage는 내비게이터 형 페이지인데 차후 다른 항목으로 확장할 가능성을 열어두기 위해서 사용합니다. 두 번째 줄은 첫 번째 항목을 생성하는 역할입니다. `tabPanel`은 내비게이터 형 페이지에서 항목을 생성해 줍니다.
 이제 해당 탭의 내용을 만들 차례입니다. `sidebarLayout()` 함수를 통해 해당 탭에서 보여질 UI를 구성합니다. `sidebarLayout()` 은 크게 2가지로 나누어 구성되는데, `sidebarPanel()`과 `mainPanel()` 입니다. `sidebarPanel()`은 왼쪽 상단의 위치에서 UI를 구성합니다. 저희가 구성할 것은 KBO 팀을 선택할 수 있는 항목과 자료의 출처인 KBO 홈페이지에 대한 링크입니다. `navlistPanel(widths = c(12,12),"원하는 팀을 선택하세요",` 이 부분은 항목의 크기를 설정하고 제목을 설정하는 부분입니다. `tabPanel(selectInput("team","Select Team",unique(scoreboard$팀)))` 이 부분은 앱 상에서 사용자가 선택할 수 있는 항목을 만들어 주는 부분으로 `selectInput()` 함수가 선택형 항목을 제공해 줍니다. 이 코드에서 선택이 대상이 되는 것은 `unique(scoreboard$팀)`으로 KBO의 전체 팀 명이 됩니다. `tags$a(h3("자료출처:KBO"),href = "https://www.koreabaseball.com/")` 코드는 KBO 홈페이지에 링크를 걸어주는 역할을 합니다. tags는 HTML 태그를 말하는 것으로 링크 기능 외에도 다양한 HTML 태그를 사용할 수 있습니다.
 다음으로 `mainPanel()`함수는 사이드 바 위치가 아닌 중앙이나 오른쪽 부분의 UI를 구성합니다. 내비게이터 형 페이지를 사용하여 여러 항목으로 확장성을 둘 수 있습니다. `tabPanel("팀 승패 그래프",plotOutput("plot")),` 로 하나의 탭을 만들고 팀 별 승패 그래프를 출력하도록 설정합니다. 출력 부분은 이후 서버 구성과 연관되어 있습니다. 서버 부분에서 만들어진 output의 이름이 곧 출력하는 항목이 되기 때문입니다. 이에 대해서는 server 코드를 살펴볼 때 더 자세히 설명하겠습니다.

```R
ui <- navbarPage("KBO 그래프",theme = shinytheme("cerulean"),
                 tabPanel("팀 그래프",
                     sidebarLayout(sidebarPanel(navlistPanel(
                         widths = c(12,12),"원하는 팀을 선택하세요",
                         tabPanel(selectInput("team","Select Team",unique(scoreboard$팀))),
                         tags$a(h3("자료출처:KBO"),href = "https://www.koreabaseball.com/")
                     )),
                     mainPanel(navbarPage(title="그래프 종류",
                         tabPanel("팀 승패 그래프",plotOutput("plot")),
                         tabPanel("팀 기록 그래프")))
                     )),
                    tabPanel("선수 그래프")
                 )

```

## Server 함수 구성하여 앱의 출력물 만들기

Server 부분은 앱에서 결과를 출력하는 부분입니다. 결과를 만들기 앞서 읽어온 csv 데이터를 DB화 해줍니다. 다음과 같은 방법으로 DB화 할 수 있습니다. DB화 하는 이유는 이후 데이터를 변경하거나 업데이트 할때 편리하기 때문입니다.

```R
con <- DBI::dbConnect(RSQLite::SQLite(), ":memory:")
    copy_to(con, scoreboard)
    scoreboard_db <- tbl(con, "scoreboard")
```

scoreboard_db 란 객체는 DB 객체로 실제의 값이 저장되어 있지는 않습니다. 따라서 실제 값을 취하기 위해 반응 함수가 필요합니다. 실제의 값을 사용하거나 사용자의 선택에 따라 변경되는 변수등을 사용하기 위해 방법으로는 3가지가 있습니다. `reactive(), observe(), observeEvent() ` 세가지 입니다. 이 3가지의 차이점은 [링크](https://stackoverflow.com/questions/53016404/advantages-of-reactive-vs-observe-vs-observeevent)를 참조해주시길 바랍니다. 우리는 DB 객체에서 값을 사용하기 위해서 reactive 함수를 사용해야 합니다. 아래와 같이 사용하면 DB 데이터를 dta라는 함수로 일반 데이터와 같이 사용할 수 있습니다. 함수 중간에 {}는 줄 바꿈이나 내용이 두 줄 이상 될 경우 필수적으로 사용하는 것 입니다.

```R
dta <- reactive({
        scoreboard_db
    })
```

이제 팀 별 승패 그래프를 그리기 위해 선택된 팀에 대한 년도별 승패 기록을 뽑아내야 합니다. 여기서는 사용자 선택에 따라 변경되는 변수인 team이 있기 때문에 `observe()` 함수를 사용합니다. 사용자에 선택에 따라 변경되는 값은 UI에 selectInput 부분에서 team이라는 이름으로 정의했습니다. 정의한 것에 따라 input$team과 같이 사용할 수 있습니다. tidyverse의 `%>%, filter(),summarize()` 등을 통해 년도별 승과 패 기록을 뽑을 수 있습니다. 이 작업은 for문을 통해 2010~2020년까지 반복됩니다.  

```R
observe({
        temp <- input$team
        win <- data.frame()
        lose <- data.frame()
        for(i in 2010:2020){
            win <- rbind(win,dta() %>%
                filter(팀==temp) %>%
                filter(year == i) %>%
                summarize(year=year,sum=sum(승패=="승")) %>%
                collect())
            lose <- rbind(lose,dta() %>%
                              filter(팀==temp) %>%
                              filter(year == i) %>%
                              summarize(year=year,sum=sum(승패=="패")) %>%
                              collect())
        }
```

데이터가 준비되었으니 이 데이터를 이용해 Plot을 그리는 일만 남았습니다. 출력하는 결과가 플롯이기에 `renderPlot()` 함수를 사용합니다. 출력결과가 테이블이라면 `renderTable()` 함수를 사용할 수 있습니다. ggplot의 기능을 이용해 년도별 승과 패 그래프를 그려 줍니다. 코드는 아래와 같습니다. 여기서 `renderPlot()`을 output$plot으로 저장하는데 이것이 UI의 `plotOutput("plot")` 부분에서 plot이라는 이름으로 사용하게 됩니다. 만약 다른 이름을 설정할 경우 UI의 부분도 마찬가지로 바꾸어야 합니다.

```R
 output$plot <- renderPlot({
            ggplot(win,aes(x=year,y=sum))+ geom_line(aes(colour = 'blue'))+
                geom_line(data = lose,aes(x=year,y=sum,colour = 'red'))+
                scale_x_continuous("year",limits = c(2010,2021),breaks = seq(2010,2020,2))+
                scale_color_discrete(name = "Win_or_Lose", labels = c("Win", "Lose"))
        })
```

server 부분의 전체 코드는 아래와 같습니다.

```R
server <- function(input, output, session) {
    con <- DBI::dbConnect(RSQLite::SQLite(), ":memory:")
    copy_to(con, scoreboard)
    scoreboard_db <- tbl(con, "scoreboard")
    dta <- reactive({
        scoreboard_db
    })

    observe({
        temp <- input$team
        win <- data.frame()
        lose <- data.frame()
        for(i in 2010:2020){
            win <- rbind(win,dta() %>%
                filter(팀==temp) %>%
                filter(year == i) %>%
                summarize(year=year,sum=sum(승패=="승")) %>%
                collect())
            lose <- rbind(lose,dta() %>%
                              filter(팀==temp) %>%
                              filter(year == i) %>%
                              summarize(year=year,sum=sum(승패=="패")) %>%
                              collect())
        }
        output$plot <- renderPlot({
            ggplot(win,aes(x=year,y=sum))+ geom_line(aes(colour = 'blue'))+
                geom_line(data = lose,aes(x=year,y=sum,colour = 'red'))+
                scale_x_continuous("year",limits = c(2010,2021),breaks = seq(2010,2020,2))+
                scale_color_discrete(name = "Win_or_Lose", labels = c("Win", "Lose"))
        })
    })
}
```

app.R의 마지막 코드인 `shinyApp(ui = ui, server = server)` 부분은 변경이 없습니다.
