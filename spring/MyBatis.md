## 1.简介

MyBatis 是一个 Java 持久化框架，它通过 XML 描述符或注解把对象与存储过程或 SQL 语句关联起来。

与其他的对象关系映射框架不同，MyBatis 并没有将 Java 对象与数据库表关联起来，而是将 Java 方法与 SQL 语句关联。

对象关系映射 ORM：用于实现面向对象编程语言里不同类型系统的数据之间的转换。数据库表与实体类相对应，数据库表的字段与实体类的属性相对应。减少对 SQL 语句的关注。

实现 ORM 思想的框架：MyBatis 和 Hibernate。

MyBatis 中的 plugin 实现责任链模式。



## 2.特点

MyBatis 中的 SQL 语句和主要业务代码分离，我们一般会把 MyBatis 中的 SQL 语句统一放在 XML 配置文件中，便于统一维护。解偶，提供 DAO 层，系统的设计更清晰，更容易单元测试。

本身就很小且简单，没有任何第三方依赖，易于使用。

MyBatis 屏蔽了原始的 JDBC 样板代码，专注于 SQL 的书写和属性与字段映射上。

MyBatis 最主要的特点就是可以手动编写 SQL 语句，能够支持多表关联查询。



## 3.整体架构

接口层：开发人员在 Mapper 或者是 Dao 接口中的接口定义，是查询、新增、更新还是删除操作。

数据处理层：配置 Mapper -> XML 层级之间的参数映射，SQL 解析，SQL 执行，结果映射的过程。
在 Mybatis 初始化过程中，会加载 mybatis-config.xml 配置文件、映射配置文件以及 Mapper 接口中的注解信息，解析后的配置信息会形成相应的对象并保存到 Configration 对象中。
之后，根据该对象创建 SqlSessionFactory 对象。待 Mybatis 初始化完成后，可以通过 SqlSessionFactory 创建 SqlSession 对象并开始数据库操作。

动态 SQL 语句：几乎可以编写出所有满足需要的 SQL。Mybatis 中 scripting 模块会根据用户传入的参数，解析映射文件中定义的动态 SQL 节点，形成数据库能执行的 SQL 语句。
SQL 语句的执行：涉及多个组件，包括 MyBatis 的四大核心，它们是：Executor、StatementHandler、ParameterHandler、ResultSetHandler。

基础支持层：给上两层提供功能支撑，包括连接管理，事务管理，配置加载，缓存处理等。
反射模块：对 Java 反射进行了很好的封装，提供了简易的 API，方便上层调用，并且对反射操作进行了一系列的优化，比如，缓存了类的元数据 MetaClass 和对象的元数据 MetaObject，提高了反射操作的性能。
类型转换模块：Mybatis 的别名机制，能够简化配置文件，该机制是类型转换模块的主要功能之一。类型转换模块的另一个功能是实现 JDBC 类型与 Java 类型的转换。在 SQL 语句绑定参数时，会将数据由 Java 类型转换成 JDBC 类型；在映射结果集时，会将数据由 JDBC 类型转换成 Java 类型。
日志模块：在 Java 中，有很多优秀的日志框架，如 Log4j、Log4j2、slf4j 等。Mybatis 除了提供了详细的日志输出信息，还能够集成多种日志框架，其日志模块的主要功能就是集成第三方日志框架。
资源加载模块：该模块主要封装了类加载器，确定了类加载器的使用顺序，并提供了加载类文件和其它资源文件的功能。
解析器模块的主要功能：一个是封装了 XPath，为 Mybatis 初始化时解析 mybatis-config.xml 配置文件以及映射配置文件提供支持（MyBatis 默认使用 XPath 来解析标签）；另一个为处理动态 SQL 语句中的占位符提供支持。
数据源模块：Mybatis 自身提供了相应的数据源实现，也提供了与第三方数据源集成的接口。数据源是开发中的常用组件之一，很多开源的数据源都提供了丰富的功能，如连接池、检测连接状态等，选择性能优秀的数据源组件，对于提供ORM 框架以及整个应用的性能都是非常重要的。
事务管理模块：一般地，Mybatis 与 Spring 框架集成，由 Spring 框架管理事务。但 Mybatis 自身对数据库事务进行了抽象，提供了相应的事务接口和简单实现。
缓存模块：Mybatis 中有一级缓存和二级缓存，这两级缓存都依赖于缓存模块中的实现。但是需要注意，这两级缓存与 Mybatis 以及整个应用是运行在同一个 JVM 中的，共享同一块内存，如果这两级缓存中的数据量较大，则可能影响系统中其它功能，所以需要缓存大量数据时，优先考虑使用 Redis、Memcache 等缓存产品。
Binding 模块：在调用 SqlSession 相应方法执行数据库操作时，需要制定映射文件中定义的 SQL 节点，如果 SQL 中出现了拼写错误，那就只能在运行时才能发现。为了能尽早发现这种错误，Mybatis 通过 Binding 模块将用户自定义的 Mapper 接口与映射文件关联起来，系统可以通过调用自定义 Mapper 接口中的方法执行相应的 SQL 语句完成数据库操作，从而避免上述问题。注意，在开发中，我们只是创建了 Mapper 接口，而并没有编写实现类，这是因为 Mybatis 自动为 Mapper 接口创建了动态代理对象。



