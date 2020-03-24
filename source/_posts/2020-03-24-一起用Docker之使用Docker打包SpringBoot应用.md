---
title: 一起用Docker之使用Docker打包SpringBoot应用
date: 2020-03-24 21:52:41
categories: 
- 技海拾贝
- 一起用Docker
tags:
- Docker
- Nexus
- 私服
---

之前使用Docker搭建了开发环境的相关服务，同时也搭建开发与测试环境中的依赖服务，再同时也使用Nexus搭建了Docker的私服，那么这次咱们一起来用Docker部署SpringBoot应用。一步一步的向全部Docker化进军。

<!-- more -->

# 搭建SpringBoot工程

工程比较简单，不做数据交互，也没有复杂的业务，仅有一句打印语句而已。

使用任何你所熟悉的方法，创建一个Maven工程，修改pom.xml文件，增加SpringBoot的依赖：

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.2.4.RELEASE</version>
        </parent>
        <groupId>net.mooko.demos</groupId>
        <artifactId>docker-spring-boot</artifactId>
        <version>0.1.0</version>

        <properties>
            <java.version>1.8</java.version>
        </properties>

        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
        </dependencies>

        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>
    </project>

下面是Application文件的内容，包括Controller一同定义:

    package net.mooko.demos.dockerspringboot;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;

    @SpringBootApplication
    @RestController
    public class Application {
        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }

        @GetMapping("/")
        public String home() {
            return "Hello docker world";
        }
    }

此时执行应用，通过浏览器访问`http://localhost:8080/`，可以看到屏幕上会打印一句：

    Hello docker world


# 使用Docker打包镜像

增加Maven的插件，通过Maven的命令来打包Docker的镜像，当前使用比较多的，有三个插件：

- spotify/docker-maven-plugin：早期的一个Maven插件，可以没有DockerFile，所有的参数都配置在pom.xml中，后来作者觉得这样不好，又新写了一个，也是咱们选择的这个`dockerfile-maven-plugin`；
- fabric8io/docker-maven-plugin：功能超级强大，不过使用起来，也有些麻烦，有兴趣的可以试试；
- spotify/dockerfile-maven-plugin：spotify/docker-maven-plugin的新版本，使用比较方便，自己编写DockerFile，之后可以自动打包、发布。

## 在项目根目录增加`Dockerfile`文件：

    FROM openjdk:8-jdk-alpine
    COPY target/*.jar app.jar
    ENTRYPOINT ["java", "-jar", "/app.jar"]
    EXPOSE 8080

内容比较简单：

- 使用OpenJdk做基础镜像
- 将Target下的jar包复制为app.jar
- 执行命令
- 暴露端口

## 修改pom.xml配置文件的`plugins`部分：

    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
            <version>1.4.13</version>
            <executions>
                <execution>
                    <id>default</id>
                    <goals>
                        <goal>build</goal>
                        <goal>push</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <username>puras</username>
                <password>123456</password>
                <repository>10.211.55.8:6000/${project.artifactId}</repository>
                <tag>${project.version}</tag>
                <buildArgs>
                    <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
                </buildArgs>
            </configuration>
        </plugin>
    </plugins>

引入新的插件进来，具体的配置可以参考`https://github.com/spotify/dockerfile-maven`，其中：

- username和password是你的Docker私服的账号和密码，如果你的私服是允许任何人上传，则可以忽略
- repository则是你私服的地址前缀，加上你的Docker镜像的名称
- tag如果不指定，则默认是latest

增加完上述的插件后，再次执行打包命令时，就可以自动生成Docker的镜像了，如果不想要这个效果，可以将`goal`注释掉：

    mvn clean package

根据官方的说法，有了这个插件，就不需要一步一步的执行了，直接执行`deploy`就可以一键发布了，但我这里有些问题，一键不行，就多键吧：

    mvn clean package dockerfile:push

这样就可以将我们写的SpringBoot应用，发布到私服库里了。从别的环境或是自己重新Pull一下此镜像，运行后访问`http://localhost:8080`也能看到下面的内容了：

    Hello docker world

# 推荐理由

- 使用统一的环境，打包好后可以在任何的Docker中执行
- 测试人员不再需要安装JDK等这些依赖，拿到镜像可以直接进行部署，降低因为环境而导致的问题
- 结合Jenkins进行自动部署，使自动构建更方便

***

一点小小的想法，希望能对大家有用。

Over~~~
