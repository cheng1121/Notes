# Android源码下载

[MAC系统使用repo下载Android源码并编译](https://yujianbin.github.io/2018/06/12/MAC系统使用repo下载android源码/)

## 安装Repo
### 创建bin目录，并且的添加到PATH中
```
mkdir ~/bin
PATH=~/bin:$PATH
```

### 下载Repo工具
使用清华大学开源镜像站时将： 将 https://android.googleapis.com/ 全部使用 https://aosp.tuna.tsinghua.edu.cn/ 代替即可。

```
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo 

chmod a+x ~/bin/repo
```



## 初始化repo
### 创建源码目录
```
mkdir -p ~/Android/AOSP/android10
```
参数 -p 创建多级目录

### 移动到该目录
```
cd ~/Android/AOSP/android10
```

### 初始化仓库
```
repo init -u https://android.googlesource.com/platform/manifest
```

