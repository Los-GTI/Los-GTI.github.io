---
layout:     post
title: 混口饭吃专题之Spring详解之六AOP
subtitle: Spring系列之六
date:       2018-04-25
author:     Los-GTI
header-img: img/ms1.jpg
catalog: true
tags:
    - 混口饭吃
---


这篇继续讲Spring的核心概念：AOP，这也是Spring框架中最为核心的一个概念。

#### 1. AOP是什么？

AOP（Aspect Oriented Programming），即面向切面编程，可以说是OOP（Object Oriented Programming，面向对象编程）的补充和完善。OOP引入封装、继承、多态等概念来建立一种对象层次结构，用于模拟公共行为的一个集合。不过OOP允许开发者定义纵向的关系，但并不适合定义横向的关系，例如日志功能。日志代码往往横向地散布在所有对象层次中，而与它对应的对象的核心功能毫无关系对于其他类型的代码，如安全性、异常处理和透明的持续性也都是如此，这种散布在各处的无关的代码被称为横切（cross cutting），在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。

AOP（Aspect Oriented Programming）恰恰相反。它利用一种称为"横切"的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其命名为"Aspect"，即切面。所谓"切面"，简单说就是那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。

使用"横切"技术，AOP把软件系统分为两个部分：核心关注点和横切关注点。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，他们经常发生在核心关注点的多处，而各处基本相似，比如权限认证、日志、事物。AOP的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。
	
　　什么是切面，什么是公共模块，那么我们概念少说，直接通过一个实例来看看 AOP 到底是什么。

#### 2.需求

现在有一张表User，然后我们要在程序中实现对User的增加和删除操作。
要求：增加和删除操作都必须要开启事务，操作完成之后要提交事务。

User.java

```
public class User {
    private int uid;
    private String uname;

    public int getUid() {
        return uid;
    }

    public void setUid(int uid) {
        this.uid = uid;
    }

    public String getUname() {
        return uname;
    }

    public void setUname(String uname) {
        this.uname = uname;
    }
}
```

#### 3.解决办法1：使用静态代理

第一步：创建UserService接口

```
public interface UserService {
    //添加User
    public void addUser(User user);
    //删除User
    public void deleteUser(int uid);
}
```

第二步：创建UserService的实现类

```
public class UserServiceImpl implements UserService{
    @Override
    public void addUser(User user) {
        System.out.println("增加User");
    }

    @Override
    public void deleteUser(int uid) {
        System.out.println("删除User");
    }
}
```

第三步：创建事务类MyTransaction

```
public class MyTransaction {
    //开启事务
    public void before(){
        System.out.println("开启事务");
    }
    //提交事务
    public void after(){
        System.out.println("提交事务");
    }
}
```

第四步：创建代理类ProxyUser.java

```
public class ProxyUser implements UserService{

    //真实类
    private UserService userService;
    //事务类
    private MyTransaction myTransaction;
    //使用构造函数实例化
    public ProxyUser(UserService userService, MyTransaction myTransaction) {
        this.userService = userService;
        this.myTransaction = myTransaction;
    }
    public void addUser(User user) {
        myTransaction.before();
        userService.addUser(user);
        myTransaction.after();
    }

    public void deleteUser(int uid) {
        myTransaction.before();
        userService.deleteUser(uid);
        myTransaction.after();
    }
}
```

测试：
```
public class testAop {
    public static void main(String[] args) {
        MyTransaction myTransaction = new MyTransaction();
        UserService userService = new UserServiceImpl();
        //产生静态代理对象
        ProxyUser proxyUser = new ProxyUser(userService,myTransaction);
        proxyUser.addUser(null);
        proxyUser.deleteUser(0);
    }
}
```

