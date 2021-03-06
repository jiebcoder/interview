## 简介

**优点**

- 低侵入式设计，代码的污染极低。
- DI 将对象之间的依赖关系交由框架处理，减低组件的耦合性。
- AOP 将一些通用任务进行集中式管理，从而提供更好的复用。
- 对于主流的应用框架提供了集成支持。

**Spring 框架特性** 

Core technologies：依赖注入，AOP，事件，资源，i18n，验证，数据绑定，类型转换，SpEL。

Testing：模拟对象，TestContext 框架，Spring MVC 测试，WebTestClient。

Data Access：事务，DAO 支持，JDBC，ORM，编组 XML。

Spring MVC 和 Spring WebFlux

Integration：远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存。

Languages：Kotlin，Groovy，动态语言。

其他模块：Aspects。



Spring5 基于 JDK8，支持候选组件索引代替类路径扫描（大型项目启动时间降低），支持 Kotlin，响应式编程，Junit5，更新依赖类库。



## 1.AOP

面向切面编程。

代理模式。

**面向对象 OOP**

允许开发者定义纵向的关系，但并适用于定义横向的关系，导致了大量代码的重复，而不利于各个模块的重用。AOP 作为 OOP 的一种补充。

#### **切面**

横切关注点可以被描述为那些与业务无关，但却对多个对象产生影响的公共行为和逻辑。

切面能将横切关注点抽取并封装为一个可重用的模块，在系统运行时需要的地方进行动态插入运行。

分离业务逻辑与系统服务，应用对象只完成业务逻辑。可用于事务处理、日志管理、权限控制、缓存控制、性能监控、异常处理等。 

**优点**

扩展功能不修改源代码，减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

**AOP 和 AspectJ**

Spring AOP 已经集成了 AspectJ。Spring AOP 属于运行时增强，而 AspectJ 是编译时增强（静态代理）。Spring AOP 基于代理，而 AspectJ 基于字节码操作。AspectJ 功能更加强大，需要特定的编译器进行处理，当切面太多的话，最好选择 AspectJ，它比 Spring AOP 快很多。

**动态代理**

Spring AOP 默认使用 JDK 动态代理。目标对象不存在接口时，使用 CGLIB。

无法拦截 static、final、private 方法，无法拦截内部方法调用。

#### **@Aspect**

AspectJ 对类的注解。

连接点 joinpoint：准备增强的方法。

切入点 pointcut：实际增强的方法。切入点表达式包含指示器 designators、通配符 wildcards 和运算符 operators。

通知 advice：实现的功能。

切面 aspect：被抽取的公共模块。

引入 introduction：向现有的类添加新的方法或者属性。

织入 weaving：把增强应用到目标对象来创建新的代理对象的过程。

连接点和切入点匹配的概念是 AOP 的关键，这使得 AOP 不同于其它仅仅提供拦截功能的旧技术。切入点使得定位通知可独立于 OO 层次。

**通知 Advice**

前置 @Before

后置 @AfterReturning：切入点方法正常执行后，和异常通知两者只能执行一个（对应 commit 和 rollback）。

最终 @After：最后执行。XML 和注解的 @After 执行顺序不一样。

异常 @AfterThrowing：切入点方法产生异常后执行后。

环绕 @Around：前面功能全有，在 @Before 或 @After 之前。环绕通知是最常用的一种通知类型，大部分基于拦截的 AOP 框架都只提供环绕通知。 

**XML 实现 Spring AOP**

xmlns:aop 和 aop:config 标签。

前面四个通知采用 XML 实现，环绕通知需要自己编码实现 ProceedingJoinPoint 接口，并 proceed() 调用切入点方法。

注解在 XML 中开启 AOP：aop:aspect-autoproxy。XML 可以被注解完全代替。

用责任链模式实现多个 AOP 叠加。

**AOP 实现 transaction、security、cache**

security：MethodSecurityInterceptor、PreInvocationAuthrizationAdviceVote、ExpressionBasedPreInvocationAdvice。

@PreAuthorize 验证机制：decide -> vote -> before 只需要一个注解就可以实现，debug 进源码看实现流程。

cache：AnnotationCacheSupport、CacheInterceptor、CacheAspectSupport，时序图 invoke -> execute -> doGet -> invokeJoinpoint -> cachePutRequest.apply。



