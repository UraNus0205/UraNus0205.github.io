## 1.数据类型

### （1）基本类型

- byte/8
- char/16
- short/16
- int/32
- float/32
- long/64
- double/64
- boolean/~

### （2）包装类型

​		基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。

```java
Integer x = 2;     // 装箱 调用了 Integer.valueOf(2)
int y = x;         // 拆箱 调用了 X.intValue()
```

### （3）缓存池

new Integer(123) 与 Integer.valueOf(123) 的区别在于：

- new Integer(123) 每次都会新建一个对象；
- Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。

```java
Integer x = new Integer(123);
Integer y = new Integer(123);
System.out.println(x == y);    // false
Integer z = Integer.valueOf(123);
Integer k = Integer.valueOf(123);
System.out.println(z == k);   // true
```

​		valueOf() 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容。

​		在 Java 8 中，Integer 缓存池的大小默认为 -128~127。

​		**编译器会在自动装箱过程调用 valueOf() 方法，因此多个值相同且值在缓存池范围内的 Integer 实例使用自动装箱来创建**，那么就会引用相同的对象。

​		基本类型对应的缓冲池如下：

- boolean values true and false
- all byte values
- short values between -128 and 127
- int values between -128 and 127
- char in the range \u0000 to \u007F

​		在使用这些基本类型对应的包装类型时，如果该数值范围在缓冲池范围内，就可以直接使用缓冲池中的对象。

## 2.String

