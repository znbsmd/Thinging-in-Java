# 第五章 初始化与清理
C 语言中 很多开发者 会忘记清理使用过的变量 导致内存耗尽。<br>
C++ 引入构造器 即 创建对象时被自动调用的方法。<br>
Java 中也采用了构造器，并额外提供了“垃圾回收器”。
## 5.1用构造器确保初始化
 Java中通过提供构造器，保证每个类的对象都进行了初始化。
```
public class Rock{
	Rock(){
		System.out.println("Rock");
	}
	public static void main(String[] args) {
		for(int i=0;i<5;i++){
			new Rock();
		}
	}
}
// output
 Rock Rock Rock Rock Rock
```
## 5.2 方法重载
对于这种方法名相同，方法参数不同的写法，称作是重载。方法重载不仅支持构造器重载，也支持普通方法重载
```
public class test{
    private int age;
    private String name;
    test(){

    }
    test(int i){
        age=i;
    }
    test(String j){
        name=j;
    }
    test(int i,String j){
        age=i;
        name=j;
    }
    public static void main(String[] args) {
        test ct1 = new test();
        test ct2 = new test(22);
        test ct3 = new test("张三");
        test ct4 = new test(23,"李四");
        System.out.println("ct1 age=" + ct1.age + "---name=" + ct1.name);
        System.out.println("ct2 age=" + ct2.age + "---name=" + ct2.name);
        System.out.println("ct3 age=" + ct3.age + "---name=" + ct3.name);
        System.out.println("ct4 age=" + ct4.age + "---name=" + ct4.name);
    }
}
// output 
ct1 age=0---name=null
ct2 age=22---name=null
ct3 age=0---name=张三
ct4 age=23---name=李四

```
#### 5.2.1 区分重载方法
独一无二的参数类型列表，(顺序不同也可以区分不同的方法)
#### 5.2.2 涉及基本类型的重载
重载方法中如果传入了低类型的参数，那么如果找不到合适的方法，会被隐式的提升成高类型的数据。如果传入了高类型的参数，如果不进行类型转换，那么编译器会报错。**如果无法找到恰好接受char参数的方法，就会把char直接提升至int。**
#### 5.2.3 以返回值区分重载方法
可以使用返回值区分，当然前提是如果你关心方法的返回值。
```
void f(){}
int f() {return 1;}
```

## 5.3 默认构造器
又名“无参”构造器 作用是创建默认对象。 如果你没有显示的在代码中创建一个构造器，那么Java会自动帮你创建一个无参构造器来完成初始化。 理解：**创建对象必须要构造器，要不Java也不回费劲帮你创建默认构造器**
## 5.4 this 关键字
表示当前对象的引用，并返回当前对象
```
public class Leaf{
    int i = 0;
	Leaf increment(){
        i++;
		return this;
	}
    void print(){
        System.out.printLn("i = " +i);
    }
    public static void main(String[] args) {
		Leaf x = new Leaf();
        x.increment().increment().increment().print();
	}
}
// output i = 3
```
解析：increment() 通过this 返回对当前对象的引用。<br>
**必要时候在使用this，我们使用高级语言的原因就是希望他们来帮我么简化一些事情。从而节省时间**

#### 5.4.1 在构造器中调用构造器
```
public class test{
    test(int i,String j){
        this(j);
//!        this(j,i);
        System.out.println("我是构造器1");
    }
    test(String arg){
        System.out.println("我是构造器2");
    }
    test(String arg, int x){
        System.out.println("我是构造器3");
    }
    testMothod(String arg, int x){
        //！ this(11);
        System.out.println("我是方法不能调用构造器");
    }
    public static void main(String[] args) {
        test tt = new test(2,"嘻嘻");
    }

```
以上代码说明 构造器可以仅用this调用另一个构造器，不能调用二个。并且 方法内不能调用构造器。
“//！“ 代表编译错误。

#### 5.4.2 static的含义
static 内部不能使用 this 反之可以。<br>
**代码出现大量static，该重新考虑自己设计了**

## 5.5 清理：终结处理和垃圾回收
Java中的内存清理使用的是Java自带的垃圾回收机制，Java的垃圾回收机制清理的是通过new创建的对象，而在某些特殊情况下，可能有些对象不是通过new创建的，这些对象如果不使用的时候，垃圾回收器是不能准确的清理他们的从而造成了这块特殊的内存区域一直得不到释放。Java中提供了finalize()方法来处理这一部分特殊的内存区域。<br>
垃圾回收只能回收 new 出来的对象
**1. 对象可能不被垃圾回收**
**2. 垃圾回收并不等于“析构”**<br>

流程是 ：在Java垃圾回收器启用之前，会先调用这个方法进行一些必要的回收操作。但是针对这一块特殊的区域，或者是new创建的需要被回收的对象，一般情况下只有当Java虚拟机内存快要消耗殆尽的时候，垃圾回收器才会启动，毕竟启动垃圾回收器也是需要消耗资源的，所以不可能说实时存在。

