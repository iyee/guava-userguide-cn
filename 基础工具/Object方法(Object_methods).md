#  Object 常用方法
## equals
如果一个对象的属性可能为null，实现`Object.equals()`就有点麻烦了，因为要单独的检查null的情况。使用`Objects.equal()`可以在可能为null的情况下执行equals判断而不抛出NullPointerException。

```java
Objects.equal("a", "a"); //true
Objects.equal(null, "a"); //false
Objects.equal("a", null); //false
Objects.equal(null, null); //true
```
注意：JDK7新增`Objects`类提供了功能类似的`Objects.equals()`方法

## hashCode
更简单的对所有属性进行hash计算。Guava的`Objects.hashCode(Object...)`根据属性顺序创建一个顺序敏感的hash计算。使用`Objects.hashCode(f1,f2,...fn)`来替代手工构建。

注意：JDK7新增的`Objects`类提供了功能类似的`Objects.hash(Object...)`方法。

## toString
在调试阶段一个好的`toString()`是非常有用的，但是写起来是非常痛苦的。使用`Objects.toStringHelper()`可以很简单的创建一个有用的`toString()`，简单示例如下：
```java
// 返回"ClassName{x=1}"
Objects.toStringHelper(this).add("x", 1).toString();

//返回"MyObject{x=1}"
Objects.toStringHelper("MyObject").add("x", 1).toString();
```

## compare/compareTo
直接实现Comparator或Comparable是痛苦的，考虑下面的实现：
```java
class Person implements Comparable<Person> {
	private String lastName;
	private String firstName;
	private int zipCode;
	public int compareTo(Person other) {
		int cmp = lastName.compareTo(other.lastName);
		if (cmp != 0) {
			return cmp;
		}
		cmp = firstName.compareTo(other.firstName);
		if (cmp != 0) {
			return cmp;
		}
		return Integer.compare(zipCode, other.zipCode);
	}
}
```
这段代码非常混乱，很难找出问题，而且非常冗余，我们应该可以做到更好。

因此，Guava提供了`ComparisonChain`。

`ComparisonChain`是惰性比较：它仅在找到非零值的时候才进行比较，完成后忽略后面的输入。

```java
public int compareTo(Foo that) {
	return ComparisonChain.start()
		.compare(this.aString, that.aString)
		.compare(this.anInt, that.anInt)
		.compare(this.anEnum, that.anEnum, Ordering.nature().nullsFirst())
		.result();
}
```
这样流式的方式具有更好的可读性，不会导致意外的错误，除了必须的不会做任何额外的工作，足够精简。其他关于`Comparison`的工具类可以在"Guava的流式比较器——[Ordering](基础工具/排序(Ordering).md)"中找到。
