大多在JDK集合框架上有经验的程序员都非常喜欢`java.util.Collections`工具类。Guava提供了更多的工具类：兼容所有集合的静态方法。这些是Guava的最流程和成熟的部分。

特定接口的方法以一种直观的方式被分组在一起：
接口 | JDK或Guava | 相关的Guava工具类
--- | --- | ---
Collection | JDK | Collections2(避免与java.util.Collection冲突)
List | JDK | Lists
Set | JDK | Sets
SortedSet | JDK | Sets
Map | JDK | Maps
SortedMap | JDK | Maps
Queue | JDK | Queues
MultiSet | Guava | Multisets
Multimap | Guava | Multimaps
BiMap | Guava | Maps
Table | Guava | Tables

寻找`transform`, `filter`或类似的方法？参见函数式编程。

# 静态构造器
JDK7之前，构造一般的集合需要重复的代码：
```java
List<TypeThatsTooLongForItsOwnGood> list = new ArrayList<TypeThatsTooLongForItsOwnGood>();
```
想必大家都会觉得这样不太友好，Guava提供了使用泛型在右侧进行类型推断的静态方法：
```java
List<TypeThatsTooLongForItsOwnGood> list = Lists.newArrayList();
```
JDK7也提供了解决这个争议的方法：
```java
List<TypeThatsTooLongForItsOwnGood> list = new ArrayList<>();
```
但Guava做的不止这些，使用工厂方法设计模式，可以很方便的使用元素初始化集合。
```java
Set<Type> copySet = Sets.newHashSet(elements);
List<String> theseElements = Lists.newArrayList("alpha", "beta", "gamma");
```
另外，由于可以给工厂方法命名，我们可以根据初始化集合大小来改进可读性：
```java
List<Type> exactly100 = Lists.newArrayListWithCapacity(100);
List<Type> approx100 = Lists.newArrayListWithExpectedSize(100);
Set<Type> approx100Set = Sets.newHashSetWithExpectedSize(100);
```
上面的表格根据相关的工具类列出了标准的静态工厂方法。

注意：新的集合类型一节没有暴露任何构造方法或在工具类中初始化的方法，它们直接暴露了静态方法，例如：
```java
Multiset<String> multiset = HashMultiset.create();
```

# `Iterables`（迭代器）
Guava在尽可能的情况下更偏向于提供访问`Iterable`迭代器的工具类而不是`Collection`。在Google，我们经常遇到这样的情况：所谓的集合并不是存在于内存之中，可能是从数据库或其他数据中心收集的数据，这些数据在没有抓取所有数据之前并不支持类似`size()`的操作。

因此，在集合中有的方法在`Iterables`中基本都有，另外，大部分`Iterables`的方法在`Iterators`中有一个对应的版本。

绝大多数`Iterables`中的方法是惰性的：它们尽在绝对需要的时候才去迭代。返回`Iterables`的方法本身返回的是惰性计算的视图，而不是在内存中明确的构造一个集合。

Guava12之后，`Iterables`是`FluentIterable`类的补充，它包装了一个`Iterable`并且为这些操作提供了流式的语法。

以下列出了常用的工具类，其中`Iterables`的很多方法都是函数式的方法，后续会讨论函数式编程。

方法 | 描述 | 参见
--- | --- | ---
concat(Iterable<Iterable>) | 连结几个迭代器并返回一个惰性视图 | concat(Iterable...)
frequency(Iterable, Object) | 返回Object的出现次数 | 对比Collections.frequency(Collection, Object);参见Multiset
partition(Iterable, int) | 按照指定大小将该迭代器分为几块，并返回其不可变的视图 | Lists.partition(List, int) paddedPartition(Iterable, int)
getFirst(Iterable, T default) | 返回该迭代器的第一个元素，如果迭代器为空，则返回默认元素 | 对比Iterable.iterator().next() FluentIterable.first()
getLast(Iterable) | 返回该迭代器的最后一个元素，如果迭代器为空，会立即抛出NoSuchElementException | getLast(Iterable, T default) FluentIterable.last()
elementsEqual(Iterable, Iterable) | 如果两个迭代器元素相同并且顺序一致，返回true | 对比List.equals(Object)
unmodifiableIterable(Iterable) | 返回该迭代器的不可变视图 | 对比Collections.unmodifiableCollection(Collection)
limit(Iterable, int) | 返回该迭代器最多包含指定数量的Iterable | FluentIterable.limit(int)
getOnlyElement() | 返回该迭代器的仅有的元素，如果迭代器为空或包含多个元素则会失败 | getOnlyElement(Iterable, T default)

```java
Iterable<Integer> concatenated = Iterables.concat(Ints.asList(1,2,3), Ints.asList(4,5,6));
//concatenated现在是{1,2,3,4,5,6}

String lastAdded = Iterables.getLast(myLinkedHashSet);

String theElement = Iterables.getOnlyElement(thisSetIsDefinitelyASingleton);
//如果该Set不是单例的就会出错
```

## 类似集合的

## `FluentIterable`

# `Lists`

## 静态工厂

# `Sets`