​		String 被声明为 final，因此它不可被继承。(Integer 等包装类也不能被继承）

​		在 Java 8 中，String 内部使用 char 数组存储数据。

​		在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 `coder` 来标识使用了哪种编码。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;

    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```

​		value 数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。

### (1)不可变的好处

#### ①可以缓存hash值

​		因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

#### ②String Pool的需要

​		如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

#### ③安全性

​		String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 的那一方以为现在连接的是其它主机，而实际情况却不一定是。

#### ④线程安全

​		String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

### （2）String,StringBuffer和StringBuilder

#### ①可变性

- String 不可变
- StringBuffer 和 StringBuilder 可变

#### ②线程安全

- String 不可变，因此是线程安全的
- StringBuilder 不是线程安全的
- StringBuffer 是线程安全的，内部使用 synchronized 进行同步

### （3）String Pool

​		**字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。**不仅如此，还可以使用 String 的 intern() 方法在运行过程将字符串添加到 String Pool 中。

​		当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

​		下面示例中，s1 和 s2 采用 new String() 的方式新建了两个不同字符串，而 s3 和 s4 是通过 s1.intern() 和 s2.intern() 方法取得同一个字符串引用。intern() 首先把 "aaa" 放到 String Pool 中，然后返回这个字符串引用，因此 s3 和 s4 引用的是同一个字符串。

```java
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
String s3 = s1.intern();
String s4 = s2.intern();
System.out.println(s3 == s4);           // true
```

​		如果是采用 "bbb" 这种字面量的形式创建字符串，会自动地将字符串放入 String Pool 中。

```java
String s5 = "bbb";
String s6 = "bbb";
System.out.println(s5 == s6);  // true
```

​		在 Java 7 之前，String Pool 被放在运行时常量池中。**而在 Java 7，String Pool 被移到堆中。这是因为常量池的空间有限，在大量使用字符串的场景下会导致 OutOfMemoryError 错误。**

### （4）new String("abc")

​		使用这种方式一共会创建两个字符串对象（前提是 String Pool 中还没有 "abc" 字符串对象）。

- "**abc" 属于字符串字面量，因此编译时期会在 String Pool 中创建一个字符串对象，指向这个 "abc" 字符串字面量；**
- 而使用 new 的方式会在堆中创建一个字符串对象。

## 3.运算

### （1）参数传递

​		Java 的参数是以值传递的形式传入方法中，而不是引用传递。

​		Java总是采用值调用，也就是说方法得到的是所有参数值的一个拷贝，并不能修改传递给它的任何参数变量的内容,一个方法不可能修改一个基本数据类型的参数，而对象引用作为参数就不同了

​		以下代码中 Dog dog 的 dog 是一个指针，存储的是对象的地址。**在将一个参数传入一个方法时，本质上是将对象的地址以值的方式传递到形参中。**

```java
public class Dog {

    String name;

    Dog(String name) {
        this.name = name;
    }

    String getName() {
        return this.name;
    }

    void setName(String name) {
        this.name = name;
    }

    String getObjectAddress() {
        return super.toString();
    }
}
```

​		在方法中改变对象的字段值会改变原对象该字段值，因为引用的是同一个对象。

```java
class PassByValueExample {
    public static void main(String[] args) {
        Dog dog = new Dog("A");
        func(dog);
        System.out.println(dog.getName());// B
    }

    private static void func(Dog dog) {
        dog.setName("B");
    }
}
```

​		**但是在方法中将指针引用了其它对象，那么此时方法里和方法外的两个指针指向了不同的对象，在一个指针改变其所指向对象的内容对另一个指针所指向的对象没有影响。**

```java
public class PassByValueExample {
    public static void main(String[] args) {
        Dog dog = new Dog("A");
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        func(dog);
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        System.out.println(dog.getName());// A
    }

    private static void func(Dog dog) {
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        dog = new Dog("B");
        System.out.println(dog.getObjectAddress()); // Dog@74a14482
        System.out.println(dog.getName());// B
    }
}
```

### （2）隐式类型转换

​		Java 不能隐式执行向下转型，因为这会使得精度降低。

因为字面量 1 是 int 类型，它比 short 类型精度要高，因此不能隐式地将 int 类型向下转型为 short 类型。

```java
short s1 = 1;
// s1 = s1 + 1;
```

但是使用 += 或者 ++ 运算符会执行隐式类型转换。

```java
s1 += 1;
s1++;
```

上面的语句相当于将 s1 + 1 的计算结果进行了向下转型：

```java
s1 = (short) (s1 + 1);
```

## 4.关键字

### （1）final

#### ①数据

​		声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。

- 对于基本类型，final 使数值不变；
- **对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。**

```java
final int x = 1;
// x = 2;  // cannot assign value to final variable 'x'
final A y = new A();
y.a = 1;
```

#### ②方法

​		**声明方法不能被子类重写。**

​		private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。

#### ③类

​		声明类不允许被继承。final类中的所有方法自动地成为final方法。

### （2）static

#### ①静态成员变量

- 静态变量：又称为类变量，也就是说这个变量属于类的，**类所有的实例都共享静态变量**，可以直接通过类名来访问它。**静态变量在内存中只存在一份。**
- 实例变量：每创建一个实例就会产生一个实例变量，它与该实例同生共死。

```java
public class A {

    private int x;         // 实例变量
    private static int y;  // 静态变量

    public static void main(String[] args) {
        // int x = A.x;  // Non-static field 'x' cannot be referenced from a static context
        A a = new A();
        int x = a.x;
        int y = A.y;
    }
}
```

#### ②静态方法

​		静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。

```java
public abstract class A {
    public static void func1(){
    }
    // public abstract static void func2();  // Illegal combination of modifiers: 'abstract' and 'static'
}
```

​		**只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字，因为这两个关键字与具体对象关联。**

```java
public class A {

    private static int x;
    private int y;

    public static void func1(){
        int a = x;
        // int b = y;  // Non-static field 'y' cannot be referenced from a static context
        // int b = this.y;     // 'A.this' cannot be referenced from a static context
    }
}
```

#### ③静态语句块

静态语句块在类初始化时运行一次。

```java
public class A {
    static {
        System.out.println("123");
    }

    public static void main(String[] args) {
        A a1 = new A();
        A a2 = new A();
    }
}
输出结果：
123
```

#### ④静态内部类

​		非静态内部类依赖于外部类的实例，也就是说需要先创建外部类实例，才能用这个实例去创建非静态内部类。而静态内部类不需要。

​		静态内部类与非静态内部类之间存在一个最大的区别，我们知道非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围类，但是静态内部类却没有。

```java
public class OuterClass {

    class InnerClass {
    }

    static class StaticInnerClass {
    }

    public static void main(String[] args) {
        // InnerClass innerClass = new InnerClass(); 
        // 'OuterClass.this' cannot be referenced from a static context
        OuterClass outerClass = new OuterClass();
        InnerClass innerClass = outerClass.new InnerClass();
        StaticInnerClass staticInnerClass = new StaticInnerClass();
    }
}
```

​		**静态内部类不能访问外部类的非静态的变量和方法。**

#### ⑤静态导包

​		在使用静态变量和方法时不用再指明 ClassName，从而简化代码，但可读性大大降低。

```java
import static com.xxx.ClassName.*
```

#### ⑥初始化顺序

​		静态变量和静态语句块优先于实例变量和构造语句块，静态变量和静态语句块的初始化顺序取决于它们在代码中的顺序。

​		**静态代码块对于定义在它之后的静态变量，可以赋值，但是不能访问。**

​		存在继承的情况下，初始化顺序为：

- 父类（静态变量、静态语句块）
- 子类（静态变量、静态语句块）
- 父类（实例变量、构造语句块）
- 父类（构造函数）
- 子类（实例变量、构造语句块）
- 子类（构造函数）

#### ⑦static{}静态代码块与{}非静态代码块(构造代码块)

- 相同点： 都是**在JVM加载类时且在构造方法执行之前执行**，在类中都可以定义多个，定义多个时按定义的顺序执行，一般在代码块中对一些static变量进行赋值。
- 不同点： 静态代码块在非静态代码块之前执行(静态代码块 -> 非静态代码块 -> 构造方法)。静态代码块可能在第一次new的时候执行，但不一定只在第一次new的时候执行。比如通过 `Class.forName("ClassDemo")`创建 Class 对象的时候也会执行。非静态代码块在每new一次就执行一次。 非静态代码块可在普通方法中定义(不过作用不大)；而静态代码块不行。

​		构造代码块与构造函数的区别是： **构造代码块是给所有对象进行统一初始化**，而构造函数是给对应的对象初始化，因为构造函数是可以多个的，运行哪个构造函数就会建立什么样的对象，但无论建立哪个对象，都会先执行相同的构造代码块。也就是说，**构造代码块中定义的是不同对象共性的初始化内容。**

### （3）this

​		this关键字用于引用类的当前实例。

```java
class Manager {
    Employees[] employees;
     
    void manageEmployees() {
        int totalEmp = this.employees.length;
        System.out.println("Total employees: " + totalEmp);
        this.report();
    }
     
    void report() { }
}
```

​		在上面的示例中，this关键字用于两个地方：

- this.employees.length：访问类Manager的当前实例的变量。
- this.report（）：调用类Manager的当前实例的方法。

​		此关键字是可选的，这意味着如果上面的示例在不使用此关键字的情况下表现相同。 但是，使用此关键字可能会使代码更易读或易懂。

## 5.Object通用方法

### 概览

```java
public native int hashCode()

public boolean equals(Object obj)

protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native Class<?> getClass()

protected void finalize() throws Throwable {}

public final native void notify()

public final native void notifyAll()

public final native void wait(long timeout) throws InterruptedException

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException
```

### （1）equals()

#### ①等价与相等

- 对于基本类型，== 判断两个值是否相等，基本类型没有 equals() 方法。
- 对于引用类型，== 判断两个变量是否引用同一个对象，而 equals() 判断引用的对象是否等价。

​		一定不能使用==运算符检测两个字符串是否相等！这个运算符**只能确定两个字符串是否放置在同一 位置上，即内存地址。**

​		在子类中定义equals方法时，首先调用超类的equals。如果检测失败，对象就不可能相等。如果超类中的域都相等，就需要比较子类中的实例域。

### （2）hashCode()

​		hashCode() 返回哈希值，而 equals() 是用来判断两个对象是否等价。**等价的两个对象散列值一定相同，但是散列值相同的两个对象不一定等价**，这是因为计算哈希值具有随机性，两个值不同的对象可能计算出相同的哈希值。

​		在覆盖 equals() 方法时应当总是覆盖 hashCode() 方法，保证等价的两个对象哈希值也相等。

​		HashSet 和 HashMap 等集合类使用了 hashCode() 方法来计算对象应该存储的位置，因此要将对象添加到这些集合类中，需要让对应的类实现 hashCode() 方法。

​		下面的代码中，新建了两个等价的对象，并将它们添加到 HashSet 中。我们希望将这两个对象当成一样的，只在集合中添加一个对象。但是 EqualExample 没有实现 hashCode() 方法，因此这两个对象的哈希值是不同的，最终导致集合添加了两个等价的对象。

```java
EqualExample e1 = new EqualExample(1, 1, 1);
EqualExample e2 = new EqualExample(1, 1, 1);
System.out.println(e1.equals(e2)); // true
HashSet<EqualExample> set = new HashSet<>();
set.add(e1);
set.add(e2);
System.out.println(set.size());   // 2
```

### （3）toString()

​		默认返回 ToStringExample@4554617c 这种形式，其中@后面的数值为散列码的无符号十六进制表示。

​		绝大对数的toString方法都遵循这样的格式：类的名字，随后是一对方括号括起来的域值。

```java
public String toString()
{
    return getClass().getName()
    	+"[name="+name
    	+",salary="+salary
    	+"]";
}
```

​		随处可见toString方法的主要原因是:**只要对象与一个字符串通过操作符“+”连接起来，Java编译就会自动地调用toString方法**，以便获得这个对象的字符串描述。

​		println方法输出一个对象时会直接调用toString方法。

```java
Employee e=new Employee("uranus",10000);
String message=e+"";//自动调用e.toString()
System.out.println(e);//输出e.toString()
```

### （4）clone()

#### ①cloneable

​		clone() 是 Object 的 protected 方法，它不是 public，**一个类不显式去重写 clone()，其它类就不能直接去调用该类实例的 clone() 方法。**

​		如果一个类没有实现Cloneable接口就调用了clone() 方法，就会抛出 CloneNotSupportedException。

clone()方法并不是Cloneable接口而是Object的方法，Cloneable接口只是规定。

```java
public class CloneExample implements Cloneable {
    private int a;
    private int b;

    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

#### ②浅拷贝

​		拷贝对象和原始对象的引用类型引用同一个对象。

#### ③深拷贝

​		拷贝对象和原始对象的引用类型引用不同对象。

#### ④clone()的替代方案

​		使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，最好不要去使用 clone()，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。

## 6.继承

### （1）访问权限

1. 仅对本类可见——private。
2. 对所有类可见——public。
3. 对本包和所有子类可见———protected。
4. 对本包可见——默认，所谓默认是指没有标明任何修饰符的情况，这是一种不太受欢迎的形式。

### （2）抽象类和接口

#### ①抽象类

- 抽象类和抽象方法都使用 abstract 关键字进行声明。**如果一个类中包含抽象方法，那么这个类必须声明为抽象类。**
- *抽象类和普通类最大的区别是，抽象类不能被实例化，只能被继承。
- **抽象方法充当着占位的角色，它们的具体实现在子类中。扩展抽象类可以有两种选择。一种是**在子类中定义部分抽象方法或抽象方法也不定义**，这样就必须将子类也标记为抽象类，另一种是定义全部的抽象方法，这样一来，子类就不是抽象的了。

```java
abstract class Person
{...}
public class Student extends Person
{...}
new Persion("Uranus");//wrong
Persion P=new Student("Uranus","Economics");//ok
```

#### ②接口

- 接口的成员（字段 + 方法）默认都是 public 的，并且不允许定义为 private 或者 protected。从 Java 9 开始，允许将方法定义为 private，这样就能定义某些复用的代码又不会把方法暴露出去。
- 接口的字段默认都是 static 和 final 的。
- **接口中虽然不能包含实例域或静态方法，但可以包含常量。**
- 接口不是类，不能使用new运算符实例化一个接口，但是**可以声明接口的变量，接口变量必须引用实现了接口的类对象。**

#### ③比较

- 从使用上来看，一个类可以实现多个接口，但是不能继承多个抽象类。
- 接口的字段只能是 static 和 final 类型的，而抽象类的字段没有这种限制。
- 接口的成员只能是 public 的，而抽象类的成员可以有多种访问权限。

#### ④使用选择

​		使用接口：

- 需要让不相关的类都实现一个方法，例如不相关的类都可以实现 Comparable 接口中的 compareTo() 方法；

- 需要使用多重继承。

  使用抽象类：

- 需要在几个相关的类中共享代码。

- 需要能控制继承来的成员的访问权限，而不是都为 public。

- 需要继承非静态和非常量字段。

​		在很多情况下，接口优先于抽象类。因为接口没有抽象类严格的类层次结构要求，可以灵活地为一个类添加行为。

### （3）super

- 使用super调用构造器的语句**必须是子类构造器的第一条语句。**
- 如果子类的构造器没有显式地调用超类的构造器，则将自动地调用超类默认（没有参数)的构造器。如果超类没有不带参数的构造器，并且在子类的构造器中又没有显式地调用超类的其他构造器，则Java编译器将报告错误。
- 访问父类的成员：如果子类重写了父类的某个方法，可以通过使用 super 关键字来引用父类的方法实现。

### （4）重写与重载

#### ①重写/覆盖（Override）

​		存在于继承体系中，指子类实现了一个与父类在方法声明上完全相同的一个方法。

​		为了满足里式替换原则，重写有以下三个限制：

- 子类方法的访问权限必须大于等于父类方法；

- 子类方法的返回类型必须是父类方法返回类型或为其子类型。

- 子类方法抛出的异常类型必须是父类抛出异常类型或为其子类型。

  使用 @Override 注解，可以让编译器帮忙检查是否满足上面的三个限制条件。

  下面的示例中，SubClass 为 SuperClass 的子类，SubClass 重写了 SuperClass 的 func() 方法。其中：

- 子类方法访问权限为 public，大于父类的 protected。

- 子类的返回类型为 ArrayList<Integer>，是父类返回类型 List<Integer> 的子类。

- 子类抛出的异常类型为 Exception，是父类抛出异常 Throwable 的子类。

- 子类重写方法使用 @Override 注解，从而让编译器自动检查是否满足限制条件。

```java
class SuperClass {
    protected List<Integer> func() throws Throwable {
        return new ArrayList<>();
    }
}

class SubClass extends SuperClass {
    @Override
    public ArrayList<Integer> func() throws Exception {
        return new ArrayList<>();
    }
}
```

​		在调用一个方法时，先从本类中查找看是否有对应的方法，如果没有再到父类中查看，看是否从父类继承来。否则就要对参数进行转型，转成父类之后看是否有对应的方法。总的来说，方法调用的优先级为：

- this.func(this)
- super.func(this)
- this.func(super)
- super.func(super)

#### ②重载（Overload）

​		存在于同一个类中，指一个方法与已经存在的方法名称上相同，但是参数类型、个数、顺序至少有一个不同。

​		应该注意的是，返回值不同，其它都相同不算是重载。

### （5）多态

​		Java中对象变量是多态的，一个Employee变量既可以引用一个Employee对象，也可以引用 Employee类的任何一个子类对象。

```java
Manager boss=new Manager("Carl Cracker",80000,1987,12,15);
Employee[] staff=new Employee[3];
staff[0]=boss;
```

​		staff[0]和boss引用同一个对象，但编译器将staff[0]看成Employee对象。

### （6）静态绑定与动态绑定

#### ①静态绑定

​		在程序**执行前**方法已经被**绑定**（也就是说在**编译过程中**就已经知道这个方法到底是哪个类中的方法），此时由**编译器或其它连接程序**实现。

​		如果是private方法、static方法、final方法或者构造器，那么编译器将可以准确地知道应该调用哪个方法，我们将这种调用方式称为静态绑定。

#### ②动态绑定

​		在**运行时**根据**具体对象的类型**进行绑定。

1. 首先，虚拟机提取类的方法表。
2. 虚拟机搜索方法签名。
3. 最后，虚拟机调用方法。

## 7.反射

### （1）基本概念

​		反射 (Reflection) 是 Java 的特征之一，它允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。

​		通过反射，我们可以在运行时获得程序或程序集中每一个类型的成员和成员的信息。程序中一般的对象的类型都是在编译期就确定下来的，而 Java 反射机制可以动态地创建对象并调用其属性，这样的对象的类型在编译期是未知的。所以我们可以通过反射机制直接创建对象，即使这个对象的类型在编译期是未知的。

​		反射的核心是 JVM 在运行时才动态加载类或调用方法/访问属性，它不需要事先（写代码的时候或编译期）知道运行对象是谁。

​		当我们在使用 IDE(如 Eclipse，IDEA)时，当我们输入一个对象或类并想调用它的属性或方法时，一按点号，编译器就会自动列出它的属性或方法，这里就会用到反射。

​		**Java 中的一大利器——注解的实现就用到了反射。**

​		Class 和 java.lang.reflect 一起对反射提供了支持，java.lang.reflect 类库主要包含了以下三个类：

- **Field** ：可以使用 get() 和 set() 方法读取和修改 Field 对象关联的字段；
- **Method** ：可以使用 invoke() 方法调用与 Method 对象关联的方法；
- **Constructor** ：可以用 Constructor 的 newInstance() 创建新的对象。

### （2）基本运用

#### ①获得class对象

​		类Test在com.company这个包下。

- 使用Class类的`forName`静态方法：

```java
Class.forName("com.company.Test");//class com.company.Test
```

- 直接获取某一个类的 class

```java
Test.class;//class com.company.Test
```

- 调用某个对象的 `getClass()` 方法

```java
Test test=new Test();
test.getClass();//class com.company.Test
```

#### ②创建实例

- 使用Class对象的newInstance()方法来创建Class对象对应类的实例。

```java
Class<?> t=Test.class;
Object obj = t.newInstance();
```

- 先通过Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance()方法来创建实例。这种方法可以用指定的构造器构造类的实例。

```java
Class<?> t=Test.class;
Constructor constructor = t.getConstructor(Test.class);
Object obj = constructor.newInstance();
```

#### ③获取方法

- `getDeclaredMethods` 方法返回类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法。
- `getMethods` 方法返回某个类的所有公用（public）方法，包括其继承类的公用方法。
- `getMethod` 方法返回一个特定的方法，其中第一个参数为方法名称，后面的参数为方法的参数对应Class的对象。

#### ④获取类成员变量

- `getFiled`：访问公有的成员变量
- `getDeclaredField`：所有已声明的成员变量，但不能得到其父类的成员变量
- `getFileds` 和 `getDeclaredFields` 方法用法同上

#### ⑤调用方法

​		当我们从类中获取了一个方法后，我们就可以用 `invoke()` 方法来调用这个方法。

```java
 Class<?> klass = methodClass.class;
//创建methodClass的实例
Object obj = klass.newInstance();
//获取methodClass类的add方法
Method method = klass.getMethod("add",int.class,int.class);
//调用method对应的方法 => add(1,4)
Object result = method.invoke(obj,1,4);
```

### （3）注意事项

​		由于反射会额外消耗一定的系统资源，因此如果不需要动态地创建一个对象，那么就不需要用反射。

​		另外，反射调用方法时可以忽略权限检查，因此可能会破坏封装性而导致安全问题。

## 8.异常

### （1）异常的体系结构

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/2020-12/Java异常类层次结构图.png)

