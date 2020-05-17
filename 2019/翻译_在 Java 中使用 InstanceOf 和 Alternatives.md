# 翻译 | 在 Java 中使用 InstanceOf 和 Alternatives

> 原文自国外技术社区dzone，作者为 John Vester，[传送门](https://dzone.com/articles/using-instanceof-and-alternatives-within-java)

在查看客户的代码库的时候，我发现在开发过程中存在将 JSON 数据映射到 POJO（简单 java 对象） 的集合中的情况。在 JSON 当中的一部分包含着 `List<Base>` —其中 Base 是一个自定义 java 父类，是用于存储所有被该元素维护的，属于简单 `List` 一部分的公共属性。

为了更容易理解，让我们假设以以下的对象来扩展 `Base` 对象：

- Circle
- Rectangle
- Triangle

在上述三个类中，每个类下所有的属性都是不同的而且并不与其余两个类共享。而与之相反的是，这些属性是 `Base` 类的一部分。

当我们在处理 `List<Base>` 时，我们需要处理以下的逻辑：

```java
private void processBaseObjects(List<Base> baseObjects) throws InvalidBaseException {
   for (Base base : baseObjects) {
      if (base instanceof Circle) {
         handleCircle(base);
      } else if (base instanceof Rectangle) {
         handleRectangle(base);
      } else if (base instanceof Triangle) {
         handlerTriangle(base);
      } else {
         // Throw some custom exception here
         throw new InvalidBaseException();
      }
   }
}
```

上述方法中有两件事情引起了我的兴趣：

- 使用多个 `else if` 的做法
- 使用 `instance of` 来处理对策抉择

在当我去使用谷歌来在这种情况下有没有其他更好的方法来处理的时候，我对在这其中的一些级别较高的讨论感到很震惊。

以下是这个假想当中一些有效的替代方法。

## 使用多态性

我们可以尝试将 `Circle`, `Rectangle`, and `Triangle` 类都实现同一个接口，这个接口的定义如下所示：

```java
public interface BaseProcessor {
void handleObject();
}
```

由此拓展，作为例子的 `Circle` 类可以更新为以下的形式：

``` java
public class Circle extends Base implements BaseProcessor {
@Overload
handleObject() {
// 在这里处理相关的逻辑 ...
}
}
```

在这个点上，原始的逻辑处理就可以得到简化了，如下所示：

``` java
private void processBaseObjects(List<Base> baseObjects) throws InvalidBaseException {
   baseObjects.forEach(base -> base.handleObject());
}
```

这种做法的一个前提是，所有 POJO 是当前集合的属性。如果 `handleObject()` 逻辑需要集成一些给定的服务的话，作为例子的 `Circle` 类的复杂度就会提升 — 这其实是不能接受的。

当然，如果不允许修改 `Circle`, `Rectangle`, and `Triangle` 类的话，那么也不可能会有一种最合适的方法了。

## 使用 getClass()

另外一种做法是使用 Object 的 `getClss()` 方法来做到和  `InstanceOf` 相同的处理方式。在初始的原始代码中，可以将其更新为这样子：

``` java
for (Base base : baseObjects) {
  if (base.getClass().equals(Circle.class)) {
    handleCircle(base);
  } else if (base.getClass().equals(Rectangle.class)) {
    handleRectangle(base);
  } else if (base.getClass().equals(Triangle.class)) {
    handlerTriangle(base);
  } else {
    // Throw some custom exception here
    throw new InvalidBaseException(base.getType());
  }
}
```

当然这种做法也能通过，很重要的一点是，始终要谨记 `getClass()` 会返回运行时类对象，但并不能确定对象是否属于特定的继承层级的。

## 实现枚举

再另外的一种做法是创建像以下的枚举类：

```java
public enum ShapeActionEnum {
  Circle {
     @Override
     void handleObject() {
       // start processing logic here ...
     }
  }, 
  Rectangle {
     @Override
     void handleObject() {
       // start processing logic here ...
     }
  },  
  Triangle {
     @Override
     void handleObject() {
       // start processing logic here ...
     }
  };
  abstract void handleObject();
}
```

在这里，父类 `Base` 会包含以下在处理映射过程时会被设置进去的属性：

``` java
ShapeActionEnum shapeActionEnum;
```

结果是，原始的处理代码会被更新成以下这样：

``` java
private void processBaseObjects(List<Base> baseObjects) throws InvalidBaseException {
   baseObjects.forEach(base -> .getShapeActionEnum().handleObject());
}
```

这种做法的一个难点是，处理逻辑会被附着在枚举类上，如果需要使用到服务交互的话，也会引发以上的问题。

## 其他做法

以下的一些做法也能达到同样的效果：

- 访问者模式
- 使用 `Map<Class, Runnable>` ，当中 `Runnable` 包含用于表现映射类所需做的事的 lambda 方法
- 实现模式匹配

## 结论

对于使用 `Instanceof` 去实现业务逻辑的其他做法会有这么多有趣的想法，我个人是感到非常惊讶的。

鉴于这在 DZone 中是一个比较流行的区域，我对其余可能实现的方法十分感兴趣，希望能够让我去了解其他实现的方法。

祝你愉快！