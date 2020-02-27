---
title: Thinking in Java 第十七章
date: 2020-02-23 20:11:47
tags:
categories: Java
---
# 第十七章 容器深入研究

Java容器类库成熟 完备 易用。

## 17.1 完整的容器分类法

Java SE5新添加了：
- Queue接口及其实现PriorityQueue及各种风格的BlockingQueue（多线程）；
- ConcurrentMap及其实现的ConcurrentHashMap（多线程）；
- CopyOnWriteArrayList和CopyOnWriteArraySet（多线程）；
- EnumSet和EnumMap（enum）；
- Collection类中的多个便利方法。

## 17.2 填充容器

Collections.nCopies() 所有引用都被设置为指向相同的对象；例： 
```
List<StringAddress>  ls = Collections.nCopies(4, new StringAddress("helli"));
//output ls
[StringAddress@82ba41 Hello,StringAddress@82ba41 Hello,StringAddress@82ba41 Hello,StringAddress@82ba41 Hello]
```

Collections.fill() 只能替换已经存在的元素，而不能添加新元素。

#### 17.2.1 一种Generator解决方案

Collection生成器CollectionData：

**所有collection子类都有一个接受另一个Collection对象的构造器，用来填充数据**

CollectionData体现了适配器模式的设计思想，它能把实现Generator接口的类的对象(包括上一章数组中的各种RandomGenerator)都复制到自身当中

**直接上代码，看完代码在看中文解释会更清晰（tij 的心得，哈哈，上来看简介是真看不懂，再不济可以Google ，只要弄明白他要说的 再回来看解析就会清晰很多）**
```
import java.util.ArrayList;
import java.util.LinkedHashSet;
import java.util.Set;

interface Generator<T> {
    T next();
}

class CollectionData<T> extends ArrayList<T> {
    private static final long serialVersionUID = 1L;
    public CollectionData(Generator<T> gen, int quantity) {
        for (int i = 0; i < quantity; i++)
            add(gen.next());
    }
    public static <T> CollectionData<T> list(Generator<T> gen, int quantity) {
        return new CollectionData<T>(gen, quantity);
    }
}

class Animal implements Generator<String> {
    private String[] items = { "Dog", "Pig", "Cat" };
    private int index;
    @Override
    public String next() {
        return items[index++];
    }
}

public class Main {
    public static void main(String[] args) {
        Set<String> set = new LinkedHashSet<String>(new CollectionData<String>(new Animal(), 2));
        System.out.println(set); // [Dog, Pig, Cat]
        set.addAll(CollectionData.list(new Animal(), 3));
        System.out.println(set); // [Dog, Pig, Cat]
    }
}
```

#### 17.2.12 Map生成器

思想与2.1 相同

```
public MapData(Generator<Pair<K,V>> gen,int quantity){
    Pair<K,V> p=gen.next();
    put(p.key,p.vaule);
}
```

#### 17.2.3 使用Abstract类

**享元模式**使得对象的一部分可以被具体化，因此，与对象中的所有事物都包含在对象内部不同，我们可以在更加高效的外部表中查找对象的一部分或整体。

书中的代码和解析比较不容易理解和啰嗦。用网上一段例子说明
典型的享元工厂类的代码如下：
```
class FlyweightFactory {
    //定义一个HashMap用于存储享元对象，实现享元池
    private HashMap flyweights = newHashMap();
    public Flyweight getFlyweight(String key){
        //如果对象存在，则直接从享元池获取
        if(flyweights.containsKey(key)){
            return(Flyweight)flyweights.get(key);
        }
        //如果对象不存在，先创建一个新的对象添加到享元池中，然后返回
        else {
            Flyweight fw = newConcreteFlyweight();
            flyweights.put(key,fw);
            return fw;
        }
    }
}
 
```
不存在才创建，存在就get出来。

## 17.3 Collection的功能方法

主要介绍Collection的主要方法，子类可以拿来即用。

## 17.4 可选操作

实现类并不需要为这些方法提供功能定义。简单的说就是 List Map Set 都可以共用 Collection接口的方法，但是 就像 List 不能使用set 一样,虽然编译器不会检查，但是运行会抛出异常。

## 17.5 List 的功能方法

常用list方法 iterator 等等。

## 17.6 Set和存储顺序

良好的编码风格：应该在覆盖equals()方法时，总是同时覆盖hashCode()方法。

