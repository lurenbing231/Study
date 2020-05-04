全名：redisObject通用数据结构  
redis内部是key、value格式。其中key一般用string类型， value多种类型，value使用robj表示  
### 结构
type：对象的数据类型。OBJ_STRING, OBJ_LIST, OBJ_SET, OBJ_ZSET, OBJ_HASH 分别对应redis五种数据类型。
encoding：对象内部表示方式（编码方式）。OBJ_ENCODING_RAW、OBJ_ENCODING_INT、OBJ_ENCODING_HT、OBJ_ENCODING_ZIPMAP、OBJ_ENCODING_LINKEDLIST、OBJ_ENCODING_ZIPLIST、OBJ_ENCODING_INTSET、OBJ_ENCODING_SKIPLIST、OBJ_ENCODING_EMBSTR、OBJ_ENCODING_QUICKLIST对应10种编码方式。  
lru：LRU替换算法。  
refcount：引用计数。robj共享时使用。  
ptr：数据指针。指向真正的数据。  

##### 特别注意  
同一个type可能对应不同的encoding。  
例如：type=OBJ_STRING时，encoding为OBJ_ENCODING_RAW（string原生表示方式，用sds结构）、OBJ_ENCODING_INT（数字表示方式，long类型）、OBJ_ENCODING_EMBSTR（嵌入式sds表示方式）；  
当type = OBJ_HASH时，encoding为OBJ_ENCODING_HT（dict表示）、OBJ_ENCODING_ZIPLIST（ziplist表示）。  

##### 十种编码方式
OBJ_ENCODING_RAW: 最原生的表示方式。其实只有string类型才会用这个encoding值（表示成sds）。  
OBJ_ENCODING_INT: 表示成数字。实际用long表示。  
OBJ_ENCODING_HT: 表示成dict。  
OBJ_ENCODING_ZIPMAP: 是个旧的表示方式，已不再用。
OBJ_ENCODING_LINKEDLIST: 是个旧的表示方式，已不再用。  
OBJ_ENCODING_ZIPLIST: 表示成ziplist。  
OBJ_ENCODING_INTSET: 表示成intset。用于set数据结构。  
OBJ_ENCODING_SKIPLIST: 表示成skiplist。用于sorted set数据结构。  
OBJ_ENCODING_EMBSTR: 表示成一种特殊的嵌入式的sds。  
OBJ_ENCODING_QUICKLIST: 表示成quicklist。用于list数据结构。  

### 作用
1、为多种数据类型提供统一的表示方式  
2、同一类型采用不同的编码方式，在某些情况下节省内存  
3、支持对象共享。当对被共享时，只占一份内存拷贝，进一步节省内存  

### 编码过程
1、set命令时，先接收value值，type=OBJ_STRING、encoding=OBJ_ENCODING_RAW的robj对象，然后执行编码过程转成其他结构