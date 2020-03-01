---
title: Thinking in Java 第十九章
date: 2020-02-28 13:57:43
tags:
categories: Java
---
# 第十九章 枚举类型

关键字enmu可以将一组具名（就是具体的意思，翻译某些地方还是挺烂的）的值的有限集合创建为一种新的类型，而这些具体的值可以作为常规的程序组建使用。这是一种非常有用的功能。

## 19.1 基本enum特性

常用枚举方法： （可以用==来比较enum实例）
```
public class Main {

    enum Shrubbery {ONE,TWo,Three}
    public static void main(String[] args) {

        for(Shrubbery s : Shrubbery.values()){

            System.out.println(s.ordinal());
            System.out.println(s.getDeclaringClass());
            System.out.println(s.name());
            System.out.println(s == Shrubbery.ONE);
            System.out.println(s.equals(Shrubbery.ONE));
            System.out.println(s.compareTo(Shrubbery.ONE));
            System.out.println("--------");
        }

    }
}
```

还可以用import static 导入枚举类

## 19.2 向enum中添加新方法
 enum中的构造器与方法和普通的类没有区别，因为除了少许限制（不能继承自一个enum）外，enum就是一个普通的类。一旦enum的定义结束，编译器就不允许我们再使用其构造器创建任何实例。

覆盖enum的方法
枚举类也可以重写toString()方法，给我们提供了另一种方式来为枚举实例生成不同的字符串描述信息。

## 19.3 switch语句中的enum

Java SE7中switch支持字符串。

## 19.4 values()的神秘之处

Enum接口并没有values()方法。
values()是由编译器添加的static方法。（反编译后可查看）
Class中有一个getEnumConstants()方法，所以即便Enum接口中没有values()方法，我们仍然可以通过Class对象取得所有enum实例。

## 19.5 实现，而非继承
enum 类都继承Java.lang.Enum类。而且 不能再继承其它类。但是可以实现一个或者多个接口。
enum CartoonCharacter implements Generator<CartoonCharacter>{}

## 19.6 随机选取

使用泛型 random 对 枚举 随机选取。（具体看书中代码） 。

## 19.7 使用接口组织枚举
```
Interface Food{
    enum Apptizer implements Food{
        SALAD, SOUP
    }
    enum MainCourse implements Food{
        LASAGEN, BURRITO
    }    
}
```

由于枚举只能实现接口不能继承，所以遍历枚举需要如上这么写，向上转型 成 Food。

## 19.8 使用EnumSet替代标志

EnumSet的设计充分考虑到了速度因素，因为它必须与非常高效的bit标志相竞争。
它说明一个二进制位（bit）是否存在时，具有更好的表达能力，并且无需担心性能。
EnumSet的基础是long（64个元素）。
*注：EnumSet只是存 状态 0 代表无 1 代表有，真正的信息还是存在枚举里。*

## 19.9 使用EnumMap

EnumMap 底层由数组模式实现，因此实现速度很快，（**这里解释下为什么如果是枚举EnumMap快于hashmap。 1: 不需要计算hashCode()。 2: 只会申请指定枚举的空间大小，没有额外的空间浪费。**）
使用命令模式 实现下：
```
import java.util.EnumMap;
import java.util.*;
interface Command { void action();}
public class EnumMapTest {
    enum AlarmPoints {KITCHEN,BATHROOM,UTILITY}

    public static void main(String[] args) {
        EnumMap<AlarmPoints,Command> em = new EnumMap<>(AlarmPoints.class);

        em.put(AlarmPoints.KITCHEN, new Command()  {
            public void action() {
                System.out.println("Kitchen fire!"); }
        });
        em.put(AlarmPoints.BATHROOM, new Command() {
            public void action() { System.out.println("Bathroom alert!"); }
        });
        for(
                Map.Entry<AlarmPoints,Command> e : em.entrySet()) {
                System.out.println(e.getKey() + ": ");
                e.getValue().action();
        }
        try { // If there’s no value for a particular key:
            em.get(AlarmPoints.UTILITY).action();
        } catch(Exception e) {
            System.out.println(e);
        }
    }

}

```

与常量相关的方法相比，EnumMap允许改变值对象。

## 19.10 常量的相关方法

