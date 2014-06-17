Guava引入了一些非常有用但JDK中没有的集合类型。这些新的集合类型并不是硬塞进JDK的集合框架中的，它们配合的非常好。

# Multiset（多重集合）
统计一篇文档中某个单词出现了多少次，传统的Java做法是这样：

```java
Map<String, Integer> counts = new HashMap<String, Integer>();
for(String word : words) {
	Integer count = counts.get(word);
	if (count == null) {
		counts.put(word, 1);
	} else {
		counts.put(word, count + 1);
	}
}
```
这种做法非常笨拙，容易导致错误，并且无法收集一些有用的统计信息，例如所有单词的总数。我们有更好的办法。

Guava提供了一个新的集合类型，`Multiset`，支持添加多个元素。维基百科对multiset的数学定义如下：一般集合的概念是允许相同的元素出现多次......而multiset，类似于集合但与元组相反，是顺序无关的：{a, a, b}与{a, b, a}这两个multiset是相等的。

像下面这两种类型：
- 类似于无序的`ArrayList<E>`
- 类似于包含元素与计数的`Map<E, Integer>`

Guava的`Multiset`的API相当于把二者结合起来了：
- 当成一般的集合来看，`Multiset`看起来更像是无序的`ArrayList`。
	- 调用`add(E)`向集合中添加一个元素。
	-  `iterator()`方法迭代集合中的每个元素。
	- `size()`方法返回集合大小。
- 其他查询操作，包括特有的性能，类似于`Map<E, Integer>`。
	- `count(Object)`返回与指定Object相关联的计数，对于`HashMultiset`，该操作时间复杂度为O(1)，对于`TreeMultiset`，时间复杂度是O(log n)。
	- `entrySet()`方法返回类似于map的entrySet的`Set<Multiset.Entry<E>>`
	- `elementSet()`方法返回类似于map的keySet的无重复元素的`Set<E>`
	- `Multiset`实现的内存占用是无重复元素个数的线性表

另外，`Multiset`与`Collection`接口是完全一致的，除了极少数的情况（之前的JDK也有类似情况），例如，`TreeMultiset`，类似于`TreeSet`，使用比较器来实现相等的判断而不是使用`Object.equals()`。特别的，`Multiset.addAll(Collection)`会在`Collection`中每个元素每次出现都会添加一次，而不像上面那样需要手动遍历`Map`。

方法 | 描述
--- | ---
count(E) | 为添加到此Multiset中的元素计数
elementSet() | 将Multiset<E>作为Set<E>查看无重复的元素
entrySet() | 类似于Map.entrySet()，返回一个Set<Multiset.Entry<E>>，其中的entries有getElement()和getCount()方法
add(E) | 将元素添加到Multiset中
remove(E, int) | 删除指定计数的指定元素
setCount(E, int) | 将指定元素的计数设置为指定的非负数
size() | 返回Multiset的大小

## `MultiSet`不是`Map`
注意`Multiset<E>`不是`Map<E, Integer>`，尽管它是`Multiset`的一部分实现。`Multiset`是Collection类型，只是它兼容了`Map`的一些功能，以下是一些不同之处：

- `Multiset`元素的计数只能为正数，不可能负数，而且计数为0的元素也不存在`Multiset`中，它们不会出现在`entrySet()`或`elementSet()`的视图中。
- `Multiset.size()`方法返回的是集合的大小，表示的是所有集合元素的个数（包括重复元素），如果想要获取不重复元素的大小，使用`elementSet().size()`，因此，`add(E)`会让`Multiset.size()`增加1。
- `Multiset.iterator()`方法会迭代所有元素，迭代器的次数等于`Multiset.size()`。
- `multiset<E>`支持添加、删除元素，或直接给指定元素设置计数值，`setCount(E, 0)`相当于删除所有指定的元素。
-  对不存在于`Multiset`中元素调用`Multiset.count(elem)`始终返回0。

## 实现
Guava有多个`Multiset`实现，个数大概与JDK的Map实现的个数差不多：

Map | 对应的Multiset | 支持null
--- | --- | ---
HashMap | HashMultiset | Yes
TreeMap | TreeMultiset | Yes(如果Comparator支持)
LinkedHashMap | LinkedHashMultiset | Yes
ConcurrentHashMap | ConcurrentHashMultiset | No
ImmutableMap | ImmutableMultiset | No

## `SortedMultiset`
`SortedMultiset`是在`Multiset`接口基础上的一个变体，它支持获取指定范围的子`Multiset`。例如，想知道100ms内网站的访问率，就要知道100ms内访问量（`latencies.subMultiset(0, BoundType.CLOSED, 100, BoundType.OPEN).size()`）与总访问量（`latencies.size()`）之比。

