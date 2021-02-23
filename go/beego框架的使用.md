# beego框架的使用

## 安装beego

- 在项目目录下执行`go get-u github.com/beego/bee/v2`命令来安装bee工具

- 配置go的环境变量

  ```shelll
  #go语言安装主根目录
  export GOROOT=/usr/local/go #替换你的目录
  #GOPATH 是自己的go项目路径，自定义设置
  export GOPATH=/Users/ding/go_workspace #替换你的目录
  #GOBIN 当我们使用go install命令编译后并且安装的二进制程序目录
  export GOBIN=$GOPATH/bin
  # 启用 Go Modules 功能
  export GO111MODULE=on
  # 配置 GOPROXY 环境变量
  export GOPROXY=https://goproxy.cn,direct
  export PATH=$PATH:$GOROOT/bin:$GOBIN
  ```

  

- 执行`bee version`查看是否安装成功

## 使用bee工具新建项目

- 新建web项目

  执行`bee new 项目名`命令创建一个web项目

- 新建api项目

  执行`bee api 项目名`命令创建一个api项目

- 运行项目

  在vs code 中使用f5运行,然后浏览器中

