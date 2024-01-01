---
layout: post title: "Java--Spring Boot3.0新特性"
category: 'Java'
---

Spring Boot 3.0 正式发布了，官网地址如下：
https://spring.io/projects/spring-boot/#learn

Spring Boot 3.0 这是一个重大的主版本更新，距离上一代的Spring Boot 2.0 的发布已经过去 4 年多了，Spring Boot 3.0 也是第一个支持 Spring 6.0+ 和 GraalVM 的 Spring
Boot 正式版本。

# Spring Boot 3.0 重大新特性

1. 最低环境要求
   
Spring Boot 3.0 最低要求 Java 17，并向上兼容支持 Java 19。

2. 大量依赖升级

Spring Boot 3.0 最低支持的 Spring 框架也变成了 Spring 6.0.2+，虽然是框架自动集成依赖的，但需要注意这点，因为前段时间发布的 Spring 6.0 也有
不少的底层升级。

3. 支持 GraalVM 原生镜像

Spring Boot 3.0 应用现在可以支持转换为 GraalVM 原生镜像了，这可以提供显著的内存和启动性能改进，能支持 GraalVM 原生镜像也是整个 Spring 产品组合中的一项重大能力的提升。