## 2.IOC

控制反转。

工厂模式。

与依赖注入 DI 是同一个概念不同角度的描述。

#### IOC 容器

将原本在程序中手动创建对象的控制权，交由 Spring 框架来管理。

IOC 容器负责管理对象之间的依赖关系，实际上就是个 map，存放的是各种对象。

IOC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件或注解即可，完全不用考虑对象是如何被创建出来的。然后在需要的地方通过反射机制，在运行时动态创建对象。

Spring 时代我们一般通过 XML 文件来配置 bean，后来开发人员觉得 XML 文件来配置不太好，于是 Spring Boot 注解配置就慢慢开始流行起来。

**优点**

- 把应用从复杂的依赖关系中解放出来，在创建实例的时候不需要了解其中的细节，简化应用的开发，增加了项目的可维护性。

- 降低耦合，把对象生成放在配置文件里进行定义，更换一个实现子类将会变得很简单，只要修改配置文件就可以，具有热插拨特性。

- 利于功能的复用。

缺点

- 引入了第三方 IOC 容器，生成对象的步骤变得复杂。

- IOC 容器通过反射方式生成对象，在运行效率上有一定的损耗。

- 需要进行大量的配制工作，比较繁琐。

**Spring IOC 的初始化过程**

XML 读取为 Resource，解析为 BeanDefinition（用来管理 bean 的属性和对象间的相互依赖关系），注册为 BeanFactory。

#### **注入方式**

- 构造函数：配置文件的 constructor-arg 标签。

- setter：配置文件的 property 标签，更常用，类中加 set 方法。

- 注解：XML 中加 xmlns:context 和 context:component-scan 标签。



Spring 基于 XML 注入 bean 还可以使用静态工厂和实例工厂注入。

构造函数方式注入必须要注入数据，不然创建对象不成功，如果用不到这些数据也必须提供。

选择 XML 或者注解需要看是用别人的 JAR 包还是自己开发。

**能注入的类型**

- 基本类型和 String：配置文件的 value 属性，其他类型用 ref 且需要配置过。

- XML 或注解配置的 bean。

- 复杂类型：构造函数和 setter 来注入。

- 集合类型：结构相同的标签可以互换，map 对应 property，只能用 XML 实现。



## 3.MVC

SpringMVC 框架是以请求为驱动，围绕 Servlet 设计，将请求发给控制器，然后通过模型对象，分派器来展示请求结果视图。

其中核心类是 DispatcherServlet，它是一个 Servlet，顶层是实现的 Servlet 接口。作为 Spring MVC 的入口函数，作用是接收请求，响应结果。
耦合低，重用性高。但使项目架构变得复杂。

SpringMVC 使用：需要在 web.xml 中配置 DispatcherServlet。并且需要配置 Spring 监听器 ContextLoaderListener。

模型 Model 是应用程序中用于处理应用程序数据逻辑的部分。JavaBean。

通常模型对象负责在数据库中存取数据。

视图 View 是应用程序中处理数据显示的部分。JSP。

通常视图是依据模型数据创建的。

控制器 Controller 是应用程序中处理用户交互的部分。Servlet，web.xml 和 springmvc.xml。

通常控制器负责读取用户输入，向模型发送数据，把数据交给视图展示。

**web.xml 配置**

DispatcherServlet

加载 springmvc.xml：

```xml
<init-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>classpath:springmvc.xml</param-value>
</init-param>
```

过滤器，解决中文乱码问题：

```xml
<filter></filter>
<filter-mapping></filter-mapping>
```

过滤器 HiddentHttpMethodFilter：改变 HTTP 请求方式，使得支持 GET、POST、PUT 和 DELETE 请求。

监听器 < listener>< /listener> 加设置配置文件 applicationContext.xml 的位置 < context-param>< /context-param>。

**springmvc.xml 配置**

视图解析器对象 < mvc:annotation-driven/> 等于配置了 HandlerMapping 和 HandlerAdapter。

自定义类型转换器 conversionService

不拦截静态资源 <mvc:resources mapping= location=/> 

文件解析器 multipartResolver

异常处理器

拦截器 < mvc:interceptors>< /mvc:interceptors> 

#### 流程

1. 客户端或浏览器发送请求，直接请求到前端控制器 DispatcherServlet。

