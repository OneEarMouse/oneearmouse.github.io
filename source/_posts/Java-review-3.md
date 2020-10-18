---
title: Java复健系列(3)：Java的面向对象
date: 2020-10-18 15:15:36
categories:
tags:
description: Java复健系列(3)：复习Java继承、多态、接口的相关知识。
---

# 面向对象三大特征
## 封装
封装是指把一个对象的状态信息（也就是属性）隐藏在对象内部，不允许外部对象直接访问对象的内部信息。但是可以提供一些可以被外界访问的方法来操作属性。就好像我们看不到挂在墙上的空调的内部的零件信息（也就是属性），但是可以通过遥控器（方法）来控制空调。

## 继承
不同类型的对象，相互之间经常有一定数量的共同点。例如，小明同学、小红同学、小李同学，都共享学生的特性（班级、学号等）。同时，每一个对象还定义了额外的特性使得他们与众不同。例如小明的数学比较好，小红的性格惹人喜爱；小李的力气比较大。继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承，可以快速地创建新的类，可以提高代码的重用，程序的可维护性，节省大量创建新类的时间 ，提高我们的开发效率。

- 子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法子类是无法访问，只是拥有。
- 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。
- 子类可以用自己的方式实现父类的方法。

## 多态
多态，顾名思义，表示一个对象具有多种的状态。具体表现为父类的引用指向子类的实例。
- 对象类型和引用类型之间具有继承（类）/实现（接口）的关系；
- 对象类型不可变，引用类型可变；
- 方法具有多态性，属性不具有多态性；
- 引用类型变量发出的方法调用的到底是哪个类中的方法，必须在程序运行期间才能确定；
- 多态不能调用“只在子类存在但在父类不存在”的方法；
- 如果子类重写了父类的方法，真正执行的是子类覆盖的方法，如果子类没有覆盖父类的方法，执行的是父类的方法。

# 继承
## 继承
关键字 extend
```
class People{
    String name;
    int age;
    int height;
   
    void say(){
        System.out.println("我的名字是 " + name + "，年龄是 " + age + "，身高是 " + height);
    }
}
class Teacher extends People{
    String school;  // 所在学校
    String subject;  // 学科
    int seniority;  // 教龄
   
    // 覆盖 People 类中的 say() 方法
    void say(){
        System.out.println("我叫" + name + "，在" + school + "教" + subject + "，有" + seniority + "年教龄");
    }
   
    void lecturing(){
        System.out.println("我已经" + age + "岁了，依然站在讲台上讲课");
    }
}
```
- 子类可以覆盖父类的方法。
- 子类可以继承父类除private以为的所有的成员。
- 构造方法不能被继承。

单继承性：Java 允许一个类仅能继承一个其它类，即一个类只能有一个父类，这个限制被称做单继承性。后面将会学到接口(interface)的概念，接口允许多继承。

## super 关键字
super 用来表示父类。super 可以用在子类中，通过点号(.)来获取父类的成员变量和方法。super 也可以用在子类的子类中，Java 能自动向上层类追溯。父类行为被调用，就好象该行为是本类的行为一样，而且调用行为不必发生在父类中，它能自动向上层类追溯。
super 关键字的功能：
- 调用父类中声明为 private 的变量。
- 点取已经覆盖了的方法。
- 作为方法名表示父类构造方法。

Java 具有追溯性，会一直向上找，直到找到该方法为止。通过 super 调用父类的private隐藏变量，必须要在父类中声明 getter 方法，因为声明为 private 的数据成员对子类是不可见的。

最后注意 super 与 this 的区别：super 不是一个对象的引用，不能将 super 赋值给另一个对象变量，它只是一个指示编译器调用父类方法的特殊关键字。

### 调用父类方法
```

}
class Animal{
    String name;
    public Animal(String name){
        this.name = name;
    }
}
class Dog extends Animal{
    int age;
    public Dog(String name, int age){
        super(name);
        this.age = age;
    }
```
- 在构造方法中调用另一个构造方法，调用动作必须置于最起始的位置。
- 不能在构造方法以外的任何方法内调用构造方法。
- 在一个构造方法内只能调用一个构造方法。

如果编写一个构造方法，既没有调用 super() 也没有调用 this()，编译器会自动插入一个调用到父类构造方法中，而且不带参数。

