# 使用/避免null
由于疏忽而使用null会导致一系列各种各样的bug。在学习Guava的时候，我们会发现95%的集合类不允许有null值，如果有null的话会在第一时间抛出异常而不是默默的接受，这样对于开发者来说是很有帮助的。

另外，null会造成模棱两可的歧义。一个返回null的方法很难明显的表达它的意义：例如`Map.get(key)`返回null可能表示这个key对应的value是null，也可能表示这个key根本不存在Map中。null可以表示失败，可以表示成功，甚至可以表示任何情况。使用其他非null的方式会使意义更加明确。

也就是说，使用null有的时候也是正确的方式。其在内存和速度上的开销很小，并且在Object数组中是不可避免的。但是在应用程序（非Library库）代码中，它是混乱、难以理解以及稀奇古怪的bug的主要来源（例如上面的`Map.get(key)`），绝大多数情况下，null不能表示一个null值的意义。

基于这些原因，大多数的Guava的工具类被设计为一旦接受到null会立即抛出异常。另外，Guava提供了一些工具来帮助我们避免并且更易的使用null。

## 特殊情况
如果你试图在Set或Map的key中使用null，很显然不应该这么做，如果你明确在执行查找操作的时候null的特殊情况。

如果试图在Map的value中使用null，那么直接不要使用这个entry；很容易混淆Map是包含了null值的key还是该key根本不存在。更简单的做法是为所有的key另外保存一份Set，这样当返回null的时候只需要考虑这个key对应的null值在程序中的意义了。

如果在List中使用null，如果该List是离散的，更好的替代方案是使用`Map<Integer, E>`，效率更高，而且实际上会更加符合我们的需求。

再看看使用null Object的时候，如果这个Object是一个enum，就需要添加一个常量表示这个enum是null时的情况，例如`java.math.RoundingMode`有一个`UNNECESSARY`常量表示“不作任何四舍五入的操作，如果需要这些操作就抛出异常”。

如果真的需要null的话，或者在使用非null的Collection实现的时候有问题，那么就使用一个不同的实现，例如：`Collections.unmodifiableList(Lists.newArrayList())`来替代`ImmutableList`

## Optional类
有时候我们想要使用null表示不存在，但是可能它已经存在只是值为null（例如`Map<Integer, E>`）。

`Optional<T>`是一个空指针的替代方案。它可以包含一个非空的引用（我们说该引用为“present”），也可以不包含任何引用（我们说该引用为“absent”），但是绝对不会包含一个null。
```java
Optional<Integer> possible = Optional.of(5);
possible.isPresent(); // returns true
possible.get(); // returns 5
```
	
以下是比较常用的`Optional`操作：
### 创建一个Optional
方法 | 描述
---- | ----
`Optional.of(T)` | 创建一个包含非null的值，如果是该值为null，立即抛出异常
`Optional.absent()`|返回某个类型的absent的Optional
`Optional.fromNullable(T)`|把传入的可能为null的引用放入Optional，把非null当作present，null当作absent

### 查询方法
以下都是非静态的方法

方法 | 描述
---- | -----
`boolean isPresent()` | 如果此Optional包含非null的引用则返回true
`T get()` | 返回包含的引用T，该引用必须是present的，否则抛出IllegalArgumentException
`T or()` | 返回该Optional中的引用，如果该引用不是"present"的，则返回参数指定的默认值
`T orNull()` | 返回该Optional中的引用，如果该引用不是"present"的，返回null，与`fromNullable(T)`是相反的操作
`Set<T> asSet()` | 返回一个Optional中包含的实例的不可变的Set集合（如果有），否则返回空的不可变集合
更多操作请参见Javadoc。
### 意义何在
Optional最大的好处是它具有傻瓜式的防范，它强制我们思考absent的情况，反之null会使我们自然的忘记某些事，即使FindBugs可以帮助我们，我们仍然不相信它能非常好的应付这种情况。

尤其在返回值可能为“present”或可能不为“present”的时候，我们常常会忘记处理返回null时的情况，而返回Optional则不会有这种情况，因为想要通过编译我们不得不把Optional中的值取出来。

## 方便快捷的方法
任何想要把null替换为其他默认值的时候，使用`Objects.firstNonNull(T, T)`。正如方法名一样，会返回第一个不为null的值，如果两个参数都是null，会立即抛出NullPointerException，这是使用Optional的一个好处。

Strings类提供了一些与可能为null的值交互的方法，都是一些顾名思义的方法：

方法 |
---- |
`emptyToNull(String)` |
`isNullOrEmpty(String)` |
`nullToEmpty()` |

我们强调一下，null String和empty String如果表示的意义一致，那么你的代码就需要重构了。当你把null String和empty String混合在一些的时候，Guava团队会感到伤心（如果他们表示不同的意义，那么情况可能就会好那么一点）
