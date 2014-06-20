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

## 类似集合的方法
集合在其他集合的基础上天生支持这些操作，但在可迭代对象上并不是。

以下每一个方法当输入确实是`Collection`的时候都会用相关的`Collection`接口作为代理。例如，如果调用`Iterable.size()`并且传递的是`Collection`，那么它会调用`Collection.size()`来替代迭代这个`Iterable`。

方法 | 类似的Collection的方法 | 等价的FluentIterable方法
--- | --- | ---
addAll(Collection addTo, Iterable toAdd) | Collection.addAll(Collection)
contains(Iterable, Object) | Collection.contains(Object) | FluentIterable.contains(Object)
removeAll(Iterable removeFrom, Colletion toRemove) | Collection.removeAll(Collection)
retainAll(Iterable removeFrom, Collection toRetain) | Collection.retainAll(Collection)
size(Iterable) | Collection.size() | FluentIterable.size()
toArray(Iterable, Class) | Collection.toArray(T[]) | FluentIterable.toArray(Class)
isEmpty(Iterable) | Collection.isEmpty() | FluentIterable.isEmpty()
get(Iterable, int) | List.get(int) | FluentIterable.get(int)
toString(Iterable) | Collection.toString | FluentIterable.toString()

## `FluentIterable`
除了上面提到的方法和函数式的方法，`FluentIterable`还有一些方便拷贝到不可变集合的方法：

ImmutableList | -
--- | ---
ImmutableSet | toImuutableSet
ImmutableSortedSet | toImmutableSortedSet

# `Lists`
为了静态构造方法和函数式编程，`Lists`提供了一系列作用于`List`的有价值的工具方法。

方法 | 描述
--- | ---
partition(List, int) | 返回一个把指定List切分成指定大小的N份的视图
reverse(List) | 返回一个反正List的视图。如果List是不可变的，使用`ImmutableList.reverse()`代替。
```java
List<Integer> countUp = Ints.asList(1,2,3,4,5);
List<Integer> countdown = Lists.reverse(countUp, 2); // {5,4,3,2,1}
List<List<Integer>> parts = Lists.partition(countUp, 2); // {{1,2}, {3,4}, {5}}
```
## 静态工厂
`Lists`提供了以下静态工厂方法：

实现 | 工厂方法
--- | ---
ArrayList | basic, with elements, from iterable, with exactly capacity, with expected size, from Iterator
LinkedList | basic, from Iterable

# `Sets`
`Sets`工具类提供了一系列犀利的方法。

## `Set`理论的操作
这里大概的是说数学上的理论操作。

我们提供了许多标准的理论操作，在参数指定的集合上以视图的方式来实现的。返回的是`SetView`，可以有以下用处：

- 直接当成`Set`使用，因为它实现了`Set`接口
- 使用`copyInto(Set)`将其拷贝到其他可变的集合中
- 使用`immutableCopy()`拷贝一份不可变集合

方法 |
--- |
union(Set, Set) |
intersection(Set, Set) |
difference(Set, Set) |
symmetricDifference(Set, Set) |

示例：
```java
Set<String> wordsWithPrimeLength = ImmutableSet.of("one", "two", "three", "six", "seven", "eight");
Set<String> primes = ImmutableSet.of("two", "three", "five", "seven");

SetView<String> intersection = Sets.intersection(primes, worksWithPrimeLength); //包含"two", "three", "seven"
return intsersection.immutableCopy();
```

## 其他`Set`工具
方法 | 描述 | 参见
--- | --- | ---
cartesianProduct<List<Set>> | 返回指定List的笛卡尔积 | cartesianProduct(Set...)
powerSet(Set) | 返回指定集合的所有子集（包括空集）

```java
Set<String> animals = ImmutableSet.of("gerbil", "hamster");
Set<String> fruits = ImmutableSet.of("apple", "orange", "banana");

Set<List<String>> product = Sets.cartesianProduct(animal, fruits);
//{{"gerbil","apple"},{"gerbil","orange"},{"gerbil", "banana"}, {"hamster","apple"}, {"hamster","orange"},{"hamster","banana"}}

Set<Set<String>> animalSets = Set.powerSet(animals);
//{{}, {"gerbil"}, {"haster"}, {"gerbil", "hamster"}}
```

## 静态工厂
`Sets`提供了下列静态工厂方法：

实现 | 工厂
--- | ---
HashSet | basic, with elements, from Iterable, with expected size, from Iterator
LinkedHashSet | basic, from Itrable, with expected size
TreeSet | basic, with Comparator, from Iterable

# `Maps`
`Maps`有一系列很酷的工具，我们单独拿出来说明一下。

## `uniqueIndex`
`Maps.uniqueIndex(Iterable, Function)`，很多对象具有唯一的属性，并且根据这些这些属性能够查找到这些对象，该方法常用于这种情况，定位到指定对象。

