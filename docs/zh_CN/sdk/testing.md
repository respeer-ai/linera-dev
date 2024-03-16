# 3.10. 编写测试

Linera应用程序可以执行单元测试和集成测试保障应用发布质量，但这两种测试与我们通常说的Rust测试有一些区别。单元测试的执行环境包含一个WebAssembly虚拟机，模拟一条微链和一个应用程序，其中系统APIs使用`linera_sdk::test`帮助函数模拟。

集成测试在WebAssembly外执行，使用一个模拟的验证器执行测试，允许创建微链和添加区块来测试多条微链和多个应用交互。

应用程序应该包含两种类型的测试，其中单元测试关注应用内部的核心功能，集成测试关注在贴近真实网络的复杂环境中的应用行为。

> 大多数情况下，执行单元测试和集成测试的最简单方法是在项目目录中运行`linera project test`命令。

## [单元测试](https://linera-dev.respeer.ai/#/zh_CN/sdk/testing?id=unit-tests)

单元测试和应用程序的源码在一起(即项目的`src`目录)。与普通的Rust单元测试相比，Linera的单元测试有一些不同：

- `wasm32-unknown-unknown`目标必须选择；
- 必须使用特定的`linera-wasm-test-runner`执行器；
- 需要使用[`#[webassembly_test\]`](https://docs.rs/webassembly-test/latest/webassembly_test/)特性代替`#[test]`特性。

其中，`linera project test`命令将会自动设置前两项。

另外，开发者也可以按照[如下步骤](https://linera-dev.respeer.ai/#/zh_CN/sdk/testing?id=manually-configuring-the-environment)自行设置开发环境，执行`cargo test`执行测试。

### [示例](https://linera-dev.respeer.ai/#/zh_CN/sdk/testing?id=example)

下面的例子展示了一个简单的单元测试，用来测试应用中的`do_something`方法是否更改应用状态。

```rust
#[cfg(test)]
mod tests {
    use crate::state::ApplicationState;
    use webassembly_test::webassembly_test;

    #[webassembly_test]
    fn test_do_something() {
        let mut application = ApplicationState {
            // Configure the application's initial state
            ..ApplicationState::default()
        };

        let result = application.do_something();

        assert!(result.is_ok());
        assert_eq!(application, ApplicationState {
            // Define the application's expected final state
            ..ApplicationState::default()
        });
    }
}
```

### [模拟系统APIs](https://linera-dev.respeer.ai/#/zh_CN/sdk/testing?id=mocking-system-apis)

单元测试执行在受限环境，这样的环境中许多系统行为并不能执行，例如，不能访问KV存储，不能执行跨链消息，不能执行跨应用调用等。然而，我们可以通过模拟系统APIs，来满足单元测试的执行需求。`linera-sdk::test`模块提供了一些帮助函数模拟系统APIs。

下面是一个模拟KV存储的例子：

```rust
#[cfg(test)]
mod tests {
    use crate::state::ApplicationState;
    use linera_sdk::test::mock_key_value_store;
    use webassembly_test::webassembly_test;

    #[webassembly_test]
    fn test_state_is_not_persisted() {
        let mut storage = mock_key_value_store();

        // Assuming the application uses views
        let mut application = ApplicationState::load(storage.clone())
            .now_or_never()
            .expect("Mock key-value store returns immediately")
            .expect("Failed to load view from mock key-value store");

        // Assuming `do_something` changes the view, but does not persist it
        let result = application.do_something();

        assert!(result.is_ok());

        // Check that the state in memory is different from the state in storage
        assert_ne!(application, ApplicatonState::load(storage));
    }
}
```

### [用`cargo test`执行单元测试](https://linera-dev.respeer.ai/#/zh_CN/sdk/testing?id=running-unit-tests-with-cargo-test)

执行`linera project test`相对简单，但是如果需要单独执行`cargo test`进行测试，Cargo需要配置使用特定的`linera-wasm-test-runner`执行器。`linera-wasm-test-runner`二进制文件可以通过`cargo install linera-sdk`从代码仓库编译得到。

```bash
cd linera-protocol
cargo build -p linera-sdk --bin linera-wasm-test-runner --release
```

上面的命令编译`linera-wasm-test-runner`二进制，并将其放置在路径`linera-protocol/target/release/linera-wasm-test-runner`。

现在我们有了执行器的二进制文件，可以设置Cargo了。我们有一些不同的方法可以设置Cargo，最快的是把`CARGO_TARGET_WASM32_UNKNOWN_UNKNOWN_RUNNER`环境变量设置成上面的二进制文件路径。

持久化的方法是修改[Cargo配置文件](https://doc.rust-lang.org/cargo/reference/config.html#hierarchical-structure)中的任意一个。例如，可以将下面的内容添加到`PROJECT_DIR/.cargo/config.toml`文件中设置项目执行器路径：

```ignore
[target.wasm32-unknown-unknown]
runner = "PATH_TO/linera-wasm-test-runner"
```

设置好测试执行器之后，单元测试即可通过如下命令执行：

```bash
cargo test --target wasm32-unknown-unknown
```

此外，也可以在`PROJECT_DIR/.cargo/config.toml`文件中做下面的设置，让项目的默认编译目标为`wasm32-unknown-unknown`：

```ignore
[build]
target = "wasm32-unknown-unknown"
```

## [集成测试](https://linera-dev.respeer.ai/#/zh_CN/sdk/testing?id=integration-tests)

通常，集成测试代码与应用程序代码是分开编写的(如将应用程序代码放在`src`目录，集成测试代码放在`test`目录)。

Linera集成测试就是普通的Rust集成测试。与单元测试不同，单元测试运行在WebAssembly虚拟机中，集成测试运行在WebAssembly虚拟机之外，因此集成测试代码将会编译为主机目标文件，而非`wasm32-unknown-unknown`目标。集成测试将在隔离的虚拟机中给每条微链创建新区块，并运行每个操作。

> 集成测试可以通过`linera project test`或`cargo test`执行。

如果开发者希望通过`cargo test`运行集成测试，而又已经将`.cargo/config.toml`文件中的默认编译目标修改为`wasm32-unknown-unknown`，那么需要通过给`cargo`指定原生目标执行集成测试，例如`cargo test --target aarch64-apple-darwin`。

集成测试使用`linera_sdk::test`中的帮助类型构造模拟的Linera网络，和给微链创建区块，从而完成应用的执行和测试。

### [示例](https://linera-dev.respeer.ai/#/zh_CN/sdk/testing?id=example-1)

下面的示例展示了在不同微链的应用实例之间发送消息的简单测试：

```rust
#[tokio::test]
async fn test_cross_chain_message() {
    let (validator, application_id) = TestValidator::with_current_application(vec![], vec![]).await;

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
