### 特点
##### 1、可动态扩展内存。
sds表示的字符串其内容可以修改，也可以追加。在很多语言中字符串会分为mutable和immutable两种，显然sds属于mutable类型的。
##### 2、二进制安全（Binary Safe）。
sds能存储任意二进制数据，而不仅仅是可打印字符。
##### 3、与传统的C语言字符串类型兼容

### 二进制安全原因
header结构：5种
sdshdr5(特殊，没有len、alloc)、sdshdr8、sdshdr16、sdshdr32、sdshdr64
包含：
len：长度  
alloc：最大容量  
flags：一个字节，最低位3个bit表示header类型  
字符数组：最大容量+1。(兼容传统C字符串，最后一位存放NULL结束符)  
### 注意事项
1、初始化的sds字符串数据最后会追加一个NULL结束符  
2、需要的内存空间一次性进行分配  
3、创建空字符串不用sdshdr5  