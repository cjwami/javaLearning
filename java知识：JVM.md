## **java知识：JVM**

### 1、平台无关性如何实现

![1560065354461](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1560065354461.png)



### 2、JVM如何加载 .class 文件

![1560065682407](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1560065682407.png)

### 3、反射

Java反射机制：

​       在运行状态时，对于任意一个类，都***能够知道这个类的所有属性和方法***；对于任意一个对象，都***能够调用它的任意方法和属性***。这种动态获取信息以及动态调用对象方法的功能叫做反射。

例子：

![1560066047448](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1560066047448.png)

### 4、谈谈classLoader

从 Java 虚拟机的角度来讲，只存在以下两种不同的类加载器：

- 启动类加载器（Bootstrap ClassLoader），使用 C++ 实现，是虚拟机自身的一部分；
- 所有其它类的加载器，使用 Java 实现，独立于虚拟机，继承自抽象类 java.lang.ClassLoader。

从 Java 开发人员的角度看，类加载器可以划分得更细致一些：

- 启动类加载器（Bootstrap ClassLoader）此类加载器负责将存放在 <JRE_HOME>\lib 目录中的，或者被 -Xbootclasspath 参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如 rt.jar，名字不符合的类库即使放在 lib 目录中也不会被加载）类库加载到虚拟机内存中。启动类加载器无法被 Java 程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给启动类加载器，直接使用 null 代替即可。
- 扩展类加载器（Extension ClassLoader）这个类加载器是由 ExtClassLoader（sun.misc.Launcher$ExtClassLoader）实现的。它负责将 <JAVA_HOME>/lib/ext 或者被 java.ext.dir 系统变量所指定路径中的所有类库加载到内存中，开发者可以直接使用扩展类加载器。
- 应用程序类加载器（Application ClassLoader）这个类加载器是由 AppClassLoader（sun.misc.Launcher$AppClassLoader）实现的。由于这个类加载器是 ClassLoader 中的 getSystemClassLoader() 方法的返回值，因此一般称为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

### 5、类加载器的双亲委派机制

​        应用程序是由三种类加载器互相配合从而实现类加载，除此之外还可以加入自己定义的类加载器。

​        下图展示了类加载器之间的层次关系，称为双亲委派模型（Parents Delegation Model）。该模型要求除了顶层的启动类加载器外，其它的类加载器都要有自己的父类加载器。这里的父子关系一般通过组合关系（Composition）来实现，而不是继承关系（Inheritance）。

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1560066169237.png" style="zoom:60%">

#### （1）工作过程

​		一个类加载器首先将类加载请求转发到父类加载器，只有当父类加载器无法完成时才尝试自己加载。

#### （2）好处

​		使得 Java 类随着它的类加载器一起具有一种带有优先级的层次关系，从而使得基础类得到统一。

​		例如 java.lang.Object 存放在 rt.jar 中，如果编写另外一个 java.lang.Object 并放到 ClassPath 中，程序可以编译通过。由于双亲委派模型的存在，所以在 rt.jar 中的 Object 比在 ClassPath 中的 Object 优先级更高，这是因为 rt.jar 中的 Object 使用的是启动类加载器，而 ClassPath 中的 Object 使用的是应用程序类加载器。rt.jar 中的 Object 优先级更高，那么程序中所有的 Object 都是这个 Object。

#### （3）实现

​		以下是抽象类 java.lang.ClassLoader 的代码片段，其中的 loadClass() 方法运行过程如下：先检查类是否已经加载过，如果没有则让父类加载器去加载。当父类加载器加载失败时抛出 ClassNotFoundException，此时尝试自己去加载。

```java
public abstract class ClassLoader {
    // The parent class loader for delegation
    //用于委派的父类加载器
    private final ClassLoader parent;

    public Class<?> loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
    }

    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                    //如果找不到类，则抛出ClassNotFoundException
                    //来自非null父类加载器
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    //如果仍未找到，则按顺序调用findClass
                    //找到类
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        throw new ClassNotFoundException(name);
    }
}
```

### 6、类加载机制

​		类是在运行期间第一次使用时动态加载的，而不是一次性加载所有类。因为如果一次性加载，那么会占用很多的内存。

#### （1）类的生命周期

![1560067212637](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1560067212637.png)

​		包括以下 7 个阶段：

