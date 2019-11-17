### Effective Java
   * [一、创建与销毁对象](#一创建与销毁对象)
       * [消除过期的对象引用](#消除过期的对象引用)

## 一、创建与销毁对象

### 消除过期的对象引用

Java中存在过期饮用的概念，过期引用就是指永远不被解除的引用。例如一下代码（栈实现代码中，pop部分的代码）
```
public class Stack {
   private Object[] elements;
   private int size = 0;
   
   public Stack() {...}
   public void push(Object e) {...}
   
   public Object pop() {
       if (size == 0) {
           throw new EnmptyStackException();
       }
       return elements[--size];
   }
   private void ensureCapacity() {
       if (elements.length == size) {
           elements = Arrays.copyOf(elements, 2 * size + 1);
       }
   }
}
```
这段代码可以执行，可以实现stack。   

但是这段代码隐藏着内存泄露的问题：如果一个栈先增长，再收缩，那么从栈中弹出的对象将不会被当作垃圾回收掉，即使不再引用这些对象，因为栈的内部维护着对这些对象的过期引用。   

java的垃圾回收机制：只要对象存在引用就不会被回收。这样会导致java存在许多对象不被垃圾回收。

解决上面例子的问题方式为：

```
   public Object pop() {
       if (size == 0) {
           throw new EnmptyStackException();
       }
       Object result = elements[--size];
       elements[size] = null;      // 将对象引用置为空
       return result;
   }
```

《Effective Java》的建议是：只要是类自己管理内存，程序员就应该警惕内存泄露的问题。

##### 内存泄露第二个常见来源是缓存。

可以使用WeakHashMap、LinkedHashMap的removeEldestEntry方法简单实现。如果缓存较为复杂，就必须使用java.lang.ref。

##### 内存泄露的第三个常见来源是监听器和其他回调。

例如一个API中，客户端在这个API中注册了回调，却没有取消注册，这些对象就会堆积。

最佳解决方法：只保存它们的弱引用，例如：只将它们保存成WeakHashMap中的key。

