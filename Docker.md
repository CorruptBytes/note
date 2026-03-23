# 概述

> Docker 是一套用于构建、分发和运行容器的工具平台。它通过镜像的方式将应用及其运行环境打包，使应用程序可以在任何具有Linux内核的计算机上迁移，而不用在意操作系统。

- 镜像(Image)

  > Docker将应用程序及其所需的依赖、函数库、环境、配置等文件打包在一起，称为镜像。通过镜像可以制作容器。镜像是仅可读的，通过镜像创建容器时，容器会从镜像复制文件，形成自己的运行环境。

- 容器(Container)

  > 镜像中的应用程序运行后形成的进程就是**容器**，Docker会给容器做隔离，对外不可见。每个容器都是独立的运行环境，具有自己独立的文件系统。这样，就可以为每个应用程序封装独立的运行环境，且每个运行环境互不干扰。

  **容器的文件系统结构类似于操作系统的文件系统，但是只保留了容器运行所必须的文件**

- 宿主机

  > 运行容器的计算机被称为宿主机

- Docker仓库

  > 镜像的托管平台，可以从中拉取镜像，又称为Docker Registry。

  ```shell
  #Docker官方镜像仓库:DockerHub
  hub.docker.com
  ```

## 架构

> Docker是一个CS架构的程序，由两部分组成:

- 服务端(server)：Docker守护进程(docker daemon)，负责处理Docker指令，管理镜像、容器等
- 客户端(client)：通过命令或RestAPI向Docker服务端发送指令。可以在本地或远程向服务端发送指令

![image-20250825111744343](D:\笔记\图片\image-20250825111744343.png)

## 安装

> Docker分为CE与EE两个版本，CE为社区版，面向个人，免费。EE为企业版，面向企业，收费。

**Ubuntu系统**

```
sudo apt install docker.io
```

**Centos系统**

1. 下载Docker依赖

   ```
   sudo yum install -y yum-utils device-mapper-persistent-data lvm2
   ```

2. 配置Docker的yum镜像源

   ```bash
   sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   
   sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo
   ```

3. 更新yum，建立缓存

   ```
   sudo yum makecache fast
   ```

4. 安装Docker

   ```
   yum install -y docker-ce
   ```

**配置镜像源**

```bash
# 创建目录
mkdir -p /etc/docker

# 复制内容
tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "http://hub-mirror.c.163.com",
        "https://mirrors.tuna.tsinghua.edu.cn",
        "http://mirrors.sohu.com",
        "https://ustc-edu-cn.mirror.aliyuncs.com",
        "https://ccr.ccs.tencentyun.com",
        "https://docker.m.daocloud.io",
        "https://docker.awsl9527.cn"
    ]
}
EOF

# 重新加载配置
systemctl daemon-reload

# 重启Docker
systemctl restart docker
```

## 卸载

**Centos系列系统**

```bash
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  docker-ce
```

# 镜像

## 组成

> Docker镜像是一个分层结构，一个Docker镜像由多个只读层(Layer)加镜像元数据(Metadata)构成。

**Base Image(基础镜像层)**

这是镜像的最底层，用于提供最基本的运行环境，包含基本的系统函数库、环境变量、文件系统。

- `Base Image`本身也是一个镜像，它通常不依赖其他镜像。

**Image Layers(镜像层)**

在`Base Image`的基础上，逐层为镜像添加的依赖与程序。可以添加多层，每一层对应一次构建操作，本质是 **只读文件系统的增量快照**。

 **Image Metadata(镜像元数据)**

- BaseImage层：
- Entrypoint：入口，是镜像中应用启动的命令
- 其它：在BaseImage基础上添加依赖、安装程序、完成整个应用的安装和配置

## 命名规范

```
[repository]:[tag]
```

- `repository`指镜像库，也可以理解为软件名，他存放了一个镜像的不同版本。
- `tag`指版本,如果不指定`tag`,则默认`是latest`，代表最新版本的镜像

