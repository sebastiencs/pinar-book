# Multi-threading

You can call a javascript function from differents threads
with the help of `JsFunctionThreadSafe`.  

The JS function will be executed on the main thread.

```rust, no_run
fn method(fun: JsFunction) -> JsResult<()> {
    let fun = fun.make_threadsafe::<(String, i64)>()?;
    
    std::thread::spawn(move || {
        std::thread::sleep_ms(2000);
        fun.call(("salut".to_string(), 1010));
    });
}
```

## Promise
WIP
