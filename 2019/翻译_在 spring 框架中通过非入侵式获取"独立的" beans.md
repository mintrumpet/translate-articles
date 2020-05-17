# 翻译 | 在 spring 框架中通过非入侵式获取"独立的" beans

> 原文自国外技术社区dzone，作者为 Michael Gantman，[传送门](https://dzone.com/articles/non-intrusive-access-to-quotorphanedquot-beans-in)

这篇文章我想和大家讨论的是如何不通过使用 `BeanFactory`， `AbstractApplicationContex ` 这些 spring 特殊类或者他们的扩展（也就是秉承 spring 的非侵入式原则），来获取那些没有注入到其他 bean 中的 spring beans。

## 问题描述

spring 其中的一个主要功能是依赖注入（DI）。另一个有相同功能的名字称为控制反转（IOC）。spring 是其中最早期（如果不是第一个的话）使用这个概念的框架之一。在大多情况下，使用这个模式非常简单。通常，你编写一个类，并将其定义为 spring bean（或者 component），而从现在开始，你可以将其作为其他被定义为 bean 的类的成员之一并注入进去。让我们看一下一个示例，假设你正在处理一个格式化文本文档的功能，你定义了一个具有如下方法的类（让我们称它为 `TextFormatter`）：

``` java
 public String formatText(String text);
```

这个类被定义为 bean 并且可以被注入到其他需要这个功能的 bean 当中。到目前还好，让我们将情况稍微复杂化一下。假设你需要对不同的文档进行不同的格式化逻辑，因此，我们的 `TextFormatter` 会成为一个接口，并且我们会编写不同的诸如 `CustomerLetterTextFormatter`，`InternalDocumentTextFormatter`，`LegalTextFormatter` 的实现，以及之后还会添加其他可能存在的实现。在这种情况下，那些需要 `TextFormatter` 的类都会持有类型为 `TextFormatter` 的成员变量，但是我们不会注入任何 bean，因为我们需要在运行时通过参数来判断应该需要哪种实现。所以很明显我们需要创建带有其他方法的 `TextFormatterFactory` 来实现这个需求：

``` java
 public static TextFormatter getTextFormatterInstance(String type)
```

通常，这个方法的实现会使用到像 `BeanFactory` 或者  `AbstractApplicationContext` 这种 spring 特殊类来检索所需的，也就是实现了 `TextFormatter` 的 bean。这是一个完全可行的方法。然而，这确实违反了 spring 的非倾入性原则，所以我比较建议使用一种不通过 spring 特殊类来获取 bean 的方法。

## 解决方法

基本上，我希望 `TextFormatter` 的每个实现都通过某种方式可以将其自身接入到 `TextFormatterFactory` 中，那么当需要提取实例时，`TextFormatterFactory` 不需要使用 spring 特殊类来获取这些 beans 了，因为它们已经通过某种神奇的方法将其接入到工厂对象当中了。诀窍就是：spring 框架在初始化的期间会实例化所用 bean。换而言之，在初始化过程中，spring 会调用每一个定义为 spring bean 的类的构造方法。对了 — 这就是解决这个问题的关键。以下让我们看看如何实现它：

首先，`TextFormatterFactory` 应该持有一个类型为 `Map <String，TextFormatter>` 的静态成员，它是用于保存和查找所有 `TextFormatter` 的实现。`TextFormatterFactory` 通常也应该有这么一个方法：

``` java
public static void addTextFormatter(String name, TextFormatter textFormatter)
```

这是一个方法，让我们的 `TextFormatter` 实现可以将它们插入到工厂对象当中（当然，这个方法会将其中的 `TextFormatter` 参数添加到上述提到的 map 当中）。

现在，让我们添加一个新类吧，创建一个实现 `TextFormatter` 接口的抽象类 `BaseTextFormatter`。所有关于 `TextFormatter` 的具体实现都会继承 `BaseTextFormatter` 类，并且其中的诀窍是：在 `BaseTextFormatter` 的构造函数中，我们会调用一下方法：

``` java
TextFormatterFactory.addTextFormatter(this.getClass().getSimpleName(), this)
```

这样做是为了将每一个 `TextFormatter` 的具体实现都可以插入到工厂对象当中。请记住每一个具体实现都被定义为 bean，并且像我们所说的，spring 在初始化期间，会调用每一个被定义为 bean 的构造函数，这就是方法的关键了。所以现在，当 spring 初始化完成后，`TextFormatterFactory` 的 map 当中就会存有所有的具体实现了。在类中如果我们需要用到 `TextFormatter`，可以通过简单调用来实现：

``` java
TextFormatterFactory.getTextFormatterInstance(LegalTextFormatter.class.getName())
```

为了获取 `LegalTextFormatter` 的实现或者其他具体实现对象，最重要的是这个方法的实现，如下：

``` java
TextFormatterFactory.getTextFormatterInstance(String name)
```

这将不再需要使用 spring 特殊类，仅通过简单地从 map 当中查找所需的实现就可以了。至此，我们的目标算是达成了。

## 解决方法的实现

现在我们知道怎么去解决这个问题了，让我们看看如何去实现吧。如果我们在同一个项目中有多种接口和多组关于它们的实现的时候我们该怎么办？我们是否需要遵循原则，使用同样的方法来实现所有接口呢？

好吧，确实需要。但是很显然的是，会存在相当的代码重复，这明显不是一件好事。所以，我实现了一些解决的基础结构并且允许使用者能够扩展基础结构的基类用于接收其他常用的功能实现。

这个包里只含有两个类：`BaseEntityFactory` 和`BaseEntity`。它们包含上述的逻辑实现以及使用者能够添加接口和实现接口的基类、扩展 `BaseEntity`、扩展实现 `BaseEntityFactory` 的 `factory` 以及无需任何代码重复来获取上述的所有功能。

这个实现现在在开源库 **MgntUtils** 中可以找到，以下是解释库中的可用工具以及如何获取的文章，链接：[Open-source MgntUtils library](https://community.oracle.com/blogs/michaelgantman/2016/01/26/open-source-java-library-with-stacktrace-filtering-silent-string-parsing-and-version-comparison)。

最后，这个库在 Maven Central 中可以找到。以下是 Maven artifacts 的信息。版本1.5.0.2是撰写本文时的最新版本，但在不远将来会被更新。需要得知最新的版本，在 [http://search.maven.org/](http://search.maven.org/) 中搜索 'MgntUtils' 吧。

Happy coding！