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
Spring允许定义一个工厂，由工厂创建真正的bean.Spring 。当一个Bean实现了FactoryBean接口后，Spring会先实例化这个工厂，然后调用`getObject`创建真正的Bean,`getObjectType`可以制定创建的Bean类型，***因为指定类型不一定与实际类型一致，可以是接口或抽象类***。   
如果定义了FactoryBean,为了和普通的Bean区分开，通常以`xxxFactoryBean`命名。
    ```java
    @Component
    public class ZoneIdFactoryBean implements FactoryBean<ZoneId>{
        String zone = "z";
        @Override
        public ZoneId getObject() throws Exception{
            return ZoneId.of(zone);
        }
        @Override
        public Class<?> getObjectType(){
            return  ZoneId.class;
        }
    }
    ```
##### 使用Resource
将Resource下的文件等资源文件以Bean的形式自动注入,之后`resource.getInputStream`就可以获取到输入流
```java
@Component
public class AppService {
    @Value("classpath:/logo.txt")
    private Resource resource;

    private String logo;

    @PostConstruct
    public void init() throws IOException {
        try (var reader = new BufferedReader(
                new InputStreamReader(resource.getInputStream(), StandardCharsets.UTF_8))) {
            this.logo = reader.lines().collect(Collectors.joining("\n"));
        }
    }
}
```
##### 注入Property配置信息
* 直接从property文件中读取
  *   ```java
        @Configuration
        @ComponentScan
        @PropertySource("app.properties")
        public class AppConfig{
            @Value("${app.zone}")//取app.zone 为key，不存在则报错
            String zoneId;

            @Value("${app.host:8080}")//取key为app.host 不存在则默认值8080
            String host;
        }
        //   传参
                @Bean
                ZoneId createZoneId(@Value("${app.zone:Z}") String zoneId){

                }
       ```
* 借助一个中间Bean  
多个Bean可以使用同一个中间Bean的配置，统一修改
  ```java
        @Component
        public class SmtpConfig {
            @Value("${smtp.host}")
            private String host;

            @Value("${smtp.port:25}")
            private int port;

            public String getHost() {
                return host;
            }

            public int getPort() {
                return port;
            }
        }
        @Component
        public class MailService{
            @Value("#{smtpConfig.host}") //#{} 使用方式
            private String smtpHost;
        }
  ```
##### 使用条件装配
根据不同生产环境装配不同的Bean
* 使用Profile
-Dspring.profiles.active=test,master
```java
// -Dspring.profiles.active=test
@Bean
@Profile("test") //test 环境下装配bean
ZoneId createZoneId(){
    return ZoneId.systemDefault();
}
@Bean
@Profile("!test")//非test环境下装配bean
ZoneId createZoneIdForTest() {
        return ZoneId.of("America/New_York");
    }
//满足多个Profile条件 
// -Dspring.profiles.active=test,master

    @Bean
    @Profile({"test","master"})
    ZoneId createZoneId(){

    }
```
* 使用Conditional
```java
// 当满足条件，OnSmtpEnvConditional时，创建Bean
@Component
@Conditional(OnSmtpEnvConditional.class)
public class SmtpMailService implements MailService{
    // ......
}
// 当存在环境变量smtp时，成功
public class OnSmtpEnvConditional implements Condition{
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata){
        return "true".equalsIgnoreCase(System.getenv("smtp"));
    }
}
```
#### AOP
类似一种代理模式的实现方式,类似新建子类继承自原来的类，之后在子类的方法调用时，可以调用自己的方法后 再调用父类的方法      
 **Spring对接口类型使用JDK动态代理，对普通类使用CGLIB创建子类。如果一个Bean的class是final，Spring将无法为其创建子类**。
1. 定义执行方法，并在方法上通过AspectJ的注解告诉Spring应该在何处调用此方法；
2. 标记@Component和@Aspect；
3. 在@Configuration类上标注@EnableAspectJAutoProxy
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>${spring.version}</version>
</dependency>
```
```java
@Aspect
@Component
public class LoggingAspect {
    // 在执行UserService的每个方法前执行:
    @Before("execution(public * com.itranswarp.learnjava.service.UserService.*(..))")
    public void doAccessCheck() {
        System.err.println("[Before] do access check...");
    }

