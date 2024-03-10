# 3.2. Creating the Application State

The `struct` which defines your application's state can be found in `src/state.rs`.

To represent our counter, we're going to need a single `u64`. To persist the counter we'll be using Linera's [view](https://linera.dev/advanced_topics/views.html) paradigm.

Views are a little like an [ORM](https://en.wikipedia.org/wiki/Object–relational_mapping), however instead of mapping data structures to a relational database like Postgres, they are instead mapped onto key-value stores like [RocksDB](https://rocksdb.org/).

In vanilla Rust, we might represent our Counter as so:

```rust
// do not use this
struct Counter {
  value: u64
}
```

However, to persist your data, you'll need to replace the existing `Application` state struct in `src/state.rs` with the following view:

```rust
/// The application state.
#[derive(RootView, async_graphql::SimpleObject)]
#[view(context = "ViewStorageContext")]
pub struct Counter {
    pub value: RegisterView<u64>,
}
```

and all other occurrences of `Application` in your app.

The `RegisterView<T>` supports modifying a single value of type `T`. There are different types of views for different use-cases, but the majority of common data structures have already been implemented:

- A `Vec` or `VecDeque` corresponds to a `LogView`
- A `BTreeMap` corresponds to a `MapView` if its values are primitive, or to `CollectionView` if its values are other views;
- A `Queue` corresponds to a `QueueView`

For an exhaustive list refer to the Views [documentation](https://linera.dev/advanced_topics/views.html).

Finally, run `cargo check` to ensure that your changes compile.