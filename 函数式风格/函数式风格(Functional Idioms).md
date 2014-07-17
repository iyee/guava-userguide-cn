# 说明
直到Java7，Java中的函数式编程只能是使用笨拙的匿名内部类。这个将在Java8中改变，但Guava却能让Java5以上的用户实现函数式风格的编程。

过多的使用Guava的函数式编程会导致代码冗长、混乱，可读性差和低效。这是目前最容易也是最常滥用的Guava的部分，and when you go to preposterous lengths to make your code "a one-liner," the Guava team weeps.

```java
Function<String, Integer> lengthFunction = new Function<String, Integer>() {
	public Integer apply(String str) {
		return str.length();
	}
}

Predicate<String> allCaps = new Predicate<String>() {
	public boolean apply(String str) {
		return CharMatcher.JAVA_UPPER_CASE.matchesAllOf(str);
	}
}
```

`FluentIterable`版本：

```java
Multiset<Integer> lengths = HashMultiset.create(
	FluentIterable.from(strings).filter(new Predicate<String>() {
		public boolean apply(String str) {
			return CharMatcher.JAVA_UPPER_CASE.matchesAllOf(str);
		}
	}).transform(new Function<String, Integer>() {
		public Integer apply(String str) {
			return str.length();
		}
	})
);
```

将以上的两份代码与下面的比较：

```java
Multiset<Integer> lengths = HashMultiset.create();
for (String s : strings) {
	if (CharMatcher.JAVA_UPPER_CASE.matchesAllOf(s)) {
		lengths.add(s.length());
	}
}
```

即使使用静态导入，即使`Function`和`Predicate`声明在其他源文件中，第一种实现方式都是更复杂，冗长，可读性差，效率低下的。

在Java7中，我们应该默认使用更加紧凑的代码。在确定明确以下情况之前，尽量少使用函数式风格：

- 使用函数式风格会大幅减少整个项目的代码量。像上面的例子，函数式风格使用了11行，而紧凑的代码使用了6行。将`Function`的定义移到其他文件中无补于事。
- 为了更高的效率，需要待转换集合的惰性计算视图，并且不能使用明确计算的集合来解决问题。另外，你需要阅读或重读《Effective Java》的55页，遵循上面的说明，做了性能基准测试来证明函数式风格效率更高，并且提供举证数据。

务必确定在使用函数式风格的时候，传统紧凑的方式具有更差的可读性。尝试那种方式是否更加糟糕？是否比你正要使用的别扭的函数式方式具有更高的可读性？

# `Function`和`Predicate`
本文仅讨论直接与`Function`和`Predicate`有关的功能。其他有关函数式风格的工具像在常数时间内返回视图的方法请参见[集合工具类](/集合/工具类(Utility classes).md)。

Guava提供了两个基础的“函数式”接口：

- `Function<A, B>`，只有一个方法`B apply(A)`。对于每一个`a.equals(b)`，都一定会有`function.apply(a).equals(function.apply(b))`。
- `Predicate<T>`，只有一个方法`boolean apply(T)`，任意时间对于相同的输入都一定会产生相同的输出。

### 特殊的`Predicate`
对于字符，有特定的`Predicate` — `CharMatcher`，具有更高的效率并且更加有用。`CharMatcher`实现了`Predicate<Character>`，使用`CharMatcher.forPredicate()`可以把一个`Predicate`转成`CharMatcher`。详细信息请参考[`CharMatcher`](字符串/字符串(Strings).md)一节。

另外，对基于比较的`Predicate`，大多数需求都可以通过`Range`类型来解决。`Range`实现了`Predicate`，测试其中的内容。例如：`Range.atMost(2)`就是一个`Predicate<Integer>`，更多详情请参考[相关内容](/区间-范围/区间-范围(Ranges).md)。

### `Function`和`Predicate`的操作
简单的`Function`构造和操作在`Functions`类中，包括：

1 | 2 | 3 | 4 | 5
--- | --- | --- | --- | ---
`forMap(Map<A, B>)` | `compose(Function<B, C>, Function<A, B>)` | `constant(T)` | `identity()` | `toStringFunction()`

详情参加Javadoc。

`Predicates`中有很多构造和操作的方法，以下是一部分：

1 | 2 | 3 | 4
--- | --- | --- | ---
`instanceOf(Class)` | `assignableFrom(Class)` | `constains(Pattern)` | `in(Collection)`
`isNull()` | `alwaysFalse()` | `alwaysTrue` | `equalTo(Object)`
`compose(Predicate, Function)` | `and(Predicate...)` | `or(Predicate...)` | `not(Predicate)`

详情参加Javadoc。

## 使用
Guava提供了很多使用`Function`和`Predicate`操作集合的工具，这些可以在集合工具类`Iterables`，`lists`，`Sets`，`Maps`，`Multimaps`等类中找到。

### `Predicates`
Predicates最常用的功能是过滤集合。所有Guava的过滤方法都会返回视图。

