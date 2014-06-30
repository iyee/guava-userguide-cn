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

__注意：__`Joiner`实例总是不可变的。`Joiner`的配置方法总是返回一个新的`Joiner`实例。这让`Joiner`是线程安全的，并且可用于静态常量。

# `Splitter`
Java内置的分割字符串的工具的行为有点奇怪。例如`String.split`会默认的丢弃路径分隔符，而`StringTokenizer`会将其表示为5个空格。

小测试：",a,,b,".split(",")的结果是什么？

1. "", "a", "", "b", ""
2. null, "a", null, "b", null
3. "a", null, "b"
4. "a", "b"
5. 以上都不是

正确的答案是5："", "a", "", "b"，只有最后一个空字符串被忽略，为什么会这样我也不太清楚。

`Splitter`允许你安心的直接使用流式模式来完全控制这些令人困惑的行为。

```java
Splitter.on(",")
	.trimResults()
	.omitEmptyStrings()
	.split("foo,bar,,   qux");
```

以上的代码返回的是一个`Iterable<String>`，其中包含"foo","bar","qux"。`Splitter`可用于任何`Pattern`, `char`, `String`或`CharMatcher`。

基本工厂方法

方法 | 描述 | 示例
--- | --- | ---
Splitter.on(char) | 使用单独的一个字符分隔指定序列 | Splitter.on(';')
Splitter.on(CharMatcher) | 使用某类的字符分隔指定序列 | Splitter.on(CharMatcher.BREAKING_WHITE) Splitter.on(CharMatcher.anyOf(";,."))
Splitter.on(String) | 使用字符串分隔指定序列 | Splitter.on(", ")
Splitter.on(Pattern) Splitter.onPattern(String) | 根据正则表达式分隔指定序列 | Splitter.onPattern("\r?\n")
Splitter.fixedLength(int) | 将指定序列分隔成指定长度的子串，最后一个子串可能小于指定的值，但永远不可能为空 | Splitter.fixedLength(3)

配置
方法 | 描述 | 示例
--- | --- | ---
omitEmptyStrings() | 在结果中忽略空字符串 | Splitter.on(',').omitEmptyStrings().split("a,,c,d")会返回"a","c","d"
trimResults() | 剔除结果中的空格，相当于trimResults(CharMatcher.WHITESPACE) | Splitter.on(',').trimResults().split("a, b, c, d")会返回"a", "b", "c", "d"
trimResults(CharMatcher) | 剔除结果中的指定字符 | Splitter.on(',').trimResults(CharMatcher.is('_')).split("_a ,_b_ ,c__")会返回"a ", "b_ ", "c"
limit(int) | 字符串分割的最大数量（即将分隔成几部分，最后一部分包含所有剩下的字符串，不会再进行分隔） | Splitter.on(',').limit(3).split("a,b,c,d")会返回"a", "b", "c,d"

TODO：`Map`分割

如果希望返回`List`，使用`Lists.newArrayList(Splitter.split(String))`或类似方法即可。

__注意：__`Splitter`实例是不可变的。分割器的配置方法始终返回一个新的`Splitter`。这样会使`Splitter`是线程安全的，并且可以用作常量。

# `CharMatcher`
之前，我们的`StringUtil`类变成了未经检查的，它有许多以下类似的方法：

- | - | - | - | -
--- | --- | --- | --- | --- | ---
allAscii | collapse | collapseControlChars | collapseWhitespace | indexOfChars
lastIndexNotOf | numSharedChars | removeChars | removeCrLf | replaceChars
retainAllChars | strip | stripAndCollapse | stripNonDigits

这些方法表示了一个跨产品的两个概念：

1. 一个“匹配”的字符由什么组成？
2. 对这些“匹配”的字符要做什么？

为了简化这个困境，我们开发了`CharMatcher`这个类。

可以把`CharMatcher`直观的想象成是一个表示字符的特殊类，像数字或空格之类。实际上，`CharMatcher`就是一个字符的布尔判判断 — 它实现了`Predicate<Character>`接口，因为它太常用了，例如：所有空格，所有小写字母等，Guava才为字符提供了这个API。

`CharMatcher`的作用就是在一个字符序列上执行一系列的操作：裁剪，折叠，移除，保留等等。一个`CharMatcher`类型的对象表示：

