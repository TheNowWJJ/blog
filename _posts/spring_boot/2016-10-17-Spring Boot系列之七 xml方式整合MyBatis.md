---
layout: post
title: Spring Boot系列之七 整合MyBatis xml配置
date: 2016-10-17 08:58:00 +0800
categories: Spring-Boot
tag: Spring Boot
---

* content
{:toc}

简介
===

可能很多人更熟悉使用xml的方式来使用MyBatis.Spring Boot 也提供了用xml来描述sql的方式.

实例
===

### 目录结构

* src/main/resources/mapper 用来存放xxMapper.xml文件

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
      <!-- druid 数据库连接池-->
      <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.0.15</version>
      </dependency>

### application.properties

      server.port=8080
      spring.datasource.url=jdbc:mysql://localhost:3306/jdbc
      spring.datasource.username=root
      spring.datasource.password=root
      spring.datasource.driver-class-name=com.mysql.jdbc.Driver

### MyBatisConfig

  这个类是MyBatis的配置类,其中

* `@MapperScan`用来配置Spring Boot自动配置功能如何扫描XXMapper.java
* `sqlSessionFactoryBean.setMapperLocations(resolver.getResources("classpath:/mapper/*.xml"));` 用来配置Spring Boot自动配置功能如何扫描xxMapper.xml.

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
         .getResources("classpath:/mapper/*.xml"));
        return sqlSessionFactoryBean.getObject();
      }

      @Bean
      @ConfigurationProperties(prefix = "spring.datasource")
      public DataSource dataSource() {
        return new DruidDataSource();
      }
      }

### User
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

### UserMapper

      package cn.veryjava;

      import org.apache.ibatis.annotations.Mapper;
      import org.apache.ibatis.annotations.Param;

      @Mapper
      public interface UserMapper {
        User findByName(@Param("name") String name);

        int insert(@Param("name") String name, @Param("age") int age);
      }

### userMapper.xml

      <?xml version="1.0" encoding="UTF-8" ?>
      <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
      <mapper namespace="cn.veryjava.UserMapper">
        <!--根据id查询用户详情-->
        <select id="findByName" parameterType="string" resultType="cn.veryjava.User">
          SELECT * FROM user WHERE name=#{name}
        </select>

        <insert id="insert" parameterType="string">
          INSERT INTO USER(NAME,AGE) VALUES(#{name},${age});
        </insert>
      </mapper>

测试
===

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
        public void testInsert() {
          userMapper.insert("AAA", 20);
          User u = userMapper.findByName("AAA");
          Assert.assertEquals(20, u.getAge());
        }


        @Test
        public void testFindByName() {
          User user = userMapper.findByName("admin");

          Assert.assertNotEquals(null, user);
        }
      }

代码
===

[Spring Boot xml方式整合MyBatis](https://github.com/sunshineasbefore/veryjava.spring.boot/tree/master/mybatis-xml)
