Guava引入了一些非常有用但JDK中没有的集合类型。这些新的集合类型并不是硬塞进JDK的集合框架中的，并且与其配合的非常好。

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
Guava有多个`Multiset`实现，大概与JDK的Map实现差不多：

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
