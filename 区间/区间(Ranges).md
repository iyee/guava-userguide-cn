## 示例
```java
List<Double> scores;
Iterable<Double> belowMedianScores = Iterables.filter(scores, Range.lessThan(median));
Range<Integer> validGrades = Range.closed(1, 12);
for (int grade : ContiguousSet.create(validGrades, DiscreteDomain.integers()) {
	...
}
```

## 简介
范围，即区间（连续的，不间断的），是一个特定的区域。通常，对于任何a<=b<=c，若`range.contains(a) && range.contains(c)`，必有`range.contains(b)`。

范围可能是无限的，例如“x>3”包括任意大的数字；也可能是有限的，例如“2<=x<5”。我们使用更加简洁的记法来表示范围（数学意义上的集合）：

- (a..b) = {x | a < x < b}
- [a..b] = {x | a <= x <= b}
- [a..b) = {x | a <= x < b}
- (a..b] = {x | a < x <= b}
- (a..+∞) = {x | x > a}
- [a..+∞) = {x | x >= a}
- (-∞..b) = {x | x < b}
- (-∞..b] = {x | x <= b}
- (-∞..+∞) = all values

上述的a和b成为端点。为了保持一致性，Guava对于范围的记法要求上限端点值不能小于下限端点值，上下限端点仅在至少有一个闭区间的情况下可以相等：

- [a..a]：包含唯一元素的区间
- [a..a); (a..a]：空区间（即空集），有效
- (a..a)：无效

Guava的范围/区间的类型是`Range<C>`，所有的范围都是_不可变的_（_Immutable_）。

## 构建Range
可以从`Range`的静态方法构建范围：
区间 | 方法
--- | ---
(a..b) | open(C, C)
[a..b] | closed(C, C)
[a..b) | closedOpen(C, C)
(a..b] | openClosed(C, C)
(a..+∞) | greaterThan(C)
[a..+∞) | atLeast(C)
(-∞..b) | lessThan(C)
(-∞..b] | atMost(C)
(-∞..+∞) | all()

```java
Range.closed("left", "right") //所有在"left"和"right"之间的字符串
Range.lessThan(4.0) //小于4.0的double值
```

另外，`Range`也可以通过传入界限类型来构建：

类型 | 方法
--- | ---
两边界限 | range(C, BoundType, C, BoundType)
无上限（仅设置下限） | range.downTo(C, BoundType)
无下限（仅设置上限） | range.upTo(C, BoundType)

这里`BoundType`是枚举类型，包括`CLOSED`和`OPEN`类型。

```java
Range.downTo(4, boundType); //可以设置是否包含4
Range.range(1, CLOSED, 4, OPEN); //Range.closedOpen(1, 4)的另一种写法
```

## 操作
`Range`最基础的操作就是`contains(C)`，其行为正如名字一样。另外，`Range`可以被当作`Predicate`接口在函数式编程中使用。任何`Range`都支持`containsAll(Iterable<? extends C>)`。

```java
Range.closed(1, 3).contains(2); //true
Range.closed(1, 3).contains(4); //false
Range.lessThan(5).contains(5); //false
Range.closed(1, 4).containsAll(Ints.asList(1, 2, 3)); //true
```

### 查询操作
要查询区间的边界端点，有以下方法：

- `hasLowerBound()`, `hasUpperBound()`，用来检查一个区间是否有边界。
- `lowerBoundType()`, `upperBoundType()`，返回相关端点的边界类型（CLOSED或OPEN）。如果该区间没有边界，将会抛出`IllegalStateException`异常。
- `lowerEndPoint()`, `upperEndPoint`，返回指定边界的端点，，如果没有边界，抛出`IllegalStateException`异常。
- `isEmpty()`检查区间是否为空，类似于[a..a)或(a..a]的形式。

```java
Range.closedOpen(4, 4).isEmpty(); //true
Range.openClosed(4, 4).isEmpty(); //true
Range.closed(4, 4).isEmpty(); //false
Range.open(4, 4).isEmpty(); //Range.open throws IllegalArgumentException

Range.closed(3, 10).lowerEndPoint(); //3
Range.open(3, 10).lowerEndPoint(); //3
Range.closed(3, 10).lowerBoundType(); //CLOSED
Range.open(3, 10).upperBoundType(); //OPEN
```

### 区间操作
#### `encloses`
`encloses(Range)`是最基础的区间关联操作，如果内部区间被外部区间包含，则返回true。这个操作依赖于两边界线端点的比较！

