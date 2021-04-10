#  Spring JDBC
**spring 提供平台无关的持久化异常，异常都继承自 DataAccessException。DataAccessException 的特殊之处在于它是一个非检查型异常** [--TODO]  
spring 认为触发异常的很多问题不能在catch中修复，不强制编写catch块代码   
![](https://i.loli.net/2020/08/10/EMGZNgk7Aw4mBQr.jpg)  
**Spring 的模板类处理数据访问的固定部分 —— 事务控制、管理资源以及处理异常。同时，应用程序相关的数据访问 —— 语句、绑定参数以及整理结果集 —— 在回调的实现中处理**。
## 数据源配置
* JNDI
   ```html
   <jee:jndi-lookup id="dataSource" jndi-name="/jdbc/SpitterDS" resource-ref="true" />
   ```
  ```java
  @Bean
  public JndiObjectFactoryBean dataSource() {
  JndiObjectFactoryBean jndiObjectFB = new JndiObjectFactoryBean();
  jndiObjectFB.setJndiName("jdbc/SpittrDS");
  jndiObjectFB.setResourceRef(true);
  jndiObjectFB.setProxyInterface(javax.sql.DataSource.class);
  return jndiObjectFB;
    }
  ```
* 连接池
   ```java
   @Bean
    public BasicDataSource dataSource() {
    BasicDataSource ds = new BasicDataSource();
    ds.setDriverClassName("org.h2.Driver");
    ds.setUrl("jdbc:h2:tcp://localhost/~/spitter");
    ds.setUsername("sa");
    ds.setPassword("");
    ds.setInitialSize(5);
    ds.setMaxActive(10);
    return ds;
    }中，因为要检查文件的MD5，同时也要检查输入流
   ```
* JDBC  
   DriverManagerDataSource 每个连接请求时，都会返回一个新建的连接   
   SingleConnectionDataSource 可是为一个连接的连接器，在每个连接请求返回同一个连接  
   SimpleDriverSource 直接使用JDBC驱动，解决特定环境下的类加载问题，环境包括OSGi容器    
   ```java
   @Bean
    public DataSource dataSource() {
    DriverManagerDataSource ds = new DriverManagerDataSource();
    ds.setDriverClassName("org.h2.Driver");
    ds.setUrl("jdbc:h2:tcp://localhost/~/spitter");
    ds.setUsername("sa");
    ds.setPassword("");
    return ds;
    }
   ```
* 嵌入式数据源
   ```java
   @Bean
    public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.H2)
        .setScript("classpath:schema.sql")
        .setScript("classpath:text-data.sql")
        .build();
    }
   ```
* profile方式配置多环境下数据源    
    ```java
    @Configuration
    public class DataSourceConfiguration {
    @Profile("development")
    @Bean
    public DataSource embeddedDataSource() {
    return new EmbeddedDatabaseBuilder()
    .setType(EmbeddedDatabaseType.H2)
    .addScript("classpath:schema.sql")
    .addScript("classpath:test-data.sql")
    .build();
    }
        
    @Profile("qa")
    @Bean
    public DataSource Data() {
        BasicDataSource ds = new BasicDataSource();
        ds.setDriverClassName("org.h2.Driver");
        ds.setUrl("jdbc:h2:tcp://localhost/-/spitter")
        ds.setUsername("sa"); 
        ds.setPassword(""); 
        ds.setlnitialSize(5);
        ds.setMaxActive(10);
        return ds;
    }
    
    @Profile ("production")
    @Bean
    public DataSource dataSource() {
        JndiObjectFactoryBean jndiObjectFactoryBean = new JndiObjectFactoryBean();
        jndiObjectFactoryBean.setJndiName("jdbc/SpittrDS");
        jndiObjectFactoryBean.setResourceRef(true);
        jndiObjectFactoryBean.setProxylnterface(javax.sql.Datasource.class);
        return (Datasource) jndiObjectFactoryBean.getObject();
    }
    }
    ```
* JDBC template接口
   ```java
     @Bean
  public JdbcTemplate jdbcTemplate(DataSource dataSource) {
    return new JdbcTemplate(dataSource);
  }
   ```
   ```java          
    //jdbc 应用jdbc template 实现数据访问与业务逻辑隔离 <3></>

    public class JdbcSpittleRepository implements SpittleRepository {

        private static final String SELECT_SPITTLE = "select sp.id, s.id as spitterId, s.username, s.password, s.fullname, s.email, s.updateByEmail, sp.message, sp.postedTime from Spittle sp, Spitter s where sp.spitter = s.id";
        private static final String SELECT_SPITTLE_BY_ID = SELECT_SPITTLE + " and sp.id=?";
        private static final String SELECT_SPITTLES_BY_SPITTER_ID = SELECT_SPITTLE + " and s.id=? order by sp.postedTime desc";
        private static final String SELECT_RECENT_SPITTLES = SELECT_SPITTLE + " order by sp.postedTime desc limit ?";
        
        private JdbcTemplate jdbcTemplate;

        public JdbcSpittleRepository(JdbcTemplate jdbcTemplate) {
            this.jdbcTemplate = jdbcTemplate;
        }

        public long count() {
            return jdbcTemplate.queryForLong("select count(id) from Spittle");
        }
        public List<Spittle> findRecent() {
            return findRecent(10);
        }

        public List<Spittle> findRecent(int count) {
            return jdbcTemplate.query(SELECT_RECENT_SPITTLES, new SpittleRowMapper(), count);
        }
   ```


