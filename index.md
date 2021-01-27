## 使用说明

## 项目引入

### maven
```xml
<dependency>
  <groupId>com.github.wpyuan</groupId>
  <artifactId>mybatis-crud</artifactId>
  <version>${latest.version}</version>
</dependency>
```
### gradle
```gradle
implementation 'com.github.wpyuan:mybatis-crud:版本号'
```

## 使用设置

### 实体类(以下称`entity`)创建
可以先用`Mybatis-Generator`生成表对应`entity`类，在做下面的修改，需保证字段不缺类型一致

主键字段添加`@com.github.mybatis.crud.annotation.Id`注解、类上添加`@com.github.mybatis.crud.annotation.Table`注解，举例如下：

`mysql`数据库的`employee`表，对应的`entity`
```java
@Table(name = "employee")
public class Employee {
    @Id
    private Long id;

    ...

}
```

`oracle`数据库的`employee`表，对应的`entity`
```java
@Table("employee")
public class EmployeeOra {
    /**
     * @mbg.generated Fri Dec 25 11:16:58 CST 2020
     */
    @Id(sequence = "EMPLOYEE_ID_S")
    private Long id;
    
    ...
}
```
需要注意的是：需在`@com.github.mybatis.crud.annotation.Table`填入`name`值指定表名，`oracle`数据库需在`@com.github.mybatis.crud.annotation.Id`填入`sequence`值指定序列

### mapper创建
组件已封装多个通用mapper，可选择继承使用，比如`com.github.mybatis.crud.mapper.DefaultMapper`包含默认`CRUD`实现，如下例子：
```java
public interface EmployeeMapper extends DefaultMapper<Employee>, BatchInsertMapper<Employee> {
}
```
其中`com.github.mybatis.crud.mapper.BatchInsertMapper`是批量插入实现，可选择继承，然后配置此mapper扫描路径（这个不做赘述）

## 开始使用
创建表对应`entity`和`mapper`后，以`插入`、`删除`、`修改`和`查询`介绍

### 插入
#### insert 
全字段插入
```java
int insert(E entity);
```

#### insertSelective
有值的字段插入
```java
int insertSelective(E entity);
```

### 删除

#### 根据主键删除记录
```java
int deleteByPrimaryKey(E entity);
```

#### 根据条件删除记录
```java
int delete(Condition<E> condition);
```

其中`Condition<E> condition`包含有

| 方法 | 说明 |
|--- |--- |
| andBetween, between, orBetween | `between` ... `and` ...  的实现|
| diy | 自定义语句实现 |
| andEq, andEq, eq, eq, orEq, orEq | 等于`=`的实现 |
| andGt, andGt, gt, gt, orGt, orGt | 大于`>`的实现 |
| andGtEq, andGtEq, gtEq, gtEq, orGtEq, orGtEq | 大于等于`>=`的实现  | 
| andIn, andIn, in, in, orIn, orIn | `in`的实现 |
| andNotNull, notNull, orNotNull | `is not null`的实现 |
| andNull, isNull, orNull |  `is null`的实现  |
| andLt, andLt, lt, lt, orLt, orLt | 小于`<`的实现  |
| andLtEq, andLtEq, ltEq, ltEq, orLtEq, orLtEq | 小于等于`<=`的实现  |
| andLeftLike, andLeftLike, andLike, andLike, andLikeDiy, andRightLike, andRightLike, like, like, likeDiy, lLike, lLike, orLeftLike, orLeftLike, orLike, orLike, orLikeDiy, orLLike, orLLike, orRightLike, orRightLike, orRLike, orRLike, rLike, rLike |    `like`的实现 |
| andNotBetween, notBetween, orNotBetween | `not between` ... `and` ... 的实现 |
| andNotEq, andNotEq, nEq, nEq, orNotEq, orNotEq | 不等于`!=`的实现  |
| andNotIn, andNotIn, notIn, notIn, orNotIn, orNotIn | `not in` 的实现  |
| andLeftNotLike, andLeftNotLike, andNotLike, andNotLike, andNotLikeDiy, andRightNotLike, andRightNotLike, lNotLike, lNotLike, notLike, notLike, notLikeDiy, orLeftNotLike, orLeftNotLike, orLNotLike, orLNotLike, orNotLike, orNotLike, orNotLikeDiy, orRightNotLike, orRightNotLike, orRNotLike, orRNotLike, rNotLike, rNotLike | `not like` 的实现 |

