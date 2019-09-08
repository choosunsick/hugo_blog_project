---
title: "구글 Voice Kit 부팅 시 바로 실행하기"
date: 2019-09-08T17:20:30+09:00
draft: False
---

## 라즈베리파이 터미널 환경과 개인 노트북 터미널 환경 연결하기 

Google Voice Kit을 자동으로 실행시키는 작업에 앞서 라즈베리파이의 실행을 본인의 노트북에서 진행하기 위한 ssh로 연결 과정을 살펴보겠습니다. ssh로 라즈베리파이와 본인의 노트북을 연결할 경우 본인의 노트북의 터미널이나 작업 환경에서 라즈베리파이를 터미널 코드를 입력할 수 있게 됩니다. 아래의 코드는 라즈베리파이와 본인의 노트북을 ssh로 연결하는 코드입니다.

```
ping raspberrypi.local

#코드가 에러가 나지 않으면 ping 메세지가 계속 나오기 때문에 컨트롤+c로 중지해줍니다.

ssh pi@raspberrypi.local
```

먼저 자신의 노트북 터미널에서 위 코드를 실행하면 라즈베리파이의 비밀번호를 요구하는데 비밀번호를 입력하면 본인의 노트북 터미널 환경이 라즈베리파이의 터미널로 연결됩니다. 참고로 라즈베리파이의 초기 아이디는 pi이고, 비밀번호는 raspberry입니다. 이 부분은 모든 라즈베리파이가 똑같기 때문에 나중에 본인이 비밀번호를 따로 설정해야 합니다. [링크](https://withcoding.com/49)를 참조하면 라즈베리파이의 비밀번호를 변경하는 방법을 알 수 있습니다.  

## 라즈베리파이 시작시 구글 보이스 킷 자동으로 실행되게 만들기

작업환경을 자신의 노트북으로 변경했으면 이제 라즈베리파이 시작 시 구글 보이스 킷을 자동으로 실행하게 만들어 줄 차례입니다. 먼저 터미널에서 예제를 실행했던 폴더인 /home/pi/AIY-projects-python/src/examples/voice에 들어가 줍니다. 자동으로 실행될 예제 파일 하나를 선택하고 복사해서 main.py로 만들어 줍니다. 참고로 예제 파일은 모두 실행되는 것이므로 아무거나 해도 상관없습니다. 터미널에서 다음의 코드로 진행합니다.

```
#터미널 내에서 폴더 환경 변경
cd /home/pi/AIY-projects-python/src/examples/voice

# 예제 파일이 있는지 확인
ls

# 예제 파일을 main.py란 이름으로 복사 (cp는 복사 명령어)
cp assistant_library_demo.py main.py
```

이제 라즈베리파이가 부팅 시 자동으로 Google Voicekit의 음성인식 기능을 자동으로 실행하도록 하는 시스템 파일을 만들어주어야 합니다. 이 시스템 파일은 /etc/systemd/system/ 에서 만들어집니다. 여기서 만들 시스템 파일에는 부팅 시 시작되는 것, 모니터 연결 없이도 시작되는 것, 사용자 입력, 실행할 파일 등의 설정이 필요합니다. 먼저 아래의 코드로 시스템 파일을 만들어 줍니다. 참고로 시스템 파일 이름은 asssist가 아니어도 상관없습니다.

```
sudo nano /etc/systemd/system/assist.service
```

시스템 파일에 들어갈 내용을 코드로 작성합니다. 아래의 코드에서 Unit 단락의 after 명령어는 네트워크가 연결된 이후에 작동하라는 명령어입니다. service 부분의 Environment=DISPLAY=:0 코드는 화면 연결이 없이도 작동하게끔 만드는 코드입니다. SyslogIdentifier= 위에서 만든 본인의 시스템 파일 이름을 적어주어야 합니다.

```
[Unit]
Description=assist
After=network.target ntpdate.service

[Service]
Environment=DISPLAY=:0
ExecStart=/home/pi/AIY-projects-python/src/examples/voice/main.py
Restart=always or on-failure
User=pi
Group=pi
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=assist

[Install]
WantedBy=multi-user.target

```

위의 코드를 시스템 파일에 작성했다면 컨트롤 x와 y, enter를 눌러 파일을 저장해줍니다. 파일 저장 이후 라즈베리파이 터미널에서 아래의 코드를 입력해주고 리부팅하면 부팅 시 작동이 되는 모습을 확인할 수 있습니다.

```
sudo systemctl enable assist.service

sudo service assist start

sudo reboot -h now
```

리부팅 이후 보이스 킷이 작동하는데 까지 몇 분의 시간이 걸릴 수 있습니다. 자신의 보이스 킷이 작동 중인지 확인하고 싶은 경우 라즈베리파이 터미널에서 `sudo service assist status` 코드로 확인할 수 있습니다. 이때 화면에 나오는 Active: 표시에 active(running) 표시가 나오면 작동되고 있다는 말입니다. 만약 아직 작동 중이 아니라면 failed 표시가 나오게 됩니다. 이상으로 구글 보이스 킷 자동으로 실행하는 방법에 대해 소개를 마칩니다.   