![img](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/690102-20160728164909622-1770558953.png)

​		异常类Exception可以分为运行时异常（RuntimeException）和非运行时异常。

​		**Java异常又可以分为不受检查异常（Unchecked Exception）和检查异常（Checked Exception）。**

- Error：大多数由JVM生成并抛出，大多数错误与代码执行者所执行的操作无关，属于程序无法处理的错误，无法通过catch进行捕获。
- Exception：程序本身可以处理的异常。

- 检查异常：在正确的程序运行过程中，很容易出现的、情理可容的异常状况，在一定程度上这种异常的发生是可以预测的，并且一旦发生该种异常，就必须采取某种方式进行处理。
- 不受检查异常：**包括`RuntimeException`及其子类和`Error`**。

> **除了`RuntimeException`及其子类以外，其他的`Exception`类及其子类都属于检查异常**，当程序中可能出现这类异常，**要么使用`try-catch`语句进行捕获，要么用`throws`子句抛出**，否则编译无法通过。

### （2）Java异常的处理机制

​		Java的异常处理本质上是抛出异常和捕获异常。

- 抛出异常throw new Exception()： 即发生了异常之后会把这个异常丢掉,不会在继续执行接下来的语句。
- 捕获异常trt{}catch(Exception e){} :发生异常之后,catch会捕获到这个异常并存于e变量中,然后再在catch语句块中进行逻辑处理,会继续执行.

