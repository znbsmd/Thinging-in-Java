# 第十三章 通过异常处理错误
Java基本理念是 “结构不佳的代码不能运行。”

发现错误的理想时机是在程序的编译过程中。异常处理是Java唯一正式的报告机制。

## 13.1 概念
&emsp;&emsp;使用异常处理能够降低处理代码的复杂度。使用异常的另一个好处是，它能够明显的降低代码的复杂程度，避免了大量的错误检查，只需要在一个特定的地方进行异常捕获，并且不需要做任何判断，异常捕获区能够捕获所有发生的错误。这种异常的处理方式与之前的错误处理方式相比，完全的将“正常做的事儿”与“出现问题怎么办”隔离开来。是代码的读写变得更加井井有条。

## 13.2 基本异常

对于一个对象引用t，传递给你的时候可能没有被初始化，所以在使用这个对象引用调用执行方法之前，进行合理的判断是非常有必要的。
```
if(t==null){
    throw new NullPointerException();
}
```
 上面代码中，我们判断当前对象引用是否没有进行初始化，如果没有进行初始化，那么就创建一个NullPointerException（）对象，然后使用throw关键字，将该对象的引用抛出。

## 13.3 捕获异常

关键字try后包围的一部分代码块，称作try块，这样做比起之前说的要在每个会产生错误的地方进行判断要容易的多，并且代码的可读性大大增强，产生异常之后抛出的异常必须在某处得到处理，这个地点就是异常处理区，异常处理区紧跟着try块，使用catch关键字表示：
```
try{ //---}catch(Type1 arg1){ //---}catch(Type2 arg2){}
```
## 13.4 创建自定义异常

Java中虽然提供了很多默认的异常类型，但是要想完全覆盖会发生的异常情况显然是不现实的，因此我们可以自定义异常来表示我们预期可能会出现的异常。自定义的形式也非常简单，只需要继承一个相似的异常类即可。建立一个新的异常类最贱的方法就是让编译器为你产生默认构造器。
```

class SimpleException extends Exception{}  
public class InheritingException {  
    public void f() throws SimpleException{  
        System.out.println("Throw SimpleException from f");  
        throw new SimpleException();  
    }  
    public static void main(String[] args) {  
        InheritingException ite = new InheritingException();  
        try{  
            ite.f();  
        }catch(SimpleException e){  
            System.out.println("Caught it!");  
        }  
    } 
}

```
带有参数的构造器
```
class MyException extends Exception{
    public MyException(){
		
	}
	public MyException(String msg){
		super(msg);
	}
}
public class FullConstructors {
	public static void f() throws MyException {
		System.out.println("Throwing myException from f()");
		throw new MyException();
	}
	public static void g() throws MyException{
		System.out.println("Throwing myException from g()");
		throw new MyException("Originated in g()");
	}
	public static void main(String[] args) {
		try{
			f();
		}catch(MyException e){
			e.printStackTrace();
		}
		try{
			g();
		}catch(MyException e){
			e.printStackTrace(System.out);
		}
	}
}
```
**有参和无参数区别可以从控制台错误信息的眼神看出。标准错误流更加的引人注目。**

## 12.5 异常说明
Java提供了相应的语法（并强制使用这个语法），使你可以使用礼貌的方式告知客户端程序员某个方法可能会抛出的异常类型，然后客户端程序员就可以进行相应的处理。这就是异常说明，它属于方法声明的一部分，紧跟在形式参数列表之后，使用throws关键字，后面接一个所有潜在异常类型的列表，所以方法定义可能看起来像是这样：
```
void f() throws TooBig,TooSmall,DivZero{};

```

## 12.6 捕获所有异常

```
catch（Exception e）{
    //---
}
```
- String getMessage（） 获取详细信息
- String getLocalizedMessage（）获取用本地语言表示的详细信息
- String toString（）返回对Throwable的简单描述
- void printStackTrace（）打印Throwable的调用栈轨迹 输出到标准错误流
- void printStackTrace（PrintStream）打印Throwable的调用栈轨迹 输出到可选择的流
void printStackTrace（PrintStream）打印Throwable的调用栈轨迹 输出到可选择的流

