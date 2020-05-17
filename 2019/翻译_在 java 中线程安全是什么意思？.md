#翻译 | 在 java 中线程安全是什么意思？

> 原文自国外技术社区dzone，作者为 Thomas Krieger，[传送门](https://dzone.com/articles/what-does-thread-safety-mean-in-java)

![](http://pic.mintrumpet.fun/blog/20191208175544.png)

java 的线程安全意味着一个类中的方法是原子性的或者是静态的。那么原子性和静态它们是什么呢？以及为什么在 java 中没有其他类型的线程安全方法？

## 怎么才叫做原子性？

当方法调用时立刻生效，那么我们就称这个方法为原子的。所以，其他线程只能感知到调用前或者调用后的状态，它们是获取不到中间态的。让我们看一下一个非原子性方法来对比下原子方法是如何保持线程安全的。

``` java
public class UniqueIdNotAtomic {
    private volatile long counter = 0;
    public  long nextId() { 
        return counter++;   
    }   
}
```

 `UniqueIdNotAtomic` 类通过使用 volatile 变量 counter 来创建唯一 id。我在第二行使用一个 volatile 域来确保线程每次看到的是当前的值。为了验证这个类是线程安全的，我们使用如下的测试：

``` java
public class TestUniqueIdNotAtomic {
    private final UniqueIdNotAtomic uniqueId = new UniqueIdNotAtomic();
    private long firstId;
    private long secondId;
    private void updateFirstId() {
        firstId  = uniqueId.nextId();
    }
    private void updateSecondId() {
        secondId = uniqueId.nextId();
    }
    @Test
    public void testUniqueId() throws InterruptedException {    
        try (AllInterleavings allInterleavings = 
                new AllInterleavings("TestUniqueIdNotAtomic");) {
        while(allInterleavings.hasNext()) { 
        Thread first = new Thread( () ->   { updateFirstId();  } ) ;
        Thread second = new Thread( () ->  { updateSecondId();  } ) ;
        first.start();
        second.start();
        first.join();
        second.join();  
        assertTrue(  firstId != secondId );
        }
        }
    }
}
```

为了测试这个计数器是否线程安全，我们需要两个线程，它们分别定义在16和17行。我们在18和19行开启这两个线程，并且，我们使用 join 来等待线程结束。在两条线程结束之后，我们在最后一行检查两个 id 是否相同。x

为了测试线程交错性，我们将完整的测试放到一个 while 循环当中，使用 vmlens 中的  `AllInterleavings` 类来遍历所有的线程。

运行这个测试类，最后得到以下的错误提示：

``` 
java.lang.AssertionError: 
    at org.junit.Assert.fail(Assert.java:91)
    at org.junit.Assert.assertTrue(Assert.java:43)
```

其实这个错误的原因是，因为操作 ++ 并不是原子性的，这两个线程可以覆盖相互的结果。我们可以在 vmlens 上看到如下的报告：

![](http://pic.mintrumpet.fun/blog/20191208210338.png)

在这个错误中，两个线程都是并行地获取到变量值，并且，它们都同时创建了相同的 id。为了解决这个问题，我们可以使用同步块（synchronized）来使这个方法原子化：

``` java
private final Object LOCK = new Object();
public  long nextId() {
  synchronized(LOCK) {
    return counter++;   
  } 
}
```

现在，该方法是原子性的。同步块中能够确保其他线程捕获不到这个方法的中间态。

无法共享状态的方法是默认为原子性的。对于只有只读状态的类中同样如此。因此，无状态和不可变类是实现线程安全的最简单的方法。所有这些方法都是默认为原子性的。 

但并非所有创建原子方法都是默认线程安全的。为相同值组合多个原子方法通常会导致争用条件出现。让我们试一下编写从 `ConcurrentHashMap` 中获取和存放的方法，来思考下为什么会这样。我们来尝试使用这些方法来在 map 中插入未映射过的值：

``` java
public class TestUpdateTwoAtomicMethods {
    public void update(ConcurrentHashMap<Integer,Integer>  map)  {
            Integer result = map.get(1);        
            if( result == null )  {
                map.put(1, 1);
            }
            else    {
                map.put(1, result + 1 );
            }   
    }
    @Test
    public void testUpdate() throws InterruptedException    {
        try (AllInterleavings allInterleavings = 
           new AllInterleavings("TestUpdateTwoAtomicMethods");) {
        while(allInterleavings.hasNext()) { 
        final ConcurrentHashMap<Integer,Integer>  map = 
           new  ConcurrentHashMap<Integer,Integer>(); 
        Thread first = new Thread( () ->   { update(map);  } ) ;
        Thread second = new Thread( () ->  { update(map);  } ) ;
        first.start();
        second.start();
        first.join();
        second.join();  
        assertEquals( 2 , map.get(1).intValue() );
        }
        }
    }   
}
```

这个测试和先前的很相似。同样的，我们使用两个线程来测试我们的方法是否线程安全。并且同样的，在线程结束时我们去检测结果的正确性。运行测试，我们会看到如下的错误：

```
java.lang.AssertionError: expected:<2> but was:<1>
    at org.junit.Assert.fail(Assert.java:91)
    at org.junit.Assert.failNotEquals(Assert.java:645)
```

错误的原因是在这两个原子方法的结合中，获取和存放并不是原子的。这个两个线程也能覆盖彼此的结果。我们可以在 vmlens 上看到下面的结果：

![](http://pic.mintrumpet.fun/blog/20191208214530.png)

在这个错误中，两个线程并行获取值，然后，两者都产生相同的值并且存放在 map 当中。为了解决这个争用问题，我们需要使用一个方法而非两个。在这个例子中，我们使用单一的方法来计算和获取、存储：

``` java
public void update() {
  map.compute(1, (key, value) -> {
    if (value == null) {
        return 1;
    } 
    return value + 1;
  });
}
```

这将可以解决争用问题，因为在方法中这个计算是原子性的。虽然在 `ConcurrentHashMap` 中同一个元素的所有操作是原子性的，但是像计算大小的这种针对整个  map 的操作确实静态的。所以，让我们看看静态是什么意思？

##怎么才叫做静态（quiescent）？ 

静态的意思是我们需要确保当我们在调用静态方法的时候没有其他方法在运行。下面的例子展示了怎么使用 `ConcurrentHashMap` 静态方法 size：

```java
ConcurrentHashMap<Integer,Integer>  map = 
    new  ConcurrentHashMap<Integer,Integer>();
Thread first  = new Thread(() -> { map.put(1,1);});
Thread second = new Thread(() -> { map.put(2,2);});
first.start();
second.start();
first.join();
second.join();  
assertEquals( 2 ,  map.size());
```

通过使用 join 来等待其他线程完成任务，我们确保当我们调用 size 方法的时候没有其他线程在访问 `ConcurrentHashMap`。

size 方法使用一个在 ss  `java.util.concurrent.atomic.LongAdder`，`LongAccumulator，` `DoubleAdder` 和  `DoubleAccumulator` 都会使用的机制来避免争用情况发生。它使用数组而非使用变量来存储长度。不同的线程会更新这个数组的不同部分，从而避免争用。这个算法[在 triped64 文档中有更详细的解释](http://hg.openjdk.java.net/jdk9/jdk9/jdk/file/f398670f3da7/src/java.base/share/classes/java/util/concurrent/atomic/Striped64.java)。

静态类和方法在高争用环境中收集统计数据中非常有用。收集数据后，可以使用单个线程来整理所收集的统计信息。

## 为什么在 java 中没有其他的线程安全方法？

在理论计算机科学中，线程安全是指数据结构满足正确性标准。最常用的正确性标准是线性，这也意味着组成这个数据结构的方法是原子性的。

对于常见的数据结构，都可以证明为先行并发的数据结构，参考这本书 [Maurice Herlihy 和 Nir Shavit 的 The Art of multiprocessor programming](https://www.elsevier.com/books/the-art-of-multiprocessor-programming-revised-reprint/herlihy/978-0-12-397337-5)。但是使数据结构线性化，需要一个非常需要资源的同步机制，例如 compare and swap，参考这个文献[Laws of Order: Expensive Synchronization in Concurrent Algorithms Cannot be Eliminated](https://www.cs.bgu.ac.il/~hendlerd/papers/p168-expensiveSynch.pdf)。

因此，像静态这种其他正确性标准会被探讨可用性。因此，我想问题并不是“为什么在 java 中没有其他类型的线程安全方法”，而是 java 中什么时候会有其他类型的线程安全量。

## 结论

java 中的线程安全意味着方法要么是原子性的，要么是静态的。当方法调用后立即生效，那么它是原子的。而静态是指我们需要确保在我们调用静态方法的时候，当前环境中没有其他方法在运行。

目前，静态方法仅用在数据信息收集，例如 `ConcurrentHashMap`的 size 方法。对于其他情况，使用原子方法。让我们看看，在将来是否有其他类型的线程安全方法吧。