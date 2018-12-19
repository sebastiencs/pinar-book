# Errors

When a function returns an Err(..), it results in an exception throwns
in Javascript.  

Panics in Rust also result in a exception in JS, so it won't crash
your whole app.  

Errors in `Pinar` are based on the `failure` crate.  
So if you want to return custom errors, you can implement
the trait `Fail`.

```rust, no_run
#[derive(Fail, Debug)]
pub enum MyError {
    #[fail(display = "Invalid input.")]
    WrongInput,
    #[fail(display = "Network error.")]
    NetworkError,
}

fn create() -> JsResult<()> {
    Err(MyError::NetworkError.into())
}

// When called from Javascript, this function will throw
// an exception with the error message "Network error."
fn method() -> JsResult<()> {
    create()?;
    Err(MyError::WrongInput.into())
}
```


An exception error in the Node API consists of an error message
and a code associated to it.
To have more control on the exception, you can implement the
trait `JsError`:

```rust, no_run
pub trait JsError: Fail + JsErrorAsFail {
    fn get_msg(&self) -> String {
        format!("{}", self)
    }
    fn get_code(&self) -> Option<String> {
        None
    }
}

impl JsError for MyError {
    fn get_code(&self) -> Option<String> {
        Some("MyError#3".to_string())
    }
}
```

Also, the `Env` structure allow to throws different kinds of errors
(range error, type error, ..).  
See the API documentation for more details.