#### 5.5.1 finalize() 用途何在
**3. 垃圾回收只和内存相关**
```
finalize()方法的通用格式如下：

protected void finalize( )
{
// finalization code here
}
```
Java 中 只有不是new 出来的对象 才用finalize()清除，由于Java 一切皆为对象，所以这种特殊情况可能采用了类C语言的做法，而非Java 通用代码。这种方法主要发生在“本地方法”中，一种Java调用非Java代码。<br>
所以不要过多用到finalize()，它不适合进行普通清理工作。
#### 5.5.2 你必须实施清理
所以 将 Java 规范化 调用恰当的Java 方法 就不会用到 finalize()。
#### 5.5.3 终结条件
调用 system.runFinalization() 会强制调用已经失去引用的对象的finalize方法 。
#### 5.54 垃圾回收器如何工作
写的比较长 总结一下：<br>
1. 垃圾回收发生在Java虚拟机中。
2. 引用计数是一种简单但速度很慢的回收技术。详解：引用技术是当对象被引用时候计数+1，当引用离开作用域-1，发现引用计数为0 释放占用空间。缺点就是两个对象循环调用时候，引用计数却不为0。
定位这类型的交互引用的对象需要的工作量极大。
3. Java垃圾回收并非基于引用计数技术。采用了一种自适应的垃圾回收技术。做法为"停止-复制"，但是这种有弊端 。具体google。
所以当没有新垃圾产生 会切换到“标记-清扫”。具体 google。
4. JIT编译技术 用以提升速度。它可以把程序的全部或部分翻译成本地机器码。（这本来是Java虚拟机要做的事情。）运行速度加以提升。
5. 惰性评估 执行多的代码才会被JIT编译。这也是代码每次执行的优化。 所以执行越多 速度越快。

## 5.6 成员初始化
Java尽量保证每个变量在使用之前都进行了初始化操作，变量分为局部变量和成员变量，局部变量如果没有显示的初始化，在使用它的时候会报错，而成员变量不会，如果没有显示的初始化一个成员变量，那么它会被默认的分配一个指定的值。

#### 5.6.1 指定的初始化
定义变量时候赋值，但在C++ 里不能这样做。

## 5.7 构造器初始化
构造器无法阻止自动初始化。它在构造器被调用之前发生。
#### 5.7.1 初始化顺序
在类内部，即使变量定义散布在方法定义之间，它仍然会在任何方法（包括构造器）被调用之前得到初始化。**初始化顺序为创建对象时，先初始化这个类的静态变量，然后在堆上为这个对象分配内存，最后执行构造函数。**
#### 5.7.2 静态数据的初始化
如前边对构造器的阐述，可以使用构造器来初始化类的成员变量，当对象被实例化之后，对象的成员变量会被初始化。**静态成员变量只有在第一次使用它是才会被初始化，后边再次用到时不会被初始化。**
#### 5.7.3 显示的静态初始化
```
class Cup {
    Cup(int marker) {
        System.out.println("Cup(" + marker + ")");
    }
    void f(int marker) {
        System.out.println("f(" + marker + ")");
    }
}

class Cups {
    static Cup cup1;
    static Cup cup2;
    static {
        cup1 = new Cup(1);
        cup2 = new Cup(2);
    }
    Cups() {
        System.out.println("Cups()");
    }
}

public class test {
    public static void main(String[] args) {
        System.out.println("Inside main()");
//        Cups.cup1.f(99); // (1)
    }
    static Cups cups1 = new Cups(); // (2)
    static Cups cups2 = new Cups(); // (2)
}
```
上面这段代码很好的解释了 静态初始化。无论创建几个对象，静态初始化都只会进行一次。
#### 5.7.4 非静态实例初始化
与 static 一样 初始化是在构造之前完成的。不同的是 多次调用 多次执行。
```
class Tester {
    String s;
    {
        s = "Initializing string in Tester";
        System.out.println(s);
    }
    Tester() {
        System.out.println("Tester()");
    }
}

public class test {
    public static void main(String[] args) {
        new Tester();
        new Tester();
    }
}
// output 
Initializing string in Tester
Tester()
Initializing string in Tester
Tester()
```

## 5.8 数组初始化
数据是一系列相同数据类型封装起来的序列，它的初始化可以发生在任何时候，int[] a1表示一个int类型的数组，这个数组内部所有的值都是int类型，a1只是这个数组的一个引用，可以显示的通过如int[] a1={1，2，3};的形式进行初始化。**如果不能确定数组的内容或者是长度，则可以通过new的形式来创建一个数组。**int[] a = new int[20];这种创建也只是创建了一个引用数组，直到数组中的每一个字段都有确切的值了，初始化才真正的完成。如a[1]=3;
#### 5.8.1 可变参数列表
&nbsp; &nbsp; &nbsp; &nbsp; 使用数组我们可以构建一个变参的函数，就是当方法的参数类型和个数都不确定的时候，我们可以使用一个数组作为形参。因为Object类是所有类型的父类，所以这个形参数组的类型就是Obejct类，对于基本数据类型，因为都有对应的包装类，所以也可以转换成Obejct类进行使用。
JDK1.5之后真正的变参函数 Object...args

## 5.9 枚举类型
这里只是初步的了解一下枚举类型，enum，Java中的enum要比C++更加完备。
```
public class EnumEx21 {
	public enum Bills {
		ONE, FIVE, TEN, TWENTY, FIFTY, HUNDRED
	}
	public static void main(String[] args) {
		for(Bills b : Bills.values())
			System.out.println(b + ", ordinal " + b.ordinal());	
	}	
}
```
枚举还可以当构造函数使用，非常方便。

这里理解枚举暂时可以当常量集理解 具体19章会详解。

## 5.10 总结

Java 初始化 有至关重要的地位。构造器能够保证正确初始化和清理。（没有正确的构造器调用，编译器就不允许创建对象）<br>

C++ 中 析构很重要。 Java就不太需要。垃圾回收可以简化编程，处理上也安全。<br>

由于需要保证所有对象被创建，构造器实际上会更加复杂。尤其是组合或继承。