SortedSet
1. Comparator comparator() 返回当前Set中使用的Comparator；或者返回null表示以自然方式排序。
2. Object first() 返回容器中第一个元素。
3. Object last() 返回容器中第末一个元素。
4. SortedSet subSet(fromElement, toElement) 生成此Set的子集，范围从fromElement（包含）到toElement（不包含）。
5. SortedSet headSet(toElement) 生成此Set的子集，由小于toElement的元素组成。
6. SortedSet tailSet(fromElement) 生成此Set的子集，由大于或等于formElement的元素组成。

**SortedSet的意思是按对象比较函数对元素排序，而不是元素插入次序，插入顺序可以用LinkedHashSet保存。**
https://blog.csdn.net/trbbadboy/article/details/6939598
贴上简单明了的例子

## 17.7 队列

1. 优先级队列 PriorityQueue  https://www.jianshu.com/p/c577796e537a 这篇文章讲解到位。
Queue指先进先出的队列。Java中Queue为一个接口，集成Collection接口，实现可以为LinkedBlockingQueue,LinkedList,PriorityQueue等。
普通队列特点，先进先出。插入队列和移出队列的时间复杂度都为O(1)。
优先级队列，不同于普通队列，优先级队列默认按照元素的自然顺序排序（也可以重写comparator接口compare方法进行自定义排序）插入。移出队列和普通队列一样，但插入队列的时间复杂度与元素个数N有关，时间复杂度为O（logN）。
 
2. 双向队列 Deque  
    两端放入元素并提取他们，不是特别常用
```
public class Deque<T> { 
    private LinkedList<T> deque = new LinkedList<T>(); 
    public void addFirst(T e) { deque.addFirst(e); } 
    public void addLast(T e) { deque.addLast(e); } 
    public T getFirst() { return deque.getFirst(); } 
    public T getLast() { return deque.getLast(); } 
    public T removeFirst() { return deque.removeFirst(); } 
    public T removeLast() { return deque.removeLast(); } 
    public int size() { return deque.size(); } 
    public String toString() { return deque.toString(); } 
    // And other methods as necessary... 
}
```

## 17.8 理解Map

开始的Map 用get 取值 效率最低 因为要从头轮询equals比较key值 ，只是简单并没有效率。

#### 17.8.1 性能

get()线性搜索，效率低下，HsahMap 因此而生。HsahMap使用了散列码，，hashcode() 是 Object中的方法，因此所有的Java对象都能产生散列码。

Treemap 查询键值对他们会被排序（由Comparable 或者 Comparator决定）。

#### 17.8.2 SortedMap

使用SortedMap（TreeMap是其现阶段的唯一实现），可以确保键处于排序状态。

1. Comparator comparator() 返回当前Map中使用的Comparator；或者返回null表示以自然方式排序。
2. T firstKey() 返回Map中的第一个键。
3. T lastKey() 返回Map中的第末一个键。
4. SortedMap subMap(fromKey, toKey) 生成此Map的子集，范围从fromKey（包含）到toKey（不包含）的键确定。
5. SortedMap headMap(toKey) 生成此Map的子集，由小于toKey的键确定的键值对组成。
6. SortedMap tailMap(fromKey) 生成此Map的子集，由大于或等于fromKey的键确定的键值对组成。

#### 17.8.2 LinkedHashMap

两个特点：
1. 为了提高速度，散列化所有元素，，遍历时候又以元素的插入顺序返回键值对，
2. 可以在构造器里第三个参数设置accessOrder true，在我们访问了一个Entry<K,V>时，我们会调用afterNodeAccess()方法，将我们当前访问的节点放入到链表的末尾，利用这个特性便可以区分谁是最近访问，谁是最近最不常访问元素了。boolean removeEldestEntry(Map.Entry)该方法返回值为true时，会删除最近最不常使用的元素，也就是double-link的头部节点，当插入一个新节点之后removeEldestEntry()方法会被put()、putAll()方法调用，我们可以通过override该方法，来控制删除最旧节点的条件。
同redis淘汰策略的 **LRU 算法**。没有访问的放在最前面。对于定期清理节省空间或实现LRU缓存以下案例得以实现。

