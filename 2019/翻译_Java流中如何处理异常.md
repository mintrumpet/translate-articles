# 翻译 | Java流中如何处理异常

> 原文自国外技术社区dzone，作者为 Brian Vermeer，[传送门](https://dzone.com/articles/exception-handling-in-java-streams)

> 如果在 lambda 中你想要使用一个抛出检查性异常的方法时，你需要额外做一些事情。

流API和 lambda 是 Java8 之后的一个巨大进步。从那时开始，我们能够使用更多函数式编码方式来开发。现在，经过这几年的代码建设，其中一个还遗留的大问题是如何在一个 lambda 表达式处理检查性异常。

大体和你知道的那样，在 lambda 中直接调起一个显性抛出检查性异常的方法是不可能的。在某种程度上，我们需要捕获这个异常使得代码能够成功编译。自然而然地，我们可以在 lambda 中使用一个简单的 try-catch 并且封装异常到`RuntimeException`中，就像下面的第一个例子一样，但是我想大家都会认为这并不是最佳的方法。

```java
myList.stream()
  .map(item -> {
    try {
      return doSomething(item);
    } catch (MyException e) {
      throw new RuntimeException(e);
    }
  })
  .forEach(System.out::println);
```

大部分人都会认为这段 lambda 又笨重而且易读性差，在编者的观点中，这应该是尽可能预防的。如果我们想在超过一行的代码中搞事情，我们大可把方法体提取出来并放到一个独立的方法中并且调用这个方法。一种更好并且更易读的解决方法是将请求封装到一个包含 try-catch 的简单方法中，并且在你的 lambda 中调用这个方法。

```java
myList.stream()
  .map(this::trySomething)
  .forEach(System.out::println);
private Item trySomething(Item item) {
  try {
    return doSomething(item);
  } catch (MyException e) {
    throw new RuntimeException(e);
  }
}
```

这种解决方法至少变得可读了并且分散了我们的关注点。如果你真的想捕获异常，做一些特定的事项而不是简单的将异常封装到`RuntimeException`当中，这或许是一个可行而且可读的解决方法。

### 运行时异常（RuntimeException）
在很多情况中，你会看到开发者会使用诸如此类的解决方法去重新包装异常到`RuntimeException`当中或者对一个非检查性异常的更具体的实现方法。通过这种做法，方法能够在 lambda 中被调用并且被使用在更高阶的方法当中。

这个做法个人认为关系不大（I can relate a bit to this practice）因为我觉得在通常情况下检查性异常并没有多大价值，但是这是另外一个讨论内容了所以我不打算在这里开展。如果你想嵌套所有包含`RuntimeException`检查的 lambda 请求，你会发现这种方式会重复许多次。为了防止一遍又一遍地重写相同的代码，为什么不将其抽象为一个应用函数（utility function, was used by Joshua Bloch in the book Effective Java to describe the methods on classes such as Arrays, Objects and Math.）呢？通过这种方法，你只需要编写一种代码并且在任何你需要的时候调用它。

要这样做，首先你需要为你的方法编写一个自定义的函数式接口（functional interface）。在这里，你需要定义一个抛出异常的方法。

```java
@FunctionalInterface
public interface CheckedFunction<T,R> {
    R apply(T t) throws Exception;
}
```

现在，你得准备编写一个接收上面定义的接口`CheckedFunction`的应用函数。你能够在函数里面处理 try-catch 并且将异常包装到`RuntimeException`当中（或者其他未检查变型（unchecked variant））。我知道我们现在以一个难看的 lambda 体结束并且你也能够将内容从这里抽象出来。你自己做选择看这个独立的应用函数是否值得付出代价。

```java
public static <T,R> Function<T,R> wrap(CheckedFunction<T,R> checkedFunction) {
  return t -> {
    try {
      return checkedFunction.apply(t);
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  };
}
```

通过一个简单的静态导入，现在你能使用全新的应用函数来封装包装 lambda 使其能抛出异常。从这里开始，所有事都变得可处理了。

```java
myList.stream()
       .map(wrap(item -> doSomething(item)))
       .forEach(System.out::println);
```

唯一遗留的问题是当异常发生了，流的处理会立刻停止。如果你觉得这样没问题，那就这样做吧。但是我能想象这种直接中止的行为对许多情况并不可行。

## Either
当使用流处理时，如果异常发生了，我们一般不希望程序中止。如果流上有非常大的一个数据需要被处理，你会希望当处理第二项的时候中止流处理吗？大概都不想吧。

让我们把思路转回来。为什么不尽可能多考虑”额外的情况“，就像我们想要一个”成功的“结果一样。让我们把这些情况当作数据，继续对流进行处理，并且决定之后我们如何处理它。我们能够处理它，但是想让其成为可能，我们需要介绍一种新的类型 — Either 类型。

Either 类型是函数式语言一种常用的类型，并且当前并还没成为 Java 的一部分。和 Java 中的 Optional 类型类似，一个`Either`相当于带有两种可能的通用包装体，它可能是左也可能是右但永远不可能都包含。无论是左还是右都可能是任意类型。例如，如果有一个 Either 变量，变量中可以持有 String 类型数据或者 Integer 类型的其中一个`Either<String, Integer>`。

如果我们使用这个原则去处理异常，我们可以说我们的`Either`类型持有`Exception`或者另一变量。简单来说，左边是一个异常而右边是执行成功的结果。你要记住这里的右边不仅仅是指左手边也是指类似于"ok"，"good"的同义词。

在下面，将会看到一个`Either`的基本实现。既然这样，我使用`Optional`类型去获得左数据或者右数据：
```java
public class Either<L, R> {
    private final L left;
    private final R right;
    private Either(L left, R right) {
        this.left = left;
        this.right = right;
    }
    public static <L,R> Either<L,R> Left( L value) {
        return new Either(value, null);
    }
    public static <L,R> Either<L,R> Right( R value) {
        return new Either(null, value);
    }
    public Optional<L> getLeft() {
        return Optional.ofNullable(left);
    }
    public Optional<R> getRight() {
        return Optional.ofNullable(right);
    }
    public boolean isLeft() {
        return left != null;
    }
    public boolean isRight() {
        return right != null;
    }
    public <T> Optional<T> mapLeft(Function<? super L, T> mapper) {
        if (isLeft()) {
            return Optional.of(mapper.apply(left));
        }
        return Optional.empty();
    }
    public <T> Optional<T> mapRight(Function<? super R, T> mapper) {
        if (isRight()) {
            return Optional.of(mapper.apply(right));
        }
        return Optional.empty();
    }
    public String toString() {
        if (isLeft()) {
            return "Left(" + left +")";
        }
        return "Right(" + right +")";
    }
}
```

现在，你能处理你的方法去返回一个`Either`变量而非抛出一个异常。但是如果想在 lambda 的右边使用已有的方法来抛出一个检查性异常，这种方法就帮不到你。因此，我们需要在上面描述上面的`Either`类型上添加一个简洁的应用函数。

```java
public static <T,R> Function<T, Either> lift(CheckedFunction<T,R> function) {
  return t -> {
    try {
      return Either.Right(function.apply(t));
    } catch (Exception ex) {
      return Either.Left(ex);
    }
  };
}
```

通过在`Either`中添加这个静态的左方法，我们现在能简单地将抛出检查性异常的方法移出并且让它返回一个`Either`。我们回到原始问题上，现在我们能够通过一连串的 Either 流来处理而不是一个使整个流毁坏的可能出现的`RuntimeException`。

```java
myList.stream()
       .map(Either.lift(item -> doSomething(item)))
       .forEach(System.out::println);
```

这仅仅意味着我们拿回来主动权。通过使用流API的 filter 方法，我们能够简单地过滤左实例和记录它们。你也能够过滤右实例并且简单地忽略掉异常情况。不管哪种方法，你都能重新掌握控制权并且你的流处理将不会被一个可能发生运行时异常导致立刻中止了。

因为`Either`是一个泛型包装器，它能被运用在任意类型当中，而不是只局限于异常处理。这给到我们机会去处理事情而并不仅仅是将异常包装到`Either`的左部分。我们现在遇到的问题是，如果`Either`仅仅保存封装的异常，并且我们会因为失去原来的数据而不能做一个重试。通过使用`Either`的能力来保存所有数据，我们能够存储异常以及在左部的数据变量。为了达成目的，我们简单地创造一个二次静态 lift 方法。

```java
public static <T,R> Function<T, Either> liftWithValue(CheckedFunction<T,R> function) {
  return t -> {
    try {
      return Either.Right(function.apply(t));
    } catch (Exception ex) {
      return Either.Left(Pair.of(ex,t));
    }
  };
}
```

可以看到在这个带有 `Pair`类型`liftWithValue`是用于把异常和原始值组成已对放到`Either`的左部。现在，如果有异常产生我们能够得到我们想要的所有信息，而并不是仅仅只有异常。

这里使用的`Pair`类型是另一个泛型类型是在 Apache 的 commons lang 库中，或者读者你们自己可以实现一个。无论如何，这相当于一个可以持有两个数值的类型。

```java
public class Pair<F,S> {
    public final F fst;
    public final S snd;
    private Pair(F fst, S snd) {
        this.fst = fst;
        this.snd = snd;
    }
    public static <F,S> Pair<F,S> of(F fst, S snd) {
        return new Pair<>(fst,snd);
    }
}
```

通过使用`liftWithValue`，现在使用在 lambda 内部中会抛出异常的方法就变得更加灵活和可控制了。当`Either`在右部时，这时可以准确地运行并且将结果提取出来。在另一方面，如果`Either`在左方，我们能知道是在哪里出现错误并且可以获取异常及原数值，这样我们就可以按我们的想法去处理事情了。通过使用`Either`类型代替将检查性异常包装到运行时异常中，我们就能够防止流中途中断了。

### Try
有些开发人员在处理异常的时候，例如 Scala，会使用`Try`来代替`Either`。`Try`类型和`Either`类型非常相似。一样地，它也有两种情况：“成功”（success）或者“失败”（failure）。failure 只能保存异常类型，success 能够保存所有你想存放的类型。所以，`Try`类型只不过是左方（failure）适配为异常类型的`Either`的一种具体实现罢了。

```java
public class Try<Exception, R> {
    private final Exception failure;
    private final R succes;
    public Try(Exception failure, R succes) {
        this.failure = failure;
        this.succes = succes;
    }
}
```

部分开发者认为这个是很容易使用，但是因为`Try`只能在 failure 部分控制异常本身，所以我认为还是会遇到在`Either`章节中第一部分的相同问题。我本人比较喜欢`Either`的灵活性。无论如何，当你使用`Try`或者`Either`，你都能解决异常处理中最初遇到的问题并且可以让流能够不被运行时异常中断。

### 库
无论是`Either`还是`Try`，都是非常容易开发者自己去实现。在另一方面，你也能够那些可用的功能性库。例如，[VAVR](http://www.vavr.io/)（原名为  Javaslang）已经将两种类型都实现了并且有可用的辅助方法。我非常建议读者们能够浏览一下它不仅局限于这两种类型。然而，你必须思考是否真的需要这个庞大的三方库，尤其只是用来处理那些通过少量的代码就可以实现的异常处理。

### 总结
当你想在 lambda 中使用一个抛出检查性异常的方法，你必须额外做一些事情。通过将其包装到`RuntimeException`中是其中一种解决方法。如果读者你更喜欢这种方法，我强烈推荐你们去编写一个简单的包装工具并且重用它，这样你就不需要为了每次去处理`try/catch`而烦恼了。

如果你想获得更多的主动权，你能够使用`Either`或者`Try`类型去包装方法的输出，因此你能够将它处理成一段数据中。在它们的帮助下，流不会再因异常的抛出而中断并且你也能按照你的意愿去处理流中的数据了。

### 译者总结
这篇文章，译者第一次快速浏览的时候以为翻译难度并不大。但是在开始后，发现里面出现的生词以及难懂的语句特别多，也因此用了比较长的时间来处理。所以如果里面有不通顺或者和原文不对称的地方，希望读者能够在下方评论指出，译者我也会通过这来提高自己的能力。

说会到文章本身，译者在写流相关的代码的时候，对异常处理并没有太过在意，读了这篇文章，发现也可以通过函数式编程的方法解决，相信对读者应该也会有所帮助，那么我们下次见。

 _小喇叭_ 

**广州芦苇科技Java开发团队**

芦苇科技-广州专业互联网软件服务公司

抓住每一处细节 ，创造每一个美好

关注我们的公众号，了解更多

想和我们一起奋斗吗？lagou搜索“ **芦苇科技** ”或者投放简历到 **server@talkmoney.cn**  加入我们吧

![](https://user-gold-cdn.xitu.io/2018/12/19/167c57a3e3d84dd5?w=640&h=356&f=gif&s=1273445)

关注我们，你的评论和点赞对我们最大的支持
