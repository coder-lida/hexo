title: 快速搭建Spring Boot项目及常用技术整合
tags:
  - 微服务
categories:
  - 技术
date: 2020-01-01 11:18:00
cover: true

---

![](http://q6pznk9ej.bkt.clouddn.com/img%20%283%29.jpeg)
<!-- more -->
### Spring Boot简介
Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。
### Spring Boot特点
* 创建独立的Spring应用程序

* 嵌入的Tomcat，无需部署WAR文件

* 简化Maven配置

* 自动配置Spring

* 提供生产就绪型功能，如指标，健康检查和外部配置

* 绝对没有代码生成并且对XML也没有配置要求
### 快速入门
####  1、访问http://start.spring.io/构建项目，也可在idea创建如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191218173310114.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RldmlsbGkwMzEw,size_16,color_FFFFFF,t_70)
![step2.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS05ZWMxNTkwMzg4MzE0ZWYxLnBuZw?x-oss-process=image/format,png)
![step3.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0xN2RlNDY2YTU4Yjk1YWQyLnBuZw?x-oss-process=image/format,png)
![step4.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1jNzQwYWIyYzFiMTE2ZGYzLnBuZw?x-oss-process=image/format,png)
#### 2、 springboot默认生成三个文件
##### 2.1 pom.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```
重点就一个gav：spring-boot-starter-web，其他可以删除。
##### 2.2 application.properties
该文件默认为空，springboot的默认启动端口号：8080，可以在改文件修改。建议用yml的格式
```
server:
  port: 8080
```
##### 2.3 启动类文件
```
public class JxcApplication {

    public static void main(String[] args) {
        SpringApplication.run(JxcApplication.class, args);
    }

}
```
##### 2.4 验证springboot
在项目包路径下创建一个Controller，写一个`HelloController `
```
@Controller
public class HelloController {

    @RequestMapping("/")
    @ResponseBody
    public String getHello() {
        return "hello";
    }
}
```
浏览器查看效果
![HelloController.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1lMDhmZmU5ZDVjZWI5Nzg4LnBuZw?x-oss-process=image/format,png)

### 完成项目
#### 完整项目目录
![project.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1kZDM5ZWI1ZTRkOGU5ZjJjLnBuZw?x-oss-process=image/format,png)
#### 1、项目依赖
* web 
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>

      <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>${aspectjweaver.version}</version>
        </dependency>
```
* mysql
```
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
```
* lombok(可选)
```
       <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
```
* pagehelper(可选)
```
       <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>${pagehelper.version}</version>
        </dependency>
```
* JWT(可选)
```
       <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>${jwt.version}</version>
        </dependency>
```
* mybatis
```
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.2</version>
        </dependency>
```
* shiro
```
       <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>1.4.0</version>
        </dependency>
```
* hutool(可选)
```
<dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.0.7</version>
        </dependency>
```
* druid
```
       <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.10</version>
        </dependency>
```
* jdbc
```
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
```
* fastjson
```
      <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>${fastjson.version}</version>
        </dependency>
```
* tomcat
```
       <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-core</artifactId>
        </dependency>
```
附上properties
```
<properties>
        <project.version>1.0</project.version>
        <java.version>1.8</java.version>
        <mysql.version>5.1.25</mysql.version>
        <pagehelper.version>1.2.12</pagehelper.version>
        <jwt.version>0.9.1</jwt.version>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <commons.lang.version>3.9</commons.lang.version>
        <aspectjweaver.version>1.9.4</aspectjweaver.version>
        <fastjson.version>1.2.62</fastjson.version>
    </properties>
```
#### 2、配置文件
##### 2.1修改`application.properties`为`application.yml`
配置端口，项目根路径，spring配置，mybatis配置，分页插件配置
```

server:
  port: 8100
  servlet:
    context-path: /api


spring:
  profiles:
    active: dev
  http:
    encoding:
      charset: UTF-8
      force: true
      enabled: true

mybatis:
  mapper-locations: classpath:/mapper/*.xml
  type-aliases-package: com.example.jxc.domain.entity.*
  configuration:
    cache-enabled: true
    lazy-loading-enabled: true
    multiple-result-sets-enabled: true
    use-column-label: true
    call-setters-on-nulls: true
    local-cache-scope: session
    map-underscore-to-camel-case: true
    default-executor-type: BATCH
    auto-mapping-behavior: PARTIAL

pagehelper:
  helperDialect: mysql
  reasonable: true
  supportMethodsArguments: true
  params: count=countSql

```

mybatis中的configuration配置，这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。 下表描述了设置中各项的意图、默认值等。
| 设置名| 描述| 有效值|默认值|
|-----|-----|------|------|
| cacheEnabled | 全局地开启或关闭配置文件中的所有映射器已经配置的任何缓存。 |  true  false |true|
| lazyLoadingEnabled| 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 fetchType 属性来覆盖该项的开关状态。 |  true  false |false|
| aggressiveLazyLoading|  	当开启时，任何方法的调用都会加载该对象的所有属性。 否则，每个属性会按需加载（参考 lazyLoadTriggerMethods)。 |  true  false |false （在 3.4.1 及之前的版本默认值为 true） |
| multipleResultSetsEnabled| 是否允许单一语句返回多结果集（需要驱动支持）。 |  true  false |true|
| useColumnLabel|  	使用列标签代替列名。不同的驱动在这方面会有不同的表现，具体可参考相关驱动文档或通过测试这两种不同的模式来观察所用驱动的结果。  |  true  false |true|
| useGeneratedKeys| 允许 JDBC 支持自动生成主键，需要驱动支持。 如果设置为 true 则这个设置强制使用自动生成主键，尽管一些驱动不能支持但仍可正常工作（比如 Derby）。 |  true  false |false |
| autoMappingBehavior| 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示取消自动映射；PARTIAL 只会自动映射没有定义嵌套结果集映射的结果集。 FULL 会自动映射任意复杂的结果集（无论是否嵌套）。 | NONE, PARTIAL, FULL  |PARTIAL|
| autoMappingUnknownColumnBehavior| 指定发现自动映射目标未知列（或者未知属性类型）的行为。NONE: 不做任何反应，WARNING: 输出提醒日志 ('org.apache.ibatis.session.AutoMappingUnknownColumnBehavior' 的日志等级必须设置为 WARN) ，FAILING: 映射失败 (抛出 SqlSessionException) |  NONE, WARNING, FAILING  |NONE|
| defaultExecutorType| 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（prepared statements）； BATCH 执行器将重用语句并执行批量更新。 |  SIMPLE REUSE BATCH  |SIMPLE|
| defaultStatementTimeout|  	设置超时时间，它决定驱动等待数据库响应的秒数。  |  任意正整数 |未设置 (null) |
| defaultFetchSize|  	为驱动的结果集获取数量（fetchSize）设置一个提示值。此参数只可以在查询设置中被覆盖。  |  任意正整数 |未设置 (null) |
| defaultResultSetType|  	Specifies a scroll strategy when omit it per statement settings. (Since: 3.5.2)  |  FORWARD_ONLY SCROLL_SENSITIVE SCROLL_INSENSITIVE  DEFAULT(same behavior with 'Not Set')  |Not Set (null) |
| safeRowBoundsEnabled| 允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。  |  true  false |false |
| safeResultHandlerEnabled|允许在嵌套语句中使用分页（ResultHandler）。如果允许使用则设置为 false。 |  true  false |false |
| mapUnderscoreToCamelCase|是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射。 |  true  false |false |
| localCacheScope| MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询。 默认值为 SESSION，这种情况下会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地会话仅用在语句执行上，对相同 SqlSession 的不同调用将不会共享数据。  |  SESSION  STATEMENT |SESSION  |
| jdbcTypeForNull| 当没有为参数提供特定的 JDBC 类型时，为空值指定 JDBC 类型。 某些驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。 |  JdbcType 常量，常用值：NULL, VARCHAR 或 OTHER。|OTHER|
| lazyLoadTriggerMethods| 指定哪个对象的方法触发一次延迟加载。 | 用逗号分隔的方法列表 |equals,clone,hashCode,toString |
| defaultScriptingLanguage| 指定动态 SQL 生成的默认语言。 | 一个类型别名或完全限定类名 |org.apache.ibatis.scripting.xmltags.XMLLanguageDriver |
| defaultEnumTypeHandler|指定 Enum 使用的默认 TypeHandler 。（新增于 3.4.5）  | 一个类型别名或完全限定类名 |org.apache.ibatis.type.EnumTypeHandler |
| callSettersOnNulls| 指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这在依赖于 Map.keySet() 或 null 值初始化的时候比较有用。注意基本类型（int、boolean 等）是不能设置成 null 的。 |  true  false |false |
| returnInstanceForEmptyRow| 当返回行的所有列都是空时，MyBatis默认返回 null。 当开启这个设置时，MyBatis会返回一个空实例。 请注意，它也适用于嵌套的结果集 （如集合或关联）。（新增于 3.4.2）  |  true  false |false |
| logPrefix| 指定 MyBatis 增加到日志名称的前缀。 |  任何字符串 |未设置|
| logImpl| 指定 MyBatis  	指定 MyBatis 所用日志的具体实现，未指定时将自动查找。 |  SLF4J,LOG4J,LOG4J2,JDK_LOGGING,COMMONS_LOGGING,STDOUT_LOGGING,NO_LOGGING |未设置|
| proxyFactory|  	指定 Mybatis 创建具有延迟加载能力的对象所用到的代理工具。 |   	CGLIB ,JAVASSIST |AVASSIST （MyBatis 3.3 以上） |
| vfsImpl| 指定 VFS 的实现  |   	自定义 VFS 的实现的类全限定名，以逗号分隔。 |未设置|
| useActualParamName| 允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的项目必须采用 Java 8 编译，并且加上 -parameters 选项。（新增于 3.4.1）  |  true   false  |true|
| configurationFactory|  	指定一个提供 Configuration 实例的类。 这个被返回的 Configuration 实例用来加载被反序列化对象的延迟加载属性值。 这个类必须包含一个签名为static Configuration getConfiguration() 的方法。（新增于 3.2.3）   |   	类型别名或者全类名.  |未设置|

##### 2.2 新建`application-dev.yml`
配置数据库信息,通过`application.yml`中的active来启用dev配置文件
```
spring:
  profiles:
    active: dev
```
`application-dev.yml`完整配置
```
spring:
  datasource:
    #   数据源基本配置
    username: root
    password:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/fhshgl
    type: com.alibaba.druid.pool.DruidDataSource
    #   数据源其他配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    #   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```
#### 3、数据库连接池
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS05Zjc5NGJmZjQ0ZDk3NjY4LnBuZw?x-oss-process=image/format,png)
```
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
        return  new DruidDataSource();
    }

    /**
     * 配置Druid的监控
     * @return
     */
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String,String> initParams = new HashMap<>();

        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","123456");
        //默认就是允许所有访问
        initParams.put("allow","");
        initParams.put("deny","192.168.15.21");

        bean.setInitParameters(initParams);
        return bean;
    }


    /**
     * 配置一个web监控的filter
     * @return
     */
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());

        Map<String,String> initParams = new HashMap<>();
        initParams.put("exclusions","*.js,*.css,/druid/*");

        bean.setInitParameters(initParams);

        bean.setUrlPatterns(Arrays.asList("/*"));

        return  bean;
    }
}
```
#### 4、shiro
##### 4.1自定义realm
![realm.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0xMzlmZmVlNDViNzQxOWIzLnBuZw?x-oss-process=image/format,png)
代码如下：
```
public class MyRealm extends AuthorizingRealm{

