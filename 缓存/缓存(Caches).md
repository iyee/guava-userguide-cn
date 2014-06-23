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