# 一、Java基础的程序设计结构

## 1.数据类型

- byte/8
- char/16
- short/16
- int/32
- float/32（有一个后缀F，例如3.1415F）
- long/64（例如3.1415）
- double/64
- boolean/~（true or false，且Java中整型值和布尔值之间不能进行相互转换）。

## 2.变量

### （1）变量初始化

​		C和C++区分变量的声明与定义：

```C++
int i=10;//定义
extern int i;//声明
```

​		在Java中不区分变量的声明和定义。

### （2）常量

​		在Java中，利用final声明变量：

```java
public class Constants
{
	public static void main(String[] args)
    {
    	final double A=5;    
    }
}
```

​		在Java中，经常希望某个常量可以在一个类中的多个方法中使用，通常将这些常量称为类常量。可以使用关键字static final设置一个类常量。

## 3.运算符

### （1）数值类型之间的转换

![image-20210409195226188](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210409195226188.png)

​		实心箭头代表无信息丢失；虚箭头表示可能有精度损失。

### （2）强制类型转换

```java
double x=9.997;
int nx=(int)x;//nx=9
```

​		强制类型转换通过截断小数部分将浮点值转换为整型。

## 4.字符串

### （1）子串

​		String类的substring方法可以从一个较大的字符串提取出一个子串：

```java
String greeting="Hello";
String s=greeting.substring(0,3);//s="Hel"
```

### （2）拼接

​		Java允许使用+号连接两个字符串

```java
String a="hello";
String b="world";
a=a+b;//helloworld
```

​		当将一个字符串与一个非字符串的值拼接时，后者被转换为字符串：

```java
String a="hello";
int i=12;
String b=a+i;//hello12
```

### （3）不可变字符串

​		如果希望将greeting的内容修改为“Help！”，不能直接地将greeting的最后两个位置的字符修改为“p!”，正确做法如下：

```java
greeting=greeting.substring(0,3)+"p!";
```

### （4）检测字符串是否相等

​		可以使用equals方法检测两个字符串是否相等：`s.equals(t)`。

​		s与t既可以是变量，也可以是常量。

​		若想要不区分大小写，可以使用equalsIgnoreCase方法。

```java
"Hello".equals(greeting);
"Hello".equalsIgnoreCase(greeting);
```

​		一定不能使用==运算符检测两个字符串是否相等！这个运算符只能确定两个字符串是否放置在同一位置上，即内存地址。

### （5）构建字符串

​		有些时候，需要由较短的字符串构建字符串，例如，按键或来自文件中的单词。采用字符串连接的方式达到此目的效率比较低。每次连接字符串，都会构建一个新的String对象，既耗时，又浪费空间。**使用StringBuilder类就可以避免这个问题的发生。**
​		如果需要用许多小段的字符串构建一个字符串，那么应该按照下列步骤进行。首先，构建一个空的字符串构建器:

```java
StringBuilder builder = new StringBuilder();
//需要添加内容时就调用append：
builder.appen(ch);
builder.appen(str);
```

​		在需要构建字符串时就调用toString方法，将可以得到一个String对象，其中包含了构建器中的字符序列。

```java
String completedString = builder.toString();
```

## 5.输入输出

### （1）读取输入

```java
Scanner in=new Scanner(System.in);
String name=in.nextLine();//读取一整行内容
String firstName=in.next();//读取一个单词（以空白符作为分隔符）
int age=in.nextInt();
System.out.println(age+name);
```

### （2）格式化输出

```java
System.out.printf("%d",x);
```

## 6.大数值

​		java.math包中有两个大数值的类：BigInteger和BigDecimal。

​		但是不能用算术运算符处理大数值，需要用到add、subtract、multiply和divide方法。

```java
BigInteger a=BigInteger.valueOf(100);
BigInteger a=BigInteger.valueOf(1000);
BigInteger c=a.add(b);
BigInteger d=c.multiply(b.add(BigInteger.valueOf(2)));//d=c*(b+2)
```

## 7.数组

```java
int[] a;//只做声明，没有初始化
int[] a=new int[100];
int a[]=new int[100];//这种形式也可以
```

### （1）for each循环

```java
for(int i=0;i<a.length;i++)
	System.out.println(a[i]);
//与下面这种写法效果一致，不必为下标的起始值和终止值而操心
for(int e:a)
	System.out.println(e);	
```

​		有个更加简单的方式打印数组中的所有值，即Arrays类的toString方法。

```java
int[] smallPrimes={2,3,4,5,6};
System.out.println(Arrays.toString(smallPrimes));
//[2, 3, 4, 5, 6]
```

### （2）数组初始化以及匿名函数

```java
int[] smallPrimes={2,3,4,5,6};
```

​		使用这种语句时，不需要调用new。

​		甚至还可以初始化一个匿名数组：

```java
new int[]{4,5,6,7,8};
smallPrimes=new int[]{4,5,6,7,8};//在不创建新变量的情况下重新初始化一个数组
```

​		PS：在Java中，允许数组长度为0：`new int[0]`

### （3）数组拷贝

​		Java中允许将一个数组变量拷贝给另一个数组变量，这时两个变量将引用同一个数组：

```java
int[] luckyNumbers=smallPrimes;
luckyNumbers[2]=100;//smallPrimes[2]此时也等于100
```

![image-20210409204846541](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210409204846541.png)

​		如果希望将一个数组的所有的值拷贝到一个新的数组（不是数组变量）中去，就要使用Arrays类的copyOf方法：

```java
int[] copiedLuckyNumbers=Arrays.copyOf(luckyNumbers,2*luckyNumbers.length);
```

​		第二个参数是新数组的长度，这个方法可用来扩容。

​		若数组元素是数值型，多余的元素将被初始化为0；若是布尔型，则初始化为false。

# 二、对象与类

## 1.一个Employee类

```java
public class EmployeeTest {
    public static void main(String[] args) {
        Employee[] staff = new Employee[3];
    }
}
class Employee
{
    private String name;
    private double salary;
    private Date hireDay;
    
    public Employee(String n,double s,int year,int month,int day)
    {
        name=n;
        salary=s;
        GregorianCalendar calendar = new GregorianCalendar(year,month-1,day);
        hireDay=calendar.getTime();
    }
    
    public String getName()
    {
        return name;
    }
    public double getSalary()
    {
        return salary;
    }
    public Date getHireDay()
    {
        return hireDay;
    }
    
    public void raiseSalary(double byPercent)
    {
        double raise = salary*byPercent/100;
        salary+=raise;
    }
}
```

​		注意，在这个示例程序中包含两个类:一个Employee类，一个带有public访问修饰符的EmployeeTest类。EmployeeTest类包含了main方法，其中使用了前面介绍的指令。

​		源文件名是EmployeeTest.java，这是因为文件名必须与public类的名字相匹配。在一个源文件中，只能有一个公有类，但可以有任意数目的非公有类。

​		接下来，当编译这段源代码的时候，编译器将在目录下创建两个类文件:EmployeeTest.class和Employee.class。

​		将程序中包含main方法的类名字提供给字节码解释器，以便启动这个程序:

​		`java EmployeeTest`

### （1）从构造器开始

​		先看看Employee的构造器：

```java
 public Employee(String n,double s,int year,int month,int day)
    {
        name=n;
        salary=s;
        GregorianCalendar calendar = new GregorianCalendar(year,month-1,day);
        hireDay=calendar.getTime();
    }
```

​		构造器总是伴随着new操作符的执行被调用，而不能对一个已经存在的对象调用构造器来达到重新设置实例域的目的。

```java
Employee James=new Employee("James Bond",100000,1950,1,1);
James.Employee("James Bond",20000,1950,1,1);//error
```

### （2）隐式参数与显式参数

```java
public void raiseSalary(double byPercent)
{
    double raise = salary*byPercent/100;
    salary+=raise;
}
James.raiseSalary(5);
```

​		`raiseSalary`方法有两个参数：

- 隐式参数：是出现在方法名之前的Employee对象；
- 显式参数：位于方法名后面括号中的数值

​		在每一个方法中，关键字this表示隐式参数，若有需要，可以用下列方式编写`raiseSalary`方法：

```java
public void raiseSalary(double byPercent)
{
    double raise = this.salary*byPercent/100;
    this.salary+=raise;
}
```

### （3）基于类的访问权限

​		**一个方法可以访问所属类的所有对象的私有数据。**

```java
class Employee
{
    ...
    boolean equals(Employee other)
    {
        return name.equals(other.name);
    }
}
Employee harry;
Employee boss;
if(harry.equals(boss))...
```

​		这个方法访问harry的私有域，这点并不会引发奇怪。然而，**还访问boss的私有域。这是合法的，其原因是boss是Employee类对象，而Employee类的方法可以访问Employee类的任何一个对象的私有域。**

### （4）final实例域

​		可以将实例域定义为final。构建对象时必须初始化这样的域。也就是说，必须确保在每一个构造器执行之后，这个域的值被设置，并且在后面的操作中，不能够再对它进行修改。例如，可以将Employee类中的name域声明为final，因为在对象构建之后，这个值不会再被修改，即没有setName方法。

```java
class Employee
{
    ...
    private final String name;
}
```

​		final修饰符大都应用于基本数据(primitive）类型域，或不可变(immutable)类的域（如果类中的每个方法都不会改变其对象，这种类就是不可变的类。例如，String类就是一个不可变的类)。对于可变的类，使用final修饰符可能会对读者造成混乱。

## 2.静态域与静态方法

### （1）静态域

​		这里给Employee类添加一个实例域id和一个静态域nextId。

```java
class Employee
{
    ...
    private int id;
    private static int nextId=1;
}
```

​		现在，每一个雇员对象都有一个自己的id域，但这个类的所有实例共享一个nextId域。**它属于类，而不属于任何独立的对象。**

### （2）静态变量

```java
class Math
{
    ...
    public static final double PI=3.1415;
    ...
}
```

​		在程序中可以采用Math.PI的形式获得这个变量。

​		如果关键字static被省略，PI就变成了Math类的一个实例域，需要通过Math类的对象访问PI。

### （3）静态方法

​		静态方法是一种不能向对象实施操作的方法，例如Math.pow(x,a)，运算时不能用任何Math对象，即没有隐式的参数。**可以认为静态方法是没有this参数的方法**。

​		因为静态方法不能操作对象，所以不能在静态方法中访问实例域，但是可以访问自身类中的静态域。

```java
public static int getNextId()
{
	return nextId;
}

//可以通过类名调用这个方法
int n=Employee.getNextId() 
```

​		static可以省略，但是那就需要通过Employee类的对象的引用调用这个方法。

> 可以使用对象调用静态方法，例如可以用harry.getNextId() 代替Employee.getNextId() 。但是这种getNextId方法计算的结果与harry毫无关系，容易造成混淆。

## 3.方法参数

- 值调用：表示方法接收的是调用者提供的值。
- 引用调用：表示方法接收的是调用者提供的变量地址。

​		一个方法可以修改传递引用所对应的变量值，而不能修改传递值引用所对应的变量值。

​		Java总是采用值调用，也就是说方法得到的是所有参数值的一个拷贝，并不能修改传递给它的任何参数变量的内容。

```java
public static void tripleValue(double x)
{
    x=3*x;
}
double percent=10;
tripleValue(percent);//percent=10
```

​		方法参数共有两种类型：

- 基本数据类型
- 对象引用

​		一个方法不可能修改一个基本数据类型的参数，而对象引用作为参数就不同了：

```java
public static void tripleValue(Employee x)
{
    x.raiseSalary(200);
}
harry=new Employee(...);
tripleSalary(harry);
```

​		此时x的薪资被成功增至3倍。

## 4.对象构造

### （1）重载

​		若多个方法有相同的名字、不同的参数，便产生了重载。

​		Java允许重载任何方法而不只是构造器方法，因此**要完整地描述一个方法，需要指出方法名以及类型参数，这叫做方法的签名。**

​		返回类型不是方法签名的一部分，也就是说不能有两个名字相同、参数类型也相同却返回不同类型值的方法。

### （2）默认域初始化

​		如果在构造器中没有显式地给域赋予初值，那么就会被自动地赋为默认值:数值为0、布尔值为flase、对象引用为null。**这是域与局部变量的主要不同点。必须明确地初始化方法中的局部变量。但是，如果没有初始化类中的域，将会被初始化为默认值（0、false或null)。**

### （3）默认构造器

​		默认构造器即没有参数的构造器：

