---
layout: post_layout
title: spring-data-jpa 中文文档(1)
time: 2016年08月28日 星期六
location: 济南
pulished: true
excerpt_separator: "#"
---

## 简介

* 为了让Spring Data的版本保持一致,可以使用maven提供的`dependencyManagement`

  ```xml
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-releasetrain</artifactId>
        <version>${release-train}</version>
        <scope>import</scope>
        <type>pom</type>
      </dependency>
    </dependencies>
  </dependencyManagement>
  ```

* Spring Boot依赖管理

* Spring Boot 会选择一个较新的版本,但是假使你想升级到一个更新的版本,你可以只配置`spring-data-releasetrain.version`属性为下列属性值中的一个.

    > `BUILD-SNAPSHOT - current snapshots`
    > `M1, M2 etc. - milestones`
    > `RC1, RC2 etc. - release candidates`
    > `RELEASE - GA release`
    > `SR1, SR2 etc. - service releases`

## 开始使用`Spring Data Repositories`.

* 核心概念 核心接口是`Repository`.它以`domain`和`domain的id类型`作为参数进行管理 . `CrudRepository` 接口提供了CRUD功能.

  ```java
  public interface Repository<T, ID extends Serializable> {

  }

  public interface CrudRepository<T, ID extends Serializable>
    extends Repository<T, ID> {

    <S extends T> S save(S entity);

    T findOne(ID primaryKey);

    Iterable<T> findAll();

    Long count();

    void delete(T entity);

    boolean exists(ID primaryKey);

    // … more functionality omitted.
  }
  ```

  Spring Data  也提供持久化的具体抽象接口 比如说`JpaRepository` 和`MongoRepository` 这些接口扩展`CrudRepository` 并暴露出底层的持久化技术,但是`CrudRepository`等类似的比较通用的持久性与具体技术无关(没有直接的实现)的接口并不包含在内.其只提供要实现的方法.

  接着`CrudRepository`有一个`PagingAndSortingRepository`的抽象接口.其有一些分页相关的功能.

  ```java
  public interface PagingAndSortingRepository<T, ID extends Serializable>
    extends CrudRepository<T, ID> {

    Iterable<T> findAll(Sort sort);

    Page<T> findAll(Pageable pageable);
  }
  ```

  获取一个每页20条第二页的`User` 信息,你可以只是简单的如此做:

  ```java
  PagingAndSortingRepository<User, Long> repository = // … get access to a bean
  Page<User> users = repository.findAll(new PageRequest(1, 20));
  ```

  除查询的方法,查询数量和删除的语句也可以用这样的方式实现.

  ```java
  public interface UserRepository extends CrudRepository<User, Long> {

    Long countByLastname(String lastname);
  }
  ```

  ```java
  public interface UserRepository extends CrudRepository<User, Long> {

    Long deleteByLastname(String lastname);

    List<User> removeByLastname(String lastname);

  }
  ```