删除`employeeName like '%1/%/2'`、删除`id=1`的记录举例，更多用法会在后续`Condition`、`LikeEscapeHandler`小节讲述:
```java
Employee employee = new Employee();
int updates = mapper.delete(new Condition<Employee>(employee).lLike("employeeName", "1/%/2"));

employee.setId(1L);
updates = mapper.delete(new Condition<Employee>(employee).eq("id")); 
```

### 修改
主要方法如下：
```java
    /**
     * 根据主键更新覆盖
     *
     * @param entity 实体类
     * @return 影响条数
     */
    int updateByPrimaryKey(E entity);

    /**
     * 根据主键更新覆盖有值列
     *
     * @param entity 实体类
     * @return 影响条数
     */
    int updateByPrimaryKeySelective(E entity);

    /**
     * 根据主键更新覆盖指定列
     *
     * @param entity 实体类
     * @param fields 指定列
     * @return 影响条数
     */
    int updateField(@Param("entity") E entity, @Param("fields") String... fields);

    /**
     * 根据条件更新覆盖指定列
     *
     * @param update 条件
     * @return 影响条数
     */
    int update(Update<E> update);
```
这里说明`int update(Update<E> update);`方法的使用：

以指定id条件为例子，这里条件可任意，具体使用更多用法在后续`Condition`小节介绍

- 更新指定列：
更新`id=1`记录的`employeeName`和`bornDate`为`abs`、当前日期，举例：
```java
Employee employee = new Employee();
employee.setId(1L);
employee.setEmployeeName("abs");
employee.setBornDate(new Date());
int updates = mapper.update(Update.<Employee>builder()
       .fields(Arrays.asList("employeeName", "bornDate"))
       .condition(Condition.<Employee>builder(employee).build().eq("id"))
```
- 更新有值列：
更新`id=1`记录的`employeeName`和`bornDate`为`abs`、当前日期，即无须指定`fields`属性，举例：
```java
Employee employee = new Employee();
employee.setId(1L);
employee.setEmployeeName("abs");
employee.setBornDate(new Date());
int updates = mapper.update(Update.<Employee>builder()
     .isUpdateSelective(true)
     .condition(Condition.<Employee>builder(employee).build().eq("id"))
     .build());
```

- 更新全字段：
更新`id=1`记录的所有字段，直接覆盖，举例：
```java
Employee employee = new Employee();
employee.setId(1L);
employee.setEmployeeName("abs");
employee.setBornDate(new Date());
updates = mapper.update(Update.<Employee>builder()
    .isUpdateSelective(false)
    .condition(Condition.<Employee>builder(employee).build().eq("id"))
    .build());
```

### 查询

主要方法如下：

```java

    /**
     * 根据主键查询
     *
     * @param entity 实体类
     * @return 查询结果
     */
    E selectByPrimaryKey(E entity);

    /**
     * 根据条件查询列表
     *
     * @param condition 条件
     * @return 查询结果
     */
    List<E> list(Condition<E> condition);

    /**
     * 根据条件查询一个结果
     *
     * @param condition 条件
     * @return 查询结果
     */
    E detail(Condition<E> condition);

    /**
     * 根据条件复杂查询出结果
     *
     * @param leftJoin 查询条件
     * @return 查询结果
     */
    List<Map<String, Object>> select(LeftJoin<E> leftJoin);

    /**
     * 根据条件复杂查询出结果并转换格式
     *
     * @param resultTypeClass 返回结果类型class
     * @param leftJoin        查询条件
     * @return 查询结果
     */
    <R> List<R> select(Class<R> resultTypeClass, LeftJoin<E> leftJoin);
    
```

