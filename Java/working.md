# 工作中的问题总结
   * [一、ENUM枚举](#一ENUM枚举)
     

## 一、ENUM枚举
在ENUM的使用过程中，要注意其是常量，new一个对象时会永久保留，其他方法再次调用时，会使用第一次new的对象。  
例如：  
1、在ENUM中使用new Date()获取当前时间，只会获得第一次运行的时间，其他时间再调用，也是第一次的时间，使new Date()达不到应有的效果