## 重写和重载
### 重写（Override）
在类继承中，子类可以修改从父类继承来的方法，也就是说子类能创建一个与父类方法有不同功能的方法，但具有相同的名称、返回值类型、参数列表。如果在新类中定义一个方法，其名称、返回值类型和参数列表正好与父类中的相同，那么，新方法被称做重写旧方法。参数列表又叫参数签名，包括参数的类型、参数的个数和参数的顺序，只要有一个不同就叫做参数列表不同。被覆盖的方法在子类中只能通过super调用。

注意：重写不会删除父类中的方法，而是对子类的实例隐藏，暂时不使用。
```
class Animal{
    String name;
    public Animal(String name){
        this.name = name;
    }
   
    public void say(){
        System.out.println("我是一只小动物，我的名字叫" + name + "，我会发出叫声");
    }
}
class Dog extends Animal{
    // 构造方法不能被继承，通过super()调用
    public Dog(String name){
        super(name);
    }
    // 覆盖say() 方法
    public void say(){
        System.out.println("我是一只小狗，我的名字叫" + name + "，我会发出汪的叫声");
    }
}
```
原则：
- 覆盖方法的返回类型、方法名称、参数列表必须与原方法的相同。
- 覆盖方法不能比原方法访问性差（即访问权限不允许缩小）。
- 覆盖方法不能比原方法抛出更多的异常。
- 被覆盖的方法不能是final类型，因为final修饰的方法是无法覆盖的。
- 被覆盖的方法不能为private，否则在其子类中只是新定义了一个方法，并没有对其进行覆盖。
- 被覆盖的方法不能为static。如果父类中的方法为静态的，而子类中的方法不是静态的，但是两个方法除了这一点外其他都满足覆盖条件，那么会发生编译错误；反之亦然。即使父类和子类中的方法都是静态的，并且满足覆盖条件，但是仍然不会发生覆盖，因为静态方法是在编译的时候把静态方法和类的引用类型进行匹配。

### 重载Overload
前面已经对Java方法重载进行了说明，这里再强调一下，Java父类和子类中的方法都会参与重载，例如，父类中有一个方法是 func(){ … }，子类中有一个方法是 func(int i){ … }，就构成了方法的重载。

覆盖和重载的不同：

方法覆盖要求参数列表必须一致，而方法重载要求参数列表必须不一致。
方法覆盖要求返回类型必须一致，方法重载对此没有要求。
方法覆盖只能用于子类覆盖父类的方法，方法重载用于同一个类中的所有方法（包括从父类中继承而来的方法）。
方法覆盖对方法的访问权限和抛出的异常有特殊的要求，而方法重载在这方面没有任何限制。
父类的一个方法只能被子类覆盖一次，而一个方法可以在所有的类中可以被重载多次

## 重写与重载之间的区别
区别点|重载方法|重写方法
--|--|--
参数列表|必须修改|一定不能修改
返回类型|可以修改|一定不能修改
异常|可以修改|可以减少或删除，一定不能抛出新的或者更广的异常
访问|可以修改|一定不能做更严格的限制（可以降低限制）

# 多态
必要条件
- 继承
- 重写
- 父类引用指向子类对象
```
public class Demo {
    public static void main(String[] args){
        Animal obj = new Animal();
        obj.cry();
        obj = new Cat();
        obj.cry();
        obj = new Dog();
        obj.cry();
    }
}
class Animal{
    // 动物的叫声
    public void cry(){
        System.out.println("不叫");
    }
   
}
class Cat extends Animal{
    // 猫的叫声
    public void cry(){
        System.out.println("喵");
    }
}
class Dog extends Animal{
    // 狗的叫声
    public void cry(){
        System.out.println("汪");
    }
}
```

多态的一个好处是：当子类比较多时，也不需要定义多个变量，可以只定义一个父类类型的变量来引用不同子类的实例

## 动态绑定
Java调用方法详细流程
- 编译器查看对象声明类型和方法名。一一列举所有类中名为func的方法和其父类中访问属性为 public 且名为func的方法。这样，编译器就获得了所有可能被调用的候选方法列表。
- 检查调用方法时提供的参数签名。重载解析（overloading resolution）。编译器自动选举对应参数签名的函数。如果没有匹配，则编译错误
- 如果方法的修饰符是private、static、final（static和final将在后续讲解），或者是构造方法，那么编译器将可以准确地知道应该调用哪个方法，我们将这种调用方式 称为静态绑定(static binding)。
- 当程序运行，并且釆用动态绑定调用方法时，JVM一定会调用与 obj 所引用对象的实际类型最合适的那个类的方法。

