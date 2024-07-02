- # 视图
  
  > 视图是 Linera 系统的一个特定功能，允许在内存中保存数据，然后无缝地将其刷新到底层的持久化数据存储中。
  
  完整的文档可以在 [crate 文档](https://docs.rs/linera-views/latest/linera_views/) 中找到，其中所有函数都有示例。
  
  具体来说，提供以下内容：
  
  - `View` trait 提供了 `load`、`rollback`、`clear`、`flush`、`delete` 等操作。其核心思想是我们可以对数据执行操作，然后将其刷新到存储数据的数据库中。
  - 其他几个 trait 包括 `HashableView`、`RootView`、`CryptoHashView`、`CryptoHashRootView`，对于计算哈希非常重要。
  - 多种标准容器：`MapView`、`SetView`、`LogView`、`QueueView`、`RegisterView`，它们实现了 `View` 和 `HashableView` trait。
  - 两种容器 `CollectionView` 和 `ReentrantCollectionView` 类似于 `MapView`，但其值本身是视图。
  - 派生宏允许在结构化数据类型上实现上述提到的 trait。
