# mysql中没有root用户问题



免密模式登录mysql后发现没有root用户,需要创建root用户

首先基本配置：

```shell
#打卡mysql的配置文件
vim /etc/my.cnf 
skip-grant-tables  #在[mysqld]下面添加这一行，忽略权限表
```

免密模式启动mysql

```shell\
mysqld_safe --skip-grant-tables
```

在新命令行窗口中登录mysql

```shell
mysql -u root
```

创建root用户:

```sql
create user 'root'@'localhost' identified by 'cheng1232'
```

此时有可能报如下错误: 未报错直接给root用户赋予权限

```shell
ERROR 1290 (HY000): The MySQL server is running with the --skip-grant-tables option so it cannot execute this statement
```

刷新权限

```sql
flush privileges;
```

再次执行创建root用户语句后，可能会如下错误:

```shell
mysql> create user 'root'@'localhost' identified by 'cheng123';
ERROR 1396 (HY000): Operation CREATE USER failed for 'root'@'localhost'
```

删除root用户,后再次创建root用户

```sql
drop user 'root'@'localhost'
```

此时应该不会再报错,赋予root权限

```sql
grant all privileges on *.* to 'root'@'localhost' with grant option; # 赋予所有库所有表操作权限
flush privileges; #刷新权限
```

退出mysql后再mysql的配置文件中去掉免密登录的配置

重启mysql服务，正常登录mysql

```shell
mysql -u root -p
```

需要修改root用户可以远程登录

```sql
update user set host='%' where user='root';
```

刷新权限即可

```sql









