---
layout:     post
title: spring注解
subtitle:   @controller、@RequestMapping
date:       2017-10-18
author:     Los-GTI
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Controller
---

### spring注解` @controller、@RequestMapping`详解

###### 1.spring注解` @controller`详解

> 一个简单的基于注解的controller

当创建一个controller时，我们需要直接或者间接的实现` org.springframework.web.servlet.mvc.Controller` 一般情况下我们是通过继承SimpleFormController 或者 MultiActionController来定义自己的Controller的。在定义Controller之后，一个重要的事件就是在Spring MVC的配置文件中通过HandlerMapping定义请求和控制器的映射关系，以便将两者关联起来。

> 来看一下基于注解的controller是如何定义到这一点的，下面是使用注解的PersonController

```
package com.huawei.controller;

import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import com.huawei.model.Person;
import com.huawei.service.IPersonService;

//1-用来标注此类是springmvc管理的控制层的类
@Controller
//2-为了防止多个控制层的类有相同的方法所以要为每个控制层类起个唯一的标识名
@RequestMapping("/personController")
public class PersonController {
    private IPersonService personService;
    public IPersonService getPersonService() {
        return personService;
    }
    @Autowired
    public void setPersonService(IPersonService personService) {
        this.personService = personService;
    }
    //3-访问此方法的URL路是/personController/showPerson
    @RequestMapping("/showPerson")
    public String showPersons(Model model){
        List<Person> persons = personService.loadPersons();
        //将参数值放到request里面
        model.addAttribute("persons", persons);
        return "showperson";
    }
}
```

从上面的代码中我们可以看出，PersonController类与普通的PoJo是没有什么大的区别的，唯一的区别就是spring MVC的注解。可以看到上图代码中主要有3处注解，第一个注解@controller的作用是让PersonController成为Spring容器的Bean。由于Spring MVC的Controller必须事先是一个Bean，所以@Controller是不可缺少的。另：@Controller、@Service、@Repository和@Component注解的作用是等价的。真正让PersonController具备Spring MVC Controller功能的是第2个注解，即@RequestMapping注解。@RequestMapping可以标注在类定义处，将Controller和特定请求关联起来，还可以标注在方法签名处，以便进一步对请求进行分流。在2处注解我们让PersonController关联“/personController”请求，而在3处的注解我们具体的指定showpersons()方法来处理请求。所以在类声明处标注的 @RequestMapping 相当于让 POJO 实现了 Controller 接口，而在方法定义处的 @RequestMapping 相当于让 POJO 扩展 Spring 预定义的 Controller（如 SimpleFormController 等）。

> 为了让基于注解的Spring MVC真正工作起来，需要在Spring MVC对应的xxx-servlet.xml配置文件中做一些手脚。在此之前让我们先看一下web.xml的配置吧。

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	id="WebApp_ID" version="2.5">
	<display-name>Archetype Created Web Application</display-name>
	<!-- 解决工程编码过滤器 -->
	<filter>
		<filter-name>characterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	<!-- 配置文件所在位置 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:spring.xml,classpath:mybatis-spring.xml</param-value>
	</context-param>
	<!-- Spring配置（监听器） -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	<!-- SpringMVC配置 -->
	<servlet>
		<servlet-name>springDispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:spring-mvc.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>springDispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
	<welcome-file-list>
		<welcome-file>index.html</welcome-file>
		<welcome-file>index.jsp</welcome-file>
	</welcome-file-list>
</web-app>
```

web.xml中定义了一个名为springDispatcherServlet的Spring MVC模块，按照Spring MVC的契约，需要在spring-mvc.xml配置文件中定义Spring MVC模块的具体配置。spring-mvc.xml的配置内容具体如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
xmlns:context="http://www.springframework.org/schema/context"
xmlns:mvc="http://www.springframework.org/schema/mvc"
xsi:schemaLocation="  http://www.springframework.org/schema/beans   
http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
http://www.springframework.org/schema/context    http://www.springframework.org/schema/context/spring-context-3.0.xsd  
http://www.springframework.org/schema/mvc  
http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd">
<!-- 启用spring mvc 注解 -->
<context:annotation-config />
<!-- ①设置使用注解的类所在的jar包，完成Bean创建和自动依赖注入的功能 -->
<context:component-scan base-package="com.huawei.controller" />
<!--②启动SpringMVC的注解功能，完成请求和注解POJO的映射 -->
    <bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter" />
    <!--③对转向页面的路径解析。prefix：前缀， suffix：后缀 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
        p:prefix="/" p:suffix=".jsp" />
</beans>
```

 因为 Spring 所有功能都在 Bean 的基础上演化而来，所以必须事先将 Controller 变成 Bean，这是通过在类中标注 @Controller 并在 spring-mvc.xml 中启用组件扫描机制来完成的，如 ① 所示。

 在 ② 处，配置了一个 AnnotationMethodHandlerAdapter，它负责根据 Bean 中的 Spring MVC 注解对 Bean 进行加工处理，使这些 Bean 变成控制器并映射特定的 URL 请求。

 而 ③ 处的工作是定义模型视图名称的解析规则，这里我们使用了 Spring 2.5 的特殊命名空间，即 p 命名空间，它将原先需要通过 <property> 元素配置的内容转化为 <bean> 属性配置，在一定程度上简化了 <bean> 的配置。

