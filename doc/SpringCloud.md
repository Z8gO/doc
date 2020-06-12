######张煌 zhanghuang@dcits.com

#Spring Cloud 学习笔记
######学习参考：
>  
csdn:  https://blog.csdn.net/u012702547/article/details/78717512  
github:  https://github.com/kswapd/spring-cloud-arch.git


######基本概念：
>服务发现——Netflix Eureka  
客服端负载均衡——Netflix Ribbon  
断路器——Netflix Hystrix  
服务调用--Feign  
服务网关——Netflix Zuul  
分布式配置——Spring Cloud Config  




##Eureka
>资料地址： https://mp.weixin.qq.com/s/K-WDRVLh-AFda_g7ga4iwA  

####Eureka Server
######配置文件
>  
 server.port=1111  
 eureka.instance.hostname=localhost   
 eureka.client.register-with-eureka=false  
 eureka.client.fetch-registry=false  
 eureka.client.service-url.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/

    说明：
        eureka.client.register-with-eureka=false   
        由于我们目前创建的应用是一个服务注册中心，而不是普通的应用，默认情况下，这个应用会向注册中心（也是它自己）注册它自己，设置为false表示禁止这种默认行为
        
       eureka.client.fetch-registry=false  
       表示不去检索其他的服务，因为服务注册中心本身的职责就是维护服务实例，它也不需要去检索其他服务
       
       eureka.client.service-url.defaultZone=http://ip1:port1/eureka/,http://ip2:port2/eureka/
       若需要配置多个注册中心，可以以上面的方式配置。以“,” 分隔

######启动一个服务注册中心的方式很简单，就是在Spring Boot的入口类上添加一个@EnableEurekaServer注解
    @SpringBootApplication
    @EnableEurekaServer
    public class Application {
    	public static void main(String[] args) {
		  SpringApplication.run(Application.class, args);
    	}
    }


<div STYLE="page-break-after: always;"></div>
####Eureka Client
***

######在配置文件中，配置服务名和<font color='red'>注册中心地址</font>

>spring.application.name=hello-service  
eureka.client.service-url.defaultZone=http://localhost:1111/eureka


######在Spring Boot的入口函数处，通过添加@EnableDiscoveryClient注解来激活Eureka中的DiscoveryClient实现。
    @EnableDiscoveryClient
    @SpringBootApplication
    public class ProviderApplication {
      public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
      }
    }
    
 

####服务的发现与消费
启动入口类上我们需要做两件事：

>1.亮明Eureka客户端身份  
 在入口类上添加@EnableDiscoveryClient注解,表示该应用是一个Eureka客户端应用，这样该应用就自动具备了发现服务的能力。  
2、提供RestTemplate的Bean  
RestTemplate可以帮助我们发起一个GET或者POST请求，这个我们在后文会详细解释，这里我们只需要提供一个RestTemplate  Bean就可以了，在提供Bean的同时，添加@LoadBalanced注解，表示开启客户端负载均衡（这个应该要在pom中添加spring-cloud-starter-ribbon依赖） 

    @SpringBootApplication
    @EnableDiscoveryClient
    public class RibbonConsumerApplication {
      public static void main(String[] args) {
        SpringApplication.run(RibbonConsumerApplication.class, args);
      }
      @LoadBalanced
      @Bean
      RestTemplate restTemplate() {
        return new RestTemplate();
      }
    }


Controller:

>说明：通过注入的restTemplate来实现对HELLO-SERVICE服务提供的/hello接口进行调用

    @RestController
    public class ConsumerController {
      @Autowired
      RestTemplate restTemplate;
      @RequestMapping(value = "/ribbon-consumer",method = RequestMethod.GET)
      public String helloController() {
          return restTemplate.getForEntity("http://HELLO-SERVICE/hello", String.class).getBody();
      }
    }

  

1、通过上面的例子，可以了解到服务的发现与消费  
2、Spring RestTemplate 的学习参考：
  
