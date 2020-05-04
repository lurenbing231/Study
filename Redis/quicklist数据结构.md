### 定义
是由ziplist组成的双向链表。  
例如：quicklist中有4个ziplist，每个ziplist有三个数据项，则quicklist对外暴露12个数据项。
### 意义
对双向链表和ziplist的折中。
### 相关参数
#### list-max-ziplist-size 
可以为正数或负数。  
正数：表示quicklist中每个ziplist包含数据项的个数。  
负数：-1～-5  
-5:每个ziplist大小不能超过64kb  
-4:每个ziplist大小不能超过32kb  
-3:每个ziplist大小不能超过16kb  
-2:每个ziplist大小不能超过8kb（redis默认值）  
-1:每个ziplist大小不能超过4kb
#### list-compress-depth
quicklist两端不能被压缩的节点个数  
0：特殊值，不进行压缩，redis默认值  
1：两端各有一个节点不被压缩  
2：两端各有两个节点不被压缩
以此类推  
压缩算法：<font color="#dd0000">LZF</font>
### quicklistNode结构
prev：是指向前一个节点的指针  
next：是指向后一个节点的指针  
zl：数据指针。如果数据没有压缩，指向ziplist结构；否则，指向quicklistLZF结构  
sz：表示zl指向的ziplist的总大小（如果压缩，是压缩前的大小）  
count：表示ziplist里面包含的数据项个数。16bit  
encoding：表示ziplist是否被压缩了。1表示没压缩，2表示压缩  
container：预留字段。用来表示quicklist节点下面是直接存储数据还是用ziplist。默认等于2。  
recompress：等于1时，表示当前数据被解压了，之后要进行重新压缩。  
### quicklistLZF表示被压缩的ziplist
sz：压缩后的大小  
compressed：柔性数组，存放压缩后的字节数组

### quicklist真正结构是struct quicklist
head：指向头节点的指针  
tail：指向尾节点的指针  
count：所有ziplist的个数总和  
len：节点个数  
fill：16bit，ziplist大小参数，存放list-max-ziplist-size的值  
compress：16bit，节点压缩深度，存放list-compress-depth的值

### quicklist创建
特点：head和tail都为null
