#Lombok库

- **@Getter和@Setter**
放在类上，为该类的所有属性自动生成Getter和Setter方法

```java
import lombok.Getter;
import lombok.Setter;
​
@Getter
@Setter
public class Student {
    int id;
    String name;
    int age;
}
```

- **@ToString**
自动重写toString方法，和我们平时使用IDEA编辑器自动生成的一样

```java
@ToString
public class Student {
    int id;
    String name;
    int age;
}
```

- **@EqualsAndHashCode**
自动生成equal(Object other)和hashCode()方法，如果某些变量不想要加入该注解，可以使用exclude进行排除

```java
import lombok.EqualsAndHashCode;
​
@EqualsAndHashCode
public class Student {
    int id;
    String name;
    int age;
}
import lombok.EqualsAndHashCode;
​
//把name属性排除在外
@EqualsAndHashCode(exclude = "name")
public class Student {
    int id;
    String name;
    int age;
}
```
问：为什么把生成equal(Object other)和hashCode()方法弄成一个注解，而不是分开使用？

答：在Java中有规定：当两个对象相等时，它们的hashcode是一定相等的。但是，当两个对象的hashcode相同，对象不一定相等。这样做是为了防止违反Java规定的情况发生。 
- **@NoArgsConstructor**
生成一个不包含任何参数的无参构造器
```java

import lombok.NoArgsConstructor;
​
@NoArgsConstructor
public class Student {
    int id;
    String name;
    int age;
}
```
源码：
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface NoArgsConstructor {

    //设置是否生成以及生成的静态构造方法的名称
    String staticName() default "";

    //设置构造方法注解
    NoArgsConstructor.AnyAnnotation[] onConstructor() default{};

    //设置构造方法的访问权限，默认是public
    AccessLevel access() default AccessLevel.PUBLIC;

    //设置强制对未赋值的final字段初始化值。
    boolean force() default false;

    @Deprecated
    @Retention(RetentionPolicy.SOURCE)
    @Target({})
    public  @interface AnyAnnotation {
    }
}
```
根据源码可知，@NoArgsConstructor注解共有staticName、onConstructor、access以及force四个属性参数可以进行配置：

- **staticName**

staticName代表的是是否生成静态构造方法，也就是说当staticName属性有值时则会生成一个静态构造方法，这时无参构造方法会被私有，然后创建一个指定名称的静态构造方法，并且是公有的，如下所示：
```java
/**
编译前代码
*/
@NoArgsConstructor(staticName = "UserStatic")
public class UserInfo() {

    private String username;
    private String password;
}
/**
编译后代码
*/
public class UserInfo(){
    
    private String username;
    private String password;

    private UserInfo(){
    }
    private UserInfo(){
        return new UserStatic();
    }
}
```

- **onConstructor**

经常写Spring或者SpringBoot代码的人应该知道，Spring对于依赖注入提供了三种写法，分别是属性注入、Setter方法注入以及构造器注入，但是在日常工作中我们更多采用的是依赖于@Autowired注解方式进行依赖注入，不过过多的依赖注入会使我们的代码过于冗长，甚至Spring4.0起就已经开始不推荐这种写法了，而是推荐使用Setter方法注入以及构造器注入，lombok的生成构造器的方法就可以很方便的实现这种写法。
举一个通过构造器注入的例子：
```java
@Controller
public class SysLoginController() {
    
    private final TokenUtils tokenUtils;

    private final SysLoginService sysLoginService;

    /**
    在这里@Autowired是可以省略的，在这里使用只是为了介绍onConstructor参数
    */
    @autowired
    public SysLoginController (TokenUtils tokenUtils, SysLoginService sysLoginService) {

        this.tokenUtils = tokenUtils;
        this.sysLoginService = sysLoginService;
    }
}
```
这样注入Bean在数量较多时我们仍需编写大量代码，这个时候就可以使用@RequiredArgsConstructor注解来解决这个问题，至于为什么不使用@AllArgsConstructor注解是因为这个注解是针对所有参数的，而在这个情境下，我们只需构造Bean所对应的属性而不是非Bean，所以我们只需在Bean对应的属性前加上final关键字进行修饰就可以只生成需要的有参构造函数，如下所示：
```java
/**
编译前
*/
@NoConstructor(onConstructor = @_(@Autowired))
public class SysLoginController() {

    private final TokenUtils tokenUtils;

    private final SysLoginService sysLoginService;
}

/*
编译后
*/
public class SysLoginController() {
    
    private final TokenUtils tokenUtils;

    private final SysLoginService sysLoginService;

    @Autowired
    public SysLoginContorller (TokenUtils tokenUtils, SysLoginService sysLoginService) {
        
        this.tokenUtils = tokenUtils;
        this.sysLoginService = sysLoginService;
    }
}
```

- **access**

有的时候我们会使用单例模式，这个时候需要我们创造一个私有的无参构造方法，那么就可以使用access这样一个属性来设置构造器的权限，如下所示：
```java
/*
编译前
*/
@NoConstructor(access = AccessLevel.PRIVATE)
public class UserInfo() {
    private String username;
    private String password;
}

/*
编译后
*/
public class UserInfo() {
    private String username;
    private String password;

    private UserInfo() {
    }
}
```
access的可选等级
```java
public enum AccessLevel {
    PUBLIC,
    MODULE,
    PROTECTED,
    PACKAGE,
    PRIVATE,
    NONE;

    private AccessLevel() {
    }
}
```

- **force**

当类中有被final关键字修饰的字段未被初始化时，编译器会报错，这时也可以设置force属性为true来为字段根据类型生成一个默认值为0/false/null，这样编译器就不会再报错了，如下所示：
```java
/*
编译前
*/
@NoConstructor(force = true)
public class UserInfo() {
    
