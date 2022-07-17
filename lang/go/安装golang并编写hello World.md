# 安装golang和编写hello world

### 下载安装
- Homebrew安装(推荐)
1. 升级和卸载
```
///安装
brew install go
///升级
brew upgrade go
///卸载
brew uninstall go
```
2. GOROOT路径
```
/usr/local/Cellar/go/<go 版本号>/libexec
```
- 官网下载
[golang官网地址](https://golang.org)
1. 卸载删除以下内容，升级需要卸载后重装
```
sudo rm -rf /usr/local/go
sudo rm -rf /etc/paths.d/go
```
2. GOROOT路径
```
/usr/local/go
```
- 验证是否安装成功
```
go version
```
输出以下内容表示成功
```
go version go1.15.6 darwin/amd64
```
- 配置环境变量
```
///查看go环境环境变量
go env

///用于存放依赖包及编译文件，比较随意，只要不和GOROOT重名即可，官方禁止这一行为。
go env -w GOPATH=/Users/cheng/Library/go
///设置代理后，在未翻墙的情况下，打开VSCode后gopls工具的加载会很快。
go env -w GOPROXY=https://goproxy.cn,direct
```

### terminal编写hello world
- 创建文件夹并进入该文件夹
```
mkdir hello
cd hello

```
- 创建hello.go文件
```
touch hello.go
```
- 打开并编写hello world代码
```
open hello.go

///输入以下代码并保存文件

package main

import "fmt"

func main(){
    ///输出hello world
	fmt.Println("Hello, World");
}

```
- terminal运行hello.go文件
```
go run hello.go
///result: Hello, World
```

- 查看更多go命令
```
go help
```
### 导入外部库并追踪依赖关系
- 导入外部库
```
package main

import "fmt"
///导入外部库
import "rsc.io/quote"

func main(){
	fmt.Println(quote.Go());
}

```
- 把代码放入模块中
```
go mod init hello

///输出：go: creating new go.mod: module hello
```
- 运行代码
```
go run hello.go

///等待下载外部库(需要VPN开启全局代理，如果超时需要设置terminal本地代理)
///输出：
go: finding module for package rsc.io/quote
go: downloading rsc.io/quote v1.5.2
go: found rsc.io/quote in rsc.io/quote v1.5.2
go: downloading rsc.io/sampler v1.3.0
go: downloading golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
Don't communicate by sharing memory, share memory by communicating.
```

### vs code安装插件
1. 插件列表
- Go: VSCode官方提供的插件，可以使用Go的大部分工具。
2. 插件配置
- 在VSCode配置文件中添加 "go.useLanguageServer": true ，保存后，右下角会提示重启VSCode
- 重启后右下角会提示安装gopls，点击安装即可。

[点击此处查看原文和其他配置](https://www.lagou.com/lgeduarticle/115939.html)



### 目前使用go module进行依赖管理工具

#### 新项目使用go module

- 执行`go mod init 项目名`命令，在项目文件夹下创建一个go.mod文件

- 手动编辑go.mod中的require依赖项或执行 `go get` 自动发现、维护依赖

  

#### 已有项目

- 在项目目录项执行`go mod init`，生成一个`go.mod` 文件
- 执行`go get`，查找并记录当前项目的依赖，同时生成一个go.sum记录每个依赖库的版本和哈希值