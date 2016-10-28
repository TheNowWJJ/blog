---
layout: post
title: Spring Boot系列之九 配置和Profiles(一)
date: 2016-10-28 11:09:00 +0800
categories: Spring-Boot
tag: Spring Boot
---

* content
{:toc}

简介
====

前面几章简单介绍了Spring Boot的基本使用,利用前面几章的内容已经可以进行简单应用的开发了.不过,我们有必要去更深入的了解Spring Boot的特性和内容,以帮助我们解决那些看起来比较困难的问题.

本章介绍一下Spring Boot的配置内容和Profiles的使用.

配置
====

Spring Boot允许使用配置文件来管理我们项目的配置.将某些配置写在配置文件的好处咱就不说了.

在Spring Boot中,我们可以使用`.properties`和`.yml`文件,也可以使用`环境变量`和`命令行参数`来管理我们的配置.

## 配置优先级以及覆盖

Spring Boot通过`PropertySource`来管理各种配置,其优先级如下:

* 命令行参数
* java:comp/env的JNDI属性
* java系统属性(System.getProperties())
* 操作系统环境变量
* random.\*产生的`RandomValuePropertySource` (在配置文件中生成随机数)
* 在打包的jar外的应用程序配置文件（application.properties，包含YAML和profile变量）
* 在打包的jar内的应用程序配置文件（application.properties，包含YAML和profile变量）
* 在@Configuration类上的@PropertySource注解
* 默认属性（使用SpringApplication.setDefaultProperties指定)

从上到下,优先级依次降低,并且优先级低的会被优先级高的覆盖掉.也就是说,如果我们在使用命令行参数时制定了一个原本在application.properties文件中制定的参数,那么该值则会使用命令行参数时制定的值.
以此类推.

## `.properties`和`.yml`文件

在使用这两种配置文件管理我们的配置时,我们可以直接在代码中使用`@Value`注解来注入我们的配置,也可以通过`@ConfigurationProperties`注解,将我们的配置直接绑定到一个Java Class中,
通过`@AutoWired`注入这个Java Class来使用.

### 配置文件

application.yml

      veryjava:
        name: 阳光如初.
        age: 88
        object: 404

### 使用`@Value`

      package cn.veryjava;

      import org.junit.Test;
      import org.junit.runner.RunWith;
      import org.springframework.beans.factory.annotation.Value;
      import org.springframework.boot.test.context.SpringBootTest;
      import org.springframework.test.context.junit4.SpringRunner;

      import static org.junit.Assert.\*;

      @RunWith(SpringRunner.class)
      @SpringBootTest(classes = SettingsApplication.class)
      public class VeryJavaPropertiesTest {

        @Value("${veryjava.name}")
        private String name;

        @Value("${veryjava.object}")
        private String object;

        @Value("${veryjava.age}")
        private String age;

        @Test
        public void testProperties1(){
          System.out.println(name);
          System.out.println(object);
          System.out.println(age);
        }
      }

### 使用`@ConfigurationProperties`

* VeryJavaProperties

      package cn.veryjava;

      import org.springframework.boot.context.properties.ConfigurationProperties;
      import org.springframework.stereotype.Component;

      @Component
      @ConfigurationProperties(prefix = "veryjava")
      public class VeryJavaProperties {
        private String name;
        private String age;
        private String object;

        public String getName() {
          return name;
        }

        public void setName(String name) {
          this.name = name;
        }

        public String getAge() {
          return age;
        }

        public void setAge(String age) {
          this.age = age;
        }

        public String getObject() {
          return object;
        }

        public void setObject(String object) {
          this.object = object;
        }
      }

* 测试

      @Autowired
      private VeryJavaProperties properties;

      @Test
      public void testProperties2(){
        System.out.println(properties.getName());
        System.out.println(properties.getAge());
        System.out.println(properties.getObject());
      }

### 使用`命令行参数`

  先来创建一个Controller

      package cn.veryjava;

      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.boot.SpringApplication;
      import org.springframework.boot.autoconfigure.SpringBootApplication;
      import org.springframework.stereotype.Controller;
      import org.springframework.web.bind.annotation.RequestMapping;
      import org.springframework.web.bind.annotation.ResponseBody;

      @SpringBootApplication
      @Controller
      public class SettingsApplication {

        @Autowired
        private VeryJavaProperties properties;

        public static void main(String[] args) {
          SpringApplication.run(SettingsApplication.class, args);
        }

        @RequestMapping("/settings")
        @ResponseBody
        public String settings(){
          return properties.getName();
        }
      }