    @Autowired
    private UserService userService;

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //拿到封装好账户密码的token
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        String userName = token.getUsername();
        //用户校验
        User user = this.userService.getUser(userName);
        if (user == null) {
            throw new AuthenticationException("用户名或密码错误！");
        }
        //加盐 计算盐值 保证每个加密后的 MD5 不一样
        ByteSource credentialsSalt = ByteSource.Util.bytes(user.getUsername());
        SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user, user.getPassword(), credentialsSalt,
                this.getName());
        return info;
    }
}
```
##### 4.2shiro配置
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1kMmViZjdkM2UwOWExNzBlLnBuZw?x-oss-process=image/format,png)

```
@Configuration
public class ShiroConfig {

    /**
     * 主要配置一些相应的URL的规则和访问权限
     */
    @Bean
    public ShiroFilterFactoryBean shiroFilter() {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager());
        //拦截器.
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
        //配置退出 过滤器,其中的具体的退出代码Shiro已经替我们实现了
        filterChainDefinitionMap.put("/system/logout", "anon");
        //过滤链定义，从上向下顺序执行，一般将/**放在最为下边
        //authc:所有url都必须认证通过才可以访问; anon:所有url都都可以匿名访问
//        filterChainDefinitionMap.put("/static/**", "anon");
        shiroFilterFactoryBean.setLoginUrl("/system/login");
        filterChainDefinitionMap.put("/**", "authc");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }


    /**
     * 注入 securityManager
     */
    @Bean
    public DefaultWebSecurityManager securityManager() {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        // 设置realm.
        securityManager.setRealm(customRealm());
        return securityManager;
    }

    /**
     * 自定义身份认证 realm;
     * <p>
     * 必须写这个类，并加上 @Bean 注解，目的是注入 MyRealm，
     * 否则会影响 MyRealm 中其他类的依赖注入
     */
    @Bean
    public MyRealm customRealm() {
        return new MyRealm();
    }


    /**
     * 开启Shiro的注解(如@RequiresRoles,@RequiresPermissions),需借助SpringAOP扫描使用Shiro注解的类,并在必要时进行安全逻辑验证
     * 配置以下两个bean(DefaultAdvisorAutoProxyCreator(可选)和AuthorizationAttributeSourceAdvisor)即可实现此功能
     *
     * @return
     */
    @Bean
    @DependsOn({"lifecycleBeanPostProcessor"})
    public DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        advisorAutoProxyCreator.setProxyTargetClass(true);
        return advisorAutoProxyCreator;
    }

    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor() {
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager());
        return authorizationAttributeSourceAdvisor;
    }

    /**
     * Shiro生命周期处理器 ---可以自定的来调用配置在 Spring IOC 容器中 shiro bean 的生命周期方法.
     *
     * @return
     */
    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }
}
```
#### 5、过滤器-跨域过滤
##### 5.1跨域过滤
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS02ODEwMTBkMGZhZGEwYTcwLnBuZw?x-oss-process=image/format,png)
```
public class CostFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse resp = (HttpServletResponse) response;
        String origin = req.getHeader("Origin");
        if (origin == null) {
            origin = req.getHeader("Referer");
        }
        // 允许指定域访问跨域资源
        resp.setHeader("Access-Control-Allow-Origin", origin);
        // 允许客户端携带跨域cookie，此时origin值不能为“*”，只能为指定单一域名
        resp.setHeader("Access-Control-Allow-Credentials", "true");

        if ("OPTIONS".equals(req.getMethod())) {
            String allowMethod = req.getHeader("Access-Control-Request-Method");
            String allowHeaders = req.getHeader("Access-Control-Request-Headers");
            // 浏览器缓存预检请求结果时间,单位:秒
            resp.setHeader("Access-Control-Max-Age", "86400");
            // 允许浏览器在预检请求成功之后发送的实际请求方法名
            resp.setHeader("Access-Control-Allow-Methods", allowMethod);
            // 允许浏览器发送的请求消息头
            resp.setHeader("Access-Control-Allow-Headers", allowHeaders);
            resp.setHeader("Content-Type", "application/json;charset=utf-8");
            return;
        }
        chain.doFilter(request, response);
    }
}
```
##### 5.2 过滤器配置
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS02ZWU3NDQzY2JkMDg2MDVkLnBuZw?x-oss-process=image/format,png)

```
@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean configureFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean<>();
        bean.setName("costFilter");
        CostFilter costFilter = new CostFilter();
        bean.setFilter(costFilter);
        bean.setOrder(1);
        List<String> urlList = new ArrayList<String>();
        urlList.add("/*");
        bean.setUrlPatterns(urlList);
        return bean;
    }
}
```
#### 6、token拦截
##### 6.1JWT
jwt工具类
```
public class JwtUtils {

