# Spring AOP

> AOP是有特定的应用场合的，它只适合那些具有横切逻辑的应用场合，如性能监测、访问控制、事务管理及日志记录。

# 一、AOP概念

## 1.AOP引入

AOP是Aspect Oriented Programing的简称，最初被译为“面向方面编程”，但更倾向于叫做“面向切面编程”。按照软件重构思想的理念，如果多个类中出现相同的代码，则应该考虑定义一个父类，将这些相同的代码提取到父类。通过引入父类消除多个类中重复代码的方式在大多数情况下是可行的，但世界并非永远这样简单，如下代码：

```java
package cn.aymeng.service;

public class ForumService {
    private TransactionManager transactionManager;
    private PerformanceMonitor pmonitor;
    private TopicDao topicDao;
    private ForumDao forumDao;
    
    public void removeTopic(int topicId){
        pmonitor.start();
        transactionManager.beginTransaction();
        topicDao.removeTopic(topicId);
        transactionManager.commit();
        pmonitor.end();
    }

    public void createTopic(Forum forum){
        pmonitor.start();
        transactionManager.beginTransaction();
        topicDao.create(forum);
        transactionManager.commit();
        pmonitor.end();
    }
}
```

代码中pmonitor是方法性能监视代码，transactionManager是事务开始和事务提交的代码，性能监视和事务管理这些代码葛藤缠树般包围着业务性代码。我们无法通过抽象父类的方式消除如上所示的重复性横切代码，因为这些横切逻辑依附在业务类方法的流程中，它们不能转移到其他地方去。AOP通过横向抽取机制为这些类无法通过纵向继承体系进行抽象的重复性代码提供了解决方案。AOP希望将这些分散在各个业务逻辑代码中的相同代码通过横向切割的方式抽取到一个独立的模块中。将这些重复的代码独立出来很容易，但如何将这些独立的代码融合到业务逻辑中以完成和原来一样的业务流程，才是事情的关键，这也是AOP要解决的主要问题。

## 2.AOP术语

### a.连接点（Joinpoint）

特定点是程序执行的某个特定位置，如类开始初始化前、类初始化后、类的某个方法调用前/调用后、方法抛出异常后。一个类或一段程序代码拥有一些具有边界性质的特定点，这些代码中的特定点被称为“连接点”。Spring仅支持方法的连接点，即仅能在方法调用前、方法调用后、方法抛出异常时这些程序执行点织入增强。

### b.切点（Pointcut）

每一个程序类都拥有多个连接点，如一个拥有两个方法的类，这两个方法都是连接点，即连接点是程序类中客观存在的食物。AOP通过切点定位特定的连接点。连接点相当于数据库中的记录，而切点相当于查询条件。在Spring中，切点通过org.springframework.aop.Pointcut接口进行描述，它使用类和方法作为连接点的查询条件，Spring AOP的规则解析引擎负责解析切点所设的的查询条件，找到对应的连接点。确切的说，应该是执行点而非连接点，因为连接点是方法执行前、执行后等包括方位信息的具体程序执行点，而切点只定位到某个方法上，所以如果希望定位到具体的连接点上，嗨需要提供方位信息。

### c.增强（Advice）

增强是织入目标连接点上的一段程序代码。在Spring中，增强除用于描述一段程序代码外，还拥有另一个和连接点相关的信息，这便是执行点的方位。结合执行点的方位信息和切点信息，就可以找到特定的连接。正因为增强既包含用于添加到目标连接点上的一段执行逻辑，有包含用于定位连接点的方位信息，所有Spring所提供的增强接口都是带方位名的，如BeforeAdvice、AfterReturningAdvice、ThrowsAdvice等。

### d.目标对象（Target）

增强逻辑的织入目标类。如果没有AOP，那么目标业务类需要自己实现所有的逻辑。在AOP的帮助下，ForumService只实现那些非横切逻辑的程序逻辑，而性能监视和事务管理等这些横切逻辑则可以使用AOP动态织入特定的连接点上。

### e.引介（Introduction）

引介是一种特殊的增强，它为类添加一些属性和方法。这样即使一个业务类原本没有实现某个接口，通过AOP的引介功能，也可以动态地为该业务添加接口的实现逻辑，让业务成为这个接口的实现类。

### f.织入（Weaving）

织入是将增强添加到目标类的具体连接点上的过程，AOP有三种织入方式：

* 编译器织入，这要求使用特殊的Java编译器。
* 类加载期织入，这要求使用特殊的类装载器。
* 动态代理织入，在运行期为目标类添加增强生成子类发方式。

Spring采用动态代理织入，而AspectJ采用编译器织入和类装载期织入。

### g.代理（Proxy）

一个类被AOP织入增强后，就产生了一个结果类，它是融合了原类和增强逻辑的代理类。根据不同的代理方式，代理类既可能是和原类具有相同接口的类，也可能就是原类的子类，所以采用与调用原类相同的方式调用代理类。

### h.切面（Aspect）

漆面由切点和增强（引介）组成，它既包括横切逻辑的定义，也包括连接点的定义。Spring AOP就是负责实施切面的框架，它将切面所定义的横切逻辑织入切面所指定的连接点中。AOP的工作重心在于如何将增强应用于目标对象的连接点上。包括两项工作：第一，如何通过切点和增强定位到连接点上；第二，如何在增强中编写切面的代码。

