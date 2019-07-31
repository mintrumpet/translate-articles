# 翻译 | 怎么在Java中替换掉繁杂的if语句

> 原文自工程师baeldung博客，[传送门](https://www.baeldung.com/java-replace-if-statements)

### 1. 概述
决策结构在大多数编程语言中占据了至为重要的一步。但是我们常常会被大量的那种让代码变得难读且难维护的内嵌if语句搞得浑身难受。

在这次的教程中，我们将来过一下可以代替内嵌if语句的各种方法。让我们来探索简化我们代码的途径吧。

### 2. 案例学习
通常我们会遇到一些需要做一系列条件处理的业务逻辑，并且它们每一个都需要不同的处理。

为了演示，我们来看一下Calulator（计算器）类的一个例子。上面是带有两个数字类型参数，一个操作符参数以及基于操作的数值返回值的一个方法：

```java
public int calculate(int a, int b, String operator) {
    int result = Integer.MIN_VALUE;
 
    if ("add".equals(operator)) {
        result = a + b;
    } else if ("multiply".equals(operator)) {
        result = a * b;
    } else if ("divide".equals(operator)) {
        result = a / b;
    } else if ("subtract".equals(operator)) {
        result = a - b;
    }
    return result;
}
```

通常我们也能使用switch语句来操作：

```java
public int calculateUsingSwitch(int a, int b, String operator) {
    switch (operator) {
    case "add":
        result = a + b;
        break;
    // other cases    
    }
    return result;
}
```

在典型的开发过程中，本质上if语句会使程序变得更为臃肿和复杂。并且，switch语句也并非所有场景都适用，当条件复杂的时候，switch语句就没什么作用了。

另一个使用嵌套条件声明编程的影响是它使得程序变得难以管理。例如，如果我们需要新添加一个操作，我们需要添加一个新的if条件以及条件的实现。

### 3. 重构
让我们尝试下使用其他更简洁和易管理的方法来代替这个复杂的if语句吧。

#### 3.1  工厂类
很多时候我们经常遇见许多条件声明，它们是用来处理每个分支中相似的操作。这提供给我们一个想法，**提取一个返回具体类型的对象并且根据具体对象行为执行操作的工厂类**。

例如在下面，让我们定义一个带有单独的*apply*方法的*操作*接口

```java
public interface Operation {
    int apply(int a, int b);
}
```

这方法带有两个数值类型参数以及数值类型的返回。让我们来定义一个实现加法的类：

```java
public class Addition implements Operation {
    @Override
    public int apply(int a, int b) {
        return a + b;
    }
}
```

我们现在将要实现一个返回基于给定操作符的操作实例的工程类：

```java
public class OperatorFactory {
    static Map<String, Operation> operationMap = new HashMap<>();
    static {
        operationMap.put("add", new Addition());
        operationMap.put("divide", new Division());
        // more operators
    }
 
    public static Optional<Operation> getOperation(String operator) {
        return Optional.ofNullable(operationMap.get(operator));
    }
}
```

现在在*Calculator*类中，我们能够通过查询工厂来获取相关的操作并且应用于其中：

```java
public int calculateUsingFactory(int a, int b, String operator) {
    Operation targetOperation = OperatorFactory
      .getOperation(operator)
      .orElseThrow(() -> new IllegalArgumentException("Invalid Operator"));
    return targetOperation.apply(a, b);
}
```

在这个例子里，我们能看到如何通过工厂类来将逻辑业务责任分发委托给一系列轻耦合对象当中。但是如果只是简单地将嵌套if语句转移成工厂类，这明显是不符合我们的目的的。

作为另外的选择，**我们能够通过维护能够被快速查询的对象仓库*map*（映射）**，正如`OperatorFactory#operationMap`，来达成我们的目的。我们也能够在运行时定义映射对象并且配置它们用于查找。

#### 3.2 使用枚举
除了映射对象（map）的使用之外，我们也可以使用枚举来标记特定的逻辑业务。在这之后，我们能通过它来代替嵌套if语句或者swtich语句了。作为其他处理，我们也可以使用它们作为对象工厂并且整理用于处理相关的业务逻辑操作。

这会减少嵌套if语句的数量并且将业务责任委托给独立的*枚举*变量中。

让我们来看看怎么去实现它。首先，我们需要定义一个枚举类：

```java
public enum Operator {
    ADD, MULTIPLY, SUBTRACT, DIVIDE
}
```

像我们看到这样，这些值是不同操作符的标签，并且会运用到之后的计算当中。就像嵌套if语句和switch语句那样，我们可以将这些值当作选项来使用。但和它们不同的地方，让我们去设计一种能够将逻辑委托给枚举本身的替代方法吧。

我们为每一个枚举量都定义了各自的方法并且进行了计算操作，例如：

```java
ADD {
    @Override
    public int apply(int a, int b) {
        return a + b;
    }
},
// other operators
 
public abstract int apply(int a, int b);
```

然后在Calculator类中，我们也定义了一个用于执行操作的方法：

```java
public int calculate(int a, int b, Operator operator) {
    return operator.apply(a, b);
}
```

现在，我们可以通过**使用`Operator#valueOf() `方法来将字符串转换为操作符**来调用方法了：

```java
@Test
public void whenCalculateUsingEnumOperator_thenReturnCorrectResult() {
    Calculator calculator = new Calculator();
    int result = calculator.calculate(3, 4, Operator.valueOf("ADD"));
    assertEquals(7, result);
}
```

#### 3.3 命令模式（command pattern）
在先前的讨论中，我们已经看到使用工厂类来返回指定操作符的对应的业务对象实例了，稍后，业务对象实例将用于之后的Claculator中执行计算操作。

**我们也能够设计一个`Calculator#calculate`方法来接收一个可以执行输入的指令**。这是另外一种来代替嵌套if语句的方法。

首先我们定义一个*Command*接口：

```java
public interface Command {
    Integer execute();
}
```

然后，让我们实现其中的一个*AddCommand*：

```java
public class AddCommand implements Command {
    // Instance variables
 
    public AddCommand(int a, int b) {
        this.a = a;
        this.b = b;
    }
 
    @Override
    public Integer execute() {
        return a + b;
    }
}
```

最后，让我们在Calculator类中定义一个用于接收操作和执行操作的新方法：

```java
public int calculate(Command command) {
    return command.execute();
}
```

通过实例化AddCommand对象并且将他作为参数传递到`Calculator#calculate`方法当中用以调用计算方法：

```java
@Test
public void whenCalculateUsingCommand_thenReturnCorrectResult() {
    Calculator calculator = new Calculator();
    int result = calculator.calculate(new AddCommand(3, 7));
    assertEquals(10, result);
}
```

#### 3.4 规则引擎（rule engine）
当我们最终编写了大量的嵌套if语句时，每一个条件都描述了特定的业务规则，用于评估正确逻辑操作的执行。规则引擎将这些复杂的草从主代码中去掉。**规则引擎是用于评估规则并且基于输入返回结果**。

让我们通过设计一个简单的规则引擎来做下试验。这个引擎是通过一组*规则*来处理*表达式*，并从选中的*规则*返回结果。首先，我们定义一个*规则*接口：

```java
public interface Rule {
    boolean evaluate(Expression expression);
    Result getResult();
}
```

接着，我们来实现一个规则引擎：

```java
public class RuleEngine {
    private static List<Rule> rules = new ArrayList<>();
 
    static {
        rules.add(new AddRule());
    }
 
    public Result process(Expression expression) {
        Rule rule = rules
          .stream()
          .filter(r -> r.evaluate(expression))
          .findFirst()
          .orElseThrow(() -> new IllegalArgumentException("Expression does not matches any Rule"));
        return rule.getResult();
    }
}
```

这个*规则引擎*接收一个*表达式*并且返回*Result*。现在，我们设计一个带有两个数值变量以及一个用于操作的*Operator*对象的*表达式*（Expression）类：

```java
public class Expression {
    private Integer x;
    private Integer y;
    private Operator operator;        
}
```

最后，我们定义一个*AddRule*类，它只在*加*操作中使用到：

```java
public class AddRule implements Rule {
    @Override
    public boolean evaluate(Expression expression) {
        boolean evalResult = false;
        if (expression.getOperator() == Operator.ADD) {
            this.result = expression.getX() + expression.getY();
            evalResult = true;
        }
        return evalResult;
    }    
}
```

现在，我们能够使用*Expression*来调用*RuleEngine*了：

```java
@Test
public void whenNumbersGivenToRuleEngine_thenReturnCorrectResult() {
    Expression expression = new Expression(5, 5, Operator.ADD);
    RuleEngine engine = new RuleEngine();
    Result result = engine.process(expression);
 
    assertNotNull(result);
    assertEquals(10, result.getValue());
}
```

### 4. 总结
在这次教程中，我们探索了一系列不同的方法来简化复杂的代码。同时我们也学到了这么去使用有效的设计模式来取代繁杂的嵌套fi声明语句。

一如既往，读者们可以在我们的[github仓库](https://github.com/eugenp/tutorials/tree/master/core-java-8)中获取到完整的源码。

我们下期见。

### 5. 译者总结
想必大家以前或多或少都会被这种无止境的if语句所困扰，这篇文章中作者介绍了4种方法用来取代原先维护成本极高的if结构，希望看了以后对大家之后的结构设计思路有所帮助，避免出现让别人叫惨的if地狱。



 _小喇叭_ 

**广州芦苇科技Java开发团队**

芦苇科技-广州专业互联网软件服务公司

抓住每一处细节 ，创造每一个美好

关注我们的公众号，了解更多

想和我们一起奋斗吗？lagou搜索“ **芦苇科技** ”或者投放简历到 **server@talkmoney.cn**  加入我们吧

![](https://user-gold-cdn.xitu.io/2018/12/19/167c57a3e3d84dd5?w=640&h=356&f=gif&s=1273445)

关注我们，你的评论和点赞对我们最大的支持