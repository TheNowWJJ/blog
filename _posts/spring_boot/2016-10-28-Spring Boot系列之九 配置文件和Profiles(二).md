---
layout: post
title: Spring Boot系列之九 配置和Profiles(二)
date: 2016-10-28 11:10:00 +0800
categories: Spring-Boot
tag: Spring Boot
---

* content
{:toc}

说明
====
写到这里突然发现很多地方都需要`--spring.profiles.active=dev`这个命令行参数.请大家别忘咯加- -!

And 这两个项目我们使用`application.yml`配置文件.因其功能比`application.properties`更加强大.下面会介绍滴.

End 其实Spring Boot的配置文件可以通过`Spring-Cloud-Config`项目来进行管理.这个以后再说吧.

Profiles
====
`@Profile`注解可以用在`@Component`和`@Configuration`注解之上,用来隔离应用程序配置,使这些配置只能在特定的环境下生效.说白了,就是用了区分生产环境,开发环境,测试环境等等环境的一个配置项.

## `@Profile`注解和Multi-profile YAML文档

在一个yml文件中定义多个profile的变量.

* VeryJavaProperties

      package cn.veryjava;

      import org.springframework.boot.context.properties.ConfigurationProperties;
      import org.springframework.context.annotation.Profile;
      import org.springframework.stereotype.Component;

      @Component
      @Profile("dev")
      @ConfigurationProperties(value = "veryjava")
      public class VeryJavaProperties {
        private String name;

        public String getName() {
          return name;
        }

        public void setName(String name) {
          this.name = name;
        }
      }

  注意`@Profile("dev")`.

* 修改配置文件application.yml如下:

      spring:
        profiles: prod
      veryjava:
        name: 阳光如初.
      ---
      spring:
        profiles: dev
      veryjava:
        name: 阳光如初 in dev.

  注意其中的`---` ,区分profile. 不写直接报错!

* 运行

  * dev

    设置IDE启动该项目的命令行参数为`spring.profiles.active=dev`或者直接用`java -jar xxx.jar --spring.profiles.active=dev`来启动.

    调用`curl http://localhost:8080/settings`或者直接浏览器打开`http://localhost:8080/settings`输出结果`阳光如初 in dev.`

  * prod

    好吧,虽然我们的配置文件里边写了两个`veryjava.name`属性,一个属于`prod`环境,另一个属于`dev`环境.但是,但是! 我们在`VeryJavaProperties`类上写了一个注解`@Profile("dev")`并指定这个类在`spring.profiles.active=dev`时才存在.所以如果这个时候我们在测试代码或者其他代码中使用`@AutoWired`注解注入`VeryJavaProperties`是不对的.

    怎么让它顺利运行呢?我们只要修改`VeryJavaProperties`的`@Profile("dev")`为`@Profile({"dev","prod"})`就可以了.

## 总结

  这个例子中说明了两个问题,其一是`@Profile("dev")`这个注解指定profile环境,其二是`application.yml`配置文件指定了多个profile下的相同参数的不同值.

代码
===
> [Spring Boot profiles](https://github.com/sunshineasbefore/veryjava.spring.boot/tree/master/profiles)
