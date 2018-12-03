# Docker_Learning
record of learning docker

## What is Docker?
Docker 是一个开源项目，它可以将应用程序自动部署在可移植的，独立的容器中。Docker 利用 Linux 内核中的资源分离机制，如 Cgroups 和 namespace 来创建独立的容器（container）。在此之前，部署一个独立的环境通常我们的做法是启动一个虚拟机（virtual machine），建立一个完全独立的操作系统，再进行应用程序的部署。

references:\
https://zh.wikipedia.org/wiki/Docker \
https://docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/container-docker-introduction/docker-defined

## Comparing Docker containers with virtual machines
听上去 Docker 容器和虚拟机技术十分相像，它们有什么不同吗？
* VM：
虚拟机是指行为方式类似与实际计算机的计算机文件（通常称为镜像），它提供了虚拟硬件，包括 CPU，内存，硬盘等其他设备，并将虚拟硬件映射到物理计算机的真实硬件上。属于完全虚拟化。
* Docker containers: Docker 容器包括了应用程序及其所有依赖项，但是它和其他容器共享操作系统内核，在主机操作系统的用户空间中作为独立进程运行。

由于 Docker 容器不需要完整的操作系统，所以它需要的资源要比 VM 少的多，而且易于部署、启动速度快。使得在同一硬件单元上可以运行更多的服务。但是，作为在同一内核上运行的“副作用”，Docker 容器的隔离程度没有 VM 高。

references:\
https://azure.microsoft.com/zh-cn/overview/what-is-a-virtual-machine/

## Install Docker in Ubuntu
在 Ubuntu 上安装 Docker，系统需要是以下几种的 64 位版本：
* Bionic 18.04 (LTS)
* Xenial 16.04 (LTS)
* Trusty 14.04 (LTS)

#### 卸载 Docker 旧版本
`sudo apt-get remove docker docker-engine docker.io`,\
`/var/lib/docker`目录下的内容，包括 images, containers, volumes, and networks 将会被保留。
#### 安装 Docker CE（社区版）
安装 Docker 社区版有几种不同的方式：
* 大多数用户会建立好 Docker 仓库，然后从仓库安装。以便于安装和升级任务，这种是推荐的方式。
* 一些用户会下载 deb 包进行手动安装和升级，这在系统无法联网的情况下非常有用。
* 在测试和开发环境中，一些用户使用自动化的脚本进行安装。

以上几种方式在下面给出的链接中都有详细描述，这里记录一种简便的方式进行安装(前提网络正常)：\
通过`wget -qO- https://get.docker.com/ | sh`\
或者`curl -sSL https://get.docker.com/ | sh`命令可以直接进行安装,\
设置开机启动：\
`sudo systemctl enable docker`\
`sudo systemctl start docker`\
测试 docker 是否正确安装：\
`docker run hello-world`，这句命令会从 https://hub.docker.com/ 拉取一个叫`hello-world`的 docker 镜像到本地（如果本地没有），并启动这个镜像，如果成功启动表示 docker 安装成功。

references:\
https://docs.docker.com/install/linux/docker-ce/ubuntu/

## 拉取镜像并使用
1. `docker pull ubuntu:latest`: 从 hub.docker.com 上拉取`ubuntu`最新版本镜像;
2. `docker image ls`: 显示镜像列表;
3. `docker run -ti unbuntu:latest /bin/bash`: 运行`ubuntu`进行镜像并进入该容器中（加上`-d`参数则不会进入容器而是后台运行）;
4. `docker attach`: 进入一个正在运行的容器中，如 `docker attach 8aae6257d7c0`, 后面的一串字符是该容器的`id`（`CONTAINER ID`），或者换成容器的名称（`NAMES`）也能进入。
5. `CTRL+P+Q`: 退出容器,容器仍然运行; `CTRL+D`: 退出并关闭容器，未保存的操作将被抛弃。

references:\
https://www.youtube.com/watch?v=UV3cw4QLJLs 视频教程

## Dockerfile
#### 什么是 Dockerfile?
`Dockerfile`定义了容器的环境里所有的动作（即容器要如何运作）。对网络接口和磁盘驱动的访问需要在容器中进行虚拟化，因此需要将容器中的端口映射到外部实际的端口，并且具体说明要复制外部的那些文件到容器中环境。
#### 创建一个 Dockerfile
创建一个空文件夹并进入（cd）该文件夹，创建一个叫 Dockerfile 的文件。\
以下 Dockerfile 内容可以用于制作一个简单的镜像：
````
# 使用 tomcat 基础镜像(镜像名称可用 docker images 命令查看)
from hub.c.163.com/library/tomcat

