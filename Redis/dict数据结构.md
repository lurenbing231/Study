完成redis哈希的结构  
### 特点
增量式重哈希  
### 结构
##### dictType
通过自定义的方式定义dict中key和value能够存储任意数据类型的数据  
包含若干个函数指针，涉及了key和value的各种操作。例如：哈希计算、深拷贝、key值比较、析构函数等
##### privdata
私有数据指针。由调用者创建dict的时候传进来
##### 两个哈希表 ht[0]、ht[1]
平时只有ht[0]有效，重哈希时ht[1]生效  
包含：  
dictEntry指针数组，当冲突时，形成链表  
size数组长度，2的指数  
used现有数据个数
##### rehashidx
当前重哈希索引。为-1时表示没有重哈希，否则记录重哈希进行到第几步。

### 操作
查找、插入、删除都会触发重哈希  
重哈希：rehashidx指向的bucket上的dictEntry移动到ht[1]上；如果rehashidx指向的没有dictEntry指针数组，就向后遍历直到有dictEntry指针数组时，把其移动到ht[1]上；rehshidx向后推进；当重哈希完成后，把ht[0]变成ht[1]，ht[1]变成ht[0]。
##### 创建
赋予初始值，其中ht[0]、ht[1]指向null,rehashidx=-1
##### 查找dictFind
1、判断是否时重哈希，是rehashidx前进一步，进行重哈希  
2、计算key的哈希值  
3、在ht[0]上查找，如果没找到且在重哈希过程中，就再ht[1]上查找
##### 插入
dictAdd：存在则插入失败
dictReplace：存在则更新
如果在重哈希，则插入到ht[1]中，否插入到ht[0]中  
插入到dictEntry头部  
dictReplace会调用dictFind和dictAdd
##### 删除dictDelete
如果不在重哈希中，则从ht[0]中查找删除，否则从ht[0]和ht[1]中查找删除  
会调用key和value的析构函数