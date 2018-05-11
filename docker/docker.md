# 开始
tuyajoker
joker!docker
# 第一部分 概览和安装
1.创建docker环境
2.新建一个镜像并将其作为一个容器运行
3.将应用运行在多个容器上
4.将应用发布至集群上
5.通过添加一个后台数据库来叠加服务
6.将应用发布至生产环境

>
docker是一个开发人员和管理用于开发、发布、运行应用的容器
利用linux容器进行应用发布也被叫做容器化
容器不是新东西 但是用于发布应用却非常便捷

>
容器化越发流行，因为容器具备以下特点：
1.弹性，非常复杂的应用也可以容器化
2.轻量，容器共享主机内核
3.可变换，你可以在云端发布更新和升级
4.便携，本地构建，云端发布，任何地方都可以运行
5.可伸缩，你可以增加和自动发布容器的拷贝
6.可叠加，可以在云端垂直叠加服务

## 镜像和容器

通过运行一个镜像来启动容器，一个镜像就是一个包含所有环境依赖、运行环境、类库、环境变量、配置文件的包
一个容器就是一个运行状态的镜像实例，当镜像被执行时加载到内存中（有状态的镜像、一个用户进程）
你可以查看正在运行的容器实例，docker ps ,就行你会在linux中做的一样

## 容器和虚拟机
容器在linux本地运行并且和其他容器共享主机系统内核。它单独运行为一个进程，相比其他可执行，无需额外内存，使其非常轻量化

相反的，一个虚拟机运行一个完全暴露的客户操作系统并通过超级XXX虚拟获取主机资源。
通常，虚拟机提供了一个尽可能多的资源，通常大多数应用用不到

## 准备一个docker环境
下载一个维护版本（社区版 专业版）

## 测试docker版本
运行 docker --version， 确保你正在使用一个维护中的版本

运行 docker info (或者 docker version 没有--)查看更多docker详细信息

> 
为了避免权限错误（避免 sudo 的使用），将你的用户添加至docker用户组

## 测试docker安装

1.测试安装是否正常工作可以通过运行一个简单的docker镜像，hello-world
docker run hello-world

2.列出本机所有已下载的 hello-world 镜像  

```bash
docker image ls
```
3.列出已存在的hello-world容器（通过镜像衍生出来的）并展示它的信息。如果容器仍在运行，无需使用 --all 参数 

```bash
docker container ls --al
```

## 总结和命令

```bash
## List Docker CLI commands
## 列出所有 docker 命令
docker
docker container --help

## Display Docker version and info
## 显示 docker 版本和信息
docker --version
docker version
docker info

## Execute Docker image
## 运行docker镜像
docker run hello-world

## List Docker images
## 列出所有docker镜像
docker image ls

## List Docker containers (running, all, all in quiet mode)
## 列出所有 docker 容器 （正在运行的，所有的、静态模式下所有的）
docker container ls
docker container ls --all
docker container ls -aq
```

## 第一章总结
容器化让CI/CD更加简单，例如：
1.应用没有系统依赖
2.更新可以被推送到已发布的应用的任何部分
3.资源使用密度可以被提高

# 第二部分  容器

## 预备
1. 下载1.13版本或更高
2. 阅读第一部分
3. 快速环境测试，确保docker已安装  

```bash
docker run hello-world
```

## 介绍
是时候使用docker来构建一个app了，我们开始构建如下的一个最底层的app
此应用需要包含本页，以上这个层面是一个服务，这个服务定义了容器在产品中如何表现（活动、运行）
这部分将在第三部分提到。最后，在栈的最顶层定义了所有服务如何交互，这部分将在第五部分提到
1.stack
2.service
3.container(当前部分)

## 你的新开发环境
在过去，如果你准备开始写一个python app ,你当务之急是在你的机器上下载一个python环境，但是，
这个你机器需要的完美执行环境总是出现各种情况，并且总是需要和生产环境区适配

