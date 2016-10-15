---
layout: post
title: Spring Boot系列之三 RESTful风格API
date: 2016-10-15 08:18:00 +0800
categories: Spring-Boot
tag: Spring Boot
---

* content
{:toc}

简介
====================================

在使用Spring Boot构建RESTful风格API时,需要先了解什么是RESTful以及Spring中控制层的一些常用注解.

### 参考文章

  > [RESTful架构理解](http://blog.csdn.net/wang_jingj/article/details/51208746)

  > [RESTful架构实践](http://blog.csdn.net/wang_jingj/article/details/51208848)

  > [Spring MVC简介以及常用注解](http://blog.csdn.net/wang_jingj/article/details/52005731)

实例
====================================

### API接口设计

  * GET `/students` # 获得students列表
  * POST `/students` # 新增一个student
  * PUT `/students/1` # 更新一个student
  * DELETE `/students/1` # 删除一个student

### Student

      package cn.veryjava;

      /**
       * 描述: TODO:
       * 包名: cn.veryjava.
       * 作者: barton.
       * 日期: 16-10-12.
       * 项目名称: veryjava.spring.boot
       * 版本: 1.0
       * JDK: since 1.8
       */
      public class Student {

        private long id;
        private int age;
        private String name;

        public Student() {}

        public Student(long id, int age, String name) {
          this.id = id;
          this.age = age;
          this.name = name;
        }

        public long getId() {
          return id;
        }

        public void setId(long id) {
          this.id = id;
        }

        public int getAge() {
          return age;
        }

        public void setAge(int age) {
          this.age = age;
        }

        public String getName() {
          return name;
        }

        public void setName(String name) {
          this.name = name;
        }

        @Override
        public String toString() {
          return "Student{" +
           "id=" + id +
           ", age=" + age +
           ", name='" + name + '\'' +
           '}';
        }
      }


### StudentController

      package cn.veryjava;

      import org.springframework.util.StringUtils;
      import org.springframework.web.bind.annotation.*;

      import java.io.IOException;
      import java.util.ArrayList;
      import java.util.List;

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
      @RequestMapping(value = "students")
      public class StudentController {
      static {
      System.out.println("我是静态代码块!");
      }

      /**
      * 查询
      */
      @RequestMapping(method = RequestMethod.GET)
      public List<Student> get() {
      return getStudent();
      }

      /**
      * 增加
      */
      @RequestMapping(method = RequestMethod.POST)
      public Student add(@RequestBody Student student) {
      List<Student> students = getStudent();

      System.out.println("新增前students的size:" + students.size());

      students.add(student);

      System.out.println("新增后students的size:" + students.size());

      return student;
      }

      /**
      * 更新
      */
      @RequestMapping(value = "/{id}", method = RequestMethod.PUT)
      public String update(@PathVariable long id, @RequestParam String name, @RequestParam int age)
      throws IOException {
      List<Student> students = getStudent();

      System.out.println("更新前:");
      students.forEach(student -> System.out.println(student.getName()));

      students.forEach(student -> {
        if (id == student.getId()) {
          if (!StringUtils.isEmpty(name)) {
            student.setName(name);
          }
          if (0 != age) {
            student.setAge(age);
          }
        }
      });

      System.out.println("更新后:");
      students.forEach(student -> System.out.println(student.getName()));

      return "success";
      }

      /**
      * 删除
      */
      @RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
      public String delete(@PathVariable long id) {
      List<Student> students = getStudent();

      System.out.println("删除前students的size :" + students.size());

      for (Student student : students) {
        if (id == student.getId()) {
          students.remove(student);
          break;
        }
      }

      System.out.println("删除后students的size :" + students.size());

      return "success";
      }

      private List<Student> getStudent() {
      List<Student> students = new ArrayList<>();

      Student s1 = new Student(1l, 12, "s1");
      Student s2 = new Student(2l, 13, "s2");
      Student s3 = new Student(3l, 14, "s3");

      students.add(s1);
      students.add(s2);
      students.add(s3);

      return students;
      }
      }

测试
====================================

### BaseTest

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

        public MockHttpServletResponse doPatch(String url, Object params) throws Exception {
          ObjectMapper om = new ObjectMapper();
          ResultActions ra = this.getMvc()
           .perform(patch(url).content(om.writeValueAsString(params)).contentType(MediaType
            .APPLICATION_JSON));
          return ra.andReturn().getResponse();
        }

        public MockHttpServletResponse doPut(String url, Object params) throws Exception {
          ObjectMapper om = new ObjectMapper();

          ResultActions ra = this.getMvc().perform(put(url, om.writeValueAsString(params))
           .contentType(MediaType.APPLICATION_JSON));

          return ra.andReturn().getResponse();
        }

        public MockHttpServletResponse doDelete(String url, Object params) throws Exception {
          ObjectMapper om = new ObjectMapper();

          ResultActions ra = this.getMvc().perform(delete(url, om.writeValueAsString(params))
           .contentType(MediaType.APPLICATION_JSON));

          return ra.andReturn().getResponse();
        }

        public MockMvc getMvc() {
          return mvc;
        }

        public void setMvc(MockMvc mvc) {
          this.mvc = mvc;
        }

      }

### StudentControllerTest

      package cn.veryjava;

      import com.fasterxml.jackson.databind.ObjectMapper;
      import org.junit.Test;
      import org.junit.runner.RunWith;
      import org.springframework.boot.test.context.SpringBootTest;
      import org.springframework.mock.web.MockServletContext;
      import org.springframework.test.context.junit4.SpringRunner;
      import org.springframework.test.web.servlet.MockMvc;
      import org.springframework.test.web.servlet.RequestBuilder;
      import org.springframework.test.web.servlet.request.MockHttpServletRequestBuilder;
      import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
      import org.springframework.test.web.servlet.setup.MockMvcBuilders;

      import java.util.HashMap;
      import java.util.List;
      import java.util.Map;

      import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.delete;
      import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.put;

      /**
       * 描述: TODO:
       * 包名: cn.veryjava.
       * 作者: barton.
       * 日期: 16-10-12.
       * 项目名称: veryjava.spring.boot
       * 版本: 1.0
       * JDK: since 1.8
       */
      @RunWith(SpringRunner.class)
      @SpringBootTest(classes = {RestfulApplication.class, MockServletContext.class})
      public class StudentControllerTest extends BaseTest {

        @Test
        public void testGet() throws Exception {
          String response = doGet("/students").getContentAsString();
          ObjectMapper om = new ObjectMapper();
          List list = om.readValue(response, List.class);

          list.forEach(l -> System.out.println(((Map) l).get("id")));
        }

        @Test
        public void testAdd() throws Exception {
          Student student = new Student(4l, 15, "s5");
          String response = doPost("/students", student).getContentAsString();
          System.out.println(response);
        }

        @Test
        public void testUpdate() throws Exception {

          MockMvc mvc = MockMvcBuilders.standaloneSetup(new StudentController()).build();
          RequestBuilder e = put("/students/1").param("name", "update s1 's name").param
           ("age", "24");
          String response = mvc.perform(e).andReturn().getResponse().getContentAsString();
          System.out.println(response);
        }

        @Test
        public void testDelete() throws Exception {
          MockMvc mvc = MockMvcBuilders.standaloneSetup(new StudentController()).build();
          RequestBuilder e = delete("/students/1");
          String response = mvc.perform(e).andReturn().getResponse().getContentAsString();
          System.out.println(response);
        }
      }

### 说明

* 使用`PUT`时,接收参数不要使用`@RequestBody`,`Spring MVC`对`PUT`的支持仅限于类似`GET`的方式,即参数要紧接在`URL`后以`http://www.veryjava.cn/sss?name=xxx&age=12`的形式表示
* 不过可以使用特定的过滤器来实现接收form表单的请求:

  > [SpringMVC控制器接收不了PUT提交的参数的解决方案](https://my.oschina.net/buwei/blog/191942)
* 如果感觉使用RESTful实在困难,那还是使用`GET,POST`吧... 选择适合自己的,才是最好的.
