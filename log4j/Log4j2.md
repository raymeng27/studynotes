# Log4j2

1. 导入pom依赖

   ```xml
   <!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core -->
   <dependency>
       <groupId>org.apache.logging.log4j</groupId>
       <artifactId>log4j-core</artifactId>
       <version>2.13.3</version>
   </dependency>
   <!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-api -->
   <dependency>
       <groupId>org.apache.logging.log4j</groupId>
       <artifactId>log4j-api</artifactId>
       <version>2.13.3</version>
   </dependency>
   ```

2. 获取Logger对象

   ```java
   // private static final Logger logger = LogManager.getLogger();
   private static final Logger logger = LogManager.getLogger(Xxx.class);
   ```

# 一、介绍


> 官方网站：https://logging.apache.org/log4j/2.x/
>
> 经验表明，日志记录是开发周期的重要组成部分。它提供几个优点。它提供有关应用程序运行的准确上下文。一旦插入代码，日志输出的生成不需要人为干涉。而且，日志输出可以保存在持久性媒介中以便以后研究。除了在开发周期中使用之外，还可以将足够丰富的日志包视为审计工具。

Log4j已被广泛采用于许多应用中。但是在2015年8月已经停止维护（End of Life）。它的代替方案SLF4J/Logback对框架进行了许多必要的改进。使用Log4j2的原因有下：

1. Log4j 1.x和Logback都会在重新配置时丢失事件。Log4j 2不会。在Log4j中，Appender中的异常永远不会对应用程序可见。在Log4j中，可以将Appender配置为允许异常渗透到应用程序。
2. 在多线程方案中，异步记录器的吞吐量比Log4j 1.x和Logback高10倍，延迟低几个数量级。
3. Log4j 2对于独立应用程序是无垃圾的，对于Web应用程序来说会产生少量垃圾。
4. Log4j 2使用插件系统，可以非常轻松的拓展框架。
5. 支持自定义日志级别。
6. 支持lambda表达式。
7. Log4j 1.x支持Appender上的过滤器。

# 二、架构

使用log4j2 API的应用程序将从LogManager请求具有特定名称的logger。LogManager将找到相应的LoggerContext，然后从中获取Logger。LoggerConfig对象是从配置中的Logger生命创建的。LoggerConfig与实际传递LogEvents的Appender相关联。

## 1.Logger层次结构

任何日志API优于普通System.out.println的首要优势在于它能够禁用某些日志语句，同时允许，其日志输出不受阻碍地打印。此功能假定日志记录空间（即所有可能的日志记录语句的空间）根据开发人员选择的某些条件进行分类。在Log4j 1.x中，Logger层次结构通过Loggers之间的关系进行维护。 在Log4j 2中，这种关系不再存在。 而是在LoggerConfig对象之间的关系中维护层次结构。Logger和LoggerConfigs是命名实体。 记录器名称区分大小写，它们遵循分层命名规则：

## 2.命名层次结构

如果LoggerConfig的名称后跟一个点事后代logger的前缀，则称其为另一个LoggerConfig的祖先。如果LoggerConfig本身与后代LoggerConfig之间没有祖先，则称其为LoggerConfig是子LoggerConfig的父节点。

例如，名为"com.foo"的LoggerConfig是名为"com.foo.Bar"的LoggerConfig的父级。根LoggerConfig位于LoggerConfig层次结构的顶部。它的特殊之处在于它始终存在，并且它是每个层次结构的一部分。可以如下获得直接链接到根LoggerConfig的logger：

Logger logger = LogMananger.getLogger(LogManager.ROOT+LOGGER_NAME);

或者

Logger logger = LogManager.getRootLogger();

## 3.LoggerContext

LoggerContext充当Logging系统的锚点。但是，根据具体情况，可以在应用程序中具有多个LoggerContexts。

## 4.Configuration

每个LoggerContext都有一个有效的Configuration。Configuration包含所有Appenders、上下文范围的Filters、LoggerConfigs，并包含对StrSubstitutor的引用。在重新配置期间，将存在两个配置对象。一旦所有日志记录器都被重定向到新的配置，旧的配置将被停止并丢弃。

