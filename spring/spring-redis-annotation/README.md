# spring 整合 redis （注解方式）

## 目录<br/>
<a href="#一说明">一、说明</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#11--Redis-客户端说明">1.1  Redis 客户端说明</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#12-Redis可视化软件">1.2 Redis可视化软件 </a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#13-项目结构说明">1.3 项目结构说明</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#13-依赖说明">1.3 依赖说明</a><br/>
<a href="#二spring-整合-jedis">二、spring 整合 jedis</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#21-新建基本配置文件和其映射类">2.1 新建基本配置文件和其映射类</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#22-单机配置">2.2 单机配置</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#23-集群配置">2.3 集群配置</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#24-单机版本测试用例">2.4 单机版本测试用例</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#25-集群版本测试用例">2.5 集群版本测试用例</a><br/>
<a href="#三spring-整合-redisson">三、spring 整合 redisson</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#21-单机配置">2.1 单机配置</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#22-集群配置">2.2 集群配置</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#23-存储基本类型测试用例">2.3 存储基本类型测试用例</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#24-存储实体对象测试用例">2.4 存储实体对象测试用例</a><br/>
<a href="#附Redis的数据结构和操作命令">附：Redis的数据结构和操作命令</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#11-预备">1.1 预备</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#111-全局命令">1.1.1 全局命令</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#112-数据结构和内部编码">1.1.2 数据结构和内部编码</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#113-单线程架构">1.1.3 单线程架构</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#12-字符串">1.2 字符串</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#13-哈希">1.3 哈希</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#14-列表">1.4 列表</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#15-集合">1.5 集合</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#16-有序集合">1.6 有序集合</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#17-键管理">1.7 键管理</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#171-单个键管理">1.7.1 单个键管理</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#1键重命名">1.键重命名  </a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#2-随机返回键">2. 随机返回键 </a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#3键过期">3.键过期</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#172-键遍历">1.7.2 键遍历</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#1-全量键遍历">1. 全量键遍历</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#2-渐进式遍历">2. 渐进式遍历</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#173-数据库管理">1.7.3 数据库管理</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#1切换数据库">1.切换数据库</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="#2flushdb/flushall">2.flushdb/flushall </a><br/>
## 正文<br/>


## 一、说明

### 1.1  Redis 客户端说明

关于 spring 整合 mybatis 本用例提供两种整合方法：

1. jedis: 官方推荐的 java 客户端，能够胜任 redis 的大多数基本使用；
2. redisson：也是官方推荐的客户端，比起 jedis 提供了更多高级的功能，比如分布式锁、集合数据切片等功能。同时提供了丰富而全面的中英文版本的 wiki。

