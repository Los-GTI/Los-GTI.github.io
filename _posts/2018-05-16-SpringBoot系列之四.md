---
layout:     post
title: SpringBoot系列之四
subtitle: Spring Boot构建RESTful API与使用Postman测试
date:       2018-04-23
author:     Los-GTI
header-img: img/ms1.jpg
catalog: true
tags:
    - SpringBoot
---

在开始之前首先说明一下前面使用到的`@Controller、@RestController、@RequestMapping`注解。
- `@Controller:`修饰class,用来创建处理http请求的对象；
- `@RestController:`Spring4之后加入的注解，原来在`@Controler`中返回json需要`@ResponseBody`来配合，如果直接使用`@RestController`替代`@Controller`就不再需要配置`@ResponseBody`，默认返回json格式。
- `@RequestMapping:`配置url映射；

下面我们使用Spring MVC来实现一组对User对象操作的RESTful API，配合注释详细说明在SpringMVC中如何映射HTTP请求、如何传参、如何编写单元测试。

**RESTful API具体设计如下：**

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/SpringBoot/springboot7.png)

**User实体定义**

```
public class User {
    private long id;
    private String name;
    private Integer age;

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

**实现对User对象的操作接口：**

```
@RestController
@RequestMapping(value = "/users")   //通过这里的配置使下面的映射都在/users下
public class UserController {

    //创建线程安全的Map
    static Map<Long,User> users = Collections.synchronizedMap(new HashMap<Long, User>());

    @RequestMapping(value = "/" ,method = RequestMethod.GET)
    public List<User> getUserList(){
        /**
         * 处理"/users/"下的get请求，用来获取用户列表
         * 还可以通过@RequestParam从页面中传递参数来进行查询条件或者翻页信息的传递
         */
        List<User> r = new ArrayList<User>(users.values());
        return r;
    }

    @RequestMapping(value = "/",method = RequestMethod.POST)
    public String postUser(@ModelAttribute User user){
        /**
         * 处理"/users/"下的post请求，用来创建User
         * 除了@ModelAttribute绑定参数外，还可以通过@RequestParam从页面中传递参数
         */
        users.put(user.getId(),user);
        return "success";
    }

    @RequestMapping(value = "/{id}",method = RequestMethod.GET)
    public User getUser(@PathVariable long id){
        /**
         * 处理"/users/{id}"的GET请求，用来获取url中id值的用户信息
         * url中的id值可以通过@PathVariable绑定到函数的参数中
         */
        return users.get(id);
    }

    @RequestMapping(value = "/{id}",method = RequestMethod.PUT)
    public String putUser(@PathVariable long id,@ModelAttribute User user){
        /**
         * 用来处理"/users/{id}"下的put请求，用来更新user
         */
        User u = users.get(id);
        u.setName(user.getName());
        u.setAge(user.getAge());
        users.put(id,u);
        return "success";
    }

    @RequestMapping(value = "/{id}",method = RequestMethod.DELETE)
    public String delUser(@PathVariable long id){
        /**
         * 用来处理"/users/{id}"的DELETE请求
         */
        users.remove(id);
        return "success";
    }
}
```

下面我们将这个demo运行起来，对该Controller通过浏览器插件Postman进行请求提交验证，想知道Postman用法的可以去看我博客的Postman系列文章。

- 首先发起一个get请求查询一下user列表，应该为空；

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/SpringBoot/springboot8.png)

- 然后发起一个post请求提交一个user

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/SpringBoot/springboot9.png)

- 发起一个get请求获取user列表，看看有没有刚刚插入的数据

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/SpringBoot/springboot10.png)

- 下面测试一个put请求，修改id为1的user

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/SpringBoot/springboot11.png)

- 然后发送一个get请求查看id为1的user，可以看到我们的id为1的user已经变成了修改后的数据

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/SpringBoot/springboot12.png)

- 最后我们再发送一个DELETE请求，删除id为1的user，然后发送一个GET请求查询用户列表，最后应该为空；

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/SpringBoot/springboot13.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/SpringBoot/springboot14.png)


至此我们通过引入web模块（没有做其他的任何配置），就可以轻松利用Spring MVC的功能，以非常简洁的代码完成了对User对象的RESTful API的创建以及单元测试的编写。