2. DispatcherServlet 根据请求信息调用处理器映射器 HandlerMapping，解析请求对应的处理器 Handler。解析出具体的类.方法。

3. 解析到对应的 Handler（也就是我们平常说的控制器 Controller）后，开始由处理器适配器 HandlerAdapter 处理。任何类.方法都会被适配为可以被适配器执行的样子。

4. HandlerAdapter 会根据 Handler 来调用真正的处理器开处理请求，并处理相应的业务逻辑。

5. 处理器处理完业务后，会返回一个 ModelAndView 对象，Model 是返回的数据对象，View 是个逻辑上的 View。

6. 视图解析器 ViewResolver 会根据逻辑 View 查找实际的 View。

7. DispatcherServlet 把返回的 Model 传给 View（视图渲染）。

8. DispatcherServlet 把 View 返回给请求者。

其中 Handler 和 View 需要自己开发，其他由框架提供。基于组件的方式。

**开发框架变迁**

Model1 时代：整个 Web 应用几乎全部用 JSP 页面组成，只用少量的 JavaBean 来处理数据库连接、访问等操作，JSP 即是控制层又是表现层。

Model2 时代：Java Bean（Model）+ JSP（View）+ Servlet（Controller），早期的 JavaWeb MVC 开发模式，抽象和封装程度还远远不够，重复造轮子，程序的可维护性和复用性低。

Struts2 比较笨重，而 Spring MVC 使用更加简单和方便，开发效率更高，并且 Spring MVC 运行速度更快。

Spring MVC 下我们一般把后端项目分为 Controller 层（控制层，返回数据给前台页面）、Service 层（处理业务）、Dao 层（数据库操作）、Entity 层（实体类）。

Spring Boot

#### SSM 三层架构

表现层 SpringMVC：接受用户参数，显示页面。

业务层 Spring：处理业务逻辑。

持久层 MyBatis：操作数据库。

**对象**

POJO：简单 Java 对象，即 JavaBean。

VO -> DTO -> DO -> PO，解耦。

VO：展示层需要显示的数据。

DTO：表现层和业务层之间的数据传输对象。

DO：业务角色的抽象。	

PO：持久化对象。

#### @Controller vs @RestController

**@Controller**

返回一个页面，单独使用 @Controller 不加 @ResponseBody 的话一般使用在要返回一个视图的情况，这种情况属于比较传统的 SpringMVC 的应用，对应于前后端不分离的情况。

@ResponseBody 注解的作用是将 Controller 的方法返回的 JavaBean 对象转换为指定的格式之后，写入到 HTTP 响应对象的 body 中。返回 JSON 格式的情况比较多。  

**@RestController**

@RestController = @Controller + @ResponseBody。

返回 JSON 或 XML 形式数据，但只返回对象，对象数据直接以 JSON 或 XML 形式写入 HTTP 响应 Response 中，这种情况属于 RESTful web 服务，这也是目前日常开发所接触最常用的前后端分离。

Spring 4 之后新加的注解。

**@RequestMapping**

将对页面的 HTTP 请求映射到方法或者类，并且默认映射所有 HTTP 方法。

映射路径 path/value，HTTP 方法 method，URL 参数限制 params，请求头限制 headers。

在方法中传入 HttpServletRequest 和 HttpServletResponse 就可以获取这两个对象以及原生 API。

**@PathVariable**

获取路径参数。

**@RequestParam**

获取查询参数。

**@RequestBody**

用于读取 Request 的 body 部分并且 Content-Type 为 application/json 格式的数据，接收到数据之后会自动将数据绑定到 Java 对象上去。系统会使用 HttpMessageConverter 或者自定义的 HttpMessageConverter 将请求的 body 中的 JSON 字符串转换为 Java 对象。

一个请求方法只可以有一个 @RequestBody，但是可以有多个 @RequestParam 和 @PathVariable。

**请求参数的绑定**

URL 的参数可以传到 Controller 中。参数过多可以封装到 JavaBean 中，方法参数传这个 JavaBean 类型，可以是表单的请求参数。还可以封装到容器中。

**参数校验**

参数 @Valid 和类 @Validated。JSR 和 Hibernate Validator 提供校验注解。或者自定义参数校验注解。



@RequestHeader

@CookieValue：获取特定的 Cookie 的值。

@SessionAttribute：model.addAttribute() 会把值存到 request 对象中，在多个控制器方法共享参数。

