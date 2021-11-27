---
title: Spring Cloud Config接入Memcached爬坑实录
tags:
  - Spring
  - Cloud
  - Memcached
categories: 技术
abbrlink: 6845
date: 2018-03-28 14:34:10
---
Spring Cloud Config是Spring Cloud全家桶的一部分，由于我之前自己实现过一个配置中心（Peso），所以知道这里面有很多看似简单的功能，实现起来并不容易。在使用Spring Cloud Config之后，发现它的功能和实现都可以说是相当棒的。然而还是会有一些小遗憾，比如：它只支持Properties和Yml两种格式的配置文件，这就对我这次接入Memcached的传统型XML配置文件造成了一定的麻烦。
<!--more-->
> ** 背景 ** 

我们的老工程是通过一个XML文件（ghpcache.xml）来保存Memcached的配置，大概是长这样：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<pools>
    <pool id="data_pool" 
            versionEnable="true"
            poolVersion="0"
            servers="10.10.10.13:11211"
            auth="false"
            opTimeout="3000"
            timeoutExceptionThreshold="1000" />
    <pool id="trade_pool" 
            versionEnable="true"
            poolVersion="0"
            servers="10.10.10.13:11211"
            auth="false"
            opTimeout="3000"
            timeoutExceptionThreshold="1000" />
</pools>
```
然后会通过一个工厂类（GhpMemcacheProxyFactory）来加载：
```java
public MemCacheConfigurer() {
    String configPath = System.getProperty("config_path");

    try {
        GhpMemcacheProxyFactory.configure("file:" + configPath + "/ghpcache.xml");
    } catch (MemcacheInitException e) {
        e.printStackTrace();
    }
}
```
这个MemCacheConfigurer会被注解为`@Configuration`，以保证其会在容器启动时被Spring加载到。
```java
@Configuration
public class MemCacheConfigurer {
    ...
}
```
使用Spring Cloud Config之后，配置文件不再是从本地读取，而是直接从ConfigServer拉取。那么我只能将这个XML文件保存到Git中，以供Config Server拉取。这时候就遇到了第一个问题：

> ** 问题一：Spring Cloud Config 并不支持XML格式的配置文件 ** 

只支持Properties和Yml，怎么办？
我的想法是将XML文件变成properties，这样就可以统一上传到Git中，与其他配置文件一样无差别的进行管理。然而，要实现这个想法，我至少需要完成两个工作：
1. 重写原先的工厂类（GhpMemcacheProxyFactory），扩展一个解析Properties的接口。
2. 要在Spring启动时正确拉取配置，并调用上述1中开发的接口，进行Memcached的初始化。

一切看起来是辣么的美好，感觉很快就能实现了呢。直到我仔细看了下`ghpcache.xml`的配置……
```xml
<?xml version="1.0" encoding="UTF-8"?>
<pools>
    <pool id="data_pool" 
            versionEnable="true"
            poolVersion="0"
            servers="10.10.10.13:11211"
            auth="false"
            opTimeout="3000"
            timeoutExceptionThreshold="1000" />
    <pool id="trade_pool" 
            versionEnable="true"
            poolVersion="0"
            servers="10.10.10.13:11211"
            auth="false"
            opTimeout="3000"
            timeoutExceptionThreshold="1000" />
</pools>
```
这里面有一个根节点`pools`，然后下面有两个同名节点`pool`，这表示Memcached配置了两个“桶”，而这两个“桶”具有相同的属性集，而Properties是严格的1:1键值对，并不能实现类似上述XML中的层次结构，于是第二个问题出现了。

> ** 问题二：怎样将XML文件改写为Properties文件？ ** 

其实这个问题也并不是很复杂，我遇到过类似的玩法：
```sh
park61.memcache.data_pool.versionEnable=true
park61.memcache.data_pool.poolVersion=0
park61.memcache.data_pool.servers=10.10.10.13:11211
park61.memcache.data_pool.auth=false
park61.memcache.data_pool.opTimeout=3000
park61.memcache.data_pool.timeoutExceptionThreshold=1000

park61.memcache.trade_pool.versionEnable=true
park61.memcache.trade_pool.poolVersion=0
park61.memcache.trade_pool.servers=10.10.10.13:11211
park61.memcache.trade_pool.auth=false
park61.memcache.trade_pool.opTimeout=3000
park61.memcache.trade_pool.timeoutExceptionThreshold=1000
```
前两层（park61.memcache）是命名空间，第三层是PoolName，第四层是具体的配置，这样就可以解决问题二了。But,新的问题出现了，Spring获取配置的方式只能是通过@Value注解，获取特定的值，而我根本不知道配置了哪些POOL怎么办？

> ** 问题三：怎样读取动态配置的Properties属性？ ** 

我尝试了很多方法，甚至把Spring Cloud的相关源码拉出来读了一遍，但是很可惜，Spring提供的Environment接口并没有提供获取某个前缀的所有配置的方法。
突然我想到这种方式在之前做Zuul（Spring Cloud的API网关服务）的时候似乎见过，于是把之前的代码翻出来看了下：
```sh
spring.application.name=api-gateway
server.port=5555