注：关于 redis 其他语言官方推荐的客户端可以在[客户端](http://www.redis.cn/clients.html) 该网页查看，其中官方推荐的用了黄色星星:star:标注。

<div align="center"> <img src="https://github.com/heibaiying/spring-samples-for-all/blob/master/pictures/redis官方推荐客户端.png"/> </div>



### 1.2 Redis可视化软件 

推荐**Redis Desktop Manager** 作为可视化查看工具，可以直观看到用例中测试关于存储实体对象序列化的情况。

### 1.3 项目结构说明

1. jedis 和 redisson 的配置类和单元测试分别位于 config 和 test 下对应的包中，其中集群的配置类以 cluster 开头。
2. 实体类 Programmer.java 用于测试 Redisson 序列化与反序列化

<div align="center"> <img src="https://github.com/heibaiying/spring-samples-for-all/blob/master/pictures/spring-redis-annotation.png"/> </div>



### 1.3 依赖说明

除了 spring 的基本依赖外，需要导入 jedis 和 redisson 对应的客户端依赖包

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.0.0</version>
</dependency>
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.9.1</version>
</dependency>
<!--redisson 中部分功能依赖了netty  -->
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.32.Final</version>
</dependency>
```



## 二、spring 整合 jedis

#### 2.1 新建基本配置文件和其映射类

```properties
redis.host=127.0.0.1
redis.port=6379
# 连接超时时间
redis.timeout=2000
# 最大空闲连接数
redis.maxIdle=8
# 最大连接数
redis.maxTotal=16
```

```java
@Configuration
@PropertySource(value = "classpath:jedis.properties")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class RedisProperty {

    @Value("${redis.host}")
    private String host;
    @Value("${redis.port}")
    private int port;
    @Value("${redis.timeout}")
    private int timeout;
    @Value("${redis.maxIdle}")
    private int maxIdle;
    @Value("${redis.maxTotal}")
    private int maxTotal;
}
```



#### 2.2 单机配置

```java
/**
 * @author : heibaiying
 * @description : Jedis 单机配置
 */
@Configuration
@ComponentScan(value = "com.heibaiying.*")
public class SingleJedisConfig {

    @Bean
    public JedisPool jedisPool(RedisProperty property) {
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setMaxIdle(property.getMaxIdle());
        poolConfig.setMaxTotal(property.getMaxTotal());
        return new JedisPool(poolConfig, property.getHost(), property.getPort(), property.getTimeout());
    }

    @Bean(destroyMethod = "close")
    public Jedis jedis(JedisPool jedisPool) {
        return jedisPool.getResource();
    }
}
```

#### 2.3 集群配置

```java
@Configuration
@ComponentScan(value = "com.heibaiying.*")
public class ClusterJedisConfig {

    @Bean
    public JedisCluster jedisCluster(RedisProperty property) {
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setMaxIdle(property.getMaxIdle());
        poolConfig.setMaxTotal(property.getMaxTotal());
        Set<HostAndPort> nodes = new HashSet<HostAndPort>();
        nodes.add(new HostAndPort("127.0.0.1", 6379));
        nodes.add(new HostAndPort("127.0.0.1", 6380));
        return new JedisCluster(nodes, 2000);
    }
}
```

#### 2.4 单机版本测试用例

1.需要注意的是，对于 jedis 而言，单机版本和集群版本注入的实例是不同的；

2.jedis 本身并不支持序列化于反序列化操作，如果需要存储实体类，需要序列化后存入。(redisson 本身就支持序列化于反序列化，详见下文)

```java
/**
 * @author : heibaiying
 * @description :redis 单机版测试
 */
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = SingleJedisConfig.class)
public class JedisSamples {

    @Autowired
    private Jedis jedis;

    @Test
    public void Set() {
        jedis.set("hello", "spring annotation");
    }

    @Test
    public void Get() {
        String s = jedis.get("hello");
        System.out.println(s);
    }

    @Test
    public void setEx() {
        String s = jedis.setex("spring", 10, "我会在 10 秒后过期");
        System.out.println(s);
    }

}
```

#### 2.5 集群版本测试用例

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = ClusterJedisConfig.class)
public class JedisClusterSamples {

    @Autowired
    private JedisCluster jedisCluster;

    @Test
    public void Set() {
        jedisCluster.set("hello", "spring");
    }

    @Test
    public void Get() {
        String s = jedisCluster.get("hello");
        System.out.println(s);
    }

    @Test
    public void setEx() {
        String s = jedisCluster.setex("spring", 10, "我会在 10 秒后过期");
        System.out.println(s);
    }


}
```



## 三、spring 整合 redisson

#### 2.1 单机配置

```java
/**
 * @author : heibaiying
 * @description : redisson 单机配置
 */
@Configuration
public class SingalRedissonConfig {

    @Bean
    public RedissonClient redissonClient() {
        Config config = new Config();
        config.setTransportMode(TransportMode.NIO);
        config.useSingleServer().setAddress("redis://127.0.0.1:6379");
        return Redisson.create(config);
    }

}
```

#### 2.2 集群配置

```java
/**
 * @author : heibaiying
 * @description : redisson 集群配置
 */
@Configuration
public class ClusterRedissonConfig {

    //@Bean
    public RedissonClient redissonClient() {
        Config config = new Config();
        config.useClusterServers()
                .setScanInterval(2000) // 集群状态扫描间隔时间，单位是毫秒
                //可以用"rediss://"来启用 SSL 连接
                .addNodeAddress("redis://127.0.0.1:6379", "redis://127.0.0.1:6380")
                .addNodeAddress("redis://127.0.0.1:6381");
        return Redisson.create(config);
    }

}
```

#### 2.3 存储基本类型测试用例

1. 这里需要注意的是，对于 Redisson 而言， 单机和集群最后在使用的时候注入的都是 RedissonClient，这和 jedis 是不同的。

