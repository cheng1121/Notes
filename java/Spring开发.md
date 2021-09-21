# Spring开发

## IoC容器

IoC：即控制翻转，也叫做依赖注入

### IoC原理

通过java的构造方法或者属性的set方法给属性赋值，类内通过属性去调用其他功能。不需要自己在类内实例化该属性，也就是当前类内不需要维护其他功能对象，交由外部去维护，并注入到当前类内。



### xml方式装配Bean

#### 1. `pom.xml`中添加Spring依赖

```xml
<dependencies>
    ...
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
    ...
    </dependencies>
```

#### 2. 编写`application.xml`

application.xml用来告诉Spring如何创建并组装bean

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userService" class="com.itranswarp.learnjava.service.UserService">
        <property name="mailService" ref="mailService" />
    </bean>

    <bean id="mailService" class="com.itranswarp.learnjava.service.MailService" />
</beans>
```



其中名字为`userService`的`bean`节点即是我们需要依赖注入的类；`property`节点是通过`mailService`方法来注入

`mailService`的`bean`节点是被注入的类

其中ref表示注入的是类；如果需要注入基本数据类型则使用value

#### 3. 创建IoC容器实例

通过容器实例加载声明好的`application.xml`文件

- 使用`ApplicationContext`加载

  ```java
  ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
  // 获取Bean:
  UserService userService = context.getBean(UserService.class);
  // 正常调用:
  User user = userService.login("bob@example.com", "password");
  
  ```

  

- 使用`BeanFactory`加载

  ```java
  
  BeanFactory factory = new XmlBeanFactory(new ClassPathResource("application.xml"));
  MailService mailService = factory.getBean(MailService.class);
  ```

`BeanFactory`和`ApplicationContext`的区别在于，`BeanFactory`的实现是按需创建，即第一次获取Bean时才创建这个Bean，而`ApplicationContext`会一次性创建所有的Bean。实际上，`ApplicationContext`接口是从`BeanFactory`接口继承而来的，并且，`ApplicationContext`提供了一些额外的功能，包括国际化支持、事件和通知机制等。通常情况下，我们总是使用`ApplicationContext`，很少会考虑使用`BeanFactory`。





### 注解方式装配Bean

xml方式是所有的Bean都一目了然，但是配置比较复杂，每次都需要修改`application.xml`文件

注解方式更简单和灵活，Spring会自动扫描注解并装配它们

- 在Bean中clas上方添加`@Component`注解，此注解相当于定义了xml中的bean节点

- 在需要注入的属性、方法、构造函数上添加`@Autowired`注解。相当于xml中的`property`节点

- 编写配置类`appConfig`，并在此类上添加`@Configuration`和`@ComponentScan`注解

  ```java
  @Configuration
  @ComponentScan
  public class AppConfig {
      public static void main(String[] args) {
          ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
          UserService userService = context.getBean(UserService.class);
          User user = userService.login("bob@example.com", "password");
          System.out.println(user.getName());
      }
  }
  ```

  此时`ApplicationContext`实现类使用的是`AnnotationConfigApplicationContext`而不是`ClassPathXmlApplicationContext`

  `@Configuration`注解表示此类是一个配置类

  `@ComponentScan`注解告诉容器，自动搜索当前类所在的包及子包，把所有标注了`@Component`注解的Bean自动创建出来并根据`@Autowired`注解机型装配

