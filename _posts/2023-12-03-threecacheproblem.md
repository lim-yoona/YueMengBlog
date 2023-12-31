---
layout: post
title: 缓存击穿、缓存穿透与缓存雪崩
subtitle: 分布式缓存常见问题
date: 2023-12-03 19:50:00 +0800
categories: 分布式缓存
author: 月梦
cover: 'https://z1.ax1x.com/2023/12/01/pisYxbD.jpg'
cover_author: 'Pexels'
cover_author_link: 'https://www.pexels.com/zh-cn/'
tags:
- 分布式缓存  
---

缓存是计算机系统中应用非常广泛的技术，最经典的，操作系统中处处是缓存，缓存可以大大提升数据访问速率。    

在业务中，数据库（`MySQL`）面对大量的并发请求，会出现两个问题:  
1. 每次请求都需要查询数据库，速度很慢；  
2. 数据库无法承受如此大的请求流量，可能引起数据库宕机；

为解决这两个问题，一般会在内存中设置缓存，通常是使用`Redis`作为缓存，对于数据库的查询请求首先查询缓存，如果查不到，再去查询数据库。  

引入缓存之后又会面临三个新的问题，即缓存击穿、缓存穿透以及缓存雪崩。  

## 缓存击穿
当缓存中没有某个数据，但是数据库中有这个数据时，对于数据的访问会直接访问数据库，于是——  

一个热点`key`每时每刻都在接受大量的并发访问，当这个热点`key`的缓存过期时，大量的并发请求同时涌入到数据库中，导致后端数据库的压力陡然增大，引发`缓存击穿`。

概括来讲，就是由于缓存`key`过期，导致大量请求涌入后端数据库而造成数据库压力骤然增大。  

### 如何解决缓存击穿？
1. 既然缓存击穿是由于热点缓存`key`的过期导致的，那么一种方法是设置热点`key`永不过期。  
2. 使用分布式锁，当查询缓存未命中时，首先申请分布式锁，再去访问后端数据库，拿到数据之后缓存到缓存中；别的请求得不到锁，就无法请求后端数据库，待解锁后，直接查询缓存，从而有效地解决了大量请求涌入后端数据库的问题。  

## 缓存穿透
缓存穿透指查询缓存未命中，同时后端数据库中也没有这个数据，那么Mysql就只能返回一个空对象，表示此次查询失败。如果这样的请求非常多，又或者有攻击者恶意地持续不断发出这样的查询请求，会给后端数据库带来很大的压力甚至崩溃，这就是`缓存穿透`。  

### 如何解决缓存穿透？
1. 缓存空对象。当返回空对象的请求到达时，缓存一个空对象，这样下一次同样的请求到达，就会查询缓存，而不会访问后端数据库。缺点就是，如果缓存大量空对象，占用了缓存的空间。  
2. 布隆过滤器。利用布隆过滤器可以判断元素不存在的特性，将一些数据存储在布隆过滤器中，用户访问数据时，首先查询布隆过滤器，如果数据不存在，那么就拒绝查询，从而避免了大量对于空对象的查询。  

## 缓存雪崩
缓存雪崩是指为一批缓存`key`设置了相同的过期时间，那么当这个过期时间到达时，这些缓存`key`同时失效，从而导致大量的访问涌入后端数据库，造成后端数据库压力陡然增大，形成`缓存雪崩`。  

有两种情况会造成`缓存雪崩`：  
1. 多个缓存`key`同时过期  
2. 缓存系统宕机

### 如何解决缓存雪崩？
解决大批`key`同时过期：  
1. 设置多级缓存，这样即使缓存失效或者多个缓存`key`同时过期，也不会造成`缓存雪崩`.  
2. 使用锁或者队列，避免大量的并发请求访问后端数据库  
3. 随机设置缓存失效时间  
4. 不设置过期时间  
5. 在分布式负载均衡场景下，可以使用一致性哈希算法来解决由于分布式集群容量动态改变而带来的`缓存雪崩`  

解决缓存系统宕机问题：  
1. 服务熔断或请求限流机制。服务熔断是指当缓存系统宕机时，暂停业务对于缓存系统的访问，直接返回错误；请求限流机制只允许少部分请求可以访问后端数据库。  
2. 构建Redis缓存高可用集群。

