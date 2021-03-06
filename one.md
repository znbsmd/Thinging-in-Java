# 第一章.对象导论
## 1.1 抽象过程
  总结一点 从一门脚本语言(php,python,javascript)转学到一门工业级语言，必须要转变的思想 就是 **万物皆为对象** 

## 1.2 每个对象都有一个接口
  一张图清晰概括所有内容 ！
  ![avatar](https://github.com/znbsmd/photo/raw/master/object-interface.jpeg)

## 1.3 每个对象都提供服务
   将对象看作是服务提供者<br>
  
   举例: 一个记账系统 将功能拆分成3个对象 <br>
   1. 输入屏幕对象
   2. 执行计算对象
   3. 打印发票对象<br>

从而实现将功能简单化 和加强代码可用性和复用性，每个对象都看成一种服务，它会使其对象以适应其设计的过程简单的多。
## 1.4 被隐藏的具体实现

1. 对于客户端开发者（业务程序员）不需要关注类创建者（底层服务类）如何实现，只需要关注如何快速开发应用，拿来即用。从而减少程序bug 减轻开发任务。

2. 对于类创建者（底层服务类）改变内部工作方式而不用担心影响到 客户端开发者（业务程序员）例如 优化 维护。可以清晰的分离得以保护。

3. public、private、protected 关键字 

4. java 默认访问权限 -> 包访问权限

## 1.5 复用具体实现

此节讲的比较抽象。 引用书中的一句话做为总结 :<br>
最简单的复用某个类的方式就是直接使用该类的一个对象，将这个类的一个对象放在一个新类中。新的类可以由任意的其他对象实现想要的功能方式所组成。这种概念就是组合。这种组合关系就是复用。（简单理解就是下一节说的继承……）

## 1.6继承

 继承创造了新的类，也可以在新的类中添加新的方法。完成代码复用。
#### 1.6.1 "是一个" 与 "像是一个" 关系

文字表达很啰嗦 用书中一张图足以总结 ![avatar](https://github.com/znbsmd/photo/raw/master/1.6.1.jpeg)

热力泵 是新类 空调是原有类 新cool() 优化替换 原有 cool() 做为"纯粹替换"。

## 1.7 伴随多态的可互换对象

1. java 多态的特性 基于 1.5 复用 和 1.6 继承 的更深一层的扩展 继承 把最基类的对象泛化 具体解析可google java 多态特性 网站有超级白话和详细说明帮助理解多态 。
2. 后期绑定概念 : 非面向对象程序采用**前期绑定**  面向对象程序采用**后期绑定** 这样才能在一个对象发送消息时候才能知道这条消息应该做什么

## 1.8 单根继承结构

在单继承结构中的所有对象都具有一个共用接口，所以它们归根到底都是相同的基本类型，3个单根继承的优点 :
1. 所有对象都可以很容易地在堆上创建
2. 参数传递也得到了极大的简化
3. 垃圾回收器的实现变得容易得多

## 1.9 容器
为了解决 存储多个对象 为多个对象创造多个空间问题 容器很好的解决了这个问题。例如 : List、Map、Set 以及诸如队列、树、堆栈等更多的数据结构。
#### 1.9.1 参数化类型
容器的参数化类型机制：创建的容器，知道自己所保存的对象的类型，从而不需要向下转型以及消除犯错误的可能。这种机制在java中 ，称为泛型。
 ArrayList<Shape>  shapes = new ArrayList<Shape>( );

## 1.10 对象的创建和生命期
java对象创建一种是创建在栈上 在编译前确定 追求执行速度 但是牺牲了灵活性 分配的内存大小时候固定的。另一种创建对象存放在堆上 称为动态分配方式。为防止内存泄漏 java编译器 采用 垃圾回收器 机制 并自动销毁。

## 1.11 异常处理 ： 处理错误
java 内置了异常处理，并且强制你使用它。

## 1.12 并发编程
采用多线程 处理并发。弊端就是共享资源。所以 在资源使用期间加锁 完成后释放。java并发也是内置在语言中。

## 1.13 java与internet
java 解决了在万维网上的程序设计问题。所谓的互联网。

#### 1.13.1 web 是什么
就是服务器，使得 客户和服务器 在同一个网络中进行交互。

#### 1.13.2 客户端编程
介绍了脚本语言所解决的问题是有限的，其余的还需要java去解决。

#### 1.13.3 服务端编程
复杂可用的系统还需要java去实现，具体论述在《企业Java编程思想》中论述。

## 1.14 总结
编写良好的java代码比等价的过程型语言要简单，也易于理解。最后要清楚认识到 oop和java 能否解决你的问题，要有清楚的认识。








