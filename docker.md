[TOC]

# Docker


## 版本

- Docker EE 
    - 由公司支持，可在经过认证的操作系统和云提供商中使用，并可运行来自Docker Store的、经过认证的容器和插件
    - https://www.docker.com/pricing
- Docker CE
    - 免费社区版
    - https://github.com/docker/docker-ce/releases

- Moby: Docker的上游项目，是Docker之母
    - https://github.com/moby/moby



## [Install](https://docs.docker.com/engine/install/centos/)

```bash
yum list docker-ce --showduplicates | sort -r  # 历史版本
sudo yum install docker-ce docker-ce-cli containerd.io   # 
sudo docker run hello-world   # 验证安装是否正确, 运行后自动退出

```

## images
- [centos docker hub](https://hub.docker.com/_/centos?tab=tags&page=1)


### docker file & build image

- 减少镜像大小
    - 减少层数, 一个命令一行
- 基础镜像
    - Alpine Linux, 小巧、安全、简单以及功能完备的特点, 被广泛应用于众多Docker容器中
    - 使用了musl，可能和其他Linux发行版使用的glibc实现会有些不同

``` bash
# docker file 

FROM     # 基础镜像, 多个FROM就产生多个镜像
ENV      # 设置环境变量
RUN      # 执行命令, shell格式, exec格式
COPY     # 从上下文目录中复制文件或者目录到容器里指定路径
WORKDIR  # 工作目录
CMD      # docker run 时运行指令


# build image
docker build -t myrefenv:1.2 -f xxx.dockerfile

# 生成的镜像也被称为黑箱镜像，除了制定镜像的人知道怎么生成的镜像，别人根本无从得知
# 有一些特殊的应用场合，比如被入侵后保存现场等
docker commit 67106c9b26e0 myrefenv:1.3  #从容器中创建新的镜像

```

### image pull&push
``` bash
# login
docker login docker-registry.xxxx.virtual      # 内部镜像存储
docker login https://index.docker.io/v1/       # hub.docker.com镜像存储
# pull
docker pull <image>
# push
docker push <image>

# logout
docker logout docker-registry.xxxx.virtual

# remove
docker rmi <image> | docker rmi <REPOSITORY:TAG>
```

## docker

``` bash
docker system df   # 查看docker磁盘占用

docker cp  # 用于容器与宿之间的数据拷贝
docker cp b6ef866a1d0f:/var/www/cacti-1.2.9/ /data/dockershare/cacti # 容器->宿主机
```

- dock run

``` bash

# -p hostPort:containerPort
# -v hostPath:containerPath
sudo docker run -d --name cacti2 --privileged=true -p 80:80 \
     -v /data/dockershare/cacti:/var/www/cacti \
     registry.kenc.ksyun.com/dev/os7.8-cacti-1.2.9:1.0 \
     /usr/sbin/init                      # 运行 docker 
sudo docker exec -it cacti2 /bin/bash    # 在运行的 docker 中执行命令

docker -p 6001:8001 run --rm  --name hello-k8s hello-k8s:v1.0

docker run -d --name dev -p 80:80 -p 6241:6241 -p 6242:6242 -p 6060:6060 -p 8080:8080 -v /root/share:/root/vmshare registry.kenc.ksyun.com/publish/centos7.8:2.4 /usr/sbin/init
docker exec -it dev /bin/zsh

```

``` bash

# for network

# for logs
docker logs 695f9340ea28 --tail 100

# for disk
docker system df        # 查看docker磁盘使用情况
docker system df -v     # 查看详情
docker system prune     # 清理磁盘，删除关闭的容器、无用的数据卷和网络，以及dangling镜像(即无tag的镜像)
docker system prune -a  # 删除更彻底, 包括未使用的镜像

```

## docker-compose


## overlay2

- 文件系统


## ref
- 开放容器标准(OCI)

## faq
- docker 长时间运行, 网络不通问题