1. 一个“匹配”的字符由什么组成？
有很多操作来回答这个问题。
2. 对这些“匹配”的字符要做什么？
The result is that API complexity increases linearly for quadratically increasing flexibility and power. Yay!

```java
String noControl = CharMatcher.JAVA_ISO_CONTROL.removeFrom(string);
//删除控制字符
String theDigits = CharMatcher.DIGIT.retainFrom(string);
//仅保留数字
String spaced = CharMatcher.WHITESPACE.trimAndCollapseFrom(string, ' ');
//剔除结尾的空白，并将空白替换为单个的空格
String noDigits = CharMatcher.JAVA_DIGIT.replaceFrom(string, "*");
String lowerAndDigit = CharMatcher.JAVA_DIGIT.or(CharMatcher.JAVA_LOWER_CASE).retainFrom(string);
//删除所有不是数字和小写字母的字符
```

___注意：__`CharMatcher`仅接受`char`值，不接受从0x10000到0x10FFFF之间的Unicode代码值。这些逻辑字符是使用surrogate pair编码为`String`的，`CharMatcher`将其作为两个分开的字符处理。

## 获取`CharMatcher`
很多需求都可以使用已提供的`CharMatcher`常量：
- | - | - | - | -
--- | --- | --- | --- | ---
ANY | NONE | WHITESPACE | BREAKING_WHITESPACE | INVISIABLE
DIGIT | JAVA_LETTER | JAVA_DIGIT | JAVA_LETTER_OR_DIGIT | JAVA_ISO_CONTROL
JAVA_LOWER_CASE | JAVA_UPPER_CASE | ASCII | SINGLE_WIDTH

其他获取`CharMatcher`的常规方法包括：

方法 | 描述
--- | ---
anyOf(CharSequence) | 包括想要匹配的所有字符。例如`CharMatcher.anyOf("aeiou")`匹配所有小写的元音字符
is(char) | 匹配指定的字符
inRange(char, char) | 匹配一个范围内的字符，例如`CharMatcher.inRange('a', 'z')`

另外，还有`negate()`，`and(CharMatcher)`和`or(CharMatcher)`，这些方法提供`CharMatcher`的简单布尔操作。

## 使用`CharMatcher`
`CharMatcher`提供了很多操作`CharSequence`的方法。以下列出的是常用的方法，更多方法参加Javadoc文档：

方法 | 描述
--- | ---
collapseFrom(CharSequence, char) | 讲连续匹配的字符替换为指定的字符，例如：`WHITESPACE.collapseFrom(string, ' ')`会将连续的空白符替换为一个单一的空格。
matchesAllOf(CharSequence) | 测试是否匹配所有字符。例如`ASCII.matchesAllOf(string)`检查string中的字符是否都是ASCII。
removeFrom(CharSequence) | 从字符序列中删除匹配的字符
retainFrom(CharSequence) | 从字符序列中删除不匹配的字符（仅保留匹配的字符）
trimFrom(CharSequence) | 从字符序列的头尾删除指定的字符
replaceFrom(CharSequence, CharSequence) | 用指定字符序列替换匹配的字符

（注意：这些方法都返回一个`String`，`matchesAllOf`返回`boolean`）

# `Charsets`
不要像下面这样做：

```java
try {
	bytes = string.getBytes("UTF-8");
} catch (UnSupportedEncodingException e) {
	throw new AssertError(e);
}
```

而要这么做：

```java
bytes = string.getBytes(Charsets.UTF_8);
```

`Charsets`提供了6种`Charset`的标准实现的常量引用，保证被所有Java平台实现支持。使用它们的名字来引用。

TODO：`Charsets`说明和应用场景

（注意：如果你使用JDK7，应该使用`StandardCharsets`中的常量！）

# `CaseFormat`
`CaseFormat`是一个用来转换ASCII大小写的工具类 — 例如，编程语言的命名约定。支持的格式如下：

格式 | 示例
--- | ---
LOWER_CAMEL | lowerCamel
LOWER_HYPHEN | lower-hyphen
LOWER_UNDERSCORE | lower_underscore
UPPER_CAMEL | UpperCamel
UPPER_UNDERSCORE | UPPER_UNDERSCORE

```java
CaseFormat.UPPER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL, "CONSTANT_NAME");
//返回"constantName"
```
