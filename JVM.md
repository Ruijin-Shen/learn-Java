# Java Virtual Machine

JVM的功能：解释和运行、内存管理、以及即时编译。

- **解释和运行**：将字节码文件中的指令实时解释成机器码，让计算机执行。

- **内存管理**：自动为对象、方法等分配内存；自动的垃圾回收机制，回收不再使用的对象。

- **即时编译**（Just-In-Time，JIT）：对热点代码进行优化，提升执行效率。



## 字节码文件

### 1. 字节码文件的组成

字节码文件（.class文件）的主要组成部分如下：

- **基础信息**：魔数（magic，0xCAFEBABE）、版本号（major_version和minor_version）、访问标志（access_flag）等、类索引（this_class）、父类索引（supper_class）等。
- **常量池**（constant_pool）：字符串常量、类或接口名、字段名、方法名等。常量池中的数据都有一个编号，编号从1开始。在字段或者字节码指令中通过编号可以快速的找到对应的数据。在字节码指令中通过编号引用常量池的过程称为**符号引用**。
- **接口**（interfaces）：当前类实现的接口列表。
- **字段**（fields）：当前类或接口声明的字段信息。
- **方法**（methods）：当前类或接口声明的方法信息以及字节码指令。
- **属性**（attributes）：描述当前类或接口所定义的一些属性信息，如源码的文件名、内部类列表等。

```Java
int i = 0, j = 0, k = 0;
// iinc 0 by 1
i++;

// iload_1
// iconst_1
// iadd
// istore_1
j = j + 1;

// iinc 2 by 1
k += 1;
```



## 类的生命周期

### 1. 类的生命周期

类的生命周期描述了一个类**加载**、**连接**、**初始化**、**使用**和**卸载**的整个过程。

#### 1.1 加载阶段

**加载**（loading）阶段第一步是**类加载器**根据类的全限定名通过不同的渠道（本地文件、动态代理生成等）以二进制流的方式获取字节码信息。类加载器在加载完类之后，JVM会在**方法区**中生成一个**InstanceKlass**对象，保存类的所有信息（包含用于实现多态的**虚方法表**）。同时，Java虚拟机还会在**堆**中生成一份与方法区中数据类似的**java.lang.Class**对象，用于在Java代码中获取类的信息以及存储**静态字段**的数据（JDK8及之后）。

为什么存储两份相似的数据？对于开发者来说，只需要访问堆中的Class对象而不需要访问方法区中所有信息。**这样Java虚拟机就能很好地控制开发者访问数据的范围**。

#### 1.2 连接阶段

**连接**（linking）阶段可分为**验证**、**准备**和**解析**三个子阶段。

- **验证**：检测字节码文件是否遵守了《Java虚拟机规范》中的约束。主要包含文件格式验证，元信息验证（如类必须有父类），执行指令语义验证（如方法内的指令不能跳转到不正确的位置）以及符号引用验证（如不能访问其他类中的private方法）。
- **准备**：准备阶段为静态变量分配内存并设置初始值。**准备阶段只会给静态变量赋默认初始值**，int 0，long 0L，short 0，char '\u0000'，byte 0，boolean false，double 0.0，引用数据类型 null。**final修饰的静态变量，且等号右边为字面量（基本数据类型字面量和String字面量），准备阶段直接对变量赋值**。
- **解析**：主要将常量池中的符号引用替换为直接引用。

#### 1.3 初始化阶段

**初始化**（initialization）阶段**为静态变量赋值**并**执行静态代码块中的代码**。初始化阶段会执行字节码文件中clinit部分的字节码指令，clinit方法中字节码的执行顺序与Java源代码中的顺序是一致的（为静态变量赋值的字节码也包含在clinit方法中）。

以下几种方式会导致类的初始化：

- 访问一个类的静态变量或者静态方法，若变量是final修饰并且等号右边是常量不会触发初始化；
- 调用Class.forName(String className)；
- new一个该类的对象时；
- 执行main方法的当前类。

clinit方法在特定的情况下不会出现：

- 无静态代码块且无静态变量赋值语句；
- 有静态变量的声明，但是没有赋值语句；
- 有赋值语句，但是变量使用final修饰且等号右边为字面量。

注：

- 访问子类中的父类静态变量，不会触发子类的初始化。
- 子类的初始化clinit调用之前，会先调用父类的clinit方法。
- 数组的创建不会导致数组中元素的类进行初始化。
- final修饰的变量如果赋值的内容需要执行指令才能得到结果，会执行clinit方法进行初始化。

#### 1.4 使用阶段

#### 1.5 卸载阶段

### 2. 类加载器

**类加载器**（ClassLoader）是Java虚拟机提供给应用程序获取类和接口字节码数据的技术。 类加载器只参与加载过程中的字节码获取和字节码加载到内存。

<img src="./resources/24040901.png" style="zoom: 67%;" />

#### 2.1 JDK8及之前版本的类加载器

##### 2.1.1 类加载器的分类