    public static SecretKey getBase64Key() {
        String stringKey = "MyJwtSecret";
        byte[] encodeKey = Base64.getDecoder().decode(stringKey);
        SecretKey key = new SecretKeySpec(encodeKey, 0, encodeKey.length, "AES");
        return key;
    }

    /**
     * 签发token
     *
     * @param userName 用户名
     * @return token
     */
    public static String create(String userName) {
        Date now = new Date(System.currentTimeMillis());
        String token = Jwts.builder()
                .setIssuedAt(now)
                .setSubject(userName)
                .setExpiration(new Date(System.currentTimeMillis() + 60 * 60 * 1000))
                .signWith(SignatureAlgorithm.HS256, getBase64Key())
                .compact();

        return token;
    }

    /**
     * 解析token
     *
     * @param token token
     * @return 用户名
     */
    public static String parse(String token) {
        String username = null;
        try {
            username = Jwts.parser()
                    .setSigningKey(getBase64Key())
                    .parseClaimsJws(token.replace("Bearer ", ""))
                    .getBody()
                    .getSubject();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return username;
    }

    /**
     * 检验token是否过期
     *
     * @param token
     * @return
     */
    public static boolean verify(String token) {
        Date expiraDate = null;
        Date currentDate = new Date();
        try {
            expiraDate = Jwts.parser()
                    .setSigningKey(getBase64Key())
                    .parseClaimsJws(token.replace("Bearer ", ""))
                    .getBody()
                    .getExpiration();
            if (currentDate.before(expiraDate)) {
                return true;
            } else {
                return false;
            }
        } catch (Exception e) {
            return false;
        }
    }
}
```
##### 6.2token拦截器
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS01MzMwZjU3ZTVkNTcxOTYzLnBuZw?x-oss-process=image/format,png)
```
@Component
public class TokenInterceptor implements HandlerInterceptor {

