

## 概述
- Linux容器的强大功能- 它们让软件开发产生革命性的变化并推动微服务转变整个行业
- 服务发现
- 服务代理

## cgroups

- 内核功能，能控制和限制一个或一组进程的资源使用
- cgroups 暴露出的操作接口时文件系统
- 在子系统中**创建目录(控制组)**, docker在每个子系统下为每个容器创建**控制组**
- 将进程PID加到控制组的task文件中, 这样就可以对进程进行限制了

```bash
mount -t cgroup  # 查看子系统, 即可限制的各种资源

cat /sys/fs/cgroup/cpu,cpuacct/cpu.cfs_period_us    # cpu period, 100ms
100000                                            
cat /sys/fs/cgroup/cpu,cpuacct/cpu.cfs_quota_us     # 配额, -1, 表示不限制, 20000 表示20ms, 即最多使用20%cpu资源
-1

cgexec -g cpu,cpuacct,memory:xxx  # 程序启动时进行对进程进行限制
```

## namspace

- 隔离


## Docker

- 启动消耗速度、资源消耗、弹性伸缩表现极为高效; 隔离性与安全性一般
- Docker组成
  - Docker Daemon
  - Docker Client
  - Docker Image
  - Docker Registry: 镜像的存储与分发
- 主要的配置文件
  - /etc/sysconfig/docker
  - /etc/sysconfig/docker-network
  - /etc/sysconfig/docker-storage
- 容器管理: kubernetes, apache mesos, fleet
  - 登录镜像存储
    - 拷贝证书,到此目录下/etc/docker/certs.d/docker-registry.xxxx.virtual/ca.crt
    - docker login docker-registry.xxxx.virtual
  - 拉取镜像 docker pull <image>
  - 上传镜像 docker push <image>
  - 退出镜像存储: docker logout docker-registry.xxxx.virtual

## 安装部署
```
- systemctl start docker
- 添加私库的证书到/etc/docker/certs.d   //参考 https://docs.docker.com/engine/security/certificates/

```

## 管理镜像
- apache mesos
- kubernetes(k8s)
  - 可跨公有云、私有云、混合云
  - http://www.dockone.io/article/932

## 创建镜像
  - 编辑Docker文件
  - docker build -t myrefenv:1.2 ./
  - docker commit 67106c9b26e0 myrefenv:1.3  #从容器中创建新的镜像
    - 生成的镜像也被称为黑箱镜像，除了制定镜像的人知道怎么生成的镜像，别人根本无从得知
    - 有一些特殊的应用场合，比如被入侵后保存现场等

## 拉取上传镜像
- 镜像名字
  - 地址/项目/镜像:Tag
- 登录镜像存储: 
  - docker login docker-registry.xxxx.virtual      #内部镜像存储
  - docker login https://index.docker.io/v1/       #hub.docker.com镜像存储
- 拉取镜像:
  - docker pull <image>
- 上传镜像:
  - docker push <image> 
- 退出镜像存储: 
  - docker logout docker-registry.xxxx.virtual

## 管理本地镜像
- 查看本地镜像: docker images 
- 删除镜像: docker rmi <image> | docker rmi <REPOSITORY:TAG>

## 管理容器
- 常用命令
  - 查看Docker系统级信息: docker info
  
  - 从registry拉取镜像: docker pull <image>  //镜像名必须为 <registry_host>/<repo>:<tag>
    - eg: docker-registry.xxxx.virtual/library/centos6:6-xxxx-15

- 启动容器
``` bash
  docker run -it REPOSITORY:TAG COMMAND   //前台运行
    eg: docker run -it docker-registry.xxxx.virtual/library/centos7:7-ixxxx-12 /bin/bash
  -p hostPort:containerPort  # 映射宿主机的指定端口到容器的指定端口, 可以有多个 -p
  -v hostPath:containerPath  # 挂载数据卷, 映射宿主机目录到容器的指定目录, 可以有多个 -v

```

  - 查看运行的镜像: docker ps 或 docker ps -a
  - 查看容器/镜像详细信息: docker inspect <image>|<container>
  - 修改镜像名称: docker tag <imageid> <name:tag>
  - 查看容器标准输出: docker logs [args] <container>
  - 连接到容器
    - 连接运行的容器: docker attach <container>
    - 连接停止的容器: 先start, docker stop <container>; docker attach <container>
  - 停止容器: docker stop [args] <container>
  - 退出容器并后台运行: **Ctrl+P+Q**
  - 查找镜像: docker search httpd
  - 停止容器:
    - docker stop $(docker ps -a | grep "Exited" | awk '{print $1 }')
  - 删除容器: docker rm <CONTAINER ID>
    - docker rm $(docker ps -a | grep "Exited" | awk '{print $1 }') 
- 常用地址
  - https://hub.docker.com/

## 容器网络
- 四种模式
  - host模式: 与主机共享 network namespace
  - container模式: container 之间共享 network namespace
  - none模式: container拥有独立的 network namespace, 但是没有进行网络配置
  - bridge模式: container拥有独立的 network namespace, 默认配置,连接主机的虚机网桥
- 如何访问外部网络
  - 自动在主机iptables配置SNAT,将源地址换为主机非虚拟地址(与外部通讯的地址)
- 如何提供外部服务
  - Dockerfile,指定EXPOSE端口
  - 启动容器时指定相关参数，如: docker run --name hello-world -p 8080:8080 -d hello-world:1.0
  - 在主机iptables配置DNAT,将收到的的目的地址转换为容器的地址
- 查看网络配置情况
  - iptables -t nat -nvL

## 容器瘦身


# 参考资料
- 容器命令: https://www.cnblogs.com/Survivalist/p/11199292.html
- https://www.ibm.com/developerworks/cn/cloud/library/cl-microservices-in-action-part-1/
- https://www.ibm.com/developerworks/cn/cloud/library/cl-bluemix-microservices-in-action-part-2-trs/
- 容器网络: https://www.cnblogs.com/gispathfinder/p/5871043.html
- 容器瘦身: http://dockone.io/article/8174


# FAQ
- 上传的镜像与本地镜像不一样大，上传的可能被压缩了
- docker run -it -p 5556:5556 docker-registry.xxxx.virtual/haozhaojun/netpathmetry:0.1 /bin/zsh
- Failed to get D-Bus connection: Operation not permitted 问题
  - https://www.cnblogs.com/liwutao/p/13353819.html