通过docker 你可以抓取一个可移动的python执行环境作为镜像，没有任何必须的安装。
然后你的构建版本中就会有一个基础的python镜像，并且和代码在一起，用以保证你的app的依赖、执行环境。
这些可移动的镜像被定义成一个叫Dockerfile的文件。

## 定义一个包含dockerfile的容器
dockerfile定义了容器内环境将要出现的东西。像网络接口、磁盘驱动这些资源获取是以虚拟化的形式出现在这个环境中，这个环境与
其他系统环境是隔离的，所有你需要映射内部端口到外部环境，对于需要拷贝到那个环境的文件需要具体化。
做完这些事之后，你通过dockerfile所构建的app将表现的一致，无论在哪里运行

## dockerfile
创建一个空目录，进入这个新目录，创建一个叫Dockerfile的文件。
复制粘贴一下文本到这个新文件中，保存。
注意用于解释每一个语句的注释。

```bash
# Use an official Python runtime as a parent image
# 使用一个官方python 运行环境作为当前镜像
FROM python:2.7-slim

# Set the working directory to /app
# 设定工作目录
WORKDIR /app

# Copy the current directory contents into the container at /app
# 将当前目录内容拷贝至 /app 容器下
ADD . /app

# Install any needed packages specified in requirements.txt
# 下载requirement.txt涉及的依赖
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
# 对外暴露容器的80端口
EXPOSE 80

# Define environment variable
# 定义环境变量 NAME 为 world
ENV NAME World

# Run app.py when the container launches
# 容器运行时启动 app.py
CMD ["python", "app.py"]
```

这个dockerfile 涉及了几对仍没有创建的文件，分别叫 app.py requirements.txt
接下来让我们来创建这些文件

## app本身

创建两个额外的文件，requirements.txt 和 app.py ，然后将他们放到dockerfile的同级目录下，
这让我们的app更加完整，在你看来这可能相当简单。当以上dockfile被构建到一个镜像中后，
app.py 和 requirements.txt 已经存在，因为dockerfile里的 ADD 命令，由于EXPOSE命令，app.py的
输出也可以通过http获取了

### requirements.txt
Flask
Redis

### app.py
```py
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)

```

现在我们看见 pip install -r requirements.txt 下载flask和redis, app 打印出环境变量 NAME,
与 socket.gethostname()输出一致。最后，因为redis没有运行（因为我们仅仅下载了redis库,并没有下载redis本身 ）
我们应该预测到此处使用它会报错，并且产生错误信息

>
注意，在容器内部获取主机名会取回容器的id,就像一个正在执行程序的进程id一样 

就是这样，你系统上不需要python 或者任何 requirements.txt上的依赖，也不需要在系统上通过构建或者运行这个镜像去下载它们
这看起来并不像你已经真正下载安装了一个包含了Python和flask的环境，但是你已经有了

## 构建app
我们已经准备好去构建这个app,确保你仍然在目录顶层，ls后可以看到以下文件

```bash
$ ls
Dockerfile		app.py			requirements.txt
```

现在运行构建命令，这将构建一个 docker 镜像，通过使用 -t 标签创建一个友好的名字

```bash
docker build -t friendlyhello .
```

你创建的镜像呢。它位于你机器的本地镜像仓库

```bash
$ docker image ls

REPOSITORY            TAG                 IMAGE ID
friendlyhello         latest              326387cea398
```

>
linux 用户可能遇到的问题
DNS 设置
一旦你的web app运行后，代理服务器可能阻碍连接到达你的服务。如你的服务位于代理服务器的下一级，添加以下几行到你的
dockerfile,使用 ENV 命令为代理服务器声明 主机地址和端口

```bash
# Set proxy server, replace host:port with values for your servers
ENV http_proxy host:port
ENV https_proxy host:port
```