## 4.核心组件

使用 xml 配置文件方式的步骤：
读取配置文件
创建 SqlSessionFactory 
创建 SqlSession。两条分支：自己写 DAO 接口实现类实现 CRUD，或者生成代理对象。
创建 DAO 接口的代理对象。动态代理。
执行 DAO 中的方法
释放资源

SqlSessionFactory：主要负责 MyBatis 框架初始化操作，为开发人员提供 SqlSession 对象。
String resource = "*/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
sqlSessionFactory.openSession();

SqlSessionFactoryBuilder 根据传入的输入流 InputStream 和 environment、properties 属性创建一个 XMLConfigBuilder 对象。
SqlSessionFactoryBuilder 调用 XMLConfigBuilder 的 parse()。XMLConfigBuilder 会解析 /configuration 标签。

以下属性都对应着一个解析方法，都是使用 XPath 把标签进行解析，解析完成后返回一个 DefaultSqlSessionFactory 对象，它是 SqlSessionFactory 的默认实现类。这就是 SqlSessionFactoryBuilder 的初始化流程，通过流程我们可以看到，初始化流程就是对一个个 /configuration 标签下子标签的解析过程。
properties：外部属性，这些属性都是可外部配置且可动态替换的，既可以在典型的 Java 属性文件中配置，亦可通过 properties 元素的子元素来传递。一般用来给 environment 标签中的 dataSource 赋值。
settings：MyBatis 中极其重要的配置，它们会改变 MyBatis 的运行时行为。cacheEnabled、lazyLoadingEnabled。
typeAliases：类型别名，类型别名是为 Java 类型设置的一个名字。它只和 XML 配置有关。
typeHandlers：类型处理器，无论是 MyBatis 在预处理语句 PreparedStatement 中设置一个参数时，还是从结果集中取出一个值时，都会用类型处理器将获取的值以合适的方式转换成 Java 类型。在 org.apache.ibatis.type 包下有很多已经实现好的 TypeHandler。或者实现 org.apache.ibatis.type.TypeHandler 接口，或继承一个很方便的类 org.apache.ibatis.type.BaseTypeHandler，来实现 Java 类型跟数据库类型的相互转换。
objectFactory：对象工厂，MyBatis 每次创建结果对象的新实例时，它都会使用一个对象工厂 ObjectFactory 实例来完成。默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候通过参数构造方法来实例化。如果想覆盖对象工厂的默认行为，则可以通过创建自己的对象工厂来实现。
plugins：插件开发，插件开发是 MyBatis 设计人员给开发人员留给自行开发的接口，MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。MyBatis 允许使用插件来拦截的方法调用包括：Executor、ParameterHandler、ResultSetHandler、StatementHandler 接口
environments：环境配置，MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中。例如，开发、测试和生产环境需要有不同的配置；或者想在具有相同 schema 的多个生产数据库中使用相同的 SQL 映射。虽然 environments 可以指定多个环境，但是 SqlSessionFactory 只能有一个，为了指定创建哪种环境，只要将它作为可选的参数传递给 SqlSessionFactoryBuilder 即可。
mappers：映射器，告诉 MyBatis 去哪里找到这些 SQL 语句，可以通过 resource、url、class、package。

