## HTTP协议





## Maven

Maven基于**项目对象模型**（POM，Project Object Model）用于管理和构建java项目。Maven能够：（**依赖管理**）方便快捷地管理项目依赖的资源，避免版本冲突问题；（**统一项目结构**）提供标准、统一的项目结构；（**项目构建**）Maven提供了标准的跨平台的自动化项目构建方式。

### 1. Maven概述

#### 1.1 Maven模型

- 项目对象模型 (Project Object Model)：每个项目都有一个唯一标识（坐标），通过坐标可以定位所需资源位置。坐标由groupId，artifactId，version三个标签组成。

- 依赖管理模型(Dependency)

- 构建生命周期/阶段(Build lifecycle & phases)：完成标准化构建流程。

![24032301](.\resources\24032301.png)

## 过滤器和拦截器

### 1. 过滤器

过滤器Filter是JavaWeb三大组件（Servlet、Filter、Listener）之一。过滤器可以把对资源的请求拦截下来，从而实现一些特殊的功能。使用了过滤器之后，要想访问web服务器上的资源，必须先经过滤器，过滤器处理完毕之后，才可以访问对应的资源。过滤器一般完成一些通用的操作，比如：登录校验、统一编码处理等。

#### 1.1 使用过滤器

定义过滤器，实现**Filter**接口。

- init方法：过滤器的初始化方法。在web服务器启动的时候会自动地创建Filter过滤器对象，在创建过滤器对象的时候会自动调用init初始化方法，这个方法只会被调用一次。
- **doFilter方法**：这个方法是在每一次拦截到请求之后都会被调用，所以这个方法是会被调用多次的，每拦截到一次请求就会调用一次doFilter()方法。
- destroy方法： 当我们关闭服务器的时候，它会自动的调用销毁方法destroy，这个销毁方法也只会被调用一次。

```Java
@WebFilter(urlPatterns = "/*")
public class LoginFilter implements Filter{
    @Override //初始化方法, 只调用一次
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override //拦截到请求之后调用, 调用多次
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        // 放行操作
        chain.doFilter(request, response);
    }

    @Override //销毁方法, 只调用一次
    public void destroy() {
    }
}
```

在定义完Filter之后，Filter其实并不会生效，还需要完成Filter的配置，需要在Filter类上添加一个**@WebFilter注解**，并指定属性**urlPatterns**，通过这个属性指定过滤器要拦截哪些请求。

当我们在Filter类上面加了@WebFilter注解之后，接下来我们还需要在启动类上面加上一个**@ServletComponentScan**注解，通过这个@ServletComponentScan注解来开启SpringBoot项目对于Servlet组件的支持。

#### 1.2 过滤器拦截路径

过滤器Filter可以根据需求，配置不同的拦截资源路径：

