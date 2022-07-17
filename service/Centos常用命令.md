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

## 4. 用户与用户组操作

### 1. 查看用户

```shell
# w 或者 who 查看当前登录用户
w
who
# 查看自己的用户名
whoami
#查看单个用户信息
finger root # 腾讯云centos7上无效
id root   #已验证有效
#查看用户登录记录
last #登录成功的记录
lastb #登录不成功的记录
#查看所有用户

```

### 2. 新增用户

```shell
[root@VM-0-4-centos ~]# useradd --help
用法：useradd [选项] 登录
      useradd -D
      useradd -D [选项]

选项：
  -b, --base-dir BASE_DIR	新账户的主目录的基目录
  -c, --comment COMMENT         新账户的 GECOS 字段
  -d, --home-dir HOME_DIR       新账户的主目录
  -D, --defaults		显示或更改默认的 useradd 配置
 -e, --expiredate EXPIRE_DATE  新账户的过期日期
  -f, --inactive INACTIVE       新账户的密码不活动期
  -g, --gid GROUP		新账户主组的名称或 ID
  -G, --groups GROUPS	新账户的附加组列表
  -h, --help                    显示此帮助信息并推出
  -k, --skel SKEL_DIR	使用此目录作为骨架目录
  -K, --key KEY=VALUE           不使用 /etc/login.defs 中的默认值
  -l, --no-log-init	不要将此用户添加到最近登录和登录失败数据库
  -m, --create-home	创建用户的主目录
  -M, --no-create-home		不创建用户的主目录
  -N, --no-user-group	不创建同名的组
  -o, --non-unique		允许使用重复的 UID 创建用户
  -p, --password PASSWORD		加密后的新账户密码
  -r, --system                  创建一个系统账户
  -R, --root CHROOT_DIR         chroot 到的目录
  -s, --shell SHELL		新账户的登录 shell
  -u, --uid UID			新账户的用户 ID
  -U, --user-group		创建与用户同名的组
  -Z, --selinux-user SEUSER		为 SELinux 用户映射使用指定 SEUSER
  
  
  #新增用户
  useradd test
  #设置密码
  passwd test
```

### 3. 修改用户

```shell
# 将test用户的登录目录改成/home/test，并加入test2组，注意这里是大G。
Usermod -d /home/test -G test2 test
gpasswd -a test test2 #将用户test加入到test2组
gpasswd -d test test2 #将用户test从test2组中移出
```

### 4. 用户组

```shell
#创建组
groupadd test
#修改组
groupmod -n test2 test
#删除用户组
groupdel test
#查看当前登录用户所在用户组
groups
#查看指定用户所在用户组
groups test
#查看全部用户组
cat /etc/group
#没有 /etc/group文件的系统查看方法
cat /etc/passwd |awk -F [:] '{print $4}' |sort|uniq | getent group |awk -F [:] '{print $1}'
```









