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

## spring data jpa 注解 ##
### **@Entity** ###
```java
public @interface Entity {
    String name() default "";
}
```
- 被Entity标注的实体类将会被JPA管理控制，在程序运行时，JPA会识别并映射到指定的数据库表
- 唯一参数name：指定实体类名称，默认为当前实体类的非限定名称。
- 若给了name属性值即@Entity(name="XXX")，则jpa在仓储层（数据层）进行自定义查询时，所查的表名应是XXX。
- 如：select s from XXX s
  
### **@Table** ###
- 当你想生成的数据库表名与实体类名称不同时，使用@Table（name=“数据库表名”），与@Entity标注并列使用，至于实体类声明语句之前
```java
@Entity
@Table(name="t_student")
public class student{
    ...
}
```
```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface Table {
    //表的名字，可选如果不填写，系统认为和实体的名字一样为表名
    String name() default "";
    //此表的catalog，可选
    String catalog() default "";
    //此表的schema，可选
    String schema() default "";
    //唯一性约束，只有创建表的时候有用，默认不需要
    UniqueConstraint[] uniqueConstraints() default {};
    //索引，只有创建表的时候使用，默认不需要
    Index[] indexes() default {};
}
```
@Table中的参数（不常用）
- catalog：用于设置表所映射到的数据库的目录
- schema：用于设置表所映射到的数据库的模式
- uniqueConstraints：设置约束条件

### **@Id** ###
@Id用于实体类的一个属性或者属性对应的getter方法的标注，被标注的属性将映射为数据库主键

### **@GeneratedValue** ###
与@Id一同使用，用于标注主键的生成策略，通过strategy属性指定。默认是JPA自动选择合适的策略
```java
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface GeneratedValue {
    
    GeneratetoinType strategy() default GenerationType.AUTO;

    String generator() default "";
}
```

在javax.persistence.GenerationType中定义了以下几种可供选择的策略：
- IDENTITY:采用数据库ID自增长的方式产生主键，Oracle不支持这种方式。
- AUTO：JPA自动选择合适的策略，是默认选项。
- SEQUENCE：通过序列产生主键，通过@SequenceGenerator标注指定序列号，MySQL不支持这种方式。
- TABLE:通过表产生主键，框架借由表模拟序列产生主键，使用该策略更易于做数据库移植。

### **@Basic** ###
@Basic表示一个简单的属性到数据库表的字段的映射，对于没有任何标注的getXxxx()方法，默认即为@Basic（fetch=FetechType.EAGER)
```java
@Target({ElementType.METHOD,ElementType.FIELD})
@Retention(RerentionPolicy.RUNTIME)
public @interface Basic {
    //可选，EAGER为立即加载，LAZY为延迟加载
    FetchType fetch() default FetchType.EAGER;
        //可选，设置这个字段是否为null，默认为true
    boolean optional() default true;
}
```
@Basic参数：
   - fetch表示该属性的加载读取策略
      - EAGER主动抓取（默认为EAGER）
      - LAZY延迟加载，只有用到该属性时才会去加载
   - optional（默认为true）
      表示该属性是否允许为null

### **@Transient** ###
@Transient表示该属性并非一个到数据库表字段的映射，表示持久化属性，与@Basic作用相反。JPA映射数据库的时候就会忽略他。