书中例子为 表驱动法 （解析：从表里面查找信息而不是使用逻辑语句（if…else…switch），当是很简单的情况时，用逻辑语句很简单，但如果逻辑很复杂，再使用逻辑语句就很麻烦了。）
```
import java.text.DateFormat;
import java.util.Date;

public enum ConstantSpecificMethod {
    DATE_TIME {
        String getInfo() {
            return DateFormat.getDateInstance().format(new Date());
        }
    },
    CLASSPATH {
        String getInfo() {
            return System.getenv("CLASSPATH");
        }
    },
    VERSION {
        String getInfo() {
            return System.getProperty("java.version");
        }
    };
    abstract String getInfo();
    public static void main(String[] args) {
        for(ConstantSpecificMethod csm : values())
            System.out.println(csm.getInfo());
    }
}
```
与使用匿名内部类相比，定义常量相关方法的语法更高效、简洁。
虽然enum有些限制，但一般而言，我们还是可以将其看做是类。

#### 19.10.1 使用enum的职责链

Enum定义的次序决定了各个解决策略在应用时的次序。（职责链是一种设计模式，类似击鼓传花、部门审批流程。本级处理不了交给上级处理。程序员以多种不同的方式来解决问题，然后将他们链接在一起。当一个请求来临时，她遍历这个链，直到链中的某个解决方法能够处理该请求。搜索 enum 职责链 查看相关代码，书中例子太长了……）

#### 19.10.2 使用enum的状态机

状态机可以具有有限个特定的状态，它根据输入，从一个状态转移到下一个状态，不过也可能存在瞬时状态，而一旦任务执行结束，状态机就会立刻离开瞬时状态。每个状态都具有某些可接受的输入，不同的输入会使状态机转移到不同的新状态。由于enum对其实例有严格限制，非常适合用来表现不同的状态和输入。

## 19.11 多路分发
#### 19.11.1 enum 多路分发
 （挺拧巴的 原理就是按照枚举顺序提前定义好顺序，循环去比，不好理解。实操起来也比较麻烦）
```
enum Outcome { WIN, LOSE, DRAW }
public enum RoShamBo2 implements Competitor<RoShamBo2> {
    // 在此处是按顺序定义好，比如 PAPER vs PAPER = DRAW
    ROCK(Outcome.LOSE, Outcome.WIN, Outcome.DRAW),
    PAPER(Outcome.DRAW, Outcome.LOSE, Outcome.WIN),
    SCISSORS(Outcome.WIN, Outcome.DRAW, Outcome.LOSE);
    private Outcome vPAPER, vSCISSORS, vROCK;
    RoShamBo2(Outcome paper,Outcome scissors,Outcome rock) {
        this.vPAPER = paper;
        this.vSCISSORS = scissors;
        this.vROCK = rock;
    }
    public Outcome compete(RoShamBo2 it) {
        switch(it) {
            default:
            case PAPER: return vPAPER;
            case SCISSORS: return vSCISSORS;
            case ROCK: return vROCK;
        }
    }

    public static <T extends Competitor<T>> void match(T a, T b) {
        System.out.println(a + " vs " + b + ": " + a.compete(b));
    }
    /**
     * 随机产生指定数量个场景（对象）进行游戏
     * 调用了Enums工具方法Enums.random
     * <T extends Enum<T> & Competitor<T>> 规定传入的参数必须同时是Enum<T>和Competitor<T>类型
     * @param rsbClass 用来比赛的对象
     * @param size 数量
     */
    public static <T extends Enum<T> & Competitor<T>>
    void play(Class<T> rsbClass, int size) {
        for(int i = 0; i < size; i++) {
            match(Enums.random(rsbClass), Enums.random(rsbClass));
        }
    }
    public static void main(String[] argv) {
        play(RoShamBo2.class, 20);
    }

}
```
#### 19.11.2 使用常量相关的方法

常量相关的方法允许我们为每个enum实例提供方法的不同实现，这使得常量相关的方法似乎是实现多路并发的完美解决方案

#### 19.11.3 使用EnumMap 分发
实现“真正”的两路分发。
#### 19.11.3 使用二维数组
原理都差不多，定义好类似矩阵顺序，去数组中匹配。

## 19.12 总结
枚举本身不复杂， 但是与Java的其他功能结合使用，涉及的功能还是挺细碎繁琐的。尤其是多路分发，需要很好的理解枚举类顺序。
