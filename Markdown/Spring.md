# 一、IOC/DI与注入对象

​		IOC 反转控制 是Spring的基础，Inversion Of Control
简单说就是创建对象由以前的程序员自己new 构造方法来调用，变成了交由Spring创建对象
​		DI 依赖注入 Dependency Inject. 简单地说就是拿到的对象的属性，已经被注入好相关值了，直接使用即可。

​		目录结构如下：

![image-20210701121053519](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/eTYbLg2PpX8fyFq.png)

## 1.Category类

```java
public class Category {
    private int id;
    private String name;

    public Category() {
    }

    public int getId() {
        return this.id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

## 2.Product类

```java
public class Product {
    private int id;
    private String name;
    private Category category;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Category getCategory() {
        return category;
    }

    public void setCategory(Category category) {
        this.category = category;
    }
}
```

## 3.applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <bean name="c" class="com.spring.pojo.Category">
        <property name="name" value="category 1" />
    </bean>
    <bean name="p" class="com.spring.pojo.Product">
        <property name="name" value="product 1" />
        <property name="category" ref="c" />
    </bean>

</beans>
```

## 4.TestSpring类

```java
public class TestSpring {
    public TestSpring() {
    }

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"applicationContext.xml"});
        Product p = (Product) context.getBean("p");

        System.out.println(p.getName());
        System.out.println(p.getCategory().getName());
        System.out.println(context);
    }
}

运行结果：
product 1
category 1
```

# 二、注解方式IOC/DI

## 1.注入对象的注解

​		相比不用注解的方式，所要修改的主要地方如下：

### （1）Product类

​		在Category对象上方加上`@Autowired`注解,也可以在setCategory方法前加上`@Autowired`。

```java
public class Product {
 
    private int id;
    private String name;
    @Autowired
    private Category category;
 
    public int getId() {
        return id;
    }
 
    public void setId(int id) {
        this.id = id;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public Category getCategory() {
        return category;
    }
 	//@Autowired
    public void setCategory(Category category) {
        this.category = category;
    }
}
```

​		除了@Autowired之外，@Resource也是常用的手段：

```java
@Resource(name="c")
private Category category;
```

### （2）applicationContext.xml:

​		添加`<context:annotation-config/>`并将注入对象的xml语句删除。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx" xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
   http://www.springframework.org/schema/beans 
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
   http://www.springframework.org/schema/aop 
   http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
   http://www.springframework.org/schema/tx 
   http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
   http://www.springframework.org/schema/context      
   http://www.springframework.org/schema/context/spring-context-3.0.xsd">
  
    <context:annotation-config/>
    <bean name="c" class="com.how2java.pojo.Category">
        <property name="name" value="category 1" />
    </bean>
    <bean name="p" class="com.how2java.pojo.Product">
        <property name="name" value="product 1" />
<!--         <property name="category" ref="c" /> -->
    </bean>
  
</beans>
```

## 2.Bean的注解

### （1）Product类与Category类

​		为Product类加上@Component注解，即表明此类是bean

```java
@Component("p")
public class Product {
}
```

​		 为Category 类加上@Component注解，即表明此类是bean

```java
@Component("c")
public class Category {
}
```

另外，因为配置从applicationContext.xml中移出来了，所以属性初始化放在属性声明上进行了。

```java
private String name="product 1";
private String name="category 1";
```

​		例如Product类：

```java
@Component("p")
public class Product {
 
    private int id;
    private String name="product 1";
     
    @Autowired
    private Category category;
 
    public int getId() {
        return id;
    }
 
