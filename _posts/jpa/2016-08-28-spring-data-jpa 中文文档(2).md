---
layout: post
#标题配置
title:  spring-data-jpa 中文文档(2)
#时间配置
date:   2016-08-28 01:08:00 +0800
#大类配置
categories: JPA
#小类配置
tag: JPA
---
spring data jpa 中文文档 2

## **JPA Repositories**
  * 简介
    * Spring命名空间
      SpringData使用了自定义的命名空间去定义repository。通常我们会使用repositories元素:

      ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:jpa="http://www.springframework.org/schema/data/jpa"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/data/jpa
            http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

          <jpa:repositories base-package="com.acme.repositories" />
        </beans>
      ```

      这个配置中启用了持久化异常处理,所有标志了`@Repository`的Bean将会被转换成为Spring的`DataAccessException`。
    * 自定义命名空间属性
      除了repositories,JPA命名空间还提供了其他的属性去控制:

        | 属性名|    属性值|
        | :-------- | :--------|
        | entity-manager-factory-ref | 默认的话,是使用`ApplicationContext`中找到的`EntityManagerFactory`,如果有多个的时候,则需要特别指明这个属性,他将会对repositories路径中找到的类进行处理 |
        |transaction-manager-ref|默认使用系统定义的`PlatformTransactionManager`,如果有多个事务管理器的话,则需特别指定。|

    * 基于注解的配置
      SpringData JPA支持JavaConfig方式的配置:

      ```java
      @Configuration
      @EnableJpaRepositories
      @EnableTransactionManagement
      class ApplicationConfig {

        @Bean
        public DataSource dataSource() {

          EmbeddedDatabaseBuilder builder = new EmbeddedDatabaseBuilder();
          return builder.setType(EmbeddedDatabaseType.HSQL).build();
        }

        @Bean
        public EntityManagerFactory entityManagerFactory() {

          HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
          vendorAdapter.setGenerateDdl(true);

          LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
          factory.setJpaVendorAdapter(vendorAdapter);
          factory.setPackagesToScan("com.acme.domain");
          factory.setDataSource(dataSource());
          factory.afterPropertiesSet();

          return factory.getObject();
        }

        @Bean
        public PlatformTransactionManager transactionManager() {

          JpaTransactionManager txManager = new JpaTransactionManager();
          txManager.setEntityManagerFactory(entityManagerFactory());
          return txManager;
        }
      }
      ```
      上面的配置中,我们设置了一个内嵌的HSQL数据库,我们也配置了`EntityManagerFactory`,并且使用Hibernate作为持久层。最后也定义了`JPATransactionManager`。最上面我们还使用了`@EnableJpaRepositories`注解。

  * 持久化实体
    * 保存实体
      保存实体,我们之前使用了CrudRepository.save(...)方法。他会使用相关的JPA EntityManager来调用persist或者merge,如果数据没存在于数据库中,则调用entityManager.persist(..),否
  则调用entityManager.merge(...)。
      * 实体状态监测策略
        SpringData JPA提供三种策略去监测实体是否存在:

        | 属性名|    属性值|
        |:-------- | :--------|
        |Id-Property inspection (default)|默认的会通过ID来监测是否新数据,如果ID属性是空的,则认为是新数据,反则认为旧数据|
        |Implementing Persistable|如果实体实现了Persistable接口,那么就会通过isNew的方法来监测。|
        |Implementing EntityInformation|这个是很少用的|

  * 查询方法
    * 查询策略
      你可以写一个语句或者从方法名中查询。
      * 声明查询方法
          虽然说方法名查询的方式很方便,可是你可能会遇到方法名查询规则不支持你所要查询的关键字或者方法名写的很长,不方便,或者很丑陋。那么你就需要通过命名查询或者在方法上使用
  `@Query`来解决。
      * 查询创建
        通常我们可以使用方法名来解析查询语句,例如:

        ```java
        public interface UserRepository extends Repository<User, Long> {

          List<User> findByEmailAddressAndLastname(String emailAddress, String lastname);
        }
        ```
        其所支持的在方法名中可以使用的关键字:

        |关键字 |    例子| JPQL片段|
        |:-------- | :--------| :------------|
        |And|findByLastnameAndFirstname|… where x.lastname = ?1 and x.firstname = ?2|
        |Or|findByLastnameOrFirstname|… where x.lastname = ?1 or x.firstname = ?2|
        |Is,Equals|findByFirstname,findByFirstnameIs,findByFirstnameEquals|… where x.firstname = ?1|
        |Between|findByStartDateBetween|… where x.startDate between ?1 and ?2|
        |LessThan|findByAgeLessThan|… where x.age < ?1|
        |LessThanEqual|findByAgeLessThanEqual|… where x.age ⇐ ?1|
        |GreaterThan|findByAgeGreaterThan|… where x.age > ?1|
        |GreaterThanEqual|findByAgeGreaterThanEqual|… where x.age >= ?1|
        |After|findByStartDateAfter|… where x.startDate > ?1|
        |Before|findByStartDateBefore|… where x.startDate < ?1|
        |IsNull|findByAgeIsNull|… where x.age is null|
        |IsNotNull,NotNull|findByAge(Is)NotNull|… where x.age not null|
        |Like|findByFirstnameLike|… where x.firstname like ?1|
        |NotLike|findByFirstnameNotLike|… where x.firstname not like ?1|
        |StartingWith|findByFirstnameStartingWith|… where x.firstname like ?1 (parameter bound with appended %)|
        |EndingWith|findByFirstnameEndingWith|… where x.firstname like ?1 (parameter bound with prepended %)|
        |Containing|findByFirstnameContaining|… where x.firstname like ?1 (parameter bound wrapped in %)|
        |OrderBy|findByAgeOrderByLastnameDesc|… where x.age = ?1 order by x.lastname desc|
        |Not|findByLastnameNot|… where x.lastname <> ?1|
        |In|findByAgeIn(Collection<Age> ages)|… where x.age in ?1|
        |NotIn|findByAgeNotIn(Collection<Age> age)|… where x.age not in ?1|
        |True|findByActiveTrue()|… where x.active = true|
        |False|findByActiveFalse()|… where x.active = false|
        |IgnoreCase|findByFirstnameIgnoreCase|… where UPPER(x.firstame) = UPPER(?1)|

        *`In`和`NotIn`也可以用`Collection`的子类.*
  * 使用JPA命名查询
    * 注解方式

      ```java
      @Entity
      @NamedQuery(name = "User.findByEmailAddress",
        query = "select u from User u where u.emailAddress = ?1")
      public class User {

      }
      ```
    * XML方式 略.
    * 声明接口
      要使用上面的命名查询,我们的接口需要这么声明

      ```java
      public interface UserRepository extends JpaRepository<User, Long> {

        List<User> findByLastname(String lastname);

        User findByEmailAddress(String emailAddress);
      }
      ```
      SpringData会先从域类中查询配置,根据”.(原点)“区分方法名,而不会使用自动方法名解析的方式去创建查询。

  * 使用`@Query`
    命名查询适合用于小数量的查询,我们可以使用@Query来替代:

    ```java
    public interface UserRepository extends JpaRepository<User, Long> {

      @Query("select u from User u where u.emailAddress = ?1")
      User findByEmailAddress(String emailAddress);
    }
    ```
    在表达式中使用Like查询,例子如下:

    ```java
    public interface UserRepository extends JpaRepository<User, Long> {
      @Query("select u from User u where u.firstname like %?1")
      List<User> findByFirstnameEndsWith(String firstname);
    }
    ```
    这个例子中,我们使用了%,当然,你的参数就没必要加入这个符号了。

    使用原生sql查询
    我们可以在`@Query`中使用本地查询,当然,你需要设置`nativeQuery=true`,必须说明的是,这样的话,*就不再支持分页以及排序。*

    ```java
    public interface UserRepository extends JpaRepository<User, Long> {

      @Query(value = "SELECT * FROM USERS WHERE EMAIL_ADDRESS = ?1", nativeQuery = true)
      User findByEmailAddress(String emailAddress);
    }
    ```
  * 使用命名参数
    使用命名查询,我们需要用到@Param来注释到指定的参数上,如下:

    ```java
    public interface UserRepository extends JpaRepository<User, Long> {

      @Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname")
      User findByLastnameOrFirstname(@Param("lastname") String lastname,
                                     @Param("firstname") String firstname);
    }
    ```

  * 使用SpEL表达式
    在Spring Data JPA 1.4以后,我们支持在`@Query`中使用`SpEL表达式`([简介](http://sishuok.com/forum/blogPost/list/2463.html))来接收变量。
    SpEL支持的变量:

    |变量名|使用方式|描述|
    |:-------|:-----------|:-----|
    |entityName|select x from #{#entityName} x|根据给定的Repository自动插入相关的entityName。有两种方式能被解析出来:如果域类型定义了@Entity属性名称。或者直接使用类名称。|

    以下的例子中,我们在查询语句中插入表达式(你也可以用@Entity(name = "MyUser")。

    ```java
    @Entity
    public class User {
      @Id
      @GeneratedValue
      Long id;
      String lastname;
    }
    public interface UserRepository extends JpaRepository<User,Long> {
      @Query("select u from #{#entityName} u where u.lastname = ?1")
      List<User> findByLastname(String lastname);
    }
    ```

    如果你想写一个通用的Repository接口,那么你也可以用这个表达式来处理:

    ```java

      @MappedSuperclass
      public abstract class AbstractMappedType {
        …
        String attribute
      }

      @Entity
      public class ConcreteType extends AbstractMappedType { … }

      @NoRepositoryBean
      public interface MappedTypeRepository<T extends AbstractMappedType>
        extends Repository<T, Long> {

        @Query("select t from #{#entityName} t where t.attribute = ?1")
        List<T> findAllByAttribute(String attribute);
      }

      public interface ConcreteRepository
        extends MappedTypeRepository<ConcreteType> { … }
    ```
  * 修改语句
    之前我们演示了如何去声明查询语句,当然我们还有修改语句。修改语句的实现,我们只需要在加多一个注解`@Modifying`:

    ```java
    @Modifying
    @Query("update User u set u.firstname = ?1 where u.lastname = ?2")
    int setFixedFirstnameFor(String firstname, String lastname);
    ```
    这样一来,我们就使用了update操作来代替select操作。当我们发起update操作后,可能会有一些过期的数据产生,我们不需要自动去清除它们,因为EntityManager会有效的丢掉那些未提
  交的变更,如果你想EntityManager自动清除,那么你可以在@Modify上添加clearAutomatically属性(true);

  * 使用`QueryHints`(查询提示  [QueryHits简介](http://www.eclipse.org/eclipselink/documentation/2.5/jpa/extensions/queryhints.htm))

    ```java
    public interface UserRepository extends Repository<User, Long> {

      @QueryHints(value = { @QueryHint(name = "name", value = "value")},
                  forCounting = false)
      Page<User> findByLastname(String lastname, Pageable pageable);
    }
    ```
  * 配置获取和负载图(loadgraph) [`@NamedEntityGraph`简介](https://yq.aliyun.com/articles/2378)
    JPA2.1 支持 通过`@EntityGraph`及其子类型`@NamedEntityGraph`来定义获取和负载.它们可以被直接在实体类上,用来配置查询结果的获取计划.获取的方式(获取/负载)可以通过`@EntityGraph`的`type`属性来进行配置.
    在一个实体类上定义 `named entity graph`

    ```java
    @Entity
    @NamedEntityGraph(name = "GroupInfo.detail",
      attributeNodes = @NamedAttributeNode("members"))
    public class GroupInfo {

      // default fetch mode is lazy.
      @ManyToMany
      List<GroupMember> members = new ArrayList<GroupMember>();

      …
    }
    ```
    在repository接口中引用在实体类上定义的`named entity graph`

    ```java
    @Repository
    public interface GroupRepository extends CrudRepository<GroupInfo, String> {

      @EntityGraph(value = "GroupInfo.detail", type = EntityGraphType.LOAD)
      GroupInfo getByGroupName(String name);
    ```
    它也可以通过`@EntityGraph`注解来直接点对点的指定`entity graphs`.假如依照`EntityGraph` `attributePaths`可以被正确的找到,就可以不用在实体类上写`@NamedEntityGraph`注解了:

    ```java
    @Repository
    public interface GroupRepository extends CrudRepository<GroupInfo, String> {

      @EntityGraph(attributePaths = { "members" })
      GroupInfo getByGroupName(String name);

    }
    ```
  * 投影(Projections)
    通常情况下 Spring Data Repositories 会返回整个domain 类.有时候,你需要因为不同的原因,修改domain类的返回结果.
    看下面的例子:

    ```java
    @Entity
    public class Person {

      @Id @GeneratedValue
      private Long id;
      private String firstName, lastName;

      @OneToOne
      private Address address;
      …
    }

    @Entity
    public class Address {

      @Id @GeneratedValue
      private Long id;
      private String street, state, country;

      …
    }
    ```

    `Person`的几个属性:

    * `id`是主键
    * `fristName`和`lastName`是数据属性.
    * `address`链接到其他的domain类型.
    现在假设我们创建了一个像下边这样的repository:

      ```java
      interface PersonRepository extends CrudRepository<Person, Long> {
        Person findPersonByFirstName(String firstName);
      }
      ```
    Spring Data将会返回domain类的所有属性.现在有两种选择去仅仅返回`address`属性:

    * 为`Address`定义一个repository:

      ```java
      interface AddressRepository extends CrudRepository<Address, Long> {}
      ```

      在这种情况下 用`PersonRepository`将会返回整个`Person`对象.使用`AddressRepository`仅仅会返回`Address`对象.
      但是,如果你真的不想暴露address的信息怎么办?你可以提供一个像下边这样的repository,仅仅提供你想暴露的属性:

      ```java
      interface NoAddresses {

        String getFirstName();

        String getLastName();
      }
      ```
      其中

      * `interface NoAddresses`定义一个接口
      * `String getFirstName();`导出`firstName`
      * `String getLastName();`导出`lastName`

      这个`NoAddress` 只有`firstName`和`lastName`的getter方法.它意味着它将不会提供任何的address信息.在定义查询方法的时候,应该用`NoAddress`代替`Person`

      ```java
      interface PersonRepository extends CrudRepository<Person, Long> {

        NoAddresses findByFirstName(String firstName);
      }
      ```
    * 改变属性的值
      现在你已经知道了怎么样对结果进行投影,已达到不暴露不必要的部分给用户.投影也可以调整要暴露的数据模型.你可以添加一个虚拟的属性:

      ```java
      interface RenamedProperty {

        String getFirstName();

        @Value("#{target.lastName}")
        String getName();
      }
      ```
      其中:

      * `interface RenamedProperty` 定义一个接口.
      * `String getFirstName();`导出`firstName`属性.
      * `@Value("#{target.lastName}") String getName();`导出`name`属性,由于此属性是虚拟的,因此他需要用``@Value("#{target.lastName}")`来指定数据的来源.

      如果你想获得一个人的全称.你可能要用`String.format("%s %s", person.getFirstName(), person.getLastName())`来拼接.用虚拟的属性可以这样来实现:

      ```java
      interface FullNameAndCountry {

        @Value("#{target.firstName} #{target.lastName}")
        String getFullName();

        @Value("#{target.address.country}")
        String getCountry();
      }
      ```
      实际上`@Value`可以完全访问对象及其内嵌的属性.`SpEL`表达式对于施加在投影方法上的定义来说也是非常强大的:
      想想你有一个如下的domain模型:

      ```java
      @Entity
      public class User {

        @Id @GeneratedValue
        private Long id;
        private String name;

        private String password;
        …
      }
      ```
      在某些情况下,你想让密码在可能的情况下不让其明文出现.这种情况下你可以用`@Value`和`SpEL表达式`来创建一个投影:

      ```java
      interface PasswordProjection {
        @Value("#{(target.password == null || target.password.empty) ? null : '******'}")
        String getPassword();
      }
      ```
      这个表达式判断当password是null或者empty的时候返回null,否则返回'\*\*\*\*\*\*'
  * 存储过程
    JPA2.1 规格介绍了JPA标准查询API对存储过程调用的支持.下面介绍下`@Procedure`注解的使用.

    ```sql
    DROP procedure IF EXISTS plus1inout
    CREATE procedure plus1inout (IN arg int, OUT res int)
    BEGIN ATOMIC
     set res = arg + 1;
    END
    ```
    存储过程的元数据可以在实体类上通过@NamedStoredProcedureQuery进行配置.

    ```java
    @Entity
    @NamedStoredProcedureQuery(name = "User.plus1", procedureName = "plus1inout", parameters = {
      @StoredProcedureParameter(mode = ParameterMode.IN, name = "arg", type = Integer.class),
      @StoredProcedureParameter(mode = ParameterMode.OUT, name = "res", type = Integer.class) })
    public class User {}
    ```
    在repository 方法上可以通过多种方式引用存储过程.存储过程可以直接通过`@Procedure`注解的`value`或者`procedureName`属性调用或者通过`name`属性.如果repository方法没有名字,则将其作为后备.

    ```java
    @Procedure("plus1inout")
    Integer explicitlyNamedPlus1inout(Integer arg);

    @Procedure(procedureName = "plus1inout")
    Integer plus1inout(Integer arg);

    @Procedure(name = "User.plus1IO")
    Integer entityAnnotatedCustomNamedProcedurePlus1IO(@Param("arg") Integer arg);

    @Procedure
    Integer plus1(@Param("arg") Integer arg);
    ```
  * JPA2 引入了criteria API 去建立查询,Spring Data JPA使用Specifications来实现这个API。在Repository中,你需要继承JpaSpecificationExecutor:

    ```java
    public interface CustomerRepository extends CrudRepository<Customer, Long>, JpaSpecificationExecutor {
     …
    }
    ```
    下面先给个例子,演示如何利用findAll方法返回所有符合条件的对象:

    ```java
    List<T> findAll(Specification<T> spec);
    ```

    Specification 接口定义如下:

    ```java
    public interface Specification<T> {
      Predicate toPredicate(Root<T> root, CriteriaQuery<?> query,
                CriteriaBuilder builder);
    }
    ```

    好了,那么我们如何去实现这个接口呢?代码如下:

    ```java
    public class CustomerSpecs {

      public static Specification<Customer> isLongTermCustomer() {
        return new Specification<Customer>() {
          public Predicate toPredicate(Root<Customer> root, CriteriaQuery<?> query,
                CriteriaBuilder builder) {

             LocalDate date = new LocalDate().minusYears(2);
             return builder.lessThan(root.get(_Customer.createdAt), date);
          }
        };
      }

      public static Specification<Customer> hasSalesOfMoreThan(MontaryAmount value) {
        return new Specification<Customer>() {
          public Predicate toPredicate(Root<T> root, CriteriaQuery<?> query,
                CriteriaBuilder builder) {

             // build query here
          }
        };
      }
    }
    ```

    好了,那如果有多个需要结合的话,我们可以这么做:

    ```java
    MonetaryAmount amount = new MonetaryAmount(200.0, Currencies.DOLLAR);
    List<Customer> customers = customerRepository.findAll(
      where(isLongTermCustomer()).or(hasSalesOfMoreThan(amount)));
    ```
  * Query by Example
    * 介绍
      这一部分介绍Query by Example和解释怎么去使用Examples.
    * 使用
      Query by Example由三部分组成.

      * `Probe` 这是填充字段的domain对象的实际范例.
      * `ExampleMatcher` `ExampleMatcher` 描述怎么去匹配特定字段的细节.他可以通过多个Examples进行重复利用.
      * `Example` 它由`ExampleMatcher`和`Probe`组成.用来创建一个查询.

      Query by Example使用与多种情况,但是它也有一些限制.
      适用于:

      * 用静态或者静态的约束来查询数据
      * domain对象的频繁重构,而不用担心破坏现有的查询
      * 根据底层数据存储API进行独立工作
      限制:
      * 用`AND`关键字进行条件查询的时候.
      * 不支持内嵌的/组合的属性约束,像`firstname = ?0 or (firstname = ?1 and lastname = ?2)`
      * 仅仅支持starts/contains/ends/regex匹配strings或者精确匹配其他属性类型.
      在使用Query by Example之前,你需要有一个domain对象:

        ```java
        public class Person {

          @Id
          private String id;
          private String firstname;
          private String lastname;
          private Address address;

          // … getters and setters omitted
        }
        ```
        这是一个简单的domain对象.你可以使用它去创建一个`Example`默认情况下,字段为`null`值时会被忽略.
        `Examples`通过使用`of`工厂方法或者`ExampleMatcher`来创建.`Example`是不可改变的.

        ```java
        Person person = new Person();
        person.setFirstname("Dave");

        Example<Person> example = Example.of(person);
        ```
        其中:

        * `Person person = new Person(); `创建一个domain对象实例
        * `person.setFirstname("Dave"); `设置firstName属性去查询
        * `Example<Person> example = Example.of(person);` 创建`Example`
        Examples用repositories来执行是非常理想的.要使用它需要让你的repository接口继承`QueryByExampleExecutor<T>`.

          ```java
          public interface QueryByExampleExecutor<T> {

            <S extends T> S findOne(Example<S> example);

            <S extends T> Iterable<S> findAll(Example<S> example);

            // … more functionality omitted.
          }
          ```
      * Example matchers
        Example不仅仅局限于默认的设置.你可以给strings定义自己的默认值然后去匹配.使用`ExampleMatcher`绑定null和特定属性的设置.

        ```java
        Person person = new Person();
        person.setFirstname("Dave");

        ExampleMatcher matcher = ExampleMatcher.matching()
          .withIgnorePaths("lastname")
          .withIncludeNullValues()
          .withStringMatcherEnding();

        Example<Person> example = Example.of(person, matcher);
        ```
        其中:

        * `Person person = new Person();`  创建一个domain对象实例.
        * 设置属性值去查询.
        * `ExampleMatcher matcher = ExampleMatcher.matching()`创建一个`ExampleMatcher `让其可以使用,但没有多余的配置项.
        * `.withIgnorePaths("lastname")` Construct a new ExampleMatcher to ignore the property path lastname.
        * `.withIncludeNullValues()` Construct a new ExampleMatcher to ignore the property path lastname and to include null values.
        * `.withStringMatcherEnding();` Construct a new ExampleMatcher to ignore the property path lastname, to include null values, and use perform suffix string matching.
        * `Example<Person> example = Example.of(person, matcher);`根据domain对象和配置的`ExampleMatcher`对象来创建一个`Example`

        你可以给个别的属性指定行为.(比如.`firstname`和`lastname`以及domain对象的嵌套属性`address.city`)
        你可以调整他让他匹配大小写敏感的选项.

        ```java
        ExampleMatcher matcher = ExampleMatcher.matching()
          .withMatcher("firstname", endsWith())
          .withMatcher("lastname", startsWith().ignoreCase());
        }
        ```
        另一种配置matcher选项的方式是通过使用Java 8 lambdas表达式.这种方式是一种可以通过询问实现者修改matcher的回调方法.你不需要去返回matcher,因为他已经保留了matcher的实例.(引用)

        ```java
        ExampleMatcher matcher = ExampleMatcher.matching()
          .withMatcher("firstname", match -> match.endsWith())
          .withMatcher("firstname", match -> match.startsWith());
        }
        ```
        `ExampleMatcher`

        |设置|范围|
        |:-----|:------|
        |Null-handling|ExampleMatcher|
        |String matching|ExampleMatcher and property path|
        |Ignoring properties|Property path|
        |Case sensitivity|ExampleMatcher and property path|
        |Value transformation|Property path|

        *Property path指 这个设置项要跟在需要设置的属性后边,而不是在ExampleMatcher对象上进行设置.*

        **对比`ExampleMatcher matcher = ExampleMatcher.matching().withMatcher("firstname", endsWith()).withMatcher("lastname", startsWith().ignoreCase());`和`ExampleMatcher matcher = ExampleMatcher.matching().withIgnorePaths("lastname").withIncludeNullValues().withStringMatcherEnding();`**
      * 执行一个Example

        ```java
        public interface PersonRepository extends JpaRepository<Person, String> { … }

        public class PersonService {

          @Autowired PersonRepository personRepository;

          public List<Person> findPeople(Person probe) {
            return personRepository.findAll(Example.of(probe));
          }
        }
        ```
        仅仅只有`SingularAttribute`(单数属性)可以被属性匹配正确使用.
        `StringMatcher`选项:

        |匹配|逻辑结果|
        |:-----|:----------|
        |DEFAULT (case-sensitive)|firstname = ?0|
        |DEFAULT (case-insensitive)|LOWER(firstname) = LOWER(?0)|
        |EXACT (case-sensitive)|firstname = ?0|
        |STARTING (case-sensitive)|firstname like ?0 + '%'|
        |STARTING (case-insensitive)|LOWER(firstname) like LOWER(?0) + '%'|
        |ENDING (case-sensitive)|firstname like '%' + ?0|
        |ENDING (case-insensitive)|LOWER(firstname) like '%' + LOWER(?0)|
        |CONTAINING (case-sensitive)|firstname like '%' + ?0 + '%'|
        |CONTAINING (case-insensitive)|LOWER(firstname) like '%' + LOWER(?0) + '%'|

  * 事务
    默认的CRUD操作在Repository里面都是事务性的。如果是只读操作,只需要设置事务readOnly为true,其他的操作则配置为@Transaction。如果你想修改一个Repository的事务性,你只需要在子接口中重写并且修改他的事务:

    ```java
    public interface UserRepository extends CrudRepository<User, Long> {
        @Override
        @Transactional(timeout = 10)
        public List\<\User\>\ findAll();
        // Further query method declarations
    }
    ```

    这样会让findAll方法在10秒内执行否则会超时的非只读事务中。
    另一种修改事务行为的方式在于使用门面或者服务层中,他们包含了多个repository。

    ```java
    @Service
    class UserManagementImpl implements UserManagement {
    private final UserRepository userRepository;
    private final RoleRepository roleRepository;
    @Autowired
    public UserManagementImpl(UserRepository userRepository,
      RoleRepository roleRepository) {
      this.userRepository = userRepository;
      this.roleRepository = roleRepository;
    }
    @Transactional
    public void addRoleToAllUsers(String roleName) {
      Role role = roleRepository.findByName(roleName);
      for (User user : userRepository.findAll()) {
        user.addRole(role);
        userRepository.save(user);
      }
    }
    ```

    这将会导致在调用addRoleToAllUsers方法的时候,创建一个或者加入一个事务中去。实际在Repository里面定义的事务将会被忽略,而外部定义的事务将会被应用。当然,要使用事务,你需要声明<tx:annotation-driven />(这个例子中,假设你已经使用了component-scan)
  要让方法在事务中,最简单的方式就是使用@Transactional注解:

    ```java
    @Transactional(readOnly = true)
    public interface UserRepository extends JpaRepository<User, Long> {
    List <User> findByLastname(String lastname);
    @Modifying
    @Transactional
    @Query("delete from User u where u.active = false")
    void deleteInactiveUsers();
    }
    ```

    一般的查询操作,你需要设置readOnly=true。在deleteInactiveUsers方法中,我们添加了Modifying注解以及覆盖了Transactional,这样这个方法执行的时候readOnly=false了.

  * 锁
    想要为方法指定锁的类型,你只需要使用`@Lock`:

    ```java
    interface UserRepository extends Repository<User, Long> {
    // Plain query method
    @Lock(LockModeType.READ)
    List<User> findByLastname(String lastname);
    }
    ```
    当然你也可以覆盖原有的方法:

    ```java
    interface UserRepository extends Repository<User, Long> {
      // Redeclaration of a CRUD method
      @Lock(LockModeType.READ);
      List<User> findAll();
    }
    ```
  * 审计
    * 基础知识
      SpringData为您跟踪谁创建或者修改数据,以及相应的时间提供了复杂的支持。你现在想要这些支持的话,仅仅需要使用几个注解或者实现接口即可。
      * 注解方式: 我们提供了`@CreatedBy`, `@LastModifiedBy`去捕获谁操作的实体,当然还有相应的时间`@CreatedDate`和`@LastModifiedDate`。

        ```java
        class Customer {

          @CreatedBy
          private User user;

          @CreatedDate
          private DateTime createdDate;

          // … further properties omitted
        }
        ```
        正如你看到的,你可以选择性的使用这些注解。操作时间方面,你可以使用org.joda.time.DateTime 或者java.util.Date或者long/Long表示。

      * 基于接口的审计:
        如果你不想用注解来做审计的话,那么你可以实现`Auditable`接口。他暴露了审计属性的get/set方法。
        如果你不想实现接口,那么你可以继承AbstractAuditable,通常来说,注解方式时更加方便的。
      * 审计织入:
        如果你在用@CreatedBy或者@LastModifiedBy的时候,想织入当前的业务操作者,那你可以使用我们提供的AuditorAware<T>接口。T表示你想织入在这两个字段上的类型。

        ```java
        class SpringSecurityAuditorAware implements AuditorAware<User> {

          public User getCurrentAuditor() {

            Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

            if (authentication == null || !authentication.isAuthenticated()) {
              return null;
            }

            return ((MyUserDetails) authentication.getPrincipal()).getUser();
          }
        }
        ```
          在这个实现类中,我们使用SpringSecurity内置的Authentication来查找用户的UserDetails。

      * 通用审计配置:
        SpringData JPA有一个实体监听器,他可以用于触发捕获审计信息。要用之前,你需要在orm.xml里面注册AuditingEntityListener.当然你得引入`spring-aspects.jar`
        审计配置:

        ```xml
        <persistence-unit-metadata>
        <persistence-unit-defaults>
          <entity-listeners>
            <entity-listener class="….data.jpa.domain.support.AuditingEntityListener" />
          </entity-listeners>
        </persistence-unit-defaults>
        </persistence-unit-metadata>
        ```

        你也可以再每个实体类上使用`@EntityListeners`注解来激活`AuditingEntityListener`监听.

        ```java
        @Entity
        @EntityListeners(AuditingEntityListener.class)
        public class MyEntity {

        }
        ```
        要启用这个审计,我们还需要在配置文件里面配置多一条:

        ```xml
        <jpa:auditing auditor-aware-ref="yourAuditorAwareBean" />
        ```
        Spring Data JPA 1.5之后,你也可以使用`@EnableJpaAuditing `注解来激活.

        ```java
        @Configuration
        @EnableJpaAuditing
        class Config {

          @Bean
          public AuditorAware<AuditableUser> auditorProvider() {
            return new AuditorAwareImpl();
          }
        }
        ```
  * 其他: 略. [点击查看](http://docs.spring.io/spring-data/jpa/docs/1.10.1.RELEASE/reference/html/#jpa.misc)

  * 附录
    * 附录A: 命名空间引用
    `<repositories />`的元素:

      |属性名称|描述|
      |:----------|:------------------|
      |base-package|定义去扫描哪些继承了*Repository接口的用户自定义Repository接口.|
      |repository-impl-postfix|定义用户自定义实现sql语句的实现类以什么结尾,以用来自动发现.默认是`Impl`|
      |query-lookup-strategy|定义查询的策略.默认的是`create-if-not-found`|
      |named-queries-location|定义去哪里寻找已经写好了named-query查询的配置文件|
      |consider-nested-repositories|考虑是否要控制内嵌的Repository接口.默认是`false`|
    * 附录B:Populators 命名空间引用
      `<populator />`的元素:

      |属性名称|描述|
      |:----------|:------------------|
      |locations|寻找要填入Repository接口的对象的值的文件.|
    * 附录C:Repository 查询关键词
      支持的查询关键字:

      <table class="tableblock frame-all grid-all spread">
      <colgroup>
      <col style="width: 25%;">
      <col style="width: 75%;">
      </colgroup>
      <thead>
      <tr>
      <th class="tableblock halign-left valign-top">Logical keyword</th>
      <th class="tableblock halign-left valign-top">Keyword expressions</th>
      </tr>
      </thead>
      <tbody>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>AND</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>And</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>OR</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Or</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>AFTER</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>After</code>, <code>IsAfter</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>BEFORE</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Before</code>, <code>IsBefore</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>CONTAINING</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Containing</code>, <code>IsContaining</code>, <code>Contains</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>BETWEEN</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Between</code>, <code>IsBetween</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>ENDING_WITH</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>EndingWith</code>, <code>IsEndingWith</code>, <code>EndsWith</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>EXISTS</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Exists</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>FALSE</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>False</code>, <code>IsFalse</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>GREATER_THAN</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>GreaterThan</code>, <code>IsGreaterThan</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>GREATER_THAN_EQUALS</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>GreaterThanEqual</code>, <code>IsGreaterThanEqual</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>IN</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>In</code>, <code>IsIn</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>IS</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Is</code>, <code>Equals</code>, (or no keyword)</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>IS_NOT_NULL</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>NotNull</code>, <code>IsNotNull</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>IS_NULL</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Null</code>, <code>IsNull</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>LESS_THAN</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>LessThan</code>, <code>IsLessThan</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>LESS_THAN_EQUAL</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>LessThanEqual</code>, <code>IsLessThanEqual</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>LIKE</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Like</code>, <code>IsLike</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>NEAR</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Near</code>, <code>IsNear</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>NOT</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Not</code>, <code>IsNot</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>NOT_IN</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>NotIn</code>, <code>IsNotIn</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>NOT_LIKE</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>NotLike</code>, <code>IsNotLike</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>REGEX</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Regex</code>, <code>MatchesRegex</code>, <code>Matches</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>STARTING_WITH</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>StartingWith</code>, <code>IsStartingWith</code>, <code>StartsWith</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>TRUE</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>True</code>, <code>IsTrue</code></p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>WITHIN</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Within</code>, <code>IsWithin</code></p></td>
      </tr>
      </tbody>
      </table>
    * 附录D:Repository查询的返回结果类型
      支持的查询返回结果类型:

      <table class="tableblock frame-all grid-all spread">
      <colgroup>
      <col style="width: 25%;">
      <col style="width: 75%;">
      </colgroup>
      <thead>
      <tr>
      <th class="tableblock halign-left valign-top">Return type</th>
      <th class="tableblock halign-left valign-top">Description</th>
      </tr>
      </thead>
      <tbody>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>void</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">Denotes no return value.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock">Primitives</p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">Java primitives.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock">Wrapper types</p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">Java wrapper types.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>T</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">An unique entity. Expects the query method to return one result at most. In case no result is found <code>null</code> is returned. More than one result will trigger an <code>IncorrectResultSizeDataAccessException</code>.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Iterator&lt;T&gt;</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">An <code>Iterator</code>.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Collection&lt;T&gt;</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">A <code>Collection</code>.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>List&lt;T&gt;</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">A <code>List</code>.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Optional&lt;T&gt;</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">A Java 8 or Guava <code>Optional</code>. Expects the query method to return one result at most. In case no result is found <code>Optional.empty()</code>/<code>Optional.absent()</code> is returned. More than one result will trigger an <code>IncorrectResultSizeDataAccessException</code>.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Stream&lt;T&gt;</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">A Java 8 <code>Stream</code>.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Future&lt;T&gt;</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">A <code>Future</code>. Expects method to be annotated with <code>@Async</code> and requires Spring’s asynchronous method execution capability enabled.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>CompletableFuture&lt;T&gt;</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">A Java 8 <code>CompletableFuture</code>. Expects method to be annotated with <code>@Async</code> and requires Spring’s asynchronous method execution capability enabled.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>ListenableFuture</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">A <code>org.springframework.util.concurrent.ListenableFuture</code>. Expects method to be annotated with <code>@Async</code> and requires Spring’s asynchronous method execution capability enabled.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Slice</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">A sized chunk of data with information whether there is more data available. Requires a <code>Pageable</code> method parameter.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>Page&lt;T&gt;</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">A <code>Slice</code> with additional information, e.g. the total number of results. Requires a <code>Pageable</code> method parameter.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>GeoResult&lt;T&gt;</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">A result entry with additional information, e.g. distance to a reference location.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>GeoResults&lt;T&gt;</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">A list of <code>GeoResult&lt;T&gt;</code> with additional information, e.g. average distance to a reference location.</p></td>
      </tr>
      <tr>
      <td class="tableblock halign-left valign-top"><p class="tableblock"><code>GeoPage&lt;T&gt;</code></p></td>
      <td class="tableblock halign-left valign-top"><p class="tableblock">A <code>Page</code> with <code>GeoResult&lt;T&gt;</code>, e.g. average distance to a reference location.</p></td>
      </tr>
      </tbody>
      </table>
    * 附录E : faq. 略 [点击查看](http://docs.spring.io/spring-data/jpa/docs/1.10.1.RELEASE/reference/html/#faq)
    * 附录F: 词汇表: 略 [点击查看](http://docs.spring.io/spring-data/jpa/docs/1.10.1.RELEASE/reference/html/#glossary)