`TreeMultiset`实现了`SortedMultiset`接口，在写本文档之时，`ImmutableSortedMultiset`正在进行GWT兼容性的测试。

# `MultiMap`（多重Map）
任何一个有经验的Java程序员，应该都实现过`Map<K, 	List<V>>`或`Map<K, Set<V>>`，然后处理这个笨拙的结构。例如，`Map<K, Set<V>>`表示一个典型的未标记的有向图。Guava的`MultiMap`框架让键到多值的映射更加简单。`MultiMap`是关联一个键到任意多个值的一种通用方法。

以下两种方式来理解`MultiMap`的概念：单个键值映射的集合：

a->1   a->2   a-4   b->3   c->5

或唯一的键到值集合的映射：

a->[1,2,4]   b->3   c->5

通常，`MultiMap`接口在第一种视图下是最好的想法，同时允许使用`asMap()`方法获取`Map<K, Collection<V>>`的视图。最重要的是，不存在一个key映射到一个空集合：一个key映射到至少一个值，否则它在`MultiMap`中就不存在。

很少会直接使用`MultiMap`接口，更经常的是使用`ListMultiMap`（映射key到List）或`SetMultimap`（映射key到Set）。

## 修改
`Multimap.get(key)`返回的是与此key关联的所有值的视图，即使它们当前不存在。对于`ListMultiMap`返回的是`List`，对于`SetMultiMap`，返回的是`Set`。

对相关的`MultiMap`修改操作，例如：

```java
Set<Person> aliceChildren = childrenMultimap.get(alice);
aliceChildren.clear();
aliceChildren.add(bob);
aliceChildren.add(carol);
```
其他的一些修改操作如下：

方法签名 | 描述 | 等价操作
--- | --- | ---
put(K, V) | 添加一个键值对 | multimap.get(K).add(V)
putAll(K, Iterable<V>) | 添加指定Iterable中的所有值并将其映射到K | Iterables.addAll(multimap.get(K), V)
remove(K, V) | 删除一个键值对映射，如果multimap改变了返回true | multimap.get(K).remove(V)
removeAll(K) | 删除并返回指定key映射的值的集合，返回的集合是可变或不可变的，但是修改这个集合不会影响到这个multimap | multimap.get(K).clear()
replaceValues(K, Iterable<V>) | 替换该key关联的值为指定的迭代器 |　multimap.get(K).clear(); Iterables.addAll(multimap.get(K), V);

## 视图
`Multimap`提供了几种强大的视图：

- `asMap`把`Multimap<K, V>`作为`Map<K, Collection<V>>`视图，返回的Map支持删除操作，并会将改变写回，但不支持`put()`或`putAll()`操作。另外，如果你需要不存在的key返回null而不是可写的空集合，使用`asMap().get(K)`（可以将其强转为需要的集合`SetMultimap`的`Set`或`ListMultimap`的`List`，但是类型系统不允许`ListMultimap<K, V>`返回`Map<K, List<V>>`）。
- `entries`返回`Multimap`中的`Collection<Map.Entry<K, V>>`视图。
- `keySet`返回`Multimap`中的不重复的key的Set集合。
- `keys`将`Multimap`中的key作为`MultiSet`视图返回，其中的元素可被删除，但不能添加，修改会被写回。
- `values`将`Multimap`中的值展平后返回`Collection<V>`，所有的值将会在一个集合中，类似于`Iterables.concat(multimap.asMap().values())`，但是返回的是一个完整的`Collection`。

## `Multimap`不是`Map`
`Multimap<K, V>`不是`Map<K, Collection<V>>`，尽管`Multimap`是使用这种方式实现的，以下是它们的差异：

- `Multimap.get(key)`返回的始终是非null但可能是空的集合，这并不是说multimap的key映射的值占用了内存，而是返回的集合视图允许你添加关联。
- 如果你更喜欢类似Map的操作（如果key不存在则返回null），使用`asMap()`视图获取`Map<K, Collection<V>>`（或从`ListMultimap`获取一个`Map<K, List<V>>`，使用静态的`Multimaps.asMap()`方法，`SetMultimap`和`SortedSetMultimap`也有类似的方法）。
- `Multimap.containsKey(key)`当且仅当存在元素与该key关联时才返回true。类似的，如果一个key之前关联了一个或多个值，但后来从multimap中删除了，该方法将返回false。
- `Multimap.entries()`返回`Multimap`中的所有key的映射，如果想要key的集合，使用`asMap().entrySet()`。
- `Multimap.size()`返回整个multimap的大小，并不是不重复key的个数，使用`Multimap.keySet().size()`来获取无重复key的个数。

