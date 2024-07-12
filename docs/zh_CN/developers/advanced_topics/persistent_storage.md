# 4.2. 持久化存储

验证器在服务器上运行，其数据存储在持久化存储中，因此我们开发了`linera-db`用于管理持久化存储。

## [可用的持久化存储](https://linera-dev.respeer.ai/#/zh_CN/advanced_topics/persistent_storage?id=available-persistent-storage)

现成可用的持久化存储有`RocksDB`, `DynamoDB`和`ScyllaDB`，几种存储各有优劣。

- [`RocksDB`](https://rocksdb.org/): 数据存储在本地硬盘上，快，但不能在分片之间共享。
- [`DynamoDB`](https://aws.amazon.com/dynamodb/): 数据需要存储在托管在AWS上远程存储上，不同分片可以共享数据。
- [`ScyllaDB`](https://www.scylladb.com/): 数据存储在远程存储，分片之间可以共享数据。

支持其他持久存储解决方案不存在根本障碍。

此外，`DynamoDB`和`ScyllaDB`中具有表的概念，这样一个给定的远程存储可以支持多个不同的业务目标。

## [`linera-db`工具](https://linera-dev.respeer.ai/#/zh_CN/advanced_topics/persistent_storage?id=the-linera-db-tool)

与持久化存储交互需要支持一些全局操作，`linera-db`命令行工具实现了下列命令支持这些操作：

- `list_tables`(`DynamoDB`和`ScyllaDB`): 列出持久化存储中所有表
- `initialize`(`RocksDB`, `DynamoDB`和`ScyllaDB`): 初始化存储
- `check_existence`(`RocksDB`, `DynamoDB`和`ScyllaDB`): 测试持久化存储是否存在。返回值为0表示存在，1表示不存在
- `check_absence`(`RocksDB`, `DynamoDB`和`ScyllaDB`): 检查持久化存储是否不存在。返回0表示不存在，1表示存在
- `delete_all`(`RocksDB`, `DynamoDB`和`ScyllaDB`): 删除持久化存储中的所有表
- `delete_single`(`DynamoDB`和`ScyllaDB`): 删除持久化存储中的一张表

与常用linux命令相同，返回0表示执行正常。错误情况下，除了`check_existence` 和 `check_absence`之外的其他命令将会返回2。执行`check_existence` 和 `check_absence`时，如果与数据库连接正常但返回结果与预期不符，返回值为1。