# 镜像作者信息
MAINTAINER getthrough maoxianbin1994@gmail.com

# 将容器外的文件复制到容器内的指定位置
COPY forFirstDockerfile.war /usr/local/tomcat/webapps
````
#### 构建一个镜像
根据上述 Dockerfile 文件构建一个镜像：`docker build -t my_docker_image:1.0 .`\
`-t`参数表示镜像的`tag`，冒号前面指定的是这个镜像的名称，最后的“.”表示当前目录。\
此时我们`docker images`一下：
````
root@tamen:/home/getthrough/TechLearning/docker/firstDockerfile# docker images
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
my_docker_image                1.0                 af62d5887b0e        11 seconds ago      292MB
````
可以看到，`my_docker_image`镜像已经构建成功。
#### 运行镜像
`docker run -ti -d -p 8080:8080 my_docker_image:1.0`。`-p`参数指定将主机的端口映射到容器内的端口\
`docker ps`查看运行的容器：
````
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS              PORTS                    NAMES
2e487668ba6f        my_docker_image:1.0   "catalina.sh run"   8 seconds ago       Up 5 seconds        0.0.0.0:8080->8080/tcp   tender_colden
````
访问“forFirstDockerfile”应用：\
`http://localhost:8080/forFirstDockerfile/index.jsp`或者\
`curl http://localhost:8080/forFirstDockerfile/index.jsp`可以收到返回数据如下:
````
<html>
<body>
<h2>Hello World!</h2>
<p>This is getthrough's first web app running in docker!</p>
</body>
</html>
````
## 将镜像推送到远程仓库
在推送到仓库之前最好给镜像打一个`tag`，这个`tag`的内容能体现这个版本的镜像的特点或者让你知道这个镜像适用于什么场合，此外`tag`中还可以包含需要推送的远程仓库的地址，如下：
````
* 如果推送到 hub.docker.com（国内网络推送和拉取比较慢）
首先需要先去该站点注册一个用户，然后使用 docker login 进行登录，
登录后进行推送：docker push getthrough/my_docker_image
* 如果推送到其他仓库（例 网易云镜像仓库）
同样，先去 https://www.163yun.com/product/repo 注册一个用户，
使用 docker login -u username -p password hub.c.163.com 进行登录，
然后为镜像打一个 tag，如：docker tag 8aae6257d7c0 hub.c.163.com/getthrough/first-try
最后进行推送 docker push hub.c.163.com/getthrough/first-try
````
references:\
https://www.163yun.com/help/documents/15587826830438400 推送到网易云镜像仓库
## 设置默认镜像仓库
默认拉取镜像的的地址是`hub.docker.com`,切换默认仓库的方式有以下几种：
1. `docker pull registry.docker-cn.com/library/ubuntu`, 在拉取时指定仓库位置
2. 在`/etc/docker/daemon.json`文件中指定镜像仓库：
````
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
````
网易云镜像仓库地址为：`http://hub-mirror.c.163.com`

references:\
https://docs.docker.com/registry/recipes/mirror/#use-case-the-china-registry-mirror

## Docker 常用命令
````
docker image ls: 显示安装的镜像 （加上`--all`会显示所有的镜像，默认隐藏一些中间镜像）
docker container: 管理容器
docker search: 搜索 docker 镜像，如搜索 redis： `docker search redis`
docker pull: 下载镜像，如下载 redis：`docker pull redis:latest`,":"表示标签（TAG）
docker ps: 显示正在运行的容器
docker run: 运行镜像，如 docker run -ti ubuntu:latest
docker start: 运行容器，如 docker start 8aae6257d7c0
docker stop: 关闭容器，如 docker stop 8aae6257d7c0
````
`docker run` \
加上`-v`参数可以将容器内的文件路径挂载到宿主机器（容器外）的文件路径上， 如 `docker run -ti -v /home/data:/data ubuntu /bin/bash`会将`ubuntu`容器中的`/data`目录挂载到宿主机的`/home/data`上;\
加上`-p`参数将容器外的端口映射到容器内，如`docker run -ti -p 192.168.7.254:3306:3306 ubuntu /bin/bash`。


references：\
https://docs.docker.com/engine/reference/commandline/cli/ 官方详细指令文档