> 对于`运行时异常`、`错误`和`检查异常`，Java技术所要求的异常处理方式有所不同。
>
> 由于运行时异常及其子类的不可查性，为了更合理、更容易地实现应用程序，Java规定，**运行时异常将由Java运行时系统自动抛出，允许应用程序忽略运行时异常**。
>
> 对于方法运行中可能出现的`Error`，当运行方法不欲捕捉时，Java允许该方法不做任何抛出声明。因为，大多数`Error`异常属于永远不能被允许发生的状况，也属于合理的应用程序不该捕捉的异常。
>
> 对于所有的检查异常，Java规定：一个方法必须捕捉，或者声明抛出方法之外。也就是说，当一个方法选择不捕捉检查异常时，它必须声明将抛出异常。

​		Java异常处理涉及到五个关键字，分别是：`try`、`catch`、`finally`、`throw`、`throws`。

　　• **try**     -- 用于监听。将要被监听的代码(可能抛出异常的代码)放在try语句块之内，当try语句块内发生异常时，异常就被抛出。
　　• **catch**  -- 用于捕获异常。catch用来捕获try语句块中发生的异常。
　　• **finally** -- **finally语句块总是会被执行**。它主要用于**回收在try块里打开的物力资源**(如数据库连接、网络连接和磁盘文件)。**只有finally块执行完成之后，才会回来执行try或者catch块中的return或者throw语句，如果finally中使用了return或者throw等终止方法的语句，则就不会跳回执行，直接停止。**
　　• **throw**  -- 用于抛出异常。
　　• **throws** -- 用在方法签名中，用于声明该方法可能抛出的异常。

