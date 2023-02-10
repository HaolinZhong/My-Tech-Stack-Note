This is a personal note for core Java related knowledges.



# OOP

## Constructor

- 没有定义构造方法时，编译器会自动创建一个默认的无参数构造方法；如果我们自定义了一个构造方法，那么，编译器就*不再*自动创建默认构造方法.

## Overload

- 方法重载是指多个方法的方法名相同，但各自的参数不同；
- 重载方法应该完成类似的功能，参考`String`的`indexOf()`；
- 重载方法返回值类型应该相同。



## Override

- 在继承关系中，子类如果定义了一个与父类方法签名完全相同的方法，被称为覆写（Override）。
- Override和Overload不同的是，如果方法签名不同，就是Overload，Overload方法是一个新方法；如果方法签名相同，并且返回值也相同，就是`Override`。



## Polymorphorism

- 多态是指，
  - 可以以父类声明变量, 并赋给变量子类的实例
  - 针对某个类型的方法调用，其真正执行的方法取决于运行时期实例的实际类型的方法。
- 在子类的覆写方法中，如果要调用父类的被覆写的方法，可以通过`super`来调用。



## Abstract Class & Interface

### Abstract Class

- 如果一个`class`定义了方法，但没有具体执行代码，这个方法就是抽象方法，抽象方法用`abstract`修饰。
- 因为无法执行抽象方法，因此这个类也必须申明为抽象类（abstract class）。
- 抽象类可以强迫子类实现其定义的抽象方法，否则编译会报错。因此，抽象方法实际上相当于定义了“规范”。
- 如果不实现抽象方法，则该子类仍是一个抽象类；
- 抽象类中也可以定义非抽象方法, 为子类所共享.



### Interface

- 如果一个抽象类没有字段，所有方法全部都是抽象方法, 就可以把该抽象类改写为接口：`interface`。
  - `interface`不能有普通字段, 但可以有静态字段，并且静态字段必须为`public static final`类型(可省略不写)。
- 接口定义的所有方法默认都是`public abstract`，所以这两个修饰符不需要写出来（写不写效果都一样）。
- 一个类只能继承自另一个类, 但一个类可以实现多个`interface`。
- 在接口中，可以定义`default`方法。实现类可以不必覆写`default`方法。`default`方法的目的是，当我们需要给接口新增一个方法时，会涉及到修改全部子类。如果新增的是`default`方法，那么子类就不必全部修改，只需要在需要覆写的地方去覆写新增方法。
  - default方法不能访问字段, 因为接口本身就没有字段.



## Static Fields & Methods

- 静态字段只有一个共享“空间”，所有实例都会共享该字段。
- 对于静态字段，无论修改哪个实例的静态字段，效果都是一样的：所有实例的静态字段都被修改了，原因是静态字段并不属于实例, 而属于类.
- 推荐用类名来访问静态字段.



- 调用实例方法必须通过一个实例变量，而调用静态方法则不需要实例变量，通过类名就可以调用。
- 因为静态方法属于`class`而不属于实例，因此，静态方法内部，无法访问`this`变量，也无法访问实例字段，它只能访问静态字段。
- 通过实例变量也可以调用静态方法，但这只是编译器自动帮我们把实例改写成类名而已。
- 静态方法经常用于工具类, 辅助方法.





## Nested class

- 一个`.java`文件只能包含一个`public`类，但可以包含多个非`public`类。如果有`public`类，文件名必须和`public`类的名字相同。
- 还有一种类，它被定义在另一个类的内部，所以称为内部类（Nested Class）

### Inner Class

- 如果一个类定义在另一个类的内部，这个类就是Inner Class
- Inner Class的实例不能单独存在，必须依附于一个Outer Class的实例
- Inner Class除了有一个`this`指向它自己，还隐含地持有一个Outer Class实例，可以用`Outer.this`访问这个实例。所以，实例化一个Inner Class不能脱离Outer实例。
- nner Class和普通Class相比，除了能引用Outer实例外，还有一个额外的“特权”，就是可以修改Outer Class的`private`字段，因为Inner Class的作用域在Outer Class内部，所以能访问Outer Class的`private`字段和方法。



### Static Nested Class

- 例

  ```java
  public class Main {
      public static void main(String[] args) {
          Outer.StaticNested sn = new Outer.StaticNested();
          sn.hello();
      }
  }
  
  class Outer {
      private static String NAME = "OUTER";
  
      private String name;
  
      Outer(String name) {
          this.name = name;
      }
  
      // static nested class
      static class StaticNested {
          void hello() {
              System.out.println("Hello, " + Outer.NAME);
          }
      }
  }
  
  ```

- 用`static`修饰的内部类和Inner Class有很大的不同，它不再依附于`Outer`的实例，而是一个完全独立的类，因此无法引用`Outer.this`，但它可以访问`Outer`的`private`静态字段和静态方法。

- 如果把`StaticNested`移到`Outer`之外，就失去了访问`private`的权限。

- 它的特色在于, 既与outer平级, 又能访问outer的private的static的字段, 方法.



### Anonymous Class

- 在方法内部定义的无名类。

- 例:

  ```java
  public class Main {
      public static void main(String[] args) {
          Outer outer = new Outer("Nested");
          outer.asyncHello();
      }
  }
  
  class Outer {
      private String name;
  
      Outer(String name) {
          this.name = name;
      }
  
      void asyncHello() {
          // 定义匿名类
          Runnable r = new Runnable() {
              @Override
              public void run() {
                  System.out.println("Hello, " + Outer.this.name);
              }
          };
          new Thread(r).start();
      }
  }
  
  ```

  - `Runnable`本身是接口，接口是不能实例化的，所以这里实际上是定义了一个实现了`Runnable`接口的匿名类，并且通过`new`实例化该匿名类，然后转型为`Runnable`。

  - 在定义匿名类的时候就必须实例化它.

    

- 除了接口外，匿名类也完全可以继承自普通类。

  ```java
  import java.util.HashMap;
  
  public class Main {
      public static void main(String[] args) {
          HashMap<String, String> map1 = new HashMap<>();		// hashmap实例
          HashMap<String, String> map2 = new HashMap<>() {}; // 匿名类, 继承hashmap!
          HashMap<String, String> map3 = new HashMap<>() {   // 同
              {									// 匿名的static方法初始化了自身
                  put("A", "1");
                  put("B", "2");
              }
          };
          System.out.println(map3.get("A"));
      }
  }
  ```





# 集合

## Intro

- 位于`java.util.Collection` 下
- 主要有三种类型(接口): List, Set, Map
  - Java集合的设计有几个特点：
    - 实现了接口和实现类相分离，例如，有序表的接口是`List`，具体的实现类有`ArrayList`，`LinkedList`等
    - 二是支持泛型，我们可以限制在一个集合中只能放入同一种数据类型的元素
- Java访问集合总是通过统一的方式——迭代器（Iterator）来实现，它最明显的好处在于无需知道集合内部元素是按什么方式存储的。
- _由于Java的集合设计非常久远，中间经历过大规模改进，我们要注意到有一小部分集合类是遗留类，不应该继续使用_：
  - _`Hashtable`：一种线程安全的`Map`实现；_
  - _`Vector`：一种线程安全的`List`实现；_
  - _`Stack`：基于`Vector`实现的`LIFO`的栈。_



## List

- 对比ArrayList与LinkedList: 绝大多数情况下优先使用ArrayList

  |                     | ArrayList    | LinkedList           |
  | :------------------ | :----------- | -------------------- |
  | 获取指定元素        | 速度很快     | 需要从头开始查找元素 |
  | 添加元素到末尾      | 速度很快     | 速度很快             |
  | 在指定位置添加/删除 | 需要移动元素 | 不需要移动元素       |
  | 内存占用            | 少           | 较大                 |

- 创建List: 除了new, 还可用`List.of()`, 如`List<Integer> list = List.of(1, 2, 5);`
- 遍历list: 优先使用`Iterator`访问
  - `Iterator`本身也是一个对象，是由`List`的实例调用`iterator()`方法的时候创建的。
  - `Iterator`对象有两个方法：`boolean hasNext()`判断是否有下一个元素，`E next()`返回下一个元素。