- <Font color=red>**加载（Loading）**</Font>
- <Font color=blue>**验证（Verification）**</Font>
- <Font color=blue>**准备（Preparation）**</Font>
- <Font color=blue>**解析（Resolution）**</Font>
- <Font color=orange>**初始化（Initialization）**</Font>
- 使用（Using）
- 卸载（Unloading）

#### （2）类加载过程

​		包含了加载、<Font color=blue>验证、准备、解析（此三个阶段为“链接”）</Font>和初始化这 5 个阶段。

##### 		<Font color=red>1）加载</Font>

​		加载是类加载的一个阶段，注意不要混淆。主要工作是<Font color=red>**通过classLoader加载class文件字节码，生成class对象**</Font>。

​		加载过程完成以下三件事：

- 通过类的完全限定名称获取定义该类的二进制字节流。

- 将该字节流表示的静态存储结构转换为方法区的运行时存储结构。

- 在内存中生成一个代表该类的 Class 对象，作为方法区中该类各种数据的访问入口。

  其中二进制字节流可以从以下方式中获取：

- 从 ZIP 包读取，成为 JAR、EAR、WAR 格式的基础。

- 从网络中获取，最典型的应用是 Applet。

- 运行时计算生成，例如动态代理技术，在 java.lang.reflect.Proxy 使用 ProxyGenerator.generateProxyClass 的代理类的二进制字节流。

- 由其他文件生成，例如由 JSP 文件生成对应的 Class 类。

  ##### <Font color=blue>2）验证（校验）</Font>

​        检验加载的class的正确性和安全性，确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

##### 		<Font color=blue>3）准备</Font>

​		***为类变量分配存储空间，并设置类变量初始值。***

​		类变量是被 static 修饰的变量，准备阶段为类变量分配内存并设置初始值，使用的是方法区的内存。

​		实例变量不会在这阶段分配内存，它会在对象实例化时随着对象一起被分配在堆中。应该注意到，实例化不是类加载的一个过程，类加载发生在所有实例化操作之前，并且类加载只进行一次，实例化可以进行多次。

​		初始值一般为 0 值，例如下面的类变量 value 被初始化为 0 而不是 123。

```java
public static int value = 123;
```

​		如果类变量是常量，那么它将初始化为表达式所定义的值而不是 0。例如下面的常量 value 被初始化为 123 而不是 0。

```java
public static final int value = 123;
```

##### 		<Font color=blue>4）解析</Font>

​		***JVM将常量池中的符号引用转换为直接引用***。其中解析过程在某些情况下可以在初始化阶段之后再开始，这是为了支持 Java 的动态绑定。

##### 		<Font color=orange>5）初始化</Font>

​		***执行类变量赋值和静态代码块***。

​		初始化阶段才真正开始执行类中定义的 Java 程序代码。初始化阶段是虚拟机执行类构造器 <clinit>() 方法的过程。在准备阶段，类变量已经赋过一次系统要求的初始值，而在初始化阶段，根据程序员通过程序制定的主观计划去初始化类变量和其它资源。

​		<clinit>() 是由编译器自动收集类中所有类变量的赋值动作和静态语句块中的语句合并产生的，编译器收集的顺序由语句在源文件中出现的顺序决定。特别注意的是，静态语句块只能访问到定义在它之前的类变量，定义在它之后的类变量只能赋值，不能访问。例如以下代码：

```java
public class Test {
    static {
        i = 0;                // 给变量赋值可以正常编译通过
        System.out.print(i);  // 这句编译器会提示“非法向前引用”
    }
    static int i = 1;
}
```

​		由于父类的 <clinit>() 方法先执行，也就意味着父类中定义的静态语句块的执行要优先于子类。例如以下代码：

```java
static class Parent {
    public static int A = 1;
    static {
        A = 2;
    }
}

static class Sub extends Parent {
    public static int B = A;
}

public static void main(String[] args) {
     System.out.println(Sub.B);  // 2
}
```

​		接口中不可以使用静态语句块，但仍然有类变量初始化的赋值操作，因此接口与类一样都会生成 <clinit>() 方法。但接口与类不同的是，执行接口的 <clinit>() 方法不需要先执行父接口的 <clinit>() 方法。只有当父接口中定义的变量使用时，父接口才会初始化。另外，接口的实现类在初始化时也一样不会执行接口的 <clinit>() 方法。

