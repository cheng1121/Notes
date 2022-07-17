# SpringBoot项目打包并运行

## `Application`类中配置

- 继承`SpringBootServletInitializer`类
- 重写`configure`方法

### 代码:

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



## `pom.xml`中的配置

```xml

    <build>
        <finalName>property</finalName>
        <!--使用SpringBoot的maven插件进行打包-->
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.cheng.properties.PropertiesApplication</mainClass>
                    <layout>JAR</layout>
                    <!--构建完整可执行程序，可以直接运行-->
                    <executable>true</executable>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

        </plugins>
        <!--使用maven渲染yml和property-->
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
```

- `mainClass`节点，需要指定程序的启动文件，否则打完的jar包会找不到启动文件
- `resource`节点中，注意资源文件的目录注意不要写错，否则打包之后，端口号、根路径等配置不会生效
- `finalName`是打包后，jar的名字



## 3. 使用Maven命令打包并运行

- 执行`mvn clean package`打包

- 成功后，使用`java -jar property.jar`命令运行jar

- 在浏览器中输入`localhost:8088/demo/property`验证是否成功运行

  网页显示json数据，表示运行成功

  ```json
  {"developer.name":"生产环境 cheng","developer.qq":123456,"developer.email":"生产环境 123456@email.com","application.version":"生产环境 1.0-SNAPSHOT","developer.phone":"生产环境 19900000129","application.name":"生产环境 properties"}
  ```

  此时可以看到使用@value@读取pom.xml中的属性，已经正确显示



## 3.打包遇到的问题

### 1. 打包成功后运行报`jar中没有主清单属性`

打包时`MANIFEST.MF`中没有Main-Class属性，解决办法:

项目右键->open module setting -> artifacts -> ➕ -> JAR -> from modules -> Main Class 选择main方法所在的类文件；选中 copy  to output directory and link via manifest, 选择`META-INF`的存放路径为src目录(PS:不要放在`main/java`路径下 )；



### 2. 打包成功后配置的端口号`8088`和根路径`/demo`不生效

原因是`pom.xml`中`build`节点下的`resources`目录错误，导致未读取到配置文件



### 3. jar运行后调用接口报错

错误提示：`org.springframework.boot.configurationprocessor.json.jsonobject class not found `

原因是configurationprocessor依赖未打包进入jar，修改代码使用`org.json`包

添加依赖：

```xml
      <dependency>
            <groupId>org.json</groupId>
            <artifactId>json</artifactId>
            <version>20210307</version>
        </dependency>
```



