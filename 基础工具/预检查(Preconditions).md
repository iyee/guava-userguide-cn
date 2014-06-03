# Preconditions
Guava提供了一系列的预检查工具，我们强烈建议使用静态导入以便更方便的使用。每个方法都有3个不同的变体（重载）：

- 无参的方法。
如果有异常会直接抛出，不会有任何错误信息
- 有一个Object参数的方法。
如果有异常会直接抛出，使用`Object.toString()`作为错误信息
- 一个String和一个可变的Object参数
类似printf，但是为了效率和兼容GWT，只接受%s占位符，例如：
```java
checkArgument(i >= 0, "Argument was %s but expected nonnegative", i);
checkArgument(i < j, "Expected i < j, but %s > %s", i, j);
```

| 签名 | 描述 | 失败时抛出异常 |
----- | ----- | -----
`checkArgument(boolean)` | 检查boolean是否为true，用于验证参数 | `IllegalArgumentException`
`checkNotNull(T)` | 检查参数是否为null，直接返回该参数，可以内联使用 | `NullPointerException`
`checkState(boolean)` | 检查Object的状态，不依赖于方法参数，例如检查Iterator是否在remove之前调用了next | `IllegalStateException`
`checkElementIndex(int index, int size)` | 检查index的索引处的元素是否在List/Array/String内，元素的索引从0（包括）到`size`（不包括），不需要直接传List/Array/String，传他们的长度即可，返回`index` | `IndexOutOfBoundsException`
`checkPositionIndex(int index, int size)` | 检查index是否是一个合法的位置索引，范围从0（包括）到`size`（包括），不需要直接传List/Array/String，传他们的长度即可，返回`index` | `IndexOutOfBoundsException`
`checkPositionIndexes(int start, int end, int size)` | 检查[start, end)的范围是否是List/Array/String的子集 | `IndexOutOfBoundsException`

相对于Apache Commons的Comparable工具类，建议使用我们的`Preconditions`检查工具，原因正如Piotr Jagielski在[discusses](http://piotrjagielski.com/blog/google-guava-vs-apache-commons-for-argument-validation/)所说：

- 使用静态导入之后，Guava的方法非常清晰，`checkNotNull()`让我们非常清晰的知道即将发生什么，将会抛出什么异常
- `checkNotNull()`在验证完参数之后将其直接返回，这样就可以在构造函数一行内这么写：`this.field = checkNotNull(field)`.
- 简单，可变参数的“printf-style”异常信息（这也是我们建议在JDK7中继续使用`checkNotNull()`而不是`Objects.requireNotNull()`的原因）

我们建议分行使用Preconditions检查，以便于调试时知道是哪个Precondition失败了。另外，如果每个Precondition在单独的一行上，应该提供一个简单的有助于理解的错误信息。