类加载器分为两类，一类是Java虚拟机底层源代码实现的，一类是Java代码实现的。虚拟机底层源代码实现的类加载器（BootstrapClassLoader）加载程序运行时的基础类。Java代码实现的类加载器（ExtClassLoader，AppClassLoader，自定义类加载器）都继承自**ClassLoader抽象类**，用于加载Java中通用的类或者应用程序使用的类。

**启动类加载器**（BootstrapClassLoader）是由Hotspot虚拟机提供的，使用C++编写的类加载器。**启动类加载器默认加载目录`/jre/lib`下的类文件**。若需通过启动类加载器加载用户jar包，可通过虚拟机参数`-Xbootclasspath/a:jar包目录/jar包名`进行拓展。

**拓展类加载器**（ExtClassLoader）和**应用程序类加载器**（AppClassLoader）都是使用Java编写的类加载器。这两个类加载器是sun.misc.Launcher的静态内部类，继承自URLClassLoader。**拓展类加载器默认加载目录`jre/lib/ext`下的类文件**。若需通过拓展类加载器加载用户jar包，可通过虚拟机参数`-Djava.ext.dirs=jar包目录`，这种方式会覆盖掉原始目录，可以用 ; （windows）或 : （macos/linux）追加上原始目录。**应用程序类加载器加载classpath下的类文件**。

##### 2.1.2 使用代码主动加载类

在Java中可以用如下两种方式主动加载一个类：

- 使用Class.forName方法，使用当前类的类加载器加载指定的类。
- 获取类加载器，通过类加载器的loadClass方法加载指定的类。

##### 2.1.3 类加载器的双亲委派机制

每个Java实现的类加载器中都保存了一个成员变量parent，表示父类加载器（非继承关系）。应用程序类加载器的parent类加载器是拓展类加载器，拓展类加载器的parent为空。启动类加载器使用C++编写，没有父类加载器。

**双亲委派机制**（Parent Delegation Mechanism）：在类加载的过程中，每个类加载器都会先检查是否已经加载了该类，如果已经加载则直接返回，否则会将加载请求委派给父类加载器。如果类加载的parent为null，则会提交给启动类加载器处理。如果所有的父类加载器都无法加载该类，则由当前类加载器尝试加载。第二次再去加载相同的类，仍然会向上进行委派，如果某个类加载器加载过就会直接返回。双亲委派机制指的是：自底向上查找是否加载过，再由顶向下进行加载。

- 如果一个类出现在三个类加载器的加载位置，应该由谁来加载？根据双亲委派机制，启动类加载器的优先级是最高的，由启动类加载器加载。
- String类能覆盖吗？不能，启动类加载器会加载`rt.jar`包中的String类。

双亲委派机制的作用：保证类加载的安全性，启动类加载器负责加载核心类，避免恶意代码替换JDK中的核心类库；避免重复加载，上层的类加载器如果加载过某个类，就会直接返回该类，避免重复加载。

双亲委派机制相关**源代码**：

ClassLoader中包含如下几个核心方法：

```Java
public Class<?> loadClass(String name) throws ClassNotFoundException {
    return loadClass(name, false);
}

protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
            }

            if (c == null) {
                // If still not found, then invoke findClass in order
                // to find the class.
                long t1 = System.nanoTime();
                c = findClass(name);

                // this is the defining class loader; record the stats
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}

// Finds the class with the specified binary name. This method should be overridden
// by class loader implementations that follow the delegation model for loading classes,
// and will be invoked by the loadClass method after checking the parent class loader
// for the requested class. The default implementation throws a ClassNotFoundException.
protected Class<?> findClass(String name) throws ClassNotFoundException


// Converts an array of bytes into an instance of class Class. Before the Class can be
// used it must be resolved.
protected final Class<?> defineClass(String name， byte[] b, int off, int len) throws ClassFormatError

// Links the specified class. This (misleadingly named) method may be used by a class
// loader to link a class. If the class c has already been linked, then this method simply
// returns. Otherwise, the class is linked as described in the "Execution" chapter of The
// Java™ Language Specification.
protected final void resolveClass(Class<?> c)
```

##### 2.1.4 自定义类加载器

正确地实现一个自定义类加载器的方式是重写findClass方法，这样不会破坏双亲委派机制。若想打破双亲委派机制，需实现自定义类加载器并且重写loadClass方法，将双亲委派机制的代码去除。Tomcat使用这种方式实现应用之间的类隔离。

