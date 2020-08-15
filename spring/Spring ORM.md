# Spring ORM
ORM(object-relational mapping) 对象/关系映射  
延迟加载，允许抓取对象的一部分数据  
预先抓取，使用一个查询获取完整的关联对象  
级联，删除对象时，也把关联的对象删除 
## Hibernate
### 创建Hibernate  Session Factory  
主要接口org.hibernate.Session以及Hibernate Session Factory。        
* org.springframework.orm.hibernate3.LocalSessionFactoryBean   使用 XML 定义映射  
* org.springframework.orm.hibernate3.annotation.AnnotationSessionFactoryBean  使用注解的方式来定义持久化  
* org.springframework.orm.hibernate4.LocalSessionFactoryBean  XML+注解
    * dataSource :属性声明从哪里获得数据库连接    
    * setPackagesToScan : Spring扫描一个或多个包以查找域类，这些类通过注解的形式表明使用
Hibernate进行持久化，可用注解JPA的@Entity@MappedSupperclass Hibernate的@entity       
    * setMappingResource  Hibernate映射/配置文件,定义了应用程序的优化策略
```java
@Bean
public LocalSessionFactoryBean sessionFactory(DataSource dataSource) {
  LocalSessionFactoryBean sfb = new LocalSessionFactoryBean();
  sfb.setDataSource(dataSource);
  lsfb.setPackagesToScan("spittr.domain");
  //sfb.setMappingResource(new String[] { "com.habuma.spittr.domain" });
  Properties props = new Properties();
  props.setProperty("dialect", "org.hibernate.dialect.H2Dialect");
  sfb.setHebernateProperties(props);
  return sfb;
}
```
### 创建HibernateRepository
编写Repository，构建不依赖于Spring的Hibernate代码
 * 1.HibernateTemplate 保证每个事务使用同一个Session,但是Repository实现会与Spring耦合   
 * 2.使用Session，将Hibernate SessionFactory 装配到Repository   
@Repository注解  
* 1 组件扫描
 * 	2 通过persistenceTranslation 进行异常处理，像Hibernate template一样不用写异常代码,为了给不使用模板的 Hibernate Repository 添加异常转换功能，我们只需在 Spring 应用上下文中添加一个 PersistenceExceptionTranslationPostProcessor bean   
     * ```java
         @Bean
        public BeanPostProcessor persistenceTranslation() {
        return new PersistenceExceptionTranslationPostProcessor();
        }
        ```
     * PersistenceExceptionTranslationPostProcesso r是一个 bean 后置处理器（bean post-processor），它会在所有拥有 @Repository 注解的类上添加一个通知器（advisor），这样就会捕获任何平台相关的异常并以 Spring 非检查型数据访问异常的形式重新抛出。
```java
@Repository
public class HibernateSpitterRepository implements SpitterRepository {
	private SessionFactory sessionFactory;
	/**
	 * Inject 自动注入
	 * @param sessionFactory
	 */
	@Inject
	public HibernateSpitterRepository(SessionFactory sessionFactory) {
		this.sessionFactory = sessionFactory;		 //<co id="co_InjectSessionFactory"/>
	}
	/**
	 * 通过sessionFactory获取当前事务的session
	 * @return
	 */
	private Session currentSession() {
		return sessionFactory.getCurrentSession();//<co id="co_RetrieveCurrentSession"/>
	}

	public long count() {
		return findAll().size();
	}

	public Spitter save(Spitter spitter) {
		Serializable id = currentSession().save(spitter);  //<co id="co_UseCurrentSession"/>
		return new Spitter((Long) id, 
				spitter.getUsername(), 
				spitter.getPassword(), 
				spitter.getFullName(), 
				spitter.getEmail(), 
				spitter.isUpdateByEmail());
	}
```
## Spring JPA
JPA(JAVA Persistence API),通过EntityManager进行管理   
两种类型的实体管理器
* application-managed-------LocalEntityManagerFactoryBean
* container-managed-------LocalContainerEntityManagerFactoryBean
### 应用程序管理类型的JPA
绝大部分配置信息来源于一个名为 persistence.xml 的配置文件。这个文件必须位于类路径下的 META-INF 目录下  
* xml 的作用在于定义一个或多个持久化单元。持久化单元是同一个数据源下的一个或多个持久化类。简单来讲，persistence.xml 列出了一个或多个的持久化类以及一些其他的配置如数据源和基于 XML 的配置文件
* ```XML
   <persistence version="2.1" xmlns="http://xmlns.jcp.org/xml/ns/persistence">
  <persistence-unit name="spitterPU">
    <class>com.habuma.spittr.domain.Spitter</class>
    <class>com.habuma.spittr.domain.Spittle</class>
    <properties>
      <property name="toplink.jdbc.driver" value="org.hsqldb.jdbcDriver" />
      <property name="toplink.jdbc.url" value="jdbc:hsqldb:hsql://localhost/spitter/spitter" />
      <property name="toplink.jdbc.user" value="sa" />
      <property name="toplink.jdbc.password" value="" />
    </properties>
  </persistence-unit>
  </persistence>
   ```      
    ```java
    @Bean
    public LocalEntityManagerFactoryBean entityManagerFactoryBean() {
    LocalEntityManagerFactoryBean emfb = new LocalEntityManagerFactoryBean();
    emfb.setPersistenceUnitName("spitterPU");
    return emfb;
    }
    ```
