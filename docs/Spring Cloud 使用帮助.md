# 服务注册与发现
### 1. 创建服务注册中心
#### pom.xml 依赖
```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
	    <groupId>org.springframework.cloud</groupId>
	    <artifactId>spring-cloud-dependencies</artifactId>
	    <version>Brixton.RELEASE</version>
	    <type>pom</type>
	    <scope>import</scope>
	</dependency>
    </dependencies>
</dependencyManagement>
```
#### Application 中增加注解: @EnableEurekaServer
> 通过@EnableEurekaServer注解启动一个服务注册中心提供给其他应用进行对话。
```
@EnableEurekaServer
@SpringBootApplication

public class Application {
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```
#### application.properties
```
server.port=1111 
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
```
> server.port : 端口  
> eureka.client.register-with-eureka : ?  
> eureka.client.fetch-registry : ?  
> eureka.client.serviceUrl.defaultZone : ?  
***
### 2. 创建服务提供方
#### pom.xml
```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
	    <groupId>org.springframework.cloud</groupId>
	    <artifactId>spring-cloud-dependencies</artifactId>
	    <version>Brixton.RELEASE</version>
	    <type>pom</type>
	    <scope>import</scope>
	</dependency>
    </dependencies>
</dependencyManagement>
```

#### Application 中增加 @EnableDiscoveryClient
> 通过加上@EnableDiscoveryClient注解，该注解能激活Eureka中的DiscoveryClient实现，才能实现Controller中对服务信息的输出。
```
@EnableDiscoveryClient
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```
#### application.properties
```
spring.application.name=compute-service
server.port=2222
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/
```
> spring.application.name :  指定微服务的名称后续在调用的时候只需要使用该名称就可以进行服务的访问。  
> server.port : 端口  
> eureka.client.serviceUrl.defaultZone : 指定服务注册中心的位置。  
### 3. Eureka Server的高可用
> Eureka Server除了单点运行之外，还可以通过运行多个实例，并进行互相注册的方式来实现高可用的部署，所以我们只需要将Eureke Server配置其他可用的serviceUrl就能实现高可用部署。
### pom.xml
> 创建application-peer1.properties，作为peer1服务中心的配置，并将serviceUrl指向peer2
```
spring.application.name=eureka-server
server.port=1111
eureka.instance.hostname=peer1
eureka.client.serviceUrl.defaultZone=http://peer2:1112/eureka/
```
> 创建application-peer2.properties，作为peer2服务中心的配置，并将serviceUrl指向peer1
```
spring.application.name=eureka-server
server.port=1112
eureka.instance.hostname=peer2
eureka.client.serviceUrl.defaultZone=http://peer1:1111/eureka/
```
> 在/etc/hosts文件中添加对peer1和peer2的转换
```
127.0.0.1 peer1
127.0.0.1 peer2
```
> 通过spring.profiles.active属性来分别启动peer1和peer2
```
java -jar eureka-server-1.0.0.jar --spring.profiles.active=peer1
java -jar eureka-server-1.0.0.jar --spring.profiles.active=peer2
```
> 此时访问peer1的注册中心,registered-replicas中已经有peer2节点的eureka-server了。访问peer2的注册中心，registered-replicas中已经有peer1节点，并且这些节点在可用分片（available-replicase）之中。关闭peer1，刷新，peer1的节点变为了不可用分片（unavailable-replicas）。  
> 在设置了多节点的服务注册中心之后，需要简单需求服务配置，将服务注册到Eureka Server集群中。
#### application.properties
```
spring.application.name=compute-service
server.port=2222
eureka.client.serviceUrl.defaultZone=http://peer1:1111/eureka/,http://peer2:1112/eureka/
```
> compute-service同时被注册到了peer1和peer2上。若此时断开peer1，由于compute-service同时也向peer2注册，因此在peer2上其他服务依然能访问到compute-service，从而实现了高可用的服务注册中心。  
> 两两注册的方式可以实现集群中节点完全对等的效果，实现最高可用性集群，任何一台注册中心故障都不会影响服务的注册与发现   
# Ribbon & Feign
### 1. Ribbon
> Ribbon 是一个基于HTTP和TCP客户端的负载均衡器.  
Ribbon 可以通过客户端配置的ribbonServerList服务端列表轮询访问.  
当Ribbon雨Eureka联合使用时,ribbobnServerList被DiscoverEnabledNIWSServerList重写,扩展成从Eureka注册中心中获取服务端列表.同时会用NIWSDiscoveryPing取代IPing,他将职责委托给Eureka来确定服务端是否已经启动.  