## 自定义镜像

Docker可以根据`DockerFile`文件构建自定义镜像。编写好`DockerFile`文件后，通过以下命令根据此`DockerFile`文件构建镜像：

```
docker build [-t myImage:1.0] [-f Dockerfile] PATH | URL
```

- `-t`设置镜像名
- `-f DockerFilePath`:指定Dockerfile文件相对于上下文目录的位置。如果 Dockerfile 名字就是 `Dockerfile` 且在上下文目录根，可以省略 `-f`选项
- `Path | URL`为构建镜像的上下文目录，Docker 会把上下文目录 **整个发送给 Docker Daemon**，因此上下文目录中要包含 COPY 需要的文件和`Dockerfile`文件。如果上下文目录为执行`build`的目录，则可以写`.`表示当前目录

### `Dockerfile`

Dockerfile 是一个用来描述如何构建 Docker 镜像的脚本文件，Docker 根据其中的指令，**逐条构建镜像层（Image Layers）并生成镜像元数据（Metadata）**。

- `Dockerfile`文件中支持`#`添加注释。

<h4>常用指令</h4>

`DokcerFile`的指令可分为两类：一类会生成镜像层；一类用于指定元数据。

**会生成镜像层的指令：**

<h5><code>FROM</code></h5>

指定基础镜像层(Base Image)。

```
FROM centos:6
```

- 必须是`DockerFile`的第一条指令
- 一个 Dockerfile 可以有多个 `FROM`（多阶段构建）

<h5><code>RUN</code></h5>

在构建阶段执行`RUN`指定的命令，会生成新的镜像层，常用于安装依赖、构建项目。

```shell
RUN apt update && apt install -y curl
```

`RUN`命令可以使用`&&`连接多个命令，这样只会生成 **一个镜像层**

```shell
RUN apt-get update \
 && apt-get install -y curl vim \
 && rm -rf /var/lib/apt/lists/*
```

-  任一步失败，构建立即失败
- 最终层不包含中间垃圾文件
- 可以配合`\`换行增强可读性

<h5><code>COPY</code></h5>

拷贝本地文件到镜像的指定目录，生成新的镜像层

```
COPY target/app.jar /app/app.jar
```

<h5><code>ADD</code></h5>

相比于`COPY`指令，`ADD`支持解压、使用URL下载文件等功能。会生成新的镜像层。

- 一般不推荐使用，能用 `COPY` 就别用 `ADD`。

**为镜像添加元数据的指令：**

<h5><code>ENV</code></h5>

设置环境变量，可以后面的指令使用。

```
ENV JAVA_OPTS="-Xms256m -Xmx512m"
```

- 后面的命令可以通过`$环境变量名`使用环境变量。

<h5><code>EXPOSE</code></h5>

**声明**容器使用的端口

```
EXPOSE 8080
```

- `EXPOSE` 本身并不会开放端口，也不会做端口映射，只是向“使用者 / 工具”声明：容器内部服务期望使用这个端口。

<h5><code>ENTRYPOINT</code></h5>

定义**固定启动命令**，在容器启动时调用

```
ENTRYPOINT ["java","-jar","app.jar"]
```

- 一般不被覆盖

<h3>Java应用镜像</h3>

Docker仓库中提供了一些基础镜像，提供了不同`Java`版本的运行环境，可以直接在这些基础镜像的基础上构建。

```dockerfile
# 使用官方 JRE 作为基础镜像
FROM openjdk:17-jre-slim

# 设置工作目录
WORKDIR /app

# 复制本地 jar 到容器中
COPY app.jar .

# 声明容器服务端口（可选）
EXPOSE 8080

