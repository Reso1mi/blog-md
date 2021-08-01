---
title: 深入理解 Java 虚拟机（三）
tags:
  - Class 结构
  - JVM
categories:
  - JVM
cover: 'http://static.imlgw.top/blog/20190905/bvODiOGH0IfM.jpg?imageslim'
date: 2019/9/5
abbrlink: '18643927'
---

> 这一篇主要记录 JVM 相关的 Class 文件结构

## Class 类文件的结构

 Class 文件是一组以** 8 个字节**为基础单位的二进制流，根据 Java 虚拟机规范，Class 文件格式采用一种类似于 C 语言结构体的伪结构来存储数据，这种伪结构中只有两种数据类型：**无符号数和表** 

**无符号数**

无符号数属于基本的数据类型，以 u1，u2，u4，u8 来分别表示 1，2，4，8 个字节大小的无符号数，无符号数用来描述数字，索引引用，数量值，或者按照 UTF-8 编码构成字符串值

**表（数组）**

表是由多个无符号数或者其他表作为数据项构成的符合数据类型，所有表都习惯性的以`_info` 结尾，表用于描述有程池关系的复合结构的数据，整个 Class 文件本质上就是一张表

### Class 文件整体结构

```java
ClassFile{
    u4                  magic（魔数 0xCAFFBABE)
    u2                  minor_version（次版本号）
    u2                  major_version（主版本号）
    u2                  constant_pool_count（常量池个数）
    cp_info             constant_pool（常量池表）
    u2                  access_flags（类或接口的访问权限）
    u2                  this_class（类名）
    u2                  super_class（父类名）
    u2                  interfaces_count（接口个数）
    u2                  interfaces（接口名）
    u2                  fields_count（字段个数）
    field_info          fields（字段表）
    u2                  methods_count（方法数）
    method_info         methods（方法表）
    u2                  attributes_count（附加属性个数）
    attribute_info      attributes（附加属性表）
}
```

