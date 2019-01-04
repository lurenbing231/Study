# 个人学习使用
| [编程思想](#编程思想) | [Effective Java](#Effective-Java)|














### 编程思想
   * [Java初始化](#Java初始化)
   * [Java类的复用](#Java类的复用)
   * [Java向上转型及final关键字](#Java向上转型及final关键字)
   * [抽象类和接口](#抽象类和接口)
   * [多态](#多态)
   * [内部类](#内部类)
   * [容器](#容器)

### Effective Java


## Java初始化

Java通过构造器确保每个对象都被初始化，构造器采用与类相同的名称。
```class Car {
   Car() {//无参构造器（默认构造器）
       System.out.println("Car");
   }

   Car(String name) {
       System.out.println("CarName:" + name);
   }
}
```
对于无参构造器在创建对象时：
```Car car = new Car();```
对于有形式参数的构造器，创建对象时：
```Car car = new Car("BMW");```
当Car(String)是Car类中唯一的构造器时，编译器将不会允许以其他任何方式创建Car对象-----Java编程思想（第四版）
如果类中没有构造器，编译器会自动创建一个默认构造区。但是如果定义了一个构造器（无论是否有参数），编译器都不会自动创建默认构造器。
构造器是没有返回值的，new表达式返回了对新建对象的引用，但构造器本身是没有任何返回值。
在Java中，可以用多种方式创建一个类，就像上面的代码一样，这里用到了方法重载，其规则是：每一个重载的方法必须都有一个独一无二的参数类型列表（其中无参和参数类型顺序也算）
对于基本类型能从一个“小”的类型自动提升至“大”的类型，在重载中也成立，下面是基本类型在重载中的情况。
```public class Overload {//《Java编程思想》的例子
    void z1(char x) { System.out.print("z1(char) "); }
    void z1(byte x) { System.out.print("z1(byte) "); }
    void z1(short x) { System.out.print("z1(short) "); }
    void z1(int x) { System.out.print("z1(int) "); }
    void z1(long x) { System.out.print("z1(long) "); }
    void z1(float x) { System.out.print("z1(float) "); }
    void z1(double x) { System.out.print("z1(double) "); }

    void z2(byte x) { System.out.print("z2(byte) "); }
    void z2(short x) { System.out.print("z2(short) "); }
    void z2(int x) { System.out.print("z2(int) "); }
    void z2(long x) { System.out.print("z2(long) "); }
    void z2(float x) { System.out.print("z2(float) "); }
    void z2(double x) { System.out.print("z2(double) "); }

    void z3(short x) { System.out.print("z3(short) "); }
    void z3(int x) { System.out.print("z3(int) "); }
    void z3(long x) { System.out.print("z3(long) "); }
    void z3(float x) { System.out.print("z3(float) "); }
    void z3(double x) { System.out.print("z3(double) "); }

    void z4(int x) { System.out.print("z4(int) "); }
    void z4(long x) { System.out.print("z4(long) "); }
    void z4(float x) { System.out.print("z4(float) "); }
    void z4(double x) { System.out.print("z4(double) "); }

    void z5(long x) { System.out.print("z5(long) "); }
    void z5(float x) { System.out.print("z5(float) "); }
    void z5(double x) { System.out.print("z5(double) "); }

    void z6(float x) { System.out.print("z6(float) "); }
    void z6(double x) { System.out.print("z6(double) "); }

    void z7(double x) { System.out.print("z7(double) "); }

    @Test
    public void testOverload() {
        System.out.println("int:" );
        z1(5);
        z2(5);
        z3(5);
        z4(5);
        z5(5);
        z6(5);
        z7(5);
    }
}
```
其结果为：
 
可以看出重载方法接受int型参数时，如果不存在int型形式参数类型时，实际数据类型会被提升。因此，各个类型的提升顺序为：
char类型：char -> int -> long -> float -> double
byte类型：byte -> short -> int -> long -> float -> double
其他基本类型在提升时根据上面的顺序一样。
this关键字
this关键字只能在方法内部使用，即在方法内部得到当前对象的引用。
```
class Car {
    Car getCar() {
        return this;
    }
}
```
返回Car对象的引用。
当一个类中存在多个构造器，并且想一个构造器调用另一个构造器时，this关键字可以发挥作用。在构造器中，如果this方法添加了参数列表，则是对符合此参数列表的某个构造器的明确调用。
```
class Car {
   Car() {
        this("BMW");//调用构造器Car(String name)
        System.out.println("Car");
   }
   Car(String name) {
        System.out.println("CarName:" + name);
   }
}
```
虽然可以调用其他构造器，但是不能同时调用两个，且this调用构造器必须在最起始的位置。
初始化
在Java中，编译器会尽可能保证所有变量在使用前都能得到恰当的初始化。
对于方法的局部变量来说，为保证初始化，对于使用时未初始化的变量，编译器会提示错误。
```
void z() {
   int i;
   i++;//Variable 'i' might not have been initialized
}
```
因此，对于局部变量来说，在使用前必须明确进行初始化。

对于类的数据成员（即字段）来说，编译器会对每一个数据成员提供一个初始值（自动进行初始化）。
```
public class InitialValues {
   boolean t;
   char c;
   byte b;
   short s;
   int i;
   long l;
   float f;
   double d;
   InitialValues initialValues;
   @Test
   public void test() {
       System.out.println("boolean: " + t);
      System.out.println("char: " + c);
      System.out.println("byte: " + b);
      System.out.println("short: " + s);
      System.out.println("int: " + i);
      System.out.println("long: " + l);
      System.out.println("float: " + f);
      System.out.println("double: " + d);
      System.out.println("initialValues: " + initialValues);
   }
}
```
结果如下图所示
 
对于基本数据类型编译器保证有一个初始值（char的初始值为0，ASCII码值为0，对应的的是空字符），对于引用数据类型，初始值为null。并且，无法阻止自动初始化的进行，所有对于 int i = 7; i首先被置为0，然后变成7，对于所有基本类型和对象引用，这种情况都是成立的。
初始化顺序
对于静态数据，无论创建多少个对象，静态数据都只占用一份存储区域。

初始化顺序为：静态数据成员初始化（仅第一次执行）、非静态数据成员初始化、构造器、其他方法。
其中静态数据成员包括静态代码块
```
class Car {
   static String name;
   static {
        name = "BMW";
   }
}
```
