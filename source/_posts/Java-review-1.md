---
title: Java复健系列(1)：Java基础知识
date: 2020-10-16 13:30:20
categories: Java
tags:
description: Java复健系列(1)：复习Java基本概念、语法、基本类型、方法等知识。
---

# Java基本概念
## Java 语言有哪些特点?
- 简单易学；
- 面向对象（封装，继承，多态）；
- 平台无关性（ Java 虚拟机实现平台无关性）；
- 可靠性；
- 安全性；
- 支持多线程（ C++ 语言没有内置的多线程机制，因此必须调用操作系统的多线程功能来进行多线程程序设计，而 Java 语言却提供了多线程支持）；
- 支持网络编程并且很方便（ Java 语言诞生本身就是为简化网络编程设计的，因此 Java 语言不仅支持网络编程而且很方便）；
- 编译与解释并存；

## 关于 JVM JDK 和 JRE
 - Java 虚拟机（JVM）是运行 Java 字节码的虚拟机。JVM 有针对不同系统的特定实现（Windows，Linux，macOS），目的是使用相同的字节码，它们都会给出相同的结果。
 - JDK 是 Java Development Kit 缩写，它是功能齐全的 Java SDK。它拥有 JRE 所拥有的一切，还有编译器（javac）和工具（如 javadoc 和 jdb）。它能够创建和编译程序。
 - JRE 是 Java 运行时环境(Java Runtime Environment)。它是运行已编译 Java 程序所需的所有内容的集合，包括 Java 虚拟机（JVM），Java 类库，java 命令和其他的一些基础构件。但是，它不能用于创建新程序。

## 字节码
在 Java 中，JVM 可以理解的代码就叫做字节码（即扩展名为 .class 的文件），它不面向任何特定的处理器，只面向虚拟机。Java 语言通过字节码的方式，在一定程度上解决了传统解释型语言执行效率低的问题，同时又保留了解释型语言可移植的特点。所以 Java 程序运行时比较高效，而且，由于字节码并不针对一种特定的机器，因此，Java 程序无须重新编译便可在多种不同操作系统的计算机上运行。

源代码（java文件）->字节码（class文件）->机器码
 
## 为什么 Java 是编译与解释共存的语言。
高级编程语言按照程序的执行方式分为编译型和解释型两种。简单来说，编译型语言是指编译器针对特定的操作系统将源代码一次性翻译成可被该平台执行的机器码；解释型语言是指解释器对源程序逐行解释成特定平台的机器码并立即执行。比如，你想阅读一本英文名著，你可以找一个英文翻译人员帮助你阅读， 有两种选择方式，你可以先等翻译人员将全本的英文名著（也就是源码）都翻译成汉语，再去阅读，也可以让翻译人员翻译一段，你在旁边阅读一段，慢慢把书读完。

Java 语言既具有编译型语言的特征，也具有解释型语言的特征，因为 Java 程序要经过先编译，后解释两个步骤，由 Java 编写的程序需要先经过编译步骤，生成字节码（*.class 文件），这种字节码必须由 Java 解释器来解释执行。因此，我们可以认为 Java 语言编译与解释并存。

在.class->机器码 这一步 JVM 类加载器首先加载字节码文件，然后通过解释器逐行解释执行，这种方式的执行速度会相对比较慢。而且，有些方法和代码块是经常需要被调用的(也就是所谓的热点代码)，所以引进了 JIT 编译器，而 JIT 属于运行时编译。当 JIT 编译器完成第一次编译后，其会将字节码对应的机器码保存下来，下次可以直接使用。

HotSpot 采用了惰性评估(Lazy Evaluation)的做法，根据二八定律，消耗大部分系统资源的只有那一小部分的代码（热点代码），而这也就是 JIT 所需要编译的部分。JVM 会根据代码每次被执行的情况收集信息并相应地做出一些优化，因此执行的次数越多，它的速度就越快。

