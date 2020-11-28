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