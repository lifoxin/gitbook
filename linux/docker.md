### 安装docker环境

**ubuntu版本安装**

	安装
	
		yum -y install docker-io 
		
	修改镜像加速
	
		vi /etc/docker/daemon.json  
			"registry-mirrors": ["http://hub-mirror.c.163.com"]   
**CentOS版本安装**

	网络问题，没有则忽略
	
		vi /etc/profile

		  export http_proxy=http://proxy.example.com:8080
		  export https_proxy=https://proxy.example.com:8080

		source /etc/profile

	卸载旧版本

		yum remove docker  docker-common docker-selinux docker-engine

	更新源

		yum update
		yum-config-manager --add-repo \
		https://download.docker.com/linux/centos/docker-ce.repo

	查看可安装版本，选最新版
	
		yum list docker-ce --showduplicates | sort -r  
		yum install docker-ce-18.06.3.ce
		
		粗暴安装
		#!/bin/bash
		set -e
		curl -fsSL https://get.docker.com | sh &&  \
		curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose \ 
				-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && \
		chmod +x /usr/local/bin/docker-compose

	查看版本

		docker version

	启动服务

		systemctl start docker
		systemctl enable docker

	docker代理，没有则忽略

		vi /lib/systemd/system/docker.service

		  [Service]
		  Environment="HTTP_PROXY=http://proxy.example.com:8080/" \
		  "HTTPS_PROXY=https://proxy.tencent.com:8080/"

		systemctl daemon-reload
		systemctl restart docker

	测试docker

		docker search mysql

### docker基本命令

| 命令|说明|
|--|--|
| docker run --name=xxx image | 创建命名容器 |
| docker run --name=felix --link name:alias image| 容器felix关联name容器|
| docker exec -it mysql bash | 进入容器 |
| docker run -v HostCatalog:ContainerCatalog image | 挂载容器目录(绝对路径)|
| docker run -d -v /dbdata --name dbdata image | 创建数据卷容器 |
| docker run -d --volumes-from dbdata image | 容器挂载dbdata容器中的数据卷|
| docker rm -v container | 删除关联数据库容器 |
| docker run -p 3306:3306 -p 8080:80 image | 端口映射 宿主:容器|
| docker run -P  image | 随机端口映射 |
| docker port container portID | 查看对应映射端口 |
| docker run -it --rm image /bin/bash | 运行后退出删除容器 | 
| docker ps -a -l | 查看容器|
| docker inspect  container | 查看容器信息 |
| docker start/attach -i container | 启动进入容器 |
| docker rm -f container | 强制删除容器 |
| docker rm $(docker ps -a -q) | 删除停止后的容器 | 
| ctrl + P / ctril + Q | 从当前容器交互模式退出 |
| docker run -d container  | 后台运行容器 |
| docker logs -tf --tail  container | 查看容器日志 |
| docker top container | 查看容器进程|
| docker exec container command parameter | 容器执行命令 |
| docker stop/kill contaioner | 结束容器|
| docker images | 查看镜像 |
| docker rmi image| 删除镜像 |
| docker search image/tag | 搜索镜像 | 
| docker pull image/tag | 拉取镜像 |
| docker push image/tag | 推送镜像 | 
| docker export container > ubuntu.tar| 导出容器快照到本地文件 |
| docker import $(cat ubuntu.tar) felix/ubuntu:v1.0 | 导入快照为镜像|

---

### 利用数据卷容器来备份、恢复、迁移数据卷

**备份**

docker run -d -v /dbdata --name dbdata training/postgres

docker run --volumes-from dbdata -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata

**恢复**

docker run -v /dbdata --name dbdata2 ubuntu

docker run --volumes-from dbdata2 -v $(pwd):/backup busybox tar xvf /backup/backup.tar

**验证**

docker run --volumes-from dbdata2 busybox /bin/ls /dbdata

---

### 搭建WordPress博客

