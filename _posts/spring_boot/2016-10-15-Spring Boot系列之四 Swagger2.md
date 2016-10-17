---
layout: post
title: Spring Boot系列之四 使用Swagger2构建RESTful API文档
date: 2016-10-15 08:28:00 +0800
categories: Spring-Boot
tag: Spring Boot
---

* content
{:toc}

简介
=======================

可能大家受够了拿个Word写API文档了,现在Swagger2可以让大家节省很多劳动力,而且展示出来的文档,风格统一,功能强大.

但是!但是!!Swagger2毕竟是得等代码写好了,最起码也得等各个接口创建好了把相应的注解整理完整后文档才会生成.所以...
有些时候,你并不能逃避Word.

当然,当然.反过来讲,对于没有文档的老代码,我们只需要增加相应的Swagger2注解就能生成一份很好看的文档咯.

废话不多说.

实例
====================

### 在pom.xml文件中引入

    <properties>
        <springfox-swagger2.version>2.6.0</springfox-swagger2.version>
      </properties>
      <!-- 添加Swagger2依赖,用于生成接口文档 -->
      <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>${springfox-swagger2.version}</version>
      </dependency>
      <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>${springfox-swagger2.version}</version>
      </dependency>

### 创建package

在`src/main/java`目录下创建一个package`swagger2`

### 创建Swagger2配置类

      package cn.veryjava;

      import org.springframework.context.annotation.Bean;
      import org.springframework.context.annotation.Configuration;
      import springfox.documentation.builders.ApiInfoBuilder;
      import springfox.documentation.builders.PathSelectors;
      import springfox.documentation.builders.RequestHandlerSelectors;
      import springfox.documentation.service.ApiInfo;
      import springfox.documentation.spi.DocumentationType;
      import springfox.documentation.spring.web.plugins.Docket;
      import springfox.documentation.swagger2.annotations.EnableSwagger2;

      @Configuration
      @EnableSwagger2
      public class Swagger2Configuration {
        private ApiInfo apiInfo() {
          return new ApiInfoBuilder()
           .title("应用接口文档")
           .description("应用接口文档 1.0版本")
           .version("1.0")
           .build();
        }

        @Bean
        public Docket createRestApi() {
          return new Docket(DocumentationType.SWAGGER_2)
           .apiInfo(apiInfo())
           .select()
           .apis(RequestHandlerSelectors.basePackage("cn.veryjava.swagger2"))
           .paths(PathSelectors.any())
           .build();
        }
      }

#### basePackage说明

  swagger2会把父级目录下的所有子目录中包含`basePackage`的目录全部扫描,也就是说,针对某一个项目创建的文档会不小心引入其他项目的接口....

  SO.老老实实的再创建一层package吧!

### SampleController

我们来创建一个例子展示Swagger2生成的文档

      package cn.veryjava.swagger2;

      import io.swagger.annotations.ApiImplicitParam;
      import io.swagger.annotations.ApiOperation;
      import org.springframework.web.bind.annotation.PathVariable;
      import org.springframework.web.bind.annotation.RequestMapping;
      import org.springframework.web.bind.annotation.RequestMethod;
      import org.springframework.web.bind.annotation.RestController;

      @RestController
      @RequestMapping("/sample")
      public class SampleController {

        @ApiOperation(value = "sample接口", notes = "sample")
        @ApiImplicitParam(value = "姓名", name = "name", required = true, dataType = "String")
        @RequestMapping(value = "/{name}", method = RequestMethod.GET)
        public String abc(@PathVariable String name) {
          return name;
        }
      }
### 效果展示

  打开浏览器输入`http://localhost:8080/swagger-ui.html`:
  ![swagger-ui.html](https://github.com/sunshineasbefore/resource/blob/master/swagger2.png?raw=true)

  可以看到,swagger2展示的文档,不仅仅只有接口地址,参数列表,Http请求状态码,还包含一个RESTful接口测试的工具.我们可以直接点击`Try it out!`得到请求结果.

总结
======

本文只是简单的介绍了一下Swagger2在Spring Boot中的应用.如果还想继续了解Swagger2的内容,可以直接去[Swagger官网](http://swagger.io/)

代码
===

[Spring Boot swagger2 api文档](https://github.com/sunshineasbefore/veryjava.spring.boot/tree/master/swagger)