假设有一个包含很多字符串的strings，并且它们的长度是唯一的，然后根据长度查找字符串：
```java
ImmutableMap<Integer, String> stringByIndex = Maps.uniqueIndex(strings, new Function<String, Integer>() {
	public Integer apply(String s) {
		return s.length();
	}
});
```
如果索引不是唯一的，参见下面的`Multimaps.index`。

## `difference`
`Maps.difference(Map, Map)`用来比较两个`Map`差异。返回的是`MapDifference`对象，它把维恩图（用圆表示集合与集合的关系）分解了：

方法 | 描述
--- | ---
entriesInCommon() | 两个Map中相同的元素（key和value都相同）
entriesDiffering() | 两个Map中key相同，value不同的元素，这个map中的values是`MapDifference.ValueDifference`类型，可以进行value的左右对比
entriesOnlyOnLeft() | 返回key仅在左边Map的元素
entriesOnlyOnRight() | 返回key仅在右边Map的元素

```java
Map<String, Integer> left = ImmutableMap.of("a", 1, "b", 2, "c", 3);
Map<String, Integer> right = ImmutableMap.of("b", 2, "c", 4, "d", 5);
MapDifference<String, Integer> diff = Maps.difference(left, right);

diff.entriesInCommon(); //{"b"->2}
diff.entriesDiffering(); //{"c"->(3,4)}
diff.entriesOnlyOnLeft(); //{"a"->1}
diff.entriesOnlyOnRight(); //{"d"->5}
```

## `BiMap`工具
`BiMap`的工具在`Maps`类中，因为`BiMap`也是一个`Map`。

`BiMap`工具 | 相关`Map`工具
--- | ---
synchronizedBiMap(BiMap) | Collections.synchronizedMap(Map)
unmodifiableBiMap(BiMap) | Collections.unmodifiableMap(Map)

## 静态工厂
`Maps`提供了下列静态工厂方法。

实现 | 工厂
--- | ---
HashMap | basic, from map, with expected size
LinkedHashMap | basic, from map
TreeMap | basic, from Comparator, from SortedMap
EnumMap | from Class, from Map
ConcurrentMap | basic
IdentityHashMap | basic

# `Multisets`
标准的`Collection`操作，例如`containsAll`，忽略了多重集合的元素，并且只关心元素是否全部在或全部不在多重集合内。`Multisets`提供一系列考虑了多重集合多种元素的方法。

方法 | 说明 | 与Collection的方法不同之处
--- | --- | ---
containsOccurrences(Multiset sup, Multiset sub) | 对于所有o如果`sub.count(o) <= super.count(o)`则返回true | `Collection.containsAll`忽略计数，仅测试是否包含元素
removeOccurrences(Multiset removeFrom, Multiset toRemove) | 从removeFrom中删除所有在toRemove中的元素 | TODO
retainOccurrences(Multiset removeFrom, Multiset toRetain) | 对于所有o确保removeFrom.count(o)<=toRetain.count(o) | TODO
intersection(Multiset, Multiset) | 返回两个集合的交集的视图 | 没有类似操作

```java
Multiset<String> multiset1 = HashMultiset.create();
multiset1.add("a", 2);
Multiset<String> multiset2 = HashMultiset.create();
multiset2.add("a", 5);

multiset1.containsAll(multiset2);
//true，所有唯一的元素都包含在内
//尽管multiset1.count("a") == 2 < multiset2.count("a") == 5
Multisets.containsOccurrences(multiset1, multiset2); //false

multiset2.removeOccurences(multiset1);
//multiset2目前包含{"a", 3}

multiset2.removeAll(multiset1);
//删除所有multiset2中的"a"相关的元素

multiset2.isEmpty(); //true
```

其他一些工具方法如下：

方法 | 描述
--- | ---
copyHighestCountFirst(Multiset) | 返回该集合从大到小排序的不可变集合的拷贝
unmodifiableMultiset(Multiset) | 返回该集合不可变集合的视图
unmodifiableSortedSet(SortedMultiSet) | 返回该有序集合的不可变的视图
```java
Multiset<String> multiset = HashMultiset.create();
multiset.add("a", 3);
multiset.add("b", 5);
multiset.add("c", 1);

ImmutableMultiset<String> highestCountFirst = Multisets.copyHighestCountFirst(multiset);
//迭代的时候会按序迭代{"b", "a", "c"}
```

# `Multimaps`
## `index`
类似`Maps.uniqueIndex()`，`Multimaps.index(Iterable, Function)`解决了根据非唯一性的属性查找对象的情况，属性不需要唯一。

例如给strings基于长度进行分组：

