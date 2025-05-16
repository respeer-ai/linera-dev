# 视图(Views)

> Views是Linera系统中的一种特定结构和功能，允许在内存中维护数据，然后无缝将内存中的数据同步到持久化存储。

[完整文档](https://docs.rs/linera-views/latest/linera_views/)可以在crate文档找到，文档中所有函数都有相应示例。

具体而言，View提供了如下功能：

- 提供`load`, `rollback`, `clear`, `flush`, `delete`方法的`View` trait。Vuew的想法是我们可以对数据进行操作，然后将其刷新到存储数据的数据库。
- `HashableView`, `RootView`, `CryptoHashView`, `CryptoHashRootView`这几个traits对于hash计算比较重要。
- 标准容器`MapView`, `SetView`, `LogView`, `QueueView`, `RegisterView`实现了`View`和`HashableView` traits。
- `CollectionView`和`ReentrantCollectionView`容器与`MapView`一样，除了值就是视图本身(译者注：此处意义不太明确)。
- 派生宏(derive)允许在成员包含View的结构上实现上面提到的traits。