简单案例：
```
import java.util.Iterator;
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.Map.Entry;
import java.util.Set;

/*
 * 为最近最少使用（LRU）缓存策略设计一个数据结构，
 * 它应该支持以下操作：获取数据（get）和写入数据（set）。
 * 获取数据get(key)：如果缓存中存在key，则获取其数据值（通常是正数），否则返回-1。
 * 写入数据set(key, value)：如果key还没有在缓存中，则写入其数据值。
 * 当缓存达到上限，它应该在写入新数据之前删除最近最少使用的数据用来腾出空闲位置。 
 *
 */
public class LRUCache<K, V>{
    LinkedHashMap<K,V> cache = null;
    private int cacheSize;
    public LRUCache(int cacheSize){
        this.cacheSize = (int) Math.ceil (cacheSize / 0.75f) + 1;  // ceil浮点数向上取整数
        cache = new LinkedHashMap<K,V>(this.cacheSize,0.75f,true){  //boolean accessOrder用来控制访问顺序的，默认设置为false，在访问之后，不会将当前访问的元素插入到链表尾部
          //内部类来重写removeEldestEntry()方法
        @Override
          protected boolean removeEldestEntry (Map.Entry<K, V> eldest){
              System.out.println("size="+size());
              return size() > cacheSize; //当前size()大于了cacheSize便删掉头部的元素
          }
        };
    }
    
    public V get(K key){   //如果使用继承的话就用getE而不是get，防止覆盖了父类的该方法
       return (V) cache.get(key);
    }
    public V set(K key,V value){
       return cache.put(key, value);
    }
    
    public int getCacheSize() {
        return cacheSize;
    }

    public void setCacheSize(int cacheSize) {
        this.cacheSize = cacheSize;
    }
    
    public void printCache(){
        for(Iterator it = cache.entrySet().iterator();it.hasNext();){
            Entry<K,V> entry = (Entry<K, V>)it.next();
            if(!"".equals(entry.getValue())){
                System.out.println(entry.getKey() + "\t" + entry.getValue()); 
            }
        }
        System.out.println("------");
    }
    
    public void PrintlnCache(){
        Set<Map.Entry<K,V>> set = cache.entrySet();
        for(Entry<K,V> entry : set){
            K key = entry.getKey();
            V value = entry.getValue();
            System.out.println("key:"+key+"value:"+value);
        }
        
    }
    
    public static void main(String[] args) {
        LRUCache<String,Integer> lrucache = new LRUCache<String,Integer>(3);
        lrucache.set("aaa", 1);
        lrucache.printCache();
        lrucache.set("bbb", 2);
        lrucache.printCache();
        lrucache.set("ccc", 3);
        lrucache.printCache();
        lrucache.set("ddd", 4);
        lrucache.printCache();
        lrucache.set("eee", 5);
        lrucache.printCache();
        System.out.println("这是访问了ddd后的结果");
        lrucache.get("ddd");
        lrucache.printCache();
        lrucache.set("fff", 6);
        lrucache.printCache();
        lrucache.set("aaa", 7);
        lrucache.printCache();
    }

}

```
## 17.9 散列与散列码

Object的hashCode()默认使用当前对象的地址计算散列码，equals()比较对象的地址。

覆盖hashCode()的同时覆盖equals()方法，正确的equals()方法的五个条件：

- 自反性
- 对称性
- 传递性
- 一致性
- 对任何不是null的x，x.equals(null)一定返回false

#### 17.9.1 理解hashcode()散列码

hashMap 使用 key 的 hashcode 来保存键值对。

#### 17.9.2 为速度而散列

通过键对象生成一个数字，将其作为数组的下标。这个数字就是散列码，由定义在Object中的、且可被覆盖的hashCode()（散列函数）方法生成。

查询一个值的过程首先是计算散列码，然后使用散列码查询数组。如果能够保证没有冲突（如果值的数量是固定的，那么就有可能），那就有了一个完美的散列函数，但是这种情况是特例。通常，冲突由外部链接处理：数组并不直接保存值，而是保存值的List。

由于散列表中的“槽位”（slot）通常被称为桶位（bucket），因此我们将表示实际散列表的数组命名为bucket。为使散列分布均匀，桶的数量通常使用质数。

**保证时间复杂度O(1) 是在 没有hash冲突的情况下。**
***

**HashMap源码分析：**
（引用自网络）