# 启动 Java 程序
CMD ["java", "-jar", "app.jar"]
```



<h4>常用基础镜像</h4>

**OpenJDK 官方镜像**

镜像名为`openjdk`。

| 镜像                  | 说明               | 适合场景                   |
| --------------------- | ------------------ | -------------------------- |
| `openjdk:17-jdk`      | 完整 JDK 17        | 开发或编译应用             |
| `openjdk:17-jre`      | 只包含 JRE 17      | 生产运行环境               |
| `openjdk:17-jdk-slim` | 精简版 JDK         | 节省体积，开发/构建        |
| `openjdk:17-jre-slim` | 精简版 JRE         | 生产运行                   |
| `openjdk:17-alpine`   | 基于 Alpine 的 JDK | 极小体积，但有些库兼容问题 |
| `openjdk:17-oracle`   | Oracle 官方 JDK    | 对兼容性要求高的场景       |

# 命令

## 镜像相关

- 查看已有镜像

  ```
  docker images
  ```

- 删除镜像

  ```
  docker rmi id或镜像名字
  ```

- 推送镜像

  ```
  docker push
  ```

- 保存镜像为压缩包

  > 保存一个或多个镜像为tar类型的压缩包

  ```
   docker save [OPTIONS] IMAGE [IMAGE...]
  ```

  - `-o`|`--output`：输出的文件地址,如果不存在会自动创建。

- 加载压缩包为镜像

  > 从一个tar类型的压缩包中加载镜像

  ```
  docker load [OPTIONS]
  ```

  - `-i`|`--input`: 值为压缩包地址
  - `-q`|`--quiet`:无值，安静模式，不会打印日志

- 构建镜像

  ```
  docker build [OPTIONS] PATH | URL | -
  ```

  - `-t`：string，构建镜像的名称与版本

## 容器相关

- 创建并运行一个容器

  > 如果不存在镜像，会根据名字拉取一个镜像

  ```
  docker run [options] [image] [COMMAND] [ARG...]
  ```

  - `--name`:`string`,容器名称

  - `-p`:`宿主机端口:容器端口`,默认容器是对外隔离的，不可以访问，通过这个参数将宿主机端口与容器端口映射

  - `-d`：后台运行，不阻塞命令行

  - `-v`:`string`，`数据卷:容器目录`,将数据卷挂载到指定容器目录,如果数据卷不存在，会自动创建。这种方式称为数据卷挂载;`宿主机目录:容器目录`,将宿主机目录直接挂载到容器目录，这种方式称为目录挂载。可以配置多个`-v`选项同时挂载多个目录。也可以挂载文件。

  - 如果挂载时，挂载目录与容器目录均有文件
    - **挂载目录 → 外部目录优先，容器目录被隐藏**
    - **挂载文件 → 外部文件优先，容器文件被忽略**
    
  - `--env-file`:指定一个文件批量加载环境变量，文件内容为一行行 `KEY=VALUE`

  - `[COMMAND]`:容器启动时的主进程命令，如果不写，则使用容器默认设置的命令，如果写了，则会覆盖。

  - `--restart`:设置容器随守护进程启动时的重启策略

    |          可选值          |                           含义                            |
    | :----------------------: | :-------------------------------------------------------: |
    |            no            |        默认,容器退出后不重启，也不会随 Docker 启动        |
    |          always          |   无论退出状态如何，容器都会自动重启，并随 Docker 启动    |
    |      unless-stopped      |      容器随 Docker 启动，除非手动 `docker stop` 过它      |
    | on-failure[:max-retries] | 仅在容器异常退出（非 0 状态码）时重启，可设置最大重试次数 |

- 暂停/恢复容器运行

  ```
  docker pause [name]
  docker unpause [name]
  ```

  - 暂停时，容器进程会被操作系统挂起，容器关联内存暂存起来，CPU不再执行容器进程
  - 恢复后，内存恢复，进程继续运行

- 停止/开启运行容器

  ```
  docker stop
  docker start [OPTIONS] CONTAINER [CONTAINER...]
  ```

  - 停止时,容器进程被杀死，关联内存回收

- 重启容器

  ```
  docker restart CONTAINER
  ```

- 查看所有运行容器及状态

  ```
  docker ps [options]
  ```

  - `-a`:查看所有容器，默认只查看运行中的容器

- 查看容器信息

  ```
  docker inspect CONTAINER
  ```

- 查看容器运行日志

  ```
  docker logs [options] [name]
  ```

  - `-f`|`--follow`：跟踪日志输出

- 进入容器执行命令

  ```
  docker exec [options] CONTAINER COMMAND [ARG...]
  ```

  `-it`:给当前进入的容器创建标准输入输出终端，允许与容器交互

- 删除指定容器

  ```
  docker rm
  ```

## 数据卷相关

> 数据卷(volume)是一个虚拟目录，指向宿主机文件系统中的某个真实目录。数据卷默认指向宿主机`/var/lib/docker/volumes/{volume_name}/_data`目录，类似文件目录的别名。

- 将容器与数据分离，解耦合，方便操作容器内数据，保证数据安全
- 可以将数据卷与容器目录绑定在一起(通常称为挂载),两边的文件会相互影响

------

- 创建一个数据卷

  ```
  docker volume create [OPTIONS] [VOLUME]
  ```

- 显示一个或多个数据卷的信息

  ```
  docker volume inspect
  ```

- 列出所有的数据卷

  ```
  docker volume ls
  ```

- 删除未使用的数据卷

  > 默认只会删除匿名的未使用的数据卷

  ```
  docker volume prune 
  ```

  - `-a`|`--all`：删除所有未使用的数据卷
  - `-f`|`--force`：不需要手动确认

- 删除指定数据卷

  ```
  docker volume rm
  ```

  

- 查看版本

  ```
  docker --version #查看docker Client版本
  docker version #查看Docker Client 与Servier版本信息
  ```

- 拉取镜像

  ```
  docker pull registry/命名空间/镜像名:版本号
  ```

  - 具有选项`--platform=`用于选择最适合当前CPU架构的镜像，一般会自动选择，不需要写

  - registry为仓库注册地址，仓库注册地址为`docker.io`时，为官方仓库，可以省略不写
  - 命名空间为`library`时，为官方命名空间，可以省略不写
  - 如果不写版本号，表示获得最新版本的镜像
  - registry/命名空间/镜像名表示一个镜像库(repository)，他存放了一个镜像的不同版本

- 根据镜像创建一个容器并运行

  ```
  docker run 镜像名
  ```

  - `-d`:detached mode,分离模式,表示容器后台执行，不阻塞当前命令行
  - 如果本地不存在此镜像，会自动去仓库下载
  - `-p`:容器与宿主机的网络是隔离的，无法直接访问到，可以通过-p参数将容器端口与宿主机端口映射`宿主机端口:容器端口`
  - `-v`：绑定宿主机与容器的文件目录，两边对文件的修改都会相互影响，这种目录又被称为挂载卷,用于对数据持久化，因为容器删除时会删除所有数据。`宿主机目录:容器目录`。这种挂载目录的方式称为绑定挂载
  - `--name`:后跟为容器起的名字，必须是唯一的
  - `-it`:控制台进入容器进行交互
  - `--rm`:容器停止时立刻删除容器

- 查看容器进程状态

  ```
  docker ps
  ```

  > ps即process status

  - `-a`：默认只显示正在运行的容器，加上选项可以显示所有容器

- 删除容器

  ```
  docker rm 容器名
  ```

  - `-f`：强制删除正在运行的容器

- 命名挂载目录

  ```
  docker volume create 目录命名空间
  ```

  - 之后使用绑定挂载时，可以直接使用名字而不是真实的地址

- 查看命名挂载的真实目录

  ```
  docker volume inspect 命名
  ```

- 停止运行容器

  ```
  docker stop id/name
  ```

- 创建一个容器

  ```
  docker create 
  ```

- 运行一个容器

  ```
  docker start id/name
  ```

## 日志相关

- 查看容器日志

  ```
  docker logs [OPTIONS] CONTAINER
  ```

  - `-f`：持续输出日志

## 网络相关

- 创建一个网络

  ```
    docker network create NETWORK
  ```

- 查看所有网络

  ```
  docker network ls
  ```

- 将容器连接进网络

  ```
  docker network connect NETWORK CONTAINER
  ```

  

  


## 其他命令

- 查看帮助信息

  ```
  docker --help
  docker [command] --help
  ```

# DockerCompose

> Docker Compose可以基于Compose文件帮我们快速的部署分布式应用，而无需手动一个个创建和运行容器。Compose文件是一个文本文件，通过指令定义集群中的每个容器如何运行。

# K8S

Kubernetes(K8S) 是一个用于**自动化部署、扩展和管理容器化应用程序**的容器编排引擎，由`Google`研发并开源。

<h3>作用</h3>

随着容器化技术发展，越来越多的企业开始使用容器化技术来构建和部署自己的服务和应用程序。随着容器数量越来越多，容器的管理也愈发困难，`K8S`可以帮我我们优雅的管理容器化的应用程序和服务。

- 提供了容器编排功能，可以通过配置文件定义应用程序的部署方式，让容器的创建，维护和管理更加高效
- 增强系统的高可用性：提供了自动重启，自动重建，自我修复等功能，使得系统可以在长时间内持续正常地运行，并不会因为某个组件或服务的故障而导致整个系统不可用。
- 增强了系统的可扩展性：系统可以根据负载的变化动态的扩展或者所见系统的资源，提高系统的性能和资源的利用率。

<h3>核心概念</h3>

Kubernetes 基于对象模型进行设计，定义了多种资源对象，它们是Kubernetes 中对集群里一切可管理事物的抽象描述，由 **API Server 统一管理**，用户通过 **YAML / API 声明期望状态**，由 K8s 负责把**实际状态**调整到**期望状态**。

- 凡是能被 API Server 管理、能用 YAML 描述的，都是资源对象
- 每一类资源对象都有一个统一结构，描述其信息和状态，Kubernetes 通过控制器不断对比期望状态与实际状态，并进行调谐，使集群最终收敛到期望状态。

<h4>节点</h4>

一个节点就是一个物理机或者虚拟机，在一个节点上，可以运行多个`POD`或` servic`。

<h4><code>pod</code></h4>

一个pod就是一个或多个容器的组合，这些容器共享一个运行环境以及环境中的一些资源，比如网络，存储以及一些运行时的配置，pod是K8S的最小调度单元。

- 一般情况下，一个`pod`中建议只运行一个容器
- `pod`不是一个稳定的实体，非常容器被创建或销毁，比如发生故障时，k8s会自动将pod销毁，然后创建一个新的pod替代它。

<h4><code>Service</code></h4>

`Service`将一组`pod`封装为一个服务，可以通过一个统一的入口访问服务

Service根据其访问范围和用途有多类：

- `ClusterlP` 默认类型，集群内部的服务
- `NodePort` 节点端口类型，将服务公开到集群节点上
- `LoadBalancer` 负载均衡类型，将服务公开到外部负载均衡器上
- `ExternalName` 外部名称类型，将服务映射到一个外部域名上
- `Headless`  无头类型，主要用于DNS解析和服务发现

<h4><code>Ingress</code></h4>

用来管理集群外部访问集群内部服务的入口和方式。可以通过`Ingress`配置不同的转发规则，从而根据不同的规则访问集群内部的`Service`，还可以配置负载均衡，SSL证书等。

<h4>ConfigMap</h4>

它用于将配置信息封装起来，供应用程序读取和使用

<h4>secret</h4>

它用于将一些敏感信息封装起来，供应用程序读取和使用

# 小知识

## Docker容器与虚拟机的区别

> Docker容器之间共用同一个系统内核，而每个虚拟机都包含一个操作系统的完整内核。相比于虚拟机，Docker容器更轻，更小，启动速度更快

![image-20250820161747543](D:\笔记\图片\image-20250820161747543.png)

![image-20250820161805565](D:\笔记\图片\image-20250820161805565.png)

- docker是一个系统进程；
- 虚拟机是在操作系统中通过Hypervisor技术虚拟化计算机硬件，然后在这个计算机硬件上搭建操作系统

## WSL

> Windows Subsystem for Linux，Windows10系统带来的全新特性，使用WSL，可以以非常轻量化的方式，得到Linux系统环境，而不需要安装完整的虚拟机。并且获得的Linux环境完全直连计算机硬件，无需通过虚拟机虚拟硬件

- 安装

  ```
  wsl --install
  ```

  - 这条命令会自动安装 WSL2 和默认的 Ubuntu 发行版。
  - 安装完成后需要重启电脑。

- 设置 WSL2 为默认版本

  ```
  wsl --set-default-version 2
  ```

- 关闭wsl

  ```
  wsl --shutdown
  ```

- wsl1与wsl2区别

  | 特性               | WSL1                                       | WSL2                                      |
  | ------------------ | ------------------------------------------ | ----------------------------------------- |
  | **内核**           | 不使用真实 Linux 内核，依赖 Windows 兼容层 | 使用完整 Linux 内核，通过轻量级虚拟机     |
  | **系统调用兼容性** | 部分系统调用不支持，某些程序无法运行       | 完全兼容 Linux 系统调用                   |
  | **文件系统性能**   | 访问 Windows 文件快；Linux 文件系统慢      | Linux 文件系统快；访问 Windows 文件稍慢   |
  | **Docker 支持**    | 不支持 Linux 容器                          | 支持 Linux 容器，适合 Docker/Kubernetes   |
  | **网络**           | 与 Windows 共享网络堆栈                    | 独立虚拟网卡（NAT），端口映射可能需要注意 |
  | **资源占用**       | 轻量                                       | 占用虚拟机资源，但仍很轻量                |
  | **适用场景**       | 轻量级开发、命令行工具、脚本               | 完整 Linux 开发环境、Docker、数据库服务等 |

  > **WSL2 = WSL 框架 + 轻量级虚拟机 + 完整 Linux 内核**，而 WSL1 只是 WSL 框架 + 系统调用翻译层。

## 修改镜像站

- 创建配置文件

  ```
  sudo vi  /etc/docker/daemon.json
  ```
  
  ```
  {
  "registry-mirrors": [
  "https://docker.m.daocloud.io",
  "https://docker.1panel.live",
  "https://hub.rat.dev"
  	]
  }
  
  ```

- 重启docker服务

  ```
  sudo service docker restart
  ```

## Docker Desktop

> 依赖于WSL，在Windows电脑上使用Docker的软件。在安装时，Docker Desktop会依赖WSL自动创建一个Linux子系统，这个子系统自带Docker，专门用来运行Docker容器并保存镜像和容器数据

在开启这个软件时，可以在Windows命令行中通过Docker命令操作docker

- 使用Docker Desktop时，内部会虚拟化一个磁盘映像用于存放容器与镜像

## 内核

> 操作系统内核的作用是与硬件交互，提供操作硬件的指令。系统应用(操作系统)封装内核指定为函数，便于使用者调用。用户程序基于系统函数库实现功能。

即使内核相同，但是由于操作系统封装的函数库不同，因此程序一般不可以跨操作系统运行

## Docker跨操作系统运行

> Docker会将应用程序与所需要调用的系统函数库一起打包，当应用程序运行到不同操作系统时，直接基于打包的库函数，借助操作系统的Linux内核来运行。因此Docker种的应用程序在运行时，只依赖当前系统的Linux内核，无关其系统函数库。

## Shell

> 命令行解释器，本质上是一个程序，它的作用是 **接收用户输入的命令、解析并交给可执行文件执行**。

- Bash

  > Linux系统的默认shell

- CMD

  > Windows系统的默认shell

- PowerShell 

### hosts文件

Sidecar，边车模式