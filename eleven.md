# 第十一章 持有对象
  之前，我们使用的都是固定数量的且生命周期已知的对象。而在一些情况中，我们可能需要不确定数量不确切类型的对象，这种创建一个单一的对象显然是不行的了。Java提供了多种支持，比如数组，数组可以保存一组基本数据类型。但是数组的大小是固定的，在更特殊的编程条件下，固定长度显然是不友好的，所以Java类库提供了一套相当完整的容器类来解决这个问题。我们也称作是集合类。本章优先学习常用的集合以及用法，后续将会更加深入的讨论其它的集合。

## 11.1 泛型和类型安全的容器

 在JavaSE5泛型的概念出来之前，容器的一个主要问题就是编译器允许你在容器中插入任意类型的对象，这在一些情况下显然是不合理也是不可靠的。考虑这样一个情况，有一个存储Apple对象的容器，我们使用最基本最可靠的ArrayList，ArrayList你可以看成是一个可以自动扩充的数组。ArrayList需要使用索引就像数组下标一样，但是不需要使用方括号，它使用add（）插入对象，使用get（）获取对象，使用size（）获取长度。