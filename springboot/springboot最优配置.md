## 配置

方法上使用@ConfigurationProperties

```java
@Bean
@ConfigurationProperties(prefix = "spring.datasource.master")
public DataSource masterDb() { return DruidDataSourceBuilder.create().build();}
```

springboot中使用AOP

```java
@Aspect //标注在类上
@Around("@annotation(com.xjt.proxy.DataSourceSelector)")//切面注解
```

springboot中使用 ThreadLocal

```java
public class DataSourceContextHolder {
  private static final ThreadLocal<String> DYNAMIC_DATASOURCE_CONTEXT = new ThreadLocal<>();
  public static void set(String dataSourceType) { 		DYNAMIC_DATASOURCE_CONTEXT.set(dataSourceType)};
}
public static String get() {return DYNAMIC_DATASOURCE_CONTEXT.get();}
public static void clear() {DYNAMIC_DATASOURCE_CONTEXT.remove()}

//在别的类中使用
DataSourceContextHolder.clear();
DataSourceContextHolder.set(dataSourceImport.value().getDataSourceName());
```



## 

## 33

## sds

### sdsd

#### sdsds







































## Dang