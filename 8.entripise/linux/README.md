## centos常用命令

### 服务器初始化

#### 1. 安装阿里的仓库

```bash
yum install wget 
wget -O /etc/yum.repos.d/CentOS-aliyum.repo http://mirrors.aliyun.com/repo/Centos-6.repo

mv CentOS-Base.repo CentOS-Base.repo.bak
mv CentOS-aliyum.repo CentOS-Base.repo

yum clean all  
yum makecache 

yum install net-tools
```

#### 2. java环境配置

> 1.7的安装

```bash
yum install java-1.7.0-openjdk.x86_64 
yum install java-1.7.0-openjdk-devel.x86_64 
yum install -y java-1.7.0-openjdk.x86_64 java-1.7.0-openjdk-devel.x86_64
```

> 1.8的安装

```bash
yum install java-1.8.0-openjdk.x86_64
yum install java-1.8.0-openjdk-devel.x86_64
yum -y install java-1.8.0-openjdk.x86_64 java-1.8.0-openjdk-devel.x86_64
```

> 卸载 jdk 版本

```bash
#列出当前的版本
rpm -qa|grep jdk    

yum -y remove java java-1.8.0-openjdk-1.8.0.131-0.b11.el6_9.x86_64
yum -y remove java java-1.8.0-openjdk-devel-1.8.0.131-0.b11.el6_9.x86_64
yum -y remove java java-1.8.0-openjdk-headless-1.8.0.131-0.b11.el6_9.x86_64

yum -y remove java java-1.8.0-openjdk-1.8.0.131-0.b11.el6_9.x86_64 java-1.8.0-openjdk-devel-1.8.0.131-0.b11.el6_9.x86_64 java-1.8.0-openjdk-headless-1.8.0.131-0.b11.el6_9.x86_64
```

> mac OS

```bash
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
http://jingyan.baidu.com/article/1612d500afc297e20f1eee7f.html
```

#### 3. 服务器间拷贝文件

```bash
yum install -y openssh-clients.x86_64 

scp /home/apache-tomcat-7.0.59 root@172.19.2.75:/opt/project 
```

> 带端口

```bash
scp -P 22223 consumer-api.war KJDS_USER@192.168.0.73:/opt/project/
```

#### 3.1 文件磁盘挂载

> nfs 共享服务器 linux 之间，挂盘的需要

```bash
yum -y install nfs-utils.x86_64 
yum -y install nfs4-acl-tools.x86_64 
```

> 给特定文件设置权限

```bash
/opt/fileserver 192.168.168.121(rw,no_root_squash,no_all_squash,sync) *(rw)
```

#### 3.2 挂载共享目录

1. 安装nfs-utils和rpcbind 复制代码代码如下:

```bash
yum install nfs-utils rpcbind
```

1. 设置开机启动服务 复制代码代码如下:

```bash
chkconfig nfs on 
chkconfig rpcbind on 
```

1. 启动相关服务 复制代码代码如下:

```bash
service rpcbind start 
service nfs start
```

1. 创建共享目录 复制代码代码如下:

```bash
mkdir -p /export/primary 
mkdir -p /export/secondary
```

1. 编辑/etc/exports文件添加如下内容,复制代码代码如下:

```bash
vi /etc/exports 

/export*(rw,async,no_root_squash,no_subtree_check) 
```

#### 3.3文件挂载

```bash
mount -t nfs4 192.168.0.213:/opt/fileserver /opt/AppFiles  文件挂载
```

6.刷新配置立即生效 复制代码代码如下:

```bash
exportfs -a
```

#### 4. tomcat 配置conf 下 的web.xml

```bash
<Context  docBase="${catalina.home}/webapps/consumer-system" path="" sessionCookieName="consumer-system" defaultSessionTimeOut="28800"/>
```

> 文件服务器配置

```bash
<Context path="/AppFiles" docBase="/opt/fileserver" workDir="/opt/fileserver" debug="5" reloadable="false" crossContext="true" /> 
```

#### 5. lixun 下常用命令

> **查看磁盘情况**

```bash
df -hl 查看磁盘空间
du -h --max-depth=1   按磁盘当前层级查看具体的文件夹大小
ls /dev/vd*      列出所有磁盘
fdisk -l         列出所有磁盘详情
cat /etc/fstab   输出磁盘注册表的文件
mount /dev/vdb1 /GeT    挂载磁盘 到指定的挂载点；
netstat -nap | grep 8080
ps -ef | grep java
unzip master  解压
tail -f logs/catalina.out    查看输出的日志
kill -9 3321  pid为相应的进程号
netstat -anp | grep 9217 
 pid直接查看指定端口的进程pid
ln -s /opt/hadoop-2.7.3 /opt/hadoop/
```

> *查看某软件是否安装*

```bash
rmp -qa | grep samba   
```

#### 6. 网络 IP config 配置

> 编辑IP 配置

```bash
vim /etc/sysconfig/network-scripts/ifcfg-eth0
```

> 配置范本 如下

```bash
DEVICE=eth0 
TYPE=Ethernet 
UUID=43e20cc5-f88b-46c6-a6d2-f4fdb4bd54f8 
ONBOOT=yes 
NM_CONTROLLED=yes 
BOOTPROTO=static 
HWADDR=00:0C:29:21:DA:EF 
IPADDR=192.168.1.8 
PREFIX=24 
GATEWAY=192.168.1.1 
DNS1=202.96.134.133 
DEFROUTE=yes 
IPV4_FAILURE_FATAL=yes 
IPV6INIT=no 
NAME="System eth0"
```

> 防火墙 关闭

```bash
service iptebles status
service iptebles stop
```

## 部署响应工具包





## mac操作



#### 挂载Anaconda目录

```
ln -sf /Volumes/MacData/AnacondaProjects /Users/jarvischang/anacondaPro
```