* 默认情况

    在默认情况下`curl http://localhost:8080/settings`或者直接在浏览器调用`http://localhost:8080/settings` 会返回`阳光如初.`.

* 使用`命令行参数`输出

    好吧,现在来测一下使用`命令行参数`指定`veryjava.name`会得到什么样的结果.

    使用`mvn package -Dmaven.test.skip` 打成jar包后,再使用`java -jar settings-0.0.1-SNAPSHOT.jar --veryjava.name=sunshineasbefore.`运行该项目.

    再次调用`curl http://localhost:8080/settings`或者直接在浏览器调用`http://localhost:8080/settings`,返回`sunshineasbefore.`

    这个地方充分说明了Spring Boot关于配置项的优先级覆盖问题.

* 例外

    问:如果我们不想使用`命令行参数`来替代配置文件中的值怎么办?

    答:在主main()中利用流式API,再追加一个`SpringApplication.setAddCommandLineProperties(false)` 就over.

## 进阶

基本的配置项说完了,再来看看Spring Boot支持的其他高级配置特性.

### 配置随机值

`RandomValuePropertySource` 可以让我们在配置文件中栉jie风沐雨产生随机值.这些随机值包含整数,long或字符串类型

几种生成随机数的例子:

my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}

* 配置项

      veryjava:
        name: 阳光如初.
        age: 88
        object: 404
        my:
          secret: ${random.value}
          number: ${random.int}
          bignumber: ${random.long}
          uuid: ${random.uuid}
          lessThanTen: ${random.int(10)}
          inRange: ${random.int[1024,65536]}

* 新建一个My类

  这个类写my下的一些个属性,对应配置文件

      package cn.veryjava;

      public class My {
        private String secret;
        private String number;
        private String bignumber;
        private String uuid;
        private String lessThanTen;
        private String inRange;

        public String getSecret() {
          return secret;
        }

        public void setSecret(String secret) {
          this.secret = secret;
        }

        public String getNumber() {
          return number;
        }

        public void setNumber(String number) {
          this.number = number;
        }

        public String getBignumber() {
          return bignumber;
        }

        public void setBignumber(String bignumber) {
          this.bignumber = bignumber;
        }

        public String getUuid() {
          return uuid;
        }

        public void setUuid(String uuid) {
          this.uuid = uuid;
        }

        public String getLessThanTen() {
          return lessThanTen;
        }

        public void setLessThanTen(String lessThanTen) {
          this.lessThanTen = lessThanTen;
        }

        public String getInRange() {
          return inRange;
        }

        public void setInRange(String inRange) {
          this.inRange = inRange;
        }
      }

* 修改VeryJavaProperties类

  新增 `private My my;` 字段和其getter/setter方法.

* 测试

      @Test
      public void testRandom(){
          System.out.println(properties.getMy().getSecret());
          System.out.println(properties.getMy().getNumber());
          System.out.println(properties.getMy().getBignumber());
          System.out.println(properties.getMy().getUuid());
          System.out.println(properties.getMy().getLessThanTen());
          System.out.println(properties.getMy().getInRange());
      }
### 加载application属性文件

  默认情况下 Spring Boot加载`src/main/resources/application.properties`文件,当然我们可以通过命令行指定配置文件的位置或名称.

* 指定名称

      java -jar myproject.jar --spring.config.name=myproject
* 指定位置

      java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties

### 属性占位符/引用其他属性值

  当然必须引用先前定义的,而不是之后定义的.

        app.name=MyApp
        app.description=${app.name} is a Spring Boot application

  Over.

### 类型安全的配置属性

我们来重新详细介绍一下`@ConfigurationProperties`注解的使用.

Java是一个强类型安全的语言.每一个变量必须有它特定的类型,这样也保证了Java语言本身的安全性.

类型安全的配置属性,也就是说给配置文件里的每一个配置项都加上类型,比如String.

几个重点:

* 在使用`@ConfigurationProperties`注解时我们可以用`prefix`来指定要加载的配置项的前缀.
* 配置项的名称和类字段的对应关系:

  eg:

  `private String firstName;`字段

  配置文件配置项:

  * person.firstName
  * person.first-name
  * PERSON_FIRST_NAME

* 属性值校验

  使用`SR-303 javax.validation约束注解`

  eg:

  使用`@NotNull`注解

  * 修改`VeryJavaProperties`

        @NotNull
        private String position;
        // getter/setter
  * 结果

    随便运行一个先前写好的测试方法,好吧,直接报错.

代码

> [Spring Boot properties](https://github.com/sunshineasbefore/veryjava.spring.boot/tree/master/settings)
