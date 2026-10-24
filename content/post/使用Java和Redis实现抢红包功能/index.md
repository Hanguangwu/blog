---
title: 使用Java和Redis实现抢红包功能
description: 本文简要描述了使用Java和Redis实现抢红包功能的流程。
date: 2025-09-06T23:34:25+08:00
draft: false
categories:
- 编程
tags:
- Java
- Redis
---

## 前言

抢红包功能实现主要包括发红包、抢红包、记红包和拆红包四块。

教程来源：[BV13R4y1v7sP](https://www.bilibili.com/video/BV13R4y1v7sP)

{{< bilibili BV13R4y1v7sP >}}

## 配置说明

JDK21


## 相关代码

### Pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.5.5</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.tianhan</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>demo</description>
    <url/>
    <licenses>
        <license/>
    </licenses>
    <developers>
        <developer/>
    </developers>
    <scm>
        <connection/>
        <developerConnection/>
        <tag/>
        <url/>
    </scm>
    <properties>
        <java.version>21</java.version>
    </properties>

    <dependencies>
        <!-- Spring Boot Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- HL: SLF4J 与日志落地（如果需要显式声明） -->
        <!-- Logback 是 Spring Boot 的默认实现，通常不必额外声明 -->
        <!-- 仅在你有特殊日志需求时才添加以下依赖 -->
        <!--
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
        </dependency>
        -->

        <!-- Spring Data Redis -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!-- MySQL 驱动（运行时） -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Lombok（编译时注解处理） -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.38</version>
            <scope>provided</scope> <!-- 如在 IDE/构建时需要，可改为无 scope -->
        </dependency>

        <!-- 测试相关 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

### RedisConfig.java

```java
package com.example.demo.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;

@Configuration
public class RedisConfig {

    /**
     * *redis序列化的工具定置类，下面这个请一定开启配置
     * *127.0.0.1:6379> keys *
     * *1) “ord:102” 序列化过
     * *2)“\xac\xed\x00\x05t\x00\aord:102” 野生，没有序列化过
     * *this.redisTemplate.opsForValue(); //提供了操作string类型的所有方法
     * *this.redisTemplate.opsForList();// 提供了操作List类型的所有方法
     * *this.redisTemplate.opsForSet(); //提供了操作set类型的所有方法
     * *this.redisTemplate.opsForHash(); //提供了操作hash类型的所有方认
     * *this.redisTemplate.opsForZSet(); //提供了操作zset类型的所有方法
     * param LettuceConnectionFactory
     * return
     */
    /**
     * 自定义Redis序列化配置
     * 使用Jackson2JsonRedisSerializer并配置ObjectMapper，避免在序列化时添加类型信息
     * @param lettuceConnectionFactory Redis连接工厂
     * @return 配置好的RedisTemplate实例
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(LettuceConnectionFactory lettuceConnectionFactory) {
        RedisTemplate<String,Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(lettuceConnectionFactory);
        
        // 创建Jackson2JsonRedisSerializer序列化器
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        
        // 配置ObjectMapper
        ObjectMapper objectMapper = new ObjectMapper();
        // 指定要序列化的域，field,get和set,以及修饰符范围，ANY是都有包括private和public
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        // 不启用默认类型，避免在序列化时添加类型信息
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
        
        // 设置key序列化方式string
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        // 设置value的序列化方式
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
}
```

### RedisKeyConstants.java

```java
package com.example.demo.constant;  
  
public class RedisKeyConstants {  
    public static final String RED_PACKAGE_KEY = "redpackage:";  
    public static final String RED_PACKAGE_CONSUME_KEY = "redpackage:consume";  
}
```

### RedPackageController.java

```java
package com.example.demo.controller;

import com.example.demo.dto.ApiResponse;
import com.example.demo.service.RedPackageService;
import jakarta.annotation.Resource;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@Slf4j
@RequestMapping("/red")
public class RedPackageController {

    @Resource
    private RedPackageService redPackageService;

    // 发送红包，返回 JSON：{ code:0, message:"success", data: { key: "...", amounts: [...] } }
    @GetMapping("/send")
    public ApiResponse<RedPackageResponse> sendRedPackage(@RequestParam int totalMoney,
                                                          @RequestParam int redpackageNumber) {
        try {
            String key = redPackageService.writeRedPackages(totalMoney, redpackageNumber);
            // 重新获取一次拆分金额用于返回
            List<Integer> amounts = redPackageService.splitRedPackageAlgorithm(totalMoney, redpackageNumber);
            RedPackageResponse payload = new RedPackageResponse(key, amounts);
            return ApiResponse.success(payload);
        } catch (Exception ex) {
            // 统一错误处理
            return ApiResponse.fail(1001, "发送红包失败：" + ex.getMessage());
        }
    }

    // 抢红包，返回 JSON：{ code:0, message:"success", data: amount } 或错误信息
    @GetMapping("/rob")
    public ApiResponse<String> robRedPackage(@RequestParam String redpackageKey,
                                             @RequestParam String userId) {
        log.info("userId = {}", userId);
        try {
            Integer amount = redPackageService.rob(redpackageKey, userId);
            if (amount == -1) {
                return ApiResponse.fail(-1, "红包已抢完");
            }
            return ApiResponse.success(amount.toString());
        } catch (IllegalStateException ie) {
            return ApiResponse.fail(-2, ie.getMessage());
        } catch (Exception ex) {
            return ApiResponse.fail(1002, "抢红包失败：" + ex.getMessage());
        }
    }

    // 内部响应数据对象，用于 /send 的 data 字段
    public static class RedPackageResponse {
        private String key;
        private List<Integer> amounts;

        public RedPackageResponse(String key, List<Integer> amounts) {
            this.key = key;
            this.amounts = amounts;
        }

        public String getKey() {
            return key;
        }

        public void setKey(String key) {
            this.key = key;
        }

        public List<Integer> getAmounts() {
            return amounts;
        }

        public void setAmounts(List<Integer> amounts) {
            this.amounts = amounts;
        }
    }
}
```

### ApiResponse.java

```java
package com.example.demo.dto;

public class ApiResponse<T> {
    private int code;
    private String message;
    private T data;

    public ApiResponse() { }

    private ApiResponse(int code, String message, T data) {
        this.code = code;
        this.message = message;
        this.data = data;
    }

    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(0, "success", data);
    }

    public static <T> ApiResponse<T> fail(int code, String message) {
        return new ApiResponse<>(code, message, null);
    }

    // getters and setters

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
}
```

### RedPackageServiceImpl.java

```java
package com.example.demo.service.impl;

import com.example.demo.constant.RedisKeyConstants;
import com.example.demo.service.RedPackageService;
import jakarta.annotation.Resource;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;

import java.util.Arrays;
import java.util.List;
import java.util.Random;
import java.util.UUID;
import java.util.concurrent.TimeUnit;

@Service
public class RedPackageServiceImpl implements RedPackageService {

    private static final Logger logger = LoggerFactory.getLogger(RedPackageServiceImpl.class);

    @Resource
    private RedisTemplate<String, Object> redisTemplate;

    @Override
    public String writeRedPackages(int totalMoney, int redpackageNumber) {

        String key = RedisKeyConstants.RED_PACKAGE_KEY + UUID.randomUUID().toString().replace("-", "");
        // 将 Integer[] 转为 List<Integer>，直接写入 Redis，会出现问题，可以参考[redisTemplate中leftPushAll隐性bug的问题](https://blog.csdn.net/cdnight/article/details/88803869)
        List<Integer> splitList = splitRedPackageAlgorithm(totalMoney, redpackageNumber);
        Integer[] splitArray = splitList.toArray(new Integer[0]);
        logger.info("Created red package key: {}, amounts: {}", key, splitArray);
        redisTemplate.opsForList().leftPushAll(key, splitArray);
        redisTemplate.expire(key, 1, TimeUnit.DAYS);

        // 这里返回 key，前端再结合 amounts 解析
        return key;
    }

    @Override
    public Integer rob(String redpackageKey, String userId) throws Exception {
        Object redPackage = redisTemplate.opsForHash().get(RedisKeyConstants.RED_PACKAGE_CONSUME_KEY + redpackageKey, userId);
        if (redPackage == null) {
            Object partRedPackage = redisTemplate.opsForList().leftPop(RedisKeyConstants.RED_PACKAGE_KEY + redpackageKey);
            if (partRedPackage != null) {
                redisTemplate.opsForHash().put(RedisKeyConstants.RED_PACKAGE_CONSUME_KEY + redpackageKey, userId, partRedPackage);
                logger.info("User {} robbed {} from {}", userId, partRedPackage, redpackageKey);
                return Integer.valueOf(partRedPackage.toString());
            }
            return -1;
        }
        // 已抢过
        throw new IllegalStateException("User " + userId + " 已经抢过了");
    }

    @Override
    public List<Integer> splitRedPackageAlgorithm(int totalMoney, int redpackageNumber) {
        Integer[] redpackageNumbers = new Integer[redpackageNumber];
        int useMoney = 0;
        Random random = new Random();

        for (int i = 0; i < redpackageNumber; i++) {
            if (i == redpackageNumber - 1) {
                redpackageNumbers[i] = totalMoney - useMoney;
            } else {
                int avgMoney = ((totalMoney - useMoney) / (redpackageNumber - i)) * 2;
                int value = 1 + random.nextInt(Math.max(1, avgMoney - 1));
                redpackageNumbers[i] = value;
                useMoney += value;
            }
        }
        logger.info("Created red package raw: {}, amounts: {}", redpackageNumbers, Arrays.asList(redpackageNumbers));
        return Arrays.asList(redpackageNumbers);
    }
}
```

### RedPackageService.java

```java
package com.example.demo.service;

import java.util.List;

public interface RedPackageService {

    String writeRedPackages(int totalMoney, int redpackageNumber);

    Integer rob(String redpackageKey, String userId) throws Exception;

    List<Integer> splitRedPackageAlgorithm(int totalMoney, int redpackageNumber);
}
```

### DemoApplication.java

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}

