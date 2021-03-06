# mybatis-crud
mybatis-crud增强，目前仅支持oracle、mysql, [使用说明](https://wpyuan.github.io/mybatis-crud-source/)

## 注意
### 1、为了避免出现异常
```
Error setting null for parameter #6 with JdbcType OTHER . Try setting a different JdbcType for this parameter or a different jdbcTypeForNull configuration property. Cause: java.sql.SQLException: 无效的列类型: 1111
```
### 需要做出配置

#### 1.1、在使用oracle数据源的时候需要配置`application.properties`：
```
mybatis.configuration.jdbc-type-for-null=NULL
```
或者是配置`mybatis-config.xml`：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="jdbcTypeForNull" value="NULL" />
    </settings>
</configuration>
```
```xml
<bean id="myAppSqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean" name="myAppSqlSessionFactory">
    <property name="dataSource" ref="myAppDataSource" />
    <property name="typeAliasesPackage" value="com.myapp.model" />
    <property name="configLocation" value="mybatis-config.xml"/>
</bean>
```
#### 1.2、在使用多数据源并含有oracle的时候，需添加如下配置

在SqlSessionFactory配置里面加入`sqlSessionFactory.getConfiguration().setJdbcTypeForNull(JdbcType.NULL);`
```
@Bean
public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
    SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
    
    ...
    
    SqlSessionFactory sqlSessionFactory = sessionFactory.getObject();

    sqlSessionFactory.getConfiguration().setJdbcTypeForNull(JdbcType.NULL);

    return sqlSessionFactory;
}
```

### 2、若项目需要使用like转义处理器`LikeEscapeHandler`或自定义`SqlSessionFactory`，则需排除预注册的bean

#### 2.1、需使用like转义处理器`LikeEscapeHandler`或使用自定义处理器的，则需排除预注册的`PluginInterceptor`
具体操作如下：
```java
// 自定义插件配置的请排除预定义的插件配置PluginInterceptor.class
@SpringBootApplication(exclude = {PluginInterceptor.class})
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }
}
```

#### 2.2、需使用自定义`SqlSessionFactory`的，则需排除预注册的`MyBatisSqlSessionFactoryConfig`
```java
// 自定义SqlSessionFactory的请排除预定义的MyBatisSqlSessionFactoryConfig.class
@SpringBootApplication(exclude = {MyBatisSqlSessionFactoryConfig.class})
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }
}
```

# 其他

需要加入源码编写的可提issue申请，或联系作者490176245@qq.com
