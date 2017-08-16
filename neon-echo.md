
# echo a string

The following tutorial will guide you through changing the code of the Neon hello_world demo to get a function that returns the string it's been passed. (And ideally we can learn a bit of Rust along the way!)

Here's the result we are trying to get at:

```js
const addon = require('../native')
console.log(addon.echo('abc')) // 'abc'
```

The first thing we need is to access our function arguments inside our addon.
At first, I expected the function we were exporting to be accepting them.

But there's already an argument there...

`fn hello(call: Call) -> ...`

`call` is a representation of the javascript function call. It has two properties:

1. `scope`: a representation of the function's scope
2. `arguments`: a representation of the function's arguments

`arguments` is a struct, a custom data type that has a bunch of fields that can themselves be of varying data types. The field that's of interest to us is called `require`. 

`require` is a method that returns an enum containing the value we are looking for. It takes two arguments: `scope` and `i`. We'll pass our function call's scope as the first argument, the index of the string that we're trying to echo as the second argument, and store that in a variable. Variable declaration in Rust is very similar to Javascript:

`let my_str = call.arguments.require(scope, 0)`

So we got our string. But it's actually wrapped up in a `struct` (JsValue) that's wrapped up in an `enum` (JsResult).

A `Result` is an `enum` used to handle errors. For now, let's just unwrap our JsValue from the JsResult by passing it to try:

`let my_str = try!(call.arguments.require(scope, 0))`

Then let's get the value of our string as a JsString (it's also wrapped in a JsResult), and ask the JsString to return it's value (as a Rust string, so we can do something with it):

`let my_str = try!(try!(call.arguments.require(scope, 0)).to_string(scope)).value()`

If we compile at this point, we will run into an error. Items from traits (like `to_string`, it's a method of the `trait` called `Value`) can only be used if the trait is in scope. So we need to add `Value` to our `use` statement:

`use neon::js::JsString;` 

to 

`use neon::js::{JsString, Value};`


We can keep the return statement like it was in the demo, and replace the "hello node" string for our Rust string. But the `new` method on JsString class takes a `&str` (I assume this is a pointer to a string, but I could be wrong) so we need to add a `&` before the name of the variable:

`Ok(JsString::new(scope, "hello node").unwrap())` 

to 

`Ok(JsString::new(scope, &my_str).unwrap())`


And that's it!


```rust
#[macro_use]
extern crate neon;

use neon::vm::{Call, JsResult};
use neon::js::{JsString, Value};

fn echo(call: Call) -> JsResult<JsString> {
    let scope = call.scope;
    let my_str = try!(try!(call.arguments.require(scope, 0)).to_string(scope)).value();

    Ok(JsString::new(scope, &my_str).unwrap())
}

register_module!(m, {
    m.export("echo", echo)
});
```