<a href="https://mp.weixin.qq.com/s/dN3ftqYspBGYVa4JqIKdiQ">Spring RestTemplate中几种常见的请求方式</a>  
<a href="https://mp.weixin.qq.com/s/uvJDmN2f9y3EEI6A3ss_aQ">RestTemplate的逆袭之路，从发送请求到负载均衡</a>  


>1、需要了解， REST 请求的一些思想  
2、RstTemplate中的几个常用的方法怎么去用。  
>> GET : getForEntity 、getForObject  
POST : postForEntity、postForObject、postForLocation  
DELETE: delete  

>3、RestTemplate上面加@LoadBalanced注解后，当得到一个请求后，都会去做哪些事情才会实现客户端负载均衡？大致的过程如下：  
>>①当一个被@LoadBalanced注解修饰的RestTemplate对象向外发起HTTP请求时，会被LoadBalancerInterceptor类的intercept方法拦截，在这个方法中直接通过getHost方法就可以获取到服务名。  
②从负载均衡服务器中挑选出一个具体的服务实例，执行  http://服务名/hello 到  http://域名/hello  的转换，然后进行请求

	


***Ribbon***  
3、Ribbon是一个基于HTTP和TCP的客户端负载均衡器，当我们将Ribbon和Eureka一起使用时，Ribbon会从Eureka注册中心去获取服务端列表，然后进行轮询访问以到达负载均衡的作用，客户端负载均衡中也需要心跳机制去维护服务端清单的有效性，当然这个过程需要配合服务注册中心一起完成。  
对RestTemplate使用 @LoadBalanced 注解后，如果本服务有多个实例向注册中心注册成功，服务消费者不会去关注服务有多少个实例，只关注服务提供者向注册中心注册的服务名称，每次请求会以轮询访问的方式去调用服务接口以到达负载均衡的作用。 

<br/>
####搭建高可用注册中心

SpringBoot项目中默认是使用application.properties，  
在原Eureka Server的项目中添加两个配置文件application-peer1.properties和application-peer2.properties


######application-peer1.properties:
>spring.application.name=eureka-server  
server.port=1111  
eureka.instance.hostname=peer1  
eureka.client.register-with-eureka=false  
eureka.client.fetch-registry=false  
eureka.client.service-url.defaultZone=http://peer2:1112/eureka/  

######application-peer2.properties:

>spring.application.name=eureka-server  
server.port=1112  
eureka.instance.hostname=peer2  
eureka.client.register-with-eureka=false  
eureka.client.fetch-registry=false  
eureka.client.service-url.defaultZone=http://peer1:1111/eureka/ 

说明：peer1、peer2是指机器可识别的机器别名或者IP地址。若需要使用别名，需要在hosts文件中指定，<font color='red'>通过配置文件中的最后一项配置可以看出，各个注册中心可以通过互相注册来达到服务信息共享的目的。</font>

<br/>
>启动时使用下面命令，来启动多个注册中心。  
java -jar eureka-server-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer1  
java -jar eureka-server-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer2



<br/>

####Eureka中的核心概念

Eureka服务治理体系中涉及到三个核心概念：服务注册中心、服务提供者以及服务消费者。

服务提供者：  只要支持Eureka通信机制，都可以作为服务提供者，向注册中心注册。

服务注册：  服务提供者在启动的时候会通过发送REST请求将自己注册到Eureka Server上，同时还携带了自身服务的一些元数据信息。Eureka Server在接收到这个REST请求之后，将元数据信息存储在一个双层结构的Map集合中，第一层的key是服务名，第二层的key是具体服务的实例名，在服务注册时，需要确认一下eureka.client.register-with-eureka=true配置是否正确，该值默认就为true，表示启动注册操作，如果设置为false则不会启动注册操作。   

服务下线：  
服务提供者在运行的过程中可能会发生关闭或者重启，当服务进行正常关闭时，它会触发一个服务下线的REST请求给Eureka Server，告诉服务注册中心我要下线了，服务注册中心收到请求之后，将该服务状态置为DOWN，表示服务已下线，并将该事件传播出去，这样就可以避免服务消费者调用了一个已经下线的服务提供者了。

