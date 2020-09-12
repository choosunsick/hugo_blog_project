---
title: "샤이니 앱에 항목 탭 추가하기"
date: 2020-09-12T19:01:49+09:00
draft: False
tags: ["R", "탭", "내비게이터", "샤이니","shiny"]
categories: ["R 샤이니"]
---

 이전 글에서는 샤이니 앱에 테마를 적용해 보았습니다. 샤이니 앱에 테마를 적용하는 방법은 [이전 글](https://choosunsick.github.io/post/shiny_apply_theme/ )을 참고해주세요! 이번 글에서는 샤이니 앱에 항목 탭을 추가해 볼 것입니다. 샤이니 테마를 적용했던것 처럼 항목 탭을 추가하는 작업 역시 매우 간단합니다. 이번에는 Dockerfile에 변화 없이 오직 app 파일만 변경하면 됩니다.
 항목 탭은 한 가지가 아닌 여러가지를 보여주기 위해 사용합니다. 항목 탭을 이용하면 클릭을 통해 새로운 페이지로 넘어가는 것과 같이 클릭을 통해 새로운 내용을 보여줄 수 있습니다. 항목 탭을 추가하기 위해서는 UI의 page 유형이 달라지고, 항목을 만들어주는 탭 판넬을 추가하는 부분이 생기게 됩니다. 페이지 유형에는 동적페이지(fluidpage), 고정형 페이지(fixedpage), 대쉬보드형 페이지(dashboardpage), 내비게이터 형 페이지(navbarpage) 등이 있습니다.

## UI 변경사항

기존의 `fluidPage()` 부분을 `navbarPage()`로 변경해 줍니다. 추가로 기존 코드에서와 달리 titlePanel이 사라진 대신 `navbarPage()` 함수에 바로 "mtcars Data" 라는 제목이 붙게 되었습니다. 이제 기존의 히스토그램을 특정 항목 안으로 집어 넣는 것을 해볼 차례 입니다.
특정 항목을 만드는 방법은 `tabPanel()` 함수로 이루어집니다. 코드를 `tabPanel()` 함수 안에 넣어주기만 하면 항목이 생성되고 거기에 속하게 됩니다.  `tabPanel()` 함수 안에 "자동차 마력 히스토그램" 부분은 해당 항목의 이름을 표기합니다.
이제 새로운 항목을 추가 해봅시다. 새로운 항목을 만드는 방법도 어렵지 않습니다. 기존의 코드를 복사 한 후 항목이름과 아웃풋 플롯의 이름만 바꾸어 주면 됩니다. 이 항목에서는 mtcar 데이터에 있는 자동차 배기량 데이터를 가지고 히스토그램을 그려보겠습니다. 다른 아웃풋이 생기게 되었으니 server 부분도 변경이 되야 합니다. UI 부분의 변경 사항은 아래의 코드와 같습니다.

```R
ui <- navbarPage("mtcars Data",theme = shinytheme("cerulean"),
                 tabPanel("자동차 마력 히스토그램",
                          sidebarLayout(
                              sidebarPanel(
                                  sliderInput("bins",
                                              "Number of bins:",
                                              min = 1,
                                              max = 30,
                                              value = 15)
                              ),
                              mainPanel(
                                  plotOutput("Hp_Plot")
                              )
                          )
                 ),
                 tabPanel("자동차 배기량 히스토그램",
                 sidebarLayout(
                     sidebarPanel(
                         sliderInput("bins",
                                     "Number of bins:",
                                     min = 1,
                                     max = 30,
                                     value = 15)
                     ),
                     mainPanel(
                         plotOutput("disp_plot")
                     )
                 )
                )
)
```

## server 변경 사항

기존 서버의 내용에 추가되는 부분은 자동차 배기량 히스토그램을 그려주는 부분이 추가 됩니다. 기존 코드를 그대로 복사한 후 output$disp_plot 이라는 이름으로 추가해 줍니다. 히스토그램 데이터에 해당하는 부분인 x를 `x <-  mtcars$disp` 같이 변경해 주면 새로운 히스토그램 플롯이 그려집니다.

```R
server <- function(input, output) {
    output$Hp_Plot <- renderPlot({
        # generate bins based on input$bins from ui.R
        x <- mtcars$hp
        bins <- seq(min(x), max(x), length.out = input$bins + 1)
        # draw the histogram with the specified number of bins
        hist(x, breaks = bins, col = 'darkgray', border = 'white')
    })
    output$disp_plot <- renderPlot({
        x <- mtcars$disp
        bins <- seq(min(x), max(x), length.out = input$bins + 1)
        hist(x, breaks = bins, col = 'darkgray', border = 'white')
    })
}
```

## 웹에서 확인하기

새로운 내용의 app을 이미지로 빌드하고 웹에서 확인해볼 차례입니다. 이전과 같은 방식으로 이미지를 빌드하고 http://localhost:3838/ 에서 확인해봅니다. 이미지가 만들어지고 앱이 실행되었다면 웹에서 다음과 같은 모습을 확인할 수 있습니다.

![자동차 마력 히스토그램]("https://user-images.githubusercontent.com/19144813/92995430-a13b0a80-f53e-11ea-8233-3e8bb790b266.png")

![자동차 배기량 히스토그램](https://user-images.githubusercontent.com/19144813/92992887-0edc3c00-f529-11ea-9362-586b2d9e2917.png)
