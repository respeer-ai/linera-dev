# 编写服务二进制文件

服务二进制是 Linera 应用程序的第二个组件。它被编译成与合约分开的字节码，并独立运行。它不计费（意味着查询应用程序的服务不会消耗 gas），可以看作是对应用程序的只读视图。

应用程序的状态可以是任意复杂的，大多数情况下，您不希望将其整体暴露给希望与您的应用程序交互的人。相反，您可能更喜欢定义一组明确定义的查询，用于与您的应用程序进行交互。

`Service` trait 是定义应用程序接口的方式。`Service` trait 的定义如下：

```rust,ignore
pub trait Service: WithServiceAbi + ServiceAbi + Sized {
    /// Immutable parameters specific to this application.
    type Parameters: Serialize + DeserializeOwned + Send + Sync + Clone + Debug + 'static;

    /// Creates a in-memory instance of the service handler.
    async fn new(runtime: ServiceRuntime<Self>) -> Self;

    /// Executes a read-only query on the state of this application.
    async fn handle_query(&self, query: Self::Query) -> Self::QueryResponse;
}
```

完整的服务 trait 定义可以在[这里找到](https://github.com/linera-io/linera-protocol/blob/{{#include ../../../.git/modules/linera-protocol/HEAD}}/linera-sdk/src/lib.rs)。

让我们为我们的计数应用程序实现 `Service`。

首先，我们类似于合约为服务创建一个新类型：

```rust,ignore
pub struct CounterService {
    state: Counter,
}
```

与 `CounterContract` 类型类似，这种类型通常有两种类型：应用程序的 `state` 和 `runtime`。如果我们不使用它们，可以省略字段，因此在这个示例中我们省略了 `runtime` 字段，因为它仅在构造 `CounterService` 类型时使用。

我们需要为实现服务 [WIT 接口](https://component-model.bytecodealliance.org/design/wit.html)生成必要的样板代码，并导出必要的资源类型和函数，以便可以执行服务。幸运的是，有一个宏来执行这些代码生成，所以只需将以下内容添加到 `service.rs`：

```rust,ignore
linera_sdk::service!(CounterService);
```

接下来，我们需要为 `CounterService` 类型实现 `Service` trait。第一步是定义 `Service` 的关联类型，即在实例化应用程序时指定的全局参数。在我们的情况下，全局参数未被使用，所以我们可以简单地指定单位类型：

```rust,ignore
#[async_trait]
impl Service for CounterService {
    type Parameters = ();
}
```

与合约类似，当实现 `Service` trait 时，我们必须实现一个 `load` 构造函数。构造函数接收运行时句柄，并应使用它来加载应用程序状态：

```rust,ignore
    async fn load(runtime: ServiceRuntime<Self>) -> Self {
        let state = Counter::load(ViewStorageContext::from(runtime.key_value_store()))
            .await
            .expect("Failed to load state");
        Ok(CounterService { state })
    }
```

服务没有 `store` 方法，因为它们是只读的，不能将任何更改持久化到存储中。

服务的实际功能从 `handle_query` 方法开始。我们将接受 GraphQL 查询，并使用 [`async-graphql` crate](https://github.com/async-graphql/async-graphql) 处理它们。为了将查询转发到下一节中将要实现的自定义 GraphQL 处理程序，我们使用以下代码：

```rust,ignore
    async fn handle_query(&mut self, request: Request) -> Response {
        let schema = Schema::build(
            // implemented in the next section
            QueryRoot { value: *self.value.get() },
            // implemented in the next section
            MutationRoot {},
            EmptySubscription,
        )
        .finish();
        schema.execute(request).await
    }
}
```

最后，与之前一样，需要以下代码将 ABI 定义整合到您的 `Service` 实现中：

```rust,ignore
impl WithServiceAbi for Counter {
    type Abi = counter::CounterAbi;
}
```

## 添加 GraphQL 兼容性

最后，我们希望我们的应用程序具有 GraphQL 兼容性。为了实现这一点，我们需要一个 `QueryRoot` 来响应查询，以及一个 `MutationRoot` 来创建可以放置在块中的序列化 `Operation` 值。

在 `QueryRoot` 中，我们只创建一个 `value` 查询，返回计数器的值：

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

在 `MutationRoot` 中，我们只创建一个 `increment` 方法，返回一个增加计数器值的序列化操作：

```rust,ignore
struct MutationRoot;

#[Object]
impl MutationRoot {
    async fn increment(&self, value: u64) -> Vec<u8> {
        bcs::to_bytes(&value).unwrap()
    }
}
```

上述代码中没有包括导入部分，留给读者练习（但请记住导入 `async_graphql::Object`）。如果您想要完整的源代码和相关测试，请查看 GitHub 上的[示例部分](https://github.com/linera-io/linera-protocol/blob/{{#include ../../../.git/modules/linera-protocol/HEAD}}/examples/counter/src/service.rs)。