---
title: docker 远程桌面
date: 2017-06-15 12:06:07
tags: docker
---
## 安装
### 下载配置文件
参考 [github vnd desktop](https://github.com/fcwu/docker-ubuntu-vnc-desktop)，将里面的image文件下载好，放到安装目录
### 需要提前下载tnit，此文件为针对docker使用的init文件
```
https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini
```
### 编辑sources.list文件
在安装目录下建立sources.list文件，并加入
```
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main multiverse restricted universe
```

<!-- more -->

### 编辑DockerFile文件
```
FROM ubuntu:16.04
MAINTAINER light <light@light.com>

ENV DEBIAN_FRONTEND noninteractive


COPY ./sources.list /etc/apt/

COPY ./image/ /
COPY ./tini /bin/tini
RUN chmod +x /bin/tini

RUN apt-get update \
    && apt-get install -y --no-install-recommends software-properties-common curl \
    && add-apt-repository ppa:fcwu-tw/ppa \
    && apt-get update \
    && apt-get install -y --no-install-recommends --allow-unauthenticated \
        supervisor \
        openssh-server pwgen sudo vim-tiny \
        net-tools \
        lxde x11vnc xvfb \
        gtk2-engines-murrine ttf-ubuntu-font-family \
        fonts-wqy-microhei \
        nginx \
        python-pip python-dev build-essential \
        mesa-utils libgl1-mesa-dri \
        gnome-themes-standard gtk2-engines-pixbuf gtk2-engines-murrine pinta \
        dbus-x11 x11-utils \
    && apt-get autoclean \
    && apt-get autoremove \
    && rm -rf /var/lib/apt/lists/*

RUN pip install --upgrade pip && pip install setuptools wheel && pip install -r /usr/lib/web/requirements.txt

EXPOSE 80
WORKDIR /root
ENV HOME=/root \
    SHELL=/bin/bash
ENTRYPOINT ["/startup.sh"]
```
### 运行
```
docker run --device=/dev/nvidiactl --device=/dev/nvidia-uvm --device=/dev/nvidia0 --device=/dev/nvidia1 --device=/dev/nvidia2 --device=/dev/nvidia3 -d -it --dns 202.107.117.11 --volume-driver=nvidia-docker --volume=nvidia_driver_367.48:/usr/local/nvidia:ro --privileged=true -p 6080:80 -e VNC_PASSWORD=lxde_docker --name lxde ab8a1798422c

```
挂载
```
docker run --device=/dev/nvidiactl --device=/dev/nvidia-uvm --device=/dev/nvidia0 --device=/dev/nvidia1 --device=/dev/nvidia2 --device=/dev/nvidia3 -d -it --dns 202.107.117.11 --volume-driver=nvidia-docker --volume=nvidia_driver_367.48:/usr/local/nvidia:ro --privileged=true -p 6080:80 -e VNC_PASSWORD=lxde_docker --volumes-from anaconda3 --name lxde d2e29991c14d
```