```java
public Employee()
{
	name="";
    salary=0;
    hireDay=new Date();
}
```

​		如果在编写一个类时没有编写构造器，那么系统就会提供一个默认构造器。

​		如果类中提供了至少一个构造器，但是没有提供默认的构造器，则在构造对象时如果没有提供构造参数就会被视为不合法。

​		如果希望所有域被赋予默认值，可以采用如下格式：

```java
public className()
{

}
```

### （4）显示域初始化

​		在类定义中可以直接将一个值赋给任何域,在执行构造器之前，先执行赋值操作。当一个类的所有构造器都希望把相同的值赋予某个特定的实例域时，这种方式特别有用。

​		初始值不一定是常量，也可以是方法：

```java
class Employee
{
	...
    static int assignId()
    {
        int r=nextId++;
        return r
    }
    ...
    private int id=assignId();
    private String name="";
}
```

### （5）参数名

​		还种常用的技巧，它基于这样的事实:参数变量用同样的名字将实例城屏敲起来。例如,如果将参数命名为salary，salary将引用这个参数，而不是实例域。但是，可以采用this.salary的形式访问实例域。回想一下，this指示隐式参数，也就是被构造的对象。下面是一个示例：

```java
public Employee(String name,double salary)
{
    this.name=name;
    this.salary=salary;
}
```

### （6）调用另一个构造器

​		如果构造器的第一个语句形如this(...)，这个构造器将调用同一个类的另一个构造器。下面是一个典型的例子:

```java
public Employee(double s)
{
    //calls Employee(String,double)
    this("Employee #"+nextId,s);
    nextId++;
}
```

​		当调用new Employee(60000)时，Employee(double)构造器将调用Employee(String, double)构造器,这样对公共的构造器代码部分只编写一次即可。

### （7）初始化块

​		在这个示例中，无论使用哪个构造器构造对象，id域都在对象初始化块中被初始化。首先运行初始化块，然后才运行构造器的主体部分。

```java
class Employee{
	public Employee(String n，double s)
    {
		name = n;
        salary = s;
    }
	public Employee()
    {
		name = "";
        salary = 0;
    }
    ...
	private static int nextId;
	private int id;
	private String name;private double salary;
    ...
	// object initialization block
    {
		id = nextId;
    	nextId++;
    }
}
```

​		由于初始化数据域有多种途径，所以列出构造过程的所有路径可能相当混乱。下面是调用构造器的具体处理步骤:

1. 所有数据域被初始化为默认值（0、false或null)。

2. 按照在类声明中出现的次序，**依次执行所有域初始化语句和初始化块。**

3. 如果构造器第一行调用了第二个构造器，则执行第二个构造器主体。

4. 执行这个构造器的主体。

   ​	如果对类的静态域进行初始化的代码比较复杂，那么可以使用静态的初始化块。

将代码放在一个块中，并标记关键字static。下面是一个示例。其功能是将雇员ID的起始值赋予一

个小于10000的随机整数。

```java
static
{
    Random generator=new Random();
    nextId=generator.nextInt(10000);
}
```

### （8）对象析构与finalize方法

​		Java有自动的垃圾回收器，不需要人工回收内存，所以Java不支持析构器。

​		当然，某些对象使用了内存之外的其他资源，例如，文件或使用了系统资源的另一个对象的句柄。在这种情况下，当资源不再需要时，将其回收和再利用将显得十分重要。

​		可以为任何一个类添加finalize方法。finalize方法将在垃圾回收器清除对象之前调用。在实际应用中，不要依赖于使用finalize方法回收任何短缺的资源，这是因为很难知道这个方法什么时候才能够调用。

# 三、继承

## 1.类、超类和子类

​		下面是由Employee类来定义Manager类的格式，关键字extends表示继承：

```java
class Manager extends Employee
{
    添加方法和域
}
```

​		关键字extends表明正在构造的新类派生于一个已存在的类。已存在的类被称为超类、基类或父类。新类被称为子类、派生类。

​		在Manager类中，增加了一个用于存储奖金信息的域，以及一个用于设置这个域的方法:

```java
class Manager extends Employee
{
    ...
    public void setBonus(double b)
    {
        bonus=b;
    }
    private double bonus;
}
```

​		由于setBonus方法不是在Employee类中定义的，所以属于Employee类的对象不能使用它。

​		然而，尽管在Manager类中没有显式地定义getName和getHireDay等方法，但属于Manager类的对象却可以使用它们，这是因为Manager类自动地继承了超类Employee中的这些方法。

​		同样，从超类中还继承了name、salary和hireDay这3个域。这样一来，每个Manager类对象就包含了4个域:name、salary、hireDay和bonus。
​		在通过扩展超类定义子类的时候，仅需要指出子类与超类的不同之处。因此在设计类的时候，应该将通用的方法放在超类中，而将具有特殊用途的方法放在子类中，这种将通用的功能放到超类的做法，在面向对象程序设计中十分普遍。
​		然而，超类中的有些方法对子类Manager并不一定适用。例如，**在Manager类中的getSalary方法应该返回薪水和奖金的总和**。为此，需要提供一个新的方法来**覆盖 (override)**超类中的这个方法:

```java
public double getSalary()
{
    return salary+bonus;//错误，Manager类的方法不能直接访问超类的私有域
}

public double getSalary()
{
    double baseSalary=getSalary();
    //错误，Manager类也有getSalary方法，这条语句会无限次调用自己
    return baseSalary+bonus;
}

//这里需要指出，我们希望调用的getSalary方法是超类Employee中的，而不是当前类的。
//为此，可以使用特定的关键字super解决这个问题：
public double getSalary()
{
    double baseSalary=super.getSalary();
    return baseSalary+bonus;
}
```

> super与this不同，super不是一个对象的引用，不能将super赋给另一个对象变量，它只是一个指示编译器调用超类方法的特有关键字。

​		最后看一下super在构造器中的应用。

```java
public Manager(String n,double s,int year,int month,int day)
{
    super(n,s,year,month,day);
    //该语句意为调用超类Employee中含有n、s、year、month和day参数的构造器
    bonus=0;
}
```

- 使用super调用构造器的语句**必须是子类构造器的第一条语句。**
- 如果子类的构造器没有显式地调用超类的构造器，则将自动地调用超类默认（没有参数)的构造器。如果超类没有不带参数的构造器，并且在子类的构造器中又没有显式地调用超类的其他构造器，则Java编译器将报告错误。

​		下面定义一个包含3个雇员的数组，将经理和雇员都放到数组中：

```java
Manager boss=new Manager("Carl Cracker",80000,1987,12,15);
boss.setBonus(5000);
Employee[] staff=new Employee[3];
staff[0]=boss;
staff[1]=new Employee("Harry Hacker",50000,1989,10,1);
staff[2]=new Employee("Tony Tester",40000,1990,3,15);
for(Employee e:staff)
    System.out.println(e.getName()+" "+e.getSalary());
//Carl Cracker 85000.0
//Harry Hacker 50000.0
//Tony Tester  40000.0
```

​		需要提到的是，e.getSalary(）能够确定应该执行哪个getSalary方法。请注意，尽管这里将e声明为Employee类型，但实际上e既可以引用Employee类型的对象，也可以引用Manager类型的对象。
​		当e引用Employee对象时，e.getSalary()调用的是Employee类中的getSalary方法，当e引用Manager对象时，e.getSalary()调用的是Manager类中的getSalary方法。**虚拟机知道e实际引用的对象类型，因此能够正确地调用相应的类方法。**

- 一个对象变量（例如，变量e）可以引用多种实际类型的现象被称为**多态**；
- 在运行时能够自动地选择调用哪个方法的现象称为**动态绑定**。

### （1）继承层次

​		Java不支持多继承，多继承功能需要通过接口实现。

![image-20210410235716681](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210410235716681.png)

### （2）多态

​	Java中对象变量是多态的，一个Employee变量既可以引用一个Employee对象，也可以引用Employee类的任何一个子类对象。

```java
Manager boss=new Manager("Carl Cracker",80000,1987,12,15);
Employee[] staff=new Employee[3];
staff[0]=boss;
```

​		staff[0]和boss引用同一个对象，但编译器将staff[0]看成Employee对象。

```java
boss.setBonus(5000);//ok
staff[0].setBonus(5000);//error
```

​		第二种用法错误的原因是staff[0]声明的类型是Employee，二setBonus不是类Employee的方法。

​		不能将一个超类的引用赋给子类变量。

```java
Manager m=staff[i];//error
```

​		**子类是超类的子集。**

​		在Java中，子类数组的引用可以转换成超类数组的引用，而不需要采用强制类型转换。例如，下面是一个经理数组：

```java
Manager[] managers=new Manager[10];
Employee[] staff=managers;//ok
```

​		这样做肯定不会有问题，请思考一下其中的缘由。毕竟，如果managerli]是一个Manager,也一定是一个Employee。然而，实际上，将会发生一些令人惊讶的事情。要切记managers和staff引用的是同一个数组。现在看一下这条语句:

```java
staff[0]=new Employee("Uranus");
```

​		编译器竟然接纳了这个赋值操作。但在这里，staff[0]与manager[0]引用的是同一个对象，似乎我们把一个普通雇员擅自归入经理行列中了。这是一种很忌讳发生的情形，当调用managers[0].setBonus(1000)的时候，将会导致调用一个不存在的实例域，进而搅乱相邻存储空间的内容。
​		为了确保不发生这类错误，**所有数组都要牢记创建它们的元素类型，并负责监督仅将类型兼容的引用存储到数组中**。例如，使用**new managers[10]创建的数组是一个经理数组**。如果**试图存储一个Employee类型的引用就会引发ArrayStoreException异常**。

### （3）动态绑定

​		弄清调用对象方法的执行过程十分重要。

1. 编译器查看对象的声明类型和方法名。假设调用`x.f(param)`，x声明为C类。需要注意：有可能存在多个名字为f，但参数类型不一样的方法，例如：`f(int),f(String)`。**编译器会一一列举C类中所有名为f的方法和其<u>超类中访问属性为public且名为f的方法</u>。**

   至此，编译器已获得所有可能被调用的候选方法。

2. 接下来，编译器将查看调用方法时提供的参数类型。如果所有名为f的方法中存在一个与提供的参数类型完全匹配的方法，就选择这个方法。否则若编译器没有找到与参数类型匹配的方法，或经过类型转换后有多个方法与之匹配，就会报错。

   至此，编译器已获得需要调用的方法名字和参数类型。

3. 如果是private方法、static方法、final方法或者构造器，那么编译器将可以准确地知道应该调用哪个方法，我们将这种调用方式称为静态绑定。

4. 当程序运行，并且采用动态绑定调用方法时，**虚拟机一定调用与x所引用对象的实际类型最合适的那个类的方法**。假设**x 的实际类型是D，它是C类的子类。如果D类定义了方法(String)，就直接调用它，否则，将在D类的超类中寻找f(String)，以此类推。**

​		每次调用方法都要进行搜索，时间开销相当大。因此，**虚拟机预先为每个类创建了一个方法表，其中列出了所有方法的签名和实际调用的方法。**这样一来，在真正调用方法的时候，虚拟机仅查找这个表就行了。在前面的例子中，虚拟机搜索D类的方法表，以便寻找与调用f(Sting)相匹配的方法。这个方法既有可能是D.f(String)，也有可能是X.f(String),这里的X是D的超类。这里需要提醒一点，**如果调用super.f(param)，编译器将对隐式参数超类的方法表进行搜索。**

​		现在，查看一下调用e.getSalary()的详细过程。e声明为Employee类型。Employee类只有一个名叫getSalary的方法，这个方法没有参数。因此，在这里不必担心重载解析的问题。

​		由**于getSalary不是private方法、static方法或final方法，所以将采用动态绑定。**虚拟机为Employee和Manager两个类生成方法表。在Employee的方法表中，列出了这个类定义的所有方法:

```java
Employee:
	getName() -> Employee.getName()
	getSalary() -> Employee.getsalary()
    getHireDay() -> Employee.getHireDay()
	raiseSalary(double) -> Employee.raiseSalary(double)
```

​		Manager方法表稍微有些不同。其中有三个方法是继承而来的，一个方法是重新定义的，还有一个方法是新增加的。

```java
Manager:
	getName() -> Employee.getName()
    getSalary() -> Manager.getSalary()
    getHireDay() -> Employee.getHireDay( )
	raiseSalary(double) -> Employee.raiseSalary(double)
    setBonus(double) -> Manager.setBonus(double)

```

