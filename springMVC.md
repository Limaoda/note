# springmvc

### MVC是什么？

MVC是一种设计模式：

- 模型（model）
- 视图（view）
- 控制器（controller）

三层架构的设计模式。用于实现前端页面的展现与后端业务数据处理的分离。

### MVC设计模式的好处有哪些？

1. 分层设计，实现了业务系统各个组件之间的解耦，有利于业务系统的可扩展性，可维护性。
2. 有利于系统的并行开发，提升开发效率。

### Spring MVC注解

#### 注解原理是什么？
注解本质是一个继承了 __Annotation__ 的特殊 __接口__ ，其具体实现类是Java __运行时__ 生成的动态 __代理类__ 。我们通过 __反射获取注解__ 时，返回的是Java运行时生成的动态代理对象。通过代理对象调用自定义注解的方法，会最终调用 __AnnotationInvocationHandler的invoke方法__ 。该方法会从 __memberValues__ 这个Map中索引出对应的值。而memberValues的 __来源是Java常量池__ 。

#### Spring MVC常用的注解有哪些？
 __@RequestMapping__ ： 
此注解即可以作用在控制器的某个方法上，也可以作用在此控制器类上。

当控制器在类级别上添加@RequestMapping注解时，这个注解会应用到控制器的所有处理器方法上。处理器方法上的@RequestMapping注解会对类级别上的@RequestMapping的声明进行补充。

例子一：@RequestMapping __仅作用在处理器方法__ 上
```java
@Controller
public class HelloController {
     @RequestMapping(value="/hello",method=RequestMethod.GET)
    public String sayHello(){
        return "hello";
    }
}
```
以上代码sayHello所响应的url=localhost:8080/hello。


例子二：@RequestMapping __仅作用在类级别__ 上
```java
@Controller
@RequestMapping("/hello")
public class HelloController {

    @RequestMapping(method= RequestMethod.GET)
    public String sayHello(){
        return "hello";
    }
}
```
以上代码sayHello所响应的url=localhost:8080/hello,效果与例子一一样，没有改变任何功能


例子三：@RequestMapping __作用在类级别和处理器方法__ 上
```java
@RestController
@RequestMapping("/hello")
public class HelloController {

    @RequestMapping(value="/sayHello",method= RequestMethod.GET)
    public String sayHello(){
        return "hello";
    }
    @RequestMapping(value="/sayHi",method= RequestMethod.GET)
    public String sayHi(){
        return "hi";
    }
}

```
这样，以上代码中的sayHello所响应的url=localhost:8080/hello/sayHello。
sayHi所响应的url=localhost:8080/hello/sayHi。

  

  

>总结：当控制器在类级别上添加@RequestMapping注解时，这个注解会应用到控制器的所有处理器方法上。处理器方法上的@RequestMapping注解会对类级别上的@RequestMapping的声明进行补充。

>组合注解：Spring4.3中引进了｛@GetMapping、@PostMapping、@PutMapping、@DeleteMapping、@PatchMapping｝，来帮助简化常用的HTTP方法的映射，并更好地表达被注解方法的语义。
>以@GetMapping为例，Spring官方文档说：@GetMapping是一个组合注解，是@RequestMapping(method = RequestMethod.GET)的缩写。该注解将HTTP Get 映射到 特定的处理方法上。

 __@requestParam__ ：
作用：把请求中的指定名称的参数传递给控制器中的形参赋值

属性：
1. value / name：请求参数中的名称 （必写参数）
2. required：请求参数中是否必须提供此参数，默认值是true，true为必须提供
3. defaultValue：默认值
```java
@Controller
@RequestMapping("/anno")
public class AnnoController {
    @RequestMapping("/testRequestParam")
    public String testRequestParam(@RequestParam(value="name") String username){
        System.out.println("执行了...");
        System.out.println(username);
        return "success";
    }
//测试
<a href="anno/testRequestParam?name123=哈哈">点击测试RequestParam</a>
//测试结果：HTTP 400 
//原因：required默认为true，请求参数为name123，接收参数为name，参数错误

@Controller
@RequestMapping("/anno")
public class AnnoController {
    @RequestMapping("/testRequestParam")
    public String testRequestParam(@RequestParam(value="name",required=false) String username){
        System.out.println("执行了...");
        System.out.println(username);
        return "success";
    }
//测试
<a href="anno/testRequestParam?name123=哈哈">点击测试RequestParam</a>
//测试结果：HTTP 200
//不报错，但username取到的值为null，因为参数不匹配
```


