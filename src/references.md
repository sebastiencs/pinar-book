# References

To keep a reference of a JS object in your Rust code, you need
to use `JsRef`

```rust, no_run
struct MyClass {
    js_object: JsRef
}

impl JsClass for MyClass {
    ...
}

impl MyClass {
    fn keep_object(&mut self, obj: JsObject)
        -> JsResult<()>
    {
        let obj = obj.as_js_ref()?;
        self.js_object = obj;
    }
}

```