## Map

- 遍历map

  - 要遍历`key`可以使用`for each`循环遍历`Map`实例的`keySet()`方法返回的`Set`集合

  - 同时遍历`key`和`value`可以使用`for each`循环遍历`Map`对象的`entrySet()`集合，它包含每一个`key-value`映射

    ```java
            for (Map.Entry<String, Integer> entry : map.entrySet()) {
                String key = entry.getKey();
                Integer value = entry.getValue();
                System.out.println(key + " = " + value);
            }
    ```

- TreeMap

  - map包括HashMap和SortedMap, TreeMap是SortedMap的实现
  - 作为`SortedMap`的Key必须实现`Comparable`接口，或者传入`Comparator`；
  - 要严格按照`compare()`规范实现比较逻辑，否则，`TreeMap`将不能正常工作。
    - 即: ab相等返回0, a优先级高于b返回-1, 否则返回1



## Set

- `Set`用于存储不重复的元素集合，它主要提供以下几个方法：

  - 将元素添加进`Set<E>`：`boolean add(E e)`
  - 将元素从`Set<E>`删除：`boolean remove(Object e)`
  - 判断是否包含元素：`boolean contains(Object e)`

- 最常用的`Set`实现类是`HashSet`，实际上，`HashSet`仅仅是对`HashMap`的一个简单封装.

  ```java
  public class HashSet<E> implements Set<E> {
      // 持有一个HashMap:
      private HashMap<E, Object> map = new HashMap<>();
  
      // 放入HashMap的value:
      private static final Object PRESENT = new Object();
  
      public boolean add(E e) {
          return map.put(e, PRESENT) == null;
      }
  
      public boolean contains(Object o) {
          return map.containsKey(o);
      }
  
      public boolean remove(Object o) {
          return map.remove(o) == PRESENT;
      }
  }
  ```

  

- `HashSet`不保证有序, 而TreeSet实现了`SortedSet`的接口, 元素有序排列



## Queue

- 在Java的标准库中，队列接口`Queue`定义了以下几个方法：

  - `int size()`：获取队列长度；

  - `boolean add(E)`/`boolean offer(E)`：添加元素到队尾；

  - `E remove()`/`E poll()`：获取队首元素并从队列中删除；

  - `E element()`/`E peek()`：获取队首元素但并不从队列中删除。

- queue常用的实现类是LinkedList



## PriorityQueue

- 初始化: `Queue<String> q = new PriorityQueue<>();`

- 放入`PriorityQueue`的元素，必须实现`Comparable`接口, 否则要提供一个`Comparator`对象来判断两个元素的顺序

  - 提供comparator:

    ```java
           Queue<Person> q = new PriorityQueue<>(new Comparator<Person>() {
                public int compare(Person p1, Person p2) {
                    return p1.name.compareTo(p2.name);
                }
            });
    ```

    

## Deque

- 允许两头都进，两头都出，这种队列叫双端队列（Double Ended Queue），学名`Deque`。`Deque`接口实际上扩展自`Queue`.
- 它的实现类有`ArrayDeque`和`LinkedList`。
- 各种方法就是queue的方法名加上First或Last



## Collections

- `Collection`是定义各类数据结构的包, 而`Collections`是一个工具类
- 最常用: `Collections.sort(list)`, 对可变list进行排序





# 原生多线程



## 创建新线程

- 要创建一个新线程非常容易，我们需要实例化一个`Thread`实例，然后调用它的`start()`方法：

  ```java
  public class Main {
      public static void main(String[] args) {
          Thread t = new Thread();
          t.start(); // 启动新线程
      }
  }
  ```

- 但是这个线程启动后实际上什么也不做就立刻结束了。我们希望新线程能执行指定的代码，有以下几种方法：

  - 从`Thread`派生一个自定义类，然后覆写`run()`方法

  - 创建`Thread`实例时，传入一个`Runnable`实例 (可用Java8引入的lambda语法简写)

    ```java
    public class Main {
        public static void main(String[] args) {
            Thread t = new Thread(() -> {
                System.out.println("start new thread!");
            });
            t.start(); // 启动新线程
        }
    }
    ```

    

- 可以对线程设定优先级，设定优先级的方法是：`Thread.setPriority(int n)`, 优先级为1~10, 默认值5
  - JVM自动把1（低）~10（高）的优先级映射到操作系统实际优先级上（不同操作系统有不同的优先级数量）。优先级高的线程被操作系统调度的优先级较高，操作系统对高优先级线程可能调度更频繁，但我们决不能通过设置优先级来确保高优先级的线程一定会先执行。



## 线程的状态

- Java线程的状态有以下几种：
  - New：新创建的线程，尚未执行；
  - Runnable：运行中的线程，正在执行`run()`方法的Java代码；
  - Blocked：运行中的线程，因为某些操作被阻塞而挂起；
  - Waiting：运行中的线程，因为某些操作在等待中；
  - Timed Waiting：运行中的线程，因为执行`sleep()`方法正在计时等待；
  - Terminated：线程已终止，因为`run()`方法执行完毕

- 一个线程还可以等待另一个线程直到其运行结束。例如，`main`线程在启动`t`线程后，可以通过`t.join()`等待`t`线程结束后再继续运行：

  ```java
  public class Main {
      public static void main(String[] args) throws InterruptedException {
          Thread t = new Thread(() -> {
              System.out.println("hello");
          });
          t.start(); // 启动
          t.join();  // 等待t结束
      }
  }
  ```

  

- `join(long)`可以指定等待时间，超过等待时间线程仍然没有结束就不再等待；对已经运行结束的线程调用`join()`方法会立刻返回。



## 中断线程

- 中断一个线程非常简单，只需要在其他线程中对目标线程调用`interrupt()`方法

  - 但在目标线程中需要设置其反复检测自身状态是否是interrupted状态，如果是，就立刻结束运行。

  ```java
  public class Main {
      public static void main(String[] args) throws InterruptedException {
          Thread t = new MyThread();
          t.start();
          Thread.sleep(1); // 暂停1毫秒
          t.interrupt(); // 中断t线程
          t.join(); // 等待t线程结束
      }
  }
  
  class MyThread extends Thread {
      public void run() {
          int n = 0;
          /* 
          ** t线程的while循环会检测isInterrupted()，
          ** 所以上述代码能正确响应interrupt()请求，
          ** 使得自身立刻结束运行run()方法。
          */
          while (! isInterrupted()) {		// isInterrupted()涉及的状态受外部影响
              n ++;
              System.out.println(n + " hello!");
          }
      }
  }
  
  ```

  - 另一种中断的方法是, 利用原理: 

    - 如果线程处于等待状态，例如，`t.join()`会让`main`线程进入等待状态，此时，如果对`main`线程调用`interrupt()`，`join()`方法会立刻抛出`InterruptedException`, 

    - 这样一来, 只要目标线程catch到 `join()`方法抛出的`InterruptedException`, 就应该停止执行

      ```java
      public class Main {
          public static void main(String[] args) throws InterruptedException {
              Thread t = new MyThread();
              t.start();
              Thread.sleep(1000);
              t.interrupt(); // 中断t线程
              t.join(); // 等待t线程结束
              System.out.println("end");
          }
      }
      
      class MyThread extends Thread {
          public void run() {
              Thread hello = new HelloThread();
              hello.start(); // 启动hello线程
              try {
                  hello.join(); // 等待hello线程结束, 但被main打断
              } catch (InterruptedException e) {
                  System.out.println("interrupted!");	   // 被打断后打印
              }	
              hello.interrupt();		// 结束前打断hello
          }
      }
      
      class HelloThread extends Thread {
          public void run() {
              int n = 0;
              while (!isInterrupted()) {
                  n++;
                  System.out.println(n + " hello!");
                  try {
                      Thread.sleep(100);		// time waiting状态, 被myThread打断
                  } catch (InterruptedException e) {
                      break;	// 打断后结束while循环, 退出
                  }
              }
          }
      }
      ```

      

  - 另一个常用的中断线程的方法是设置标志位。我们通常会用一个`running`标志位来标识线程是否应该继续运行，在外部线程中，通过把`HelloThread.running`置为`false`，就可以让线程结束：

    - 线程间共享变量需要使用`volatile`关键字标记，确保每个线程都能读取到更新后的变量值。

      ```java
      public class Main {
          public static void main(String[] args)  throws InterruptedException {
              HelloThread t = new HelloThread();
              t.start();
              Thread.sleep(1);
              t.running = false; // 标志位置为false
          }
      }
      
      class HelloThread extends Thread {
          public volatile boolean running = true;		// volatile关键字!
          public void run() {
              int n = 0;
              while (running) {
                  n ++;
                  System.out.println(n + " hello!");
              }
              System.out.println("end!");
          }
      }
      ```

    - 为什么要使用`volatile`关键字: JVM内存模型

      - 在Java虚拟机中，变量的值统一保存在主内存中
      - 当线程访问变量时，它会先获取一个副本，并保存在自己的工作内存中。如果线程修改了变量的值，虚拟机会在某个时刻把修改后的值回写到主内存，但是，这个时间是不确定的！
      - 也就是说, 不同线程中共享的同一变量的值可能不会随时一致. 而且, 线程只在修改变量值后才会联系主内存. 
      - 在上面的case中, hello线程本身没有对running做更新, 所以不加volatile的话永远不会读主内存
      - 因此，`volatile`关键字的目的是告诉虚拟机：
        - 每次访问变量时，总是获取主内存的最新值；
        - 每次修改变量后，立刻回写到主内存。
      - `volatile`并不能保证线程的同步, 只能保证读取变量时也从主内存拿, 以及及时的从主内存存取变量