运行结果：
```
开启事务
增加User
提交事务
开启事务
删除User
提交事务
```
这是一个很基础的静态代理，业务类UserServiceImpl只需要关注业务逻辑本身，保证了业务的重用性，这也是代理类的优点，没什么好说的。我们主要说说这样写的缺点：
- 1.代理对象的一个接口只服务于一种类型的对象，如果要代理的方法很多，势必要为每一种方法都进行代理，静态代理在程序规模稍大时就无法胜任了。
- 2.如果接口增加一个方法，比如 UserService 增加修改 updateUser(）方法，则除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度。


#### 4.解决办法2：使用JDK动态代理

动态代理就不要自己手动生成代理类了，我们去掉ProxyUser.java类，增加一个ObjectInterceptor.java类；

```
package com.qc.service;

import com.qc.test.MyTransaction;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * @ Description:
 * @ Author: Chong Qin.
 * @ Date: Created in 2018/4/24,17:59
 */
public class ObjectInterceptor implements InvocationHandler {
    //目标类
    private Object target;
    //切面类(这里指事务类)
    private MyTransaction transaction;

    //通过构造器赋值
    public ObjectInterceptor(Object target, MyTransaction transaction) {
        this.target = target;
        this.transaction = transaction;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //开启事务
        this.transaction.before();
        //调用目标类方法
        method.invoke(this.target,args);
        //提交事务
        this.transaction.after();
        return null;
    }
}
```

测试：

```
public class testAop {
    public static void main(String[] args) {
        //目标类
        Object target = new UserServiceImpl();
        //事务类
        MyTransaction myTransaction = new MyTransaction();
        ObjectInterceptor objectInterceptor = new ObjectInterceptor(target,myTransaction);
        /**
         * 三个参数的含义：
         * 1、目标类的类加载器
         * 2、目标类所有实现的接口
         * 3、拦截器
         */
        UserService userService = (UserService) Proxy.newProxyInstance(target.getClass().getClassLoader(),
           target.getClass().getInterfaces(),objectInterceptor);
        userService.addUser(null);
    }
}
```

- 那么使用动态代理来完成这个需求就很好了，后期在 UserService 中增加业务方法，都不用更改代码就能自动给我们生成代理对象。而且将 UserService 换成别的类也是可以的。
- 也就是做到了代理对象能够代理多个目标类，多个目标方法。
- 注意：我们这里使用的是 JDK 动态代理，要求是必须要实现接口。与之对应的另外一种动态代理实现模式 Cglib，则不需要，我们这里就不讲解 cglib 的实现方式了。
- 不管是哪种方式实现动态代理。本章的主角：AOP 实现原理也是动态代理

#### 5.AOP关键术语

- 1.target：目标类，需要被代理的类。例如UserService
- 2.Joinpoint：连接点，被拦截到的点，因为Spring只支持方法类型的连接点，所以在Spring中连接点是指那些可能被拦截到的方法，例如：所有的方法；实际上连接点还可以是字段或者构造器；
- 3.PointCut：切入点，指已经被增强的连接点，对连接点进行拦截的定义，例如：addUser()
- 4.advice：通知/增强，指的是拦截到连接点之后要执行的代码，通知分为前置、后置、异常、最终、环绕通知五类。例如：after，before
- 5.Weaving：织入：是指把增强advice应用到目标对象target来创建新的代理对象proxy的过程；
- 6.proxy：代理类，通知+切入点
- 7.Aspect：切面，类是对物体特征的抽象，切面就是对横切关注点的抽象；指切入点pointcut和通知advice的结合
- 8.横切关注点：对哪些方法进行拦截，拦截后怎么处理，这些关注点称为横切关注点

具体可以根据下面这张图来理解：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/spring/Spring6_1.png)

#### 6.AOP的通知类型

Spring按照通知Advice在目标类方法的连接点位置，可以分为5类：
- 前置通知 org.springframework.aop.MethodBeforeAdvice
	- 在目标方法执行前实施增强，比如上面例子的 before()方法
- 后置通知 org.springframework.aop.AfterReturningAdvice
	- 在目标方法执行后实施增强，比如上面例子的 after()方法
- 环绕通知 org.aopalliance.intercept.MethodInterceptor
	- 在目标方法执行前后实施增强
- 异常抛出通知 org.springframework.aop.ThrowsAdvice
	- 在方法抛出异常后实施增强
- 引介通知 org.springframework.aop.IntroductionInterceptor
	- 在目标类中添加一些新的方法和属性 

#### 7.Spring对AOP的支持

**Spring中AOP代理由Spring的IOC容器负责生成、管理，其依赖关系也由IOC容器负责管理。**因此，AOP代理可以直接使用容器中的其它bean实例作为目标，这种关系可由IOC容器的依赖注入提供。Spring创建代理的规则为：
- 默认使用Java动态代理来创建AOP代理，这样就可以为任何接口实例创建代理了；
- 当需要代理的类不是代理接口的时候，Spring会切换为使用CGLIB代理，也可强制使用CGLIB

AOP编程其实是很简单的事情，纵观AOP编程，程序员只需要参与三个部分：
- 1.定义普通业务组件；
- 2.定义切入点，一个切入点可能横切多个业务组件；
- 3.定义增强处理，增强处理就是在AOP框架为普通业务组件织入的处理动作；

所以进行AOP编程的关键就是定义切入点和定义增强处理，一旦定义了合适的切入点和增强处理，AOP框架将自动生成AOP代理，即：代理对象的方法=增强处理+被代理对象的方法。

#### 8.使用Spring AOP解决上面的需求