在这之前，先介绍一下负载因子和容量的属性。大家都知道其实一个HashMap的实际容量就是因子×容量，其默认值是16×0.75=12；这个很重要，对效率很一定影响！当存入HashMap的对象超过这个容量时，HashMap就会重新构造存取表。这就是一个大问题，我后面慢慢介绍，反正，如果你已经知道你大概要存放多少个对象，最好设为该实际容量的能接受的数字。

两个关键的方法，put和get：

先有这样一个概念，HashMap是声明了Map，Cloneable, Serializable接口，继承AbstractMap类，里面的Iterator其实主要都是其内部类HashIterator和其他几个iterator类实现，当然还有一个很重要的继承了Map.Entry的Entry内部类。Entry内部类包含了hash、value、key和next这四个属性，很重要。put的源码如下：
```
public Object put(Object key, Object value) {
    Object k = maskNull(key); // 这个就是判断键值是否为空，如果为空，它会返回一个static Object作为键值，这就是为什么HashMap允许空键值的原因。
    int hash = hash(k);
    int i = indexFor(hash, table.length);
    // 其中hash就是通过key这个Object的hashCode()进行hash，然后通过indexFor获得在Object table的索引值。
    // put其实是一个有返回的方法，它会把相同键值的put覆盖掉并返回旧的值。如下方法彻底说明了HashMap的结构，其实就是一个表加上在相应位置的Entry的链表：
    for (Entry e = table[i]; e != null; e = e.next) {
        if (e.hash == hash && eq(k, e.key)) {
            Object oldvalue = e.value;
            e.value = value; // 把新的值赋予给对应键值。
            e.recordAccess(this); // 空方法，留待实现
            return oldvalue; // 返回相同键值的对应的旧的值。
        }
    }
    modCount++; // 结构性更改的次数
    addEntry(hash, k, value, i); // 添加新元素，关键所在！
    return null; // 没有相同的键值返回
}
```

我们把关键的方法拿出来分析：
```
void addEntry(int hash, Object key, Object value, int bucketIndex) {
    
    table[bucketIndex] = new Entry(hash, key, value, table[bucketIndex]);
    // 因为hash的算法有可能令不同的键值有相同的hash码并有相同的table索引，它们经过indexfor之后的索引一定为i，这样在new的时候这个Entry的next就会指向这个原本的table[i]，再有下一个也如此，形成一个链表，和put的循环对定e.next获得旧的值。
    if (size++ >= threshold) // 这个threshold就是能实际容纳的量
        resize(2 * table.length); // 超出这个容量就会将Object table重构
        // 所谓的重构也不神，就是建一个两倍大的table，然后再一个个indexfor进去。注意，这就是效率，如果你能让你的HashMap不需要重构那么多次，效率会大大提高！

 ```
 **所谓的重构也不神，就是建一个两倍大的table，然后再一个个indexfor进去。注意，这就是效率，如果你能让你的HashMap不需要重构那么多次，效率会大大提高！**

 #### 17.9.2 覆盖hashCode()

无论何时，对同一个对象调用hashCode()都应该生成相同的值。

要使hashCode()实用，它必须速度快，并且有意义。（即必须基于对象内容生成散列码）。

那么如何设计一个 hashCode 算法，书中设计了一个算法：

1. 把某个非 0 的常数值，比如 17，保存在一个名为 result 的 int 类型的变量中。
2. 对于对象中的每个域，做如下操作：

    * 为该域计算 int 类型的哈希值 c ：

        - 如果该域是 boolean 类型，则计算 (f?1:0)

        - 如果该域是 byte、char、short 或者 int 类型，则计算 (int)f
        - 如果该域是 long 类型，则计算 (int)(f^(f>>>32))
        - 如果该域是 float 类型，则计算 Float.floatToIntBits(f)
        - 如果该域是 double 类型，则计算 Double.doubleToLongBits(f)，然后重复第三个步骤。
        - 如果该域是一个对象引用，并且该类的 equals 方法通过递归调用 equals 方法来比较这个域，同样为这个域递归的调用 hashCode，如果这个域为 null，则返回0。
        - 如果该域是数组，则要把每一个元素当作单独的域来处理，递归的运用上述规则，如果数组域中的每个元素都很重要，那么可以使用 Arrays.hashCode 方法。


    * 按照公式 result = 31 * result + c，把上面步骤 2.1 中计算得到的散列码 c 合并到 result 中。

    **选择值31是因为它是奇数质数。如果是偶数且乘法运算溢出，则信息将丢失，因为乘以2等于移位。使用质数的优势尚不清楚，但这是传统的。31的一个不错的特性是乘法可以用移位和减法来代替，以获得更好的性能：31 * i == (i << 5) - i。JVM自动执行这种优化。**


