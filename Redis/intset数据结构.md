为了实现集合set（无序、无重复）  
intset是一个由整数组成的有序集合。  
是连续的一整块内存空间，对大整数和小整数采用不同的编码。  
### 结构
##### encoding 数据编码
表示intset中每个数据元素用几个字节存储。三个可能取值：  
INTSET_ENC_INT16：表示每个数据用2个字节存储（刚插入数据时使用）  
INTSET_ENC_INT32：表示每个数据用4个字节存储  
INTSET_ENC_INT64：表示每个数据用8个字节存储
##### length表示intset元素个数
##### contents 柔性数组
存储数据，数据总字节数 = encoding * length  
单独分配内存空间

### ziplist和intset的区别
1、ziplist可以存任意二进制串，intset只能存整数  
2、ziplist无序，intset从小到大排序  
3、ziplist可以对每个数据项进行不同的编码。intset只能整体使用统一编码
### 