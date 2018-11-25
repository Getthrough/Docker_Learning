# Docker_Learning
record of learning docker

## What is Docker?
Docker 是一个开源项目，它可以将应用程序自动部署在可移植的，独立的容器中。Docker 利用 Linux 内核中的资源分离机制，如 Cgroups 和 namespace 来创建独立的容器（container）。在此之前，部署一个独立的环境通常我们的做法是启动一个虚拟机（virtual machine），建立一个完全独立的操作系统，再进行应用程序的部署。

references:\
https://zh.wikipedia.org/wiki/Docker\
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
`$ sudo apt-get remove docker docker-engine docker.io`\
`/var/lib/docker`目录下的内容，包括 images, containers, volumes, and networks 将会被保留。
#### 安装 Docker CE（社区版）
安装 Docker 社区版有几种不同的方式：
* 大多数用户会建立好 Docker 仓库，然后从仓库安装。以便于安装和升级任务，这种是推荐的方式。
* 一些用户会下载 deb 包进行手动安装和升级，这在系统无法联网的情况下非常有用。
* 在测试和开发环境中，一些用户使用自动化的脚本进行安装。

以上几种方式在下面给出的链接中都有详细描述，这里记录一种简便的方式进行安装：\
通过`wget -qO- https://get.docker.com/ | sh`命令可以直接进行安装。\
设置开机启动：\
`sudo systemctl enable docker`\
`sudo systemctl start docker`\
测试 docker 是否正确安装：\
`docker run hello-world`
## Docker 常用命令
`docker ps`:


references:\
https://docs.docker.com/install/linux/docker-ce/ubuntu/