    public void setId(int id) {
        this.id = id;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public Category getCategory() {
        return category;
    }
     
    public void setCategory(Category category) {
        this.category = category;
    }
}
```

### （2）applicationContext.xml

​		修改applicationContext.xml，什么都去掉，只新增：

`<context:component-scan base-package="com.spring.pojo"/>`

​		其作用是告诉Spring，bean都放在com.spring.pojo这个包下。

# 三、AOP

​		AOP 即 Aspect Oriented Program 面向切面编程。

​		首先，在面向切面编程的思想里面，把功能分为**核心业务功能**，和**辅助功能**，比如登陆，增加数据，删除数据都叫核心业务；所谓的辅助功能，比如性能统计，日志，事务管理等等。

​		辅助功能在Spring的面向切面编程AOP思想里，即被定义为**切面**，在面向切面编程AOP的思想里面，核心业务功能和切面功能分别**独立进行开发**，然后把切面功能和核心业务功能 **"编织"** 在一起，这就叫AOP。

​		目录结构如下：

![image-20210701234301568](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210701234301568.png)

## 1.LoggerAspect类

```java
public class LoggerAspect {
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("start log:" + joinPoint.getSignature().getName());
        Object object = joinPoint.proceed();
        System.out.println("end log:" + joinPoint.getSignature().getName());
        return object;
    }
}
```

## 2.ProductService类

```java
public class ProductService {
    public void doSomeService(){
        System.out.println("doSomeService");
    }
}
```

## 3.applicationContext.xml

```xml
	<!--声明业务对象-->
	<bean name="c" class="com.spring.pojo.Category">
        <property name="name" value="yyy" />
    </bean>

    <bean name="p" class="com.spring.pojo.Product">
        <property name="name" value="product1" />
        <property name="category" ref="c" />
    </bean>


    <bean name="s" class="com.spring.service.ProductService">

    </bean>
	
	<!--声明日志切面-->
    <bean id="loggerAspect" class="com.spring.aspect.LoggerAspect"/>

    <aop:config>
        <!--指定核心业务功能-->
        <aop:pointcut id="loggerCutpoint"
                      expression=
                              "execution(* com.spring.service.ProductService.*(..)) "/>
 
		<!--指定辅助功能，然后通过aop:config把业务对象与辅助功能编织在一起。-->
        <aop:aspect id="logAspect" ref="loggerAspect">
            <aop:around pointcut-ref="loggerCutpoint" method="log"/>
        </aop:aspect>
    </aop:config>

execution(* com.spring.service.ProductService.*(..))
这表示对满足如下条件的方法调用，进行切面操作：
* 返回任意类型
com.how2java.service.ProductService.* 包名以 com.how2java.service.ProductService 开头的类的任意方法
(..) 参数是任意数量和类型
```

## 4.TestSpring类

```java
public class TestSpring {
 
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext(
                new String[] { "applicationContext.xml" });
        ProductService s = (ProductService) context.getBean("s");
        s.doSomeService();
    }
}


输出结果：
start log:doSomeService
doSomeService
end log:doSomeService
```

# 四、注解方式AOP

## 1.注解配置业务类

```java
@Component("s")
public class ProductService {
    public void doSomeService(){
        System.out.println("doSomeService");
    }
}
```

## 2.注解配置切面

​		@Aspect 注解表示这是一个切面
​		@Component 表示这是一个bean,由Spring进行管理
​		@Around(value = "execution(* com.how2java.service.ProductService.*(..))") 表示对com.how2java.service.ProductService 这个类中的所有方法进行切面操作

```java
@Aspect
@Component
public class LoggerAspect { 
     
    @Around(value = "execution(* com.how2java.service.ProductService.*(..))")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("start log:" + joinPoint.getSignature().getName());
        Object object = joinPoint.proceed();
        System.out.println("end log:" + joinPoint.getSignature().getName());
        return object;
    }
}
```

## 3.applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
   http://www.springframework.org/schema/beans 
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
   http://www.springframework.org/schema/aop 
   http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
   http://www.springframework.org/schema/tx 
   http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
   http://www.springframework.org/schema/context      
   http://www.springframework.org/schema/context/spring-context-3.0.xsd">
  
	<!--扫描扫描包com.spring.aspect和com.spring.service，定位业务类和切面类-->
    <context:component-scan base-package="com.spring.aspect"/>
    <context:component-scan base-package="com.spring.service"/>
    <!--找到被注解了的切面类，进行切面配置-->
    <aop:aspectj-autoproxy/>  
   
</beans>
```

# 五、注解方式测试