### **@Column** ###
源码
```java
@Target({ElementType.METHOD,ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Column {
        //数据库中表的列明。可选，默认为与实体属性名保持一样，如果实体属性名中由大写，他会自动分割
    String name() default "";
        //是否唯一，默认false,可选
    boolean unique() default false;
        //数据字段是否允许为空，可选，默认为true
    boolean nullable() default true;
        //执行insert操作的时候是否包含此字段，可选，默认为true
    boolean insertable() default true;
    //执行update的时候是否包含此字段，可选，默认为true
    boolean updatable() default true;
        //表示该字段在数据库中的实际类型
    String columnDefinition() default "";
        //数据库字段的长度，可选，默认为255
    int length() default 255;
}
```
@Column注解来标识实体类中属性与数据表中字段的对应关系
从定义可以看出，@Column注解一共有10个属性，这10个属性均为可选属性，各属性含义分别如下：
**name**
name属性定义了被标注字段在数据库中所对应字段的名称：
**unique**
unique属性表示该字段是否为唯一标识，默认为false。如果表中有一个字段需要唯一标识，则既可以使用该标记，也可以使用@Table标记中的@UniqueConstraint
**nullable**
nullable属性表示该字段是否可以为null值，默认为true。
**insertable**
insertable属性表示在使用“INSERT”脚本插入数据时，是否需要插入该字段的值。
**updatable**
updatable属性表示在使用“UPDATE”脚本插入数据时，是否需要更新该字段的值。insertable和updatable属性一般多用于只读的属性，例如主键和外键等。这些字段的值通常是自动生成的。
**columnDefinition**
columnDefinition属性表示创建表时，该字段创建的SQL语句，一般用于通过Entity生成表定义时使用。（也就是说，如果DB中表已经建好，该属性没有必要使用。）
**table**
table属性定义了包含当前字段的表名
**length**
length属性表示字段的长度，当字段的类型varchar时，该属性才有效，默认为255个字符。
**precision和scale**
precision属性和scale属性表时精度，当字段类型为double时，precision表示数值的总长度，scale表示小数点所占的位数。
在使用此@Column标记时，需要注意以下几个问题：
此标记可以标注在getter方法或属性前，例如以下的两种标注方法都是正确的：
标注在属性前：
```java
@Entity
@Table(name = "contact")
public class ContactEO{

    @Column(name = "contact")
    private String name;

    public String getName(){
        return name;
    }

    public void setName(String name){
        this.name=name;
    }
}
```
标注在getter方法前：
```java
@Entity
@Table(name = "contact")
public class ConactEO{
    private String name;

    @Column(name = "contact_name")
    public String getName(){
        return name;
    };

    public void setName(String name){
        this.name=name;
    }
}
```

### **@Temporal** ###
@Temporal用来设置Date类型的属性映射到对应精准的字段。

- @Temporal(TemporalType.DATA)映射为日期--data
- @Temporal(TemporalType.TIME)映射为日期--time
- @Temporal(TemporalType.TIMESTAMP)映射为日期--data time

### **@Enumerated** ###
@Enumerated很好用，直接映射enum枚举类型的字段
源码
```java
public @interface Enumerated {
    //枚举映射的类型，默认为ORDINAL
    EnumType value() default EnumType.ORDINAL;
}
```
EnumType可选：
- ORDINAL：映射枚举的下标
- SIRING：映射枚举的name
  
示例：
```java
//定义一个枚举类
public enum Gender{
    MAIL("男性"),FMAIL("女性")

    private String value;

    private Gender(String value){
        this.value = value
    }
}
//实体类中的使用
@Entity
@Table(name = "tb_user")
public class User implements Serializable{
    @Enumerated(EnumType.STRING)
    @Column(name = "user_gender")
    private Gender gender;

}
```
这是插入两条数据，数据库里面的值是MAIL/FMAIL，而不是男性、女性，如果是我们用@Enumerated(EnumType.ORDINAL),那么这时数据库里存储的是0，1.但是实际工作中不建议这样做，因为下标使会发生变化。

### **@Lob** ###

Lob将属性映射为数据库支持的大对象类型，支持以下两种数据库类型的字段
- Clob类型是长字符串类型，java.sql.Clob、Character[]、char[]、String将被映射为Clobl类型
- Blob类型是字节类型，java.sql.Blob、Byte[]、byte[]和实现了Serializable接口的类型将被映射为Blob类型
- Clob、Blob占用内存空间较大，一般配合@Basic(fatch=FechType.LAZY)将其设置为延迟加载

### **@JoinColumn** ###

定义外键关联的字段名称
```java
public @interface JoinColumn {
        //目标表的字段名，必填
    String name() default "";
        //本实体的字段名，非必填，默认为本表ID
    String referencedColumnName() default "";
        //外键字段是否唯一，默认为false
    boolean unique() default false;
        //外键字段是否允许为空，默认为允许
    boolean nullable() default true;
        //是否跟随一起新增
    boolean insertable() default true;
        //是否跟随一起修改
    boolean updatable() default true;
}
```
用法：主要配合@OneToOne、@ManyToOne、@OneToMany一起使用，但是使用没有意义。

