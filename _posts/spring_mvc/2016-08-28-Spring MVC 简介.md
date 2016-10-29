---
layout: post
title: Spring MVC 简介
date: 2016-08-28 01:08:00 +0800
categories: Spring-MVC
tag: Spring MVC
keywords: Spring Boot,Spring MVC,web开发
---

* content
{:toc}

本文主要介绍Spring MVC.

MVC概述
====================================

* MVC 是 Model-View-Control 的简称，即模型-视图-控制器。它是一个存在于服务器表达层的模型，它将应用分开，改变应用之间的高度耦合。
* 视图
    * 数据的展现。视图是用户看到并与之交互的界面。视图向用户显示相关的数据，并能接收用户的输入数据，但是它并不进行任何实际的业务处理。视图可以向模型查询业务状态，但不能改变模型。视图还能接受模型发出的数据更新事件，从而对用户界面进行同步更新。
* 模型
    * 应用对象。模型是应用程序的主体部分。 模型代表了业务数据和业务逻辑； 当数据发生改变时，它要负责通知视图部分；一个模型能为多个视图提供数据。由于同一个模型可以被多个视图重用，所以提高了应用的可重用性。
*  控制器
    *  逻辑处理、控制实体数据在视图上展示、调用模型处理业务请求。当 Web 用户单击 Web 页面中的提交按钮来发送 HTML 表单时，控制器接收请求并调用相应的模型组件去处理请求，然后调用相应的视图来显示模型返回的数据。
* MVC 模型运行机制
在 MVC 模式中，Web 用户向服务器提交的所有请求都由控制器接管。接受到请求之后，控制器负责决定应该调用哪个模型来进行处理；然后模型根据用户请求进行相应的业务逻辑处理，并返回数据；最后控制器调用相应的视图来格式化模型返回的数据，并通过视图呈现给用户。

