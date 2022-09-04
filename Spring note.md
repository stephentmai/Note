# Spring MVC的请求映射注解 #
|注解|描述|
|---|:---:|
|@RequestMapping|通用的请求处理|
|@GetMapping|处理HTTP GET请求|
|@PostMapping|处理HTTP POST请求|
|@PutingMapping|处理HTTP PUT请求|
|@DeleteMapping|处理HTTP DELETE请求|
|@PatchMapping|处理HTTP PATCH请求|

#注解解释#

## @SpringBootApplication是一个组合注解，它组合了3个其他的注解。 ##

- @SpringBootConfiguration：将该类声明为配置类。这个注解实际上是@Configuration注解的特殊形式。
- @EnableAutoConfiguration：启用Spring Boot的自动配置。
- @ComponentScan：启用组件扫描。这样我们能够通过像@Component、@Controller、@Service这样的注解声明其他类，Spring会自动发现它们并将它们注册为Spring应用上下文中的组件。
 
## @Valid 注解类型 ##
**@Valid**注解可以实现数据的验证，你可以定义实体，在实体的属性上添加校验规则，而在API接收数据时添加@valid关键字，这时你的实体将会开启一个校验的功能。
**@Null**
限制只能为null
**@NotNull**
限制必须不为null
**@AssertFalse**
限制必须为false
**@AssertTrue**
限制必须为true
**@DecimalMax(value)**
限制必须为一个不大于指定值的数字
**@DecimalMin(value)**
限制必须为一个不小于指定值的数字
**@Digits(integer,fraction)**
限制必须为一个小数，且整数部分的位数不能超过integer，小数部分的位数不能超过fraction
**@Future**
限制必须是一个将来的日期
**@Max(value)**
限制必须为一个不大于指定值的数字
**@Min(value)**
限制必须为一个不小于指定值的数字
**@Past**
限制必须是一个过去的日期
**@Pattern(value)**
限制必须符合指定的正则表达式
**@Size(max,min)**
限制字符长度必须在min到max之间
**@Past**
验证注解的元素值（日期类型）比当前时间早
**@NotEmpty**
验证注解的元素值不为null且不为空（字符串长度不为0、集合大小不为0）
**@NotBlank**
验证注解的元素值不为空（不为null、去除首位空格后长度为0），不同于@NotEmpty，@NotBlank只应用于字符串且在比较时会去除字符串的空格
**@Email**
验证注解的元素值是Email，也可以通过正则表达式和flag指定自定义的email格式

## @Autowired ##
### Spring中@Autowired 注解的注入规则 ###
默认根据类型，匹配不到则根据bean名字
**1.声明一个service接口**

```java
public interface HelloService {
    void sayHello();
}
```
**2.service接口的实现类，此时bean名字是 helloServiceImpl**
```java
@Service
public class HelloServiceImpl implements HelloService {
    @Override
    public void sayHello() {
        System.out.println("say hello impl");
    }
}
```
**3.增加一个Controller，注入service**
```java
// 生成一个bean，名字为 helloController
@Controller
public class HelloController {
    @Autowired
    private HelloService helloService;
    
    public void hello() {
        helloService.sayHello();
    }
}
```
**4.测试①：**
```java
public class AppTest {
    
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        HelloController controller = (HelloController) context.getBean("helloController");
        controller.hello();
    }
}
```
结果如下
say hello impl

成功将Service层的实现类注入到Controller层中，可以把步骤3 代码修改一下
```java
// 生成一个bean，名字为 helloController
@Controller
public class HelloController {
    @Autowired
    private HelloService abc;
    
    public void hello() {
        abc.sayHello();
    }
}
```
结果也是可以的，因为@Autowired 第一是按照类型去匹配的，此时IoC容器中HelloService 接口只有一个实现类，所以属性名字怎么写都没关系，都可以注入进去
**测试②：增加一个实现类，此时bean名字是 newHelloServiceImpl**
```java
@Service
public class NewHelloServiceImpl implements HelloService {
    @Override
    public void sayHello() {
        System.out.println("new say hello impl");
    }
}
```
现在IoC容器中有两个 HelloService接口的实现类，继续运行测试方法，结果为
 
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: 
Error creating bean with name 'helloController': 
Unsatisfied dependency expressed through field 'abc'; 
nested exception is org.springframework.beans.factory.NoUniqueBeanDefinitionException: 
No qualifying bean of type 'com.convict.service.HelloService' available: 
expected single matching bean but found 2: helloServiceImpl,newHelloServiceImpl