## 5.Logger

Loggers是通过调用LogManager.getLogger创建的。Logger本身不执行任何直接操作。它只是一个名称，并与LoggerConfig相关联。它拓展了AbstractLogger并实现了所需的方法。随着配置的修改，Logger可能会与不同的LoggerConfig相关联，从而导致其行为被修改。调用LoggerManager.getLogger方法传入相同的名称将始终返回对完全相同的Logger对象的引用。

Log4j可以轻松地按软件组件命名Loggers。 这可以通过在每个类中实例化Logger来实现，其中Logger名称等于类的完全限定名称。 这是定义Logger的有用且直接的方法。 由于日志输出带有生成Logger的名称，因此该命名策略可以轻松识别日志消息的来源。 但是，尽管很常见，这只是命名记录器的一种可能的策略。 Log4j不限制各种可能的命名loggers方式。 开发人员可以根据需要自由命名记录器。

由于以Logger所属类的名称命名Logger是一种非常常见的习惯用法，因此提供了方便的方法LogManager.getLogger()来自动使用调用类的完全限定类名作为Logger名称。

## 6.LoggerConfig

在日志记录配置中声明Logger时，将创建LoggerConfig对象。LoggerConfig包含一组Filters，它们必须允许LogEvent在传递给任何Appender之前先通过Filters。它包含对应该一组用于处理时间的Appender的引用。

实例1：

| logger名 | 分配的LoggerConfig | LoggerConfig 级别 | Logger 级别 |
| -------- | ------------------ | ----------------- | ----------- |
| root     | root               | DEBUG             | DEBUG       |
| X        | root               | DEBUG             | DEBUG       |
| X.Y      | root               | DEBUG             | DEBUG       |
| X.Y.Z    | root               | DEBUG             | DEBUG       |

在上面的示例1中，仅配置了根记录器并具有日志级别。 所有其他Logger引用根LoggerConfig并使用其Level。

实例2：

| logger名 | 分配的LoggerConfig | LoggerConfig 级别 | Logger 级别 |
| -------- | ------------------ | ----------------- | ----------- |
| root     | root               | DEBUG             | DEBUG       |
| X        | X                  | ERROR             | ERROR       |
| X.Y      | X.Y                | INFO              | INFO        |
| X.Y.Z    | X.Y.Z              | WARN              | WARN        |

在示例2中，所有记录器都具有已配置的LoggerConfig并从中获取其日志级别。

示例3:

| logger名 | 分配的LoggerConfig | LoggerConfig 级别 | Logger 级别 |
| -------- | ------------------ | ----------------- | ----------- |
| root     | root               | DEBUG             | DEBUG       |
| X        | X                  | ERROR             | ERROR       |
| X.Y      | X                  | ERROR             | ERROR       |
| X.Y.Z    | X.Y.Z              | WARN              | WARN        |

在示例3中，记录器root，X和X.Y.Z均具有已配置的具有相同名称的LoggerConfig。Logger X.Y没有配置的具有匹配名称的LoggerConfig，因此使用LoggerConfig X的配置，LoggerConfig会按照其名称与Logger名称的开头最长匹配。

示例4:

| logger名 | 分配的LoggerConfig | LoggerConfig 级别 | Logger 级别 |
| -------- | ------------------ | ----------------- | ----------- |
| root     | root               | DEBUG             | DEBUG       |
| X        | X                  | ERROR             | ERROR       |
| X.Y      | X                  | ERROR             | ERROR       |
| X.Y.Z    | X                  | ERROR             | ERROR       |

在示例4中，root和X 都有自己配置的同名LoggerConfig。 记录器X.Y和X.Y.Z没有配置LoggerConfigs，因此从分配给它们的LoggerConfig获取它们的日志级别，LoggerConfig按照名称与Logger名称开头的进行匹配。

示例5:

| logger名 | 分配的LoggerConfig | LoggerConfig 级别 | Logger 级别 |
| -------- | ------------------ | ----------------- | ----------- |
| root     | root               | DEBUG             | DEBUG       |
| X        | X                  | ERROR             | ERROR       |
| X.Y      | X.Y                | INFO              | INFO        |
| X.YZ     | X                  | ERROR             | ERROR       |

