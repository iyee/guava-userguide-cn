# `Joiner`
将一个字符串序列用一个分隔符连接起来还是比较棘手的 — 但不应该是这样。如果序列包含null就更困难了，流式风格的`Joiner`让这一切变得简单起来。

```java
Joiner joiner = Joiner.on("; ").skipNulls();
return joiner.join("Harry", null, "Ron", "Hermione");
```

以上代码返回的是"Harry; Ron; Hermione"。另外，也可以使用`useForNull(String)`来指定一个字符串作为null的替代。

同样，`Joiner`也可以在Object上，使用`toString()`转换成字符串然后再合并。

```java
//返回1,5,7
Joiner.on(",").join(Arrays.asList(1,5,7));
```

__注意：__`Joiner`实例总是不可变的。`Joiner`的配置方法总是返回一个新的`Joiner`实例。