#### pom.xml
```
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-ribbon</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
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
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
	    <groupId>org.springframework.cloud</groupId>
	    <artifactId>spring-cloud-dependencies</artifactId>
	    <version>Brixton.RELEASE</version>
	    <type>pom</type>
	    <scope>import</scope>
	</dependency>
    </dependencies>
</dependencyManagement>
```
#### Application 中增加 @EnableDiscoveryClient
> @EnableDiscoveryClient注解来添加发现服务能力。
```
@SpringBootApplication
@EnableDiscoveryClient
public class RibbonApplication {
	@Bean
	@LoadBalanced
	RestTemplate restTemplate() {
		return new RestTemplate();
	}
	public static void main(String[] args) {
		SpringApplication.run(RibbonApplication.class, args);
	}
}
```
> 创建RestTemplate实例，并通过@LoadBalanced注解开启均衡负载能力。
#### application.properties
```
spring.application.name=ribbon-consumer
server.port=3333
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/
```
> spring.application.name :  指定微服务的名称后续在调用的时候只需要使用该名称就可以进行服务的访问。  
> server.port : 端口  
> eureka.client.serviceUrl.defaultZone : 指定服务注册中心的位置。 
***
### 2. Feign
> Feign是一个声明式Web Service客户端.我们只需要使用Feign来创建一个接口并用注解来配置它既可完成。它具备可插拔的注解支持，包括Feign注解和JAX-RS注解。Feign也支持可插拔的编码器和解码器。Spring Cloud为Feign增加了对Spring MVC注解的支持，还整合了Ribbon和Eureka来提供均衡负载的HTTP客户端实现。

#### pom.xml
```
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
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
</dependencies>
<dependencyManagement>
    <dependencies>
        <dependency>
	    <groupId>org.springframework.cloud</groupId>
	    <artifactId>spring-cloud-dependencies</artifactId>
	    <version>Brixton.RELEASE</version>
	    <type>pom</type>
	    <scope>import</scope>
	</dependency>
    </dependencies>
</dependencyManagement>
```
#### Application 增加 @EnableFeignClients
> @EnableFeignClients注解开启Feign功能.
```
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class FeignApplication {
	public static void main(String[] args) {
		SpringApplication.run(FeignApplication.class, args);
	}
}
```
#### application.properties
```
spring.application.name=feign-consumer
server.port=3333
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/
```
> spring.application.name :  指定微服务的名称后续在调用的时候只需要使用该名称就可以进行服务的访问。  
> server.port : 端口  
> eureka.client.serviceUrl.defaultZone : 指定服务注册中心的位置。   
#### ???
> Feign中也使用Ribbon  
> Feign与Ribbon端口一致 ???
# 断路器
> 在微服务架构中，我们将系统拆分成了一个个的服务单元，各单元间通过服务注册与订阅的方式互相依赖。由于每个单元都在不同的进程中运行，依赖通过远程调用的方式执行，这样就有可能因为网络原因或是依赖服务自身问题出现调用故障或延迟，而这些问题会直接导致调用方的对外服务也出现延迟，若此时调用方的请求不断增加，最后就会出现因等待出现故障的依赖方响应而形成任务积压，最终导致自身服务的瘫痪。