在示例5中，记录器root,X和X.Y都有自己配置的同名LoggerConfig。日志记录器X.YZ 没有配置LoggerConfig，因此从分配给它的LoggerConfig X获取其日志级别，LoggerConfig按照名称与Logger名称开头的进行匹配。它不与LoggerConfig X.Y关联，因为句点之后的标记必须精确匹配。

示例6:

| logger名 | 分配的LoggerConfig | LoggerConfig 级别 | Logger 级别 |
| -------- | ------------------ | ----------------- | ----------- |
| root     | root               | DEBUG             | DEBUG       |
| X        | X                  | ERROR             | ERROR       |
| X.Y      | X.Y                |                   | ERROR       |
| X.Y.Z    | X.Y                |                   | ERROR       |

在示例6中，LoggerConfig X.Y它没有配置级别，因此它从LoggerConfig X继承其级别.Logger X.Y.Z使用LoggerConfig X.Y，因为它没有名称完全匹配的LoggerConfig。 它也从LoggerConfig X继承了它的日志级别。

## 7.Filter

Log4j提供Filter可以应用在把控制传递给任何LoggerConfig之前，在控制传递给LoggerConfig之后但在调用任何Appender之前，在控制传递到一个LoggerConfig之后，但在调用特定的Appender之前，以及每个Appender。以一种与防火墙过滤器非常相似的方式，每个过滤器可以返回三个结果中的一个，及Accept（接受）、Deny(拒绝)或Neutral（中立）。Accept的响应意味着不应该调用其它Filter，事件应该被处理。Deny的响应意味着应该立即忽略事件，并将控制权返回给调用方。Netrual的响应表示事件应该传递给其它的Filter。如果没有其他的Filter，事件将被处理。

## 8.Appender

根据logger，程序有选择地启用或禁用日志记录请求只是Log4j能力其中的一部分。Log4j允许将日志请求打印到多个目的地。在log4j中，输出目的地称为Appender。目前，存在用于控制台、文件、远程套接字服务器、Apache Flume、JMS、远程UNIX Syslog守护进程和各种数据库api的Appender。Logger可以绑定多个Appender。

可以通过调用当前Configuration 的addLoggerAppender方法将Appender添加到Logger。如果不存在与Logger名称匹配的LoggerConfig，将创建一个并将Appender与它绑定，然后将通知所有Logger更新其LoggerConfig引用。

**对于给定的logger,每个启用的日志记录请求都将转发给该Logger的LoggerConfig中的所有appender以及LoggerConfig父项的Apperders。**换句话说，Appender是从LoggerConfig结构中附加的继承的。例如，如果将控制台appender添加到root logger,则所有启用的日志记录请求将至少在控制台上打印。如果另外将文件appender添加到LoggerConfig，例如C，则对C和C的子节点启用的日志记录请求将打印在文件和控制台上。 通过在配置文件中的Logger声明上设置additivity =“false”，可以覆盖此默认行为，以便Appender累积不再是累加的。

* Appender可加性

  Logger L的日志语句的输出将转到与L关联的LoggerConfig中的所有Appender以及该LoggerConfig的祖先。 这就是术语“appender additivity”的含义。

  但是，如果与Logger L关联的LoggerConfig的祖先（比如P）将additivity标志设置为false，那么L 输出将被定向到L 的LoggerConfig中的所有appender(包含P但不包含P的祖先Appender)。

  Logger 默认情况下将其可加性标志设置为true。

* Layout

  通常，用户不仅要定制输出目的地，还要定制输出格式。这是通过将Layout与Appender相关联来实现的。Layout服装根据用户的意愿格式化LogEvent，而appender负责发送格式化的输出到目的地。PattrenLayout是标准log4j发行版的一部分，它允许用户根据类似于C语言printf的转换模式指定输出格式功能。

  例如，具有转换模式“％r [％t]％-5p％c - ％m％n”的PatternLayout将输出类似于:
   176 [main] INFO org.foo.Bar - Located nearest gas station.

  第一个字段是自程序启动以来经过的毫秒数。 第二个字段是发出日志请求的线程。 第三个字段是日志语句的级别。 第四字段是与日志请求关联的记录器的名称。 ' - '后面的文字是日志打印的内容。

  Log4j为各种不同的情况提供了许多不同的Layout，例如JSON，XML，HTML和Syslog。

