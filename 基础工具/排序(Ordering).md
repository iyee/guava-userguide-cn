# Ordering
## 示例
```java
assertTrue(byLengthOrdering.reverse().isOrdered(list));
```
## 概述
Ordering是Guava的流式的Comparator类，可以构建复杂的Comparator并将其应用到集合类上。

其核心仅仅是一个特殊的Comparator实例。Ordering简单的把依赖于Comparator的方法（例如`Collections.max()`）变成可见的实例方法。另外，Ordering类提供了链式方法来优化和增强现有的Comparators。
## 创建
常用的排序是使用静态方法提供的：

方法 | 描述
----- | -----
`natural()` | 使用自然排序
`usingToString()` | 使用基于String的排序（`toString()`的返回值）

将已有的Comparator转换成Ordering，使用`Ordering.from(Comparator)`。

创建一个自定义的Ordering更通用的做法是直接集成抽象的Ordering类并覆写其抽象方法：
```java
Ordering<String> byLengthOrdering = new Ordering<String>() {
	public int compare(String left, String right) {
		return Ints.compare(left.length(), right.length());
	}
};
```
## 链式
给定一个Ordering，可以将其包装成一个衍生的Ordering。必将常用的变体如下：

方法 | 描述
----- | -----
`reverse()` | 返回一个反转的Ordering
`nullsFirst()` | 返回一个Ordering，排序规则为null在前，非null在后，其余按正常排序。另请参见`nullsLast()`
`compound(Comparator)` | 返回一个使用打破常规的特殊Comparator
`lexicographical()` | 返回一个遍历元素并按字典排序的Ordering
`onResultOf(Function)` | 返回一个Ordering，排序规则为比较Function接口的值

例如有以下这个类，要为其添加一个Comparator：
```java
Class Foo {
	@Nullable
	String sortedBy;
	int notSortedBy;
}
```
`sortedBy`字段可以为null，以下是在链式方法基础上的一个解决方案：
```java
Ordering<Foo> ordering = Ordering.natural().nullsFirst().onResultOf(new Function() {
	@Override
	public String apply(Foo foo) {
		return foo.sortedBy;
	}
});
```

当调用Oedering的链式方法时，排序优先级是从右至左的（后向排序）。上面的例子会将Foo实例按以下步骤排序：

1. 查找sortedBy字段
2. 如果sortedBy字段的值为null，将其移至最前
3. 将剩下不为null的sortedBy字段按String自然排序

之所以会后向排序，是因为每一个链式调用都会把前一个Ordering包装成一个新的实例。

（后向排序例外：对于链式方法`compound()`，是从左至右的，为避免混乱，尽量避免该方法与其他链式方法的混合调用）。

如果链式调用过长的话会难以理解，建议最多使用3个链式调用（上面的例子），另外，把中间对象（例如Function）分离开来会使代码更加简洁：

```Ordering<Foo> ordering = Ordering.natural().nullsFirst().onResultOf(sortKeyFunction);```

## 应用
Guava提供了一系列使用Ordering来操作或检查值或集合的方法，比较常用的如下：

方法 | 描述 | See also
----- | ----- | -----
`greatestOf(Iterable iterable, int k)` | 根据Ordering从大到小排序，并返回指定迭代器中前k个最大的元素 | `leasetOf()`
`isOrdered(Iteratable)` | 根据Ordering检查指定的Iterable是否是非递减排序的 | `isStrictOrdered()`
`sortedCopy(Iterable)` | 把指定元素的已排序的副本作为List返回 | `immutableSortedCopy()`
`min(E, E)` | 根据Oedering返回两个参数中较小的一个，如果相等返回第一个 | `max(E, E)`
`min(E, E, E, E...)` |　根据Ordering返回参数中最小的一个，如果有多个，返回第一个 | `max(E, E, E, E...)`
`min(Iterable)` | 返回指定Iteratable中最小的元素，如果Iterable为空，抛出NoSuchElementException | `max(Iterable)` `min(Iterator)` `max(Iterator)`
