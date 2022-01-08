# 读取配置文件

## 1. 在`Resource`文件夹中添加配置文件

- 添加`META-INF`文件夹并添加`additional-spring-configuration-metadata.json`文件

  在json文件中添加如下内容：

  ```json
  {
  "properties": [{
    "name": "application.name",
    "description": "Default value is artifactId in pom.xml.",
    "type": "java.lang.String"
  },
    {
      "name": "application.version",
      "description": "Default value is version in pom.xml.",
      "type": "java.lang.String"
    },
  
    {
      "name": "developer.name",
      "description": "the developer name.",
      "type": "java.lang.String"
    },
    {
      "name": "developer.phone",
      "description": "the developer phone number.",
      "type": "java.lang.String"
    },
    {
      "name": "developer.email",
      "description": "the developer email.",
      "type": "java.lang.String"
    },
    {
      "name": "developer.qq",
      "description": "the developer qq.",
      "type": "java.lang.Integer"
    },
    {
      "name": "developer.website",
      "description": "the developer website.",
      "type": "java.lang.String"
    }
  ]
  ```

- 添加程序启动配置文件 `application.yml`

  ```yml
  # 配置使用的端口号和 网址方位的路径如：localhost:8088/demo/property
  server:
    port: 8088
    servlet:
      context-path: /demo
  #切换生产环境和测试环境
  spring:
    profiles:
      active: dev
  ```

  

- 添加测试环境的程序启动配置文件`application-dev.yml`

  ```yml
  application:
    name: 测试环境 @artifactId@
    version: 测试环境 @version@
  
  developer:
    name: 测试环境 cheng
    phone: 测试环境 19900000129
    email: 测试环境 123456@email.com
    website: 测试环境 www.baidu.com
    qq:  123456
  ```

- 添加生成环境的程序启动配置文件`application-prod.yml`

  ```yml
  application:
    name: 生产环境 @artifactId@
    version: 生产环境 @version@
  
  developer:
    name: 生产环境 cheng
    phone: 生产环境 19900000129
    email: 生产环境 123456@email.com
    website: 生产环境 www.baidu.com
    qq:  123456
  ```

1. `json`文件中定义的是，将要读取的属性的类型、名称和相关的描述信息；

2. `yml`文件中定义的各个属性字段对应的内容；

3. `yml`文件中引用其内部的字段使用 `${value}`

4. `yml`文件中读取`pom.xml`中的属性使用`@version@`，并且`pom.xml`需要额外配置`build`节点

   ```xml
       <build>
           <finalName>property</finalName>
           <!--使用maven渲染yml和property-->
           <resources>
               <resource>
                   <directory>src/main/resource</directory>
                   <filtering>true</filtering>
               </resource>
           </resources>
       </build>
   ```

   

## 2. 在`pom.xml`文件中添加需要用到的依赖

```xml
  <dependencies>
        <!--web项目依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--读取配置文件的依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
        </dependency>

    </dependencies>
```



## 3. 编写属性对应的java类

```java
/**
 * 读取配置文件中的属性
 * 方式一
 * 直接使用属性名称
 *
 * App的属性
 */

@Component
public class ApplicationProperty {
    @Value("${application.name}")
    private String name;
    @Value("${application.version}")
    private String version;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getVersion() {
        return version;
    }

    public void setVersion(String version) {
        this.version = version;
    }

}



/**
 * 开发人员信息
 *  读取配置文件中的属性
 *  方式二，限制属性前缀
 */
@ConfigurationProperties(prefix = "developer")
@Component
public class DeveloperProperty {

    private String name;
    private String phone;
    private String email;
    private String website;
    private int qq;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getWebsite() {
        return website;
    }

    public void setWebsite(String website) {
        this.website = website;
    }

    public int getQq() {
        return qq;
    }

    public void setQq(int qq) {
        this.qq = qq;
    }


}

```



## 4. 编写接口controller

```java
@RestController
public class PropertiesController {

    @Autowired
    private ApplicationProperty applicationProperty;
    @Autowired
    private DeveloperProperty developerProperty;

    @GetMapping("/property")
    public String index(){
      Map<String,Object> map = new HashMap<>();
      map.put("application.name",applicationProperty.getName());
      map.put("application.version",applicationProperty.getVersion());
      map.put("developer.name",developerProperty.getName());
      map.put("developer.phone",developerProperty.getPhone());
      map.put("developer.email",developerProperty.getEmail());
      map.put("developer.qq",developerProperty.getQq());

        JSONObject jsonObject = new JSONObject(map);
        return jsonObject.toString();
    }

}

```

读取的属性转为json返回



## 5. idea中运行项目并验证

- application类的内容

  ```java
  @SpringBootApplication
  public class PropertiesApplication extends SpringBootServletInitializer {
      public static void main(String[] args) {
          SpringApplication.run(PropertiesApplication.class,args);
      }
  
      /**
       * SpringBoot项目打包时，需要在Application类中继承 SpringBootServletInitializer类，并重写configure方法
       * @param builder
       * @return
       */
      @Override
      protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
          return builder.sources(this.getClass());
      }
  }
  
  ```

- 浏览器中输入地址`localhost:8088/demo/property`验证

  浏览器中的返回值:

  ```json
  {"application.version":"测试环境 @version@","developer.phone":"测试环境 19900000129","developer.name":"测试环境 cheng","developer.qq":123456,"developer.email":"测试环境 123456@email.com","application.name":"测试环境 @artifactId@"}
  ```

  

  