这里讲下两个`select`方法的区别：

两个都提供了左外连接的实现，目前出于可维护性考虑，不考虑继续推出多外联表方法实现，建议写入`xml`文件，增加可读性

- 返回值类型为`List<Map<String, Object>>`
这个是直接查询数据库，返回的类型都是数据库类型，未经转换为`javaType`，注意使用，需自行转换
- 返回值类型为`List<R>`
这个根据传入的`Class<R> resultTypeClass`转换为指定的结构返回

这里讲下两个`select`方法的使用：
- 返回值类型为`List<Map<String, Object>>`
```java
Employee employee = new Employee();
UserInfo userInfo = UserInfo.builder().build();

List<Map<String, Object>> data = mapper.select(new LeftJoin<Employee>(Employee.class)
    .field(As.of(UserInfo.class, "userId", "userId"))
    .field(UserInfo.class, "userName")
    on(On.of(Employee.class, "id"), On.of(UserInfo.class, "employeeId"))
);
```
其中涉及`LeftJoin`左外连接、`As`别名设置，即查询列返回的列别名，具体含义在后续`LeftJoin`、`As`、`On`小节说明

- 返回值类型为`List<R>`
```java
Employee employee = new Employee();
employee.setId(1L);
List<EmployeeVO> employeeVOS = mapper.select(EmployeeVO.class,
    new LeftJoin<Employee>(Employee.class)
    .field(As.of(UserInfo.class, "userId", "userId"))
    .field(UserInfo.class, "userName")
    .on(On.of(Employee.class, "id"), On.of(UserInfo.class, "employeeId"), OnCondition.builder(employee).build().eq("id"))
    .where(WhereCondition.builder(employee).build().eq("id"))
);
```
其中`where`指定where条件，涉及`OnCondition`、`WhereCondition`均在后续小节说明

## 其他类说明
### LikeEscapeHandler
like 转义处理器，处理like语句条件值中的`%`,`_`通配符为文本处理，若不加处理器则作为原通配符处理

可选择性启用，启用方式：
```java
@Configuration
public class PluginInterceptor {

    @Bean
    public DefaultMybatisInterceptor defaultMybatisPlugin() {
        DefaultMybatisInterceptor defaultMybatisInterceptor = new DefaultMybatisInterceptor();
        defaultMybatisInterceptor.setPluginHandlers(new LikeEscapeHandler());
        return defaultMybatisInterceptor;
    }
}


@Configuration
public class MyBatisConfig {

   
    @Autowired
    private DefaultMybatisInterceptor defaultMybatisInterceptor;

    @Bean
    public SqlSessionFactory sqlSessionFactory() throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setPlugins(defaultMybatisInterceptor);
        
		...
		
		SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBean.getObject();
		
		...
		
		return sqlSessionFactory;
    }
}
```

### Condition
单表条件构造器，构造方法和功能方法如下

#### 构造方法

|构造器				| 说明	|
| --- | --- |
|Condition(Class<E> eClass)		| 使用 `new Condition(Employee.class)`创建条件构造器，其中`Employee`更换为条件对应的表的entity |
|Condition(Condition<E> source)	| 复制`Condition`方法，不是同一引用 |
|Condition(E entity)			| 由`entity`初始化条件构造器 |

也可以用builder创建条件构造器，如：
```java 

/**
 * 有条件值的entity，构造Condition如下
 */
Employee employee = new Employee();
employee.setId(1L);
Condition.<Employee>builder(employee).build()

/**
 * 无条件值的entity，构造Condition如下
 */
Condition.<Employee>builder(Employee.class).build()

```

#### 功能方法

##### 连接符方法
支持`group`、`and`、`or`连接符，其中:
- `group`分组，即 `(...)` ，举例:
```sql
where (1=1)
```
- `and`且，即`and (...)`，举例：
```sql
where 1=1 and (1=0)
```
- `or`或，即`or (...)`，举例：
```sql
where 1=1 or (1=0)
```

##### 语句方法