## 3.AOP的实现者

### a.AspectJ

AspectJ是语言级的AOP实现，2001年由Xerox PAPC的AOP小组发布。AspectJ拓展了Java语言，定义了AOP语法，能够在编译器提供横切代码的织入，所以它有一个专门的编译器用来生成遵守Java字节编码规范的Class文件。

### b.AspectWerkz

现在Aspect和AspectWerkz项目已经合并。

### c.JBoss AOP

主页：http://www.jboss.org/products/aop

### d.Spring AOP

Spring AOP使用纯Java实现，它不需要专门的编译过程，也不需要特殊的类加载器，它在运行期间提供代理方式想目标类织入增强代码。Spring并不尝试提供最完整的AOP实现，它侧重于提供一种和Spring IoC容器整合的AOP实现。在Spring中可以无缝地将Spring AOP、IoC和AspectJ整合在一起。

## 4.代理知识

### a.JDK动态代理

自Java1.3以后，Java提供了动态代理技术，允许开发者在运行期间创建接口的代理实例。JDK动态代理主要涉及java.lang.reflect中的两个类：Proxy和InvocationHandler。其中InvocationHandler是一个接口，可以通过实现该接口定义横切逻辑，并通过反射机制调用目标类的代码，动态地将横切逻辑和业务逻辑编织在一起。Proxy利用InvocationHandler动态创建一个符合某一接口的实例，生成目标类的代理对象。实例如下：

* 首先将业务逻辑与监视代码分离开来，cityDao只负责具体的业务逻辑，如下：

  ```java
  package cn.aymeng.daoImpl;
  
  import cn.aymeng.dao.CityDao;
  
  public class CityDaoImpl implements CityDao {
      @Override
      public void deleteCity(int id) {
          System.out.println("模拟删除了一条记录：" + id);
      }
  }
  ```

* 创建一个PerformanceHandler，用来编写监视逻辑，它需要实现InvocationHandler接口，代码如下：

  ```java
  package cn.aymeng.proxy;
  
  import java.lang.reflect.InvocationHandler;
  import java.lang.reflect.Method;
  
  public class PerformanceHandler implements InvocationHandler {
      private final Object target;
  
      public PerformanceHandler(Object target) {
          this.target = target;
      }
  
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  
          System.out.println("删除记录前");
          Object obj = method.invoke(target, args);
          System.out.println("删除记录后");
          return obj;
      }
  }
  ```

  invoke()方法中编写性能监视的横切代码，method.invoke()语句通过Java反射机制间接调用目标对象的方法，这样InvocationHandler的invoke()方法就将横切逻辑代码和业务类方法的业务逻辑代码就编织到一起。

  该类实现InvocationHandler接口，该接口定义了一个invoke方法，其中，proxy是最终生成的代理实例，一般不会用到；method是被代理目标实例的某个具体方法，通过它可以发起目标实例方法的反射调用；args是被代理实例某个方法的入参，在方法反射调用时使用。

* 编写测试类，代码如下：

  ```java
  public class JdkProxyTest {
  
      @Test
      public void proxyTest() {
          CityDao target = new CityDaoImpl();
          PerformanceHandler handler = new PerformanceHandler(target);
          CityDao proxy = (CityDao) Proxy.newProxyInstance(
                  target.getClass().getClassLoader(), target.getClass().getInterfaces(), handler);
          proxy.deleteCity(201100);
      }
  }
  ```

  运行结果如下：

  ```javascript
  删除记录前
  模拟删除了一条记录：201100
  删除记录后
  
  Process finished with exit code 0
  ```

  上面的代码完成了业务类代码和横切代码的编织工作并生成了代理实例。首先创建个目标类，然后让PreformanceHandler将横切逻辑编织到target实例中，然后通过Proxy的newProxyInstance()静态方法为编织了业务类逻辑和横切逻辑的handler创建了一个符合CityDao接口的代理实例。该方法的第一个入参为类加载器，第二个入参为创建代理实例所需实现的一组接口，第三个入参是整合了业务逻辑和横切逻辑的编织器对象。

### b.CGLib动态代理

JDK创建代理有一个限制，即它只能为接口创建代理实例，可以从Proxy的newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler in)中可以看出，第二个入参interfaces需要代理实例实现的接口列表。

CGLib采用底层的字节码技术，可以为一个类创建子类，在子类中采用方法拦截的技术拦截所有父类方法的调用并顺势织入横街逻辑。

## 5.AOP联盟

AOP联盟是众多开源AOP项目的联合组织，该组织的目的是为了制定一套规范描述AOP的标准，定义标准的AOP标准，以便各种遵守标准的具体实现可以互相调用。

## 6.Spring增强类

Spring使用增强类定义横切逻辑，由于Spring只支持方法连接点，增强还包括在方法的哪一点加入横切代码的方位信息，所以增强既包括横切逻辑，又包含部分连接点的信息。

AOP联盟为增强定义了org.aopalliance.aop.Advice接口，Spring支持5中类型的增强，如下：

