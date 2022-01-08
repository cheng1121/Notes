# MySQL安装

## 下载MySQ安装包

安装包下载地址[MySql](https://dev.mysql.com/downloads/mysql/)

下载好后双击安装，一路同意安装就可,

最后会让输入数据库密码

使用`mysql -u root -p`登陆mysql，会提示输入密码，就是安装时最后一步让输入的密码

如果提示myslq命令未找到则需要配置环境变量：

```shell
#mysql环境变量
export PATH=$PATH:/usr/local/mysql/bin
```

### 查看MySQL端口号

```shell
# 登陆mysql
mysql -u root -p
#查看端口号
show variables like "port";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| port          | 3306  |
+---------------+-------+
1 row in set (0.00 sec)
```





## centos中服务器安装MySQL

### 1.下载

下载mysql压缩包,如果服务器下载速度慢，可以本地下载后再上传到服务器上

使用wget 命令下载mysql8.0版本

```shell
wget https://dev.mysql.com/get/downloads/mysql/mysql-8.0.26-linux-glibc2.12-x86_64.tar.xz
```

解压压缩包:

```shell
tar -zxvf 压缩包名
```

如果解压报错：

```shell
gzip: stdin: not in gzip format
```

则需要使用`file`命令查看文件类型

```shell
file 压缩包名
结果：mysql-8.0.26-linux-glibc2.12-x86_64.tar.xz: XZ compressed data
```

从结果看，此压缩包是xz类型的，那么解压命令中需要把`z`去掉

```shell
tar -xvf 压缩包名
```

解压后的文件夹放入`/usr/local`目录中

``` shell
mv mysql-8.0.26-linux-glibc2.12-x86_64 /usr/local/mysql
```

将`mysql/bin`加入环境变量

```shell
vim /etc/profile
#按i编辑profile文件,在文件最后添加如下内容
export PATH=$PATH:/usr/local/mysql/bin
#输入:wq保存并退出vim模式
:wq
#让新输入的环境变量生效
source /etc/profile
```

### 2. 安装mysql依赖libaio(非必须)

```shell
yum install libaio
```

### 3. 创建mysql用户

不需要登录的一个系统账号，启动MySql服务时会使用该账户

```shell
#创建用户组
groupadd mysql
#mysql用户组中添加用户mysql
useradd -g mysql mysql
#mysql用户授权
chown -R mysql.mysql /usr/local/mysql/
```

## 4. 添加配置文件

在/etc/目录下添加或者编辑my.cnf文件，添加如下内容

```shell
[mysqld]
#设置3306端口
port=3306
#设置mysql安装目录
basedir=/usr/local/mysql
#设置mysql数据库的数据存放目录
datadir=/usr/local/mysql/data
#允许最大连接数
max_connections=200
#允许连接失败的次数。为了防止有人从该主机视图攻击数据库系统
max_connect_errors=10
#服务端使用的字符集默认为utf8
character-set-server=utf8mb4
#创建新表时使用的默认存储引擎
default-storage-engine=INNODB
#默认使用mysql_native_password插件认证
default_authentication_plugin=mysql_native_password

user=mysql
lower_case_table_names=1
default-time-zone='+8:00'
sql_mode=NO_AUTO_VALUE_ON_ZERO,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE

[client]
#设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8mb4
```



## 5. 初始化数据库

```shell
mysqld --initialize --console
```

如果报如下错误：

```shell
[ERROR] [MY-010457] [Server] --initialize specified but the data directory has files in it. Aborting.
[ERROR] [MY-013236] [Server] The designated data directory /var/lib/mysql/ is unusable. You can remove all files that the server added to it.

```

则需要进入`/var/lib/mysql`文件夹内删除内部的全部文件

## 6. 建立mysql服务，并开机自启动

建立mysql服务

```shell
ln -s /usr/local/mysql/bin/mysql /usr/bin
```

将新建立的服务加入自启动

```shell
cp -a ./support-files/mysql.server /etc/init.d/mysql
#添加权限
chmod +x /etc/init.d/mysql
#加入自启动
chkconfig --add mysql
#检查服务是否生效
chkconfig --list mysql
#启动服务
service mysql start
#停止服务
service mysql stop
#查看服务启动状态
service mysql status
```



## 7. 安全模式登陆MySQL,并修改密码

```shell
#安全模式启动mysql
mysqld_safe --skip-grant-tables
```

打开新的命令窗口登陆mysql

```shell
mysql -u root
```

第一次登陆会要求修改默认密码

```shell
ALTER USER 'root'@'localhost' IDENTIFIED with mysql_native_password BY 'root';
```

修改host为通配符%,允许远程访问

```shell
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
```

修改密码和host成功后，需要调用下方命令刷新权限列表

```shell
FLUSH PRIVILEGES;
```

## 8. 本地客户端连接服务器mysql

navicat 连接服务器

连接 -> 输入连接名 -> 主机ip -> 主机端口号 --> 用户名 --> 密码 --> 保存密码 -> 测试连接 --> 保存

