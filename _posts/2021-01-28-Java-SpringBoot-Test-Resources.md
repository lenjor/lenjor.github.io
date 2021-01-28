---
layout: post
title: SpringBoot maven 配置多环境 & Junit单元测试加载src/main/resources目录下资源文件
tags: Java  
---

- [SpringBoot项目多种环境配置文件选择](#springboot项目多种环境配置文件选择)
- [使用Maven的形式配置项目多环境](#使用maven的形式配置项目多环境)
- [单元测试使用和 `src/main/resources` 一样的配置文件](#单元测试使用和-srcmainresources-一样的配置文件)
- [参考文献](#参考文献)

# SpringBoot项目多种环境配置文件选择
（1） SpringBoot官方推荐的形式properties多环境配置，通过 `application.properties` 设置 `spring.profile.active` 的值实现，具体的实现可以参考这篇文章

[SpringBoot多环境切换: https://blog.csdn.net/liu911025/article/details/81489117](https://blog.csdn.net/liu911025/article/details/81489117)

（2）通过maven配置实现环境配置切换，上面第一种配置文件的切换有一个很不友好的地方，它切换的只是properties不同环境的配置，但是正常使用中，配置文件往往有很多份，比如下图是一个企业级项目的配置文件，这时候就要使用maven配置多环境，在编译的时候指定对应的编译环境

![](/images/posts/myBlog/2021-01-28-Java-SpringBoot-Test-Resources-01.png)

# 使用Maven的形式配置项目多环境
``` xml
 <!--配置环境-->
    <profiles>
        <profile>
            <id>prod</id>
            <properties>
                <profile.env>ae</profile.env>
            </properties>
        </profile>
        <profile>
            <id>dev</id>
            <properties>
                <profile.env>jp</profile.env>
            </properties>
        </profile>
        <profile>
            <id>test</id>
            <properties>
                <profile.env>test</profile.env>
            </properties>
            <!-- 默认环境 -->
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
    </profiles>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>

        <resources>
            <!--指定编译打包包含的resources目录下的配置文件-->
            <resource>
                <directory>src/main/resources/</directory>
                <filtering>true</filtering>
                <includes>
                    <include>*.properties</include>
                    <include>*.xml</include>
                    <include>i18n/*</include>
                </includes>
            </resource>
            <!--指定当前环境变量下的配置文件夹目录的配置文件，如果和resources目录下的文件名同名会覆盖-->
            <resource>
                <directory>src/main/resources/${profile.env}</directory>
                <!--必须要增加filtering，会扰乱执行顺序 -->
                <filtering>true</filtering>
                <includes>
                    <include>*.*</include>
                </includes>
            </resource>
        </resources>
    </build>
```
# 单元测试使用和 `src/main/resources` 一样的配置文件
没有加以下的配置的时候，`@SpringBootTest` 注解，只会加载test路径下的资源文件(即xml配置)，并不会加载main路径下的资源文件，要使用main路径下的配置文件，需要加上以下的配置，指定testResources使用的配置文件
        
``` xml    
    <!--和上面的配置一样，放到 build 节点里面 -->
        <!--单元测试时引用src/main/resources下的资源文件-->
        <testResources>
            <testResource>
                <directory>src/main/resources/</directory>
                <filtering>true</filtering>
                <includes>
                    <include>*.properties</include>
                    <include>*.xml</include>
                    <include>i18n/*</include>
                </includes>
            </testResource>
             <testResource>
                <directory>src/main/resources/${profile.env}</directory>
            </testResource>
        </testResources>
```


# 参考文献
1. [SpringBoot多环境切换:https://blog.csdn.net/liu911025/article/details/81489117](https://blog.csdn.net/liu911025/article/details/81489117)
2. [springboot Junit单元测试之坑](https://blog.csdn.net/MuErHuoXu/article/details/86750497)