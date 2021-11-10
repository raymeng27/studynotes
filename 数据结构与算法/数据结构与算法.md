# 数据结构与算法

## 1.Compareable接口

Compareable是排序接口。如果一个类实现了Compareable接口，就意味着该类支持排序。可以通过Collections.sort或者Arrays.sort进行排序。Compareable接口仅仅包含一个方法compareTo(T o)：

public int compareable(T o);

Compareable可以让实现它的类的对象进行比较，具体的规则是按照compareTo方法中的规则进行。这种顺序称为自然顺序。compareTo方法的返回值有三种：

1. e1.compareTo(e2) > 0 即 e1 > e2
2. e1.compareTo(e2) < 0 即 e1 < e2
3. e1.compareTo(e2) = 0 即 e1 = e2

例：实现Compareable接口的类

```java
public class Student implements Compareable<Student>{
    private int age;
    @Override
    public int compareTo(Student o){
        return this.age - o.age;
    }
}
```

测试类：

```java
public class Test{
    public static void main(String[] args){
        Student student1 = new Student(5);
        Student student2 = new Student(2);
        System.out.println(student1.conpareTo(student2));
    }
}
```

输出结果为：3

## 2.Comparator接口

Comparator则是在外部指定排序规则，然后作为排序策略传递给某些类，使用方法为：

* 创建一个Comparator接口的实现类，并且创建一个新的对象
* 在compare方法中针对自定义类写排序规则
* 将Comparator对象作为参数传递给排序类的某个方法
* 向排序类中添加compare方法中使用的自定义类

例：

```java
// 1.创建一个实现Comparator接口的对象
Comparator comparator = new Comparator(){
    @Override
    public int compare(Object o1, Object o2){
        if(o1 instanceof Student && o2 instanceof Student){
            Student s1 = (Student)o1;
            Student s2 = (Student)o2;
            return s1.age - s2.age;
        }
    }
}
// 2.将此对象作为形参传递给TreeSet的构造器中
TreeSet treeset = new TreeSet(comparator);
// 3.向TreeSet中添加步骤1中compare方法中设计类的对象
treeset.add(new Student(2));
treeset.add(new Student(3);
treeset.add(new Student(6));
treeset.add(new Student(1));
treeset.add(new Student(7));
```

# 一、简单排序

## 1.冒泡排序

冒泡排序是一种计算机科学领域的较简单的排序算法。

排序原理：

1. 比较相邻的元素。如果前一个元素比后一个元素大，就交换这两个元素的位置。
2. 对每一个相邻元素做同样的工作，从开始第一对元素直到结尾的最后一对元素，最终最后位置的元素就是最大值。

代码：

```java
public class Test {
    public static void main(String[] args) {
        int[] arrays = {2,7,1,5,9,3,0};
        bubbleSort(arrays);
    }
    public static void bubbleSort(int[] array){
        for (int i = array.length - 1; i > 0; i--) {
            for (int j = 0; j < i; j++) {
                if (array[j] > array[j+1]){
                    int temp = array[j];
                    array[j] = array[j+1];
                    array[j+1] = temp;
                }
            }
        }
        System.out.println(Arrays.toString(array));
    }
}
```

冒泡排序的时间复杂度分析：

冒泡排序使用了双层循环，其中内层循环的循环体是真正完成排序的代码。分析冒泡排序主要分析内层排序的执行次数，在最坏情况下（逆序），那么元素的比较次数为：

(N-1)+(N-2)+...+2+1=N∧2/2-N/2;

元素交换的次数为：

(N-1)+(N-2)+...+2+1=N∧2/2-N/2;

总执行次数为：N∧2-N

按照大O推导法则，保留函数中的最高阶项，冒泡排序的时间复杂度为O(N∧2)。

## 2.选择排序

选择排序是一种更加简单直观的排序方法。

排序原理：

1. 每一次遍历的过程中，都假定第一索引处的元素是最小值，和其它索引处的值一次进行比较，如果当前索引处的值大于其它某个索引处的值，则假定其它某个索引处的值为最小值，最后可以找到最小值所在的索引。
2. 交换第一个索引处和最小值所在的索引处的值。

代码：