# 三、API

> Log4j2 API提供应用程序应该编写的接口，并提供开发人员创建日志记录实现所需的适配器组件。尽管Log4j在API和实现之间被分开，但这样做的主要目的不是允许多个实现，尽管这当然是可以这样做，但要明确定义哪些类和方法可以安全地在普通应用程序代码中使用。

获取Logger，例：

```java
public class HelloWorld {
    private static final Logger logger = LogManager.getLogger(HelloWorld.class);
    public static void main(String[] args){
        logger.info("Hello World!");
    }
}
```

## 1.替换参数

日志记录的目的通常是提供有关系统中发生的事情的信息，这需要日志输出中包含有关被操纵对象的信息。Log4j 1.x的方式如下：

```java
if(logger.isDebugEnabled){
    logger.debug("Logger in user " + user.getName() + " with birthday " + user.getBirthday());
}
```

在2.x中你只需要：

```java
logger.debug("Logger in user {} with birthday {}", user.getName(), user.getBirthday());
```

## 2.格式化参数

如果toString()不是您想要的格式，Formatter Loggers可以按照你需要的方式格式化。

```java
public static Logger logger = LogManager.getFormatterLogger("Foo");
 
logger.debug("Logging in user %s with birthday %s", user.getName(), user.getBirthdayCalendar());
logger.debug("Logging in user %1$s with birthday %2$tm %2$te,%2$tY", user.getName(), user.getBirthdayCalendar());
logger.debug("Integer.MAX_VALUE = %,d", Integer.MAX_VALUE);
logger.debug("Long.MAX_VALUE = %,d", Long.MAX_VALUE);
```

要使用logger的formatter，必须调用其中一个LogManageer的getFormatterLogger方法。输出如下：

```javascript
2012-12-12 11:56:19,633 [main] DEBUG: User John Smith with birthday java.util.GregorianCale
2012-12-12 11:56:19,643 [main] DEBUG: User John Smith with birthday 05 23, 1995
2012-12-12 11:56:19,643 [main] DEBUG: Integer.MAX_VALUE = 2,147,483,647
2012-12-12 11:56:19,643 [main] DEBUG: Long.MAX_VALUE = 9,223,372,036,854,775,807
```

## 3.将Logger与格式化Logger混合

Formatter记录器对输出格式进行细粒度控制，但缺点是必须指正正确的类型（例如，为%d格式参数控制传递十进制整数以外的任何内容会产生异常）

如果主要使用的是使用{}参数，但偶尔需要进行细粒度控制输出格式，可以使用printf方法：

```java
public static Logger logger = LogManager.getLogger("Foo");
 
logger.debug("Opening connection to {}...", someDataSource);
logger.printf(Level.INFO, "Logging in user %1$s with birthday %2$tm %2$te,%2$tY", user.getName(), user.getBirthdayCalendar());
```

## 4.支持Java 8 lambda

在2.4版中，Logger接口添加了对lambda表达式的支持。这允许客户端代码简介的编写日志输出代码，而无需显示检查是否启用了请求的日志级别。例如，以前会写：

```java
// pre-Java 8 style optimization: 显式检查日志级别
// 确保仅在必要时调用expensiveOperation（）方法
if (logger.isTraceEnabled()) {
logger.trace("Some long-running operation returned {}", expensiveOperation());
}
```

使用Java 8，可以使用lambda表达式实现相同的效果。如下：

```java
// Java-8 style optimization: 不需要显示的检查日志级别
// 如果未启用TRACE级别，则不执行lambda表达式
logger.trace("Some long-running operation returned {}", () -> expensiveOperation());
```

## 5.Logger名称

