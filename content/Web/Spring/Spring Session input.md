---
aliases: 
date created: 四月 22日 2023, 6:47:40 晚上
date modified: 三月 5日 2024, 4:05:13 下午
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