```java
public class Test {
    public static void main(String[] args) {
        int[] arrays = {2,7,1,5,9,3,0};
        selection(arrays);
    }
    public static void selection(int[] array) {
        for (int i = 0; i <= array.length - 2; i++) {
            int minIndex = i;
            for (int j = i + 1; j < array.length; j++) {
                if (array[minIndex] > array[j]) {
                    minIndex = j;
                }
            }
            int temp = array[i];
            array[i] = array[minIndex];
            array[minIndex] = temp;
        }
        System.out.println(Arrays.toString(array));
    }
}
```

选择排序的时间复杂度分析：

选择排序使用了双层for循环，其中外层循环完成了数据交换，内存循环完成了数据比较，分别统计数据交换次数和数据比较次数：

数据比较次数：(N-1)+(N-2)+...+2+1=N∧2/2-N/2;

数据交换次数：N-1;

时间复杂度：N∧2/2-N/2+N-1=N∧2/2+N/2-1

根据大O推导法则，保留最高阶项，去除常数因子，时间复杂度为O(N∧2);

## 3.插入排序

插入排序是一种简单直观且稳定的排序算法。插入排序的工作方式就像排序一张扑克牌一样。开始说，左手为空，然后一次拿走一张牌并将它插入到左手正确的位置，为了找到一张牌的正确位置，需要从左到右将它与已在手中的每张牌进行比较。

排序原理：

1. 把所有的元素分为两组，已经排序的和未排序的。
2. 找到未排序的组中的第一个元素，向已经排序的组中进行插入。
3. 倒叙便利已经排序的元素，依次和待插入的元素进行比较，直到找到元素小于待插入元素，那么就把待插入元素放到这个位置，其它的元素向后移动一位。

代码：

```java
public class Test {
    public static void main(String[] args) {
        int[] arrays = {2,7,1,5,9,3,0};
        insertion(arrays);
    }
    public static void insertion(int[] array){
        for (int i = 1; i < array.length; i++) {
            for (int j = i; j > 0; j--) {
                if (array[j-1] > array[j]){
                    int temp = array[j-1];
                    array[j-1] = array[j];
                    array[j] = temp;
                }else {
                    break;
                }
            }
        }
        System.out.println(Arrays.toString(array));
    }
}
```

插入排序时间复杂度分析：

插入排序使用了双层for循环，其中内层循环是真正完成排序的代码，所以主要分析下内层循环体的执行次数即可。

最坏情况（逆序）时，则比较的次数为：(N-1)+(N-2)+...+2+1=N∧2/2-N/2;

交换的次数为：(N-1)+(N-2)+...+2+1=N∧2/2-N/2;

总执行次数为：N∧2-N;

按照大O推导法则，保留函数的最高阶项，插入排序的时间复杂度为O(N∧2)。

# 二、高级排序

## 1.希尔排序

希尔排序是插入排序的一种，又称为“缩小增量排序”，是插入排序算法的一种更高效的改进版本。当使用插入排序的时候，如果已排序的分组元素为(2,5,7,9,10)，未排序的分组为（1,8），那么下一个待插入元素为1，需要拿1从后往前排，依次和10,9,7,5,3进行交换位置，才能完成真正的插入，每次交换只能和相邻的元素交换位置，如果要提高效率，直观的方法就是一次交换，能把1放到更前面的位置，比如一次交换就能把1插到2和5之间，这样一次交换1就向前走了5个位置，可以减少交换的次数，希尔排序就是这样的原理。

排序原理：

1. 选定一个增长量h，按照增长量h作为数据分组的依据，对数据进行分组。该增量有一定的规则：初始值为1，根据数组的长度确定值：while (h < array.length / 2)  h = 2 * h + 1;每次循环减少增量值：h = h / 2;
2. 对分好组的每一组数据完成插入排序。
3. 减小增长量，最小减为1，重复第二步骤。

代码：

```java
public class Algorithm {

    public void shell(int[] array) {
        // 根据数组的长度，确定初始的h值
        int h = 1;
        while (h < array.length / 2)
            h = 2 * h + 1;
        // 排序结束条件，增量为1时，则可以排序完成，h位置及往后位置为待插入元素，依次遍历
        while (h >= 1) {
            for (int i = h; i < array.length; i++) // 每趟排序的，开始的第一组插入排序应从h出开始
                for (int j = i; j >= h; j -= h) {  // 每组依次进行插入排序
                    if (array[j] < array[j - h]) {
                        int temp = array[j];
                        array[j] = array[j - h];
                        array[j - h] = temp;
                    }else {
                        break;
                    }
                }
            // 每一次循环减少h增长值，当h为1时，数据只有一组，排序完成
            h = h / 2;
        }
    }
}
```

