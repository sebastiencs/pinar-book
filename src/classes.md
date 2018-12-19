# Classes

You can create JS classes from Rust.  
Every Rust structures that implement the trait `JsClass`  
can be use as a JS class.  

The trait is defined as follow:

```rust, no_run
pub trait JsClass : Sized + 'static {
    // The name of this class in JS
    const CLASSNAME: &'static str;
    
    // The type that the constructor of this class
    // will receive as arguments.
    // Example: (String, Arc<usize>)
    type ArgsConstructor: FromArguments;

    // Static function called to create the class
    fn constructor(args: Self::ArgsConstructor) -> Result<Self> ;

    // Optional function to define default properties
    // of this class
    fn default_properties(builder: ClassBuilder<Self>) -> ClassBuilder<Self> { builder }
}
```

An example of implemention to `JsClass`:

```rust, no_run
struct MyClass {
    state: Arc<State>,
    n: i64
}

impl MyClass {
    fn inc(&mut self) {
        self.n += 1;
    }
    // This function is used to set/get Self::n
    fn accessor_n(&mut self, new_n: Option<i64>) -> i64 {
        if let Some(new_n) = new_n {
            self.n = new_n;
        }
        self.n
    }
    fn sample(&mut self, args: (String, MyCustomType)) {}
}

impl JsClass for MyClass {
    const CLASSNAME: &'static str = "MyRustClass";
    
    type ArgsConstructor = (Arc<State>, i64);

    fn constructor((state, n): Self::ArgsConstructor)
        -> Result<Self>
    {
        Ok(MyClass { state, n })
    }

    fn default_properties(builder: ClassBuilder<Self>)
        -> ClassBuilder<Self>
    {
        builder.with_method("increment", MyClass::inc)
               .with_method("sample", MyClass::sample)
               .with_accessor("n", MyClass::accessor_n)
    }
    
}

```

There are 2 ways to make a new instance of this class:  

### - ClassBuilder

`ClassBuilder` allows you to build a `JsClass` and get  
its constructor.


```rust, no_run
fn create_class(env: &Env) -> JsResult<JsFunction> {
    let constructor = ClassBuilder::<MyClass>::start_build()
          .with_method("increment", MyClass::inc)
          .with_method("sample", MyClass::sample)
          .with_accessor("n", MyClass::accessor_n)
          .create(env)?;
    // Note that with_[method,accessor] are not necessary
    // in our case because we implemented `default_properties`
    // for our class. But I still write them for the
    // example
    
    constructor
}
```

The `JsFunction` that `create_class` returns can be used in
JS or Rust:

##### From JS

```javascript, no_run
const module = require("./module.node");

const MyClass = module.create_class();
// Get the constructor

const external_value = module.get_state();
// The function `get_state` returns an `Arc<State>`

const instance = new MyClass(external_value, 10);
// The `constructor` function from `JsClass` trait has been
// called, and we now have an instance of the class

instance.increment();
console.log(instance.n); // 11
instance.n = 200;
```

##### From Rust

```rust, no_run
fn get_instance(
    env: &Env,
    state: Arc<State>
) -> JsResult<JsObject>
{
    let constructor: JsFunction = create_class(env)?;
    constructor.new_instance((state, 10))
    // Call the `constructor` function of the `JsClass`
    // trait and return a new instance of the class
}

```

### - \<MyClass as JsClass\>::new_instance

A faster way to instanciate a JS class directly from Rust:

```rust, no_run
fn get_instance(env: Env, state: Arc<State>)
    -> JsResult<JsObject>
{
    MyClass::new_instance(&env, (state, 10))
    // Build the class & call the constructor directly
    // Returns an instance of the class
}

```

With this method, you don't need to "build" the class  
and can instantiate it directly.  

Note that you will have to define `default_properties`
function.  
Otherwise your class won't have properties.  

Also, `new_instance` doesn't convert the arguments  
from/to javascript values.  