## 守护线程

- Java程序入口就是由JVM启动`main`线程，`main`线程又可以启动其他线程。当所有线程都运行结束时，JVM退出，进程结束。
  - 如果有一个线程没有退出，JVM进程就不会退出。所以，必须保证所有线程都能及时结束。
  - 但是有一种线程的目的就是无限循环. 如果这个线程不结束，JVM进程就无法结束。问题是，由谁负责结束这个线程？
  - 然而这类线程经常没有负责人来负责结束它们。但是，当其他线程结束时，JVM进程又必须要结束，怎么办？ 答案是使用`守护线程（Daemon Thread）`。

- 守护线程是指为其他线程服务的线程。在JVM中，所有非守护线程都执行完毕后，无论有没有守护线程，虚拟机都会自动退出。因此，JVM退出时，不必关心守护线程是否已结束。

- 创建守护线程方法和普通线程一样，只是在调用`start()`方法前，调用`setDaemon(true)`把该线程标记为守护线程

  ```java
  Thread t = new MyThread();
  t.setDaemon(true);
  t.start();
  ```

  

- 在守护线程中，编写代码要注意：守护线程**不能持有任何需要关闭的资源**，例如打开文件等，因为虚拟机退出时，守护线程没有任何机会来关闭文件，这会导致数据丢失。



## 线程同步

- 对变量进行读取和写入时，结果要正确，必须保证是原子操作。原子操作是指不能被中断的一个或一系列操作。

  - 对于语句`n = n + 1`, 它看上去是一行语句, 实际上对应了3条指令: `ILOAD`, `IADD`, `ISTORE`
  - 在执行这三条语句期间, 如果有其他线程进行了其他操作, 就可能对执行的结果造成影响, 破坏了原子性, 造成逻辑上的错误.
  - 所以, 多线程模型下，要保证逻辑正确，对共享变量进行读写时，必须保证一组指令以原子方式执行：即某一个线程执行时，其他线程必须等待.

- Java程序使用`synchronized`关键字对一个对象进行加锁, `synchronized`保证了代码块在任意时刻最多只有一个线程能执行。

  ```java
  public class Main {
      public static void main(String[] args) throws Exception {
          var add = new AddThread();
          var dec = new DecThread();
          add.start();
          dec.start();
          add.join();
          dec.join();
          System.out.println(Counter.count);
      }
  }
  
  // Counter作为一个公共变量的存放地, 拥有static的属性
  class Counter {	
      // 所谓"锁"也没有什么高级, 就是一个static final Object
      // 需要这个玩意是因为synchronize关键字后面要填一个object, 也是类似Pthread的做法
      public static final Object lock = new Object();   
      public static int count = 0;
  }
  
  class AddThread extends Thread {
      public void run() {
          for (int i=0; i<10000; i++) {
              synchronized(Counter.lock) {
                  Counter.count += 1;
              }
          }
      }
  }
  
  class DecThread extends Thread {
      public void run() {
          for (int i=0; i<10000; i++) {
              synchronized(Counter.lock) {
                  Counter.count -= 1;
              }
          }
      }
  }
  ```

  

- java中释放锁是在代码块结束时自动发生的:

  ```java
  synchronized(Counter.lock) { // 获取锁
      ...
  } // 释放锁
  ```

  

- 使用`synchronized`解决了多线程同步访问共享变量的正确性问题。但是，它的缺点是带来了性能下降。因为`synchronized`代码块无法并发执行。此外，加锁和解锁需要消耗一定的时间，所以，`synchronized`会降低程序的执行效率。



- 不需要同步的操作: 单条原子操作的语句不需要同步
  - JVM规范定义了几种原子操作：
    - 基本类型（`long`和`double`除外）赋值，例如：`int n = m`；
    - 引用类型赋值，例如：`List<String> list = anotherList`。
    - `long`和`double`是64位数据，JVM没有明确规定64位赋值操作是不是一个原子操作，不过在x64平台的JVM是把`long`和`double`的赋值作为原子操作实现的。
  - 但是，如果是多行赋值语句，就必须保证是同步操作



## 同步方法

- 让线程自己选择锁对象往往会使得代码逻辑混乱，也不利于封装。更好的方法是把`synchronized`逻辑封装在方法内部。

  ```java
  public class Counter {
      private int count = 0;
      public void add(int n) {
          synchronized(this) {   // 方法内部对实例自身加锁
              count += n;
          }
      }
      public void dec(int n) {
          synchronized(this) {
              count -= n;
          }
      }
      public int get() {		// 读一个int变量不需要同步
          return count;
      }
  }
  ```

  

- 如果一个类被设计为允许多线程正确访问，我们就说这个类就是“线程安全”的（thread-safe）

  - 如: `StringBuffer`
  - `String`，`Integer`，`LocalDate`，它们的所有成员变量都是`final`，多线程同时访问时只能读不能写，这些不变类也是线程安全的。
  - 类似`Math`这些只提供静态方法，没有成员变量的类，也是线程安全的。
  - 大部分类，例如`ArrayList`，都是非线程安全的类，我们不能在多线程中修改它们。但是，如果所有线程都只读取，不写入，那么`ArrayList`是可以安全地在线程间共享的。



- 当我们锁住的是`this`实例时，实际上可以用`synchronized`修饰这个方法。下面两种写法是等价的：

  ```java
  public void add(int n) {
      synchronized(this) { // 锁住this
          count += n;
      } // 解锁
  }
  
  // 可以直接用synchronized来修饰方法
  
  public synchronized void add(int n) { // 锁住this
      count += n;
  } // 解锁
  ```

  - 如果对一个静态方法添加`synchronized`修饰符，它不可能锁实例, 因为它没有实例. 因此, 锁的其实是那个反射中用到的该类的`Class`实例。



## 死锁

- 核心原因: 以A, B两个锁为例 
  - 嵌套使用A, B两个锁时, 一个线程获取A后再获取B, 另一个线程获取B后再获取A
  - 有几率出现 t1持有A等待B, t2持有B等待A, 这时两方都不可能获取到想要的锁/释放对方需要的锁
- 避免死锁：线程获取锁的顺序要一致。



## wait & notify

- 同`Pthread`中的`cond_wait`和`cond_signal`