SqlSession
调用 DefaultSqlSessionFactory 中的 openSessionFromDataSource，创建出执行器对象 Executor 和 SqlSession 对象。
SqlSession 对象是 MyBatis 中最重要的一个对象，这个接口能够让你执行命令，获取映射，管理事务。
SqlSession 中定义了一系列模版方法，让你能够执行简单的 CRUD 操作，也可以通过 getMapper 获取 Mapper 层，执行自定义 SQL 语句。
因为 SqlSession 在执行 SQL 语句之前是需要先开启一个会话，涉及到事务操作，所以还会有 commit、rollback、close 等方法。

MapperProxy
MapperProxy 是 Mapper 映射 SQL 语句的关键对象，我们写的 Dao 层或者 Mapper 层都是通过 MapperProxy 来和对应的 SQL 语句进行绑定的。
DefaultSqlSession 对象是 SqlSession 的默认实现类，DefaultSqlSession -> Configuration -> MapperRegistry -> MapperProxyFactory（有 Proxy.newProxyInstance）。
mapperMethod.execute() -> executor.query()

Executor
负责增删改查的具体操作，我们可以简单的将它理解为 JDBC 中 Statement 的封装版。也可以理解为 SQL 的执行引擎，理解为发起人的角色。
Executor 执行器，它有两个实现类，分别是 BaseExecutor 和 CachingExecutor。BaseExecutor 是一个抽象类，是 Executor 的默认实现。这种通过抽象的实现接口的方式是适配器模式的体现。
Executor 是由 Configuration 创建的。ExecutorType。
当有一个查询请求访问的时候，首先会经过 Executor 的实现类 CachingExecutor，先从缓存中查询 SQL 是否是第一次执行。如果是第一次执行的话，那么就直接执行 SQL 语句，并创建缓存（创建 Executor 执行器执行 SQL 语句）。如果第二次访问相同的 SQL 语句的话，那么就会直接从缓存中提取。

StatementHandler：是四大组件中最重要的一个对象，负责操作 Statement 对象与数据库进行交互，在工作时还会使用 ParameterHandler 和 ResultSetHandler 对参数进行映射，对结果进行实体类的绑定。管理创建 Statement 对象或者是 PreparedStatement 对象。
ParameterHandler：负责为 PreparedStatement 的 SQL 语句参数动态赋值。Executor 的 parameterSize() -> StatementHandler 的 setParameters() -> ParameterHandler。
ResultSetHandler：处理 Statement 执行后产生的结果集，生成结果列表处理存储过程执行后的输出参数。按照 Mapper 文件中配置的 ResultType 或 ResultMap 来封装成对应的对象，最后将封装的对象返回即可。



## 5.动态 SQL 

动态 SQL 是 MyBatis 的主要特性之一，在 MyBatis 中我们可以把参数传到 XML 文件，由 MyBatis 对 SQL 及其语法进行解析，MyBatis 支持使用 ${} 和 #{}。

#### $ 和 # 的区别

- 使用 ${} 传入的参数，MyBatis 不会对它进行特殊处理，一般用于传入数据库对象。

- 使用 #{} 传入的参数，MyBatis 默认会将其当成字符串，加双引号。

- 预编译处理中不一样。# 类似 JDBC 中的 PreparedStatement，对于传入的参数，在预处理阶段会使用 ? 代替，待真正查询的时候即在数据库管理系统 DBMS 中才会代入参数，可以有效防止 SQL 注入。而 $ 则是简单的替换，可能导致 SQL 注入成功。因此能使用 # 的地方应尽量使用 #。
- MyBatis 排序时使用 order by 动态参数时需要注意，用 $ 而不是 #。



MyBatis 使用 OGNL 表达式解析对象字段的值，#{} 和 ${} 括号中的值为 POJO 属性名称。

EL 表达式：表达式语言，能替换和简化 JSP 页面中 Java 代码的编写。

语法：${表达式}。

JSP 默认支持 EL 表达式，但也能忽略。

使用：运算和获取值。

隐式对象

