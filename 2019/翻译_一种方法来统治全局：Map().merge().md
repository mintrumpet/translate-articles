# 翻译 | 一种方法来统治全局：Map().merge()

![mark](https://image.talkmoney.cn/luwei/20190310/Q7D1wOsJR2Tq.jpg?imageslim)

> 原文自国外技术社区dzone，作者为 Tomasz Nurkiewicz，[ 传送门](https://dzone.com/articles/one-method-to-rule-them-all-mapmerge)

我不常分析一个在 JDK 下的一个方法，但当我这样做的时候，那就是关于 ` Map.merge()`。这可能是 key-value 领域中最通用的操作。但也是不被人所知的而且极少被人使用。

`merge()`  可以解释为如下：它要么将新值放置到给定的键上（如果不存在）或者将给定的值更新到键上（UPSERT）。让我们以最基本的例子开始说起：计算每个单词出现的次数。Java8 之前的实现代码十分混乱并且在实现细节中也看不出本质：

```java
var map = new HashMap<String, Integer>();
words.forEach(word -> {
    var prev = map.get(word);
    if (prev == null) {
        map.put(word, 1);
    } else {
        map.put(word, prev + 1);
    }
});
```

然而，这是可行的，并且对于给定的输入，能够得到期望的输出：

``` java
var words = List.of("Foo", "Bar", "Foo", "Buzz", "Foo", "Buzz", "Fizz", "Fizz");
//...
{Bar=1, Fizz=2, Foo=3, Buzz=2}
```

好的，让我们对它重构一下，从而去除判断逻辑处理：

```java
words.forEach(word -> {
    map.putIfAbsent(word, 0);
    map.put(word, map.get(word) + 1);
});
```

棒棒的！:+1:

`putIfAbsent()` 是必要的；否则，代码会在第一次出现未知的单词中中断。并且我发现在 `map.put()` 中使用 `map.get(word)` 会有点不合适，于是我们也把它处理掉吧！

```java
words.forEach(word -> {
    map.putIfAbsent(word, 0);
    map.computeIfPresent(word, (w, prev) -> prev + 1);
});
```

`computeIfPresent()` 只有在（word）键存在的时候才会执行给定的转换代码，反之不会做任何处理。我们通过将键值初始化为0确保它存在，因此递增总是生效。是否能做得更好？其实我们可以减少额外的初始化工作，但是我不推荐这么做：

```java
words.forEach(word ->
        map.compute(word, (w, prev) -> prev != null ? prev + 1 : 1)
);
```

`compute ()` 像 `computeIfPresent()`，但它的调用与键存在与否无关。如果键值不存在，`prev` 参数就会是 `null`。将一个简单的 `if` 判断移动到隐藏在 lambda 中的三元表达式永远不是最好的。这时候就体现 `merge()` 方法的优势所在了。在我给你们展示最终的结果之前，让我们先看下 `Map.merge()` 中稍微简化的 default 实现吧：

``` java
default V merge(K key, V value, BiFunction<V, V, V> remappingFunction) {
    V oldValue = get(key);
    V newValue = (oldValue == null) ? value :
               remappingFunction.apply(oldValue, value);
    if (newValue == null) {
        remove(key);
    } else {
        put(key, newValue);
    }
    return newValue;
}
```

这段代码胜过千言万语。`merge()` 在两个场景中运行。如果给定的键不存在，它会随之变为 `put(key, value)`。但是，如果这个键已经包含一些值，我们的 `remappingFunction` 会与旧的值合并为一。这方法可以随意地做以下的事情：

- 通过简单地返回新值来覆盖旧值：`(old, new) -> new`
- 通过简单地返回旧值来保留旧值：`(old, new) -> old`
- 以某种方式结合两者，例如：`(old, new) -> old + new`
- 甚至移除旧值：`(old, new) -> null`

正如你看到的那样，`merge()` 能做很多事情，那么，对于我们上述的问题，`merge（）` 到底是怎么样操作的呢？很简单：

```java
words.forEach(word ->
        map.merge(word, 1, (prev, one) -> prev + one)
);
```

你可以这样去解读：如果键 `word` 存在，将1 设进去；否则，在原有的值中增加1。我将其中一个参数命名为 `one`，是因为在我们的例子中它就是。。。1。

遗憾的是，`remappingFunction` 带有两个参数，其中第二个使我们将要 upsert（插入或更新） 进去的值。从技术的层面上看，我们已经知道这个值了，所以 `(word, 1, prev -> prev + 1)`  会更容易让人理解，但可惜的是并没有类似的API。

好的，但 `merge()` 是否真的有用呢？假设你要开发一个账目操作（省略构造函数、getter和其他有用的属性）：

```java
class Operation {
    private final String accNo;
    private final BigDecimal amount;
}
```

以及一大堆关于不同账目的操作：

```java
var operations = List.of(
    new Operation("123", new BigDecimal("10")),
    new Operation("456", new BigDecimal("1200")),
    new Operation("123", new BigDecimal("-4")),
    new Operation("123", new BigDecimal("8")),
    new Operation("456", new BigDecimal("800")),
    new Operation("456", new BigDecimal("-1500")),
    new Operation("123", new BigDecimal("2")),
    new Operation("123", new BigDecimal("-6.5")),
    new Operation("456", new BigDecimal("-600"))
);
```

我们想要计算每个账目的余额，在不使用 `merge()` 的情况下，这变得十分笨重：

``` java
var balances = new HashMap<String, BigDecimal>();
operations.forEach(op -> {
    var key = op.getAccNo();
    balances.putIfAbsent(key, BigDecimal.ZERO);
    balances.computeIfPresent(key, (accNo, prev) -> prev.add(op.getAmount()));
});
```

但经过 `merge()` 的小小帮助下：

``` java
operations.forEach(op ->
        balances.merge(op.getAccNo(), op.getAmount(), 
                (soFar, amount) -> soFar.add(amount))
);
```

你在这里看到使用方法引用的机会吗？

```java
operations.forEach(op ->
        balances.merge(op.getAccNo(), op.getAmount(), BigDecimal::add)
);
```

我发现这可读性变得非常强。对于每一步操作，将给定的 `amount` 通过 `add` 到给定的 `accNo` 中。最后是期望的结果：

```
{123=9.5, 456=-100}
```

## ConcurrentHashMap

如果你意识到 `map.merge()` 正好被应用在 `ConcurrentHashMap` 的时候，你会更加惊讶。这意味着我们能够进行原子性的插入或更新操作了 — 单行并且线程安全。

`ConcurrentHashMap` 显然是线程安全，但并非所有操作都是，例如 `get()` 然后 `put()`。但更重要的是，`merge()` 可以确保所有的更新不会丢失。