- 多线程协调运行的原则就是：当条件不满足时，线程进入等待状态；当条件满足时，线程被唤醒，继续执行任务。

  - `wait()`
    - `wait()`方法必须在当前获取的锁对象上调用，如果作为锁的对象是`this`，那就调用`this.wait()`。
    - 调用`wait()`方法后，线程进入等待状态，`wait()`方法不会返回，直到将来某个时刻，线程从等待状态被其他线程唤醒后，`wait()`方法才会返回，然后，继续执行下一条语句。
    - `wait()`方法的执行机制非常复杂。
      - 首先，它不是一个普通的Java方法，而是定义在`Object`类的一个`native`方法，也就是由JVM的C代码实现的。
      - 其次，必须在`synchronized`块中才能调用`wait()`方法，因为`wait()`方法调用时，会**释放线程获得的锁**，`wait()`方法返回后，线程又会**重新试图获得锁**。

  - `notify()`
    - 在相同的锁对象上调用`notify()`方法, 就可让等待的线程被重新唤醒，然后从`wait()`方法返回
    - `notifyAll()`: 唤醒所有在等待的线程, `notify()`只唤醒其中一个





# Concurent多线程

## ReentrantLock

- 从Java 5开始，引入了一个高级的处理并发的`java.util.concurrent`包，它提供了大量更高级的并发功能，能大大简化多线程程序的编写. 它提供了`ReentrantLock`用于替代`synchronized`加锁

  ```java
  // synchronized写法
  public class Counter {
      private int count;
  
      public void add(int n) {
          synchronized(this) {
              count += n;
          }
      }
  }
  
  // reentrantlock写法
  public class Counter {
      private final Lock lock = new ReentrantLock();	// 创建锁对象
      private int count;
  
      public void add(int n) {
          lock.lock();
          try {
              count += n;
          } finally {
              lock.unlock();
          }
      }
  }
  ```

  

  - 因为`synchronized`是Java语言层面提供的语法，所以我们不需要考虑异常，而`ReentrantLock`是Java代码实现的锁，就需要考虑. 必须先获取锁，执行操作时用`try finally`来保证正确的释放锁。

  - 和`synchronized`不同的是，`ReentrantLock`可以尝试获取锁：

    ```java
    /*
    在尝试获取锁的时候，最多等待1秒。如果1秒后仍未获取到锁，tryLock()返回false，程序就可以做一些额外处理，而不是无限等待下去。
    */
    if (lock.tryLock(1, TimeUnit.SECONDS)) { 
        try {
            ...
        } finally {
            lock.unlock();
        }
    }
    ```

  - 所以，使用`ReentrantLock`比直接使用`synchronized`更安全，线程在`tryLock()`失败的时候不会导致死锁。



## Condition

- `synchronized`可以配合`wait`和`notify`实现线程在条件不满足时等待，条件满足时唤醒; 用`ReentrantLock`我们怎么编写`wait`和`notify`的功能呢？ ---- 使用`Condition`对象。

  ```java
  class TaskQueue {
      private final Lock lock = new ReentrantLock();
      // 创建condition对象
      // 引用的Condition对象必须从Lock实例的newCondition()返回
      private final Condition condition = lock.newCondition();
      private Queue<String> queue = new LinkedList<>();
  
      public void addTask(String s) {
          lock.lock();
          try {
              queue.add(s);
              condition.signalAll();		// signal / broadcast
          } finally {
              lock.unlock();
          }
      }
  
      public String getTask() {
          lock.lock();
          try {
              while (queue.isEmpty()) {
                  condition.await();		// wait
              }
              return queue.remove();
          } finally {
              lock.unlock();
          }
      }
  }
  ```

  

- `Condition`提供的`await()`、`signal()`、`signalAll()`原理和`synchronized`锁对象的`wait()`、`notify()`、`notifyAll()`是一致的，并且其行为也是一样的.

  

- 和`tryLock()`类似，`await()`可以在等待指定时间后，如果还没有被其他线程通过`signal()`或`signalAll()`唤醒，可以自己醒来：

  ```java
  if (condition.await(1, TimeUnit.SECOND)) {
      // 被其他线程唤醒
  } else {
      // 指定时间内没有被其他线程唤醒
  }
  ```

  

## ReadWriteLock

- `synchronized` 和 `ReentrantLock` 都是互斥锁, 保证只有一个线程可以执行临界区代码. 但是有些时候，这种保护有点过头。例如, 仅读数据的一种操作, 实际上允许多个线程同时调用, 而且安全。

  - 实际上我们想要的是：允许多个线程同时读，但只要有一个线程在写，其他线程就必须等待

    

- 使用`ReadWriteLock`可以解决这个问题，它保证：

  - 只允许一个线程写入（其他线程既不能写入也不能读取）；

  - 没有写入时，多个线程允许同时读（提高性能）。

    

- 用`ReadWriteLock`实现这个功能十分容易:

  ```java
  public class Counter {
      private final ReadWriteLock rwlock = new ReentrantReadWriteLock();
      private final Lock rlock = rwlock.readLock();
      private final Lock wlock = rwlock.writeLock();
      private int[] counts = new int[10];
  
      public void inc(int index) {
          wlock.lock(); // 加写锁
          try {
              counts[index] += 1;
          } finally {
              wlock.unlock(); // 释放写锁
          }
      }
  
      public int[] get() {
          rlock.lock(); // 加读锁
          try {
              return Arrays.copyOf(counts, counts.length);
          } finally {
              rlock.unlock(); // 释放读锁
          }
      }
  }
  ```



- 其实完全可以通过`synchronized`或`ReentrantLock`来实现该锁, 这里是提供了一个封装好的功能.





## StampedLock

- `ReadWriteLock` 在读的过程中不允许写，这是一种悲观的读锁。

- `StampedLock`和`ReadWriteLock`相比，改进之处在于：读的过程中也允许获取写锁后写入！这样一来，我们读的数据就可能不一致，所以，需要一点额外的代码来判断读的过程中是否有写入，这种读锁是一种乐观锁。

  - 乐观锁的意思就是乐观地估计读的过程中大概率不会有写入，因此被称为乐观锁。反过来，悲观锁则是读的过程中拒绝有写入，也就是写入必须等待。显然**乐观锁的并发效率更高，但一旦有小概率的写入导致读取的数据不一致，需要能检测出来，再读一遍就行**。

  

- 例子:

  ```java
  public class Point {
      private final StampedLock stampedLock = new StampedLock();
  
      private double x;
      private double y;
  
      public void move(double deltaX, double deltaY) {
          long stamp = stampedLock.writeLock(); // 获取写锁			
          try {
              x += deltaX;
              y += deltaY;
          } finally {
              stampedLock.unlockWrite(stamp); // 释放写锁
          }
      }
  
      public double distanceFromOrigin() {
          long stamp = stampedLock.tryOptimisticRead(); // 获得一个乐观读锁
          
          // 注意下面两行代码不是原子操作
          
          // 假设x,y = (100,200)
          double currentX = x;
          // 此处已读取到x=100，但x,y可能被写线程修改为(300,400)
          double currentY = y;
          
          // 此处已读取到y，如果没有写入，读取是正确的(100,200)
          // 如果有写入，读取是错误的(100,400)
          
          if (!stampedLock.validate(stamp)) { // 检查乐观读锁后是否有其他写锁发生
              stamp = stampedLock.readLock(); // 获取一个悲观读锁
              try {
                  currentX = x;
                  currentY = y;
              } finally {
                  stampedLock.unlockRead(stamp); // 释放悲观读锁
              }
          }
          return Math.sqrt(currentX * currentX + currentY * currentY);
      }
  }
  ```

  - 读取方先通过stampLock获得乐观读锁 (`stampedLock.tryOptimisticRead()`), 然后读取, 然后用`stampLock.validate()`来验证乐观读锁. 
    - 如果验证失败, 说明有线程正在写入, 因此获取悲观读锁, 等待写入完成后再读
    - 否则, 认为写没有发生, 当前读取数据合理, 直接执行后续逻辑



## Semaphore

- 本质上锁的目的是保护一种受限资源，保证同一时刻只有一个线程能访问（ReentrantLock），或者只有一个线程能写入（ReadWriteLock）。
- 还有一种受限资源，它需要保证同一时刻最多有N个线程能访问，比如同一时刻最多创建100个数据库连接，最多允许10个用户下载等。
  - 这种限制数量的锁，如果用Lock数组来实现，就太麻烦了。
  - 这种情况就可以使用`Semaphore`