> 举个例子，在一个电商网站中，我们可能会将系统拆分成，用户、订单、库存、积分、评论等一系列的服务单元。用户创建一个订单的时候，在调用订单服务创建订单的时候，会向库存服务来请求出货（判断是否有足够库存来出货）。此时若库存服务因网络原因无法被访问到，导致创建订单服务的线程进入等待库存申请服务的响应，在漫长的等待之后用户会因为请求库存失败而得到创建订单失败的结果。如果在高并发情况之下，因这些等待线程在等待库存服务的响应而未能释放，使得后续到来的创建订单请求被阻塞，最终导致订单服务也不可用。

> 在微服务架构中，存在着那么多的服务单元，若一个单元出现故障，就会因依赖关系形成故障蔓延，最终导致整个系统的瘫痪，这样的架构相较传统架构就更加的不稳定。为了解决这样的问题，因此产生了断路器模式。

## 什么是断路器
> 断路器模式源于Martin Fowler的Circuit Breaker一文。“断路器”本身是一种开关装置，用于在电路上保护线路过载，当线路中有电器发生短路时，“断路器”能够及时的切断故障电路，防止发生过载、发热、甚至起火等严重后果。

> 在分布式架构中，断路器模式的作用也是类似的，当某个服务单元发生故障（类似用电器发生短路）之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个错误响应，而不是长时间的等待。这样就不会使得线程因调用故障服务被长时间占用不释放，避免了故障在分布式系统中的蔓延。
### Netflix Hystrix
> 在Spring Cloud中使用了Hystrix 来实现断路器的功能。Hystrix是Netflix开源的微服务框架套件之一，该框架目标在于通过控制那些访问远程系统、服务和第三方库的节点，从而对延迟和故障提供更强大的容错能力。Hystrix具备拥有回退机制和断路器功能的线程和信号隔离，请求缓存和请求打包，以及监控和配置等功能。
### 1. Ribbon中引入Hystrix
#### pom.xml
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```
#### Application 增加@EnableCircuitBreaker
> @EnableCircuitBreaker注解开启断路器功能
```
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker
public class RibbonApplication {
	@Bean
	@LoadBalanced
	RestTemplate restTemplate() {
		return new RestTemplate();
	}
	public static void main(String[] args) {
		SpringApplication.run(RibbonApplication.class, args);
	}
}
```
> 在使用ribbon消费服务的函数上增加@HystrixCommand注解来指定回调方法。
```
@Service
public class ComputeService {
    @Autowired
    RestTemplate restTemplate;
    @HystrixCommand(fallbackMethod = "addServiceFallback")
    public String addService() {
        return restTemplate.getForEntity("http://COMPUTE-SERVICE/add?a=10&b=20", String.class).getBody();
    }
    public String addServiceFallback() {
        return "error";
    }
}
```
***
### 2. Feign使用Hstrix
> 我们不需要在Feigh工程中引入Hystix，Feign中已经依赖了Hystrix.

#### 使用@FeignClient注解中的fallback属性指定回调类
```
@FeignClient(value = "compute-service", fallback = ComputeClientHystrix.class)
public interface ComputeClient {
    @RequestMapping(method = RequestMethod.GET, value = "/add")
    Integer add(@RequestParam(value = "a") Integer a, @RequestParam(value = "b") Integer b);
}
```
#### 创建回调类ComputeClientHystrix，实现@FeignClient的接口，此时实现的方法就是对应@FeignClient接口中映射的fallback函数。
```
@Component
public class ComputeClientHystrix implements ComputeClient {
    @Override
    public Integer add(@RequestParam(value = "a") Integer a, @RequestParam(value = "b") Integer b) {
        return -9999;
    }
}
```
# 分布式配置中心
### 1. 构建Config Server
#### pom.xml
```
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>1.3.5.RELEASE</version>
	<relativePath/> <!-- lookup parent from repository -->
</parent>
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-config-server</artifactId>
	</dependency>
