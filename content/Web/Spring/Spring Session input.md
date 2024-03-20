---
aliases: 
date created: 2023-04-22 18:04:00
date modified: 2024-03-20 11:03:08
title: Spring Session
tags: [input]
---

# Spring Session

#### 引入依赖
```xml
<dependency> 
	<groupId>org.springframework.session</groupId>
	<artifactId>spring-session-data-redis</artifactId> 
</dependency>
```

#### 配置文件
```properties
## Session 存储方式
spring.session.store-type=redis

## Session 过期时间，默认单位为 s
server.servlet.session.timeout=600
## Session 存储到 Redis 键的前缀
spring.session.redis.namespace=test:spring:session

## Redis 相关配置
spring.redis.host=127.0.0.1
spring.redis.password=****
spring.redis.port=6379
```