响应返回值是 void 需要 forward 或者 redirect，分别为 request.getRequestDispatcher().forward(request,response) 和 response.sendRedirect(request.getContextPath()+)。

或者直接进行响应：response.getWriter().print()。

设置中文乱码问题：response.setCharacterEncoding("UTF-8");response.setContentType("text/html;charset=UTF-8");

返回 ModelAndView：底层是 map。是传入 model 返回 String 的底层。

返回 String 中使用 forward 和 redirect。



文件上传：传统方法 new File(path)，解析 upload.parseRequest(request)，上传。上传到服务端文件夹下。SpringMVC 方法配置文件解析器，并在 Controller 中传参 MultipartFile。跨服务器上传。

异常处理：目标是浏览器显示友好的提示而不是出错信息。需要编写自定义异常类，Controller 抛出这个异常，还要编写异常处理类。

拦截器 interceptor：预处理和后处理，只会拦截 Controller 的方法。编写拦截器类，配置。还有页面完成后处理。多个拦截器按先进后出的顺序。

Spring 整合 SpringMVC：启动 Tomcat 服务器的时候加载 Spring 的配置文件 applicationContext.xml，用到监听器 ContextLoaderListener 监听 ServletContext 对象，让监听器取加载 Spring 的配置文件。

Spring 整合 MyBatis：配置连接池，配置 SqlSessionFactory，配置 DAO 所在包。有了 applicationContext.xml 可以替代 MyBatis 的主配置文件 configuration.xml。只需要编写 Mapper 和 SQL。 

Spring 整合声明式事务：配置事务管理器，事务通知，AOP。



## 4.bean

可重用组件。

bean 的定义以及 bean 相互间的依赖关系将通过配置元数据来描述。

#### bean 的生命周期

bean 实例化过程，不同作用域 bean 的生命周期不同，@PostConstruct 和 @PreDestory 注解生命周期。

1. 实例化 bean，BeanFactory 容器调用 createBean 进行实例化，ApplicationContext 容器启动结束后，通过获取 BeanDefinition 对象中的信息，实例化所有的 bean。
2. 设置对象属性，实例化后的对象被封装在 BeanWrapper 对象中，然后 Spring 根据 BeanDefinition 中的信息以及通过 BeanWrapper 提供的设置属性的接口完成依赖注入。
3. 如果 bean 实现了 BeanNameAware 接口，调用 setBeanName() 传入 Spring 配置文件中 bean 的 id 值。
4. 同理，其他 Aware 接口。
5. 对 bean 进行一些自定义的处理，实现 BeanPostProcessor 接口，调用 postProcessBeforeInitialization()。
6. 调用 InitializingBean 的 afterPropertiesSet()。
7. 调用自定义的初始化方法：如果 bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
8. BeanPostProcessor 调用 postProcessAfterInitialization()。
9. 使用 bean，将一直存在于应用上下文中，直到该应用上下文被销毁。
10. 调用 DisposableBean 的 destroy()。
11. 调用自定义的销毁方法：如果 bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

#### bean 的作用域

@scope 声明 bean 的作用域。

**singleton**

唯一 bean 实例。

Spring 中的 bean 默认都是单例的。

创建容器时就同时自动创建了一个 bean 对象，或者指定延迟初始化。容器销毁 bean 死亡。

Spring 的单例是基于 Spring 容器的，单例 Bean 在此容器内只有一个，Java 的单例是基于 JVM，每个 JVM 内只有一个实例。

**单例 bean 的线程安全问题**

当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

解决方法：在类中定义一个 ThreadLocal 成员变量，将需要的可变成员变量保存在 ThreadLocal 中。

**prototype**

每次请求都会创建一个新的 bean 实例。创建容器的时候并没有实例化，而是延迟初始化。JVM 垃圾回收。

有状态的 bean 应该使用 prototype 作用域，而对无状态的 bean 则应该使用 singleton 作用域。

**request**

每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效。

**session**

每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效。

**global-session**

全局 session 作用域，仅仅在基于 portlet 的 web 应用中才有意义，Spring 5 已经没有了。

集群环境，负载均衡。

#### 将一个类声明为 bean 的注解

@Component：通用的注解，可标注任意类为 Spring 组件。如果一个 bean 不知道属于哪个层，可以使用 @Component 注解标注。创建对象，把当前对象存入 Spring 容器中，相当于 XML 的 < bean>。