```
public class ExceptionMethods {
	public static void main(String[] args) {
		try{
			throw new Exception("My Exception");
		}catch(Exception e){
			System.out.println("Caught Exception");
			System.out.println("getMessage():" + e.getMessage());
			System.out.println("getLocalizedMessage():" + e.getLocalizedMessage());
			System.out.println("toString()" + e.toString());
			System.out.println("printStackTrace():--==");
			e.printStackTrace();
			System.out.println("printStackTrace(System.out):--==");
			e.printStackTrace(System.out);
		}
	}
}
```

printStackTrace（）方法所提供的信息可以使用getStackTrace（）方法获取到，这个方法返回一个由栈轨迹元素所构成的数组，其中每一个元素都表示栈中的一帧。元素0为栈顶元素，表示调用序列最后的一个方法，也就是Throwable创建和抛出的地方，数组中最后一个元素为方法调用序列中的第一个调用方法。下面程序简单演示：
```
public class WhoCalled {
	static void f(){
		try{
			throw new Exception();
		}catch(Exception e){
			for(StackTraceElement ste : e.getStackTrace()){
				System.out.println(ste.getMethodName());
			}
		}
	}
	static void g(){
		f();
	}
	static void h(){
		g();
	}
	public static void main(String[] args) {
		f();
		System.out.println("------------");
		g();
		System.out.println("------------");
		h();
	}
}
```

## 12.7 Java标准异常

 Throwable这个类表示任何可以被作为异常抛出的类，Error表示编译时的系统错误，一般不需要我们关心。Exception是可以被抛出的基本类型，在Java类库、用户方法以及运行时故障都可以抛出此类异常。所以Java程序员通常需要关心此类的异常信息，同时Java异常要求可以见名知意。Java中还有一种Runtime异常，它表示程序编译错误，无法由程序员自己控制，这种异常不需要我们自己主动捕获并抛出，程序会自动抛出此类异常，如前文提到的NullPointerException。

## 12.8 使用finally进行清理

一些代码，可能希望无论是否抛出异常，这些代码都必须要执行。这通常包括内存回收之外的一些操作，因为内存回收是由垃圾回收器完成。为了达到这个效果，可以在catch块之后加上finally块。在finally块中处理这些代码。完整的异常处理如下：
```
try{
	//---
}catch(Exception e1){
	//---
}catch(Exception e2){
	//---
}finally{
	//---
}

```
finally字句不管异常是否抛出都能被执行：

```
public class FinallyWorks {
	static int count = 0;
	public static void main(String[] args) {
		while(true){
			try{
				if(count++ == 0){
					throw new Exception();
				}
				System.out.println("No Exception");
				}catch(Exception e){
					System.out.println("Exception");
				}finally{
					System.out.println("In finally clause");
					if(count == 2){
						break;
					}
				}
			}
		}
	}
```
不管异常是否被抛出，finally字句都执行了。这个程序告诉我们，当程序发生了异常之后不能正确的回到原来的执行顺序上，我们可以在finally块中处理一些必要做的事情。比如打开文件的操作，当文件操作发生异常的时候，我们要确保文件被正确的关闭。

## 12.9 异常的限制

异常会被继承的基类和接口所限制
```
class BaseballException extends Exception{
	
}
class Foul extends BaseballException{
	
}
class Strike extends BaseballException{
	
}
abstract class Inning{
	public Inning() throws BaseballException{
		
	}
	public void event() throws BaseballException{
		
	}
	public abstract void atBat() throws Strike,Foul;
	public void walk(){
		
	}
}
class StormException extends Exception{
	
}
class RainedOut extends StormException{
	
}
class PopFoul extends Foul{
	
}
interface Storm{
	public void events() throws RainedOut;
	public void rainHard() throws RainedOut;
}
public class StormyInning extends Inning implements Storm{
 
	public StormyInning() throws BaseballException {
		super();
		// TODO Auto-generated constructor stub
	}
 
	@Override
	public void rainHard() throws RainedOut {
		// TODO Auto-generated method stub
		
	}
 
	@Override
	public void atBat() throws Strike, Foul {
		// TODO Auto-generated method stub
		
	}
 
	@Override
	public void events() throws RainedOut {
		// TODO Auto-generated method stub
		
	}
	//抽象类中的该方法没有抛出任何异常 所以覆盖重写的时候 也不能抛出异常
//	public void walk() throws RainedOut{
//		
//	}
	//抽象类中的该方法抛出了异常 重写的时候可以选择不抛出异常
//	public void event(){
//		
//	}
	//抽象类中的该方法抛出了BaseballException 所以可以抛出其异常的子异常
	public void event() throws Foul{
		
	}
}
```

