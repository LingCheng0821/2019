# Spring Cloud

## 1. 微服务概述

### 1.1 什么是微服务

微服务的概念源于2014年3月Martin Fowler所写的一篇文章“Microservices”。

就目前而言，对于微服务业界并没有一个统一的、标准的定义。

但通常而言，微服务是一种架构风格。

微服务架构风格是**将传统的一站式应用，根据业务拆分成一个一个的服务，彻底地松耦合**，每一个微服务提供单个业务功能的服务，**每种应用程序都运行在自己的进程中，并与轻量级机制（通常是基于HTTP协议的RESTful API）进行通信。**这些服务是围绕业务功能构建的，可以通过全自动部署机制独立部署。

一个简单的微服务应用例子：航班预订应用，将航班预订应用划分为预订航班、时间表查询、计算票价、分配座位、管理奖励、更新客户、调整库存七个微服务实施。

从技术角度看就是一种小而独立的处理过程，类似进程概念，能够自行单独启动或销毁，拥有自己独立的数据库。

### 1.2 优缺点
**微服务架构的好处**

​      1.每个服务都比较简单，只关注于一个业务功能，容易开发、理解和维护。

​      2.微服务架构方式是松耦合的，可以提供更高的灵活性，可以独立扩展。

​      3.每个微服务可由不同团队独立开发，互不影响，加快推出市场的速度。

​      4.微服务架构模式是每个微服务独立的部署，保持系统其他部分的可用性和稳定性。

**微服务架构的不足**

​       1.运维开销及成本增加：以前可能只需部署至一小片集群，而微服务架构可能变成十个独立的服务；

​       2.分布式系统的复杂性：例如网络延迟、容错性、消息序列化、不可靠的网络，版本化等。

​       3.服务通信成本；

​       4.可测性的挑战：在动态环境下服务间的交互会产生非常微妙的行为，难以可视化及全面测试。



### 1.3 springCloud和Dubbo有哪些区别

- 总体架构：二者模式接近，都需要服务提供方，注册中心，服务消费方。

![dubbo架构](/Users/laura/Desktop/未命名文件夹/dubbo架构.jpg)



​	Provider：暴露服务的提供方，可以通过 jar 或者容器的方式启动服务。

​	Consumer：调用远程服务的服务消费方。

​	Registry：服务注册中心和发现中心。

​	Monitor：统计服务和调用次数，调用时间监控中心。

​	Container：服务运行的容器。

![springcloud架构](/Users/laura/Desktop/未命名文件夹/springcloud架构.jpg)



​	Service Provider： 暴露服务的提供方。

​	Service Consumer：调用远程服务的服务消费方。

​	EureKa Server： 服务注册中心和服务发现中心。

- Dubbo 只是实现了服务治理，而 Spring Cloud 子项目分别覆盖了微服务架构下的众多部件，服务治理只是其中的一个方面。Dubbo 提供了各种 Filter，对于上述中“无”的要素，可以通过扩展 Filter 来完善。

![核心要素](/Users/laura/Desktop/未命名文件夹/核心要素.jpg)

- 通信协议

  Dubbo 使用 RPC 通讯协议；Spring Cloud 使用 HTTP 协议的 REST API。

- 性能比较

  Dubbo 支持各种通信协议，而且消费方和服务方使用长链接方式交互，通信速度上略胜 Spring Cloud。

- 服务依赖

  - 服务提供方与消费方通过接口的方式依赖，因此需要为每个微服务定义各自的 Interface 接口；

  - 服务提供方和服务消费方通过 Json 方式交互，只需要定义好相关 Json 字段即可，无接口依赖。

- 后续改进

  - dubbo的改进是通过dubbofilter，很多东西没有，需要自己继承，如监控，如日志，如限流，如追踪
  - springcloud自己带了很多监控、限流措施，进行适当改造，就是ServletFilter

### 1.4 Springboot & Springcloud

SpringBoot专注于快速方便的开发单个个体微服务，**SpringCloud是关注全局的微服务协调整理治理框架**。

