# 第七章 复用类
## 7.1 组合语法
当实例对象是一个对象的引用时，编译器则为其初始化为null，这样在使用这个对象时，就会发生错误。因此我们需要如下几个方法，初始化组合中的引用：
1. 在定义对象的地方，也就是对象引用建立的时候直接指定对象，这意味着对象的初始化发生在构造器调用之前。
```
class Construct{
	public String cont = "Hello World";
}
```
2. 在类的构造器中，如第5章所述，我们可以通过构造器的形式，也就是new，初始化一个对象。
```
package com.chenxyt.java.practice;
class Construct{
	public String cont;
	Construct(String cont){
		this.cont = cont;
	}
}
public class ConstructTest {
	Construct construct = new Construct("Hello");
}
```
3. 就在这样使用这些对象之前，也就是惰性初始化，当我们必须要使用这个对象的时候，发现没有初始化该对象呢，那么进行初始化，这种方式可以提供系统性能，降低资源消耗，在必须用到的时候才创建。
```
public class ConstructTest {
	public static String s;
	public static void main(String[] args) {
		if(s==null){
			s=new String("Hello");
		}
	}
}
```
4. 使用实例初始化，这种与构造器的相同之处都是在代码中显式的去创建一个对象，区别在于一个调用了构造器方法，一个赋值了一个实例。
```
public class ConstructTest {
	public static String s;
	public static void main(String[] args) {
		s="Hello";
	}
}
```
## 7.2 继承语法

另一种复用类的手段就是继承，我们创建一个新的类，如果没有显示的指明继承哪个类，那么它默认继承自Object类，我们也可以通过extends关键字继承一个指定的类。继承表现的是一种is-a的关系或者说是like-a，被继承的类称作父类或者基类，继承之后新生成的类称为子类或者派生类。子类拥有父类中所有被public或protected域修饰的方法、属性。
```
package com.chenxyt.java.practice;
class Father{
	public String Name;
	public int age;
	private double money;
	public void doPrint(){
		System.out.println("Hello World");
	}
}
public class Son extends Father {
	public static void main(String[] args) {
		Son son = new Son();
		son.Name = "Zhang San";
		son.doPrint();
		//money是父类私有的域，所以子类不能进行访问
		//son.money = 2014;
	}
}
```

## 7.3 代理
在使用组合的过程中，不提供成员对象的直接访问，而是通过一个新的方法暴露出来给其它使用者调用。避免直接暴露组合的对象。
```
package com.chenxyt.java.practice;
class Construct{
	public void print(){
		System.out.println("print");
	}
	public void say(){
		System.out.println("say");
	}
	public void clean(){
		System.out.println("clean");
	}
}
public class Son{
	public Construct construct = new Construct();
	public void delegation(){
		construct.clean();
	}
	public static void main(String[] args) {
		Son son = new Son();
		son.delegation();
	}
}
```

## 7.4 结合使用组合和继承
代码说明：
```
package com.chenxyt.java.practice;
class Father{
	//---
}
class Mother{
	//---
}
public class Son extends Father{
	public Mother mother = new Mother();
	public static void main(String[] args) {
		//---
	}
}
```
一个继承父类的子类中引用一个其它的类实例对象。
**子类可以继承父类的全部，所以对于父类重载了的方法，子类也是可以继承并可以重载使用。**
```
package com.chenxyt.java.practice;
class Father{
	public void doFunc(){
		//---
	}
	public void doFunc(String s){
		//--
	}
}
public class Son extends Father{
	public void doFunc(int i){
		//--
	}
	public static void main(String[] args) {
		Son son = new Son();
		son.doFunc();
		son.doFunc("H");
		son.doFunc(1);
	}
}
```

## 7.5 在组合和继承之间选择
组合和继承都允许在新类中放置子对象，组合是显式的，而继承是隐式的。一般情况下，使用组合的地方多出是想使用其对象，就是在新类内部引用一个对象，然后利用这个对象完成一系列功能。而继承的使用场景主要是新类想使用原来类的一部分接口这种。在使用组合和继承的时候，要注意访问权限控制。

## 7.6 protected关键字
 主要是为了控制继承类的访问权限。父类方法或变量被子类使用的这种。