​		虚拟机会保证一个类的 <clinit>() 方法在多线程环境下被正确的加锁和同步，如果多个线程同时初始化一个类，只会有一个线程执行这个类的 <clinit>() 方法，其它线程都会阻塞等待，直到活动线程执行 <clinit>() 方法完毕。如果在一个类的 <clinit>() 方法中有耗时的操作，就可能造成多个线程阻塞，在实际过程中此种阻塞很隐蔽。

​		***Class.forname得到的class是已经初始化完成的；classLoader.loadClass的道德的class是还未链接的。***

### 7、Java内存模型

​		**运行时数据区域**

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1560068834942.png" style="zoom:50%">

#### （1）程序计数器

- 当前线程所执行的字节码行号指示器，***记录正在执行的虚拟机字节码指令的地址***（如果正在执行的是本地方法则为空）。

- 改变计数器的值来选取下一条需要执行的字节码指令。
- 和线程是一对一的关系，即线程私有。
- 对java方法计数，如果是native方法，则计数器的值为undefined。
- 不会发生内存泄漏

#### （2）Java 虚拟机栈

​		每个 Java 方法在执行的同时会创建一个栈帧用于存储局部变量表、操作数栈、常量池引用等信息。从方法调用直至执行完成的过程，对应着一个栈帧在 Java 虚拟机栈中入栈和出栈的过程。

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1560069463838.png" style="zoom:70%">

- java方法执行的内存模型。
- 包含多个栈帧，每个栈帧包含局部变量表、操作数栈、常量池引用等信息。

> 局部变量表：包含方法执行过程的所有变量。
>
> 操作数栈：入栈、出栈、复制、交换、产生消费变量。

- 当线程请求的栈深度超过最大值，会抛出 StackOverflowError 异常。
- 栈进行动态扩展时如果无法申请到足够内存，会抛出 OutOfMemoryError 异常。

#### （3）本地方法栈

​        本地方法栈与 Java 虚拟机栈类似，它们之间的区别只不过是本地方法栈为本地方法服务。

​		本地方法一般是用其它语言（C、C++ 或汇编语言等）编写的，并且被编译为基于本机硬件和操作系统的程序，对待这些方法需要特别处理。

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1560070173091.png"  style="zoom:50%">

#### （4）元空间（MetaSpace）与永久代（PermGen）

- <Font color=red>***元空间使用本地内存。永久代使用jvm内存***</Font>

- MetaSpace与PermGen相比，具有的优势：

  > · 字符串常量池存在永久代中，容易出现性能问题和内存溢出；
  >
  > · 类和方法的信息大小难以确定，给永久代的大小指定带来困难；
  >
  > · 永久代会为GC带来不必要的复杂性；
  >
  > · 方便HotSpot与其他JVM如Jrockit的集成

#### （5）Java堆（Heap）

​		所有对象都在这里分配内存，是垃圾收集的主要区域（"GC 堆"）。

​		<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1560175812775.png"  style="zoom:35%">

​		现代的垃圾收集器基本都是采用分代收集算法，其主要的思想是针对不同类型的对象采取不同的垃圾回收算法。可以将堆分成两块：

- 新生代（Young Generation）

- 老年代（Old Generation）

  堆不需要连续内存，并且可以动态增加其内存，增加失败会抛出 OutOfMemoryError 异常。

​        可以通过 -Xms 和 -Xmx 这两个虚拟机参数来指定一个程序的堆内存大小，第一个参数设置初始值，第二个参数设置最大值。

```
java -Xms1M -Xmx2M HackTheJava
```

### 8、JVM三大性能调优参数 -Xms -Xmx -Xss的含义

- -Xss：规定了每个线程虚拟机栈（堆栈）的大小
- -Xms：堆的初始值
- -Xmx：堆能够达到的最大值

### 9、Java内存模型中堆与栈的区别——内存分配策略

> 静态存储：编译时确定每个数据目标在运行时的存储空间需求。
>
> 栈式存储：数据区需求在编译时位置，运行时模块入口前确定。
>
> 堆式模型：编译时或运行时模块入口都无法确定，动态分配。

- 联系：引用对象、数组时，栈里定义变量保存堆中目标的首地址。

  <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1560220118697.png"  style="zoom:50%">

- 管理方式：栈自动释放，堆需要GC。
- 空间大小：栈<堆。
- 碎片相关：栈产生的碎片远远小于堆。
- 分配方式：栈支持静态与动态分配，而堆仅支持动态分配。

![1560327494559](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1560327494559.png)