__@PathVariable__ ： 可以用来获取请求路线上面的变量； 如请求路径：http://127.0.0.1/user/1 可以通过@PathVariable
 ```java
@RequestMapping(value = "/user/{id}", method = RequestMethod.GET)
 ```

 __@RequestBody__ ： 注解实现接收http请求的 json数据 ，将 __json转换为java对象__ 。

 __@ResponseBody__ ： 注解实现将controller方法 __返回对象转化为json对象__ 响应给客户。

>@Component：
>@Resource：
>@Autowired：为bean实体类进行自动装配（getter）
>@Repository：

#### Sping MVC中的控制器注解是什么？
一般用@ __Controller__ 注解; 也可以使用 __@RestController__ 

>@RestController注解相当于@ResponseBody ＋ @Controller

 __@RestController__ 

Spring4之后新加入的注解，原来返回json需要@ResponseBody和@Controller配合。
即@RestController是@ResponseBody和@Controller的组合注解。
```java
@RestController
public class HelloController {

    @RequestMapping(value="/hello",method= RequestMethod.GET)
    public String sayHello(){
        return "hello";
    }
}

/* 与下面的代码作用一样 */
@Controller
@ResponseBody
public class HelloController {

    @RequestMapping(value="/hello",method= RequestMethod.GET)
    public String sayHello(){
        return "hello";
    }
}
```

#### @Controller注解的作用

在Spring MVC 中，控制器Controller 负责处理由 __DispatcherServlet__  分发的请求，它把 __用户请求的数据__ 经过业务处理层处理之后 __封装成一个Model__ ，然后再把该Model 返回给对应的View 进行展示。在Spring MVC 中提供了一个非常简便的定义Controller 的方法，你无需继承特定的类或实现特定的接口，只需使用@Controller 标记一个类是Controller ，然后使用@RequestMapping 和@RequestParam 等一些注解用以定义URL 请求和Controller 方法之间的映射，这样的Controller 就能被外界访问到。此外Controller 不会直接依赖于HttpServletRequest 和HttpServletResponse 等HttpServlet 对象，它们可以通过Controller 的方法参数灵活的获取到。
@Controller 用于标记在一个类上，使用它标记的类就是一个Spring MVC Controller 对象。分发处理器将会扫描使用了该注解的类的方法，并检测该方法是否使用了@RequestMapping 注解。@Controller 只是定义了一个控制器类，而使用@RequestMapping 注解的方法才是真正处理请求的处理器。

#### @RequestMapping注解的作用?

RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。
RequestMapping注解有六个属性

-  __value__ ： 指定 __请求的实际地址__ ，指定的地址可以是URI Template 模式；
-  __method__ ： 指定 __请求的method类型__ ， GET、POST、PUT、DELETE等；

-  __consumes__ ： 指定处理 __请求的提交内容类型__ （Content-Type），例如application/json, text/html;
-  __produces__ : 指定 __返回的内容类型__ ，仅当request请求头中的(Accept)类型中包含该指定类型才返回；
-  __params__ ： 指定request中 __必须包含某些参数值__ 时，才让该方法处理。
-  __headers__ ： 指定request中 __必须包含某些指定的header值__ ，才能让该方法处理请求。

#### @ResponseBody注解的作用是什么？

-  __作用__ ： 该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区。
-  __使用时机__ ： 返回的数据不是html标签的页面，而是其他某种格式的数据时（如json、xml等）使用；

#### @PathVariable和@RequestParam的区别?

-  __@PathVariable__ ： 可以用来获取请求路线上面的变量； 如请求路径：http://127.0.0.1/user/1 可以通过@PathVariable
 ```java
@RequestMapping(value = "/user/{id}", method = RequestMethod.GET)
 ```
来获取路径在的变量id
-  __@RequestParam__ ： 用来获得静态的URL请求入参 spring注解时action里用到。

#### SpringMVC 中系统如何分层 ？

- 系统分为表现层（UI）： 数据的展现，操作页面，请求转发。
- 业务层（服务层）： 封装业务处理逻辑
- 持久层（数据访问层）： 封装数据访问逻辑

各层之间的关系： 表示层通过接口调用业务层，业务层通过接口调用持久层，这样，当下一层发生变化改变，不影响上一层的数据。 MVC是一种表现层的架构

![tapd_58612640_base64_1629298431_46](C:\Users\HuQiaoDong\Desktop\笔记\images\tapd_58612640_base64_1629298431_46.png)

