# Centos常用命令

## 1. 检查系统版本

```shell
cat /etc/redhat-release
#输出结果：CentOS Linux release 7.5.1804 (Core) 
```



## 2. 上传文件到Centos中

服务器地址：root@119.45.218.209

### 1. 上传命令

```shell
scp mysql8.tar root@119.45.218.209:~/go
```

使用scp 命令上传本地mysql8.tar文件到服务器的~/go目录下

### 2. 下载命令

```shell
scp root@119.45.218.209:~/go/mysql8.tar ~/downloads/
```

调换上传命令的两个参数位置即可



## 3. 服务相关

### 1. 检查服务状态

```shell
service mysql status
```

### 2. 启动服务

```shell
service mysql start
```

### 3. 停止服务

```shell
service mysql stop
```

### 