当继承或者实现一个类的时候，这个子类只能抛出基类中异常列表中的异常，或者其子异常，或者不抛出异常，但是不能抛出其它异常，或者抛出异常的基类异常。

## 12.10 构造器
“如果异常发生了，所有东西能被正确清理吗？”大多数情况都是非常安全的，可以使用finally进行清理。但是当使用构造函数的时候，就有些问题了。比如使用构造函数构造文件操作，因为finally语句不管构造成功或者失败都会执行。假如构造失败了，如果文件没有被打开，那么这时候如果执行finally关闭文件操作就是不正确的。还有，如果文件构造成功了，因为文件需要在后边使用，所以在finally中关闭文件显然是不合理的。
```
public class InputFile {
	private BufferedReader in;
	public InputFile(String name) throws Exception{
		try{
			in = new BufferedReader(new FileReader(name));
		}catch(FileNotFoundException e1){
			//没有找到文件也就是文件没有打开 所以不需要清理
			System.out.println("The file is not found");
			throw e1;
		}catch(Exception e2){
			//所有其它的异常都发生在文件打开之后 因此需要进行清理工作
			try{
				in.close();
			}catch(IOException e3){
				//---
			}
			throw e2;
		}finally{
			//此处由于构造成功也会执行 所以不能进行文件关闭操作
		}
	}
}
```
 对于这种需要在构造器中创建的对象，合理的做法是使用嵌套try-catch
```
public class CleanUp {
	public static void main(String[] args) {
		try{
			InputFile in = new InputFile("CleanUp.java");
			try{
				//
			}catch(Exception e1){
				//此处为文件操作异常
			}finally{
				//-- 此处清理操作关闭文件
			}
		}catch(Exception e2){
			//--此处捕获构造异常
		}
	}
}
```

## 12.11 异常匹配
在抛出异常之后，异常处理系统会按照代码的编写顺序找到最近的异常处理程序并执行，此时便任务异常已经得到了处理，就不会继续进行下去。查找的时候并不要求抛出的异常同处理程序所声明的异常完全匹配，子类的对象也可以匹配到其基类的处理程序。
```
class Annyoance extends Exception{
	
}
class Sneeze extends Annyoance{
	
}
public class Human {
	public static void main(String[] args) {
		try{
			throw new Sneeze();
		}catch(Sneeze s){
			System.out.println("Catch Sneeze");
		}catch(Annyoance a){
			System.out.println("Catch Annyoance1");
		}
		try{
			throw new Sneeze();
		}catch(Annyoance a){
			System.out.println("Catch Annyoance2");
		}
	}
}

```
 编译器发现Annyoance是Sneeze的基类，所以编译的过程会提示警告。
## 12.12 其他可选方式
- 历史 在java中 对于范型用于异常说明的方式存在着一些限制。
- 观点 好的程序设计语言能帮助写出好程序，但是避免不了程序员用它写出坏程序
- 把异常传递给控制台
- 被检查的异常转换成不检查的异常 RuntimeException可以抛出异常，但是还是要用 try 捕获你想你肯定想得到的异常。

## 12.13 异常使用指南

1. 知道如何处理情况下捕获
2. 解决问题重新调用产生问题的方法
3. 进行修补，绕过异常发生的地方继续执行。
4. 用别的数据进行计算，以代替方法预计会返回的值。
5. 把当前运行环境的事情做完，相同的异常抛出到更高层。
6. 把当前运行环境的事情做完，不相同的异常抛出到更高层。
7. 终止程序。
8. 简化异常。
9. 让类库和程序更安全。

## 12.14 总结
异常是Java编程中不可或缺的一部分，它将程序正常执行的路径与错误处理分开，减少了编码判断的冗余。使用try-catch的方式进行异常处理，同时我们也可以创建自己的异常，继承其它的异常类。一些RuntimeException无需我们自己捕捉，程序会自动捕捉。注意构造器内发生异常的情况，因为构造器往往是只是简单的构造了一个对象，对象还需要使用，所以在使用finally进行清理的时候需要注意这一点。因为finally域的代码块不管异常是否发生都会执行。处理异常的时候可以使用基类Throwable提供的方法，比如使用printStackTrace打印栈的调用顺序。