​		在以下三种特殊情况下，finally块不会被执行：

1. 在 `try` 或 `finally`块中用了 `System.exit(int)`退出程序。但是，**如果 `System.exit(int)` 在异常语句之后，`finally` 还是会被执行**
2. 程序所在的线程死亡。
3. 关闭 CPU。

### （3）try-with-resources

1. **适用范围（资源的定义）：** 任何实现 `java.lang.AutoCloseable`或者 `java.io.Closeable` 的对象
2. **关闭资源和 finally 块的执行顺序：** 在 `try-with-resources` 语句中，**任何 catch 或 finally 块在声明的资源关闭后运行。**

​		面对必须要关闭的资源，我们总是应该优先使用 `try-with-resources` 而不是`try-finally`。随之产生的代码更简短，更清晰，产生的异常对我们也更有用。`try-with-resources`语句让我们更容易编写必须要关闭的资源的代码，若采用`try-finally`则几乎做不到这点。

```java
//读取文本文件的内容
Scanner scanner = null;
try {
	scanner = new Scanner(new File("D://read.txt"));
	while (scanner.hasNext()) {
		System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException e) {
   e.printStackTrace();
} finally {
    if (scanner != null) {
        scanner.close();
    }
}
```

​		使用 Java 7 之后的 `try-with-resources` 语句改造上面的代码:

