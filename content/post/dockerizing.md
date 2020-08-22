---
title: "R 샤이니 앱 도커라이징(Dockerizing) 하기"
date: 2020-08-22T14:03:50+09:00
draft: False
tags: ["R", "도커", "샤이니","shiny"]
categories: ["R 샤이니"]
---

## R 샤이니란 ?

 샤이니는 R과 직접적으로 상호작용할 수 있는 앱을 만들어주는 패키지입니다. 예를 들면 데이터 분석 결과에 대해 앱의 사용자가 웹과 상호작용하여 여러가지 분석 결과를 확인할 수 있습니다. 샤이니의 사용방법 및 자세한 설명은 [R shiny 공식 사이트](https://shiny.rstudio.com/)를 참조해주세요.

## 왜 샤이니 앱을 Dockerizing 하는가 ?

 샤이니 앱을 만드는 방법에는 javascript를 이용하거나 HTML을 이용하는 등 다양한 방식이 있습니다. javascript와 HTML을 이용하여 웹을 만들고 웹에 올리기 위해서는 javascript와 HTML을 잘 다루어야 한다는 조건이 있습니다. 그렇기 때문에 만약 javascript와 HTML을 모른다면 자신이 만든 샤이니 앱을 웹에 올리는 과정은 쉬운 길이 아닐 것입니다. 그러나 도커를 이용한다면 달라집니다. 도커를 이용할 경우 해당 언어들을 몰라도 샤이니 앱을 쉽게 만들고 웹에 올릴 수 있습니다. 도커를 이용하면 누구나 앱을 쉽게 재현할 수 있다는 장점도 있습니다. 도커의 컨테이너에 앱 사용과 구현에 필요한 모든 조건들이 종속되어 있기 때문입니다.

## 샤이니 앱을 Dockerizing 하는 방법

 샤이니 앱을 도커라이징 하기위해 필요한 준비물을 알아봅시다. 먼저 샤이니 앱을 만들 수 있는 R studio가 필요합니다. 다음 준비물은 도커(Docker)입니다. R studio의 설치는 해당 [링크](https://rstudio.com/products/rstudio/download/)에서 무료로 다운 받을 수 있습니다.

### 도커의 설치와 r-shiny 도커 실행시키기
  다음으로 도커를 설치해야합니다. [도커 홈페이지](https://docs.docker.com/engine/install/)에서 자신의 os 체제에 맞는 것을 설치하세요. 도커가 잘 설치되었는지 확인하려면 터미널 혹은 cmd에서 `docker run hello-world`를 실행하면 아래와 같은 문구가 출력되는 것을 볼 수 있습니다. 만약 그렇지 않다면 도커가 제대로 설치되지 않은 것입니다.

```

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

 이제 R과 샤이니, tidyverse가 설치된 도커 컨테이너를 가져온다음 실행시켜 보겠습니다. 터미널 혹은 CMD에서 `docker pull rocker/shiny-verse` 명령어로 도커 컨테이너를 가져와 설치합니다. 이 컨테이너가 설치된 이후 `docker run --rm -p 3838:3838 rocker/shiny-verse` 통해 설치된 컨테이너를 실행시킵니다. 해당 도커 컨테이너 실행 및 설치와 관련해서 더 자세한 내용은 [rocker-shiny-verse 소개 페이지](https://github.com/rocker-org/shiny)에서 확인할 수 있습니다.

```
docker pull rocker/shiny-verse

docker run --rm -p 3838:3838 rocker/shiny-verse
```

 이 도커 컨테이너가 실행 되면 다음 http://localhost:3838 을 통해 확인할 수 있습니다. 컨테이너가 실행된 것이 확인되면 터미널이나 CMD에서 control c를 통해 샤이니 앱 실행을 종료할 수 있습니다. 이후 도커라이징 과정을 위해서 컨테이너가 실행된 것이 확인되면 control c를 통해 종료해주시기 바랍니다.

### 샤이니 앱 만들기

 Rstudio를 설치하고 실행하고나면 Rstudio의 상단 메뉴 중 file에서 new project를 눌러 새로운 프로젝트를 만들 수 있습니다. 이때 new directory를 누르면 여러가지 카테고리가 나오는데 여기서 필요한 것은 shiny web application 입니다. 이것을 누르면 샤이니 웹 앱과 관련된 새 프로젝트가 생성되면서 기본적인 app.R 파일이 생성됩니다.

 기본 앱을 도커라이징해도 되지만 데이터를 약간 바꿔서 올려봅시다. 먼저 기존에 Faithfull 데이터 대신 mtcars 데이터를 사용해 봅시다. mtcars 데이터는 R의 기본 데이터로 1974년 US magazine에 Motor Trend에 대한 데이터입니다. 이제 mtcars 자료의 마력(horse power) 데이터를 히스토그램으로 그려봅시다.

 기본 앱에서 바뀌는 부분은 많지 않습니다. 바뀌는 부분은 크게 2가지로 ui와 server 부분입니다. 먼저 ui 부분에서는 titlePanel 안에 이름 부분을 "mtcars Hp Data"로 변경해 줍니다. 그 다음 히스토그램의 최소 최대 빈도 수를 변경해 줍니다. 최소 최대 빈도 수를 변경하는 이유는 mtcars의 데이터 수가 32개이기 때문에 그 이상의 빈도 수로 히스토그램을 나타내는 것은 의미가 없기 때문입니다. ui의 sliderInput안의 min 값을 1로 max 값을 30으로 변경해 줍니다.
 또한, 현재 value 값도 30에서 1과 30의 중간인 15로 변경해줍니다. 이렇게 변경하면 ui 코드에서 변경할 내용은 끝입니다. 다음으로 server 코드에서 변경할 내용은 더욱 간단합니다. 서버 함수에서 데이터가 들어가는 부분인 x 변수를 정의하는 코드 줄에서 `faithful[,2]` 를 `mtcars$hp`로 변경하면 끝입니다.

### Dockerizing 하기

이제 마지막으로 도커 컨테이너를 만들고 실행시키는 일만 남았습니다. 도커 컨테이너를 만드는데 중요한 것은 Dockerfile과 스크립트 파일을 만드는 것입니다. Dockerfile은 앱을 실행시키는데 필요한 패키지나 데이터 등의 조건들을 가져옵니다. Dockerfile에 대한 자세한 내용은 [Dockerfile best practice](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)에서 확인할 수 있습니다. 아래는 Dockerfile의 기본적인 형식입니다.

```
FROM rocker/shiny-verse:latest

# system libraries of general use
RUN apt-get update && apt-get install -y \
    sudo \
    pandoc \
    pandoc-citeproc \
    libcurl4-gnutls-dev \
    libcairo2-dev \
    libxt-dev \
    libssl-dev \
    libssh2-1-dev

# install R packages required
# (change it dependeing on the packages you need)
RUN R -e "install.packages('shiny', repos='http://cran.rstudio.com/')"
RUN R -e "install.packages('shinydashboard', repos='http://cran.rstudio.com/')"
RUN R -e "install.packages('tidyverse', repos='http://cran.rstudio.com/')"

# copy the app to the image
COPY shiny-server.sh /usr/bin/shiny-server.sh
COPY app.R /srv/shiny-server

# select port
EXPOSE 3838

# allow permission
RUN ["chmod", "+x", "/usr/bin/shiny-server.sh"]


# run app
CMD ["/usr/bin/shiny-server.sh"]

```

맨 첫 줄의 FROM은 Docker가 사용할 기본 이미지를 알려주는 역할을 합니다. 샤이니 앱을 도커 컨테이너로 만드는 경우 기본 이미지 위에서 그것을 바탕으로 앱의 기능을 추가하여 작동하게 됩니다. 이 도커 이미지에 대한 자세한 내용은 [rocker-shiny-verse 소개 페이지](https://hub.docker.com/r/rocker/shiny-verse)에서 확인하실 수 있습니다.

`RUN apt-get update && apt-get install -y \`는 샤이니 서버를 실행하는데 필요한 Linux 유틸리티들을 설치해줍니다.

`RUN R -e "install.packages(~)"` 는 만들어질 도커 컨테이너에 필요한 모든 R 패키지를 설치하게 해줍니다. 자신의 샤이니 앱에서 필요한 패키지들을 `RUN R -e "install.packages('패키지이름',repos='http://cran.rstudio.com/')"` 의 형식으로 추가해주면 됩니다. 참고로 한 줄로 여러 개의 패키지를 설치하고 싶다면 패키지 이름 부분에 pkgs=c() 인수와 설치하려는 패키지들의 이름을 추가하면 됩니다.

`COPY shiny-server.sh /usr/bin/shiny-server.sh`, `COPY app.R /srv/shiny-server` 는 도커 이미지에 샤이니 서버 구동에 필요한 스크립트와 샤이니 앱을 복사해주는 역할을 합니다. 차후 샤이니 앱을 구동하는데 필요한 외부 csv 데이터나 보조 스크립트 등이 있다면 이 줄에 마찬가지로 복사시켜줘야 합니다.

`EXPOSE 3838` 은 도커 이미지를 실행시켜 포트에 연결하는 것입니다.

`RUN ["chmod", "+x", "/usr/bin/shiny-server.sh"]` 는 도커 이미지 컨테이너에서 chmod 명령어를 통해 사용 권한을 주어 파일을 실행할 수 있도록 해줍니다.

마지막으로 `CMD ["/usr/bin/shiny-server.sh"]` 는 샤이니가 서버를 실행하고 우리가 만든 앱 파일을 실행하도록 해줍니다. 이렇게 만든 Dockerfile은 우리가 만든 app.R 파일과 같은 위치에 저장해 줍니다.

이제 스크립트 파일을 만들 차례입니다. 스크립트에서는 샤이니 서버의 로그기록을 저장할 곳을 만들어 주는 역할을 합니다. 스크립트의 내용은 아래와 같습니다. 아래 내용을 빈 파일에 만들고 shiny-server.sh 로 Dockerfile과 같은 위치에 저장해 줍니다.

```
#!/bin/sh

# Make sure the directory for individual app logs exists
mkdir -p /var/log/shiny-server
chown shiny.shiny /var/log/shiny-server

exec shiny-server >> /var/log/shiny-server.log 2>&1

```
## 앱 만들어 실행하기

마지막으로 만들어진 앱을 실행할 차례입니다. 먼저 터미널이나 CMD를 열어서 Dockerfile이 포함된 폴더로 이동해줍니다. `docker build -t <NAME> <DOCKERFILE_PATH>` 는 도커 이미지를 생성해줍니다. 참고로 `<NAME>`은 도커 이미지 파일의 이름을 의미하며, 이후 이미지를 실행시킬 때 사용됩니다. `<DOCKERFILE_PATH>` 는 Dockerfile의 파일 경로입니다. 참고로 이미 폴더에 있는 경우에는 `.` 을 입력하면 됩니다. 이제 직접 `docker build -t test-shiny-app  .`와 같이 입력하여 도커 이미지를 생성합니다. 이미지가 생성되는 도중 에러가 발생하지 않고 무사히 생성되면 맨 마지막에 아래의 두 줄이 출력됩니다.

```
> docker build -t test-shiny-app  .

Sending build context to Docker daemon  2.814MB
Step 1/12 : FROM rocker/shiny-verse:latest
 ---> 8be2555ecf1d
Step 2/12 : RUN apt-get update && apt-get install -y     sudo     pandoc     pandoc-citeproc     libcurl4-gnutls-dev     libcairo2-dev     libxt-dev     libssl-dev     libssh2-1-dev
 ---> Using cache
 ---> 94a312813121
Step 3/12 : RUN R -e "install.packages('shiny', repos='http://cran.rstudio.com/')"
 ---> Using cache
 ---> 7941ffbd8a88
Step 4/12 : RUN R -e "install.packages('shinydashboard', repos='http://cran.rstudio.com/')"
 ---> Using cache
 ---> ae0e7734fc03
Step 5/12 : RUN R -e "install.packages('plotly', repos='http://cran.rstudio.com/')"
 ---> Using cache
 ---> 5393c96c0124
Step 6/12 : RUN R -e "install.packages('tidyverse', repos='http://cran.rstudio.com/')"
 ---> Using cache
 ---> 5e2cd4405c54
Step 7/12 : RUN R -e "install.packages('lubridate', repos='http://cran.rstudio.com/')"
 ---> Using cache
 ---> 8dce593b0aa9
Step 8/12 : COPY shiny-server.sh /usr/bin/shiny-server.sh
 ---> Using cache
 ---> 36d349eab349
Step 9/12 : COPY app.R /srv/shiny-server
 ---> Using cache
 ---> b8f3863507d8
Step 10/12 : EXPOSE 3838
 ---> Using cache
 ---> 70c2b53ebde2
Step 11/12 : RUN ["chmod", "+x", "/usr/bin/shiny-server.sh"]
 ---> Using cache
 ---> 09589452e211
Step 12/12 : CMD ["/usr/bin/shiny-server.sh"]
 ---> Using cache
 ---> b5dc252f66fa
Successfully built b5dc252f66fa
Successfully tagged test-shiny-app:latest

```

이제 이미지가 생성되었으니 터미널이나 CMD에서 아래의 명령어로 실행하면 지정한 포트번호에서 앱이 실행되는 것을 확인할 수 있습니다.

```
> docker run -p 3838:3838 test-shiny-app
```

실행된 앱을 웹에서 확인하고 싶으시면 인터넷 창에서 http://localhost:3838/ 을 입력하면 됩니다.