​		在运行的时候，调用e.getSalary()的解析过程为:

1. 首先，虚拟机提取e的实际类型的方法表。既可能是Employee、Manager的方法表，也可能是Employee类的其他子类的方法表。
2. 接下来，虚拟机搜索定义getSalary签名的类。此时，虚拟机已经知道应该调用哪个方法。
3. 最后，虚拟机调用方法。

​		动态绑定有一个非常重要的特性:无需对现存的代码进行修改，就可以对程序进行扩展。假设增加一个新类Executive，并且变量e有可能引用这个类的对象，我们不需要对包含调用e.getSalary()的代码进行重新编译。如果e恰好引用一个Executive类的对象，就会自动地调用Executive.getSalary()方法。

> **在覆盖一个方法的时候，子类方法不能低于超类方法的可见性。特别是，如果超类方法是public，子类方法一定要声明为public。**经常会发生这类错误:在声明子类方法的时候，遗漏了public修饰符。此时，**编译器将会把它解释为试图降低访问权限。**

### （4）阻止继承：final类和方法

​		有时候，**可能希望阻止人们利用某个类定义子类。不允许扩展的类被称为final类。**如果在定义类的时候使用了final修饰符就表明这个类是final类。例如，假设希望阻止人们定义Executive类的子类，就可以在定义这个类的时候，使用final修饰符声明。声明格式如下所示：

```java
final class Executive extends Manager
{
    ...
}
```

​		**类中的方法也可以被声明为final。如果这样做，子类就不能覆盖这个方法（final类中的所有方法自动地成为final方法)。**例如:

```java
class Employee
{
    ...
    public final String getName()
    {
        return name;
    }
    ...
}
```

> 注前面曾经说过，域也可以被声明为final。对于final域来说，构造对象之后就不允许改变它们的值了。不过，**如果将一个类声明为final，只有其中的方法自动地成为final，而不包括域。**

### （5）强制类型转换

​		有时候可能需要将某个类的对象引用转换成另外一个类的对象引用。例如：

```java
Manager boss=(Manager) staff[0];
```

​		进行类型转换的唯一原因是:在暂时忽视对象的实际类型之后，使用对象的全部功能。在managerTest类中，由于某些项是普通雇员，所以staff数组必须是Employee对象的数组。**我们需要将数组中引用经理的元素复原成Manager类，以便能够访问新增加的所有变量。**

​		将一个值存入变量时，编译器将检查是否允许该操作。将一个子类的引用赋给一个超类变量，编译器是允许的。但**将一个超类的引用赋给一个子类变量，必须进行类型转**换，这样才能够通过运行时的检查。

​		若试图在继承链上进行向下的类型转换，程序会抛出ClassCastException异常。若没有捕获异常，则会终止程序。正确的做法是在进行类型转换之前，先查看一下能否成功转换：

```java
if(staff[1] instanceof Manager)
{
    //istanceof用于测试左边的对象是否是它右边的类的实例
    //（可理解为测试是否是右边的类继承链向下的关系）
    boss=(Manager)staff[1];
    ...
}
```

### （6）抽象类

​		如果自下而上仰视类的继承层次结构，位于上层的类更具有通用性，甚至可能更加抽象。从某种角度看，祖先类更加通用，人们只将它作为派生其他类的基类，而不作为想使用的特定的实例类。例如，考虑一下对Employee类层次的扩展。一名雇员是一个人，一名学生也是一个人。

![image-20210411114259658](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210411114259658.png)

​		现在可以添加一个getDescription方法，返回对一个人的简短描述。

```
an employee with a salary of s50,000.00
a student majoring in computer science
```

​		在Employee类和Student类中实现这个方法很容易。但是在Person类中应该提供什么内容呢?除了姓名之外，Person类一无所知。

​		可以使用abstract关键字：

```java
public abstract String getDescription();
```

​		为了提高程序的清晰度，包含一个或多个抽象方法的类本身必须被声明为抽象的。除了抽象方法，抽象类还可以包含具体数据和具体方法：

```java
abstract class Person
{
    ...
    public abstract String getDescription();
    public Person(String n)
    {
        name=n;
    }
    private String name;
}
```

​		抽象方法充当着占位的角色，它们的具体实现在子类中。扩展抽象类可以有两种选择。一种是**在子类中定义部分抽象方法或抽象方法也不定义**，这样就必须将子类也标记为抽象类，另一种是定义全部的抽象方法，这样一来，子类就不是抽象的了。

- 类即使不含抽象方法，也可以将类声明为抽象类；

- 抽象类不能被实例化；

- 可以定义一个抽象类的对象变量，但是它只能引用非抽象子类的对象

  ```java
  new Person("Uranus");//wrong
  Person P=new Studen("Uranus","Economics");//ok
  
  Person[] people = new Person[2];
  people[0] = new Employee(. . .);
  people[1] = new Student(. . .);
  for (Person p : people)
  	System.out.printin(p.getName() + "," + p.getDescription());
  
  ```

  由于不能构造抽象类Person的对象，所以**变量p永远不会引用Person对象，而是引用诸如Employee或Student这样的具体子类对象**，而在**这些对象中都定义了getDescription方法。**

### （7）受保护访问

​		最好将类中的域标记为private，而方法标记为public。任何声明为private的内容对其他类都是不可见的。前面已经看到，这对于子类来说也完全适用，即**子类也不能访问超类的私有域**。

​		然而，在有些时候，人们希望超类中的某些方法允许被子类访问，或允许子类的方法访问超类的某个域。为此，需要将这些方法或域声明为protected。例如，如果将超类Employee中的hireDay声明为proteced，而不是私有的，Manager中的方法就可以直接地访问它。

​		不过，**<u>Manager类中的方法</u>只能够访问Manager对象中的hireDay城，而不能访问其他Employee对象中的这个域。**这种限制有助于避免滥用受保护机制，使得子类只能获得访问受保护域的权利。

1. 仅对本类可见——private。
2. 对所有类可见——public。
3. 对本包和所有子类可见———protected。
4. 对本包可见——默认，所谓默认是指没有标明任何修饰符的情况，这是一种不太受欢迎的形式。

## 2.Object：所有类的超类

​		可以使用Object类型的变量引用任何类型的对象：

```java
Object obj=new Employee("Uranus",30000);
```

​		当然，Object类型的变量只能用于作为各种值的通用持有者。**要想对其中的内容进行具体的操作，还需要清楚对象的原始类型，并进行相应的类型转换**:

```java
Employee e=(Employee)obj;
```

​		java中只有基本类型不是对象，所有数组类型都扩展于Object类。

### （1）equals方法

​		equals方法用于检测两个对象是否具有相同的引用，若具有相同的引用，它们一定相等。

​		在子类中定义equals方法时，首先调用超类的equals。如果检测失败，对象就不可能相等。如果超类中的域都相等，就需要比较子类中的实例域。

```java
class Manager extends Employee
{
	. . .
	public boolean equa1s(Object otherObject)
    {
		if (!super.equals(other0bject)) return false;
		Manager other = (Manager) otherObject;
		return bonus == other.bonus;
	}
}
```

### （2）相等测试与继承

​		下面给出一个完美的equals方法的建议：

1. 显式参数命名为otherObject

2. 检测this与otherObject是否引用同一个对象：

   ```java
   if(this==otherObject) return true;
   ```

   这条语句只是一个优化。实际上，这是一种经常采用的形式。因为计算这个等式要比一个一个地比较类中的域所付出的代价小得多。

3. 检测otherObject是否为null，若为null，则返回false。

   ```java
   if(otherObject==null) return false;
   ```

4. 比较this与otherObject是否属于同一个类。

   - 如果equals的语义在每个子类中有所改变，就用getClass检测；
   - 若所有的子类都有统一的语义，就用instanceof检测：

   ```java
   if(getClass()!=otherObject.getClass()) return false;
   if(!(otherObject instanceof ClassName)) return false;
   ```

5. 将otherObject转换为相应的类类型变量：

   ```java
   ClassName other=(ClassName)otherObject;
   ```

6. 现在开始对所有需要比较的域进行比较。使用==比较基本类型域，使用equals比较对象域。

   ```java
   return field1==other.field1 && field2.equals(other.field2)&&...;
   ```

   > **对于数组类型的域，可以使用静态的Arrays.equals方法检测相应的数组元素是否相等。**

### （3）hashCode方法

​		散列码(hash code)是由对象导出的一个整型值。散列码是没有规律的。如果x和y是两个不同的对象，x.hashCode( )与y.hashCode()基本上不会相同。

```java
String s ="Ok";
StringBuilder sb = new StringBuilder(s);
System.out.println(s.hashCode() + " " +sb.hashCode());String t = new String("0k");
StringBuilder tb = new StringBuilder(t);
System.out.println(t.hashCode() + " “ ÷tb.hashCode());
```

​		请注意，字符串s与t拥有相同的散列码，这是因为**字符串的散列码是由内容导出的**。而字符串缓冲sb与tb却有着不同的散列码，这是因为在**StringBuffer类中没有定义hashCode方法，它的散列码是由Object类的默认hashCode方法导出的对象存储地址。**

​		如果重新定义equals方法，就必须重新定义hashCode方法，以便用户可以将对象插入到散列表中。**Equals与hashCode的定义必须一致:如果x.equals(y)返回true，那么x.hashCode()就必须与y.hashCode()具有相同的值。**例如，如果用定义的Employee.equals比较雇员的ID,那么hashCode方法就需要散列ID，而不是雇员的姓名或存储地址。

> 如果存在数组类型的域，那么**可以使用静态的Arrays.hashCode方法计算一个散列码，这个散列码由数组元素的散列码组成。**

### （4）toString方法

​		toString方法返回表示对象值的字符串。绝大对数的toString方法都遵循这样的格式：类的名字，随后是一对方括号括起来的域值。

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

> 数组类型需要调用Arrays.toString()生成字符串。

## 3.泛型数组列表

​		为了解决运行时动态更改数组的问题，可以使用ArrayList类。使用起来类似数组，但在添加或删除元素时具有自动调节数组容量的功能。

​		ArrayList是一个采用类型参数的泛型类。为指定数组列表保存的元素对象类型，需要用一对尖括号将类名括起来加在后面， 例如`ArrayList<Employee>`.

​		下面声明和构造一个保存Employee对象的数组列表：

```java
ArrayList<Employee> staff=new ArrayList<Employee>();
staff.add(new Employee("Uranus",...));
```

​		数组列表管理着对象引用的一个内部数组。最终，数组的全部空间有可能被用尽。这就显现出数组列表的操作魅力:如果调用add且内部数组已经满了，数组列表就将自动地创建一个更大的数组，并将所有的对象从较小的数组中拷贝到较大的数组中。

​		如果已经清楚或能够估计出数组可能存储的元素数量，就可以在填充数组之前调用ensureCapacity方法:

```java
staff.ensureCapacity(100);
```

​		另外还可以把初始容量传递给ArrayList构造器：

```java
ArrayList<Employee> staff=new ArrayList<Employee>(100);
```

​		size方法将返回数组列表中包含的实际元素数目。例如`staff.size()`.

​		一旦能够确认数组列表的大小不再发生变化，就可以调用`trimToSize`方法。这个方法将存储区域的大小调整为当前元素数量所需要的存储空间数目。垃圾回收器将回收多余的存储空间。

​		一旦整理了数组列表的大小，添加新元素就需要花时间再次移动存储块，所以应该在确认不会添加任何元素时，再调用trimToSize。

> C++向量是值拷贝。如果a和b是两个向量，赋值操作a = b将会构造一个与b长度相同的新向量a，并将所有的元素由b拷贝到a，而在Java中，**这条赋值语句的操作结果是让a和b引用同一个数组列表，即对a的修改也会同时在b中生效。**

### （1）访问数组列表元素

​		**使用get和set方法实现访问或改变数组元素的操作**，而不使用人们喜爱的[]语法格式。例如,要设置第i个元素，可以使用:

```java
staff.set(i,Uranus);
//等价于对数组a的元素赋值
a[i]=harry;
```

​		**使用add方法为数组添加新元素，而不要使用set方法，它只能替换数组中已经存在的元素内容。**

​		同理使用get获得数组列表的元素：

```java
Employee e=staff.get(i);
//等价于
Employee e=a[i];
```

