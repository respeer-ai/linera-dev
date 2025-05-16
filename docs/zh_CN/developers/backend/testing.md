# 测试代码编写规范​​

Linera应用可通过标准Rust单元测试或集成测试进行验证。单元测试使用模拟运行时执行，适用于验证应用在单链独立运行时的场景；集成测试通过模拟验证器构建多链环境，支持创建链、添加区块等操作，实现跨微链及多应用交互测试。

应用应同时包含两类测试：
​​单元测试​​，聚焦应用内部核心功能；
​​集成测试​​，验证复杂网络环境下的实际行为。
两类测试均在原生Rust环境中运行。


> Rust测试可通过 `cargo test` 命令同时执行单元测试与集成测试。

## 单元测试

单元测试编写于应用源码旁（即项目 `src` 目录）。主要目的是在隔离环境中测试应用组件，外部依赖通常通过模拟实现。当 `linera-sdk` 启用 test 特性编译时，`ContractRuntime` 和 `SystemRuntime` 类型会成为模拟运行时，可针对不同测试配置特定返回值。

### 示例

以下单元测试验证 `Counter` 应用的 `execute_operation` 方法是否正确变更应用状态：

```rust,ignore
    #[test]
    fn operation() {
        let runtime = ContractRuntime::new().with_application_parameters(());
        let state = CounterState::load(runtime.root_view_storage_context())
            .blocking_wait()
            .expect("Failed to read from mock key value store");
        let mut counter = CounterContract { state, runtime };

        let initial_value = 72_u64;
        counter
            .instantiate(initial_value)
            .now_or_never()
            .expect("Initialization of counter state should not await anything");

        let increment = 42_308_u64;
        let response = counter
            .execute_operation(increment)
            .now_or_never()
            .expect("Execution of counter operation should not await anything");

        let expected_value = initial_value + increment;

        assert_eq!(response, expected_value);
        assert_eq!(*counter.state.value.get(), initial_value + increment);
    }
```

## 集成测试

集成测试通常独立于应用源码编写（即置于与 `src` 目录同级的 `tests` 目录中）。

集成测试通过调用 `linera_sdk::test` 提供的辅助类型搭建模拟的Linera网络，并向微链发布区块以执行程序。

### 示例

以下是一个简单的集成测试示例，该测试通过执行包含`Counter`应用操作的区块来验证功能。

```rust,ignore
#[tokio::test(flavor = "multi_thread")]
async fn single_chain_test() {
    let (validator, module_id) =
        TestValidator::with_current_module::<counter::CounterAbi, (), u64>().await;
    let mut chain = validator.new_chain().await;

    let initial_state = 42u64;
    let application_id = chain
        .create_application(module_id, (), initial_state, vec![])
        .await;

    let increment = 15u64;
    chain
        .add_block(|block| {
            block.with_operation(application_id, increment);
        })
        .await;

    let final_value = initial_state + increment;
    let QueryOutcome { response, .. } =
        chain.graphql_query(application_id, "query { value }").await;
    let state_value = response["value"].as_u64().expect("Failed to get the u64");
    assert_eq!(state_value, final_value);
}
```