大多数日志记录实现使用分层方案将Logger名称与日志记录配置进行匹配 在此方案中，记录器名称层次结构由记录器名称中的“.”字符表示，其方式与用于Java包名称的层次结构非常相似。例如，org.apache.logging.appender和org.apache.logging.filter都将org.apache.logging作为其父级。 在大多数情况下，应用程序通过将当前类的名称传递给LogManager.getLogger（...）来命名其记录器。 因为这种用法非常常见，所以当logger name参数被省略或为null时，Log4j 2将当前类名作为默认值。 例如，在下面的所有示例中，Logger的名称为“org.apache.test.MyTest”

```java
public class MyTest {
    private static final Logger logger = LogManager.getLogger(MyTest.class.getName());
}
```

```java
public class MyTest {
    private static final Logger logger = LogManager.getLogger(MyTest.class);
}
```

```java
public class MyTest {
    private static final Logger logger = LogManager.getLogger();
}
```

# 四、Configuration

Log4j 2的配置可以通过一下四种方式之一完成：

1. 通过以XML、JSON、YAML或properties格式编写的配置文件。
2. 以编程方式，通过创建ConfigurationFactory和Configuration实现。
3. 以编程方式，通过调用Configuration结构中暴露的API将组件添加到默认配置。
4. 以编程方式，通过调用内部Logger类上的方法。

以下内容侧重于通过配置文件配置Log4j。

## 1.自动配置

Log4j能够在初始化期间自动配置自身。当Log4j启动时，它将找到所有ConfigurationFactory插件，并按照从最高到最低的权重排列它们。Log4j包含四个ConfigurationFactory实现：一个用于JSON，一个用于YAML，一个用于properties，一个用于XML。

* “log4j.configurationFile”系统属性
* 类路径中查找log4j2-test.properties
* 类路径中查找log4j2-test.yaml或log4j2-test.yml
* 类路径中查找log4j2-test.json或log4j2-test.jsn
* 类路径中查找log4j2-test.xml
* 类路径上查找log4j2.properties
* 查找类路径上的log4j2.yaml或log4j2.yml
* 在类路径上查找log4j2.json或log4j2.jsn
* 在类路径上找到log4j2.xml
* 如果找不到配置文件，则将使用DefaultConfiguration。 这将导致日志输出转到控制台

如果无法找到配置文件，log4j将提供默认配置。在默认配置类中提供的默认配置将设置：

* ConsoleAppender绑定到root Logger。
* PatternLayout绑定到ConsoleAppender，并设置“％d {HH：mm：ss.SSS} [％t]％-5level％logger {36} - ％msg％n”
* 默认情况下，Log4j日志级别是Level.ERROR。

默认配置文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <Root level="error">
      <AppenderRef ref="Console"/>
    </Root>
  </Loggers>
</Configuration>
```

## 2.可加性

如果想单独希望配置com.foo.Bar的日志级别。可以在配置中添加新的Logger定义：

```xml
<Logger name="com.foo.Bar" level="TRANCE"/>
<Root level="ERROR">
	<Appender ref="STDOUT"/>
</Root>
```

使用此配置，将记录来自com.foo.Bar的所有日志事件，同时记录来自所有其他组件的错误事件。

在前面的示例中，com.foo.Bar中的所有事件仍写入控制台。 这是因为com.foo.Bar的记录器没有配置任何appender，而其父进程已配置。 实际上，是以下配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <Logger name="com.foo.Bar" level="trace">
      <AppenderRef ref="Console"/>
    </Logger>
    <Root level="error">
      <AppenderRef ref="Console"/>
    </Root>
  </Loggers>
</Configuration>
```

使用以上配置，来自com.foo.Bar的Trance消息会出两次。这是因为首先使用与Logger com.foo.Bar关联的appender，它将第一个实例写入Console。接下来，引用com.foo.Bar的父级，本例中时root。然后将事件传递给它的appender，它也会写入Console，从而产生第二个。这被称为可加性。 虽然可加性可以是一个非常方便的功能，但在许多情况下，此行为被认为是不合需要的，因此可以通过将记录器上的additivity属性设置false来禁用它：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <Logger name="com.foo.Bar" level="trace" additivity="false">
      <AppenderRef ref="Console"/>
    </Logger>
    <Root level="error">
      <AppenderRef ref="Console"/>
    </Root>
  </Loggers>
