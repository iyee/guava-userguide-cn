# 示例
```java
LoadingCahce<Key, Graph> graphs = CacheBuilder.newBuilder()
	.maximumSize(1000)
	.expireAfterWrite(10, TimeUnit.MINUTES)
	.removalListener(MY_LISTENER)
	.build(new CacheLoader<Key, Graph>() {
		public Graph load(Key key) throws AnyException {
			return createExpensiveGraph(key);
		}
	});
```

# 应用场景
缓存有着非常广泛的使用场景，并且作用也是非常惊人的。比如一个值的计算或获取的代价是很昂贵的，那么就应该考虑来缓存它，后续可能不止一次的使用到。

`Cache`很类似于`ConcurrentMap`，但又不完全一样。最基本的不同之处在于`ConcurrentMap`持久化所有元素直到它们被移除，而`Cache`是被配置为自动回收的，用来限制内存的使用。某些情况下，`LoadingCache`的自动缓存加载非常有用，即使它不回收内容。

总之，Guava的缓存适用于以下情况：

- 希望用空间换取时间，即划出一部分内存用户缓存，来换取加载速度
- keys不止一次被查询
- 缓存数据不会存储在除了内存的其他地方（Guava的缓存应用本地单例的。它们不会往文件或其他外部服务器写入数据，如果这些不能满足需求，请考虑使用Memcached）

如果你有以上的使用场景，那么Guava的缓存工具将会是你正确的选择。

`Cache`是通过建造者模式使用`CacheBuilder`来获取的，像上面的例子那样。自定义缓存是比较有趣的一部分。

_注意_，如果你不需要`Cache`的功能，那么`ConcurrentMap`的内存效率将会更高，但是任何`ConcurrentMap`都不能或很难拥有`Cache`的功能。

# Population
关于缓存你需要问自己的第一问题是：有没有实用的默认函数来加载或计算与一个key关联的值？如果有，那么你需要使用`CacheLoader`。如果没有，或者你需要覆写默认的“获取如果不存在”的行为，那么你需要传入一个`Callable`来接受回调。可以使用`Cache.put`直接插入元素，但是对于所有缓存的内容来说，自动缓存加载会更加容易。

## 从`CacheLoader`加载
`LoadingCache`是一个通过`CacheLoader`构建出来的`Cache`，创建一个`CacheLoader`非常容易，实现`V load(K key) throws Exception`方法，例如，可以通过下列方法创建一个`LoadingCache`：

```java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder().maximumSize(100).build(new CacheLoader<Key, Graph>() {
	public Graph load(Key k) throws AnyException {
		return createExpansiveGraph(k);
	}
});
...
try {
	return graphs.get(key);
} catch(EcecutionException e) {
	throw new OtherException(e.getCause());
}
```

查询`LoadingCache`的权威方法是通过`get(K)`方法，要么返回一个已缓存的值，要么使用`CacheLoader`来加载一个新的值。`CacheLoader`可能会抛出异常，`LoadingCache.get(K)`将会抛出`ExecutionException`。如果定义了不抛出经检查的异常，在获取的时候可以使用`getUnchecked(K)`来查询缓存。注意不要使用`getUnchecked(K)`方法来查询定义了检查异常的`CacheLoader`。

```java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder().expireAfterAccess(10, TimeUnit.MINUTES).build(new CacheLoader<Key, Graph>() {
	public Graph load(Key k) {
		return createExpansiveGraph(k);
	}
});
...
return graphs.getUnchecked(key);
```

批量查询可以使用`getAll(Iterable<? extends K>)`。默认情况下，`getAll`对于每一个不存在于缓存中的key都将触发一个单独的`CacheLoader.load`调用。当批量获取比单独获取跟家高效的时候，可以覆写`CacheLoader.loadAll()`改变这个行为。`getAll`的性能会有相应的提高。

## 从`Callable`加载
所有Guava的缓存，加载或不加载的，都支持`get(K, Callable<V>)`。这个方法返回与此key相关联的值，或从`Callable`中计算并添加到缓存中。直到加载完毕`Cache`的状态才会改变。这个方法使用了一种简单的方式来代替了“如果有，返回；如果没有，创建、存储并返回”模式。

```java
Cache<Key, Value> cache = CacheBuilder.newBuilder().maximumSize(1000).build(); //没有CacheLoader
try {
	cache.get(key, new Callable() {
		public Value call() throws AnyException {
			return doThingsTheHardWay();
		}
	})
} catch (ExecutionException e) {
	throw new OtherException(e.getCause());
}
```