我们只需要在applicationContext.xml中进行如下配置：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--1.创建目标类-->
    <bean id="userService" class="com.qc.test.UserServiceImpl"/>
    <!--2.创建切面类（通知）-->
    <bean id="transaction" class="com.qc.test.MyTransaction"/>

    <!--3.aop编程
    3.1 导入命名空间
    3.2 使用<aop:config>进行配置，proxy-target-class="true" 声明时使用cglib代理
    如果不声明，Spring会自动选择cglib代理还是JDK动态代理
    <aop:pointcut>切入点，从目标对象获得具体方法
    <aop:advisor>:特殊的切面，只有一个通知和一个切入点
    advice-ref:通知引用
    pointcut-ref:切入点引用
    3.3 切入点表达式
        execution(* com.qc.service.*.*(..))
         选择方法         返回值任意   包             类名任意   方法名任意   参数任意-->
    <aop:config>
        <!--切入点表达式-->
        <aop:pointcut expression="execution(* com.qc.test.*.*(..))" id="myPointCut" />
        <aop:aspect ref="transaction">
            <!--配置前置通知，注意method的值要和对应切面的类方法名称相同-->
            <aop:before method="before" pointcut-ref="myPointCut" />
            <aop:after method="after" pointcut-ref="myPointCut" />
        </aop:aspect>
    </aop:config>
</beans>
```

测试类：

```
public class testAop {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService userService = (UserService)context.getBean("userService");
        userService.addUser(null);
    }
}
```

运行结果：

```
开启事务
增加User
提交事务
```
下面重点讲解一下execution切入点表达式(用来匹配执行方法的连接点)：
语法结构： execution(方法修饰符 **方法返回值** 方法所属类 **匹配方法名（方法中的形参表）** 方法声明抛出的异常)
> 其中加粗部分是不能省略的，各部分都支持通配符"*"来匹配全部；

其比较特殊的为形参表部分，其支持两种通配符：
- “*”：代表一个任意类型的参数；
- “..”：代表零个或多个任意类型的参数；
- 例如：()匹配一个无参方法；(..)匹配一个可接受任意类型参数的方法；(*)匹配一个接受一个任意类型参数的方法；(*,Integer)匹配一个两个参数的方法，第一个可以为任意类型，第二个必须为Integer；

**下面给出一些切入点表达式的实例：**
- `execution(public * *(..))`：任意公共方法的执行；
- `execution(* set*(..))`：任何一个以set开始的方法的执行；
- `execution(* com.xyz.service.AccountService.*(..))`：AccountService接口定义的任意方法的执行；
- `execution(* com.xyz.service.*.*(..))`：在service包中定义的任意方法的执行；
- `execution(* com.xyz.service..*.*(..))`：在service包或其子包中定义的方法的执行；

**SpringAOP的具体加载步骤：**
- 1.当Spring容器启动的时候，加载了Spring的配置文件；
- 2.为配置文件中的所有bean创建对象；
- 3.Spring容器会解析aop:config的配置
	- 解析切入点表达式，用切入点表达式和纳入Spring容器中的bean做匹配，如果匹配成功，则会为该bean创建代理对象，代理对象的方法=目标方法+通知，如果匹配不成功则不会创建代理对象；
- 4.在客户端利用context.getBean()获取对象时，如果该对象有代理对象则返回代理对象；如果没有则返回目标对象；说明：如果目标类没有实现接口，则Spring容器会采用cglib的方式产生代理对象，如果实现了接口，则会采用jdk的方式；


#### 9.基于Spring的AOP简单实现

**注意一下：在讲解之前，说明一点：使用Spring AOP，要成功运行起代码，只用Spring提供给开发者的jar包是不够的，请额外上网下载两个jar包：1.aopalliance.jar 2. aspectjweaver.jar，下面开始讲一下用Spring AOP的XMl实现方式，先定义一个接口：**

```
public interface HelloWorld {
    void printHelloWorld();
    void doPrint();
}
```

然后定义两个接口实现类：

```
public class HelloWorldImpl1 implements HelloWorld{
    @Override
    public void doPrint() {
        System.out.println("Enter HelloWorldImpl1.doPrint()");
    }

    @Override
    public void printHelloWorld() {
        System.out.println("Enter HelloWorldImpl1.printHelloWorld()");
    }
}
```

```
public class HelloWorldImpl2 implements HelloWorld{
    @Override
    public void printHelloWorld() {
        System.out.println("Enter HelloWorldImpl2.printHelloWorld()");
    }

