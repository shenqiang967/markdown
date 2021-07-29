# Java 修饰符

Java语言提供了很多修饰符，主要分为以下两类：

- 访问修饰符
- 非访问修饰符

## 访问控制修饰符

Java中，可以使用访问控制符来保护对类、变量、方法和构造方法的访问。Java 支持 4 种不同的访问权限。

- **default** (即默认，什么也不写）: 在同一包内可见，不使用任何修饰符。使用对象：类、接口、变量、方法。
- **private** : 在同一类内可见。使用对象：变量、方法。 **注意：不能修饰类（外部类）**
- **public** : 对所有类可见。使用对象：类、接口、变量、方法
- **protected** : 对同一包内的类和所有子类可见。使用对象：变量、方法。 **注意：不能修饰类（外部类）**。

我们可以通过以下表来说明访问权限：

| 修饰符      | 当前类 | 同一包内 | 子孙类(同一包) | 子孙类(不同包) | 其他包 |
| :---------- | :----- | :------- | :------------- | :------------- | :----- |
| `public`    | Y      | Y        | Y              | Y              | Y      |
| `protected` | Y      | Y        | Y              | Y/N            | N      |
| `default`   | Y      | Y        | Y              | N              | N      |
| `private`   | Y      | N        | N              | N              | N      |

### 默认访问修饰符-不使用任何关键字

使用默认访问修饰符声明的变量和方法，对同一个包内的类是可见的。接口里的变量都隐式声明为 **public static final**,而接口里的方法默认情况下访问权限为 **public**。

如下例所示，变量和方法的声明可以不使用任何修饰符。

```java
String version = "1.5.1";
boolean processOrder() {
   return true;
}
```

### 私有访问修饰符-private

私有访问修饰符是最严格的访问级别，所以被声明为 **private** 的方法、变量和构造方法只能被所属类访问，并且类和接口不能声明为 **private**。

声明为私有访问类型的变量只能通过类中公共的 getter 方法被外部类访问。

Private 访问修饰符的使用主要用来隐藏类的实现细节和保护类的数据。

下面的类使用了私有访问修饰符：

```java
public class Logger {
   private String format;
   public String getFormat() {
      return this.format;
   }
   public void setFormat(String format) {
      this.format = format;
   }
}
```

实例中，Logger 类中的 format 变量为私有变量，所以其他类不能直接得到和设置该变量的值。为了使其他类能够操作该变量，定义了两个 public 方法：getFormat() （返回 format的值）和 setFormat(String)（设置 format 的值）

### 公有访问修饰符-public

被声明为 public 的类、方法、构造方法和接口能够被任何其他类访问。

如果几个相互访问的 public 类分布在不同的包中，则需要导入相应 public 类所在的包。由于类的继承性，类所有的公有方法和变量都能被其子类继承。

Java 程序的 main() 方法必须设置成公有的，否则，Java 解释器将不能运行该类。

### 受保护的访问修饰符-protected

protected 需要从以下两个点来分析说明：

- **子类与基类在同一包中**：被声明为 protected 的变量、方法和构造器能被同一个包中的任何其他类访问；
- **子类与基类不在同一包中**：那么在子类中，子类实例可以访问其从基类继承而来的 protected 方法，而不能访问基类实例的protected方法。

> 这里用代码解释一下"类实例可以访问其从基类继承而来的 protected 方法，而不能访问基类实例的protected方法。"

```java
public class ProtectedFathorSamePackage {
    protected void say(){
        System.out.println("ProtectedFathorSamePackage说。。。");
    }
}
public class ProtectedSonSamePackage extends ProtectedFathorSamePackage{
    @Override
    protected void say(){
        //同包，基类实例的protected方法访问，可以访问
        new ProtectedFathorSamePackage().say();
        //基类继承而来的 protected 方法，可以访问
        super.say();
    }
}
public class ProtectedSonDiffPackage extends ProtectedFathorSamePackage {
    public void say(){
        //不同包，基类实例的protected方法访问，不能访问
        //new ProtectedFathorSamePackage().say();
        //基类继承而来的 protected 方法，可以访问
        super.say();
    }
}
```

![image-20210727153330906](C:\Users\18055\Desktop\笔记\java基础\java修饰符.assets\image-20210727153330906-16273712122474.png)

> protected 可以修饰数据成员，构造方法，方法成员，**不能修饰类（内部类除外）**。
>
> 如果我们只想让该方法对其所在类的子类可见，则将该方法声明为 protected。

### 访问控制和继承

请注意以下方法继承的规则：

- 父类中声明为 public 的方法在子类中也必须为 public。
- 父类中声明为 protected 的方法在子类中要么声明为 protected，要么声明为 public，不能声明为 private。
- 父类中声明为 private 的方法，不能够被继承。

![image-20210727152330380](C:\Users\18055\Desktop\笔记\java基础\java修饰符.assets\image-20210727152330380-16273706124873.png)

![image-20210727152316927](C:\Users\18055\Desktop\笔记\java基础\java修饰符.assets\image-20210727152316927-16273705983662.png)

> 记住一点，继承来的访问控制权限范围只能大不能小

---

## 非访问修饰符

为了实现一些其他的功能，Java 也提供了许多非访问修饰符。

static 修饰符，用来修饰类方法和类变量。

final 修饰符，用来修饰类、方法和变量，final 修饰的类不能够被继承，修饰的方法不能被继承类重新定义，修饰的变量为常量，是不可修改的。

abstract 修饰符，用来创建抽象类和抽象方法。

synchronized 和 volatile 修饰符，主要用于线程的编程。

为了实现一些其他的功能，Java 也提供了许多非访问修饰符。

static 修饰符，用来修饰类方法和类变量。

final 修饰符，用来修饰类、方法和变量，final 修饰的类不能够被继承，修饰的方法不能被继承类重新定义，修饰的变量为常量，是不可修改的。

abstract 修饰符，用来创建抽象类和抽象方法。

synchronized 和 volatile 修饰符，主要用于线程的编程。

### static 修饰符

- **静态变量：**

  static 关键字用来声明独立于对象的静态变量，无论一个类实例化多少对象，它的静态变量只有一份拷贝。 静态变量也被称为类变量。局部变量不能被声明为 static 变量。

- **静态方法：**

  static 关键字用来声明独立于对象的静态方法。静态方法不能使用类的非静态变量。静态方法从参数列表得到数据，然后计算这些数据。

对于静态变量、静态初始化块、变量、初始化块、构造器，它们的初始化顺序依次是（静态变量、静态初始化块）>（变量、初始化块）>构造器。

```java
public class StaticClassTest {
    public static void main(String[] args) {
        Son son = new Son();
    }
}
/**
 * 顶层父类
 * */
class GrandFathor{
    private static String type = "grandFathor";
    static {
        System.out.println("静态代码块"+GrandFathor.type);
    }
    {
        System.out.println("grandFathor代码块");
    }
}

/**
 * 父类继承自顶层父类，
 * */
class Fathor extends GrandFathor{
    private static String type = "fathor";
    static {
        System.out.println("静态代码块"+Fathor.type);
    }
    {
        System.out.println("fathor代码块");
    }

}
/**
 * 子类 继承自 父类
 * */
class Son extends Fathor{
    private static String type = "son";
    static {
        System.out.println("静态代码块"+Son.type);
    }
    {
        System.out.println("Son代码块");
    }
}

```

执行main方法结果为：

静态代码块grandFathor
静态代码块fathor
静态代码块son
grandFathor代码块
fathor代码块
Son代码块

关于静态变量初始化时机：

```java
private static int num = 11;
private static int num2;
private static final int num3 = 12
```

在类的准备阶段 num,num2都是0；num3是12（此时num3被final修饰，类字段的属性就存在ConstantValue中，而准备阶段变量num3就会被初始化为ConstantValue属性所指定的值），在类得初始化阶段由于num2没有赋值，默认赋值为0，而num则被赋值为11

> 所有静态成员在类加载完成之后都已经或显示或隐式的完成了初始化赋值的操作。

### final 修饰符

**final 变量：**

final 表示"最后的、最终的"含义，变量一旦赋值后，不能被重新赋值。被 final 修饰的实例变量必须显式指定初始值。

final 修饰符通常和 static 修饰符一起使用来创建类常量。

> final修饰的字段在运行时被初始化，可以直接赋值，也可以在实例构造器中赋值，赋值后不可修改

```java
public class FinalClassTest{
    public static void main(String[] args) {
        Son son = new Son();
        son.saySon();
        son.say();
    }
}

class FinalField{
    final int value = 10;
    // 下面是声明常量的实例
    public static final int BOXWIDTH = 6;
    static final String TITLE = "Manager";

    public void changeValue(){
        //value = 12; //将输出一个错误
    }
}
final class FinalFathor{

}
class FinalMethodFathor{
    final void say(){
        System.out.println("FinalMethodFathor");
    }
}
/**
 * 编译时报错，因为final类不能继承
 * */
//class Son extends FinalFathor{}

class Son extends FinalMethodFathor{
    /**
     * 编译时报错，不能重写或覆盖父类的final方法
     * */
//    public void say(){}
    /**
     * 可以继承自父类的final方法
     * */
    void saySon(){
        super.say();
    }
}
```

### abstract 修饰符

**抽象类：**

抽象类不能用来实例化对象，声明抽象类的唯一目的是为了将来对该类进行扩充。

一个类不能同时被 abstract 和 final 修饰。如果一个类包含抽象方法，那么该类一定要声明为抽象类，否则将出现编译错误。

抽象类可以包含抽象方法和非抽象方法。

```java
public class AbstractClassTest {
    public static void main(String[] args) {
        new Son().sayP();
        new Son().say();
    }
}
class DefaultClassAbstractMethod{
    /**
     * 编译时报错，因为只有抽象类和接口才可以定义抽象方法，反之有抽象方法的类必然是抽象类
     * */
//    abstract void say(){}
}

interface InterfaceAbstractMethod{
    abstract void say();
}

abstract class AbstractClassAbstractMethod{
    abstract void say();
    void sayP(){
        System.out.println("AbstractClassAbstractMethod");
    }
}

class Son extends AbstractClassAbstractMethod{
    /**
     * 继承自抽象类的非抽象子类必须实现其抽象方法/或者抽象子类不实现，继承其父类方法等待其下一个子类实现
     * */
    @Override
    void say() {
        sayP();
    }
}
abstract class AbstractSon extends AbstractClassAbstractMethod{
    /**
     * 等待继承自AbstractSon的子类来实现该父类以及父类的父类里所有的抽象方法
     * */
}
```

### synchronized 修饰符

synchronized 关键字声明的方法同一时间只能被一个线程访问。synchronized 修饰符可以应用于四个访问修饰符。

synchronized锁的三种形式

- 普通方法：锁是当前对象实例(非static变量)
- 静态方法：锁是当前类的Class对象(static变量)
- 同步方法块：锁是synchronized括号里的对象

### volatile 修饰符

volatile 修饰的成员变量在每次被线程访问时，都强制从共享内存中重新读取该成员变量的值。而且，当成员变量发生变化时，会强制线程将变化值回写到共享内存。这样在任何时刻，两个不同的线程总是看到某个成员变量的同一个值。

一个 volatile 对象引用可能是 null。

```java
public class MyRunnable implements Runnable
{
    private volatile boolean active;
    public void run()
    {
        active = true;
        while (active) // 第一行
        {
            // 代码
        }
    }
    public void stop()
    {
        active = false; // 第二行
    }
}
```

通常情况下，在一个线程调用 run() 方法（在 Runnable 开启的线程），在另一个线程调用 stop() 方法。 如果 ***第一行*** 中缓冲区的 active 值被使用，那么在 ***第二行*** 的 active 值为 false 时循环不会停止。

但是以上代码中我们使用了 volatile 修饰 active，所以该循环会停止。

