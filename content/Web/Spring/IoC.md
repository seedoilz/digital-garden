---
aliases:
  - Inversion of Control
title: IoC
date created: 2024-03-05 16:03:00
date modified: 2024-03-20 11:03:09
tags: [code/web]
---
## 概述
IoC 是 Inversion of Control 的简写，译为“控制反转”，它不是一门技术，而是一种设计思想，是一个重要的面向对象编程法则，能够指导我们如何设计出松耦合、更优良的程序。

Spring 通过 IoC 容器来管理所有 Java 对象的实例化和初始化，控制对象与对象之间的依赖关系。我们将由 IoC 容器管理的 Java 对象称为 Spring Bean，它与使用关键字 new 创建的 Java 对象没有任何区别。

### 思想
- 控制反转是一种思想。
- 控制反转是为了降低程序耦合度，提高程序扩展力。
- 控制反转，反转的是什么？
  - 将对象的创建权利交出去，交给第三方容器负责。
  - 将对象和对象之间关系的维护权交出去，交给第三方容器负责。

- 控制反转这种思想如何实现呢？
	- **[[DI]]（Dependency Injection）**：依赖注入

Bean管理说的是：Bean对象的创建，以及Bean对象中属性的赋值（或者叫做Bean对象之间关系的维护）。

### IoC容器在Spring的实现
Spring 的 IoC 容器就是 IoC思想的一个落地的产品实现。IoC容器中管理的组件也叫做 bean。在创建 bean 之前，首先需要创建IoC 容器。Spring 提供了IoC 容器的两种实现方式：
#### ①BeanFactory
这是 IoC 容器的基本实现，是 Spring 内部使用的接口。面向 Spring 本身，不提供给开发人员使用。
#### ②ApplicationContext
BeanFactory 的子接口，提供了更多高级特性。面向 Spring 的使用者，几乎所有场合都使用 ApplicationContext 而不是底层的 BeanFactory。
#### ③ApplicationContext的主要实现类
![iamges](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/img005.png)

## 基于XML管理Bean

### 三种方式
#### 1. 根据id获取
```java
User user = (User) context.getBean("user");
```

#### 2. 根据类型获取
```java
User user2 = context.getBean(User.class);
```

#### 3. 根据类型和id获取
```java
User user3 = context.getBean("user", User.class);
```


