## 什么是 Spring Boot

[Spring Boot 中文文档](https://springdoc.cn/spring-boot/index.html)

Spring Boot 是一个用于构建独立、生产级别的 Spring 应用程序的框架。它旨在简化 Spring 应用程序的开发和部署，通过约定大于配置的思想，使开发者可以更快地上手。

- 简化配置：Spring Boot 自动配置了大部分常见的配置，无需手动设置。这使得开发者不必花费太多时间在配置上，可以专注于业务逻辑。

- 内嵌服务器：Spring Boot 包含了多个内嵌的 Web 服务器（如 Tomcat、Jetty、Undertow），无需单独配置即可运行 Web 应用程序。

- 开箱即用：Spring Boot 提供了大量的“启动器”（starters），这些是预配置的依赖项集合，可以轻松集成各种技术，如数据库、消息队列、安全性等。

- 生产就绪：Spring Boot 提供了一组工具和特性，使得应用程序更容易部署到生产环境中，如性能调优、健康检查、监控等。

## Spring Boot 程序目录结构

### 目录结构

Spring Boot 项目通常遵循一种特定的项目结构，这有助于组织代码并使其易于维护。在这个结构中，src/main/java 包含主要的应用程序代码，src/test/java 包含测试代码，src/main/resources 包含配置文件和其他资源。src/main/java 中通常有一个启动类，这个类负责引导整个 Spring Boot 应用程序的启动过程。这个结构的分层和组织使得项目更具可读性和可维护性。

启动类在外层，所有其他组件在内层，这样才能自动扫描注册其他组件。

```java
my-spring-boot-app/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── example/
│   │   │           └── application/
│   │   │               └── controller/
│   │   │               └── service/
│   │   │               └── model/
│   │   │               └── repository/
│   │   │               └── Application.java
│   │   ├── resources/
│   │   │   └── application.properties
│   ├── test/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── example/
│   │   │           └── application/
│   │   │               └── controller/
│   │   │               └── service/
│   │   │               └── ApplicationTests.java
│   └── pom.xml // Maven 构建配置文件
└── target/ // 构建输出目录，包含生成的编译和打包文件
```

### 启动类

Spring Boot 应用程序的启动类是一个带有 main 方法的 Java 类。这个类的主要任务是引导应用程序的启动过程。通过 @SpringBootApplication 注解，它告诉 Spring Boot 这是应用程序的主类。在 main 方法中，通过 SpringApplication.run() 方法加载 Spring 应用程序上下文，启动嵌入式的 Web 服务器（如 Tomcat）并执行应用程序。这个类是应用程序的入口点，它在整个应用程序生命周期中起着关键作用。

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {
  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }
}
```

## 依赖管理

Spring Boot 使用 Maven 或 Gradle 进行依赖管理，这使得添加、移除和管理依赖项变得非常简单。

在 Maven 中，您可以在 pom.xml 文件中定义依赖，而在 Gradle 中，您可以在 build.gradle 文件中定义它们。

Spring Boot 通过提供各种 "Starter" 依赖项来简化配置，这些 Starter 是预配置的依赖项集合，可以轻松集成各种技术，如 Web、数据访问、安全性等。同时，Spring Boot 插件可以帮助您构建可执行的 JAR 文件，这些文件包含了所有必要的依赖项，使得部署变得更加简单。

### Maven

```xml
<!-- 定义项目的 Maven 坐标 -->
<groupId>com.example</groupId>
<artifactId>my-spring-boot-app</artifactId>
<version>1.0.0</version>
<packaging>jar</packaging>

<properties>
    <java.version>20</java.version>
</properties>

<!-- 引入 Spring Boot 的父 POM -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.1.3</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<!-- 定义项目依赖 -->
<dependencies>
    <!-- Spring Boot Starter 依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
    </dependency>
    <!-- 其他依赖 -->
</dependencies>

<!-- Maven 插件配置 -->
<build>
    <plugins>
        <!-- Spring Boot Maven 插件，用于构建可执行的 JAR 文件 -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

### Gradle