代理服务器设定
DNS 错误配置可能在使用pip的时候出现问题，你需要设置你的DNS 地址让pip正常的工作
你可能想要改变docker 守护进程的DNS设置。
你可以在 /etc/docker/daemin.json 下编辑（或者创建）一个配置项 'dns',像下面这样
```json
{
  "dns": ["your_dns_address", "8.8.8.8"]
}
```
在上面的案例中，列表中的第一个元素是你自己的DNS服务器，第二项是谷歌的DNS服务器，此项将在第一项不可用时作为备用

继续运行之前，保存daemon.json 并且重启docker服务

```bash
sudo service docker restart
```
一旦修复完成，重新运行build命令进行构建

## 运行app
运行app,映射本地机器的4000端口到容器的发布端口80

```bash
docker run -p 4000:80 friendlyhello
```

应该在 http://0.0.0.0:80 查看Python服务信息 但是这个信息来自于内部容器并且不知道你映射了容器端口80到本地4000
使用正确的url http://localhost:4000

>
注意：如果你正在win7上使用Docker Toolbox ，使用docker的机器ip代替localhost
举个栗子：http://192.168.99.100:4000/
使用 docker-machine ip 获取docker machine ip

你也可以在shell中使用 curl 命令来查看同样的内容

```bash
$ curl http://localhost:4000

<h3>Hello World!</h3><b>Hostname:</b> 8fc990912a14<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
```

这里的端口映射 4000:80 是用来示范dockerfile中EXPOSE暴露的端口和publish时
使用docker run -p 之间的区别的
在后面的步骤中，我们将映射本机的80端口到容器80端口
然后使用 http://localhost 访问

敲 ctrl+c 退出

>
在 windows 上 ，彻底地关掉容器
在win上 ，ctrl+c 不会停止容器，先ctrl+c 退出（或者重开一个shell）
然后 docker container ls 列出所有正在运行的容器，紧接着 docker container stop [ name | dockerid]
来停止容器。
否则，当你运行下一步（重启docker）你可能会得到一个来自于守护进程的错误  

现在我们在后台以detach模式启动这个app

```bash
docker run -d -p 4000:80 friendlyhello
```

你将得到一个很长的容器id,然后退回到终端。你的容器正在后台运行。
你也可以通过 docker container ls 看到一个缩写的id(长短id在运行命令时可以相互使用)

```bash
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED
1fa4ab2cf395        friendlyhello       "python app.py"     28 seconds ago
```
注意，容器id与http://localhost:4000的内容相匹配

现在使用 container id （容器id）和 docker container stop 命令来终止一个进程

```bash
docker container stop 1fa4ab2cf395
```

## 分享你的镜像
为了展示刚刚我们创建的镜像的便携性，让我们上传我们构建的镜像，然后在其他某些地方运行
在这之后，当你需要发布容器到生产环境时，你需要理解如何推送至仓库。

仓库是一个repo集合，一个存放点是一个镜像集合---和gayhub很相似啦，除了代码已经构建。
一个账号可以创建很多个repo。 docker命令行工具使用docker的官方仓库作为默认仓库

>
注意，我们使用docker官方仓库，仅仅是因为它免费的和预配置的，但是仍然有很多开放仓库可供选择，
你甚至可以设置使用你的私有仓库，通过Docker Trusted Registry

## 使用Docker ID 进行登录
如果你没有docker账户，在cloud.docker.com上注册一个，记住你的用户名
在本地机器上登录docker公开仓库
```bash
docker login
```

## 标记镜像
用来关联本地的镜像和仓库项目的标记是 username/repository:tag 这个tag是可选的，但是我们强烈建议使用
因为这是仓库镜像版本管理的一个标志。
运行 docker tag image 并且带上你的 用户名、仓库、和tag,这样就可以上传镜像到你想要上传的地方了

```bash
docker tag image username/repository:tag

##示例
docker tag friendlyhello john/get-started:part2

##查看镜像
docker image ls

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
friendlyhello            latest              d9e555c53008        3 minutes ago       195MB
john/get-started         part2               d9e555c53008        3 minutes ago       195MB
python                   2.7-slim            1c7128a655f6        5 days ago          183MB
...
```

