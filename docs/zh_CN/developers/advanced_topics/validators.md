# 验证器

验证器运行在服务器上，用户可以从验证器下载区块，或将自己微链的新区块提交给验证器。验证器验证、执行并对所有链的区块进行密码学认证。

> 在Linera中，所有微链都运行在同一组验证器上，拥有同样的安全级别。

验证器的主要功能是通过下列方式保障基础设施的完整性：

- 每一个区块都是有效的，即区块有正确的格式，区块内的操作都是允许的，消息以正确的顺序接收，和账户余额计算正确，诸如此类。
- 微链接收到的每一条消息都一定是另一条微链发送的。
- 如果一个区块在某给定高度已经被认证过，那么该高度不应该再有其他区块被认证。

只要2/3(按照质押计算)的验证器遵循协议运行，这些特性就能够得到保证。将来，背离协议的验证器将被认为是恶意的，其*质押*将被罚没。

在可用性方面，验证器也承担重要作用，所有微链的历史记录都要由验证器维护。然后，验证器并不负责为大多数微链创建区块(见[下一节](zh_CN/developers/advanced_topics/block_creation.md))，因而验证器并不保证特定的操作或消息将一定会被微链执行。取而代之的是，链所有者决定何时创建区块，以及选取哪些操作和消息打包。当前实现中，Linera客户端自动将所有接收到的消息包含在新区块。操作在这里指的是链所有者明确添加的行为，例如转账。

## [验证器架构](zh_CN/developers/advanced_topics/validators.md#验证器架构)
由于所有微链都使用同一组验证器，添加新微链并不需要添加验证器。当全部微链需要的计算能力超过验证器的能力，验证器需要支持扩容更多计算单元，这些计算单元称为“工作节点”或“物理分片”。

最终，一个Linera验证器就像一个由下列组件组成的Web2服务

- 一个负载均衡器(aka. ingress/egress)，当前由`linera-proxy`可执行文件实现，
- 一群工作节点，当前由`linera-server`可执行文件实现，
- 一个共享数据库，当前由`linera-storage`抽象接口实现。

```ignore
Linera网络示例

                    │                                             │
                    │                                             │
┌───────────────────┼───────────────────┐     ┌───────────────────┼───────────────────┐
│ validator 1       │                   │     │ validator N       │                   │
│             ┌─────┴─────┐             │     │             ┌─────┴─────┐             │
│             │   load    │             │     │             │   load    │             │
│       ┌─────┤  balancer ├────┐        │     │       ┌─────┤  balancer ├──────┐      │
│       │     └───────────┘    │        │     │       │     └─────┬─────┘      │      │
│       │                      │        │     │       │           │            │      │
│       │                      │        │     │       │           │            │      │
│  ┌────┴─────┐           ┌────┴─────┐  │     │  ┌────┴───┐  ┌────┴────┐  ┌────┴───┐  │
│  │  worker  ├───────────┤  worker  │  │ ... │  │ worker ├──┤  worker ├──┤ worker │  │
│  │    1     │           │    2     │  │     │  │    1   │  │    2    │  │    3   │  │
│  └────┬─────┘           └────┬─────┘  │     │  └────┬───┘  └────┬────┘  └────┬───┘  │
│       │                      │        │     │       │           │            │      │
│       │                      │        │     │       │           │            │      │
│       │     ┌───────────┐    │        │     │       │     ┌─────┴─────┐      │      │
│       └─────┤  shared   ├────┘        │     │       └─────┤  shared   ├──────┘      │
│             │ database  │             │     │             │ database  │             │
│             └───────────┘             │     │             └───────────┘             │
└───────────────────────────────────────┘     └───────────────────────────────────────┘
```

一个验证器内的组件通过验证器内部网络通信，尤其对于工作节点而言，不同工作节点之间直接使用远程过程调用(RPC)提交跨链消息。

不同验证器的工作节点数量可以是不同的，上面的图例中负载均衡器和共享数据库都只有一个，但生产环境中他们都是可以水平扩展的。

> 当前Linera本地测试网只有一个工作节点，使用RocksDB作为数据库。