```Java
import org.apache.commons.io.IOUtils;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.StandardCharsets;
import java.security.ProtectionDomain;
import java.util.regex.Matcher;

public class BreakClassLoader extends ClassLoader{
    private String basePath;
    private final static String FILE_EXT = ".class";
    
    public void setBasePath(String basePath) {
        this.basePath = basePath;
    }
    
    private byte[] loadClassData(String name) {
        try {
            String tempName = name.replaceAll("\\.", Matcher.quoteReplacement(File.separator));
            FileInputStream fis = new FileInputStream(basePath + tempName + FILE_EXT);
            try {
                return IOUtils.toByteArray(fis);
            } finally {
                IOUtils.closeQuietly(fis);
            }

        } catch (Exception e) {
            System.out.println("自定义类加载器加载失败：" + e.getMessage());
            return null;
        }
    }
    
    @Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        if(name.startsWith("java.")) {
            return super.loadClass(name);
        }
        byte[] data = loadClassData(name);
        return defineClass(name, data, 0, data.length);
    }

    public static void main(String[] args) throws Exception {
        BreakClassLoader classLoader = new BreakClassLoader();
        classLoader.setBasePath("");
        Class<?> clazz1 = classLoader.loadClass("com.example.A");
     }
}
```

- 两个自定义类加载器加载相同限定名的类，会冲突吗？不会冲突，在同一个Java虚拟机中，只有相同类加载器+相同的类限定名才会被认为是同一个类。

#### 2.2 JDK9及之后版本的类加载器

[内容]



### Java内存区域

Java虚拟机在运行Java程序过程中管理的内存区域，称之为**运行时数据区**。Java虚拟机在执行Java程序的过程中会把它管理的内存划分成若干个不同的数据区域。

<div style="display: flex; justify-content: center; gap: 10px;">
    <img src="./resources/25020202.png" alt="图片1" width="45%">
    <img src="./resources/25020201.png" alt="图片2" width="45%">
</div>
- **程序计数器**：每个线程会通过程序计数器记录将要执行的字节码指令的地址；顺序执行、分支、循环、跳转、异常处理、线程恢复等功能都需要依赖程序计数器完成。

- **Java虚拟机栈和本地方法栈**：Java采用栈的结构管理方法调用中的数据，每一个方法调用使用一个栈帧来保存。Java虚拟机栈和本地方法栈随着线程的创建而创建，在线程销毁时回收。每个栈帧中都包含：局部变量表、操作数栈、动态链接和方法返回地址。虚拟机栈为虚拟机执行Java方法服务，本地方法栈为虚拟机用到的native方法服务，在HotSpot虚拟机中虚拟机栈和本地方法栈共享同一个栈空间。

  <img src="./resources/25020203.png" style="zoom: 35%;" />

  **局部变量表**存放了方法执行过程中的所有局部变量（this、方法的参数、方法体中声明的局部变量），源文件编译成字节码文件时就可以确定局部变量表的内容；栈帧中的局部变量表是一个数组，数组中的每一个位置称为槽（slot），long和double类型占用两个槽，其他类型占用一个槽；为了节省空间，局部变量表中的槽式可以复用的，一旦某个局部变量不再生效，槽就可以再次被使用。

  **操作数栈**是虚拟机在执行指令的过程中用来存放中间数据的一块区域，是栈的结构，如果一条指令将一个值压入操作数栈，则后面的指令可以弹出并使用该值；操作数栈的最大深度在编译器就可以确定。

  当字节码指令引用了其他类的属性或方法时，需要将符号引用转换成对应的运行时常量池中的地址，**动态链接**就保存了符号引用到运行时常量池中地址的映射。

  **方法返回地址指**的是方法在正确或异常结束时，当前栈帧被弹出后，程序计数器应该指向上一个栈中的下一条将要执行指令的地址。

  **异常表**存放异常处理相关信息，包括异常捕获的生效范围以及异常发生后跳转到的字节码指令位置。

  可以使用**参数`-Xss`设置虚拟机栈的大小**，后跟字节数（必须是1024的倍数）、k或者K、m或者M、g或者G，如`-Xss1m`。也可以使用`-XX:ThreadStackSize`配置栈大小，如``-XX:ThreadStackSize=1024`。

- **方法区**：方法区是JVM运行时数据区的一块逻辑区域，是各个线程共享的内存区域。方法区会存储类的**元信息**、**运行时常量池**和**字符串常量池**（JDK1.7及以前）。永久代（JDK1.7及之前）和元空间（JDK1.8及之后）是HotSpot虚拟机对方法区的两种实现方式**。可以**使用 `-XX：MaxMetaspaceSize` 标志设置最大元空间大小**，默认值为 unlimited。

- **运行时常量池**：字节码文件中有存放编译期生成的字面量和符号引用的常量池表。常量池表会在类加载后放到方法区的运行时常量池中。

- **字符串常量池**：

- **堆**：堆是所有线程共享的一块内存区域，几乎所有的对象实例以及数组都在堆分配内存。从JDK1.7开始默认开启逃逸分析，如果某些方法中的对象引用没有被返回或者未被外面使用，那么对象可能直接在栈上分配内存。Java堆是垃圾收集管理的主要区域，因此也被称作GC堆。使用**参数`-Xms`设置堆的初始大小**，**参数`-Xmx`设置堆的最大大小**。**JDK8之后永久代已被元空间取代，元空间使用的是本地内存**。

  <img src="./resources/25020301.png" style="zoom: 45%;" />

