title: MyBatis返回Map键值对数据Key值大小写问题
tags:
  - MyBatis
categories:
  - Dev
  - Frame
date: 2020-01-23 08:56:00
cover: true

---
![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/mybatis.png)
<!-- more -->
## Controller
```
@RestController
@RequestMapping("/web")
public class MapKeyTest {
    @Autowired
    private InvoicingBuyOrderService invoicingBuyOrderService;

    @GetMapping("/testKey")
    public Map<String,Object> testMayKey(String id){
        Map<String,Object> map = new HashMap<>();
        Map<String,String> result = this.invoicingBuyOrderService.test(id);
        map.put("data",result);
        return map;
    }
}
```
## Service
```
 Map<String,String> test(String id);
```
## ServiceImpl
```
    @Override
    public Map<String, String> test(String id) {
          Map<String, String> map = this.invoicingPackageunitMapper.selectPackageNameByCommodityDetailId(id);
          return map;
    }
```
## Mapper
```
Map<String,String> selectPackageNameByCommodityDetailId(@Param("commodityDetailId") String commodityDetailId);
```
## XML
### ①我们不设置字段的别名
```
<select id="selectPackageNameByCommodityDetailId" 
        parameterType="java.lang.String" resultType="java.util.HashMap">
select * from invoicing_commoditydetailprice icd where icd.COMMODITYDETAILPRICE_ID=#{commodityDetailId}
```
postMan请求：
`localhost:8001/FHSHGL/web/testKey?id=15b67d1756134936a6fbc8a5d5007bef`
返回值：
```
{
    "data": {
        "SMALL_PACKAGE_COUNT": "300",
        "SMALL_PACKAGE_UNIT_ID": "0f899de1f92911e8bb4500ffa0c803da",
        "PHP_ID": 80270,
        "MIDDLE_PACKAGE_UNIT_ID": "0f8995cef92911e8bb4500ffa0c803da",
        "CREATED_TIME": 1570839233000,
        "UPDATED_TIME": 1570839233000,
        "PRODUCTION_TIME": 1522425600000,
        "IN_PRICE": "15.00",
        "MIDDLE_PACKAGE_COUNT": "20",
        "OUT_PRICE": "15.00",
        "INVOICING_COMMODITY_ID": "25422c6135ed497dafa5f6e50fd6abcd",
        "CODE": "11325882056318109808267349261365",
        "BIG_PACKAGE_UNIT_ID": "0f894c61f92911e8bb4500ffa0c803da",
        "COMMODITYDETAILPRICE_ID": "15b67d1756134936a6fbc8a5d5007bef"
    }
}
```
### ②设置返回字段的别名
```
  <select id="selectPackageNameByCommodityDetailId" parameterType="java.lang.String" resultType="java.util.HashMap">
    SELECT
        ips.`NAME` AS smallName,
        ipm.`NAME` AS middleName,
        ipb.`NAME` AS bigName
    FROM
        invoicing_commoditydetailprice icd
    LEFT JOIN invoicing_packageunit ips ON icd.SMALL_PACKAGE_UNIT_ID = ips.PACKAGEUNIT_ID
    LEFT JOIN invoicing_packageunit ipm ON icd.MIDDLE_PACKAGE_UNIT_ID = ipm.PACKAGEUNIT_ID
    LEFT JOIN invoicing_packageunit ipb ON icd.BIG_PACKAGE_UNIT_ID = ipb.PACKAGEUNIT_ID
    WHERE icd.COMMODITYDETAILPRICE_ID=#{commodityDetailId}
  </select>
```
postMan请求：
`localhost:8001/FHSHGL/web/testKey?id=15b67d1756134936a6fbc8a5d5007bef`
返回值：
```
{
    "data": {
        "bigName": "箱",
        "middleName": "瓶",
        "smallName": "克"
    }
}
```
## springboot中处理mybatis返回Map时key值的大小写
为了统一不同数据库返回key值大小写不一致的问题，特自定义ObjectWrapperFactory来做统一的处理
### 1、首先自定义MapWrapper
```
/**
 * 将Map的key全部转换为小写
 * */
public class MapKeyLowerWrapper extends MapWrapper {

    public MapKeyLowerWrapper(MetaObject metaObject, Map<String, Object> map) {
        super(metaObject, map);
    }
    @Override
    public String findProperty(String name, boolean useCamelCaseMapping) {
        return name==null?"":name.toLowerCase() ;
    }
}
```
### 2、自定义ObjectWrapperFactory
mybatis默认的ObjectWrapperFactory
```
public class DefaultObjectWrapperFactory implements ObjectWrapperFactory {
    public boolean hasWrapperFor(Object object) {
        return false;
    }

    public ObjectWrapper getWrapperFor(MetaObject metaObject, Object object) {
        throw new ReflectionException(
                "The DefaultObjectWrapperFactory should never be called to provide an ObjectWrapper.");
    }
}
```
我们自定义的如下：
```
public class MapWrapperFactory implements ObjectWrapperFactory {

    @Override
    public boolean hasWrapperFor(Object object) {
         return object != null && object instanceof Map;
    }

    @Override
    public ObjectWrapper getWrapperFor(MetaObject metaObject, Object object) {
        return new MapKeyLowerWrapper(metaObject, (Map) object);
    }
}
```
### 3、在mybatis的配置中添加 MapWrapperFactory 的配置
```
@Bean(name = "sqlSessionFactory")
@ConditionalOnBean(name = "dataSource")
public SqlSessionFactory sqlSessionFactoryBean(@Qualifier("dataSource") DataSource dataSource) {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setObjectWrapperFactory(new MapWrapperFactory());
        bean.setDataSource(dataSource);
        // 添加XML目录
        ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        try {
            bean.setMapperLocations(resolver.getResources("classpath*:com/ultrapower/ioss/**/mapper/**/*.xml"));
            return bean.getObject();
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }
}

@Bean("sqlSessionTemplate")
@ConditionalOnBean(name = "sqlSessionFactory")
public SqlSessionTemplate sqlSessionTemplate(
            @Qualifier("sqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
}

@ConditionalOnBean(name = "dataSource")
@Bean(name = "transactionManager")
public PlatformTransactionManager transactionManager(@Qualifier("dataSource") DataSource dataSource) {
        DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager(dataSource);
        return dataSourceTransactionManager;
}
```