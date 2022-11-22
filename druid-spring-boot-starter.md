<a name="Ubv0O"></a>
# 1.引入依赖
```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.15</version>
</dependency>
```
<a name="F9Srt"></a>
# 2. application.yml 配置
```yaml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driverClassName: com.mysql.cj.jdbc.Driver
    druid:
        url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8
        username: root
        password: root
				# 数据库连接池初始值
        initialSize: 5
        # 最小连接池数量
        minIdle: 10
        # 最大连接池数量
        maxActive: 20
        # 配置获取连接等待超时的时间
        maxWait: 60000
        # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
        timeBetweenEvictionRunsMillis: 60000
        # 配置一个连接在池中最小生存的时间，单位是毫秒
        minEvictableIdleTimeMillis: 300000
        # 配置一个连接在池中最大生存的时间，单位是毫秒
        maxEvictableIdleTimeMillis: 900000
        # 配置检测连接是否有效
        validationQuery: SELECT 1 FROM DUAL
        testWhileIdle: true
        testOnBorrow: false
        testOnReturn: false
        webStatFilter:
          enabled: true
        statViewServlet:
          enabled: true
          # 设置白名单，不填则允许所有访问
          allow:
          url-pattern: /druid/*
          # 控制台管理用户名和密码
          login-username: test
          login-password: test
        filter:
          stat:
            enabled: true
            # 慢SQL记录
            log-slow-sql: true
            slow-sql-millis: 1000
            merge-sql: true
          wall:
            config:
              multi-statement-allow: true
```

- 开发环境可打印执行的sql，方便开发、排查问题，添加配置：
```properties
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```
<a name="Zy18J"></a>
## 监控有关配置：
```properties

# 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
spring.datasource.druid.filters=stat,wall
# 通过connectProperties属性来打开mergeSql功能；慢SQL记录
spring.datasource.druid.connection-properties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
#是否启用StatFilter默认值false，用于采集 web-jdbc 关联监控的数据。
spring.datasource.druid.web-stat-filter.enabled=true
#需要监控的 url
spring.datasource.druid.web-stat-filter.url-pattern=/*
#排除一些静态资源，以提高效率
spring.datasource.druid.web-stat-filter.exclusions=*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*
#是否启用StatViewServlet（监控页面）默认值为false（考虑到安全问题默认并未启动，如需启用建议设置密码或白名单以保障安全）
spring.datasource.druid.stat-view-servlet.enabled=true
#内置的监控页面地址，例如 /druid/*，则内置监控页面的首页是 /druid/index.html
spring.datasource.druid.stat-view-servlet.url-pattern=/druid/*
#是否允许清空统计数据
spring.datasource.druid.stat-view-servlet.reset-enable=false
spring.datasource.druid.stat-view-servlet.login-username=admin
spring.datasource.druid.stat-view-servlet.login-password=admin
```
<a name="Kapob"></a>
# druid-spring-boot-starter自动注入原理

- spring boot自动装配的原理：springboot是在SpringApplication.run(...)容器启动时执行了selectImports()方法，找到自动配置类的全限类名去加载对应的class，然后将自动配置类注入Spring容器中，即通过加载注入全限类名对应的自动配置类来完成容器启动时组件的自动装载。
- druid-spring-boot-starter的自动配置类：DruidDataSourceAutoConfigure
```java

@Configuration
@ConditionalOnClass({DruidDataSource.class})
@AutoConfigureBefore({DataSourceAutoConfiguration.class})
@EnableConfigurationProperties({DruidStatProperties.class, DataSourceProperties.class})
@Import({DruidSpringAopConfiguration.class, DruidStatViewServletConfiguration.class, DruidWebStatFilterConfiguration.class, DruidFilterConfiguration.class})
public class DruidDataSourceAutoConfigure {
    private static final Logger LOGGER = LoggerFactory.getLogger(DruidDataSourceAutoConfigure.class);
 
    public DruidDataSourceAutoConfigure() {
    }
 
    @Bean(
        initMethod = "init"
    )
    @ConditionalOnMissingBean
    public DataSource dataSource() {
        LOGGER.info("Init DruidDataSource");
        return new DruidDataSourceWrapper();
    }
}
```
<a name="jVjCC"></a>
## @Configuration