## 发布镜像 
发布你已经tag的镜像到仓库
```bash
docker push username/repository:tag
```

一旦发布成功，这次发布将是公开可用的。如果你登录 docker hub 你将会看到一个新镜像

## 从远程仓库拉取一个镜像并运行
到现在为止，你可以使用docker run 在任何机器上运行你的app

```bash
docker run -p 4000:80 username/repository:tag
```
如果这个镜像无法在本地获取，docker将会从远程仓库pull下来

```bash
$ docker run -p 4000:80 john/get-started:part2
Unable to find image 'john/get-started:part2' locally
part2: Pulling from john/get-started
10a267c67f42: Already exists
f68a39a6a5e4: Already exists
9beaffc0cf19: Already exists
3c1fe835fb6b: Already exists
4c9f1fa8fcb8: Already exists
ee7d8f576a14: Already exists
fbccdcced46e: Already exists
Digest: sha256:0601c866aab2adcc6498200efd0f754037e909e5fd42069adeff72d1e2439068
Status: Downloaded newer image for john/get-started:part2
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
```

无论 docker run 在哪里执行，它将会拉你的镜像，同时还有python 和 requirements.txt中的依赖
然后运行你的代码。所有依赖会被整体拉下来，运行docker，不需要在主机上安装任何东西

## 第二部分总结
下一部分将学习如何设置一个运行容器的可伸缩性

## 命令集

```bash

docker build -t friendlyhello .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyhello  # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyhello         # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a             # List all containers, even those not running
docker container stop <hash>           # Gracefully stop the specified container
docker container kill <hash>         # Force shutdown of the specified container
docker container rm <hash>        # Remove specified container from this machine
docker container rm $(docker container ls -a -q)         # Remove all containers
docker image ls -a                             # List all images on this machine
docker image rm <image id>            # Remove specified image from this machine
docker image rm $(docker image ls -a -q)   # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry

```


# 第三部分  服务

## 准备工作
1.下载1.13版本或以上
2.安装
3.阅读第一第二部分
4.确保你已经发布了friendhello镜像
5.确保你的镜像与预期一致，运行以下命令
docker run -p 80:80 username/repo:tag
访问 http://localhost

## 介绍
第三部分，我们将会弹性设置应用，以实现负载均衡。实现这一点，需要在发布应用的基础上提升一个等级

## 关于服务
在一个已发布的应用中，app的不同组成部分称之为“服务”，举个栗子，你假想一个视频分享网站，当用户上传点什么的时候，在后台，它可能包含一个请求数据存储服务，
一个视频转码服务，一个用于前后交互的服务等等。

服务仅仅是“产品中的容器”，一个服务仅仅运行一个镜像，但它决定镜像运行的规则（端口的使用、实例数量），服务具备这些必须的功能等等
伸缩服务将会改变软件某一部分容器的运行实例数量，为该服务获取更多计算资源
幸运的是，通过docker平台很好定义，运行，伸缩服务---只需要写一个docker-compose.yml

## 年轻人的第一个docker-compose.yml

docker-compose.yml是一个YAML格式的文件，用于定义docker容器在生产环境中如何运行。

### docker-compose.yml
将以下文件存为docker-compose.yml ，放在任何你想放的地方。但是必须保证你已经将你创建的镜像推送到仓库，
替换此文件中的 username/repo:tag 为你自己镜像的信息

```yaml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
    networks:
      - webnet
networks:
  webnet:
```
年轻人的第一个docker-compose.yml 告诉docker按以下方式执行

1. 拉下我们在第二部分创建的镜像
2. 运行5个此镜像的实例，并命名为web,限制每个实例最多使用不超过10%的cpu(每个核心)，50M内存
3. 立即重启容器，如果其中一个失败
4. 将主机80端口映射到web的80端口
5. 委托web容器通过网络负载均衡分享80端口，称之为webnet(在内部，容器通过一个短端口发布至80端口)
6. 使用默认设置定义 webnet 网络（一个负载网络 ****）

