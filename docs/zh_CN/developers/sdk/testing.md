# 编写测试

Linera 应用程序可以使用普通的 Rust 单元测试或集成测试进行测试。单元测试使用模拟运行时进行执行，因此对于像在单个链上独立运行的应用程序测试很有用。集成测试使用模拟验证器进行测试，允许创建链并向其添加块，以便测试多个微链和多个应用程序之间的交互。

应用程序应考虑同时具备这两种类型的测试。单元测试应专注于应用程序的内部和核心功能。集成测试应用于测试应用程序在更复杂的环境中的行为，这种环境更接近真实网络。

> 对于 Rust 测试，可以使用 `cargo test` 命令来运行单元测试和集成测试。

## 单元测试

单元测试与应用程序的源代码一起编写（即在项目的 `src` 目录中）。单元测试的主要目的是在隔离的环境中测试应用程序的各个部分。通常会对外部依赖进行模拟。当 `linera-sdk` 编译时启用了 `test` 特性时，`ContractRuntime` 和 `SystemRuntime` 类型实际上是模拟运行时，并且可以配置为在不同的测试中返回特定的值。

### 示例

下面是一个简单的单元测试示例，测试应用程序合约的 `do_something` 方法是否改变了应用程序状态。

```rust,ignore
#[cfg(test)]
mod tests {
    use crate::{ApplicationContract, ApplicationState};
    use linera_sdk::{util::BlockingWait, ContractRuntime};

    #[test]
    fn test_do_something() {
        let runtime = ContractRuntime::new();
        let mut contract = ApplicationContract::load(runtime).blocking_wait();

        let result = contract.do_something();

        // Check that `do_something` succeeded
        assert!(result.is_ok());
        // Check that the state in memory was updated
        assert_eq!(contract.state, ApplicationState {
            // Define the application's expected final state
            ..ApplicationState::default()
        });
        // Check that the state in memory is different from the state in storage
        assert_ne!(
            contract.state,
            ApplicatonState::load(ViewStorageContext::from(runtime.key_value_store()))
        );
    }
}
```

## 集成测试

集成测试通常与应用程序的源代码分开编写（即在 `src` 目录旁边的 `tests` 目录中）。

集成测试使用 `linera_sdk::test` 中的辅助类型来设置模拟的 Linera 网络，并向微链发布块以执行应用程序。

### 示例

下面是一个简单的测试示例，演示在不同链上的应用程序实例之间发送消息。

```rust,ignore
#[tokio::test]
async fn test_cross_chain_message() {
    let parameters = vec![];
    let instantiation_argument = vec![];

    let (validator, application_id) =
        TestValidator::with_current_application(parameters, instantiation_argument).await;

    let mut sender_chain = validator.get_chain(application_id.creation.chain_id).await;
    let mut receiver_chain = validator.new_chain().await;

    sender_chain
        .add_block(|block| {
            block.with_operation(
                application_id,
                Operation::SendMessageTo(receiver_chain.id()),
            )
        })
        .await;

    receiver_chain.handle_received_messages().await;

    assert_eq!(
        receiver_chain
            .query::<ChainId>(application_id, Query::LastSender)
            .await,
        sender_chain.id(),
    );
}
```