启动 Tomcat，发送 <http://localhost/personController/showperson> URL 请求，PersonController的 showPerson() 方法将响应这个请求，并转向 WEB-INF/jsp/showperson.jsp 的视图页面。

##### 2.spring注解@RequestMapping详解

RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。

RequestMapping注解有六个属性，下面我们把她分成三类进行说明。

###### 2.1、value,method

value：     指定请求的实际地址，指定的地址可以是URI Template 模式（后面将会说明）；

method：  指定请求的method类型， GET、POST、PUT、DELETE等；

###### 2.2、consumes,produces

consumes： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;

produces:    指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回；

###### 2.3、params,headers

params： 指定request中必须包含某些参数值是，才让该方法处理。

headers： 指定request中必须包含某些指定的header值，才能让该方法处理请求

> 示例

###### 2.4.1、value/method示例

```
@Controller  
@RequestMapping("/appointments")  
public class AppointmentsController {  
  
    private final AppointmentBook appointmentBook;  
      
    @Autowired  
    public AppointmentsController(AppointmentBook appointmentBook) {  
        this.appointmentBook = appointmentBook;  
    }  
  
    @RequestMapping(method = RequestMethod.GET)  
    public Map<String, Appointment> get() {  
        return appointmentBook.getAppointmentsForToday();  
    }  
  
    @RequestMapping(value="/{day}", method = RequestMethod.GET)  
    public Map<String, Appointment> getForDay(@PathVariable @DateTimeFormat(iso=ISO.DATE) Date day, Model model) {  
        return appointmentBook.getAppointmentsForDay(day);  
    }  
  
    @RequestMapping(value="/new", method = RequestMethod.GET)  
    public AppointmentForm getNewForm() {  
        return new AppointmentForm();  
    }  
  
    @RequestMapping(method = RequestMethod.POST)  
    public String add(@Valid AppointmentForm appointment, BindingResult result) {  
        if (result.hasErrors()) {  
            return "appointments/new";  
        }  
        appointmentBook.addAppointment(appointment);  
        return "redirect:/appointments";  
    }  
}
```

value的uri值为以下三类：

A） 可以指定为普通的具体值；

B)  可以指定为含有某变量的一类值(URI Template Patterns with Path Variables)；

C) 可以指定为含正则表达式的一类值( URI Template Patterns with Regular Expressions);

example B)

```
@RequestMapping(value="/owners/{ownerId}", method=RequestMethod.GET)  
public String findOwner(@PathVariable String ownerId, Model model) {  
  Owner owner = ownerService.findOwner(ownerId);    
  model.addAttribute("owner", owner);    
  return "displayOwner";   
}  
```

example C)

```
@RequestMapping("/spring-web/{symbolicName:[a-z-]+}-{version:\d\.\d\.\d}.{extension:\.[a-z]}")  
  public void handle(@PathVariable String version, @PathVariable String extension) {      
    // ...  
  }  
}  
```

###### 2.4.2 consumes/produces示例

> consumes的样例

```
@Controller  
@RequestMapping(value = "/pets", method = RequestMethod.POST, consumes="application/json")  
public void addPet(@RequestBody Pet pet, Model model) {      
    // implementation omitted  
}  

方法仅处理request Content-Type为“application/json”类型的请求。
```

> produces的样例

```
@Controller  
@RequestMapping(value = "/pets/{petId}", method = RequestMethod.GET, produces="application/json")  
@ResponseBody  
public Pet getPet(@PathVariable String petId, Model model) {      
    // implementation omitted  
}  
```

方法仅处理request请求中Accept头中包含了"application/json"的请求，同时暗示了返回的内容类型为application/json;

###### 2.4.3 params/headers示例

> params的样例

```
@Controller  
@RequestMapping("/owners/{ownerId}")  
public class RelativePathUriTemplateController {  
  
  @RequestMapping(value = "/pets/{petId}", method = RequestMethod.GET, params="myParam=myValue")  
  public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {      
    // implementation omitted  
  }  
}  
```

 仅处理请求中包含了名为“myParam”，值为“myValue”的请求；

> headers的样例

```
@Controller  
@RequestMapping("/owners/{ownerId}")  
public class RelativePathUriTemplateController {  
  
@RequestMapping(value = "/pets", method = RequestMethod.GET, headers="Referer=http://www.ifeng.com/")  
  public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {      
    // implementation omitted  
  }  
}  
```

 仅处理request的header中包含了指定“Refer”请求头和对应值为“`http://www.ifeng.com/`”的请求；

上面仅仅介绍了，RequestMapping指定的方法处理哪些请求，下面一篇将讲解怎样处理request提交的数据（数据绑定）和返回的数据。

> 摁钉：author：chong Los-GTI