* 查询方法(Query Methods)

  * 标准的CRUD功能仓库实现的查询比较底层.用Spring Data,定义这些查询变成了四步:
    * 定义一实现了`Repository`接口或者它的子接口的接口,并且它将会绑定输入`domain类`和`domain类的ID类型`.

      ```java
      interface PersonRepository extends Repository<Person, Long> {
          List<Person> findByLastname(String lastname);
      }
      ```

    * 定义查询的方法

      ```java
      interface PersonRepository extends Repository<Person, Long> {
        List<Person> findByLastname(String lastname);
      }
      ```

    * 让`spring`为这些接口创建代理实例

      ```java
      import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

      @EnableJpaRepositories
      class Config {}
      ```

    * 获得`repository`实例并且使用

      ```java
      public class SomeClient {

        @Autowired
        private PersonRepository repository;

        public void doSomething() {
          List<Person> persons = repository.findByLastname("Matthews");
        }
      }
      ```

    接下来详细解释每一步:

  * 定义repository接口
    第一部中你定义一个特定`domain`类型的repository接口.这个接口你必须继承`Repository`接口并且定义`domain类`和`ID类型`.如果你想暴露CRUD方法,你可以继承`CrudRepository`.

    * 让Repository定义的更有规则
      通常,你的repository接口会继承`Repository`,`CrudRepository`或者`PagingAndSortingRepository`.如果你不想继承`Spring Data interfaces` 你也可以用`@RepositoryDefinition`自己定义repository接口.继承`CrudRepository` 暴露了完整的管理你的实体的方法,如果你更喜欢自己定义哪些方法需要去暴露,只需要把要暴露的方法从`CrudRepository`中复制出来就可以了.

      ```java
      @NoRepositoryBean
      interface MyBaseRepository<T, ID extends Serializable> extends Repository<T, ID> {

        T findOne(ID id);

        T save(T entity);
      }

      interface UserRepository extends MyBaseRepository<User, Long> {
        User findByEmailAddress(EmailAddress emailAddress);
      }
      ```

      **你需要确保你自己定义的repository接口有@NoRepositoryBean注解.这样可以保证Spring Data可以实例化它**

    * 利用`multiple Spring Data modules`来使用`Repositories`
      使用应用程序中的一个独特的`Spring Data Module` 让事情变得很简单.因此在定义范围内的所有repository接口都会绑定到 `Spring Data Module` .有时候,应用程序需要多个`Spring Data Module`,这种情况下,它需要用持久化技术来区分不同的repository.`Spring Data Module`进入`strict repository mode` ,因为它检测到在类路径上有多个资源库的工厂.`strict repository mode`要求在repository或者domain的细节来决定一个`repository`定义的`Spring Data module`绑定:

      * 如果`repository`定义 继承`the module-specific repository`
      * 如果domain被`the module-specific type annotation`注解.例如JPA's `@Entity`,或者说Spring Data MongoDb/Spring Data Elasticsearch的`@Document`.

        ```java
        interface MyRepository extends JpaRepository<User, Long> { }

        @NoRepositoryBean
        interface MyBaseRepository<T, ID extends Serializable> extends JpaRepository<T, ID> {
          …
        }

        interface UserRepository extends MyBaseRepository<User, Long> {
          …
        }

        ```
          MyRepository 和UserRepository 继承了JpaRepository 在他们的类型结构中,这是有效的.

        ```java
        interface AmbiguousRepository extends Repository<User, Long> {
        …
        }

        @NoRepositoryBean
        interface MyBaseRepository<T, ID extends Serializable> extends CrudRepository<T, ID> {
          …
        }

        interface AmbiguousUserRepository extends MyBaseRepository<User, Long> {
          …
        }
        ```
        AmbiguousRepository 和AmbiguousUserRepository 分别继承Repository 和CrudRepository在他们的类型结构中,虽然这也是非常好的,`multiple modules `不能区分哪一个特定的`Spring Data`是repositories要绑定的.

        ```java
        interface PersonRepository extends Repository<Person, Long> {
         …
        }

        @Entity
        public class Person {
          …
        }

        interface UserRepository extends Repository<User, Long> {
         …
        }

        @Document
        public class User {
          …
        }
        ```
        domain类被第三方(如@Entity或者@Document)注解注解了的.

* 定义查询方法