| 方法 | 说明 |
|--- |--- |
| andBetween, between, orBetween | `between` ... `and` ...  的实现|
| diy | 自定义语句实现 |
| andEq, andEq, eq, eq, orEq, orEq | 等于`=`的实现 |
| andGt, andGt, gt, gt, orGt, orGt | 大于`>`的实现 |
| andGtEq, andGtEq, gtEq, gtEq, orGtEq, orGtEq | 大于等于`>=`的实现  | 
| andIn, andIn, in, in, orIn, orIn | `in`的实现 |
| andNotNull, notNull, orNotNull | `is not null`的实现 |
| andNull, isNull, orNull |  `is null`的实现  |
| andLt, andLt, lt, lt, orLt, orLt | 小于`<`的实现  |
| andLtEq, andLtEq, ltEq, ltEq, orLtEq, orLtEq | 小于等于`<=`的实现  |
| andLeftLike, andLeftLike, andLike, andLike, andLikeDiy, andRightLike, andRightLike, like, like, likeDiy, lLike, lLike, orLeftLike, orLeftLike, orLike, orLike, orLikeDiy, orLLike, orLLike, orRightLike, orRightLike, orRLike, orRLike, rLike, rLike |    `like`的实现 |
| andNotBetween, notBetween, orNotBetween | `not between` ... `and` ... 的实现 |
| andNotEq, andNotEq, nEq, nEq, orNotEq, orNotEq | 不等于`!=`的实现  |
| andNotIn, andNotIn, notIn, notIn, orNotIn, orNotIn | `not in` 的实现  |
| andLeftNotLike, andLeftNotLike, andNotLike, andNotLike, andNotLikeDiy, andRightNotLike, andRightNotLike, lNotLike, lNotLike, notLike, notLike, notLikeDiy, orLeftNotLike, orLeftNotLike, orLNotLike, orLNotLike, orNotLike, orNotLike, orNotLikeDiy, orRightNotLike, orRightNotLike, orRNotLike, orRNotLike, rNotLike, rNotLike | `not like` 的实现 |

其中`andBetween`与`between`类似没有前缀`and`等效，是为了方便书写和简洁，这里默认了`and`前缀连接符

#### 具体使用
根据如上构造方法、连接符和语句方法，可写出高度灵活复杂的条件，举例如下：
```java
Condition<Employee> condition = new Condition<Employee>(Employee.class)
        .and(
                c -> c.group(
                        c1 -> c1.diy(null, null, "1", "=", 1, null)
                                .and(
                                        c2 -> c2.diy(null, null, "1", "=", 1, null)
                                                .diy(null, "or", "1", "=", 1, null)
                                )
                )
                .or(
                        c3 -> c3.group(
                                c4 -> c4.diy(null, null, "1", "=", 1, null)
                                        .diy(null, "or", "1", "=", 1, null)
                        ).diy(null, "and", "1", "=", 0, null)
                )
        );
```

即实现语句：
```sql
and( ( 1 = 1 and ( 1 = 1 or 1 = 1 ) ) or ( ( 1 = 1 or 1 = 1 ) and 1 = 0 ) )
```

其中的`diy`为自定义语句，可替换上述`语句方法`小点表格中任意方法

为了可读性和易维护性，不建议复杂的语句用组件写，建议写在`xml`文件，这里只是功能展示

更多`Condition`方法尽请期待...

### OnCondition
外连接`On`条件构造器，与`Condition`单表条件构造器不同，专注与构造外连接时，`on`后面的连接条件，即是涉及两表的条件构造器

#### 构造方法
含有`Condition`单表条件构造器所有构造方法外，还有
|构造器								| 说明	|
|---|---|
|OnCondition(Class<?> eClass, Class<?> eClass2)	| 使用`new OnCondition(Employee.class, UserInfo.class)`即初始化出`Employee`和`UserInfo`对应两表的条件构造器|
|OnCondition(Object entity, Object entity2)		| 使用`new OnCondition(employee, userInfo)`即初始化出`employee`和`userInfo`对应两表的条件构造器|

也可以用builder创建条件构造器，与`Condition`单表条件构造器用法一样不做赘述