​		下面这个技巧可以一举两得，既可以灵活地扩展数组，又可以方便地访问数组元素。	

​		首先，创建一个数组，并添加所有的元素，之后使用toArray方法将数组列表元素拷贝到一个数组中。

```java
ArrayList<X> list = new ArrayList<X>();
while(...)
{
    x=...;
    list.add(x);
}
X[] a=new X[list.size()];
list.toArray(a);//此时数组a就拥有了list中所有元素
```

​		除了在数组列表的尾部追加元素之外，还可以在数组列表的中间插入元素，使用带索引参数的add方法。**为了插人一个新元素，位于n之后的所有元素都要向后移动一个位置。如果插入新元素后，数组列表的大小超过了容量，数组列表就会被重新分配存储空间。**

​		同样地,可以从数组列表中删除一个元素。

```java
int n=staff.size()/2;
staff.add(n,e);
Employee e=staff.remove(n);
```

## 4.对象包装器与自动打包

​		有时，需要将int这样的基本类型转换为对象。所有的基本类型都有一个与之对应的类。例如，Integer类对应基本类型int。通常，**这些类称为包装器(wrapper)**。这些对象包装器类拥有很鲜明的名字:Integer、Long、Float、Double、Short、Byte、Character、Void和Boolean (前6个类派生于公共的超类Number)。对象包装器类是不可变的，即一旦构造了包装器，就不允许更改包装在其中的值。同时，对象包装器类还是final，因此不能定义它们的子类。

​		假设想定义一个整型数组列表。而尖括号中的类型参数不允许是基本类型：

```java
ArrayList<int> list=new ArrayList<int>();//error
ArrayList<Integer> list=new ArrayList<Integer>();//ok        
```

​		下面这个调用会发生自动变换，这种变换被称为**自动打包或装箱**。

```java
list.add(3);//翻译成如下
list.add(new Integer(3));
```

​		相反地，当将一个Integer对象赋给一个int值时，将会自动拆包或拆箱。

```java
int n=list.get(i);//翻译成如下
int n=list.get(i).intValue();
```

​		因为Integer是对象类型，所以下面的比较通常会不成立。

```java
Integer a=1000;
Integer b=1000;
if(a==b)...
```

> **自动打包规范要求boolean、byte、char≤127，介于一128~127之间的short和int被包装到固定的对象中。例如，如果在前面的例子中将a和b初始化为100，对它们进行比较的结果一定成立。**

## 5.参数数量可变的方法

​		现在的java版本提供了可以用可变的参数数量调用的方法。用户可以定义可变参数的方法，并将参数指定为任意类型，甚至是基本类型。下面是一个简单的示例:其功能为计算若干个数值的最大值。

```java
public static double max(double...values)
{
    double largest=Double.MIN_VALUE;
    for(double v:values)
        if(v>largest) 
            largest=v;
    return largest;
}
```

​		允许将一个数组传递给可变参数方法的最后一个参数。例如:

`System.out.printf("%d %s",new 0bject[]{ new Integer(1)，"widgets"} );`

​		**因此，可以将已经存在且最后一个参数是数组的方法重新定义为可变参数的方法**，而不会破坏任何已经存在的代码。

## 6.枚举类

​		下面是一个典型的例子，实际上，这个声明定义的类型是一个类，它刚好有4个实例，尽量不要构造新对象。

```java
public enum Size{SMALL,MEDIUM,LARGE,EXTRA_LARGE};
```

​		在比较两个枚举类型的值时，直接使用“==”就可以了。

​		如果需要的话，可以在枚举类型中添加一些构造器、方法和域。当然，构造器只是在构造枚举常量的时候被调用。下面是一个示例:

```java
enum Size
{
    SMALL("S")，NEDIUMK“"M"")，LARGE(""L"")，EXTRA_LARCE(""XL");
	private Size(String abbreviation){this.abbreviation=abbreviation;}
    public String getAbbreviation( ) {return abbreviation; }
	
    private String abbreviation;
}
```

​		所有的枚举类型都是Enum类的子类。它们继承了这个类的许多方法。其中最有用的一个是toString，这个方法能够返回枚举常量名。例如，Size.SMALL.toString( )将返回字符串“SMALL”。

​		toString的逆方法是静态方法valueOf。例如，语句:

```java
Size s=(Size) Enum.valueOf(Size.class,"SMALL");
```

​		将s设置成Size.SMALL。

​		每个枚举类型都有一个静态的values方法，它将返回一个包含全部枚举值的数组。例如,如下调用：`Size[] values - Size.values();`
​		返回包含元素Size.SMALL,Size.MEDIUM,Size.LARGE和Size.EXTRA_LARGE的数组。

​		ordinal方法返回enum声明中枚举常量的位置，位置从0开始计数。例如:Size.MEDIUM.ordinal()返回1。

## 7.反射

​		**能够分析类能力的程序被称为反射（reflective)。**

### （1）Class类

​		在程序运行期间，Java运行时系统始终为所有的对象维护一个被称为运行时的类型标识。这个信息保存着每个对象所属的类足迹。虚拟机利用运行时信息选择相应的方法执行。

​		然而，可以通过专门的Java类访问这些信息。保存这些信息的类被称为Class，这个名字很容易让人混淆。Object类中的getClass()方法将会返回一个Class类型的实例。

```java
Employee e=new Employee("Uranus",10000);
Class c1=e.getClass();//class Employee
String name=c1.getName();//Employee
```

​		如果类在一个包里，包的名字也作为类名的一部分：

```java
Date d=new Date();
String name=d.getClass().getName();
System.out.println(name);//java.util.Date
```

​		获得Class类对象的第三种方法非常简单。如果T是任意的Java类型，T.class将代表匹配的类对象。例如:

```java
System.out.println(int[].class);//class[I
```

​		请注意，一个Class对象实际上表示的是一个类型，而这个类型未必一定是一种类。例如，int不是类，但int.class是一个Class类型的对象。

### （2）捕获异常

```java
try
{
	statements that might throw exceptions
}
catch(Exception e)
{
    handler action
}
```

### （3）利用反射分析类的能力

​		下面简要地介绍一下反射机制最重要的内容——检查类的结构。

​		在java.lang.reflect包中有三个类**Field、Method和Constructor分别用于描述类的域、方法和构造器。**这三个类都有一个叫做getName的方法，用来返回项目的名称。

​		Field类有一个getType方法，用来返回描述域所属类型的Class对象。Method和Constructor类有能够报告参数类型的方法，Method类还有一个可以报告返回类型的方法。

​		这三个类还有一个叫做getModifiers的方法，它将返回一个整型数值，用不同的位开关描述public和static这样的修饰符使用状况。另外，还可以利用java.lang.reflect包中的Modifier类的静态方法分析getModifiers返回的整型数值。例如，可以使用Modifier类中的isPublic、isPrivate或isFinal判断方法或构造器是否是public、private或final。我们需要做的全部工作就是调用Modifier类的相应方法，并对返回的整型数值进行分析，另外，还可以利用Modifier.toString方法将修饰法打印出来。
​		Class类中的getFields、getMethods和getConstructors方法将分别返回类提供的public域、方法和构造器数组，其中包括超类的公有成员。Class类的getDeclareFields、getDeclareMethods和getDeclaredConstructors方法将分别返回类中声明的全部域、方法和构造器，其中包括私有和受保护成员，但不包括超类的成员。

### （4）在运行时使用反射分析对象

# 四、接口和内部类

## 1.接口

​		在Java程序设计语言中，接口不是类，而是对类的一组需求描述，这些类要遵从接口描述的统一格式进行定义。

​		Arrays类中的sort方法承诺可以对对象数组进行排序，但要求满足下列前提：对象所属的类必须实现了Comparable接口。

​		下面是Comparable接口的代码：

```java
public interface Comparable<T>
{
	int compareTo(T other);	
}
```

​		接口中的所有方法自动地属于public。因此，在接口中声明方法时，不必提供关键字public。当然，接口中还有一个没有明确说明的附加要求:在调用x.compareTo(y)的时候，这个compareTo方法必须确实比较两个对象的内容，并返回比较的结果。**当x小于y时，返回一个负数:当x等于y时，返回0，否则返回一个正数。**

​		接口绝不能含有实例域，也不能在接口中实现方法。提供实例域和方法实现的任务应该由实现接口的那个类来完成。因此，可以将接口看成是没有实例域的抽象类,但是这两个概念还是有一定区别的.

​		现在，假设希望使用Arrays类的sort方法对Employee对象数组进行排序，Employee类就必须实现Comparable接口。

```java
class Employee implements Comparable
{
    public int compareTo(Employee other)
    {//实现接口时必须将方法声明为public
        if(salary<other.salary) return -1;
        if(salary>other.salary) return 1;
        return 0;
    }
}
Employee[ ] staff = new Employee[3];
staff[0] = new Employee( "Harry Hacker"，35000);
staff[1] = new Employee("Car1 Cracker"，75000);
staff[2] = new Employee("Tony Tester", 38000);
Arrays.sort(staff);
```

### （1）接口的特性

​		接口不是类，不能使用new运算符实例化一个接口，但是可以声明接口的变量，接口变量必须引用实现了接口的类对象。也可以使用instanceof检查一个对象是否实现了某个特定的接口。

```java
x=new Comparable(...);//error
Comparable x;//ok
x=new Employee(...);//ok
if(anObject instanceof Comparable){...}
```

​		接口也可以被扩展。

​		**接口中虽然不能包含实例域或静态方法，但可以包含常量。**

```java
public interface Moveable
{
    void move(double x,double y);
}
public interface entends Moveavle
{
    double milesPerGallon();
    double PI=3.14;
}
```

​		与接口中的方法都自动地被设置为public一样，**接口中的域将被自动设为public static final。**

​		尽管每个类只能够拥有一个超类，但却可以实现多个接口。这就为定义类的行为提供了极大的灵活性。例如，Java程序设计语言有一个非常重要的内置接口，称为Cloneable(将在下一节中给予详细的讨论)。如果某个类实现了这个Cloncable接口，Object类中的clone方法就可以创建类对象的一个拷贝。如果希望自己设计的类拥有克隆和比较的能力，只要实现这两个接口就可以了。

```java
class Employee implements Cloneable,Comparable
```

### （2）接口和抽象类

​		为什么Java程序设计语言还要不辞辛苦地引入接口概念?为什么不将Comparable直接设计成抽象类？

​		使用抽象类表示通用属性存在这样一个问题:每个类只能扩展于一个类。假设Employee类已经扩展于一个类，例如Person，它就不能扩展第二个类了。

## 2.对象克隆

​		当拷贝一个变量时，原始变量与拷贝变量引用同一个对象，如图6-1所示。这就是说，改变一个变量所引用的对象将会对另一个变量产生影响。

![image-20210411224743532](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210411224743532.png)

​		如果创建一个对象的新的copy，它的最初状态与original一样，但以后将可以各自改变各自的状态，那就需要使用clone方法。

```java
Employee original=new Employee("John Public",50000);
Employee copy=original.clone();
copy.raiseSalary(10);//OK,original unchanged
```

​		不过，事情并没有这么简单。clone方法是Object类的一个proteced方法，也就是说，在用户编写的代码中不能直接调用它。只有Employee类才能够克隆Employee对象。

​		默认的克隆操作是浅拷贝，它并没有克隆包含在对象中的内部对象，因此必须重新定义clone方法，以便实现克隆子对象的深拷贝。

​		对每一个类都需要做出如下判断：

1. 默认的clone方法能否满足需求
2. 默认的clone方法能否通过调用可变子对象的clone得到修补
3. 是否不应该使用clone

​		事实上，选项3是否认的，若要选择1或2，类必须

- 实现Cloneable接口
- **使用public访问修饰符重新定义clone方法**（Object子类只能调用受保护的clone方法克隆它自己，为此必须重新定义clone方法，并将它声明为public，这才能让所有的方法克隆对象。）

​		在这里，Cloneable接口的出现与接口的正常使用没有任何关系。尤其是，它并没有指定clone方法，这个方法是从Object类继承而来的。接口在这里只是作为一个标记，表明类设计者知道要进行克隆处理。如果一个对象需要克隆，而没有实现Cloneable接口，就会产生一个已检睑异常(checked exception)。

​		为了实现深拷贝，必须克隆所有可变的实例域：