</Configuration>
```

一旦事件到Logger且其可加性设置为false，该事件将不会传递给任何其父记Logger，无论其可加性如何设置。

## 3.自动重新配置

从文件加载配置时，Log4j能够自动检测配置文件的更改并重新加载配置。 如果在配置元素上指定了monitorInterval属性并将其设置为非零值，则下次评估和/或记录日志事件时将检查该文件，并且自上次检查后已经过了monitorInterval。 下面的示例显示了如何配置属性，以便仅在至少30秒后检查配置文件的更改。最小配置间隔为5秒。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration monitorInterval="30">
...
</Configuration>
```

## 4.配置语法

从版本2.9开始，处于安全原因，Log4j不处理XML文件中的DTD。

XML文件中的configuration元素接受一下几个属性（常用）：

|     属性名      |                             描述                             |
| :-------------: | :----------------------------------------------------------: |
| monitorlnterval |     检查文件配置更改之前必须经过的最短时间（以秒为单位）     |
|      name       |                          配置的名称                          |
|     scheme      | 标示类加载器的位置，以定位用于验证配置的XML架构。仅在strict设置为true时有效。 |
|  shutdownHook   |   指定在JVM关闭时Log4j是否应自动关闭。 默认情况下启用关闭    |
| shutdownTimeout | 指定JVM关闭时,多少毫秒关闭appender和后台任务。 默认值为零。  |
|     status      | 应记录到控制台的内部Log4j事件的级别。此属性的有效值为“trace”，“debug”，“info”，“warn”，“error”和“fatal”。 |
|     strict      |            允许使用严格的XML格式。 JSON配置不支持            |
|     verbose     |                    加载插件时启用诊断信息                    |

可以使用两种XML风格配置Log4j简洁和严谨。下面的文件标示XML配置的结构，简洁方式：

```xml
<?xml version="1.0" encoding="UTF-8"?>;
<Configuration>
  <Properties>
    <Property name="name1">value</property>
    <Property name="name2" value="value2"/>
  </Properties>
  <filter  ... />
  <Appenders>
    <appender ... >
      <filter  ... />
    </appender>
    ...
  </Appenders>
  <Loggers>
    <Logger name="name1">
      <filter  ... />
    </Logger>
    ...
    <Root level="level">
      <AppenderRef ref="name"/>
    </Root>
  </Loggers>
</Configuration>
```

严格的XML配置

除了上面简洁的XML格式之外，Log4j还允许以更“正常”的XML方式指定配置，可以使用XML Schema进行验证。

```xml
<?xml version="1.0" encoding="UTF-8"?>;
<Configuration>
  <Properties>
    <Property name="name1">value</property>
    <Property name="name2" value="value2"/>
  </Properties>
  <Filter type="type" ... />
  <Appenders>
    <Appender type="type" name="name">
      <Filter type="type" ... />
    </Appender>
    ...
  </Appenders>
  <Loggers>
    <Logger name="name1">
      <Filter type="type" ... />
    </Logger>
    ...
    <Root level="level">
      <AppenderRef ref="name"/>
    </Root>
  </Loggers>
</Configuration>
```