```java
public class AccessLimitControl {
    // 任意时刻仅允许最多3个线程获取许可:
    final Semaphore semaphore = new Semaphore(3);

    public String access() throws Exception {
        // 如果超过了许可数量,其他线程将在此等待:
        semaphore.acquire();
        try {
            // TODO: 真正的执行逻辑
            return UUID.randomUUID().toString();
        } finally {
            semaphore.release();
        }
    }
}
```

- 使用`Semaphore`先调用`acquire()`获取，然后通过`try ... finally`保证在`finally`中释放。

- 调用`acquire()`可能会进入等待，直到满足条件为止。也可以使用`tryAcquire()`指定等待时间：

  ```java
  if (semaphore.tryAcquire(3, TimeUnit.SECONDS)) {
      // 指定等待时间3秒内获取到许可:
      try {
          // TODO:
      } finally {
          semaphore.release();
      }
  }
  ```



## Concurrent集合

- 针对`List`、`Map`、`Set`、`Queue` 、`Deque`等，`java.util.concurrent`包提供了对应的并发集合类。

  - | interface | non-thread-safe         | thread-safe                              |
    | :-------- | :---------------------- | :--------------------------------------- |
    | List      | ArrayList               | CopyOnWriteArrayList                     |
    | Map       | HashMap                 | ConcurrentHashMap                        |
    | Set       | HashSet / TreeSet       | CopyOnWriteArraySet                      |
    | Queue     | ArrayDeque / LinkedList | ArrayBlockingQueue / LinkedBlockingQueue |
    | Deque     | ArrayDeque / LinkedList | LinkedBlockingDeque                      |

  - 使用这些并发集合与使用非线程安全的集合类完全相同。



- `java.util.Collections`工具类还提供了一个旧的线程安全集合转换器, 用法示例: `Map threadSafeMap = Collections.synchronizedMap(new HashMap());`
  - 但是它实际上是用一个包装类包装了非线程安全的`Map`，然后对所有读写方法都用`synchronized`加锁，这样获得的线程安全集合的性能比`java.util.concurrent`集合要低很多，所以不推荐使用。



- 尽量使用Java标准库提供的并发集合，避免自己编写同步代码。



## Atomic

- Java的`java.util.concurrent`包除了提供底层锁、并发集合外，还提供了一组原子操作的封装类，它们位于`java.util.concurrent.atomic`包。

  - 以`AtomicInteger`为例，它提供的主要操作有：

    - 增加值并返回新值：`int addAndGet(int delta)`
    - 加1后返回新值：`int incrementAndGet()`
    - 获取当前值：`int get()`
    - 用CAS方式设置：`int compareAndSet(int expect, int update)`

  - Atomic类是通过**无锁（lock-free）的方式实现的线程安全（thread-safe）访问**。它的主要原理是利用了**CAS：Compare and Set**。

    - 如果我们自己通过CAS编写`incrementAndGet()`，它大概长这样：

      ```java
      public int incrementAndGet(AtomicInteger var) {
          int prev, next;
          do {
              prev = var.get();
              next = prev + 1;
          } while ( ! var.compareAndSet(prev, next));			
          return next;
      }
      ```

    - CAS是指，在这个操作中，

      - 如果`AtomicInteger`的当前值是`prev`，那么就更新为`next`，返回`true`。
      - 如果`AtomicInteger`的当前值不是`prev`，就什么也不干，返回`false`。
      - 通过CAS操作并配合`do ... while`循环，即使其他线程修改了`AtomicInteger`的值，最终的结果也是正确的。



- 我们利用`AtomicLong`可以编写一个多线程安全的全局唯一ID生成器：

  ```java
  class IdGenerator {
      AtomicLong var = new AtomicLong(0);
  
      public long getNextId() {
          return var.incrementAndGet();
      }
  }
  ```

  

- 通常情况下，我们并不需要直接用`do ... while`循环调用`compareAndSet`实现复杂的并发操作，而是用`incrementAndGet()`这样的封装好的方法，因此，使用起来非常简单。





## 线程池

- Java标准库提供了`ExecutorService`接口表示线程池，它的典型用法如下：

  ```java
  // 创建固定大小的线程池:
  ExecutorService executor = Executors.newFixedThreadPool(3);
  // 提交任务:
  executor.submit(task1);
  executor.submit(task2);
  executor.submit(task3);
  executor.submit(task4);
  executor.submit(task5);
  ```



- 因为`ExecutorService`只是接口，Java标准库提供的几个常用实现类有：
  - FixedThreadPool：线程数固定的线程池；
    - n个n个的执行任务
  - CachedThreadPool：线程数根据任务动态调整的线程池；
    - 扩容到任务数后一同执行
  - SingleThreadExecutor：仅单线程执行的线程池。



- 线程池在程序结束的时候要关闭。使用`shutdown()`方法关闭线程池的时候，它会等待正在执行的任务先完成，然后再关闭。`shutdownNow()`会立刻停止正在执行的任务，`awaitTermination()`则会等待指定的时间让线程池关闭。



- 限制线程数量范围的动态线程池:

  ```java
  int min = 4;
  int max = 10;
  ExecutorService es = new ThreadPoolExecutor(min, max,
          60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
  ```

  

- ScheduledThreadPool

  - 还有一种任务，需要定期反复执行，例如，每秒刷新证券价格。这种任务本身固定，需要反复执行的，可以使用`ScheduledThreadPool`。放入`ScheduledThreadPool`的任务可以定期反复执行。

    ```java
    ScheduledExecutorService ses = Executors.newScheduledThreadPool(4);
    // 1秒后执行一次性任务:
    ses.schedule(new Task("one-time"), 1, TimeUnit.SECONDS);		// Task implements Runnable
    // 2秒后开始执行定时任务，每3秒执行:
    ses.scheduleAtFixedRate(new Task("fixed-rate"), 2, 3, TimeUnit.SECONDS);	
    // 2秒后开始执行定时任务，以3秒为间隔执行:
    ses.scheduleWithFixedDelay(new Task("fixed-delay"), 2, 3, TimeUnit.SECONDS);
    ```

    

  - 注意FixedRate和FixedDelay的区别。FixedRate是指任务总是以固定时间间隔触发，不管任务执行多长时间; 而FixedDelay是指，上一次任务执行完毕后，等待固定的时间间隔，再执行下一次任务.



## Future

- 使用Java标准库提供的线程池非常方便, 提交的任务只需要实现`Runnable`接口，就可以让线程池去执行.

  - 但是, `Runnable`接口有个问题，它的方法没有返回值。如果任务需要一个返回结果，那么只能保存到变量，还要提供额外的方法读取，非常不便。所以，Java标准库还提供了一个`Callable`接口，和`Runnable`接口比，它多了一个返回值:

  ```java
  class Task implements Callable<String> {
      public String call() throws Exception {
          return longTimeCalculation(); 
      }
  }
  ```

  - 并且`Callable`接口是一个泛型接口，可以返回指定类型的结果。

    

- 现在的问题是，如何获得异步执行的结果？

  - 如果仔细看`ExecutorService.submit()`方法，可以看到，它返回了一个`Future`类型，一个`Future`类型的实例代表一个未来能获取结果的对象

    ```java
    ExecutorService executor = Executors.newFixedThreadPool(4); 
    // 定义任务:
    Callable<String> task = new Task();
    // 提交任务并获得Future:
    Future<String> future = executor.submit(task);
    // 从Future获取异步执行返回的结果:
    String result = future.get(); // 可能阻塞
    ```

    

  - 当我们提交一个`Callable`任务后，我们会同时获得一个`Future`对象，然后，我们在主线程某个时刻调用`Future`对象的`get()`方法，就可以获得异步执行的结果。在调用`get()`时，**如果异步任务已经完成，我们就直接获得结果。如果异步任务还没有完成，那么`get()`会阻塞，直到任务完成后才返回结果。**



- 一个`Future<V>`接口表示一个未来可能会返回的结果 (V是结果的类型)，它定义的方法有：
  - `get()`：获取结果（可能会等待）
  - `get(long timeout, TimeUnit unit)`：获取结果，但只等待指定的时间；
  - `cancel(boolean mayInterruptIfRunning)`：取消当前任务；
  - `isDone()`：判断任务是否已完成。



## Completable Future