Springboot简化创建产品级的 Spring 应用和服务，简化了配置文件，使用嵌入式web服务器。Springcloud，它将SpringBoot开发的一个个单体微服务整合并管理起来，为各个微服务之间提供，**配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等集成服务**;

SpringBoot可以离开SpringCloud独立使用开发项目，但是SpringCloud离不开SpringBoot，属于依赖的关系.

SpringBoot专注于快速、方便的开发单个微服务个体，SpringCloud关注全局的服务治理框架。

### 1.5  微服务技术栈有哪些

- 微服务条目落地技术备注服务开发Springboot、Spring、SpringMVC

- 服务配置与管理Netflix公司的Archaius、阿里的Diamond等

- 服务注册与发现Eureka、Consul、Zookeeper等
- 服务调用Rest、RPC、gRPC 服务熔断器Hystrix、Envoy等
- 负载均衡Ribbon、Nginx等服务接口调用(客户端调用服务的简化工具)Feign等
- 消息队列Kafka、RabbitMQ、ActiveMQ等
- 服务配置中心管理SpringCloudConfig、Chef等
- 服务路由（API网关）Zuul等
- 服务监控Zabbix、Nagios、Metrics、Spectator等
- 全链路追踪Zipkin，Brave、Dapper等
- 服务部署Docker、OpenStack、Kubernetes等
- 数据流操作开发包SpringCloud Stream（封装与Redis,Rabbit、Kafka等发送接收消息）
- 事件消息总线Spring Cloud Bus......

### 1.6 eureka 和 zk 的区别

**Eureka 和 Zookeeper都可以提供服务注册与发现的功能。为什么Eureka比ZooKeeper更适合做服务发现？**

著名的CAP理论指出，一个分布式系统不可能同时满足C（一致性）、A（可用性）、和P（分区容错性）。由于分区容错性P在分布式系统中必须要保证的，因此我们只能在A和C之间进行权衡。当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，但是不能接受服务直接down掉不可用。也就是说，**服务注册功能对可用性的要求要高于一致性。Zookeeper保证的是CP， Eureka则是AP，(RDBMS:CA)。**

**一句话：某时刻某一个微服务不可用了，eureka不会立刻清理，依旧会对该微服务的信息进行保存。**

- **Zoopkeeper保证CP，Eureka保证AP**

  - 如果服务器宕机，Eureka不会有类似于ZooKeeper的选举leader的过程；Eureka各个节点都是平等的，Eureka的客户端在向某个Eureka注册时如果发现连接失败，则会自动切换至其他的节点，只要有一台Eureka还在，就能保证注册服务可用（保证可用性），只不过查到的信息可能不是最新的（不保证一致性）。
  - Zoopkeeper 在 当master节点因网路故障与其他节点失去联系时，剩余的节点会重新进行leader选举。问题在于，选举leader的时间太长，30~120s，且选举期间整个zk集群是都是不可用的，这就导致在选举期间注册服务瘫痪。

- 自我保护机制 

  ​	除此之外，Eureka还有一种自我保护机制，如果Eureka服务节点在短时间里丢失了大量的心跳连接，那么这个Eureka节点会进入”自我保护模式“，同时保留那些“心跳死亡“的服务注册信息不过期。此时，这个Eureka节点对于新的服务还能提供注册服务，当网络故障恢复后，这个Eureka节点会退出”自我保护模式“，当前实例新的注册信息会被同步到其它节点中；

  因此，Eureka可以很好的应对因网络故障导致节点失去联系的情况，而不会像zookeeper那样使整个注册服务瘫痪。


### 1.7 ribbon 和 feign 区别

**Ribbon+RestTemplate** 调用 REST  服务；

**Feign** 采用的是基于接口的注解； 整合了ribbon，具有负载均衡的能力；整合了Hystrix，具有熔断的能力。



## 2. 五大功能

### 2.1 注册发现

