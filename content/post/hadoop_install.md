---
title: "라즈베리파이 클러스터에 하둡 설치하기"
date: 2020-12-07T18:46:36+09:00
draft: false
tags: ["Hadoop","raspberrypi"]
categories: ["raspberrypi"]
---

## 라즈베리파이 os 설치하기

raspberrypi imager를 통해 라즈베리파이의 os를 깔아줍니다. 해당 프로그램은 [링크](https://www.raspberrypi.org/software/)에서 자신의 컴퓨터 os에 맞게 설치할 수 있습니다. 설치 후 sd카드를 꽂으면 이미저에서 sd 카드를 인식하고 아래 사진처럼 옵션을 선택하여 라즈베리파이 os를 설치할 수 있습니다.

![raspberrypi imager](https://user-images.githubusercontent.com/19144813/101337219-bf56a380-38be-11eb-9746-05cfc3e540a8.png)

설치가 종료된후 sd 카드가 자동으로 언마운트 되므로 다시 삽입 해주신 후 터미널이나 cmd를 열고 아래와 같이 치고 sd카드를 뺀 후 라즈베리파이에 장착해 줍니다. 같은 과정을 자신의 라즈베리파이 수 만큼 반복해줍니다.

```bash
cd ~/Volumes/boot
touch ssh
cd
```

## 라즈베리파이 간 서로 연동하기

먼저 아래 명령어로 라즈베리파이에 ssh로 접속해줍니다.

```bash
ssh-keygen -R raspberrypi.local
ssh pi@raspberrypi.local
```

이후 아래 코드로 각 라즈베리파이의 업데이트와 업그레이드를 해줍니다.

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

아래의 명령어로 설정에 들어가고 시스템 옵션 Hostname 칸을 눌러 사용자 이름을 piX 로 변경해줍니다. 여기서 X는 숫자로 마스터가 될 라즈베리파이가 1이 되고 이후 워커가 될 파이들을 pi2, pi3 ... 같이 설정해줍니다.

```bash
sudo raspi-config
```

각 파이의 업데이트와 업그레이드가 끝나면 자신의 모든 라즈베리파이에 java와 에디터 vim을 설치해줍니다. java 설치에는 시간이 좀 소요됩니다.

```bash
sudo apt-get install vim -y

sudo apt install openjdk-8-jdk openjdk-8-jre -y
```

이제 모든 라즈베리파이에서 hosts 파일을 변경해줍니다. hosts의 내용은 각 파이의 명칭과 ip를 적어줍니다. ip는 각 파이에서 ifconfig로 확인할 수 있습니다.

```bash
sudo vim /etc/hosts
```

```bash
#hosts 파일의 내용
192.168.0.17 pi1
192.168.0.15 pi2
192.168.0.14 pi3
```

```bash
cd ~/
mkdir .ssh
```

이제 마스터로 사용할 파이에서 설정을 만들어줍니다. 설정파일에는 아래와 같이 각 파이의 명칭과 ip를 적은 정보를 입력합니다.

```bash
cd ~/.ssh
touch config

vim config
```

```bash
#config 파일 내용
Host pi1
User pi
Hostname 192.168.0.17

Host pi2
User pi
Hostname 192.168.0.15

Host pi3
User pi
Hostname 192.168.0.14
```

이제 모든 파이에서 키값을 만들어줍니다.

```bash
ssh-keygen –t ed25519
```

마스터를 제외한 라즈베리파이에서 아래의 명령어를 쳐 마스터의 권한 부여합니다.

```bash
cat ~/.ssh/id_ed25519.pub | ssh pi@pi1.local 'cat >> .ssh/authorized_keys'
```

다시 마스터 라즈베리파이로 돌아와 마스터의 키에 권한을 부여합니다.

```bash
cd
cat .ssh/id_ed25519.pub >> .ssh/authorized_keys
```

모든 파이 수 만큼 아래의 명령어를 입력합니다. 여기서는 총 2개의 워커를 사용하므로 pi2, pi3에 대해 입력합니다.

```bash
scp ~/.ssh/authorized_keys pi2.local:~/.ssh/authorized_keys
scp ~/.ssh/authorized_keys pi3.local:~/.ssh/authorized_keys
scp ~/.ssh/config pi2.local:~/.ssh/config
scp ~/.ssh/config pi3.local:~/.ssh/config
```

## 마스터에서 다른 파이 조작하기

마스터에서 워커 파이를 조작할 수 있게 설정파일에 함수를 입력해줍니다.

```bash
cd ~ && vim ~/.bashrc
```

```bash
#bashrc 파일 내용

function otherpis {
  grep "pi" /etc/hosts | awk '{print $2}' | grep -v $(hostname)
}

function clustercmd {
  for pi in $(otherpis); do ssh $pi "$@"; done
  $@
}

function clusterreboot {
  clustercmd sudo shutdown -r now
}

function clustershutdown {
  clustercmd sudo shutdown now
}

function clusterscp {
  for pi in $(otherpis); do
    cat $1 | ssh $pi "sudo tee $1" > /dev/null 2>&1
  done
}
```

변경된 설정내용을 다른 모든 파이에 적용해줍니다. 마스터 라즈베리파이에서 아래의 명령어로 입력하면 다른 모든 파이에 변경된 설정이 적용됩니다.

```bash
source ~/.bashrc
clusterscp ~/.bashrc
clustercmd source ~/.bashrc
```

## 하둡 설치

마스터 라즈베리파이에 하둡을 설치해 줍니다. 버전은 3.2.1 버전을 사용해 줍니다. 3.3.0의 경우 java나 차후에 설치할 spark와 연동을 위해 3.2.1버전을 사용합니다.

```bash
wget https://downloads.apache.org/hadoop/common/hadoop-3.2.1/hadoop-3.2.1.tar.gz

sudo tar -xvf hadoop-3.2.1.tar.gz -C /opt/
rm hadoop-3.2.1.tar.gz && cd /opt
sudo mv hadoop-3.2.1 hadoop

sudo chown pi:pi -R /opt/hadoop
```

하둡 관련 폴더 설정 경로를 설정파일에 지정해줍니다.

```bash
vim ~/.bashrc
```

```bash
# bashrc 파일을 열고 맨 처음에 보면 for examples이 보인다  그 밑에 아래의 내용을 입력한다.

export JAVA_HOME=$(readlink –f /usr/bin/java | sed "s:bin/java::")
export HADOOP_HOME=/opt/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_INSTALL=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
```

아래 명령어로 적용

```bash
source ~/.bashrc
```

하둡 환경설정에도 자바 경로를 입력해준다.

```bash
vim /opt/hadoop/etc/hadoop/hadoop-env.sh
```

```bash
# hadoop-env의 내용
export JAVA_HOME=$(readlink –f /usr/bin/java | sed "s:bin/java::")
```

아래 명령어로 마스터 라즈베리파이에 하둡이 잘 설치된지 확인할 수 있다.

```bash
hadoop version | grep Hadoop
```

## 하둡 싱글 노드 설정 적용하기

XML 파일이 있는 위치로 디렉토리를 변경해줍니다.

```bash
cd /opt/hadoop/etc/hadoop
```

먼저 core-site.xml, hdfs-site.xml 2개의 파일을 변경해줍니다. 파일 내용을 입력할 때는 `<configuration>` 아래에 입력해야한다.

```bash
vim core-site.xml
```

```bash
#core-site.xml 내용
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://pi1:9000</value>
</property>

```

```bash
vim hdfs-site.xml
```

```bash
#hdfs-site.xml 내용
<property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///opt/hadoop_tmp/hdfs/datanode</value>
</property>
<property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///opt/hadoop_tmp/hdfs/namenode</value>
</property>
<property>
    <name>dfs.replication</name>
    <value>1</value>
</property>
```

데이터노드와 네임노드를 저장할 공간을 만들어 준다.

```bash
sudo mkdir -p /opt/hadoop_tmp/hdfs/datanode
sudo mkdir -p /opt/hadoop_tmp/hdfs/namenode

sudo chown pi:pi -R /opt/hadoop_tmp
```

yarn 설정과 관련된 다른 두 개 설정 파일도 변경해준다.

```bash
vim mapred-site.xml
```

```bash
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
```

```bash
vim yarn-site.xml
```

```bash
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<property>
    <name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>  
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
```

아래의 명령어로 마스터 노드에 하둡 시스템이 잘 작동하는지 확인할 수 있습니다.

```bash
hdfs namenode -format

start-dfs.sh
start-yarn.sh

jps
```