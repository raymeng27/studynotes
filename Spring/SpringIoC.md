# Spring

> Spring是分层的Java SE/EE应用一站式的轻量级开源框架，以IOC（Inverse of Control，控制翻转）和AOP（Aspect Oriented Programming，切面编程）为内核，提供了展现出Spring MVC、持久层Spring JDBC及业务层事务管理等一站式的企业级应用技术。

# 一、IoC

> 因为IoC确实不够开门见山，因此业界曾进行了广泛的讨论，最终软件界的泰斗级人物Martin Fowler提出来DI（Dependency Injection，依赖注入）的概念来代替IoC，即让调用类对某一接口实现类的依赖关系由第三方（容器或协作类）注入，以移除调用类对某一接口实现类的依赖。

## 1.编程模式的分析

* 传统模式，直接在调用类调用某一接口实现类，如下：

  ```java
  public class CityServiceImpl implements CityService {
  
      public static void main(String[] args) {
          CityDao cityDao = new CityDaoImpl();
      }
  }
  ```

  调用类与接口实现类耦合太紧。

* 工厂模式，调用类并不直接创建接口实现类对象，而是由工厂提供，如下：

  ```java
  public class CityFactory {
      public static CityDao getCityDao(){
          return new CityDaoImpl();
      }
  }
  ```

  ```java
  public class CityServiceImpl implements CityService {
  
      public static void main(String[] args) {
          CityDao cityDao = CityFactory.getCityDao();
      }
  }
  ```

  调用类与接口实现类虽然实现了解耦，但是这些工作在代码中已然存在，只是转移到了工厂类而已。

* 工厂模式+配置文件

  第三方容器可以帮助完成类的初始化与装配工作，让开发者从这些底层实现类的实例化、依赖关系装配等工作中解脱出来，专注于更有意义的业务逻辑开发工作。Spring就是这样的一个容器，它通过配置文件或注解描述类和类之间的依赖关系，自动完成类的初始化和依赖注入工作，Spring的配置如下：

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      <beans>
          <bean id="city" class="cn.aymeng.serviceImpl.CityServiceImpl"/>
      </beans>
  </beans>
  ```

  调用类：

  ```java
  public class test {
      public static void main(String[] args) {
          ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
          CityService cityService = applicationContext.getBean("cityService", CityService.class);
      }
  }
  ```

  通过new ClassPathXmlApplicationContext("spring.xml")等方式即可启动容器。在容器启动时，Spring根据配置文件的描述信息，自动实例化Bean并完成依赖关系的装配，从容器中即可返回准备就绪的Bean实例，后续可以直接使用。

## 2.Spring资源访问

### 1.资源访问

JDK所提供的资源访问的类（如java.net.URL、File等）并不能很好的满足各种底层资源的访问需求，比如缺少从类路径或者从Web容器的上下文中获取资源的操作类，因此，Spring设计了一个Resource接口，它为应用提供了更强的底层资源访问能力。Resource接口的主要方法：

* boolean exists()：资源是否存在
* boolean isPoen()：资源是否打开
* URL getURL() throws IOException：如果底层资源可以表示成一个URL，则该方法返回对应的URL对象。
* File getFile() throws IOException：如果底层资源对应一个文件，则该方法返回对应的File对象。
* InpurStream getInputStream() throws IOException：返回资源对应的输入流。

Resource在Spring框架中起着不可或缺的作用，Spring框架使用Resource装载各种资源，包括配置文件资源，国际化属性文件资源等，Resource的具体实现类有以下：

* ByteArrayResource：二进制数组表示的资源，二进制数组资源可以在内存中通过程序构造。
* ClassPathResource：类路径下的资源，资源以相对于类路径的方式表示。
* FileSystemResource：文件系统资源，资源以文件系统路径的方式表示。如：D:/conf/bean.xml
* InputStreamResource：以输入流返回表示的资源。
* ServletCotextResource：为访问Web容器上下文的资源而设计的类，负责相对于Web应用根目录的路径加载资源。
* UrlResource：URL封装了java.net.URL，它使用户能够访问任何可以通过URL表示的资源，如文件系统，HTTP资源，FTP资源。
* PathResource：Spring4.0提供的读取资源文件的新类。Path封装了java.net.URL、java.nio.file.path、文件系统资源，它使用户能够访问任何可以通过URL、Path、系统文件路径表示的资源，如文件系统资源、HTTP资源、FTP资源。

资源加载时默认采用系统编码读取资源内容。如果资源文件采用特殊的编码格式，那么可以通过EncodedResource对资源进行编码，以保证资源内容操作的正确性，如下代码：

```java
public class test {
    public static void main(String[] args) {
        Resource resource = new ClassPathResource("conf/file.txt");
        EncodedResource encodedResource = new EncodedResource(resource, "UTF-8");
    }
}
```

### 2.资源加载

为了访问不同类型的资源，必须使用相应的Resource类，这是比较麻烦的。Spring提供了一个强大的加载资源的机制，不但能够通过"classpath:"、"file:"等资源地址前缀识别不同的资源类型，还支持Ant风格带通配符的资源地址。

#### 1.资源地址表达式

|  地址前缀  |                示例                |                        对应的资源类型                        |
| :--------: | :--------------------------------: | :----------------------------------------------------------: |
| classpath: |    classpath:cn/aymeng/bean.xml    | 从类路径中加载资源，classpath:和classpath:/是等价的，都是相对于类的根路径。 |
|   file:    |      file:d:\\conf\\bean.xml       | 使用UrlResource从文件系统目录中装载资源，可采用绝对路径或者相对路径。 |
|  http://   | http://www.aymeng.cn/conf/bean.xml |             使用UrlResource从Web服务器中装载资源             |
|   ftp://   | ftp://www.aymeng.cn/conf/bean.xml  |             使用UrlResource从ftp服务器中装载资源             |
|  没有前缀  |      cn/aymeng/conf/bean.xml       |  根据ApplicationContext的具体实现类采用对应类型的Reasource   |

"classpath:"对应的还有另一种前缀"classpath*:"，假设有多个jar包或者文件系统类路径都拥有一个相同的包名(如cn.aymeng)。"classpath"只会在第一个加载的cn.aymeng包的类路径下查找，而"classpath*\*:"会扫描所有这些jar包及类路径下出现的类路径。

Ant风格的资源地址支持3种匹配符。

* ?：匹配文件名中的一个字符，如classpath:conf/t?xt.xml。
* *：匹配文件名中的任意字符，如file:D:/conf/\*.xml
* **：匹配多层路径，如classpath:conf/\**\*/\*.xml

#### 2. 资源加载器

Spring定义了一套资源加载的接口，并提供了实现类。ResourceLoader接口仅有一个getResource(String location)方法，可以根据一个资源地址加载文件资源。不过，资源地址仅支持带资源类型前缀的表达式，不支持Ant风格的资源路径表达式。ResourcePatternResolver拓展了ResourceLoader接口，定义了一个新的接口方法getResource(String locationPattern)，该方法支持带资源类型前缀及Ant风格的资源路径表达式。PathMatchingResourcePatternResolver是Spring提供的标准实现类。

[![gw8vPf.png](https://z3.ax1x.com/2021/05/12/gw8vPf.png)](https://imgtu.com/i/gw8vPf)

## 3.Spring容器

> Spring通过一个配置文件描述Bean与Bean之间的依赖关系，利用Java语言的反射功能实例化Bean并建立Bean之间的依赖关系。Spring的IoC容器在完成这些底层工作的基础上，还提供了Bean实例缓存，生命周期管理、Bean实例代理、事件发布，资源装载等高级服务。
>
> Bean工厂BeanFactory（com.springframework.beans.factory.BeanFactory）是Spring框架最核心的接口，它提供了高级IoC的配置机制。BeanFactory使管理不同类型的Java对象成为可能，应用上下文ApplicationContext（com.springframework.context.ApplicationContext）建立在BeanFactory基础之上，提供了更多面向应用的功能，它提供了国际化支持和框架事件体系。对于二者的用途，BeanFactory是Spring框架的基础设施，面向Spring本身；ApplicationContext面向使用Spring框架的开发者。

### 1.BeanFactory

Spring为BeanFactory提供多种实现，最常用的是XmlBeanFactory，但在Spring3.2中已被废弃，建议使用XmlBeamDefinitionReader、DefaultListableBeanFactory代替。BeanFactory的类继承体系如下：

[![gwhvxU.png](https://z3.ax1x.com/2021/05/12/gwhvxU.png)](https://imgtu.com/i/gwhvxU)

BeanFactory位于类结构树的顶端，它最主要的方法就是getBean(String beanName)，该方法从容器中返回特定名称的bean。BeanFactory的功能通过其他接口得到不断拓展：

* ListableBeanFactory：该接口定义了访问容器中Bean基本信息的若干方法，如查看bean的个数、获取某一类型的Bean的配置名、查看容器中是否包含某一个Bean等。
* HierarchicalBeanFactory：父子级联IoC容器的接口，子容器可以通过接口方法访问父容器。
* ConfigurableBeanFactory：这个接口增强了IoC容器的可定制性，它定义了设置类装载器、属性编辑器、容器初始化后置处理器等。
* AutowireCapableBeanFactory：定义了将容器Bean中的Bean按某种规则（如名字匹配、类型匹配）进行自动装配的方法。
* SingletonBeanFactory：定义了允许在运行期间容器注册单例Bean的方法。
* BeanDefinitionRegistry：Spring配置文件中每一个<bean>节点元素在Spring容器里都通过一个BeanDefinition对象表示，它描述了Bean的配置信息。

下面通过XmlBeanDefinitionReader、DefaultListableBeanFactory实现类启动Spring IoC容器：

XML配置(springapplication.xml)：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="cityService" class="cn.aymeng.serviceImpl.CityServiceImpl"/>
</beans>
```

