---
layout:     post
title: 混口饭吃专题之Spring详解之七AspectJ实现AOP
subtitle: Spring系列之七
date:       2018-05-01
author:     Los-GTI
header-img: img/ms1.jpg
catalog: true
tags:
    - 混口饭吃
---


上一篇博客我们引出了 AOP 的概念，以及 AOP 的具体实现方式。但是为什么要这样实现？以及提出的切入点表达式到底该怎么理解？

这篇博客我们通过对 AspectJ 框架的介绍来详细了解。

#### 1.什么是AspectJ?

AspectJ是一个面向切面的框架，它扩展了Java语言。AspectJ定义了AOP语法，也可以说 AspectJ 是一个基于 Java 语言的 AOP 框架。通常我们在使用 Spring AOP 的时候，都会导入 AspectJ 的相关 jar 包。

在 spring2.0以后，spring新增了对AspectJ 切点表达式的支持；Aspect1.5新增注解功能，通过 JDK5的注解技术，能直接在类中定义切面；新版本的 spring 框架，也都建议使用 AspectJ 来实现 AOP。所以说在 spring AOP 的核心包 Spring-aop-3.2.jar 里面也有对 AspectJ 的支持。

#### 2. 切入点表达式

上一篇博客中，我们在Spring的配置文件中配置如下：

```
<!--切入点表达式-->
<aop:pointcut expression="execution(* com.qc.entity.*.*(..))" id="myPointCut" />
```

那么它表达的意思是**返回值任意，包名为com.qc.entity下的任意类中的任意方法名，参数任意。**那么这到底是什么意思呢？

首先execution是AspectJ框架定义的一个切入点函数，其语法形式如下：

```
execution（modifiers-pattern?(类修饰符) ref-type-pattern（返回值） declaring-type-pattern?（方法所在的包） name-pattern(param-pattern)（方法名） throws-pattern?（方法抛出的异常））
//其中类修饰符、方法所在的包和抛出的异常可以省略，返回值、方法名不能省略
```

简单点来说就是：

`execution(修饰符 返回值 包.类.方法名(参数) throws异常)`

具体解释我们用下面一张思维导图来看：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/spring/Spring7_1.png)

> 注意：如果切入点表达式有多个不同目录呢？可以通过||来表示或的关系。

`<aop:pointcut expression="execution(* com.qc.entity.*.*(..)) || execution(* com.qc.service.*.*(..))" id="myPointCut" />`

表示匹配com.qc.entity和com.qc.service包下的任意类的任意方法！

**AOP切入点表达式支持多种形式的定义规则：**
- 1.execution：匹配方法的执行（常用），例如execution(public *.*(..))
- 2.within：匹配包或子包中的方法（了解），例如within(com.qc.entuty..*)
- 3.this：匹配实现接口的代理对象中的方法（了解），例如this(com.qc.dao.UserDao)
- 4.target：匹配实现接口的目标对象中的方法（了解），例如target(com.qc.dao.UserDao)
- 5.args：匹配参数格式符合标准的方法（了解），例如args(int,int)
- 6.bean(id)：对指定的bean所有的方法（了解），例如bean('userServiceId')

#### 3.Aspect通知类型

Aspect通知类型，定义了类型名称以及方法格式。类型如下：

- **before:前置通知（应用：各种校验）**：在方法执行前执行，如果通知抛出异常，阻止方法运行；
- **afterReturning:后置通知（应用：常规数据处理）**：方法正常返回后执行，如果方法中抛出异常，通知无法执行；必须在方法执行后才执行，所以可以获得方法的返回值；
- **around：环绕通知（应用：十分强大，可以做任何事情）**：方法执行前后分别执行，可以阻止方法的执行；必须手动执行目标方法；
- **afterThrowing:抛出异常通知（应用：包装异常信息）**：方法抛出异常后执行，如果没有方法抛出异常，无法执行；
- **after:最终通知（应用：清理现场）**:方法执行完毕后执行，无论方法中是否出现异常；

**这里最重要的是around环绕通知，它可以代替上面的任意通知！！！**

在程序中表示的意思如下：
```
try{
     //前置：before
    //手动执行目标方法
    //后置：afterRetruning
} catch(){
    //抛出异常 afterThrowing
} finally{
    //最终 after
}
```

#### 4.AOP具体实例

**1.创建接口**

```
public interface UserServiceInterface {
    //添加user
    public void addUser();
    //删除user
    public void deleteUser();
}
```

**2.接口的实现类**

```
public class UserServiceImpl implements UserServiceInterface{
    @Override
    public void deleteUser() {
        System.out.println("删除 User");
    }

    @Override
    public void addUser() {
        System.out.println("增加 User");
    }
}
```

**3.创建切面类（包含各种通知）**

```
public class MyAspect {
    /**
     * JoinPoint:能获取目标方法的一些基本信息
     *
     * @param joinpoint
     */
    public void myBefore(JoinPoint joinPoint) {
        System.out.println("前置通知：" + joinPoint.getSignature().getName());
    }

    public void myAfterReturning(JoinPoint joinPoint, Object ret) {
        System.out.println("后置通知：" + joinPoint.getSignature().getName() + ",-->" + ret);
    }

    public void myAfter(){
        System.out.println("最终通知");
    }
}
```

