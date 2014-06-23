# 介绍
有时候你需要自己实现一些集合的扩展。在元素被添加到List的时候你可能需要添加一些特殊的行为，或者需要你一个支持数据库查询的`Iterable`。Guava为你同样也为我们自己提供了一系列的工具来简化这些操作。

# Forwarding装饰器
Guava为所有集合接口提供了抽象的`Forwarding`类来简化装饰模式的使用。

`Forwarding`类仅定义了一个抽象方法 - `delegate()`，需要覆写它并返回一个被包装的对象。所有其他的方法都会直接使用返回的对象代理：例如`ForwardingList.get(int)`的就是简单的使用`delegate.get(int)`来实现的。

继承`ForwardingXxx`并实现`delegate()`方法，可以只实现目标类的某个或某些方法 — 不需要自己处理所有代理方法。

另外，很多方法都有一个`standardMethod`实现，可以用来恢复预期的行为，提供了相同的好处，例如扩展JDK的`AbstractList`或其他核心框架类。

来看一个例子。假设现在需要包装一个List，以实现当元素被添加进去时可以记录日志。当然，无论通过什么方式添加元素，都需要记录下来 — `add(int, E)`，`add(E)`或`addAll()`，所以必须覆写这些方法。

```java
class AddLoggingList<E> extends ForwardingList<E> {
	final List<E> delegate;
	@Override
	protected List<E> delegate() {
		return delegate;
	}

	@Override
	public void add(int index, E elem) {
		log(elem);
		super.add(int, E);
	}

	@Override
	public boolean add(E) {
		return standardAdd(elem);
	}

	@Override
	public boolean addAll(Collection<? extends E> c) {
	return standardAddAll(c);
}
}
```

注意，默认情况下所有方法都直接重定位到代理对象，所以覆写`ForwardingMap.put()`并不会改变`ForwardingMap.putAll()`的行为。覆写必须要改变默认行为的方法需要格外小心，并确保符合被包装的集合的限制。

一般情况，像`AbstractList`这样的框架类的大部分方法，在`Forwarding`包装器中已经提供了标准的实现。

提供特殊视图的接口有时同样提供了那些视图的`standard`实现。例如，`ForwardingMap`提供了`standardKeySet`，`standardValues`，`standardEntrySet`，每个方法如果可能都被代理到了包装他们的`Map`中，如果不能被代理，则保持他们是抽象的（abstract method）。

接口 | Forwarding装饰器
--- | ---
Collection | ForwardingCollection
List | ForwardingList
Set | ForwardingSet
SortedSet | ForwardingSortedSet
Map | ForwardingMap
SortedMap | ForwardingSortedMap
ConcurrentMap | ForwardingConcurrentMap
Map.Entry | ForwardingMapEntry
Queue | ForwardingQueue
Iterator | ForwardingIterator
ListIterator | ForwardingListIterator
Multiset | ForwardingMultiList
Multimap | ForwardingMultimap
ListMultimap | ForwardingListMultimap
SetMultimap | ForwardingSetMultimap

# `PeekingIterator`
有时候，标准的`Iterator`接口不能满足我们的需求。

`Iterators`支持`Iterators.peekingIterator(Iterator)`方法，它包装了一个`Iterator`并返回一个子类`PeekongIterator`，可以使用`peek()`返回下一次调用`next()`时返回的元素（按照字面意思，即是在还没有`next()`时偷窥下一个元素）。

_注意_，在调用`peek()`之后就不能再使用`remove()`方法了。

举例，去除连续重复的元素并添加到List：

```java
List<E> result = Lists.newArrayListT();
PeekingIterator<E> iter = Iterators.peekingIterator(source.iterator());
while (iter.haxNext()) {
	E current = iter.next();
	while (iter.hasNext() && iter.peek().equals(current)) {
		//跳过重复的元素
		iter.next();
	}
	result.add(current);
}
```

# `AbstractIterator`
想实现自己的`Iterator`吗？`AbstractIterator`会让这边的更加容易。

举例说明，假设我们需要包装一个`Iterator`用来忽略null值。

```java
public static Iterator<String> skipNulls(final Iterator<String> in) {
	return new AbstractIterator() {
		protected String computeNext() {
			while(in.hasNext()) {
				String s = in.next();
				if (s != null) {
					return s;
				}
			}
			return endOfData();
		}
	};
}
```

我们只实现了一个`computeNext()`方法来计算下一个元素的值。当所有工作完成的时候，返回`endOfData()`来标识迭代的结束。

_注意_：`AbstractIterator`继承自不允许`remove()`的`UnmodifiableIterator`，如果需要一个支持`remove()`的迭代器，不要继承`AbstractIterator`。

# `AbstractSequentialIterator`

`AbstractSequentialIterator`提供了另一种表达迭代的方式。
```java
Iterator<Integer> powerdOfTwo = new AbstractSequentialIterator<Integer>(1) { //注意初始值
	protected Integer computeNext(Integer previous) {
		return (previous == 1 << 30) ? null : previous * 2;
	}
};
```
这里我们实现了`computeNext(T)`，它的参数是上一个迭代的值。

注意这里必须传入一个初始值，或者null（迭代立即结束）。`computeNext()`认为null值代表迭代的结束，也就是说，`AbstractSequentialIterator`不能用于实现可能包含null值的迭代器。
