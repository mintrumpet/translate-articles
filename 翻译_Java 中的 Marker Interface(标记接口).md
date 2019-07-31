# 翻译 | Java 中的 Marker Interface(标记接口)

> 原文自工程师baeldung博客，作者为 baeldung，[传送门](https://www.baeldung.com/java-marker-interfaces)

## 1. 综述

在这片教程中，我们将会学习关于 Java 中的标记接口。

## 2. 标记接口

标记接口是指**在内部没有定义方法及常量**的接口。它提供有关对象的运行时类型信息，因此编译器和 JVM 可以获得关于对象的额外信息。

marker interface 也称作 tagging interface。

虽然标记接口仍然在被使用，但会存在代码异味情况，需要谨慎对待。主要原因是，因为标记并没有定义任何行为，导致它会模糊接口所表示的事物。新的开发模式更喜欢使用注解来解决相同的问题。

## 3. JDK 标记接口

Java 中有许多内建的标记接口，譬如 *Serializable*，*Cloneable* 和 *Remote*。

让我们来看下关于 *Cloneable* 接口的例子吧。如果我们尝试去 clone 一个没有实现该接口的对象的时候，JVM 会抛出一个  *CloneNotSupportedException* 异常。因此，**Cloneable 标记接口是 JVM 的一个指示器**，用于我们去调用 *Object.clone()* 方法。

同样，在调用 *ObjectOutputStream.writeObject()* 方法的时候，**JVM 会检查对象是否实现了 *Serializable* 标记接口**。如果不对应，则抛出 *NotSerializableException* 异常。因此，对象并未被序列化到输出流。

## 4. 自定义标记接口

让我们创建我们自定义的标记接口吧。

例如，我们可以创建一个用于指示是否可以从数据库中删除对象的一个标记：

```java
public interface Deletable {
}
```

为了删除数据库中的一个实体，表示这个实体的对象必须实现我们的 *Deletable* 标记接口：

```java
public class Entity implements Deletable {
    // implementation details
}
```

假设我们有一个 DAO 对象，当中有一个可以从数据库中删除实体的方法。我们可以编写一个 *delete()* 方法，来使得实现我们的标记接口的对象才能够被删除：

```java
public class ShapeDao {
 
    // other dao methods
 
    public boolean delete(Object object) {
        if (!(object instanceof Deletable)) {
            return false;
        }
 
        // delete implementation details
         
        return true;
    }
}
```

像我们所看到那样，我们给于 JVM 一个指示，告诉它对于我们的对象在运行时应该表现怎么样的行为。如果对象是实现我们的标记接口的话，它能够在数据库上被删除。

## 5. 标记接口 vs. 注解

通过引入注解，Java 给我们提供了一种可以实现和标记接口相同结果的代替方法。此外，和标记接口一样，我们可以将注解应用于任何的类中，并且我们可以将它们作为指示器来执行某些操作。

那么，关键的区别在哪里呢？

和注解不同，接口允许我们**利用多态性的好处**。因此，我们可以**对标记接口添加额外的限制条件**。

例如，让我们对上述的代码增加限制条件，譬如，只允许 **Shape** 类型能够从数据库中删除：

```java
public interface Shape {
    double getArea();
    double getCircumference();
}
```

在这情况下，我们称作 *DeletableShape* 的标记接口，会和如下所示：

```java
public interface DeletableShape extends Shape {
}
```

然后我们的类会实现这个标记接口：

```java
public interface DeletableShape extends Shape {
}
```

因此，**所有的 *DeletableShape* 接口实现也同样是 *Shape* 的接口实现**。显然，这在注释中是做不到的。

然而，每个设计决策都需要权衡并且**多态性能够被当做标记接口的反参数(counter-argument)**。在我们的例子中，**每一个继承了 Rectangle 的类都会自动地继承了 *DeletableShape*。**

## 6. 标记接口 vs. 典型接口

在先前的例子中，我们可以通过修改 DAO 中的 **delete()** 方法来测试我们的对象是否为 *Shape*，而不是测试它是否是一个 *Deletable* 对象，来获得相同的结果：

```java
public class ShapeDao { 
 
    // other dao methods 
     
    public boolean delete(Object object) {
        if (!(object instanceof Shape)) {
            return false;
        }
     
        // delete implementation details
         
        return true;
    }
}
```

所以当我们能够在使用普通的接口来获得相同结果的情况下，为什么要创建一个标记接口呢？

让我们想想，除了 *Shape* 类型之外，我们还希望从数据库删除 *Person* 类型的对象的时候，我们有两个方法可以实现它：

第一种是，**在我们之前的 *delete()* 方法中添加一个额外的检查**，用来验证要删除的对象是否是 *Person* 的实例。

```java
public boolean delete(Object object) {
    if (!(object instanceof Shape || object instanceof Person)) {
        return false;
    }
     
    // delete implementation details
         
    return true;
}
```

但是如果我们还有许多其他类型希望能够从数据库中删除呢？显然，这并不是一个好的方法，因为我们必须**为了每一个新的类型修改我们的方法**。

第二种是，**让 *Person* 类实现 *Shape* 标记接口**。但是 *Peson* 对象是否真的为 *Shape* 呢？答案是并不是，因此这使得第二种选择比第一种选择更为糟糕。

因此，虽然我们**可以通过典型的接口来作为标记来达到相同的效果，但最终会得出一个糟糕的设计**。

## 7. 结论

在这篇文章中，我们讨论了什么是标记接口以及如何使用它们。然后我们看了下在 Java 内建的相同类型的接口例子并且了解在 JDK 中如何使用它们。

接着，我们创建了自己的标记接口并且将它与注解的使用作了一个对比。最后，我们通过了解在某些场景中为什么使用标记接口而非普通接口来结束这次的探究。

和往常一样，在 [GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop) 上可以找到我们的例子代码。