    @Override
    public void doPrint() {
        System.out.println("Enter HelloWorldImpl2.doPrint()");
    }
}
```

横切关注点，这里是打印时间：

```
public class TimeHandler {
    public void printTime(){
        System.out.println("CurrentTime = " + System.currentTimeMillis());
    }
}
```

有这三个类就可以实现一个简单的Spring AOP了，看一下applicationContext.xml的配置：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="helloWorldImpl1" class="com.qc.entity.HelloWorldImpl1" />
    <bean id="helloWorldImpl2" class="com.qc.entity.HelloWorldImpl2" />
    <bean id="timeHandler" class="com.qc.entity.TimeHandler" />
    <bean id="logHandler" class="com.qc.entity.LogHandler" />
    <aop:config>
        <aop:aspect id="time" ref="timeHandler" order="1">
            <aop:pointcut id="addAllMethod" expression="execution(* com.qc.entity.*.*(..))"></aop:pointcut>
            <aop:before method="printTime" pointcut-ref="addAllMethod"></aop:before>
            <aop:after method="printTime" pointcut-ref="addAllMethod"></aop:after>
        </aop:aspect>
    </aop:config>
</beans>
```

写一个main函数调用一下：

```
public class Test {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        HelloWorld hw1 = (HelloWorld) context.getBean("helloWorldImpl1");
        HelloWorld hw2 = (HelloWorld) context.getBean("helloWorldImpl2");
        hw1.printHelloWorld();
        System.out.println();
        hw1.doPrint();

        System.out.println();
        hw2.printHelloWorld();
        System.out.println();
        hw2.doPrint();
    }
}
```

运行结果为：
```
CurrentTime = 1524647537948
Enter HelloWorldImpl1.printHelloWorld()
CurrentTime = 1524647537948

CurrentTime = 1524647537949
Enter HelloWorldImpl1.doPrint()
CurrentTime = 1524647537949

CurrentTime = 1524647537950
Enter HelloWorldImpl2.printHelloWorld()
CurrentTime = 1524647537950

CurrentTime = 1524647537952
Enter HelloWorldImpl2.doPrint()
CurrentTime = 1524647537952
```

我们可以看到给HelloWorld接口的两个实现类的所有方法都加上了代理，代理内容就是打印时间。

#### 10.基于Spring的AOP使用其他细节

增加一个横切关注点，打印日志，java类为：

```
public class LogHandler {
    public void LogBefore(){
        System.out.println("Log Before method");
    }

    public void LogAfter(){
        System.out.println("Log After method");
    }
}
```

然后在applicationContext.xml中增加一个切面：

```
<aop:aspect id="log" ref="logHandler" order="2">
            <aop:pointcut id="printLog" expression="execution(* com.qc.entity.*.*(..))"></aop:pointcut>
            <aop:before method="LogBefore" pointcut-ref="printLog"></aop:before>
            <aop:after method="LogAfter" pointcut-ref="printLog"></aop:after>
        </aop:aspect>
```

测试类不变，打印结果为：

```
CurrentTime = 1524647537948
Log Before method
Enter HelloWorldImpl1.printHelloWorld()
Log After method
CurrentTime = 1524647537948

CurrentTime = 1524647537949
Log Before method
Enter HelloWorldImpl1.doPrint()
Log After method
CurrentTime = 1524647537949

CurrentTime = 1524647537950
Log Before method
Enter HelloWorldImpl2.printHelloWorld()
Log After method
CurrentTime = 1524647537950

CurrentTime = 1524647537952
Log Before method
Enter HelloWorldImpl2.doPrint()
Log After method
CurrentTime = 1524647537952
```
想要让logHandler在timeHandler前使用有两个办法：
- 1.aspect里面有一个order属性，order属性的数字就是横切关注点的顺序；
- 2.把logHandler定义在timeHandler前面，Spring默认以aspect的定义顺序作为织入顺序；

如果是只想织入接口中的某些方法，修改pointcut的expression就好了：

```
<aop:aspect id="time" ref="timeHandler" order="1">
            <aop:pointcut id="addAllMethod" expression="execution(* com.qc.entity.HelloWorld.print*(..))"></aop:pointcut>
            <aop:before method="printTime" pointcut-ref="addAllMethod"></aop:before>
            <aop:after method="printTime" pointcut-ref="addAllMethod"></aop:after>
        </aop:aspect>
```
表示timeHandler只会织入HelloWorld接口print开头的方法。

**关于强制使用CGLIB生成代理的问题：**

前面说过Spring使用动态代理或是CGLIB生成代理是有规则的，高版本的Spring会自动选择是使用动态代理还是CGLIB生成代理内容，当然我们也可以强制使用CGLIB生成代理，那就是<aop:config>里面有一个"proxy-target-class"属性，这个属性值如果被设置为true，那么基于类的代理将起作用，如果proxy-target-class被设置为false或者这个属性被省略，那么基于接口的代理将起作用。