JSTL：JSP 标准标签库。和 EL 表达式作用一样。



## 6.其他

insert 可以加 selectKey 获取新增的属性值。
以查询所有满足条件的 User 并封装为 List 为例（自己编写 DAO 实现类）：findAll() -> selectList() -> query() -> queryFromDatabase() -> doQuery() -> query() 出现 JDBC 的 PreparedStatement 对象完成查询 -> handleResultSets() 完成查询结果的封装。
insert 和 delete 最终调用的都是 update。
MyBatis 在 DAO 类中设置 SqlSession sqlSession=sqlSessionFactory.openSession(true); 设置事务自动提交。
抽取重复的 SQL 语句。
映射配置文件中可以设置 *Fields 和 include refid="*Fields"，抽取需要查询属性。

MyBatis 的 DAO 接口也叫 Mapper。
xml 配置文件分为主配置文件 configuration（配置环境和指定映射配置文件或注解方式）和映射配置文件 mapper。
xml 配置文件和注解的差别在得到 mappers 对象的过程，后面创建代理对象和执行查询是一样的。

MyBatis 无需写 DAO 实现类的要求（当然也可以写实现类，但不推荐）：
MyBatis 的映射配置文件位置必须和 DAO 接口的包结构相同。
映射配置文件 mapper 标签的 namespace 属性的值必须是 DAO 接口的全限定类名。
映射配置文件的操作（如 select）的 id 属性的取值必须是 DAO 接口的方法名。

映射配置文件的作用：
根据 namespace + id 找到一条 SQL 语句。获取到 PreparedStatement 对象。
获取结果对象 resultTyepe。

MyBatis XML 标签类型
resultTyepe：在配置中指定把 select 结果封装到一实体类（全限定类名）。用到反射技术。如果是注解方式则是范型代表的实体类。
parameterType：insert update delete。
resultMap：查询结果的列名和实体类的属性名相对应。对应在 SQL 语句中起别名，但开发效率要高。
property：配置连接池。
typeAlias：配置别名。package 也是配置别名，配置后该包下的所有类都会注册别名，而且类名就是别名，不再区分大小写。
id，result 标签的类型属性为 javaType（可选）。

注解：在 DAO 接口上方法上加注解，@Select，@Insert，@Update，@Delete。
使用了注解方式就不能在项目里的特定位置有映射配置文件，否则会报错。
@Results 能将数据库列名与实体类属性相对应。并且可以被多个方法使用。@Results 内有 @One 属性，能完成一对一查询。@Many 一对多。多表查询的 @One 和 @Many 有 select 和 fetchType 属性。
注解使用二级缓存：@CacheNamespace。

在主配置文件中 dataSource 指定 POOLED 和 UNPOOLED 配置数据库连接池。

动态 SQL：根据条件产生不同的 SQL。
if 标签：条件，放在 where 标签里。
foreach 标签：与 SQL 的 in 一起使用表示在某一范围内查询。

多表查询
一对一：resultMap 封装主表属性和从表属性 association，再在主表的 select 标签写 SQL。
一对多：resultMap 从表改为 collection。
多对多：SQL 额外加 left outer join 让数据库表中有多行数据。

一对一、多对一需要立即加载，而一对多、多对多需要延迟加载。
在主配置文件设置延迟加载 lazyLoadingEnabled。

一级缓存：SqlSession 对象的缓存，当调用增删改、commit()、close() 方法时会清空缓存。
二级缓存：SqlSessionFactory 对象的缓存，同一个 SqlSessionFactory 创建的 SqlSession 共享其缓存。需要配置支持二级缓存。二级缓存内存放的数据而不是对象。



## 7.JPA

Java 持久化 API，是一种规范，Hibernate 和 TopLink 都是其实现方式。

#### 优点

标准化：任何符合 JPA 标准的框架都遵循同样的架构，提供相同的访问 API，经过少量的修改就能够在环境下运行。

容器级特性的支持：JPA 框架中支持大数据集、事务、并发等容器级事务。

查询能力：JPA 的查询语言是面向对象而非面向数据库的，它的查询语句 JPQL 查询的是实体类和实体类的属性。

高级特性：JPA 中能够支持面向对象的高级特性，如类之间的继承、多态和类之间的复杂关系。

