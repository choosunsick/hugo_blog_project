---
title: "샤이니 앱에 테마 적용하기"
date: 2020-09-12T19:01:31+09:00
draft: False
tags: ["R", "테마", "샤이니","shiny"]
categories: ["R 샤이니"]
---

이전 글에서 샤이니 앱을 도커화(dockerizing) 해보았습니다. 이에 대한 자세한 내용은 [이전 글](https://choosunsick.github.io/post/dockerizing/ )을 참고해주시면 감사하겠습니다. 자신만의 샤이니 앱을 만들 때 샤이니 앱 테마를 적용하는 방법에 대해 소개하겠습니다. 샤이니 앱에 테마를 적용하는 방법은 아주 간단합니다. 참고로 샤이니 테마란 샤이니 테마란 폰트와 배경 색, 글자색 등을 변경해주는 것입니다.

먼저 샤이니 테마를 사용할 수 있게 패키지를 인스톨하고 불러와 줍니다. 이전 글에서 기존의 app.R 파일에 ui 윗 부분에 `library(shinythemes)`를 추가해줍니다. 일반적으로 R에서 외부 라이브러리를 사용할 때 `install.packages("shinythemes")` 와 같은 코드도 쳐주어야 하는데, 샤이니앱을 도커화 했기 때문에 외부 라이브러리 설치는 도커파일에서 이루어지게 됩니다.

## Dockerfile 변경사항

기존에 Dockerfile에서 패키지를 인스톨 하는 부분을 다시 살펴 봅시다. 기존에는 shiny, shinydashboard, tidyverse 3가지 패키지만 인스톨 했습니다. 여기에 `RUN R -e "install.packages('shinythemes', repos='http://cran.rstudio.com/',dependencies=TRUE)"` 한 줄을 추가해줍니다. 기존 코드들과 달리 dependencies=TRUE는 해당 라이브러리의 의존성이 있는 패키지까지 설치 해주는 기능입니다. 그러면 외부 라이브러리 설치가 이루어집니다.

```
# install R packages required
# (change it dependeing on the packages you need)
RUN R -e "install.packages('shiny', repos='http://cran.rstudio.com/')"
RUN R -e "install.packages('shinydashboard', repos='http://cran.rstudio.com/')"
RUN R -e "install.packages('tidyverse', repos='http://cran.rstudio.com/',dependencies=TRUE)"
RUN R -e "install.packages('shinythemes', repos='http://cran.rstudio.com/',dependencies=TRUE)
```

## ui 함수 변경사항  

샤이니 테마 패키지의 설치가 끝났으니 이제 테마를 적용해볼 차례 입니다. app.R의 ui 부분에 한 줄만 추가해주면 샤이니 테마의 적용이 끝납니다. 기존 ui 부분의 코드에서 `fluidpage()` 함수에 `theme = shinytheme("cerulean")` 를 추가 해줍니다. 이러면 테마의 적용이 완료됩니다. 제가 적용한 테마의 이름은 cerulean로 다른 테마를 적용해 보실 분들은 [샤이니 테마 홈페이지](https://rstudio.github.io/shinythemes/ )에서 자신이 원하는 테마의 이름을 적어주시면 됩니다.

```R
ui <- fluidPage(theme = shinytheme("cerulean"),

    # Application title
    titlePanel("mtcars Hp Data"),

    # Sidebar with a slider input for number of bins
    sidebarLayout(
        sidebarPanel(
            sliderInput("bins",
                        "Number of bins:",
                        min = 1,
                        max = 30,
                        value = 15)
        ),

        # Show a plot of the generated distribution
        mainPanel(
           plotOutput("distPlot")
        )
    )
)
```

## 이미지 빌드하고 웹에서 확인하기

앱 부분에 변경 사항이 생겼으니 다시 도커 이미지를 빌드해주어야 합니다. 이미지 빌드 방법은 이전 글과 똑같이 이루어 집니다. 터미널이나 CMD에서 아래와 같이 치면 메세지와 함께 이미지가 만들어집니다. 앱을 실행 시키는 것 역시 같습니다. 이미지 빌드가 끝난 이후 `docker run -p 3838:3838 test-shiny-app` 를 치면 이미지가 실행되어 앱이 실행됩니다. 웹 상에서 실행된 앱을 http://localhost:3838/ 에서 확인해보면 기존과 달리 테마의 적용으로 제목의 글자색이 파란색으로 변경된 것을 확인하실 수 있습니다.

```
> docker build -t test-shiny-app  .
> docker run -p 3838:3838 test-shiny-app
```
