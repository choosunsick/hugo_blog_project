---
title: "라즈베리파이 클러스터에 하둡 설치하기 2"
date: 2020-12-07T20:14:56+09:00
draft: false
tags: ["Hadoop","raspberrypi"]
categories: ["raspberrypi"]
---

## 하둡 분산 시스템 설정하기

이 글은 앞선 [라즈베리 파이에 하둡 설치하기 글](https://choosunsick.github.io/post/hadoop_install/)에 이어지는 글입니다. 먼저 글을 보고 와주시면 감사하겠습니다.
다른 모든 파이에도 하둡을 설치하기 위해 폴더를 만들고 권한을 설정합니다.

```bash
clustercmd sudo mkdir -p /opt/hadoop_tmp/hdfs
clustercmd sudo chown pi:pi –R /opt/hadoop_tmp
clustercmd sudo mkdir -p /opt/hadoop
clustercmd sudo chown pi:pi /opt/hadoop
```

아래 명령어를 통해 다른 파이들에게 하둡 설치와 설정을 복사해줍니다.

```bash
for pi in $(otherpis); do rsync –avxP $HADOOP_HOME $pi:/opt; done
```

복사가 끝나면 설정을 다시한번 복사 및 적용해줍니다.

```bash
clusterscp ~/.bashrc
clustercmd source ~/.bashrc
```

아래 명령어에 3.2.1이 다른 파이의 개수만큼 뜨면 다른 파이에 하둡이 설치가 잘 되었다는 것입니다.

```bash
clustercmd hadoop version | grep Hadoop
```

## 분산 시스템을 위한 config 변경

먼저 설정 파일이 있는 디렉토리로 변경해줍니다.

```bash
cd /opt/hadoop/etc/hadoop
```

이제 앞서 단일 하둡 시스템에서 편집한 4개의 파일을 다시 편집해 줍니다. 마찬가지로 `<configuration>` 사이에 내용을 입력해줍니다.

```bash
vim core-site.xml
```

```bash
<property>
    <name>fs.default.name</name>
    <value>hdfs://pi1:9000</value>
</property>
```

```bash
vim hdfs-site.xml
```

```bash
<property>
    <name>dfs.datanode.data.dir</name>
    <value>/opt/hadoop_tmp/hdfs/datanode</value>
</property>
<property>
    <name>dfs.namenode.name.dir</name>
    <value>/opt/hadoop_tmp/hdfs/namenode</value>
</property>
<property>
    <name>dfs.replication</name>
    <value>2</value>
</property>
```

```bash
vim mapred-site.xml
```

```bash
<property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
</property>
<property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>\
</property>
<property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
</property>
<property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
</property>
<property>
        <name>yarn.app.mapreduce.am.resource.memory-mb</name>
        <value>512</value>
</property>
<property>
        <name>mapreduce.map.resource.memory-mb</name>
        <value>256</value>
</property>
<property>
        <name>mapreduce.reduce.resource.memory-mb</name>
        <value>256</value>
</property>

```

```bash
vim yarn-site.xml
```

```bash
<property>
        <name>yarn.acl.enable</name>
        <value>0</value>
</property>
<property>
        <name>yarn.resourcemanager.hostname</name>
        <value>pi1</value>
</property>
<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
</property>
<property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>1536</value>
</property>
<property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>1536</value>
</property>
<property>
        <name>yarn.scheduler.minimum-allocation-mb</name>
        <value>128</value>
</property>
<property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
</property>
```

설정 파일 변경이 끝나면 다른 파이에서 데이터 노드와 네임 노드를 삭제하여 초기화 시켜줍니다. 이후 name node가 마스터에서 작동하지 않을 수 있기에 초기화 과정을 거쳐줍니다.

```bash
clustercmd rm –rf /opt/hadoop_tmp/hdfs/datanode/*
clustercmd rm –rf /opt/hadoop_tmp/hdfs/namenode/*
```

하둡 설정 파일을 저장하는 공간에 마스터와 워커들에 대한 정보를 만들어줍니다.

```bash
cd /opt/hadoop/etc/hadoop

touch master
vim master
```bash
pi1
```

```bash
touch workers
vim workers
```

```bash
# localhost를 지우고 아래 내용을 입력합니다.

pi2
pi3
```

이제 변경된 설정파일의 적용을 위해서 전체 파이의 재부팅을 해줍니다.

```bash
clusterreboot
```

각 파이들을 재시작 한 후 마스터 라즈베리파이에서 아래의 명령어를 실행합니다.

```bash
hdfs namenode -format
start-dfs.sh && start-yarn.sh
jps
```

jps 를 쳤을때 마스터에는 namenode가 있고 워커 라즈베리파이에 jps를 쳤을 때 datanode가 있으면 분산 시스템이 잘 적용된 것입니다.
