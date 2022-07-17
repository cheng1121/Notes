# GoWebService

## 一、初始化go mod

- 创建项目根目录

  ```shell
  mkdir web_service
  ```

- 在项目根目录下初始化mod

  ```shell
  cd web_service
  go mod init cheng.com/web_service
  ```

- 按照提示执行如下命令

  ```shell
  go mod tidy
  ```

  

## 二、编写最简单的web服务器代码

项目根目录下创建main.go文件，并添加如下代码:

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"strings"
)

func main() {

	http.HandleFunc("/", sayHello)           //设置访问的路由
	err := http.ListenAndServe(":9090", nil) //设置监听的端口
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}

func sayHello(w http.ResponseWriter, r *http.Request) {
	r.ParseForm()
	fmt.Println(r.Form)
	fmt.Println("path", r.URL.Path)
	fmt.Println("scheme", r.URL.Scheme)
	fmt.Println(r.Form["url_long"])
	for k, v := range r.Form {
		fmt.Println("key:", k)
		fmt.Println("val", strings.Join(v, ""))
	}

	fmt.Println("hello web service")
	fmt.Fprintf(w, "hello web ") //由w把 hello web 输出到客户端

}

```



## 三、运行代码

- vscode 在main.go文件中按F5即可运行
- 浏览器中输入localhost:9090 则会输出 hello web