</dependencies>
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>Brixton.RELEASE</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```
#### Application 增加@EnableConfigServer
> @EnableConfigServer注解，开启Config Server
```
@EnableConfigServer
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```
#### application.properties
```
spring.application.name=config-server
server.port=7001
# git管理配置
spring.cloud.config.server.git.uri=
spring.cloud.config.server.git.searchPaths=
spring.cloud.config.server.git.username=username
spring.cloud.config.server.git.password=password
```
> spring.cloud.config.server.git.uri：配置git仓库位置  
> spring.cloud.config.server.git.searchPaths：配置仓库路径下的相对搜索位置，可以配置多个  
> spring.cloud.config.server.git.username：访问git仓库的用户名  
> spring.cloud.config.server.git.password：访问git仓库的用户密码  

> 我们只需要设置属性spring.profiles.active=native，Config >Server会默认从应用的src/main/resource目录下检索配置文件。也可以通过spring.cloud.config.server.native.searchLocat>ions=file:F:/properties/属性来指定配置文件的位置。

> URL与配置文件的映射关系如下：
```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```
> 上面的url会映射{application}-{profile}.properties对应的配置文件，{label}对应git上不同的分支，默认为master。
### 2. 微服务端映射配置
#### pom.xml
```
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>1.3.5.RELEASE</version>
	<relativePath/> <!-- lookup parent from repository -->
</parent>
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-config</artifactId>
	</dependency>
</dependencies>
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>Brixton.RELEASE</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```
#### AppLication 
```
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```
#### bootstrap.properties
```
spring.application.name=didispace
spring.cloud.config.profile=dev
spring.cloud.config.label=master
spring.cloud.config.uri=http://localhost:7001/
server.port=7002
```
> 上面这些属性必须配置在bootstrap.properties中，config部分内容才能被正确加载。因为config的相关配置会先于application.properties，而bootstrap.properties的加载也是先于application.properties。  
#### 创建一个Rest Api来返回配置中心的from属性，具体如下：
```
@RefreshScope
@RestController
class TestController {
    @Value("${from}")
    private String from;
    @RequestMapping("/from")
    public String from() {
        return this.from;
    }
}
```
> 通过@Value("${ }")绑定配置服务中配置的属性。
### 2. config-client配置
#### pom.xml
```
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-config</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka</artifactId>
	</dependency>