## 运行新的负载均衡app

在我们可以使用 docker stack depoly 命令，我们可以先运行 
```bash
docker swarm init
```
>
注意：我们将在第四部分涉及以上命令的意义，如果你不运行docker swarm init
你将触发一个“this node is not a swarm manager”的错误。

现在 让我们来运行它。你需要给你的app一个name 这里我们设置成 getstartedlab

```bash
docker stack deploy -c docker-compose.yml getstartedlab
```

在单一主机上，我们的单任务栈一次运行5个已发布镜像的容器实例

通过以下命令获取在我们的应用中 服务ID(service ID)
```bash
docker service ls
```

查看web服务的输出，前置上你app的name 如果你将他命名为跟本案例一样的name,那么name是
getstartedlab_web
服务ID也会被列出来，还有实例的数量，镜像名称，对外暴露的端口

一个单独的容器在服务中叫task,task有一个唯一id,数字上递增，与docker-compose.yml中的实例数量相关联

列出某服务下所有的task

```bash
docker service ps getstartedlab_web
```

当你列出当前系统上所有container时，也会出现，当然，此处列出的容器并非按service筛选的
```bash
docker container ls -q
```

你可以间歇性运行 curl -4 http://localhost ，或者在浏览器内间隙性刷新
两种方式返回的 container ID 都会变化，示例负载均衡
每一个请求，5个 tasks 里面的某一个task会在轮训中被选中，响应请求。
返回的container ID 与 命令列出的 ID 是一致的 （docker container ls -q)

>
在win10中运行
win10 powershell 中curl已经可用，如果没有，可以下一个 Git bash 或者 下一个 wget for win

>
响应过慢
受到你运行环境的网络配置的影响，从请求到容器响应可能需要30s,这并不能反应docker 或者 swarm 的性能。
除了未配置
的redis依赖（后续会增加）。现在访客计数功能由于同样的原因还不能工作。我们仍未添加数据持久化服务。

## app 服务伸缩
通过配置 docker-compose.yml中的 replicas项的值，重新运行 docker stack deploy 命令

```bash
docker stack deploy -c docker-compose.yml getstartedlab
```

docker 以平滑的方式更新，无需停止整个栈或者杀掉container进程

现在重新运行 docker container ls -q 会看到发布的实例重新配置了，如果你扩容了实例，因此会有更多tasks,econtainer开启

## 关闭 app 和 swarm 
1.使用 docker stack rm 关闭 app

```bash
docker stack rm getstartedlab
```

2.关闭 swarm

```bash
docker swarm leave --force
```

使用docker开启或者扩容一个应用很简单。你已经跨出了学习如何在生产环境中运行一个容器的一大步。
下一步，你要学习如何在docker机器集群上运行将此服务集群化


## 命令
```bash
docker stack ls                                            # List stacks or apps
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker service ls                 # List running services associated with an app
docker service ps <service>                  # List tasks associated with an app
docker inspect <task or container>                   # Inspect task or container
docker container ls -q                                      # List container IDs
docker stack rm <appname>                             # Tear down an application
docker swarm leave --force      # Take down a single node swarm from the manager
```

## 第四部分集群

### 预备
1.2.3.4和前面一样

5.确保你的镜像上传到仓库了
6.确保容器按预期运行
7.有一份docker-compose.yml拷贝

## 介绍
//todo

## 理解集群
一个集群就是一组运行docker的机器，并且加入了一个群组。当这一切发生后，你仍然像以前运行docker
命令。但是现在它们被一个集群管理器在群组中执行。集群中的机器可以是实体机器或者虚拟机。当我们加入一个集群后，它们将关联到一个节点。

集群管理器会使用多种策略去运行容器，列如“空白节点”--使用容器最小限度地填充未使用的机器
或者“全局”---确保每个机器可以获得一个完整的容器实例。
就像你已经使用的compose文件，你也是这样去配置swarm manager的运行策略

