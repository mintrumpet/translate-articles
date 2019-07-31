# SpringBoot 当中的测试集成

>原文自国外技术社区dzone，作者为 Rajesh Bhojwani，[传送门](https://dzone.com/articles/integration-testing-in-spring-boot-1n)

> 这篇文章中，我们去看如何在 SpringBoot 应用当中集成测试。

## 概述

当我们谈及 SpringBoot 应用的测试集成时，大多数都是关于通过 *ApplicationContext* 来跑应用程序和测试程序。Spring 框架中确实有一个专门的测试模块去集成测试，它被称作 *spring-test*。如果我们使用 spring-boot，我们需要使用将 *spring-test* 和其他依赖库集成到内部的 *spring-boot-starter-test*。

在这篇文章，我们看如何在 SpringBoot 应用当中集成测试。

## @SpringBootTest

*spring-boot* 提供一个基于 *spring-test* 模块集成 spring-boot 特征的 `@SpringBootTest` 注解。这个注解通过 *SpringApplication* 创建使用在测试中的 *ApplicationContext* 来运作。它启动内嵌的服务器，创建一个独立的 web 环境并且使用 *@Test* 方法去集成测试。

默认地，*@SpringBootTest* 并没有在服务器上启动。我们需要添加 *webEnvironment* 属性去进一步完善你的测试运行实例。这里有几点选项：

- MOCK（默认选项）：加载一个 web 应用上下文（ApplicationContext）并且提供一个 mock web环境
- RANDOM_PORT：加载一个 web 服务器应用上下文（WebServerApplicationContext）并且提供一个真实的 web 环境。内嵌的服务器会被启动并且监听一个随机的端口。这是其中一种被用来做集成测试的方法
- DEFINED_PORT：加载一个 web 服务器应用上下文并且提供一个真实的 web 环境
- NONE：通过使用 *SpringApplication* 加载一个应用上下文，但不提供任何的 web 环境

在 Spring 的测试框架中，为了识别哪个 spring 的 *@Configuration* 被加载，我们曾经使用 *@ContextConfiguration* 注解去处理。然而，这在 spring-boot 中并不需要因为它在未定义时已经自动去搜索主配置（primary configuration）了。

如果我们想定制主配置，我们能够使用一个名为 *@TestConfiguration* 的内嵌类来代替定义程序主配置。

## 启动程序

让我们开启一个简单的 REST API 应用程序并且看下是怎么在上面编写集成测试的。我们将创建一个 用于生成和查询的学生 API 并且在控制层上编写具体的集成测试。

### Maven 依赖

在这个例子中，我们使用 spirng-boot、spring-data、h2 数据库以及 spirng-boot test 依赖。

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
    </dependency>
</dependencies>
```

### 控制器

控制层创建 *createStudent()* 和 *retrieveStudent()* 这两个 REST API：

```java
@RestController
public class StudentController {
    @Autowired
    private StudentService studentService;
    @PostMapping("/students")
    public ResponseEntity<Void> createStudent() {
        List<Student>students  =  studentService.createStudent();
        URI location = ServletUriComponentsBuilder.fromCurrentRequest().path(
            "/{id}").buildAndExpand(students.get(0).getId()).toUri();
        return ResponseEntity.created(location).build();
    }
    @GetMapping("/students/{studentId}")
    public Student retrieveStudent(@PathVariable Integer studentId) {
        return studentService.retrieveStudent(studentId);
    }
}
```

### 服务层

这里的实现是调用数据处理去创建和查询学生记录：

```java
@Component
public class StudentService {
    @Autowired
    private StudentRepository repository;
    public List<Student> createStudent() {
        List<Student> students = new ArrayList<Student>();
        List<Student> savedStudents = new ArrayList<Student>();
        students.add(new Student("Rajesh Bhojwani", "Class 10"));
        students.add(new Student("Sumit Sharma", "Class 9"));
        students.add(new Student("Rohit Chauhan", "Class 10"));
        Iterable<Student> itrStudents=repository.saveAll(students);
        itrStudents.forEach(savedStudents::add);
        return savedStudents;
    }
    public Student retrieveStudent(Integer studentId) {
       return repository.findById(studentId).orElse(new Student());
    }
}
```

### 数据层

创建一个实现所有 CRUD 操作的 spring data Repository， *StudentRepository*。

```java
@Repository
public interface StudentRepository extends CrudRepository<Student, Integer>{
}
```

## TestRestTemplate

如上所述，为了集成 spirng-boot 应用程序的测试，我们需要使用 *@SpringBootTest*。spring-boot 也提供了其他类去做。例如 *TestRestTemplate* 去处理测试 REST API。例如 *RestTemplate*，也提供 *getForObject()、postForObject()、 exchange()* 这样的方法。让我们实现一个 *@Test* 方法去测试生成和查询方法吧。

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class StudentControllerTests {
    @LocalServerPort
    private int port;
    TestRestTemplate restTemplate = new TestRestTemplate();
    HttpHeaders headers = new HttpHeaders();
    @Test
    public void testCreateStudent() throws Exception {
        HttpEntity<String> entity = new HttpEntity<String>(null, headers);
        ResponseEntity<String> response = restTemplate.exchange(
          createURLWithPort("/students"), HttpMethod.POST, entity, String.class);
        String actual = response.getHeaders().get(HttpHeaders.LOCATION).get(0);
        assertTrue(actual.contains("/students"));
    }    
    @Test
    public void testRetrieveStudent() throws Exception {
        HttpEntity<String> entity = new HttpEntity<String>(null, headers);
        ResponseEntity<String> response = restTemplate.exchange(
          createURLWithPort("/students/1"), HttpMethod.GET, entity, String.class);
        String expected = "{\"id\":1,\"name\":\"Rajesh Bhojwani\",\"description\":\"Class 10\"}";
        JSONAssert.assertEquals(expected, response.getBody(), false);
    }
    private String createURLWithPort(String uri) {
        return "http://localhost:" + port + uri;
    }
}
```

在上面的代码中，我们使用 *WebEnvironment.RANDOM_PORT* 通过暂定的随机端口去运行。*@LocalServerPort* 帮助查询当前使用端口并且通过模板类来构建 URI 去执行。我们也使用 *exchange()*  方法去返回 *ResponseEntity* 来获得结果。

## @MockBean

通过使用 *TestRestTemplate*，我们已经测试了从控制层到数据库层的所有层面了。然而，有时候在不需要 DB 连接以及三方的服务的情况中你仍然想在 scope 中测试所有层方法。这时候就需要 mocking 所有额外的系统要求和服务。*@MockBean* 协助去 mocking 一个确定的层。在这个例子上，我们将 mocking 这个 *StudentRepository*：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class StudentControllerMockTests {
    @Autowired
    private StudentService studentService;
    @MockBean
    private StudentRepository studentRepository;
    @Test
    public void testRetrieveStudentWithMockRepository() throws Exception {
        Optional<Student> optStudent = Optional.of( new Student("Rajesh","Bhojwani"));
        when(studentRepository.findById(1)).thenReturn(optStudent);
        assertTrue(studentService.retrieveStudent(1).getName().contains("Rajesh"));
    }
}
```

我们使用带着 *@MockBean* 注解的 Mockito 方法去设置预期的返回。（译者：上述并没有 Mockito 方法，可能是笔者笔误）

## MockMvc

这里还有一个测试那些不需要启动服务器的层方法。在这个方法中，Spring 处理输入的 HTTP 请求并且交由给控制层处理。在这个方法，几乎覆盖整个过程，并且就像在处理一个真实的 HTTP 请求一样，代码也会准确地执行，而去掉了启动服务器的消耗。我们通过使用 spring 中的 *MockMvc* 來完成，并且在注入中我们使用另一个注解 *@AutoConfigureMockMvc*。让我们使用它来实现创建和查询吧：

```java

@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class StudentControllerMockmvcTests {
    @Autowired
    private MockMvc mockMvc;
    @Test
    public void testCreateRetrieveWithMockMVC() throws Exception {
        this.mockMvc.perform(post("/students")).andExpect(status().is2xxSuccessful());
        this.mockMvc.perform(get("/students/1")).andDo(print()).andExpect(status().isOk())
          .andExpect(content().string(containsString("Rajesh")));
    }
}
```

## WebTestClient

在 spring 5中，Webflux 是用于处理响应式流。在这个情况下，我们需要使用 *WebTestClient* 去处理 REST API 的测试。

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class WebTestClientExampleTests {
    @Autowired
    private WebTestClient webClient;
    @Test
    public void exampleTest() {
    this.webClient.get().uri("/students/1").exchange().expectStatus().isOk()
    .expectBody(String.class).isEqualTo("Rajesh Bhojwani");
    }
}
```

## 结论

在这篇文章中，我们认识到我们能够通过使用 Spring Boot 测试框架以及几种不同注解的支持来处理集成测试了。

像往常一样，你能在我的[Github](https://github.com/RajeshBhojwani/springboot-integrationtest)上找到这篇文章相关的源码。

## 译者总结



 _小喇叭_ 

**广州芦苇科技Java开发团队**

芦苇科技-广州专业互联网软件服务公司

抓住每一处细节 ，创造每一个美好

关注我们的公众号，了解更多

想和我们一起奋斗吗？lagou搜索“ **芦苇科技** ”或者投放简历到 **server@talkmoney.cn**  加入我们吧

![](https://user-gold-cdn.xitu.io/2018/12/19/167c57a3e3d84dd5?w=640&h=356&f=gif&s=1273445)

关注我们，你的评论和点赞对我们最大的支持