```java
public class AlgorithmTest {
    public Algorithm algorithm;

    @Before
    public void setAlgorithm(){
        algorithm = new Algorithm();
    }

    @Test
    public void shellTest(){
        int[] array = new int[]{2, 7, 1, 5, 9, 3, 0,7,9,43,97,4,24,67,7,23,64,34,52,56,9,0};
        algorithm.shell(array);
        System.out.println(Arrays.toString(array));
    }
}
```

希尔排序时间复杂度分析：采用事后分析法，与插入排序进行对比。......

## 2.归并排序

递归：定义方法时，在方法内部调用方法本身，称之为递归，代码：

```java
public void show(){
    show();
}
```

作用：它通常把一个大型复杂的问题，层层转换为一个与原问题相似的，规模较小的问题来求解，在递归中，不能无限制的调用自己，必须要有出口条件，能够让递归结束，因为每一次递归调用都会在栈内存中开辟新的空间，重新执行方法，如果递归的层级太深，很容易造成栈内存溢出。

归并排序是建立在归并操作上的一种有效的排序算法，该算法是采用分治法的一个非常典型的作用。将已有的子序列合并，得到完全有序的序列。若将两个有序表合并成一个有序表，称为二路归并。

排序原理：

1. 尽可能的将一组数据拆分成两个元素相等的子组，并对每个子组继续拆分，直到拆分后的每个子组的元素个数是1为止。
2. 将相邻的两个子组进行合并为一个有序的大组。
3. 不断的重复步骤二，直到最终为一个组。

代码如下：

```java
package cn.aymeng;

public class Merge {

    private int[] assist = {};

    // 递归
    public void sort(int[] array, int low, int high) {

        assist = new int[array.length];
        if (high <= low)
            return;
        int mid = low + (high - low) / 2;
        sort(array, low, mid);
        sort(array, mid + 1, high);
        merges(array, low, mid, high);
    }

    // 归并
    public void merges(int[] array, int low, int mid, int high) {
        int p1 = low;
        int p2 = mid + 1;
        int p3 = low;
        while (p1 <= mid && p2 <= high)
            assist[p3++] = array[p1] <= array[p2] ? array[p1++] : array[p2++];
        while (p1 <= mid)
            assist[p3++] = array[p1++];
        while (p2 <= high)
            assist[p3++] = array[p2++];
        for (int i = low; i <= high; i++) {
            array[i] = assist[i];
        }
    }
}
```

归并排序时间复杂度分析：

归并排序是分治思想的最典型的例子，它先将数组分为array[low....mid]和array[mid+1....high]两部分，分别通过递归将它们单独进行排序，最后将有序的子组合并为最终的排序结果。该递归的出口在于如果一个数组不能再被分为两个子数组，那么就会执行归并，在归并的时候判断元素的大小进行排序。

