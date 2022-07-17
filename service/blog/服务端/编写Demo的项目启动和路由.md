# Demo项目启动

## `main`包编写test路由

```go
package main

import (
	"fmt"
	"net/http"

	"cheng.com/blog/pkg/setting"
	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()
	r.GET("/test", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "test",
		})
	})

	s := &http.Server{
		Addr:           fmt.Sprintf(":%d", setting.HTTPPort),
		Handler:        r,
		ReadTimeout:    setting.ReadTimeout,
		WriteTimeout:   setting.WriteTimeout,
		MaxHeaderBytes: 1 << 20,
	}
	s.ListenAndServe()
}

```



## 运行项目

```shell
go run main.go
```

查看命令行是否显示：

```shell
API server listening at: 127.0.0.1:16054
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /test                     --> main.main.func1 (3 handlers)

```



使用`curl 127.0.0.1:8000/test`检查是否返回`{"message":"test"}`

## 抽离路由在单独的`routers`包下

- 在`routers`目录下添加`router.go`文件

  ```go
  package routers
  
  import (
  	"cheng.com/blog/pkg/setting"
  	"github.com/gin-gonic/gin"
  )
  
  //InitRouter 初始化路由
  func InitRouter() *gin.Engine {
  	r := gin.New()
  	r.Use(gin.Logger())
  	r.Use(gin.Recovery())
  
  	gin.SetMode(setting.RunMode)
  
  	r.GET("/test", func(c *gin.Context) {
  		c.JSON(200, gin.H{
  			"message": "test",
  		})
  	})
  
  	return r
  
  }
  	
  ```

  

- 修改`main.go`文件中的`main`函数

  ```go
  package main
  
  import (
  	"fmt"
  	"net/http"
  
  	"cheng.com/blog/pkg/setting"
  	"cheng.com/blog/routers"
  )
  
  func main() {
  	r := routers.InitRouter()
  
  	s := &http.Server{
  		Addr:           fmt.Sprintf(":%d", setting.HTTPPort),
  		Handler:        r,
  		ReadTimeout:    setting.ReadTimeout,
  		WriteTimeout:   setting.WriteTimeout,
  		MaxHeaderBytes: 1 << 20,
  	}
  	s.ListenAndServe()
  }
  
  ```

- 重启服务，使用`curl 127.0.0.1:8000/test`验证结果