</dependencies>
```
> 配置刷新 
> 在config-clinet的pom.xml中新增spring-boot-starter-actuator监控模块，其中包含了/refresh刷新API。  
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
### Application @EnableDiscoveryClient
> @EnableDiscoveryClient注解，用来发现config-server服务，利用其来加载应用配置
```
@EnableDiscoveryClient
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```
#### bootstrap.properties
```
spring.application.name=didispace
server.port=7002
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/
spring.cloud.config.discovery.enabled=true
spring.cloud.config.discovery.serviceId=config-server
spring.cloud.config.profile=dev
```
> eureka.client.serviceUrl.defaultZone参数指定服务注册中心，用于服务的注册与发现  
> spring.cloud.config.discovery.enabled参数设置为true，开启通过服务来访问Config Server的功能  
> spring.cloud.config.discovery.serviceId参数来指定Config Server注册的服务名。  
> spring.application.name和spring.cloud.config.profile 用来定位Git中的资源。  
# 网关服务 Zuul
#### pom.xml
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```
#### Application @EnableZuulProxy
> @EnableZuulProxy注解开启Zuul
```
@EnableZuulProxy
@SpringCloudApplication
public class Application {
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```
> @SpringCloudApplication注解 它整合了@SpringBootApplication、@EnableDiscoveryClient、@EnableCircuitBreaker，主要目的还是简化配置。
#### application.properties
```
spring.application.name=api-gateway
server.port=5555
```
### Zuul配置
#### 服务路由
- 1. 通过url直接映射，我们可以如下配置
```
# routes to url
zuul.routes.api-a-url.path=/api-a-url/**
zuul.routes.api-a-url.url=http://localhost:2222/
```
> 该配置，定义了，所有到Zuul的中规则为：/api-a-url/**的访问都映射到http://localhost:2222/上，也就是说当我们访问http://localhost:5555/api-a-url/add?a=1&b=2的时候，Zuul会将该请求路由到：http://localhost:2222/add?a=1&b=2上。
- 2. 将Zuul注册到eureka server上去发现其他服务，我们就可以实现对serviceId的映射。
```
zuul.routes.api-a.path=/api-a/**
zuul.routes.api-a.serviceId=service-A
zuul.routes.api-b.path=/api-b/**
zuul.routes.api-b.serviceId=service-B
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/
```
> 尝试通过服务网关来访问service-A和service-B，根据配置的映射关系，分别访问下面的url  
> http://localhost:5555/api-a/add?a=1&b=2：通过serviceId映射访问service-A中的add服务  
> http://localhost:5555/api-b/add?a=1&b=2：通过serviceId映射访问service-B中的add服务  
> http://localhost:5555/api-a-url/add?a=1&b=2：通过url映射访问service-A中的add服务  
> erviceId映射方式支持断路器，对于服务故障的情况下，可以有效的防止故障蔓延到服务网关上而影响整个系统的对外服务
### 服务过滤
> 在服务网关中定义过滤器只需要继承ZuulFilter抽象类实现其定义的四个抽象函数就可对请求进行拦截与过滤。  
> 下面的例子，定义了一个Zuul过滤器，实现了在请求被路由之前检查请求中是否有accessToken参数，若有就进行路由，若没有就拒绝访问，返回401 Unauthorized错误。
```
public class AccessFilter extends ZuulFilter  {
    private static Logger log = LoggerFactory.getLogger(AccessFilter.class);
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
        log.info(String.format("%s request to %s", request.getMethod(), request.getRequestURL().toString()));
        Object accessToken = request.getParameter("accessToken");
        if(accessToken == null) {
            log.warn("access token is empty");
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            return null;
        }
        log.info("access token ok");
        return null;
    }
}
```
> 自定义过滤器的实现，需要继承ZuulFilter，需要重写实现下面四个方法：
- filterType：返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型，具体如下：
> pre：可以在请求被路由之前调用  
> routing：在路由请求时候被调用  
> post：在routing和error过滤器之后被调用  
> error：处理请求时发生错误时被调用  
- filterOrder：通过int值来定义过滤器的执行顺序
- shouldFilter：返回一个boolean类型来判断该过滤器是否要执行，所以通过此函数可实现过滤器的开关。在上例中，我们直接返回true，所以该过滤器总是生效。
- run：过滤器的具体逻辑。需要注意，这里我们通过ctx.setSendZuulResponse(false)令zuul过滤该请求，不对其进行路由，然后通过ctx.setResponseStatusCode(401)设置了其返回的错误码，当然我们也可以进一步优化我们的返回，比如，通过ctx.setResponseBody(body)对返回body内容进行编辑等。
> 在实现了自定义过滤器之后，还需要实例化该过滤器才能生效，我们只需要在应用主类中增加如下内容：
```
@EnableZuulProxy
@SpringCloudApplication
public class Application {
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
	@Bean
	public AccessFilter accessFilter() {
		return new AccessFilter();
	}
}
```
# 使用Spring Cloud Security OAuth2搭建授权服务
> spring Cloud Security OAuth2 是 Spring 对 OAuth2 的开源实现，优点是能与Spring Cloud技术栈无缝集成，如果全部使用默认配置，开发者只需要添加注解就能完成 OAuth2 授权服务的搭建。
####　pom.xml
```
<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-security</artifactId>
</dependency>

<dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-oauth2</artifactId>
 </dependency>
```
#### Application @EnableAuthorizationServer
```
@SpringBootApplication
@EnableAuthorizationServer
public class AlanOAuthApplication {
    public static void main(String[] args) {
        SpringApplication.run(AlanOAuthApplication.class, args);
    }
}
```
### 分配 client_id, client_secret
> Spring Security OAuth2的配置方法是编写@Configuration类继承AuthorizationServerConfigurerAdapter，然后重写void configure(ClientDetailsServiceConfigurer clients)方法
```
@Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory() // 使用in-memory存储
                .withClient("client") // client_id
                .secret("secret") // client_secret
                .authorizedGrantTypes("authorization_code") // 该client允许的授权类型
                .scopes("app"); // 允许的授权范围
    }
```