    public Log log = LogFactory.getLog(TokenInterceptor.class);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler){
        if (request.getMethod().equals("OPTIONS")) {
            response.setStatus(HttpServletResponse.SC_OK);
            return true;
        }
        response.setCharacterEncoding("utf-8");
        String token = request.getHeader("Authorization");
        if (token != null) {
            boolean result = JwtUtils.verify(token);
            if (result) {
                return true;
            }
        }
        log.error("认证失败");
        response.setStatus(HttpServletResponse.SC_NON_AUTHORITATIVE_INFORMATION);
        return false;
    }
}

```
##### 6.3配置拦截器
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1lZjEyNWRjZWNhZjcyODA4LnBuZw?x-oss-process=image/format,png)
```
@Configuration
public class InterceptorConfig extends WebMvcConfigurationSupport {

    @Autowired
    private TokenInterceptor tokenInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(tokenInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/**/login")
                .excludePathPatterns("/**/logOut");
    }
}
```

#### 7、完成一个登录接口`LoginController`
```
@RestController
@RequestMapping("/system")
public class LoginController extends BaseController {

    @Autowired
    private UserService userService;

    /**
     * 浏览器点击登录
     *
     * @param user
     * @return
     */
    @PostMapping("/login")
    public R login(@RequestBody User user) {
        log.debug("------浏览器点击登录------");
        String userName = user.getUsername();
        String passWord = user.getPassword();
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken(userName, MD5.md5Salt(passWord, userName));
        Subject subject = SecurityUtils.getSubject();
        try {
            subject.login(usernamePasswordToken);
            String token = JwtUtils.create(userName);
            return R.ok(R.SUCCESS, R.MSG_SUCCESS, token);
        } catch (AuthenticationException e) {
            e.printStackTrace();
            return R.error(R.MSG_LOGIN_ERROR);
        }
    }
}
```


