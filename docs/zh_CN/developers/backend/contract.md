# 合约二进制开发

合约二进制是 Linera 应用的首个核心组件，其具备直接修改应用状态的能力。

创建合约需遵循以下步骤：​​定义新类型​​, 其后​​实现合约特性​​。具体实现示例如下：

```rust,ignore
pub trait Contract: WithContractAbi + ContractAbi + Sized {
    /// The type of message executed by the application.
    type Message: Serialize + DeserializeOwned + Debug;

    /// Immutable parameters specific to this application (e.g. the name of a token).
    type Parameters: Serialize + DeserializeOwned + Clone + Debug;

    /// Instantiation argument passed to a new application on the chain that created it
    /// (e.g. an initial amount of tokens minted).
    type InstantiationArgument: Serialize + DeserializeOwned + Debug;

    /// Event values for streams created by this application.
    type EventValue: Serialize + DeserializeOwned + Debug;

    /// Creates an in-memory instance of the contract handler.
    async fn load(runtime: ContractRuntime<Self>) -> Self;

    /// Instantiates the application on the chain that created it.
    async fn instantiate(&mut self, argument: Self::InstantiationArgument);

    /// Applies an operation from the current block.
    async fn execute_operation(&mut self, operation: Self::Operation) -> Self::Response;

    /// Applies a message originating from a cross-chain message.
    async fn execute_message(&mut self, message: Self::Message);

    /// Finishes the execution of the current transaction.
    async fn store(self);
}
```

本节涉及较多技术细节，我们将采用分步解析法，每次聚焦一个核心方法。

本应用需使用以下三个核心方法：， ​​数据加载​​（`load`）, ​​操作执行​​（`execute_operation`）, ​​状态存储​​（`store`）。

## ​合约生命周期

为实现应用合约，需首先定义合约类型：

```rust,ignore
linera_sdk::contract!(CounterContract);

pub struct CounterContract {
    state: CounterState,
    runtime: ContractRuntime<Self>,
}
```

此类型通常包含至少两个字段：​​持久化状态​​，前文定义的链上数据存储结构；​​运行时句柄​​，提供当前执行上下文访问能力（含交易信息查询、消息发送等接口）。
可添加其他临时字段用于存储：​​事务级数据​​，仅在当前交易执行期间有效的中间状态；​​缓存数据​​，需在交易完成后清理的临时计算结果。

当交易执行时，需通过调用 `Contract::load` 方法创建合约实例。​​运行时句柄传递​​，该方法接收供合约使用的运行时句柄；
状态加载机制​​，通过句柄加载持久化状态数据；​​实例化合约​​，在本实现中，完成状态加载后创建 `CounterContract` 实例。


```rust,ignore
    async fn load(runtime: ContractRuntime<Self>) -> Self {
        let state = CounterState::load(runtime.root_view_storage_context())
            .await
            .expect("Failed to load state");
        CounterContract { state, runtime }
    }
```

当交易成功执行完毕后，系统会触发最终步骤：调用所有已加载应用合约的 `Contract::store` 方法，完成最终状态校验并将数据持久化至存储层。该步骤可类比为执行​​析构函数​​，其核心逻辑如下：
    ​​状态校验​​，验证合约数据的完整性约束；
    ​​持久化操作​​，将最新状态写入链上存储引擎；
    ​​资源释放​​，清理临时缓存数据（若存在）。
在本实现中，状态持久化具体实现为：

```rust,ignore
    async fn store(mut self) {
        self.state.save().await.expect("Failed to save state");
    }
```

除基础状态持久化外，开发者还可实现更复杂的终结逻辑。[`Contract 终结流程章节`](../../../zh_CN/developers/advanced_topics/contract_finalize.md)将详细阐述这些高级操作（如资源回收、状态校验增强等）的实现规范。

## ​应用实例化流程​​

从字节码创建应用程序的首要步骤是​​实例化​​，其核心实现为调用合约的 `Contract::instantiate` 方法。

`Contract::instantiate` 方法具有以下双重约束：​​单次调用​​，仅在应用实例化时触发一次；​​链级隔离​​，仅能在创建该应用的所属微链上调用。

若应用状态采用​​视图范式​​，则在其他微链部署时，系统将自动采用该状态下所有子视图的`​​默认`值​​进行初始化。

针对示例应用，需通过其实例化参数在创建时指定一个​​任意数值​​作为初始状态值：

```rust,ignore
    async fn instantiate(&mut self, value: u64) {
        // Validate that the application parameters were configured correctly.
        self.runtime.application_parameters();

        self.state.value.set(value);
    }
```

## 递增操作实现

在完成计数器状态的定义与初始化方法后，需实现计数器值的递增逻辑。区块链系统中，来自区块提议者或其他应用实体的执行请求统称为​​操作​​（Operation），其核心流程如下：

处理链上操作需实现合约的 `Contract::execute_operation` 方法。以计数器场景为例，该方法接收的​​操作参数​​为 `u64` 类型数值，该值用于指定计数器的增量幅度：

```rust,ignore
    async fn execute_operation(&mut self, operation: u64) -> u64 {
        let new_value = self.state.value.get() + operation;
        self.state.value.set(new_value);
        new_value
    }
```

## ABI声明规范​​

最终，需将 `Contract` 特性的实现与应用程序的 ABI 进行绑定，建立接口与底层逻辑的关联：

```rust,ignore
impl WithContractAbi for CounterContract {
    type Abi = CounterAbi;
}
```

## 参考资料​​

- `Contract` 特性的完整定义可在[此处](https://github.com/linera-io/linera-protocol/blob/3e867613343b937b4bcdb6994a0c7b459e9d497e/linera-sdk/src/lib.rs)查阅。

- The full `Counter` example application can be found   完整的 `Counter` 示例应用可在[此处](https://github.com/linera-io/linera-protocol/tree/3e867613343b937b4bcdb6994a0c7b459e9d497e/examples/counter)查阅。
