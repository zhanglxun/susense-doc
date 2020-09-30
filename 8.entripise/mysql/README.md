## mysql数据库 


### 1. Mysql 5.6、5.7、8.0  安装



> 1、**查看系统里面有没有mysql 的repo**


```bash
yum repolist all | grep mysql
```
> 2、**如果没有发现，则需要配置repo**

   注意，如果要使用5.7 或者其他任何版本，只能有一个是 enabled=1的，其他的都得enabled=0。

- 5.6 目前这样安装正常
```bash
#vim /etc/yum.repos.d/mysql-community.repo
# Enable to use MySQL 5.6
[mysql56-community]
name=MySQL 5.6 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=
```

- 5.7 也可以按照
```bash
#vim /etc/yum.repos.d/mysql-community.repo
# Enable to use MySQL 5.7
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=
```
- 8.0 的安装
```bash
#vim /etc/yum.repos.d/mysql-community.repo
# Enable to use MySQL 8.0
[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=http://repo.mysql.com/yum/mysql-8.0-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

> 3、**再次查看**

```bash
yum repolist all | grep mysql
```
> 出现

```bash
mysql56-community   MySQL 5.6 Community Server
mysql56-community   MySQL 5.7 Community Server
mysql56-community   MySQL 8.0 Community Server
```

> 然后可以重建 

```bash
yum clean all
yum makecache
```

> 4、需要安装签名，参考官方的地址

参考地址：

```html
http://dev.mysql.com/doc/refman/5.6/en/checking-gpg-signature.html
https://dev.mysql.com/doc/refman/5.7/en/checking-gpg-signature.html
https://dev.mysql.com/doc/refman/8.0/en/checking-gpg-signature.html


```
> 写入如下内容:

```bash
vim /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

```bash
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v1.4.9 (SunOS)

mQGiBD4+owwRBAC14GIfUfCyEDSIePvEW3SAFUdJBtoQHH/nJKZyQT7h9bPlUWC3
RODjQReyCITRrdwyrKUGku2FmeVGwn2u2WmDMNABLnpprWPkBdCk96+OmSLN9brZ
……
jHGNO1By4fUUmwCbBYr2+bBEn/L2BOcnw9Z/QFWuhRMAoKVgCFm5fadQ3Afi+UQl
AcOphrnJ
=443I
-----END PGP PUBLIC KEY BLOCK-----
```
> **5、引入并生效 **==重要==

```bash
rpm --import /etc/pki/rpm-gpg/RPM* 
```
> **6、安装** ，安装8.0 源的问题，比较慢，要好几个小时

```bash
yum install mysql-community-server
```



### 2. 数据库的操作和权限

> 第一次登陆，先启动。5.7, 8.0 会随机一个秘密，要求输入密码
```bash
# 启动
systemctl start mysqld
登陆
mysql -u root -p

# 查看密码
cat /var/log/mysqld.log | grep password

2020-02-13T08:35:08.875653Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: 409QhOz>dvAq
```

>修改权限，修改密码，

```bash
chown -R mysql:mysql /var/lib/mysql
```

```bash
mysql -u root -p 
# 使用默认密码

ALTER USER 'root'@'localhost' IDENTIFIED BY 'Jarvis@2020';

```
> 远程设置

```
use mysql;
update user set host='%' where user='root';


```

> 允许任何主机访问数据库

```

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'Jarvis' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

> 允许myuser用户使用mypassword密码从任何主机连接到mysql服务器

```
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%'IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
```
> 安全启动

```
# 防火墙状态
firewall-cmd --state
# 关闭防火墙
systemctl stop firewalld.service

mysqld_safe &
```

> 查看错误日志

```
tail /var/log/mysqld.log
```

> 自动重新初始化数据的文件，较危险的操作

```
mysql_install_db
```

### 3.卸载数据，和重新安装

1. 查看已安装的


```
rpm -qa | grep -i mysql

mysql-community-server-5.6.47-2.el6.x86_64
mysql-community-common-5.6.47-2.el6.x86_64
mysql-community-client-5.6.47-2.el6.x86_64
mysql-community-libs-5.6.47-2.el6.x86_64
[root@weiyu opt]# systemctl stop mysqld
```


2. 停止服务，逐一卸载 remove


```
systemctl stop mysqld 

yum -y remove mysql-community-server-5.6.47-2.el6.x86_64
……

```

3. 重新查看

```
rpm -qa | grep -i mysql
```

4. 按以上步骤安装

```
# 安装
```

### 4.启动和登录


> 1、启动

```
service mysqld start
```
> 2、停止

```
service mysqld stop
```
> 3、登录

```
mysql -u root -p
```
这种需设置密码，第一次可以不加-p 的参数；

### 5.配置及设置

4、设置其他人可以访问

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123' WITH GRANT OPTION;
```

5、设置字符串编码

```
show variables like "%char%";

SET character_set_client='utf8';
SET character_set_connection='utf8';
SET character_set_results='utf8';
```

6、使用数据库
```
show databases；
show tables；

查看mysql 版本
status;
select version();

```

7、修改密码

第一次直接可以通过 mysql -u root 登录进来

```
[root@localhost ~]# mysql -u root 
mysql>use mysql
mysql>update user set password=password("hicross@2016") where user="root";
mysql>flush privileges;
mysql>exit
```

8、防火墙 centos 7


```
systemctl stop firewalld.service
systemctl start firewalld.service

#单独加端口
firewall-cmd --zone=public --add-port=3306/tcp --permanent

```

> IP地址转换函数


```bash
SELECT INET_ATON('209.207.224.40');
SELECT INET_ATON('127.0.0.1'), 
insert INET_ATON('127.1');
```
```bash
主键 bigint unsigned
```


9、查看链接数 和设置

```bash
 show processlist;

show variables like 'max_connections';
(查可以看当前的最大连接数)

set global max_connections=1000;
(设置最大连接数为1000，可以再次查看是否设置成功)

set global max_connect_errors=1000;
（设置最大错误 连接数）
```

10、设置导入最大文件限制

```bash
 vim /etc/my.cnf
 # 增加
max_allowed_packet=400M

# 重启服务

service mysqld stop
service mysqld start

# 进入命令行
show variables like 'max_allowed_packet';

No connection. Trying to reconnect...
Connection id:    2
Current database: platformdb_1

+--------------------+-----------+
| Variable_name      | Value     |
+--------------------+-----------+
| max_allowed_packet | 419430400 |
+--------------------+-----------+
1 row in set (0.05 sec)
```

