#### Spring
##### IOC容器
容器-为某种特定组件的运行提供必要支持的一个软件环境。Tomcat，Servlet容器。
IOC容器可以管理轻量的JavaBean组件，提供包括组件的生命周期管理、配置和组装服务、AOP支持，以及建立在AOP基础上的声明式事务服务等底层服务。  
IOC---Inversion of Control 控制反转，也称依赖注入(Dependency Injection)，将组件的创建+配置与组件的使用分离，由IOC容器负责管理组件的声明周期。
* 谁创建组件
* 谁负责根据依赖关系组装组件
* 销毁时，如何按照依赖顺序依次销毁

```java
public class BookService {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```
创建三个bean，并将`dataSource`这个bean通过属性dataSource（即调用setDataSource()方法）注入到到`bookService`和`userService`两个bean中,根据Id来区分bean。
```xml
<beans>
    <bean id="dataSource" class="HikariDataSource" />
    <bean id="bookService" class="BookService">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="userService" class="UserService">
        <property name="dataSource" ref="dataSource" />
    </bean>
</beans>
```
也可以通过构造函数注入
```java
public class BookService {
    private DataSource dataSource;

    public BookService(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```
Spring的IoC容器是一个高度可扩展的无侵入容器。所谓无侵入，是指应用程序的组件无需实现Spring的特定接口，或者说，组件根本不知道自己在Spring的容器中运行
* 应用程序组件既可以在Spring的IoC容器中运行，也可以自己编写代码自行组装配置
* 测试的时候并不依赖Spring容器，可单独进行测试，大大提高了开发效率
##### 装配Bean
```xml
<!-- 装配Bean -->
  <bean id="userService" class="com.itranswarp.learnjava.service.UserService">
        <property name="mailService" ref="mailService" />
    </bean>
<!-- 装配属性 -->
    <bean id="dataSource" class="com.zaxxer.hikari.HikariDataSource">
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test" />
    <property name="username" value="root" />
    <property name="password" value="password" />
    <property name="maximumPoolSize" value="10" />
    <property name="autoCommit" value="true" />
</bean>
```
Bean的使用，Spring容器就是ApplicationContext，这个接口有很多实现类，ClassPathXmlApplicationContext可以根据XML文件构建bean，之后通过ApplicationContext实例根据Bean的ID获取Bean     
```java
  ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        UserService userService = context.getBean(UserService.class);
        User user = userService.login("bob@example.com", "password");
        System.out.println(user.getName());
```
Spring还提供BeanFactory容器  
ApplicationContext会一次性创建所有的Bean，而BeanFactory会按需创建。  
```java
BeanFactory factory = new XmlBeanFactory(new ClassPathResource("application.xml"));
```
##### 使用Annotation配置
`@Component`标识这是一个Bean，默认Bean id 为类的小写开头的类名，一般把@Autowired写在字段上，通常使用package权限的字段，便于测试    
字段上
```java
@Component
public class UserService {
    @Autowired
    MailService mailService;

    ...
}
```
构造函数上
```java
@Component
public class UserService {
    MailService mailService;

    public UserService(@Autowired MailService mailService) {
        this.mailService = mailService;
    }
    ...
}
```
使用，`@Configuration`标识配置类，`ComponentScan`自动扫描当前包及其子包，把所有标注为`@Component`的Bean自动创建，并根据`@Autowired`自动装配。   
Spring容器ApplicationContext的实现类，AnnotationConfigApplicationContext
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
##### 定制Bean
* Scope
  * 正常Bean为单例模式，Scope每次都会返回一个新的实例，成为Prototype原型
      ```java
        @Component
        @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE) // @Scope("prototype")
        public class MailSession {
            ...
        }
      ```
* 注入List
  * 注入接口的不同实现类的`List<Bean>`,会自动把所有类型为Validator的Bean装配为一个List注入进来。
     ```java
     @Component
    public class EmailValidator implements Validator {
    }
    @Component
    public class PasswordValidator implements Validator {
    }
    @Component
    public class NameValidator implements Validator {
    }

            @Component
        public class Validators {
            @Autowired
            List<Validator> validators;

            public void validate(String email, String password, String name) {
                for (var validator : this.validators) {
                    validator.validate(email, password, name);
                }
            }
        }
     ```
     可以通过`@Order`指定注入顺序，否则不确定
     ```java
            @Component
        @Order(1)
        public class EmailValidator implements Validator {
            ...
        }

        @Component
        @Order(2)
        public class PasswordValidator implements Validator {
            ...
        }
     ```
* 可选注入
  * 当标记了`Autowired`但是未找到Bean时，抛出`NoSuchBeanDefinitionException`异常，适合有定义就使用定义，没有就使用默认值的情况
     ```java
     @Component
    public class MailService {
        @Autowired(required = false)
        ZoneId zoneId = ZoneId.systemDefault();
    }
     ```
