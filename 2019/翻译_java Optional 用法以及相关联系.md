# 翻译 | java Optional 用法以及相关联系

> 原文自国外技术社区dzone，作者为 Hopewell Mutanda，[传送门](https://dzone.com/articles/java-8-optional-usage-and-best-practices)

根据 [oracle 的文档](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)，Optional 是一个可包含非空值得容器对象。它在 java8 中被引入用于缓解 `NullPointerExceptions` 的诅咒。事实上，Optional 是一个包含其他对象引用的包装类。在这情况下，对象只是指向内存位置的指针，并且可以不指向任何位置。它的另一个作用是提供一种类型级别的解决方法来表示可选值不是空引用。

## Optional 的前生

在 java8 前，程序员会返回 null 而不是 Optional。这种做法有几个缺点。第一是这不是一种明确的方法来表示空值为一特殊的值。相比之下，返回 Optional 是在 API 中一种清晰的表达返回中没有值。如果我们想确保我们不会获取到空指针异常，那么，如下所示，我们需要对每个引用进行显式的非空检查，并且这会产生一堆样板代码。

``` java
// Life before Optional
    private void getIsoCode( User user){
        if (user != null) {
            Address address = user.getAddress();
            if (address != null) {
                Country country = address.getCountry();
                if (country != null) {
                    String isocode = country.getIsocode();
                    if (isocode != null) {
                        isocode = isocode.toUpperCase();
                    }
                }
            }
        }
    }
```

为了简化这个过程，我们看看如何使用 Optional 类来取代，从创建和验证实例到使用它提供的不同的方法，并将它与返回相同类型的方法相结合在一起。而后者是 Optional 的真正强大的原因。

## Optional 的特点

Optional 类提供大约10个方法，我们可以使用它们来创建和使用 Optional 类并且在下面它们是怎么被使用的。

### 创建 Optional

这里有三种生成方法来创建 optional 实例。

#### 1. **static <T> Optional<T> empty()**

返回一个空的 Optional 实例。

``` java
// Creating an empty optional
Optional<String> empty = Optional.empty();
```

返回一个空的 {Optional} 实例。这个 Optional 对象不提供值。但是，在避免如果一个空对象使用 {==} 进行对比中，使用 `Option.empty()` 返回的实例来测试就显得很有用了，虽然并不能保证这个 Optional 对象是单例。在相反的情况中，使用 `isPresent()`。

####2. **static <T> Optional<T> of(T value)**

返回一个带有特定非空值得 Optional 对象。

``` java
/ Creating an optional using of
String name = "java";
Optional<String> opt = Optional.of(name);
```

这个静态方法需要一个非空参数；否则它会抛出一个 `nullpointer` 异常。所以，如果我们不知道参数是否为空时，我们应该使用下面描述的 `ofNullable` 方法。

#### 3. **static <T> Optional<T> ofNullable(T value)**

返回一个描述特定值的 Optional 对象，如果为空，则会返回一个空的 Optional 对象。

``` java
// Possible null value
    Optional<String> optional = Optional.ofNullable(name());
    private String name(){
        String name = "Java";
        return (name.length() > 5) ? name :  null;
    }
```

通过这样做，如果我们传入一个空的引用对象，它不会抛出异常，而是返回一个空的 Optional 对象。

所以，这些事动态或者手动创建 Optional 的三种方法。下一组方法是用于检查值是否存在。

#### 1. **boolean isPresent()**

如果存在值，返回 true，否则返回 false。如果包含的对象不为空则返回 true，否则返回 false。在对对象执行任何操作之前，通常会在 Optional 中调用这个方法。

``` java
//ispresent
Optional<String> optional1 = Optional.of("javaone");
if (optional1.isPresent()){   
  //Do something, normally a get
}
```

#### 2. **boolean isEmpty()**

如果存在值，返回 false，否则返回 true。这方法作为 `isPresent()` 的相反面并且只在 java11 或更高的版本中才提供。

``` java
//isempty
Optional<String> optional1 = Optional.of("javaone");
if (optional1.isEmpty()){  
  //Do something
}
```

#### 3. **void ifPresent(Consumer<? super T> consumer)**

如果存在值，则通过该值调用特定的 consumer，否则不做任何事。

如果你不熟悉 java8，那么你可能想知道：什么是 consumer？简单来说，consumer 是一种接收参数并且不返回任何值的方法。但是用 `ifPresent()` 的时候，就到达一石二鸟的效果。我们能够通过一个方法来进行值存在的检查以及执行预期的操作，像下面写的那样。

``` java
//ifpresent
Optional<String> optional1 = Optional.of("javaone");
optional1.ifPresent(s -> System.out.println(s.length()));
```

optional 类提供另一组方法来从 Optional 对象中获取值。

#### 1. **T get()**

如果此 Optional 对象中存在值，返回该值，否则抛出 `NoSuchElementException`。之前说过了，我们想得到的是存储在 Optional 中的值，我们可以通过简单地调用 `get()` 方法来得到它。然而，这个方法在返回值为空时会抛出异常；这时候就是 `orElse()` 方法发挥作用了。

``` java
//get
Optional<String> optional1 = Optional.of("javaone");
if (optional1.isPresent()){ 
  String value = optional1.get();
}
```

#### 2. **T orElse(Tother)**

如果此 Optional 对象中存在值，返回该值，否则返回其他值。

`orElse()` 方法是用于检索包装在 Optional 实例中的值。它使用一个参数作为默认值。当值存在时，`orElse()` 方法会返回包装的值，否则：

``` java
//orElse
String nullName = null;
String name = Optional.ofNullable(nullName).orElse("default_name");
```

如果这还不够，Optional 类提供另一个获取即使为空的值的方法，叫做 `orElseGet()`。

#### 3. **T orElseGet(Supplier<? extends T> other)**

如果此 Optional 对象中存在值，返回该值，否则滴啊用 other 方法并且返回调用的结果值。

`orElseGet()` 方法和 `orElse() ` 方法有点相像。但是如果不存在 Optional 值，则不会返回值，而是使用 supplier 函数式接口，这个接口会被调用并且返回调用的值。

``` java
//orElseGet
String name = Optional.ofNullable(nullName).orElseGet(() -> "john");
```

所以，`orElse()` 和 `orElseGet()` 有什么不同呢。

第一眼看，似乎这俩方法有相同的效果。但是事实并非如此。让我们创建一些例子来看看两者之间的相似性和差异性。

首先，让我们看看对象为空时他们的行为：

``` java
String text = null;
String defaultText = Optional.ofNullable(text).orElseGet(this::getDefaultValue);
defaultText = Optional.ofNullable(text).orElse(getDefaultValue());
public String getDefaultValue() {
    System.out.println("Getting Default Value");
    return "Default Value";
}
```

在上面例子中，我们将一个空文本包装在 Optional 对象中，并且试着使用这两个方法来获取包装值，返回如下：

``` java
Getting default value...
Getting default value...
```

在两种情况下都会调用默认方法。刚好在包装值不存在的情况下，`orElse()` 和 `orElseGet()` 都会以相同的方法工作。

现在，我们进行另一个值存在的测试，理想情况下默认值是不会被创建的。

在这个简单的示例中，创建默认对象不会花费大量的成本，因为 JVM 知道如何处理这个问题，但是，当像 `default` 的默认方法进行 web 服务调用或者查询数据库操作时，消耗成本会变得非常明显。

## 使用 Optional 的最佳实践

像编程语言的其他功能一样，它可以被正确利用，也有可能被滥用。为了了解 Optional 的最佳使用方法，需要了解一下内容：

### 1. 需要处理什么问题

Optional 尝试通过添加构建更具表现力的 API 的可能性来减少 java 系统中的空指针异常的数量，这些 API 诠释返回值丢失的可能性。

如果一开始就有 Optional，那么大多数库和应用能够更好地处理返回值缺失、减少空指针异常数目以及一系列 bug 的问题了。

### 2. 不需要处理什么问题

Optional 并非一种能够避免所有空指针情况的机制，譬如方法和构造函数的强制输入参数仍然需要被校验。

就像当使用 null 的时候，Optional 并不能帮助去传达缺失值的意义。类似的方式，null 能够表示许多不同的意思（未找到值等等），同样 absent Optional 也可以做到。

当然，为了正确地使用它，方法的调用者仍然需要检查方法的 javadoc 用以理解 absent Optional 的意义。

此外，类似的方法可以在空块中捕获已检查异常，没什么事可以阻止调用者调用 `get()` 方法。

### 3. 什么时候使用它

Optional 的预期作用**主要是作为返回类型**。获取这种类型的实例后，如果存在时，你可以提取值或者不存在时为它提供一个替代的行为。

Optional 类中的一个非常有用的使用例子是将它与流或者其他返回 Optional 值得方法结合用以构建流API。参阅下面的代码段。

``` java
User user = users.stream().findFirst().orElse(new User("default", "1234"));
```

### 4. 什么时候不使用它

a) 不要将它作为一个类的字段，因为它是不可序列化的

如果确实需要序列化包含 Optional 值的对象，jackson 库提供将 Optional 作为 普通对象的支持。这意味着 jackson 将空对象处理为 null 以及将具有值的对象视为包含该值得字段。这个功能可以在 jackson-modules-java8 项目中找到。

b) 不要将它作为构造函数和方法的参数，因为它会导致产生不必要的复杂代码。

``` java
User user = new User("john@gmail.com", "1234", Optional.empty());
```

## 最后的想法

这是一个基于值得类；在 Optional 的实例中使用身份敏感操作（包括引用相等（==）、标识 hashcode或者同步）可能会产生不可预测的结果，应该尽量避免。

希望可以帮到你们！