---
layout: post
title: Spring Boot系列之六 整合MyBatis 注解
date: 2016-10-15 08:48:00 +0800
categories: Spring-Boot
tag: Spring Boot
---

* content
{:toc}

简介
====

### 选择Mybatis的原因

* 本人技术历史背景原因,使用MyBatis更多.
* 在某个阶段使用Spring-Data-JPA的过程中踩到了很多很多坑,比如说各种级联产生的json序列化问题.
* 本人更倾向于更自由理解起来更简单的MyBatis...(脑子不好使,没办法.)

当然Spring Boot更倾向于使用JPA.因为Spring Boot的宗旨是简单,而JPA也正好符合这一宗旨.

### 推荐

  [Java Persistence with MyBatis 3(中文版).pdf](http://download.csdn.net/detail/qq457557442/9654697)

实例
====

### pom.xml

      <dependency>
         <groupId>mysql</groupId>
         <artifactId>mysql-connector-java</artifactId>
         <scope>runtime</scope>
       </dependency>
      <dependency>
         <groupId>org.mybatis.spring.boot</groupId>
         <artifactId>mybatis-spring-boot-starter</artifactId>
         <version>1.1.1</version>
       </dependency>

### application.properties

      server.port=8080
      spring.datasource.url=jdbc:mysql://localhost:3306/jdbc
      spring.datasource.username=root
      spring.datasource.password=root
      spring.datasource.driver-class-name=com.mysql.jdbc.Driver

### user.sql

      CREATE TABLE USER (
        ID   BIGINT      NOT NULL PRIMARY KEY AUTO_INCREMENT,
        NAME VARCHAR(32) NOT NULL,
        AGE  INT         NOT NULL
      );

      INSERT INTO USER (NAME, AGE) VALUES ('admin', 12);
      INSERT INTO USER (NAME, AGE) VALUES ('admin2', 13);

### 实体映射 User

      package cn.veryjava;

      public class User {
        private long id;
        private String name;
        private int age;

        public long getId() {
          return id;
        }

        public void setId(long id) {
          this.id = id;
        }

        public String getName() {
          return name;
        }

        public void setName(String name) {
          this.name = name;
        }

        public int getAge() {
          return age;
        }

        public void setAge(int age) {
          this.age = age;
        }
      }

### Mapper

      package cn.veryjava;

      import org.apache.ibatis.annotations.Insert;
      import org.apache.ibatis.annotations.Mapper;
      import org.apache.ibatis.annotations.Param;
      import org.apache.ibatis.annotations.Select;

      @Mapper
      public interface UserMapper {
        @Select("SELECT * FROM USER WHERE NAME = #{name}")
        User findByName(@Param("name") String name);

        @Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name}, #{age})")
        int insert(@Param("name") String name, @Param("age") int age);
      }

测试
====

### MybatisApplicationTests

      package cn.veryjava;

      import org.junit.Assert;
      import org.junit.Test;
      import org.junit.runner.RunWith;
      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.boot.test.context.SpringBootTest;
      import org.springframework.test.annotation.Rollback;
      import org.springframework.test.context.junit4.SpringRunner;

      @RunWith(SpringRunner.class)
      @SpringBootTest(classes = MyBatisApplication.class)
      public class MybatisApplicationTests {

        @Autowired
        private UserMapper userMapper;

        @Test
        @Rollback
        public void contextLoads() {
          userMapper.insert("AAA", 20);
          User u = userMapper.findByName("AAA");
          Assert.assertEquals(20, u.getAge());
        }

      }

总结
====

这种方式我使用的很少,没法确切的说这种做法有什么好处,什么不好.
从个人角度来看,这种做法,混淆了sql和java代码,虽然有一定的混乱,但好像更加清晰了,没有影响可读性.