## 直接插入
可以使用`Cache.put(key, value)`直接把值插入到缓存中。会覆盖该key之前关联的值。使用`Cache.asMap()`视图暴露的`ConcurrentMap`的方法也会同步改变，但是不会自动加载任何不存在的key-value，任何在该视图上的原子的操作都在自动缓存加载的作用域外。所以不管是使用`CacheLoader`还是`Callable`来加载不存在的key-value，尽量优先使用`Cache.get(Key, Callable<V>)`而不是`Cache.asMap().putIfAbsent()`。

# 回收
一个残酷的现实是我们不可能使用内存来缓存所有想缓存的数据。必须决定谁将先从缓存中删除。Guava提供了基本的回收类型：基于内存占用大小的回收，基于时间的回收和基于引用的回收。

# 基于内存占用大小的回收
如果不希望缓存占用过大的内存空间，查看`CacheBuilder.maximumSize(long)`。最近最久未使用的key-value将被回收。注意：`Cache`可能在尚未达到此限制的时候就进行回收了，尤其当大小接近限制的时候。

另外，如果不同的缓存具有不同的大小，例如，缓存的值内存模型完全不同，可能就需要指定`CacheBuilder.weigher(Wheigher)`和`CacheBuilder.maximumWeight(long)`。与`maximumSize`不同，weight在创建的时候就已经计算了，之后一直保持不变。

```java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder().maximumWeight(100000).weigher(new Weigher<Key, Graph>() {
	public int weigh(Key k, Graph g) {
		return g.vertices().size();
	}
}).build(new CacheLoader<Key, Graph>() {
	public Graph load(Key k) {
		return createExpensiveGraph(k);
	}
});
```

# 基于时间的回收
Guava提供了两种基于时间的回收方式：

- `expireAfterAccess(long, TimeUnit)`，仅在最后一次读写访问之后在指定的时间后才会回收，回收顺序类似于基于大小的回收。
- `expireAfterWrite(long, TimeUnit)`，在key-value被创建、修改后经过指定的时间后被回收，这对于一段时间后缓存的数据已经无效的情况下非常有用。

## 测试基于时间的回收
基于时间的回收的测试没有多麻烦，测试一个2秒回收的缓存并不需要实际等上2秒。使用`Ticker`接口和`CacheBuilder.ticker(Ticker)`方法来指定一个时间源，来代替系统的时钟。

# 基于引用的回收
Guava可以使用key或value的弱引用、value的弱引用来设置缓存以允许垃圾回收器回收entry。

- `CacheBuilder.weakKeys()`用弱引用来存储keys，允许垃圾回收器回收不再被其他变量引用（强引用或软引用）的key的entry。因为垃圾回收器使用地址相等性（==）来判断两个对象是否一致，所以这就会导致所有缓存使用==而不是`equals()`来判断key的相等性。
- `CacheBuilder.weakValues()`同上（keys改为values）。
-  `CacheBuilder.softValues()`使用软引用存储values。软引用是一种全局的最近最少使用的对象的回收机制。因为软引用的性能潜在的问题，推荐使用基于大小的回收。该方法同样会导致所有缓存使用==而不是`equals()`来判断value的相等性。

## 显式的删除
任何时候，都可以显式的刷新缓存而不必等待其被回收，通过以下方式：

- 单个的移除，使用`Cache.invalidate(key)`
- 批量的移除，使用`Cache.invalidateAll(keys)`
- 移除所有，使用`Cache.invalidateAll()`

## 移除监听器
在ebtry被移除的时候可以设置一个监听器来做一些额外的操作：`CacheBuilder.removalListener(RemovalListener)`，监听器会收到一个`RemovalNotification`参数，该参数指定了`RemovalCause`，包含key和value。

`RemovalListener`所有抛出的异常都会被记录和接收。

```java
CacheLoader<Key, DatabaseConnection> loader = new CacheLoader<Key, DatabaseConnection>() {
	public DatabaseConnection load(Key k) throws Exception {
		return openConnection(k);
	}
};

RemovalListener<Key, DatabaseConnection> removalListener = new RemovalListener<Key, DatabaseConnection>() {
	public void onRemoval(RemovalNotification<Key, DatabaseConnection> removal) {
		DatabaseConnection conn = removal.getValue();
		conn.close();
	}
};

return CacheBuilder.newBuilder().expireAfter(2, TimeUnit.MINUTES).removalListener(removalListener).build(loader);
```