@Repository：对应持久层，主要用于数据库相关操作。

@Service：对应服务层，主要涉及一些复杂的逻辑。

@Controller：对应控制层，主要用户接受用户请求并返回数据给前端页面。

**@Component 和 @Bean 的区别**

- @Component 注解作用于类，而 @Bean 注解作用于方法。

- @Component 通常是通过类路径扫描来自动侦测以及自动装配到 Spring 容器中，@Bean 注解通常是在标有该注解的方法中定义产生这个 bean。

- @Bean 注解比 Component 注解的自定义性更强，而且很多地方我们只能通过 @Bean 注解来注册 bean。比如当我们引用第三方库中的类需要装配到 Spring 容器时，则只能通过 @Bean 来实现。

#### 自动装配 @Autowired 

自动导入对象到类中。

被注入进的类同样要被 Spring 容器管理。

@Autowired 可用于构造函数、Setter 方法和成员变量。

在使用 @Autowired 注解之前需要在 Spring 配置文件进行配置，< context:annotation-config/>。

在启动 Spring IOC 时，容器自动装载了一个 AutowiredAnnotationBeanPostProcessor 后置处理器，当容器扫描到 @Autowied、@Resource 或 @Inject 时，就会在 IOC 容器自动查找需要的 bean，并装配给该对象的属性。

在 Spring 框架 XML 配置中共有 5 种自动装配：no、byName、byType、constructor 和 autodetect。

**容器中查询对应类型的 bean**

1. 如果查询结果刚好为一个，就将该 bean 装配给 @Autowired 指定的数据。

2. 如果查询的结果不止一个，那么 @Autowired 会根据名称来查找。

3. 如果上述查找的结果为空，那么会抛出异常。解决方法：使用 required=false。

**@Autowired 和 @Resource 之间的区别**

@Autowired 默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在，也可以设置它 required 属性为 false。

@Resource 默认是按照名称来装配注入的，只有当找不到与名称匹配的 bean 才会按照类型来装配注入。



@Qualifier：在按照类注入的基础上再根据名称注入。给类注入时不能单独使用，需要配合 @Autowired。给方法和变量注入时可以单独使用。

@Value：基本类型和 String 无法用 @Autowired、@Resource 和 @Qualifier 实现，需要使用 @Value。可以配合 Spring 的 EL 表达式 SpEL。

#### Spring 中 bean 生存的容器

BeanFactory、ApplicationContext、webApplicationContext。

BeanFactory 是 Spring 里面最底层的接口，包含了各种 bean 的定义，读取 bean 配置文档，管理 bean 的加载、实例化，控制 bean 的生命周期，维护 bean 之间的依赖关系。

ApplicationContext 是 BeanFactory 的子接口。ApplicationContext 除了提供 BeanFactory 所具有的功能外，还提供了更完整的框架功能。

先获取 ApplicationContext，再 getBean()。

ApplicationContext 三个实现类 ClassPathXmlApplicationContext（更常用）、FileSystemXmlApplicationContext、AnnotationConfigApplicationContext。

**BeanFactory 和 ApplicationContext 的区别**

- BeanFactroy 采用的是延迟加载形式来注入 bean 的，即只有在使用到某个 bean 时（即调用 getBean()），才对该 bean 进行加载实例化。这样，我们就不能发现一些存在的 Spring 的配置问题。如果 bean 的某一个属性没有注入，BeanFacotry 加载后，直至第一次使用调用 getBean 方法才会抛出异常。

- ApplicationContext 是在容器启动时，一次性创建了所有的 bean。这样，在容器启动时，我们就可以发现 Spring 中存在的配置错误，这样有利于检查所依赖属性是否注入。ApplicationContext 启动后预载入所有的单实例 bean，通过预载入单实例 bean，确保当你需要的时候，你就不用等待，因为它们已经创建好了。

- 相对于基本的 BeanFactory，ApplicationContext 唯一的不足是占用内存空间。当应用程序配置 Bean 较多时，程序启动较慢。

- BeanFactory 通常以编程的方式被创建，ApplicationContext 还能以声明的方式创建，如使用 ContextLoader。