```java
try (Scanner scanner = new Scanner(new File("test.txt"))) {
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException fnfe) {
    fnfe.printStackTrace();
}
```

​		通过使用分号分隔，可以在`try-with-resources`块中声明多个资源。

## 9.泛型

​		泛型程序设计意味着编写的代码可以被很多不同类型的对象所重用。

### （1）泛型的使用

#### ①泛型类

```java
public class Generic<T>{ 
    //key这个成员变量的类型为T,T的类型由外部指定  
    private T key;

    public Generic(T key) { //泛型构造方法形参key的类型也为T，T的类型由外部指定
        this.key = key;
    }

    public T getKey(){ //泛型方法getKey的返回值类型为T，T的类型由外部指定
        return key;
    }
}
```

- 泛型的类型参数只能是类类型，不能是简单类型。
- 不能对确切的泛型类型使用instanceof操作。如下面的操作是非法的，编译时会出错。

```java
if(ex_num instanceof Generic<Number>)
{   

} 
```

#### ②泛型接口

```java
//定义一个泛型接口
public interface Generator<T> {
    public T next();
}
```

#### ③泛型方法

​		**泛型类，是在实例化类的时候指明泛型的具体类型；泛型方法，是在调用方法的时候指明泛型的具体类型** 。

​		并不是说泛型类中的方法就是泛型方法。泛型方法可以定义在普通类中，也可以定义在泛型类中。