```groovy
plugins {
    id 'java'  // Java 插件
    id 'org.springframework.boot' version '3.1.2'  // 使用 Spring Boot 插件
    id 'io.spring.dependency-management' version '1.1.2' // 引入 Spring 依赖管理插件
}

group = 'com.example'
version = '1.0.0'

sourceCompatibility = '19'

repositories {
    mavenCentral()  // 使用 Maven 中央仓库
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'  // Spring Boot Starter 依赖
    // 其他依赖

    testImplementation 'org.springframework.boot:spring-boot-starter-test'  // 测试依赖
}

// Spring Boot 插件配置
bootJar {
    mainClassName = 'com.example.MySpringBootApplication'  // 指定应用程序的主类
}
```

### Starter

**spring-boot-starter-web**: 用于构建 Web 应用程序，包含了 Spring MVC 以及内嵌的 Tomcat 容器等。

**spring-boot-starter-data-jpa**: 用于访问关系型数据库，包括 JPA 和 Hibernate 支持。

**spring-boot-starter-data-redis**: 用于访问 Redis 缓存和数据存储。

**spring-boot-starter-security**: 用于实现安全性和身份验证。

**spring-boot-starter-test**: 用于单元测试和集成测试，包括 JUnit、TestNG 等。

**spring-boot-starter-logging**: 用于日志记录，集成了常见的日志框架，如 Logback。

**spring-boot-starter-actuator**: 用于监控和管理 Spring Boot 应用程序。

**spring-boot-starter-batch**: 用于构建批处理作业。

**spring-boot-starter-websocket**: 用于构建 WebSocket 应用程序。

**spring-boot-starter-amqp**: 用于访问消息代理，如 RabbitMQ。

**spring-boot-starter-cloud-connectors**: 用于连接云服务，如数据库、消息代理等。

**spring-boot-starter-data-elasticsearch**: 用于访问 Elasticsearch 搜索引擎。

**spring-boot-starter-data-jest**: 用于访问 Elasticsearch 的 Jest 客户端。

**spring-boot-starter-data-solr**: 用于访问 Apache Solr 搜索引擎。

**spring-boot-starter-graphql**: 用于构建 GraphQL API。

**spring-boot-starter-validation**: 用于数据验证。

## 常见注解

**@SpringBootApplication**：这是一个元注解，它将多个注解组合在一起，包括 **@SpringBootConfiguration**、**@EnableAutoConfiguration** 和 **@ComponentScan**。它告诉 Spring Boot 这是一个 Spring 应用程序的主类。

**@EnableAutoConfiguration**：开启 Spring Boot 的自动配置，通常用于主应用程序类上。

**@Configuration**：用于定义 Java 配置类，替代传统的 XML 配置文件，这个类在容器初始化时被传入。

**@SpringBootConfiguration**：允许在 Context 中注册额外的 Bean 或导入额外的配置类。这是 Spring 标准的 **@Configuration** 的替代方案，有助于在你的集成测试中检测配置。

**@ComponentScan**：在启动类上添加，你的所有应用组件（**@Component**、**@Service**、**@Repository**、**@Controller** 和其他）都会自动注册为 Spring Bean。必须合理设计包的层次结构，启动类在外层，所有其他组件在下层。

**@Controller**：用于标记一个类作为 Spring MVC 控制器。方法的返回值会被解释为视图名称，Spring MVC 会尝试根据该名称查找匹配的视图模板文件。

**@RestController**：**@RestController** 是 **@Controller** 的特殊变体，通常用于创建 RESTful Web 服务。控制器类标记为 **@RestController** 后，其中的方法默认情况下不会返回视图名称，而是将方法的返回值直接作为 HTTP 响应的主体。

**@RequestMapping**：映射 HTTP 请求到具体的处理方法。

**@RequestParam**：用于从 HTTP 请求中获取请求参数的值，通常用于控制器方法的参数上。

**@PathVariable**：从 URL 中获取路径参数的值，通常用于 RESTful 风格的控制器方法。

**@RequestBody**：将 HTTP 请求的请求体（通常是 JSON 或 XML 数据）绑定到一个对象上，通常用于接收 POST 请求的数据。

**@ResponseBody**：指示一个方法的返回值应该直接作为 HTTP 响应的主体，通常用于 RESTful 风格的控制器方法。