- BeanFactory 和 ApplicationContext 都支持 BeanPostProcessor、BeanFactoryPostProcessor 的使用，但 BeanFactory 需要手动注册，而 ApplicationContext 则是自动注册。

ApplicationContext 更好，配合单例，也可以指定是否延迟初始化。



## 5.注解

#### 配置

**@Configuration**

声明配置类，可以使用 @Component 注解替代。

**@SpringBootApplication**

由 @SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan 组成。@AliasFor 桥接到其他注解。

**@EnableAutoConfiguration**

启动 Spring 应用程序上下文时进行自动配置。

通常是基于项目 classpath 中引入的类和已定义的 Bean 来实现的。在此过程中，被自动配置的组件来自项目自身和项目依赖的 JAR 包中。

被 @EnableAutoConfiguration 注解的类所在 package 还具有特定的意义，通常会被作为扫描注解@Entity的根路径。

**@ComponentScan**

从扫描的路径中找出标识了需要装配的类，自动装配到 Spring 的 bean 容器中。相当于在 XML 中加了 < context:component-scan> 标签。默认会扫描该类所在的包下所有的类。@Filter。

@ConfigurationProperties：读取配置信息并与 bean 绑定。

@PropertySource：读取指定 properties 文件。

@Import：导入子配置类。

#### 统一异常处理

减少 try...catch...。

**@ControllerAdvice**

**@ExceptionHandler**

注解声明异常处理方法。修饰方法会在 Controller 出现异常后被调用，用于处理捕获到的异常。

会优先找到最匹配的，源码中 getMappedMethod() 会首先找到可以匹配处理异常的所有方法信息，然后对其进行从小到大的排序，最后取最小（即匹配度最高的）那一个匹配的方法。

**@ModelAttribute**

放在方法上会在控制器方法执行前执行。有有返回值和无返回值（需要 Map）两种情况，用来给在表单提交数据之外的参数赋值。加在参数前则可以从 Map 中取值。用于为 Model 对象绑定参数。

**@DataBinder**

修饰的方法会在 Controller 方法执行前被调用，用于绑定参数的转换器。

**步骤**

1. 或者写一个枚举封装全局异常：枚举类包含了异常的唯一标识、HTTP 状态码以及错误信息。而不用定义许多异常类。

2. 定义异常处理器。

3. 封装返回结果。

#### JPA

@Entity：声明一个类对应一个数据库实体。

@Table：设置表名。

@Id：声明一个字段为主键。@GeneratedValue 指定主键生成策略，默认 GenerationType.AUTO。一般使用 MySQL 数据库的话，使用GenerationType.IDENTITY 策略比较普遍一点（分布式系统的话需要另外考虑使用分布式 ID）。

@GenericGenerator：声明一个主键策略，然后 @GeneratedValue 使用这个策略。

@Column：声明字段。

@Transient：指定不持久化特定字段。或者在变量前加 static、final 或 transient。

@Lob：声明某个字段为大字段。

@Enumerated：创建枚举类型的字段。

@EnableJpaAuditing：开启 JPA 审计功能。用于 Spring Security。

@Modifying：修改操作，配合 @Transactional 使用。

关联关系：@OneToOne 等。

#### JSON

@JsonIgnoreProperties：作用在类上用于过滤掉特定字段不返回或者不解析。

@JsonIgnore：作用在成员变量。

@JsonFormat：格式化 JSON 数据。

@JsonUnwrapped：扁平对象。

#### 测试

@ActiveProfiles：一般作用于测试类上， 用于声明生效的 Spring 配置文件。

@Test：声明一个方法为测试方法。

@WithMockUser：Spring Security 提供的，用来模拟一个真实用户，并且可以赋予权限。

@RunWith 和 @ContextConfiguration：Spring 整合 JUnit。版本要对应。



@GetMapping 等：HTTP 请求。

@AnnotationConfigApplicationContext：代替 XML，当传入了所有需要的 .class 时 @Configuration 可以不写。

@Param：给参数取别名，如果只有一个参数且在 < if> 里使用，必须加 @Param。

@Deprecated：不推荐使用。

**字段验证的注解**

@NotEmpty 等。



## 6.Spring 用到的设计模式

代理模式&工厂模式

责任链模式：多个 AOP 叠加。

单例模式：Spring 中的 bean 默认作用域是单例的。

模板方法：Spring 中以 Template 结尾的对数据库操作的类用到了模板模式。

