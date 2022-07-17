# swaggo使用

## 1.安装

```shell
go install github.com/swaggo/swag/cmd/swag@latest
```

添加环境变量

```shell
#go环境变量
GO_HOME=/Users/cheng/go
PATH=$GO_HOME/bin:$PATH
#go path/bin目录添加到环境变量
#我的go path 路径为：/Users/cheng/go/path
PATH=$GO_HOME/path/bin:$PATH
```

 ## 2.生成接口文档

在项目根目录下使用如下命令：

```shell
swag init
```

会在docs目录下生成`docs.go`、`swagger.json`、`swagger.yaml`等文件



## 3.编写接口文档

1. main方法上添加如下内容用来生成接口文档：

   ```go
   // @title 博客系统
   // @version 1.0
   // @description 个人博客系统
   // @termsOfService  cheng
   // @host localhost:8000
   // @BasePath /api/v1
   ```

2. Controller层中的接口方法中添加说明

```go
// @Summary 删除标签
// @Produce  json
// @Param id path int true "标签 ID"
// @Success 200 {string} string "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1/tags/{tid} [delete]



// @Summary 更新标签
// @Produce  json
// @Param id path int true "标签 ID"
// @Param state body int false "状态" Enums(0, 1) default(1)
// @Param modified_by body string true "修改者" minlength(3) maxlength(100)
// @Success 200 {array} model.Tag "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1/tags/{tagId} [put]

// @Summary 更新标签
// @Produce  json
// @Param id path int true "标签 ID"
// @Param name body string false "标签名称" minlength(3) maxlength(100)
// @Param state body int false "状态" Enums(0, 1) default(1)
// @Param modified_by body string true "修改者" minlength(3) maxlength(100)
// @Success 200 {array} model.Tag "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1/tags/{eid} [put]


// @Summary 新增标签
// @Produce  json
// @Param name body string true "标签名称" minlength(3) maxlength(100)
// @Param state body int false "状态" Enums(0, 1) default(1)
// @Param created_by body string true "创建者" minlength(3) maxlength(100)
// @Success 200 {object} model.Tag "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1/tags [post]


// @Summary 获取多个标签
// @Produce  json
// @Param name query string false "标签名称" maxlength(100)
// @Param state query int false "状态" Enums(0, 1) default(1)
// @Param page query int false "页码"
// @Param page_size query int false "每页数量"
// @Success 200 {object} model.Tag "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1/tags [get]


// @Summary 获取文章详情
// @Produce  json
// @Param id path int true "文章id"
// @Success 200 {object} model.Tag "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/articles/:id [get]

// @Summary 获取文章列表
// @Produce  json
// @Param page query int false "页码"
// @Param page_size query int false "每页数量"
// @Success 200 {object} model.Article "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1//articles [get]


// @Summary 新增文章
// @Produce  json
// @Param name body string true "文章名称" minlength(3) maxLength(100)
// @Param desc body string true "文章简述" maxlength(200)
// @Param content body string true "文章内容"
// @Success 200 {object} model.Article "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1//articles [post]

// @Summary 删除文章
// @Produce  json
// @Param id path int true "文章 ID"
// @Success 200 {string} string "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1/tags/{id} [delete]

// @Summary 修改文章状态
// @Produce  json
// @Param id path int true "文章 ID"
// @Param state body int false "状态" Enums(0, 1) default(1)
// @Param modified_by body string true "修改者" minlength(3) maxlength(100)
// @Success 200 {array} model.Tag "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1/tags/{id} [put]
```





**关键:** 需要导入生成的docs包如： `_ "cheng.com/blog/docs"` ，否则网页会报错：

**failed to load API definition. Fetch error Internal Server Error doc.json**



## 3.查看生成的接口文档

http://127.0.0.1:8000/swagger/index.html