* 前置增强：org.springframework.aop.BeforeAdvice代表前置增强。因为Spring只支持方法级的增强，所有MethodBeforeAdvice是目前可用的前置增强，表示在目标方法执行前执行的实施增强。
* 后置增强：org.springframework.aop.AfterReturningAdvice代表后置增强，表示在目标方法执行后实施增强。
* 环绕增强：org.aopalliance.intercept.MethodInterceptor代表环绕增强，表示在目标方法执行前后实施增强。
* 异常抛出增强：org.springframework.aop.ThrowsAdvice：代表抛出异常增强，表示在目标方法抛出异常后实施增强。
* 引介增强：org.springframework.IntroductionInterceptor代表引介增强，表示在目标类中添加一些新的方法和属性。

这些增强接口都有一些方法，通过实现这些接口方法，并在接口方法中定义横切逻辑，就可以将它们织入目标类方法的相应连接点位置。

### a.前置增强

下面使用一个简单的例子演示增强（剑网三）：

* 定义一个奶妈接口：

  ```java
  package cn.aymeng.jx3;
  
  public interface WetNurse{
      void talk(String name);
  }
  ```

* 创建一个奶花类：

  ```java
  package cn.aymeng.jx3;
  
  public class LiJingYiDao implements WetNurse{
      @Override
      public void talk(String name) {
          System.out.println("奶花正在对" + name + "施展提针！");
      }
  }
  ```

* 新建一个增强逻辑：

  ```java
  package cn.aymeng.proxy;
  
  import org.springframework.aop.MethodBeforeAdvice;
  
  import java.lang.reflect.Method;
  
  public class TianCeAdvice implements MethodBeforeAdvice {
      @Override
      public void before(Method method, Object[] args, Object target) throws Throwable {
          System.out.println("小奶花一个人呀！");
      }
  }
  ```

  BeforeAdvice是Spring前置增强的接口，方法前置增强的MethodBeforeAdvice是其子接口。Spring目前只提供方法调用的前置增强。MethodBeforeAdvice仅定义了一个唯一的方法：before(Method method, Object[] args, Object obj) throws Throwable。其中，Method是目标类的方法，args为目标类方法的入参，obj是目标类实例，当该方法发生异常时，将阻止目标类方法的执行。

* 测试增强织入：

  ```java
  public class JX3Test {
      @Test
      public void jx3AdviceTest(){
          // 创建一个目标对象
          WetNurse target = new LiJingYiDao();
          // 创建一个增强
          BeforeAdvice advice = new TianCeAdvice();
          // 创建代理工厂对象
          ProxyFactory pf = new ProxyFactory();
          // 创建一个切面，横切逻辑+业务逻辑
          pf.setTarget(target);
          pf.addAdvice(advice);
          // 生成一个代理对象
          WetNurse nh = (WetNurse) pf.getProxy();
          nh.talk("自己");
      }
  }
  ```

  运行结果为：

  ```javascript
  小奶花一个人呀！
  奶花正在对自己施展提针！
  
  Process finished with exit code 0
  ```

