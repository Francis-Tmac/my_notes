## 类加载
### 类加载简介
- 加载阶段：查找并且加载类的二进制数据文件（class 文件）
- 连接阶段：可以细分为三个阶段
    - 验证：主要是确保类文件的正确性，class 的版本，class 文件的魔数因子，他的作用就是 check 文件，确保class 对jvm 无安全影响。
    - 准备：为类的静态变量分配内存，并为其初始化默认值。
    - 解析：把类中的符号引用转化为直接引用。
- 初始化阶段：为类的静态变量赋予正确的初始值。

### 类的主动使用和被动使用
每个类或者接口被Java 程序首次主动使用时才会对其进行初始化，jvm 规范了6 中主动使用类的场景
1. 通过new 关键字会导致类的初始化。
2. 访问类的静态变量，包括读取和更新会导致类的初始化。
```java
public class Simple{  
    static {  
        System.out.println("i will be initialized");  
    }
    // 静态变量
    public static int x = 10;  
    
    // 静态方法
    public static void test(){  
      
    }
    // 反射初始化
    public static void main(String[] args) throws ClassNotFoundException{  
    Class.forName("com.frank.multihread.thread.Task");  
    }
}
```
这段代码直接访问变量 `x` 也会导致类的初始化。
3. 访问类的静态方法，会导致类的初始化。如上的 `test()` 方法
4. 对某个类进行反射操作，会导致类的初始化
5. 初始化子类会导致父类的初始化
```java
public class Parent {  
    static{  
        System.out.println("The parent is initialized");  
  }  
    public static int y = 100;  
}
public class Child extends Parent{  
    static{  
        System.out.println("the child will be initialized");  
  }  
    public static int x = 10;  
}
public class ActiveLoadTest {  
    public static void main(String[] args) {  
        System.out.println(Child.x);  
  }  
}
//--------------------------------------------------------------
//The parent is initialized
//the child will be initialized
//10

```
6. 启动类执行main 函数所在的类会导致初始化

#### 除了这六种情况，其余的都称为被动使用，不会导致类的加载和初始化
1. 构造某个类的数组时不会导致类的初始化，
```java
public class ActiveLoadTest {  
    public static void main(String[] args) {  
        Child[] children = new Child[10];  
         System.out.println(children.length);  
  }  
}
//--------------------------------------------------------------
//10    
```
此段代码只是开辟了一段连续的地址空间 `4byte * 10`
2. 引用类的静态常量不会导致类的初始化
```java
public class GlobalConstants {  
    static{  
        System.out.println("the GlobalConstants will be initialized.");  
  }  
  // 在其他类中使用MAX 不会导致 GlobalConstants 的初始化，静态代码块不会输出  
  public final static int MAX = 100;  
  
  // 此静态常量，使用 new 方法导致此类也被初始化  
  public final static Child child = new Child();  
}
public class ActiveLoadTest {  
    public static void main(String[] args) {  
        Child x = GlobalConstants.child;  
  }  
}
//--------------------------------------------------------------
//the GlobalConstants will be initialized.
//The parent is initialized
//the child will be initialized   
```

### 详解

```java
public class Singleton {  
      // 1  
      private static Singleton instance = new Singleton();  
  
      public static int x = 0;  
  
      public static int y;  
      // 2  
//    private static Singleton instance = new Singleton();  
  
      private Singleton(){  
            System.out.println("construct Singleton");  
            x++;  
            y++;  
      }  
      public static Singleton getInstance(){  
            System.out.println("static getInstance");  
            return instance;  
      }  
  
}

public class ActiveLoadTest {  
    public static void main(String[] args) {  
      System.out.println("begin exe main");  
      Singleton singleton = Singleton.getInstance();  
      System.out.println(singleton.x);  
      System.out.println(singleton.y);  
  }  
}
//--------------------------------------------------------------
//begin exe main
//construct Singleton
//static getInstance
//0
//1    
```
当将 instance 类变量的放在 2 出执行后输出会不一样

#### 类加载阶段
- 将class 文件中的二进制数据读取到内存中，然后将该字节 流锁代表的静态存储结构转换为方法区中运行时的数据结构，并且在堆内存中生成一个该类的 `java.lang.Class` 对象，作为访问方法区数据结构的入口。
![类被加载后在栈内存中的分布](../../../img/java/class_loader_1.jpg)

