---
layout: post
title: Spring Boot系列之五 使用JDBC访问数据库
date: 2016-10-15 08:48:00 +0800
categories: Spring-Boot
tag: Spring Boot
---

* content
{:toc}

简介
=====

关于Spring Boot基础框架搭建和Spring Boot的Web开发已经大体学习完了,现在我们要开始学习怎么去访问数据库.

本文仅用于学习目的,没有使用ORM框架,因此省略了数据库表的列和Java Bean的映射.

> 离了数据,么系统都没有意义.

实例
===

### pom.xml文件中引入

      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
      </dependency>

      <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.21</version>
      </dependency>

#### spring-boot-configuration-processor

      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-configuration-processor</artifactId>
          <optional>true</optional>
      </dependency>

这个jar包可以让你通过在java类上使用`@ConfigurationProperties`注解,读取自定义的配置项.当然该Java类是一个简单的Java Bean, 需要有getter/setter方法.

### 配置文件

  application.properties

      server.port=8080

      jdbc.db-path=classpath:db.sql  # 初始化表的sql文件

      spring.datasource.url=jdbc:mysql://localhost:3306/jdbc
      spring.datasource.username=root
      spring.datasource.password=root
      spring.datasource.driver-class-name=com.mysql.jdbc.Driver

### db.sql

  该文件位于`src/main/resources`目录下.

      CREATE TABLE student (
      id   BIGINT      NOT NULL PRIMARY KEY,
      name VARCHAR(32) NOT NULL,
      age  INT         NOT NULL
      );

### Properties

      package cn.veryjava;

      import org.springframework.boot.context.properties.ConfigurationProperties;
      import org.springframework.core.io.Resource;
      import org.springframework.stereotype.Component;

      /**
       * 描述: TODO:
       * 包名: cn.veryjava.
       * 作者: barton.
       * 日期: 16-10-13.
       * 项目名称: veryjava.spring.boot
       * 版本: 1.0
       * JDK: since 1.8
       */
      @Component
      @ConfigurationProperties(prefix = "jdbc")
      public class Properties {
        private int port;

        private Resource dbPath;

        public int getPort() {
          return port;
        }

        public void setPort(int port) {
          this.port = port;
        }

        public Resource getDbPath() {
          return dbPath;
        }

        public void setDbPath(Resource dbPath) {
          this.dbPath = dbPath;
        }
      }

### CommandLineConfig

  该类用于在Spring Boot项目启动时执行一些操作.

      package cn.veryjava;

      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.boot.CommandLineRunner;
      import org.springframework.jdbc.core.JdbcTemplate;
      import org.springframework.stereotype.Component;

      import java.io.BufferedReader;
      import java.io.IOException;
      import java.io.InputStream;
      import java.io.InputStreamReader;
      import java.util.List;

      @Component
      public class CommandLineConfig implements CommandLineRunner {

        @Autowired
        private Properties p;

        @Autowired
        private JdbcTemplate jdbcTemplate;

        @Override
        public void run(String... args) throws IOException {
          try {
            String query = "SELECT COUNT(1) FROM STUDENT";
            List count = jdbcTemplate.queryForList(query);
            if (count.size() == 0) {
              String sql = getStreamString(p.getDbPath().getInputStream());
              jdbcTemplate.update(sql);
            }
          } catch (Exception e) {
            String sql = getStreamString(p.getDbPath().getInputStream());
            jdbcTemplate.update(sql);
          }
        }

        // 将一个输入流转化为字符串
        public static String getStreamString(InputStream tInputStream) {
          if (tInputStream != null) {
            try {
              BufferedReader tBufferedReader = new BufferedReader(new InputStreamReader(tInputStream));
              StringBuffer tStringBuffer = new StringBuffer();
              String sTempOneLine = new String("");
              while ((sTempOneLine = tBufferedReader.readLine()) != null) {
                tStringBuffer.append(sTempOneLine);
              }
              return tStringBuffer.toString();
            } catch (Exception ex) {
              ex.printStackTrace();
            }
          }
          return null;
        }
      }

**注意下 本文中如何读取db.sql**

