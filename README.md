# Docker_Learning
record of learning docker

## What is Docker?
Docker 是一个开源项目，它可以将应用程序自动部署在可移植的，独立的容器中。\
Docker 是 LXC（Linux Container）的最佳实践，它利用 Linux 内核中的资源分离机制，如 Cgroups 和 namespace 来创建独立的容器（container）。
在此之前，部署一个独立的环境通常我们的做法是启动一个虚拟机（virtual machine）或者直接使用一台安装了 OS 的服务器，建立一个完全独立的操作系统，再进行应用程序的部署。\
这种情况下除了应用程序包可以保证完全相同，其他的如环境变量，JDK 版本，MYSQL 版本，redis 版本等保证与开发环境完全一致并不是那么容易。这就可能导致开发环境下运行正常的程序部署到生产环境中出现问题的情况时有发生。\
如果使用虚拟机进行分布式（微服务）的模拟，多台虚拟机需要分配的硬件资源十分可观。而 Docker 的每个容器就是服务器上被“隔离”的一个进程，与主机系统共享主机资源，资源占用低，并且容易模拟分布式环境，并且每一个镜像中都包含了（需要在 Dockerfile 中定义）所有的依赖，一旦容器正常运行，那么容器的环境将会和定义文件中描述的完全一致。（这时候生产环境出问题，只能是代码的问题了

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

## Install Docker for CentOS
需要注意: CentOS 7 之前的 docker 版本不再提供维护和测试.
#### 卸载 Docker 旧版本
````
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
````
### 卸载新版本 docker-ce
1. 卸载 docker ce 包
````
sudo yum remove docker-ce
````
2. 删除所有镜像,容器,和卷(volumes)
````
sudo rm -rf /var/lib/docker
````
在测试卸载docker时,发现只执行上述两个命令,重新安装会提示文件冲突,再删除`docker-ce-cli`后才能正常安装:
````
sudo yum remove docker-ce-cli
````
#### 在 CentOS 上安装 Docker CE 的几种方式
* 通过设置 Docker 仓库并从仓库安装(推荐的方式) (1)
* 在没有联网的情况下,可以通过下载 RPM 包进行手动安装 (2)
* 在测试和开发的环境中,可以选择自动话的脚本进行安装 Docker (3)

##### 方式1 设置 docker 仓库进行安装
###### 安装最新的 Docker CE 版本
1. 安装必须的包
````
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
````
2. 设置稳定的仓库(stable repository)
````
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
````
3. (可选) 让 edge 和 test 仓库可用(默认不可用)
````
sudo yum-config-manager --enable docker-ce-edge
sudo yum-config-manager --enable docker-ce-test
````
4. 安装 Docker CE
````
sudo yum install docker-ce
````
###### 安装指定的 Docker CE 版本
1. 列出在你的仓库中可用的 docker 版本, 下面的排序方式是从高版本到低版本:
````
yum list docker-ce --showduplicates | sort -r
// 输出
docker-ce.x86_64            3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64            18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                    docker-ce-stable
...
````
2. 安装指定版本
````
// 例如安装排序输出的第一个版本
sudo yum install docker-ce-3:18.09.1-3.el7
````
至此 Docker 安装已经完成,但是并没有运行.名为 docker 的用户组已经创建.
###### 启动 Docker
在启动前执行`docker ps`命令:
````
[root@localhost ~]# docker ps
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
````
使用如下命令启动:
````
sudo systemctl start docker
````
再执行 `docker ps`
````
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS 
````
验证 docker 是否正确安装:
````
root@localhost ~]# sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete 
Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
````
###### 更新 Docker CE 版本
重复之前的安装步骤,选择新的版本进行安装
````
// 更新前
[root@localhost docker_rpm]# docker -v
Docker version 18.06.0-ce, build 0ffa825
// 执行更新语句
sudo yum install docker-ce-18.06.1.ce-3.el7
// 更新后
[root@localhost docker_rpm]# docker -v
Docker version 18.06.1-ce, build 0ffa825
````
**NOTE : 只能更新新的版本,新版本无法使用该语句回退到旧版本**
##### 方式2 下载 RPM 包进行安装
1. 下载 RPM 包:\
RPM 包下载地址:`https://download.docker.com/linux/centos/7/x86_64/stable/Packages/`
2. 安装 docker ce
````
sudo yum install /tmp/docker_rpm/docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm
````
3. 启动 docker
````
sudo systemctl start docker
````
4. 验证是否正确安装
````
sudo docker run hello-world
````
###### 更新 Docker CE 版本
将`yum install`改为`yum upgrade`,并指定更新版本即可:
````
sudo yum upgrade /tmp/docker_rpm/docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm
````
##### 方式3 使用便捷脚本进行安装
不推荐使用脚本的方式在**生产环境**上安装docker,可能会有一些潜在的风险.
````
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
````
同样需要手动启动 docker.

