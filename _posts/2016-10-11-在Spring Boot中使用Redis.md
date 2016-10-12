---
layout: post_layout
title: 在Spring Boot中使用Redis
time: 2016年10月12日 星期三
location: 济南
pulished: true
excerpt_separator: "#"
---

## Redis简介

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。 

> [Redis中国用户组](http://www.redis.cn/)

## Linux下Redis安装(单机)

* 下载[Redis3.2.3版本](http://www.redis.cn/download.html)并解压

  ```bash
  wget http://download.redis.io/releases/redis-3.2.3.tar.gz
  ```
    
  ```bash
  tar -zxf redis-3.2.3.tar.gz ./
  ```

## 编译并安装Redis

* 编译Redis

    ```bash
        cd xx/redis-3.2.3
    ```
    
    ```bash
        make
    ```
* 启动Redis

  * Redis配置文件redis.conf在根目录下,其中有很多默认配置和详细说明.在此就不全贴出来了.
    
    ```bash
    #修改redis是否已守护进程的方式运行 yes则启动守护进程
    daemonize no
    
    #redis以守护进程运行时 制定其pidfile文件
    pidfile /var/run/redis.pid
    
    #Redis端口号,默认6379
    port 6379
    
    #客户端闲置超时时间,0则表示关闭该功能
    timeout 300
    
    #日志记录级别，Redis共支持四个级别：debug、verbose、notice、warning，默认为verbose
    loglevel verbose
    
    #日志输出方式,默认为标准输出,即输出到控制台;如果redis以守护进程的方式运行,在这个地方配置标准输出的话,则会将日志发送给/dev/null,即什么都看不到
    logfile stdout
    
    #数据库的数量,默认为0,可以使用SELECT <dbid>命令在redis-cli客户端切换数据库
    databases 0
    
    #指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合(redis是有机制将内存数据保存到硬盘的)
    #Redis默认配置文件中提供了三个条件：
    #save 900 1
    #save 300 10
    #save 60 10000
    #分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。
    save <seconds> <changes>
    
    #制定Redis存储本地数据库时是否压缩数据,默认yes.不启用的话会导致数据库文件很大
    rdbcompression yes
    
    #本地数据库文件名,默认dump.rdb
    dbfilename dump.rdb
    
    #本地数据库存放目录
    dir ./
    
    #设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步(在master-slave模式时启用)
    slaveof <masterip> <masterport>
    
    #master服务密码
    masterauth <master-password>
    
    #设置redis链接密码.客户端链接Reids时需要通过AUTH <password>指定,默认关闭
    requirepass foobared
    
    #设置同一时间内最大客户端连接数,默认没有限制.是否跟ulimit命令有关系
    maxclients 128
    
    #为Redis指定最大内存限制.达到最大内存后,开始清理已到期或即将到期的key.注意linux的swap分区.
    maxmemory <bytes>
    
    #是否在每次更新操作后进行日志记录,Redis在默认情况下是异步的把数据写入磁盘,如果不开启,可能会在断电时导致一段时间内的数据丢失.因为 redis本身同步数据文件是按上面save条件来同步的,所以有的数据会在一段时间内只存在于内存中.默认为no
    appendonly no
    
    #更新日志文件名，默认为appendonly.aof
    appendfilename appendonly.aof
    
    #日志更新条件
    #no:等操作系统进行数据缓存同步到磁盘(快)
    #always:每次更新操作后手动调用fsync()将数据写到磁盘(慢,安全)
    #everysec:表示每秒同步一次(折中,默认值)
    appendfsync everysec
    
    #是否启用虚拟内存机制,默认值为no
    vm-enabled no
    
    #虚拟内存文件路径,默认值为/tmp/redis.swap,不可多个Redis实例共享
    vm-swap-file /tmp/redis.swap
    
    #将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0
    vm-max-memory 0
    
    #Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值
    vm-page-size 32
    
    #设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。
    vm-pages 134217728
    
    #设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4
    vm-max-threads 4
    
    #设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
    glueoutputbuf yes
    
    #指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
    hash-max-zipmap-entries 64
    hash-max-zipmap-value 512
    
    #指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）
    activerehashing yes
    
    #指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
    include /path/to/local.conf
    ```
    
  * 关于vm开头的配置的说明
    
    redis从2.4版本之后取消了以vm开头的配置.
    
    > The use of Virtual Memory is strongly discouraged.
    
  * 启动Redis服务端
    
    ```bash
    cd xx/redis-3.2.3/src
    ```
    
    ```bash
    ./redis-server
    ```
    
  * 启动Redis客户端
  
    ```bash
    cd xx/redis-3.2.3/src
    ```
    
    ```bash
    ./redis-cli
    ```

## Spring Boot 连接Redis

* pom.xml
  
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
      <groupId>sunshineasbefore</groupId>
      <artifactId>redis</artifactId>
      <version>0.0.1-SNAPSHOT</version>
      <packaging>jar</packaging>
      <name>redis</name>
      <description>redis</description>
      <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
      </parent>
      <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
      </properties>
      <dependencies>
        <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-devtools</artifactId>
        </dependency>
        <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-test</artifactId>
          <scope>test</scope>
        </dependency>
        <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
        <dependency>
          <groupId>com.alibaba</groupId>
          <artifactId>fastjson</artifactId>
          <version>1.2.17</version>
        </dependency>
      </dependencies>
      <build>
        <plugins>
          <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
          </plugin>
        </plugins>
      </build>
    </project>
    ```
    
* application.properties

    ```
    spring.cache.type=redis
    spring.cache.cache-names=redis-test
    # REDIS (RedisProperties)
    # Database index used by the connection factory.
    spring.redis.database=0
    # server host
    spring.redis.host=localhost
    # server password
    spring.redis.password=
    # connection port
    spring.redis.port=6379
    # pool settings ...
    spring.redis.pool.max-idle=8
    spring.redis.pool.min-idle=0
    spring.redis.pool.max-active=8
    spring.redis.pool.max-wait=-1
    spring.redis.timeout=10
    # name of Redis server
    #spring.redis.sentinel.master=
    # comma-separated list of host:port pairs
    #spring.redis.sentinel.nodes=
    ```

* redis config

    ```java
    package redis;
    
    import com.fasterxml.jackson.annotation.JsonAutoDetect;
    import com.fasterxml.jackson.annotation.PropertyAccessor;
    import com.fasterxml.jackson.databind.ObjectMapper;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.cache.CacheManager;
    import org.springframework.cache.annotation.CachingConfigurerSupport;
    import org.springframework.cache.annotation.EnableCaching;
    import org.springframework.cache.interceptor.KeyGenerator;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.data.redis.cache.RedisCacheManager;
    import org.springframework.data.redis.connection.RedisConnectionFactory;
    import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
    import org.springframework.data.redis.core.RedisTemplate;
    import org.springframework.data.redis.core.StringRedisTemplate;
    import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
    import redis.clients.jedis.JedisPoolConfig;
    
    /**
     * 描述: TODO:
     * 包名: redis.
     * 作者: barton.
     * 日期: 16-9-24.
     * 项目名称: redis
     * 版本: 1.0
     * JDK: since 1.8
     */
    @Configuration
    @EnableCaching
    public class RedisConfig extends CachingConfigurerSupport {
    
      @Value("${spring.redis.host}")
      private String host;
      @Value("${spring.redis.port}")
      private int port;
      @Value("${spring.redis.timeout}")
      private int timeout;
    
      /**
       * 对于key的生成,不能使用随机生成.
       * 第一次访问时会生成一个key值,如果redis中不存在,则将此key值和对应的value值放置在redis中,
       * 第二次访问时会再次根据一定条件生成key值,如果此key值在redis中存在,则直接取.
       * 也就是说对于一个特定的对象,它生成的key值一定是要唯一的.
       */
      @Bean
      public KeyGenerator keyGenerator() {
        return (target, method, params) -> {
          StringBuilder sb = new StringBuilder();
          sb.append(target.getClass().getName());
          sb.append(method.getName());
          for (Object obj : params) {
            sb.append(obj.toString());
          }
          return sb.toString();
        };
      }
    
      @Bean
      public CacheManager cacheManager(RedisTemplate redisTemplate) {
        RedisCacheManager cacheManager = new RedisCacheManager(redisTemplate);
        // Number of seconds before expiration. Defaults to unlimited (0)
        cacheManager.setDefaultExpiration(10); //设置key-value超时时间
        return cacheManager;
      }
    
      @Bean
      public JedisConnectionFactory redisConnectionFactory() {
        JedisConnectionFactory factory = new JedisConnectionFactory();
        factory.setHostName(host);
        factory.setPort(port);
        factory.setTimeout(timeout); //设置连接超时时间
        factory.setUsePool(true);
        factory.setPoolConfig(jedisPoolConfig());
        return factory;
      }
    
      @Bean
      public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
        StringRedisTemplate template = new StringRedisTemplate(factory);
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
      }
    
      @Bean
      public JedisPoolConfig jedisPoolConfig() {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        //    jedisPoolConfig.set ...
        return jedisPoolConfig;
      }
    }
    ```
    
* POJO对象

    ```java
    public class Student {
      public Student() {
      }
    
      public Student(String id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
      }
    
      private String id;
      private String name;
      private int age;
      
      // 省略getter setter...
    }
    ```
    
* 创建一个RedisService来获取数据

    ```java
    package redis;
    
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.cache.annotation.Cacheable;
    import org.springframework.data.redis.core.RedisTemplate;
    import org.springframework.stereotype.Service;
    
    /**
     * 描述: TODO:
     * 包名: redis.
     * 作者: barton.
     * 日期: 16-9-24.
     * 项目名称: redis
     * 版本: 1.0
     * JDK: since 1.8
     */
    @Service
    public class RedisService {
    
      @Autowired
      private RedisTemplate<String, String> redisTemplate;
    
      // keyGenerator = "keyGenerator"注意结合Spring IOC/DI的概念.
      @Cacheable(value = "studentcache", keyGenerator = "keyGenerator")
      public Student getStudent(String id, String name, int age) {
        System.out.println("如果没有缓存,则会输出这一行内容!");
        return new Student(id, name, age);
      }
    }
    ```
    
* 使用JUnit Test测试getStudent方法

    ```java
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
    
    /**
     * 描述: TODO:
     * 包名: redis.
     * 作者: barton.
     * 日期: 16-9-24.
     * 项目名称: redis
     * 版本: 1.0
     * JDK: since 1.8
     */
    @RunWith(SpringJUnit4ClassRunner.class)
    @SpringBootTest(classes = {RedisApplication.class})
    public class RedisServiceTest {
    
      @Autowired
      private RedisService service;
    
      @Test
      public void testGetStudent() {
        System.out.println("第一次:");
        System.out.println(service.getStudent("1", "barton", 22));
        System.out.println("第二次:");
        System.out.println(service.getStudent("1", "barton", 22));
      }
    }
    ```
    
* 输出结果
    * 在首次运行时会输出:
    
    ```
    第一次:
    如果没有缓存,则会输出这一行内容!
    redis.Student@6c5747db
    第二次:
    redis.Student@ba4f370
    ```
    
    * 再次运行时会输出:
    
    ```
    第一次:
    redis.Student@1ffd0114
    第二次:
    redis.Student@b3857e2
    ```
    
    * 解释:
    再次运行时,redis中已经存在该对象的key了.所以两次都是从缓存中取得value值.
    
    如果将redis停止后,再启动,其运行结果同再次运行时输出的结果.原因,redis会根据配置文件的相关配置将value值进行持久化,而不仅仅是存放在内存中,断电后就没有了.