![MVC 模型运行机制](http://dl.iteye.com/upload/attachment/576353/d3d769c0-5df9-3c31-9984-42f1fcbd00bb.jpg)

三层架构概述
====================================

* Presentation tier + Application tier + Data tier (展现层 + 应用层 + 数据访问层)
* 实际上MVC只存在三层架构的展现层,M实际商是数据模型,是包含数据的对象.
* Service和Dao层反馈在应用层和数据访问层

Spring MVC 介绍
====================================

* Spring Web MVC处理请求的流程
  具体执行步骤如下：
  *  首先用户发送请求----->前端控制器，前端控制器根据请求信息（如URL）来决定选择哪一个页面控制器进行处理并把请求委托给它，即以前的控制器的控制逻辑部分；图2-1中的1、2步骤；

  *  页面控制器接收到请求后，进行功能处理，首先需要收集和绑定请求参数到一个对象，这个对象在Spring Web MVC中叫命令对象，并进行验证，然后将命令对象委托给业务对象进行处理；处理完毕后返回一个ModelAndView（模型数据和逻辑视图名）；图2-1中的3、4、5步骤；

  *  前端控制器收回控制权，然后根据返回的逻辑视图名，选择相应的视图进行渲染，并把模型数据传入以便视图渲染；图2-1中的步骤6、7；

  *  前端控制器再次收回控制权，将响应返回给用户，图2-1中的步骤8；至此整个结束
![Spring Web MVC处理请求的流程](http://sishuok.com/forum/upload/2012/7/14/529024df9d2b0d1e62d8054a86d866c9__1.JPG)

Spring MVC的优势
====================================

  * 清晰的角色划分：前端控制器（DispatcherServlet）、请求到处理器映射（HandlerMapping）、处理器适配器（HandlerAdapter）、视图解析器（ViewResolver）、处理器或页面控制器（Controller）、验证器（   Validator）、命令对象（Command  请求参数绑定到的对象就叫命令对象）、表单对象（Form Object 提供给表单展示和提交到的对象就叫表单对象）。

  * 分工明确，而且扩展点相当灵活，可以很容易扩展，虽然几乎不需要；

  * 由于命令对象就是一个POJO，无需继承框架特定API，可以使用命令对象直接作为业务对象；

  * 和Spring 其他框架无缝集成，是其它Web框架所不具备的；

  * 可适配，通过HandlerAdapter可以支持任意的类作为处理器；

  * 可定制性，HandlerMapping、ViewResolver等能够非常简单的定制；

  * 功能强大的数据验证、格式化、绑定机制；

  * 利用Spring提供的Mock对象能够非常简单的进行Web层单元测试；

  * 本地化、主题的解析的支持，使我们更容易进行国际化和主题的切换。

  * 强大的JSP标签库，使JSP编写更容易。

  * ………………还有比如RESTful风格的支持、简单的文件上传、约定大于配置的契约式编程支持、基于注解的零配置支持等等。

Spring MVC的常用注解
====================================

* @ Controller 表明这个类是Spring MVC里的Controller.Dispatcher Servlet 会自动扫描注解了此注解的类.在声明普通Bean的时候,使用@Component,@Service,@Repository和@Controller是等同的,因为@Service,@Repository,@Controller都组合了@Component元注解.但在Spring MVC声明控制器Bean的时候,只能使用@Controller.
* @RequestMapping 用来映射Web请求(访问路径和参数),处理类和方法.其支持Servlet的request和response作为参数.
* @ResponseBody 支持将返回值放在response体内,而不是返回一个页面,此注解可放置在返回值前或者方法上.
* @RequestBody 允许request的参数在request体中,而不是直接链接在地址后面.此注解放置在参数前.
* @PathVariable 用来接收路径参数,此注解放置在参数前.
* @RestController 这是一个组合注解,组合了@Controller和@ResponseBody
* **延伸阅读:
  > [什么是request,response](http://blog.csdn.net/jcx5083761/article/details/9340209)**

Spring MVC 基本配置
====================================
  Spring MVC的定制配置需要我们的配置类集成一个WebMvcConfigurerAdapter类,并在此类使用@EnableWebMvc注解,来开启Spring MVC的配置支持.

Spring MVC 静态资源配置
====================================
  [Spring Boot默认的静态资源配置](http://blog.csdn.net/isea533/article/details/50412212)
  如果需要直接访问静态资源,可以在我们的配置类中重写 addResourceHandlers方法

  * 快捷的ViewController
    无需做任何业务处理,只是简单的页面转向,可以使用addViewControllers方法来实现.

        package com.wangge.buzmgt.config;
        import com.wangge.json.JSONFormatMethodProcessor;
        import org.springframework.context.annotation.Bean;
        import org.springframework.context.annotation.Configuration;
        import org.springframework.http.converter.ByteArrayHttpMessageConverter;
        import org.springframework.http.converter.HttpMessageConverter;
        import org.springframework.http.converter.ResourceHttpMessageConverter;
        import org.springframework.http.converter.StringHttpMessageConverter;
        import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;
        import org.springframework.http.converter.support.AllEncompassingFormHttpMessageConverter;
        import org.springframework.http.converter.xml.Jaxb2RootElementHttpMessageConverter;
        import org.springframework.http.converter.xml.SourceHttpMessageConverter;
        import org.springframework.web.method.support.HandlerMethodReturnValueHandler;
        import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
        import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
        import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
        import java.util.ArrayList;
        import java.util.List;

        @Configuration
        @EnableWebMvc
        public class WebMvcConfig extends WebMvcConfigurerAdapter {

          @Override
          public void addViewControllers (ViewControllerRegistry registry) {
            registry.addViewController ("/").setViewName ("index");
            registry.addViewController ("/left").setViewName ("left");
          }

          @Override
          public void addReturnValueHandlers (List<HandlerMethodReturnValueHandler> returnValueHandlers) {
            returnValueHandlers.add (new JSONFormatMethodProcessor (messageConverter ()));
          }

          private List<HttpMessageConverter<?>> messageConverter () {
            List<HttpMessageConverter<?>> converters = new ArrayList<> ();
            converters.add (new ByteArrayHttpMessageConverter ());
            converters.add (new StringHttpMessageConverter ());
            converters.add (new ResourceHttpMessageConverter ());
            converters.add (new SourceHttpMessageConverter<> ());
            converters.add (new AllEncompassingFormHttpMessageConverter ());
            converters.add (new Jaxb2RootElementHttpMessageConverter ());
            converters.add (new MappingJackson2HttpMessageConverter ());
            return converters;
          }

          @Override
          public void addResourceHandlers (ResourceHandlerRegistry registry) {
            registry.addResourceHandler ("/static/**").addResourceLocations ("classpath:/static/");
          }
        }

    其中 addResourceLocations 指的是文件放置的目录,addResourceHandler指的是对外暴露的访问路径.
