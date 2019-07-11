# 2. 一切都是对象
## 2.1 用引用操作对象

String s; 只是创建引用并初始化。String s = "asdf"; 这个代码只是把s初始化。引用到 "asdf" 的内存地址。类似于c的引用指针，只不过把对象抽象成内存地址。

## 2.2 必须由你创建的所有对象
（这标题一看就是翻译过来的） 只看标题一脸蒙蔽……
String s = new String("asdf"); 创建好对象了。
#### 2.2.1 储存到什么地方
有五种不同的地方可以存储数据：

1. 寄存器，这是最快的存储，在处理器内部
2. 栈，这是存在随机访问内存，但是有处理器通过 栈指针 的直接支持 （一般存放确定好的静态变量，通过指针分配内存）
3. 堆，这是通用的存储区域，所有 Java 对象都存储在这
4. 常量存储，常量值一般都直接放置于程序代码内
5. 非随机访问内存存储，完全存储在程序外的数据，和该程序是否在运行无关，且不受该程序控制
#### 2.2.2 基本类型
介绍基本的数据类型以及占用的空间大小。
#### 2.2.3 Java重的数组
Java 数组一定会被初始化并且无法访问其区间之外的部分，位置索引在运行时会被确认是否有效。（Java 看到null 就知道这个指针还没有指到某个对象。程序会报错。所以常规的数组错误在Java可以避免。）

## 2.3 永远不要销毁对象
Java 会自动帮我们销毁对象。
#### 2.3.1 作用域
作用域是由花括号 {} 的位置决定。作用域之内定义的变量在花括号结束前都可以被访问。
```
{
    int x = 12;
    {
        int x = 96; // Illegal
    }
}
```
编译器报错（x已经定义过了）。白话就是**外面东西里面能用，里面的东西外边用不了**
#### 2.3.2 对象的作用域
new 一个Java 对象 他可以存活在作用域之外。Java 有 垃圾回收 机制，它观察所有使用 new 创建的对象，判断哪些对象不再被引用，并释放那些不再被引用的对象。你可以只管创建对象，当你不使用他们时，他们会自行被释放。

## 2.4 创建新的数据类型： 类
例如：
```
class ATypeName { /* This is Class body */ }

ATypeName a = new ATypeName();
```
类创建好了！ 就是这样！
#### 2.4.1 字段和方法
对象内可以创建字段和方法，但是普通字段不能在对象间共享
```
class DataOnly {
    int i;
    double d;
    boolean b;
}
```
想要给对象里字段赋值 先引用一个对象 然后对其字段进行赋值操作。
```
DataOnly data = new DataOnly();

data.i = 47;
data.d = 1.1;
data.b = false;
```
类中的某个成员是基本数据类型 即使没有初始化，Java也会给一个默认值。防止程序错误产生。

基本数据类型	默认值<br>

boolean	false <br>
char	‘\u0000’ (null)<br>
byte	(byte)0<br>
short	(short)0<br>
int	0<br>
long	0L<br>
float	0.0f<br>
double	0.0d<br>
但是上面说的不适用于 局部变量，局部作用域中不初始化会报错。

## 2.5 方法、参数和返回值
介绍了方法的组成
```
ReturnType methodName (/* Argument list */){
    /* Method body */
}
```
#### 2.5.1 参数列表
方法的参数有强类型控制 如果是String 类型 则必须传String 对象 否则报错。

## 2.6 构建一个Java程序
#### 2.6.1 名字可见性
Java 为了保证命名唯一，采用了反转域名的规范建立包名，例如 net.mindview
.my.project.
#### 2.6.2 运用其他构件
引用包写法
```
import java.util.*;
```
#### 2.6.3 static 关键字
1. static 修饰的变量是共享内存的
2. static 字段对每个类来说都只有一份存储空间，而非static对象则是每个对象都有一个存储空间。
3. static 好处是不创建任何对象就能调用它，这一点对定义main()入口文件很重要。

