`

### 1、版本查看及升级内核

1. CentOS 具体要求如下：
    - 必须64位操作系统
    - 建议内核在3.8 以上

2. 查询内核，执行命令

   ```bash
   $ uname -r
   4.4.0-116-generic
   ```

3. 查询操作系统版本

   ```bash
   cat /etc/lsb-release
   DISTRIB_ID=Ubuntu
   DISTRIB_RELEASE=16.04
   DISTRIB_CODENAME=xenial
   DISTRIB_DESCRIPTION="Ubuntu 16.04.5 LTS"
   ```

4. CemtOs 6.5 的版本 ，默认是2.6的，需要升级

    ```bash
    rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
    rpm -ivh http://www.elrepo.org/elrepo-release-6-5.el6.elrepo.noarch.rpm
    yum -y --enablerepo=elrepo-kernel install kernel-lt
    ```
5. 升级后，需要修改配置重启生效
    > 将default=1修改为default=0。
    ```bash
    vi /etc/grub.conf
    #重启系统，内核生效
    reboot
    ```
### 2、docker 的 安装

#### 2.1 ubuntu安装docker CE

1. Docker CE 支持以下版本的 [Ubuntu](https://www.ubuntu.com/server) 操作系统：

   - Bionic 18.04 (LTS)
   - Xenial 16.04 (LTS)

2. 卸载旧版本

   ```bash
   sudo apt-get remove docker \
                  docker-engine \
                  docker.io
   ```

3. 通过公网环境安装

   ```bash
   curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
   ```

4. 通过阿里云内网在ECS安装

   ```bash
   # step 1: 安装必要的一些系统工具
   sudo apt-get update
   sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common

   # step 2: 安装GPG证书
   curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

   # Step 3: 写入软件源信息
   sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

   # Step 4: 更新并安装 Docker-CE
   sudo apt-get -y update
   sudo apt-get -y install docker-ce
   ```



#### 2.2 centos 安装

1. 通过yum 安装，阿里云的镜像

   ```bash
   # step 1: 安装必要的一些系统工具
   sudo yum install -y yum-utils device-mapper-persistent-data lvm2

   # Step 2: 添加软件源信息
   sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

   # Step 3: 更新并安装 Docker-CE
   sudo yum makecache fast
   sudo yum -y install docker-ce

   # Step 4: 开启Docker服务
   sudo service docker start
   ```

#### 2.3 说明地址和校验

1. 说明的地址

   ```html
   https://yq.aliyun.com/articles/110806?spm=5176.8351553.0.0.66b41991jPRMxT
   ```

   > 有一些注释在这块查看

2. 校验安装情况

   ```bash
   root@iZbp12adskpuoxodbkqzjfZ:$ docker version
   Client:
    Version:      17.03.0-ce
    API version:  1.26
    Go version:   go1.7.5
    Git commit:   3a232c8
    Built:        Tue Feb 28 07:52:04 2017
    OS/Arch:      linux/amd64

   Server:
    Version:      17.03.0-ce
    API version:  1.26 (minimum version 1.12)
    Go version:   go1.7.5
    Git commit:   3a232c8
    Built:        Tue Feb 28 07:52:04 2017
    OS/Arch:      linux/amd64
    Experimental: false
   ```

#### 2.3 镜像加速


1. 镜像加速地址
    ```bash
    # 阿里云的镜像加速地址
    https://41xm9v8m.mirror.aliyuncs.com
    #daoCloud的加速地址
    http://f1361db2.m.daocloud.io
    ```


### 3、常规信息查看

1. 查看版本是否安装成功；
    ```bash
    # 版本信息
    docker version
    # 查看系统信息
    docker info
    ```

2. centos启动服务

    ```bash
    service docker status                   查看服务运行状态
    service docker start                    启动
    sudo chkconfig docker on                增加到启动项
    ```
3. ubuntu启动服务

    ```bash
    $ sudo systemctl enable docker
    $ sudo systemctl start docker
    ```
4. 测试是否安装正确

   ```bash
   docker run hello-world
   ```

5. 查询信息，有些许安装才能生效

    ```bash
    $ docker --version
    Docker version 1.12.3, build 6b644ec
    $ docker-compose --version
    docker-compose version 1.8.1, build 878cff1
    $ docker-machine --version
    docker-machine version 0.8.2, build e18a919

    ```

### 4、常规的使用

#### 4.1 国内较慢需增加镜像

1. Ubuntu 16.04+、Debian 8+、CentOS 7

    > 对于使用 `systemd` 的系统，请在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）

    ```bash
    vim /etc/docker/daemon.json
    {
      "registry-mirrors": [
        "https://dockerhub.azk8s.cn",
        "https://reg-mirror.qiniu.com"
      ]
    }
    ```
2. 重启服务

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```

