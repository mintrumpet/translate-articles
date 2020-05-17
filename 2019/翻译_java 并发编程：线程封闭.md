# 翻译 | java 并发编程：线程封闭

> 原文自国外技术社区dzone，作者为 Akshansh Jain，[传送门](https://dzone.com/articles/java-concurrency-thread-confinement)

读者你们好！在这篇文章中，我将和大家探讨什么叫线程封闭（thread confinement），并且如何实现它。现在就让我们直奔主题吧。

## 线程封闭

大多数并发问题仅发生在我们想在多线程间共享可变变量或者可变状态时。如果在多个线程中共享可变状态，那么所有线程都能够读取并且修改状态中的值了，从而导致产生错误或者意想不到的结果。避免此问题发生的其中一个方法是简单地不再对线程间共享数据。这种技术称为线程封闭，是在我们的应用中实现线程安全的最简单的方法之一。

java 语言本身并没有强制执行线程封闭的方法。线程封闭是通过不允许多个线程去使用同一个状态的方式来实现，因此是由实现来强制执行。如下所述，这里有几种线程封闭的方法。

### Ad-Hoc（临时的） 线程封闭

临时线程封闭是这么一种方式，通过应用的开发者，明确大家的责任来确保对象的使用仅限于单个线程。这种方法异常的脆弱，并且在大多情况下是避免使用这种方法的。

其中一个使用临时线程封闭的特例是在 volatile 变量上。只要确保 volatile 变量仅从单个线程写入，就可以安全地对共享 volatile 变量进行读 — 修改 — 写操作。在这种情况下，你需要限制单个线程的修改来防止竞争关系的发生，并且 volatile 变量的可见性保证确保其他线程读取到的是最新的值。

### 堆栈封闭

堆栈封闭是通过将变量后者对象拷贝到线程的堆栈中从而达到封闭。它比临时线程封闭要健壮得多，因为它通过定义堆栈本身中的变量状态来进一步限制对象的范围。例如，思考下下面的代码：

``` java
private long numberOfPeopleNamedJohn(List<Person> people) {
  List<Person> localPeople = new ArrayList<>();
  localPeople.addAll(people);
  return localPeople.stream().filter(person -> person.getFirstName().equals("John")).count();
}
```

在上面的代码中，我们传递了 person 的列表参数进方法但并没有直接使用它。相反，创建了方法的 list `localPeople`，是当前线程的本地列表，并将参数中的 `people` 数据添加到 `localPeople`。由于我们仅在 `numberOfPeopleNamedJohn` 方法中定义列表，因此使得 `localPeople` 受到堆栈限制，因此只存在于单个线程的堆栈中，其他线程无法访问。这使得 `localPeople` 变得线程安全。我们唯一需要注意的是，我们不应当允许 `localPeople` 离开当前的方法区，用以保持堆栈封闭限制。在定义这种变量时，应当使用文档或者注释来记录上，因为通常来说，只有当时的开发者认为不允许让它出现在方法外，但是在之后，如果没表述好，可能会给其他的开发者带来困扰。

### ThreadLocal

`ThreadLocal` 允许开发者将每个线程与持值对象相关联。允许我们为不同的线程存储不同的对象，并且维护线程和对象之间的关系。通过定义 set 和 get 访问方法来维护每个线程使用的值得独立副本。get 方法始终返回当前执行线程传递给 set 方法的最新值。让我们看看一个例子：

``` java
public class ThreadConfinementUsingThreadLocal {
    public static void main(String[] args) {
        ThreadLocal<String> stringHolder = new ThreadLocal<>();
        Runnable runnable1 = () -> {
            stringHolder.set("Thread in runnable1");
            try {
                Thread.sleep(5000);
                System.out.println(stringHolder.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };
        Runnable runnable2 = () -> {
            stringHolder.set("Thread in runnable2");
            try {
                Thread.sleep(2000);
                stringHolder.set("string in runnable2 changed");
                Thread.sleep(2000);
                System.out.println(stringHolder.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };
        Runnable runnable3 = () -> {
            stringHolder.set("Thread in runnable3");
            try {
                Thread.sleep(5000);
                System.out.println(stringHolder.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };
        Thread thread1 = new Thread(runnable1);
        Thread thread2 = new Thread(runnable2);
        Thread thread3 = new Thread(runnable3);
        thread1.start();
        thread2.start();
        thread3.start();
    }
}
```

在上述例子中，我们使用相同的 `ThreadLocal` 对象 `stringHolder` 来执行三个线程。正如你所看到的，首先我们对 `stringHolder` 中的每个线程设置一个字符串，使其包含三个字符串。然后，在一段时间的休眠后，我们修改了第二个线程的值，下面是程序的输出：

``` 
string in runnable2 changed
Thread in runnable1
Thread in runnable3
```

就像上面的输出所示，第二个线程的 string 值被更改，但线程1和3的字符串并不受影响。如果我们在取得  `ThreadLocal` 中特定线程的值之前并不设定任何值，它则会返回 `null`。线程终止后 `ThreadLocal` 中的线程特定对象将准备好进行GC了。

这就是所有关于线程分别的内容。希望这篇文章可以帮助到你并且能够从中获得新的知识。