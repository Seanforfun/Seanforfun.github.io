---
layout: post
title:  "Redis| Data structure in Redis"
date:   2018-01-21 14:44
author: Botao Xiao
categories: Redis
comment: true
description: Redis is a kind of NoSQL(Not only sql), which means redis is not a relation based database. Redis works on RAM so we always setup a redis server (not in the EE project) as a first level or second level database. In this post, I will include the five data structure that can be saved in Redis.
---
# Redis| Data structure in Redis

## Introduction
Redis is a kind of NoSQL(Not only sql), which means redis is not a relation based database. Redis works on RAM so we always setup a redis server (not in the EE project) as a first level or second level database. In this post, I will include the five data structure that can be saved in Redis.

### Five Data structrues in Redis
![Imgur](https://i.imgur.com/Aa5i665.png)
1. STRING
    * String in Redis is a k-v saving format, there are three basic actions to redis: GET, SET, DEL.
    * ![Imgur](https://i.imgur.com/BGIEToQ.png)

2. LIST: LIST is a bi-directional linkedlist, which is a container of elements. In the list, we can save duplicate strings.
    * LPUSH/RPUSH: Allow us to push string into the list from left and right end.
    * LPOP/RPOP: Allow us to pop string from left and right end.
    * LINDEX: Get the string on index in the list.
    * LRANGE: Return all of the elements in the given range. For example, lrange 0, -1 # 0 means the start of the list and -1 means the end of the current list.
    ![Imgur](https://i.imgur.com/DDXQrtE.png)

3. SET: Set data structure guaranteed that no dulicate string is contained. And very similary as set in Java, the strings saved in SET is unordered.
    * SADD: Add string into the set.
    * SREM: Remove string from the set.
    * SISMEMBER: Check if one string is saved in the set, same as set.containsKey() in java.
    * SMEMBERS: Show all members in current set. Since we need to traversal all strings in the set but we need to be very careful when using this command.
    ![Imgur](https://i.imgur.com/X7BLntz.png)
    
4. HASH
5. ZSET

## Reference
1. [Redis in Action](https://redislabs.com/community/ebook/)