因为一个接口有多个实现，所以@Autowired 就按照属性名字去找，即找一个名字为 abc的bean注入，然而IoC容器不存在一个名字叫abc的 bean，因此报错，把属性名改为下面任意一种就可以匹配到了
```java
// 生成一个bean，名字为 helloController
@Controller
public class HelloController {
    @Autowired
    private HelloService helloServiceImpl;
    
    @Autowired
    private HelloService newHelloServiceImpl;
    
    public void hello() {
        helloServiceImpl.sayHello();
        newHelloServiceImpl.sayHello();
    }
}
```
say hello impl
new say hello impl

**测试③：**

那我就要把属性名叫 abc，同时有多个实现，而且还能注入，那么在声明组件的时候取个名字就好了，比如
```java
@Service("abc")
public class HelloServiceImpl implements HelloService {
    @Override
    public void sayHello() {
        System.out.println("say hello impl");
    }
}
```
然后Controller 注入的还是abc，结果注入成功
```java
// 生成一个bean，名字为 helloController
@Controller
public class HelloController {
    @Autowired
    private HelloService abc;
    
    public void hello() {
        abc.sayHello();
    }
}
```
say hello impl

**测试④：**

属性名叫 abc，同时有多个实现，同时可以注入，且不在注解处声明bean 的名字，那么这时候使用新的注解@Qualifier 配合@Autowired 一起使用

```java
@Controller
public class HelloController {
    @Autowired
    @Qualifier("helloServiceImpl")
    private HelloService abc;
    
    public void hello() {
        abc.sayHello();
    }
}
```
say hello impl
@Qualifier是指定 一个bean的名字

**总结：**
1.一个接口只有一个实现的情况下，属性名字怎么写都无所谓，因为按照类型匹配就只有一个bean
2.一个接口多个实现的情况下：
　　① 属性名字跟组件名字一致，组件名字可以在声明的时候指定，比如 @Service("abc")
 　　② 属性名字跟组件名字不一致，配合@Qualifier 注解指定组件名字