* 在测试中，使用ProxyFactory代理工厂将TianCeAdvice的增强织入到目标类LiJingYiDao中。ProxyFactory就是使用JDK或CGLbib动态代理技术将增强应用到目标类中的。

  Spring定义了org.springframework.aop.framework.AopProxy接口，并提供了两个Final类型的实现类，如下：

  [![gf0irT.png](https://z3.ax1x.com/2021/05/18/gf0irT.png)](https://imgtu.com/i/gf0irT)

  其中CglibAopProxy使用CGLib动态代理技术创建代理，而JdkDynamicAopProxy使用JDK动态代理技术创建代理。如果通过ProxyFactory的setInterfaces(Class[] interfaces)方法指定目标接口进行代理，则Proxy使用JdkDynamicAopProxy；如果是针对类的代理，则使用CglibAopProxy。还可以通过ProxyFactory的setOptimize(true)方法让ProxyFactory启动优化代理方式，这样，针对接口的代理也会使用CglibAopProxy。

  ProxyFactory通过addAdvice(Advice)方法添加一个增强，还可以使用该方法添加多个增强。多个增强形成一个增强链，他们的调用顺序和添加顺序一致，可以通过addAdvice(int, Advice)方法将增强添加到增强链的具体位置。

* 以XML的方式在Spring中配置增强

  配置方式如下：

  ```xml
  <bean id="tianCeAdvice" class="cn.aymeng.proxy.TianCeAdvice"/>
  <bean id="target" class="cn.aymeng.jx3.LiJingYiDao"/>
  <bean id="proxy" class="org.springframework.aop.framework.ProxyFactoryBean">
      <property name="proxyInterfaces" value="cn.aymeng.jx3.WetNurse"/>
      <property name="interceptorNames" value="tianCeAdvice"/>
      <property name="target" ref="target"/>
  </bean>
  ```

  ProxyFactoryBean是FactoryBean接口的实现类，它的内部使用ProxyFactory来完成这项工作，下面是几个常用的可选配置属性：

  * target：代理的目标对象
  * proxyInterfaces：代理所要实现的接口，可以是多个接口，该属性还有一个别名属性interfaces。
  * interceptorNames：需要织入目标对象的Bean列表，采用Bean的名称指定，这些Bean必须实现了org.aopallicance.MethodInterceptor或org.springframework.aop.Advisor的Bean。
  * singleton：返回的代理是否是单实例，默认为单实例。
  * optimize：当设置为true时，强制使用CGLib动态代理。对于singleton的代理，推荐使用CGLib，其它作用域，推荐使用JDK动态代理。
  * proxyTargetClass：是否对类进行代理（而不是对接口进行代理）。当设置为true时，使用CGLib动态代理。

  使用spring容器进行代理，如下

  ```java
  @Test
  public void xmlJx3AdviceTest(){
      ApplicationContext ac = new ClassPathXmlApplicationContext("classpath:springApplication.xml");
      WetNurse proxy = ac.getBean("proxy", WetNurse.class);
      proxy.talk("自己");
  }
  ```

  ***Tips：interceptorNames是String[]类型的，它接受增强Bean的名称而非增强Bean的实例。这是因为ProxyBeanFactory内部在生成代理类时，需要使用增强Bean的类，而非增强Bean的实例，以织入增强类中所写的横切逻辑代码，因而可以说增强是类级别的。***

### c.后置增强

后置增强在目标类方法调用后执行。代码如下：

```java
public class TianCeAfterAdvice implements AfterReturningAdvice {
    @Override
    public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
        System.out.println("小奶花一个人呀！");
    }
}
```

通过实现AfterReturningAdvice来定义后置增强逻辑，AfterReturningAdvice接口也仅定义了唯一的方法afterReturning(Object returnObj, Method method, Object[] args, Object obj) throws Throwable。其中returenObj为目标实例方法返回的结果；method为目标类的方法；args为目标实例方法的入参；而obj为目标类实例。假设在后置增强中抛出异常，如果该异常是目标方法声明的异常，则该异常归并到目标方法中；如果不是目标方法所声明的异常，则Spring将其转换为运行期异常抛出。XML配置如下：

```xml
<bean id="tianCeAfterAdvice" class="cn.aymeng.proxy.TianCeAfterAdvice"/>
<bean id="target" class="cn.aymeng.jx3.LiJingYiDao"/>
<bean id="proxy" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces" value="cn.aymeng.jx3.WetNurse"/>
    <property name="interceptorNames" value="tianCeAfterAdvice"/>
    <property name="target" ref="target"/>
</bean>
```

测试方法如下：

```java
@Test
public void xmlJx3AfterAdviceTest(){
    ApplicationContext ac = new ClassPathXmlApplicationContext("classpath:springApplication.xml");
    WetNurse proxy = ac.getBean("proxy", WetNurse.class);
    proxy.talk("自己");
}
```

### c.环绕通知

环绕通知允许在目标类方法调用前后织入横切逻辑，它综合实现了前置、后置增强的功能，代码如下：

```java
public class TianCeSurroundAdvice implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        Object[] args = invocation.getArguments();
        String name = (String) args[0];
        System.out.println("小奶花在奶谁呀？哦，是在奶" + name + "呀！");
        Object proceed = invocation.proceed();
        System.out.println("奶不上的0.0");
        return proceed;
    }
}
```

Spring直接使用AOP联盟所定义的MethodInterceptor作为环绕增强的接口。该接口有唯一以的接口方法Object invoke(MethodInvacation invocation) throws Throwable。MethodInvocation不但封装了目标方法及其参数组，还封装了目标方法所在的实例对象。其继承体系为：MethodInvocation-> Invocation -> Joinpoint。

* org.aopalliance.intercept.MethodInvocation

  Method getMethod()

* org.aopalliance.intercept.Invocation 

  Object[] getArguments();

* org.aopalliance.intercept.Joinpoint

  Object proceed() throws Throwable;

  Object getThis();

  AccessibleObject getStaticPart();

XML配置：

```xml
<bean id="tianCeSurroundAdvice" class="cn.aymeng.advice.TianCeSurroundAdvice"/>
<bean id="target" class="cn.aymeng.jx3.LiJingYiDao"/>
<bean id="proxy" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces" value="cn.aymeng.jx3.WetNurse"/>
    <property name="interceptorNames" value="tianCeSurroundAdvice"/>
    <property name="target" ref="target"/>
</bean>
```

测试代码：

```java
@Test
public void xmlJx3SurroundAdviceTest(){
    ApplicationContext ac = new ClassPathXmlApplicationContext("classpath:springApplication.xml");
    WetNurse proxy = ac.getBean("proxy", WetNurse.class);
    proxy.talk("气纯");
}
```

运行结果：

```javascript
小奶花在奶谁呀？哦，是在奶气纯呀！
奶花正在对气纯施展提针！
奶不上的0.0

Process finished with exit code 0
```

### d.异常抛出增强

异常抛出异常最适合的应用场景是事务管理，当参与事务的某个DAO发生异常时，事务管理就必须回滚事务。下面模拟个业务异常，WetNurseService类：

```java
public class WetNurseService {
    public void cureSect(){
        throw new RuntimeException("运行时期异常");
    }
    public void buffSect() throws SQLException {
        throw new SQLException("数据库异常");
    }
}
```

下面创建个异常抛出增强类TransactionManager：

```java
public class TransactionManager implements ThrowsAdvice {

    public void afterThrowing(Method method, Object[] args, Object target, Exception ex) throws Exception{
        System.out.println("--------");
        System.out.println("方法：" + method.getName());
        System.out.println("抛出了异常：" + ex.getMessage());
    }
}
```

ThrowsAdvice异常抛出增强接口没有定义任何方法，它是一个标签接口，在运行期Spring使用反射机制自行判断，必须采用一下方法签名形式定义异常抛出的增强方法：void afterThrowing(Method method, Object[] args, Object target, Throwable th);方法名必须为afterThrowing，方法入参规定如下：method，args，target是可选的，但是最后一个参数必须有且是Throwable或其子类，可以在同一个异常抛出增强中定义多个afterThrowing()方法，当目标类方法抛出异常时，Spring会自动选用最匹配的增强方法。

### e.引介增强

引介增强是一种比较特殊的增强类型，它不是在目标方法周围织入增强，而是为了目标类创建新的方法和属性，所以引介增强的连接点是类级别的，而非方法级别的。通过引介增强，可以为目标类添加一个接口的实现，即原来目标类未实现某个接口，通过引介增强可以为目标类创建实现某接口的代理。

Spring定了以引介增强接口IntroductionInterceptor，该接口没有定义任何方法，Spring为该接口提供了DelegatingIntroductionInterceptor实现类。一般情况下，提供拓展该实现类定义自己的引介增强类。

引介增强的配置与一般的配置有较大的区别，首先需要指定引介增强所实现的接口。其次，由于只能通过为目标类创建子类的方式生成引介增强的代理，所以需要强制使用CGLib，就必须将proxyTartgeClass设置为true，否则就会报错。

## 7.创建切面

在介绍切面增强时，会有一个问题：增强被织入目标类的所有方法中。假设需要有选择地织入目标类的某些特定方法中，就需要使用切点目标连接点的定位。增强提供了连接点信息，如织入到方法前面、后面等，而切点进一步描述了织入哪些类的哪些方法上。

Spring通过org.springframework.aop.Pointcut接口描述切点，Poincut有ClassFilter和MethodMatcher构成，它通过ClassFilter定位到某些特定类上，通过MethodMatcher定位到某些特定方法上。

* org.springframework.aop.Pointcut

  getClassFilter();

  getMethodMatcher();

* org.springframework.aop.ClassFilter

  matches(Class clazz);

* org.springframework.aop.MethodMatcher

  matches(Method m, Class targetClass);

  isRuntime();

  matches(Method m, Class targetClass, Object[] args)

Spring支持两种方法匹配器：静态方法匹配器和动态方法匹配器。静态方法匹配器，仅对方法签名（包括方法名和入参类型及顺序）进行匹配；而动态方法匹配器会在运行期间检查方法入参的值。静态匹配仅会判别一次，而动态匹配因为每次调用方法的入参都可能不一样，所以每次调用方法都必须判断，因此，动态匹配对性能的影响很大。一般情况下，动态匹配不常使用。方法匹配器的类型由isRuntime()方法的返回值决定，返回true表示是动态方法匹配器，返回false表示静态方法匹配器。

### a.切点类型

Spring提供了6中切点类型：

* 静态方法切点：org.springframework.aop.support.StaticMethodMatcherPointcut是静态方法切点的抽象基类，默认情况下它匹配所有的类。StaticMethodMatcherPointcut包括两个主要的子类，分别是NameMatchMethodPointcut和AbstractRegexpMethodPointcut，前者提供简单字符串匹配方法签名，而后者使用正则表达式匹配方法签名。
* 动态方法切点：org.springframework.aop.support.DynamicMethodMatcherPointcut是动态方法切点的抽象基类，默认情况下它匹配所有的类。
* 注解切点：org.springframework.sop.support.annotation.AnnotationMatchingPointcut实现了表示注解切点。使用AnnotationMatchingPoint支持在Bean中直接通过Java5.0注解标签定义的切点。
* 表达式切点：org.springframework.aop.support.ExpressionPointcut接口主要是为了支持AspectJ切点表达式语法而定义的接口。
* 流程切点：org.springframework.aop.support.ControlFlowPointcut实现类表示控制流程切点。
* 复合切点：org.springframework.aop.support.ComposablePointcut实现类是为创建多个切点而提供的方便操作类。它所有的方法都返回ComposablePointcut类，这样就可以使用链接表达式对切点进行操作。

### b.切面类型

由于增强既包括横切代码，又包括部分连接点信息（方法前，方法后主方位信息），所以仅通过增强类就能生成一个切面。但切点仅代表目标连接点的部分信息（类和方法的定位），所有仅有切点无法制作出一个切面，必须结合增强才能制作出切面。Spring使用org.springframework.aop.Advisor接口表示切面的概念，一个切面同时包含横切代码和连接点信息。切面可以分为三类：一般切面、切点切面和引介切面：

* Advisor：代表一般切面，仅包含一个Advice。因为Advice包含了横切代码和连接点信息，所以Advice本身就是一个简单的切面，只不过它代表的横切的连接点是所有目标类的所有方法，因为这个切面太宽泛，所以一般不会直接使用。
* PointcutAdvisor：代表具有切点的切面，包含Advice和Pointcut两个类，这样就可以通过类、方法名及方法方位等信息灵活地定义切面的连接点，通过更具适用性的切面。
* IntriductionAdvisor：代表引介切面。引介切面是对应引介增强的特殊的切面，它应用于类层面上，所以引介切点适用ClassFilter进行定义。

PointcutAdvice主要有六个具体的实现类：

* DefaultPointcutAdvisor：最常见的切面类型，他可以通过任意Pointcut和Advice定义一个切面，唯一不支持的是引介切面类型，一般可以通过拓展该类实现自定义的切面。
* NameMatchMethodPointcutAdvisor：通过该类可以定义按方法名定义切点的切面。
* RegexpMethodPointcutAdvisor：对于按正则表达式匹配方法名进行切点定义的切面，可以通过拓展该实现类进行操作。RegexpMethodPointcutAdvisor允许用户以正则表达式模式串定义方法匹配的切点，其内部通过JdkRegexpMethodPointcut构造出正则表达式方法名切点。
* StaticMethodMatcherPointcutAdvisor：静态匹配器切点定义的切面，默认情况下匹配所有的目标类。
* AspectJExpessionPointcutAdvisor：用于AspectJ切点表达式定义切点的切面。
* AspectJPointcutAdvisor：用于AspectJ语法定义切点的切面。

这些Advisor的实现类都可以在Pointcut中找到对应物，实际上，他们都是通过拓展对应的Pointcut实现类并实现PointcutAdvisor接口进行定义的。

### c.静态普通方法名匹配切面

StaticMethodMatcherPointcutAdvisor代表一个静态方法匹配切面，它通过StaticMethodMatcherPointcut来定义切点，并通过类过滤和方法名来匹配所定义的切点。如下面的例子：

```java
public class WaiterService {
    public void greetTo(String name){
        System.out.println("Waiter greet to " + name);
    }
    public void serveTo(String name){
        System.out.println("Waiter serving to " + name);
    }
}
```

```java
public class SellerService {
    public void greetTo(String name){
        System.out.println("Seller greet to " + name);
    }
}
```

WaiterService有两个方法，分别是greetTo()和serveTo()，SellerService有一个greetTo()方法。现在，通过StaticMethodMatcherPointcutAdvisor定义一个切面，在greetTo方法调用前织入一个增强，如下：

```java
public class GreetingAdvisor extends StaticMethodMatcherPointcut {
    @Override
    public boolean matches(Method method, Class<?> aClass) {
        return "greetTo".equals(method.getName());
    }

    public ClassFilter getClassFilter(){
        return new ClassFilter() {
            @Override
            public boolean matches(Class<?> aClass) {
                return WaiterService.class.isAssignableFrom(aClass);
            }
        };
    }
}
```

StaticMethodMatcherPointcutAdvisor抽象类唯一需要定义的是matches()方法。在默认情况下，该切面匹配所有的类，这里通过覆盖getClassFilter()方法，让它仅匹配Waiter类极其子类，Advisor还需要一个增强类的配合，下面定义一个前置代码：

```java
public class GreetingBeforeAdvice implements MethodBeforeAdvice {

    @Override
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        System.out.println("切点为：" + o.getClass().getName() + "." + method.getName());
        System.out.println("How are you! Mr." + objects[0]);
    }
}
```

接下来使用Spring配置XML来定义切面：

```xml
<bean id="waiterTarget" class="cn.aymeng.service.WaiterService"/>
<bean id="sellerTarget" class="cn.aymeng.service.SellerService"/>
<bean id="greetAdvice" class="cn.aymeng.advisor.GreetingBeforeAdvice"/>
<!--向切面织入一个增强-->
<bean id="greetingAdvisor" class="cn.aymeng.advisor.GreetingAdvisor">
    <property name="advice" ref="greetAdvice"/>
</bean>
<!--定义一个父类，配置公共信息-->
<bean id="parent" abstract="true" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="interceptorNames" value="greetingAdvisor"/>
    <property name="proxyTargetClass" value="true"/>
</bean>
<!--生成代理-->
<bean id="waiter" parent="parent">
    <property name="target" ref="waiterTarget"/>
</bean>
<bean id="seller" parent="parent">
    <property name="target" ref="sellerTarget"/>
</bean>
```

StaticMethodMatcherPointcutAdvisor除了具有advice属性外，还有另外两个属性：

* classFilter：类匹配过滤器
* order：切面织入时的顺序

下面编写测试方法：

```java
public class SpringPointcutTest {

    @Test
    public void staticPointcutTest(){
        ApplicationContext ac = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
        WaiterService waiter = ac.getBean("waiter", WaiterService.class);
        waiter.greetTo("John");
        waiter.serveTo("John");
        SellerService seller = ac.getBean("seller", SellerService.class);
        seller.greetTo("John");
    }
}
```

运行结果如下：

```javascript
切点为：cn.aymeng.service.WaiterService.greetTo
How are you! Mr.John
Waiter greet to John
Waiter serving to John
Seller greet to John

Process finished with exit code 0
```

可见切面只织入了WaiterService的greetTo()方法调用前的连接点上，WaiterService的severTo()和SellerService的greetTo()方法没有织入切面。

### d.静态正则表达式方法匹配切面

在StaticMethodMatcherPointcutAdvisor中，仅能通过方法名定义切点，这种方式不够灵活。假设目标有多个方法，且它们都满足一定的命名规范，使用正则表达式进行匹配描述就灵活多了。StaticMethodMatcherPointcutAdvisor是正则表达式方法匹配的切面实现类，该类已是功能齐备的实现类，一般情况下无需拓展该类，下面直接使用StaticMethodMatcherPointcutAdvisor通过配置的方式定义一个切面：

```xml
<bean id="regexpAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    <property name="advice" ref="greetAdvice"/>
    <property name="patterns">
        <list>
            <value>.*greetTo.*</value>
        </list>
    </property>
</bean>
```

RegexpMethodPointcutAdvisor除了有patterns属性外，还有两个属性：

* pattern：如果只有一个匹配模式串，可以使用该属性。patterns属性用于定义多个匹配模式串。
* order：切面在织入时对应的顺序。

正则表达式语法：

| 符号  |              说明              | 符号  |                            说明                             |
| :---: | :----------------------------: | :---: | :---------------------------------------------------------: |
|   .   | 匹配除换行符外的所有单个的字符 |  \*   |                  匹配\*前面的字符0次或n次                   |
|   +   |    匹配+前面的字符1次或n次     |   ^   |                 表示匹配的字符必须在最前边                  |
|   $   |        匹配最末尾的字符        |  ？   |                   匹配?前面的字符0次或1次                   |
| x\|y  |            匹配x或y            | [xyz] | 字符列表，匹配列表中的任意字符<br />可以通过‘-’指出一个范围 |
|  {n}  |    n是正整数匹配前面n个字符    | {n,}  |                     匹配前面至少n个字符                     |
| {n,m} |  匹配至少n个最多m个前面的字符  |       |                                                             |

### e.动态切面

以后再补充

### f.流程切面

以后再补充

### g.复合切点切面

以后再补充

### h.引介切面

以后再补充

## 8.自动创建代理

### 1.BeanNameAutoProxyCreater

以后再补充

### 2.DefaultAdvisorAutoProxyCreater

以后再补充

# 二、基于@AspectJ的AOP

前面分别使用Pointcut和Advice接口描述切点和增强，并用Advisor整合二者描述切面，@AspectJ则采用注解来描述切点、增强，二者只是表达方式不同，描述内容的本质是完全相同的。

Spring在处理@AspectJ注解表达式时，需要将Spring的asm模块添加到类路径中。asm是轻量级的字节码处理框架，因为Java的反射机制无法获取入参名，Spring就利用asm处理@AspectJ中所描述的方法入参名。此外，Spring采用AspectJ提供的@AspectJ注解类库及相应的解析类库，需要在pom.xml文件中添加aspectj.weaver和aspectj.tools类包的依赖。下面是一个简单的例子：

下面定义一个NaiveWaiter类：

```java
public class NaiveWaiter implements Waiter{
    @Override
    public void greetTo(String clientName) {
        System.out.println("NaiveWaiter.greetTo");
    }

    @Override
    public void serveTo(String clientName) {
        System.out.println("NaiveWaiter.serveTo");
    }
}
```

下面使用@AspectJ注解定义一个切面：

```java
// 通过注解将类标识为切面
@Aspect
public class PreGreetingAdvice {
    // 定义切点和增强类型
    @Before("execution(* greetTo(..))")
    public void beforeGreeting(){ // 定义横切逻辑
        System.out.println("PreGreetingAdvice.beforeGreeting");
    }
}
```

首先，在PreGreetingAdvvice类定义处标注了@AspectJ注解，这样，第三方处理程序就可以通过类是否拥有@AspectJ注解判断其是否为一个切面。然后在beforeGreeting()方法定义处标注了@Before注解，并为该注解通过了成员值“execution(* greetTo(..))”，@Before注解表示该增强是前置增强，而成员是一个@AspectJ切点表达式。它的意思是在目标类的greetTo()方法上织入增强，greetTo()方法可以是带任意的入参和任意的返回值。

下面通过AspectJProxyFactory生成一个代理：

```java
public class AspectJTest {

    @Test
    public void aspectJProxy(){
        Waiter waiter = new NaiveWaiter();
        AspectJProxyFactory factory = new AspectJProxyFactory();
        // 设置目标对象
        factory.setTarget(waiter);
        // 设置切面类
        factory.addAspect(PreGreetingAdvice.class);
        // 生成代理对象
        Waiter proxy = factory.getProxy();
        proxy.greetTo("John");
        proxy.serveTo("John");
    }
}
```

通过XML配置使用@AspectJ切面：

* 使用自动代理器创建：

  ```xml
  <!--目标Bean-->
  <bean id="waiter" class="cn.aymeng.service.NaiveWaiter"/>
  <!--切面Bean-->
  <bean class="cn.aymeng.aspectj.PreGreetingAdvice"/>
  <!--自动代理创建器，自动将@AspectJ注解切面织入到目标Bean中-->
  <bean class="org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator"/>
  ```

* 使用基于Schema的aop命名空间配置：

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:aop="http://www.springframework.org/schema/aop"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/aop
          https://www.springframework.org/schema/aop/spring-aop.xsd">
      
      <!--基于@AspectJ切面的驱动器-->
      <aop:aspectj-autoproxy />
      <bean id="waiter" class="cn.aymeng.service.NaiveWaiter"/>
      <bean class="cn.aymeng.aspectj.PreGreetingAdvice"/> 
  </beans>
  ```

  首先在配置文件中引入aop的命名空间，然后通过aop命名空间的\<aop:aspectj-sutoproxy/\>自动为Spring容器中那些匹配@AspectJ切面的Bean创建代理，完成切面织入。Spring内部依旧采用AnnotationAwareAspectJAutoProxyCreator进行自动创建代理的工作。\<aop:aspectj-sutoproxy/\>有一个proxy-target-class属性，默认为false，表示使用JDK动态代理织入增强，当配置为true时，表示使用CGLib动态代理织入增强，不过即使设置为false，如果目标类没有声明接口，则Spring将自动使用CGLib动态代理。

下面测试方法：

```java
public class AspectJTest {
    @Test
    public void aopAspectJTest(){
        ApplicationContext ac = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
        Waiter waiter = ac.getBean("waiter", Waiter.class);
        waiter.greetTo("John");
        waiter.serveTo("John");
    }
}
```

运行结果：

```javascript
PreGreetingAdvice.beforeGreeting
NaiveWaiter.greetTo
NaiveWaiter.serveTo

Process finished with exit code 0
```

## 1.@AspectJ语法基础

### a.切点表达式函数

AspectJ5.0的切点表达式由关键字和操作参数组成，如execution(* greetTo(..))，execution为关键字，而* greetTo(..)为操作参数。这里execution代表类执行某一方法，"* greetTo(..)"描述目标方法的匹配模式串，二者联合起来表示目标类方法的连接点。

Spring支持9个@AspectJ切点表达式函数，它们用不同的方式描述目标类的连接点。根据描述对象的不同，可以大致分为4种类型。

* 方法切点函数：通过描述目标类方法的信息定义连接点。
* 方法入参切点函数：通过描述目标类方法入参的信息定义连接点。
* 目标类切点函数：通过描述目标类类型的信息定义连接点。
* 代理类切点函数：通过描述目标类的代理类的信息定义连接点。

切点函数如下：

|       类别       |     函数      |      入参      |                             说明                             |
| :--------------: | :-----------: | :------------: | :----------------------------------------------------------: |
|   方法切点函数   |  execution()  | 方法匹配模式串 | 表示满足某一匹配模式的所有目标类方法连接点。如execution(* greetTo(..))表示所有目标类中的greetTo()方法 |
|   方法切点函数   | @annotation() |  方法注解类名  | 表示标注了特定注解的目标类方法连接点。如@annotation(com.aymeng.NeetTest)表示任何标注了@NeedTest注解的目标类方法 |
| 方法入参切点函数 |    args()     |      类名      | 通过判别目标类方法运行时入参对象的类型定义指定连接点。如args(com.aymeng.Waiter)表示所有有且仅有一个按类型匹配于Waiter入参的方法 |
| 方法入参切点函数 |    @args()    |  类型注解类名  | 通过判别目标类方法运行时入参对象的类是否标注特定注解来指定连接点，如@args(com.aymeng.Monitorable)表示任何这样的一个目标方法：它有一个入参且入参对象的类标注@Monitorable注解 |
|  目标类切点函数  |   within()    |   类名匹配串   | 表示特定域下的所有连接点。如within(cn.aymeng.service.*)表示service包下的所有连接点，即包中所有类的所有方法。within(cn.aymeng.service.*Service)表示service包下所有以Service结尾的类的所有连接点 |
|  目标类切点函数  |   target()    |      类名      | 假如目标类按类型匹配于指定类，则目标类的所有连接点匹配这个切点。如通过target(cn.aymeng.Waiter)定义的切点，Waiter及其实现类的所有连接点都匹配这个切点。 |
|  目标类切点函数  |   @within()   |  类型注解类名  | 假如目标类按类型匹配于某个类A，且类A标注了特定注解，则目标类的所有连接点匹配这个切点。如@within(cn.aymeng.Monitorable)定义的切点，假如Waiter标注了@Monitorable注解，则Waiter及其实现类的所有连接点都匹配这个切点 |
|  目标类切点函数  |    @target    |  类型注解类名  | 假如目标类标注了特定注解，则目标类的所有连接点都匹配该切点，如@target(cn.aymeng.Monitorable)，假如NaiveWaiter标注了@Monitorable，则NaiveWaiter的所有连接点都匹配这个切点 |
|  代理类切点函数  |    this()     |      类名      | 代理类按类型匹配于指定类，则被代理的目标类的所有连接点都匹配该切点。 |

### b.在函数入参中使用通配符

以后再补充

### c.逻辑运算符

以后再补充

### d.不同增强类型

* @Before

  前置增强，相当于BeforeAdvice。它拥有两个成员

  * value：该成员用于定义切点。
  * argNames：由于无法通过Java反射机制获取方法入参名，所有如果在Java编译时未启用调试信息，或者需要在运行期解析切点，就必须通过这个成员指定注解所标注增强方法的参数名，多个参数名用逗号分隔。

* @AfterReturning

  后置增强，相当于AfterReturningAdvice。它拥有4个成员。

  * value：如上
  * pointcut：表示切点的信息。如果显示指定pointcut值，那么它将覆盖value的设置值，可以将pointcut成员看作value的同义词。
  * returning：将目标对象方法的返回值绑定给增强的方法。
  * argNames：如上

* @Around

  环绕增强，相当于MethodInterceptor，它有两个成员

  * value：如上
  * argNames：如上

* AfterThrowing

  异常抛出增强，相当于ThrowsAdvice，它有4个成员

  * value：如上
  * pointcut：如上
  * throwing：将抛出的异常绑定到增强方法中。
  * argNames：如上

* @After

  Final增强，不管是抛出异常还是正常退出，该增强都会得到执行。该增强没有对应的增强接口，一般用于释放资源，它有两个成员

  * value：如上
  * argNames：如上

* @DeclareParents

  引介增强，相当于IntroductionInterceptor，它有两个成员

  * value：如上
  * defaultImpl：默认的接口实现类