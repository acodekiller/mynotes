# <font color=green>1、Mybatis简介</font>

## <font color=blue>1）Mybatis是什么</font>

MyBatis 是一款优秀的持久层框架，一个半 ORM（对象关系映射）框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

## <font color=blue>2）ORM是什么</font>

ORM（Object Relational Mapping），对象关系映射，是一种为了解决关系型数据库数据与简单Java对象（POJO）的映射关系的技术。简单的说，ORM是通过使用描述对象和数据库之间映射的元数据，将程序中的对象自动持久化到关系型数据库中。

## <font color=blue>3）为什么说Mybatis是半自动ORM映射工具？它与全自动的区别在哪里？</font>

Hibernate属于全自动ORM映射工具，使用Hibernate查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。

而Mybatis在查询关联对象或关联集合对象时，需要手动编写sql来完成，所以，称之为半自动ORM映射工具。

## <font color=blue>4）JDBC编程有哪些不足之处，MyBatis是如何解决这些问题的？</font>

1、数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库连接池可解决此问题。

**解决：**在mybatis-config.xml中配置数据链接池，使用连接池管理数据库连接。

2、Sql语句写在代码中造成代码不易维护，实际应用sql变化的可能较大，sql变动需要改变java代码。

**解决：**将Sql语句配置在XXXXmapper.xml文件中与java代码分离。

3、向sql语句传参数麻烦，因为sql语句的where条件不一定，可能多也可能少，占位符需要和参数一一对应。

**解决：** Mybatis自动将java对象映射至sql语句。

4、对结果集解析麻烦，sql变化导致解析代码变化，且解析前需要遍历，如果能将数据库记录封装成pojo对象解析比较方便。

**解决：**Mybatis自动将sql执行结果映射至java对象。

## <font color=blue>5）mybatis的优缺点</font>

优点

与传统的数据库访问技术相比，ORM有以下优点：

- 基于SQL语句编程，相当灵活，不会对应用程序或者数据库的现有设计造成任何影响，SQL写在XML里，解除sql与程序代码的耦合，便于统一管理；提供XML标签，支持编写动态SQL语句，并可重用
- 与JDBC相比，减少了50%以上的代码量，消除了JDBC大量冗余的代码，不需要手动开关连接
- 很好的与各种数据库兼容（因为MyBatis使用JDBC来连接数据库，所以只要JDBC支持的数据库MyBatis都支持）
- 提供映射标签，支持对象与数据库的ORM字段关系映射；提供对象关系映射标签，支持对象关系组件维护
- 能够与Spring很好的集成

缺点

- SQL语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写SQL语句的功底有一定要求
- SQL语句依赖于数据库，导致数据库移植性差，不能随意更换数据库

## <font color=blue>6）Hibernate 和 MyBatis 的区别</font>

**相同点**

都是对jdbc的封装，都是持久层的框架，都用于dao层的开发。

**不同点**

- 映射关系
  - MyBatis 是一个半自动映射的框架，配置Java对象与sql语句执行结果的对应关系，多表关联关系配置简单
    Hibernate 是一个全表映射的框架，配置Java对象与数据库表的对应关系，多表关联关系配置复杂
- SQL优化和移植性
  - Hibernate 对SQL语句封装，提供了日志、缓存、级联（级联比 MyBatis 强大）等特性，此外还提供 HQL（Hibernate Query Language）操作数据库，数据库无关性支持好，但会多消耗性能。如果项目需要支持多种数据库，代码开发量少，但SQL语句优化困难。
  - MyBatis 需要手动编写 SQL，支持动态 SQL、处理列表、动态生成表名、支持存储过程。开发工作量相对大些。直接使用SQL语句操作数据库，不支持数据库无关性，但sql语句优化容易。
- 开发难易程度和学习成本
  - Hibernate 是重量级框架，学习使用门槛高，适合于需求相对稳定，中小型的项目，比如：办公自动化系统
  - MyBatis 是轻量级框架，学习使用门槛低，适合于需求变化频繁，大型的项目，比如：互联网电子商务系统

**总结**

MyBatis 是一个小巧、方便、高效、简单、直接、半自动化的持久层框架，

Hibernate 是一个强大、方便、高效、复杂、间接、全自动化的持久层框架。

