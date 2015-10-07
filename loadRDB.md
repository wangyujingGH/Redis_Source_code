## 读取RDB文件

`(while循环）` 
### 1、读取“REDIS0006”这9个字节，版本号匹配
PS：读取函数`rioRead`，其中有一个变量控制着目前读取到的位置。

### 2、读取EOF类型
EOF标志着数据读取结束，break。

### 4、读取类型指示，决定了之后如何读入跟着的数据
rdg.h中定义了很多类型，一个字节存放，例如REDIS_RDB_OPCODE_EOF为255，REDIS_RDB_OPCODE_SELECTED为254，类型后面跟的就是相应的数据。

### 5、读入过期时间类型，读入过期时间值；过期时间之后跟的是对应的数据键值对。
### 6、读入对应的数据键值对。
### 7、添加次键值对到数据字典中，设置过期时间。
### 8、最后读取8个字节的校验和进行校验。

<br/>
********************

# RDB文件中数据存放的格式：
### 1、整体结构：
    REDIS    version(0006)	database 0	database 3	EOF	check_sum
### 2、Database 0 中数据格式：
    SELECTDB	db_number	key_value_pairs
### 3、Key_value_pairs中数据格式：
* 键值对不带过期时间
        TYPE	key	value
* 键值对带过期时间
        EXPIRETIMR_MS	ms	TYPE	key	value

> PS：Key对象的存放类型是string，这里的TYPE指的是value的数据类型，（这里指的是在RDB文件中的类型）
  这里总共有14种（`REDIS_RDB_TYPE_*，STRING，LIST，SET，ZSET，HASH，LIST_ZIPLIST，SET_INTSET，ZSET_ZIPLIST，
  HASH_ZIPLIDT，HASH_ZIPMAP，LIST_ZIPLIST，SET_INTSET，ZSET_ZIPLIST，HASH_ZIPLIST`）。

### 字符串对象存储格式：
    字节数	value
### 这里value的数据存储格式：
    节点个数	字节数	value1	字节数	value2	...
    
> 对于value，根据TYPE类型以及字节数来决定value的编码格式，为value创建相应的redisObject（类型，编码）。

### 对象编码
    #define REDIS_ENCODING_RAW 0     /* Raw representation */
    #define REDIS_ENCODING_INT 1     /* Encoded as integer */
    #define REDIS_ENCODING_HT 2      /* Encoded as hash table */
    #define REDIS_ENCODING_ZIPMAP 3  /* Encoded as zipmap */
    #define REDIS_ENCODING_LINKEDLIST 4 /* Encoded as regular linked list */
    #define REDIS_ENCODING_ZIPLIST 5 /* Encoded as ziplist */
    #define REDIS_ENCODING_INTSET 6  /* Encoded as intset */
    #define REDIS_ENCODING_SKIPLIST 7  /* Encoded as skiplist */
    #define REDIS_ENCODING_EMBSTR 8  /* Embedded sds string encoding */

