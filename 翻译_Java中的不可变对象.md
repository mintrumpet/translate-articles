# 翻译 | Java 中的不可变对象

![mark](https://image.talkmoney.cn/luwei/20190407/vjx2l2Js1IYd.jpg?imageslim)

> 原文自国外技术社区dzone，作者为  Yogen Rai ，[传送门](https://dzone.com/articles/java-immutable-objects)

如果对象的状态在构造后不能更改，那么这个对象就被认为是*不可变的*。因为它们不能改变状态，所以它们不能被线程所干扰破坏，也不能再不一致的状态下观察到，这使得在并发编程中他们发挥了重要作用。

例如，字符串对象在 Java 中是不可变的。

``` java
import org.junit.Test;
import static org.junit.Assert.assertEquals;
public class ImmutableTest {
    @Test
    public void testImmutableObjs() {
        String firstName = "yogen";
        String lastName = "rai";
        // concatenate two strings
        String fullName = firstName.concat(" ").concat(lastName);
        assertEquals("yogen", firstName);
        assertEquals("rai", lastName);
        assertEquals("yogen rai", fullName);
    }
}
```

对 firstName、" "（空格）和 lastName 的连接并不会修改到它们的其中一个，而是创建一个通过 fullName 引用的新对象 。

![mark](https://image.talkmoney.cn/luwei/20190407/LR7Vf9KL6hKL.png?imageslim)

## 自定义不可变对象

下面的规则定义了一个创建不可变对象的简单策略：

1. 不要提供 "setter"方法 — 它能够修改字段或者通过字段引用的对象
2. 定义所有字段为 "final" 和 "private"
3. 不允许子类重载方法。做到这点最简单的方法是将类定义为 final。更复杂的方法是将构造函数定义为 private，并且通过工厂方法构造实例
4. 如果实例字段包含对可变对象的引用，则不允许更改这些对象：
   1. 不要提供可以修改可变对象的方法
   2. 不要共享引用给可变对象。永远不要对传递到构造函数的外部不可变对象存储引用；如果需要，创建副本，并存储对副本的引用。类似的，在必要时创建内部可变对象的副本，以避免在方法中返回原始对象

## 通过 Date 对象自定义不可变对象

在 java.util.* 中的 Date 对象不是线程不可变的。

这是个好消息，在 JDK8 后新的 java.time.* API，将 Date 对象作为不可变对象。例如，在下面的例子中，即使属性 `dateJoined` 是 Date 对象，Student 对象也是不可变的。

``` java
import java.time.LocalDate;
final class Student {
    private final String name;
    private final int age;
    private final LocalDate dateJoined;
    public Student(String name, int age, LocalDate dateJoined) {
        this.name = name;
        this.age = age;
        this.dateJoined = dateJoined;
    }
    public String getName() {
        return name;
    }
    public int getAge() {
        return age;
    }
    public LocalDate getDateJoined() {
        return dateJoined;
    }
}
public class TestImmutable {
    @Test
    public void testImmutableObject() {
        Student original = new Student("Yogen", 23, LocalDate.of(2016, 5, 1));
        LocalDate modifiedLocalDate = original.getDateJoined().plusYears(2);
        Student expected = new Student("Yogen", 23, LocalDate.of(2016, 5, 1));
        assertEquals(expected, original);
    }
}
```

## 不可变对象的好处

- 因为没有副作用，不可变对象更容易构造、测试和使用
- 因为对象上的修改不会影响它的原始状态，不可变对象更容易缓存
- 真正的不可变对象总是线程安全的
- 避免了身份可变性问题
- 它们有助于时空耦合（temporal coupling，二个动作只因为同时间发生，就被包装在一个模块中）