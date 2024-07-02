# 创建应用程序状态

您的应用程序状态的 `struct` 可以在 `src/state.rs` 中找到。应用程序状态是在事务之间在存储上持久化的数据。

为了表示我们的计数器，我们将需要一个 `u64` 类型的变量。为了持久化计数器，我们将使用 Linera 的 [view](../advanced_topics/views.md) 范例。

Views 类似于 [ORM（对象关系映射）](https://en.wikipedia.org/wiki/Object–relational_mapping)，但是不同于将数据结构映射到像 Postgres 这样的关系数据库，它们是映射到像 [RocksDB](https://rocksdb.org/) 这样的键值存储。

在普通的 Rust 中，我们可能会这样表示我们的计数器：

```rust,ignore
// do not use this
struct Counter {
  value: u64
}
```

然而，为了持久化您的数据，您需要将 `src/state.rs` 中现有的 `Application` 状态结构替换为以下视图：

```rust,ignore
/// The application state.
#[derive(RootView, async_graphql::SimpleObject)]
#[view(context = "ViewStorageContext")]
pub struct Counter {
    pub value: RegisterView<u64>,
}
```

并且在您的整个应用程序中替换所有 `Application` 的出现。

`RegisterView<T>` 支持修改类型为 `T` 的单个值。不同类型的视图适用于不同的用例，但是大多数常见的数据结构已经实现了对应的视图类型：

- `Vec` 或 `VecDeque` 对应于 `LogView`
- `BTreeMap` 对应于 `MapView`，如果其值是原始类型，则对应于 `CollectionView` 如果其值是其他视图；
- `Queue` 对应于 `QueueView`

要查看详细列表，请参阅 Views 的 [文档](../advanced_topics/views.md)。

最后，运行 `cargo check` 来确保您的更改能够编译通过。