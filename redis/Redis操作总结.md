## Redis操作总结

[redis常用命令](https://github.com/redisson/redisson/wiki/11.-redis%E5%91%BD%E4%BB%A4%E5%92%8Credisson%E5%AF%B9%E8%B1%A1%E5%8C%B9%E9%85%8D%E5%88%97%E8%A1%A8)

#### 限流器(RateLimiter)

基于Redis的分布式限流器(RateLimiter)可以用来在分布式环境下现在请求方的调用频率。既适用于不同Redisson实例下的多线程限流，也适用于相同Redisson实例下的多线程限流。该算法不保证公平性。除了同步接口外，还提供了异步(Async)、反射式(Reactive)和RxJava2标准的接口。

```
RRateLimiter rateLimiter = redisson.getRateLimiter("myRateLimiter");
//初始化
//最大流速 = 每1秒钟产生10个令牌
rateLimiter.trySetRate(RateType.OVERALL,10,1,RateIntervalUnit.SECONDS);
CountDownLatch latch = new CountDownLatch(2);
limiter.acquire(3);

Thread t = new Thread(() ->{
	limiter.acquire(2);
})

```

#### 分布式集合 - 映射

基于Redis的Redisson的分布式映射结构的RMap java对象实现了 java.util.concurrent.ConcurrentMap接口和java.util.Map接口。与HashMap不同的是，RMap保持了元素的插入顺序。该对象的最大容量受Redis限制，最大元素数量是4 294 967 295个。

```
help @generic
```

#### 字符串指令

字符串结构，其实是Redis中最基础的K-V结构。其键和值都是字符串。类似Java的Map<String,String>



















