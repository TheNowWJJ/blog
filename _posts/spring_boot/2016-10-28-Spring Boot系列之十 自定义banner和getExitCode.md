---
layout: post
title: Spring Boot系列之十 自定义banner和getExitCode
date: 2016-10-28 11:12:00 +0800
categories: Spring-Boot
tag: Spring Boot
---

* content
{:toc}

简介
====
本文主要就说下在Spring Boot项目启动时控制台打印的banner图标如何自定义以及Spring Context生命周期结束时执行的`ExitCodeGenerator`

自定义banner
=====

* 链接

国外有一个专门用来生成banner的网址:[banner](http://patorjk.com/software/taag),打开这个网址,生成你想要的字儿.

![](https://github.com/sunshineasbefore/resource/blob/master/bannertxt.png?raw=true)

生成时,我们可以选择自己喜欢的字体等信息.

完成后,选择`select&copy`复制到`banner.txt`文件并将其放到`src/main/resources/`目录下,重新启动应用程序.

![](https://github.com/sunshineasbefore/resource/blob/master/bannerview.png?raw=true)

额,名字太长也是种病啊`/ch`.

ExitCodeGenerator
================
生命周期结束时,会执行`ExitCodeGenerator`接口的`getExitCode()`.这个接口也只有这一个方法.

这个接口让我们在`SpringApplication`结束时,可以做一些我们自己想做的,或者必须要去做的工作,比如说数据库链接的关闭,IO流的关闭等等.当然前提时我们需要手动调用`SpringApplication`的结束操作.

直接杀掉进程(kill -9 pid)的话是不会执行的,你想想,进程都死了,还执行个啥? kill 的级别不记得了,应该还是有能够执行的.

## 示例

* ExitCodeConfig

      package cn.veryjava;

      import org.springframework.boot.ExitCodeGenerator;
      import org.springframework.stereotype.Component;

      @Component
      public class ExitCodeConfig implements ExitCodeGenerator {
        @Override
        public int getExitCode() {
          // 自定义程序结束时返回的退出码
          System.out.println("// the application exited;");
          return 1024;
        }
      }

* BannerAndLifeApplication

    这个,我懒,这个主main类,我写了N多注解...

      package cn.veryjava;

      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.boot.SpringApplication;
      import org.springframework.boot.autoconfigure.SpringBootApplication;
      import org.springframework.context.annotation.Configuration;
      import org.springframework.context.support.ApplicationObjectSupport;
      import org.springframework.stereotype.Controller;
      import org.springframework.web.bind.annotation.RequestMapping;
      import org.springframework.web.bind.annotation.ResponseBody;

      @SpringBootApplication
      @Controller
      @Configuration
      public class BannerAndLifeApplication extends ApplicationObjectSupport {

        @Autowired
        private ExitCodeConfig exitCodeConfig;

        public static void main(String[] args) {
          SpringApplication.run(BannerAndLifeApplication.class, args);
        }

        @RequestMapping("/exit")
        @ResponseBody
        public String exit() {
          SpringApplication.exit(super.getApplicationContext(), exitCodeConfig);
          return "ok";
        }

      }

* 测试

  启动完成,打开浏览器输入`http://localhost:8080/exit`或者`curl http://localhost:8080/exit` 执行SpringApplication的关闭操作.

  查看控制台输出:`// the application exited;`

代码
=====

> [Spring Boot 自定义banner和getExitCode](https://github.com/sunshineasbefore/veryjava.spring.boot/tree/master/bannerandlife)
