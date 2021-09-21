# AS环境配置


## 一. 北极狐版本环境配置
由于AS新版本北极狐的java版本提升到了11，导致使用java 1.8版本报错。有以下几种解决办法：

> ps:使用jdk 16时，android原生正常，flutter中报错

### 1. 升级jdk 版本到11

- 下载jdk11并配置环境变量
- 修改gradle版本到7.0以上
- 修改kotlin版本到1.5.21版本
- 旧项目需要打开 open module setting，并在sdks中，选择java 11版本。如果没有则点击加号添加或者下载jdk11



### 2. 不升级jdk版本的办法(未验证)

- 在.gradle目录中新增一个全局gradle.properties文件

- 在文件内部添加以下内容，并重启AS以读取配置

  ```groovy
  org.gradle.java.home=<本地java目录>
  ```

  