```

### application.yaml

```yaml
server:
  port: 8080
spring:
  application:
    name: demo
  data:
    redis:
      host: localhost
      port: 6379
      lettuce:
        pool:
          max-active: 8
          max-wait: -1ms
          max-idle: 8
          min-idle: 0
```


## 测试

访问`http://localhost:8080/red/send?totalMoney=100&redpackageNumber=5`，输出结果如下所示：

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "key": "redpackage:7fac1ca15feb4173859f3101a13e9f97",
    "amounts": [12, 28, 15, 29, 16]
  }
}
```

访问`http://localhost:8080/red/rob?redpackageKey=1abe520c30dc4ec0b4153e3708c01fd3&userId=2`，输出结果如下所示：

```json
{
  "code": 0,
  "message": "success",
  "data": "54"
}
```

注意：这里的ID要和上面的一致

## 附录——深入了解 RedisTemplate 的 leftPushAll 与序列化问题

在使用 Spring Data Redis 时，`RedisTemplate` 提供了丰富的操作方法，其中 `leftPushAll`（左侧批量推送）是一个常用的列表操作方法。本文围绕两个常见疑问展开：① leftPushAll 能否传入 `Object...`、`Collection`、以及为什么有时需要将 `ArrayList` 转换为数组；② RedisTemplate 的序列化问题以及潜在的隐性 Bug。