* 创建第三方Bean
* 一个Bean不在我们自己的package管理之内,Spring对`@Bean`的方法只调用一次，返回的仍然是单例
    ```java
    @Bean
    ZoneId createZoneId(){
        return ZoneId.of("Z");
    }
    ```
* 初始化和销毁
  * Bean在注入依赖完毕后，需要进行初始化
    ```xml
        <dependency>
        <groupId>javax.annotation</groupId>
        <artifactId>javax.annotation-api</artifactId>
        <version>1.3.2</version>
    </dependency>
    ```
    在Bean的初始化和清理方法上标记`@PostConstruct`和`@PreDestroy`,根据注解查方法，和方法名无关。
    ```java
            @Component
        public class MailService {
            @Autowired(required = false)
            ZoneId zoneId = ZoneId.systemDefault();

            @PostConstruct
            public void init() {
                System.out.println("Init mail service with zoneId = " + this.zoneId);
            }

            @PreDestroy
            public void shutdown() {
                System.out.println("Shutdown mail service");
            }
        }
    ```
* 使用别名
  * 同一种Bean的多个实例
   ```java
        @Configuration
        @ComponentScan
        public class AppConfig {
            @Bean("z")
            ZoneId createZoneOfZ() {
                return ZoneId.of("Z");
            }

            @Bean
            @Qualifier("utc8")
            ZoneId createZoneOfUTC8() {
                return ZoneId.of("UTC+08:00");
            }
        }
   ```
   注入某个特定名字的Bean
   ```java
    @Component
    public class MailService {
        @Autowired(required = false)
        @Qualifier("z") // 指定注入名称为"z"的ZoneId
        ZoneId zoneId = ZoneId.systemDefault();
        ...
    }
   ```
   指定默认注入的Bean `@Primary`
   ```java
        @Configuration
        @ComponentScan
        public class AppConfig {
            @Bean
            @Primary
            DataSource createMasterDataSource() {
                ...
            }

            @Bean
            @Qualifier("slave")
            DataSource createSlaveDataSource() {
                ...
            }
        }
   ```
* 使用FactoryBean

针对设备图标部署繁琐的部署流程，设计并实现了一个设备图标自动部署后台管理工具
针对放大器的统计需求，结合现有统计方案，设计并实现了放大器日活、激活统计80多项作业，并协同米家参与了放大器数据差异性调查，负责对接米家相关数据
针对多个集群间设备重复激活问题，通过对现有统计逻辑的修补，实现去重后的各个集群间的数据统计，提高统计数据的精确性10-20%


1.编写代码要有一定的规范，码出高效，码出质量
2.做工具的制造者，不做重复低效工作
3.技术上保持学习
4.多加强沟通，明确问题，理清思路再行动



{"zh_TW":{"data_url":"https://cdn.cnbj1.fds.api.mi-img.com/miwifi/34d861c3-0cae-4977-8ab9-ec6e48c5a1c8.txt?GalaxyAccessKeyId=AKN4YH6B3OTPPRKHHA&Expires=1913020109925&Signature=Bc/dWcggfmEvmkHoO7P3bNlxDK0=","data_md5":"18cba028a6ee4587fe4515d6af4c8eb8","ts":1596796110144},"zh_HK":{"data_url":"https://cdn.cnbj1.fds.api.mi-img.com/miwifi/4447e5bc-a2f1-401b-beab-fc634e61c150.txt?GalaxyAccessKeyId=AKN4YH6B3OTPPRKHHA&Expires=1913020123998&Signature=mpKhReS0wkRXxISuZ9NUJs4cvZc=","data_md5":"87866afde7b77669324fe9c035054e6c","ts":1596796124000},"ko":{"data_url":"https://cdn.cnbj1.fds.api.mi-img.com/miwifi/c4dfe7e8-f7f3-480f-9727-7eede1bb59f5.txt?GalaxyAccessKeyId=AKN4YH6B3OTPPRKHHA&Expires=1913020124130&Signature=boUjVw5LmXbvv6HbUUHF5By/i5I=","data_md5":"5a4256fd31d6227982e6d74aeaf1e444","ts":1596796124132},"en":{"data_url":"https://cdn.cnbj1.fds.api.mi-img.com/miwifi/5a973c62-2a74-4297-b1ab-cb212d320f8d.txt?GalaxyAccessKeyId=AKN4YH6B3OTPPRKHHA&Expires=1913020124326&Signature=tPIewjiGqL/8k337di0lcQcn/9s=","data_md5":"3a68a846c08e39b0a54b1a5863042c7a","ts":1596796124327},"zh_CN":{"data_url":"https://cdn.cnbj1.fds.api.mi-img.com/miwifi/fd289c07-0a1a-4518-90a1-c9538403df46.txt?GalaxyAccessKeyId=AKN4YH6B3OTPPRKHHA&Expires=1913020124525&Signature=l01moTUQw5WElmpkEAuoEL24J6A=","data_md5":"a7b02ec77d3d3434001781eceeff669e","ts":1596796124526}}