#### 功能方法

##### 连接符方法
与`Condition`单表条件构造器一样不做赘述
##### 语句方法
含有`Condition`单表条件构造器所有语句方法外，还有
| 方法 | 说明 |
|--- |--- |
| andEq, eq, orEq | 两表等于`=`的实现|
| andNotEq, nEq, orNotEq | 两表不等于`!=`的实现  |
| diy | 两表自定义语句实现 |
| andGt, gt, orGt | 两表大于`>`的实现 |
| andGtEq, gtEq, orGtEq | 两表大于等于`>=`的实现  | 
| andLt, lt, orLt | 两表小于`<`的实现  |
| andLtEq, ltEq, orLtEq | 两表小于等于`<=`的实现  |

#### 具体使用
根据如上方法，可写出两表联表查询，举例如下：
```java
Employee employee = new Employee();
employee.setId(1L);
List<Map<String, Object>> data = mapper.select(new LeftJoin<Employee>(Employee.class)
        .field(UserInfoOra.class,"userName", "userId")
        .on(On.of(Employee.class, "id"), On.of(UserInfo.class, "employeeId"),
                OnCondition.<UserInfo>builder(UserInfo.class).build().eq("isEnable", false),
        ).where(WhereCondition.builder(employee).build().eq("id"))
);
```

即实现语句：
```sql
select e.*, u.user_id, u.user_name
from employee e
left join user_info u on u.employee_id = e.id and u.is_enable = 0
where e.id = 1
```

其中`LeftJoin`、`On`下文单独小节说明

更多`OnCondition`方法尽请期待...

### WhereCondition
外连接`Where`条件构造器，也是涉及两表的条件构造器，与`OnCondition`不同的是，`WhereCondition`专注于`where`的条件，其余全部和`OnCondition`雷同，这里不作赘述

### As
别名标注，用法如下：
```java
new LeftJoin<Employee>(Employee.class)
        .field(As.of(UserInfo.class, "userId", "createdUserId"))
```
即，查询`UserInfo`对应表的`user_id`字段返回`createdUserId`别名，`As`需配合`LeftJoin`等外连接类使用

### On
连接条件标注，用法如下：
```java
new LeftJoin<Employee>(Employee.class)
        .on(On.of(Employee.class, "id"), On.of(UserInfo.class, "employeeId"))
```
即，查询`Employee`对应表左外连`UserInfo`对应表记录，连接条件是`Employee`的`id`字段等于`UserInfo`的`employeeId`字段

### LeftJoin
左外连接构造器

#### 构造方法
|构造器		| 说明	|
| --- |---|
|LeftJoin(Class<E> eClass)	| 即使用`new LeftJoin(Employee.class)`创建左外连接构造器 |
|LeftJoin(E entity)			| 即使用`new LeftJoin(employee)`创建左外连接构造器 |
目前两种方法推荐使用第一种创建

#### 功能方法

|限定符和类型					|方法											| 说明	|
|---|---| --- |
|LeftJoin<E>					|field(As... fieldAs)									| 查询两表的字段别名设置 |
|LeftJoin<E>					|field(Class<?> eClass, String... field)				| 指定查询字段，默认全查主表字段，这里供外连接表查询字段设置 |
|LeftJoin<E>					|on(On onLeft, On onRight, OnCondition... onCondition)	| 外连接`on`条件，`onLeft`与`onRight`指定连接关系，`OnCondition`其他连接条件，`on`上面的条件 不支持`(` `)`符号	|
|LeftJoin<E>					|where(WhereCondition... whereConditions)				| `where`条件, 不支持`(` `)`符号 |

如下举例：
```java
new LeftJoin<Employee>(Employee.class)
        .field(UserInfoOra.class,"userName", "userId")
        .on(On.of(Employee.class, "id"), On.of(UserInfo.class, "employeeId"),
                OnCondition.<UserInfo>builder(UserInfo.class).build().eq("isEnable", false),
        ).where(WhereCondition.builder(employee).build().eq("id"))
```