测试类：

```java
public class BeanFactoryTest {

    @Test
    public void beanFactoryTest(){
        Resource resource = new ClassPathResource("springApplication.xml");
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
        reader.loadBeanDefinitions(resource);

        CityService cityService = factory.getBean("cityService", CityService.class);
    }
}
```

XmlBeanDefinitionReader通过Resource装载Spring配置信息并启动IoC容器，然后就可以通过BeanFactory的getBean(beanName)方法从IoC容器中获取Bean。通过BeanFactory启动IoC容器时，并不会初始化配置文件中定义的Bean，初始化动作发生在第一个调用时。对于单例的Bean来说，BeanFactory会缓存Bean实例。所以第二次使用getBean()获取Bean时，将会直接从IoC容器的缓存中获取Bean实例。

### 2.ApplicationContext

ApplicationContext由BeanFactory派生而来，提供更多面向实际应用的功能。在BeanFactory中，很多功能需要以编程的方式实现，而在ApplicationContext中则可以通过配置的方式实现。

#### 1.ApplicationContext类体系结构

ApplicationContext的主要实现类时ClassPathXmlApplicationContext和FileSystemXmlApplicationContext，前者默认从类路径下加载配置文件，后者默认从文件系统中装载配置文件，类继承体系如下：

[![gw56ht.png](https://z3.ax1x.com/2021/05/12/gw56ht.png)](https://imgtu.com/i/gw56ht)

ApplicationContext继承了HierarchicalBeanFactory和ListavleBeanFactory接口，还通过多个其他的接口拓展了BeanFactory的功能：

* ApplicationEventPublisher：让容器拥有上下文事件的功能，包括容器的启动事件、关闭事件。
* MessageSource：为应用提供i18n国际化消息访问的功能。
* ResourcePatternResolver：所有ApplicationContext实现类都实现了类似于PathMatchingResourcePatternResolver的功能，可以通过带前缀的Ant风格的资源文件路径装载Spring的配置文件。
* LifeCycle：该接口提供了start()和stop()两个方法，主要用于控制异步处理过程。
* ConfigurableApplicationContext：该接口拓展与ApplicationContext，它新增两个主要的方法：refresh()和close()，让ApplicationContext具有启动、刷新和关闭应用上下文的功能。

和BeanFactory初始化相似，ApplicationContext的初始化也很简单，如果配置文件放在类路径下，优先考虑ClassPathXmlApplicationConetxt，如果配置文件放置在文件系统的路径下，优先考虑使用FileSystemXmlApplicationContext，如下：

```java
public class BeanFactoryTest {

    @Test
    public void beanFactoryTest(){
        // 配置文件放在文件系统路径下
        // ApplicationContext ac = new FileSystemXmlApplicationContext("file:D:\\conf\\springApplication.xml");
        // 配置文件放在类路径下
        ApplicationContext ac = new ClassPathXmlApplicationContext("classpath:springApplication.xml");
        ac.getBean("cityService");
    }
}
```

还可以指定一组配置文件，Spring会自动将多个配置文件在内存中“整合”成一个配置文件，如下：

```java
ApplicationContext ac = new ClassPathXmlApplicationContext(new String[]{"classpath:springApplication.xml","classpath:springApplication2.xml"});
```

FileSystemXmlApplicationContext和ClassPathXmlApplicationContext都可以显式使用带资源类型前缀的路径，它们的区别在于如果不显示指定资源类型前缀，则分别将路径解析为文件系统路径和类路径。

Application的初始化和BeanFactory有一个重大的区别：BeanFactory在初始化容器时，并未实例化Bean，直到第一次访问某个Bean时才实例化目标Bean；而ApplicationContext则在初始化应用上下文就实例化所有单实例的Bean。

Spring支持基于类注解的配置方式，主要功能来自Spring的一个名为JavaConfig的子项目，JavaConfig现在已经升级为Spring核心框架的一部分。一个标注@Configuration的注解的POJO即可以提供Spring所需的Bean配置信息。如下：

```java
package cn.aymeng.context;

import cn.aymeng.service.CityService;
import cn.aymeng.serviceImpl.CityServiceImpl;
import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.context.annotation.Bean;

@Configurable
public class SpringBeans {

    @Bean(name = "cityService")
    public CityService buildCityService(){
        return new CityServiceImpl();
    }
}
```

Spring为基于注解类的配置提供了专门的ApplicationContext实现类：AnnotationConfigApplicationContext，启动容器如下：

```java
public class BeanFactoryTest {

    @Test
    public void annotationBeanTest(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(SpringBeans.class);
        ac.getBean("cityService");
    }
}
```

AnnotationConfigApplicationContext将加载SpringBeans.class中的Bean定义并调用SpringBeans.class中的方法实例化Bean，启动容器并装配Bean。

Spring4.0支持使用Groovy DSL来进行Bean定义配置。其与基于XML文件的配置类似，只不过基于Groovy脚本语音，可以实现复杂、灵活的Bean配置逻辑，Spring为基于Groovy的配置专门提供了的Application实现类：GenericGroovyApplicationContext。

AnnotationApplicationContext和GenericGroovyApplicationContext的类继承结构体系如下：

[![gBSeJK.png](https://z3.ax1x.com/2021/05/13/gBSeJK.png)](https://imgtu.com/i/gBSeJK)

#### 2.WebApplicationContext类体系结构

WebApplicationContext是专门为Web准备的，它允许从相对于Web根目录的路径中装载配置文件完成初始化工作。可以从它里面获得ServletContext的引入，整个Web应用上下文对象将作为属性放置到ServletContext中，以便Web应用环境可以访问Spring应用上下文。在非Web应用的环境下，Bean只有singleton和prototype两种作用域。WebApplicationContext为Bean添加了三个新的作用域：request、session和global session。WebApplicationContext的类继承体系如下：

[![gB15F0.png](https://z3.ax1x.com/2021/05/13/gB15F0.png)](https://imgtu.com/i/gB15F0)

由于Web应用比一般应用拥有更多的特性，因此WebApplicationContext扩展了ApplicationContext。WebApplicationContext定义了一个常量ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE，在上下文启动时，WebApplicationContext实例即以此为键放在在ServletContext的属性列表中，可以通过使用一下语句从Web容器中获取WebApplicationContext：

```java
WebApplicationContext ac = (WebApplicationContext) servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
```

WebApplicationContext的初始化和BeabFactory、ApplicationContext有所区别，因为WebApplicationContext需要ServletContext实例，所以，它必须在拥有Web容器的前提下才能完成启动工作。Spring分别提供了用于启动WebApplicationContext的Servlet和Web容器监听器：org.springframework.web.context.ContextLoaderServlet、org.springframework.web.context.ContextLoaderListenter。***Spring3.0以后移除了ContextLoaderServlet和Log4jConfigServlet***

Spring配置文件applicationContext.xml放在/WEB-INF/applicationContext.xml，配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="cityService" class="cn.aymeng.serviceImpl.WebServiceImpl"/>
</beans>
```

* 使用ContextLoaderListenter启动WebApplicationContext容器，代码如下（web.xml）：

  ```java
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
           version="4.0">
      <!--指定配置文件-->
      <context-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>/WEB-INF/applicationContext.xml</param-value>
      </context-param>
  
      <!--声明Web容器监听器-->
      <listener>
          <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
      </listener>
  </web-app>
  ```

  ContextLoaderListenter通过Web容器上下文参数contextConfigLocation参数获取Spring配置文件的位置。用户可以指定多个配置文件，用逗号、空格或冒号分隔均可。对于未带资源类型前缀的配置文件路径，WebApplicationContext默认这些路径相对于Web的部署跟路劲。带资源类型前缀的路径配置也是一样的。

* 使用@Configuration的Java类配置信息启动容器，代码如下（web.xml）：

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
           version="4.0">
      <!--通过指定context参数，让spring使用AnnotationConfigWebApplication而非XmlWenApplicationContext启动容器-->
      <context-param>
          <param-name>contextClass</param-name>
          <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
      </context-param>
  
      <!--指定标注了@Configuration的配置类，多个可以使用逗号或者空格分隔-->
      <context-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>cn.aymeng.context.SpringBeans</param-value>
      </context-param>
  
      <!--声明spring容器的监听器-->
      <listener>
          <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
      <listener>
  </web-app>
  ```

## 4.Spring配置——在IoC容器中装配Bean

Bean元数据信息在Spring容器中的内部对应物是由一个个BeanDefinition形成的Bean注册表，Spring实现了Bean元数据信息内部表示和外部定义的解耦。Spring支持多种形式的Bean配置方式。Spring1.0仅支持基于XML的配置，Spring2.0新增基于注解配置的支持，Spring3.0新增基于Java类配置的支持，Spring4.0则新增基于Groovy动态语言配置的支持。

Spring配置信息首先定义了Bean的实现及依赖关系，Spring容器根据各种形式的配置信息在容器内部建立Bean定义注册表，然后根据注册表加载、实例化Bean，并建立Bean与Bean之间的依赖关系，最后将这些准备就绪的Bean放到Bean缓存池中，以供外层的应用程序使用。

对于基于XML的配置，Spring1.0的配置文件采用DTD格式，Spring2.0以后采用Schema格式。Spring配置的Schema文件放置在各模块JAR文件内一个名为config的目录下，下面是些常用的Schema文件：

|    Schema文件    |                         说明                          |                     命名空间和Schema文件                     |
| :--------------: | :---------------------------------------------------: | :----------------------------------------------------------: |
| spring-beans.xsd |         Spring主要的配置Schema，用于配置Bean          | 【命名空间】：xmlns="http://www.springframework.org/schema/beans"<br />【Schema文件】：http://www.springframework.org/schema/beans/spring-beans.xsd |
|  spring-aop.xsd  |                  AOP配置定义的Schema                  | 【命名空间】：xmlns:aop="http://www.springframework.org/schema/aop"<br />【Schema文件】：https://www.springframework.org/schema/aop/spring-aop.xsd |
|  spring-context  |                   扫描Bean需要使用                    | 【命名空间】：xmlns:context="http://www.springframework.org/schema/context"<br />【Schema文件】：https://www.springframework.org/schema/context/spring-context.xsd |
|  spring-mvc.xsd  |          MVC配置的Schema，是Spring3.0新增的           | 【命名空间】：xmlns:mvc="http://www.springframework.org/schema/mvc"<br />【Schema文件】：https://www.springframework.org/schema/mvc/spring-mvc.xsd |
|  spring-tx.xsd   |              声明式事务配置定义的Schema               | 【命名空间】：xmlns:tx="http://www.springframework.org/schema/tx"<br />【Schema文件】：https://www.springframework.org/schema/tx/spring-tx.xsd |
| spring-jdbc.xsd  | 为配置Spring内置数据库提供的Schema，是Spring3.0新增的 | 【命名空间】：xmlns:jdbc="http://www.springframework.org/schema/jdbc"<br />【Schema文件】：https://www.springframework.org/schema/jdbc/spring-jdbc.xsd |

### 1.Bean基本配置

在Spring容器的配置文件中定义一个简单的Bean配置：

```xml
<bean id="cityService" class="cn.aymeng.service.impl.CityServiceImpl"/>
```

在配置Bean时，需要为其指定一个id属性作为Bean的名称，id在配置种是唯一的，并且需要满足XML对id的命名规范，如果需要用一些特殊字符进行命名，可以使用name属性，name在没有字符上的限制，几乎可以使用任意字符。id和name都可以指定多个名字，中间用逗号、分号或者空格进行分隔。Spring配置文件不允许出现两个相同id的<bean>，但却可以出现两个相同name的<bean>。如果有多个name相同的bean，通过容器获取bean时将返回后面声明的那个Bean，原因是后面的Bean覆盖了前面相同的Bean，如果id和name两个属性都未指定，那么Spring自动将全限定类名作为Bean的名称。

### 2.依赖注入

#### a.属性注入

属性注入指通过setXxx()方法注入Bean的属性值或依赖对象，属性注入是实际应用中最常采用的注入方式。属性注入要求Bean提供一个默认的构造函数，并为需要注入的属性提供对应的Setter方法。Spring先调用Bean的默认构造方法实例化Bean对象，然后通过反射的方式调用Setter方法注入属性的值。简单例子如下：

City实体类：

```java
package cn.aymeng.entity;

public class City {
    private String name;
    private String capital;

    public City() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getCapital() {
        return capital;
    }

    public void setCapital(String capital) {
        this.capital = capital;
    }
}
```

则在文件中对Car进行属性注入的配置为：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="city" class="cn.aymeng.entity.City">
        <property name="name" value="上海"/>
        <property name="capital" value="上海"/>
    </bean>
</beans>
```

Spring只会检查Bean中是否有对应的Setter方法，至于Bean中是否有对应的属性成员则不做要求。属性变量名必须满足：***变量的前两个字母要么全部大写，要么全部小写***。

#### b.构造函数注入

构造函数注入的前提是Bean必须提供带参的构造函数。如下的Studeng的Bean类：

```java
package cn.aymeng.entity;

public class Student {
    private int no;
    private String name;

    public Student(int no, String name) {
        this.no = no;
        this.name = name;
    }
}
```

构造函数的注入的配置和属性注入的配置方式不同，下面是在Spring中使用构造函数方式装配Student的Bean类：

```xml
<bean id="student" class="cn.aymeng.entity.Student">
    <constructor-arg type="int"><value>1515925192</value></constructor-arg>
    <constructor-arg type="java.lang.String"><value>Ray</value></constructor-arg>
</bean>
```

在<constructor-arg>的元素中有一个type属性，它为Spring提供了判断配置项和构造函数入参对应关系的“信息”。如果构造函数有两个类型相同的入参，那么仅通过type就无法确定对应关系了，这是需要通过入参索引的方式进行确定。配置如下：

```xml
<bean id="student" class="cn.aymeng.entity.Student">
    <constructor-arg index="0"><value>1515925192</value></constructor-arg>
    <constructor-arg index="1"><value>Ray</value></constructor-arg>
</bean>
```

构造函数的第一个参数索引为0，第二个为1，以此类推。有时需要type和index联合使用才能确定配置项和构造函数入参的对应关系。配置如下：

```xml
<bean id="student" class="cn.aymeng.entity.Student">
    <constructor-arg index="0" type="int"><value>1515925192</value></constructor-arg>
    <constructor-arg index="1" type="java.lang.String"><value>Ray</value></constructor-arg>
</bean>
```

Spring容器能对构造函数配置的Bean进行实例化又一个前提，即Bean构造函数入参引用的对象必须已经准备就绪。由于这个机制的限制，如果两个Bean都采用构造函数注入，而且都通过构造函数入参引用对方，就会发生类似于线程死锁的循环依赖问题，这种情况下，只需要修改Bean的代码，将构造函数注入方式调整为属性注入方式就可以了。

#### c.工厂方法注入

工厂方法是在应用中经常使用的设计模式，它是控制反转和单实例设计思想的主要实现方法。由于Spring IoC容器以框架的方式提供工厂方法的功能，并以透明的方式提供给开发者，所以很少需要手工编写基于工厂方法的类。在一些遗留的系统或第三方类库，也还会有工厂方法，这时可以使用Spring工厂方法注入的方式进行配置。

* 非静态工厂方法

  有些工厂方法是非静态，即必须实例化工厂类后才能调用工厂方法。如下为City提供一个非静态的工厂类：

  ```java
  package cn.aymeng.factory;
  import cn.aymeng.entity.City;
  public class CityFactory {
      // 创建City的工厂方法
      public City createCity(){
          City city = new City();
          city.setName("上海");
          city.setCapital("上海");
          return city;
      }
  }
  ```

  Spring配置为：

  ```xml
  <bean id="cityFactory" class="cn.aymeng.factory.CityFactory"/>
  <bean id="city" factory-bean="cityFactory" factory-method="createCity"/>
  ```

  由于CityFactory工厂类的工厂方法不是静态的，所以首先需要定义一个工厂类的Bean，然后通过factory-bean引入工厂类实例，最后通过factory-method指定对应的工厂类方法。

* 静态工厂方法

  很多工厂类方法都是静态的，这意味着用户在无须创建工厂类实例的情况下就可以调用工厂类方法，因此，静态工厂方法比静态工厂方法更易使用。使createCity()方法调整为静态的：

  ```java
  package cn.aymeng.factory;
  import cn.aymeng.entity.City;
  public class CityFactory {
      // 创建City的工厂方法
      public static City createCity(){
          City city = new City();
          city.setName("上海");
          city.setCapital("上海");
          return city;
      }
  }
  ```

  Spring配置为：

  ```xml
  <bean id="city" class="cn.aymeng.factory.CityFactory" factory-method="createCity"/>
  ```

  当使用静态工厂类型的方法后，就无须配置定义工厂类的Bean，直接在<bean>中通过class属性指定工厂类，再通过factory-method指定对应的工厂方法。

### 3.注入参数详解

#### a.字面值

所谓“字面值”一般是指可用字符串表示的值，这些值可以通过<value>元素标签进行注入，如下：

```xml
<bean id="city" class="cn.aymeng.entity.City">
    <property name="name">
        <value>上海</value>
    </property>
    <property name="capital">
        <value>上海</value>
    </property>
</bean>
```

如果字符串包含一个XML的特殊符号，需要加上<![CDATA[]]>。如下：

```xml
<bean id="city" class="cn.aymeng.entity.City">
    <property name="name">
        <value>上海</value>
    </property>
    <property name="capital">
        <value><![CDATA[&上海&]]></value>
    </property>
</bean>
```

<![CDATA[]]>的作用是让XML解析器将标签中的字符当做普通的文本对待。XML中共有5个特殊字符，分别是&、<、>、"、'。或者使用特殊字符的转义。XML特殊字符实体符号：

| 特殊符号 |  实体符号  | 特殊符号 |  实体符号  |
| :------: | :--------: | :------: | :--------: |
|    <     |  &amp;lt;  |    >     |  &amp;gt;  |
|    &     | &amp;amp;  |    "     | &amp;quot; |
|    '     | &amp;apos; |          |            |

#### b.引用其它Bean

Spring可以通过<ref>元素引用其它Bean，<ref>可以通过一下两个属性引用容器中其它的Bean。

* bean：通过该属性可以引用同一容器或父容器的Bean，这是最常见的形式。
* parent：容器中的Bean，如<ref parent="city">的配置说明city的Bean是父容器的Bean。

例：

```xml
<bean id="city" class="cn.aymeng.entity.City">
    <property name="capital"><ref bean="capital" /></property>
</bean>
```

#### c.内部Bean

如果一个Bean只被另外一个Bean引用，而不被容器中任何其他的Bean引用，则可以将它以内部Bean的方式注入，如下：

```xml
<bean id="city" class="cn.aymeng.entity.City">
    <property name="capital">
        <bean class="cn.aymeng.entity.capital"/>
    </property>
</bean>
```

内部Bean即使提供id、name、scope属性，也会被忽略，scope默认为prototype。

#### d.null值

<null/>元素标签可以为Bean的字符串或其他对象类型的属性注入null值，如下：

```xml
<bean id="city" class="cn.aymeng.entity.City">
    <property name="name">
        <null/>
    </property>
</bean>
```

#### e.级联属性

Spring没有对级联属性的层数进行限制，只要配置的Bean拥有对应于级联属性的类结构，就可以配置任意层级的级联属性，如下定义了具有三级结构的级联属性：

```xml
<bean id="city" class="cn.aymeng.entity.City">
    <property name="name" value="city.capital.value"/>
</bean>
```

#### f.集合类型属性

* List

  ```xml
  <bean id="city" class="cn.aymeng.entity.City">
      <property name="capital">
          <list>
              <value>闵行</value>
              <value>黄浦</value>
              <value>静安</value>
          </list>
      </property>
  </bean>
  ```

* Set

  ```xml
  <bean id="city" class="cn.aymeng.entity.City">
      <property name="capital">
          <set>
              <value>闵行</value>
              <value>黄浦</value>
              <value>静安</value>
          </set>
      </property>
  </bean>
  ```

* Map

  ```xml
  <bean id="city" class="cn.aymeng.entity.City">
      <property name="capital">
          <map>
              <entry>
                  <key><value>id</value></key>
                  <value>201100</value>
              </entry>
              <entry>
                  <key><value>name</value></key>
                  <value>闵行区</value>
              </entry>
          </map>
      </property>
  </bean>
  ```

* Properties

  ```xml
  <bean id="city" class="cn.aymeng.entity.City">
      <property name="capital">
          <props>
              <prop key="id">201100</prop>
              <prop key="name">闵行区</prop>
          </props>
      </property>
  </bean>
  ```

#### g.自动装配

Spring IoC容器知道所有Bean的配置信息，此外，通过Java反射机制还可以获知实现类的结构信息，如构造函数方法的结构、属性等信息。<bean>元素提供了一个指定自动装配类型的属性：autowire="自动装配类型"。Spring提供了4中自动装配类型，如下所示：

| 自动装配类型 |                             说明                             |
| :----------: | :----------------------------------------------------------: |
|    byName    | 根据名称进行自动装配。假设一个类有一个名为city的属性，如果容器中刚好有一个名为city的Bean，Spring就会自动将其装配给这个类的city属性 |
|    byType    | 根据类型进行自动匹配，假设一个类中有一个City类型的属性，如果容器中刚好有一个名为City类型的Bean，Spring就会自动将其装配给这个类的这个属性 |
| constructor  | 与byType类似，只不过它是针对构造函数注入而言的。如果一个类的构造函数包含一个City类型的入参，如果容器中有一个City类型的Bean，则Spring将会自动把这个Bean作为这个类构造函数的入参，如果没有的话，Spring将抛出异常 |
|  autodetect  | 根据Bean的自省机制决定采用byType还是constructor进行自动装配。如果Bean提供了默认的构造函数，则采用byType，否则采用constructor。 |

<beans>元素标签中的default-autowire属性可以配置全局自动匹配，default-autowire属性的默认值为no，表示不启用自动装配，不过在<beans>中定义的自动装配策略可以被<bean>的自动装配策略覆盖。在实际开发中，XML配置方式很少启用自动装配功能，而基于注解的配置方式默认采用byType自动装配策略。

#### h.整合多个配置文件

对于一个大型应用来说，可能存在多个XML配置文件，在启动Spring容器时，可以通过一个String数组指定这些配置文件。如下：

```java
ApplicationContext ac = new ClassPathXmlApplicationContext(new String[]{"classpath:springApplication.xml","classpath:springApplication1.xml"});
```

Spring还允许通过<import>将多个配置文件引用到一个文件中，进行配置文件的集成，这样，在启动Spring容器时，仅需指定这个合并好的配置文件即可，如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <import resource="classpath:springApplication.xml"/>
    <bean id="cityService" class="cn.aymeng.serviceImpl.WebServiceImpl"/>
    <bean id="student" class="cn.aymeng.entity.Student"/>
</beans>
```

### 5.Bean作用域

Spring配置文件中可以定义Bean的作用域，Spring 4.0支持的所有作用域如下：

|     类型      |                             说明                             |
| :-----------: | :----------------------------------------------------------: |
|   singleton   | 在Spring IoC容器中仅存在一个Bean实例，Bean以单实例的方式存在 |
|   prototype   |          每次从容器中获取Bean时，都返回一个新的实例          |
|    request    | 每次HTTP请求都会创建一个新的Bean。该作用域仅适用于WebApplicationContext环境 |
|    session    | 同一个HTTP session共享一个Bean，该作用域仅适用于WebApplicationContext环境 |
| globalsession | 同一个全局Session共享一个Bean，一般用于Portlet环境，该作用域仅适用于WebApplicationContext环境 |

在低版本的Spring中，仅支持两个Bean作用域，所以采用singleton="true|false"的配置方式。Spring向后兼容，已然支持这种配置方式。不过Spring推荐采用新的配置方式：scope="作用域类型"。

#### a.singleton作用域

一般情况下，无状态或者状态不可变的类适合使用单例模式，不过Spring对此实现了超越。在传统开发中，由于DAO类持有Connection这个非线程安全的变量，因此往往未采用单例模式。而在Spring环境中，对于所有的DAO类都可以采用单例模式，因为Spring利用AOP和LocalThread功能，对非线程安全的变量进行了特殊处理，使这些非线程安全的类变成了线程安全的类，所以在实际应用中，大部分Bea都能以单实例的方式运行，Spring将Bean的默认作用域定位singleton。如下配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="city" class="cn.aymeng.entity.City" scope="singleton"/>
    <bean id="student1" class="cn.aymeng.entity.Student">
        <property name="city" ref="city"/>
    </bean>
    <bean id="student2" class="cn.aymeng.entity.Student">
        <property name="city" ref="city"/>
    </bean>
    <bean id="student3" class="cn.aymeng.entity.Student">
        <property name="city" ref="city"/>
    </bean>
</beans>
```

在默认情况下，Spring的ApplicationContext容器在启动时，自动实例化所有singleton的Bean并缓存与容器中，在运行时用到该Bean时就无须再实例化了，提高了运行效率。如果不希望在容器启动时提前实例化singleton的Bean，则可以通过lazy-init属性进行控制，如下：

```xml
<bean id="city" class="cn.aymeng.entity.City" lazy-init="true"/>
```

#### b.prototype作用域

采用scope="prototyepe"指定非单例作用域的Bean，如下：

```xml
<bean id="city" class="cn.aymeng.entity.City" scope="prototype"/>
<bean id="student1" class="cn.aymeng.entity.Student">
    <property name="city" ref="city"/>
</bean>
<bean id="student2" class="cn.aymeng.entity.Student">
    <property name="city" ref="city"/>
</bean>
<bean id="student3" class="cn.aymeng.entity.Student">
    <property name="city" ref="city"/>
</bean>
```

在默认情况下，Spring容器在启动时不实例化prototype的Bean。此外，Spring容器将prototype的Bean交给调用者后，就不再管理它的生命周期。

#### c.Web相关的作用域

如果使用Spring的WebApplicationContext，则可使用另外3中Bean的作用域：request、session和globalSession。在使用这些作用域之前，首先必须在Web容器中进行一些额外的配置。在低版本的中Web容器中（Servlet 2.3之前），用户可以使用HTTP请求过滤器进行配置，如下：

```xml
<filter>
    <filter-name>requestContextFilter</filter-name>
    <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>requestContextFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

在高版本的Web容器中，则可以利用HTTP请求监听器进行配置。如下：

```xml
<listener>
    <listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
</listener>
```

在整合Spring容器时使用ContextLoaderListenter，它实现了ServletContextListenter监听器接口，ServletContextListenter只负责监听Web容器启动和关闭的事件。而RequestContextListenter实现ServletRequestListenter监听器接口，该监听器监听HTTP请求事件，Web服务器接收的每一次请求都会通知该监听器。

* request作用域

  request作用域的Bean对应一个HTTP请求和生命周期，每次HTTP请求调用Bean，Spring容器就会创建一个新的Bean，请求完毕后，就会销毁这个Bean，配置如下：

  ```xml
  <bean id="city" class="cn.aymeng.entity.City" scope="request"/>
  ```

* session作用域

  session作用域横跨整个HTTP Session，Session中的所有HTTP请求都共享一个Bean。当HTTP Session结束后，实例才销毁，配置如下：

  ```xml
  <bean id="city" class="cn.aymeng.entity.City" scope="session"/>
  ```

* globalSession作用域

  globalSession作用域类似session作用域，不过仅在Portlet的Web应用中使用，Porlet规范定义了全局Session的概念，它被组成Portlet Web应用的所有子Portlet共享。如果不在Portlet Web环境下，那么globalSession作用域等价于Session，配置如下：

  ```xml
  <bean id="city" class="cn.aymeng.entity.City" scope="golbalSession"/>
  ```


## 5.基于注解的配置

### 1.@Component、@Repository、@Service、@Controller

Spring从2.0开始就引入了基于注解的配置方式，在2.5时得到完善，在4.0进一步加强。Spring使用@Component注解进行标注，它可以被Spring容器识别，Spring容器自动将POJO转化为容器管理的Bean，如下：

```java
package cn.aymeng.service;

import cn.aymeng.dao.StudentDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

/**
 * @author Ray
 * @version 1.0
 * @date 2021/5/17 1:24
 */
@Component
public class StudentService {
    private StudentDao studentDao;
    private String name;

    public StudentService(StudentDao studentDao, String name) {
        this.studentDao = studentDao;
        this.name = name;
    }
}
```

它和下面的配置是等效的：

```xml
<bean id="StudentService" class="cn.aymeng.service.StudentService"/>
```

除@Component外，Spring还提供了3个功能基本和@Component等效的注解，分别对于DAO、Service及Web层的Controller进行注解：

* @Repository：用于对DAO层实现类进行注解。
* @Service：用于对Service实现类进行注解。
* @Controller：用于对Controller实现类进行注解。

Spring提供了一个context命名空间，他提供了通过扫描类包已应用注解定义Bean的方式，代码如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="cn.aymeng.service"/>
</beans>
```

声明context命名空间后，即可通过context命名空间的component-scan的base-package属性指定一个需要扫描的基类包，如果仅希望扫描特定的类而非基包下的所有类，那么可以使用resource-pattren属性过滤特定的类，如下：

```xml
<context:component-scan base-package="cn.aymeng" resource-pattern="service/*.class"/>
```

这里将基类设置为cn.aymeng，默认情况下可以扫描基类包里的所有类，将resource-pattren设置为"entity/*.class"，则Spring仅会扫描基类包里entity子包中的类。

\<context:component-scan\>有两个子标签可以过滤子元素：\<context:include-filter\>表示要包含的目标类。\<context:exclude-filer\>表示要排除的目标类，如下：

```xml
<context:component-scan base-package="cn.aymeng" resource-pattern="service/*.class">
    <context:include-filter type="regex" expression="cn\.aymeng\.service\.*"/>
    <context:exclude-filter type="aspectj" expression="cn.aymeng..*StudentService+"/>
</context:component-scan>
```

\<context:component-scan\>下可以有多个子标签，且include-filter必须放在exclude-filter上方，这两个过滤元素均支持多种类型的过滤表达式，说明如下：

|    类别    |           示例           |                             说明                             |
| :--------: | :----------------------: | :----------------------------------------------------------: |
| annotation | com.aymeng.XxxAnnotation | 所有标注了XxxAnotation的类，该类型采用目标类是否标注了某个注解进行过滤 |
| assignable |   cn.aymeng.XxxService   | 所有继承或拓展了XxxService的类，该类型采用目标类是否继承或拓展了某个特定类进行过滤 |
|  aspectj   |   ca.aymeng..*Service+   |          所有类名以Service结束及继承或拓展他们的类           |
|   regex    | com\.aymeng\.service\.*  | 所有cn.aymeng.service类包下的类，该类型采用正则表达式根据目标类的类名进行过滤 |
|   custom   | com.aymeng.XxxTypeFilter | 采用XxxTypeFilter代码方式实现过滤，该类必须实现org.springframework.core.type.TypeFilter接口 |

### 2.@Autowired

Spring通过@Autowired注解实现Bean的依赖注入，如下：

```java
@Component
public class StudentService {
    @Autowired
    private StudentDao studentDao;
    @Autowired
    private String name;

    public StudentService(StudentDao studentDao, String name) {
        this.studentDao = studentDao;
        this.name = name;
    }
}

```

@Autowired默认按类型（byType）匹配的方式在容器中查找匹配的Bean，当且仅有一个匹配的Bean时，Spring将其注入@Autowired标注的变量中，如果容器中没有一个和标注变量类型匹配的Bean，那么Spring容器启动时将 报NoSuchBeanDefinitionException异常，如果希望Spring即使匹配不到的Bean完成注入也不报异常可以使用@Autowired(required=false)进行标注，如下：

```java
@Component
public class StudentService {
    @Autowired(required = false)
    private StudentDao studentDao;
    @Autowired(required = false)
    private String name;

    public StudentService(StudentDao studentDao, String name) {
        this.studentDao = studentDao;
        this.name = name;
    }
}
```

### 3.@Qualifier

如果容器中有一个以上匹配的Bean时，可以通过@Qualifier注解限定Bean的名称，如下：

```java
@Component
public class StudentService {
    @Autowired(required = false)
    @Qualifier("studentDao")
    private StudentDao studentDao;
    @Autowired(required = false)
    private String name;

    public StudentService(StudentDao studentDao, String name) {
        this.studentDao = studentDao;
        this.name = name;
    }
}
```

也可以对类方法进行标注，如下：

```java
@Component
public class StudentService {
    private StudentDao studentDao;
    private String name;

    @Autowired
    public StudentService(@Qualifier("studentDao") StudentDao studentDao, String name) {
        this.studentDao = studentDao;
        this.name = name;
    }
}
```

***tips：Spring官方不建议对类属性进行注解，可以对构造方法或get方法进行注解***

### 4.@Lazy

Spring4.0支持延迟依赖注入，即在Bean容器启动的时候，对于在Bean上标注@Lazy及@Autowired注解的属性，不会立即注入属性值，而是延迟到调用此属性的时候才会注入属性值，如下：

```java
package cn.aymeng.service;

import cn.aymeng.dao.StudentDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Component;

/**
 * @author Ray
 * @version 1.0
 * @date 2021/5/17 1:24
 */
@Lazy
@Component
public class StudentService {
    private StudentDao studentDao;
    private String name;

    @Lazy
    @Autowired
    public StudentService(@Qualifier("studentDao") StudentDao studentDao, String name) {
        this.studentDao = studentDao;
        this.name = name;
    }
}
```

对Bean实施延迟依赖注入，要注意@Lazy注解必须同时标注在属性及目标Bean上，二者缺一，则延迟注入无效。

### 5.@Scope

Spring为注解配置提供了一个@Scope注解，可以通过它显示指定Bean的作用范围，如下：

```java
@Scope("prototype")
@Service
public class CityService {
    private CityDao cityDao;
    private String name;

    public CityService(CityDao cityDao, String name) {
        this.cityDao = cityDao;
        this.name = name;
    }
}
```

### 6.@PostConstruct、@PreDestroy

Spring从2.5开始支持@PostConstruct、@PreDestroy注解，在Spring相当于init-method和destroy-method属性的功能，可以在一个Bean中定义多个@PostConstruct、@PreDestroy方法，如下：

```java
@Scope("prototype")
@Service
public class CityService {
    private CityDao cityDao;
    private String name;

    @Autowired
    public CityService(CityDao cityDao, String name) {
        this.cityDao = cityDao;
        this.name = name;
    }

    @PostConstruct
    public void init01(){
        System.out.println("init01...");
    }
    @PostConstruct
    public void init02(){
        System.out.println("init02...");
    }
    @PreDestroy
    public void destroy01(){
        System.out.println("destroy01...");
    }
    @PreDestroy
    public void destroy02(){
        System.out.println("destroy02...");
    }
}
```

## 6.基于Java类的配置

### 1.@Configuration配置Spring容器

普通的POJO只要标注@Configuration注解，就可以为Spring容器提供Bean定义的信息，每个标注了@Bean的类方法都相当于提供了一个Bean的定义信息，如下代码：

```java
package cn.aymeng.context;

import cn.aymeng.dao.CityDao;
import cn.aymeng.dao.StudentDao;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author Ray
 * @version 1.0
 * @date 2021/5/17 3:37
 */
@Configuration
public class AppConf {

    @Bean
    public CityDao cityDao(){
        return new CityDao();
    }

    @Bean
    public StudentDao studentDao(){
        return new StudentDao();
    }
}
```

AppConf类的定义标注了@Configuration注解，说明这个类可用于为Spring提供Bean的定义信息。该类的方法可标注@Bean注解，Bean的类型由方法返回值的类型决定，名称默认和方法名相同，也可以通过入参显示指定Bean的名称，如@Bean(name="cityDao")。

由于@Configuration注解类本身已经标注了@Component注解，所以任何标注了@Configuration的类，本身也相当于标注了@Component，即它们可以像普通的Bean一样被注入到其他Bean中，如下：

```java
@Configuration
public class ServiceConf {

    @Autowired
    private AppConf appConf;

    @Bean
    public CityDao cityDao(){
       return appConf.cityDao();
    }
}
```

同样可以在Bean方法上使用@scope注解来定义Bean的作用域，如下

```java
@Configuration
public class AppConf {
    
    @Scope("prototype")
    @Bean
    public CityDao cityDao(){
        return new CityDao();
    }

    @Bean
    public StudentDao studentDao(){
        return new StudentDao();
    }
}
```

### 2.使用基于Java类的配置信息启动Spring容器

Spring提供了一个AnnotationConfigApplicationContext类，它能够直接通过标注@Configuration的Java类启动容器，如下：

```java
/**
 * @author Ray
 * @version 1.0
 * @date 2021/5/17 3:57
 */
public class SpringConfigTest {
    
    @Test
    public void annotationApplicationContext(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConf.class);
        ac.getBean("cityDao");
    }
}
```

此外，也可以启动多个@Configuration配置类，然后通过刷新容器应用这些配置类，如下：

```java
public class SpringConfigTest {
	@Test
    public void javaConfigTest(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext();
        ac.register(AppConf.class);
        ac.register(ServiceConf.class);
        ac.refresh();

        ac.getBean("cityDao");
    }
}
```

### 3.通过XML配置文件引用@Configuration配置

标注了@Configuration的配置类与标注了@Component的类一样也是一个Bean它可以被Spring的\<context:component-scan\>扫描到，如果希望将配置类装到XML配置文件中，可以如下配置：

```xml
<context:component-scan base-package="cn.aymeng.context" resource-pattern="AppCon.class"/>
```

### 4.@ImportResource：通过@Configuration配置类引用XML配置信息

在@Configuration配置类中可以通过@ImportResource引入XML配置文件，在Bean方法中直接通过@Autowired引用XML配置文件中定义的Bean，代码如下：

```java
package cn.aymeng.context;

import cn.aymeng.dao.StudentDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ImportResource;

/**
 * @author Ray
 * @version 1.0
 * @date 2021/5/17 4:10
 */
@Configuration
@ImportResource("classpath:applicationContext.xml")
public class AnnotationConf {
    
    @Bean
    @Autowired
    public SchoolDao schoolDao(StudentDao studentDao){
        SchoolDao schoolDao = new SchoolDao();
        schoolDao.setStudentDao(studentDao);
        return schoolDao;
    }
}
```