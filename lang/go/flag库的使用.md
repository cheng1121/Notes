# Flag库的使用

## 一. 基本使用

### 1. 初始化项目

```shell
#创建目录 learn
mkdir learn
#移动到learn 目录下
cd learn
#初始化go 项目的go modules设置项目的模块路基
go init learn
# 创建main.go文件
touch main.go文件
```



### 2. 代码编写

打开`main.go`文件

```shell
code main.go
```

输入如下语句：

```go
package main
import(
"flag"
  "log"
)

func main(){
  var name string
//长命令的写法
	flag.StringVar(&name, "name", "flag库学习", "帮助信息")
	//短命令的写法
	flag.StringVar(&name, "n", "Flag库学习", "帮助信息")
	flag.Parse()

	log.Printf("name: %s", name)
}

```

### 3. 验证

输入如下命令：

```shell
go run main.go -name=ccc -n=成
# name: 成
```

### 4. 语法说明

```shell
-flag：仅支持布尔类型。
-flag x ：仅支持非布尔类型。
-flag=x：均支持
```



### 5. 子命令的实现

```go
package main

import (
	"flag"
	"log"
)

//变量
var name string

//程序入口
func main() {
	//将命令行解析为定义的标志
	flag.Parse()
	//局部变量，获取命令集合
	args := flag.Args()

	//获取命令的第一位，判断是什么命令
	switch args[0] {

	case "go":
		//go语言命令
		//返回带有指定名称和错误处理属性的空命令集：创建一个空命令集去支持子命令
		goCmd := flag.NewFlagSet("go", flag.ExitOnError)
		//执行子命令
		goCmd.StringVar(&name, "name", "Go 语言", "帮助信息")
		_ = goCmd.Parse(args[1:])
	case "php":
		phpCmd := flag.NewFlagSet("php", flag.ExitOnError)
		phpCmd.StringVar(&name, "n", "PHP 语言", "帮助信息")

		_ = phpCmd.Parse(args[1:])
	}

	log.Printf("name: %s", name)

}

```

验证结果：

```shell
go run main.go go -name=eddycjy
#name: eddycjy
```

