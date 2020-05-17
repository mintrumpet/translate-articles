# 翻译 | Java 中 Stream 与 Array 间的相互转换

> 原文自工程师baeldung博客，作者为 baeldung，[传送门](https://www.baeldung.com/java-stream-to-array)

## 1. 综述

将各种各样的动态数据类型转换为数组在 java 中是非常普遍的。

在这篇文章中，将说明如何处理 stream 和 array 间的相互转换。

## 2. 将 stream 转换为 array

### 2.1. 方法引用

**将 stream 转换为 array 的最好方法是使用 stream 的 *toArray()* 方法**：

``` java
public String[] usingMethodReference(Stream<String> stringStream) {
    return stringStream.toArray(String[]::new);
}
```

现在，我们可以简单测试下转换是否能够顺利执行：

```java
Stream<String> stringStream = Stream.of("baeldung", "convert", "to", "string", "array");
assertArrayEquals(new String[] { "baeldung", "convert", "to", "string", "array" },
    usingMethodReference(stringStream));
```

### 2.2. lambda 表达式

另一种相当的方法是**传递一个 lambda 表达式**到 *toArray()* 方法的：

```java
public static String[] usingLambda(Stream<String> stringStream) {
    return stringStream.toArray(size -> new String[size]);
}
```

这和使用方法引用一样，能够获得相同的结果。

### 2.3. 自定义类

或者，我们能够用自己的方式，创建一个完整的类来处理。

将 *IntFunction* 作为参数，它将数组大小作为输入，并且返回该数组的大小。

当然，*IntFunction* 是作为一个接口，所以我们可以实现它：

```java
class MyArrayFunction implements IntFunction<String[]> {
    @Override
    public String[] apply(int size) {
        return new String[size];
    }
};
```

然后我们可以正常的构建并且使用：

```java
public String[] usingCustomClass(Stream<String> stringStream) {
    return stringStream.toArray(new MyArrayFunction());
}
```

所以，我们可以使用之前的断言来测试它是否成功。

### 2.4. 基本数据类型数组

在先前的章节中，我们探索了如何将 *String stream* 转换为 *String array*。事实上，我们可以使用上述转换方法去处理任何 *Object* 并且和上述的 *String* 例子非常相像。

但对基本数据类型有点不一样。例如，如果我们需要将 *Integer stream* 转换为 *int[]*，首先我们需要调用 *mapToInt()* 方法：

```java
public int[] intStreamToPrimitiveIntArray(Stream<Integer> integerStream) {
    return integerStream.mapToInt(i -> i).toArray();
}
```

我们还可以使用 *mapToLong()* 和 *mapToDouble()* 方法。另外，请注意这次我们并没有向 *toArray()* 传递任何参数：

最终，让我们使用相同的断言来判断，并且确认我们能够正确地获取到 *int* 数组：

```java
Stream<Integer> integerStream = IntStream.rangeClosed(1, 7).boxed();
assertArrayEquals(new int[]{1, 2, 3, 4, 5, 6, 7}, intStreamToPrimitiveIntArray(integerStream));
```

但是，如果我们想要去做相反的事呢？让我们往后看看吧。

## 3. 将 array 转换为 stream

理所当然，我们也可以去处理相反的情况。并且 java 也提供了一些专门的方法来处理。

### 3.1. *objects* 数组

我们能够转换 array 到 stream ，通过使用 **Arrays.stream()** 或者 **Stream.of()** 方法：

```java
public Stream<String> stringArrayToStreamUsingArraysStream(String[] stringArray) {
    return Arrays.stream(stringArray);
}
 
public Stream<String> stringArrayToStreamUsingStreamOf(String[] stringArray) {
    return Stream.of(stringArray);
}
```

我们应该注意到，在这两种情况下，我们的 stream 和 array 是同时出现的。

### 3.2. 基础数据类型数组

相同地，我们可以对基础数据类型数组进行转换：

```java
public IntStream primitiveIntArrayToStreamUsingArraysStream(int[] intArray) {
    return Arrays.stream(intArray);
}
 
public Stream<int[]> primitiveIntArrayToStreamUsingStreamOf(int[] intArray) {
    return Stream.of(intArray);
}
```

但是，与转换 *Objects* 数组相比，这里存在重要的差异。当转换基础数据类型数组的时候，***Arrays.stream()* 返回 *IntStream*, 但是 *Stream.of()* 则返回 *Stream<int[]>***。

### 3.3. *Arrays.stream* vs. *Stream.of*

为了理解前面几节中各自的差异，我们来看看相应方法各自的实现。

首先让我们看看 java 对于这两种方法的实现：

```java
public <T> Stream<T> stream(T[] array) {
    return stream(array, 0, array.length);
}
 
public <T> Stream<T> of(T... values) {
    return Arrays.stream(values);
}
```

可以看到，*Stream.of()* 实际上是在内部调用 * c*，很明显，这就是我们能够得到相同结果的原因。

现在，当我们需要转换一个基本类型数组时，我们来检查下这种情况下的方法：

```java
public IntStream stream(int[] array) {
    return stream(array, 0, array.length);
}
 
public <T> Stream<T> of(T t) {
    return StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
}
```

这次，*Stream.of()* 方法再也没调用 *Stream.of()* 方法了。

## 4. 结论

在这篇文章中，我们看到在 java 中 stream 和 array 是怎么做相互转换的。我们也解释了为什么在转换 Objects 数组和基本数据类型数组时得到了不同的结果。

 和往常一样，在 [GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays) 上可以找到我们的例子代码。