---
layout: _post
title: Spring 中 Bean 的生命周期
date: 20124-04-18
tags: 
    - Spring
categories: 
    - Spring
---

## 1.背景

本文旨在分析 Bean 生命周期的整体流程，Bean 就是一些 Java 对象，只不过这些 Bean 不是我们主动 new 出来的，而是交个 Spring IOC 容器创建并管理的，因此 Bean 的生命周期受 Spring IOC 容器控制，Bean 生命周期大致分为以下几个阶段：

1. Bean的实例化(Instantiation) :

   Spring 框架会取出 BeanDefinition 的信息进行判断当前 Bean 的范围是否是 singleton 的，是否不是延迟加载的，是否不是 FactoryBean 等，最终将一个普通的 singleton 的 Bean 通过反射进行实例化

2. Bean 的属性赋值(Populate) :
	Bean 实例化之后还仅仅是个"半成品"，还需要对 Bean 实例的属性进行填充，Bean 的属性赋值就是指 Spring 容器根据 BeanDefinition 中属性配置的属性值注入到 Bean 对象中的过程。

3. Bean 的初始化(Initialization) :
对 Bean 实例的属性进行填充完之后还需要执行一些 Aware 接口方法、执行 BeanPostProcessor 方法、执行 InitializingBean 接口的初始化方法、执行自定义初始化 init 方法等。该阶段是 Spring 最具技术含量和复杂度的阶段，并且 Spring 高频面试题 Bean 的循环引用问题也是在这个阶段体现的；
4. Bean 的使用阶段:
经过初始化阶段，Bean 就成为了一个完整的 Spring Bean，被存储到单例池 singletonObjects 中去了，即完成了 Spring Bean 的整个生命周期，接下来 Bean 就可以被随心所欲地使用了。
5. Bean 的销毁(Destruction) ：
Bean 的销毁是指 Spring 容器在关闭时，执行一些清理操作的过程。在 Spring 容器中， Bean 的销毁方式有两种：销毁方法 destroy-method 和 DisposableBean 接口。



## 2. 生命周期接口方法分类

Bean 的完整生命周期经历了各种方法调用，这些方法可以划分为以下几类：

1. Bean 自身的方法:
包括了 Bean 本身调用的方法和通过配置文件中<bean>的 init-method 和 destroy-method 指定的方法
2. Bean 级生命周期接口方法:
包括了 BeanNameAware、BeanFactoryAware、InitializingBean 和 DiposableBean 这些接口的方法
3. 容器级生命周期接口方法:
包括了InstantiationAwareBeanPostProcessor 和 BeanPostProcessor 这两个接口实现，一般称它们的实现类为“后处理器”。
4. 工厂后处理器接口方法:
包括了 AspectJWeavingEnabler, ConfigurationClassPostProcessor, CustomAutowireConfigurer 等等非常有用的工厂后处理器　　接口的方法。工厂后处理器也是容器级的。在应用上下文装配配置文件之后立即调用。



## 3. Bean 生命周期

![bean初始化](bean初始化.jpg)



## 4. 代码验证

### 4.1 引入依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>5.3.20</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.20</version>
</dependency>
```

### 4.2 Bean定义

```java
package com.example.bean.lifecycle;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class Person implements BeanNameAware, ApplicationContextAware, InitializingBean, DisposableBean {
    private String name;

    public Person() {
        System.out.println("[构造器]调用Person的构造器实例化");
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        System.out.println("[注入属性]注入属性name");
        this.name = name;
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("[BeanNameAware接口]调用BeanNameAware#setBeanName");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("[ApplicationContextAware]调用ApplicationContextAware#setApplicationContext");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("[InitializingBean接口]调用InitializingBean#afterPropertiesSet");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("[DisposableBean接口]调用DisposableBean#destroy");
    }

    @PostConstruct
    public void postConstruct() {
        System.out.println("[PostConstruct接口]调用PostConstruct注解的方法");
    }

    @PreDestroy
    public void preDestroy() {
        System.out.println("[PreDestroy接口]调用了PreDestroy注解的方法");
    }

    public void selfInit() {
        System.out.println("[init-method]调用<bean>的init-method属性指定的初始化方法");
    }

    public void selfDestroy() {
        System.out.println("[destroy-method]调用<bean>的destroy-method属性指定的销毁方法");
    }
}

```

### 4.3 InstantiationAwareBeanPostProcessor 定义

```java
package com.example.bean.lifecycle;


import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor;
import org.springframework.stereotype.Component;

@Component
public class MyInstantiationAwareBeanPostProcessor implements InstantiationAwareBeanPostProcessor {
    @Override
    public Object postProcessBeforeInstantiation(Class<?> BeanClass, String BeanName) throws BeansException {
        System.out.println("[InstantiationAwareBeanPostProcessor接口]调用InstantiationAwareBeanPostProcessor前置方法, BeanName:" + BeanName);
        return null;
    }

    @Override
    public boolean postProcessAfterInstantiation(Object Bean, String BeanName) throws BeansException {
        System.out.println("[InstantiationAwareBeanPostProcessor接口]调用InstantiationAwareBeanPostProcessor后置方法, BeanName:" + BeanName);
        return false;
    }
}


```



### 4.4 BeanPostProcessor定义

```java
package com.example.bean.lifecycle;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.stereotype.Component;

@Component
public class MyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("[BeanPostProcessor接口]调用了BeanPostProcessor的前置处理方法");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("[BeanPostProcessor接口]调用了BeanPostProcessor的后置处理方法");
        return bean;
    }
}

```

### 4.5 测试入口

```plain
package com.example.bean.lifecycle;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@ComponentScan(basePackages = {"com.example.bean"})
@Configuration
public class AppConfig {
    @Bean(initMethod = "selfInit", destroyMethod = "selfDestroy")
    public Person person() {
        Person person = new Person();
        person.setName("tester");
        return person;
    }

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        context.close();
    }
}

```



### 4.6 执行结果

```bash
[InstantiationAwareBeanPostProcessor接口]调用InstantiationAwareBeanPostProcessor前置方法, BeanName:person
[构造器]调用Person的构造器实例化
[注入属性]注入属性name
[InstantiationAwareBeanPostProcessor接口]调用InstantiationAwareBeanPostProcessor后置方法, BeanName:person

[BeanNameAware接口]调用BeanNameAware#setBeanName
[ApplicationContextAware]调用ApplicationContextAware#setApplicationContext
[BeanPostProcessor接口]调用了BeanPostProcessor的前置处理方法
[PostConstruct接口]调用PostConstruct注解的方法
[InitializingBean接口]调用InitializingBean#afterPropertiesSet
[init-method]调用<bean>的init-method属性指定的初始化方法
[BeanPostProcessor接口]调用了BeanPostProcessor的后置处理方法

[PreDestroy接口]调用了PreDestroy注解的方法
[DisposableBean接口]调用DisposableBean#destroy
[destroy-method]调用<bean>的destroy-method属性指定的销毁方法
```

控制台打印结果验证了 person 对象的生命周期相关方法执行顺序严格遵从上面流程图，同时当我们执行容器`applicationContext`的关闭方法`close()`会触发调用 bean 的销毁回调方法