失效剔除：  
正常的服务下线发生流程有一个前提那就是服务正常关闭,但是在实际运行中服务有可能没有正常关闭，比如系统故障、网络故障等原因导致服务提供者非正常下线，那么这个时候对于已经下线的服务Eureka采用了定时清除：Eureka Server在启动的时候会创建一个定时任务，每隔60秒就去将当前服务提供者列表中超过90秒还没续约的服务剔除出去，通过这种方式来避免服务消费者调用了一个无效的服务。


服务同步：  
有两个服务注册中心，地址分别是http://localhost:1111 和 http://localhost:1112， <font color='red'>在两个服务注册中也相互注册的情况下（参考搭建高可用注册中心）</font>，
如果有两个服务提供者，地址分别是http://localhost:8080 和 http://localhost:8081， 然后我将8080这个服务提供者注册到1111这个注册中心上去，将8081这个服务提供者注册到1112这个注册中心上去，此时我在服务消费者中如果只向1111这个注册中心去查找服务提供者，那么就能获取到8080  和 8081这两个的服务。


服务续约：  
在注册完服务之后，服务提供者会维护一个心跳来不停的告诉Eureka Server：“我还在运行”，以防止Eureka Server将该服务实例从服务列表中剔除，这个动作称之为服务续约，和服务续约相关的属性有两个，如下：

    eureka.instance.lease-expiration-duration-in-seconds=90  
    eureka.instance.lease-renewal-interval-in-seconds=30
    
上面两个都是默认配置，第一个配置用来定义服务失效时间，默认为90秒，第二个用来定义服务续约的间隔时间，默认为30秒。


服务消费者：  
消费者主要是从服务注册中心获取服务列表，拿到服务提供者的列表之后，服务消费者就知道去哪里调用它所需要的服务了。  
当我们启动服务消费者的时候，它会发送一个REST请求给服务注册中心来获取服务注册中心上面的服务提供者列表，而Eureka Server上则会维护一份只读的服务清单来返回给客户端，这个服务清单并不是实时数据，而是一份缓存数据，默认30秒更新一次，如果想要修改清单更新的时间间隔，可以通过eureka.client.registry-fetch-interval-seconds=30来修改，单位为秒(注意这个修改是在eureka-server上来修改)。另一方面，我们的服务消费端要确保具有获取服务提供者的能力，此时要确保eureka.client.fetch-registry=true这个配置为true。


自我保护：  
Eureka Server在运行期间会去统计心跳失败比例在15分钟之内是否低于85%，如果低于85%，Eureka Server会将这些实例保护起来，让这些实例不会过期，但是在保护期内如果服务刚好这个服务提供者非正常下线了，此时服务消费者就会拿到一个无效的服务实例，此时会调用失败，对于这个问题需要服务消费者端要有一些容错机制，如重试，断路器等。我们在单机测试的时候很容易满足心跳失败比例在15分钟之内低于85%，这个时候就会触发Eureka的保护机制，一旦开启了保护机制，则服务注册中心维护的服务实例就不是那么准确了，此时我们可以使用eureka.server.enable-self-preservation=false来关闭保护机制，这样可以确保注册中心中不可用的实例被及时的剔除。



**从上面的内容基本上对 Eureka 和 ribbon 已经有一个理解了**


<div STYLE="page-break-after: always;"></div>
##Hystrix 
**断路器**

>  看了一些文章，说一说我的理解，所谓断路器，就是说在微服务中，一般会有服务之间互相调用的情况，如果某一个服务挂掉了之后不采取任何措施，那么这个服务的消费者在调用这个服务时会出现请求失败或者超时的情况，高并发的情况下，后果无法想象，这个时候断路器的作用就很重要了。  

#####服务消费者中加入断路器

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix</artifactId>
    </dependency>