### **@OneToOne** ###
源码：
```java
@Target({ElementType.METHOD,ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface OneToOne {
        //关系目标实体，非必填，默认为该字段的类型
    class targetEntity() default void.class;
        //cascade 级联操作策略
        //PERSIST 级联新建
        //REMOVE 级联删除
        //REFRESH 级联刷新
        //MERGE 级联更新
        //ALL四项权限
    CascadeType[] cascade() default{};
        //数据获取方式，立即加载和延迟加载
    FetchType fetch() default FetchType.EAGER;
        //是否允许为空
    boolean optional() default true;
        //关联关系被谁维护，非必填，一般不需要特别指定
    String mappedBy() default "";
        //是否级联删除，和CascadeType.REMOVE的效果一样，只要配置了两种的一种就会自动级联删除
    boolean orphanRemoval() default false;
}
```
用法：@OneToOne需要配合@JoinColumn一起使用。注意：可以双向关联，也可以只配置一方，需要视需求而定。
示例：假设一个部门只有一个员工
```java
@OneToOne
@JoinColumn(name = "employee_id",referencedColumnName = "id")
private Employee employeeAttribute = new Employee();
```
employee_id指的是Department里面的字段，而referencedColumnName=“id”指的是Employee表里面的字段
如果需要双相关联，Employee的内容如下
```java
@OneToOne(mappedBy = "employeeAttribute")
private Department department;
```
当然不使用mappedBy，和下面效果一样
```java
@OneToOne
@JoinColumn(name = "id",referencedColumnName = "employee_id")
private Employee employeeAttribute = new Employee();
```
### **@OneToMany与@ManyToOne** ###
@OneToMany与@ManyToOne可以相对存在，也可以只存在一方
```java
@Entity
@Table(name = "user")
public class User {
    @Id
    private Long id;

    @OneToMany(
            cascade = CascadeType.ALL,
            fetch = FetchType.LAZY,
            mappedBy = "user"
    )
    private Set<Role> setRole;

}

@Entity
@Table(name = "role")
public class Role {
    @Id
    private Long id;

    @ManyToOne(
            cascade =  CascadeType.ALL,
            fetch = FetchType.EAGER
    )
    @JoinColumn(name = "user_id")
    private User user;
}
```

### **@OrderBy** ###
```java
public @interface OrderBy {
    //要排序的字段，格式如下
    //orderby_list::=orderby_item[,orderby_item]*
    //orderby_item::=[property_or_field_name][ASC][DESC]
    String value() default "";
}
```
用法
```java
@Entity
@Table(name = "user")
public class User {
    @Id
    private Long id;

    @OneToMany(
            cascede = CascadeType.ALL,
            fetch = FetchType.LAZY,
            mappedBy = "user"
    )
    @OrderBy("roleName DESC")
    private Set<Role> setRole;
}
```
OrderBy中的字段对应的是实体中的名称

### **@JoinTable** ###
如果对象与对象之间有一个关联表的时候，就会用到@JoinTable,一般和@ManyToMany一起使用。
源码：
```java
public @interface JoinTable {

    String name() default "";

    String catalog() default "";

    String schema() default "";

    JoinColumn[] joinColumns9 default {};

    JoinColumn[] inverseJoinColumns() default {};
}
```
示例：
假设Blog和Tag是多对多的关系，有一个关联关系表blog_tag_relation，表中有两个属性blog_id和tag_id，那么Blog尸体里面的写法如下：
```java
@Entity
public class Blog {
    @Id
    private Long id;
    @ManyToMany
    @JoinTable(
            name = "blog_tag_relation",
            joinColumns = @JoinColumn(name = "blog_id",referencedColumnName = "id"),
            inverseJoinColumns = @JoinColumn(name = "tag_id",referencedColumnName = "id")
    )
    private List<Tag> tags = new ArrayList<>();
}
```

### **@ManyToMany** ###