### StudentService

      package cn.veryjava;

      import java.util.List;

      /**
       * 描述: TODO:
       * 包名: cn.veryjava.
       * 作者: barton.
       * 日期: 16-10-13.
       * 项目名称: veryjava.spring.boot
       * 版本: 1.0
       * JDK: since 1.8
       */
      public interface StudentService {

        /**
         * 查询
         */
        List select();

        /**
         * 新增
         */
        void add();

        /**
         * 更新
         */
        void update();

        /**
         * 删除
         */
        void delete();

      }

### StudentServiceImpl

      package cn.veryjava;

      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.jdbc.core.JdbcTemplate;
      import org.springframework.stereotype.Service;

      import java.util.List;

      /**
       * 描述: TODO:
       * 包名: cn.veryjava.
       * 作者: barton.
       * 日期: 16-10-13.
       * 项目名称: veryjava.spring.boot
       * 版本: 1.0
       * JDK: since 1.8
       */
      @Service
      public class StudentServiceImpl implements StudentService {

        @Autowired
        private JdbcTemplate jdbc;

        @Override
        public List select() {
          return jdbc.queryForList("SELECT * FROM STUDENT");
        }

        @Override
        public void add() {
          jdbc.update("INSERT INTO STUDENT(ID,NAME,AGE)VALUES(4,'NAME4',14)");
        }

        @Override
        public void update() {
          jdbc.update("UPDATE STUDENT SET NAME='NAME44' WHERE ID=4");
        }

        @Override
        public void delete() {
          jdbc.update("DELETE FROM STUDENT WHERE ID=1");
        }
      }

### 测试

* BaseTest

      package cn.veryjava;

      import com.fasterxml.jackson.databind.ObjectMapper;
      import org.junit.Before;
      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.http.MediaType;
      import org.springframework.mock.web.MockHttpServletResponse;
      import org.springframework.test.web.servlet.MockMvc;
      import org.springframework.test.web.servlet.ResultActions;
      import org.springframework.test.web.servlet.setup.MockMvcBuilders;
      import org.springframework.web.context.WebApplicationContext;

      import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;

      public class BaseTest {
        @Autowired
        private WebApplicationContext context;

        private MockMvc mvc;

        @Before
        public void setUp() {
          this.setMvc(MockMvcBuilders.webAppContextSetup(this.context).build());
        }

        public MockHttpServletResponse doGet(String url) throws Exception {
          ResultActions ra = this.getMvc().perform(get(url));
          return ra.andReturn().getResponse();

        }

        public MockHttpServletResponse doPost(String url, Object params) throws Exception {
          ObjectMapper om = new ObjectMapper();
          ResultActions ra = this.getMvc()
           .perform(post(url).content(om.writeValueAsString(params)).contentType(MediaType
            .APPLICATION_JSON));
          return ra.andReturn().getResponse();
        }

        public MockMvc getMvc() {
          return mvc;
        }

        public void setMvc(MockMvc mvc) {
          this.mvc = mvc;
        }

      }

* StudentServiceTest

      package cn.veryjava;

      import org.junit.Test;
      import org.junit.runner.RunWith;
      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.boot.test.context.SpringBootTest;
      import org.springframework.test.context.junit4.SpringRunner;

      import static org.junit.Assert.*;

      /**
       * 描述: TODO:
       * 包名: cn.veryjava.
       * 作者: barton.
       * 日期: 16-10-13.
       * 项目名称: veryjava.spring.boot
       * 版本: 1.0
       * JDK: since 1.8
       */
      @RunWith(SpringRunner.class)
      @SpringBootTest(classes = JdbcApplication.class)
      public class StudentServiceTest {

        @Autowired
        private StudentService service;

        @Test
        public void select() throws Exception {
          service.select().forEach(student -> System.out.println(student));
        }

        @Test
        public void add() throws Exception {
          service.add();
          select();
        }

        @Test
        public void update() throws Exception {
          service.update();
          select();
        }

        @Test
        public void delete() throws Exception {
          service.delete();
          select();
        }

      }

总结
====

从上面的例子中可以看出,Spring Boot在使用JDBC连接数据库的操作上非常简单.我们只需要引入相应的`start pom`文件,配置下数据库的连接属性,然后就可以直接在代码中使用`@Autowired private JdbcTemplate template`来进行数据库的操作.从而避免了使用xml配置时大量的xml文件产生.

代码
===

[Spring Boot使用JDBC连接数据库](https://github.com/sunshineasbefore/veryjava.spring.boot/tree/master/jdbc)