每次调用方法都要进行搜索，时间开销相当大，因此，JVM预先为每个类创建了一个方法表(method lable)，其中列出了所有方法的名称、参数签名和所属的类。这样一来，在真正调用方法的时候，虚拟机仅查找这个表就行了。

## instanceof 运算符
多态性带来了一个问题，就是如何判断一个变量所实际引用的对象的类型 。 C++使用runtime-type information(RTTI)，Java 使用 instanceof 操作符。instanceof 运算符用来判断一个变量所引用的对象的实际类型，注意是它引用的对象的类型，不是变量的类型。
```
variable instanceof classname
```
如果变量引用的是当前类或它的子类的实例，instanceof 返回 true，否则返回 false。

## 多态对象的类型转换
我们将子类向父类转换称为“向上转型”，将父类向子类转换称为“向下转型”。很多时候，我们会将变量定义为父类的类型，却引用子类的对象，这个过程就是向上转型。程序运行时通过动态绑定来实现对子类方法的调用，也就是多态性。然而有些时候为了完成某些父类没有的功能，我们需要将向上转型后的子类对象再转成子类，调用子类的方法，这就是向下转型。\
注意：不能直接将父类的对象强制转换为子类类型，只能将向上转型后的子类对象再次转换为子类类型。也就是说，子类对象必须向上转型后，才能再向下转型。
```
class Demo {
    public static void main(String args[]) {
        SuperClass superObj = new SuperClass();
        SonClass sonObj = new SonClass();
        // 下面的代码运行时会抛出异常，不能将父类对象直接转换为子类类型
        // SonClass sonObj2 = (SonClass)superObj;
        // 先向上转型，再向下转型
        superObj = sonObj;
        SonClass sonObj1 = (SonClass)superObj;
    }
}
class SuperClass{ }
class SonClass extends SuperClass{ }
```
因为向下转型存在风险，所以在接收到父类的一个引用时，请务必使用 instanceof 运算符来判断该对象是否是你所要的子类。



# 内部类
在 Java 中，允许在一个类（或方法、语句块）的内部定义另一个类，称为内部类(Inner Class)，有时也称为嵌套类(Nested Class)。

内部类和外层封装它的类之间存在逻辑上的所属关系，一般只用在定义它的类或语句块之内，实现一些没有通用意义的功能逻辑，在外部引用它时必须给出完整的名称。

使用内部类的主要原因有：
- 内部类可以访问外部类中的数据，包括私有的数据。
- 内部类可以对同一个包中的其他类隐藏起来。
- 当想要定义一个回调函数且不想编写大量代码时，使用匿名(anonymous)内部类比较便捷。
- 减少类的命名冲突。

```
public class Demo {
    private int size;
    public class Inner {
        private int counter = 10;
        public void doStuff() {
            size++;
        }
    }
    public static void main(String args[]) {
        Demo outer = new Demo();
        Inner inner = outer.new Inner();
        inner.doStuff();
        System.out.println(outer.size);
        System.out.println(inner.counter);
        // 编译错误，外部类不能访问内部类的变量
//        System.out.println(counter);
    }
}
```
编译，会生成两个 `.class `文件：`Outer.class` 和 `Outer$Inner.class`。也就是说，内部类会被编译成独立的字节码文件。内部类是一种编译器现象，与虚拟机无关。编译器将会把内部类翻译成用`$`符号分隔外部类名与内部类名的常规类文件，而虚拟机则对此一无所知。

注意：必须先有外部类的对象才能生成内部类的对象，因为内部类有时需要访问外部类中的成员变量，成员变量必须实例化才有意义。

内部类是 Java 1.1 的新增特性，有些程序员认为这是一个值得称赞的进步，但是内部类的语法很复杂，严重破坏了良好的代码结构， 违背了Java要比C++更加简单的设计理念。

## 内部类的分类
内部类可以是静态(static)的，可以使用 public、protected 和 private 访问控制符，而外部类只能使用 public，或者默认。

在外部类内部直接定义（不在方法内部或代码块内部）的类就是成员式内部类，它可以直接使用外部类的所有变量和方法，即使是 private 的。外部类要想访问内部类的成员变量和方法，则需要通过内部类的对象来获取。