```java
public class GenericTest {
   //这个类是个泛型类，在上面已经介绍过
   public class Generic<T>{     
        private T key;

        public Generic(T key) {
            this.key = key;
        }

        //虽然在方法中使用了泛型，但是这并不是一个泛型方法。
        //这只是类中一个普通的成员方法，只不过他的返回值是在声明泛型类已经声明过的泛型。
        //所以在这个方法中才可以继续使用T这个泛型。
        public T getKey(){
            return key;
        }
    }

    /** 
     * 这才是一个真正的泛型方法。
     * 首先在public与返回值之间的<T>必不可少，这表明这是一个泛型方法，并且声明了一个泛型T
     * 泛型的数量也可以为任意多个 
     *    如：public <T,K> K showKeyName(Generic<T> container){
     *        }
     */
    public <T> T showKeyName(Generic<T> container){
        System.out.println("container key :" + container.getKey());
        T test = container.getKey();
        return test;
    }

    //这也不是一个泛型方法，这就是一个普通的方法，只是使用了Generic<Number>这个泛型类做形参而已。
    public void showKeyValue1(Generic<Number> obj){
        Log.d("泛型测试","key value is " + obj.getKey());
    }

    //这也不是一个泛型方法，这也是一个普通的方法，只不过使用了泛型通配符?
    //同时这也印证了泛型通配符章节所描述的，?是一种类型实参，可以看做为Number等所有类的父类
    public void showKeyValue2(Generic<?> obj){
        Log.d("泛型测试","key value is " + obj.getKey());
    }
}
```

​		若想在类中的静态方法使用泛型，则必须将静态方法也定义成泛型方法。**静态方法无法访问类上定义的泛型；如果静态方法操作的引用数据类型不确定的时候，必须要将泛型定义在方法上。**

```java
public class StaticGenerator<T> {
    ....
    ....
    /**
     * 如果在类中定义使用泛型的静态方法，需要添加额外的泛型声明（将这个方法定义成泛型方法）
     * 即使静态方法要使用泛型类中已经声明过的泛型也不可以。
     * 如：public static void show(T t){..},此时编译器会提示错误信息：
          "StaticGenerator cannot be refrenced from static context"
     */
    public static <T> void show(T t){

    }
}
```

### （2）类型变量的限定

​		在使用泛型的时候，我们还可以为传入的泛型类型实参进行上下边界的限制，如：类型实参只准传入某种类型的父类或某种类型的子类。

​		为泛型添加上边界，即传入的类型实参必须是指定类型的子类型

```java
public static <T extends Comparable> T min(Test<T> a)
//public static <T> T min(Test<T extends Comparable> a)是错误写法
```

​		**选择关键字extends而不是implements的原因是更接近子类的概念**，并且Java的设计者也不打算在 语言中再添加一个新的关键字（如sub)。

​		一个类型变量或通配符可以有多个限定，例如：

```java
T extends Comparable & Serializable
```

在Java的继承中，可以根据需要**拥有多个接口超类型，但限定中至多有一个类。如果用一个类作为 限定，它必须是限定列表中的第一个。**

### （3）通配符

- 对于类型限定：

```java
? extends T getFirst()
void setFirst(? extends T)
```

​		上界<? extends T>不能往里存，只能往外取。编译器只知道传入的是T的子类，但具体是哪一个编译器不知道，他只标注了一个占位符，当？传过来时，他不知道这能不能和占位符匹配，所以set不了。

- 对于通配符的超类型限定

​		通配符限定与类型变量限定十分类似，但是，还有一个附加的能力，即可以指定一个超类型限定， 如下所示: ? super T .这个通配符限制为T的所有超类型。

```java
void setFirst(? super T)
? super T getFirst()
```

​		下界<? super T>不影响往里存，但往外取只能放在Object对象里。下边界已经限制了？的粒度，他只可能是T本身或者是T的父类。我们想想，我想要一个T，你却返回给我一个比T小的Object，这样我们就因为精度损失而拿不到想要的数据了。

