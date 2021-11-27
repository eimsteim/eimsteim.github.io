---
title: 使用Feign的方式访问Zuul网关
tags:
  - Spring
  - Cloud
  - Zuul
  - Feign
categories: 技术
abbrlink: 44878
date: 2018-04-24 00:42:45
---
在微服务架构中，服务是零散的，去中心化的。对于调用者来说，调用这些服务就需要了解服务的地址。比如：我们有一个手机APP，需要访问多个微服务接口才能满足不同的业务功能，而不同的微服务地址显然是不同的，如果微服务数量激增，则对于APP开发者来说，维护服务地址的工作几乎是灾难性的。
因此Spring Cloud引入了Zuul网关服务来解决这个问题。APP只需要访问Zuul，并告诉Zuul自己所需要的服务名称，即可触达对应的服务实例。
引入网关服务的好处有很多，网上的资料也非常多，这里就不再赘述。本文想讨论的是另一个场景的用法：既然我们可以在外部通过REST的方式来访问网关，并赋予网关更多的职能，那么可不可以在内部服务调度的时候，也经过网关？由于我们的内部服务调度是通过Feign的形式，所以如何使用Feign的方式访问Zuul网关就成了一个问题。这里，不论是书籍还是网上资料，几乎没有找到合适的资料。所以我觉得有必要写下这篇文章，以备后来者查阅。
<!--more-->
首先说一下，如何定义一个Zuul网关服务。非常简单：
1、创建一个Spring Boot工程，并在启动类上加入`@EnableZuulProxy`注解
```java
@EnableZuulProxy
@SpringBootApplication
public class MicroserviceApiGatewayApplication {

	public static void main(String[] args) {
		SpringApplication.run(MicroserviceApiGatewayApplication.class, args);
	}
}
```
2、核心`pom.xml`配置：
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.6.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <spring-cloud.version>Edgware.RELEASE</spring-cloud.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zuul</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
</dependencies>
```
3、配置`application.properties`（我个人还是比较喜欢properties，可能现在更流行yml，这个看个人喜好）
```shell
spring.application.name=api-gateway
server.port=5555
#资源中心服务路由
zuul.routes.resource-central.path=/resource-central/**
zuul.routes.resource-central.serviceId=resource-central
#配置中心服务路由
zuul.routes.config-central.path=/config-central/**
zuul.routes.config-central.serviceId=config-central
#消息中心服务路由
zuul.routes.message-central.path=/message-central/**
zuul.routes.message-central.serviceId=message-central

#注册服务中心地址
eureka.client.serviceUrl.defaultZone = http://192.168.10.111:8761/eureka/
```
通过上三个步骤，一个Zuul网关服务就搭建好了。当然前提是你的`eureka`和其他微服务也是正常运行的。

接下来，我们来创建一个普通的Feign客户端（实际上就是一个普通的Java工程，并不需要以Spring Boot方式创建）：
```java
@FeignClient("resource-central")
public interface ResourceCentralClient {

    @LoadBalanced
    @RequestMapping(value = "/uploadFile", method = RequestMethod.POST)
    ObjectResponse uploadFile(@RequestBody String base64);
}
```
这个客户端显然是可以通过eureka寻址到一个名字叫做resource-central的微服务，如果调用uploadFile，则可以获得服务端的uploadFile接口返回值。但是这样做有一个问题：这个请求并未经过Zuul。

怎样使其能够经过Zuul呢？让我们来做一点点小小的改造：
```java
@FeignClient("api-gateway")
public interface ResourceCentralClient {

    @LoadBalanced
    @RequestMapping(value = "/resource-central/uploadFile", method = RequestMethod.POST)
    ObjectResponse uploadFile(@RequestBody String base64);
}
```
可以看到，我们把`@FeignClient`中的value改成了Zuul网关的服务名。如此一来，我们调用此接口时，就会首先向eureka索取Zuul的服务实例，然后告诉Zuul，我需要一个地址为：/resource-central/uploadFile 的服务！
根据上面Zuul网关服务的路由配置，Zuul便可以定位到serviceId，然后再向eureka索取对应的微服务实例，发起请求，并返回结果。
至此，我们的目的已经全部实现，你可以在Zuul网关中做你想做的任何事了。其实原理很简单，就是有一点绕弯子。

OK，今天就讲到这里。

