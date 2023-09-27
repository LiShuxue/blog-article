## Spring 是什么

[Spring 中文文档](https://springdoc.cn/docs/)

Spring，在不同的语境中，可以意味着不同的东西。它可以用来指代 Spring 框架项目本身，它是一切的开始。随着时间的推移，其他 Spring 项目也被建立在 Spring 框架之上。大多数时候，当人们说 "Spring" 时，他们指的是整个项目家族（全家桶）。

## 核心概念

### Spring 模块

Spring 框架被分为多个模块，每个模块提供不同的功能，你可以根据需要选择性地使用。一些常见的模块包括：

- **Spring Core Container：** 这是 Spring 框架的核心模块，提供了配置模型和依赖注入机制。
- **Spring AOP：** 用于面向切面编程，允许你添加额外的功能而不改变主要的代码。
- **Spring JDBC：** 简化了数据库访问。
- **Spring MVC：** 用于构建 Web 应用程序的模块。

### 依赖注入

依赖注入（Dependency Injection，DI）是 Spring 的核心概念之一，它允许你将一个组件所依赖的其他组件（例如服务或资源）注入到它中，而不是由组件自己创建或管理这些依赖。这使得组件之间的耦合性降低，提高了代码的可维护性和可测试性。

Spring 支持多种方式的依赖注入，包括构造函数注入、Setter 方法注入和字段注入（通过注解）等。这些方式允许你以不同的方式将依赖项注入到 Bean 中。

```java
public class UserService {
    // 注解注入
    @Autowired
    private UserRepository userRepository;

    // 构造函数注入
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    // Setter方法注入
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### 控制反转

控制反转（Inversion of Control，IoC）是一种软件设计原则，它指的是将程序的控制权从程序内部转移到外部，从而实现程序的解耦和灵活性。

在传统的编程模型中，程序的控制逻辑通常由程序自身来控制对象的创建和管理。而在控制反转中，控制权被反转，由外部框架或容器来控制对象的创建、管理和注入依赖。

控制反转通常和依赖注入密切相关。依赖注入是控制反转的一种具体实现方式，通过将依赖对象通过构造函数、属性注入等方式传递给需要的类。通过依赖注入，对象可以获取它所需要的依赖，而不需要自己负责创建这些依赖。

### 配置模型

Spring 提供了一种灵活的配置模型，允许你以多种方式配置应用程序的组件和行为。

Spring 允许你使用 XML、注解或 Java 代码来配置应用程序的 Bean 和依赖关系。配置文件是告诉 Spring 容器如何创建和连接 Bean 的地方。

```xml
<!-- applicationContext.xml -->
<bean id="userDAO" class="com.example.demo.UserDAO" />
<bean class="com.example.demo.UserService" id="userService">
    <property name="userDAO" ref="userDAO"/>
</bean>
```

随着 Spring 3.0 以后的版本，XML 配置文件不再是唯一的配置方式。Spring 还支持基于 Java 的配置（通过 Java 类）和注解配置（通过使用注解标记 Bean）等方式。这些方法提供了更灵活和类型安全的配置选项。因此，在一些现代的 Spring 项目中，你可能会看到更多使用基于 Java 或注解的配置方式。

### IoC 容器

Spring 的核心就是提供了一个 IoC 容器，它可以管理所有轻量级的 JavaBean 组件，提供的底层服务包括组件的生命周期管理、配置和组装服务、AOP 支持，以及建立在 AOP 基础上的声明式事务服务等。

Spring IoC 容器负责管理你的应用程序中的所有对象（bean），这个容器负责创建、配置和连接 bean，使它们可以一起工作。如 ApplicationContext 接口。

容器在启动时需要实例化，通过配置文件中配置的依赖关系来自动创建实例，关联 bean 等。持有 IoC 容器后，通过 getBean() 方法获取 Bean 的引用。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

IUserService s = (IUserService)context.getBean("userService");
Demo1User user = new Demo1User("zz", 22);
s.insert(user);
```

在 Spring 3.0 之后，Spring 引入了一种新的方式来配置容器，称为基于 Java 的配置（Java Configuration）。这种方式不再强制依赖于 XML 配置文件，而是通过纯粹的 Java 类来配置 Spring 容器和 Bean 定义。这种方式通常使用 @Configuration 注解标记配置类，使用 @Bean 注解来定义 Bean。

```java
// 使用基于Java的配置实例化容器
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

// 通过容器获取Bean
MyBean myBean = context.getBean(MyBean.class);
```

### Bean

在 Spring 中，术语 "Bean" 通常指的是被 Spring 容器所管理和控制的对象实例。这些 Bean 可以是各种不同的对象，包括普通的 POJO（Plain Old Java Object）如 User 对象，也可以是特定于 Spring 框架的类，比如 UserService，UserDAO 等。

当配置一个 Bean 时，你通常会在 Spring 的配置文件（如 XML 配置文件或 Java 配置类）中定义 Bean 的标识符、类名以及相关的属性或依赖项。然后，Spring 容器会根据配置来实例化和初始化这些 Bean，并在需要时将它们注入到其他 Bean 中。

#### 生命周期

Spring 允许你定义 Bean 的生命周期方法，例如初始化方法和销毁方法。这些方法在 Bean 的创建和销毁时自动调用，你可以在其中执行一些自定义的逻辑。

### POJO

POJO（Plain Old Java Object）是指普通的 Java 对象，它通常是一个简单的 Java 类，没有特殊的要求或依赖于特定框架。

在 Java 中，POJO 通常用于以下方面：

- **数据封装：** POJO 用于封装应用程序中的数据。这些对象通常具有一组字段（属性）和与之相关的 getter 和 setter 方法，用于访问和修改这些字段。

- **领域模型：** POJO 可用于表示应用程序的领域模型，也就是应用程序处理的领域中的实体、对象或概念。例如，一个在线商店应用程序可能会有一个 Product POJO 来表示产品。

### 面向切面编程（AOP）

面向切面编程是指你在应用程序上添加一些额外的功能，而不必改变主要的代码。这些额外的功能可以包括日志记录、安全性、事务管理等。

当调用某个方法时，如何对调用方法进行拦截，并在拦截前后进行安全检查、日志、事务等处理，就相当于完成了所有业务功能呢？Spring 的 AOP 实现就是基于 JVM 的动态代理。

使用 AOP，我们需要给@Configuration 类加上一个@EnableAspectJAutoProxy 注解，然后在日志类等需要切进去的类上，标记@Component 和@Aspect，使用@Before，@After，@Around 等拦截器。

Spring 的 IoC 容器看到@EnableAspectJAutoProxy 这个注解，就会自动查找带有@Aspect 的 Bean，然后根据每个方法的@Before、@Around 等注解把 AOP 注入到特定的 Bean 中。

## Spring 常见项目结构

**Controller 层 / Web 服务层：** src/main/java/com/example/application/controller：Controller 层的 Java 类文件。Controller 层通常包括处理请求的方法，并返回响应。

**Service 层：** src/main/java/com/example/application/service：Service 层的 Java 类文件。Service 层负责处理应用程序的业务逻辑，例如数据处理、计算和交互。

**DAO（Data Access Object）层：** src/main/java/com/example/application/dao：DAO 层的 Java 类文件。DAO 层用于处理与数据存储的交互，例如数据库访问。DAO 类负责执行数据库查询、插入、更新和删除操作。

**Model 层 / Bean 层 / POJO 层：** src/main/java/com/example/application/model：领域模型类（POJO）的 Java 类文件。这些类用于表示应用程序中的领域对象或实体。这些对象可以在整个应用程序中被多个层次共享和重用。在 Spring 中，这些 POJO 类通常会被配置为 Spring Bean，并由 Spring 容器进行管理。

**配置层：** src/main/java/com/example/application/config：Spring 配置类的 Java 类文件，通常包括应用程序的核心配置。它的作用是定义和配置 Spring 容器以及应用程序中的 Bean。

**工具类层：** src/main/java/com/example/application/util：实用工具类的 Java 类文件。

**自定义异常层：** src/main/java/com/example/application/exception：自定义异常类的 Java 类文件。

**安全层：** src/main/java/com/example/application/security：安全相关类的 Java 类文件（如果应用程序需要安全功能）。管理和控制应用程序的安全性。Spring Security 是 Spring 框架的一个模块，用于处理身份验证和授权，以保护应用程序的资源和数据。

**src/main/resources：** 资源目录，包含应用程序的配置文件、静态资源文件、国际化资源文件等。

**pom.xml：** Maven 项目的配置文件，用于管理项目的依赖项和构建配置。

除上述层外，还可能有 **定时任务层，消息队列层，缓存层，数据转换层** 等。

## 常见注解

Spring 强调注解的使用，如 **@Component、@Service、@Controller** 等，以简化配置并提高开发效率。

- **@Configuration**：用于定义 Java 配置类，替代传统的 XML 配置文件，这个类在容器初始化时被传入。

- **@ComponentScan**：在启动类上添加，你的所有应用组件（@Component、@Service、@Repository、@Controller 和其他）都会自动注册为 Spring Bean。必须合理设计包的层次结构，启动类在外层，所有其他组件在下层。

- **@Component**：将一个类标记为 Spring 组件，告诉 Spring 它需要被扫描并创建为 Bean。通常用于普通的 Java 类，在容器初始化时会创建一个单例 bean。

- **@Scope**：指定 Bean 的作用域，例如单例（Singleton）、原型（Prototype）、会话（Session）等。

- **@Controller**：标记一个类作为 Spring MVC 控制器，用于处理 HTTP 请求和生成 HTTP 响应。

- **@Service**：用于标记一个类作为 service 层的 Bean，通常用于编写业务逻辑。

- **@Repository**：标记一个类作为数据访问层（DAO）的 Bean，通常用于数据访问类。

- **@Autowired**：自动注入依赖，通常用于构造函数、Setter 方法或字段上。Spring 会自动解析并注入所需的 Bean。如果一个 Bean 有多个构造函数，你需要用 **@Autowired** 注解来告诉 Spring 该用哪个构造函数进行注入。不推荐使用。

- **@Bean**：创建第三方 Bean，在 **@Configuration** 类中编写一个 Java 方法创建并返回它，该方法用 **@Bean** 注解。

- **@Qualifier**：相同类型的 Bean 只能有一个指定为 **@Primary**，其他必须用 **@Qualifier("beanName")** 指定别名。可以用 **@Bean("name")** 指定别名，也可以用 **@Bean**+**@Qualifier("name")** 指定别名，注入时与 **@Autowired** 一起使用。

- **@PostConstruct**：在 Bean 初始化后立即执行的方法，通常用于执行一些初始化操作。

- **@PreDestroy**：在 Bean 销毁之前执行的方法，通常用于执行一些清理操作。

- **@Value**：用于注入配置文件或者其他文件的值，可以用于字段、构造函数参数、Setter 方法等。**@Value("classpath:/logo.txt")**。注入 Resource 最常用的方式是通过 classpath，即类似 **classpath:/logo.txt** 表示在 classpath 中搜索 logo.txt 文件。

- **@PropertySource**：自动读取配置文件，配合 **@Value** 就可以取到值。**@PropertySource("app.properties")** 表示读取 classpath 的 app.properties，**@Value("${app.name}")** 表示读取配置文件中 key 为 app.name 的值。

- **@Profile**：应用程序可以通过 Profile 配置来表示不同的环境。所以代码中可以根据注解 **@Profile** 来决定在某个环境是否创建某个 bean。

- **@Conditional**：Spring 还可以根据 **@Conditional** 决定是否创建某个 Bean。

## 其他

如果使用 Spring 构建 RESTful API，通常使用 Spring MVC 或 Spring Boot 来实现。

如果使用 Spring 对数据库访问，可以使用 Spring Data JPA 或者 Spring Data JDBC 等，当然也可以使用 MyBatis 或者 Hibernate。
