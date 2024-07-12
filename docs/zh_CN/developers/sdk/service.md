# 3.5. 编写服务代码

Linera应用的第二个组件是服务程序，服务程序编译的字节码与合约字节码是分离的，且独立执行。服务程序不进行计量(意味着请求应用服务程序不消耗gas)，可以认为服务程序是应用的只读视图。

应用状态可以是复杂的，大部分时候开发者不会将全部状态暴露给与应用交互的用户，而代之以一系列预先明确定义的请求集合。

`Service` trait中定义了应用交互接口：

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

完整的`Service` trait定义参见[这里](https://github.com/linera-io/linera-protocol/blob/main/linera-sdk/src/lib.rs)。

现在让我们实现计数应用的`Service`部分。

与合约类似，我们首先定义服务类型：

```rust,ignore
pub struct CounterService {
    state: Counter,
}
```

与合约类型相同，服务类型通常定义两个字段：应用状态`state`和运行时`runtime`。我们可以忽略实作上不会用到的字段，在本例中，构造`CounterService`不需要`runtime`字段，因此我们只需要定义`state`。

我们需要生成实现服务的模板文件[WIT接口](https://component-model.bytecodealliance.org/design/wit.html)，用来导出主机(执行字节码的进程)访问服务的必要资源类型和函数。Linera SDK中有预先定义的宏来执行需要的代码生成过程，因此我们只需要在`service.rs`文件中添加如下代码即可：

```rust,ignore
linera_sdk::service!(CounterService);
```

接下来，我们需要为 `CounterService` 类型实现 `Service` trait。`Service` trait中包含初始化应用需要的全局参数，如果应用程序有自己特定的初始化参数定义，应该在trait的实现中将该参数类型重新声明为应用程序自己的初始化参数类型。本例中我们不需要传递初始化参数，因此将该参数声明为单元类型：

```rust,ignore
#[async_trait]
impl Service for CounterService {
    type Parameters = ();
}
```

与合约类似，当实现 `Service` trait 时，我们必须实现一个 `load` 构造函数，该函数携带`runtime`参数，我们可以用该参数来加载应用程序状态：

```rust,ignore
    async fn load(runtime: ServiceRuntime<Self>) -> Self {
        let state = Counter::load(ViewStorageContext::from(runtime.key_value_store()))
            .await
            .expect("Failed to load state");
        Ok(CounterService { state })
    }
```

服务是只读的，不能将任何变更持久化存储，因而没有`store`方法。

服务功能实现在`handler_query`方法中。客户端向服务发送的GraphQL请求被[`async-graphql` crate](https://github.com/async-graphql/async-graphql)处理，然后被转发到特定的处理程序，我们将在下面的章节讲解该部分的路由细节。下面的代码是实例程序的`handler_query`实现：

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

最后，与合约代码一样，我们需要将`Service`实现与ABI定义相关联：

```rust,ignore
impl WithServiceAbi for Counter {
    type Abi = counter::CounterAbi;
}
```

## [GraphQL支持](https://linera-dev.respeer.ai/#/zh_CN/sdk/service?id=adding-graphql-compatibility)

我们希望应用支持GraphQL请求，因此，我们需要一个`QueryRoot`拦截查询请求，以及一个`MutationRoot`拦截修改请求。`MutationRoot`将序列化`Operation`值，该序列化输出将被打包到新区块。

`QueryRoot`实现`value`查询方法，该方法返回计数器当前值：

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

`MutationRoot`中实现`increment`方法，序列化给定的值，合约实现将会使用该序列化结果修改当前计数值：

```rust,ignore
struct MutationRoot;

#[Object]
impl MutationRoot {
    async fn increment(&self, value: u64) -> Vec<u8> {
        bcs::to_bytes(&value).unwrap()
    }
}
```

上面的示例中我们没有包含依赖导入部分，我们将这一部分留给读者作为练习(记得导入`async_graphql::Object`)。完整的示例代码可以从GitHub上的[示例](https://github.com/linera-io/linera-protocol/blob/main/examples/counter/src/service.rs)找到。