## @Repository 注解##
在Spring2.0之前的版本中，@Repository注解可以标记在任何的类上，用在表明该类是用来执行与数据库相关的操作（即dao对象),并支持自动处理数据库操作产生的异常
在Spring2.5版本中，引入了更多的Spring类注解：@Component，@Service，@Controller。Component是一个通用的Spring容器管理的单例bean组件。而@Repository，@Service，@Controller
因此，当你的一个类被@Component所注解，那么就意味同样可以用@Repository，@Service，@Controller来替代它，同时这些注解会具备有更多的功能，而且功能各异。
最后，如果你不知道要在项目的业务层采用@Service还是@Component注解。那么，@Service是一个更好的选择。
就如上文所说的，@Repository早已被支持了在你的持久层作为一个标记可以去自动处理数据库操作产生的异常（译者注：因为原生的java操作数据库所产生的异常只定义了几种，但是产生数据库异常的原因却有很多种，这样对于数据库操作的报错排查造成了一定的影响；而Spring拓展了原生的持久层异常，针对不同的产生原因有了更多的异常进行描述。
所以，在注解了@Repository的类上如果数据库操作中抛出了异常，就能对其进行处理，转而抛出的是翻译后的Spring专属数据库异常，方便我们对异常进行排查处理。
通过上面的话来看就是如果是数据库持久层的就使用@Repository
根据SpringDataJPA的项目，我们会首先定义一个接口，在这个接口上可能会继承CrudRepository。
如下面的代码
```java
@Repository
public interface VisaCheckeeRepository extends CrudRepository<VisaCheckee, Long> {

    VisaCheckee findVisaCheckeeByCheckeeVisaId(String checkeeVisaId);

}
```
上面的代码中，就算不使用@Repository
当然我们还是建议使用@Repository
如果你还是实现类的话，也记得把你的实现类用@Repository
![](https://user-images.githubusercontent.com/88578919/188180181-7deccad8-4958-4f95-93a2-9d29e55573a2.png)
如果，我们来看看上面的图，就能比较直观的了解@Repository
## @SessionAttributes ##
@SessionAttributes用于在会话中存储Model的属性，一般作用在类的级别。像下面的代码，model的属性user会被存储到session中，因为@ModelAttribute与@SessionAttributes有相同的注解。
```java
@Controller
@SessionAttributes("user")
public class ModelController {
    
    @ModelAttribute("user")
    public User initUser(){
        User user =new User();
        user.setName("default");
        return user;
    }
}
```
@SessionAttribute是用于获取已经存储的session数据，并且作用在方法的层面上。
```java
@RequestMapping("/session")
public String session(
    @SessionAttribute("user") User user
){
    //do something
    return "index";
}
```
### 实例 ###
1.准备@SessionAttributes的文件，用于存储session
```java
@Controller
@RequestMapping("/model")
@SessionAttributes("user")
public class ModelController {
    
    @ModelAttribute("user")
    public User initUser(){
        User user = new User();
        user.setName("default");
        return user;
    }
    @RequestingMapping("/parmeter")
    public String parameter(
        @ModelAttribute("user") User user
    ){
        return "index";
    }
}
```
2.准备@SessionAttribute的文件，用于检索session，以验证注解是否正确。
```java
@Controller
public class SessionController {
    @RequestMapping("/session")
    public String session (
@SessionAttribute("user") User user,HttpServletRequest request
    ){
        return "index";
    }
}
```
3.进行测试
不经过@SessionAttributes会直接报错
![](https://user-images.githubusercontent.com/88578919/188309273-6ee60b7b-df1c-4928-b2a0-257d11de5690.png)
先经过@SessionAttributes
首先访问/model/parameter的url
![](https://user-images.githubusercontent.com/88578919/188309283-8b40a9f4-1e23-4ba0-978e-25567734211e.png)
然后访问/session的url。这个地址，我们没有传递任何参数，可以看到从session中获取user对象成功了
![](https://user-images.githubusercontent.com/88578919/188309290-5e33dd1d-2d0a-45d6-8f76-8c3ff354ca1b.png)

## @ModelAttribute ##
### 方法使用@ModelAttribute标准 ###
**解说一**
@ModelAttribute标注可被应用在方法或方法参数上。
标准在方法上的@ModelAttribute说明方法是用于添加一个或多个属性到model上。这样的方法能接受与@RequestMapping标注相同的参数类型，只不过不能直接被映射到具体的请求上。
在同一个控制器上，标注了@ModelAttribute的方法实际上会在@RequestMapping方法之前被调用。
以下是示例：
```java
//Add one attribute
//The return value of the method is added to the model under the name "account"
//You can customize the name via @ModelAttibute("myAccount")

@ModelAttribute
pubulic Account addAccount(@RequestParam String number) {
    return accountManager.findAccount(number);
}

//Add multiple attributes

@ModelAttributes
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountManager.findAccount(number));
    //add more ...
}
```
@ModelAttribute方法通常被用来填充一些公共需要的属性或数据，比如一个下拉列表所预设的几种状态，或者宠物的几种类型，或者去取得一个HTML表单渲染所需要的命令对象，比如Account等。

@ModelAttribute标准方法有两种风格：
- 在第一种写法中，方法通过返回值的方式默认地将添加一个属性；
- 在第二种写法中，方法接收一个Model对象，然后可以向其中添加任意数量的属性。

可以在根据需要，在两种风格中选择合适的一种。
一个控制器可以拥有多个@ModelAttribute方法。同个控制器内所有这些方法，都会在@RequestMapping方法之前被调用。
@ModelAttribute方法也可以定义在@ControllerAdvice标准的类中，并且这些@ModelAttribute可以同时对许多控制器生效。

    属性名没有被显示指定的时候又当如何呢？在这种情况下，框架将根据属性的类型给予一个默认名称。举个例子，若方法返回一个Account类型的对象，则默认的属性名为“account”。可以通过设置@ModelAttribute标注的值来改变默认值。当向Model中直接添加属性时，请使用合适的重载方法addAttribute(..)-即带或不带属性名的方法。
@ModelAttribute标注也可以被用在@RequestMapping方法上。这种情况下，@RequestMapping方法的返回值将会被解释为model的一个属性，而非一个视图名，此时视图名将以视图命名约定的方式来确定。

**解说二**
首先说明一下，被@ModelAttribute注解的方法会在controller每个方法执行之前都执行，因此对于一个controller中包含多个URL的时候，要谨慎使用。

1) 使用@ModelAttribute注解无返回值的方法
```java
@Controller
@RequestMapping(value = "/modelattribute")
public class ModelAttributeController {
    
    @ModelAttribute
    public void myModel(@RequestParam(required = false) String abd, Model model) {
        model.addAttribute("attributeName",abc);
    }

    @RequestMapping(value = "/method")
    public String method() {
        return "method";
    }
}
```
这个例子，在请求/modelattribute/method?abc=aaa后，会执行myModel方法，然后执行method方法，参数abc的值被放到Model中后，接着被带到method方法中。

当返回视图/modelattribute/method时，Model会被带到页面上，当然你在使用@RequestParam的时候可以使用required来指定参数是否是必须的。

如果把myModel和method合二为一，代码如下，这也是我们常用的方法：
```java
@RequestMapping(value = "/method")
public String method(@RequestParam(requird = false) String abc, Model model) {
    model.addAttribute("attributeName",abc);
    return "method";
}
```

2) 使用@ModelAttribute注解带有返回值的方法
```java
@ModelAttribute
public String myModel(@RequestParam(required = false) String abc) {
    return abc;
}

@ModelAttribute
public Student myModel(@RequestParam(required = false) String abc) {
    Student student = new student(abc);
    return student;
}

@ModelAttribute
public int myModel(@RequestParam(required = false) int number) {
    return number
}
```
对于这种情况，返回值对象会被默认放到隐含的Model中，在Model中key为返回值首字母小写，value为返回的值。
上面3种情况等同于：
```java
model.addAttribute("string",abc);
model.addAttribute("int",number);
model.addAttribute("student",student);
```
在JSP页面使用${int}表达式时会报错：javax.el.ElException:Failed to parse the expression [${int}]。
解决方法：
在tomcat的配置文件conf/catalina.properties添加配置org.apache.el.parser.SKIP_IDENTIFIER_CHECK=true
如果只能这样，未免太局限了，我们很难接受key为string、int、float等等这样的。
想自定义其实很简单，只需要给@ModelAttribute添加value属性即可，如下：
```java
@ModelAttribute(value = "num")
public int myModel(@RequestParam(requied = false) int number) {
    return number;
}
```
这样就相当于model.addAttribute("num",number);。

### 方法参数使用@ModelAttribute标注 ###
**解说一**
标注在方法参数上的@ModelAttribute说明了该方法参数的值将由model中取得。如果model中找不到，那么该参数会先被实例化，然后被添加model中。在model中存在以后，请求中所有名称匹配的参数都会填充到该参数中。
这在Spring MVC中被称为数据绑定，一个非常有用的特性，我们不用每次都手动从表格数据中转换这些字段数据。
```java
@RequestMapping(path = "/owners/{ownerId}/pets/{petId}/edit",method = RequestMethod.POST)
public String processSubmit(@ModelAttribute Pet pet) { }
```
以上面的代码为例，这个Pet类型的实力可能来自哪里呢？有几种可能：

- 它可能因为@SessionAttribute标注的使用已经存在于model中
- 它可能因为在同个控制器使用了@ModelAttribute方法已经存在于model中————正如上一小节所叙述的
- 它可能是由URI模板变量和类型转换中取得的（下面会详细讲解）
- 它可能是调用了自身的默认构造器被实例化出来的

@ModelAttribute方法常用于从数据库中取一个属性值，该值可能通过@SessionAttributes标注在请求中间传递。在一些情况下，使用URI模板变量和类型转换的方式来取得一个属性是更方便的方式。这里有个例子：
```java
@RequestMapping(path = "/accounts/{account}", method = RequestMethod.PUT)
public String save(@ModelAttribute("account") Account account) {

}
```

这个例子中，model属性的名称("account")与URI模板变量的名称相匹配。如果配置了一个可以将String类型的账户值转换成Account类型实例的转换器Converter <String,Account>，那么上面这段代码就可以工作的很好，而不需要再额外写一个@ModelAttribute方法。

下一步就是数据的绑定。WebDataBinder类能将请求参数————包括字符串的查询参数和表单字段等————通过名字匹配到model的属性上。成功匹配的字段在需要的时候会进行一次类型转换（从String类型到目标字段的类型），然后被填充到model对应的属性中。

进行了数据绑定后，则可能会出现一些错误，比如没有提供必须的字段、类型转换过程错误等。若想检查这些错误，可以在标注了@ModelAttribute的参数紧跟着声明一个BindingResult参数：
```java
@RequestMapping(path = "/owner/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) {
    if (result.hasErrors()) {
        return "petForm";
    }

    // ...

}
```
拿到BindingResult参数后，可以检查是否有错误，可以通过Spring的<errors>表单标签来在同一个表单上显示错误信息。

BindingResult被用于记录数据过程的错误，因此除了数据绑定外，还可以把该对象传给自己定制的验证器来调用验证。这使得数据绑定过程和验证过程出现的错误可以被搜集到一起，然后一并返回给用户：
```java
@RequestMapping(path = "/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) {
    
    new PetValidator().validate(pet,result);
    if (result.hasErrors()) {
        return "petForm";
    }

    //...

}
```
又或者可以通过添加一个JSR-303规范的@Valid标注，这样验证器会自动被调用。
```java
@RequestMapping(path = "/owners/{ownerId}/pet/{petId}/edit", method = RequestMethod.POST)
public String processSubmit(@Valid @ModelAttribute("pet") Pet pet, BindingResult result) {
    
    if (result.hasErrors()) {
        return "petForm";
    }

    //...

}
```
**解说二**

```java
@Controller
@RequestMapping(value = "/modelattribute")
public class ModelAttributeParamController {

    @ModelAttribute(value = "attributeName")
    public String myModel(@RequestParam(required = false) String abc) {
        return abc;
    }

    @ModelAttribute
    public void myModel3(Model model) {
        model.addAttribute("name","zong");
        model.addAttribute("age",20);
    }

    @RequestMapping(value = "/param")
    public String param(
        @ModelAttribute("attributeName") String str,
        @ModelAttribute("name") String str2,
        @ModelAttribute("age") int str3) {
        return "param";
        }
}
```

代码中可以看出，使用@ModelAttribute注解的参数，意思是从前面的Model中提取对应名称的属性。
这里提及一下@SessionAttributes的使用：
- 如果在类上面使用了@SessionAttributes("attributeName")注解，而本类中恰巧存在attributeName，则会将attributeName放入到session作用域。
- 如果在类上面使用了@SessionAttributes("attributeName")注解，SpringMVC会在执行方法之前，自动从session中读取key为attributeName的值，并注入到Model中。所以我们在方法的参数中使用ModelAttribute("attributeName")就会正常的从Model读取这个值，也就相当于获取了session中的值。
- 使用了@SessionAttributes之后，Spring无法知道什么时候要清掉@SessionAttributes存进去的数据，如果要明确告知，也就是在方法中传入sessionstatus对象参数，并调用status.setCompete就可以了。

应用在方法上，并且方法上也使用了@RequestMapping
```java
@Controller
@RequestMapping(value = " /modelattribute")
public class ModelAttributeController {

    @RequestMapping(value = "/test")
    @ModelAttribute("name")
    public String test(@RequestParam(required =false) String name) {
        return name;
    }
}
```
这种情况下，返回值String（或者其他对象）就不再是视图了，而是放入到Model中的值，此时对应的页面就是@RequestMapping的值test。
如果类上有@RequestMapping，则视图路径还要加上类的@RequestMapping的值，本例中视图路径为modelattribute/test.jsp。

## @