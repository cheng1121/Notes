# HelloWorld

## 1. 安装

在pom.xml文件中添加,如下节点继承String Boot 的默认配置

```xml
 <parent>
      <artifactId>spring-boot-starter-parent</artifactId>
      <groupId>org.springframework.boot</groupId>
      <version>2.5.0</version>
 </parent>
```

- `spring-boot-starter-parent`是一个特殊的启动器，提供有用的`Maven`默认值；还提供`dependency-management`部分，以便可以省略`version`标签
- parent本身不提供依赖，需要另外添加依赖



## 2. 添加依赖

在`pom.xml`添加web开发依赖

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
</dependencies>
```



## 3. 编写程序启动代码

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@EnableAutoConfiguration
public class Application {
    public static void main(String[] args) throws  Exception{
        SpringApplication.run(Application.class,args);
    }

    @RequestMapping("/")
    String example(){
      return "hello world";
    }

}
```

使用maven命令启动web项目

```shell
mvn spring-boot:run
```

或者在idea中运行main方法左侧的运行按钮

出现如下画面则表示运行成功：

```shell
Connected to the target VM, address: '127.0.0.1:50082', transport: 'socket'

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.0)

2021-06-12 11:23:02.207  WARN 34290 --- [           main] o.s.boot.StartupInfoLogger               : InetAddress.getLocalHost().getHostName() took 5002 milliseconds to respond. Please verify your network configuration (macOS machines may need to add entries to /etc/hosts).
2021-06-12 11:23:07.215  INFO 34290 --- [           main] Application                              : Starting Application using Java 1.8.0_201 on lichengdeMacBook-Pro.local with PID 34290 (/Users/cheng/projects/java/demo/main/target/classes started by cheng in /Users/cheng/projects/java/demo)
2021-06-12 11:23:07.216  INFO 34290 --- [           main] Application                              : No active profile set, falling back to default profiles: default
2021-06-12 11:23:07.831  INFO 34290 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2021-06-12 11:23:07.837  INFO 34290 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-06-12 11:23:07.838  INFO 34290 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.46]
2021-06-12 11:23:07.895  INFO 34290 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-06-12 11:23:07.895  INFO 34290 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 648 ms
2021-06-12 11:23:08.207  INFO 34290 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-06-12 11:23:08.214  INFO 34290 --- [           main] Application                              : Started Application in 21.31 seconds (JVM running for 26.774)

```

此时，在浏览器地址栏输入127.0.0:8080即可显示hello world文字