[![gxzRBj.png](https://z3.ax1x.com/2021/05/25/gxzRBj.png)](https://imgtu.com/i/gxzRBj)

归并排序的时间复杂度为O(n㏒n)；

归并排序的缺点：

需要申请额外的数组空间，导致空间复杂度提升，是典型的以空间换时间的操作。

## 3.快速排序

快速排序是对冒泡排序的一种改进。它的基本思想是：通过一趟排序将待排序的数据分割成独立的两部分，其中一部分的数据都比另外一部分的所有数据要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以使用递归，以此达到整个数据变成有序数列。

排序原理：

* 首先设置一个分界值，通过该分界值将数组分成左右两部分。
* 将大于或等于分界值的数据放到数组右边，小于分界值的数据放到左边。此时左边部分中各元素都小于或等于分界值，而右边部分中各元素都大于或等于分界值。
* 然后，左边和右边的数据可以独立排序。对于左边的数据，又可以取一个分界值，将该部分数据分成左右两部分，同样在左边放置较小值，右边放置较大值。右侧的数据做类似处理。
* 重复上述过程。通过递归将左侧部分排好序后，再递归排好右侧部分的顺序。当左侧和右侧两个部分的数据排完序后，整个数组的排序也就完成了。

切分原理，把一个数组切分为两个子数组的基本思想：

1. 找一个基准值，用两个指针分别指向数组的头部和尾部（头部为数组下标0，尾部为数组下标length）
2. 先从尾部向头部开始搜索一个比基准值小的元素，搜索到即停止，并记录指针的位置。
3. 再从头部向尾部开始搜索一个比基准值大的元素，搜索到即停止，并记录指针的位置。
4. 交换当前左边指针和右边指针位置的元素。
5. 重复2，3，4步骤，直到左边指针大于右边指针的值停止。

代码：

```java
public class Quick {

    public void sort(int[] array, int low, int high) {
        // 安全性校验
        if (high <= low)
            return;
        // 对从low到high进行分组
        int partition = partition(array, low, high);
        // 让左子组有序
        sort(array, low, partition-1);
        // 让右子组有序
        sort(array, partition + 1, high);
    }

    public int partition(int[] array, int low, int high){
        // 数组最左侧的值当做分界值
        int key = array[low];
        // 定义左侧的指针，指向array的最小索引处
        int left = low;
        // 定义右侧的指针，指向high+1
        int right = high + 1;

        // 切分数组
        while (true){
            // 先从右往左扫描，移动right指针，找到一个比分界值小的元素，停止
            while (array[--right] > key){
                if (right == low)
                    break;
            }
            // 再从左往右扫描，移动left指针，找到一个比分界值大的元素，停止
            while (array[++left] <= key){
                if (left == high)
                    break;
            }
            // 如果两个扫描完后，left>=right，则证明元素扫描完毕，结束循环，如果不是，则交换元素即可
            if (left >= right)
                break;
            else{
              int temp = array[left];
              array[left] = array[right];
              array[right] = temp;
            }
        }
        // 交换分界值
        int temp = array[low];
        array[low] = array[right];
        array[right] = temp;

        return right;
    }
}
```

时间复杂度为：最优为O(n㏒n)，最差为O(n²)。

# 三、表、栈和队列

抽象数据类型（abstract data type，ADT）是带有一组操作的一些对象的集合。抽象数据类型是数学的抽象。在ADT的定义中没有地方提到关于这组操作是如何实现的任何解释。诸如表、集合、图以及与它们各自的操作一起形成的这些对象都可以被看做是抽象数据类型。

## 1.表ADT

我们将处理形如A0,A1,A2....An-1的一般的表。表的大小是N。大小为0的表称为空表。对于除空表以外的任何表，说Ai后继Ai-1并称Ai-1前驱Ai。表的第一个元素是A0，而最后一个元素是An-1。没有定义A0的前驱，也没有定义An-1的后继元。元素Ai在表中的位置为i+1。

* 表的简单数组实现

  对表的所有这些操作都可以通过使用数组来实现。虽然数组是由固定容量创建的，但在需要的时候可以用双倍的容量创建一个不同的数组。它解决了数组固定长度的这个问题。数组的实现可以使的打印list以线性时间被执行，而查询操作则花费常数时间。不过，插入和删除的花费需要有昂贵的花销，删除或插入一个元素所花费的时间最坏的情况为O(N)。

* 简单链表实现

  为了避免插入和删除的线性开销，我们需要保证表可以不连续存储。链表由一系列节点组成，这些节点不必在内存中相连，每一个节点均含有表元素到包含该元素后继元的节点的链，称之为next链。最后一个next链引用为null。链表的打印和查询操作是线性时间的。让每一个节点持有一个指向它在表中的前驱节点的链，称之为双链表。

## 2.Java Collections API

* Collection接口

  Collection接口位于java.util包中。集合的概念在Collection接口中得到抽象，它存储一组类型相同的对象。Collection接口拓展了Iterable接口。实现Iterable接口的那些类可以拥有增强的for循环，该循环施于这些类之上以观察它们所有的项。

* Iterator接口

  实现Iterable接口的集合必须提供一个称为iterator的方法，该方法返回一个Iterator类型的对象。Iterator接口的思路是通过iterator方法，每个集合均可创建并返回给客户一个实现Iterator的对象，并将当前位置的概念在对象内部存储下来。每次对next的调用都给出集合的下一项，hasNext用来判断是否还有下一项。Iterator接口还包含一个方法，叫做remove。该方法可以删除有next最新返回的项（此后，不能再调用remove，直到对next再一次调用以后）。

* List接口、ArrayList类和LinkedList类

  List ADT有两种流行的实现方式。ArrayList类提供了List ADT的一种可增长数组的实现，它的get和set的调用花费常数时间。缺点是插入项和删除代价昂贵。LinkedList类则提供了List ADT的双链表实现，它的新插入项和删除项钧开销很小，缺点是它不容易做索引，因此对get的调用时昂贵的。

* ListIterator接口

  ListIterator拓展了List的Iterator的功能。方法previous和hasPrevious使得对表从后向前的遍历得以完成。add方法将一个新的项从当前位置放入表中。

## 3.ArrayList类的实现

见java.util.ArrayList

## 4.LinkedList类的实现

见java.util.LinkedList

## 5.栈

栈是限制插入和删除只能在一个位置上进行的表，该位置是表的末端，叫做栈的顶（top）。对栈的基本操作有push(进栈)和pop（出栈），前者相当于插入，后者则是删除最后的元素。栈有时又叫做LIFO（后进先出），栈操作是常数时间的。

* 栈的第一种实现是单链表。通过在表的顶端插入来实现push，通过删除表顶端元素实现pop。
* 栈的另一种实现使用数组实现。这些操作不仅以常数时间运行，而且是非常快的常数时间运行。

栈的应用：

* 平衡符号

  编译器检查程序的圆括号、方括号和花括号序列是否合法的，可以使用栈的算法来实现：做一个空栈。读入字符知道文件结尾，如果字符是一个开放符号，则将其推入栈中。如果字符是一个封闭符号，则当栈空时报错。否则，将栈元素弹出。如果弹出的符号不是对应的开放符号，则报错。在文件结尾，如果栈非空则报错。

* 后缀表达式

  正常的二元算术式的写法如下：4.99\*1.06+5.99+6.99\*1.06=，但是如果计算机实现的话，需要进行好多算法判断。然后可以使用后缀或逆波兰记法，从而变的简单，如下：4.99 1.06\*5.99+6.99 1.06\*+。这种方法的算法是使用一个栈。当见到一个数时就把它推入栈中；在遇到一个运算时该算符就作用于从该栈弹出的两个数上，再将所得结果推入栈中。

## 6.队列

队列也是一种表，然后，使用队列时插入在一端进行而删除则在另一端进行。队列的基本操作是enqueue（入队），它是在表的末端插入一个元素，和dequeue（出队），它是删除在表的开头的元素。

# 四、树

树可以用几种方式定义。定义树的一种自然的方式是递归的方式。一棵树是一些节点的集合。这个集合可以是空集，如不是空集，则树由根的节点以及0个或多个非空的子树组成，这些子树中没一棵的根都被来自根r的一条有向的边所连结。从节点n1到nk的路径定义为节点n1,n2...nk的一个序列，使得对于1<=i<k节点ni是n(i+1)的父亲。对于任意节点ni，ni的**深度**为从根到ni的唯一的路径的长。ni的**高**是从ni到一片树叶的最长路径的长。

实现树的一种方法可以是在每一个节点除数据外还要有一条链，使得该节点的每一个儿子都有一个链指向它。然而，由于每个节点的儿子树可以变化很大并且事先不知道，因此在数据结构中建立到各（儿子）节点直接的链接是不可行的，因为这样会产生太多浪费的空间。实际上解决方法很简单：将每一个节点的所有儿子都放在树节点的链表中。如下：

```java
class TreeNode{
    Object element;
    TreeNode firstChild;
    TreeNode nextSibling;
}
```

以上方法指出一棵树的实现方法：每个节点有一个数据element，firstChild是指向第一个儿子的链，nextSibiling是指向下一个兄弟的水平的链。

## 1.二叉树

二叉树是一棵树，其中的每个节点都不能有多于两个的儿子。

二叉树的一个性质是一课平均二叉树的深度要比节点个数N小得多。分析表明，其平均深度为O(√N)，对于特殊类型的二叉树，即二叉查找树，其深度的平均值是O(㏒N)。但是深度也可以大到N-1的。