# 抽象类
在自上而下的继承层次结构中，位于上层的类更具有通用性，甚至可能更加抽象。从某种角度看，祖先类更加通用，它只包含一些最基本的成员，人们只将它作为派生其他类的基类，而不会用来创建对象。甚至，你可以只给出方法的定义而不实现，由子类根据具体需求来具体实现。这种只给出方法定义而不具体实现的方法被称为抽象方法，抽象方法是没有方法体的，在代码的表达上就是没有“{}”。包含一个或多个抽象方法的类也必须被声明为抽象类。

抽象类除了包含抽象方法外，还可以包含具体的变量和具体的方法。类即使不包含抽象方法，也可以被声明为抽象类，防止被实例化。抽象类不能被实例化，抽象方法必须在子类中被实现。
```
import static java.lang.System.*;
public  class Demo{
    public static void main(String[] args) {
        Teacher t = new Teacher();
        t.setName("王明");
        t.work();
       
        Driver d = new Driver();
        d.setName("小陈");
        d.work();
    }
}
// 定义一个抽象类
abstract class People{
    private String name;  // 实例变量
   
    // 共有的 setter 和 getter 方法
    public void setName(String name){
        this.name = name;
    }
    public String getName(){
        return this.name;
    }
   
    // 抽象方法
    public abstract void work();
}
class Teacher extends People{
    // 必须实现该方法
    public void work(){
        out.println("我的名字叫" + this.getName() + "，我正在讲课，请大家不要东张西望...");
    }
}
class Driver extends People{
    // 必须实现该方法
    public void work(){
        out.println("我的名字叫" + this.getName() + "，我正在开车，不能接听电话...");
    }
}
```

关于抽象类的几点说明：
- 抽象类不能直接使用，必须用子类去实现抽象类，然后使用其子类的实例。然而可以创建一个变量，其类型是一个抽象类，并让它指向具体子类的一个实例，也就是可以使用抽象类来充当形参，实际实现类作为实参，这是多态的应用。
- 不能有抽象构造方法或抽象静态方法。

在下列情况下，一个类将成为抽象类：
- 当一个类的一个或多个方法是抽象方法时；
- 当类是一个抽象类的子类，并且不能为任何抽象方法提供任何实现细节或方法主体时；
- 当一个类实现一个接口，并且不能为任何抽象方法提供实现细节或方法主体时；

注意：这里说的是这些情况下一个类将成为抽象类，没有说抽象类一定会有这些情况。一个典型的错误：抽象类一定包含抽象方法。 但是反过来说“包含抽象方法的类一定是抽象类”就是正确的。事实上，抽象类可以是一个完全正常实现的类

# 接口
## 接口的概念
在抽象类中，可以包含一个或多个抽象方法；但在接口(interface)中，所有的方法必须都是抽象的，不能有方法体，它比抽象类更加“抽象”。

接口使用 interface 关键字来声明，可以看做是一种特殊的抽象类，可以指定一个类必须做什么，而不是规定它如何去做。

现实中也有很多接口的实例，比如说串口电脑硬盘，Serial ATA委员会指定了Serial ATA 2.0规范，这种规范就是接口。Serial ATA委员会不负责生产硬盘，只是指定通用的规范。

希捷、日立、三星等生产厂家会按照规范生产符合接口的硬盘，这些硬盘就可以实现通用化，如果正在用一块160G日立的串口硬盘，现在要升级了，可以购买一块320G的希捷串口硬盘，安装上去就可以继续使用了。

下面的代码可以模拟Serial ATA委员会定义以下串口硬盘接口：
```
public interface Demo{
    //连接线的数量
    public static final int CONNECT_LINE=4;
    //写数据
    public void writeData(String data);
    //读数据
    public String readData();
}
```

注意：接口中声明的成员变量默认都是 public static final 的，必须显式地初始化。因而在常量声明时可以省略这些修饰符。

接口是若干常量和抽象方法的集合，目前看来和抽象类差不多。确实如此，接口本就是从抽象类中演化而来的，因而除特别规定，接口享有和类同样的“待遇”。比如，源程序中可以定义多个类或接口，但最多只能有一个public 的类或接口，如果有则源文件必须取和public的类和接口相同的名字。和类的继承格式一样，接口之间也可以继承，子接口可以继承父接口中的常量和抽象方法并添加新的抽象方法等。

