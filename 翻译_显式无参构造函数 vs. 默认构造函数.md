# 翻译 | 显式无参构造函数 vs. 默认构造函数

> 原文自国外技术社区dzone，作者为  Dustin Marx，[传送门](https://dzone.com/articles/explicit-no-arguments-constructor-vs-default-const)

大多数刚接触 Java 的程序员在上手没多久都会知道，当他们并没有对类声明至少一个显式的构造函数的时候，一个隐式的"默认构造函数"会被创建（由 Javac）出来。Java 语言规范中的[8.8.9章节](https://docs.oracle.com/javase/specs/jls/se12/html/jls-8.html#jls-8.8.9)中简洁地指出，"如果一个类没有声明构造函数，则隐式声明默认构造函数。"这章节还进一步介绍了隐式创建默认构造函数的特性，没有参数、没有 `throws` 子句、并且调用其超类中同样不含参数的构造函数。Java 开发人员可以选择显式实现类似于默认构造函数的无参构造函数（例如不接受参数并且没有 `throws` 子句）。在这篇文章中，我会去探寻一下开发者会决定实现显式无参构造函数而非依赖隐式默认构造函数的原因。

## 显式声明无参构造函数的几个原因

### 阻止类的实例化

实现显式无参构造函数的一个最常见的原因是阻止使用 `public` 访问性的隐式默认构造函数。如果类中有其他接受参数的显式构造函数的话，实现显式无参构造函数是没必要的，因为任何显式构造函数的存在都会阻止生成隐式默认构造函数。然而，如果并没有其他显式的构造函数的话（例如在所有都为静态方法的 "utility" 类中），通过实现使用 `private` 访问的显式无参构造函数，可以阻止隐式默认构造函数的产生。Java 语言规范中的[8.8.10章节]([Section 8.8.10](https://docs.oracle.com/javase/specs/jls/se12/html/jls-8.html#jls-8.8.10))中指出，使用所有私有的显式构造函数可以阻止类的实例化。

### 强制通过 Builder 或者静态初始化工厂来实例化类

另一个显式声明无参 `private` 构造函数的原因是强制通过 Builder 或者静态初始化工厂来实例化类对象。[Effective Java](http://marxsoftware.blogspot.com/2018/01/new-in-effective-java-3.html)（第三版）的前两项概述了使用静态初始化工厂方法和 Builder 而不是直接使用构造函数来实现实例化的优点。

### 多个构造函数，并且需要一个无参的构造函数

比起上述原因，相同程度或者更常见的原因去实现无参构造函数是，和那些需要参数的构造函数一样，无参构造函数也是需要被声明的。在这种情况下，由于存在存在带参数的构造函数，一个无参的构造函数必须被显式声明，因为一个包含一或多个显式构造函数的默认构造函数从来不会在类中被隐式声明。

### 使用 Javadoc 构造的文档对象

另一个显式声明无参构造函数而非依赖隐式默认构造函数的原因是需要再构造方法里声明 Javadoc 文档注释。这是 [JDK-8224174](https://bugs.openjdk.java.net/browse/JDK-8224174) 的陈述理由（"java.lang.Number 有一个默认构造函数"），并且现在是 JDK13 的一部分，也在当前未处理的  [JDK-8071961](https://bugs.openjdk.java.net/browse/JDK-8071961) 有所阐述（创建默认构造函数时添加 javac lint警告）。最近编写的 CSR [JDK-8224232](https://bugs.openjdk.java.net/browse/JDK-8224232) （"java.lang.Number 有一个默认构造函数"）详细地阐述了这一点：默认构造函数不适用于 *well*-*documented* API 中。

### 比起隐式更喜欢显式

某些开发者普遍更喜欢显式声明而非隐式创建。在 Java 中的不少领域需要对显式声明或者隐式创建进行一个选择。如果他们更重视交互层面或者假定显式构造函数具有更高的可读性，或许比起隐式构造函数，开发人员更喜欢显式声明无参构造函数。

## 在 JDK 中使用显式无参构造函数代替默认构造函数

在 JDK 的一些案例中，隐式默认构造函数已经被显式无参构造函数取缔了。包括以下这些内容：

- 在 JDK9 中被解决了的 [JDK-8071959](https://bugs.openjdk.java.net/browse/JDK-8071959) （java.lang.Object 使用隐式默认构造函数），使用一个显式的无参构造函数替换了 java.lang.Object 的"隐式默认构造函数"。在阅读问题的"描述"时我笑出了声，"在修改 java.lang.Object 的文档描述时（[JDK-8071434](https://bugs.openjdk.java.net/browse/JDK-8071434)），突然注意到类并*没有*显式的构造函数，而是依赖 javac 来创建隐式默认构造函数，真尴尬！"
- 在 JDK9 中被解决了的 [JDK-8177153](https://bugs.openjdk.java.net/browse/JDK-8177153) （"LambdaMetafactory 具有默认构造函数"），使用 `private` 修饰的显式无参构造函数取代了隐式默认构造函数。
- 计划实施于 JDK13 的 [JDK-8224174](https://bugs.openjdk.java.net/browse/JDK-8224174) （"java.lang.Number 有一个默认构造函数"），将会使用显式无参构造函数取代 java.lang.Number 的隐式默认构造函数。

## 关于默认构造函数中潜在的 javac lint 警告

有一天 javac 可能会使用可用的 lint 警告来指出带有默认构造函数的类，当前不针对任何 JDK 版本的 [JDK-8071961](https://bugs.openjdk.java.net/browse/JDK-8071961) （"创建默认构造函数时添加javac lint警告"）指出："[JLS 8.8.9章节](https://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.8.9) 描述到，如果类没有至少声明一个构造函数，编译器会默认创建一个构造器。虽然这个策略十分方便，但是对于那些正式的类文件，这是一个糟糕的编码习惯，如果没有其他原因，默认构造函数将不会有 Javadoc。使用默认构造函数可能是一个 javac lint 警告。"

## 结论

依赖在编译过程中创建的的默认构造函数确实很方便，但是有一些情况，即使显式声明并非是必须的，声明一个显式的无参构造函数会更利于扩展。