__注意__：`RemovalListener`默认是同步执行的，因为缓存的维护是在缓存的操作中进行的，所以大量的操作会减慢缓存的功能。如果在移除监听器中有大量的操作，使用`RemovalListeners.asynchronous(RemovalListener, Executor)`包装一个`RemovalListener`来执行异步操作。

## 清理工作何时执行？
`CacheBuilder`构建的缓存不会“自动的”或在values过期后立即执行清理回收工作。相应的，在写操作后，或在写操作很少的情况下，在读操作后，会进行小量的维护工作。

原因如下：如果我们需要持续的维护缓存，就需要创建一个线程，那么它的操作就会和用户的操作为共享的锁产生竞争。另外一些环境也限制了线程的创建，会让`CacheBuilder`变的不可用。

但是我们让用户做出选择。如果你的缓存需要高吞吐量，那么就不需要担心清理过期的entry来进行缓存的维护操作；如果很少需要写操作，并且不希望清理操作阻塞读操作，那么可以创建自己的清理线程，并在其中间隔性的调用`Cache.cleanup()`。

如果需要为一个不经常进行写操作的`Cache`安排常规的缓存维护计划，使用`ScheduledExecutorService`。

## 刷新
刷新和回收不完全一致。像`LoadingCache.refresh(K)`，刷新key会为该key加载一个新值（可能是异步的）。旧值会在该key刷新时返回，与回收相反（回收会强制在值过期后才会去重新获取）。

如果在刷新的时候抛出了异常，旧值会保留，异常被记录并且接收（不处理也不向上抛）。

`CacheLoader`可以通过覆写`CacheLoader.reload(K, V)`在刷新时指定更加敏捷的行为 — 使用旧值计算新值。

```java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder().maximumSize(1000).refreshAfterWrite(1, TimeUnit.MINUTES).build(new CacheLoader<Key, Graph>() {
	public Graph load(Key k) {
		return getGraphFromDatabase(k);
	}

	public ListenableFuture<Graph> reload(final Key k, Graph prevGraph) {
		if (neverNeedsRefresh) {
			return Futures.immediateFuture(preGraph);
		} else {
			//异步
			ListenableFutureTask<Graph> task = ListenableFutureTask.create(new Callable<Graph>() {
				public Graph call() {
					return getGraphFromDatabase(k);
				}
			});
			executor.execute(task);
			return task;
		}
	}
});
```

可以通过`CacheBuilder.refreshAfterWrite(long, TimeUnit)`为缓存加上自动刷新。与`expireAfterWrite()`相反，`refreshAfterWrite()`能在一段时间后让某个key自动刷新，但是这个刷新动作仅在查询时才会发生（如果`CacheLoader.reload()`是异步实现的，刷新不会减慢查询的速度）。例如，可以在一个`Cache`上同时指定`refreshAfterWrite()`和`expireAfterWrite()`，在任何可以刷新的情况下，过期的计时器不会盲目的重置，而是在可以刷新之后却没有查询操作，此时才会真的过期。

# 功能
## 统计
使用`CacheBuilder.recordStats()`，可以打开Guava缓存的统计功能。`Cache.stats()`方法返回一个`CacheStats`对象，提供了以下统计功能：

- `hitRate()`返回请求的次数
- `averageLoadPenalty()`加载新值的平均时间，单位是纳秒
- `evictionCount()`缓存回收次数

除此之外还有很多统计方法。这些统计在缓存调整上非常严格，建议在有严格性能要求的程序上留意这些统计信息。

## `asMap`
在任何`Cache`上使用`asMap`以`ConcurrentMap`视图查看缓存，但是`asMap`和`Cache`是怎么交互的需要说明一下：

- `Cache.asMap`包含了当前`Cache`中已加载的全部entry。例如`Cache.asMap().keySet()`包含所有已加载的keys。
- `asMap().get(key)`本质上相当于`Cache.getIfAbsent(key)`，并且不会导致值的加载，这与`Map`的限制是一致的。
- 读写都会重置访问的时间（包括`Cache.asMap().get(Object)`和`Cache.asMap().put(K, V)`），但是不包含`containsKey(Object)`和其他`asMap()`视图操作。例如迭代`Cache.entrySet()`就不会重置访问的时间。

# 中断
加载方法（像`get`）不会抛出`InterruptedException`，之前曾经这么做过但是我们的支持尚未到位，它的好处只有一点但是成本偏高，如需详细信息请继续阅读。

`get`方法将请求未缓存的数据分为两大类：加载值和等待其他线程内加载值。//TODO （有点难）
