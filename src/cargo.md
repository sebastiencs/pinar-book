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
