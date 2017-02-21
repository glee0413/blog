---
title: spring boot 初体验1
date: 2017-02-20 00:26:31
tags: spring
---
# spring boot web开发初体验
## 1 spring boot安装

开发环境：mint18 安装java，maven，gradle
```
sudo apt-get install openjdk-8-jdk
sudo apt-get install maven
curl -s get.sdkman.io | bash
source "$HOME/.sdkman/bin/sdkman-init.sh" #初始化sdkman
sdk install gradle #安装gradle
sdk install springboot
```
<!-- more -->
maven需要修改repo为阿里的repo，编辑/etc/maver/setting.xml，在mirrors中添加
```
<mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>        
</mirror>
```
## 2 创建POM文件
创建一个maven 的pom.xml文件，将下面的样例copy进去：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.0.BUILD-SNAPSHOT</version>
    </parent>

    <!-- Additional lines to be added here... -->

    <!-- (you don't need this if you are using a .RELEASE version) -->
    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
            <snapshots><enabled>true</enabled></snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
        </pluginRepository>
        <pluginRepository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </pluginRepository>
    </pluginRepositories>
</project>
```

执行 **mvn package** 下载测试pom文件

## 3 添加classpath依赖
Spring Boot提供了许多“启动器”，使得容易将jar添加到类路径。 示例应用程序已经在POM的父部分中使用了spring-boot-starter-parent。 spring-boot-starter-parent是一个特殊的启动器，提供了有用的Maven默认值。 它还提供了一个依赖关系管理部分，以便您可以省略依赖关系的版本标签。

其他“Starters”只是提供了开发特定类型应用程序时可能需要的依赖关系。 由于我们正在开发一个Web应用程序，我们将添加一个spring-boot-starter-web依赖关系
**mvn dependency：tree**命令打印项目依赖关系的树表示。 您可以看到，spring-boot-starter-parent本身不提供依赖性。 让我们编辑我们的pom.xml，并在parent部分下面添加spring-boot-starter-web依赖
```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```
如果再次运行mvn dependency：tree，将看到现在还有许多其他依赖项，包括Tomcat Web服务器和Spring Boot本身。
## 4 测试代码
添加**src/main/java/Example.java**文件
```
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Example.class, args);
    }

}
```
Example的**@RestController**注解，表示这个类是一个web控制器，来处理web请求
**@RequestMapping** 表示一个路由请求

**@EnableAutoConfiguration** 告诉spring boot基于你添加的包的依赖如何配置spring boot，由于 **spring-boot-starter-web** 添加了tomcat及Spring MVC，自动配置假设你在开发一个web应用
main函数是一个遵照java约定标准方法，用于应用程序的启动入口。我们的main方法通过调用run来委托Spring Boot的SpringApplication类，SpringApplication将引导我们的应用程序，启动Spring，从而启动自动配置的Tomcat Web服务器。 我们需要传递Example.class作为run方法的参数来告诉SpringApplication这是主要的Spring组件。
## 5 运行应用程序
运行**mvn spring-boot:run**,然后通过8080端口访问
## 6 创建一个可执行的jar文件
要创建可执行的jar，我们需要将spring-boot-maven-plugin添加到我们的pom.xml中。 在dependencies部分下面插入以下行：
```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
运行 **mvn package** 打包，查看target目录生成myproject-0.0.1-SNAPSHOT.jar文件，myproject-0.0.1-SNAPSHOT.jar.original文件为打包为spring boot前的文件。
运行应用程序通过下面的命令
```java -jar target/myproject-0.0.1-SNAPSHOT.jar```

参考：https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#getting-started-first-application