---

- 使用 `leftPushAll` 时，理论上可以用两种形式：
    - 变参形式：`leftPushAll(key, value1, value2, ...)`
    - 集合形式：`leftPushAll(key, collection)`
- 如果遇到将 `ArrayList` 直接传入无法生效的情况，可以尝试将集合转换为数组再调用，作为一种兼容性手段，但并非必要的做法，优先确保版本与重载匹配正确。
- 序列化是影响 Redis 数据可用性的关键因素。统一、明确的序列化策略能避免大多数问题。建议使用 `String` 键 + JSON（或自定义序列化器）的组合，并确保在写入和读取时使用同样的序列化配置。

### 1) RedisTemplate 中 leftPushAll 的参数形式

- **支持的参数形式**
  
    - `leftPushAll(K key, V... values)`：可以传一组可变参数，等价于将多个值逐个推入到左边。
    - `leftPushAll(K key, Collection<V> values)`：也可以传入一个 `Collection`，将集合中的元素逐个推入到左边。
- **常见的困惑点**
  
    - 有些开发者在实践中遇到将 `ArrayList<V>` 直接作为参数传入时失败的问题，或者感觉“不工作”。其实问题通常出在 Java 方法重载分辨的类型匹配、或者序列化导致的写入失败，而不一定是 API 自身对 `Collection` 的支持问题。
    - 关键点在于：确保传入的类型与 RedisTemplate 的泛型参数一致，并且底层的序列化策略能够正确序列化这些对象。