## Oracle JDK与OpenJDK
1. Oracle JDK 大概每 6 个月发一次主要版本，而 OpenJDK 版本大概每三个月发布一次。详情参见：https://blogs.oracle.com/java-platform-group/update-and-faq-on-the-java-se-release-cadence 。
OpenJDK 是一个参考模型并且是完全开源的，而 Oracle JDK 是 OpenJDK 的一个实现，并不是完全开源的；
2. Oracle JDK 比 OpenJDK 更稳定。OpenJDK 和 Oracle JDK 的代码几乎相同，但 Oracle JDK 有更多的类和一些错误修复。因此，如果您想开发企业/商业软件，我建议您选择 Oracle JDK，因为它经过了彻底的测试和稳定。某些情况下，有些人提到在使用 OpenJDK 可能会遇到了许多应用程序崩溃的问题，但是，只需切换到 Oracle JDK 就可以解决问题；
3. 在响应性和 JVM 性能方面，Oracle JDK 与 OpenJDK 相比提供了更好的性能；
Oracle JDK 不会为即将发布的版本提供长期支持，用户每次都必须通过更新到最新版本获得支持来获取最新版本；
4. Oracle JDK 根据二进制代码许可协议获得许可，而 OpenJDK 根据 GPL v2 许可获得许可。

## Java和C++区别
- 都是面向对象的语言，都支持封装、继承和多态
- Java 不提供指针来直接访问内存，程序内存更加安全
- Java 的类是单继承的，C++ 支持多重继承；虽然 Java 的类不可以多继承，但是可以通过接口实现多继承。
- Java 有自动内存管理垃圾回收机制(GC)，不需要程序员手动释放无用内存
- 在 C 语言中，字符串或字符数组最后都会有一个额外的字符'\0'来表示结束。但是，Java 语言中没有结束符这一概念。 这是一个值得深度思考的问题。简单来说可以视为Java万物皆对象，不需要额外的'\0'来表示结束而是通过length确定。具体原因推荐看这篇文章： https://blog.csdn.net/sszgg2006/article/details/49148189


## Java程序的主类是什么？应用程序和Applet程序的主类区别？
一个程序中可以有多个类，但只能有一个类是主类。在 Java 应用程序中，这个主类是指包含 main() 方法的类。而在 Java 小程序中，这个主类是一个继承自系统类 JApplet 或 Applet 的子类。应用程序的主类不一定要求是 public 类，但小程序的主类要求必须是 public 类。主类是 Java 程序执行的入口点。

## Java主程序与Applet小程序区别
简单说应用程序是从主线程启动(也就是 main() 方法)。applet 小程序没有 main() 方法，主要是嵌在浏览器页面上运行(调用init()或者run()来启动)，嵌入浏览器这点跟 flash 的小游戏类似。

## Java和javax有什么区别？
刚开始的时候 JavaAPI 所必需的包是 java 开头的包，javax 当时只是扩展 API 包来使用。然而随着时间的推移，javax 逐渐地扩展成为 Java API 的组成部分。但是，将扩展从 javax 包移动到 java 包确实太麻烦了，最终会破坏一堆现有的代码。因此，最终决定 javax 包将成为标准 API 的一部分。
所以，实际上 java 和 javax 没有区别。这都是一个名字。

# Java基本类型
Java中基本变量类型(primitive type)占据内存空间如图所示。Java中基本变量的大小不随硬件架构变化而变化。这种不变性是Java程序更加具有可移植性的原因之一。

变量类型|存储大小|最小值|最大值|注释|包装器类型
---|:---|:--:|:--:|:---:|---:
byte|1byte|-128|+127|字节|Character
int|4byte|$-2^{31}$|$+2^{31}-1$|整形|Integer
short|2bytes|$-2^{15}$|$+2^{15}-1$|短整形|Short
long|8bytes|$-2^{63}$|$+2^{63}-1$|长整形|Long
float|4bytes|IEEE754|IEEE753|单精度浮点数|Float
double|8bytes|IEEE754|IEEE754|双精度浮点数|Double
char|2bytes|Unicode 0|Unicode $2^{16}-1$|字符|Character
boolean|1bit|-|-|布尔值|Boolean

