# Setup

You will need to insert this in your `Cargo.toml`

```toml
[lib]
crate-type = ["cdylib"]

[profile.dev]
panic = "unwind"

[profile.release]
panic = "unwind"
```

Use the macro `register_module` in your `src/lib.rs` to export functions and classes

```rust, no_run
// Example:
register_module!(|module: ModuleBuilder| {
    module.with_function("one_arg", one_arg)
          .with_function("three_args", three_args)
          .with_class("my_struct", || {
              ClassBuilder::<MyStruct>::start_build()
                  .with_method("add", MyStruct::add)
                  .with_accessor("a", MyStruct::get_a)
          })
          .build()
});
```
In this example, your module with be defined in JS as:
```javascript
{
    one_arg, // function
    three_args, // function
    my_struct // class constructor
}
```
