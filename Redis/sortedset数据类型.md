sorted set是一个有序的数据集合  
底层使用：skiplist、ziplist、dict实现。  
1、当数据较少时，sorted set是由ziplist来实现的  
2、当数据较多时，由一个dict + 一个skiplist实现，形成zset
### ziplist转zset的条件
#### 1、zset-max-ziplist-entries 128
当ziplist中数据对的数量达到128对时，即数据项为256个时
#### 2、zset-max-ziplist-value 64
当插入任何一个数据长度超过64时