接口特性：
1. 接口中只能定义抽象方法，这些方法默认为 public abstract 的，因而在声明方法时可以省略这些修饰符。试图在接口中定义实例变量、非抽象的实例方法及静态方法，都是非法的。例如：
```
public interface SataHdd{
    //连接线的数量
    public int connectLine; //编译出错，connectLine被看做静态常量，必须显式初始化
    //写数据
    protected void writeData(String data); //编译出错，必须是public类型
    //读数据
    public static String readData(){ //编译出错，接口中不能包含静态方法
        return "数据"; //编译出错，接口中只能包含抽象方法，
    }
}
```
2. 接口没有构造方法，不能初始化
3.  一个接口不实现另一个接口，但可以继承多个其他接口。接口的多继承特点弥补了类的单继承。例如：
```
//串行硬盘接口
public interface SataHdd extends A,B{
    // 连接线的数量
    public static final int CONNECT_LINE = 4;
    // 写数据
    public void writeData(String data);
    // 读数据
    public String readData();
}
interface A{
    public void a();
}
interface B{
    public void b();
}
```

## why 接口
大型项目开发中，可能需要从继承链的中间插入一个类，让它的子类具备某些功能而不影响它们的父类。例如 A -> B -> C -> D -> E，A 是祖先类，如果需要为C、D、E类添加某些通用的功能，最简单的方法是让C类再继承另外一个类。但是问题来了，Java 是一种单继承的语言，不能再让C继承另外一个父类了，只到移动到继承链的最顶端，让A再继承一个父类。这样一来，对C、D、E类的修改，影响到了整个继承链，不具备可插入性的设计。

接口是可插入性的保证。在一个继承链中的任何一个类都可以实现一个接口，这个接口会影响到此类的所有子类，但不会影响到此类的任何父类。此类将不得不实现这个接口所规定的方法，而子类可以从此类自动继承这些方法，这时候，这些子类具有了可插入性。

我们关心的不是哪一个具体的类，而是这个类是否实现了我们需要的接口。

接口提供了关联以及方法调用上的可插入性，软件系统的规模越大，生命周期越长，接口使得软件系统的灵活性和可扩展性，可插入性方面得到保证。

接口在面向对象的 Java 程序设计中占有举足轻重的地位。事实上在设计阶段最重要的任务之一就是设计出各部分的接口，然后通过接口的组合，形成程序的基本框架结构。

## 接口的使用
接口的使用与类的使用有些不同。在需要使用类的地方，会直接使用new关键字来构建一个类的实例，但接口不可以这样使用，因为接口不能直接使用 new 关键字来构建实例。

接口必须通过类来实现(implements)它的抽象方法，然后再实例化类。类实现接口的关键字为implements。

如果一个类不能实现该接口的所有抽象方法，那么这个类必须被定义为抽象方法。

不允许创建接口的实例，但允许定义接口类型的引用变量，该变量指向了实现接口的类的实例。

一个类只能继承一个父类，但却可以实现多个接口。

实现接口的格式如下：
```
修饰符 class 类名 extends 父类 implements 多个接口(逗号分隔) {
实现方法
}
```

```
import static java.lang.System.*;
public class Demo{
  public static void main(String[] args) {
      SataHdd sh1=new SeagateHdd(); //初始化希捷硬盘
      SataHdd sh2=new SamsungHdd(); //初始化三星硬盘
  }
}
//串行硬盘接口
interface SataHdd{
    //连接线的数量
    public static final int CONNECT_LINE=4;
    //写数据
    public void writeData(String data);
    //读数据
    public String readData();
}
// 维修硬盘接口
interface fixHdd{
    // 维修地址
    String address = "北京市海淀区";
    // 开始维修
    boolean doFix();
}
//捷硬盘
class SeagateHdd implements SataHdd, fixHdd{
    //希捷硬盘读取数据
    public String readData(){
        return "数据";
    }
    //希捷硬盘写入数据
    public void writeData(String data) {
        out.println("写入成功");
    }
    // 维修希捷硬盘
    public boolean doFix(){
        return true;
    }
}
//三星硬盘
class SamsungHdd implements SataHdd{
    //三星硬盘读取数据
    public String readData(){
        return "数据";
    }
    //三星硬盘写入数据
    public void writeData(String data){
        out.println("写入成功");
    }
}
//某劣质硬盘，不能写数据
abstract class XXHdd implements SataHdd{
    //硬盘读取数据
    public String readData() {
        return "数据";
    }
}
```