以下是使用严格格式的示例配置。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="debug" strict="true" name="XMLConfigTest"
               packages="org.apache.logging.log4j.test">
  <Properties>
    <Property name="filename">target/test.log</Property>
  </Properties>
  <Filter type="ThresholdFilter" level="trace"/>
 
  <Appenders>
    <Appender type="Console" name="STDOUT">
      <Layout type="PatternLayout" pattern="%m MDC%X%n"/>
      <Filters>
        <Filter type="MarkerFilter" marker="FLOW" onMatch="DENY" onMismatch="NEUTRAL"/>
        <Filter type="MarkerFilter" marker="EXCEPTION" onMatch="DENY" onMismatch="ACCEPT"/>
      </Filters>
    </Appender>
    <Appender type="Console" name="FLOW">
      <Layout type="PatternLayout" pattern="%C{1}.%M %m %ex%n"/><!-- class and line number -->
      <Filters>
        <Filter type="MarkerFilter" marker="FLOW" onMatch="ACCEPT" onMismatch="NEUTRAL"/>
        <Filter type="MarkerFilter" marker="EXCEPTION" onMatch="ACCEPT" onMismatch="DENY"/>
      </Filters>
    </Appender>
    <Appender type="File" name="File" fileName="${filename}">
      <Layout type="PatternLayout">
        <Pattern>%d %p %C{1.} [%t] %m%n</Pattern>
      </Layout>
    </Appender>
  </Appenders>
 
  <Loggers>
    <Logger name="org.apache.logging.log4j.test1" level="debug" additivity="false">
      <Filter type="ThreadContextMapFilter">
        <KeyValuePair key="test" value="123"/>
      </Filter>
      <AppenderRef ref="STDOUT"/>
    </Logger>
 
    <Logger name="org.apache.logging.log4j.test2" level="debug" additivity="false">
      <AppenderRef ref="File"/>
    </Logger>
 
    <Root level="trace">
      <AppenderRef ref="STDOUT"/>
    </Root>
  </Loggers>
</Configuration>
```

使用Properties配置。从版本2.4开始，Log4j现在支持通过properties文件进行配置，属性语法与Log4j 1.x中使用的语法不同。

在2.6版本之前，properties配置要求在具有这些名称的属性中以逗号分隔的列表列出appenders、 filters和logger的标识符。然后，些组件中的每一个都以组件开头的属性集定义。标识符不必匹配正在定义的组件的名称，但必须唯一地标识所有属性和子组件， 是组件的一部分。如果标识符列表不存在，则标识符必须不包含'.'。每个组件必须有一个指定的“type”属性来标识组件的插件类型。

从版本2.6开始，不再需要此标识符列表，因为在首次使用时会推断出名称，但是如果您希望使用更复杂的标识，则必须仍然使用该列表。 如果列表存在，将使用它。

与基本组件不同，在创建子组件时，您无法指定包含标识符列表的元素。 相反，您必须使用其类型定义包装器元素，如下面的RollingFileappender中的策略定义中所示。 然后定义该包装元素下面的每个子组件，下面定义了TimeBasedTriggeringPolicy和SizeBasedTriggeringPolicy。

配置如下：

```properties
status = error
dest = err
name = PropertiesConfig
 
property.filename = target/rolling/rollingtest.log
 
filter.threshold.type = ThresholdFilter
filter.threshold.level = debug
 
appender.console.type = Console
appender.console.name = STDOUT
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = %m%n
appender.console.filter.threshold.type = ThresholdFilter
appender.console.filter.threshold.level = error
 
appender.rolling.type = RollingFile
appender.rolling.name = RollingFile
appender.rolling.fileName = ${filename}
appender.rolling.filePattern = target/rolling2/test1-%d{MM-dd-yy-HH-mm-ss}-%i.log.gz
appender.rolling.layout.type = PatternLayout
appender.rolling.layout.pattern = %d %p %C{1.} [%t] %m%n
appender.rolling.policies.type = Policies
appender.rolling.policies.time.type = TimeBasedTriggeringPolicy
appender.rolling.policies.time.interval = 2
appender.rolling.policies.time.modulate = true
appender.rolling.policies.size.type = SizeBasedTriggeringPolicy
appender.rolling.policies.size.size=100MB
appender.rolling.strategy.type = DefaultRolloverStrategy
appender.rolling.strategy.max = 5
 
logger.rolling.name = com.example.my.app
logger.rolling.level = debug
logger.rolling.additivity = false
logger.rolling.appenderRef.rolling.ref = RollingFile
 
