# ​服务二进制开发​​


服务二进制是 Linera 应用的第二个核心组件，其具有以下技术特征：
    ​​独立编译​​，与合约二进制分离编译，生成独立的字节码；
    ​​独立运行​​，作为自治服务进程运行，不依赖合约执行环境；
    ​​零Gas消耗​​，服务查询操作不产生计量费用；
    ​​只读视图​​，提供应用状态的只读访问接口。

应用状态可能具有高度复杂性，且通常情况下，您不希望向应用交互方​​完整暴露​​所有状态数据。替代方案是定义一组​​独立查询接口​​，以可控方式开放应用数据访问权限。

`服务特性`是定义应用程序交互接口的核心方式，其具体定义如下

```rust,ignore
pub trait Service: WithServiceAbi + ServiceAbi + Sized {
    /// Immutable parameters specific to this application.
    type Parameters: Serialize + DeserializeOwned + Send + Sync + Clone + Debug + 'static;

    /// Creates an in-memory instance of the service handler.
    async fn new(runtime: ServiceRuntime<Self>) -> Self;

    /// Executes a read-only query on the state of this application.
    async fn handle_query(&self, query: Self::Query) -> Self::QueryResponse;
}
```

下面我们将为计数器应用实现 `Service` 特性接口，具体步骤如下。

首先，需为服务定义新类型，其实现方式与合约类型定义类似：

```rust,ignore
linera_sdk::service!(CounterService);

pub struct CounterService {
    state: CounterState,
    runtime: Arc<ServiceRuntime<Self>>,
}
```

与 `CounterContract` 类型类似，此服务类型通常包含两个核心字段：​​应用`状态`字段​​,存储业务数据的核心结构;`​​运行时`字段​​,提供链上执行环境访问能力。
若字段未被使用可选择性省略。本示例中省略了 `runtime` 字段，该字段仅在构造 `CounterService` 类型时作为初始化参数使用

与之前示例类似，`service`! 宏会自动生成实现服务 [WIT接口](https://component-model.bytecodealliance.org/design/wit.html)所需的样板代码，并导出必要的资源类型与函数，从而确保服务具备可执行性。

下一步需为 `CounterService` 类型实现 `Service` 特性。具体步骤如下：
    ​​定义关联类型​​，声明`服务`的关联类型，该类型对应应用实例化时指定的​​全局参数​​；
    ​​单元类型简化​​，由于本示例未使用全局参数，可直接采用 () 单元类型占位。


```rust,ignore
impl Service for CounterService {
    type Parameters = ();

    // ...
}
```

与合约实现类似，在实现 `Service` 特性时需定义​​`加载`构造函数​​。该构造函数接收运行时句柄，并应通过其完成应用状态的加载：

```rust,ignore
    async fn new(runtime: ServiceRuntime<Self>) -> Self {
        let state = CounterState::load(runtime.root_view_storage_context())
            .await
            .expect("Failed to load state");
        CounterService {
            state,
            runtime: Arc::new(runtime),
        }
    }
```

服务组件不包含 store 方法，因为它是​​只读的​​，并且无法将状态变更持久化至链上存储。

服务的实际功能始于 `handle_query` 方法。我们将通过 [`async-graphql` crate](https://github.com/async-graphql/async-graphql) 库接收并处理 GraphQL 查询。为将查询转发至下节将实现的定制 GraphQL 处理程序，我们采用如下代码：

```rust,ignore
    async fn handle_query(&self, request: Request) -> Response {
        let schema = Schema::build(
            QueryRoot {
                value: *self.state.value.get(),
            },
            MutationRoot {
                runtime: self.runtime.clone(),
            },
            EmptySubscription,
        )
        .finish();
        schema.execute(request).await
    }
```

最后，与之前步骤类似，需添加如下代码以将 ABI 定义整合至`Service`实现中：

```rust,ignore
impl WithServiceAbi for CounterService {
    type Abi = counter::CounterAbi;
}
```

## GraphQL兼容性集成​​

最终，我们需要使应用程序具备GraphQL兼容性。为实现此目标，需定义以下核心组件：
    ​​查询根节点（`QueryRoot`）​​，负责处理客户端发起的GraphQL查询请求
    ​​变更根节点（`MutationRoot`）​​，用于生成可序列化的`Operation`值

在`QueryRoot`中，我们仅定义一个​​`value`查询接口​​，用于返回计数器的当前状态值：

```rust,ignore
struct QueryRoot {
    value: u64,
}

#[Object]
impl QueryRoot {
    async fn value(&self) -> &u64 {
        &self.value
    }
}
```

在`MutationRoot`中，我们仅定义一个​`increment`方法​​，其返回可序列化的操作指令，用于按客户端提供的`value`对计数器进行增量更新：

```rust,ignore
struct MutationRoot {
    runtime: Arc<ServiceRuntime<CounterService>>,
}

#[Object]
impl MutationRoot {
    async fn increment(&self, value: u64) -> [u8; 0] {
        self.runtime.schedule_operation(&value);
        []
    }
}
```

上述代码中未包含导入语句。如需查看完整源代码及相关测试，请参阅GitHub上的[示例](https://github.com/linera-io/linera-protocol/blob/3e867613343b937b4bcdb6994a0c7b459e9d497e/examples/counter/src/service.rs)部分。

## 参考资料​​

- `Service` 特性的完整定义可在[此处](https://github.com/linera-io/linera-protocol/blob/3e867613343b937b4bcdb6994a0c7b459e9d497e/linera-sdk/src/lib.rs)查阅。

- 完整的 `Counter` 示例应用可在[此处](https://github.com/linera-io/linera-protocol/tree/3e867613343b937b4bcdb6994a0c7b459e9d497e/examples/counter)查阅。