源码：
```java
public @interface ManyToMang {
    class targetEntity() default void.class;

    CascadeType[] cascade() dafault {};

    FetchType fetch() default FetchType.LAZY;

    String mappedBy() default "";
}
```
@ManyToMany表示多对多，和@OneToOne、@ManyToOne一样也有单向、双向之分。单向双向和注解没有关系，只看实体类之间是否相互引用。
示例：一个博客可以有多个标签，一个标签也可以在多个博客上，Blog和Tag就是多对多的关系
```java
//单向多对多
@Entity
public class Blog {
    @Id
    private Long id;
    @ManyToMany
    @JoinTable(
            name = "blog_tag_relation",
            joinColumns = @JoinColumn(name = "blog_id", referencedColumnName = "id"),
            inverseJoinColumns = @JoinColumn(name = "tag_id", referencedColumnName = "id")
    )
    private List<Tag> tags = new ArrayList<>();
}

@Entity
public class BlogTagRelation {
    @Column(name = "blog_id")
    private Integer blogId;

    @Column(name = "tag_id")
    private Integer tag_id
}

//双向多对多
@Entity
public class Blog {
    @Id
    private Long id;
    @ManyToMany(cascade=CascadeType.ALL)
    @JoinTable(
            name = "blog_tag_relation",
            joinColumns = @JoinColumn(name = "blog_id", referencedColumnName = "id"),
            inverseJoinColumns = @JoinColumn(name = "tag_id", referencedColumnName = "id")
    )
    private List<Tag> tags = new ArrayList<>();
}
@Entity
public class Tag {
    @Id
    private String id;

    @ManyToMany(mappedBy = "BlogTagRelation")
    private List<Blog> blog = new ArrayList<>();
}
```

### **@EntityGraph** ###
JPA2.1推出@EntityGraph、@NamedEntityGraph用来提高查询效率，很好地解决了N+1条SQL的问题。两者配合起来使用，缺一不可。
1.现在Entity里面定义了@NameEntityGraph，其他不变，其中，@NamedAttributeNode可以有多个，也可以有一个
```java
@NamedEntityGraph(
        name = "UserInfoEntity.addressEntityList",
        attributeNodes = {
            @NamedAttributeNode("addressEntityList"),
            @NamedAttributeNode("userBlogEntityList")
        }
)
@Entity(name = "UserInfoEntity")
@Table(name = "User_info", schema = “test”)
public class UserInfoEntity implements Serializable {
    @Id
    @Column(name = "id", nullable = true)
    private Integer id;

    @OneToOne(optional = true)
    @JoinColumn(
            referencedColumnName = "id",
            name = "address_id",
            nullable = false
    )
    private UserReceivingAddressEntity addressEntityList;

    @OneToMany
    @JoinColumn(name = "create_user_id", referencedColumnName = "id")
    private List<UserBlogEntity> userBlogEntityList;
}
```
2.只需要在查询方法上加@EntityGraph注解即可，其中value就是@NamedEntityGraph中的name。
```java
public interface UserRepository extends JpaRepository<User,Long> {
    @Override
    @EntityGraph(value = "UserInfoEntity.addressEntityList")
    List<User> findAll();
}
```
### **@PrePersist注解** ###
在使用JPA对数据库进行操作的时候，我们时常会出现数据库字段设置末不能为空，而我们保存的字段为null导致程序报错，这个时候我们就可以使用@Prepersist@PostPersist注解回调方法来解决问题
回调方法是附加到实体生命周期事件的用户定义方法，并且在发生这些事件时由JPA自动调用。
我们可以发现有很多类似的注解可以使用：
- @Prepersist 在新实体持久化之前（添加到EntityManager）
- @PostPersist 在数据库中存储新实体（在commit或期间flush）
- @PostLoad 从数据库中检索实体后。
- @PreUpdate 当一个实体被识别为被修改时EntityManager
- @PostUpdate 更新数据中的实体（在commit或期间flush）
- @PreRemove 在EntityManager中标记要删除的实体时
- @PostRemove 从数据库中删除实体（在commit或期间flush）
```java
@PostPersist
public void setTel() {
    //当我们要报错电话号码时，如果该实体的电话为null，则设置默认值。这样我们程序就不会报错
    if(null == this.Tel){
        this.Tel = "";
    }
}
```
```java
@PrePersist
public void setDefaultUpdateTime() {
    //在业务中无关的数据给一个默认值保存在数据库中。
    this.updateTime = new Date();
}
```
```java
@Data
@ToString
@MappedSuperclass
@EqualsAndHashCode
public class BaseEntity {

    /**
     * Create time.
     */
    @Column(name = "create_time")
    @Temporal(TemporalType.TIMESTAMP)
    private Date createTime;

    /**
     * Update time.
     */
    @Column(name = "update_time")
    @Temporal(TemporalType.TIMESTAMP)
    private Date updateTime;

    @PrePersist
    protected void prePersist() {
        Date now = DateUtils.now();
        if (createTime == null) {
            createTime = now;
        }

        if (updateTime == null) {
            updateTime = now;
        }
    }

    @PreUpdate
    protected void preUpdate() {
        updateTime = new Date();
    }

    @PreRemove
    protected void preRemove() {
        updateTime = new Date();
    }

}
```
### **@Query** ###

