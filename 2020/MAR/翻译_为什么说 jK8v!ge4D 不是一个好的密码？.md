# 翻译 | 为什么说 jK8v!ge4D 不是一个好的密码？

> 原文自博客发布平台medium，作者为 Jacob Bergdahl，[传送门](https://towardsdatascience.com/why-password-validation-is-garbage-56e0d766c12e?gi=a5e730d73ecf)

![](http://pic.mintrumpet.fun/blog/20200315164129.png)

首先我们来看一下这两个密码：

- jK8v!ge4D
- greenelephantswithtophats

你会认为哪个密码需要更长时间去破解？你认为哪个密码更容易被记住？这两个问题的答案都是第二个密码。但是，人们还是会理所当然地创建像第一个那样的密码。人们过往会在没有任何理由的情况下，被教导需要使用那些难以记住的密码。

今天让我们说说这个事情吧。

在互联网上有许多奇怪的标准规范。验证是其中之一。作为一个前端开发者，我需要验证用户输入到那称为输入框的内容。人们可以在上面输入自己的用户名、邮箱地址、家庭住址、邮政编号等等。前端开发者的任务是确保用户不会在这上面输入那些恶意的或者不正确的内容。

例如，邮政编号要求只允许输入空格和数字，并如果我们还知道用户生活在哪个国家，还可以将其限制为一定数量的字符。电话号码通常包含数字、加号（仅在开头）以及破折号。如果宽松点的规则还能包含括号。邮箱地址相对比较难验证，但是有一种常见的方法是，即使一个安全有效的电子邮箱没有其他特征，也必须在这其中加入 at 符号（@）。有些网站通过强制用户让他们的用户名取为定长或者强制让用户使用确定的字符来做用户名的校验，但这种验证方式没有一点用处，因为每个人的名字都是不尽相同的。

实施验证有几个原因。第一个是安全问题。验证可防止用户将那些恶意代码输入到应用中而更改数据库信息或者执行其他恶意的操作。另一个是强制特定数据类型。如果一个字段仅由特定的数字组成，数据库工程师会对此设定一个字段来只允许存储数字，这也意味着输入非数字会产生异常。

但验证最主要的原因，说实话，是为了帮助用户防止产生错误。

## 强制你的密码集

出于某些原因，前端工程师应该让用户输入那些传统上被认为是好的密码。它至少有8个字符的长度，包括大写和小写字符、数字，并且如果我们还是会感到担心，也应该加入像感叹号这种特殊字符。

下面是一个被认为是强密码的例子：jK8v!ge4D。考虑到你们会经常被要求输入像上述那样的密码，你会认为如果这被判定为一个号密码是一件非常正常的事。

但其实不然。这很愚蠢。这是一个坏的选择。

首先，有谁可以记住它吗？最终会发生的是，人们没法记到大脑上然后将其记录在某些地方，例如笔记本上。然后最终它们都被黑掉了。

其次，用户会在不同的服务上使用相同的密码，因为要记录一大堆复杂的密码是令人生厌的。当你在为一个网站创建账号时，一些神奇的代码会将你的密码转换为一串哈希（它被错误地理解为加密）。你的密码会像这样被存放在数据库上：k5ka2p8bFuSmoVT1tzOyyuaREkkKBcCNqoDKzYiJL9RaE8yMnPgh2XzzF0NDrUhgrcLwg78xs1w5pJiypEdFX。即使数据库遭受攻击，黑客也不能使用这些信息做任何事情。如果密码足够简单而且哈希算法也足够简单，那么是有可能找到原密码的。那如果使用稍微复杂的密码和正确的哈希方式，这就足够安全了。

问题是，并非是所有的应用服务会将用户的密码哈希掉。如果你在许多服务中使用相同的密码，最终可能会出现一个低级的管理服务以纯文本的形式存储你的密码在数据库当中。如果它们被入侵了，黑客将会持有你的密码然后访问你那些有相同账号密码的应用当中。这非常可怕，并且这发生的次数比人们想象中的要多得多。

这就是为什么你**必须**对不同的网站使用不同的密码。然而，今天的用户在无数的网站上都拥有账号。他们是怎么记住他们的所有的密码的？会搭理的用户会使用密码工具，但你不能指望一个普通用户会这样做。

好吧，其实有更好的方式。

## 它需要多长时间来破解？

看看这个字符串：gtypohgt。这是八个随机全小写字符。现代的计算机只需要用几分钟的时间就能暴力破解它。将一些字符替换成数字，这样的密码需要花费一个小时来破解（g9ypo3gt）。将一些字符换为大写，这会需要几天的时间来破解（g9YPo3gT）。放置一些特殊的字符，这会用到一个月（g9Y!o3gT）。

g9Y!o3gT 在技术上来说是一个完整的密码了。没有人会猜得中它，它并不在任何的常规密码的列表中，并且计算机需要花费大量的时间来破解它。问题是，这个密码对于人来说非常难记，因为没有任何的规律。

现在来看看这个：greenelephantswithtophats（戴着高帽的绿色大象）。这是一个带有24个全小写字母的喵喵。没有数字，没有随机字符，也没有什么诡异的地方。然而，这个密码也需要几千年的时间才能破解。每添加一个字符，计算机破解的时间会大量地增加。greenelephantswithtophats 也不存在于常规密码的列表中，并且也没有人会猜到它是什么。

## 这样就是一个好的密码

一个密码代表一个故事。需要一个 Facebook 的密码吗？这个 afaceforabookbutapizzaforahorse（a face for a book but a pizza for a horse） 怎么样？将它变得形象起来。我们那些特殊的记忆也是我们的强烈的记忆。突然，你拥有了一个非常强的密码，它易记而且对每一个网站都是独一无二的。密码必须是那种即使人们非常了解但就是猜不到是什么的内容。你应该不常讨论海龟吧？你之前有看过紫色的海龟吗？没有吧？把它形象化。现在你就能有属于自己的密码了：ioncesawapurpleturtleiswear（I once saw a purple turtle I swear）。这需要花费现代计算机将近数百万年的时间来破解它，但即使是你的妹妹也猜不到是什么。

这些密码很容易想象出来。flyingcarsthatcannotflyarenotflying、applesmaybegreatbutpearsarelikeheaven、goatswithshoesenjoytrainsonrainydays。没人会猜到是什么。

但是，现在一些网站不允许使用这些密码。它们会抱怨你没有使用数字、大写字母、长度太长了或者一些其他的非技术原因。

所以，你可以稍微欺骗一下系统。在所有的密码后面加上 A1!，这样就没有系统会认为这个密码无效了。现在，你具有一个拥有大写字母、数字和特殊字符的密码了。即使在所有密码中都具有这三个字符，密码的其余部分也会弥补上。ioncesawapurpleturtleiswear 和 ioncesawapurpleturtleiswearA1! 对于计算机来说都是难以破解的，这只是意味着在结尾处输入这些字符会增添些许不便。

开发者的意图是善意的。人们输入差质量的密码，网站管理员并不希望自己的网站出现什么丑闻，所以会强迫用户输入强密码，然而这其实最后会带来麻烦。

下次需要创建密码的时候，记住这个规则。要使计算机难以破解并且对你来说容易被记住，而不是反其为之。

噢，差点忘了提醒你，请记住千万不要取像 123456、password123 或者 qwerty 这样的密码。它们*确实*是一些非常差的密码。

哈哈，但我猜那也许是我们为什么要让你强制将密码改为像 jK8v!ge4D 的原因了。

回到原点。