## 7.7 向上转型
编译器会知道子类传递的参数引用属于父类哪一种类型，并将其子类对象引用的类型转换成父类，这个过程称作向上引用。
```
package com.chenxyt.java.practice;
class Father{
	private static String s;
	Father(String s){
		this.s = s;
	}
	public static void doFunc(Father father){
		System.out.println(s);
	}
}
public class Son extends Father{
	Son(String s) {
		super(s);
		// TODO Auto-generated constructor stub
	}
 
	public static void main(String[] args) {
		Son son = new Son("Hello");
		Father.doFunc(son);
	}
}
```
解析：父类中有一个static的方法，并且方法的参数是父类的对象，在子类中直接通过类名.方法的形式调用了该方法，并传递了子类的对象引用作为参数。编译器没有报错。

## 7.8 final关键字

1. final 数据 ：一个固定的常量，或者一个在运行时候被初始化的值而我们却不希望它被改变，那么就可以使用final来修饰。
```
public class FinalTest{
	private static final String ARG_NAME = "Hello World";
	public static void main(String[] args) {
		System.out.println(ARG_NAME);
	}
}
```
2. final参数：Java中允许方法使用final修饰的参数，这意味着这个参数指向的对象不可以被修改.
```
class Gzino{
	public void doFunc(){
		//--
	}
}
public class FinalTest{
	public void with(final Gzino g){
		//error! 此处不可以修改该对象的引用
		//g = new Gzino();
		//可以使用该参数
		g.doFunc();
	}
	public void without(Gzino g){
		//没有被final修饰，可以使用
		g = new Gzino();
		g.doFunc();
	}
	public static void main(String[] args) {
		FinalTest ft = new FinalTest();
		ft.with(new Gzino());
		ft.without(new Gzino());
	}
}

```
3. final方法：类中所有的private域都被隐式的注为private，因为不允许其它人修改。还有一种是final类，被final类修饰的方法我们认为这个类不能被其它人修改，也不能被继承，当你想达到这样的目的时，可以使用final类。

## 7.9 初始化及类的加载
在许多语言中，程序在启动的过程中发生了加载，然后是初始化，最后是程序运行。但Java采用不同的加载机制， 每个类的编译文件都在它自己的代码文件中，该文件只有在用到该程序的时候才被加载。所以一般可以说“类的代码在初次使用时才发生加载“，这通常是指加载发生在创建第一个类对象时，但是当static域被访问时，也会发生加载。
```
class Insect{
	private int i = 9;
	protected int j;
	public Insect() {
		System.out.println("i=" + i + "--j=" + j);
		j=39;
	}
	private static int x1 = printInit("static Insect.x1 initialized");
	static int printInit(String s){
		System.out.println(s);
		return 53;
	}
}
public class LoadTest extends Insect{
	private int k = printInit("LoadTest.k initialized");
	public LoadTest(){
		System.out.println("k= " + k);
		System.out.println("j= " + j);
	}
	private static int x2 = printInit("static LoadTest.x2 initialized");
	public static void main(String[] args) {
		System.out.println("LoadTest constructor");
		LoadTest lt = new LoadTest();
	}
}
// output
static Insect.x1 initialized
static LoadTest.x2 initialized
LoadTest constructor
i=9--j=0
LoadTest.k initialized
k= 53
j= 39
```
解析：程序在启动的时候先查找main函数，main函数都是程序的入口，发现main方法里面有LoadTest类，这时开始加载LoadTest类，在加载的过程中，发现这个类有基类，于是他继续加载基类，如果还有基类那么继续加载另一个基类，因为子类中可能会用到基类的内容。然后基类中的static域初始化，这时第一行打印。然后子类中的static域初始化，这时第二行执行。这时必要的类资源已经加载完毕，可以进行对象创建，第三行打印。初始化对象，首先对象中的所有基本类型被赋初始值0，对象被赋值为null，这是通过将对象内存设置为二进制零实现的。然后基类的构造函数被调用，这里是自动调用的，也可以通过super方法直接调用，这时第四行被打印。基类构造器和子类构造器执行相同的方式经过相同的过程。基类构造器执行完毕之后，实例变量按顺序被创建，最后执行子类的构造方法，所以最后两行被打印。

## 7.10 总结
 讲解就看小节就okay了。 这里说下注意点：
 组合一般是将现有类型做为新类型底层实现的一部分加以复用。
 继承复用的是接口。<br>
 导出类举有基类接口，由此可以向上转型。这对多态来说很关键。<br>
 理解final。