- 只需要将@Query标记在继承了Repository的自定义接口的方法上，就不再需要遵循查询方法命名规则。
- 支持命名参数及索引参数的使用
- 本地查询

**案例**
- 查询Id最大的员工信息
```java
@Query("select o from Employee o where id=(select max(id) from Employee t1)")
    Employee getEmployeeById();
```
注意：Query语句中第一个Employee是类名

测试类：
```java
@Test
    public void getEmployyeeByMaxId() throws Exception {
        Employee employee = employeeRepository.getEmployeeByMaxId();
        System.out.println("id:" + employee.getId() + " name: " + employee.getName() + "age:" + employee.getAge());
    }
```

- 根据占位符进行查询
注意：占位符从1开始
```java
@Query("select o from Employee o where o.name=?1 and o.age=?2")
    List<Employee> queryParams1(String name, Integer age);
```
测试方法：
```java
@Test
    public void queryParams1() throws Exception {
        List<Employee> employees = employeeRepository.queryParams1("zhangsan", 20);
        for (Employee employee : employees) {
            System.out.println("id:" +employee.getId() + " name: " + employee.getName() + " age:" + employee.getAge());
        }
    }
```

- 根据命名参数的方式
```java
@Query("select o from Employee o where o.name=:name and o.age=:age")
    List<Employee> queryParams2(@Param("name") String name, @Param("age") Integer age);
```
- like查询语句
```java
@Query("select o from Employee o where o.name like %?1%")
    List<Employee> queryLike1(String name);
@Test
    public void queryLike1() throw Exception {
        List<Employee> employees = employeeRepository.queryLike1("test");
        for (Employee employee : employees) {
            System.out.println("id:" + employee.getId() + " name:" +employee.getName() + "age:" + employee.getAge());
        }
    }
```

- like语句使用命名参数
```java
@Query("select o from Employee o where o.name like %:name%")
    List<Employee> queryLike2(@Param("name") String name);
```
**本地查询**
所谓本地查询，就是使用原生的SQL语句进行查询数据库的操作。但是在Query中原生态查询默认是关闭的，需要手动设置为true：
```java
@Query(nativeQuery = true, value = "select count(1) from employee")
    long getCount();
```
更新操作整合事物使用
- 在DAO中定义方法根据Id来更新年龄（Modifying注解代表允许修改）
```java
@Modifying
    @Query("update Employee o set o.age = :age where o.id =:id")
    void update(@Param("id") Integer id, @Param("age") Integer age);
```
要注意，执行更新或者删除操作是需要事务支持，所以通过service层来增加事物功能，在update方法上添加Transactional注解。
```java
package com.xx.service;

import com.xx.repository.EmployeeRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.sterotype.Service;

@Service
public class EmployeeService {
    
    @Autowired
    private EmployeeRepository employeeRepository;

    @Transactional
    public void update(Integer id, Integer age) {
        employeeRepository.update(id,age);
    }
}
```