**4.创建spring配置文件applicationContext.xml**

我们首先测试前置通知、后置通知、最终通知

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
    <bean id="userService" class="com.qc.aop.UserServiceImpl"></bean>
    <!--2.创建切面类（通知）-->
    <bean id="myAspect" class="com.qc.aop.MyAspect"></bean>


    <!--3.aop编程
        3.1 导入命名空间
        3.2 使用<aop:config>进行配置
            proxy-target-class="true" 声明时使用cglib代理，如果不声明，
            Spring会自动选择是使用cglib还是jdk动态代理
            <aop:pointcut> 切入点 ，从目标对象获得具体方法
            <aop:advisor> 特殊的切面，只有一个通知 和 一个切入点
                advice-ref 通知引用
                pointcut-ref 切入点引用
        3.3 切入点表达式-->

    <aop:config>
        <aop:aspect ref="myAspect">
            <!--切入点表达式-->
            <aop:pointcut expression="execution(* com.qc.aop.*.*(..))" id="myPointCut"></aop:pointcut>
            <!--前置通知
                <aop:before method="" pointcut="" point-ref="" />
                method:通知，方法名
                pointcut:切入点表达式，此表达式只能当前通知使用
                pointcut-ref:切入点引用，可以与其他通知共享切入点
                通知方法格式：public void myBefore(JoinPoint joinPoint){
                      参数1：org.aspectJ.lang.JoinPoint 用于描述连接点（目标方法），获得目标方法名等
                }-->
            <aop:before method="myBefore" pointcut-ref="myPointCut" />
            <!--后置通知 方法后执行，获得返回值
                <aop:after-returning method="" point-ref="" returning="" />
                returning:通知方法第二个参数的名称
                通知方法格式：public void myAfterReturning(JoinPoint joinPoint,Object ret)
                参数1：连接点描述，参数2：类型Object，参数名returning="ret" 配置的 -->
            <aop:after-returning method="myAfterReturning" pointcut-ref="myPointCut" returning="ret" />
            <!--最终通知-->
            <aop:after method="myAfter" pointcut-ref="myPointCut" />
        </aop:aspect>
    </aop:config>
</beans>
```

**5.测试类**

```
public class AopTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserServiceInterface userServiceInterface = (UserServiceInterface) context.getBean("userService");
        userServiceInterface.addUser();
    }
}
```

**6.控制台打印**

```
前置通知：addUser
增加 User
后置通知：addUser,-->null
最终通知
```

注意，后置通知的返回值为 null，是因为我们的目标方法 addUser() 没有返回值。如果有返回值，这里就是addUser() 的返回值。

#### 5.测试异常通知

目标接口保持不变，目标类中我们引入异常：

```
@Override
    public void addUser() {
        int i = 1/0;//显然这里会抛出除数不能为0
        System.out.println("增加 User");
    }
```

接着配置切面：MyAspect.java

```
public void myAfterThrowing(JoinPoint joinPoint,Throwable throwable){
        System.out.println("抛出异常通知：" + throwable.getMessage());
    }
```

接着在applicationContext.xml中配置如下：

```
 <!--抛出异常
                <aop:after-throwing method="" pointcut-ref="" throwing="" />
                throwing:通知方法的第二个参数名称
                通知方法格式：public void myAfterThrowing(JoinPoint joinPoint,Throwable e){
                    参数1：连接点描述对象，参数2：获得异常信息，类型throwable,参数名由
                    throwing="e"设置
                }-->
            <aop:after-throwing method="myAfterThrowing" pointcut-ref="myPointCut" throwing="throwable"/>
```

测试：

```
public class AopTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserServiceInterface userServiceInterface = (UserServiceInterface) context.getBean("userService");
        userServiceInterface.addUser();
    }
}
```

控制台打印：
```
前置通知：addUser
最终通知
抛出异常通知：/ by zero
```

#### 6.测试环绕通知

目标接口和目标类不变，切面MyAspect修改如下：

```
public Object myAround(ProceedingJoinPoint joinPoint) throws Throwable{
        System.out.println("前置通知");
        //手动执行目标方法
        Object obj = joinPoint.proceed();

        System.out.println("后置通知");
        return obj;
    }
```

applicationContext.xml配置如下：

```
<!-- 环绕通知
                <aop:around method="" pointcut-ref=""/>
                通知方法格式：public Object myAround(ProceedingJoinPoint joinPoint) throws Throwable{
                    返回值类型：Object
                    方法名：任意
                    参数：org.aspectj.lang.ProceedingJoinPoint
                    抛出异常
                执行目标方法：Object obj = joinPoint.proceed();
        -->
            <aop:around method="myAround" pointcut-ref="myPointCut" />
```

控制台打印：
```
前置通知
增加 User
后置通知
```
 
那么至此，通过 xml 配置的方式我们讲解了Spring AOP 的配置。