    private final String gender;
    private String username;
    private String password;
}
/*
编译后
*/
public class UserInfo() {
    private final String gender = null;
    private String username;
    private String password;

    private UserInfo() {
    }
}
```

- **@AllArgsConstructor**
生成一个包含所有参数的构造器

```java
import lombok.AllArgsConstructor;
​
@AllArgsConstructor
public class Student {
    int id;
    String name;
    int age;
}
```
如何使用
@ALLArgsConstructor在类上使用，这个注解可以生成全参构造函数，且默认不生成无参构造函数。
不过需要注意的是，这里所说的全参并不包括已经被初始化的被final关键字修饰的字段，因为字段一旦被final关键字修饰被赋值后就不能在被修改，如下所示：
```java
/*
编译前
*/
@AllArgsConstructor
public class UserInfo() {

    private final String gender;
    private final Integer age = 18;

    private String username;
    private String password;
}
/*
编译后
*/
public class UserInfo() {
    
    private final String gender;
    private final Integer age = 18;

    private String username;
    private String password;

    public User(String gender, String username, String password) {
        
        this.gender = gender;
        this.username = username;
        this.password = password;
    }
}
```
源码详解
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface AllArgsConstructor {
    String staticName() default "";

    AllArgsConstructor.AnyAnnotation[] onConstructor() default {};

    AccessLevel access() default AccessLevel.PUBLIC;

    @Deprecated
    @Retention(RetionPolicy.SOURCE)
    @Target({})
    public @interface AnyAnnotation {
    }
}
```
参数配置参照@NoArgsConstructor源码详解

- **@RequiredArgsConstructor**
为“特定参数”生成构造器，这里的“特定参数”，特指那些加上final修饰词的属性

```java
import lombok.RequiredArgsConstructor;
​
@RequiredArgsConstructor
public class Student {
    int id;
    final String name;
    int age;
​
    public static void main(String[] args) {
        Student student = new Student("33");
    }
}
```
这里我们只为name加上final修饰，可以发现，我们只生成了一个包含name属性的构造器。另外，如果所有的属性都没有final修饰的话，使用@RequiredArgsConstructor会生成一个无参的构造器。
如何使用
@RequiredArgsConstructor在类上使用，这个注解可以生成带参或者不带参的构造方法。
若带参数，只能是类中所有带有@NonNull注解的和以final修饰的未经初始化的字段，如下所示：
```java
/*
编译前
*/
@RequiredArgConstructor
public class UserInfo() {

    private final String gender;
    @NonNull
    private String username;
    private String password;
}
/*
编译后
*/
public class UserInfo() {
    
    private final String gender;
    @NonNull
    private String username;
    private String password;

    public UserInfo(String gender, @NonNull String username) {

        if (username == null) {
            throws new NullPointerException("username is marked @NonNull but is null");
        } else {
            this.gender = gender;
            this.username = username;
        }
    }
}
```
源码详解
源码：
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface RequiredArgsConstructor {
    
    String staticName() default "";

    RequiredArgsConstructor.AnyAnnotation[] onConstructor() default{};
    
    AccessLevel access() default AccessLevel.PUBLIC;

    @Deprecated
    @Retention(RetentionPolicy.SOURCE)
    @Target({})
    public @interface AnyAnnotation {
    }
}
```
参数配置参照@NoArgConstructor源码详解

- **@Data**
这是一个组合注解，加了这个注解，相当于加入了@Getter、@Setter、@ToString、@EqualsAndHashCode和@RequiredArgsConstructor这五个注解。

- **@Value**
这也是一个组合注解，但是会把所有的变量都设置为final的，其他的就和@Data一样了。等同于加入了@Getter、@ToString、@EqualsAndHashCode和@RequiredArgsConstructor这四个注解（由于所有属性是final的，所以没有@setter注解了）。

- **@Builder**
流式的set值写法，不过毕竟是给属性赋值，基本的setter还是需要有的，一般来说，@Builder会和@Data一起使用。
```java
import lombok.Builder;
import lombok.Data;

@Builder
@Data
public class Student {
    int id;
    String name;
    int age;

    public static void main(String[] args) {
    Student student = 
    Student.builder().id(1).name("water").age(18).build();
	}
}
```

- **@Slf4j**
自动生成该类的log静态常量，就可以直接打印日志了，不用去new一个log的静态常量了。

```java
@Slf4j
public class Student {
    int id;
    String name;
    int age;

    public static void main(String[] args) {
        log.info("hello world");
    }
}
```
```java
@Slf4j
public class DesignTacoController {}
类似于
private static final org.slf4j.Logger log =
org.slf4j.LoggerFactory.getLogger(DesignTacoController.class);
```

**有Lombok写法**

动态生成getter和setter方法，以及equals()、hashCode()、toString()等方法
类级别的@Data注解就是由Lombok提供的，它会告诉Lombok生成所有缺失的方法，同时还会生成所有以final属性作为参数的构造器
```java
import lombok.Data;
import lombok.RequiredArgsConstructor;
@Data
@RequiredArgsConstructor
public class Ingredient {
 private final String id;
 private final String name;
 private final Type type;
 public static enum Type {
 WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
 }
}
 ```

**无Lombok写法**

```java
import java.util.Objects;

public class Ingredient {
    private final String id;
    private final String name;
    private final Type type;
    public static enum Type{
        WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
    }

    Ingredient(String id, String name, Type type) {
        this.id = id;
        this.name = name;
        this.type = type;
    }

    public String getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public Type getType() {
        return type;
    }

    @Override
    public String toString() {
        return "Ingredient{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", type=" + type +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Ingredient that = (Ingredient) o;
        return id.equals(that.id) && name.equals(that.name) && type == that.type;
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name, type);
    }
}
 ```
