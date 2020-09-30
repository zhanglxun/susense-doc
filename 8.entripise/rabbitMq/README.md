## RabbitMQ



## RabbitMQ安装环境部署

### 1. 环境
```
系统版本：CentOS 6.6
RabbitMQ－Server：3.5.1
```
### 2.安装erlang

```
wget http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
rpm --import http://packages.erlang-solutions.com/rpm/erlang_solutions.asc
```
### 3.安装 添加RPMforge支持(64位)
> ==连接不到源，未安装也可以继续==
```
wget http://packages.sw.be/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm  
## 导入 key 
rpm --import http://apt.sw.be/RPM-GPG-KEY.dag.txt 
## 安装 RPMforge
rpm -i rpmforge-release-0.5.2-2.el6.rf.*.rpm
```

### 4.安装 erlang

```
# EL7:
su -c 'rpm -Uvh https://download.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm'

yum install erlang
```

### 5.检查版本

```
erl -version
```


## 安装部署RabbitMQ

> ==刚开始 包邮问题，删除包重新下载后，就可以了==

> 3.6.12

```
wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.12/rabbitmq-server-3.6.12-1.el7.noarch.rpm

wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.15/rabbitmq-server-3.6.15-1.el7.noarch.rpm

rpm --import http://www.rabbitmq.com/rabbitmq-signing-key-public.asc 

yum install rabbitmq-server-3.6.12-1.el7.noarch.rpm
```

> 3.5.12

```
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.5.1/rabbitmq-server-3.5.1-1.noarch.rpm
rpm --import http://www.rabbitmq.com/rabbitmq-signing-key-public.asc 
yum install rabbitmq-server-3.5.1-1.noarch.rpm
```

```
# 最新
rpm --import https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc
```



## 管理和启动

> 配置为守护进程随系统自动启动

```
chkconfig rabbitmq-server on
```
> 启动rabbitMQ服务
```
/sbin/service rabbitmq-server start
```
> 启动终端

```
rabbitmq-plugins enable rabbitmq_management
```

> 出现如下提示
```
The following plugins have been enabled:
  mochiweb
  webmachine
  rabbitmq_web_dispatch
  amqp_client
  rabbitmq_management_agent
  rabbitmq_management

Applying plugin configuration to rabbit@localhost... started 6 plugins.
```

## 基础使用

### 1.打开管理界面
```
http://localhost:15672/
http://192.168.1.217:15672/
```

### 2.一些基本操作

> ==服务器启动停止及权限操作等==
```
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

> ==数据及数据查看操作等==


```
rabbitmqctl list_queues
```

### 3.添加远程访问用户


```
rabbitmqctl add_user jarvis 85894113
rabbitmqctl add_user jarvis weiyu@2020

rabbitmqctl set_user_tags jarvis administrator
```

### 4.简书上的教程，参考资料

```
http://www.jianshu.com/p/d985a547eac8
http://www.jianshu.com/p/ce725e41edab
http://www.jianshu.com/p/e3af4cf97820
#RabbitMQ 远程设置，及角色说明
```
### 5.远程连接排查问题

- 新建一个程序使用的账户，如springcloud
- 在Admin标签页下，点击新增的用户"admin"，授予权限
- 进入授权页面，默认直接点击"set permission"，为该用户对VirtualHost"/" 的访问授权，==这个很重要，否则会出现无法连接到服务器的错误==

---