集群中只可以使用swarm manager去执行命令。或者授权其他机器加入这个集群作为一个worker(类似主从)
workers仅仅提供功能服务，无法调度机器去做什么或不做什么

至此，你已经在本地机器上使用单主机模式运行docker。但是docker也可以切到集群模式，这种模式支持集群的使用。
开启集群模式将直接在当前机器上建立一个swarm manager。此后，docker将在整个集群中运行你执行的命令而
不仅仅是当前机器

## 设置你的集群
一个集群有多个节点组成，节点既可以是物理的也可以是虚拟机。基础概念足够简单
运行 docker swarm init 开启集群模式 ，并为当前机器创建一个 swarm manager
然后在其他机器上运行 docker swarm join 以 wokers 的身份加入集群

在下面面板选择查看如何在不同的环境中运行（win10以外的系统和win10）
我们使用虚拟机快速创建一个两台机器组成的集群，然后切换到集群模式。

你需要一个能创建虚拟机的玩意儿 virtualbox  


>注意，如果机器上有Hyper-v 就不要下载了

现在使用 docker-machine 创建一组虚拟机

```bash
docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2
```
### 列出虚拟机，拿到ip
现在有两个虚拟机被创建了 分别叫 myvm1 myvm2

使用以下命令列出机器，拿到ip

```bash
docker-machine ls
```

以下是命令输出示例：
```bash
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.2-ce   
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.06.2-ce   
```

## 初始化集群并添加节点
第一个机器用作管理员，用来执行管理命令和管理wokers加入集群的权限
第二个机器就是一个worker

你可以使用docker-machine ssh发送 docker swarm init 命令到虚拟机，配置myvm1作为swarm manager
输出如下所示

```bash

$ docker-machine ssh myvm1 "docker swarm init --advertise-addr <myvm1 ip>"
Swarm initialized: current node <node ID> is now a manager.

To add a worker to this swarm, run the following command:

  docker swarm join \
  --token <token> \
  <myvm ip>:<port>

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

```

>2376和2377端口
运行 docker swarm init 和 docker swarm join 务必使用2377端口（集群管理端口）
或者干脆不要使用端口，让它使用默认配置

使用 docker-machine ls 返回的ip信息中包含2376端口，这个是docker守护进程的端口。
不要用，有坑在召唤。


>使用SSH有问题的话可以尝试一下 --native-ssh 标志
docker machine允许自行配置使用系统的ssh
如果因为某些原因导致命令发送至 swarm manager出现问题
使用ssh时加上 --native-ssh标志位就完事了

```bash
docker-machine --native-ssh ssh myvm1
```
现在你能看到，docker swarm init 返回的信息中包含了一个让你可以在任何节点添加worker的预配置 docker swarm join命令。复制这个命令，通过docker-machine ssh 发送到myvm2 ,加入一个新的 worker

```bash
$ docker-machine ssh myvm2 "docker swarm join \
--token <token> \
<ip>:2377"

This node joined a swarm as a worker.
```

恭喜，你创建了你的第一个集群

在 docker manager上运行 docker node ls 查看集群上的节点

```
docker-machine ssh myvm1 "docker node ls"

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
brtu9urxwfd5j0zrmkubhpkbd     myvm2               Ready               Active
rihwohkh3ph38fhillhhb84sk *   myvm1               Ready               Active              Leader
```

>离开集群
如果你想开始结束，你可以在每个节点上运行 docke swarm leave 


## 将应用发布至集群

最难的部分结束了，现在你可以重复第三部分在新集群上发布
记住，只有像 myvm1 这种 swarm managers 可以执行docker commands;workers仅仅是执行

### 为 swarm manager 配置 docker-machine shell
至此，你已经将docker command 包裹在 docker-machine ssh 去告知虚拟机。
另一个选项就是运行
docker-machine env <machine> 获取并运行一个命令，配置当前shell，然后告知vm上的 docker 守护进程
这种方式在下一步种运行的更好，因为它允许你使用本地的 docker-compose.yml 文件去发布应用到远端，而不需要
到处复制