观察者模式：Spring 事件驱动模型。如果一个 bean 实现了 ApplicationListener 接口，当一个 ApplicationEvent 被发布以后，bean 会自动被通知。

适配器模式：Spring AOP 的 advice 和 Spring MVC 的 controller 使用到了适配器模式。

装饰器模式：连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。



## 7.事务

Spring 事务的本质其实就是数据库对事务的支持。

#### Spring 事务管理接口

事务管理：按照给定的事务规则来执行提交或者回滚操作。

Spring 并不直接管理事务，而是提供了多种事务管理器，将事务管理的职责委托给 Hibernate 或者 JTA 等持久化机制所提供的相关平台框架的事务来实现。

**PlatformTransactionManager**

（平台）事务管理器，根据不同持久层框架提供对应的接口实现类。具体的实现就是各个平台自己的事情了。

**TransactionDefinition**

事务定义信息（事务隔离级别、传播行为、超时、只读、回滚规则）。事务属性可以理解成事务的一些基本配置，描述了事务策略如何应用到方法上。

**隔离级别**

定义了一个事务可能受其他并发事务影响的程度。Spring 事务中的隔离级别多了 DEFAULT，使用后端数据库默认的隔离级别，MySQL 默认采用的 REPEATABLE_READ 隔离级别，Oracle 默认采用的 READ_COMMITTED 隔离级别。

**传播行为**

处理多个事务同时存在时的行为。当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。需要正确设置传播行为，否则事务可能会回滚失败。可以对两个方法进行双向的回滚控制，即有两种回滚影响方式。

**默认 Propagation.REQUIRED**

- 在外围方法未开启事务的情况下 Propagation.REQUIRED 修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。

- 在外围方法开启事务的情况下 Propagation.REQUIRED 修饰的内部方法会加入到外围方法的事务中，所有 Propagation.REQUIRED 修饰的内部方法和外围方法均属于同一事务，只要一个方法回滚，整个事务均回滚。

Propagation.NOT_SUPPORTED：不需要事务管理的（只查询）的方法。

**超时属性**

一个事务允许执行的最长时间。如果超过该时间限制但事务还没有完成，则自动回滚事务。

**只读属性**

对事物资源是否执行只读操作。将事务标志为只读以提高事务处理的性能。

**回滚规则**

定义事务回滚规则。

**TransactionStatus**

事务运行状态。该接口定义了一组方法，用来获取或判断事务的相应状态信息。

PlatformTransactionManager.getTransaction(…) 方法返回一个 TransactionStatus 对象。返回的 TransactionStatus 对象可能代表一个新的或已经存在的事务（如果在当前调用堆栈有一个符合条件的事务）。           

#### @Transactional

**作用于类**

当把 @Transactional 注解放在类上时，表示所有该类的 public 方法都配置相同的事务属性信息。建议在具体的类上使用 @Transactional 注解，而不是接口或父类。

**作用于方法**

当类配置了@Transactional，方法也配置了 @Transactional，方法的事务会覆盖类的事务配置信息。方法处理过程尽量简单，可以将常规的数据库查询操作放在事务前面进行，而事务内进行增、删、改、加锁查询等操作。只有作用到 public 方法上事务才生效。

**rollbackFor**

在 @Transactional 注解中如果不配置 rollbackFor 属性，那么事务只会在遇到 RuntimeException 和 Error 的时候才会回滚。配置 rollbackFor 属性，遇到非运行时异常时也回滚。

如果异常被 try{}...catch{} 了，事务就不回滚了，如果想让事务回滚必须再往外抛 try{}...catch{ throw Exception}。

**原理**

@Transactional 的工作机制是基于 AOP 实现的，如果一个类或者一个类中的 public 方法上被标注@Transactional 注解的话，Spring 容器就会在启动的时候为其创建一个代理类，在调用被 @Transactional 注解的 public 方法的时候，实际调用的是，TransactionInterceptor 类中的 invoke() 方法。这个方法的作用就是在目标方法之前开启事务，方法执行过程中如果遇到异常的时候回滚事务，方法调用完成之后提交事务。

**Spring AOP 自调用问题**

若同一类中的其他没有 @Transactional 注解的方法内部调用有 @Transactional 注解的方法，有@Transactional 注解的方法的事务会失效。

