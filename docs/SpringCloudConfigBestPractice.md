# Spring Cloud Config Best Practice

```
The HTTP service has resources in the form:

/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```
where the "application" is injected as the spring.config.name in the > SpringApplication (i.e. what is normally "application" in a regular Spring Boot > app), "profile" is an active profile (or comma-separated list of properties), and > "label" is an optional git label (defaults to "master".)
With the spring cloud config built-in mechanism, the spring cloud config will expose all the properties as the REST resources, so where:

application is the spring boot client project spring.application.name
profile is the spring.profiles.active in the spring boot client project
label is the git branch name where git git repo is specific by the spring cloud config server , e.g. spring.cloud.config.server.git.uri
Then the client can GET all the properties against the rules above.

Normally for a spring boot client project, just need to config the spring cloud config server like:

```
spring:
  application:
    name: eureka
  cloud:
    config: 
      uri: http://localhost:8888
  profiles:
    active: dev, prod
```

So the client will GET all the properties: eureka-dev.yml and eureka-prod.yml in the spring cloud config server.

