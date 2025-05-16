# 数据块应用规范​​

部分应用场景需使用静态资源（如图片或其他数据）：例如 `non-fungible` 示例应用实现 NFT 时，每个 NFT 均关联一张配图。

数据块是二进制数据单元，其特性表现为
​跨链通用性​​，任一链上发布的数据块，可在全链网络中被调用；
​​格式开放性​​，数据格式与应用场景由读取方定义（如非同质化示例应用中，数据块存储NFT关联的图像文件）。

用户可通过`linera publish-data-blob`命令将文件内容作为区块操作发布至链上（输出含哈希值的新数据块ID），或通过`linera service`调用`publishDataBlob` GraphQL变更操作实现相同功能。

应用程序可通过 `runtime.read_data_blob(blob_hash)` 方法读取数据块，该操作支持跨链调用（不限数据发布源链）。客户端首次执行包含数据块读取的区块时，若本地未缓存该数据，则会自动从验证节点下载。

应用程序通过 `runtime.read_data_blob(blob_hash)` 方法实现数据块读取，该操作支持跨链调用（源链不限）。客户端首次执行含数据块读取的区块时，若本地无缓存，将自动从验证节点同步数据。

如需查看完整代码，请参阅 [`non-fungible` 合约](https://github.com/linera-io/linera-protocol/blob/3e867613343b937b4bcdb6994a0c7b459e9d497e/examples/non-fungible/src/contract.rs)及[服务实现](https://github.com/linera-io/linera-protocol/blob/3e867613343b937b4bcdb6994a0c7b459e9d497e/examples/non-fungible/src/service.rs)。