```java
class Employee implements Cloneable
{
    ...
    public Employee clone() throws CloneNotSupportedException
    {
        Employee cloned=(Employee)super.clone();
        cloned.hireDay=(Date)hireDay.clone();
        return cloned;
    }
}
```

​		只要在clone中含有没有实现Cloneable接口的对象，Object类的clone方法就会抛出一个CloneNot-SupportException异常。当然，Employee和Date类都实现了Cloneable接口，因此不会抛出异常。但是编译器并不知道这些情况，因此需要声明异常.

​		必须谨慎地实现子类的克隆。例如，一旦为Employee类定义了clone方法，任何人都可以利用它克隆Manager对象。Employee的克隆方法能够完成这项重任吗?这将取决于Manager类中包含哪些域。在前面列举的示例中，由于bonus城属于基本类型，所以不会出现任何问题。但是，在Manager类中有可能存在一些需要深拷贝的域，或者包含一些没有实现Cloneable接口的域。没有人能够保证子类实现的clone一定正确。鉴于这个原因，应该将Object类中的clone方法声明为protected。但是，如果想让用户调用clone方法，就不能这样做。

> 注释:所有的数组类型均包含一个clone方法，这个方法被设为public，而不是protected。可以利用这个方法创建一个包含所有数据元素拷贝的一个新数组。
>
> ```java
> int[] luckyNumbers={2,3,4,5};
> int[] cloned=(int[]) luckyNumbers.clone();
> cloned[0]=10;//luckyNumber[0]不会被更改
> ```

# 五、泛型设计程序

## 1.为什么要使用泛型程序设计

​		泛型程序设计意味着编写的代码可以被很多不同类型的对象所重用。例如，我们并不希望为聚集String和File对象分别设计不同的类。实际上，也不需要这样做，因为一个ArrayList类可以聚集任何类型的对象。这是一个泛型程序设计的实例。

## 2.简单泛型类的定义

​		一个泛型类就是具有一个或多个类型变量的类。对于这个类来说，我们只关注泛型，而不会为数据存储的细节烦恼。下面是Pair类的代码:

```java
public class Pair<T>
{
    public Pair(){first=null;second=null;}
	public Pair(T first,T second) 
    {this.first = first;this.second = second;}
	
    public T getFirst() { return first; }
	public T getSecond() { return second;}
	
    public void setFirst(T newValue) { first = newValue; }
    public void setSecond(T newValue) { second = newValue; }
	
    private T first;
	private T second;
}
```

​		Pair类引入了一个类型变量T，用尖括号(<>）括起来，并放在类名的后面。泛型类可以有多个类型变量。例如，可以定义Pair类，其中第一个域和第二个域使用不同的类型:

```java
public class Pair<T,U>{...}
```

## 3.泛型方法

​		前面已经介绍了如何定义一个泛型类。实际上，还可以定义一个带有类型参数的简单方法。

```java
class ArrayAlg
{
    public static <T>T getMiddle(T[] a)
    {
        return a[a.length/2];
    }
}
```

​		**这个方法是在普通类中定义的，而不是在泛型类中定义的。然而，这是一个泛型方法，可以从尖括号和类型变量看出这一点。**注意，类型变量放在修饰符（这里是public static）的后面，返回类型的前面。

​		泛型方法可以定义在普通类中，也可以定义在泛型类中。

​		当调用一个泛型方法时，在方法名前的尖括号中放入具体的类型:

```java
String[] names={"A","B","C"};
String middle=ArrayAlg.<String>getMiddle(names);
```

## 4.类型变量的限定

​		有时，类或方法需要对类型变量加以约束。下面是一个典型的例子。我们要计算数组中的最小元素:

```java
class ArrayAlg
{
    public static <T> T min(T[] a)
    {
        if(a==null || a.length==0) return null;
        T smallest=a[0];
        for(int i=1;i<a.length;i++)
        {
            if(smallest.compareTo(a[i]>0)) smallest=a[i];
        }
        return smallest;
    }
}
```

​		但是，这里有一个问题。请看一下min方法的代码内部。变量smallest类型为T，这意味着它可以是任何一个类的对象。怎么才能确信T所属的类有compareTo方法呢?

解决这个问题的方案是将T限制为实现了Comparable接口（只含一个方法compareTo的标准接口）的类。可以通过对类型变量T设置限定（bound）实现这一点:

```java
public static <T extends Comparable> T min(T[] a)...
```

​		选择关键字extends而不是implements的原因是更接近子类的概念，并且Java的设计者也不打算在语言中再添加一个新的关键字（如sub)。

​		一个类型变量或通配符可以有多个限定，例如：

```java
T extends Comparable & Serializable
```

​		在Java的继承中，可以根据需要拥有多个接口超类型，**但限定中至多有一个类。如果用一个类作为限定，它必须是限定列表中的第一个。**

## 5.泛型代码和虚拟机

​		无论何时定义一个泛型类型，都自动提供了一个相应的原始类型(raw type)。原始类型的名字就是删去类型参数后的泛型类型名。擦除(erased)类型变量，并替换为限定类型（无限定的变量用Object)。

​		例如，Pair<T>的原始类型如下所示：

```java
public class Pair
{
    public Pair(){first=null;second=null;}
	public Pair(Object first,Object second) 
    {this.first = first;this.second = second;}
	
    public Object getFirst() { return first; }
	public Object getSecond() { return second;}
	
    public void setFirst(Object newValue) { first = newValue; }
    public void setSecond(Object newValue) { second = newValue; }
	
    private Object first;
	private Object second;
}
```

### （1）翻译泛型表达式

​		当程序调用泛型方法时，如果擦除返回类型，编译器将插入强制类型转换。例如，下面这个语句序列：

```java
Pair<Employee> buddies = . . .;
Employee buddy = buddies.getFirst();
```

​		擦除getFirst的返回类型后将返回Object类型。编译器自动插入Employee的强制类型转换。也就是说，编译器把这个方法调用翻译为两条虚拟机指令:

- 对原始方法Pair.getFirst的调用。
- 将返回的Object类型强制转换为Employee类型。

​		当存取一个泛型域时也要插入强制类型转换。

```java
Employee buddy = buddies.first;
```

### （2）翻译泛型方法

​		类型擦除也会出现在泛型方法中。程序员通常认为下述的泛型方法是一个完整的方法族，而擦除类型后就只剩下一个方法。

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

​		假设DateInterval方法也覆盖了gerSecond方法：

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

## 6.约束与局限性

### （1）不能用基本类型实例化类型参数

​		不能用类型参数代替基本类型。因此，没有Pair<double>，只有Pair<Double>。当然，其原因是类型擦除。擦除之后，Pair类含有Object类型的域，而Object不能存储double值。

### （2）运行时类型查询只适用于原始类型

​		虚拟机中的对象总有一个特定的非泛型类型。因此，所有的类型查询只产生原始类型。例如：`if(a instanceof Pair<String>)`，实际上仅仅测试a是否是任意类型的一个Pair。下面的测试同样为真：`if(a instanceof Part<T>)`或强制类型转换：

```java
Pair<String> p=(Pair<String>) a;//WARNING--can only test that a is a Pair
```

​		同样的道理，`a.getClass()`返回的是`Pair.class`。

### （3）不能抛出也不能捕获泛型类实例

​		不能抛出也不能捕获泛型类的对象。事实上，泛型类连扩展Throwable都不合法。例如，下面的定义将不会通过编译:

```java
public class Problem<T> extends Exception{...}//error
```

### （4）参数化类型的数组不合法

​		不能声明参数化类型的数组：

```java
Pair<String>[] table = new Pair<String>[10];//error
```

​		擦除之后，table的类型是Pair[],数组可以记住它的元素类型，若试图存入一个错误类型的元素，就会抛出一个ArrayStoreException异常：

```java
Object[] obj=table;
obj[0]=="Hello";//error--component type is Pair
obj[0]=new Pair<Employee>();//error,可以通过数组存储的检测，但仍然会导致类型错误    
```

> 如果需要收集参数化类型的对象，最好直接使用ArrayList<Pair<String>>

### （5）不能实例化类型变量

​		不能使用像new T(...)，new T[..]或T.class这样的表达式中的类型变量。例如，下面的Pair<T>构造器就是非法的:

```java
public Pair()
{
    first=new T();
    second=new T();
}//error
```

### （6）泛型类的静态上下文中类型变量无效

​		不能在静态域或方法中引用类型变量。例如，下列高招将无法施展:

![image-20210412113445530](C:\Users\Ursnus\AppData\Roaming\Typora\typora-user-images\image-20210412113445530.png)

​		如果这个程序能够运行，就可以声明一个Singleton<Random>共享随机数生成器，声明一个Singleton<JFileChooser>共享文件选择器对话框。但是，这个程序无法工作。类型擦除之后，只剩下Singleton类，它只包含一个singleInstance域。因此，禁止使用带有类型变量的静态域和方法。

### （7）注意擦除后的冲突

​		假定像下面这样将equals方法添加到Pair类中：

```java
public class Pair<T>
{
	public boolean equals(T value) 
    {return first.equals(value)& second.equals(value);}
}
```

​		考虑一个Pair<String>。从概念上讲，它有两个equals方法:

```java
boolean equals(String);//定义于Pair<T>
boolean equals(Object);//继承自Object
```

​		但是方法擦除`boolean equals(T)`就是`boolean equals(Object)`，与继承自Object的equals方法发生冲突。

## 7.泛型类型的继承规则

​		直观的情况开始。考虑一个类和一个子类，如Employee和Manager。Pair<Manager>是Pair<Employee>的一个子类吗?答案是“不是”。

![image-20210412114608145](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210412114608145.png)

​		这一限制看起来过于严格，但对于类型安全非常必要。假设允许将Pair<Manager>转换为Pair<Employee>。考虑下面代码:

```java
pairdManager> managerBuddies = new PaircManager>(ceo，cfo);
Pair<Employee> employeeBuddies = managerBuddies;
// illegal，but suppose it wasn't
employeeBuddies.setFirst(1owlyEmployee);
```

​		显然，最后一句是合法的。但是employeeBuddies和managerBuddies引用了同样的对象。现在将CFO和一个普通员工组成一对，这对于Pair<Manager>来说应该是不可能的。

​		**必须注意泛型与Java数组之间的重要区别。**可以将一个Manager[]数组赋给一个类型为Employee[]的变量:

```java
Manager[] managerBuddies = { ceo，cfo };
Employee[] employeeBuddies = managerBuddies;// OK
```


​		然而，**数组带有特别的保护**。如果试图将一个低级别的雇员存储到employeeBuddies[0],虚拟机将会抛出ArrayStoreException异常。

​		最后，泛型类可以扩展或实现其他的泛型类。例如ArrayList<T>可以实现List<T>接口，这意味着ArrayList<Manager>可以被转换为List<Manager>.

![image-20210412115941345](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210412115941345.png)

## 8.通配符类型

​		`Pair<? extends Employee>`表示任何泛型Pair类型，它的类型参数是Employee的子类，如Pair<Manager>.

​		假设编写一个打印雇员的方法：

```java
public static void printBuddies(Pair<Employee> p)
{
    Employee first=p.getFirst();
    Employee second=p.getSecond();
    System.out.print1n(first.getName() + " and " + second.getName( ) + " are buddies.";
}
```

​		正如前面讲到的，不能将Pair<Manager>传递给这个方法，这一点很受限制。解决的方法很简单:使用通配符类型:

```java
public static void printBuddies(Pair<? extends Employee> p)
```

​		![image-20210412120315343](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210412120315343.png)

​		使用通配符会通过Pair<? extends Employee>的引用破坏Pair<Manager>吗?

```java
Pair<Manager> managerBuddies = new PairdManager>(ceo,cfo);

Pair<? extends Employee> wildcardBuddies = managerBuddies; //oK
wildcardBuddies.setFirst(lowlyEmployee); // compile-time error
```

​		这可能不会引起破坏。对setFirst的调用有一个类型错误。要了解其中的缘由，请仔细看一看类型Pair<? extends Employee>。其方法似乎是这样的:

```
? extends Employee getFirst()
void setFirst(? extends Employee)
```

​		**这样将不可能调用setFirst方法。编译器只知道需要某个Employee的子类型，但不知道具体是什么类型。它拒绝传递任何特定的类型。**毕竟，?不能用来匹配。
​		使用getFirst就不存在这个问题:将getFirst的返回值赋给一个Employee的引用完全合法。

​		这就是引入有限定的通配符的关键之处。现在已经有办法区分安全的访问器方法和不安全的更改器方法了。