- 类加载的最终产物是堆内存中的class 对象，对用一个classLoader 来讲，不管某个类被加载多少次，对应到堆内存中的class 对象始终是同一个。
- 类加载必须通过一个全限定名（包名+类名）获取二进制数据流，但是有很多获取方式。
    1. 运行时动态生成，动态代理 java.lang.Proxy 也可以生成代理类的二进制字节流
    2. 网络获取
    3. 读取 zip 文件，jar, war包
    4. 将类的二进制数据存储在数据库的 blob 字段类型中。
    5. 运行时生成class 文件，并且动态加载。

#### 连接阶段
#####  验证
- 第一阶段文件格式验证：验证字节码是否符合Class 文件格式的规范
- 第二阶段元数据验证：对字节码描述的信息进行语义分析，以确保符合 《Java 语言规范》
    - 是否有父类除了 Object 之外，所有的类都应当有父类。
    - 这个类的父类是否继承了不允许被继承的类（final)。
    - 如果这个类不是抽象类是否实现了所有父类或接口中要求实现的所有方法。
- 第三阶段字节码验证：这个阶段是对类的方法体（Class 文件中的Code 属性）进行校验分析，保证被校验类中的方法在运行时不会做出危害虚拟机安全的行为，例如：
    - 在操作栈放置了一个 int 类型的数据，使用时却按long 类型假如本地变量表中
    - 一个子类可以复制给父类数据类型，这是安全的，但是把父类对象赋值给子类数据类型，是不合法的。
- 第四阶段符号引用验证：最后一个阶段发生在虚拟机将符号引用转换为直接引用的时候，也就是连接的第三阶段（解析阶段），符号引用可以看作是对类自身以外（常量池中的各种符号引用）的各类信息进行匹配性校验，通俗来说就是，该类是否缺少或者被禁止访问他依赖的某些外部类，方法，字段等资源。
    - 符号引用中通过字符串描述的全限定名是否能找到对应的类。
    - 在指定类中是否存在符合方法的字段描述以及简单名称锁描述的方法和字段。
    - 符号引用中的类，字段，方法的可访问性是否可被当前类访问。
- -Xverify:none 可以关闭大部分的类验证措施，以缩短虚拟机类加载的时间


##### 准备
- 通过验证过程后，开始为类变量也就是静态变量，分配内存并且设置初始值，类变量的内存会被分配到方法区中。在JDK7 后，类变量会随着Class 对象一起存放在 Java 堆中。
- 真正对类变量赋值的 putstatic 指令是在程序被编译后，存放于类构造器 `<clinit>()` 方法中，所以把 child 赋值为 `new Child()` 的动作要到类的初始化阶段才能被执行

```
// access flags 0x8
  static <clinit>()V
   L0
    LINENUMBER 11 L0
    NEW com/frank/multihread/thread/Child
    DUP
    INVOKESPECIAL com/frank/multihread/thread/Child.<init> ()V
    PUTSTATIC com/frank/multihread/thread/ActiveLoadTest.child : Lcom/frank/multihread/thread/Child;
    RETURN
    MAXSTACK = 2
    MAXLOCALS = 0
```
- 有 final 修饰的静态变量不会导致类的初始化，是一种被动引用。常量在编译阶段 javac 会将其value 生成一个 constantValue 属性，直接赋值。


