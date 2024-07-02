# 持久存储

验证者运行服务器，并且数据存储在持久存储中。因此，我们需要一个工具来处理持久存储，所以我们添加了 `linera-db` 来实现这个目的。

## 可用的持久存储

目前可用的持久存储包括 `RocksDB`、`DynamoDB` 和 `ScyllaDB`。每种存储都有其优势和劣势。

- [`RocksDB`](https://rocksdb.org/): 数据存储在磁盘上，不能在分片之间共享，但速度非常快。
- [`DynamoDB`](https://aws.amazon.com/dynamodb/): 数据存储在远程存储上，必须在 AWS 上。数据可以在分片之间共享。
- [`ScyllaDB`](https://www.scylladb.com/): 数据存储在远程存储上。数据可以在分片之间共享。

对于其他持久存储解决方案，没有根本性障碍阻止其添加。

此外，`DynamoDB` 和 `ScyllaDB` 具有表的概念，这意味着一个给定的远程位置可以用于完全独立的多个目的。

## `linera-db` 工具

在操作持久存储时，可能需要一些全局操作。命令行工具 `linera-db` 有助于执行这些操作。

功能如下：

- `list_tables`（`DynamoDB` 和 `ScyllaDB`）：列出在持久存储上创建的所有表。
- `initialize`（`RocksDB`、`DynamoDB` 和 `ScyllaDB`）：初始化持久存储。
- `check_existence`（`RocksDB`、`DynamoDB` 和 `ScyllaDB`）：检测持久存储是否存在。如果错误代码为 0，则表存在；如果错误代码为 1，则表不存在。
- `check_absence`（`RocksDB`、`DynamoDB` 和 `ScyllaDB`）：检测持久存储是否不存在。如果错误代码为 0，则表不存在；如果错误代码为 1，则表存在。
- `delete_all`（`RocksDB`、`DynamoDB` 和 `ScyllaDB`）：删除持久存储中的所有表。
- `delete_single`（`DynamoDB` 和 `ScyllaDB`）：删除持久存储中的单个表。

如果操作过程中出现错误，则返回错误代码 2；如果一切正常（对于 `check_existence` 和 `check_absence` 除外，如果与数据库的连接正确建立但结果与预期不符，则可能返回值 1）。