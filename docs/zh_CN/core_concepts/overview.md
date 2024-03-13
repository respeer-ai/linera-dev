# 概览

Linera是一个去中心化的Web3基础设施，特别服务于海量活跃用户并发访问时依然需要保障服务性能的Web3应用。

Linera协议的核心在于使用同一组验证其并发运行大量称为**微链**的轻量级区块链。

## [工作机制](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/overview?id=how-does-it-work)

在Linera中，用户钱包操作用户自己的微链，微链的所有者决定何时创建新区块，以及区块中包含什么内容。如果微链中只包含所有者一个用户，我们将这样的微链称为**用户链**。

微链的所有者创建区块以处理下列事件：
- 处理从其他微链发送给本链**消息**
- 执行用户账户安全**操作**，例如给另一个用户(译者注：通常在另一条微链上)转账

需要指出的是，验证器确保所有新区块都**有效**。例如，转账操作应该来自余额足够的账户；收到的消息确实从另一条微链发出。验证器使用同样的方法验证每一条微链的区块。

Linera**应用**是一个Wasm程序，其中包含状态和操作。用户可以在一条微链上发布字节码创建并初始化应用。如果该应用在其他微链(译者注：并非发布应用的微链)被调用，应用将会被自动部署到其他微链。每个应用在每条微链上拥有独立的状态存储。

应用依赖于异步**跨链消息**确保不同微链上的应用实例跨链协作。异步消息的载荷部分(payload)在应用内部解析，系统其余部分不需要感知消息载荷。

```ignore
                               ┌───┐     ┌───┐     ┌───┐
                       Chain A │   ├────►│   ├────►│   │
                               └───┘     └───┘     └───┘
                                                     ▲
                                           ┌─────────┘
                                           │
                               ┌───┐     ┌─┴─┐     ┌───┐
                       Chain B │   ├────►│   ├────►│   │
                               └───┘     └─┬─┘     └───┘
                                           │         ▲
                                           │         │
                                           ▼         │
                               ┌───┐     ┌───┐     ┌─┴─┐
                       Chain C │   ├────►│   ├────►│   │
                               └───┘     └───┘     └───┘
```

单条微链上的应用数量可以是无限的。同一条微链上的应用实例之间通过异步调用**组合**。

最新的Linera SDK基于Rust进行开发，其上的应用也需要通过使用Rust编程语言，然后编译成Wasm应用。Rust程序员可以使用通用的Rust工具链和他们最喜欢的开发环境开发Linera应用。

## [Linera和其他多链架构的比较](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/overview?id=how-does-linera-compare-to-existing-multi-chain-infrastructure)

Linera是第一个设计支持并发执行很多链的架构，尤其是通过用户钱包操作任意数量的**用户链**。

传统多链架构中，每一条平行链由一组验证器执行，其中包含一个完整的区块链协议。这样的架构创建新的平行链或者在平行链之间交换消息成本是高昂的，因而平行链的数量通常是有限的。很多平行链只能处理特定场景，这样的链也被称为`应用链`。

相反，Linera是为了并发运行大量的用户链设计和优化的：

- 用户仅在需要的时候在自己的微链创建新区块；
- 创建新的微链不需要添加验证器；
- 所有微链具有相同安全级别；
- 微链之间通过验证器的内部网络高效通信；
- 验证器像常规Web服务一样内部分片，因此可以通过增加或减少工作节点弹性调整验证器的能力。