3. 重新查看docker info 是否生效

   ```bash
   docker info
   # 出现
   Registry Mirrors:
     https://41xm9v8m.mirror.aliyuncs.com/
     https://dockerhub.azk8s.cn/
     https://reg-mirror.qiniu.com/
   ```

4. MacOS 系统

   > 对于使用 macOS 的用户，在任务栏点击 Docker Desktop 应用图标 -> Perferences... -> Daemon -> Registry mirrors。在列表中填写加速器地址 `https://dockerhub.azk8s.cn`。修改完成之后，点击 `Apply & Restart` 按钮，Docker 就会重启并应用配置的镜像地址了。



#### 4.2 使用镜像

##### 1.获取镜像

1. 获取镜像的格式为：

   ```bash
   docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]

   docker pull --help
   ```

2. 可以直接找地址获取：

   ```bash
   # docker pull registry.hub.docker.com/ubuntu:12.04

   docker pull ubuntu:18.04
   ```

3. 完成后，可以启动该镜像

   ```bash
   # 交互式进入容器中
   docker run -it --rm \
       ubuntu:18.04 \
       bash
   ```

- `-it`：这是两个参数，一个是 `-i`：交互式操作，一个是 `-t` 终端。我们这里打算进入 `bash` 执行一些命令并查看返回结果，因此我们需要交互式终端。
- `--rm`：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 `docker rm`。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 `--rm` 可以避免浪费空间。
- `ubuntu:18.04`：这是指用 `ubuntu:18.04` 镜像为基础来启动容器。
- `bash`：放在镜像名后的是 **命令**，这里我们希望有个交互式 Shell，因此用的是 `bash`。



4. 在交互式里面执行

   ```bash
   cat /etc/os-release
   #查看版本号
   ```

4. 退出

   ```bash
   exit

   # 交互式进入容器
   sudo docker run -it centos /bin/bash

   #启动拉取一个nginx
   $ docker run -d -p 8080:80 --name webserver nginx

   # 列出所有container
   docker ps -a

   # 列出运行的contains
   docker ps
   docker container ls -a

   #删除容器
   docker container rm cc3f2ff51cab cd20b396a061

   # 列出最后一次启动的
   docker ps -l
   ```


##### 2. 列出镜像/容器

1. 列出镜像

   ```bash
   # 检索镜像,两个命令基本一样
   docker images
   docker image ls

   REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   ubuntu              18.04               a2a15febcdf3        2 weeks ago         64.2MB
   hello-world         latest              fce289e99eb9        8 months ago        1.84kB

   # 列出正在运行的，带状态
   docker ps -a
   ```

2. 查看所占的空间

   ```bash
   docker system df

   TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
   Images              2                   1                   64.19MB             64.19MB (99%)
   Containers          1                   0                   0B                  0B
   Local Volumes       0                   0                   0B                  0B
   Build Cache         0                   0                   0B                  0B
   ```

3. 显示虚悬镜像(**dangling image**)

   ```bash
   # 显示虚悬镜像
   docker image ls -f dangling=true

   # 可是删除
   docker image prune
   ```

4. 中间层镜像

   ```bash
   $ docker image ls -a
   ```