### 容器管理类型的JPA
配置 LocalContainerEntityManagerFactoryBean     
* a 设定 dataSource数据源
* b 设定 JpaVendorAdapter
* c 设定 @Entity 类
```java
  @Bean
  public LocalContainerEntityManagerFactoryBean emf(DataSource dataSource, JpaVendorAdapter jpaVendorAdapter) {
    LocalContainerEntityManagerFactoryBean emf = new LocalContainerEntityManagerFactoryBean();
    emf.setDataSource(dataSource);
    emf.setPersistenceUnitName("spittr");
    emf.setJpaVendorAdapter(jpaVendorAdapter);
//    查找带有@Entity的类
//    Spring扫描一个或多个包以查找域类，这些类通过注解的形式表明使用Hibernate进行持久化，可用注解JPA的@Entity @MappedSupperclass Hibernate的@entity
    emf.setPackagesToScan("spittr.domain");
    return emf;
  }
```           
 jpaVendorAdapter 指明所使用的是哪一个厂商的JPA实现
```java
  @Bean
  public JpaVendorAdapter jpaVendorAdapter() {
    HibernateJpaVendorAdapter adapter = new HibernateJpaVendorAdapter();
    adapter.setDatabase(Database.H2);
    adapter.setShowSql(true);
    adapter.setGenerateDdl(false);
    adapter.setDatabasePlatform("org.hibernate.dialect.H2Dialect");
    return adapter;
  }
```
Entity 指定所要存储的Entity以及ID
```java
@Entity
public class Spitter {
	
	private Spitter() {}

	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	private Long id;

	@Column(name="username")
	private String username;

	@Column(name="password")
	private String password;

	@Column(name="fullname")
	private String fullName;

	@Column(name="email")
	private String email;

	@Column(name="updateByEmail")
	private boolean updateByEmail;

	public Spitter(Long id, String username, String password, String fullName, String email, boolean updateByEmail) {
		this.id = id;
		this.username = username;
		this.password = password;
		this.fullName = fullName;
		this.email = email;
		this.updateByEmail = updateByEmail;
	}
```
### 不使用模板的 纯粹基于JPA的repository
*  @Transactional 表明这个Repository中的持久化方法是在事务上下文中执行的
*  @Repository      
   *  1 线性扫描   
	 * 2 PersistenceExceptionTranslation-PostProcessor 就会知道要将这个 bean 产生的异常转换成 Spring 的统一数据访问异常    

**EntityManager不是线程安全，一般不适合注入Repository 这类共享的单例bean中，so，加注解@PersistenceContext**         
@PersistenceContext 注解 Spring 将EntityManagerFactory "注入"到Repository
* 它没有将真正的 EntityManager 设置给 Repository，而是给了它一个 EntityManager 的代理。真正的 EntityManager 是与当前事务相关联的那一个
* 始终以线程安全的方式使用EntityManager    
 @PersistenceUnit @PersistenceContext 不是Spring 注解。由JPA规范提供，为了让Spring理解 并注入EntityFactory 或者EntityManager　需要配置Spring的Persistence-AnnotationBeanPostProcessor
* <context:annotation-config> 或者<context:component-scan></> 自动注册PersistenceAnnotationBeanPostProcessor bean
*  显式注册 　　
    ```java
        　@Bean
            public PersistenceAnnotationBeanPostProcessor paPostProcessor(){
                return new PersistenceAnnotationBeanPostProcessor();
            }
    ```