- [3..6] encloses [4..5]
- (3..6) encloses (3..6)
- [3..6] encloses [4..4)（即使后者为空）
- (3..6] does not enclose [3..6]
- [4..5] does not enclose (3..6)（即使前者包含后者所有的元素）
- [3..6] does not enclose (1..1]（即使前者包含后者所有的元素）

`encloses`是[部分排序](https://code.google.com/p/guava-libraries/wiki/GuavaTermsExplained#partial_ordering)的。

因此，`Range`还提供了以下操作：

#### `isConnected`
`Range.isConnected(Range)`测试这些区间是否是连续的。特别地，`isConnected`也可以检查是否有区间被这些区间包含，等价于数学上对于并集的定义：这些集合的并集必须可以组成一个连续的集合。

`isConnected`是反射的，反对称的映射。

```java
Range.closed(3, 5).isConnected(Range.open(5, 10)); //true
Range.closed(0, 9).isConnected(Range.closed(3, 4)); //true
Range.closed(0, 5).isConnected(Range.open(3, 9)); //true
Range.open(3, 5).isConnected(Range.open(5, 10)); //false
Range.closed(1, 5).isConnected(Range.open(6, 10)); //false
```

#### `intersection`
`Range.intersection(Range)`返回两个区间的最大交集，如果没有交集，抛出`IllegalArgumentException`。

`intersection`是可交换的关联操作。

```java
Range.closed(3, 5).intersection(Range.open(5, 10)); //(5..5]
Range.closed(0, 9).intersection(Range.closed(3, 4)); //[3..4]
Range.closed(0, 5).intersection(Range.closed(3, 9)); //[3..5]
Range.open(3, 5).intersection(Range.open(5, 10)); //IllegalArgumentException
Range.closed(1, 5).intersection(Range.closed(6, 10)); //IllegalArgumentException
```

#### `span`
`Range.span(Range)`返回两个区间的最小并集。

`span`是可交换的关联的封闭的操作。
```java
Range.closed(3, 5).span(Range.open(5, 10)); //[3..10)
Range.closed(0, 9).span(Range.closed(3, 4)); //[0..9]
Range.closed(0, 5).span(Range.closed(3, 9)); //[0..9]
Range.open(3, 5).span(Range.open(5, 10)); //(3..10)
Range.closed(1, 5).span(Range.closed(6, 10)); //[1, 10]
```

## 非连续区域
有一些类型（但不是所有的可比较的类型）是非连续的，即两边的边界是可以列举出来的。

Guava中，`DiscreteDomain<C>`为C类型的数据实现了非连续的操作。一个非连续的区域表示该类型的所有制（全集），不能表示部分区域例如质数，长度为5的字符串，午夜时间戳等。

`DiscreteDomain`类提供了`DiscreteDomain`实例：

类型 | DiscreteDomain
--- | ---
Integer | integers()
Long | longs()

一旦获取到了`DiscreteDomain`，即可使用以下`Range`操作：

- `ContiguousSet.create(range, domain)`：将`Range<C>`作为一个`ImmutableSet<C>`视图
- `canonical(domain)`：把ranges变为“规范形式”。如果`ContiguousSet.create(a, domain).equals(ContiguousSet.create(b, domain))`并且`!a.isEmpty()`，那么`a.canonical(domain).equals(b.canonical(domain))`。

```java
ImmutableSortedSet<Integer> set = ContiguousSet.create(Range.open(1, 5), DiscreteDomain.integers());
// set包含[2,3,4]

ContiguousSet.create(Range.greaterThan(0), DiscreteDomain.integers());
//set包含[1,2,3...,Integer.MAX_VALUE]
```

`Contiguous.create`并不是真正的构造完整的范围，只是返回一个范围的Set视图。

### 自定义非连续区域
可以自定义我们自己的`DiscreteDomain`对象，但是请记住以下关于`DiscreteDomain`的几条重要原则：

- 一个非连续的区域总是表示该类型的全集；不能表示部分区域（质数、长度为5的字符串等）。例如，不能通过精确到秒的`JODA DateTime`来构造`DiscreteDomain`来查看工作日的集合，因为它不是该类型的全集。
- `DiscreteDomain`可能是无穷大的，例如`BigInteger`的`DiscreteDomain`，这种情况下，应该使用默认的抛出`NoSuchElementException`的`minValue()`，`maxValue()`。禁止在一个无限大的范围上使用`ContiguousSet.create()`方法。

### 如果需要一个`Comparator`该怎么做？
TODO
