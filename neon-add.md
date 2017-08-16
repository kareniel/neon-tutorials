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

use neon::vm::{Call, JsResult};
use neon::js::{JsNumber};
use neon::mem::{Handle};

fn add(call: Call) -> JsResult<JsNumber> {
    let a = try!(try!(call.arguments.require(call.scope, 0)).check::<JsNumber>());
    let b = try!(try!(call.arguments.require(call.scope, 1)).check::<JsNumber>());
    let result = a.value() as f64 + b.value() as f64;

    Ok(JsNumber::new(call.scope, result))
}

register_module!(m, {
    m.export("add", add)
});
```
