# Redis

**Redis**（**Re**mote **Di**ctionary **S**erver）是一个基于内存的键值型NoSQL数据库。



## Redis常见命令

### 1. Redis基本数据类型

Redis是一个键值数据库，键一般是String类型，值的类型多种多样。值有String，Hash，List，Set和SortedSet几种类型。

### 2. 通用命令

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

### 3. String命令

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

- **SETNX**: 当键不存在时设置键对应的string类型的值

  ```redis
  SETNX key value
  ```

  **Time complexity**: $O(1)$.

- **SETEX**: 设置键对应的string类型的值及过期时间，键不存在时创建键值对

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

- **INCR**: 将键对应的整数值加一，键不存在时使用 0 作为初始值

  ```redis
  INCR key
  ```

  An error is returned if the key contains a value of the wrong type or contains a string that can not be represented as integer. **This operation is limited to 64 bit signed integers**.

  **Time complexity**: $O(1)$

  **Note**:  Redis stores integers in their integer representation, so for string values that actually hold an integer, there is no overhead for storing the string representation of the integer.

- **INCRBY**: 将键对应的整数值增加特定值，键不存在时使用 0 作为初始值

  ```redis
  INCRBY key increment
  ```

  **Time complexity**: $O(1)$

- **INCRBYFLOAT**: 将键对应的浮点值增加特定值，键不存在时则使用 0 作为初始值

  ```redis
  INCRBYFLOAT key increment
  ```

  An error is returned if one of the following conditions occur:

  - The key contains a value of the wrong type (not a string).
  - The current key content or the specified increment are not parsable as a **double precision floating point number**.

  Both the value already contained in the string key and the increment argument can be optionally provided in exponential notation. **The precision of the output is fixed at 17 digits** after the decimal point regardless of the actual internal precision of the computation.

  **Time complexity**: $O(1)$

