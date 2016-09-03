---
layout: post_layout
title: Spring Boot的Web开发
time: 2016年08月28日 星期六
location: 济南
pulished: true
excerpt_separator: "#"
---

## Spring Boot的Web开发支持
  Spring Boot提供了spring-boot-starter-web为Web开发予以支持.它为我们提供了嵌入的Tomcat以及Spring MVC的依赖.

## Thymeleaf模板引擎

  Spring Boot 推荐使用Thymeleaf作为模板引擎.因为其提供了完整的Spring MVC支持.
  因为使用嵌入的Servlet容器来运行JSP的话有一些小问题,内嵌Tomcat,Jetty不支持以jar的形式运行JSP,而且Undertow不支持JSP.

* Thymeleaf基础知识
  Thymeleaf是一个java类库,它是一个xml/xhtml/html5的模板引擎,可以作为MVC的Web应用的View层.

  > [Thymeleaf基础知识](http://www.cnblogs.com/dreamfree/p/4158557.html?utm_source=tuicool)
* 补充:
  在javascript中访问model

  ```javascript
  <script th:inline="javascript">
  var single=[[${singlePerson}]];
  console.log(single.name + "/" + single.age);
  </script>
  ```

## Spring Boot的Thymeleaf支持
  Spring Boot通过自动配置功能对Thymeleaf进行了自动配置,因此可以直接使用.

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" lang="en">
<head>
    <meta charset="UTF-8"/>
    <meta http-equiv="X-UA-COMPATIBLE" content="IE=edge"/>
    <meta name="viewport" content="width=device-width, initial-scale=1"/>
    <title>Thymeleaf</title>
    <link th:src="@{bootstrap/css/bootstrap.min.css}" rel="stylesheet"/>
    <link th:src="@{bootstrap/css/bootstrap-theme.min.css}" rel="stylesheet"/>
    <script th:src="@{js/jquery-2.2.3.js}" type="text/javascript"/>
    <script th:src="@{bootstrap/js/bootstrap.min.js}"/>
    <script>
        $(function () {
            console.log(123);
        });
    </script>
</head>
<body>
<div class="panel panel-primary">
    <div class="panel-heading">
        <h3 class="panel-title"> 访问model</h3>
    </div>
    <div class="panel-body">
        <span th:text="${singlePerson.name}"></span>
    </div>
</div>

<div th:if="${not #lists.isEmpty(people)}">
    <div class="panel panel-primary">
        <div class="panel-heading">
            <h3 class="panel-title">列表</h3>
        </div>
        <div class="panel-body">
            <ul class="list-group">
                <li class="list-group-item" th:each="person:${people}">
                    <span th:text="${person.name}"></span>
                    <span th:text="${person.age}"></span>
                    <button class="btn" th:onclick="'getName(\''+${person.name}+'\');'">获得名字
                    </button>
                </li>
            </ul>
        </div>
    </div>
</div>
<script th:inline="javascript">
    var single = [[${singlePerson}]];
    console.log(single.name + "/" + single.age);
    function getName(name) {
        console.log(name);
    }
</script>
</body>
</html>
```

```java
package person.learn.thymeleaf;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.ArrayList;
import java.util.List;

/**
 * 1.使用Thymeleaf作为模板引擎,要在application.yml里设置spring.thymeleaf.caching设置为false
 * 2.用了模板引擎之后,原先对于jsp的设置无效
 * 3.(idea编辑器需要手动点击CTRL+F9,手动编译,因为IDEA文件不需要手动保存)修改Thymeleaf模板的内容后,要不重启项目就生效的话,需要make一下,
 * 或者使用热部署:http://mamicode.com/info-detail-1346413.html
 * 4.要使用Thymeleaf模板 请将pom.xml中相应的jar依赖注释去掉
 * Created by barton on 16-5-19.
 */
@Controller
public class ThymeleafController {

    @RequestMapping("/thymeleaf")
    public String index(Model model) {
        Person single = new Person("aa", 11);

        List<Person> people = new ArrayList<>();

        Person p1 = new Person("xx", 11);
        Person p2 = new Person("yy", 22);
        Person p3 = new Person("zz", 33);

        people.add(p1);
        people.add(p2);
        people.add(p3);

        model.addAttribute("singlePerson", single);
        model.addAttribute("people", people);

        return "thymeleaf/thymeleaf";
    }
}

```

```java
package person.learn.thymeleaf;

/**
 * Created by barton on 16-5-19.
 */
public class Person {

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    private String name;
    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}

```

## Spring Boot自动配置的静态资源

* 类路径文件
  把类路径下的/static,/public,/resources和/META-INF/resources文件夹下的静态文件直接映射为/**,可以通过http://localhost:8080/**来访问.
* webjar
  webjar就是将我们常用的脚本框架封装在jar包中的jar包.
  把webjar的/META-INF/resources/webjars/下的静态文件映射为/webjar/**,可以通过http://localhost:8080/webjar/** 来访问

## Spring Boot 对静态首页的支持

* classpath:/META-INF/resources/index.html
* classpath:/resources/index.html
* classpath:/static/index.html
* classpath:/public/index.html

## 将Tomcat替换为Jetty

```xml
<dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
          <exclusions>
              <exclusion>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-tomcat</artifactId>
              </exclusion>
          </exclusions>
      </dependency>

      <!-- 使用jetty替代tomcat,如果是使用jsp作为模板，则不能使用内嵌的jetty容器。 -->
      <!-- 内嵌的jetty容器不支持jsp模板 -->
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-jetty</artifactId>
      </dependency>
```

## 将Tomcat替换为Undertow

```xml
<dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
         <exclusions>
             <exclusion>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-tomcat</artifactId>
             </exclusion>
         </exclusions>
     </dependency>
<dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-undertow</artifactId>
  </dependency>
```
## 设置Favicon
	只需将自己的favicon.ico 防止在类路径根目录,类路径META-INF/resources/下,类路径resources/下,类路径static/下或者类路径public/下.



