# 创建应用状态

`src/state.rs`文件中包含定义应用状态的`struct`，交易和事务执行时将会修改应用状态，并持久化存储。

我们使用`u64`来表示应用中的计数值，并使用Linear的[View](https://linera-dev.respeer.ai/#/zh_CN/advanced_topics/views)模型持久化存储计数值。

Views有点儿像[ORM](https://en.wikipedia.org/wiki/Object–relational_mapping)，与ORM通常用于将数据结构映射到像Postgres这样的关系数据库不完全一样的是，Views将数据结构映射到像[RocksDB](https://rocksdb.org/)一样的KV存储。

原生的Rust中，我们用如下结构描述计数器：

```terminal
// do not use this
struct Counter {
  value: u64
}
```

然而，为了持久化存储数据，我们需要将`src/state.rs`中的所有`应用`状态用下面的View替换：

```terminal
/// The application state.
#[derive(RootView, async_graphql::SimpleObject)]
#[view(context = "ViewStorageContext")]
pub struct Counter {
    pub value: RegisterView<u64>,
}
```

应用中其他需要持久化存储的状态也需要使用对应的View类型修饰。

`RegisterView<T>`支持修改一个`T`类型的数值。不同应用场景我们将会使用不同的视图类型，但是主要的公共数据类型都已经实现：

- `Vec`或`VecDeque`对应`LogView`
- 如果`BTreeMap`的元素为元类型，对应`MapView`，否则对应`CollectionView`
- `Queue`对应`QueueView`

Views的详细列表参见[文档](zh_CN/developers/advanced_topics/views.md)。

最后，执行`cargo check`编译上面的修改。