输入 docker-machine env myvm1 , 然后复制粘贴输出中最后一行运行，配置shell告知myvm1(swarm manager)
配置shell的命令在不同系统上有所不同

### linux 和 mac
运行 docker-machine env myvm1 获取配置shell的命令，并告知 myvm1

```bash
$ docker-machine env myvm1
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/sam/.docker/machine/machines/myvm1"
export DOCKER_MACHINE_NAME="myvm1"
# Run this command to configure your shell:
# eval $(docker-machine env myvm1)
```
运行给出的命令去配置shell,并告知myvm1

```bash
eval $(docker-machine env myvm1)
```

运行 docker-machine ls 验证 myvm1 现在是属于激活状态的，旁边有星号标示

```bash
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   *        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.2-ce   
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.06.2-ce   
```

## 在 swarm manager 上发布应用
现在你有了myvm1,你可以使用它作为 swarm manager 去发布你的 app,同样是
使用第三部分提到的的 docker stack deploy 命令 和 你本地的 docker-compose.yml
副本。这个命令可能需要几秒钟去完成，并且整个部署需要持续一些时间才可用。
在 swarm manager 使用docker service ps <服务名> 命令
去验证你所有服务已经被重写发布了

通过使用 docker-machine 的shell配置你可以连接到 myvm1 。你仍然具有获取本机文件的
权限。确保你仍和 docker-compose.yml 在同一级目录 

和之前一样，运行以下命令发布app到myvm1

```bash
docker stack deploy -c docker-compose.yml getstartedlab
```

就是这样，这个应用已经被发布到集群上了

>注意：如你你的镜像被存储在私有仓库上而不是docker hub上你需要使用docker login <仓库>
然后需要给以上命令添加 --with-registry-auth flag
```bash
docker login registry.example.com

docker stack deploy --with-registry-auth -c docker-compose.yml getstartedlab

```
这会使用加密的WAL(Write ahead log)将本地的登录口令传递到服务部署的集群节点。
获取到这些信息，节点就可以登录到仓库，并拉下镜像

现在你可以使用第三部分同样的命令，但是这次你会发现services(关联了容器)被分发到myvm1 和 myvm2

```
$ docker stack ps getstartedlab

ID            NAME                  IMAGE                   NODE   DESIRED STATE
jq2g3qp8nzwx  getstartedlab_web.1   john/get-started:part2  myvm1  Running
88wgshobzoxl  getstartedlab_web.2   john/get-started:part2  myvm2  Running
vbb1qbkb0o2z  getstartedlab_web.3   john/get-started:part2  myvm2  Running
ghii74p9budx  getstartedlab_web.4   john/get-started:part2  myvm1  Running
0prmarhavs87  getstartedlab_web.5   john/get-started:part2  myvm2  Running

```

>通过 docker-machine env 和 docker-machine ssh
1.需要配置你的shell和不同的 machine 取得联系，只要在同一个或者不同的shell中简单运行 docker-machine env 
然后 运行给定的命令 指向myvm2。对于当前的shell,这些都是标准化的。
如果使用一个未配置或者新的shell ，你需要重新运行这些命令。
使用 docker-machine ls 列出所有机器，查看状态和ip，取到ip，查看你连接的是哪一个机器
2.或者，你可以将docker command 以 docker-machine ssh <machine> "command" 的形式包裹起来，
这种方式可以直接连接vm,但是不具备直接获取本机文件的权限
3.在mac或者linux 上，你可以直接使用 docker-machine scp <file> <machine>:~ 实现机器间的文件转移，
win上需要类似于Git bash 这种 linux终端仿真工具
这些示例demo，包括docker-machine ssh 和 docker-machine env ,因为docker-machine CLI 在各个平台上都是可用。

## 访问集群

你可以通过 myvm1 或 myvm2 的ip地址访问到应用
你创建的网络可以被它们共享并且均衡负载。运行 docker-machine ls
获取虚拟机的ip地址，在流浪器访问其中任意一个ip,点击刷新