# <font color=green>2、MyBatis的解析和运行原理</font>

## <font color=blue>1）MyBatis编程步骤是什么样的？</font>

1、 创建SqlSessionFactory

2、 通过SqlSessionFactory创建SqlSession

3、 通过sqlsession执行数据库操作

4、 调用session.commit()提交事务

5、 调用session.close()关闭会话

## <font color=blue>2）Mybatis的工作原理</font>

![](imgs/mybatis-1.png)

1）读取 MyBatis 配置文件：mybatis-config.xml 为 MyBatis 的全局配置文件，配置了 MyBatis 的运行环境等信息，例如数据库连接信息。

2）加载映射文件。映射文件即 SQL 映射文件，该文件中配置了操作数据库的 SQL 语句，需要在 MyBatis 配置文件 mybatis-config.xml 中加载。mybatis-config.xml 文件可以加载多个映射文件，每个文件对应数据库中的一张表。

3）构造会话工厂：通过 MyBatis 的环境等配置信息构建会话工厂 SqlSessionFactory。

4）创建会话对象：由会话工厂创建 SqlSession 对象，该对象中包含了执行 SQL 语句的所有方法。

5）Executor 执行器：MyBatis 底层定义了一个 Executor 接口来操作数据库，它将根据 SqlSession 传递的参数动态地生成需要执行的 SQL 语句，同时负责查询缓存的维护。

6）MappedStatement 对象：在 Executor 接口的执行方法中有一个 MappedStatement 类型的参数，该参数是对映射信息的封装，用于存储要映射的 SQL 语句的 id、参数等信息。

7）输入参数映射：输入参数类型可以是 Map、List 等集合类型，也可以是基本数据类型和 POJO 类型。输入参数映射过程类似于 JDBC 对 preparedStatement 对象设置参数的过程。

8）输出结果映射：输出结果类型可以是 Map、 List 等集合类型，也可以是基本数据类型和 POJO 类型。输出结果映射过程类似于 JDBC 对结果集的解析过程。

> **预编译**
>
> 1. **预编译起源**
>
> （1）数据库SQL语句编译特性：
>
>  数据库接受到sql语句之后，需要词法和语义解析，优化sql语句，制定执行计划。这需要花费一些时间。但是很多情况，我们的一条sql语句可能会反复执行，或者`每次执行的时候只有个别的值不同`（比如query的where子句值不同，update的set子句值不同,insert的values值不同）。
>
>  （2）减少编译的方法
>
>  如果每次都需要经过上面的词法语义解析、语句优化、制定执行计划等，则效率就明显不行了。为了解决上面的问题，于是就有了预编译，预编译语句就是将这类语句中的`值用占位符替代`，可以视为将`sql语句模板化或者说参数化`。一次编译、多次运行，省去了解析优化等过程。
>
>  （3）缓存预编译
>
>  预编译语句被DB的编译器编译后的执行代码被缓存下来,那么下次调用时只要是相同的预编译语句就不需要编译,只要将参数直接传入编译过的语句执行代码中(相当于一个涵数)就会得到执行。
>  `并不是所以预编译语句都一定会被缓存`,数据库本身会用一种策略（内部机制）。
>
>  (4) 预编译的实现方法
>
>  预编译是通过PreparedStatement和占位符来实现的。
>
> 2. **预编译的作用**
>
> - 预编译阶段可以优化 sql 的执行
>
>  预编译之后的 sql 多数情况下可以直接执行，DBMS 不需要再次编译，越复杂的sql，编译的复杂度将越大，预编译阶段可以合并多次操作为一个操作。可以提升性能。
>
> - 防止SQL注入
>
>  使用预编译，而其后注入的参数将`不会再进行SQL编译`。也就是说其后注入进来的参数系统将不会认为它会是一条SQL语句，而默认其是`一个参数`，参数中的or或者and 等就不是SQL语法保留字了。

## <font color=blue>3）Mybatis都有哪些Executor执行器？它们之间的区别是什么？</font>

Mybatis有三种基本的Executor执行器，SimpleExecutor、ReuseExecutor、BatchExecutor。

**SimpleExecutor**：每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象。

**ReuseExecutor**：执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，用完后，不关闭Statement对象，而是放置于Map<String, Statement>内，供下一次使用。简言之，就是重复使用Statement对象。

