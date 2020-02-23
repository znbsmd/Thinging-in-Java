---
title: Thinking in Java 第十六章
date: 2020-02-23 16:41:52
tags: 
categories: Java
---
# 第十六章 数组

&emsp;&emsp;时隔半年，又看回来了，在有点java基础知识和代码开发经验去面试发现基础细节还是不到位，于是需要继续巩固基础和细节……（任重而道远）

&emsp;&emsp;数组这章讲的比较少，简单总结一下， 数组尺寸不能该改变。大多时候，更复杂的结构，容器会更加适合。

## 16.1 数组为什么特殊
效率，效率，效率。没有泛型的容器是不可以持有基本类型的。 数组可以。

## 16.2 数组是一级对象

[]语法是访问数组对象的唯一方式，可以隐式创建和显示的创建（new）。

## 16.3 返回一个数组

相比 c/c++可以返回一整个数组，而不用为数组负责。使用完后，垃圾回收器会自动清理。

## 16.4 多维数组

```
int[][] a = {{1,2,3},{4,5,6}};
```
## 16.5 数组与泛型

数组和泛型不能很好的结合，因为编译器会擦除参数化类型信息，（具体可以查看.class文件）而数组必须知道确切类型，保证类型安全。

可以参数化数组本身类型 ，但是毫无意义。还可以创建一个非泛型数组，把泛型数据转型放入。

```
List<String> ls;
List[] la = new List[10];
ls = (List<string>[]) la;
```
## 16.6 创建测试数据

Array.fill(); 填充数组。 Generator 数据生成器。

## 16.7 Array 实用功能

Arrays工具类有equals() sort() binarySearch() toString() hashCode() asList()

复制数组 System.arraycopy() 注⚠️ System.arraycopy() 不会自动包装和自动拆包，两个数组必须有确切类型。

Arrays重写了 equals() 比较数组。 数组相等的条件是元素个数必须相等，并且对应元素也相等。

数组元素比较 java.lang.Comparable  与 java.util.Collects.Comparator。
Comparable 具有天生的比较 只有 compareTo() 方法。
Comparator 可以自定义 重写compare 和equal。

数组排序
```
Arrays.sort(sa, String.CASE_INSENSITIVE_ORDER);
// 忽略大小写，默认是大写在前
```

Java 标准类库中的排序算法针对正排序的特殊类型进行了优化 – 针对基本类型设计的 ”快速排序“（Quicksort），以及针对对象设计的 ”稳定归并排序“。

如果数组已经排好序了，可以使用 Arrays.binarySearch() 执行快速查找。如果找到了目标，返回值大于等于 0，否则，它产生负返回值，表示若要保持数组的排序状态此目标元素所应该插入的位置。这个负值的计算公式是

- (插入点) - 1
”插入点“ 是指，第一个大于查找对象的元素在数组中的位置。

如果数组包含重复的元素，则无法保证找到的是这些副本中的哪一个。

如果使用 Comparator 排序了某个对象数组（基本类型数组无法使用 Comparator 进行排序），在使用 binarySearch() 时必须提供同样的 Comparator（使用 binarySearch() 方法的重载版本）。

## 16.8 总结

数组强调性能而不是灵活。 java的设计如果当时抛弃了设计基本类型低级数组。java也许会成为真正的纯面向对象语言。随着时间推移，容器基本上是更好的选择。



