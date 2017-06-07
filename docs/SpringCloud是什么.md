[Spring Cloud](http://cloud.spring.io/)是一个基于[Spring Boot](http://projects.spring.io/spring-boot/)实现的云应用开发工具，它为基于JVM的云应用开发中的配置管理、服务发现、断路器、智能路由、微代理、控制总线、全局锁、决策竞选、分布式会话和集群状态管理等操作提供了一种简单的开发方式，Spring Cloud 1.0在经过将近半年的开发和发布3个里程碑版本和3个RC版本后，其正式版终于发布了，且已提供在[Maven中央仓库](http://search.maven.org/#search%7Cga%7C1%7Cspring%20Cloud)和[Spring的仓库](http://repo.spring.io/libs-milestone-local/)中。Spring Cloud 1.0正式版所实现的主要改进内容包括：

* [Spring Cloud Config](http://cloud.spring.io/spring-cloud-config)子项目支持使用Git配置客服务器 
* Spring Cloud Config子项目支持客户端的配置信息刷新、加密/解密配置、基于Spring应用的声明周期阶段配置 
* [Spring Cloud Commons](https://github.com/spring-cloud/spring-cloud-commons)子项目实现了负载均衡、服务发现和断路器（Circuit Breaker） 
* [Spring Cloud Security](https://github.com/spring-cloud/spring-cloud-security)子项目实现了声明式的SSO和基于代理的认证策略 
* Eureka子项目实现了非JVM客户端的支持 
* 使用Zuul实现了自动反向代理 
* Spring配置模型已支持Zuul过滤器和云中间层服务库[Ribbon](https://github.com/Netflix/ribbon)负载均衡配置 
* 通过Ribbon集成实现了伪声明式的Web服务客户端 
* 实现了RestTemplate同Ribbon的集成 
* 分布式系统的延迟和容错库Hystrix通过客户端操作界面即可实现断路器 
* 实时流、低延时、高吞吐量的聚合器[Turbine](https://github.com/Netflix/Turbine)实现了环路聚合、基于HTTP的Pull操作和基于AMQP的Push操作 
* 对AWS服务的集成实现了对相关数据库、消息、EC2元数据的支持 
* 为AMQP总线定义了一套操作事件，如配置变化等 
* Groovy CLI实现了对上述多数功能的支持

Pivotal于去年六月份[公布了Spring Cloud 1.0源码](http://www.infoq.com/news/2014/06/spring-cloud-platform-abstract)，代码托管在[GitHub](https://github.com/spring-cloud)。Spring Cloud工程包括Spring Cloud Config、[Spring Cloud Netflix](https://github.com/spring-cloud/spring-cloud-netflix)、[Spring Cloud CloudFoundry](https://github.com/spring-cloud/spring-cloud-cloudfoundry)、[Spring Cloud AWS](https://github.com/spring-cloud/spring-cloud-aws)、Spring Cloud Security、Spring Cloud Commons、[Spring Cloud Zookeeper](https://github.com/spring-cloud/spring-cloud-zookeeper)、[Spring Cloud CLI](https://github.com/spring-cloud/spring-cloud-cli)等项目。更多关于Spring Cloud的内容以及如何将Spring Cloud部署到Cloud Foundry和Heroku，读者可以参考[关于Spring Cloud简介](http://spring.io/blog/2014/06/03/introducing-spring-cloud)的博文。


![huaban](https://user-images.githubusercontent.com/11325103/26866117-58aa1e72-4b92-11e7-991b-32acc7c84fcc.png)
