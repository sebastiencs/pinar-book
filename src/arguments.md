# Arguments

### - Javascript to Rust

Arguments are automatically converted to Rust values.  
The Rust types need to implement the trait `FromArguments`  


All basic Rust type implement this trait.  
`Pinar` provides a macro derive to implement the trait for your
customs types:  

```rust, no_run
#[derive(Deserialize)]
enum MyEnum {
    S(String),
    N(usize)
}

#[derive(Deserialize, FromArguments)]
struct MyCustomType {
    a: i64,
    b: String,
    c: MyEnum
}
// You can now receive MyCustomType in your function
// arguments
```

### Javascript types

If you want to have access to the javascript values,  
without converting them to Rust, you can specify the JS  
values that `Pinar` provides.  


```rust, no_run
fn my_method1(s: JsObject) {}

fn my_method2(a: i64, b: JsBigInt) {}

fn my_method3(n: JsNumber, e: JsExternal) {}
```

### - Multiple arguments

```rust, no_run
// 1 argument
fn my_method1(s: String) {}

// 3 argument
fn my_method2(a: i64, b: i64, c: MyCustomType) {}

fn my_method3(a: Vec<MyCustomType>, b: i64) {}

// Optional arguments
// You can call this function from js with or without a 2nd arg
fn my_method4(a: i64, b: Option<i64>) {}

// With 0 argument:
fn my_method5() {}
```

### Overloading

You can overload a function that can receive different
arguments:

```rust, no_run
fn method1(s: String) {}

fn method2(a: i64, b: i64) {}

register_module!(my_module, |module: ModuleBuilder| {
    module.with_function("my_function", method1)
          .with_function("my_function", method2)
    // 'my_function' correspond to the Rust functions
    // method1 and method2. The first function that
    // match the argument will be called
});

```

### Special arguments

#### Env
The `Env` argument give you access to some methods  
to manipulate javascript values.  

```rust, no_run
fn my_method1(env: Env, bigint: JsBigInt)
    -> JsResult<()>
{
    env.throw("some error")
}
```

#### JsThis
`JsThis` is the `this` of the function. It dereferences  
to a `JsAny`.

```rust, no_run
fn my_method1(env: Env, number: JsNumber, this: JsThis)
{
    let this: JsObject = this.as_jsobject().unwrap();
}
```

Note: For `Env` and `JsThis`, their place in the arguments  
      list isn't relevant.