| 拦截路径     | urlPatterns值 | 含义                              |
| :----------- | :------------ | :-------------------------------- |
| 拦截具体路径 | /login        | 只有访问/login路径时，才会被拦截  |
| 目录拦截     | /emps/*       | 访问/emps下的所有资源，都会被拦截 |
| 拦截所有     | /*            | 访问所有资源，都会被拦截          |

#### 1.3 过滤器链

在一个web应用程序当中，可以配置多个过滤器，多个过滤器就形成了一个过滤器链。

这个链上的过滤器在执行的时候会一个一个的执行，会先执行第一个Filter，放行之后再来执行第二个Filter，如果执行到了最后一个过滤器放行之后，才会访问对应的web资源。访问完web资源之后，还会回到过滤器当中来执行过滤器放行后的逻辑，而在执行放行后的逻辑的时候，顺序是反着的。

以注解方式配置的Filter过滤器，它的执行优先级是按过滤器类名的自然排序确定的，类名排名越靠前，优先级越高。

### 2. 拦截器

拦截器Interceptor是一种动态拦截控制器方法执行的机制，类似于过滤器。拦截器在指定方法调用前后，根据业务需要执行预先设定的代码。

#### 2.1 使用拦截器

定义拦截器，实现**HandlerInterceptor**接口。

- preHandle方法：目标资源方法执行前执行， true，放行；false，不放行。

- postHandle方法：目标资源方法执行后执行。

- afterCompletion方法：视图渲染完毕后执行，最后执行。

```Java
@Component
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {       
        return true; //true，放行；false，不放行
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    }

    //视图渲染完毕后执行，最后执行
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    }
}
```

**注册配置拦截器**，实现**WebMvcConfigurer**接口，并重写**addInterceptors**方法。

```Java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    //自定义的拦截器对象
    @Autowired
    private LoginInterceptor loginInterceptor;
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
       //注册自定义拦截器对象，设置拦截器拦截的请求路径，/** 表示拦截所有请求
        registry.addInterceptor(loginCheckInterceptor).addPathPatterns("/**");
    }
}
```

#### 2.2 拦截路径

在注册配置拦截器的时候，我们要指定拦截器的拦截路径，通过`addPathPatterns("拦截路径")`方法，就可以指定要拦截哪些资源，`/**`表示拦截所有资源。在配置拦截器时，不仅可以指定要拦截哪些资源，还可以指定不拦截哪些资源，只需要调用`excludePathPatterns("不拦截路径")`方法，指定哪些资源不需要拦截。

在拦截器中除了可以设置`/**`拦截所有资源外，还有一些常见拦截路径设置：

| 拦截路径  | 含义                 | 举例                                                |
| :-------- | :------------------- | :-------------------------------------------------- |
| /*        | 一级路径             | 能匹配/depts，/emps，/login，不能匹配 /depts/1      |
| /**       | 任意级路径           | 能匹配/depts，/depts/1，/depts/1/2                  |
| /depts/*  | /depts下的一级路径   | 能匹配/depts/1，不能匹配/depts/1/2，/depts          |
| /depts/** | /depts下的任意级路径 | 能匹配/depts，/depts/1，/depts/1/2，不能匹配/emps/1 |

#### 2.3 拦截器执行流程

![24032401](.\resources\24032401.png)

当我们打开浏览器访问部署在web服务器中的web应用时，此时所定义的过滤器会拦截到这次请求。拦截到这次请求之后，它会先执行放行前逻辑，然后再执行放行操作。由于我们是基于Spring Boot开发，所以放行之后是进入到了Spring的环境当中，也就是访问controller当中的接口方法。

Tomcat并不识别所编写的Controller程序，但是它识别Servlet程序，所以在Spring的web环境中提供了一个非常核心的Servlet：**DispatcherServlet**（前端控制器），所有请求都会先进行到DispatcherServlet，再将请求转给Controller。

当我们定义了拦截器后，会在执行Controller的方法之前，请求被拦截器拦截住，执行`preHandle()`方法。这个方法执行完成后需要返回一个布尔类型的值，如果返回true，就表示放行本次操作，才会继续访问controller中的方法；如果返回false，则不会放行（controller中的方法也不会执行）。

controller中的方法执行完毕之后，再回过来执行`postHandle()`方法以及`afterCompletion()` 方法，再返回给DispatcherServlet，最终再来执行过滤器当中放行后的这一部分逻辑，执行完毕之后，最终给浏览器响应数据。

**过滤器和拦截器之间的区别**：

- 接口规范不同：过滤器需要实现Filter接口，而拦截器需要实现HandlerInterceptor接口；
- 拦截范围不同：过滤器Filter会拦截所有的资源，而Interceptor只会拦截Spring环境中的资源。

### 3. 全局异常处理器

#### 3.1 使用全局异常处理器

定义一个异常处理器类，在类上加上一个注解**@RestControllerAdvice**。@RestControllerAdvice = @ControllerAdvice + @ResponseBody。

```Java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)  //指定处理的异常类型
    public Result ex(Exception e){
        e.printStackTrace();
        return Result.error("对不起，操作失败，请联系管理员。");
    }
}
```



## Spring事务管理

### 1. 事务

**事务**是一组操作的集合，它是一个不可分割的工作单位。事务会把所有的操作作为一个整体，一起向数据库提交或者是撤销操作请求。所以这组操作要么同时成功，要么同时失败。事务具有四个特征：原子性Atomicity、一致性Consistency、隔离性Isolation和持续性Durability。

事务的操作主要有三步：

1. 开启事务（一组操作开始前，开启事务）：start transaction / begin;
2. 提交事务（这组操作全部成功后，提交事务）：commit;
3. 回滚事务（中间任何一个操作出现异常，回滚事务）：rollback;

### 2. Spring事务管理

#### 2.1 @Transactional注解

**@Transactional**注解使得当前这个方法执行开始之前开启事务，方法执行完毕之后提交事务。如果在这个方法执行的过程当中出现了异常，就会进行事务的回滚操作。@Transactional注解书写位置：**方法**，当前方法交给Spring进行事务管理；**类**，当前类中所有的方法都交由Spring进行事务管理；**接口**，接口下所有的实现类当中所有的方法都交给Spring进行事务管理。

可以在application.yml配置文件中开启事务管理日志，这样就可以在控制台看到和事务相关的日志信息了

```yaml
logging:
  level:
    org.springframework.jdbc.support.JdbcTransactionManager: debug
```

默认情况下只有出现RuntimeException才进行回滚操作，**rollbackFor**属性用于控制出现何种异常类型时回滚事务，`@Transactional(rollbackFor=Exception.class)`。

#### 2.2 事务的传播行为

事务的传播行为指的是是当一个事务方法被另一个事务方法调用时，这个事务方法应该如何进行事务控制。@Transactional注解的**propagation**属性用于指定事务的传播行为。

| **属性值**       | **含义**                                                     |
| ---------------- | ------------------------------------------------------------ |
| **REQUIRED**     | 【默认值】需要事务，有则加入，无则创建新事务                 |
| **REQUIRES_NEW** | 需要新事务，无论有无，总是创建新事务                         |
| SUPPORTS         | 支持事务，有则加入，无则在无事务状态中运行                   |
| NOT_SUPPORTED    | 不支持事务，在无事务状态下运行，如果当前存在已有事务，则挂起当前事务 |
| MANDATORY        | 必须有事务，否则抛异常                                       |
| NEVER            | 必须没事务，否则抛异常                                       |
| ...              |                                                              |

#### 2.3 事务的隔离级别

事务的隔离性是指在并发环境中，一个事务的执行不应该被其他事务干扰。 事务的隔离性主要是为了解决并发操作中的数据不一致问题。

在并发环境中，可能会出现：

- **脏读**（Dirty Read）：如果一个事务读到了另一个未提交事务修改过的数据。例如，事务1修改了一个数据，然后事务2在事务1提交前读取了这个数据，如果事务1最后失败并回滚，那么事务2读取到的数据就是“脏”数据。
- **不可重复读**（Nonrepeatable Read）：在一个事务内多次读取同一个数据，出现前后两次读到的数据不一样的情况，这种现象主要是由于并发事务的写入造成的。例如，事务1读取了一个数据，事务2随后修改了这个数据并提交，然后事务1再次读取同一数据，发现数据已经和前一次读取的不一样了。
- **幻读**（Phantom Read）：在一个事务内多次查询符合某个查询条件的记录数量，出现前后两次查询到的记录数量不一样的情况。也就是说，第二次查询能检索到第一次查询中未出现的记录，或者某些记录已经被删除。这种现象是由于并发事务的插入或删除造成的。

为了解决这些问题，SQL标准定义了四个**隔离级别**： 

- **读未提交**（Read Uncommitted）：最低的隔离级别，允许读取尚未提交的数据变更（一个事务还没提交时，它做的变更就能被其他事务看到），可能导致脏读、不可重复读、幻读。

- **读已提交**（Read Committed）：只能读取已经提交的数据变更（一个事务提交之后，它做的变更才能被其他事务看到），可以防止脏读，但是可能出现不可重复读、幻读。

- **可重复读**（Repeatable Read）：在同一事务中多次读取同一记录（行）会得到相同的结果，MySQL的默认隔离级别。可以防止脏读、不可重复读，但幻读仍可能出现，**MySQL InnoDB 引擎的默认隔离级别**。

- **串行化**（Serializable）：最高的隔离级别，完全避免了脏读、不可重复读、幻读。 会对记录加上读写锁，在多个事务对这条记录进行读写操作时，如果发生了读写冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行，这种隔离级别会导致性能下降。

@Transactional注解的**isolation**属性用于指定事务的传播行为（ISOLATION_DEFAULT，ISOLATION_READ_UNCOMMITTED，ISOLATION_READ_COMMITTED，ISOLATION_REPEATABLE_READ，ISOLATION_SERIALIZABLE）。



## Spring AOP

### 1. AOP概述

AOP（Aspect Oriented Programming），**面向切面/方面编程**，面向切面编程就是面向特定方法编程。AOP在程序运行期间在不修改源代码的基础上对已有方法进行增强（无侵入性，解耦）。

Spring AOP是通过**动态代理**实现的。如果我们为Spring的某个bean配置了切面，那么Spring在创建这个bean的时候，实际上创建的是这个bean的一个代理对象，我们后续对bean中方法的调用，实际上调用的是代理类重写的代理方法。Spring AOP使用了两种动态代理技术，分别是**JDK动态代理**，以及**CGLib动态代理**。

**Spring默认使用JDK动态代理实现AOP，类如果实现了接口，Spring就会使用这种方式实现动态代理**。JDK实现动态代理需要两个组件，首先第一个就是`InvocationHandler`接口。我们在使用JDK动态代理时，需要编写一个类，去实现这个接口，然后重写`invoke`方法，这个方法就是代理方法。然后JDK动态代理需要使用的第二个组件就是`Proxy`这个类，我们可以通过这个类的`newProxyInstance`方法，返回一个代理对象。生成的代理类实现了原来那个类的所有接口，并对接口的方法进行了代理，我们通过代理对象调用这些方法时，底层将通过反射，调用我们实现的`invoke`方法。

```Java
class XXXAspect implements InvocationHandler{
    private final Object target;
    
    public XXXAspect(Object target){
        this.target = target;
    }
    
    @Override
    public Object invoke(Object proxy, Method m, Object[] args) throws Throwable{
        return m.invoke(target, args);
    }
}

Map<Integer, Integer> proxyMap = (Map<Integer, Integer>) Proxy.newProxyInstance(ClassLoader.getSystemClassLoader(), new Class[]{Map.class}, new XXXAspect(new HashMap<Integer, Integer>()));
```

JDK动态代理存在限制，那就是被代理的类必须是一个实现了接口的类，代理类需要实现相同的接口，代理接口中声明的方法。若需要代理的类没有实现接口，此时JDK动态代理将没有办法使用，于是Spring会使用CGLib的动态代理来生成代理对象。CGLib直接操作字节码，生成类的子类，重写类的方法完成代理。

### 2. Spring AOP

#### 2.1 切面类

首先在`pom.xml`中导入AOP的依赖。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

针对特定方法根据业务需要编写切面类。

```Java
@Component
@Aspect
public class TimeAspect{
    @Around("execution(* com.example.service.*.*(..))")
    public Object recordTime(ProceedingJoinPoint joinPoint) throws Throwable{
        long begin = System.currentTimeMillis();
        
        Object result = joinPoint.proceed();
        
        long end = System.currentTimeMillis();
        
        return result;
    }
}
```

#### 2.2 AOP核心概念

- **连接点**（JoinPoint）：可以被AOP控制的方法。在Spring AOP提供的JoinPoint中封装了连接点方法在执行时的相关信息。

- **通知**（Advice）：在切面的某个特定的连接点上执行的动作。

- **切入点**（Pointcut）：匹配连接点的条件。通知和一个切入点表达式关联，并在满足这个切入点的连接点上运行。

- **切面**（Aspect）：描述通知与切入点的对应关系。当通知和切入点结合在一起，就形成了一个切面。通过切面就能够描述当前AOP程序需要针对于哪个原始方法，在什么时候执行什么样的操作。

- **目标对象**（Target）：通知所应用的对象。

#### 2.3 通知类型

Spring中AOP的通知类型：

- @Around：环绕通知，此注解标注的通知方法在目标方法前、后都被执行。
- @Before：前置通知，此注解标注的通知方法在目标方法前被执行。
- @After ：后置通知，此注解标注的通知方法在目标方法后被执行，无论是否有异常都会执行
- @AfterReturning ： 返回后通知，此注解标注的通知方法在目标方法后被执行，有异常不会执行
- @AfterThrowing ： 异常后通知，此注解标注的通知方法发生异常后执行。

@Around环绕通知需要自己调用 ProceedingJoinPoint.proceed() 来让原始方法执行，其他通知不需要考虑目标方法执行（JointPoint）。@Around环绕通知方法的返回值，必须指定为Object，来接收原始方法的返回值，否则原始方法执行完毕，是获取不到返回值的。

在Spring中用JoinPoint抽象了连接点，用它可以获得方法执行时的相关信息，如目标类名、方法名、方法参数等。

- 对于@Around通知，获取连接点信息只能使用**ProceedingJoinPoint**类型
- 对于其他四种通知，获取连接点信息只能使用**JoinPoint**，它是ProceedingJoinPoint的父类型。

  ```Java
  @Around("execution(* com.example.service.*.*(..))")
  public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
      String name = joinPoint.getTarget().getClass().getName();
      String methodName = joinPoint.getSignature().getName();
      Object[] args = joinPoint.getArgs();
      Object returnValue = joinPoint.proceed();
  	return returnValue;
  }
  ```


#### 2.4 @PointCut注解

Spring提供了**@PointCut**注解，该注解的作用是将公共的切入点表达式抽取出来，需要用到时引用该切入点表达式即可。

```Java
@Component
@Aspect
class XXXAspect{
    // 切入点方法（公共切入点表达式）
    @Pointcut("execution(* com.example.service.*.*(..))")
    private void pt(){}
    
    // 引用切入点
    @Around("pt()")
    public void around(ProceedingJoinPoint joinPoint) throws Throwable{
        ...
    }
}
```

当切入点方法使用private修饰时，仅能在当前切面类中引用该表达式。当外部其他切面类中也要引用当前类中的切入点表达式时，就需要把private改为public，而在引用的时候，具体的语法为全类名.方法名()。

```java
@Component
@Aspect
public class YYYAspect {
    //引用XXXAspect切面类中的切入点表达式
    @Before("com.example.aspect.XXXAspect.pt()")
    public void before(JoinPoint joinPoint){
        ...
    }
}
```

#### 2.5 通知顺序和@Order注释

当有多个切面的切入点都匹配到了目标方法，目标方法运行时，多个通知方法都会被执行。不同切面类中，默认按照切面类的类名字母排序：

- 目标方法前的通知方法：字母排名靠前的先执行。

- 目标方法后的通知方法：字母排名靠前的后执行。

使用Spring提供的**@Order**注解可以控制通知的执行顺序。前置通知，数字越小先执行；后置通知，数字越小越后执行。

```java
@Component
@Aspect
@Order(1)
public class XXXAspect {
    @Before("execution(* com.example.service.*.*(..))")
    public void before(){
        log.info("MyAspect2 -> before ...");
    }

    @After("execution(* com.example.service.*.*(..))")
    public void after(){
        log.info("MyAspect2 -> after ...");
    }
}
```

#### 2.6 切入点表达式

切入点表达式是描述切入点方法的一种表达式，用来决定项目中的哪些方法需要加入通知。

常见形式：

- **execution(...)：根据方法的签名来匹配。**

  ```java
  @Around("execution(public void com.example.service.*.*(..))")
  ```

  execution主要根据方法的返回值、包名、类名、方法名、方法参数等信息来匹配，语法为:

  ```Java
  execution(访问修饰符?  返回值  包名.类名.?方法名(方法参数) throws 异常?)
  ```

  其中带`?`的表示可以省略的部分：

  - 访问修饰符：可省略（比如: public，protected）
  - 包名.类名： 可省略
  - throws 异常：可省略（注意是方法上声明抛出的异常，不是实际抛出的异常）

  可以使用通配符描述切入点：

  - `*` ：单个独立的任意符号，可以通配任意返回值、包名、类名、方法名、任意类型的一个参数，也可以通配包、类、方法名的一部分。
  - `..` ：多个连续的任意符号，可以通配任意层级的包，或任意类型、任意个数的参数。

- **@annotation(...) ：根据注解匹配。**

  ```Java
  // 自定义注解
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface MyLog{}
  
  // 业务类
  @Service
  public class XXXServiceImpl implements XXXService{
      @MyLog
      public void insert(Employee employee){
          ...
      }
  }
  
  // 切面类
  public class XXXAspect{
      @Around("@annotation(com.example.anno.MyLog)")
      public void before(JoinPoint joinPoint){
          ...
      }
  }
  ```
  

根据业务需要，可以使用 且（`&&`）、或（`||`）、非（`!`） 来组合比较复杂的切入点表达式。