```java
@Repository
public class JpaSpitterRepository implements SpitterRepository {
	/**
	 *
	 */
	@PersistenceContext
	private EntityManager entityManager;

	public long count() {
		return findAll().size();
	}

	public Spitter save(Spitter spitter) {
		entityManager.persist(spitter);
		return spitter;
	}

```
### Spring Data实现自动化JPARepository
@EnableJpaRepositories(basePackages="spittr.db")
*  EnableJpaRepositories 类似component-scan 自动扫描base-package    
*  查找扩展自Spring data JPA Repository接口的所有类，（在应用启动时-Spring应用上下文创建的时候）自动生成这个接口的实现
#### 实现自定义混合  
* 配置LocalContainerEntityManagerFactoryBean
  * dataSource 数据源
  * setPersistenceUnitName 设置持久化数据对象
  * 设定 JpaVendorAdapter 指明哪家厂商的JPA实现
  * setPackagesToScan 设定Entity
```JAVA
@Bean
  public DataSource dataSource() {
    EmbeddedDatabaseBuilder edb = new EmbeddedDatabaseBuilder();
    edb.setType(EmbeddedDatabaseType.H2);
    edb.addScript("spittr/db/jpa/schema.sql");
    edb.addScript("spittr/db/jpa/test-data.sql");
    EmbeddedDatabase embeddedDatabase = edb.build();
    return embeddedDatabase;
  }
@Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource, JpaVendorAdapter jpaVendorAdapter) {
    LocalContainerEntityManagerFactoryBean emf = new LocalContainerEntityManagerFactoryBean();
    emf.setDataSource(dataSource);
    emf.setPersistenceUnitName("spittr");
    emf.setJpaVendorAdapter(jpaVendorAdapter);
    emf.setPackagesToScan("spittr.domain");
    return emf;
  }
  
  @Bean
  public JpaVendorAdapter jpaVendorAdapter() {
    HibernateJpaVendorAdapter adapter = new HibernateJpaVendorAdapter();
    adapter.setDatabase(Database.H2);
    adapter.setShowSql(true);
    adapter.setGenerateDdl(false);
    adapter.setDatabasePlatform("org.hibernate.dialect.H2Dialect");
    return adapter;
  }
```
* 声明简单的查询方法   
**给定方法签名让Spring DATA JPA 自动生成实现**,当创建Repository时，Spring Data检查Repository接口的所有方法，解析方法名称，基于被持久化的对象推测方法目的
    * 继承继承JpaRepository，设定持久化对象，ID     
    * 有Entity Manager传统实现Repository方法时，继承impl类的父类     

    **方法签名-readSpitterByFirstnameOrLastnameOrderByLastname**    
    方法签名-查询动词+主题（）+断言（条件+属性+比较操作（默认相等比较）） 
  * 要查询的对象类型是通过如何参数化 JpaRepository 接口来确定的，而不是方法名称中的主题
  * 动词有 get find read 和count,其中get find read同义。对应方法都会查询数据后返回对象  count返回匹配对象的数量  
    ```java
    List<Spitter> readByFirstnameOrLastname(String first, String last);
    * 		List<Spitter> readByFirstnameIgnoringCaseOrLastnameIgnoringCase(String first, String last);
    * 	 	List<Spitter> readByFirstnameOrLastnameAllIgnoresCase(String first, String last);
    * 	 	List<Spitter> readByFirstnameOrLastnameOrderByLastnameAsc(String first, String last);
    * 	 	List<Spitter> readByFirstnameOrLastnameOrderByLastnameAscFirstnameDesc(String first, String last);
    ```
* 声明自定义查询
  ```java
  @Query("select s from Spitter s where s.email like '%gmail.com'")
    List<Spitter> findAllGmailSpitters();
  ```

* 传统entityManager 传统方法    
  **当Spring Data JPA为Repository生成接口实现时，会查找名字与接口相同，添加Impl后缀的类，将Impl类的方法同JPA生成的方法结合**
    ```
        public interface SpitterSweeper {
            int eliteSweep();
        }

        /**
        * 			通过EntityManager自定义查询
        */
        public class SpitterRepositoryImpl implements SpitterSweeper {

            @PersistenceContext
            private EntityManager em;
            
            public int eliteSweep() {
            String update = 
                "UPDATE Spitter spitter " +
                    "SET spitter.status = 'Elite' " +
                    "WHERE spitter.status = 'Newbie' " +
                    "AND spitter.id IN (" +
                    "SELECT s FROM Spitter s WHERE (" +
                    "  SELECT COUNT(spittles) FROM s.spittles spittles) > 10000" +
                    ")";
                return em.createQuery(update).executeUpdate();
            }
            }
    ```
* 混合
    ```java
    public interface SpitterRepository extends JpaRepository<Spitter, Long>, SpitterSweeper {
        
        Spitter findByUsername(String username);
        
        List<Spitter> findByUsernameOrFullNameLike(String username, String fullName);

        @Query("select s from Spitter s where s.email like '%gmail.com'")
        List<Spitter> findAllGamilSpitters();
    }
    ```