# Mybatis

- [Mybatis](#mybatis)
  * [一、简介](#一简介)
  * [二、Mybatis简单使用](#二Mybatis简单使用)
    + [1.安装](#1安装)
    + [2.Mybatis案例](#2Mybatis案例)
    + [3.Mybatis案例改进（官方推荐）](#3Mybatis案例改进官方推荐)
  * [三、Mybatis核心配置](#三Mybatis核心配置)
    + [1.properties（属性）](#1properties属性)
    + [2.settings（设置）](#2settings设置)
    + [3.类型别名（typeAliases）](#3类型别名typeAliases)
    + [4.类型处理器（typeHandlers）](#4类型处理器typeHandlers)
    + [5.对象工厂（objectFactory）](#5对象工厂objectFactory)
    + [6.插件（plugins）](#6插件plugins)
    + [7.环境配置（environments）](#7环境配置environments)
    + [8.数据库厂商标识（databaseIdProvider）](#8数据库厂商标识databaseIdProvider)
    + [9.映射器（mapper）](#9映射器mapper)
  * [四、映射器Mapper.xml配置](#四映射器Mapper.xml配置)
    + [**MybatisUtils**](#MybatisUtils)
    + [1.cache](#1cache)
      - [1.一级缓存](#1一级缓存)
      - [2.二级缓存](#2二级缓存)
      - [3.脏数据的产生和避免](#3脏数据的产生和避免)
    + [2.select](#2select)
    + [3.insert，update和delete](#3insertupdate和delete)
    + [4.sql片段](#4sql片段)
    + [5.参数](#5参数)
    + [6.字符串替换](#6字符串替换)
    + [7.结果映射](#7结果映射)

生成github目录工具：<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## 一、简介

> MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

Mybatis的前身是iBatis，是Clinton Begin在2001年发起的一个开源项目，最初侧重于密码软件的开发，后来发展成为一款基于Java的持久层框架。2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。2013年11月迁移到Github。

与其它ORM（对象关系映射）框架不同，Mybatis并没有将Java对象与数据库表关联起来，而是将Java方法与SQL语句关联，Mybatis允许用户充分利用数据库的各种功能，例如存储过程、试图、各种复杂的查询及数据库的专有特性。与JDBC相比，Mybatis简化了相关代码，SQL语句在一行中就能执行。Mybatis提供了一个映射引擎，声明式地将SQL语句的执行结果与对象树映射起来。

Mybatis支持声明式数据缓存，当一条SQL语句被标记为“可缓存”后，首次执行它时从数据库获取的所有数据会被存储在高速缓存中，后面再执行这条语句时就会从高速缓存中读取结果，而不是再次命中数据库。

tips：

* Mybatis官方文档

  https://mybatis.org/mybatis-3/zh/index.html

* Github代码库

  https://github.com/mybatis/mybatis-3

## 二、Mybatis简单使用

### 1.安装

要使用Mybatis，只需将*mybatis-x.x.x.jar* 文件置于类路径中即可，如果使用Maven构建项目，只需将下面的依赖代码置于pom.xml文件中：

```xml
<dependency>
	<groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>x.x.x</version>
</dependency>
```

### 2.Mybatis案例

1. 创建数据库simple，创建数据表city，city表有两列数据：id，name

   ```sql
   CREATE DATABASE `simple`;
   CREATE TABLE city(
   	id INT PRIMARY KEY AUTO_INCREMENT,
   	NAME VARCHAR(50) NOT NULL
   )ENGINE=INNODB,CHARSET=utf8;
   INSERT INTO `city` VALUES(NULL,'上海');
   INSERT INTO `city` VALUES(NULL,'杭州');
   INSERT INTO `city` VALUES(NULL,'北京');
   ```

2. 使用idea创建Maven项目：simplemybatis

![maven-mybatis](https://z3.ax1x.com/2021/04/27/g963dS.png)

3. 在pom.xml中添加Mybatis，MySQL驱动的依赖和Log4j，JUnit的依赖

   ![g92BeP.png](https://z3.ax1x.com/2021/04/27/g92BeP.png)

4. 在src/main/resources目录下创建Mybatis的核心配置文件mybatis-config.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"/>
               <dataSource type="POOLED">
                   <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                   <property name="url" value="jdbc:mysql://localhost:3306/simple?characterEncoding=utf8&amp;useSSL=true&amp;serverTimezone=Asia/Shanghai"/>
                   <property name="username" value="root"/>
                   <property name="password" value="root"/>
               </dataSource>
           </environment>
       </environments>
   </configuration>
   ```

5. 在src/main/java下创建cn.aymeng.entity包，并创建实体类City

   ```java
   package cn.aymeng.entity;
   
   public class City {
       private Integer id;
       private String name;
   
       public City() {
       }
   
       public City(Integer id, String name) {
           this.id = id;
           this.name = name;
       }
   
       public Integer getId() {
           return id;
       }
   
       public void setId(Integer id) {
           this.id = id;
       }
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       @Override
       public String toString() {
           return "City{" +
                   "id=" + id +
                   ", name='" + name + '\'' +
                   '}';
       }
   }
   ```

6. 在src/main/resources目录下创建目录cn.aymeng.mapper，创建映射文件CityMapper.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="cn.aymeng.CityMapper">
       <select id="selectAll" resultType="cn.aymeng.entity.City">
           select * from city
       </select>
   </mapper>
   ```

7. 配置好mapper映射之后需要在Mybatis核心配置文件mappers元素中配置映射器，另外SQL语句可以通过日志输入，可以在settings元素中设置。配置完成的mybatis-config.xml为：

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
       <settings>
           <setting name="logImpl" value="LOG4J"/>
       </settings>
       
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"/>
               <dataSource type="POOLED">
                   <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                   <property name="url" value="jdbc:mysql://localhost:3306/simple?characterEncoding=utf8&amp;useSSL=true&amp;serverTimezone=Asia/Shanghai"/>
                   <property name="username" value="root"/>
                   <property name="password" value="root"/>
               </dataSource>
           </environment>
       </environments>
       
       <mappers>
           <mapper resource="cn/aymeng/mapper/CityMapper.xml"/>
       </mappers>
   </configuration>
   ```

8. 在src/main/resources下创建日志配置文件log4j.properties

   ```properties
   # 全局日志配置
   log4j.rootLogger=info, stdout
   # MyBatis 日志配置
   log4j.logger.cn.aymeng.mapper.CityMapper=TRACE
   # 控制台输出
   log4j.appender.stdout=org.apache.log4j.ConsoleAppender
   log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
   log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
   ```

9. 在src/test/java目录下创建包cn.aymeng，并且创建测试类MybatisTest

   ```java
   package cn.aymeng;
   
   import cn.aymeng.entity.City;
   import org.apache.ibatis.io.Resources;
   import org.apache.ibatis.session.SqlSession;
   import org.apache.ibatis.session.SqlSessionFactory;
   import org.apache.ibatis.session.SqlSessionFactoryBuilder;
   
   import java.io.IOException;
   import java.io.InputStream;
   import java.util.List;
   
   public class MybatisTest {
       public static void main(String[] args) {
           try {
               String resource = "mybatis-config.xml";
               InputStream inputStream = Resources.getResourceAsStream(resource);
               SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
               SqlSession sqlSession = sqlSessionFactory.openSession();
               List<City> cities = sqlSession.selectList("cn.aymeng.mapper.CityMapper.selectAll");
               for (City city : cities) {
                   System.out.println(city);
               }
           } catch (IOException e) {
               e.printStackTrace();
           }
       }
   }
   ```

   运行测试程序，输出结果为：

   ```javascript
   DEBUG [main] - ==>  Preparing: select * from city
   DEBUG [main] - ==> Parameters: 
   TRACE [main] - <==    Columns: id, name
   TRACE [main] - <==        Row: 1, 上海
   TRACE [main] - <==        Row: 2, 杭州
   TRACE [main] - <==        Row: 3, 北京
   DEBUG [main] - <==      Total: 3
   City{id=1, name='上海'}
   City{id=2, name='杭州'}
   City{id=3, name='北京'}
   
   Process finished with exit code 0
   ```

   tips：

   由于Java中的基本类型会有默认值，例如当某个类中有int类型的age字段，创建这个类时，age会有默认值0。当使用age属性时，它总会有值，因此在某些情况下，无法实现使age为mull，并且在动态SQL的部分，如果使用age != null进行判断，结果总会为true，因而会导致很多隐藏的问题，所以，在实体类中不要使用基本类型，使用包装类型。

### 3.Mybatis案例改进（官方推荐）

上面的方式可以正常工作，但是有些问题。上面的方式使用SqlSession通过命名空间调用Mybatis方法时，首先需要用到命名空间和方法组成的字符串来调用对应的方法如，sqlSession.selectList("cn.aymeng.mapper.CityMapper.selectAll")，当参数多于一个的时候，需要将所有的参数放到一个Map对象中，通过Map传递多个参数，使用起来不方便。而且转换类型的时候类型可能不安全，或者可能出现出错的字符串字面值。

Mybatis3.0支持使用接口来调用方法，Mybatis使用Java动态代理可以直接用过接口来调用相应的方法，不需要提供接口的实现类，更不需要在现实类中使用SqlSession以通过命名空间间接调用。另外，当有多个参数的时候，通过参数注解@Param设置参数的名字省去了手动构造Map的过程，尤其在使用Spring中使用的时候，可以配置为自动扫描所有的接口类，直接将接口注入到需要用到的地方。

使用此方式则必须定义mapper接口，且mapper映射的namespace命名空间必须为对应的接口的全限定名。

1. 在src/main/java下创建cn.aymeng.mappper包，并且创建CityMapper接口

   ```java
   package cn.aymeng.mapper;
   
   import cn.aymeng.entity.City;
   
   import java.util.List;
   
   public interface CityMapper {
       List<City> selectAll();
   }
   ```

   此时的CityMapper.xml映射文件的namespace必须为：cn.aymeng.mapper.CityMapper

2. 此时在Mybatis核心配置文件中，mapper映射的配置需要变为使用class属性

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
       <settings>
           <setting name="logImpl" value="LOG4J"/>
       </settings>
       
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"/>
               <dataSource type="POOLED">
                   <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                   <property name="url" value="jdbc:mysql://localhost:3306/simple?characterEncoding=utf8&amp;useSSL=true&amp;serverTimezone=Asia/Shanghai"/>
                   <property name="username" value="root"/>
                   <property name="password" value="root"/>
               </dataSource>
           </environment>
       </environments>
   
       <mappers>
           <mapper class="cn.aymeng.mapper.CityMapper" />
       </mappers>
   </configuration>
   ```

3. 测试代码

   ```java
   public class MybatisTest {
       public static void main(String[] args) {
           try {
               String resource = "mybatis-config.xml";
               InputStream inputStream = Resources.getResourceAsStream(resource);
               SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
               SqlSession sqlSession = sqlSessionFactory.openSession();
               CityMapper cityMapper = sqlSession.getMapper(CityMapper.class);
               List<City> cities = cityMapper.selectAll();
               for (City city : cities) {
                   System.out.println(city);
               }
           } catch (IOException e) {
               e.printStackTrace();
           }
       }
   }
   ```

   结果和方式一结果一致。


## 三、Mybatis核心配置

MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。 配置文档的顶层结构如下：

* configuration（配置）
  * properties（属性）
  * settings（设置）
  * typeAliases（类型别名）
  * typeHandlers（类型处理器）
  * objectFactory（对象工厂）
  * plugins（插件）
  * environments（环境配置）
    * environment（环境变量）
      * transactionManager（事务处理器）
      * dataSource（数据源）
  * databaseIdProvider（数据量厂商标识）
  * mappers（映射器）

### 1.properties（属性）

属性可以在外部进行配置，并可以进行动态替换。既可以在典型的Java属性文件中配置属性，也可以在properties元素的子元素中设置属性：例如Mybatis案例中配置数据库环境的参数可以在src/main/resources中新建database.properties属性文件，然后使用properties元素配置在mybatis-config配置文件中。

database.properties：

```properties
mysql_driver=com.mysql.cj.jdbc.Driver
mysql_url=jdbc:mysql://localhost:3306/simple?characterEncoding=utf8&useSSL=true&serverTimezone=Asia/Shanghai
mysql_username=root
mysql_password=root
```

属性配置文件使用properties文件中，然后使用${变量值}进行使用

mybatis-config.xml：

```xml
<properties resource="database.properties" />
<environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${mysql_driver}"/>
                <property name="url" value="${mysql_url}"/>
                <property name="username" value="${mysql_username}"/>
                <property name="password" value="${mysql_password}"/>
            </dataSource>
        </environment>
    </environments>
```

也可以在property元素中直接定义属性：

```xml
    <properties>
        <property name="username" value="root"/>
    </properties>
```

也可以在SqlSessionFactoryBuilder.build()方法中直接传入属性值：

```java
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, props);
```

tips：如果一个属性不只在一个地方进行了配置，那么，Mybatis将按照下面的顺序来加载：

* 首先读取在properties元素体内的指定的属性
* 然后根据properties元素中的resource属性读取路径下属性文件，或根据url指定路径读取，并覆盖之前读取过的同名属性
* 最后读取作为方法参数传递的属性，并覆盖之前读取过的同名属性

因此，通过方法参数传递的属性具有最高优先级，resource/url属性指定的配置次之，最低优先级的则是properties元素内的属性。

### 2.settings（设置）

这是Mybatis中极为重要的调整设置，他们会改变Mybatis的运行时行为。这里简单介绍几个常用设置，其它的参考官方文档：[settings](https://mybatis.org/mybatis-3/zh/configuration.html#settings)

|       设置名       |                           描述                           |                            有效值                            | 默认值 |
| :----------------: | :------------------------------------------------------: | :----------------------------------------------------------: | :----: |
|    cacheEnabled    | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。 |                        true \| false                         |  true  |
| lazyLoadingEnabled |                   延迟加载的全局开关。                   |                        true \| false                         | false  |
|      logImpl       |  指定 MyBatis 所用日志的具体实现，未指定时将自动查找。   | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置 |

一个配置完整的 settings 元素的示例如下：

```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

### 3.类型别名（typeAliases）

类型别名可为Java类型设置一个缩写名字。它仅用于XML配置，意在降低冗余的全限定类名书写。例如：

```xml
<typeAliases>
	<typeAlias alias="City" type="cn.aymeng.entity.City" />
</typeAliases>
```

这样配置时，City可以用在任何使用cn.aymeng.entity.City的地方。

也可以指定一个包名，Mybatis会在包名下搜索需要的Java Bean，如：

```xml
<typeAliases>
	<package name="cn.aymeng.entity"/>
</typeAliases>
```

每一个在包cn.aymeng.entity中的Java Bean，在没有注解的情况下，会使用Bean的首字母小写的非限定类名来作为它的别名。

下面是一些为常见的Java类型内建的类型别名。它们都是不区分大小写的，为了应对基本类型的命名重复，采用了特殊的命名风格。

|别名|映射的类型|别名|映射的类型|别名|映射的类型|
|:---:|:---:|:---:|:---:|:---:|:---:|
|_byte|byte|_long|long|_short|short|
|_int|int|_integer|int|_double|double|
|_float|float|_boolean|boolean|string|String|
|byte|Byte|short|Short|int|Integer|
|integer|Integer|double|Double|float|Float|
|boolean|Boolean|date|Date|decimal|BigDecimal|
|bigdecimal|BigDecimal|object|Object|map|Map|
|hashmap|HashMap|list|List|arraylist|ArrayList|
|collection|Collection|iterator|Iterator|||

### 4.类型处理器（typeHandlers）

### 5.对象工厂（objectFactory）

### 6.插件（plugins）

### 7.环境配置（environments）

Mybatis可以配置多种环境，这种机制有助于将SQL映射应用于多种数据库之中。不过，尽管可以配置多个环境，但每个SqlSessionFactory实例只能选择一种环境。如果想连接两个数据库，就需要创建两个 SqlSessionFactory 实例，每个数据库对应一个。而如果是三个数据库，就需要三个实例。为了指定创建哪种环境，只要将它作为可选的参数传递给 SqlSessionFactoryBuilder 即可。可以接受环境配置的两个方法签名是：

```java
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment);
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment, properties);
```

如果忽略了环境参数，那么将会加载默认环境。environments 元素定义了如何配置环境。

```java
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC" />
    <dataSource type="POOLED">
      <property name="driver" value="${msyql_driver}"/>
      <property name="url" value="${msyql_url}"/>
      <property name="username" value="${msyql_username}"/>
      <property name="password" value="${msyql_password}"/>
    </dataSource>
  </environment>
</environments>
```

- 默认使用的环境 ID（比如：default="development"）。
- 每个 environment 元素定义的环境 ID（比如：id="development"）。
- 事务管理器的配置（比如：type="JDBC"）。
- 数据源的配置（比如：type="POOLED"）。

默认环境和环境 ID 顾名思义。 环境可以随意命名，但务必保证默认的环境 ID 要匹配其中一个环境 ID。

1. 事务管理器（transactionManager）

   在 MyBatis 中有两种类型的事务管理器（也就是 type="[JDBC|MANAGED]"）：

   * JDBC – 这个配置直接使用了 JDBC 的提交和回滚设施，它依赖从数据源获得的连接来管理事务作用域。

   * MANAGED – 这个配置几乎没做什么。它从不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接。然而一些容器并不希望连接被关闭，因此需要将 closeConnection 属性设置为 false 来阻止默认的关闭行为。例如:

     ```java
     <transactionManager type="MANAGED">
       <property name="closeConnection" value="false"/>
     </transactionManager>
     ```

2. 数据源（dataSource）

   dataSource元素使用标准的JDBC数据源接口来配置JDBC连接对象的资源，有三种内建的数据源类型（type="[UNPOOLED|POOLED|JNDI"）

   + **UNPOOLED**– 这个数据源的实现会每次请求时打开和关闭连接。虽然有点慢，但对那些数据库连接可用性要求不高的简单应用程序来说，是一个很好的选择。 性能表现则依赖于使用的数据库，对某些数据库来说，使用连接池并不重要，这个配置就很适合这种情形。UNPOOLED 类型的数据源仅仅需要配置以下 5 种属性：

     - `driver` – 这是 JDBC 驱动的 Java 类全限定名（并不是 JDBC 驱动中可能包含的数据源类）。
     - `url` – 这是数据库的 JDBC URL 地址。
     - `username` – 登录数据库的用户名。
     - `password` – 登录数据库的密码。
     - `defaultTransactionIsolationLevel` – 默认的连接事务隔离级别。
     - `defaultNetworkTimeout` – 等待数据库操作完成的默认网络超时时间（单位：毫秒）。查看 `java.sql.Connection#setNetworkTimeout()` 的 API 文档以获取更多信息。

     作为可选项，你也可以传递属性给数据库驱动。只需在属性名加上“driver.”前缀即可，例如：

     - `driver.encoding=UTF8`

   + **POOLED**– 这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这种处理方式很流行，能使并发 Web 应用快速响应请求。除了上述提到 UNPOOLED 下的属性外，还有更多属性用来配置 POOLED 的数据源：

     - `poolMaximumActiveConnections` – 在任意时间可存在的活动（正在使用）连接数量，默认值：10
     - `poolMaximumIdleConnections` – 任意时间可能存在的空闲连接数。
     - `poolMaximumCheckoutTime` – 在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒）
     - `poolTimeToWait` – 这是一个底层设置，如果获取连接花费了相当长的时间，连接池会打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直失败且不打印日志），默认值：20000 毫秒（即 20 秒）。
     - `poolMaximumLocalBadConnectionTolerance` – 这是一个关于坏连接容忍度的底层设置， 作用于每一个尝试从缓存池获取连接的线程。 如果这个线程获取到的是一个坏的连接，那么这个数据源允许这个线程尝试重新获取一个新的连接，但是这个重新尝试的次数不应该超过 `poolMaximumIdleConnections` 与 `poolMaximumLocalBadConnectionTolerance` 之和。 默认值：3（新增于 3.4.5）
     - `poolPingQuery` – 发送到数据库的侦测查询，用来检验连接是否正常工作并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动出错时返回恰当的错误消息。
     - `poolPingEnabled` – 是否启用侦测查询。若开启，需要设置 `poolPingQuery` 属性为一个可执行的 SQL 语句（最好是一个速度非常快的 SQL 语句），默认值：false。
     - `poolPingConnectionsNotUsedFor` – 配置 poolPingQuery 的频率。可以被设置为和数据库连接超时时间一样，来避免不必要的侦测，默认值：0（即所有连接每一时刻都被侦测 — 当然仅当 poolPingEnabled 为 true 时适用）。

   + **JNDI** – 这个数据源实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的数据源引用。这种数据源配置只需要两个属性：

     - `initial_context` – 这个属性用来在 InitialContext 中寻找上下文（即，initialContext.lookup(initial_context)）。这是个可选属性，如果忽略，那么将会直接从 InitialContext 中寻找 data_source 属性。
     - `data_source` – 这是引用数据源实例位置的上下文路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。

     和其他数据源配置类似，可以通过添加前缀“env.”直接把属性传递给 InitialContext。比如：

     - `env.encoding=UTF8`

   + 也可以通过实现接口org.apache.ibatis.datasource.DataSourceFactory来使用第三方数据源实现：

     ```java
     public interface DataSourceFactory {
       void setProperties(Properties props);
       DataSource getDataSource();
     }
     ```

     `org.apache.ibatis.datasource.unpooled.UnpooledDataSourceFactory` 可被用作父类来构建新的数据源适配器，比如下面这段插入 C3P0 数据源所必需的代码：

     ```java
     import org.apache.ibatis.datasource.unpooled.UnpooledDataSourceFactory;
     import com.mchange.v2.c3p0.ComboPooledDataSource;
     
     public class C3P0DataSourceFactory extends UnpooledDataSourceFactory {
     
       public C3P0DataSourceFactory() {
         this.dataSource = new ComboPooledDataSource();
       }
     }
     ```

     为了令其工作，记得在配置文件中为每个希望 MyBatis 调用的 setter 方法增加对应的属性。 下面是一个可以连接至 PostgreSQL 数据库的例子：

     ```java
     <dataSource type="cn.aymeng.C3P0DataSourceFactory">
       <property name="driver" value="${msyql_driver}"/>
       <property name="url" value="${msyql_url}"/>
       <property name="username" value="${msyql_username}"/>
       <property name="password" value="${msyql_password}"/>
     </dataSource>
     ```

### 8.数据库厂商标识（databaseIdProvider）

### 9.映射器（mapper）

既然 MyBatis 的行为已经由上述元素配置完了，我们现在就要来定义 SQL 映射语句了。 但首先，我们需要告诉 MyBatis 到哪里去找到这些语句。 在自动查找资源方面，Java 并没有提供一个很好的解决方案，所以最好的办法是直接告诉 MyBatis 到哪里去找映射文件。 你可以使用相对于类路径的资源引用，或完全限定资源定位符（包括 `file:///` 形式的 URL），或类名和包名等。例如：

```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
```

```xml
<!-- 使用完全限定资源定位符（URL） -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>
```

```xml
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
```

```xml
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

## 四、映射器Mapper.xml配置

### **MybatisUtils**

为了方便以后测试，在src/mian/java下新建包cn.aymeng.util并且创建生成SqlSession的工具类MybatisUtils：

```java
package cn.aymeng.util;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;
    static {
        String resource = "mybatis-config.xml";
        try {
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 获取SqlSession对象
     * @return session
     */
    public static SqlSession getSession(){
        return sqlSessionFactory.openSession();
    }
}
```



SQL 映射文件只有很少的几个顶级元素（按照应被定义的顺序列出）：

- `cache` – 该命名空间的缓存配置。
- `cache-ref` – 引用其它命名空间的缓存配置。
- `resultMap` – 描述如何从数据库结果集中加载对象，是最复杂也是最强大的元素。
- `parameterMap` – 老式风格的参数映射。此元素已被废弃，并可能在将来被移除！请使用行内参数映射。文档中不会介绍此元素。
- `sql` – 可被其它语句引用的可重用语句块。
- `insert` – 映射插入语句。
- `update` – 映射更新语句。
- `delete` – 映射删除语句。
- `select` – 映射查询语句。

### 1.cache

#### 1.一级缓存

使用缓存可以使应用更快地获取数据，避免频繁的数据库交互，尤其是在查询越多，缓存命中率越高的情况下，使用缓存的作用就越明显。一般提到Mybatis缓存的时候，都是指二级缓存。一级缓存（也叫本地缓存）默认会启用，它仅仅对一个会话中的数据进行缓存。

先通过一个简单示例来看下一级缓存如何起作用，在cn.aymeng下新建测试类LocalCacheTest：

```java
public class LocalCacheTest {
    public static void main(String[] args) {
        try {
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
            SqlSession sqlSession = sqlSessionFactory.openSession();
            CityMapper cityMapper = sqlSession.getMapper(CityMapper.class);
            System.out.println("第一次查询：");
            List<City> cities01 = cityMapper.selectAll();
            for (City city : cities01) {
                System.out.println(city);
            }
            System.out.println("第二次查询：");
            List<City> cities02 = cityMapper.selectAll();
            for (City city : cities02) {
                System.out.println(city);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

运行程序结果：

```javascript
第一次查询：
DEBUG [main] - ==>  Preparing: select * from city
DEBUG [main] - ==> Parameters: 
TRACE [main] - <==    Columns: id, name
TRACE [main] - <==        Row: 1, 上海
TRACE [main] - <==        Row: 2, 杭州
TRACE [main] - <==        Row: 3, 北京
DEBUG [main] - <==      Total: 3
City{id=1, name='上海'}
City{id=2, name='杭州'}
City{id=3, name='北京'}
第二次查询：
City{id=1, name='上海'}
City{id=2, name='杭州'}
City{id=3, name='北京'}

Process finished with exit code 0
```

在第一次执行selectAll方法获取city数据时，真正执行了数据库查询，得到了cities01的结果。第二次执行获取cities02的时候便直接获得了数据，第二次查询并没有执行数据库操作。

Mybatis的一级缓存存在于SqlSession的声明周期中，在同一个SqlSession中查询时，Mybatis会把执行的方法和参数通过算法生成缓存的键值，将键值和查询结果存入一个map对象中。如果同一个SqlSession中执行的方法和参数完全一致，那么通过算法会生成相同的键值，当map缓存对象中已经存在该键值时，则会返回缓存中的对象。

如果不想在查询方法使用一级缓存，可以作出如下修改：

```xml
<select id="selectAll" flushCache="true" resultType="city">
        select * from city
</select>
```

该修改在原来的基础上增加了 flushCache="true"，这个属性配置为true后，会在查询数据前清空当前的一级缓存，因此该方法每次都会重新从数据库中查询数据。但是由于这个方法清空了一级缓存，会影响当前SqlSession中所有缓存的查询，因此在需要反复查询获取只读数据的情况下，会增加数据库的查询次数，所有避免这么使用。另外，任何的INSERT、UPDATE、DELETE操作都会清空一级缓存。

#### 2.二级缓存

Mybatis的二级缓存不同于一级缓存只存在于SqlSession的声明后期中，可以理解为存在于SqlSessionFactory的声明周期中。在Mybatis的核心配置文件<settings>中又一个参数cacheEnabled，这个参数是二级缓存的全局开关，默认值是true，初始状态为启用状态。如果这个参数设置为flase，即使有后面的二级缓存配置，也不会生效。由于这个参数的默认为true，所以不必配置。

Mybatis的二级缓存是和命名空间绑定的，即二级缓存需要配置在Mapper.xml映射文件中或者配置在Mapper.xml接口中。

* 在Mapper.xml中配置二级缓存

  在保证二级缓存的全局配置开启的情况下，要启用全局的二级缓存，只需要在SQL映射文件中添加<cache/>元素即可。例如给CityMapper.xml开启二级缓存，配置如下：

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="cn.aymeng.mapper.CityMapper">
      <cache />
      <select id="selectAll" resultType="city">
          select * from city
      </select>
  </mapper>
  ```

  二级缓存会有如下效果

  - 映射语句文件中的所有 select 语句的结果将会被缓存。
  - 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
  - 缓存会使用最近最少使用算法（LRU，Least Recently Used，最近最少使用的）算法来清除不需要的缓存。
  - 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
  - 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
  - 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。

  所有这些属性都可以通过缓存元素的属性来修改，如：

  ```xml
  <cache eviction="FIFO" flushInterval="60000" size="512" readOnly="true" />
  ```

  这个配置创建了一个FIFO缓存，并每隔60秒刷新一次，存储集合或对象的512个引用，而且返回的对象是只读的。

  cache可以配置的属性如下：

  * eviction（收回策略）
    * LRU（最近最少使用的）：移除最长时间不被使用的对象，这是默认值。
    * FIFO（先进先出）：按对象进入缓存的顺序来移除它们。
    * SOFT（软引用）：移除基于垃圾回收器状态和软引用规则的对象。
    * WEAK（弱引用）：更及基地移除基于垃圾收集器状态和弱引用规则的对象。
  * flushInterval（刷新间隔）：可以设置为任意的正整数，单位为毫秒。默认不设置
  * sieze（引用数目）：可以被设置为任意正整数，基础缓存的对象数目和运行环境的可用内存资源数目。默认值是1024.
  * readOnly（只读）：属性可以为true或flase。只读的缓存会给所有调用者返回缓存对象的相同实例，因此对象不能被修改。可读写的缓存会通过序列化返回缓存对象的拷贝，线程安全，这种方式会慢一点，但是安全，默认为flase。

* 在Mapper接口中配置二级缓存

  使用注解方式时，如果想启用二级缓存，需要在Mapper接口中进行配置，需要增加@CacheNamespace即可，例如要给CityMapper接口使用二级缓存注解，如下：

  ```java
  package cn.aymeng.mapper;
  
  import cn.aymeng.entity.City;
  import org.apache.ibatis.annotations.CacheNamespace;
  
  import java.util.List;
  @CacheNamespace
  public interface CityMapper {
      List<City> selectAll();
  }
  ```

  该注解同样可以配置各项属性，如下：

  ```java
  @CacheNamespace(eviction = FifoCache.class, flushInterval = 60000, size = 512, readWrite = true)
  ```

  这里的readWrite属性和XML中的readOnly属性一样。

  当使用注解的方式和XML映射文件同时配置了二级缓存，需要特殊配置否则就会报错，这是因为Mapper接口和对应的XML文件是相同的命名空间。想使用二级缓存，必须同时配置（如果接口不存在使用注解方式的方法，可以只在XML中配置）。这个时候应该使用参照缓存：

  * 在Mapper接口中，参照缓存配置如下

    ```java
    @CacheNamespaceRef(CityMapper.class)
    public interface CityMapper {
        List<City> selectAll();
    }
    ```

  * 在映射文件XML中，参照缓存配置如下

    ```xml
    <cache-ref namespace="cn.aymeng.mapper.CityMapper"/>
    ```

  Mybatis中很少会同时使用Mapper接口注解方式和XML映射文件，所以参照缓存并不是为了解决这个问题二设计的。参照缓存除了能够通过引用其它缓存减少配置外，主要的作用是解决脏读。

* 二级缓存的使用

  对CityMapper配置二级缓存后，当调用CityMapper所有的select查询方法时，二级缓存就已经开始起作用了。需要注意的是，由于配置的是可读写（readOnly默认为false）的缓存，而Mybatis使用SerializedCache序列化来实现可读写缓存类，并通过序列化和反序列化来保证通过缓存数据数据时，得到的是一个新的实例。如果配置为只读缓存，Mybatis就会使用Map来存储缓存值，这种情况下，从缓存中获取的对象就是同一个实例。

  因为使用可读写缓存，可以使用SerializedCache序列化缓存。这个缓存类要求所有被序列化的对象必须实现Serializable（java.io.Serializable）接口，所以需要修改City（实体类）对象，如下：

  ```java
  package cn.aymeng.entity;
  
  import java.io.Serializable;
  
  public class City implements Serializable {
      private Integer id;
      private String name;
      //setter,getter方法...
      //toString方法...
  }
  ```

  在src/test/java的cn.aymeng包下创建levelTwoCacheTest类：

  ```java
  package cn.aymeng;
  
  import cn.aymeng.entity.City;
  import cn.aymeng.mapper.CityMapper;
  import cn.aymeng.util.MybatisUtils;
  import org.apache.ibatis.session.SqlSession;
  
  import java.util.List;
  
  public class LevelTwoCacheTest {
      public static void main(String[] args) {
          // 第一次查询
          System.out.println("第一次查询：");
          SqlSession session01 = MybatisUtils.getSession();
          CityMapper cityMapper01 = session01.getMapper(CityMapper.class);
          List<City> cities01 = cityMapper01.selectAll();
          for (City city : cities01)
              System.out.println(city);
          session01.close();
          System.out.println("第二次查询：");
          SqlSession session02 = MybatisUtils.getSession();
          CityMapper cityMapper02 = session02.getMapper(CityMapper.class);
          List<City> cities02 = cityMapper02.selectAll();
          for (City city : cities02)
              System.out.println(city);
          session02.close();
      }
  }
  ```

  执行上面的测试代码，输出结果为：

  ```javascript
  第一次查询：
  DEBUG [main] - Cache Hit Ratio [cn.aymeng.mapper.CityMapper]: 0.0
  DEBUG [main] - ==>  Preparing: select * from city
  DEBUG [main] - ==> Parameters: 
  TRACE [main] - <==    Columns: id, name
  TRACE [main] - <==        Row: 1, 上海
  TRACE [main] - <==        Row: 2, 杭州
  TRACE [main] - <==        Row: 3, 北京
  DEBUG [main] - <==      Total: 3
  City{id=1, name='上海'}
  City{id=2, name='杭州'}
  City{id=3, name='北京'}
  第二次查询：
  DEBUG [main] - Cache Hit Ratio [cn.aymeng.mapper.CityMapper]: 0.5
  City{id=1, name='上海'}
  City{id=2, name='杭州'}
  City{id=3, name='北京'}
  
  Process finished with exit code 0
  
  ```

  Cache Hit Ratio是当前执行方法的缓存命中率。在第一次查询获取cities01的时候由于没有缓存，所以执行了数据库查询，当调用close方法关闭SqlSession时，SqlSession才会保存查询数据到二级缓存中，在这之后二级缓存才有了缓存数据。所以第一次查询的时候，命中率为0。第二次查询时，并没有执行数据库查询，而是直接命中缓存，获得了缓存的值。

  Mybatis默认提供的缓存实现是基于Map实现的内存缓存，已经基本满足应用的使用。但是当需要缓存大量的数据时，不能仅仅提高内存来使用Mybatis的二级缓存，还可以选择一些类似EhCache的缓存框架或Redis缓存数据库等工具来保存Mybatis的二级缓存数据。

#### 3.脏数据的产生和避免

二级缓存虽然能提高应用效率，减轻数据库服务器的压力，但是如果使用不当，很容易产生脏数据。Mybatis的二级缓存是和命名空间绑定的，所以通常情况下每一个Mapper映射文件都拥有自己的二级缓存，不同Mapper的二级缓存互不影响。在常见的数据库操作中，多表联合查询非常常见，由于关系型数据库的设计，使得很多时候需要关联多个表才能获得想要的数据。在关联多表查询时肯定会将改查询放到某个命名空间下的映射文件中，这样一个多表的查询就会缓存在改命名空间的二级缓存中。涉及这些表的增、删、改操作通常不在一个映射文件中，它们的命名空间不同，因此当有数据变化时，多表查询的缓存未必会被清空，这种情况下就会产生脏数据。

使用参照缓存可以避免脏数据的出现。当某几个表可以作为一个业务整体时，通常是让几个会关联的ER表同时使用同一个二级缓存，这样就能解决脏数据问题。虽然这样可以解决脏数据的问题，但是并不是所有的关联查询都可以这么解决，如果有几十个甚至所有表都以不同的关联关系存在于各自的映射文件中时，使用参照缓存显然没有意义。

二级缓存虽然好处很多，但并不是什么时候都可以使用。在以下场景中，推荐使用二级缓存。

* 以查询为主的应用中，只有尽可能少的增、删、改操作。
* 绝大多数以单表操作存在时，由于很少存在互相关联的情况，因此不会出现脏数据。
* 可以按业务划分对表进行分组时，如关联的表比较少，可以通过参照缓存进行配置。

除了推荐使用的情况，如果脏读对系统没有影响，也可以考虑使用。在无法保证数据不出现脏读的情况下，建议在业务层使用可控制的缓存代替二级缓存。

### 2.select

查询语句是Mybatis中最常用的元素之一，下面是个简单的查询select：

```xml
<select id="selectCity" parameterType="int" resultType="hashmap">
	SELECT * FROM city WHERE id = #{id}
</select>
```

这个语句为selectCity，接受一个int（或者Integer）类型的参数，并返回一个HashMap的对象，其中的键是列名，值便是结果行中的对应值。这就告诉Mybatis创建一个预处理语句（PreparedStatement）参数，在SQL语句中，这样的一个参数在SQL中会由一个?来标识，并传递一个新的预处理语句。

XML中的select标签的id属性值和定义的接口方法是一样的。Mybatis就是通过这种方式将接口方法和XML中定义的SQL语句关联到一起的，如果接口方法没有和XML中的id属性相对应，程序就会报错。映射和接口的命名需要符合如下规则。

* 当只使用XML而不使用接口的时候，namespace的值可以设置为任意不重复的名称。
* 标签的id属性值在任何时候都不能出现英文“.”，并且同一个命名空间下不能出现重复的id。
* 因为接口方法可以重载，所以接口可以出现多个同名但参数不同的方法，但是XML中的id值不能重复，因而接口中的所有同名方法会对应着XML中的同一个id的方法。最常见的是，同名方法中其中一个方法增加一个RowBound类型的参数用于实现分页查询。

select有很多属性配置可以用来配置每条语句的行为细节：

|       属性       |                             描述                             |
| :--------------: | :----------------------------------------------------------: |
|        id        |      在命名空间中唯一的标识符，可以被用来引用这条语句。      |
|  parameterType   | 将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器（TypeHandler）推断出具体传入语句的参数，默认值为未设置（unset）。 |
| ~~parameterMap~~ | ~~用于引用外部 parameterMap 的属性，目前已被废弃。请使用行内参数映射和 parameterType 属性。~~ |
|    resultType    | 期望从这条语句中返回结果的类全限定名或别名。 注意，如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身的类型。 resultType 和 resultMap 之间只能同时使用一个。 |
|    resultMap     | 对外部 resultMap 的命名引用。结果映射是 MyBatis 最强大的特性，如果你对其理解透彻，许多复杂的映射问题都能迎刃而解。 resultType 和 resultMap 之间只能同时使用一个。 |
|    flushCache    | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：false。 |
|     useCache     | 将其设置为 true 后，将会导致本条语句的结果被二级缓存缓存起来，默认值：对 select 元素为 true。 |
|     timeout      | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。 |
|    fetchSize     | 这是一个给驱动的建议值，尝试让驱动程序每次批量返回的结果行数等于这个设置值。 默认值为未设置（unset）（依赖驱动）。 |
|  statementType   | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
|  resultSetType   | FORWARD_ONLY，SCROLL_SENSITIVE, SCROLL_INSENSITIVE 或 DEFAULT（等价于 unset） 中的一个，默认值为 unset （依赖数据库驱动）。 |
|    databaseId    | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。 |
|  resultOrdered   | 这个设置仅针对嵌套结果 select 语句：如果为 true，将会假设包含了嵌套结果集或是分组，当返回一个主结果行时，就不会产生对前面结果集的引用。 这就使得在获取嵌套结果集的时候不至于内存不够用。默认值：`false`。 |
|    resultSets    | 这个设置仅适用于多结果集的情况。它将列出语句执行后返回的结果集并赋予每个结果集一个名称，多个名称之间以逗号分隔。 |

### 3.insert，update和delete

insert，update和delete语句非常相近，分别如下：

insert语句：

```xml
<insert id="insertCity">
    insert into city values (#{id},#{name})
</insert>
```

update语句：

```xml
<update id="updateCity">
    update city set
    name = #{name}
    where id = #{id}
</update>
```

delete语句：

```xml
<delete id="deleteCity">
    delete from city where id = #{id}
</delete>
```

insert，update和delete元素的属性：

|       属性       |                             描述                             |
| :--------------: | :----------------------------------------------------------: |
|        id        |      在命名空间中唯一的标识符，可以被用来引用这条语句。      |
|  parameterType   | 将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器（TypeHandler）推断出具体传入语句的参数，默认值为未设置（unset）。 |
| ~~parameterMap~~ | ~~用于引用外部 parameterMap 的属性，目前已被废弃。请使用行内参数映射和 parameterType 属性。~~ |
|    flushCache    | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：（对 insert、update 和 delete 语句）true。 |
|     timeout      | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。 |
|  statementType   | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| useGeneratedKeys | （仅适用于 insert 和 update）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系型数据库管理系统的自动递增字段），默认值：false。 |
|   keyProperty    | （仅适用于 insert 和 update）指定能够唯一识别对象的属性，MyBatis 会使用 getGeneratedKeys 的返回值或 insert 语句的 selectKey 子元素设置它的值，默认值：未设置（`unset`）。如果生成列不止一个，可以用逗号分隔多个属性名称。 |
|    keyColumn     | （仅适用于 insert 和 update）设置生成键值在表中的列名，在某些数据库（像 PostgreSQL）中，当主键列不是表中的第一列的时候，是必须设置的。如果生成列不止一个，可以用逗号分隔多个属性名称。 |
|    databaseId    | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。 |

以上元素useGeneratedKeys，keyProperty和keyColumn属性是用来设置数据库主键的。

* 使用JDBC方式返回主键自增的值

  在使用主键自增（如MySQL，SQL Server数据库时）可以使用JDBC方式返回主键自增的值。useGeneratedKeys设置为true后，Mybatis会使用JDBC的getGeneratedKeys方法来取出由数据库内部生成的主键，获得主键值后将其赋值给keyProperty配置的id属性。当需要设置多个属性时，使用逗号隔开。由于要使用数据库返回的主键值，所以SQL上下两部分的列中去掉id列和#{id}属性，如下：

  ```xml
  <insert id="insertCity" useGeneratedKeys="true" keyProperty="id">
      insert into city(name) values (#{name})
  </insert>
  ```

  上面这种方式只适用于支持主键自增的数据库。有些数据库（如Oracle）不提供主键自增的功能，而是使用序列得到一个值，然后将这个值赋值给id，再将数据插入数据库。对于这种情况，需要采用另外一种方式：使用<selectKey>标签来获取主键的值，这种情况不仅适用于不提供主键自增功能的数据库，也适用于主键自增功能的数据库。

* 使用selectKey返回主键的值

  例：

  ```xml
  <insert id="insertCity">
      insert into city(name) values (#{name})
      <selectKey keyColumn="id" resultType="int" keyProperty="id" order="AFTER">
          SELECT LAST_INSERT_ID()
      </selectKey>
  </insert>
  ```

  selectKey标签的keyColumn、keyProperty和上面的useGeneratedKeys的用法含义相同，这里的resultType用于设置返回值类型。order属性的设置可数据库有关。在MySQL数据库中，order属性值是AFTER，因为当前记录的主键值是在insert语句执行成功后才能获得到。而在Oracle数据库中，order的值要设置为BEFORE，这是因为Oracle中需要先从序列获取值，然后将值作为主键插入到数据库中。selectKey元素放置的位置和SQL语句并无影响，影响执行顺序的是order属性。

  Oracle方式的insert语句中需要明确写出id列和值#{id}，因为执行selectKey中的语句后id就有值了，需要把这个序列值作为主键值插入到数据库中，所以必须指定id列，如果不指定id列，数据库就会因为主键不能为空而抛出异常。如下：

  ```xml
  <insert id="insertCity">
      <selectKey keyColumn="id" resultType="int" keyProperty="id" order="AFTER">
          SELECT SEQ_ID.nextval from city
      </selectKey>
      insert into city(id,name) values (#{id},#{name})
  </insert>
  ```

  SELECT SEQ_ID.nextval from city是Oracle数据库获取序列的SQL语句。

### 4.sql片段

<sql>元素可以用来定义重用的SQL代码片段，可以在其他语句中使用<include>标签引用。如下：

```xml
<sql id="cityColumns" >
    id,name
</sql>
<select id="selectAll" resultType="city">
    select 
    <include refid="cityColumns"></include>
    from city
</select>
```

并且可以在不同的include元素中定义不同的参数值。如下：

```xml
<sql id="cityColumns" >
    id,name
</sql>
<sql id="someTable">
    ${table}
</sql>
<select id="selectAll" resultType="city">
    select
    <include refid="cityColumns"></include>
    from 
    <include refid="someTable">
        <property name="table" value="city"/>
    </include>
</select>
```

### 5.参数

* 参数为一个的时候

  之前见到的所有语句中的参数只有一个，参数的类型可以分为两种，一种是基本类型，另一种是javaBean，对于大多数简单的使用场景，并不需要使用复杂的参数，如下：

  ```xml
  <select id="selectOne" resultType="cn.aymeng.entity.City">
    select id, name, capital
    from city
    where id = #{id}
  </select>
  ```

  上面的这个示例说明了一个非常简单的命名参数映射。鉴于参数类型（parameterType）会被自动设置为 `int`，这个参数可以随意命名。原始类型或简单数据类型（比如 `Integer` 和 `String`）因为没有其它属性，会用它们的值来作为参数。

* 当参数为多个的时候

  当参数为多个的时候，再使用上面的方式用#{参数名称}来匹配参数，程序则会报错，如下：

  ```xml
  <insert id="insertOne" >
      insert into city(id, name) values (#{id}, #{name})
  </insert>
  ```

  使用如上方法运行程序，则会报以下错误：

  ```javascript
  ### Error updating database.  Cause: org.apache.ibatis.binding.BindingException: Parameter 'name' not found. Available parameters are [arg1, arg0, param1, param2]
  ```

  这个错误表示，XML中可用的参数只有arg0、arg1、param1、param2，没有name。这些参数名称都是Mybatis根据参数自定义的名字如果将参数#{id}，#{name}换成#{arg0}，#{arg1}或者#{param1}，#{param2}运行程序，就可以正常运行。

  如果不想使用param1、param2..... 则可以使用javaBean对象，在Mapper接口中直接传入javaBean对象。如果上面的方式直接以City类型的参数传递到了语句中，Mybatis会查找id，name属性，然后将它们的值传入预处理语句的参数中。如下直接在Mapper接口中直接传如类：

  ```java
  public interface CityMapper {
      int insertOne(City city);
  }
  ```

  上述方法用起来很方便，但是并不适合全部的情况，因为不能为了两三个参数就去创建新的 java类。对于参数比较少的情况，还有两种方式可以采用：使用map类型作为参数或使用@Param注解。使用map类型不建议使用，所以最常用的方法还是在接口中使用@Param注解，如下：

  ```java
  public interface CityMapper {
      int insertOne(@Param("id") String id, @Param("name") String name);
  }
  ```

  给参数配置@Param注解后，Mybatis就会自动将参数封装成map类型，@Param注解值会作为map中的key。当参数类型是java类的时候，用法略有不同，代码如下：

  ```java
  public interface CityMapper {
      int insertOne(@Param("city") City city, String capital);
  }
  ```

  此时，mapper配置XML中就不能直接使用#{id}和#{name}了，而是通过对象方式使用${city.id}和${city.name}从JavaBean中取出指定属性的值。代码如下：

  ```xml
  <insert id="insertOne" >
      insert into city(id, name) values (${city,id}, ${city.name})
  </insert>
  ```

### 6.字符串替换

默认情况下，使用 `#{}` 参数语法时，MyBatis 会创建 `PreparedStatement` 参数占位符，并通过占位符安全地设置参数（就像使用 ? 一样）。 这样做更安全，更迅速，通常也是首选做法，不过有时你就是想直接在 SQL 语句中直接插入一个不转义的字符串。 比如 SELECT子句，这时候你可以：

```xml
<select id="selectAll" resultType="cn.aymeng.entity.City">
    select id, name from city where id = ${id}
</select>
```

其中 `${id}` 会被直接替换， 这样，就能完成同样的任务。

tips：

用这种方式接受用户的输入，并用作语句参数是不安全的，会导致潜在的 SQL 注入攻击。因此，要么不允许用户输入这些字段，要么自行转义并检验这些参数。

### 7.结果映射（resultMap）

resultMap元素是Mybatis中最重要最强大的元素，用于配置Java对象的属性和查询结果列的对应关系，通过resultMap中配置的column和property可以将查询列的映射到type对象的属性上。resultMap包含的属性如下：

* id：必填，并且唯一。在select中，resultMap指定的值即为此处id所设置的值。
* type：必填，用于配置查询列所映射到的Java对象类型。
* extends：选填，可以配置当前的resultMap继承其他的resultMap，属性值为继承resultMap的id。
* autoMapping：选填，可选值为true或flase，用于配置是否启用非映射字段（没有在resultMap中配置的字段）的自动映射功能。

以上是resultMap的属性，resultMap包含的标签如下：

* constructor：配置使用构造方法注入结果。
* id：一个id结果，标记结果为id（唯一值）可以帮助提高整体性能。
* result：注入到字段或JavaBean属性的普通结果。
* association：一个复杂的关联对象，许多结果将宝成这种类型。
  * 嵌套结果映射 – 关联可以是 `resultMap` 元素，或是对其它结果映射的引用
* collection：一个复杂类型的集合。
  * 嵌套结果映射 – 关联可以是 `resultMap` 元素，或是对其它结果映射的引用
* discriminator：根据结果值来决定使用哪个结果映射。

目前暂时先介绍constructor、id、result。

* constructor通过构造方法注入属性的结果值。构造方法中的idArg、arg参数分别对应着resultMap中的id，result标签，他们的含义相同，只是注入方式不同。
* resultMap中的id和result标签包含的属性相同，不同的地方在于,id代表的是主键（或者唯一值）的字段（可以有多个），他们的属性值是通过setter方法注入的。result是普通字段。id和result包含的属性：
  * column：从数据库得到的列名，或者是列的别名。
  * property：映射到列结果的属性。可以映射到Java类的属性，也可以映射到一些复杂对象中的属性，例如"city.name"，这会通过"."方式的属性嵌套赋值。
  * javaType：一个java类的完全限定名，或者是一个类型别名。
  * jdbcType：列对应的数据库类型。
  * typeHandler：使用这个属性可以覆盖默认的类型处理器。这个属性值是类的完全限定名或类型别名。

例：

```xml
<resultMap id="city" type="cn.aymeng.entity.City">
    <id property="id" column="id"/>
    <result property="name" column="name"/>
</resultMap>
```

或者：

```xml
<resultMap id="city" type="cn.aymeng.entity.City">
    <constructor>
        <idArg name="id" column="id"/>
        <arg name="name" column="name"/>
    </constructor>
</resultMap>
```

## 五、动态SQL