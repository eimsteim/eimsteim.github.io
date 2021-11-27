---
title: Maven多模块继承和聚合
tags:
  - Maven
categories: 技术
abbrlink: 5863
date: 2016-02-19 10:48:14
---
最近在尝试用模块化的方式重构Maven工程，然而网上很多中文资料大都语焉不详，仅仅只是罗列配置和代码。于是我根据自己的需求重新阅读了Maven的官方文档，温故而知新，在这里将心得分享给大家。
<!--more-->
***
> 问题1：`<packaging>`标签在Maven模块化过程中扮演了什么样的角色？

参照[官方文档](http://maven.apache.org/pom.html)的解释，`<packaging>` 标识了一个模块的构建类型，主要的packaging类型包括：`pom`,`jar`,`maven-plugin`,`ejb`,`war`,`ear`,`rar`,`par`，其中，如果不配置<packaging>，其默认值是`jar`。

> 问题2：怎样正确地声明两个模块之间的依赖（Inheritance）关系？

如果两个模块在打包时存在依赖关系，必须在parent模块中将packaging声明为`pom`，而后可以在child模块中通过`<parent>`标签来声明其对parent模块的依赖。例如：

** Parent模块：**
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.codehaus.mojo</groupId>
  <artifactId>my-parent</artifactId>
  <version>2.0</version>
  <packaging>pom</packaging>
</project>
```
** Child模块 **
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>my-parent</artifactId>
    <version>2.0</version>
    <relativePath>../my-parent</relativePath>
  </parent>

  <artifactId>my-project</artifactId>
</project>
```

Parent模块中的pom配置将会继承到Child模块，继承的属性包括：
- dependencies（依赖包）
- developers and contributors（开发者和贡献者）
- plugin lists（插件列表）
- reports lists（report列表）
- plugin executions with matching ids（不知道……）
- plugin configuration（插件配置）

至此，我突然明白了两件事：
1. Maven中的Parent是被继承的模块，并且和Java一样，Maven的继承是“单继承”。
2. “被继承”的并不是Parent模块本身，而是其配置的各种依赖包和插件。

这与我之前的理解确实存在一些出入。另外，我还注意到`<relativePath>`这个标签，按文档的说法，这个标签其实并非必须，但可以通知Maven优先在哪里寻找到Parent的配置，是本地，还是远程仓库。

> 问题3：`<modules>`标签的作用？聚合（Aggregation）的意义到底是什么？

首先，`<modules>`标签是用于聚合多个有关联的模块，需要注意的是：只有packaging被标识为pom的工程才可以定义为聚合工程，例如：
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.codehaus.mojo</groupId>
  <artifactId>my-parent</artifactId>
  <version>2.0</version>
  <packaging>pom</packaging>

  <modules>
    <module>my-project</module>
    <module>another-project</module>
  </modules>
</project>
```

你并不需要特别考虑modules的顺序，Maven会自动进行排序。

关于聚合的意义，我在文档中找到了下面两句话：
> A project with modules is known as a multimodule, or aggregator project. Modules are projects that this POM lists, and are executed as a group. An pom packaged project may aggregate the build of a set of projects by listing them as modules, which are relative directories to those projects.
***

> ** A final note on Inheritance v. Aggregation **
> Inheritance and aggregation create a nice dynamic to control builds through a single, high-level POM. You will often see projects that are both parents and aggregators. For example, the entire maven core runs through a single base POM org.apache.maven:maven, so building the Maven project can be executed by a single command: mvn compile. However, although both POM projects, an aggregator project and a parent project are not one in the same and should not be confused. A POM project may be inherited from - but does not necessarily have - any modules that it aggregates. Conversely, a POM project may aggregate projects that do not inherit from it.

我的理解是：Aggregation指定了构建时应该一并build的所有模块，而Inheritance仅仅是标识了继承关系。从最后一句即可看出：一个POM工程可能聚合了一系列子模块，但并不继承于它们，即：指定了modules，但并未将其作为其parent。

另外：如果希望在applicationContext.xml中<import resource="classpath:/spring/database-config.xml" />时不会报错（database-config.xml在依赖或聚合的模块中），那么你还需要在IDE（如IntelliJ IDEA）的Project Settings中为Child模块添加了Parent模块的module dependency，这样做了以后在classes目录中就会出现maojiang-core的字节码和xml配置。
***
** 综上所述 **
定义parent只是标识父子模块在dependency上的继承关系，而modules也仅仅是定义了哪些模块应该一起build，但build的目标目录仍然是在各自自己的输出目录中，而只有在IDE中定义了对子模块的依赖关系，才能真正将其打包到自己的输出目录。三者缺一不可。
