# ToJs

The trait `ToJs` allows you to convert Rust values to JS values automatically.  
This apply to returns values of your functions, but also to parameters
of many functions that `Pinar` provides.  

Similar to `FromArguments`, there is a macro derive to implement the
trait easily:

```rust, no_run
#[derive(Serialize, ToJs)]
struct MyCustomType {
    a: i64,
    b: i64,
}
```

### - Return value

You can now return `MyCustomType` from your functions, and it will  
be converted to javascript.  

```rust, no_run
fn custom_type(_: ()) -> MyCustomType {
    MyCustomType { a: 1, b: 2 } 
}

fn array(_: ()) -> Vec<usize> {
    vec![1, 2, 3]
}
```

### - Pinar functions

The trait `ToJs` is heavily used in `Pinar` functions,
for example:

```rust, no_run
fn modify_object(obj: JsObject) -> JsResult<()> {
    obj.set("my_prop", MyCustomType { a: 1, b: 2 })?;
    // Set the object property with MyCustomType, because
    // it implement ToJs
    
    obj.set("other_prop", Arc::new(10))?;
    // Arc<_> implements ToJs, it is converted to a 
    // JsExternal value
    
    obj.set("text", "some text abc")?;
    // &str implement ToJs and is converted to a JS
    // String
    Ok(())
}

fn call(fun: JsFunction) -> JsResult<()> {
    fun.call((
        HashMap::new(),
        MyCustomType { a: 1, b: 2 }, 
        vec![1, 2, 3]
    ))?;
    // All of these arguments implement ToJs
    Ok(())
}
```

