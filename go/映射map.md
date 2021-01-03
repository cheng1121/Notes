# 映射
映射将键映射到值  
映射的零值为`nil`。`nil`映射既没有键，也不能添加键  
`make`函数会返回给定类型的映射，并将其初始化

```
m := make(map[string]int)
m["数值1"] = 100
fmt.Println(m["数值1"])

```
## 文法
```
var m = map[string]int{
	"1": 1,
	"2": 2,
}
fmt.Println(m)
结果：map[1:1 2:2]
```

## 修改映射
在映射 m 中插入或修改元素：
```
m[key] = elem
```
获取元素：
```
elem = m[key]
```
删除元素：
```
delete(m, key)
```
通过双赋值检测某个键是否存在：
```
elem, ok = m[key]
```
若 key 在 m 中，ok 为 true ；否则，ok 为 false。

若 key 不在映射中，那么 elem 是该映射元素类型的零值。

同样的，当从映射中读取某个不存在的键时，结果是映射的元素类型的零值。

注 ：若 elem 或 ok 还未声明，你可以使用短变量声明：
```
elem, ok := m[key]
```