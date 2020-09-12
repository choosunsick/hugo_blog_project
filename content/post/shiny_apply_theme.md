---
title: "샤이니 앱에 테마 적용하기"
date: 2020-09-12T19:01:31+09:00
draft: False
tags: ["R", "테마","theme", "샤이니","shiny"]
categories: ["R 샤이니"]
---

 이전 글에서 샤이니 앱을 도커화(dockerizing) 해보았습니다. 이에 대한 자세한 내용은 [이전 글](https://choosunsick.github.io/post/dockerizing/ )을 참고해주시면 감사하겠습니다. 자신만의 샤이니 앱을 만들 때 샤이니 테마를 적용하는 방법에 대해 소개하겠습니다. 샤이니 앱에 테마를 적용하는 방법은 아주 간단합니다. 참고로 샤이니 테마란 폰트와 배경 색, 글자색 등을 변경해주는 것입니다.

 먼저 샤이니 테마를 사용할 수 있게 패키지를 인스톨하고 불러와 줍니다. 이전 글에서 기존의 app.R 파일에 UI 윗 부분에 `library(shinythemes)`를 추가해 줍니다. 일반적으로 R에서 외부 라이브러리를 사용할 때 `install.packages("shinythemes")` 와 같은 코드도 쳐주어야 하는데, 샤이니앱을 도커화(dockerizing) 했기 때문에 외부 라이브러리 설치는 Dockerfile 에서 이루어지게 됩니다.

## Dockerfile 변경사항

Dockerfile에서 패키지를 인스톨 하는 부분을 다시 살펴 봅시다. 이전 글과 달라진 부분은 6번 줄 부분입니다. 참고로 dependencies=TRUE는 해당 라이브러리의 의존성이 있는 패키지까지 설치 해주는 기능입니다.

```
# install R packages required
# (change it dependeing on the packages you need)
RUN R -e "install.packages('shiny', repos='http://cran.rstudio.com/')"
RUN R -e "install.packages('shinydashboard', repos='http://cran.rstudio.com/')"
RUN R -e "install.packages('tidyverse', repos='http://cran.rstudio.com/',dependencies=TRUE)"
RUN R -e "install.packages('shinythemes', repos='http://cran.rstudio.com/',dependencies=TRUE)"
```

## UI 함수 변경사항  

샤이니 테마 패키지의 설치가 끝났으니 이제 테마를 적용해볼 차례 입니다. UI 함수는 앱 화면의 구성요소를 변경해주는 역할을 합니다. UI 코드에서 `fluidpage()` 함수에 `theme = shinytheme("cerulean")` 인수를 추가 해 테마를 적용합니다. 다른 테마를 적용해 보실 분들은 [샤이니 테마 홈페이지](https://rstudio.github.io/shinythemes/ )에서 자신이 원하는 테마의 이름을 적어주시면 됩니다.

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

앱 부분에 변경 사항이 생겼으니 다시 도커 이미지를 빌드 해주어야 합니다. 이미지 빌드 방법은 [이전 글](https://choosunsick.github.io/post/dockerizing/ )을 참고해주세요 변경된 앱의 화면은 아래의 사진과 같습니다.

```
> docker build -t test-shiny-app  .
> docker run -p 3838:3838 test-shiny-app
```

![테마가 적용된 모습](https://user-images.githubusercontent.com/19144813/92995541-80bf8000-f53f-11ea-8852-f5a2dd146cc1.png)
