---
layout: post
title:  "Backend | Primary Key Generation Methods"
date:   2019-02-01 09:34
author: Botao Xiao
comment: true
categories: JavaBackend
description: Primary key is the unique id for a instance saving in the database. In this article, I want to discribe three methods to generate the primary key, which are UUID, database auto_increment and snowflake algorithm from Twitter.
---

Primary key is the unique id for a instance saving in the database. In this article, I want to discribe three methods to generate the primary key, which are UUID, database auto_increment and snowflake algorithm from Twitter.

### UUID(Universal Unique Id)
#### Introduction
1. The length of UUID is 32 bytes and we can ensure that UUID is unique.
2. UUID is not time related, it means UUID won't represent the generation order of two UUIDs.

#### Generation method:
Generation method of UUID is super simple, we can directly using UUID library provided in Java:
```Java
String uuid = UUID.randomUUID().toString();
// Result:
// b83e4e60-bd10-46f6-8581-71ece0a31646
// ab20e2e6-4aa3-459c-83c3-761b5e117265
```

#### Cons and pros:
1. Generation is super simple and well encapsulated. Only one line code is required.
2. It lead to low efficiency of database insert.
  * Most of ORM database are based on B+ tree.
  * B+ tree will spilt B+ index for inserting data.
  ![Imgur](https://i.imgur.com/jBFZXOd.png)
  * Since UUID is not time related, it will create a lot of unfilled split index, it will slow down the database.

### Database auto_increment
#### Introduction
1. Database will help us maintain the increment of primary key, we can set up the start index and step size of id. It can be applied to distributed system(using different step size).

#### Generation method:
Here shows the auto_increment primary in mysql.
```MYSQL
Create table user(
  id bigint primary key auto_increment not null,
  name varchar(100) not null
);
```

#### Cons and pros
1. The database will help us maintaining the primary key.
2. Generation of id highly based on database so it will decrease the efficiency of database.
3. Once the database is down, all services are down.

### Twitter snowflake
#### Introduction
1. When Twitter migrate their database from mysql to Cassandra, they found Cassandra didn't have the function of auto_increment, so they developed snowflake for generating id in memory.
2. Composition of snowflake id:
```
0 - 0000000000 0000000000 0000000000 0000000000 0 - 00000 - 00000 - 000000000000
```
  1. 1 Index bit: In java, highest bit shows current number is positive or negative. So normally 0.
  2. 41 bits for time stamp. The value represents in unit of milliseconds. normally we won't save current time, we always save the time different from a global beginning time.
  3. 10 bits for location. first 5 bits for data center and last 5 bits for machine id.
  4. 12 bits for serial id. It supports for 4096 ids in 1 millisecond.

#### Generation Method
```Java
/**
 * 描述: Twitter的分布式自增ID雪花算法snowflake (Java版)
 *
 * @author yanpenglei
 * @create 2018-03-13 12:37
 **/
public class SnowFlake {

    /**
     * 起始的时间戳
     */
    private final static long START_STMP = 1480166465631L;

    /**
     * 每一部分占用的位数
     */
    private final static long SEQUENCE_BIT = 12; //序列号占用的位数
    private final static long MACHINE_BIT = 5;   //机器标识占用的位数
    private final static long DATACENTER_BIT = 5;//数据中心占用的位数

    /**
     * 每一部分的最大值
     */
    private final static long MAX_DATACENTER_NUM = -1L ^ (-1L << DATACENTER_BIT);
    private final static long MAX_MACHINE_NUM = -1L ^ (-1L << MACHINE_BIT);
    private final static long MAX_SEQUENCE = -1L ^ (-1L << SEQUENCE_BIT);

    /**
     * 每一部分向左的位移
     */
    private final static long MACHINE_LEFT = SEQUENCE_BIT;
    private final static long DATACENTER_LEFT = SEQUENCE_BIT + MACHINE_BIT;
    private final static long TIMESTMP_LEFT = DATACENTER_LEFT + DATACENTER_BIT;

    private long datacenterId;  //数据中心
    private long machineId;     //机器标识
    private long sequence = 0L; //序列号
    private long lastStmp = -1L;//上一次时间戳

    public SnowFlake(long datacenterId, long machineId) {
        if (datacenterId > MAX_DATACENTER_NUM || datacenterId < 0) {
            throw new IllegalArgumentException("datacenterId can't be greater than MAX_DATACENTER_NUM or less than 0");
        }
        if (machineId > MAX_MACHINE_NUM || machineId < 0) {
            throw new IllegalArgumentException("machineId can't be greater than MAX_MACHINE_NUM or less than 0");
        }
        this.datacenterId = datacenterId;
        this.machineId = machineId;
    }

    /**
     * 产生下一个ID
     *
     * @return
     */
    public synchronized long nextId() {
        long currStmp = getNewstmp();
        if (currStmp < lastStmp) {
            throw new RuntimeException("Clock moved backwards.  Refusing to generate id");
        }

        if (currStmp == lastStmp) {
            //相同毫秒内，序列号自增
            sequence = (sequence + 1) & MAX_SEQUENCE;
            //同一毫秒的序列数已经达到最大
            if (sequence == 0L) {
                currStmp = getNextMill();
            }
        } else {
            //不同毫秒内，序列号置为0
            sequence = 0L;
        }

        lastStmp = currStmp;

        return (currStmp - START_STMP) << TIMESTMP_LEFT //时间戳部分
                | datacenterId << DATACENTER_LEFT       //数据中心部分
                | machineId << MACHINE_LEFT             //机器标识部分
                | sequence;                             //序列号部分
    }

    private long getNextMill() {
        long mill = getNewstmp();
        while (mill <= lastStmp) {
            mill = getNewstmp();
        }
        return mill;
    }

    private long getNewstmp() {
        return System.currentTimeMillis();
    }

    public static void main(String[] args) {
        SnowFlake snowFlake = new SnowFlake(2, 3);

        long start = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {
            System.out.println(snowFlake.nextId());
        }

        System.out.println(System.currentTimeMillis() - start);
    }
}
```

#### Cons and pros
1. Time related, and distributed system available. High efficiency for data insert.
2. Everything is not RAM and not related to database, so it is easy for database migration.
3. It is related to system time so if machine time is incorrect, it will cause some problems.

### Reference
1. [SnowFlake](https://github.com/souyunku/SnowFlake)
2. [漫画：什么是SnowFlake算法?](https://www.sohu.com/a/232008315_453160)
