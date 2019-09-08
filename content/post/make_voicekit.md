---
title: "구글 보이스 킷을 조립해보자 !"
date: 2019-09-08T16:30:30+09:00
draft: False
---

이번에 시간이 없어서 그동안 조립하지 못했던 Google AIY Voice Kit을 조립해 보았습니다. Google AIY Voice Kit은 음성인식 AI입니다. 현재 AI 스피커는 구글 홈, 네이버의 클로바, 카카오미니 등 여러 가지 제품이 있습니다. 이 중 구글 홈과 같은 음성인식 AI 스피커의 음성인식 기술의 원류를 Google AIY Voice Kit에서 찾아 볼 수 있습니다. 현재 Google AIY Voice Kit은 최신 버전 Google AIY Voice Kit 2가 나와 있는 상태입니다. 아쉽게도 이번에 제가 조립할 Google AIY Voice Kit은 최신 버전이 아닌 초창기 버전의 것입니다. 조립하기에 앞서 요즘에 나오는 Voice Kit 제품에는 작은 드라이버와 충전기를 빼고는 필요한 것들이 다 키트 내에 포함되어 있지만, 구 버전의 것에는 ![라즈베리파이](https://user-images.githubusercontent.com/19144813/64485106-0a66ae80-d257-11e9-9996-2f007490b235.jpeg)를 별도로 구매해야 했습니다. 나아가 ![라즈베리파이 충전기](https://user-images.githubusercontent.com/19144813/64485141-85c86000-d257-11e9-8618-84cd81de595f.jpeg)는 역시 별도 구매해야하지만 휴대폰 충전기로도 대체할 수 있습니다.

## 조립 과정

먼저 제품 상자와 내용물을 살펴보겠습니다.

![보이스 킷 상자](https://user-images.githubusercontent.com/19144813/64485200-6251e500-d258-11e9-90ab-3d3e0d3aa244.jpeg)

![조립 부품들](https://user-images.githubusercontent.com/19144813/64485213-962d0a80-d258-11e9-804c-d9ba2cb0d279.jpeg)

사진에서는 많은 부품이 있는 것처럼 보이지만 실제로는 그렇게 많은 부품이 있지 않기 때문에 조립하는 것은 어렵지 않습니다. 키트 상자 안에 그림으로 설명된 설명서가 함께 동봉되어 있으며, [유튜브](https://www.youtube.com/watch?v=HER_885yVDM)에서 자세한 조립 동영상도 찾아볼 수 있습니다. 그 외 [구글 AIY 프로젝트의 공식 사이트](https://aiyprojects.withgoogle.com/voice/)에서도 자세한 조립 방법과 순서를 살펴보실 수 있습니다. 조립을 시작하기 전에 빠진 부품 없이 모든 부품이 다 있는지 확인하고 조립을 시작해봅시다.  

음성 인식 보드를 설명서를 보고 조립해줍니다.
![음성 인식 보드](https://user-images.githubusercontent.com/19144813/64485374-ffae1880-d25a-11e9-90b6-571f9f27c9d5.jpeg)

음성 인식 보드에 라즈베리파이를 장착합니다.
![라즈베리파이 장착](https://user-images.githubusercontent.com/19144813/64485385-28361280-d25b-11e9-8bea-55f8cb82f2dc.jpeg)

보드에 스피커에 연결할 커넥터를 장착합니다.
![커넥터 장착](https://user-images.githubusercontent.com/19144813/64485389-3c7a0f80-d25b-11e9-8262-193ac9a0a993.jpeg)

보드를 보호할 상자를 조립합니다.
![상자 조립](https://user-images.githubusercontent.com/19144813/64485395-5ddafb80-d25b-11e9-8552-6263c29cdcea.jpeg)

상자 안에 보드를 넣어줍니다.
![상자와 보드 스피커 합체](https://user-images.githubusercontent.com/19144813/64485406-78ad7000-d25b-11e9-9a31-c4f31f96774c.jpeg)

상자에 버튼을 붙여줍니다.
![버튼 장착](https://user-images.githubusercontent.com/19144813/64485496-831c3980-d25c-11e9-8c94-22cbe839d29b.jpeg)

조립 완성
![완성](https://user-images.githubusercontent.com/19144813/64485504-98916380-d25c-11e9-80a1-ca561f141114.jpeg)

보이스 킷의 조립이 끝났다면, 제품에 키보드, 마우스, 모니터(HDMI 단자)를 연결해주고 전원선을 꽂아 라즈베리파이를 부팅해서 작동시켜줍니다. 보이스 킷이 작동되면 라즈베리파이의 기본설정을 하게 되는데 이 때 국적과 시간대를 한국과 서울로 시작하면 와이파이를 인식하지 못 하기 때문에 일단 영국과 런던으로 맞추어 줍니다. 차후에 와이파이가 연결된 이후 라즈베리파이의 설정을 통해 국적과 시간대를 재조정할 수 있으니 걱정말고 영국으로 설정하셔도 됩니다.

## 보이스 킷 음성 테스트 하기

이제 보이스 킷의 조립이 제대로 되었는지 기능을 시험해 볼 차례입니다. 가장 먼저 마이크와 스피커의 기능을 검사하는 check audio를 눌러 줍니다. 이 때 터미널의 메세지에 playing a test sound가 나오면 오디오 테스트 시작됩니다.

[test 정상작동](https://user-images.githubusercontent.com/19144813/64485512-bb237c80-d25c-11e9-92a6-ef694d7e9f4e.png)

만약 위의 사진과 같은 화면이 아니라 사운드 카드가 설치되지 않았다는 터미널에 에러 메세지가 나올 경우 아래의 코드를 통해 문제를 해결할 수 있습니다.

```
# config.txt를 수정
sudo pico /boot/config.txt

# config.txt 맨 아래에 코드 추가 이후 컨트롤 x와 y를 눌러서 config.txt 변경 사항을 저장
dtoverlay=googlevoicehat-soundcard

# 리부트
sudo reboot -h now
```

새로운 터미널에서 위의 코드 입력을 마치면 재부팅이 됩니다. 재부팅 이후 다시 바탕화면이 보이면 check audio를 눌러서 다시 한번 마이크와 스피커를 테스트해 줍니다. 위의 사진과 같이 오디오 테스트가 정상적으로 작동하면 터미널에서 출력되는 요청에 따라서 테스트를 진행하게 됩니다.

## 라즈베리파이 설정하기

오디오 기능에 문제가 없다면, 라즈베리파이의 설정을 통해 지역과 시간대를 알맞게 변경할 차례입니다. 설정 변경에 앞서 라즈베리파이의 모듈들을 업데이트와 업그레이드를 진행해 줍니다. 업데이트와 업그레이드 과정은 생각보다 오랜 시간이 걸립니다. 업데이트와 업그레이드가 끝난다면 이어서 한글 폰트를 설치해줍니다. 아래는 그 방법입니다.

```

sudo apt-get update
sudo apt-get upgrade

# 한글 폰트 설치 코드

sudo apt-get install ibus ibus-hangul fonts-unfonts-core
```
한글 폰트가 설치되면 다시 원래의 국적과 시간대로 돌아올 차례입니다. 라즈베리파이 설정 변경 방법은 다음과 같습니다. 바탕화면의 라즈베리파이 모양을 누르고 preferences의 Raspberry Pi Configuration의 Localization을 선택해 줍니다. 이제 Set Locale과 Set Timezone 등의 설정을 한국으로 변경해줍니다. 변경된 설정을 적용하기 위해 리부트를 눌러줍니다. 재부팅이 되면 본래 한국과 서울의 시간대로 돌아온 것을 확인하실 수 있습니다.

## 구글 클라우드 API 설정하기

설정을 변경했다면, 이제 보이스 킷 실행까지 한 단계만 남았습니다. 남은 것은 구글 클라우드 API 설정 및 적용입니다. 구글 클라우드의 API 설정은 굳이 라즈베리파이 내에서 진행할 필요가 없습니다. 본인의 pc나 노트북에서 [링크](https://console.cloud.google.com/)로 가서 구글에 로그인합니다. 로그인 되면 프로젝트 선택 후 새로운 프로젝트(NEW PROJECT)를 클릭해 줍니다. 새로운 프로젝트의 이름을 설정해 만들었다면 이제 해당 프로젝트를 클릭하여 선택해 줍니다.  이 프로젝트에서 google assistant API를 설정해 줍니다. API 및 서비스 사용자 설정에 들어가서 "google assistant"를 검색해줍니다. 검색 결과로 google assistant API를 클릭하고 사용설정을 눌러줍니다.
이제 API 사용을 위한 사용자 인증 정보를 만들어야 합니다. 사용자 인증 정보 만들기를 눌르면 3 가지 목록이 나오는데 이 중 0Auth 클라이언트 ID를 눌러줍니다. 클라이언트 ID를 만들기 위해 동의 화면 구성을 눌러 제품 이름과 이메일 등의 정보를 입력 후 저장한 다음 애플리케이션 유형에 기타를 누르고 이름을 입력해 ID를 생성해줍니다. 생성한 클라이언트 ID를 옆에 다운로드 버튼을 눌러줍니다. 클라이언트 ID가 json 형식으로 다운로드되면 이 파일을 본인의 이메일로 보내줍니다. 다시 라즈베리파이로 이동해서 클라이언트 ID가 담긴 json 파일을 내려받아 줍니다. 내려받은 json 파일의 이름을 assistant.json으로 변경한 뒤 라즈베리파이의 home/pi에 넣어줍니다.

## google voice kit 실제로 작동하기

구글 보이스 킷 사용을 위한 기본 설정은 이제 모두 끝났습니다. 이제 보이스 킷이 실제 작동하는 것을 확인해볼 차례입니다. 보이스 킷은 예제 파일을 통해 실행할 수 있습니다. 터미널을 열고 아래의 코드를 통해 예제 파일을 실행할 수 있습니다. 여러 예제(demo) 파일 중 하나를 선택해서 보이스 킷의 기능을 작동시킬 수 있습니다.

```
cd /home/pi/AIY-projects-python/src/examples/voice

python3 assistant_library_demo.py
```

위의 코드를 실행하면 웹 페이지에 AIY projects의 구글 엑세스 관련 요청이 나옵니다. 여기서 허용을 눌러주면 이제 인공지능 스피커를 테스트해 볼 수 있습니다. OK google이라는 시동 어를 말하고 궁금한 영어 문장들을 말하면 발음이 정확할 경우 그에 대해 대답을 해줍니다.

이상으로 보이스 킷 조립기를 마칩니다. 이어서 다음 글에서는 구글 보이스 킷을 라즈베리파이 부팅 시 자동으로 실행하는 방식에 대해 소개하겠습니다.
