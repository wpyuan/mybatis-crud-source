# mybatis-crud
mybatis-crud增强，目前仅支持oracle、mysql

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