##### 解析
- 符号引用：以一组符号来描述锁引用的目标，符号可以是任务形式的字面量，只要使用是能够无歧义的定位到目标即可。引用的目标不一定是已经加载到虚拟机内存当中的内容。
- 直接引用：直接指向目标的指针，相对偏移量。如果有了直接引用，那引用目标必定已经存在虚拟机的内存中。
- 解析就是在常量池中寻找类，接口，字段和方法的符号引用，并且为这些符号引用替换成直接引用的过程。
```java
public class ActiveLoadTest {  
     static Child child = new Child();  
     public static void main(String[] args) {  
        System.out.println(child);  
      }  
}

```
上面代码中用到 Child 类，在编写程序的时候可以直接使用这个 child 引用去访问 Child 类中可见的属性和方法，但是在 class 字节码中它会被编译成相应的助记符，这些助记符被称为符号引用，在类解析过程中，助记符还需要得到进一步的解析，才能正确地找到所对应的堆内存中的 Child 数据结构，
字节码信息
``` 
public class com/frank/multihread/thread/ActiveLoadTest {

  // compiled from: ActiveLoadTest.java

  // access flags 0x8
  static Lcom/frank/multihread/thread/Child; child

  // access flags 0x1
  public <init>()V
   L0
    LINENUMBER 10 L0
    ALOAD 0
    INVOKESPECIAL java/lang/Object.<init> ()V
    RETURN
   L1
    LOCALVARIABLE this Lcom/frank/multihread/thread/ActiveLoadTest; L0 L1 0
    MAXSTACK = 1
    MAXLOCALS = 1

  // access flags 0x9
  public static main([Ljava/lang/String;)V
   L0
    LINENUMBER 13 L0
    // 在常量池中通过 getstatic 指令获取PrintStream
    GETSTATIC java/lang/System.out : Ljava/io/PrintStream;
    // 同样也适用于获取Child
    // 在字节码的执行过程中， getstatic 被执行之前，就需要进行解析。
    GETSTATIC com/frank/multihread/thread/ActiveLoadTest.child : Lcom/frank/multihread/thread/Child;
    // 然后通过 invokespecial 指令 将 child 传递给 PrintStream.println 方法。
    INVOKEVIRTUAL java/io/PrintStream.println (Ljava/lang/Object;)V
   L1
    LINENUMBER 14 L1
    RETURN
   L2
    LOCALVARIABLE args [Ljava/lang/String; L0 L2 0
    MAXSTACK = 2
    MAXLOCALS = 1

  // access flags 0x8
  static <clinit>()V
   L0
    LINENUMBER 11 L0
    NEW com/frank/multihread/thread/Child
    DUP
    INVOKESPECIAL com/frank/multihread/thread/Child.<init> ()V
    PUTSTATIC com/frank/multihread/thread/ActiveLoadTest.child : Lcom/frank/multihread/thread/Child;
    RETURN
    MAXSTACK = 2
    MAXLOCALS = 0
}


```
解析过程主要正对类接口，字段，类方法和接口方法这四类
- 类接口的解析
- 字段的解析
    - 就是访问的类或者接口中的字段，在 解析 类或者变量的时候，如果改字段不存在或者出现错误就会抛出异常。
    - 如果 Child 类本身存在这个字段，这直接返回字段的引用，当然也要对改字段所属的类提前进行类加载。
    - 如果 Child 类中 不存在改字段，则会根据继承关系自下而上，查找父类或者接口字段，找到即可返回，童谣需要提前对找到的字段进行类加载过程。
    - 如果一直找到 Object 还是没有，这标识查询失败，抛出异常。

#### 初始化
- 最主要的就是执行 `<clinit>()` 方法 （class initialize ），`<clinit>()` 方法中所有的类变量都会被赋予正确的值，也就是编写的时候指定的值。
-  `<clinit>()` 方法在编译器生成，`<clinit>()` 与类的构造函数不同，不需要显示调度父类的构造器，虚拟机会保证父类的 `<clinit>()`  最先执行
- `<clinit>()` 方法虽然是真实存在的，但是它只能被虚拟机执行，在主动触发了类的初始化之后会调用这个方法。


***
## 类加载器
对于任意一个 `class` 都需要有加载他的类加载器和这个类本身确立在 JVM 中的唯一性，也就是运行时包。
- 根加载器
- 扩展类加载器

- 双亲委派模型
- 破坏双亲委派模型
    - 热部署：不停止服务下增加新功能。

- 类 加载器命名空间
    - 同一个class 实例在同一个类加载器命名空间之下是唯一的。
- 运行时包是有类加载器的命名空间和类的全限定名称共同组成的。
- `VM options: -verbose:class` 打印加载的class 

### 双亲委派模型

委托机制：
- 当前线程的类加载器加载线程中的第一个类，可以通过线程的上下文类加载器加载类（当前线程的类加载器可以通过Thread类的getContextClassLoader()获得，也可以通过setContextClassLoader()自己设置类加载器。）
- A类中引用了B类，将使用A类的加载器加载B类
- 直接调用 ClassLoader.loadClass 方法来指定某个类的加载器

破坏双亲委派模型：
- 使用线程上下文类加载器加载类

意义：
- 一个类只会被加载一次，虚拟机中只会存在一份字节码，一个class 对象。
- 防止基类被篡改


## 线程上下文类加载器
为什么要有线程上下文加载器呢？这与JVM类加载器双亲委托机制自身的缺陷分不开。






