### （1）通配符的超类型限定

​		通配符限定与类型变量限定十分类似，但是，还有一个附加的能力，即可以指定一个超类型限定，如下所示:`? super Manager`.**这个通配符限制为Manager的所有超类型。**

​		带有超类型限定的通配符的行为与前面介绍的相反。**可以为方法提供参数，但不能使用返回值。**例如，Pair<? super Manager>有方法：

```java
void setFirst(? super Manager)
? super Manager getFirst()
```

​		**编译器不知道setFirst方法的确切类型，但是可以用任意Manager对象（或子类型，例如，Executive)调用它，而不能用Employee对象调用。**然而，如果调用getFirst，返回的对象类型就不会得到保证。只能把它赋给一个Object。

​		**直观地讲，带有超类型限定的通配符可以向泛型对象写入，带有子类型限定的通配符可以从泛型对象读取。**

### （2）无限定通配符

​		还可以使用无限定的通配符，例如，`Pair<?>`。初看起来，这好像与原始的Pair类型一样。实际上，有很大的不同。类型`Pair<?>`有方法如:

```java
? getFirst();
void setFirst(?)
```

​		getFirst的返回值只能赋给一个Object。setFirst方法不能被调用，甚至不能用Object调用。Pair<?>和Pair本质的不同在于:**可以用任意Object对象调用原始的Pair类的setObject方法。**

​		下面这个方法将用来测试一个pair是否包含了指定的对象，它不需要实际的类型：

```java
public static boolean hasNu11s(Pair<?> p)
{
	return p.getFirst() ==null || p.getSecond( ) == nu11;
}
```

​		通过将contains转换成泛型方法，可以避免使用通配符类型:

```java
public static <T> boolean hasNulls(Pair<T> p)
```

​		但是带有通配符的版本可读性更强。

### （3）通配符捕获

​		编写一个交换一个pair元素的方法：

```java
public static void swap(Pair<?> p)
```

​		通配符不是类型变量，因此，不能在编写代码中使用“?”作为一种类型。也就是说，下述代码是非法的:

```java
? t=p.getFirst();//error
p.setFirst(p.getSecond());
p.setSecond(t);
```

​		这是一个问题，因为在交换的时候必须临时保存第一个元素。幸运的是，这个问题有一个有趣的解决方案。我们可以写一个辅助方法swapHelper，如下所示::

```java
public static <T> void swapHelper(Pair<T> p)
{
    T t=p.getFirst();
	p.setFirst(p.getSecond());
	p.setSecond(t);
}
```

​		注意，swapHelper是一个泛型方法，而swap不是，它具有固定的Pair<?>类型的参数。

​		现在可以由swap调用swapHelper :

```java
public static void swap(Pair<?> p){ swapHelper(p);}
```

​		在这种情况下，swapHelper方法的参数T捕获通配符。它不知道是哪种类型的通配符，但是，这是一个明确的类型，并且<T>swapHelper的定义只有在T指出类型时才有明确的含义。

​		当然，在这种情况下，并不是一定要使用通配符。我们已经直接实现了没有通配符的泛型方法<T>void swap(Pair<T> p)。然而，下面看一个通配符类型出现在计算中间的示例:

```java
public static void maxminBonus(Manager[] a,Pair<? super Manager> result)
{
	minmaxBonus(a,result);
	PairA1g.swapHe1per(result);// OK--swapHelper captures wildcard type
}
```

​		在这里，通配符捕获机制是不可避免的。

# 六、集合

## 1.集合接口

### （1）将集合的接口与实现分离

​		一个队列接口的最小形式可能类似下面这样：

```java
interface Queue<E>
{
    void add(E element);
    E remove();
    int size();
}
```

​		这个接口并没有说明队列是如何实现的。队列通常有两种实现方式:一种是使用循环数组﹔另一种是使用链表。

​		每一个实现都可以通过一个实现了Queue接口的类表示。

![image-20210412194641155](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210412194641155.png)

​		当在程序中使用队列时，一旦构建了集合就不需要知道究竟使用了哪种实现。因此，**只有在构建集合对象时，使用具体的类才有意义**。可以使用接口类型存放集合的引用。

```java
Queue<Customer> expressLane = new CircularArrayQueue<Customer>(100);
expressLane.add(new Customer("Harry"));
```

​		利用这种方式，一旦改变了想法，可以轻松地使用另外一种不同的实现。只需要对程序的一个地方做出修改，即调用构造器的地方。如果最终觉得LinkedListQueue是个更好的选择，就将代码修改为:

```java
Queue<Customer> expressLane=new LinkedistQueue<Customer>();
expressLane.add(new Customer("Harry"));
```

### （2）Java类库中的集合接口和迭代器接口

​		在Java类库中，集合类的基本接口是Collection接口。

​		这个接口有两个基本方法：

```java
public interface Collection<E>
{
	boolean add(E element);
    Iterator<E> iterator();
}
```

​		add方法用于向集合中添加元素。如果添加元素确实改变了集合就返回true，如果集合没有发生变化就返回false。例如，如果试图向set中添加一个对象，而这个对象在集中已经存在，这个添加请求就没有实效，因为集中不允许有重复的对象。

​		iterator方法用于返回一个实现了lterator接口的对象。可以使用这个迭代器对象依次访问集合中的元素。

#### ①迭代器

​		Iterator接口包含3个方法：

```java
public interface Iterator<E>
{
    E next();
	boolean hasNext();
    void remove();
}
```

​		通过反复调用next方法，可以逐个访问集合中的每个元素。但是，如果到达了集合的末尾，next方法将抛出一个NoSuchElementException。因此，需要在调用next之前调用hasNext方法。如果迭代器对象还有多个供访问的元素，这个方法就返回true**。如果想要查看集合中的所有元素，就请求一个迭代器，并在hasNext返回true时反复地调用next方法。**

​		编译器简单地将“for each”循环翻译为带有迭代器的循环，“for each”循环可以与任何实现了Iterable接口的对象一起工作，这个接口只包含一个方法：

```java
public interface Iterable<E>
{
    Iterator<E> iterator();
}
```

​		Collection接口扩展了lterable接口。因此，对于标准类库中的任何集合都可以使用“foreach”循环。

​		如果对ArrayList进行迭代，迭代器将从索引0开始，每迭代一次，索引值加1。然而，如果访问HashSet中的元素，每个元素将会按照某种随机的次序出现。虽然可以确定在迭代过程中能够遍历到集合中的所有元素，但却无法预知元素被访问的次序。这对于计算总和或统计符合某个条件的元素个数这类与顺序无关的操作来说，并不是

​		Java集合类库中的迭代器与其他类库中的迭代器在概念上有着重要的区别。在传统的集合类库中，例如，C++的标准模版库，迭代器是根据数组索引建模的。如果给定这样一个迭代器，就可以查看指定位置上的元素，就像知道数组索引i就可以查看数组元素a[i]一样。不需要查找元素，就可以将迭代器向前移动一个位置。这与不需要执行查找操作就可以通过i++将数组索引向前移动一样。但是，Java迭代器并不是这样操作的。查找操作与位置变更是紧密相连的。查找一个元素的惟一方法是调用next，而在执行查找操作的同时，迭代器的位置随之向前移动。

​		**因此，应该将Java迭代器认为是位于两个元素之间。当调用next时，迭代器就越过下一个元素，并返回刚刚越过的那个元素的引用。**

![image-20210412202422196](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210412202422196.png)

#### ②删除元素

​		**Iterator接口的remove方法将会删除上次调用next方法时返回的元素**。在大多数情况下，在决定删除某个元素之前应该先看一下这个元素是很具有实际意义的。然而，**如果想要删除指定位置上的元素，仍然需要越过这个元素**。下面是如何删除字符串集合中第一个元素的方法:

```java
Iterator<String> it = c.iterator();
it.next(); // skip over the first element
it.remove();// now remove it
```

​		更重要的是，对next方法和remove方法的调用具有互相依赖性。如果调用remove之前没有调用next将是不合法的。如果这样做，将会抛出一个IllegalStateException异常。

​		如果想删除两个相邻的元素，不能直接地这样调用:

```java
it.remove();
it.remove();//error
```

​		必须先调用next越过将要删除的元素：

```java
it.next();
it.remove();
it.next();
it.remove();//ok
```

## 2.具体的集合

​		以下集合中，除了以Map结尾的类实现了Map接口以外，其他类都实现了Collection接口。

![image-20210412203319239](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210412203319239.png)

### （1）链表

​		有很多示例已经使用了数组以及动态的ArrayList类。然而,数组和数组列表都有一个重大的缺陷。这就是从数组的中间位置删除一个元素要付出很大的代价，其原因是数组中处于被删除元素之后的所有元素都要向数组的前端移动（，在数组中间的位置上插入一个元素也是如此。
​		另外一个数据结构—一链表( linked list)解决了这个问题。尽管数组在连续的存储位置上存放对象引用，但链表却将每个对象存放在独立的结点中。每个结点还存放着序列中下一个结点的引用。**在Java程序设计语言中，所有链表实际上都是双向链接的**(doubly linked)——即每个结点还存放着指向前驱结点的引用。

​		在下面的代码示例中，先添加3个元素，然后再将第2个元素删除:

```java
List<String> staff = new LinkedList<String(); // Linkediist implements List
staff.add("Amy");
staff.add("Bob");
staff.add("Car1");
Iterator iter = staff.iterator();
String first = iter.next(); // visit first element
String second = iter.next();// visit second element
iter.remove(); // remove last visited element
```

![image-20210412204344755](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210412204344755.png)

​		但是，链表与泛型集合之间有一个重要的区别。链表是一个有序集合,每个对象的位置十分重要。LinkedList.add方法将对象添加到链表的尾部。但是，常常需要将元素添加到链表的中间。由于迭代器是描述集合中位置的，所以这种依赖于位置的add方法将由迭代器负责。**只有对自然有序的集合使用迭代器添加元素才有实际意义**。例如Set类型，其中的元素完全无序。因此，在Iterator接口中就没有add方法。相反地，集合类库提供了子接口ListIterator，其中包含add方法:

```java
interface ListIterator<E> extends Iterator<E>
{
	void add(E element);
    ···
}
```

​		与Collection.add不同，这个方法不返回boolean类型的值，它假定添加操作总会改变链表。另外，ListIterator接口有两个方法，可以用来反向遍历链表。

```java
E previous()
boolean hasPrevious()
```

​		set方法用一个新元素取代调用next或previous方法返回的上一个元素。例如，下面的代码将用一个新值取代链表的第一个元素:

```java
ListIterator<String> iter = list.listIterator();
String oldvalue = iter.next();// returns first element
iter.set(newValue); // sets first element to newValue
```

​		LinkedList类提供了一个用来访问某个特定元素的get方法:

```java
LinkedList<String> list = . . .;
String obj = list.get(n);
```

​		绝对不应该使用这种让人误解的随机访问方法来遍历链表。下面这段代码的效率极低:

```java
for (int i = 0; i < list.size(); i++)
	do something with list.get(i);
```

​		最后需要说一下，如果有一个整数索引n,`list.listIterator(n)`将返回一个迭代器**，这个迭代器指向索引为n的元素前面的位置**。也就是说，**调用next与调用list.get(n)会产生同一个元素**，只是获得这个迭代器的效率比较低。

### （2）数组列表

​		集合类库提供了一种大家熟悉的ArrayList类，这个类也实现了List接口。ArrayList封装了一个动态再分配的对象数组。

### （3）散列集

​		如果不在意元素的顺序，可以有几种能够快速查找元素的数据结构。其缺点是无法控制元素出现的次序。它们将按照有利于其操作目的的原则组织数据。

​		有一种众所周知的数据结构，可以快速地查找所需要的对象，这就是散列表(hash table)。散列表为每个对象计算一个整数，称为散列码(hash code)。散列码是由对象的实例域产生的一个整数。更准确地说，具有不同数据域的对象将产生不同的散列码。

​		如果自定义类，就要负责实现这个类的hashCode方法。注意，自己实现的hashCode方法应该与equals方法兼容，即如果a.equals(b)为true，a与b必须具有相同的散列码。

​		散列表可以用于实现几个重要的数据结构。其中最简单的是set类型。set是没有重复元素的元素集合。set的add方法首先在集中查找要添加的对象，如果不存在，就将这个对象添加进去。