5. 列出指定的

   ```bash
   docker image ls ubuntu

   REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   ubuntu              18.04               a2a15febcdf3        2 weeks ago         64.2MB
   ```

6. 列出指定仓库

   ```bash
   # 指定仓库名和标签
   $ docker image ls ubuntu:18.04

   #列出版本后的镜像 --filter
   docker image ls -f since=mongo:3.2

   # 会输出一个完整的表格
   docker image ls -q
   ```

7. 特定格式

   下面的命令会直接列出镜像结果，并且只包含镜像ID和仓库名

   ```bash
   $ docker image ls 

   a2a15febcdf3: ubuntu
   fce289e99eb9: hello-world

   # 或者打算以表格等距显示，并且有标题行，和默认一样，不过自己定义列
   docker image ls 

   IMAGE ID            REPOSITORY          TAG
   a2a15febcdf3        ubuntu              18.04
   fce289e99eb9        hello-world         latest
   ```

##### 3. 启动和停止

1. 查询列出，带状态

   ```bash
   # 列出正在运行的，带状态
   docker ps -a
   CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
   777fbd1f738a        nginx:v2            "nginx -g 'daemon of…"   7 minutes ago       Exited (0) 3 minutes ago                       web2
   3465747644e2        nginx               "nginx -g 'daemon of…"   36 minutes ago      Exited (0) 4 minutes ago                       webserver
   d48c9f954362        hello-world         "/hello"                 3 hours ago         Exited (0) 3 hours ago                         dazzling_chaplygin
   ```

2. 启动

   ```bash
   # 根据名称
   docker start web2
   # 根据id
   docker start 777fbd1f738a
   ```

##### 4. 获取和删除

1. 停止和删除

   ```bash
   #先停止一下
   docker stop id
   ```



2. 用id，镜像名，摘要删除镜像

   ```bash
   #先停止一下
   docker stop id
   # 列出
   docker image ls

   REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   ubuntu              18.04               a2a15febcdf3        2 weeks ago         64.2MB
   hello-world         latest              fce289e99eb9        8 months ago        1.84kB

   #根据id删除
   docker image rm fce
   #根据仓库名：标签删除
   docker image rm centos

   #更精确的是使用 镜像摘要 删除镜像
   docker image ls --digests

   docker image rm node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
   Untagged: node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
   ```

3. 配合使用

   ```bash
   # 比如需要删除所有仓库名为 redis 的镜像
   $ docker image rm $(docker image ls -q redis)
   # 删除所有在 mongo:3.2 之前的镜像
   $ docker image rm $(docker image ls -q -f before=mongo:3.2)
   ```



##### 5. commit镜像构成

1. 启动一个

   ```bash
   # 运行本地镜像的一个nginx 容器
   # 命名为webserver，在localhost 查看运行
   docker run --name webserver -d -p 8080:80 nginx
   ```
   > 本地浏览器访问localhost

2. 交互方式进入容器，修改文件

   ```bash
   $ docker exec -it webserver bash
   root@3729b97e8226:/# echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
   root@3729b97e8226:/# exit
   exit
   ```

   - 此命令修改，不要漏了这个 符号【>】，也可以使用如下命令修改：

   ```bash
   cd /usr/share/nginx/html/
   tee index.html <<- 'EOF'
   > Welcom to javis docker registry~!
   > EOF
   ```

   - 刷新页面，即可查看到修改

3. 查看修改的改动

   ```bash
   docker diff webserver
   ```

4. 保存 commit

   ```bash
   # docker commit 的语法格式为：
   docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]

   docker commit \
       --author "jarvis2chang <lixun522@gmail.com>" \
       --message "修改了默认网页" \
       webserver \
       nginx:v2

   sha256:cc34b45d4b75a8baf77799835c7712e70f8609b4a47d611a45e319c34b4d901f

   # 查看新的
   docker image ls nginx

   REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   nginx               v2                  cc34b45d4b75        34 seconds ago      126MB
   nginx               latest              5a3221f0137b        13 days ago         126MB
   
   # 在启动一个镜像，会创建容器
   docker run --name web2 -d -p 8081:80 nginx:v2
   ```