**创建docker博客配置文件**

	mkdir myweb && cd myweb 
	vim docker-compose.yaml

	version: '3.1'
	services:
	wordpress:
	image: wordpress
	restart: always
	volumes:
		- ./website:/var/www/html #本地：容器
	ports:
		- 80:80
	environment:
		WORDPRESS_DB_HOST: db #连接数据库的地址,必须要用依赖的服务的名字,这里是 db
		WORDPRESS_DB_USER: root #你自定义连接数据库的用户名,但需要和下面的mysql设置的要一致
		WORDPRESS_DB_PASSWORD: mysqlpass #你自定义连接数据库的密码,但需要和下面的mysql设置的要一致
		WORDPRESS_DB_NAME: exampledb #你自定义的数据库名字,但需要和下面的mysql设置的要一致
	db:
	image: mysql:5.7
	restart: always
	volumes:
		- ./mysql_data/data:/var/lib/mysql 
	environment:
		MYSQL_DATABASE: exampledb #数据库名字
		MYSQL_USER: exampleuser #数据库用户名
		MYSQL_ROOT_PASSWORD: mysqlpass #数据库密码

**运行**
 
	docker-compose up -d

---

### Dockerfile
使用 Dockerfile 可以允许用户创建自定义的镜像。

**基本结构**

Dockerfile 由一行行命令语句组成，并且支持以 # 开头的注释行。

一般的，Dockerfile 分为四部分：基础镜像信息、维护者信息、镜像操作指令和容
器启动时执行指令。

例如
```bash
# This dockerfile uses the ubuntu image

# Base image to use, this must be set as the first line
FROM ubuntu

# Maintainer: docker_user <docker_user at email.com> (@docker_user)
MAINTAINER docker_user docker_user@email.com

# Commands to update the image
RUN echo "deb http://archive.ubuntu.com/ubuntu/raring/main/universe" >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y nginx
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
# Commands when creating a new container
CMD /usr/sbin/nginx
```
其中，一开始必须指明所基于的镜像名称，接下来推荐说明维护者信息。

后面则是镜像操作指令，例如 RUN 指令， RUN 指令将对镜像执行跟随的命令。
每运行一条 RUN 指令，镜像添加新的一层，并提交。

最后是 CMD 指令，来指定运行容器时的操作命令。

下面是一个更复杂的例子
```bash
# Nginx
FROM ubuntu
MAINTAINER Victor Vieux <victor@docker.com>
RUN apt-get update && apt-get install -y inotify-tools nginx apache2 openssh-server

# Firefox over VNC
FROM ubuntu
RUN apt-get update && apt-get install -y x11vnc xvfb firefox
RUN mkdir /.vnc
RUN x11vnc -storepasswd 1234 ~/.vnc/passwd
# Autostart firefox (might not be the best way, but it does the
trick)
RUN bash -c 'echo "firefox" >> /.bashrc'
EXPOSE 5900
CMD ["x11vnc", "-forever", "-usepw", "-create"]

# Multiple images example
FROM ubuntu
RUN echo foo > bar
# Will output something like ===> 907ad6c2736f
FROM ubuntu
RUN echo moo > oink
# Will output something like ===> 695d7793cbe4
# You᾿ll now have two images, 907ad6c2736f with /bar, and 695d7793cbe4 with /oink.
```
**指令**

指令的一般格式为 INSTRUCTION arguments ，指令包括
FROM 、 MAINTAINER 、 RUN 等。

### FROM

格式为 FROM <image> 或 FROM <image>:<tag> 。

第一条指令必须为 FROM 指令。并且，如果在同一个Dockerfile中创建多个镜像
时，可以使用多个 FROM 指令（每个镜像一次）。

### MAINTAINER

格式为 MAINTAINER <name> ，指定维护者信息。

### RUN
格式为 RUN <command> 或 RUN ["executable", "param1", "param2"] 。

前者将在 shell 终端中运行命令，即 /bin/sh -c ；后者则使用 exec 执行。指
定使用其它终端可以通过第二种方式实现，例如 RUN ["/bin/bash", "-c",
"echo hello"] 。

每条 RUN 指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较
长时可以使用 \ 来换行。

### CMD
支持三种格式

- CMD ["executable","param1","param2"] 使用 exec 执行，推荐方式；
- CMD command param1 param2 在 /bin/sh 中执行，提供给需要交互的应
用；
- CMD ["param1","param2"] 提供给 ENTRYPOINT 的默认参数；

指定启动容器时执行的命令，每个 Dockerfile 只能有一条 CMD 命令。如果指定了
多条命令，只有最后一条会被执行。