```java
/**
 * @author : heibaiying
 * @description :redisson 操作普通数据类型
 */

@RunWith(SpringRunner.class)
@ContextConfiguration(classes = SingalRedissonConfig.class)
public class RedissonSamples {

    @Autowired
    private RedissonClient redissonClient;

    @Test
    public void Set() {
        // key 存在则更新 不存在则删除
        RBucket<String> rBucket = redissonClient.getBucket("redisson");
        rBucket.set("annotation Value");
        redissonClient.shutdown();
    }

    @Test
    public void Get() {
        // key 存在则更新 不存在则删除
        RBucket<String> rBucket = redissonClient.getBucket("redisson");
        System.out.println(rBucket.get());
    }

    @Test
    public void SetEx() {
        // key 存在则更新 不存在则删除
        RBucket<String> rBucket = redissonClient.getBucket("redissonEx");
        rBucket.set("我在十秒后会消失", 10, TimeUnit.SECONDS);
    }


    @After
    public void close() {
        redissonClient.shutdown();
    }
}

```

#### 2.4 存储实体对象测试用例

```java
/**
 * @author : heibaiying
 * @description :redisson 对象序列化与反序列化
 */


@RunWith(SpringRunner.class)
@ContextConfiguration(classes = SingalRedissonConfig.class)
public class RedissonObjectSamples {

    @Autowired
    private RedissonClient redissonClient;

    // Redisson 的对象编码类是用于将对象进行序列化和反序列化 默认采用 Jackson

    @Test
    public void Set() {
        RBucket<Programmer> rBucket = redissonClient.getBucket("programmer");
        rBucket.set(new Programmer("xiaoming", 12, 5000.21f, new Date()));
        redissonClient.shutdown();
        //存储结果: {"@class":"com.heibaiying.com.heibaiying.bean.Programmer","age":12,"birthday":["java.util.Date",1545714986590],"name":"xiaoming","salary":5000.21}
    }

    @Test
    public void Get() {
        RBucket<Programmer> rBucket = redissonClient.getBucket("programmer");
        System.out.println(rBucket.get());
    }

    @After
    public void close() {
        redissonClient.shutdown();
    }
}
```

## 附：Redis的数据结构和操作命令

### 1.1 预备

#### 1.1.1 全局命令

1. 查看所有键： **keys \*** 

2. 查看键总数：**dbsize**  

3. 检查键是否存在：**exists key**

4. 删除键：**del key [key ...]**   支持删除多个键

5. 键过期：**expire key seconds**

   ttl 命令会返回键的剩余过期时间， 它有 3 种返回值：

   - 大于等于 0 的整数： 键剩余的过期时间。
   - -1： 键没设置过期时间。
   - -2： 键不存在 

6. 键的数据结构 **type key**

#### 1.1.2 数据结构和内部编码

type 命令实际返回的就是当前键的数据结构类型， 它们分别是：**string**（字符串） 、 **hash**（哈希） 、 **list**（列表） 、 **set**（集合） 、 **zset**（有序集合） 

#### 1.1.3 单线程架构

1. 纯内存访问， Redis 将所有数据放在内存中， 内存的响应时长大约为 100 纳秒， 这是 Redis 达到每秒万级别访问的重要基础。
2. 非阻塞 I/O， Redis 使用 epoll 作为 I/O 多路复用技术的实现， 再加上 Redis 自身的事件处理模型将 epoll 中的连接、 读写、 关闭都转换为事件， 不在网络 I/O 上浪费过多的时间， 如图 2-6 所示。 
3. 单线程避免了线程切换和竞态产生的消耗。 

### 1.2 字符串

| 作用                   | 格式                                                         | 参数或示例                                                   |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 设置值                 | set key value \[ex seconds]\[px milliseconds][nx\|xx] setnx setex | ex seconds： 为键设置秒级过期时间。 <br/>px milliseconds： 为键设置毫秒级过期时间。<br/>nx： 键必须不存在， 才可以设置成功， 用于添加。<br/>xx： 与 nx 相反， 键必须存在， 才可以设置成功， 用于更新。 |
| 获取值                 | get key                                                      | r 如果获取的键不存在 ，则返回 nil(空)                          |
| 批量设置               | mset key value [key value ...]                               | mset a 1 b 2 c 3 d 4                                         |
| 批量获取值             | mget key [key ...]                                           | mget a b c d                                                 |
| 计数                   | incr key decr key incrby key increment（指定数值自增）<br/>decrby key decrement（指定数值自减）<br/>incrbyfloat key increment （浮点数自增） | 值不是整数， 返回错误。 值是整数， 返回自增或自减后的结果。<br/>键不存在，创建键，并按照值为 0 自增或自减， 返回结果为 1。 |
| 追加值                 | append key value                                             | 向字符串的默认追加值                                         |
| 字符串长度             | strlen key                                                   | 获取字符串长度，中文占用三个字节                             |
| 设置并返回原值         | getset key value                                             |                                                              |
| 设置指定位置的租字符串 | setrange key offeset value                                   |                                                              |
| 获取部分字符串         | getrange key start end                                       |                                                              |