- 使用`Future`获得异步执行结果时，要么调用阻塞方法`get()`，要么轮询看`isDone()`是否为`true`，这两种方法都不是很好，因为主线程也会被迫等待。

- 从Java 8开始引入了`CompletableFuture`，它针对`Future`做了改进，**可以传入回调对象，当异步任务完成或者发生异常时，自动调用回调对象的回调方法。**

  ```java
  public class Main {
      public static void main(String[] args) throws Exception {
          // 创建异步执行任务:
          CompletableFuture<Double> cf = CompletableFuture.supplyAsync(Main::fetchPrice);
          // 如果执行成功:
          cf.thenAccept((result) -> {								// 设置consumer对象
              System.out.println("price: " + result);
          });
          // 如果执行异常:												// 设置Function对象
          cf.exceptionally((e) -> {
              e.printStackTrace();
              return null;
          });
          // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
          Thread.sleep(200);
      }
  
      static Double fetchPrice() {
          try {
              Thread.sleep(100);
          } catch (InterruptedException e) {
          }
          if (Math.random() < 0.3) {
              throw new RuntimeException("fetch price failed!");
          }
          return 5 + Math.random() * 20;
      }
  }
  
  ```

  

- 创建一个`CompletableFuture`是通过`CompletableFuture.supplyAsync()`实现的，它需要一个实现了`Supplier`接口的对象：

  - ```java
    public interface Supplier<T> {
        T get();
    }
    ```

  - 这里我们用lambda语法简化了一下，直接传入`Main::fetchPrice`，因为`Main.fetchPrice()`静态方法的签名符合`Supplier`接口的定义（除了方法名外）。 (??)

- 紧接着，`CompletableFuture`已经被提交给默认的线程池执行了，我们需要定义的是`CompletableFuture`完成时和异常时需要回调的实例。

  - 完成时，`CompletableFuture`会调用`Consumer`对象：

    ```java
    public interface Consumer<T> {
        void accept(T t);
    }
    ```

  - 异常时，`CompletableFuture`会调用`Function`对象：

    ```java
    public interface Function<T, R> {
        R apply(T t);
    }
    ```



- 可见`CompletableFuture`的优点是：
  - 异步任务结束时，会自动回调某个对象的方法；
  - 异步任务出错时，会自动回调某个对象的方法；
  - 主线程设置好回调后，不再关心异步任务的执行。



- `CompletableFuture`更强大的功能是，多个`CompletableFuture`可以串行执行

  - 例如，定义两个`CompletableFuture`，第一个`CompletableFuture`根据证券名称查询证券代码，第二个`CompletableFuture`根据证券代码查询证券价格

    ```java
    public class Main {
        public static void main(String[] args) throws Exception {
            // 第一个任务:
            CompletableFuture<String> cfQuery = CompletableFuture.supplyAsync(() -> {
                return queryCode("中国石油");
            });
            // cfQuery成功后继续执行下一个任务:
            CompletableFuture<Double> cfFetch = cfQuery.thenApplyAsync((code) -> {
                return fetchPrice(code);
            });
            // cfFetch成功后打印结果:
            cfFetch.thenAccept((result) -> {
                System.out.println("price: " + result);
            });
            // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
            Thread.sleep(2000);
        }
    
        static String queryCode(String name) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
            }
            return "601857";
        }
    
        static Double fetchPrice(String code) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
            }
            return 5 + Math.random() * 20;
        }
    }
    
    ```

    

- 除了串行执行外，多个`CompletableFuture`还可以并行执行。

  - 例如, 同时从新浪和网易查询证券代码，只要任意一个返回结果，就进行下一步查询价格，查询价格也同时从新浪和网易查询，只要任意一个返回结果，就完成操作

    ```java
    public class Main {
        public static void main(String[] args) throws Exception {
            // 两个CompletableFuture执行异步查询:
            CompletableFuture<String> cfQueryFromSina = CompletableFuture.supplyAsync(() -> {
                return queryCode("中国石油", "https://finance.sina.com.cn/code/");
            });
            CompletableFuture<String> cfQueryFrom163 = CompletableFuture.supplyAsync(() -> {
                return queryCode("中国石油", "https://money.163.com/code/");
            });
    
            // 用anyOf合并为一个新的CompletableFuture:
            CompletableFuture<Object> cfQuery = CompletableFuture.anyOf(cfQueryFromSina, cfQueryFrom163);
    
            // 两个CompletableFuture执行异步查询:
            CompletableFuture<Double> cfFetchFromSina = cfQuery.thenApplyAsync((code) -> {
                return fetchPrice((String) code, "https://finance.sina.com.cn/price/");
            });
            CompletableFuture<Double> cfFetchFrom163 = cfQuery.thenApplyAsync((code) -> {
                return fetchPrice((String) code, "https://money.163.com/price/");
            });
    
            // 用anyOf合并为一个新的CompletableFuture:
            CompletableFuture<Object> cfFetch = CompletableFuture.anyOf(cfFetchFromSina, cfFetchFrom163);
    
            // 最终结果:
            cfFetch.thenAccept((result) -> {
                System.out.println("price: " + result);
            });
            // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
            Thread.sleep(200);
        }
    
        static String queryCode(String name, String url) {
            System.out.println("query code from " + url + "...");
            try {
                Thread.sleep((long) (Math.random() * 100));
            } catch (InterruptedException e) {
            }
            return "601857";
        }
    
        static Double fetchPrice(String code, String url) {
            System.out.println("query price from " + url + "...");
            try {
                Thread.sleep((long) (Math.random() * 100));
            } catch (InterruptedException e) {
            }
            return 5 + Math.random() * 20;
        }
    }
    
    ```

    

- 总结: `CompletableFuture`可以指定异步处理流程：
  - `thenAccept()`处理正常结果；
  - `exceptional()`处理异常结果；
  - `thenApplyAsync()`用于串行化另一个`CompletableFuture`；
  - `anyOf()`和`allOf()`用于并行化多个`CompletableFuture`。



## ThreadLocal

- 在一个线程中，横跨若干方法调用，需要传递的对象，我们通常称之为上下文（Context），它是一种状态，可以是用户身份、任务信息等。

  - 给每个方法增加一个context参数非常麻烦，而且有些时候，如果调用链有无法修改源码的第三方库，`User`对象就传不进去了。

- Java标准库提供了一个特殊的`ThreadLocal`，它可以在一个线程中传递同一个对象。

  - `ThreadLocal`实例通常总是以静态字段初始化如下：

    ```java
    static ThreadLocal<User> threadLocalUser = new ThreadLocal<>();
    ```

  - 它的典型使用方式如下：

    ```java
    void processUser(user) {
        try {
            threadLocalUser.set(user);
            step1();
            step2();
        } finally {
            threadLocalUser.remove();
        }
    }
    ```

  - 通过设置一个`User`实例关联到`ThreadLocal`中，在移除之前，所有方法都可以随时获取到该`User`实例：

    ```java
    // 普通的方法调用一定是同一个线程执行的，所以，
    // step1()、step2()以及log()方法内，threadLocalUser.get()获取的User对象是同一个实例。
    void step1() {
        User u = threadLocalUser.get();
        log();
        printUser();
    }
    
    void log() {
        User u = threadLocalUser.get();
        println(u.name);
    }
    
    void step2() {
        User u = threadLocalUser.get();
        checkUser(u.id);
    }
    ```

    

- 实际上，可以把`ThreadLocal`看成一个全局`Map<Thread, Object>`：每个线程获取`ThreadLocal`变量时，总是使用`Thread`自身作为key
  - 因此，`ThreadLocal`相当于给每个线程都开辟了一个独立的存储空间，各个线程的`ThreadLocal`关联的实例互不干扰。



- 特别注意`ThreadLocal`一定要在`finally`中清除. 这是因为当前线程执行完相关代码后，很可能会被重新放入线程池中，如果`ThreadLocal`没有被清除，该线程执行其他代码时，会把上一次的状态带进去。





# 函数式编程

## Lambda表达式

- 在Java程序中，我们经常遇到一大堆单方法接口，即一个接口只定义了一个方法：
  - Comparator, Runnable, Callable

