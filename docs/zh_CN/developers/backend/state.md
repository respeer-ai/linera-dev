# 应用状态构建​​

Linera 应用的状态由链上数据构成，这些数据在交易之间持久化存储。

应用状态的逻辑`结构体`定义位于 `src/state.rs` 文件中。为表示计数器模型，我们将采用 `u64` 整型数据。

尽管可为应用状态直接使用基础数据结构：

```rust
struct Counter {
  value: u64
}
```

通常情况下，我们采用"视图"概念实现持久化数据管理：


> [视图](https://docs.rs/linera-views/latest/linera_views/)允许应用程序将持久化数据加载至内存，并以灵活方式管理数据变更。
>
> 视图的工作原理类似于对象关系映射[ORM](https://en.wikipedia.org/wiki/Object%E2%80%93relational_mapping)框架中的持久化对象，区别在于其底层存储采用键值对形式（而非传统SQL表中的行结构）。

在此场景下，src/state.rs 文件中的结构体应替换为视图接口实现。

```rust
# extern crate linera_sdk;
# extern crate async_graphql;
# use linera_sdk::linera_base_types::*;
# use linera_sdk::*;
# use std::collections::HashSet;
# use linera_sdk::views::{linera_views, RegisterView, RootView, ViewStorageContext};
# use crate::linera_sdk::views::View as _;
/// The application state.
#[derive(RootView, async_graphql::SimpleObject)]
#[view(context = "ViewStorageContext")]
pub struct Counter {
    pub value: RegisterView<u64>,
    // Additional fields here will get their own key in storage.
}
```

项目中所有其他位置的 `Application` 应替换为 `Counter`。

派生宏 `async_graphql::SimpleObject` 与[下一节](../../../zh_CN/developers/backend/service.md)将讨论的 GraphQL 查询功能相关联。

`RegisterView<T>` 支持修改类型为 `T` 的单个值。[`linera_views`](https://docs.rs/linera-views/latest/linera_views/) 库还提供以下其他数据结构：

- `LogView` 适用于管理动态增长的数值向量；
- `QueueView` 适用于管理队列数据结构；
- `MapView` and `CollectionView` 适用于管理关联映射数据结构, `MapView`专用于处理静态值的键值映射,  `CollectionView` 当映射值为其他视图类型时使用。

如需查看不同构造类型的完整列表，请参阅对应库的官方[文档](https://docs.rs/linera-views/latest/linera_views/)。