#### 核心配置文件 persistence.xml

META-INF 文件夹下。

需要配置 persistence-unit，包括 provider 和 properties。properties 包含数据库信息和 JPA 实现方的配置信息。

hibernate.hbm2ddl.auto 对应的 value 为 create 时运行时创建数据库表，并且原来有表先删除再创建新表。update 如果有表不会创建表，none 不会创建表。

JPA 在实体类上进行注解，完成实体类与数据库表以及实体类属性和数据库表字段的映射。

@GeneratedValue：配置主键的生成策略。

Persistence

EntityManagerFactory

EntityManager：有很多方法实现增删改查。entityManager.find() 立即加载，entityManager.getReference() 通过动态代理延迟加载

Tansaction

工具类 JPAUtils：静态代码块。

如何使用 JPA 在数据库中非持久化一个字段？static、final，推荐 transient 和 @Transient。

#### MyBatis 与 Hibernate 的比较

MyBatis 轻量级、上手快、插件丰富，优化起来也方便。Hibernate 重量级、功能齐全、精通较难。

MyBatis 性能稍高。Hibernate 的封装方法性能低，Native 方法性能与 MyBatis 差不多。

MyBatis 的 SQL 自由度高，提供灵活的 SQL 编写方式。Hibernate 的 SQL 自由度低，不过也支持手动写 SQL。

MyBatis 开发效率低，需要自己维护 SQL。Hibernate 开发效率高，DAO 层开发简单，支持 JPA。

MyBatis 所以 SQL 都是依赖数据库编写的，需要针对特定数据库维护 SQL。Hibernate 高度解偶，封装了 JDBC，只需要在配置中指定数据库。

MyBatis 是 POJO 与 SQL 的映射，半 ORM。Hibernate 是 POJO 与数据库的映射，完全 ORM。

MyBatis 自身缓存机制较差。Hibernate 自身缓存机制较好，可避免脏读。

MyBatis 适合复杂查询，集群间跨数据库事务时。Hibernate 适合单数据库，数据量小，无多表关联，数据库结构不稳定。

#### Spring Data JPA

是对 JPA 的封装。

applicationContext.xml：

创建 EntityManagerFactory 对象并交给 Spring 容器管理，包括扫描实体类所在包，JPA 实现方和数据库相关的实现方适配器。

jpaDialect

jpaProperties

数据库连接池

整合 Spring Data JPA

事务管理器和声明式事务

配置包扫描

注解：实体类和表的映射为 @Entity 和 @Table，实体类属性和表字段的映射为 @Id、@GeneratedValue 和 @Column。

DAO 层继承 JpaRepository 和 JpaSpecificationExecutor。使用动态代理 JdkDynamicAopProxy 和 SimpleJpaRepository，最后通过 EntityManager 完成增删改查。

count() exist() findOne() getOne()

@Query：JPQL 或者 SQL 语句。

方法命名规则查询 findBy

Specification 动态查询

JpaSpecificationExecutor 的方法：findOne()，findAll() 分页和排序（方法中的参数），count() 统计。

自定义查询条件：实现 Specification 和 toPredicate(Root,CriteriaBuilder)。其中，Root 获取需要查询的对象属性，CriteriaBuilder 构造查询条件。

root.get()  cb.equal() 

多条件 and 拼接

模糊查询 cb.like()，指定参数类型 .as()

多表关系

ORM 思想，用类的关系表示多表关系。

主表在从表对应实体类的集合上加声明关系 @OneToMany(targetEntity) 和配置外键 @JoinColumn(name,referenceColumnName)。需要主表维护外键。

从表 @ManyToOne(targetEntity) 和外键。

主表 @OneToMany(mappedBy,cascade) 放弃对外键的维护，添加级联。

多对多：@ManyToMany(targetEntity) 和 @JoinTable(name,joinColumns,inverseJoinColumns)。joinColumns 为当前对象在中间表的外键，inverseJoinColumns 对方。

导航查询：多表关系里查询可以查到主表和从表的数据。主表默认延迟加载，从表默认立即加载 。@OneToMany(mappedBy,cascade,fetch) 中的 fetch 可以更改加载方式。