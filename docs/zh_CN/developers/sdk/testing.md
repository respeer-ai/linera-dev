# 编写测试

Linera应用程序可以执行单元测试和集成测试保障应用发布质量。单元测试通常在Mock运行时执行，用于测试应用程序在单条微链上的运行状态。集成测试包含一个模拟验证器，允许用户创建新的微链，并添加新区块到这些微链，从而可以很方便地验证多链、多应用交互的运行状态。

应用程序应该包含两种类型的测试，其中单元测试关注应用内部的核心功能，集成测试关注在贴近真实网络的复杂环境中的应用行为。

> 运行`cargo test`命令来运行单元测试和集成测试。

## [单元测试](zh_CN/developers/sdk/testing.md#单元测试)

单元测试和应用程序的源码在一起(即项目的`src`目录)。单元测试的主要目的是在隔离的环境中测试每个应用程序组件，因此外部依赖将被Mock。如果编译`linera-sdk`时启用`test`特性，`ContractRuntime`和`SystemRuntime`类型将被Mock，并且可以在不同测试中配置特定返回值。

### [示例]

下面的例子展示了一个简单的单元测试，用来测试应用中的`do_something`方法是否更改应用状态。

```terminal
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

## [集成测试](zh_CN/developers/sdk/testing.md#集成测试)

通常，集成测试代码与应用程序代码是分开编写的(如将应用程序代码放在`src`目录，集成测试代码放在`tests`目录)。

集成测试使用 `linera_sdk::test` 中的辅助类型来Mock Linera网络，并向微链添加新区块以执行应用程序。

### [示例]

下面的示例展示了在不同微链的应用实例之间发送消息的测试：

```terminal
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
