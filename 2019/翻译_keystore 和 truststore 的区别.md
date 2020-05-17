# 翻译 | keystore 和 truststore 的区别

> 原文自国外技术社区dzone，作者为 Crumb Peter，[传送门](https://dzone.com/articles/differences-between-keystore-amp-truststore)

![](http://pic.mintrumpet.fun/blog/20191027170318.png)

keystore 和 truststore 对于 SSL 证书中的通信环节中都是重要且必不可少的。在概念和结构中这两者也非常相似，因为两者都是由钥工具命令锁管理的。

truststore 是用于存储从数字证书认证机构（CA）派发来的证书的，而这些证书是用于在 SSL 连接中验证从服务器所提供的证书。另一方面，keystore 是用于存储在验证过程中被标识的密钥和身份证书。

在 SSL 的握手过程中，truststore 的工作是验证凭证，而 keystore 的工作则是提供这些凭证。这是 truststore 和 keystore 之间最关键的差别所在，但这并不是唯一一个。它们的差异在 java 当中可以表现为这些：

- truststore 是用在信任管理中，而 keystore 是用在 钥管理中。它们之间提供不同的功能
- keystore 包含密钥并且仅当服务端运行在 SSL 连接中时才会使用到，而 truststore 则存储公钥和那些由证书认证机构所颁发的证书
- 为了指定 keystore 和 truststore 的路径，我们需要在 java 中加入不同的扩展（keystore 中使用 -Djavax.net.ssl.keyStore，truststore 使用 -Djavax.net.ssl.trustStore）
- 在两者的密码当中也有区别（例如，对于 keystore，由扩展 -Djavax.net.ssl.keyStorePassword 给出，而 truststore 则由 -Djavax.net.ssl.trustStorePassword 给出）
- keystore 为每个域存储一个密钥，但 truststore 是不存储密钥的

可以使用 keytool 程序从 java keystore 中完成证书的查询、删除和添加。几乎所有的 SSL 客户端都有访问 truststore 的权限。

## keystore 和 truststore 的安全注意事项

keystore 存储着密钥，而 truststore 并没有。keystore 的安全处理非常严格，例如：

- Hadoop SSL（安全套接字层）要求所有 truststore 和 truststore 的密码需要以纯文本格式保存在配置文件中，以便能够轻易地访问到
- keystore 和密钥也是以纯文本保存，但是是只能给特定组的成员所访问
- 由于 truststore 不包含任何私密和敏感的信息，因此对于集群环境来说，一个 truststore 就足够了
- 但是对于像密码这种重要信息，truststore 和 keystore 应当是不相同的，因为 truststore 的密码是被存放在对所有人可访问的文件当中。如果和 keystore 的密码相同，就很容易受到恶意程序和黑客的攻击了

## 创建多个 truststore

如果用户使用的是默认的 truststore，他需要在 truststore 中添加或者删除 CA 认证。如果你想添加一个自定义的 truststore，你需要通过导入信任证书到新的 truststore 中才可以。在创建 truststore 的时候，一个密码会被创建或者挑选出来，并且它们因当在给定的服务当中是相同的。

keystore 存在的目的是通过密码算法来保护隐私和完整性。而密钥是用于确保私密信息的安全并且保护它们免于有害三方的侵害，并且只能由某些拥有密码的人访问。