- 注册中心 

  - pom 

    ```xml
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    ```

  - 配置文件

    ```properties
    // 1. 修改端口
    // 2. 设置eureka服务端的实例名称：eureka.instance.hostname
    // 3. 不向注册中心注册自己：eureka.client.register-with-eureka=false
    // 4. 自己端就是注册中心:eureka.client.fetch-registry=false
    // 5. 设置交互地址：eureka.client.service-url.defaultZone: 
        http://eureka7001.com:7001/eureka/,
        http://eureka7002.com:7002/eureka/, http://eureka7003.com:7003/eureka/  
    ```

  - 主启动类添加注解 **@EnableEurekaServer** 

    ```java
    @EnableEurekaServer     //表示是 EurekaServer服务器端启动类,接受其它微服务注册进来。
    ```

- 服务提供

  - pom

    ```xml
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    ```

  - 配置文件

    ```properties
    eureka.client.service-url.defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
    ```

  - 启动类 **@EnableEurekaClient** 

    ```java 
    @EnableEurekaClient     //本服务启动后会自动注册进eureka服务中
    ```

- 消费者

  - 配置类 : 添加 **RestTemplate**，并加入 容器管理；
  - 服务调用：**restTemplate.postForObject**(URL, PARAM, RETURN CLASS);

### 2.2 Ribbon使用

Ribbon 是一个 **负载均衡客户端**。

- 修改客户端：

  - 添加 POM 依赖：结合 Eureka 一起用；

    ```xml
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    ```

  - 配置文件：添加 eureka 服务注册地址：**eureka. client.service-url.defaultZone**

  ```java
  private static final String REST_URL_PREFIX = "http://MICROSERVICECLOUD-DEPT";// 1. 配置类 RestTemplate 开启负载均衡
  @LoadBalanced

  // 2. 主启动类
  @EnableEurekaClient

  // 3. 服务调用：服务调用
  private static final String REST_URL_PREFIX = "http://MICROSERVICECLOUD-DEPT";
  ```

### 3.3 Feign使用

```java 
// 1. 服务接口， 来指定调用哪个服务
@FeignClient(value = "MICROSERVICECLOUD-DEPT")

// 2. 消费者 Controller 不调用 RestTemple, 调用上一步 服务接口
@Autowired
private DeptClientService service = null;

// 3. 消费者 启动类 ServiceFeignApplication 
@EnableEurekaClient
@EnableFeignClients(basePackages= {"com.fang.springcloud"})
@ComponentScan("com.fang.springcloud")
```







​	

## 3. 服务消费

### 3.1 配置类

```java
@Configuration
public class ConfigBean{
    // RestTemplate提供了多种便捷访问远程Http服务的方法， 
    // 是一种简单便捷的访问restful服务模板类，是Spring提供的用于访问Rest服务的客户端模板工具集
    @Bean
    public RestTemplate getRestTemplate(){
         return new RestTemplate();
    }
}
```

### 3.2 消费

```java
@RestController
public class DeptController_Consumer{
    private static final String REST_URL = "http://localhost:8001";
    
    @Autowired
    private RestTemplate restTemplate;
    
    @RequestMapping(value="/consumer/dept/add")
    public boolean add(Dept dept){
         return restTemplate.postForObject(REST_URL+"/dept/add", dept, Boolean.class);
    }
    
    @RequestMapping(value="/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id){
         return restTemplate.getForObject(REST_URL+"/dept/get/"+id, Dept.class);
    }
    
    @RequestMapping(value="/consumer/dept/list")
    public List<Dept> list(){
         return restTemplate.getForObject(REST+"/dept/list", List.class);
    }   
}
 
```



## 4. Eureka-服务注册与发现

### 4.1 架构

Eureka Server 提供服务注册和发现
Service Provider服务提供方将自身服务注册到Eureka，从而使服务消费方能够找到
Service Consumer服务消费方从Eureka获取注册服务列表，从而能够消费服务

### 3.2 注册中心

- pom

```xml
<artifactId>spring-cloud-starter-eureka-server</artifactId>
```

- 配置

```yaml
server: 
  port: 7001
 
eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
  client:
    register-with-eureka: false #false表示不向注册中心注册自己。
    fetch-registry: false #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/        
      #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
```

- **主程序：@EnableEurekaServer**

```java
@SpringBootApplication
@EnableEurekaServer//EurekaServer服务器端启动类,接受其它微服务注册进来
public class EurekaServer { ... }
```

### 3.3 服务注册

