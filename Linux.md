## Linux安装jdk

```linux
JAVA_HOME=/usr/local/jdk-11.0.17
PATH=$JAVA_HOME/bin:$PATH
```







## 开启指定端口

```linux
firewall-cmd --zone=public --add-port=8080/tcp --permanent //永久开放8080端口

firewall-cmd --reload //重新加载配置，使之生效

firewall-cmd --zone=public --list-ports//查看开放的端口

```





## 安装Mysql



### 安装

首先，尝试一下直接使用 yum 安装 MySQL

```text
yum install mysql-community-server
```

如果成功，表示不需要配置MySQL rpm 源信息，直接就安装完成了

但是，如果出现以下错误：

```text
Loading mirror speeds from cached hostfile
没有可用软件包 mysql-community-server。
错误：无须任何处理
```

表示我们没有添加安装包的源信息，需要安装 MySQL rpm 源信息



### 安装 MySQL rpm 源信息

打开 [http://dev.mysql.com/downloads/repo/yum/](https://link.zhihu.com/?target=http%3A//dev.mysql.com/downloads/repo/yum/)

![img](https://pic1.zhimg.com/80/v2-c7386b38916fc9958490b651a93d0860_720w.webp)

根据你的系统版本，选择对应的安装包，例如我的是CentOS 7.5，这个系统的Linux内核是 Linux 7，所以我选择了红框内的地址，大家依次类推。

拼接下载地址头：[http://dev.mysql.com/get/](https://link.zhihu.com/?target=http%3A//dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm)，得到以下地址

```text
 http://dev.mysql.com/get/mysql80-community-release-el7-7.noarch.rpm
```

使用 wget + 刚才拼接的地址，下载安装包源信息

```text
wget  http://dev.mysql.com/get/mysql80-community-release-el7-7.noarch.rpm
```

rpm 安装源信息

```text
rpm -ivh mysql80-community-release-el7-7.noarch.rpm
```



### 安装

再尝试使用 yum 安装MySQL

```text
yum install mysql-community-server
```

安装过程中，会提示让我们确认，一律输入 `y` 按回车即可

安装完成后，yum会自动覆盖自带的mariaDB，所以不需要我们手动卸载它



### 检查安装是否成功

检查一下刚才的安装是否成功

```text
rpm -qa | grep mysql
```

CetOS中自带了mariadb，与mysql冲突

```
rpm -qa | grep mariadb //查询是否安装mariadb
rpm -e --nodeps mariadb-libs-5.5.68-1.el7.x86_64 //卸载软件
```

```text
# 启动
systemctl start mysqld

# 第一次启动后，可以查看mysql初始化密码
grep 'temporary password' /var/log/mysqld.log

# 重启
systemctl restart mysqld

# 停止
systemctl stop mysqld

#查看状态
systemctl status mysqld

#开机启动
systemctl enable mysqld
systemctl daemon-reload
```



### 修改密码策略

```text
1、先按照mysql的要求，修改一次密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Root_123';

2、退出mysql
exit

3、重新登录mysql
mysql -u root -p'Root_123'

4、安装密码验证插件
install plugin validate_password soname 'validate_password.so';

5、查看是否启用了插件
select plugin_name, plugin_status from information_schema.plugins where plugin_name like 'validate%';
+-------------------+---------------+
| plugin_name       | plugin_status |
+-------------------+---------------+
| validate_password | ACTIVE        |
+-------------------+---------------+
输出这样的内容，表示成功启用
```





查看验证策略的键、值信息

```text
SHOW VARIABLES LIKE 'validate_password%';

对于高版本的mysql，例如mysql 8，验证策略的key，是 validate_password.xxx
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password.check_user_name    | ON    |
| validate_password.dictionary_file    |       |
| validate_password.length             | 8     |
| validate_password.mixed_case_count   | 1     |
| validate_password.number_count       | 1     |
| validate_password.policy             | MEDIUM|
| validate_password.special_char_count | 1     |
+--------------------------------------+-------+


对于低版本的mysql，例如mysql 5.7，验证策略的key，是 validate_password_xxx
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password.check_user_name    | ON    |
| validate_password.dictionary_file    |       |
| validate_password.length             | 8     |
| validate_password.mixed_case_count   | 1     |
| validate_password.number_count       | 1     |
| validate_password.policy             | MEDIUM|
| validate_password.special_char_count | 1     |
+--------------------------------------+-------+
```



我们修改密码策略和密码长度

我的策略信息的 key ，是 `validate_password.xxx`这个格式的，所以按照如下进行设置

```text
设置密码校验策略为：0（只验证密码长度）
set global validate_password.policy=0;

设置密码最低长度=N，例如设置密码最低长度=6，也就是密码最少要设置6个字符及以上
set global validate_password.length=6;
```



好了，现在密码就可以按照你刚才配置的策略，来进行设置密码了

```text
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```



### 开放 root 账户远程登录

```text
# 登录
mysql -u root -p'123456'

# 如果你的数据库是 mysql 8 及以上
# 1、进入数据库
use mysql
# 2、修改user表
update user set host='%' where user='root';

# mysql 5.7 及之前，执行这行代码即可
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '你的密码' WITH GRANT OPTION;

# 重载授权表
FLUSH PRIVILEGES;

# 退出
exit

# 重启
systemctl restart mysqld
```

## 启动项目



nohup命令:英文全称no hang up (不挂起)，用于不挂断地运行指定命令，退出终端不会影响程序的运行
语法格式: nohup Command [Arg ... ][&]
参数说明:
Command:要执行的命令
Arg: -些参数，可以指定输出文件
&:让命令在后台运行

### 后台启动项目

```shell
nohup java -jar reggie_takeout-1.0-SNAPSHOT.jar &>reggie.log &
```

后台运行java -jar命令，并将日志输出到hello.log文件

### 关闭项目

```shell
ps -ef | grep java

kill -15 [进程id]
```



### 复制数据库

```sql
mysqldump -u root -p yiibaidb > d:\database_bak\yiibaidb.sql
Enter password: **********
```

```sql
mysql -u root -p reggie < reggie.sql
Enter password: **********
```

## 安装Redis

redis 的安装极为简单，使用 CentOS 7 自带的 yum 安装即可

```shell
yum install redis
```



### 启动等操作

```text
# 启动
systemctl start redis

# 查看状态
systemctl status redis

# 停止
systemctl stop redis

# 重启
systemctl restart redis
```



### 查看版本号

```text
redis-server -v
redis-server --version

输出
Redis server v=3.2.12 sha=00000000:0 malloc=jemalloc-3.6.0 bits=64 
build=7897e7d0e13773f
```



### 验证安装

安装 `redis`，都会附带安装 `redis-cli`，这是 Redis 的客户端工具

我们可以使用它，验证 redis 是否正常运行

```text
# 进入客户端
redis-cli

# 成功连接并且进入客户端
127.0.0.1:6379> 

# 添加数据
127.0.0.1:6379> set test-key test-value
OK

# 查询数据
127.0.0.1:6379> get test-key
"test-value"

# 删除数据
127.0.0.1:6379> del test-key
(integer) 1

# 退出客户端
127.0.0.1:6379> exit
```

能正常连接，且操作数据，表示安装成功。



### 开启远程连接

```text
# 进入并编辑redsi.conf文件
vim /etc/redis.conf

# 找到bind 127.0.0.1 将其注释（#）或修改为 bind 0.0.0.0
bind 0.0.0.0

# 为了安全，我们需要开启密码保护
# 找到 # requirepass xxx ，默认是被注释的，将注释符号去掉，并添加自己的密码
# 找到它要往下拉很久，我们也可以直接在 bind 0.0.0.0 下面添加一个 requirepass xxx
requirepass 123456

# 按 esc 键，并输入 :wq 保存

# 重启，让配置生效
systemctl restart redis
```
