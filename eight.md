# 第八章 多态
多态是继数据抽象和继承之后的第三种基本特征。
## 8.1 再讨论向上转型

在第七章中我们说过，对象引用既可以作为它自己本身的类型使用，也可以作为它的基类型使用，这种把某个类型引用作为它的基类型使用的做法被称为向上转型。
```
class Instrument{
	public Instrument() {
		//---
	}
	public void print(){
		System.out.println("Instrument-----:");
	}
}
class Wind extends Instrument{
	public void print(){
		System.out.println("Wind-----:");
	}
}
public class Music{
	public static void play(Instrument i){
		i.print();
	}
	public static void main(String[] args) {
		Wind wind = new Wind();
		play(wind);
	}
}
// output 
Wind-----:
```
Main方法中play方法传递wind引用的时候，不需要做任何类型转换。这样做是允许的，因为Wind自Instrument类继承而来，所以Instrument类的接口必定存在于Wind类中。这种向上转型的特性，避免了重新编写方法带来的程序代码冗余问题。

## 8.2 转机
当有多个子类的时候，编译器是怎样知道我们传递给基类引用的参数是哪个子类呢？比如我们把刚才的程序做个修改，再增加一个子类：
```
class Instrument{
	public Instrument() {
		//---
	}
	public void print(){
		System.out.println("Instrument-----:");
	}
}
class Wind extends Instrument{
	public void print(){
		System.out.println("Wind-----:");
	}
}
class Rain extends Instrument{
	public void print(){
		System.out.println("Rain-----");
	}
}
public class Music{
	public static void play(Instrument i){
		i.print();
	}
	public static void main(String[] args) {
		Rain rain = new Rain();
		play(rain);
	}
}
// output 
Rain-----
```
<br>
     在play方法中，既然做到了向上转型，那么编译器怎样知道类型从哪里转来的呢？即执行哪个对应的print()方法呢？解决这个问题有一个新的概念叫**后期绑定**，就是程序运行时根据对象的类型来进行绑定，也叫做动态绑定。一种语言要想实现动态绑定，那么它必须具有某种特定的机制来支持它在运行时准确的找到对象引用对应的类型，随着语言的不同这种机制有所不同，但大体上都是在对象中增加了某种类型信息。<br>
    **Java中除了static和final之外，其它所有的方法都是后期绑定**，所以我们无需显示的去做什么操作，因为动态绑定会自动发生。而前期绑定并不会对性能造成什么影响，使用final修饰的意图是防止被覆盖，并且告诉编译器这个是前期绑定，那么编译器可以更好的为其分配资源。<br>
    上述代码中，不同的子类与基类都有相同的方法（返回值、方法名、参数列表都相同），但是方法体内部不相同，这种操作叫做方法的重写或者方法的覆盖，即子类覆盖了父类方法的实现，当参数传递为子类对象的引用时，虽然看似调用了父类的这个方法，但是实际上由于动态绑定调用了子类的方法，实现了不同的功能。这也就是面向对象编程中多态的意义所在。

#### 8.2.4 缺陷： “覆盖” 私有方法
```
public class PrivateOverride{
	private void f(){
		System.out.println("private void f");
	}
	public static void main(String[] args) {
		PrivateOverride po = new Derived();
		po.f();
	}
}
class Derived extends PrivateOverride{
	public void f(){
		System.out.println("public void f");
	}
}
// output private void f
```
解析：私有方法被认为是final 不会被覆盖，而且是被继承类屏蔽的。所以Derived()中的f()当做一个新的方法执行。所以不能被覆盖。
#### 8.2.5 缺陷:  域与静态方法
对于域和静态方法都不是多态的。域是在访问的时候编译期进行解析的。
```
class Super{
	public int field = 0;
	public int getField(){
		return field;
	}
}
class Sub extends Super{
	public int field = 1;
	public int getField(){
		return field;
	}
	public int getSuperField(){
		return super.field;
	}
}
public class fieldAccess{
	public static void main(String[] args) {
		Super sup = new Sub();
		System.out.println("sup.field" + sup.field + "---sup.GetField" + sup.getField());
		Sub sub = new Sub();
		System.out.println("sub.field" + sub.field + "---sub.GetField" + sub.getField() + "---sub.GetSuperField" + sub.getSuperField());
	}
}
//output 
sup.field0---sup.GetField1
sub.field1---sub.GetField1---sub.GetSuperField0
```
解析：<br>
1. 任何域访问操作都将由对象编译器解析，因此不是多态的。 **Super.field 和 Sub.field 分配了不同的空间。**
2.  静态方法由于只与类有关，而不与对象牵连，因此它不存在多态的形式。

## 8.3 构造器的多态
构造器属于类的，它是一个隐式的static方法，因此不存在多态。这里继续分析构造器的调用过程。
```
class First{
	public First(){
		System.out.println("First---");
	}
}
class Second{
	public Second(){
		System.out.println("Second---");
	}
}
class Third{
	public Third(){
		System.out.println("Third---");
	}
}
class Fourty extends Third{
	public Fourty(){
		System.out.println("Fourty---");
	}
}
class Fifty extends Fourty{
	public Fifty(){
		System.out.println("Fifty---");
	}
}
public class Sixty extends Fifty{
	public Sixty(){
		System.out.println("Sixty---");
	}
	First first = new First();
	Second second = new Second();
	public static void main(String[] args) {
		new Sixty();
	}	
}
```
**类的加载过程是自上而下的，因此会先寻找基类进行加载，待所有继承类就加载完成之后，加载子类的成员变量以及构造函数。**
最终程序执行顺序如下：
1. 初始化基类中用到的静态变量，静态方法。
2. 初始化main（）方法中的常量，如果是有静态变量的对象，先初始化静态变量，然后加载其构造器。
3. 加载基类构造器。
4. 按顺序初始化成员变量。
5. 加载子类构造器。

## 8.4 协变返回类型

JavaSE5中新增了协变返回类型，在子类中的覆盖方法，可以返回其基类中该方法返回类型的某一个子类型。
```
class Father{
	public String toString(){
		return "Father";
	};
}
class Son extends Father{
	public String toString(){
		return "Son";
	}
}
class Mill{
	Father process(){
		return new Father();
	}
}
class wheatMill extends Mill{
	Son process(){
		return new Son();
	}
}
public class Test{
	public static void main(String[] args) {
		Mill m = new Mill();
		Father f = m.process();
		System.out.println(f);
		m = new wheatMill();
		f = new Son();
		System.out.println(f);
	}
}
// output 
Father
Son
```
## 8.5 用继承进行设计
继承会增加程序的复杂性，在程序设计过程中应优先使用组合。如果单纯的是想使用某一个类，让这个类的对象完成一些功能，那么使用组合会更好一些。此外， 对于继承中多态的实现，很大部分原因是由于向上转型与动态绑定，针对向上转型，与之相对的叫做向下转型，我们都知道向上转型是安全的，而由于扩展性的原因，向下转型并不是安全的。因为父类可能并没有子类中的一个方法。
## 8.6 总结

如果不运用数据抽象和继承，就不能季节多态的例子。 多态是面向对象“封装”、“继承”、“多态”三大特性之一，理解多态的特性能够更好的设计程序，同时，掌握程序初始化加载的过程能够更好的理解程序。