**@GetMapping**、**@PostMapping**、**@PutMapping**、**@DeleteMapping**：处理 HTTP 不同类型的请求。

**@ExceptionHandler**：处理控制器中抛出的异常，它通常用于全局异常处理，也可以用于特定控制器方法。

**@ControllerAdvice**：用于全局异常处理和全局数据绑定。在 Spring Boot 应用程序中，它通常与 @ExceptionHandler 注解一起使用。修饰的类用于捕获和处理在控制器中抛出的异常。

**@ConfigurationProperties**：用于将配置属性绑定到 JavaBean 上。它允许您将应用程序的配置参数（通常在 `application.properties` 或 `application.yml` 文件中定义）映射到 Java 类的字段或属性上。

## 配置文件

Spring Boot 可以让你将配置外部化，这样你就可以在不同的环境中使用相同的应用程序代码。 你可以使用各种外部配置源，包括 Java properties 文件、YAML 文件、环境变量和命令行参数。

属性值可以通过使用 @Value 注解直接注入你的 Bean，也可以通过 Spring 的 Environment 访问，或者通过 @ConfigurationProperties 绑定到对象。

### 配置文件的形式

- application.properties：这是 Spring Boot 应用程序的主要属性配置文件。您可以在其中定义应用程序的各种属性，例如数据库连接信息、端口号、日志级别等。

- application.yml：与.properties 文件类似，.yml 文件也用于配置应用程序属性。

- 多环境配置：Spring Boot 允许您创建多个配置文件，以适应不同的环境，例如 application-dev.properties 用于开发环境，application-prod.properties 用于生产环境等。在启动应用程序时，可以通过 spring.profiles.active 属性来选择要使用的配置文件。

- 外部配置：Spring Boot 支持从外部文件加载配置，例如从/etc/myapp/application.properties 加载配置。您可以使用 spring.config.name 和 spring.config.location 属性来指定外部配置文件的名称和位置。

- 自定义属性：除了内置属性外，您还可以在配置文件中定义自定义属性，然后在应用程序中使用@Value 注解或@ConfigurationProperties 来注入这些属性。

### 内置属性

#### Web 服务器配置

server.port：指定 Web 服务器监听的端口号。

server.servlet.context-path：指定应用程序的上下文路径。

#### 数据源配置

spring.datasource.url：数据源的 URL。

spring.datasource.username：数据库用户名。

spring.datasource.password：数据库密码。

#### 日志配置

logging.level.\*：指定各个日志记录器的级别。

logging.file：指定日志文件的位置。

#### 应用程序配置

spring.application.name：应用程序的名称。

spring.profiles.active：激活的 Spring 配置文件（用于多环境配置）。

#### Spring Boot Actuator（监视和管理 Spring Boot 应用程序）

management.endpoints.web.exposure.include：指定哪些 Actuator 端点暴露给 HTTP 请求。

management.server.port：指定 Actuator 端点的 HTTP 端口。

#### Spring Boot DevTools

spring.devtools.restart.enabled：启用或禁用 DevTools 的自动重启。

#### Spring Security

spring.security.user.name：定义内置的用户的用户名。

spring.security.user.password：定义内置用户的密码。

## 一个简单的 Spring Boot 程序

### 步骤 1：创建 Spring Boot 项目