### 1.3 哈希

| 作用                      | 格式                                                         | 参数或示例                                                   |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 设置值                    | hset key field value                                         | hset user:1 name tom<br/>hset user:1 age 12                  |
| 获取值                    | hget key field                                               | hget user:1 name                                             |
| 删除 field                 | hdel key field [field ...]                                   |                                                              |
| 计算 field 个数             | hlen key                                                     |                                                              |
| 批量设置或获取 field-value | hmget key field [field]<br/>hmset key field value [field value...] | hmset user:1 name mike age 12 city tianjin<br/>hmget user:1 name city |
| 判断 field 是否存在         | hexists key field                                            |                                                              |
| 获取所有 field             | hkeys key                                                    |                                                              |
| 获取所有 value             | hvals key                                                    |                                                              |
| 获取所有的 filed-value     | hgetall key                                                  | 如果哈希元素个数比较多， 会存在阻塞 Redis 的可能。<br/>获取全部 可以使用 hscan 命令， 该命令会渐进式遍历哈希类型 |
| 计数                      | hincrby key field<br/>hincrbyfloat key field                 |                                                              |

### 1.4 列表

| 作用     | 格式                                                         | 参数或示例                                                   |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 增       | 左侧插入：lpush key value [value ...] 右侧插入：rpush key value [value ...] 某个指定元素前后插入：linsert key before\|after pivot value |                                                              |
| 查       | 获取指定范围内的元素列表：lrange key start end 获取列表指定索引下标的元素：lindex key index 获取列表指定长度：llen key | lrange listkey 0 -1                                          |
| 删       | 从列表左侧弹出元素：lpop key 从列表右侧弹出元素：rpop key 删除指定元素：lrem key count value 截取列表：ltrim key start end | count>0， 从左到右， 删除最多 count 个元素。<br/>count<0， 从右到左， 删除最多 count 绝对值个元素。<br/>count=0， 删除所有 |
| 改       | 修改指定索引下标的元素：lset key index newValue              |                                                              |
| 阻塞操作 | blpop key [key ...] timeout brpop key [key ...] timeout      | key[key...]： 多个列表的键。 timeout： 阻塞时间\|等待时间（单位： 秒） |



### 1.5 集合

集合（set） 类型也是用来保存多个的字符串元素， 但和列表类型不一样的是， **集合中不允许有重复元素**， 并且集合中的元素是无序的， **不能通过索引下标获取元素**。  

**集合内操作**：

| 作用                 | 格式                           | 参数或示例                                |
| -------------------- | ------------------------------ | ----------------------------------------- |
| 添加元素             | sadd key element [element ...] | 返回结果为添加成功的元素个数              |
| 删除元素             | srem key element [element ...] | 返回结果为成功删除的元素个数              |
| 计算元素个数         | scard key                      |                                           |
| 判断元素是否在集合中 | sismember key element          |                                           |
| 随机返回             | srandmember key [count]        | 随机从集合返回指定个数元素，count 默认为 1 |
| 从集合随机弹出元素   | spop key                       | srandmember 不会从集合中删除元素，spop 会 |
| 获取集合中所有元素   | smembers key                   | 可用 sscan 代替                            |

**集合间操作**：

| 作用                         | 格式                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| 求多个集合的交集             | sinter key [key ...]                                         |
| 求多个集合的并集             | suinon key [key ...]                                         |
| 求多个集合的差集             | sdiff key [key ...]                                          |
| 将交集、并集、差集的结果保存 | sinterstore destination key [key ...] <br/>suionstore destination key [key ...]<br/>sdiffstore destination key [key ...] |

### 1.6 有序集合

有序集合中的元素可以排序。 但是它和列表使用索引下标作为排序依据不同的是， 它给每个元素设置一个分数（score） 作为排序的依据。  

**集合内操作**：