- 以`Comparator`为例，我们想要调用`Arrays.sort()`时，可以传入一个`Comparator`实例, 以匿名类方式编写

  - ```java
    String[] array = ...
    Arrays.sort(array, new Comparator<String>() {
        public int compare(String s1, String s2) {
            return s1.compareTo(s2);
        }
    });
    ```

    

  - 上述写法非常繁琐。从Java 8开始，我们可以用Lambda表达式替换单方法接口。改写上述代码如下：

    ```java
    String[] array = ...
    Arrays.sort(array, (s1, s2) -> {
        return s1.compareTo(s2);
    });
    ```

    

- 观察Lambda表达式的写法，它只需要写出方法定义.
  - 其中，参数是`(s1, s2)`，参数类型可以省略，因为编译器可以自动推断出`String`类型。`-> { ... }`表示方法体，所有代码写在内部即可。Lambda表达式没有`class`定义，因此写法非常简洁。
  - 如果只有一行`return xxx`的代码，完全可以用更简单的写法：
    - `Arrays.sort(array, (s1, s2) -> s1.compareTo(s2));`
    - 返回值的类型也是由编译器自动推断的，这里推断出的返回值是`int`，因此，只要返回`int`，编译器就不会报错。
  - Lambda表达式的参数和返回值均可由编译器自动推断。



- Functional Interface

  - 只定义了单方法的接口称之为`FunctionalInterface`. 所谓单方法, 指的是只有单个抽象方法. 可以有其他的default或static方法.

  - 接收`FunctionalInterface`作为参数的时候，可以把实例化的匿名类改写为Lambda表达式，能大大简化代码。

    



## 方法引用

- 除了Lambda表达式，我们还可以直接传入方法引用。

  - 代码在`Arrays.sort()`中直接传入了静态方法`cmp`的引用，用`Main::cmp`表示。

  - ```java
    ...
    	Arrays.sort(array, Main::cmp);
    ...
        
        static int cmp(String s1, String s2) {
            return s1.compareTo(s2);
        }
    ```



- 所谓方法引用，是指如果某个方法签名和接口恰好一致，就可以直接传入方法引用。
  - 因为`Comparator<String>`接口定义的方法是`int compare(String, String)`，和静态方法`int cmp(String, String)`相比，除了方法名外，方法参数一致，返回类型相同，因此，我们说两者的方法签名一致，可以直接把方法名作为Lambda表达式传入.
  - 注意：在这里，方法签名**只看参数类型和返回类型**，不看方法名称，也不看类的继承关系。

​	

- 实例方法也可通过方法引用来使用. 实例方法作为方法引用时, 认为其第一个参数为this, 因此其方法签名中第一个参数的类型为实例本身的数据类型.



- 构造方法引用: Person是一个类, 具有`String name`这一field. 现有一个`List<String>`存name, 想转换成`List<Person>`

  - 要简单地实现`String`到`Person`的转换，我们可以引用`Person`的构造方法

    ```java
    List<String> names = List.of("Bob", "Alice", "Tim");
    List<Person> persons = names.stream().map(Person::new).collect(Collectors.toList());
    ```

  - 这里的`map()`需要传入的FunctionalInterface的定义是:

    - ```java
      @FunctionalInterface
      public interface Function<T, R> {
          R apply(T t);
      }
      ```

    - 把泛型对应上就是方法签名`Person apply(String)`，即传入参数`String`，返回类型`Person`。

    - 而`Person`类的构造方法恰好满足这个条件，因为构造方法的参数是`String`，而构造方法虽然没有`return`语句，但它会隐式地返回`this`实例，类型就是`Person`，因此，此处可以引用构造方法。

    - 构造方法的引用写法是`类名::new`，因此，此处传入`Person::new`。

  

  

- 总结

  - `FunctionalInterface`允许传入：
    - 接口的实现类（传统写法，代码较繁琐）；
    - Lambda表达式（只需列出参数名，由编译器推断类型）；
    - 符合方法签名的静态方法；
    - 符合方法签名的实例方法（实例类型被看做第一个参数类型）；
    - 符合方法签名的构造方法（实例类型被看做返回类型）。
  - `FunctionalInterface`
    - 不强制继承关系，
    - 不需要方法名称相同，
    - 只要求方法参数（类型和数量）与方法返回类型相同，即认为方法签名相同。



## Stream

- Java从8开始，不但引入了Lambda表达式，还引入了一个全新的流式API：Stream API。它位于`java.util.stream`包中。
  - 这个`Stream`不同于`java.io`的`InputStream`和`OutputStream`，它代表的是任意Java对象的序列。
  - 这个`Stream`和`List`也不一样，`List`存储的每个元素都是已经存储在内存中的某个Java对象，而`Stream`输出的元素可能并没有预先存储在内存中，而是实时计算出来的。
    - `List`的用途是操作一组已存在的Java对象，而`Stream`实现的是惰性计算
    - `List`元素已分配并存储在内存, `Stream`元素可能未分配需实时计算
    - `List`用于操作一组已存在的Java对象, `Stream`用于惰性计算



- 惰性计算的特点是：一个`Stream`转换为另一个`Stream`时，实际上只存储了转换规则，并没有任何计算发生。

  - 例如，创建一个全体自然数的`Stream`，不会进行计算，把它转换为上述`s2`这个`Stream`，也不会进行计算。再把`s2`这个无限`Stream`转换为`s3`这个有限的`Stream`，也不会进行计算。只有最后，调用`forEach`确实需要`Stream`输出的元素时，才进行计算。

    ```java
    Stream<BigInteger> naturals = createNaturalStream(); // 不计算
    Stream<BigInteger> s2 = naturals.map(BigInteger::multiply); // 不计算
    Stream<BigInteger> s3 = s2.limit(100); // 不计算
    s3.forEach(System.out::println); // 计算
    ```

  - 我们通常把`Stream`的操作写成链式操作，代码更简洁：

    ```java
    createNaturalStream()
        .map(BigInteger::multiply)
        .limit(100)
        .forEach(System.out::println);
    ```



- 因此，Stream API的基本用法就是：创建一个`Stream`，然后做若干次转换，最后调用一个求值方法获取真正计算的结果：

  ```java
  int result = createNaturalStream() // 创建Stream
               .filter(n -> n % 2 == 0) // 任意个转换
               .map(n -> n * n) // 任意个转换
               .limit(100) // 任意个转换
               .sum(); // 最终计算结果
  ```



### 创建stream

- `Stream.of()`

  - 创建`Stream`最简单的方式是直接用`Stream.of()`静态方法，传入可变参数即创建了一个能输出确定元素的`Stream`

  - 虽然这种方式基本上没啥实质性用途，但测试的时候很方便。

  - ```java
    Stream<String> stream = Stream.of("A", "B", "C", "D");
    // forEach()方法相当于内部循环调用，
    // 可传入符合Consumer接口的void accept(T t)的方法引用：
    stream.forEach(System.out::println);
    ```

    

- 基于数组或Collection

  - 第二种创建`Stream`的方法是基于一个数组或者`Collection`，这样该`Stream`输出的元素就是数组或者`Collection`持有的元素

  - ```java
    public class Main {
        public static void main(String[] args) {
            Stream<String> stream1 = Arrays.stream(new String[] { "A", "B", "C" });
            Stream<String> stream2 = List.of("X", "Y", "Z").stream();
            stream1.forEach(System.out::println);
            stream2.forEach(System.out::println);
        }
    }
    ```

  - 把数组变成`Stream`使用`Arrays.stream()`方法。对于`Collection`（`List`、`Set`、`Queue`等），直接调用`stream()`方法就可以获得`Stream`。

  - 上述创建`Stream`的方法都是把一个现有的序列变为`Stream`，它的元素是固定的。



