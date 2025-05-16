# Defining the ABI

The Application Binary Interface (ABI) of a Linera application defines how to
interact with this application from other parts of the system. It includes the
data structures, data types, and functions exposed by on-chain contracts and
services.

ABIs are usually defined in `src/lib.rs` and compiled across all architectures
(Wasm and native).

For a reference guide, check out the
[documentation of the crate](https://docs.rs/linera-base/latest/linera_base/abi/).

## Defining a marker struct

The library part of your application (generally in `src/lib.rs`) must define a
public empty struct that implements the `Abi` trait.

```rust
struct CounterAbi;
```

The `Abi` trait combines the `ContractAbi` and `ServiceAbi` traits to include
the types that your application exports.

```rust,ignore
/// A trait that includes all the types exported by a Linera application (both contract
/// and service).
pub trait Abi: ContractAbi + ServiceAbi {}
```

Next, we're going to implement each of the two traits.

## Contract ABI

The `ContractAbi` trait defines the data types that your application uses in a
contract. Each type represents a specific part of the contract's behavior:

```rust,ignore
/// A trait that includes all the types exported by a Linera application contract.
pub trait ContractAbi {
    /// The type of operation executed by the application.
    ///
    /// Operations are transactions directly added to a block by the creator (and signer)
    /// of the block. Users typically use operations to start interacting with an
    /// application on their own chain.
    type Operation: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The response type of an application call.
    type Response: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// How the `Operation` is deserialized
    fn deserialize_operation(operation: Vec<u8>) -> Result<Self::Operation, String> {
        bcs::from_bytes(&operation)
            .map_err(|e| format!("BCS deserialization error {e:?} for operation {operation:?}"))
    }

    /// How the `Operation` is serialized
    fn serialize_operation(operation: &Self::Operation) -> Result<Vec<u8>, String> {
        bcs::to_bytes(operation)
            .map_err(|e| format!("BCS serialization error {e:?} for operation {operation:?}"))
    }

    /// How the `Response` is deserialized
    fn deserialize_response(response: Vec<u8>) -> Result<Self::Response, String> {
        bcs::from_bytes(&response)
            .map_err(|e| format!("BCS deserialization error {e:?} for response {response:?}"))
    }

    /// How the `Response` is serialized
    fn serialize_response(response: Self::Response) -> Result<Vec<u8>, String> {
        bcs::to_bytes(&response)
            .map_err(|e| format!("BCS serialization error {e:?} for response {response:?}"))
    }
}
```

All these types must implement the `Serialize`, `DeserializeOwned`, `Send`,
`Sync`, `Debug` traits, and have a `'static` lifetime.

In our example, we would like to change our `Operation` to `u64`, like so:

```rust,ignore
pub struct CounterAbi;

impl ContractAbi for CounterAbi {
    type Operation = u64;
    type Response = u64;
}
```

## Service ABI

The `ServiceAbi` is in principle very similar to the `ContractAbi`, just for the
service component of your application.

The `ServiceAbi` trait defines the types used by the service part of your
application:

```rust,ignore
/// A trait that includes all the types exported by a Linera application service.
pub trait ServiceAbi {
    /// The type of a query receivable by the application's service.
    type Query: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The response type of the application's service.
    type QueryResponse: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;
}
```

For our `Counter` example, we'll be using GraphQL to query our application so
our `ServiceAbi` should reflect that:


```rust,ignore
use async_graphql::{Request, Response};

impl ServiceAbi for CounterAbi {
    type Query = Request;
    type QueryResponse = Response;
}
```

## References

- The full trait definition of `Abi` can be found
  [here](https://github.com/linera-io/linera-protocol/blob/{{#include../../../.git/modules/linera-protocol/HEAD}}/linera-base/src/abi.rs).

- The full `Counter` example application can be found
  [here](https://github.com/linera-io/linera-protocol/blob/{{#include../../../.git/modules/linera-protocol/HEAD}}/examples/counter).
