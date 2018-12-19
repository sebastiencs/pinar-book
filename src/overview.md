# Overview

Pinar is a library which make writing native node modules in Rust easy.

This book is in development, pull requests are welcome.

## Small example

This snippet will introduce you to the basic features of
Pinar, please continue to read the book for more details
and features

```rust,no_run
// Exported functions to Javascript

// The argument is converted directly to a Rust String
fn one_arg(arg: String) {
    println!("received one argument: {}", arg);
}

// Multiple arguments are handled with a tuple,
// optional arguments are possible
fn three_args(args : (String, i64, Option<String>)) {
    println!("three_args: {:?}", args);
}

// Return values are converted to javascript automatically
// as long as they implement the trait ToJs
fn add((n1, n2): (usize, usize)) -> usize {
    n1 + n2
}

// All basic Rust types implements the trait ToJs
fn get_object(_: ()) -> HashMap<i64, &'static str> {
    let mut map = HashMap::new();
    map.insert(1, "hello");
    map.insert(2, "from");
    map.insert(3, "rust");
    map
}

// You can also use the macro derive ToJs to implement
// the trait easily

#[derive(Serialize, Deserialize, ToJs, FromArguments)]
struct MyStruct {
    a: i64,
    b: String,
}

fn custom_type(_a: MyStruct) -> MyStruct {
    MyStruct { 
        a: 10,
        b: "my_struct".to_string()
    }
}

// Call javascript functions from rust is possible
fn call_function((f1, f2): (JsFunction, JsFunction)) {
    // 1 argument:
    f1.call("my_parameter");
    // 3 arguments, with a tuple:
    f1.call((1, "hello", vec![1, 2, 3]));
    f2.call(Box::new(10));
}

// You can also create javascript classes
// by implementing the trait JsClass
impl JsClass for MyStruct {
    const CLASSNAME: &'static str = "MyStruct";
    type ArgsConstructor = (i64, String);

    fn constructor(arg: Self::ArgsConstructor) -> Result<Self> {
        Ok(MyStruct{ a: arg.0, b: arg.1 })
    }
}

impl MyStruct {
    // These functions will be callable from javascript
    fn add(&mut self, n: usize) -> usize {
        self.a += n;
        self.a
    }
    fn get_a(&mut self, _: Option<usize>) -> usize {
        self.a
    }
}

// You can create instance of the class in Rust 
// (or Javascript)
fn create_class(env: Env) -> Result<JsObject> {
    MyStruct::new_instance(&env, (70, String::from("test")))
}

register_module!(sebastien, |module: ModuleBuilder| {
    module.with_function("one_arg", one_arg)
          .with_function("three_args", three_args)
          .with_function("add", add)
          .with_function("get_object", get_object)
          .with_function("custom_type", custom_type)
          .with_function("call_function", call_function)
          .with_function("create_class", create_class)
          .with_class("my_struct", || {
              ClassBuilder::<MyStruct>::start_build()
                  .with_method("add", MyStruct::add)
                  .with_accessor("a", MyStruct::get_a)
          })
          .build()
});

```

From javascript:

```javascript, no_run
const module = require("./module.node");

module.one_arg("hello");
// Print 'received one argument: hello'

module.three_args("hello", 10);
// Print three_args: (hello, 10, None)'

console.log(module.add(1, 3));
// 4

console.log(module.get_object());
// { 1: "hello", 2: "from", 3: "rust" }

console.log(module.custom_type({ a: 1, b: "abc" )));
// { a: 10, b: "my_struct" } 

module.call_function(() => {}, () => {});

const c = new module.my_struct(10, "js");
// create instance of the class

c.add(100);
// call the rust function
console.log(c.a);
// c.a calls the rust function MyStruct::get_a
// 110
```