​		Java集合类库提供了一个HashSet类，它实现了基于散列表的集。可以用add方法添加元素。contains方法已经被重新定义，用来快速地查看是否某个元素已经出现在集中。它只在某个桶中查找元素，而不必查看集合中的所有元素。

​		散列集迭代器将依次访问所有的桶。由于散列将元素分散在表的各个位置上，所以访问它们的顺序几乎是随机的。只有不关心集合中元素的顺序时才应该使目HachSet。

### （4）树集

​		TreeSet类与散列集十分类似，不过，它比散列集有所改进。树集是一个有序集合。可以以任意顺序将元素插入到集合中。在对集合进行遍历时，每个值将自动地按照排序后的顺序呈现。

​		正如TreeSet类名所示，排序是用树结构完成的（目前使用的是红黑树）。

​		每次将一个元素添加到树中时，都被放置在正确的排序位置上。因此，迭代器总是以排好序的顺序访问每个元素。

​		将一个元素添加到树中要比添加到散列表中慢，但是，与将元素添加到数组或链表的正确位置上相比还是快很多的。     

### （5）对象的比较

​		TreeSet如何知道希望元素怎样排列呢?在默认情况时，树集假定插入的元素实现了Comparable接口。这个接口定义了一个方法:

```java
public interface Comparable<T>
{
	int compareTo(T other);
}
```

​		如果a与b相等，调用a.compareTo(b)一定返回0:如果排序后a位于b之前，则返回负值，如果a位于b之后，则返回正值。

​		如果要插入自定义的对象，就必须通过实现Comparable接口自定义排列顺序。在Object类中，没有提供任何compareTo接口的默认实现。

​		例如，下面的代码展示了如何用部件编号对Item对象进行排序:

```java
class Item implements Comparable<Item>
{
	public int compareTo(Item other)
    {
		return partNumber - other.partNumber;
    }
}
```

​		如果对两个正整数进行比较，就像上面示例中的部件编号，就可以直接地返回它们的差,只有整数在一个足够小的范围内，才可以使用这个技巧,防止溢出。

​		然而，使用Comparable接口定义排列排序显然有其局限性。对于-一个给定的类，只能够实现这个接口一次。如果在一个集合中需要按照部件编号进行排序，在另一个集合中却要按照描述信息进行排序，该怎么办呢?另外，如果需要对一个类的对象进行排序，而这个类的创建者又没有费心实现Comparable接口，又该怎么办呢?

​		在这种情况下，可以通过将Comparator对象传递给TreeSet构造器来告诉树集使用不同的比较方法。Comparator接口声明了一个带有两个显式参数的compare方法:

```java
pub1ic interface Comparator<T>
{
	int compare(T a,T b);
}
```

​		与compareTo方法一样，如果a位于b之前compare方法则返回负值，如果a和b相等则返回0,否则返回正值。

​		如果按照描述信息进行排序，就直接定义一个实现Comparator接口的类:

```java
class ItemComparator implements Comparator<Item>
public int compare(Item a，Item b)
{
	String descrA m a.getDescription();
	String descrB m b.getDescription();
    return descrA.compareTo(descrB);
}
```

​		然后将这个类的对象传递给树集的构造器:

```java
ItemComparator comp = new ItemComparator();
SortedSet<Item> sortByDescription = new TreeSet<Item>(comp);
```

​		如果构造了一棵带比较器的树，就可以在需要比较两个元素时使用这个对象。

​		注意，这个比较器没有任何数据。它只是比较方法的持有器。有时将这种对象称为函数对象( function object)。函数对象通常被定义为“瞬时的”，即匿名内部类的实例.

### （6）队列与双端队列

​		有两个端头的队列，即双端队列，可以让人们有效地在头部和尾部同时添加或删除元素。不支持在队列中间添加元素。在Java SE 6中引入了Deque接口，并由ArrayDeque和LinkedList类实现。

### （7）优先级队列

​		优先级队列(priority queue)中的元素可以按照任意的顺序插入，却总是按照排序的顺序进行检索。也就是说，**无论何时调用remove方法，总会获得当前优先级队列中最小的元素**。然而，优先级队列并没有对所有的元素进行排序。如果用迭代的方式处理这些元素，并不需要对它们进行排序。优先级队列使用了一个优雅且高效的数据结构，称为堆(heap)。堆是一个可以自我调整的二叉树，对树执行添加（add）和删除（remore)操作，可以让最小的元素移动到根，而不必花费时间对元素进行排序。

### （8）映射表

​		集是一个集合，它可以快速地查找现有的元素。但是，要查看一个元素，需要有要查找元素的精确副本。这不是一种非常通用的查找方式。通常，我们知道某些键的信息，并想要查找与之对应的元素。映射表（map）数据结构就是为此设计的。映射表用来存放键/值对。如果提供了键，就能够查找到值。例如，有一张关于员工信息的记录表，键为员工ID，值为Employee对象。

​		Java类库为映射表提供了两个通用的实现:HashMap和TreeMap。这两个类都实现了Map接口。

​		**散列映射表对键进行散列，树映射表用键的整体顺序对元素进行排序，并将其组织成搜索树。**散列或比较函数只能作用于键。与键关联的值不能进行散列或比较。

​		下列代码将为存储的员工信息建立一个散列映射表:

```java
Map<String，Employee> staff = new HashMap<String，Employee>();// HashMap implements Map
Employee harry = new Employee("Harry Hacker");
staff.put("987-98-9996"，harry);

String s = "987-98-9996";
e = staff.get(s); // gets harry
```

​		如果在映肘表中没有与给定键对应的信息，get将返回null。

​		键必须是唯一的。不能对同一个键存放两个值。如果对同一个键两次调用put方法，第二个值就会取代第一个值。实际上，put将返回用这个键参数存储的上一个值。

​		remove方法用于从映射表中删除给定键对应的元素。size方法用于返回映射表中的元素数。

​		集合框架并没有将映射表本身视为一个集合（其他的数据结构框架则将映射表视为对(pairs)的集合，或者视为用键作为索引的值的集合)。然而，可以获得映射表的视图，这是一组实现了Collection接口对象，或者它的子接口的视图。

​		有3个视图，它们分别是:键集、值集合（不是集）和键/值对集。键与键/值对形成了一个集，这是因为在映射表中一个键只能有一个副本。下列方法将返回这3个视图（条目集entrySet的元素是静态内部类Map.Entry的对象)。

```java
Set<K> keySet()
Co1lection<K> values()
Set<Map.Entry<K, V> entrySet()
```

​		注意，keySet既不是HashSet，也不是TreeSet，而是实现了Set接口的某个其他类的对象。Set接口扩展了Collection接口。因此，可以与使用任何集合一样使用keySet。

​		**如果想要同时查看键与值，就可以通过枚举各个条目（entries）查看，以避免对值进行查找。**可以使用下面这段代码框架:

```java
for (Map.Entry<String,Employee> entry : staff.entrySet())
{
	String key = entry.getKey();
    Employee value = entry.getValue();
    //do something with key, value
}
```

### （9）专用集与映射表类

#### ①弱散列映射表

​		设计WeakHashMap类是为了解决一个有趣的问题。如果有一个值，对应的键已经不再使用了，将会出现什么情况呢﹖假定对某个键的最后一次引用已经消亡,不再有任何途径引用这个值的对象了。但是，由于在程序中的任何部分没有再出现这个键，所以，这个键/值对无法从映射表中删除。为什么垃圾回收器不能够删除它呢?难道删除无用的对象不是垃圾回收器的工作吗?

​		遗憾的是，事情没有这样简单。垃圾回收器跟踪活动的对象。只要映射表对象是活动的，其中的所有桶也是活动的，它们不能被回收。因此，需要由程序负责从长期存活的映射表中删除那些无用的值。或者使用WeakHashMap完成这件事情。当对键的惟一引用来自散列表条目时，这一数据结构将与垃圾回收器协同工作一起删除键/值对。

​		下面是这种机制的内部运行情况。WeakHashMap使用弱引用(weak references)保存键。WeakReference对象将引用保存到另外一个对象中，在这里，就是散列表键。对于这种类型的对象，垃圾回收器用一种特有的方式进行处理。**通常，如果垃圾回收器发现某个特定的对象已经没有他人引用了，就将其回收。然而，如果某个对象只能由WeakReference引用，垃圾回收器仍然回收它，但要将引用这个对象的弱引用放入队列中。WeakHashMap将周期性地检查队列，以便找出新添加的弱引用。**一个弱引用进入队列意味着这个键不再被他人使用，并且已经被收集起来。于是，WeakHashMap将删除对应的条目。

#### ②链接散列集和链接映射表

​		Java SE 1.4增加了两个类:LinkedHashSet和LinkedHashMap，用来记住插入元素项的顺序。这样就可以避免在散列表中的项从表面上看是随机排列的。当条目插入到表中时，就会并入到双向链表中。

​		链接散列映射表将用访问顺序，而不是插入顺序，对映射表条目进行迭代。每次调用get或put，受到影响的条目将从当前的位置删除，并放到条目链表的尾部（只有条目在链表中的位置会受影响，而散列表中的桶不会受影响。一个条目总位于与键散列码对应的桶中)。要项构造这样一个的散列映射表，请调用

```java
LinkedHashMap<K,V>(initialCapacity,loadFactor,true)
```

​		访问顺序对于实现高速缓存的“最近最少使用”原则十分重要。例如，可能希望将访问频率高的元素放在内存中，而访问频率低的元素则从数据库中读取。当在表中找不到元素项且表又已经满时，可以将迭代器加入到表中，并将枚举的前几个元素删除掉。这些是近期最少使用的几个元素。

​		甚至可以让这一过程自动化。即构造一个LinkedHashMap的子类，然后覆盖下面这个方法:

	protected boolean removeEldestEntry(Map.Entry<K, v> eldest)

​		每当方法返回true时，就添加一个新条目，从而导致删除eldest条目。例如，下面的高速缓存可以存放100个元素:

```java
Map<K,V>cache = newLinkedHashMap<K，V>(128,0.75F,true)
{
	protected boolean removeE1destEntry(NMap.Entry<x，V eldest)
	{
		return size() > 100;
	}
};
```

#### ③枚举集与枚举映射表

​		EnumSet是一个枚举类型元素集的高效实现。由于枚举类型只有有限个实例，所以EnumSet内部用位序列实现。如果对应的值在集中，则相应的位被置为1。

​		EnumSet类没有公共的构造器。可以使用静态工厂方法构造这个集:

```kava
enum weekday { MONDAY，TUESDAY，WEDNESDAY，THURSDAY，FRIDAY，SATURDAY，SUNDAY };
EnumSet<weekdayo always = EnumSet.a110f(weekday.class);
EnumSet<weekday never = EnumSet.noneOf(weekday.class);
EnumSet<weekday> workday = EnumSet.range(Weekday.MONDAY，weekday.FRIDAY);
EnumSetcweekday> mwf = EnumSet.of(Weekday.NONDAY，weekday.WEDNESDAY，weekday.FRIDAY);
```

​		可以使用Set接口的常用方法来修改EnumSet。

​		EnumMap是一个键类型为枚举类型的映射表。它可以直接且高效地用一个值数组实现。在使用时，需要在构造器中指定键类型:

```java
EnumMap<weekday，Employee> personInCharge = new EnumMap<Weekday，Employee>(weekday.class);
```

#### ④标识散列映射表

​		Java SE 1.4还为另外一个特殊目的增加了另一个类IdentityHashMap。在这个类中，键的散列值不是用hashCode函数计算的，而是用System.identityHashCode方法计算的。**这是Object.hashCode方法根据对象的<u>内存地址</u>来计算散列码时所使用的方式。而且，在对两个对象进行比较时，IdentityHashMap类使用==，而不使用equals。**

​		也就是说，不同的键对象，即使内容相同，也被视为是不同的对象。在实现对象遍历算法(如对象序列化)时，这个类非常有用，可以用来跟踪每个对象的遍历状况。

## 3.集合框架

​		框架是一个类的集，它奠定了创建高级功能的基础。框架包含很多超类，这些超类拥有非常有用的功能、策略和机制。框架使用者创建的子类可以扩展超类的功能，而不必重新创建这些基本的机制。例如，Swing就是一种用户界面的机制。

​		Java集合类库构成了集合类的框架。它为集合的实现者定义了大量的接口和抽象类。

​		集合有两个基本的接口:Collection和Map。

​		可以使用下列方法向集合中插入元素：`boolean add(E element)`

