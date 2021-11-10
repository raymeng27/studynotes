# lamba表达式

> lambda表达式是一个可传递的代码块，可以在以后执行一次或多次。到目前为止，在Java中传递一个代码段并不容易，不能直接传递代码段。Java是一种面向对象语言，所以必须构造一个对象，这个对象的类需要有一个方法包含所需的代码。

## 1.表达式形式

* 参数，箭头（->）以及一个表达式。如果代码要完成的计算无法放在一个表达式中，可以像写方法一样，把代码放在{}中。

  ```java
  (String first, String second) -> {if first.lenggth() < second.length return -1;
                                   else return 0;}
  ```

* 即使lambda表达式没有参数，仍然要提高空括号，就像无参方法一样。

  ```java
  () -> System.out.println("lambda");
  ```

* 如果可以推导出一个lambda表达式的参数类型，则可以忽略其类型。

  ```java
  Comparator<String> comp = (first, second) -> first.length() - second.length();
  ```

* 如果方法只有一个参数，而且这个参数的类型可以推导得出，那么甚至还可以省略小括号。

  ```java
  ActionListener listener = event -> System.out.println("The time is " + Instant.ofEpochMilli(event.getWhen()));
  ```

tips：如果一个lambda表达式只在某些分支返回一个值，而另外一些分支不返回值，这是不合法的。

## 2.函数式接口

Java中有很多封装代码块的接口，如ActionListener或Comparator。lambda表达式与这些接口是兼容的。对于只有一个抽象方法的接口，需要这种接口的对象时，就可以提供一个lambda表达式。这种接口称为函数式接口。

## 3.方法引用

有时，lambda表达式涉及一个方法。例如，假设希望只要出现一个定时器事件就打印这个事件对象。可以调用：

```java
Timer timer = new Timer(1000, System.out.println(enent));
```

但是，如果直接把println方法传递到Timer构造器就更好了。做法如下：

```java
var timer = new Timer(1000,System.out::println);
```

表达式System.out.println是一个方法引用，它指示编译器生成一个函数式接口的实例，覆盖这个接口的抽象方法来调用给定的方法。要用::运算符分隔方法名与对象或类名。主要有三种情况：

1. object::instanceMethod
2. Class::instanceMethod
3. Class::staticMethod

在第一种情况下，方法引用等价于向方法传递参数的lambda表达式。对于System.out::println，对象是System.out，所以方法表达式等价于x -> Syste.out.println(x)。

对于第二种情况，第一个参数会成为方法的隐式参数。例如，String::compareToIgnoreCase等同于(x,y) -> x.compareToIgnoreCase(y)。

在第三种情况下，所有参数都传递到静态方法：Math::pow等价于(x,y) -> Math.pow(x,y)。

Tips：只有当lambda表达式的体只调用一个方法而不做其它操作时，才能把lambda表达式重写为方法引用。例如：s -> s.length() == 0;这里有一个方法调用，但是还有一个比较，所以这里不能使用方法引用。

## 4.构造器引用

构造器引用与方法引用很类似，只不过方法名为new。例如Person::new是Person构造器的一个引用。使用哪一个构造器要取决于上下文。假设有一个字符串列表。可以把它转换为一个Person对象数组，为此要在各个字符串上调用构造器，如下：

```java
ArrayList<String> names = new ArrayList<>();
Stream<Person> stream = names.stream().map(Person::new);
List<Person> people = stream.collect(Collectors.toList());
```

可以用数组类型建立构造器引用。例如，int[]::new是一个构造器引用，它有一个参数：即数组的长度。这等价于lambda表达式x -> new int[]。

Java有一个限制，无法构造泛型类型T的数组。数组构造器引用对于克服这个限制很有用。表达式new T(n)会产生错误，因为这会改为new Object(n)。假设我们需要一个Person对象数组。Stream接口有一个toArray方法可以返回Object数组：Object[] people = stream.toArray();这并不是期望所得。希望得到一个Person引用数组，而不是Object引用数组。流库构造器利用构造器引用解决了这个问题。可以把Person[]::new传入toArray方法：

```java
Person[] people = stream.toArray(Person::new);
```

## 5.变量作用域

例：

```java
public static void repeatMessage(String next, int delay){
    ActionListener listener = event -> {
        System.out.println(next);
        Toolkit.getDefaultToolkit().beep();
    }
    new Timer(delay, listener).start();
}
```

这个表达式有一个自由变量next。表达式lambda的数据结构必须存储自由变量的值，在这里就是字符串“Hello”。可以说它被lambda表达式捕获。lambda表达式可以捕获外围作用域中变量的值。在Java中，要确保所捕获的值是明确定义的，这里有一个限制。在lambda表达式中，只能引用值不会改变的变量。例如，下面这个例子是错误的：

```java
public static void countDown(int start, int delay){
    ActionListener listener = event -> {
        start++; //错误：不能改变变量
        System.out.println(start);
    }
    new Timer(delay, listener).start();
}
```

这个限制是有原因的。如果在lambda表达式中更改变量，并发执行多个动作时就会不安全。另外，如果在lambda表达式中引用一个变量，而这个变量可能在外部改变，这也是不合法的。

这里有一条规则：lambda表达式中捕获的变量实际上是事实最终变量。事实最终变量是指，这个变量初始化之后就不会再为它赋新值。lambda表达式的体与嵌套块有相同的作用域。这里同样适用命名冲突和遮蔽的有关规定。在lambda表达式中声明与一个局部变量同名的参数或局部变量是不合法的。