**BatchExecutor**：执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同。

作用范围：Executor的这些特点，都严格限制在SqlSession生命周期范围内。

# <font color=green>3、映射器</font>

## <font color=blue>1）#{}和${}的区别</font>

1. #{}是占位符，预编译处理；${}是拼接符，字符串替换，没有预编译处理；
2. #{}对应的变量会自动加上单引号，${}对应的变量不会加上单引号；
3. #{}能防止sql 注入，${}不能防止sql 注入；

> #{} 和 ${} 在使用中的技巧和建议
>
> （1）不论是单个参数，还是多个参数，一律都建议使用注解@Param("")
>
> （2）能用 #{} 的地方就用 #{}，不用或少用 ${}
>
> （3）表名作参数时，必须用 ${}。如：select * from ${tableName}
>
> （4）order by 时，必须用 ${}。如：select * from t_user order by ${columnName}
>
> （5）使用 ${} 时，要注意何时加或不加单引号，即 ${} 和 '${}'

## <font color=blue>3）模糊查询like语句该怎么写</font>

（1）’%${question}%’ 可能引起SQL注入，不推荐

（2）"%"#{question}"%" 注意：因为#{…}解析成sql语句时候，会在变量外侧自动加单引号’ '，所以这里 % 需要使用双引号" "，不能使用单引号 ’ '，不然会查不到任何结果。

