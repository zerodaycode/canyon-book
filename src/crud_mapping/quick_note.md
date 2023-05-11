# The Canyon Runtime

Before getting started with `Canyon`, there is an essential requirement to consider.


`Canyon-SQL` requires an asynchronous runtime to function, which means that the signature of functions or methods should be modified with the `async` modifier.

```rust
// from
fn a_synchronous_function(){
    // ...
}

// to
async fn an_asynchronous_function() {
    // ...
}
```

To use the asynchronous runtime, `Canyon` re-exports the `tokio` crate. Which can be enabled by modifying the main function as follows:

```rust
#[tokio::main]
async fn main() { 
    /* code in main */ 
}
```

`Canyon` also comes with a prebuilt solution to this that reduces the presented above to only this:

```rust
#[canyon]
fn main() { 
    /* code in main */ 
}
```

Either solution is acceptable. The second option is simpler and enables additional features that will be discussed later in the [migrations section](../the_migrations.md)