- 基于Supplier

  - 创建`Stream`还可以通过`Stream.generate()`方法，它需要传入一个`Supplier`对象：

    - `Stream<String> s = Stream.generate(Supplier<String> sp);`

  - 基于`Supplier`创建的`Stream`会不断调用`Supplier.get()`方法来不断产生下一个元素，这种`Stream`保存的不是元素，而是算法，它可以用来表示无限序列。

    - 例如，我们编写一个能不断生成自然数的`Supplier`，它的代码非常简单，每次调用`get()`方法，就生成下一个自然数：

    - ```java
      public class Main {
          public static void main(String[] args) {
              Stream<Integer> natual = Stream.generate(new NatualSupplier());
              // 注意：无限序列必须先变成有限序列再打印:
              natual.limit(20).forEach(System.out::println);
          }
      }
      
      class NatualSupplier implements Supplier<Integer> {
          int n = 0;
          public Integer get() {
              n++;
              return n;
          }
      }
      ```

    - 对于无限序列，如果直接调用`forEach()`或者`count()`这些最终求值操作，会进入死循环，因为永远无法计算完这个序列，所以正确的方法是先把无限序列变成有限序列，例如，用`limit()`方法可以截取前面若干个元素，这样就变成了一个有限序列，对这个有限序列调用`forEach()`或者`count()`操作就没有问题。



- 其他方法
  - 创建`Stream`的第三种方法是通过一些API提供的接口，直接获得`Stream`。
  - 例如，`Files`类的`lines()`方法可以把一个文件变成一个`Stream`，每个元素代表文件的一行内容.
  - 另外，正则表达式的`Pattern`对象有一个`splitAsStream()`方法，可以直接把一个长字符串分割成`Stream`序列而不是数组.



- 基本类型

  - 因为Java的范型不支持基本类型，所以我们无法用`Stream<int>`这样的类型，会发生编译错误。为了保存`int`，只能使用`Stream<Integer>`，但这样会产生频繁的装箱、拆箱操作。为了提高效率，Java标准库提供了`IntStream`、`LongStream`和`DoubleStream`这三种使用基本类型的`Stream`，它们的使用方法和范型`Stream`没有大的区别，设计这三个`Stream`的目的是提高运行效率：

  - ```java
    // 将int[]数组变为IntStream:
    IntStream is = Arrays.stream(new int[] { 1, 2, 3 });
    // 将Stream<String>转换为LongStream:
    LongStream ls = List.of("1", "2", "3").stream().mapToLong(Long::parseLong);
    ```



### map

- `Stream.map()`是`Stream`最常用的一个转换方法，它把一个`Stream`转换为另一个`Stream`。

  - 所谓`map`操作，就是把一种操作运算，映射到一个序列的每一个元素上。例如，对`x`计算它的平方，可以使用函数`f(x) = x * x`。我们把这个函数映射到一个序列1，2，3，4，5上，就得到了另一个序列1，4，9，16，25.

  - 可见，`map`操作，把一个`Stream`的每个元素一一对应到应用了目标函数的结果上。

  - ```java
    Stream<Integer> s = Stream.of(1, 2, 3, 4, 5);
    Stream<Integer> s2 = s.map(n -> n * n);
    ```

    

- 如果我们查看`Stream`的源码，会发现`map()`方法接收的对象是`Function`接口对象，它定义了一个`apply()`方法，负责把一个`T`类型转换成`R`类型：

  - ```java
    Stream<R> map(Function<? super T, ? extends R> mapper);
    ```

  - 其中，`Function`的定义是：

    ```java
    @FunctionalInterface
    public interface Function<T, R> {
        // 将T类型转换为R:
        R apply(T t);
    }
    ```

    

- 利用`map()`，不但能完成数学计算，对于字符串操作，以及任何Java对象都是非常有用的。

  ```java
  public class Main {
      public static void main(String[] args) {
          List.of("  Apple ", " pear ", " ORANGE", " BaNaNa ")
                  .stream()
                  .map(String::trim) // 去空格
                  .map(String::toLowerCase) // 变小写
                  .forEach(System.out::println); // 打印
      }
  }
  ```

  

### filter

- `Stream.filter()`是`Stream`的另一个常用转换方法。

  - 所谓`filter()`操作，就是对一个`Stream`的所有元素一一进行测试，不满足条件的就被“滤掉”了，剩下的满足条件的元素就构成了一个新的`Stream`。

  - 例如，我们对1，2，3，4，5这个`Stream`调用`filter()`，传入的测试函数`f(x) = x % 2 != 0`用来判断元素是否是奇数，这样就过滤掉偶数，只剩下奇数，因此我们得到了另一个序列1，3，5.

  - 用IntStream写出上述逻辑，代码如下：

    ```java
    public class Main {
        public static void main(String[] args) {
            IntStream.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
                    .filter(n -> n % 2 != 0)
                    .forEach(System.out::println);
        }
    }
    ```

  - 从结果可知，经过`filter()`后生成的`Stream`元素可能变少。



- `filter()`方法接收的对象是`Predicate`接口对象，它定义了一个`test()`方法，负责判断元素是否符合条件：

  ```java
  @FunctionalInterface
  public interface Predicate<T> {
      // 判断元素t是否符合条件:
      boolean test(T t);
  }
  ```

  

- `filter()`除了常用于数值外，也可应用于任何Java对象。例如，从一组给定的`LocalDate`中过滤掉工作日，以便得到休息日：

  ```java
  public class Main {
      public static void main(String[] args) {
          Stream.generate(new LocalDateSupplier())
                  .limit(31)
                  .filter(ldt -> ldt.getDayOfWeek() == DayOfWeek.SATURDAY || ldt.getDayOfWeek() == DayOfWeek.SUNDAY)
                  .forEach(System.out::println);
      }
  }
  
  class LocalDateSupplier implements Supplier<LocalDate> {
      LocalDate start = LocalDate.of(2020, 1, 1);
      int n = -1;
      public LocalDate get() {
          n++;
          return start.plusDays(n);
      }
  }
  ```

  

### Reduce

- `map()`和`filter()`都是`Stream`的转换方法，而`Stream.reduce()`则是`Stream`的一个聚合方法，它可以把一个`Stream`的所有元素按照聚合函数聚合成一个结果。

  

- `reduce()`方法传入的对象是`BinaryOperator`接口，它定义了一个`apply()`方法，负责把上次累加的结果和本次的元素 进行运算，并返回累加的结果：

  - ```java
    @FunctionalInterface
    public interface BinaryOperator<T> {
        // Bi操作：两个输入，一个输出
        T apply(T t, T u);
    }
    ```

  - 可理解为:

    ```java
    Stream<Integer> stream = ...
    int sum = 0;	// reduce()操作首先初始化结果为指定值（这里是0）
    // 紧接着，reduce()对每个元素依次调用(sum, n) -> sum + n，其中，sum是上次计算的结果
    for (n : stream) {
        sum = (sum, n) -> sum + n;
    }
    ```

    

- 如果去掉初始值，我们会得到一个`Optional<Integer>`：

  - ```java
    Optional<Integer> opt = stream.reduce((acc, n) -> acc + n);
    if (opt.isPresent()) {
        System.out.println(opt.get());
    }
    // 这是因为Stream的元素有可能是0个，这样就没法调用reduce()的聚合函数了，因此返回Optional对象，需要进一步判断结果是否存在。
    ```

    

- 求积:

  ```java
  public class Main {
      public static void main(String[] args) {
          int s = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9).reduce(1, (acc, n) -> acc * n); 
          // 注意：计算求积时，初始值必须设置为1。
          System.out.println(s); // 362880
      }
  }
  ```

  



- 除了可以对数值进行累积计算外，灵活运用`reduce()`也可以对Java对象进行操作。下面的代码演示了如何将配置文件的每一行配置通过`map()`和`reduce()`操作聚合成一个`Map<String, String>`：

  ```java
  public class Main {
      public static void main(String[] args) {
          // 按行读取配置文件:
          List<String> props = List.of("profile=native", "debug=true", "logging=warn", "interval=500");
          Map<String, String> map = props.stream()
                  // 把k=v转换为Map[k]=v:
                  .map(kv -> {
                      String[] ss = kv.split("\\=", 2);
                      return Map.of(ss[0], ss[1]);
                  })
                  // 把所有Map聚合到一个Map:
                  .reduce(new HashMap<String, String>(), (m, kv) -> {
                      m.putAll(kv);
                      return m;
                  });
          // 打印结果:
          map.forEach((k, v) -> {
              System.out.println(k + " = " + v);
          });
      }
  }
  ```

  





