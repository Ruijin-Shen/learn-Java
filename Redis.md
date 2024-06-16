# Redis

**Redis**（**Re**mote **Di**ctionary **S**erver）是一个基于内存的键值型NoSQL数据库。



## 1. Redis常见命令

### 1.1 Redis基本数据类型

Redis是一个键值数据库，键一般是String类型，值的类型多种多样。值有String，Hash，List，Set和SortedSet几种类型。

### 1.2 通用命令

* **KEYS**：返回符合模板的所有key

  ```redis
  KEYS pattern
  ```

  Supported glob-style patterns:

  - `h?llo` matches `hello`, `hallo` and `hxllo`
  - `h*llo` matches `hllo` and `heeeello`
  - `h[ae]llo` matches `hello` and `hallo,` but not `hillo`
  - `h[^e]llo` matches `hallo`, `hbllo`, ... but not `hello`
  - `h[a-b]llo` matches `hallo` and `hbllo`

  Use `\` to escape special characters if you want to match them verbatim.

  **Time complexity**: $O(N)$ with $N$ being the number of keys in the database.

  **Warning**: consider `KEYS` as a command that should only be used in production environments with extreme care.

- **DEL**：删除一个或多个键，不存在的键将被忽略

  ```redis
  DEL key [key ...]
  ```

  **Time complexity**: $O(N)$ where $N$ is the number of keys that will be removed.

- **EXISTS**：判断一个或多个键是否存在，返回存在的键的个数

  ```redis
  EXISTS key [key ...]
  ```

  **Time complexity**: $O(N)$ where $N$ is the number of keys to check.

- **EXPIRE**：设置键的过期时间，以秒为单位

  ```redis
  EXPIRE key seconds [NX | XX | GT | LT]
  ```

  After the timeout has expired, the key will automatically be deleted. A key with an associated timeout is often said to be **volatile** in Redis terminology. The timeout will only be cleared by commands that delete or overwrite the contents of the key, including `DEL`, `SET`, and `GETSET` commands. This means that all the operations that conceptually alter the value stored at the key without replacing it with a new one will leave the timeout untouched. For instance, incrementing the value of a key with `INCR`, pushing a new value into a list with `LPUSH`, or altering the field value of a hash with `HSET` are all operations that will leave the timeout untouched.

  **The timeout can also be cleared, turning the key back into a persistent key, using the PERSIST command**.

  The `EXPIRE` command supports a set of options:

  - `NX` -- Set expiry only when the key has no expiry
  - `XX` -- Set expiry only when the key has an existing expiry
  - `GT` -- Set expiry only when the new expiry is greater than current one
  - `LT` -- Set expiry only when the new expiry is less than current one

  **Time complexity**: $O(1)$.

- **TTL**：返回键的剩余生存时间

  ```redis
  TTL key
  ```

  The return value in case of error:

  - The command returns `-2` if the key does not exist.
  - The command returns `-1` if the key exists but has no associated expire.
  
  **Time complexity**: $O(1)$.

### 1.3 String类型

String是字符串类型，是Redis中最简单的存储类型。根据字符串的格式不同，可以分为3类：string普通字符串，int整数类型，float浮点类型。

- **SET**: 设置或重写一个string类型的键值对

  ```redis
  SET key value [NX | XX] [GET] [EX seconds | PX milliseconds | EXAT unix-time-seconds | PXAT unix-time-milliseconds | KEEPTTL]
  ```

  **If `key` already holds a value, it is overwritten**, regardless of its type. Any previous time to live associated with the key is discarded on successful `SET` operation.

  The `SET` command supports a set of options that modify its behavior:

  - `NX` -- Only set the key if it does not already exist.
  - `XX` -- Only set the key if it already exists.

  **Time complexity**: $O(1)$.

- **SETNX** *deprecated*: 当键不存在时设置键对应的string类型的值

  ```redis
  SETNX key value
  ```

  **Time complexity**: $O(1)$.

- **SETEX** *deprecated*: 设置键对应的string类型的值及过期时间，键不存在时创建键值对

  ```redis
  SETEX key seconds value
  ```

  **Time complexity**: $O(1)$.

- **GET**: 获取键对应的值

  ```redis
  GET key
  ```

  Get the value of `key`. If the key does not exist the special value `nil` is returned. An error is returned if the value stored at `key` is not a string, because `GET` only handles string values.

  **Time complexity**: $O(1)$.

- **MSET**: 原子性地创建或修改一个或多个string类型的键值对

  ```redis
  MSET key value [key value ...]
  ```

  `MSET` replaces existing values with new values, just as regular `SET`. `MSET` is **atomic**, so all given keys are set at once.

  **Time complexity**: $O(N)$ where $N$ is the number of keys to set.

- **MGET**: 原子性地返回一个或多个键对应的string类型的值

  ```redis
  MGET key [key ...]
  ```

  Returns the values of all specified keys. For every key that does not hold a string value or does not exist, the special value `nil` is returned. Because of this, the operation never fails.

  **Time complexity**: $O(N)$ where $N$ is the number of keys to retrieve.

- **GETSET**: 原子性地设置键的值对并返回旧值

  ```redis
  GETSET key value
  ```

  Returns an error when `key` exists but does not hold a string value. Any previous time to live associated with the key is discarded on successful `SET` operation.

  **Time complexity**: $O(1)$.

- **INCR**: 将键对应的整数值加一，键不存在时使用 0 作为初始值

  ```redis
  INCR key
  ```

  An error is returned if the key contains a value of the wrong type or contains a string that can not be represented as integer. **This operation is limited to 64 bit signed integers**.

  **Time complexity**: $O(1)$.

  **Note**:  Redis stores integers in their integer representation, so for string values that actually hold an integer, there is no overhead for storing the string representation of the integer.

- **INCRBY**: 将键对应的整数值增加指定值，键不存在时使用 0 作为初始值

  ```redis
  INCRBY key increment
  ```

  **Time complexity**: $O(1)$.

- **INCRBYFLOAT**: 将键对应的浮点值增加特定值，键不存在时则使用 0 作为初始值

  ```redis
  INCRBYFLOAT key increment
  ```

  An error is returned if one of the following conditions occur:

  - The key contains a value of the wrong type (not a string).
  - The current key content or the specified increment are not parsable as a **double precision floating point number**.

  Both the value already contained in the string key and the increment argument can be optionally provided in exponential notation. **The precision of the output is fixed at 17 digits** after the decimal point regardless of the actual internal precision of the computation.

  **Time complexity**: $O(1)$.

### 1.4 Hash类型

在存储对象时，string类型是将对象序列化成JSON字符串后存储，因此对某个字段的修改很不方便。Hash类型可以将对象中的每个字段独立存储，方便对单个字段增删改查。

- **HSET**: 创建或修改hash中字段的值

  ```redis
  HSET key field value [field value ...]
  ```

  This command overwrites the values of specified fields that exist in the hash. If key doesn't exist, a new key holding a hash is created.

  **Time complexity**: $O(1)$ for each field/value pair added.

- **HGET**: 返回hash中字段的值

  ```redis
  HGET key field
  ```

  **Time complexity**: $O(1)$.

- **HMSET** *deprecated*: 设置多个字段的值

  ```redis
  HMSET key field value [field value ...]
  ```

  **Time complexity**: $O(N)$ where $N$ is the number of fields being set.

- **HMGET**: 返回hash中多个字段的值

  ```redis
  HMGET key field [field ...]
  ```

  For every `field` that does not exist in the hash, a `nil` value is returned. Because non-existing keys are treated as empty hashes, running `HMGET` against a non-existing `key` will return a list of `nil` values.

  **Time complexity**: $O(N)$ where $N$ is the number of fields being requested.

- **HGETALL**: 返回键对应hash的所有字段和值

  ```redis
  HGETALL key
  ```

  In the returned value, every field name is followed by its value, so the length of the reply is twice the size of the hash.

  **Time complexity**: $O(N)$ where $N$ is the size of the hash.

- **HKEYS**: 返回键对应hash的所有字段名

  ```redis
  HKEYS key
  ```

  **Time complexity**: $O(N)$ where $N$ is the size of the hash.

- **HINCRBY**：将hash中字段的整数值增加指定值

  ```redis
  HINCRBY key field increment
  ```

  If `key` does not exist, a new key holding a hash is created. If `field` does not exist the value is set to `0` before the operation is performed. The range of values supported by `HINCRBY` is limited to 64 bit signed integers.

  **Time complexity**: $O(1)$.

- **HSETNX**: 仅当字段不存在时，设置字段的值

  ```redis
  HSETNX key field value
  ```

  Sets `field` in the hash stored at `key` to `value`, only if `field` does not yet exist. If `key` does not exist, a new key holding a hash is created. If `field` already exists, this operation has no effect.

  **Time complexity**: $O(1)$.

### 1.5 List类型

Redis的列表是存储字符串的链表，可用于实现栈和队列。

- **LPUSH**:  从左侧将指定的元素插入到键对应的列表

  ```redis
  LPUSH key element [element ...]
  ```

  If `key` does not exist, it is created as empty list before performing the push operations. When `key` holds a value that is not a list, an error is returned.

  It is possible to push multiple elements using a single command call just specifying multiple arguments at the end of the command. Elements are inserted one after the other to the head of the list, from the leftmost element to the rightmost element.

  **Time complexity**: $O(N)$ where $N$ is the number of elements to push.

- **LPOP**：从左侧删除并返回键对应列表的元素

  ```redis
  LPOP key [count]
  ```

  By default, the command pops a single element from the beginning of the list. When provided with the optional `count` argument, the reply will consist of up to `count` elements, depending on the list's length. **Deletes the list if the last element was poped**.

  **Time complexity**: $O(N)$ where $N$ is the number of elements to pop.

- **RPUSH**：从右侧将指定的元素插入到键对应的列表

  ```redis
  RPUSH key element [element ...]
  ```

  When specifying multiple arguments at the end of the command, elements are inserted one after the other to the tail of the list, from the leftmost element to the rightmost element. See `LPUSH` for more details.

  **Time complexity**: $O(N)$ where $N$ is the number of elements to push.

- **RPOP**: 从右侧删除并返回键对应列表的元素

  ```redis
  RPOP key [count]
  ```

  See `LPOP` for more details.

  **Time complexity**: $O(N)$ where $N$ is the number of elements to pop.

- **LRANGE**: 返回列表中指定范围的元素

  ```redis
  LRANGE key start stop
  ```

  The offsets `start` (included) and `stop` (**included**) are zero-based indexes, with `0` being the first element of the list (the head of the list), `1` being the next element and so on.

  These offsets can also be negative numbers indicating offsets starting at the end of the list. For example, `-1` is the last element of the list, `-2` the penultimate, and so on. Out of range indexes will not produce an error. If `start` is larger than the end of the list, an empty list is returned. If `stop` is larger than the actual end of the list, Redis will treat it like the last element of the list.

  **Time complexity**: $O(S + N)$ where $S$ is the distance of start offset from HEAD for small lists, from nearest end (HEAD or TAIL) for large lists; and N is the number of elements in the specified range.

- **BLPOP**: 阻塞版`LPOP`

  ```redis
  BLPOP key [key ...] timeout
  ```

  It is the blocking version of `LPOP` because it blocks the connection when there are no elements to pop from any of the given lists.

  When `BLPOP` is called, if at least one of the specified keys contains a non-empty list, an element is popped from the head of the list and returned to the caller together with the `key` it was popped from.  An element is popped from the head of the first list that is non-empty, with the given keys being checked in the order that they are given.

  If none of the specified keys exist, `BLPOP` blocks the connection until another client performs an `LPUSH` or `RPUSH` operation against one of the keys. Once new data is present on one of the lists, the client returns with the name of the key unblocking it and the popped value.

  When `BLPOP` causes a client to block and a non-zero timeout is specified, the client will unblock returning a `nil` multi-bulk value when the specified timeout has expired without a push operation against at least one of the specified keys.

  **The timeout argument is interpreted as a double value specifying the maximum number of seconds to block. A timeout of zero can be used to block indefinitely.**

  **Time complexity**: $O(N)$ where $N$ is the number of provided keys.

- **BRPOP**: 阻塞版`RPOP`

  ```redis
  BRPOP key [key ...] timeout
  ```

  See `BLPOP` for more details.

  **Time complexity**: $O(N)$ where $N$ is the number of provided keys.

### 1.6 Set类型

Redis的集是唯一字符串的无序集合。

- **SADD**: 将指定的成员添加到键对应的集中

  ```redis
  SADD key member [member ...]
  ```

  Specified members that are already a member of this set are ignored. If `key` does not exist, a new set is created before adding the specified members. An error is returned when the value stored at `key` is not a set.

  **Time complexity**: $O(N)$ where $N$ is the number of members to be added.

- **SREM**: 从键对应的集中删除指定的成员

  ```redis
  SREM key member [member ...]
  ```

  **Specified members that are not a member of this set are ignored**. If `key` does not exist, it is treated as an empty set and this command returns `0`. An error is returned when the value stored at `key` is not a set.

  **Time complexity**: $O(N)$ where $N$ is the number of members to be removed.

- **SCARD**: 返回集的元素个数

  ```redis
  SCARD key
  ```

  **Time complexity**: $O(1)$.

- **SISMEMBER**: 判断成员是否属于集

  ```redis
  SISMEMBER key member
  ```

  Returns `1` if `member` is a member of the set stored at `key`, otherwise `0`.

  **Time complexity**: $O(1)$.

- **SMEMBERS**: 返回集的所有成员

  ```redis
  SMEMBERS key
  ```

  **Time complexity**: $O(N)$ where $N$ is the set cardinality.

- **SINTER**: 返回多个集的交集

  ```redis
  SINTER key [key ...]
  ```

  Returns the members of the set resulting from the intersection of all the given sets.

  **Time complexity**: $O(N*M)$ worst case where $N$ is the cardinality of the smallest set and $M$ is the number of sets.

- **SUNION**: 返回多个集的交集

  ```redis
  SUNION key [key ...]
  ```

  Returns the members of the set resulting from the union of all the given sets.

  **Time complexity**: $O(N)$ where $N$ is the total number of elements in all given sets.

- **SDIFF**: 返回多个集的差集

  ```redis
  SDIFF key [key ...]
  ```

  Returns the members of the set resulting from the **difference between the first set and all the successive sets**.

  **Time complexity**: $O(N)$ where $N$ is the total number of elements in all given sets.

### 1.7 SortedSet类型

TODO: command for SortedSet

- ZADD
- ZREM
- ZSCORE
- ZRANK
- ZCOUNT
- ZINCRBY
- ZRANGE
- ZRANGEBYSCORE
- ZDIFF
- ZINTER
- ZUNION



## 2. Jedis

### 2.1 Jedis快速开始程序

引入Jedis依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>5.0.0</version>
</dependency>
```