3. 返回 result
 

## 17.10 选择接口的不同实现

Map、List、Set和Queue

#### 17.10.1 性能测试框架

计算所有操作所需的纳秒数。封装好Test 对象 使用。输出各种容器的纳秒数。

#### 17.10.2 对List 的选择

将ArrayList作为默认首选，只有需要使用额外的功能，或者当程序的性能因为经常从表中间进行插入和删除操作而变差时，才去选择LinkedList。如果使用的是固定数量的元素，那么既可以使用背后有数组支撑的List，也可以选择真正的数组。

#### 17.10.3  微基准测试的危险 

剖析器可以把性能分析工作做得更好。Java Profiler

#### 17.10.4 对Set的选择

HashSet的性能基本上总是比TreeSet好，特别是在添加和查询元素时，而这两个操作也是最重要的操作。TreeSet存在的唯一原因是它可以维持元素的排序状态，所以当需要一个排好序的Set时，才应该使用TreeSet。

#### 17.10.5 对Map的选择

调整HashMap以提高性能：

- 容量：表中的桶位数。
- 初始容量：表在创建时所拥有的桶位数。
- 尺寸：表中当前存储的项数。
- 负载因子：尺寸/容量。（如0.75 是达到3/4再散列）

## 17.11 实用方法 

介绍 Collections 详细方法。

#### 17.11.1 List的排序和查询

List的排序和查询所使用的方法与对象数组所使用的相应方法有相同的名字和语法，只是用Collections的static方法替换为Arrays的方法而已。

#### 17.11.2 设定Collection或Map为不可修改

Collections.unmodifiableCollection() 有时候需要对它进行保护，避免返回结果被人修改。

Collections.unmodifiableCollection这个可以得到一个集合的镜像，它的返回结果不可直接被改变，否则会提示 :
Exception in thread "main" java.lang.UnsupportedOperationException

```
public class CollectionsTest {  
      
    @Test  
    public void test(){  
        Collection<String> c = new ArrayList<String>();  
          
        Collection<String> s = Collections.unmodifiableCollection(c);  
          
        c.add("str");  
          
        System.out.println(s);  
    }  
  
}  
```

#### 17.11.3 Collection或Map同步控制

Collections.synchronizedCollection() 采用装饰者模式实现 线程安全几个操作方法。

快速报错

获取遍历对象前添加元素 会抛出异常 ConcurrentModificationException，防止多个进程同时修改同一个容器的内容。


```
public static void main(String[] args) {

        try {
            // creating object of List<String>
            List<String> vector = new ArrayList<String>();

            Iterator<String> iterator = vector.iterator();

            vector.add("A");

            String next = iterator.next();

        }

        catch (ConcurrentModificationException e) {
            System.out.println("Exception thrown : " + e);
        }
    }
```
ConcurrentHashMap  CopyOnWriteArrayList  CopyOnWriteArraySet 都是使用了避免 ConcurrentModificationException 的错误。

## 17.12 持有引用

StrongReference SoftReference  WeakReference PhantomReference （后三个使用暂时有些迷糊 继续往后看）
WeakHashMap
这是一种节约存储空间的技术，因为WeakHashMap允许垃圾回收器自动清理键和值，所以它显得十分便利。

## 17.13 Java 1.0/1.1 的容器

1. Vector和Enumeration
    Vector基本可看做ArrayList。
    Enumeration（枚举）只是接口而不是实现，应尽量使用iterator。

2. Hashtable
3. Stack
    继承Vector，而不是用Vector创建Stack。

4. BitSet
    高效存储大量“开/关”信息。其效率仅对空间而言，如果需要高效的访问时间，BitSet比本地数组稍慢一点。
BitSet的最小容量是long：64位。
如果拥有一个可以命名的固定的标志集合，那么EnumSet与BitSet相比，通常是一个更好的选择。

## 17.14 总结
java容器对于面向对象来说是最重要的类库。最重要的hashcode 和如何编写 hashcode.