## 2.7 你的第一个Java程序
在每个程序文件的开头，都必须放置一个import语句
```
import java.util.*;
public class HelloDate {
    public static void main(String[] args) {
        System.out.println("Hello, it's: ");
        System.out.println(new Date());
    }
}

```
main()的形式是固定的，如上

虽然并为用到 args，但是Java编译器必须要这样做，因为 args 要用来存储命令行参数
（Java 最强大的优势之一就是它具有庞大的标准类库集）
#### 2.7.1 编译和运行
javac HelloDate.java <br>
java HelloDate<br>
注 ：**JDK11以后 不需要用javac 编译 直接java 编译+运行 （Java也在像轻量改变）** 

## 2.8 注释和嵌入式文档
Java里提供两种注释风格。

第一种风格，注释以“/*”开始，随后是注释内容，并可跨越多行，最后以“*/”结束。<br>
第二种风格，单行注释，以“//”开头，直到句末。
#### 2.8.1 注释文档
javadoc 将文件同文档链接起来。将注释提取出来放在一个文件内。javadoc输出的是一个HTML文件，可以用Web浏览器查看。
#### 2.8.2 语法
独立文档标签是一些以“@”字符开头的命令，且要置于注释行的最前面。
所有的javadoc命令都只能在“/**”注释中出现，注释结束于“*/”。
```
/** A class comment */
public class Documentation1 {
    /** A field comment */
    public int i;
    /** A method comment */
    public void f() {}
}
```
**javadoc只能为public和protected成员进行文档注释。private和包内可访问成员的注释会被忽略掉。可以使用-private进行标记，可以将private成员的注释也包括在内。**
#### 2.8.3 嵌入式HTML
所有类型的注释文档都支持嵌入式HTML。
#### 2.8.4 一些标签示例
1. @see：引用其他类
```
/**
 * @see classname
 * @see fully-qualified-classname
 * @see fully-qualified-classname#method-name
 */
```
2. {@link package.class#member label} 该标签与@see极其相似，只是它用于行内，并且用“label”作为超链接文本。
3. {@docRoot} 该标签产生到文档根目录的相对路径，用于文档树页面的显式超链接。
4. {@inheritDoc}
该标签从当前这个类的最直接的基类中继承相关文档到当前的文档注释中。
5. @version 代码版本的相关信息
6. @author 作者
7. @since 指定程序最早使用的版本
8. @param 参数
9. @return 返回值
10. @throws 产生异常的相关信息。
11. @deprecated 新特性改进。
#### 2.8.5 文档示例
```
//:first/HelloDate.java
import java.util.Date;
/**
 *  The first Thinking in Java example program.
 *  Displays a string and today's date.
 * @author xjd
 * @author www.xxx.com.cn
 * @version 4.0
 */
public class HelloDate {
    /**
     *  Entry point to class & application
     * @param args array of string arguments
     * @throws exceptions No exceptions thrown
     */
    public static void main(String[] args) {
        System.out.println("Hello, it's: ");
        System.out.println(new Date());
    }/* Output:(55% match)
    Hello,it's:
    Thu Jun 28 17:56:45 CST 2018
    *///:~
}
```
第一行 用一个“:”作为特殊几号说明这是包含原文件名的注释行。

“///:~”标志源代码清单的结束。

/*Output标签表示输出的开始部分将由这个文件生成，通过这种形式，它会自动地测试以验证其准确性。

(55% match)在向测试系统说明程序的当前运行和醒一次运行的输出存在这很大差异，55%是两次输出与其的相关性程度。

## 2.9 编码风格
类名 首字母大写的**驼峰风格**
```
class AllTheColorsOfTheRainbow{
    int anIntegerRepresentingColors;
    void changeTheHueOfTheColor(int newHue) {}
}
```
## 2.10 总结
大家了解如何编写一个简单程序的Java编程知识，对Java语言及它的一些基本思想也有了一个总体认识。