    // 在执行MailService的每个方法前后执行:
    @Around("execution(public * com.itranswarp.learnjava.service.MailService.*(..))")
    public Object doLogging(ProceedingJoinPoint pjp) throws Throwable {
        System.err.println("[Around] start " + pjp.getSignature());
        Object retVal = pjp.proceed();
        System.err.println("[Around] done " + pjp.getSignature());
        return retVal;
    }
}
//在config类 通过注解 EnableAspectAutoProxy使能 aop
@Configuration
@ComponentScan
@EnableAspectJAutoProxy
public class AppConfig {
    ...
}
// <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
// <aop:aspectj-autoproxy proxy-target-class="true"></aop:aspectj-autoproxy>
// proxy-target-class属性值决定是基于接口的还是基于类的代理被创建。如果proxy-target-class 属性值被设置为true，那么基于类的代理将起作用（这时需要cglib库）。如果proxy-target-class属值被设置为false或者这个属性被省略，那么标准的JDK 基于接口的代理将起作用。
```

    @Before：这种拦截器先执行拦截代码，再执行目标代码。如果拦截器抛异常，那么目标代码就不执行了；

    @After：这种拦截器先执行目标代码，再执行拦截器代码。无论目标代码是否抛异常，拦截器代码都会执行；

    @AfterReturning：和@After不同的是，只有当目标代码正常返回时，才执行拦截器代码；

    @AfterThrowing：和@After不同的是，只有当目标代码抛出了异常时，才执行拦截器代码；

    @Around：能完全控制目标代码是否执行，并可以在执行前后、抛异常后执行任意拦截代码，可以说是包含了上面所有功能

* execution 语法   
`execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern)
                throws-pattern?)`
ret-type-pattern 、name-pattern 、parameter-pattern 必选
   * `*`可表示为匹配所有的返回类型   
   * com.xyz.service.AccountService.* 匹配AccountService类的方法
   * ()表示方法参数为空
   * (..)匹配一个或多个参数
   * (*)参数可以为任意类型
   * (*,String)两个参数，第一个参数任意类型，第二个为String
   * example
     *    execution(public * *(..)) 匹配任意public方法
     *    execution(* set*(..)) 匹配set开头的任意方法
     *   execution(* com.xyz.service.*.*(..)) 匹配service包下所有方法
     *       execution(* com.xyz.service..*.*(..)) 匹配service包下或其子包下所有方法
##### 使用注解装配AOP
通过自定义注解方法，声明方法，同时`@Around("@annotation(metricTime)"))`,metricTime 为注解的小写开头的名字，实现AOP装配
```java
@Target(METHOD)
@Retention(RUNTIME)
public @interface MetricTime(){
    String value();   
}
//  在切点上 标明注解
@Component
public class UserService {
    // 监控register()方法性能:
    @MetricTime("register")
    public User register(String email, String password, String name) {
        ...
    }
    ...
}

@Aspect
@Component
public class MetricAspect {
    @Around("@annotation(metricTime)")
    public Object metric(ProceedingJoinPoint joinPoint, MetricTime metricTime) throws Throwable {
        String name = metricTime.value();
        long start = System.currentTimeMillis();
        try {
            return joinPoint.proceed();
        } finally {
            long t = System.currentTimeMillis() - start;
            // 写入日志或发送至JMX:
            System.err.println("[Metrics] " + name + ": " + t + "ms");
        }
    }
}
```
一次AOP异常
* 访问被注入的Bean时，总是调用方法而不是直接访问字段
* 编写Beans时，如果有可能被代理，不写public final 方法

```java
@Component
public class UserService {
    // 成员变量:
    public final ZoneId zoneId = ZoneId.systemDefault();

    // 构造方法:
    public UserService() {
        System.out.println("UserService(): init...");
        System.out.println("UserService(): zoneId = " + this.zoneId);
    }

    // public方法:
    public ZoneId getZoneId() {
        return zoneId;
    }

    // public final方法:
    public final ZoneId getFinalZoneId() {
        return zoneId;
    }
}

```
```java
@Component
public class MailService {
    @Autowired
    UserService userService;

    public String sendMail() {
        //  NPE问题
        ZoneId zoneId = userService.zoneId;
        String dt = ZonedDateTime.now(zoneId).toString();
        return "Hello, it is " + dt;
    }
}
```
```java
@Aspect
@Component
public class LoggingAspect {
    @Before("execution(public * com..*.UserService.*(..))")
    public void doAccessCheck() {
        System.err.println("[Before] do access check...");
    }
}
```
```java
@Configuration
@ComponentScan
@EnableAspectAutoProxy
public class AppConfig {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        MailService mailService = context.getBean(MailService.class);
        System.out.println(mailService.sendMail());
    }
}
```
* 通过CGLIB创建的UserService的子类，该代理类会覆写所有public和protected方法，并在内部将调用委托给原始的UserService实例
* Spring通过CGLIB创建的代理类，构造函数中，没有`super()` 不会初始化代理类自身继承的任何成员变量，包括final类型的成员变量！
* 自动加super()的功能是Java编译器实现的 ,Spring使用CGLIB构造的Proxy类，是直接生成字节码，并没有源码-编译-字节码这个步骤
* 访问被代理的Bean的字段 通过接口访问。！！！