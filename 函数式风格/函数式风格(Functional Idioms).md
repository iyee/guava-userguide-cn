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
