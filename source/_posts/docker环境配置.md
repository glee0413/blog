---
title: docker环境配置
date: 2017-06-15 09:03:59
tags: docker
---

## 安装docker
```
sudo apt-get install docker docker.io
sudo service docker start
sudo docker info
```
### 不使用sudo进行任务服务查看
由于docker使用unix域套接字，除了docker和root，普通用户无法直接使用，用root又不太安全因此需要将当前用户加入到docker组中
```
sudo groupadd docker
sudo usermod -aG docker cosmos
```
<!-- more -->
## 设置docker代理
### docker代理需要独立设置
```
mkdir /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/http-proxy.conf
```
添加如下内容
```
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80/"
Environment="NO_PROXY=localhost,127.0.0.0/8,docker-registry.somecorporation.com" //如果有本地registy的使用这个跳过代理
```
### 查看环境变量
```
sudo systemctl show docker --property Environment
```
### 添加dns
```
#添加新配置文件
/etc/docker/daemon.json
```
填入一下内容
```
{
    "dns": ["domainIP"]
}
```
### 刷新环境变量
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## docker镜像保存路径
docker默认保存在```/var/lib/docker```路径下，磁盘分区不足的情况下需要更改位置
## 安装运行镜像
### 安装并运行
```
## 以registry为例
```
docker pull registry
docker run -d -p 5000:5000 --restart=always --name registry
```
### 修改提交镜像
```
docker commit -m "add vim and sudo command" -a "author" $CONTAINER_ID author/domain:tag
```
### 上传镜像
```
# 通过tag表示要上传的registy
docker tag domain/image:tag ip:5000/domain/image:tag
# 如果是dockerhub或阿里其他的非本地registry，需要登入
docker login --username=username registry.cn-hangzhou.aliyuncs.com
docker push 10.10.142.69:5000/domain/image:tag
# 删除
docker rmi 10.10.142.69:5000/domain/image:tag
```
本地registry配置时需要删除代理，并跳过证书
### 跳过证书配置
```
# 再/etc/default/docker或/etc/sysconfig/docker中添加
DOCKER_OPTS="--insecure-registry registryIP:5000"
```

## 进入docker
```
docker ps -a # 查看容器
docker images # 查看镜像
docker exec -it containerId /bin/bash
docker attach --sig-proxy=false $CONTAINER_ID
```
## 构建开发环境
1，编写Dockerfile文件，用语构建环境
```
FROM ubuntu:16.04  
MAINTAINER light "light@light.com" 
WORKDIR /
CMD ["/bin/bash"]
```
2，执行构建
```
docker build -t light/lpr_build .
```
## 查看docker镜像及容器的继承关系
使用[dockviz](https://github.com/justone/dockviz)
```
dockviz images -d -l | dot -Tpng -o images.png
```
## 容器与宿主及文件交互
```
docker cp opencv $CONTAINER_ID:/root
```
## docker 数据卷
可实现创造一个data volmumes，然后启动
```
docker run -d --dns domainIP -v /root/anaconda3 --name anaconda3 domain/anaconda3 echo "Anaconda3 data volmume"
```
在需要使用数据卷的地方调用
```
docker run --device=/dev/nvidiactl --device=/dev/nvidia-uvm --device=/dev/nvidia0 --device=/dev/nvidia1 --device=/dev/nvidia2 --device=/dev/nvidia3 -d -it --dns domainIP --volume-driver=nvidia-docker --volume=nvidia_driver_367.48:/usr/local/nvidia:ro --privileged=true --volumes-from anaconda3 -p 7080:80  --name develop cb5c90540bdb
```
注意数据卷不意为这保存数据，数据的保存还是要通过将宿主的目录挂载到docker中来实现，而数据卷可以用来保存一些公用的程序及配置
## nvidia docker配置
### 下载安装
```
wget https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.1/nvidia-docker-1.0.1-1.x86_64.rpm
sudo rpm -ivh nvidia-docker-1.0.1-1.x86_64.rpm
sudo systemctl start nvidia-docker
```
### 运行
```
docker run --device=/dev/nvidiactl --device=/dev/nvidia-uvm --device=/dev/nvidia0 --device=/dev/nvidia1 --device=/dev/nvidia2 --device=/dev/nvidia3 -d -it --dns dnsip --volume=nvidia_driver_367.48:/usr/local/nvidia:ro --privileged=true domain/image:tag /bin/bash

docker run --device=/dev/nvidiactl --device=/dev/nvidia-uvm --device=/dev/nvidia0 --device=/dev/nvidia1 --device=/dev/nvidia2 --device=/dev/nvidia3 -d -it --dns dnsip --volume-driver=nvidia-docker
--volume=nvidia_driver_367.48:/usr/local/nvidia:ro --privileged=true --name u16_cuda domain/image:tag /bin/bash
# 如果不想每次都使用  --volume-driver=nvidia-docker 可以使用下面的命令，先将volume建出来
docker volume create --name=nvidia_driver_367.48-d nvidia-docker
```

### 更新ld.conf.d
添加/etc/ld.so.conf.d/cuda.conf文件，并加入 /usr/local/cuda/lib64 
添加nvidia.conf文件，并加入
```
/usr/local/nvidia/lib
/usr/local/nvidia/lib64
```
