## 版本更新升级说明

### 一、部署结构图

#### 1、部署结构图



<img src="http://pxn.hillsoft.cn/%E9%83%A8%E7%BD%B2%E6%9E%B6%E6%9E%84%E5%9B%BE.png" style="zoom:50%;" />

#### 2、此次更新包地址

1. 地址和对照

   ```
   http://183.11.235.36:8096/
   ```

2. 参考说明

   <img src="http://pxn.hillsoft.cn/WX20200521-161155@2x.png" style="zoom:50%;" />

### 二、mysql数据库

#### 1、查看数据版本

1. 查看数据库版本

   ```bash
   select VERSION()
   
   5.7.29
   ```

2. 如果数据库版本低于5.7 ，则需要升级数据库；

   ```bash
   #数据库版本升级，查看依赖的版本
   
   yum repolist all | grep mysql
   
   mysql56-community   MySQL 5.6 Community Server
   ```

   - 停止服务

   ```bash
   # 停止服务
   systemctl stop mysqld 
   # 卸载旧版本
   yum -y remove mysql-community-server-5.6.47-2.el6.x86_64
   ```

   - 更新源

   ```bash
   安装新的respo
   #vim /etc/yum.repos.d/mysql-community.repo
   #Enable to use MySQL 5.7
   [mysql57-community]
   name=MySQL 5.7 Community Server
   baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
   enabled=1
   gpgcheck=1
   gpgkey=
   ```
   
   - 清理并更新 repo源
   
   ```bash
   yum clean all
   yum makecache
   ```

   
   
   - 重新查看
   
   ```
   yum repolist all | grep mysql
   mysql56-community   MySQL 5.7 Community Server
   ```
   
   - 开始安装
   
   ```bash
   # 写证书
   vim /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
   
   # 去这找证书
   https://dev.mysql.com/doc/refman/5.7/en/checking-gpg-signature.html
   
   #引入并生效 重要
   rpm --import /etc/pki/rpm-gpg/RPM* 
   
   # 安装
   yum install mysql-community-server
   ```
   
3. 备份

   >注意：

   升级前一定要先进行数据库的备份

   升级前确保数据库引擎在升级过程中不发生修改

   mysql 更新后，要验证原账号密码是否正确

   源数据库要检查数据是否丢失



#### 2、新建库，导入数据库

1. 新建一个数据库，将更新包重点最新的数据，导入到新建的库中。

   ```
   platformdb-20200520-uat.sql 

   最近的数据库文件
   ```

#### 3、对比表结构

1. 通过数据库工具，对比新的库结构，和原来的库结构，生成数据库脚本。

   ```
   表的差异，可能但不限于：
   【新建的表】
   g_field_extend
   g_field_hospital
   g_integrate_log
   g_pregnant_log

   g_pregnant 【字段有差】
   s_users【字段有差】
   g_monitor【字段有差】
   ……
   ```

   

2. 通过数据库工具，对比新的库配置表数据，和原来的数据库表数据，生成数据库脚本。

   ```
   数据的差异，可能但不限于：

   s_dictionary
g_field_extend【基础数据需初始化】

   ……
   ```
   
   > 此部分工作量较大。请注意时间，需要逐个表对比一下，否则更新时候，会报错。
   >
   > 此部分容易出问题，请慎重。

#### 4、生成脚本导入【重要】

1. 将差异的数据库结构脚本，在环境执行；

2. 将差异的数据库sql，在相应的数据库表中写入

   > 注意：
   >
   > 此工作要慎重，一点确认清楚表的结果，数据，才写入。
   >
   > 做此操作之前，建议备份原数据库。



### 三、hbase 数据存储



#### 1、登录hbase 命令行

1. 路径

   ```bash
   cd /data/hbase/bin

   hbase shell
   ```


#### 2、查看现有表结构

1. 查询表，目前最新的hbase 表有如下7个；

   ```
   hbase(main):003:0* list

   TABLE
   bloodPressureData
   bloodSugarData
   fetalHeartData
   fetalMonitoringData
   fetalMovementData
   pregnantInfoData
   users
   ```


#### 3、对比并增加表结构

1. 表结构

   ```bash
   新建 pregnantInfoData

   create 'pregnantInfoData','customInformation'
   ```

2. 其他表结构是否有缺，需要对比，找出版本差异。



### 四、redis 中间件

#### 1. 部署redis

1. 安装依赖

   ```bash
   # 安装依赖
   yum install epel-release 
   ```
   
2. 安装

   ```bash
   # 安装redis
   yum install redis
   ```

