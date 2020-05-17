# 翻译 | Java 中的变型（Variance）

![mark](https://image.talkmoney.cn/luwei/20190414/MvxzzAn4LUHF.jpg)

> 原文自国外Java社区javacodegeeks，作者为 George Aristy，[传送门](https://www.javacodegeeks.com/2019/04/variance-java.html)

前几天，我在偶然的情况下看到一篇文章，讲述了文章作者在使用了 GO 8个多月后对其的利弊看法。在使用 GO 工作了相当长的一段时间的我来说，基本上同意作者说的点。

尽管是这种序言，但这篇文章时关于 Java 的变型的，目标是重新理解什么是变型，以及在 Java 实现中的一些细微差别。

## 什么是变型？

维基上是这样描述[变型](https://zh.wikipedia.org/wiki/%E5%8D%8F%E5%8F%98%E4%B8%8E%E9%80%86%E5%8F%98)的：

>  所谓的**变型**是指如何根据组成类型之间的子类型关系，来确定更复杂的类型之间的子类型关系。

“更复杂的类型”在这里是指诸如容器、函数等高级别的结构。因此，变型是关于由通过类型层次结构（[Type Hierarchy](https://en.wikipedia.org/wiki/Class_hierarchy)）连接的参数组成的容器及函数之间的赋值兼容。它允许参数和子类型多态性的安全集成。例如，是否可以将一个方法中返回的cat 列表赋值到类型为 “list of animals” 的变量中？我能否一将奥迪汽车的对象列表传递给一个接受 Cars 列表的方法当中？

在 Java，是定义在**使用点变型**（use-site）当中。

## 变型的4种类型

在维基中的阐述中，类型构造器指：

- **协变**（Covariant）：接受子类型不接受超类型
- **逆变**（Contravariant）：接受超类型不接受子类型
- **双变**（Bivariant）：同时接受子类型和超类型
- **不可变**（Invariant）：不接受子类型和超类型

（显然，声明的类型参数在所有情况下都是可以接受的）

## Java 中的不可变性（Invariance）

使用点变型在类型参数中必须不设定边界。

如果 `A` 是 `B` 的其中一个超类型，那么 `GenericType<A>` 并不是 `GenericType<B>` 的超类型，反之亦然。

这表示两种类型彼此没有联系，并且在任何情况下都无法转换成对方。

### 不变容器

在 Java 中，不变量可能是你遇到过的第一个，并且是最直观的泛型示例。正如所期望的，类型参数的方法是可使用的。参数类型的所有方法都是可访问的。

但它们无法互换：

```java
// 类型层级：Person :> Joe :> JoeJr
List<Person> p = new ArrayList<Joe>(); // 编译错误
List<Joe> j = new ArrayList<Person>(); // 编译错误
```

但能够添加对象：

```java
// 类型层级：Person :> Joe :> JoeJr
List<Person> p = new ArrayList<>();
p.add(new Person()); // ok
p.add(new Joe()); // ok
p.add(new JoeJr()); // ok
```

也能够读取到：

```java
// 类型层级：Person :> Joe :> JoeJr
List<Joe> joes = new ArrayList<>();
Joe j = joes.get(0); // ok
Person p = joes.get(0); // ok
```

## Java 中的协变

使用点变型必须对类型参数有一个公开的下界。

如果 `B` 是 `A` 的子类型，那么 `GenericType<B>` 是 `GenericType<? extends A>` 的子类型。

### Java 中的数组一直是协变的

在 Java 1.5 引入泛型之前，数组是唯一可用的泛型容器。它们一直具有协变性，例如，`Integer[]` 是 `Object[]` 的子类型。编译器允许你将 `Integer[]` 传递给接收 `Object[]` 的方法中。如果方法插入一个 `Integer` 的超类型， ArrayStoreException 异常会在*运行时*抛出。协变泛型类型规则在**编译时**实现了此类检查，在第一时间防止错误的发生。

```java
public static void main(String... args) {
  Number[] numbers = new Number[]{1, 2, 3, 4, 5};
  trick(numbers);
}
 
private static void trick(Object[] objects) {
  objects[0] = new Float(123);  // ok
  objects[1] = new Object();  // ArrayStoreException 在运行时抛出
}
```

### 协变容器

Java 允许子类型（协变）泛型类型，但是它根据最小惊讶原则（POLA）限制了这些泛型类型怎样做到“流入和流出”。换而言之，返回类型参数值的方法是可访问的，而具有类型参数输入参数的方法是不可访问的。

你可以将超类型替换为子类型：

```java
// 类型层级：Person :> Joe :> JoeJr
List<? extends Joe> = new ArrayList<Joe>(); // ok
List<? extends Joe> = new ArrayList<JoeJr>(); // ok
List<? extends Joe> = new ArrayList<Person>(); // 编译错误
```

从容器中*读取*也很直观：

```java
// 类型层级：Person :> Joe :> JoeJr
List<? extends Joe> joes = new ArrayList<>();
Joe j = joes.get(0); // ok
Person p = joes.get(0); // ok
JoeJr jr = joes.get(0); //
```

但不允许跨层*写入*（违反直觉），以预防数组陷阱。例如，在下面的例子中，`List<Joe>` 的调用者/拥有者会感到*惊讶*如果其他带有协变参数 `List<? extends Person>` 的方法添加一个 `Jill` 对象。

```java
// 类型层级：Person :> Joe :> JoeJr
List<? extends Joe> joes = new ArrayList<>();
joes.add(new Joe());  // 编译错误 (你不清楚哪种 Joe 的超类型在列表中)
joes.add(new JoeJr()); // 编译错误 (同上)
joes.add(new Person()); // 编译错误
joes.add(new Object()); // 编译错误
```

## Java 中的逆变

使用点变型必须对类型参数有一个公开的**上**界。

如果 `A` 是 `B` 的超类型，那么 `GenericType<A>` 是 `GenericType<? extends B>` 的超类型。

### 逆变容器

逆变容器的行为和常识相反：与协变容器相反，访问具有类型参数返回值的方法是*不可行的*，而访问具有类型参数入参的方法是可行的：

你可以将子类型替换为超类型：

```java
// 类型层级：Person :> Joe :> JoeJr
List<? super Joe> joes = new ArrayList<Joe>();  // ok
List<? super Joe> joes = new ArrayList<Person>(); // ok
List<? super Joe> joes = new ArrayList<JoeJr>(); // 编译错误
```

无法在读取时捕获特定类型：

```java
// 类型层级：Person :> Joe :> JoeJr
List<? super Joe> joes = new ArrayList<>();
Joe j = joes.get(0); // 编译错误 (能够为 Object 或者 Person)
Person p = joes.get(0); // 编译错误 (同上)
Object o = joes.get(0); // 允许，因为在 Java everything IS-A Object
```

你*可以*添加“下界”的子类型：

```java
// 类型层级：Person :> Joe :> JoeJr
List<? super Joe> joes = new ArrayList<>();
joes.add(new JoeJr()); // 允许
```

但你*不能*添加超类型：

```java
// 类型层级：Person :> Joe :> JoeJr
List<? super Joe> joes = new ArrayList<>();
joes.add(new Person()); // 编译错误
joes.add(new Object()); // 编译错误
```

## 双变类型

使用点变型必须在类型参数中声明**无界通配符**。

具有无界通配符的泛型类型是同一泛型类型的所有有界变体的超类型。例如，`GenericType<?>` 是 `GenericType<String>` 的超类型。由于无界类型是 `hierarchy` 类型的根，因此，对于它的参数类型，它只能访问继承自 `java.lang.Object` 的方法。

将 `GenericType<?>`  视为 `GenericType<Object>`。

## N型参数结构的变型

Java 允许使用协变返回类型和异常类型重写方法：

```java
interface Person {
  Person get();
  void fail() throws Exception;
}
 
interface Joe extends Person {
  JoeJr get();
  void fail() throws IOException;
}
 
class JoeImpl implements Joe {
  public JoeJr get() {} // 重写
  public void fail() throws IOException {} // 重写
}
```

但是试图用协变*参数*覆盖方法只会导致重载：

```java
interface Person {
  void add(Person p);
}
 
interface Joe extends Person {
  void add(Joe j);
}
 
class JoeImpl implements Joe {
  public void add(Person p) {}  // 重载
  public void add(Joe j) {} // 重载
 }
```

## 结语

变型为 Java 带来了额外的复杂性。虽然围绕变型的类型规则很容易理解，但是关于类型参数方法的可访问性规则是违反常识的。理解它们不仅仅要达到“显而易见” – 需要停下来来思考当中的逻辑。

然而，我的日常经验是告诉我，这些细微的差别通常都不碍事：

- 我一直没试过必须声明一个逆变参数的实例，而且我也很少遇到它们（尽管它们确实存在）
- 协变参数视似乎更常见一些，但幸运的是他们也更容易推理出来

考虑到子类型是面向对象编程中其中一种基本的技术，而变型就是其最大的一个优点。

**结论**：变型在我日常编程中提供适当的收益，特别是当需要与子类型兼容的时候（这在面向对象编程中很常见）。