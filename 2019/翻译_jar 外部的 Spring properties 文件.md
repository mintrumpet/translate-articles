# 翻译 | jar 外部的 Spring properties 文件

![mark](![mark](https://image.talkmoney.cn/luwei/20190428/agf1DIby7sc8.jpg)

> 原文自工程师baeldung博客，作者为 Andy Beck，[传送门](https://www.baeldung.com/spring-properties-file-outside-jar)

## 1. 综述

property 文件是我们用于存储项目中的特定信息的一种常用方法。理想情况下，我们应该将它放置在包的外部，以便随时能够根据需要更改配置。

## 2. 使用默认路径

按照约定，SpringBoot 会在4个预先确定的位置中查找一个外部配置文件，application.properties 或者 application.yaml，并且会按以下优先顺序排列：

- 当前目录的 /config 子目录
- 当前目录
- 包类路径 /config 目录
- 类路径根目录

因此，**定义在 application.proerties 并且放置在当前目录中的 /config 子目录中的配置信息会被加载**。在发生冲突时，还将覆盖其他位置的属性。

## 3. 使用命令行

如果我们不想实行上述的约定，我们也可以**直接在命令行中配置路径**：

```
java -jar app.jar --spring.config.location=file:///Users/home/config/jdbc.properties
```

我们也可以通过配置一个文件夹的位置来查找配置文件：

```
java -jar app.jar --spring.config.name=application,jdbc --spring.config.location=file:///Users/home/config
```

并且，另一种方法是通过 Maven 插件来启动 SpringBoot 程序。这里我们可以使用 -D 参数指明：

```
mvn spring-boot:run -Dspring.config.location="file:///Users/home/jdbc.properties"
```

## 4. 使用环境变量

或者，我们并不能改变启动命令。但最棒的是 **SpringBoot 还会读取  SPRING_CONFIG_NAME 和 SPRING_CONFIG_LOCATION 这两个环境变量**：

```
export SPRING_CONFIG_NAME=application,jdbc
export SPRING_CONFIG_LOCATION=file:///Users/home/config
java -jar app.jar
```

需要注意的是，默认路径文件仍然会被加载。但是在发生属性冲突的时候，**特定环境的配置文件将会优先被加载**。

## 5. 编码方式执行

或者，如果希望通过编码方式来处理，我们可以通过注册一个 *PropertySourcesPlaceholderConfigurer* bean 来处理：

```java
public PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
    PropertySourcesPlaceholderConfigurer properties = new PropertySourcesPlaceholderConfigurer();
    properties.setLocation(new FileSystemResource("/Users/home/conf.properties"));
    properties.setIgnoreResourceNotFound(false);
    return properties;
}
```

这里，我们使用 *PropertySourcesPlaceholderConfigurer* 来从自定义的位置加载配置属性。

## 6. 从 FatJar 中排除文件

Maven Boot 插件会自动将会把在 *src/main/resources* 目录下的所有文件加载到 jar 包中。如果我们不想让一个文件成为 jar 的一部分，那么我们可以通过一个简单的配置来移除掉：

``` xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
            <excludes>
                <exclude>**/conf.properties</exclude>
            </excludes>
        </resource>
    </resources>
</build>
```

在这个例子中，我们从包含在结果 jar 中的 *conf.properties* 文件排除在外了。

## 7. 结论

正如我们所看到的，SpringBoot 框架本身会帮我们处理外部的配置文件。

通常来说，我们只需要将属性值放置在正确的文件和位置上即可，但是我们也可以使用 Spring 中的 Java API 来做更灵活的处理。

和往常一样，在我们的 [Github](https://github.com/eugenp/tutorials/tree/master/spring-boot-ops) 上可以得到示例中的完整源代码。