3. 配置

   ```
   vim /usr/local/redis-3.2.8/redis.conf

   # 注释 这个
   #bind 127.0.0.1
   protected-mode no
   daemonize yes

   # 密码就在这里
   requirepass ******
   ```

4. 重启

   ```
   redis-server /usr/local/redis-3.2.8/redis.conf
   ```

#### 2. 部署测试

1. 使用桌面工具连接

   ```
   桌面工具medis 连接，并测试运行正常
   ```

2. 使用命令行连接

   ```bash
   redis-cli -p 6379
   # 认证密码
   > auth ******

   # 进入到命令行
   redis> set foo bar
   OK
   redis> get foo
   "bar"
   ```

#### 3. Apollo 配置中心

1. 配置地址：redis_url:192.168.1.1
2. 配置密码：redis_password:******

>  配置好之后，将ip地址 、端口、密码，给到开发，配置到apollo 里面。
>
> 内网访问，可以只是要内网的IP 地址



### 五、RabbitMQ 部署

#### 1. 部署rabbitMQ

1. 安装依赖

   ```bash
   系统版本：CentOS 7.6
   RabbitMQ－Server：3.6.15

   # 安装依赖、安装erlang
   wget http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
   rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
   rpm --import http://packages.erlang-solutions.com/rpm/erlang_solutions.asc
   ```
   
2. 安装依赖

   ```bash
   # 安装 添加RPMforge支持(64位)
   wget http://packages.sw.be/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm  
   ## 导入 key 
   rpm --import http://apt.sw.be/RPM-GPG-KEY.dag.txt 
   ## 安装 RPMforge
   rpm -i rpmforge-release-0.5.2-2.el6.rf.*.rpm

   # 安装EL7:
   su -c 'rpm -Uvh https://download.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm'

   #安装 erlang
   yum install erlang
   ```

3. 检查版本

   ```
   erl -version
   ```
   
4. 安装rabbitMQ

   ```bash
   # 3.6.12
   wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.12/rabbitmq-server-3.6.12-1.el7.noarch.rpm

   rpm --import http://www.rabbitmq.com/rabbitmq-signing-key-public.asc 

   yum install rabbitmq-server-3.6.12-1.el7.noarch.rpm

   #配置
   rpm --import https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc
   ```

#### 2. 配置和启动

2. 启动

   ```bash
   # 配置为守护进程随系统自动启动
   chkconfig rabbitmq-server on

   # 启动rabbitMQ服务
   /sbin/service rabbitmq-server start

   #启动终端
   rabbitmq-plugins enable rabbitmq_management

   出现：
   The following plugins have been enabled:
     mochiweb
     webmachine
     rabbitmq_web_dispatch
     amqp_client
     rabbitmq_management_agent
     rabbitmq_management

   Applying plugin configuration to rabbit@localhost... started 6 plugins.
   ```

3. 配置权限

   ```
   增加用户
   rabbitmqctl add_user jarvis weiyu@2020
   
   rabbitmqctl set_user_tags jarvis administrator
   ```


4. 登录检查界面

   

5. 常用操作

   ```bash
    $ chkconfig rabbitmq-server on  # 添加开机启动RabbitMQ服务

   $ /sbin/service rabbitmq-server start # 启动服务

   $ /sbin/service rabbitmq-server status  # 查看服务状态

   $/sbin/service rabbitmq-server stop   # 停止服务

   # 查看当前所有用户
   $ rabbitmqctl list_users

   # 查看默认guest用户的权限
   $ sudo rabbitmqctl list_user_permissions guest

   # 由于RabbitMQ默认的账号用户名和密码都是guest。为了安全起见, 先删掉默认用户
   $ sudo rabbitmqctl delete_user guest

   # 添加新用户
   $ sudo rabbitmqctl add_user username password

   # 设置用户tag
   $ sudo rabbitmqctl set_user_tags username administrator

   # 赋予用户默认vhost的全部操作权限
   $ sudo rabbitmqctl set_permissions -p / username ".*" ".*" ".*"

   # 查看用户的权限
   $ sudo rabbitmqctl list_user_permissions username
   ```

   

#### 3. apollo 中心配置

1. 需要在apollo 配置对应的地址

   >1. 配置地址：mq_host:192.168.1.1
   >2. 配置账户：mq_username:jarvis
   >3. 配置密码：mq_password:**
   
   

### 六、硬件对接

#### 1、服务器地址配置

