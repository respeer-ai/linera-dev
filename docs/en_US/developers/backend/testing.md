# Writing Tests

Linera applications can be tested using normal Rust unit tests or integration
tests. Unit tests use a mock runtime for execution, so it's useful for testing
the application as if it were running by itself on a single chain. Integration
tests use a simulated validator for testing. This allows creating chains and
adding blocks to them in order to test interactions between multiple microchains
and multiple applications.

Applications should consider having both types of tests. Unit tests should be
used to focus on the application's internals and core functionality. Integration
tests should be used to test how the application behaves on a more complex
environment that's closer to the real network. Both types of test are running in
native Rust.

> For Rust tests, the `cargo test` command can be used to run both the unit and
> integration tests.

## Unit tests

Unit tests are written beside the application's source code (i.e., inside the
`src` directory of the project). The main purpose of a unit test is to test
parts of the application in an isolated environment. Anything that's external is
usually mocked. When the `linera-sdk` is compiled with the `test` feature
enabled, the `ContractRuntime` and `SystemRuntime` types are actually mock
runtimes, and can be configured to return specific values for different tests.

### Example

A simple unit test is shown below, which tests if the method `execute_operation`
method changes the application state of the `Counter` application.

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

## Integration tests

Integration tests are usually written separately from the application's source
code (i.e., inside a `tests` directory that's beside the `src` directory).

Integration tests use the helper types from `linera_sdk::test` to set up a
simulated Linera network, and publish blocks to microchains in order to execute
the application.

### Example

A simple integration test that execution a block containing an operation for the
`Counter` application is shown below.

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