5. 领取镜像
   ```bash
   # 拉取镜像
   docker pull nginx
   ```

##### 6. 列出docker-machine

1. 列出

   ```bash
   docker-machine ls     列出docker-machine
   docker-machine help   列出docker-machine 的帮助文档

   #搜索查询镜像
   docker search redis
   ```

### 5、Dockerfile

#### 5.1 定制一个镜像

1. 使用from

   ```bash
   $ mkdir mynginx
   $ cd mynginx
   $ touch Dockerfile
   ```

> 编辑内容为：

```bash
   FROM nginx
   RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

> dockerfile的正确写法，层数不能太多。**Union FS 是有最大层数限制的，比如 AUFS，曾经是最大不得超过 42 层，现在是不得超过 127 层。**

   ```

FROM debian:stretch

RUN buildDeps='gcc libc6-dev make wget' \
       && apt-get update \
       && apt-get install -y $buildDeps \
       && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
       && mkdir -p /usr/src/redis \
       && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
       && make -C /usr/src/redis \
       && make -C /usr/src/redis install \
       && rm -rf /var/lib/apt/lists/* \
       && rm redis.tar.gz \
       && rm -r /usr/src/redis \
       && apt-get purge -y --auto-remove $buildDeps
   ```



1. 构建镜像，运行刚才的上下文

   ```bash
   docker build -t nginx:v3 .
   ```

   命令的格式为：

   ```bash
   docker build [选项] <上下文路径/URL/->
   ```

   启动这个镜像

   ```
   docker run --name web3 -d -p 8083:80 nginx:v3
   ```

2. Context 的概念

#### 5.2 指令详解

##### 1. copy

1. 格式

   ```bash
   COPY [--chown=<user>:<group>] <源路径>... <目标路径>
   COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]
   ```

2. `<源路径>` 可以是多个，甚至可以是通配符

   ```bash
   COPY hom* /mydir/
   COPY hom?.txt /mydir/
   ```

3. 部署自己的网站的例子

   ```
   # 查看容器
   dokcer container ls -a
   #删除容器
   docker container rm 8b7ebb43054f
   #删除镜像
   docker image rm nginx:v3

   #根据 dockerfile 和contenx目录构建镜像
   docker build -t nginx:v3 .
   #启动镜像设定端口
   docker run --name web2 -d -p 8081:80 nginx:v2

   #可以访问ip端口了
   192.168.1.73:8083
   ```

##### 2. add

 1. 如果 `<源路径>` 为一个 `tar` 压缩文件的话，压缩格式为 `gzip`, `bzip2` 以及 `xz` 的情况下，`ADD` 指令将会自动解压缩这个压缩文件到 `<目标路径>` 去。

 2. ```
    FROM scratch

    ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /
    ```
   ```

2. 因此在 `COPY` 和 `ADD` 指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用 `COPY` 指令，仅在需要自动解压缩的场合使用 `ADD`。

3. 在使用该指令的时候还可以加上 `--chown=<user>:<group>` 选项来改变文件的所属用户及所属组

   ```
   ADD --chown=55:mygroup files* /mydir/
   ADD --chown=bin files* /mydir/
   ADD --chown=1 files* /mydir/
   ADD --chown=10:11 files* /mydir/
   ```

##### 3.cmd



##### 4.entrypoint入口点



##### 5.env 环境变量



##### 6.arg构建参数



##### 7.volume 匿名卷



##### 8. expose 暴露端口



##### 9.user 指定用户



##### 10.healthcheck健康检查



##### 11.onbuild 为他人作嫁衣



### 6、容器操作

##### 6.1启动

1. 创建并启动

2. ```bash
   docker run ubuntu:18.04 /bin/echo 'Hello world'
   Hello world
   ```

3. bash 终端

   ```
   docker run -t -i ubuntu:18.04 /bin/bash
   ```

4. 当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：

   - 检查本地是否存在指定的镜像，不存在就从公有仓库下载
   - 利用镜像创建并启动一个容器
   - 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
   - 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
   - 从地址池配置一个 ip 地址给容器
   - 执行用户指定的应用程序
   - 执行完毕后容器被终止

5. 可以利用 `docker container start` 命令，直接将一个已经终止的容器启动运行

   ```
   root@wehealth:/opt/dockerfile# ps
   PID TTY          TIME CMD
   19153 pts/11   00:00:00 ps
   19825 pts/11   00:00:00 bash

   docker container start id
   ```



##### 6.2守护态运行

1. 如果不使用 `-d` 参数运行容器

   ```
   docker run ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
   hello world
   hello world
   hello world
   hello world
   hello world
   hello world
   ```

2. 如果使用了 `-d` 参数运行容器

   ```
   docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
   444f0b780c150687c779751f42df8a6beb3d331c3f3813df86f9c0f799da7fd8
   ```

3. 使用命令查看日志

   ```
    docker container logs [container ID or NAMES]
   ```

##### 6.3终止

1. 终止容器

   ```
   docker container stop 444f0b780c15
   ```

2. 启动容器

   ```
   docker container ls -a

   docker container start [container ID or NAMES]

   docker container restart [container ID or NAMES]
   ```

##### 6.4进入容器

1. 进入容器

   > 包括使用 `docker attach` 命令或 `docker exec` 命令，推荐大家使用 `docker exec` 命令

   ```
   docker run -dit ubuntu
   # 列出容器
   docker container ls
   # 根据id进入容器
   docker attach id
   ```

   - **如果从这个 stdin 中 exit，会导致容器的停止**

2. exec 命令

   - 只用 `-i` 参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回
   - 当 `-i` `-t` 参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符

   ```
   docker run -dit ubuntu
   69d137adef7a8a689cbcb059e94da5489d3cddd240ff675c640c8d96e84fe1f6

   $ docker container ls
   CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
   69d137adef7a        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           zealous_swirles

   $ docker exec -i 69d1 bash
   ls
   bin
   boot
   dev
   ...

   $ docker exec -it 69d1 bash
   root@69d137adef7a:/#
   ```

   - 如果从这个 stdin 中 exit，不会导致容器的停止。这就是为什么推荐大家使用 `docker exec` 的原因。

     更多参数说明请使用 `docker exec --help` 查看。

##### 6.5导入导出

1. 导出容器

   可以使用 `docker export` 命令

   ```
   docker container ls -a

   docker export 7691a814370e > ubuntu.tar
   ```

2. 导入容器

   ```
   cat ubuntu.tar | docker import - test/ubuntu:v1.0

   docker image ls
   ```

3. 外部导入

   ```
   docker import http://example.com/exampleimage.tgz example/imagerepo
   ```


##### 6.6删除

1. 可以使用 `docker container rm` 来删除一个处于终止状态的容器

   ```
   $ docker container rm  trusting_newton
   trusting_newton
   ```

   - 如果要删除一个运行中的容器，可以添加 `-f` 参数。Docker 会发送 `SIGKILL` 信号给容器

2. 清理所有处于终止状态的容器

   ```
   $ docker container prune
   ```




### 7、访问仓库

##### 7.1 docker hub

1. docker hub

   ```
   https://hub.docker.com
   ```

2. 搜索镜像

   ```
   docker search centos
   ```

3. 拉取镜像

   ```
   docker pull centos
   ```

4. 推送镜像

   ```
   # username 请替换为你的 Docker 账号用户名
   docker tag ubuntu:18.04 username/ubuntu:18.04
   ```

5. 自动构建

   配置自动构建，包括如下的步骤：

   - 登录 Docker Hub；
   - 在 Docker Hub 点击右上角头像，在账号设置（Account Settings）中关联（Linked Accounts）目标网站；
   - 在 Docker Hub 中新建或选择已有的仓库，在 `Builds` 选项卡中选择 `Configure Automated Builds`；
   - 选取一个目标网站中的项目（需要含 `Dockerfile`）和分支；
   - 指定 `Dockerfile` 的位置，并保存。

##### 7.2 私有仓库

1. 通过registry镜像运行创建私有仓库

   ```
   docker run -d -p 5000:5000 --restart=always --name registry registry
   ```

   - 默认情况下，仓库会被创建在容器的 `/var/lib/registry` 目录下。你可以通过 `-v` 参数来将镜像文件存放在本地的指定路径。例如下面的例子将上传的镜像放到本地的 `/opt/data/registry` 目录。

   ```
   docker run -d \
       -p 5000:5000 \
       -v /opt/data/registry:/var/lib/registry \
       registry
   ```



2. 在私用仓库搜索，上传，下载镜像

   > 用 `docker tag` 将 `ubuntu:18.04` 这个镜像标记为 `127.0.0.1:5000/ubuntu:18.04`

   ```
   # 查看本机已有镜像
   docker image ls
   # 将本机的镜像，标记后推送到registry
   docker tag ubuntu:18.04 127.0.0.1:5000/ubuntu:18.04
   ```

   > 用 `docker push` 上传标记的镜像

   ```
   docker push 127.0.0.1:5000/ubuntu:18.04

   #查看目录
   root@wehealth:/opt/data/registry# ll
   total 12
   drwxr-xr-x 3 root root 4096 Sep  3 10:09 ./
   drwxr-xr-x 3 root root 4096 Sep  3 09:49 ../
   drwxr-xr-x 3 root root 4096 Sep  3 10:09 docker/
   ```

   > 查看仓库中的镜像

   ```
   curl 127.0.0.1:5000/v2/_catalog

   {"repositories":["ubuntu"]}
   ```

   > 删除上述镜像，在从registry 拉取

   ```
   # 删除这个镜像,删除不了的时候，用这个命令 可以删除
   docker image rm 127.0.0.1:5000/ubuntu:18.04

   # 拉取这个
   docker images

   docker pull 127.0.0.1:5000/ubuntu:18.04

   docker image ls
   ```

3. 使用远程的仓库配置

   > 于使用 `systemd` 的系统，请在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）

   ```
   {
     "registry-mirror": [
       "https://registry.docker-cn.com"
     ],
     "insecure-registries": [
       "192.168.199.100:5000"
     ]
   }
   
   ```

##### 7.3 仓库的高级配置

1. 高级配置

   ```
   
   ```

   

##### 7.4 往镜像仓库里面push



1. 阿里云

   ```
   docker login --username=zhanglxun@aliyun.com registry.cn-shenzhen.aliyuncs.com
   jarvis@2019
   docker pull registry.cn-shenzhen.aliyuncs.com/jarvis2chang/demo:0.3
   
   docker tag jarvis/demo:0.3 registry.cn-shenzhen.aliyuncs.com/jarvis2chang/demo:0.3
   
   docker push registry.cn-shenzhen.aliyuncs.com/jarvis2chang/demo:0.3
   ```

   docker push registry.cn-shenzhen.aliyuncs.com/jarvis2chang/demo:0.3

2. 私有仓库

   ```
   docker tag SOURCE_IMAGE[:TAG] harbor.wehealth.net.cn/wehealth/IMAGE[:TAG]
   docker push harbor.wehealth.net.cn/wehealth/IMAGE[:TAG]
   
   docker login --username=admin harbor.wehealth.net.cn
   
   docker login 192.168.1.209
   harbor123@jarvis
   
   docker tag jarvis/demo:0.3 192.168.1.209/wehealth/demo:0.3
   docker push harbor.wehealth.net.cn/wehealth/demo:0.3
   ```

   
##### 7.5 habor 的安装配置

1. 配置安装

```
https://juejin.im/post/5d9c2f25f265da5bbb1e3de5
```



##### 7.6 gitlab runner 的安装配置   



1. 安装配置

   ```
   curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash
   
   ```

2. 运行

   ```
   sudo yum install gitlab-runner
   sudo gitlab-runner register
   ```

   

### 8、 数据管理

##### 8.1 数据卷

1. 创建一个数据卷

   ```
   docker volume create my-vol
   ```

2. 查看所有数据卷

   ```
   docker volume ls

   docker volume inspect my-vol
   [
       {
           "CreatedAt": "2019-09-03T18:04:34+08:00",
           "Driver": "local",
           "Labels": {},
           "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
           "Name": "my-vol",
           "Options": {},
           "Scope": "local"
       }
   ]

   ```

3. 启动挂载数据卷的容器

   ```
   docker run -d -P \
       --name web \
       # -v my-vol:/wepapp \
       --mount source=my-vol,target=/webapp \
       training/webapp \
       python app.py
   ```

   > 在主机查看web容器信息

   ```
   docker inspect web
   ```

4. 挂载本地的一个wabsite

   ```
   docker run \
   --name website -d -p 8081:80 \
   --mount type=bind,source=/opt/data/nginx/website,target=/usr/share/nginx/html/ \
   nginx
   ```



5. 删除数据卷

   ```
   docker volume rm my-vol
   ```

   - 清理无主的数据卷，释放空间

   ```
   docker volume prune
   ```



##### 8.2 挂载主机目录

1. 挂载主机目录做数据卷

   ```
   docker run \
   --name website -d -p 8081:80 \
   --mount type=bind,source=/opt/data/nginx/website,target=/usr/share/nginx/html/ \
   nginx
   ```

   >Imac 上挂载一个网站

   ```
   # 直接目录挂载

   docker run \
   --name website -d -p 8081:80 \
   --mount type=bind,source=/opt/data/nginx/website,target=/usr/share/nginx/html/ \
   nginx
   ```



2. 查看数据卷具体信息

   ```
   docker inspect website

   "Mounts": [
               {
                   "Type": "bind",
                   "Source": "/opt/data/nginx/website",
                   "Destination": "/usr/share/nginx/html",
                   "Mode": "",
                   "RW": true,
                   "Propagation": "rprivate"
               }
           ],
   ```



3. 挂载单个主机文件

   ```
   docker run --rm -it \
      # -v $HOME/.bash_history:/root/.bash_history \
      --mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history \
      ubuntu:18.04 \
      bash
   ```

##### 8.3 区别

1. 使用 run 的方式和使用 dockerfile 的方式

   - Dockerfile生成目标镜像的过程就是不断docker run + docker commit的过程；

   - 会docker生成了临时容器0c842ec90849

   ```
     "Mounts": [
         {
             "Name": "8827c361d103c1272907da0b82268310415f8b075b67854f27dbca0b59a31a1a",
             "Source": "/mnt/sda1/var/lib/docker/volumes/8827c361d103c1272907da0b82268310415f8b075b67854f27dbca0b59a31a1a/_data",
             "Destination": "/var/lib/mysql",
             "Driver": "local",
             "Mode": "",
             "RW": true,
             "Propagation": ""
         }
     ]
   ```

   - 通过VOLUME，我们得以绕过docker的Union File System，从而直接对宿主机的目录进行直接读写，实现了容器内数据的持久化和共享化。

### 9、使用网络



##### 9.1 外部访问容器



##### 9.2 容器互联



##### 9.3 配置DNS



### 10、高级网络配置



### 11、三剑客之 Compose项目



##### 11.1 简介

1. `Compose` 中有两个重要的概念：
   - 服务 (`service`)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
   - 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元，在 `docker-compose.yml` 文件中定义。

##### 11.2 安装卸载

1. 安装

   ```
   curl -L https://github.com/docker/compose/releases/download/1.25.0-rc2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
   
   sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
   
   ```
   
2. 卸载

   ```
   sudo rm /usr/local/bin/docker-compose
   chmod +x /usr/local/bin/docker-compose
   ```

##### 11.3 使用

1. 查看版本

   ```
   docker-compose --version
   docker-compose version 1.24.1, build 4667896b
   ```

2. 查看资源















