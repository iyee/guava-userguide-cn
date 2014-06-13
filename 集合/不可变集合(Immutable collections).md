# 示例
```java
public static final ImmutableSet<String> COLOR_NAMES = ImmutableSet.of(
"red",
"orange",
"yellow",
"green",
"blue",
"purple");

class Foo {
	Set<Bar> bars;
	Foo(Set<Bar> bars) {
		this.bars = ImmutableSet.copyOf(bars);
	}
}
```

# Why
不可变对象有很多好处，包括：

- 可以安全的被不信任的库使用
- 线程安全：可用在多线程中，且不存在资源竞争
- 不可变性，可以节省时间和空间。所有不可变集合的实现都比其可变的实现占用较少的资源，效率也较较高（[分析](http://code.google.com/p/memory-measurer/wiki/ElementCostInDataStructures)）
- 可以当作常量使用

使用对象的不可变拷贝是一个好的防御式编程技术，Guava为每一个标准的集合类型都提供了简单，方便使用的不可变版本，包括Guava自己的集合类。

JDK提供了`Collections.unmodifiableXxx`方法，但是在我们看来，它有以下缺点：

- 笨重并且冗余，在任何想使用防御式拷贝的地方使用非常不友好
- 不安全，返回的集合仅在没有任何地方持有其引用的时候才真正的不可变
- 效率低下，这种数据结构所需的开销仍与可变集合相同，包括并发写检查，哈希表所占用的额外空间等

**当不需要改变集合或希望将其作为常量的时候，使用防御式拷贝将其拷贝到不可变的集合中是非常好的实践。**

**注意：**
Guava的所有不可变集合的实现都不允许null值。我们在Google内部基础代码上作了详细的测试，报告显示仅有5%的时候是允许null元素的存在，其余95%的时候都是在遇到null的时候越快抛出异常越好。如果你需要null值，考虑使用`Collections.unmodifiableList`等允许null的方法的实现。更详细的信息参见[这里](https://code.google.com/p/guava-libraries/wiki/UsingAndAvoidingNullExplained)。

# How
有以下几种方式创建一个`ImmutableXxx`集合：

- 使用`copyOf`方法，例如`ImmutableSet.copyOf(set)`
- 使用`of`方法，例如`ImmutableSet.of("a", "b", "c")`或`ImmutableMap.of("a", 1, "b", 2)`
- 使用`Builder`，例如：

```java
public static final ImmutableSet<Color> GOOGLE_COLORS = 
	ImmutableSet.<Color>builder()
	.addAll(WEBSAFE_COLORS)
	.add(new Color(0, 191, 255))
	.build();
```
除了有序集合，顺序在构造的时候就已经处理过了，例如：
`ImmutableSet.of("a", "b", "c", "a", "d", "b")`
在迭代的时候的顺序为a->b->c->d。

## `copyOf`比想象中的要智能
`ImmuableXxx.copyOf()`仅在安全的时候才会拷贝数据 - 确切的信息未指定，但实现是非常智能的，例如：

```java
ImmutableSet<String> foobar = ImmutableSet.of("foo", "bar", "baz");
thingamajig(foobar);

void thingamajig(Collection<String> collection) {
	ImmutableList<String> defensiveCopy = ImmutableList.copyOf(collection);
	...
}
```
上面的代码，`ImmutableList.copyOf(foobar)`会智能的返回`foobar.asList()`，它是`ImmutableSet`的一个时间常数的视图。

`ImmutableXxx.copyOf(ImmutableCollection)`会在以下几种情况下避免线性拷贝：

- 在时间常数之内相关的数据结构可用。例如`ImmutableSet.copyOf(ImmutableList)`在时间常数内就无法完成。
- 不会造成内存泄漏，例如，有一个`ImmutableList<String>`的hugeList，然后调用`ImmutableList.copyOf(hugeList.subList(0, 10))`，拷贝会立即执行，是为了避免持有不需要的hugeList的引用。
- 不会改变语义，`ImmutableSet.copyOf(myImmutableSortedSet)`会执行一个拷贝，因为`ImmutableSet`使用的`hashCode()`和`equals()`方法与基于`Comparator`行为的`ImmutableSortedSet`具有不同的语义。

这些都是为了在防御式编程风格中保持最小的性能开销。

## `asList`
所有的不可变集合都通过`asList`方法提供了一个`ImmutableList`视图，所以如果你有数据存在`ImmutableSortedSet`中，依然可以通过`sortedSet.asList().get(k)`来获取第k小的元素。

返回的`ImmutableList`通常（但不总是）是开销固定的视图而不是一个明确的拷贝。意思就是说它通常比一般的`List`要智能，例如，它使用了后向集合的更高效的`contains()`方法。

# 更多
## Where
Interface | JDK or Guava? | Immutable Version
--- | --- | ---
Collection | JDK |ImmutableCollection
List | JDK | ImmutableList
Set | JDK | ImmutableSet
SortedSet/NavigableSet | JDK | ImmutableSortedSet
Map | JDK | ImmutableMap
SortedMap | JDK | ImmutableSortedMap
Multiset | Guava | ImmutableMultiset
SortedMultiset | Guava | ImmutableSortedMultiset
Multimap | Guava | ImmutableMultimap
ListMultimap | Guava | ImmutableListMultimap
SetMultimap | Guava | ImmutableSetMultimap
BiMap | Guava | ImmutableBiMap
ClassToInstanceMap | Guava | ImmutableClassToInstanceMap
Table | Guava | ImmutableTable
