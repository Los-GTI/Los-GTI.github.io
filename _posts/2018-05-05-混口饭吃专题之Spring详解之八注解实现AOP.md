---
layout:     post
title: 混口饭吃专题之Spring详解之八注解实现AOP
subtitle: Spring系列之八
date:       2018-05-05
author:     Los-GTI
header-img: img/ms1.jpg
catalog: true
tags:
    - 混口饭吃
---

### Spring详解（八）------ AOP注解

上一篇博客我们讲解了AspectJ框架如何实现AOP，然后具体的实现方式我们是通过xml来进行配置的。xml方式思路清晰，便于理解，但是书写过于麻烦。这篇博客我们将使用注解的方式来进行AOP配置。

为了便于大家理解，讲解方式是这样的。我们先给出xml的配置，然后介绍如何通过注解来进行替代。

#### 1.xml的方式实现AOP

**1.接口UserService**

```
public interface UserServiceInterface {
    //添加user
    public void addUser();
    //删除user
    public void deleteUser();
}
```

**2.实现类UserServiceImpl**

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

**3.切面类，也就是通知类MyAspect**

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

    public void myAfterThrowing(JoinPoint joinPoint,Throwable throwable){
        System.out.println("抛出异常通知：" + throwable.getMessage());
    }
}
```

#### 4.AOP配置文件applicationContext.xml

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
          
            <aop:after method="myAfter" pointcut-ref="myPointCut" />

            <!--抛出异常
                <aop:after-throwing method="" pointcut-ref="" throwing="" />
                throwing:通知方法的第二个参数名称
                通知方法格式：public void myAfterThrowing(JoinPoint joinPoint,Throwable e){
                    参数1：连接点描述对象，参数2：获得异常信息，类型throwable,参数名由
                    throwing="e"设置
                }-->

            <aop:after-throwing method="myAfterThrowing" pointcut-ref="myPointCut" throwing="throwable"/>
            
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

#### 2.注解实现AOP

**1.导入相应jar包，以及在applicationContext.xml文件中导入相应的命名空间**

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"     xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans                          http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd">
```

**2.注解配置bean，下面我们就利用注解来替代xml配置**

原xml配置为：
```
<!--1.创建目标类-->
    <bean id="userService" class="com.qc.aop.UserServiceImpl"></bean>
    <!--2.创建切面类（通知）-->
    <bean id="myAspect" class="com.qc.aop.MyAspect"></bean>
```

注解配置为：
目标类：
```
@Service("userService")
public class UserServiceImpl implements UserServiceInterface{
   //中间部分不变，这里为节省篇幅省略
   ......
}
```
切面类：

```
@Component("myAspect")
public class MyAspect {
    //中间部分不变，省略
    ......
}
```

**3.配置注解扫描器识别注解**

我们必须告诉Spring哪些类上加了注解，Spring如何去识别呢？我们在applicationContext.xml配置文件中添加如下配置：

```
<!--配置注解扫描器
            base-package：表示含有注解类的包名
            如果扫描多个包可以将下面的代码写多行，改变base-package里面的内容即可！
            -->
    <context:component-scan base-package="com.qc.aop"></context:component-scan>
```

**4.注解配置AOP**
我们原先利用xml配置的时候是在applicationContext.xml中配置`<aop:config><aop:aspect ref="myAspect" /></aop:config>`告诉Spring哪个是切面类，下面我们利用注解来配置。

```
@Component("myAspect")
@Aspect
public class MyAspect {
	//中间省略
	.....
}
```

**5.如何让Spring认识我们所配置的AOP注解呢？光有前面的类注解扫描是不够的，这里我们要额外配置AOP注解识别**

我们在applicationContext.xml文件中增加如下配置：

```
	<!--确定aop注解生效-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

**6.注解配置前置通知和后置通知为例**
我们先看一下通过xml配置的前置通知和后置通知：

```
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
```

那么注解的方式如下：

```
@Component("myAspect")
@Aspect
public class MyAspect {
    /**
     * JoinPoint:能获取目标方法的一些基本信息
     *
     * @param joinpoint
     */
    @Before("execution(* com.qc.aop.*.*(..))")
    public void myBefore(JoinPoint joinPoint){
        System.out.println("前置通知：" + joinPoint.getSignature().getName());
    }
    @AfterReturning(value = "execution(* com.qc.aop.*.*(..))",returning = "ret")
    public void myAfterReturning(JoinPoint joinPoint, Object ret) {
        System.out.println("后置通知：" + joinPoint.getSignature().getName() + ",-->" + ret);
    }
    }
```

**7.测试代码如下：**

```
public class AopTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserServiceInterface userServiceInterface = (UserServiceInterface) context.getBean("userService");
        userServiceInterface.addUser();
        userServiceInterface.deleteUser();
    }
}
```

运行结果：

```
前置通知：addUser
增加 User
后置通知：addUser,-->null
前置通知：deleteUser
删除 User
后置通知：deleteUser,-->null
```

**8.注解改进**

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/spring/Spring8_1.png)

注意看红色框住的部分，很显然这里是重复的，而且如果我们有多个通知方法，那就得在每个方法名都写上该注解，而且如果包名够复杂，也很容易写错。那么怎么办呢？

**解决办法就是声明公共切入点：在切面类MyAspect中新增一个切入点方法myPointCut()，然后在这个方法上添加@Pointcut注解**

```
@Pointcut("execution(* com.qc.aop.*.*(..))")
public void myPointCut(){
}
```

**那么前置通知和后置通知我们可以进行如下改写：**

```
 @Before(value="myPointCut()")
    public void myBefore(JoinPoint joinPoint){
        System.out.println("前置通知：" + joinPoint.getSignature().getName());
    }
    @AfterReturning(value ="myPointCut()",returning = "ret")
    public void myAfterReturning(JoinPoint joinPoint, Object ret) {
        System.out.println("后置通知：" + joinPoint.getSignature().getName() + ",-->" + ret);
    }
```

**9.总结**

上面我们只进行了前置通知和后置通知的讲解，还有比如最终通知、环绕通知、抛出异常通知等，配置方式都差不多，这里就不进行一一讲解了。然后我们看一下这些通知的注解：
- @Before：前置
- @AfterReturning：后置
- @Around：环绕
- @AfterThrowing：抛出异常
- @After：最终
- @Pointcut：切入点，修饰方法private void xxx(){}，之后通过“方法名”获得切入点引用
