# MySql常用命令

## 一、基础

### 1.`show`语句

- 查看表中的所有字段以及类型

```sql
show columns form blog;
#可以简写为:
describe blog;
```

- 显示广泛的服务器状态

  ```sql
  show status;
  ```

- 显示建库语句

  ```sql
  show create database blog;
  ```

- 显示建表语句

  ```sql
  show create table blog;
  ```

- 显示所有用户的安全权限

  ```sql
  show grants;
  ```

- 显示错误或者警告信息

  ```sql
  show errors;
  show warnings;
  ```

- 查看全部的`show`语句