**NOTE : docker 默认在 root 用户下才能使用,如果其他用户(non-root user)需要使用 docker,需要将另外的用户加入到 docker 用户组中.**
````
sudo usermod -aG docker your-user
````
例如,使用 root 创建一个用户名为 getthrough2 的用户,并尝试使用docker:
````
// 创建用户
useradd -d /usr/getthrough2 -m getthrough2
// 指定密码
passwd getthrough2
// 切换用户
su - getthrough2
// 尝试查看docker容器
[getthrough2@localhost ~]$ docker ps
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get
http://%2Fvar%2Frun%2Fdocker.sock/v1.39/containers/json: dial unix /var/run/docker.sock: connect: permission denied
````
提示拒绝链接.需要将 getthrough2 用户加入 docker 用户组:
````
[root@localhost getthrough2]# sudo usermod -aG docker getthrough2
[root@localhost getthrough2]# su getthrough2
[getthrough2@localhost ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[getthrough2@localhost ~]$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
````

## Install Docker for Ubuntu
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

## 保存镜像和加载镜像
在没有联网的环境下无法从外网环境拉取镜像,可以通过提前将镜像保存后再进行加载的方式使用.
### 保存镜像文件
查看已有的 docker 镜像:
````
[getthrough2@localhost ~]$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        2 weeks ago         1.84kB
````
有一个 hello-world 镜像,将其保存:
````
// 指定镜像既可以使用镜像名称也可以指定镜像id
[getthrough2@localhost ~]$ docker save -o hello-world.tar hello-world
[getthrough2@localhost ~]$ ls
hello-world.tar
````
### 加载镜像文件
在`/usr/getthrough2`目录下有镜像文件`hello-world.tar`:
````
[getthrough2@localhost ~]$ docker load -i hello-world.tar
Loaded image: hello-world:latest
[getthrough2@localhost ~]$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        2 weeks ago         1.84kB
````
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
下面关于 Dockerfile 的几个命令引用自阮一峰的博客 http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html
````
# 该 image 文件继承官方的 node image，冒号表示标签，这里标签是8.4，即8.4版本的 node
FROM node:8.4
# 将当前目录下的所有文件（除了.dockerignore排除的路径），都拷贝进入 image 文件的/app目录
COPY . /app
# 指定接下来的工作路径为/app
WORKDIR /app
# 在/app目录下，运行npm install命令安装依赖。注意，安装后所有的依赖，都将打包进入 image 文件
RUN npm install --registry=https://registry.npm.taobao.org
# 将容器 3000 端口暴露出来， 允许外部连接这个端口
EXPOSE 3000
# CMD 命令表示容器启动后自动执行的语句
CMD node demos/01.js
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
## Docker Compose
#### 什么是 Docker-Compose？
Docker-Compose 是 Docker 官方提供的一个工具软件，用于将多个容器组成一个应用。想象一下，启动一个 Java 应用可能需要同步启动 JVM，MYSQL，MQ，REDIS，ES 等一大堆容器，也是比较繁琐的。
使用 Docker-Compose 在`docker-compose.yml`文件中定义好各个服务以及服务间的连接方式，就能一行命令启动了。
#### 安装 Docker-Compose
安装了 docker 的 Mac 和 Windows 系统无需另外安装 Docker-Compose;\
Linux 下安装 Docker-Compose 步骤如下：
1. 下载最新版本：`sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
2. 添加可执行权限：`sudo chmod +x /usr/local/bin/docker-compose`
3. （可选）安装命令行补全工具：
* Bash 命令补全：\
在 `/etc/bash_completion.d/`（或者 `/usr/local/etc/bash_completion.d/` on a Mac）执行如下命令：
````
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.23.1/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
````
Mac 上还有几步省略，具体步骤见引用地址。

#### 使用 Docker-Compose
以下示例会使用 Docker-Compose 构建一个简单的 Python web 应用。
##### 步骤1 设置
1) 为此项目创建一个目录：
````
mkdir composetest
cd composetest
````
2) 创建 app.py 文件，该文件内容如下：
````
import time

import redis
from flask import Flask


app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
````
3) 创建 requirements.txt 文件，内容如下：
````
flask
redis
````
##### 步骤2 创建 Dockerfile
这一步会创建一个用于构建镜像的 Dockerfile 文件，构建好的镜像会包含所有这个 Python 应用所有的依赖。\
在项目根目录（composetest 目录）下，创建 Dockerfile 文件，包含如下内容：
````
FROM python:3.4-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
````
上述内容告诉 docker 做如下动作:
````
* 构建一个基于 python 3.4 的镜像
* 添加当前目录到镜像中的 /code 目录
* 设置工作目录为 /code
* 安装 python 依赖
* 设置容器启动后运行的命令 python app.py
````
##### 步骤3 在 Compose 文件中定义一些服务
在项目根目录下创建 docker-compose.yml 文件，其内容包括：
````
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
    volumes:
     - .:/code
  redis:
    image: "redis:alpine"
````
上述 docker-compose.yml 文件定义了 web 和 redis 两个服务。\
web 服务：
````
* 使用当前目录下的 Dockerfile 构建镜像
* 将容器内的 5000 端口映射到主机上的 5000 端口
* 将容器内的 /code 目录映射到主机当前目录，此时在主机上对当前目录下文件的操作也会同步到容器内
````
redis 服务：
````
* 使用从远程仓库拉取的 redis 镜像
````
##### 步骤4 使用 Compose 构建和运行项目
1. 在项目根目录下启动命令：`docker-compose up`
2. 在浏览器中访问`http://localhost:5000/`，启动成功后页面会展示如下内容：
    ````
    Hello World! I have been seen 1 times.
    ````
3. 刷新页面，次数变成 2 次。
4. 打开另一个终端，使用`docker image ls`命令，会发现镜像列表会包含`redis`和`composetest_web`
5. 可以在刚打开的另一个终端里执行`docker-compose down`关闭应用，或者在应用启动的终端按`CTRL+C`
##### 一些其他命令
* `docker-compose up -d`后台运行程序
* `docker-compose run`可以运行一次性的命令，类似本次运行指定某个环境变量等，\
比如`docker-compose run web env`可以查看 web 服务有哪些环境变量
* 如果是以后台方式启动的容器，可以使用`docker-compose stop`进行关闭，\
如果使用`docker-compose down`则会关闭所有，并且将删除容器文件。
 


references：\
https://docs.docker.com/compose/completion/#install-command-completion

## Docker 常用命令
````
docker image ls/docker images: 显示安装的镜像 （加上`--all`会显示所有的镜像，默认隐藏一些中间镜像）
docker container: 管理容器
docker search: 搜索 docker 镜像，如搜索 redis： `docker search redis`
docker pull: 下载镜像，如下载 redis：`docker pull redis:latest`,":"表示标签（TAG）
docker ps: 显示正在运行的容器
docker run: 运行镜像，如 docker run -ti ubuntu:latest
docker start: 运行容器，如 docker start 8aae6257d7c0
docker stop: 关闭容器，如 docker stop 8aae6257d7c0
docker image rm: 删除镜像，如 docker image rm ngnix 或者 docker image rm 62f816a209e6（容器ID）
docker container inspect: 查询某个容器详细的配置信息，如 docker container inspect ubuntu（先使用 docker container ls -l 列出容器）
````
`docker run` \
加上`-v`或者`--volume`参数可以将容器内的文件路径映射到宿主机器（容器外）的文件路径上， 如 `docker run -ti -v /home/data:/data ubuntu /bin/bash`会将`ubuntu`容器中的`/data`目录挂载到宿主机的`/home/data`上;\
加上`-p`参数将容器外的端口映射到容器内，如`docker run -ti -p 192.168.7.254:3306:3306 ubuntu /bin/bash`。\
加上`--rm`参数可以在容器停止的时候删除容器文件。容器文件：启动一个镜像会产生相应的一个文件。\
加上`--link`参数表示将当前容器连接到某个容器，如`--link mysqldb:mysql`,":"表示别名。

`docker run`与`docker start`的区别：`run`命令表示启动一个镜像，每一次启动镜像都会生成一个镜像文件，而`start`命令表示启动一个已经存在的镜像文件。

references：\
https://docs.docker.com/engine/reference/commandline/cli/ 官方详细指令文档