​		但是，由于映射表保存的是键/值对，所以可以使用put方法进行插入。`v put(K key，v value)`

​		要想从集合中读取某个元素，就需要使用迭代器访问它们。也可以用get方法从映射表读取值:`v get(K key)`

​		List是一个有序集合。元素可以添加到容器中某个特定的位置。将对象放置在某个位置上可以采用两种方式:使用整数索引或使用列表迭代器。List接口定义了几个用于随机访问的方法:

```java
void add(int index，E element)
E get(int index)
void remove(int index)
```

![image-20210413201105062](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210413201105062.png)

### 1.视图与包装器

​		通过使用视图可以获得其他的实现了集合接口和映射表接口的对象。**映射表类的keySet方法就是一个这样的示例**。初看起来，好像这个方法创建了一个新集，并将映射表中的所有键都填进去，然后返回这个集。但是，情况并非如此。取而代之的是:**keySet方法返回一个实现Set接口的类对象，这个类的方法对原映射表进行操作。这种集合称为视图。**

#### ①轻量级集包装器

​		Arrays类的静态方法asList将返回一个包装了普通Java数组的List包装器。这个方法可以**将数组传递给一个期望得到列表或集合变元的方法**。例如:

```java
Card[] cardDeck = new Card[52];
List<Card> cardList = Arrays.asList(cardDeck):
```

​		返回的对象不是ArrayList。**它是一个视图对象，带有访问底层数组的get和set方法。**改变数组大小的所有方法（例如，与迭代器相关的add和remove方法）都会抛出一个UnsupportedOperationException异常。

​	从Java SE 5.0开始，asList方法声明为一个具有可变数量参数的方法。除了可以传递一个数组之外，还可以将各个元素直接传递给这个方法。例如:

```java
List<String> names = Arrays.asList("Amy"，"Bob"，"Car1");
```

​		`collections.nCopies(n，anbject)`将**返回一个实现了List接口的不可修改的对象**，并给人一种包含n个元素，每个元素都像是一个anObject的错觉。

​	例如，下面的调用将创建一个包含100个字符串的List，每个串都被设置为“DEFAULT";

```java
List<String> settings = Collections.nCopies(100，"DEFAULT");
```

​		由于字符串对象只存储了一次，所以付出的存储代价很小。这是视图技术的一种巧妙的应用。

​		`Collections.singleton(anObject)`则将返回一个视图对象。**这个对象实现了Set接口**（与产生List的ncopies方法不同)。**返回的对象实现了一个不可修改的单元素集，而不需要付出建立数据结构的开销**。singletonList方法与singletonMap方法类似。

#### ②子范围

​		可以为很多集合建立子范围视图。例如，假设有一个列表staff ，想从中取出第10个-第19个元素。可以使用subList方法来获得一个列表的子范围视图。

```java
List group2 = staff. subList(10，20);
```

​		第一个索引包含在内，第二个索引则不包含在内。这与String类的substring操作中的参数情况相同。

​		可以将任何操作应用于子范围，并且能够自动地反映整个列表的情况。例如，可以删除整个子范围:

```java
group2.clear(); // staff reduction
```

​		现在，元素自动地从staff列表中清除了，并且group2为空。

​		对于有序集和映射表，可以使用排序顺序而不是元素位置建立子范围。SortedSet接口声明了3个方法:

```java
SortedSet<E> subSet(E from，E to)
SortedSet<E> headSet(E to)
SortedSet<E> tai1Set(E from)
```

​		这些方法将返回大于等于from且小于to的所有元素子集。有序映射表也有类似的方法:

```javascript
SortedMap<K,V>subMap(K from，K to)
SortedMap<K,V>headMap(K to)
SortedMap<K,V>tai1Map(K from)
```

​		返回映射表视图，该映射表包含键落在指定范围内的所有元素。

​		Java SE 6引入的NavigableSet接口赋予子范围操作更多的控制能力。可以指定是否包括边界:

```java
NavigableSet<E> subSet(E from，boolean fromInclusive，E to，boolean toInclusive)
NavigableSet<E> headSet(E to，boolean toInclusive)
Navigable5et<E> tai1Set(E from,boolean fromInclusive)
```

#### ③不可修改的视图

​		Collections还有几个方法，用于产生集合的不可修改视图(unmodifiable views)。这些视图对现有集合增加了一个运行时的检查。如果发现试图对集合进行修改，就抛出一个异常，同时这个集合将保持未修改的状态。
可以使用下面6种方法获得不可修改视图:

```java
Collections.unmodifiableCo1lection
Collections.unmodifiablelist
Collections.unmodifiableSet
Collections.unmodifiableSortedSetCo1lections.unmodifiableMap
Collections.unmodifiableSortedMap
```

​		不可修改视图并不是集合本身不可修改。仍然可以通过集合的原始引用（在这里是staff)对集合进行修改。并且仍然可以让集合的元素调用更改器方法。

​		由于视图只是包装了接口而不是实际的集合对象，所以**视图只能访问接口中定义的方法**。例如,LinkedList类有一些非常方便的方法，addFirst和addLast，**它们都不是List接口的方法，不能通过不可修改视图进行访问。**

#### ④同步视图

​		如果由多个线程访问集合，就必须确保集不会被意外地破坏。例如，如果一个线程试图将元素添加到散列表中，同时另一个线程正在对散列表进行再散列，其结果将是灾难性的。

​		**类库的设计者使用视图机制来确保常规集合的线程安全，而不是实现线程安全的集合类**。例如，Collections类的静态synchronizedMap方法可以将任何一个映射表转换成具有同步访问方法的Map:

```java
Map<String,Employee> map = Collections.synchronizedMap(new HashMap<String,Employee>());
```

​		现在，就可以由多线程访问map对象了。像get和put这类方法都是串行操作的，即在另一个线程调用另一个方法之前，刚才的方法调用必须彻底完成。

#### ⑤被检验视图	

​		Java SE 5.0增加了一组“被检验”视图，用来对泛型类型发生问题时提供调试支持。实际上将错误类型的元素私自带到泛型集合中的问题极有可能发生。	

```java
ArrayList<String> strings = new ArrayList<String>();
ArrayList rawList = strings;// get warning only,not an error,for compatibility with legacy code
rawList.add(new Date(0); // now strings contains a Date object!
```

​		这个错误的add命令在运行时检测不到。相反，只有在稍后的另一部分代码中调用get方法，并将结果转化为String时，这个类才会抛出异常。被检验视图可以探测到这类问题。下面定义了一个安全列表:

```java
List<String> safeStrings = Collections.checkedList(strings,String.class);
```

​		视图的add方法将检测插入的对象是否属于给定的类。如果不属于给定的类，就立即抛出一个ClassCastException。这样做的好处是错误可以在正确的位置得以报告:

```java
ArrayList rawList = safeStrings;
rawlist.add(new Date()); // Checked list throws a ClassCastException
```

#### ⑥关于可选操作的说明	

​		通常，视图有一些局限性，即可能只可以读、无法改变大小、只支持删除而不支持插入，这些与映射表的键视图情况相同。如果试图进行不恰当的操作，受限制的视图就会抛出一个UnsupportedOperationException。

### （2）批操作

​		到现在为止，列举的绝大多数示例都采用迭代器遍历集合，一次遍历一个元素。然而，可以使用类库中的批操作避免频繁地使用迭代器。
​		假设希望找出两个集的交（intersection)，即两个集中共有的元素。首先，要建立一个新集，用于存放结果。

```java
Set<String> result = new HashSet<String>(a);
```

​		这里利用了这样一个事实:每一个集合有一个构造器，其参数是保存初始值的另一个集合。接着,调用retainAll方法:

```java
result.retainAll(b);
```

​		**result中保存了既在a中出现，也在b中出现的元素**。这时已经构成了交集，而且没有使用循环。

​		可以将这个思路向前推进一步，将批操作应用于视图。例如，假如有一个映射表，将员工的ID映射为员工对象，并且建立了一个将要结束聘用期的所有员工的ID集。

```java
Map<String,Employee> staffMap = ...;
Set<String> terminatedIDs = ...;
```

​		直接建立一个键集，并删除终止聘用关系的所有员工的ID即可。

```java
staffMap.keySet().removeAll(terminatedIDs);
```

​		由于键集是映射表的一个视图，所以，键与对应的员工名将会从映射表中自动地删除。

​		通过使用一个子范围视图，可以将批操作限制于子列表和子集的操作上。例如，假设希望将一个列表的前10个元素添加到另一个容器中，可以建立一个子列表用于选择前10个元素:

```java
relocated.addAll(staff.subList(0,10));
```

​		这个子范围也可以成为更改操作的对象。	

```java
staff.subList(0,10).clear();
```

### （3）集合与数组之间的转换

​		如果有一个数组需要转换为集合。Arrays.asList的包装器就可以实现这个目的。例如：

```java
String[] values = . . .;
HashSet<String> staff = new HashSet<String>(Arrays.asList(values));
```

​		反过来，将集合转换为数组就有点难了。当然，可以使用toArray方法:

```java
Object[] values = staff.toArray();
```

​		但是，**这样做的结果是产生一个对象数组**。即使知道集合中包含一个特定类型的对象，也不能使用类型转换:

```java
String[] values = (String[]) staff.toArray(); // Error!
```

​		由toArray方法返回的数组是一个Object[]数组，无法改变其类型。相反，**必须使用另外一种toArray方法，并将其设计为所希望的元素类型且长度为0的数**组。随后返回的数组将与所创建的数组一样:

```java
String[] values = (String[]) staff.toArray(new String[0]); 
```

​		如果愿意的话，可以构造一个指定大小的数组:

```java
staff.toArray(new String[staff.size()]);
```

​		**在这种情况下，没有创建任何一个新数组。**

## 4.算法

​		仔细考虑一下，为了高效地使用这个算法所需要的最小集合接口。采用get和set方法进行随机访问要比直接迭代层次高。在计算链表中最大元素的过程中已经看到，这项任务并不需要进行随机访问。直接用迭代器遍历每个元素就可以计算最大元素。因此，**可以将max方法实现为能够接收任何实现了Collection接口的对象的方法**。

![image-20210413212256785](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210413212256785.png)

​		现在就可以使用一个方法计算链表、数组列表或数组中最大元素了  

### （1）排序与混排

​		Collections类中的sort方法可以对实现了List接口的集合进行排序。

​		这个方法假定列表元素实现了Comparable接口。如果想采用其他方式对列表进行排序，可以将Comparator对象作为第二个参数传递给sort方法。

![image-20210413212713166](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210413212713166.png)

​		如果想按照降序对列表进行排序，可以使用一种非常方便的静态方法`Collections.reverseOrder()`。这个方法将返回一个比较器，比较器则返回b.eompareTo(a)。

```java
Collections.sort(staff,Collections.reverseOrder())
Collections.sort(items,Collections.reverseOrder(itemComparator))
```

​		**排序算法传递的列表必须是可修改的，不必是可改变大小的。**

- 如果列表**支持set(i,e)方法，则是可修改的。**
- 如果列表**支持add和remove方法，则是可改变大小的。**

​		Collections类有一个算法shuffle，其功能与排序刚好相反，即随机地混排列表中元素的顺序。例如:

```java
ArrayList<Card> cards = ...;
Collections.shuffle(cards);
```

​		**如果提供的列表没有实现RandomAccess接口，shuffle方法将元素复制到数组中，然后打乱数组元素的顺序，最后再将打乱顺序后的元素复制回列表。**

### （2）二分查找

​		Collections类的binarySearch方法实现了这个算法。注意，集合必须是排好序的，否则算法将返回错误的答案。要想查找某个元素，必须提供集合（这个集合要实现List接口，下面还要更加详细地介绍这个问题)以及要查找的元素。如果集合没有采用Comparable接口的compareTo方法进行排序，就还要提供一个比较器对象。

```java
i = Collections.binarySearch(c,element);
i = Collections.binarySearch(c,element,comparator);
```

​		如果binarySearch方法返回的数值大于等于0，则表示匹配对象的索引。也就是说，c.get(i)等于在这个比较顺序下的element。如果返回负值，则表示没有匹配的元素。但是，**可以利用返回值计算应该将element插入到集合的哪个位置，以保持集合的有序性**。插人的位置是:

```java
insertionPoint = -i - 1;
if (i < 0)
	c.add(-i-1,element);//将把元素插入到正确的位置上
```

