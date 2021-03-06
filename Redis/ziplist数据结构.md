### 定义
一个经过特殊编码的双向链表，可以存储字符串或整数，其中整数是用真正的二进制表示进行编码的。  
ziplist将表中每一项存放在连续的地址空间。

### 结构
##### zlbytes 32bit
表示ziplist占用字节总数（包括自己）
##### zltail 32bit
表示ziplist表中最后一项（entry）在ziplist中的偏移字节数。方便找到最后一项，进行push和pop操作
##### zllen 16bit
表示ziplist数据项（entry）的个数。最大值2^16-1。  
其中规定：当数据项数量小于最大值时，zllen表示数据项个数；否则，不表示数据项个数，查询个数要遍历整个ziplist。
##### entry
表示真正存放数据的数据项。
##### zlend 8bit
最后一个字节，是一个结束标记，值固定等于255。  

zlbytes、zltail、zllen是小端模式存储

### entry结构
##### prevrawlen
表示前一个数据项占用的总字节数。为了从后向前遍历找到前一项的起始位置。采用变长编码  
###### 变长编码方式：
1个字节或5个字节。  
1、如果前一个数据项占用字节数小于254，就用1个字节表示，这个字节值就是前一个数据项占用字节数。  
2、如果前一个数据项占用字节数大于等于254，就用5个字节表示，其中第一个字节为254（作为这种情况的标记），后面4个字节组成整型值，存储前一个数据项占用字节数。
##### len
表示当前数据项的长度。采用变长编码
###### 变长编码方式：1～3是存储字符串，4～9是存储整数
1、00xxxxxx 1字节，第一个字节最高两位为00，那么len只有1个字节，剩余6个bit表示数据长度，最高为63（2^6-1）。  
2、01xxxxxx|xxxxxxxx 2字节，第一个字节最高两位为01，那么len有2个字节，剩余14个bit表示数据长度，最高为16383（2^14-1）。  
3、10_|xxxxxxxx|xxxxxxxx|xxxxxxxx|xxxxxxxx 5字节。第一个字节最高两位为10，用32个bit表示长度（第一字节剩余6个bit不用），最高表示2^32-1。
4、11000000 1字节，值为0xC0，后面数据为2字节的int16_t类型。  
5、11010000 1字节，值为0xD0，后面数据为4字节的int32_t类型。  
6、11100000 1字节，值为0xE0，后面数据为8字节的int64_t类型。  
7、11110000 1字节，值为0xF0，后面数据为3字节的整数。
8、11111110 1字节，职位0xFE，后面数据为1字节的整数。
9、1111xxxx 1字节（xxxx的值为0001～1101之间，0～12），用这13个值表示真正的数据。这种情况下后面不需要data存储数据。
##### data
真正的数据
### 插入数据
待插入位置原数据为p
1、计算待插入位置前一个数据项长度，作为新数据的prevrawlen。  
2、计算新数据项总字节数，其中尝试把数据转成整数。
3、原数据p的prevrawlen要根据新数据变化。  
4、计算出新的ziplist字节大小，重新调整内存大小，如果当前位置内存不够，进行数据拷贝。  
5、原数据p及之后的数据项向后移动，修改zltail字段。  
6、将新数据放到插入位置，插入结束。
### hash和ziplist联系
当hast数据量比较少时，采用ziplist实现；当数据量增多时，采用dict实现。当变成dict时，存储效率会下降。  
当第一次执行创建hash命令时，底层是ziplist。type=OBJ_HASH，encoding=OBJ_ENCODING_ZIPLIST的robj对象。
##### ziplist转dict的条件
###### 配置
hash-max-ziplist-entries 512  
hash-max-ziplist-value 64  
###### 满足一个
1、hash数据数目（key-value对）超过512时，即ziplist超过1024时。  
2、value长度超过64时
###### 原因
1、ziplist变大时，插入或修改时，造成内存拷贝的概率增加  
2、当数据量增多时，查找指定数据的性能变低