原因：Spring AOP 代理造成，因为只有当 @Transactional 注解的方法在类以外被调用的时候，Spring 事务管理才生效。

解决办法：避免同一类中自调用或者使用 AspectJ 取代 Spring AOP 代理。

#### Spring 管理事务的方式

**编程式事务**

硬编码，不推荐使用。通过 TransactionTemplate 手动管理事务。但是管理事务的范围更小（到方法内），而声明式事务对于整个方法而言。

**声明式事务**

基于 XML 和基于注解，在配置文件中配置，推荐使用。实际是通过 AOP 实现，代码侵入性小。对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

实现声明式事务的四种方式：基于 TransactionInterceptor、基于 TransactionProxyFactoryBean、基于 < tx> 和 < aop> 命名空间、基于 @Transactional。

XML：在 XML 中配置事务管理器和事务通知（配置事务的属性），还要在 aop:config 中建立切入点表达式和事务通知之间的对应关系。使用 XML 方式可以避免多次 @Transactional。

注解：在 XML 中配置事务管理器和开启 Spring 对注解事务的支持，使用 @Transactional。

纯注解需要写 Configuration 类，连接数据库的配置类，事务管理器类。加上 @Configuration、@ComponentScan、@PropertySource、@EnableTransactionManagment、@Transactional、@Value、@Bean 等。



## 8.Spring Boot

SpringBoot 简化了 Spring 开发的配置。

约定优于配置。

#### spring-boot-starter

起步依赖，自动加载了项目起步所需要的的相关 JAR 包，配置相应的初始化参数。

自动配置

spring-boot-starter-parent 依赖于 spring-boot-dependencies，包含了依赖的软件的版本。

spring-boot-starter-web

**过程**

1. Spring Boot 通过 @EnableAutoConfiguration 注解开启自动配置。
2. 加载各个 JAR 包 META-INF 目录下 spring.factories 配置文件中注册的各种 AutoConfiguration 类。
3. 当某个 AutoConfiguration 类满足其注解 @Conditional 指定的生效条件，三方组件的依赖及配置 Starters（Spring Boot 已经预置的组件），那么就会实例化该 AutoConfiguration 类中定义的 Bean（组件等），并注入 Spring 容器，至此就完成了依赖框架的自动配置。

**@SpringBootApplication**

启动类，是 @SpringbootConfiguration、@EnableAutoConfiguration 和 @ComponentScan 注解的集合。

和启动类同级和其子包下会自动扫描。



写一个 application.yml/.properties 文件来自定义配置。@Value 获取这个配置文件的属性，@ConfigurationProperties 获取对象并填充到对象对应的各条属性中。

spring-boot-configuration-processor 在写 .yml 文件时会根据类的属性提示。

Spring Boot 整合 MyBatis：application.yml 中写数据库连接信息、MyBatis 包扫描和加载 MyBatis 映射文件。JPA 类似。

@SpringBootTest

Spring Boot 整合 Redis：RedisTemplate。

spring-boot-devtools：热部署。

#### 线程池

Spring Boot 启动类上加 @EnableAsync。

我们可以使用 Spring Boot 默认的线程池，不过一般我们会自定义线程池，因为比较灵活，配置方式有：XML 文件和 @Configuration。

applicationContext.xml 中引入：开启异步并引入线程池和定义线程池。

或者引入线程池的配置 < import resource="threadPool.xml" /> <task:annotation-driven executor="WhifExecutor" />

@Async 创建异步方法。

注解方式：@Configuration 和 @EnableAsync 写一个配置类，创建异步方法。

@PostConstruct：项目启动时就执行一次该方法。

**如下方式会使 @Async 失效**

- 异步方法使用 static 修饰。
- 异步类没有使用 @Component 注解（或其他注解）导致 Spring 无法扫描到异步类。
- 异步方法不能与被调用的异步方法在同一个类中。
- 类中需要使用 @Autowired 或 @Resource 等注解自动注入，不能自己手动 new 对象。
- 没加 @EnableAsync。

#### 项目监控 Spring Boot Actuator

端点 Endpoints：监控程序的入口，分为 Spring Boot 内置的端点和自定义端点。内置唯一关闭的端点，起关闭服务器的作用。

监控方式：HTTP 或者 JMX。

访问路径：如 /actuator/health。返回 JSON 格式。

注意要对端点进行权限控制。