# 3.3. Defining the ABI

The Application Binary Interface (ABI) of a Linera application defines how to interact with this application from other parts of the system. It includes the data structures, data types, and functions exposed by on-chain contracts and services.

ABIs are usually defined in `src/lib.rs` and compiled across all architectures (Wasm and native).

For a reference guide, check out the [documentation of the crate](https://docs.rs/linera-base/latest/linera_base/abi/).

## [Defining a marker struct](https://linera.dev/sdk/abi.html#defining-a-marker-struct)

The library part of your application (generally in `src/lib.rs`) must define a public empty struct that implements the `Abi` trait.

```rust
struct CounterAbi;
```

The `Abi` trait combines the `ContractAbi` and `ServiceAbi` traits to include the types that your application exports.

```rust
/// A trait that includes all the types exported by a Linera application (both contract
/// and service).
pub trait Abi: ContractAbi + ServiceAbi {}
```

Next, we're going to implement each of the two traits.

## [Contract ABI](https://linera.dev/sdk/abi.html#contract-abi)

The `ContractAbi` trait defines the data types that your application uses in a contract. Each type represents a specific part of the contract's behavior:

```rust
/// A trait that includes all the types exported by a Linera application contract.
pub trait ContractAbi {
    /// Immutable parameters specific to this application (e.g. the name of a token).
    type Parameters: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// Initialization argument passed to a new application on the chain that created it
    /// (e.g. an initial amount of tokens minted).
    ///
    /// To share configuration data on every chain, use [`ContractAbi::Parameters`]
    /// instead.
    type InitializationArgument: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The type of operation executed by the application.
    ///
    /// Operations are transactions directly added to a block by the creator (and signer)
    /// of the block. Users typically use operations to start interacting with an
    /// application on their own chain.
    type Operation: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The type of message executed by the application.
    ///
    /// Messages are executed when a message created by the same application is received
    /// from another chain and accepted in a block.
    type Message: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The argument type when this application is called from another application on the same chain.
    type ApplicationCall: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The argument type when a session of this application is called from another
    /// application on the same chain.
    ///
    /// Sessions are temporary objects that may be spawned by an application call. Once
    /// created, they must be consumed before the current transaction ends.
    type SessionCall: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The type for the state of a session.
    type SessionState: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The response type of an application call.
    type Response: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;
}
```

All these types must implement the `Serialize`, `DeserializeOwned`, `Send`, `Sync`, `Debug` traits, and have a `'static` lifetime.

In our example, we would like to change our `InitializationArgument`, `Operation` to `u64`, like so:

```rust
impl ContractAbi for CounterAbi {
    type InitializationArgument = u64;
    type Parameters = ();
    type Operation = u64;
    type ApplicationCall = ();
    type Message = ();
    type SessionCall = ();
    type Response = ();
    type SessionState = ();
}
```

## [Service ABI](https://linera.dev/sdk/abi.html#service-abi)

The `ServiceAbi` is in principle very similar to the `ContractAbi`, just for the service component of your application.

The `ServiceAbi` trait defines the types used by the service part of your application:

```rust
/// A trait that includes all the types exported by a Linera application service.
pub trait ServiceAbi {
    /// Immutable parameters specific to this application (e.g. the name of a token).
    type Parameters: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The type of a query receivable by the application's service.
    type Query: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The response type of the application's service.
    type QueryResponse: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;
}
```

For our Counter example, we'll be using GraphQL to query our application so our `ServiceAbi` should reflect that:

```rust
impl ServiceAbi for CounterAbi {
    type Query = async_graphql::Request;
    type QueryResponse = async_graphql::Response;
    type Parameters = ();
}
```