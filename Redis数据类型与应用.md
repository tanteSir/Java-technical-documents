## Redis数据类型与应用

#### 一、Redis常用的数据类型

```java
reids常用的数据类型有以下五种
    1.String
    2.Hash
    3.List
    4.Set
    5.Sorted Set
这里先看看Redis内部内存管理图
```

![](../知识库\图库\Redis内存管理图.png)

```java
	Redis使用RedisObject对象来表示所有的key-value，其中type代表value是什么类型，encoding代表不同数据类型在redis内部的存储方式
```

#### 二、各种数据类型的应用和实现方式

##### 1、String

```java
String是简单的 Key-Value类型，value可以是String或者数值
常用命令：
	get、set、incr、decr、mget等
应用场景：
	普通的 key-value缓存存储，常规计数（会员数等）
实现方式：
	String在 Redis中默认存储为字符串，被 RedisObject引用。当使用incr、decr等操作时，就会转变为数  值类型，此时encoding类型为int
```

##### 2、Hash

```
常用命令：hget、hset、hgetall等
应用场景：
	拿获取用户对象举例，以用户ID为Key，存储的value包括用户多方面信息，如果用普通的key-value方式，目前有两种方式
	1.ID为Key，其他信息封装成对象以序列化方式存储。缺点是：增加序列化、反序列化的开销，每次修改信息		  都要拿出整个对象，并且需要对并发进行保护
	2.用户有多少属性就有多少key-value对，用ID+属性名作为唯一表示，虽然省去序列化、反序列化开销，但	    	  是存放大量重复的ID也会消耗内存
	
    所以这里使用 Hash，Redis的 Hash实际是内部存储的 value为 HashMap，并提供获取这个Map成员的接口，如下图：
```

![](../知识库\图库\Redis的Hash存储结构.png)

```java
	在这里，Key仍然是ID，value是一个Map，这个Map的key是属性名，value是属性值，这样对数据的修改和存取都可以直接通过内部Map的Key，也就是通过 Key(用户ID) + field(属性标签)就可以操作数据，不用读取整个对象与序列化、反序列化的不必要开销

使用场景：存储部分变更数据，如用户信息
实现方式：
	Redis Hash内部对应的 Value是一个HashMap，实际会有两种不同实现。
	Hash成员比较少时，Redis会采取类似一维数组来紧凑存储，对应的value redisObject的encoding为 zipmap；当成员数增大时才会转化为真正的HashMap，此时encoding为int
```

##### 3、List

```
常用命令：lpush、rpush、lpop、rpop、lrange等
应用场景：
	比如微博的关注列表、粉丝列表都可以用 Redis的list结构实现。
	List就是链表，使用List我们可以实现最新消息排行等功能，List的另一个应用就是消息队列，可以利用list的 PUSH操作，将任务存于List中，然后工作线程在POP操作将任务取出。

实现方式：
	Redis的List是一个双向链表，支持反向查找和遍历，更方便操作不过带来了额外的内存开销，发送缓冲队列也是用的这个数据结构
	Redis的List是每个子元素都是String类型的，可以通过push操作和pop操作，从链表的头部或尾部添加或删除元素，这样List即可以作为栈，也可以作为队列

使用场景：
	消息队列，使用sorted set甚至可以构建有优先级的消息队列	
```

```java
//把当前登录人添加到链表里
ret = r.lpush("login:last_login_times", uid)

//保持链表只有N位
ret = redis.ltrim("login:last_login_times", 0, N-1)

//获得前N个最新登陆的用户Id列表
last_login_list = r.lrange("login:last_login_times", 0, N-1)
```

##### 4、Set

```java
常用命令：sadd、spop、smembers、sunion等
应用场景：
	与List相似，区别是Set可以自动排重，可以去掉重复的组合。Set是String类型的无序集合，通过hashTable实现的，可以获取交集、并集、差集等
	
实现方式：
	set内部是 value值永远为 null的HashMap,通过计算hash值来快速排重的
使用场景：
	交集、并集、差集等
```

##### 5、Sorted Set

```java
常用命令：zadd、zrange、zrem、zcard
使用场景：
	与Set类似，区别是sorted set可以通过提供一个优先级参数来进行自动排序。比如全班同学成绩，value是学号，scope是得分，这样数据插入时就已经进行排序了
实现方式：
	内部使用 HashMap和跳跃表(SkipList)来保证数据有序存储，HashMap里放的是成员到 scope的映射，而跳跃表存放所有成员，以此是实现高的查找效率
```



#### 三、Redis实际应用场景

##### 1、缓存

```
	用于热点数据存储（经常被查询，不常修改）
读取顺序：先访问redis寻找数据，若有找到则返回redis中的数据，若没找到则进数据库查找，将结果存入redis中，			再返回结果
修改或删除顺序：先删除或修改数据库中数据，再删除或修改redis中数据
```

##### 2、计数器

```
	比如点击数等应用，因为它是单线程，可以避免并发问题，反应也快
```

##### 3、队列

```
	与activeMQ相似
```

##### 4、位操作（大数据处理）

```
	用于数据量上亿的情况，比如腾讯十亿用户要在几毫秒内查询到某用户的是否在线，该怎么做？这里如果用 Key-value挨个存储会消耗大量内存，所以只能用位操作：setbit、getbit、bitcount

原理：在Redis内创建一个足够长的数组，每个元素值只能用0或1表示，index下标就是用户ID，这样就能快速访问到用户的登录状态了
```

##### 5、分布式锁与单线程机制

```java
	验证前端重复请求：每次将请求者IP、参数、接口名作为Key存于Redis中，并设定有效期，等下次请求过来时先看Redis中有无存在此数据，从而验证是不是这一定时间内的重复请求
	
	秒杀：因为Redis是单线程的，所以把商品库存存于Redis中，可以放在数据库被爆破以及出现多卖的局面
```

##### 6、最新列表

```java
	比如获取最新的新闻列表，总量很大的情况下就尽量别用 select XXX from XXX limit 10，可以使用Redis的lpush命令构建list，把数据一个个放进链表头部就可以了。
	假如内存清理掉了，也可以用MySQL查询并初始化一个list进Redis中
```

##### 7、排行版

```java
	谁分数高谁在上面，可以用zadd，scope为分数，value为学号
```

