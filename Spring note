#Spring MVC的请求映射注解
|注解|描述|
|---|:---:|
|@RequestMapping|通用的请求处理|
|@GetMapping|处理HTTP GET请求|
|@PostMapping|处理HTTP POST请求|
|@PutingMapping|处理HTTP PUT请求|
|@DeleteMapping|处理HTTP DELETE请求|
|@PatchMapping|处理HTTP PATCH请求|

#注解解释
##@SpringBootApplication是一个组合注解，它组合了3个其他的注解。

- @SpringBootConfiguration：将该类声明为配置类。这个注解实际上是@Configuration注解的特殊形式。
- @EnableAutoConfiguration：启用Spring Boot的自动配置。
- @ComponentScan：启用组件扫描。这样我们能够通过像@Component、@Controller、@Service这样的注解声明其他类，Spring会自动发现它们并将它们注册为Spring应用上下文中的组件。
 
##@Valid 注解类型
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