zuul.routes.resource-central.path=/resource-central/**
zuul.routes.resource-central.serviceId=resource-central

zuul.routes.config-central.path=/config-central/**
zuul.routes.config-central.serviceId=config-central

#注册服务中心地址
eureka.client.serviceUrl.defaultZone = http://10.10.10.21:8761/eureka/
```
我觉得Zuul能实现，我也应该能，虽然人家是亲儿子……于是我就开始跟Zuul的源码，不一会就找到了：
```java
package org.springframework.cloud.netflix.zuul;
...

@Configuration
@EnableConfigurationProperties({ ZuulProperties.class })
@ConditionalOnClass(ZuulServlet.class)
@ConditionalOnBean(ZuulServerMarkerConfiguration.Marker.class)
// Make sure to get the ServerProperties from the same place as a normal web app would
@Import(ServerPropertiesAutoConfiguration.class)
public class ZuulServerAutoConfiguration {

    @Autowired
    protected ZuulProperties zuulProperties;
    ...
}
```
发现了一种新的玩法，关键在于`@EnableConfigurationProperties`和`ZuulProperties`，前者的意思是开启支持`@ConfigurationProperties`注解的Bean，详见[官方文档](https://docs.spring.io/spring-boot/docs/2.0.0.RC1/api/org/springframework/boot/context/properties/EnableConfigurationProperties.html)。后者就是那个被注解的Bean，关键代码如下：
```java
package org.springframework.cloud.netflix.zuul.filters;
...

@ConfigurationProperties("zuul")
public class ZuulProperties {
    /**
        * Map of route names to properties.
        */
    private Map<String, ZuulRoute> routes = new LinkedHashMap<>();

    ...

    public static class ZuulRoute {

        /**
            * The ID of the route (the same as its map key by default).
            */
        private String id;

        /**
            * The path (pattern) for the route, e.g. /foo/**.
            */
        private String path;

        /**
            * The service ID (if any) to map to this route. You can specify a physical URL or
            * a service, but not both.
            */
        private String serviceId;

        /**
            * A full physical URL to map to the route. An alternative is to use a service ID
            * and service discovery to find the physical address.
            */
        private String url;
        ...
    }
}
```
重点来了，首先讲一下`@ConfigurationProperties`，这个注解的作用是将配置文件的内容抽取出来注入到被注解的类实例中。注入规则如下：
1. 注解中的参数"zuul"是命名空间第一层，成员变量是第二层。
2. 然后第三层会作为routes的key，而第四层作为值注入到ZuulRoute对象的同名属性中。

例如，刚刚的zuul配置：
```sh
zuul.routes.resource-central.path=/resource-central/**
zuul.routes.resource-central.serviceId=resource-central

zuul.routes.config-central.path=/config-central/**
zuul.routes.config-central.serviceId=config-central
```

`resource-central`和`config-central`会被作为`private Map<String, ZuulRoute> routes;` 的两个Key，每个Key会对应一个ZuulRoute实例，而`path`和`serviceId`会被填充到对应ZuulRoute实例的属性中。

这样，在`ZuulServerAutoConfiguration`中的`ZuulProperties`就可以自动的填充属性了。这里有一个小细节需要注意下，就是`ZuulProperties`和`ZuulRoute`的属性，必须都有getter和setter，否则无法成功注入属性值。

接下来就是我们自己的实现了，我模仿`ZuulProperties`写了一个`GhpProperties`，代码就不贴了，因为除了属性名，代码结构几乎一模一样。
最后，我们来实现最初想法的第二步：在Spring启动时正确拉取配置，并调用上述1中开发的接口，进行Memcached的初始化。

这个地方我是这样实现的，在`MemCacheConfigurer`中加了一个方法，并注解为`@PostConstruct`，保证其在容器启动时执行，代码如下：
```java
@PostConstruct
public void init(){

    Map<String, Map<String, Object>> props = new HashMap<>();
    //将含有PoolParam的Map转化为Object的Map，方便ProxyFactory的调用
    Map<String, GhpProperties.PoolParam> temp = ghpProperties.getMemcache();
    for (Map.Entry<String, GhpProperties.PoolParam> entry : temp.entrySet()) {
        Map<String, Object> map = new HashMap<>();
        String key = entry.getKey();

        map.put("versionEnable", entry.getValue().getVersionEnable());
        map.put("poolVersion", entry.getValue().getPoolVersion());
        map.put("servers", entry.getValue().getServers());
        map.put("auth", entry.getValue().getAuth());
        map.put("opTimeout", entry.getValue().getOpTimeout());
        map.put("timeoutExceptionThreshold", entry.getValue().getTimeoutExceptionThreshold());

        props.put(key, map);
    }

    try {
        GhpMemcacheProxyFactory.configure(props);
    } catch (MemcacheInitException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
这个代码就很简单了，无非就是做了一个对象转换，然后调用了`GhpMemcacheProxyFactory.configure(props);`进行Memcached的初始化，这里不再赘述。

完结撒花。