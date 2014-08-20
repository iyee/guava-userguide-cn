# 概述
Java的原生类型是其基础数据类型：
`byte`     `short`    `int`     `long`
`float`   `double`   `char`   `boolean`

__在查找一个Guava的方法前，应该先在`Arrays`或相关JDK的包装类型中查找，例如，`Integer`。__

这些类型不能用作对象或泛型的类型参数，这意味着很多通用的工具与它们不兼容。Guava在原生类型和集合API之间提供了一些通用的工具类、方法和接口，把特定类型转换为`byte`数组，并且支持特定类型的无符号操作。

原生类型 | Guava工具（在com.google.common.primitives包中）
--- | ---
byte | Bytes, SignedBytes, UnsignedBytes
short | Shorts
int | Ints, UnsignedInts, UnsignedInteger
long | Longs, UnsignedLongs, UnsignedLong
float | Floats
double | Doubles
char | Chars
boolean | Booleans

`Bytes`中有符号和无符号的字节的方法的不同之处在于字节被忽略，目前仅适用于`SignedBytes`和`UnsignedBytes`，因为有符号的字节比其他有符号的类型更加xxxx。

`int`和`long`的无符号类型的变体是`UnsignedInts`和`UnsignedLongs`，因为它们的大多数使用场景是有符号的，`Ints`和`Longs`默认将输入的参数作为有符号的。

另外，Guava还为`int`和`long`提供了无符号的包装类型`UnsignedInteger`, `UnsignedLong`，它为类型系统区别有无符号提供了支持（以一点性能损失作为代价），这些类直接支持简单的数学运算（类似`BigInteger`）。

所有方法的签名使用`Wrapper`来引用相关的JDK包装类型，`Prim`来引用原生类型（`Prims`引用相关Guava的工具类）

# 原生数组工具
原生数组是原生类型最有效率（内存和性能）的聚合。Guava提供了一系列这方面的工具：

签名 | 描述 | 集合 | 适用范围
--- | --- | --- | ---
List<Wrapper> asList(prim... backingArray) | 把原生类型的数组转换成对应包装类型的List | Arrays.asList | Sign-independent
prim[] toArray(Collection<Wrapper> collection) | 将集合拷贝到一个prim[]，类似于collection.toArray()一样也是线程安全的 | Collection.toArray() | Sign-independent
prim[] concat(prim[]... arrays) | 将几个原生类型的数组连结起来 | Iterables.concat | Sign-independent
boolean contains(prim[] array, prim target) | 检查指定元素是否存在 | Collection.contains | Sign-independent
int indexOf(prim[] array, prim target) | 查找指定元素的索引，没有则返回-1 | List.indexOf | Sign-independent
int lastIndexOf(prim[] array, prim target) | 查找指定元素的索引(反向查找)，没有则返回-1 | List.lastIndexOf | Sign-independent
prim min(prim... array) | 返回数组中最小的元素 | Collections.min | Sign-dependent
prim max(prim... array) | 返回数组中最大的元素 | Collections.max | Sign-dependent
String join(String separator, prim... array) | 用指定数组和分隔符构建一个字符串 | Joiner.on(separator).join | Sign-dependent
Comparator<prim[]> lexicographicalComparator() | 一个按字典顺序比较原生类型数组的比较器 | Ordering.natural().lexicographical() | Sign-dependent

Sign-independent方法包括：`Bytes`, `Shorts`, `Ints`, `Longs`, `Floats`, `Doubles`, `Chars`, `Booleans`。不包括：`UnsignedInts`, `UnsignedLongs`, `SignedBytes`, `UnsignedBytes`。

Sign-dependent方法包括：`SignedBytes`, `UnsignedBytes`, `Shorts`, `Ints`, `Longs`, `Floats`, `Doubles`, `Chars`, `Booleans`, `UnsignedInts`, `UnsignedLongs`。不包括`Bytes`。

# 通用工具方法
Guava提供了一系列JDK6没有的方法，其中有一些在JDK7中已经提供。
签名 | 描述 | 适用范围
--- | --- | ---
int compare(prim a, prim b) | 原生类型的比较器，已在JDK7的包装类中提供 | Sign-dependent
prim checkedCast(long value) | 把指定值转换成prim，如果不能转换则抛出IllegalArgumentException | Sign-dependent for integral types only
prim saturatedCast(long value) | 把指定值转换成prim，如果不能转换则返回最近的prim值

这里的integral types包括：`byte`, `short`, `int`, `long`。不包括：`char`, `boolean`, `float`, `double`。

_注意：_`double`的近似值是在`com.google.common.math.DoubleMath`包中提供，且包含很多不同的舍入模式，详情[查看这里](数学运算/数学运算(Math).md)。

# 字节转换方法
Guava提供了使用大端序（big-endian）将原生类型转换为字节数组或字节数组转换为原生类型的方法。所有方法都是Sign-independent的（`Booleans`未提供这些方法所以除外）。

签名 | 描述
--- | ---
int BYTES | prim值的字节大小的常量
prim fromByteArray(byte[] bytes) | 
prim fromBytes(byte b1, ... byte bk) |
byte[] toByteArray(prim value) | 返回value的大端序的字节数组

## 无符号支持
`UnsignedInts`和`UnsignedLongs`工具类提供了一些通用方法，这些方法是Java通过包装类给有符号类型提供的方法。`UnsignedInts`和`UnsignedLongs`直接和原生类型交互：确认传入这些方法的都是无符号类型。

另外，对于`int`和`long`，Guava提供了"无符号的"包装类型（`UnsignedInteger`和`UnsignedLong`）来确保在类型系统中无符号和有符号值的重复问题，当然，有一点性能上的损耗。

### 通用方法
下面这些方法的有符号版本已经在JDK的包装类中有相应的实现（这里仅是无符号的方法）：

签名 | 说明
--- | ---
int UnsignedInts.parseUnsignedInt(String) long UnsignedLongs.parseUnsignedLong(String) | 以10进制将一个字符串解析为无符号数值
int UnsignedInts.parseUnsignedInt(String, int radix) long UnsignedLongs.parseUnsignedLong(String, int radix) | 以指定进制将一个字符串解析为无符号数值
String UnsignedInts.toString(int) String UnsignedLongs.toString(long) | 以10进制将无符号数值转为字符串
String UnsignedInts.toString(int value, int radix) String UnsignedLongs.toString(long value, int radix) | 以指定进制将无符号数值转为字符串

### 包装
通过无符号类型的包装可以让使用和转换更加方便。

签名 | 说明
--- | ---
UnsignedPrim plus(UnsignedPrim), minus, times, dividedBy, mod | 简单数学运算
UnsignedPrim valueOf(BigInteger) | 将一个BigInteger转换为UsignedPrim，若BigInteger是负数或不能转换则抛出IllegalArgumentException
UnsignedPrim valueOf(long) | 将一个long转换为UsignedPrim，若BigInteger是负数或不能转换则抛出IllegalArgumentException
UnsignedPrim fromPrimBits(prim value) | 将给定的值作为无符号查看。例如，UnsignedInteger.fromIntBits(1 << 31)得到2^31，即使1<<31是应该是一个int负值。
BigInteger bigIntegerValue() | 将UnsignedPrim转换为BigInteger
toString(), toString(int radix) | 返回该无符号值的字符串形式。