- pom

```xml
<artifactId>spring-cloud-starter-eureka</artifactId>
<artifactId>spring-cloud-starter-config</artifactId>
```

- 配置

```yaml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url: 
      defaultZone: http://localhost:7001/eureka
```

- **主程序：@EnableEurekaClient**

```java
@SpringBootApplication
@EnableEurekaClient //本服务启动后会自动注册进eureka服务中
public class DeptProvider8001_App
```

- 主机名:服务名称修改

```yaml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url: 
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: microservicecloud-dept8001   #自定义服务名称信息
    prefer-ip-address: true                   #访问路径可以显示IP地址     
```

- 微服务info内容详细信息

  - pom

  ```xml
   <artifactId>spring-boot-starter-actuator</artifactId>
  ```

  - 配置

  ```yaml
  info:
    app.name: atguigu-microservicecloud
    company.name: www.atguigu.com
    build.artifactId: $project.artifactId$
    build.version: $project.version$
  ```

  - 注册中心 pom

  ```xml
  <build>
      <finalName>microservicecloud</finalName>
      <resources>
          <resource>
              <directory>src/main/resources</directory>
              <filtering>true</filtering>
          </resource>
      </resources>
      <plugins>
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-resources-plugin</artifactId>
              <configuration>
                  <delimiters>
                      <delimit>$</delimit>
                  </delimiters>
              </configuration>
          </plugin>
      </plugins>
  </build>
  ```


### 3.4 集群

- 三台注册中心：yml

  ```yaml
  server: 
    port: 7001
  eureka: 
    instance:
      hostname: eureka7001.com #eureka服务端的实例名称
    client: 
      register-with-eureka: false     
      fetch-registry: false     
      service-url: 
        # defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
        defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  ```

- 服务提供者

  ```yaml
  eureka:
    client: #客户端注册进eureka服务列表内
      service-url: 
        defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,
        http://eureka7003.com:7003/eureka/
  ```



## 4. Ribbon负载均衡

### 4.1 概述

Ribbon 是一个 **负载均衡客户端**。可以很好的控制htt和tcp的一些行为。Feign默认集成了ribbon。

**LB，即负载均衡(Load Balance)**，在微服务或分布式集群中经常用的一种应用。负载均衡简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA。

### 4.2 修改消费者

- pom

  ```xml
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
  <artifactId>spring-cloud-starter-config</artifactId>   
  ```

- 配置文件

  ```yaml
  server:
    port: 80

  eureka:
    client:
      register-with-eureka: false
      service-url: 
        defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,
        http://eureka7003.com:7003/eureka/
  ```

- **修改配置类：@LoadBalanced**

  ```java
  @Configuration
  public class ConfigBean{
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
     return new RestTemplate();
    }
  }
  ```

- 主配置类：**@EnableEurekaClient**

  ```java
  @SpringBootApplication
  @EnableEurekaClient
  public class DeptConsumer80_App
  ```

- 客户端访问类

  ```java
  @RestController
  public class DeptController_Consumer{
    //private static final String REST_URL_PREFIX = "http://localhost:8001";
    // 三个 服务提供者 暴露为同一个服务名称：MICROSERVICECLOUD-DEPT
    private static final String REST_URL_PREFIX = "http://MICROSERVICECLOUD-DEPT";
  ```

### 4.3 现有的IRule 

```java
RoundRobinRule					// 轮循 默认
RandomRule                    	// 随机
AvailabilityFilteringRule
WeightedResponseTimeRule		// 加权
RetryRule						// 轮循的方式 重试，死掉的服务在尝试几次后，就调过该服务
BestAvailableRule				// 选择最小请求值
ZoneAvoidanceRule               // 根据server的zone区域和可用性来轮循选择。
```

### 4.4 自定义 策略

- 主启动类：**@RibbonClient**

  ```java
  @SpringBootApplication
  @EnableEurekaClient
  @RibbonClient(name = "MICROSERVICECLOUD-DEPT", configuration = MyselfRule.class)
  public static class DeptProvider8001_App 
  ```