- **为什么有时需要把 `ArrayList` 转换为数组 `V[]`？**
  
    - 某些版本的 `RedisTemplate` 的重载实现或编译期行为，可能在编译时推断出具体的 `V... values` 的数组形式，从而更直接地匹配到 `leftPushAll(K, V...)`。如果传入的是 `Collection<V>`，理论上也应工作，但在具体实现或版本差异上，可能出现边缘情况导致调用不走你期望的重载。
    - 将 `ArrayList<V>` 手动转换为 `V[]`（例如 `values.toArray((V[]) new Object[0])`）可以确保调用到你明确的变参重载，从而规避某些版本的坑。但这并非必要的普遍做法，更多的是一个“保险性”的兼容手段。
- **实操建议**
  
    - 优先使用你熟悉的形式：
        - `redisTemplate.opsForList().leftPushAll(key, valuesArray)`，其中 `valuesArray` 为 `V[]`。
        - 或者 `redisTemplate.opsForList().leftPushAll(key, collection)`，直接传 `Collection<V>`。
    - 如果遇到某个版本出现“无法调用正确的重载”或运行时异常，尝试将 Collection 转换为数组再调用一次：
      
        ```java
        List<V> list = new ArrayList<>();//  filling list ...V[] arr = (V[]) list.toArray(new Object[0]);redisTemplate.opsForList().leftPushAll(key, arr);
        ```
        
    - 但请注意：直接将 `List<V>` 转换为 `V[]` 时，需要确保类型安全，避免 `ClassCastException`。在 Java 泛型擦除的场景下，通常使用适配代码或者通过具体类型实例化一个正确类型的数组。

---

### 2) RedisTemplate 的序列化问题

**为什么序列化很关键？**

- RedisTemplate 在往 Redis 写入数据前，会把 Java 对象序列化成字节数组。读取时再把字节数组反序列化成 Java 对象。序列化策略直接决定了你能否正确存取、检索和比较数据。
- 常见序列化策略有：JDK 序列化、JSON（如 `Jackson2JsonRedisSerializer`）、String 序列化、FastJSON 等。

**常见的问题点**

- 序列化不一致：写入时使用一种序列化方式，读取时使用另一种，可能导致反序列化失败或数据不可读。
- 非对称的对象结构：自定义对象若没有正确的序列化实现（如缺少无参构造、缺失 `Serializable` 等），可能在反序列化时出错。
- 字符串化 vs 对象化：把对象错误地序列化成字符串，取回时需要再次解析，容易出错。
- 存入原始字节但读取时却以对象方式处理，或对同一 Key 使用了不同的 Template（不同的序列化策略）导致数据不可用。

**如何正确配置序列化？**

- 统一的序列化策略是关键。推荐做法：
	- 对键使用 `StringRedisSerializer`，对值使用 `Jackson2JsonRedisSerializer`（或自定义的 JSON 序列化器），对哈希键/值也保持一致。
	- 为自定义对象定义明确的序列化/反序列化配置，确保对象具备可序列化的结构。
- 示例配置（简化版，基于 Spring Boot）：

```java
@Bean
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
    RedisTemplate<String, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(connectionFactory);

    // 键序列化
    StringRedisSerializer stringSerializer = new StringRedisSerializer();
    template.setKeySerializer(stringSerializer);
    template.setHashKeySerializer(stringSerializer);

    // 值序列化（对象 -> JSON）
    Jackson2JsonRedisSerializer<Object> jacksonSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
    ObjectMapper om = new ObjectMapper();
    om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
    om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
    jacksonSerializer.setObjectMapper(om);

    template.setValueSerializer(jacksonSerializer);
    template.setHashValueSerializer(jacksonSerializer);

    template.afterPropertiesSet();
    return template;
}
```

注意：如果你明确知道存入的是某一类具体对象，可以将 `Jackson2JsonRedisSerializer` 的泛型改为该对象类型，以提升反序列化的安全性和效率。

### 参考资料

[redisTemplate中leftPushAll隐性bug的问题](https://blog.csdn.net/cdnight/article/details/88803869)







