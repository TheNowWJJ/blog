---
layout: post
title: Spring Boot系列之八 数据源配置整体介绍
date: 2016-10-28 11:08:00 +0800
categories: Spring-Boot
tag: Spring Boot
---

* content
{:toc}

简介
============

前面两章在介绍Spring Boot整合MyBatis的时候简单的看了一下Spring Boot如何去配置单一的数据源.并没有详细的单独去看数据源的配置.

本文就简要介绍下在Spring Boot中对内置内存数据库H2,多数据源等的配置.

由于本文档的代码基本都是基于`MyBatis`搭建，所以关于`Spring-Data-JPA`的内容并不能展示出来，Spring Boot中对于Repository/Dao的设计并不能完美的体现出来，不过，各个持久层框架自有它自己的好处，无所谓谁坏谁差，还是要看我们具体怎么去应用。

对内置内存数据库的支持
==================

在开发时使用Spring Boot的内置内存数据库是很有帮助的.它不需要持久化数据存储,仅仅用来在开发时验证我们的业务逻辑是否正确.

在讲内置内存数据库之前先讲一个小的知识点:`CommandLineRunner`, 其支持我们在应用启动的时候自动执行一些代码,以帮助我们完成内置内存数据库数据的初始化操作.

## CommandLineRunner

Spring Boot 应用程序在启动后,会自动遍历`CommandLineRunner`接口的实例并运行它的run方法.在设置`CommandLineRunner`的时候,可以使用`@Order`注解指定某个`run`方法的执行顺序.

看具体实例:

      package cn.veryjava;

      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.boot.CommandLineRunner;
      import org.springframework.jdbc.core.JdbcTemplate;
      import org.springframework.stereotype.Component;
      import org.springframework.core.annotation.Order;

      import java.io.BufferedReader;
      import java.io.IOException;
      import java.io.InputStream;
      import java.io.InputStreamReader;
      import java.util.List;

      @Component
      ＠Order(value=1)
      public class CommandLineConfig implements CommandLineRunner {

        @Autowired
        private Properties p;

        @Autowired
        private JdbcTemplate jdbcTemplate;

        @Override
        public void run(String... args) throws IOException {
            String query = "SELECT COUNT(1) FROM STUDENT";
            // ......
        }
      }

可以看出，在实现`CommandLineRunner`接口的实例时，也可以直接使用`@AutoWired`注入Spring Context中的Bean。说明`CommandLineRunner`的实例是在SpringApplication执行完成后，再运行的。

## 对内置内存数据库的支持

基于Spring Boot的自动化配置，其实我们也仅仅需要在pom.xml文件中引入我们想使用的内置内存数据库的jar依赖即可。Spring Boot会自动扫描classpath下是否有某个jar包，如果有，就加载默认配置。

### pom.xml引入依赖

基于mybatis的配置：

      <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.1.1</version>
      </dependency>
      <dependency>
        <groupId>org.hsqldb</groupId>
        <artifactId>hsqldb</artifactId>
        <scope>runtime</scope>
      </dependency>

使用内置内存数据库需要引入`spring-jdbc`，其在`mybatis-spring-boot-starter`已经包含。

实际使用过程中，因为内置的内存数据库并不能保留表结构，所以，如果使用JPA就需要在配置文件中将ddl语句设置为`spring.jpa.hibernate.ddl-auto=create`，如果使用MyBatis，那就需要使用`CommandLineRunner`来保证应用程序在启动时，能够生成表结构。

使用生产数据库
=============

### Spring Boot自动配置对数据库连接池的选择原则：

 * 优先选择使用Tomcat的数据库连接池
 * 如果存在HikariCP，则启用
 * 如果存在Commons DBCP，则启用
 * 如果存在Commons DBCP2，则启用

按照官方文档的说法，我们在引入`mybatis-spring-boot-starter`其实已经引入了`tomcat-jdbc`的依赖，也就使用了tomcat的数据库连接池。