注意：
1. 对于boolean，官方文档未明确定义，它依赖于 JVM 厂商的具体实现。逻辑上理解是占用 1位，但是实际中会考虑计算机高效存储因素。
2. Java 里使用 long 类型的数据一定要在数值后面加上 L，否则将作为整型解析

## 包装类（装箱与拆箱）

八种基本数据类型并不支持面向对象编程，基本类型的数据不具备“对象”的特性——不携带属性、没有方法可调用。Java为每种基本数据类型分别设计了对应的类，称之为包装类(Wrapper Classes)。每个包装类的对象可以封装一个相应的基本类型的数据，并提供了其它一些有用的方法。包装类对象一经创建，其内容（所封装的基本类型数据值）不可改变。

基本类型和对应的包装类可以相互装换：
- 由基本类型向对应的包装类转换称为装箱，例如把 int 包装成 Integer 类的对象；
- 包装类向对应的基本类型转换称为拆箱，例如把 Integer 类的对象重新简化为 int。

### 实现 int 和 Integer 的相互转换
可以通过 Integer 类的构造方法将 int 装箱，通过 Integer 类的 intValue 方法将 Integer 拆箱。

### 将字符串转换为整数
Integer 类有一个静态的 paseInt() 方法，可以将字符串转换为整数，语法为：
```
parseInt(String s, int radix);

```
### 将整数转换为字符串
Integer 类有一个静态的 toString() 方法，可以将整数转换为字符串。

### 自动拆箱和装箱
上面的例子都需要手动实例化一个包装类，称为手动拆箱装箱。Java 1.5(5.0) 之前必须手动拆箱装箱。

Java 1.5 之后可以自动拆箱装箱，也就是在进行基本数据类型和对应的包装类转换时，系统将自动进行
```
public class Demo {
    public static void main(String[] args) {
        int m = 500;
        Integer obj = m;  // 自动装箱
        int n = obj;  // 自动拆箱
        System.out.println("n = " + n);
      
        Integer obj1 = 500;
        System.out.println("obj 等价于 obj1？" + obj.equals(obj1));
    }
}
```

## 常量池
Java 基本类型的包装类的大部分都实现了常量池技术，即 Byte,Short,Integer,Long,Character,Boolean；前面 4 种包装类默认创建了数值[-128，127] 的相应类型的缓存数据，Character创建了数值在[0,127]范围的缓存数据，Boolean 直接返回True Or False。如果超出对应范围仍然会去创建新的对象。

应用场景：
- Integer i1=40；Java 在编译的时候会直接将代码封装成 Integer i1=Integer.valueOf(40);，从而使用常量池中的对象。
- Integer i1 = new Integer(40);这种情况下会创建新的对象。
```
  Integer i1 = 40;
  Integer i2 = 40;
  Integer i3 = 0;
  Integer i4 = new Integer(40);
  Integer i5 = new Integer(40);
  Integer i6 = new Integer(0);
  
  System.out.println("i1=i2   " + (i1 == i2));
  System.out.println("i1=i2+i3   " + (i1 == i2 + i3));
  System.out.println("i1=i4   " + (i1 == i4));
  System.out.println("i4=i5   " + (i4 == i5));
  System.out.println("i4=i5+i6   " + (i4 == i5 + i6));   
  System.out.println("40=i5+i6   " + (40 == i5 + i6));   
```
输出：
```
i1=i2   true
i1=i2+i3   true
i1=i4   false
i4=i5   false
i4=i5+i6   true
40=i5+i6   true
```

# Java基本语法
## 数组
数组声明：
```
int[] a
```
声明时数组没有分配空间。使用new创建数组：
```
int[] a = new int[100]
```
声明同时赋值（变长数组）：
```
int[] a = new int[]{1,3,5,7,9}
```

