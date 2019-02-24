# Basic Types

`Pinar` provides types for basic JS types, for a complete
list of their methods, please see the API documentation.

### JsString
```rust, no_run
fn method(string: JsString) -> JsResult<()> {
    let s: String = string.to_rust()?;
    Ok(())
}
```
### JsObject
```rust, no_run
fn method(obj: JsObject) -> JsResult<()> {
    let value: JsAny = obj.get("my_prop")?;
    obj.set("other_prop", HashMap::new())?;
    obj.set("abc", "cba")?;
    obj.set(12, vec![1, 2, 3])?;
    obj.has_property("test")?;
    Ok(())
}
```
### JsArray
```rust, no_run
fn method(array: JsArray) -> JsResult<()> {
    let value: JsAny = array.get(0)?;
    array.set(0, HashMap::new())?;
    array.set(1, vec![1, 2, 3])?;
    
    for value in array.iter()? {
        ...
    }
    
    Ok(())
}
```
### JsNumber
### JsSymbol
### JsExternal
`JsExternal` represents an external value.  
From the Node API documentation:  
```
This is used to pass external data through JavaScript
code, so it can be retrieved later by native code
```
A `JsExternal` can be created with either a `Box`, `Rc` or an `Arc`.
```rust, no_run
fn return_external() -> Box<usize> {
    Box::new(10) // This is converted to a JsExternal
}
fn call_with_external(fun: JsFunction) -> JsResult<()> {
    fun.call(("hey", Arc::new(10)))?;
}
```
In brief, everywhere a `ToJs` trait is expected, if you pass a `Box`,
`Rc` or `Arc`, it is converted to a `JsExternal`.

They are different ways to retrieve the external value.  

First, the tedious way:  
`JsExternal` has three methods `take_box`, `get_rc` and `get_arc`.
```rust, no_run
// Let's consider that the external value is a Box<String>
// JsExternal::take_box returns a JsResult<Option<Box<T>>>
fn external(external: JsExternal) {
    let value = external.take_box::<String>()?;
    // value is Some(Box<String>)
    
    let value = external.take_box::<String>()?;
    // The box has been taken, it's not possible to retrieve it.
    // value is None
    
    external.get_rc::<String>()?;
    // This panics, the external value is not a Rc
    
    external.take_box::<usize>()?;
    // This panics, the external value is not a usize
}

// Let's consider that the external value is a Rc<usize>
// JsExternal::get_rc returns a JsResult<Rc<T>>
fn external(external: JsExternal) {
    let value = external.get_rc::<usize>()?;
    // value is a Rc<usize>
    
    let value = external.get_rc::<usize>()?;
    // value is another Rc<usize>
    
    external.take_box::<usize>()?; // panic
    external.get_rc::<String>()?; // panic
}
```
Or the easy way:
```rust, no_run
// Here the argument will be converted directly from a 
// JsExternal to an Arc<usize>
// It will panic if the external value is not a Arc<usize>
fn external_arg(external: Arc<usize>) {
}
```

In all cases, it will panic if you try to extract a different
types (either the inner type or if it's a `Box` and you try to get a `Rc` or `Arc`)

### JsFunction
Call JS functions from Rust:
```rust, no_run
fn method(fun: JsFunction) -> JsResult<()> {
    fun.call("some_text")?; // 1 arg
    
    fun.call((1, 2, "text", vec![10, 50], Box::new(21)))?;
    // You can call a function with differents arguments
    // using a tuple
    // Each argument have to implement ToJs
    
    fun.call(())?;
    // If no argument, you need to use an empty tuple
    
    fun.call_with_this(HashMap::new(), "hello")?;
    // The `this` of the function will be a empty object
    
    fun.new_instance((1, 2))?;
    // In case the JsFunction represent a constructor, you
    // can create an instance with `new_instance`
    
    Ok(())
}
```
### JsUndefined
### JsNull
### JsBoolean
### JsBigInt
### JsAny
`JsAny` is a special type. It's an enum representing all kinds
of JS values:
```rust, no_run
pub enum JsAny<'e> {
    String(JsString<'e>),
    Object(JsObject<'e>),
    Array(JsArray<'e>),
    Number(JsNumber<'e>),
    Symbol(JsSymbol<'e>),
    External(JsExternal<'e>),
    Function(JsFunction<'e>),
    Undefined(JsUndefined<'e>),
    Null(JsNull<'e>),
    Boolean(JsBoolean<'e>),
    BigInt(JsBigInt<'e>),
}
```
See the API documentation for more details.