> 除了用户链，[Linera协议](https://linera.io/whitepaper)也支持其他类型微链，称为`许可链`和`公开链`。公开链和经典区块链一样，由验证器操作(译者注：即由验证器创建新区块，这里验证器就是矿工)。许可链通常用户用户之间的临时交互，比如原子交换。

## [为什么要在Linera上开发?](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/overview?id=why-build-on-top-of-linera)

我们认为，当前的Web3基础设施服务于**海量活跃用户**并发访问的场景时，不能保证同等服务质量(不可预测的手续费，延迟等)，导致很多高价值的应用场景不能实施。

如下的应用场景中，海量用户将会创建大量交易，这些交易需要在规定时间内完成处理：

- 实时微支付和微奖励，
- 社交数据流，
- 实时拍卖系统，
- 回合制游戏，
- 软件、数据流水线或者AI训练流水线的版本控制系统。

除了提供弹性扩展能力，轻量级用户链还给系统带来了其他好处。相比传统区块链，Linera的轻量级用户链的区块少得多，因此可以将用户链的全节点(译者注：此处指维护用户所拥有的微链的服务)嵌入到用户钱包，这些钱包通常以浏览器插件的方式部署。

这样的实现意味着连接到钱包的Web应用界面可以通过成熟的前端框架(React/GraphQL)直接(不需要对接API提供商，不需要轻客户端)查询用户链状态。此外，钱包可以以多种方式，例如向用户展示有意义的消息确认信息，增强全节点的安全性。

## [Linera开发状态](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/overview?id=what-is-the-current-state-of-the-development-of-linera)

Linera协议的[开源参考实现](https://github.com/linera-io/linera-protocol)当前正在积极开发中。参考实现中提供封装好的Web3 SDK，其中包含构建简单的Web3应用所需的必要功能，当前实现支持开发者在同一台机器上的本地网络测试他们的应用。值得指出的是，开发者已经可以基于内嵌的Wasm GraphQL服务构建(可能是响应式的)Web应用界面，并通过浏览器执行测试。

当前版本Web3 SDK有下面的已知限制：

- Web用户界面需要请求作为钱包的本地HTTP服务。该服务是临时的，且仅作为测试用途。将来的版本中，与其他区块链相同，Web用户界面将会通过安全通道连接到浏览器钱包插件。
- 本文档撰写时，Linera SDK仅能支持用户链，其他类型的微链(称为“公开链”和“许可链”)将在未来支持。

Linera的开发工作流可以拆分如下，其中包含除了SDK之外的其他开发工作：

### [核心协议](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/overview?id=core-protocol)

-  User chains
-  Permissioned chain (core protocol only)
-  Cross-chain messages
-  Cross-chain pub/sub channels (initial version)
-  Bytecode publishing
-  Application creation
-  Reconfigurations of validators
-  Initial support for gas fees
-  Initial support for storage fees and storage limits
-  External services to help users create their first chain
-  Permissioned chains (adding operation access control, demo of atomic swaps, etc)
-  Public chains (adding leader election, inbox constraints, etc)
-  Support for easy onboarding of user chains into a new application (removing the need to accept requests)
-  Improved pub/sub channels (removing the need to accept subscriptions)
-  Blob storage for applications (generalizing bytecode storage)
-  Support for archiving chains
-  Wallet-friendly chain clients (compile to Wasm/JS, do not maintain execution states for other chains)
-  General tokenomics and incentives for all stakeholders
-  Governance on the admin chain (e.g. DPoS, onboarding of validators)
-  Auditing procedures

### [集成Wasm虚拟机](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/overview?id=wasm-vm-integration)

-  Support for the Wasmer VM
-  Support for the Wasmtime VM (experimental)
-  Test gas metering and deterministic execution across VMs
-  Composing Wasm applications on the same chain (initial version)
-  Enhanced composability with "sessions"
-  Support for non-blocking (yet deterministic) calls to storage
-  Support for read-only GraphQL services in Wasm
-  Support for mocked system APIs (initial version)
-  More efficient cross-application calls
-  Improve host/guest stub generation to make mocks easier (currently wit-bindgen)
-  Compile user full node to Wasm/JS

### [存储](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/overview?id=storage)

-  Object management library ("linera-views") on top of Key-Value store abstraction
-  Support for Rocksdb
-  Experimental support for DynamoDb
-  Initial derive macros for GraphQL
-  Initial support for ScyllaDb
-  Make library fully extensible by users (requires better GraphQL macros)
-  Performance benchmarks and improvements (including faster state hashing)
-  Production-grade support for the chosen main database
-  Support global object locks (needed for dynamic sharding)
-  Tooling for debugging
-  Make the storage library easy to use outside of Linera

### [验证器基础设施](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/overview?id=validator-infrastructure)

-  Simple TCP/UDP networking (used for benchmarks only)
-  GRPC networking
-  Basic frontend (aka. proxy) supporting fixed internal shards
-  Observability
-  Initial kubernetes support in CI
-  Initial deployment using a cloud provider
-  New frontend to support dynamic shard assignment
-  Cloud integration to demonstrate elastic scaling

### [Web3 SDK](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/overview?id=web3-sdk)

-  Initial traits for contract and service interfaces
-  Support for unit testing
-  Support for integration testing
-  Local GraphQL service to query and browse system state
-  Local GraphQL service to query and browse application states
-  Use GraphQL mutations to execute operations and create blocks
-  Initial support for unit tests
-  Support for integration tests
-  Initial ABIs for contract and service interfaces
-  Allowing message sender to pay for message execution fees
-  Bindings to use native cryptographic primitives from Wasm
-  Allowing applications to pay for user fees
-  Allowing applications to use permissioned chains and public chains
-  Wallet as a browser extension (no VM)
-  Wallet as a browser extension (with Wasm VM)
