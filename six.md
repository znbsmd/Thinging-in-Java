### **由于此书讲的太细，一小节都总结太细 太影响效率，故从此章节改为大体总结，不扣细节。**

# 第六章 访问权限控制

为了让自己的程序变得更加易用、可读和完善的。在重构修改的过程中，避免他们对自己类库内部的程序进行改动。Java中提供了访问权限控制的概念，提供了三种不同级别的访问控制，访问开放程度由高到低依次为“public”、“protected”“private”，这样就能区分哪些内容是可用的，哪些内容是不可用的，从而将变动的事物与不变的事物区分开来。此外，Java提供了package加以控制，而访问权限控制的作用会因为类库是否在一个相同的package还是不同的package受到影响。

## 6.1 包: 库 单元
#### 6.1.1  ～ 6.1.5
 包内包含一组类，它们在单一的名字空间下被组织在了一起。声明一个类所属的包使用package关键字，同时在另一个包中的类要访问其它包中的类使用import关键字导入要使用的包。这种方式可以在一定程度上避免重名的问题，因为包的名字要避免重名，而不同包内的类是可以根据具体的需求命相同的名字。包有效的将不同类的内容进行了隔离，同时也可以相互联系。
```
* // access/mypackage/MyPackagedClass.java
*
* package access.mypackage;
*
* public class MyPackagedClass {
*	public MyPackagedClass() {System.out.println("MyPackagedClass()");}
* }	
*/
// 以上是 新建package 里java

public class UnpackagedMyClass {
	public static void main(String[] args) {
		access.mypackage.MyPackagedClass m = new access.mypackage.MyPackagedClass();
	}
}
```

## 6.2 Java 访问权限修饰词
1. public：所有可见，被public修饰的内容在同一个包中的所有类都可见。同时Java提供默认的访问权限，即不被任何修饰符修饰的内容默认为public权限。

2. private：私有可见，只有该类可见，该类的对象都不可见。如果一个类的构造函数被声明为private，那么就不能通过这个类的构造函数来进行初始化对象。

3. protected:受保护的可见，与private不同，除了只有自己的类可见之外，该类的继承者也可见被修饰的域。**除此之外还可以被当前包的类访问，但是其它包的类不可以访问，即便是使用了import的关键字**

访问权限控制对程序结构控制的重要手段。

## 6.3 接口和实现
为了方便他人使用，会在具体方法实现外部建立一层接口，只提供接口给外部开发人员调用，而不提供具体实现的方法。

## 6.4 类的访问权限

在Java中，访问权限控制也可以确定包中的哪些类可以被访问，也就是说可以用来修饰类，一个文件中最多只能有一个使用public修饰的类。如果希望客户端程序员使用该类，并可以创建对象，那么就可以将该类修饰为public。并且被修饰为public的类必须要与该文件的名字完全相同

## 6.5 总结
讲述了public、private和protected 三种类型的概念和应用场景。