### 数组初始化
```
// 静态初始化
// 静态初始化的同时就为数组元素分配空间并赋值
int intArray[] = {1,2,3,4};

// 动态初始化
float floatArray[] = new float[3]; // 分配空间
// 赋值
floatArray[0] = 1.0f;
floatArray[1] = 132.63f;
floatArray[2] = 100F;
```
### 引用
```
arrayName[index]
```
index从0开始。与C、C++不同，Java对数组元素要进行越界检查以保证安全性。\
每个数组都有一个length属性来指明它的长度，例如 intArray.length 指明数组 intArray 的长度。

### 数组遍历
经典for循环
```
int arrayDemo[] = {1, 2, 4, 7, 9, 192, 100};
for(int i=0,len=arrayDemo.length; i<len; i++){
    System.out.println(arrayDemo[i] + ", ");
}
```
Java特例。foreach循环。每次循环自动获取下一个元素的值
```
for(arrayType varName:arrayName){
    //some code
}
```
```
int arrayDemo[] = {1, 2, 4, 7, 9, 192, 100};
for(int x: arrayDemo){
    System.out.println(x + ", ");
}
```

### 二维数组
```
int intArray[ ][ ] = { {1,2}, {2,3}, {4,5} };
int a[ ][ ] = new int[2][3];
a[0][0] = 12;
a[0][1] = 34;
// ......
a[1][2] = 93;
```
java中，二维数组被看做数组的数组，空间不连续分配。因此每一维大小可以不同
```
int intArray[ ][ ] = {{1,2}, {2,3,4,5}};
int a[ ][ ] = new int[2][ ];
a[0] = new int[3];
a[1] = new int[5];
```

- 上面讲的是静态数组。静态数组一旦被声明，它的容量就固定了，不容改变。所以在声明数组时，一定要考虑数组的最大容量，防止容量不够的现象。
- 如果想在运行程序时改变容量，就需要用到数组列表(ArrayList，也称动态数组)或向量(Vector)。
- 正是由于静态数组容量固定的缺点，实际开发中使用频率不高，被 ArrayList 或 Vector 代替，因为实际开发中经常需要向数组中添加或删除元素，而它的容量不好预估。

## 表达式
### 数学表达式
名称|种类
--|--
1+2|加法
4-3|减法
3*6|乘法
7/2|除法
7%2|取余

### 布尔表达式
名称|种类
--|--
true&&flase|and
true\|flase|or
!true|not

### 位运算
名称|种类
--|--
&|and
\||or
^|xor
~|not
5<<3|left shift 3 bits
6>>1|right shift 1 bits

## 控制结构
### if
```
if (conditon1) {
  statements;
    ...
}
else if (condition2) {
  statements;
    ...
}
else {
  statements;
    ...
}
```
### while
```
while (condition) {
statements;
}
```
### do while
```
do {
statements;
}while(condition);
```
### for
```
for (initial; condition; update) {
statements;
}
```
### jump out
```
break;
continue;
```
### switch
```
switch(expression) {
case 1:
    statements; 
    break; 
case 2:
    statements; 
    break; 
...
default:
    statements; 
    break; 
}
```

## 字符串
String类
```
String str = "java study"
```
字符串可以用'+'链接，基本数据类型与字符串'+'会自动转换为字符串
```
public class Demo {
    public static void main(String[] args){
        String stuName = "小明";
        int stuAge = 17;
        float stuScore = 92.5f;
       
        String info = stuName + "的年龄是 " + stuAge + "，成绩是 " + stuScore;
        System.out.println(info);
    }
}
```
String类似数组，初始化后长度不变，内容也无法改变。如果修改就会产生新字符串。\
如下，在str后加上'world!'。程序首先产生了str1字符串，并在内存中申请了一段空间。此时要追加新的字符串是不可能的，因为字符串被初始化后，长度是固定的。如果要改变它，只有放弃原来的空间，重新申请能够容纳“Hello World!”字符串的内存空间，然后将“Hello World!”字符串放到内存中。
```
String str = "Hello";
str + = "World!";
```
实际上，String 是java.lang包下的一个类，按照标准的面向对象的语法，其格式应该为：
```
String stringName = new String("string content");
```
例如
```
String url = new String("http://www.baidu.com");
```
但是由于String特别常用，所以Java提供了一种简化的语法。

