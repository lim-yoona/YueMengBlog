---
layout: post
title: Spring框架的理解与总结
subtitle: Java学习笔记
date: 2024-03-09 19:50:00 +0800
categories: Java
author: 月梦
cover: 'https://z1.ax1x.com/2023/12/01/pisYxbD.jpg'
cover_author: 'Pexels'
cover_author_link: 'https://www.pexels.com/zh-cn/'
tags:
- Java  
---

**Spring** 是 `Java` 生态中一个举足轻重的框架，大大简化了开发，它的主要核心特性包括两个，分别是 **控制反转** 以及 **依赖注入**。  

### 控制反转
**_Inversion of control_** ，简写为 **_IoC_** ，译为 **控制反转**，是一种设计思想，它将对象的 **创建** 和 **管理** 交给了 `Spring` 来管理，更具体地说，是交给了 **_IoC_** 容器来管理。  

**_IoC_** 容器是控制反转的一个实现，它存放着开发者交给 `Spring` 管理的对象，其底层是一个 `Map`。

**_IoC_** 的好处是可以帮我们管理对象间的依赖关系，隐藏了对象的创建逻辑，当我们需要一个对象实例时直接去问 `Spring` 要就行了。这样降低了依赖，减小了耦合。比方说 **A** 类中依赖了 **B** 类，如果没有 **_控制反转_**， **A** 类需要自己去在代码中创建对象 **B** 的实例，倘若对象 **B** 的构造函数或者说具体实现在之后有改变，那么所有依赖了 **B** 类的地方代码都需要重新改，依赖关系比较简单还好，如果依赖关系错综复杂，那简直无处下手。  

常常与 **_控制反转_** 一起出现的一个概念叫做 **_依赖注入_** ，即 **_Dependency Injection_** ，简称 **_DI_** 。 **_依赖注入_** 指程序运行过程中，如果需要依赖另一个对象，那么无需创建这个对象，而是通过外部的注入。**_依赖注入_** 是 **_IoC_** 最常见的一种实现方式。  

### Spring Bean
简单来讲，**_Spring Bean_** 指的就是那些被 **_IoC_** 容器管理的 `Java` 对象。  

如果我们想告诉 `Spring` 哪些对象想要交给它管理，就要进行配置，通常有三种方式，分别是 **XML配置**、**注解** 以及 **Java配置类** 。  

**_XML配置_** 方式通过在项目的 `resources` 文件夹下编写 `Spring` 配置文件来实现配置以及依赖注入，分别通过 `bean` 标签和 `property` 标签来完成，基于 **_XML配置_** 的方式并不方便所以不常用，此处略去。  

基于 **_注解_** 来配置更加方便，是常用的方式。  

#### 基于注解配置
首先需要在 `Spring` 配置文件中开启组件扫描，`Spring` 会自动扫描指定的包及其子包下的所有类，如果类上有相关注解，会执行相应的操作。  
```xml
<context:component-scan base-package="包路径"></context:component-scan>
```

使用注解定义对象，将注解标注在 `Java` 类上，将它们定义为 `Spring Baen` ，具体来说，有以下四个注解都可以用来创建类。  

**@Component** 、 **@Repository** 、 **@Service** 以及 **@Controller**  

这四个注解的功能都是相同的，不同之处仅有含义的区分，其中 **@Component** 是通用的，**@Repository** 用在数据访问层（DAO层），**@Service** 用在业务层，而 **@Controller** 用在控制层。  

与这四个注解不同的是，还有一个同样可以实现将一个对象标注为 `SpringBean` 的注解是 **@Bean**  

**@Bean** 与 **@Component** 有什么区别呢？具体表现在以下方面：  
1. 前者作用于方法，而后者作用于类
2. 后者是通过开启组件扫描从而自动装配的，而前者标注在方法上，在方法中产生了 ***Bean*** ，告诉 `Spring` 你来帮我管理这个 ***Bean***
3. 当我们想要将第三方包中的类装配到 `Spring` 中时，则只能通过 **@Bean** 来实现  


**_属性注入_** 的实现依赖于两个注解，分别是 **@Autowired** 以及 **@Resource**

**@Autowired** 默认根据数据类型装配，也就是说在 **_IoC_** 容器中选择匹配的类型注入  

这个注解可以被用在 **构造方法上** 、 **方法上** 、 **形参上** 、 **属性上** 以及 **注解上**  

较为常用的方式是 **在属性上** 以及 在 **属性的`set`方法** 上写 **@Autowired** 注解来实现依赖注入  

默认来讲，**@Autowired** 是根据类型注入的，但是当一个接口拥有两个实现类时，就无法使用基于类型注入了，所以需要 **@Autowired** 配合 **@Qualifier** 注解实现基于名称的注入

```java
@Qualifier(value = "")
```

另有一注解 **@Resource** 也可以实现属性注入，它不是 `Spring` 的注解，而是属于 `JDK` 扩展包，可以用在 **属性上** 以及 **set方法上** ，它默认根据名称进行装配（指定 `name` 属性），若未指定名称，则按照属性名进行装配，如果通过名称找不到，则会根据类型装配  

前面提到 **基于注解配置** 中要在 `Spring` 配置文件中配置开启组件扫描，如果想要实现全注解的开发，就要使用 **Java配置类** 来进行配置  

写一个配置类，上使用注解 **@Configuration** 标明这是一个配置类，再添加一个注解 **@ComponentScan()** 括号中填写想要扫描的包名 **开启组件扫描** ，这样就代替了 `Spring` 配置文件  

#### Bean的作用域

这里只介绍常见的两种，分别是 ***singleton*** 以及 ***prototype*** 

***singleton*** 是单例模式，这是 `Spring` 默认的 **Bean** 作用域，这样的 **Bean** 在整个 ***IoC*** 容器中只有一个实例  

***prototype*** 每次获取都会创建一个新的 **Bean** 实例，它不是单例的  

配置 **Bean** 作用域的方法在于使用 **@Scope** 注解，此注解的 `value` 属性标识这个 **Bean** 的作用于  

未完待续......