## 实现
`Multimap`提供了多种实现，在大多数情况下可以用来替代`Map<K, Collection<V>>`。

实现 | key的行为类似... | value的行为类似...
--- | --- | ---
ArrayListMultimap | HashMap | ArrayList
HashMultimap | HashMap | HashSet
LinkedListMultimap[^1] | LinkedHashMap | LinkedList
LinkedHashMultimap[^2] | LinkedHashMap | LinkedHashSet
TreeMultimap | TreeMap | TreeSet
ImmutableListMultimap | ImmutableMap | ImmutableList
ImmutableSetMultimap | ImmutableMap | ImmutableSet

除了不可变的实现，其他都支持null key 和null value。

[^1] `LinkedListMultimap.entries()`为无重复的key value保留了迭代顺序。

[^2] `LinkedHashMultimap`同时保留了entry的插入顺序，key的插入顺序以及每个key的value的顺序。

注意，以上并不是每个都是用`Map<K, Collection<V>>`实现的（一部分`Multimap`为了最小化资源占用使用了自定义的哈希表）。

# `BiMap`
传统的键值双向映射是维护两个`Map`并保持它们的同步，这样很容易导致bug的出现，并且当value已经在map中的时候会让人感到困惑，例如：
```java
Map<String, Integer> nameToId = Maps.newHashMap();
Map<Integer, String> idToName = Maps.newHashMap();

nameToId.put("Bob", 42);
idToName.put(42, "Bob");
//如果Bob或42已经存在Map中该怎么办？
//如果忘记同步了会出现奇怪的bug...
```

`BiMap<K, V>`是一个`Map<K, V>`，它能够：

- 允许使用`inverse()`反转查看`BiMap<V, K>`
- 保证values的惟一性，将values作为一个Set集合

`BiMap.put(key, value)`会抛出`IllegalArgumentException`如果映射key到一个已存在的value。如果要删除与此value关联的key，使用`BiMap.forcePut(key, value)`。
```java
BiMap<String, Integer> userId = HashBiMap.create();	
...
String userForId = userId.inverse().get(id);
```

## 实现
key-value映射实现 | value-key 映射实现 | 相关BiMap
--- | --- | ---
HashMap | HashMap | HashBiMap
ImmutableMap | ImmutableMap | ImmutableBiMap
EnumMap | EnumMap | EnumBiMap
EnumMap | HashMap | EnumHashBiMap

注意：`BiMap`类似`SyncronizedBiMap`的工具在`Maps`类中。

# `Table`
```java
Table<Vertex, Vertex, Double> weightedGraph = HashBasedTable.create();
weightedGraph.put(v1, v2, 4);
weightedGraph.put(v1, v3, 20);
weightedGraph.put(v2, v3, 5);

weightedGraph.row(v1); //返回一个映射（v2->4, v3->20）
weightedGraph.column(v3); //返回一个映射(v1->20, v2->5)
```
当想要同时以多个key索引时，会使用到`Map<FirstName, Map<LastName, Person>>`这种丑陋的结构。Guava提供了一个新的集合类型 - `Table`，适用于这种基于“行列”的情景。它有一系列的视图可供使用：

- `rowMap()`，将`Table<R, C, V>`作为`Map<R, Map<C, V>>`视图查看，类似的，`rowKeySet()`返回一个`Set<R>`
- `row(r)`返回一个非null的`Map<C, V>`，对这个`Map`的操作将会影响到关联的`Table`
- 类似的基于列的方法：`columnMap()`, `columnKeySet()`, `column(c)`（基于列的访问效率要低于基于行的访问）
- `cellSet()`将`Table.Cell<R, C, V>`的集合作为`Table`的视图返回

以下是`Table`的实现：

- `HashBasedTable`，由`HashMap<R, HashMap<C, V>>`实现
- `TreeBasedTable`，由`TreeMap<R, TreeMap<C, V>>`实现
- `ImmutableTable`，由`ImmutableMap<R, ImmutableMap<C, V>>`（`ImmutableTable`是为稀疏和密集数据集优化的实现）
- `ArrayTable`，全部的行列需要在构造的时候指定，当该`Table`是密集型的时候为了内存和速度的效率，它由二维数组实现的。`ArrayTable`的实现与其他的实现有些不同，具体参见Javadoc。

