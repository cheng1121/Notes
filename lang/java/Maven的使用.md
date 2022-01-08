# Maven的使用

## 1. 安装Maven

使用命令安装：

```shell
brew install maven
```

验证安装是否成功:

```shell
mvn -version
```



## 2. Maven项目结构

```shell
├─main
|  ├─pom.xml
|  ├─src
|  |  ├─test
|  |  |  ├─java
|  |  ├─main
|  |  |  ├─resources
|  |  |  ├─java
```



## 3. Maven的常用命令

- 查看依赖树

  ```shell
  mvn dependency:tree
  ```

- 启动`spring-boot`程序

  ```shell
  mvn spring-boot:run
  ```

- 