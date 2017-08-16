# add two numbers

**wip!**


goal:

```js
const addon = require('../native')
console.log(addon.add(1, 1)) // 2
```

working implementation:

lib.rs
```rust
#[macro_use]
extern crate neon;

mod utils;

use utils::{CheckArgument};
use neon::vm::{Call, JsResult};
use neon::js::{JsNumber};

fn add(mut call: Call) -> JsResult<JsNumber> {
    let a = call.check_argument::<JsNumber>(0)?.value() as f64;
    let b = call.check_argument::<JsNumber>(1)?.value() as f64;

    Ok(JsNumber::new(call.scope, a + b))
}

register_module!(m, {
    m.export("add", add)
});
```

utils.rs
```rust
use neon::js::Value;
use neon::vm::{JsResult, This, FunctionCall};

pub trait CheckArgument<'a> {
  fn check_argument<V: Value>(&mut self, i: i32) -> JsResult<'a, V>;
}

impl<'a, T: This> CheckArgument<'a> for FunctionCall<'a, T> {
  fn check_argument<V: Value>(&mut self, i: i32) -> JsResult<'a, V> {
    self.arguments.require(self.scope, i)?.check::<V>()
  }
}
```