使用简化语法的另外一个原因是，按照标准的面向对象的语法，在内存使用上存在比较大的浪费。例如String str = new String(“abc”);实际上创建了两个String对象，一个是”abc”对象，存储在常量空间中，一个是使用new关键字为对象str申请的空间。

## 字符型常量与字符串常量的区别？
- 形式上: 字符常量是单引号引起的一个字符; 字符串常量是双引号引起的0个或若干个字符
- 含义上: 字符常量相当于一个整型值( ASCII 值),可以参加表达式运算; 字符串常量代表一个地址值(该字符串在内存中存放位置)
- 占内存大小 字符常量只占 2 个字节; 字符串常量占若干个字节 (注意： char 在 Java 中占两个字节),

## 标识符和关键字？
在我们编写程序的时候，需要大量地为程序、类、变量、方法等取名字，于是就有了标识符，简单来说，标识符就是一个名字。但是有一些标识符，Java 语言已经赋予了其特殊的含义，只能用于特定的地方，这种特殊的标识符就是关键字。因此，关键字是被赋予特殊含义的标识符。比如，在我们的日常生活中 ，“警察局”这个名字已经被赋予了特殊的含义，所以如果你开一家店，店的名字不能叫“警察局”，“警察局”就是我们日常生活中的关键字。

## Java常见关键字

| 访问控制             | private  | protected  | public   |              |            |           |        |
| -------------------- | -------- | ---------- | -------- | ------------ | ---------- | --------- | ------ |
| 类，方法和变量修饰符 | abstract | class      | extends  | final        | implements | interface | native |
|                      | new      | static     | strictfp | synchronized | transient  | volatile  |        |
| 程序控制             | break    | continue   | return   | do           | while      | if        | else   |
|                      | for      | instanceof | switch   | case         | default    |           |        |
| 错误处理             | try      | catch      | throw    | throws       | finally    |           |        |
| 包相关               | import   | package    |          |              |            |           |        |
| 基本类型             | boolean  | byte       | char     | double       | float      | int       | long   |
|                      | short    | null       | true     | false        |            |           |        |
| 变量引用             | super    | this       | void     |              |            |           |        |
| 保留字               | goto     | const      |          |              |            |           |        |


## ++和--
自增运算符（++)和自减运算符（--）。++和--运算符可以放在变量之前，也可以放在变量之后，当运算符放在变量之前时(前缀)，先自增/减，再赋值；当运算符放在变量之后时(后缀)，先赋值，再自增/减。例如，当 b = ++a 时，先自增（自己增加 1），再赋值（赋值给 b）；当 b = a++ 时，先赋值(赋值给 b)，再自增（自己增加 1）。也就是，++a 输出的是 a+1 的值，a++输出的是 a 值。符号在前就先加/减，符号在后就后加/减。

## continue、break、和return的区别是什么？
在循环结构中，当循环条件不满足或者循环次数达到要求时，循环会正常结束。但是，有时候可能需要在循环的过程中，当发生了某种条件之后 ，提前终止循环，这就需要用到下面几个关键词：
1. continue ：指跳出当前的这一次循环，继续下一次循环。
2. break ：指跳出整个循环体，继续执行循环下面的语句。
3. return 用于跳出所在方法，结束该方法的运行。

return 一般有两种用法：
1. return; ：直接使用 return 结束方法执行，用于没有返回值函数的方法
2. return value; ：return 一个特定值，用于有返回值函数的方法