如果用户启动容器时候指定了运行的命令，则会覆盖掉 CMD 指定的命令。

### EXPOSE
格式为 EXPOSE <port> [<port>...] 

告诉 Docker 服务端容器暴露的端口号，供互联系统使用。在启动容器时需要通过 -
P，Docker 主机会自动分配一个端口转发到指定的端口。

### ENV
格式为 ENV <key> <value> 。 指定一个环境变量，会被后续 RUN 指令使用，
并在容器运行时保持。

例如

    ENV PG_MAJOR 9.3
    ENV PG_VERSION 9.3.4
    RUN curl -SL http://example/$PG_VERSION.tar.xz 
    ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
### ADD
格式为 ADD <src> <dest> 。

该命令将复制指定的 <src> 到容器中的 <dest> 。 其中 <src> 可以是
Dockerfile所在目录的一个相对路径；也可以是一个 URL；还可以是一个 tar 文件
（自动解压为目录）。

### COPY
格式为 COPY <src> <dest> 。

复制本地主机的 <src> （为 Dockerfile 所在目录的相对路径）到容器中的
<dest> 。

当使用本地目录为源目录时，推荐使用 COPY 。
### NTRYPOINT
两种格式：
- ENTRYPOINT ["executable", "param1", "param2"]
- ENTRYPOINT command param1 param2 （shell中执行）。

配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。

每个 Dockerfile 中只能有一个 ENTRYPOINT ，当指定多个时，只有最后一个起
效。
### VOLUME
格式为 VOLUME ["/data"] 。

创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保
持的数据等。
### USER
格式为 USER daemon 。

指定运行容器时的用户名或 UID，后续的 RUN 也会使用指定用户。

当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建
所需要的用户，例如： RUN groupadd -r postgres && useradd -r -g
postgres postgres 。要临时获取管理员权限可以使用 gosu ，而不推荐
sudo 。
### WORKDIR
格式为 WORKDIR /path/to/workdir 。

为后续的 RUN 、 CMD 、 ENTRYPOINT 指令配置工作目录。

可以使用多个 WORKDIR 指令，后续命令如果参数是相对路径，则会基于之前命令
指定的路径。例如

    WORKDIR /a
    WORKDIR b
    WORKDIR c
    RUN pwd
则最终路径为 /a/b/c 。
### ONBUILD
格式为 ONBUILD [INSTRUCTION] 。

配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。

例如，Dockerfile 使用如下的内容创建了镜像 image-A 。

    [...]
    ONBUILD ADD . /app/src
    ONBUILD RUN /usr/local/bin/python-build --dir /app/src
    [...]
如果基于 image-A 创建新的镜像时，新的Dockerfile中使用 FROM image-A 指定
基础镜像时，会自动执行 ONBUILD 指令内容，等价于在后面添加了两条指令。

    FROM image-A
    #Automatically run the following
    ADD . /app/src
    RUN /usr/local/bin/python-build --dir /app/src
使用 ONBUILD 指令的镜像，推荐在标签中注明，例如 ruby:1.9-onbuild 。

### 创建镜像
编写完成 Dockerfile 之后，可以通过 docker build 命令来创建镜像。

基本的格式为 docker build [选项] 路径 ，该命令将读取指定路径下（包括子
目录）的 Dockerfile，并将该路径下所有内容发送给 Docker 服务端，由服务端来
创建镜像。因此一般建议放置 Dockerfile 的目录为空目录。也可以通过
.dockerignore 文件（每一行添加一条匹配模式）来让 Docker 忽略路径下的目
录和文件。

要指定镜像的标签信息，可以通过 -t 选项，例如

$ sudo docker build -t myrepo/myapp /tmp/test1/

---
### docker machine
**作用：负责在多种平台上快速安装Docker环**

**支持多种后端驱动，包括虚拟机、本地主机和云平台**

curl -L https://github.com/docker/machine/releases/download/v0.3.1-rc1 /docker-machine_linux-amd64 > /usr/local/bin/docker-machine

chmod +x /usr/local/bin/docker-machine

---
### docker swarm 
**作用：管理集群**

docker pull swarm

查看版本

docker run --rm swarm -v  

---
### docker Kubernetes 
**作用 ：**
**在分布式系统中，部署，调度，伸缩是基础的功能**