| 作用                     | 格式                                                         | 参数或示例                                                   |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 添加成员                 | zadd key score member [score member ...]                     | nx： member 必须不存在， 才可设置成功， 用于添加。<br> xx： member 必须存在， 才可以设置成功， 用于更新。<br/>ch： 返回此次操作后， 有序集合元素和分数发生变化的个数<br/>incr： 对 score 做增加， 相当于后面介绍的 zincrby。 |
| 计算成员个数             | zcard key                                                    |                                                              |
| 计算某个成员的分数       | zscore key member                                            |                                                              |
| 计算某个成员的排名       | zrank key member  zrevrank key member                        | zrank 是从分数从低到高返回排名， zrevrank 反之。               |
| 删除成员                 | zrem key member [member ...]                                 |                                                              |
| 增加成员分数             | zincrby key increment member                                 | zincrby user:ranking 9 tom                                   |
| 返回指定排名范围的成员   | zrange key start end [withscores] zrange key start end [withscores] | zrange 是从低到高返回， zrevrange 反之。                       |
| 返回指定分数范围内的成员 | zrangebyscore key min max \[withscores][limit offset count] zrevrangebyscore key max min \[withscores][limit offset count] | 其中 zrangebyscore 按照分数从低到高返回， zrevrangebyscore 反之。 [limit offset count]选项可以限制输出的起始位置和个数： 同时 min 和 max 还支持开区间（小括号） 和闭区间（中括号） ， -inf 和 +inf 分别代表无限小和无限大 |
| 删除指定排名内的升序元素 | zremrangerank key start end                                  |                                                              |
| 删除指定分数范围的成员   | zremrangebyscore key min max                                 |                                                              |

**集合间操作**：

| 作用 | 格式                                                         |
| ---- | ------------------------------------------------------------ |
| 交集 | zinterstore destination numkeys key \[key ...]  [weights weight [weight ...]] \[aggregate sum\|min\|max] |
| 并集 | zunionstore destination numkeys key \[key ...] [weights weight [weight ...]] \[aggregate sum\|min\|max] |

- destination： 交集计算结果保存到这个键。
- numkeys： 需要做交集计算键的个数。
- key[key...]： 需要做交集计算的键。 
- weights weight[weight...]： 每个键的权重， 在做交集计算时， 每个键中的每个 member 会将自己分数乘以这个权重， 每个键的权重默认是 1。
- aggregate sum|min|max： 计算成员交集后， 分值可以按照 sum（和） 、min（最小值） 、 max（最大值） 做汇总， 默认值是 sum。 

### 1.7 键管理

#### 1.7.1 单个键管理

##### 1.键重命名  

**rename key newkey** 

 为了防止被强行 rename， Redis 提供了 renamenx 命令， 确保只有 newKey 不存在时候才被覆盖。

##### 2. 随机返回键 

 **random  key**

##### 3.键过期

- expire key seconds： 键在 seconds 秒后过期。
- expireat key timestamp： 键在秒级时间戳 timestamp 后过期。 
- pexpire key milliseconds： 键在 milliseconds 毫秒后过期。
- pexpireat key milliseconds-timestamp 键在毫秒级时间戳 timestamp 后过期 

注意：

1. 如果 expire key 的键不存在， 返回结果为 0 
2. 如果设置过期时间为负值， 键会立即被删除， 犹如使用 del 命令一样 
3. persist  key  t 命令可以将键的过期时间清除 
4. 对于字符串类型键， 执行 set 命令会去掉过期时间， 这个问题很容易在开发中被忽视 
5. Redis 不支持二级数据结构（例如哈希、 列表） 内部元素的过期功能， 例如不能对列表类型的一个元素做过期时间设置
6. setex 命令作为 set+expire 的组合， 不但是原子执行， 同时减少了一次网络通讯的时间  

#### 1.7.2 键遍历

##### 1. 全量键遍历

**keys pattern** 

##### 2. 渐进式遍历

scan cursor \[match pattern] \[count number] 

- cursor 是必需参数， 实际上 cursor 是一个游标， 第一次遍历从 0 开始， 每次 scan 遍历完都会返回当前游标的值， 直到游标值为 0， 表示遍历结束。
- match pattern 是可选参数， 它的作用的是做模式的匹配， 这点和 keys 的模式匹配很像。
- count number 是可选参数， 它的作用是表明每次要遍历的键个数， 默认值是 10， 此参数可以适当增大。 

#### 1.7.3 数据库管理

##### 1.切换数据库

**select dbIndex**

##### 2.flushdb/flushall 

flushdb/flushall 命令用于清除数据库， 两者的区别的是 flushdb 只清除当前数据库， flushall 会清除所有数据库。 