```java
ImmutableSet<String> digits = ImmutableSet.of("zero", "one", "two", "three", "four","five", "six", "seven", "eight", "nine");
Function<String, Integer> lengthFunction = new Function<String, Integer>() {
	public Integer apply(String s) {
		return s.length();
	}
}
ImmutableListMultimap<Integer, String> digisByLength = Multimaps.index(digits, lengthFunction);
/*
 * 3 -> {"one", "two", "six"}
 * 4 -> {"zero", "four", "five", "nine"}
 * 5 -> {"three", "seven", "eight"}
 */
```
## `invertFrom`
因为`Multimap`的映射可以是一对多，也可以是多对一，所以反转这个`Multimap`也是有用的。Guava提供了`invertFrom(Multimap toInvert, Multimap dest)`方法来执行这个操作，不需要自己选择实现。

_注意：_如果使用的是`ImmutableMultimap`，使用`ImmutableMultimap.inverse()`。

```java
ArrayListMultimap<String, Integer> multimap = ArrayListMultimap.create();
multimap.putAll("b", Ints.asList(2, 4, 6));
multimap.putAll("a", Ints.asList(4, 2, 1));
multimap.putAll("c", Ints.asList(2, 5, 3));

TreeMultimap<Integer, String> inverse = Multimaps.invertFrom(multimap, TreeMultimap.<String, Integer> create());
//这里我们选择了具体的实现类，所以这里得到的是有序的结果
/*
 * 反转的map：
 * 1 => {"a"}
 * 2 => {"a", "b", "c"}
 * 3 => {"c"}
 * 4 => {"a", "b"}
 * 5 => {"c"}
 * 6 => {"b"}
 */
```

## `forMap`
需要在`Map`上使用`Multimap`的方法？`forMap(Map)`将`Map`作为`SetMultimap`视图查看。这种做法是很有用的，例如和`Multimap.invertFrom()`联合使用。

```java
Map<String, Integer> map = Immutable.of(("a", 1, "b", 1, "c", 2);
SetMultimap<String, Integer> multimap = Multimaps.forMap(map);
//["a"->{1}, "b"->{1}, "c"->{2}]
Multimap<Integer, String> inverse = Multimaps.invertFrom(multimap, HashMultimap.<Integer, String> create());
//反转的map：[1->{"a","b"}, 2->{"c"}]
```

## 包装器
`Multimaps`提供了传统的包装方法，包括基于`Map`和`Collection`实现的自定义`Multimap`实现的工具。

- | - | - | - | - |
--- | --- | --- | --- | ---
Unmodifiable | Mulimap | ListMultimap | SetMultimap | SortedMultimap
Synchronized | Mulimap | ListMultimap |  SetMultimap | SortedMultimap
Custom Implemention | Mulimap | ListMultimap |  SetMultimap | SortedMultimap

自定义的`Multimap`实现可以指定返回的`Multimap`的实现，如下：

- multimap拥有工厂返回的map和lists的完全所有权，这些对象不应该手动更新，首次提供的时候是空的，并且它们不能是软引用，弱引用和虚引用
- 若修改了`Multimap`，无法保证map的内容是什么样的。
- 多线程操作multimap不是线程安全的，即使是工厂生成的map和实例。多线程读操作是正确工作的。如果需要考虑Synchronized包装器。
- multimap是可序列化的，如果序列化，其中的map，list等内容都会序列化。
- `Multimap.get(key)`返回的集合与`Supplier`返回的集合不是同一类型，即使提供者返回了`RandonAccess`的lists，`Multimap.get(key)`返回的lists也是随机访问的。

注意自定义的`Multimap`需要一个`Supplier`参数来生成新的集合，以下是一个示例，使用`Treemap`映射到`LinkedList`来实现的`ListMultimap`.

```java
ListMultimap<String, Integer> myMultimap = Multimaps.newListMultimap(
	Maps.<String, Collection<Initeger>>newTreeMap(), new Supplier<LinkedList<Integer>>() {
	public LinkedList<Integer> get() {
		return Lists.newLinkedList();
	}
}
);
```

# `Tables`
## `customTable`
与`Multimaps.newXxxMultimap(Map, Supplier)`相比，`Tables.newCustomTable(Map, Supplier<Map>)`允许指定一个行或列映射的`Table`实现。

```java
//使用LinkedHashMaps代替HashMaps
Table<String, Character, Integer> table = Tables.newCustonTable(
Maps.<String, Map<Character, Integer>>newLinkedHashMap(), new Supplier<Map<Character, Integer>>() {
	public Map<Character, Integer> get() {
		return Maps.newLinkedHashMap();
	}
}
);
```

## `transpose`
`transpose(Table<R, C, V>)`允许把`Table<R, C, V>`作为`Table<C, R, V>`视图查看。例如，if you're using a Table to model a weighted digraph, this will let you view the graph with all the edges reversed.

## 包装器
不可变的包装器，大多数情况下请使用`ImmutableTable`。
- | - | -
--- | --- | ---
Unmodifiable | Table | RowSortedTable
