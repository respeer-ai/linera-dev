# 3.5. 编写服务代码

Linera应用的第二个组件是服务程序，服务程序编译的字节码与合约字节码是分离的，且独立执行。服务程序不进行计量(意味着请求应用服务程序不消耗gas)，可以认为服务程序是应用的只读视图。

应用状态可以是复杂的，大部分时候开发者不会将全部状态暴露给与应用交互的用户，而代之以一系列预先明确定义的请求集合。

`Service` trait中定义了应用交互接口：

```rust
/// The service interface of a Linera application.
#[async_trait]
pub trait Service: WithServiceAbi + ServiceAbi {
    /// Type used to report errors to the execution environment.
    type Error: Error + From<serde_json::Error>;

    /// The desired storage backend used to store the application's state.
    type Storage: ServiceStateStorage;

    /// Executes a read-only query on the state of this application.
    async fn handle_query(
        self: Arc<Self>,
        context: &QueryContext,
        argument: Self::Query,
    ) -> Result<Self::QueryResponse, Self::Error>;
}
```

完整的`Service` trait定义参见[这里](https://github.com/linera-io/linera-protocol/blob/main/linera-sdk/src/lib.rs)。

现在让我们实现技术程序的`Service`部分。

首先，我们希望生成实现服务的模板文件[WIT接口](https://component-model.bytecodealliance.org/design/wit.html)，用来导出主机(执行字节码的进程)访问服务的必要资源类型和函数。Linera SDK中有预先定义的宏来执行需要的代码生成过程，因此我们只需要在`service.rs`文件中添加如下代码即可：

```rust
linera_sdk::service!(Counter);
```

接下来，让我们开始实现`计数器`应用的`Service`。如下，我们需要定义`Service`的相关类型，实现`handle_query`，并且定义`Error`类型：

```rust
#[async_trait]
impl Service for Counter {
    type Error = Error;
    type Storage = ViewStateStorage<Self>;

    async fn handle_query(
        self: Arc<Self>,
        _context: &QueryContext,
        request: Request,
    ) -> Result<Response, Self::Error> {
        let schema = Schema::build(
            // implemented in the next section
            QueryRoot { value: *self.value.get() },
            // implemented in the next section
            MutationRoot {},
            EmptySubscription,
        )
        .finish();
        Ok(schema.execute(request).await)
    }
}

/// An error that can occur during the contract execution.
#[derive(Debug, Error)]
pub enum Error {
    /// Invalid query argument; could not deserialize GraphQL request.
    #[error("Invalid query argument; could not deserialize GraphQL request")]
    InvalidQuery(#[from] serde_json::Error),
}
```

最后，与合约到吗一样，我们需要将`Service`实现与ABI定义相关联：

```rust
impl WithServiceAbi for Counter {
    type Abi = counter::CounterAbi;
}
```

## [GraphQL支持](https://linera-dev.respeer.ai/#/zh_CN/sdk/service?id=adding-graphql-compatibility)

我们希望应用支持GraphQL请求，因此，我们需要一个`QueryRoot`拦截查询请求，以及一个`MutationRoot`拦截修改请求。

```rust
struct MutationRoot;

#[Object]
impl MutationRoot {
    async fn increment(&self, value: u64) -> Vec<u8> {
        bcs::to_bytes(&value).unwrap()
    }
}

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

上面的示例中我们没有包含依赖导入部分，我们将这一部分留给读者作为练习(记得导入`async_graphql::Object`)。完整的示例代码可以从GitHub上的[示例](https://github.com/linera-io/linera-protocol/blob/main/examples/counter/src/service.rs)找到。

