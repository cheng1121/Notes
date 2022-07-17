# 编写models包

## 在`models`目录下添加`models.go`文件

编写导入`gorm`，读取数据库配置，并连接数据库

```go
package models

import (
	"fmt"
	"log"
  _ "github.com/jinzhu/gorm/dialects/mysql"
	"cheng.com/blog/pkg/setting"
	"github.com/jinzhu/gorm"
)

var db *gorm.DB

//Model 数据库字段基类 ``作用为不转义纯字符串
// 字段类型是json
type Model struct {
	ID         int `gorm:"primary_key" json:"id"`
	CreateOn   int `json:"created_on"`
	ModifiedOn int `json:"modified_on"`
}

///连接数据库并配置
func init() {
	var (
		err                                               error
		dbType, dbName, user, password, host, tablePrefix string
	)
	sec, err := setting.Cfg.GetSection("database")
	if err != nil {
		log.Fatal(2, "Fail to get section 'database': $v", err)
	}

	dbType = sec.Key("TYPE").String()
	dbName = sec.Key("NAME").String()
	user = sec.Key("USER").String()
	password = sec.Key("PASSWORD").String()
	host = sec.Key("HOST").String()
	tablePrefix = sec.Key("TABLE_PREFIX").String()
	//打开数据库
	db, err = gorm.Open(dbType, fmt.Sprintf("%s:%s@tcp(%s)/%s?charset=utf8&parseTime=True&loc=Local",
		user,
		password,
		host,
		dbName,
	))
	if err != nil {
		log.Println(err)
	}
	///默认添加前缀
	gorm.DefaultTableNameHandler = func(db *gorm.DB, defaultTableName string) string {
		return tablePrefix + defaultTableName
	}
  //全局禁用表名复数，此时表名默认为结构体首字母小写形式
	db.SingularTable(true)
	db.LogMode(true)
	//设置最大连接数
	db.DB().SetMaxIdleConns(10)
	//设置最大打开数
	db.DB().SetMaxOpenConns(100)
}

//CloseDB 关闭数据库
func CloseDB() {
	defer db.Close()
}

```

- 由于gorm中，默认表名是结构体名的复数形式，列名是字段名的蛇形小写

- 通过`db.SingularTable(true)`函数全局禁用表名复数，此时默认为结构体名首字母小写形式

- 通过实现`TableName`函数，为结构体加上`TableName`方法可以显式的返回自定义的表名，也可以通过`DefaultTableNameHandler`来指定统一的项目表名

- 通过Table API显式声明

  ```go
  db.Table("table_name").CreateTable(&YourStruct{})
  ```

- 结构体的字段到数据表字段的默认映射规则是用下划线分隔每个大写字母开头的单词，即使上述规则都不遵守，也必须将结构体字段使用大写字母命名



## 自动创建表

查看地址：https://www.tizi365.com/archives/982.html

