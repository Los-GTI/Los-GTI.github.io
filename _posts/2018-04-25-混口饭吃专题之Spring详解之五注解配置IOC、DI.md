---
layout:     post
title: 混口饭吃专题之Spring详解之五注解配置IOC、DI
subtitle: Spring系列之五
date:       2018-04-25
author:     Los-GTI
header-img: img/ms1.jpg
catalog: true
tags:
    - 混口饭吃
---


Annotation(注解)是JDK1.5及以后版本引入的。它可以用于创建文档，跟踪代码中的依赖性，甚至执行基本编译时检查。注解是以'@注解名'在代码中存在的。
前面讲解IOC和DI都是通过xml文件来配置，我们发现xml配置还是比较麻烦的，那么如何简化配置呢？答案就是使用注解！

#### 1、注解 @Component

我们这里有个类Person.java

```
package com.qc.annotation;

/**
 * @ Description:
 * @ Author: Chong Qin.
 * @ Date: Created in 2018/4/24,10:21
 */
public class Person {
    private int pid;
    private String pname;
    private String psex;

    public int getPid() {
        return pid;
    }

    public void setPid(int pid) {
        this.pid = pid;
    }

    public String getPname() {
        return pname;
    }

    public void setPname(String pname) {
        this.pname = pname;
    }

    public String getPsex() {
        return psex;
    }

    public void setPsex(String psex) {
        this.psex = psex;
    }
}

```

如果我们不使用注解，通过前面讲解过的，要想让Spring容器帮我们产生Person对象，我们要在Spring配置文件applicationContext.xml中进行如下配置：

```
<bean id="person" class="com.qc.annotation.Person"></bean>
```

**那么如何使用注解呢？**

第一步：在applicationContext.xml中引入命名空间

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/spring/Spring3_1.png)

第二步：在applicationContext.xml中引入注解扫描器

```
<!--组件扫描，扫描含有注解的类-->
<context:component-scan base-package="com.qc.annotation"></context:component-scan>
```

base-package:表示含有注解类的包名，如果扫描多个包，则上面的代码书写多行，改变base-package里面的内容即可；

第三步：在Person类中添加注解@Component

```
@Component
public class Person {
    private int pid;
    private String pname;
    private String psex;
    ...
    }
```

第四步：测试

```
public static void testAnnotation(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        Person person = (Person) context.getBean("person");
        System.out.println(person.getPname());
    }

    public static void main(String[] args) {
        testAnnotation();
    }
```

如果你看完上面的配置一脸懵逼，没关系下面我们来详细讲解：

**@Component：把普通POJO实例化到Spring容器中，相当于配置文件中的`<bean id="" class=""></bean>`**

如果一个类加上了**@Component**注解就会进行如下的法则：
- 如果其value属性的值为""
```
@Component
public class Person{
}
```
等价于`<bean id="person" class="...Person"></bean>`

- 如果value属性的值不为""
```
@Component("p")
public class Person{
}
```
等价于`<bean id="p" class="...Person">`
这样就很好理解我们在测试程序中，我们直接`context.getBean("person")`这样写。

#### 2. 注解@Repository 、@Service、@Controller

此外：下面三个注解是@Component注解的衍生注解，功能一样；

```
@Controller:用于标注控制层组件（如Struts中的action）（web层）
@Service：用于标注业务层组件（service层）
@Repository：用于标注数据访问层组件（dao层）
```

#### 3.注解 @Resource

@Resource注解：它可以对类成员变量、方法以及构造函数进行标注，完成自动装配的工作。通过@Resource的使用消除set、get方法；

例子：首先创建一个Student类

```
@Component("student")
public class Student{
	public void descStudent(){
		System.out.println("descStudent...");
	}
}
```

然后在Person类中添加一个属性Student

```
@Component
public class Person(){
	Student student;

	public void showStudent(){
		this.student.descStudent();
	}
}
```

那么如何获取Person对象并调用showStudent()方法呢？这个问题就是如何给属性Student实例化，也就是依赖注入：

**不使用注解：**

```
<bean id="person" class="..Person">
	<property name="student">
		<ref bean="student"/>
	</property>
</bean>
<bean id="student" class="..Student"></bean>
```

**使用注解：**

```
	@Resource(name = "student")
    private Student student;

    public  void showStudent(){
        this.student.descStudent();
    }
```

```
@Component("student")
public class Student {
    public void descStudent(){
        System.out.println("descStudent...");
    }
}
```

- @Resource注解以后，判断该注解name的属性是否为""(name没有写)
	- 如果name属性没有写，则会让属性的名称的值和Spring配置文件bean中id值做匹配（如果没有进行配置，也和注解@Component进行匹配）
	- 如果有name属性，则会按照name属性的值和Spring的bean中id进行匹配，匹配成功则赋值，不成功则报错；

#### 4.注解@Autowired

功能和注解@Resource一样，可以对类成员变量、方法以及构造函数进行标注和自动装配的工作，只不过注解@Resource是按照名称来装配，而@Autowired是按照类型来进行装配；

例子：第一步：创建接口PersonDao

```
public interface PersonDao{
	public void savePerson();
}
```

第二步：创建一个接口实现类PersonDaoImpl

```
@Component("personDaoImpl")
public class PersonImpl implements PersonDao{
	public void savePerson(){
		System.out.println("sava Person...");
	}
}
```

第三步：创建PersonService

```
@Service("personService")
public class PersonService{
	@Autowired
	private PersonDao personDao;

	public void savePerson(){
		this.personDao.savePerson();
	}
}
```

注意：这里我们在 private PesronDao personDao 上面添加了注解 @Autowired，它首先会根据类型去匹配，PersonDao 是一个接口，它的实现类是 PesronDaoImpOne，那么这里的意思就是：
`PersonDao personDao = new PersonDaoImpOne();`

那么问题来了，如果 PersonDao 的实现类有多个呢？我们创建第二个实现类 PersonDaoImplTwo；

```
@Component("personDaoImplTwo")
public class PersonDaoImplTwo implements PersonDao{
	public void savePerson(){
		System.out.println("save Person two");
	}
}
```

如果还是像上面那样写，那么测试就会报错。怎么解决呢？

**第一种方法：更改名称**

```
@Service("personService")
public class PersonService{
	@Autowired
	//private PersonDao personDao;
	private PersonDao personDaoImplTwo;
	
	public void savePerson(){
		this.personDaoImplTwo.savePerson();
}
}
```

> @Autowired注解默认按照类型进行匹配，如果类型匹配的结果有多个，则按照名称进行匹配

**第二种方法：@Autowired和@Qualifier("名称")配合使用**

```
public class PersonService{
	@Autowired
	@Qualifier("personDaoImpl")
	private PersonDao personDao;
	
	public void savePerson(){
		this.personDao.savePerson();
	}
}
```

在使用@Autowired是，首先在容器中查询对应类型的bean
- 如果查询结果刚好为一个，将该bean装配给@Autowired指定的数据；
- 如果查询的结果不止一个，那么@Autowired会根据名称来查找；
- 如果查询的结果为空，那么会抛出异常。解决方法是使用required=false;