#####修改服务消费者启动入口类
引入hystrix之后，我们需要在入口类上通过@EnableCircuitBreaker开启断路器功能，如下:


    @EnableCircuitBreaker
    @SpringBootApplication
    @EnableDiscoveryClient
    public class RibbonConsumerApplication {

        public static void main(String[] args) {
            SpringApplication.run(RibbonConsumerApplication.class, args);
        }
        @LoadBalanced
        @Bean
        RestTemplate restTemplate() {
            return new RestTemplate();
        }
    }

也可以使用一个名为@SpringCloudApplication 的注解代替这三个注解，@SpringCloudApplication 注解的定义如下：

    @Target({ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Inherited
    @SpringBootApplication
    @EnableDiscoveryClient
    @EnableCircuitBreaker
    public @interface SpringCloudApplication {}
    
    

#####Service的中使用
    @Service
    public class HelloService {
     @Autowired
     private RestTemplate restTemplate;
	
     @HystrixCommand(fallbackMethod = "error")
     public String hello() {
       ResponseEntity<String> responseEntity = restTemplate.getForEntity("http://HELLO-SERVICE/hello", String.class);
       return responseEntity.getBody();
     }
		
     
     public String error() {
       return "error";
     }
    }


**说明：**
> 1.error方法是一个请求失败时回调的方法。  
2.在hello方法上通过@HystrixCommand注解来指定请求失败时回调的方法


上面讲到的是使用注解的方式使用断路器，另外还有一种方式是通过继承的方式使用。

https://mp.weixin.qq.com/s/r05WF7WP3Qd7As_fTBkxaw  



######Hystrix的服务降级与异常处理
在Service中：

    @HystrixCommand(fallbackMethod = "testBackup")
    public Book test() {
        return restTemplate.getForObject("http://HELLO-SERVICE/getbook1", Book.class);
    }

    @HystrixCommand(fallbackMethod = "error")
    public Book testBackup() {
        //TODO  发起某个网络请求（可能失败）
        return null;
    }
    public Book error(Throwable throwable) {
      System.out.println(throwable.getMessage());
      return new Book();
    }


如上，当test 执行出现问题时，先做一个降级处理，去调用testBackup 再去执行一个动作，当这个降级的操作也出现问题时，再去执行错误方法。


######通过注解开启缓存
可以通过注解来开启缓存，和缓存相关的注解一共有三个，分别是@CacheResult、@CacheKey和@CacheRemove  
详细参考： https://mp.weixin.qq.com/s/YpWODLrwzFXUQRtIAHLF3Q



#####Hystrix的请求合并
核心类：HystrixCollapser

>非注解方式实现请求合并先不做学习，但是非注解方式有利于更清晰的理解代码，等后面做详细的学习时再研究，此部分以使用上手为目的。

######通过注解更优雅的实现请求合并


在Service 中进行修改：

    @HystrixCollapser(batchMethod = "test11",collapserProperties = {@HystrixProperty(name ="timerDelayInMilliseconds",value = "100")})
    public Future<Book> test10(Long id) {
        return null;
    }

    @HystrixCommand
    public List<Book> test11(List<Long> ids) {
        System.out.println("test9---------"+ids+"Thread.currentThread().getName():" + Thread.currentThread().getName());
        Book[] books = restTemplate.getForObject("http://HELLO-SERVICE/getbook6?ids={1}", Book[].class, StringUtils.join(ids, ","));
        return Arrays.asList(books);
    }


在test10方法上添加@HystrixCollapser注解实现请求合并，用batchMethod属性指明请求合并后的处理方法，collapserProperties属性指定其他属性。

以上可以看出，请求的合并就是在高并发的情况下，当同时有多个相同的请求时，请求合并可以将这些请求的参数封装到一个集合中，再以集合的为参数进行服务端的操作。有点就是节省了网络带宽和线程池，缺点就是可能请求的时间较长。但是很多场景下这种时间的延迟都是微不足道的，高并发也是请求合并的一个非常重要的场景。






<div STYLE="page-break-after: always;"></div>
##Feign
Declarative REST Client: Feign（一个声明式web 服务调用服务）。  
Feign整合了Ribbon和Hystrix。  


入口类main方法增加注解表示开启feign
>@EnableFeignClients


编写Service接口 HelloService，示例如下：

    @FeignClient("hello-service")
    public interface HelloService {
        @RequestMapping("/hello")
        String hello(@RequestHeader("name") String name);
    }

>解释：  
@FeignClient("hello-service") 表示去 hello-service 这个服务中进行请求。
@RequestMapping("/hello") 表示去 hello-service/hello 中进行请求

请注意参数必须声明参数的数据来源，否则会报错。


|  声明  | 说明  |
|  ----  | ----  |
| @RequestParam("name")  String name   | 获取表单形式携带参数请求  |
| @RequestHeader("name")  String name    | 获取表单形式携带参数请求，比上面的范围更大一些 |
| @RequestBody Book book     | 获取以对象形式携带参数的请求 |



Controller 中正常调用，示例如下：

    @RestController
    public class FeignConsumerController {
        @Autowired
        HelloService helloService;

        @RequestMapping("/hello")
        public String hello() {
            return helloService.hello();
        }
    }
    
 ######Feign配置详解
 
 参考： <a href="https://mp.weixin.qq.com/s/R5PX6fJhJ-Yi7YN7p32G1Q" target="_blank">Feign配置详解</a>  
 
 
 
<div STYLE="page-break-after: always;"></div>
##Zuul
#####服务网关

最简项目的依赖配置为：	

    <dependencies>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter</artifactId>
      </dependency>
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-zuul</artifactId>
      </dependency>
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-eureka</artifactId>
      </dependency>
	</dependencies>


入口类增加注解：

	@EnableZuulProxy



######配置路由规则

application.properties文件中的配置可以分为两部分，一部分是Zuul应用的基础信息，还有一部分则是路由规则，如下：


> 基础信息配置 
>>spring.application.name=api-gateway   
>>server.port=2006  

>路由规则配置，api-a是路由的名字，一组path和serviceId映射关系的路由名要相同。
>>zuul.routes.api-a.path=/api-a/\*\*  
  zuul.routes.api-a.serviceId=feign-consumer

>API网关也将作为一个服务注册到eureka-server上
>>eureka.client.service-url.defaultZone=http://localhost:1111/eureka/



<br/>
以上配置说明：
[comment]:\*是为了转译配置为zuul.routes.api-a.path=/api-a/** 

>如果请求 http://localhost:2006/api-a/hello1 接口则相当于请求 http://localhost:2005/hello1 (这里feign-consumer的地址为http://localhost:2005 )，在路由规则中配置的api-a是路由的名字，可以任意定义，但是一组path和serviceId映射关系的路由名要相同。  
配置细节： zuul.routes.api-a.path=/api-a/\*\*  


>>\*\*（两个星号）表示匹配的字符长度不限制，如匹配/feign-consumer/aaa,feign-consumer/bbb,/feign-consumer/ccc等，也可以匹配/feign-consumer/a/b/c  
  \*（一个星号）表示匹配任意数量的字符，如匹配/feign-consumer/aaa,feign-consumer/bbb,/feign-consumer/ccc等，无法匹配/feign-consumer/a/b/c  
  ?	匹配任意单个字符，如匹配/feign-consumer/a,/feign-consumer/b,/feign-consumer/c等



#####请求过滤

对一些请求或者请求的参数进行过滤，达到对请求的过滤。  

过滤器编写：

> 1、继承ZuulFilter   
重写 run方法实现过滤的细节。

Demo如下：

    public class PermisFilter extends ZuulFilter {
        @Override
        public String filterType() {
            return "pre";
        }

        @Override
        public int filterOrder() {
            return 0;
        }

        @Override
        public boolean shouldFilter() {
            return true;
        }

        @Override
        public Object run() {
            RequestContext ctx = RequestContext.getCurrentContext();
            HttpServletRequest request = ctx.getRequest();
            String login = request.getParameter("login");
            if (login == null) {
                ctx.setSendZuulResponse(false);
                ctx.setResponseStatusCode(401);
                ctx.addZuulResponseHeader("content-type","text/html;charset=utf-8");
                ctx.setResponseBody("非法访问");
            }
            return null;
        }
    }


配置细节：

######不对外部提供请求：
> zuul.ignored-services=hello-service

在配置后当从路由请求时，会直接被 Zuul Server 过滤掉。



######让路由的匹配按照配置文件的配置顺序依次匹配

此时则不能使用application.properties(解析后是以Map存储的，是无序的），必须使用 application.yml文件进行配置。

    spring:
      application:
        name: api-gateway
    server:
      port: 2006
    zuul:
      routes:
        feign-consumer-hello:
          path: /feign-consumer/hello/**
          serviceId: feign-consumer-hello
        feign-consumer:
          path: /feign-consumer/**
          serviceId: feign-consumer
    eureka:
      client:
        service-url:
          defaultZone: http://localhost:1111/eureka/



######异常处理
参考： https://mp.weixin.qq.com/s/Y73q8XrVoEuVqCNpexiVHg

重点：自定义异常

    @Component
    public class MyErrorAttribute extends DefaultErrorAttributes {
        @Override
        public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes, boolean includeStackTrace) {
            Map<String, Object> result = super.getErrorAttributes(requestAttributes, includeStackTrace);
            result.put("status", 222);
            result.put("error", "error");
            result.put("exception", "exception");
            result.put("message", "message");
            return result;
        }
    }


##Spring Cloud Config
######分布式配置

#####Config-Server:  
流程：
1、将配置文件上传至版本管理工具或者简单的文件访问服务器，当然最好是版本管理工具服务器。配置文件可以根据不同的环境上传多种文件。
   
	    默认配置：helloservice.properties  
    	测试环境：helloservice-test.properties   
    	开发环境：helloservice-dev.properties  
    	生产环境：helloservice-prod.properties

2、在项目中配置服务器地址、管理文件路径、用户名、密码相关。  
3、启动项目，可以访问文件信息。  访问方式   ip:端口/{application}/{profile}/{label}。其中application表示配置的服务的名字（例如 hello-service），profile表示环境，我们有dev、test、prod还有默认，label表示分支，默认我们都是放在master分支上

需要注意的细节：
>入口类主方法 增加注解： @EnableConfigServer  
主要配置：  
spring.application.name=config-server  
server.port=6100  
spring.cloud.config.server.git.uri=https://gitee.com/Z8gO/conf-repo.git
spring.cloud.config.server.git.search-paths=config-repo  
spring.cloud.config.server.git.username=username  
spring.cloud.config.server.git.password=password  
eureka.client.service-url.defaultZone=http://localhost:1111/eureka/  



#####Config-Client:  
1、在bootstrap.properties 中配置 服务名称，需要与上传的文件中的服务名称相同。  
2、在类中可以用常用的方式进行配置属性的引用。

            @Value("${sang}")
            String sang;



>主要配置：  
spring.application.name=app  
server.port=6200  
eureka.client.service-url.defaultZone=http://localhost:1111/eureka  
spring.cloud.config.profile=dev  
spring.cloud.config.label=master  
spring.cloud.config.discovery.enabled=true  
spring.cloud.config.discovery.service-id=config-server  
spring.cloud.config.fail-fast=true  


***   
至此Spring Cloud 五大组件的入门级别学习完成
***

一些有用的思想：

>在模块拆分为多个服务后，数据库层面也会得到拆分。功能之间的相互依赖会变为服务之间相互调用，在此中应该尽可能的做到服务之间的 “无事务”







<br/><br/><br/><br/><br/><br/><br/><br/>
<br/><br/><br/><br/><br/><br/><br/><br/>
<br/><br/><br/><br/><br/><br/><br/><br/>