这里可能有5个container ID 随机循环，用于示例负载均衡

 //todo



## 迭代和伸缩应用
从这里开始，你可以使用任何你在第二部和第三部学习到知识做任何事情

通过修改 docker-compose.yml 伸缩app
改变应用逻辑，修改代码，然后重新构建镜像，然后上传新镜像
与此同时，简单重新运行 docker stack deploy 去发布这些改变。
你可以加入任何机器，物理设备或者虚拟机到这个集群。
使用你用在myvm2上同样的 docker swarm join 命令，把功能加到集群。
然后运行 docker stack deploy ,然后你的应用就可以使用新的资源了

## 清除和重启
你可以使用 docker stack rm  关闭一个stack

```bash
docker stack rm getstartedlab
```

>保留或者删除集群
某个节点后，如果你想删除这些集群，可以使用
docker-machine ssh myvm2 "docker swarm leave"(在worker上使用)
docker-machine ssh myvm2 "docker swarm lerave --force"（在manager上使用）
第五部分还需要用，暂时不要删除‘


## 去掉之前 docker-machine shell 的环境变量设置

mac和linux 上
```bash
eval $(docker-machine env -u)
```
win上
```bash
  & "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env -u | Invoke-Expression
```

这会与docker-machine 创建的虚拟机shell 断开连接，并且仍然允许你在同一个shell下工作
现在使用本地 docker 命令

## 重启 docker machines
如果你停止本地服务，docker machines 同样停止运行。你可以使用 docker-machine ls
获取这些 machines 的状态信息

```bash
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL   SWARM   DOCKER    ERRORS
myvm1   -        virtualbox   Stopped                 Unknown
myvm2   -        virtualbox   Stopped                 Unknown
```

重启停止的machine:
```bash
docker-machine start <machine-name>
```
例如：
```bash
$ docker-machine start myvm1
Starting "myvm1"...
(myvm1) Check network to re-create if needed...
(myvm1) Waiting for an IP...
Machine "myvm1" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.

$ docker-machine start myvm2
Starting "myvm2"...
(myvm2) Check network to re-create if needed...
(myvm2) Waiting for an IP...
Machine "myvm2" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env`
```


## 命令集
```bash
docker-machine create --driver virtualbox myvm1 # Create a VM (Mac, Win7, Linux)
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1 # Win10
docker-machine env myvm1                # View basic information about your node
docker-machine ssh myvm1 "docker node ls"         # List the nodes in your swarm
docker-machine ssh myvm1 "docker node inspect <node ID>"        # Inspect a node
docker-machine ssh myvm1 "docker swarm join-token -q worker"   # View join token
docker-machine ssh myvm1   # Open an SSH session with the VM; type "exit" to end
docker node ls                # View nodes in swarm (while logged on to manager)
docker-machine ssh myvm2 "docker swarm leave"  # Make the worker leave the swarm
docker-machine ssh myvm1 "docker swarm leave -f" # Make master leave, kill swarm
docker-machine ls # list VMs, asterisk shows which VM this shell is talking to
docker-machine start myvm1            # Start a VM that is currently not running
docker-machine env myvm1      # show environment variables and command for myvm1
eval $(docker-machine env myvm1)         # Mac command to connect shell to myvm1
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression   # Windows command to connect shell to myvm1
docker stack deploy -c <file> <app>  # Deploy an app; command shell must be set to talk to manager (myvm1), uses local Compose file
docker-machine scp docker-compose.yml myvm1:~ # Copy file to node's home dir (only required if you use ssh to connect to manager and deploy the app)
docker-machine ssh myvm1 "docker stack deploy -c <file> <app>"   # Deploy an app using ssh (you must have first copied the Compose file to myvm1)
eval $(docker-machine env -u)     # Disconnect shell from VMs, use native docker
docker-machine stop $(docker-machine ls -q)               # Stop all running VMs
docker-machine rm $(docker-machine ls -q) # Delete all VMs and their disk images
```