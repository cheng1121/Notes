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





### 安装sequel pro查看本地数据库

- 下载sequel pro

  地址：[http://www.sequelpro.com/](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.sequelpro.com%2F)

- 配置mysql

  - 进入系统偏好设置
  - MySQL
  - start MySQL Server
  - 点击initialize Database
  - 输入数据库密码
  - 选择Use Legacy Password Encryption
  - 点击ok保存配置

- 配置sequel pro

  - name：可选
  - Host：填写127.0.0.1
  - Username：root
  - Password：安装时MySQL时输入的密码
  - Database：不填
  - Port：使用默认的端口号3306

  

## 服务器安装MySQL