- 自定义规则：**这个类不能 在 主启动类 的包及其子包下** 

  ```java
  @Configuration
  public class MyselfRule(){
    @Bean
    public IRule myRule(){
      return new RandomRule(); // 自已定义：extends AbstractLoadBalancerRule  模仿 源码
    }
  }
  ```



## 5. Feign 负载均衡

### 5.1 修改API

- pom

  ```xml
  <artifactId>spring-cloud-starter-feign</artifactId>
  ```

- 新增接口：**@FeignClient**

  ```java
  @FeignClient(value = "MICROSERVICECLOUD-DEPT")
  public interface DeptClientService{
    @RequestMapping(value = "/dept/get/{id}",method = RequestMethod.GET)
    public Dept get(@PathVariable("id") long id);
    @RequestMapping(value = "/dept/list",method = RequestMethod.GET)
    public List<Dept> list();
    @RequestMapping(value = "/dept/add",method = RequestMethod.POST)
    public boolean add(Dept dept);
  }
  ```

### 5.2 消费者

- pom

  ```xml
  <artifactId>spring-cloud-starter-feign</artifactId>
  ```

- 修改 Controller

  ```java
  @RestController
  public class DeptController_Feign{
    // 不是用 RestTemplate
    @Autowired
    private DeptClientService service = null;
    
    @RequestMapping(value = "/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id){
     return this.service.get(id);
    }
  }
  ```


- 主启动类

  ```java
   
  @SpringBootApplication
  @EnableEurekaClient
  @EnableFeignClients(basePackages= {"com.fang.springcloud"})
  @ComponentScan("com.fang.springcloud")
  public class DeptConsumer80_Feign_App
  ```

## 6. Hystrix 断路器 

### 6.1 概述

Hystrix是一个用于**处理分布式系统的延迟和容错**的开源库，保证在一个依赖出问题的情况下，**不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。**

“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），**向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常**，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

作用：① 服务降级； ② 服务熔断； ③ 服务限流； ④ 接近实时的监控。

### 6.2 使用方法

- ### 在ribbon使用断路器

  - 服务提供者 添加依赖

  ```xml
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
  ```

  - 服务提供者 主启动类：**@EnableHystrix  或者 @EnableCircuitBreaker** 

  ```java
  @EnableEurekaClient    //本服务启动后会自动注册进eureka服务中
  @EnableCircuitBreaker   //对hystrixR熔断机制的支持
  ```

  - 服务提供者 对该方法创建了熔断器的功能，并指定了fallbackMethod熔断方法：**@HystrixCommand

  ```java
  @HystrixCommand(fallbackMethod = "hiError")
  public String hiService(String name) {
     return restTemplate.getForObject("http://SERVICE-HI/hi?name="+name,String.class);
  }
  public String hiError(String name) {
     return "hi,"+name+",sorry,error!";
  }
  ```

- ### Feign中使用断路器

  修改 API 工程。

  - 客户端 配置文件中配置打开断路器： **feign.hystrix.enabled=true**
  - 提供服务接口

  ```java
  @FeignClient(value = "MICROSERVICECLOUD-DEPT",fallbackFactory=FallbackFactory.class)
  public interface DeptClientService{
    @RequestMapping(value = "/dept/get/{id}",method = RequestMethod.GET)
    public Dept get(@PathVariable("id") long id);
  }

  @Component//不要忘记添加，不要忘记添加
  public class FallbackFactory implements FallbackFactory<DeptClientService>{
    @Override
    public DeptClientService create(Throwable throwable) {
     return new DeptClientService() {
       @Override
       public Dept get(long id){
         return new Dept().setDeptno(id).setDname("该ID："+id+"没有没有对应的信息");
       }
     };
    }
  }
  ```

  ​

  ​






----------------------------------------------

什么是微服务？
微服务之间是如何独立通讯的
springCloud和Dubbo有哪些区别？
SpringBoot和SpringCloud，请你谈谈对他们的理解
什么是服务熔断？什么是服务降级
微服务的优缺点分别是什么？说下你在项目开发中碰到的坑
你所知道的微服务技术栈有哪些？请列举一二
eureka和zookeeper都可以提供服务注册与发现的功能，请说说两个的区别？

