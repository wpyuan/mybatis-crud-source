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
其中涉及`As`别名设置，即查询列返回的列别名，具体含义在后续`As`小节，`on`指定连接条件

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
### Condition
### OnCondition
### WhereCondition
### As
### On
