### skiplist、平衡树、哈希表对比
1、skiplist和平衡树的元素是有序的，哈希表无序。  
2、范围查找方面，skiplist简单，平衡树需要中序遍历，哈希表要遍历所有节点  
3、平衡树插入删除可能触发子树调整，skiplist和哈希表较简单  
4、在查找单个key，skiplist和平衡树时间复杂度为O(log n)；哈希表在保持较低的哈希值冲突概率的前提下，复杂度接近O(1)。  
5、算法实现难度上，skiplist比平衡树简单。  
### skiplist对外暴露的数据结构为：sorted set。
sorted set是一个有序的数据集合  
会对key进行排序，key相同对value排序  
1、当数据较少时，sorted set是由ziplist来实现的  
2、当数据较多时，由一个dict + 一个skiplist实现
### redis中skiplist特殊性
1、redis的skiplist的key允许重复  
2、redis的skiplist通过数据内容本身做唯一标识，而不是用key  
3、第一层链表是双向链表  
4、可以很方便地计算出每个元素的排名
### 结构
##### ZSKIPLIST_MAXLEVEL
对应跳表的层数MaxLevel。节点最大层数  
##### ZSKIPLIST_P
对应跳表算法的P。如果节点有第i层，那该节点有i+1层的概率为p
##### zskiplistNode节点
###### obj
存放节点数据，类型为string robj。  
在插入数据时，对数据进行解码，因此obj字段中存的一定是sds。  
原因：方便查找时对数据进行字典序比较。
###### score 数据对应的分数
###### backward
节点指向前一个链表的指针。  
节点只有一个该指针，在第一层链表中。
###### level[] 
柔性数组，当插入节点时才单独分配内存。  
存放各层链表向后的指针。  
每层有forward、span组成。forward表示指针、span表示当前指针跨越多少个节点（不包括起点节点，包含终点节点）。  
span用于计算排名。
##### zskiplist真正的skiplist结构
###### header头指针、tail尾指针
###### length链表长度，即节点总数。
###### level表示总层数，即所有节点最大层数