引入单元测试JUnit依赖

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.10.2</version>
    <scope>test</scope>
</dependency>
```

创建Jedis客户端对象

```Java
private static Jedis jedis;

@BeforeAll
static void setUp() {
    String host = "192.168.1.158";
    int port = 6379;
    // 建立连接
    jedis = new Jedis(host, port);
    // 登录认证
    jedis.auth("password");
    // 选择库
    jedis.select(0);
}
```

数据库操作

```Java
@Test
void testHash() {
    jedis.hset("user:1", "name", "Alice");
    jedis.hset("user:1", "age", "20");

    Map<String, String> result = jedis.hgetAll("user:1");
    System.out.println(result);
}
```

关闭与Redis服务器的连接

```Java
@AfterAll
static void tearDown() {
    if (jedis != null) jedis.close();
}
```

## 3. 分布式锁

### 3.1 使用`SETNX`和`GETSET`实现分布式锁

- **获取锁**

  `SETNX` can be used, and was historically used, as a locking primitive. For example, to acquire the lock of the key `foo`, the client could try the following:

  ```redis
  SETNX lock.foo <current Unix time + lock timeout + 1>
  ```

  If `SETNX` returns `1` the client acquired the lock, setting the `lock.foo` key to the Unix time at which the lock should no longer be considered valid. The client will later use `DEL lock.foo` in order to release the lock.

  If `SETNX` returns `0` the key is already locked by some other client. We can either return to the caller if it's a non blocking lock, or enter a loop retrying to hold the lock until we succeed or some kind of timeout expires.

- **释放超时锁**

  If the timestamp is less than or equal to the current Unix time the lock is no longer valid.

  - Client C1 crashed after holding the lock.

  - C2 sends `SETNX lock.foo` in order to acquire the lock.

  - The crashed client C1 still holds it, so Redis will reply with `0` to C2.

  - C2 sends `GET lock.foo` to check if the lock expired. If it is not, it will sleep for some time and retry from the start.

  - Instead, if the lock is expired because the Unix time at `lock.foo` is older than the current Unix time, C2 tries to perform：

    ```redis
    GETSET lock.foo <current Unix timestamp + lock timeout + 1>
    ```

  - Because of the `GETSET` semantic, C2 can check if the old value stored at key is still an expired timestamp. If it is, the lock was acquired.
  
  - If another client, for instance C3, was faster than C2 and acquired the lock with the `GETSET` operation, the C2 `GETSET` operation will return a non-expired timestamp. C2 will simply restart from the first step. Note that even if C2 set the key a bit a few seconds in the future this is not a problem.
  
- **释放锁**

  Instead of releasing the lock with `DEL`, send a script that only removes the key if the the timeout was not modified. This avoids that a client will try to release the lock after the expire time deleting the key created by another client that acquired the lock later. See next section for a more robust and flexible implementation.


### 3.2 使用`SET`和Lua脚本实现分布式锁



### 3.3 使用Redlock算法实现分布式锁

The above patterns are discouraged in favor of **the Redlock algorithm** which is only a bit more complex to implement, but offers better guarantees and is fault tolerant.