集合类型 | 过滤方法 | - | -
--- | --- | --- | ---
Iterable | Iterables.filter(Iterable, Predicate) | FluentIterable.filter(Predicate)
Iterator | Iterators.filter(Iterator, Predicate)
Collection | Collections2.filter(Collection, Predicate)
Set | Sets.filter(Set, Predicate)
SortedSet | Sets.filter(SortedSet, Predicate)
Map | Maps.filterKeys(Map, Predicate) | Maps.filterValues(Map, Predicate) | Maps.filterEntries(Map, Predicate)
SortedMap | Maps.filterKeys(SortedMap, Predicate) | Maps.filterValues(SortedMap, Predicate) | Maps.filterEntries(SortedMap, Predicate)
MultiMap | MultiMaps.filterKeys(MultiMap, Predicate) | MultiMaps.filterValues(MultiMap, Predicate) | MultiMaps.filterEntries(MultiMap, Predicate)

`List`过滤的视图是被忽略的，因为不能很好的支持像`get(int)`这样的操作。使用`Lists.newArrayList(Collections2.filter(list, predicate))`生成一个拷贝。

护理过滤，Guava还提供了很多工具使用Predicate来操作迭代器，尤其是`Iterables`和`FluentIterables`类。

Iterables方法签名 | 说明 | 另请参考
--- | --- | ---
boolean all(Iterable, Predicate) | 是否所有元素满足Predicate？惰性判断，如果有一个不满足，则不会再继续迭代 | Iterators.all(Iterator, Predicate) FluentIterable.allMatch(Predicate)
boolean any(Iterable, Predicate) | 迭代器中有元素满足Predicate吗？惰性判断，一直迭代直到找到一个满足的元素 | Iterators.any(Iterator, Predicate) FluentIterable.anyMatch(Predicate)
T find(Iterable, Predicate) | 查找满足条件的元素或者抛出NoSuchElementException | Iterators.find(Iterator, Predicate) Iterables.find(Iterable, Predicate, T default) Iterators.find(Iterator, Predicate, T default)
Optional<T> tryFind(Iterable,Predicate) | 返回找到的元素，或者Optional.absent() | Iterators.tryFind(Iterator, Predicate) FluentIterable.firstMatch(Predicate) OPtional
indexOf(Iterable, Predicate) | 返回找到的元素的索引，如果没找到则返回-1 | Iterators.indexOf(Iterator, Predicate) 
removeIf(Iterable, Predicate) | 返回所有满足条件的元素，使用Iterator.remove()方法 | Iterators.removeIf(Iterator, Predicate)

### `Functions`
目前函数式风格最常用的场景就是集合转换。Guava所有的转换方法都返回常规集合的视图。

集合类型 | 转换方法
--- | ---
Iterable | Iterables.transform(Iterable, Function) FluentIterable.transform(Function)
Iterator | Iterators.transform(Iterator)
Collection | Collection2.transform(Collection, Function)
List | Lists.transform(List, Function)
Maps* | Maps.transformValues(Map, Function) Maps.transformEntries(Map, EntryTransformer)
SortedMap* | Maps.transformValues(SortedMap, Function) Maps.transformEntries(SortedMap, EntryTransformer)
Multimap* | Multimaps.transformValues(Multimap, Function) Multimaps.transformEntries(Multimap, EntryTransformer)
ListMultimap* | Multimaps.transformValues(ListMultimap, Function) Multimaps.transformEntries(ListMultimap, EntryTransformer)
Table | Tables.transformValues(Table, Function)

*`Map`和`Multimap`有一个接受`EntryTransformer(K, V1, V2)`的特殊方法，它用key和之前的value计算出新的value，并将其与该key关联起来。

**`Set`的转换是被忽略的，因为它不能很好的支持`contains(Object)`方法。作为补充，使用`Sets.newHashSet(Collections2.transform(set, function))`创建一个已转换的Set的副本。

```java
List<String> names;
Map<String, Person> personWithName;
List<Person> people = Lists.transform(names, Functions.forMap(personWithName));
```

```java
//first name映射到同一个人的所有last name
ListMultimap<String, String> firstNameToLastNames;
ListMultimap<String, String> firstNameToName = Multimaps.transformEntries(firstNameToLastNames, new EntryTransformer<String, String, String>() {
	public String transformEntry(String firstName, String lastName) {
		return firstName + " " + lastName;
	}
});
```

一些Functions的可用场景如下：
类型 | 方法
--- | ---
Ordering | Ordering.onResultOf(Function)
Predicate | Predicates.conpose(Predicate, Function)
Equivalence | Equivalence.onResultOf(Function)
Supplier | Suppliers.compose(Function, Supplier)
Function | Functions.compose(Function, Function)

另外，`ListenableFuture`可以转换监听的事件(listenable futures)。`Futures`同样提供了一个接受`AsyncFunction`（`Function`的一个变体，可以异步的计算结果）的方法。

`Futures.transform(ListenableFuture, Function)`
`Futures.transform(ListenableFuture, AsyncFunction)`
`Futures.transform(ListenableFuture, Function, Executor)`
`Futures.transform(ListenableFuture, AsyncFunction, Executor)`
