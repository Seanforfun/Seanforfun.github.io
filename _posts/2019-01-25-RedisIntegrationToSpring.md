---
layout: post
title:  "Redis| integrateion to Spring"
date:   2018-01-25 17:15
author: Botao Xiao
categories: Redis
comment: true
description: Redis is a kind of NoSQL(Not only sql), which means redis is not a relation based database. Redis works on RAM so we always setup a redis server (not in the EE project) as a first level or second level database. In this post, I will include the five data structure that can be saved in Redis.
---
# Redis| integrateion to Spring

### Introduction
In this file, I want to record the steps to integrate Redis to Spring including the installation of Redis, setup of Redis configure file and visiting redis using Jedis.

### Installation
1. Redis is written using C so we can go to its official website and download the source code.
2. Then we just need to compile the source code.
3. Normally, we want user with authentification to visit redis, so we need to go redis.conf file and uncomment the line "requirepass" and setup personal password.
3. After installation, we need to start the redeis-server so it will run in RAM and works as nosql.
4. In order to visit the database, we just need to run redis-cli and use command line.
5. Redis is always running as a back-end service so we want to run this service using ./utils/install_server.sh
    * Select a port for redis, default 6379
    * Select the configuration file for redis serivce.
    * Select the log file for redis service.
    * Select the data directory for redis service.
    * Select the redis executatble file path.
Currently, when linus is startup, it will follow the init.d and run a bash script to start redis, redis will be started during bootstrap and run as a daemon service.

### Integration with SpringBoot
1. Add dependency of Jedis
    ```xml
    <!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>3.0.1</version>
    </dependency>
    ```

2. Add dependency of FastJson, we also can use protobuf but after serialization, pritobuf is not readable while FastJson can be read in Json format.
    ```xml
    <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.54</version>
    </dependency>
    ```

3. Configuration in SpringBoot:
    * Setup some parameters for initializing JedisPool in Application.properties.
    ```Properties
    redis.host=${spring.redis.host}
    redis.port=${spring.redis.port}
    redis.timeout=3
    redis.password=${spring.redis.password}
    redis.maxActive=${spring.redis.jedis.pool.max-active}
    redis.maxIdle=${spring.redis.jedis.pool.min-idle}
    redis.maxWait=3
    ```
    * Create a Configuration Bean instance for loading all parameters.
    ```Java
    @Component
    @ConfigurationProperties(prefix = "redis")
    @Slf4j
    public class RedisConfig {
        @Getter@Setter
        private String host;
        @Getter@Setter
        private int port;
        @Getter@Setter
        private int timeout;
        @Getter@Setter
        private String password;
        @Getter@Setter
        private int maxActive;
        @Getter@Setter
        private int maxIdle;
        @Getter@Setter
        private int maxWait;
    }
    ```
    * Create a JedisPoolFactory method for getting jedisPool instance.
    ```Java
    @Bean
    public JedisPool jedisPoolFactory(){
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setMaxIdle(redisConfig.getMaxIdle());
        poolConfig.setMaxTotal(redisConfig.getMaxActive());
        poolConfig.setMaxWaitMillis(redisConfig.getMaxWait() * 1000);
        JedisPool pool = new JedisPool(poolConfig, redisConfig.getHost(), redisConfig.getPort(), redisConfig.getTimeout() * 1000, redisConfig.getPassword());
        return pool;
    }
    ```

### Usage
1. Every time we want to have a connection with Redis, we need to retrieve a connection from redisPool and close the connection after finish our transaction.
```Java
public <T> T get(String key, Class<T> clazz){
    Jedis jedis = null;
    try {
        jedis = this.jedisPool.getResource();   // Get connection from redis pool.
        String string  =jedis.get(key);
        return stringToBean(string, clazz);
    }finally {
        returnToPool(jedis);    // close the connect if it exists.
    }
}
```

2. As mentioned before, we want to use FastJson and Jedis together, which means if we want to save an object, we need to change it to json format and load a object, we first get a string and then decode it to a object, I can use the following two methods:
```Java
private <T> String beanToString(T value, Class<T> clazz){
    if(value == null || clazz != value.getClass()){
        return null;
    }else if(clazz == int.class || clazz == Integer.class){
        return "" + value;
    }else if(clazz == String.class){
        return (String) value;
    }else if(clazz == long.class || clazz == Long.class){
        return "" + value;
    }else{
        return JSON.toJSONString(value);
    }
}
private <T> T stringToBean(String string, Class<T> clazz) {
    if(string == null || string.length() <= 0 || clazz == null){
        return null;
    }else if(clazz == int.class || clazz == Integer.class){
        return (T)Integer.valueOf(string);
    }else if(clazz == String.class){
        return (T)string;
    }else if(clazz == long.class || clazz == Long.class){
        return (T)Long.valueOf(string);
    }else{
        return JSON.toJavaObject(JSON.parseObject(string), clazz);
    }
}
```

### Reference
1. [ApplicationContext.xml](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)

