---
layout: post
title: Spring Boot系列之一 起手式
date: 2016-10-13 08:08:00
categories: Spring-Boot
tag: Spring Boot
---

* content
{:toc}

简介
====================================

Spring Boot精简了基于Spring创建的项目的配置,使用javaconfig的方式去除了xml带来的混乱;而且Spring Boot提供了一系列的自动化配置
使开发人员能够很方便的集成Spring生态圈和其他工具链的整合,比如说Redis,EhCache,MongoDB;Spring Boot还提供了比如Tomcat,Jetty等
servlet容器,使其可以用`java -jar xxx.jar`的方式运行一个web应用.

搭建基础Spring Boot框架
====================================

## 环境说明
  本文采用Spring Boot 1.4.1.RELEASE , JDK1.8.0_101.

## Spring Initializr
  使用`Spring Initializr`生成Spring Boot maven/gradle 项目

  * 访问[Spring Initializr](http://start.spring.io/)
    ![spring_initializr.png](https://github.com/sunshineasbefore/resource/blob/master/spring_initializr.png?raw=true)
  * 生成Spring Boot
    * **Generate a** 选择`Maven Project`
    * **Project Metadata** 酌情填写
    * **Dependencies** 填写Web
    * 点击**Generate Project alt +** 生成使用Maven构建的Spring Boot项目
    * 浏览器会自动下载一个压缩包
  * 其他
    `Spring Initializr`中还可以选择其他依赖包,自己酌情选择即可.

## 导入到IDE
  * 将下载的压缩包解压后用IDE导入,注意需要转换为Maven Project.此时,IDE会自动下载所依赖的jar包
  * 目录结构 ![目录结构](https://github.com/sunshineasbefore/resource/blob/master/%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84.png?raw=true)
  * 目录结构说明
    * `src/main/java` java代码目录
    * `src/main/resources/application.properties` 配置文件
    * `src/main/resources/templates` 页面模板
    * `src/main/resources/static` 静态资源
    * `src/main/test` java测试代码目录

## 初始pom.xml说明
  * groupId,artifactId,version,packaging

        <groupId>cn.veryjava</groupId>
        <artifactId>web-base</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <packaging>jar</packaging>
        <name>web-base</name>
        <description>spring boot web应用基础架构</description>

  * parent

        <parent>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-parent</artifactId>
          <version>1.4.1.RELEASE</version>
          <relativePath/> <!-- lookup parent from repository -->
        </parent>

  * dependencies

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
        </dependencies>
  * maven构建工具

        <build>
          <plugins>
          <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
          </plugin>
          </plugins>
        </build>

## Hello World实例
  通过上面的介绍,现在我们对Spring Boot有了一个大体了解,那么我们来写个Hello World!

  * 创建HelloController类

        package cn.veryjava;

        import org.springframework.web.bind.annotation.RequestMapping;
        import org.springframework.web.bind.annotation.RequestMethod;
        import org.springframework.web.bind.annotation.RestController;

        /**
        * 描述: TODO:
        * 包名: cn.veryjava.
        * 作者: barton.
        * 日期: 16-10-12.
        * 项目名称: veryjava.spring.boot
        * 版本: 1.0
        * JDK: since 1.8
        */
        @RestController
        @RequestMapping
        public class HelloController {

        @RequestMapping(value = "hello", method = RequestMethod.GET)
        public String hello() {
        return "Hello World! Hello Spring Boot!";
        }
        }

  * 测试
    打开浏览器输入:`http://localhost:8080/hello`

    输出: ![Hello World! Hello Spring Boot!](https://github.com/sunshineasbefore/resource/blob/master/hellospringboot.png?raw=true)

总结
===========
到了这里,简单几步,一个简单的能够接受Http请求的Spring Boot项目就搭建完了,当然这是最简单的项目,不过我们可以将这个项目作为其他项目的脚手架来用.
