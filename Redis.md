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

- **EXPIRE**：设置键的过期时间，以秒为单位

  ```redis
  EXPIRE key seconds [NX | XX | GT | LT]
  ```

  After the timeout has expired, the key will automatically be deleted. A key with an associated timeout is often said to be **volatile** in Redis terminology. The timeout will only be cleared by commands that delete or overwrite the contents of the key, including `DEL`, `SET`, and `GETSET` commands. This means that all the operations that conceptually alter the value stored at the key without replacing it with a new one will leave the timeout untouched. For instance, incrementing the value of a key with `INCR`, pushing a new value into a list with `LPUSH`, or altering the field value of a hash with `HSET` are all operations that will leave the timeout untouched.

  **The timeout can also be cleared, turning the key back into a persistent key, using the PERSIST command**.

- **TTL**：返回键的剩余生存时间

  ```redis
  TTL key
  ```

  The return value in case of error:

  - The command returns `-2` if the key does not exist.
  - The command returns `-1` if the key exists but has no associated expire.

### 3. String命令

String是字符串类型，是Redis中最简单的存储类型。根据字符串的格式不同，可以分为3类：string普通字符串，int整数类型，float浮点类型。