当然我们也可以配置自己的数据库连接池，比如配置阿里爸爸的`Druid`来作为我们的数据库连接池工具

      package cn.veryjava;

      import com.alibaba.druid.pool.DruidDataSource;
      import org.apache.ibatis.session.SqlSessionFactory;
      import org.mybatis.spring.SqlSessionFactoryBean;
      import org.mybatis.spring.annotation.MapperScan;
      import org.springframework.boot.context.properties.ConfigurationProperties;
      import org.springframework.context.annotation.Bean;
      import org.springframework.context.annotation.Configuration;
      import org.springframework.core.io.support.PathMatchingResourcePatternResolver;

      import javax.sql.DataSource;

      @Configuration
      @MapperScan("com.*.*.mapper")
      public class MyBatisConfig {
        @Bean
        public SqlSessionFactory sqlSessionFactoryBean() throws Exception {

          SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
          sqlSessionFactoryBean.setDataSource(dataSource());

          PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();

          sqlSessionFactoryBean.setMapperLocations(resolver
           .getResources("classpath:/mapper/\*.xml"));
          return sqlSessionFactoryBean.getObject();
        }

        @Bean
        @ConfigurationProperties(prefix = "spring.datasource")
        public DataSource dataSource() {
          return new DruidDataSource();
        }
      }

使用JDBC
========

> 请看：[springboot-jdbc](https://github.com/sunshineasbefore/veryjava.spring.boot/tree/master/jdbc)

两个数据源的配置
===============

## 最基本的配置


      @Bean
      @Primary
      @ConfigurationProperties(prefix="datasource.primary")
      public DataSource primaryDataSource() {
          return DataSourceBuilder.create().build();
      }

      @Bean
      @ConfigurationProperties(prefix="datasource.secondary")
      public DataSource secondaryDataSource() {
          return DataSourceBuilder.create().build();
      }

## 进阶

今天多说一点就是,两个数据源分别针对不同的包下的mapper文件:

* JDBCDataSourceConfig

      package cn.veryjava.datasources.config;

      import com.alibaba.druid.pool.DruidDataSource;
      import org.mybatis.spring.annotation.MapperScan;
      import org.springframework.boot.context.properties.ConfigurationProperties;
      import org.springframework.context.annotation.Bean;
      import org.springframework.context.annotation.Configuration;
      import org.springframework.context.annotation.Primary;

      import javax.sql.DataSource;

      /**
       \* 描述: TODO:
       \* 包名: cn.veryjava.
       \* 作者: barton.
       \* 日期: 16-10-28.
       \* 项目名称: veryjava.spring.boot
       \* 版本: 1.0
       \* JDK: since 1.8
       \*/
      @Configuration
      @MapperScan(value = "cn.veryjava.datasources.jdbcmapper")
      public class JDBCDataSourceConfig {

        @Bean
        @Primary
        @ConfigurationProperties(prefix = "primary.datasource")
        public DataSource dataSource() {
          return new DruidDataSource();
        }
      }

* SSABDataSourceConfig

      package cn.veryjava.datasources.config;

      import com.alibaba.druid.pool.DruidDataSource;
      import org.mybatis.spring.annotation.MapperScan;
      import org.springframework.boot.context.properties.ConfigurationProperties;
      import org.springframework.context.annotation.Bean;
      import org.springframework.context.annotation.Configuration;

      import javax.sql.DataSource;

      /**
       \* 描述: TODO:
       \* 包名: cn.veryjava.
       \* 作者: barton.
       \* 日期: 16-10-28.
       \* 项目名称: veryjava.spring.boot
       \* 版本: 1.0
       \* JDK: since 1.8
       \*/
      @Configuration
      @MapperScan(value = "cn.veryjava.datasources.ssabmapper")
      public class SSABDataSourceConfig {

        @Bean
        @ConfigurationProperties(prefix = "secondary.datasource")
        public DataSource dataSource() {
          return new DruidDataSource();
        }
      }

对比这两个类,其不同点在于:

  * `@MapperScan`的扫描路径不一致,这个的用处主要在于用不同的包区分不同数据源对于数据的操作.
  * `@ConfigurationProperties`的`prefix`不一致,这个用来区分不同的数据源.
  * `JDBCDataSourceConfig`类中的`dataSource()`多了一个`@Primary`注解,指定某一个dataSource为主要的datasource.

### 代码

  > [Spring Boot 多数据源配置](https://github.com/sunshineasbefore/veryjava.spring.boot/tree/master/datasources)