## Java泛型
泛型(generics)的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。

Java的泛型是伪泛型，这是因为Java在编译期间，所有的泛型信息都会被擦掉，这也就是通常所说类型擦除 。如在代码中定义List<Object>和List<String>等类型，在编译后都会变成List，JVM看到的只是List，而由泛型附加的类型信息对JVM是看不到的。Java编译器会在编译时尽可能的发现可能出错的地方，但是仍然无法在运行时刻出现的类型转换异常的情况，类型擦除也是Java的泛型与C++模板机制实现方式之间的重要区别。 更多关于类型擦除的问题，可以查看这篇文章：[《Java泛型类型擦除以及类型擦除带来的问题](https://www.cnblogs.com/wuqinglong/p/9456193.html)。

泛型一般有三种使用方式:泛型类、泛型接口、泛型方法。
### 1. 泛型类
    ```
    //此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
    //在实例化泛型类时，必须指定T的具体类型
    public class Generic<T>{ 
    
        private T key;

        public Generic(T key) { 
            this.key = key;
        }

        public T getKey(){ 
            return key;
        }
    }

    Generic<Integer> genericInteger = new Generic<Integer>(123456);
    ```

### 2. 泛型接口
    ```
    public interface Generator<T> {
        public T method();
    }
    ```
    实现泛型接口，不指定类型：
    ```
    class GeneratorImpl<T> implements Generator<T>{
        @Override
        public T method() {
            return null;
        }
    }
    ```
    实现泛型接口，指定类型：
    ```
    class GeneratorImpl<T> implements Generator<String>{
        @Override
        public String method() {
            return "hello";
        }
    }
    ```
### 3. 泛型方法
   ```
    public static < E > void printArray( E[] inputArray ){         
         for ( E element : inputArray ){        
            System.out.printf( "%s ", element );
         }
         System.out.println();
    }
   ```
常用的通配符为： T，E，K，V，？
- ？ 表示不确定的 java 类型
- T (type) 表示具体的一个java类型
- K V (key value) 分别代表java键值中的Key Value
- E (element) 代表Element

## ==，equals与hashCode
### 1.==
它的作用是判断两个对象的地址是不是相等。即判断两个对象是不是同一个对象。
- 若操作数的类型是基本数据类型，则该关系操作符判断的是左右两边操作数的“值”是否相等
- 若操作数的类型是引用数据类型，则该关系操作符判断的是左右两边操作数的内存地址是否相同。也就是说，若此时返回true,则该操作符作用的一定是同一个对象。

### 2.equals()
它的作用也是判断两个对象是否相等。equals()方法存在于Object类中，而Object类是所有类的直接或间接父类。因此所有类都可以调用equals()来判断。它不能用于比较基本数据类型的变量，因为基本数据类型不是类。

Object类中的equals()方法如下所示。在Object类中，equals方法是用来比较两个对象的引用是否相等，即是否指向同一个对象。
```
public boolean equals(Object obj) {
     return (this == obj);
}
```

equals() 方法存在两种使用情况：
- 情况 1：类没有覆盖 equals()方法。则通过 equals()比较该类的两个对象时，等价于通过“==”比较这两个对象。使用的默认是 Object类equals()方法。
- 情况 2：类覆盖了 equals()方法。一般，我们都覆盖 equals()方法来两个对象的内容相等；若它们的内容相等，则返回 true(即，认为这两个对象相等)。

对于内置类的equals方法，Java内部实现分为三个步骤：
1. 先比较引用是否相同(是否为同一对象),
2. 再判断类型是否一致（是否为同一类型）,
3. 最后比较内容是否一致

Java 中所有内置的类的 equals 方法的实现步骤均是如此，特别是诸如 Integer，Double 等包装器类。例如String类的equals就被重载过，具体实现为对于string中的每个char一一比较来比较两个字符串是否相等。
```
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String) anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                        return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

```
public class test1 {
    public static void main(String[] args) {
        String a = new String("ab"); // a 为一个引用
        String b = new String("ab"); // b为另一个引用,对象的内容一样
        String aa = "ab"; // 放在常量池中
        String bb = "ab"; // 从常量池中查找
        if (aa == bb) // true
            System.out.println("aa==bb");
        if (a == b) // false，非同一对象
            System.out.println("a==b");
        if (a.equals(b)) // true
            System.out.println("aEQb");
        if (42 == 42.0) { // true
            System.out.println("true");
        }
    }
}
```

equals重写规则
- 对称性： 如果x.equals(y)返回是“true”，那么y.equals(x)也应该返回是“true” ；
- 自反性： x.equals(x)必须返回是“true” ；
- 类推性： 如果x.equals(y)返回是“true”，而且y.equals(z)返回是“true”，那么z.equals(x)也应该返回是“true” ；
- 一致性： 如果x.equals(y)返回是“true”，只要x和y内容一直不变，不管你重复x.equals(y)多少次，返回都是“true” ；
- 对称性： 如果x.equals(y)返回是“true”，那么y.equals(x)也应该返回是“true”。
- 任何情况下，x.equals(null)【应使用关系比较符 ==】，永远返回是“false”；x.equals(和x不同类型的对象)永远返回是“false”

equals本意是比较两个对象的 content 是否相同。必要的时候，我们需要重写该方法，避免违背本意，且要遵循上述原则

### 3.hashCode()
#### hashCode是什么
hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个 int 整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。hashCode() 定义在 JDK 的 Object 类中，这就意味着 Java 中的任何类都包含有 hashCode() 函数。另外需要注意的是： Object 的 hashcode 方法是本地方法，也就是用 c 语言或 c++ 实现的，该方法通常用来将对象的 内存地址 转换为整数之后返回。

一般来讲，equals 这个方法是给用户调用的，而 hashcode 方法一般用户不会去调用。

#### 为什么要有hashCode
HashCode来源于Hash表。当你把对象加入 HashSet 时，HashSet 会先计算对象的 hashcode 值来判断对象加入的位置，同时也会与其他已经加入的对象的 hashcode 值作比较，如果没有相符的 hashcode，HashSet 会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用 equals() 方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。这样我们就大大减少了 equals 的次数，相应就大大提高了执行速度。

### 为什么重写 equals 时必须重写 hashCode 方法？
当一个对象作为HashSet对象元素时，HashSet需要通过hashCode与equals来判断，并且是先调用hashCode再调用equals。如果一个对象的equals需要重写，那么他的hashCode也必定需要重写来适应。否则该class的两个对象永远不会相等，即使他们指向相同的数据。

如果两个对象相等，则 hashcode 一定也是相同的。两个对象相等,对两个对象分别调用 equals 方法都返回 true。但是，两个对象有相同的 hashcode 值，它们也不一定是相等的 。因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖。

### 为什么两个对象有相同的 hashcode 值，它们也不一定是相等的？
因为在哈希表中 hashCode() 所使用的杂凑算法也许刚好会让多个对象传回相同的杂凑值。越糟糕的杂凑算法越容易碰撞，但这也与数据值域分布的特性有关（所谓碰撞也就是指的是不同的对象得到相同的 hashCode。

我们刚刚也提到了 HashSet,如果 HashSet 在对比的时候，同样的 hashcode 有多个对象，它会使用 equals() 来判断是否真的相同。也就是说 hashcode 只是用来缩小查找成本。

## 正确使用equals
Object的equals方法容易抛空指针异常，应使用常量或确定有值的对象来调用 equals。

举个例子：
```
// 不能使用一个值为null的引用类型变量来调用非静态方法，否则会抛出异常
String str = null;
if (str.equals("SnailClimb")) {
  ...
} else {
  ..
}
```

运行上面的程序会抛出空指针异常，但是我们把第二行的条件判断语句改为下面这样的话，就不会抛出空指针异常，else 语句块得到执行。：
```
"SnailClimb".equals(str);// false 
```

不过更推荐使用 java.util.Objects#equals(JDK7 引入的工具类)。
```
Objects.equals(null,"SnailClimb");// false
```

原因见java.util.Objects#equals的源码：
```
public static boolean equals(Object a, Object b) {
    // 可以避免空指针异常。如果a==null的话此时a.equals(b)就不会得到执行，避免出现空指针异常。
    return (a == b) || (a != null && a.equals(b));
}
```

## 包装类的比较
所有整型包装类对象值的比较必须使用equals方法。

先看下面这个例子：
```
Integer x = 3;
Integer y = 3;
System.out.println(x == y);// true
Integer a = new Integer(3);
Integer b = new Integer(3);
System.out.println(a == b);//false
System.out.println(a.equals(b));//true
```

当使用自动装箱方式创建一个Integer对象时，当数值在-128 ~127时，会将创建的 Integer 对象缓存起来，当下次再出现该数值时，直接从缓存中取出对应的Integer对象。所以上述代码中，x和y引用的是相同的Integer对象。

# 方法
## 什么是方法的返回值?返回值在类的方法里的作用是什么?
方法的返回值是指我们获取到的某个方法体中的代码执行后产生的结果！（前提是该方法可能产生结果）。返回值的作用是接收出结果，使得它可以用于其他的操作！

## 为什么 Java 中只有值传递？
首先回顾一下在程序设计语言中有关将参数传递给方法（或函数）的一些专业术语。按值调用(call by value)表示方法接收的是调用者提供的值，而按引用调用（call by reference)表示方法接收的是调用者提供的变量地址。一个方法可以修改传递引用所对应的变量值，而不能修改传递值调用所对应的变量值。 它用来描述各种程序设计语言（不只是 Java)中方法参数传递方式。

Java 程序设计语言总是采用按值调用。也就是说，方法得到的是所有参数值的一个拷贝，也就是说，方法不能修改传递给它的任何参数变量的内容。

很多程序设计语言（特别是，C++和 Pascal)提供了两种参数传递的方式：值调用和引用调用。有些程序员认为 Java 程序设计语言对对象采用的是引用调用，实际上，这种理解是不对的。理由很简单，方法得到的是对象引用的拷贝，也就是将参数对象的引用赋值到了拷贝对象中，对象引用及拷贝对象同时引用同一个对象。

因此
- 一个方法不能修改一个基本数据类型的参数（即数值型或布尔型）。
- 一个方法可以改变一个对象参数的状态。
- 一个方法不能让对象参数引用一个新的对象。

## 重载和重写的区别
重载（Overloading）就是同样的一个方法能够根据输入数据的不同，做出不同的处理。发生在同一个类中，方法名必须相同，参数类型不同、个数不同、顺序不同，方法返回值和访问修饰符可以不同。

重写（Overwrite）就是当子类继承自父类的相同方法，输入数据一样，但要做出有别于父类的响应时，你就要覆盖父类方法
- 返回值类型、方法名、参数列表必须相同，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类。
- 如果父类方法访问修饰符为 private/final/static 则子类就不能重写该方法，但是被 static 修饰的方法能够被再次声明。
- 构造方法无法被重写

区别点|重载方法|重写方法
----|----|--
发生范围|同一个类|子类
参数列表|必须修改|一定不能修改
返回类型|可修改|子类方法返回值类型应比父类方法返回值类型更小或相等
异常|可修改|子类方法声明抛出的异常类应比父类方法声明抛出的异常类更小或相等；
访问修饰符|可修改|一定不能做更严格的限制（可以降低限制）
发生阶段|编译期|运行期

## 深拷贝和浅拷贝
- 浅拷贝：对基本数据类型进行值传递，对引用数据类型进行引用传递般的拷贝，此为浅拷贝。
- 深拷贝：对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，此为深拷贝。