rootLogger.level = info
rootLogger.appenderRef.stdout.ref = STDOUT
```

## 5.配置Loggers

使用logger元素配置LoggerConfig。 logger元素必须具有指定的name属性，通常会指定level属性，并且还可以指定additivity属性。 可以使用TRACE，DEBUG，INFO，WARN，ERROR，ALL或OFF之一日志级别。 如果未指定日志级别，则默认为ERROR。 可以为additivity属性分配值true或false。 如果省略该属性，则将使用默认值true。

LoggerConfig还可以配置一个或多个AppenderRef元素。 引用的每个appender将与指定的LoggerConfig关联。 如果在LoggerConfig上配置有多个appender，在处理日志记录事件时，每个都会被调用。
 如果未配置，则将使用默认根LoggerConfig，其级别为ERROR且绑定了一个Console appender。 根记录器和其他记录器之间的主要区别是：

* 根Logger没有name属性
* 根Logger不支持additivity，因为它没有父级

## 6.配置Appender

使用特定的appender插件的名称或appender元素以及包含appender插件名称的type属性配置appender。 此外，每个appender必须具有一个name属性，该属性的值在appender集中是唯一的。如上一节中所述，loggers将使用该名称来引用appender。
 大多数appender还支持layout（可以使用特定的Layout插件的名称作为元素或使用“layout”作为元素名称以及包含layout插件名称的type属性来指定布局。各种appender将包含其正常运行所需的其他属性或元素。

## 7.默认属性

可以在任何Loggers，Filters，Appender等之后和之前直接放置Properties元素，并且在Properties中声明默认属性映射。

## 8.XInclude

XML配置文件可以使用XInclude包含文件。 以下log4j2.xml配置，其中包含另外两个文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration xmlns:xi="http://www.w3.org/2001/XInclude"
               status="warn" name="XIncludeDemo">
  <properties>
    <property name="filename">xinclude-demo.log</property>
  </properties>
  <ThresholdFilter level="debug"/>
  <xi:include href="log4j-xinclude-appenders.xml" />
  <xi:include href="log4j-xinclude-loggers.xml" />
</configuration>
```

log4j-xinclude-appecders.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<appenders>
  <Console name="STDOUT">
    <PatternLayout pattern="%m%n" />
  </Console>
  <File name="File" fileName="${filename}" bufferedIO="true" immediateFlush="true">
    <PatternLayout>
      <pattern>%d %p %C{1.} [%t] %m%n</pattern>
    </PatternLayout>
  </File>
</appenders>
```

log4j-xinclude-loggers.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<loggers>
  <logger name="org.apache.logging.log4j.test1" level="debug" additivity="false">
    <ThreadContextMapFilter>
      <KeyValuePair key="test" value="123" />
    </ThreadContextMapFilter>
    <AppenderRef ref="STDOUT" />
  </logger>
 
  <logger name="org.apache.logging.log4j.test2" level="debug" additivity="false">
    <AppenderRef ref="File" />
  </logger>
 
  <root level="error">
    <AppenderRef ref="STDOUT" />
  </root>
</loggers>
```

## 9.在Maven测试中使用log4j

Maven可以在构建期间运行单元和功能测试。默认情况下，放置在src / test / resources中的任何文件都会自动复制到target / test-classes，并在执行任何测试期间包含在类路径中。因此，将log4j2-test.xml放入放置在src / test / resources中将导致使用它而不是可能存在的log4j2.xml或log4j2.json。因此，在测试期间可以使用与生产中使用的不同的日志配置。

Log4j 2广泛使用的第二种方法是在junit测试类中使用@BeforeClass注释的方法中设置log4j.configurationFile属性。 这将允许在测试期间使用任意命名的文件。

Log4j 2广泛使用的第三种方法是使用LoggerContextRule JUnit的@Rule注解，该注解为测试提供了额外的便利方法。 这需要将log4j-core ，test-jar依赖项添加到测试范围依赖项中。 例如：

```xml
public class AwesomeTest {
    @Rule
    public LoggerContextRule init = new LoggerContextRule("MyTestConfig.xml");
 
    @Test
    public void testSomeAwesomeFeature() {
        final LoggerContext ctx = init.getLoggerContext();
        final Logger logger = init.getLogger("org.apache.logging.log4j.my.awesome.test.logger");
        final Configuration cfg = init.getConfiguration();
        final ListAppender app = init.getListAppender("List");
        logger.warn("Test message");
        final List<LogEvent> events = app.getEvents();
        // etc.
    }
}
```