1. 使用 Spring Initializer 创建项目：你可以访问 [Spring Initializer](https://start.spring.io/) 网站来创建一个新的 Spring Boot 项目。在项目的初始化过程中，你可以选择所需的依赖，确保选择了 Spring Web 和 Spring Data JPA 依赖。

2. 导入项目到你的 IDE：下载生成的项目压缩文件，然后导入到你喜欢的集成开发环境（IDE）中，如 Eclipse、IntelliJ IDEA 或 Visual Studio Code。

### 步骤 2：创建数据库

1. 创建 MySQL 数据库：在你的 MySQL 数据库中创建一个新的数据库，用于存储用户数据。你可以使用 MySQL 命令行工具或数据库管理工具来完成这一步。

2. 定义用户表：创建一个用户表，用于存储用户信息。表可以包括字段如下：id（主键，自增长）、username（用户名）、email（电子邮件）、password（密码）等。

### 步骤 3：定义实体类

1. 创建用户实体类：在项目的主包（通常是 com.example.demo）下创建一个用户实体类，用于映射到数据库中的用户表。这个类需要使用 JPA 注解来标记，以便 Spring Boot 可以将其与数据库表进行映射。例如：

   ```java
   @Entity
   @Table(name = "user")
   public class User {
       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Long id;

       private String username;
       private String email;
       private String password;

       // 省略构造函数、Getter 和 Setter 方法
   }
   ```

### 步骤 4：创建数据访问层

1. 创建 UserRepository 接口：在项目中创建一个 UserRepository 接口，它继承自 Spring Data JPA 的 JpaRepository 接口。这个接口将用于执行对用户表的 CRUD 操作。

   ```java
   public interface UserRepository extends JpaRepository<User, Long> {
       // 可以定义一些自定义的查询方法
   }
   ```

### 步骤 5：创建控制器

1. 创建用户控制器：创建一个用户控制器类，用于处理 RESTful API 的请求和响应。使用 @RestController 注解标记这个类。

   ```java
   @RestController
   @RequestMapping("/api/users")
   public class UserController {
       // 控制器方法将在下一步中添加
   }
   ```

### 步骤 6：实现 RESTful API 接口

1. 添加 API 方法：在用户控制器中添加方法来实现创建、读取、更新和删除（CRUD）用户的操作。使用 @PostMapping、@GetMapping、@PutMapping 和 @DeleteMapping 注解来映射不同的 HTTP 请求。

   ```java
   @PostMapping
   public ResponseEntity<User> createUser(@RequestBody User user) {
       // 创建新用户的逻辑
   }

   @GetMapping("/{id}")
   public ResponseEntity<User> getUserById(@PathVariable Long id) {
       // 根据ID获取用户的逻辑
   }

   @PutMapping("/{id}")
   public ResponseEntity<User> updateUser(@PathVariable Long id, @RequestBody User user) {
       // 更新用户的逻辑
   }

   @DeleteMapping("/{id}")
   public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
       // 删除用户的逻辑
   }
   ```

### 步骤 7：配置数据库连接

1. 配置数据库连接信息：在 application.properties 或 application.yml 文件中，配置数据库连接信息，包括数据库 URL、用户名和密码。

   ```properties
   spring.datasource.url=jdbc:mysql://localhost:3306/your_database
   spring.datasource.username=your_username
   spring.datasource.password=your_password
   ```

### 步骤 8：运行应用程序

1. 运行 Spring Boot 应用程序：在 IDE 中运行你的 Spring Boot 应用程序。它将启动内嵌的 Tomcat 服务器，并在指定的端口上监听请求。

2. 测试 API 接口：使用工具如 Postman 或浏览器来测试你的 RESTful API 接口。可以发送 POST、GET、PUT 和 DELETE 请求来测试创建、读取、更新和删除用户的功能。

### 步骤 9：部署应用程序

1. 部署到生产环境：当你的应用程序准备好了，可以将其部署到生产环境中。选择一个合适的云主机提供商，如阿里云，来部署你的 Spring Boot 应用程序。

   1. 构建可执行 JAR 文件：运行构建命令，例如 mvn clean package 或 ./gradlew build
   2. 在服务器上创建一个用于存储生产环境属性的配置文件，使用外部化配置来动态加载配置属性。
   3. 使用 `java -jar your-application.jar` 来启动 jar 包

2. 配置数据库：在生产环境中配置数据库连接，确保安全性和性能。

3. 监控和日志：配置监控和日志记录，以便及时发现和解决问题。

### 步骤 10：配置反向代理

1. 为了增加安全性和性能，通常建议在生产环境中使用反向代理服务器，如 Nginx。配置反向代理服务器以将客户端请求转发到 Spring Boot 应用程序的端口（通常是 8080）。

2. 配置 SSL/TLS：如果你的应用程序需要通过 HTTPS 提供安全性，那么你需要配置 SSL/TLS 证书以加密通信。可以使用免费的证书颁发机构（如 Let's Encrypt）获取 SSL 证书。