## 接口作为类使用
接口作为引用类型来使用，任何实现该接口的类的实例都可以存储在该接口类型的变量中，通过这些变量可以访问类中所实现的接口中的方法，Java 运行时系统会动态地确定应该使用哪个类中的方法，实际上是调用相应的实现类的方法。下面例子中可以看到接口可以作为一个类型来使用，把接口作为方法的参数和返回类型。
```
public class Demo{
    public void test1(A a) {
        a.doSth();
    }
    public static void main(String[] args) {
        Demo d = new Demo();
        A a = new B();
        d.test1(a);
    }
}
interface A {
    public int doSth();
}
class B implements A {
    public int doSth() {
        System.out.println("now in B");
        return 123;
    }
}
```

# 接口与抽象类
类是对象的模板，抽象类和接口可以看做是具体的类的模板。

由于从某种角度讲，接口是一种特殊的抽象类，它们的渊源颇深，有很大的相似之处，所以在选择使用谁的问题上很容易迷糊。

相同点：
- 都代表类树形结构的抽象层。在使用引用变量时，尽量使用类结构的抽象层，使方法的定义和实现分离，这样做对于代码有松散耦合的好处。
- 都不能被实例化。
- 都能包含抽象方法。抽象方法用来描述系统提供哪些功能，而不必关心具体的实现。

区别
1. 抽象类可以为部分方法提供实现，避免了在子类中重复实现这些方法，提高了代码的可重用性，这是抽象类的优势；而接口中只能包含抽象方法，不能包含任何实现。
```
public abstract class A{
    public abstract void method1();
    public void method2(){
        //A method2
    }
}
public class B extends A{
    public void method1(){
        //B method1
    }
}
public class C extends A{
    public void method1(){
        //C method1
    }
}
```

抽象类A有两个子类B、C，由于A中有方法method2的实现，子类B、C中不需要重写method2方法，我们就说A为子类提供了公共的功能，或A约束了子类的行为。method2就是代码可重用的例子。A 并没有定义 method1的实现，也就是说B、C 可以根据自己的特点实现method1方法，这又体现了松散耦合的特性。

```
public interface A{
    public void method1();
    public void method2();
}
public class B implements A{
    public void method1(){
        //B method1
    }
    public void method2(){
        //B method2
    }
}
public class C implements A{
    public void method1(){
        //C method1
    }
    public void method2(){
        //C method2
    }
}
```
接口A无法为实现类B、C提供公共的功能，也就是说A无法约束B、C的行为。B、C可以自由地发挥自己的特点现实 method1和 method2方法，接口A毫无掌控能力。

2.  一个类只能继承一个直接的父类（可能是抽象类），但一个类可以实现多个接口，这个就是接口的优势。
```
interface A{
    public void method2();
}
interface B{
    public void method1();
}
class C implements A,B{
    public void method1(){
        //C method1
    }
    public void method2(){
        //C method2
    }
}
//可以如此灵活的使用C，并且C还有机会进行扩展，实现其他接口
A a=new C();
B b=new C();
abstract class A{
    public abstract void method1();
}
abstract class B extends A{
    public abstract void method2();
}
class C extends B{
    public void method1(){
        //C method1
    }
    public void method2() {
        //C method2
    }
}
```

综上所述，接口和抽象类各有优缺点，在接口和抽象类的选择上，必须遵守这样一个原则：
- 行为模型应该总是通过接口而不是抽象类定义，所以通常是优先选用接口，尽量少用抽象类。
- 选择抽象类的时候通常是如下情况：需要定义子类的行为，又要为子类提供通用的功能。


接口和抽象类的区别是什么？
- 接口的方法默认是 public，所有方法在接口中不能有实现(Java 8 开始接口方法可以有默认实现），而抽象类可以有非抽象的方法。
- 接口中除了 static、final 变量，不能有其他变量，而抽象类中则不一定。
- 一个类可以实现多个接口，但只能实现一个抽象类。接口自己本身可以通过 extends 关键字扩展多个接口。
- 接口方法默认修饰符是 public，抽象方法可以有 public、protected 和 default 这些修饰符（抽象方法就是为了被重写所以不能使用 private 关键字修饰！）。
- 从设计层面来说，抽象是对类的抽象，是一种模板设计，而接口是对行为的抽象，是一种行为的规范。
  
总结一下 jdk7~jdk9 Java 中接口概念的变化：
- 在 jdk 7 或更早版本中，接口里面只能有常量变量和抽象方法。这些接口方法必须由选择实现接口的类实现。
- jdk8 的时候接口可以有默认方法和静态方法功能。
- Jdk 9 在接口中引入了私有方法和私有静态方法。