​		直观地讲，**带有超类型限定的通配符可以向泛型对象写入，带有子类型限定的通配符可以从泛型对 象读取。**

### （4）泛型擦除

​		无论何时定义一个泛型类型，都自动提供了一个相应的原始类型(raw type)。原始类型的名字就是删去类型参数后的泛型类型名。擦除(erased)类型变量，并替换为限定类型（无限定的变量用Object)。

```java
public static <T extends Comparable> T min(T[] a)
public static Comparable min(Comparable[] a)//擦除类型后
```

​		方法的擦除带来了复杂问题：

```java
class DateInterval extends Pair<Date>
{
    public void setSecond(Date second)
        //这里的Date不用变，因为是继承Pair类而不是本身是Pair类
    {
        if(second.compareTo(getFirst())>=0)
            super.setSecond(second);
    }   
    ...
}
```

​		一个日期区间是一对Date对象，并且需要覆盖这个方法来确保第二个值永远不小于第一个值。这个类擦除后变成

```java
class DateInterval extends Pair
{
    public void setSecond(Date second)
    {
        ...
    }   
    ...
}
```

​			而此时存在另一个从Pair继承的setSecond方法，即：

```java
public void setSecond(Object second)
```

​		这显然是一个不同的方法，因为它有一个不同类型的参数——Object，而不是Date。然而，不应该不一样。考虑下面的语句序列:

```java
DateInterval interval=new DateInterval(...);
Pair<Date> pair=interval;
pair.setSecond(aDate);
```

​		这里，希望对setSecond的调用具有多态性，并调用最合适的那个方法。由于pair引用DateInterval对象，所以应该调用DateInterval.setSecond。问题在于类型擦除与多态发生了冲突。要解决这个问题，就需要编译器**在DateInterval类中生成一个桥方法** （bridge method):

```java
public void setSecond(Object second)
{
	setSecond((Date)second);
}
```

​		变量pair已经声明为类型Pair<Date>，并且这个类型只有一个简单的方法叫setSecond，即setSecond(Object)。虚拟机用pair引用的对象调用这个方法。这个对象是DateInterval类型的,因而将会调用DateInterval.setSecond(Object)方法。这个方法是合成的桥方法。它调用DatcInterval.setSecond(Date)，这正是我们所期望的操作效果。

假设DateInterval方法也覆盖了gerSecond方法：

```java
public Date getSecond()
{
    return (Date)super.getSecond().clone();
}
```

​		擦除的类型中，有两个getSecond方法：

```java
Date getSecond();//DateInterval
Object getSecond();//Pair
```

​		不能这样编写Java代码，具有相同参数类型的两个方法是不合法的。但是，**在虚拟机中，用参数类型和返回类型确定一个方法**。因此，编译器可能产生两个仅返回类型不同的方法字节码，虚拟机能够正确地处理这一情况。

- 虚拟机中没有泛型，只有普通的类和方法
- 所有的类型参数都用它们的限定类型替换
- 桥方法被合成来保持多态
- 为保持类型安全性，必要时插入强制类型转换

## 10.枚举

​		枚举在很多时候会和常量拿来对比，可能因为本身我们大量实际使用枚举的地方就是为了替代常量。那么这种方式由什么优势呢？

​		**以这种方式定义的常量使代码更具可读性，允许进行编译时检查，预先记录可接受值的列表，并避免由于传入无效值而引起的意外行为。**

```java
public enum ProductStatusEnum{
    ON_SALE(1,"在线"),
    NO_SALE(2,"下线");
    private String value;
    private int code;
    ProductStatusEnum(int code,String value){
    	this.code = code;
        this.value = value;
    }

    public String getValue() {
        return value;
    }

    public int getCode() {
        return code;
    }
}
```

​		在比较两个枚举类型的值时，直接使用“==”就可以了。

​		所有的枚举类型都是Enum类的子类。它们继承了这个类的许多方法。其中最有用的一个是 toString，这个方法能够返回枚举常量名。

```java
Test test=new Test();
Test.ProductStatusEnum productStatusEnum=ProductStatusEnum.ON_SALE;
System.out.println(productStatusEnum.toString());//ON_SALE
```

​		toString的逆方法是静态方法valueOf。

```java
Test test=new Test();
Test.ProductStatusEnum productStatusEnum;
productStatusEnum=ProductStatusEnum.valueOf(ProductStatusEnum.class,"NO_SALE");
System.out.println(productStatusEnum.toString());//NO_SALE
```

​		每个枚举类型都有一个静态的values方法，它将返回一个包含全部枚举值的数组。

```java
Test test=new Test();
ProductStatusEnum[] values = Test.ProductStatusEnum.values();
for(ProductStatusEnum p:values)
{
	System.out.println(p.toString());
}
//ON_SALE
//NO_SALE
```