SpringData通过方法名有两种方式去解析出用户的查询意图:一种是直接通过方法的命名规则去解析,第二种是通过Query去解析,那么当同时存在几种方式时,SpringData怎么去选择这两
种方式呢?好了,SpringData有一个策略去决定到底使用哪种方式:

  * 查询策略
    接下来我们将介绍策略的信息,你可以通过配置`<repository>`的`query-lookup-strategy`属性来决定。或者通过Java config的` Enable${store}Repositories`注解的`queryLookupStrategy`属性来指定:
    * CREATE 通过解析方法名字来创建查询。这个策略是删除方法中固定的前缀,然后再来解析其余的部分。
    * USE_DECLARED_QUERY 它会根据已经定义好的语句去查询,如果找不到,则会抛出异常信息。这个语句可以在某个注解或者方法上定义。根据给定的规范来查找可用选项,如果在方法被调用时没有找到定义的查
    询,那么会抛出异常。
    * CREATE_IF_NOT_FOUND 这个策略结合了以上两个策略。他会优先查询是否有定义好的查询语句,如果没有,就根据方法的名字去构建查询。这是一个默认策略,如果不特别指定其他策略,那么这个策略会在项目
    中沿用。

  * 构建查询
    查询构造器是内置在SpringData中的,他是非常强大的,这个构造器会从方法名中剔除掉类似find...By, read...By, 或者get...By的前缀,然后开始解析其余的名字。你可以在方法名中加入更多的表达式,例如你需要Distinct的约束,那么你可以在方法名中加入Distinct即可。在方法中,第一个By表示着查询语句的开始,你也可以用And或者Or来关联多个条件。

    ```java
    public interface PersonRepository extends Repository<User, Long> {
    List<Person> findByEmailAddressAndLastname(EmailAddress emailAddress, String lastname);
    // 需要在语句中使用Distinct 关键字,你需要做的是如下
    List<Person> findDistinctPeopleByLastnameOrFirstname(String lastname, String firstname);
    List<Person> findPeopleDistinctByLastnameOrFirstname(String lastname, String firstname);
    // 如果你需要忽略大小写,那么你要用IgnoreCase 关键字,你需要做的是如下
    List<Person> findByLastnameIgnoreCase(String lastname);
    // 所有属性都忽略大小写呢?AllIgnoreCase 可以帮到您
    List<Person> findByLastnameAndFirstnameAllIgnoreCase(String lastname, String firstname);
    // 同样的,如果需要排序的话,那你需要:OrderBy
    List<Person> findByLastnameOrderByFirstnameAsc(String lastname);
    List<Person> findByLastnameOrderByFirstnameDesc(String lastname);
    }
    ```
  根据方法名解析的查询结果跟数据库是相关,但是,还有几个问题需要注意:
  多个属性的查询可以通过连接操作来完成,例如`And,Or`。当然还有其他的,例如`Between,LessThan,GreaterThan,Like`。这些操作时跟数据库相关的,当然你还需要看看相关的数据库文档是否支持这些操作。
  你可以使用`IngoreCase`来忽略被标记的属性的大小写,也可以使用`AllIgnoreCase`来忽略全部的属性,当然这个也是需要数据库支持才允许的。
  你可以使用`OrderBy`来进行排序查询,排序的方向是Asc跟Desc,如果需要动态排序,请看后面的章节。

  * 属性表达式
    具体的方法名解析查询需要怎样的规则呢?这种方法名查询只能用在被管理的实体类上,就好像之前的案例。假设一个类`Person`中有个`Address`,并且`Address`还有`ZipCode`,那么根据ZipCode来查询这个`Person`需要怎么做呢?

    ```java
    List<Person> findByAddressZipCode(ZipCode zipCode);
    ```

    在上面的例子中,我们用`x.address.zipCode`去检索属性,这种解析算法会在方法名中先找出实体属性的完整部分(`AddressZipCode`),检查这部分是不是实体类的属性,如果解析成功,则按
    照驼峰式从右到左去解析属性,如:`AddressZipCode`将分为`AddressZip`跟`Code`,在这个时候,我们的属性解析不出Code属性,则会在此用同样的方式切割,分为`Address`跟`ZipCode`(如果
    第一次分割不能匹配,解析器会向左移动分割点),并继续解析。
    为了避免这种解析的问题,你可以用`“_”`去区分,如下所示:

    ```java
    List<Person> findByAddress_ZipCode(ZipCode zipCode);
    ```
  * 特殊参数处理
    上面的例子已经展示了绑定简单的参数,那么除此之外,我们还可以绑定一些指定的参数,如`Pageable`和`Sort`来动态的添加分页、排序查询。

    ```java
    Page<User> findByLastname(String lastname, Pageable pageable);
    List<User> findByLastname(String lastname, Sort sort);
    List<User> findByLastname(String lastname, Pageable pageable);
    ```

    第一个方法通过传递`org.springframework.data.domain.Pageable`来实现分页功能,排序也绑定在里面。如果需要排序功能,那么需要添加参数`org.springframework.data.domain.Sort`,如第二行中,返回的对象可以是`List`,当然也可以是`Page`类型的。
  * 限制查询结果
    查询结果可以通过`first`或者`top`来进行限制,`first`或者`top`是可以替换的.可以用一个数字追加在`top/first`后边以指定返回结果的条数.如果这个数字在`first/top`左边,则返回结果大小为`1`.

    ```java
    User findFirstByOrderByLastnameAsc();

    User findTopByOrderByAgeDesc();

    Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);

    Slice<User> findTop3ByLastname(String lastname, Pageable pageable);

    List<User> findFirst10ByLastname(String lastname, Sort sort);

    List<User> findTop10ByLastname(String lastname, Pageable pageable);
    ```

    表达的限制条件也可以使用`Distinct`关键字

  * 将查询结果放入到`Stream`中
    查询结果可以用Java 8 `Stream<T>` 作为返回结果类型进行处理.

    ```java
    @Query("select u from User u")
    Stream<User> findAllByCustomQueryAndStream();

    Stream<User> readAllByFirstnameNotNull();

    @Query("select u from User u")
    Stream<User> streamAllPaged(Pageable pageable);
    ```

    但并不是所有的 Spring Data modules都能正确的支持`Stream<T>`作为返回结果类型.

  * 异步查询结果
    Repository查询可以通过Spring的异步处理方法 异步执行.这意味着查询方法会立即返回.真是的查询会被作为一个task放入到`Spring TaskExecutor`中.

    ```java
    @Async
    Future<User> findByFirstname(String firstname);

    @Async
    CompletableFuture<User> findOneByFirstname(String firstname);

    @Async
    ListenableFuture<User> findOneByLastname(String lastname);
    Use java.util.concurrent.Future as return type.
    Use a Java 8 java.util.concurrent.CompletableFuture as return type.
    Use a org.springframework.util.concurrent.ListenableFuture as return type.
    ```
  * 创建Repository实体
    创建已定义的Repository接口,最简单的方式就是使用Spring配置文件,当然,需要JPA的命名空间。
  * XML配置
    你可以使用JPA命名空间里面的repositories去自动检索路径下的repositories元素:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans:beans xmlns:beans="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns="http://www.springframework.org/schema/data/jpa"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/data/jpa
        http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

      <repositories base-package="com.acme.repositories" />

    </beans:beans>
    ```
    在本例中,Spring能够通过`base-package`检测出指定路径下所有继承Repository或者其子接口的接口(有点绕口)。每找到一个接口的时候,`FactoryBean`就会创建一个合适的代理去处理以及调用里面的查询方法。每个注册的Bean的名称都是源于接口名称,例如:UserRepository将会被注册为userRepository。`base-package`允许使用通配符作为扫描格式。

    * 使用过滤器
      在默认的设置中,将使用全路径扫描的方式去检索接口,当然,你在业务上可能需要更细致的操作,这时候,你可以在`<repositories>`中使用`<include-filter>`或者`<exclude-filter>`。这样的话,
      你可以指定扫描的路径包含或者不包含指定的路径。

      ```xml
      <repositories base-package="com.acme.repositories">
        <context:exclude-filter type="regex" expression=".*SomeRepository" />
      </repositories>
      ```
      这个例子中,我们排除了所有以`SomeRepository`结尾的接口。

  * JavaConfig
    可以通过在一个JavaConfig 类上用`@Enable${store}Repositories`注解来触发.

    ```java
    @Configuration
    @EnableJpaRepositories("com.acme.repositories")
    class ApplicationConfiguration {

      @Bean
      public EntityManagerFactory entityManagerFactory() {
        // …
      }
    }
    ```

  * 独立使用
    你可以不在Spring容器里面使用repository。但是你还需要Spring的依赖包在你的`classpath`中,你需要使用`RepositoryFactory`来实现,代码如下:

    ```java
    RepositoryFactorySupport factory = ... // 初始化
    UserRepository repository = factory.getRepository(UserRepository. class);
    ```
  * 自定义Repository实现
      我们可以自己实现repository的方法。
  * 在repository中添加自定义方法
    * 自定义接口:

    ```java
    interface UserRepositoryCustom {
      public void someCustomMethod(User user);
    }
    ```
    * 自定义接口的实现类

    ```java
    class UserRepositoryImpl implements UserRepositoryCustom {

      public void someCustomMethod(User user) {
        // Your custom implementation
      }
    }
    ```
    * 扩展CRUDRepository

    ```java
    interface UserRepository extends CrudRepository<User, Long>, UserRepositoryCustom {
      // Declare query methods here
    }
    ```
      这样的话,就能够在常用的Repository中实现自己的方法。

  * 配置
    在XML的配置里面,框架会自动搜索base-package里面的实现类,这些实现类的后缀必须满足repository-impl-postfix中指定的命名规则,默认的规则是:Impl

    ```xml
    <repositories base- package ="com.acme.repository" />
    <repositories base- package ="com.acme.repository" repository-impl-postfix="FooBar" />
    ```
    第一个配置我们将找到com.acme.repository.UserRepositoryImpl,而第二个配置我们将找到com.acme.repository.UserRepositoryFooBar。
  * 人工装配
    前面的代码中,我们使用了注释以及配置去自动装载。如果你自己定义的实现类需要特殊的装载,那么你可以跟普通bean一样声明出来就可以了,框架会手工的装载起来,而不是创建本身。

    ```xml
    <repositories base-package="com.acme.repository"/>
    <beans:bean id="userRepositoryImpl" class="…">
    </beans:bean>
    ```
  * 为所有的repository添加自定义方法
    假如你要为所有的repository添加一个方法,那么前面的方法都不可行。你可以这样做:
    1.你需要先声明一个中间接口,然后让你的接口来继承这个中间接口而不是Repository接口,代码如下:
    中间接口:

    ```java
    @NoRepositoryBean
    public interface MyRepository<T, ID extends Serializable>
      extends PagingAndSortingRepository<T, ID> {

      void sharedCustomMethod(ID id);
    }
    ```
    2.这时候,我们需要创建我们的实现类,这个实现类是基于Repository中的基类的,这个类会作为Repository代理的自定义类来执行。

    ```java
    public class MyRepositoryImpl<T, ID extends Serializable>
      extends SimpleJpaRepository<T, ID> implements MyRepository<T, ID> {

      private final EntityManager entityManager;

      public MyRepositoryImpl(JpaEntityInformation entityInformation,
                              EntityManager entityManager) {
        super(entityInformation, entityManager);

        // Keep the EntityManager around to used from the newly introduced methods.
        this.entityManager = entityManager;
      }

      public void sharedCustomMethod(ID id) {
        // implementation goes here
      }
    }
    ```

    3.配置自定义Repository的`base class`

    * 用JavaConfig类

      ```java
      @Configuration
      @EnableJpaRepositories(repositoryBaseClass = MyRepositoryImpl.class)
      class ApplicationConfiguration { … }
      ```

    * 用XML

      ```xml
      <repositories base-package="com.acme.repository" repository-base-class="….MyRepositoryImpl" />
      ```

  * Spring Data 扩展
    这部分我们将会把SpringData扩展到其他框架中,目前我们继承的目标是SpringMVC。

  * Querydsl扩展

    ```java
    public interface QueryDslPredicateExecutor<T> {

        T findOne(Predicate predicate);

        Iterable<T> findAll(Predicate predicate);

        long count(Predicate predicate);

        boolean exists(Predicate predicate);

        // … more functionality omitted.
    }
    ```

    > Finds and returns a single entity matching the `Predicate`.
    Finds and returns all entities matching the `Predicate`.
    Returns the number of entities matching the `Predicate`.
    Returns if an entity that matches the `Predicate` exists.

    > [`Predicate`简介](http://www.iteye.com/topic/1132035)

  * 利用`Querydsl`,在你的Repository接口上继承`QueryDslPredicateExecutor`

    ```java
    interface UserRepository extends CrudRepository<User, Long>, QueryDslPredicateExecutor<User> {}
    ```
  * 然后可以用Querydsl `Predicate`写类型安全的查询

    ```java
    Predicate predicate = user.firstname.equalsIgnoreCase("dave")
    .and(user.lastname.startsWithIgnoreCase("mathews"));
    userRepository.findAll(predicate);
    ```
  * Web支持
    SpringData支持很多web功能。当然你的应用也要有SpringMVC的Jar包,有的还需要继承Spring HATEOAS。
    通常来说,你可以在你的JavaConfig配置类中加入@EnableSpringDataWebSupport即可:

    ```java
      @Configuration
      @EnableWebMvc
      @EnableSpringDataWebSupport
      class WebConfiguration { }
    ```

    这个注解注册了几个功能,我们稍后会说,他也能检测Spring HATEOAS,并且注册他们。
    如果你用XML配置的话,那么你可以用下面的配置:
    在xml中配置:

    ```xml
      <bean class="org.springframework.data.web.config.SpringDataWebConfiguration" />
      <!-- If you're using Spring HATEOAS as well register this one *instead* of the former -->
      <bean class="org.springframework.data.web.config.HateoasAwareSpringDataWebConfiguration" />
    ```

    * 基本的web支持
      上面的配置注册了以下的几个功能:
        1. DomainClassConverter将会让SpringMVC能从请求参数或者路径参数中解析出来。
        2. HandlerMethodArgumentResolver 能让SpringMVC从请求参数中解析出Pageable(分页)与Sort(排序)。
    * DomainClassConverter
      这个类允许你在SpringMVC控制层的方法中直接使用你的领域类型(Domain types),如下:

      ```java
      @Controller
      @RequestMapping("/users")
      public class UserController {

        @RequestMapping("/{id}")
        public String showUserForm(@PathVariable("id") User user, Model model) {

          model.addAttribute("user", user);
          return "userForm";
        }
      }
      ```
      正如你所见,上面的方法直接接收了一个User对象,你不需要做任何的搜索操作,这个转换器自动的设id的值进去对象中,并且最终调用了`findOne`方法查询出实体。`(注:当前的Repository
  必须实现CrudRepository)`
    * HandlerMethodArgumentResolver分页排序
      这个配置项同时注册了PageableHandlerMethodArgumentResolver 和 SortHandlerMethodArgumentResolver,使得Pageable跟Sort能作为控制层的参数使用:

      ```java
      @Controller
      @RequestMapping("/users")
      public class UserController {

        @Autowired UserRepository repository;

        @RequestMapping
        public String showUsers(Model model, Pageable pageable) {

          model.addAttribute("users", repository.findAll(pageable));
          return "users";
        }
      }
      ```

      这个配置会让SpringMVC传递一个Pageable实体参数,下面是默认的参数:

      | 参数名                    |    说明  |
      | :---------------------- | :-----------|
      | page| 你要获取的页数 |
      |size |一页中最大的数据量|
      |sort | 需要被排序的属性(格式:属性1,属性2(ASC/DESC)),默认是ASC,使用多个字段排序,你可以使用sort=first&sort=last,asc |

      如果你需要对多个表写多个分页或排序,那么你需要用@Qualifier来区分,请求参数的前缀是${qualifire}_,那么你的方法可能变成这样:

      ```java
      public String showUsers(Model model,
          @Qualifier("foo") Pageable first,
          @Qualifier("bar") Pageable second) { … }
      ```

       你需要填写foo_page和bar_page等。
       默认的Pageable相当于new PageRequest(0,20),你可以用@PageableDefaults注解来放在Pageable上。

    * 超媒体分页
      Spring HATEOAS有一个PagedResources类,他丰富了Page实体以及一些让用户更容易导航到资源的请求方式。Page转换到PagedResources是由一个实现了Spring HATEOAS
  ResourceAssembler接口的实现类:PagedResourcesAssembler提供转换的。

      ```java
      @Controller
      class PersonController {
        @Autowired PersonRepository repository;
        @RequestMapping(value = "/persons", method = RequestMethod.GET)
        HttpEntity<PagedResources<Person>> persons(Pageable pageable,
          PagedResourcesAssembler assembler) {
          Page<Person> persons = repository.findAll(pageable);
          return new ResponseEntity<>(assembler.toResources(persons), HttpStatus.OK);
        }
      }
      ```

      上面的toResources方法会执行以下的几个步骤:

      1. Page对象的内容会转换成为PagedResources对象。
      2. PagedResources会的到一个PageMetadata的实体附加,包含Page跟PageRequest。
      3. PagedResources会根据状态得到prev跟next链接,这些链接指向URI所匹配的方法中。分页参数会根据PageableHandlerMethodArgumentResolver配置,以让其在后面的方法中
      4. 解析使用。
      假使我们现在有30个Person实例在数据库中,你可以通过`GET` http://localhost:8080/persons 并且 你会得到像下边这样的一些反馈:

      ```json
      { "links" : [ { "rel" : "next",
                  "href" : "http://localhost:8080/persons?page=1&size=20 }
        ],
        "content" : [
           … // 20 Person instances rendered here
        ],
        "pageMetadata" : {
          "size" : 20,
          "totalElements" : 30,
          "totalPages" : 2,
          "number" : 0
        }
      }
      ```
    * Querydsl web支持
      它可以从`Request`的query string中提取出一些属性,并转换成Querydsl样式.
      这意味着它可以把

      ```http
      ?firstname=Dave&lastname=Matthews
      ```
      这样的query string 解析成:

      ```java
      QUser.user.firstname.eq("Dave").and(QUser.user.lastname.eq("Matthews"))
      ```
    * 使用`QuerydslPredicateArgumentResolver`.

      *在未来当Querydsl在classpath中被发现时,仅仅使用`@EnableSpringDataWebSupport`就可以激活*
      在方法上增加一个`@QuerydslPredicate`将会提供一个可以通过`QueryDslPredicateExecutor`来执行的`Predicate`
      *使用`QueryDslPredicateExecutor`中的`root`属性来确定`@QuerydslPredicate`的返回值类型*

      ```java
      @Controller
      class UserController {

        @Autowired UserRepository repository;

        @RequestMapping(value = "/", method = RequestMethod.GET)
        String index(Model model, @QuerydslPredicate(root = User.class) Predicate predicate,
                Pageable pageable, @RequestParam MultiValueMap<String, String> parameters) {

          model.addAttribute("users", repository.findAll(predicate, pageable));

          return "index";
        }
      }
      ```

      为User解析query string来匹配`Predicate`
      默认的绑定方式如下:

      1.`Object` on simple properties as `eq`.
      2.`Object` on collection like properties as `contains`.
      3.`Collection` on simple properties as `in`.
      这样的绑定可以通过`@QuerydslPredicate`的`bingdings`属性来定制,也可以使用Java 8 的`default methods` 在Repository接口上追加`QuerydslBinderCustomizer`

      ```java
      interface UserRepository extends CrudRepository<User, String>,
                                    QueryDslPredicateExecutor<User>,
                                    QuerydslBinderCustomizer<QUser> {
        @Override
        default public void customize(QuerydslBindings bindings, QUser user) {
          bindings.bind(user.username).first((path, value) -> path.contains(value))
          bindings.bind(String.class)
            .first((StringPath path, String value) -> path.containsIgnoreCase(value));
          bindings.excluding(user.password);
        }
      }
      ```

      * `QueryDslPredicateExecutor` 给`Predicate` 提供具体的查找方法.
      * 在Repository 接口中`QuerydslBinderCustomizer` 的定义和`@QuerydslPredicate(bindings=…​)`的快捷方式将会被自动抓取
      * `username` 属性的绑定就是一个简单的`contains`绑定.
      * 默认的`String`属性绑定是不区分大小写的匹配`contains`
      * Exclude the password property from Predicate resolution.

  * Repository填充
    如果你用过Spring JDBC,那么你肯定很熟悉使用SQL去填写数据源(DataSource),在这里,我们可以使用XML或者Json去填写数据,而不再使用SQL填充。
    假如你有一个data.json的文件,如下:

    ```json
    [ { "_class" : "com.acme.Person",
     "firstname" : "Dave",
      "lastname" : "Matthews" },
      { "_class" : "com.acme.Person",
     "firstname" : "Carter",
      "lastname" : "Beauford" } ]
    ```

    要PersonRepository填充这些数据进去,你需要做如下的声明:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:repository="http://www.springframework.org/schema/data/repository"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/data/repository
        http://www.springframework.org/schema/data/repository/spring-repository.xsd">
    <repository:jackson2-populator locations="classpath:data.json" />
    </beans>
    ```
  这个声明使得data.json能够通过Jackson ObjectMapper被其他地方读取,反序列化。
  * Legacy Web(传统web)支持

    * 在SpringMVC中绑定领域类(Domain class)
      你在开发web项目的时候,你经常需要从URL或者请求参数中解析领域类中的ID,你可能是这么做得:

      ```java
      @Controller
      @RequestMapping("/users")
      public class UserController {

        private final UserRepository userRepository;

        @Autowired
        public UserController(UserRepository userRepository) {
          Assert.notNull(repository, "Repository must not be null!");
          this.userRepository = userRepository;
        }

        @RequestMapping("/{id}")
        public String showUserForm(@PathVariable("id") Long id, Model model) {

          // Do null check for id
          User user = userRepository.findOne(id);
          // Do null check for user

          model.addAttribute("user", user);
          return "user";
        }
      }
      ```
      首先你要注入一个UserRepository ,然后通过findOne查询出结果。幸运的是,Spring提供了自定义组件允许你从String类型到任意类型的转换。
    * PropertyEditors(属性编辑器)
      在Spring3.0之前,Java的PropertyEditor已经被使用。现在我们要集成它,SpringData提供了一个DomainClassPropertyEditorRegistrar类,他能在ApplicationContext中查找SpringData的
  Repositories,并且注册自定义的PropertyEditor。

      ```xml
      <bean class="….web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
        <property name="webBindingInitializer">
          <bean class="….web.bind.support.ConfigurableWebBindingInitializer">
            <property name="propertyEditorRegistrars">
              <bean class="org.springframework.data.repository.support.DomainClassPropertyEditorRegistrar" />
            </property>
          </bean>
        </property>
      </bean>
      ```

      如果你做了上面的工作,那么你在前面的例子中,会大大减少工作量:

      ```java
        @Controller
        @RequestMapping("/users")
        public class UserController {
        @RequestMapping("/{id}")
        public String showUserForm(@PathVariable("id") User user, Model model) {
        model.addAttribute("user", user);
        return "userForm";
        }
        }
      ```
  * 转换服务
    在Spring3以后,PropertyEditor已经被转换服务取代了,SpringData现在用DomainClassConverter模仿
  DomainClassPropertyEditorRegistrar中的实现。你可以使用如下的配置:

    ```xml
      <mvc:annotation-driven conversion-service="conversionService"/>
      <bean class="org.springframework.data.repository.support.DomainClassConverter">
      <constructor-arg ref="conversionService"/>
    ```

    如果你是用JavaConfig,你可以集成SpringMVC的WebMvcConfigurationSupport并且处理FormatingConversionService,那么你可以这么做:

    ```java
    public class WebConfiguration extends WebMvcConfigurationSupport {
      // 省略其他配置
      @Bean
      public DomainClassConverter<?> domainClassConverter() {
      return new DomainClassConverter<FormattingConversionService>(mvcConversionService());
      }
    }
    ```