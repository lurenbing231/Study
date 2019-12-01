### Effective Java  
   * [一、创建与销毁对象](#一创建与销毁对象)
       * [消除过期的对象引用](#消除过期的对象引用)
       * [避免使用终结方法](#避免使用终结方法)  
   * [二、所有对象的通用方法](#二所有对象的通用方法)
       * [equals方法](#equals方法)
       * [hashcode方法](#hashcode方法)
       * [谨慎覆盖clone](#谨慎覆盖clone)
       * [compareTo方法](#compareto方法)

## 一、创建与销毁对象  

### 消除过期的对象引用  

Java中存在过期饮用的概念，过期引用就是指永远不被解除的引用。例如一下代码（栈实现代码中，pop部分的代码）
```java
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

```java
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

### 避免使用终结方法

终结方法是不可预测的，一般情况下是没有必要的。

终结方法的主要缺点是：不能保证被及时地执行。从一个对象变得不可达开始，到它的终结方法被执行，所花费的时间是任意长的。这意味着注重时间的任务不应该由终结方法来完成。

例如：用终结方法关闭一个打开的文件。

及时的执行终结方法是垃圾回收算法的主要功能之一，但是这些算法在不同的JVM实现中是不同的。如果程序依赖于终结方法被执行的时间点，那么这个程序在不同的JVM中运行的表现可能会截然不同。

同时Java语言规范：1、不能保证终结方法会被执行； 2、不能保证终结方法什么时候被执行； 3、终结方法中出现异常，不会打印出栈轨迹，甚至连警告都不会打印出来。

因此终结方法不能用来释放共享资源上的锁。

System.gc和System.runFinalization这两个方法确实增加了终结方法被执行的机会，但是并不保证终结方法一定会被执行。

System.runFinalizedsOnExit和Runtime.runFinzlizersOnExit这两个方法虽然能保证，但是都存在致命的缺陷，已经被废弃了。

可以使用显式的终止方法来避免使用终结方法。显式的终止方法通常与try-finally结构结合使用，确保及时终止。

因此，除非作为安全网或者为了终止非关键的本地资源，否则不要使用终结方法。

## 二、所有对象的通用方法

### equals方法

注意事项：  

1、使用==操作符检查“参数是否为这个对象的引用”。如果是，则返回true。（这是种优化，当比较操作相对昂贵时可以使用）  

2、使用instanceof操作符检查“参数是否为正确的类型”。  

3、把参数转换成正确的类型。因为进行过instanceof判断操作，所以确保会成功。  

4、对于类中每个“关键”域，检查参数中的域是否和该对象中对应的域相匹配。  

存在一些对象的引用域允许为null，所以为了避免空指针异常，可以使用以下方式进行域比较：

```java
(field == null ? o.field == null : field.equals(o.field))
// 通常field和o.field是相同的对象引用，可以优化成以下形式
(field == o.field || (field != null && field.equals(o.field)))
```

### hashcode方法

hashmap有一项优化：会将每个项相关联的散列码缓存起来，如果散列码不匹配，就不会校验对象的等同性。

编写hashcode的方法：

1、把某个非零的常数值，比如17，保存在一个名为result的int类型的变量中；

2、对于对象中每个关键域f，完成以下步骤：

​    a、为该域计算int类型散列码c：

​        i、如果该域是boolean类型，则计算(f ? 1 : 0)；

​        ii、如果该域是byte、char、short或者int类型，则计算(int)f；

​        iii、如果该域是long类型，则计算(int)(f ^ (f >>> 32))；

​        iv、如果该域是float类型，则计算Float.floatToIntBits(f)；

​        v、如果该域是double类型，则计算Double.doubleToLongBits(f)，然后按照步骤2.a.ii，为得到的long类型值计算散列值；

​        vi、如果该域是个数组，则把每一个元素当作单独的域来处理（递归的调用以上规则）。如果数组域中每个元素都很重要，可以使用Arrays.hashCode方法。

​    b、按照下面的公式，把步骤2.a中计算得到的散列码c合并到result中：

​        result = 31 * result + c;

3、返回result

### 谨慎覆盖clone

协变返回类型：覆盖方法的返回类型可以是被覆盖方法的放回类型的子类。

clone方法就是另一个构造器，必须确保不会影响原始对象，并能确保正确地创建被克隆对象中的约束条件。

例如：对象中包含的域包含了可变的对象，为了确保原始对象和克隆对象相互不受影响，可变对象也要递归地调用clone。

### compareTo方法

注意：(x.compareTo(y) == 0) 与 (x.equals(y))不一定相等，虽然推荐实现这个规则。

例如：HashSet中，添加new BigDecimal("1.0")和new BigDecimal("1.00")，这个集合中就包含两个元素，因为新增的两个实例，通过equals比较是不想等的。 但是使用TreeSet时，集合中只包含一个实例，因为两个实例通过compareTo比较是相等的。

原因：普通集合是通过equals方法来实现等同定义的，但是有序集合是通过compareTo方法实现的。

