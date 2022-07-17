# 在centos中安装Go

## 下载安装包

- 下载 [go语言中文网](https://studygolang.com/dl)

  ```shell 
  wget https://studygolang.com/dl/golang/go1.15.8.linux-amd64.tar.gz
  ```

- 解压

  ```shell
  tar -zxvf go1.15.8.linux-amd64.tar.gz	
  ```

- 移动go/文件夹到 /usr/local中

  ```shell
  mv go/ /usr/local
  ```

- 配置环境变量 `/etc/profile`

  使用`vi /etc/profile`打开profile文件

  进入vi 命令模式,按下i进入插入模式 ，输入如下环境变量

  ```shell
  export GOROOT=/usr/local/go
  export PATH=$PATH:$GOROOT/bin
  ```

  使用`:`进入底线命令模式

  - w:保存文件
  - q:退出vi模式

  使用`source /etc/profile`命令让环境变量生效

- 检查是否安装成功

  ```shell
  go version
  #结果：go version go1.15.8 linux/amd64
  ```

  