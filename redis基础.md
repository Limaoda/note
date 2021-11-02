#### 数据类型

* string（字符串）

  string 是 redis **最基本**的类型，一个 key 对应一个 value。

  string 类型是二进制安全的。意思是 redis 的 string 可以包含任何数据。比如jpg图片或者序列化的对象。

  string 类型是 Redis 最基本的数据类型，string 类型的值最大能存储 512MB。

  ```sql
  SET mykey "safsaffa"
  get mykey
  -- safsaffa
  ```
  > **注意：**string类型的值一个键最大能存储 512MB。

* hash（哈希）

  

* list（列表）

* set（集合）

* zset(sorted set：有序集合)



