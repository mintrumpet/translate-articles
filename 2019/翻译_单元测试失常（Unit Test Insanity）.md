#翻译 | 单元测试失常（Unit Test Insanity）

> 原文自国外技术社区dzone，作者为 John Vester，[传送门](https://dzone.com/articles/unit-test-insanity)

![你是否正在想通过重复执行你的单元测试来获得不同的测试结果？](http://pic.mintrumpet.fun/blog/20191222213332.png)

那天我在帮我女儿搬家到她的新公寓的时候，恰好是位于公寓楼的9楼，当我在等电梯的时候，我注意到一个有趣的现象。当人们在走到电梯前等电梯的时候，即使那个按钮已经被按过，现在在激活的状态下，他们都会习惯性地按下那个常见的向上箭头按钮。

我开始怀疑这时候人们做这个动作的思考模式。他们是不是认为如果这个按钮再按多几次是不是会来得快一些？或者想获得一个不同的结果？

信不信由你，这让我想起了那个我称为单元测试时常的事儿。

## 单元测试失常

从我几十年前看到我第一次写的单元测试的值的时候，我就一直提倡将单元测试加入到开发过程当中，以此来验证被测系统（SUT）的功能。然而，就像信息技术的其他事物那样，往往会出现过度构造，使得这些过程的价值变为零。

在这篇文章，我将重点放在添加单元测试的场景当中，这个场景沿用了一个已经使用过了的测试。当这种情况发生的时候，就像预测它会产生不同的结果那样，测试组件会一次又一次地执行相同的测试。

这太疯狂了。

## 一个例子

在微服务栏目中的我的“[制止单元测试重载应用上下文](https://dzone.com/articles/avoid-reloading-application-context-on-unit-tests)”的文章里面，我列举过以下的这个例子：

``` java
@RequiredArgsConstructor
@Service
public class WidgetServiceImpl implements WidgetService {
  private final AccountService accountService;
  private final WidgetRepository widgetRepository;

  /**
  * {@inheritDoc}
  */
  @Override
  public List<Widget> getWidgetsByAccountId(Long accountId, String authId) throws AccountException {
    Account account = accountService.getAccountById(accountId, authId);
    return widgetRepository.getWidgetsByAccountId(accountId);
  }
}
```

在这个非常简单的例子当中，在 `List<Widget>` 被返回之前，调用 `AccountService` 来提供某种级别下的授权或者验证功能，这样 `authId` 就具有适当的权限来查看所提供的 `accountId` 关联的 widget 信息。

如果没法获得具体适当的授权，`AccountService`  会抛出一个  `AccountException` 异常。这个异常会简单地传递给调用 `getWidgetsByAccountId()` 的对象当中。

## 单元测试覆盖范围

在上述的例子中，当 `AccountService` 被引用时，开发者在单元测试中会考虑囊括 `getAccountById()` 的所有考虑范围。

下面是其中的一些单元测试：

- 一个有效的 `authId`，对某个合法的 `accountId` 生效，并且返回一个 `Account` 对象
- 非法的 `accountId`，会抛出一个 `AccountException`
- 非法的 `authId`，会抛出一个 `AccountException`
- `authId` 对应的 `accountId` 没有适当的权限，会抛出  `AccountException`

关于 `getWidgetsByAccountId()` 方法，需要模拟一个 `AccountService` 对象并且创建一个 `when()` 实例，来说明在什么时候 `accountId` 和 `authId` 都为合法的并且最终返回一个 `account`  对象。

下面是一个非常简单的单元测试例子：

``` java
public class WidgetServiceTest extends BaseServiceTest {
  private final AccountService accountService = Mockito.mock(AccountService.class);
  private final WidgetRepository widgetRespository = Mockito.mock(WidgetRepository.class);
  WidgetServiceImpl widgetService = new WidgetService(accountService, widgetRepository);

  @Test
  void testGetWidgetsByAccountId() throws Exception {
    long accountId = 1L;
    long widgetId = 2L;
    String authId = "notARealAuthId";
    Account account = new Account();
    account.setId(accountId);
    List<Widget> widgets = new ArrayList<>();
    Widget widget = new Widget();
    widget.setId(widgetId);
    widgets.add(widget);
    when(accountService.getAccountById(accountId, authId)).thenReturn(account);
    when(widgetRepository.getWidgetsByAccountId(accountId)).thenReturn(widgets);
    List<Widget> testWidgets = widgetService.getWidgetsByAccountId(accountId, authId);
    assertEquals(widgets, testWidgets);
  }
}
```

在这个例子中，实际上是不需要测试这个简单抛出 `AccountException` 的用例。因为这个结果实际上不是被测系统（SUT）的一部分，并且已经被假定包含在 `AccountServiceTest` 类当中了。

## 结论

当在编写单元测试的时候，重点在于将大部分精力放在被测系统（SUT）上。在使用中，正在测试的方法可能会被注入服务所抛出的异常影响，如果异常方法仅将异常转发到调用 SUT 的方法中的话，那么就不需要才测试这个异常了。对于我来说，这样的行为我称为“单元测试失常”。

这个规则的例外情况是，当捕获的是从属的异常并且 SUT 的流程被更改了。在这种情况下，就像模拟一个合法的数据一样，抛出异常是非常有必要的，但是在测试中关注的重点应该是验证 SUT 中的逻辑过程是否是按照预期一样进行。

祝你打码愉快！