1. 将代理服务器 nginx 的地址，配置到硬件：

   > 此处设置 安丘医院部署的服务器代理网关的地址：

   ```
   http://192.168.1.73:25555/integrate/v1/shuangjia/saveReport
   ```

   

2. 检查配置正确，并保存。

#### 2、数据上报测试

1. 测试一条数据，上传到系统；
2. 到后台检查数据是否正常。

### 七、交付小程序

#### 1、SSL证书

1. Centos 环境准备

   ```
   # 环境准备

   yum -y install pcre pcre-devel 
   yum -y groupinstall "Development Tools" 
   yum install -y zlib-devel

   # 需要SSL 模块，则按照此依赖
   yum install -y openssl-devel
   ```

2. 在nginx 中安装 SSL 模块配置

   ```
   # 解压
   tar -zxvf tengine-2.3.2.tar.gz

   # 停止服务
   /usr/local/nginx/sbin/nginx -s stop

   # 配置模块
   ./configure --prefix=/usr/local/nginx --with-stream --with-http_ssl_module --with-http_stub_status_module

   make
   make install

   #启动服务
   cd /usr/local/nginx/sbin
   ./nginx -s reload
   ```

   

3. 配置nginx 的

   ```bash
    server {
           listen       443;
           server_name  api.wehealth.net.cn;
       
           ssl     on;
       
           location / {
               proxy_pass http://172.18.12.246:5555;
       
               #proxy_set_header Host $host:5555
               proxy_redirect default;
               rewrite  ^./?(.*)$ /$1 break;
               include  uwsgi_params;
            }
       
           ssl_certificate      /usr/local/nginx/cert/3886002_api.wehealth.net.cn.pem;
           ssl_certificate_key  /usr/local/nginx/cert/3886002_api.wehealth.net.cn.key;
       
           #ssl_session_cache    shared:SSL:1m;
           ssl_session_timeout  5m;
       
           ssl_ciphers     ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
           ssl_protocols TLSv1 TLSv1.1 TLSv1.2;   #修改protocols。
           ssl_prefer_server_ciphers  on;
       
       }
   ```

   

#### 2、网关部署

1. 确保网关上，nginx 的配置正确，代理的 zuul 网关

   ```
   http://ip地址:5555/swagger-ui.html
   ```

2. 网关对小程序，需要证书。



#### 3、域名备案

1. 网关的 接口，注册到微信后台

   > - 域名测试好之后，提交给开发工程师 田通，注册到微信后台；
   > - 同时 给到 陈丽，修改微语的后台配置；



### 八、微服务配置

#### 1、微服务说明

1. 微服务的总数
   - 总计14个服务包
2. 微服务的环境
   - 依赖java 8的环境

#### 2、微服务调度

1. producer
   - system
   - message
   - guardianship
   - mall
   - documen
   - content
   - analysis
2. consumer
   - console
   - mobile
   - Obsetric
   - integrate
3. 监控中心：admin
4. 注册中心：eureka
5. 网关接入：zuul

x
1. 查看服务器运行状态，监控中心

   ```
   http://127.0.0.1:8060/login
   ```

   <img src="http://pxn.hillsoft.cn/WX20200521-155402@2x.png" style="zoom:50%;" />

2. 确保各服务运行正常。

#### 4、注册中心

1. 注册中心

   ```
   http://127.0.0.1:1111/
   ```

   <img src="http://pxn.hillsoft.cn/WX20200521-155502@2x.png" style="zoom:50%;" />

2. 确保各服务运行正常。

#### 5、网关接口

1. 注册中心

   ```
   http://127.0.0.1:5555/swagger-ui.html
   ```

   ![](http://pxn.hillsoft.cn/WechatIMG302.png)

   

2. **确保各服务运行正常。**



#### 6、原院外系统

> 部署

   ```
 nginx 中指定的目录，更新此目录
   ```

> 监查

1. 院外系统正常运行
2. 登录后查询各功能数据正常
3. 测试下单流程，支付流程，数据监测上传流程，小程序正常

#### 7、院内系统

> 部署

   ```bash
   nginx 中指定的目录，更新此目录
   ```

> 检查

1. 原内系统正常运行
2. 登录后查询各功能数据正常
3. 测试建档流程，小程序建档流程 正常使用



### 九、配置后需回执的配置

> 部分配置好之后，需将配置回执配到apollo 、微信后台、和微语的医院配置：

1. 注册中心地址（未变化，则不用重新配置）
2. msql数据库地址、账号、密码（未变化，则不用重新配置）
3. Redis 地址，账号，密码
4. rabbitMQ地址、账号、密码
5. SSL 带证书的域名
