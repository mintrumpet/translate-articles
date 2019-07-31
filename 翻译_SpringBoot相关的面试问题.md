# 翻译 | SpringBoot相关的面试问题

> 原文自工程师baeldung博客，[传送门](https://www.baeldung.com/spring-boot-interview-questions)

> 最近到了春招的时间，各位同学们都开始努力准备面试相关的事项，这边 Eugen 老师在  baeldung 中发表了一篇和 SpringBoot 相关的面试题目以及解答，希望对大家有帮助。 

## 概述

从诞生那天开始，SpringBoot 就在 Spring 生态中扮演一个重要的地位。这个项目的自动化部署功能使我们的编程工作变得更方便。

在这篇文章当中，我们将介绍一些在工程师面试中可能会出现的 Spring Boot 相关的常见问题。

## 问题

### Q1. Spring 和 SpringBoot 有什么不同？

Spring 框架提供多种特性使得 web 应用开发变得更简便，包括依赖注入、数据绑定、切面编程、数据存取等等。

随着时间推移，Spring 生态变得越来越复杂了，并且应用程序所必须的配置文件也令人觉得可怕。这就是 Spirng Boot 派上用场的地方了 – 它使得 Spring 的配置变得更轻而易举。

实际上，Spring 是 *unopinionated*（予以配置项多，倾向性弱） 的，Spring Boot 在平台和库的做法中更 *opinionated* ，使得我们更容易上手。

这里有两条 SpringBoot 带来的好处：

- 根据 classpath 中的 artifacts 的自动化配置应用程序
- 提供非功能性特性例如安全和健康检查给到生产环境中的应用程序

### Q2. 怎么使用 Maven 来构建一个 SpringBoot 程序？

就像引入其他库一样，我们可以在 Maven 工程中加入 SpringBoot 依赖。然而，最好是从 *spring-boot-starter-parent* 项目中继承以及声明依赖到 Spring Boot starters。这样做可以使我们的项目可以重用 SpringBoot 的默认配置。

继承 *spring-boot-starter-parent* 项目依赖很简单 – 我们只需要在 *pom.xml* 中定义一个 *parent* 节点：

``` xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.1.RELEASE</version>
</parent>
```

我们可以在 Maven central 中找到 *spring-boot-starter-parent* 的最新版本。

**使用 starter 父项目依赖很方便，但并非总是可行**。例如，如果我们公司都要求项目继承标准 POM，我们就不能依赖 SpringBoot starter 了。

这种情况，我们可以通过对 POM 元素的依赖管理来处理：

``` xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.1.1.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

最后，我们可以添加 SpringBoot starter 中一些依赖，然后我们就可以开始了。

### Q3. SpringBoot starter 作用在什么地方？

依赖管理是所有项目中至关重要的一部分。当一个项目变得相当复杂，管理依赖会成为一个噩梦，因为当中涉及太多 artifacts 了。

这时候 SpringBoot starter 就派上用处了。每一个 stater 都在扮演着提供我们所需的 Spring 特性的一站式商店角色。其他所需的依赖以一致的方式注入并且被管理。

所有的 starter 都归于 *org.springframework.boot* 组中，并且它们都以由 *spring-boot-starter-* 开头取名。**这种命名方式使得我们更容易找到 starter 依赖，特别是当我们使用那些支持通过名字查找依赖的 IDE 当中**。

在写这篇文章的时候，已经有超过50个 starter了，其中最常用的是：

- *spring-boot-starter*：核心 starter，包括自动化配置支持，日志以及 YAML
- *spring-boot-starter-aop*：Spring AOP 和 AspectJ 相关的切面编程 starter
- *spring-boot-starter-data-jpa*：使用 Hibernate Spring Data JPA 的 starter
- *spring-boot-starter-jdbc*：使用 HikariCP 连接池 JDBC 的 starter
- *spring-boot-starter-security*：使用 Spring Security 的 starter
- *spring-boot-starter-test*：SpringBoot 测试相关的 starter
- *spring-boot-starter-web*：构建 restful、springMVC 的 web应用程序的  starter

### Q4. 怎么禁用某些自动配置特性？

如果我们想禁用某些自动配置特性，可以使用 *@EnableAutoConfiguration* 注解的 *exclude* 属性来指明。例如，下面的代码段是使 *DataSourceAutoConfiguration* 无效：

``` java
// other annotations
@EnableAutoConfiguration(exclude = DataSourceAutoConfiguration.class)
public class MyConfiguration { }
```

如果我们使用 *@SpringBootApplication* 注解 — 那个将 *@EnableAutoConfiguration* 作为元注解的项，来启用自动化配置，我们能够使用相同名字的属性来禁用自动化配置：

``` java
// other annotations
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class MyConfiguration { }
```

我们也能够使用 *spring.autoconfigure.exclude* 环境属性来禁用自动化配置。*application.properties* 中的这项配置能够像以前那样做同样的事情：

``` properties
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

### Q5. 怎么注册一个定制的自动化配置？

为了注册一个自动化配置类，我们必须在 *META-INF/spring.factories* 文件中的    *EnableAutoConfiguration* 键下列出它的全限定名：

``` properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.baeldung.autoconfigure.CustomAutoConfiguration
```

如果我们使用 Maven 构建项目，这个文件需要放置在在 package 阶段被写入完成的 *resources/META-INF* 目录中。

### Q6. 当 bean 存在的时候怎么置后执行自动配置？

为了当 bean 已存在的时候通知自动配置类置后执行，我们可以使用 *@ConditionalOnMissingBean* 注解。这个注解中最值得注意的属性是：

- value：被检查的 beans 的类型
- name：被检查的 beans 的名字

当将 *@Bean* 修饰到方法时，目标类型默认为方法的返回类型：

``` java
@Configuration
public class CustomConfiguration {
    @Bean
    @ConditionalOnMissingBean
    public CustomService service() { ... }
}
```

### Q7. 怎么将 SpringBoot web 应用程序部署为 JAR 或 WAR 文件？

通常，我们将 web 应用程序打包成 WAR 文件，然后将它部署到另外的服务器上。这样做使得我们能够在相同的服务器上处理多个项目。当 CPU 和内存有限的情况下，这是一种最好的方法来节省资源。

然而，事情发生了转变。现在的计算机硬件相比起来已经很便宜了，并且现在的注意力大多转移到服务器配置上。部署中对服务器配置的一个细小的失误都会导致无可预料的灾难发生。

Spring 通过提供插件来解决这个问题，也就是 *spring-boot-maven-plugin* 来打包 web 应用程序到一个额外的 JAR 文件当中。为了引入这个插件，只需要在 *pom.xml* 中添加一个 *plugin* 属性：

``` xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

有了这个插件，我们会在执行 *package* 步骤后得到一个 JAR 包。这个 JAR 包包含所需的所有依赖以及一个嵌入的服务器。因此，我们不再需要担心去配置一个额外的服务器了。

我们能够通过运行一个普通的 JAR 包来启动应用程序。

注意一点，为了打包成 JAR 文件，*pom.xml* 中的 *packgaing*  属性必须定义为 *jar*：

``` xml
<packaging>jar</packaging>
```

如果我们不定义这个元素，它的默认值也为 *jar*。

如果我们想构建一个 WAR 文件，将 *packaging* 元素修改为 *war*：

``` xml
<packaging>war</packaging>
```

并且将容器依赖从打包文件中移除：

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```

执行 Maven 的 *package* 步骤之后，我们得到一个可部署的 WAR 文件。

### Q8. 怎么使用 SpringBoot 去执行命令行程序？

像其他 Java 程序一样，一个 SpringBoot 命令行程序必须要有一个 *main* 方法。这个方法作为一个入口点，通过调用 *SpringApplication#run* 方法来驱动程序执行：

``` java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class);
        // other statements
    }
}
```

*SpringApplication* 类会启动一个 Spirng 容器以及自动化配置 beans。

要注意的是我们必须把一个配置类传递到 *run* 方法中作为首要配置资源。按照惯例，这个参数一般是入口类本身。

在调用 *run* 方法之后，我们可以像平常的程序一样执行其他语句。

### Q9. 有什么外部配置的可能来源？

SpringBoot 对外部配置提供了支持，允许我们在不同环境中运行相同的应用。我们可以使用 properties 文件、YAML 文件、环境变量、系统参数和命令行选项参数来声明配置属性。

然后我们可以通过 *@Value* 这个通过 *@ConfigurationProperties* 绑定的对象的注解或者实现 *Enviroment* 来访问这些属性。

以下是最常用的外部配置来源：

- 命令行属性：命令行选项参数是以双连字符（例如，=）开头的程序参数，例如 *–server.port=8080*。SpringBoot将所有参数转换为属性并且添加到环境属性当中。
- 应用属性：应用属性是指那些从 *application.properties* 文件或者其 YAML 副本中获得的属性。默认情况下，SpringBoot会从当前目录、classpath 根目录或者它们自身的 *config* 子目录下搜索该文件。
- 特定 *profile* 配置：特殊概要配置是从 *application-{profile}.properties* 文件或者自身的 YAML 副本。*{profile}* 占位符引用一个在用的 *profile*。这些文件与非特定配置文件位于相同的位置，并且优先于它们。

### Q10. SpringBoot 支持松绑定代表什么？

SpringBoot中的松绑定适用于配置属性的类型安全绑定。使用松绑定，环境属性的键不需要与属性名完全匹配。这样就可以用驼峰式、短横线式、蛇形式或者下划线分割来命名。

例如，在一个有 *@ConfigurationProperties* 声明的 bean 类中带有一个名为 *myProp* 的属性，它可以绑定到以下任何一个参数中，*myProp*、 *my-prop*、*my_prop* 或者  *MY_PROP*。

### Q11. SpringBoot DevTools 的用途是什么？

SpringBoot 开发者工具，或者说 DevTools，是一系列可以让开发过程变得简便的工具。为了引入这些工具，我们只需要在 *POM.xml* 中添加如下依赖：

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

*spring-boot-devtools* 模块在生产环境中是默认禁用的，archives 的 repackage 在这个模块中默认也被排除。因此，它不会给我们的生产环境带来任何开销。

通常来说，DevTools 应用属性适合于开发环境。这些属性禁用模板缓存，启用 web 组的调试日志记录等等。最后，我们可以在不设置任何属性的情况下进行合理的开发环境配置。

每当 classpath 上的文件发生更改时，使用 DevTools 的应用程序都会重新启动。这在开发中非常有用，因为它可以为修改提供快速的反馈。

默认情况下，像视图模板这样的静态资源修改后是不会被重启的。相反，资源的更改会触发浏览器刷新。注意，只有在浏览器中安装了 LiveReload 扩展并以与 DevTools 所包含的嵌入式 LiveReload 服务器交互时，才会发生。

### Q12. 怎么编写一个集成测试？

当我们使用 Spring 应用去跑一个集成测试时，我们需要一个 *ApplicationContext*。

为了使我们开发更简单，SpringBoot 为测试提供一个注解 – *@SpringBootTest*。这个注释由其 classes 属性指示的配置类创建一个 *ApplicationContext*。

**如果没有配置 classes 属性，SpringBoot 将会搜索主配置类**。搜索会从包含测试类的包开始直到找到一个使用 *@SpringBootApplication* 或者 *@SpringBootConfiguration* 的类为止。

注意如果使用 JUnit4，我们必须使用 *@RunWith(SpringRunner.class)* 来修饰这个测试类。

### Q13. SpringBoot的 Actuator 是做什么的？

本质上，Actuator 通过启用 production-ready 功能使得 SpringBoot 应用程序变得更有生命力。这些功能允许我们对生产环境中的应用程序进行监视和管理。

集成 SpringBoot Actuator 到项目中非常简单。我们需要做的只是将 *spring-boot-starter-actuator* starter 引入到 POM.xml 文件当中：

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

SpringBoot Actuaor 可以使用 HTTP 或者 JMX endpoints来浏览操作信息。大多数应用程序都是用 HTTP，作为 endpoint 的标识以及使用 */actuator* 前缀作为 URL路径。

这里有一些常用的内置 endpoints Actuator：

- *auditevents*：查看 audit 事件信息
- *env*：查看 环境变量
- *health*：查看应用程序健康信息
- *httptrace*：展示 HTTP 路径信息
- *info*：展示 *arbitrary* 应用信息
- *metrics*：展示 metrics 信息
- *loggers*：显示并修改应用程序中日志器的配置
- *mappings*：展示所有 *@RequestMapping* 路径信息
- *scheduledtasks*：展示应用程序中的定时任务信息
- *threaddump*：执行 *Thread Dump*

## 3. 总结

这篇文章我们讨论了一些关于 SpringBoot 的关键问题，这些问题可能会在你们的技术面试中遇到。我们希望这篇文章能够帮你找到理想的工作。

## 4. 译者总结

原文囊括了一些 SpringBoot 中提供的特性以及特点，译者在翻译的时候也重温了其中的一些注意点，希望可以帮到读者们，更好地认识 SpringBoot，能够从容不迫地面对面试官的问题。

在这里也提一句，如果想和我们一起工作，可以将简历投放到我们的邮箱，我们等待你们的加入！

---

 _小喇叭_ 

**广州芦苇科技Java开发团队**

芦苇科技-广州专业互联网软件服务公司

抓住每一处细节 ，创造每一个美好

关注我们的公众号，了解更多

想和我们一起奋斗吗？lagou搜索“ **芦苇科技** ”或者投放简历到 **server@talkmoney.cn**  加入我们吧

![](https://user-gold-cdn.xitu.io/2018/12/19/167c57a3e3d84dd5?w=640&h=356&f=gif&s=1273445)

关注我们，你的评论和点赞对我们最大的支持