- 删除操作
删除操作同样需要Query注解，Modifying注解和Transactional注解
```java
@Modifying
    @Query("delete from Employee o where o.id = :id")
    void delete(@Param("id") Ingeter id);

    @Transactional
    public void delete(Integer id) {
        employeeRepository.delete(id);
    }
```

### **@Transactional注解** ###

@Transactional注解可以作用于接口、接口方法、类以及类方法上
1.当作用于类上上时，该类的所有public方法将都具有该类型的事务属性
2.当作用在方法级别时会覆盖类级别的定义
3.当作用在接口和接口方法时则只有在使用基于接口的代理时它才会生效，也就是JDK动态代理，而不是Cglib代理
4.当在protected、private或者默认可见性的方法上使用@Transactional注解时是不会生效的，也不会抛出任何异常
5.默认情况下，只有在来自外部的方法调用才会被AOP代理捕获，也就是，类内部方法调用本类内部的其他方法并不会引起事务行为，即使被调用方法使用@Transactional注解进行修饰

**@Transactional注解的可用参数**

**readonly** 
该属性用于设置当前事务是否为只读事务，设置为true表示只读，false则表示可读写，默认值为false

**rollbackFor** 
该属性用于设置需要进行回滚的异常类名称数组，当方法中抛出指定异常数组中的异常时，则进行事务回滚。例如：
1.指定单一异常类：@Transactional(rollbackFor=RuntimeException.class)
2.指定多个异常类：@Transactional(rollbackFor={RuntimeException.class,BusnessException.class})

**rollbackForClassName** 
该属性用于设置需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，则进行事务回滚。例如：
1.指定单一异常类名称：@Transactional(rollbackForClassName="RuntimeException")
2.指定多个异常类名称：@Transactional(rollbackForClassName={"RuntimeException","BusnessException"})

**noRollbackFor** 
该属性用于设置不需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，不进行事务回滚

**noRollbackForClassName** 
参照上方例子

**timeout** 
该属性用于设置事物的超时秒数，默认值为-1表示永不超时

**propagation** 
该属性用于设置事物的传播行为
例如：@Transactional(propagation=Propagation.NOT_SUPPORTED)
事物传播行为介绍：
1.@Transactional(propagation=Propagation.REQUIRED)如果有事务，没有的话新建一个(默认)
2.@Transactional(propagation=Propagation.NOT_SUPPORTED)容器不为这个方法开启事务
3.@Transactional(propagation=Propagation.REQUIRES_NEW)不管是否存在事务，都创建一个新的事务，原来的挂起，新的执行完毕，继续执行老的事务
4.@Transactional(propagation=Propagation.MANDATORY)必须在一个已有的事物中执行，否则抛出异常
5.@Transactional(propagation=Propagation.NEVER)必须在一个没有的事务中执行，否则抛出异常（与propagation.MANDATORY相反
6.@Transactional(propagation=Propagation.SUPPORTS)如果其他bean调用这个方法，在其他bean中声明事务，那就用事务，如果其他bean没有声明事务，那就不用事务<br>

**isolation**  
该属性用于设置底层数据库的事务隔离级别
事务隔离级别介绍
1.@Transactional(isolation = Isolation.READ_UNCOMMITTED)读取未提交数据（会出现脏读，不可重复读）基本不使用<br>
2.@Transactional(isolation = Isolation.READ_COMMITTED)读取已提交数据（会出现不可重复读和幻读）<br>
3.@Transactional(isolation = Isolation.REPEATABLE_READ)可重复读（会出现幻读）<br>
4.@Transactional(isolation = Isolation.SERIALIZABLE)串行化<br>
什么是脏读、幻读、不可重复读？
1.脏读：一个事务读取到另一个事务未提交的更新数据
2.不可重复读：在同一事务中，多次读取同一数据返回的结果有所不同，换句话说，后续读取可以读到另一事务已提交的更新数据。相反，“可重复读”在同一事务中多次读取数据时，能够保证所读数据一样，也就是后续读取不能读到另一个事务已提交的更新数据
3.幻读：一个事务读到另一个事务已提交的insert数据