（3）CONCAT(’%’,#{question},’%’) 使用CONCAT()函数，推荐

（4）使用bind标签

```xml
<select id="listUserLikeUsername" resultType="com.jourwon.pojo.User">
　　<bind name="pattern" value="'%' + username + '%'" />
　　select id,sex,age,username,password from person where username LIKE #{pattern}
</select>
```

## <font color=blue>4）Mybatis如何执行批量操作</font>

- **使用foreach标签**

> foreach主要用在构建in条件中，它可以在SQL语句中进行迭代一个集合。foreach标签的属性主要有item，index，collection，open，separator，close。
>
> - item　　   表示集合中每一个元素进行迭代时的别名，随便起的变量名；
> - index　　 指定一个名字，用于表示在迭代过程中，每次迭代到的位置，不常用；
> - open　　 表示该语句以什么开始，常用“(”；
> - separator 表示在每次进行迭代之间以什么符号作为分隔符，常用“,”；
> - close　　 表示以什么结束，常用“)”。
>
> 具体用法：
>
> ```xml
> <!-- 批量保存(foreach插入多条数据两种方法)
>        int addEmpsBatch(@Param("emps") List<Employee> emps); -->
> <!-- MySQL下批量保存，可以foreach遍历 mysql支持values(),(),()语法 --> //推荐使用
> <insert id="addEmpsBatch">
>     INSERT INTO emp(ename,gender,email,did)
>     VALUES
>     <foreach collection="emps" item="emp" separator=",">
>         (#{emp.eName},#{emp.gender},#{emp.email},#{emp.dept.id})
>     </foreach>
> </insert>
> ```
>
> ```xml
> <!-- 这种方式需要数据库连接属性allowMutiQueries=true的支持
>  如jdbc.url=jdbc:mysql://localhost:3306/mybatis?allowMultiQueries=true -->  
> <insert id="addEmpsBatch">
>     <foreach collection="emps" item="emp" separator=";">                                 
>         INSERT INTO emp(ename,gender,email,did)
>         VALUES(#{emp.eName},#{emp.gender},#{emp.email},#{emp.dept.id})
>     </foreach>
> </insert>
> ```

- **使用ExecutorType.BATCH**

> Mybatis内置的ExecutorType有3种，默认为simple,该模式下它为每个语句的执行创建一个新的预处理语句，单条提交sql；而batch模式重复使用已经预处理的语句，并且批量执行所有更新语句，显然batch性能将更优； 但batch模式也有自己的问题，比如在Insert操作时，在事务没有提交之前，是没有办法获取到自增的id，这在某型情形下是不符合业务要求的
>
> 具体用法如下：
>
> ```java
> //批量保存方法测试
> @Test  
> public void testBatch() throws IOException{
> SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
> //可以执行批量操作的sqlSession
> SqlSession openSession = sqlSessionFactory.openSession(ExecutorType.BATCH);
> 
> //批量保存执行前时间
> long start = System.currentTimeMillis();
> try {
> EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);
> for (int i = 0; i < 1000; i++) {
>    mapper.addEmp(new Employee(UUID.randomUUID().toString().substring(0, 5), "b", "1"));
> }
> 
> openSession.commit();
> long end = System.currentTimeMillis();
> //批量保存执行后的时间
> System.out.println("执行时长" + (end - start));
> //批量 预编译sql一次==》设置参数==》10000次==》执行1次   677
> //非批量  （预编译=设置参数=执行 ）==》10000次   1121
> 
> } finally {
> openSession.close();
> }
> }
> ```
>
> mapper和mapper.xml如下
>
> ```java
> public interface EmployeeMapper {   
> //批量保存员工
> Long addEmp(Employee employee);
> }
> ```
>
> ```xml
> <mapper namespace="com.jourwon.mapper.EmployeeMapper"
> <!--批量保存员工 -->
> <insert id="addEmp">
> insert into employee(lastName,email,gender)
> values(#{lastName},#{email},#{gender})
> </insert>
> </mapper>
> 
> ```

## <font color=blue>5）简述Mybatis的Xml映射文件和Mybatis内部数据结构之间的映射关系？</font>

Mybatis将所有Xml配置信息都封装到All-In-One重量级对象Configuration内部。在Xml映射文件中，\<parameterMap>标签会被解析为ParameterMap对象，其每个子元素会被解析为ParameterMapping对象。\<resultMap>标签会被解析为ResultMap对象，其每个子元素会被解析为ResultMapping对象。每一个\<select>、\<insert>、\<update>、\<delete>标签均会被解析为MappedStatement对象，标签内的sql会被解析为BoundSql对象。

# <font color=green>4、高级查询</font>

## <font color=blue>1）MyBatis实现一对一，一对多有几种方式，怎么操作的？</font>

有联合查询和嵌套查询。

- 联合查询是几个表联合查询，只查询一次，通过在resultMap里面的association，collection节点配置一对一，一对多的类就可以完成
- 嵌套查询是先查一个表，根据这个表里面的结果的外键id，去再另外一个表里面查询数据，也是通过配置association，collection，但另外一个表的查询通过select节点配置。

## <font color=blue>2）Mybatis是否可以映射Enum枚举类？</font>

Mybatis可以映射枚举类，不单可以映射枚举类，Mybatis可以映射任何对象到表的一列上。映射方式为自定义一个TypeHandler，实现TypeHandler的setParameter()和getResult()接口方法。

TypeHandler有两个作用，一是完成从javaType至jdbcType的转换，二是完成jdbcType至javaType的转换，体现为setParameter()和getResult()两个方法，分别代表设置sql问号占位符参数和获取列查询结果。

# <font color=green>5、插件模块</font>

## <font color=blue>Mybatis是如何进行分页的？分页插件的原理是什么？</font>

Mybatis使用RowBounds对象进行分页，它是针对ResultSet结果集执行的内存分页，而非物理分页，可以在sql内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

# <font color=green>6、缓存</font>

## <font color=blue>1）Mybatis的一级、二级缓存</font>

1）一级缓存: 基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空，默认打开一级缓存。

2）二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同在于其存储作用域为 Mapper(Namespace)，并且可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现Serializable序列化接口(可用来保存对象的状态),可在它的映射文件中配置<cache/> ；

3）对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存Namespaces)的进行了C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。

## <font color=blue>2）为什么MyBatis有二级缓存，还要用redis</font>

- Mybatis一级缓存作用域是session，session commit之后缓存就失效了；
- Mybatis二级缓存作用域是sessionfactory，该缓存是以namespace为单位的（也就是一个xxxmapper.xml文件），不同namespace下的操作互不影响；
- 所有对数据表的改变操作都会刷新缓存，但是一般不要用二级缓存，因为：如果在UserMapper.xml中有大多数针对user单表的操作，但是另一个xxxMapper.xml也有对user单表的操作的话，这会导致user在两个命名空间不一致。
- 而Redis是中央缓存数据库，不仅不会出现以上问题，还可以保证微服务架构下所有服务使用统一的缓存数据。

