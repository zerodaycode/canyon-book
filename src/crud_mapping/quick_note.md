# The main function

Before you getting started, you may know about one indispensable requisite.

`Canyon` needs an asincronous runtime in order to work. This means that you will
must modify the signature of your functions or methods with the async modifier.

And, for getting and asyncronous runtime, you must rely in some library to achieve it.
`Canyon` comes with a reexport of `tokio` which is a crate that will provides you
that asyncronous runtime. But, for enable your program to work with `Canyon`, 
the *main function* must be modified in this way:

```
#[tokio::main]
async fn main() { 
    /* code in main */ 
}
```

But, if you prefer, `Canyon` comes with a prebuild solution to this that reduces the presented
above to only this:

```
#[canyon]
fn main() { 
    /* code in main */ 
}
```

Whatever you choose will be fine. Second solution it's much simpler, but enables features that
won't be discussed until [here](../the_final_boss.md). This won't affect you in any bad way, 
but we're just discussing what means that up until this point.