> 在网上看了 [国外的大佬](https://www.infoq.cn/article/Secrets-of-the-Bytecode-Ninjas) 的一张图，挺有意思的！[mark](http://static.imlgw.top/blog/20190902/4JDgENgQ9eIo.png?imageslim)

**魔数和 Class 文件的版本**

每个 Class 文件的**头 4 个字节**称为魔数，他唯一的作用是确定这个文件是否为一个能被虚拟机接收的 Class 文件，这个值为`0xCAFFBABE` ，紧跟着魔数的** 4 个字节**存储的是 Class 文件的版本号，高版本的 JDK 能兼容低版本的字节码，而低版本 JVM 的无法兼容高版本的 Class 文件

## 常量池

紧跟着版本号之后的就是常量池入口，一个 Java 类中定义的很多信息都是由常量池来维护和描述的，可以将常量池看作 Class 文件的资源仓库，比如说 Java 类中定义的方法和变量信息，都是存储在常量池中，常量池中主要存储的两类常量：

- **字面量：** 文本字符串，Java 中声明为 final 的常量值等
- **符号引用：**类和接口的全限定名，字段的名称和描述符，方法的名称和描述符等

> 关于符号引用，其实在之前的文章中有介绍过，Java 代码在编译的时候并不像 C++/C 一样有连接的步骤，而是在虚拟机加载 Class 文件的时候进行**动态链接**，也就是说，在 Class 文件中不会保存各个方法，字段的最终内存布局信息，因此这些字段，方法的符号引用无法直接被虚拟机使用，当虚拟机运行的时候，需要从常量池中获得对应得符号引用，在类创建或运行得时候解析，翻译到具体的内存地址中。

### 常量池项目类型

常量池的每一项都是一个表，JDK1.7 之前一共有 11 种**结构不同**的表结构，也就是下面的 11 种，JDK1.7 之后为了更好的支持动态语言的调用，又额外的增加了 3 种（CONSTANT_MethodHandle_info，CONSTANT_MethodType_info，CONSTANT_InvokeDynamic_info）这些表结构都有一个共同的特点，**表开始的第一位都是一个`u1`类型的标志位`tag`，目的就是区分这个常量属于那种类型的常量**，后面的内容都各有各的结构，`index`代表的是常量池中的对应的常量索引，`bytes`代表的就是字节数据

### cp_info

![常量池表结构](http://static.imlgw.top/blog/20190829/vjjWxwvrt0qg.png)

这张表上的数据不用记住，用的时候知道去哪里查就行了（虽然用的机会很少😂），下面我们编译一段代码，看一下字节码长啥样

```java
public class Test1 {
    
    private int a=1;
    
    public int getA(){
        return a;
    }
    public void setA(int a){
        this.a=a;
    }
}
```

🎯**编译完成后用 16 进制的编译器 (winHex) 打开 Class 文件**

![mark](http://static.imlgw.top/blog/20190901/EOnrpm9sx8wE.png?imageslim)

⚡ 这里前面的 4 个字节`0xCAFEBABE` 代表的就是魔数，后面的 4 个字节`0x00000034`代表的就是版本号，再往后 2 个字节`0x0018`就是常量池的入口，对应的就是常量池的大小（constant_pool_count），转换为 10 进制就是 24，但是实际上并不是 24 个，常量池计数是从 1 而不是 0 开始的，设计者将 0 位置的项空出来目的是为了表示后面某些指向常量池的索引值的数据在特定情况下表示**不引用任何一个常量池项目**（大师就是大师，各种细节都能考虑到）

⚡ 再往后看，`0x0A`，这个就是我们前面说的`tag`标志位，转换为 10 进制就是`10` ，查一下表对应的常量类型是`CONSTANT_Methodref_info`，紧跟着的两个字节`0x0004`是一个`index`类型的数据，指向声明方法描述符`CONSTANT_Class_info` 的索引项，也就是常量池的第 4 项，再往后两个字节`0x0014`代表的就是指向类型描述符`CONSTANT_NameAndType`的索引项，也就是常量池的第 20 项

这里我们就不一一的去分析了，我们借助`javap` 来看看反编译的结果和我们分析的是不是一致的

```java
Last modified 2019-9-1; size 485 bytes
  MD5 checksum e8148a01ff25087c42827d62a9b827b0
  Compiled from "Test1.java"
public class jvmstudy.classfile_stu.Test1
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#20         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#21         // jvmstudy/classfile_stu/Test1.a:I
   #3 = Class              #22            // jvmstudy/classfile_stu/Test1
   #4 = Class              #23            // java/lang/Object
   #5 = Utf8               a
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Ljvmstudy/classfile_stu/Test1;
  #14 = Utf8               getA
  #15 = Utf8               ()I
  #16 = Utf8               setA
  #17 = Utf8               (I)V
  #18 = Utf8               SourceFile
  #19 = Utf8               Test1.java
  #20 = NameAndType        #7:#8          // "<init>":()V
  #21 = NameAndType        #5:#6          // a:I
  #22 = Utf8               jvmstudy/classfile_stu/Test1
  #23 = Utf8               java/lang/Object
{
  public jvmstudy.classfile_stu.Test1();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: iconst_1
         6: putfield      #2                  // Field a:I
         9: return
      LineNumberTable:
        line 7: 0
        line 9: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      10     0  this   Ljvmstudy/classfile_stu/Test1;

  public int getA();
    descriptor: ()I
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field a:I
         4: ireturn
      LineNumberTable:
        line 12: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Ljvmstudy/classfile_stu/Test1;

  public void setA(int);
    descriptor: (I)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: iload_1
         2: putfield      #2                  // Field a:I
         5: return
      LineNumberTable:
        line 16: 0
        line 17: 5
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Ljvmstudy/classfile_stu/Test1;
            0       6     1     a   I
}
SourceFile: "Test1.java"

```

可以看到和我们分析结果是一致的，那这个`Methodref`是表示的那个方法呢？其实根据反编译的结果也可以看出来，这个方法是我们默认的无参构造方法

## 访问标志

| 标志名称       | 标志值   | 含义                                                         |
| -------------- | -------- | ------------------------------------------------------------ |
| ACC_PUBLIC     | 0x00 01  | 是否为 Public 类型                                             |
| ACC_FINAL      | 0x00 10  | 是否被声明为 final，只有类可以设置                            |
| ACC_SUPER      | 0x00 20  | 是否允许使用 invokespecial 字节码指令的新语义．jdk1.0.2 之后都为真 |
| ACC_INTERFACE  | 0x02 00  | 标志这是一个接口                                             |
| ACC_ABSTRACT   | 0x04 00  | 是否为 abstract 类型，对于接口或者抽象类来说，次标志值为真，其他类型为假 |
| ACC_SYNTHETIC  | 0x10 00  | 标志这个类并非由用户代码产生                                 |
| ACC_ANNOTATION | 0x20 00  | 标志这是一个注解                                             |
| ACC_ENUM       | ０x40 00 | 标志这是一个枚举                                             |

在常量池结束之后，紧接着的**两个字节**代表访问标志（`access_flags`），这个标志用于识别类或接口层次的访问信息，包括：这个 Class 是接口还是方法？是否定义为 public 类型？是否定义为 abstract？等，两个字节 16 个标志位，可以表示 2^16 种状态，但是当前只定义了 8 个标志位没有使用到的一律要求为 0

还是参考上面的字节码，常量池是图中从`0A~74` 的紫色部分，紧跟着后面的两个字节`0x0021`对应的就是访问标志位，也就是 `ACC_PUBLIC | ACC_SUPER` 的值

## 类索引，父类索引接口索引集合

类索引 (`this_class`) 和父类索引 (`super_class`) 都是一个 u2 类型的数据，而接口索引集合 (`interfaces`) 是一组`u2` 类型的**数据的集合**，Class 文件中由这三项数据来表示这个类的继承关系

类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名，由于 Java 语言不允许多重继承，所以父类索引只有一个，除了 java.lang.Object 之外，所有的 Java 类都有父类，因此除了 java.lang.Object 外，所有 Java 类的父类索引都不为 0。接口索引集合就用来描述这个类实现了哪些接口，这些被实现的接口将按 implements 语句（如果这个类本身是一个接口，则应当是 extends 语句）后的接口顺序从左到右排列在接口索引集合中。

类索引、父类索引和接日索引集合都按顺序排列在访问标志之后，类索引和父类索引用两个 u2 类型的索引值表示，它们各自指向一个类型为`CONSTANT_Class_info`的类描述符常量，通过`CONSTANT_Class_info`类型的常量中的索引值以找到定义在`CONSTANT_Utf8_info`类型的常量中的全限定名字符串

我们接着上面的字节码文件分析，紧接着访问标志符后面的 u2 是`0x0003` 也就是`this_class`在常量池中的索引，再往后的`0x0004`对应的就是`super_class`在常量池的索引`0x0000` 说明没有父接口，后面的集合也就没有了

```java
Constant pool:
   #1 = Methodref          #4.#20         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#21         // jvmstudy/classfile_stu/Test1.a:I
   #3 = Class              #22            // jvmstudy/classfile_stu/Test1
   #4 = Class              #23            // java/lang/Object
   #5 = Utf8               a
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Ljvmstudy/classfile_stu/Test1;
  #14 = Utf8               getA
  #15 = Utf8               ()I
  #16 = Utf8               setA
  #17 = Utf8               (I)V
  #18 = Utf8               SourceFile
  #19 = Utf8               Test1.java
  #20 = NameAndType        #7:#8          // "<init>":()V
  #21 = NameAndType        #5:#6          // a:I
  #22 = Utf8               jvmstudy/classfile_stu/Test1
  #23 = Utf8               java/lang/Object
```

结合反编译的结果，常量池第三项和第四项都对应了一个`CONSTANT_Class_info`的索引常量，其最终指向了一个`CONSTANT_Utf8_info` 的常量，这个常量的值就是我们的`this_class`和`super_class` 的全限定名

## 字段表集合

紧跟着上面类索引等信息后面的`u2`类型的数据`0x0001`就是代表的`fields_count`，这里只有一个字段 a 所以这里是`1`

### field_info

字段表（`field_info`）用于描述接口或者类中声明的变量。字段（`field`）包括类级变量以及实例级变量，但不包括在方法内部声明的局部变量

字段的各种修饰符其实都是个布尔值，要么有要么没有，所以很好表示适合用标志位来表示，但是字段的类型，名字这些就很难固定

| 类型           | 名称             | 数量             |
| -------------- | ---------------- | ---------------- |
| u2             | access_flags     | 1                |
| u2             | name_index       | 1                |
| u2             | descriptor_index | 1                |
| u2             | attributes_count | 1                |
| attribute_info | attributes       | attributes_count |

前三项分别代表，访问修饰符，名字的索引以及_描述符_（**对字段来说就是字段的类型**）的索引，这三项就可以构成一个字段的完整的信息了，这里我们重点看的也是前三项的信息，字段修饰符`access_flags`其实和类中的`access_flags` 类似，只不过可以设置的修饰符不太一样

**字段访问标志**

| 标志名称      | 标志值 | 含义                       |
| ------------- | ------ | -------------------------- |
| ACC_PUBLIC    | 0x0001 | 字段是否 public             |
| ACC_PRIVATE   | 0x0002 | 字段是否 private            |
| ACC_PROTECTED | 0x0004 | 字段是否 protected          |
| ACC_STATIC    | 0x0008 | 字段是否 static             |
| ACC_FINAL     | 0x0010 | 字段是否 final              |
| ACC_VOLATILE  | 0x0040 | 字段是否 volatile           |
| ACC_TRANSIENT | 0x0080 | 字段是否 transient          |
| ACC_SYNTHETIC | 0x1000 | 字段是否由编译器自动产生的 |
| ACC_ENUM      | 0x4000 | 字段是否 enum               |

🎯**继续接着上面的类索引分析**

![mark](http://static.imlgw.top/blog/20190902/s5dWhYXNIuxq.png?imageslim)

⚡`0x0001` 对应`fields_count`，值为 1 字段的数量为 1，后面的`field_info`数量为 1

⚡`0x0002` 对应字段的`access_flags`，为`ACC_PRIVATE`

⚡`0x0005` 对应字段的名字的索引，在常量池中的第 5 项，结合上面反编译的结果第 5 项为  `#5 = Utf8    a`

⚡`0x0006` 对应字段的描述符索引，在常量池中第 6 项，为`#6 = Utf8   I`

到这里其实我们就可以推断出这个变量的定义为 `private int a`

> 字段表集合中不会列出从超类或者父接口中继承而来的字段，但是有可能会列出原本 Java 代码中没有的字段，比如内部类中自动添加指向外部类的字段。从所周知，在** Java **中字段是不能重载的，只要两个字段的名字是不能一样的，必须使用不同的名称，注意，这是** Java 的语言规范**，其实在 Class 字节码来讲，只要两个字段的描述符不一样，重名就是合法的。

再往后两个字节`0x0000`代表`attributes_count`，其值为 0 所以后面的属性表就没有了，如果这里将字段改为`static final int a` 那么`attributes_count` 将为 1，后面会多出一条指向常量池`ConstantValue`数据的 index（下文会演示）

## 方法表集合

紧跟着字段表之后的 u2 类型的数据 `0x0003`代表的就是`method_count` ，值为 3 代表有三个方法，这里面包含了 JVM 自动生成的默认无参数构造器方法，所以是 3 个

### method_info

方法表的结构其实和字段表的结构是一样的

| 类型           | 名称             | 数量             |
| -------------- | ---------------- | ---------------- |
| u2             | access_flags     | 1                |
| u2             | name_index       | 1                |
| u2             | descriptor_index | 1                |
| u2             | attributes_count | 1                |
| attribute_info | attributes       | attributes_count |

但是具体的`access_flags` 和`attributes`集合不太一样

**方法访问标志**

| 标志名称          | 标志值  | 含义                             |
| ----------------- | ------- | -------------------------------- |
| ACC_PUBLIC        | 0x00 01 | 方法是否为 public                 |
| ACC_PRIVATE       | 0x00 02 | 方法是否为 private                |
| ACC_PROTECTED     | 0x00 04 | 方法是否为 protected              |
| ACC_STATIC        | 0x00 08 | 方法是否为 static                 |
| ACC_FINAL         | 0x00 10 | 方法是否为 final                  |
| ACC_SYHCHRONRIZED | 0x00 20 | 方法是否为 synchronized           |
| ACC_BRIDGE        | 0x00 40 | 方法是否是由编译器产生的桥接方法 |
| ACC_VARARGS       | 0x00 80 | 方法是否接受参数                 |
| ACC_NATIVE        | 0x01 00 | 方法是否为 native                 |
| ACC_ABSTRACT      | 0x04 00 | 方法是否为 abstract               |
| ACC_STRICTFP      | 0x08 00 | 方法是否为 strictfp               |
| ACC_SYNTHETIC     | 0x10 00 | 方法是否是有编译器自动产生的     |

🎯**我们继续接着上面的字段表后面分析**

![mark](http://static.imlgw.top/blog/20190902/BuNVfOyD3mJy.png?imageslim)

⚡`0x0003` 方法表的入口，代表`methods_count` 方法的数量，值为 3，有三个方法（包括了编译器自动生成的`<init>`方法）

⚡ `0x0001` 第一个方法的访问标志值，代表`ACC_PUBLIC`

⚡ `0x0007` 方法名的`name_index`，值为 7 代表常量池中第 7 项常量，结合反编译的结果其值为

`#7 = Utf8  <init>`  也就是我们的实例构造方法（编译器自动帮我们生成的）

⚡ `0x0008` 方法描述符的`descriptor_index` ，值为 8 代表常量池中第 8 项常量，其值为

`#8 = Utf8  ()V`  自动生成的无参构造器的描述

⚡ `0x0001` 方法属性表 (`attribute_info`) 的入口，代表`attributes_count` ，值为 1 代表有一个属性表

⚡ `0x0009` 属性名称的索引，常量池中为 `#9 = Utf8   Code`说明此属性是方法的字节码描述，也就是方法中的代码

> 在方法表中，如果子类没有重写父类的方法就不会出现父类的方法信息，但是会出现编译器自动添加的方法，最典型的就是`<clinit>`和`<init>`，这里并没有 静态变量的赋值和静态语句块所以并没有生成`<clinit>` 方法，另外一点和上面字段表的一样，Java 语言中重载是不能以返回值来界定的，但是在 Class 文件格式中，只要描述符不一致，就是可以共存的，而方法的描述符就包括了参数列表和返回值

## 属性表集合

属性表（attribute_info），Class 文件，字段表，方法表中可以携带自己的属性表，说有 attribute_info 的前两项都是 u2 类型的`attribute_name_index`和 u4 类型的`attribute_length`分别代表属性名字的索引和属性值的大小 ，Java 虚拟机中预定义了一些属性，这些属性都各有各的含义和结构

| 属性名称                             | 使用位置           | 含义                                                         |
| :----------------------------------- | ------------------ | :----------------------------------------------------------- |
| Code                                 | 方法表             | Java 代码编译成的字节码指令                                  |
| ConstantValue                        | 字段表             | final 关键字定义的常量值                                     |
| Deprecated                           | 类、方法表、字段表 | 被声明为 `deprecated` 的方法和字段                           |
| Exceptions                           | 方法表             | 方法抛出的异常                                               |
| EnclosingMethod                      | 类文件             | 仅当一个类为局部类或匿名类时才能拥有这个属性，这个属性用于标识这个类所在的外围方法 |
| InnerClasses                         | 类文件             | 内部类列表                                                   |
| LineNumberTable                      | Code 属性          | Java 源码的行号与字节码指令的对应关系                        |
| LocalVariableTable                   | Code 属性          | 方法的局部变量描述                                           |
| StackMapTable                        | Code 属性          | JDK 6 新增的属性，供新的类型检查验证器（Type Checker）检查和处理目标方法的局部变量和操作数栈所需要的类型是否匹配 |
| Signature                            | 类、方法表、字段表 | JDK 5 新增的属性，用于支持泛型情况下的方法签名，在 Java 语言中，任何类、接口、初始化方法或成员的泛型签名如果包含了类型变量（Type Variables）或参数化类型（Parameterized Types），则 Signature 属性会为它记录泛型签名信息。由于 Java 的泛型采用擦除法实现，在为了避免类型信息被擦除后导致签名混乱，需要这个属性记录泛型中的相关信息 |
| SourceFile                           | 类文件             | 记录源文件名称                                               |
| SourceDebugExtension                 | 类文件             | JDK 6 新增属性，用于存储额外的调试信息。譬如在进行 JSP 文件调试时，无法通过 Java 堆栈来定位到 JSP 文件的行号，JSR-45 规范为这些非 Java 语言编写，却需要编译成字节码并运行在 Java 虚拟机中的程序提供了一个进行调试的标准机制，使用该属性就可以存储这个标准所新加入的调试信息 |
| Synthetic                            | 类、方法表、字段表 | 表示方法或字段是由编译器自动生成的                           |
| LocalVariableTypeTable               | 类                 | JDK 5 新增属性，它使用特征签名代替描述符，为了引入泛型语法之后能描述泛型参数化类型而添加 |
| RuntimeVisibleAnnotations            | 类、方法表、字段表 | JDK 5 新增属性，为动态注解提供支持。该属性指明哪些注解是运行时（实际上运行时就是进行反射调用）可见的 |
| RuntimeInvisibleAnnotations          | 类、方法表、字段表 | JDK 5 新增属性，与 `RuntimeVisibleAnnotations` 作用相反，用于指明哪些注解运行时不可见 |
| RuntimeVisibleParameterAnnotations   | 方法表             | JDK 5 新增属性，作用与 `RuntimeVisibleAnnotations` 作用相似，只不过作用对象为方法参数 |
| RuntimeInvisibleParameterAnnotations | 方法表             | JDK 5 新增属性，作用与 `RuntimeInvisibleAnnotations` 作用相似，只不过作用对象为方法参数 |
| AnnotationDefault                    | 方法表             | JDK 5 新增属性，用于记录注解类元素的默认值                   |
| BootstrapMethods                     | 类文件             | JDK 7 新增属性，用于存储 `invokedynamic` 指令引用的引导方法限定符 |

### Code 属性

Java 程序方法体中的代码经过 Javac 编译器处理过后，最终变为字节码指令存储在 Code 属性内

Code 属性出现在**方法表的属性集合**中，但是并非所有的方法表都必须存在这个 Code 属性，**像接口和抽象类中的方法就不存在 Code 属性**

**Code 属性结构**

| 类型            | 名称                   | 数量                   |
| --------------- | ---------------------- | ---------------------- |
| u2              | attribute_name_index   | 1                      |
| u4              | attribute_length       | 1                      |
| u2              | max_stack              | 1                      |
| u2              | max_locals             | 1                      |
| u4              | code_length            | 1                      |
| u1              | code                   | code_length            |
| u2              | exception_table_length | 1                      |
| exception_info  | exception_table        | exception_table_length |
| u2              | attributes_count       | 1                      |
| attributes_info | attributes             | attributes_count       |

🎯**紧接着上面的方法表分析**

_这里分析的还是方法表第一个方法（`<init>`）所携带的属性表_

![mark](http://static.imlgw.top/blog/20190904/Gd5cWl6Yr84x.png?imageslim)

⚡ `0x0009` 对应`attribute_name_index`，属性名称的索引，常量池中为 `#9 = Utf8   Code`说明此属性是方法的字节码描述，也就是方法中的代码

⚡ `0x00000038` 代表`attribute_length` 顾名思义就是属性值的长度，这里是 56（不包含 attribute_name_index 和 attribute_length）

⚡`0x0002`代表`max_stack` 操作数栈（Operand Stacks）深度的最大值，虚拟机运行的时候会根据这个值来分配栈帧（StackFrame）中的操作栈深度

⚡`0x0001`代表`max_locals` 代表了**局部变量表**所需的存储空间，这里的内存分配单位是`Slot`

> `Slot`是虚拟机为局部变量分配内存所使用的最小单位，对于 byte，char，float，int，short，boolean 等长度不超过 32 位的数据类型，每个局部变量占用一个`Slot` ，但是对于 double 和 long 等 64 位的数据类型则需要两个`Slot` 
>
> 方法参数（包括实例方法中的 this 引用），trycatch 语句中 catch 中定义的异常，方法体中定义的局部变量 都需要使用局部变量表来存放，但是最后并不是把所有这些局部变量占用的`Slot`加起来就是`max_locals` 因为**局部变量表中的 Slot 是可以复用的**，当代码执行超过一个局部变量的作用域时，这个变量占用的 Slot 就可以被其他局部变量所使用，所以编译器会根据作用域来分配`Slot` 给各个局部变量使用

⚡`0x0000000A`u4 类型的`code_length` 代表的就是字节码的长度，这里是 10，说明后面有 10 个字节长度的字节码指令流

⚡u1 类型的`code` 就是具体用字节码指令，每个指令都是一个 u1 的单字节指令，也就是说最多有 256 个指令，目前 Java 虚拟机已经定义了其中约 200 条编码值对应的指令，这里有连续的 10 个单字节指令，构成了`<init>`方法的字节码指令

⚡`0x0000`代表`exception_table_length` 异常表的长度，这里`<init>`方法没有异常抛出所以为 0

⚡ `exception_info`类型的`exception_table` 异常表，存放处理异常的信息（try-catch 中的异常），前面的长度为 0 所以这里不存在这一项数据

**异常表结构**

| 类型 | 名称       | 数量 |
| ---- | ---------- | ---- |
| u2   | start_pc   | 1    |
| u2   | end_pc     | 1    |
| u2   | handler_pc | 1    |
| u2   | catch_type | 1    |

每个 exception_table 表项由 start_pc，end_pc，handler_pc，catch_type（指向常量池中 CONSTANT_Class_info 类型的常量）组成

当字节码在 start_pc 到 end_pc 之间出现了类型为 catch_type 或者其子类的异常，就转到 handler_pc 行继续执行，当 catch_type 为 0 时表示处理所有的异常

⚡`0x0002` 代表`attributes_count` 是`Code` 属性表的**附加属性**的入口（一层套一层啊😂），值为 2 意味着附加属性表的数量为 2

### LineNumberTable 属性

这个就是上面**`<init>`方法 Code 属性附加的第一个属性**，这个属性用于描述 Java 源代码行号和字节码行号（偏移量）之间的对应关系，他并不是运行的必须属性，但是默认会生成到 Class 文件中，可以使用`-g:none`或`-g:lines`取消生成这个属性，取消之后程序抛异常的时候不会显示出错的行号，并且在调试的时候，也无法按照源码行来设置断点，其结构如下表

| 类型             | 名称                     | 数量                     |
| ---------------- | ------------------------ | ------------------------ |
| u2               | attribute_name_index     | 1                        |
| u4               | attribute_length         | 1                        |
| u2               | line_number_table_length | 1                        |
| line_number_info | line_number_table        | line_number_table_length |

🎯**我们再接着上面的 Code 属性的属性表分析，看看 Code 的属性表是啥**

![mark](http://static.imlgw.top/blog/20190904/O7MuBcPXNPKT.png?imageslim)

⚡`0x000A` attributes_info 的第一项，对应的是 atttibute 的名字的索引，常量池中对应第 10 项的索引是 

 `#10 = Utf8   LineNumberTable` 说明这个属性是 LineNumberTable 属性，然后根据上面给出的表格继续分析

⚡`0x0000000A` 对应 attribute_length，说明该属性值长度为 10

⚡`0x0002`对应 line_number_table_length，说明有两处对应关系

⚡ 后面紧跟的字节就对应的`line_number_table` 该属性又有两个属性，分别为`start_pc`和`line_number`两个 u2 类型的数据项，前者是字节码行号，后者是源码行号，前面 line_number_table_length 为 2，所以这里后面有两个 line_number_table

### LocalVariableTable 属性

这个就是上面**`<init>` 方法 Code 属性附加的第二个属性**，这个属性主要用于描述**栈帧中局部变量表**中的变量和 Java 源代码中定义的变量之间的关系，没有这项属性当在其他地方使用该方法的时候关于参数的名称都会丢失，最典型的就是 IDE 中有时候反编译一些框架的代码就会看见一些方法参数什么的都是 arg0，arg1 什么的

**LocalVariableTable 属性结构**

| 类型                | 名称                        | 数量                        |
| ------------------- | --------------------------- | --------------------------- |
| u2                  | attribute_name_index        | 1                           |
| u4                  | attribute_length            | 1                           |
| u2                  | local_variable_table_length | 1                           |
| local_variable_info | local_variable_table        | local_variable_table_length |

🎯 **我们接着上面的 LineNumberTable 属性分析`LocalVariableTable`**

![mark](http://static.imlgw.top/blog/20190905/TafXz1oMkG4d.png?imageslim)

⚡`0x000B`和之前所有的 attributes_info 一样，这个第一项代表该属性的名字在常量池的索引值，这里对应常量池第 11 项 `#11 = Utf8   LocalVariableTable` 

⚡`0x0000000C` 对应 attribute_length，说明该属性值长度为 12

⚡`0x0001`对应 local_variable_table_length，值为 1 说明只有一个局部变量

> 其实到现在我们分析的属性表都还是在分析这个类的第一个方法，JVM 自动生成的`<init>`方法所对应的属性表，而这个`<init>`很明显是没有参数的，是一个无参的空构造器，那么问题来了，这里的局部变量是从哪里来的？为什么不是 0？
>
> 其实这很好解释，平常编码的时候大家肯定都使用过`this` 这个关键字，通过这个关键字可以在实例方法中拿到当前的实例对象，这个 1 代表其实就是这个`this`，在 Javac 编译的时候会将对 this 的访问转换为对一个方法参数的访问，而这个方法参数会在运行这个实例方法的时候由 JVM 自动的传入，所以局部变量表中至少会存在一个指向当前实例的局部变量

我们继续分析后面的`local_variable_info` 

**local_variable_info 属性结构**

| 类型 | 名称             | 数量 |
| ---- | ---------------- | ---- |
| u2   | start_pc         | 1    |
| u2   | length           | 1    |
| u2   | name_index       | 1    |
| u2   | descriptor_index | 1    |
| u2   | index            | 1    |

⚡`0x0000` 代表`start_pc`，这个局部变量的生命周期开始的字节码偏移量

⚡`0x000A`代表`length`代表这个局部变量其作用范围覆盖的长度

⚡`0x000C`代表`name_index` 是这个局部变量的名字索引，指向常量池中第 12 项常量 `#12 = Utf8   this`符合我们前面的分析

⚡`0x000D`代表`decriptor_index` 局部变量描述符（对变量来说就是变量的类型）的索引，指向常量池第 13 项常量

`#13 = Utf8    Ljvmstudy/classfile_stu/Test1;` 当前实例对象的**全限定名**

⚡`0x0000` 对应`index` 代表这个局部变量在**栈帧局部变量表**中 Slot 的位置

> _**⏳ 到这里我们的方法表的第一个方法`<init>`的 Code 属性就结束了，由于`<init>`方法不包含其他的属性所以`<init>`方法在字节码中也已经结束了**_，后面的方法就不再逐个字节的分析了，都是一样的，主要的是要搞清楚这些属性的层级和包含关系，不要搞混了

### Exceptions 属性

`Exceptions` 属性和上面 Code 属性是平级的，和 Code 属性附带的`exception_table`并不是一个东西，不要搞混了，Exceptions 属性的作用是列举出方法中可能抛出的受检查异常，也就是方法描述时在`throws`关键字后面列举的异常，结构如下

| 类型 | 名称                  | 数量                 |
| ---- | --------------------- | -------------------- |
| u2   | attribute_name_index  | 1                    |
| u4   | attribute_length      | 1                    |
| u2   | number_of_exceptions  | 1                    |
| u2   | exception_index_table | number_of_exceptions |

`number_of_exceptions`表示可能抛出多少种受检查异常，每种异常都是一个`exception_index_table` 很明显这个是一个索引，指向常量池中对应的 Exception 的描述符

这里我们的`<init>` 方法并没有抛出异常，所以这一项属性并不存在，那我们找一个有异常的来看看

```java
public class Test2 {

    private static int bbbb = 99;

    private final static int aaaa = 99;

    private List<Integer> list=null;

    public Test2(int a) {
        bbbb = a;
    }

    public int inc() throws ArithmeticException{
        int x;
        try {
            x=1;
            return  x;
        }catch (Exception e){
            x=2;
            return x;
        }finally {
            x=3;
        }
    }

    @Deprecated
    public void deprecatedMethod(){

    }
}
```

![mark](http://static.imlgw.top/blog/20190905/uG5AQrVoHphH.png?imageslim)

可以看到`exception_table` 里面记录了这个方法的异常处理表，也就是`try-catch`里面的异常处理，这也是 Java 代码的一部分，编译器使用了异常处理表去处理异常和 finally 机制，在 jdk1.4 之前使用的是简单的跳转指令来实现，而 Exceptions 属性和 Code 平级，只是列举了一些可能抛出的异常

```java
Exceptions:
  throws java.lang.ArithmeticException
```
## SourceFile 属性

这个属性是属于 Class 文件的属性，很明显是用来记录生成这个 Class 文件的源代码名称，其值指向一个 CONSTANT_Utf8_info 的索引，值就是源文件的名字

## ConstantValue 属性

这个属性在前一篇 [类加载器](http://imlgw.top/2019/08/17/shen-ru-li-jie-java-xu-ni-ji-er/#%E5%87%86%E5%A4%87) 中提到过， `ConstantValue`属性的作用是通知虚拟机自动为静态变量赋值，只有被 static 关键字修饰的变量（类变量）可以使用这项属性

如果**同时使用 final 和 static 来修饰一个变量**，并且这个变量的数据类型是基本类型或者`java.lang.String`的话，就生成`ConstantValue`属性，用于在**准备阶段**给变量赋初始值，如果这个变量没有被 final 修饰或者并非基本类型及字符串，则在准备阶段会被初始化为默认的零值，在`<clinit>`方法中进行真正的初始化

**依然是上面 Exceptions 的 Demo**

![mark](http://static.imlgw.top/blog/20190905/U8zwbs2c0n8a.png?imageslim)

可以看到`aaaa` 这个常量字段附带了 `ConstantValue` 属性其值就是指向常量池中 99 的一个索引

> 🎯 后面还有一些属性就不再详细介绍了
>
> Signature 用来记录泛型的信息，StackMapTable 用来验证字节码，BootstarapMethods 用来保存动态调用的引导方法限定符，Deprecated 和 Synthetic 两个 boolean 属性。.....

## Synchnorized 字节码分析

**synchnorized **关键字可以用来修饰方法体，或者方法体内的代码块，修饰方法体的时候同步是隐式的，无需通过字节码指令来控制，它实现在方法调用和返回中，为了看到字节码的变化，这里我们不讨论这种（两种方式的底层实现其实还是一样的）

```java
public class Test4 {
    public static Integer a = 0;

    public void setA(int x) {
        synchronized (this) {
            a = x;
        }
    }
}
```

编译后利用工具查看 setA 方法内生成的字节码指令（省略了次要的一些信息）

```java
0: aload_0           //对象引用 this 入栈
1: dup	             //复制栈顶元素 (this)
2: astore_2          //将栈顶元素 (this) 存入局部变量表 Slot 索引为 2 的位置
3: monitorenter      //以栈顶元素 (this) 为锁进入同步块
4: iload_1	         //局部变量 Slot 1 位置的元素 (x) 入栈
5: invokestatic  #2  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
8: putstatic     #3  // Field a:Ljava/lang/Integer; 用 x 给 a 赋值
11: aload_2          //局部变量表 Slot 2 元素 (this) 入栈
12: monitorexit      //退出同步
13: goto          21 //跳转到 21 行结束
16: astore_3         //到这里说明异常了，将栈顶元素（异常对象）存入局部变量表 Slot 3 位置
17: aload_2          //局部变量表 Slot 2 元素 (this) 位置入栈
18: monitorexit      //退出同步
19: aload_3          //局部变量表 Slot 3 位置元素入栈
20: athrow           //抛出异常
21: return           //方法正常返回
Exception table:
    from    to  target type
       4    13    16   any
      16    19    16   any

```
最开始没有学字节码的时候一直很纳闷为啥有两个`monitorexit` ，现在算是明白了，这是为了保证 synchnorized 在方法异常的情况下仍然可以正常的释放锁，而不至于导致锁泄露，这也是比较推荐使用 synchnorized 的原因之一

通过字节码可以看到，编译器为我们自动的生成了一个异常表，也就是前面 Code 属性中携带的 exception_table 属性，如果在指定的程序段内发生异常，会按照异常表指定的 target 进行跳转，无论如何都会释放这个锁

## 总结 & 参考

这一部分主要记录了 Class 字节码的文件结构相关的内容，也算是是逐个字节的分析了一遍，收获还是挺大的，对 JVM 平台的理解又加深了，当然这一篇也是给下一篇**虚拟机节码的执行引擎**做铺垫

**_《深入理解 Java 虚拟机》——周志明_**