- 加载DruidDataSourceAutoConfigure注入spring容器。
<a name="qhD8y"></a>
## @ConditionalOnClass({DruidDataSource.class})

- 类路径上存在DruidDataSource的class，才会加载当前类。
<a name="dtu3k"></a>
## Druid的默认连接池类DruidAbstractDataSource，是所有连接池属性默认值声明的地方：
```java
 public DruidAbstractDataSource(boolean lockFair) {
        this.validationQuery = DEFAULT_VALIDATION_QUERY;
        this.validationQueryTimeout = -1;
        this.testOnBorrow = false;
        this.testOnReturn = false;
        this.testWhileIdle = true;
        this.poolPreparedStatements = false;
        this.sharePreparedStatements = false;
        this.maxPoolPreparedStatementPerConnectionSize = 10;
        this.inited = false;
        this.initExceptionThrow = true;
        this.logWriter = new PrintWriter(System.out);
        this.filters = new CopyOnWriteArrayList();
        this.clearFiltersEnable = true;
        this.exceptionSorter = null;
        this.maxWaitThreadCount = -1;
        this.accessToUnderlyingConnectionAllowed = true;
        this.timeBetweenEvictionRunsMillis = 60000L;
        this.numTestsPerEvictionRun = 3;
        this.minEvictableIdleTimeMillis = 1800000L;
        this.maxEvictableIdleTimeMillis = 25200000L;
        this.keepAliveBetweenTimeMillis = 120000L;
        this.phyTimeoutMillis = -1L;
        this.phyMaxUseCount = -1L;
        this.removeAbandonedTimeoutMillis = 300000L;
        this.maxOpenPreparedStatements = -1;
        this.timeBetweenConnectErrorMillis = 500L;
        this.validConnectionChecker = null;
        this.activeConnections = new IdentityHashMap();
        this.connectionErrorRetryAttempts = 1;
        this.breakAfterAcquireFailure = false;
        this.transactionThresholdMillis = 0L;
        this.createdTime = new Date();
        this.errorCount = 0L;
        this.dupCloseCount = 0L;
        this.startTransactionCount = 0L;
        this.commitCount = 0L;
        this.rollbackCount = 0L;
        this.cachedPreparedStatementHitCount = 0L;
        this.preparedStatementCount = 0L;
        this.closedPreparedStatementCount = 0L;
        this.cachedPreparedStatementCount = 0L;
        this.cachedPreparedStatementDeleteCount = 0L;
        this.cachedPreparedStatementMissCount = 0L;
        this.transactionHistogram = new Histogram(new long[]{1L, 10L, 100L, 1000L, 10000L, 100000L});
        this.dupCloseLogEnable = false;
        this.executeCount = 0L;
        this.executeQueryCount = 0L;
        this.executeUpdateCount = 0L;
        this.executeBatchCount = 0L;
        this.isOracle = false;
        this.isMySql = false;
        this.useOracleImplicitCache = true;
        this.activeConnectionLock = new ReentrantLock();
        this.createErrorCount = 0;
        this.creatingCount = 0;
        this.directCreateCount = 0;
        this.createCount = 0L;
        this.destroyCount = 0L;
        this.createStartNanos = 0L;
        this.useUnfairLock = null;
        this.useLocalSessionState = true;
        this.statLogger = new DruidDataSourceStatLoggerImpl();
        this.asyncCloseConnectionEnable = false;
        this.maxCreateTaskCount = 3;
        this.failFast = false;
        this.failContinuous = 0;
        this.failContinuousTimeMillis = 0L;
        this.initVariants = false;
        this.initGlobalVariants = false;
        this.onFatalError = false;
        this.onFatalErrorMaxActive = 0;
        this.fatalErrorCount = 0;
        this.fatalErrorCountLastShrink = 0;
        this.lastFatalErrorTimeMillis = 0L;
        this.lastFatalErrorSql = null;
        this.lastFatalError = null;
        this.connectionIdSeed = 10000L;
        this.statementIdSeed = 20000L;
        this.resultSetIdSeed = 50000L;
        this.transactionIdSeed = 60000L;
        this.metaDataIdSeed = 80000L;
        this.lock = new ReentrantLock(lockFair);
        this.notEmpty = this.lock.newCondition();
        this.empty = this.lock.newCondition();
    }
```
<a name="H7i5q"></a>
## @AutoConfigureBefore({DataSourceAutoConfiguration.class})