# `ClassToInstanceMap`
有些时候，Map的key不全是一致的类型：它们是某一个特定的类型（types），如果希望映射到它们具体的类型，Guava为此提供了`ClassToInstanceMap`。

为了继承`Map`接口，为了避免不必要的类型转换和强制类型安全，`ClassToInstanceMap`提供了`T getInstance(Class<T>)`和`T putInstance(Class<T>, T)`方法。

`ClassToInstanceMap`只有一个类型参数，命名为`B`，表示该map管理的类型的基类，例如：
```java
ClassToInstanceMap<Number> numberDefaults = MutableClassToInstanceMap.create();
numberDefaults.putIntance(Integer.class, Integer.valueOf(0));
```

`ClassToInstanceMap<B>`实现了`Map<Class<? extends B>, B>`，或者这么说，它是B的子类的类型到B的实例的映射。这会让`ClassToInstanceMap`中的泛型更易于理解，只要记住一点：`B`是Map中的基类类型 - 通常，`B`就是一个`Object`。

Guava提供了两个有用的实现：`MutableClassToInstanceMap`和`ImmutableClassToINstanceMap`。

注意：类似其他的`Map<Class, Object>`，`ClassToInstanceMap`可能包含基本数据类型，而且基本数据类型和它的包装类型可能映射到不同的值。

# `RangeSet`
注：以下用范围集来表示数学上的集合的概念，以免与Java的集合混淆。

`RangeSet`描述了一组断开的、非空的范围集（并集）。当把一个范围添加到一个范围集的时候，所有可连接的范围集（交集）都会合并为一个范围，空集将被忽略。例如：
```java
RangeSet<Integer> rangeSet = TreeRangeSet.create();
rangeSet.add(Range.closed(1, 10)); //{[1, 10]}
rangeSet.add(Range.closedOpen(11, 15)); //断开的范围集，{[1, 10], [11, 15)}
rangeSet.add(Range.closedOpen(15, 20)); //可连接的范围（有交集），{[1, 10], [11, 20)}
rangeSet.add(Range.openClosed(0, 0)); //空集，{[1, 10], [11, 20)}
rangeSet。remove(Range.open(5, 10)); //[1, 10]区间被分隔了，{[1, 5], [10, 10], [11, 20]}
```
注意合并像`Range.closed(1, 10)`和`Range.closedOpen(11, 15)`的范围集，必须先使用`Range.canonical(DiscreteDomain)`预先处理范围，例如，`DiscreteDomain.withIntegers()`。

**注意：**`RangeSet`不支持GWT，也不支持JDK1.5，因为它需要JDK1.6中的`NavigableMap`的支持。

## 视图
//TODO 未完成

`RangeSet`实现支持很多范围的视图，包括：

- `complement()`：
- `sunRangeSet(Range<C>)`：返回该范围集和指定的范围的交集
- `asRanges()`：
- `asSet(DiscreteDomain<C>)`：

## 查询
为了在视图上操作，`RangeSet`直接提供了几个查询操作，常用的有：

- `contains(C)`：`RangeSet`最基础的操作，查询该`RangeSet`中的`Range`是否包含指定的元素。
- `rangeContaining(C)`：返回包含该元素的`Range`，如果不包含该元素，返回null。
- `encloses(Range<C>)`：测试`RangeSet`中的`Range`是否包含指定的`Range`。
- `span()`：//TODO

# `RangeMap`
`RangeMap`是一组范围集到值的映射，不像`RangeSet`，`RangeMap`从来不会合并相邻的映射，即使相邻的范围集映射到相同的值。例如：
```java
RangeMap<Integer, String> rangeMap = TreeRangeMap.create();
rangeMap.put(Range.closed(1, 10), "foo"); //{[1, 10] -> "foo"}
rangeMap.put(Range.open(3, 6), "bar"); //{[1, 3] - > "foo", (3, 6) -> "bar", [6, 10] -> "foo"}
rangeMap.put(Range.open(10, 20), "foo"); //{[1, 3] - > "foo", (3, 6) -> "bar", [6, 10] -> "foo", (10, 20) -> "foo"}
rangeMap.remove(Range.closed(5, 11)); //{[1, 3] - > "foo", (3, 5) -> "bar", [11, 20) -> "foo"}
```

## 视图
`RangeMap`提供了两种视图：

- `asMapOfRanges()`：将`RangeMap`作为`Map<Range<K>, V>`视图返回，支持迭代该`RangeMap`。
- `subRangeMap(Range<K>)`，返回该`RangeMap`与指定`Range`的交集视图，传统的`headMap`, `subMap`, `tailMap`操作就是建立在此基础上。