- 加载当前类之后再去加载DataSourceAutoConfiguration类,因为
- DataSourceAutoConfiguration类的PooledDataSourceConfiguration：
```java
@Configuration(
        proxyBeanMethods = false
    )
    @Conditional({DataSourceAutoConfiguration.PooledDataSourceCondition.class})
    @ConditionalOnMissingBean({DataSource.class, XADataSource.class})
    @Import({Hikari.class, Tomcat.class, Dbcp2.class, OracleUcp.class, Generic.class, DataSourceJmxConfiguration.class})
    protected static class PooledDataSourceConfiguration {
        protected PooledDataSourceConfiguration() {
        }
    }
```

- Hikari.class：
```java
 @Configuration(
        proxyBeanMethods = false
    )
    @ConditionalOnClass({HikariDataSource.class})
    @ConditionalOnMissingBean({DataSource.class})
    @ConditionalOnProperty(
        name = {"spring.datasource.type"},
        havingValue = "com.zaxxer.hikari.HikariDataSource",
        matchIfMissing = true
    )
    static class Hikari {
        Hikari() {
        }
 
        @Bean
        @ConfigurationProperties(
            prefix = "spring.datasource.hikari"
        )
        HikariDataSource dataSource(DataSourceProperties properties) {
            HikariDataSource dataSource = (HikariDataSource)DataSourceConfiguration.createDataSource(properties, HikariDataSource.class);
            if (StringUtils.hasText(properties.getName())) {
                dataSource.setPoolName(properties.getName());
            }
 
            return dataSource;
        }
    }
```

- PooledDataSourceConfiguration的@ConditionalOnMissingBean({ DataSource.class, XADataSource.class }) 表示如果存在数据源就不会被加载，相应Import里面默认的Hikari连接池就不会被spring加载，这样就避免了数据源的冲突
<a name="ow7IU"></a>
# 自定义的DataSource优先级大于默认的DataSource(默认的DataSource使用的是HikariDataSource)
<a name="jAvfj"></a>
## @EnableConfigurationProperties({DruidStatProperties.class, DataSourceProperties.class})

- 将DruidStatProperties、DataSourceProperties 2个配置类注入spring容器。
- DruidStatProperties：监控有关配置类。
- DataSourceProperties：数据源配置类。
<a name="Kzz1p"></a>
## @Import()

- @Import注解就是之前xml配置中的import标签，可以用于依赖包、三方包中bean的配置和加载。这里是Druid监控有关配置类的加载。
<a name="pg8Fk"></a>
## dataSource()
```java
@Bean(
        initMethod = "init"
    )
    @ConditionalOnMissingBean
    public DataSource dataSource() {
        LOGGER.info("Init DruidDataSource");
        return new DruidDataSourceWrapper();
    }
```

- 创建DruidDataSourceWrapper(DruidDataSource的包装类)，并且在创建Bean之后执行init方法。init方法在父类DruidDataSource中存在。
```java

// 注入spring.datasource.druid的配置
@ConfigurationProperties("spring.datasource.druid")
public class DruidDataSourceWrapper extends DruidDataSource implements InitializingBean {
    @Autowired
    private DataSourceProperties basicProperties;
 
    public DruidDataSourceWrapper() {
    }
	// 如果没有配置spring.datasource.druid.username/password/url/driverClassName，
    // 而配置了spring.datasource.username/password/url/driverClassName，那么就使用spring.datasource.username/password/url/driverClassName的属性值
    public void afterPropertiesSet() throws Exception {
        if (super.getUsername() == null) {
            super.setUsername(this.basicProperties.determineUsername());
        }
 
        if (super.getPassword() == null) {
            super.setPassword(this.basicProperties.determinePassword());
        }
 
        if (super.getUrl() == null) {
            super.setUrl(this.basicProperties.determineUrl());
        }
 
        if (super.getDriverClassName() == null) {
            super.setDriverClassName(this.basicProperties.getDriverClassName());
        }
 
    }
    ...
}
 
@ConfigurationProperties(
    prefix = "spring.datasource"
)
public class DataSourceProperties implements BeanClassLoaderAware, InitializingBean {}
```
[文章出处](https://blog.csdn.net/yzh_1346983557/article/details/117673280)
